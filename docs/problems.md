---
title: Sunday - Features - Detailed Error Responses via Problems
---
# Detailed Error Responses via Problems

Sunday supports detailed error responses based on [Problem Details for HTTP APIs (RFC 7807)](https://www.rfc-editor.org/rfc/rfc7807). The support is complete and baked into the [Generator](../sunday-generator) as well each of the client library implementations and all generated server stubs.

## RAML

The Sunday [Generator](../sunday-generator) extends RAML via annotations to support easy declaration and usage of problems in API definitions. Using Sunday's RAML annotations, problems can be defined globally and then methods can reference those problem defintions to easily add detailed responses without redeclaration.

## Clients

Sunday client implementations map problem responses to language specific error types. For example, in Kotlin problems are deserialized as exceptions and the service methods throw the exception instead of returning a value, allowing consumers to catch exceptions explicitly and easily.

## Servers

Sunday servers implementations suppoert the ability to throw (or otherwise return) language specific error types from method stub implementtions. For example, in Kotlin, problems types are generated from problem declarations as exceptions and the stub implementations are allowed to throw the exceptions as responses.

