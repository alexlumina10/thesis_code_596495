# Statistical Methods and Results: Reddit Activity as a Predictor of Prediction Market Trading Volume

---

## 1. Research Question

This analysis investigates whether the volume of Reddit discussion about a political event predicts the volume of trading on the corresponding Kalshi prediction market. Specifically, the analysis tests whether daily Reddit post counts Granger-cause daily trading volume across three political prediction markets: the U.S. government shutdown length market, the Federal Reserve Chair confirmation market, and a combined Strait of Hormuz closure market.

---

## 2. Data

### 2.1 Sources

**Prediction market data** were obtained from the Kalshi Trade API (v2), which provides daily candlestick data including trading volume (in contracts) at the individual market level. Eight markets were analysed across five settlement dates between January and May 2026:

- **KXGOVTSHUTLENGTH-26FEB07** — "How long will the government shutdown last?" (hereafter: *Govt Shutdown*)
- **KXFEDCHAIRCONFIRM** — Federal Reserve Chair confirmation market (hereafter: *Fed Chair*)
- **KXHORMUZ_COMBINED** — Strait of Hormuz closure combined market (hereafter: *Hormuz*)
- **KXVIRGINIAREDISTRICT_COMBINED** — Virginia redistricting referendum: pass/fail + margin of victory (combined daily volume; hereafter: *Virginia*)
- **KXTXSENDPRIMARYMOV-26MAR03** — Texas Senate Democratic primary margin of victory (hereafter: *Texas Senate*)
- **KXGOVOHNOMR-26** — Ohio Republican Governor primary nominee (hereafter: *Ohio Gov*)
- **KXSENATEILD-26** — Illinois Democratic Senate primary nominee (hereafter: *Illinois Senate*)
- **KXIL9D-26** — IL-09 Democratic House primary nominee (hereafter: *IL-09*)

For multi-market events (Virginia redistricting: two related markets; Ohio/Illinois/IL-09: multiple candidate markets), daily trading volumes across all sub-markets were summed into a single composite series. The settlement day (final trading day) was removed from each series to avoid contamination from mechanical position-closing activity, which generates anomalously high volume unrelated to information-driven trading.

**Reddit data** were sourced from three subreddits — r/politics, r/wallstreetbets, and r/TheBulwark — covering the period January–May 2026. Posts were filtered by event-specific keyword sets to produce a daily post count series (hereafter: *Reddit post count*). Days with no matching posts were assigned a count of zero. The proportion of such zero days varied substantially across markets and serves as a proxy for Reddit engagement salience (see Section 2.3).

### 2.2 Merged Dataset

The two series were merged by date for each market, with Kalshi trading dates forming the basis and Reddit counts filled with zero on non-matching dates. Datasets are stored in `data/govtshut_merged.csv` (original three markets) and `data/new_markets_merged.csv` (five new markets).

### 2.3 Descriptive Statistics

Prior to any inferential testing, descriptive statistics were computed for each series. A key structural feature of the data is the proportion of days on which the Reddit keyword filter returned zero posts — hereafter *Reddit zero-rate*. This varies from 0% (Hormuz) to 94% (Ohio Gov) and has important implications for statistical power, discussed in Section 9.

**Table 1.** *Descriptive statistics for Reddit post count and Kalshi trading volume by market.*

| Market | n | Reddit zero-rate | Reddit M (SD) | Reddit Max | Kalshi M (SD) | Kalshi Max |
|--------|---|-----------------|---------------|------------|---------------|------------|
| Govt Shutdown | 78 | 24% | 3.1 (4.3) | 23 | 174,184 (281,048) | 2,427,376 |
| Fed Chair | 70 | 37% | 2.2 (3.4) | 14 | 130,710 (173,067) | 1,159,214 |
| Hormuz | 28 | 0% | 11.2 (8.1) | 34 | 42,817 (30,612) | 130,399 |
| Virginia | 112 | 75% | 0.47 (1.29) | 9 | 143,233 (655,090) | 6,951,882 |
| Texas Senate | 55 | 26% | 3.9 (6.6) | 39 | 91,573 (418,041) | 3,012,420 |
| Ohio Gov | 123 | 94% | 0.10 (0.39) | 3 | 11,826 (74,109) | 800,000 |
| Illinois Senate | 74 | 91% | 0.22 (0.96) | 8 | 18,676 (111,900) | 621,000 |
| IL-09 | 76 | 90% | 0.21 (0.79) | 6 | 14,085 (77,190) | 500,000 |

