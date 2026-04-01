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
- **Read Tokens**: For each data file a single read subagent is invoked with the only prompt to read the file at the provided filepath and return "Done" once finished and do nothing more. The token extraction script searches for the read tool result and extracts only the read tokens of it.
- **Output Tokens**: For each data file three "benchmark-full-test" subagent are invoked with data, questions and answers template files and the instructions to read everything and answer all questions in a single write tool use. The token extraction script aggregates all output tokens until and including the write tool result.

## 2. Results

### 2.1 TLDR: Token Efficiency Analysis

*Note: All columns ranked best-to-worst. ↑ = lower value is better (ascending). ↓ = higher value is better (descending).*

#### 2.1.1 Best results

- Lowest total token cost:
   - Optional: JSON_COMPACT 30568 tokens
   - Mandatory: TOON_DEFAULT 32551 tokens
- Lowest read token cost:
   - Optional: JSON_COMPACT 6970 tokens
   - Mandatory: JSON_COMPACT 7498 tokens
- Lowest output token cost:
   - Optional: TOON_DEFAULT 22004 tokens
   - Mandatory: TOON_DEFAULT 21111 tokens
- Lowest output token cost drift:
   - Optional: XML_COMPACT ↓ -2.92% ↑ 3.46%
   - Mandatory: XML_PRETTY ↓ -2.34% ↑ 3.33%
- Highest accuracy:
   - Optional: TOON_DEFAULT 98.39%
   - Mandatory: TOON_DEFAULT 99.73%
- Lowest accuracy drift:
   - Optional: TOON_DEFAULT ↓ 0.00% ↑ 0.00%
   - Mandatory: TOON_DEFAULT ↓ -0.54% ↑ 0.27%
- Most useful tokens:
   - Optional: XML_PRETTY 42142 / 42949 tokens
   - Mandatory: XML_PRETTY 43053 / 44121 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 98.47
   - Mandatory: TOON_DEFAULT 95.41
- Lowest delta (optional-mandatory):
   - Total tokens: JSON_PRETTY -588 tokens
   - Accuracy: TOON_KEYFOLD -0.27%
   - Token efficiency: JSON_PRETTY 0.92

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 42949 tokens
   - Mandatory: XML_PRETTY 44121 tokens
- Highest read token cost:
   - Optional: XML_PRETTY 17760 tokens
   - Mandatory: XML_PRETTY 18291 tokens
- Highest output token cost:
   - Optional: XML_COMPACT 26275 tokens
   - Mandatory: JSON_COMPACT 26036 tokens
- Highest output token drift:
   - Optional: JSON_PRETTY ↓ -12.17% ↑ 20.78%
   - Mandatory: TOON_KEYFOLD ↓ -16.60% ↑ 17.19%
- Lowest accuracy:
   - Optional: JSON_PRETTY 97.58%
   - Mandatory: XML_PRETTY 97.58%
- Highest accuracy drift:
   - Optional: JSON_PRETTY ↓ -1.65% ↑ 0.83%
   - Mandatory: XML_COMPACT ↓ -3.02% ↑ 1.92%
- Most wasted tokens:
   - Optional: JSON_PRETTY 935 / 38654 tokens
   - Mandatory: XML_PRETTY 1068 / 44121 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 71.30
   - Mandatory: XML_PRETTY 68.33
