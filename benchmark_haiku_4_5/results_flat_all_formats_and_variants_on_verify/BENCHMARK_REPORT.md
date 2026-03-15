# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-15
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Extended Thinking**: on
- **Data Structure**: flat
- **Formats Tested**: 8 (CSV, JSON_COMPACT, JSON_PRETTY, TOON_SAFE, TOON_UNSAFE, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: Second iteration

## Executive Summary

This benchmark evaluates token efficiency and information accuracy across 8 file formats using Claude Haiku 4.5 (claude-haiku-4-5-20251001) as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context, while a format that accurately conveys information may justify higher token cost.

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
- 8 formats tested: CSV, JSON_COMPACT, JSON_PRETTY, TOON_SAFE, TOON_UNSAFE, XML_COMPACT, XML_PRETTY, YAML
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
   - These represent more the "intellectual" aspect of the model and will differ greatly depending on the model. Also if done deterministic the model still needs to do field retrival and structure awarness on the result.

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
   - Optional: YAML ↓ -0.22 % ↑ 0.43 %
   - Mandatory: XML_PRETTY ↓ -0.22 % ↑ 0.11 %
- Highest accuracy:
   - Optional: TOON_UNSAFE 66.94 %
   - Mandatory: TOON_UNSAFE 72.31 %
- Lowest accuracy drift:
   - Optional: CSV ↓ -0.50 % ↑ 0.99 %
   - Mandatory: JSON_PRETTY ↓ -1.15 % ↑ 1.17 %
- Most useful tokens:
   - Optional: XML_PRETTY 10191 / 15285 tokens
   - Mandatory: XML_PRETTY 11490 / 16441 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 69.53
   - Mandatory: TOON_UNSAFE 79.51
- Lowest delta (optional-mandatory):
   - Total tokens: CSV -184 tokens
   - Accuracy: JSON_COMPACT 0.26 %
   - Token efficiency: JSON_PRETTY 0.45

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 15285 tokens
   - Mandatory: XML_PRETTY 16441 tokens
- Highest output token drift:
   - Optional: JSON_COMPACT ↓ -95.68 % ↑ 49.76 %
   - Mandatory: CSV ↓ -93.63 % ↑ 48.04 %
- Lowest accuracy:
   - Optional: CSV 54.30 %
   - Mandatory: CSV 63.17 %
- Highest accuracy drift:
   - Optional: JSON_COMPACT ↓ -6.17 % ↑ 6.17 %
   - Mandatory: TOON_SAFE ↓ -13.44 % ↑ 12.65 %
- Most wasted tokens:
   - Optional: XML_PRETTY 5095 / 15285 tokens
   - Mandatory: XML_PRETTY 4950 / 16441 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 50.36
   - Mandatory: XML_PRETTY 48.95
- Highest delta (optional-mandatory):
   - Total tokens: TOON_UNSAFE 4528 tokens
   - Accuracy: CSV -8.87 %
   - Token efficiency: TOON_UNSAFE -18.10

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| toon_safe ≈ 84831 s | csv ≈ 7172  | toon_unsafe ≈ 2029  | toon_unsafe ≈ 72 % | toon_unsafe ≈ 73 % | toon_unsafe ≈ 80  | toon_unsafe ≈ 80  |
| xml_pretty ( +0.2 %) | toon_unsafe ( +2.2 %) | toon_safe ( +15.7 %) | yaml (-1.6 %) | yaml (-2.3 %) | toon_safe (-3.8 %) | toon_safe (-4.1 %) |
| yaml ( +8.7 %) | toon_safe ( +2.3 %) | csv ( +30.2 %) | xml_pretty (-2.4 %) | xml_pretty (-3.2 %) | csv (-7.4 %) | csv (-6.9 %) |
| json_compact ( +11.1 %) | json_compact ( +32.0 %) | json_compact ( +63.1 %) | json_pretty (-3.0 %) | json_pretty (-3.7 %) | json_compact (-14.9 %) | json_compact (-14.9 %) |
| json_pretty ( +27.4 %) | xml_compact ( +65.7 %) | yaml ( +85.4 %) | toon_safe (-4.3 %) | toon_safe (-4.6 %) | xml_compact (-22.6 %) | xml_compact (-22.8 %) |
| xml_compact ( +30.9 %) | yaml ( +79.0 %) | xml_compact ( +92.1 %) | xml_compact (-5.1 %) | xml_compact (-5.4 %) | yaml (-23.4 %) | yaml (-23.9 %) |
| toon_unsafe ( +36.6 %) | json_pretty ( +103.0 %) | json_pretty ( +120.0 %) | json_compact (-7.3 %) | json_compact (-7.3 %) | json_pretty (-31.4 %) | json_pretty (-31.9 %) |
| csv ( +49.6 %) | xml_pretty ( +129.2 %) | xml_pretty ( +144.0 %) | csv (-9.1 %) | csv (-8.6 %) | xml_pretty (-38.4 %) | xml_pretty (-38.9 %) |


##### Optional

| ↑ Total Duration) | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| yaml ≈ 69375 s | csv ≈ 6988  | json_compact ≈ 3099  | toon_unsafe ≈ 67 % | toon_unsafe ≈ 69 % | json_compact ≈ 70  | json_compact ≈ 71  |
| json_compact ( +9.9 %) | json_compact ( +27.9 %) | csv ( +3.1 %) | xml_pretty (-0.3 %) | xml_pretty (-0.4 %) | csv (-2.2 %) | csv (-2.8 %) |
| csv ( +10.6 %) | xml_compact ( +60.5 %) | toon_unsafe ( +26.5 %) | json_pretty (-1.1 %) | json_pretty (-0.8 %) | toon_unsafe (-11.7 %) | toon_unsafe (-11.2 %) |
| json_pretty ( +24.2 %) | toon_safe ( +69.6 %) | xml_compact ( +33.3 %) | json_compact (-1.6 %) | json_compact (-1.9 %) | xml_compact (-12.6 %) | xml_compact (-12.9 %) |
| xml_pretty ( +26.5 %) | toon_unsafe ( +69.6 %) | toon_safe ( +33.6 %) | toon_safe (-1.9 %) | toon_safe (-3.2 %) | toon_safe (-13.5 %) | toon_safe (-14.3 %) |
| toon_safe ( +30.8 %) | yaml ( +72.4 %) | yaml ( +45.3 %) | xml_compact (-3.8 %) | xml_compact (-4.6 %) | yaml (-16.9 %) | yaml (-17.1 %) |
| xml_compact ( +33.1 %) | json_pretty ( +95.3 %) | json_pretty ( +50.4 %) | yaml (-4.3 %) | yaml (-5.1 %) | json_pretty (-20.9 %) | json_pretty (-20.0 %) |
| toon_unsafe ( +42.5 %) | xml_pretty ( +118.7 %) | xml_pretty ( +64.4 %) | csv (-12.6 %) | csv (-13.6 %) | xml_pretty (-27.6 %) | xml_pretty (-26.9 %) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval % | ↓ Structure Awareness % | ↓ Filtering % | ↓ Aggregation % |
|---|---|---|---|
| json_compact ≈ 75 % | csv ≈ 80 % | toon_unsafe ≈ 71 % | xml_pretty ≈ 71 % |
| json_pretty (0.0 %) | toon_unsafe (-2.5 %) | yaml (-3.2 %) | toon_unsafe (-3.2 %) |
| xml_compact (0.0 %) | xml_pretty (-7.4 %) | json_pretty (-4.8 %) | yaml (-6.3 %) |
| yaml (-0.6 %) | toon_safe (-8.6 %) | toon_safe (-6.4 %) | csv (-11.1 %) |
| toon_unsafe (-3.0 %) | yaml (-9.9 %) | xml_pretty (-7.9 %) | json_pretty (-11.1 %) |
| xml_pretty (-4.2 %) | xml_compact (-11.1 %) | xml_compact (-7.9 %) | toon_safe (-11.1 %) |
| toon_safe (-4.2 %) | json_compact (-12.3 %) | json_compact (-9.5 %) | xml_compact (-22.2 %) |
| csv (-14.5 %) | json_pretty (-12.3 %) | csv (-19.0 %) | json_compact (-31.7 %) |


