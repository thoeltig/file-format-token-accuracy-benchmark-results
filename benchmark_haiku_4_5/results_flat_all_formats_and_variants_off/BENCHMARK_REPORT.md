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

1. JSON_PRETTY achieves the highest raw accuracy at 82.53% for mandatory data but consumes 14,613 tokens per query. No other format exceeds 80% accuracy in the mandatory variant establishing a clear accuracy ceiling for this model and structure configuration.

2. TOON_DEFAULT delivers the best weighted efficiency score for mandatory data (82.13) at only 7,556 tokens but its token count surges 57% to 11,900 for optional data. This structural sensitivity makes it unreliable for datasets with sparse or optional fields.

3. CSV is the cheapest format at 7,032 tokens (optional) but consistently produces the lowest accuracy across both variants (65.48% and 67.58%). The aggregation category drops to 36.19% for optional data which is the worst score across all tested configurations.

4. Aggregation is the weakest question category across all formats with peak performance of only 58.73% (JSON_PRETTY mandatory). This indicates a model-level reasoning constraint that format selection alone cannot overcome.

5. XML_COMPACT and YAML are the only formats that improve accuracy from mandatory to optional variants (+5.11% and +4.57% respectively). Both show significant structure awareness gains in the optional variant.

6. JSON_COMPACT provides the most stable efficiency score across both data variants (76.46 mandatory, 77.14 optional). This predictability makes it the most reliable choice for production pipelines operating on datasets with mixed field schemas.

7. XML_PRETTY is the weakest format overall, combining the highest token cost (16,490 mandatory) with the lowest efficiency scores (53.66 mandatory, 57.24 optional). It provides no accuracy advantage over JSON_PRETTY while consuming 13% more tokens.

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
   - Optional: CSV 7032 tokens
   - Mandatory: CSV 7263 tokens
- Lowest output token cost drift:
   - Optional: XML_COMPACT ↓ -0.32% ↑ 0.32%
   - Mandatory: JSON_PRETTY ↓ -1.22% ↑ 0.77%
- Highest accuracy:
   - Optional: YAML 80.38%
   - Mandatory: JSON_PRETTY 82.53%
- Lowest accuracy drift:
   - Optional: YAML ↓ -3.68% ↑ 3.33%
   - Mandatory: JSON_PRETTY ↓ -1.31% ↑ 1.62%
- Most useful tokens:
   - Optional: XML_PRETTY 11854 / 15419 tokens
   - Mandatory: XML_PRETTY 12633 / 16490 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 77.14
   - Mandatory: TOON_DEFAULT 82.13
- Lowest delta (optional-mandatory):
   - Total tokens: CSV -231 tokens
   - Accuracy: XML_PRETTY 0.27%
   - Token efficiency: JSON_PRETTY 0.45

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 15419 tokens
   - Mandatory: XML_PRETTY 16490 tokens
- Highest output token drift:
   - Optional: JSON_PRETTY ↓ -97.84% ↑ 34.20%
   - Mandatory: CSV ↓ -96.26% ↑ 25.10%
- Lowest accuracy:
   - Optional: CSV 65.48%
   - Mandatory: CSV 67.58%
- Highest accuracy drift:
   - Optional: CSV ↓ -16.25% ↑ 13.30%
   - Mandatory: TOON_DEFAULT ↓ -22.37% ↑ 8.04%
- Most wasted tokens:
   - Optional: XML_PRETTY 3565 / 15419 tokens
   - Mandatory: XML_PRETTY 3857 / 16490 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 57.24
   - Mandatory: XML_PRETTY 53.66
- Highest delta (optional-mandatory):
   - Total tokens: TOON_DEFAULT 4344 tokens
   - Accuracy: XML_COMPACT 5.11%
   - Token efficiency: TOON_DEFAULT -14.43

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 50s | CSV ≈ 7263 | TOON_DEFAULT ≈ 1747 | JSON_PRETTY ≈ 83% | JSON_PRETTY ≈ 82% | TOON_DEFAULT ≈ 82 | TOON_DEFAULT ≈ 82 |
| YAML (+50.6%) | TOON_DEFAULT (+4.0%) | JSON_COMPACT (+20.7%) | JSON_COMPACT (-4.5%) | JSON_COMPACT (-4.4%) | CSV (-6.8%) | CSV (-6.8%) |
| XML_COMPACT (+59.3%) | JSON_COMPACT (+32.2%) | CSV (+34.8%) | TOON_DEFAULT (-5.7%) | TOON_DEFAULT (-5.5%) | JSON_COMPACT (-6.9%) | JSON_COMPACT (-7.0%) |
| CSV (+60.6%) | XML_COMPACT (+68.7%) | JSON_PRETTY (+46.1%) | XML_PRETTY (-5.9%) | XML_PRETTY (-6.5%) | XML_COMPACT (-21.1%) | XML_COMPACT (-22.1%) |
| XML_PRETTY (+71.6%) | YAML (+77.3%) | YAML (+78.3%) | YAML (-6.7%) | YAML (-8.0%) | YAML (-21.4%) | JSON_PRETTY (-22.6%) |
| JSON_PRETTY (+91.0%) | JSON_PRETTY (+101.2%) | XML_COMPACT (+86.6%) | XML_COMPACT (-9.1%) | XML_COMPACT (-10.0%) | JSON_PRETTY (-22.4%) | YAML (-22.7%) |
| JSON_COMPACT (+111.6%) | XML_PRETTY (+127.1%) | XML_PRETTY (+120.8%) | CSV (-15.0%) | CSV (-14.7%) | XML_PRETTY (-34.7%) | XML_PRETTY (-35.4%) |


##### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 42s | CSV ≈ 7032 | JSON_COMPACT ≈ 2122 | YAML ≈ 80% | YAML ≈ 80% | JSON_COMPACT ≈ 77 | JSON_COMPACT ≈ 76 |
| XML_PRETTY (+76.7%) | JSON_COMPACT (+29.0%) | YAML (+11.8%) | JSON_PRETTY (-1.7%) | XML_COMPACT (-1.4%) | CSV (-1.7%) | CSV (-0.6%) |
| JSON_COMPACT (+79.9%) | XML_COMPACT (+63.4%) | CSV (+14.4%) | XML_COMPACT (-1.9%) | JSON_PRETTY (-1.7%) | XML_COMPACT (-8.2%) | XML_COMPACT (-7.2%) |
| JSON_PRETTY (+102.2%) | TOON_DEFAULT (+69.2%) | XML_COMPACT (+16.4%) | XML_PRETTY (-3.5%) | XML_PRETTY (-3.3%) | YAML (-9.0%) | YAML (-8.4%) |
| CSV (+105.9%) | YAML (+72.0%) | TOON_DEFAULT (+35.1%) | JSON_COMPACT (-3.8%) | JSON_COMPACT (-4.5%) | TOON_DEFAULT (-12.2%) | TOON_DEFAULT (-11.9%) |
| XML_COMPACT (+108.6%) | JSON_PRETTY (+93.8%) | JSON_PRETTY (+36.7%) | TOON_DEFAULT (-4.5%) | TOON_DEFAULT (-4.7%) | JSON_PRETTY (-16.8%) | JSON_PRETTY (-16.3%) |
| YAML (+147.4%) | XML_PRETTY (+119.3%) | XML_PRETTY (+68.0%) | CSV (-14.9%) | CSV (-14.4%) | XML_PRETTY (-25.8%) | XML_PRETTY (-25.3%) |


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

The benchmark reveals three performance tiers. TOON_DEFAULT mandatory and JSON_COMPACT form the top efficiency tier achieving efficiency scores above 76 through different mechanisms: TOON_DEFAULT minimizes token cost while maintaining near-average accuracy and JSON_COMPACT prioritizes field retrieval precision with moderate token overhead. The middle tier contains CSV, XML_COMPACT, YAML, and JSON_PRETTY where the optimal choice depends on whether accuracy or token cost is the binding constraint. XML_PRETTY occupies the bottom tier alone with no configuration in which it outperforms a cheaper alternative on a holistic basis.

The most operationally significant finding is TOON_DEFAULT's token surge under optional data: A 57% increase for an equivalent record count signals that the format's mandatory efficiency is structural rather than fundamental. Formats with stable optional-to-mandatory token ratios such as CSV (-3.18%) and JSON_COMPACT (-5.49%) offer more predictable behavior across schema variations.

Aggregation accuracy peaks at 58.73% and falls as low as 36.19%, confirming that this question category reflects a reasoning constraint at the model level. Format selection has limited leverage over aggregation performance, and systems relying on LLM-based aggregation from context should account for an error rate above 40% regardless of format choice.

