# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: off
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
   - Optional: CSV 17841 tokens
   - Mandatory: CSV 17432 tokens
- Lowest read token cost:
   - Optional: CSV 6728 tokens
   - Mandatory: CSV 7022 tokens
- Lowest output token cost:
   - Optional: TOON_DEFAULT 9109 tokens
   - Mandatory: XML_COMPACT 7149 tokens
- Lowest output token cost drift:
   - Optional: YAML ↓ -17.44% ↑ 15.47%
   - Mandatory: JSON_PRETTY ↓ -6.93% ↑ 4.67%
- Highest accuracy:
   - Optional: YAML 80.38%
   - Mandatory: JSON_PRETTY 82.53%
- Lowest accuracy drift:
   - Optional: YAML ↓ -3.68% ↑ 3.33%
   - Mandatory: JSON_PRETTY ↓ -1.31% ↑ 1.62%
- Most useful tokens:
   - Optional: YAML 20078 / 24979 tokens
   - Mandatory: JSON_PRETTY 21829 / 26450 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 80.33
   - Mandatory: TOON_DEFAULT 79.14
- Lowest delta (optional-mandatory):
   - Total tokens: CSV 409 tokens
   - Accuracy: XML_PRETTY 0.27%
   - Token efficiency: CSV -2.76

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: YAML 24979 tokens
   - Mandatory: XML_PRETTY 26915 tokens
- Highest read token cost:
   - Optional: XML_PRETTY 15117 tokens
   - Mandatory: XML_PRETTY 16186 tokens
- Highest output token cost:
   - Optional: YAML 13188 tokens
   - Mandatory: JSON_COMPACT 13636 tokens
- Highest output token drift:
   - Optional: JSON_COMPACT ↓ -43.41% ↑ 58.21%
   - Mandatory: CSV ↓ -16.79% ↑ 39.99%
- Lowest accuracy:
   - Optional: CSV 65.48%
   - Mandatory: CSV 67.58%
- Highest accuracy drift:
   - Optional: CSV ↓ -16.25% ↑ 13.30%
   - Mandatory: TOON_DEFAULT ↓ -22.37% ↑ 8.04%
- Most wasted tokens:
   - Optional: CSV 6159 / 17841 tokens
   - Mandatory: XML_PRETTY 6295 / 26915 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 61.62
   - Mandatory: XML_PRETTY 53.66
