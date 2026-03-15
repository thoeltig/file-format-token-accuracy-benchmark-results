# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-15
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Extended Thinking**: on
- **Data Structure**: flat
- **Formats Tested**: 8 (CSV, JSON_COMPACT, JSON_PRETTY, TOON_SAFE, TOON_UNSAFE, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: First iteration

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
   - Optional: CSV 6931 tokens
   - Mandatory: TOON_UNSAFE 7279 tokens
- Lowest output token cost drift:
   - Optional: JSON_PRETTY ↓ -0.60 % ↑ 0.60 %
   - Mandatory: CSV ↓ -0.10 % ↑ 0.20 %
- Highest accuracy:
   - Optional: TOON_SAFE 68.82 %
   - Mandatory: JSON_PRETTY 71.51 %
- Lowest accuracy drift:
   - Optional: TOON_UNSAFE ↓ -2.03 % ↑ 2.85 %
   - Mandatory: JSON_PRETTY ↓ -0.76 % ↑ 1.50 %
- Most useful tokens:
   - Optional: XML_PRETTY 10173 / 15321 tokens
   - Mandatory: XML_PRETTY 11051 / 16509 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 70.84
   - Mandatory: TOON_SAFE 77.83
- Lowest delta (optional-mandatory):
   - Total tokens: CSV -387 tokens
   - Accuracy: XML_PRETTY -0.54 %
   - Token efficiency: CSV -0.48

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 15321 tokens
   - Mandatory: XML_PRETTY 16509 tokens
- Highest output token drift:
   - Optional: XML_PRETTY ↓ -94.40 % ↑ 49.79 %
   - Mandatory: JSON_COMPACT ↓ -96.05 % ↑ 48.90 %
- Lowest accuracy:
   - Optional: CSV 54.84 %
   - Mandatory: CSV 57.26 %
- Highest accuracy drift:
   - Optional: XML_COMPACT ↓ -7.79 % ↑ 6.97 %
   - Mandatory: CSV ↓ -14.09 % ↑ 12.68 %
- Most wasted tokens:
   - Optional: XML_PRETTY 5148 / 15321 tokens
   - Mandatory: XML_PRETTY 5458 / 16509 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 50.22
   - Mandatory: XML_PRETTY 46.89
- Highest delta (optional-mandatory):
   - Total tokens: TOON_UNSAFE 4624 tokens
   - Accuracy: JSON_PRETTY -8.07 %
   - Token efficiency: TOON_UNSAFE -16.34

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| xml_compact ≈ 75338 s | toon_unsafe ≈ 7279  | toon_safe ≈ 2186  | json_pretty ≈ 72 % | json_pretty ≈ 72 % | toon_safe ≈ 78  | toon_safe ≈ 78  |
| csv ( +3.9 %) | csv ( +0.5 %) | toon_unsafe ( +4.7 %) | toon_safe (-1.1 %) | toon_safe (-2.0 %) | toon_unsafe (-1.2 %) | toon_unsafe (-1.4 %) |
| toon_unsafe ( +7.4 %) | toon_safe ( +1.5 %) | csv ( +43.1 %) | toon_unsafe (-3.0 %) | xml_pretty (-4.0 %) | csv (-11.6 %) | csv (-11.0 %) |
| yaml ( +17.2 %) | json_compact ( +30.5 %) | json_compact ( +55.3 %) | xml_compact (-4.3 %) | toon_unsafe (-4.1 %) | json_compact (-14.0 %) | json_compact (-13.5 %) |
| xml_pretty ( +36.2 %) | xml_compact ( +65.4 %) | xml_compact ( +80.6 %) | xml_pretty (-4.6 %) | xml_compact (-4.3 %) | xml_compact (-21.6 %) | xml_compact (-20.7 %) |
| json_pretty ( +40.8 %) | yaml ( +75.7 %) | json_pretty ( +90.6 %) | yaml (-5.4 %) | yaml (-5.4 %) | yaml (-25.5 %) | yaml (-24.7 %) |
| json_compact ( +45.8 %) | json_pretty ( +100.9 %) | yaml ( +98.1 %) | json_compact (-7.3 %) | json_compact (-7.6 %) | json_pretty (-28.1 %) | json_pretty (-27.2 %) |
| toon_safe ( +47.4 %) | xml_pretty ( +126.8 %) | xml_pretty ( +149.7 %) | csv (-14.3 %) | csv (-14.5 %) | xml_pretty (-39.8 %) | xml_pretty (-38.4 %) |


##### Optional

| ↑ Total Duration) | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| json_pretty ≈ 73175 s | csv ≈ 6931  | json_compact ≈ 2907  | toon_safe ≈ 69 % | json_compact ≈ 71 % | json_compact ≈ 71  | json_compact ≈ 73  |
| toon_safe ( +6.3 %) | json_compact ( +31.1 %) | csv ( +7.7 %) | json_compact (-0.8 %) | toon_safe (-0.0 %) | csv (-3.5 %) | csv (-4.6 %) |
| xml_pretty ( +7.9 %) | xml_compact ( +62.5 %) | toon_safe ( +27.8 %) | xml_pretty (-2.4 %) | toon_unsafe (-2.1 %) | toon_safe (-11.7 %) | toon_safe (-12.2 %) |
| json_compact ( +13.2 %) | toon_unsafe ( +71.7 %) | xml_compact ( +33.4 %) | toon_unsafe (-3.0 %) | xml_compact (-3.5 %) | xml_compact (-12.0 %) | xml_compact (-12.8 %) |
| yaml ( +21.0 %) | toon_safe ( +71.8 %) | toon_unsafe ( +39.8 %) | xml_compact (-3.2 %) | yaml (-3.5 %) | toon_unsafe (-14.6 %) | toon_unsafe (-14.2 %) |
| xml_compact ( +30.1 %) | yaml ( +74.6 %) | yaml ( +46.7 %) | yaml (-4.0 %) | xml_pretty (-4.0 %) | yaml (-16.5 %) | yaml (-16.4 %) |
| csv ( +50.1 %) | json_pretty ( +97.7 %) | json_pretty ( +72.4 %) | json_pretty (-5.4 %) | json_pretty (-5.7 %) | json_pretty (-24.9 %) | json_pretty (-25.3 %) |
| toon_unsafe ( +51.5 %) | xml_pretty ( +121.1 %) | xml_pretty ( +77.1 %) | csv (-14.0 %) | csv (-14.4 %) | xml_pretty (-29.1 %) | xml_pretty (-30.7 %) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval % | ↓ Structure Awareness % | ↓ Filtering % | ↓ Aggregation % |
|---|---|---|---|
| json_compact ≈ 75 % | json_pretty ≈ 81 % | yaml ≈ 73 % | toon_unsafe ≈ 70 % |
| toon_safe (0.0 %) | xml_compact (-6.2 %) | xml_pretty (-4.8 %) | toon_safe (-6.4 %) |
| json_pretty (-0.6 %) | xml_pretty (-6.2 %) | toon_unsafe (-6.3 %) | json_pretty (-11.1 %) |
| yaml (-0.6 %) | toon_safe (-8.6 %) | json_pretty (-7.9 %) | xml_compact (-15.9 %) |
| xml_pretty (-1.8 %) | toon_unsafe (-13.6 %) | toon_safe (-9.5 %) | csv (-22.2 %) |
| xml_compact (-4.8 %) | json_compact (-13.6 %) | xml_compact (-9.5 %) | xml_pretty (-30.2 %) |
| toon_unsafe (-5.5 %) | yaml (-16.0 %) | json_compact (-14.3 %) | yaml (-30.2 %) |
| csv (-15.2 %) | csv (-17.3 %) | csv (-20.6 %) | json_compact (-31.7 %) |