For teams optimizing primarily for accuracy JSON_PRETTY mandatory and YAML optional are the recommendation. Teams optimizing for efficiency should use TOON_DEFAULT mandatory which needs about half the tokens while only scoring 5.7% less on accuracy compared to JSON_PRETTY. For efficient optional formats use JSON_COMPACT which scores only 3.8% lower on accuracy and needs 72% less tokens than YAML.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Char/Token | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7022 | 241 | 7263 | 1.438 | 0.931 | 1.940 | 67.58 | 76.40 | 4908.065 | 2354.535 | 76.54 | 76.40 |
| CSV | opt | 6728 | 304 | 7032 | 1.423 | 0.931 | 2.448 | 65.48 | 75.81 | 4604.292 | 2427.308 | 75.80 | 75.81 |
| JSON_COMPACT | man | 9296 | 304 | 9600 | 2.143 | 0.813 | 2.452 | 78.03 | 76.24 | 7490.880 | 2109.120 | 76.46 | 76.24 |
| JSON_COMPACT | opt | 8768 | 305 | 9073 | 2.108 | 0.844 | 2.458 | 76.61 | 76.25 | 6950.672 | 2122.128 | 77.14 | 76.25 |
| JSON_PRETTY | man | 14311 | 302 | 14613 | 1.691 | 0.565 | 2.433 | 82.53 | 63.43 | 12059.834 | 2552.833 | 63.74 | 63.43 |
| JSON_PRETTY | opt | 13395 | 231 | 13626 | 1.677 | 0.578 | 1.863 | 78.71 | 63.80 | 10725.025 | 2900.975 | 64.19 | 63.80 |
| TOON_DEFAULT | man | 7298 | 258 | 7556 | 1.393 | 1.018 | 2.076 | 76.88 | 81.95 | 5808.668 | 1746.832 | 82.13 | 81.95 |
| TOON_DEFAULT | opt | 11590 | 309 | 11899 | 1.694 | 0.638 | 2.493 | 75.90 | 67.20 | 9031.467 | 2867.700 | 67.69 | 67.20 |
| XML_COMPACT | man | 11950 | 303 | 12253 | 2.315 | 0.599 | 2.441 | 73.39 | 63.87 | 8992.232 | 3260.435 | 64.82 | 63.87 |
| XML_COMPACT | opt | 11181 | 309 | 11490 | 2.288 | 0.683 | 2.492 | 78.50 | 70.75 | 9019.650 | 2470.350 | 70.81 | 70.75 |
| XML_PRETTY | man | 16186 | 304 | 16490 | 1.931 | 0.465 | 2.454 | 76.61 | 52.96 | 12633.244 | 3857.089 | 53.66 | 52.96 |
| XML_PRETTY | opt | 15117 | 302 | 15419 | 1.913 | 0.499 | 2.435 | 76.88 | 56.98 | 11854.127 | 3564.873 | 57.24 | 56.98 |
| YAML | man | 12574 | 302 | 12876 | 1.661 | 0.589 | 2.433 | 75.81 | 63.36 | 9761.043 | 3114.624 | 64.54 | 63.36 |
| YAML | opt | 11791 | 304 | 12095 | 1.646 | 0.665 | 2.452 | 80.38 | 69.83 | 9721.961 | 2373.039 | 70.21 | 69.83 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Accuracy Man (%) | Accuracy Opt (%) | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7263 | 7032 | -231 | -3.18 | 67.58 | 65.48 | -2.10 | 67.37 | 65.49 | -1.88 | 76.54 | 75.80 | -0.74 | 76.40 | 75.81 | -0.58 |
| JSON_COMPACT | 9600 | 9073 | -527 | -5.49 | 78.03 | 76.61 | -1.42 | 77.71 | 75.34 | -2.37 | 76.46 | 77.14 |  +0.68 | 76.24 | 76.25 |  +0.01 |
| JSON_PRETTY | 14613 | 13626 | -987 | -6.75 | 82.53 | 78.71 | -3.82 | 82.08 | 78.15 | -3.93 | 63.74 | 64.19 |  +0.45 | 63.43 | 63.80 |  +0.37 |
| TOON_DEFAULT | 7556 | 11900 |  +4344 |  +57.49 | 76.88 | 75.90 | -0.98 | 76.63 | 75.19 | -1.44 | 82.13 | 67.69 | -14.43 | 81.95 | 67.20 | -14.75 |
| XML_COMPACT | 12253 | 11490 | -763 | -6.23 | 73.39 | 78.50 |  +5.11 | 72.03 | 78.42 |  +6.39 | 64.82 | 70.81 |  +5.99 | 63.87 | 70.75 |  +6.89 |
| XML_PRETTY | 16490 | 15419 | -1071 | -6.49 | 76.61 | 76.88 |  +0.27 | 75.61 | 76.51 |  +0.90 | 53.66 | 57.24 |  +3.58 | 52.96 | 56.98 |  +4.02 |
| YAML | 12876 | 12095 | -781 | -6.07 | 75.81 | 80.38 |  +4.57 | 74.13 | 79.84 |  +5.71 | 64.54 | 70.21 |  +5.67 | 63.36 | 69.83 |  +6.47 |

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
| CSV | man | 1.438 | 10.296 | 226.516 | 0.931 |
| CSV | opt | 1.423 | 10.662 | 217.032 | 0.931 |
| JSON_COMPACT | man | 2.143 | 13.630 | 299.871 | 0.813 |
| JSON_COMPACT | opt | 2.108 | 13.895 | 282.839 | 0.844 |
| JSON_PRETTY | man | 1.691 | 20.984 | 461.645 | 0.565 |
| JSON_PRETTY | opt | 1.677 | 21.228 | 432.097 | 0.578 |
| TOON_DEFAULT | man | 1.393 | 10.701 | 235.419 | 1.018 |
| TOON_DEFAULT | opt | 1.694 | 18.368 | 373.871 | 0.638 |
| XML_COMPACT | man | 2.315 | 17.522 | 385.484 | 0.599 |
| XML_COMPACT | opt | 2.288 | 17.719 | 360.677 | 0.683 |
| XML_PRETTY | man | 1.931 | 23.733 | 522.129 | 0.465 |
| XML_PRETTY | opt | 1.913 | 23.957 | 487.645 | 0.499 |
| YAML | man | 1.661 | 18.437 | 405.613 | 0.589 |
| YAML | opt | 1.646 | 18.686 | 380.355 | 0.665 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.438 | 1.423 | -0.015 | -1.04 | 10.296 | 10.662 |  +0.366 |  +3.55 | 226.516 | 217.032 | -9.484 | -4.19 | 0.931 | 0.931 | 0.000 | 0.00 |
| JSON_COMPACT | 2.143 | 2.108 | -0.035 | -1.63 | 13.630 | 13.895 |  +0.265 |  +1.94 | 299.871 | 282.839 | -17.032 | -5.68 | 0.813 | 0.844 |  +0.031 |  +3.81 |
| JSON_PRETTY | 1.691 | 1.677 | -0.014 | -0.83 | 20.984 | 21.228 |  +0.244 |  +1.16 | 461.645 | 432.097 | -29.548 | -6.40 | 0.565 | 0.578 |  +0.013 |  +2.30 |
| TOON_DEFAULT | 1.393 | 1.694 |  +0.301 |  +21.61 | 10.701 | 18.368 |  +7.667 |  +71.65 | 235.419 | 373.871 |  +138.452 |  +58.81 | 1.018 | 0.638 | -0.380 | -37.30 |
| XML_COMPACT | 2.315 | 2.288 | -0.027 | -1.17 | 17.522 | 17.719 |  +0.197 |  +1.12 | 385.484 | 360.677 | -24.807 | -6.44 | 0.599 | 0.683 |  +0.084 |  +14.02 |
| XML_PRETTY | 1.931 | 1.913 | -0.018 | -0.93 | 23.733 | 23.957 |  +0.224 |  +0.94 | 522.129 | 487.645 | -34.484 | -6.60 | 0.465 | 0.499 |  +0.034 |  +7.31 |
| YAML | 1.661 | 1.646 | -0.015 | -0.90 | 18.437 | 18.686 |  +0.249 |  +1.35 | 405.613 | 380.355 | -25.258 | -6.23 | 0.589 | 0.665 |  +0.076 |  +12.90 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| CSV | man | 7263 | 4908 | 2355 | 67.58 | 67.37 | 76.54 | 76.40 |
| CSV | opt | 7032 | 4604 | 2427 | 65.48 | 65.49 | 75.80 | 75.81 |
| JSON_COMPACT | man | 9600 | 7491 | 2109 | 78.03 | 77.71 | 76.46 | 76.24 |
| JSON_COMPACT | opt | 9073 | 6951 | 2122 | 76.61 | 75.34 | 77.14 | 76.25 |
| JSON_PRETTY | man | 14613 | 12060 | 2553 | 82.53 | 82.08 | 63.74 | 63.43 |
| JSON_PRETTY | opt | 13626 | 10725 | 2901 | 78.71 | 78.15 | 64.19 | 63.80 |
| TOON_DEFAULT | man | 7556 | 5809 | 1747 | 76.88 | 76.63 | 82.13 | 81.95 |
| TOON_DEFAULT | opt | 11899 | 9031 | 2868 | 75.90 | 75.19 | 67.69 | 67.20 |
| XML_COMPACT | man | 12253 | 8992 | 3260 | 73.39 | 72.03 | 64.82 | 63.87 |
| XML_COMPACT | opt | 11490 | 9020 | 2470 | 78.50 | 78.42 | 70.81 | 70.75 |
| XML_PRETTY | man | 16490 | 12633 | 3857 | 76.61 | 75.61 | 53.66 | 52.96 |
| XML_PRETTY | opt | 15419 | 11854 | 3565 | 76.88 | 76.51 | 57.24 | 56.98 |
| YAML | man | 12876 | 9761 | 3115 | 75.81 | 74.13 | 64.54 | 63.36 |
| YAML | opt | 12095 | 9722 | 2373 | 80.38 | 79.84 | 70.21 | 69.83 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7263 | 7032 | -231 | -3.18 | 4908 | 4604 | -304 | -6.19 | 2355 | 2428 |  +73 |  +3.09 | 67.58 | 65.48 | -2.10 | 76.54 | 75.804 | -0.74 | -0.97 |
| JSON_COMPACT | 9600 | 9073 | -527 | -5.49 | 7491 | 6951 | -540 | -7.21 | 2109 | 2122 |  +13 |  +0.62 | 78.03 | 76.61 | -1.42 | 76.46 | 77.135 |  +0.68 |  +0.88 |
| JSON_PRETTY | 14613 | 13626 | -987 | -6.75 | 12060 | 10725 | -1335 | -11.07 | 2553 | 2901 |  +348 |  +13.64 | 82.53 | 78.71 | -3.82 | 63.74 | 64.194 |  +0.45 |  +0.70 |
| TOON_DEFAULT | 7556 | 11900 |  +4344 |  +57.49 | 5809 | 9032 |  +3223 |  +55.48 | 1747 | 2868 |  +1121 |  +64.16 | 76.88 | 75.90 | -0.98 | 82.13 | 67.693 | -14.43 | -17.57 |
| XML_COMPACT | 12253 | 11490 | -763 | -6.22 | 8992 | 9019 |  +27 |  +0.30 | 3260 | 2470 | -790 | -24.24 | 73.39 | 78.50 |  +5.11 | 64.82 | 70.808 |  +5.99 |  +9.24 |
| XML_PRETTY | 16490 | 15419 | -1071 | -6.50 | 12633 | 11854 | -779 | -6.17 | 3857 | 3565 | -292 | -7.58 | 76.61 | 76.88 |  +0.27 | 53.66 | 57.238 |  +3.58 |  +6.67 |
| YAML | 12876 | 12095 | -781 | -6.06 | 9761 | 9722 | -39 | -0.40 | 3115 | 2373 | -742 | -23.81 | 75.81 | 80.38 |  +4.57 | 64.54 | 70.209 |  +5.67 |  +8.79 |

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

