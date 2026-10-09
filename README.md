# UK Electricity Mix After the Russia–Ukraine War

**Did Russia's invasion of Ukraine cause a detectable shift in the UK's electricity generation mix, and if so, which sources changed the most?**

This project analyses six years of UK grid data (January 2020 – January 2026) using visual trend analysis, t-tests and an interrupted time-series (ITS) regression. I implemented the regression, standard errors and p-values from scratch in NumPy.

*Originally completed as coursework for Foundations of Data Science at the University of Edinburgh (2026).*

---

## Key findings

- **Gas (CCGT) generation fell by 5.4 percentage points** of demand after the invasion, while **wind rose by 4.9 percentage points**, a near one-for-one substitution.
- **The effect was not immediate.** Gas spiked through winter 2022 (to about 50% of demand) before declining sharply through 2023–2024.
- The ITS regression finds a **statistically significant post-invasion downward trend in gas of about 0.018 pp per day** (≈ 6.5 pp per year, p < 0.001), with no significant pre-invasion trend (p = 0.053).
- **Coal did not recover** despite high gas prices, and continued its decline to effectively zero by 2025.

| Source | Pre-invasion (% of demand) | Post-invasion (% of demand) | Change (pp) | t-statistic | Significant? |
|---|---:|---:|---:|---:|---|
| Gas (CCGT) | 38.91 | 33.48 | −5.43 | 9.01 | Yes (p < 0.0001) |
| Wind | 21.01 | 25.94 | +4.93 | −8.66 | Yes (p < 0.0001) |
| Nuclear | 17.91 | 15.71 | −2.21 | 13.96 | Yes (p < 0.0001) |
| Biomass | 7.25 | 6.53 | −0.72 | 7.01 | Yes (p < 0.0001) |
| Solar | 4.40 | 5.76 | +1.37 | −7.71 | Yes (p < 0.0001) |
| Coal | 1.71 | 0.73 | −0.98 | 15.19 | Yes (p < 0.0001) |



---

## Data

- **Source:** [Gridwatch](https://gridwatch.templar.co.uk), run by Templar Consultancy Ltd, using data from the Elexon portal and Sheffield University's solar estimates.
- **Coverage:** 1 January 2020 – 1 January 2026, at **5-minute intervals** (629,421 rows, 26 columns): demand, grid frequency, and generation in MW for coal, nuclear, CCGT, wind, solar, biomass, hydro, pumped storage, oil and the interconnectors (France, Netherlands, Ireland, Belgium, Norway, Denmark).
- The raw CSV is **not included** in this repository because of its size. Download it from the Gridwatch download page: select all generation categories and the date range above, save it as `gridwatch.csv`, and place it in the project root.

## Method

1. **Processing:** parse timestamps, resample from 5-minute readings to **daily means**, and express each source as a **percentage of total demand** to account for seasonal changes in consumption. Add a binary post-invasion flag (from 24 February 2022). Only 2 of 2,192 days contained missing values (0.09%); these were excluded from the tests.
2. **Visual trend analysis:** a stacked area chart of the generation mix, and per-source plots with **90-day rolling averages**.
3. **Independent-samples t-tests:** compare each source's mean share before and after the invasion.
4. **Interrupted time-series regression** on gas share:

   `ccgt_pctₜ = β₀ + β₁·t + β₂·post_invasionₜ + β₃·time_since_invasionₜ + εₜ`

   - β₁: pre-invasion trend
   - β₂: immediate level change at the invasion
   - β₃: change in trend after the invasion

   I fitted it with **ordinary least squares in NumPy** (`np.linalg.lstsq`), and computed standard errors, t-statistics and p-values by hand from the residual variance and (XᵀX)⁻¹, rather than using a statistics library.

| ITS term | Coefficient | t-stat | p-value |
|---|---:|---:|---:|
| Constant | 37.38 | 41.07 | < 0.001 |
| Time (pre-invasion trend) | 0.0039 | 1.94 | 0.053 |
| Post-invasion (level change) | 2.77 | 2.44 | 0.015 |
| Time since invasion (trend change) | −0.0178 | −8.16 | < 0.001 |

R² = 0.14

---

## Limitations

- **Seasonal confounding.** The invasion happened in February, when gas demand is high. The rolling averages and ITS only partly control for seasonality.
- **An existing trend.** The UK's shift to renewables was already underway before 2022, which makes it hard to separate the war's effect from the long-run transition. The pre-invasion gas trend was slightly positive but not significant.
- **Low R² (0.14).** The ITS model explains a small share of day-to-day variation in gas output. Most of it comes from weather, seasonality and other factors not in the model.
- **External shocks.** French nuclear output was unusually low in 2022 because of maintenance outages, which may have affected interconnector flows and UK generation.
- **Before/after design.** This shows a strong association with the timing of the invasion, not proof of cause.

## Possible extensions

- Add **seasonal controls** (month dummies or Fourier terms) and weather variables (wind speed, temperature) to the ITS model.
- Use **UK wholesale gas prices** directly, to link price shocks to changes in the generation mix.
- Extend the baseline before 2020 to separate the war's effect from the coal phase-out trend.
- Analyse **interconnector flows**, to see whether UK reliance on European imports changed during the crisis.

---

## How to run

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook
```

Place `gridwatch.csv` in the project root (see [Data](#data)), then run the notebook from top to bottom.

**Requirements:** Python 3.10+, `pandas`, `numpy`, `scipy`, `matplotlib`, `jupyter`.

## Project structure

```
├── FDS-Project-Code.ipynb   # analysis notebook
├── report.pdf               # full written report (optional)
├── figures/                 # exported plots
├── requirements.txt
└── README.md
```

## References

1. Carbon Brief. *Analysis: UK renewables enjoy record year in 2025.* 2026.
2. Energy and Climate Intelligence Unit. *The cost of gas since the Russian invasion of Ukraine.* 2023.
3. House of Commons Library. *Gas and electricity prices during the 'energy crisis' and beyond.* 2024.
4. International Energy Agency. *Russia's war on Ukraine.* 2024.
