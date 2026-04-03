# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: on
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
   - Optional: JSON_COMPACT 18386 tokens
   - Mandatory: JSON_COMPACT 18653 tokens
- Lowest read token cost:
   - Optional: JSON_COMPACT 9788 tokens
   - Mandatory: JSON_COMPACT 10315 tokens
- Lowest output token cost:
   - Optional: XML_COMPACT 7518 tokens
   - Mandatory: XML_PRETTY 7696 tokens
- Lowest output token cost drift:
   - Optional: JSON_PRETTY ↓ -7.09% ↑ 13.58%
   - Mandatory: JSON_PRETTY ↓ -9.60% ↑ 8.62%
- Highest accuracy:
   - Optional: YAML 80.64%
   - Mandatory: YAML 78.76%
- Lowest accuracy drift:
   - Optional: XML_COMPACT ↓ -1.79% ↑ 3.56%
   - Mandatory: YAML ↓ -2.73% ↑ 2.40%
- Most useful tokens:
   - Optional: YAML 21861 / 27110 tokens
   - Mandatory: JSON_PRETTY 22092 / 28736 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 85.29
   - Mandatory: JSON_COMPACT 82.32
- Lowest delta (optional-mandatory):
   - Total tokens: JSON_COMPACT -267 tokens
   - Accuracy: XML_PRETTY 0.26%
   - Token efficiency: JSON_PRETTY 2.76

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 29473 tokens
   - Mandatory: JSON_PRETTY 28736 tokens
- Highest read token cost:
   - Optional: XML_PRETTY 19583 tokens
   - Mandatory: XML_PRETTY 20114 tokens
- Highest output token cost:
   - Optional: YAML 13057 tokens
   - Mandatory: YAML 11294 tokens
- Highest output token drift:
   - Optional: JSON_COMPACT ↓ -96.08% ↑ 130.68%
   - Mandatory: XML_COMPACT ↓ -37.98% ↑ 71.76%
- Lowest accuracy:
   - Optional: JSON_PRETTY 72.85%
   - Mandatory: XML_PRETTY 73.39%
- Highest accuracy drift:
   - Optional: JSON_PRETTY ↓ -11.43% ↑ 10.71%
   - Mandatory: XML_PRETTY ↓ -13.19% ↑ 15.38%
- Most wasted tokens:
   - Optional: XML_PRETTY 7766 / 29473 tokens
   - Mandatory: XML_PRETTY 7400 / 27810 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 51.58
   - Mandatory: JSON_PRETTY 55.83
