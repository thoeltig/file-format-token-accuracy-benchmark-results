# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-17
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Extended Thinking**: off
- **Data Structure**: nested
- **Formats Tested**: 6 (JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: First iteration

## Executive Summary

This benchmark evaluates token efficiency and information accuracy across 6 file formats using Claude Haiku 4.5 (claude-haiku-4-5-20251001) as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context, while a format that accurately conveys information may justify higher token cost.

### Key Findings

1. **JSON_COMPACT dominates efficiency** — lowest token cost (9950–10468 tokens) and highest efficiency score (74.57–75.02), delivering 0.629–0.647 information value per token. It leads all formats across both variants without exception.

2. **XML_COMPACT achieves the highest raw accuracy on mandatory data (67.74%)** and ranks 2nd in efficiency score (68.71), making it the best choice when accuracy is the primary constraint and data is fully populated.

3. **Pretty-printed formats are consistently poor value** — JSON_PRETTY and XML_PRETTY consume 72–99% more tokens than their compact counterparts with no meaningful accuracy gain. XML_PRETTY scores the worst efficiency (43.31–44.34) across all conditions.

4. **YAML is the most fragile format under optional data** — accuracy drops 7.26% (66.13% → 58.87%) with virtually no token reduction (+83 tokens, +0.58%). This is the worst accuracy-to-stability trade-off in the benchmark.

5. **Aggregation is the universal weak point across all formats** — mandatory best is XML_COMPACT at 68.25%, optional best is JSON_COMPACT at 53.33%. The drop on optional data is severe for XML_COMPACT (68.25% → 38.10%, −30.16 pp), signaling a critical failure mode for numerical operations on sparse data.

6. **TOON_DEFAULT excels at structure awareness with optional data (77.78%)** and shows the most stable efficiency score across variants (Δ −0.13), making it the most consistent format when data sparsity varies.

7. **XML_COMPACT suffers the worst accuracy robustness among compact formats** — a 6.45% accuracy drop and 17.71% increase in wasted tokens when switching to optional data, driven primarily by the aggregation collapse.

## 1. Methodology

### 1.1 Research Purpose

The underlying question: **Which file format delivers maximum information value per token consumed?**

This requires measuring:
- **Token Cost**: How many tokens does each format consume for equivalent data?
- **Information Fidelity**: How accurately can the model understand and answer questions about the data?
- **Robustness**: How consistent is performance across data variants (mandatory vs optional fields)?

### 1.2 Test Design

**Data Generation:**
- 6 formats tested: JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML
- 2 variants per format: mandatory (22 fields, dense) and optional (19 mandatory + 3 optional, sparse)
- Record Counts: 31

**Question Distribution:**
- 4 question categories reflecting practical use cases:
   - **Field Retrieval (55 questions, 37.50% weight):** Extract specific values from specific records
   - **Filtering (21 questions, 20.83% weight):** Count records matching criteria
   - **Aggregation (21 questions, 12.50% weight):** Sum, average, min/max calculations
   - **Structure Awareness (27 questions, 29.17% weight):** Understand data shape, organization, metadata

**Weighting Rationale:**
- Field retrieval + structure awareness = 66.67%
   - These represent the file format itself. Understanding "what data exists and how it's organized" which is fundamental to avoiding context confusion.
- Filtering + aggregation = 33.33%
   - These represent more the "intellectual" aspect of the model and will differ greatly depending on the model. Also if done deterministic the model still needs to do field retrieval and structure awareness on the result.

### 1.3 Metrics Definition

**Token Metrics:**
- `readTokens`: Tokens consumed reading the data file
- `outputTokens`: Tokens consumed during inference (answering questions + creating the file content)
- `totalTokens`: readTokens + outputTokens

**Accuracy Metrics:**
- `accuracy`: Correct answers / total questions
- `weightedAccuracy`: Accuracy weighted by question category importanc

**Information Value Metrics:**
- `informationValuePerToken`: (accuracy% / totalTokens) × 100
- `costOfInaccuracy`: totalTokens × (1 - accuracy% / 100) — tokens wasted on inaccurate output

**Efficiency Score:**
- Composite metric balancing accuracy with normalized token cost (favour towards accuracy)
- normalizedTokenCost = (((maxTotalTokens+10)-currenTotalTokens)/((maxTotalTokens+10)-(minTotalTokens-10)))*100
- `efficiencyScore`: (accuracy% x 0.7) + (normalizedTokenCost * 0.3)
- `weightedEfficiencyScore`: (weightedAccuracy% x 0.7) + (normalizedTokenCost * 0.3)

### 1.4 Token Usage Measurements

Tokens usage measured in this benchmark are no estimates but the real token usage the model used in this test. The token usage is reported to the user indirectly in the conversation transcript. Both read and output Tokens are directly extracted from the transcripts of the subagents:
- **Read Tokens**: For each data file a single read subagent is invoked with the only prompt to read the file at the provided filepath and return "Done" once finished and do nothing more. The tokens extraction script searches for the read tool use result and extracted the tokens for that action from it.
- **Output Tokens**: For each data file a three full tests subagent are invoked with all necessary files and the test setup and the instruction to write a file once with the answers. The token extraction script searches for the write tool use result and extracted the tokens for that action from it.

## 2. Results

### 2.1 TLDR: Token Efficiency Analysis

*Note: All columns ranked best-to-worst. ↑ = lower value is better (ascending). ↓ = higher value is better (descending).*

#### 2.1.1 Best results

- Lowest total token cost:
   - Optional: JSON_COMPACT 9950 tokens
   - Mandatory: JSON_COMPACT 10468 tokens
- Lowest output token cost drift:
   - Optional: XML_COMPACT ↓ -0.33% ↑ 0.33%
   - Mandatory: XML_COMPACT ↓ -0.33% ↑ 0.33%
- Highest accuracy:
   - Optional: TOON_DEFAULT 64.98%
   - Mandatory: XML_COMPACT 67.74%
- Lowest accuracy drift:
   - Optional: YAML ↓ 0.00% ↑ 0.00%
   - Mandatory: JSON_PRETTY ↓ -1.98% ↑ 2.98%
- Most useful tokens:
   - Optional: XML_PRETTY 12075 / 19876 tokens
   - Mandatory: XML_PRETTY 12679 / 20506 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 75.02
   - Mandatory: JSON_COMPACT 74.57
- Lowest delta (optional-mandatory):
   - Total tokens: YAML 83 tokens
   - Accuracy: XML_PRETTY -1.08%
   - Token efficiency: TOON_DEFAULT -0.13

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 19876 tokens
   - Mandatory: XML_PRETTY 20506 tokens
- Highest output token drift:
   - Optional: XML_PRETTY ↓ -95.61% ↑ 50.24%
   - Mandatory: YAML ↓ -93.89% ↑ 49.06%
- Lowest accuracy:
   - Optional: YAML 58.87%
   - Mandatory: XML_PRETTY 61.83%
- Highest accuracy drift:
   - Optional: JSON_PRETTY ↓ -11.46% ↑ 12.34%
   - Mandatory: XML_COMPACT ↓ -9.52% ↑ 10.72%
- Most wasted tokens:
   - Optional: XML_PRETTY 7801 / 19876 tokens
   - Mandatory: XML_PRETTY 7827 / 20506 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 44.34
   - Mandatory: XML_PRETTY 43.31
- Highest delta (optional-mandatory):
   - Total tokens: JSON_PRETTY -927 tokens
   - Accuracy: YAML -7.26%
   - Token efficiency: YAML -5.32

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 40225s | JSON_COMPACT ≈ 10468 | JSON_COMPACT ≈ 3579 | XML_COMPACT ≈ 68% | XML_COMPACT ≈ 67% | JSON_COMPACT ≈ 75 | JSON_COMPACT ≈ 74  |
| JSON_PRETTY (+91.4%) | XML_COMPACT (+24.3%) | XML_COMPACT (+17.3%) | TOON_DEFAULT (-1.4%) | YAML (-0.4%) | XML_COMPACT (-7.9%) | XML_COMPACT (-7.7%) |
| XML_COMPACT (+98.2%) | YAML (+37.3%) | YAML (+36.0%) | YAML (-1.6%) | TOON_DEFAULT (-1.0%) | YAML (-14.5%) | YAML (-13.2%) |
| XML_PRETTY (+102.4%) | TOON_DEFAULT (+38.4%) | TOON_DEFAULT (+36.3%) | JSON_COMPACT (-1.9%) | JSON_PRETTY (-1.6%) | TOON_DEFAULT (-14.8%) | TOON_DEFAULT (-14.3%) |
| JSON_COMPACT (+103.3%) | JSON_PRETTY (+71.8%) | JSON_PRETTY (+75.9%) | JSON_PRETTY (-2.7%) | JSON_COMPACT (-2.2%) | JSON_PRETTY (-29.4%) | JSON_PRETTY (-28.3%) |
| YAML (+113.3%) | XML_PRETTY (+95.9%) | XML_PRETTY (+118.7%) | XML_PRETTY (-5.9%) | XML_PRETTY (-5.4%) | XML_PRETTY (-41.9%) | XML_PRETTY (-41.6%) |


##### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 53833s | JSON_COMPACT ≈ 9950 | JSON_COMPACT ≈ 3547 | TOON_DEFAULT ≈ 65% | TOON_DEFAULT ≈ 67% | JSON_COMPACT ≈ 75 | JSON_COMPACT ≈ 75  |
| YAML (+24.9%) | XML_COMPACT (+28.3%) | XML_COMPACT (+39.3%) | JSON_COMPACT (-0.6%) | JSON_COMPACT (-2.0%) | XML_COMPACT (-13.5%) | XML_COMPACT (-12.3%) |
| XML_PRETTY (+27.7%) | TOON_DEFAULT (+42.8%) | TOON_DEFAULT (+40.3%) | XML_COMPACT (-3.7%) | XML_COMPACT (-3.8%) | TOON_DEFAULT (-15.5%) | TOON_DEFAULT (-14.2%) |
| XML_COMPACT (+41.8%) | YAML (+45.2%) | YAML (+67.6%) | JSON_PRETTY (-4.0%) | JSON_PRETTY (-4.9%) | YAML (-22.1%) | YAML (-21.6%) |
| JSON_PRETTY (+75.3%) | JSON_PRETTY (+71.5%) | JSON_PRETTY (+87.5%) | XML_PRETTY (-4.2%) | XML_PRETTY (-5.0%) | JSON_PRETTY (-30.0%) | JSON_PRETTY (-29.5%) |
| JSON_COMPACT (+75.9%) | XML_PRETTY (+99.8%) | XML_PRETTY (+119.9%) | YAML (-6.1%) | YAML (-7.0%) | XML_PRETTY (-40.9%) | XML_PRETTY (-40.2%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 74% | YAML ≈ 70% | YAML ≈ 62% | XML_COMPACT ≈ 68% |
| JSON_COMPACT (-0.5%) | JSON_PRETTY (-0.7%) | JSON_PRETTY (-1.9%) | JSON_COMPACT (-8.3%) |
| TOON_DEFAULT (-1.9%) | XML_COMPACT (-1.2%) | JSON_COMPACT (-3.8%) | TOON_DEFAULT (-10.5%) |
| XML_COMPACT (-2.4%) | TOON_DEFAULT (-2.3%) | TOON_DEFAULT (-4.2%) | XML_PRETTY (-19.0%) |
| JSON_PRETTY (-3.0%) | XML_PRETTY (-7.4%) | XML_COMPACT (-6.3%) | JSON_PRETTY (-19.7%) |
| XML_PRETTY (-4.8%) | JSON_COMPACT (-9.6%) | XML_PRETTY (-7.9%) | YAML (-23.8%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 67% | TOON_DEFAULT ≈ 78% | JSON_PRETTY ≈ 68% | JSON_COMPACT ≈ 53% |
| JSON_PRETTY (-1.2%) | XML_COMPACT (-4.9%) | TOON_DEFAULT (-1.6%) | TOON_DEFAULT (-8.4%) |
| TOON_DEFAULT (-1.6%) | JSON_COMPACT (-10.4%) | JSON_COMPACT (-4.4%) | XML_PRETTY (-12.1%) |
| XML_PRETTY (-2.4%) | XML_PRETTY (-11.1%) | XML_PRETTY (-6.3%) | YAML (-13.6%) |
| XML_COMPACT (-3.0%) | YAML (-14.8%) | YAML (-6.3%) | JSON_PRETTY (-13.7%) |
| YAML (-4.2%) | JSON_PRETTY (-16.1%) | XML_COMPACT (-6.4%) | XML_COMPACT (-15.2%) |


#### 2.1.5 Conclusion

Three distinct performance tiers emerge from this benchmark:

**Tier 1 — High Efficiency:** JSON_COMPACT and XML_COMPACT. JSON_COMPACT wins on token economy and composite efficiency; XML_COMPACT wins on raw accuracy (mandatory). Both compact formats outperform their pretty-printed counterparts decisively.

**Tier 2 — Balanced Mid-range:** TOON_DEFAULT and YAML. Both consume similar tokens (~14,200–14,500) and deliver similar accuracy (~63–66%). YAML leads on mandatory field retrieval and structure awareness but degrades sharply with optional data. TOON_DEFAULT is more stable across variants and uniquely strong at structure awareness on optional data.

**Tier 3 — Poor Value:** JSON_PRETTY and XML_PRETTY. Both consume 71–99% more tokens than their compact equivalents with no accuracy benefit. XML_PRETTY is the weakest format in the benchmark on every composite metric.

The central finding is that **whitespace formatting adds substantial token cost with zero accuracy return for Haiku 4.5 on nested data.** The model extracts information equally well from compact and pretty-printed formats, but compact formats do so at half the token cost. For production use, JSON_COMPACT is the default recommendation; XML_COMPACT is justified only when mandatory-field aggregation accuracy is critical.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Char/Token | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10163 | 305 | 10468 | 2.246 | 0.629 | 2.461 | 65.81 | 73.90 | 6889.122 | 3579.078 | 74.57 | 73.90 |
| JSON_COMPACT | opt | 9645 | 305 | 9950 | 2.219 | 0.647 | 2.458 | 64.35 | 75.37 | 6402.696 | 3547.104 | 75.02 | 75.37 |
| JSON_PRETTY | man | 17682 | 307 | 17989 | 1.777 | 0.361 | 2.473 | 65.00 | 53.00 | 11692.590 | 6296.010 | 52.67 | 53.00 |
| JSON_PRETTY | opt | 16757 | 305 | 17062 | 1.767 | 0.358 | 2.460 | 61.02 | 53.17 | 10411.232 | 6650.768 | 52.51 | 53.17 |
| TOON_DEFAULT | man | 14183 | 309 | 14492 | 1.840 | 0.458 | 2.492 | 66.33 | 63.36 | 9612.566 | 4879.468 | 63.52 | 63.36 |
| TOON_DEFAULT | opt | 13948 | 258 | 14206 | 1.848 | 0.458 | 2.078 | 64.98 | 64.69 | 9230.843 | 4974.824 | 63.39 | 64.69 |
| XML_COMPACT | man | 12705 | 305 | 13010 | 2.551 | 0.521 | 2.460 | 67.74 | 68.25 | 8812.974 | 4197.026 | 68.71 | 68.25 |
| XML_COMPACT | opt | 12455 | 307 | 12762 | 2.499 | 0.480 | 2.476 | 61.29 | 66.09 | 7821.830 | 4940.170 | 64.90 | 66.09 |
| XML_PRETTY | man | 20204 | 302 | 20506 | 1.985 | 0.302 | 2.438 | 61.83 | 43.19 | 12679.014 | 7827.236 | 43.31 | 43.19 |
| XML_PRETTY | opt | 19671 | 205 | 19876 | 1.974 | 0.306 | 1.653 | 60.75 | 45.09 | 12074.670 | 7801.330 | 44.34 | 45.09 |
| YAML | man | 14155 | 213 | 14368 | 1.808 | 0.460 | 1.715 | 66.13 | 64.13 | 9501.338 | 4866.329 | 63.73 | 64.13 |
| YAML | opt | 14148 | 303 | 14451 | 1.787 | 0.407 | 2.441 | 58.87 | 59.10 | 8507.108 | 5943.559 | 58.41 | 59.10 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Accuracy Man (%) | Accuracy Opt (%) | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10468 | 9950 | -518 | -4.95 | 65.81 | 64.35 | -1.46 | 64.86 | 64.85 | -0.01 | 74.57 | 75.02 |  +0.45 | 73.90 | 75.37 |  +1.46 |
| JSON_PRETTY | 17989 | 17062 | -927 | -5.15 | 65.00 | 61.02 | -3.98 | 65.47 | 61.96 | -3.51 | 52.67 | 52.51 | -0.16 | 53.00 | 53.17 |  +0.17 |
| TOON_DEFAULT | 14492 | 14206 | -286 | -1.97 | 66.33 | 64.98 | -1.35 | 66.11 | 66.84 |  +0.73 | 63.52 | 63.39 | -0.13 | 63.36 | 64.69 |  +1.32 |
| XML_COMPACT | 13010 | 12762 | -248 | -1.91 | 67.74 | 61.29 | -6.45 | 67.08 | 63.00 | -4.08 | 68.71 | 64.90 | -3.81 | 68.25 | 66.09 | -2.15 |
| XML_PRETTY | 20506 | 19876 | -630 | -3.07 | 61.83 | 60.75 | -1.08 | 61.66 | 61.82 |  +0.16 | 43.31 | 44.34 |  +1.03 | 43.19 | 45.09 |  +1.90 |
| YAML | 14368 | 14451 |  +83 |  +0.58 | 66.13 | 58.87 | -7.26 | 66.70 | 59.85 | -6.85 | 63.73 | 58.41 | -5.32 | 64.13 | 59.10 | -5.03 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 13 | 781.769 | 0.42 | 81772 | 0.004 | 659.45 | 81785 | 781.773 | 527.65 |
| JSON_COMPACT | opt | 9 | 1071.667 | 0.29 | 94662 | 0.003 | 763.40 | 94671 | 1071.670 | 610.78 |
| JSON_PRETTY | man | 262 | 67.489 | 8.45 | 76737 | 0.004 | 618.85 | 76999 | 67.493 | 496.77 |
| JSON_PRETTY | opt | 270 | 62.063 | 8.71 | 94082 | 0.003 | 758.73 | 94352 | 62.066 | 608.72 |
| TOON_DEFAULT | man | 50 | 283.660 | 1.61 | 82789 | 0.004 | 667.66 | 82839 | 283.664 | 534.45 |
| TOON_DEFAULT | opt | 45 | 309.956 | 1.45 | 94772 | 0.003 | 764.29 | 94817 | 309.959 | 611.72 |
| XML_COMPACT | man | 13 | 977.308 | 0.42 | 79723 | 0.004 | 642.93 | 79736 | 977.312 | 514.43 |
| XML_COMPACT | opt | 12 | 1037.917 | 0.39 | 76345 | 0.004 | 615.69 | 76357 | 1037.921 | 492.63 |
| XML_PRETTY | man | 13 | 1554.154 | 0.42 | 81393 | 0.004 | 656.40 | 81406 | 1554.158 | 525.20 |
| XML_PRETTY | opt | 6 | 3278.500 | 0.19 | 68743 | 0.003 | 554.38 | 68749 | 3278.503 | 443.54 |
| YAML | man | 12 | 1179.583 | 0.39 | 85781 | 0.002 | 691.78 | 85793 | 1179.585 | 553.51 |
| YAML | opt | 11 | 1286.182 | 0.35 | 67236 | 0.005 | 542.22 | 67247 | 1286.187 | 433.85 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) | Total Man (s) | Total Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 13 | 9 | -4 | -30.77 | 81.77 | 94.66 |  +12.89 |  +15.76 | 81.79 | 94.67 |  +12.89 |  +15.76 |
| JSON_PRETTY | 262 | 270 |  +8 |  +3.05 | 76.74 | 94.08 |  +17.35 |  +22.60 | 77.00 | 94.35 |  +17.35 |  +22.54 |
| TOON_DEFAULT | 50 | 45 | -5 | -10.00 | 82.79 | 94.77 |  +11.98 |  +14.47 | 82.84 | 96.45 |  +13.61 |  +16.43 |
| XML_COMPACT | 13 | 12 | -1 | -7.69 | 79.72 | 76.34 | -3.38 | -4.24 | 79.74 | 76.36 | -3.38 | -4.24 |
| XML_PRETTY | 13 | 6 | -7 | -53.85 | 81.39 | 68.74 | -12.65 | -15.54 | 81.41 | 68.75 | -12.66 | -15.55 |
| YAML | 12 | 11 | -1 | -8.33 | 85.78 | 67.24 | -18.55 | -21.62 | 85.79 | 67.25 | -18.55 | -21.62 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Char/Token | Token/Value | Token/Object | Info/Token |
|---|---|---|---|---|---|
| JSON_COMPACT | man | 2.246 | 14.902 | 327.839 | 0.629 |
| JSON_COMPACT | opt | 2.219 | 15.285 | 311.129 | 0.647 |
| JSON_PRETTY | man | 1.777 | 25.927 | 570.387 | 0.361 |
| JSON_PRETTY | opt | 1.767 | 26.556 | 540.548 | 0.358 |
| TOON_DEFAULT | man | 1.840 | 20.796 | 457.516 | 0.458 |
| TOON_DEFAULT | opt | 1.848 | 22.105 | 449.935 | 0.458 |
| XML_COMPACT | man | 2.551 | 18.629 | 409.839 | 0.521 |
| XML_COMPACT | opt | 2.499 | 19.739 | 401.774 | 0.480 |
| XML_PRETTY | man | 1.985 | 29.625 | 651.742 | 0.302 |
| XML_PRETTY | opt | 1.974 | 31.174 | 634.548 | 0.306 |
| YAML | man | 1.808 | 20.755 | 456.613 | 0.460 |
| YAML | opt | 1.787 | 22.422 | 456.387 | 0.407 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 2.246 | 2.219 | -0.027 | -1.20 | 14.902 | 15.285 |  +0.383 |  +2.57 | 327.839 | 311.129 | -16.710 | -5.10 | 0.629 | 0.647 |  +0.018 |  +2.86 |
| JSON_PRETTY | 1.777 | 1.767 | -0.010 | -0.56 | 25.927 | 26.556 |  +0.629 |  +2.43 | 570.387 | 540.548 | -29.839 | -5.23 | 0.361 | 0.358 | -0.003 | -0.83 |
| TOON_DEFAULT | 1.840 | 1.848 |  +0.008 |  +0.43 | 20.796 | 22.105 |  +1.309 |  +6.29 | 457.516 | 449.935 | -7.581 | -1.66 | 0.458 | 0.458 | -0.001 | -0.11 |
| XML_COMPACT | 2.551 | 2.499 | -0.052 | -2.04 | 18.629 | 19.739 |  +1.110 |  +5.96 | 409.839 | 401.774 | -8.065 | -1.97 | 0.521 | 0.480 | -0.041 | -7.87 |
| XML_PRETTY | 1.985 | 1.974 | -0.011 | -0.55 | 29.625 | 31.174 |  +1.549 |  +5.23 | 651.742 | 634.548 | -17.194 | -2.64 | 0.302 | 0.306 |  +0.004 |  +1.32 |
| YAML | 1.808 | 1.787 | -0.021 | -1.16 | 20.755 | 22.422 |  +1.667 |  +8.03 | 456.613 | 456.387 | -0.226 | -0.05 | 0.460 | 0.407 | -0.053 | -11.52 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10468 | 6889 | 3579 | 65.81 | 64.86 | 74.57 | 73.90 |
| JSON_COMPACT | opt | 9950 | 6403 | 3547 | 64.35 | 64.85 | 75.02 | 75.37 |
| JSON_PRETTY | man | 17989 | 11693 | 6296 | 65.00 | 65.47 | 52.67 | 53.00 |
| JSON_PRETTY | opt | 17062 | 10411 | 6651 | 61.02 | 61.96 | 52.51 | 53.17 |
| TOON_DEFAULT | man | 14492 | 9613 | 4879 | 66.33 | 66.11 | 63.52 | 63.36 |
| TOON_DEFAULT | opt | 14206 | 9231 | 4975 | 64.98 | 66.84 | 63.39 | 64.69 |
| XML_COMPACT | man | 13010 | 8813 | 4197 | 67.74 | 67.08 | 68.71 | 68.25 |
| XML_COMPACT | opt | 12762 | 7822 | 4940 | 61.29 | 63.00 | 64.90 | 66.09 |
| XML_PRETTY | man | 20506 | 12679 | 7827 | 61.83 | 61.66 | 43.31 | 43.19 |
| XML_PRETTY | opt | 19876 | 12075 | 7801 | 60.75 | 61.82 | 44.34 | 45.09 |
| YAML | man | 14368 | 9501 | 4866 | 66.13 | 66.70 | 63.73 | 64.13 |
| YAML | opt | 14451 | 8507 | 5944 | 58.87 | 59.85 | 58.41 | 59.10 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10468 | 9950 | -518 | -4.95 | 6889 | 6403 | -486 | -7.06 | 3579 | 3547 | -32 | -0.89 | 65.81 | 64.35 | -1.46 | 74.57 | 75.017 |  +0.45 |  +0.60 |
| JSON_PRETTY | 17989 | 17062 | -927 | -5.15 | 11693 | 10412 | -1281 | -10.96 | 6296 | 6651 |  +355 |  +5.63 | 65.00 | 61.02 | -3.98 | 52.67 | 52.512 | -0.16 | -0.30 |
| TOON_DEFAULT | 14492 | 14206 | -286 | -1.98 | 9613 | 9231 | -382 | -3.97 | 4879 | 4974 |  +95 |  +1.95 | 66.33 | 64.98 | -1.35 | 63.52 | 63.3855 | -0.13 | -0.21 |
| XML_COMPACT | 13010 | 12762 | -248 | -1.91 | 8813 | 7822 | -991 | -11.25 | 4197 | 4940 |  +743 |  +17.71 | 67.74 | 61.29 | -6.45 | 68.71 | 64.898 | -3.81 | -5.55 |
| XML_PRETTY | 20506 | 19876 | -630 | -3.07 | 12679 | 12075 | -604 | -4.77 | 7827 | 7801 | -26 | -0.33 | 61.83 | 60.75 | -1.08 | 43.31 | 44.341 |  +1.03 |  +2.38 |
| YAML | 14368 | 14451 |  +83 |  +0.58 | 9501 | 8507 | -994 | -10.46 | 4866 | 5943 |  +1077 |  +22.14 | 66.13 | 58.87 | -7.26 | 63.73 | 58.414 | -5.32 | -8.34 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| JSON_COMPACT | man | 82 | 42 | 0 | 65.81 |
| JSON_COMPACT | opt | 80 | 44 | 0 | 64.35 |
| JSON_PRETTY | man | 81 | 43 | 0 | 65.00 |
| JSON_PRETTY | opt | 76 | 48 | 0 | 61.02 |
| TOON_DEFAULT | man | 82 | 42 | 0 | 66.33 |
| TOON_DEFAULT | opt | 81 | 43 | 0 | 64.98 |
| XML_COMPACT | man | 84 | 40 | 0 | 67.74 |
| XML_COMPACT | opt | 76 | 48 | 0 | 61.29 |
| XML_PRETTY | man | 77 | 47 | 0 | 61.83 |
| XML_PRETTY | opt | 75 | 49 | 0 | 60.75 |
| YAML | man | 82 | 42 | 0 | 66.13 |
| YAML | opt | 73 | 51 | 0 | 58.87 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 82 | 80 | -2 | -2.44 | 42 | 44 |  +2 |  +4.76 | 0 | 0 | 0 | 0.00 | 65.81 | 64.35 | -1.46 |
| JSON_PRETTY | 81 | 76 | -5 | -6.17 | 43 | 48 |  +5 |  +11.63 | 0 | 0 | 0 | 0.00 | 65.00 | 61.02 | -3.98 |
| TOON_DEFAULT | 82 | 81 | -1 | -1.22 | 42 | 43 |  +1 |  +2.38 | 0 | 0 | 0 | 0.00 | 66.33 | 64.98 | -1.35 |
| XML_COMPACT | 84 | 76 | -8 | -9.52 | 40 | 48 |  +8 |  +20.00 | 0 | 0 | 0 | 0.00 | 67.74 | 61.29 | -6.45 |
| XML_PRETTY | 77 | 75 | -2 | -2.60 | 47 | 49 |  +2 |  +4.26 | 0 | 0 | 0 | 0.00 | 61.83 | 60.75 | -1.08 |
| YAML | 82 | 73 | -9 | -10.98 | 42 | 51 |  +9 |  +21.43 | 0 | 0 | 0 | 0.00 | 66.13 | 58.87 | -7.26 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 65.81 | 73.46 | 60.74 | 58.09 | 60.00 |
| JSON_COMPACT | opt | 64.35 | 67.27 | 67.41 | 63.81 | 53.33 |
| JSON_PRETTY | man | 65.00 | 70.91 | 69.63 | 60.00 | 48.57 |
| JSON_PRETTY | opt | 61.02 | 66.06 | 61.73 | 68.25 | 39.68 |
| TOON_DEFAULT | man | 66.33 | 72.05 | 68.06 | 57.74 | 57.74 |
| TOON_DEFAULT | opt | 64.98 | 65.71 | 77.78 | 66.67 | 44.90 |
| XML_COMPACT | man | 67.74 | 71.52 | 69.14 | 55.55 | 68.25 |
| XML_COMPACT | opt | 61.29 | 64.24 | 72.84 | 61.90 | 38.10 |
| XML_PRETTY | man | 61.83 | 69.09 | 62.96 | 53.97 | 49.21 |
| XML_PRETTY | opt | 60.75 | 64.85 | 66.67 | 61.90 | 41.27 |
| YAML | man | 66.13 | 73.94 | 70.37 | 61.90 | 44.45 |
| YAML | opt | 58.87 | 63.03 | 62.96 | 61.90 | 39.68 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 73.46 | 67.27 | -6.19 |
| JSON_PRETTY | 70.91 | 66.06 | -4.85 |
| TOON_DEFAULT | 72.05 | 65.71 | -6.33 |
| XML_COMPACT | 71.52 | 64.24 | -7.28 |
| XML_PRETTY | 69.09 | 64.85 | -4.25 |
| YAML | 73.94 | 63.03 | -10.91 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 60.74 | 67.41 |  +6.67 |
| JSON_PRETTY | 69.63 | 61.73 | -7.90 |
| TOON_DEFAULT | 68.06 | 77.78 |  +9.72 |
| XML_COMPACT | 69.14 | 72.84 |  +3.70 |
| XML_PRETTY | 62.96 | 66.67 |  +3.70 |
| YAML | 70.37 | 62.96 | -7.41 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 58.09 | 63.81 |  +5.71 |
| JSON_PRETTY | 60.00 | 68.25 |  +8.26 |
| TOON_DEFAULT | 57.74 | 66.67 |  +8.93 |
| XML_COMPACT | 55.55 | 61.90 |  +6.35 |
| XML_PRETTY | 53.97 | 61.90 |  +7.94 |
| YAML | 61.90 | 61.90 |  +0.00 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 60.00 | 53.33 | -6.67 |
| JSON_PRETTY | 48.57 | 39.68 | -8.89 |
| TOON_DEFAULT | 57.74 | 44.90 | -12.84 |
| XML_COMPACT | 68.25 | 38.10 | -30.16 |
| XML_PRETTY | 49.21 | 41.27 | -7.93 |
| YAML | 44.45 | 39.68 | -4.76 |

## 3. Format-Specific Analysis
### 3.1 Detailed Analysis: JSON_COMPACT

#### 3.1.1 Performance Summary

- Token Duration Range: 82 - 95 seconds
- Token Cost Range: 9950 - 10468 tokens
- Wasted Token Range: 3547 - 3579 tokens
- Accuracy Range: 64.35 - 65.81%
- Efficiency Score Range: 74.57 - 75.02

#### 3.1.2 Strengths

- **Lowest total token cost** across all formats and variants (9950 opt, 10468 man) — 47.9% fewer tokens than XML_PRETTY optional
- **Highest efficiency score** (75.02 opt, 74.57 man) and best information value per token (0.647 opt, 0.629 man)
- **Most stable wasted tokens** across variants (Δ −32 tokens, −0.89%) — optional data introduces almost no additional waste
- **Strong field retrieval on mandatory data** (73.46%) — 2nd best across all formats
- **Fewest total wasted tokens** (3547–3579) — smallest absolute cost of inaccuracy

#### 3.1.3 Weaknesses

- **Weakest structure awareness on mandatory data** (60.74%) — ranks last among all formats; the dense, keyless format makes schema inference harder
- **Aggregation degrades with optional data** (60.00% man → 53.33% opt, −6.67 pp) — numerical operations on sparse data become less reliable
- **Lower absolute accuracy than XML_COMPACT on mandatory** (65.81% vs 67.74%) — pays ~2% accuracy for 24% token savings
- **High accuracy variance on mandatory** (−10.55% to +7.84% drift) — less predictable across individual runs

#### 3.1.4 Use Case Recommendation

- ✓ Use when token budget is the primary constraint and cost efficiency outweighs maximizing accuracy
- ✓ Use when data is dense with mandatory fields and field retrieval is the dominant query type
- ✓ Use when optional/sparse data is present — robustness is strong (only −1.46% accuracy drop)
- ❌ Avoid when schema/structure understanding is critical — structure awareness is the weakest of all formats on mandatory data
- ❌ Avoid when aggregation on optional data is a primary use case

#### 3.1.5 Trade-offs

- Saves 24% tokens vs XML_COMPACT (mandatory) while sacrificing only 1.93% absolute accuracy — the efficiency score gap (74.57 vs 68.71) confirms this is a favourable trade for most use cases
- The optional variant scores a marginally higher efficiency score (75.02) than mandatory (74.57) — one of only two formats where optional data improves efficiency, indicating resilience to sparse fields
- Structure awareness weakness (60.74% on mandatory) is the primary risk; if the model needs to reason about data organisation rather than retrieve specific values, this format underperforms relative to YAML or TOON_DEFAULT

### 3.2 Detailed Analysis: JSON_PRETTY

#### 3.2.1 Performance Summary

- Token Duration Range: 77 - 94 seconds
- Token Cost Range: 17062 - 17989 tokens
- Wasted Token Range: 6296 - 6651 tokens
- Accuracy Range: 61.02 - 65.00%
- Efficiency Score Range: 52.51 - 52.67

#### 3.2.2 Strengths

- **Best filtering accuracy on optional data** (68.25%) — ranks 1st in filtering for optional variant, suggesting whitespace aids comprehension of conditional record matches
- **Most consistent mandatory accuracy** across runs (drift −1.98% to +2.98%) — smallest accuracy variance of any format on mandatory data
- **Good structure awareness on mandatory** (69.63%) — 2nd best, indicating readable formatting helps schema inference on dense data

#### 3.2.3 Weaknesses

- **72% more tokens than JSON_COMPACT** (17989 man, 17062 opt) for essentially equal accuracy — whitespace adds no information value
- **Second-worst efficiency score** (52.51–52.67) — only XML_PRETTY is worse
- **Largest wasted tokens after XML_PRETTY** (6296 man, 6651 opt) — more tokens wasted on optional than mandatory, opposite of compact formats
- **Worst aggregation after XML_PRETTY** (48.57% man, 39.68% opt) — poor numerical reasoning across both variants
- **High accuracy variance on optional** (−11.46% to +12.34% drift) — highly inconsistent run-to-run performance

#### 3.2.4 Use Case Recommendation

- ✓ Use when human readability is required alongside machine processing (debugging, audit trails, manual review)
- ✓ Use when filtering is the dominant query type and data is optional/sparse
- ❌ Avoid when token cost matters — 72% overhead for no accuracy benefit is indefensible
- ❌ Avoid when aggregation is a primary use case — consistently poor across both variants
- ❌ Avoid when run-to-run consistency is required — high accuracy variance on optional data

#### 3.2.5 Trade-offs

- Pays 72% more tokens than JSON_COMPACT (mandatory) for a 0.81% raw accuracy gain — poor value by any measure; the efficiency score gap (52.67 vs 74.57) makes the cost explicit
- The only scenario where JSON_PRETTY has a legitimate advantage over JSON_COMPACT is filtering on optional data (68.25% vs 63.81%), but this single-category improvement does not justify the overall token premium
- High output token variance on optional data (−11.46% to +12.34%) indicates the model struggles with consistency when reading pretty-printed sparse structures, making it unreliable for production workloads

### 3.3 Detailed Analysis: TOON_DEFAULT

#### 3.3.1 Performance Summary

- Token Duration Range: 40 - 54 seconds
- Token Cost Range: 14206 - 14492 tokens
- Wasted Token Range: 4879 - 4975 tokens
- Accuracy Range: 64.98 - 66.33%
- Efficiency Score Range: 63.39 - 63.52

#### 3.3.2 Strengths

- **Best weighted accuracy on optional data** (66.84%) and highest structure awareness on optional (77.78%) — outperforms all formats on schema understanding when data is sparse
- **Most stable efficiency score across variants** (Δ −0.13) — the most consistent format when data density changes
- **Lowest token delta between mandatory and optional** among non-XML formats (Δ −286 tokens, −1.97%) — minimal format overhead for missing optional fields
- **Fastest total processing time** (lowest total duration: ~40–54s vs 67–113s for other formats) — relevant when API latency is a constraint

#### 3.3.3 Weaknesses

- **Aggregation degrades significantly with optional data** (57.74% man → 44.90% opt, −12.84 pp) — second-worst aggregation drop after XML_COMPACT
- **Field retrieval on optional is weakest** (65.71%) among all formats tied with YAML — extracting specific values from sparse records is less reliable
- **High accuracy variance on mandatory** (−12.47% to +5.77%) — largest run-to-run range, making individual inference less predictable
- **Moderate efficiency score** (63.39–63.52) — middle of the pack, neither cost-optimal nor accuracy-optimal

#### 3.3.4 Use Case Recommendation

- ✓ Use when structure awareness and schema understanding are the primary query type — uniquely strong (77.78% on optional)
- ✓ Use when data contains optional/sparse fields and consistent performance across variants is needed
- ✓ Use when processing latency is a constraint (fastest format overall)
- ❌ Avoid when aggregation on optional data is a primary use case — significant accuracy degradation (−12.84 pp)
- ❌ Avoid when individual run consistency is critical — high accuracy variance on mandatory data

#### 3.3.5 Trade-offs

- Occupies the middle ground between JSON_COMPACT (efficiency) and XML_COMPACT (accuracy): 38% more tokens than JSON_COMPACT, 11% fewer than XML_COMPACT, with accuracy between the two
- The strong weighted accuracy on optional (66.84%) despite moderate raw accuracy (64.98%) indicates the format handles high-weight question categories (field retrieval, structure awareness) better than low-weight ones (aggregation)
- Unique value proposition: the only format where optional data improves weighted accuracy vs mandatory (+0.73 pp weighted, vs −1.35 raw) — the format seems to benefit from reduced data density when reasoning about structure

### 3.4 Detailed Analysis: XML_COMPACT

#### 3.4.1 Performance Summary

- Token Duration Range: 76 - 80 seconds
- Token Cost Range: 12762 - 13010 tokens
- Wasted Token Range: 4197 - 4940 tokens
- Accuracy Range: 61.29 - 67.74%
- Efficiency Score Range: 64.90 - 68.71

#### 3.4.2 Strengths

- **Highest raw accuracy on mandatory data** (67.74%) — best absolute accuracy of any format in the benchmark
- **Best aggregation accuracy on mandatory** (68.25%) — uniquely strong at numerical operations on dense data, outperforming the next format (JSON_COMPACT 60.00%) by 8.25 pp
- **Highest character density** (2.551 chars/token man) — most information per token at the character level
- **Most predictable output token behaviour** (±0.33% drift on both variants) — most consistent inference behaviour
- **2nd best efficiency score on mandatory** (68.71) — strong composite value

#### 3.4.3 Weaknesses

- **Largest accuracy drop from mandatory to optional** among compact formats (−6.45 pp) — significant degradation with sparse data
- **Aggregation collapses on optional data** (68.25% → 38.10%, −30.16 pp) — catastrophic failure; worst single-category drop in the entire benchmark
- **Wasted tokens increase 17.71%** with optional data (+743 tokens) — the highest proportional waste increase of any format
- **Filtering is weakest on mandatory** (55.55%) — bottom of all formats for mandatory filtering

#### 3.4.4 Use Case Recommendation

- ✓ Use when data is dense with mandatory fields only and aggregation accuracy is critical
- ✓ Use when maximum raw accuracy on fully-populated records is the priority
- ✓ Use when predictable output volume is required (tightest output token drift)
- ❌ Avoid when data contains optional/sparse fields — both aggregation and field retrieval degrade severely
- ❌ Avoid when filtering on mandatory data matters — ranks last for mandatory filtering (55.55%)
- ❌ Avoid when any aggregation operations will be performed on optional/sparse data

#### 3.4.5 Trade-offs

- On mandatory data, XML_COMPACT is the clear accuracy winner; on optional data, it drops to 4th place (61.29%) — accuracy is highly dependent on data density
- The aggregation collapse on optional (68.25% → 38.10%) is the most dramatic single-metric failure in the benchmark; the XML tag structure appears to confuse the model when optional fields create irregular patterns
- Efficiency score on mandatory (68.71) is strong, but drops to 64.90 on optional — the gap between mandatory and optional efficiency (Δ −3.81) is the largest among compact formats, indicating XML_COMPACT's value proposition is fragile under real-world conditions where data is rarely fully populated

### 3.5 Detailed Analysis: XML_PRETTY

#### 3.5.1 Performance Summary

- Token Duration Range: 69 - 81 seconds
- Token Cost Range: 19876 - 20506 tokens
- Wasted Token Range: 7801 - 7827 tokens
- Accuracy Range: 60.75 - 61.83%
- Efficiency Score Range: 43.31 - 44.34

#### 3.5.2 Strengths

- **Most stable accuracy across mandatory/optional variants** (Δ −1.08 pp) — lowest accuracy degradation when data becomes sparse, making it the most robust format for mixed-density data
- **Highest absolute useful token count** (12679 man, 12075 opt) — most tokens applied correctly in absolute terms
- **Structure awareness improves with optional data** (+3.70 pp, 62.96% → 66.67%) — one of two formats where optional data helps structure understanding
- **Filtering improves with optional data** (+7.94 pp, 53.97% → 61.90%) — most significant filtering improvement across variants

#### 3.5.3 Weaknesses

- **Highest token cost by a wide margin** (19876–20506) — consumes ~99.8% more tokens than JSON_COMPACT optional and 59.8% more than XML_COMPACT mandatory
- **Worst composite efficiency score** (43.31–44.34) — bottom of all formats on every efficiency metric
- **Most absolute wasted tokens** (7801–7827) — nearly 40% of all tokens consumed yield no useful output
- **Extremely high output token variance on optional** (−95.61% to +50.24%) — most unstable inference behaviour in the benchmark
- **Lowest aggregation on mandatory** (49.21%) — worst among formats not named JSON_PRETTY

#### 3.5.4 Use Case Recommendation

- ✓ Use when accuracy stability across mandatory and optional variants is the single most critical requirement and token cost is unconstrained
- ✓ Use when human readability alongside machine processing is mandatory and the data is XML-native
- ❌ Avoid in any token-budget-sensitive environment — the ~2x token overhead vs XML_COMPACT delivers no meaningful accuracy advantage
- ❌ Avoid when aggregation is required — consistently poor across both variants
- ❌ Avoid when output predictability matters — extreme output token variance on optional data

#### 3.5.5 Trade-offs

- The only genuine advantage over XML_COMPACT is accuracy stability (Δ −1.08 pp vs Δ −6.45 pp) — but this costs 57.6% more tokens on mandatory; the stability gain does not justify the cost
- Despite consuming the most tokens, XML_PRETTY does not achieve the best accuracy — XML_COMPACT beats it by 5.91 pp on mandatory while using 35.5% fewer tokens; pretty-printing actively harms the accuracy-per-token ratio
- The extreme output token variance on optional (−95.61% to +50.24%) suggests the model's response strategy changes dramatically between runs when reading verbose XML with sparse data — a reliability concern beyond just token count

### 3.6 Detailed Analysis: YAML

#### 3.6.1 Performance Summary

- Token Duration Range: 67 - 86 seconds
- Token Cost Range: 14368 - 14451 tokens
- Wasted Token Range: 4866 - 5944 tokens
- Accuracy Range: 58.87 - 66.13%
- Efficiency Score Range: 58.41 - 63.73

#### 3.6.2 Strengths

- **Best field retrieval on mandatory data** (73.94%) — highest single-category accuracy in the benchmark for the mandatory variant
- **Best structure awareness on mandatory data** (70.37%) — leads all formats for schema understanding on dense data
- **Most stable filtering performance** (61.90% on both mandatory and optional, 0.00% change) — uniquely consistent filtering regardless of data density
- **Most stable total token count** across variants (+83 tokens, +0.58%) — minimal format overhead for optional fields

#### 3.6.3 Weaknesses

- **Worst accuracy on optional data** (58.87%) — lowest overall accuracy in the benchmark across any format/variant combination
- **Largest accuracy drop from mandatory to optional** (−7.26 pp) — the most fragile format under sparse data conditions
- **Worst aggregation on mandatory** (44.45%) — bottom of all formats for numerical operations on dense data
- **Largest efficiency score drop** from mandatory to optional (Δ −5.32) — worst robustness in composite efficiency
- **Wasted tokens increase 22.14%** with optional data (+1077 tokens) — second-highest proportional waste increase after XML_COMPACT

#### 3.6.4 Use Case Recommendation

- ✓ Use when data is always fully populated with mandatory fields — YAML excels on dense, complete records
- ✓ Use when field retrieval and structure awareness are the primary query types and data is never sparse
- ✓ Use when token count stability between data variants is needed for predictable API costs
- ❌ Avoid when optional/sparse data is common — 7.26 pp accuracy drop with essentially no token savings is the worst trade-off in the benchmark
- ❌ Avoid when aggregation is a use case — worst aggregation accuracy on mandatory data (44.45%)
- ❌ Avoid in production pipelines where data completeness is not guaranteed

#### 3.6.5 Trade-offs

- YAML presents a misleading picture on mandatory data: strong field retrieval (73.94%) and structure awareness (70.37%) suggest a capable format, but the aggregation floor (44.45%) and optional data collapse (58.87%) reveal fundamental limitations
- The token stability across variants (+83 tokens) is the only metric where YAML is uniquely best, but it provides no accuracy benefit — a stable token count while losing 7.26% accuracy is not a feature; it means the format consumes the same tokens to produce worse results
- For any realistic use case where data completeness is not 100% guaranteed, YAML is the riskiest format choice: the accuracy loss with optional data is the highest in the benchmark while the token savings are essentially zero

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-17
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Extended Thinking**: off
- **Structure**: nested
- **Formats Tested**: JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML
- **Record Counts**: 31
- **Total Test Cases**: 12

### 4.2 Appendix B: Benchmark Configuration
- **Field Retrieval**: 55 questions (37.50% weight)
- **Filtering**: 21 questions (20.83% weight)
- **Aggregation**: 21 questions (12.50% weight)
- **Structure Awareness**: 27 questions (29.17% weight)

---

- **Report Generated**: 2026-03-17
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **With the help of**: Claude Sonnet 4.6
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Related Benchmark Results**: 
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)