# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-17
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Extended Thinking**: on
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
   - Optional: JSON_COMPACT 10124 tokens
   - Mandatory: JSON_COMPACT 10651 tokens
- Lowest output token cost drift:
   - Optional: XML_COMPACT ↓ -0.39 % ↑ 0.48 %
   - Mandatory: JSON_COMPACT ↓ -0.60 % ↑ 1.19 %
- Highest accuracy:
   - Optional: TOON_DEFAULT 67.07 %
   - Mandatory: JSON_COMPACT 68.82 %
- Lowest accuracy drift:
   - Optional: JSON_COMPACT ↓ -2.02 % ↑ 2.82 %
   - Mandatory: YAML ↓ -0.79 % ↑ 1.57 %
- Most useful tokens:
   - Optional: XML_PRETTY 12428 / 19927 tokens
   - Mandatory: XML_PRETTY 13307 / 20456 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 76.64
   - Mandatory: JSON_COMPACT 76.62
- Lowest delta (optional-mandatory):
   - Total tokens: TOON_DEFAULT -227 tokens
   - Accuracy: XML_COMPACT -1.35 %
   - Token efficiency: JSON_COMPACT 0.02

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 19927 tokens
   - Mandatory: XML_PRETTY 20456 tokens
- Highest output token drift:
   - Optional: JSON_PRETTY ↓ -96.05 % ↑ 49.34 %
   - Mandatory: JSON_PRETTY ↓ -25.06 % ↑ 49.22 %
- Lowest accuracy:
   - Optional: JSON_PRETTY 61.02 %
   - Mandatory: XML_PRETTY 65.05 %
- Highest accuracy drift:
   - Optional: YAML ↓ -12.56 % ↑ 11.73 %
   - Mandatory: XML_PRETTY ↓ -11.98 % ↑ 14.05 %
- Most wasted tokens:
   - Optional: XML_PRETTY 7499 / 19927 tokens
   - Mandatory: XML_PRETTY 7149 / 20456 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 45.22
   - Mandatory: XML_PRETTY 45.56
- Highest delta (optional-mandatory):
   - Total tokens: JSON_PRETTY -1148 tokens
   - Accuracy: JSON_PRETTY -5.38 %
   - Token efficiency: TOON_DEFAULT 1.69

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| toon_default ≈ 37017 s | json_compact ≈ 10651  | json_compact ≈ 3321  | json_compact ≈ 69 % | json_compact ≈ 69 % | json_compact ≈ 77  | json_compact ≈ 76  |
| json_compact ( +87.6 %) | xml_compact ( +23.9 %) | xml_compact ( +32.4 %) | yaml (-0.5 %) | yaml (-0.5 %) | xml_compact (-11.6 %) | xml_compact (-11.0 %) |
| xml_pretty ( +91.7 %) | toon_default ( +35.6 %) | yaml ( +39.9 %) | xml_compact (-2.1 %) | xml_compact (-1.5 %) | yaml (-15.6 %) | yaml (-15.6 %) |
| xml_compact ( +102.6 %) | yaml ( +37.5 %) | toon_default ( +49.6 %) | json_pretty (-2.4 %) | json_pretty (-1.7 %) | toon_default (-17.3 %) | toon_default (-18.1 %) |
| yaml ( +147.2 %) | json_pretty ( +71.6 %) | json_pretty ( +84.9 %) | toon_default (-3.2 %) | xml_pretty (-3.7 %) | json_pretty (-31.0 %) | json_pretty (-30.4 %) |
| json_pretty ( +153.7 %) | xml_pretty ( +92.1 %) | xml_pretty ( +115.3 %) | xml_pretty (-3.8 %) | toon_default (-4.1 %) | xml_pretty (-40.5 %) | xml_pretty (-40.5 %) |


##### Optional

