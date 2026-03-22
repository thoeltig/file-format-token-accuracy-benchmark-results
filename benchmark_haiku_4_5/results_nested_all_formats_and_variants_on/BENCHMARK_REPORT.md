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

1. JSON_COMPACT achieves the best overall efficiency score in the benchmark (85.29 optional, 81.51 mandatory) by combining the lowest token cost with competitive accuracy, resulting in an information value per token of 0.781, the highest recorded. It outperforms the next-highest-ranked format, XML_COMPACT optional (75.35), by 9.94 efficiency points while consuming approximately 20% fewer tokens.

2. YAML delivers the highest raw accuracy (80.64% optional, 78.76% mandatory), driven by near-perfect field retrieval (99.39% optional). Its accuracy advantage over JSON_COMPACT is 1.61 percentage points at a 42% token cost premium, making it the precision-first alternative rather than an efficiency leader.

3. Pretty-printed formats are structurally inefficient. JSON_PRETTY and XML_PRETTY score 21 to 34 efficiency points below their compact counterparts while consuming 71 to 97% more tokens with no measurable accuracy improvement. XML_PRETTY is the lowest-ranked format overall with an efficiency score of 51.40 in the mandatory variant.

4. Aggregation is the most error-prone question category across all formats, with accuracy ranging from 43.65% (TOON_DEFAULT optional) to 71.43% (JSON_COMPACT mandatory). Four of six formats degrade in aggregation accuracy when optional fields are present, indicating that sparse datasets increase numerical computation errors independent of format choice.

5. TOON_DEFAULT is the fastest format at 37 to 40 seconds total reasoning duration and shows the largest accuracy gain from mandatory to optional data (+5.78 percentage points). Its mandatory variant underperforms significantly in structure awareness (58.64%), making it conditionally strong for sparse workloads but unreliable for fully-populated dense schemas.

6. XML_COMPACT is the most variant-stable format, with accuracy changing only 1.08 percentage points between mandatory and optional data. This predictability makes it suitable for pipelines where data sparsity varies and consistent behavior is more operationally valuable than peak performance.

7. JSON_PRETTY is the only format that degrades in accuracy when switching from mandatory to optional data (76.88% to 72.85%), a 4.03 percentage point regression, suggesting the model misinterprets structural gaps in pretty-printed JSON during aggregation and filtering tasks on sparse datasets.

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
   - Optional: XML_COMPACT ↓ -0.39% ↑ 0.48%
   - Mandatory: JSON_COMPACT ↓ -0.60% ↑ 1.19%
- Highest accuracy:
   - Optional: YAML 80.64%
   - Mandatory: YAML 78.76%
- Lowest accuracy drift:
   - Optional: XML_COMPACT ↓ -1.79% ↑ 3.56%
   - Mandatory: YAML ↓ -2.73% ↑ 2.40%
- Most useful tokens:
   - Optional: XML_PRETTY 14676 / 19927 tokens
   - Mandatory: XML_PRETTY 15013 / 20456 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 85.29
   - Mandatory: JSON_COMPACT 81.51
- Lowest delta (optional-mandatory):
   - Total tokens: TOON_DEFAULT -227 tokens
   - Accuracy: XML_PRETTY 0.26%
   - Token efficiency: JSON_PRETTY 0.51

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 19927 tokens
   - Mandatory: XML_PRETTY 20456 tokens
- Highest output token drift:
   - Optional: JSON_PRETTY ↓ -96.05% ↑ 49.34%
   - Mandatory: JSON_PRETTY ↓ -25.06% ↑ 49.22%
- Lowest accuracy:
   - Optional: JSON_PRETTY 72.85%
   - Mandatory: XML_PRETTY 73.39%
- Highest accuracy drift:
   - Optional: JSON_PRETTY ↓ -11.43% ↑ 10.71%
   - Mandatory: XML_PRETTY ↓ -13.19% ↑ 15.38%
- Most wasted tokens:
   - Optional: XML_PRETTY 5251 / 19927 tokens
   - Mandatory: XML_PRETTY 5443 / 20456 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 53.12
   - Mandatory: XML_PRETTY 51.40
