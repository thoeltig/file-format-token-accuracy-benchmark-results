# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Sonnet 4.6
- **Thinking**: on (medium)
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
   - Mandatory: JSON_COMPACT 7702 tokens
- Lowest output token cost drift:
   - Optional: TOON_DEFAULT ↓ -0.22% ↑ 0.11%
   - Mandatory: XML_PRETTY ↓ -0.23% ↑ 0.46%
- Highest accuracy:
   - Optional: TOON_DEFAULT 98.39%
   - Mandatory: TOON_DEFAULT 99.73%
- Lowest accuracy drift:
   - Optional: TOON_DEFAULT ↓ 0.00% ↑ 0.00%
   - Mandatory: TOON_DEFAULT ↓ -0.54% ↑ 0.27%
- Most useful tokens:
   - Optional: XML_PRETTY 17712 / 18051 tokens
   - Mandatory: XML_PRETTY 18132 / 18582 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 98.47
   - Mandatory: JSON_COMPACT 97.88
- Lowest delta (optional-mandatory):
   - Total tokens: TOON_KEYFOLD -147 tokens
   - Accuracy: TOON_KEYFOLD -0.27%
   - Token efficiency: TOON_KEYFOLD 0.20

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 18051 tokens
   - Mandatory: XML_PRETTY 18582 tokens
- Highest output token drift:
   - Optional: JSON_PRETTY ↓ -95.02% ↑ 48.50%
   - Mandatory: XML_COMPACT ↓ -97.48% ↑ 48.74%
- Lowest accuracy:
   - Optional: JSON_PRETTY 97.58%
   - Mandatory: XML_PRETTY 97.58%
- Highest accuracy drift:
   - Optional: JSON_PRETTY ↓ -1.65% ↑ 0.83%
   - Mandatory: XML_COMPACT ↓ -3.02% ↑ 1.92%
- Most wasted tokens:
   - Optional: JSON_PRETTY 364 / 15059 tokens
   - Mandatory: XML_PRETTY 450 / 18582 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 70.11
   - Mandatory: XML_PRETTY 68.33
- Highest delta (optional-mandatory):
   - Total tokens: YAML -2974 tokens
   - Accuracy: YAML -1.88%
   - Token efficiency: YAML 6.55

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 263s | JSON_COMPACT ≈ 7702 | TOON_DEFAULT ≈ 32 | TOON_DEFAULT ≈ 100% | YAML ≈ 100% | JSON_COMPACT ≈ 98 | JSON_COMPACT ≈ 98 |
| TOON_KEYFOLD (+7.7%) | XML_COMPACT (+34.5%) | YAML (+25.0%) | YAML (0.0%) | TOON_DEFAULT (-0.1%) | XML_COMPACT (-7.6%) | XML_COMPACT (-7.6%) |
| JSON_PRETTY (+12.5%) | TOON_KEYFOLD (+49.3%) | JSON_COMPACT (+228.0%) | JSON_COMPACT (-1.1%) | JSON_COMPACT (-1.2%) | TOON_DEFAULT (-10.1%) | TOON_DEFAULT (-10.2%) |
| XML_COMPACT (+14.4%) | TOON_DEFAULT (+52.4%) | TOON_KEYFOLD (+483.8%) | TOON_KEYFOLD (-1.3%) | TOON_KEYFOLD (-1.7%) | TOON_KEYFOLD (-10.4%) | TOON_KEYFOLD (-10.6%) |
| YAML (+19.8%) | YAML (+90.5%) | XML_COMPACT (+514.5%) | JSON_PRETTY (-1.6%) | XML_COMPACT (-1.8%) | YAML (-18.1%) | YAML (-18.0%) |
| XML_PRETTY (+23.0%) | JSON_PRETTY (+108.8%) | JSON_PRETTY (+853.8%) | XML_COMPACT (-1.6%) | JSON_PRETTY (-1.8%) | JSON_PRETTY (-23.0%) | JSON_PRETTY (-23.2%) |
| JSON_COMPACT (+23.1%) | XML_PRETTY (+141.3%) | XML_PRETTY (+1318.5%) | XML_PRETTY (-2.2%) | XML_PRETTY (-2.6%) | XML_PRETTY (-30.2%) | XML_PRETTY (-30.5%) |


##### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 279s | JSON_COMPACT ≈ 7266 | JSON_COMPACT ≈ 156 | TOON_DEFAULT ≈ 98% | TOON_DEFAULT ≈ 99% | JSON_COMPACT ≈ 98 | JSON_COMPACT ≈ 99 |
| JSON_COMPACT (+9.8%) | XML_COMPACT (+37.4%) | TOON_DEFAULT (+18.4%) | TOON_KEYFOLD (-0.3%) | TOON_KEYFOLD (-0.3%) | XML_COMPACT (-7.3%) | XML_COMPACT (-7.3%) |
| TOON_KEYFOLD (+10.9%) | TOON_KEYFOLD (+56.2%) | TOON_KEYFOLD (+36.6%) | XML_PRETTY (-0.3%) | YAML (-0.4%) | TOON_KEYFOLD (-10.8%) | TOON_KEYFOLD (-10.7%) |
| JSON_PRETTY (+11.9%) | TOON_DEFAULT (+58.2%) | XML_COMPACT (+37.4%) | JSON_COMPACT (-0.5%) | XML_PRETTY (-0.5%) | TOON_DEFAULT (-11.0%) | TOON_DEFAULT (-10.8%) |
| YAML (+12.3%) | YAML (+61.0%) | YAML (+61.0%) | XML_COMPACT (-0.5%) | XML_COMPACT (-0.7%) | YAML (-11.9%) | YAML (-11.7%) |
| XML_PRETTY (+16.0%) | JSON_PRETTY (+107.2%) | XML_PRETTY (+117.2%) | YAML (-0.5%) | JSON_COMPACT (-0.7%) | JSON_PRETTY (-21.1%) | JSON_PRETTY (-21.1%) |
| XML_COMPACT (+18.0%) | XML_PRETTY (+148.4%) | JSON_PRETTY (+133.3%) | JSON_PRETTY (-0.8%) | JSON_PRETTY (-1.0%) | XML_PRETTY (-28.8%) | XML_PRETTY (-28.8%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 100% | JSON_COMPACT ≈ 100% | YAML ≈ 100% | TOON_DEFAULT ≈ 100% |
| JSON_PRETTY (0.0%) | TOON_DEFAULT (0.0%) | TOON_DEFAULT (-1.6%) | XML_PRETTY (0.0%) |
| TOON_DEFAULT (0.0%) | XML_PRETTY (0.0%) | JSON_COMPACT (-4.8%) | YAML (0.0%) |
| TOON_KEYFOLD (0.0%) | YAML (0.0%) | TOON_KEYFOLD (-4.8%) | TOON_KEYFOLD (-1.6%) |
| XML_PRETTY (-0.6%) | JSON_PRETTY (-1.2%) | XML_COMPACT (-4.8%) | XML_COMPACT (-1.6%) |
| YAML (-0.6%) | XML_COMPACT (-1.2%) | JSON_PRETTY (-6.3%) | JSON_COMPACT (-3.2%) |
| XML_COMPACT (-1.2%) | TOON_KEYFOLD (-2.5%) | XML_PRETTY (-12.7%) | JSON_PRETTY (-3.2%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 100% | JSON_PRETTY ≈ 100% | TOON_DEFAULT ≈ 100% | XML_PRETTY ≈ 92% |
| JSON_PRETTY (0.0%) | TOON_DEFAULT (0.0%) | YAML (0.0%) | JSON_COMPACT (-1.6%) |
| TOON_DEFAULT (0.0%) | TOON_KEYFOLD (0.0%) | JSON_COMPACT (-1.6%) | JSON_PRETTY (-1.6%) |
| TOON_KEYFOLD (0.0%) | XML_COMPACT (0.0%) | TOON_KEYFOLD (-1.6%) | TOON_DEFAULT (-1.6%) |
| XML_COMPACT (0.0%) | XML_PRETTY (0.0%) | XML_COMPACT (-3.2%) | TOON_KEYFOLD (-1.6%) |
| XML_PRETTY (0.0%) | YAML (0.0%) | XML_PRETTY (-3.2%) | XML_COMPACT (-1.6%) |
| YAML (0.0%) | JSON_COMPACT (-1.2%) | JSON_PRETTY (-4.8%) | YAML (-4.8%) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Char/Token | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 7498 | 204 | 7702 | 3.190 | 1.281 | 1.645 | 98.65 | 97.85 | 7598.023 | 103.977 | 97.88 | 97.85 |
| JSON_COMPACT | opt | 6970 | 296 | 7266 | 3.226 | 1.347 | 2.387 | 97.85 | 98.66 | 7109.781 | 156.219 | 98.47 | 98.66 |
| JSON_PRETTY | man | 15787 | 296 | 16083 | 2.094 | 0.610 | 2.390 | 98.12 | 75.18 | 15780.966 | 302.367 | 75.32 | 75.18 |
| JSON_PRETTY | opt | 14858 | 201 | 15059 | 2.105 | 0.648 | 1.618 | 97.58 | 77.82 | 14694.247 | 364.420 | 77.66 | 77.82 |
| TOON_DEFAULT | man | 11440 | 301 | 11741 | 2.370 | 0.849 | 2.425 | 99.73 | 87.90 | 11708.967 | 31.700 | 87.94 | 87.90 |
| TOON_DEFAULT | opt | 11195 | 298 | 11493 | 2.394 | 0.856 | 2.401 | 98.39 | 87.95 | 11307.635 | 185.032 | 87.66 | 87.95 |
| TOON_KEYFOLD | man | 11289 | 206 | 11495 | 2.382 | 0.856 | 1.664 | 98.39 | 87.44 | 11310.258 | 185.075 | 87.65 | 87.44 |
| TOON_KEYFOLD | opt | 11044 | 304 | 11348 | 2.407 | 0.865 | 2.452 | 98.12 | 88.11 | 11134.658 | 213.342 | 87.85 | 88.11 |
| XML_COMPACT | man | 10164 | 198 | 10362 | 3.371 | 0.947 | 1.599 | 98.12 | 90.38 | 10167.521 | 194.812 | 90.46 | 90.38 |
| XML_COMPACT | opt | 9684 | 297 | 9981 | 3.407 | 0.980 | 2.395 | 97.85 | 91.49 | 9766.408 | 214.592 | 91.28 | 91.49 |
| XML_PRETTY | man | 18291 | 291 | 18582 | 2.321 | 0.525 | 2.344 | 97.58 | 68.02 | 18131.991 | 449.676 | 68.33 | 68.02 |
| XML_PRETTY | opt | 17760 | 291 | 18051 | 2.319 | 0.544 | 2.349 | 98.12 | 70.28 | 17711.968 | 339.365 | 70.11 | 70.28 |
| YAML | man | 14475 | 198 | 14673 | 1.835 | 0.680 | 1.599 | 99.73 | 80.21 | 14633.715 | 39.618 | 80.18 | 80.21 |
| YAML | opt | 11404 | 296 | 11700 | 2.301 | 0.836 | 2.384 | 97.85 | 87.13 | 11448.124 | 251.543 | 86.73 | 87.13 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Accuracy Man (%) | Accuracy Opt (%) | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7702 | 7266 | -436 | -5.66 | 98.65 | 97.85 | -0.80 | 98.61 | 98.12 | -0.49 | 97.88 | 98.47 |  +0.59 | 97.85 | 98.66 |  +0.81 |
| JSON_PRETTY | 16083 | 15058 | -1025 | -6.37 | 98.12 | 97.58 | -0.54 | 97.92 | 97.82 | -0.10 | 75.32 | 77.66 |  +2.33 | 75.18 | 77.82 |  +2.64 |
| TOON_DEFAULT | 11741 | 11493 | -248 | -2.11 | 99.73 | 98.39 | -1.34 | 99.67 | 98.81 | -0.86 | 87.94 | 87.66 | -0.28 | 87.90 | 87.95 |  +0.05 |
| TOON_KEYFOLD | 11495 | 11348 | -147 | -1.28 | 98.39 | 98.12 | -0.27 | 98.09 | 98.48 |  +0.39 | 87.65 | 87.85 |  +0.20 | 87.44 | 88.11 |  +0.66 |
| XML_COMPACT | 10362 | 9981 | -381 | -3.68 | 98.12 | 97.85 | -0.27 | 98.00 | 98.15 |  +0.15 | 90.46 | 91.28 |  +0.82 | 90.38 | 91.49 |  +1.11 |
| XML_PRETTY | 18582 | 18052 | -530 | -2.85 | 97.58 | 98.12 |  +0.54 | 97.13 | 98.35 |  +1.22 | 68.33 | 70.11 |  +1.78 | 68.02 | 70.28 |  +2.26 |
| YAML | 14673 | 11699 | -2974 | -20.27 | 99.73 | 97.85 | -1.88 | 99.77 | 98.41 | -1.36 | 80.18 | 86.73 |  +6.55 | 80.21 | 87.13 |  +6.92 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 22 | 340.818 | 0.71 | 323514 | 0.001 | 2608.98 | 323536 | 340.819 | 2087.33 |
| JSON_COMPACT | opt | 20 | 348.500 | 0.65 | 306728 | 0.001 | 2473.61 | 306748 | 348.501 | 1979.02 |
| JSON_PRETTY | man | 311 | 50.762 | 10.03 | 295508 | 0.001 | 2383.13 | 295819 | 50.763 | 1908.51 |
| JSON_PRETTY | opt | 366 | 40.596 | 11.81 | 312166 | 0.001 | 2517.47 | 312532 | 40.597 | 2016.34 |
| TOON_DEFAULT | man | 36 | 317.778 | 1.16 | 262869 | 0.001 | 2119.91 | 262905 | 317.779 | 1696.16 |
| TOON_DEFAULT | opt | 30 | 373.167 | 0.97 | 279384 | 0.001 | 2253.10 | 279414 | 373.168 | 1802.67 |
| TOON_KEYFOLD | man | 12 | 940.750 | 0.39 | 283145 | 0.001 | 2283.42 | 283157 | 940.751 | 1826.82 |
| TOON_KEYFOLD | opt | 30 | 368.133 | 0.97 | 309913 | 0.001 | 2499.30 | 309943 | 368.134 | 1999.63 |
| XML_COMPACT | man | 35 | 290.400 | 1.13 | 300623 | 0.001 | 2424.38 | 300658 | 290.401 | 1939.73 |
| XML_COMPACT | opt | 10 | 968.400 | 0.32 | 329615 | 0.001 | 2658.18 | 329625 | 968.401 | 2126.61 |
| XML_PRETTY | man | 29 | 630.724 | 0.94 | 323223 | 0.001 | 2606.63 | 323252 | 630.725 | 2085.49 |
| XML_PRETTY | opt | 30 | 592.000 | 0.97 | 324069 | 0.001 | 2613.46 | 324099 | 592.001 | 2090.96 |
| YAML | man | 28 | 516.964 | 0.90 | 314910 | 0.001 | 2539.59 | 314938 | 516.965 | 2031.86 |
| YAML | opt | 44 | 259.182 | 1.42 | 313627 | 0.001 | 2529.25 | 313671 | 259.183 | 2023.68 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) | Total Man (s) | Total Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 22 | 20 | -2 | -9.09 | 323.51 | 306.73 | -16.79 | -5.19 | 323.54 | 306.75 | -16.79 | -5.19 |
| JSON_PRETTY | 311 | 366 |  +55 |  +17.68 | 295.51 | 312.17 |  +16.66 |  +5.64 | 295.82 | 312.53 |  +16.71 |  +5.65 |
| TOON_DEFAULT | 36 | 30 | -6 | -16.67 | 262.87 | 279.38 |  +16.52 |  +6.28 | 262.91 | 279.41 |  +16.51 |  +6.28 |
| TOON_KEYFOLD | 12 | 30 |  +18 |  +150.00 | 283.14 | 309.91 |  +26.77 |  +9.45 | 283.16 | 309.94 |  +26.79 |  +9.46 |
| XML_COMPACT | 35 | 10 | -25 | -71.43 | 300.62 | 329.61 |  +28.99 |  +9.64 | 300.66 | 329.62 |  +28.97 |  +9.63 |
| XML_PRETTY | 29 | 30 |  +1 |  +3.45 | 323.22 | 324.07 |  +0.85 |  +0.26 | 323.25 | 324.10 |  +0.85 |  +0.26 |
| YAML | 28 | 44 |  +16 |  +57.14 | 314.91 | 313.63 | -1.28 | -0.41 | 314.94 | 313.67 | -1.27 | -0.40 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Char/Token | Token/Value | Token/Object | Info/Token |
|---|---|---|---|---|---|
| JSON_COMPACT | man | 3.190 | 10.994 | 241.871 | 1.281 |
| JSON_COMPACT | opt | 3.226 | 11.046 | 224.839 | 1.347 |
| JSON_PRETTY | man | 2.094 | 23.148 | 509.258 | 0.610 |
| JSON_PRETTY | opt | 2.105 | 23.547 | 479.290 | 0.648 |
| TOON_DEFAULT | man | 2.370 | 16.774 | 369.032 | 0.849 |
| TOON_DEFAULT | opt | 2.394 | 17.742 | 361.129 | 0.856 |
| TOON_KEYFOLD | man | 2.382 | 16.553 | 364.161 | 0.856 |
| TOON_KEYFOLD | opt | 2.407 | 17.502 | 356.258 | 0.865 |
| XML_COMPACT | man | 3.371 | 14.903 | 327.871 | 0.947 |
| XML_COMPACT | opt | 3.407 | 15.347 | 312.387 | 0.980 |
| XML_PRETTY | man | 2.321 | 26.820 | 590.032 | 0.525 |
| XML_PRETTY | opt | 2.319 | 28.146 | 572.903 | 0.544 |
| YAML | man | 1.835 | 21.224 | 466.935 | 0.680 |
| YAML | opt | 2.301 | 18.073 | 367.871 | 0.836 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 3.190 | 3.226 |  +0.036 |  +1.13 | 10.994 | 11.046 |  +0.052 |  +0.47 | 241.871 | 224.839 | -17.032 | -7.04 | 1.281 | 1.347 |  +0.066 |  +5.15 |
| JSON_PRETTY | 2.094 | 2.105 |  +0.011 |  +0.53 | 23.148 | 23.547 |  +0.399 |  +1.72 | 509.258 | 479.290 | -29.968 | -5.88 | 0.610 | 0.648 |  +0.038 |  +6.23 |
| TOON_DEFAULT | 2.370 | 2.394 |  +0.024 |  +1.01 | 16.774 | 17.742 |  +0.968 |  +5.77 | 369.032 | 361.129 | -7.903 | -2.14 | 0.849 | 0.856 |  +0.007 |  +0.82 |
| TOON_KEYFOLD | 2.382 | 2.407 |  +0.025 |  +1.05 | 16.553 | 17.502 |  +0.949 |  +5.73 | 364.161 | 356.258 | -7.903 | -2.17 | 0.856 | 0.865 |  +0.009 |  +1.05 |
| XML_COMPACT | 3.371 | 3.407 |  +0.036 |  +1.07 | 14.903 | 15.347 |  +0.444 |  +2.98 | 327.871 | 312.387 | -15.484 | -4.72 | 0.947 | 0.980 |  +0.033 |  +3.48 |
| XML_PRETTY | 2.321 | 2.319 | -0.002 | -0.09 | 26.820 | 28.146 |  +1.326 |  +4.94 | 590.032 | 572.903 | -17.129 | -2.90 | 0.525 | 0.544 |  +0.019 |  +3.62 |
| YAML | 1.835 | 2.301 |  +0.466 |  +25.40 | 21.224 | 18.073 | -3.151 | -14.85 | 466.935 | 367.871 | -99.064 | -21.22 | 0.680 | 0.836 |  +0.156 |  +22.94 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 7702 | 7598 | 104 | 98.65 | 98.61 | 97.88 | 97.85 |
| JSON_COMPACT | opt | 7266 | 7110 | 156 | 97.85 | 98.12 | 98.47 | 98.66 |
| JSON_PRETTY | man | 16083 | 15781 | 302 | 98.12 | 97.92 | 75.32 | 75.18 |
| JSON_PRETTY | opt | 15059 | 14694 | 364 | 97.58 | 97.82 | 77.66 | 77.82 |
| TOON_DEFAULT | man | 11741 | 11709 | 32 | 99.73 | 99.67 | 87.94 | 87.90 |
| TOON_DEFAULT | opt | 11493 | 11308 | 185 | 98.39 | 98.81 | 87.66 | 87.95 |
| TOON_KEYFOLD | man | 11495 | 11310 | 185 | 98.39 | 98.09 | 87.65 | 87.44 |
| TOON_KEYFOLD | opt | 11348 | 11135 | 213 | 98.12 | 98.48 | 87.85 | 88.11 |
| XML_COMPACT | man | 10362 | 10168 | 195 | 98.12 | 98.00 | 90.46 | 90.38 |
| XML_COMPACT | opt | 9981 | 9766 | 215 | 97.85 | 98.15 | 91.28 | 91.49 |
| XML_PRETTY | man | 18582 | 18132 | 450 | 97.58 | 97.13 | 68.33 | 68.02 |
| XML_PRETTY | opt | 18051 | 17712 | 339 | 98.12 | 98.35 | 70.11 | 70.28 |
| YAML | man | 14673 | 14634 | 40 | 99.73 | 99.77 | 80.18 | 80.21 |
| YAML | opt | 11700 | 11448 | 252 | 97.85 | 98.41 | 86.73 | 87.13 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7702 | 7266 | -436 | -5.66 | 7598 | 7110 | -488 | -6.43 | 104 | 156 |  +52 |  +50.23 | 98.65 | 97.85 | -0.80 | 97.88 | 98.469 |  +0.59 |  +0.61 |
| JSON_PRETTY | 16083 | 15058 | -1025 | -6.37 | 15781 | 14694 | -1087 | -6.89 | 302 | 364 |  +62 |  +20.55 | 98.12 | 97.58 | -0.54 | 75.32 | 77.656 |  +2.33 |  +3.10 |
| TOON_DEFAULT | 11741 | 11493 | -248 | -2.11 | 11709 | 11308 | -401 | -3.43 | 32 | 185 |  +153 |  +479.16 | 99.73 | 98.39 | -1.34 | 87.94 | 87.661 | -0.28 | -0.32 |
| TOON_KEYFOLD | 11495 | 11348 | -147 | -1.28 | 11310 | 11134 | -176 | -1.55 | 185 | 213 |  +28 |  +15.28 | 98.39 | 98.12 | -0.27 | 87.65 | 87.854 |  +0.20 |  +0.23 |
| XML_COMPACT | 10362 | 9981 | -381 | -3.68 | 10168 | 9767 | -401 | -3.94 | 195 | 215 |  +20 |  +10.14 | 98.12 | 97.85 | -0.27 | 90.46 | 91.283 |  +0.82 |  +0.91 |
| XML_PRETTY | 18582 | 18052 | -530 | -2.85 | 18132 | 17712 | -420 | -2.32 | 450 | 340 | -110 | -24.51 | 97.58 | 98.12 |  +0.54 | 68.33 | 70.114 |  +1.78 |  +2.61 |
| YAML | 14673 | 11699 | -2974 | -20.27 | 14634 | 11448 | -3186 | -21.77 | 40 | 252 |  +212 |  +529.81 | 99.73 | 97.85 | -1.88 | 80.18 | 86.735 |  +6.55 |  +8.17 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| JSON_COMPACT | man | 122 | 2 | 0 | 98.65 |
| JSON_COMPACT | opt | 121 | 3 | 0 | 97.85 |
| JSON_PRETTY | man | 122 | 2 | 0 | 98.12 |
| JSON_PRETTY | opt | 121 | 3 | 0 | 97.58 |
| TOON_DEFAULT | man | 124 | 0 | 0 | 99.73 |
| TOON_DEFAULT | opt | 122 | 2 | 0 | 98.39 |
| TOON_KEYFOLD | man | 122 | 2 | 0 | 98.39 |
| TOON_KEYFOLD | opt | 122 | 2 | 0 | 98.12 |
| XML_COMPACT | man | 122 | 2 | 0 | 98.12 |
| XML_COMPACT | opt | 121 | 3 | 0 | 97.85 |
| XML_PRETTY | man | 121 | 3 | 0 | 97.58 |
| XML_PRETTY | opt | 122 | 2 | 0 | 98.12 |
| YAML | man | 124 | 0 | 0 | 99.73 |
| YAML | opt | 121 | 3 | 0 | 97.85 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 122 | 121 | -1 | -0.82 | 2 | 3 |  +1 |  +50.00 | 0 | 0 | 0 | 0.00 | 98.65 | 97.85 | -0.80 |
| JSON_PRETTY | 122 | 121 | -1 | -0.82 | 2 | 3 |  +1 |  +50.00 | 0 | 0 | 0 | 0.00 | 98.12 | 97.58 | -0.54 |
| TOON_DEFAULT | 124 | 122 | -2 | -1.61 | 0 | 2 |  +2 | 0.00 | 0 | 0 | 0 | 0.00 | 99.73 | 98.39 | -1.34 |
| TOON_KEYFOLD | 122 | 122 | 0 | 0.00 | 2 | 2 | 0 | 0.00 | 0 | 0 | 0 | 0.00 | 98.39 | 98.12 | -0.27 |
| XML_COMPACT | 122 | 121 | -1 | -0.82 | 2 | 3 |  +1 |  +50.00 | 0 | 0 | 0 | 0.00 | 98.12 | 97.85 | -0.27 |
| XML_PRETTY | 121 | 122 |  +1 |  +0.83 | 3 | 2 | -1 | -33.33 | 0 | 0 | 0 | 0.00 | 97.58 | 98.12 |  +0.54 |
| YAML | 124 | 121 | -3 | -2.42 | 0 | 3 |  +3 | 0.00 | 0 | 0 | 0 | 0.00 | 99.73 | 97.85 | -1.88 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 98.65 | 100.00 | 100.00 | 95.24 | 96.83 |
| JSON_COMPACT | opt | 97.85 | 100.00 | 98.77 | 98.41 | 90.48 |
| JSON_PRETTY | man | 98.12 | 100.00 | 98.77 | 93.65 | 96.83 |
| JSON_PRETTY | opt | 97.58 | 100.00 | 100.00 | 95.24 | 90.48 |
| TOON_DEFAULT | man | 99.73 | 100.00 | 100.00 | 98.41 | 100.00 |
| TOON_DEFAULT | opt | 98.39 | 100.00 | 100.00 | 100.00 | 90.48 |
| TOON_KEYFOLD | man | 98.39 | 100.00 | 97.53 | 95.24 | 98.41 |
| TOON_KEYFOLD | opt | 98.12 | 100.00 | 100.00 | 98.41 | 90.48 |
| XML_COMPACT | man | 98.12 | 98.79 | 98.77 | 95.24 | 98.41 |
| XML_COMPACT | opt | 97.85 | 100.00 | 100.00 | 96.83 | 90.48 |
| XML_PRETTY | man | 97.58 | 99.39 | 100.00 | 87.30 | 100.00 |
| XML_PRETTY | opt | 98.12 | 100.00 | 100.00 | 96.83 | 92.07 |
| YAML | man | 99.73 | 99.39 | 100.00 | 100.00 | 100.00 |
| YAML | opt | 97.85 | 100.00 | 100.00 | 100.00 | 87.30 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 100.00 | 100.00 | 0.00 |
| JSON_PRETTY | 100.00 | 100.00 | 0.00 |
| TOON_DEFAULT | 100.00 | 100.00 | 0.00 |
| TOON_KEYFOLD | 100.00 | 100.00 | 0.00 |
| XML_COMPACT | 98.79 | 100.00 |  +1.21 |
| XML_PRETTY | 99.39 | 100.00 |  +0.61 |
| YAML | 99.39 | 100.00 |  +0.61 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 100.00 | 98.77 | -1.23 |
| JSON_PRETTY | 98.77 | 100.00 |  +1.23 |
| TOON_DEFAULT | 100.00 | 100.00 | 0.00 |
| TOON_KEYFOLD | 97.53 | 100.00 |  +2.47 |
| XML_COMPACT | 98.77 | 100.00 |  +1.23 |
| XML_PRETTY | 100.00 | 100.00 | 0.00 |
| YAML | 100.00 | 100.00 | 0.00 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 95.24 | 98.41 |  +3.17 |
| JSON_PRETTY | 93.65 | 95.24 |  +1.58 |
| TOON_DEFAULT | 98.41 | 100.00 |  +1.59 |
| TOON_KEYFOLD | 95.24 | 98.41 |  +3.17 |
| XML_COMPACT | 95.24 | 96.83 |  +1.59 |
| XML_PRETTY | 87.30 | 96.83 |  +9.53 |
| YAML | 100.00 | 100.00 | 0.00 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 96.83 | 90.48 | -6.35 |
| JSON_PRETTY | 96.83 | 90.48 | -6.35 |
| TOON_DEFAULT | 100.00 | 90.48 | -9.52 |
| TOON_KEYFOLD | 98.41 | 90.48 | -7.93 |
| XML_COMPACT | 98.41 | 90.48 | -7.93 |
| XML_PRETTY | 100.00 | 92.07 | -7.93 |
| YAML | 100.00 | 87.30 | -12.70 |

## 3. Format-Specific Analysis
### 3.1 Detailed Analysis: JSON_COMPACT

#### 3.1.1 Performance Summary

- Token Duration Range: 307 - 324 seconds
- Token Cost Range: 7266 - 7702 tokens
- Wasted Token Range: 104 - 156 tokens
- Accuracy Range: 97.85 - 98.65%
- Efficiency Score Range: 97.88 - 98.47

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

- Token Duration Range: 296 - 313 seconds
- Token Cost Range: 15059 - 16083 tokens
- Wasted Token Range: 302 - 364 tokens
- Accuracy Range: 97.58 - 98.12%
- Efficiency Score Range: 75.32 - 77.66

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

- Token Duration Range: 263 - 279 seconds
- Token Cost Range: 11493 - 11741 tokens
- Wasted Token Range: 32 - 185 tokens
- Accuracy Range: 98.39 - 99.73%
- Efficiency Score Range: 87.66 - 87.94

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

- Token Duration Range: 283 - 310 seconds
- Token Cost Range: 11348 - 11495 tokens
- Wasted Token Range: 185 - 213 tokens
- Accuracy Range: 98.12 - 98.39%
- Efficiency Score Range: 87.65 - 87.85

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

- Token Duration Range: 301 - 330 seconds
- Token Cost Range: 9981 - 10362 tokens
- Wasted Token Range: 195 - 215 tokens
- Accuracy Range: 97.85 - 98.12%
- Efficiency Score Range: 90.46 - 91.28

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

- Token Duration Range: 323 - 324 seconds
- Token Cost Range: 18051 - 18582 tokens
- Wasted Token Range: 339 - 450 tokens
- Accuracy Range: 97.58 - 98.12%
- Efficiency Score Range: 68.33 - 70.11

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

- Token Duration Range: 314 - 315 seconds
- Token Cost Range: 11700 - 14673 tokens
- Wasted Token Range: 40 - 252 tokens
- Accuracy Range: 97.85 - 99.73%
- Efficiency Score Range: 80.18 - 86.73

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
- **Model**: Sonnet 4.6
- **Thinking**: on (medium)
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