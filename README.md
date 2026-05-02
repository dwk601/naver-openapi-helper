# Naver Open API Helper

[![skills.sh](https://skills.sh/b/dwk601/naver-openapi-helper)](https://skills.sh/dwk601/naver-openapi-helper)

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

### Via skills.sh (Recommended)

```bash
npx skills add dwk601/naver-openapi-helper
```

### Manual

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

## API Setup

1. Register an application at [Naver Developer Center](https://developers.naver.com/apps/#/register)
2. Enable the APIs you need under **API Settings**:
   - **Search APIs** → enable **"검색"**
   - **Datalab APIs** → enable **"데이터랩 (검색어트렌드)"**
   - **Shopping API** → enable **"쇼핑"**
3. Set environment variables:
   ```bash
   export NAVER_CLIENT_ID="your_client_id"
   export NAVER_CLIENT_SECRET="your_client_secret"
   ```

## License

MIT
