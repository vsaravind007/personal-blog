---
title: "Sub Minute Lambda Invocations With AWS Cloudwatch"
date: 2022-12-01T13:17:05+05:30
summary: Invoke lambda functions periodically with shorter than 1 minute frequency
cover:
    image: "/assets/images/long-running-lambda-functions/intro.webp"
draft: false
tags: ['backend', 'nodejs', 'aws', 'lambda','cloudwatch']
categories:
- software
- aws
- architecture
---

> ***Note To The Reader*** Before jumping into the actual content, I want you, the reader to understand that there are a lot of ways to do this. When you’re making a decision, make sure to do your homework and study your requirements throughly. The approach I’m discussing here was chosen only because it was a requirement. There are far superior ways to do this!

If you’re used to building Serverless scheduled jobs(CRON jobs) using AWS Cloudwatch & Lambda, you’ll be familiar with the fact that the maximum frequency at which you can invoke your lambda is 1 minute. This means, if you need sub minute invocations, you’re out of luck out of the box. This is a requirement sometimes, like when you need to keep your DB updated with a fast changing sports score board or a commentary system that is updated multiple times per minute.

For those who are not familiar with this setup, a typical AWS Cloudwatch Event + Lambda CRON with logging enabled will have the following structure:

![Deployment diagram](/assets/images/sub-minute-lambda-invocations/deployment_diagram.webp)

Cloudwatch will trigger events periodically at the set interval without you needing to run any application code for triggering. Its pretty cheap too, costing around $1 per million events. If you convert that into minute frequency, you can easily invoke a lambda or whatever continuously over a month for under $1, because if you keep triggering something every minute for a 31 day month, the number of invocations will be 24*60*31=44,640 events, which is way under 1 million. Below is an example using Serverless Framework for the service we just discussed:

{{< gist vsaravind007 36714015de2ada189718132e95293a3a >}}

These CRON functions are very useful in building serverless application stacks, some of the use cases I can think of now are:

- Building tiny housekeeping services like log file removal
- Starting an external service sync like fetching a CSV file
- Triggering a build pipeline every ‘x’ hours etc

Cloudwatch Event + Lambda CRONs are very reliable too, [AWS says](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html) that there could be a deviation in seconds from the point of triggering and execution by lambda, at Diagnal, we use hundreds of Cloudwatch Event + Lambda CRON jobs across many projects, we are yet to see problems with Cloudwatch’s event triggering.

### What if you want to trigger a lambda every, say 10 seconds ? ###

AWS doesn’t give you this feature out of the box at the time of writing this article, maybe they’ll in future. Application requiring sub-minute lambda invocations are not that hard to find, especially in serverless application stacks. AWS does provide you with a solution to do this with AWS Step Functions, which I found hard to setup & use.

We can build such a service with Cloudwatch Events and a Lambda function that will trigger other Lambdas or services every ‘x’ seconds, logic flow a crude version with 10 second interval looks like this:

![Deployment diagram](/assets/images/sub-minute-lambda-invocations/logic_flow.webp)

For the example I’ve used Nodejs along with the library nanotimer, For this use-case, we cannot use the built in `setInterval()` as it will not give us the accuracy we’re looking for, hence the nanotimer library. It provides a very accurate timer function, suitable for our needs.

{{< gist vsaravind007 e31098e4e922dd7fbd781500e4fc7288 >}}

If you look at the code above, its pretty much what we have on the flow diagram just above, to get this working, you’ll have to assign correct IAM roles for the source Lambda function as we’re using the SDK function `lambda.invoke()`, notice that I’ve used the InvocationType as `Event` for async invocation. You can invoke one function or multiple ones as you wish, I’m just triggering one for brevity.

Below is the serverless definition file for our example source lambda and destination lambda:

{{< gist vsaravind007 137cbc7b1b55b2f874ddea85017f9601 >}}

If you look at the timeout config for the source function, it is set to61 seconds, after 61 seconds, the lambda will be terminated by AWS. After deployment, our triggering function will invoke the destination lambda every 10 seconds, 6 times per source lambda invocation ideally. Since we have the timeout config set at 61 seconds, after 61 seconds, our source lambda will be terminated & at the same time, Cloudwatch will trigger a new source lambda. This will continue till you delete the service or disable the event from Cloudwatch console.

For realistic applications I won’t be triggering them every 1 second, the frequency has to be much lower, anything >10 seconds should work just fine without issues, you’ll have to adjust other variables in play accordingly.

### Conclusion/Things to keep in mind ### 

- Never, I mean NEVER go with this approach if you have the flexibility to try other options.
- Do not use this approach for extremely time critical applications, this is great for applications where 2 or 3second deviations are fine as per design.
- If you invoke many functions at sub minute frequency, get ready for spikes in lambda usage costs.
- Make sure that your destination lambda processing will be over before the next invocation, for example if my invocation interval is 10 seconds and my destination lambda takes 12 seconds to complete its job, it won’t be sensible for us to keep triggering it every 10 seconds, it’ll end up in multiple f#kups with race conditions and others.
- Keep in mind that you are invoking a lambda function every minute, you’ll need to count in the cost for running this lambda, it’ll cost around $4.49 per month for this lambda alone. Use can use the pricing calculator here: https://aws.amazon.com/lambda/pricing/
- Disable Cloudwatch logs for the source Lambda before rolling into production, it doesn’t make much sense to simply log invocations unless its necessary, you’ll save a few bucks there.