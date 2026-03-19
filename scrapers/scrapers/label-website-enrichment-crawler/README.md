#  Website Enrichment Crawler

A private-use, project-based crawler built for discovering, validating, and enriching official music label websites.

This project is designed as a structured enrichment layer for label research workflows. It combines lightweight HTTP-based parsing with browser-based crawling when needed, in order to collect high-value public data while keeping the workflow efficient and scalable.

## What It Does

The crawler receives a list of labels and official website URLs, then extracts structured website intelligence for downstream research and automation flows.

It is built to identify and collect:
- page availability and accessibility
- main website text
- validated business emails
- official social and platform links
- contact and about pages
- submission/demo pages
- native submission forms
- third-party submission platforms such as SubmitHub, Groover, Trackstack, and LabelRadar

## Core Approach

This crawler uses an HTTP-first strategy:

1. try lightweight HTML fetching first
2. statically parse the page for emails, links, socials, and submission signals
3. skip browser usage when enough data is already available
4. escalate to Puppeteer only when deeper rendering or interaction is needed
5. enrich results through follow-up scans on relevant internal pages

This makes the workflow more cost-efficient while still supporting dynamic websites when necessary.

## Main Use Cases

- official website validation for music labels
- contact and submission path discovery
- social/profile extraction
- enrichment pipelines for label databases
- structured research workflows
- feeding downstream automation and review systems

## Extracted Data

Typical output may include:
- label ID and label name
- official website URL
- page alive status
- main text
- business email addresses
- social links
- contact page
- about page
- submission page
- submission type
- submission method
- submission details

## Submission Detection

The crawler supports detection of both:
- native website forms
- third-party submission platforms

Supported platform detection includes:
- SubmitHub
- Groover
- Trackstack
- LabelRadar
- Repost Network
- ToneDen

## Performance Design

This project includes workflow-level efficiency logic such as:
- HTTP-first crawling
- selective browser escalation
- proxy mode control
- domain concurrency throttling
- blocked tracking/analytics requests
- response quality checks
- runtime and performance metrics

## Stack

- JavaScript / Node.js
- Apify
- Crawlee / PuppeteerCrawler
- Cheerio
- got-scraping
- puppeteer-extra with stealth plugin

## Notes

- built for private/internal use and project-based automation workflows
- focused on practical enrichment outputs, not generic consumer scraping
- public repository versions may be simplified or cleaned compared to internal working versions
- intended as part of larger research, enrichment, and orchestration systems
