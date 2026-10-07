---
title: "Content Steering: Solving Multi-CDN Complexity in Modern Video Delivery"
date: 2026-10-05T23:48:59+05:30
draft: false
summary: What is content steering and how it will help solve the multi-CDN complexities in modern media delivery at scale
cover:
    image: "/assets/images/content-steering/cover.jpg"
tags: ['technical notes', 'video delivery', 'technology', 'engineering','emerging tech','content-steering','thoughts']
categories:
- software
- engineering
- leadership
- video engineering
- innovation
- architecture
---

![Content Steering](/assets/images/content-steering/cover.jpg "Content Steering: Solving Multi-CDN Complexity in Modern Media Delivery")

### Backbone of Online Video Delivery - The CDN!
The backbone of any modern large scale video delivery service is the video CDN, lets say you are in India and the server where the video files are in is in the US, the time it takes for a packet to reach your device from US will definitely be greater than if the server was in China. CDNs make sure the video you're watching is geographically closer to you so that the video chunks reach your device through a shorter path. CDN performance can largely be attributed to buffering and a bad watch experience for the end user. 

Video CDNs on a high level does the following:
- Handle concurrent requests for video streams, reducing the load at origin servers - CDNs are purpose built for concurrency and 
- Store video content across multiple geographic locations called POPs - Point of Presence's
- Route the user requests to the nearest POP with cached content, this is commonly a DNS level implementation.

![Content Steering](/assets/images/content-steering/intro_2.png "CDN")

The primary goal of a video CDN is to ensure smooth video playback by minimizing buffering, reducing latency, and maintaining consistent streaming(playback) performance while not overwhelming the origin servers.

In theory, it may seem like the CDN can single handedly manage all of the incoming traffic.

However, like everything else, this is far from the reality! When it comes to large scale video delivery, a single CDN cannot handle whole of the incoming traffic, take the example of IPL - [32 million concurrent viewers on JioCinema for the season finale](https://www.sportspro.com/news/broadcast-ott/ipl-2023-final-viewership-viacom18-jiocinema-streaming-record/), no single CDN can handle this type of load and sustain it for hours!

### Multi-CDN
For the sake of brevity, lets ignore the involement of ISPs, cloud providers and telcos in large scale video delivery. No single CDN can handle millions of users streaming for hours and hence a technique called *multi CDN* is commonly used by high traffic platforms - multiple CDNs will be responsible for serving content. This has a few advantages apart from being able to serve more traffic:
- No single point of failure - Even if one CDN fails, the other can continue serving active users.
- Lower operating costs - Platform can choose to serve more users via the cheapest CDN.

From an implementation perspective, there multiple places(and ways) at which the CDN selection can be made, mainly:

1. Backend assigns a CDN - Backend returns the CDN to be used based on rules, transparent to the player
2. Player initiated CDN switch - Player gets a list of CDNs and it can switch based on some clientside logic like http errors or chunk failures or latency.
3. DNS based CDN switch - The DNS service takes care of which CDN to be used based on rules, transparent to the player

All of these approaches have their pros and cons, like one CDN suddenly getting a spike of requests, not being able to control individual sessions(in case of DNS controlled multi CDNs), lost playback continuity when failures happen etc.

Most commonly implemented approach is the assignment by the backend:

1. User requests for a playback session
2. Backend looks at the rules for routing - network parameters, CDN capacity available, location and/or entitlements in some cases
3. The outcome of the rule evaluation will be which CDN to use - backend sends a playback URL with the chosen CDN & a fallback one
4. Playback starts with the chosen CDN
5. In case there is an issue with the primary CDN, player may choose the secondary/fallback CDN for better a UX


In hindsight, this looks like a perfect solution, until it isn't! What happens if the CDN fails midway through the stream(think live sports), what happens if the network conditions change and the user is closer to a different CDN POP or maybe a complicated use-case - how do I optmise selecting a CDN that is cheaper after a certain time without disconnecting the users who are already streaming?

### Enter Content Steering!

Content Steering is a multi CDN technique where CDN switching happens while an active playback session is in progress.

This is done using an orchestration server called "Steering Server", the Steering Server controls the following:

1. When to do the switch over - Time between switches
2. Where to switch over to  - Which CDN to use going forward

Content Steering is useful because the the ability to control when and where to switch gives greater flexibility to operators and platforms in terms of performance, reliability and cost. Complex rules can be easily applied without a lot changes.


![Content Steering](/assets/images/content-steering/steering.png "Content Steering")

This a technique introduced by Apple as part of their HLS spec in 2021 and is fully supported by the Apple ecosystem. It is a fairly new concept, however the DASH Industry Forum has adopted the specification and is now part of MPEG-DASH implementation. All modern browsers and devices supporting DASH has built in support for content steering. Its one of those less used tech that is already existing.

### How?

Content Steering is surprisingly simple from a technical perspective, only additional component to a traditional Multi CDN system is an orchestration server called the "Steering Server". When playback is requested, player instances will receive the initial playback URL as usual along with the content steering server and a TTL. The player instances will start the playback normally while pinging the content server at the specified refresh intervals.

The steering server may direct the player to use a different CDN than the one it is currently using and the player will automatically switch over to the new CDN when it is instructed to. The logic and rules that drive where the player should fetch video chunks from can get complex and dynamic.

When the players ask for steering updates, the steering server will respond with the desired CDN load balancing to each player according to the rules and state of the system. The steering server response looks something like the following:

```
{
    "VERSION": 1,
    "TTL": 60, //Next refresh TTL
    "RELOAD-URI": "https://example.com/steering",
    "PATHWAY-PRIORITY": ["CDN_X", "CDN_Y"] // Available CDN providers, CDN_X is given priority in this case
}
```

In case `CDN_Y` is to be given first priority, the server would respond with the following:

```
"PATHWAY-PRIORITY": ["CDN_Y", "CDN_X"]
```

The field `TTL` defines the interval at which the player should check for steering updates.

### Closing Thoughts

The media industry is going through a lot of changes now - consoldiation of solutions, platforms merging, costs getting cut down, complexities fading thanks to AI and what not. In an atmospehere where cost & reliability plays prime importance, novel technologies like Content Steering will definitely see a rise in popularity, extacting every last bit of value out of what is available.