# Website Social / Contact Extractor

A private-use, project-based crawler built to scan official websites and extract structured public-facing data for research, validation, and automation workflows.

This actor is designed to discover key website assets such as contact pages, about pages, submission links, forms, emails, and social profiles, while also capturing a short block of main-site text for downstream review and enrichment.

## What It Does

The crawler receives a list of website entries, visits each official site, and extracts relevant structured fields from both the main page and selected follow-up pages.

It is built to identify and collect:
- main site text
- contact pages
- about pages
- submission/demo pages
- native contact or submission forms
- mailto links
- public email addresses
- social and platform links
- external submission tools such as Trackstack

## Main Use Cases

- official website validation
- social/contact discovery
- demo or submission path detection
- website-based enrichment workflows
- source preparation for downstream research systems
- feeding structured outputs into tables, databases, or automation pipelines

## How It Works

1. receives a list of website entries
2. visits the main website page
3. extracts core text content from the page
4. scans links for contact, about, and submission-related destinations
5. extracts emails, mailto links, and social/platform URLs
6. follows selected internal pages for deeper scanning
7. detects native forms and classifies submission paths
8. stores structured results for each record

## Extracted Fields

Typical structured output may include:
- official website
- main text
- contact page
- about page
- submission page
- contact form
- submission type
- email addresses
- mailto links
- Instagram
- Facebook
- YouTube
- Beatport
- Spotify
- SoundCloud
- TikTok
- X / Twitter
- Linktree
- Bandcamp
- Trackstack

## Stack

- JavaScript / Node.js
- Apify
- Crawlee / PuppeteerCrawler
- Cheerio
- dataset-based structured output

## Notes

- built for private/internal usage and project-based workflows
- focused on practical extraction and structured outputs
- public repository versions may be cleaned or simplified compared to internal working versions
- intended as a component inside larger automation and enrichment systems
