# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jesse is an advanced crypto trading framework for backtesting, optimizing, and live trading strategies. It's built in Python with a FastAPI web backend and supports multiple exchanges, timeframes, and symbols simultaneously.

## Development Commands

### Testing
```bash
# Run all tests
pytest

# Install test dependencies (if not in requirements.txt)
pip install pytest

# Run specific test file
pytest tests/test_backtest.py

# Run tests matching a pattern
pytest -k "test_pattern"
```

### Installation and Setup
```bash
# Install Jesse in development mode
pip install -e . -U

# Install dependencies
pip install -r requirements.txt

# Install live trading plugin (optional)
jesse install-live

# Run Jesse web application
jesse run
```

### Database Operations
```bash
# Database migrations are run automatically when starting Jesse
# Manual migration debugging can be done through the jesse.services.migrator module
```

## Architecture Overview

### Core Components

**Strategy Framework (`jesse/strategies/`)**
- `Strategy.py`: Abstract base class for all trading strategies
- Strategies inherit from `Strategy` and implement `should_long()`, `should_short()`, `go_long()`, `go_short()` methods
- Built-in risk management, position sizing, and order management

**Web Application (`jesse/__init__.py`)**
- FastAPI application serving both API endpoints and static frontend
- CLI interface using Click for commands like `jesse run`, `jesse install-live`
- WebSocket support for real-time communication

**Trading Modes (`jesse/modes/`)**
- `backtest_mode.py`: Historical strategy testing
- `data_provider.py`: Configuration management
- `optimize_mode/`: Strategy parameter optimization using Optuna
- `import_candles_mode/`: Historical data import from various exchanges

**Services (`jesse/services/`)**
- `broker.py`: Order execution and position management
- `candle.py`: OHLCV data handling
- `metrics.py`: Performance calculation and reporting
- `db.py`: Database operations using Peewee ORM
- `redis.py`: Caching and pub/sub messaging
- `notifier.py`: Alert system (Telegram, Slack, Discord)

**Controllers (`jesse/controllers/`)**
- RESTful API endpoints organized by functionality
- `backtest_controller.py`: Backtesting operations
- `strategy_controller.py`: Strategy management
- `live_controller.py`: Live trading (requires jesse-live plugin)

**Data Models (`jesse/models/`)**
- Database models for candles, trades, orders, positions
- Uses Peewee ORM with PostgreSQL backend

### Key Architecture Patterns

**Plugin System**: Optional `jesse-live` plugin for live trading functionality, detected via `HAS_LIVE_TRADE_PLUGIN` flag.

**Multi-Processing**: Background task management through `jesse.services.multiprocessing.process_manager` for CPU-intensive operations like backtesting and optimization.

**Caching**: Redis-based caching for candle data and computed indicators to improve performance.

**State Management**: Global state stored in `jesse.store` for managing exchanges, positions, orders across strategy execution.

## Strategy Development

### Basic Strategy Structure
```python
from jesse.strategies import Strategy
import jesse.indicators as ta
import jesse.utils as utils

class MyStrategy(Strategy):
    def should_long(self):
        # Entry logic for long positions
        return some_condition
    
    def should_short(self):
        # Entry logic for short positions  
        return some_condition
    
    def go_long(self):
        # Position sizing and order placement
        qty = utils.size_to_qty(self.balance * 0.05, self.price)
        self.buy = qty, self.price
        self.stop_loss = qty, self.price * 0.95
        self.take_profit = qty, self.price * 1.10
    
    def go_short(self):
        # Short position logic
        pass
```

### Key Strategy Features
- Access to 300+ technical indicators via `jesse.indicators` 
- Multi-timeframe data through `self.get_candles()`
- Risk management helpers in `jesse.utils`
- Hyperparameter optimization support via `hyperparameters()` method
- Real-time debugging and logging capabilities

## Data Flow

1. **Data Import**: Historical candles imported via exchange drivers in `import_candles_mode/drivers/`
2. **Strategy Execution**: Strategies process candles chronologically during backtests/live trading
3. **Order Management**: `Broker` service handles order lifecycle and position updates
4. **Metrics Collection**: Performance metrics calculated and stored after strategy execution
5. **Web Interface**: FastAPI serves results and provides real-time monitoring via WebSockets

## Database Schema

- **Candles**: OHLCV data with exchange/symbol/timeframe indexing
- **Trades**: Completed trade records with entry/exit details
- **Orders**: Order history and status tracking
- **Routes**: Strategy-symbol-timeframe-exchange mappings

## Testing Approach

Tests are located in `tests/` directory with comprehensive coverage of:
- Strategy backtesting (`test_backtest.py`)
- Indicators (`test_indicators.py`) 
- Order management (`test_broker.py`, `test_order.py`)
- Data handling (`test_candle_service.py`)
- Utilities (`test_utils.py`, `test_helpers.py`)

## Live Trading

Live trading requires the separate `jesse-live` plugin which adds:
- Real-time data feeds
- Exchange API integrations
- Paper trading mode
- Enhanced monitoring and alerting

The main framework provides the foundation while the plugin handles exchange-specific implementations.