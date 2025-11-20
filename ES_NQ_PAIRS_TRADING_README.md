# ES/NQ Pairs Trading Strategy

## Overview

This Pine Script v6 strategy implements a **statistical arbitrage** approach to trade the spread between ES (S&P 500 E-mini) and NQ (Nasdaq 100 E-mini) futures contracts. The strategy uses mean reversion principles to identify and exploit temporary divergences in the price relationship between these two highly correlated instruments.

## Strategy Concept

### What is Pairs Trading?

Pairs trading is a market-neutral strategy that involves taking simultaneous long and short positions in two correlated assets. When the spread between them diverges from its historical mean, we expect it to revert, creating a profit opportunity.

### ES/NQ Relationship

- **ES1!** (S&P 500 E-mini): Broader market exposure across 500 large-cap US stocks
- **NQ1!** (Nasdaq 100 E-mini): Tech-heavy index with 100 largest non-financial Nasdaq stocks

These instruments are highly correlated but can diverge due to:
- Sector rotation (tech vs. broad market)
- Market regime changes
- Risk-on/risk-off sentiment shifts
- Economic data releases

## How It Works

### 1. Spread Calculation

The strategy calculates a normalized spread between ES and NQ:

```
Spread = ES Price - (NQ Price × Ratio)
```

Where the ratio normalizes NQ to ES scale to account for different price levels.

### 2. Z-Score Calculation

The spread is converted to a z-score to measure standard deviations from the mean:

```
Z-Score = (Current Spread - Mean Spread) / Standard Deviation
```

### 3. Entry Signals

- **Long Spread** (Long ES, Short NQ): Enter when z-score < -2.0 (spread is oversold)
  - Interpretation: ES is relatively cheap compared to NQ
  - Expectation: Spread will widen (ES outperforms NQ)

- **Short Spread** (Short ES, Long NQ): Enter when z-score > +2.0 (spread is overbought)
  - Interpretation: ES is relatively expensive compared to NQ
  - Expectation: Spread will narrow (NQ outperforms ES)

### 4. Exit Signals

Positions are exited when:
- **Mean Reversion**: Z-score returns within ±0.5 (profit target)
- **Stop Loss**: Z-score exceeds ±4.0 (loss limit)
- **Time-Based**: Maximum bars in trade reached (optional)

## Parameters

### Core Parameters

| Parameter | Default | Range | Description |
|-----------|---------|-------|-------------|
| **Lookback Period** | 100 | 20-500 | Bars used to calculate spread mean and standard deviation |
| **Entry Threshold** | 2.0 | 0.5-5.0 | Z-score threshold to enter trades (standard deviations) |
| **Exit Threshold** | 0.5 | 0.0-3.0 | Z-score threshold to exit trades |
| **Stop Loss Threshold** | 4.0 | 2.0-10.0 | Z-score threshold for stop loss |

### Normalization Methods

1. **Ratio (Recommended)**: `ES - (NQ × Ratio)`
   - Most stable for different price levels
   - Ratio calculated from historical averages

2. **Z-Score Individual**: Normalize each instrument separately
   - Good for comparing relative performance
   - More sensitive to individual instrument moves

3. **Price Difference**: Direct subtraction (not recommended)
   - Doesn't account for scale differences
   - May produce unreliable signals

### Risk Management

- **Enable Stop Loss**: Toggle stop loss functionality
- **Enable Time-Based Exit**: Exit after max bars in trade
- **Max Bars in Trade**: Maximum holding period (default: 50)

## Installation & Usage

### Step 1: Add to TradingView

1. Open TradingView Pine Editor
2. Copy the contents of `ES_NQ_Pairs_Trading_Strategy.pine`
3. Paste into Pine Editor
4. Click "Add to Chart"

### Step 2: Chart Setup

**Recommended Setup:**
- **Primary Chart**: Any liquid instrument (ES1!, NQ1!, or SPY for example)
- **Timeframe**: 15min, 30min, 1H, or 4H (strategy works on various timeframes)
- **Strategy**: Added as indicator to the chart

**Important:** The strategy will automatically fetch ES1! and NQ1! data regardless of what's on your main chart.

### Step 3: Configure Parameters

Adjust parameters in the strategy settings:

