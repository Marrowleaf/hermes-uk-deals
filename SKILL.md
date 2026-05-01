---
name: uk-deals
version: 1.0.0
description: Search HotUKDeals for UK deals across categories, filter by preferences, deliver summaries via Telegram or Obsidian, support cron alerts, and flag watch-list price matches.
author: hermes-agent
tags:
  - deals
  - uk
  - hotukdeals
  - shopping
  - alerts
  - Obsidian
  - Telegram
categories:
  - productivity
dependencies:
  - browser (Playwright/Puppeteer for scraping)
  - cron scheduler
  - Obsidian vault access (optional)
channels:
  - telegram
  - obsidian
  - stdout
config_schema:
  min_discount_pct:
    type: integer
    default: 20
    description: Minimum discount percentage to include in results
  min_vote_score:
    type: integer
    default: 50
    description: Minimum HotUKDeals vote score to include
  preferred_retailers:
    type: array
    items:
      type: string
    default: []
    description: List of preferred retailers to prioritise (e.g. Amazon UK, Argos, Currys)
  excluded_retailers:
    type: array
    items:
      type: string
    default: []
    description: Retailers to exclude from results
  max_deals_per_category:
    type: integer
    default: 10
    description: Maximum number of deals to return per category
  obsidian_vault_path:
    type: string
    default: ~/obsidian-vault
    description: Path to Obsidian vault for saving deal summaries
  watch_list_path:
    type: string
    default: ""
    description: Path to YAML watch-list file for price-matching
  cron_schedule:
    type: string
    default: ""
    description: Cron expression for scheduled alerts (e.g. '0 9 * * *' for daily 9am)
  delivery:
    type: string
    enum: [telegram, obsidian, both, stdout]
    default: telegram
    description: Where to deliver deal summaries
  location:
    type: string
    default: "Oakham, Rutland, UK"
    description: User location for region-specific deals
---

# UK Deals Skill

