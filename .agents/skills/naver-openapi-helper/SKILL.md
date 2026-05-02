---
name: naver-openapi-helper
description: Generate Python code for Naver Open APIs (News Search, Blog Search, Web Search, Datalab Trends, Image Search, Book Search, and other openapi.naver.com APIs). Use this skill whenever the user wants to call Naver APIs, search Naver News/Blog/Web/Images, fetch Datalab keyword trends, or write Python scripts that interact with Naver's Open API platform. This includes requests for code examples, API integration, parameter explanations, endpoint lookup, or troubleshooting Naver Open API calls. Even if the user just mentions 'Naver API', 'Naver news data', 'search trends', or 'openapi.naver.com', activate this skill immediately.
allowed-tools: Bash(playwright-cli:*) Bash(npx:*) Bash(npm:*) Bash(python3:*) Bash(pip:*)
---

# Naver Open API Helper

Generate clean, production-ready Python `requests` code for Naver Open APIs by browsing the official live documentation at `developers.naver.com`.

## Covered APIs

All Naver Open APIs on `https://openapi.naver.com` using simple header authentication:
- **Search APIs**: News (`/v1/search/news`), Blog (`/v1/search/blog`), Web (`/v1/search/webkr`), Image (`/v1/search/image`), Book (`/v1/search/book`), etc.
- **Datalab APIs**: Search trends (`/v1/datalab/search`), Shopping trends, etc.

**Not covered**: Naver Search Ad API (`api.searchad.naver.com`) — that requires HMAC-SHA256 signatures and a different docs site.

## Authentication

Naver Open APIs use simple header-based auth:
- `X-Naver-Client-Id`
- `X-Naver-Client-Secret`

Always load these from environment variables in generated code:
```python
import os
CLIENT_ID = os.environ.get("NAVER_CLIENT_ID")
CLIENT_SECRET = os.environ.get("NAVER_CLIENT_SECRET")
```

If the user provides explicit credentials, use them but add a comment warning:
```python
# WARNING: Hardcoding credentials is insecure. Use environment variables in production.
```

## Important: API Scope Enablement

