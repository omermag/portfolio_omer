# Generate Israeli Leads V2

**Goal**: Generate high-quality B2B leads for Israeli businesses using Google Maps data, with advanced enrichment and content understanding.

## Inputs
- `categories`: One or more Hebrew business categories (comma-separated). Example: `"משרד עורכי דין,משרד רואי חשבון"`
- `locations`: One or more Israeli locations (comma-separated). Example: `"תל אביב,ירושלים,חיפה"`
- `max_leads`: Total number of verified leads to generate across all categories/locations
- `sheet_name`: (Optional) Custom Google Sheet name. If not provided, auto-generated from inputs.

## Outputs
- New Google Sheet created in folder: `15BF9AUit6FKAnImOGKIQJ3a7-EhPb6-v`
- Columns:
  - `lead_id` - Unique identifier (MD5 hash)
  - `business_name` - Company name
  - `city` - Location
  - `category` - Business category
  - `website` - Website URL
  - `email` - Primary email (prioritized: info@, contact@)
  - `secondary_email` - Additional email found
  - `phone` - Phone number
  - `business_summary` - 1-2 paragraph Hebrew summary (professional, non-marketing)
  - `hooks` - 2-3 concrete outreach hooks (Hebrew)
  - `instagram` - Instagram URL
  - `facebook` - Facebook URL
  - `linkedin` - LinkedIn URL
  - `x` - X/Twitter URL
  - `additional_contact_form` - Contact form URL if found
  - `founder` - Founder/Owner name if found
  - `lead_status` - Processing status (success/failed/skipped)

## Tools & Scripts
- `execution/orchestrate_pipeline.py` - Main orchestrator

## Instructions

### Recommended Usage (Windows - Hebrew Support)
Due to Windows CMD encoding issues with Hebrew characters, use a Python runner script:

```python
# run_test.py - Create this file with your parameters
# -*- coding: utf-8 -*-
import sys
import os
sys.stdout.reconfigure(encoding='utf-8')
sys.path.insert(0, os.path.join(os.path.dirname(__file__), 'execution'))
import asyncio
from orchestrate_pipeline import run_pipeline

CATEGORIES = ["משרד נדלן"]  # Hebrew categories work correctly here
LOCATIONS = ["ירושלים"]      # Hebrew locations work correctly here  
MAX_LEADS = 20
SHEET_URL = "https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID/edit"  # Optional

if __name__ == "__main__":
    if os.name == 'nt':
        asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
    asyncio.run(run_pipeline(CATEGORIES, LOCATIONS, MAX_LEADS, sheet_url=SHEET_URL))
```

Then run:
```bash
python run_pipeline.py
# Note: Edit the CATEGORIES and LOCATIONS variables inside the script first.
```

### Alternative: Command Line (English only)
For English-only searches, you can use the CLI directly:
```bash
python execution/orchestrate_pipeline.py \
  --categories "lawyer office" \
  --locations "Tel Aviv" \
  --max-leads 20 \
  --sheet-name "Israeli Leads Jan 2026"
```

### Batch Mode (Many Categories/Locations)
The orchestrator automatically splits `max_leads` across all combinations:
- 2 categories × 3 locations = 6 batches
- 60 total leads → ~10 leads per batch

### Processing Flow
1. **Discovery** - Fetch businesses via Apify (Google Maps crawler)
2. **Filter** - Remove businesses without websites
3. **Scrape** - Fetch homepage + up to 5 internal pages (contact, about, team)
4. **Layer 1 Enrichment** - Extract contact info (regex/heuristic)
5. **Layer 2 Enrichment** - LLM generates Hebrew summary + hooks (GPT-4o-mini)
6. **Fallback Search** - DuckDuckGo search for missing emails
7. **Save** - Append to Google Sheet

## Edge Cases
- **No Website**: Lead is immediately skipped
- **Scraping Failed** (404/timeout): Lead is skipped
- **No Content**: If website has minimal content, Layer 2 is skipped
- **No Email After All Steps**: Lead is marked `skipped` but may still be saved if other data is valuable
- **Duplicate `lead_id`**: Checked via hash, skipped if already in batch
- **Apify Credits**: Pipeline fails gracefully if Apify quota exceeded

## Performance
- **Parallel Scraping**: Up to 10 concurrent website scrapes
- **Parallel Enrichment**: Batch LLM calls where possible
- **Cost**: ~$0.01-0.03 per fully enriched lead (depends on website size)
- **Speed**: ~20-30 leads per minute (network dependent)

## Monitoring
Console output shows:
- `[Discovery]` - Apify results fetched
- `[Filter]` - Websites validated
- `[Scrape]` - Pages downloaded
- `[L1]` - Contact extraction
- `[L2]` - LLM summary generation
- `[Fallback]` - DuckDuckGo search
- `[Save]` - Sheet update

## Troubleshooting
- **"Apify error"**: Check `.env` for `APIFY_API_TOKEN`
- **"OpenAI error"**: Check `.env` for `OPENAI_API_KEY`
- **"Google Sheets error"**: Ensure `credentials.json` and `token.json` are valid
- **Low yield**: Increase `max_leads` or broaden search terms