**Conservative Settings (Lower Frequency, Higher Win Rate):**
- Entry Threshold: 2.5
- Exit Threshold: 0.5
- Stop Loss: 5.0
- Lookback: 150

**Aggressive Settings (Higher Frequency, More Trades):**
- Entry Threshold: 1.5
- Exit Threshold: 0.3
- Stop Loss: 3.0
- Lookback: 50

### Step 4: Backtest

1. Use TradingView's Strategy Tester to analyze performance
2. Review key metrics:
   - Net Profit
   - Win Rate
   - Profit Factor
   - Max Drawdown
   - Sharpe Ratio
3. Adjust parameters based on backtest results

## Visualization Guide

### Main Panel (Below Price Chart)

- **Blue Line**: Spread Z-Score (current position relative to mean)
- **Red/Green Dashed Lines**: Entry bands (±2.0 std dev)
- **Orange/Lime Circles**: Exit bands (±0.5 std dev)
- **Maroon/Navy Crosses**: Stop loss bands (±4.0 std dev)
- **Gray Line**: Mean (zero line)

### Trade Indicators

- **Green Background**: Currently in Long Spread position
- **Red Background**: Currently in Short Spread position
- **"LONG SPREAD" Label**: Entry point for long spread
- **"SHORT SPREAD" Label**: Entry point for short spread
- **X Markers**: Exit points

### Statistics Table (Top Right)

Displays real-time information:
- Current ES and NQ prices
- Current spread z-score
- Position status (Long/Short/Flat)
- Bars in current trade

## Understanding the Results

### Strategy Performance Metrics

When backtesting, focus on:

1. **Profit Factor** (>1.5 is good): Total profit / Total loss
2. **Win Rate** (50-70% typical): Percentage of winning trades
3. **Max Drawdown** (<20% ideal): Largest peak-to-trough decline
4. **Sharpe Ratio** (>1.0 good): Risk-adjusted returns
5. **Number of Trades**: Ensure sufficient sample size (>30 trades)

### Interpreting Signals

**Long Spread Signal** means:
- The z-score has dropped below -2.0
- ES is underperforming NQ (spread is compressed)
- Strategy expects ES to outperform NQ going forward
- In practice: Buy ES contracts, Sell NQ contracts

**Short Spread Signal** means:
- The z-score has risen above +2.0
- ES is outperforming NQ (spread is extended)
- Strategy expects NQ to outperform ES going forward
- In practice: Sell ES contracts, Buy NQ contracts

## Implementation Considerations

### For Live Trading

**This strategy shows the SPREAD as a synthetic instrument.** To trade this live, you need to:

1. **Determine Position Sizes**:
   - Calculate appropriate contract ratios based on:
     - Dollar value per point (ES: $50/point, NQ: $20/point)
     - Price levels (ES ~4000-5000, NQ ~15000-17000)
   - Common ratio: 1 ES contract to 1 NQ contract
   - Adjust based on beta and volatility

2. **Execute Both Legs**:
   - When signal says "Long Spread": Buy ES + Sell NQ simultaneously
   - When signal says "Short Spread": Sell ES + Buy NQ simultaneously
   - Use bracket orders or basket orders to minimize execution risk

3. **Monitor Slippage**:
   - Both legs must be filled for proper pairs trading
   - Liquid instruments like ES/NQ have tight spreads
   - Consider using limit orders during high volatility

4. **Margin Requirements**:
   - Futures require margin (~$12,000 per ES, ~$16,000 per NQ)
   - Pairs trading may qualify for portfolio margin (reduced requirements)
   - Check with your broker for specific requirements

### Commissions & Costs

Strategy includes:
- **Commission**: $4.24 per round trip (adjust based on your broker)
- **Slippage**: 2 ticks (0.25 points for ES, 0.25 points for NQ)

Update these values in the strategy settings to match your actual costs.

## Risk Warnings

### ⚠️ Important Disclaimers

1. **Correlation Risk**: ES and NQ correlation can break down during market stress
2. **Tail Risk**: Extreme divergences (black swan events) can cause large losses
3. **Execution Risk**: Must fill both legs simultaneously; partial fills create directional exposure
4. **Leverage Risk**: Futures use significant leverage; losses can exceed initial capital
5. **Model Risk**: Past performance does not guarantee future results
6. **Data Risk**: Historical spread relationships may not persist