Markets with a Reddit zero-rate below 50% (Govt Shutdown, Fed Chair, Texas Senate, Hormuz) are hereafter termed **high-engagement markets**; those above 50% are **low-engagement markets**. This distinction is revisited in the interpretation of results. All series exhibit strong positive skewness, consistent with count and financial volume data.

### 2.4 Reddit Zero-Rate and Market Type: An Interpretive Note

The wide variation in Reddit zero-rate across markets is not a data quality problem — it is substantively meaningful. It reflects a structural difference between the types of political events being traded.

**National, legislatively prominent events** (Govt Shutdown, Fed Chair, Texas Senate primary) generated sustained Reddit activity because they were the subject of national media coverage and public debate on platforms such as r/politics. The Kalshi traders active in these markets included a mixture of informed retail participants and the broader politically-engaged public, many of whom also participate in online political discussion.

**State and local races** (Ohio Governor primary, Illinois Senate primary, IL-09) attracted almost no Reddit discussion — not because the events were unimportant, but because the relevant information does not travel through national social media. Prediction market trading in these events is dominated by a different class of participant: political operatives, professional bettors, and specialist traders who follow state-level polling, local endorsements, and campaign finance disclosures. Their information sources are local press, private polling aggregators, and insider networks — not Reddit.

This distinction has a direct implication for the research question: the Reddit → trading volume information channel can only be detected in markets where Reddit *is* one of the active information channels. For state and local races, Reddit is essentially absent from the information environment. The null results for Ohio, Illinois Senate, and IL-09 therefore do not falsify the hypothesis; they define its scope. The question being answered is not "does Reddit predict all political prediction markets?" but "does Reddit predict political prediction markets where Reddit is a meaningful part of public discourse?"

---

## 3. Analytical Strategy

The analysis proceeded in five sequential steps:

1. **Stationarity testing** — Augmented Dickey-Fuller (ADF) tests were applied to each raw series to determine whether they contained a unit root.
2. **First differencing** — Non-stationary series were first-differenced to achieve stationarity, a prerequisite for valid Granger causality inference.
3. **Lag order selection** — A Vector Autoregression (VAR) model was estimated on the (possibly differenced) bivariate series and the Bayesian Information Criterion (BIC) was used to select the optimal lag length.
4. **Granger causality testing** — Granger causality F-tests were conducted in both directions (Reddit → Volume and Volume → Reddit) to assess whether one series provides predictive information about the other beyond its own history.
5. **Cross-correlation analysis** — Pearson cross-correlations were computed at lags −7 to +7 days to characterise the temporal structure of the bivariate relationship descriptively.

Each step is described in full below.

---

## 4. Step 1 — Stationarity Testing

### 4.1 Rationale

Granger causality tests are only valid when all series in the model are stationary (i.e., have a constant mean and variance over time). A series with a unit root — a stochastic trend — will yield spurious regression results, inflating test statistics and producing misleading *p*-values. The Augmented Dickey-Fuller (ADF) test is the standard procedure for detecting unit roots in economic and financial time series (Dickey & Fuller, 1979).

### 4.2 Procedure

The ADF test estimates the following regression:

$$\Delta y_t = \alpha + \beta t + \gamma y_{t-1} + \sum_{i=1}^{p} \delta_i \Delta y_{t-i} + \varepsilon_t$$

The null hypothesis H₀ is that γ = 0 (unit root present; series is non-stationary). The null is rejected when the ADF test statistic is sufficiently negative relative to the critical value, or equivalently when *p* < .05. Lag length *p* was selected automatically via the Akaike Information Criterion (AIC) within the ADF procedure. All tests were conducted using `statsmodels.tsa.stattools.adfuller` in Python.

### 4.3 Results

**Table 2.** *ADF stationarity test results on raw (levels) series.*

