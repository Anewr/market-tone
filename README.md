# Macro Market Tone Scoring Dashboard

A comprehensive dashboard for analyzing and scoring macro market conditions across three key dimensions: **Liquidity**, **Rates**, and **Risk Appetite**. The system aggregates multiple data sources to produce actionable market tone scores ranging from -2 (bearish/restrictive) to +2 (bullish/supportive).

## 🎯 Features

- **Multi-Dimensional Scoring**: Analyzes liquidity conditions, rate pressures, and risk sentiment
- **Real-Time Data**: Fetches live data from FRED API, yfinance, CFTC, and other sources
- **Intelligent Caching**: Caches data to minimize API calls and improve performance
- **Interactive Dashboard**: Beautiful Streamlit interface with detailed score explanations
- **Historical Tracking**: Save and visualize historical snapshots of market conditions
- **Transparent Calculations**: Detailed explanations of how each score component is calculated

## 📊 Score Interpretation

### Liquidity Score (-2 to +2)
- **-2**: Tightening fast
- **-1**: Tightening
- **0**: Neutral
- **+1**: Easing
- **+2**: Strong easing

### Rates Score (-2 to +2)
- **Negative**: Tightening/restrictive rate environment
- **Positive**: Easing/supportive rate environment

### Risk Appetite Score (-2 to +2)
- **Negative**: Risk-off/defensive behavior
- **Positive**: Risk-on behavior

### Composite Score
Weighted average of all three dimensions (40% Liquidity, 30% Rates, 30% Risk)

## 🚀 Installation

