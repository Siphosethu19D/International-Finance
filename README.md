# International Finance Assignment 2 — FX Intervention and Firm Exposure

This project estimates firm-level exchange-rate exposure and tests whether the
association between central-bank intervention and absolute exposure differs
between crisis and calm periods. It follows Sikarwar (2020), Eq. 1–6, and adds
period-specific exposure and intervention interactions guided by Sikarwar
(2023).

## Install and run

From this folder:

```powershell
pip install -r requirements.txt
python run_all.py
```

The scripts are re-runnable. Successfully retrieved source files are cached in
`data/raw/`; set `FORCE_DOWNLOAD = True` in `config.py` to force downloads.
Derived observations and analysis-ready series are written to `data/clean/`.
Tables, figures, logs, and the Excel workbook are written to `output/`.

## Data sources and definitions

| Data | Source / URL | Frequency / definition |
|---|---|---|
| Broad nominal effective exchange rates | BIS Data Portal, <https://data.bis.org/topics/EER>; SDMX dataflow `WS_EER`, monthly nominal broad basket | Monthly NEER; `R_s = ln(NEER_t / NEER_t-1)`. Positive values mean appreciation. If the API is unavailable, save the BIS CSV as `data/raw/bis_neer_manual.csv`. |
| Local equity market indices and firms | Yahoo Finance, <https://finance.yahoo.com/> | Monthly adjusted close; log returns. Original candidates and selected backups are recorded in `tickers.csv` and `output/dropped_firms_log.csv`. |
| Reserves, money and exchange rates | World Bank Indicators API, <https://api.worldbank.org/v2/> | Annual `FI.RES.XGLD.CD`, `FM.LBL.BMNY.CN`, and `PA.NUS.FCRF`; proxy is the annual change in reserves converted to local currency divided by broad money. |
| Country controls | World Bank Indicators API, <https://api.worldbank.org/v2/> | Annual `NE.TRD.GNFS.ZS`, `FM.LBL.BMNY.GD.ZS`, and `NY.GDP.PCAP.CD`; GDP per capita enters in logs. |
| Crisis windows | NBER recession chronology, <https://www.nber.org/research/data/us-business-cycle-expansions-and-contractions>, and Sikarwar (2023), Table A2 | GFC primary window is 2007-08 to 2009-06. COVID windows are country-specific; Indonesia, Korea and Thailand dates are flagged VERIFY. |

The IMF monthly IFS request was tried once more with a browser User-Agent and
returned HTTP 403. The pipeline therefore keeps the stated annual World Bank
fallback. Annual observations are weighted by the number of months overlapping
each period; they are never interpolated to monthly values.

## Sample and limitations

- The replication window is 2001-04 to 2017-03; the extension runs through
  2025-12 or the latest available observation.
- Bloomberg's original market data are replaced by BIS NEER, Yahoo Finance
  market/firm prices, and IMF/World Bank public series. The intervention
  estimate is annual, not monthly, due to blocked IMF access. This limits
  within-year timing and inference in short crisis windows.
- The original paper used 751 firms. This project uses a smaller current
  constituent list (about 10 candidates per country) and screens by observed
  price-return coverage. Fewer firms are retained where source coverage is
  incomplete; firms are never backfilled or imputed.
- Current index constituents create survivorship bias. Some originally listed
  firms have no usable Yahoo Finance history or ticker and are documented in
  `output/dropped_firms_log.csv`.
- Yahoo Finance fundamentals have limited historical depth. Only SIZE is
  considered, and it is the current market-cap value—not a historical
  time-varying size measure. Other fundamentals are not imputed or included.
- The COVID ON window is short (approximately 11–15 months, depending on
  country), so estimates can be imprecise.
- Country-clustered standard errors use only the available subset of the ten
  countries and should be treated cautiously.
- Indonesia, Korea, and Thailand COVID dates are not listed in Sikarwar (2023)
  Table A2 and remain flagged VERIFY in `config.py`.
- Countries with fewer than three firms meeting a window's screen are excluded
  from that window. Countries with an index history too short for the required
  aligned estimation sample are also excluded rather than backfilled.

## Main equations

```text
Eq. (1): R_it = b0 + b_m R_m,t + b_s R_s,t + e_it
Extension A: R_it = b0 + b_m R_m,t + b_OFF(OFF*R_s,t) + b_ON(CRISIS*R_s,t) + e_it
Extension B: R_it = b0 + b_m R_m,t + b_OFF(OFF*R_s,t) + b_GFC(GFC*R_s,t)
              + b_COVID(COVID*R_s,t) + e_it
Stage 2: |b_s,i,p| = a + g1 INT_c,p + g2 CRISIS_p
          + g3(INT_c,p * CRISIS_p) + controls + industry effects + e_i,p
```

`g3` is the formal test of whether the intervention-exposure association changes
between crisis and calm periods. Results are computed only from downloaded and
estimated values.
