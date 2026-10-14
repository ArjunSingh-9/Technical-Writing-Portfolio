# Supporting Assets — API Troubleshooting Guide

These assets document the controlled failures and recovery checks used in the 400, 401, 403, and 404 investigations.

## Screenshots

- [ReqRes_Experiment2B_Response.png](ReqRes_Experiment2B_Response.png) — Registration request missing the required password, returning `400 Bad Request`.
- [ReqRes_Experiment3A_Response.png](ReqRes_Experiment3A_Response.png) — Request missing the `x-api-key` header, returning `401 Unauthorized`.
- [ReqRes_Experiment4A_Response.png](ReqRes_Experiment4A_Response.png) — Write request using a read-only public key, returning `403 Forbidden`.
- [ReqRes_Experiment5_Response.png](ReqRes_Experiment5_Response.png) — Known-good request for user ID `2`, returning `200 OK`.
- [ReqRes_Experiment5A_Response.png](ReqRes_Experiment5A_Response.png) — Request for user ID `999999`, returning `404 Not Found`.

## Response outputs

- [ReqRes_Experiment3A_Output.json](ReqRes_Experiment3A_Output.json) — Captured 401 response with its diagnostic hints and documentation links.
- [ReqRes_Experiment4A_Output_Excerpt.json](ReqRes_Experiment4A_Output_Excerpt.json) — Selected 403 response fields relevant to the read-only permission diagnosis.
- [ReqRes_Experiment4B_Output_Sanitized.json](ReqRes_Experiment4B_Output_Sanitized.json) — Selected fields from the successful product-creation response after correcting the permissions.
