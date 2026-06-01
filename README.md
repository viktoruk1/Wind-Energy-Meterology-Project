# Wind-Energy-Meterology-Project
This repo contains data, notebook and explanations to Wind Energy Meterology Project. This project is conducted as part of the Wind Physics Measurement in Summer 2026 at ForWind, Oldenburg.
[README.md](https://github.com/user-attachments/files/28461535/README.md)
# README — Wind Energy Meteorology Project

## Project overview

This project analyses long-term wind measurements from the BGC Jena weather station (10 m height, central Germany) covering 2017–2024 at 10-minute resolution. The goal is to characterise the site's wind climate statistically and physically, using the methodology of the Wind Energy Meteorology lecture (ForWind, summer term 2026).

Dataset 1: long_term_jena_10min.csv — ~420,000 rows of (datetime, wind speed [m/s], wind direction [°], temperature [°C]).
Dataset 2 (used for later tasks, not in this README): *_20Hz_*.csv — one day of high-frequency (20 Hz) wind data at four heights (50, 110, 175, 250 m).
Site source: BGC Jena weather station, MPI for Biogeochemistry. Documentation: https://www.bgc-jena.mpg.de/wetter/


## Environment setup

- Anaconda (Python distribution): https://www.anaconda.com/download
- A working internet connection for package installation
~2 GB free disk space

**Create the environment**
**Open Anaconda Prompt and run:**

# 1. Configure conda to use the conda-forge channel (avoids university network issues)
conda config --add channels conda-forge
conda config --set channel_priority strict

# 2. Create the project environment with all required packages
conda create -n windmet python=3.12 numpy pandas matplotlib scipy statsmodels jupyterlab -y

# 3. Activate it
conda activate windmet

# 4. Install windrose (PyPI-only package)
pip install windrose

Launch the workspace
conda activate windmet
jupyter lab

### TASKS

#### Task 1 — Exploratory Data Analysis (Scales)

What: Visualise one full year of 10-min wind speed data alongside its 24-hour moving mean and 24-hour moving standard deviation. Identify periods of elevated vs low wind, and interpret what each smoothing scale reveals.

Why: Real wind data contains structure at multiple timescales — synoptic (days), diurnal (hours), turbulent (seconds). Moving averages filter out the fastest variability, leaving the slower trend visible. This is the first step in understanding any time series: see what's there before quantifying it.

**Steps:**

1. Select one year (e.g. 2023) from the dataset.
2. Compute a 24-hour rolling mean (window = 144 ten-minute samples) and rolling standard deviation.
3. Plot the raw wind speed with rolling mean and ±σ band overlaid.
4. Annotate periods of high vs low wind, and discuss which scales of variability are filtered out by the 24-hour smoothing.

#### Task 2 — Statistical Description

##### 2(a) — Weibull distribution fit

What: Fit the Weibull distribution to wind speed for two chosen years; report and compare shape parameter kk
k and scale parameter AA
A.

Why: Wind speed is strictly non-negative and right-skewed; the Weibull distribution captures this naturally. Its two parameters carry physical meaning — AA
A (scale, m/s) controls the typical wind magnitude, kk
k (shape, dimensionless) controls the steadiness. Higher kk
k implies more predictable wind; site assessment standards rely on these two parameters.

**Steps:**

1. Pick two years (e.g. 2020 and 2023).
For each year, fit Weibull with scipy.stats.weibull_min.fit(speeds, floc=0). Always pin floc=0 — wind speed can be exactly zero.
2. Plot a histogram (density=True) with the fitted Weibull PDF overlaid.
3. Interpret: which year was windier (AA
A)? Which was steadier (kk
k)? Where does the fit deviate from the histogram, and what does that mean physically?

##### 2(b) — Wind rose

What: Polar plot showing the frequency distribution of wind direction, colour-coded by wind speed.

Why: Wind direction is circular — a normal histogram fails for it. The wind rose answers two questions at a glance: where does wind come from most often (petal length) and where does strong wind come from (petal colour). Critical for wind-farm siting and turbine orientation.

**Steps:**

1.Choose one year for clarity.
2. Drop rows where speed or direction is NaN.
3. Use windrose.WindroseAxes with speed bins like [0, 1, 2, 3, 4, 6, 10] and normed=True to show percentages.
4. Identify: prevailing direction, secondary maximum (if any), where the strongest winds originate.

##### 2(c) — Directional energy proxy

What: For each compass sector, compute the percentage contribution to total wind energy using Eθ=100⋅∑i∈θui3∑iui3E_\theta = 100 \cdot \frac{\sum_{i \in \theta} u_i^3}{\sum_i u_i^3}
Eθ​=100⋅∑i​ui3​∑i∈θ​ui3​​.

Why: Wind power scales as U3U^3
U3 (cube law) — a moderately strong wind direction can deliver more energy than a far more frequent but weaker direction. Frequency-based wind roses can therefore *mislead* about energy availability. The energy proxy reveals which direction actually matters for production.

**Steps:**

1. Bin wind directions into 12 sectors of 30°.
2. For each sector, compute frequency (% of year), mean wind speed, and energy contribution (EθE_\theta
Eθ​).
3. Plot all three as bar charts side by side.
4. Compare frequency vs energy distributions — identify any sectors where the two disagree, and explain the cube-law amplification.


#### Task 3 — Detrending (Scale Separation)

What: Decompose a 3-month wind-speed segment into trend TtT_t
Tt​ (slow synoptic background), seasonal StS_t
St​ (repeating daily cycle), and residual RtR_t
Rt​ (random fluctuations), using yt=Tt+St+Rty_t = T_t + S_t + R_t
yt​=Tt​+St​+Rt​. Quantify each component's standard deviation.

Why: A raw wind signal mixes multi-day weather, daily rhythms, and microscale randomness. Separating them lets each be studied independently — useful for forecasting (trend), scheduling (seasonal), and understanding load variability (residual). Comparing the magnitudes reveals which physical process dominates the site.

**Steps:**
1. Pick a 3-month window (e.g. Jan–Mar 2023).
2. Resample onto a clean 10-min grid: .resample('10min').mean().interpolate(limit=6).dropna().
3. Apply statsmodels.tsa.seasonal.seasonal_decompose with model='additive', period=144 (one day = 144 ten-minute samples).
4. Plot the four panels: data, trend, seasonal, residual.
Compute σT\sigma_T
σT​, σS\sigma_S
σS​, σR\sigma_R
σR​ with np.nanstd, and compute the ratio σR/σS\sigma_R / \sigma_S
σR​/σS​ to test whether random variability dominates the diurnal cycle (lecture: expect ~2 for onshore sites).


#### Task 4 — Seasonal Diurnal Cycle and Burstiness

**4(a) — Mean diurnal cycle by season**

What: For each of the four meteorological seasons (DJF, MAM, JJA, SON), compute the mean wind speed at each hour of the day; plot as four curves on one figure.

Why: The interaction between solar heating and atmospheric stability produces a daily wind cycle that varies strongly with season. Strong summer heating drives convective mixing → afternoon wind peak; weak winter sun → little diurnal variation, synoptic weather dominates. This is a direct empirical probe of boundary-layer physics.

**Steps:**
1. Tag every timestamp with its season (function mapping month → season) and its hour.
2. Group by (season, hour) and take the mean of wind speed.
3. Pivot with .unstack('season') to get a 24×4 table.
4. Plot four curves, one per season.
5. Interpret: which season is windiest in the afternoon? Which has the flattest daily cycle? Link to atmospheric stability — daytime mixing vs nocturnal decoupling vs winter's synoptic dominance.


**4(b) — Diurnal burstiness**

What: For each (season, hour) bucket, compute B(h)=σu′(h)/∣u′∣‾(h)B(h) = \sigma_{u'}(h) / \overline{|u'|}(h)
B(h)=σu′​(h)/∣u′∣​(h), where u′=u−uˉu' = u - \bar{u}

u′=u−uˉ is the residual after a 1-hour moving-average detrend. Plot as four curves on a second figure.

Why: Standard deviation alone (and hence TI = σ/mean) cannot distinguish smooth turbulence from intermittent burst events that share the same variance. 
***Burstiness*** — the ratio of fluctuation spread to typical fluctuation magnitude — reveals whether the wind is uniformly noisy (low B, near Gaussian value 1.253) or spike-dominated (high B). Same TI can hide very different physical regimes; turbine loads care about the difference.

**Methodological note:** The exercise sheet's formula B=σu′/∣u′‾∣B = \sigma_{u'} / |\overline{u'}|
B=σu′​/∣u′∣ becomes numerically unstable when ∣u′‾∣|\overline{u'}|
∣u′∣ approaches zero (which it does when averaging mean-zero residuals over many samples). We use the lecture's stable diurnal variant B(h)=σu′(h)/∣u′∣‾(h)B(h) = \sigma_{u'}(h) / \overline{|u'|}(h)
B(h)=σu′​(h)/∣u′∣​(h) — mean of *absolute values*, never zero — which preserves the physical interpretation.


**Steps:**

1. Detrend the full dataset: uprime = u - u.rolling(6, center=True).mean().
2. Tag with season and hour.
Group by (season, hour); compute the standard deviation of uprime and the mean of |uprime|; take their ratio.
3. Plot four curves of BB
B vs hour.
4. Interpret: when is wind burstiest? Compare across seasons. Link to stability transitions (sunrise/sunset) vs continuous daytime mixing vs synoptic gust events in winter. Emphasise that TI alone would hide this distinction.
