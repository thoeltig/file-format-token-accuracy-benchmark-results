# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Sonnet 4.6
- **Thinking**: off
- **Data Structure**: flat
- **Formats Tested**: 7 (CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML)
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
   - Optional: CSV 3925 tokens
   - Mandatory: CSV 7039 tokens
- Lowest output token cost drift:
   - Optional: JSON_COMPACT ↓ -0.22% ↑ 0.11%
   - Mandatory: XML_COMPACT ↓ 0.00% ↑ 0.00%
- Highest accuracy:
   - Optional: JSON_COMPACT 98.12%
   - Mandatory: JSON_PRETTY 99.46%
- Lowest accuracy drift:
   - Optional: JSON_COMPACT ↓ -0.55% ↑ 0.28%
   - Mandatory: XML_COMPACT ↓ 0.00% ↑ 0.00%
- Most useful tokens:
   - Optional: XML_PRETTY 11992 / 12222 tokens
   - Mandatory: XML_PRETTY 13023 / 13383 tokens
- Highest token efficiency (%/token):
   - Optional: CSV 97.90
   - Mandatory: TOON_DEFAULT 89.17
- Lowest delta (optional-mandatory):
   - Total tokens: YAML -783 tokens
   - Accuracy: JSON_COMPACT -0.54%
   - Token efficiency: JSON_PRETTY 1.66

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 12222 tokens
   - Mandatory: XML_PRETTY 13383 tokens
- Highest output token drift:
   - Optional: XML_PRETTY ↓ -93.63% ↑ 47.55%
   - Mandatory: YAML ↓ -95.45% ↑ 49.24%
- Lowest accuracy:
   - Optional: XML_COMPACT 96.50%
   - Mandatory: XML_PRETTY 97.31%
- Highest accuracy drift:
   - Optional: CSV ↓ -2.77% ↑ 2.22%
   - Mandatory: JSON_COMPACT ↓ -2.73% ↑ 1.36%
- Most wasted tokens:
   - Optional: XML_COMPACT 285 / 8147 tokens
   - Mandatory: XML_PRETTY 360 / 13383 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 72.39
   - Mandatory: XML_PRETTY 68.15
- Highest delta (optional-mandatory):
   - Total tokens: XML_COMPACT -3595 tokens
   - Accuracy: XML_COMPACT -2.69%
   - Token efficiency: JSON_COMPACT 10.25

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 273s | CSV ≈ 7039 | JSON_PRETTY ≈ 62 | JSON_PRETTY ≈ 99% | JSON_PRETTY ≈ 99% | TOON_DEFAULT ≈ 89 | TOON_DEFAULT ≈ 89 |
| JSON_COMPACT (+6.3%) | TOON_DEFAULT (+0.9%) | TOON_DEFAULT (+24.5%) | XML_COMPACT (-0.3%) | XML_COMPACT (-0.4%) | CSV (-0.4%) | CSV (-0.5%) |
| XML_PRETTY (+10.4%) | JSON_COMPACT (+32.5%) | XML_COMPACT (+54.4%) | TOON_DEFAULT (-0.5%) | TOON_DEFAULT (-0.5%) | JSON_COMPACT (-8.1%) | JSON_COMPACT (-8.1%) |
| YAML (+16.6%) | YAML (+37.5%) | JSON_COMPACT (+102.9%) | JSON_COMPACT (-0.8%) | JSON_COMPACT (-0.8%) | YAML (-9.4%) | YAML (-9.4%) |
| XML_COMPACT (+22.0%) | JSON_PRETTY (+62.1%) | YAML (+110.5%) | YAML (-0.8%) | YAML (-0.9%) | JSON_PRETTY (-14.9%) | JSON_PRETTY (-14.9%) |
| TOON_DEFAULT (+22.5%) | XML_COMPACT (+66.8%) | CSV (+114.8%) | CSV (-1.3%) | CSV (-1.4%) | XML_COMPACT (-16.3%) | XML_COMPACT (-16.3%) |
| CSV (+26.1%) | XML_PRETTY (+90.1%) | XML_PRETTY (+484.4%) | XML_PRETTY (-2.1%) | XML_PRETTY (-2.7%) | XML_PRETTY (-23.6%) | XML_PRETTY (-24.0%) |


##### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| XML_PRETTY ≈ 281s | CSV ≈ 3925 | JSON_COMPACT ≈ 112 | JSON_COMPACT ≈ 98% | JSON_COMPACT ≈ 98% | CSV ≈ 98 | CSV ≈ 98 |
| JSON_PRETTY (+1.5%) | JSON_COMPACT (+52.1%) | CSV (+3.5%) | JSON_PRETTY (0.0%) | XML_PRETTY (0.0%) | JSON_COMPACT (-5.8%) | JSON_COMPACT (-5.5%) |
| JSON_COMPACT (+3.9%) | XML_COMPACT (+107.6%) | TOON_DEFAULT (+47.2%) | TOON_DEFAULT (0.0%) | JSON_PRETTY (-0.0%) | XML_COMPACT (-14.0%) | XML_COMPACT (-13.9%) |
| TOON_DEFAULT (+11.0%) | TOON_DEFAULT (+123.9%) | JSON_PRETTY (+77.4%) | XML_PRETTY (0.0%) | TOON_DEFAULT (-0.0%) | TOON_DEFAULT (-15.0%) | TOON_DEFAULT (-14.7%) |
| YAML (+16.3%) | YAML (+126.6%) | YAML (+91.8%) | YAML (-0.5%) | YAML (-0.6%) | YAML (-15.7%) | YAML (-15.4%) |
| XML_COMPACT (+18.9%) | JSON_PRETTY (+169.7%) | XML_PRETTY (+104.8%) | CSV (-1.1%) | CSV (-1.5%) | JSON_PRETTY (-20.8%) | JSON_PRETTY (-20.5%) |
| CSV (+29.0%) | XML_PRETTY (+211.4%) | XML_COMPACT (+154.1%) | XML_COMPACT (-1.6%) | XML_COMPACT (-1.8%) | XML_PRETTY (-26.1%) | XML_PRETTY (-25.8%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| CSV ≈ 100% | CSV ≈ 100% | JSON_COMPACT ≈ 100% | JSON_PRETTY ≈ 98% |
| JSON_PRETTY (0.0%) | JSON_PRETTY (0.0%) | JSON_PRETTY (-1.6%) | XML_COMPACT (0.0%) |
| TOON_DEFAULT (0.0%) | TOON_DEFAULT (0.0%) | XML_COMPACT (-1.6%) | YAML (0.0%) |
| XML_COMPACT (0.0%) | YAML (0.0%) | TOON_DEFAULT (-3.2%) | JSON_COMPACT (-1.6%) |
| XML_PRETTY (0.0%) | XML_COMPACT (-1.2%) | XML_PRETTY (-3.2%) | TOON_DEFAULT (-1.6%) |
| JSON_COMPACT (-0.6%) | JSON_COMPACT (-2.5%) | YAML (-4.8%) | XML_PRETTY (-1.6%) |
| YAML (-0.6%) | XML_PRETTY (-7.4%) | CSV (-6.3%) | CSV (-3.2%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| CSV ≈ 100% | JSON_COMPACT ≈ 100% | JSON_PRETTY ≈ 100% | CSV ≈ 92% |
| JSON_COMPACT (0.0%) | XML_PRETTY (0.0%) | TOON_DEFAULT (0.0%) | JSON_COMPACT (-1.6%) |
| JSON_PRETTY (0.0%) | JSON_PRETTY (-1.2%) | YAML (0.0%) | JSON_PRETTY (-1.6%) |
| TOON_DEFAULT (0.0%) | TOON_DEFAULT (-1.2%) | JSON_COMPACT (-1.6%) | TOON_DEFAULT (-1.6%) |
| XML_COMPACT (0.0%) | CSV (-2.5%) | XML_PRETTY (-1.6%) | XML_PRETTY (-1.6%) |
| XML_PRETTY (0.0%) | YAML (-2.5%) | XML_COMPACT (-3.2%) | YAML (-3.2%) |
| YAML (0.0%) | XML_COMPACT (-3.7%) | CSV (-6.4%) | XML_COMPACT (-4.8%) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Char/Token | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6748 | 291 | 7039 | 1.496 | 1.394 | 2.344 | 98.12 | 88.77 | 6906.340 | 132.327 | 88.80 | 88.77 |
| CSV | opt | 3633 | 292 | 3925 | 2.636 | 2.472 | 2.355 | 97.04 | 97.85 | 3808.820 | 116.180 | 97.90 | 97.85 |
| JSON_COMPACT | man | 9027 | 300 | 9327 | 2.206 | 1.058 | 2.419 | 98.66 | 81.93 | 9202.018 | 124.982 | 81.93 | 81.93 |
| JSON_COMPACT | opt | 5669 | 300 | 5969 | 3.261 | 1.644 | 2.417 | 98.12 | 92.44 | 5856.456 | 112.211 | 92.18 | 92.44 |
| JSON_PRETTY | man | 11204 | 203 | 11407 | 2.160 | 0.872 | 1.634 | 99.46 | 75.92 | 11345.071 | 61.596 | 75.91 | 75.92 |
| JSON_PRETTY | opt | 10288 | 299 | 10587 | 2.183 | 0.927 | 2.414 | 98.12 | 77.80 | 10388.291 | 199.042 | 77.56 | 77.80 |
| TOON_DEFAULT | man | 6798 | 301 | 7099 | 1.495 | 1.393 | 2.430 | 98.92 | 89.18 | 7022.660 | 76.673 | 89.17 | 89.18 |
| TOON_DEFAULT | opt | 8487 | 301 | 8788 | 2.313 | 1.117 | 2.425 | 98.12 | 83.49 | 8622.459 | 165.208 | 83.26 | 83.49 |
| XML_COMPACT | man | 11444 | 298 | 11742 | 2.417 | 0.845 | 2.403 | 99.19 | 74.60 | 11646.890 | 95.110 | 74.66 | 74.60 |
| XML_COMPACT | opt | 7851 | 296 | 8147 | 3.258 | 1.184 | 2.387 | 96.50 | 84.27 | 7861.855 | 285.145 | 84.15 | 84.27 |
| XML_PRETTY | man | 13087 | 296 | 13383 | 2.389 | 0.727 | 2.384 | 97.31 | 67.78 | 13022.673 | 359.994 | 68.15 | 67.78 |
| XML_PRETTY | opt | 12018 | 204 | 12222 | 2.407 | 0.803 | 1.645 | 98.12 | 72.64 | 11992.226 | 229.774 | 72.39 | 72.64 |
| YAML | man | 9479 | 198 | 9677 | 2.203 | 1.020 | 1.594 | 98.66 | 80.77 | 9547.000 | 129.667 | 80.82 | 80.77 |
| YAML | opt | 8696 | 198 | 8894 | 2.232 | 1.097 | 1.594 | 97.58 | 82.76 | 8678.440 | 215.227 | 82.55 | 82.76 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Accuracy Man (%) | Accuracy Opt (%) | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7039 | 3925 | -3114 | -44.24 | 98.12 | 97.04 | -1.08 | 98.08 | 96.97 | -1.11 | 88.80 | 97.90 |  +9.10 | 88.77 | 97.85 |  +9.08 |
| JSON_COMPACT | 9327 | 5969 | -3358 | -36.00 | 98.66 | 98.12 | -0.54 | 98.66 | 98.48 | -0.18 | 81.93 | 92.18 |  +10.25 | 81.93 | 92.44 |  +10.50 |
| JSON_PRETTY | 11407 | 10588 | -819 | -7.18 | 99.46 | 98.12 | -1.34 | 99.47 | 98.45 | -1.02 | 75.91 | 77.56 |  +1.66 | 75.92 | 77.80 |  +1.88 |
| TOON_DEFAULT | 7099 | 8787 |  +1688 |  +23.78 | 98.92 | 98.12 | -0.80 | 98.94 | 98.45 | -0.49 | 89.17 | 83.26 | -5.91 | 89.18 | 83.49 | -5.69 |
| XML_COMPACT | 11742 | 8147 | -3595 | -30.62 | 99.19 | 96.50 | -2.69 | 99.11 | 96.67 | -2.44 | 74.66 | 84.15 |  +9.50 | 74.60 | 84.27 |  +9.67 |
| XML_PRETTY | 13383 | 12222 | -1161 | -8.68 | 97.31 | 98.12 |  +0.81 | 96.78 | 98.48 |  +1.70 | 68.15 | 72.39 |  +4.24 | 67.78 | 72.64 |  +4.86 |
| YAML | 9677 | 8894 | -783 | -8.09 | 98.66 | 97.58 | -1.08 | 98.58 | 97.89 | -0.69 | 80.82 | 82.55 |  +1.72 | 80.77 | 82.76 |  +2.00 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 31 | 217.677 | 1.00 | 343819 | 0.001 | 2772.73 | 343850 | 217.678 | 2218.38 |
| CSV | opt | 58 | 62.638 | 1.87 | 362068 | 0.001 | 2919.90 | 362126 | 62.639 | 2336.29 |
| JSON_COMPACT | man | 28 | 322.393 | 0.90 | 289898 | 0.001 | 2337.88 | 289926 | 322.394 | 1870.49 |
| JSON_COMPACT | opt | 32 | 177.156 | 1.03 | 291539 | 0.001 | 2351.12 | 291571 | 177.157 | 1881.10 |
| JSON_PRETTY | man | 20 | 560.200 | 0.65 | 272696 | 0.001 | 2199.16 | 272716 | 560.201 | 1759.46 |
| JSON_PRETTY | opt | 18 | 571.556 | 0.58 | 284970 | 0.001 | 2298.15 | 284988 | 571.557 | 1838.63 |
| TOON_DEFAULT | man | 36 | 188.833 | 1.16 | 334156 | 0.001 | 2694.81 | 334192 | 188.834 | 2156.08 |
| TOON_DEFAULT | opt | 71 | 119.535 | 2.29 | 311501 | 0.001 | 2512.11 | 311572 | 119.536 | 2010.14 |
| XML_COMPACT | man | 36 | 317.889 | 1.16 | 332635 | 0.001 | 2682.54 | 332671 | 317.890 | 2146.26 |
| XML_COMPACT | opt | 31 | 253.258 | 1.00 | 333716 | 0.001 | 2691.26 | 333747 | 253.259 | 2153.20 |
| XML_PRETTY | man | 37 | 353.703 | 1.19 | 300920 | 0.001 | 2426.78 | 300957 | 353.704 | 1941.66 |
| XML_PRETTY | opt | 37 | 324.811 | 1.19 | 280701 | 0.001 | 2263.72 | 280738 | 324.812 | 1811.21 |
| YAML | man | 10 | 947.900 | 0.32 | 317843 | 0.001 | 2563.25 | 317853 | 947.901 | 2050.66 |
| YAML | opt | 23 | 378.087 | 0.74 | 326573 | 0.001 | 2633.65 | 326596 | 378.088 | 2107.07 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) | Total Man (s) | Total Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 31 | 58 |  +27 |  +87.10 | 343.82 | 362.07 |  +18.25 |  +5.31 | 343.85 | 362.13 |  +18.28 |  +5.32 |
| JSON_COMPACT | 28 | 32 |  +4 |  +14.29 | 289.90 | 291.54 |  +1.64 |  +0.57 | 289.93 | 291.57 |  +1.65 |  +0.57 |
| JSON_PRETTY | 20 | 18 | -2 | -10.00 | 272.70 | 284.97 |  +12.27 |  +4.50 | 272.72 | 284.99 |  +12.27 |  +4.50 |
| TOON_DEFAULT | 36 | 71 |  +35 |  +97.22 | 334.16 | 311.50 | -22.66 | -6.78 | 334.19 | 311.57 | -22.62 | -6.77 |
| XML_COMPACT | 36 | 31 | -5 | -13.89 | 332.63 | 333.72 |  +1.08 |  +0.32 | 332.67 | 333.75 |  +1.08 |  +0.32 |
| XML_PRETTY | 37 | 37 | 0 | 0.00 | 300.92 | 280.70 | -20.22 | -6.72 | 300.96 | 280.74 | -20.22 | -6.72 |
| YAML | 10 | 23 |  +13 |  +130.00 | 317.84 | 326.57 |  +8.73 |  +2.75 | 317.85 | 326.60 |  +8.74 |  +2.75 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Char/Token | Token/Value | Token/Object | Info/Token |
|---|---|---|---|---|---|
| CSV | man | 1.496 | 9.894 | 217.677 | 1.394 |
| CSV | opt | 2.636 | 5.758 | 117.194 | 2.472 |
| JSON_COMPACT | man | 2.206 | 13.236 | 291.194 | 1.058 |
| JSON_COMPACT | opt | 3.261 | 8.984 | 182.871 | 1.644 |
| JSON_PRETTY | man | 2.160 | 16.428 | 361.419 | 0.872 |
| JSON_PRETTY | opt | 2.183 | 16.304 | 331.871 | 0.927 |
| TOON_DEFAULT | man | 1.495 | 9.968 | 219.290 | 1.393 |
| TOON_DEFAULT | opt | 2.313 | 13.450 | 273.774 | 1.117 |
| XML_COMPACT | man | 2.417 | 16.780 | 369.161 | 0.845 |
| XML_COMPACT | opt | 3.258 | 12.442 | 253.258 | 1.184 |
| XML_PRETTY | man | 2.389 | 19.189 | 422.161 | 0.727 |
| XML_PRETTY | opt | 2.407 | 19.046 | 387.677 | 0.803 |
| YAML | man | 2.203 | 13.899 | 305.774 | 1.020 |
| YAML | opt | 2.232 | 13.781 | 280.516 | 1.097 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.496 | 2.636 |  +1.140 |  +76.20 | 9.894 | 5.758 | -4.136 | -41.80 | 217.677 | 117.194 | -100.483 | -46.16 | 1.394 | 2.472 |  +1.078 |  +77.33 |
| JSON_COMPACT | 2.206 | 3.261 |  +1.055 |  +47.82 | 13.236 | 8.984 | -4.252 | -32.12 | 291.194 | 182.871 | -108.323 | -37.20 | 1.058 | 1.644 |  +0.586 |  +55.39 |
| JSON_PRETTY | 2.160 | 2.183 |  +0.023 |  +1.06 | 16.428 | 16.304 | -0.124 | -0.75 | 361.419 | 331.871 | -29.548 | -8.18 | 0.872 | 0.927 |  +0.055 |  +6.31 |
| TOON_DEFAULT | 1.495 | 2.313 |  +0.818 |  +54.72 | 9.968 | 13.450 |  +3.482 |  +34.93 | 219.290 | 273.774 |  +54.484 |  +24.85 | 1.393 | 1.117 | -0.276 | -19.81 |
| XML_COMPACT | 2.417 | 3.258 |  +0.841 |  +34.80 | 16.780 | 12.442 | -4.338 | -25.85 | 369.161 | 253.258 | -115.903 | -31.40 | 0.845 | 1.184 |  +0.339 |  +40.12 |
| XML_PRETTY | 2.389 | 2.407 |  +0.018 |  +0.75 | 19.189 | 19.046 | -0.143 | -0.75 | 422.161 | 387.677 | -34.484 | -8.17 | 0.727 | 0.803 |  +0.076 |  +10.45 |
| YAML | 2.203 | 2.232 |  +0.029 |  +1.32 | 13.899 | 13.781 | -0.118 | -0.85 | 305.774 | 280.516 | -25.258 | -8.26 | 1.020 | 1.097 |  +0.077 |  +7.55 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| CSV | man | 7039 | 6906 | 132 | 98.12 | 98.08 | 88.80 | 88.77 |
| CSV | opt | 3925 | 3809 | 116 | 97.04 | 96.97 | 97.90 | 97.85 |
| JSON_COMPACT | man | 9327 | 9202 | 125 | 98.66 | 98.66 | 81.93 | 81.93 |
| JSON_COMPACT | opt | 5969 | 5856 | 112 | 98.12 | 98.48 | 92.18 | 92.44 |
| JSON_PRETTY | man | 11407 | 11345 | 62 | 99.46 | 99.47 | 75.91 | 75.92 |
| JSON_PRETTY | opt | 10587 | 10388 | 199 | 98.12 | 98.45 | 77.56 | 77.80 |
| TOON_DEFAULT | man | 7099 | 7023 | 77 | 98.92 | 98.94 | 89.17 | 89.18 |
| TOON_DEFAULT | opt | 8788 | 8622 | 165 | 98.12 | 98.45 | 83.26 | 83.49 |
| XML_COMPACT | man | 11742 | 11647 | 95 | 99.19 | 99.11 | 74.66 | 74.60 |
| XML_COMPACT | opt | 8147 | 7862 | 285 | 96.50 | 96.67 | 84.15 | 84.27 |
| XML_PRETTY | man | 13383 | 13023 | 360 | 97.31 | 96.78 | 68.15 | 67.78 |
| XML_PRETTY | opt | 12222 | 11992 | 230 | 98.12 | 98.48 | 72.39 | 72.64 |
| YAML | man | 9677 | 9547 | 130 | 98.66 | 98.58 | 80.82 | 80.77 |
| YAML | opt | 8894 | 8678 | 215 | 97.58 | 97.89 | 82.55 | 82.76 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7039 | 3925 | -3114 | -44.23 | 6906 | 3808 | -3098 | -44.85 | 132 | 116 | -16 | -12.23 | 98.12 | 97.04 | -1.08 | 88.80 | 97.896 |  +9.10 |  +10.25 |
| JSON_COMPACT | 9327 | 5969 | -3358 | -36.01 | 9202 | 5856 | -3346 | -36.36 | 125 | 112 | -13 | -10.22 | 98.66 | 98.12 | -0.54 | 81.93 | 92.183 |  +10.25 |  +12.51 |
| JSON_PRETTY | 11407 | 10588 | -819 | -7.18 | 11345 | 10388 | -957 | -8.43 | 62 | 199 |  +137 |  +221.69 | 99.46 | 98.12 | -1.34 | 75.91 | 77.564 |  +1.66 |  +2.18 |
| TOON_DEFAULT | 7099 | 8787 |  +1688 |  +23.78 | 7023 | 8623 |  +1600 |  +22.78 | 77 | 166 |  +89 |  +114.98 | 98.92 | 98.12 | -0.80 | 89.17 | 83.26 | -5.91 | -6.62 |
| XML_COMPACT | 11742 | 8147 | -3595 | -30.62 | 11647 | 7862 | -3785 | -32.50 | 95 | 285 |  +190 |  +200.04 | 99.19 | 96.50 | -2.69 | 74.66 | 84.154 |  +9.50 |  +12.72 |
| XML_PRETTY | 13383 | 12222 | -1161 | -8.67 | 13023 | 11993 | -1030 | -7.91 | 360 | 230 | -130 | -36.17 | 97.31 | 98.12 |  +0.81 | 68.15 | 72.39 |  +4.24 |  +6.22 |
| YAML | 9677 | 8894 | -783 | -8.09 | 9547 | 8678 | -869 | -9.10 | 130 | 216 |  +86 |  +65.82 | 98.66 | 97.58 | -1.08 | 80.82 | 82.547 |  +1.72 |  +2.13 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| CSV | man | 122 | 2 | 0 | 98.12 |
| CSV | opt | 120 | 4 | 0 | 97.04 |
| JSON_COMPACT | man | 122 | 2 | 0 | 98.66 |
| JSON_COMPACT | opt | 122 | 2 | 0 | 98.12 |
| JSON_PRETTY | man | 123 | 1 | 0 | 99.46 |
| JSON_PRETTY | opt | 122 | 2 | 0 | 98.12 |
| TOON_DEFAULT | man | 123 | 1 | 0 | 98.92 |
| TOON_DEFAULT | opt | 122 | 2 | 0 | 98.12 |
| XML_COMPACT | man | 123 | 1 | 0 | 99.19 |
| XML_COMPACT | opt | 120 | 4 | 0 | 96.50 |
| XML_PRETTY | man | 121 | 3 | 0 | 97.31 |
| XML_PRETTY | opt | 122 | 2 | 0 | 98.12 |
| YAML | man | 122 | 2 | 0 | 98.66 |
| YAML | opt | 121 | 3 | 0 | 97.58 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 122 | 120 | -2 | -1.64 | 2 | 4 |  +2 |  +100.00 | 0 | 0 | 0 | 0.00 | 98.12 | 97.04 | -1.08 |
| JSON_COMPACT | 122 | 122 | 0 | 0.00 | 2 | 2 | 0 | 0.00 | 0 | 0 | 0 | 0.00 | 98.66 | 98.12 | -0.54 |
| JSON_PRETTY | 123 | 122 | -1 | -0.81 | 1 | 2 |  +1 |  +100.00 | 0 | 0 | 0 | 0.00 | 99.46 | 98.12 | -1.34 |
| TOON_DEFAULT | 123 | 122 | -1 | -0.81 | 1 | 2 |  +1 |  +100.00 | 0 | 0 | 0 | 0.00 | 98.92 | 98.12 | -0.80 |
| XML_COMPACT | 123 | 120 | -3 | -2.44 | 1 | 4 |  +3 |  +300.00 | 0 | 0 | 0 | 0.00 | 99.19 | 96.50 | -2.69 |
| XML_PRETTY | 121 | 122 |  +1 |  +0.83 | 3 | 2 | -1 | -33.33 | 0 | 0 | 0 | 0.00 | 97.31 | 98.12 |  +0.81 |
| YAML | 122 | 121 | -1 | -0.82 | 2 | 3 |  +1 |  +50.00 | 0 | 0 | 0 | 0.00 | 98.66 | 97.58 | -1.08 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| CSV | man | 98.12 | 100.00 | 100.00 | 93.65 | 95.24 |
| CSV | opt | 97.04 | 100.00 | 97.53 | 93.65 | 92.07 |
| JSON_COMPACT | man | 98.66 | 99.39 | 97.53 | 100.00 | 96.83 |
| JSON_COMPACT | opt | 98.12 | 100.00 | 100.00 | 98.41 | 90.48 |
| JSON_PRETTY | man | 99.46 | 100.00 | 100.00 | 98.41 | 98.41 |
| JSON_PRETTY | opt | 98.12 | 100.00 | 98.77 | 100.00 | 90.48 |
| TOON_DEFAULT | man | 98.92 | 100.00 | 100.00 | 96.83 | 96.83 |
| TOON_DEFAULT | opt | 98.12 | 100.00 | 98.77 | 100.00 | 90.48 |
| XML_COMPACT | man | 99.19 | 100.00 | 98.77 | 98.41 | 98.41 |
| XML_COMPACT | opt | 96.50 | 100.00 | 96.30 | 96.83 | 87.30 |
| XML_PRETTY | man | 97.31 | 100.00 | 92.59 | 96.83 | 96.83 |
| XML_PRETTY | opt | 98.12 | 100.00 | 100.00 | 98.41 | 90.48 |
| YAML | man | 98.66 | 99.39 | 100.00 | 95.24 | 98.41 |
| YAML | opt | 97.58 | 100.00 | 97.53 | 100.00 | 88.89 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 100.00 | 100.00 | 0.00 |
| JSON_COMPACT | 99.39 | 100.00 |  +0.61 |
| JSON_PRETTY | 100.00 | 100.00 | 0.00 |
| TOON_DEFAULT | 100.00 | 100.00 | 0.00 |
| XML_COMPACT | 100.00 | 100.00 | 0.00 |
| XML_PRETTY | 100.00 | 100.00 | 0.00 |
| YAML | 99.39 | 100.00 |  +0.61 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 100.00 | 97.53 | -2.47 |
| JSON_COMPACT | 97.53 | 100.00 |  +2.47 |
| JSON_PRETTY | 100.00 | 98.77 | -1.23 |
| TOON_DEFAULT | 100.00 | 98.77 | -1.23 |
| XML_COMPACT | 98.77 | 96.30 | -2.47 |
| XML_PRETTY | 92.59 | 100.00 |  +7.41 |
| YAML | 100.00 | 97.53 | -2.47 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 93.65 | 93.65 | -0.00 |
| JSON_COMPACT | 100.00 | 98.41 | -1.59 |
| JSON_PRETTY | 98.41 | 100.00 |  +1.59 |
| TOON_DEFAULT | 96.83 | 100.00 |  +3.17 |
| XML_COMPACT | 98.41 | 96.83 | -1.59 |
| XML_PRETTY | 96.83 | 98.41 |  +1.59 |
| YAML | 95.24 | 100.00 |  +4.76 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 95.24 | 92.07 | -3.17 |
| JSON_COMPACT | 96.83 | 90.48 | -6.35 |
| JSON_PRETTY | 98.41 | 90.48 | -7.93 |
| TOON_DEFAULT | 96.83 | 90.48 | -6.35 |
| XML_COMPACT | 98.41 | 87.30 | -11.11 |
| XML_PRETTY | 96.83 | 90.48 | -6.35 |
| YAML | 98.41 | 88.89 | -9.52 |

## 3. Format-Specific Analysis
### 3.1 Detailed Analysis: CSV

#### 3.1.1 Performance Summary

- Token Duration Range: 344 - 362 seconds
- Token Cost Range: 3925 - 7039 tokens
- Wasted Token Range: 116 - 132 tokens
- Accuracy Range: 97.04 - 98.12%
- Efficiency Score Range: 88.80 - 97.90

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

- Token Duration Range: 290 - 292 seconds
- Token Cost Range: 5969 - 9327 tokens
- Wasted Token Range: 112 - 125 tokens
- Accuracy Range: 98.12 - 98.66%
- Efficiency Score Range: 81.93 - 92.18

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

- Token Duration Range: 273 - 285 seconds
- Token Cost Range: 10587 - 11407 tokens
- Wasted Token Range: 62 - 199 tokens
- Accuracy Range: 98.12 - 99.46%
- Efficiency Score Range: 75.91 - 77.56

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

- Token Duration Range: 312 - 334 seconds
- Token Cost Range: 7099 - 8788 tokens
- Wasted Token Range: 77 - 165 tokens
- Accuracy Range: 98.12 - 98.92%
- Efficiency Score Range: 83.26 - 89.17

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

- Token Duration Range: 333 - 334 seconds
- Token Cost Range: 8147 - 11742 tokens
- Wasted Token Range: 95 - 285 tokens
- Accuracy Range: 96.50 - 99.19%
- Efficiency Score Range: 74.66 - 84.15

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

- Token Duration Range: 281 - 301 seconds
- Token Cost Range: 12222 - 13383 tokens
- Wasted Token Range: 230 - 360 tokens
- Accuracy Range: 97.31 - 98.12%
- Efficiency Score Range: 68.15 - 72.39

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

- Token Duration Range: 318 - 327 seconds
- Token Cost Range: 8894 - 9677 tokens
- Wasted Token Range: 130 - 215 tokens
- Accuracy Range: 97.58 - 98.66%
- Efficiency Score Range: 80.82 - 82.55

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
- **Thinking**: off
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