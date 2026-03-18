# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-17
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Extended Thinking**: on
- **Data Structure**: flat
- **Formats Tested**: 7 (CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: Second iteration

## Executive Summary

This benchmark evaluates token efficiency and information accuracy across 7 file formats using Claude Haiku 4.5 (claude-haiku-4-5-20251001) as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context, while a format that accurately conveys information may justify higher token cost.

### Key Findings

1. **TOON_DEFAULT dominates mandatory data** — highest weighted efficiency score (78.34), 2nd lowest token cost (7327), and fastest processing (~42s). However, optional data causes a +61.76% token explosion and -17.24 efficiency collapse, making it format-sensitive.

2. **CSV minimizes tokens but not costs** — fewest tokens (6988-7172) yet lowest accuracy (54.30-63.17%). The token savings are partially cancelled by poor information quality; wasted tokens increase disproportionately on optional data (+20.90%).

3. **JSON_COMPACT is the most robust format** — accuracy delta of only +0.26% between mandatory and optional, consistent token count (-5.62%), and best optional efficiency score (69.53). Structure awareness actually improves with optional data (+12.35%).

4. **Pretty-printed formats (XML_PRETTY, JSON_PRETTY) provide no accuracy return on their token investment** — XML_PRETTY uses 2.3x more tokens than CSV but achieves only ~7% higher accuracy. Efficiency scores (48.95-54.97) are the lowest in the benchmark.

5. **Aggregation is universally the weakest category and collapses with optional data** — all formats drop 6-25% on aggregation when switching to optional data (CSV: -25.40%, XML_PRETTY: -23.81%, YAML: -19.04%). This suggests a model capability ceiling, not a format-specific issue.

6. **Optional data degrades every format, but to very different degrees** — accuracy drops range from +0.26% (JSON_COMPACT, stable) to -8.87% (CSV, severe). TOON_DEFAULT shows the extreme case with +4525 additional tokens on optional data.

7. **Structure Awareness improves with optional data in JSON/XML formats** — JSON_PRETTY: +16.05%, JSON_COMPACT: +12.35%, XML_PRETTY: +11.11%. These formats communicate absent fields clearly via null values and missing keys, while CSV and YAML show no improvement or degradation.

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
   - Optional: CSV 6988 tokens
   - Mandatory: CSV 7172 tokens
- Lowest output token cost drift:
   - Optional: YAML ↓ -0.22% ↑ 0.43%
   - Mandatory: XML_PRETTY ↓ -0.22% ↑ 0.11%
- Highest accuracy:
   - Optional: XML_PRETTY 66.67%
   - Mandatory: YAML 70.70%
- Lowest accuracy drift:
   - Optional: CSV ↓ -0.50% ↑ 0.99%
   - Mandatory: JSON_PRETTY ↓ -1.15% ↑ 1.17%
- Most useful tokens:
   - Optional: XML_PRETTY 10191 / 15285 tokens
   - Mandatory: XML_PRETTY 11490 / 16441 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 69.53
   - Mandatory: TOON_DEFAULT 78.01
- Lowest delta (optional-mandatory):
   - Total tokens: CSV -184 tokens
   - Accuracy: JSON_COMPACT 0.26%
   - Token efficiency: JSON_PRETTY 0.45

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 15285 tokens
   - Mandatory: XML_PRETTY 16441 tokens
- Highest output token drift:
   - Optional: JSON_COMPACT ↓ -95.68% ↑ 49.76%
   - Mandatory: CSV ↓ -93.63% ↑ 48.04%
- Lowest accuracy:
   - Optional: CSV 54.30%
   - Mandatory: CSV 63.17%
- Highest accuracy drift:
   - Optional: JSON_COMPACT ↓ -6.17% ↑ 6.17%
   - Mandatory: TOON_DEFAULT ↓ -16.09% ↑ 9.19%
- Most wasted tokens:
   - Optional: XML_PRETTY 5095 / 15285 tokens
   - Mandatory: XML_PRETTY 4950 / 16441 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 50.36
   - Mandatory: XML_PRETTY 48.95
- Highest delta (optional-mandatory):
   - Total tokens: TOON_DEFAULT 4525 tokens
   - Accuracy: CSV -8.87%
   - Token efficiency: TOON_DEFAULT -17.24

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 42420s | CSV ≈ 7172 | TOON_DEFAULT ≈ 2186 | YAML ≈ 71% | TOON_DEFAULT ≈ 71% | TOON_DEFAULT ≈ 78 | TOON_DEFAULT ≈ 78  |
| XML_PRETTY (+100.3%) | TOON_DEFAULT (+2.2%) | CSV (+20.8%) | TOON_DEFAULT (-0.5%) | YAML (-0.0%) | CSV (-5.6%) | CSV (-5.0%) |
| YAML (+117.3%) | JSON_COMPACT (+32.0%) | JSON_COMPACT (+51.3%) | XML_PRETTY (-0.8%) | XML_PRETTY (-0.9%) | JSON_COMPACT (-13.3%) | JSON_COMPACT (-13.1%) |
| JSON_COMPACT (+122.1%) | XML_COMPACT (+65.7%) | YAML (+72.0%) | JSON_PRETTY (-1.4%) | JSON_PRETTY (-1.4%) | XML_COMPACT (-21.1%) | XML_COMPACT (-21.2%) |
| JSON_PRETTY (+154.8%) | YAML (+79.0%) | XML_COMPACT (+78.2%) | XML_COMPACT (-3.5%) | XML_COMPACT (-3.1%) | YAML (-21.9%) | YAML (-22.3%) |
| XML_COMPACT (+161.8%) | JSON_PRETTY (+103.0%) | JSON_PRETTY (+104.1%) | JSON_COMPACT (-5.6%) | JSON_COMPACT (-5.0%) | JSON_PRETTY (-30.1%) | JSON_PRETTY (-30.5%) |
| CSV (+199.3%) | XML_PRETTY (+129.2%) | XML_PRETTY (+126.4%) | CSV (-7.5%) | CSV (-6.3%) | XML_PRETTY (-37.2%) | XML_PRETTY (-37.6%) |


##### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 45387s | CSV ≈ 6988 | JSON_COMPACT ≈ 3099 | XML_PRETTY ≈ 67% | XML_PRETTY ≈ 69% | JSON_COMPACT ≈ 70 | JSON_COMPACT ≈ 71  |
| YAML (+52.9%) | JSON_COMPACT (+27.9%) | CSV (+3.1%) | TOON_DEFAULT (-0.7%) | JSON_PRETTY (-0.4%) | CSV (-2.2%) | CSV (-2.8%) |
| JSON_COMPACT (+68.1%) | XML_COMPACT (+60.5%) | TOON_DEFAULT (+30.0%) | JSON_PRETTY (-0.8%) | TOON_DEFAULT (-1.3%) | XML_COMPACT (-12.6%) | TOON_DEFAULT (-12.7%) |
| CSV (+69.1%) | TOON_DEFAULT (+69.6%) | XML_COMPACT (+33.3%) | JSON_COMPACT (-1.4%) | JSON_COMPACT (-1.6%) | TOON_DEFAULT (-12.6%) | XML_COMPACT (-12.9%) |
| JSON_PRETTY (+89.8%) | YAML (+72.4%) | YAML (+45.3%) | XML_COMPACT (-3.5%) | XML_COMPACT (-4.3%) | YAML (-16.9%) | YAML (-17.1%) |
| XML_PRETTY (+93.4%) | JSON_PRETTY (+95.3%) | JSON_PRETTY (+50.4%) | YAML (-4.0%) | YAML (-4.7%) | JSON_PRETTY (-20.9%) | JSON_PRETTY (-20.0%) |
| XML_COMPACT (+103.5%) | XML_PRETTY (+118.7%) | XML_PRETTY (+64.4%) | CSV (-12.4%) | CSV (-13.2%) | XML_PRETTY (-27.6%) | XML_PRETTY (-26.9%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 75% | CSV ≈ 80% | TOON_DEFAULT ≈ 68% | XML_PRETTY ≈ 71% |
| JSON_PRETTY (0.0%) | TOON_DEFAULT (-5.6%) | YAML (-0.0%) | YAML (-6.3%) |
| XML_COMPACT (0.0%) | XML_PRETTY (-7.4%) | JSON_PRETTY (-1.6%) | TOON_DEFAULT (-7.1%) |
| YAML (-0.6%) | YAML (-9.9%) | XML_PRETTY (-4.8%) | CSV (-11.1%) |
| TOON_DEFAULT (-3.6%) | XML_COMPACT (-11.1%) | XML_COMPACT (-4.8%) | JSON_PRETTY (-11.1%) |
| XML_PRETTY (-4.2%) | JSON_COMPACT (-12.3%) | JSON_COMPACT (-6.4%) | XML_COMPACT (-22.2%) |
| CSV (-14.5%) | JSON_PRETTY (-12.3%) | CSV (-15.9%) | JSON_COMPACT (-31.7%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| TOON_DEFAULT ≈ 67% | XML_PRETTY ≈ 84% | TOON_DEFAULT ≈ 67% | TOON_DEFAULT ≈ 50% |
| XML_COMPACT (0.0%) | JSON_PRETTY (-0.0%) | JSON_PRETTY (-2.4%) | JSON_COMPACT (-0.8%) |
| XML_PRETTY (0.0%) | JSON_COMPACT (-3.7%) | XML_COMPACT (-2.4%) | XML_PRETTY (-2.4%) |
| JSON_COMPACT (-1.8%) | TOON_DEFAULT (-9.3%) | YAML (-4.0%) | YAML (-4.0%) |
| YAML (-2.4%) | YAML (-13.6%) | JSON_COMPACT (-5.6%) | JSON_PRETTY (-4.0%) |
| JSON_PRETTY (-2.4%) | XML_COMPACT (-14.8%) | XML_PRETTY (-5.6%) | XML_COMPACT (-7.1%) |
| CSV (-9.1%) | CSV (-24.7%) | CSV (-10.3%) | CSV (-15.1%) |


#### 2.1.5 Conclusion

For **mandatory (dense) data**, TOON_DEFAULT is the clear winner: it combines the 2nd lowest token cost with the highest efficiency score (78.01/78.34) and fastest processing speed. CSV costs fewer tokens but sacrifices ~7% accuracy — a trade-off that rarely justifies itself in practice.

For **optional (sparse) data**, JSON_COMPACT offers the best balance: near-identical performance to mandatory data (+0.26% accuracy, -5.62% tokens) and the highest efficiency score (69.53/70.70) in the optional variant. TOON_DEFAULT is disqualified for optional data due to its +61.76% token overhead.

**Verbose formats (XML_PRETTY, JSON_PRETTY) are consistently inefficient** — spending 2-3x the tokens of compact alternatives without proportional accuracy gains. Their only niche is structure awareness on optional data, where the whitespace formatting appears to help the model recognize absent fields.

**Aggregation accuracy remains weak across all formats** (~35-71%), indicating this reflects a model capability limit at the Haiku level rather than anything format-specific. Field retrieval and structure awareness are the most reliable categories for benchmarking format quality.

**Format choice should be driven by data sparsity**: use TOON_DEFAULT or CSV for dense mandatory data when efficiency matters; use JSON_COMPACT when optional fields are present; avoid XML_PRETTY and JSON_PRETTY unless token cost is not a constraint.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Tokens/Char | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6968 | 204 | 7172 | 1.449 | 0.881 | 1.645 | 63.17 | 74.44 | 4530.552 | 2641.448 | 73.61 | 74.44 |
| CSV | opt | 6687 | 301 | 6988 | 1.432 | 0.777 | 2.427 | 54.30 | 68.73 | 3794.484 | 3193.516 | 67.98 | 68.73 |
| JSON_COMPACT | man | 9255 | 213 | 9468 | 2.152 | 0.687 | 1.715 | 65.06 | 68.05 | 6159.664 | 3308.003 | 67.66 | 68.05 |
| JSON_COMPACT | opt | 8727 | 208 | 8935 | 2.118 | 0.731 | 1.680 | 65.32 | 70.70 | 5836.560 | 3098.773 | 69.53 | 70.70 |
| JSON_PRETTY | man | 14254 | 308 | 14562 | 1.697 | 0.476 | 2.484 | 69.35 | 54.41 | 10098.747 | 4463.253 | 54.53 | 54.41 |
| JSON_PRETTY | opt | 13346 | 303 | 13649 | 1.683 | 0.483 | 2.446 | 65.86 | 56.56 | 8989.451 | 4659.882 | 54.97 | 56.56 |
| TOON_DEFAULT | man | 7018 | 309 | 7327 | 1.448 | 0.958 | 2.492 | 70.16 | 78.34 | 5140.565 | 2186.352 | 78.01 | 78.34 |
| TOON_DEFAULT | opt | 11540 | 312 | 11852 | 1.701 | 0.557 | 2.514 | 66.00 | 61.69 | 7822.100 | 4029.566 | 60.77 | 61.69 |
| XML_COMPACT | man | 11672 | 209 | 11881 | 2.370 | 0.566 | 1.683 | 67.20 | 61.72 | 7983.808 | 3896.859 | 61.51 | 61.72 |
| XML_COMPACT | opt | 10909 | 309 | 11218 | 2.345 | 0.563 | 2.489 | 63.17 | 61.59 | 7086.200 | 4131.467 | 60.79 | 61.59 |
| XML_PRETTY | man | 16137 | 304 | 16441 | 1.937 | 0.425 | 2.449 | 69.89 | 48.86 | 11490.382 | 4950.285 | 48.95 | 48.86 |
| XML_PRETTY | opt | 15076 | 209 | 15285 | 1.918 | 0.436 | 1.688 | 66.67 | 51.69 | 10190.732 | 5094.601 | 50.36 | 51.69 |
| YAML | man | 12533 | 304 | 12837 | 1.666 | 0.551 | 2.449 | 70.70 | 60.87 | 9075.524 | 3761.143 | 60.94 | 60.87 |
| YAML | opt | 11742 | 308 | 12050 | 1.653 | 0.520 | 2.481 | 62.63 | 58.62 | 7546.706 | 4502.961 | 57.78 | 58.62 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Accuracy Man (%) | Accuracy Opt (%) | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7172 | 6988 | -184 | -2.57 | 63.17 | 54.30 | -8.87 | 64.36 | 55.37 | -8.99 | 73.61 | 67.98 | -5.63 | 74.44 | 68.73 | -5.71 |
| JSON_COMPACT | 9468 | 8936 | -532 | -5.62 | 65.06 | 65.32 |  +0.26 | 65.62 | 67.00 |  +1.38 | 67.66 | 69.53 |  +1.87 | 68.05 | 70.70 |  +2.65 |
| JSON_PRETTY | 14562 | 13649 | -913 | -6.27 | 69.35 | 65.86 | -3.49 | 69.18 | 68.12 | -1.06 | 54.53 | 54.97 |  +0.45 | 54.41 | 56.56 |  +2.15 |
| TOON_DEFAULT | 7327 | 11852 |  +4525 |  +61.76 | 70.16 | 66.00 | -4.16 | 70.63 | 67.32 | -3.31 | 78.01 | 60.77 | -17.24 | 78.34 | 61.69 | -16.65 |
| XML_COMPACT | 11881 | 11218 | -663 | -5.58 | 67.20 | 63.17 | -4.03 | 67.49 | 64.31 | -3.18 | 61.51 | 60.79 | -0.72 | 61.72 | 61.59 | -0.13 |
| XML_PRETTY | 16441 | 15286 | -1155 | -7.03 | 69.89 | 66.67 | -3.22 | 69.76 | 68.57 | -1.19 | 48.95 | 50.36 |  +1.41 | 48.86 | 51.69 |  +2.83 |
| YAML | 12837 | 12050 | -787 | -6.13 | 70.70 | 62.63 | -8.07 | 70.60 | 63.83 | -6.77 | 60.94 | 57.78 | -3.16 | 60.87 | 58.62 | -2.25 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 23 | 302.957 | 0.74 | 126926 | 0.002 | 1023.60 | 126949 | 302.959 | 819.03 |
| CSV | opt | 14 | 477.643 | 0.45 | 76723 | 0.004 | 618.73 | 76737 | 477.647 | 495.08 |
| JSON_COMPACT | man | 24 | 385.625 | 0.77 | 94209 | 0.002 | 759.75 | 94233 | 385.627 | 607.96 |
| JSON_COMPACT | opt | 13 | 671.308 | 0.42 | 76265 | 0.003 | 615.04 | 76278 | 671.311 | 492.11 |
| JSON_PRETTY | man | 30 | 475.133 | 0.97 | 108073 | 0.003 | 871.55 | 108103 | 475.136 | 697.44 |
| JSON_PRETTY | opt | 36 | 370.722 | 1.16 | 86112 | 0.004 | 694.45 | 86148 | 370.726 | 555.79 |
| TOON_DEFAULT | man | 26 | 269.923 | 0.84 | 100335 | 0.004 | 809.16 | 100361 | 269.926 | 647.49 |
| TOON_DEFAULT | opt | 20 | 577.000 | 0.65 | 94789 | 0.003 | 764.42 | 94809 | 577.003 | 611.67 |
| XML_COMPACT | man | 19 | 614.316 | 0.61 | 111045 | 0.002 | 895.52 | 111064 | 614.318 | 716.54 |
| XML_COMPACT | opt | 23 | 474.304 | 0.74 | 92319 | 0.003 | 744.51 | 92342 | 474.307 | 595.76 |
| XML_PRETTY | man | 18 | 896.500 | 0.58 | 84964 | 0.004 | 685.19 | 84982 | 896.504 | 548.27 |
| XML_PRETTY | opt | 17 | 886.824 | 0.55 | 87747 | 0.002 | 707.63 | 87764 | 886.826 | 566.22 |
| YAML | man | 21 | 596.810 | 0.68 | 92165 | 0.003 | 743.27 | 92186 | 596.813 | 594.75 |
| YAML | opt | 31 | 378.774 | 1.00 | 69344 | 0.004 | 559.23 | 69375 | 378.778 | 447.58 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) | Total Man (s) | Total Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 23 | 14 | -9 | -39.13 | 126.93 | 76.72 | -50.20 | -39.55 | 126.95 | 76.74 | -50.21 | -39.55 |
| JSON_COMPACT | 24 | 13 | -11 | -45.83 | 94.21 | 76.26 | -17.94 | -19.05 | 94.23 | 76.28 | -17.96 | -19.05 |
| JSON_PRETTY | 30 | 36 |  +6 |  +20.00 | 108.07 | 86.11 | -21.96 | -20.32 | 108.10 | 86.15 | -21.95 | -20.31 |
| TOON_DEFAULT | 26 | 20 | -6 | -23.08 | 100.34 | 94.79 | -5.55 | -5.53 | 100.36 | 103.33 |  +2.97 |  +2.96 |
| XML_COMPACT | 19 | 23 |  +4 |  +21.05 | 111.05 | 92.32 | -18.73 | -16.86 | 111.06 | 92.34 | -18.72 | -16.86 |
| XML_PRETTY | 18 | 17 | -1 | -5.56 | 84.96 | 87.75 |  +2.78 |  +3.28 | 84.98 | 87.76 |  +2.78 |  +3.27 |
| YAML | 21 | 31 |  +10 |  +47.62 | 92.17 | 69.34 | -22.82 | -24.76 | 92.19 | 69.38 | -22.81 | -24.74 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Char/Token | Token/Value | Token/Object | Info/Token |
|---|---|---|---|---|---|
| CSV | man | 1.449 | 10.217 | 224.774 | 0.881 |
| CSV | opt | 1.432 | 10.597 | 215.710 | 0.777 |
| JSON_COMPACT | man | 2.152 | 13.570 | 298.548 | 0.687 |
| JSON_COMPACT | opt | 2.118 | 13.830 | 281.516 | 0.731 |
| JSON_PRETTY | man | 1.697 | 20.900 | 459.806 | 0.476 |
| JSON_PRETTY | opt | 1.683 | 21.151 | 430.516 | 0.483 |
| TOON_DEFAULT | man | 1.448 | 10.290 | 226.387 | 0.958 |
| TOON_DEFAULT | opt | 1.701 | 18.288 | 372.258 | 0.557 |
| XML_COMPACT | man | 2.370 | 17.114 | 376.516 | 0.566 |
| XML_COMPACT | opt | 2.345 | 17.288 | 351.903 | 0.563 |
| XML_PRETTY | man | 1.937 | 23.661 | 520.548 | 0.425 |
| XML_PRETTY | opt | 1.918 | 23.892 | 486.323 | 0.436 |
| YAML | man | 1.666 | 18.377 | 404.290 | 0.551 |
| YAML | opt | 1.653 | 18.609 | 378.774 | 0.520 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.449 | 1.432 | -0.017 | -1.17 | 10.217 | 10.597 |  +0.380 |  +3.72 | 224.774 | 215.710 | -9.064 | -4.03 | 0.881 | 0.777 | -0.104 | -11.80 |
| JSON_COMPACT | 2.152 | 2.118 | -0.034 | -1.58 | 13.570 | 13.830 |  +0.260 |  +1.92 | 298.548 | 281.516 | -17.032 | -5.70 | 0.687 | 0.731 |  +0.044 |  +6.40 |
| JSON_PRETTY | 1.697 | 1.683 | -0.014 | -0.82 | 20.900 | 21.151 |  +0.251 |  +1.20 | 459.806 | 430.516 | -29.290 | -6.37 | 0.476 | 0.483 |  +0.007 |  +1.47 |
| TOON_DEFAULT | 1.448 | 1.701 |  +0.253 |  +17.47 | 10.290 | 18.288 |  +7.998 |  +77.73 | 226.387 | 372.258 |  +145.871 |  +64.43 | 0.958 | 0.557 | -0.401 | -41.86 |
| XML_COMPACT | 2.370 | 2.345 | -0.025 | -1.05 | 17.114 | 17.288 |  +0.174 |  +1.02 | 376.516 | 351.903 | -24.613 | -6.54 | 0.566 | 0.563 | -0.003 | -0.53 |
| XML_PRETTY | 1.937 | 1.918 | -0.019 | -0.98 | 23.661 | 23.892 |  +0.231 |  +0.98 | 520.548 | 486.323 | -34.225 | -6.57 | 0.425 | 0.436 |  +0.011 |  +2.59 |
| YAML | 1.666 | 1.653 | -0.013 | -0.78 | 18.377 | 18.609 |  +0.232 |  +1.26 | 404.290 | 378.774 | -25.516 | -6.31 | 0.551 | 0.520 | -0.031 | -5.63 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| CSV | man | 7172 | 4531 | 2641 | 63.17 | 64.36 | 73.61 | 74.44 |
| CSV | opt | 6988 | 3794 | 3194 | 54.30 | 55.37 | 67.98 | 68.73 |
| JSON_COMPACT | man | 9468 | 6160 | 3308 | 65.06 | 65.62 | 67.66 | 68.05 |
| JSON_COMPACT | opt | 8935 | 5837 | 3099 | 65.32 | 67.00 | 69.53 | 70.70 |
| JSON_PRETTY | man | 14562 | 10099 | 4463 | 69.35 | 69.18 | 54.53 | 54.41 |
| JSON_PRETTY | opt | 13649 | 8989 | 4660 | 65.86 | 68.12 | 54.97 | 56.56 |
| TOON_DEFAULT | man | 7327 | 5141 | 2186 | 70.16 | 70.63 | 78.01 | 78.34 |
| TOON_DEFAULT | opt | 11852 | 7822 | 4030 | 66.00 | 67.32 | 60.77 | 61.69 |
| XML_COMPACT | man | 11881 | 7984 | 3897 | 67.20 | 67.49 | 61.51 | 61.72 |
| XML_COMPACT | opt | 11218 | 7086 | 4131 | 63.17 | 64.31 | 60.79 | 61.59 |
| XML_PRETTY | man | 16441 | 11490 | 4950 | 69.89 | 69.76 | 48.95 | 48.86 |
| XML_PRETTY | opt | 15285 | 10191 | 5095 | 66.67 | 68.57 | 50.36 | 51.69 |
| YAML | man | 12837 | 9076 | 3761 | 70.70 | 70.60 | 60.94 | 60.87 |
| YAML | opt | 12050 | 7547 | 4503 | 62.63 | 63.83 | 57.78 | 58.62 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7172 | 6988 | -184 | -2.57 | 4531 | 3795 | -736 | -16.25 | 2641 | 3193 |  +552 |  +20.90 | 63.17 | 54.30 | -8.87 | 73.61 | 67.978 | -5.63 | -7.64 |
| JSON_COMPACT | 9468 | 8936 | -532 | -5.62 | 6160 | 5837 | -323 | -5.25 | 3308 | 3099 | -209 | -6.32 | 65.06 | 65.32 |  +0.26 | 67.66 | 69.525 |  +1.87 |  +2.76 |
| JSON_PRETTY | 14562 | 13649 | -913 | -6.27 | 10099 | 8990 | -1109 | -10.98 | 4463 | 4660 |  +197 |  +4.41 | 69.35 | 65.86 | -3.49 | 54.53 | 54.974 |  +0.45 |  +0.82 |
| TOON_DEFAULT | 7327 | 11852 |  +4525 |  +61.75 | 5141 | 7823 |  +2682 |  +52.16 | 2186 | 4029 |  +1843 |  +84.32 | 70.16 | 66.00 | -4.16 | 78.01 | 60.765 | -17.24 | -22.10 |
| XML_COMPACT | 11881 | 11218 | -663 | -5.58 | 7984 | 7086 | -898 | -11.24 | 3897 | 4132 |  +235 |  +6.02 | 67.20 | 63.17 | -4.03 | 61.51 | 60.792 | -0.72 | -1.17 |
| XML_PRETTY | 16441 | 15286 | -1155 | -7.03 | 11490 | 10190 | -1300 | -11.31 | 4950 | 5094 |  +144 |  +2.92 | 69.89 | 66.67 | -3.22 | 48.95 | 50.36 |  +1.41 |  +2.87 |
| YAML | 12837 | 12050 | -787 | -6.13 | 9076 | 7547 | -1529 | -16.84 | 3761 | 4503 |  +742 |  +19.72 | 70.70 | 62.63 | -8.07 | 60.94 | 57.779 | -3.16 | -5.18 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| CSV | man | 78 | 46 | 0 | 63.17 |
| CSV | opt | 67 | 57 | 0 | 54.30 |
| JSON_COMPACT | man | 81 | 43 | 0 | 65.06 |
| JSON_COMPACT | opt | 81 | 43 | 0 | 65.32 |
| JSON_PRETTY | man | 86 | 38 | 0 | 69.35 |
| JSON_PRETTY | opt | 82 | 42 | 0 | 65.86 |
| TOON_DEFAULT | man | 87 | 37 | 0 | 70.16 |
| TOON_DEFAULT | opt | 82 | 42 | 0 | 66.00 |
| XML_COMPACT | man | 83 | 41 | 0 | 67.20 |
| XML_COMPACT | opt | 78 | 46 | 0 | 63.17 |
| XML_PRETTY | man | 87 | 37 | 0 | 69.89 |
| XML_PRETTY | opt | 83 | 41 | 0 | 66.67 |
| YAML | man | 88 | 36 | 0 | 70.70 |
| YAML | opt | 78 | 46 | 0 | 62.63 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 78 | 67 | -11 | -14.10 | 46 | 57 |  +11 |  +23.91 | 0 | 0 | 0 | 0.00 | 63.17 | 54.30 | -8.87 |
| JSON_COMPACT | 81 | 81 | 0 | 0.00 | 43 | 43 | 0 | 0.00 | 0 | 0 | 0 | 0.00 | 65.06 | 65.32 |  +0.26 |
| JSON_PRETTY | 86 | 82 | -4 | -4.65 | 38 | 42 |  +4 |  +10.53 | 0 | 0 | 0 | 0.00 | 69.35 | 65.86 | -3.49 |
| TOON_DEFAULT | 87 | 82 | -5 | -5.75 | 37 | 42 |  +5 |  +13.51 | 0 | 0 | 0 | 0.00 | 70.16 | 66.00 | -4.16 |
| XML_COMPACT | 83 | 78 | -5 | -6.02 | 41 | 46 |  +5 |  +12.20 | 0 | 0 | 0 | 0.00 | 67.20 | 63.17 | -4.03 |
| XML_PRETTY | 87 | 83 | -4 | -4.60 | 37 | 41 |  +4 |  +10.81 | 0 | 0 | 0 | 0.00 | 69.89 | 66.67 | -3.22 |
| YAML | 88 | 78 | -10 | -11.36 | 36 | 46 |  +10 |  +27.78 | 0 | 0 | 0 | 0.00 | 70.70 | 62.63 | -8.07 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| CSV | man | 63.17 | 60.00 | 80.25 | 52.38 | 60.32 |
| CSV | opt | 54.30 | 58.18 | 59.26 | 57.14 | 34.92 |
| JSON_COMPACT | man | 65.06 | 74.55 | 67.90 | 61.90 | 39.68 |
| JSON_COMPACT | opt | 65.32 | 65.45 | 80.25 | 61.90 | 49.21 |
| JSON_PRETTY | man | 69.35 | 74.55 | 67.90 | 66.66 | 60.32 |
| JSON_PRETTY | opt | 65.86 | 64.85 | 83.95 | 65.08 | 46.03 |
| TOON_DEFAULT | man | 70.16 | 70.91 | 74.69 | 68.26 | 64.28 |
| TOON_DEFAULT | opt | 66.00 | 67.27 | 74.69 | 67.46 | 50.00 |
| XML_COMPACT | man | 67.20 | 74.55 | 69.14 | 63.49 | 49.21 |
| XML_COMPACT | opt | 63.17 | 67.27 | 69.14 | 65.08 | 42.86 |
| XML_PRETTY | man | 69.89 | 70.30 | 72.84 | 63.49 | 71.43 |
| XML_PRETTY | opt | 66.67 | 67.27 | 83.95 | 61.90 | 47.62 |
| YAML | man | 70.70 | 73.94 | 70.37 | 68.25 | 65.08 |
| YAML | opt | 62.63 | 64.85 | 70.37 | 63.49 | 46.03 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 60.00 | 58.18 | -1.82 |
| JSON_COMPACT | 74.55 | 65.45 | -9.10 |
| JSON_PRETTY | 74.55 | 64.85 | -9.70 |
| TOON_DEFAULT | 70.91 | 67.27 | -3.64 |
| XML_COMPACT | 74.55 | 67.27 | -7.28 |
| XML_PRETTY | 70.30 | 67.27 | -3.03 |
| YAML | 73.94 | 64.85 | -9.09 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 80.25 | 59.26 | -20.99 |
| JSON_COMPACT | 67.90 | 80.25 |  +12.35 |
| JSON_PRETTY | 67.90 | 83.95 |  +16.05 |
| TOON_DEFAULT | 74.69 | 74.69 |  +0.00 |
| XML_COMPACT | 69.14 | 69.14 | 0.00 |
| XML_PRETTY | 72.84 | 83.95 |  +11.11 |
| YAML | 70.37 | 70.37 | 0.00 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 52.38 | 57.14 |  +4.76 |
| JSON_COMPACT | 61.90 | 61.90 |  +0.00 |
| JSON_PRETTY | 66.66 | 65.08 | -1.58 |
| TOON_DEFAULT | 68.26 | 67.46 | -0.80 |
| XML_COMPACT | 63.49 | 65.08 |  +1.59 |
| XML_PRETTY | 63.49 | 61.90 | -1.59 |
| YAML | 68.25 | 63.49 | -4.76 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 60.32 | 34.92 | -25.40 |
| JSON_COMPACT | 39.68 | 49.21 |  +9.52 |
| JSON_PRETTY | 60.32 | 46.03 | -14.29 |
| TOON_DEFAULT | 64.28 | 50.00 | -14.29 |
| XML_COMPACT | 49.21 | 42.86 | -6.35 |
| XML_PRETTY | 71.43 | 47.62 | -23.81 |
| YAML | 65.08 | 46.03 | -19.04 |

## 3. Format-Specific Analysis
### 3.1 Detailed Analysis: CSV

#### 3.1.1 Performance Summary

- Token Duration Range: 77 - 127 seconds
- Token Cost Range: 6988 - 7172 tokens
- Wasted Token Range: 2641 - 3194 tokens
- Accuracy Range: 54.30 - 63.17%
- Efficiency Score Range: 67.98 - 73.61

#### 3.1.2 Strengths

- **Lowest token cost across all formats** (6988-7172 tokens) — 2.2x cheaper than XML_PRETTY; best for token-constrained environments with fully populated data
- **Best structure awareness on mandatory data** (80.25%) — the header row provides a clear and compact field schema
- **Most stable token count across variants** (only -2.57% optional vs mandatory) — predictable token budgeting
- **Filtering improves with optional data** (+4.76%) — CSV handles sparse filtering questions slightly better

#### 3.1.3 Weaknesses

- **Lowest overall accuracy** across both variants (54.30-63.17%) — worst in the benchmark on both raw and weighted accuracy
- **Structure awareness collapses on optional data** (-20.99%) — the header row cannot convey which fields are absent in individual records
- **Aggregation on optional data is severely degraded** (34.92%) — missing values in sparse rows cause significant calculation errors
- **Accuracy sensitivity to optional data is the worst** (-8.87% delta) — the flat row format fails to distinguish present vs absent optional fields

#### 3.1.4 Use Case Recommendation

- ✓ Use when token minimization is the primary constraint and data is fully populated (mandatory-only)
- ✓ Use for simple field retrieval tasks on dense, complete datasets
- ✓ Use when the downstream system consumes CSV natively and re-encoding would cost additional tokens
- ❌ Avoid when optional or sparse fields are present — accuracy degrades severely
- ❌ Avoid for aggregation tasks, especially with optional data
- ❌ Avoid when structure awareness of the data schema is important

#### 3.1.5 Trade-offs

- CSV saves ~5000-9000 tokens compared to verbose formats, but the accuracy cost is real: 9-19% below top performers
- The information value per token (0.777-0.881) is the highest in the benchmark, but only because the token count is so low — absolute useful tokens (3794-4531) are the lowest
- For mandatory data, the efficiency score (73.61) is strong enough to justify CSV if accuracy requirements are loose; for optional data, the efficiency score (67.98) no longer stands out against JSON_COMPACT (69.53)

### 3.2 Detailed Analysis: JSON_COMPACT

#### 3.2.1 Performance Summary

- Token Duration Range: 76 - 94 seconds
- Token Cost Range: 8935 - 9468 tokens
- Wasted Token Range: 3099 - 3308 tokens
- Accuracy Range: 65.06 - 65.32%
- Efficiency Score Range: 67.66 - 69.53

#### 3.2.2 Strengths

- **Most robust format across variants** — accuracy delta of only +0.26% (mandatory 65.06% → optional 65.32%), identical correct/incorrect answer counts in both variants
- **Best optional data efficiency score** (69.53/70.70 weighted) — the top performer when data includes optional fields
- **Highest chars/token ratio** (2.118-2.152) — most characters encoded per token among all formats, indicating compact tokenization of the JSON syntax
- **Structure awareness improves with optional data** (+12.35%) — absent keys in compact JSON are semantically unambiguous, helping the model understand schema gaps

#### 3.2.3 Weaknesses

- **Worst aggregation on mandatory data** (39.68%) — lowest aggregation score of all formats in that variant; bracket and delimiter noise appears to disrupt numerical extraction
- **High output token variance between runs** (up to ↓-95.68% / ↑+49.76% on optional) — inference behavior is unstable, suggesting the compact format triggers variable reasoning paths
- **28-35% more tokens than CSV** (8935-9468 vs 6988-7172) without a proportional accuracy gain over CSV on mandatory data

#### 3.2.4 Use Case Recommendation

- ✓ Use when data contains sparse optional fields and consistency across variants is required
- ✓ Use when JSON is the native format of the consuming system and re-encoding would add overhead
- ✓ Use as the default format when uncertain about data density — it degrades the least with optional fields
- ❌ Avoid for aggregation-heavy workloads, especially on mandatory data
- ❌ Avoid when inference consistency between runs is critical (high output token variance)

#### 3.2.5 Trade-offs

- JSON_COMPACT pays ~28-35% more tokens than CSV to gain ~10% accuracy on mandatory data and near-perfect variant stability (+0.26% accuracy delta vs CSV's -8.87%)
- The compact representation avoids the whitespace overhead of JSON_PRETTY while retaining the semantic clarity of JSON structure — a good middle ground between token cost and structural expressiveness
- Aggregation weakness on mandatory data is the main caveat; if aggregation tasks dominate the workload, TOON_DEFAULT or YAML are stronger choices for mandatory data

### 3.3 Detailed Analysis: JSON_PRETTY

#### 3.3.1 Performance Summary

- Token Duration Range: 86 - 108 seconds
- Token Cost Range: 13649 - 14562 tokens
- Wasted Token Range: 4463 - 4660 tokens
- Accuracy Range: 65.86 - 69.35%
- Efficiency Score Range: 54.53 - 54.97

#### 3.3.2 Strengths

- **Best structure awareness on optional data** (83.95%, tied with XML_PRETTY) — indentation and whitespace make absent optional fields unambiguous to the model
- **Most stable efficiency score across variants** (+0.45 delta) — despite higher token cost, the performance is consistent and predictable
- **Lowest accuracy drift on mandatory data** (↓-1.15% / ↑+1.17%) — the most consistent within-run accuracy of all mandatory tests
- **Balanced category performance on mandatory data** — field retrieval (74.55%), filtering (66.66%), aggregation (60.32%) are all above-average

#### 3.3.3 Weaknesses

- **2nd highest token cost** (13649-14562 tokens) — approximately 2x CSV with only moderate accuracy gains
- **Lowest efficiency score in JSON family** (54.53-54.97) — tokens are not translating into proportional accuracy gains
- **Aggregation drops significantly on optional data** (-14.29%) — whitespace formatting doesn't help with sparse numerical fields
- **No meaningful accuracy advantage over compact formats** — JSON_COMPACT achieves similar accuracy with 35-40% fewer tokens

#### 3.3.4 Use Case Recommendation

- ✓ Use when human readability alongside AI inference is required (hybrid human-AI workflows)
- ✓ Use for structure-awareness-heavy tasks with optional fields where schema understanding is critical
- ✓ Use when per-run accuracy consistency on mandatory data is important
- ❌ Avoid when token budget is a constraint — the whitespace overhead provides minimal accuracy benefit
- ❌ Avoid for aggregation tasks with optional data

#### 3.3.5 Trade-offs

- JSON_PRETTY spends ~45-108% more tokens than JSON_COMPACT for a -3.49 to +0.26% accuracy difference — the whitespace and indentation overhead delivers no meaningful accuracy improvement
- The format's only real advantage is structure awareness on optional data (+16.05% vs mandatory), but XML_PRETTY achieves the same score with clearer structural separation
- If human readability is not a requirement, JSON_COMPACT delivers similar or better accuracy at 35-40% lower token cost

### 3.4 Detailed Analysis: TOON_DEFAULT

#### 3.4.1 Performance Summary

- Token Duration Range: 42 - 45 seconds
- Token Cost Range: 7327 - 11852 tokens
- Wasted Token Range: 2186 - 4030 tokens
- Accuracy Range: 66.00 - 70.16%
- Efficiency Score Range: 60.77 - 78.01

#### 3.4.2 Strengths

- **Highest weighted efficiency score on mandatory data** (78.01/78.34) — best overall performer for dense data, combining low token cost with strong accuracy
- **Fastest processing time** (~42-45 seconds total) — up to 4.7x faster than CSV; significant advantage in throughput-sensitive applications
- **Lowest wasted tokens on mandatory data** (2186 tokens) — most efficient use of tokens in the benchmark
- **Most consistent structure awareness between variants** (74.69% in both mandatory and optional, 0.00% delta) — the format explicitly represents structure regardless of field presence
- **Best balanced category accuracy on mandatory data** — no single category collapses; filtering (68.26%) and aggregation (64.28%) are among the highest

#### 3.4.3 Weaknesses

- **Largest optional vs mandatory token degradation** (+4525 tokens, +61.76%) — the format likely uses explicit markers for absent optional fields, causing severe token inflation with sparse data
- **Largest efficiency score drop** on optional data (-17.24) — the mandatory data advantage completely disappears
- **Highest within-run accuracy drift on mandatory data** (↓-16.09% / ↑+9.19%) — most unstable inference behavior of all mandatory tests
- **Aggregation drops -14.29%** on optional data — sparse fields disrupt the custom format's numerical handling

#### 3.4.4 Use Case Recommendation

- ✓ Use for dense mandatory-only datasets where processing speed and efficiency are critical constraints
- ✓ Use when throughput matters — 4.7x speed advantage over the slowest format is significant at scale
- ✓ Use when balanced performance across all question categories is needed on mandatory data
- ❌ Avoid with sparse or optional field data — the token cost increase (+61.76%) negates all efficiency advantages
- ❌ Avoid when run-to-run consistency is required (highest within-run drift)

#### 3.4.5 Trade-offs

- On mandatory data, TOON_DEFAULT is the best format: lower tokens than all but CSV, higher accuracy than CSV (+7%), and fastest processing. The efficiency score (78.01) is the highest in the benchmark.
- On optional data, the format becomes one of the most expensive (11852 tokens, comparable to XML_COMPACT at 11218), while accuracy drops to 66.00% — no longer competitive against JSON_COMPACT (65.32%, 8935 tokens)
- The format's custom design clearly optimizes for dense records; the optional field handling appears to be a structural weakness, not a model comprehension issue

### 3.5 Detailed Analysis: XML_COMPACT

#### 3.5.1 Performance Summary

- Token Duration Range: 92 - 111 seconds
- Token Cost Range: 11218 - 11881 tokens
- Wasted Token Range: 3897 - 4131 tokens
- Accuracy Range: 63.17 - 67.20%
- Efficiency Score Range: 60.79 - 61.51

#### 3.5.2 Strengths

- **Highest chars/token ratio** (2.345-2.370) — most characters packed per token of all formats; the XML tag syntax tokenizes efficiently
- **Most stable performance between variants** — accuracy delta -4.03%, efficiency score delta -0.72% (near-identical), weighted efficiency delta -0.13%
- **Consistent structure awareness** (69.14%, identical between mandatory and optional, 0.00% delta) — tag-based structure is invariant to field presence
- **Good field retrieval consistency** (67-74%) — closing tags and attribute names aid value lookup

#### 3.5.3 Weaknesses

- **High absolute token cost** (11218-11881 tokens) despite the high chars/token ratio — XML tags add substantial character overhead that still accumulates into high token counts
- **Worst aggregation score** on mandatory data (49.21%) and optional (42.86%) among mid-tier formats — tag delimiters between values appear to disrupt numerical extraction
- **No accuracy advantage over JSON_COMPACT** (67.20% vs 65.06% mandatory) despite 20-25% more tokens
- **Below-average efficiency scores** (60.79-61.51) — mid-range costs with mid-range accuracy

#### 3.5.4 Use Case Recommendation

- ✓ Use when XML is required by the consuming system and re-encoding is not feasible
- ✓ Use for field retrieval and structure awareness tasks where variant consistency is important
- ❌ Avoid for aggregation-heavy workloads — worst aggregation performance in its token cost tier
- ❌ Avoid when token budget is a constraint — no accuracy advantage over JSON_COMPACT at 25-33% lower cost
- ❌ Avoid as a general-purpose format: no category where XML_COMPACT leads over all alternatives

#### 3.5.5 Trade-offs

- XML_COMPACT's high chars/token (2.37) is misleading: it reflects efficient tokenization of character sequences, but the XML tags are pure structural overhead — the actual data density is lower than the ratio implies
- Compared to JSON_COMPACT: 25-33% more tokens, only ~2% higher accuracy on mandatory data, nearly identical performance on optional data. There is no efficiency justification for XML_COMPACT over JSON_COMPACT unless XML is a system requirement
- The stable variant performance (-0.72% efficiency delta) is XML_COMPACT's strongest argument — if consistency across mandatory/optional data is more important than peak efficiency, XML_COMPACT is predictable

### 3.6 Detailed Analysis: XML_PRETTY

#### 3.6.1 Performance Summary

- Token Duration Range: 85 - 88 seconds
- Token Cost Range: 15285 - 16441 tokens
- Wasted Token Range: 4950 - 5095 tokens
- Accuracy Range: 66.67 - 69.89%
- Efficiency Score Range: 48.95 - 50.36

#### 3.6.2 Strengths

- **Highest accuracy on optional data** (66.67%) — best raw accuracy in the optional variant
- **Best structure awareness on optional data** (83.95%, tied with JSON_PRETTY) — whitespace formatting makes absent optional fields unambiguous
- **Best aggregation on mandatory data** (71.43%) — highest aggregation score of all formats; the verbose tag structure may help isolate numerical values
- **Most absolute useful tokens** (11490 mandatory, 10191 optional) — highest absolute information throughput despite high total token cost

#### 3.6.3 Weaknesses

- **Highest total token cost** (15285-16441 tokens) — worst token economy in the benchmark, 2.2-2.3x more expensive than CSV
- **Most wasted tokens** in absolute terms (4950-5095 tokens) — highest absolute waste despite strong accuracy
- **Lowest efficiency score** (48.95-50.36) — worst efficiency ratio in the benchmark
- **Aggregation collapses on optional data** (-23.81% drop, from 71.43% to 47.62%) — the worst aggregation degradation across all formats

#### 3.6.4 Use Case Recommendation

- ✓ Use when raw accuracy on structure awareness and optional field comprehension is the primary objective and token cost is not constrained
- ✓ Use for aggregation tasks on mandatory data where the highest accuracy is required
- ✓ Use in human-AI hybrid workflows where the file must also be human-readable
- ❌ Avoid in token-constrained environments — 2.2-2.3x the cost of CSV for modest accuracy gains
- ❌ Avoid for aggregation tasks with optional data — performance collapses dramatically
- ❌ Avoid as a general-purpose format — the efficiency score (~49-50) is the lowest in the benchmark

#### 3.6.5 Trade-offs

- XML_PRETTY achieves the highest accuracy for optional data (66.67%) and structure awareness (83.95%), but at the highest possible token cost (15285 tokens optional) — that is a 2.19x token premium over CSV for a 12.37% accuracy gain
- The aggregation strength on mandatory data (71.43%) is unique and not matched by any other format, but it completely disappears on optional data — the format is highly inconsistent across task types
- For any token-aware workload, the efficiency score (~49-50) makes XML_PRETTY an unjustifiable choice; its strengths only matter when token cost is genuinely irrelevant

### 3.7 Detailed Analysis: YAML

#### 3.7.1 Performance Summary

- Token Duration Range: 69 - 92 seconds
- Token Cost Range: 12050 - 12837 tokens
- Wasted Token Range: 3761 - 4503 tokens
- Accuracy Range: 62.63 - 70.70%
- Efficiency Score Range: 57.78 - 60.94

#### 3.7.2 Strengths

- **Highest raw accuracy on mandatory data** (70.70%) — best mandatory accuracy in the benchmark across all formats
- **2nd best aggregation on mandatory data** (65.08%) — strong numerical extraction on dense data
- **Consistent structure awareness** (70.37%, identical between mandatory and optional, 0.00% delta) — indented key-value structure is invariant to field presence
- **Stable filtering performance** across variants (-4.76% delta) — filtering degrades the least of all numerical tasks

#### 3.7.3 Weaknesses

- **2nd largest accuracy drop between variants** (-8.07%) — high sensitivity to optional data, second only to CSV (-8.87%)
- **High token cost** (12050-12837 tokens) — 68-84% more than CSV, 65-75% more than TOON_DEFAULT for mandatory data
- **Aggregation drops severely on optional data** (-19.04%, from 65.08% to 46.03%) — 3rd worst aggregation degradation
- **Low efficiency score** (57.78-60.94) — the mandatory accuracy advantage over TOON_DEFAULT (~0.5%) does not justify the 75% token overhead

#### 3.7.4 Use Case Recommendation

- ✓ Use when mandatory data accuracy is the single priority and token cost is secondary
- ✓ Use when YAML is the native data format of the source system and re-encoding adds overhead
- ✓ Use when structure awareness consistency across variants is important alongside reasonable accuracy
- ❌ Avoid with sparse optional data — accuracy degrades significantly (-8.07%)
- ❌ Avoid for aggregation tasks with optional fields — severe performance drop
- ❌ Avoid when efficiency is a consideration — TOON_DEFAULT achieves comparable accuracy at 75% fewer tokens for mandatory data

#### 3.7.5 Trade-offs

- YAML achieves the highest raw accuracy on mandatory data (70.70%) but at 75% more tokens than TOON_DEFAULT (12837 vs 7327), which scores only 0.54% lower (70.16%). That extra ~5500 tokens buys essentially nothing.
- On optional data, YAML's accuracy falls to 62.63% — below JSON_COMPACT (65.32%) at 8935 tokens. YAML is more expensive and less accurate on optional data.
- The format is readable and familiar, which may benefit human-AI workflows, but purely for AI inference, YAML's token cost is difficult to justify against either TOON_DEFAULT (mandatory) or JSON_COMPACT (optional)

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-17
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Extended Thinking**: on
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

- **Report Generated**: 2026-03-17
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **With the help of**: Claude Sonnet 4.6
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Related Benchmark Result**: [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)