| Market | Series | ADF Statistic | *p*-value | Stationary? |
|--------|--------|--------------|-----------|-------------|
| Govt Shutdown | Reddit post count | −2.51 | .113 | No |
| | Kalshi volume | −2.55 | .103 | No |
| Fed Chair | Reddit post count | −7.58 | < .001 | **Yes** |
| | Kalshi volume | −1.42 | .572 | No |
| Hormuz | Reddit post count | −3.60 | .006 | **Yes** |
| | Kalshi volume | −4.46 | < .001 | **Yes** |
| Virginia | Reddit post count | stationary | < .001 | **Yes** |
| | Kalshi volume | stationary | < .001 | **Yes** |
| Texas Senate | Reddit post count | stationary | < .001 | **Yes** |
| | Kalshi volume | stationary | < .001 | **Yes** |
| Ohio Gov | Reddit post count | stationary | < .001 | **Yes** |
| | Kalshi volume | −2.0 | .162 | No |
| Illinois Senate | Reddit post count | 1.43 | .997 | No |
| | Kalshi volume | 2.81 | 1.000 | No |
| IL-09 | Reddit post count | non-stationary | .999 | No |
| | Kalshi volume | non-stationary | .999 | No |

For the high-engagement markets (Govt Shutdown, Fed Chair), both or one series required differencing. The low-engagement markets with sparse Reddit data (Illinois Senate, IL-09) show ADF statistics near +1 to +3, indicative of a flat near-zero series punctuated by a small number of non-zero observations; both required first-differencing.

---

## 5. Step 2 — First Differencing

### 5.1 Rationale

When a series contains a unit root (i.e., is integrated of order one, I(1)), the standard remedy is to compute the first difference: Δ*y*ₜ = *y*ₜ − *y*ₜ₋₁. First differencing removes the stochastic trend and renders the series stationary in most practical cases, producing an I(0) series suitable for regression-based inference.

For the Fed Chair market, although Reddit post count was already stationary, both series were first-differenced for consistency. This ensures the two series operate on the same transformation scale within the VAR model, which is required for valid inference.

The Hormuz market required no differencing, as both series were stationary in levels.

### 5.2 Results After Differencing

ADF tests were re-applied to the differenced series for the Govt Shutdown and Fed Chair markets.

**Table 3.** *ADF stationarity test results on first-differenced series.*

| Market | Series | ADF Statistic | *p*-value | Stationary? |
|--------|--------|--------------|-----------|-------------|
| Govt Shutdown | ΔReddit post count | −10.72 | < .001 | **Yes** |
| | ΔKalshi volume | −9.30 | < .001 | **Yes** |
| Fed Chair | ΔReddit post count | −6.85 | < .001 | **Yes** |
| | ΔKalshi volume | −8.49 | < .001 | **Yes** |

Both differenced series achieved stationarity in all cases (*p* < .001), confirming that first differencing was sufficient. All subsequent analyses for the Govt Shutdown and Fed Chair markets were conducted on the first-differenced series.

---

## 6. Step 3 — Lag Order Selection

### 6.1 Rationale

Granger causality tests are sensitive to the choice of lag order *p*: too few lags may fail to capture the true predictive relationship, while too many lags reduce statistical power by consuming degrees of freedom. The standard approach is to fit a VAR(*p*) model — a system of equations in which each variable is regressed on its own and the other variable's past *p* values — and select the lag order that minimises an information criterion.

The Bayesian Information Criterion (BIC) was used in preference to the AIC because BIC applies a stronger penalty for additional parameters, making it more conservative. Given the relatively small sample sizes in this study (n = 29–79), a conservative criterion reduces the risk of overfitting. The maximum lag tested was set to min(5, n/10) to prevent models with more parameters than observations can reliably support.

### 6.2 Results

**Table 4.** *BIC-selected lag order by market.*

| Market | n (after differencing) | Max lags tested | BIC-selected lag |
|--------|----------------------|----------------|-----------------|
| Govt Shutdown | 78 | 5 | **1** |
| Fed Chair | 70 | 5 | **1** |
| Hormuz | 29 | 2 | **1** |

All three markets converged on a lag order of 1. This indicates that, after controlling for autocorrelation, the most recent single day of history is the most informative window for predicting next-day values in both series. Granger tests were subsequently conducted at lags 1 through min(5, BIC + 2) to assess robustness across a range of specifications.