## 3. Format-Specific Analysis
### 3.1 Detailed Analysis: CSV

#### 3.1.1 Performance Summary

- Token Duration Range: 81 - 87 seconds
- Token Cost Range: 7032 - 7263 tokens
- Wasted Token Range: 2355 - 2427 tokens
- Accuracy Range: 65.48 - 67.58%
- Efficiency Score Range: 75.80 - 76.54

#### 3.1.2 Strengths

- Lowest total token cost of all tested formats (7,032 optional, 7,263 mandatory) preserving maximum available context budget for other inputs.
- Most stable tokenization across variants: chars-per-token changes only 1.04% between mandatory (1.438) and optional (1.423) providing predictable token budgeting.
- Field retrieval accuracy is disproportionately higher than overall accuracy (75.64% mandatory, 79.27% optional) which suggests the row-column layout supports direct value lookup reasonably well.

#### 3.1.3 Weaknesses

- Lowest overall accuracy of all formats in both variants (67.58% mandatory, 65.48% optional) trailing the benchmark average by more than 8 percentage points.
- Aggregation collapses in the optional variant to 36.19% which is the lowest aggregation score across all tested formats and variants. The drop from mandatory (53.33%) is 17.14 percentage points.
- Highest accuracy variance in the optional variant with a run-to-run swing up to 29.55 percentage points which indicats unstable inference behavior across repeated runs.
- Structure awareness degrades further in optional data (62.22%) which is consistent with CSV's inability to signal field presence or absence through its format structure.

