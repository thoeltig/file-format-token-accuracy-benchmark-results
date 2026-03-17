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
   - Optional: JSON_COMPACT 9950 tokens
   - Mandatory: JSON_COMPACT 10468 tokens
- Lowest output token cost drift:
   - Optional: XML_COMPACT ↓ -0.33 % ↑ 0.33 %
   - Mandatory: XML_COMPACT ↓ -0.33 % ↑ 0.33 %
- Highest accuracy:
   - Optional: TOON_DEFAULT 64.98 %
   - Mandatory: XML_COMPACT 67.74 %
- Lowest accuracy drift:
   - Optional: YAML ↓ 0.00 % ↑ 0.00 %
   - Mandatory: JSON_PRETTY ↓ -1.98 % ↑ 2.98 %
- Most useful tokens:
   - Optional: XML_PRETTY 12075 / 19876 tokens
   - Mandatory: XML_PRETTY 12679 / 20506 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 75.02
   - Mandatory: JSON_COMPACT 74.57
- Lowest delta (optional-mandatory):
   - Total tokens: YAML 83 tokens
   - Accuracy: XML_PRETTY -1.08 %
   - Token efficiency: TOON_DEFAULT -0.13

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 19876 tokens
   - Mandatory: XML_PRETTY 20506 tokens
- Highest output token drift:
   - Optional: XML_PRETTY ↓ -95.61 % ↑ 50.24 %
   - Mandatory: YAML ↓ -93.89 % ↑ 49.06 %
- Lowest accuracy:
   - Optional: YAML 58.87 %
   - Mandatory: XML_PRETTY 61.83 %
- Highest accuracy drift:
   - Optional: JSON_PRETTY ↓ -11.46 % ↑ 12.34 %
   - Mandatory: XML_COMPACT ↓ -9.52 % ↑ 10.72 %
- Most wasted tokens:
   - Optional: XML_PRETTY 7801 / 19876 tokens
   - Mandatory: XML_PRETTY 7827 / 20506 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 44.34
   - Mandatory: XML_PRETTY 43.31
- Highest delta (optional-mandatory):
   - Total tokens: JSON_PRETTY -927 tokens
   - Accuracy: YAML -7.26 %
   - Token efficiency: YAML -5.32

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| toon_default ≈ 40225 s | json_compact ≈ 10468  | json_compact ≈ 3579  | xml_compact ≈ 68 % | xml_compact ≈ 67 % | json_compact ≈ 75  | json_compact ≈ 74  |
| json_pretty ( +91.4 %) | xml_compact ( +24.3 %) | xml_compact ( +17.3 %) | toon_default (-1.4 %) | yaml (-0.4 %) | xml_compact (-7.9 %) | xml_compact (-7.7 %) |
| xml_compact ( +98.2 %) | yaml ( +37.3 %) | yaml ( +36.0 %) | yaml (-1.6 %) | toon_default (-1.0 %) | yaml (-14.5 %) | yaml (-13.2 %) |
| xml_pretty ( +102.4 %) | toon_default ( +38.4 %) | toon_default ( +36.3 %) | json_compact (-1.9 %) | json_pretty (-1.6 %) | toon_default (-14.8 %) | toon_default (-14.3 %) |
| json_compact ( +103.3 %) | json_pretty ( +71.8 %) | json_pretty ( +75.9 %) | json_pretty (-2.7 %) | json_compact (-2.2 %) | json_pretty (-29.4 %) | json_pretty (-28.3 %) |
| yaml ( +113.3 %) | xml_pretty ( +95.9 %) | xml_pretty ( +118.7 %) | xml_pretty (-5.9 %) | xml_pretty (-5.4 %) | xml_pretty (-41.9 %) | xml_pretty (-41.6 %) |


##### Optional

