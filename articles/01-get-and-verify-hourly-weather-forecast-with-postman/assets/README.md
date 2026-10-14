# Supporting Assets — Weather Forecast Tutorial

These assets document the successful request, the deliberate error, and the corrected request used in the tutorial.

- [OM_Experiment5_Response.png](OM_Experiment5_Response.png) — Initial successful 12-hour forecast request, including its parameters and returned data.
- [OM_Experiment6_Response.png](OM_Experiment6_Response.png) — Deliberate `tempeture_2m` typo and resulting `400 Bad Request` response.
- [OM_Experiment7_Response.png](OM_Experiment7_Response.png) — Corrected request returning a successful response.
- [OM_Experiment7_Output.json](OM_Experiment7_Output.json) — Captured JSON response containing 12 forecast timestamps and their corresponding temperature values.

**Note:** The JSON file is a point-in-time capture from testing, not a live forecast. Later requests may return different forecast values
