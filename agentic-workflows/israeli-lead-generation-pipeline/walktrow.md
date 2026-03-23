# System Walkthrough: Intelligent Lead Generation

A deep dive into the Lead Generation System -- the problem, the architecture, and how it dynamically captures leads.

## The Problem

Finding and qualifying B2B leads manually is tedious, prone to human error, and lacks scale. Traditional scrapers only pull raw data without evaluating whether a business fits specific investment or brandability criteria.

## The Solution

An AI-powered orchestration engine that not only gathers data but deeply analyzes a brand's footprint. The system follows a **3-layer architecture**:

| Layer | Role | Location |
|-------|------|----------|
| **Directives** | SOPs defining goals, inputs, outputs, edge cases | `directives/` |
| **Orchestration** | AI reads directives, routes to scripts, handles errors | The LLM agent |
| **Execution** | Deterministic Python scripts doing the actual work | `execution/` |

This separation ensures that probabilistic AI decisions don't compound errors -- the heavy lifting happens in deterministic, testable code.

## Core Workflow

1. **Targeting & Querying**: Takes predefined categories (e.g., "Mercedes Garage") and locations (e.g., "Tel Aviv").
2. **Discovery**: Pulls raw entity data via Apify Google Maps crawler (`execution/apify_wrapper.py`).
3. **Filtering**: Discards businesses without websites, malformed entries, or irrelevant entities.
4. **Multi-Page Scraping**: Fetches homepage + up to 5 internal pages (contact, about, team) via `execution/scraper_service.py`.
5. **L1 Enrichment**: Regex/heuristic extraction of emails, phones, social links.
6. **L2 Enrichment (LLM)**: GPT-4o-mini generates Hebrew business summaries and outreach hooks (`execution/enrichment_service.py`).
7. **Fallback Search**: DuckDuckGo Hebrew search for leads still missing emails (`execution/search_service.py`).
8. **Deduplication**: `execution/enrichment_guard.py` checks lead_id, website domain, and name+city pairs against existing Google Sheet data.
9. **Relevance Scoring**: Batch outlier detection removes semantic mismatches (`execution/relevance_service.py`).
10. **Export**: Appends enriched leads to Google Sheet (`execution/drive_adapter.py`).

## Architecture Detail

- **`execution/orchestrate_pipeline.py`**: The pipeline engine. Fully async, maps I/O flows between all services.
- **`execution/enrichment_service.py`**: Crafts specialized prompts to evaluate leads. Outputs Hebrew summaries and concrete outreach hooks.
- **`execution/scraper_service.py`**: Concurrent multi-page scraper with smart retry logic and proxy support.
- **`execution/drive_adapter.py`**: Google Sheets adapter -- creates sheets, appends rows, fetches existing lead identifiers for dedup.
- **`execution/utils.py`**: Shared utilities -- lead ID generation (MD5), email validation, website normalization, aggressive system-domain filtering.

## Example Execution

```bash
# From the project root:
python examples/run_automotive_leads.py
```

Console output traces each async step:
```
[Discovery] Queuing: "Mercedes Garage" in "Tel Aviv"
[Discovery] Apify returned 142 raw results
[Filter] 98 businesses with valid websites
[Scrape] Fetching 98 websites (10 concurrent)...
[L1] Extracted contacts from 85 pages
[L2] LLM enrichment: 85 leads processed
[Fallback] DuckDuckGo search for 13 missing emails
[Save] Pushed 92 leads to Google Sheets
```

## Key Design Decisions

**Why Apify for discovery?** Google Maps has the most comprehensive local business data for Israel. Apify's crawler handles pagination, captchas, and rate limiting.

**Why GPT-4o-mini for enrichment?** Cost-effective ($0.01-0.03/lead) while producing high-quality Hebrew summaries. Full GPT-4o would be 10x the cost with minimal quality improvement for this use case.

**Why Google Sheets as output?** Serves as a "database for humans" -- clients can view, filter, and annotate leads without any technical setup. The `drive_adapter.py` handles all API complexity.

**Why deduplication before export?** Running multiple batches against the same sheet is a common workflow. The EnrichmentGuard prevents duplicate entries across runs.

## Future Possibilities

- Integration with CRMs (Salesforce, HubSpot) for direct emailing campaigns
- Vision models (VLM) for scoring social media aesthetic quality
- Automated outreach via email/WhatsApp based on lead score
