# Short-Term Building Load Forecasting Service

**Service 5 — Technical Description**
Municipality of Athens

---

## 1. Service Description

Service 5 gives the Municipality of Athens a forward-looking operational tool for anticipating electricity consumption across its public building portfolio at short time horizons. Where Services 1 and 2 act on consumption as it occurs, Service 5 acts on consumption **before** it occurs, generating building-level load forecasts that enable proactive scheduling decisions rather than reactive responses.

Accurate short-term forecasts are the informational foundation on which demand response optimisation, PV self-consumption scheduling and anomaly detection baselines all depend, making this service a **critical enabler for the broader service portfolio** rather than a standalone analytical product.

The service produces **probabilistic load forecasts at 15-minute resolution** for horizons spanning from one hour to twenty-four hours ahead, covering every smart-metered building in the municipal estate and refreshing continuously as new meter readings, weather observations and occupancy information become available.

---

## 2. Operational Context

The service operates across the full portfolio of smart-metered municipal buildings, each presenting a distinct load shape governed by its use category, occupancy schedule, thermal characteristics and sensitivity to outdoor conditions:

- **Administrative offices** exhibit pronounced weekday patterns tied to working hours
- **Schools** show sharp morning ramp-up profiles modulated by term dates and break periods
- **Sports and cultural facilities** display irregular, event-driven consumption patterns that are harder to anticipate from schedule data alone

This heterogeneity means a single forecasting model architecture applied uniformly across the portfolio would perform well for some building types and poorly for others. The service therefore maintains a **dedicated forecasting model per building**, trained and updated on that building's own consumption history and calibrated to its specific contextual drivers.

---

## 3. Data Inputs

| Input | Resolution | Role |
|---|---|---|
| Real-time consumption (Service 1 core datasets) | 15 min | Primary training and input signal per building |
| Building metadata (Service 1 core datasets) | Static | Use category, area and characteristics informing model selection and calibration |
| Weather observations & forecasts | 15 min / hourly | Outdoor condition drivers of thermally sensitive loads |
| Calendar & occupancy features | Static / periodic | Time-of-day and day-of-week encodings, Greek public holidays, term dates, operating schedules |
| PV generation data | 15 min | Net consumption forecasting for buildings with installed PV |

---

## 4. Methodology and Technical Approach

Each building-level forecasting model is trained on a **rolling historical window of up to twelve months** of hourly or 15-minute consumption data, updated continuously as new observations arrive. Model inputs include recent consumption history, time-of-day and day-of-week encodings, calendar and occupancy features.

The forecasting approach is **selected per building** based on the length and quality of the available history, the strength of identifiable consumption patterns and the volatility of the residual after accounting for known drivers:

- Buildings with stable, regular consumption profiles are well served by structured time-series models that exploit their predictable periodicity
- Buildings with irregular or event-driven consumption require more flexible approaches that can accommodate high variability and weaker autocorrelation structure

Buildings undergoing operational changes — schedule modifications, system replacements or occupancy regime changes — are flagged for **accelerated model recalibration** to prevent the historical training window from anchoring forecasts to a consumption regime that no longer applies.

Meter data quality issues, including gaps, resets and implausible readings, are handled at ingestion and do not propagate into the training data or forecast inputs.

---

## 5. Service Outputs

For each building and forecast cycle, the service produces:

- A time series of predicted consumption values at hourly or 15-minute resolution covering the forecast horizon
- Model version, training data window, input feature set and data quality flags applicable at the time of generation, ensuring full traceability

---

## 6. Evaluation and Validation

Forecast accuracy is evaluated continuously using a **rolling walk-forward protocol** in which each forecast is assessed against the actual consumption observed in the corresponding interval, using only the information available at the time the forecast was generated.

| Indicator | Description |
|---|---|
| **Mean absolute error (MAE)** | Computed at building level, stratified by building category, forecast horizon and time of day |
| **Mean absolute percentage error (MAPE)** | Computed at building level with the same stratification, identifying where forecast quality is strongest and where it degrades |

Forecast performance is reported in the service's operational dashboard.

---

## 7. Implementation and Availability

The service is exposed through an **authenticated REST API** delivering building-level and portfolio-level load forecasts, confidence intervals, net consumption forecasts and model performance summaries in structured JSON format, with a **push notification mechanism** alerting consuming services when updated forecasts are available for a given building or horizon.

The forecasting pipeline is structured as independent, versioned modules:

1. Data ingestion and quality control
2. Feature engineering
3. Model training and updating
4. Forecast generation
5. Performance monitoring

Each module is versioned and logged to ensure reproducibility of all historical forecasts.
