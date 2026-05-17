# Pocket Option Telegram Bot

Telegram bot for Pocket Option market monitoring, signal generation, analytics, paper trading, and optional QT demo/real trade execution.

## Project Status

Development is stopped.

Reason: Pocket Option support stated that automated trading through a custom Python bot is prohibited on Pocket Option. According to that support response, Pocket Option officially supports automated trading only through its own Telegram Signal Bot, and third-party automation such as custom Python scripts, browser extensions, or other bots can be treated as a Terms and Conditions violation and may lead to account restrictions or blocking.

This repository is kept as-is for reference. There is no support, no warranty, and no promise that the code will continue to work with Pocket Option.

## Important Warning

Use this project only at your own risk.

This project is:

- not official Pocket Option software;
- not approved or supported by Pocket Option;
- not financial advice;
- not a guaranteed-profit bot;
- not recommended for real trading;
- not maintained as a supported product.

Possible risks include:

- Pocket Option account restrictions or blocking;
- loss of access to the platform;
- financial loss;
- broken connectivity if Pocket Option changes its internal protocol;
- incorrect signals, trade results, payouts, or analytics;
- operational failures caused by network, database, server, or Telegram issues.

If you want compliant automation, use Pocket Option's official Telegram Signal Bot instead of this project.

## What The Bot Does

The bot can:

- connect to Pocket Option through an internal raw Socket.IO client;
- collect market/candle data for multiple assets;
- calculate signals with built-in strategies;
- store candles, signals, trades, settings, and events in PostgreSQL;
- expose settings and status through Telegram commands;
- send Telegram notifications;
- run in safe `signal_only` mode;
- run paper trading inside the project;
- optionally send QT demo or QT real orders if explicitly enabled.

All trade execution is intended to pass through:

```text
Strategy -> SignalSelector -> RiskManager -> Trader
```

Strategies only generate signals. They should not place trades directly.

## Default Safety Settings

By default, trading is disabled:

```env
SIGNAL_ONLY=true
TRADING_ENABLED=false
PAPER_TRADING=false
QT_ACCOUNT_MODE=demo
REAL_TRADING_UNLOCKED=false
REAL_TRADING_ADMIN_CONFIRMED=false
```

With these settings, the bot should only observe the market, save candles/signals, and send notifications. It should not open Pocket Option trades.

Real trading requires multiple explicit switches and confirmations. Because of the support response described above, enabling real trading is strongly discouraged.

## Tech Stack

- Python 3.11+
- aiogram 3.x
- PostgreSQL
- SQLAlchemy 2.x async
- asyncpg
- Alembic
- pydantic-settings
- pytest
- ruff
- Docker Compose for local PostgreSQL
- systemd example for Debian 12

## Local Setup

Create a virtual environment and install dependencies:

```bash
python3.11 -m venv .venv
.venv/bin/pip install -r requirements.txt
```

Create `.env`:

```bash
cp .env.example .env
```

Edit `.env` and set at least:

```env
TELEGRAM_BOT_TOKEN=your_telegram_bot_token
TELEGRAM_ADMIN_IDS=123456789
TELEGRAM_NOTIFY_CHAT_ID=123456789
```

Optional Pocket Option settings:

```env
POCKET_OPTION_PROXY=
POCKET_OPTION_USE_ENV_PROXY=false
POCKET_OPTION_WS_URL=
```

The Pocket Option SSID can be stored through Telegram with `/set_ssid <ssid>`. Do not commit SSID or Telegram tokens.

## Database

Start PostgreSQL:

```bash
docker compose up -d postgres
```

Apply migrations:

```bash
PYTHONPATH=. .venv/bin/alembic upgrade head
```

Default database URL:

```env
DATABASE_URL=postgresql+asyncpg://pocket_bot:change_me@localhost:5432/pocket_bot
```

## Run Locally

Start the application:

```bash
PYTHONPATH=. .venv/bin/python -m app.main
```

The main process starts Telegram polling and the engine worker. Normally you do not need to run the worker manually.

## Docker

Start PostgreSQL only:

```bash
docker compose up -d postgres
```

Start the app profile:

```bash
docker compose --profile app up -d
```

## Tests And Linting

Run tests:

```bash
PYTHONPATH=. .venv/bin/pytest
```

Run ruff if installed in the environment:

```bash
.venv/bin/ruff check .
```

## Debian 12 systemd

Example service file is in:

```text
deploy/pocket-option-telegram-bot.service
```

Typical install path:

```text
/opt/pocket-option-telegram-bot
```

Typical commands:

```bash
sudo cp deploy/pocket-option-telegram-bot.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable pocket-option-telegram-bot
sudo systemctl start pocket-option-telegram-bot
sudo systemctl status pocket-option-telegram-bot
```

Check logs:

```bash
journalctl -u pocket-option-telegram-bot -f
```

## Telegram Access

