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
- `accuracy`: Correct answers / total questions
- `weightedAccuracy`: Accuracy weighted by question category importance

**Efficiency Score:**
- Composite metric balancing accuracy with normalized token cost (weighted towards accuracy)
- `normalizedTokenCost` = (((maxTotalTokens + 10) − currentTotalTokens) / ((maxTotalTokens + 10) − (minTotalTokens − 10))) × 100
- `efficiencyScore` = (accuracy% × 0.7) + (normalizedTokenCost × 0.3)
- `weightedEfficiencyScore` = (weightedAccuracy% × 0.7) + (normalizedTokenCost × 0.3)

### 1.2 Formats Tested

| Format ID | Description |
|---|---|
| `CSV` | Comma-separated values — flat structure only |
| `JSON_COMPACT` | Minified JSON (no whitespace or newlines) |
| `JSON_PRETTY` | Standard indented JSON |
| `XML_COMPACT` | Minified XML (no whitespace or indentation) |
| `XML_PRETTY` | Standard indented XML |
| `TOON_DEFAULT` | [Token-Oriented Object Notation](https://toonformat.dev/) with key folding disabled — nested structures expanded as-is |
| `YAML` | Standard YAML with hierarchical indentation |

## 2. Simplified Results

> [!NOTE]
>
> All columns ranked best-to-worst. ↑ = lower value is better (ascending). ↓ = higher value is better (descending).
> Column abbreviations: **Wtd Accuracy** = Weighted Accuracy, **Eff Score** = Efficiency Score, **Wtd Eff Score** = Weighted Efficiency Score. See [§1.1](#11-metric-definitions) for formulas.

### 2.1 Flat Structure With Thinking On

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)

#### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 40458s | CSV ≈ 7318 | TOON_DEFAULT ≈ 2238 | JSON_PRETTY ≈ 72% | JSON_PRETTY ≈ 72% | TOON_DEFAULT ≈ 77 | TOON_DEFAULT ≈ 77  |
| XML_COMPACT (+86.2%) | TOON_DEFAULT (+0.2%) | CSV (+39.8%) | TOON_DEFAULT (-2.0%) | TOON_DEFAULT (-3.0%) | CSV (-11.0%) | CSV (-10.3%) |
| CSV (+93.5%) | JSON_COMPACT (+29.8%) | JSON_COMPACT (+51.7%) | XML_COMPACT (-4.3%) | XML_PRETTY (-4.0%) | JSON_COMPACT (-13.5%) | JSON_COMPACT (-12.9%) |
| YAML (+118.2%) | XML_COMPACT (+64.5%) | XML_COMPACT (+76.4%) | XML_PRETTY (-4.6%) | XML_COMPACT (-4.3%) | XML_COMPACT (-21.1%) | XML_COMPACT (-20.2%) |
| XML_PRETTY (+153.7%) | YAML (+74.7%) | JSON_PRETTY (+86.2%) | YAML (-5.4%) | YAML (-5.4%) | YAML (-25.1%) | YAML (-24.2%) |
| JSON_PRETTY (+162.1%) | JSON_PRETTY (+99.8%) | YAML (+93.5%) | JSON_COMPACT (-7.3%) | JSON_COMPACT (-7.6%) | JSON_PRETTY (-27.6%) | JSON_PRETTY (-26.7%) |
| JSON_COMPACT (+171.5%) | XML_PRETTY (+125.6%) | XML_PRETTY (+143.9%) | CSV (-14.3%) | CSV (-14.5%) | XML_PRETTY (-39.4%) | XML_PRETTY (-38.0%) |

#### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 38908s | CSV ≈ 6931 | JSON_COMPACT ≈ 2907 | JSON_COMPACT ≈ 68% | JSON_COMPACT ≈ 71% | JSON_COMPACT ≈ 71 | JSON_COMPACT ≈ 73  |
| JSON_PRETTY (+88.1%) | JSON_COMPACT (+31.1%) | CSV (+7.7%) | TOON_DEFAULT (-0.7%) | TOON_DEFAULT (-1.1%) | CSV (-3.5%) | CSV (-4.6%) |
| XML_PRETTY (+103.0%) | XML_COMPACT (+62.5%) | XML_COMPACT (+33.4%) | XML_PRETTY (-1.6%) | XML_COMPACT (-3.5%) | XML_COMPACT (-12.0%) | XML_COMPACT (-12.8%) |
| JSON_COMPACT (+112.9%) | TOON_DEFAULT (+71.7%) | TOON_DEFAULT (+33.7%) | XML_COMPACT (-2.4%) | YAML (-3.5%) | TOON_DEFAULT (-13.1%) | TOON_DEFAULT (-13.2%) |
| YAML (+127.6%) | YAML (+74.6%) | YAML (+46.7%) | YAML (-3.2%) | XML_PRETTY (-4.0%) | YAML (-16.5%) | YAML (-16.4%) |
| XML_COMPACT (+144.7%) | JSON_PRETTY (+97.7%) | JSON_PRETTY (+72.4%) | JSON_PRETTY (-4.6%) | JSON_PRETTY (-5.7%) | JSON_PRETTY (-24.9%) | JSON_PRETTY (-25.3%) |
| CSV (+182.2%) | XML_PRETTY (+121.1%) | XML_PRETTY (+77.1%) | CSV (-13.2%) | CSV (-14.4%) | XML_PRETTY (-29.1%) | XML_PRETTY (-30.7%) |


### 2.2 Flat Structure With Thinking Off

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)

#### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 50392s | CSV ≈ 7263 | TOON_DEFAULT ≈ 2485 | JSON_PRETTY ≈ 72% | JSON_PRETTY ≈ 73% | TOON_DEFAULT ≈ 75 | TOON_DEFAULT ≈ 76  |
| YAML (+50.6%) | TOON_DEFAULT (+4.0%) | CSV (+12.7%) | JSON_COMPACT (-4.6%) | JSON_COMPACT (-4.5%) | CSV (-4.0%) | CSV (-4.5%) |
| XML_COMPACT (+59.3%) | JSON_COMPACT (+32.2%) | JSON_COMPACT (+26.9%) | TOON_DEFAULT (-4.7%) | TOON_DEFAULT (-4.6%) | JSON_COMPACT (-8.6%) | JSON_COMPACT (-8.4%) |
| CSV (+60.6%) | XML_COMPACT (+68.7%) | JSON_PRETTY (+66.0%) | XML_PRETTY (-5.4%) | XML_PRETTY (-6.0%) | XML_COMPACT (-21.9%) | XML_COMPACT (-22.8%) |
| XML_PRETTY (+71.6%) | YAML (+77.3%) | XML_COMPACT (+73.6%) | YAML (-6.5%) | YAML (-7.7%) | YAML (-24.0%) | YAML (-25.0%) |
| JSON_PRETTY (+91.0%) | JSON_PRETTY (+101.2%) | YAML (+79.7%) | XML_COMPACT (-7.0%) | XML_COMPACT (-8.2%) | JSON_PRETTY (-25.3%) | JSON_PRETTY (-25.1%) |
| JSON_COMPACT (+111.6%) | XML_PRETTY (+127.1%) | XML_PRETTY (+123.0%) | CSV (-10.3%) | CSV (-10.8%) | XML_PRETTY (-38.2%) | XML_PRETTY (-38.4%) |

#### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 42037s | CSV ≈ 7032 | CSV ≈ 3119 | YAML ≈ 66% | YAML ≈ 68% | CSV ≈ 69 | CSV ≈ 70  |
| XML_PRETTY (+76.7%) | JSON_COMPACT (+29.0%) | JSON_COMPACT (+5.6%) | JSON_PRETTY (-0.8%) | XML_COMPACT (-0.7%) | JSON_COMPACT (-1.2%) | JSON_COMPACT (-2.0%) |
| JSON_COMPACT (+79.9%) | XML_COMPACT (+63.4%) | XML_COMPACT (+28.8%) | XML_COMPACT (-1.1%) | JSON_PRETTY (-1.0%) | XML_COMPACT (-10.9%) | XML_COMPACT (-10.3%) |
| JSON_PRETTY (+102.2%) | TOON_DEFAULT (+69.2%) | YAML (+31.4%) | XML_PRETTY (-1.3%) | XML_PRETTY (-1.5%) | YAML (-12.6%) | YAML (-12.3%) |
| CSV (+105.9%) | YAML (+72.0%) | TOON_DEFAULT (+38.8%) | JSON_COMPACT (-2.4%) | TOON_DEFAULT (-3.0%) | TOON_DEFAULT (-14.3%) | TOON_DEFAULT (-14.4%) |
| XML_COMPACT (+108.6%) | JSON_PRETTY (+93.8%) | JSON_PRETTY (+51.5%) | TOON_DEFAULT (-2.5%) | JSON_COMPACT (-3.4%) | JSON_PRETTY (-20.5%) | JSON_PRETTY (-20.1%) |
| YAML (+147.4%) | XML_PRETTY (+119.3%) | XML_PRETTY (+74.1%) | CSV (-10.5%) | CSV (-10.6%) | XML_PRETTY (-29.2%) | XML_PRETTY (-28.8%) |


### 2.3 Nested Structure With Thinking On

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)

*CSV excluded — flat-only format, does not support nested structures.*

#### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 37017s | JSON_COMPACT ≈ 10651 | JSON_COMPACT ≈ 3321 | JSON_COMPACT ≈ 69% | JSON_COMPACT ≈ 69% | JSON_COMPACT ≈ 77 | JSON_COMPACT ≈ 76  |
| JSON_COMPACT (+87.6%) | XML_COMPACT (+23.9%) | XML_COMPACT (+32.4%) | YAML (-0.5%) | YAML (-0.5%) | XML_COMPACT (-11.6%) | XML_COMPACT (-11.0%) |
| XML_PRETTY (+91.7%) | TOON_DEFAULT (+35.6%) | YAML (+39.9%) | XML_COMPACT (-2.1%) | XML_COMPACT (-1.5%) | YAML (-15.6%) | YAML (-15.6%) |
| XML_COMPACT (+102.6%) | YAML (+37.5%) | TOON_DEFAULT (+49.6%) | JSON_PRETTY (-2.4%) | JSON_PRETTY (-1.7%) | TOON_DEFAULT (-17.3%) | TOON_DEFAULT (-18.1%) |
| YAML (+147.2%) | JSON_PRETTY (+71.6%) | JSON_PRETTY (+84.9%) | TOON_DEFAULT (-3.2%) | XML_PRETTY (-3.7%) | JSON_PRETTY (-31.0%) | JSON_PRETTY (-30.4%) |
| JSON_PRETTY (+153.7%) | XML_PRETTY (+92.1%) | XML_PRETTY (+115.3%) | XML_PRETTY (-3.8%) | TOON_DEFAULT (-4.1%) | XML_PRETTY (-40.5%) | XML_PRETTY (-40.5%) |

#### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 39891s | JSON_COMPACT ≈ 10124 | JSON_COMPACT ≈ 3374 | TOON_DEFAULT ≈ 67% | TOON_DEFAULT ≈ 69% | JSON_COMPACT ≈ 77 | JSON_COMPACT ≈ 78  |
| XML_COMPACT (+60.3%) | XML_COMPACT (+25.6%) | XML_COMPACT (+30.7%) | JSON_COMPACT (-0.4%) | JSON_COMPACT (-1.1%) | XML_COMPACT (-11.0%) | XML_COMPACT (-11.2%) |
| XML_PRETTY (+94.1%) | TOON_DEFAULT (+40.4%) | TOON_DEFAULT (+38.7%) | YAML (-0.7%) | YAML (-2.1%) | TOON_DEFAULT (-15.1%) | TOON_DEFAULT (-14.2%) |
| JSON_PRETTY (+111.8%) | YAML (+42.1%) | YAML (+43.3%) | XML_COMPACT (-1.8%) | XML_COMPACT (-2.9%) | YAML (-16.4%) | YAML (-16.8%) |
| JSON_COMPACT (+120.4%) | JSON_PRETTY (+69.2%) | JSON_PRETTY (+97.9%) | XML_PRETTY (-4.7%) | XML_PRETTY (-6.4%) | JSON_PRETTY (-31.6%) | JSON_PRETTY (-31.8%) |
| YAML (+136.8%) | XML_PRETTY (+96.8%) | XML_PRETTY (+122.2%) | JSON_PRETTY (-6.0%) | JSON_PRETTY (-7.5%) | XML_PRETTY (-41.0%) | XML_PRETTY (-41.2%) |