| ↑ Total Duration) | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| toon_default ≈ 39891 s | json_compact ≈ 10124  | json_compact ≈ 3374  | toon_default ≈ 67 % | toon_default ≈ 69 % | json_compact ≈ 77  | json_compact ≈ 78  |
| xml_compact ( +60.3 %) | xml_compact ( +25.6 %) | xml_compact ( +30.7 %) | json_compact (-0.4 %) | json_compact (-1.1 %) | xml_compact (-11.0 %) | xml_compact (-11.2 %) |
| xml_pretty ( +94.1 %) | toon_default ( +40.4 %) | toon_default ( +38.7 %) | yaml (-0.7 %) | yaml (-2.1 %) | toon_default (-15.1 %) | toon_default (-14.2 %) |
| json_pretty ( +111.8 %) | yaml ( +42.1 %) | yaml ( +43.3 %) | xml_compact (-1.8 %) | xml_compact (-2.9 %) | yaml (-16.4 %) | yaml (-16.8 %) |
| json_compact ( +120.4 %) | json_pretty ( +69.2 %) | json_pretty ( +97.9 %) | xml_pretty (-4.7 %) | xml_pretty (-6.4 %) | json_pretty (-31.6 %) | json_pretty (-31.8 %) |
| yaml ( +136.8 %) | xml_pretty ( +96.8 %) | xml_pretty ( +122.2 %) | json_pretty (-6.0 %) | json_pretty (-7.5 %) | xml_pretty (-41.0 %) | xml_pretty (-41.2 %) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval % | ↓ Structure Awareness % | ↓ Filtering % | ↓ Aggregation % |
|---|---|---|---|
| yaml ≈ 74 % | json_pretty ≈ 72 % | json_compact ≈ 67 % | json_compact ≈ 71 % |
| json_pretty (-1.2 %) | xml_pretty (-2.5 %) | xml_compact (0.0 %) | toon_default (-5.6 %) |
| toon_default (-2.7 %) | yaml (-2.5 %) | json_pretty (-6.4 %) | yaml (-11.1 %) |
| xml_compact (-3.6 %) | xml_compact (-3.7 %) | yaml (-6.4 %) | xml_pretty (-12.7 %) |
| xml_pretty (-4.2 %) | json_compact (-3.7 %) | toon_default (-7.1 %) | xml_compact (-15.9 %) |
| json_compact (-4.9 %) | toon_default (-13.0 %) | xml_pretty (-12.7 %) | json_pretty (-22.2 %) |


##### Optional

