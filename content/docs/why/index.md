---
title: "Why"
weight: 4
summary: Why did I create Piped?
---

## Why did I create Piped?

YouTube has an extremely invasive privacy policy which relies on using user data in unethical ways.

Here are some things about YouTube:

-   Tracking via third-party cookies for other purposes without your consent.
-   YouTube can delete your content if you violate the terms
-   Reduction of legal period for cause of action
-   YouTube may use your personal information for marketing purposes
-   YouTube can view your browser history
-   YouTube can use your content for all their existing and future services
-   YouTube gathers information about you through third parties
-   YouTube can license user content to third parties
-   YouTube provider makes no warranty regarding uninterrupted, timely, secure or error-free service
-   Deleted videos are not really deleted
-   Your data may be processed and stored anywhere in the world
-   YouTube is only available to users over a certain age
-   YouTube can suspend your account for several reasons
-   YouTube has non-exclusive use of your content
-   The court of law governing the terms is in the US
-   YouTube collects your IP address for location use

Source: https://tosdr.org/en/service/274

A lot of inspiration came from NewPipe and Invidious.

I created Piped to fix issues in NewPipe and Invidious which are architectural issues and cannot be fixed easily.

### NewPipe

-   Your IP is exposed to YouTube.
-   Feeds are slow to load.

### Invidious

-   Uses way too much resources.
-   Total bandwidth limited by the peak capacity of the load balancer.
-   Coded in Crystal, a language that is relatively hard for beginners.
-   Caching is done at a backend level.
-   Invidious was a learning project.
-   Invidious crashes all the time.
-   Various hacks are required to keep an instance running at a reasonable stablity.

However, there are some drawbacks of Piped:

-   JavaScript is required
-   Browsers without Service-Workers support will feel significantly slower. Eg: Tor Browser