Only users listed in `TELEGRAM_ADMIN_IDS` are allowed. Unknown users receive:

```text
Access denied.
```

Use comma-separated IDs:

```env
TELEGRAM_ADMIN_IDS=123456789,987654321
```

## Telegram Commands

Main:

- `/start` - main menu.
- `/help` - command help.
- `/status` - bot, engine, connection, mode, settings, and open trades status.
- `/settings` - current runtime settings.
- `/balance` - Pocket Option balance.
- `/assets` - available Pocket Option assets.
- `/stats` - today's stats.
- `/stats 1h` - stats for the last hour.
- `/stats 12h` - stats for the last 12 hours.
- `/stats 1d` - stats for the last day.
- `/stats 2d` - stats for the last 2 days.
- `/last_trades` - latest trades.
- `/open_trades` - currently open trades.

Pocket Option session:

- `/ssid_status` - show whether SSID is saved, masked.
- `/set_ssid <ssid>` - save Pocket Option SSID.
- `/clear_ssid` - delete saved SSID.

Modes:

- `/signal_only` - enable safe signal-only mode and disable trading.
- `/paper_trading` - request confirmation for internal paper trading.
- `/paper_status` - paper trading status.
- `/paper_report` - today's paper trading report.
- `/stop_paper` - stop paper trading and return to signal-only mode.
- `/qt_demo` - set QT account mode to demo.
- `/qt_real` - request confirmation to set QT account mode to real.
- `/start_demo_trading` - request confirmation to enable QT demo trading.
- `/start_trading` - request confirmation to enable QT real trading.
- `/stop_trading` - disable trading.
- `/confirm <code>` - confirm a pending dangerous action.
- `/cancel` - cancel pending confirmation.

Strategies and market settings:

- `/strategies` - list available strategies.
- `/set_strategy trend_pullback` - use trend pullback strategy.
- `/set_strategy bollinger_mean_reversion` - use Bollinger mean reversion strategy.
- `/set_strategy breakout_retest` - use breakout retest strategy.
- `/set_strategy all` - run all strategies.
- `/set_assets EURUSD_otc,GBPUSD_otc` - set active assets.
- `/sets_assets EURUSD_otc,GBPUSD_otc` - alias for `/set_assets`.
- `/set_timeframe 60` - set candle timeframe in seconds.
- `/set_expiration 60` - set expiration in seconds.
- `/set_expiration_auto` - enable automatic expiration selection.
- `/set_expiration_fixed 60` - set fixed expiration.
- `/slow_signals` - preset for slower 5-minute signal mode.
- `/set_min_payout 0.75` - set minimum payout filter.

Risk limits:

- `/limits` - show risk limits.
- `/set_amount 1` - set trade amount; upward changes may require confirmation.
- `/set_max_trades_day 5` - max total trades per day.
- `/set_max_trades_asset 1` - max trades per asset per day.
- `/set_max_open_trades 1` - max open trades total.
- `/set_max_open_asset 1` - max open trades per asset.
- `/set_max_losses_day 2` - max losses per day.
- `/set_max_consecutive_losses 2` - max consecutive losses.
- `/set_daily_stop_loss 10` - daily stop loss.
- `/set_daily_take_profit 20` - daily take profit.
- `/set_loss_cooldown 900` - cooldown after loss in seconds.
- `/set_asset_cooldown 300` - per-asset cooldown in seconds.

Chat cleanup:

- `/clear_chat 50` - delete recent bot/user messages where Telegram permits it.
- `/clear_history 50` - alias for `/clear_chat`.

## Suggested Safe Use

Because third-party automation may violate Pocket Option rules, the safest option is not to use this project with a Pocket Option account.

If you still decide to run it:

1. Keep `SIGNAL_ONLY=true`.
2. Keep `TRADING_ENABLED=false`.
3. Do not enable QT real trading.
4. Do not store secrets in git.
5. Use a separate test Telegram bot and test chat.
6. Start with one or two assets.
7. Watch logs and `/status`.
8. Treat all analytics as experimental.
9. Do not judge a strategy on a tiny sample.

For analytics, a more meaningful sample usually needs days or weeks of data:

- 3-5 days for a first technical check;
- 7-14 days for an initial read;
- 30 days for a better read;
- 300+ signals minimum;
- 1000+ signals preferred.

## Strategies

Available strategy names:

- `trend_pullback`
- `bollinger_mean_reversion`
- `breakout_retest`
- `all`

All strategies use closed candles only and return signals. They do not open trades directly.

## Secrets

Never publish or commit:

- `POCKET_OPTION_SSID`
- `TELEGRAM_BOT_TOKEN`
- production database credentials
- proxy credentials

The project includes secret masking, but that is not a substitute for careful operational handling.

## Final Note

Pocket Option support's position means this project should be treated as an unsupported experimental codebase, not as a production trading tool. If you use it anyway, you accept the platform, account, technical, and financial risks yourself.
