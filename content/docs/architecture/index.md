---
title: "Architecture"
weight: 4
summary: How is Piped architectured?
---

## How is Piped's architecture?

Piped has 3 components:

-   A frontend in VueJS
-   A backend in Java which uses NewPipeExtractor
-   A proxy in Golang

links: https://github.com/TeamPiped/Piped\, https://github.com/TeamPiped/Piped-Backend and, https://github.com/FireMasterK/http3-ytproxy

## Frontend

-   Uses videojs
-   Uses a router for a single page application.

## Backend

-   Uses Java 11
-   Uses the native HTTP client introduced in Java 9
-   Uses netty-reactor to achieve maximum performance and a low footprint.
-   Supports OpenJ9
-   Each running instance should configure their own proxy, thus allowing multi-gigabit content delivery.
-   Uses ~70-130 MB of ram. (on OpenJ9)

# Proxy

-   Uses Golang
-   Has HTTP/2 support. (HTTP/3 is unstable in the current library)
-   Low memory footprint and high throughput.
-   Can be used to replace the proxy in various other frontends.

# Server-Side Caching

Caching is done at a Reverse-Proxy/CDN level to reduce the load to the backend. This also makes it more scalable.

# LBRY

LBRY streams are automatically added to stream content via LBRY if the same video is available there.
