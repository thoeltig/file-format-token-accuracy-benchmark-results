# File Format Token Efficiency Benchmark: Comprehensive Report
**Date**: 2026-03-13
- **Model**: Claude Haiku 4.5
- **Extended Thinking**: off
- **Data Structure**: flat
- **Formats Tested**: 8 (CSV, JSON_COMPACT, JSON_PRETTY, TOON_SAFE, TOON_UNSAFE, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: First iteration

---

## Executive Summary
This benchmark evaluates token efficiency and information accuracy across 8 file formats using Claude Haiku 4.5 as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context, while a format that accurately conveys information may justify higher token cost.

### Key Findings

<ADD_CONTENT_HERE: Insert 5-7 key findings from analysis>

1. Finding 1

2. Finding 2

3. Finding 3

4. Finding 4

5. Finding 5

---

## 1. Methodology

### 1.1 Research Purpose

The underlying question: **Which file format delivers maximum information value per token consumed?**

This requires measuring:
- **Token Cost**: How many tokens does each format consume for equivalent data?
- **Information Fidelity**: How accurately can the model understand and answer questions about the data?
- **Robustness**: How consistent is performance across data variants (mandatory vs optional fields)?

### 1.2 Test Design

**Data Generation:**
- 8 formats tested: CSV, JSON_COMPACT, JSON_PRETTY, TOON_SAFE, TOON_UNSAFE, XML_COMPACT, XML_PRETTY, YAML
- 2 variants per format: mandatory (22 fields, dense) and optional (19 mandatory + 3 optional, sparse)
- Record Counts: 31

**Question Distribution:**
- 4 question categories reflecting practical use cases:
   - **Field Retrieval (60 questions, 37.50% weight):** Extract specific values from specific records
   - **Filtering (21 questions, 20.83% weight):** Count records matching criteria
   - **Aggregation (21 questions, 12.50% weight):** Sum, average, min/max calculations
   - **Structure Awareness (27 questions, 29.17% weight):** Understand data shape, organization, metadata

**Weighting Rationale:**
- Field retrieval + structure awareness = 66.67%
   - These represent the file format itself. Understanding "what data exists and how it's organized" which is fundamental to avoiding context confusion.
- Filtering + aggregation = 33.33%
   - These represent more the "intellactual" aspect of the model and will differ greatly depending on the model. Also if done deterministic the model still needs to do field retrival and structure awarness on the result.

### 1.3 Metrics Definition

**Token Metrics:**
- `readTokens`: Tokens consumed reading the data file
- `outputTokens`: Tokens consumed during inference (answering questions + creating the file content)
- `totalTokens`: readTokens + outputTokens

**Accuracy Metrics:**
- `rawAccuracy`: Correct answers / total questions
- `weightedAccuracy`: Accuracy weighted by question category importanc

**Information Value Metrics:**
- `informationValuePerToken`: (accuracy% / totalTokens) × 100
- `costOfInaccuracy`: totalTokens × (1 - accuracy% / 100) — tokens wasted on inaccurate output

**Efficiency Score:**
- Composite metric balancing accuracy with normalized token cost (favour towards accuracy)
- normalizedTokenCost = (((maxTotalTokens+10)-currenTotalTokens)/((maxTotalTokens+10)-(minTotalTokens-10)))*100
- `efficiencyScore`: (accuracy% x 0.3) + (normalizedTokenCost * 0.3)
- `weightedEfficiencyScore`: (weightedAccuracy% x 0.3) + (normalizedTokenCost * 0.3)

---

## 2. Results

### 2.1 TLDR: Token Efficiency Analysis

#### 2.1.1 Best results

- Lowest total token cost:
   - Optional: CSV 6931 tokens
   - Mandatory: TOON_UNSAFE 7279 tokens
- Lowest output token cost drift:
   - Optional: JSON_PRETTY ↓ -0.60 % ↑ 0.60 %
   - Mandatory: CSV ↓ -0.10 % ↑ 0.20 %
- Highest accuracy:
   - Optional: TOON_SAFE 68.82 %
   - Mandatory: JSON_PRETTY 71.51 %
- Lowest accuracy drift:
   - Optional: TOON_UNSAFE ↓ -2.03 % ↑ 2.85 %
   - Mandatory: JSON_PRETTY ↓ -0.76 % ↑ 1.50 %
- Most useful tokens:
   - Optional: XML_PRETTY 10173 / 15321 tokens
   - Mandatory: XML_PRETTY 11051 / 16509 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 70.84
   - Mandatory: TOON_SAFE 77.83
- Lowest delta (optional-mandatory):
   - Total tokens: XML_PRETTY -1187 tokens
   - Accuracy: XML_PRETTY -0.54 %
   - Token efficiency: CSV -0.48

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 15321 tokens
   - Mandatory: XML_PRETTY 16509 tokens
- Highest output token drift:
   - Optional: XML_PRETTY ↓ -94.40 % ↑ 49.79 %
   - Mandatory: JSON_COMPACT ↓ -96.05 % ↑ 48.90 %
- Lowest accuracy:
   - Optional: CSV 54.84 %
   - Mandatory: CSV 57.26 %
- Highest accuracy drift:
   - Optional: XML_COMPACT ↓ -7.79 % ↑ 6.97 %
   - Mandatory: CSV ↓ -14.09 % ↑ 12.68 %
- Most wasted tokens:
   - Optional: XML_PRETTY 5148 / 15321 tokens
   - Mandatory: XML_PRETTY 5458 / 16509 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 50.22
   - Mandatory: XML_PRETTY 46.89
- Highest delta (optional-mandatory):
   - Total tokens: TOON_UNSAFE 4624 tokens
   - Accuracy: JSON_PRETTY -8.07 %
   - Token efficiency: TOON_UNSAFE -16.34

#### 2.1.3 Conclussion

<ADD_CONTENT_HERE>Analyze token usage patterns here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Tokens/Char | Info/Token | Token/Answer | Raw Acc (%) | Wtd Acc (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6989 | 329 | 7318 | 1.444 | 0.782 | 2.656 | 57.26 | 69.26 | 4190.477 | 3127.856 | 68.84 | 69.26 |
| CSV | opt | 6700 | 231 | 6931 | 1.429 | 0.791 | 1.863 | 54.84 | 69.43 | 3800.960 | 3130.040 | 68.36 | 69.43 |
| JSON_COMPACT | man | 9268 | 228 | 9496 | 2.149 | 0.677 | 1.836 | 64.25 | 67.28 | 6100.966 | 3394.701 | 66.93 | 67.28 |
| JSON_COMPACT | opt | 8748 | 338 | 9086 | 2.113 | 0.749 | 2.723 | 68.01 | 72.74 | 6179.162 | 2906.505 | 70.84 | 72.74 |
| JSON_PRETTY | man | 14283 | 341 | 14624 | 1.694 | 0.489 | 2.750 | 71.51 | 56.59 | 10457.622 | 4166.378 | 55.98 | 56.59 |
| JSON_PRETTY | opt | 13367 | 336 | 13703 | 1.680 | 0.463 | 2.710 | 63.44 | 54.30 | 8693.183 | 5009.817 | 53.21 | 54.30 |
| TOON_SAFE | man | 7048 | 343 | 7391 | 1.442 | 0.953 | 2.769 | 70.43 | 77.78 | 5205.716 | 2185.617 | 77.83 | 77.78 |
| TOON_SAFE | opt | 11570 | 339 | 11909 | 1.697 | 0.578 | 2.731 | 68.82 | 63.88 | 8195.545 | 3713.122 | 62.58 | 63.88 |
| TOON_UNSAFE | man | 7048 | 231 | 7279 | 1.442 | 0.942 | 1.860 | 68.55 | 76.71 | 4989.526 | 2289.141 | 76.87 | 76.71 |
| TOON_UNSAFE | opt | 11561 | 342 | 11903 | 1.698 | 0.553 | 2.755 | 65.86 | 62.44 | 7839.096 | 4063.571 | 60.53 | 62.44 |
| XML_COMPACT | man | 11693 | 343 | 12036 | 2.366 | 0.558 | 2.763 | 67.20 | 61.66 | 8087.968 | 3947.699 | 61.05 | 61.66 |
| XML_COMPACT | opt | 10930 | 335 | 11265 | 2.340 | 0.582 | 2.704 | 65.59 | 63.46 | 7388.932 | 3876.401 | 62.33 | 63.46 |
| XML_PRETTY | man | 16166 | 343 | 16509 | 1.934 | 0.405 | 2.763 | 66.94 | 47.92 | 11050.902 | 5457.765 | 46.89 | 47.92 |
| XML_PRETTY | opt | 15089 | 232 | 15321 | 1.917 | 0.433 | 1.874 | 66.40 | 50.43 | 10173.365 | 5147.968 | 50.22 | 50.43 |
| YAML | man | 12554 | 231 | 12785 | 1.664 | 0.517 | 1.863 | 66.13 | 58.56 | 8454.720 | 4330.279 | 57.96 | 58.56 |
| YAML | opt | 11771 | 333 | 12104 | 1.649 | 0.535 | 2.688 | 64.78 | 60.83 | 7841.187 | 4263.146 | 59.14 | 60.83 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Acc Man (%) | Acc Opt (%) | Diff (%) | Wtd Acc Man (%) | Wtd Acc Opt (%) | Diff (%) | Eff Man (%) | Eff Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7318 | 6931 | -387 | -5.29 | 57.26 | 54.84 | -2.42 | 57.86 | 56.37 | -1.49 | 68.84 | 68.36 | -0.48 | 69.26 | 69.43 | 0.17 |
| JSON_COMPACT | 9496 | 9086 | -410 | -4.32 | 64.25 | 68.01 | 3.76 | 64.75 | 70.72 | 5.97 | 66.93 | 70.84 | 3.91 | 67.28 | 72.74 | 5.46 |
| JSON_PRETTY | 14624 | 13703 | -921 | -6.30 | 71.51 | 63.44 | -8.07 | 72.39 | 65.00 | -7.39 | 55.98 | 53.21 | -2.77 | 56.59 | 54.30 | -2.29 |
| TOON_SAFE | 7391 | 11908 | 4517 | 61.11 | 70.43 | 68.82 | -1.61 | 70.36 | 70.67 | 0.31 | 77.83 | 62.58 | -15.25 | 77.78 | 63.88 | -13.90 |
| TOON_UNSAFE | 7279 | 11903 | 4624 | 63.53 | 68.55 | 65.86 | -2.69 | 68.32 | 68.58 | 0.26 | 76.87 | 60.53 | -16.34 | 76.71 | 62.44 | -14.27 |
| XML_COMPACT | 12036 | 11266 | -770 | -6.40 | 67.20 | 65.59 | -1.61 | 68.07 | 67.20 | -0.87 | 61.05 | 62.33 | 1.28 | 61.66 | 63.46 | 1.80 |
| XML_PRETTY | 16509 | 15322 | -1187 | -7.19 | 66.94 | 66.40 | -0.54 | 68.42 | 66.69 | -1.73 | 46.89 | 50.22 | 3.33 | 47.92 | 50.43 | 2.50 |
| YAML | 12785 | 12104 | -681 | -5.33 | 66.13 | 64.78 | -1.35 | 66.98 | 67.19 | 0.21 | 57.96 | 59.14 | 1.18 | 58.56 | 60.83 | 2.27 |

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
| TOON_SAFE | man | 4 | 1762.000 | 0.13 | 111019 | 0.003 | 895.31 | 111023 | 1762.003 | 716.28 |
| TOON_SAFE | opt | 8 | 1446.250 | 0.26 | 77811 | 0.004 | 627.51 | 77819 | 1446.254 | 502.06 |
| TOON_UNSAFE | man | 3 | 2349.333 | 0.10 | 80914 | 0.003 | 652.53 | 80917 | 2349.336 | 522.04 |
| TOON_UNSAFE | opt | 5 | 2312.200 | 0.16 | 110859 | 0.003 | 894.03 | 110864 | 2312.203 | 715.25 |
| XML_COMPACT | man | 4 | 2923.250 | 0.13 | 75334 | 0.005 | 607.53 | 75338 | 2923.255 | 486.05 |
| XML_COMPACT | opt | 9 | 1214.444 | 0.29 | 95202 | 0.004 | 767.76 | 95211 | 1214.448 | 614.26 |
| XML_PRETTY | man | 11 | 1469.636 | 0.35 | 102619 | 0.003 | 827.57 | 102630 | 1469.639 | 662.13 |
| XML_PRETTY | opt | 17 | 887.588 | 0.55 | 78965 | 0.003 | 636.81 | 78982 | 887.591 | 509.56 |
| YAML | man | 8 | 1569.250 | 0.26 | 88278 | 0.003 | 711.92 | 88286 | 1569.253 | 569.59 |
| YAML | opt | 11 | 1070.091 | 0.35 | 88549 | 0.004 | 714.10 | 88560 | 1070.095 | 571.35 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) | Total Man (s) | Total Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 20 | 15 | -5 | -25.00 | 78.26 | 109.80 | 31.54 | 40.30 | 78.28 | 109.81 | 31.53 | 40.28 |
| JSON_COMPACT | 28 | 10 | -18 | -64.29 | 109.80 | 82.81 | -26.99 | -24.58 | 109.83 | 82.82 | -27.01 | -24.59 |
| JSON_PRETTY | 25 | 13 | -12 | -48.00 | 106.03 | 73.16 | -32.86 | -31.00 | 106.05 | 73.17 | -32.88 | -31.00 |
| TOON_SAFE | 4 | 8 | 4 | 100.00 | 111.02 | 77.81 | -33.21 | -29.91 | 111.02 | 77.82 | -33.20 | -29.91 |
| TOON_UNSAFE | 3 | 5 | 2 | 66.67 | 80.91 | 110.86 | 29.95 | 37.01 | 80.92 | 110.86 | 29.95 | 37.01 |
| XML_COMPACT | 4 | 9 | 5 | 125.00 | 75.33 | 95.20 | 19.87 | 26.37 | 75.34 | 95.21 | 19.87 | 26.38 |
| XML_PRETTY | 11 | 17 | 6 | 54.55 | 102.62 | 78.97 | -23.65 | -23.05 | 102.63 | 78.98 | -23.65 | -23.04 |
| YAML | 8 | 11 | 3 | 37.50 | 88.28 | 88.55 | 0.27 | 0.31 | 88.29 | 88.56 | 0.27 | 0.31 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Char/Token | Token/Value | Token/Object | Info/Token |
|---|---|---|---|---|---|
| CSV | man | 1.444 | 10.248 | 225.452 | 0.782 |
| CSV | opt | 1.429 | 10.618 | 216.129 | 0.791 |
| JSON_COMPACT | man | 2.149 | 13.589 | 298.968 | 0.677 |
| JSON_COMPACT | opt | 2.113 | 13.864 | 282.194 | 0.749 |
| JSON_PRETTY | man | 1.694 | 20.943 | 460.742 | 0.489 |
| JSON_PRETTY | opt | 1.680 | 21.184 | 431.194 | 0.463 |
| TOON_SAFE | man | 1.442 | 10.334 | 227.355 | 0.953 |
| TOON_SAFE | opt | 1.697 | 18.336 | 373.226 | 0.578 |
| TOON_UNSAFE | man | 1.442 | 10.334 | 227.355 | 0.942 |
| TOON_UNSAFE | opt | 1.698 | 18.322 | 372.935 | 0.553 |
| XML_COMPACT | man | 2.366 | 17.145 | 377.194 | 0.558 |
| XML_COMPACT | opt | 2.340 | 17.322 | 352.581 | 0.582 |
| XML_PRETTY | man | 1.934 | 23.704 | 521.484 | 0.405 |
| XML_PRETTY | opt | 1.917 | 23.913 | 486.742 | 0.433 |
| YAML | man | 1.664 | 18.408 | 404.968 | 0.517 |
| YAML | opt | 1.649 | 18.655 | 379.710 | 0.535 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.444 | 1.429 | -0.015 | -1.04 | 10.248 | 10.618 | 0.370 | 3.61 | 225.452 | 216.129 | -9.323 | -4.14 | 0.782 | 0.791 | 0.009 | 1.15 |
| JSON_COMPACT | 2.149 | 2.113 | -0.036 | -1.68 | 13.589 | 13.864 | 0.275 | 2.02 | 298.968 | 282.194 | -16.774 | -5.61 | 0.677 | 0.749 | 0.072 | 10.64 |
| JSON_PRETTY | 1.694 | 1.680 | -0.014 | -0.83 | 20.943 | 21.184 | 0.241 | 1.15 | 460.742 | 431.194 | -29.548 | -6.41 | 0.489 | 0.463 | -0.026 | -5.32 |
| TOON_SAFE | 1.442 | 1.697 | 0.255 | 17.68 | 10.334 | 18.336 | 8.002 | 77.43 | 227.355 | 373.226 | 145.871 | 64.16 | 0.953 | 0.578 | -0.375 | -39.35 |
| TOON_UNSAFE | 1.442 | 1.698 | 0.256 | 17.75 | 10.334 | 18.322 | 7.988 | 77.30 | 227.355 | 372.935 | 145.580 | 64.03 | 0.942 | 0.553 | -0.389 | -41.30 |
| XML_COMPACT | 2.366 | 2.340 | -0.026 | -1.10 | 17.145 | 17.322 | 0.177 | 1.03 | 377.194 | 352.581 | -24.613 | -6.53 | 0.558 | 0.582 | 0.024 | 4.30 |
| XML_PRETTY | 1.934 | 1.917 | -0.017 | -0.88 | 23.704 | 23.913 | 0.209 | 0.88 | 521.484 | 486.742 | -34.742 | -6.66 | 0.405 | 0.433 | 0.028 | 6.91 |
| YAML | 1.664 | 1.649 | -0.015 | -0.90 | 18.408 | 18.655 | 0.247 | 1.34 | 404.968 | 379.710 | -25.258 | -6.24 | 0.517 | 0.535 | 0.018 | 3.48 |

### 2.6 Answer Quality Breakdown
#### 2.6.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Raw Acc (%) |
|---|---|---|---|---|---|
| CSV | man | 71 | 53 | 0 | 57.26 |
| CSV | opt | 68 | 56 | 0 | 54.84 |
| JSON_COMPACT | man | 80 | 44 | 0 | 64.25 |
| JSON_COMPACT | opt | 84 | 40 | 0 | 68.01 |
| JSON_PRETTY | man | 89 | 35 | 0 | 71.51 |
| JSON_PRETTY | opt | 79 | 45 | 0 | 63.44 |
| TOON_SAFE | man | 87 | 37 | 0 | 70.43 |
| TOON_SAFE | opt | 85 | 39 | 0 | 68.82 |
| TOON_UNSAFE | man | 85 | 39 | 0 | 68.55 |
| TOON_UNSAFE | opt | 82 | 42 | 0 | 65.86 |
| XML_COMPACT | man | 83 | 41 | 0 | 67.20 |
| XML_COMPACT | opt | 81 | 43 | 0 | 65.59 |
| XML_PRETTY | man | 83 | 41 | 0 | 66.94 |
| XML_PRETTY | opt | 82 | 42 | 0 | 66.40 |
| YAML | man | 82 | 42 | 0 | 66.13 |
| YAML | opt | 80 | 44 | 0 | 64.78 |

### 2.7 Token Utilization Efficiency
#### 2.7.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Raw Acc (%) | Wtd Acc (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| CSV | man | 7318 | 4190 | 3128 | 57.26 | 57.86 | 68.84 | 69.26 |
| CSV | opt | 6931 | 3801 | 3130 | 54.84 | 56.37 | 68.36 | 69.43 |
| JSON_COMPACT | man | 9496 | 6101 | 3395 | 64.25 | 64.75 | 66.93 | 67.28 |
| JSON_COMPACT | opt | 9086 | 6179 | 2907 | 68.01 | 70.72 | 70.84 | 72.74 |
| JSON_PRETTY | man | 14624 | 10458 | 4166 | 71.51 | 72.39 | 55.98 | 56.59 |
| JSON_PRETTY | opt | 13703 | 8693 | 5010 | 63.44 | 65.00 | 53.21 | 54.30 |
| TOON_SAFE | man | 7391 | 5206 | 2186 | 70.43 | 70.36 | 77.83 | 77.78 |
| TOON_SAFE | opt | 11909 | 8196 | 3713 | 68.82 | 70.67 | 62.58 | 63.88 |
| TOON_UNSAFE | man | 7279 | 4990 | 2289 | 68.55 | 68.32 | 76.87 | 76.71 |
| TOON_UNSAFE | opt | 11903 | 7839 | 4064 | 65.86 | 68.58 | 60.53 | 62.44 |
| XML_COMPACT | man | 12036 | 8088 | 3948 | 67.20 | 68.07 | 61.05 | 61.66 |
| XML_COMPACT | opt | 11265 | 7389 | 3876 | 65.59 | 67.20 | 62.33 | 63.46 |
| XML_PRETTY | man | 16509 | 11051 | 5458 | 66.94 | 68.42 | 46.89 | 47.92 |
| XML_PRETTY | opt | 15321 | 10173 | 5148 | 66.40 | 66.69 | 50.22 | 50.43 |
| YAML | man | 12785 | 8455 | 4330 | 66.13 | 66.98 | 57.96 | 58.56 |
| YAML | opt | 12104 | 7841 | 4263 | 64.78 | 67.19 | 59.14 | 60.83 |

### 2.8 Category Performance Analysis
#### 2.8.1 Metrics
| Format | Variant | Raw Acc (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| CSV | man | 57.26 | 59.40 | 64.20 | 52.38 | 47.62 |
| CSV | opt | 54.84 | 56.36 | 60.49 | 63.49 | 34.92 |
| JSON_COMPACT | man | 64.25 | 74.55 | 67.90 | 58.73 | 38.10 |
| JSON_COMPACT | opt | 68.01 | 64.85 | 87.66 | 71.43 | 47.62 |
| JSON_PRETTY | man | 71.51 | 73.94 | 81.48 | 65.08 | 58.73 |
| JSON_PRETTY | opt | 63.44 | 66.06 | 75.31 | 61.90 | 42.86 |
| TOON_SAFE | man | 70.43 | 74.55 | 72.84 | 63.49 | 63.49 |
| TOON_SAFE | opt | 68.82 | 64.85 | 82.72 | 71.43 | 58.73 |
| TOON_UNSAFE | man | 68.55 | 69.09 | 67.90 | 66.67 | 69.84 |
| TOON_UNSAFE | opt | 65.86 | 66.66 | 82.72 | 71.43 | 36.51 |
| XML_COMPACT | man | 67.20 | 69.70 | 75.31 | 63.49 | 53.97 |
| XML_COMPACT | opt | 65.59 | 67.27 | 76.55 | 66.67 | 46.03 |
| XML_PRETTY | man | 66.94 | 72.73 | 75.31 | 68.26 | 39.68 |
| XML_PRETTY | opt | 66.40 | 67.27 | 69.14 | 65.08 | 61.90 |
| YAML | man | 66.13 | 73.94 | 65.43 | 73.02 | 39.68 |
| YAML | opt | 64.78 | 67.27 | 79.01 | 69.84 | 34.92 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 59.40 | 56.36 | -3.03 |
| JSON_COMPACT | 74.55 | 64.85 | -9.70 |
| JSON_PRETTY | 73.94 | 66.06 | -7.89 |
| TOON_SAFE | 74.55 | 64.85 | -9.70 |
| TOON_UNSAFE | 69.09 | 66.66 | -2.43 |
| XML_COMPACT | 69.70 | 67.27 | -2.43 |
| XML_PRETTY | 72.73 | 67.27 | -5.46 |
| YAML | 73.94 | 67.27 | -6.67 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 64.20 | 60.49 | -3.70 |
| JSON_COMPACT | 67.90 | 87.66 | 19.76 |
| JSON_PRETTY | 81.48 | 75.31 | -6.17 |
| TOON_SAFE | 72.84 | 82.72 | 9.88 |
| TOON_UNSAFE | 67.90 | 82.72 | 14.81 |
| XML_COMPACT | 75.31 | 76.55 | 1.24 |
| XML_PRETTY | 75.31 | 69.14 | -6.17 |
| YAML | 65.43 | 79.01 | 13.58 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 52.38 | 63.49 | 11.11 |
| JSON_COMPACT | 58.73 | 71.43 | 12.70 |
| JSON_PRETTY | 65.08 | 61.90 | -3.17 |
| TOON_SAFE | 63.49 | 71.43 | 7.94 |
| TOON_UNSAFE | 66.67 | 71.43 | 4.76 |
| XML_COMPACT | 63.49 | 66.67 | 3.18 |
| XML_PRETTY | 68.26 | 65.08 | -3.18 |
| YAML | 73.02 | 69.84 | -3.18 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 47.62 | 34.92 | -12.70 |
| JSON_COMPACT | 38.10 | 47.62 | 9.52 |
| JSON_PRETTY | 58.73 | 42.86 | -15.87 |
| TOON_SAFE | 63.49 | 58.73 | -4.76 |
| TOON_UNSAFE | 69.84 | 36.51 | -33.34 |
| XML_COMPACT | 53.97 | 46.03 | -7.93 |
| XML_PRETTY | 39.68 | 61.90 | 22.22 |
| YAML | 39.68 | 34.92 | -4.76 |



#### Category Difficulty Ranking (31-Record Dataset):

| Rank | Category | Avg Accuracy (%) | Easiest Format | Hardest Format |
|------|----------|--------------|----------------|----------------|
| 1 | structure_awareness | 74.00 | JSON_PRETTY | CSV |
| 2 | field_retrieval | 68.03 | YAML | CSV |
| 3 | filtering | 65.77 | YAML | CSV |
| 4 | aggregation | 48.41 | TOON_SAFE | YAML |

<ADD_CONTENT_HERE: Analyze performance across question categories>
- Field retrieval performance:
- Structure awareness patterns:
- Aggregation/filtering challenges:
- Per-format strengths and weaknesses:

---

## Format-Specific Analysis
### CSV: Detailed Analysis
**Performance Summary:**
- Best Configuration: 31-record mandatory (57.86% weighted accuracy)
- Token Cost Range: 6931 - 7318 tokens
- Average Weighted Accuracy: 57.11%

**Strengths:**
<ADD_CONTENT_HERE: List format strengths based on category and variant analysis>
- 
- 

**Weaknesses:**
<ADD_CONTENT_HERE: List format weaknesses and failure modes>
- 
- 

**Use Case Recommendation:**
<ADD_CONTENT_HERE: When and why to use this format>
- ✓ Use when:
- ❌ Avoid when:

**Trade-offs:**
<ADD_CONTENT_HERE: Discuss accuracy vs token cost trade-offs specific to this format>

---

### JSON_COMPACT: Detailed Analysis
**Performance Summary:**
- Best Configuration: 31-record optional (70.72% weighted accuracy)
- Token Cost Range: 9086 - 9496 tokens
- Average Weighted Accuracy: 67.73%

**Strengths:**
<ADD_CONTENT_HERE: List format strengths based on category and variant analysis>
- 
- 

**Weaknesses:**
<ADD_CONTENT_HERE: List format weaknesses and failure modes>
- 
- 

**Use Case Recommendation:**
<ADD_CONTENT_HERE: When and why to use this format>
- ✓ Use when:
- ❌ Avoid when:

**Trade-offs:**
<ADD_CONTENT_HERE: Discuss accuracy vs token cost trade-offs specific to this format>

---

### JSON_PRETTY: Detailed Analysis
**Performance Summary:**
- Best Configuration: 31-record mandatory (72.39% weighted accuracy)
- Token Cost Range: 13703 - 14624 tokens
- Average Weighted Accuracy: 68.69%

**Strengths:**
<ADD_CONTENT_HERE: List format strengths based on category and variant analysis>
- 
- 

**Weaknesses:**
<ADD_CONTENT_HERE: List format weaknesses and failure modes>
- 
- 

**Use Case Recommendation:**
<ADD_CONTENT_HERE: When and why to use this format>
- ✓ Use when:
- ❌ Avoid when:

**Trade-offs:**
<ADD_CONTENT_HERE: Discuss accuracy vs token cost trade-offs specific to this format>

---

### TOON_SAFE: Detailed Analysis
**Performance Summary:**
- Best Configuration: 31-record optional (70.67% weighted accuracy)
- Token Cost Range: 7391 - 11909 tokens
- Average Weighted Accuracy: 70.52%

**Strengths:**
<ADD_CONTENT_HERE: List format strengths based on category and variant analysis>
- 
- 

**Weaknesses:**
<ADD_CONTENT_HERE: List format weaknesses and failure modes>
- 
- 

**Use Case Recommendation:**
<ADD_CONTENT_HERE: When and why to use this format>
- ✓ Use when:
- ❌ Avoid when:

**Trade-offs:**
<ADD_CONTENT_HERE: Discuss accuracy vs token cost trade-offs specific to this format>

---

### TOON_UNSAFE: Detailed Analysis
**Performance Summary:**
- Best Configuration: 31-record optional (68.58% weighted accuracy)
- Token Cost Range: 7279 - 11903 tokens
- Average Weighted Accuracy: 68.45%

**Strengths:**
<ADD_CONTENT_HERE: List format strengths based on category and variant analysis>
- 
- 

**Weaknesses:**
<ADD_CONTENT_HERE: List format weaknesses and failure modes>
- 
- 

**Use Case Recommendation:**
<ADD_CONTENT_HERE: When and why to use this format>
- ✓ Use when:
- ❌ Avoid when:

**Trade-offs:**
<ADD_CONTENT_HERE: Discuss accuracy vs token cost trade-offs specific to this format>

---

### XML_COMPACT: Detailed Analysis
**Performance Summary:**
- Best Configuration: 31-record mandatory (68.07% weighted accuracy)
- Token Cost Range: 11265 - 12036 tokens
- Average Weighted Accuracy: 67.63%

**Strengths:**
<ADD_CONTENT_HERE: List format strengths based on category and variant analysis>
- 
- 

**Weaknesses:**
<ADD_CONTENT_HERE: List format weaknesses and failure modes>
- 
- 

**Use Case Recommendation:**
<ADD_CONTENT_HERE: When and why to use this format>
- ✓ Use when:
- ❌ Avoid when:

**Trade-offs:**
<ADD_CONTENT_HERE: Discuss accuracy vs token cost trade-offs specific to this format>

---

### XML_PRETTY: Detailed Analysis
**Performance Summary:**
- Best Configuration: 31-record mandatory (68.42% weighted accuracy)
- Token Cost Range: 15321 - 16509 tokens
- Average Weighted Accuracy: 67.56%

**Strengths:**
<ADD_CONTENT_HERE: List format strengths based on category and variant analysis>
- 
- 

**Weaknesses:**
<ADD_CONTENT_HERE: List format weaknesses and failure modes>
- 
- 

**Use Case Recommendation:**
<ADD_CONTENT_HERE: When and why to use this format>
- ✓ Use when:
- ❌ Avoid when:

**Trade-offs:**
<ADD_CONTENT_HERE: Discuss accuracy vs token cost trade-offs specific to this format>

---

### YAML: Detailed Analysis
**Performance Summary:**
- Best Configuration: 31-record optional (67.19% weighted accuracy)
- Token Cost Range: 12104 - 12785 tokens
- Average Weighted Accuracy: 67.09%

**Strengths:**
<ADD_CONTENT_HERE: List format strengths based on category and variant analysis>
- 
- 

**Weaknesses:**
<ADD_CONTENT_HERE: List format weaknesses and failure modes>
- 
- 

**Use Case Recommendation:**
<ADD_CONTENT_HERE: When and why to use this format>
- ✓ Use when:
- ❌ Avoid when:

**Trade-offs:**
<ADD_CONTENT_HERE: Discuss accuracy vs token cost trade-offs specific to this format>

---

## Conclusions & Recommendations
### Format Selection Framework
**Decision Matrix:**

| Scenario | Recommended Format | Alternative | Avoid |
|----------|------------------|------------|-------|
| <ADD_SCENARIO_1> | <FORMAT> | <FORMAT> | <FORMAT> |
| <ADD_SCENARIO_2> | <FORMAT> | <FORMAT> | <FORMAT> |
| <ADD_SCENARIO_3> | <FORMAT> | <FORMAT> | <FORMAT> |
| <ADD_SCENARIO_4> | <FORMAT> | <FORMAT> | <FORMAT> |
| <ADD_SCENARIO_5> | <FORMAT> | <FORMAT> | <FORMAT> |

### Token Efficiency vs Accuracy Trade-off
<ADD_CONTENT_HERE: Discuss the fundamental trade-off between token cost and accuracy>
- Cheapest format (tokens):
- Most accurate format:
- Best efficiency score:
- Recommendation for different budgets:

### Scaling Characteristics
<ADD_CONTENT_HERE: Analyze how formats scale with record count and data complexity>
- Linear scaling validation:
- Fixed overhead (per-format):
- Recommendations for large datasets:

### Open Research Questions
<ADD_CONTENT_HERE: List questions for future iterations>
1. Questions 1
2. Questions 2
3. Questions 3
4. Questions 4
5. Questions 5

---

## Appendices
### Appendix A: Complete Data Tables
#### A.1 Token Cost Breakdown by Format and Variant
| Format | Records | Variant | Read Tokens | Output Tokens | Total Tokens |
|---|---|---|---|---|---|
| CSV | 31 | man | 6989 | 329 | 7318 |
| CSV | 31 | opt | 6700 | 231 | 6931 |
| JSON_COMPACT | 31 | man | 9268 | 228 | 9496 |
| JSON_COMPACT | 31 | opt | 8748 | 338 | 9086 |
| JSON_PRETTY | 31 | man | 14283 | 341 | 14624 |
| JSON_PRETTY | 31 | opt | 13367 | 336 | 13703 |
| TOON_SAFE | 31 | man | 7048 | 343 | 7391 |
| TOON_SAFE | 31 | opt | 11570 | 339 | 11909 |
| TOON_UNSAFE | 31 | man | 7048 | 231 | 7279 |
| TOON_UNSAFE | 31 | opt | 11561 | 342 | 11903 |
| XML_COMPACT | 31 | man | 11693 | 343 | 12036 |
| XML_COMPACT | 31 | opt | 10930 | 335 | 11265 |
| XML_PRETTY | 31 | man | 16166 | 343 | 16509 |
| XML_PRETTY | 31 | opt | 15089 | 232 | 15321 |
| YAML | 31 | man | 12554 | 231 | 12785 |
| YAML | 31 | opt | 11771 | 333 | 12104 |

#### A.2 Accuracy Comparison
| Format | Records | Variant | Raw Accuracy (%) | Weighted Accuracy (%) | Delta (%) |
|---|---|---|---|---|---|
| CSV | 31 | man | 57.26 | 57.86 | 0.60 |
| CSV | 31 | opt | 54.84 | 56.37 | 1.53 |
| JSON_COMPACT | 31 | man | 64.25 | 64.75 | 0.50 |
| JSON_COMPACT | 31 | opt | 68.01 | 70.72 | 2.71 |
| JSON_PRETTY | 31 | man | 71.51 | 72.39 | 0.88 |
| JSON_PRETTY | 31 | opt | 63.44 | 65.00 | 1.56 |
| TOON_SAFE | 31 | man | 70.43 | 70.36 | -0.07 |
| TOON_SAFE | 31 | opt | 68.82 | 70.67 | 1.85 |
| TOON_UNSAFE | 31 | man | 68.55 | 68.32 | -0.23 |
| TOON_UNSAFE | 31 | opt | 65.86 | 68.58 | 2.72 |
| XML_COMPACT | 31 | man | 67.20 | 68.07 | 0.87 |
| XML_COMPACT | 31 | opt | 65.59 | 67.20 | 1.61 |
| XML_PRETTY | 31 | man | 66.94 | 68.42 | 1.48 |
| XML_PRETTY | 31 | opt | 66.40 | 66.69 | 0.29 |
| YAML | 31 | man | 66.13 | 66.98 | 0.85 |
| YAML | 31 | opt | 64.78 | 67.19 | 2.41 |

#### A.3 Efficiency and Cost Analysis
| Format | Records | Variant | Info/Token | Efficiency | Wtd Efficiency | Cost Inaccuracy |
|---|---|---|---|---|---|---|
| CSV | 31 | man | 0.782 | 68.84 | 69.26 | 3127.856 |
| CSV | 31 | opt | 0.791 | 68.36 | 69.43 | 3130.040 |
| JSON_COMPACT | 31 | man | 0.677 | 66.93 | 67.28 | 3394.701 |
| JSON_COMPACT | 31 | opt | 0.749 | 70.84 | 72.74 | 2906.505 |
| JSON_PRETTY | 31 | man | 0.489 | 55.98 | 56.59 | 4166.378 |
| JSON_PRETTY | 31 | opt | 0.463 | 53.21 | 54.30 | 5009.817 |
| TOON_SAFE | 31 | man | 0.953 | 77.83 | 77.78 | 2185.617 |
| TOON_SAFE | 31 | opt | 0.578 | 62.58 | 63.88 | 3713.122 |
| TOON_UNSAFE | 31 | man | 0.942 | 76.87 | 76.71 | 2289.141 |
| TOON_UNSAFE | 31 | opt | 0.553 | 60.53 | 62.44 | 4063.571 |
| XML_COMPACT | 31 | man | 0.558 | 61.05 | 61.66 | 3947.699 |
| XML_COMPACT | 31 | opt | 0.582 | 62.33 | 63.46 | 3876.401 |
| XML_PRETTY | 31 | man | 0.405 | 46.89 | 47.92 | 5457.765 |
| XML_PRETTY | 31 | opt | 0.433 | 50.22 | 50.43 | 5147.968 |
| YAML | 31 | man | 0.517 | 57.96 | 58.56 | 4330.279 |
| YAML | 31 | opt | 0.535 | 59.14 | 60.83 | 4263.146 |

### Appendix B: Detailed Performance Data
#### B.1 Read & Output Performance
| Format | Records | Variant | Output Tokens | Reasoning Duration (ms) | Q Count | Correct | Incorrect | Unanswered |
|---|---|---|---|---|---|---|---|---|
*Note: These detailed metrics are extracted from metrics.json*
- Read duration shows file read performance
- Reasoning duration shows inference/thinking time
- Q Count and answer distribution shows answer quality

#### B.2 Structural Efficiency (Per Value/Object)
| Format | Records | Variant | Tokens/Value | Tokens/Object | Output/Answer |
|---|---|---|---|---|---|
*Note: These metrics show format overhead at different granularities*
- Tokens/Value: Lower = less overhead per data element
- Tokens/Object: Lower = less overhead per record
- Output/Answer: Shows answer conciseness

#### B.3 Token Utilization Details
| Format | Records | Variant | Efficiently Used Tokens | Weighted Utilized | Utilization % |
|---|---|---|---|---|---|
*Note: These metrics break down token usage into utilized vs wasted*
- Efficiently Used: Tokens that contributed to correct answers
- Weighted Utilized: Same but weighted by question importance
- Utilization %: Percentage of tokens producing useful output

### Appendix C: Test Infrastructure
- **Test Date**: 2026-03-13
- **Model**: Claude Haiku 4.5
- **Extended Thinking**: off
- **Structure**: <ADD_STRUCTURE>
- **Formats Tested**: csv, json_compact, json_pretty, toon_safe, toon_unsafe, xml_compact, xml_pretty, yaml
- **Record Counts**: 31
- **Total Test Cases**: 16
### Appendix D: Benchmark Configuration
- **Field Retrieval**: 60 questions (37.50% weight)
- **Filtering**: 21 questions (20.83% weight)
- **Aggregation**: 21 questions (12.50% weight)
- **Structure Awareness**: 27 questions (29.17% weight)

---

- **Report Generated**: 2026-03-13
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **With the help of**: <ADD_MODEL_NAME>
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)