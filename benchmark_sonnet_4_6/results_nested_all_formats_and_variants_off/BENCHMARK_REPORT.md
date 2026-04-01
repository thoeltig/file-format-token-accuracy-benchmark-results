# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-29
- **Model**: Sonnet 4.6
- **Thinking**: off
- **Data Structure**: nested
- **Formats Tested**: 7 (JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, TOON_KEYFOLD, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: First iteration

## Executive Summary

This benchmark evaluates token efficiency and information accuracy across 7 file formats using Sonnet 4.6 as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context, while a format that accurately conveys information may justify higher token cost.

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
- 7 formats tested: JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, TOON_KEYFOLD, XML_COMPACT, XML_PRETTY, YAML
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
   - Optional: JSON_COMPACT 32368 tokens
   - Mandatory: JSON_COMPACT 35020 tokens
- Lowest read token cost:
   - Optional: JSON_COMPACT 6970 tokens
   - Mandatory: JSON_COMPACT 7498 tokens
- Lowest output token cost:
   - Optional: JSON_PRETTY 25238 tokens
   - Mandatory: JSON_PRETTY 21944 tokens
- Lowest output token cost drift:
   - Optional: XML_COMPACT ↓ -1.71% ↑ 1.74%
   - Mandatory: JSON_PRETTY ↓ -4.13% ↑ 6.34%
- Highest accuracy:
   - Optional: JSON_PRETTY 98.66%
   - Mandatory: TOON_DEFAULT 99.19%
- Lowest accuracy drift:
   - Optional: JSON_PRETTY ↓ -0.27% ↑ 0.54%
   - Mandatory: XML_COMPACT ↓ -0.81% ↑ 0.82%
- Most useful tokens:
   - Optional: XML_PRETTY 42448 / 43860 tokens
   - Mandatory: XML_PRETTY 41622 / 42419 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 98.47
   - Mandatory: JSON_COMPACT 92.12
- Lowest delta (optional-mandatory):
   - Total tokens: TOON_DEFAULT 628 tokens
   - Accuracy: JSON_COMPACT -0.80%
   - Token efficiency: TOON_KEYFOLD 1.81

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 43860 tokens
   - Mandatory: XML_PRETTY 42419 tokens
- Highest read token cost:
   - Optional: XML_PRETTY 17760 tokens
   - Mandatory: XML_PRETTY 18291 tokens
- Highest output token cost:
   - Optional: TOON_DEFAULT 26156 tokens
   - Mandatory: JSON_COMPACT 27522 tokens
- Highest output token drift:
   - Optional: TOON_DEFAULT ↓ -17.29% ↑ 10.00%
   - Mandatory: YAML ↓ -19.87% ↑ 18.48%
- Lowest accuracy:
   - Optional: XML_PRETTY 96.78%
   - Mandatory: JSON_PRETTY 96.50%
- Highest accuracy drift:
   - Optional: TOON_DEFAULT ↓ -1.92% ↑ 1.37%
   - Mandatory: TOON_KEYFOLD ↓ -1.92% ↑ 2.20%
- Most wasted tokens:
   - Optional: XML_PRETTY 1412 / 43860 tokens
   - Mandatory: JSON_PRETTY 1321 / 37731 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 67.77
   - Mandatory: XML_PRETTY 72.47
- Highest delta (optional-mandatory):
   - Total tokens: JSON_PRETTY 5196 tokens
   - Accuracy: JSON_PRETTY 2.16%
   - Token efficiency: JSON_PRETTY -12.03

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 292s | JSON_COMPACT ≈ 7498 | JSON_PRETTY ≈ 21944 | JSON_COMPACT ≈ 35020 | TOON_DEFAULT ≈ 297 | TOON_DEFAULT ≈ 99% | XML_COMPACT ≈ 99% | JSON_COMPACT ≈ 92 | JSON_COMPACT ≈ 92 |
| XML_PRETTY (+5.7%) | XML_COMPACT (+35.6%) | YAML (+7.4%) | YAML (+0.6%) | XML_COMPACT (+0.4%) | XML_COMPACT (0.0%) | TOON_DEFAULT (-0.1%) | YAML (-0.4%) | YAML (-0.2%) |
| YAML (+8.0%) | TOON_KEYFOLD (+50.6%) | XML_PRETTY (+10.0%) | TOON_DEFAULT (+4.9%) | YAML (+27.9%) | YAML (-0.3%) | YAML (-0.2%) | TOON_DEFAULT (-4.4%) | TOON_DEFAULT (-4.4%) |
| TOON_DEFAULT (+12.3%) | TOON_DEFAULT (+52.6%) | TOON_DEFAULT (+15.2%) | XML_COMPACT (+5.3%) | JSON_COMPACT (+58.9%) | JSON_COMPACT (-0.5%) | JSON_COMPACT (-0.7%) | XML_COMPACT (-4.9%) | XML_COMPACT (-4.7%) |
| XML_COMPACT (+15.4%) | YAML (+55.4%) | TOON_KEYFOLD (+20.4%) | TOON_KEYFOLD (+7.7%) | XML_PRETTY (+168.1%) | XML_PRETTY (-1.1%) | XML_PRETTY (-1.3%) | TOON_KEYFOLD (-8.2%) | TOON_KEYFOLD (-8.3%) |
| TOON_KEYFOLD (+16.2%) | JSON_PRETTY (+110.5%) | XML_COMPACT (+21.8%) | JSON_PRETTY (+7.7%) | TOON_KEYFOLD (+172.6%) | TOON_KEYFOLD (-1.3%) | TOON_KEYFOLD (-1.6%) | JSON_PRETTY (-9.3%) | JSON_PRETTY (-9.8%) |
| JSON_COMPACT (+24.3%) | XML_PRETTY (+143.9%) | JSON_COMPACT (+25.4%) | XML_PRETTY (+21.1%) | JSON_PRETTY (+344.0%) | JSON_PRETTY (-2.7%) | JSON_PRETTY (-3.5%) | XML_PRETTY (-21.3%) | XML_PRETTY (-21.4%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| YAML ≈ 323s | JSON_COMPACT ≈ 6970 | JSON_PRETTY ≈ 25238 | JSON_COMPACT ≈ 32368 | JSON_PRETTY ≈ 575 | JSON_PRETTY ≈ 99% | JSON_PRETTY ≈ 99% | JSON_COMPACT ≈ 98 | JSON_COMPACT ≈ 99 |
| JSON_PRETTY (+0.3%) | TOON_KEYFOLD (+58.5%) | YAML (+0.4%) | YAML (+13.5%) | JSON_COMPACT (+21.0%) | XML_COMPACT (-0.5%) | XML_COMPACT (-0.4%) | YAML (-12.0%) | YAML (-11.9%) |
| TOON_KEYFOLD (+0.3%) | TOON_DEFAULT (+60.6%) | JSON_COMPACT (+0.6%) | TOON_KEYFOLD (+13.7%) | XML_COMPACT (+24.9%) | JSON_COMPACT (-0.8%) | JSON_COMPACT (-0.9%) | TOON_KEYFOLD (-12.3%) | TOON_KEYFOLD (-12.3%) |
| TOON_DEFAULT (+1.9%) | YAML (+63.6%) | XML_COMPACT (+1.9%) | TOON_DEFAULT (+15.4%) | TOON_DEFAULT (+39.6%) | TOON_DEFAULT (-0.8%) | TOON_DEFAULT (-0.9%) | TOON_DEFAULT (-13.2%) | TOON_DEFAULT (-13.2%) |
| JSON_COMPACT (+2.3%) | XML_COMPACT (+79.3%) | TOON_KEYFOLD (+2.0%) | XML_COMPACT (+18.1%) | YAML (+71.8%) | YAML (-1.3%) | YAML (-1.4%) | XML_COMPACT (-15.3%) | XML_COMPACT (-15.1%) |
| XML_COMPACT (+2.3%) | JSON_PRETTY (+153.8%) | XML_PRETTY (+3.4%) | JSON_PRETTY (+32.6%) | TOON_KEYFOLD (+89.4%) | TOON_KEYFOLD (-1.6%) | TOON_KEYFOLD (-1.7%) | JSON_PRETTY (-27.4%) | JSON_PRETTY (-27.2%) |
| XML_PRETTY (+4.6%) | XML_PRETTY (+154.8%) | TOON_DEFAULT (+3.6%) | XML_PRETTY (+35.5%) | XML_PRETTY (+145.5%) | XML_PRETTY (-1.9%) | XML_PRETTY (-2.2%) | XML_PRETTY (-31.2%) | XML_PRETTY (-31.3%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 100% | TOON_DEFAULT ≈ 100% | YAML ≈ 98% | JSON_COMPACT ≈ 100% |
| TOON_DEFAULT (0.0%) | XML_COMPACT (0.0%) | XML_COMPACT (-1.6%) | JSON_PRETTY (0.0%) |
| TOON_KEYFOLD (0.0%) | XML_PRETTY (0.0%) | TOON_KEYFOLD (-3.2%) | TOON_DEFAULT (0.0%) |
| XML_COMPACT (0.0%) | JSON_COMPACT (-1.2%) | JSON_COMPACT (-3.2%) | XML_COMPACT (-1.6%) |
| XML_PRETTY (0.0%) | YAML (-1.2%) | TOON_DEFAULT (-3.2%) | XML_PRETTY (-1.6%) |
| YAML (0.0%) | JSON_PRETTY (-2.5%) | XML_PRETTY (-7.9%) | TOON_KEYFOLD (-3.2%) |
| JSON_COMPACT (-0.6%) | TOON_KEYFOLD (-3.7%) | JSON_PRETTY (-15.9%) | YAML (-3.2%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 100% | JSON_PRETTY ≈ 100% | JSON_COMPACT ≈ 100% | JSON_PRETTY ≈ 92% |
| JSON_PRETTY (0.0%) | TOON_KEYFOLD (0.0%) | JSON_PRETTY (0.0%) | JSON_COMPACT (-1.6%) |
| TOON_DEFAULT (0.0%) | XML_COMPACT (0.0%) | TOON_DEFAULT (0.0%) | XML_PRETTY (-1.6%) |
| TOON_KEYFOLD (0.0%) | XML_PRETTY (0.0%) | XML_COMPACT (0.0%) | TOON_DEFAULT (-1.6%) |
| XML_COMPACT (0.0%) | YAML (-1.2%) | YAML (-3.2%) | TOON_KEYFOLD (-3.2%) |
| XML_PRETTY (0.0%) | JSON_COMPACT (-2.5%) | TOON_KEYFOLD (-6.4%) | XML_COMPACT (-3.2%) |
| YAML (0.0%) | TOON_DEFAULT (-2.5%) | XML_PRETTY (-9.5%) | YAML (-3.2%) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Char/Token | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 7498 | 27522 | 35020 | 3.190 | 0.282 | 221.949 | 98.65 | 91.96 | 34546.901 | 472.766 | 92.12 | 91.96 |
| JSON_COMPACT | opt | 6970 | 25398 | 32368 | 3.226 | 0.302 | 204.823 | 97.85 | 98.64 | 31672.088 | 695.912 | 98.47 | 98.64 |
| JSON_PRETTY | man | 15787 | 21944 | 37731 | 2.094 | 0.256 | 176.965 | 96.50 | 82.95 | 36410.094 | 1320.573 | 83.55 | 82.95 |
| JSON_PRETTY | opt | 17688 | 25238 | 42926 | 1.768 | 0.230 | 203.535 | 98.66 | 71.77 | 42351.120 | 575.213 | 71.52 | 71.77 |
| TOON_DEFAULT | man | 11440 | 25283 | 36723 | 2.370 | 0.270 | 203.895 | 99.19 | 87.93 | 36425.544 | 297.456 | 88.06 | 87.93 |
| TOON_DEFAULT | opt | 11195 | 26156 | 37351 | 2.394 | 0.262 | 210.933 | 97.85 | 85.65 | 36547.628 | 803.039 | 85.48 | 85.65 |
| TOON_KEYFOLD | man | 11289 | 26419 | 37708 | 2.382 | 0.259 | 213.056 | 97.85 | 84.33 | 36897.278 | 810.722 | 84.55 | 84.33 |
| TOON_KEYFOLD | opt | 11044 | 25752 | 36796 | 2.407 | 0.264 | 207.680 | 97.04 | 86.54 | 35707.162 | 1089.171 | 86.36 | 86.54 |
| XML_COMPACT | man | 10164 | 26719 | 36883 | 3.371 | 0.269 | 215.476 | 99.19 | 87.61 | 36584.248 | 298.752 | 87.64 | 87.61 |
| XML_COMPACT | opt | 12494 | 25730 | 38224 | 2.641 | 0.257 | 207.503 | 98.12 | 83.74 | 37505.716 | 718.617 | 83.40 | 83.74 |
| XML_PRETTY | man | 18291 | 24128 | 42419 | 2.321 | 0.231 | 194.581 | 98.12 | 72.26 | 41621.523 | 797.477 | 72.47 | 72.26 |
| XML_PRETTY | opt | 17760 | 26100 | 43860 | 2.319 | 0.221 | 210.487 | 96.78 | 67.81 | 42448.030 | 1412.303 | 67.77 | 67.81 |
| YAML | man | 11649 | 23577 | 35226 | 2.280 | 0.281 | 190.137 | 98.92 | 91.76 | 34845.559 | 380.441 | 91.77 | 91.76 |
| YAML | opt | 11404 | 25338 | 36742 | 2.301 | 0.265 | 204.336 | 97.31 | 86.89 | 35753.316 | 988.351 | 86.69 | 86.89 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7498 | 6970 | -528 | -7.04 | 27522 | 25398 | -2124 | -7.72 | 35020 | 32368 | -2652 | -7.57 | 98.42 | 98.09 | -0.33 | 91.96 | 98.64 |  +6.68 |
| JSON_PRETTY | 15787 | 17688 |  +1901 |  +12.04 | 21944 | 25239 |  +3295 |  +15.02 | 37731 | 42927 |  +5196 |  +13.77 | 95.65 | 99.01 |  +3.36 | 82.95 | 71.77 | -11.19 |
| TOON_DEFAULT | 11440 | 11195 | -245 | -2.14 | 25283 | 26156 |  +873 |  +3.45 | 36723 | 37351 |  +628 |  +1.71 | 99.01 | 98.09 | -0.92 | 87.93 | 85.65 | -2.28 |
| TOON_KEYFOLD | 11289 | 11044 | -245 | -2.17 | 26419 | 25752 | -667 | -2.52 | 37708 | 36796 | -912 | -2.42 | 97.53 | 97.29 | -0.24 | 84.33 | 86.54 |  +2.21 |
| XML_COMPACT | 10164 | 12494 |  +2330 |  +22.92 | 26719 | 25730 | -989 | -3.70 | 36883 | 38224 |  +1341 |  +3.64 | 99.14 | 98.61 | -0.53 | 87.61 | 83.74 | -3.87 |
| XML_PRETTY | 18291 | 17760 | -531 | -2.90 | 24128 | 26100 |  +1972 |  +8.17 | 42419 | 43860 |  +1441 |  +3.40 | 97.82 | 96.83 | -0.99 | 72.26 | 67.81 | -4.45 |
| YAML | 11649 | 11404 | -245 | -2.10 | 23577 | 25338 |  +1761 |  +7.47 | 35226 | 36742 |  +1516 |  +4.30 | 98.91 | 97.59 | -1.32 | 91.76 | 86.89 | -4.87 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 35 | 214.229 | 1.13 | 363175 | 0.076 | 2928.83 | 363210 | 214.305 | 2343.29 |
| JSON_COMPACT | opt | 19 | 366.842 | 0.61 | 330751 | 0.077 | 2667.34 | 330770 | 366.919 | 2134.00 |
| JSON_PRETTY | man | 319 | 49.489 | 10.29 | 291933 | 0.075 | 2354.30 | 292252 | 49.564 | 1885.50 |
| JSON_PRETTY | opt | 331 | 53.438 | 10.68 | 324103 | 0.078 | 2613.73 | 324434 | 53.516 | 2093.12 |
| TOON_DEFAULT | man | 54 | 211.852 | 1.74 | 328166 | 0.077 | 2646.50 | 328220 | 211.929 | 2117.55 |
| TOON_DEFAULT | opt | 40 | 279.875 | 1.29 | 329622 | 0.079 | 2658.24 | 329662 | 279.954 | 2126.85 |
| TOON_KEYFOLD | man | 35 | 322.543 | 1.13 | 339495 | 0.078 | 2737.86 | 339530 | 322.621 | 2190.52 |
| TOON_KEYFOLD | opt | 41 | 269.366 | 1.32 | 324478 | 0.079 | 2616.76 | 324519 | 269.445 | 2093.67 |
| XML_COMPACT | man | 110 | 92.400 | 3.55 | 337256 | 0.079 | 2719.81 | 337366 | 92.479 | 2176.55 |
| XML_COMPACT | opt | 116 | 107.707 | 3.74 | 330916 | 0.078 | 2668.68 | 331032 | 107.785 | 2135.69 |
| XML_PRETTY | man | 37 | 494.351 | 1.19 | 308772 | 0.078 | 2490.10 | 308809 | 494.429 | 1992.32 |
| XML_PRETTY | opt | 42 | 422.857 | 1.35 | 338337 | 0.077 | 2728.52 | 338379 | 422.934 | 2183.09 |
| YAML | man | 85 | 137.047 | 2.74 | 315630 | 0.075 | 2545.40 | 315715 | 137.122 | 2036.87 |
| YAML | opt | 81 | 140.790 | 2.61 | 323407 | 0.078 | 2608.12 | 323488 | 140.868 | 2087.02 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) | Total Man (s) | Total Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 35 | 19 | -16 | -45.71 | 363.17 | 330.75 | -32.42 | -8.93 | 363.21 | 330.77 | -32.44 | -8.93 |
| JSON_PRETTY | 319 | 331 |  +12 |  +3.76 | 291.93 | 324.10 |  +32.17 |  +11.02 | 292.25 | 324.43 |  +32.18 |  +11.01 |
| TOON_DEFAULT | 54 | 40 | -14 | -25.93 | 328.17 | 329.62 |  +1.46 |  +0.44 | 328.22 | 329.66 |  +1.44 |  +0.44 |
| TOON_KEYFOLD | 35 | 41 |  +6 |  +17.14 | 339.50 | 324.48 | -15.02 | -4.42 | 339.53 | 324.52 | -15.01 | -4.42 |
| XML_COMPACT | 110 | 116 |  +6 |  +5.45 | 337.26 | 330.92 | -6.34 | -1.88 | 337.37 | 331.03 | -6.33 | -1.88 |
| XML_PRETTY | 37 | 42 |  +5 |  +13.51 | 308.77 | 338.34 |  +29.56 |  +9.57 | 308.81 | 338.38 |  +29.57 |  +9.58 |
| YAML | 85 | 81 | -4 | -4.71 | 315.63 | 323.41 |  +7.78 |  +2.46 | 315.71 | 323.49 |  +7.77 |  +2.46 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Char/Token | Token/Value | Token/Object | Info/Token |
|---|---|---|---|---|---|
| JSON_COMPACT | man | 3.190 | 10.994 | 241.871 | 0.282 |
| JSON_COMPACT | opt | 3.226 | 11.046 | 224.839 | 0.302 |
| JSON_PRETTY | man | 2.094 | 23.148 | 509.258 | 0.256 |
| JSON_PRETTY | opt | 1.768 | 28.032 | 570.581 | 0.230 |
| TOON_DEFAULT | man | 2.370 | 16.774 | 369.032 | 0.270 |
| TOON_DEFAULT | opt | 2.394 | 17.742 | 361.129 | 0.262 |
| TOON_KEYFOLD | man | 2.382 | 16.553 | 364.161 | 0.259 |
| TOON_KEYFOLD | opt | 2.407 | 17.502 | 356.258 | 0.264 |
| XML_COMPACT | man | 3.371 | 14.903 | 327.871 | 0.269 |
| XML_COMPACT | opt | 2.641 | 19.800 | 403.032 | 0.257 |
| XML_PRETTY | man | 2.321 | 26.820 | 590.032 | 0.231 |
| XML_PRETTY | opt | 2.319 | 28.146 | 572.903 | 0.221 |
| YAML | man | 2.280 | 17.081 | 375.774 | 0.281 |
| YAML | opt | 2.301 | 18.073 | 367.871 | 0.265 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 3.190 | 3.226 |  +0.036 |  +1.13 | 10.994 | 11.046 |  +0.052 |  +0.47 | 241.871 | 224.839 | -17.032 | -7.04 | 0.282 | 0.302 |  +0.020 |  +7.09 |
| JSON_PRETTY | 2.094 | 1.768 | -0.326 | -15.57 | 23.148 | 28.032 |  +4.884 |  +21.10 | 509.258 | 570.581 |  +61.323 |  +12.04 | 0.256 | 0.230 | -0.026 | -10.16 |
| TOON_DEFAULT | 2.370 | 2.394 |  +0.024 |  +1.01 | 16.774 | 17.742 |  +0.968 |  +5.77 | 369.032 | 361.129 | -7.903 | -2.14 | 0.270 | 0.262 | -0.008 | -2.96 |
| TOON_KEYFOLD | 2.382 | 2.407 |  +0.025 |  +1.05 | 16.553 | 17.502 |  +0.949 |  +5.73 | 364.161 | 356.258 | -7.903 | -2.17 | 0.259 | 0.264 |  +0.005 |  +1.93 |
| XML_COMPACT | 3.371 | 2.641 | -0.730 | -21.66 | 14.903 | 19.800 |  +4.897 |  +32.86 | 327.871 | 403.032 |  +75.161 |  +22.92 | 0.269 | 0.257 | -0.012 | -4.46 |
| XML_PRETTY | 2.321 | 2.319 | -0.002 | -0.09 | 26.820 | 28.146 |  +1.326 |  +4.94 | 590.032 | 572.903 | -17.129 | -2.90 | 0.231 | 0.221 | -0.010 | -4.33 |
| YAML | 2.280 | 2.301 |  +0.021 |  +0.92 | 17.081 | 18.073 |  +0.992 |  +5.81 | 375.774 | 367.871 | -7.903 | -2.10 | 0.281 | 0.265 | -0.016 | -5.69 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 35020 | 34547 | 473 | 98.65 | 98.42 | 92.12 | 91.96 |
| JSON_COMPACT | opt | 32368 | 31672 | 696 | 97.85 | 98.09 | 98.47 | 98.64 |
| JSON_PRETTY | man | 37731 | 36410 | 1321 | 96.50 | 95.65 | 83.55 | 82.95 |
| JSON_PRETTY | opt | 42926 | 42351 | 575 | 98.66 | 99.01 | 71.52 | 71.77 |
| TOON_DEFAULT | man | 36723 | 36426 | 297 | 99.19 | 99.01 | 88.06 | 87.93 |
| TOON_DEFAULT | opt | 37351 | 36548 | 803 | 97.85 | 98.09 | 85.48 | 85.65 |
| TOON_KEYFOLD | man | 37708 | 36897 | 811 | 97.85 | 97.53 | 84.55 | 84.33 |
| TOON_KEYFOLD | opt | 36796 | 35707 | 1089 | 97.04 | 97.29 | 86.36 | 86.54 |
| XML_COMPACT | man | 36883 | 36584 | 299 | 99.19 | 99.14 | 87.64 | 87.61 |
| XML_COMPACT | opt | 38224 | 37506 | 719 | 98.12 | 98.61 | 83.40 | 83.74 |
| XML_PRETTY | man | 42419 | 41622 | 797 | 98.12 | 97.82 | 72.47 | 72.26 |
| XML_PRETTY | opt | 43860 | 42448 | 1412 | 96.78 | 96.83 | 67.77 | 67.81 |
| YAML | man | 35226 | 34846 | 380 | 98.92 | 98.91 | 91.77 | 91.76 |
| YAML | opt | 36742 | 35753 | 988 | 97.31 | 97.59 | 86.69 | 86.89 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 35020 | 32368 | -2652 | -7.57 | 34547 | 31672 | -2875 | -8.32 | 473 | 696 |  +223 |  +47.18 | 98.65 | 97.85 | -0.80 | 92.12 | 98.469 |  +6.35 |  +6.89 |
| JSON_PRETTY | 37731 | 42927 |  +5196 |  +13.77 | 36410 | 42351 |  +5941 |  +16.32 | 1321 | 576 | -745 | -56.42 | 96.50 | 98.66 |  +2.16 | 83.55 | 71.522 | -12.03 | -14.40 |
| TOON_DEFAULT | 36723 | 37351 |  +628 |  +1.71 | 36426 | 36548 |  +122 |  +0.34 | 297 | 803 |  +506 |  +170.23 | 99.19 | 97.85 | -1.34 | 88.06 | 85.485 | -2.57 | -2.92 |
| TOON_KEYFOLD | 37708 | 36796 | -912 | -2.42 | 36897 | 35707 | -1190 | -3.23 | 811 | 1089 |  +278 |  +34.33 | 97.85 | 97.04 | -0.81 | 84.55 | 86.362 |  +1.81 |  +2.14 |
| XML_COMPACT | 36883 | 38224 |  +1341 |  +3.64 | 36584 | 37505 |  +921 |  +2.52 | 299 | 719 |  +420 |  +140.42 | 99.19 | 98.12 | -1.07 | 87.64 | 83.397 | -4.24 | -4.84 |
| XML_PRETTY | 42419 | 43860 |  +1441 |  +3.40 | 41622 | 42449 |  +827 |  +1.99 | 797 | 1412 |  +615 |  +77.14 | 98.12 | 96.78 | -1.34 | 72.47 | 67.772 | -4.69 | -6.48 |
| YAML | 35226 | 36742 |  +1516 |  +4.30 | 34846 | 35754 |  +908 |  +2.61 | 380 | 988 |  +608 |  +159.98 | 98.92 | 97.31 | -1.61 | 91.77 | 86.694 | -5.08 | -5.53 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| JSON_COMPACT | man | 122 | 2 | 0 | 98.65 |
| JSON_COMPACT | opt | 121 | 3 | 0 | 97.85 |
| JSON_PRETTY | man | 120 | 4 | 0 | 96.50 |
| JSON_PRETTY | opt | 122 | 2 | 0 | 98.66 |
| TOON_DEFAULT | man | 123 | 1 | 0 | 99.19 |
| TOON_DEFAULT | opt | 121 | 3 | 0 | 97.85 |
| TOON_KEYFOLD | man | 121 | 3 | 0 | 97.85 |
| TOON_KEYFOLD | opt | 120 | 4 | 0 | 97.04 |
| XML_COMPACT | man | 123 | 1 | 0 | 99.19 |
| XML_COMPACT | opt | 122 | 2 | 0 | 98.12 |
| XML_PRETTY | man | 122 | 2 | 0 | 98.12 |
| XML_PRETTY | opt | 120 | 4 | 0 | 96.78 |
| YAML | man | 123 | 1 | 0 | 98.92 |
| YAML | opt | 121 | 3 | 0 | 97.31 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 122 | 121 | -1 | -0.82 | 2 | 3 |  +1 |  +50.00 | 0 | 0 | 0 | 0.00 | 98.65 | 97.85 | -0.80 |
| JSON_PRETTY | 120 | 122 |  +2 |  +1.67 | 4 | 2 | -2 | -50.00 | 0 | 0 | 0 | 0.00 | 96.50 | 98.66 |  +2.16 |
| TOON_DEFAULT | 123 | 121 | -2 | -1.63 | 1 | 3 |  +2 |  +200.00 | 0 | 0 | 0 | 0.00 | 99.19 | 97.85 | -1.34 |
| TOON_KEYFOLD | 121 | 120 | -1 | -0.83 | 3 | 4 |  +1 |  +33.33 | 0 | 0 | 0 | 0.00 | 97.85 | 97.04 | -0.81 |
| XML_COMPACT | 123 | 122 | -1 | -0.81 | 1 | 2 |  +1 |  +100.00 | 0 | 0 | 0 | 0.00 | 99.19 | 98.12 | -1.07 |
| XML_PRETTY | 122 | 120 | -2 | -1.64 | 2 | 4 |  +2 |  +100.00 | 0 | 0 | 0 | 0.00 | 98.12 | 96.78 | -1.34 |
| YAML | 123 | 121 | -2 | -1.63 | 1 | 3 |  +2 |  +200.00 | 0 | 0 | 0 | 0.00 | 98.92 | 97.31 | -1.61 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 98.65 | 99.39 | 98.77 | 95.24 | 100.00 |
| JSON_COMPACT | opt | 97.85 | 100.00 | 97.53 | 100.00 | 90.48 |
| JSON_PRETTY | man | 96.50 | 100.00 | 97.53 | 82.54 | 100.00 |
| JSON_PRETTY | opt | 98.66 | 100.00 | 100.00 | 100.00 | 92.07 |
| TOON_DEFAULT | man | 99.19 | 100.00 | 100.00 | 95.24 | 100.00 |
| TOON_DEFAULT | opt | 97.85 | 100.00 | 97.53 | 100.00 | 90.48 |
| TOON_KEYFOLD | man | 97.85 | 100.00 | 96.30 | 95.24 | 96.83 |
| TOON_KEYFOLD | opt | 97.04 | 100.00 | 100.00 | 93.65 | 88.89 |
| XML_COMPACT | man | 99.19 | 100.00 | 100.00 | 96.83 | 98.41 |
| XML_COMPACT | opt | 98.12 | 100.00 | 100.00 | 100.00 | 88.89 |
| XML_PRETTY | man | 98.12 | 100.00 | 100.00 | 90.47 | 98.41 |
| XML_PRETTY | opt | 96.78 | 100.00 | 100.00 | 90.47 | 90.48 |
| YAML | man | 98.92 | 100.00 | 98.77 | 98.41 | 96.83 |
| YAML | opt | 97.31 | 100.00 | 98.77 | 96.83 | 88.89 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 99.39 | 100.00 |  +0.61 |
| JSON_PRETTY | 100.00 | 100.00 | 0.00 |
| TOON_DEFAULT | 100.00 | 100.00 | 0.00 |
| TOON_KEYFOLD | 100.00 | 100.00 | 0.00 |
| XML_COMPACT | 100.00 | 100.00 | 0.00 |
| XML_PRETTY | 100.00 | 100.00 | 0.00 |
| YAML | 100.00 | 100.00 | 0.00 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 98.77 | 97.53 | -1.24 |
| JSON_PRETTY | 97.53 | 100.00 |  +2.47 |
| TOON_DEFAULT | 100.00 | 97.53 | -2.47 |
| TOON_KEYFOLD | 96.30 | 100.00 |  +3.70 |
| XML_COMPACT | 100.00 | 100.00 | 0.00 |
| XML_PRETTY | 100.00 | 100.00 | 0.00 |
| YAML | 98.77 | 98.77 | 0.00 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 95.24 | 100.00 |  +4.76 |
| JSON_PRETTY | 82.54 | 100.00 |  +17.46 |
| TOON_DEFAULT | 95.24 | 100.00 |  +4.76 |
| TOON_KEYFOLD | 95.24 | 93.65 | -1.59 |
| XML_COMPACT | 96.83 | 100.00 |  +3.17 |
| XML_PRETTY | 90.47 | 90.47 | 0.00 |
| YAML | 98.41 | 96.83 | -1.59 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 100.00 | 90.48 | -9.52 |
| JSON_PRETTY | 100.00 | 92.07 | -7.93 |
| TOON_DEFAULT | 100.00 | 90.48 | -9.52 |
| TOON_KEYFOLD | 96.83 | 88.89 | -7.94 |
| XML_COMPACT | 98.41 | 88.89 | -9.52 |
| XML_PRETTY | 98.41 | 90.48 | -7.93 |
| YAML | 96.83 | 88.89 | -7.94 |

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-29
- **Model**: Sonnet 4.6
- **Thinking**: off
- **Structure**: nested
- **Formats Tested**: JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, TOON_KEYFOLD, XML_COMPACT, XML_PRETTY, YAML
- **Record Counts**: 31
- **Total Test Cases**: 14

### 4.2 Appendix B: Benchmark Configuration
- **Field Retrieval**: 55 questions (37.50% weight)
- **Filtering**: 21 questions (20.83% weight)
- **Aggregation**: 21 questions (12.50% weight)
- **Structure Awareness**: 27 questions (29.17% weight)

---

- **Report Generated**: 2026-04-01
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code < 2.1.86
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)