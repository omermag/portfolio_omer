# Lead Generation System

An end-to-end automated B2B lead generation pipeline. Discovers businesses via Google Maps, scrapes and enriches them with AI, deduplicates, scores relevance, and exports to Google Sheets.

Built on a **3-layer architecture** that separates human intent (directives) from AI orchestration and deterministic execution, ensuring reliability at scale.

## Architecture

```
                    +-----------------------+
                    |   Layer 1: Directives |
                    |   (SOPs in Markdown)  |
                    +-----------+-----------+
                                |
                    +-----------v-----------+
                    | Layer 2: Orchestration|
                    |  (AI decision-making) |
                    +-----------+-----------+
                                |
         +----------+-----------+----------+-----------+
         |          |           |          |           |
    +----v---+ +----v----+ +---v----+ +---v----+ +----v----+
    | Apify  | | Scraper | | LLM    | | Search | | Google  |
    | (Maps) | | (httpx) | | (GPT)  | | (DDG)  | | Sheets  |
    +--------+ +---------+ +--------+ +--------+ +---------+
```

### Pipeline Flow

1. **Discovery** -- Apify Google Maps crawler fetches businesses by category + location
2. **Filter** -- Removes entries without websites
3. **Scrape** -- Async multi-page scraping (homepage + up to 5 internal pages)
4. **L1 Enrichment** -- Regex/heuristic extraction of emails, phones, social links
5. **L2 Enrichment** -- GPT-4o-mini generates Hebrew business summary + outreach hooks
6. **Fallback Search** -- DuckDuckGo Hebrew search for missing emails
7. **Deduplication** -- EnrichmentGuard checks against existing sheet data
8. **Relevance Check** -- Batch LLM outlier detection removes semantic mismatches
9. **Export** -- Appends enriched leads to Google Sheet

## Project Structure

```
Lead_Generation_Portfolio/
|-- directives/                        # Layer 1: What to do
|   |-- generate_israeli_leads_v2.md   # Main pipeline SOP
|   +-- score_lead_relevance.md        # Scoring rules
|
|-- execution/                         # Layer 3: Deterministic scripts
|   |-- orchestrate_pipeline.py        # Main pipeline engine (async)
|   |-- apify_wrapper.py               # Apify Google Maps integration
|   |-- scraper_service.py             # Multi-page website scraper
|   |-- scraper_utils.py               # Scraping helpers
|   |-- enrichment_service.py          # GPT-4o-mini enrichment
|   |-- enrichment_guard.py            # Deduplication guard
|   |-- relevance_service.py           # Batch outlier detection
|   |-- search_service.py              # DuckDuckGo fallback
|   |-- category_service.py            # Category whitelist/blacklist
|   |-- drive_adapter.py               # Google Sheets read/write
|   +-- utils.py                       # Lead ID, email validation, etc.
|
|-- examples/                          # Entry points
|   |-- run_automotive_leads.py        # 700 leads, 15 categories, 2 cities
|   |-- run_luxury_car_service.py      # 300 leads, single city
|   |-- run_test_cafes.py              # Quick test (LLM disabled)
|   |-- score_leads.py                 # Post-run relevance scoring
|   +-- rescore_medium_leads.py        # Re-evaluate Medium leads
|
|-- example.me/
|   +-- walkthrough.md                 # Deep architecture walkthrough
|
|-- .env.example                       # API key template
|-- .gitignore
+-- requirements.txt
```

## Setup

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

### 2. Configure environment

Copy `.env.example` to `.env` and fill in your keys:

```bash
cp .env.example .env
```

| Variable | Source |
|----------|--------|
| `APIFY_API_TOKEN` | [apify.com](https://apify.com) -- Google Maps crawler |
| `OPENAI_API_KEY` | [platform.openai.com](https://platform.openai.com) -- GPT-4o-mini |
| `GOOGLE_DRIVE_FOLDER_ID` | Target Google Drive folder for output sheets |

### 3. Google Sheets authentication

Place a Google Cloud service account JSON as `credentials.json` in the project root. The service account needs:
- Google Sheets API enabled
- Google Drive API enabled
- Editor access to the target Drive folder

## Quick Start

### Generate leads

```bash
# Quick 20-lead test (LLM disabled for speed)
python examples/run_test_cafes.py
```

### Full production run

```bash
# 700 leads across Tel Aviv & Haifa
python examples/run_automotive_leads.py
```

### Score leads after generation

```bash
python examples/score_leads.py
```

### Custom run

Create a new runner script:

```python
import sys, os, asyncio
sys.stdout.reconfigure(encoding='utf-8')
sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..', 'execution'))
from orchestrate_pipeline import run_pipeline

CATEGORIES = ["your_category_here"]
LOCATIONS = ["your_city_here"]

if __name__ == "__main__":
    if os.name == 'nt':
        asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
    asyncio.run(run_pipeline(CATEGORIES, LOCATIONS, max_leads=50))
```

## Output

Each lead is exported to Google Sheets with:

| Column | Description |
|--------|-------------|
| `lead_id` | Unique MD5 hash |
| `business_name` | Company name |
| `city` | Location |
| `category` | Business category |
| `website` | URL |
| `email` | Primary email (info@, contact@ prioritized) |
| `secondary_email` | Additional email |
| `phone` | Phone number |
| `business_summary` | 1-2 paragraph Hebrew summary |
| `hooks` | 2-3 outreach hooks (Hebrew) |
| `instagram`, `facebook`, `linkedin`, `x` | Social links |
| `additional_contact_form` | Contact form URL |
| `founder` | Founder/owner name |
| `lead_status` | success / failed / skipped |

## Performance

- ~20-30 leads per minute (network dependent)
- ~$0.01-0.03 per fully enriched lead (Apify + GPT-4o-mini)
- Up to 10 concurrent website scrapes
- Batch LLM processing for cost efficiency

## How Directives Work

Directives (`directives/`) are Markdown SOPs that define *what* to do. The AI orchestration layer reads them and routes to the appropriate execution scripts. This separation means:

- **Execution scripts** are deterministic and testable
- **Directives** capture business logic in natural language
- **The orchestrator** handles errors, retries, and edge cases

See [example.me/walkthrough.md](example.me/walkthrough.md) for a deep dive.
