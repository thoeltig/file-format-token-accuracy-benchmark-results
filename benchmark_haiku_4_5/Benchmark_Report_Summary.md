# File Format Token Efficiency Benchmark: Comprehensive Report

## 1. Methodology

The underlying question: **Which file format delivers maximum information value per token consumed?**

This requires measuring:
- **Token Cost**: How many tokens does each format consume for equivalent data?
- **Information Fidelity**: How accurately can the model understand and answer questions about the data?
- **Robustness**: How consistent is performance across data variants (mandatory vs optional fields)?

### 1.1 Metric Definitions

**Token Metrics:**
- `readTokens`: Tokens consumed reading the data file
- `outputTokens`: Tokens consumed during inference (answering questions + creating the file content)
- `totalTokens`: readTokens + outputTokens
- `wastedTokens`: totalTokens × (1 − accuracy% / 100) — tokens spent on inaccurate output

**Accuracy Metrics:**
- `rawAccuracy`: Correct answers / total questions
- `weightedAccuracy`: Accuracy weighted by question category importance

**Efficiency Score:**
- Composite metric balancing accuracy with normalized token cost (weighted towards accuracy)
- `normalizedTokenCost` = (((maxTotalTokens + 10) − currentTotalTokens) / ((maxTotalTokens + 10) − (minTotalTokens − 10))) × 100
- `efficiencyScore` = (accuracy% × 0.7) + (normalizedTokenCost × 0.3)
- `weightedEfficiencyScore` = (weightedAccuracy% × 0.7) + (normalizedTokenCost × 0.3)

### 1.2 Formats Tested

| Format ID | Description |
|---|---|
| `csv` | Comma-separated values — flat structure only |
| `json_compact` | Minified JSON (no whitespace or newlines) |
| `json_pretty` | Standard indented JSON |
| `xml_compact` | Minified XML (no whitespace or indentation) |
| `xml_pretty` | Standard indented XML |
| `toon_default` | [Token-Oriented Object Notation](https://toonformat.dev/) with key folding disabled — nested structures expanded as-is |
| `yaml` | Standard YAML with hierarchical indentation |

## 2. Simplified Results

> [!NOTE]
>
> All columns ranked best-to-worst. ↑ = lower value is better (ascending). ↓ = higher value is better (descending).
> Column abbreviations: **Acc** = Raw Accuracy, **Wtd Acc** = Weighted Accuracy, **Eff Score** = Efficiency Score, **Wtd Eff Score** = Weighted Efficiency Score. See [§1.1](#11-metric-definitions) for formulas.

### 2.1 Flat Structure With Thinking On

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)

#### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| toon_default ≈ 40458 s | csv ≈ 7318  | toon_default ≈ 2238  | json_pretty ≈ 72 % | json_pretty ≈ 72 % | toon_default ≈ 77  | toon_default ≈ 77  |
| xml_compact ( +86.2 %) | toon_default ( +0.2 %) | csv ( +39.8 %) | toon_default (-2.0 %) | toon_default (-3.0 %) | csv (-11.0 %) | csv (-10.3 %) |
| csv ( +93.5 %) | json_compact ( +29.8 %) | json_compact ( +51.7 %) | xml_compact (-4.3 %) | xml_pretty (-4.0 %) | json_compact (-13.5 %) | json_compact (-12.9 %) |
| yaml ( +118.2 %) | xml_compact ( +64.5 %) | xml_compact ( +76.4 %) | xml_pretty (-4.6 %) | xml_compact (-4.3 %) | xml_compact (-21.1 %) | xml_compact (-20.2 %) |
| xml_pretty ( +153.7 %) | yaml ( +74.7 %) | json_pretty ( +86.2 %) | yaml (-5.4 %) | yaml (-5.4 %) | yaml (-25.1 %) | yaml (-24.2 %) |
| json_pretty ( +162.1 %) | json_pretty ( +99.8 %) | yaml ( +93.5 %) | json_compact (-7.3 %) | json_compact (-7.6 %) | json_pretty (-27.6 %) | json_pretty (-26.7 %) |
| json_compact ( +171.5 %) | xml_pretty ( +125.6 %) | xml_pretty ( +143.9 %) | csv (-14.3 %) | csv (-14.5 %) | xml_pretty (-39.4 %) | xml_pretty (-38.0 %) |

#### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| toon_default ≈ 38908 s | csv ≈ 6931  | json_compact ≈ 2907  | json_compact ≈ 68 % | json_compact ≈ 71 % | json_compact ≈ 71  | json_compact ≈ 73  |
| json_pretty ( +88.1 %) | json_compact ( +31.1 %) | csv ( +7.7 %) | toon_default (-0.7 %) | toon_default (-1.1 %) | csv (-3.5 %) | csv (-4.6 %) |
| xml_pretty ( +103.0 %) | xml_compact ( +62.5 %) | xml_compact ( +33.4 %) | xml_pretty (-1.6 %) | xml_compact (-3.5 %) | xml_compact (-12.0 %) | xml_compact (-12.8 %) |
| json_compact ( +112.9 %) | toon_default ( +71.7 %) | toon_default ( +33.7 %) | xml_compact (-2.4 %) | yaml (-3.5 %) | toon_default (-13.1 %) | toon_default (-13.2 %) |
| yaml ( +127.6 %) | yaml ( +74.6 %) | yaml ( +46.7 %) | yaml (-3.2 %) | xml_pretty (-4.0 %) | yaml (-16.5 %) | yaml (-16.4 %) |
| xml_compact ( +144.7 %) | json_pretty ( +97.7 %) | json_pretty ( +72.4 %) | json_pretty (-4.6 %) | json_pretty (-5.7 %) | json_pretty (-24.9 %) | json_pretty (-25.3 %) |
| csv ( +182.2 %) | xml_pretty ( +121.1 %) | xml_pretty ( +77.1 %) | csv (-13.2 %) | csv (-14.4 %) | xml_pretty (-29.1 %) | xml_pretty (-30.7 %) |


### 2.2 Flat Structure With Thinking Off

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)

#### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| toon_default ≈ 50392 s | csv ≈ 7263  | toon_default ≈ 2485  | json_pretty ≈ 72 % | json_pretty ≈ 73 % | toon_default ≈ 75  | toon_default ≈ 76  |
| yaml ( +50.6 %) | toon_default ( +4.0 %) | csv ( +12.7 %) | json_compact (-4.6 %) | json_compact (-4.5 %) | csv (-4.0 %) | csv (-4.5 %) |
| xml_compact ( +59.3 %) | json_compact ( +32.2 %) | json_compact ( +26.9 %) | toon_default (-4.7 %) | toon_default (-4.6 %) | json_compact (-8.6 %) | json_compact (-8.4 %) |
| csv ( +60.6 %) | xml_compact ( +68.7 %) | json_pretty ( +66.0 %) | xml_pretty (-5.4 %) | xml_pretty (-6.0 %) | xml_compact (-21.9 %) | xml_compact (-22.8 %) |
| xml_pretty ( +71.6 %) | yaml ( +77.3 %) | xml_compact ( +73.6 %) | yaml (-6.5 %) | yaml (-7.7 %) | yaml (-24.0 %) | yaml (-25.0 %) |
| json_pretty ( +91.0 %) | json_pretty ( +101.2 %) | yaml ( +79.7 %) | xml_compact (-7.0 %) | xml_compact (-8.2 %) | json_pretty (-25.3 %) | json_pretty (-25.1 %) |
| json_compact ( +111.6 %) | xml_pretty ( +127.1 %) | xml_pretty ( +123.0 %) | csv (-10.3 %) | csv (-10.8 %) | xml_pretty (-38.2 %) | xml_pretty (-38.4 %) |

#### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| toon_default ≈ 42037 s | csv ≈ 7032  | csv ≈ 3119  | yaml ≈ 66 % | yaml ≈ 68 % | csv ≈ 69  | csv ≈ 70  |
| xml_pretty ( +76.7 %) | json_compact ( +29.0 %) | json_compact ( +5.6 %) | json_pretty (-0.8 %) | xml_compact (-0.7 %) | json_compact (-1.2 %) | json_compact (-2.0 %) |
| json_compact ( +79.9 %) | xml_compact ( +63.4 %) | xml_compact ( +28.8 %) | xml_compact (-1.1 %) | json_pretty (-1.0 %) | xml_compact (-10.9 %) | xml_compact (-10.3 %) |
| json_pretty ( +102.2 %) | toon_default ( +69.2 %) | yaml ( +31.4 %) | xml_pretty (-1.3 %) | xml_pretty (-1.5 %) | yaml (-12.6 %) | yaml (-12.3 %) |
| csv ( +105.9 %) | yaml ( +72.0 %) | toon_default ( +38.8 %) | json_compact (-2.4 %) | toon_default (-3.0 %) | toon_default (-14.3 %) | toon_default (-14.4 %) |
| xml_compact ( +108.6 %) | json_pretty ( +93.8 %) | json_pretty ( +51.5 %) | toon_default (-2.5 %) | json_compact (-3.4 %) | json_pretty (-20.5 %) | json_pretty (-20.1 %) |
| yaml ( +147.4 %) | xml_pretty ( +119.3 %) | xml_pretty ( +74.1 %) | csv (-10.5 %) | csv (-10.6 %) | xml_pretty (-29.2 %) | xml_pretty (-28.8 %) |


### 2.3 Nested Structure With Thinking On

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)

*CSV excluded — flat-only format, does not support nested structures.*

#### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| toon_default ≈ 37017 s | json_compact ≈ 10651  | json_compact ≈ 3321  | json_compact ≈ 69 % | json_compact ≈ 69 % | json_compact ≈ 77  | json_compact ≈ 76  |
| json_compact ( +87.6 %) | xml_compact ( +23.9 %) | xml_compact ( +32.4 %) | yaml (-0.5 %) | yaml (-0.5 %) | xml_compact (-11.6 %) | xml_compact (-11.0 %) |
| xml_pretty ( +91.7 %) | toon_default ( +35.6 %) | yaml ( +39.9 %) | xml_compact (-2.1 %) | xml_compact (-1.5 %) | yaml (-15.6 %) | yaml (-15.6 %) |
| xml_compact ( +102.6 %) | yaml ( +37.5 %) | toon_default ( +49.6 %) | json_pretty (-2.4 %) | json_pretty (-1.7 %) | toon_default (-17.3 %) | toon_default (-18.1 %) |
| yaml ( +147.2 %) | json_pretty ( +71.6 %) | json_pretty ( +84.9 %) | toon_default (-3.2 %) | xml_pretty (-3.7 %) | json_pretty (-31.0 %) | json_pretty (-30.4 %) |
| json_pretty ( +153.7 %) | xml_pretty ( +92.1 %) | xml_pretty ( +115.3 %) | xml_pretty (-3.8 %) | toon_default (-4.1 %) | xml_pretty (-40.5 %) | xml_pretty (-40.5 %) |

#### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| toon_default ≈ 39891 s | json_compact ≈ 10124  | json_compact ≈ 3374  | toon_default ≈ 67 % | toon_default ≈ 69 % | json_compact ≈ 77  | json_compact ≈ 78  |
| xml_compact ( +60.3 %) | xml_compact ( +25.6 %) | xml_compact ( +30.7 %) | json_compact (-0.4 %) | json_compact (-1.1 %) | xml_compact (-11.0 %) | xml_compact (-11.2 %) |
| xml_pretty ( +94.1 %) | toon_default ( +40.4 %) | toon_default ( +38.7 %) | yaml (-0.7 %) | yaml (-2.1 %) | toon_default (-15.1 %) | toon_default (-14.2 %) |
| json_pretty ( +111.8 %) | yaml ( +42.1 %) | yaml ( +43.3 %) | xml_compact (-1.8 %) | xml_compact (-2.9 %) | yaml (-16.4 %) | yaml (-16.8 %) |
| json_compact ( +120.4 %) | json_pretty ( +69.2 %) | json_pretty ( +97.9 %) | xml_pretty (-4.7 %) | xml_pretty (-6.4 %) | json_pretty (-31.6 %) | json_pretty (-31.8 %) |
| yaml ( +136.8 %) | xml_pretty ( +96.8 %) | xml_pretty ( +122.2 %) | json_pretty (-6.0 %) | json_pretty (-7.5 %) | xml_pretty (-41.0 %) | xml_pretty (-41.2 %) |


