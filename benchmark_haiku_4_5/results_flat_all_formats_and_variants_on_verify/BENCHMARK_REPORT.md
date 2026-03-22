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

<ADD_CONTENT_HERE>Insert 5-7 key findings from analysis</ADD_CONTENT_HERE>

1. Finding 1

2. Finding 2

3. Finding 3

4. Finding 4

5. Finding 5

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
   - Optional: TOON_DEFAULT 79.84%
   - Mandatory: YAML 81.18%
- Lowest accuracy drift:
   - Optional: TOON_DEFAULT ↓ -2.02% ↑ 4.03%
   - Mandatory: JSON_PRETTY ↓ -1.00% ↑ 0.99%
- Most useful tokens:
   - Optional: XML_PRETTY 12204 / 15285 tokens
   - Mandatory: XML_PRETTY 12993 / 16441 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 78.00
   - Mandatory: TOON_DEFAULT 84.41
- Lowest delta (optional-mandatory):
   - Total tokens: CSV -184 tokens
   - Accuracy: TOON_DEFAULT 0.54%
   - Token efficiency: JSON_PRETTY 1.20

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 15285 tokens
   - Mandatory: XML_PRETTY 16441 tokens
- Highest output token drift:
   - Optional: JSON_COMPACT ↓ -95.68% ↑ 49.76%
   - Mandatory: CSV ↓ -93.63% ↑ 48.04%
- Lowest accuracy:
   - Optional: CSV 62.90%
   - Mandatory: CSV 69.08%
- Highest accuracy drift:
   - Optional: JSON_COMPACT ↓ -9.38% ↑ 7.28%
   - Mandatory: TOON_DEFAULT ↓ -16.61% ↑ 10.84%
- Most wasted tokens:
   - Optional: YAML 3141 / 12050 tokens
   - Mandatory: XML_PRETTY 3448 / 16441 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 59.58
   - Mandatory: XML_PRETTY 55.35
- Highest delta (optional-mandatory):
   - Total tokens: TOON_DEFAULT 4525 tokens
   - Accuracy: YAML -7.25%
   - Token efficiency: TOON_DEFAULT -13.95

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 42s | CSV ≈ 7172 | TOON_DEFAULT ≈ 1517 | YAML ≈ 81% | YAML ≈ 79% | TOON_DEFAULT ≈ 84 | TOON_DEFAULT ≈ 84 |
| XML_PRETTY (+100.3%) | TOON_DEFAULT (+2.2%) | CSV (+46.2%) | JSON_PRETTY (-0.5%) | JSON_PRETTY (-0.7%) | CSV (-7.9%) | CSV (-6.9%) |
| YAML (+117.3%) | JSON_COMPACT (+32.0%) | JSON_COMPACT (+51.0%) | TOON_DEFAULT (-1.9%) | TOON_DEFAULT (-1.1%) | JSON_COMPACT (-10.9%) | JSON_COMPACT (-11.1%) |
| JSON_COMPACT (+122.1%) | XML_COMPACT (+65.7%) | YAML (+59.3%) | XML_PRETTY (-2.2%) | XML_PRETTY (-2.0%) | XML_COMPACT (-17.8%) | XML_COMPACT (-18.3%) |
| JSON_PRETTY (+154.8%) | YAML (+79.0%) | XML_COMPACT (+68.4%) | XML_COMPACT (-2.7%) | XML_COMPACT (-2.4%) | YAML (-19.1%) | YAML (-19.9%) |
| XML_COMPACT (+161.8%) | JSON_PRETTY (+103.0%) | JSON_PRETTY (+85.8%) | JSON_COMPACT (-5.4%) | JSON_COMPACT (-4.8%) | JSON_PRETTY (-26.0%) | JSON_PRETTY (-27.1%) |
| CSV (+199.3%) | XML_PRETTY (+129.2%) | XML_PRETTY (+127.3%) | CSV (-12.1%) | CSV (-10.1%) | XML_PRETTY (-34.4%) | XML_PRETTY (-35.2%) |


##### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 45s | CSV ≈ 6988 | JSON_COMPACT ≈ 2018 | TOON_DEFAULT ≈ 80% | XML_PRETTY ≈ 80% | JSON_COMPACT ≈ 78 | JSON_COMPACT ≈ 78 |
| YAML (+52.9%) | JSON_COMPACT (+27.9%) | TOON_DEFAULT (+18.4%) | XML_PRETTY (0.0%) | TOON_DEFAULT (-0.7%) | CSV (-5.1%) | CSV (-5.2%) |
| JSON_COMPACT (+68.1%) | XML_COMPACT (+60.5%) | XML_COMPACT (+24.0%) | JSON_PRETTY (-1.6%) | JSON_PRETTY (-1.1%) | XML_COMPACT (-9.0%) | XML_COMPACT (-9.9%) |
| CSV (+69.1%) | TOON_DEFAULT (+69.6%) | CSV (+28.5%) | XML_COMPACT (-2.2%) | JSON_COMPACT (-2.5%) | TOON_DEFAULT (-9.7%) | TOON_DEFAULT (-10.3%) |
| JSON_PRETTY (+89.8%) | YAML (+72.4%) | JSON_PRETTY (+47.3%) | JSON_COMPACT (-2.4%) | XML_COMPACT (-3.1%) | YAML (-15.8%) | YAML (-16.1%) |
| XML_PRETTY (+93.4%) | JSON_PRETTY (+95.3%) | XML_PRETTY (+52.7%) | YAML (-5.9%) | YAML (-6.3%) | JSON_PRETTY (-18.4%) | JSON_PRETTY (-18.0%) |
| XML_COMPACT (+103.5%) | XML_PRETTY (+118.7%) | YAML (+55.7%) | CSV (-16.9%) | CSV (-17.1%) | XML_PRETTY (-23.6%) | XML_PRETTY (-23.6%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 100% | CSV ≈ 80% | TOON_DEFAULT ≈ 68% | XML_PRETTY ≈ 71% |
| XML_COMPACT (0.0%) | TOON_DEFAULT (-5.6%) | YAML (-0.0%) | YAML (-6.3%) |
| JSON_COMPACT (-1.2%) | XML_PRETTY (-7.4%) | JSON_PRETTY (-1.6%) | TOON_DEFAULT (-7.1%) |
| YAML (-2.4%) | YAML (-9.9%) | XML_PRETTY (-4.8%) | CSV (-11.1%) |
| TOON_DEFAULT (-8.5%) | XML_COMPACT (-11.1%) | XML_COMPACT (-4.8%) | JSON_PRETTY (-11.1%) |
| XML_PRETTY (-9.1%) | JSON_COMPACT (-12.3%) | JSON_COMPACT (-6.4%) | XML_COMPACT (-22.2%) |
| CSV (-26.7%) | JSON_PRETTY (-12.3%) | CSV (-15.9%) | JSON_COMPACT (-31.7%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| XML_COMPACT ≈ 100% | XML_PRETTY ≈ 84% | TOON_DEFAULT ≈ 67% | TOON_DEFAULT ≈ 50% |
| TOON_DEFAULT (-1.5%) | JSON_PRETTY (-0.0%) | JSON_PRETTY (-2.4%) | JSON_COMPACT (-0.8%) |
| XML_PRETTY (-3.0%) | JSON_COMPACT (-3.7%) | XML_COMPACT (-2.4%) | XML_PRETTY (-2.4%) |
| JSON_COMPACT (-7.3%) | TOON_DEFAULT (-9.3%) | YAML (-4.0%) | YAML (-4.0%) |
| JSON_PRETTY (-7.3%) | YAML (-13.6%) | JSON_COMPACT (-5.6%) | JSON_PRETTY (-4.0%) |
| YAML (-9.7%) | XML_COMPACT (-14.8%) | XML_PRETTY (-5.6%) | XML_COMPACT (-7.1%) |
| CSV (-22.4%) | CSV (-24.7%) | CSV (-10.3%) | CSV (-15.1%) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Char/Token | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6968 | 204 | 7172 | 1.449 | 0.963 | 1.645 | 69.08 | 77.94 | 4954.418 | 2217.582 | 77.74 | 77.94 |
| CSV | opt | 6687 | 301 | 6988 | 1.432 | 0.900 | 2.427 | 62.90 | 73.82 | 4395.452 | 2592.548 | 74.00 | 73.82 |
| JSON_COMPACT | man | 9255 | 213 | 9468 | 2.152 | 0.801 | 1.715 | 75.81 | 74.41 | 7177.438 | 2290.229 | 75.18 | 74.41 |
| JSON_COMPACT | opt | 8727 | 208 | 8935 | 2.118 | 0.866 | 1.680 | 77.42 | 77.86 | 6917.735 | 2017.598 | 78.00 | 77.86 |
| JSON_PRETTY | man | 14254 | 308 | 14562 | 1.697 | 0.554 | 2.484 | 80.65 | 61.09 | 11744.253 | 2817.747 | 62.44 | 61.09 |
| JSON_PRETTY | opt | 13346 | 303 | 13649 | 1.683 | 0.573 | 2.446 | 78.23 | 63.88 | 10677.873 | 2971.460 | 63.63 | 63.88 |
| TOON_DEFAULT | man | 7018 | 309 | 7327 | 1.448 | 1.082 | 2.492 | 79.30 | 83.75 | 5810.244 | 1516.672 | 84.41 | 83.75 |
| TOON_DEFAULT | opt | 11540 | 312 | 11852 | 1.701 | 0.674 | 2.514 | 79.84 | 69.88 | 9462.371 | 2389.296 | 70.45 | 69.88 |
| XML_COMPACT | man | 11672 | 209 | 11881 | 2.370 | 0.661 | 1.683 | 78.50 | 68.40 | 9326.324 | 2554.343 | 69.42 | 68.40 |
| XML_COMPACT | opt | 10909 | 309 | 11218 | 2.345 | 0.693 | 2.489 | 77.69 | 70.18 | 8715.005 | 2502.662 | 70.96 | 70.18 |
| XML_PRETTY | man | 16137 | 304 | 16441 | 1.937 | 0.481 | 2.449 | 79.03 | 54.27 | 12993.059 | 3447.608 | 55.35 | 54.27 |
| XML_PRETTY | opt | 15076 | 209 | 15285 | 1.918 | 0.522 | 1.688 | 79.84 | 59.48 | 12203.810 | 3081.523 | 59.58 | 59.48 |
| YAML | man | 12533 | 304 | 12837 | 1.666 | 0.632 | 2.449 | 81.18 | 67.08 | 10420.806 | 2415.861 | 68.27 | 67.08 |
| YAML | opt | 11742 | 308 | 12050 | 1.653 | 0.614 | 2.481 | 73.93 | 65.30 | 8908.319 | 3141.348 | 65.69 | 65.30 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Accuracy Man (%) | Accuracy Opt (%) | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7172 | 6988 | -184 | -2.57 | 69.08 | 62.90 | -6.18 | 69.36 | 62.64 | -6.72 | 77.74 | 74.00 | -3.74 | 77.94 | 73.82 | -4.12 |
| JSON_COMPACT | 9468 | 8936 | -532 | -5.62 | 75.81 | 77.42 |  +1.61 | 74.71 | 77.23 |  +2.52 | 75.18 | 78.00 |  +2.81 | 74.41 | 77.86 |  +3.45 |
| JSON_PRETTY | 14562 | 13649 | -913 | -6.27 | 80.65 | 78.23 | -2.42 | 78.73 | 78.58 | -0.15 | 62.44 | 63.63 |  +1.20 | 61.09 | 63.88 |  +2.79 |
| TOON_DEFAULT | 7327 | 11852 |  +4525 |  +61.76 | 79.30 | 79.84 |  +0.54 | 78.36 | 79.02 |  +0.66 | 84.41 | 70.45 | -13.95 | 83.75 | 69.88 | -13.87 |
| XML_COMPACT | 11881 | 11218 | -663 | -5.58 | 78.50 | 77.69 | -0.81 | 77.04 | 76.58 | -0.46 | 69.42 | 70.96 |  +1.53 | 68.40 | 70.18 |  +1.78 |
| XML_PRETTY | 16441 | 15286 | -1155 | -7.03 | 79.03 | 79.84 |  +0.81 | 77.49 | 79.70 |  +2.21 | 55.35 | 59.58 |  +4.23 | 54.27 | 59.48 |  +5.21 |
| YAML | 12837 | 12050 | -787 | -6.13 | 81.18 | 73.93 | -7.25 | 79.47 | 73.38 | -6.09 | 68.27 | 65.69 | -2.58 | 67.08 | 65.30 | -1.77 |

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
| CSV | man | 1.449 | 10.217 | 224.774 | 0.963 |
| CSV | opt | 1.432 | 10.597 | 215.710 | 0.900 |
| JSON_COMPACT | man | 2.152 | 13.570 | 298.548 | 0.801 |
| JSON_COMPACT | opt | 2.118 | 13.830 | 281.516 | 0.866 |
| JSON_PRETTY | man | 1.697 | 20.900 | 459.806 | 0.554 |
| JSON_PRETTY | opt | 1.683 | 21.151 | 430.516 | 0.573 |
| TOON_DEFAULT | man | 1.448 | 10.290 | 226.387 | 1.082 |
| TOON_DEFAULT | opt | 1.701 | 18.288 | 372.258 | 0.674 |
| XML_COMPACT | man | 2.370 | 17.114 | 376.516 | 0.661 |
| XML_COMPACT | opt | 2.345 | 17.288 | 351.903 | 0.693 |
| XML_PRETTY | man | 1.937 | 23.661 | 520.548 | 0.481 |
| XML_PRETTY | opt | 1.918 | 23.892 | 486.323 | 0.522 |
| YAML | man | 1.666 | 18.377 | 404.290 | 0.632 |
| YAML | opt | 1.653 | 18.609 | 378.774 | 0.614 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.449 | 1.432 | -0.017 | -1.17 | 10.217 | 10.597 |  +0.380 |  +3.72 | 224.774 | 215.710 | -9.064 | -4.03 | 0.963 | 0.900 | -0.063 | -6.54 |
| JSON_COMPACT | 2.152 | 2.118 | -0.034 | -1.58 | 13.570 | 13.830 |  +0.260 |  +1.92 | 298.548 | 281.516 | -17.032 | -5.70 | 0.801 | 0.866 |  +0.065 |  +8.11 |
| JSON_PRETTY | 1.697 | 1.683 | -0.014 | -0.82 | 20.900 | 21.151 |  +0.251 |  +1.20 | 459.806 | 430.516 | -29.290 | -6.37 | 0.554 | 0.573 |  +0.019 |  +3.43 |
| TOON_DEFAULT | 1.448 | 1.701 |  +0.253 |  +17.47 | 10.290 | 18.288 |  +7.998 |  +77.73 | 226.387 | 372.258 |  +145.871 |  +64.43 | 1.082 | 0.674 | -0.408 | -37.71 |
| XML_COMPACT | 2.370 | 2.345 | -0.025 | -1.05 | 17.114 | 17.288 |  +0.174 |  +1.02 | 376.516 | 351.903 | -24.613 | -6.54 | 0.661 | 0.693 |  +0.032 |  +4.84 |
| XML_PRETTY | 1.937 | 1.918 | -0.019 | -0.98 | 23.661 | 23.892 |  +0.231 |  +0.98 | 520.548 | 486.323 | -34.225 | -6.57 | 0.481 | 0.522 |  +0.041 |  +8.52 |
| YAML | 1.666 | 1.653 | -0.013 | -0.78 | 18.377 | 18.609 |  +0.232 |  +1.26 | 404.290 | 378.774 | -25.516 | -6.31 | 0.632 | 0.614 | -0.018 | -2.85 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| CSV | man | 7172 | 4954 | 2218 | 69.08 | 69.36 | 77.74 | 77.94 |
| CSV | opt | 6988 | 4395 | 2593 | 62.90 | 62.64 | 74.00 | 73.82 |
| JSON_COMPACT | man | 9468 | 7177 | 2290 | 75.81 | 74.71 | 75.18 | 74.41 |
| JSON_COMPACT | opt | 8935 | 6918 | 2018 | 77.42 | 77.23 | 78.00 | 77.86 |
| JSON_PRETTY | man | 14562 | 11744 | 2818 | 80.65 | 78.73 | 62.44 | 61.09 |
| JSON_PRETTY | opt | 13649 | 10678 | 2971 | 78.23 | 78.58 | 63.63 | 63.88 |
| TOON_DEFAULT | man | 7327 | 5810 | 1517 | 79.30 | 78.36 | 84.41 | 83.75 |
| TOON_DEFAULT | opt | 11852 | 9462 | 2389 | 79.84 | 79.02 | 70.45 | 69.88 |
| XML_COMPACT | man | 11881 | 9326 | 2554 | 78.50 | 77.04 | 69.42 | 68.40 |
| XML_COMPACT | opt | 11218 | 8715 | 2503 | 77.69 | 76.58 | 70.96 | 70.18 |
| XML_PRETTY | man | 16441 | 12993 | 3448 | 79.03 | 77.49 | 55.35 | 54.27 |
| XML_PRETTY | opt | 15285 | 12204 | 3082 | 79.84 | 79.70 | 59.58 | 59.48 |
| YAML | man | 12837 | 10421 | 2416 | 81.18 | 79.47 | 68.27 | 67.08 |
| YAML | opt | 12050 | 8908 | 3141 | 73.93 | 73.38 | 65.69 | 65.30 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7172 | 6988 | -184 | -2.57 | 4954 | 4395 | -559 | -11.28 | 2218 | 2593 |  +375 |  +16.91 | 69.08 | 62.90 | -6.18 | 77.74 | 73.998 | -3.74 | -4.82 |
| JSON_COMPACT | 9468 | 8936 | -532 | -5.62 | 7177 | 6917 | -260 | -3.62 | 2290 | 2017 | -273 | -11.91 | 75.81 | 77.42 |  +1.61 | 75.18 | 77.995 |  +2.81 |  +3.74 |
| JSON_PRETTY | 14562 | 13649 | -913 | -6.27 | 11744 | 10678 | -1066 | -9.08 | 2818 | 2972 |  +154 |  +5.45 | 80.65 | 78.23 | -2.42 | 62.44 | 63.633 |  +1.20 |  +1.92 |
| TOON_DEFAULT | 7327 | 11852 |  +4525 |  +61.75 | 5810 | 9462 |  +3652 |  +62.86 | 1517 | 2390 |  +873 |  +57.52 | 79.30 | 79.84 |  +0.54 | 84.41 | 70.453 | -13.95 | -16.53 |
| XML_COMPACT | 11881 | 11218 | -663 | -5.58 | 9326 | 8715 | -611 | -6.55 | 2554 | 2502 | -52 | -2.02 | 78.50 | 77.69 | -0.81 | 69.42 | 70.956 |  +1.53 |  +2.21 |
| XML_PRETTY | 16441 | 15286 | -1155 | -7.03 | 12993 | 12204 | -789 | -6.07 | 3448 | 3082 | -366 | -10.62 | 79.03 | 79.84 |  +0.81 | 55.35 | 59.579 |  +4.23 |  +7.63 |
| YAML | 12837 | 12050 | -787 | -6.13 | 10421 | 8909 | -1512 | -14.51 | 2416 | 3141 |  +725 |  +30.03 | 81.18 | 73.93 | -7.25 | 68.27 | 65.689 | -2.58 | -3.78 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| CSV | man | 86 | 38 | 0 | 69.08 |
| CSV | opt | 78 | 46 | 0 | 62.90 |
| JSON_COMPACT | man | 94 | 30 | 0 | 75.81 |
| JSON_COMPACT | opt | 96 | 28 | 0 | 77.42 |
| JSON_PRETTY | man | 100 | 24 | 0 | 80.65 |
| JSON_PRETTY | opt | 97 | 27 | 0 | 78.23 |
| TOON_DEFAULT | man | 98 | 26 | 0 | 79.30 |
| TOON_DEFAULT | opt | 99 | 25 | 0 | 79.84 |
| XML_COMPACT | man | 97 | 27 | 0 | 78.50 |
| XML_COMPACT | opt | 96 | 28 | 0 | 77.69 |
| XML_PRETTY | man | 98 | 26 | 0 | 79.03 |
| XML_PRETTY | opt | 99 | 25 | 0 | 79.84 |
| YAML | man | 101 | 23 | 0 | 81.18 |
| YAML | opt | 92 | 32 | 0 | 73.93 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 86 | 78 | -8 | -9.30 | 38 | 46 |  +8 |  +21.05 | 0 | 0 | 0 | 0.00 | 69.08 | 62.90 | -6.18 |
| JSON_COMPACT | 94 | 96 |  +2 |  +2.13 | 30 | 28 | -2 | -6.67 | 0 | 0 | 0 | 0.00 | 75.81 | 77.42 |  +1.61 |
| JSON_PRETTY | 100 | 97 | -3 | -3.00 | 24 | 27 |  +3 |  +12.50 | 0 | 0 | 0 | 0.00 | 80.65 | 78.23 | -2.42 |
| TOON_DEFAULT | 98 | 99 |  +1 |  +1.02 | 26 | 25 | -1 | -3.85 | 0 | 0 | 0 | 0.00 | 79.30 | 79.84 |  +0.54 |
| XML_COMPACT | 97 | 96 | -1 | -1.03 | 27 | 28 |  +1 |  +3.70 | 0 | 0 | 0 | 0.00 | 78.50 | 77.69 | -0.81 |
| XML_PRETTY | 98 | 99 |  +1 |  +1.02 | 26 | 25 | -1 | -3.85 | 0 | 0 | 0 | 0.00 | 79.03 | 79.84 |  +0.81 |
| YAML | 101 | 92 | -9 | -8.91 | 23 | 32 |  +9 |  +39.13 | 0 | 0 | 0 | 0.00 | 81.18 | 73.93 | -7.25 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| CSV | man | 69.08 | 73.33 | 80.25 | 52.38 | 60.32 |
| CSV | opt | 62.90 | 77.57 | 59.26 | 57.14 | 34.92 |
| JSON_COMPACT | man | 75.81 | 98.79 | 67.90 | 61.90 | 39.68 |
| JSON_COMPACT | opt | 77.42 | 92.73 | 80.25 | 61.90 | 49.21 |
| JSON_PRETTY | man | 80.65 | 100.00 | 67.90 | 66.66 | 60.32 |
| JSON_PRETTY | opt | 78.23 | 92.73 | 83.95 | 65.08 | 46.03 |
| TOON_DEFAULT | man | 79.30 | 91.51 | 74.69 | 68.26 | 64.28 |
| TOON_DEFAULT | opt | 79.84 | 98.48 | 74.69 | 67.46 | 50.00 |
| XML_COMPACT | man | 78.50 | 100.00 | 69.14 | 63.49 | 49.21 |
| XML_COMPACT | opt | 77.69 | 100.00 | 69.14 | 65.08 | 42.86 |
| XML_PRETTY | man | 79.03 | 90.91 | 72.84 | 63.49 | 71.43 |
| XML_PRETTY | opt | 79.84 | 96.97 | 83.95 | 61.90 | 47.62 |
| YAML | man | 81.18 | 97.57 | 70.37 | 68.25 | 65.08 |
| YAML | opt | 73.93 | 90.30 | 70.37 | 63.49 | 46.03 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 73.33 | 77.57 |  +4.24 |
| JSON_COMPACT | 98.79 | 92.73 | -6.06 |
| JSON_PRETTY | 100.00 | 92.73 | -7.27 |
| TOON_DEFAULT | 91.51 | 98.48 |  +6.97 |
| XML_COMPACT | 100.00 | 100.00 | 0.00 |
| XML_PRETTY | 90.91 | 96.97 |  +6.06 |
| YAML | 97.57 | 90.30 | -7.27 |

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
- Wasted Token Range: 2218 - 2593 tokens
- Accuracy Range: 62.90 - 69.08%
- Efficiency Score Range: 74.00 - 77.74

#### 3.1.2 Strengths

- <ADD_CONTENT_HERE>List format strengths based on category and variant analysis</ADD_CONTENT_HERE>
- 
- 

#### 3.1.3 Weaknesses

- <ADD_CONTENT_HERE>List format weaknesses and failure modes</ADD_CONTENT_HERE>
- 
- 

#### 3.1.4 Use Case Recommendation

- <ADD_CONTENT_HERE>When and why to use this format (✓ Use when, ❌ Avoid when)</ADD_CONTENT_HERE>
- 
- 

#### 3.1.5 Trade-offs

- <ADD_CONTENT_HERE>Discuss accuracy vs token cost trade-offs specific to this format</ADD_CONTENT_HERE>
- 
- 

### 3.2 Detailed Analysis: JSON_COMPACT

#### 3.2.1 Performance Summary

- Token Duration Range: 76 - 94 seconds
- Token Cost Range: 8935 - 9468 tokens
- Wasted Token Range: 2018 - 2290 tokens
- Accuracy Range: 75.81 - 77.42%
- Efficiency Score Range: 75.18 - 78.00

#### 3.2.2 Strengths

- <ADD_CONTENT_HERE>List format strengths based on category and variant analysis</ADD_CONTENT_HERE>
- 
- 

#### 3.2.3 Weaknesses

- <ADD_CONTENT_HERE>List format weaknesses and failure modes</ADD_CONTENT_HERE>
- 
- 

#### 3.2.4 Use Case Recommendation

- <ADD_CONTENT_HERE>When and why to use this format (✓ Use when, ❌ Avoid when)</ADD_CONTENT_HERE>
- 
- 

#### 3.2.5 Trade-offs

- <ADD_CONTENT_HERE>Discuss accuracy vs token cost trade-offs specific to this format</ADD_CONTENT_HERE>
- 
- 

### 3.3 Detailed Analysis: JSON_PRETTY

#### 3.3.1 Performance Summary

- Token Duration Range: 86 - 108 seconds
- Token Cost Range: 13649 - 14562 tokens
- Wasted Token Range: 2818 - 2971 tokens
- Accuracy Range: 78.23 - 80.65%
- Efficiency Score Range: 62.44 - 63.63

#### 3.3.2 Strengths

- <ADD_CONTENT_HERE>List format strengths based on category and variant analysis</ADD_CONTENT_HERE>
- 
- 

#### 3.3.3 Weaknesses

- <ADD_CONTENT_HERE>List format weaknesses and failure modes</ADD_CONTENT_HERE>
- 
- 

#### 3.3.4 Use Case Recommendation

- <ADD_CONTENT_HERE>When and why to use this format (✓ Use when, ❌ Avoid when)</ADD_CONTENT_HERE>
- 
- 

#### 3.3.5 Trade-offs

- <ADD_CONTENT_HERE>Discuss accuracy vs token cost trade-offs specific to this format</ADD_CONTENT_HERE>
- 
- 

### 3.4 Detailed Analysis: TOON_DEFAULT

#### 3.4.1 Performance Summary

- Token Duration Range: 42 - 45 seconds
- Token Cost Range: 7327 - 11852 tokens
- Wasted Token Range: 1517 - 2389 tokens
- Accuracy Range: 79.30 - 79.84%
- Efficiency Score Range: 70.45 - 84.41

#### 3.4.2 Strengths

- <ADD_CONTENT_HERE>List format strengths based on category and variant analysis</ADD_CONTENT_HERE>
- 
- 

#### 3.4.3 Weaknesses

- <ADD_CONTENT_HERE>List format weaknesses and failure modes</ADD_CONTENT_HERE>
- 
- 

#### 3.4.4 Use Case Recommendation

- <ADD_CONTENT_HERE>When and why to use this format (✓ Use when, ❌ Avoid when)</ADD_CONTENT_HERE>
- 
- 

#### 3.4.5 Trade-offs

- <ADD_CONTENT_HERE>Discuss accuracy vs token cost trade-offs specific to this format</ADD_CONTENT_HERE>
- 
- 

### 3.5 Detailed Analysis: XML_COMPACT

#### 3.5.1 Performance Summary

- Token Duration Range: 92 - 111 seconds
- Token Cost Range: 11218 - 11881 tokens
- Wasted Token Range: 2503 - 2554 tokens
- Accuracy Range: 77.69 - 78.50%
- Efficiency Score Range: 69.42 - 70.96

#### 3.5.2 Strengths

- <ADD_CONTENT_HERE>List format strengths based on category and variant analysis</ADD_CONTENT_HERE>
- 
- 

#### 3.5.3 Weaknesses

- <ADD_CONTENT_HERE>List format weaknesses and failure modes</ADD_CONTENT_HERE>
- 
- 

#### 3.5.4 Use Case Recommendation

- <ADD_CONTENT_HERE>When and why to use this format (✓ Use when, ❌ Avoid when)</ADD_CONTENT_HERE>
- 
- 

#### 3.5.5 Trade-offs

- <ADD_CONTENT_HERE>Discuss accuracy vs token cost trade-offs specific to this format</ADD_CONTENT_HERE>
- 
- 

### 3.6 Detailed Analysis: XML_PRETTY

#### 3.6.1 Performance Summary

- Token Duration Range: 85 - 88 seconds
- Token Cost Range: 15285 - 16441 tokens
- Wasted Token Range: 3082 - 3448 tokens
- Accuracy Range: 79.03 - 79.84%
- Efficiency Score Range: 55.35 - 59.58

#### 3.6.2 Strengths

- <ADD_CONTENT_HERE>List format strengths based on category and variant analysis</ADD_CONTENT_HERE>
- 
- 

#### 3.6.3 Weaknesses

- <ADD_CONTENT_HERE>List format weaknesses and failure modes</ADD_CONTENT_HERE>
- 
- 

#### 3.6.4 Use Case Recommendation

- <ADD_CONTENT_HERE>When and why to use this format (✓ Use when, ❌ Avoid when)</ADD_CONTENT_HERE>
- 
- 

#### 3.6.5 Trade-offs

- <ADD_CONTENT_HERE>Discuss accuracy vs token cost trade-offs specific to this format</ADD_CONTENT_HERE>
- 
- 

### 3.7 Detailed Analysis: YAML

#### 3.7.1 Performance Summary

- Token Duration Range: 69 - 92 seconds
- Token Cost Range: 12050 - 12837 tokens
- Wasted Token Range: 2416 - 3141 tokens
- Accuracy Range: 73.93 - 81.18%
- Efficiency Score Range: 65.69 - 68.27

#### 3.7.2 Strengths

- <ADD_CONTENT_HERE>List format strengths based on category and variant analysis</ADD_CONTENT_HERE>
- 
- 

#### 3.7.3 Weaknesses

- <ADD_CONTENT_HERE>List format weaknesses and failure modes</ADD_CONTENT_HERE>
- 
- 

#### 3.7.4 Use Case Recommendation

- <ADD_CONTENT_HERE>When and why to use this format (✓ Use when, ❌ Avoid when)</ADD_CONTENT_HERE>
- 
- 

#### 3.7.5 Trade-offs

- <ADD_CONTENT_HERE>Discuss accuracy vs token cost trade-offs specific to this format</ADD_CONTENT_HERE>
- 
- 

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