#### 3.1.4 Use Case Recommendation

- ✓ Use when token budget is the binding constraint and approximate answers in the 65-68% accuracy range are acceptable.
- ✓ Use for mandatory-only dense datasets to avoid the aggregation degradation that occurs with optional fields.
- ❌ Avoid when structural queries, aggregation or accuracy above 70% is required.
- ❌ Avoid for datasets with optional or nullable fields where the lack of structural markers degrades model comprehension.

#### 3.1.5 Trade-offs

- CSV's token savings over JSON_COMPACT are 2,337 tokens (mandatory) and 2,041 tokens (optional). In both cases JSON_COMPACT achieves over 10 percentage points higher accuracy while actually wasting fewer tokens in absolute terms (2,109 vs 2,355 mandatory). The efficiency argument for CSV only holds when the token budget is too tight for JSON_COMPACT's minimum of approximately 9,073 tokens.
- The information-value-per-token metric (0.931) appears favorable compared to JSON_PRETTY (0.565) but this reflects dense content encoding rather than accurate answers. Token efficiency metrics that incorporate accuracy tell the more relevant story.

### 3.2 Detailed Analysis: JSON_COMPACT

#### 3.2.1 Performance Summary

- Token Duration Range: 76 - 107 seconds
- Token Cost Range: 9073 - 9600 tokens
- Wasted Token Range: 2109 - 2122 tokens
- Accuracy Range: 76.61 - 78.03%
- Efficiency Score Range: 76.46 - 77.14

#### 3.2.2 Strengths

- Highest field retrieval accuracy of all mandatory formats at 98.18% which indicats that compact JSON key-value pairs are highly legible for direct value extraction.
- Most consistent efficiency score across variants (76.46 mandatory, 77.14 optional) which makes it the most predictable format for production systems with variable data schemas.
- Lowest wasted tokens in the mandatory variant (2,109) and near-lowest in optional (2,122) meaning token waste is minimal relative to total consumption.
- Token count decreases predictably from mandatory to optional (-527 tokens, -5.49%) proportional to the actual data reduction.

#### 3.2.3 Weaknesses

