---
layout: default
title: "How an HTTP Request Travels: Follow One Exchange from Client to Server and Back"
---

# How an HTTP Request Travels: Follow One Exchange from Client to Server and Back

## Introduction

HTTP is a stateless, application-level request/response protocol. But the apparently simple action of making an HTTPS request involves more than the core HTTP message itself. Several other mechanisms work alongside HTTP to help locate the destination, establish communication, and protect the exchange.

Concepts such as DNS, TLS, HTTPS, and HTTP are often encountered separately, making it difficult to see how they interact.

This is the knowledge gap that this article addresses. We’ll follow one concrete HTTPS exchange to understand how these different mechanisms work together and how they relate to the HTTP message itself.

After reading this article, you’ll understand:

- What happens before the HTTP message is exchanged.
- What the HTTP message itself contains.
- What happens at the server.
- How the response travels back to the client.
- Most importantly, what belongs to HTTP and what does not.

## The Request We’ll Follow

We’ll follow a request to this concrete HTTPS URL throughout the article:

```
https://httpbin.org/get?item=42
```

Our example makes a simple HTTPS GET request to the `/get` endpoint with a query parameter named `item` whose value is `42`.

We used the following curl command to send the request and inspect the exchange:

```powershell
curl.exe -v --http1.1 "https://httpbin.org/get?item=42"
```

The URL can be broken into these parts, which we’ll explain later:

```
https://
httpbin.org
/get
?item=42
```

We sent this request during live testing and received a readable response. The curl trace gave us a real record of the exchange, as shown by this small excerpt:

```
* Host httpbin.org:443 was resolved.
* Connected to httpbin.org (...) port 443
> GET /get?item=42 HTTP/1.1
...
< HTTP/1.1 200 OK
```

[View the complete sanitized curl trace](assets/successful-http1_1-curl-trace.txt)

After defining the starting point, let’s begin at the destination.

## Start with the Destination

In the example HTTPS URL, `https://httpbin.org/get?item=42`, the `httpbin.org` portion is the **hostname**. Though human-friendly, a hostname is **not** the network address that the client needs to communicate with the destination.

DNS is the system used to obtain information associated with domain names, including network-address records. In our exchange, curl obtained address information for the hostname through DNS resolution.

For the example HTTPS URL, the sequence begins with:

```
  https://httpbin.org/get?item=42
             ↓
    httpbin.org (hostname)
             ↓
      DNS resolution
             ↓
    Address information
```

The curl trace of the example URL also shows:

```
...
* Host httpbin.org:443 was resolved.
* IPv6: (none)
* IPv4: 44.194.227.11, 32.194.63.46, ...
* Trying 44.194.227.11:443...
* Connected to httpbin.org (44.194.227.11) port 443
...
```

One hostname can resolve to multiple addresses. The output shows multiple address options, followed by an attempt to connect to one of them.

**DNS helps the client find address information. It does not itself establish the HTTP connection.**

Once the client knows the connection destination, it next needs to establish the communication path.

## Prepare the Communication Channel

Once the client has address information, it needs an underlying communication mechanism through which the HTTP exchange can take place. HTTP is **not** itself responsible for providing the underlying transport path.

For this example, the underlying mechanism means establishing a TCP connection to the destination’s port 443.

The trace shows curl attempting the connection and then establishing it:

```
* Trying 44.194.227.11:443...
* Connected to httpbin.org (44.194.227.11) port 443
```

The HTTP request appears afterward:

```
> GET /get?item=42 HTTP/1.1
```

**The connection provides the communication path; the HTTP request is the application-level message that travels through it.**

**Note:** Port 443, the conventional port associated with HTTPS, is part of the communication endpoint information; it is not part of the HTTP request line.

**Important:** For this experiment, our HTTP/1.1 exchange uses TCP-based transport. Other HTTP deployments can use different underlying transport mechanisms; for example, HTTP/3 uses QUIC.

The client now has an established communication path for the HTTP exchange. The communication still needs protection because this is an HTTPS exchange.

## Secure the Connection