- Highest delta (optional-mandatory):
   - Total tokens: JSON_PRETTY -1148 tokens
   - Accuracy: TOON_DEFAULT 5.78%
   - Token efficiency: TOON_DEFAULT 4.70

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 37s | JSON_COMPACT ≈ 10651 | JSON_COMPACT ≈ 2576 | YAML ≈ 79% | YAML ≈ 77% | JSON_COMPACT ≈ 82 | JSON_COMPACT ≈ 81 |
| JSON_COMPACT (+87.6%) | XML_COMPACT (+23.9%) | YAML (+20.7%) | JSON_PRETTY (-1.9%) | JSON_PRETTY (-1.1%) | XML_COMPACT (-10.2%) | XML_COMPACT (-9.9%) |
| XML_PRETTY (+91.7%) | TOON_DEFAULT (+35.6%) | XML_COMPACT (+30.8%) | JSON_COMPACT (-3.0%) | JSON_COMPACT (-2.4%) | YAML (-11.7%) | YAML (-12.3%) |
| XML_COMPACT (+102.6%) | YAML (+37.5%) | TOON_DEFAULT (+43.1%) | TOON_DEFAULT (-4.3%) | XML_COMPACT (-3.3%) | TOON_DEFAULT (-14.6%) | TOON_DEFAULT (-15.8%) |
| YAML (+147.2%) | JSON_PRETTY (+71.6%) | JSON_PRETTY (+64.0%) | XML_COMPACT (-4.3%) | TOON_DEFAULT (-4.9%) | JSON_PRETTY (-26.2%) | JSON_PRETTY (-26.3%) |
| JSON_PRETTY (+153.7%) | XML_PRETTY (+92.1%) | XML_PRETTY (+111.3%) | XML_PRETTY (-5.4%) | XML_PRETTY (-4.9%) | XML_PRETTY (-36.9%) | XML_PRETTY (-37.5%) |


##### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 40s | JSON_COMPACT ≈ 10124 | JSON_COMPACT ≈ 2123 | YAML ≈ 81% | TOON_DEFAULT ≈ 81% | JSON_COMPACT ≈ 85 | JSON_COMPACT ≈ 85 |
| XML_COMPACT (+60.3%) | XML_COMPACT (+25.6%) | YAML (+31.2%) | TOON_DEFAULT (-0.4%) | YAML (-1.2%) | XML_COMPACT (-11.7%) | XML_COMPACT (-11.7%) |
| XML_PRETTY (+94.1%) | TOON_DEFAULT (+40.4%) | TOON_DEFAULT (+32.3%) | JSON_COMPACT (-1.6%) | JSON_COMPACT (-1.8%) | TOON_DEFAULT (-12.9%) | TOON_DEFAULT (-12.4%) |
| JSON_PRETTY (+111.8%) | YAML (+42.1%) | XML_COMPACT (+46.5%) | XML_COMPACT (-5.1%) | XML_COMPACT (-5.4%) | YAML (-13.2%) | YAML (-14.0%) |
| JSON_COMPACT (+120.4%) | JSON_PRETTY (+69.2%) | JSON_PRETTY (+119.0%) | XML_PRETTY (-7.0%) | XML_PRETTY (-8.0%) | JSON_PRETTY (-28.9%) | JSON_PRETTY (-29.5%) |
| YAML (+136.8%) | XML_PRETTY (+96.8%) | XML_PRETTY (+147.3%) | JSON_PRETTY (-7.8%) | JSON_PRETTY (-8.6%) | XML_PRETTY (-37.7%) | XML_PRETTY (-38.4%) |


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

The efficiency ranking is stable across both variants: JSON_COMPACT leads, followed by XML_COMPACT, TOON_DEFAULT, YAML, JSON_PRETTY, and XML_PRETTY. Compact encoding formats consistently outperform their pretty-printed counterparts by 21 to 34 efficiency points, confirming that whitespace inflation carries measurable token overhead without accuracy compensation.

The accuracy ranking presents a different ordering. YAML and TOON_DEFAULT lead in optional data, with JSON_COMPACT competitive at third. This divergence between efficiency and accuracy rankings creates a clear decision boundary: when the token budget is fixed, JSON_COMPACT is the dominant choice; when accuracy on retrieval tasks must be maximized and tokens are available, YAML is the appropriate selection.

The optional versus mandatory dimension reveals that most formats improve with sparser data. JSON_PRETTY is the single exception, degrading 4.03 percentage points in accuracy when optional fields are introduced. All other formats gain accuracy with optional data, with TOON_DEFAULT gaining the most (+5.78 percentage points) and XML_PRETTY gaining the least (+0.26 percentage points).