### Prerequisites
- Python 3.8 or higher
- FRED API key ([Get one here](https://fred.stlouisfed.org/docs/api/api_key.html))

### Setup

1. **Clone the repository**
   ```bash
   git clone <your-repo-url>
   cd macro_engine
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Configure environment variables**
   
   Create a `.env` file in the project root (copy from `.env.example`):
   ```bash
   cp .env.example .env
   ```
   
   Then edit `.env` and add your FRED API key:
   ```bash
   FRED_API_KEY=your_fred_api_key_here
   ```
   
   **Get your free FRED API key**: https://fred.stlouisfed.org/docs/api/api_key.html
   
   > ⚠️ **Important**: Never commit your `.env` file to Git! It's already in `.gitignore`.

4. **Run the application**
   ```bash
   streamlit run app.py
   ```

   The dashboard will open in your browser at `http://localhost:8501`

## 📁 Project Structure

```
macro_engine/
├── app.py                 # Streamlit dashboard UI
├── requirements.txt       # Python dependencies
├── .env                   # Environment variables (create this)
│
├── ingestion/             # Data fetching modules
│   ├── liquidity.py      # Liquidity data and scoring
│   ├── rates.py          # Rates data and scoring
│   ├── risk.py           # Risk appetite data and scoring
│   ├── cftc.py           # CFTC positioning data
│   └── russell_breadth.py # Russell 3000 breadth data
│
├── scoring/              # Score orchestration
│   ├── all_scores.py     # Composite score calculation
│   ├── liquidity_score.py
│   ├── rates_score.py
│   └── risk_score.py
│
├── utils/                # Utilities
│   ├── config.py         # Configuration and SSL setup
│   ├── cache.py          # Caching logic
│   └── io.py             # File I/O operations
│
└── data/                 # Data storage (auto-created)
    ├── raw/              # Cached raw data
    └── snapshots/        # Historical snapshots
```

## 📈 How Scores Are Calculated

### Liquidity Score Components

1. **Baa Corporate Yield Z-Score** (FRED: BAA)
   - Measures corporate credit conditions
   - Z-score < -2 → +1 (very low yields, easing)
   - Z-score > +2 → -1 (very high yields, tightening)

2. **LQD/IEF RSI Divergence**
   - LQD: iShares Investment Grade Corporate Bond ETF
   - IEF: iShares 7-10 Year Treasury Bond ETF
   - Detects bullish/bearish divergences between price and RSI

3. **LQD/IEF RSI Ratio**
   - Ratio > 1 → +1 (corporate bonds outperforming)
   - Ratio < 1 → -1 (treasuries outperforming)

4. **Russell 3000 Breadth**
   - High/Low ratio ≥ 2 → +1 (strong breadth)
   - High/Low ratio ≤ 0.5 → -1 (weak breadth)

**Final Score**: Average of all components, clipped to [-2, +2]

### Rates Score Components

1. **Fed Funds Momentum** (FRED: EFFR)
   - 1-month change < -0.05% → +1 (easing)
   - 1-month change > +0.05% → -1 (tightening)

2. **Yield Curve Level** (10Y - 2Y Spread)
   - Spread > 0.75% → +1 (healthy curve)
   - Spread < 0% → -1 (inverted curve)

3. **Yield Curve Momentum**
   - Steepening > 0.10% → +1 (growth expectations)
   - Flattening < -0.10% → -1 (concern)

**Final Score**: Average of all components, clipped to [-2, +2]

### Risk Appetite Score Components

1. **VIX Z-Score** (FRED: VIXCLS)
   - Z-score < -1 → +1 (low volatility, risk-on)
   - Z-score > +1 → -1 (high volatility, risk-off)

2. **Copper/Gold Ratio Momentum**
   - Ratio increasing → +1 (growth preference)
   - Ratio decreasing → -1 (defensive preference)

3. **S&P 500 Momentum** (FRED: SP500)
   - Positive change → +1 (risk-on)
   - Negative change → -1 (risk-off)

4. **CFTC Equity Positioning**
   - Large speculator z-score > +1 → +1 (bullish)
   - Large speculator z-score < -1 → -1 (bearish)

**Final Score**: Average of all components, clipped to [-2, +2]

## 🔌 Data Sources

- **FRED (Federal Reserve Economic Data)**: Interest rates, yields, volatility, equity indices
- **yfinance**: ETF prices (LQD, IEF, GLD)
- **CFTC**: Commitment of Traders positioning data
- **Barchart**: Russell 3000 breadth data

## ⚙️ Configuration

### Environment Variables

#### Local Development

Create a `.env` file in the project root:
```bash
FRED_API_KEY=your_api_key_here
```

The `.env` file is automatically ignored by Git (see `.gitignore`), so your API key will never be committed.

#### Streamlit Cloud Deployment

If deploying to Streamlit Cloud:

1. Go to your app's settings in Streamlit Cloud
2. Navigate to **Secrets**
3. Add your API key:
   ```toml
   FRED_API_KEY = "your_api_key_here"
   ```

The application will automatically check Streamlit secrets first, then fall back to environment variables.

### SSL Certificates (macOS)

If you encounter SSL certificate errors on macOS, the application automatically handles this. For production use, you can install certificates properly:

```bash
/Applications/Python\ 3.13/Install\ Certificates.command
```

## 📝 Usage

1. **Start the dashboard**
   ```bash
   streamlit run app.py
   ```

2. **View scores**: The dashboard displays three dimension scores and a composite score

3. **Explore details**: Click "Drivers" to see underlying data, or "How is this calculated?" for methodology

4. **Save snapshots**: Use the "Save Snapshot" button to capture point-in-time market states

5. **View history**: Historical liquidity scores are displayed in a chart at the bottom

## 🛠️ Development

### Adding New Data Sources

1. Create a new fetcher function in the appropriate `ingestion/` module
2. Follow the caching pattern using `utils.cache`
3. Integrate into the scoring logic

### Modifying Score Calculations

- Edit the scoring functions in `ingestion/liquidity.py`, `ingestion/rates.py`, or `ingestion/risk.py`
- Update the explanation text in `app.py` if calculation logic changes

## 📦 Dependencies

- **streamlit**: Web dashboard framework
- **pandas/numpy**: Data manipulation
- **plotly**: Interactive charts
- **fredapi**: FRED API client
- **yfinance**: Yahoo Finance data
- **requests/beautifulsoup4**: Web scraping
- **python-dotenv**: Environment variable management

See `requirements.txt` for complete list with versions.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

[Add your license here]

## 🙏 Acknowledgments

- Federal Reserve Economic Data (FRED) for economic data
- CFTC for positioning data
- Barchart for market breadth data

---

**Note**: This tool is for informational purposes only and should not be used as the sole basis for investment decisions.