Each Naver API family must be enabled separately in the [Naver Developer Center](https://developers.naver.com/apps/#/list):

- **Search APIs** → enable **"검색"**
- **Datalab APIs** → enable **"데이터랩 (검색어트렌드)"**
- **Shopping API** → enable **"쇼핑"**

If you get `errorCode: "024"` (Scope Status Invalid), your key is valid but the specific API is not enabled. Go to your app's API Settings and check the relevant box.

## Workflow

### Step 1: Identify the API

From the user's request, determine which API family they need:

| User mentions | API | Docs URL |
|---|---|---|
| "news", "article", "headline" | News Search | `https://developers.naver.com/docs/serviceapi/search/news/news.md` |
| "blog", "blog post" | Blog Search | `https://developers.naver.com/docs/serviceapi/search/blog/blog.md` |
| "web", "web document", "webkr" | Web Search | `https://developers.naver.com/docs/serviceapi/search/web/web.md` |
| "image", "picture", "photo" | Image Search | `https://developers.naver.com/docs/serviceapi/search/image/image.md` |
| "book", "책" | Book Search | `https://developers.naver.com/docs/serviceapi/search/book/book.md` |
| "datalab", "trend", "keyword trend", "search trend" | Datalab Search | `https://developers.naver.com/docs/serviceapi/datalab/search/search.md` |
| "shopping", "shop" | Shopping Search | `https://developers.naver.com/docs/serviceapi/search/shop/shop.md` |
| "encyc", "encyclopedia", "knowledge" | Encyclopedia | `https://developers.naver.com/docs/serviceapi/search/encyc/encyc.md` |
| "cafe", "cafegroup" | Cafe Search | `https://developers.naver.com/docs/serviceapi/search/cafearticle/cafearticle.md` |

If unsure, browse the Naver API docs index or ask the user.

### Step 2: Browse Live Docs with Playwright

Use `npx @playwright/cli` to open the docs page and extract endpoint details.

```bash
npx @playwright/cli open --browser=chromium <docs-url>
npx @playwright/cli snapshot
```

From the snapshot, extract:
1. **Request URL** (e.g., `https://openapi.naver.com/v1/search/news.json`)
2. **HTTP Method** (GET or POST)
3. **Parameters** (name, type, required, description)
4. **Request/Response examples**
5. **Error codes**

If the page content is not fully visible, scroll and take additional snapshots:
```bash
npx @playwright/cli eval "window.scrollTo(0, document.body.scrollHeight)"
npx @playwright/cli snapshot
```

If the docs URL is a general index, navigate to the specific API page first:
```bash
npx @playwright/cli goto <specific-docs-url>
npx @playwright/cli snapshot
```

### Step 3: Generate Python Code

Produce a complete, runnable Python script using the `requests` library.

#### Code Template

```python
import os
import requests
import json
from urllib.parse import quote


def main():
    # Load credentials from environment variables
    client_id = os.environ.get("NAVER_CLIENT_ID")
    client_secret = os.environ.get("NAVER_CLIENT_SECRET")

    if not client_id or not client_secret:
        raise ValueError("NAVER_CLIENT_ID and NAVER_CLIENT_SECRET must be set")

    # API endpoint
    url = "<endpoint-url>"

    # Auth headers
    headers = {
        "X-Naver-Client-Id": client_id,
        "X-Naver-Client-Secret": client_secret,
    }

    # For POST requests (Datalab), add Content-Type
    # headers["Content-Type"] = "application/json"

    # Parameters
    params = {
        "query": quote("<search-term>"),
        "display": 10,
        "start": 1,
        "sort": "sim",  # sim (accuracy) or date
    }

    # Make request
    response = requests.get(url, headers=headers, params=params)
    # For POST: response = requests.post(url, headers=headers, json=params)

    # Handle response
    if response.status_code == 200:
        data = response.json()
        print(json.dumps(data, ensure_ascii=False, indent=2))
    elif response.status_code == 400:
        print(f"Error 400 - Bad Request: {response.text}")
        print("Check your parameters (query encoding, display/start/sort values).")
    elif response.status_code == 401:
        print(f"Error 401 - Unauthorized: {response.text}")
        print("Check your Client ID and Secret. Ensure the API scope is enabled in Naver Developer Center.")
    elif response.status_code == 403:
        print(f"Error 403 - Forbidden: {response.text}")
        print("Ensure the API is enabled for your application in the Naver Developer Center.")
    elif response.status_code == 500:
        print(f"Error 500 - Internal Server Error: {response.text}")
    else:
        print(f"Error {response.status_code}: {response.text}")


if __name__ == "__main__":
    main()
```

#### Requirements

1. **Always use `requests`**, not `urllib`.
2. **Always URL-encode Korean queries**. Use `urllib.parse.quote(query, safe='')` or let `requests` handle it via the `params` dict (which encodes automatically).
3. **Always include error handling** for non-200 status codes, especially 400, 401, 403, and 500.
4. **Always load credentials from `os.environ`** unless the user explicitly provides them inline.
5. **For Datalab (POST)**: Set `Content-Type: application/json` and pass the body via `json=body_dict`.
6. **For Search APIs (GET)**: Pass query parameters via `params=`. `requests` will URL-encode them.
7. **Pretty-print JSON responses** with `ensure_ascii=False` and `indent=2`.
8. **Add comments** explaining each parameter based on the live docs.

#### Optional Enhancements

If the user asks for a CLI tool, add `argparse`:
```python
import argparse

parser = argparse.ArgumentParser(description="Naver API Client")
parser.add_argument("query", help="Search query")
parser.add_argument("--display", type=int, default=10, help="Results per page (1-100)")
parser.add_argument("--sort", choices=["sim", "date"], default="sim", help="Sort order")
args = parser.parse_args()
```

If the user asks to save results, include file I/O:
```python
with open("output.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=2)
```

If the user asks for data processing (e.g., extracting titles from news results), include parsing logic:
```python
for item in data.get("items", []):
    print(item["title"].replace("<b>", "").replace("</b>", ""))
```

### Step 4: Validate the Code

Before presenting the code to the user:
1. Verify the endpoint URL matches what you saw in the docs.
2. Verify all required parameters are included.
3. Verify the HTTP method (GET vs POST) is correct.
4. Check that auth headers are present.
5. Run a syntax check if possible:
   ```bash
   python3 -m py_compile generated_script.py
   ```

## Important Notes

- **Base URL is always `https://openapi.naver.com`** for Open APIs. Do not confuse with `https://api.searchad.naver.com` (Search Ad API).
- **Daily rate limits**: Search APIs = 25,000 calls/day. Datalab = 1,000 calls/day.
- **The live docs are the source of truth** — always browse them to confirm the latest parameter names and endpoints.
- **Search API queries must be UTF-8 encoded.** `requests` does this automatically when you pass a string via `params`, but if building URLs manually, use `urllib.parse.quote`.

## Examples

**Example 1: News Search**
Input: "Generate Python code to search Naver News for 'AI' and print the titles."
Output: A Python script that calls `GET /v1/search/news.json?query=AI`, parses the `items` array, strips `<b>` tags from titles, and prints them.

**Example 2: Datalab Trends**
Input: "Write a script to compare search trends for 'Python' vs 'JavaScript' monthly in 2024."
Output: A Python script that calls `POST /v1/datalab/search` with two `keywordGroups`, monthly `timeUnit`, and prints the ratio data for each period.

**Example 3: Blog Search with CLI**
Input: "Make a CLI tool to search Naver blogs."
Output: A Python script with `argparse` that accepts a query, optional display count and sort order, calls `GET /v1/search/blog.json`, and prints results.
