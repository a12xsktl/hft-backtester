# HFT Backtesting Engine and Inventory-Aware Market Making

## Overview

This project implements a simplified high-frequency trading research framework for inventory-aware market making. The system replays historical limit order book data and evaluates market making strategies under deterministic execution assumptions.

The implementation includes:

* a replay-based backtesting engine,
* order placement and cancellation,
* inventory and cash accounting,
* execution simulation,
* baseline market making,
* the Avellaneda–Stoikov model,
* an enhanced inventory-aware strategy using order book and trade-flow signals,
* parameter sweep experiments.

The project was implemented in Python using historical order book and trade data.

---

# Data

Two datasets were used in the project:

1. Limit order book snapshots containing 25 bid and ask levels.
2. Trade data containing aggressive buy and sell executions.

Both datasets were sorted chronologically using timestamps. The order book data was used for replay and execution simulation, while trade data was used for signal construction and flow analysis.

---

# Feature Engineering

Several groups of market microstructure features were constructed from the raw data.

## Basic Features

The following variables were calculated from the top level of the order book:

* best bid,
* best ask,
* mid-price,
* spread,
* microprice,
* short-term volatility,
* top-level imbalance.

The microprice was used as a liquidity-weighted estimate of fair value.

---

## Multi-Level Order Book Features

Liquidity imbalance was computed using multiple order book depths:

* 5 levels,
* 10 levels,
* 20 levels,
* 25 levels.

Both raw and exponentially smoothed versions of these features were tested.

The objective was to measure short-term buying and selling pressure inside the order book.

---

## Trade Flow Features

Additional features were extracted from aggressive trades:

* signed volume,
* signed notional,
* trade imbalance.

Trades were mapped to the most recent order book snapshot using timestamps.

To avoid look-ahead bias, all predictive signals were shifted into the past before evaluating their predictive power.

---

# Signal Construction

Several order book and trade-flow variables were combined into a single alpha signal.

The final signal included:

* short-depth imbalance,
* medium-depth imbalance,
* top-level imbalance,
* trade imbalance.

The resulting signal demonstrated a meaningful positive correlation with future short-term price changes after removing leakage effects.

This signal was later incorporated into the market making strategy to improve quote placement.

---

# Backtesting Engine

A replay-based backtesting engine was implemented to simulate passive market making.

The framework supports:

* limit order placement,
* order cancellation,
* inventory tracking,
* cash accounting,
* mark-to-market PnL,
* turnover calculation.

The execution logic follows the assignment assumption:

orders are executed only when the future market price crosses the submitted quote level.

No probabilistic fill model or queue-priority simulation was used.

---

# Baseline Strategy

The baseline strategy continuously quoted at the current best bid and best ask prices.

This approach generated positive PnL but accumulated very large inventory exposure. The inventory process drifted significantly over time, producing unstable equity dynamics and large directional risk.

As a result, the strategy behaved more like uncontrolled inventory accumulation than a realistic market making system.

---

# Avellaneda–Stoikov Model

The classical Avellaneda–Stoikov framework was implemented to control inventory risk dynamically.

The strategy adjusts its reservation price according to current inventory and volatility conditions. Quotes are placed around this reservation price while limiting excessive inventory accumulation.

Compared to the baseline strategy, the model significantly improved risk control:

* inventory remained bounded,
* inventory volatility dropped sharply,
* equity dynamics became smoother.

However, the strategy remained unprofitable overall. This behavior is consistent with adverse selection effects commonly observed in passive market making.

---

# Enhanced Inventory-Aware Strategy

An enhanced version of the strategy was developed using:

* microprice,
* order book imbalance,
* trade-flow alpha signals.

The alpha signal was incorporated into the reservation price to skew quotes toward the predicted short-term market direction.

The enhanced strategy improved performance relative to the classical Avellaneda–Stoikov model:

* smaller drawdowns,
* lower inventory volatility,
* fewer unnecessary trades,
* improved overall PnL.

Although the strategy remained slightly unprofitable under the current assumptions, the experiments demonstrated that predictive alpha signals can partially mitigate adverse selection.

---

# Simulation Experiments

A parameter sweep was conducted across several model parameters:

* signal strength,
* spread width,
* inventory penalty.

The experiments produced several consistent observations:

1. Wider spreads improved performance by reducing adverse selection.
2. Moderate alpha strength performed better than overly aggressive skewing.
3. Excessive inventory penalties reduced profitability.
4. Inventory-aware quoting substantially stabilized the trading process.

The best-performing configuration used:

* spread = 3 ticks,
* signal weight = 1.0,
* inventory penalty = 0.005.

---

# Results Summary

The experiments demonstrated a clear trade-off between profitability and inventory control.

The baseline strategy produced positive PnL but accumulated excessive directional inventory.

The classical Avellaneda–Stoikov model significantly improved inventory stability but generated negative PnL due to adverse selection.

The enhanced strategy improved both profitability and stability by incorporating predictive order flow and imbalance signals into the quoting process.

---

# Limitations

Several important market microstructure effects were intentionally simplified:

* queue position was not modeled,
* latency effects were ignored,
* fills were deterministic,
* transaction costs and exchange fees were excluded,
* probabilistic execution was not implemented.

These simplifications make the framework computationally efficient but less realistic than production-grade HFT simulators.

---

# Future Work

Potential future improvements include:

* queue-aware execution modeling,
* latency simulation,
* probabilistic fill estimation,
* machine learning alpha models,
* adaptive spread optimization,
* reinforcement learning for quote placement.

---

# Conclusion

This project demonstrates a complete workflow for inventory-aware high-frequency market making research.

The implementation includes:

* historical replay,
* inventory-aware quoting,
* signal engineering,
* execution simulation,
* parameter optimization.

The experiments show that inventory management alone is insufficient for profitable market making. Predictive alpha signals are necessary to reduce adverse selection and improve quote quality.

Even though the final strategy remained slightly unprofitable, the framework successfully reproduced several realistic market making behaviors and provides a solid foundation for further quantitative research.