Because this is an HTTPS exchange, the communication carrying the HTTP messages is protected using TLS.

HTTP itself defines the application-level request/response exchange; it does not provide the security properties that TLS provides. In an HTTPS exchange, TLS protects the communication carrying those HTTP messages.

| Element / mechanism | Main responsibility |
| --- | --- |
| HTTP | Defines the application-level request/response exchange |
| TLS | Protects the communication |
| HTTPS | HTTP communicated over TLS |

TLS provides:

- **Confidentiality:** Others should not be able to read the protected communication.
- **Integrity:** Data should not be silently modified without detection.
- **Authentication:** TLS can authenticate the server’s identity; in typical HTTPS deployments, the client uses the server’s certificate as part of that process.

Before the HTTP messages in this HTTPS exchange are carried as protected application data, the client and server establish the TLS security context.

The successful curl run demonstrated a completed HTTPS exchange, but the verbose output did not expose the TLS handshake details.

With the communication protected, the next section will look at the actual HTTP request.

## What the HTTP Request Actually Contains

We now move from the surrounding mechanisms to the actual HTTP message itself.

The HTTP request is an application-level message containing information that tells the server what the client wants and provides relevant request metadata.

In the previous example’s curl trace, the actual HTTP/1.1 request was:

```
GET /get?item=42 HTTP/1.1
Host: httpbin.org
User-Agent: curl/8.9.1
Accept: */*
```

**The URL is not the HTTP request.** The URL gives the client the destination and request information it needs; the HTTP request is the application-level message the client sends.

### HTTP request anatomy

In the HTTP/1.1 representation used by our example, the request message has three main parts:

- **Request line:** Contains the method, request target, and HTTP version.
- **Headers:** Provide additional metadata about the request.
- **Body (optional):** Contains data sent to the server when the request includes a message body.

For our example:

| HTTP request part | Example |
| --- | --- |
| Request line | `GET /get?item=42 HTTP/1.1` |
| Headers | `Host: httpbin.org`, `User-Agent: curl/8.9.1`, `Accept: */*` |
| Body | Not present |

### Request line

The request line contains three components:

```
GET /get?item=42 HTTP/1.1
```

- **Method:** `GET` indicates that the client wants to retrieve a representation of the target resource.
- **Request target:** `/get?item=42` identifies the target for this request.
- **HTTP version:** `HTTP/1.1` identifies the HTTP version used by this message.

### Headers

The request contains these headers:

- **`Host: httpbin.org`** identifies the host associated with this HTTP request.
- **`User-Agent: curl/8.9.1`** identifies the client software making the request.
- **`Accept: */*`** indicates that the client is willing to accept any response media type.

An HTTP request can include a message body, but a body is not present in every request. *Our GET request does not include one.*

The earlier stages established the destination and protected the communication. The HTTP request is the application-level message travelling through that communication.

After displaying the HTTP message, curl reported that the request was completely sent:

```
* Request completely sent off
```

Next, we look at how the server processes this message.

---

## The Request Reaches the Server

The curl trace also reports that the HTTP request was completely sent.

```
* Request completely sent off
```

After the request reaches the server, the server receives and interprets the HTTP message to determine how to handle the request. The server-side application then processes the request to produce a response.

Conceptually, the server-side process is:

```
Client
  │
  │ HTTP request
  ▼
Server
  │
  │ interprets request
  ▼
Server-side processing
  │
  │ produces response
  ▼
HTTP response
```

The server receives the HTTP request examined in the previous section, including its request target and headers. The returned JSON gives us evidence that the service processed information from that request:

```json
{
  "args": {
    "item": "42"
  }
}
```

This returned JSON shows that the service reflected the `item=42` query parameter in the response.

Now that the server has handled the request, the next section will look at the HTTP response being sent back.

## The Server Creates an HTTP Response

After processing the request, the server produces and sends an HTTP response back to the client.

An HTTP response is the server’s application-level message back to the client, containing a status and additional response information, with an optional body.

The successful curl trace showed the response start as:

```
< HTTP/1.1 200 OK
...
< Content-Type: application/json
< Content-Length: 281
...
```

