# Bajaj Broking – Simplified Trading SDK (Backend Simulation)

## Overview

This project implements a **simplified trading backend and SDK simulation**, inspired by real-world online stock broking platforms such as Bajaj Broking.

The system exposes REST APIs that allow users to:
- View tradable financial instruments
- Place BUY / SELL orders (MARKET and LIMIT)
- Check order status
- View executed trades
- Fetch portfolio holdings

The implementation focuses on **backend design, API structure, and trading workflow simulation**.  
No real market or exchange integration is involved.

---

## Tech Stack

- **Language**: Python 3.10+
- **Framework**: FastAPI
- **Dependency Management**: Poetry
- **Data Storage**: In-memory (Python data structures)
- **Testing**: Pytest
- **API Documentation**: Swagger / OpenAPI (auto-generated)

---

## Assumptions

- Single mocked user authentication using an API key
- Orders execute immediately if conditions are satisfied (MARKET orders execute instantly, LIMIT orders execute if price conditions are met)
- No partial fills or order matching engine
- No persistence across server restarts (all data stored in-memory)
- Selling more than available holdings is not allowed
- Portfolio is updated based on executed trades
- Instrument prices are static and predefined

---

## Setup Instructions

### Prerequisites
- Python 3.10 or higher
- Poetry installed ([Installation Guide](https://python-poetry.org/docs/#installation))

### Install Dependencies
```bash
poetry install
```

### Run the Server
```bash
poetry run uvicorn app.main:app --reload
```

The server will start at:
```
http://127.0.0.1:8000
```

---

## API Documentation

Interactive Swagger UI is available at:
```
http://127.0.0.1:8000/docs
```

Alternative ReDoc documentation:
```
http://127.0.0.1:8000/redoc
```

---

## Authentication

All APIs require the following header:

```
X-API-KEY: test-api-key
```

---

## API Endpoints

### 1. Fetch Instruments
**Endpoint**: `GET /api/v1/instruments`

**Description**: Retrieves the list of tradable financial instruments.

**Response Example**:
```json
[
  {
    "symbol": "RELIANCE",
    "exchange": "NSE",
    "instrument_type": "EQUITY",
    "last_traded_price": 2450.75
  },
  {
    "symbol": "TCS",
    "exchange": "NSE",
    "instrument_type": "EQUITY",
    "last_traded_price": 3567.50
  }
]
```

---

### 2. Place Order
**Endpoint**: `POST /api/v1/orders`

**Description**: Places a new buy or sell order.

**Request Body**:
```json
{
  "symbol": "RELIANCE",
  "side": "BUY",
  "order_type": "MARKET",
  "quantity": 10,
  "price": 2450.00
}
```

**Fields**:
- `symbol` (string, required): Trading symbol
- `side` (string, required): "BUY" or "SELL"
- `order_type` (string, required): "MARKET" or "LIMIT"
- `quantity` (integer, required): Must be > 0
- `price` (float, optional): Required for LIMIT orders

**Response Example**:
```json
{
  "order_id": "ORD-1234567890",
  "symbol": "RELIANCE",
  "side": "BUY",
  "order_type": "MARKET",
  "quantity": 10,
  "price": 2450.75,
  "status": "EXECUTED",
  "created_at": "2026-01-09T10:30:00Z"
}
```

---

### 3. Fetch Order Status
**Endpoint**: `GET /api/v1/orders/{order_id}`

**Description**: Retrieves the status of a specific order.

**Response Example**:
```json
{
  "order_id": "ORD-1234567890",
  "symbol": "RELIANCE",
  "side": "BUY",
  "order_type": "MARKET",
  "quantity": 10,
  "price": 2450.75,
  "status": "EXECUTED",
  "created_at": "2026-01-09T10:30:00Z"
}
```

**Order Status Values**:
- `NEW`: Order created but not yet placed
- `PLACED`: Order placed and awaiting execution
- `EXECUTED`: Order successfully executed
- `CANCELLED`: Order cancelled

---

### 4. Fetch Trades
**Endpoint**: `GET /api/v1/trades`

**Description**: Retrieves all executed trades for the user.

**Response Example**:
```json
[
  {
    "trade_id": "TRD-9876543210",
    "order_id": "ORD-1234567890",
    "symbol": "RELIANCE",
    "side": "BUY",
    "quantity": 10,
    "price": 2450.75,
    "executed_at": "2026-01-09T10:30:01Z"
  }
]
```

---

### 5. Fetch Portfolio
**Endpoint**: `GET /api/v1/portfolio`

**Description**: Retrieves current portfolio holdings.

**Response Example**:
```json
[
  {
    "symbol": "RELIANCE",
    "quantity": 10,
    "average_price": 2450.75,
    "current_value": 24507.50
  }
]
```

---

## Sample Usage

### Using cURL

#### Fetch Instruments
```bash
curl -X GET "http://127.0.0.1:8000/api/v1/instruments" \
  -H "X-API-KEY: test-api-key"
```

#### Place a Market Order
```bash
curl -X POST "http://127.0.0.1:8000/api/v1/orders" \
  -H "X-API-KEY: test-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "symbol": "RELIANCE",
    "side": "BUY",
    "order_type": "MARKET",
    "quantity": 10
  }'
```

#### Fetch Order Status
```bash
curl -X GET "http://127.0.0.1:8000/api/v1/orders/ORD-1234567890" \
  -H "X-API-KEY: test-api-key"
```

### Using PowerShell

#### Fetch Instruments
```powershell
Invoke-RestMethod -Uri "http://127.0.0.1:8000/api/v1/instruments" `
  -Headers @{ 'X-API-KEY' = 'test-api-key' }
```

#### Place a Market Order
```powershell
$body = @{
  symbol = "RELIANCE"
  side = "BUY"
  order_type = "MARKET"
  quantity = 10
} | ConvertTo-Json

Invoke-RestMethod -Uri "http://127.0.0.1:8000/api/v1/orders" `
  -Method Post `
  -Headers @{ 'X-API-KEY' = 'test-api-key'; 'Content-Type' = 'application/json' } `
  -Body $body
```

---

## Testing

Run unit tests:
```bash
poetry run pytest
```

Run tests with coverage:
```bash
poetry run pytest --cov=app --cov-report=html
```

---

## Project Structure

```
.
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI application entry point
│   ├── routes/              # API route handlers
│   ├── models/              # Pydantic models and schemas
│   ├── services/            # Business logic layer
│   └── database/            # In-memory data storage
├── tests/                   # Unit tests
├── pyproject.toml           # Poetry configuration
├── README.md                # This file
└── .gitignore
```

---

## Assignment Mapping

| Requirement | Status |
|------------|--------|
| Instrument APIs | ✅ Implemented |
| Order placement (BUY/SELL) | ✅ Implemented |
| MARKET & LIMIT orders | ✅ Implemented |
| Order status tracking | ✅ Implemented |
| Trade execution simulation | ✅ Implemented |
| Portfolio holdings | ✅ Implemented |
| RESTful API design | ✅ Implemented |
| Error handling | ✅ Implemented |
| Swagger documentation | ✅ Implemented |
| Unit tests | ✅ Implemented |

---

## Error Handling

The API returns appropriate HTTP status codes:

- `200 OK`: Successful request
- `201 Created`: Resource created successfully
- `400 Bad Request`: Invalid input or validation error
- `401 Unauthorized`: Missing or invalid API key
- `404 Not Found`: Resource not found
- `500 Internal Server Error`: Server error

Error Response Format:
```json
{
  "detail": "Error message description"
}
```

---

## Future Enhancements

- Order cancellation functionality
- Partial order execution
- Persistent storage (SQLite/PostgreSQL)
- Python client SDK wrapper
- Basic frontend UI
- WebSocket support for real-time updates
- Advanced order types (Stop-Loss, Trailing Stop)
- Multi-user support with proper authentication

---

## Reference

This project was developed as part of a backend engineering assignment based on the Bajaj Broking API specification:
- [Bajaj Broking API Documentation](https://apitrading.bajajbroking.in/)

---

## Author

Developed as part of a backend engineering assignment to demonstrate trading system design and REST API implementation.

---

## License

This project is for educational and assignment purposes only.
