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
| TOON_DEFAULT ≈ 40s | CSV ≈ 7318 | TOON_DEFAULT ≈ 1499 | JSON_PRETTY ≈ 83% | JSON_PRETTY ≈ 82% | TOON_DEFAULT ≈ 84 | TOON_DEFAULT ≈ 83 |
| XML_COMPACT (+86.2%) | TOON_DEFAULT (+0.2%) | JSON_COMPACT (+61.8%) | TOON_DEFAULT (-3.2%) | TOON_DEFAULT (-4.1%) | JSON_COMPACT (-12.2%) | JSON_COMPACT (-11.9%) |
| CSV (+93.5%) | JSON_COMPACT (+29.8%) | JSON_PRETTY (+67.9%) | XML_PRETTY (-4.8%) | XML_PRETTY (-4.2%) | CSV (-13.1%) | CSV (-12.2%) |
| YAML (+118.2%) | XML_COMPACT (+64.5%) | CSV (+77.2%) | XML_COMPACT (-5.4%) | XML_COMPACT (-5.2%) | XML_COMPACT (-19.2%) | XML_COMPACT (-18.6%) |
| XML_PRETTY (+153.7%) | YAML (+74.7%) | XML_COMPACT (+81.4%) | YAML (-5.6%) | YAML (-5.6%) | YAML (-22.2%) | YAML (-21.8%) |
| JSON_PRETTY (+162.1%) | JSON_PRETTY (+99.8%) | YAML (+94.9%) | JSON_COMPACT (-8.3%) | JSON_COMPACT (-8.5%) | JSON_PRETTY (-24.3%) | JSON_PRETTY (-24.0%) |
| JSON_COMPACT (+171.5%) | XML_PRETTY (+125.6%) | XML_PRETTY (+142.8%) | CSV (-19.1%) | CSV (-18.6%) | XML_PRETTY (-35.3%) | XML_PRETTY (-34.6%) |

#### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 39s | CSV ≈ 6931 | JSON_COMPACT ≈ 1856 | XML_PRETTY ≈ 80% | JSON_COMPACT ≈ 81% | JSON_COMPACT ≈ 79 | JSON_COMPACT ≈ 80 |
| JSON_PRETTY (+88.1%) | JSON_COMPACT (+31.1%) | XML_COMPACT (+24.0%) | TOON_DEFAULT (-0.3%) | TOON_DEFAULT (-0.3%) | CSV (-6.0%) | CSV (-6.6%) |
| XML_PRETTY (+103.0%) | XML_COMPACT (+62.5%) | TOON_DEFAULT (+29.3%) | JSON_COMPACT (-0.5%) | XML_COMPACT (-1.5%) | XML_COMPACT (-8.6%) | XML_COMPACT (-9.9%) |
| JSON_COMPACT (+112.9%) | TOON_DEFAULT (+71.7%) | CSV (+37.5%) | XML_COMPACT (-0.5%) | YAML (-2.0%) | TOON_DEFAULT (-10.9%) | TOON_DEFAULT (-11.3%) |
| YAML (+127.6%) | YAML (+74.6%) | YAML (+42.0%) | YAML (-1.9%) | XML_PRETTY (-2.2%) | YAML (-13.1%) | YAML (-13.6%) |
| XML_COMPACT (+144.7%) | JSON_PRETTY (+97.7%) | XML_PRETTY (+64.2%) | JSON_PRETTY (-3.8%) | JSON_PRETTY (-4.6%) | JSON_PRETTY (-21.1%) | JSON_PRETTY (-22.2%) |
| CSV (+182.2%) | XML_PRETTY (+121.1%) | JSON_PRETTY (+74.7%) | CSV (-16.9%) | CSV (-17.1%) | XML_PRETTY (-24.2%) | XML_PRETTY (-26.4%) |


### 2.2 Flat Structure With Thinking Off

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)

#### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 50s | CSV ≈ 7263 | TOON_DEFAULT ≈ 1747 | JSON_PRETTY ≈ 83% | JSON_PRETTY ≈ 82% | TOON_DEFAULT ≈ 82 | TOON_DEFAULT ≈ 82 |
| YAML (+50.6%) | TOON_DEFAULT (+4.0%) | JSON_COMPACT (+20.7%) | JSON_COMPACT (-4.5%) | JSON_COMPACT (-4.4%) | CSV (-6.8%) | CSV (-6.8%) |
| XML_COMPACT (+59.3%) | JSON_COMPACT (+32.2%) | CSV (+34.8%) | TOON_DEFAULT (-5.7%) | TOON_DEFAULT (-5.5%) | JSON_COMPACT (-6.9%) | JSON_COMPACT (-7.0%) |
| CSV (+60.6%) | XML_COMPACT (+68.7%) | JSON_PRETTY (+46.1%) | XML_PRETTY (-5.9%) | XML_PRETTY (-6.5%) | XML_COMPACT (-21.1%) | XML_COMPACT (-22.1%) |
| XML_PRETTY (+71.6%) | YAML (+77.3%) | YAML (+78.3%) | YAML (-6.7%) | YAML (-8.0%) | YAML (-21.4%) | JSON_PRETTY (-22.6%) |
| JSON_PRETTY (+91.0%) | JSON_PRETTY (+101.2%) | XML_COMPACT (+86.6%) | XML_COMPACT (-9.1%) | XML_COMPACT (-10.0%) | JSON_PRETTY (-22.4%) | YAML (-22.7%) |
| JSON_COMPACT (+111.6%) | XML_PRETTY (+127.1%) | XML_PRETTY (+120.8%) | CSV (-15.0%) | CSV (-14.7%) | XML_PRETTY (-34.7%) | XML_PRETTY (-35.4%) |

#### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 42s | CSV ≈ 7032 | JSON_COMPACT ≈ 2122 | YAML ≈ 80% | YAML ≈ 80% | JSON_COMPACT ≈ 77 | JSON_COMPACT ≈ 76 |
| XML_PRETTY (+76.7%) | JSON_COMPACT (+29.0%) | YAML (+11.8%) | JSON_PRETTY (-1.7%) | XML_COMPACT (-1.4%) | CSV (-1.7%) | CSV (-0.6%) |
| JSON_COMPACT (+79.9%) | XML_COMPACT (+63.4%) | CSV (+14.4%) | XML_COMPACT (-1.9%) | JSON_PRETTY (-1.7%) | XML_COMPACT (-8.2%) | XML_COMPACT (-7.2%) |
| JSON_PRETTY (+102.2%) | TOON_DEFAULT (+69.2%) | XML_COMPACT (+16.4%) | XML_PRETTY (-3.5%) | XML_PRETTY (-3.3%) | YAML (-9.0%) | YAML (-8.4%) |
| CSV (+105.9%) | YAML (+72.0%) | TOON_DEFAULT (+35.1%) | JSON_COMPACT (-3.8%) | JSON_COMPACT (-4.5%) | TOON_DEFAULT (-12.2%) | TOON_DEFAULT (-11.9%) |
| XML_COMPACT (+108.6%) | JSON_PRETTY (+93.8%) | JSON_PRETTY (+36.7%) | TOON_DEFAULT (-4.5%) | TOON_DEFAULT (-4.7%) | JSON_PRETTY (-16.8%) | JSON_PRETTY (-16.3%) |
| YAML (+147.4%) | XML_PRETTY (+119.3%) | XML_PRETTY (+68.0%) | CSV (-14.9%) | CSV (-14.4%) | XML_PRETTY (-25.8%) | XML_PRETTY (-25.3%) |


### 2.3 Nested Structure With Thinking On

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)

*CSV excluded — flat-only format, does not support nested structures.*

#### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 37s | JSON_COMPACT ≈ 10651 | JSON_COMPACT ≈ 2576 | YAML ≈ 79% | YAML ≈ 77% | JSON_COMPACT ≈ 82 | JSON_COMPACT ≈ 81 |
| JSON_COMPACT (+87.6%) | XML_COMPACT (+23.9%) | YAML (+20.7%) | JSON_PRETTY (-1.9%) | JSON_PRETTY (-1.1%) | XML_COMPACT (-10.2%) | XML_COMPACT (-9.9%) |
| XML_PRETTY (+91.7%) | TOON_DEFAULT (+35.6%) | XML_COMPACT (+30.8%) | JSON_COMPACT (-3.0%) | JSON_COMPACT (-2.4%) | YAML (-11.7%) | YAML (-12.3%) |
| XML_COMPACT (+102.6%) | YAML (+37.5%) | TOON_DEFAULT (+43.1%) | TOON_DEFAULT (-4.3%) | XML_COMPACT (-3.3%) | TOON_DEFAULT (-14.6%) | TOON_DEFAULT (-15.8%) |
| YAML (+147.2%) | JSON_PRETTY (+71.6%) | JSON_PRETTY (+64.0%) | XML_COMPACT (-4.3%) | TOON_DEFAULT (-4.9%) | JSON_PRETTY (-26.2%) | JSON_PRETTY (-26.3%) |
| JSON_PRETTY (+153.7%) | XML_PRETTY (+92.1%) | XML_PRETTY (+111.3%) | XML_PRETTY (-5.4%) | XML_PRETTY (-4.9%) | XML_PRETTY (-36.9%) | XML_PRETTY (-37.5%) |

#### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 40s | JSON_COMPACT ≈ 10124 | JSON_COMPACT ≈ 2123 | YAML ≈ 81% | TOON_DEFAULT ≈ 81% | JSON_COMPACT ≈ 85 | JSON_COMPACT ≈ 85 |
| XML_COMPACT (+60.3%) | XML_COMPACT (+25.6%) | YAML (+31.2%) | TOON_DEFAULT (-0.4%) | YAML (-1.2%) | XML_COMPACT (-11.7%) | XML_COMPACT (-11.7%) |
| XML_PRETTY (+94.1%) | TOON_DEFAULT (+40.4%) | TOON_DEFAULT (+32.3%) | JSON_COMPACT (-1.6%) | JSON_COMPACT (-1.8%) | TOON_DEFAULT (-12.9%) | TOON_DEFAULT (-12.4%) |
| JSON_PRETTY (+111.8%) | YAML (+42.1%) | XML_COMPACT (+46.5%) | XML_COMPACT (-5.1%) | XML_COMPACT (-5.4%) | YAML (-13.2%) | YAML (-14.0%) |
| JSON_COMPACT (+120.4%) | JSON_PRETTY (+69.2%) | JSON_PRETTY (+119.0%) | XML_PRETTY (-7.0%) | XML_PRETTY (-8.0%) | JSON_PRETTY (-28.9%) | JSON_PRETTY (-29.5%) |
| YAML (+136.8%) | XML_PRETTY (+96.8%) | XML_PRETTY (+147.3%) | JSON_PRETTY (-7.8%) | JSON_PRETTY (-8.6%) | XML_PRETTY (-37.7%) | XML_PRETTY (-38.4%) |


### 2.4 Nested Structure With Thinking Off

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)

*CSV excluded — flat-only format, does not support nested structures.*

#### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 40s | JSON_COMPACT ≈ 10468 | JSON_COMPACT ≈ 2516 | YAML ≈ 77% | YAML ≈ 76% | JSON_COMPACT ≈ 82 | JSON_COMPACT ≈ 80 |
| JSON_PRETTY (+91.4%) | XML_COMPACT (+24.3%) | XML_COMPACT (+22.4%) | XML_COMPACT (-1.1%) | XML_COMPACT (-1.9%) | XML_COMPACT (-8.5%) | XML_COMPACT (-8.2%) |
| XML_COMPACT (+98.2%) | YAML (+37.3%) | YAML (+29.0%) | JSON_COMPACT (-1.5%) | TOON_DEFAULT (-2.0%) | YAML (-12.3%) | YAML (-11.4%) |
| XML_PRETTY (+102.4%) | TOON_DEFAULT (+38.4%) | TOON_DEFAULT (+38.8%) | TOON_DEFAULT (-1.5%) | JSON_COMPACT (-2.8%) | TOON_DEFAULT (-14.0%) | TOON_DEFAULT (-13.6%) |
| JSON_COMPACT (+103.3%) | JSON_PRETTY (+71.8%) | JSON_PRETTY (+84.6%) | JSON_PRETTY (-3.2%) | JSON_PRETTY (-3.0%) | JSON_PRETTY (-27.6%) | JSON_PRETTY (-26.9%) |
| YAML (+113.3%) | XML_PRETTY (+95.9%) | XML_PRETTY (+141.1%) | XML_PRETTY (-7.0%) | XML_PRETTY (-7.3%) | XML_PRETTY (-39.6%) | XML_PRETTY (-39.6%) |

#### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 54s | JSON_COMPACT ≈ 9950 | JSON_COMPACT ≈ 2182 | JSON_COMPACT ≈ 78% | TOON_DEFAULT ≈ 77% | JSON_COMPACT ≈ 85 | JSON_COMPACT ≈ 83 |
| YAML (+24.9%) | XML_COMPACT (+28.3%) | TOON_DEFAULT (+47.8%) | TOON_DEFAULT (-0.8%) | JSON_COMPACT (-0.8%) | XML_COMPACT (-12.2%) | XML_COMPACT (-11.3%) |
| XML_PRETTY (+27.7%) | TOON_DEFAULT (+42.8%) | XML_COMPACT (+47.8%) | XML_COMPACT (-3.3%) | XML_COMPACT (-2.9%) | TOON_DEFAULT (-14.9%) | TOON_DEFAULT (-13.8%) |
| XML_COMPACT (+41.8%) | YAML (+45.2%) | JSON_PRETTY (+101.8%) | JSON_PRETTY (-3.9%) | JSON_PRETTY (-4.2%) | YAML (-23.9%) | YAML (-23.1%) |
| JSON_PRETTY (+75.3%) | JSON_PRETTY (+71.5%) | YAML (+115.4%) | XML_PRETTY (-6.8%) | XML_PRETTY (-6.6%) | JSON_PRETTY (-27.1%) | JSON_PRETTY (-27.0%) |
| JSON_COMPACT (+75.9%) | XML_PRETTY (+99.8%) | XML_PRETTY (+162.0%) | YAML (-10.6%) | YAML (-10.1%) | XML_PRETTY (-38.9%) | XML_PRETTY (-38.6%) |


## 3. Conclusion & Decision Matrix

### 3.1 Cross-Configuration Findings

**Format efficiency leadership is structure-dependent, not universal.**
TOON_DEFAULT and JSON_COMPACT do not compete in the same tier across data topologies. TOON_DEFAULT achieves the highest weighted efficiency score in every flat configuration, ranging from 82 to 83 for mandatory data and 76 to 80 for optional data. JSON_COMPACT holds that position in all four nested configurations, scoring between 80 and 85 across mandatory and optional variants with both thinking modes. No single format leads across both structures, which means the data topology must be resolved before selecting a format.

**TOON_DEFAULT degrades significantly on sparse (optional) flat data.**
For flat mandatory data TOON_DEFAULT requires approximately 7.3K read tokens. When the variant shifts to optional fields the read token count rises to approximately 11.6K, a 59% increase. Over the same transition JSON_COMPACT moves from approximately 9.3K to 9.1K, a negligible change. The weighted efficiency score of TOON_DEFAULT drops from 82 to 84 (mandatory) down to 67 to 70 (optional), while JSON_COMPACT holds in the 76 to 80 range. This behavior implies that TOON_DEFAULT encodes absent optional fields with non-trivial token overhead, and the deployment assumption of dense mandatory schemas should be validated before choosing it.

**XML_PRETTY delivers the worst value proposition in every configuration.**
XML_PRETTY produces the highest or second-highest total token counts and ranks last or second-to-last in weighted efficiency score across all four configurations. Nested mandatory XML_PRETTY consumes 20.1K to 20.5K read tokens compared to 10.2K to 10.7K for JSON_COMPACT, without delivering compensating accuracy gains. The weighted efficiency score for XML_PRETTY in nested configurations falls to 48 to 52, a gap of 28 to 33 points below JSON_COMPACT. The verbose indentation and repeated tag names add structural overhead that does not translate into measurable comprehension benefit.

