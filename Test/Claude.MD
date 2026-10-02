# Project Plan: Does vulnerable employment increase during a recession?

## 1. Question and expectation

**Question:** Does vulnerable employment increase during a recession?

**Expectation (hypothesis):** Vulnerable employment rises during recessions. When an economy contracts, firms stop hiring and lay off formal wage workers. In most countries there is little unemployment insurance, so displaced workers can't afford to stay unemployed. Instead they move into own-account work (self-employment with no employees) or unpaid work in a family business. These are the two categories the ILO counts as "vulnerable employment."

I expect the effect to be:
- **positive**: the share of vulnerable employment rises in recession years relative to normal years;
- **larger in low- and middle-income countries**, where informality is common and safety nets are thin;
- **possibly delayed**, because labour markets often respond to output with a lag.

## 2. Data sources

All data come from the **World Bank World Development Indicators (WDI)** API (`api.worldbank.org/v2`), pulled inside the notebook with the Python package **`wbgapi`**. The World Bank API is free and needs **no API key**.

| Series code | Indicator | Role |
|---|---|---|
| `SL.EMP.VULN.ZS` | Vulnerable employment, total (% of total employment), modeled ILO estimate | Outcome |
| `NY.GDP.MKTP.KD.ZG` | GDP growth (annual %, constant prices) | Defines recessions |
| `NY.GDP.MKTP.CD` | GDP (current US$) | Cross-check only |

Country metadata (aggregate flag, income group, region) come from `wb.economy.DataFrame()`.

The CSV downloads saved in the project folder (`API_SL.EMP.VULN.ZS_...zip`, `API_NY.GDP.MKTP.CD_...zip`) are used only to verify that the API pull matches the downloaded files.

## 3. Cleaning steps

1. **Date range:** 1991–2025. The ILO modeled vulnerable-employment series starts in 1991.
2. **Reshape:** pull with one column per series and one row per country-year, and convert the `YR2009` time codes to integer years.
3. **Drop aggregates:** remove regional and income-group rows such as "World", "Euro area" and "Low income" (48 aggregates), so that no country is counted twice.
4. **Merge** country metadata (name, income group, region) onto the panel.
5. **Missing values:** keep countries with at least 10 years of both vulnerable employment and GDP growth. Rows missing either variable are dropped from each test rather than imputed.
6. **Transformations:**
   - `recession` = 1 if real GDP growth < 0, else 0
   - `recession_lag` = recession in the previous year
   - `onset` = recession year that follows a non-recession year
   - `d_vuln` = year-over-year change in vulnerable employment (percentage points). Differencing removes each country's permanent level.
7. **Cross-check:** compare the API values with the saved CSVs. They should match exactly.

## 4. Charts and tests

| Step | Chart / test | What it shows |
|---|---|---|
| A | Bar chart: share of countries in recession by year | Where the recession observations come from (2009 and 2020 spikes) |
| B | Bar chart with 95% CIs, plus a **Welch t-test** | Mean `d_vuln` in recession vs. non-recession years |
| C | **Panel OLS with country and year fixed effects**, country-clustered standard errors | Recession effect within countries, net of global shocks; adds a lagged recession term and a continuous GDP-growth version |
| D | **Event study**: line chart of vulnerable employment from t−3 to t+4 around recession onsets, relative to non-recession windows | Timing and persistence of the effect, and whether there are pre-trends |
| E | Coefficient plot by **World Bank income group** | Whether the effect is larger where informality is common |

## 5. What would support or contradict the expectation

**Supports the hypothesis:**
- Mean `d_vuln` is significantly higher in recession years (test B).
- The fixed-effects `recession` coefficient is positive and significant, and the `gdp_growth` coefficient is negative (test C).
- In the event study, the line is flat before onset and rises at or after t = 0 (test D).
- The effect is larger in low- and middle-income countries (test E).

**Contradicts the hypothesis:**
- The recession coefficient is zero or negative, or is insignificant once country and year fixed effects are added. Vulnerable employment would then just follow long-run trends, not the business cycle.
- The event study rises *before* onset. That would point to pre-existing trends or reverse causality rather than a recession effect.
- The effect is concentrated in high-income countries. That wouldn't fit the "no formal jobs → informal work" mechanism.

**Known limitation to keep in mind:** `SL.EMP.VULN.ZS` is an ILO *modeled* estimate. For country-years without labour-force surveys, it is imputed using macro variables that include GDP growth, so part of the measured relationship could be built into the data.