- Aggregation accuracy in the mandatory variant is 40.48% which is the lowest aggregation result across all mandatory test cases. Despite leading in field retrieval the format does not support numerical reasoning well.
- Structure awareness degrades 8.15 percentage points from mandatory (77.78%) to optional (69.63%) suggesting that sparse fields reduce the model's ability to infer schema relationships.
- Token cost is approximately 30% higher than CSV and TOON_DEFAULT mandatory which limits applicability when context windows are heavily constrained.

#### 3.2.4 Use Case Recommendation

- ✓ Use when field retrieval precision is the primary workload such as in automated extraction or lookup agents.
- ✓ Use for datasets with mixed mandatory and optional fields where consistent, predictable efficiency is required.
- ❌ Avoid when aggregation tasks dominate the workload as accuracy in that category does not exceed 52% in any variant.

#### 3.2.5 Trade-offs

- Compared to CSV, JSON_COMPACT consumes 32% more tokens but delivers 10 percentage points more accuracy while wasting fewer tokens in absolute terms. The trade-off is favorable unless the context budget is constrained. JSON_COMPACT uses only ~65% of the tokens which JSON_PRETTY needs while reducing accuracy by 4.5 percentage points. Compared to JSON_PRETTY its efficiency score (76.46) is 12 points higher. For most production use cases JSON_COMPACT represents the better value unless maximum raw accuracy is the only requirement.

### 3.3 Detailed Analysis: JSON_PRETTY

#### 3.3.1 Performance Summary

- Token Duration Range: 85 - 96 seconds
- Token Cost Range: 13626 - 14613 tokens
- Wasted Token Range: 2553 - 2901 tokens
- Accuracy Range: 78.71 - 82.53%
- Efficiency Score Range: 63.74 - 64.19

#### 3.3.2 Strengths

- Highest raw accuracy of all tested formats in the mandatory variant at 82.53% with the best performance in structure awareness (83.95%) and aggregation (58.73%).
- Lowest accuracy variance in the mandatory variant with a run-to-run swing of only 2.93 percentage points which makes it the most reproducible format for consistent results.
- Strong performance across all four question categories in mandatory with no single category falling below 58%.

#### 3.3.3 Weaknesses

- Second highest token cost after XML_PRETTY (14,613 mandatory, 13,626 optional) consuming approximately twice the tokens of CSV or TOON_DEFAULT mandatory.
- Lowest information-value-per-token of all formats except XML_PRETTY (0.565 mandatory, 0.578 optional) which reflects the overhead introduced by indentation and whitespace.
- Accuracy drops 3.82 percentage points from mandatory to optional which is the largest absolute drop among mid-tier formats and a meaningful reduction in the accuracy premium over cheaper alternatives.
- Extreme output token drift in the optional variant (minimum drift of -97.84%) signals occasional output instability that may affect downstream parsers relying on consistent response length.

#### 3.3.4 Use Case Recommendation

- ✓ Use when maximum accuracy is the primary requirement and token budget is unconstrained.
- ✓ Use for mandatory-only datasets where the 82.53% accuracy and sub-3-point run-to-run drift provide reliable results.
- ✓ Use when structure awareness queries are a large portion of the workload as JSON_PRETTY leads this category in mandatory at 83.95%.
- ❌ Avoid when context window space is at a premium as it consumes 2x the tokens of TOON_DEFAULT for approximately 6 percentage points more accuracy.
- ❌ Avoid for optional-heavy datasets where the accuracy premium shrinks but the token cost remains elevated.

#### 3.3.5 Trade-offs

- JSON_PRETTY's accuracy premium over JSON_COMPACT is 4.5 percentage points at a cost of 5,013 additional tokens per query (mandatory). Its efficiency score (63.74) is 12 points below JSON_COMPACT (76.46) meaning the composite token-adjusted evaluation does not favor JSON_PRETTY except when raw accuracy is the sole metric. Teams that can tolerate 4-5 percentage points less accuracy recover more than a third more tokens per query by switching to JSON_COMPACT.

### 3.4 Detailed Analysis: TOON_DEFAULT

#### 3.4.1 Performance Summary

- Token Duration Range: 42 - 50 seconds
- Token Cost Range: 7556 - 11899 tokens
- Wasted Token Range: 1747 - 2868 tokens
- Accuracy Range: 75.90 - 76.88%
- Efficiency Score Range: 67.69 - 82.13

#### 3.4.2 Strengths

- Best efficiency score of all formats in the mandatory variant (82.13), achieved through the second-lowest token cost (7,556) paired with 76.88% accuracy.
- Highest information-value-per-token in mandatory (1.018), meaning each token contributes more measurable informational value than in any other tested format in that variant.
- Lowest absolute wasted tokens in the mandatory variant (1,747), outperforming every other format on token waste.
- Fastest inference time in both variants (approximately 50 seconds mandatory, 42 seconds optional), providing a speed advantage for latency-sensitive applications.