### 2.4 Nested Structure With Thinking Off

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)

*CSV excluded — flat-only format, does not support nested structures.*

#### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| toon_default ≈ 40225 s | json_compact ≈ 10468  | json_compact ≈ 3579  | xml_compact ≈ 68 % | xml_compact ≈ 67 % | json_compact ≈ 75  | json_compact ≈ 74  |
| json_pretty ( +91.4 %) | xml_compact ( +24.3 %) | xml_compact ( +17.3 %) | toon_default (-1.4 %) | yaml (-0.4 %) | xml_compact (-7.9 %) | xml_compact (-7.7 %) |
| xml_compact ( +98.2 %) | yaml ( +37.3 %) | yaml ( +36.0 %) | yaml (-1.6 %) | toon_default (-1.0 %) | yaml (-14.5 %) | yaml (-13.2 %) |
| xml_pretty ( +102.4 %) | toon_default ( +38.4 %) | toon_default ( +36.3 %) | json_compact (-1.9 %) | json_pretty (-1.6 %) | toon_default (-14.8 %) | toon_default (-14.3 %) |
| json_compact ( +103.3 %) | json_pretty ( +71.8 %) | json_pretty ( +75.9 %) | json_pretty (-2.7 %) | json_compact (-2.2 %) | json_pretty (-29.4 %) | json_pretty (-28.3 %) |
| yaml ( +113.3 %) | xml_pretty ( +95.9 %) | xml_pretty ( +118.7 %) | xml_pretty (-5.9 %) | xml_pretty (-5.4 %) | xml_pretty (-41.9 %) | xml_pretty (-41.6 %) |

#### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| toon_default ≈ 53833 s | json_compact ≈ 9950  | json_compact ≈ 3547  | toon_default ≈ 65 % | toon_default ≈ 67 % | json_compact ≈ 75  | json_compact ≈ 75  |
| yaml ( +24.9 %) | xml_compact ( +28.3 %) | xml_compact ( +39.3 %) | json_compact (-0.6 %) | json_compact (-2.0 %) | xml_compact (-13.5 %) | xml_compact (-12.3 %) |
| xml_pretty ( +27.7 %) | toon_default ( +42.8 %) | toon_default ( +40.3 %) | xml_compact (-3.7 %) | xml_compact (-3.8 %) | toon_default (-15.5 %) | toon_default (-14.2 %) |
| xml_compact ( +41.8 %) | yaml ( +45.2 %) | yaml ( +67.6 %) | json_pretty (-4.0 %) | json_pretty (-4.9 %) | yaml (-22.1 %) | yaml (-21.6 %) |
| json_pretty ( +75.3 %) | json_pretty ( +71.5 %) | json_pretty ( +87.5 %) | xml_pretty (-4.2 %) | xml_pretty (-5.0 %) | json_pretty (-30.0 %) | json_pretty (-29.5 %) |
| json_compact ( +75.9 %) | xml_pretty ( +99.8 %) | xml_pretty ( +119.9 %) | yaml (-6.1 %) | yaml (-7.0 %) | xml_pretty (-40.9 %) | xml_pretty (-40.2 %) |


## 3. Conclusion & Decision Matrix

### 3.1 Cross-Configuration Findings

**Structure is the dominant variable.** Switching from flat to nested completely changes which format wins: TOON_DEFAULT leads flat mandatory but drops to mid-tier on nested data (eff: 77.5 → ~63). JSON_COMPACT leads all nested configurations regardless of thinking mode or data variant. Thinking on vs off does not substantially change format rankings — it only shifts absolute accuracy levels.

