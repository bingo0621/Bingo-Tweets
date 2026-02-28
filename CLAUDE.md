# CLAUDE.md - AI Assistant Guide for Bingo-Tweets (sumTweets)

## Project Overview

**sumTweets** is an automated Twitter/X content aggregation and summarization tool that:
- Fetches tweets from specified Twitter accounts or lists via Nitter RSS feeds
- Uses AI (LLM via LiteLLM) to summarize and curate tweets by topic relevance
- Converts summaries to HTML-formatted markdown
- Sends curated content via email on a cron schedule (every 20 minutes via GitHub Actions)

**Author**: Frank Lin (bingo0621) | **License**: MIT

## Repository Structure

```
Bingo-Tweets/
├── .github/workflows/main.yml   # GitHub Actions cron workflow (every 20 min)
├── .gitignore                    # Standard Python gitignore
├── LICENSE                       # MIT License
├── README.md                     # Project documentation (Chinese)
├── requirements.txt              # Python dependencies (no version pins)
├── main.py                       # Entire application (~110 lines)
└── CLAUDE.md                     # This file
```

This is a **single-file Python application** — all logic lives in `main.py`.

## Tech Stack

- **Language**: Python 3.x
- **Dependencies**: `litellm`, `feedparser`, `pandas`, `requests`, `bs4` (BeautifulSoup), `markdown`, `python-dotenv`
- **Runtime**: GitHub Actions (scheduled cron) or local execution
- **AI Provider**: OpenAI-compatible API via LiteLLM (supports proxies and alternative providers)
- **Data Source**: Nitter (privacy-focused Twitter frontend) RSS feeds

## Key Functions in main.py

| Function | Purpose |
|----------|---------|
| `sumTweets(prompt, lang, length, model, mail, render)` | Main entry: fetches tweets via RSS, filters by time window, resolves quoted tweets, sends to LLM for summarization, renders to HTML, and emails result |
| `sendEmail(message, receiver, subject)` | Sends HTML email via SMTP (default: 163.com) |

### Data Flow

```
Nitter RSS → feedparser → pandas DataFrame → time filtering →
regex link resolution → quoted tweet embedding → LLM summarization →
markdown → HTML rendering → SMTP email delivery
```

## Environment Variables

**Required** (set via `.env` file locally or GitHub Actions secrets/variables):

| Variable | Source | Description |
|----------|--------|-------------|
| `OPENAI_API_KEY` | secret | OpenAI API key |
| `API_BASE_URL` | variable | API endpoint URL (supports proxies) |
| `MAILTO` | secret | Recipient email(s), semicolon-separated |
| `MAIL` | variable | Sender email address |
| `SMTP` | variable | SMTP server (e.g., `smtp.163.com`) |
| `MAILPWD` | secret | SMTP password |
| `NITTER` | variable | Nitter instance domain |
| `MINS` | variable | Tweet lookback window in minutes |
| `TARGET` | variable | Twitter handles/list IDs, semicolon-separated |
| `INFO` | variable | Topic keyword for filtering (e.g., `AI/人工智能`) |

**Optional**:
| Variable | Description |
|----------|-------------|
| `PROMPT` | Custom AI prompt template (uses default Chinese prompt if empty) |

## Development Workflow

### Running Locally
```bash
# 1. Create .env file with required variables (see README.md)
# 2. Install dependencies
pip install -r requirements.txt
# 3. Run
python main.py
```

### CI/CD
- **GitHub Actions** runs `main.py` every 20 minutes via cron (`*/20 * * * *`)
- Workflow file: `.github/workflows/main.yml`
- Environment name: `sumTweetsCron`
- Uses pip caching for faster builds

### No Tests
This project has no test suite, linter, or formatter configured.

## Code Conventions

- **Language**: Comments and docstrings are in Chinese
- **Style**: Procedural — no classes, two module-level functions
- **Imports**: Compact style (`import os,re` on one line)
- **Data manipulation**: Heavy use of pandas DataFrames
- **No type checking**: No mypy, pyright, or type annotations beyond basic hints
- **No logging framework**: Uses `print()` for output
- **Error handling**: Minimal — basic try-except for SMTP only

## Important Notes for AI Assistants

1. **Single file codebase**: All changes go in `main.py` unless adding new files is necessary
2. **Sensitive data**: Never commit `.env` files — they contain API keys and passwords
3. **Nitter dependency**: The project depends on third-party Nitter instances which may go down; check instance availability at https://status.d420.de
4. **LiteLLM model format**: Models use `provider/model-name` format (e.g., `openai/gpt-3.5-turbo-1106`)
5. **TARGET format**: Semicolon-separated, supports both usernames (`elonmusk`) and list IDs (`i/lists/1234567890`)
6. **Default prompt**: The built-in prompt is in Chinese and formats output as a markdown article with tweet timestamps, authors, links, and AI commentary
7. **No breaking changes**: The cron job runs every 20 minutes in production — test changes carefully before pushing