#### 3.4.3 Weaknesses

- Token count surges 57.49% from mandatory (7,556) to optional (11,900), an increase of 4,344 tokens. No other format exhibits a remotely comparable optional overhead relative to its mandatory baseline.
- The efficiency score collapses from 82.13 in mandatory to 67.69 in optional, a 14.44-point drop that is the largest single-format degradation in the benchmark.
- Highest run-to-run accuracy drift in the mandatory variant with a max swing of 30.41 percentage points, indicating non-deterministic behavior across repeated runs.
- Filtering (60.32%) and aggregation (49.21%) accuracy in mandatory are among the lowest of all formats in their respective categories.

#### 3.4.4 Use Case Recommendation

- ✓ Use for mandatory-only datasets where token efficiency is the primary optimization target and accuracy in the 76-77% range is sufficient.
- ✓ Use in latency-sensitive pipelines where inference speed is a factor, as it consistently achieves the fastest total duration.
- ❌ Avoid for datasets with optional or sparse fields, where the token count becomes unpredictable and the efficiency advantage disappears entirely.
- ❌ Avoid when run-to-run consistency is critical, given the high accuracy drift observed in the mandatory variant.

#### 3.4.5 Trade-offs

- TOON_DEFAULT's efficiency advantage over JSON_COMPACT in mandatory (82.13 vs 76.46) comes from token savings rather than higher accuracy. JSON_COMPACT is actually more accurate (78.03% vs 76.88%) while costing 2,044 more tokens. The format rewards mandatory-only workloads with the best available token-efficiency ratio, but this advantage reverses entirely with optional data. TOON_DEFAULT mandatory and optional should be treated as functionally different formats with incompatible performance characteristics rather than two variants of the same format.

### 3.5 Detailed Analysis: XML_COMPACT

#### 3.5.1 Performance Summary

- Token Duration Range: 80 - 88 seconds
- Token Cost Range: 11490 - 12253 tokens
- Wasted Token Range: 2470 - 3260 tokens
- Accuracy Range: 73.39 - 78.50%
- Efficiency Score Range: 64.82 - 70.81

#### 3.5.2 Strengths

- One of only two formats to improve accuracy from mandatory to optional (+5.11%), driven by a 13.58-percentage-point improvement in structure awareness for the optional variant.
- Field retrieval in the optional variant reaches 97.58%, the second highest score in that category across all optional tests.
- Consistent tokenization: chars-per-token differs by only 1.17% between mandatory (2.315) and optional (2.288), ensuring predictable token budgeting.
- The optional variant reduces wasted tokens by 24.24% compared to mandatory (2,470 vs 3,260), meaning efficiency improves as data becomes sparser.

#### 3.5.3 Weaknesses

- Mandatory variant accuracy (73.39%) is the second lowest of all formats, 3 percentage points below the benchmark average and below even XML_PRETTY mandatory.
- Highest total token cost among compact-format peers in mandatory (12,253 tokens), consuming 2,653 more tokens than JSON_COMPACT without a corresponding accuracy benefit in that variant.
- Aggregation accuracy is inconsistent: 57.14% in mandatory falls to 41.27% in optional, a 15.87-percentage-point regression that is one of the largest category-level drops in the dataset.

#### 3.5.4 Use Case Recommendation

- ✓ Use for optional or sparse field datasets where the XML attribute representation communicates field presence clearly to the model.
- ✓ Use when structure awareness queries are a primary concern, as the optional variant leads this category at 81.48%.
- ❌ Avoid for mandatory-only datasets where the high token cost does not yield an accuracy advantage over JSON_COMPACT or TOON_DEFAULT.
- ❌ Avoid for aggregation-heavy workloads due to the large regression in that category under optional data.

#### 3.5.5 Trade-offs

- XML_COMPACT exhibits an unusual inversion where the optional variant is strictly better than the mandatory variant on both accuracy (+5.11 percentage points) and token cost (-763 tokens). This makes the optional variant the only configuration to recommend for this format. Compared to JSON_COMPACT optional (9,073 tokens, 76.61% accuracy), XML_COMPACT optional costs 2,417 more tokens for 1.89 percentage points more accuracy and a 6.33-point lower efficiency score. The token overhead is difficult to justify unless structure awareness performance is the critical requirement.

### 3.6 Detailed Analysis: XML_PRETTY

#### 3.6.1 Performance Summary

- Token Duration Range: 74 - 86 seconds
- Token Cost Range: 15419 - 16490 tokens
- Wasted Token Range: 3565 - 3857 tokens
- Accuracy Range: 76.61 - 76.88%
- Efficiency Score Range: 53.66 - 57.24

#### 3.6.2 Strengths

- Most consistent accuracy across mandatory and optional variants (0.27 percentage point difference), the smallest accuracy delta of all formats in the benchmark.
- Zero no-answer responses across all runs and both variants, indicating the verbose element structure always provides sufficient context for an answer.
- Structure awareness in the optional variant reaches 80.25%, benefiting from the explicit XML element hierarchy that makes structural relationships unambiguous.