### Best Practices

- ✅ Start with small position sizes
- ✅ Always use stop losses
- ✅ Monitor correlation regularly
- ✅ Test thoroughly in paper trading
- ✅ Understand futures margin requirements
- ✅ Have risk management plan
- ❌ Don't over-leverage
- ❌ Don't ignore stop losses
- ❌ Don't trade during extreme volatility without adjusting parameters

## Optimization Tips

### Parameter Optimization

1. **Lookback Period**:
   - Shorter (50-100): More responsive, more trades, more noise
   - Longer (100-200): More stable, fewer trades, slower to adapt
   - Test different market regimes (trending vs. ranging)

2. **Entry Threshold**:
   - Lower (1.0-1.5): More trades, lower win rate, more whipsaws
   - Higher (2.0-3.0): Fewer trades, higher win rate, miss opportunities
   - Optimize based on timeframe

3. **Exit Threshold**:
   - Lower (0.2-0.5): Quick exits, more trades, less profit per trade
   - Higher (0.5-1.0): Hold longer, fewer trades, more profit per trade
   - Balance with stop loss distance

### Market Regime Considerations

**Trending Markets** (strong directional moves):
- Increase entry threshold (2.5-3.0)
- Wider stop loss (4.0-5.0)
- Correlation may weaken temporarily

**Range-Bound Markets** (sideways consolidation):
- Decrease entry threshold (1.5-2.0)
- Tighter exit (0.3-0.5)
- Correlation typically stronger

**High Volatility Markets**:
- Increase lookback period (150-200)
- Wider bands (3.0 entry, 5.0 stop)
- Reduce position size

## Technical Details

### Pine Script Version
- **Version**: 6
- **Overlay**: False (separate panel)
- **Calculation**: On bar close (no repainting)

### Data Requirements
- Requires TradingView data access to ES1! and NQ1!
- Works on any timeframe (5min to Daily)
- Minimum lookback period requires sufficient historical data

### Performance Optimization
- Uses `request.security()` with proper lookahead settings
- No repainting: All calculations use confirmed bars
- Efficient calculations: Minimal redundant computations

## Troubleshooting

### Common Issues

**Issue**: No trades appearing in backtest
- **Solution**: Reduce entry threshold or increase lookback period
- **Check**: Ensure ES1! and NQ1! data is available for your timeframe

**Issue**: Too many whipsaw trades
- **Solution**: Increase entry threshold, decrease exit threshold
- **Adjust**: Use longer lookback period for more stability

**Issue**: Large drawdowns
- **Solution**: Enable stop loss, reduce position size
- **Review**: Check if correlation has broken down in test period

**Issue**: Strategy not loading
- **Solution**: Verify Pine Script v6 syntax, check for TradingView updates
- **Check**: Ensure you have required data subscriptions

## Additional Resources

### Understanding Pairs Trading
- Research "cointegration" and "mean reversion" strategies
- Study correlation vs. cointegration
- Learn about Augmented Dickey-Fuller (ADF) test for pairs selection

### Futures Trading
- CME Group: ES and NQ contract specifications
- Understand futures margin, mark-to-market, and settlement
- Learn about futures roll dates and contract selection

### Risk Management
- Position sizing methodologies (Kelly Criterion, Fixed Fractional)
- Portfolio heat and maximum risk per trade
- Drawdown management and recovery strategies

## Version History

**v1.0** (2025-11-20)
- Initial release
- Core pairs trading functionality
- Z-score based entry/exit
- Multiple normalization methods
- Comprehensive visualization
- Integrated statistics table

## License & Disclaimer

© 2025 BucksTRDR - All Rights Reserved

**FOR EDUCATIONAL AND RESEARCH PURPOSES ONLY**

This strategy is provided "as is" without warranty of any kind. Trading futures involves substantial risk of loss and is not suitable for all investors. Past performance is not indicative of future results. The developer assumes no responsibility for any losses incurred from using this strategy. Always consult with a licensed financial advisor before trading.

## Support & Feedback

For questions, issues, or suggestions:
- Review TradingView Pine Script documentation
- Test thoroughly in paper trading before live use
- Maintain a trading journal to track performance

---

**Happy Trading! Always manage your risk.** 📊
