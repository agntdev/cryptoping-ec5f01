# CryptoPing — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot for private crypto price tracking with threshold/percent alerts, watchlist price checks, and owner metrics. Users manage personalized coin lists with quick-add buttons for Bitcoin, Ethereum, Toncoin, and custom tickers. Alerts suppress spam after triggers, with quiet hours and optional daily digests. Owner receives non-identifying usage stats.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individual crypto traders
- Crypto holders seeking price alerts
- Bot administrator for metrics

## Success criteria

- Users can add/remove coins and set alerts via inline buttons
- Alerts trigger with correct price thresholds/percent changes
- Owner dashboard shows aggregate metrics without user-identifying data

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Begin onboarding: request timezone and offer quick-add buttons for Bitcoin/Ethereum/Toncoin
- **/price** (command, actor: user, command: /price) — Show current price for all watchlist coins or specific coin when ticker is provided
- **Add Bitcoin** (button, actor: user, callback: add_coin:BTC) — Quick-add Bitcoin to watchlist with optional alert setup
  - inputs: coin ticker
  - outputs: watchlist entry
- **Add Custom Ticker** (button, actor: user, callback: add_coin:custom) — Prompt user to type arbitrary coin ticker
  - inputs: coin ticker
  - outputs: watchlist entry

## Flows

### Onboarding
_Trigger:_ /start

1. Confirm Telegram identity
2. Request timezone (or auto-detect)
3. Show quick-add buttons for Bitcoin/Ethereum/Toncoin
4. Prompt to set initial alerts

_Data touched:_ user profile

### Add Watchlist Coin
_Trigger:_ add_coin:BTC or add_coin:custom

1. Validate ticker symbol
2. Show coin price confirmation
3. Prompt to create threshold/percent alerts
4. Save to watchlist with default enabled status

_Data touched:_ watchlist entry

### Alert Trigger
_Trigger:_ price update from data source

1. Check all active alert rules against current price
2. Send notification with snooze/disable buttons
3. Mark rule as paused for suppression period
4. Log event for owner metrics

_Data touched:_ alert rule, notification record

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User Profile** _(retention: persistent)_ — Telegram ID, timezone, quiet hours, digest time, alert pause duration
  - fields: telegram_id, timezone, quiet_hours_start, quiet_hours_end, digest_time, alert_pause_hours
- **Watchlist Entry** _(retention: persistent)_ — Coin ticker, friendly name, enabled status, last-known price
  - fields: ticker, friendly_name, enabled, last_price, last_alert_ts
- **ThresholdAlert** _(retention: persistent)_ — Price threshold alert rule
  - fields: direction, price, last_fired
- **PercentAlert** _(retention: persistent)_ — Percent change alert rule with window
  - fields: direction, percent, window, last_fired
- **PriceSourceState** _(retention: session)_ — Market data API reliability tracking
  - fields: last_success, retry_schedule

## Integrations

- **Telegram** (required) — Bot API messaging and inline buttons
- **Market Data API** (required) — Real-time crypto price data
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View total active users
- See top 10 most-triggered alert coins
- Configure default alert suppression duration
- Set price data source retry parameters

## Notifications

- Price alert notifications with snooze/disable buttons
- Morning digest summary of watchlist changes
- Quiet hours alert summary at window end

## Permissions & privacy

- User data stored privately per Telegram ID
- Owner sees only aggregate metrics (no individual watchlists)
- Price data source has no access to user identities

## Edge cases

- Misspelled tickers with fuzzy matching suggestions
- Price data source outages with silent retries
- Alert suppression during quiet hours
- Multiple alert triggers during suppression period

## Required tests

- Verify alert suppression for 6 hours after trigger
- Test digest delivery with 3%+ 24h movers
- Validate quiet hours queueing with 100+ queued alerts

## Assumptions

- Price data source provides 1m-resolution data
- Default quiet hours set to 10pm-7am local time
- Morning digest defaults to 7am local time
