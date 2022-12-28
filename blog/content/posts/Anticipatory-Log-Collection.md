---
title: "Anticipatory Log Collection"
date: 2022-12-28T13:07:40+05:30
draft: false
summary: Implementing long running lambda functions in AWS
cover:
    image: "/assets/images/anticipatory-log-collection/cover.jpeg"
tags: ['backend', 'nodejs', 'aws', 'lambda','cloudwatch']
categories:
- software
- aws
- architecture
---

> ***Note To The Reader*** Idea of this article is to provide insights into the importance of having accurate logs that provide insights into the application functioning and proactive log collection. This should also provide an idea on how to optimise the technique for production use. Having access to sub-low level logs will save a lot of time during the first few weeks after a launch.

![Logs](/assets/images/anticipatory-log-collection/cover.jpeg "Logs")

Anticipatory log collection is a technique used in software engineering to proactively collect log data from a system or application before an error or issue occurs. This can be useful for a number of reasons, including debugging, monitoring, and troubleshooting.Traditional log collection methods involve waiting for an error or issue to occur and then collecting the logs after the fact. This can be problematic because the logs *may not capture the full context of the issue, or they may be incomplete or missing important data*.

By contrast, anticipatory log collection involves actively collecting log data in real-time, even when the system is functioning normally. This allows engineers to see the entire sequence of events leading up to an issue, providing a more complete picture of what went wrong. This can be especially useful when dealing with complex systems or applications that may have multiple dependencies or interactions.

To begin with, the logging system must be robust & well defined. Log levels must be defined and usage within code should be peer reviewed. Following table should prorivde a high level overview:

| Log level | Description                                                                                                           |
|---------------|---------------------------------------------------------------------------------------------------------------------------|
| debug         | When this is the log level, everything should be logged - applications states at different check points, details external API calls etc - covers error & info logs as well |
| info          | General application flow logs - details about critical checkpoints - no debug logs will be generated.                                                       |
| error         | When this is the selected log level, only errors should be logged - Make sure to log everything related to that error.    |

Log entries should provide maximum details to the reader/reviewer - They should be meaningful in terms of log level, details it provides and MUST be sequential with the execution flow. 

Having sequential logs might not work for serverless systems or systems that handle very high traffic - For these kind of systems, rely on a session id or a token that will be common for session which should be part of logs to enable easier filtering & correlation, this unfortunately has a side effect of interfereing with edge caching, handling needs to be there in code/infra to manage this. Logging framework should be part of the application and *not an after thought!*

Identify critical and non-critical parts of the system - A payment system will be critical while a favourite API might not be as critical. Critical systems should have anticipatory logs enabled at all times with a sensible retention period - retention period should be based on the use-case and application domain. If your service/product is offering realtime consumable content or time critical service, keeping logs older than 2 weeks might not be logical.

### Log Reviews ###
A technology/support/operations stake holder must have knowledege about the available logging system within the product and should also be well aware of the limitations. Reviewing logs during development time is an important excercise that will save a lot of headaches & unhapppy customers - log reviews MUST be part of sprint planning similar to code reviews(Ideally log reviews should be done by support/TecOps teams). One way to confirm the effectiveness of available logs within a newly implemented system would be to do a scenario testing by the Support/TecOps team.

### Practical Approach ###
Anticipatory log collection is an expensive technique especially if the system is expecting high traffic. Eventhough logs are simple text entries, high traffic systems can generate millions of lines consuming gigabytes of storage space every day. Interest to keep anticipatory logs ON can wear off if there are no issues for a while which is equivalent to flying blind. Following are some of the key takeaways:

1. Collect a lot of logs - Collecting & reviewing operational logs for a period of about a week after first launch will help in fine tuning the logging system.
2. Know what and when to stop - Anticipatory log collection from an entire app won't provide a lot of info, log collection from non critical systems can be kept to a minimum.
3. Configurable log retention - Log retention is an important factor for keeping costs in check, having a system with configurable log retention will be helpful.
4. Continous improvement - After a week of app soaking in production will generate a lot of insights into the operational aspects of the application, logging improvements will be a key take away and continous log reviews must be done religiously.

