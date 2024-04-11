---
title: "API Documentation"
weight: 4
summary: A guide on how to use Piped's API.
---

## API Servers

A list of public instances can be found at https://github.com/TeamPiped/Piped/wiki/Instances

To keep up-to date with the instances list, you are expected to dynamically parse it.

An example on how this can be done is available at https://github.com/TeamPiped/Piped/blob/85e2296dc410fd20375d623465900c55b5483da9/src/components/Preferences.vue#L202-L221

## Base URL

For all endpoints in this documentation, you will have to use an API url as a prefix. For example, the official instance's base URL would be https://pipedapi.kavin.rocks.

## API Paths

The OpenAPI specification for Piped is located at https://github.com/TeamPiped/OpenAPI/main/swagger.yaml. In order to properly view and inspect the available API routes, you
can open https://petstore.swagger.io in a web browser, paste https://raw.githubusercontent.com/TeamPiped/OpenAPI/main/swagger.yaml into the text field at the top and finally
click explore. Swagger UI allows you to test the documented API paths and see the required and returned parameters for requests and their responses.