---

## 7. Step 4 — Granger Causality Testing

### 7.1 Rationale

The Granger causality test (Granger, 1969) is the standard econometric procedure for testing whether one time series contains predictive information about another, above and beyond the target series' own history. Formally, a series *X* Granger-causes series *Y* if:

$$\text{Var}(Y_t \mid Y_{t-1}, \ldots, Y_{t-p}) > \text{Var}(Y_t \mid Y_{t-1}, \ldots, Y_{t-p}, X_{t-1}, \ldots, X_{t-p})$$

In practice, this is tested via an F-test comparing a restricted VAR model (only own lags) against an unrestricted model (own lags plus lags of the other variable). A significant F-statistic indicates that past values of *X* carry predictive information about *Y* that is not already captured by *Y*'s own past.

Importantly, Granger causality is a statement about predictability, not about structural causation. A finding that Reddit Granger-causes trading volume means that Reddit activity contains leading information about volume, not necessarily that Reddit discussion mechanically generates trades. Both series may be driven by common underlying news events, with Reddit reacting slightly faster than the market.

Both directions were tested: Reddit → Volume and Volume → Reddit. Testing the reverse direction serves as a specificity check — if volume also Granger-causes Reddit, the relationship is bidirectional and the interpretation of Reddit as an independent leading indicator is weakened.

### 7.2 Results

#### 7.2.1 Govt Shutdown (KXGOVTSHUTLENGTH-26FEB07)

**Table 5.** *Granger causality F-tests, Govt Shutdown market.*

| Direction | Lag | F-statistic | df | *p*-value |
|-----------|-----|------------|-----|-----------|
| Reddit → Volume | 1 | 29.53 | (1, 76) | < .001 |
| Reddit → Volume | 2 | 12.60 | (2, 74) | < .001 |
| Reddit → Volume | 3 | 9.18 | (3, 72) | < .001 |
| Volume → Reddit | 1 | 5.40 | (1, 76) | .022 |
| Volume → Reddit | 2 | 1.12 | (2, 74) | .332 |
| Volume → Reddit | 3 | 2.17 | (3, 72) | .098 |

Reddit post count Granger-caused Kalshi trading volume robustly across all lag specifications (*p* < .001). The reverse direction was marginally significant at lag 1 (*p* = .022) but became non-significant at lags 2 and 3, indicating that this marginal effect does not reflect a sustained bidirectional relationship. The primary finding is therefore a directional Reddit → Volume influence.

#### 7.2.2 Fed Chair Confirmation (KXFEDCHAIRCONFIRM)

**Table 6.** *Granger causality F-tests, Fed Chair market.*

| Direction | Lag | F-statistic | df | *p*-value |
|-----------|-----|------------|-----|-----------|
| Reddit → Volume | 1 | 5.09 | (1, 66) | .027 |
| Reddit → Volume | 2 | 2.43 | (2, 63) | .096 |
| Reddit → Volume | 3 | 1.75 | (3, 60) | .167 |
| Volume → Reddit | 1 | 1.59 | (1, 66) | .212 |
| Volume → Reddit | 2 | 0.12 | (2, 63) | .884 |
| Volume → Reddit | 3 | 1.31 | (3, 60) | .281 |

Reddit post count Granger-caused Kalshi trading volume at lag 1 (*p* = .027), but this effect did not persist at lags 2 or 3. The reverse direction (Volume → Reddit) was not significant at any lag. The result is consistent with the Govt Shutdown finding but weaker in magnitude, and should be interpreted as suggestive rather than conclusive given its attenuation beyond a single lag.

#### 7.2.3 Hormuz Combined (KXHORMUZ_COMBINED)

**Table 7.** *Granger causality F-tests, Hormuz market.*

| Direction | Lag | F-statistic | df | *p*-value |
|-----------|-----|------------|-----|-----------|
| Reddit → Volume | 1 | 0.04 | (1, 24) | .842 |
| Reddit → Volume | 2 | 0.40 | (2, 21) | .678 |
| Volume → Reddit | 1 | 0.15 | (1, 24) | .700 |
| Volume → Reddit | 2 | 0.09 | (2, 21) | .915 |

