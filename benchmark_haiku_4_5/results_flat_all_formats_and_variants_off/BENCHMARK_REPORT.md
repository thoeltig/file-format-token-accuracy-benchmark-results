# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-17
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Extended Thinking**: off
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
   - Optional: CSV 7032 tokens
   - Mandatory: CSV 7263 tokens
- Lowest output token cost drift:
   - Optional: XML_COMPACT ↓ -0.32 % ↑ 0.32 %
   - Mandatory: JSON_PRETTY ↓ -1.22 % ↑ 0.77 %
- Highest accuracy:
   - Optional: YAML 66.13 %
   - Mandatory: JSON_PRETTY 71.77 %
- Lowest accuracy drift:
   - Optional: YAML ↓ -3.66 % ↑ 3.66 %
   - Mandatory: JSON_PRETTY ↓ -2.24 % ↑ 1.13 %
- Most useful tokens:
   - Optional: XML_PRETTY 9990 / 15419 tokens
   - Mandatory: XML_PRETTY 10950 / 16490 tokens
- Highest token efficiency (%/token):
   - Optional: CSV 68.92
   - Mandatory: TOON_DEFAULT 75.29
- Lowest delta (optional-mandatory):
   - Total tokens: CSV -231 tokens
   - Accuracy: XML_COMPACT 0.26 %
   - Token efficiency: JSON_COMPACT -0.73

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 15419 tokens
   - Mandatory: XML_PRETTY 16490 tokens
- Highest output token drift:
   - Optional: JSON_PRETTY ↓ -97.84 % ↑ 34.20 %
   - Mandatory: CSV ↓ -96.26 % ↑ 25.10 %
- Lowest accuracy:
   - Optional: CSV 55.65 %
   - Mandatory: CSV 61.45 %
- Highest accuracy drift:
   - Optional: CSV ↓ -13.05 % ↑ 10.13 %
   - Mandatory: TOON_DEFAULT ↓ -19.49 % ↑ 6.94 %
- Most wasted tokens:
   - Optional: XML_PRETTY 5429 / 15419 tokens
   - Mandatory: XML_PRETTY 5541 / 16490 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 48.77
   - Mandatory: XML_PRETTY 46.51
- Highest delta (optional-mandatory):
   - Total tokens: TOON_DEFAULT 4344 tokens
   - Accuracy: JSON_PRETTY -6.45 %
   - Token efficiency: TOON_DEFAULT -16.19

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| toon_default ≈ 50392 s | csv ≈ 7263  | toon_default ≈ 2485  | json_pretty ≈ 72 % | json_pretty ≈ 73 % | toon_default ≈ 75  | toon_default ≈ 76  |
| yaml ( +50.6 %) | toon_default ( +4.0 %) | csv ( +12.7 %) | json_compact (-4.6 %) | json_compact (-4.5 %) | csv (-4.0 %) | csv (-4.5 %) |
| xml_compact ( +59.3 %) | json_compact ( +32.2 %) | json_compact ( +26.9 %) | toon_default (-4.7 %) | toon_default (-4.6 %) | json_compact (-8.6 %) | json_compact (-8.4 %) |
| csv ( +60.6 %) | xml_compact ( +68.7 %) | json_pretty ( +66.0 %) | xml_pretty (-5.4 %) | xml_pretty (-6.0 %) | xml_compact (-21.9 %) | xml_compact (-22.8 %) |
| xml_pretty ( +71.6 %) | yaml ( +77.3 %) | xml_compact ( +73.6 %) | yaml (-6.5 %) | yaml (-7.7 %) | yaml (-24.0 %) | yaml (-25.0 %) |
| json_pretty ( +91.0 %) | json_pretty ( +101.2 %) | yaml ( +79.7 %) | xml_compact (-7.0 %) | xml_compact (-8.2 %) | json_pretty (-25.3 %) | json_pretty (-25.1 %) |
| json_compact ( +111.6 %) | xml_pretty ( +127.1 %) | xml_pretty ( +123.0 %) | csv (-10.3 %) | csv (-10.8 %) | xml_pretty (-38.2 %) | xml_pretty (-38.4 %) |


##### Optional