**TOON_DEFAULT is a flat mandatory specialist.** It wins efficiency on flat mandatory data under both thinking modes (77.5 thinking-on, 75.4 thinking-off) and has the lowest processing duration. However, it has no advantage on nested structures and degrades significantly with optional data (~62% token increase).

**JSON_COMPACT is the format with the most consistent performance across configurations.** It wins all nested configurations (eff: 74.6–77.8) and flat optional with thinking-on (70.8). It shows the most stable performance across mandatory and optional variants. The only configuration it doesn't win is flat mandatory (where it ranks 3rd behind TOON_DEFAULT and CSV).

**CSV wins raw token count but rarely wins efficiency.** It only beats JSON_COMPACT once in efficiency score — flat optional with thinking-off, by a margin of 0.8 points (68.9 vs 68.1). In every other configuration its low accuracy turns the token savings into net waste.

**XML_PRETTY finishes last in every configuration without exception.** Across all 8 configuration/variant combinations it has the lowest efficiency score. There is no workload in this benchmark where it is the right choice for LLM context consumption.

**Result stability is confirmed.** A second iteration of flat + thinking-on (verify run) produced consistent rankings with minor drift (TOON_DEFAULT mandatory: 77.5 → 78.0, JSON_COMPACT optional: 70.8 → 69.5). Conclusions hold across runs.

### 3.2 Decision Matrix

> Best = highest efficiency score for this use case. Alternative = viable fallback. Avoid = consistently poor efficiency or accuracy for this use case.

| Use Case | Best | Alternative | Avoid |
|---|---|---|---|
| Dense data, flat structure | TOON_DEFAULT | CSV † | XML_PRETTY |
| Dense data, nested structure | JSON_COMPACT | XML_COMPACT | XML_PRETTY |
| Sparse / optional data (any structure) | JSON_COMPACT | XML_COMPACT | XML_PRETTY |
| Minimize token cost | CSV † | TOON_DEFAULT ‡ | XML_PRETTY |
| Minimize wasted tokens | JSON_COMPACT | TOON_DEFAULT ‡ (flat mandatory) | CSV |
† CSV wins on raw token count but delivers the worst accuracy (54–63%). Only justified when the token budget is severely constrained and accuracy ≥ 54% is acceptable. On optional data the accuracy drop is severe enough that the token savings are largely negated by wasted tokens.

‡ TOON_DEFAULT has the lowest token cost among high-accuracy formats, but only on flat mandatory data. On optional or nested data it loses this advantage.

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure

- **Test Date**: 2026-03-18
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Extended Thinking**: on & off
- **Structure**: flat & nested
- **Variant**: mandatory (22 fields, dense) and optional (19 mandatory + 3 optional, sparse)
- **Formats Tested**: csv, json_compact, json_pretty, toon_default, xml_compact, xml_pretty, yaml
- **Record Counts**: 31

### 4.2 Appendix B: Benchmark Configuration

**Question Distribution:**

- **Field Retrieval:** Extract specific values from specific records (55 questions, 37.50% weight)
- **Filtering:** Count records matching criteria (21 questions, 20.83% weight)
- **Aggregation:** Sum, average, min/max calculations (21 questions, 12.50% weight)
- **Structure Awareness:** Understand data shape, organization, metadata (27 questions, 29.17% weight)

**Weighting Rationale:**

- Field retrieval + structure awareness = 66.67%
   - These represent the file format itself. Understanding "what data exists and how it's organized" which is fundamental to avoiding context confusion.
- Filtering + aggregation = 33.33%
   - These represent more the "intellectual" aspect of the model and will differ greatly depending on the model. Also if done deterministic the model still needs to do field retrieval and structure awareness on the result.

---

- **Report Generated**: 2026-03-18
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **With the help of**: Claude Sonnet 4.6
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Full Benchmark Reports**: 
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking on - verification run](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_flat_all_formats_and_variants_on_verify/BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)