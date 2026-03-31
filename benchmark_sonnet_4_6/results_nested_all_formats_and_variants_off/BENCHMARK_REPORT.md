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
- **Read Tokens**: For each data file a single read subagent is invoked with the only prompt to read the file at the provided filepath and return "Done" once finished and do nothing more. The tokens extraction script searches for the read tool use result and extracted the tokens for that action from it.
- **Output Tokens**: For each data file a three full tests subagent are invoked with all necessary files and the test setup and the instruction to write a file once with the answers. The token extraction script searches for the write tool use result and extracted the tokens for that action from it.

## 2. Results

### 2.1 TLDR: Token Efficiency Analysis

*Note: All columns ranked best-to-worst. ↑ = lower value is better (ascending). ↓ = higher value is better (descending).*

#### 2.1.1 Best results

- Lowest total token cost:
   - Optional: JSON_COMPACT 7266 tokens
   - Mandatory: JSON_COMPACT 7792 tokens
- Lowest output token cost drift:
   - Optional: XML_PRETTY ↓ 0.00% ↑ 0.00%
   - Mandatory: TOON_KEYFOLD ↓ -0.22% ↑ 0.11%
- Highest accuracy:
   - Optional: JSON_PRETTY 98.66%
   - Mandatory: TOON_DEFAULT 99.19%
- Lowest accuracy drift:
   - Optional: JSON_PRETTY ↓ -0.27% ↑ 0.54%
   - Mandatory: XML_COMPACT ↓ -0.81% ↑ 0.82%
- Most useful tokens:
   - Optional: JSON_PRETTY 17744 / 17985 tokens
   - Mandatory: XML_PRETTY 18242 / 18591 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 98.47
   - Mandatory: JSON_COMPACT 97.64
- Lowest delta (optional-mandatory):
   - Total tokens: TOON_DEFAULT -150 tokens
   - Accuracy: JSON_COMPACT -0.80%
   - Token efficiency: TOON_KEYFOLD 0.34

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 18061 tokens
   - Mandatory: XML_PRETTY 18591 tokens
- Highest output token drift:
   - Optional: TOON_KEYFOLD ↓ -93.80% ↑ 46.90%
   - Mandatory: TOON_DEFAULT ↓ -95.61% ↑ 48.29%
- Lowest accuracy:
   - Optional: XML_PRETTY 96.78%
   - Mandatory: JSON_PRETTY 96.50%
- Highest accuracy drift:
   - Optional: TOON_DEFAULT ↓ -1.92% ↑ 1.37%
   - Mandatory: TOON_KEYFOLD ↓ -1.92% ↑ 2.20%
- Most wasted tokens:
   - Optional: XML_PRETTY 582 / 18061 tokens
   - Mandatory: JSON_PRETTY 563 / 16084 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 69.17
   - Mandatory: XML_PRETTY 68.71
