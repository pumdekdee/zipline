# Claude Project Context: zipline

This file provides context for the **zipline** repository, used as part of an AI trading-knowledge project.

## Project Overview
* **Name**: Zipline
* **Language**: Python
* **Type**: Event-driven backtesting library (originally built by Quantopian)
* **Purpose**: Backtest equity (and to a lesser extent futures) trading algorithms against historical daily/minute bar data using a Pipeline API for cross-sectional factor computation.
* **License**: Apache 2.0
* **Status**: Archived/no longer actively maintained by Quantopian (Quantopian shut down) — community forks such as `zipline-reloaded` (maintained by Stefan Jansen's team) are the actively updated alternative. Treat the original repo as a stable historical reference; point users to `zipline-reloaded` for current installs.

## Core System Boundaries
* **Algorithm structure**: algorithms are defined via `initialize(context)` and `handle_data(context, data)` (or `before_trading_start`) functions — a callback-based model rather than an OOP `Strategy` subclass.
* **Pipeline API**: a separate, powerful system (`zipline.pipeline`) for computing cross-sectional factors/filters across an entire universe of assets each day — central to factor research workflows (used heavily in `machine-learning-for-trading` chapter 4-5 examples).
* **Data bundles**: historical data must be "ingested" into a local bundle (e.g. `quandl`, `quantopian-quandl`, or custom bundles) before backtests can run — `zipline ingest -b <bundle-name>`.
* **Calendars**: trading calendars (NYSE, etc.) determine valid simulation dates — mismatched calendars are a common source of errors.

## Repository Structure & Key Directories
* `/zipline/` - Core package
  * `/zipline/algorithm.py` - `TradingAlgorithm` base class and core API functions (`order`, `record`, `symbol`, etc.)
  * `/zipline/pipeline/` - Pipeline API: `Factor`, `Filter`, `Classifier`, built-in factors (e.g. `Returns`, `SimpleMovingAverage`)
  * `/zipline/data/` - Data bundle infrastructure, readers/writers for bcolz/HDF5-based bar data
  * `/zipline/finance/` - Order execution, slippage, and commission models
  * `/zipline/utils/` - Calendars and misc utilities
* `/tests/` - Reference usage patterns for algorithms and Pipeline factors

## Standard Operational Commands
* **Install (prefer the maintained fork)**: `pip install zipline-reloaded` (original `pip install zipline` often fails on modern Python due to outdated pinned dependencies)
* **Ingest a data bundle**: `zipline ingest -b quandl` (requires API key for some bundles)
* **Run a backtest**: `zipline run -f algo.py --start 2015-1-1 --end 2018-1-1 -b quandl -o out.pickle`

## Standard Python API Usage (Algorithm Blueprint)
```python
from zipline.api import order_target_percent, symbol, record

def initialize(context):
    context.asset = symbol("AAPL")

def handle_data(context, data):
    price = data.current(context.asset, "price")
    record(price=price)
    if price > data.history(context.asset, "price", 50, "1d").mean():
        order_target_percent(context.asset, 1.0)
    else:
        order_target_percent(context.asset, 0.0)
```

## AI Assistant Guidelines for this Project
1. **Recommend the maintained fork**: when a user wants to install/run Zipline today, point them to `zipline-reloaded` (and `pip install zipline-reloaded`) rather than the original PyPI `zipline` package, due to dependency-compatibility issues on modern Python.
2. **Two paradigms**: distinguish between the simple `initialize`/`handle_data` algorithm API (good for single/few-asset strategies) and the Pipeline API (for cross-sectional, universe-wide factor research) — pick based on what the user is trying to do.
3. **Data bundle setup is a common blocker**: many issues stem from missing/incorrectly ingested data bundles — walk users through `zipline ingest` before debugging algorithm code.
4. **Equities/futures focus**: Zipline's calendar and asset model is built around US equities (and some futures); it is not designed for crypto or forex — for those, point to `ccxt`/`freqtrade`/`nautilus_trader` instead.
5. **Pair with alphalens/pyfolio**: factor evaluation (`alphalens-reloaded`) and performance analysis (`pyfolio-reloaded`) are companion libraries commonly used alongside Zipline in the `zipline-reloaded` ecosystem.
