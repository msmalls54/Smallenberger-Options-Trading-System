# System Architecture

## Agent Coordination Model

The system uses a PM/Executor separation model inspired by institutional trading desks:

### Trade Executor Agent
**Responsibility**: Opening new positions

The Executor receives macro context, directional signals, and portfolio state. It then:
1. Evaluates candidate structures against liquidity/width gates
2. Sizes positions within budget constraints
3. Constructs order payloads with ticker-only intents (no OPRA in prompts)
4. Defines exit plans and Protected Hold Window parameters

**Key Constraints**:
- Daily budget cap ($75k max deployment)
- Per-symbol cap (50% of daily budget)
- Width gate (≤12% spread width)
- No naked short calls
- DTE ≤2 requires catalyst confirmation

### Portfolio Manager Agent
**Responsibility**: Managing open positions

The PM reviews existing positions and decides on:
1. Trim/scale based on P&L targets
2. Exit based on technical invalidation
3. Hold enforcement during Protected Hold Windows
4. Greeks-based adjustments

**Protected Hold Window (PHW)**:
- 3-hour minimum hold on new positions
- Prevents overtrading and whipsaw exits
- Override only with explicit justification + catalyst confirmation

---

## Signal Generation Pipeline

### News Processing
```
Raw RSS Items
    │
    ▼
Publisher Tier Classification
    │ T1: reuters, bloomberg, wsj, ft, cnbc, sec.gov
    │ T2: theverge, benzinga, barrons, marketbeat
    │ T3: newser, aggregators
    │
    ▼
Symbol Extraction
    │ • URL parsing (quote/symbol patterns)
    │ • Strict ticker hits ($AAPL, NASDAQ:AAPL)
    │ • Alias matching (Chipotle → CMG)
    │
    ▼
Catalyst Classification
    │ • PT↑/PT↓ (price target changes)
    │ • Rating upgrades/downgrades
    │ • Insider buy/sell
    │ • Guidance changes
    │ • Product/regulatory/M&A events
    │
    ▼
Forward Signal Construction
    │ • Earnings window estimation (+91d from last)
    │ • OPEX proximity detection
    │ • Sector cycle mapping
```

### Technical Analysis
Multi-timeframe analysis across:
- **Daily**: Trend direction, MA positioning
- **Hourly**: Short-term momentum
- **Weekly**: Structural regime

Outputs include:
- Direction bias
- Support/resistance levels
- Expected move calculations
- MA crossover signals

---

## Candidate Generation

### Strategy Selection Logic

The system evaluates 8 strategy types per expiration:

| Strategy | Direction | Type | Risk Profile |
|----------|-----------|------|--------------|
| Bull Put Spread | Bullish | Credit | Defined |
| Bear Call Spread | Bearish | Credit | Defined |
| Bull Call Spread | Bullish | Debit | Defined |
| Bear Put Spread | Bearish | Debit | Defined |
| Long Call | Bullish | Debit | Premium at risk |
| Long Put | Bearish | Debit | Premium at risk |
| Short Put (CSP) | Bullish | Credit | Cash-secured |
| Short Call | Bearish | Credit | Spread-only (no naked) |

### Scoring Weights
```javascript
weights = {
  Liquidity: 25,  // BA spread, volume, OI
  Edge: 35,       // Signal alignment
  Profit: 30,     // R:R ratio
  PortFit: 7,     // DTE preference
  RiskPen: 3      // Structure risk penalty
}
```

### Dynamic Thresholds

Thresholds adjust based on:
- **Volatility regime**: HIGH_VOL loosens BA gates, tightens R:R floors
- **Liquidity score**: Low liquidity → stricter BA requirements
- **Risk appetite**: Higher appetite → looser R:R, wider BA tolerance

---

## Order Construction

### Flow
```
Executor Intent (ticker-only)
    │
    │ { underlying: "AMD", structure: "Bull Call Spread",
    │   exp: "2025-10-10", strikes: [215, 230],
    │   side: "debit", target_limit: 3.96, qty_hint: 26 }
    │
    ▼
OPRA Symbol Builder
    │ AMD251010C00215000
    │ AMD251010C00230000
    │
    ▼
Alpaca Order Payload
    │ { order_class: "mleg", qty: "26", type: "limit",
    │   limit_price: "3.96", time_in_force: "day",
    │   legs: [...] }
    │
    ▼
HTTP POST → Alpaca API
    │
    ▼
Fill Status Polling
```

### Validation Rules
- No OPRA symbols in agent prompts (prevents formatting errors)
- Price normalization to 2 decimal places
- Quantity as string (Alpaca requirement)
- Unique client_order_id generation

---

## Greeks Aggregation

The Portfolio Impact Calculator aggregates Greeks across all positions:

### Vendor Greeks (Preferred)
Direct from Alpaca snapshot data when available:
```
Net Δ = Σ (leg.delta × qty × side × CM)
Net Γ = Σ (leg.gamma × qty × side × CM)
Net Θ = Σ (leg.theta × qty × side × CM)
Net ν = Σ (leg.vega × qty × side × CM)
```
Where CM = contract multiplier (100)

### Black-Scholes Fallback
When vendor Greeks unavailable, computed from:
- Spot price (from TA)
- Strike
- Time to expiration
- Implied volatility
- Risk-free rate (configurable)

---

## Logging Architecture

### Google Sheets Integration

**Trade Journal**
- Entry/exit records with rationale
- Strategy, direction, risk rating
- Exit plan documentation

**PM/Executor Comms**
- Inter-agent communication
- Hold metadata
- Override justifications

**PM Actions**
- Internal notes
- Severity levels
- Recheck timestamps

This separation enables:
1. Audit trail for all decisions
2. Pattern analysis for strategy refinement
3. Clear accountability between agents