- Highest delta (optional-mandatory):
   - Total tokens: TOON_DEFAULT 3318 tokens
   - Accuracy: TOON_DEFAULT 5.78%
   - Token efficiency: XML_COMPACT 6.09

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 37s | JSON_COMPACT ≈ 10315 | XML_PRETTY ≈ 7696 | JSON_COMPACT ≈ 18653 | JSON_COMPACT ≈ 4512 | YAML ≈ 79% | YAML ≈ 77% | JSON_COMPACT ≈ 82 | JSON_COMPACT ≈ 81 |
| JSON_COMPACT (+87.6%) | XML_COMPACT (+24.6%) | JSON_COMPACT (+8.3%) | XML_COMPACT (+17.2%) | YAML (+20.5%) | JSON_PRETTY (-1.9%) | JSON_PRETTY (-1.1%) | XML_COMPACT (-11.7%) | XML_COMPACT (-11.4%) |
| XML_PRETTY (+91.7%) | TOON_DEFAULT (+36.7%) | TOON_DEFAULT (+14.5%) | TOON_DEFAULT (+22.8%) | XML_COMPACT (+23.7%) | JSON_COMPACT (-3.0%) | JSON_COMPACT (-2.4%) | TOON_DEFAULT (-15.1%) | TOON_DEFAULT (-16.3%) |
| XML_COMPACT (+102.6%) | YAML (+38.7%) | XML_COMPACT (+17.1%) | YAML (+37.2%) | TOON_DEFAULT (+29.7%) | TOON_DEFAULT (-4.3%) | XML_COMPACT (-3.3%) | YAML (-20.3%) | YAML (-21.0%) |
| YAML (+147.2%) | JSON_PRETTY (+72.8%) | JSON_PRETTY (+41.7%) | XML_PRETTY (+49.1%) | JSON_PRETTY (+47.2%) | XML_COMPACT (-4.3%) | TOON_DEFAULT (-4.9%) | XML_PRETTY (-32.1%) | JSON_PRETTY (-32.4%) |
| JSON_PRETTY (+153.7%) | XML_PRETTY (+95.0%) | YAML (+46.8%) | JSON_PRETTY (+54.1%) | XML_PRETTY (+64.0%) | XML_PRETTY (-5.4%) | XML_PRETTY (-4.9%) | JSON_PRETTY (-32.2%) | XML_PRETTY (-32.6%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 40s | JSON_COMPACT ≈ 9788 | XML_COMPACT ≈ 7518 | JSON_COMPACT ≈ 18386 | JSON_COMPACT ≈ 3855 | YAML ≈ 81% | TOON_DEFAULT ≈ 81% | JSON_COMPACT ≈ 85 | JSON_COMPACT ≈ 85 |
| XML_COMPACT (+60.3%) | XML_COMPACT (+26.4%) | JSON_COMPACT (+14.4%) | XML_COMPACT (+8.2%) | XML_COMPACT (+26.2%) | TOON_DEFAULT (-0.4%) | YAML (-1.2%) | XML_COMPACT (-7.6%) | XML_COMPACT (-7.7%) |
| XML_PRETTY (+94.1%) | TOON_DEFAULT (+41.6%) | JSON_PRETTY (+30.0%) | TOON_DEFAULT (+42.6%) | TOON_DEFAULT (+34.4%) | JSON_COMPACT (-1.6%) | JSON_COMPACT (-1.8%) | TOON_DEFAULT (-23.8%) | TOON_DEFAULT (-23.4%) |
| JSON_PRETTY (+111.8%) | YAML (+43.6%) | XML_PRETTY (+31.6%) | JSON_PRETTY (+45.1%) | YAML (+36.1%) | XML_COMPACT (-5.1%) | XML_COMPACT (-5.4%) | YAML (-26.3%) | YAML (-27.2%) |
| JSON_COMPACT (+120.4%) | JSON_PRETTY (+72.7%) | TOON_DEFAULT (+64.5%) | YAML (+47.5%) | JSON_PRETTY (+87.8%) | XML_PRETTY (-7.0%) | XML_PRETTY (-8.0%) | JSON_PRETTY (-31.3%) | JSON_PRETTY (-31.9%) |
| YAML (+136.8%) | XML_PRETTY (+100.1%) | YAML (+73.7%) | XML_PRETTY (+60.3%) | XML_PRETTY (+101.4%) | JSON_PRETTY (-7.8%) | JSON_PRETTY (-8.6%) | XML_PRETTY (-39.5%) | XML_PRETTY (-40.2%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 98% | JSON_PRETTY ≈ 72% | JSON_COMPACT ≈ 67% | JSON_COMPACT ≈ 71% |
| JSON_PRETTY (-1.2%) | XML_PRETTY (-2.5%) | XML_COMPACT (0.0%) | TOON_DEFAULT (-5.6%) |
| TOON_DEFAULT (-6.4%) | YAML (-2.5%) | JSON_PRETTY (-6.4%) | YAML (-11.1%) |
| XML_PRETTY (-9.1%) | XML_COMPACT (-3.7%) | YAML (-6.4%) | XML_PRETTY (-12.7%) |
| XML_COMPACT (-9.7%) | JSON_COMPACT (-3.7%) | TOON_DEFAULT (-7.1%) | XML_COMPACT (-15.9%) |
| JSON_COMPACT (-12.7%) | TOON_DEFAULT (-13.0%) | XML_PRETTY (-12.7%) | JSON_PRETTY (-22.2%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99% | JSON_COMPACT ≈ 83% | TOON_DEFAULT ≈ 71% | YAML ≈ 56% |
| TOON_DEFAULT (-3.0%) | TOON_DEFAULT (-0.0%) | XML_PRETTY (-7.9%) | XML_COMPACT (-0.0%) |
| JSON_COMPACT (-5.5%) | XML_COMPACT (-6.2%) | YAML (-7.9%) | JSON_COMPACT (-3.2%) |
| JSON_PRETTY (-9.1%) | YAML (-7.4%) | XML_COMPACT (-7.9%) | XML_PRETTY (-3.2%) |
| XML_PRETTY (-10.3%) | XML_PRETTY (-16.0%) | JSON_COMPACT (-9.5%) | JSON_PRETTY (-9.5%) |
| XML_COMPACT (-12.1%) | JSON_PRETTY (-16.1%) | JSON_PRETTY (-9.5%) | TOON_DEFAULT (-11.9%) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Char/Token | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10315 | 8338 | 18653 | 2.213 | 0.406 | 67.239 | 75.81 | 81.36 | 14140.587 | 4512.080 | 82.32 | 81.36 |
| JSON_COMPACT | opt | 9788 | 8598 | 18386 | 2.186 | 0.430 | 69.336 | 79.03 | 85.13 | 14530.193 | 3855.474 | 85.29 | 85.13 |
| JSON_PRETTY | man | 17828 | 10908 | 28736 | 1.762 | 0.268 | 87.965 | 76.88 | 55.04 | 22091.981 | 6643.686 | 55.83 | 55.04 |
| JSON_PRETTY | opt | 16899 | 9770 | 26669 | 1.752 | 0.273 | 78.788 | 72.85 | 57.97 | 19428.124 | 7240.543 | 58.60 | 57.97 |
| TOON_DEFAULT | man | 14096 | 8813 | 22909 | 1.851 | 0.325 | 71.073 | 74.46 | 68.11 | 17058.041 | 5850.959 | 69.88 | 68.11 |
| TOON_DEFAULT | opt | 13859 | 12368 | 26227 | 1.860 | 0.309 | 99.740 | 80.24 | 65.22 | 21044.411 | 5182.422 | 64.96 | 65.22 |
| XML_COMPACT | man | 12848 | 9013 | 21861 | 2.522 | 0.341 | 72.685 | 74.46 | 72.10 | 16277.701 | 5583.299 | 72.71 | 72.10 |
| XML_COMPACT | opt | 12368 | 7518 | 19886 | 2.517 | 0.380 | 60.629 | 75.54 | 78.58 | 15021.884 | 4864.116 | 78.80 | 78.58 |
| XML_PRETTY | man | 20114 | 7696 | 27810 | 1.993 | 0.264 | 62.062 | 73.39 | 54.87 | 20409.515 | 7400.152 | 55.89 | 54.87 |
| XML_PRETTY | opt | 19583 | 9890 | 29473 | 1.982 | 0.250 | 79.758 | 73.65 | 50.87 | 21706.864 | 7766.135 | 51.58 | 50.87 |
| YAML | man | 14306 | 11294 | 25600 | 1.789 | 0.308 | 91.078 | 78.76 | 64.29 | 20162.298 | 5437.369 | 65.62 | 64.29 |
| YAML | opt | 14053 | 13057 | 27110 | 1.799 | 0.297 | 105.296 | 80.64 | 62.00 | 21861.235 | 5248.432 | 62.86 | 62.00 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10315 | 9788 | -527 | -5.11 | 8338 | 8598 |  +260 |  +3.12 | 18653 | 18386 | -267 | -1.43 | 74.44 | 78.80 |  +4.36 | 81.36 | 85.13 |  +3.77 |
| JSON_PRETTY | 17828 | 16899 | -929 | -5.21 | 10908 | 9770 | -1138 | -10.43 | 28736 | 26669 | -2067 | -7.19 | 75.74 | 71.96 | -3.78 | 55.04 | 57.97 |  +2.94 |
| TOON_DEFAULT | 14096 | 13859 | -237 | -1.68 | 8813 | 12368 |  +3555 |  +40.34 | 22909 | 26227 |  +3318 |  +14.48 | 71.94 | 80.60 |  +8.66 | 68.11 | 65.22 | -2.90 |
| XML_COMPACT | 12848 | 12368 | -480 | -3.74 | 9013 | 7518 | -1495 | -16.59 | 21861 | 19886 | -1975 | -9.03 | 73.59 | 75.23 |  +1.64 | 72.10 | 78.58 |  +6.48 |
| XML_PRETTY | 20114 | 19583 | -531 | -2.64 | 7696 | 9890 |  +2194 |  +28.51 | 27810 | 29473 |  +1663 |  +5.98 | 71.93 | 72.63 |  +0.70 | 54.87 | 50.87 | -4.00 |
| YAML | 14306 | 14053 | -253 | -1.77 | 11294 | 13057 |  +1763 |  +15.61 | 25600 | 27110 |  +1510 |  +5.90 | 76.86 | 79.41 |  +2.55 | 64.29 | 62.00 | -2.29 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 24 | 429.792 | 0.77 | 69429 | 0.120 | 559.91 | 69453 | 429.912 | 448.08 |
| JSON_COMPACT | opt | 9 | 1087.556 | 0.29 | 87923 | 0.098 | 709.06 | 87932 | 1087.654 | 567.31 |
| JSON_PRETTY | man | 267 | 66.772 | 8.61 | 93654 | 0.116 | 755.27 | 93921 | 66.888 | 605.94 |
| JSON_PRETTY | opt | 272 | 62.129 | 8.77 | 84236 | 0.116 | 679.32 | 84508 | 62.245 | 545.21 |
| TOON_DEFAULT | man | 10 | 1409.600 | 0.32 | 74257 | 0.118 | 598.85 | 74267 | 1409.718 | 479.14 |
| TOON_DEFAULT | opt | 20 | 692.950 | 0.65 | 96657 | 0.127 | 779.49 | 96677 | 693.077 | 623.72 |
| XML_COMPACT | man | 11 | 1168.000 | 0.35 | 74985 | 0.120 | 604.72 | 74996 | 1168.120 | 483.85 |
| XML_COMPACT | opt | 9 | 1374.222 | 0.29 | 63937 | 0.118 | 515.62 | 63946 | 1374.340 | 412.55 |
| XML_PRETTY | man | 11 | 1828.545 | 0.35 | 70941 | 0.108 | 572.11 | 70952 | 1828.653 | 457.76 |
| XML_PRETTY | opt | 11 | 1780.273 | 0.35 | 77410 | 0.128 | 624.27 | 77421 | 1780.401 | 499.49 |
| YAML | man | 12 | 1192.167 | 0.39 | 91492 | 0.123 | 737.84 | 91504 | 1192.290 | 590.35 |
| YAML | opt | 8 | 1756.625 | 0.26 | 94465 | 0.138 | 761.81 | 94473 | 1756.763 | 609.50 |

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
| JSON_COMPACT | man | 2.213 | 15.125 | 332.742 | 0.406 |
| JSON_COMPACT | opt | 2.186 | 15.512 | 315.742 | 0.430 |
| JSON_PRETTY | man | 1.762 | 26.141 | 575.097 | 0.268 |
| JSON_PRETTY | opt | 1.752 | 26.781 | 545.129 | 0.273 |
| TOON_DEFAULT | man | 1.851 | 20.669 | 454.710 | 0.325 |
| TOON_DEFAULT | opt | 1.860 | 21.964 | 447.065 | 0.309 |
| XML_COMPACT | man | 2.522 | 18.839 | 414.452 | 0.341 |
| XML_COMPACT | opt | 2.517 | 19.601 | 398.968 | 0.380 |
| XML_PRETTY | man | 1.993 | 29.493 | 648.839 | 0.264 |
| XML_PRETTY | opt | 1.982 | 31.035 | 631.710 | 0.250 |
| YAML | man | 1.789 | 20.977 | 461.484 | 0.308 |
| YAML | opt | 1.799 | 22.271 | 453.323 | 0.297 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 2.213 | 2.186 | -0.027 | -1.22 | 15.125 | 15.512 |  +0.387 |  +2.56 | 332.742 | 315.742 | -17.000 | -5.11 | 0.406 | 0.430 |  +0.024 |  +5.91 |
| JSON_PRETTY | 1.762 | 1.752 | -0.010 | -0.57 | 26.141 | 26.781 |  +0.640 |  +2.45 | 575.097 | 545.129 | -29.968 | -5.21 | 0.268 | 0.273 |  +0.005 |  +1.87 |
| TOON_DEFAULT | 1.851 | 1.860 |  +0.009 |  +0.49 | 20.669 | 21.964 |  +1.295 |  +6.27 | 454.710 | 447.065 | -7.645 | -1.68 | 0.325 | 0.309 | -0.016 | -4.77 |
| XML_COMPACT | 2.522 | 2.517 | -0.005 | -0.20 | 18.839 | 19.601 |  +0.762 |  +4.04 | 414.452 | 398.968 | -15.484 | -3.74 | 0.341 | 0.380 |  +0.039 |  +11.44 |
| XML_PRETTY | 1.993 | 1.982 | -0.011 | -0.55 | 29.493 | 31.035 |  +1.542 |  +5.23 | 648.839 | 631.710 | -17.129 | -2.64 | 0.264 | 0.250 | -0.014 | -5.30 |
| YAML | 1.789 | 1.799 |  +0.010 |  +0.56 | 20.977 | 22.271 |  +1.294 |  +6.17 | 461.484 | 453.323 | -8.161 | -1.77 | 0.308 | 0.297 | -0.011 | -3.57 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 18653 | 14141 | 4512 | 75.81 | 74.44 | 82.32 | 81.36 |
| JSON_COMPACT | opt | 18386 | 14530 | 3855 | 79.03 | 78.80 | 85.29 | 85.13 |
| JSON_PRETTY | man | 28736 | 22092 | 6644 | 76.88 | 75.74 | 55.83 | 55.04 |
| JSON_PRETTY | opt | 26669 | 19428 | 7241 | 72.85 | 71.96 | 58.60 | 57.97 |
| TOON_DEFAULT | man | 22909 | 17058 | 5851 | 74.46 | 71.94 | 69.88 | 68.11 |
| TOON_DEFAULT | opt | 26227 | 21044 | 5182 | 80.24 | 80.60 | 64.96 | 65.22 |
| XML_COMPACT | man | 21861 | 16278 | 5583 | 74.46 | 73.59 | 72.71 | 72.10 |
| XML_COMPACT | opt | 19886 | 15022 | 4864 | 75.54 | 75.23 | 78.80 | 78.58 |
| XML_PRETTY | man | 27810 | 20410 | 7400 | 73.39 | 71.93 | 55.89 | 54.87 |
| XML_PRETTY | opt | 29473 | 21707 | 7766 | 73.65 | 72.63 | 51.58 | 50.87 |
| YAML | man | 25600 | 20162 | 5437 | 78.76 | 76.86 | 65.62 | 64.29 |
| YAML | opt | 27110 | 21861 | 5248 | 80.64 | 79.41 | 62.86 | 62.00 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 18653 | 18386 | -267 | -1.43 | 14141 | 14531 |  +390 |  +2.76 | 4512 | 3855 | -657 | -14.55 | 75.81 | 79.03 |  +3.22 | 82.32 | 85.294 |  +2.97 |  +3.61 |
| JSON_PRETTY | 28736 | 26669 | -2067 | -7.19 | 22092 | 19428 | -2664 | -12.06 | 6644 | 7241 |  +597 |  +8.98 | 76.88 | 72.85 | -4.03 | 55.83 | 58.596 |  +2.76 |  +4.95 |
| TOON_DEFAULT | 22909 | 26227 |  +3318 |  +14.48 | 17058 | 21044 |  +3986 |  +23.37 | 5851 | 5182 | -669 | -11.43 | 74.46 | 80.24 |  +5.78 | 69.88 | 64.963 | -4.92 | -7.03 |
| XML_COMPACT | 21861 | 19886 | -1975 | -9.03 | 16278 | 15022 | -1256 | -7.71 | 5583 | 4864 | -719 | -12.88 | 74.46 | 75.54 |  +1.08 | 72.71 | 78.799 |  +6.09 |  +8.38 |
| XML_PRETTY | 27810 | 29473 |  +1663 |  +5.98 | 20410 | 21707 |  +1297 |  +6.36 | 7400 | 7766 |  +366 |  +4.95 | 73.39 | 73.65 |  +0.26 | 55.89 | 51.582 | -4.31 | -7.71 |
| YAML | 25600 | 27110 |  +1510 |  +5.90 | 20162 | 21861 |  +1699 |  +8.43 | 5437 | 5248 | -189 | -3.48 | 78.76 | 80.64 |  +1.88 | 65.62 | 62.858 | -2.76 | -4.21 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| JSON_COMPACT | man | 94 | 30 | 0 | 75.81 |
| JSON_COMPACT | opt | 98 | 26 | 0 | 79.03 |
| JSON_PRETTY | man | 95 | 29 | 0 | 76.88 |
| JSON_PRETTY | opt | 90 | 34 | 0 | 72.85 |
| TOON_DEFAULT | man | 92 | 32 | 0 | 74.46 |
| TOON_DEFAULT | opt | 100 | 25 | 0 | 80.24 |
| XML_COMPACT | man | 92 | 32 | 0 | 74.46 |
| XML_COMPACT | opt | 94 | 30 | 0 | 75.54 |
| XML_PRETTY | man | 91 | 33 | 0 | 73.39 |
| XML_PRETTY | opt | 91 | 33 | 0 | 73.65 |
| YAML | man | 98 | 26 | 0 | 78.76 |
| YAML | opt | 100 | 24 | 0 | 80.64 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 94 | 98 |  +4 |  +4.26 | 30 | 26 | -4 | -13.33 | 0 | 0 | 0 | 0.00 | 75.81 | 79.03 |  +3.22 |
| JSON_PRETTY | 95 | 90 | -5 | -5.26 | 29 | 34 |  +5 |  +17.24 | 0 | 0 | 0 | 0.00 | 76.88 | 72.85 | -4.03 |
| TOON_DEFAULT | 92 | 100 |  +8 |  +8.70 | 32 | 25 | -7 | -21.88 | 0 | 0 | 0 | 0.00 | 74.46 | 80.24 |  +5.78 |
| XML_COMPACT | 92 | 94 |  +2 |  +2.17 | 32 | 30 | -2 | -6.25 | 0 | 0 | 0 | 0.00 | 74.46 | 75.54 |  +1.08 |
| XML_PRETTY | 91 | 91 | 0 | 0.00 | 33 | 33 | 0 | 0.00 | 0 | 0 | 0 | 0.00 | 73.39 | 73.65 |  +0.26 |
| YAML | 98 | 100 |  +2 |  +2.04 | 26 | 24 | -2 | -7.69 | 0 | 0 | 0 | 0.00 | 78.76 | 80.64 |  +1.88 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 75.81 | 84.85 | 67.90 | 66.67 | 71.43 |
| JSON_COMPACT | opt | 79.03 | 93.94 | 82.72 | 61.91 | 52.38 |
| JSON_PRETTY | man | 76.88 | 96.36 | 71.60 | 60.31 | 49.20 |
| JSON_PRETTY | opt | 72.85 | 90.30 | 66.67 | 61.90 | 46.03 |
| TOON_DEFAULT | man | 74.46 | 91.21 | 58.64 | 59.52 | 65.87 |
| TOON_DEFAULT | opt | 80.24 | 96.36 | 82.72 | 71.43 | 43.65 |
| XML_COMPACT | man | 74.46 | 87.88 | 67.90 | 66.67 | 55.55 |
| XML_COMPACT | opt | 75.54 | 87.27 | 76.54 | 63.49 | 55.55 |
| XML_PRETTY | man | 73.39 | 88.48 | 69.14 | 53.97 | 58.73 |
| XML_PRETTY | opt | 73.65 | 89.09 | 66.67 | 63.49 | 52.38 |
| YAML | man | 78.76 | 97.58 | 69.14 | 60.31 | 60.32 |
| YAML | opt | 80.64 | 99.39 | 75.31 | 63.49 | 55.56 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 84.85 | 93.94 |  +9.09 |
| JSON_PRETTY | 96.36 | 90.30 | -6.06 |
| TOON_DEFAULT | 91.21 | 96.36 |  +5.15 |
| XML_COMPACT | 87.88 | 87.27 | -0.61 |
| XML_PRETTY | 88.48 | 89.09 |  +0.61 |
| YAML | 97.58 | 99.39 |  +1.82 |

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

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: on
- **Structure**: nested
- **Formats Tested**: JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML
- **Record Counts**: 31
- **Total Test Cases**: 12

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
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_flat_all_formats_and_variants_off\BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_flat_all_formats_and_variants_on\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_off\BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)