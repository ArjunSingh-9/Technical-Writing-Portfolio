# Portfolio Record for Article 4

**Article title:** Why Did My API Request Fail? A Diagnostic Workflow for 400, 401, 403, and 404 Responses

**Portfolio role:** Troubleshoot

**Article type:** Practical diagnostic guide

**Primary technology / tools:** ReqRes API • Postman

**Audience:** Beginner developers and technical learners who can make basic API requests but need a practical method for diagnosing common 4xx responses

**URL slug:** `why-did-my-api-request-fail`

## What this demonstrates

Teach evidence-based API troubleshooting through controlled comparison rather than guesswork.

## Core workflow

Known-good request → one deliberate change → observed failure → request/response evidence → working diagnosis → targeted correction → verified result

## Hands-on / evidence basis

- 400: missing `password`
- 401: missing `x-api-key`
- 403: read-only public key used on a write
- 404: changed resource ID;
- Each error corrected and re-verified

## Sources

- ReqRes API documentation
- API Testing with Postman Using ReqRes
- Hands-on Postman experiments

## Article

Read the full article here: [Article 4: Why Did My API Request Fail? A Diagnostic Workflow for 400, 401, 403, and 404 Responses](Article-04.md)

## Assets

Publication-ready screenshots, diagrams, and supporting output files for this article are present in the `assets/` folder.

**Note:** The examples reflect the behavior observed during the verification runs; API behavior and documentation can change over time.
