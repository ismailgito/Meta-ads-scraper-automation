# Meta Ads Competitor Analysis — n8n Automation

An n8n workflow that pulls competitor ad creatives from the Meta Ads Library, filters for likely "winning" ads, classifies them with AI, and logs the results to Google Sheets — with a Telegram notification when each run completes.

## Problem

Keeping tabs on what competitors are running in the Meta Ads Library today means scrolling through the library by hand, eyeballing which ads have been live the longest, and manually noting the hook, visual style, copy angle, and CTA for each one. It's slow, inconsistent between team members, and produces notes that are hard to compare or search later. There's no easy way to turn "I saw this ad running for months" into structured, filterable data your team can actually act on.

## Solution

This workflow automates the research loop end to end:

1. **Scrape** — Queries the Meta Ads Library (via the Apify `facebook-ads-library-scraper` actor) for a given set of search terms and countries.
2. **Filter** — Keeps only ads that have been running 30+ days, used as a proxy signal for ads that are likely performing well (advertisers don't keep paying for losers).
3. **Normalize** — Maps the raw scrape output into clean fields: Library ID, company name, status, days running, platform, headline, body copy, CTA, landing page, and launch date.
4. **Classify with AI** — An AI Agent (Google Gemini 2.5 Flash) analyzes each ad against a controlled vocabulary and returns a structured JSON object covering hook type, visual format, copy angle, CTA, longevity, and a longevity-based "result" label.
5. **Log** — Parses the AI output and appends a new row per ad to a Google Sheet.
6. **Notify** — Sends a Telegram message summarizing the run once the sheet is updated.

The result is a living, structured spreadsheet of competitor ad creative — built without anyone manually copying and pasting from the Ads Library.

## Target Audience

- **Performance Marketers** — quickly reverse-engineer what's working in competitors' paid campaigns before building your own creative briefs
- **Marketing Agencies** — run repeatable creative research across multiple client accounts and competitors without extra manual hours
- **Digital Marketers** — keep an always-on pulse on messaging and creative trends without manually browsing the Ads Library
- **Growth Marketers** — spot creative patterns (hooks, formats, angles) worth testing, backed by longevity data instead of gut feel

## Architecture

```
Trigger
  → HTTP Request (Apify: Facebook Ads Library Scraper)
  → Filter (daysRunning >= 30)
  → Edit Fields (normalize raw scrape output)
  → AI Agent (Gemini 2.5 Flash — classifies hook / format / angle / CTA / longevity / result)
  → Code (parse AI Agent's JSON output)
  → Google Sheets (append row)
  → Wait
  → Telegram (send completion notification)
```

## Setup

1. Import `Workflow.json` into your n8n instance.
2. Add your own credentials for each of these nodes (all credential references were stripped before publishing):
   - **Apify API** — on the HTTP Request node
   - **Google Gemini (PaLM) API** — on the AI Agent's chat model
   - **Google Sheets OAuth2** — on the Append row node
   - **Telegram API** — on the Send a text message node
3. Replace the placeholder values:
   - `YOUR_GOOGLE_SHEET_ID` (appears in both the Google Sheets node and the Telegram message text) → your target spreadsheet ID
   - `YOUR_TELEGRAM_CHAT_ID` → your Telegram chat ID
4. Edit the Apify request body on the HTTP Request node to set your own `searchTerms`, `countries`, and `maxAds`.
5. Trigger the workflow manually, or swap the Manual Trigger for a Schedule Trigger to run it on a recurring basis.

## Notes

- The **"result"** field is a longevity-based proxy, not real performance data — the Meta Ads Library doesn't expose spend, CTR, or ROAS for standard ads, so "likely winning" is inferred from how long an ad has stayed active.
- No API keys, OAuth tokens, or personal identifiers (chat IDs, spreadsheet IDs, instance ID) are included in this repo. You'll need to connect your own credentials after import.

## License

No license specified yet — add one (e.g. MIT) if you want others to reuse or modify this workflow.
