# UK Deals

> Search HotUKDeals for UK bargains across categories, filter by preferences, deliver summaries via Telegram or Obsidian, and flag watch-list price matches.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://github.com/Marrowleaf/hermes-uk-deals)

## Features

- Search deals across 5 categories: Tech, Fitness, Supplements, Food, Home
- Configurable minimum discount percentage and vote score thresholds
- Preferred and excluded retailer filtering
- Watch-list price matching with ⚡ flag for target price hits
- Telegram delivery with formatted deal cards
- Obsidian vault integration with daily deal notes
- Cron-scheduled deal alerts (e.g., daily at 9am)
- Deduplication within 24-hour windows
- Browser-based scraping with JS rendering support

## Installation

```bash
hermes skills install productivity/uk-deals
```

Or manually clone into `~/.hermes/skills/productivity/uk-deals/`.

## Usage

```
uk-deals tech
uk-deals fitness --min-discount 40 --max 3
uk-deals food --retailer "Tesco"
uk-deals all
uk-deals cron "0 9 * * *" tech fitness
uk-deals cron remove tech
uk-deals tech --watch-list
```

## Configuration

- `min_discount_pct`: Minimum discount % to include (default: 20)
- `min_vote_score`: Minimum HotUKDeals vote score (default: 50)
- `preferred_retailers`: Priority retailers list (e.g., Amazon UK, Argos, Currys)
- `excluded_retailers`: Blocked retailers list
- `max_deals_per_category`: Cap per category (default: 10)
- `delivery`: Output channel — telegram, obsidian, both, or stdout (default: telegram)
- `location`: Default "Oakham, Rutland, UK"
- Watch-list file at `~/.hermes/skills/productivity/uk-deals/references/watch-list.yaml`

## Requirements

- Hermes Agent v0.12+
- Browser tools (Playwright/Puppeteer for HotUKDeals scraping)
- Cron scheduler (for automated alerts)
- Obsidian vault (optional, for deal history)

## License

MIT

## Related Hermes Skills
- [hermes-airtable](https://github.com/Marrowleaf/hermes-airtable) — Airtable integration and automation
- [hermes-budget-tracker](https://github.com/Marrowleaf/hermes-budget-tracker) — Personal budget tracking and analysis
- [hermes-consensus-engine](https://github.com/Marrowleaf/hermes-consensus-engine) — Multi-model consensus and decision engine
- [hermes-nano-pdf](https://github.com/Marrowleaf/hermes-nano-pdf) — Lightweight PDF processing and manipulation
- [hermes-notion](https://github.com/Marrowleaf/hermes-notion) — Notion integration and management
