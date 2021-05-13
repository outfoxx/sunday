---
type: Sunday - Features - Alternative Request/Response Encodings
---
# Alternative Request/Response Encodings

Sunday supports alternative request & response encodings to allow choosing between performance, efficiency and debuggability.

Coupled with [Dynamic Content-Type Negotiation](dynamic-content-type-negotiation.md), Sunday makes it easy to switch default encodings as needed or use specific encodings only on some methods.

## CBOR

[CBOR (RFC 7807)](https://www.rfc-editor.org/rfc/rfc7807) is a binary encoding format that closely matchees JSON. CBOR's close match to JSON means that using CBOR allows seemless switching between CBOR & JSON. When debuggability or platform support is important JSON can be used. While CBOR can be used when performance is of primary concern.