##### Optional

| ↓ Field Retrieval % | ↓ Structure Awareness % | ↓ Filtering % | ↓ Aggregation % |
|---|---|---|---|
| toon_safe ≈ 67 % | xml_pretty ≈ 84 % | toon_unsafe ≈ 70 % | toon_safe ≈ 54 % |
| toon_unsafe (0.0 %) | json_pretty (-0.0 %) | json_pretty (-4.8 %) | json_compact (-4.8 %) |
| xml_compact (0.0 %) | json_compact (-3.7 %) | toon_safe (-4.8 %) | xml_pretty (-6.3 %) |
| xml_pretty (0.0 %) | toon_unsafe (-3.7 %) | xml_compact (-4.8 %) | yaml (-7.9 %) |
| json_compact (-1.8 %) | yaml (-13.6 %) | yaml (-6.4 %) | json_pretty (-7.9 %) |
| yaml (-2.4 %) | toon_safe (-14.8 %) | json_compact (-7.9 %) | toon_unsafe (-7.9 %) |
| json_pretty (-2.4 %) | xml_compact (-14.8 %) | xml_pretty (-7.9 %) | xml_compact (-11.1 %) |
| csv (-9.1 %) | csv (-24.7 %) | csv (-12.7 %) | csv (-19.0 %) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Tokens/Char | Info/Token | Token/Answer | Acc (%) | Wtd Acc (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6968 | 204 | 7172 | 1.449 | 0.881 | 1.645 | 63.17 | 74.44 | 4530.552 | 2641.448 | 73.61 | 74.44 |
| CSV | opt | 6687 | 301 | 6988 | 1.432 | 0.777 | 2.427 | 54.30 | 68.73 | 3794.484 | 3193.516 | 67.98 | 68.73 |
| JSON_COMPACT | man | 9255 | 213 | 9468 | 2.152 | 0.687 | 1.715 | 65.06 | 68.05 | 6159.664 | 3308.003 | 67.66 | 68.05 |
| JSON_COMPACT | opt | 8727 | 208 | 8935 | 2.118 | 0.731 | 1.680 | 65.32 | 70.70 | 5836.560 | 3098.773 | 69.53 | 70.70 |
| JSON_PRETTY | man | 14254 | 308 | 14562 | 1.697 | 0.476 | 2.484 | 69.35 | 54.41 | 10098.747 | 4463.253 | 54.53 | 54.41 |
| JSON_PRETTY | opt | 13346 | 303 | 13649 | 1.683 | 0.483 | 2.446 | 65.86 | 56.56 | 8989.451 | 4659.882 | 54.97 | 56.56 |
| TOON_SAFE | man | 7029 | 309 | 7338 | 1.446 | 0.927 | 2.495 | 68.01 | 76.70 | 4990.800 | 2347.533 | 76.47 | 76.70 |
| TOON_SAFE | opt | 11540 | 309 | 11849 | 1.701 | 0.549 | 2.495 | 65.05 | 60.56 | 7707.991 | 4141.342 | 60.11 | 60.56 |
| TOON_UNSAFE | man | 7018 | 309 | 7327 | 1.448 | 0.987 | 2.488 | 72.31 | 79.94 | 5297.792 | 2028.708 | 79.51 | 79.94 |
| TOON_UNSAFE | opt | 11540 | 314 | 11854 | 1.701 | 0.565 | 2.532 | 66.94 | 62.82 | 7935.068 | 3918.932 | 61.42 | 62.82 |
| XML_COMPACT | man | 11672 | 209 | 11881 | 2.370 | 0.566 | 1.683 | 67.20 | 61.72 | 7983.808 | 3896.859 | 61.51 | 61.72 |
| XML_COMPACT | opt | 10909 | 309 | 11218 | 2.345 | 0.563 | 2.489 | 63.17 | 61.59 | 7086.200 | 4131.467 | 60.79 | 61.59 |
| XML_PRETTY | man | 16137 | 304 | 16441 | 1.937 | 0.425 | 2.449 | 69.89 | 48.86 | 11490.382 | 4950.285 | 48.95 | 48.86 |
| XML_PRETTY | opt | 15076 | 209 | 15285 | 1.918 | 0.436 | 1.688 | 66.67 | 51.69 | 10190.732 | 5094.601 | 50.36 | 51.69 |
| YAML | man | 12533 | 304 | 12837 | 1.666 | 0.551 | 2.449 | 70.70 | 60.87 | 9075.524 | 3761.143 | 60.94 | 60.87 |
| YAML | opt | 11742 | 308 | 12050 | 1.653 | 0.520 | 2.481 | 62.63 | 58.62 | 7546.706 | 4502.961 | 57.78 | 58.62 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Acc Man (%) | Acc Opt (%) | Diff (%) | Wtd Acc Man (%) | Wtd Acc Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7172 | 6988 | -184 | -2.57 | 63.17 | 54.30 | -8.87 | 64.36 | 55.37 | -8.99 | 73.61 | 67.98 | -5.63 | 74.44 | 68.73 | -5.71 |
| JSON_COMPACT | 9468 | 8936 | -532 | -5.62 | 65.06 | 65.32 |  +0.26 | 65.62 | 67.00 |  +1.38 | 67.66 | 69.53 |  +1.87 | 68.05 | 70.70 |  +2.65 |
| JSON_PRETTY | 14562 | 13649 | -913 | -6.27 | 69.35 | 65.86 | -3.49 | 69.18 | 68.12 | -1.06 | 54.53 | 54.97 |  +0.45 | 54.41 | 56.56 |  +2.15 |
| TOON_SAFE | 7338 | 11849 |  +4511 |  +61.47 | 68.01 | 65.05 | -2.96 | 68.34 | 65.70 | -2.64 | 76.47 | 60.11 | -16.36 | 76.70 | 60.56 | -16.14 |
| TOON_UNSAFE | 7327 | 11855 |  +4528 |  +61.80 | 72.31 | 66.94 | -5.37 | 72.92 | 68.94 | -3.98 | 79.51 | 61.42 | -18.10 | 79.94 | 62.82 | -17.12 |
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
| TOON_SAFE | man | 18 | 390.500 | 0.58 | 84813 | 0.004 | 683.98 | 84831 | 390.504 | 547.30 |
| TOON_SAFE | opt | 21 | 549.524 | 0.68 | 90754 | 0.003 | 731.89 | 90775 | 549.527 | 585.65 |
| TOON_UNSAFE | man | 26 | 269.923 | 0.84 | 115858 | 0.003 | 934.33 | 115884 | 269.926 | 747.64 |
| TOON_UNSAFE | opt | 20 | 577.000 | 0.65 | 98823 | 0.003 | 796.96 | 98843 | 577.003 | 637.70 |
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
| TOON_SAFE | 18 | 21 |  +3 |  +16.67 | 84.81 | 90.75 |  +5.94 |  +7.01 | 84.83 | 90.78 |  +5.94 |  +7.01 |
| TOON_UNSAFE | 26 | 20 | -6 | -23.08 | 115.86 | 98.82 | -17.03 | -14.70 | 115.88 | 98.84 | -17.04 | -14.70 |
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
| TOON_SAFE | man | 1.446 | 10.306 | 226.742 | 0.927 |
| TOON_SAFE | opt | 1.701 | 18.288 | 372.258 | 0.549 |
| TOON_UNSAFE | man | 1.448 | 10.290 | 226.387 | 0.987 |
| TOON_UNSAFE | opt | 1.701 | 18.288 | 372.258 | 0.565 |
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
| TOON_SAFE | 1.446 | 1.701 |  +0.255 |  +17.63 | 10.306 | 18.288 |  +7.982 |  +77.45 | 226.742 | 372.258 |  +145.516 |  +64.18 | 0.927 | 0.549 | -0.378 | -40.78 |
| TOON_UNSAFE | 1.448 | 1.701 |  +0.253 |  +17.47 | 10.290 | 18.288 |  +7.998 |  +77.73 | 226.387 | 372.258 |  +145.871 |  +64.43 | 0.987 | 0.565 | -0.422 | -42.76 |
| XML_COMPACT | 2.370 | 2.345 | -0.025 | -1.05 | 17.114 | 17.288 |  +0.174 |  +1.02 | 376.516 | 351.903 | -24.613 | -6.54 | 0.566 | 0.563 | -0.003 | -0.53 |
| XML_PRETTY | 1.937 | 1.918 | -0.019 | -0.98 | 23.661 | 23.892 |  +0.231 |  +0.98 | 520.548 | 486.323 | -34.225 | -6.57 | 0.425 | 0.436 |  +0.011 |  +2.59 |
| YAML | 1.666 | 1.653 | -0.013 | -0.78 | 18.377 | 18.609 |  +0.232 |  +1.26 | 404.290 | 378.774 | -25.516 | -6.31 | 0.551 | 0.520 | -0.031 | -5.63 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Acc (%) | Wtd Acc (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| CSV | man | 7172 | 4531 | 2641 | 63.17 | 64.36 | 73.61 | 74.44 |
| CSV | opt | 6988 | 3794 | 3194 | 54.30 | 55.37 | 67.98 | 68.73 |
| JSON_COMPACT | man | 9468 | 6160 | 3308 | 65.06 | 65.62 | 67.66 | 68.05 |
| JSON_COMPACT | opt | 8935 | 5837 | 3099 | 65.32 | 67.00 | 69.53 | 70.70 |
| JSON_PRETTY | man | 14562 | 10099 | 4463 | 69.35 | 69.18 | 54.53 | 54.41 |
| JSON_PRETTY | opt | 13649 | 8989 | 4660 | 65.86 | 68.12 | 54.97 | 56.56 |
| TOON_SAFE | man | 7338 | 4991 | 2348 | 68.01 | 68.34 | 76.47 | 76.70 |
| TOON_SAFE | opt | 11849 | 7708 | 4141 | 65.05 | 65.70 | 60.11 | 60.56 |
| TOON_UNSAFE | man | 7327 | 5298 | 2029 | 72.31 | 72.92 | 79.51 | 79.94 |
| TOON_UNSAFE | opt | 11854 | 7935 | 3919 | 66.94 | 68.94 | 61.42 | 62.82 |
| XML_COMPACT | man | 11881 | 7984 | 3897 | 67.20 | 67.49 | 61.51 | 61.72 |
| XML_COMPACT | opt | 11218 | 7086 | 4131 | 63.17 | 64.31 | 60.79 | 61.59 |
| XML_PRETTY | man | 16441 | 11490 | 4950 | 69.89 | 69.76 | 48.95 | 48.86 |
| XML_PRETTY | opt | 15285 | 10191 | 5095 | 66.67 | 68.57 | 50.36 | 51.69 |
| YAML | man | 12837 | 9076 | 3761 | 70.70 | 70.60 | 60.94 | 60.87 |
| YAML | opt | 12050 | 7547 | 4503 | 62.63 | 63.83 | 57.78 | 58.62 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Acc (%) Man | Acc (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7172 | 6988 | -184 | -2.57 | 4531 | 3795 | -736 | -16.25 | 2641 | 3193 |  +552 |  +20.90 | 63.17 | 54.30 | -8.87 | 73.61 | 67.978 | -5.63 | -7.64 |
| JSON_COMPACT | 9468 | 8936 | -532 | -5.62 | 6160 | 5837 | -323 | -5.25 | 3308 | 3099 | -209 | -6.32 | 65.06 | 65.32 |  +0.26 | 67.66 | 69.525 |  +1.87 |  +2.76 |
| JSON_PRETTY | 14562 | 13649 | -913 | -6.27 | 10099 | 8990 | -1109 | -10.98 | 4463 | 4660 |  +197 |  +4.41 | 69.35 | 65.86 | -3.49 | 54.53 | 54.974 |  +0.45 |  +0.82 |
| TOON_SAFE | 7338 | 11849 |  +4511 |  +61.47 | 4991 | 7708 |  +2717 |  +54.44 | 2348 | 4142 |  +1794 |  +76.40 | 68.01 | 65.05 | -2.96 | 76.47 | 60.107 | -16.36 | -21.39 |
| TOON_UNSAFE | 7327 | 11855 |  +4528 |  +61.79 | 5298 | 7935 |  +2637 |  +49.78 | 2029 | 3919 |  +1890 |  +93.16 | 72.31 | 66.94 | -5.37 | 79.51 | 61.416 | -18.10 | -22.76 |
| XML_COMPACT | 11881 | 11218 | -663 | -5.58 | 7984 | 7086 | -898 | -11.24 | 3897 | 4132 |  +235 |  +6.02 | 67.20 | 63.17 | -4.03 | 61.51 | 60.792 | -0.72 | -1.17 |
| XML_PRETTY | 16441 | 15286 | -1155 | -7.03 | 11490 | 10190 | -1300 | -11.31 | 4950 | 5094 |  +144 |  +2.92 | 69.89 | 66.67 | -3.22 | 48.95 | 50.36 |  +1.41 |  +2.87 |
| YAML | 12837 | 12050 | -787 | -6.13 | 9076 | 7547 | -1529 | -16.84 | 3761 | 4503 |  +742 |  +19.72 | 70.70 | 62.63 | -8.07 | 60.94 | 57.779 | -3.16 | -5.18 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Acc (%) |
|---|---|---|---|---|---|
| CSV | man | 78 | 46 | 0 | 63.17 |
| CSV | opt | 67 | 57 | 0 | 54.30 |
| JSON_COMPACT | man | 81 | 43 | 0 | 65.06 |
| JSON_COMPACT | opt | 81 | 43 | 0 | 65.32 |
| JSON_PRETTY | man | 86 | 38 | 0 | 69.35 |
| JSON_PRETTY | opt | 82 | 42 | 0 | 65.86 |
| TOON_SAFE | man | 84 | 40 | 0 | 68.01 |
| TOON_SAFE | opt | 81 | 43 | 0 | 65.05 |
| TOON_UNSAFE | man | 90 | 34 | 0 | 72.31 |
| TOON_UNSAFE | opt | 83 | 41 | 0 | 66.94 |
| XML_COMPACT | man | 83 | 41 | 0 | 67.20 |
| XML_COMPACT | opt | 78 | 46 | 0 | 63.17 |
| XML_PRETTY | man | 87 | 37 | 0 | 69.89 |
| XML_PRETTY | opt | 83 | 41 | 0 | 66.67 |
| YAML | man | 88 | 36 | 0 | 70.70 |
| YAML | opt | 78 | 46 | 0 | 62.63 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Acc (%) Man | Acc (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 78 | 67 | -11 | -14.10 | 46 | 57 |  +11 |  +23.91 | 0 | 0 | 0 | 0.00 | 63.17 | 54.30 | -8.87 |
| JSON_COMPACT | 81 | 81 | 0 | 0.00 | 43 | 43 | 0 | 0.00 | 0 | 0 | 0 | 0.00 | 65.06 | 65.32 |  +0.26 |
| JSON_PRETTY | 86 | 82 | -4 | -4.65 | 38 | 42 |  +4 |  +10.53 | 0 | 0 | 0 | 0.00 | 69.35 | 65.86 | -3.49 |
| TOON_SAFE | 84 | 81 | -3 | -3.57 | 40 | 43 |  +3 |  +7.50 | 0 | 0 | 0 | 0.00 | 68.01 | 65.05 | -2.96 |
| TOON_UNSAFE | 90 | 83 | -7 | -7.78 | 34 | 41 |  +7 |  +20.59 | 0 | 0 | 0 | 0.00 | 72.31 | 66.94 | -5.37 |
| XML_COMPACT | 83 | 78 | -5 | -6.02 | 41 | 46 |  +5 |  +12.20 | 0 | 0 | 0 | 0.00 | 67.20 | 63.17 | -4.03 |
| XML_PRETTY | 87 | 83 | -4 | -4.60 | 37 | 41 |  +4 |  +10.81 | 0 | 0 | 0 | 0.00 | 69.89 | 66.67 | -3.22 |
| YAML | 88 | 78 | -10 | -11.36 | 36 | 46 |  +10 |  +27.78 | 0 | 0 | 0 | 0.00 | 70.70 | 62.63 | -8.07 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Acc (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| CSV | man | 63.17 | 60.00 | 80.25 | 52.38 | 60.32 |
| CSV | opt | 54.30 | 58.18 | 59.26 | 57.14 | 34.92 |
| JSON_COMPACT | man | 65.06 | 74.55 | 67.90 | 61.90 | 39.68 |
| JSON_COMPACT | opt | 65.32 | 65.45 | 80.25 | 61.90 | 49.21 |
| JSON_PRETTY | man | 69.35 | 74.55 | 67.90 | 66.66 | 60.32 |
| JSON_PRETTY | opt | 65.86 | 64.85 | 83.95 | 65.08 | 46.03 |
| TOON_SAFE | man | 68.01 | 70.30 | 71.61 | 65.08 | 60.32 |
| TOON_SAFE | opt | 65.05 | 67.27 | 69.14 | 65.08 | 53.97 |
| TOON_UNSAFE | man | 72.31 | 71.52 | 77.78 | 71.43 | 68.25 |
| TOON_UNSAFE | opt | 66.94 | 67.27 | 80.25 | 69.84 | 46.03 |
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
| TOON_SAFE | 70.30 | 67.27 | -3.03 |
| TOON_UNSAFE | 71.52 | 67.27 | -4.25 |
| XML_COMPACT | 74.55 | 67.27 | -7.28 |
| XML_PRETTY | 70.30 | 67.27 | -3.03 |
| YAML | 73.94 | 64.85 | -9.09 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 80.25 | 59.26 | -20.99 |
| JSON_COMPACT | 67.90 | 80.25 |  +12.35 |
| JSON_PRETTY | 67.90 | 83.95 |  +16.05 |
| TOON_SAFE | 71.61 | 69.14 | -2.47 |
| TOON_UNSAFE | 77.78 | 80.25 |  +2.47 |
| XML_COMPACT | 69.14 | 69.14 | 0.00 |
| XML_PRETTY | 72.84 | 83.95 |  +11.11 |
| YAML | 70.37 | 70.37 | 0.00 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 52.38 | 57.14 |  +4.76 |
| JSON_COMPACT | 61.90 | 61.90 |  +0.00 |
| JSON_PRETTY | 66.66 | 65.08 | -1.58 |
| TOON_SAFE | 65.08 | 65.08 | -0.00 |
| TOON_UNSAFE | 71.43 | 69.84 | -1.59 |
| XML_COMPACT | 63.49 | 65.08 |  +1.59 |
| XML_PRETTY | 63.49 | 61.90 | -1.59 |
| YAML | 68.25 | 63.49 | -4.76 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 60.32 | 34.92 | -25.40 |
| JSON_COMPACT | 39.68 | 49.21 |  +9.52 |
| JSON_PRETTY | 60.32 | 46.03 | -14.29 |
| TOON_SAFE | 60.32 | 53.97 | -6.35 |
| TOON_UNSAFE | 68.25 | 46.03 | -22.22 |
| XML_COMPACT | 49.21 | 42.86 | -6.35 |
| XML_PRETTY | 71.43 | 47.62 | -23.81 |
| YAML | 65.08 | 46.03 | -19.04 |

## 3. Format-Specific Analysis
### 3.1 Detailed Analysis: CSV

#### 3.1.1 Performance Summary

- Token Duration Range: 77 - 127 seconds
- Token Cost Range: 6988 - 7172 tokens
- Wasted Token Range: 2641 - 3194 tokens
- Accuracy Range: 54.30 - 63.17 %
- Efficiency Score Range: 67.98 - 73.61

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
- Wasted Token Range: 3099 - 3308 tokens
- Accuracy Range: 65.06 - 65.32 %
- Efficiency Score Range: 67.66 - 69.53

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
- Wasted Token Range: 4463 - 4660 tokens
- Accuracy Range: 65.86 - 69.35 %
- Efficiency Score Range: 54.53 - 54.97

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

### 3.4 Detailed Analysis: TOON_SAFE

#### 3.4.1 Performance Summary

- Token Duration Range: 85 - 91 seconds
- Token Cost Range: 7338 - 11849 tokens
- Wasted Token Range: 2348 - 4141 tokens
- Accuracy Range: 65.05 - 68.01 %
- Efficiency Score Range: 60.11 - 76.47

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

### 3.5 Detailed Analysis: TOON_UNSAFE

#### 3.5.1 Performance Summary

- Token Duration Range: 99 - 116 seconds
- Token Cost Range: 7327 - 11854 tokens
- Wasted Token Range: 2029 - 3919 tokens
- Accuracy Range: 66.94 - 72.31 %
- Efficiency Score Range: 61.42 - 79.51

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

### 3.6 Detailed Analysis: XML_COMPACT

#### 3.6.1 Performance Summary

- Token Duration Range: 92 - 111 seconds
- Token Cost Range: 11218 - 11881 tokens
- Wasted Token Range: 3897 - 4131 tokens
- Accuracy Range: 63.17 - 67.20 %
- Efficiency Score Range: 60.79 - 61.51

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

### 3.7 Detailed Analysis: XML_PRETTY

#### 3.7.1 Performance Summary

- Token Duration Range: 85 - 88 seconds
- Token Cost Range: 15285 - 16441 tokens
- Wasted Token Range: 4950 - 5095 tokens
- Accuracy Range: 66.67 - 69.89 %
- Efficiency Score Range: 48.95 - 50.36

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

### 3.8 Detailed Analysis: YAML

#### 3.8.1 Performance Summary

- Token Duration Range: 69 - 92 seconds
- Token Cost Range: 12050 - 12837 tokens
- Wasted Token Range: 3761 - 4503 tokens
- Accuracy Range: 62.63 - 70.70 %
- Efficiency Score Range: 57.78 - 60.94

#### 3.8.2 Strengths

- <ADD_CONTENT_HERE>List format strengths based on category and variant analysis</ADD_CONTENT_HERE>
- 
- 

#### 3.8.3 Weaknesses

- <ADD_CONTENT_HERE>List format weaknesses and failure modes</ADD_CONTENT_HERE>
- 
- 

#### 3.8.4 Use Case Recommendation

- <ADD_CONTENT_HERE>When and why to use this format (✓ Use when, ❌ Avoid when)</ADD_CONTENT_HERE>
- 
- 

#### 3.8.5 Trade-offs

- <ADD_CONTENT_HERE>Discuss accuracy vs token cost trade-offs specific to this format</ADD_CONTENT_HERE>
- 
- 

## 4. Conclusions & Recommendations

### 4.1 Format Selection Framework

| Scenario | Recommended Format | Alternative | Avoid |
|----------|------------------|------------|-------|
| <ADD_CONTENT_HERE>Scenario 1</ADD_CONTENT_HERE> | <FORMAT> | <FORMAT> | <FORMAT> |
| <ADD_CONTENT_HERE>Scenario 2</ADD_CONTENT_HERE> | <FORMAT> | <FORMAT> | <FORMAT> |
| <ADD_CONTENT_HERE>Scenario 3</ADD_CONTENT_HERE> | <FORMAT> | <FORMAT> | <FORMAT> |
| <ADD_CONTENT_HERE>Scenario 4</ADD_CONTENT_HERE> | <FORMAT> | <FORMAT> | <FORMAT> |
| <ADD_CONTENT_HERE>Scenario 5</ADD_CONTENT_HERE> | <FORMAT> | <FORMAT> | <FORMAT> |

### 4.2 Token Efficiency vs Accuracy Trade-off
<ADD_CONTENT_HERE>Discuss the fundamental trade-off between token cost and accuracy</ADD_CONTENT_HERE>
- Cheapest format (tokens):
- Most accurate format:
- Best efficiency score:
- Recommendation for different budgets:

### 4.3 Scaling Characteristics
<ADD_CONTENT_HERE>Analyze how formats scale with record count and data complexity</ADD_CONTENT_HERE>
- Linear scaling validation:
- Fixed overhead (per-format):
- Recommendations for large datasets:

### 4.4 Open Research Questions
<ADD_CONTENT_HERE>List questions for future iterations</ADD_CONTENT_HERE>
1. Questions 1
2. Questions 2
3. Questions 3
4. Questions 4
5. Questions 5

## 5. Appendices

### 5.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-15
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Extended Thinking**: on
- **Structure**: flat
- **Formats Tested**: csv, json_compact, json_pretty, toon_safe, toon_unsafe, xml_compact, xml_pretty, yaml
- **Record Counts**: 31
- **Total Test Cases**: 16

### 5.2 Appendix B: Benchmark Configuration
- **Field Retrieval**: 55 questions (37.50% weight)
- **Filtering**: 21 questions (20.83% weight)
- **Aggregation**: 21 questions (12.50% weight)
- **Structure Awareness**: 27 questions (29.17% weight)

---

- **Report Generated**: 2026-03-15
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **With the help of**: Claude Sonnet 4.6
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Related Benchmark Results**: [Report1](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results), [Report2](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results), [Report3](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)