- Highest delta (optional-mandatory):
   - Total tokens: XML_COMPACT 2232 tokens
   - Accuracy: JSON_PRETTY 2.16%
   - Token efficiency: XML_COMPACT -6.65

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 292s | JSON_COMPACT ≈ 7792 | XML_COMPACT ≈ 85 | TOON_DEFAULT ≈ 99% | XML_COMPACT ≈ 99% | JSON_COMPACT ≈ 98 | JSON_COMPACT ≈ 97 |
| XML_PRETTY (+5.7%) | XML_COMPACT (+34.3%) | TOON_DEFAULT (+11.3%) | XML_COMPACT (0.0%) | TOON_DEFAULT (-0.1%) | XML_COMPACT (-6.9%) | XML_COMPACT (-6.7%) |
| YAML (+8.0%) | TOON_KEYFOLD (+48.8%) | JSON_COMPACT (+24.1%) | YAML (-0.3%) | YAML (-0.2%) | TOON_DEFAULT (-10.0%) | TOON_DEFAULT (-10.0%) |
| TOON_DEFAULT (+12.3%) | TOON_DEFAULT (+49.4%) | YAML (+52.2%) | JSON_COMPACT (-0.5%) | JSON_COMPACT (-0.7%) | TOON_KEYFOLD (-10.9%) | YAML (-10.9%) |
| XML_COMPACT (+15.4%) | YAML (+53.3%) | TOON_KEYFOLD (+194.1%) | XML_PRETTY (-1.1%) | XML_PRETTY (-1.3%) | YAML (-11.1%) | TOON_KEYFOLD (-11.0%) |
| TOON_KEYFOLD (+16.2%) | JSON_PRETTY (+106.4%) | XML_PRETTY (+312.3%) | TOON_KEYFOLD (-1.3%) | TOON_KEYFOLD (-1.6%) | JSON_PRETTY (-24.0%) | JSON_PRETTY (-24.5%) |
| JSON_COMPACT (+24.3%) | XML_PRETTY (+138.6%) | JSON_PRETTY (+564.0%) | JSON_PRETTY (-2.7%) | JSON_PRETTY (-3.5%) | XML_PRETTY (-29.6%) | XML_PRETTY (-29.7%) |


##### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| YAML ≈ 323s | JSON_COMPACT ≈ 7266 | JSON_COMPACT ≈ 156 | JSON_PRETTY ≈ 99% | JSON_PRETTY ≈ 99% | JSON_COMPACT ≈ 98 | JSON_COMPACT ≈ 99 |
| JSON_PRETTY (+0.3%) | TOON_KEYFOLD (+54.9%) | XML_COMPACT (+52.8%) | XML_COMPACT (-0.5%) | XML_COMPACT (-0.4%) | TOON_KEYFOLD (-11.3%) | TOON_KEYFOLD (-11.3%) |
| TOON_KEYFOLD (+0.3%) | TOON_DEFAULT (+58.2%) | JSON_PRETTY (+54.3%) | JSON_COMPACT (-0.8%) | JSON_COMPACT (-0.9%) | TOON_DEFAULT (-11.4%) | TOON_DEFAULT (-11.3%) |
| TOON_DEFAULT (+1.9%) | YAML (+61.0%) | TOON_DEFAULT (+58.2%) | TOON_DEFAULT (-0.8%) | TOON_DEFAULT (-0.9%) | YAML (-12.3%) | YAML (-12.2%) |
| JSON_COMPACT (+2.3%) | XML_COMPACT (+74.8%) | YAML (+101.4%) | YAML (-1.3%) | YAML (-1.4%) | XML_COMPACT (-14.4%) | XML_COMPACT (-14.2%) |
| XML_COMPACT (+2.3%) | JSON_PRETTY (+147.5%) | TOON_KEYFOLD (+113.2%) | TOON_KEYFOLD (-1.6%) | TOON_KEYFOLD (-1.7%) | JSON_PRETTY (-28.2%) | JSON_PRETTY (-28.1%) |
| XML_PRETTY (+4.6%) | XML_PRETTY (+148.6%) | XML_PRETTY (+272.3%) | XML_PRETTY (-1.9%) | XML_PRETTY (-2.2%) | XML_PRETTY (-29.7%) | XML_PRETTY (-29.8%) |


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
| JSON_COMPACT | man | 7498 | 294 | 7792 | 3.190 | 1.266 | 2.374 | 98.65 | 97.47 | 7687.137 | 105.196 | 97.64 | 97.47 |
| JSON_COMPACT | opt | 6970 | 296 | 7266 | 3.226 | 1.347 | 2.384 | 97.85 | 98.64 | 7109.455 | 156.212 | 98.47 | 98.64 |
| JSON_PRETTY | man | 15787 | 297 | 16084 | 2.094 | 0.600 | 2.392 | 96.50 | 73.61 | 15520.739 | 562.928 | 74.21 | 73.61 |
| JSON_PRETTY | opt | 17688 | 297 | 17985 | 1.768 | 0.549 | 2.395 | 98.66 | 70.94 | 17744.001 | 240.999 | 70.69 | 70.94 |
| TOON_DEFAULT | man | 11440 | 205 | 11645 | 2.370 | 0.852 | 1.653 | 99.19 | 87.70 | 11550.675 | 94.324 | 87.83 | 87.70 |
| TOON_DEFAULT | opt | 11195 | 300 | 11495 | 2.394 | 0.851 | 2.422 | 97.85 | 87.45 | 11248.183 | 247.150 | 87.28 | 87.45 |
| TOON_KEYFOLD | man | 11289 | 308 | 11597 | 2.382 | 0.844 | 2.481 | 97.85 | 86.79 | 11347.339 | 249.328 | 87.02 | 86.79 |
| TOON_KEYFOLD | opt | 11044 | 210 | 11254 | 2.407 | 0.862 | 1.691 | 97.04 | 87.53 | 10920.558 | 333.109 | 87.36 | 87.53 |
| XML_COMPACT | man | 10164 | 302 | 10466 | 3.371 | 0.948 | 2.433 | 99.19 | 90.91 | 10380.895 | 84.772 | 90.94 | 90.91 |
| XML_COMPACT | opt | 12494 | 204 | 12698 | 2.641 | 0.773 | 1.642 | 98.12 | 84.64 | 12458.951 | 238.716 | 84.29 | 84.64 |
| XML_PRETTY | man | 18291 | 300 | 18591 | 2.321 | 0.528 | 2.422 | 98.12 | 68.50 | 18241.816 | 349.517 | 68.71 | 68.50 |
| XML_PRETTY | opt | 17760 | 301 | 18061 | 2.319 | 0.536 | 2.427 | 96.78 | 69.21 | 17479.436 | 581.564 | 69.17 | 69.21 |
| YAML | man | 11649 | 295 | 11944 | 2.280 | 0.828 | 2.379 | 98.92 | 86.84 | 11815.005 | 128.995 | 86.85 | 86.84 |
| YAML | opt | 11404 | 293 | 11697 | 2.301 | 0.832 | 2.363 | 97.31 | 86.57 | 11382.351 | 314.649 | 86.37 | 86.57 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Accuracy Man (%) | Accuracy Opt (%) | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7792 | 7265 | -527 | -6.76 | 98.65 | 97.85 | -0.80 | 98.42 | 98.09 | -0.33 | 97.64 | 98.47 |  +0.83 | 97.47 | 98.64 |  +1.16 |
| JSON_PRETTY | 16084 | 17985 |  +1901 |  +11.82 | 96.50 | 98.66 |  +2.16 | 95.65 | 99.01 |  +3.36 | 74.21 | 70.69 | -3.52 | 73.61 | 70.94 | -2.67 |
| TOON_DEFAULT | 11645 | 11495 | -150 | -1.29 | 99.19 | 97.85 | -1.34 | 99.01 | 98.09 | -0.92 | 87.83 | 87.28 | -0.54 | 87.70 | 87.45 | -0.25 |
| TOON_KEYFOLD | 11597 | 11254 | -343 | -2.96 | 97.85 | 97.04 | -0.81 | 97.53 | 97.29 | -0.24 | 87.02 | 87.36 |  +0.34 | 86.79 | 87.53 |  +0.74 |
| XML_COMPACT | 10466 | 12698 |  +2232 |  +21.33 | 99.19 | 98.12 | -1.07 | 99.14 | 98.61 | -0.53 | 90.94 | 84.29 | -6.65 | 90.91 | 84.64 | -6.27 |
| XML_PRETTY | 18591 | 18061 | -530 | -2.85 | 98.12 | 96.78 | -1.34 | 97.82 | 96.83 | -0.99 | 68.71 | 69.17 |  +0.47 | 68.50 | 69.21 |  +0.71 |
| YAML | 11944 | 11697 | -247 | -2.07 | 98.92 | 97.31 | -1.61 | 98.91 | 97.59 | -1.32 | 86.85 | 86.37 | -0.47 | 86.84 | 86.57 | -0.27 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 35 | 214.229 | 1.13 | 363175 | 0.001 | 2928.83 | 363210 | 214.230 | 2343.29 |
| JSON_COMPACT | opt | 19 | 366.842 | 0.61 | 330751 | 0.001 | 2667.34 | 330770 | 366.843 | 2134.00 |
| JSON_PRETTY | man | 319 | 49.489 | 10.29 | 291933 | 0.001 | 2354.30 | 292252 | 49.490 | 1885.50 |
| JSON_PRETTY | opt | 331 | 53.438 | 10.68 | 324103 | 0.001 | 2613.73 | 324434 | 53.439 | 2093.12 |
| TOON_DEFAULT | man | 54 | 211.852 | 1.74 | 328166 | 0.001 | 2646.50 | 328220 | 211.853 | 2117.55 |
| TOON_DEFAULT | opt | 40 | 279.875 | 1.29 | 329622 | 0.001 | 2658.24 | 329662 | 279.876 | 2126.85 |
| TOON_KEYFOLD | man | 35 | 322.543 | 1.13 | 339495 | 0.001 | 2737.86 | 339530 | 322.544 | 2190.52 |
| TOON_KEYFOLD | opt | 41 | 269.366 | 1.32 | 324478 | 0.001 | 2616.76 | 324519 | 269.367 | 2093.67 |
| XML_COMPACT | man | 110 | 92.400 | 3.55 | 337256 | 0.001 | 2719.81 | 337366 | 92.401 | 2176.55 |
| XML_COMPACT | opt | 116 | 107.707 | 3.74 | 330916 | 0.001 | 2668.68 | 331032 | 107.708 | 2135.69 |
| XML_PRETTY | man | 37 | 494.351 | 1.19 | 308772 | 0.001 | 2490.10 | 308809 | 494.352 | 1992.32 |
| XML_PRETTY | opt | 42 | 422.857 | 1.35 | 338337 | 0.001 | 2728.52 | 338379 | 422.858 | 2183.09 |
| YAML | man | 85 | 137.047 | 2.74 | 315630 | 0.001 | 2545.40 | 315715 | 137.048 | 2036.87 |
| YAML | opt | 81 | 140.790 | 2.61 | 323407 | 0.001 | 2608.12 | 323488 | 140.791 | 2087.02 |

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
| JSON_COMPACT | man | 3.190 | 10.994 | 241.871 | 1.266 |
| JSON_COMPACT | opt | 3.226 | 11.046 | 224.839 | 1.347 |
| JSON_PRETTY | man | 2.094 | 23.148 | 509.258 | 0.600 |
| JSON_PRETTY | opt | 1.768 | 28.032 | 570.581 | 0.549 |
| TOON_DEFAULT | man | 2.370 | 16.774 | 369.032 | 0.852 |
| TOON_DEFAULT | opt | 2.394 | 17.742 | 361.129 | 0.851 |
| TOON_KEYFOLD | man | 2.382 | 16.553 | 364.161 | 0.844 |
| TOON_KEYFOLD | opt | 2.407 | 17.502 | 356.258 | 0.862 |
| XML_COMPACT | man | 3.371 | 14.903 | 327.871 | 0.948 |
| XML_COMPACT | opt | 2.641 | 19.800 | 403.032 | 0.773 |
| XML_PRETTY | man | 2.321 | 26.820 | 590.032 | 0.528 |
| XML_PRETTY | opt | 2.319 | 28.146 | 572.903 | 0.536 |
| YAML | man | 2.280 | 17.081 | 375.774 | 0.828 |
| YAML | opt | 2.301 | 18.073 | 367.871 | 0.832 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 3.190 | 3.226 |  +0.036 |  +1.13 | 10.994 | 11.046 |  +0.052 |  +0.47 | 241.871 | 224.839 | -17.032 | -7.04 | 1.266 | 1.347 |  +0.081 |  +6.40 |
| JSON_PRETTY | 2.094 | 1.768 | -0.326 | -15.57 | 23.148 | 28.032 |  +4.884 |  +21.10 | 509.258 | 570.581 |  +61.323 |  +12.04 | 0.600 | 0.549 | -0.051 | -8.50 |
| TOON_DEFAULT | 2.370 | 2.394 |  +0.024 |  +1.01 | 16.774 | 17.742 |  +0.968 |  +5.77 | 369.032 | 361.129 | -7.903 | -2.14 | 0.852 | 0.851 | -0.001 | -0.12 |
| TOON_KEYFOLD | 2.382 | 2.407 |  +0.025 |  +1.05 | 16.553 | 17.502 |  +0.949 |  +5.73 | 364.161 | 356.258 | -7.903 | -2.17 | 0.844 | 0.862 |  +0.018 |  +2.13 |
| XML_COMPACT | 3.371 | 2.641 | -0.730 | -21.66 | 14.903 | 19.800 |  +4.897 |  +32.86 | 327.871 | 403.032 |  +75.161 |  +22.92 | 0.948 | 0.773 | -0.175 | -18.46 |
| XML_PRETTY | 2.321 | 2.319 | -0.002 | -0.09 | 26.820 | 28.146 |  +1.326 |  +4.94 | 590.032 | 572.903 | -17.129 | -2.90 | 0.528 | 0.536 |  +0.008 |  +1.52 |
| YAML | 2.280 | 2.301 |  +0.021 |  +0.92 | 17.081 | 18.073 |  +0.992 |  +5.81 | 375.774 | 367.871 | -7.903 | -2.10 | 0.828 | 0.832 |  +0.004 |  +0.48 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 7792 | 7687 | 105 | 98.65 | 98.42 | 97.64 | 97.47 |
| JSON_COMPACT | opt | 7266 | 7109 | 156 | 97.85 | 98.09 | 98.47 | 98.64 |
| JSON_PRETTY | man | 16084 | 15521 | 563 | 96.50 | 95.65 | 74.21 | 73.61 |
| JSON_PRETTY | opt | 17985 | 17744 | 241 | 98.66 | 99.01 | 70.69 | 70.94 |
| TOON_DEFAULT | man | 11645 | 11551 | 94 | 99.19 | 99.01 | 87.83 | 87.70 |
| TOON_DEFAULT | opt | 11495 | 11248 | 247 | 97.85 | 98.09 | 87.28 | 87.45 |
| TOON_KEYFOLD | man | 11597 | 11347 | 249 | 97.85 | 97.53 | 87.02 | 86.79 |
| TOON_KEYFOLD | opt | 11254 | 10921 | 333 | 97.04 | 97.29 | 87.36 | 87.53 |
| XML_COMPACT | man | 10466 | 10381 | 85 | 99.19 | 99.14 | 90.94 | 90.91 |
| XML_COMPACT | opt | 12698 | 12459 | 239 | 98.12 | 98.61 | 84.29 | 84.64 |
| XML_PRETTY | man | 18591 | 18242 | 350 | 98.12 | 97.82 | 68.71 | 68.50 |
| XML_PRETTY | opt | 18061 | 17479 | 582 | 96.78 | 96.83 | 69.17 | 69.21 |
| YAML | man | 11944 | 11815 | 129 | 98.92 | 98.91 | 86.85 | 86.84 |
| YAML | opt | 11697 | 11382 | 315 | 97.31 | 97.59 | 86.37 | 86.57 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7792 | 7265 | -527 | -6.76 | 7687 | 7109 | -578 | -7.52 | 105 | 156 |  +51 |  +48.59 | 98.65 | 97.85 | -0.80 | 97.64 | 98.469 |  +0.83 |  +0.85 |
| JSON_PRETTY | 16084 | 17985 |  +1901 |  +11.82 | 15521 | 17744 |  +2223 |  +14.32 | 563 | 241 | -322 | -57.18 | 96.50 | 98.66 |  +2.16 | 74.21 | 70.692 | -3.52 | -4.74 |
| TOON_DEFAULT | 11645 | 11495 | -150 | -1.29 | 11551 | 11249 | -302 | -2.62 | 94 | 247 |  +153 |  +162.58 | 99.19 | 97.85 | -1.34 | 87.83 | 87.285 | -0.54 | -0.62 |
| TOON_KEYFOLD | 11597 | 11254 | -343 | -2.96 | 11347 | 10920 | -427 | -3.76 | 249 | 333 |  +84 |  +33.65 | 97.85 | 97.04 | -0.81 | 87.02 | 87.357 |  +0.34 |  +0.39 |
| XML_COMPACT | 10466 | 12698 |  +2232 |  +21.33 | 10381 | 12459 |  +2078 |  +20.02 | 85 | 239 |  +154 |  +181.11 | 99.19 | 98.12 | -1.07 | 90.94 | 84.294 | -6.65 | -7.31 |
| XML_PRETTY | 18591 | 18061 | -530 | -2.85 | 18242 | 17480 | -762 | -4.18 | 350 | 582 |  +232 |  +66.30 | 98.12 | 96.78 | -1.34 | 68.71 | 69.175 |  +0.47 |  +0.68 |
| YAML | 11944 | 11697 | -247 | -2.07 | 11815 | 11382 | -433 | -3.66 | 129 | 315 |  +186 |  +143.92 | 98.92 | 97.31 | -1.61 | 86.85 | 86.373 | -0.47 | -0.55 |

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

