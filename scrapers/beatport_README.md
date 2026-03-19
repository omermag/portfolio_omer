# Beatport Labels Scraper

A private-use, project-based scraper built to extract label names from Beatport chart and release-related pages.

This actor is designed for structured label discovery workflows, where Beatport pages are used as a source for collecting music label data for research, enrichment, and downstream automation systems.

## What It Does

The scraper receives a list of Beatport URLs as input, visits each page, and extracts label names from supported page structures.

It supports multiple page layouts, including:
- chart-style track tables
- release table views
- fallback extraction from label links found on the page

Duplicate labels are filtered across the crawl, and only unique label names are stored in the output dataset.

## Main Use Cases

- discovering labels from Beatport charts
- extracting label names from release pages
- collecting structured inputs for enrichment pipelines
- feeding music research and automation workflows
- supporting downstream validation and label discovery systems

## How It Works

1. receives a list of Beatport page URLs
2. loads each page using PuppeteerCrawler
3. detects the matching page structure
4. extracts label names from supported selectors
5. removes duplicate labels across the crawl
6. stores unique results in the dataset

## Output

Each result contains a structured label entry:

```json
{
  "label": "Example Label"
}