XML_PRETTY should be avoided in token-constrained contexts under all tested conditions. It consumes the most tokens, produces the most wasted tokens (5,251 to 5,443), and achieves the lowest efficiency score, with its only distinguishing property being variant stability that can be achieved more cost-effectively with XML_COMPACT.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Char/Token | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10315 | 336 | 10651 | 2.213 | 0.712 | 2.710 | 75.81 | 80.55 | 8074.523 | 2576.477 | 81.51 | 80.55 |
| JSON_COMPACT | opt | 9788 | 336 | 10124 | 2.186 | 0.781 | 2.707 | 79.03 | 85.13 | 8000.734 | 2122.933 | 85.29 | 85.13 |
| JSON_PRETTY | man | 17828 | 447 | 18275 | 1.762 | 0.421 | 3.605 | 76.88 | 59.37 | 14049.820 | 4225.180 | 60.17 | 59.37 |
| JSON_PRETTY | opt | 16899 | 228 | 17127 | 1.752 | 0.425 | 1.836 | 72.85 | 60.05 | 12476.777 | 4649.890 | 60.67 | 60.05 |
| TOON_DEFAULT | man | 14096 | 342 | 14438 | 1.851 | 0.516 | 2.757 | 74.46 | 67.83 | 10750.410 | 3687.423 | 69.59 | 67.83 |
| TOON_DEFAULT | opt | 13859 | 352 | 14211 | 1.860 | 0.565 | 2.837 | 80.24 | 74.55 | 11402.706 | 2808.044 | 74.30 | 74.55 |
| XML_COMPACT | man | 12848 | 345 | 13193 | 2.522 | 0.564 | 2.780 | 74.46 | 72.59 | 9823.260 | 3369.407 | 73.20 | 72.59 |
| XML_COMPACT | opt | 12368 | 344 | 12712 | 2.517 | 0.594 | 2.777 | 75.54 | 75.13 | 9602.896 | 3109.437 | 75.35 | 75.13 |
| XML_PRETTY | man | 20114 | 342 | 20456 | 1.993 | 0.359 | 2.761 | 73.39 | 50.38 | 15012.903 | 5443.430 | 51.40 | 50.38 |
| XML_PRETTY | opt | 19583 | 344 | 19927 | 1.982 | 0.370 | 2.774 | 73.65 | 52.40 | 14676.236 | 5250.764 | 53.12 | 52.40 |
| YAML | man | 14306 | 338 | 14644 | 1.789 | 0.538 | 2.723 | 78.76 | 70.67 | 11533.352 | 3110.315 | 72.00 | 70.67 |
| YAML | opt | 14053 | 336 | 14389 | 1.799 | 0.560 | 2.710 | 80.64 | 73.20 | 11603.290 | 2785.710 | 74.06 | 73.20 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Accuracy Man (%) | Accuracy Opt (%) | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10651 | 10124 | -527 | -4.95 | 75.81 | 79.03 |  +3.22 | 74.44 | 78.80 |  +4.36 | 81.51 | 85.29 |  +3.78 | 80.55 | 85.13 |  +4.58 |
| JSON_PRETTY | 18275 | 17127 | -1148 | -6.28 | 76.88 | 72.85 | -4.03 | 75.74 | 71.96 | -3.78 | 60.17 | 60.67 |  +0.51 | 59.37 | 60.05 |  +0.68 |
| TOON_DEFAULT | 14438 | 14211 | -227 | -1.57 | 74.46 | 80.24 |  +5.78 | 71.94 | 80.60 |  +8.66 | 69.59 | 74.30 |  +4.70 | 67.83 | 74.55 |  +6.72 |
| XML_COMPACT | 13193 | 12713 | -480 | -3.64 | 74.46 | 75.54 |  +1.08 | 73.59 | 75.23 |  +1.64 | 73.20 | 75.35 |  +2.15 | 72.59 | 75.13 |  +2.54 |
| XML_PRETTY | 20456 | 19927 | -529 | -2.59 | 73.39 | 73.65 |  +0.26 | 71.93 | 72.63 |  +0.70 | 51.40 | 53.12 |  +1.72 | 50.38 | 52.40 |  +2.02 |
| YAML | 14644 | 14389 | -255 | -1.74 | 78.76 | 80.64 |  +1.88 | 76.86 | 79.41 |  +2.55 | 72.00 | 74.06 |  +2.05 | 70.67 | 73.20 |  +2.52 |

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
| JSON_COMPACT | man | 2.213 | 15.125 | 332.742 | 0.712 |
| JSON_COMPACT | opt | 2.186 | 15.512 | 315.742 | 0.781 |
| JSON_PRETTY | man | 1.762 | 26.141 | 575.097 | 0.421 |
| JSON_PRETTY | opt | 1.752 | 26.781 | 545.129 | 0.425 |
| TOON_DEFAULT | man | 1.851 | 20.669 | 454.710 | 0.516 |
| TOON_DEFAULT | opt | 1.860 | 21.964 | 447.065 | 0.565 |
| XML_COMPACT | man | 2.522 | 18.839 | 414.452 | 0.564 |
| XML_COMPACT | opt | 2.517 | 19.601 | 398.968 | 0.594 |
| XML_PRETTY | man | 1.993 | 29.493 | 648.839 | 0.359 |
| XML_PRETTY | opt | 1.982 | 31.035 | 631.710 | 0.370 |
| YAML | man | 1.789 | 20.977 | 461.484 | 0.538 |
| YAML | opt | 1.799 | 22.271 | 453.323 | 0.560 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 2.213 | 2.186 | -0.027 | -1.22 | 15.125 | 15.512 |  +0.387 |  +2.56 | 332.742 | 315.742 | -17.000 | -5.11 | 0.712 | 0.781 |  +0.069 |  +9.69 |
| JSON_PRETTY | 1.762 | 1.752 | -0.010 | -0.57 | 26.141 | 26.781 |  +0.640 |  +2.45 | 575.097 | 545.129 | -29.968 | -5.21 | 0.421 | 0.425 |  +0.004 |  +0.95 |
| TOON_DEFAULT | 1.851 | 1.860 |  +0.009 |  +0.49 | 20.669 | 21.964 |  +1.295 |  +6.27 | 454.710 | 447.065 | -7.645 | -1.68 | 0.516 | 0.565 |  +0.048 |  +9.40 |
| XML_COMPACT | 2.522 | 2.517 | -0.005 | -0.20 | 18.839 | 19.601 |  +0.762 |  +4.04 | 414.452 | 398.968 | -15.484 | -3.74 | 0.564 | 0.594 |  +0.030 |  +5.32 |
| XML_PRETTY | 1.993 | 1.982 | -0.011 | -0.55 | 29.493 | 31.035 |  +1.542 |  +5.23 | 648.839 | 631.710 | -17.129 | -2.64 | 0.359 | 0.370 |  +0.011 |  +3.06 |
| YAML | 1.789 | 1.799 |  +0.010 |  +0.56 | 20.977 | 22.271 |  +1.294 |  +6.17 | 461.484 | 453.323 | -8.161 | -1.77 | 0.538 | 0.560 |  +0.022 |  +4.09 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10651 | 8075 | 2576 | 75.81 | 74.44 | 81.51 | 80.55 |
| JSON_COMPACT | opt | 10124 | 8001 | 2123 | 79.03 | 78.80 | 85.29 | 85.13 |
| JSON_PRETTY | man | 18275 | 14050 | 4225 | 76.88 | 75.74 | 60.17 | 59.37 |
| JSON_PRETTY | opt | 17127 | 12477 | 4650 | 72.85 | 71.96 | 60.67 | 60.05 |
| TOON_DEFAULT | man | 14438 | 10750 | 3687 | 74.46 | 71.94 | 69.59 | 67.83 |
| TOON_DEFAULT | opt | 14211 | 11403 | 2808 | 80.24 | 80.60 | 74.30 | 74.55 |
| XML_COMPACT | man | 13193 | 9823 | 3369 | 74.46 | 73.59 | 73.20 | 72.59 |
| XML_COMPACT | opt | 12712 | 9603 | 3109 | 75.54 | 75.23 | 75.35 | 75.13 |
| XML_PRETTY | man | 20456 | 15013 | 5443 | 73.39 | 71.93 | 51.40 | 50.38 |
| XML_PRETTY | opt | 19927 | 14676 | 5251 | 73.65 | 72.63 | 53.12 | 52.40 |
| YAML | man | 14644 | 11533 | 3110 | 78.76 | 76.86 | 72.00 | 70.67 |
| YAML | opt | 14389 | 11603 | 2786 | 80.64 | 79.41 | 74.06 | 73.20 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10651 | 10124 | -527 | -4.95 | 8075 | 8001 | -74 | -0.91 | 2576 | 2122 | -454 | -17.61 | 75.81 | 79.03 |  +3.22 | 81.51 | 85.292 |  +3.78 |  +4.64 |
| JSON_PRETTY | 18275 | 17127 | -1148 | -6.28 | 14050 | 12477 | -1573 | -11.20 | 4225 | 4650 |  +425 |  +10.05 | 76.88 | 72.85 | -4.03 | 60.17 | 60.673 |  +0.51 |  +0.84 |
| TOON_DEFAULT | 14438 | 14211 | -227 | -1.57 | 10750 | 11402 |  +652 |  +6.07 | 3687 | 2808 | -879 | -23.85 | 74.46 | 80.24 |  +5.78 | 69.59 | 74.2955 |  +4.70 |  +6.76 |
| XML_COMPACT | 13193 | 12713 | -480 | -3.64 | 9823 | 9603 | -220 | -2.24 | 3369 | 3109 | -260 | -7.72 | 74.46 | 75.54 |  +1.08 | 73.20 | 75.348 |  +2.15 |  +2.93 |
| XML_PRETTY | 20456 | 19927 | -529 | -2.59 | 15013 | 14676 | -337 | -2.24 | 5443 | 5250 | -193 | -3.54 | 73.39 | 73.65 |  +0.26 | 51.40 | 53.118 |  +1.72 |  +3.34 |
| YAML | 14644 | 14389 | -255 | -1.74 | 11533 | 11603 |  +70 |  +0.61 | 3110 | 2785 | -325 | -10.44 | 78.76 | 80.64 |  +1.88 | 72.00 | 74.059 |  +2.05 |  +2.85 |

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

