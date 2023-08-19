---
title: "Configuration"
weight: 4
summary: A list of all configuration options and what they do.
---

## Configuration

### PORT:

The port the backend should listen on.

### PROXY_PART:

The Base URL address of the proxy. You should use https://github.com/TeamPiped/piped-proxy for this. Each instance of the backend should have this on a different hostname for maximum bandwidth.

### CAPTCHA_BASE_URL:

The base of the URL of the captcha solving service. In most cases, you would use https://api.capmonster.cloud/ or https://anti-captcha.com/

### CAPTCHA_API_KEY:

The api key to use when connecting to the captcha solving service.
