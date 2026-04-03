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
- **Read Tokens**: For each data file a single read subagent is invoked with the only prompt to read the file at the provided filepath and return "Done" once finished and do nothing more. The token extraction script searches for the read tool result and extracts only the read tokens of it.
- **Output Tokens**: For each data file three "benchmark-full-test" subagent are invoked with data, questions and answers template files and the instructions to read everything and answer all questions in a single write tool use. The token extraction script aggregates all output tokens until and including the write tool result.

## 2. Results

### 2.1 TLDR: Token Efficiency Analysis

*Note: All columns ranked best-to-worst. ↑ = lower value is better (ascending). ↓ = higher value is better (descending).*

#### 2.1.1 Best results

- Lowest total token cost:
   - Optional: JSON_COMPACT 18884 tokens
   - Mandatory: CSV 16625 tokens
- Lowest read token cost:
   - Optional: CSV 6700 tokens
   - Mandatory: CSV 6989 tokens
- Lowest output token cost:
   - Optional: JSON_PRETTY 8509 tokens
   - Mandatory: XML_COMPACT 9555 tokens
- Lowest output token cost drift:
   - Optional: CSV ↓ -3.26% ↑ 2.35%
   - Mandatory: JSON_COMPACT ↓ -9.78% ↑ 7.63%
- Highest accuracy:
   - Optional: XML_PRETTY 80.11%
   - Mandatory: JSON_PRETTY 82.80%
- Lowest accuracy drift:
   - Optional: JSON_PRETTY ↓ -1.76% ↑ 3.52%
   - Mandatory: JSON_COMPACT ↓ -0.36% ↑ 0.73%
- Most useful tokens:
   - Optional: XML_PRETTY 20228 / 25250 tokens
   - Mandatory: JSON_PRETTY 22862 / 27611 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 80.22
   - Mandatory: TOON_DEFAULT 80.21
- Lowest delta (optional-mandatory):
   - Total tokens: YAML -791 tokens
   - Accuracy: TOON_DEFAULT 0.27%
   - Token efficiency: XML_COMPACT -1.71

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 25250 tokens
   - Mandatory: XML_PRETTY 29037 tokens
- Highest read token cost:
   - Optional: XML_PRETTY 15089 tokens
   - Mandatory: XML_PRETTY 16166 tokens
- Highest output token cost:
   - Optional: CSV 12999 tokens
   - Mandatory: JSON_COMPACT 13355 tokens
- Highest output token drift:
   - Optional: JSON_COMPACT ↓ -42.13% ↑ 48.94%
   - Mandatory: CSV ↓ -38.64% ↑ 32.44%
- Lowest accuracy:
   - Optional: CSV 63.18%
   - Mandatory: CSV 63.71%
- Highest accuracy drift:
   - Optional: JSON_COMPACT ↓ -10.81% ↑ 7.43%
   - Mandatory: CSV ↓ -18.99% ↑ 15.19%
- Most wasted tokens:
   - Optional: CSV 7253 / 19699 tokens
   - Mandatory: XML_PRETTY 6400 / 29037 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 65.24
   - Mandatory: XML_PRETTY 54.60