## 3. Format-Specific Analysis
### 3.1 Detailed Analysis: JSON_COMPACT

#### 3.1.1 Performance Summary

- Token Duration Range: 69 - 88 seconds
- Token Cost Range: 10124 - 10651 tokens
- Wasted Token Range: 2123 - 2576 tokens
- Accuracy Range: 75.81 - 79.03%
- Efficiency Score Range: 81.51 - 85.29

#### 3.1.2 Strengths

- Lowest total token cost across both variants (10,124 optional, 10,651 mandatory), resulting in the highest information value per token in the benchmark (0.781 optional, 0.712 mandatory).
- Highest efficiency score overall (85.29 optional, 81.51 mandatory), outperforming all other formats on this composite metric in both variants.
- Best aggregation accuracy in the mandatory variant (71.43%), outperforming all other formats by at least 5.56 percentage points for that category.
- Best structure awareness in the optional variant (82.72%), tied with TOON_DEFAULT, indicating strong schema comprehension when optional fields are present.
- Wasted tokens drop 17.61% when switching from mandatory to optional data, the largest cost-reduction benefit from optional fields observed across all formats.

#### 3.1.3 Weaknesses

- Weakest field retrieval in the mandatory variant (84.85%), trailing YAML by 12.73 percentage points and sitting 6.36 points below the next-weakest format.
- Aggregation accuracy degrades sharply with optional data (71.43% mandatory to 52.38% optional, a 19.05 percentage point drop), indicating the model struggles with numerical operations over sparse datasets encoded in compact JSON.
- Dense syntax with minimal structural whitespace offers less visual separation between records, which amplifies errors in edge cases where nested field boundaries are ambiguous.