| ↑ Total Duration) | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| toon_default ≈ 42037 s | csv ≈ 7032  | csv ≈ 3119  | yaml ≈ 66 % | yaml ≈ 68 % | csv ≈ 69  | csv ≈ 70  |
| xml_pretty ( +76.7 %) | json_compact ( +29.0 %) | json_compact ( +5.6 %) | json_pretty (-0.8 %) | xml_compact (-0.7 %) | json_compact (-1.2 %) | json_compact (-2.0 %) |
| json_compact ( +79.9 %) | xml_compact ( +63.4 %) | xml_compact ( +28.8 %) | xml_compact (-1.1 %) | json_pretty (-1.0 %) | xml_compact (-10.9 %) | xml_compact (-10.3 %) |
| json_pretty ( +102.2 %) | toon_default ( +69.2 %) | yaml ( +31.4 %) | xml_pretty (-1.3 %) | xml_pretty (-1.5 %) | yaml (-12.6 %) | yaml (-12.3 %) |
| csv ( +105.9 %) | yaml ( +72.0 %) | toon_default ( +38.8 %) | json_compact (-2.4 %) | toon_default (-3.0 %) | toon_default (-14.3 %) | toon_default (-14.4 %) |
| xml_compact ( +108.6 %) | json_pretty ( +93.8 %) | json_pretty ( +51.5 %) | toon_default (-2.5 %) | json_compact (-3.4 %) | json_pretty (-20.5 %) | json_pretty (-20.1 %) |
| yaml ( +147.4 %) | xml_pretty ( +119.3 %) | xml_pretty ( +74.1 %) | csv (-10.5 %) | csv (-10.6 %) | xml_pretty (-29.2 %) | xml_pretty (-28.8 %) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval % | ↓ Structure Awareness % | ↓ Filtering % | ↓ Aggregation % |
|---|---|---|---|
| json_compact ≈ 74 % | json_pretty ≈ 84 % | json_pretty ≈ 67 % | json_pretty ≈ 59 % |
| yaml (-0.3 %) | toon_default (-4.1 %) | xml_pretty (-3.2 %) | xml_compact (-1.6 %) |
| json_pretty (-0.9 %) | json_compact (-6.2 %) | json_compact (-3.6 %) | csv (-5.4 %) |
| xml_pretty (-0.9 %) | xml_pretty (-13.6 %) | csv (-4.8 %) | toon_default (-9.5 %) |
| toon_default (-3.3 %) | xml_compact (-16.1 %) | yaml (-4.8 %) | yaml (-9.5 %) |
| xml_compact (-4.6 %) | csv (-17.3 %) | toon_default (-6.4 %) | xml_pretty (-11.1 %) |
| csv (-11.8 %) | yaml (-19.8 %) | xml_compact (-9.5 %) | json_compact (-18.3 %) |


##### Optional

