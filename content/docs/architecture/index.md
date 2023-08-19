---
title: "Architecture"
weight: 4
summary: How is Piped architectured?
---

## How is Piped's architecture?

Piped has 3 components:

-   [A frontend in VueJS](https://github.com/TeamPiped/Piped)
-   [A backend in Java](https://github.com/TeamPiped/Piped-Backend) which uses [NewPipeExtractor](https://github.com/TeamNewPipe/NewPipeExtractor)
-   [A proxy in Rust](https://github.com/TeamPiped/piped-proxy)


## Frontend

-   Uses shaka-player for streaming.
-   Uses a router for a single page application.

## Backend

-   Uses Java 17
-   Uses a JNI wrapper around [Reqwest](https://github.com/seanmonstar/reqwest), a Rust HTTP client.
-   Uses ActiveJ to achieve maximum performance. Which is [really fast](https://web-frameworks-benchmark.netlify.app/result)
-   Supports OpenJ9, and Hotspot
-   Each running instance should configure their own proxy, thus allowing multi-gigabit content delivery.
-   Uses ~70-130 MB of ram. (on OpenJ9)

## Database

-   We currently support PostgreSQL, CockroachDB and YugabyteDB for high availability deployments.

# Proxy

-   Uses Rust.
-   Has HTTP/2 support.
-   Uses [actix-web](https://github.com/actix/actix-web) and [reqwest](https://github.com/seanmonstar/reqwest) for maximum performance.
-   Low memory footprint and high throughput.
-   Converts `jpeg` images to `webp` on the fly to reduce bandwidth usage.

# Server-Side Caching

Caching is done at a Reverse-Proxy/CDN level to reduce the load to the backend. This also makes it more scalable.

# LBRY

LBRY streams are automatically used to stream content via LBRY if the same video is available there.