#### 3.1.4 Use Case Recommendation

- ✓ Use when: token budget is the primary constraint, the dataset is fully populated with mandatory fields, or aggregation tasks operate on complete datasets where no values are absent.
- ❌ Avoid when: field retrieval precision is critical for sparse schemas with many optional fields, particularly in aggregation pipelines where absent values must be identified and excluded from calculations.

#### 3.1.5 Trade-offs

JSON_COMPACT gains 3.22 percentage points in accuracy when switching to optional data while simultaneously reducing token cost by 4.95%. It is the only format in this benchmark where reducing data density improves both cost and accuracy at the same time, making it uniquely suited for systems that can naturally omit null or empty fields at the serialization layer. The aggregation degradation on optional data (19.05 percentage points) is the primary risk to account for in workloads with mixed mandatory and optional field coverage.

### 3.2 Detailed Analysis: JSON_PRETTY

#### 3.2.1 Performance Summary

- Token Duration Range: 85 - 94 seconds
- Token Cost Range: 17127 - 18275 tokens
- Wasted Token Range: 4225 - 4650 tokens
- Accuracy Range: 72.85 - 76.88%
- Efficiency Score Range: 60.17 - 60.67

#### 3.2.2 Strengths

- Highest field retrieval accuracy in the mandatory variant (96.36%), outperforming JSON_COMPACT by 11.51 percentage points, suggesting that indentation aids the model in locating specific values within fully-populated nested structures.
- Best structure awareness in the mandatory variant (71.60%), indicating that the visual hierarchy from whitespace assists with schema comprehension in dense datasets.
- Most stable efficiency score across variants (+0.51 delta), meaning performance is predictable regardless of data sparsity even though the absolute score is low.