- Highest delta (optional-mandatory):
   - Total tokens: YAML -3169 tokens
   - Accuracy: YAML -1.88%
   - Token efficiency: XML_COMPACT -6.73

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 263s | JSON_COMPACT ≈ 7498 | TOON_DEFAULT ≈ 21111 | TOON_DEFAULT ≈ 32551 | TOON_DEFAULT ≈ 88 | TOON_DEFAULT ≈ 100% | YAML ≈ 100% | TOON_DEFAULT ≈ 95 | TOON_DEFAULT ≈ 95 |
| TOON_KEYFOLD (+7.7%) | XML_COMPACT (+35.6%) | TOON_KEYFOLD (+4.0%) | XML_COMPACT (+1.4%) | YAML (+15.9%) | YAML (0.0%) | TOON_DEFAULT (-0.1%) | XML_COMPACT (-2.2%) | XML_COMPACT (-2.3%) |
| JSON_PRETTY (+12.5%) | TOON_KEYFOLD (+50.6%) | XML_COMPACT (+8.2%) | TOON_KEYFOLD (+2.2%) | JSON_COMPACT (+415.1%) | JSON_COMPACT (-1.1%) | JSON_COMPACT (-1.2%) | TOON_KEYFOLD (-2.6%) | TOON_KEYFOLD (-2.8%) |
| XML_COMPACT (+14.4%) | TOON_DEFAULT (+52.6%) | YAML (+10.1%) | JSON_COMPACT (+3.0%) | TOON_KEYFOLD (+509.2%) | TOON_KEYFOLD (-1.3%) | TOON_KEYFOLD (-1.7%) | JSON_COMPACT (-3.1%) | JSON_COMPACT (-3.1%) |
| YAML (+19.8%) | YAML (+93.1%) | JSON_PRETTY (+11.1%) | YAML (+15.9%) | XML_COMPACT (+605.9%) | JSON_PRETTY (-1.6%) | XML_COMPACT (-1.8%) | YAML (-12.0%) | YAML (-11.9%) |
| XML_PRETTY (+23.0%) | JSON_PRETTY (+110.5%) | XML_PRETTY (+22.4%) | JSON_PRETTY (+20.6%) | JSON_PRETTY (+739.4%) | XML_COMPACT (-1.6%) | JSON_PRETTY (-1.8%) | JSON_PRETTY (-16.7%) | JSON_PRETTY (-16.8%) |
| JSON_COMPACT (+23.1%) | XML_PRETTY (+143.9%) | JSON_COMPACT (+23.3%) | XML_PRETTY (+35.5%) | XML_PRETTY (+1114.9%) | XML_PRETTY (-2.2%) | XML_PRETTY (-2.6%) | XML_PRETTY (-28.4%) | XML_PRETTY (-28.7%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 279s | JSON_COMPACT ≈ 6970 | TOON_DEFAULT ≈ 22004 | JSON_COMPACT ≈ 30568 | TOON_DEFAULT ≈ 534 | TOON_DEFAULT ≈ 98% | TOON_DEFAULT ≈ 99% | JSON_COMPACT ≈ 98 | JSON_COMPACT ≈ 99 |
| JSON_COMPACT (+9.8%) | XML_COMPACT (+38.9%) | YAML (+5.2%) | TOON_DEFAULT (+8.6%) | JSON_COMPACT (+23.0%) | TOON_KEYFOLD (-0.3%) | TOON_KEYFOLD (-0.3%) | TOON_DEFAULT (-5.5%) | TOON_DEFAULT (-5.4%) |
| TOON_KEYFOLD (+10.9%) | TOON_KEYFOLD (+58.5%) | JSON_COMPACT (+7.2%) | YAML (+13.0%) | TOON_KEYFOLD (+24.9%) | XML_PRETTY (-0.3%) | YAML (-0.4%) | YAML (-8.9%) | YAML (-8.7%) |
| JSON_PRETTY (+11.9%) | TOON_DEFAULT (+60.6%) | JSON_PRETTY (+8.1%) | TOON_KEYFOLD (+16.2%) | YAML (+39.0%) | JSON_COMPACT (-0.5%) | XML_PRETTY (-0.5%) | TOON_KEYFOLD (-10.9%) | TOON_KEYFOLD (-10.8%) |
| YAML (+12.3%) | YAML (+63.6%) | TOON_KEYFOLD (+11.2%) | XML_COMPACT (+17.6%) | XML_COMPACT (+44.6%) | XML_COMPACT (-0.5%) | XML_COMPACT (-0.7%) | XML_COMPACT (-12.1%) | XML_COMPACT (-12.1%) |
| XML_PRETTY (+16.0%) | JSON_PRETTY (+113.2%) | XML_PRETTY (+14.5%) | JSON_PRETTY (+26.5%) | XML_PRETTY (+51.1%) | YAML (-0.5%) | JSON_COMPACT (-0.7%) | JSON_PRETTY (-18.3%) | JSON_PRETTY (-18.3%) |
| XML_COMPACT (+18.0%) | XML_PRETTY (+154.8%) | XML_COMPACT (+19.4%) | XML_PRETTY (+40.5%) | JSON_PRETTY (+75.0%) | JSON_PRETTY (-0.8%) | JSON_PRETTY (-1.0%) | XML_PRETTY (-27.6%) | XML_PRETTY (-27.6%) |


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
| JSON_COMPACT | man | 7498 | 26036 | 33534 | 3.190 | 0.294 | 209.968 | 98.65 | 92.45 | 33081.291 | 452.709 | 92.48 | 92.45 |
| JSON_COMPACT | opt | 6970 | 23598 | 30568 | 3.226 | 0.320 | 190.309 | 97.85 | 98.66 | 29911.114 | 657.219 | 98.47 | 98.66 |
| JSON_PRETTY | man | 15787 | 23455 | 39242 | 2.094 | 0.250 | 189.153 | 98.12 | 79.35 | 38504.250 | 737.750 | 79.49 | 79.35 |
| JSON_PRETTY | opt | 14858 | 23796 | 38654 | 2.105 | 0.252 | 191.906 | 97.58 | 80.58 | 37718.898 | 935.435 | 80.41 | 80.58 |
| TOON_DEFAULT | man | 11440 | 21111 | 32551 | 2.370 | 0.306 | 170.247 | 99.73 | 95.36 | 32462.780 | 87.887 | 95.41 | 95.36 |
| TOON_DEFAULT | opt | 11195 | 22004 | 33199 | 2.394 | 0.296 | 177.449 | 98.39 | 93.33 | 32664.168 | 534.499 | 93.04 | 93.33 |
| TOON_KEYFOLD | man | 11289 | 21964 | 33253 | 2.382 | 0.296 | 177.129 | 98.39 | 92.71 | 32717.627 | 535.373 | 92.92 | 92.71 |
| TOON_KEYFOLD | opt | 11044 | 24474 | 35518 | 2.407 | 0.276 | 197.371 | 98.12 | 87.97 | 34850.262 | 667.738 | 87.72 | 87.97 |
| XML_COMPACT | man | 10164 | 22837 | 33001 | 3.371 | 0.297 | 184.172 | 98.12 | 93.20 | 32380.908 | 620.425 | 93.28 | 93.20 |
| XML_COMPACT | opt | 9684 | 26275 | 35959 | 3.407 | 0.272 | 211.892 | 97.85 | 86.77 | 35185.556 | 773.111 | 86.56 | 86.77 |
| XML_PRETTY | man | 18291 | 25830 | 44121 | 2.321 | 0.221 | 208.304 | 97.58 | 68.01 | 43052.947 | 1067.720 | 68.33 | 68.01 |
| XML_PRETTY | opt | 17760 | 25189 | 42949 | 2.319 | 0.228 | 203.137 | 98.12 | 71.46 | 42141.559 | 807.441 | 71.30 | 71.46 |
| YAML | man | 14475 | 23241 | 37716 | 1.835 | 0.264 | 187.430 | 99.73 | 84.02 | 37614.499 | 101.834 | 83.99 | 84.02 |
| YAML | opt | 11404 | 23144 | 34548 | 2.301 | 0.283 | 186.642 | 97.85 | 90.07 | 33804.892 | 742.775 | 89.68 | 90.07 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7498 | 6970 | -528 | -7.04 | 26036 | 23598 | -2438 | -9.36 | 33534 | 30568 | -2966 | -8.84 | 98.61 | 98.12 | -0.49 | 92.45 | 98.66 |  +6.21 |
| JSON_PRETTY | 15787 | 14858 | -929 | -5.88 | 23455 | 23796 |  +341 |  +1.45 | 39242 | 38654 | -588 | -1.50 | 97.92 | 97.82 | -0.10 | 79.35 | 80.58 |  +1.23 |
| TOON_DEFAULT | 11440 | 11195 | -245 | -2.14 | 21111 | 22004 |  +893 |  +4.23 | 32551 | 33199 |  +648 |  +1.99 | 99.67 | 98.81 | -0.86 | 95.36 | 93.33 | -2.03 |
| TOON_KEYFOLD | 11289 | 11044 | -245 | -2.17 | 21964 | 24474 |  +2510 |  +11.43 | 33253 | 35518 |  +2265 |  +6.81 | 98.09 | 98.48 |  +0.39 | 92.71 | 87.97 | -4.73 |
| XML_COMPACT | 10164 | 9684 | -480 | -4.72 | 22837 | 26274 |  +3437 |  +15.05 | 33001 | 35958 |  +2957 |  +8.96 | 98.00 | 98.15 |  +0.15 | 93.20 | 86.77 | -6.43 |
| XML_PRETTY | 18291 | 17760 | -531 | -2.90 | 25830 | 25189 | -641 | -2.48 | 44121 | 42949 | -1172 | -2.66 | 97.13 | 98.35 |  +1.22 | 68.01 | 71.46 |  +3.44 |
| YAML | 14475 | 11404 | -3071 | -21.22 | 23241 | 23143 | -98 | -0.42 | 37716 | 34547 | -3169 | -8.40 | 99.77 | 98.41 | -1.36 | 84.02 | 90.07 |  +6.05 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 22 | 340.818 | 0.71 | 323514 | 0.080 | 2608.98 | 323536 | 340.898 | 2087.33 |
| JSON_COMPACT | opt | 20 | 348.500 | 0.65 | 306728 | 0.077 | 2473.61 | 306748 | 348.577 | 1979.02 |
| JSON_PRETTY | man | 311 | 50.762 | 10.03 | 295508 | 0.079 | 2383.13 | 295819 | 50.841 | 1908.51 |
| JSON_PRETTY | opt | 366 | 40.596 | 11.81 | 312166 | 0.076 | 2517.47 | 312532 | 40.672 | 2016.34 |
| TOON_DEFAULT | man | 36 | 317.778 | 1.16 | 262869 | 0.080 | 2119.91 | 262905 | 317.858 | 1696.16 |
| TOON_DEFAULT | opt | 30 | 373.167 | 0.97 | 279384 | 0.079 | 2253.10 | 279414 | 373.246 | 1802.67 |
| TOON_KEYFOLD | man | 12 | 940.750 | 0.39 | 283145 | 0.078 | 2283.42 | 283157 | 940.828 | 1826.82 |
| TOON_KEYFOLD | opt | 30 | 368.133 | 0.97 | 309913 | 0.079 | 2499.30 | 309943 | 368.212 | 1999.63 |
| XML_COMPACT | man | 35 | 290.400 | 1.13 | 300623 | 0.076 | 2424.38 | 300658 | 290.476 | 1939.73 |
| XML_COMPACT | opt | 10 | 968.400 | 0.32 | 329615 | 0.080 | 2658.18 | 329625 | 968.480 | 2126.61 |
| XML_PRETTY | man | 29 | 630.724 | 0.94 | 323223 | 0.080 | 2606.63 | 323252 | 630.804 | 2085.49 |
| XML_PRETTY | opt | 30 | 592.000 | 0.97 | 324069 | 0.078 | 2613.46 | 324099 | 592.078 | 2090.96 |
| YAML | man | 28 | 516.964 | 0.90 | 314910 | 0.074 | 2539.59 | 314938 | 517.038 | 2031.86 |
| YAML | opt | 44 | 259.182 | 1.42 | 313627 | 0.074 | 2529.25 | 313671 | 259.256 | 2023.68 |

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
| JSON_COMPACT | man | 3.190 | 10.994 | 241.871 | 0.294 |
| JSON_COMPACT | opt | 3.226 | 11.046 | 224.839 | 0.320 |
| JSON_PRETTY | man | 2.094 | 23.148 | 509.258 | 0.250 |
| JSON_PRETTY | opt | 2.105 | 23.547 | 479.290 | 0.252 |
| TOON_DEFAULT | man | 2.370 | 16.774 | 369.032 | 0.306 |
| TOON_DEFAULT | opt | 2.394 | 17.742 | 361.129 | 0.296 |
| TOON_KEYFOLD | man | 2.382 | 16.553 | 364.161 | 0.296 |
| TOON_KEYFOLD | opt | 2.407 | 17.502 | 356.258 | 0.276 |
| XML_COMPACT | man | 3.371 | 14.903 | 327.871 | 0.297 |
| XML_COMPACT | opt | 3.407 | 15.347 | 312.387 | 0.272 |
| XML_PRETTY | man | 2.321 | 26.820 | 590.032 | 0.221 |
| XML_PRETTY | opt | 2.319 | 28.146 | 572.903 | 0.228 |
| YAML | man | 1.835 | 21.224 | 466.935 | 0.264 |
| YAML | opt | 2.301 | 18.073 | 367.871 | 0.283 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 3.190 | 3.226 |  +0.036 |  +1.13 | 10.994 | 11.046 |  +0.052 |  +0.47 | 241.871 | 224.839 | -17.032 | -7.04 | 0.294 | 0.320 |  +0.026 |  +8.84 |
| JSON_PRETTY | 2.094 | 2.105 |  +0.011 |  +0.53 | 23.148 | 23.547 |  +0.399 |  +1.72 | 509.258 | 479.290 | -29.968 | -5.88 | 0.250 | 0.252 |  +0.002 |  +0.80 |
| TOON_DEFAULT | 2.370 | 2.394 |  +0.024 |  +1.01 | 16.774 | 17.742 |  +0.968 |  +5.77 | 369.032 | 361.129 | -7.903 | -2.14 | 0.306 | 0.296 | -0.010 | -3.27 |
| TOON_KEYFOLD | 2.382 | 2.407 |  +0.025 |  +1.05 | 16.553 | 17.502 |  +0.949 |  +5.73 | 364.161 | 356.258 | -7.903 | -2.17 | 0.296 | 0.276 | -0.020 | -6.76 |
| XML_COMPACT | 3.371 | 3.407 |  +0.036 |  +1.07 | 14.903 | 15.347 |  +0.444 |  +2.98 | 327.871 | 312.387 | -15.484 | -4.72 | 0.297 | 0.272 | -0.025 | -8.42 |
| XML_PRETTY | 2.321 | 2.319 | -0.002 | -0.09 | 26.820 | 28.146 |  +1.326 |  +4.94 | 590.032 | 572.903 | -17.129 | -2.90 | 0.221 | 0.228 |  +0.007 |  +3.17 |
| YAML | 1.835 | 2.301 |  +0.466 |  +25.40 | 21.224 | 18.073 | -3.151 | -14.85 | 466.935 | 367.871 | -99.064 | -21.22 | 0.264 | 0.283 |  +0.019 |  +7.20 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 33534 | 33081 | 453 | 98.65 | 98.61 | 92.48 | 92.45 |
| JSON_COMPACT | opt | 30568 | 29911 | 657 | 97.85 | 98.12 | 98.47 | 98.66 |
| JSON_PRETTY | man | 39242 | 38504 | 738 | 98.12 | 97.92 | 79.49 | 79.35 |
| JSON_PRETTY | opt | 38654 | 37719 | 935 | 97.58 | 97.82 | 80.41 | 80.58 |
| TOON_DEFAULT | man | 32551 | 32463 | 88 | 99.73 | 99.67 | 95.41 | 95.36 |
| TOON_DEFAULT | opt | 33199 | 32664 | 534 | 98.39 | 98.81 | 93.04 | 93.33 |
| TOON_KEYFOLD | man | 33253 | 32718 | 535 | 98.39 | 98.09 | 92.92 | 92.71 |
| TOON_KEYFOLD | opt | 35518 | 34850 | 668 | 98.12 | 98.48 | 87.72 | 87.97 |
| XML_COMPACT | man | 33001 | 32381 | 620 | 98.12 | 98.00 | 93.28 | 93.20 |
| XML_COMPACT | opt | 35959 | 35186 | 773 | 97.85 | 98.15 | 86.56 | 86.77 |
| XML_PRETTY | man | 44121 | 43053 | 1068 | 97.58 | 97.13 | 68.33 | 68.01 |
| XML_PRETTY | opt | 42949 | 42142 | 807 | 98.12 | 98.35 | 71.30 | 71.46 |
| YAML | man | 37716 | 37614 | 102 | 99.73 | 99.77 | 83.99 | 84.02 |
| YAML | opt | 34548 | 33805 | 743 | 97.85 | 98.41 | 89.68 | 90.07 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 33534 | 30568 | -2966 | -8.84 | 33081 | 29911 | -3170 | -9.58 | 453 | 658 |  +205 |  +45.15 | 98.65 | 97.85 | -0.80 | 92.48 | 98.473 |  +6.00 |  +6.48 |
| JSON_PRETTY | 39242 | 38654 | -588 | -1.50 | 38504 | 37719 | -785 | -2.04 | 738 | 936 |  +198 |  +26.79 | 98.12 | 97.58 | -0.54 | 79.49 | 80.411 |  +0.92 |  +1.16 |
| TOON_DEFAULT | 32551 | 33199 |  +648 |  +1.99 | 32463 | 32664 |  +201 |  +0.62 | 88 | 535 |  +447 |  +507.51 | 99.73 | 98.39 | -1.34 | 95.41 | 93.037 | -2.37 | -2.48 |
| TOON_KEYFOLD | 33253 | 35518 |  +2265 |  +6.81 | 32718 | 34851 |  +2133 |  +6.52 | 535 | 667 |  +132 |  +24.74 | 98.39 | 98.12 | -0.27 | 92.92 | 87.721 | -5.20 | -5.59 |
| XML_COMPACT | 33001 | 35958 |  +2957 |  +8.96 | 32381 | 35186 |  +2805 |  +8.66 | 620 | 773 |  +153 |  +24.63 | 98.12 | 97.85 | -0.27 | 93.28 | 86.558 | -6.73 | -7.21 |
| XML_PRETTY | 44121 | 42949 | -1172 | -2.66 | 43053 | 42142 | -911 | -2.12 | 1068 | 808 | -260 | -24.37 | 97.58 | 98.12 |  +0.54 | 68.33 | 71.296 |  +2.97 |  +4.34 |
| YAML | 37716 | 34547 | -3169 | -8.40 | 37614 | 33804 | -3810 | -10.13 | 102 | 743 |  +641 |  +628.37 | 99.73 | 97.85 | -1.88 | 83.99 | 89.677 |  +5.69 |  +6.77 |

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

- **Report Generated**: 2026-04-01
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code < 2.1.86
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)