## 3. Format-Specific Analysis
### 3.1 Detailed Analysis: JSON_COMPACT

#### 3.1.1 Performance Summary

- Token Duration Range: 331 - 363 seconds
- Token Cost Range: 7266 - 7792 tokens
- Wasted Token Range: 105 - 156 tokens
- Accuracy Range: 97.85 - 98.65%
- Efficiency Score Range: 97.64 - 98.47

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

- Token Duration Range: 292 - 324 seconds
- Token Cost Range: 16084 - 17985 tokens
- Wasted Token Range: 241 - 563 tokens
- Accuracy Range: 96.50 - 98.66%
- Efficiency Score Range: 70.69 - 74.21

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

- Token Duration Range: 328 - 330 seconds
- Token Cost Range: 11495 - 11645 tokens
- Wasted Token Range: 94 - 247 tokens
- Accuracy Range: 97.85 - 99.19%
- Efficiency Score Range: 87.28 - 87.83

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

### 3.4 Detailed Analysis: TOON_KEYFOLD

#### 3.4.1 Performance Summary

- Token Duration Range: 325 - 340 seconds
- Token Cost Range: 11254 - 11597 tokens
- Wasted Token Range: 249 - 333 tokens
- Accuracy Range: 97.04 - 97.85%
- Efficiency Score Range: 87.02 - 87.36

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

- Token Duration Range: 331 - 337 seconds
- Token Cost Range: 10466 - 12698 tokens
- Wasted Token Range: 85 - 239 tokens
- Accuracy Range: 98.12 - 99.19%
- Efficiency Score Range: 84.29 - 90.94

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

- Token Duration Range: 309 - 338 seconds
- Token Cost Range: 18061 - 18591 tokens
- Wasted Token Range: 350 - 582 tokens
- Accuracy Range: 96.78 - 98.12%
- Efficiency Score Range: 68.71 - 69.17

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

- Token Duration Range: 316 - 323 seconds
- Token Cost Range: 11697 - 11944 tokens
- Wasted Token Range: 129 - 315 tokens
- Accuracy Range: 97.31 - 98.92%
- Efficiency Score Range: 86.37 - 86.85

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

- **Report Generated**: 2026-03-31
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code < 2.1.86
   - Note: All tests run with the same read tool overhead (line number format + system reminder)
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)