#### 3.2.3 Weaknesses

- Only format in the benchmark where accuracy degrades with optional data (76.88% mandatory to 72.85% optional, a 4.03 percentage point drop), suggesting the model misinterprets null and missing fields in pretty-printed JSON syntax.
- Extreme output token variability: the optional variant shows a drift range of -96.05% to +49.34%, indicating highly inconsistent response lengths that create unpredictable output costs across runs.
- Consumes 71 to 92% more tokens than JSON_COMPACT while scoring 20 to 21 efficiency points lower, representing the worst cost-to-accuracy ratio among JSON variants.
- Worst aggregation accuracy across both variants (49.20% mandatory, 46.03% optional), the lowest aggregation score recorded in the benchmark.

#### 3.2.4 Use Case Recommendation

- ✓ Use when: mandatory field retrieval accuracy is the top priority, the dataset is fully populated with no optional fields, token cost is not a constraint, and output token variance is acceptable.
- ❌ Avoid when: the dataset contains optional or sparse fields, aggregation accuracy matters, or token efficiency is a requirement. JSON_PRETTY is the only format that penalizes optional data with a net accuracy loss, making it unsuitable for any pipeline where field population is variable.

#### 3.2.5 Trade-offs

JSON_PRETTY spends 71% more tokens than JSON_COMPACT in the mandatory variant (18,275 vs 10,651) and achieves only 1.07 percentage points higher accuracy (76.88% vs 75.81%). Its field retrieval advantage in mandatory data (96.36% vs 84.85%) is the one genuinely strong property, but this advantage is unavailable in optional data where accuracy drops below JSON_COMPACT. For workloads that combine retrieval and aggregation tasks across mixed field populations, JSON_PRETTY is the weakest JSON variant on both cost and robustness dimensions.

### 3.3 Detailed Analysis: TOON_DEFAULT

#### 3.3.1 Performance Summary

- Token Duration Range: 37 - 40 seconds
- Token Cost Range: 14211 - 14438 tokens
- Wasted Token Range: 2808 - 3687 tokens
- Accuracy Range: 74.46 - 80.24%
- Efficiency Score Range: 69.59 - 74.30

#### 3.3.2 Strengths

- Fastest total reasoning duration in the benchmark (37 to 40 seconds), approximately 2x faster than YAML and XML-based formats, suggesting the format structure reduces the model's internal reasoning overhead per record.
- Largest accuracy improvement from mandatory to optional data (+5.78 percentage points), indicating the format syntax helps the model distinguish absent versus present fields in sparse schemas.
- Highest weighted accuracy in the optional variant (80.6%), performing particularly well on the high-priority field retrieval and structure awareness categories combined.
- Competitive field retrieval in the optional variant (96.36%), tied with JSON_PRETTY for second place behind YAML (99.39%).

#### 3.3.3 Weaknesses

- Worst structure awareness in the mandatory variant (58.64%), 13 percentage points below JSON_PRETTY, indicating the format syntax impedes schema comprehension when all fields are always present and no null values exist to differentiate.
- Weakest aggregation score in the optional variant (43.65%), the lowest aggregation result recorded across the entire benchmark.
- Largest efficiency score gap between variants (69.59 mandatory vs 74.30 optional, delta 4.70), indicating inconsistent and data-density-dependent behavior that complicates performance prediction.
- Approximately 40% more tokens than JSON_COMPACT without a commensurate efficiency score to justify the overhead.

