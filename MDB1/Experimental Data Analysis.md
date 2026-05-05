# Analysis of TAL220B + HX711 Experimental Data 

## 1. Available Data

The current data folder contains three experimental data groups. Each group includes four signals:

```text
force_N
raw_adc
valid
saturated
```

| Data Group | Main Files | Duration | Interpretation |
|---|---|---:|---|
| `noload` | `force_n_noload.mat`, `raw_adc_noload.mat`, `valid_noload.mat`, `saturated_noload.mat` | about 45 min | No-load zero drift data |
| `2514` | `force_2514.mat`, `raw_adc_2514.mat`, `valid_2514.mat`, `saturated_2514.mat` | about 34.3 min | Approximately 24.26 N constant-load data, usable as preliminary creep / stability data |
| `2995_0` | `force_2995_0.mat`, `raw_adc_2995_0.mat`, `valid_2995_0.mat`, `saturated_2995_0.mat` | about 120 min | Load-then-unload data, including an approximately 29.36 N loaded segment and an unloaded zero-return segment |

Data quality:

- `saturated = 0` for all datasets, so HX711 full-scale saturation did not occur.
- `noload` and `2514` have about 10% valid samples, indicating that the logged signal rate is higher than the HX711 valid update rate. Post-processing must therefore use only `valid == 1`.
- `2995_0` has about 99.9% valid samples and is the most complete and useful dataset.

## 2. Temperature Record and Its Meaning

The handwritten temperature record is:

| Time | Temperature |
|---:|---:|
| 0 s | 24.8°C |
| 1820 s | 24.6°C |
| 3380 s | 24.6°C |
| 7200 s | 24.5°C |

The total temperature change over 7200 s is:

```text
Delta T = 24.5 - 24.8 = -0.3°C
```

According to the TAL220B datasheet, the zero temperature coefficient and span temperature coefficient are typically specified as:

```text
±0.05 %FS / 10°C
```

For a 0.3°C temperature change, the maximum estimated temperature-related effect is approximately:

```text
0.05 %FS / 10°C * 0.3°C = 0.0015 %FS
```

For the span temperature effect, the relative change is approximately:

```text
0.0015 % of reading
```

For the approximately 29.36 N loaded data, the force error caused by span temperature variation is approximately:

```text
29.36 N * 0.0015 % = 0.00044 N
```

Therefore, in today's dataset, the temperature variation is too small to explain the approximately `0.1045 N` change observed in `2514`, or the approximately `0.0286 N` change observed in the loaded segment of `2995_0`. The dominant observed changes are more likely caused by:

- load cell / fixture mechanical creep;
- post-loading mechanical settling;
- small looseness or stress relaxation in the connection structure;
- zero-return behavior after unloading;
- low-frequency electrical or data acquisition drift.

## 3. How to Use the No-Load Zero Drift Data

The `noload` data should be used for zero-drift and deadband estimation.

Important note: in this dataset, `force_N` is essentially zero because the firmware deadband suppresses small force values. Therefore, zero drift must be analyzed from `raw_adc`, not from `force_N`.

Current analysis results:

| Metric | Value |
|---|---:|
| Duration | about 45 min |
| First 60 s raw mean | 128159.114 counts |
| Last 60 s raw mean | 127953.254 counts |
| Raw drift | -205.860 counts |
| Equivalent tension drift | -0.00458 N |
| Drift slope | -0.000160 N/min |
| Last 60 s noise standard deviation | 0.00444 N |

Uses:

1. Estimate the zero-drift trend.
2. Set the deadband parameter.
3. Determine whether slow zero tracking is needed during ground idle state.
4. Define the pre-flight zero-check threshold.

Recommended deadband rule:

```text
deadband_N ≈ 3 * sigma_zero_N
```

Current estimate:

```text
deadband_N ≈ 3 * 0.00444 N = 0.0133 N
```

With engineering margin, a practical value is:

```text
deadband_N = 0.02 N
```

