# Smallenberger Options Trading System

An autonomous multi-agent AI system for options trading, built on n8n workflow automation. The system combines real-time news analysis, technical signals, and macro regime detection to generate, execute, and manage options positions across 150+ securities.

**Live execution via Alpaca Markets API** | **Multi-agent coordination** | **Institutional-grade risk management**

---

## System Architecture

### Data Layer
**Inputs:** RSS Feeds (CNBC, Bloomberg, Reuters, Fed) → Yahoo Finance → Perplexity AI → Alpaca Market Data

### Signal Generation
**News Parser** → Publisher tiering (T1/T2/T3) → Symbol extraction → Catalyst classification

**Forward Signals** → Earnings windows, OPEX detection, insider activity, sector cycles

### AI Agents

| Agent | Role | Key Functions |
|-------|------|---------------|
| **Macro Reporter** | Market regime analysis | Cross-asset synthesis, Fed pulse, risk flags |
| **Directional Analyst** | Signal scoring | Bias (-1 to +1), confidence levels, event windows |
| **Trade Executor** | Position entry | Structure selection, sizing, order construction |
| **Portfolio Manager** | Position management | Greeks monitoring, exits, Protected Hold Windows |

### Execution Layer
**Candidate Generation** → Liquidity gates (width ≤12%, BA thresholds) → OPRA construction → Alpaca API

### Logging
Google Sheets: Trade Journal, PM/Executor Comms, Entry/Exit Logs

---

## Core Components

### Signal Generation
- **News Parser**: Processes RSS feeds from 8+ sources with publisher tiering (Reuters/Bloomberg → T1, aggregators → T3)
- **Catalyst Detection**: Identifies earnings windows, insider transactions, price target changes, regulatory events
- **Forward-Looking Signals**: Projects earnings dates, OPEX exposure, sector-specific cycles (EV deliveries, retail holiday)

### AI Agents (Grok-4 / xAI)

| Agent | Role | Key Functions |
|-------|------|---------------|
| **Macro Reporter** | Market regime analysis | Cross-asset synthesis, Fed pulse, risk flag identification |
| **Directional Analyst** | Signal scoring | Bias generation (-1 to +1), confidence levels, event window classification |
| **Trade Executor** | Position entry | Structure selection, sizing, liquidity validation, order construction |
| **Portfolio Manager** | Position management | Greeks monitoring, Protected Hold Windows, exit planning, trim/scale decisions |

### Risk Management
- **Width Gate**: Hard cap at 12% spread width
- **Liquidity Gates**: BA spread thresholds, OI requirements
- **Protected Hold Window (PHW)**: Minimum 3-hour hold on new positions to prevent overtrading
- **Per-Symbol Exposure Cap**: 50% of daily budget maximum per underlying
- **Greeks Aggregation**: Real-time portfolio delta, gamma, theta, vega tracking

### Execution Infrastructure
- **Strategy Support**: Bull/Bear Call Spreads, Bull/Bear Put Spreads, Long/Short Calls & Puts
- **Order Types**: Limit orders, multi-leg (mleg) spreads via Alpaca API
- **Validation**: OPRA symbol construction, price normalization, duplicate order prevention

---

**Coverage:** 150+ securities across tech, semiconductors, fintech, consumer, crypto infrastructure, and international ADRs.

---

## Technical Stack

| Component | Technology |
|-----------|------------|
| Workflow Orchestration | n8n (193 nodes) |
| AI Models | Grok-4 (xAI) |
| Execution | Alpaca Markets API |
| Market Data | Yahoo Finance, Alpaca |
| News/Research | RSS, Perplexity AI |
| Logging | Google Sheets |
| Technical Analysis | Custom multi-timeframe system |

---

## Key Design Decisions

**Why Multi-Agent?**
Single-prompt systems struggle with the complexity of trading decisions. Separating concerns (macro → signals → execution → management) allows each agent to specialize and enables clear accountability when reviewing trades.

**Why Options?**
Defined risk structures (spreads) allow for precise position sizing and risk management. The system can express directional views with capped downside.

**Why n8n?**
Visual workflow design enables rapid iteration on complex multi-step processes. Native support for HTTP nodes, code execution, and AI model integration.

---

*Paper trading only — no real capital deployed.*

---

## Repository Contents

```
├── Smallenberger_Options_Hedge_fund.json   # Complete n8n workflow (importable)
├── README.md                                # This file
└── docs/
    └── architecture.md                      # Detailed component documentation
```

---

## Setup

1. Import `Smallenberger_Options_Hedge_fund.json` into n8n
2. Configure credentials:
   - Alpaca API (paper trading)
   - xAI/Grok API
   - Google Sheets OAuth
   - Perplexity API
3. Create required Google Sheets (Trade Journal, PM/Executor Comms)
4. Activate workflow triggers

---

## Author

**Michael Smallenberger**  
Finance, Fordham University Gabelli School of Business '26  
[LinkedIn](https://www.linkedin.com/in/mike-smalls-smallenberger-4a9121214/) | [Email](mailto:msmallenberger@fordham.edu)

---

## License

This project is shared for educational and portfolio demonstration purposes. The trading logic, prompts, and system architecture represent original work.