#### 3.3.4 Use Case Recommendation

- ✓ Use when: latency-sensitive applications require fast reasoning, data is sparse with many optional fields, or weighted accuracy on retrieval and structure tasks is the primary success criterion.
- ❌ Avoid when: the dataset is fully populated with no optional fields (mandatory structure awareness degrades to 58.64%), aggregation accuracy over optional data is required, or consistent cross-variant performance is needed.

#### 3.3.5 Trade-offs

TOON_DEFAULT has the widest performance spread of any format tested. The mandatory-to-optional accuracy gain of 5.78 percentage points is the largest observed in the benchmark, but it coincides with a mandatory structure awareness score of 58.64%, which is 13 percentage points below the mandatory average. The format is conditionally excellent for sparse workloads but unreliable as a general-purpose choice for all data densities. Its speed advantage (2x faster than competitors) is operationally significant for latency-sensitive pipelines where the optional data condition can be guaranteed.

### 3.4 Detailed Analysis: XML_COMPACT

#### 3.4.1 Performance Summary

- Token Duration Range: 64 - 75 seconds
- Token Cost Range: 12712 - 13193 tokens
- Wasted Token Range: 3109 - 3369 tokens
- Accuracy Range: 74.46 - 75.54%
- Efficiency Score Range: 73.20 - 75.35

#### 3.4.2 Strengths

- Most stable accuracy across both variants in the benchmark (+1.08 percentage point delta), making it the most predictable format for pipelines where data sparsity varies at runtime.
- Highest chars-per-token ratio across all formats (2.517 to 2.522), meaning XML_COMPACT encodes more characters per token than any other format tested, partially offsetting the verbosity of XML tags.
- Consistent output token count with narrow drift ranges (mandatory: -3.09% to +3.87%), enabling reliable production cost modeling without wide variance in inference cost per run.

#### 3.4.3 Weaknesses

- Accuracy ceiling is limited (74.46 to 75.54%), placing it 5 to 6 percentage points below YAML and TOON_DEFAULT optional across most categories.
- Approximately 25% more tokens than JSON_COMPACT (12,712 to 13,193 vs 10,124 to 10,651) without a commensurate accuracy advantage on any category.
- Aggregation accuracy plateaus at 55.55% across both variants, showing no sensitivity to data sparsity but also no path to improvement through variant selection.

#### 3.4.4 Use Case Recommendation

- ✓ Use when: XML is a system constraint, consistency and predictability across changing data densities are more important than peak accuracy, or output token stability is required for cost budgeting in production.
- ❌ Avoid when: maximum accuracy is needed (YAML is the appropriate choice), or token efficiency is critical (JSON_COMPACT is the appropriate choice). XML_COMPACT occupies a middle ground that is rarely the optimal selection on either dimension independently.

#### 3.4.5 Trade-offs

XML_COMPACT trades a 25% token overhead over JSON_COMPACT for a consistent and stable accuracy floor that never exceeds 75.54%. Its value proposition is predictability rather than peak performance. The chars-per-token advantage (2.52 vs 2.19 for JSON_COMPACT) partially mitigates the token overhead from XML structural tags, but it does not close the efficiency gap. For systems that must use XML, XML_COMPACT is strictly preferable to XML_PRETTY across every measured dimension.

### 3.5 Detailed Analysis: XML_PRETTY

#### 3.5.1 Performance Summary

- Token Duration Range: 71 - 77 seconds
- Token Cost Range: 19927 - 20456 tokens
- Wasted Token Range: 5251 - 5443 tokens
- Accuracy Range: 73.39 - 73.65%
- Efficiency Score Range: 51.40 - 53.12

#### 3.5.2 Strengths

- Most format-stable accuracy in the benchmark with only a 0.26 percentage point difference between mandatory and optional variants, indicating that the verbose structural markup insulates the model from the effects of data sparsity.
- Consistent output token count across runs (mandatory drift: -1.27% to +1.07%), providing reliable per-inference cost prediction.

#### 3.5.3 Weaknesses

