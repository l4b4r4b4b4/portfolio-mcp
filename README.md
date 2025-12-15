# portfolio-mcp

A portfolio analysis MCP server powered by [mcp-refcache](https://github.com/l4b4r4b4b4/mcp-refcache) for building AI agent tools that handle financial data efficiently.

[![Tests](https://github.com/l4b4r4b4b4/portfolio-mcp/actions/workflows/ci.yml/badge.svg)](https://github.com/l4b4r4b4b4/portfolio-mcp/actions/workflows/ci.yml)
[![Coverage](https://img.shields.io/badge/coverage-81%25-green)](https://github.com/l4b4r4b4b4/portfolio-mcp)
[![Python](https://img.shields.io/badge/python-3.12+-blue.svg)](https://www.python.org/downloads/)
[![PyPI](https://img.shields.io/pypi/v/portfolio-mcp)](https://pypi.org/project/portfolio-mcp/)
[![Docker](https://img.shields.io/badge/docker-ghcr.io-blue)](https://github.com/l4b4r4b4b4/portfolio-mcp/pkgs/container/portfolio-mcp)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Features

- **Portfolio Management**: Create, read, update, delete portfolios with persistent storage
- **Data Sources**: Yahoo Finance (stocks/ETFs), CoinGecko (crypto), Synthetic (GBM simulation)
- **Analysis Tools**: Returns, volatility, Sharpe ratio, Sortino ratio, VaR, drawdowns, correlations
- **Optimization**: Efficient Frontier, Monte Carlo simulation, weight optimization
- **Reference-Based Caching**: Large datasets cached via mcp-refcache to avoid context bloat

## Installation

### Option 1: PyPI (Recommended)

```bash
# Install and run with uvx (no installation needed)
uvx portfolio-mcp stdio

# Or install globally with pip
pip install portfolio-mcp
portfolio-mcp stdio
```

### Option 2: Docker

```bash
# Pull and run the latest image
docker pull ghcr.io/l4b4r4b4b4/portfolio-mcp:latest

# Run with docker-compose
docker compose up -d

# Or run directly
docker run -p 8000:8000 ghcr.io/l4b4r4b4b4/portfolio-mcp:latest
```

### Option 3: From Source

```bash
# Clone the repository
git clone https://github.com/l4b4r4b4b4/portfolio-mcp
cd portfolio-mcp

# Install dependencies with uv
uv sync

# Run the server
uv run portfolio-mcp stdio
```

## Quick Start

### Connect to Claude Desktop

Add to your Claude Desktop configuration:

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`
**Linux**: `~/.config/claude/claude_desktop_config.json`

```json
{
  "mcpServers": {
    "portfolio-mcp": {
      "command": "uvx",
      "args": ["portfolio-mcp", "stdio"]
    }
  }
}
```

Or use Docker:

```json
{
  "mcpServers": {
    "portfolio-mcp": {
      "command": "npx",
      "args": ["mcp-remote", "http://localhost:8000/mcp"]
    }
  }
}
```

### Basic Usage

Once connected, you can use natural language to:

```
"Create a portfolio called 'tech_stocks' with AAPL, GOOG, and MSFT"
"Analyze the returns and volatility of my tech_stocks portfolio"
"Optimize my portfolio for maximum Sharpe ratio"
"Show me the efficient frontier with 20 points"
"Compare my portfolios by Sharpe ratio"
```

### Quick Showcase Demo

Try this 5-step demo to see portfolio-mcp in action:

```
1. Create a tech portfolio with AAPL, GOOG, MSFT using 1 year of Yahoo Finance data
   (Expected: ~27% return, ~24% volatility, Sharpe ratio ~1.05)

2. Optimize the tech_stocks portfolio for maximum Sharpe ratio
   (Watch optimal weights improve risk-adjusted returns)

3. Show me the efficient frontier with 30 points for tech_stocks
   (Visualize all optimal portfolios and identify the sweet spot)

4. Get comprehensive metrics for tech_stocks including VaR and Sortino
   (Understand downside risk and tail risk metrics)

5. Create a crypto portfolio with BTC, ETH, SOL and compare it with tech_stocks
   (See how different asset classes perform on risk/return metrics)
```

This demonstrates portfolio creation, optimization, risk analysis, and comparison—all through natural language!

## Available Tools

### Portfolio Management (6 tools)
- `create_portfolio` - Create a new portfolio with symbols and weights
- `get_portfolio` - Retrieve portfolio details and metrics
- `list_portfolios` - List all stored portfolios
- `delete_portfolio` - Remove a portfolio
- `update_portfolio_weights` - Modify portfolio weights
- `clone_portfolio` - Create a copy with optional new weights

### Analysis Tools (8 tools)
- `get_portfolio_metrics` - Comprehensive metrics (return, volatility, Sharpe, Sortino, VaR)
- `get_returns` - Daily, log, or cumulative returns
- `get_correlation_matrix` - Asset correlation analysis
- `get_covariance_matrix` - Variance-covariance structure
- `get_individual_stock_metrics` - Per-asset statistics
- `get_drawdown_analysis` - Maximum drawdown and recovery analysis
- `compare_portfolios` - Side-by-side portfolio comparison

### Optimization Tools (4 tools)
- `optimize_portfolio` - Optimize weights (max Sharpe, min volatility, target return/vol)
- `get_efficient_frontier` - Generate efficient frontier curve
- `run_monte_carlo` - Monte Carlo simulation for portfolio analysis
- `apply_optimization` - Apply optimization and update stored portfolio

### Data Tools (8 tools)
- `generate_price_series` - Generate synthetic GBM price data
- `generate_portfolio_scenarios` - Create multiple scenario datasets
- `get_sample_portfolio_data` - Get sample data for testing
- `get_trending_coins` - Trending cryptocurrencies from CoinGecko
- `search_crypto_coins` - Search for crypto assets
- `get_crypto_info` - Detailed cryptocurrency information
- `list_crypto_symbols` - Available crypto symbol mappings
- `get_cached_result` - Retrieve cached large results by reference ID

## Architecture

```
portfolio-mcp/
├── app/
│   ├── __init__.py
│   ├── __main__.py      # Typer CLI entry point
│   ├── config.py        # Pydantic settings
│   ├── server.py        # FastMCP server setup
│   ├── storage.py       # RefCache-based portfolio storage
│   ├── models.py        # Pydantic models for I/O
│   ├── data_sources.py  # Yahoo Finance + CoinGecko APIs
│   └── tools/           # MCP tool implementations
│       ├── portfolio.py
│       ├── analysis.py
│       ├── optimization.py
│       └── data.py
└── tests/               # 163 tests, 81% coverage
```

## Reference-Based Caching

This server uses [mcp-refcache](https://github.com/l4b4r4b4b4/mcp-refcache) to handle large results efficiently:

1. **Large results are cached** - When a tool returns data that exceeds the preview size, it's stored in the cache
2. **References are returned** - The tool returns a `ref_id` and a preview/sample of the data
3. **Full data on demand** - Use `get_cached_result(ref_id=...)` to retrieve the complete data

This prevents context window bloat when working with large datasets like price histories or Monte Carlo simulations.

## Development

### Prerequisites

- Python 3.12+
- [uv](https://docs.astral.sh/uv/) (recommended) or pip

### Setup

```bash
# Clone and install
git clone https://github.com/l4b4r4b4b4/portfolio-mcp
cd portfolio-mcp
uv sync

# Run tests
uv run pytest --cov

# Lint and format
uv run ruff check .
uv run ruff format .
```

### Running Locally

```bash
# stdio mode (for MCP clients)
uv run portfolio-mcp stdio

# SSE mode (for web clients)
uv run portfolio-mcp sse --port 8080

# Streamable HTTP mode
uv run portfolio-mcp streamable-http --port 8080
```

## Configuration

Environment variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `LOG_LEVEL` | Logging level | `INFO` |
| `CACHE_TTL` | Default cache TTL in seconds | `3600` |

## License

MIT License - see [LICENSE](LICENSE) for details.

## Related Projects

- [mcp-refcache](https://github.com/l4b4r4b4b4/mcp-refcache) - Reference-based caching for MCP servers
- [fastmcp-template](https://github.com/l4b4r4b4b4/fastmcp-template) - Template this project was built from
- [FinQuant](https://github.com/fmilthaler/FinQuant) - Financial portfolio analysis library
