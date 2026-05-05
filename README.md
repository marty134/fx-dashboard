# FX Dashboard

A browser-based foreign exchange dashboard for quick currency analysis, pair comparison, and volatility scanning.

## Who This Is For

- **Users** who want to run the dashboard locally and explore FX insights.
- **Contributors** who want to understand the code structure and safely extend features.

## What The App Includes

The UI is organized into five tabs:

1. **Market Watch** (`MW`)  
   Compares a selected base currency against the configured currency basket, with mini trend charts and percentage change.

   <details>
   <summary>How Market Watch metrics are calculated</summary>

   - For each quote currency, the app builds a time series from historical daily rates (`MW.render`).
   - Current value is the latest point in the series.
   - Percent change is calculated as:
     - `((current - start) / start) * 100`
   - Card color is green for non-negative change, red for negative change.
   - The sparkline uses the same series and applies local min/max scaling for readability.

   Related concepts:
   - Arithmetic return: <https://en.wikipedia.org/wiki/Rate_of_return>

   </details>

2. **Arbitrage Scanner** (`ARB`)  
   Uses RSI-based signals over selectable windows (1M to 5Y) to flag overbought/oversold conditions.

   <details>
   <summary>How Arbitrage Scanner metrics are calculated</summary>

   - For each base/quote pair, the app slices recent prices based on slider window (`ARB.render`).
   - RSI is computed with a smoothed average-gain/average-loss method (`ARB.calcRSI`).
   - Signal thresholds:
     - `RSI < 30`: oversold (buy base / sell quote)
     - `RSI > 70`: overbought (sell base / buy quote)
     - otherwise: hold / fair value
   - The chart color also reflects RSI state (green, red, neutral gray).

   Related concepts:
   - Relative Strength Index (RSI): <https://en.wikipedia.org/wiki/Relative_strength_index>

   </details>

3. **Correlation Matrix** (`CORR`)  
   Builds a correlation heatmap for major FX pairs over a recent period.

   <details>
   <summary>How Correlation Matrix metrics are calculated</summary>

   - The app converts EUR-based source rates into target pair prices (`CORR.getReturns`).
   - Daily arithmetic returns are computed as:
     - `(price_t - price_(t-1)) / price_(t-1)`
   - Pair-vs-pair relationship is measured with Pearson correlation (`CORR.pearson`).
   - Display coloring:
     - positive high values are green,
     - negative high-magnitude values are red,
     - near-zero values stay neutral.

   Related concepts:
   - Pearson correlation coefficient: <https://en.wikipedia.org/wiki/Pearson_correlation_coefficient>
   - Rate of return: <https://en.wikipedia.org/wiki/Rate_of_return>

   </details>

4. **Bollinger Bands** (`BB`)  
   Ranks pairs by Bollinger Bandwidth to identify potential squeezes and shows a band chart.

   <details>
   <summary>How Bollinger Band metrics are calculated</summary>

   - For each pair, the app computes a 20-period simple moving average and standard deviation (`BB.calcBB`).
   - Bands are:
     - `upper = SMA + 2 * SD`
     - `lower = SMA - 2 * SD`
   - Bandwidth (used for ranking squeeze candidates) is:
     - `(upper - lower) / SMA`
   - Lower bandwidth ranks first as the tightest squeeze.

   Related concepts:
   - Bollinger Bands: <https://en.wikipedia.org/wiki/Bollinger_Bands>
   - Standard deviation: <https://en.wikipedia.org/wiki/Standard_deviation>

   </details>

5. **Strength Meter** (`CSM`)  
   Scores each currency by relative performance against a basket of peers.

   <details>
   <summary>How Strength Meter scores are calculated</summary>

   - For each currency, the app compares start vs end cross-rates against every other currency (`CSM.update`).
   - Cross-rate for `curr/opponent` is derived from EUR-based feed as:
     - `(EUR/opponent) / (EUR/curr)`
   - Change versus each opponent is:
     - `((endRate - startRate) / startRate) * 100`
   - Final strength score is the arithmetic average of those opponent-level changes.
   - Bars are sorted descending by score and scaled by max absolute score.

   Related concepts:
   - Cross rates: <https://en.wikipedia.org/wiki/Exchange_rate>
   - Arithmetic mean: <https://en.wikipedia.org/wiki/Arithmetic_mean>
   - Rate of return: <https://en.wikipedia.org/wiki/Rate_of_return>

   </details>

## Data Sources And Runtime Dependencies

From the current code in `index.html`:

- Exchange-rate data is requested from `https://api.frankfurter.dev`.
- Charts are rendered with CDN scripts:
  - `chart.js`
  - `chartjs-adapter-date-fns`
- Flag images are loaded from `https://flagcdn.com`.

Because these are remote services, an internet connection is required for normal use.

## Quick Start (Windows)

No build tooling is currently present in this repository (single-page app in `index.html`).

### Option 1: Open directly

Open `index.html` in your browser.

### Option 2: Serve locally (recommended)

Using Python:

```powershell
cd C:\Users\marten\PycharmProjects\fx-dashboard
python -m http.server 8000
```

Then open `http://localhost:8000`.

## Project Structure

- `index.html` - Entire app (markup, styles, and JavaScript modules).
- `README.md` - Project documentation.
- `LICENSE` - GPL v3 license text.
- `favicon.ico` - Browser icon.

## Internal Architecture (Contributor Notes)

Core pieces in `index.html`:

- `CONFIG` - Supported currencies and metadata.
- `DateUtils` - Date parsing, formatting, and range helpers.
- `Loader` - Global loading overlay toggles.
- `App.switchTab(tabId)` - Tab navigation and per-module init dispatch.
- Feature modules:
  - `MW` - Market Watch
  - `ARB` - Arbitrage Scanner
  - `CORR` - Correlation Matrix
  - `BB` - Bollinger Bands
  - `CSM` - Currency Strength Meter

### Typical Change Workflow

1. Update module logic in the relevant object (for example `BB.process` or `ARB.calcRSI`).
2. Keep UI wiring consistent with tab buttons and DOM IDs.
3. Validate behavior in browser across all tabs after changes.
4. If adding dependencies, document them in this README.

## Known Limitations

- Limited explicit user-facing error handling (some paths only log to console).
- API availability and response format are external dependencies.
- All logic lives in one file (`index.html`), which can make large changes harder to maintain.

## Contribution Guidelines

- Keep code style consistent with existing plain JavaScript approach.
- Prefer small, focused changes with clear reasoning.
- Document any new module, config field, or external dependency in `README.md`.
- Test manually in browser before opening a PR.

## License

This project is licensed under the GNU General Public License v3.0. See `LICENSE` for full details.