| ↓ Field Retrieval % | ↓ Structure Awareness % | ↓ Filtering % | ↓ Aggregation % |
|---|---|---|---|
| xml_compact ≈ 67 % | xml_compact ≈ 81 % | yaml ≈ 73 % | json_compact ≈ 52 % |
| yaml (0.0 %) | xml_pretty (-1.2 %) | json_pretty (-6.4 %) | xml_pretty (0.0 %) |
| json_pretty (-0.7 %) | json_pretty (-5.9 %) | csv (-10.2 %) | toon_default (-4.8 %) |
| toon_default (-1.4 %) | yaml (-7.4 %) | json_compact (-11.1 %) | json_pretty (-4.8 %) |
| json_compact (-1.5 %) | toon_default (-7.8 %) | xml_compact (-11.1 %) | yaml (-6.3 %) |
| xml_pretty (-3.0 %) | json_compact (-11.9 %) | toon_default (-12.2 %) | xml_compact (-11.1 %) |
| csv (-10.2 %) | csv (-19.3 %) | xml_pretty (-14.3 %) | csv (-16.2 %) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Tokens/Char | Info/Token | Token/Answer | Acc (%) | Wtd Acc (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7022 | 241 | 7263 | 1.438 | 0.846 | 1.940 | 61.45 | 72.77 | 4462.868 | 2799.732 | 72.25 | 72.77 |
| CSV | opt | 6728 | 304 | 7032 | 1.423 | 0.791 | 2.448 | 55.65 | 69.99 | 3913.085 | 3118.515 | 68.92 | 69.99 |
| JSON_COMPACT | man | 9296 | 304 | 9600 | 2.143 | 0.699 | 2.452 | 67.14 | 69.79 | 6445.440 | 3154.560 | 68.84 | 69.79 |
| JSON_COMPACT | opt | 8768 | 305 | 9073 | 2.108 | 0.702 | 2.458 | 63.71 | 68.62 | 5780.281 | 3292.519 | 68.11 | 68.62 |
| JSON_PRETTY | man | 14311 | 302 | 14613 | 1.691 | 0.491 | 2.433 | 71.77 | 57.07 | 10487.511 | 4125.156 | 56.21 | 57.07 |
| JSON_PRETTY | opt | 13395 | 231 | 13626 | 1.677 | 0.479 | 1.863 | 65.32 | 55.88 | 8900.503 | 4725.497 | 54.82 | 55.88 |
| TOON_DEFAULT | man | 7298 | 258 | 7556 | 1.393 | 0.888 | 2.076 | 67.11 | 76.17 | 5070.496 | 2485.004 | 75.29 | 76.17 |
| TOON_DEFAULT | opt | 11590 | 309 | 11899 | 1.694 | 0.535 | 2.493 | 63.62 | 59.93 | 7570.251 | 4328.916 | 59.10 | 59.93 |
| XML_COMPACT | man | 11950 | 303 | 12253 | 2.315 | 0.529 | 2.441 | 64.79 | 58.78 | 7938.503 | 4314.164 | 58.80 | 58.78 |
| XML_COMPACT | opt | 11181 | 309 | 11490 | 2.288 | 0.566 | 2.492 | 65.05 | 62.80 | 7474.245 | 4015.755 | 61.39 | 62.80 |
| XML_PRETTY | man | 16186 | 304 | 16490 | 1.931 | 0.403 | 2.454 | 66.40 | 46.91 | 10949.581 | 5540.752 | 46.51 | 46.91 |
| XML_PRETTY | opt | 15117 | 302 | 15419 | 1.913 | 0.420 | 2.435 | 64.79 | 49.82 | 9989.970 | 5429.030 | 48.77 | 49.82 |
| YAML | man | 12574 | 302 | 12876 | 1.661 | 0.507 | 2.433 | 65.32 | 57.15 | 8410.386 | 4465.281 | 57.20 | 57.15 |
| YAML | opt | 11791 | 304 | 12095 | 1.646 | 0.547 | 2.452 | 66.13 | 61.40 | 7998.423 | 4096.577 | 60.23 | 61.40 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Acc Man (%) | Acc Opt (%) | Diff (%) | Wtd Acc Man (%) | Wtd Acc Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7263 | 7032 | -231 | -3.18 | 61.45 | 55.65 | -5.80 | 62.19 | 57.17 | -5.02 | 72.25 | 68.92 | -3.33 | 72.77 | 69.99 | -2.78 |
| JSON_COMPACT | 9600 | 9073 | -527 | -5.49 | 67.14 | 63.71 | -3.43 | 68.50 | 64.44 | -4.06 | 68.84 | 68.11 | -0.73 | 69.79 | 68.62 | -1.17 |
| JSON_PRETTY | 14613 | 13626 | -987 | -6.75 | 71.77 | 65.32 | -6.45 | 72.99 | 66.84 | -6.15 | 56.21 | 54.82 | -1.39 | 57.07 | 55.88 | -1.18 |
| TOON_DEFAULT | 7556 | 11900 |  +4344 |  +57.49 | 67.11 | 63.62 | -3.49 | 68.37 | 64.81 | -3.56 | 75.29 | 59.10 | -16.19 | 76.17 | 59.93 | -16.24 |
| XML_COMPACT | 12253 | 11490 | -763 | -6.23 | 64.79 | 65.05 |  +0.26 | 64.76 | 67.06 |  +2.30 | 58.80 | 61.39 |  +2.60 | 58.78 | 62.80 |  +4.02 |
| XML_PRETTY | 16490 | 15419 | -1071 | -6.49 | 66.40 | 64.79 | -1.61 | 66.97 | 66.28 | -0.69 | 46.51 | 48.77 |  +2.26 | 46.91 | 49.82 |  +2.91 |
| YAML | 12876 | 12095 | -781 | -6.07 | 65.32 | 66.13 |  +0.81 | 65.26 | 67.80 |  +2.54 | 57.20 | 60.23 |  +3.04 | 57.15 | 61.40 |  +4.25 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 28 | 250.786 | 0.90 | 80925 | 0.003 | 652.62 | 80953 | 250.789 | 522.28 |
| CSV | opt | 10 | 672.800 | 0.32 | 86539 | 0.004 | 697.90 | 86549 | 672.804 | 558.38 |
| JSON_COMPACT | man | 36 | 258.222 | 1.16 | 106585 | 0.003 | 859.56 | 106621 | 258.225 | 687.88 |
| JSON_COMPACT | opt | 9 | 974.222 | 0.29 | 75636 | 0.004 | 609.97 | 75645 | 974.226 | 488.03 |
| JSON_PRETTY | man | 13 | 1100.846 | 0.42 | 96223 | 0.003 | 775.99 | 96236 | 1100.849 | 620.88 |
| JSON_PRETTY | opt | 15 | 893.000 | 0.48 | 85002 | 0.003 | 685.50 | 85017 | 893.003 | 548.50 |
| TOON_DEFAULT | man | 33 | 221.152 | 1.06 | 95028 | 0.003 | 766.35 | 95061 | 221.154 | 613.29 |
| TOON_DEFAULT | opt | 14 | 827.857 | 0.45 | 74050 | 0.005 | 597.17 | 74064 | 827.861 | 477.83 |
| XML_COMPACT | man | 16 | 746.875 | 0.52 | 80267 | 0.004 | 647.31 | 80283 | 746.879 | 517.95 |
| XML_COMPACT | opt | 15 | 745.400 | 0.48 | 87671 | 0.004 | 707.02 | 87686 | 745.404 | 565.72 |
| XML_PRETTY | man | 18 | 899.222 | 0.58 | 86445 | 0.004 | 697.13 | 86463 | 899.226 | 557.82 |
| XML_PRETTY | opt | 9 | 1679.667 | 0.29 | 74275 | 0.004 | 598.99 | 74284 | 1679.671 | 479.25 |
| YAML | man | 11 | 1143.091 | 0.35 | 75884 | 0.004 | 611.97 | 75895 | 1143.095 | 489.65 |
| YAML | opt | 8 | 1473.875 | 0.26 | 103981 | 0.003 | 838.56 | 103989 | 1473.878 | 670.90 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) | Total Man (s) | Total Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 28 | 10 | -18 | -64.29 | 80.93 | 86.54 |  +5.61 |  +6.94 | 80.95 | 86.55 |  +5.60 |  +6.91 |
| JSON_COMPACT | 36 | 9 | -27 | -75.00 | 106.58 | 75.64 | -30.95 | -29.04 | 106.62 | 75.65 | -30.98 | -29.05 |
| JSON_PRETTY | 13 | 15 |  +2 |  +15.38 | 96.22 | 85.00 | -11.22 | -11.66 | 96.24 | 85.02 | -11.22 | -11.66 |
| TOON_DEFAULT | 33 | 14 | -19 | -57.58 | 95.03 | 74.05 | -20.98 | -22.08 | 95.06 | 86.71 | -8.35 | -8.79 |
| XML_COMPACT | 16 | 15 | -1 | -6.25 | 80.27 | 87.67 |  +7.40 |  +9.22 | 80.28 | 87.69 |  +7.40 |  +9.22 |
| XML_PRETTY | 18 | 9 | -9 | -50.00 | 86.44 | 74.28 | -12.17 | -14.08 | 86.46 | 74.28 | -12.18 | -14.09 |
| YAML | 11 | 8 | -3 | -27.27 | 75.88 | 103.98 |  +28.10 |  +37.03 | 75.89 | 103.99 |  +28.09 |  +37.02 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Char/Token | Token/Value | Token/Object | Info/Token |
|---|---|---|---|---|---|
| CSV | man | 1.438 | 10.296 | 226.516 | 0.846 |
| CSV | opt | 1.423 | 10.662 | 217.032 | 0.791 |
| JSON_COMPACT | man | 2.143 | 13.630 | 299.871 | 0.699 |
| JSON_COMPACT | opt | 2.108 | 13.895 | 282.839 | 0.702 |
| JSON_PRETTY | man | 1.691 | 20.984 | 461.645 | 0.491 |
| JSON_PRETTY | opt | 1.677 | 21.228 | 432.097 | 0.479 |
| TOON_DEFAULT | man | 1.393 | 10.701 | 235.419 | 0.888 |
| TOON_DEFAULT | opt | 1.694 | 18.368 | 373.871 | 0.535 |
| XML_COMPACT | man | 2.315 | 17.522 | 385.484 | 0.529 |
| XML_COMPACT | opt | 2.288 | 17.719 | 360.677 | 0.566 |
| XML_PRETTY | man | 1.931 | 23.733 | 522.129 | 0.403 |
| XML_PRETTY | opt | 1.913 | 23.957 | 487.645 | 0.420 |
| YAML | man | 1.661 | 18.437 | 405.613 | 0.507 |
| YAML | opt | 1.646 | 18.686 | 380.355 | 0.547 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.438 | 1.423 | -0.015 | -1.04 | 10.296 | 10.662 |  +0.366 |  +3.55 | 226.516 | 217.032 | -9.484 | -4.19 | 0.846 | 0.791 | -0.055 | -6.50 |
| JSON_COMPACT | 2.143 | 2.108 | -0.035 | -1.63 | 13.630 | 13.895 |  +0.265 |  +1.94 | 299.871 | 282.839 | -17.032 | -5.68 | 0.699 | 0.702 |  +0.003 |  +0.43 |
| JSON_PRETTY | 1.691 | 1.677 | -0.014 | -0.83 | 20.984 | 21.228 |  +0.244 |  +1.16 | 461.645 | 432.097 | -29.548 | -6.40 | 0.491 | 0.479 | -0.012 | -2.44 |
| TOON_DEFAULT | 1.393 | 1.694 |  +0.301 |  +21.61 | 10.701 | 18.368 |  +7.667 |  +71.65 | 235.419 | 373.871 |  +138.452 |  +58.81 | 0.888 | 0.535 | -0.353 | -39.75 |
| XML_COMPACT | 2.315 | 2.288 | -0.027 | -1.17 | 17.522 | 17.719 |  +0.197 |  +1.12 | 385.484 | 360.677 | -24.807 | -6.44 | 0.529 | 0.566 |  +0.037 |  +6.99 |
| XML_PRETTY | 1.931 | 1.913 | -0.018 | -0.93 | 23.733 | 23.957 |  +0.224 |  +0.94 | 522.129 | 487.645 | -34.484 | -6.60 | 0.403 | 0.420 |  +0.017 |  +4.22 |
| YAML | 1.661 | 1.646 | -0.015 | -0.90 | 18.437 | 18.686 |  +0.249 |  +1.35 | 405.613 | 380.355 | -25.258 | -6.23 | 0.507 | 0.547 |  +0.040 |  +7.89 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Acc (%) | Wtd Acc (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| CSV | man | 7263 | 4463 | 2800 | 61.45 | 62.19 | 72.25 | 72.77 |
| CSV | opt | 7032 | 3913 | 3119 | 55.65 | 57.17 | 68.92 | 69.99 |
| JSON_COMPACT | man | 9600 | 6445 | 3155 | 67.14 | 68.50 | 68.84 | 69.79 |
| JSON_COMPACT | opt | 9073 | 5780 | 3293 | 63.71 | 64.44 | 68.11 | 68.62 |
| JSON_PRETTY | man | 14613 | 10488 | 4125 | 71.77 | 72.99 | 56.21 | 57.07 |
| JSON_PRETTY | opt | 13626 | 8901 | 4725 | 65.32 | 66.84 | 54.82 | 55.88 |
| TOON_DEFAULT | man | 7556 | 5070 | 2485 | 67.11 | 68.37 | 75.29 | 76.17 |
| TOON_DEFAULT | opt | 11899 | 7570 | 4329 | 63.62 | 64.81 | 59.10 | 59.93 |
| XML_COMPACT | man | 12253 | 7939 | 4314 | 64.79 | 64.76 | 58.80 | 58.78 |
| XML_COMPACT | opt | 11490 | 7474 | 4016 | 65.05 | 67.06 | 61.39 | 62.80 |
| XML_PRETTY | man | 16490 | 10950 | 5541 | 66.40 | 66.97 | 46.51 | 46.91 |
| XML_PRETTY | opt | 15419 | 9990 | 5429 | 64.79 | 66.28 | 48.77 | 49.82 |
| YAML | man | 12876 | 8410 | 4465 | 65.32 | 65.26 | 57.20 | 57.15 |
| YAML | opt | 12095 | 7998 | 4097 | 66.13 | 67.80 | 60.23 | 61.40 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Acc (%) Man | Acc (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7263 | 7032 | -231 | -3.18 | 4463 | 3913 | -550 | -12.32 | 2800 | 3119 |  +319 |  +11.39 | 61.45 | 55.65 | -5.80 | 72.25 | 68.923 | -3.33 | -4.61 |
| JSON_COMPACT | 9600 | 9073 | -527 | -5.49 | 6445 | 5780 | -665 | -10.32 | 3155 | 3293 |  +138 |  +4.37 | 67.14 | 63.71 | -3.43 | 68.84 | 68.105 | -0.73 | -1.06 |
| JSON_PRETTY | 14613 | 13626 | -987 | -6.75 | 10488 | 8901 | -1587 | -15.13 | 4125 | 4725 |  +600 |  +14.55 | 71.77 | 65.32 | -6.45 | 56.21 | 54.821 | -1.39 | -2.48 |
| TOON_DEFAULT | 7556 | 11900 |  +4344 |  +57.49 | 5070 | 7570 |  +2500 |  +49.30 | 2485 | 4329 |  +1844 |  +74.20 | 67.11 | 63.62 | -3.49 | 75.29 | 59.096999999999994 | -16.19 | -21.50 |
| XML_COMPACT | 12253 | 11490 | -763 | -6.22 | 7939 | 7475 | -464 | -5.85 | 4314 | 4016 | -298 | -6.92 | 64.79 | 65.05 |  +0.26 | 58.80 | 61.393 |  +2.60 |  +4.42 |
| XML_PRETTY | 16490 | 15419 | -1071 | -6.50 | 10950 | 9990 | -960 | -8.76 | 5541 | 5429 | -112 | -2.02 | 66.40 | 64.79 | -1.61 | 46.51 | 48.775 |  +2.26 |  +4.87 |
| YAML | 12876 | 12095 | -781 | -6.06 | 8410 | 7998 | -412 | -4.90 | 4465 | 4096 | -369 | -8.26 | 65.32 | 66.13 |  +0.81 | 57.20 | 60.234 |  +3.04 |  +5.31 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Acc (%) |
|---|---|---|---|---|---|
| CSV | man | 76 | 48 | 0 | 61.45 |
| CSV | opt | 69 | 55 | 0 | 55.65 |
| JSON_COMPACT | man | 83 | 41 | 0 | 67.14 |
| JSON_COMPACT | opt | 79 | 45 | 0 | 63.71 |
| JSON_PRETTY | man | 89 | 35 | 0 | 71.77 |
| JSON_PRETTY | opt | 81 | 43 | 0 | 65.32 |
| TOON_DEFAULT | man | 83 | 41 | 0 | 67.11 |
| TOON_DEFAULT | opt | 79 | 45 | 0 | 63.62 |
| XML_COMPACT | man | 80 | 44 | 0 | 64.79 |
| XML_COMPACT | opt | 81 | 43 | 0 | 65.05 |
| XML_PRETTY | man | 82 | 42 | 0 | 66.40 |
| XML_PRETTY | opt | 80 | 44 | 0 | 64.79 |
| YAML | man | 81 | 43 | 0 | 65.32 |
| YAML | opt | 82 | 42 | 0 | 66.13 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Acc (%) Man | Acc (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 76 | 69 | -7 | -9.21 | 48 | 55 |  +7 |  +14.58 | 0 | 0 | 0 | 0.00 | 61.45 | 55.65 | -5.80 |
| JSON_COMPACT | 83 | 79 | -4 | -4.82 | 41 | 45 |  +4 |  +9.76 | 0 | 0 | 0 | 0.00 | 67.14 | 63.71 | -3.43 |
| JSON_PRETTY | 89 | 81 | -8 | -8.99 | 35 | 43 |  +8 |  +22.86 | 0 | 0 | 0 | 0.00 | 71.77 | 65.32 | -6.45 |
| TOON_DEFAULT | 83 | 79 | -4 | -4.82 | 41 | 45 |  +4 |  +9.76 | 0 | 0 | 0 | 0.00 | 67.11 | 63.62 | -3.49 |
| XML_COMPACT | 80 | 81 |  +1 |  +1.25 | 44 | 43 | -1 | -2.27 | 0 | 0 | 0 | 0.00 | 64.79 | 65.05 |  +0.26 |
| XML_PRETTY | 82 | 80 | -2 | -2.44 | 42 | 44 |  +2 |  +4.76 | 0 | 0 | 0 | 0.00 | 66.40 | 64.79 | -1.61 |
| YAML | 81 | 82 |  +1 |  +1.23 | 43 | 42 | -1 | -2.33 | 0 | 0 | 0 | 0.00 | 65.32 | 66.13 |  +0.81 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Acc (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| CSV | man | 61.45 | 61.82 | 66.67 | 61.90 | 53.33 |
| CSV | opt | 55.65 | 57.09 | 62.22 | 62.86 | 36.19 |
| JSON_COMPACT | man | 67.14 | 73.64 | 77.78 | 63.09 | 40.48 |
| JSON_COMPACT | opt | 63.71 | 65.82 | 69.63 | 61.90 | 52.38 |
| JSON_PRETTY | man | 71.77 | 72.73 | 83.95 | 66.67 | 58.73 |
| JSON_PRETTY | opt | 65.32 | 66.54 | 75.56 | 66.67 | 47.62 |
| TOON_DEFAULT | man | 67.11 | 70.31 | 79.84 | 60.32 | 49.21 |
| TOON_DEFAULT | opt | 63.62 | 65.86 | 73.66 | 60.84 | 47.62 |
| XML_COMPACT | man | 64.79 | 69.09 | 67.90 | 57.14 | 57.14 |
| XML_COMPACT | opt | 65.05 | 67.27 | 81.48 | 61.90 | 41.27 |
| XML_PRETTY | man | 66.40 | 72.73 | 70.37 | 63.49 | 47.62 |
| XML_PRETTY | opt | 64.79 | 64.24 | 80.25 | 58.73 | 52.38 |
| YAML | man | 65.32 | 73.34 | 64.20 | 61.90 | 49.21 |
| YAML | opt | 66.13 | 67.27 | 74.08 | 73.02 | 46.03 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 61.82 | 57.09 | -4.73 |
| JSON_COMPACT | 73.64 | 65.82 | -7.82 |
| JSON_PRETTY | 72.73 | 66.54 | -6.19 |
| TOON_DEFAULT | 70.31 | 65.86 | -4.45 |
| XML_COMPACT | 69.09 | 67.27 | -1.82 |
| XML_PRETTY | 72.73 | 64.24 | -8.49 |
| YAML | 73.34 | 67.27 | -6.07 |

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

## 3. Format-Specific Analysis
### 3.1 Detailed Analysis: CSV

#### 3.1.1 Performance Summary

- Token Duration Range: 81 - 87 seconds
- Token Cost Range: 7032 - 7263 tokens
- Wasted Token Range: 2800 - 3119 tokens
- Accuracy Range: 55.65 - 61.45 %
- Efficiency Score Range: 68.92 - 72.25

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

- Token Duration Range: 76 - 107 seconds
- Token Cost Range: 9073 - 9600 tokens
- Wasted Token Range: 3155 - 3293 tokens
- Accuracy Range: 63.71 - 67.14 %
- Efficiency Score Range: 68.11 - 68.84

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

- Token Duration Range: 85 - 96 seconds
- Token Cost Range: 13626 - 14613 tokens
- Wasted Token Range: 4125 - 4725 tokens
- Accuracy Range: 65.32 - 71.77 %
- Efficiency Score Range: 54.82 - 56.21

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

- Token Duration Range: 42 - 50 seconds
- Token Cost Range: 7556 - 11899 tokens
- Wasted Token Range: 2485 - 4329 tokens
- Accuracy Range: 63.62 - 67.11 %
- Efficiency Score Range: 59.10 - 75.29

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

- Token Duration Range: 80 - 88 seconds
- Token Cost Range: 11490 - 12253 tokens
- Wasted Token Range: 4016 - 4314 tokens
- Accuracy Range: 64.79 - 65.05 %
- Efficiency Score Range: 58.80 - 61.39

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

- Token Duration Range: 74 - 86 seconds
- Token Cost Range: 15419 - 16490 tokens
- Wasted Token Range: 5429 - 5541 tokens
- Accuracy Range: 64.79 - 66.40 %
- Efficiency Score Range: 46.51 - 48.77

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

- Token Duration Range: 76 - 104 seconds
- Token Cost Range: 12095 - 12876 tokens
- Wasted Token Range: 4097 - 4465 tokens
- Accuracy Range: 65.32 - 66.13 %
- Efficiency Score Range: 57.20 - 60.23

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
- **Test Date**: 2026-03-17
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Extended Thinking**: off
- **Structure**: flat
- **Formats Tested**: csv, json_compact, json_pretty, toon_default, xml_compact, xml_pretty, yaml
- **Record Counts**: 31
- **Total Test Cases**: 14

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