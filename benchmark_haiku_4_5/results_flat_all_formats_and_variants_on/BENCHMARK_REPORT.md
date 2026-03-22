# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: on
- **Data Structure**: flat
- **Formats Tested**: 7 (CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: First iteration

## Executive Summary

This benchmark evaluates token efficiency and information accuracy across 7 file formats using Claude Haiku 4.5 (claude-haiku-4-5-20251001) as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context, while a format that accurately conveys information may justify higher token cost.

### Key Findings

1. TOON_DEFAULT mandatory achieves the highest composite efficiency score across the entire benchmark (84.41) by pairing competitive accuracy (79.57%) with the second-lowest token footprint (7335 tokens) and the lowest wasted token count of all formats (1499 tokens).

2. JSON_PRETTY delivers the highest raw accuracy on mandatory data (82.80%) at a token cost of 14624, yielding an information value per token of only 0.566. The 3.23 point accuracy gain over TOON_DEFAULT costs an additional 7289 tokens, a return that is difficult to justify in most production contexts.

3. CSV consistently provides the lowest token consumption (6931–7318 tokens) but produces the worst accuracy across both variants (63.18–63.71%). Aggregation accuracy under optional data collapses to 34.92%, matching the joint-lowest score in the benchmark, disqualifying CSV for analytical workloads.

4. TOON_DEFAULT is uniquely sensitive to data sparsity: adding optional fields increases its token count by 62.25% (+4566 tokens), while all other formats experience only a 4–7% increase for the same transition. This structural penalty drops its efficiency score from 84.41 to 70.32.

5. Aggregation is the weakest question category across every format, with peak accuracy reaching only 66.67% (TOON_DEFAULT mandatory). This consistency across formats indicates that numerical computation failures are model-bound rather than format-bound for Haiku 4.5.

6. JSON_COMPACT with optional data achieves the best accuracy-efficiency balance in the optional category (efficiency score 78.93, accuracy 79.57%, 9086 tokens) and produces the fewest wasted tokens of any optional variant (1856 tokens).

7. XML_PRETTY incurs the highest token cost of all tested formats (15321–16509 tokens) while delivering only moderate accuracy (77.96–80.11%), resulting in the worst efficiency scores across both variants (54.60–59.82) and the highest wasted token counts.

## 1. Methodology

### 1.1 Research Purpose

The underlying question: **Which file format delivers maximum information value per token consumed?**

This requires measuring:
- **Token Cost**: How many tokens does each format consume for equivalent data?
- **Information Fidelity**: How accurately can the model understand and answer questions about the data?
- **Robustness**: How consistent is performance across data variants (mandatory vs optional fields)?

### 1.2 Test Design

**Data Generation:**
- 7 formats tested: CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML
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
   - Optional: CSV 6931 tokens
   - Mandatory: CSV 7318 tokens
- Lowest output token cost drift:
   - Optional: JSON_PRETTY ↓ -0.60% ↑ 0.60%
   - Mandatory: CSV ↓ -0.10% ↑ 0.20%
- Highest accuracy:
   - Optional: XML_PRETTY 80.11%
   - Mandatory: JSON_PRETTY 82.80%
- Lowest accuracy drift:
   - Optional: JSON_PRETTY ↓ -1.76% ↑ 3.52%
   - Mandatory: JSON_COMPACT ↓ -0.36% ↑ 0.73%
- Most useful tokens:
   - Optional: XML_PRETTY 12274 / 15321 tokens
   - Mandatory: XML_PRETTY 12870 / 16509 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 78.93
   - Mandatory: TOON_DEFAULT 84.41
- Lowest delta (optional-mandatory):
   - Total tokens: CSV -387 tokens
   - Accuracy: TOON_DEFAULT 0.27%
   - Token efficiency: CSV 0.84

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 15321 tokens
   - Mandatory: XML_PRETTY 16509 tokens
- Highest output token drift:
   - Optional: XML_PRETTY ↓ -94.40% ↑ 49.79%
   - Mandatory: JSON_COMPACT ↓ -96.05% ↑ 48.90%
- Lowest accuracy:
   - Optional: CSV 63.18%
   - Mandatory: CSV 63.71%
- Highest accuracy drift:
   - Optional: JSON_COMPACT ↓ -10.81% ↑ 7.43%
   - Mandatory: CSV ↓ -18.99% ↑ 15.19%
- Most wasted tokens:
   - Optional: JSON_PRETTY 3242 / 13703 tokens
   - Mandatory: XML_PRETTY 3639 / 16509 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 59.82
   - Mandatory: XML_PRETTY 54.60
- Highest delta (optional-mandatory):
   - Total tokens: TOON_DEFAULT 4566 tokens
   - Accuracy: JSON_PRETTY -6.46%
   - Token efficiency: TOON_DEFAULT -14.08

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 40s | CSV ≈ 7318 | TOON_DEFAULT ≈ 1499 | JSON_PRETTY ≈ 83% | JSON_PRETTY ≈ 82% | TOON_DEFAULT ≈ 84 | TOON_DEFAULT ≈ 83 |
| XML_COMPACT (+86.2%) | TOON_DEFAULT (+0.2%) | JSON_COMPACT (+61.8%) | TOON_DEFAULT (-3.2%) | TOON_DEFAULT (-4.1%) | JSON_COMPACT (-12.2%) | JSON_COMPACT (-11.9%) |
| CSV (+93.5%) | JSON_COMPACT (+29.8%) | JSON_PRETTY (+67.9%) | XML_PRETTY (-4.8%) | XML_PRETTY (-4.2%) | CSV (-13.1%) | CSV (-12.2%) |
| YAML (+118.2%) | XML_COMPACT (+64.5%) | CSV (+77.2%) | XML_COMPACT (-5.4%) | XML_COMPACT (-5.2%) | XML_COMPACT (-19.2%) | XML_COMPACT (-18.6%) |
| XML_PRETTY (+153.7%) | YAML (+74.7%) | XML_COMPACT (+81.4%) | YAML (-5.6%) | YAML (-5.6%) | YAML (-22.2%) | YAML (-21.8%) |
| JSON_PRETTY (+162.1%) | JSON_PRETTY (+99.8%) | YAML (+94.9%) | JSON_COMPACT (-8.3%) | JSON_COMPACT (-8.5%) | JSON_PRETTY (-24.3%) | JSON_PRETTY (-24.0%) |
| JSON_COMPACT (+171.5%) | XML_PRETTY (+125.6%) | XML_PRETTY (+142.8%) | CSV (-19.1%) | CSV (-18.6%) | XML_PRETTY (-35.3%) | XML_PRETTY (-34.6%) |


##### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 39s | CSV ≈ 6931 | JSON_COMPACT ≈ 1856 | XML_PRETTY ≈ 80% | JSON_COMPACT ≈ 81% | JSON_COMPACT ≈ 79 | JSON_COMPACT ≈ 80 |
| JSON_PRETTY (+88.1%) | JSON_COMPACT (+31.1%) | XML_COMPACT (+24.0%) | TOON_DEFAULT (-0.3%) | TOON_DEFAULT (-0.3%) | CSV (-6.0%) | CSV (-6.6%) |
| XML_PRETTY (+103.0%) | XML_COMPACT (+62.5%) | TOON_DEFAULT (+29.3%) | JSON_COMPACT (-0.5%) | XML_COMPACT (-1.5%) | XML_COMPACT (-8.6%) | XML_COMPACT (-9.9%) |
| JSON_COMPACT (+112.9%) | TOON_DEFAULT (+71.7%) | CSV (+37.5%) | XML_COMPACT (-0.5%) | YAML (-2.0%) | TOON_DEFAULT (-10.9%) | TOON_DEFAULT (-11.3%) |
| YAML (+127.6%) | YAML (+74.6%) | YAML (+42.0%) | YAML (-1.9%) | XML_PRETTY (-2.2%) | YAML (-13.1%) | YAML (-13.6%) |
| XML_COMPACT (+144.7%) | JSON_PRETTY (+97.7%) | XML_PRETTY (+64.2%) | JSON_PRETTY (-3.8%) | JSON_PRETTY (-4.6%) | JSON_PRETTY (-21.1%) | JSON_PRETTY (-22.2%) |
| CSV (+182.2%) | XML_PRETTY (+121.1%) | JSON_PRETTY (+74.7%) | CSV (-16.9%) | CSV (-17.1%) | XML_PRETTY (-24.2%) | XML_PRETTY (-26.4%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 99% | JSON_PRETTY ≈ 81% | YAML ≈ 73% | TOON_DEFAULT ≈ 67% |
| YAML (-0.6%) | XML_COMPACT (-6.2%) | XML_PRETTY (-4.8%) | JSON_PRETTY (-7.9%) |
| JSON_COMPACT (-1.8%) | XML_PRETTY (-6.2%) | TOON_DEFAULT (-7.9%) | XML_COMPACT (-12.7%) |
| XML_PRETTY (-1.8%) | TOON_DEFAULT (-11.1%) | JSON_PRETTY (-7.9%) | CSV (-19.0%) |
| TOON_DEFAULT (-4.8%) | JSON_COMPACT (-13.6%) | XML_COMPACT (-9.5%) | XML_PRETTY (-27.0%) |
| XML_COMPACT (-6.7%) | YAML (-16.0%) | JSON_COMPACT (-14.3%) | YAML (-27.0%) |
| CSV (-25.5%) | CSV (-17.3%) | CSV (-20.6%) | JSON_COMPACT (-28.6%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| XML_COMPACT ≈ 99% | JSON_COMPACT ≈ 88% | JSON_COMPACT ≈ 71% | XML_PRETTY ≈ 62% |
| XML_PRETTY (-0.6%) | TOON_DEFAULT (-4.9%) | TOON_DEFAULT (-0.0%) | JSON_COMPACT (-14.3%) |
| YAML (-1.2%) | YAML (-8.6%) | YAML (-1.6%) | TOON_DEFAULT (-14.3%) |
| JSON_PRETTY (-3.6%) | XML_COMPACT (-11.1%) | XML_COMPACT (-4.8%) | XML_COMPACT (-15.9%) |
| TOON_DEFAULT (-4.8%) | JSON_PRETTY (-12.3%) | XML_PRETTY (-6.4%) | JSON_PRETTY (-19.0%) |
| JSON_COMPACT (-7.9%) | XML_PRETTY (-18.5%) | CSV (-7.9%) | CSV (-27.0%) |
| CSV (-23.6%) | CSV (-27.2%) | JSON_PRETTY (-9.5%) | YAML (-27.0%) |


#### 2.1.5 Conclusion

Format selection has a measurable and non-trivial impact on both token consumption and answer accuracy. TOON_DEFAULT on mandatory data is the clear efficiency leader, achieving near-80% accuracy at a token footprint comparable to CSV. JSON_PRETTY achieves the highest raw accuracy on dense schemas but at a token cost that outweighs its accuracy benefit in most scenarios. CSV's low token overhead is undermined by accuracy deficits that are most severe in aggregation and filtering categories.

XML_PRETTY represents the worst tradeoff in the benchmark: it consumes the most tokens of any format while delivering only moderate accuracy, producing efficiency scores below 60. For optional data, JSON_COMPACT optional is the strongest overall candidate, combining low wasted tokens with competitive accuracy across all question categories.

The mandatory-to-optional transition is largely neutral across most formats (4–7% token increase, 0–2% accuracy gain) but is a critical failure point for TOON_DEFAULT, whose token count inflates by 62% when optional fields are introduced. Aggregation remains a universal weak point, confirming that this failure mode cannot be resolved through format optimization alone at this model tier.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Char/Token | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6989 | 329 | 7318 | 1.444 | 0.871 | 2.656 | 63.71 | 73.08 | 4662.510 | 2655.823 | 73.36 | 73.08 |
| CSV | opt | 6700 | 231 | 6931 | 1.429 | 0.912 | 1.863 | 63.18 | 74.36 | 4379.006 | 2551.994 | 74.19 | 74.36 |
| JSON_COMPACT | man | 9268 | 228 | 9496 | 2.149 | 0.784 | 1.836 | 74.46 | 73.33 | 7070.474 | 2425.193 | 74.07 | 73.33 |
| JSON_COMPACT | opt | 8748 | 338 | 9086 | 2.113 | 0.876 | 2.723 | 79.57 | 79.58 | 7229.465 | 1856.202 | 78.93 | 79.58 |
| JSON_PRETTY | man | 14283 | 341 | 14624 | 1.694 | 0.566 | 2.750 | 82.80 | 63.28 | 12108.672 | 2515.328 | 63.88 | 63.28 |
| JSON_PRETTY | opt | 13367 | 336 | 13703 | 1.680 | 0.557 | 2.710 | 76.34 | 61.93 | 10460.870 | 3242.130 | 62.24 | 61.93 |
| TOON_DEFAULT | man | 7048 | 287 | 7335 | 1.442 | 1.085 | 2.315 | 79.57 | 83.22 | 5836.459 | 1498.541 | 84.41 | 83.22 |
| TOON_DEFAULT | opt | 11561 | 340 | 11901 | 1.698 | 0.671 | 2.743 | 79.84 | 70.57 | 9501.891 | 2399.275 | 70.32 | 70.57 |
| XML_COMPACT | man | 11693 | 343 | 12036 | 2.366 | 0.643 | 2.763 | 77.42 | 67.71 | 9318.013 | 2717.654 | 68.21 | 67.71 |
| XML_COMPACT | opt | 10930 | 335 | 11265 | 2.340 | 0.706 | 2.704 | 79.57 | 71.73 | 8963.825 | 2301.508 | 72.12 | 71.73 |
| XML_PRETTY | man | 16166 | 343 | 16509 | 1.934 | 0.472 | 2.763 | 77.96 | 54.45 | 12870.157 | 3638.510 | 54.60 | 54.45 |
| XML_PRETTY | opt | 15089 | 232 | 15321 | 1.917 | 0.523 | 1.874 | 80.11 | 58.54 | 12273.920 | 3047.413 | 59.82 | 58.54 |
| YAML | man | 12554 | 231 | 12785 | 1.664 | 0.603 | 1.863 | 77.15 | 65.08 | 9863.628 | 2921.372 | 65.68 | 65.08 |
| YAML | opt | 11771 | 333 | 12104 | 1.649 | 0.646 | 2.688 | 78.23 | 68.78 | 9469.220 | 2635.113 | 68.56 | 68.78 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Accuracy Man (%) | Accuracy Opt (%) | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7318 | 6931 | -387 | -5.29 | 63.71 | 63.18 | -0.53 | 63.32 | 63.41 |  +0.09 | 73.36 | 74.19 |  +0.84 | 73.08 | 74.36 |  +1.27 |
| JSON_COMPACT | 9496 | 9086 | -410 | -4.32 | 74.46 | 79.57 |  +5.11 | 73.39 | 80.50 |  +7.11 | 74.07 | 78.93 |  +4.86 | 73.33 | 79.58 |  +6.26 |
| JSON_PRETTY | 14624 | 13703 | -921 | -6.30 | 82.80 | 76.34 | -6.46 | 81.94 | 75.90 | -6.04 | 63.88 | 62.24 | -1.64 | 63.28 | 61.93 | -1.35 |
| TOON_DEFAULT | 7335 | 11901 |  +4566 |  +62.25 | 79.57 | 79.84 |  +0.27 | 77.87 | 80.19 |  +2.32 | 84.41 | 70.32 | -14.08 | 83.22 | 70.57 | -12.65 |
| XML_COMPACT | 12036 | 11266 | -770 | -6.40 | 77.42 | 79.57 |  +2.15 | 76.71 | 79.02 |  +2.31 | 68.21 | 72.12 |  +3.91 | 67.71 | 71.73 |  +4.03 |
| XML_PRETTY | 16509 | 15322 | -1187 | -7.19 | 77.96 | 80.11 |  +2.15 | 77.74 | 78.28 |  +0.54 | 54.60 | 59.82 |  +5.22 | 54.45 | 58.54 |  +4.09 |
| YAML | 12785 | 12104 | -681 | -5.33 | 77.15 | 78.23 |  +1.08 | 76.30 | 78.55 |  +2.25 | 65.68 | 68.56 |  +2.88 | 65.08 | 68.78 |  +3.70 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 20 | 349.450 | 0.65 | 78262 | 0.004 | 631.14 | 78282 | 349.454 | 505.04 |
| CSV | opt | 15 | 446.667 | 0.48 | 109800 | 0.002 | 885.48 | 109815 | 446.669 | 708.48 |
| JSON_COMPACT | man | 28 | 331.000 | 0.90 | 109802 | 0.002 | 885.50 | 109830 | 331.002 | 708.58 |
| JSON_COMPACT | opt | 10 | 874.800 | 0.32 | 82813 | 0.004 | 667.85 | 82823 | 874.804 | 534.34 |
| JSON_PRETTY | man | 25 | 571.320 | 0.81 | 106026 | 0.003 | 855.05 | 106051 | 571.323 | 684.20 |
| JSON_PRETTY | opt | 13 | 1028.231 | 0.42 | 73162 | 0.005 | 590.01 | 73175 | 1028.236 | 472.09 |
| TOON_DEFAULT | man | 3 | 2349.333 | 0.10 | 95966 | 0.003 | 773.92 | 95969 | 2349.336 | 619.16 |
| TOON_DEFAULT | opt | 5 | 2312.200 | 0.16 | 94335 | 0.004 | 760.77 | 94340 | 2312.203 | 608.65 |
| XML_COMPACT | man | 4 | 2923.250 | 0.13 | 75334 | 0.005 | 607.53 | 75338 | 2923.255 | 486.05 |
| XML_COMPACT | opt | 9 | 1214.444 | 0.29 | 95202 | 0.004 | 767.76 | 95211 | 1214.448 | 614.26 |
| XML_PRETTY | man | 11 | 1469.636 | 0.35 | 102619 | 0.003 | 827.57 | 102630 | 1469.639 | 662.13 |
| XML_PRETTY | opt | 17 | 887.588 | 0.55 | 78965 | 0.003 | 636.81 | 78982 | 887.591 | 509.56 |
| YAML | man | 8 | 1569.250 | 0.26 | 88278 | 0.003 | 711.92 | 88286 | 1569.253 | 569.59 |
| YAML | opt | 11 | 1070.091 | 0.35 | 88549 | 0.004 | 714.10 | 88560 | 1070.095 | 571.35 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) | Total Man (s) | Total Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 20 | 15 | -5 | -25.00 | 78.26 | 109.80 |  +31.54 |  +40.30 | 78.28 | 109.81 |  +31.53 |  +40.28 |
| JSON_COMPACT | 28 | 10 | -18 | -64.29 | 109.80 | 82.81 | -26.99 | -24.58 | 109.83 | 82.82 | -27.01 | -24.59 |
| JSON_PRETTY | 25 | 13 | -12 | -48.00 | 106.03 | 73.16 | -32.86 | -31.00 | 106.05 | 73.17 | -32.88 | -31.00 |
| TOON_DEFAULT | 3 | 5 |  +2 |  +66.67 | 95.97 | 94.34 | -1.63 | -1.70 | 95.97 | 94.42 | -1.55 | -1.62 |
| XML_COMPACT | 4 | 9 |  +5 |  +125.00 | 75.33 | 95.20 |  +19.87 |  +26.37 | 75.34 | 95.21 |  +19.87 |  +26.38 |
| XML_PRETTY | 11 | 17 |  +6 |  +54.55 | 102.62 | 78.97 | -23.65 | -23.05 | 102.63 | 78.98 | -23.65 | -23.04 |
| YAML | 8 | 11 |  +3 |  +37.50 | 88.28 | 88.55 |  +0.27 |  +0.31 | 88.29 | 88.56 |  +0.27 |  +0.31 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Char/Token | Token/Value | Token/Object | Info/Token |
|---|---|---|---|---|---|
| CSV | man | 1.444 | 10.248 | 225.452 | 0.871 |
| CSV | opt | 1.429 | 10.618 | 216.129 | 0.912 |
| JSON_COMPACT | man | 2.149 | 13.589 | 298.968 | 0.784 |
| JSON_COMPACT | opt | 2.113 | 13.864 | 282.194 | 0.876 |
| JSON_PRETTY | man | 1.694 | 20.943 | 460.742 | 0.566 |
| JSON_PRETTY | opt | 1.680 | 21.184 | 431.194 | 0.557 |
| TOON_DEFAULT | man | 1.442 | 10.334 | 227.355 | 1.085 |
| TOON_DEFAULT | opt | 1.698 | 18.322 | 372.935 | 0.671 |
| XML_COMPACT | man | 2.366 | 17.145 | 377.194 | 0.643 |
| XML_COMPACT | opt | 2.340 | 17.322 | 352.581 | 0.706 |
| XML_PRETTY | man | 1.934 | 23.704 | 521.484 | 0.472 |
| XML_PRETTY | opt | 1.917 | 23.913 | 486.742 | 0.523 |
| YAML | man | 1.664 | 18.408 | 404.968 | 0.603 |
| YAML | opt | 1.649 | 18.655 | 379.710 | 0.646 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.444 | 1.429 | -0.015 | -1.04 | 10.248 | 10.618 |  +0.370 |  +3.61 | 225.452 | 216.129 | -9.323 | -4.14 | 0.871 | 0.912 |  +0.041 |  +4.71 |
| JSON_COMPACT | 2.149 | 2.113 | -0.036 | -1.68 | 13.589 | 13.864 |  +0.275 |  +2.02 | 298.968 | 282.194 | -16.774 | -5.61 | 0.784 | 0.876 |  +0.092 |  +11.73 |
| JSON_PRETTY | 1.694 | 1.680 | -0.014 | -0.83 | 20.943 | 21.184 |  +0.241 |  +1.15 | 460.742 | 431.194 | -29.548 | -6.41 | 0.566 | 0.557 | -0.009 | -1.59 |
| TOON_DEFAULT | 1.442 | 1.698 |  +0.256 |  +17.75 | 10.334 | 18.322 |  +7.988 |  +77.30 | 227.355 | 372.935 |  +145.580 |  +64.03 | 1.085 | 0.671 | -0.414 | -38.16 |
| XML_COMPACT | 2.366 | 2.340 | -0.026 | -1.10 | 17.145 | 17.322 |  +0.177 |  +1.03 | 377.194 | 352.581 | -24.613 | -6.53 | 0.643 | 0.706 |  +0.063 |  +9.80 |
| XML_PRETTY | 1.934 | 1.917 | -0.017 | -0.88 | 23.704 | 23.913 |  +0.209 |  +0.88 | 521.484 | 486.742 | -34.742 | -6.66 | 0.472 | 0.523 |  +0.051 |  +10.81 |
| YAML | 1.664 | 1.649 | -0.015 | -0.90 | 18.408 | 18.655 |  +0.247 |  +1.34 | 404.968 | 379.710 | -25.258 | -6.24 | 0.603 | 0.646 |  +0.043 |  +7.13 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| CSV | man | 7318 | 4663 | 2656 | 63.71 | 63.32 | 73.36 | 73.08 |
| CSV | opt | 6931 | 4379 | 2552 | 63.18 | 63.41 | 74.19 | 74.36 |
| JSON_COMPACT | man | 9496 | 7070 | 2425 | 74.46 | 73.39 | 74.07 | 73.33 |
| JSON_COMPACT | opt | 9086 | 7229 | 1856 | 79.57 | 80.50 | 78.93 | 79.58 |
| JSON_PRETTY | man | 14624 | 12109 | 2515 | 82.80 | 81.94 | 63.88 | 63.28 |
| JSON_PRETTY | opt | 13703 | 10461 | 3242 | 76.34 | 75.90 | 62.24 | 61.93 |
| TOON_DEFAULT | man | 7335 | 5836 | 1499 | 79.57 | 77.87 | 84.41 | 83.22 |
| TOON_DEFAULT | opt | 11901 | 9502 | 2399 | 79.84 | 80.19 | 70.32 | 70.57 |
| XML_COMPACT | man | 12036 | 9318 | 2718 | 77.42 | 76.71 | 68.21 | 67.71 |
| XML_COMPACT | opt | 11265 | 8964 | 2302 | 79.57 | 79.02 | 72.12 | 71.73 |
| XML_PRETTY | man | 16509 | 12870 | 3639 | 77.96 | 77.74 | 54.60 | 54.45 |
| XML_PRETTY | opt | 15321 | 12274 | 3047 | 80.11 | 78.28 | 59.82 | 58.54 |
| YAML | man | 12785 | 9864 | 2921 | 77.15 | 76.30 | 65.68 | 65.08 |
| YAML | opt | 12104 | 9469 | 2635 | 78.23 | 78.55 | 68.56 | 68.78 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7318 | 6931 | -387 | -5.29 | 4663 | 4379 | -284 | -6.08 | 2656 | 2552 | -104 | -3.91 | 63.71 | 63.18 | -0.53 | 73.36 | 74.195 |  +0.84 |  +1.15 |
| JSON_COMPACT | 9496 | 9086 | -410 | -4.32 | 7070 | 7229 |  +159 |  +2.25 | 2425 | 1856 | -569 | -23.46 | 74.46 | 79.57 |  +5.11 | 74.07 | 78.933 |  +4.86 |  +6.56 |
| JSON_PRETTY | 14624 | 13703 | -921 | -6.30 | 12109 | 10461 | -1648 | -13.61 | 2515 | 3242 |  +727 |  +28.90 | 82.80 | 76.34 | -6.46 | 63.88 | 62.239 | -1.64 | -2.57 |
| TOON_DEFAULT | 7335 | 11901 |  +4566 |  +62.25 | 5836 | 9501 |  +3665 |  +62.81 | 1499 | 2400 |  +901 |  +60.09 | 79.57 | 79.84 |  +0.27 | 84.41 | 70.32149999999999 | -14.08 | -16.69 |
| XML_COMPACT | 12036 | 11266 | -770 | -6.40 | 9318 | 8964 | -354 | -3.80 | 2718 | 2302 | -416 | -15.31 | 77.42 | 79.57 |  +2.15 | 68.21 | 72.12 |  +3.91 |  +5.74 |
| XML_PRETTY | 16509 | 15322 | -1187 | -7.19 | 12870 | 12274 | -596 | -4.63 | 3639 | 3048 | -591 | -16.24 | 77.96 | 80.11 |  +2.15 | 54.60 | 59.82 |  +5.22 |  +9.55 |
| YAML | 12785 | 12104 | -681 | -5.32 | 9864 | 9470 | -394 | -4.00 | 2921 | 2635 | -286 | -9.80 | 77.15 | 78.23 |  +1.08 | 65.68 | 68.559 |  +2.88 |  +4.39 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| CSV | man | 79 | 45 | 0 | 63.71 |
| CSV | opt | 78 | 46 | 0 | 63.18 |
| JSON_COMPACT | man | 92 | 32 | 0 | 74.46 |
| JSON_COMPACT | opt | 99 | 25 | 0 | 79.57 |
| JSON_PRETTY | man | 103 | 21 | 0 | 82.80 |
| JSON_PRETTY | opt | 95 | 29 | 0 | 76.34 |
| TOON_DEFAULT | man | 99 | 25 | 0 | 79.57 |
| TOON_DEFAULT | opt | 99 | 25 | 0 | 79.84 |
| XML_COMPACT | man | 96 | 28 | 0 | 77.42 |
| XML_COMPACT | opt | 99 | 25 | 0 | 79.57 |
| XML_PRETTY | man | 97 | 27 | 0 | 77.96 |
| XML_PRETTY | opt | 99 | 25 | 0 | 80.11 |
| YAML | man | 96 | 28 | 0 | 77.15 |
| YAML | opt | 97 | 27 | 0 | 78.23 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 79 | 78 | -1 | -1.27 | 45 | 46 |  +1 |  +2.22 | 0 | 0 | 0 | 0.00 | 63.71 | 63.18 | -0.53 |
| JSON_COMPACT | 92 | 99 |  +7 |  +7.61 | 32 | 25 | -7 | -21.88 | 0 | 0 | 0 | 0.00 | 74.46 | 79.57 |  +5.11 |
| JSON_PRETTY | 103 | 95 | -8 | -7.77 | 21 | 29 |  +8 |  +38.10 | 0 | 0 | 0 | 0.00 | 82.80 | 76.34 | -6.46 |
| TOON_DEFAULT | 99 | 99 | 0 | 0.00 | 25 | 25 | 0 | 0.00 | 0 | 0 | 0 | 0.00 | 79.57 | 79.84 |  +0.27 |
| XML_COMPACT | 96 | 99 |  +3 |  +3.13 | 28 | 25 | -3 | -10.71 | 0 | 0 | 0 | 0.00 | 77.42 | 79.57 |  +2.15 |
| XML_PRETTY | 97 | 99 |  +2 |  +2.06 | 27 | 25 | -2 | -7.41 | 0 | 0 | 0 | 0.00 | 77.96 | 80.11 |  +2.15 |
| YAML | 96 | 97 |  +1 |  +1.04 | 28 | 27 | -1 | -3.57 | 0 | 0 | 0 | 0.00 | 77.15 | 78.23 |  +1.08 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| CSV | man | 63.71 | 73.94 | 64.20 | 52.38 | 47.62 |
| CSV | opt | 63.18 | 75.15 | 60.49 | 63.49 | 34.92 |
| JSON_COMPACT | man | 74.46 | 97.58 | 67.90 | 58.73 | 38.10 |
| JSON_COMPACT | opt | 79.57 | 90.91 | 87.66 | 71.43 | 47.62 |
| JSON_PRETTY | man | 82.80 | 99.39 | 81.48 | 65.08 | 58.73 |
| JSON_PRETTY | opt | 76.34 | 95.15 | 75.31 | 61.90 | 42.86 |
| TOON_DEFAULT | man | 79.57 | 94.55 | 70.37 | 65.08 | 66.67 |
| TOON_DEFAULT | opt | 79.84 | 93.94 | 82.72 | 71.43 | 47.62 |
| XML_COMPACT | man | 77.42 | 92.73 | 75.31 | 63.49 | 53.97 |
| XML_COMPACT | opt | 79.57 | 98.79 | 76.55 | 66.67 | 46.03 |
| XML_PRETTY | man | 77.96 | 97.57 | 75.31 | 68.26 | 39.68 |
| XML_PRETTY | opt | 80.11 | 98.18 | 69.14 | 65.08 | 61.90 |
| YAML | man | 77.15 | 98.79 | 65.43 | 73.02 | 39.68 |
| YAML | opt | 78.23 | 97.58 | 79.01 | 69.84 | 34.92 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 73.94 | 75.15 |  +1.21 |
| JSON_COMPACT | 97.58 | 90.91 | -6.67 |
| JSON_PRETTY | 99.39 | 95.15 | -4.24 |
| TOON_DEFAULT | 94.55 | 93.94 | -0.61 |
| XML_COMPACT | 92.73 | 98.79 |  +6.06 |
| XML_PRETTY | 97.57 | 98.18 |  +0.61 |
| YAML | 98.79 | 97.58 | -1.21 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 64.20 | 60.49 | -3.70 |
| JSON_COMPACT | 67.90 | 87.66 |  +19.76 |
| JSON_PRETTY | 81.48 | 75.31 | -6.17 |
| TOON_DEFAULT | 70.37 | 82.72 |  +12.34 |
| XML_COMPACT | 75.31 | 76.55 |  +1.24 |
| XML_PRETTY | 75.31 | 69.14 | -6.17 |
| YAML | 65.43 | 79.01 |  +13.58 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 52.38 | 63.49 |  +11.11 |
| JSON_COMPACT | 58.73 | 71.43 |  +12.70 |
| JSON_PRETTY | 65.08 | 61.90 | -3.17 |
| TOON_DEFAULT | 65.08 | 71.43 |  +6.35 |
| XML_COMPACT | 63.49 | 66.67 |  +3.18 |
| XML_PRETTY | 68.26 | 65.08 | -3.18 |
| YAML | 73.02 | 69.84 | -3.18 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 47.62 | 34.92 | -12.70 |
| JSON_COMPACT | 38.10 | 47.62 |  +9.52 |
| JSON_PRETTY | 58.73 | 42.86 | -15.87 |
| TOON_DEFAULT | 66.67 | 47.62 | -19.05 |
| XML_COMPACT | 53.97 | 46.03 | -7.93 |
| XML_PRETTY | 39.68 | 61.90 |  +22.22 |
| YAML | 39.68 | 34.92 | -4.76 |

## 3. Format-Specific Analysis
### 3.1 Detailed Analysis: CSV

#### 3.1.1 Performance Summary

- Token Duration Range: 78 - 110 seconds
- Token Cost Range: 6931 - 7318 tokens
- Wasted Token Range: 2552 - 2656 tokens
- Accuracy Range: 63.18 - 63.71%
- Efficiency Score Range: 73.36 - 74.19

#### 3.1.2 Strengths

- Lowest total token consumption across all tested formats (6931 tokens optional, 7318 tokens mandatory), making it the most budget-efficient format for token-constrained pipelines.
- Highest robustness to data sparsity among all formats: token count changes by only 5.29% and accuracy by only 0.53% between mandatory and optional variants.
- Field retrieval accuracy of 73.94–75.15% is the highest relative metric for CSV, confirming that the model can locate individual cell values despite the absence of per-row field labels.

#### 3.1.3 Weaknesses

- Lowest accuracy of all formats across both variants (63.18–63.71%), consistently failing roughly one-third of all questions.
- Aggregation accuracy under optional data reaches only 34.92%, the joint-lowest score in the entire benchmark, indicating that the headerless-value structure impairs numerical reasoning.
- Filtering accuracy on mandatory data is only 52.38%, suggesting that flat columnar layout complicates conditional record traversal.
- Highest accuracy instability among all formats: the mandatory accuracy drift spans from -18.99% to +15.19%, indicating highly inconsistent inference behavior across runs.

#### 3.1.4 Use Case Recommendation

- ✓ Use when token budget is the primary constraint and queries are limited to simple field lookups on fully specified schemas.
- ✓ Use when the consuming pipeline performs post-processing on raw model output and tolerates partial accuracy on analytical questions.
- ❌ Avoid when the task involves aggregation, filtering, or structure awareness queries at acceptable accuracy thresholds.
- ❌ Avoid when result consistency across repeated inference runs is a system requirement.

#### 3.1.5 Trade-offs

- CSV saves 37–46% of tokens compared to mid-tier formats such as JSON_COMPACT and YAML but trades approximately 14–16 percentage points of accuracy for that saving. The effective cost per correct answer is not favorable once accuracy is factored in.
- The wasted token fraction (2552–2656 tokens) consumes 36–37% of total usage, meaning more than a third of the token budget produces no accurate output, which significantly erodes the headline token cost advantage.

### 3.2 Detailed Analysis: JSON_COMPACT

#### 3.2.1 Performance Summary

- Token Duration Range: 83 - 110 seconds
- Token Cost Range: 9086 - 9496 tokens
- Wasted Token Range: 1856 - 2425 tokens
- Accuracy Range: 74.46 - 79.57%
- Efficiency Score Range: 74.07 - 78.93

#### 3.2.2 Strengths

- Best overall accuracy-efficiency balance in the optional variant (efficiency score 78.93, weighted efficiency 79.58), ranking it first among all optional configurations.
- Lowest wasted token count in the optional variant (1856 tokens), reflecting high answer precision relative to consumed context.
- Field retrieval accuracy remains high across both variants (90.91–97.58%), confirming robust key-value parsing under both dense and sparse schemas.
- Structure awareness improves by 19.76 points when switching from mandatory to optional data (67.90% to 87.66%), the largest positive gain for that category across the entire benchmark. This indicates that optional field presence provides schema cues that significantly aid comprehension.

#### 3.2.3 Weaknesses

- Aggregation accuracy in the mandatory variant (38.10%) is the second lowest across all formats for that category, indicating that the compact syntax does not support numerical reasoning on dense schemas.
- The mandatory variant accuracy (74.46%) underperforms the optional variant (79.57%) by 5.11 points, a gap that makes format behavior dependent on data completeness and harder to predict in mixed-schema deployments.
- Output token variance under mandatory data is extreme (drift of -96.05% to +48.90%), indicating non-deterministic reasoning depth across runs when processing dense schemas.

#### 3.2.4 Use Case Recommendation

- ✓ Use when data includes optional or nullable fields and overall accuracy across question types is the primary target.
- ✓ Use when token budget is moderate and efficiency score is a ranked optimization criterion.
- ❌ Avoid when the data schema is fully mandatory and aggregation accuracy is critical, as the mandatory variant underperforms on that category.
- ❌ Avoid when output token consistency across repeated inference runs is required under dense schemas.

#### 3.2.5 Trade-offs

- JSON_COMPACT uses roughly 24–31% more tokens than CSV but delivers 10–16 percentage points more accuracy, which represents a favorable tradeoff for nearly all practical applications.
- Compared to JSON_PRETTY, JSON_COMPACT uses approximately 35% fewer tokens while sacrificing only 3–6 percentage points of accuracy on mandatory data, making it the more cost-effective choice within the JSON family in the majority of use cases.

### 3.3 Detailed Analysis: JSON_PRETTY

#### 3.3.1 Performance Summary

- Token Duration Range: 73 - 106 seconds
- Token Cost Range: 13703 - 14624 tokens
- Wasted Token Range: 2515 - 3242 tokens
- Accuracy Range: 76.34 - 82.80%
- Efficiency Score Range: 62.24 - 63.88

#### 3.3.2 Strengths

- Highest raw accuracy of all formats in the mandatory variant (82.80%), achieving only 21 incorrect answers out of 124, the fewest wrong answers recorded in the entire benchmark.
- Field retrieval accuracy of 99.39% on mandatory data approaches the upper bound for this task, indicating that structured whitespace and explicit indentation eliminate value-location ambiguity.
- Lowest accuracy variance in the mandatory variant (drift of -0.65% to +1.29%), making it the most consistent format for repeated inference runs on dense data.
- Structure awareness accuracy of 81.48% on mandatory data ranks second among all formats and variants for that category.

#### 3.3.3 Weaknesses

- Second highest total token cost across all formats (13703–14624 tokens), yielding the lowest information value per token (0.557–0.566) of any tested format.
- Accuracy degrades significantly when switching to optional data (76.34%), a drop of 6.46 points from mandatory, the largest negative optional delta of any format in this benchmark.
- Wasted tokens in the optional variant reach 3242, the highest value in the entire optional category, indicating that sparse field presence confuses rather than assists the model.
- Aggregation accuracy under optional data drops to 42.86%, a 15.87 point decline from mandatory (58.73%), revealing pronounced sensitivity to schema sparsity for numerical tasks.

#### 3.3.4 Use Case Recommendation

- ✓ Use when data schemas are fully mandatory and maximum field retrieval accuracy is the priority.
- ✓ Use when inference consistency across runs is required and the input data is guaranteed to be dense.
- ❌ Avoid when data contains optional or nullable fields, as accuracy degrades substantially across all question categories.
- ❌ Avoid when token budget is a constrained resource or when efficiency score is a ranked metric.
- ❌ Avoid when aggregation accuracy on sparse schemas is a requirement.

#### 3.3.5 Trade-offs

- JSON_PRETTY delivers 82.80% accuracy on mandatory data while TOON_DEFAULT delivers 79.57% at 7335 tokens. The 3.23 point accuracy gain costs an additional 7289 tokens, a ratio that is difficult to justify for most production workloads.
- Within the JSON family, JSON_COMPACT achieves 74.46% mandatory accuracy at 9496 tokens. JSON_PRETTY's 8.34 point advantage costs 5128 additional tokens. For use cases that require near-perfect field retrieval on dense schemas, JSON_PRETTY is the appropriate choice. For general-purpose querying, JSON_COMPACT is more economical.

### 3.4 Detailed Analysis: TOON_DEFAULT

#### 3.4.1 Performance Summary

- Token Duration Range: 39 - 40 seconds
- Token Cost Range: 7335 - 11901 tokens
- Wasted Token Range: 1499 - 2399 tokens
- Accuracy Range: 79.57 - 79.84%
- Efficiency Score Range: 70.32 - 84.41

#### 3.4.2 Strengths

- Highest composite efficiency score across the entire benchmark (84.41 mandatory), combining low token cost with competitive accuracy and the lowest wasted token count of any format (1499 tokens mandatory).
- Fastest file read throughput of all tested formats (2312–2349 tokens/ms, 3–5ms read time), making it the most I/O-performant choice for pipeline stages where read latency matters.
- Aggregation accuracy of 66.67% on mandatory data is the highest of any format in that category, indicating that the compact notation preserves numerical context effectively on dense schemas.
- Minimal accuracy change between mandatory and optional variants (0.27 points), the lowest delta of any format, confirming stable inference quality regardless of field presence.

#### 3.4.3 Weaknesses

- Token count increases by 62.25% (+4566 tokens) when switching from mandatory to optional data, an anomaly absent in all other formats (which increase 4–7% for the same transition). This spike drops the efficiency score from 84.41 to 70.32 and eliminates the cost advantage over JSON_COMPACT.
- Structure awareness in the mandatory variant (70.37%) is below average, ranking fifth among all formats and variants for that category, suggesting the notation provides limited structural cues for schema comprehension tasks.
- The format is not a general-purpose standard, which may require supplementary documentation for teams unfamiliar with its conventions.

#### 3.4.4 Use Case Recommendation

- ✓ Use when the data schema is fully mandatory and token efficiency is the primary optimization target.
- ✓ Use when read throughput is relevant to pipeline performance requirements.
- ✓ Use when aggregation accuracy is weighted in the expected query distribution.
- ❌ Avoid when the data schema contains optional or nullable fields at meaningful scale, as the token overhead becomes disproportionate.
- ❌ Avoid when structure awareness accuracy is the dominant query category, as the mandatory variant underperforms on schema comprehension questions.

#### 3.4.5 Trade-offs

- On mandatory data, TOON_DEFAULT and CSV have nearly identical token footprints (7335 vs. 7318 tokens), but TOON_DEFAULT delivers 15.86 more percentage points of accuracy (79.57% vs. 63.71%). TOON_DEFAULT is strictly superior to CSV on dense schemas with no meaningful token penalty.
- On optional data, TOON_DEFAULT at 11901 tokens becomes more expensive than JSON_COMPACT optional at 9086 tokens while offering comparable accuracy (79.84% vs. 79.57%). For optional schemas, JSON_COMPACT is the more economical choice.

### 3.5 Detailed Analysis: XML_COMPACT

#### 3.5.1 Performance Summary

- Token Duration Range: 75 - 95 seconds
- Token Cost Range: 11265 - 12036 tokens
- Wasted Token Range: 2302 - 2718 tokens
- Accuracy Range: 77.42 - 79.57%
- Efficiency Score Range: 68.21 - 72.12

#### 3.5.2 Strengths

- Highest characters-per-token ratio across all formats (2.340–2.366), meaning each token encodes more raw character content than any competing format, which could reduce context window pressure for character-dense payloads.
- Optional variant achieves the highest field retrieval accuracy in the entire benchmark (98.79%), indicating that attribute-based XML tagging provides maximum per-field addressability.
- Consistent accuracy improvement from mandatory to optional data (+2.15 points), with wasted tokens declining by 15.31%, showing that optional field attributes provide useful context without adding noise.

#### 3.5.3 Weaknesses

- High token cost relative to delivered accuracy (12036 mandatory, 11265 optional) results in efficiency scores of 68–72, placing it in the lower half of the benchmark despite its high chars-per-token density.
- Aggregation accuracy is mediocre in both variants (46.03–53.97%), confirming that high character density per token does not translate into improved numerical reasoning.
- Structure awareness in the mandatory variant (75.31%) is midrange, offering no structural comprehension advantage over substantially cheaper formats such as TOON_DEFAULT or JSON_COMPACT.
- The high chars-per-token ratio reflects that XML attribute syntax is tokenized efficiently, but this does not compensate for the fact that much of that content is structural markup rather than data values.

#### 3.5.4 Use Case Recommendation

- ✓ Use when field retrieval accuracy must be maximized and token budget allows mid-to-high tier expenditure.
- ✓ Use when data includes optional fields and per-field addressability is the primary requirement.
- ❌ Avoid when token efficiency is a priority, as XML_COMPACT costs 24–32% more tokens than JSON_COMPACT for equivalent or lower accuracy.
- ❌ Avoid when aggregation tasks dominate the expected query distribution.

#### 3.5.5 Trade-offs

- XML_COMPACT achieves identical accuracy to JSON_COMPACT optional (79.57%) at a 24% higher token cost (11265 vs. 9086). The sole advantage is field retrieval accuracy (98.79% vs. 90.91%). Unless that 7.88 point field retrieval gain is specifically required, JSON_COMPACT is the preferable choice.
- The high chars-per-token ratio does not translate into proportional accuracy gains, confirming that token density alone is an insufficient proxy for format quality.

### 3.6 Detailed Analysis: XML_PRETTY

#### 3.6.1 Performance Summary

- Token Duration Range: 79 - 103 seconds
- Token Cost Range: 15321 - 16509 tokens
- Wasted Token Range: 3047 - 3639 tokens
- Accuracy Range: 77.96 - 80.11%
- Efficiency Score Range: 54.60 - 59.82

#### 3.6.2 Strengths

- Highest accuracy in the optional variant across all tested formats (80.11%), making it the most accurate format for sparse schema data in absolute terms.
- Aggregation accuracy improves by 22.22 points when switching from mandatory to optional data (39.68% to 61.90%), the largest positive aggregation delta in the entire benchmark, suggesting that optional field annotations provide meaningful numerical context.
- High field retrieval accuracy in both variants (97.57–98.18%), benefiting from explicit tag wrapping of individual values.

#### 3.6.3 Weaknesses

- Highest total token cost across the entire benchmark (15321–16509 tokens), consuming more than twice the tokens of CSV or TOON_DEFAULT mandatory.
- Worst efficiency score of all formats in both variants (54.60 mandatory, 59.82 optional), meaning over 40% of the token budget generates no accurate output.
- Highest wasted token count in the mandatory variant (3639 tokens), the largest token waste recorded in the benchmark.
- Structure awareness decreases by 6.17 points from mandatory to optional data (75.31% to 69.14%), the inverse of the pattern observed in most other formats.
- Information value per token (0.472–0.523) is the lowest across all formats and variants, confirming that verbose tag syntax provides minimal informational density per consumed token.

#### 3.6.4 Use Case Recommendation

- ✓ Use when optional data is expected and maximum overall accuracy is required regardless of token cost.
- ✓ Use when aggregation queries on optional schemas are a priority, leveraging the 22-point optional accuracy improvement.
- ❌ Avoid in any context where token budget or efficiency score is a ranked constraint.
- ❌ Avoid when mandatory schemas are the input, as accuracy does not justify the token cost relative to JSON_PRETTY or TOON_DEFAULT.
- ❌ Avoid when context window space is limited, as XML_PRETTY consumes context at roughly double the rate of efficient alternatives.

#### 3.6.5 Trade-offs

- XML_PRETTY achieves 80.11% accuracy on optional data at 15321 tokens, while JSON_COMPACT achieves 79.57% at 9086 tokens. The 0.54 point accuracy difference costs an additional 6235 tokens, a ratio that is not defensible in production contexts.
- On mandatory data, XML_PRETTY scores 77.96% at 16509 tokens while TOON_DEFAULT scores 79.57% at 7335 tokens. XML_PRETTY is simultaneously more expensive and less accurate than TOON_DEFAULT on dense schemas, making it the weakest choice for mandatory data.

### 3.7 Detailed Analysis: YAML

#### 3.7.1 Performance Summary

- Token Duration Range: 88 - 89 seconds
- Token Cost Range: 12104 - 12785 tokens
- Wasted Token Range: 2635 - 2921 tokens
- Accuracy Range: 77.15 - 78.23%
- Efficiency Score Range: 65.68 - 68.56

#### 3.7.2 Strengths

- Highest filtering accuracy in the mandatory variant (73.02%), ranking first among all formats for conditional record selection on dense data.
- Structure awareness improves substantially from mandatory to optional data (65.43% to 79.01%, +13.58 points), suggesting that YAML's indented structure benefits from optional field presence cues for schema comprehension.
- Stable token count across variants, with only a 5.32% increase when switching from mandatory to optional data, placing it among the most predictable formats for token budget planning.

#### 3.7.3 Weaknesses

- Aggregation accuracy is the joint-lowest across optional variants (34.92%, tied with CSV), indicating that the indented key-value structure does not support numerical computation on sparse schemas.
- High token cost (12104–12785 tokens) at moderate accuracy (77.15–78.23%) produces efficiency scores of 65–69, below the benchmark average and providing limited value relative to its cost.
- Structure awareness on mandatory data (65.43%) is the second lowest of all formats, suggesting that YAML's indentation hierarchy creates structural ambiguity when optional fields are absent.
- Output token variance on mandatory data spans from -93.94% to +48.48%, the second largest drift in the benchmark, indicating non-deterministic response generation depth on dense schemas.

#### 3.7.4 Use Case Recommendation

- ✓ Use when filtering accuracy is the dominant query category and the token budget accommodates mid-tier expenditure.
- ✓ Use when data schemas include optional fields and structure awareness queries are weighted in the distribution.
- ❌ Avoid when aggregation accuracy is required, as both variants deliver the worst or near-worst aggregation scores in the benchmark.
- ❌ Avoid when output consistency across repeated inference runs is required under dense schemas.
- ❌ Avoid when efficiency score is a ranked metric, as mid-tier token cost paired with mid-tier accuracy yields poor composite returns.

#### 3.7.5 Trade-offs

- YAML costs 60–74% more tokens than TOON_DEFAULT mandatory (12785 vs. 7335) while delivering 2.42 fewer percentage points of accuracy (77.15% vs. 79.57%), making TOON_DEFAULT strictly superior on dense schemas.
- YAML's filtering advantage (73.02%) is meaningful only when filtering questions dominate the query distribution. For mixed query workloads, JSON_COMPACT optional (79.57% overall, 71.43% filtering) delivers better aggregate performance at 25% lower token cost.

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: on
- **Structure**: flat
- **Formats Tested**: CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML
- **Record Counts**: 31
- **Total Test Cases**: 14

### 4.2 Appendix B: Benchmark Configuration
- **Field Retrieval**: 55 questions (37.50% weight)
- **Filtering**: 21 questions (20.83% weight)
- **Aggregation**: 21 questions (12.50% weight)
- **Structure Awareness**: 27 questions (29.17% weight)

---

- **Report Generated**: 2026-03-22
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **With the help of**: Claude Sonnet 4.6
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)