The response body then contained JSON including:

```json
{
  "args": {
    "item": "42"
  },
  ...
  "url": "https://httpbin.org/get?item=42"
}
```

### HTTP response anatomy

In the HTTP/1.1 representation used by our example, the response has three main parts:

- **Status line:** Contains the HTTP version, a numeric status code, and a reason phrase.
- **Response headers:** Provide metadata about the response.
- **Response body (optional):** Contains the returned content.

### Status line

For our example, the status line is:

```
HTTP/1.1 200 OK
```

It contains three components:

- **HTTP version:** `HTTP/1.1` identifies the HTTP version used by this message.
- **Status code:** `200` indicates that the request was successfully handled.
- **Reason phrase:** `OK` is the textual reason phrase associated with the status.

### Response headers

The example’s response contains these main headers:

- **`Content-Type: application/json`** tells the client the media type of the response content.
- **`Content-Length: 281`** indicates the length of the response content in bytes.

### Response body

An HTTP response can include a message body, but a body is not present in every response.

Our example’s response contains a body including:

```json
"args": {
  "item": "42"
}
```

It also echoes request-related information:

```json
"headers": {
  "Accept": "*/*",
  "Host": "httpbin.org",
  "User-Agent": "curl/8.9.1",
  ...
},
...
"url": "https://httpbin.org/get?item=42"
```

Observe the connection between the request and its response:

> **Client requested `item=42` → Server response contains `item=42`**
> 

Conceptually, the HTTP exchange is a two-sided process:

```
Client → HTTP request → Server
Client ← HTTP response ← Server
```

The request and response have parallel structures:

| HTTP Request | HTTP Response |
| --- | --- |
| Request line | Status line |
| Headers | Headers |
| Optional body | Optional body |

**The response is not simply “data coming back”.** It is another HTTP message, generated by the server and directed back to the client.

The response now has to travel back through the communication that was established earlier, as we’ll see in the next section.

## The Response Travels Back to the Client

Once the server sends the response, it travels back to the client through the established communication. The client receives the TLS-protected communication and can then interpret the HTTP response.

In our example, curl received the HTTP response described in the previous section.

The successful trace then ends with:

```
* Connection #0 to host httpbin.org left intact
```

After displaying the response, the trace reports that the connection remained available after the exchange.

**The important concept is the symmetry of the HTTP exchange:**

```
Client → HTTP request → Server
Client ← HTTP response ← Server
```

The established communication enables the response to travel back to the client.

Conceptually, the response’s return journey is:

```
Server
   │
   │ HTTP response
   ▼
Established protected communication
   │
   ▼
Client
   │
   │ interprets response
   ▼
Response interpreted by client
```

Looking back at this entire journey, the next section will examine which parts actually belong to HTTP.

## What Belongs to HTTP—and What Doesn’t?

We have now followed the complete HTTPS exchange. Now we can distinguish which parts belong to HTTP and which support or protect the HTTP exchange.

**HTTP defines the application-level request/response exchange.** Other mechanisms help identify the destination, provide communication, and protect that communication.

The elements involved play these roles:

| Element / mechanism | Role in the journey |
| --- | --- |
| URL | Identifies the destination and resource being requested |
| DNS | Provides address information associated with the hostname |
| Transport | Provides the underlying communication mechanism |
| TLS | Protects the communication |
| HTTP | Defines the application-level request/response exchange |

**HTTPS refers to HTTP communication protected with TLS:**

```
HTTP + TLS protection = HTTPS
```

### What belongs to HTTP?

Our HTTP/1.1 exchange contained two HTTP messages: a request and a response. The request used a request line and headers, while the response used a status line, headers, and a body.

**The request and response messages are the HTTP portion of the exchange.**

### What does not belong to HTTP?

#### DNS

DNS is a supporting mechanism that can provide address information for the hostname. Hostname resolution is separate from the HTTP message.

#### Transport

Transport provides the underlying communication mechanism. Connection establishment is separate from the HTTP request message.

#### TLS