- Highest delta (optional-mandatory):
   - Total tokens: JSON_COMPACT -4464 tokens
   - Accuracy: XML_COMPACT 5.11%
   - Token efficiency: JSON_COMPACT 13.10

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 50s | CSV ≈ 7022 | XML_COMPACT ≈ 7149 | CSV ≈ 17432 | TOON_DEFAULT ≈ 4370 | JSON_PRETTY ≈ 83% | JSON_PRETTY ≈ 82% | TOON_DEFAULT ≈ 79 | TOON_DEFAULT ≈ 79 |
| YAML (+50.6%) | TOON_DEFAULT (+2.3%) | YAML (+33.2%) | TOON_DEFAULT (+8.4%) | JSON_PRETTY (+5.7%) | JSON_COMPACT (-4.5%) | JSON_COMPACT (-4.4%) | CSV (-2.4%) | CSV (-2.3%) |
| XML_COMPACT (+59.3%) | JSON_COMPACT (+32.4%) | CSV (+45.6%) | XML_COMPACT (+9.6%) | JSON_COMPACT (+15.3%) | TOON_DEFAULT (-5.7%) | TOON_DEFAULT (-5.5%) | XML_COMPACT (-3.9%) | XML_COMPACT (-4.9%) |
| CSV (+60.7%) | XML_COMPACT (+70.2%) | XML_PRETTY (+50.1%) | YAML (+26.8%) | XML_COMPACT (+16.3%) | XML_PRETTY (-5.9%) | XML_PRETTY (-6.5%) | YAML (-13.7%) | YAML (-15.0%) |
| XML_PRETTY (+71.6%) | YAML (+79.1%) | TOON_DEFAULT (+63.9%) | JSON_COMPACT (+31.5%) | YAML (+22.3%) | YAML (-6.7%) | YAML (-8.0%) | JSON_COMPACT (-15.1%) | JSON_COMPACT (-15.2%) |
| JSON_PRETTY (+91.0%) | JSON_PRETTY (+103.8%) | JSON_PRETTY (+69.8%) | JSON_PRETTY (+51.7%) | CSV (+29.3%) | XML_COMPACT (-9.1%) | XML_COMPACT (-10.0%) | JSON_PRETTY (-25.1%) | JSON_PRETTY (-25.3%) |
| JSON_COMPACT (+111.6%) | XML_PRETTY (+130.5%) | JSON_COMPACT (+90.7%) | XML_PRETTY (+54.4%) | XML_PRETTY (+44.1%) | CSV (-15.0%) | CSV (-14.7%) | XML_PRETTY (-32.2%) | XML_PRETTY (-32.9%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 42s | CSV ≈ 6728 | TOON_DEFAULT ≈ 9109 | CSV ≈ 17841 | JSON_COMPACT ≈ 4320 | YAML ≈ 80% | YAML ≈ 80% | JSON_COMPACT ≈ 80 | JSON_COMPACT ≈ 79 |
| XML_PRETTY (+76.7%) | JSON_COMPACT (+30.3%) | XML_PRETTY (+2.5%) | JSON_COMPACT (+3.5%) | XML_COMPACT (+12.4%) | JSON_PRETTY (-1.7%) | XML_COMPACT (-1.4%) | CSV (-7.2%) | CSV (-6.2%) |
| JSON_COMPACT (+79.9%) | XML_COMPACT (+66.2%) | JSON_COMPACT (+6.5%) | TOON_DEFAULT (+16.7%) | YAML (+13.5%) | XML_COMPACT (-1.9%) | JSON_PRETTY (-1.7%) | TOON_DEFAULT (-9.9%) | TOON_DEFAULT (-9.5%) |
| JSON_PRETTY (+102.2%) | TOON_DEFAULT (+74.0%) | JSON_PRETTY (+16.1%) | XML_COMPACT (+26.6%) | TOON_DEFAULT (+16.1%) | XML_PRETTY (-3.5%) | XML_PRETTY (-3.3%) | XML_COMPACT (-14.5%) | XML_COMPACT (-13.7%) |
| CSV (+105.9%) | YAML (+75.3%) | CSV (+22.0%) | JSON_PRETTY (+34.4%) | JSON_PRETTY (+18.2%) | JSON_COMPACT (-3.8%) | JSON_COMPACT (-4.5%) | JSON_PRETTY (-19.8%) | JSON_PRETTY (-19.4%) |
| XML_COMPACT (+108.6%) | JSON_PRETTY (+99.1%) | XML_COMPACT (+25.2%) | XML_PRETTY (+37.1%) | XML_PRETTY (+30.9%) | TOON_DEFAULT (-4.5%) | TOON_DEFAULT (-4.7%) | YAML (-22.3%) | YAML (-21.9%) |
| YAML (+147.3%) | XML_PRETTY (+124.7%) | YAML (+44.8%) | YAML (+40.0%) | CSV (+42.6%) | CSV (-14.9%) | CSV (-14.4%) | XML_PRETTY (-23.3%) | XML_PRETTY (-22.8%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 98% | JSON_PRETTY ≈ 84% | JSON_PRETTY ≈ 67% | JSON_PRETTY ≈ 59% |
| YAML (-1.2%) | TOON_DEFAULT (-4.1%) | XML_PRETTY (-3.2%) | XML_COMPACT (-1.6%) |
| JSON_PRETTY (-1.2%) | JSON_COMPACT (-6.2%) | JSON_COMPACT (-3.6%) | CSV (-5.4%) |
| XML_PRETTY (-2.4%) | XML_PRETTY (-13.6%) | CSV (-4.8%) | TOON_DEFAULT (-9.5%) |
| TOON_DEFAULT (-5.9%) | XML_COMPACT (-16.1%) | YAML (-4.8%) | YAML (-9.5%) |
| XML_COMPACT (-9.7%) | CSV (-17.3%) | TOON_DEFAULT (-6.4%) | XML_PRETTY (-11.1%) |
| CSV (-22.5%) | YAML (-19.8%) | XML_COMPACT (-9.5%) | JSON_COMPACT (-18.3%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99% | XML_COMPACT ≈ 81% | YAML ≈ 73% | JSON_COMPACT ≈ 52% |
| XML_COMPACT (-1.8%) | XML_PRETTY (-1.2%) | JSON_PRETTY (-6.4%) | XML_PRETTY (0.0%) |
| JSON_PRETTY (-2.7%) | JSON_PRETTY (-5.9%) | CSV (-10.2%) | TOON_DEFAULT (-4.8%) |
| JSON_COMPACT (-4.5%) | YAML (-7.4%) | JSON_COMPACT (-11.1%) | JSON_PRETTY (-4.8%) |
| TOON_DEFAULT (-5.9%) | TOON_DEFAULT (-7.8%) | XML_COMPACT (-11.1%) | YAML (-6.3%) |
| XML_PRETTY (-7.9%) | JSON_COMPACT (-11.9%) | TOON_DEFAULT (-12.2%) | XML_COMPACT (-11.1%) |
| CSV (-20.1%) | CSV (-19.3%) | XML_PRETTY (-14.3%) | CSV (-16.2%) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Char/Token | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7022 | 10410 | 17432 | 1.438 | 0.388 | 83.955 | 67.58 | 77.13 | 11780.816 | 5651.584 | 77.27 | 77.13 |
| CSV | opt | 6728 | 11113 | 17841 | 1.423 | 0.367 | 89.624 | 65.48 | 74.52 | 11682.549 | 6158.851 | 74.51 | 74.52 |
| JSON_COMPACT | man | 9296 | 13636 | 22932 | 2.143 | 0.340 | 109.968 | 78.03 | 67.00 | 17893.840 | 5038.160 | 67.23 | 67.00 |
| JSON_COMPACT | opt | 8768 | 9700 | 18468 | 2.108 | 0.415 | 78.226 | 76.61 | 79.44 | 14148.335 | 4319.665 | 80.33 | 79.44 |
| JSON_PRETTY | man | 14311 | 12139 | 26450 | 1.691 | 0.312 | 97.892 | 82.53 | 58.96 | 21828.910 | 4620.757 | 59.27 | 58.96 |
| JSON_PRETTY | opt | 13395 | 10579 | 23974 | 1.677 | 0.328 | 85.313 | 78.71 | 64.02 | 18869.739 | 5104.011 | 64.41 | 64.02 |
| TOON_DEFAULT | man | 7182 | 11720 | 18902 | 1.415 | 0.407 | 94.517 | 76.88 | 78.97 | 14531.922 | 4370.162 | 79.14 | 78.97 |
| TOON_DEFAULT | opt | 11709 | 9109 | 20818 | 1.677 | 0.367 | 73.461 | 75.90 | 71.91 | 15800.989 | 5017.178 | 72.41 | 71.91 |
| XML_COMPACT | man | 11950 | 7149 | 19099 | 2.315 | 0.384 | 57.653 | 73.39 | 75.13 | 14016.756 | 5082.244 | 76.08 | 75.13 |
| XML_COMPACT | opt | 11181 | 11406 | 22587 | 2.288 | 0.348 | 91.984 | 78.50 | 68.59 | 17730.795 | 4856.205 | 68.64 | 68.59 |
| XML_PRETTY | man | 16186 | 10729 | 26915 | 1.931 | 0.285 | 86.522 | 76.61 | 52.96 | 20619.326 | 6295.341 | 53.66 | 52.96 |
| XML_PRETTY | opt | 15117 | 9335 | 24452 | 1.913 | 0.314 | 75.285 | 76.88 | 61.36 | 18798.954 | 5653.379 | 61.62 | 61.36 |
| YAML | man | 12574 | 9523 | 22097 | 1.661 | 0.343 | 76.798 | 75.81 | 67.13 | 16751.736 | 5345.264 | 68.31 | 67.13 |
| YAML | opt | 11791 | 13188 | 24979 | 1.646 | 0.322 | 106.352 | 80.38 | 62.03 | 20077.853 | 4900.814 | 62.41 | 62.03 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7022 | 6728 | -294 | -4.19 | 10410 | 11113 |  +703 |  +6.75 | 17432 | 17841 |  +409 |  +2.35 | 67.37 | 65.49 | -1.88 | 77.13 | 74.52 | -2.61 |
| JSON_COMPACT | 9296 | 8768 | -528 | -5.68 | 13636 | 9700 | -3936 | -28.86 | 22932 | 18468 | -4464 | -19.47 | 77.71 | 75.34 | -2.37 | 67.00 | 79.44 |  +12.44 |
| JSON_PRETTY | 14311 | 13395 | -916 | -6.40 | 12139 | 10579 | -1560 | -12.85 | 26450 | 23974 | -2476 | -9.36 | 82.08 | 78.15 | -3.93 | 58.96 | 64.02 |  +5.06 |
| TOON_DEFAULT | 7182 | 11709 |  +4527 |  +63.03 | 11720 | 9109 | -2611 | -22.28 | 18902 | 20818 |  +1916 |  +10.14 | 76.63 | 75.19 | -1.44 | 78.97 | 71.91 | -7.06 |
| XML_COMPACT | 11950 | 11181 | -769 | -6.44 | 7149 | 11406 |  +4257 |  +59.55 | 19099 | 22587 |  +3488 |  +18.26 | 72.03 | 78.42 |  +6.39 | 75.13 | 68.59 | -6.54 |
| XML_PRETTY | 16186 | 15117 | -1069 | -6.60 | 10729 | 9336 | -1393 | -12.98 | 26915 | 24453 | -2462 | -9.15 | 75.61 | 76.51 |  +0.90 | 52.96 | 61.36 |  +8.40 |
| YAML | 12574 | 11791 | -783 | -6.23 | 9523 | 13188 |  +3665 |  +38.49 | 22097 | 24979 |  +2882 |  +13.04 | 74.13 | 79.84 |  +5.71 | 67.13 | 62.03 | -5.10 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 28 | 250.786 | 0.90 | 80925 | 0.129 | 652.62 | 80953 | 250.915 | 522.28 |
| CSV | opt | 10 | 672.800 | 0.32 | 86539 | 0.128 | 697.90 | 86549 | 672.928 | 558.38 |
| JSON_COMPACT | man | 36 | 258.222 | 1.16 | 106585 | 0.128 | 859.56 | 106621 | 258.350 | 687.88 |
| JSON_COMPACT | opt | 9 | 974.222 | 0.29 | 75636 | 0.128 | 609.97 | 75645 | 974.350 | 488.03 |
| JSON_PRETTY | man | 13 | 1100.846 | 0.42 | 96223 | 0.126 | 775.99 | 96236 | 1100.972 | 620.88 |
| JSON_PRETTY | opt | 15 | 893.000 | 0.48 | 85002 | 0.124 | 685.50 | 85017 | 893.124 | 548.50 |
| TOON_DEFAULT | man | 27 | 266.000 | 0.87 | 95028 | 0.123 | 766.35 | 95055 | 266.123 | 613.26 |
| TOON_DEFAULT | opt | 27 | 433.667 | 0.87 | 74050 | 0.122 | 597.17 | 74077 | 433.789 | 477.91 |
| XML_COMPACT | man | 16 | 746.875 | 0.52 | 80267 | 0.089 | 647.31 | 80283 | 746.964 | 517.95 |
| XML_COMPACT | opt | 15 | 745.400 | 0.48 | 87671 | 0.130 | 707.02 | 87686 | 745.530 | 565.72 |
| XML_PRETTY | man | 18 | 899.222 | 0.58 | 86445 | 0.124 | 697.13 | 86463 | 899.346 | 557.82 |
| XML_PRETTY | opt | 9 | 1679.667 | 0.29 | 74275 | 0.126 | 598.99 | 74284 | 1679.793 | 479.25 |
| YAML | man | 11 | 1143.091 | 0.35 | 75884 | 0.125 | 611.97 | 75895 | 1143.216 | 489.65 |
| YAML | opt | 8 | 1473.875 | 0.26 | 103981 | 0.127 | 838.56 | 103989 | 1474.002 | 670.90 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) | Total Man (s) | Total Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 28 | 10 | -18 | -64.29 | 80.93 | 86.54 |  +5.61 |  +6.94 | 80.95 | 86.55 |  +5.60 |  +6.91 |
| JSON_COMPACT | 36 | 9 | -27 | -75.00 | 106.58 | 75.64 | -30.95 | -29.04 | 106.62 | 75.65 | -30.98 | -29.05 |
| JSON_PRETTY | 13 | 15 |  +2 |  +15.38 | 96.22 | 85.00 | -11.22 | -11.66 | 96.24 | 85.02 | -11.22 | -11.66 |
| TOON_DEFAULT | 27 | 27 | 0 | 0.00 | 95.03 | 74.05 | -20.98 | -22.08 | 95.05 | 86.71 | -8.34 | -8.78 |
| XML_COMPACT | 16 | 15 | -1 | -6.25 | 80.27 | 87.67 |  +7.40 |  +9.22 | 80.28 | 87.69 |  +7.40 |  +9.22 |
| XML_PRETTY | 18 | 9 | -9 | -50.00 | 86.44 | 74.28 | -12.17 | -14.08 | 86.46 | 74.28 | -12.18 | -14.09 |
| YAML | 11 | 8 | -3 | -27.27 | 75.88 | 103.98 |  +28.10 |  +37.03 | 75.89 | 103.99 |  +28.09 |  +37.02 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Char/Token | Token/Value | Token/Object | Info/Token |
|---|---|---|---|---|---|
| CSV | man | 1.438 | 10.296 | 226.516 | 0.388 |
| CSV | opt | 1.423 | 10.662 | 217.032 | 0.367 |
| JSON_COMPACT | man | 2.143 | 13.630 | 299.871 | 0.340 |
| JSON_COMPACT | opt | 2.108 | 13.895 | 282.839 | 0.415 |
| JSON_PRETTY | man | 1.691 | 20.984 | 461.645 | 0.312 |
| JSON_PRETTY | opt | 1.677 | 21.228 | 432.097 | 0.328 |
| TOON_DEFAULT | man | 1.415 | 10.531 | 231.677 | 0.407 |
| TOON_DEFAULT | opt | 1.677 | 18.556 | 377.710 | 0.367 |
| XML_COMPACT | man | 2.315 | 17.522 | 385.484 | 0.384 |
| XML_COMPACT | opt | 2.288 | 17.719 | 360.677 | 0.348 |
| XML_PRETTY | man | 1.931 | 23.733 | 522.129 | 0.285 |
| XML_PRETTY | opt | 1.913 | 23.957 | 487.645 | 0.314 |
| YAML | man | 1.661 | 18.437 | 405.613 | 0.343 |
| YAML | opt | 1.646 | 18.686 | 380.355 | 0.322 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.438 | 1.423 | -0.015 | -1.04 | 10.296 | 10.662 |  +0.366 |  +3.55 | 226.516 | 217.032 | -9.484 | -4.19 | 0.388 | 0.367 | -0.021 | -5.41 |
| JSON_COMPACT | 2.143 | 2.108 | -0.035 | -1.63 | 13.630 | 13.895 |  +0.265 |  +1.94 | 299.871 | 282.839 | -17.032 | -5.68 | 0.340 | 0.415 |  +0.075 |  +22.06 |
| JSON_PRETTY | 1.691 | 1.677 | -0.014 | -0.83 | 20.984 | 21.228 |  +0.244 |  +1.16 | 461.645 | 432.097 | -29.548 | -6.40 | 0.312 | 0.328 |  +0.016 |  +5.13 |
| TOON_DEFAULT | 1.415 | 1.677 |  +0.262 |  +18.52 | 10.531 | 18.556 |  +8.025 |  +76.20 | 231.677 | 377.710 |  +146.033 |  +63.03 | 0.407 | 0.367 | -0.041 | -10.06 |
| XML_COMPACT | 2.315 | 2.288 | -0.027 | -1.17 | 17.522 | 17.719 |  +0.197 |  +1.12 | 385.484 | 360.677 | -24.807 | -6.44 | 0.384 | 0.348 | -0.036 | -9.38 |
| XML_PRETTY | 1.931 | 1.913 | -0.018 | -0.93 | 23.733 | 23.957 |  +0.224 |  +0.94 | 522.129 | 487.645 | -34.484 | -6.60 | 0.285 | 0.314 |  +0.029 |  +10.18 |
| YAML | 1.661 | 1.646 | -0.015 | -0.90 | 18.437 | 18.686 |  +0.249 |  +1.35 | 405.613 | 380.355 | -25.258 | -6.23 | 0.343 | 0.322 | -0.021 | -6.12 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| CSV | man | 17432 | 11781 | 5652 | 67.58 | 67.37 | 77.27 | 77.13 |
| CSV | opt | 17841 | 11683 | 6159 | 65.48 | 65.49 | 74.51 | 74.52 |
| JSON_COMPACT | man | 22932 | 17894 | 5038 | 78.03 | 77.71 | 67.23 | 67.00 |
| JSON_COMPACT | opt | 18468 | 14148 | 4320 | 76.61 | 75.34 | 80.33 | 79.44 |
| JSON_PRETTY | man | 26450 | 21829 | 4621 | 82.53 | 82.08 | 59.27 | 58.96 |
| JSON_PRETTY | opt | 23974 | 18870 | 5104 | 78.71 | 78.15 | 64.41 | 64.02 |
| TOON_DEFAULT | man | 18902 | 14532 | 4370 | 76.88 | 76.63 | 79.14 | 78.97 |
| TOON_DEFAULT | opt | 20818 | 15801 | 5017 | 75.90 | 75.19 | 72.41 | 71.91 |
| XML_COMPACT | man | 19099 | 14017 | 5082 | 73.39 | 72.03 | 76.08 | 75.13 |
| XML_COMPACT | opt | 22587 | 17731 | 4856 | 78.50 | 78.42 | 68.64 | 68.59 |
| XML_PRETTY | man | 26915 | 20619 | 6295 | 76.61 | 75.61 | 53.66 | 52.96 |
| XML_PRETTY | opt | 24452 | 18799 | 5653 | 76.88 | 76.51 | 61.62 | 61.36 |
| YAML | man | 22097 | 16752 | 5345 | 75.81 | 74.13 | 68.31 | 67.13 |
| YAML | opt | 24979 | 20078 | 4901 | 80.38 | 79.84 | 62.41 | 62.03 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 17432 | 17841 |  +409 |  +2.35 | 11781 | 11683 | -98 | -0.83 | 5652 | 6159 |  +507 |  +8.97 | 67.58 | 65.48 | -2.10 | 77.27 | 74.513 | -2.76 | -3.57 |
| JSON_COMPACT | 22932 | 18468 | -4464 | -19.47 | 17894 | 14148 | -3746 | -20.93 | 5038 | 4320 | -718 | -14.26 | 78.03 | 76.61 | -1.42 | 67.23 | 80.326 |  +13.10 |  +19.49 |
| JSON_PRETTY | 26450 | 23974 | -2476 | -9.36 | 21829 | 18870 | -2959 | -13.56 | 4621 | 5104 |  +483 |  +10.46 | 82.53 | 78.71 | -3.82 | 59.27 | 64.413 |  +5.14 |  +8.68 |
| TOON_DEFAULT | 18902 | 20818 |  +1916 |  +10.14 | 14532 | 15801 |  +1269 |  +8.73 | 4370 | 5017 |  +647 |  +14.81 | 76.88 | 75.90 | -0.98 | 79.14 | 72.409 | -6.74 | -8.51 |
| XML_COMPACT | 19099 | 22587 |  +3488 |  +18.26 | 14017 | 17731 |  +3714 |  +26.50 | 5082 | 4856 | -226 | -4.45 | 73.39 | 78.50 |  +5.11 | 76.08 | 68.645 | -7.44 | -9.77 |
| XML_PRETTY | 26915 | 24453 | -2462 | -9.15 | 20619 | 18799 | -1820 | -8.83 | 6295 | 5653 | -642 | -10.20 | 76.61 | 76.88 |  +0.27 | 53.66 | 61.622 |  +7.96 |  +14.84 |
| YAML | 22097 | 24979 |  +2882 |  +13.04 | 16752 | 20078 |  +3326 |  +19.86 | 5345 | 4901 | -444 | -8.32 | 75.81 | 80.38 |  +4.57 | 68.31 | 62.41 | -5.90 | -8.64 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| CSV | man | 84 | 40 | 0 | 67.58 |
| CSV | opt | 81 | 43 | 0 | 65.48 |
| JSON_COMPACT | man | 97 | 27 | 0 | 78.03 |
| JSON_COMPACT | opt | 95 | 29 | 0 | 76.61 |
| JSON_PRETTY | man | 102 | 22 | 0 | 82.53 |
| JSON_PRETTY | opt | 98 | 26 | 0 | 78.71 |
| TOON_DEFAULT | man | 95 | 29 | 0 | 76.88 |
| TOON_DEFAULT | opt | 94 | 30 | 0 | 75.90 |
| XML_COMPACT | man | 91 | 33 | 0 | 73.39 |
| XML_COMPACT | opt | 97 | 27 | 0 | 78.50 |
| XML_PRETTY | man | 95 | 29 | 0 | 76.61 |
| XML_PRETTY | opt | 95 | 29 | 0 | 76.88 |
| YAML | man | 94 | 30 | 0 | 75.81 |
| YAML | opt | 100 | 24 | 0 | 80.38 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 84 | 81 | -3 | -3.57 | 40 | 43 |  +3 |  +7.50 | 0 | 0 | 0 | 0.00 | 67.58 | 65.48 | -2.10 |
| JSON_COMPACT | 97 | 95 | -2 | -2.06 | 27 | 29 |  +2 |  +7.41 | 0 | 0 | 0 | 0.00 | 78.03 | 76.61 | -1.42 |
| JSON_PRETTY | 102 | 98 | -4 | -3.92 | 22 | 26 |  +4 |  +18.18 | 0 | 0 | 0 | 0.00 | 82.53 | 78.71 | -3.82 |
| TOON_DEFAULT | 95 | 94 | -1 | -1.05 | 29 | 30 |  +1 |  +3.45 | 0 | 0 | 0 | 0.00 | 76.88 | 75.90 | -0.98 |
| XML_COMPACT | 91 | 97 |  +6 |  +6.59 | 33 | 27 | -6 | -18.18 | 0 | 0 | 0 | 0.00 | 73.39 | 78.50 |  +5.11 |
| XML_PRETTY | 95 | 95 | 0 | 0.00 | 29 | 29 | 0 | 0.00 | 0 | 0 | 0 | 0.00 | 76.61 | 76.88 |  +0.27 |
| YAML | 94 | 100 |  +6 |  +6.38 | 30 | 24 | -6 | -20.00 | 0 | 0 | 0 | 0.00 | 75.81 | 80.38 |  +4.57 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| CSV | man | 67.58 | 75.64 | 66.67 | 61.90 | 53.33 |
| CSV | opt | 65.48 | 79.27 | 62.22 | 62.86 | 36.19 |
| JSON_COMPACT | man | 78.03 | 98.18 | 77.78 | 63.09 | 40.48 |
| JSON_COMPACT | opt | 76.61 | 94.91 | 69.63 | 61.90 | 52.38 |
| JSON_PRETTY | man | 82.53 | 96.97 | 83.95 | 66.67 | 58.73 |
| JSON_PRETTY | opt | 78.71 | 96.73 | 75.56 | 66.67 | 47.62 |
| TOON_DEFAULT | man | 76.88 | 92.32 | 79.84 | 60.32 | 49.21 |
| TOON_DEFAULT | opt | 75.90 | 93.53 | 73.66 | 60.84 | 47.62 |
| XML_COMPACT | man | 73.39 | 88.49 | 67.90 | 57.14 | 57.14 |
| XML_COMPACT | opt | 78.50 | 97.58 | 81.48 | 61.90 | 41.27 |
| XML_PRETTY | man | 76.61 | 95.76 | 70.37 | 63.49 | 47.62 |
| XML_PRETTY | opt | 76.88 | 91.51 | 80.25 | 58.73 | 52.38 |
| YAML | man | 75.81 | 96.97 | 64.20 | 61.90 | 49.21 |
| YAML | opt | 80.38 | 99.39 | 74.08 | 73.02 | 46.03 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 75.64 | 79.27 |  +3.64 |
| JSON_COMPACT | 98.18 | 94.91 | -3.27 |
| JSON_PRETTY | 96.97 | 96.73 | -0.24 |
| TOON_DEFAULT | 92.32 | 93.53 |  +1.21 |
| XML_COMPACT | 88.49 | 97.58 |  +9.09 |
| XML_PRETTY | 95.76 | 91.51 | -4.24 |
| YAML | 96.97 | 99.39 |  +2.42 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 66.67 | 62.22 | -4.44 |
| JSON_COMPACT | 77.78 | 69.63 | -8.15 |
| JSON_PRETTY | 83.95 | 75.56 | -8.39 |
| TOON_DEFAULT | 79.84 | 73.66 | -6.17 |
| XML_COMPACT | 67.90 | 81.48 |  +13.58 |
| XML_PRETTY | 70.37 | 80.25 |  +9.87 |
| YAML | 64.20 | 74.08 |  +9.88 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 61.90 | 62.86 |  +0.95 |
| JSON_COMPACT | 63.09 | 61.90 | -1.19 |
| JSON_PRETTY | 66.67 | 66.67 | -0.00 |
| TOON_DEFAULT | 60.32 | 60.84 |  +0.53 |
| XML_COMPACT | 57.14 | 61.90 |  +4.76 |
| XML_PRETTY | 63.49 | 58.73 | -4.77 |
| YAML | 61.90 | 73.02 |  +11.11 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 53.33 | 36.19 | -17.14 |
| JSON_COMPACT | 40.48 | 52.38 |  +11.90 |
| JSON_PRETTY | 58.73 | 47.62 | -11.11 |
| TOON_DEFAULT | 49.21 | 47.62 | -1.59 |
| XML_COMPACT | 57.14 | 41.27 | -15.87 |
| XML_PRETTY | 47.62 | 52.38 |  +4.76 |
| YAML | 49.21 | 46.03 | -3.17 |

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
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

- **Report Generated**: 2026-04-03
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