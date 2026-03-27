<div align="center">

<h3><b>quant_sim_backend</b></h3>
<p><b>Real-time quantitative trading simulation API — live prices, order books, and AI-generated market news.</b></p>

<p>
  <a href="#features"><strong>Features</strong></a> ·
  <a href="#api-endpoints"><strong>API Reference</strong></a> ·
  <a href="#tech-stack"><strong>Tech Stack</strong></a> ·
  <a href="#getting-started"><strong>Getting Started</strong></a> ·
  <a href="#related"><strong>Related</strong></a>
</p>

<p>

[![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-async-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

</p>

</div>

---

## Overview

`quant_sim_backend` is the server-side engine for [Quant-Sim](https://github.com/UnripePlum/quant_sim_app) — a paper-trading platform where users practice quantitative strategies without real money. The backend drives every live data feed: prices tick every second via a stochastic generator, orders are matched and persisted in PostgreSQL, and GPT-4 produces market-moving news headlines on a randomized schedule.

## Features

<details>
<summary><b>Real-Time Market Data</b></summary>

- Streams OHLCV candlestick ticks every second over WebSocket
- Seeds 20 minutes of price history on first connection so charts render instantly
- Stochastic price model with configurable delta bounds per symbol

</details>

<details>
<summary><b>Order Management</b></summary>

- Places buy/sell orders with full audit trail (price, quantity, total, timestamp)
- Queries order book per symbol or per user/symbol pair
- Order feed streamed live to all connected clients via `/ws/order`

</details>

<details>
<summary><b>AI-Generated News</b></summary>

- GPT-4 generates fictional market headlines every 20–40 seconds
- Each news item carries a sentiment score (-1 to 1) and impact rating (1–10)
- Broadcast to subscribers via `/ws/news` with automatic fallback on API errors

</details>

<details>
<summary><b>User Accounts & Portfolio Tracking</b></summary>

- Creates users with hashed passwords and email uniqueness enforcement
- Each account starts with ₩100,000,000 virtual balance
- Portfolio and holdings stored as JSON; live account state pushed via `/ws/user/{email}`

</details>

## API Endpoints

### REST

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/backend/user/` | Register a new user |
| `GET` | `/api/backend/user/` | List all users |
| `GET` | `/api/backend/user/{email}` | Get user by email |
| `POST` | `/api/backend/order/` | Place an order |
| `GET` | `/api/backend/order/{symbol}` | Get all orders for a symbol |
| `GET` | `/api/backend/order/{symbol}/user/{user_id}` | Get a user's orders for a symbol |
| `GET` | `/api/backend/stock/list` | List all available stocks |
| `POST` | `/api/backend/stock/init` | Seed initial stock data |

### WebSocket

| Endpoint | Description |
|----------|-------------|
| `ws://…/ws` | Live OHLCV price feed for all symbols |
| `ws://…/ws/news` | AI-generated news headline stream |
| `ws://…/ws/user/{email}` | Live portfolio and balance updates |
| `ws://…/ws/order` | Live order book across all symbols |

> [!NOTE]
> Interactive docs are available at `/api/backend/docs` once the server is running.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | [FastAPI](https://fastapi.tiangolo.com) |
| Language | [Python 3.12](https://python.org) |
| Database | [PostgreSQL 15](https://postgresql.org) via [SQLAlchemy](https://sqlalchemy.org) (async) |
| Async Driver | [asyncpg](https://github.com/MagicStack/asyncpg) |
| Validation | [Pydantic v2](https://docs.pydantic.dev) |
| AI News | [OpenAI GPT-4](https://openai.com) |
| Auth Utilities | [python-jose](https://github.com/mpdavis/python-jose), [passlib](https://passlib.readthedocs.io) |
| Containerization | [Docker](https://docker.com) + [docker-compose](https://docs.docker.com/compose/) |

## Getting Started

### Prerequisites

- Python 3.12+
- Docker and docker-compose
- An OpenAI API key

### 1. Clone and install

```bash
git clone https://github.com/UnripePlum/quant_sim_backend
cd quant_sim_backend
pip install -r requirements.txt
```

### 2. Configure environment

Create a `.env` file in the project root:

```env
DATABASE_URL=postgresql+asyncpg://postgres:password@localhost:5432/quant_sim
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_DB=quant_sim
OPENAI_API_KEY=sk-...
```

### 3. Start the database

```bash
docker-compose up -d
```

### 4. Run the server

```bash
uvicorn app.main:app --reload
```

The API is now live at `http://localhost:8000`. Visit `http://localhost:8000/api/backend/docs` for the interactive Swagger UI.

### 5. Seed stock data

```bash
curl -X POST http://localhost:8000/api/backend/stock/init
```

## Related

- **[quant_sim_app](https://github.com/UnripePlum/quant_sim_app)** — The frontend client that connects to this backend, built with real-time charting and a trading UI.

## License

MIT
