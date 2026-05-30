# EWE-Conf
Extreme Weather Event Confidence Calibration Benchmark

## Task Description
EWE-Conf is a benchmark created to evaluate the accuracy of LLMs and their ability to predict global weather patterns and extreme weather events and give a calculation of confidence. It is important for AI to accurately predict extreme weather, as they can highly impact hundreds to thousands of lives. A model should be reliable enough that scientists can trust the results to increase readiness and reduce disaster. A model that cannot give an estimated confidence in an output would not be meaningful, as scientists would not be able to calculate risk based on it.

Multiple LLMS (Claude, ChatGPT, Gemini) will be evaluated by comparing results to ground-truth. A total of 50 historical weather events were compiled and given the same prompt to predict the weather state (temperature, wind speed/direction, mean sea-level pressure) and give an extreme event prediction (yes/no + severity level) and a confidence level, given the weather state 6 hours prior.

These items were then scored against the ground-truth weather states and confidence levels, based on correctness, severity alignment, and calibration score.

## Data Schema
### Ground Truth File — `extreme_weather_events_50.xlsx`
| Column | Type | Description |
|--------|------|-------------|
| `Item ID` | `int` | Unique event identifier, 1–50 |
| `Event Name` | `str` | Common name of the historical event |
| `Event Date` | `str` | Date of event landfall or peak impact |
| `Event Type` | `str` | Meteorological classification
| `Location` | `str` | Specific observation point at landfall or impact site |
| `Latitude` | `float` | Decimal degrees, WGS84 |
| `Longitude` | `float` | Decimal degrees, WGS84 |
| `Severity` | `str` | Four-tier ordinal scale based on observed fatalities, damage, and meteorological intensity. Distribution: EXTREME 19, HIGH 18, MEDIUM 8, LOW 5 |
| `Severity Reasoning` | `str` | Justification citing observed peak intensity, casualties, or damage — e.g. *"Cat 3 at landfall (peaked Cat 5 over Gulf); 1,833 deaths; $125B damage"* |
| `GT Confidence (%)` | `int` | Retrospective forecast confidence derived from operational NWP skill scores, track errors, and ensemble spread at T-6h.|
| `Confidence Methodology` | `str` | Source and method used to derive the GT confidence — e.g. *"NHC 2005 Annual Verification Report; track error ~55 km (near 5-yr mean)"* |
| `Forecast Timestamp (T-6h)` | `str` | UTC/local timestamp of the precursor observation |
| `Forecast Source (T-6h)` | `str` | Operational advisory or station that issued the T-6h observation |
| `Temperature (T-6h)` | `str` | Surface or low-level temperature at the observation point, dual-unit |
| `Wind Speed/Direction (T-6h)` | `str` | Sustained wind speed, direction of movement, and storm motion vector |
| `MSLP (T-6h)` | `str` | Mean sea-level pressure at T-6h |
| `At-event Timestamp (T+0)` | `str` | Timestamp of peak/landfall observation |
| `At-event Source (T+0)` | `str` | Advisory or observation source for T+0 data |
| `Temperature (T+0)` | `str` | Surface temperature at event time |
| `Wind Speed/Direction (T+0)` | `str` | Peak sustained winds and motion at event time |
| `MSLP (T+0)` | `str` | Mean sea-level pressure at event time |
| `Source Links` | `str` | Pipe-delimited URLs to primary verification sources |

### Prediction Files — Model Output Schema

Each model was given only the T-6h input columns and asked to generate the following predictions. Column names varied across models; the consolidated file standardises all three to the schema below.

| Column | Type | Description |
|--------|------|-------------|
| `Item ID` | `int` | Matches ground truth `Item ID` |
| `Event Type` | `str` | Input echo — event type given to the model |
| `Location` | `str` | Input echo — observation location |
| `Latitude` | `float` | Input echo |
| `Longitude` | `float` | Input echo |
| `Temperature (T-6h)` | `str` | Input echo |
| `Wind (T-6h)` | `str` | Input echo |
| `MSLP (T-6h)` | `str` | Input echo |
| `Event Will Occur` | `str` | Model's binary prediction: `Yes` / `No`|
| `Affected Region` | `str` | Free-text description of the geographic area predicted to be impacted |
| `Severity` | `str` | Predicted severity: `Extreme` / `High` / `Medium` / `Low` |
| `Forecast Confidence (%)` | `int` | Model's self-reported confidence, normalised to 0–100 |
| `At-Event Temperature` | `str` | Predicted temperature at event time |
| `At-Event Wind` | `str` | Predicted wind speed/direction at event time |
| `At-Event MSLP` | `str` | Predicted MSLP at event time (hPa) |