#### 3.6.3 Weaknesses

- Highest token cost of all formats in both variants (16,490 mandatory, 15,419 optional), consuming 2.3 times the tokens of CSV and TOON_DEFAULT mandatory.
- Lowest efficiency score of all formats in both variants (53.66 mandatory, 57.24 optional), with no other format approaching this level of inefficiency.
- Highest absolute wasted tokens in the benchmark (3,857 mandatory, 3,565 optional), representing the largest absolute token inefficiency of any format tested.
- Provides no accuracy advantage over JSON_PRETTY despite costing 1,877 more tokens (mandatory). JSON_PRETTY outperforms XML_PRETTY by 5.92 percentage points in mandatory at lower cost.
- Lowest information-value-per-token across all formats (0.465 mandatory, 0.499 optional).

#### 3.6.4 Use Case Recommendation

- ✓ Use only when human readability of the source data file is required and token cost is entirely unconstrained.
- ✓ Use when near-zero accuracy variance across data schema versions is the sole optimization target.
- ❌ Avoid for all token-constrained applications. No other tested format simultaneously costs more and performs worse on the composite efficiency metric.
- ❌ Avoid when accuracy is being optimized, as JSON_PRETTY achieves higher accuracy at lower token cost in every measured variant.

#### 3.6.5 Trade-offs

- XML_PRETTY has no favorable accuracy-to-cost trade-off in this benchmark. JSON_PRETTY achieves 82.53% accuracy at 14,613 tokens (mandatory) while XML_PRETTY achieves 76.61% at 16,490 tokens. XML_PRETTY costs 13% more tokens to deliver 5.92 percentage points less accuracy. The only metric that approaches competitive value is the near-zero mandatory-to-optional accuracy delta (0.27 percentage points), which is a narrow property that does not compensate for the substantial overhead in typical use cases.

### 3.7 Detailed Analysis: YAML

#### 3.7.1 Performance Summary

- Token Duration Range: 76 - 104 seconds
- Token Cost Range: 12095 - 12876 tokens
- Wasted Token Range: 2373 - 3115 tokens
- Accuracy Range: 75.81 - 80.38%
- Efficiency Score Range: 64.54 - 70.21

#### 3.7.2 Strengths

- Highest accuracy of all optional variants at 80.38%, surpassing JSON_PRETTY optional (78.71%) while consuming 1,531 fewer tokens.
- Best filtering accuracy in the optional variant at 73.02%, which is 6.35 percentage points above the next best format (JSON_PRETTY optional at 66.67%).
- Near-perfect field retrieval in the optional variant at 99.39%, the highest field retrieval score across all formats and variants in the entire benchmark.
- Accuracy improves from mandatory to optional (+4.57%), one of only two formats with this positive trajectory. Wasted tokens also decrease 23.81% (3,115 to 2,373) as data becomes sparser, meaning both accuracy and efficiency improve together.

#### 3.7.3 Weaknesses

- Mandatory performance is materially weaker: structure awareness (64.20%) and overall accuracy (75.81%) both rank below JSON_PRETTY and TOON_DEFAULT in their respective mandatory configurations.
- Aggregation accuracy is below average in both variants (49.21% mandatory, 46.03% optional), placing YAML in the lower half of the benchmark for this category regardless of schema variant.
- Token cost in the optional variant (12,095) is 3,022 tokens higher than JSON_COMPACT optional (9,073) for a 3.77-percentage-point accuracy gain, which may not justify the overhead in budget-constrained contexts.
- Highest output reasoning duration in the optional variant (103,981 ms), the longest of any optional variant, indicating increased inference effort for complex questions.

#### 3.7.4 Use Case Recommendation

- ✓ Use for optional or sparse field datasets where strong field retrieval combined with above-average filtering accuracy is required.
- ✓ Use when the optional variant's accuracy (80.38%) and efficiency improvement over mandatory are the target operating conditions.
- ❌ Avoid for mandatory-only datasets where TOON_DEFAULT and JSON_COMPACT deliver better efficiency and comparable or superior accuracy.
- ❌ Avoid for aggregation-dominant workloads where performance consistently falls below 50% regardless of variant.

#### 3.7.5 Trade-offs

- YAML optional is one of the benchmark's most compelling configurations: 80.38% accuracy at 12,095 tokens with a 70.21 efficiency score. Compared to JSON_PRETTY optional (78.71% at 13,626 tokens), YAML provides 1.67 more accuracy points at 11.2% lower token cost, making it strictly preferable for optional data within this token range. The comparison against JSON_COMPACT optional is closer: YAML gains 3.77 accuracy points at the cost of 3,022 additional tokens and a 6.93-point reduction in efficiency score. Teams that prioritize raw accuracy will prefer YAML optional, while teams optimizing the accuracy-to-token ratio will favor JSON_COMPACT optional.

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