**JSON_PRETTY achieves peak raw accuracy in flat structures but pays a severe token penalty.**
JSON_PRETTY scores 82.5% to 82.8% accuracy in flat configurations, the highest of any format in that structure. However, the total token count of approximately 14.6K for flat mandatory is 97% higher than TOON_DEFAULT and 52% higher than JSON_COMPACT. In nested structures the accuracy drops to 72% to 77% while the high token count persists, which drives the weighted efficiency score into the 59 to 63 range. JSON_PRETTY is the correct choice only when raw accuracy is the overriding constraint and token cost is secondary.

**CSV is structurally unsuitable for accuracy-dependent workloads.**
CSV achieves the lowest read token count for flat data but converts that advantage into poor efficiency outcomes. Accuracy ranges from 63% to 68% across all flat configurations, trailing the next worst format by 7 to 10 percentage points. The wasted token budget from incorrect responses offsets the savings on the read side. CSV is excluded from nested structures by design, and based on the accuracy floor observed, it should only be considered when a 63 to 68% accuracy rate is operationally acceptable.

**Extended thinking improves accuracy marginally and at increasing token cost.**
Comparing equivalent formats at the same structure and variant, enabling extended thinking shifts accuracy by at most 2 to 4 percentage points in most cases. For example, TOON_DEFAULT flat mandatory accuracy moves from 76.9% (thinking off) to 79.6% (thinking on), a gain of 2.7 points. Output tokens increase measurably with thinking enabled, particularly for formats like JSON_PRETTY and TOON_DEFAULT where output token variance is high. For latency-sensitive or cost-sensitive deployments the accuracy return on the added thinking tokens is consistently low across the formats and structures tested.

**YAML is competitive on nested accuracy but cannot match JSON_COMPACT on efficiency.**
YAML achieves the highest or joint-highest raw accuracy in three of the four nested configurations, peaking at 80.6% for nested optional data with thinking on. The weighted efficiency score remains in the 70 to 73 range for nested YAML because it consumes approximately 14K to 14.6K tokens, 35% to 44% more than JSON_COMPACT. The accuracy premium that YAML provides over JSON_COMPACT in nested configurations is typically 1 to 5 percentage points, which does not offset the token volume difference under the 70/30 accuracy-cost weighting applied in the efficiency score formula.

**XML_COMPACT occupies a consistent middle tier without excelling at any dimension.**
XML_COMPACT is never the most accurate, the most efficient, or the cheapest format in any configuration. It sits between the compact formats (TOON_DEFAULT, JSON_COMPACT) and the verbose formats (XML_PRETTY, JSON_PRETTY) on both token cost and efficiency score. Its chars-per-token ratio of 2.3 to 2.6 is the highest across all formats, reflecting dense encoding, but the tag structure still results in higher total token counts than JSON_COMPACT for equivalent data.

### 3.2 Decision Matrix

| Scenario | Recommended Format | Weighted Eff. Score | Accuracy | Rationale |
|---|---|---|---|---|
| Flat structure, dense mandatory fields | `TOON_DEFAULT` | 82–83 | 77–80% | Lowest flat read token count (~7.3K), highest flat efficiency score across both thinking modes |
| Flat structure, sparse optional fields | `JSON_COMPACT` | 76–80 | 76–81% | TOON_DEFAULT token count rises 59% on optional schemas; JSON_COMPACT is stable at ~9K tokens |
| Nested structure, any field density | `JSON_COMPACT` | 80–85 | 75–79% | Lowest nested token count (~10–10.7K), highest nested efficiency score across all four nested configurations |
| Maximum accuracy required, flat structure | `JSON_PRETTY` | 63 | ~83% | Highest flat accuracy at 82.5–82.8%; accept a ~97% token cost premium over TOON_DEFAULT |
| Maximum accuracy required, nested structure | `YAML` (thinking on) | 73 | ~79–81% | YAML achieves highest nested accuracy (78.8–80.6%) with thinking on; JSON_COMPACT is the best accuracy option if thinking is off |
| Token budget critical, flat only | `CSV` | 75–76 | 63–68% | Lowest token count (~7K); only viable when an accuracy floor of 63–68% is operationally acceptable |
| Avoid in all use cases | `XML_PRETTY` | 48–55 | 70–78% | Last or near-last weighted efficiency score in all four configurations; no accuracy advantage justifies the token overhead |

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