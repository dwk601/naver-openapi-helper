# Naver Open API Helper

A Claude skill for generating clean, production-ready Python `requests` code for Naver Open APIs.

## APIs Supported

- **News Search** (`/v1/search/news`)
- **Blog Search** (`/v1/search/blog`)
- **Web Document Search** (`/v1/search/webkr`)
- **Image Search** (`/v1/search/image`)
- **Book Search** (`/v1/search/book`)
- **Datalab Search Trends** (`/v1/datalab/search`)
- **Shopping Search** (`/v1/search/shop`)
- And other Naver Open APIs on `openapi.naver.com`

## What It Does

When you ask for Naver API integration, this skill:

1. **Browses live docs** using Playwright to extract the exact endpoint, parameters, and response format from `developers.naver.com`
2. **Generates Python code** using the `requests` library with:
   - Proper authentication (`X-Naver-Client-Id`, `X-Naver-Client-Secret`)
   - Environment variable credential loading
   - Error handling for 400/403/500 status codes
   - Pretty-printed JSON output
   - Optional `argparse` CLI interface

## Installation

Copy the `naver-openapi-helper` directory into your skills directory (e.g., `~/.agents/skills/` or `~/.claude/skills/`).

## Example Usage

```
Generate Python code to search Naver News for 'artificial intelligence' and return top 20 results sorted by date.
```

```
Write a Python script that fetches Naver Datalab search trends for 'Python' vs 'JavaScript' from Jan 2024 to Dec 2024, monthly granularity.
```

## Requirements

- `npx @playwright/cli` (for live doc browsing)
- Python `requests` library (for generated code)

## License

MIT
