# Portfolio Record for Article 3

**Article title:** How an HTTP Request Travels—Follow One Exchange from Client to Server and Back

**Portfolio role:** Explain

**Article type:** Conceptual technical explainer

**Primary technology / tools:** HTTP • DNS • TLS • curl • httpbin

**Audience:** Beginner developers and technical learners who can make basic HTTP/API requests but need a clearer mental model of the surrounding mechanisms

**URL slug:** `how-an-http-request-travels`

## What this demonstrates

Explain one concrete HTTPS exchange while keeping the HTTP message distinct from the mechanisms that locate, carry, and protect it.

## Core workflow

Follow one concrete HTTPS exchange end to end and distinguish the responsibilities of URL, DNS, transport, TLS, and HTTP.

## Hands-on / evidence basis

- Successful `curl.exe -v --http1.1` exchange
- Hostname resolution
- TCP connection
- HTTP/1.1 selection
- Request sent
- 200 response
- JSON body
- Connection left intact

## Sources

- RFC 9110, RFC 9112, RFC 9114, RFC 3986, RFC 1034, RFC 1035, RFC 9846
- curl documentation
- httpbin

## Article

Read the full article here: [Article 3: How an HTTP Request Travels—Follow One Exchange from Client to Server and Back](Article-03.md)

## Assets

Publication-ready screenshots, diagrams, and supporting output files for this article are present in the `assets/` folder.
