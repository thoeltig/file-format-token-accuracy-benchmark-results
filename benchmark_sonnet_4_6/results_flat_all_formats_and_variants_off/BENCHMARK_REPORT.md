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
- **Read Tokens**: For each data file a single read subagent is invoked with the only prompt to read the file at the provided filepath and return "Done" once finished and do nothing more. The token extraction script searches for the read tool result and extracts only the read tokens of it.
- **Output Tokens**: For each data file three "benchmark-full-test" subagent are invoked with data, questions and answers template files and the instructions to read everything and answer all questions in a single write tool use. The token extraction script aggregates all output tokens until and including the write tool result.

## 2. Results

### 2.1 TLDR: Token Efficiency Analysis

*Note: All columns ranked best-to-worst. ↑ = lower value is better (ascending). ↓ = higher value is better (descending).*

#### 2.1.1 Best results

- Lowest total token cost:
   - Optional: JSON_COMPACT 28450 tokens
   - Mandatory: TOON_DEFAULT 31802 tokens
- Lowest read token cost:
   - Optional: CSV 3633 tokens
   - Mandatory: CSV 6748 tokens
- Lowest output token cost:
   - Optional: XML_PRETTY 22243 tokens
   - Mandatory: JSON_PRETTY 20772 tokens
- Lowest output token cost drift:
   - Optional: TOON_DEFAULT ↓ -5.04% ↑ 2.85%
   - Mandatory: CSV ↓ -2.35% ↑ 3.05%
- Highest accuracy:
   - Optional: JSON_COMPACT 98.12%
   - Mandatory: JSON_PRETTY 99.46%
- Lowest accuracy drift:
   - Optional: JSON_COMPACT ↓ -0.55% ↑ 0.28%
   - Mandatory: XML_COMPACT ↓ 0.00% ↑ 0.00%
- Most useful tokens:
   - Optional: XML_PRETTY 33617 / 34261 tokens
   - Mandatory: XML_COMPACT 37409 / 37714 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 98.65
   - Mandatory: TOON_DEFAULT 88.38
- Lowest delta (optional-mandatory):
   - Total tokens: YAML 339 tokens
   - Accuracy: JSON_COMPACT -0.54%
   - Token efficiency: YAML -1.85

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_COMPACT 34591 tokens
   - Mandatory: XML_COMPACT 37714 tokens
- Highest read token cost:
   - Optional: XML_PRETTY 12018 tokens
   - Mandatory: XML_PRETTY 13087 tokens
- Highest output token cost:
   - Optional: CSV 28947 tokens
   - Mandatory: CSV 28443 tokens
- Highest output token drift:
   - Optional: XML_PRETTY ↓ -12.15% ↑ 20.28%
   - Mandatory: JSON_COMPACT ↓ -18.55% ↑ 35.79%
- Lowest accuracy:
   - Optional: XML_COMPACT 96.50%
   - Mandatory: XML_PRETTY 97.31%
- Highest accuracy drift:
   - Optional: CSV ↓ -2.77% ↑ 2.22%
   - Mandatory: JSON_COMPACT ↓ -2.73% ↑ 1.36%
- Most wasted tokens:
   - Optional: XML_COMPACT 1211 / 34591 tokens
   - Mandatory: XML_PRETTY 976 / 36293 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_COMPACT 77.67
   - Mandatory: XML_COMPACT 69.47