### 2.4 Nested Structure With Thinking Off

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)

*CSV excluded — flat-only format, does not support nested structures.*

#### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 40225s | JSON_COMPACT ≈ 10468 | JSON_COMPACT ≈ 3579 | XML_COMPACT ≈ 68% | XML_COMPACT ≈ 67% | JSON_COMPACT ≈ 75 | JSON_COMPACT ≈ 74  |
| JSON_PRETTY (+91.4%) | XML_COMPACT (+24.3%) | XML_COMPACT (+17.3%) | TOON_DEFAULT (-1.4%) | YAML (-0.4%) | XML_COMPACT (-7.9%) | XML_COMPACT (-7.7%) |
| XML_COMPACT (+98.2%) | YAML (+37.3%) | YAML (+36.0%) | YAML (-1.6%) | TOON_DEFAULT (-1.0%) | YAML (-14.5%) | YAML (-13.2%) |
| XML_PRETTY (+102.4%) | TOON_DEFAULT (+38.4%) | TOON_DEFAULT (+36.3%) | JSON_COMPACT (-1.9%) | JSON_PRETTY (-1.6%) | TOON_DEFAULT (-14.8%) | TOON_DEFAULT (-14.3%) |
| JSON_COMPACT (+103.3%) | JSON_PRETTY (+71.8%) | JSON_PRETTY (+75.9%) | JSON_PRETTY (-2.7%) | JSON_COMPACT (-2.2%) | JSON_PRETTY (-29.4%) | JSON_PRETTY (-28.3%) |
| YAML (+113.3%) | XML_PRETTY (+95.9%) | XML_PRETTY (+118.7%) | XML_PRETTY (-5.9%) | XML_PRETTY (-5.4%) | XML_PRETTY (-41.9%) | XML_PRETTY (-41.6%) |

#### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 53833s | JSON_COMPACT ≈ 9950 | JSON_COMPACT ≈ 3547 | TOON_DEFAULT ≈ 65% | TOON_DEFAULT ≈ 67% | JSON_COMPACT ≈ 75 | JSON_COMPACT ≈ 75  |
| YAML (+24.9%) | XML_COMPACT (+28.3%) | XML_COMPACT (+39.3%) | JSON_COMPACT (-0.6%) | JSON_COMPACT (-2.0%) | XML_COMPACT (-13.5%) | XML_COMPACT (-12.3%) |
| XML_PRETTY (+27.7%) | TOON_DEFAULT (+42.8%) | TOON_DEFAULT (+40.3%) | XML_COMPACT (-3.7%) | XML_COMPACT (-3.8%) | TOON_DEFAULT (-15.5%) | TOON_DEFAULT (-14.2%) |
| XML_COMPACT (+41.8%) | YAML (+45.2%) | YAML (+67.6%) | JSON_PRETTY (-4.0%) | JSON_PRETTY (-4.9%) | YAML (-22.1%) | YAML (-21.6%) |
| JSON_PRETTY (+75.3%) | JSON_PRETTY (+71.5%) | JSON_PRETTY (+87.5%) | XML_PRETTY (-4.2%) | XML_PRETTY (-5.0%) | JSON_PRETTY (-30.0%) | JSON_PRETTY (-29.5%) |
| JSON_COMPACT (+75.9%) | XML_PRETTY (+99.8%) | XML_PRETTY (+119.9%) | YAML (-6.1%) | YAML (-7.0%) | XML_PRETTY (-40.9%) | XML_PRETTY (-40.2%) |


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
- **Formats Tested**: CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML
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