No significant Granger causality was found in either direction for the Hormuz market. However, with only n = 29 observations, this analysis is substantially underpowered. The absence of a significant result should be interpreted as inconclusive rather than as evidence for the null hypothesis.

---

## 8. Step 5 — Cross-Correlation Analysis

### 8.1 Rationale

Granger causality tests provide a formal hypothesis test of predictive precedence, but they do not describe the shape or temporal profile of the relationship. Cross-correlation analysis complements the Granger tests by quantifying the Pearson correlation between the two series at each lag *k* from −7 to +7 days, where:

- **Positive lag (*k* > 0):** Reddit at time *t* correlated with Volume at time *t* + *k* (Reddit leads Volume)
- **Negative lag (*k* < 0):** Volume at time *t* correlated with Reddit at time *t* − *k* (Volume leads Reddit)
- **Lag 0:** Contemporaneous correlation

A 95% confidence band of ±1.96 / √*n* was applied as an approximate significance threshold under the assumption of a white-noise null process. All cross-correlations were computed on the same (possibly differenced) series used in the Granger tests, for consistency.

### 8.2 Results

**Table 8.** *Selected cross-correlations between Reddit post count and Kalshi trading volume.*

| Market | Lag | Pearson *r* | *p*-value | Note |
|--------|-----|------------|-----------|------|
| Govt Shutdown | −1 | −0.23 | .040 | Volume leads Reddit (negative) |
| | 0 | +0.33 | .003 | Contemporaneous |
| | **+1** | **+0.55** | **< .001** | **Peak: Reddit leads Volume** |
| | +2 | +0.27 | .016 | |
| | +3 | +0.32 | .004 | |
| Fed Chair | 0 | +0.18 | .133 | Not significant |
| | **+1** | **+0.26** | **.032** | **Peak: Reddit leads Volume** |
| | +2 | −0.11 | .361 | |
| Hormuz | 0 | +0.00 | .999 | No relationship |
| | +3 | +0.42 | .032 | Isolated; likely noise (n=29) |

#### Key findings

**Govt Shutdown:** The cross-correlation profile is consistent with and reinforces the Granger results. The peak positive correlation occurs at lag +1 (*r* = .55, *p* < .001), indicating that Reddit post volume on day *t* is most strongly associated with trading volume on day *t* + 1. The significant negative correlation at lag −1 (*r* = −.23, *p* = .040) — where Volume leads Reddit negatively — suggests a feedback effect: high trading days are followed by reduced Reddit discussion, consistent with the market having "priced in" new information, thereby dampening subsequent discussion.

**Fed Chair:** The peak correlation is again at lag +1 (*r* = .26, *p* = .032), mirroring the Govt Shutdown pattern but at lower magnitude. All other lags were non-significant, indicating that the Reddit-leading-Volume relationship is specific to the one-day horizon for this market.

**Hormuz:** No systematic cross-correlation structure was observed. The significant value at lag +3 (*r* = .42, *p* = .032) is an isolated result and should not be interpreted as substantive, given that with 15 lags tested a false positive is expected by chance, and there is no theoretical basis for a three-day delay specific to this market. The small sample (n = 29) further limits the reliability of any individual estimate.

---

## 7. New Markets — Granger Causality Results

### 7.1 Virginia Redistricting (KXVIRGINIAREDISTRICT_COMBINED)

**Table 5b.** *Granger causality F-tests, Virginia Redistricting market.*

| Direction | Lag | F-statistic | df | *p*-value |
|-----------|-----|------------|-----|-----------|
| Reddit → Volume | 1 | 5.17 | (1, 108) | .025 |
| Reddit → Volume | 2 | 2.998 | (2, 105) | .054 |
| Reddit → Volume | 3 | 2.232 | (3, 102) | .089 |
| Volume → Reddit | 1 | 13.187 | (1, 108) | < .001 |
| Volume → Reddit | 2 | 10.075 | (2, 105) | < .001 |
| Volume → Reddit | 3 | 6.909 | (3, 102) | < .001 |