This is consistent with the current firmware setting.

Reference basis:

- HX711 datasheet: valid and invalid ADC readings must be distinguished.
- TAL220B datasheet: zero output has temperature-related and zero-offset errors.
- NIST Handbook 44: automatic zero tracking must be limited and must not be applied under real load.

## 4. How to Use the `2514` Constant-Load Data

The `2514` dataset can be used as preliminary constant-load stability or creep data.

Current analysis results:

| Metric | Value |
|---|---:|
| Loaded duration | about 34.29 min |
| Median force | 24.257 N |
| Mean force | 24.263 N |
| First 60 s mean | 24.266 N |
| Last 60 s mean | 24.370 N |
| Force change | +0.1045 N |

The change is approximately:

```text
0.1045 / 24.26 ≈ 0.43 % of measured load
```

This change is much larger than the magnitude that can be explained by the recorded temperature variation. Therefore, it should not be simply attributed to temperature. It should be treated as preliminary evidence of system-level loaded stability, creep, fixture settling, or stress relaxation.

Limitations:

- The duration is less than 1 hour.
- The actual applied mass was not explicitly recorded.
- The load application time was not marked.
- Creep, mechanical settling, and low-frequency electrical drift cannot be separated cleanly.

Uses:

1. Use as preliminary observation of loaded stability.
2. Use to show that the real system exhibits slow variation after loading.
3. Do not use as the only final quantitative creep result.

Reference basis:

- TAL220B datasheet: the sensor has a specified creep behavior.
- OIML R 60: creep should be tested under constant load and stable conditions.
- JCGM 100 / GUM: this variation should be included in uncertainty or limitation statements.

## 5. How to Use the `2995_0` Load-Unload Data

`2995_0` is the most valuable dataset currently available because it includes both an approximately 1-hour loaded segment and an approximately 55-minute unloaded recovery segment.

### 5.1 Loaded Segment

Current analysis results:

| Metric | Value |
|---|---:|
| Loaded duration | about 63.16 min |
| Median force | 29.357 N |
| Mean force | 29.357 N |
| First 60 s mean | 29.377 N |
| Last 60 s mean | 29.348 N |
| Force change | -0.0286 N |

The change is approximately:

```text
0.0286 / 29.36 ≈ 0.098 % of measured load
```

This dataset is the most reliable constant-load creep / stability dataset from the current experiment.

Since the temperature changed by only about 0.3°C over the full 7200 s, the loaded-segment change should not mainly be attributed to temperature.

### 5.2 Unloaded Zero-Return Segment

Current analysis results:

| Metric | Value |
|---|---:|
| Recovery duration | about 55.00 min |
| First 60 s recovery mean | -0.0346 N |
| Last 60 s recovery mean | -0.0432 N |
| Recovery change | -0.00863 N |

Interpretation:

- After unloading, the reading is close to zero but not exactly zero.
- The recovery stage still shows slow variation.
- It should not be assumed that the sensor immediately returns to the original zero after sustained loading.

Uses:

1. Define post-flight zero-check strategy.
2. Show that a settling period or re-tare may be required after unloading.
3. Support the design rule that zero tracking should only be enabled on the ground when the tether is confirmed to be slack.

Reference basis:

- TAL220B datasheet: creep and zero-return related errors exist.
- OIML R 60: load-cell tests should consider zero return after loading.
- NIST Handbook 44: automatic zero adjustment must be limited.
- JCGM 100 / GUM: zero-return error should be included in the system error budget.

## 6. Is It Acceptable to Only Add a Multi-Point Static Calibration?

Yes, but the limitations must be stated clearly.

If only one additional experiment is performed, the current data coverage becomes:

| Item | Current Status | Comment |
|---|---|---|
| Zero drift | Mostly available | 45 min no-load raw ADC data can estimate zero drift and deadband |
| Constant-load creep | Mostly available | `2995_0` has about 63 min loaded data; `2514` can be used as supporting evidence |
| Zero-return recovery | Mostly available | `2995_0` has about 55 min unloaded recovery data |
| Temperature record | Mostly available | Temperature variation is small, supporting the conclusion that temperature is not the dominant effect |
| Multi-point calibration | Missing | Must be added |
| Linearity error | Missing | Requires multi-point calibration |
| Hysteresis | Missing | Requires loading / unloading calibration |
| Repeatability | Insufficient | If not repeated, it must be treated conservatively |
| Dynamic response | Missing | Can be stated as a limitation if the project does not focus on fast protection |

Therefore, if only one additional experiment is added, the priority should be:

```text
multi-point static loading / unloading calibration
```

Recommended load points:

```text
0 %, 10 %, 20 %, 40 %, 60 %, 80 % of selected safe full-scale load
```

Hold each load point for 5 minutes and use the final 60 s average.

After this experiment, the following can be obtained:

- actual `counts_per_N`;
- zero offset;
- linearity error;
- loading / unloading hysteresis;
- stable noise at each load point;
- location of the existing 24 N and 29 N data on the calibration curve.

## 7. Consequences of Not Collecting Other Data

### Not Adding a Full 1-Hour No-Load Zero Drift Test

Consequences:

- The current no-load dataset is only 45 min.
- Zero drift can still be estimated, but it should not be called a strict 1-hour zero drift test.
- In the report, it should be described as a “45 min no-load zero drift observation”.

Impact level: medium.  
Reason: the current zero drift is small and raw ADC data are available.

### Not Adding a Multi-Temperature Test

Consequences:

- `k_zero_T` and `k_span_T` cannot be measured experimentally.
- A system-level temperature compensation model cannot be identified.
- The TAL220B datasheet must be used as a Type B uncertainty source.

Impact level: low to medium.  
Reason: today's temperature variation is only about 0.3°C, so the temperature effect can be estimated from the datasheet and shown to be small.

### Not Adding Repeatability Tests

Consequences:

- Repeatability under repeated loading cannot be evaluated at the system level.
- Repeatability must be taken from the TAL220B datasheet or approximated using stable-segment noise.
- The uncertainty budget must be more conservative.

Impact level: medium.

### Not Adding Dynamic Response Tests

Consequences:

- The effect of filter parameters on force peaks and response delay cannot be verified.
- If the system is used only for offline drift and static measurement analysis, the impact is small.
- If the system is intended for real-time flight protection, the impact is large.

Impact level depends on the project objective.  
If the course project focuses on drift and static measurement, this can be stated as a limitation. If the project includes real-time safety monitoring, dynamic response testing should be added.

## 8. Recommended Use of Existing Data in the Final Report

The final report can be organized as follows:

1. Add a multi-point static calibration to determine `counts_per_N`.
2. Use the `noload` data to show that no-load zero drift is small and to justify the deadband.
3. Use the loaded segment of `2995_0` as the main constant-load creep / stability dataset.
4. Use the unloaded segment of `2995_0` as the zero-return recovery dataset.
5. Use the `2514` dataset as a supporting observation at a second load level.
6. Use the temperature record to show that the experimental temperature variation was small and therefore not the dominant cause of the observed drift.
7. Treat unperformed multi-temperature, repeatability, and dynamic-response tests as limitations or Type B uncertainty sources.

## 9. Final Judgment

If only one additional multi-point static calibration experiment is added, the current data are sufficient to support a master's course project focused on load-cell drift, as long as the experimental boundaries are stated clearly.

The most reasonable conclusion is:

```text
The existing data can be used to analyze zero drift, constant-load stability, creep trend, and unloaded zero-return behavior.
The temperature record shows that the temperature variation during the experiment was very small, so temperature was not the dominant source of drift.
A multi-point static loading / unloading calibration is still required to determine counts_per_N, linearity error, and hysteresis.
If no other experiments are added, temperature compensation, repeatability, and dynamic response should be treated as limitations or Type B uncertainty components based on datasheet information.
```