TLS protects the communication carrying the HTTP exchange. It is not another HTTP header or field. In HTTPS, the HTTP message remains an HTTP message even though the communication carrying it is protected by TLS.

#### URL

**The URL itself is not HTTP.** The URL helps the client determine where and what to request; the HTTP request is the separate application-level message used to make that request.

### The key distinction

**The supporting mechanisms locate, connect, and protect the exchange; HTTP defines the application-level request and response.**

Now that these boundaries are clear, the next section will reconstruct the entire journey in one compact mental model.

## What You Learned

We’ve followed the exchange from the initial URL to the client’s receipt of the response. Now we can view the entire exchange as one coherent journey.

The entire exchange can be summarized visually:

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     │ HTTPS URL
     ▼
  Hostname
     │
     │ DNS resolution
     ▼
Address information
     │
     │ Establish communication
     ▼
TLS-protected communication
     │
     │ HTTP request
     ▼
┌────────────┐
│   Server   │
└────┬───────┘
     │
     │ Process request
     ▼
 HTTP response
     │
     │ Travel back
     ▼
┌──────────┐
│  Client  │
└──────────┘
```

The full HTTPS exchange can be recapped as:

1. **Identify the destination:** Start with the HTTPS URL and its hostname.
2. **Resolve the hostname:** Obtain address information for the destination.
3. **Establish communication:** Set up the underlying communication mechanism.
4. **Protect the communication:** For HTTPS, TLS protects the HTTP exchange.
5. **Send the HTTP request:** The client sends an application-level HTTP message.
6. **Process the request:** The server receives and handles the request.
7. **Create the HTTP response:** The server generates an HTTP response.
8. **Return the response:** The response travels back to the client, which interprets it.

Our example started with this HTTPS URL:

```
https://httpbin.org/get?item=42
```

The exchange can then be summarized as:

```
URL
 ↓
httpbin.org
 ↓
DNS/address resolution
 ↓
communication + TLS
 ↓
GET /get?item=42 HTTP/1.1
 ↓
200 OK + JSON
```

The key mental model is simple: each component has a distinct responsibility. The URL identifies the destination and resource information. DNS can provide address information. Transport provides communication. TLS protects the communication. HTTP defines the application-level request/response exchange.

Now that the complete journey is clear, let’s look at the next steps.

## Next Steps

Build on this mental model by:

1. **Explore HTTP methods and status codes:** Learn how different methods, status codes, and request/response semantics affect an HTTP exchange.
2. **Build and inspect API requests:** Send different HTTP requests to a test API and inspect how the request and response change. This connects directly with our hands-on example without requiring a new conceptual framework.
3. **Learn to diagnose HTTP failures:** Explore common HTTP failure scenarios and use the request and response to identify what went wrong.
4. **Explore HTTP/2 and HTTP/3:** Because this article uses an HTTP/1.1 example, a natural next step is to explore how HTTP/2 and HTTP/3 change the way HTTP messages are framed and transported.

Once you can separate the destination, communication, protection, and HTTP message itself, an HTTPS exchange becomes much easier to reason about.

## Sources and Verification

### Primary technical references

- RFC 9110 — HTTP Semantics
- RFC 9112 — HTTP/1.1
- RFC 9114 — HTTP/3
- RFC 3986 — Uniform Resource Identifier (URI): Generic Syntax
- RFC 1034 — Domain Names: Concepts and Facilities
- RFC 1035 — Domain Names: Implementation and Specification
- RFC 9846 — The Transport Layer Security (TLS) Protocol Version 1.3

### Tool and service references

- curl documentation
- httpbin — HTTP Request & Response Service

### Hands-on verification

The concrete request and response examples in this article were verified with `curl.exe` in Windows PowerShell using:

```powershell
curl.exe -v --http1.1 "https://httpbin.org/get?item=42"
```

The successful trace captured hostname resolution, TCP connection establishment, HTTP/1.1 negotiation, the HTTP request, the `200 OK` response, and the JSON response body. A separate TLS 1.3 diagnostic run was not used as evidence of a successful TLS handshake because the Windows Schannel initialization failed before the HTTP request was sent.