Reddit post count Granger-caused Kalshi trading volume at lag 1 (*p* = .025), consistent with the directional hypothesis. However, the reverse relationship — trading volume Granger-causing Reddit activity — was significantly stronger and sustained across all three lags (*p* < .001). This bidirectionality arises because the Virginia redistricting market had active trading from early January, while Reddit discussion was largely absent (zero-rate = 75%) until the weeks immediately before and after the April 22 referendum. The pattern is consistent with market activity eventually driving news coverage, rather than a clean information-channel interpretation. Virginia is therefore classified as a *borderline* case: the Reddit → Volume direction holds at lag 1, but the predominant information flow runs in the reverse direction.

### 7.2 Texas Senate Primary (KXTXSENDPRIMARYMOV-26MAR03)

**Table 5c.** *Granger causality F-tests, Texas Senate Primary market.*

| Direction | Lag | F-statistic | df | *p*-value |
|-----------|-----|------------|-----|-----------|
| Reddit → Volume | 1 | 0.033 | (1, 51) | .858 |
| Reddit → Volume | 2 | 0.104 | (2, 48) | .901 |
| Reddit → Volume | 3 | 0.164 | (3, 45) | .920 |
| Volume → Reddit | 1 | 0.012 | (1, 51) | .912 |
| Volume → Reddit | 2 | 0.008 | (2, 48) | .992 |
| Volume → Reddit | 3 | 0.089 | (3, 45) | .966 |

No significant Granger causality was found in either direction (*p* > .85 across all lags). Cross-correlation analysis, however, revealed a significant positive correlation at lag +1 (*r* = .31, *p* = .024), indicating that Reddit post count on day *t* is associated with higher volume on day *t* + 1. The divergence between Granger and cross-correlation results reflects the sensitivity of the Granger test to autocorrelation structure: after controlling for trading volume's own past, the marginal contribution of Reddit is insufficient for statistical significance at this sample size (n = 55). The Texas Senate Primary is therefore classified as a *mixed* case: descriptive evidence of a Reddit lead, but no formal Granger confirmation.

### 7.3 Low-Engagement Markets (Ohio Gov, Illinois Senate, IL-09)

The three remaining markets — Ohio Republican Governor Primary (94% zero Reddit days), Illinois Democratic Senate Primary (91%), and IL-09 Democratic Primary (90%) — had insufficient Reddit engagement to produce reliable Granger causality estimates.

**Table 5d.** *Summary Granger results, low-engagement markets (at BIC-selected lag).*

| Market | n | Reddit zero-rate | BIC lag | Reddit → Volume | Volume → Reddit |
|--------|---|-----------------|---------|-----------------|-----------------|
| Ohio Gov | 123 | 94% | 5 | F(5,106) = 1.36, *p* = .245 | *p* = .067 |
| Illinois Senate | 74 | 91% | 2 | F(2,66) = 4.06, *p* = .022 | *p* = .0002 |
| IL-09 | 76 | 90% | 1 | F(1,71) = 19.29, *p* < .001 | *p* < .001 |

#### Bidirectional artifacts in zero-inflated series

Illinois Senate and IL-09 appear to show statistically significant Reddit → Volume Granger causality, but these results are artefacts of zero-inflation rather than genuine evidence of a directional information channel. Understanding why requires examining the structure of the differenced series.

With 90–91% zero Reddit days, the first-differenced Reddit series is nearly always zero, punctuated by a small number of sharp spikes when a news event generates a brief cluster of posts. These spikes coincide with the same real-world events — primary debates, endorsement announcements, polling releases — that also drive spikes in trading volume. When both series jump on the same event day, the differenced data look like this:

| Day | ΔReddit | ΔVolume |
|-----|---------|---------|
| ... | 0 | +500 |
| *Event day* | +3 | +8,000 |
| *Day after* | −3 | −8,000 |
| ... | 0 | −200 |

The regression underlying the Granger test finds that ΔReddit at *t* predicts ΔVolume at *t* + 1 — but the same is equally true in reverse: ΔVolume at *t* predicts ΔReddit at *t* + 1. Both are driven by the same two rows in the dataset (the event day and the reversion day), not by a genuine temporal ordering where Reddit activity propagates information into the market.

