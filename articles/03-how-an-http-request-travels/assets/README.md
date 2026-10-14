# Supporting Assets — HTTP Request Explainer

- [successful-http1_1-curl-trace.txt](successful-http1_1-curl-trace.txt) — Sanitized verbose curl output documenting a successful HTTPS exchange using HTTP/1.1.

The trace records hostname resolution, connection establishment, HTTP/1.1 negotiation, the request, the `200 OK` response, and the returned JSON body.

**Redactions:** The local username, request-origin IP, and trace ID have been removed. The file's opening note explains why `Content-Length` retains the value from the original response.