| ↑ Total Duration) | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| toon_default ≈ 53833 s | json_compact ≈ 9950  | json_compact ≈ 3547  | toon_default ≈ 65 % | toon_default ≈ 67 % | json_compact ≈ 75  | json_compact ≈ 75  |
| yaml ( +24.9 %) | xml_compact ( +28.3 %) | xml_compact ( +39.3 %) | json_compact (-0.6 %) | json_compact (-2.0 %) | xml_compact (-13.5 %) | xml_compact (-12.3 %) |
| xml_pretty ( +27.7 %) | toon_default ( +42.8 %) | toon_default ( +40.3 %) | xml_compact (-3.7 %) | xml_compact (-3.8 %) | toon_default (-15.5 %) | toon_default (-14.2 %) |
| xml_compact ( +41.8 %) | yaml ( +45.2 %) | yaml ( +67.6 %) | json_pretty (-4.0 %) | json_pretty (-4.9 %) | yaml (-22.1 %) | yaml (-21.6 %) |
| json_pretty ( +75.3 %) | json_pretty ( +71.5 %) | json_pretty ( +87.5 %) | xml_pretty (-4.2 %) | xml_pretty (-5.0 %) | json_pretty (-30.0 %) | json_pretty (-29.5 %) |
| json_compact ( +75.9 %) | xml_pretty ( +99.8 %) | xml_pretty ( +119.9 %) | yaml (-6.1 %) | yaml (-7.0 %) | xml_pretty (-40.9 %) | xml_pretty (-40.2 %) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval % | ↓ Structure Awareness % | ↓ Filtering % | ↓ Aggregation % |
|---|---|---|---|
| yaml ≈ 74 % | yaml ≈ 70 % | yaml ≈ 62 % | xml_compact ≈ 68 % |
| json_compact (-0.5 %) | json_pretty (-0.7 %) | json_pretty (-1.9 %) | json_compact (-8.3 %) |
| toon_default (-1.9 %) | xml_compact (-1.2 %) | json_compact (-3.8 %) | toon_default (-10.5 %) |
| xml_compact (-2.4 %) | toon_default (-2.3 %) | toon_default (-4.2 %) | xml_pretty (-19.0 %) |
| json_pretty (-3.0 %) | xml_pretty (-7.4 %) | xml_compact (-6.3 %) | json_pretty (-19.7 %) |
| xml_pretty (-4.8 %) | json_compact (-9.6 %) | xml_pretty (-7.9 %) | yaml (-23.8 %) |


##### Optional

