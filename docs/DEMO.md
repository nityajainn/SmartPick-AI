# Five-minute SmartPick-AI demonstration

Complete [setup](SETUP.md) first. Prices and specifications are catalogue snapshot values;
they are not live offers. This guide describes the current Search and Ask / Compare tabs.

1. **Introduce the project.** Explain evidence-grounded product search, with smartphones as the
   first validated category. A product answer should be traceable to a stored record.
2. **Search.** In Search, enter `Samsung Galaxy`, choose hybrid retrieval, expand Strict filters,
   and set maximum price INR 30,000, minimum RAM 8 GB, minimum storage 128 GB, and include Samsung.
   Inspect result IDs, source links, ratings, and scores. Python applies filters before ranking.
3. **Show a boundary.** Try an impossible budget to demonstrate an empty result, then reset it.
   Explain that unavailable specifications are not invented.
4. **Optionally ask or compare.** With storage and a real provider configured, use Ask / Compare
   for `Compare Samsung and OnePlus phones under INR 30000 by lowest price.`
   Inspect extracted constraints, citations, tool history, and verification. A provider request may
   incur cost and its response is not guaranteed to match a previous example.
5. **Explain the evidence.** Open [results](RESULTS.md): 20 retrieval cases and 16 scripted workflow
   scenarios. Show the semantic-recall/hybrid-ranking trade-off and the lack of a held-out set.

Without a provider, demonstrate Search and the scripted evaluator. Explicitly identify scripted
results as mock-model decisions. The current interface has no shortlist or standalone comparison
table; natural-language comparisons use the configured agent workflow.

## Search through the API

Against a ready local API:

```powershell
$request = @{
  query = 'Samsung Galaxy'
  mode = 'hybrid'
  alpha = 0.25
  limit = 5
  constraints = @{
    included_brands = @('Samsung')
    max_price_inr = 30000
    min_ram_gb = 8
    min_storage_gb = 128
  }
} | ConvertTo-Json -Depth 4
Invoke-RestMethod http://127.0.0.1:8000/search -Method Post -ContentType 'application/json' -Body $request
```

Inspect the returned records rather than promising a hard-coded result. Use a returned product ID,
URL-encoded with `[uri]::EscapeDataString(...)`, with `GET /products/{product_id}` to check stored
evidence when PostgreSQL is ready.

Invalid requests return 422, unknown product IDs 404, and unavailable components 503.
Treat setup/provider failures as failures; do not present them as verified answers.
See [API contracts](API_AND_UI.md). No new live demonstration was recorded for Phase 8.
