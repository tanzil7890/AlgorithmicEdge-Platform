# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Spring Boot application that implements high-frequency trading (HFT) algorithms for educational purposes. The system demonstrates core algorithmic trading concepts including signal generation, risk management, and order execution through a REST API interface.

**Important**: This is an educational implementation, not a production-ready HFT system. Real HFT requires ultra-low latency hardware, co-location, and microsecond-level timing precision.

## Build and Development Commands

### Essential Commands
```bash
# Build the application
./mvnw clean package

# Run the application  
java -jar target/hft-application-0.0.1-SNAPSHOT.jar
# OR use Maven wrapper
./mvnw spring-boot:run

# Run tests
./mvnw test

# Clean build artifacts
./mvnw clean
```

### Maven Wrapper
The project uses Maven wrapper (`mvnw`/`mvnw.cmd`) so no local Maven installation is required. The build uses Java 21.

## High-Level Architecture

### Core Components
The application follows a multi-threaded architecture with these main components running concurrently:

1. **HFTAlgorithm** (`core/HFTAlgorithm.java`) - Main orchestrator that manages:
   - Market data processing thread
   - Signal generation thread  
   - Order management thread
   - Risk management thread

2. **SignalGenerator** (`core/signal/SignalGenerator.java`) - Implements three trading strategies:
   - Statistical Arbitrage (40% weight)
   - Mean Reversion (40% weight) 
   - Momentum (20% weight)

3. **RiskManager** (`core/risk/RiskManager.java`) - Enforces limits:
   - Maximum position size
   - Maximum order size
   - Daily loss limits
   - Triggers emergency shutdown if limits exceeded

4. **OrderManager** (`core/execution/OrderManager.java`) - Handles order lifecycle:
   - Order submission and cancellation
   - Position tracking and P&L calculation

### Data Flow
1. Market data flows into the algorithm via `updateMarketData()` or simulated feeds
2. Data is stored in order books and historical queues (fixed-size with lookback period)
3. Signal generator analyzes data and produces combined trading signals
4. If signal exceeds threshold, orders are created and submitted
5. Risk manager continuously monitors positions and P&L
6. Order manager tracks active orders and updates positions

### Configuration Parameters
Key algorithm parameters are configurable via `application.properties`:
- `algorithm.maxPositionSize` - Maximum position value per symbol
- `algorithm.maxOrderSize` - Maximum individual order size
- `algorithm.maxDailyLoss` - Daily loss limit triggering emergency shutdown
- `algorithm.lookbackPeriod` - Historical data window size
- `algorithm.signalThreshold` - Minimum signal strength to trade

### Default Symbols
The system initializes with three test symbols:
- BTC-USD (Bitcoin)
- ETH-USD (Ethereum) 
- SOL-USD (Solana)

## REST API Structure

The algorithm is controlled via REST endpoints in `AlgorithmController`:

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/algorithm/start` | POST | Start the trading algorithm |
| `/api/algorithm/stop` | POST | Stop the trading algorithm |
| `/api/algorithm/status` | GET | Check if algorithm is running |
| `/api/algorithm/positions` | GET | Get current positions and P&L |
| `/api/algorithm/orderbooks` | GET | Get current order book data |
| `/api/algorithm/performance` | GET | Get performance metrics |
| `/api/algorithm/updatemarket` | POST | Update market data (for testing) |
| `/api/algorithm/addsymbol` | POST | Add new trading symbol |

## Key Implementation Details

### Threading Model
- Uses `ExecutorService` with fixed thread pool (4 threads)
- Each core component runs in its own thread with different frequencies:
  - Market data processor: 1ms sleep
  - Signal generator: 5ms sleep  
  - Order manager: 10ms sleep
  - Risk manager: 100ms sleep

### Data Structures
- `ConcurrentHashMap` for thread-safe shared state
- `PriorityQueue` for historical market data with fixed size
- `AtomicBoolean` and `AtomicLong` for performance counters

### Risk Controls
- Position limits enforced per symbol
- Daily P&L monitoring with automatic shutdown
- Emergency shutdown procedure that cancels all orders and closes positions

### Testing and Market Data
- For testing, use `/updatemarket` endpoint to feed market data
- Algorithm includes basic market data simulation if no real feeds are connected
- Service layer (`AlgorithmService`) provides abstraction over core algorithm

## Development Notes

### Technology Stack
- Spring Boot 3.4.5
- Java 21
- Spring Web, JPA, Actuator
- H2 database (runtime scope)
- Lombok for boilerplate reduction

### Package Structure
- `controller/` - REST API endpoints
- `core/` - Core algorithm logic
  - `execution/` - Order management
  - `risk/` - Risk management
  - `signal/` - Signal generation
- `model/` - Data models (MarketTick, Order, OrderBook, Position)
- `service/` - Service layer abstraction

### Performance Monitoring
The algorithm tracks several metrics accessible via `/performance` endpoint:
- Average processing latency
- Message count and order count
- Daily P&L
- Total position value