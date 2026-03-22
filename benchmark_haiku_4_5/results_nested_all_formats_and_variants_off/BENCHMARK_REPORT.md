# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: off
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
   - Optional: JSON_COMPACT 78.07%
   - Mandatory: YAML 77.42%
- Lowest accuracy drift:
   - Optional: YAML ↓ -3.19% ↑ 2.79%
   - Mandatory: YAML ↓ -2.08% ↑ 3.13%
- Most useful tokens:
   - Optional: XML_PRETTY 14160 / 19876 tokens
   - Mandatory: XML_PRETTY 14443 / 20506 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 84.62
   - Mandatory: JSON_COMPACT 81.68
- Lowest delta (optional-mandatory):
   - Total tokens: YAML 83 tokens
   - Accuracy: JSON_PRETTY 0.00%
   - Token efficiency: XML_COMPACT -0.42

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 19876 tokens
   - Mandatory: XML_PRETTY 20506 tokens
- Highest output token drift:
   - Optional: XML_PRETTY ↓ -95.61% ↑ 50.24%
   - Mandatory: YAML ↓ -93.89% ↑ 49.06%
- Lowest accuracy:
   - Optional: YAML 67.47%
   - Mandatory: XML_PRETTY 70.43%
- Highest accuracy drift:
   - Optional: XML_PRETTY ↓ -9.43% ↑ 15.47%
   - Mandatory: XML_COMPACT ↓ -11.27% ↑ 11.97%
- Most wasted tokens:
   - Optional: XML_PRETTY 5716 / 19876 tokens
   - Mandatory: XML_PRETTY 6064 / 20506 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 51.68
   - Mandatory: XML_PRETTY 49.33
- Highest delta (optional-mandatory):
   - Total tokens: JSON_PRETTY -927 tokens
   - Accuracy: YAML -9.95%
   - Token efficiency: YAML -7.20

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 40s | JSON_COMPACT ≈ 10468 | JSON_COMPACT ≈ 2516 | YAML ≈ 77% | YAML ≈ 76% | JSON_COMPACT ≈ 82 | JSON_COMPACT ≈ 80 |
| JSON_PRETTY (+91.4%) | XML_COMPACT (+24.3%) | XML_COMPACT (+22.4%) | XML_COMPACT (-1.1%) | XML_COMPACT (-1.9%) | XML_COMPACT (-8.5%) | XML_COMPACT (-8.2%) |
| XML_COMPACT (+98.2%) | YAML (+37.3%) | YAML (+29.0%) | JSON_COMPACT (-1.5%) | TOON_DEFAULT (-2.0%) | YAML (-12.3%) | YAML (-11.4%) |
| XML_PRETTY (+102.4%) | TOON_DEFAULT (+38.4%) | TOON_DEFAULT (+38.8%) | TOON_DEFAULT (-1.5%) | JSON_COMPACT (-2.8%) | TOON_DEFAULT (-14.0%) | TOON_DEFAULT (-13.6%) |
| JSON_COMPACT (+103.3%) | JSON_PRETTY (+71.8%) | JSON_PRETTY (+84.6%) | JSON_PRETTY (-3.2%) | JSON_PRETTY (-3.0%) | JSON_PRETTY (-27.6%) | JSON_PRETTY (-26.9%) |
| YAML (+113.3%) | XML_PRETTY (+95.9%) | XML_PRETTY (+141.1%) | XML_PRETTY (-7.0%) | XML_PRETTY (-7.3%) | XML_PRETTY (-39.6%) | XML_PRETTY (-39.6%) |


##### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 54s | JSON_COMPACT ≈ 9950 | JSON_COMPACT ≈ 2182 | JSON_COMPACT ≈ 78% | TOON_DEFAULT ≈ 77% | JSON_COMPACT ≈ 85 | JSON_COMPACT ≈ 83 |
| YAML (+24.9%) | XML_COMPACT (+28.3%) | TOON_DEFAULT (+47.8%) | TOON_DEFAULT (-0.8%) | JSON_COMPACT (-0.8%) | XML_COMPACT (-12.2%) | XML_COMPACT (-11.3%) |
| XML_PRETTY (+27.7%) | TOON_DEFAULT (+42.8%) | XML_COMPACT (+47.8%) | XML_COMPACT (-3.3%) | XML_COMPACT (-2.9%) | TOON_DEFAULT (-14.9%) | TOON_DEFAULT (-13.8%) |
| XML_COMPACT (+41.8%) | YAML (+45.2%) | JSON_PRETTY (+101.8%) | JSON_PRETTY (-3.9%) | JSON_PRETTY (-4.2%) | YAML (-23.9%) | YAML (-23.1%) |
| JSON_PRETTY (+75.3%) | JSON_PRETTY (+71.5%) | YAML (+115.4%) | XML_PRETTY (-6.8%) | XML_PRETTY (-6.6%) | JSON_PRETTY (-27.1%) | JSON_PRETTY (-27.0%) |
| JSON_COMPACT (+75.9%) | XML_PRETTY (+99.8%) | XML_PRETTY (+162.0%) | YAML (-10.6%) | YAML (-10.1%) | XML_PRETTY (-38.9%) | XML_PRETTY (-38.6%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99% | YAML ≈ 70% | YAML ≈ 62% | XML_COMPACT ≈ 68% |
| JSON_COMPACT (-3.0%) | JSON_PRETTY (-0.7%) | JSON_PRETTY (-1.9%) | JSON_COMPACT (-8.3%) |
| TOON_DEFAULT (-5.8%) | XML_COMPACT (-1.2%) | JSON_COMPACT (-3.8%) | TOON_DEFAULT (-10.5%) |
| JSON_PRETTY (-7.8%) | TOON_DEFAULT (-2.3%) | TOON_DEFAULT (-4.2%) | XML_PRETTY (-19.0%) |
| XML_COMPACT (-8.5%) | XML_PRETTY (-7.4%) | XML_COMPACT (-6.3%) | JSON_PRETTY (-19.7%) |
| XML_PRETTY (-10.9%) | JSON_COMPACT (-9.6%) | XML_PRETTY (-7.9%) | YAML (-23.8%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 98% | TOON_DEFAULT ≈ 78% | JSON_PRETTY ≈ 68% | JSON_COMPACT ≈ 53% |
| JSON_PRETTY (-2.4%) | XML_COMPACT (-4.9%) | TOON_DEFAULT (-1.6%) | TOON_DEFAULT (-8.4%) |
| XML_COMPACT (-3.6%) | JSON_COMPACT (-10.4%) | JSON_COMPACT (-4.4%) | XML_PRETTY (-12.1%) |
| TOON_DEFAULT (-4.7%) | XML_PRETTY (-11.1%) | XML_PRETTY (-6.3%) | YAML (-13.6%) |
| XML_PRETTY (-9.7%) | YAML (-14.8%) | YAML (-6.3%) | JSON_PRETTY (-13.7%) |
| YAML (-15.8%) | JSON_PRETTY (-16.1%) | XML_COMPACT (-6.4%) | XML_COMPACT (-15.2%) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Char/Token | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10163 | 305 | 10468 | 2.246 | 0.726 | 2.461 | 75.97 | 79.92 | 7952.692 | 2515.508 | 81.68 | 79.92 |
| JSON_COMPACT | opt | 9645 | 305 | 9950 | 2.219 | 0.785 | 2.458 | 78.07 | 83.48 | 7767.809 | 2181.991 | 84.62 | 83.48 |
| JSON_PRETTY | man | 17682 | 307 | 17989 | 1.777 | 0.412 | 2.473 | 74.19 | 58.44 | 13345.742 | 4642.858 | 59.10 | 58.44 |
| JSON_PRETTY | opt | 16757 | 305 | 17062 | 1.767 | 0.435 | 2.460 | 74.19 | 60.96 | 12658.298 | 4403.702 | 61.73 | 60.96 |
| TOON_DEFAULT | man | 14183 | 309 | 14492 | 1.840 | 0.524 | 2.492 | 75.91 | 69.03 | 11000.903 | 3491.131 | 70.22 | 69.03 |
| TOON_DEFAULT | opt | 13948 | 258 | 14206 | 1.848 | 0.544 | 2.078 | 77.30 | 71.98 | 10980.980 | 3224.686 | 72.01 | 71.98 |
| XML_COMPACT | man | 12705 | 305 | 13010 | 2.551 | 0.587 | 2.460 | 76.34 | 73.34 | 9931.834 | 3078.166 | 74.73 | 73.34 |
| XML_COMPACT | opt | 12455 | 307 | 12762 | 2.499 | 0.586 | 2.476 | 74.73 | 74.05 | 9537.043 | 3224.957 | 74.31 | 74.05 |
| XML_PRETTY | man | 20204 | 302 | 20506 | 1.985 | 0.343 | 2.438 | 70.43 | 48.29 | 14442.552 | 6063.698 | 49.33 | 48.29 |
| XML_PRETTY | opt | 19671 | 205 | 19876 | 1.974 | 0.358 | 1.653 | 71.24 | 51.29 | 14159.662 | 5716.338 | 51.68 | 51.29 |
| YAML | man | 14155 | 213 | 14368 | 1.808 | 0.539 | 1.715 | 77.42 | 70.81 | 11123.448 | 3244.219 | 71.63 | 70.81 |
| YAML | opt | 14148 | 303 | 14451 | 1.787 | 0.467 | 2.441 | 67.47 | 64.20 | 9749.865 | 4700.802 | 64.43 | 64.20 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Accuracy Man (%) | Accuracy Opt (%) | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10468 | 9950 | -518 | -4.95 | 75.97 | 78.07 |  +2.10 | 73.45 | 76.44 |  +2.99 | 81.68 | 84.62 |  +2.94 | 79.92 | 83.48 |  +3.56 |
| JSON_PRETTY | 17989 | 17062 | -927 | -5.15 | 74.19 | 74.19 | 0.00 | 73.24 | 73.09 | -0.15 | 59.10 | 61.73 |  +2.63 | 58.44 | 60.96 |  +2.52 |
| TOON_DEFAULT | 14492 | 14206 | -286 | -1.97 | 75.91 | 77.30 |  +1.39 | 74.21 | 77.26 |  +3.05 | 70.22 | 72.01 |  +1.78 | 69.03 | 71.98 |  +2.95 |
| XML_COMPACT | 13010 | 12762 | -248 | -1.91 | 76.34 | 74.73 | -1.61 | 74.35 | 74.36 |  +0.01 | 74.73 | 74.31 | -0.42 | 73.34 | 74.05 |  +0.71 |
| XML_PRETTY | 20506 | 19876 | -630 | -3.07 | 70.43 | 71.24 |  +0.81 | 68.94 | 70.68 |  +1.74 | 49.33 | 51.68 |  +2.35 | 48.29 | 51.29 |  +3.01 |
| YAML | 14368 | 14451 |  +83 |  +0.58 | 77.42 | 67.47 | -9.95 | 76.25 | 67.13 | -9.12 | 71.63 | 64.43 | -7.20 | 70.81 | 64.20 | -6.62 |

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
| JSON_COMPACT | man | 2.246 | 14.902 | 327.839 | 0.726 |
| JSON_COMPACT | opt | 2.219 | 15.285 | 311.129 | 0.785 |
| JSON_PRETTY | man | 1.777 | 25.927 | 570.387 | 0.412 |
| JSON_PRETTY | opt | 1.767 | 26.556 | 540.548 | 0.435 |
| TOON_DEFAULT | man | 1.840 | 20.796 | 457.516 | 0.524 |
| TOON_DEFAULT | opt | 1.848 | 22.105 | 449.935 | 0.544 |
| XML_COMPACT | man | 2.551 | 18.629 | 409.839 | 0.587 |
| XML_COMPACT | opt | 2.499 | 19.739 | 401.774 | 0.586 |
| XML_PRETTY | man | 1.985 | 29.625 | 651.742 | 0.343 |
| XML_PRETTY | opt | 1.974 | 31.174 | 634.548 | 0.358 |
| YAML | man | 1.808 | 20.755 | 456.613 | 0.539 |
| YAML | opt | 1.787 | 22.422 | 456.387 | 0.467 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 2.246 | 2.219 | -0.027 | -1.20 | 14.902 | 15.285 |  +0.383 |  +2.57 | 327.839 | 311.129 | -16.710 | -5.10 | 0.726 | 0.785 |  +0.059 |  +8.13 |
| JSON_PRETTY | 1.777 | 1.767 | -0.010 | -0.56 | 25.927 | 26.556 |  +0.629 |  +2.43 | 570.387 | 540.548 | -29.839 | -5.23 | 0.412 | 0.435 |  +0.023 |  +5.58 |
| TOON_DEFAULT | 1.840 | 1.848 |  +0.008 |  +0.43 | 20.796 | 22.105 |  +1.309 |  +6.29 | 457.516 | 449.935 | -7.581 | -1.66 | 0.524 | 0.544 |  +0.020 |  +3.82 |
| XML_COMPACT | 2.551 | 2.499 | -0.052 | -2.04 | 18.629 | 19.739 |  +1.110 |  +5.96 | 409.839 | 401.774 | -8.065 | -1.97 | 0.587 | 0.586 | -0.001 | -0.17 |
| XML_PRETTY | 1.985 | 1.974 | -0.011 | -0.55 | 29.625 | 31.174 |  +1.549 |  +5.23 | 651.742 | 634.548 | -17.194 | -2.64 | 0.343 | 0.358 |  +0.015 |  +4.37 |
| YAML | 1.808 | 1.787 | -0.021 | -1.16 | 20.755 | 22.422 |  +1.667 |  +8.03 | 456.613 | 456.387 | -0.226 | -0.05 | 0.539 | 0.467 | -0.072 | -13.36 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10468 | 7953 | 2516 | 75.97 | 73.45 | 81.68 | 79.92 |
| JSON_COMPACT | opt | 9950 | 7768 | 2182 | 78.07 | 76.44 | 84.62 | 83.48 |
| JSON_PRETTY | man | 17989 | 13346 | 4643 | 74.19 | 73.24 | 59.10 | 58.44 |
| JSON_PRETTY | opt | 17062 | 12658 | 4404 | 74.19 | 73.09 | 61.73 | 60.96 |
| TOON_DEFAULT | man | 14492 | 11001 | 3491 | 75.91 | 74.21 | 70.22 | 69.03 |
| TOON_DEFAULT | opt | 14206 | 10981 | 3225 | 77.30 | 77.26 | 72.01 | 71.98 |
| XML_COMPACT | man | 13010 | 9932 | 3078 | 76.34 | 74.35 | 74.73 | 73.34 |
| XML_COMPACT | opt | 12762 | 9537 | 3225 | 74.73 | 74.36 | 74.31 | 74.05 |
| XML_PRETTY | man | 20506 | 14443 | 6064 | 70.43 | 68.94 | 49.33 | 48.29 |
| XML_PRETTY | opt | 19876 | 14160 | 5716 | 71.24 | 70.68 | 51.68 | 51.29 |
| YAML | man | 14368 | 11123 | 3244 | 77.42 | 76.25 | 71.63 | 70.81 |
| YAML | opt | 14451 | 9750 | 4701 | 67.47 | 67.13 | 64.43 | 64.20 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10468 | 9950 | -518 | -4.95 | 7953 | 7768 | -185 | -2.32 | 2516 | 2182 | -334 | -13.26 | 75.97 | 78.07 |  +2.10 | 81.68 | 84.621 |  +2.94 |  +3.60 |
| JSON_PRETTY | 17989 | 17062 | -927 | -5.15 | 13346 | 12659 | -687 | -5.15 | 4643 | 4404 | -239 | -5.15 | 74.19 | 74.19 | 0.00 | 59.10 | 61.731 |  +2.63 |  +4.45 |
| TOON_DEFAULT | 14492 | 14206 | -286 | -1.98 | 11001 | 10981 | -20 | -0.18 | 3491 | 3225 | -266 | -7.63 | 75.91 | 77.30 |  +1.39 | 70.22 | 72.0095 |  +1.78 |  +2.54 |
| XML_COMPACT | 13010 | 12762 | -248 | -1.91 | 9932 | 9537 | -395 | -3.97 | 3078 | 3225 |  +147 |  +4.77 | 76.34 | 74.73 | -1.61 | 74.73 | 74.306 | -0.42 | -0.57 |
| XML_PRETTY | 20506 | 19876 | -630 | -3.07 | 14443 | 14160 | -283 | -1.96 | 6064 | 5717 | -347 | -5.73 | 70.43 | 71.24 |  +0.81 | 49.33 | 51.684 |  +2.35 |  +4.77 |
| YAML | 14368 | 14451 |  +83 |  +0.58 | 11123 | 9749 | -1374 | -12.35 | 3244 | 4701 |  +1457 |  +44.90 | 77.42 | 67.47 | -9.95 | 71.63 | 64.434 | -7.20 | -10.05 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| JSON_COMPACT | man | 94 | 30 | 0 | 75.97 |
| JSON_COMPACT | opt | 97 | 27 | 0 | 78.07 |
| JSON_PRETTY | man | 92 | 32 | 0 | 74.19 |
| JSON_PRETTY | opt | 92 | 32 | 0 | 74.19 |
| TOON_DEFAULT | man | 94 | 30 | 0 | 75.91 |
| TOON_DEFAULT | opt | 96 | 28 | 0 | 77.30 |
| XML_COMPACT | man | 95 | 29 | 0 | 76.34 |
| XML_COMPACT | opt | 93 | 31 | 0 | 74.73 |
| XML_PRETTY | man | 87 | 37 | 0 | 70.43 |
| XML_PRETTY | opt | 88 | 36 | 0 | 71.24 |
| YAML | man | 96 | 28 | 0 | 77.42 |
| YAML | opt | 84 | 40 | 0 | 67.47 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 94 | 97 |  +3 |  +3.19 | 30 | 27 | -3 | -10.00 | 0 | 0 | 0 | 0.00 | 75.97 | 78.07 |  +2.10 |
| JSON_PRETTY | 92 | 92 | 0 | 0.00 | 32 | 32 | 0 | 0.00 | 0 | 0 | 0 | 0.00 | 74.19 | 74.19 | 0.00 |
| TOON_DEFAULT | 94 | 96 |  +2 |  +2.13 | 30 | 28 | -2 | -6.67 | 0 | 0 | 0 | 0.00 | 75.91 | 77.30 |  +1.39 |
| XML_COMPACT | 95 | 93 | -2 | -2.11 | 29 | 31 |  +2 |  +6.90 | 0 | 0 | 0 | 0.00 | 76.34 | 74.73 | -1.61 |
| XML_PRETTY | 87 | 88 |  +1 |  +1.15 | 37 | 36 | -1 | -2.70 | 0 | 0 | 0 | 0.00 | 70.43 | 71.24 |  +0.81 |
| YAML | 96 | 84 | -12 | -12.50 | 28 | 40 |  +12 |  +42.86 | 0 | 0 | 0 | 0.00 | 77.42 | 67.47 | -9.95 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 75.97 | 96.36 | 60.74 | 58.09 | 60.00 |
| JSON_COMPACT | opt | 78.07 | 98.18 | 67.41 | 63.81 | 53.33 |
| JSON_PRETTY | man | 74.19 | 91.64 | 69.63 | 60.00 | 48.57 |
| JSON_PRETTY | opt | 74.19 | 95.76 | 61.73 | 68.25 | 39.68 |
| TOON_DEFAULT | man | 75.91 | 93.64 | 68.06 | 57.74 | 57.74 |
| TOON_DEFAULT | opt | 77.30 | 93.51 | 77.78 | 66.67 | 44.90 |
| XML_COMPACT | man | 76.34 | 90.91 | 69.14 | 55.55 | 68.25 |
| XML_COMPACT | opt | 74.73 | 94.55 | 72.84 | 61.90 | 38.10 |
| XML_PRETTY | man | 70.43 | 88.48 | 62.96 | 53.97 | 49.21 |
| XML_PRETTY | opt | 71.24 | 88.48 | 66.67 | 61.90 | 41.27 |
| YAML | man | 77.42 | 99.39 | 70.37 | 61.90 | 44.45 |
| YAML | opt | 67.47 | 82.42 | 62.96 | 61.90 | 39.68 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 96.36 | 98.18 |  +1.82 |
| JSON_PRETTY | 91.64 | 95.76 |  +4.12 |
| TOON_DEFAULT | 93.64 | 93.51 | -0.13 |
| XML_COMPACT | 90.91 | 94.55 |  +3.64 |
| XML_PRETTY | 88.48 | 88.48 | 0.00 |
| YAML | 99.39 | 82.42 | -16.97 |

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
- Wasted Token Range: 2182 - 2516 tokens
- Accuracy Range: 75.97 - 78.07%
- Efficiency Score Range: 81.68 - 84.62

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
- Wasted Token Range: 4404 - 4643 tokens
- Accuracy Range: 74.19 - 74.19%
- Efficiency Score Range: 59.10 - 61.73

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
- Wasted Token Range: 3225 - 3491 tokens
- Accuracy Range: 75.91 - 77.30%
- Efficiency Score Range: 70.22 - 72.01

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
- Wasted Token Range: 3078 - 3225 tokens
- Accuracy Range: 74.73 - 76.34%
- Efficiency Score Range: 74.31 - 74.73

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
- Wasted Token Range: 5716 - 6064 tokens
- Accuracy Range: 70.43 - 71.24%
- Efficiency Score Range: 49.33 - 51.68

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
- Wasted Token Range: 3244 - 4701 tokens
- Accuracy Range: 67.47 - 77.42%
- Efficiency Score Range: 64.43 - 71.63

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
- **Test Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: off
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