| ↓ Field Retrieval % | ↓ Structure Awareness % | ↓ Filtering % | ↓ Aggregation % |
|---|---|---|---|
| json_compact ≈ 67 % | toon_default ≈ 78 % | json_pretty ≈ 68 % | json_compact ≈ 53 % |
| json_pretty (-1.2 %) | xml_compact (-4.9 %) | toon_default (-1.6 %) | toon_default (-8.4 %) |
| toon_default (-1.6 %) | json_compact (-10.4 %) | json_compact (-4.4 %) | xml_pretty (-12.1 %) |
| xml_pretty (-2.4 %) | xml_pretty (-11.1 %) | xml_pretty (-6.3 %) | yaml (-13.6 %) |
| xml_compact (-3.0 %) | yaml (-14.8 %) | yaml (-6.3 %) | json_pretty (-13.7 %) |
| yaml (-4.2 %) | json_pretty (-16.1 %) | xml_compact (-6.4 %) | xml_compact (-15.2 %) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Tokens/Char | Info/Token | Token/Answer | Acc (%) | Wtd Acc (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
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
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Acc Man (%) | Acc Opt (%) | Diff (%) | Wtd Acc Man (%) | Wtd Acc Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
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
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Acc (%) | Wtd Acc (%) | Eff Score | Wtd Eff Score |
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
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Acc (%) Man | Acc (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10468 | 9950 | -518 | -4.95 | 6889 | 6403 | -486 | -7.06 | 3579 | 3547 | -32 | -0.89 | 65.81 | 64.35 | -1.46 | 74.57 | 75.017 |  +0.45 |  +0.60 |
| JSON_PRETTY | 17989 | 17062 | -927 | -5.15 | 11693 | 10412 | -1281 | -10.96 | 6296 | 6651 |  +355 |  +5.63 | 65.00 | 61.02 | -3.98 | 52.67 | 52.512 | -0.16 | -0.30 |
| TOON_DEFAULT | 14492 | 14206 | -286 | -1.98 | 9613 | 9231 | -382 | -3.97 | 4879 | 4974 |  +95 |  +1.95 | 66.33 | 64.98 | -1.35 | 63.52 | 63.3855 | -0.13 | -0.21 |
| XML_COMPACT | 13010 | 12762 | -248 | -1.91 | 8813 | 7822 | -991 | -11.25 | 4197 | 4940 |  +743 |  +17.71 | 67.74 | 61.29 | -6.45 | 68.71 | 64.898 | -3.81 | -5.55 |
| XML_PRETTY | 20506 | 19876 | -630 | -3.07 | 12679 | 12075 | -604 | -4.77 | 7827 | 7801 | -26 | -0.33 | 61.83 | 60.75 | -1.08 | 43.31 | 44.341 |  +1.03 |  +2.38 |
| YAML | 14368 | 14451 |  +83 |  +0.58 | 9501 | 8507 | -994 | -10.46 | 4866 | 5943 |  +1077 |  +22.14 | 66.13 | 58.87 | -7.26 | 63.73 | 58.414 | -5.32 | -8.34 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Acc (%) |
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
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Acc (%) Man | Acc (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 82 | 80 | -2 | -2.44 | 42 | 44 |  +2 |  +4.76 | 0 | 0 | 0 | 0.00 | 65.81 | 64.35 | -1.46 |
| JSON_PRETTY | 81 | 76 | -5 | -6.17 | 43 | 48 |  +5 |  +11.63 | 0 | 0 | 0 | 0.00 | 65.00 | 61.02 | -3.98 |
| TOON_DEFAULT | 82 | 81 | -1 | -1.22 | 42 | 43 |  +1 |  +2.38 | 0 | 0 | 0 | 0.00 | 66.33 | 64.98 | -1.35 |
| XML_COMPACT | 84 | 76 | -8 | -9.52 | 40 | 48 |  +8 |  +20.00 | 0 | 0 | 0 | 0.00 | 67.74 | 61.29 | -6.45 |
| XML_PRETTY | 77 | 75 | -2 | -2.60 | 47 | 49 |  +2 |  +4.26 | 0 | 0 | 0 | 0.00 | 61.83 | 60.75 | -1.08 |
| YAML | 82 | 73 | -9 | -10.98 | 42 | 51 |  +9 |  +21.43 | 0 | 0 | 0 | 0.00 | 66.13 | 58.87 | -7.26 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Acc (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
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
- Accuracy Range: 64.35 - 65.81 %
- Efficiency Score Range: 74.57 - 75.02

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

### 3.2 Detailed Analysis: JSON_PRETTY

#### 3.2.1 Performance Summary

- Token Duration Range: 77 - 94 seconds
- Token Cost Range: 17062 - 17989 tokens
- Wasted Token Range: 6296 - 6651 tokens
- Accuracy Range: 61.02 - 65.00 %
- Efficiency Score Range: 52.51 - 52.67

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

### 3.3 Detailed Analysis: TOON_DEFAULT

#### 3.3.1 Performance Summary

- Token Duration Range: 40 - 54 seconds
- Token Cost Range: 14206 - 14492 tokens
- Wasted Token Range: 4879 - 4975 tokens
- Accuracy Range: 64.98 - 66.33 %
- Efficiency Score Range: 63.39 - 63.52

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

### 3.4 Detailed Analysis: XML_COMPACT

#### 3.4.1 Performance Summary

- Token Duration Range: 76 - 80 seconds
- Token Cost Range: 12762 - 13010 tokens
- Wasted Token Range: 4197 - 4940 tokens
- Accuracy Range: 61.29 - 67.74 %
- Efficiency Score Range: 64.90 - 68.71

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

### 3.5 Detailed Analysis: XML_PRETTY

#### 3.5.1 Performance Summary

- Token Duration Range: 69 - 81 seconds
- Token Cost Range: 19876 - 20506 tokens
- Wasted Token Range: 7801 - 7827 tokens
- Accuracy Range: 60.75 - 61.83 %
- Efficiency Score Range: 43.31 - 44.34

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

### 3.6 Detailed Analysis: YAML

#### 3.6.1 Performance Summary

- Token Duration Range: 67 - 86 seconds
- Token Cost Range: 14368 - 14451 tokens
- Wasted Token Range: 4866 - 5944 tokens
- Accuracy Range: 58.87 - 66.13 %
- Efficiency Score Range: 58.41 - 63.73

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

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-17
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Extended Thinking**: off
- **Structure**: nested
- **Formats Tested**: json_compact, json_pretty, toon_default, xml_compact, xml_pretty, yaml
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
- **Related Benchmark Results**: [Report1](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results), [Report2](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results), [Report3](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)