## Prompt & Rubric Design
### Prompt
For the LLMs Claude Sonnet 4.6, ChatGPT 5.5, and Google Gemini 3, the file `extreme_weather_events_50_no_gt.xlsx' without ground-truth confidence and event name, and the following prompt was fed in:
```
For the given temperature, wind speed and direction, MSLP, and location 6 hours prior, predict whether each item's event type will occur, the affected region, its severity from low/medium/high/extreme, a forecast confidence, and the at-event conditions with the same variables.
```

### Rubric
Each predicted event is scored on three independent sub-dimensions. Scores are summed to a **maximum of 4.0 points**.

```
Total Score = Correctness + Severity Alignment + Calibration Score
```

---

### Dimension 1 — Correctness `(0–2)`

Evaluates whether the model correctly identified **what** happened and **where**.

| Score | Condition |
|-------|-----------|
| `2` | Correct event type **AND** generally correct affected region |
| `1` | Correct event type **OR** generally correct affected region (not both) |
| `0` | Wrong event type **AND** wrong affected region |

**Region matching method:** Token-overlap between the predicted affected region and the ground truth location string, after removing stop words (`the`, `and`, `of`, `in`, `county`, `parish`, `city`, `usa`, `united`, `states`, `area`, `region`, `province`, `district`, `island`). A match is declared if any non-stop content token appears in both strings.

**Example — Score 2:**
> GT Location: `"Buras, Plaquemines Parish, Louisiana, USA"`
> Predicted Region: `"Plaquemines Parish & SE Louisiana coast"`
> → Token overlap on `plaquemines`, `louisiana` → region ✓ | event type ✓ → **2**

**Example — Score 1:**
> GT Event: `"Tornado Outbreak"` | Predicted: `"Tornado Outbreak"` ✓
> GT Location: `"Tuscaloosa, Alabama"` | Predicted Region: `"Central United States"` ✗
> → **1**

---

### Dimension 2 — Severity Alignment `(0–1)`

Evaluates how close the predicted severity tier is to the ground truth on the four-level ordinal scale.

**Severity scale (ordered):**


| Score | Condition |
|-------|-----------|
| `1.0` | Exact match — same tier |
| `0.5` | One tier off |
| `0.0` | Two or more tiers off |

**Example — Score 1.0:**
> GT: `EXTREME` | Predicted: `Extreme` → **1.0**

**Example — Score 0.5:**
> GT: `MEDIUM` | Predicted: `High` → 1 tier off → **0.5**

**Example — Score 0.0:**
> GT: `LOW` | Predicted: `Extreme` → 3 tiers off → **0.0**

---

### Dimension 3 — Calibration Score `(0–1)`

Evaluates how well the model's self-reported confidence tracks the ground truth forecastability score.

$$\text{Calibration} = 1 - \frac{|\text{Predicted Confidence} - \text{GT Confidence}|}{100}$$

**Examples**
| Predicted Conf | GT Conf | \|Δ\| | Score |
|---------------|---------|--------|-------|
| 97% | 92% | 5 | 0.95 |
| 99% | 75% | 24 | 0.76 |
| 88% | 35% | 53 | 0.47 |
| 60% | 60% | 0 | 1.00 |

---

## Results Summary

| Model | N | Mean Correctness | Mean Sev. Alignment | Mean Calibration | **Mean Total** |
|-------|---|-----------------|--------------------|--------------------|----------------|
| ChatGPT | 50 | 2.000 | 0.680 | 0.764 | **3.444** |
| Claude | 50 | 2.000 | 0.622 | 0.724 | **3.346** |
| Gemini | 50 | 1.780 | 0.700 | 0.823 | **3.303** |

### Limitations

- **LLM Incosistency:** General LLMs may not output data structure the way I would like if not prompted correctly. Good prompt engineering is required to get desired data.
- **Misuse Risk:** Miscalibrated bencmarks could influence real decisions. This benchmark was not created by an expert and should not be used in place of experts in the field.