##### Optional

| ↓ Field Retrieval % | ↓ Structure Awareness % | ↓ Filtering % | ↓ Aggregation % |
|---|---|---|---|
| xml_compact ≈ 67 % | json_compact ≈ 88 % | json_compact ≈ 71 % | xml_pretty ≈ 62 % |
| xml_pretty (0.0 %) | toon_safe (-4.9 %) | toon_unsafe (0.0 %) | toon_safe (-3.2 %) |
| yaml (0.0 %) | toon_unsafe (-4.9 %) | toon_safe (-0.0 %) | json_compact (-14.3 %) |
| toon_unsafe (-0.6 %) | yaml (-8.6 %) | yaml (-1.6 %) | xml_compact (-15.9 %) |
| json_pretty (-1.2 %) | xml_compact (-11.1 %) | xml_compact (-4.8 %) | json_pretty (-19.0 %) |
| json_compact (-2.4 %) | json_pretty (-12.3 %) | xml_pretty (-6.4 %) | toon_unsafe (-25.4 %) |
| toon_safe (-2.4 %) | xml_pretty (-18.5 %) | csv (-7.9 %) | csv (-27.0 %) |
| csv (-10.9 %) | csv (-27.2 %) | json_pretty (-9.5 %) | yaml (-27.0 %) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Tokens/Char | Info/Token | Token/Answer | Acc (%) | Wtd Acc (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6989 | 329 | 7318 | 1.444 | 0.782 | 2.656 | 57.26 | 69.26 | 4190.477 | 3127.856 | 68.84 | 69.26 |
| CSV | opt | 6700 | 231 | 6931 | 1.429 | 0.791 | 1.863 | 54.84 | 69.43 | 3800.960 | 3130.040 | 68.36 | 69.43 |
| JSON_COMPACT | man | 9268 | 228 | 9496 | 2.149 | 0.677 | 1.836 | 64.25 | 67.28 | 6100.966 | 3394.701 | 66.93 | 67.28 |
| JSON_COMPACT | opt | 8748 | 338 | 9086 | 2.113 | 0.749 | 2.723 | 68.01 | 72.74 | 6179.162 | 2906.505 | 70.84 | 72.74 |
| JSON_PRETTY | man | 14283 | 341 | 14624 | 1.694 | 0.489 | 2.750 | 71.51 | 56.59 | 10457.622 | 4166.378 | 55.98 | 56.59 |
| JSON_PRETTY | opt | 13367 | 336 | 13703 | 1.680 | 0.463 | 2.710 | 63.44 | 54.30 | 8693.183 | 5009.817 | 53.21 | 54.30 |
| TOON_SAFE | man | 7048 | 343 | 7391 | 1.442 | 0.953 | 2.769 | 70.43 | 77.78 | 5205.716 | 2185.617 | 77.83 | 77.78 |
| TOON_SAFE | opt | 11570 | 339 | 11909 | 1.697 | 0.578 | 2.731 | 68.82 | 63.88 | 8195.545 | 3713.122 | 62.58 | 63.88 |
| TOON_UNSAFE | man | 7048 | 231 | 7279 | 1.442 | 0.942 | 1.860 | 68.55 | 76.71 | 4989.526 | 2289.141 | 76.87 | 76.71 |
| TOON_UNSAFE | opt | 11561 | 342 | 11903 | 1.698 | 0.553 | 2.755 | 65.86 | 62.44 | 7839.096 | 4063.571 | 60.53 | 62.44 |
| XML_COMPACT | man | 11693 | 343 | 12036 | 2.366 | 0.558 | 2.763 | 67.20 | 61.66 | 8087.968 | 3947.699 | 61.05 | 61.66 |
| XML_COMPACT | opt | 10930 | 335 | 11265 | 2.340 | 0.582 | 2.704 | 65.59 | 63.46 | 7388.932 | 3876.401 | 62.33 | 63.46 |
| XML_PRETTY | man | 16166 | 343 | 16509 | 1.934 | 0.405 | 2.763 | 66.94 | 47.92 | 11050.902 | 5457.765 | 46.89 | 47.92 |
| XML_PRETTY | opt | 15089 | 232 | 15321 | 1.917 | 0.433 | 1.874 | 66.40 | 50.43 | 10173.365 | 5147.968 | 50.22 | 50.43 |
| YAML | man | 12554 | 231 | 12785 | 1.664 | 0.517 | 1.863 | 66.13 | 58.56 | 8454.720 | 4330.279 | 57.96 | 58.56 |
| YAML | opt | 11771 | 333 | 12104 | 1.649 | 0.535 | 2.688 | 64.78 | 60.83 | 7841.187 | 4263.146 | 59.14 | 60.83 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Acc Man (%) | Acc Opt (%) | Diff (%) | Wtd Acc Man (%) | Wtd Acc Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7318 | 6931 | -387 | -5.29 | 57.26 | 54.84 | -2.42 | 57.86 | 56.37 | -1.49 | 68.84 | 68.36 | -0.48 | 69.26 | 69.43 |  +0.17 |
| JSON_COMPACT | 9496 | 9086 | -410 | -4.32 | 64.25 | 68.01 |  +3.76 | 64.75 | 70.72 |  +5.97 | 66.93 | 70.84 |  +3.91 | 67.28 | 72.74 |  +5.46 |
| JSON_PRETTY | 14624 | 13703 | -921 | -6.30 | 71.51 | 63.44 | -8.07 | 72.39 | 65.00 | -7.39 | 55.98 | 53.21 | -2.77 | 56.59 | 54.30 | -2.29 |
| TOON_SAFE | 7391 | 11908 |  +4517 |  +61.11 | 70.43 | 68.82 | -1.61 | 70.36 | 70.67 |  +0.31 | 77.83 | 62.58 | -15.25 | 77.78 | 63.88 | -13.90 |
| TOON_UNSAFE | 7279 | 11903 |  +4624 |  +63.53 | 68.55 | 65.86 | -2.69 | 68.32 | 68.58 |  +0.26 | 76.87 | 60.53 | -16.34 | 76.71 | 62.44 | -14.27 |
| XML_COMPACT | 12036 | 11266 | -770 | -6.40 | 67.20 | 65.59 | -1.61 | 68.07 | 67.20 | -0.87 | 61.05 | 62.33 |  +1.28 | 61.66 | 63.46 |  +1.80 |
| XML_PRETTY | 16509 | 15322 | -1187 | -7.19 | 66.94 | 66.40 | -0.54 | 68.42 | 66.69 | -1.73 | 46.89 | 50.22 |  +3.33 | 47.92 | 50.43 |  +2.50 |
| YAML | 12785 | 12104 | -681 | -5.33 | 66.13 | 64.78 | -1.35 | 66.98 | 67.19 |  +0.21 | 57.96 | 59.14 |  +1.18 | 58.56 | 60.83 |  +2.27 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 20 | 349.450 | 0.65 | 78262 | 0.004 | 631.14 | 78282 | 349.454 | 505.04 |
| CSV | opt | 15 | 446.667 | 0.48 | 109800 | 0.002 | 885.48 | 109815 | 446.669 | 708.48 |
| JSON_COMPACT | man | 28 | 331.000 | 0.90 | 109802 | 0.002 | 885.50 | 109830 | 331.002 | 708.58 |
| JSON_COMPACT | opt | 10 | 874.800 | 0.32 | 82813 | 0.004 | 667.85 | 82823 | 874.804 | 534.34 |
| JSON_PRETTY | man | 25 | 571.320 | 0.81 | 106026 | 0.003 | 855.05 | 106051 | 571.323 | 684.20 |
| JSON_PRETTY | opt | 13 | 1028.231 | 0.42 | 73162 | 0.005 | 590.01 | 73175 | 1028.236 | 472.09 |
| TOON_SAFE | man | 4 | 1762.000 | 0.13 | 111019 | 0.003 | 895.31 | 111023 | 1762.003 | 716.28 |
| TOON_SAFE | opt | 8 | 1446.250 | 0.26 | 77811 | 0.004 | 627.51 | 77819 | 1446.254 | 502.06 |
| TOON_UNSAFE | man | 3 | 2349.333 | 0.10 | 80914 | 0.003 | 652.53 | 80917 | 2349.336 | 522.04 |
| TOON_UNSAFE | opt | 5 | 2312.200 | 0.16 | 110859 | 0.003 | 894.03 | 110864 | 2312.203 | 715.25 |
| XML_COMPACT | man | 4 | 2923.250 | 0.13 | 75334 | 0.005 | 607.53 | 75338 | 2923.255 | 486.05 |
| XML_COMPACT | opt | 9 | 1214.444 | 0.29 | 95202 | 0.004 | 767.76 | 95211 | 1214.448 | 614.26 |
| XML_PRETTY | man | 11 | 1469.636 | 0.35 | 102619 | 0.003 | 827.57 | 102630 | 1469.639 | 662.13 |
| XML_PRETTY | opt | 17 | 887.588 | 0.55 | 78965 | 0.003 | 636.81 | 78982 | 887.591 | 509.56 |
| YAML | man | 8 | 1569.250 | 0.26 | 88278 | 0.003 | 711.92 | 88286 | 1569.253 | 569.59 |
| YAML | opt | 11 | 1070.091 | 0.35 | 88549 | 0.004 | 714.10 | 88560 | 1070.095 | 571.35 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) | Total Man (s) | Total Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 20 | 15 | -5 | -25.00 | 78.26 | 109.80 |  +31.54 |  +40.30 | 78.28 | 109.81 |  +31.53 |  +40.28 |
| JSON_COMPACT | 28 | 10 | -18 | -64.29 | 109.80 | 82.81 | -26.99 | -24.58 | 109.83 | 82.82 | -27.01 | -24.59 |
| JSON_PRETTY | 25 | 13 | -12 | -48.00 | 106.03 | 73.16 | -32.86 | -31.00 | 106.05 | 73.17 | -32.88 | -31.00 |
| TOON_SAFE | 4 | 8 |  +4 |  +100.00 | 111.02 | 77.81 | -33.21 | -29.91 | 111.02 | 77.82 | -33.20 | -29.91 |
| TOON_UNSAFE | 3 | 5 |  +2 |  +66.67 | 80.91 | 110.86 |  +29.95 |  +37.01 | 80.92 | 110.86 |  +29.95 |  +37.01 |
| XML_COMPACT | 4 | 9 |  +5 |  +125.00 | 75.33 | 95.20 |  +19.87 |  +26.37 | 75.34 | 95.21 |  +19.87 |  +26.38 |
| XML_PRETTY | 11 | 17 |  +6 |  +54.55 | 102.62 | 78.97 | -23.65 | -23.05 | 102.63 | 78.98 | -23.65 | -23.04 |
| YAML | 8 | 11 |  +3 |  +37.50 | 88.28 | 88.55 |  +0.27 |  +0.31 | 88.29 | 88.56 |  +0.27 |  +0.31 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Char/Token | Token/Value | Token/Object | Info/Token |
|---|---|---|---|---|---|
| CSV | man | 1.444 | 10.248 | 225.452 | 0.782 |
| CSV | opt | 1.429 | 10.618 | 216.129 | 0.791 |
| JSON_COMPACT | man | 2.149 | 13.589 | 298.968 | 0.677 |
| JSON_COMPACT | opt | 2.113 | 13.864 | 282.194 | 0.749 |
| JSON_PRETTY | man | 1.694 | 20.943 | 460.742 | 0.489 |
| JSON_PRETTY | opt | 1.680 | 21.184 | 431.194 | 0.463 |
| TOON_SAFE | man | 1.442 | 10.334 | 227.355 | 0.953 |
| TOON_SAFE | opt | 1.697 | 18.336 | 373.226 | 0.578 |
| TOON_UNSAFE | man | 1.442 | 10.334 | 227.355 | 0.942 |
| TOON_UNSAFE | opt | 1.698 | 18.322 | 372.935 | 0.553 |
| XML_COMPACT | man | 2.366 | 17.145 | 377.194 | 0.558 |
| XML_COMPACT | opt | 2.340 | 17.322 | 352.581 | 0.582 |
| XML_PRETTY | man | 1.934 | 23.704 | 521.484 | 0.405 |
| XML_PRETTY | opt | 1.917 | 23.913 | 486.742 | 0.433 |
| YAML | man | 1.664 | 18.408 | 404.968 | 0.517 |
| YAML | opt | 1.649 | 18.655 | 379.710 | 0.535 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.444 | 1.429 | -0.015 | -1.04 | 10.248 | 10.618 |  +0.370 |  +3.61 | 225.452 | 216.129 | -9.323 | -4.14 | 0.782 | 0.791 |  +0.009 |  +1.15 |
| JSON_COMPACT | 2.149 | 2.113 | -0.036 | -1.68 | 13.589 | 13.864 |  +0.275 |  +2.02 | 298.968 | 282.194 | -16.774 | -5.61 | 0.677 | 0.749 |  +0.072 |  +10.64 |
| JSON_PRETTY | 1.694 | 1.680 | -0.014 | -0.83 | 20.943 | 21.184 |  +0.241 |  +1.15 | 460.742 | 431.194 | -29.548 | -6.41 | 0.489 | 0.463 | -0.026 | -5.32 |
| TOON_SAFE | 1.442 | 1.697 |  +0.255 |  +17.68 | 10.334 | 18.336 |  +8.002 |  +77.43 | 227.355 | 373.226 |  +145.871 |  +64.16 | 0.953 | 0.578 | -0.375 | -39.35 |
| TOON_UNSAFE | 1.442 | 1.698 |  +0.256 |  +17.75 | 10.334 | 18.322 |  +7.988 |  +77.30 | 227.355 | 372.935 |  +145.580 |  +64.03 | 0.942 | 0.553 | -0.389 | -41.30 |
| XML_COMPACT | 2.366 | 2.340 | -0.026 | -1.10 | 17.145 | 17.322 |  +0.177 |  +1.03 | 377.194 | 352.581 | -24.613 | -6.53 | 0.558 | 0.582 |  +0.024 |  +4.30 |
| XML_PRETTY | 1.934 | 1.917 | -0.017 | -0.88 | 23.704 | 23.913 |  +0.209 |  +0.88 | 521.484 | 486.742 | -34.742 | -6.66 | 0.405 | 0.433 |  +0.028 |  +6.91 |
| YAML | 1.664 | 1.649 | -0.015 | -0.90 | 18.408 | 18.655 |  +0.247 |  +1.34 | 404.968 | 379.710 | -25.258 | -6.24 | 0.517 | 0.535 |  +0.018 |  +3.48 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Acc (%) | Wtd Acc (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| CSV | man | 7318 | 4190 | 3128 | 57.26 | 57.86 | 68.84 | 69.26 |
| CSV | opt | 6931 | 3801 | 3130 | 54.84 | 56.37 | 68.36 | 69.43 |
| JSON_COMPACT | man | 9496 | 6101 | 3395 | 64.25 | 64.75 | 66.93 | 67.28 |
| JSON_COMPACT | opt | 9086 | 6179 | 2907 | 68.01 | 70.72 | 70.84 | 72.74 |
| JSON_PRETTY | man | 14624 | 10458 | 4166 | 71.51 | 72.39 | 55.98 | 56.59 |
| JSON_PRETTY | opt | 13703 | 8693 | 5010 | 63.44 | 65.00 | 53.21 | 54.30 |
| TOON_SAFE | man | 7391 | 5206 | 2186 | 70.43 | 70.36 | 77.83 | 77.78 |
| TOON_SAFE | opt | 11909 | 8196 | 3713 | 68.82 | 70.67 | 62.58 | 63.88 |
| TOON_UNSAFE | man | 7279 | 4990 | 2289 | 68.55 | 68.32 | 76.87 | 76.71 |
| TOON_UNSAFE | opt | 11903 | 7839 | 4064 | 65.86 | 68.58 | 60.53 | 62.44 |
| XML_COMPACT | man | 12036 | 8088 | 3948 | 67.20 | 68.07 | 61.05 | 61.66 |
| XML_COMPACT | opt | 11265 | 7389 | 3876 | 65.59 | 67.20 | 62.33 | 63.46 |
| XML_PRETTY | man | 16509 | 11051 | 5458 | 66.94 | 68.42 | 46.89 | 47.92 |
| XML_PRETTY | opt | 15321 | 10173 | 5148 | 66.40 | 66.69 | 50.22 | 50.43 |
| YAML | man | 12785 | 8455 | 4330 | 66.13 | 66.98 | 57.96 | 58.56 |
| YAML | opt | 12104 | 7841 | 4263 | 64.78 | 67.19 | 59.14 | 60.83 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Acc (%) Man | Acc (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7318 | 6931 | -387 | -5.29 | 4190 | 3800 | -390 | -9.30 | 3128 | 3130 |  +2 |  +0.07 | 57.26 | 54.84 | -2.42 | 68.84 | 68.357 | -0.48 | -0.70 |
| JSON_COMPACT | 9496 | 9086 | -410 | -4.32 | 6101 | 6179 |  +78 |  +1.28 | 3395 | 2907 | -488 | -14.38 | 64.25 | 68.01 |  +3.76 | 66.93 | 70.841 |  +3.91 |  +5.85 |
| JSON_PRETTY | 14624 | 13703 | -921 | -6.30 | 10458 | 8694 | -1764 | -16.87 | 4166 | 5009 |  +843 |  +20.25 | 71.51 | 63.44 | -8.07 | 55.98 | 53.209 | -2.77 | -4.95 |
| TOON_SAFE | 7391 | 11908 |  +4517 |  +61.12 | 5206 | 8196 |  +2990 |  +57.43 | 2186 | 3714 |  +1528 |  +69.88 | 70.43 | 68.82 | -1.61 | 77.83 | 62.584 | -15.25 | -19.59 |
| TOON_UNSAFE | 7279 | 11903 |  +4624 |  +63.53 | 4990 | 7840 |  +2850 |  +57.11 | 2289 | 4063 |  +1774 |  +77.52 | 68.55 | 65.86 | -2.69 | 76.87 | 60.531 | -16.34 | -21.25 |
| XML_COMPACT | 12036 | 11266 | -770 | -6.40 | 8088 | 7389 | -699 | -8.64 | 3948 | 3877 | -71 | -1.81 | 67.20 | 65.59 | -1.61 | 61.05 | 62.334 |  +1.28 |  +2.10 |
| XML_PRETTY | 16509 | 15322 | -1187 | -7.19 | 11051 | 10173 | -878 | -7.94 | 5458 | 5148 | -310 | -5.68 | 66.94 | 66.40 | -0.54 | 46.89 | 50.223 |  +3.33 |  +7.11 |
| YAML | 12785 | 12104 | -681 | -5.32 | 8455 | 7841 | -614 | -7.26 | 4330 | 4263 | -67 | -1.55 | 66.13 | 64.78 | -1.35 | 57.96 | 59.144 |  +1.18 |  +2.04 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Acc (%) |
|---|---|---|---|---|---|
| CSV | man | 71 | 53 | 0 | 57.26 |
| CSV | opt | 68 | 56 | 0 | 54.84 |
| JSON_COMPACT | man | 80 | 44 | 0 | 64.25 |
| JSON_COMPACT | opt | 84 | 40 | 0 | 68.01 |
| JSON_PRETTY | man | 89 | 35 | 0 | 71.51 |
| JSON_PRETTY | opt | 79 | 45 | 0 | 63.44 |
| TOON_SAFE | man | 87 | 37 | 0 | 70.43 |
| TOON_SAFE | opt | 85 | 39 | 0 | 68.82 |
| TOON_UNSAFE | man | 85 | 39 | 0 | 68.55 |
| TOON_UNSAFE | opt | 82 | 42 | 0 | 65.86 |
| XML_COMPACT | man | 83 | 41 | 0 | 67.20 |
| XML_COMPACT | opt | 81 | 43 | 0 | 65.59 |
| XML_PRETTY | man | 83 | 41 | 0 | 66.94 |
| XML_PRETTY | opt | 82 | 42 | 0 | 66.40 |
| YAML | man | 82 | 42 | 0 | 66.13 |
| YAML | opt | 80 | 44 | 0 | 64.78 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Acc (%) Man | Acc (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 71 | 68 | -3 | -4.23 | 53 | 56 |  +3 |  +5.66 | 0 | 0 | 0 | 0.00 | 57.26 | 54.84 | -2.42 |
| JSON_COMPACT | 80 | 84 |  +4 |  +5.00 | 44 | 40 | -4 | -9.09 | 0 | 0 | 0 | 0.00 | 64.25 | 68.01 |  +3.76 |
| JSON_PRETTY | 89 | 79 | -10 | -11.24 | 35 | 45 |  +10 |  +28.57 | 0 | 0 | 0 | 0.00 | 71.51 | 63.44 | -8.07 |
| TOON_SAFE | 87 | 85 | -2 | -2.30 | 37 | 39 |  +2 |  +5.41 | 0 | 0 | 0 | 0.00 | 70.43 | 68.82 | -1.61 |
| TOON_UNSAFE | 85 | 82 | -3 | -3.53 | 39 | 42 |  +3 |  +7.69 | 0 | 0 | 0 | 0.00 | 68.55 | 65.86 | -2.69 |
| XML_COMPACT | 83 | 81 | -2 | -2.41 | 41 | 43 |  +2 |  +4.88 | 0 | 0 | 0 | 0.00 | 67.20 | 65.59 | -1.61 |
| XML_PRETTY | 83 | 82 | -1 | -1.20 | 41 | 42 |  +1 |  +2.44 | 0 | 0 | 0 | 0.00 | 66.94 | 66.40 | -0.54 |
| YAML | 82 | 80 | -2 | -2.44 | 42 | 44 |  +2 |  +4.76 | 0 | 0 | 0 | 0.00 | 66.13 | 64.78 | -1.35 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Acc (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| CSV | man | 57.26 | 59.40 | 64.20 | 52.38 | 47.62 |
| CSV | opt | 54.84 | 56.36 | 60.49 | 63.49 | 34.92 |
| JSON_COMPACT | man | 64.25 | 74.55 | 67.90 | 58.73 | 38.10 |
| JSON_COMPACT | opt | 68.01 | 64.85 | 87.66 | 71.43 | 47.62 |
| JSON_PRETTY | man | 71.51 | 73.94 | 81.48 | 65.08 | 58.73 |
| JSON_PRETTY | opt | 63.44 | 66.06 | 75.31 | 61.90 | 42.86 |
| TOON_SAFE | man | 70.43 | 74.55 | 72.84 | 63.49 | 63.49 |
| TOON_SAFE | opt | 68.82 | 64.85 | 82.72 | 71.43 | 58.73 |
| TOON_UNSAFE | man | 68.55 | 69.09 | 67.90 | 66.67 | 69.84 |
| TOON_UNSAFE | opt | 65.86 | 66.66 | 82.72 | 71.43 | 36.51 |
| XML_COMPACT | man | 67.20 | 69.70 | 75.31 | 63.49 | 53.97 |
| XML_COMPACT | opt | 65.59 | 67.27 | 76.55 | 66.67 | 46.03 |
| XML_PRETTY | man | 66.94 | 72.73 | 75.31 | 68.26 | 39.68 |
| XML_PRETTY | opt | 66.40 | 67.27 | 69.14 | 65.08 | 61.90 |
| YAML | man | 66.13 | 73.94 | 65.43 | 73.02 | 39.68 |
| YAML | opt | 64.78 | 67.27 | 79.01 | 69.84 | 34.92 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 59.40 | 56.36 | -3.03 |
| JSON_COMPACT | 74.55 | 64.85 | -9.70 |
| JSON_PRETTY | 73.94 | 66.06 | -7.89 |
| TOON_SAFE | 74.55 | 64.85 | -9.70 |
| TOON_UNSAFE | 69.09 | 66.66 | -2.43 |
| XML_COMPACT | 69.70 | 67.27 | -2.43 |
| XML_PRETTY | 72.73 | 67.27 | -5.46 |
| YAML | 73.94 | 67.27 | -6.67 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 64.20 | 60.49 | -3.70 |
| JSON_COMPACT | 67.90 | 87.66 |  +19.76 |
| JSON_PRETTY | 81.48 | 75.31 | -6.17 |
| TOON_SAFE | 72.84 | 82.72 |  +9.88 |
| TOON_UNSAFE | 67.90 | 82.72 |  +14.81 |
| XML_COMPACT | 75.31 | 76.55 |  +1.24 |
| XML_PRETTY | 75.31 | 69.14 | -6.17 |
| YAML | 65.43 | 79.01 |  +13.58 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 52.38 | 63.49 |  +11.11 |
| JSON_COMPACT | 58.73 | 71.43 |  +12.70 |
| JSON_PRETTY | 65.08 | 61.90 | -3.17 |
| TOON_SAFE | 63.49 | 71.43 |  +7.94 |
| TOON_UNSAFE | 66.67 | 71.43 |  +4.76 |
| XML_COMPACT | 63.49 | 66.67 |  +3.18 |
| XML_PRETTY | 68.26 | 65.08 | -3.18 |
| YAML | 73.02 | 69.84 | -3.18 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 47.62 | 34.92 | -12.70 |
| JSON_COMPACT | 38.10 | 47.62 |  +9.52 |
| JSON_PRETTY | 58.73 | 42.86 | -15.87 |
| TOON_SAFE | 63.49 | 58.73 | -4.76 |
| TOON_UNSAFE | 69.84 | 36.51 | -33.34 |
| XML_COMPACT | 53.97 | 46.03 | -7.93 |
| XML_PRETTY | 39.68 | 61.90 |  +22.22 |
| YAML | 39.68 | 34.92 | -4.76 |

## 3. Format-Specific Analysis
### 3.1 Detailed Analysis: CSV

#### 3.1.1 Performance Summary

- Token Duration Range: 78 - 110 seconds
- Token Cost Range: 6931 - 7318 tokens
- Wasted Token Range: 3128 - 3130 tokens
- Accuracy Range: 54.84 - 57.26 %
- Efficiency Score Range: 68.36 - 68.84

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

- Token Duration Range: 83 - 110 seconds
- Token Cost Range: 9086 - 9496 tokens
- Wasted Token Range: 2907 - 3395 tokens
- Accuracy Range: 64.25 - 68.01 %
- Efficiency Score Range: 66.93 - 70.84

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

- Token Duration Range: 73 - 106 seconds
- Token Cost Range: 13703 - 14624 tokens
- Wasted Token Range: 4166 - 5010 tokens
- Accuracy Range: 63.44 - 71.51 %
- Efficiency Score Range: 53.21 - 55.98

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

- Token Duration Range: 78 - 111 seconds
- Token Cost Range: 7391 - 11909 tokens
- Wasted Token Range: 2186 - 3713 tokens
- Accuracy Range: 68.82 - 70.43 %
- Efficiency Score Range: 62.58 - 77.83

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

- Token Duration Range: 81 - 111 seconds
- Token Cost Range: 7279 - 11903 tokens
- Wasted Token Range: 2289 - 4064 tokens
- Accuracy Range: 65.86 - 68.55 %
- Efficiency Score Range: 60.53 - 76.87

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

- Token Duration Range: 75 - 95 seconds
- Token Cost Range: 11265 - 12036 tokens
- Wasted Token Range: 3876 - 3948 tokens
- Accuracy Range: 65.59 - 67.20 %
- Efficiency Score Range: 61.05 - 62.33

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

- Token Duration Range: 79 - 103 seconds
- Token Cost Range: 15321 - 16509 tokens
- Wasted Token Range: 5148 - 5458 tokens
- Accuracy Range: 66.40 - 66.94 %
- Efficiency Score Range: 46.89 - 50.22

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

- Token Duration Range: 88 - 89 seconds
- Token Cost Range: 12104 - 12785 tokens
- Wasted Token Range: 4263 - 4330 tokens
- Accuracy Range: 64.78 - 66.13 %
- Efficiency Score Range: 57.96 - 59.14

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