- Highest delta (optional-mandatory):
   - Total tokens: JSON_COMPACT -3771 tokens
   - Accuracy: XML_COMPACT -2.69%
   - Token efficiency: JSON_COMPACT 11.81

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 273s | CSV ≈ 6748 | JSON_PRETTY ≈ 20772 | TOON_DEFAULT ≈ 31802 | JSON_PRETTY ≈ 173 | JSON_PRETTY ≈ 99% | JSON_PRETTY ≈ 99% | TOON_DEFAULT ≈ 88 | TOON_DEFAULT ≈ 88 |
| JSON_COMPACT (+6.3%) | TOON_DEFAULT (+0.7%) | JSON_COMPACT (+11.7%) | JSON_PRETTY (+0.5%) | XML_COMPACT (+76.9%) | XML_COMPACT (-0.3%) | XML_COMPACT (-0.4%) | JSON_PRETTY (-0.2%) | JSON_PRETTY (-0.2%) |
| XML_PRETTY (+10.4%) | JSON_COMPACT (+33.8%) | XML_PRETTY (+11.7%) | JSON_COMPACT (+1.3%) | TOON_DEFAULT (+98.9%) | TOON_DEFAULT (-0.5%) | TOON_DEFAULT (-0.5%) | JSON_COMPACT (-1.7%) | JSON_COMPACT (-1.8%) |
| YAML (+16.6%) | YAML (+40.5%) | YAML (+15.0%) | YAML (+4.9%) | JSON_COMPACT (+150.1%) | JSON_COMPACT (-0.8%) | JSON_COMPACT (-0.8%) | YAML (-5.9%) | YAML (-6.0%) |
| XML_COMPACT (+22.0%) | JSON_PRETTY (+66.0%) | TOON_DEFAULT (+20.4%) | CSV (+10.7%) | YAML (+159.0%) | YAML (-0.8%) | YAML (-0.9%) | CSV (-13.0%) | CSV (-13.1%) |
| TOON_DEFAULT (+22.5%) | XML_COMPACT (+69.6%) | XML_COMPACT (+26.5%) | XML_PRETTY (+14.1%) | CSV (+283.2%) | CSV (-1.3%) | CSV (-1.4%) | XML_PRETTY (-17.7%) | XML_PRETTY (-18.1%) |
| CSV (+26.1%) | XML_PRETTY (+93.9%) | CSV (+36.9%) | XML_COMPACT (+18.6%) | XML_PRETTY (+465.4%) | XML_PRETTY (-2.1%) | XML_PRETTY (-2.7%) | XML_COMPACT (-21.4%) | XML_COMPACT (-21.5%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| XML_PRETTY ≈ 281s | CSV ≈ 3633 | XML_PRETTY ≈ 22243 | JSON_COMPACT ≈ 28450 | JSON_COMPACT ≈ 535 | JSON_COMPACT ≈ 98% | JSON_COMPACT ≈ 98% | JSON_COMPACT ≈ 99 | JSON_COMPACT ≈ 99 |
| JSON_PRETTY (+1.5%) | JSON_COMPACT (+56.0%) | JSON_PRETTY (+0.9%) | CSV (+14.5%) | JSON_PRETTY (+15.1%) | JSON_PRETTY (0.0%) | XML_PRETTY (0.0%) | JSON_PRETTY (-14.0%) | JSON_PRETTY (-14.0%) |
| JSON_COMPACT (+3.9%) | XML_COMPACT (+116.1%) | JSON_COMPACT (+2.4%) | JSON_PRETTY (+15.1%) | TOON_DEFAULT (+19.8%) | TOON_DEFAULT (0.0%) | JSON_PRETTY (-0.0%) | CSV (-14.3%) | CSV (-14.6%) |
| TOON_DEFAULT (+11.0%) | TOON_DEFAULT (+133.6%) | YAML (+12.5%) | YAML (+18.5%) | XML_PRETTY (+20.4%) | XML_PRETTY (0.0%) | TOON_DEFAULT (-0.0%) | YAML (-17.6%) | YAML (-17.6%) |
| YAML (+16.3%) | YAML (+139.4%) | TOON_DEFAULT (+15.1%) | TOON_DEFAULT (+19.8%) | YAML (+52.5%) | YAML (-0.5%) | YAML (-0.6%) | TOON_DEFAULT (-18.5%) | TOON_DEFAULT (-18.4%) |
| XML_COMPACT (+18.9%) | JSON_PRETTY (+183.2%) | XML_COMPACT (+20.2%) | XML_PRETTY (+20.4%) | CSV (+80.3%) | CSV (-1.1%) | CSV (-1.5%) | XML_PRETTY (-19.0%) | XML_PRETTY (-19.0%) |
| CSV (+29.0%) | XML_PRETTY (+230.8%) | CSV (+30.1%) | XML_COMPACT (+21.6%) | XML_COMPACT (+126.4%) | XML_COMPACT (-1.6%) | XML_COMPACT (-1.8%) | XML_COMPACT (-21.3%) | XML_COMPACT (-21.3%) |


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
| CSV | man | 6748 | 28443 | 35191 | 1.496 | 0.279 | 229.379 | 98.12 | 76.84 | 34529.409 | 661.591 | 76.87 | 76.84 |
| CSV | opt | 3633 | 28947 | 32580 | 2.636 | 0.298 | 233.446 | 97.04 | 84.50 | 31615.955 | 964.378 | 84.55 | 84.50 |
| JSON_COMPACT | man | 9027 | 23194 | 32221 | 2.206 | 0.306 | 187.048 | 98.66 | 86.84 | 31789.239 | 431.761 | 86.84 | 86.84 |
| JSON_COMPACT | opt | 5669 | 22781 | 28450 | 3.261 | 0.345 | 183.720 | 98.12 | 98.90 | 27915.467 | 534.866 | 98.65 | 98.90 |
| JSON_PRETTY | man | 11204 | 20772 | 31976 | 2.160 | 0.311 | 167.513 | 99.46 | 88.20 | 31802.998 | 172.669 | 88.20 | 88.20 |
| JSON_PRETTY | opt | 10288 | 22446 | 32734 | 2.183 | 0.300 | 181.019 | 98.12 | 85.04 | 32118.928 | 615.405 | 84.81 | 85.04 |
| TOON_DEFAULT | man | 6798 | 25004 | 31802 | 1.495 | 0.311 | 201.648 | 98.92 | 88.39 | 31458.868 | 343.465 | 88.38 | 88.39 |
| TOON_DEFAULT | opt | 8487 | 25602 | 34089 | 2.313 | 0.288 | 206.470 | 98.12 | 80.66 | 33448.454 | 640.879 | 80.43 | 80.66 |
| XML_COMPACT | man | 11444 | 26270 | 37714 | 2.417 | 0.263 | 211.855 | 99.19 | 69.41 | 37408.517 | 305.483 | 69.47 | 69.41 |
| XML_COMPACT | opt | 7851 | 26740 | 34591 | 3.258 | 0.279 | 215.645 | 96.50 | 77.79 | 33380.315 | 1210.685 | 77.67 | 77.79 |
| XML_PRETTY | man | 13087 | 23206 | 36293 | 2.389 | 0.268 | 187.142 | 97.31 | 72.37 | 35316.394 | 976.273 | 72.74 | 72.37 |
| XML_PRETTY | opt | 12018 | 22243 | 34261 | 2.407 | 0.286 | 179.379 | 98.12 | 80.13 | 33616.893 | 644.107 | 79.88 | 80.13 |
| YAML | man | 9479 | 23892 | 33371 | 2.203 | 0.296 | 192.677 | 98.66 | 83.07 | 32923.829 | 447.171 | 83.13 | 83.07 |
| YAML | opt | 8696 | 25014 | 33710 | 2.232 | 0.289 | 201.728 | 97.58 | 81.49 | 32894.543 | 815.790 | 81.28 | 81.49 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6748 | 3633 | -3115 | -46.16 | 28443 | 28947 |  +504 |  +1.77 | 35191 | 32580 | -2611 | -7.42 | 98.08 | 96.97 | -1.11 | 76.84 | 84.50 |  +7.66 |
| JSON_COMPACT | 9027 | 5669 | -3358 | -37.20 | 23194 | 22781 | -413 | -1.78 | 32221 | 28450 | -3771 | -11.70 | 98.66 | 98.48 | -0.18 | 86.84 | 98.90 |  +12.06 |
| JSON_PRETTY | 11204 | 10288 | -916 | -8.18 | 20772 | 22447 |  +1675 |  +8.06 | 31976 | 32735 |  +759 |  +2.37 | 99.47 | 98.45 | -1.02 | 88.20 | 85.04 | -3.17 |
| TOON_DEFAULT | 6798 | 8487 |  +1689 |  +24.85 | 25004 | 25602 |  +598 |  +2.39 | 31802 | 34089 |  +2287 |  +7.19 | 98.94 | 98.45 | -0.49 | 88.39 | 80.66 | -7.73 |
| XML_COMPACT | 11444 | 7851 | -3593 | -31.40 | 26270 | 26740 |  +470 |  +1.79 | 37714 | 34591 | -3123 | -8.28 | 99.11 | 96.67 | -2.44 | 69.41 | 77.79 |  +8.38 |
| XML_PRETTY | 13087 | 12018 | -1069 | -8.17 | 23206 | 22243 | -963 | -4.15 | 36293 | 34261 | -2032 | -5.60 | 96.78 | 98.48 |  +1.70 | 72.37 | 80.13 |  +7.76 |
| YAML | 9479 | 8696 | -783 | -8.26 | 23892 | 25014 |  +1122 |  +4.70 | 33371 | 33710 |  +339 |  +1.02 | 98.58 | 97.89 | -0.69 | 83.07 | 81.49 | -1.58 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 31 | 217.677 | 1.00 | 343819 | 0.083 | 2772.73 | 343850 | 217.760 | 2218.38 |
| CSV | opt | 58 | 62.638 | 1.87 | 362068 | 0.080 | 2919.90 | 362126 | 62.718 | 2336.29 |
| JSON_COMPACT | man | 28 | 322.393 | 0.90 | 289898 | 0.080 | 2337.88 | 289926 | 322.473 | 1870.49 |
| JSON_COMPACT | opt | 32 | 177.156 | 1.03 | 291539 | 0.078 | 2351.12 | 291571 | 177.234 | 1881.10 |
| JSON_PRETTY | man | 20 | 560.200 | 0.65 | 272696 | 0.076 | 2199.16 | 272716 | 560.276 | 1759.46 |
| JSON_PRETTY | opt | 18 | 571.556 | 0.58 | 284970 | 0.079 | 2298.15 | 284988 | 571.635 | 1838.63 |
| TOON_DEFAULT | man | 36 | 188.833 | 1.16 | 334156 | 0.075 | 2694.81 | 334192 | 188.908 | 2156.08 |
| TOON_DEFAULT | opt | 71 | 119.535 | 2.29 | 311501 | 0.082 | 2512.11 | 311572 | 119.617 | 2010.14 |
| XML_COMPACT | man | 36 | 317.889 | 1.16 | 332635 | 0.079 | 2682.54 | 332671 | 317.968 | 2146.26 |
| XML_COMPACT | opt | 31 | 253.258 | 1.00 | 333716 | 0.080 | 2691.26 | 333747 | 253.338 | 2153.20 |
| XML_PRETTY | man | 37 | 353.703 | 1.19 | 300920 | 0.077 | 2426.78 | 300957 | 353.780 | 1941.66 |
| XML_PRETTY | opt | 37 | 324.811 | 1.19 | 280701 | 0.079 | 2263.72 | 280738 | 324.890 | 1811.21 |
| YAML | man | 10 | 947.900 | 0.32 | 317843 | 0.075 | 2563.25 | 317853 | 947.975 | 2050.66 |
| YAML | opt | 23 | 378.087 | 0.74 | 326573 | 0.077 | 2633.65 | 326596 | 378.164 | 2107.07 |

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
| CSV | man | 1.496 | 9.894 | 217.677 | 0.279 |
| CSV | opt | 2.636 | 5.758 | 117.194 | 0.298 |
| JSON_COMPACT | man | 2.206 | 13.236 | 291.194 | 0.306 |
| JSON_COMPACT | opt | 3.261 | 8.984 | 182.871 | 0.345 |
| JSON_PRETTY | man | 2.160 | 16.428 | 361.419 | 0.311 |
| JSON_PRETTY | opt | 2.183 | 16.304 | 331.871 | 0.300 |
| TOON_DEFAULT | man | 1.495 | 9.968 | 219.290 | 0.311 |
| TOON_DEFAULT | opt | 2.313 | 13.450 | 273.774 | 0.288 |
| XML_COMPACT | man | 2.417 | 16.780 | 369.161 | 0.263 |
| XML_COMPACT | opt | 3.258 | 12.442 | 253.258 | 0.279 |
| XML_PRETTY | man | 2.389 | 19.189 | 422.161 | 0.268 |
| XML_PRETTY | opt | 2.407 | 19.046 | 387.677 | 0.286 |
| YAML | man | 2.203 | 13.899 | 305.774 | 0.296 |
| YAML | opt | 2.232 | 13.781 | 280.516 | 0.289 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.496 | 2.636 |  +1.140 |  +76.20 | 9.894 | 5.758 | -4.136 | -41.80 | 217.677 | 117.194 | -100.483 | -46.16 | 0.279 | 0.298 |  +0.019 |  +6.81 |
| JSON_COMPACT | 2.206 | 3.261 |  +1.055 |  +47.82 | 13.236 | 8.984 | -4.252 | -32.12 | 291.194 | 182.871 | -108.323 | -37.20 | 0.306 | 0.345 |  +0.039 |  +12.75 |
| JSON_PRETTY | 2.160 | 2.183 |  +0.023 |  +1.06 | 16.428 | 16.304 | -0.124 | -0.75 | 361.419 | 331.871 | -29.548 | -8.18 | 0.311 | 0.300 | -0.011 | -3.54 |
| TOON_DEFAULT | 1.495 | 2.313 |  +0.818 |  +54.72 | 9.968 | 13.450 |  +3.482 |  +34.93 | 219.290 | 273.774 |  +54.484 |  +24.85 | 0.311 | 0.288 | -0.023 | -7.40 |
| XML_COMPACT | 2.417 | 3.258 |  +0.841 |  +34.80 | 16.780 | 12.442 | -4.338 | -25.85 | 369.161 | 253.258 | -115.903 | -31.40 | 0.263 | 0.279 |  +0.016 |  +6.08 |
| XML_PRETTY | 2.389 | 2.407 |  +0.018 |  +0.75 | 19.189 | 19.046 | -0.143 | -0.75 | 422.161 | 387.677 | -34.484 | -8.17 | 0.268 | 0.286 |  +0.018 |  +6.72 |
| YAML | 2.203 | 2.232 |  +0.029 |  +1.32 | 13.899 | 13.781 | -0.118 | -0.85 | 305.774 | 280.516 | -25.258 | -8.26 | 0.296 | 0.289 | -0.007 | -2.36 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| CSV | man | 35191 | 34529 | 662 | 98.12 | 98.08 | 76.87 | 76.84 |
| CSV | opt | 32580 | 31616 | 964 | 97.04 | 96.97 | 84.55 | 84.50 |
| JSON_COMPACT | man | 32221 | 31789 | 432 | 98.66 | 98.66 | 86.84 | 86.84 |
| JSON_COMPACT | opt | 28450 | 27915 | 535 | 98.12 | 98.48 | 98.65 | 98.90 |
| JSON_PRETTY | man | 31976 | 31803 | 173 | 99.46 | 99.47 | 88.20 | 88.20 |
| JSON_PRETTY | opt | 32734 | 32119 | 615 | 98.12 | 98.45 | 84.81 | 85.04 |
| TOON_DEFAULT | man | 31802 | 31459 | 343 | 98.92 | 98.94 | 88.38 | 88.39 |
| TOON_DEFAULT | opt | 34089 | 33448 | 641 | 98.12 | 98.45 | 80.43 | 80.66 |
| XML_COMPACT | man | 37714 | 37409 | 305 | 99.19 | 99.11 | 69.47 | 69.41 |
| XML_COMPACT | opt | 34591 | 33380 | 1211 | 96.50 | 96.67 | 77.67 | 77.79 |
| XML_PRETTY | man | 36293 | 35316 | 976 | 97.31 | 96.78 | 72.74 | 72.37 |
| XML_PRETTY | opt | 34261 | 33617 | 644 | 98.12 | 98.48 | 79.88 | 80.13 |
| YAML | man | 33371 | 32924 | 447 | 98.66 | 98.58 | 83.13 | 83.07 |
| YAML | opt | 33710 | 32895 | 816 | 97.58 | 97.89 | 81.28 | 81.49 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 35191 | 32580 | -2611 | -7.42 | 34529 | 31616 | -2913 | -8.44 | 662 | 965 |  +303 |  +45.74 | 98.12 | 97.04 | -1.08 | 76.87 | 84.55 |  +7.68 |  +9.99 |
| JSON_COMPACT | 32221 | 28450 | -3771 | -11.70 | 31789 | 27915 | -3874 | -12.19 | 432 | 535 |  +103 |  +23.87 | 98.66 | 98.12 | -0.54 | 86.84 | 98.652 |  +11.81 |  +13.60 |
| JSON_PRETTY | 31976 | 32735 |  +759 |  +2.37 | 31803 | 32119 |  +316 |  +0.99 | 173 | 616 |  +443 |  +255.92 | 99.46 | 98.12 | -1.34 | 88.20 | 84.808 | -3.39 | -3.84 |
| TOON_DEFAULT | 31802 | 34089 |  +2287 |  +7.19 | 31459 | 33449 |  +1990 |  +6.32 | 343 | 640 |  +297 |  +86.71 | 98.92 | 98.12 | -0.80 | 88.38 | 80.429 | -7.95 | -9.00 |
| XML_COMPACT | 37714 | 34591 | -3123 | -8.28 | 37409 | 33381 | -4028 | -10.77 | 305 | 1210 |  +905 |  +296.79 | 99.19 | 96.50 | -2.69 | 69.47 | 77.674 |  +8.21 |  +11.82 |
| XML_PRETTY | 36293 | 34261 | -2032 | -5.60 | 35316 | 33616 | -1700 | -4.81 | 976 | 644 | -332 | -34.03 | 97.31 | 98.12 |  +0.81 | 72.74 | 79.875 |  +7.13 |  +9.81 |
| YAML | 33371 | 33710 |  +339 |  +1.02 | 32924 | 32895 | -29 | -0.09 | 447 | 816 |  +369 |  +82.47 | 98.66 | 97.58 | -1.08 | 83.13 | 81.276 | -1.85 | -2.23 |

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

- **Report Generated**: 2026-04-01
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code < 2.1.86
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)