Search [HotUKDeals](https://www.hotukdeals.com) for bargains across configurable categories, apply intelligent filters, and deliver curated summaries to Telegram or an Obsidian vault. Supports cron-scheduled alerts and watch-list price matching.

## Supported Categories

| Category | Keyword | HotUKDeals Tag |
|----------|---------|----------------|
| Tech | `tech` | Electronics, Computing, Phones |
| Fitness | `fitness` | Sports & Fitness, Exercise Equipment |
| Supplements | `supplements` | Health & Beauty, Vitamins |
| Food | `food` | Groceries, Food & Drink |
| Home | `home` | Home & Garden, DIY, Appliances |

## Usage

### On-Demand Deal Search

```
/uk-deals [category] [--min-discount 30] [--min-votes 100] [--retailer "Amazon UK"] [--max 5]
```

**Examples:**

- `/uk-deals tech` — search tech deals with defaults
- `/uk-deals fitness --min-discount 40 --max 3` — top 3 fitness deals with ≥40% off
- `/uk-deals food --retailer "Tesco"` — food deals from Tesco only
- `/uk-deals all` — search every category

### Scheduled Alerts (Cron)

Set up a recurring deal alert:

```
/uk-deals cron "0 9 * * *" tech fitness
```

This sends a Telegram message every day at 9am with the best tech and fitness deals.

Remove a cron alert:

```
/uk-deals cron remove tech
```

### Watch-List Price Matching

Maintain a YAML watch list of products you're tracking. The skill flags any deal that matches an item at or below your target price.

Watch-list file format (`watch-list.yaml`):

```yaml
# ~/.hermes/skills/productivity/uk-deals/references/watch-list.yaml
items:
  - name: "Sony WH-1000XM5"
    category: tech
    target_price: 200
    match_aliases:
      - "WH1000XM5"
      - "xm5 headphones"
  - name: "Optimum Nutrition Gold Standard Whey"
    category: supplements
    target_price: 35
    match_aliases:
      - "ON Gold Standard"
```

```
/uk-deals tech --watch-list
```

Any deal matching a watch-list item at or below `target_price` is flagged with ⚡ in the output.

## How It Works

### 1. Scraping

The skill uses the browser tool (Playwright/Puppeteer) to load HotUKDeals category/tag pages. It extracts the following from each deal card:

| Field | Selector Strategy |
|-------|------------------|
| Title | Deal card heading |
| Price | `£` prefixed price element |
| Original Price | Strikethrough price element (used to calculate discount %) |
| Retailer | Merchant badge or text |
| Expiry | Timer/countdown element if present |
| Vote Score | Temperature/vote count element |
| URL | Link to deal page |

The scraper respects HotUKDeals' structure and handles pagination up to `max_deals_per_category`.

### 2. Filtering

After scraping, deals are filtered through:

1. **Minimum discount** — deals below `min_discount_pct` are excluded
2. **Minimum vote score** — deals below `min_vote_score` are excluded
3. **Preferred retailers** — if set, these are sorted to the top
4. **Excluded retailers** — deals from these retailers are dropped
5. **Watch-list match** — deals matching a watch-list item at ≤ target price get a ⚡ flag

### 3. Delivery

#### Telegram

Delivers a formatted message with deal cards:

```
🇬🇧 UK Deals — Tech (9 May 2026)

🔥 Sony WH-1000XM5 — £189 (was £320, −41%)
   Retailer: Amazon UK | ⬆️ 312° | ⚡ WATCH MATCH
   🔗 https://hotukdeals.com/...

📉 Samsung 990 Pro 2TB — £129 (was £220, −41%)
   Retailer: Currys | ⬆️ 187°
   🔗 https://hotukdeals.com/...
```

#### Obsidian

Creates/updates a Markdown note in the vault. Default path:

```
{obsidian_vault_path}/Daily Deals/{YYYY-MM-DD} — {Category} Deals.md
```

Uses the template at `references/obsidian-template.md`.

#### Stdout

Prints the same structured output to the terminal for debugging or piping.

## File Structure

```
~/.hermes/skills/productivity/uk-deals/
├── SKILL.md                           # This file
└── references/
    ├── category-urls.yaml             # Category → HotUKDeals URL mappings
    ├── telegram-template.md           # Telegram message template
    ├── obsidian-template.md           # Obsidian note template
    └── watch-list.yaml                # Example watch-list file
```

## Configuration Defaults

| Key | Default | Description |
|-----|---------|-------------|
| `min_discount_pct` | 20 | Minimum discount % |
| `min_vote_score` | 50 | Minimum vote score (°) |
| `preferred_retailers` | `[]` | Priority retailers |
| `excluded_retailers` | `[]` | Blocked retailers |
| `max_deals_per_category` | 10 | Cap per category |
| `obsidian_vault_path` | `~/obsidian-vault` | Vault location |
| `watch_list_path` | *empty* | Path to watch-list YAML |
| `cron_schedule` | *empty* | Cron expression for alerts |
| `delivery` | `telegram` | Output channel |
| `location` | Oakham, Rutland, UK | For region-specific deals |

Override any setting by editing the YAML frontmatter or passing CLI flags.

## Cron Scheduling

The skill integrates with the system cron scheduler. When a `cron_schedule` is set:

1. A crontab entry is created that invokes the skill at the specified times
2. The skill runs the search, applies filters, and delivers results to the configured channel
3. Duplicate alerts within a 24-hour window are suppressed

Cron entries are managed via:

```
/uk-deals cron "<cron-expr>" <categories...>   # Add/update
/uk-deals cron list                              # List active cron alerts
/uk-deals cron remove [category]                # Remove a category alert
```

## Watch-List Matching Logic

1. Load the watch list from `watch_list_path`
2. For each scraped deal, compare the deal title against every watch-list item's `name` and `match_aliases`
3. Matching uses case-insensitive substring search
4. If the deal price ≤ the item's `target_price`, flag it with ⚡
5. Watch-match deals are **always included** regardless of discount/vote thresholds

## Notes & Limitations

- HotUKDeals may update their HTML structure; scraper selectors may need periodic maintenance
- The site loads deal cards dynamically — the browser tool must wait for JS rendering
- Rate-limiting: avoid scraping more than once per minute to be respectful
- Some deals lack explicit original prices; discount % is estimated when possible or omitted
- Expiry data is not always available; deals without expiry show "No expiry listed"
- Location-specific deals are filtered by UK availability (HotUKDeals is UK-focused by default)

## Changelog

- **1.0.0** — Initial skill: category search, filtering, Telegram/Obsidian delivery, cron, watch-list matching