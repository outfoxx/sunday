---
title: Sunday - Features - Efficient Notifications via Server-Sent Events
---
# Efficient Notifications via Server-Sent Events

Sunday supports notifications systems via Server-Sent Events as a more efficent mechanism than resorting to WebSockets.

* [Learn about Server-Sent Event](https://en.wikipedia.org/wiki/Server-sent_events)
* [Server-Sent Event Web API](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)

## RAML

The Sunday [Generator](generator/index.md) extends RAML via annotations to support easily declaring methods that produce _Server-Sent Events_. Events are defined as RAML types and therefore types are generated in each language to easily consume strongly type events in clients and produce events in server implmentations.

## Clients

Sunday client libraries ensure that an [EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource) implementation is available to consume events. Each client library also provides reactive types that adapt EventSources to produce strongly typed events that are easily subscribed to and cancelled. 

## Servers

Sunday server implementations rely on the underlying frameworks to support Server-Sent Events. Events types are generated to easily produce  events using the server framework.