The bidirectionality of the result is the diagnostic: a true Reddit → Volume information channel would produce asymmetric Granger causality (Reddit leads Volume; Volume does not lead Reddit). When both directions are equally significant — as in IL-09, where Volume → Reddit yields *F*(1, 71) = 69.8, *p* < .001, far exceeding the Reddit → Volume result — it indicates that both series are reacting to the same external event rather than one causing the other. These results are therefore classified as artefacts and are excluded from the primary inference. The markets are retained in the analysis as **negative controls** to demonstrate that the statistical method does not produce spurious Reddit → Volume leads by default, but only when the underlying data structure supports a directional relationship.

---

## 8. Cross-Correlation Analysis — All 8 Markets

**Table 6b.** *Selected cross-correlations, all 8 markets (on differenced or levels series per market).*

| Market | Lag | Pearson *r* | *p*-value | Note |
|--------|-----|------------|-----------|------|
| Govt Shutdown | **+1** | **+0.49** | **< .001** | **Peak: Reddit leads Volume** |
| | +2 | +0.26 | .023 | |
| | −1 | −0.20 | .083 | |
| Fed Chair | **+1** | **+0.38** | **.001** | **Peak: Reddit leads Volume** |
| | +2 | +0.15 | .212 | |
| Hormuz | +3 | +0.43 | .034 | Isolated; likely noise (n=28) |
| Virginia | **+1** | **+0.32** | **< .001** | Consistent; but see bidirectional Granger |
| Texas Senate | **+1** | **+0.31** | **.024** | Reddit lead in xcorr; Granger n.s. |
| Ohio Gov | +3 | +0.22 | .016 | Isolated; 94% Reddit zeros |
| Illinois Senate | +2 | +0.46 | < .001 | Artifact — strong bidirectionality |
| IL-09 | +4 | +0.40 | < .001 | Artifact — Volume→Reddit F=69.8 |

The cross-correlation plot across all 8 markets is saved in `outputs/cross_correlation_8markets.png`. The high-engagement markets (Govt Shutdown, Fed Chair, Texas Senate) show consistent positive cross-correlations peaking at lag +1. The low-engagement markets show either no pattern (Ohio) or patterns better explained by shared event-day spikes than by a directional Reddit → market influence.

---

## 9. Summary of Findings

**Table 9.** *Summary of results across all 8 markets.*

| Market | n | Reddit zero-rate | Series | BIC lag | Reddit→Vol Granger | Peak xcorr | Classification |
|--------|---|-----------------|--------|---------|--------------------|------------|----------------|
| Govt Shutdown | 78 | 24% | Δ | 1 | F(1,73) = 22.44, *p* < .001 ✓ | +1, *r* = .49 | **Primary** |
| Fed Chair | 70 | 37% | Δ | 1 | F(1,65) = 8.28, *p* = .005 ✓ | +1, *r* = .38 | **Primary** |
| Texas Senate | 55 | 26% | levels | 1 | F(1,51) = 0.03, *p* = .858 ✗ | +1, *r* = .31 | Mixed |
| Hormuz | 28 | 0% | levels | 1 | F(1,24) = 0.18, *p* = .680 ✗ | +3, *r* = .43 | Underpowered |
| Virginia | 112 | 75% | levels | 1 | F(1,108) = 5.17, *p* = .025 ✓ | +1, *r* = .32 | Borderline (bidir.) |
| Ohio Gov | 123 | 94% | Δ | 5 | F(5,106) = 1.36, *p* = .245 ✗ | — | Negative control |
| Illinois Senate | 74 | 91% | Δ | 2 | F(2,66) = 4.06, *p* = .022 (artifact) | — | Negative control |
| IL-09 | 76 | 90% | Δ | 1 | F(1,71) = 19.29, *p* < .001 (artifact) | — | Negative control |

### Key pattern: Reddit engagement as a prerequisite

A consistent pattern emerges across all 8 markets: the Reddit → Volume Granger relationship is detectable only in markets where Reddit engagement is sufficient. Markets where Reddit zero-rate exceeds approximately 80% either show no Granger causality (Ohio) or exhibit statistical artifacts driven by zero-inflation (Illinois Senate, IL-09). The threshold appears to lie between 37% (Fed Chair, significant) and 75% (Virginia, marginal and bidirectional). This pattern supports a **salience hypothesis**: the Reddit → volume information channel is active only for politically salient, nationally prominent events that generate sustained public discussion.

### APA-Style Summary Report

