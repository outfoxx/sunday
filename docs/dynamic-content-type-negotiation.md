---
title: Sunday - Features - Dynamic Content-Type Negotation
---
# Dynamic Content-Type Negotation

Sunday has extensive support for dynamic content negotation. This feature allows clients to select the encoding method to use per method or per service or as defaults.

## Clients

Sunday client implementations support standard formats like JSON, URL Encoded, Binary, etc. Additionally, client implementations allow easy extension by registering custom [Media-Type](https://en.wikipedia.org/wiki/Media_type) encoders and/or decoders to handle any type imaginable wether it be standards based or custom.

## Servers

Sunday server support for media-type negotiation relies on the underlying framework (e.g JAX-RS).