| ↓ Field Retrieval % | ↓ Structure Awareness % | ↓ Filtering % | ↓ Aggregation % |
|---|---|---|---|
| yaml ≈ 67 % | json_compact ≈ 83 % | toon_default ≈ 71 % | yaml ≈ 56 % |
| toon_default (-0.6 %) | toon_default (-0.0 %) | xml_pretty (-7.9 %) | xml_compact (-0.0 %) |
| json_compact (-1.2 %) | xml_compact (-6.2 %) | yaml (-7.9 %) | json_compact (-3.2 %) |
| xml_compact (-3.0 %) | yaml (-7.4 %) | xml_compact (-7.9 %) | xml_pretty (-3.2 %) |
| json_pretty (-3.6 %) | xml_pretty (-16.0 %) | json_compact (-9.5 %) | json_pretty (-9.5 %) |
| xml_pretty (-3.6 %) | json_pretty (-16.1 %) | json_pretty (-9.5 %) | toon_default (-11.9 %) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Tokens/Char | Info/Token | Token/Answer | Acc (%) | Wtd Acc (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10315 | 336 | 10651 | 2.213 | 0.646 | 2.710 | 68.82 | 76.41 | 7330.018 | 3320.982 | 76.62 | 76.41 |
| JSON_COMPACT | opt | 9788 | 336 | 10124 | 2.186 | 0.659 | 2.707 | 66.67 | 77.82 | 6749.449 | 3374.218 | 76.64 | 77.82 |
| JSON_PRETTY | man | 17828 | 447 | 18275 | 1.762 | 0.363 | 3.605 | 66.40 | 53.16 | 12134.600 | 6140.400 | 52.83 | 53.16 |
| JSON_PRETTY | opt | 16899 | 228 | 17127 | 1.752 | 0.356 | 1.836 | 61.02 | 53.05 | 10450.692 | 6675.975 | 52.39 | 53.05 |
| TOON_DEFAULT | man | 14096 | 342 | 14438 | 1.851 | 0.454 | 2.757 | 65.59 | 62.58 | 9469.775 | 4968.058 | 63.38 | 62.58 |
| TOON_DEFAULT | opt | 13859 | 352 | 14211 | 1.860 | 0.472 | 2.837 | 67.07 | 66.75 | 9531.150 | 4679.600 | 65.08 | 66.75 |
| XML_COMPACT | man | 12848 | 345 | 13193 | 2.522 | 0.505 | 2.780 | 66.67 | 67.98 | 8795.551 | 4397.116 | 67.75 | 67.98 |
| XML_COMPACT | opt | 12368 | 344 | 12712 | 2.517 | 0.514 | 2.777 | 65.32 | 69.08 | 8303.696 | 4408.637 | 68.19 | 69.08 |
| XML_PRETTY | man | 20114 | 342 | 20456 | 1.993 | 0.318 | 2.761 | 65.05 | 45.45 | 13306.845 | 7149.488 | 45.56 | 45.45 |
| XML_PRETTY | opt | 19583 | 344 | 19927 | 1.982 | 0.313 | 2.774 | 62.37 | 45.73 | 12428.470 | 7498.530 | 45.22 | 45.73 |
| YAML | man | 14306 | 338 | 14644 | 1.789 | 0.466 | 2.723 | 68.28 | 64.47 | 9998.696 | 4644.971 | 64.67 | 64.47 |
| YAML | opt | 14053 | 336 | 14389 | 1.799 | 0.461 | 2.710 | 66.40 | 64.77 | 9554.296 | 4834.704 | 64.09 | 64.77 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Acc Man (%) | Acc Opt (%) | Diff (%) | Wtd Acc Man (%) | Wtd Acc Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10651 | 10124 | -527 | -4.95 | 68.82 | 66.67 | -2.15 | 68.53 | 68.35 | -0.18 | 76.62 | 76.64 |  +0.02 | 76.41 | 77.82 |  +1.40 |
| JSON_PRETTY | 18275 | 17127 | -1148 | -6.28 | 66.40 | 61.02 | -5.38 | 66.87 | 61.96 | -4.91 | 52.83 | 52.39 | -0.44 | 53.16 | 53.05 | -0.11 |
| TOON_DEFAULT | 14438 | 14211 | -227 | -1.57 | 65.59 | 67.07 |  +1.48 | 64.44 | 69.46 |  +5.02 | 63.38 | 65.08 |  +1.69 | 62.58 | 66.75 |  +4.17 |
| XML_COMPACT | 13193 | 12713 | -480 | -3.64 | 66.67 | 65.32 | -1.35 | 67.00 | 66.59 | -0.41 | 67.75 | 68.19 |  +0.45 | 67.98 | 69.08 |  +1.11 |
| XML_PRETTY | 20456 | 19927 | -529 | -2.59 | 65.05 | 62.37 | -2.68 | 64.88 | 63.09 | -1.79 | 45.56 | 45.22 | -0.34 | 45.45 | 45.73 |  +0.28 |
| YAML | 14644 | 14389 | -255 | -1.74 | 68.28 | 66.40 | -1.88 | 67.99 | 67.37 | -0.62 | 64.67 | 64.09 | -0.58 | 64.47 | 64.77 |  +0.30 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 24 | 429.792 | 0.77 | 69429 | 0.005 | 559.91 | 69453 | 429.797 | 448.08 |
| JSON_COMPACT | opt | 9 | 1087.556 | 0.29 | 87923 | 0.004 | 709.06 | 87932 | 1087.560 | 567.31 |
| JSON_PRETTY | man | 267 | 66.772 | 8.61 | 93654 | 0.005 | 755.27 | 93921 | 66.777 | 605.94 |
| JSON_PRETTY | opt | 272 | 62.129 | 8.77 | 84236 | 0.003 | 679.32 | 84508 | 62.132 | 545.21 |
| TOON_DEFAULT | man | 10 | 1409.600 | 0.32 | 74257 | 0.005 | 598.85 | 74267 | 1409.605 | 479.14 |
| TOON_DEFAULT | opt | 20 | 692.950 | 0.65 | 96657 | 0.004 | 779.49 | 96677 | 692.954 | 623.72 |
| XML_COMPACT | man | 11 | 1168.000 | 0.35 | 74985 | 0.005 | 604.72 | 74996 | 1168.005 | 483.85 |
| XML_COMPACT | opt | 9 | 1374.222 | 0.29 | 63937 | 0.005 | 515.62 | 63946 | 1374.227 | 412.55 |
| XML_PRETTY | man | 11 | 1828.545 | 0.35 | 70941 | 0.005 | 572.11 | 70952 | 1828.550 | 457.76 |
| XML_PRETTY | opt | 11 | 1780.273 | 0.35 | 77410 | 0.004 | 624.27 | 77421 | 1780.277 | 499.49 |
| YAML | man | 12 | 1192.167 | 0.39 | 91492 | 0.004 | 737.84 | 91504 | 1192.171 | 590.35 |
| YAML | opt | 8 | 1756.625 | 0.26 | 94465 | 0.004 | 761.81 | 94473 | 1756.629 | 609.50 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) | Total Man (s) | Total Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 24 | 9 | -15 | -62.50 | 69.43 | 87.92 |  +18.49 |  +26.64 | 69.45 | 87.93 |  +18.48 |  +26.61 |
| JSON_PRETTY | 267 | 272 |  +5 |  +1.87 | 93.65 | 84.24 | -9.42 | -10.06 | 93.92 | 84.51 | -9.41 | -10.02 |
| TOON_DEFAULT | 10 | 20 |  +10 |  +100.00 | 74.26 | 96.66 |  +22.40 |  +30.17 | 74.27 | 77.14 |  +2.87 |  +3.87 |
| XML_COMPACT | 11 | 9 | -2 | -18.18 | 74.99 | 63.94 | -11.05 | -14.73 | 75.00 | 63.95 | -11.05 | -14.73 |
| XML_PRETTY | 11 | 11 | 0 | 0.00 | 70.94 | 77.41 |  +6.47 |  +9.12 | 70.95 | 77.42 |  +6.47 |  +9.12 |
| YAML | 12 | 8 | -4 | -33.33 | 91.49 | 94.47 |  +2.97 |  +3.25 | 91.50 | 94.47 |  +2.97 |  +3.24 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Char/Token | Token/Value | Token/Object | Info/Token |
|---|---|---|---|---|---|
| JSON_COMPACT | man | 2.213 | 15.125 | 332.742 | 0.646 |
| JSON_COMPACT | opt | 2.186 | 15.512 | 315.742 | 0.659 |
| JSON_PRETTY | man | 1.762 | 26.141 | 575.097 | 0.363 |
| JSON_PRETTY | opt | 1.752 | 26.781 | 545.129 | 0.356 |
| TOON_DEFAULT | man | 1.851 | 20.669 | 454.710 | 0.454 |
| TOON_DEFAULT | opt | 1.860 | 21.964 | 447.065 | 0.472 |
| XML_COMPACT | man | 2.522 | 18.839 | 414.452 | 0.505 |
| XML_COMPACT | opt | 2.517 | 19.601 | 398.968 | 0.514 |
| XML_PRETTY | man | 1.993 | 29.493 | 648.839 | 0.318 |
| XML_PRETTY | opt | 1.982 | 31.035 | 631.710 | 0.313 |
| YAML | man | 1.789 | 20.977 | 461.484 | 0.466 |
| YAML | opt | 1.799 | 22.271 | 453.323 | 0.461 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 2.213 | 2.186 | -0.027 | -1.22 | 15.125 | 15.512 |  +0.387 |  +2.56 | 332.742 | 315.742 | -17.000 | -5.11 | 0.646 | 0.659 |  +0.013 |  +2.01 |
| JSON_PRETTY | 1.762 | 1.752 | -0.010 | -0.57 | 26.141 | 26.781 |  +0.640 |  +2.45 | 575.097 | 545.129 | -29.968 | -5.21 | 0.363 | 0.356 | -0.007 | -1.93 |
| TOON_DEFAULT | 1.851 | 1.860 |  +0.009 |  +0.49 | 20.669 | 21.964 |  +1.295 |  +6.27 | 454.710 | 447.065 | -7.645 | -1.68 | 0.454 | 0.472 |  +0.018 |  +3.96 |
| XML_COMPACT | 2.522 | 2.517 | -0.005 | -0.20 | 18.839 | 19.601 |  +0.762 |  +4.04 | 414.452 | 398.968 | -15.484 | -3.74 | 0.505 | 0.514 |  +0.009 |  +1.78 |
| XML_PRETTY | 1.993 | 1.982 | -0.011 | -0.55 | 29.493 | 31.035 |  +1.542 |  +5.23 | 648.839 | 631.710 | -17.129 | -2.64 | 0.318 | 0.313 | -0.005 | -1.57 |
| YAML | 1.789 | 1.799 |  +0.010 |  +0.56 | 20.977 | 22.271 |  +1.294 |  +6.17 | 461.484 | 453.323 | -8.161 | -1.77 | 0.466 | 0.461 | -0.005 | -1.07 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Acc (%) | Wtd Acc (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10651 | 7330 | 3321 | 68.82 | 68.53 | 76.62 | 76.41 |
| JSON_COMPACT | opt | 10124 | 6749 | 3374 | 66.67 | 68.35 | 76.64 | 77.82 |
| JSON_PRETTY | man | 18275 | 12135 | 6140 | 66.40 | 66.87 | 52.83 | 53.16 |
| JSON_PRETTY | opt | 17127 | 10451 | 6676 | 61.02 | 61.96 | 52.39 | 53.05 |
| TOON_DEFAULT | man | 14438 | 9470 | 4968 | 65.59 | 64.44 | 63.38 | 62.58 |
| TOON_DEFAULT | opt | 14211 | 9531 | 4680 | 67.07 | 69.46 | 65.08 | 66.75 |
| XML_COMPACT | man | 13193 | 8796 | 4397 | 66.67 | 67.00 | 67.75 | 67.98 |
| XML_COMPACT | opt | 12712 | 8304 | 4409 | 65.32 | 66.59 | 68.19 | 69.08 |
| XML_PRETTY | man | 20456 | 13307 | 7149 | 65.05 | 64.88 | 45.56 | 45.45 |
| XML_PRETTY | opt | 19927 | 12428 | 7499 | 62.37 | 63.09 | 45.22 | 45.73 |
| YAML | man | 14644 | 9999 | 4645 | 68.28 | 67.99 | 64.67 | 64.47 |
| YAML | opt | 14389 | 9554 | 4835 | 66.40 | 67.37 | 64.09 | 64.77 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Acc (%) Man | Acc (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10651 | 10124 | -527 | -4.95 | 7330 | 6749 | -581 | -7.92 | 3321 | 3374 |  +53 |  +1.60 | 68.82 | 66.67 | -2.15 | 76.62 | 76.64 |  +0.02 |  +0.03 |
| JSON_PRETTY | 18275 | 17127 | -1148 | -6.28 | 12135 | 10451 | -1684 | -13.88 | 6140 | 6676 |  +536 |  +8.72 | 66.40 | 61.02 | -5.38 | 52.83 | 52.392 | -0.44 | -0.83 |
| TOON_DEFAULT | 14438 | 14211 | -227 | -1.57 | 9470 | 9531 |  +61 |  +0.65 | 4968 | 4680 | -288 | -5.81 | 65.59 | 67.07 |  +1.48 | 63.38 | 65.0765 |  +1.69 |  +2.67 |
| XML_COMPACT | 13193 | 12713 | -480 | -3.64 | 8796 | 8304 | -492 | -5.59 | 4397 | 4409 |  +12 |  +0.26 | 66.67 | 65.32 | -1.35 | 67.75 | 68.194 |  +0.45 |  +0.66 |
| XML_PRETTY | 20456 | 19927 | -529 | -2.59 | 13307 | 12429 | -878 | -6.60 | 7149 | 7498 |  +349 |  +4.88 | 65.05 | 62.37 | -2.68 | 45.56 | 45.222 | -0.34 | -0.75 |
| YAML | 14644 | 14389 | -255 | -1.74 | 9999 | 9555 | -444 | -4.44 | 4645 | 4835 |  +190 |  +4.08 | 68.28 | 66.40 | -1.88 | 64.67 | 64.091 | -0.58 | -0.89 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Acc (%) |
|---|---|---|---|---|---|
| JSON_COMPACT | man | 85 | 39 | 0 | 68.82 |
| JSON_COMPACT | opt | 83 | 41 | 0 | 66.67 |
| JSON_PRETTY | man | 82 | 42 | 0 | 66.40 |
| JSON_PRETTY | opt | 76 | 48 | 0 | 61.02 |
| TOON_DEFAULT | man | 81 | 43 | 0 | 65.59 |
| TOON_DEFAULT | opt | 83 | 41 | 0 | 67.07 |
| XML_COMPACT | man | 83 | 41 | 0 | 66.67 |
| XML_COMPACT | opt | 81 | 43 | 0 | 65.32 |
| XML_PRETTY | man | 81 | 43 | 0 | 65.05 |
| XML_PRETTY | opt | 77 | 47 | 0 | 62.37 |
| YAML | man | 85 | 39 | 0 | 68.28 |
| YAML | opt | 82 | 42 | 0 | 66.40 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Acc (%) Man | Acc (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 85 | 83 | -2 | -2.35 | 39 | 41 |  +2 |  +5.13 | 0 | 0 | 0 | 0.00 | 68.82 | 66.67 | -2.15 |
| JSON_PRETTY | 82 | 76 | -6 | -7.32 | 42 | 48 |  +6 |  +14.29 | 0 | 0 | 0 | 0.00 | 66.40 | 61.02 | -5.38 |
| TOON_DEFAULT | 81 | 83 |  +2 |  +2.47 | 43 | 41 | -2 | -4.65 | 0 | 0 | 0 | 0.00 | 65.59 | 67.07 |  +1.48 |
| XML_COMPACT | 83 | 81 | -2 | -2.41 | 41 | 43 |  +2 |  +4.88 | 0 | 0 | 0 | 0.00 | 66.67 | 65.32 | -1.35 |
| XML_PRETTY | 81 | 77 | -4 | -4.94 | 43 | 47 |  +4 |  +9.30 | 0 | 0 | 0 | 0.00 | 65.05 | 62.37 | -2.68 |
| YAML | 85 | 82 | -3 | -3.53 | 39 | 42 |  +3 |  +7.69 | 0 | 0 | 0 | 0.00 | 68.28 | 66.40 | -1.88 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Acc (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 68.82 | 69.09 | 67.90 | 66.67 | 71.43 |
| JSON_COMPACT | opt | 66.67 | 66.06 | 82.72 | 61.91 | 52.38 |
| JSON_PRETTY | man | 66.40 | 72.73 | 71.60 | 60.31 | 49.20 |
| JSON_PRETTY | opt | 61.02 | 63.64 | 66.67 | 61.90 | 46.03 |
| TOON_DEFAULT | man | 65.59 | 71.21 | 58.64 | 59.52 | 65.87 |
| TOON_DEFAULT | opt | 67.07 | 66.66 | 82.72 | 71.43 | 43.65 |
| XML_COMPACT | man | 66.67 | 70.30 | 67.90 | 66.67 | 55.55 |
| XML_COMPACT | opt | 65.32 | 64.24 | 76.54 | 63.49 | 55.55 |
| XML_PRETTY | man | 65.05 | 69.70 | 69.14 | 53.97 | 58.73 |
| XML_PRETTY | opt | 62.37 | 63.63 | 66.67 | 63.49 | 52.38 |
| YAML | man | 68.28 | 73.94 | 69.14 | 60.31 | 60.32 |
| YAML | opt | 66.40 | 67.27 | 75.31 | 63.49 | 55.56 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 69.09 | 66.06 | -3.03 |
| JSON_PRETTY | 72.73 | 63.64 | -9.09 |
| TOON_DEFAULT | 71.21 | 66.66 | -4.55 |
| XML_COMPACT | 70.30 | 64.24 | -6.06 |
| XML_PRETTY | 69.70 | 63.63 | -6.06 |
| YAML | 73.94 | 67.27 | -6.67 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 67.90 | 82.72 |  +14.82 |
| JSON_PRETTY | 71.60 | 66.67 | -4.94 |
| TOON_DEFAULT | 58.64 | 82.72 |  +24.07 |
| XML_COMPACT | 67.90 | 76.54 |  +8.64 |
| XML_PRETTY | 69.14 | 66.67 | -2.47 |
| YAML | 69.14 | 75.31 |  +6.17 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 66.67 | 61.91 | -4.76 |
| JSON_PRETTY | 60.31 | 61.90 |  +1.59 |
| TOON_DEFAULT | 59.52 | 71.43 |  +11.91 |
| XML_COMPACT | 66.67 | 63.49 | -3.18 |
| XML_PRETTY | 53.97 | 63.49 |  +9.52 |
| YAML | 60.31 | 63.49 |  +3.18 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 71.43 | 52.38 | -19.05 |
| JSON_PRETTY | 49.20 | 46.03 | -3.17 |
| TOON_DEFAULT | 65.87 | 43.65 | -22.22 |
| XML_COMPACT | 55.55 | 55.55 |  +0.00 |
| XML_PRETTY | 58.73 | 52.38 | -6.35 |
| YAML | 60.32 | 55.56 | -4.76 |

## 3. Format-Specific Analysis
### 3.1 Detailed Analysis: JSON_COMPACT

#### 3.1.1 Performance Summary

- Token Duration Range: 69 - 88 seconds
- Token Cost Range: 10124 - 10651 tokens
- Wasted Token Range: 3321 - 3374 tokens
- Accuracy Range: 66.67 - 68.82 %
- Efficiency Score Range: 76.62 - 76.64

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

- Token Duration Range: 85 - 94 seconds
- Token Cost Range: 17127 - 18275 tokens
- Wasted Token Range: 6140 - 6676 tokens
- Accuracy Range: 61.02 - 66.40 %
- Efficiency Score Range: 52.39 - 52.83

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

- Token Duration Range: 37 - 40 seconds
- Token Cost Range: 14211 - 14438 tokens
- Wasted Token Range: 4680 - 4968 tokens
- Accuracy Range: 65.59 - 67.07 %
- Efficiency Score Range: 63.38 - 65.08

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

- Token Duration Range: 64 - 75 seconds
- Token Cost Range: 12712 - 13193 tokens
- Wasted Token Range: 4397 - 4409 tokens
- Accuracy Range: 65.32 - 66.67 %
- Efficiency Score Range: 67.75 - 68.19

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

- Token Duration Range: 71 - 77 seconds
- Token Cost Range: 19927 - 20456 tokens
- Wasted Token Range: 7149 - 7499 tokens
- Accuracy Range: 62.37 - 65.05 %
- Efficiency Score Range: 45.22 - 45.56

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

- Token Duration Range: 92 - 94 seconds
- Token Cost Range: 14389 - 14644 tokens
- Wasted Token Range: 4645 - 4835 tokens
- Accuracy Range: 66.40 - 68.28 %
- Efficiency Score Range: 64.09 - 64.67

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
- **Test Date**: 2026-03-17
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Extended Thinking**: on
- **Structure**: nested
- **Formats Tested**: json_compact, json_pretty, toon_default, xml_compact, xml_pretty, yaml
- **Record Counts**: 31
- **Total Test Cases**: 12

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