- Highest delta (optional-mandatory):
   - Total tokens: JSON_PRETTY -5735 tokens
   - Accuracy: JSON_PRETTY -6.46%
   - Token efficiency: JSON_COMPACT 12.60

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 40s | CSV ≈ 6989 | XML_COMPACT ≈ 9555 | CSV ≈ 16625 | TOON_DEFAULT ≈ 3860 | JSON_PRETTY ≈ 83% | JSON_PRETTY ≈ 82% | TOON_DEFAULT ≈ 80 | TOON_DEFAULT ≈ 79 |
| XML_COMPACT (+86.2%) | TOON_DEFAULT (+0.8%) | CSV (+0.8%) | TOON_DEFAULT (+13.6%) | JSON_PRETTY (+23.0%) | TOON_DEFAULT (-3.2%) | TOON_DEFAULT (-4.1%) | CSV (-7.0%) | CSV (-6.0%) |
| CSV (+93.5%) | JSON_COMPACT (+32.6%) | YAML (+12.1%) | XML_COMPACT (+27.8%) | XML_COMPACT (+24.3%) | XML_PRETTY (-4.8%) | XML_PRETTY (-4.2%) | XML_COMPACT (-9.0%) | XML_COMPACT (-8.2%) |
| YAML (+118.2%) | XML_COMPACT (+67.3%) | TOON_DEFAULT (+23.9%) | JSON_COMPACT (+36.1%) | YAML (+37.7%) | XML_COMPACT (-5.4%) | XML_COMPACT (-5.2%) | YAML (-15.3%) | YAML (-14.7%) |
| XML_PRETTY (+153.7%) | YAML (+79.6%) | XML_PRETTY (+34.7%) | YAML (+39.9%) | JSON_COMPACT (+49.7%) | YAML (-5.6%) | YAML (-5.6%) | JSON_COMPACT (-15.7%) | JSON_COMPACT (-15.4%) |
| JSON_PRETTY (+162.1%) | JSON_PRETTY (+104.4%) | JSON_PRETTY (+39.5%) | JSON_PRETTY (+66.1%) | CSV (+56.3%) | JSON_COMPACT (-8.3%) | JSON_COMPACT (-8.5%) | JSON_PRETTY (-23.4%) | JSON_PRETTY (-23.0%) |
| JSON_COMPACT (+171.5%) | XML_PRETTY (+131.3%) | JSON_COMPACT (+39.8%) | XML_PRETTY (+74.7%) | XML_PRETTY (+65.8%) | CSV (-19.1%) | CSV (-18.6%) | XML_PRETTY (-31.9%) | XML_PRETTY (-31.1%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 39s | CSV ≈ 6700 | JSON_PRETTY ≈ 8509 | JSON_COMPACT ≈ 18884 | JSON_COMPACT ≈ 3858 | XML_PRETTY ≈ 80% | JSON_COMPACT ≈ 81% | JSON_COMPACT ≈ 80 | JSON_COMPACT ≈ 81 |
| JSON_PRETTY (+88.1%) | JSON_COMPACT (+30.6%) | JSON_COMPACT (+19.1%) | CSV (+4.3%) | XML_COMPACT (+19.6%) | TOON_DEFAULT (-0.3%) | TOON_DEFAULT (-0.3%) | XML_COMPACT (-11.1%) | XML_COMPACT (-12.3%) |
| XML_PRETTY (+103.0%) | XML_COMPACT (+63.1%) | XML_PRETTY (+19.4%) | JSON_PRETTY (+15.8%) | TOON_DEFAULT (+22.7%) | JSON_COMPACT (-0.5%) | XML_COMPACT (-1.5%) | JSON_PRETTY (-11.8%) | YAML (-12.4%) |
| JSON_COMPACT (+112.9%) | TOON_DEFAULT (+72.6%) | YAML (+25.8%) | YAML (+19.0%) | YAML (+26.8%) | XML_COMPACT (-0.5%) | YAML (-2.0%) | YAML (-12.0%) | JSON_PRETTY (-12.9%) |
| YAML (+127.6%) | YAML (+75.7%) | XML_COMPACT (+36.9%) | XML_COMPACT (+19.6%) | XML_PRETTY (+30.2%) | YAML (-1.9%) | XML_PRETTY (-2.2%) | TOON_DEFAULT (-13.6%) | TOON_DEFAULT (-14.0%) |
| XML_COMPACT (+144.7%) | JSON_PRETTY (+99.5%) | TOON_DEFAULT (+40.1%) | TOON_DEFAULT (+24.4%) | JSON_PRETTY (+34.2%) | JSON_PRETTY (-3.8%) | JSON_PRETTY (-4.6%) | CSV (-16.8%) | CSV (-17.2%) |
| CSV (+182.2%) | XML_PRETTY (+125.2%) | CSV (+52.8%) | XML_PRETTY (+33.7%) | CSV (+88.0%) | CSV (-16.9%) | CSV (-17.1%) | XML_PRETTY (-18.7%) | XML_PRETTY (-20.9%) |


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

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Char/Token | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6989 | 9636 | 16625 | 1.444 | 0.383 | 77.712 | 63.71 | 74.30 | 10592.000 | 6033.333 | 74.57 | 74.30 |
| CSV | opt | 6700 | 12999 | 19699 | 1.429 | 0.321 | 104.831 | 63.18 | 66.95 | 12445.828 | 7253.172 | 66.78 | 66.95 |
| JSON_COMPACT | man | 9268 | 13355 | 22623 | 2.149 | 0.329 | 107.702 | 74.46 | 66.88 | 16845.086 | 5777.914 | 67.62 | 66.88 |
| JSON_COMPACT | opt | 8748 | 10136 | 18884 | 2.113 | 0.421 | 81.745 | 79.57 | 80.87 | 15026.264 | 3858.069 | 80.22 | 80.87 |
| JSON_PRETTY | man | 14283 | 13328 | 27611 | 1.694 | 0.300 | 107.484 | 82.80 | 60.82 | 22861.908 | 4749.092 | 61.42 | 60.82 |
| JSON_PRETTY | opt | 13367 | 8509 | 21876 | 1.680 | 0.349 | 68.617 | 76.34 | 70.44 | 16699.757 | 5175.743 | 70.74 | 70.44 |
| TOON_DEFAULT | man | 7048 | 11844 | 18892 | 1.442 | 0.426 | 95.512 | 79.57 | 79.02 | 15031.967 | 3859.533 | 80.21 | 79.02 |
| TOON_DEFAULT | opt | 11561 | 11923 | 23484 | 1.698 | 0.345 | 96.154 | 79.84 | 69.56 | 18749.626 | 4734.374 | 69.31 | 69.56 |
| XML_COMPACT | man | 11693 | 9555 | 21248 | 2.366 | 0.364 | 77.059 | 77.42 | 72.52 | 16450.459 | 4797.874 | 73.01 | 72.52 |
| XML_COMPACT | opt | 10930 | 11652 | 22582 | 2.340 | 0.352 | 93.968 | 79.57 | 70.92 | 17968.497 | 4613.503 | 71.30 | 70.92 |
| XML_PRETTY | man | 16166 | 12871 | 29037 | 1.934 | 0.268 | 103.798 | 77.96 | 54.44 | 22637.245 | 6399.755 | 54.60 | 54.44 |
| XML_PRETTY | opt | 15089 | 10161 | 25250 | 1.917 | 0.317 | 81.946 | 80.11 | 63.96 | 20228.042 | 5022.291 | 65.24 | 63.96 |
| YAML | man | 12554 | 10711 | 23265 | 1.664 | 0.332 | 86.376 | 77.15 | 67.36 | 17948.691 | 5315.976 | 67.96 | 67.36 |
| YAML | opt | 11771 | 10702 | 22473 | 1.649 | 0.348 | 86.309 | 78.23 | 70.85 | 17580.888 | 4892.445 | 70.63 | 70.85 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6989 | 6700 | -289 | -4.14 | 9636 | 12999 |  +3363 |  +34.90 | 16625 | 19699 |  +3074 |  +18.49 | 63.32 | 63.41 |  +0.09 | 74.30 | 66.95 | -7.35 |
| JSON_COMPACT | 9268 | 8748 | -520 | -5.61 | 13355 | 10136 | -3219 | -24.10 | 22623 | 18884 | -3739 | -16.53 | 73.39 | 80.50 |  +7.11 | 66.88 | 80.87 |  +14.00 |
| JSON_PRETTY | 14283 | 13367 | -916 | -6.41 | 13328 | 8509 | -4819 | -36.16 | 27611 | 21876 | -5735 | -20.77 | 81.94 | 75.90 | -6.04 | 60.82 | 70.44 |  +9.61 |
| TOON_DEFAULT | 7048 | 11561 |  +4513 |  +64.03 | 11844 | 11924 |  +80 |  +0.68 | 18892 | 23485 |  +4593 |  +24.31 | 77.87 | 80.19 |  +2.32 | 79.02 | 69.56 | -9.46 |
| XML_COMPACT | 11693 | 10930 | -763 | -6.53 | 9555 | 11652 |  +2097 |  +21.95 | 21248 | 22582 |  +1334 |  +6.28 | 76.71 | 79.02 |  +2.31 | 72.52 | 70.92 | -1.60 |
| XML_PRETTY | 16166 | 15089 | -1077 | -6.66 | 12871 | 10161 | -2710 | -21.06 | 29037 | 25250 | -3787 | -13.04 | 77.74 | 78.28 |  +0.54 | 54.44 | 63.96 |  +9.52 |
| YAML | 12554 | 11771 | -783 | -6.24 | 10711 | 10703 | -8 | -0.07 | 23265 | 22474 | -791 | -3.40 | 76.30 | 78.55 |  +2.25 | 67.36 | 70.85 |  +3.48 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 20 | 349.450 | 0.65 | 78262 | 0.123 | 631.14 | 78282 | 349.573 | 505.04 |
| CSV | opt | 15 | 446.667 | 0.48 | 109800 | 0.118 | 885.48 | 109815 | 446.785 | 708.48 |
| JSON_COMPACT | man | 28 | 331.000 | 0.90 | 109802 | 0.122 | 885.50 | 109830 | 331.122 | 708.58 |
| JSON_COMPACT | opt | 10 | 874.800 | 0.32 | 82813 | 0.122 | 667.85 | 82823 | 874.922 | 534.34 |
| JSON_PRETTY | man | 25 | 571.320 | 0.81 | 106026 | 0.126 | 855.05 | 106051 | 571.446 | 684.20 |
| JSON_PRETTY | opt | 13 | 1028.231 | 0.42 | 73162 | 0.116 | 590.01 | 73175 | 1028.347 | 472.09 |
| TOON_DEFAULT | man | 3 | 2349.333 | 0.10 | 95966 | 0.123 | 773.92 | 95969 | 2349.456 | 619.16 |
| TOON_DEFAULT | opt | 5 | 2312.200 | 0.16 | 94335 | 0.125 | 760.77 | 94340 | 2312.325 | 608.65 |
| XML_COMPACT | man | 4 | 2923.250 | 0.13 | 75334 | 0.127 | 607.53 | 75338 | 2923.377 | 486.05 |
| XML_COMPACT | opt | 9 | 1214.444 | 0.29 | 95202 | 0.122 | 767.76 | 95211 | 1214.566 | 614.26 |
| XML_PRETTY | man | 11 | 1469.636 | 0.35 | 102619 | 0.125 | 827.57 | 102630 | 1469.761 | 662.13 |
| XML_PRETTY | opt | 17 | 887.588 | 0.55 | 78965 | 0.129 | 636.81 | 78982 | 887.717 | 509.56 |
| YAML | man | 8 | 1569.250 | 0.26 | 88278 | 0.121 | 711.92 | 88286 | 1569.371 | 569.59 |
| YAML | opt | 11 | 1070.091 | 0.35 | 88549 | 0.121 | 714.10 | 88560 | 1070.212 | 571.35 |

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
| CSV | man | 1.444 | 10.248 | 225.452 | 0.383 |
| CSV | opt | 1.429 | 10.618 | 216.129 | 0.321 |
| JSON_COMPACT | man | 2.149 | 13.589 | 298.968 | 0.329 |
| JSON_COMPACT | opt | 2.113 | 13.864 | 282.194 | 0.421 |
| JSON_PRETTY | man | 1.694 | 20.943 | 460.742 | 0.300 |
| JSON_PRETTY | opt | 1.680 | 21.184 | 431.194 | 0.349 |
| TOON_DEFAULT | man | 1.442 | 10.334 | 227.355 | 0.426 |
| TOON_DEFAULT | opt | 1.698 | 18.322 | 372.935 | 0.345 |
| XML_COMPACT | man | 2.366 | 17.145 | 377.194 | 0.364 |
| XML_COMPACT | opt | 2.340 | 17.322 | 352.581 | 0.352 |
| XML_PRETTY | man | 1.934 | 23.704 | 521.484 | 0.268 |
| XML_PRETTY | opt | 1.917 | 23.913 | 486.742 | 0.317 |
| YAML | man | 1.664 | 18.408 | 404.968 | 0.332 |
| YAML | opt | 1.649 | 18.655 | 379.710 | 0.348 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.444 | 1.429 | -0.015 | -1.04 | 10.248 | 10.618 |  +0.370 |  +3.61 | 225.452 | 216.129 | -9.323 | -4.14 | 0.383 | 0.321 | -0.062 | -16.19 |
| JSON_COMPACT | 2.149 | 2.113 | -0.036 | -1.68 | 13.589 | 13.864 |  +0.275 |  +2.02 | 298.968 | 282.194 | -16.774 | -5.61 | 0.329 | 0.421 |  +0.092 |  +27.96 |
| JSON_PRETTY | 1.694 | 1.680 | -0.014 | -0.83 | 20.943 | 21.184 |  +0.241 |  +1.15 | 460.742 | 431.194 | -29.548 | -6.41 | 0.300 | 0.349 |  +0.049 |  +16.33 |
| TOON_DEFAULT | 1.442 | 1.698 |  +0.256 |  +17.75 | 10.334 | 18.322 |  +7.988 |  +77.30 | 227.355 | 372.935 |  +145.580 |  +64.03 | 0.426 | 0.345 | -0.081 | -19.13 |
| XML_COMPACT | 2.366 | 2.340 | -0.026 | -1.10 | 17.145 | 17.322 |  +0.177 |  +1.03 | 377.194 | 352.581 | -24.613 | -6.53 | 0.364 | 0.352 | -0.012 | -3.30 |
| XML_PRETTY | 1.934 | 1.917 | -0.017 | -0.88 | 23.704 | 23.913 |  +0.209 |  +0.88 | 521.484 | 486.742 | -34.742 | -6.66 | 0.268 | 0.317 |  +0.049 |  +18.28 |
| YAML | 1.664 | 1.649 | -0.015 | -0.90 | 18.408 | 18.655 |  +0.247 |  +1.34 | 404.968 | 379.710 | -25.258 | -6.24 | 0.332 | 0.348 |  +0.016 |  +4.82 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| CSV | man | 16625 | 10592 | 6033 | 63.71 | 63.32 | 74.57 | 74.30 |
| CSV | opt | 19699 | 12446 | 7253 | 63.18 | 63.41 | 66.78 | 66.95 |
| JSON_COMPACT | man | 22623 | 16845 | 5778 | 74.46 | 73.39 | 67.62 | 66.88 |
| JSON_COMPACT | opt | 18884 | 15026 | 3858 | 79.57 | 80.50 | 80.22 | 80.87 |
| JSON_PRETTY | man | 27611 | 22862 | 4749 | 82.80 | 81.94 | 61.42 | 60.82 |
| JSON_PRETTY | opt | 21876 | 16700 | 5176 | 76.34 | 75.90 | 70.74 | 70.44 |
| TOON_DEFAULT | man | 18892 | 15032 | 3860 | 79.57 | 77.87 | 80.21 | 79.02 |
| TOON_DEFAULT | opt | 23484 | 18750 | 4734 | 79.84 | 80.19 | 69.31 | 69.56 |
| XML_COMPACT | man | 21248 | 16450 | 4798 | 77.42 | 76.71 | 73.01 | 72.52 |
| XML_COMPACT | opt | 22582 | 17968 | 4614 | 79.57 | 79.02 | 71.30 | 70.92 |
| XML_PRETTY | man | 29037 | 22637 | 6400 | 77.96 | 77.74 | 54.60 | 54.44 |
| XML_PRETTY | opt | 25250 | 20228 | 5022 | 80.11 | 78.28 | 65.24 | 63.96 |
| YAML | man | 23265 | 17949 | 5316 | 77.15 | 76.30 | 67.96 | 67.36 |
| YAML | opt | 22473 | 17581 | 4892 | 78.23 | 78.55 | 70.63 | 70.85 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 16625 | 19699 |  +3074 |  +18.49 | 10592 | 12446 |  +1854 |  +17.50 | 6033 | 7253 |  +1220 |  +20.22 | 63.71 | 63.18 | -0.53 | 74.57 | 66.785 | -7.79 | -10.44 |
| JSON_COMPACT | 22623 | 18884 | -3739 | -16.53 | 16845 | 15026 | -1819 | -10.80 | 5778 | 3858 | -1920 | -33.23 | 74.46 | 79.57 |  +5.11 | 67.62 | 80.223 |  +12.60 |  +18.63 |
| JSON_PRETTY | 27611 | 21876 | -5736 | -20.77 | 22862 | 16700 | -6162 | -26.95 | 4749 | 5176 |  +427 |  +8.98 | 82.80 | 76.34 | -6.46 | 61.42 | 70.744 |  +9.32 |  +15.17 |
| TOON_DEFAULT | 18892 | 23485 |  +4593 |  +24.31 | 15032 | 18750 |  +3718 |  +24.73 | 3860 | 4735 |  +875 |  +22.66 | 79.57 | 79.84 |  +0.27 | 80.21 | 69.3125 | -10.89 | -13.58 |
| XML_COMPACT | 21248 | 22582 |  +1334 |  +6.28 | 16450 | 17968 |  +1518 |  +9.23 | 4798 | 4614 | -184 | -3.84 | 77.42 | 79.57 |  +2.15 | 73.01 | 71.3 | -1.71 | -2.35 |
| XML_PRETTY | 29037 | 25250 | -3787 | -13.04 | 22637 | 20228 | -2409 | -10.64 | 6400 | 5023 | -1377 | -21.52 | 77.96 | 80.11 |  +2.15 | 54.60 | 65.239 |  +10.64 |  +19.49 |
| YAML | 23265 | 22474 | -791 | -3.40 | 17949 | 17581 | -368 | -2.05 | 5316 | 4892 | -424 | -7.97 | 77.15 | 78.23 |  +1.08 | 67.96 | 70.625 |  +2.67 |  +3.92 |

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

- **Report Generated**: 2026-04-03
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code < 2.1.86
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_flat_all_formats_and_variants_off\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_off\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_on_verify\BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)