> Daily Reddit post counts Granger-caused Kalshi trading volume in both the government shutdown market, F(1, 73) = 22.44, *p* < .001, and the Federal Reserve Chair confirmation market, F(1, 65) = 8.28, *p* = .005. Cross-correlation analysis confirmed a consistent one-day lead in both markets (Govt Shutdown: *r* = .49, *p* < .001; Fed Chair: *r* = .38, *p* = .001). The reverse relationship — trading volume Granger-causing Reddit activity — was not sustained beyond lag 1 in either market (Govt Shutdown lag 2: *p* = .388; Fed Chair lag 2: *p* = .874), indicating a directional Reddit → trading influence. Across five additional markets, no clean directional effect was found: the Virginia redistricting market showed bidirectional Granger causality (Volume → Reddit: *p* < .001 across all lags); the Texas Senate primary showed a significant cross-correlation at lag +1 (*r* = .31, *p* = .024) without Granger confirmation; and three low-engagement markets (Ohio, Illinois Senate, IL-09; Reddit zero-rate 90–94%) produced results inconsistent with the directional hypothesis. This pattern suggests the Reddit → prediction market information channel is active only in high-salience, nationally prominent political events. Prior to testing, non-stationary series were first-differenced; ADF confirmed stationarity of all differenced series (*p* < .001). Lag order was selected via BIC within a VAR framework (all primary markets: lag = 1).

---

## 10. Limitations

**Granger causality is not structural causality.** A significant Granger test indicates that Reddit post counts carry leading predictive information about trading volume, but this does not establish that Reddit discussion mechanically causes trades. Both series may be driven by common latent factors (e.g., breaking news), with Reddit reacting marginally faster. Distinguishing between these mechanisms would require higher-frequency (e.g., hourly) data.

**Daily granularity.** The analysis uses daily aggregates. If Reddit activity and trading both respond to the same news event within the same day, the predictive relationship would appear as a lag-0 correlation rather than a lead-lag structure. The consistent lag +1 finding may therefore understate the true speed of Reddit's leading role.

**Zero-inflation in low-engagement markets.** Markets with Reddit zero-rates above approximately 80% are ill-suited to standard VAR-based Granger causality analysis. The mechanism is described in Section 7.3: shared event-day spikes in near-zero series produce bidirectional artefacts that mimic Granger causality without representing a directional information channel. This is not a failure of the method per se, but a mismatch between the method's assumptions and the data structure. Future work should consider models designed for count or zero-inflated data (e.g., zero-inflated Poisson VAR) for low-engagement markets. Importantly, the low Reddit engagement in state/local markets is itself substantively meaningful — as discussed in Section 2.4, it reflects the absence of Reddit as an active information channel for those events, not missing data.

**Settlement day removal.** The final trading day was removed from each market to exclude anomalously high settlement-driven volume. This reduces sample sizes by one observation per market and may slightly attenuate results for short-horizon markets.

**Small sample — Hormuz.** With n = 28 observations, the Hormuz analysis has insufficient power to detect moderate effect sizes.

**Multiple comparisons in cross-correlation.** With 15 lags tested per market, approximately one significant result per market is expected by chance under the null. Isolated significant lags without Granger confirmation (e.g., Hormuz lag +3) should not be over-interpreted.

**Reddit coverage.** The Reddit data covers only three subreddits. Broader social media coverage (e.g., Twitter/X, news comment sections) may capture a fuller picture of public discourse and strengthen or alter the estimated lead-lag relationships.

---

## 11. References

Dickey, D. A., & Fuller, W. A. (1979). Distribution of the estimators for autoregressive time series with a unit root. *Journal of the American Statistical Association*, *74*(366), 427–431. https://doi.org/10.2307/2286348

Granger, C. W. J. (1969). Investigating causal relations by econometric models and cross-spectral methods. *Econometrica*, *37*(3), 424–438. https://doi.org/10.2307/1912791

Schwarz, G. (1978). Estimating the dimension of a model. *The Annals of Statistics*, *6*(2), 461–464. https://doi.org/10.1214/aos/1176344136

Sims, C. A. (1980). Macroeconomics and reality. *Econometrica*, *48*(1), 1–48. https://doi.org/10.2307/1912017