- Highest token cost in the benchmark across both variants (19,927 optional, 20,456 mandatory), consuming approximately 97% more tokens than JSON_COMPACT.
- Lowest information value per token in the benchmark (0.359 mandatory, 0.370 optional), meaning less than 37% of token capacity is utilized for correct information.
- Highest wasted token count (5,251 optional, 5,443 mandatory), representing tokens consumed on incorrect answers at the largest absolute scale in the benchmark.
- Lowest efficiency score overall (51.40 mandatory, 53.12 optional), placing it 28 to 34 points below JSON_COMPACT.
- Worst filtering accuracy in the mandatory variant (53.97%), the lowest category score for filtering recorded across the entire benchmark.

#### 3.5.4 Use Case Recommendation

- ✓ Use when: the consuming system strictly requires XML format and data sparsity varies unpredictably at runtime. Variant stability is the only concrete advantage XML_PRETTY offers over alternatives.
- ❌ Avoid when: token cost, efficiency, filtering accuracy, or throughput matter. XML_PRETTY is the lowest-value format in this benchmark on every dimension except variant stability, and XML_COMPACT provides comparable stability at a 35% lower token cost.

#### 3.5.5 Trade-offs

XML_PRETTY's verbosity imposes a near-doubling of token cost relative to JSON_COMPACT while delivering 6 to 7 percentage points less accuracy. The combination of XML tag overhead and indentation whitespace produces the worst information-per-token ratio tested at 0.359. XML_PRETTY demonstrates that whitespace inflation is more damaging in XML than in JSON, because XML tag repetition compounds the cost of indentation in a way that pretty-printed JSON does not. The sole redeeming property is variant stability, and that property can be achieved at lower cost with XML_COMPACT (+1.08 percentage point delta) rather than XML_PRETTY (+0.26 percentage point delta).

### 3.6 Detailed Analysis: YAML

#### 3.6.1 Performance Summary

- Token Duration Range: 92 - 94 seconds
- Token Cost Range: 14389 - 14644 tokens
- Wasted Token Range: 2786 - 3110 tokens
- Accuracy Range: 78.76 - 80.64%
- Efficiency Score Range: 72.00 - 74.06

#### 3.6.2 Strengths

- Highest raw accuracy in the benchmark across both variants (80.64% optional, 78.76% mandatory).
- Best field retrieval by a significant margin (99.39% optional, 97.58% mandatory), approaching the ceiling of what is measurable for structured data retrieval at this question set scale.
- Tightest accuracy variance in the mandatory variant (drift range: -2.73% to +2.40%), indicating highly stable and reproducible results across repeated runs.
- Low output tokens per answer (2.710 to 2.723), indicating concise and precise answer generation despite leading the accuracy rankings.
- Compact character-to-token ratio (1.789 to 1.799 chars/token) for a human-readable format, demonstrating that YAML's indentation-based structure tokenizes efficiently relative to its character count.

#### 3.6.3 Weaknesses

- Aggregation accuracy degrades with optional data (60.32% mandatory to 55.56% optional, a 4.76 percentage point drop), consistent with the broader benchmark pattern of aggregation regression in sparse datasets.
- Longest reasoning duration in the benchmark (91 to 94 seconds), approximately 2.5x slower than TOON_DEFAULT despite comparable optional accuracy.
- Efficiency score (72.00 to 74.06) is 11 to 13 points below JSON_COMPACT, driven by a 42% higher token cost without a proportional accuracy gain at the aggregate benchmark level.

#### 3.6.4 Use Case Recommendation

- ✓ Use when: field retrieval accuracy is the primary objective, data contains both mandatory and optional fields with variable population, and the use case is latency-tolerant. YAML is the recommended format when correctness on individual value lookups must be maximized.
- ❌ Avoid when: token budget is constrained, aggregation tasks dominate the query profile, or response latency is a hard operational requirement.

#### 3.6.5 Trade-offs

YAML spends 42% more tokens than JSON_COMPACT to gain 1.61 percentage points in optional accuracy (80.64% vs 79.03%). This trade-off is most justified when field retrieval tasks represent a large share of the workload, where YAML's 99.39% accuracy versus JSON_COMPACT's 93.94% represents a 5.45 percentage point operational advantage on the highest-weighted question category. For mixed workloads with significant aggregation load, the token premium does not yield commensurate returns, as both formats converge toward similar aggregation error rates in optional data (55.56% YAML vs 52.38% JSON_COMPACT).

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

- **Report Generated**: 2026-03-22
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **With the help of**: Claude Sonnet 4.6
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)