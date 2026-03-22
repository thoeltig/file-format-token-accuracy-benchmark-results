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

1. **JSON_COMPACT achieves the highest overall efficiency**: Lowest token cost (10,124–10,651 tokens) combined with the highest efficiency score (76.62–76.64) and best mandatory accuracy (68.82%) — the only format that leads across all three primary metrics simultaneously.

2. **Pretty-printed formats impose significant token overhead with no accuracy benefit**: JSON_PRETTY uses 70% more tokens than JSON_COMPACT and XML_PRETTY uses 92% more, yet both score lower in accuracy. XML_PRETTY has the worst efficiency score of all tested formats (45.22–45.56).

3. **Structure awareness improves with optional/sparse data for compact formats**: JSON_COMPACT (+14.82%), TOON_DEFAULT (+24.07%), XML_COMPACT (+8.64%), and YAML (+6.17%) all show improved structure awareness with sparse fields. JSON_PRETTY (−4.94%) and XML_PRETTY (−2.47%) show the opposite effect — indicating that compact syntax aids structural comprehension under sparsity, while verbose syntax obscures it.

4. **Aggregation is the weakest and most volatile category across all formats**: Scores range from 43.65% (TOON_DEFAULT optional) to 71.43% (JSON_COMPACT mandatory). JSON_COMPACT and TOON_DEFAULT both show large aggregation drops when switching to optional data (−19.05% and −22.22% respectively), revealing that sparse numeric fields are a consistent challenge for calculation accuracy.

5. **XML_COMPACT is the most robust format under data sparsity**: Only −1.35% accuracy drop from mandatory to optional (smallest of all formats), near-zero wasted token increase (+0.26%), and the lowest output token drift of all formats on optional data (±0.48%). The trade-off is a 24–26% token overhead over JSON_COMPACT.

6. **YAML shows the highest run-to-run variability on optional data**: Accuracy drift spans −12.56% to +11.73% (24.29% spread) on optional data, making it the least predictable format under sparsity. By contrast, YAML on mandatory data is the most stable format overall (only 2.36% spread), revealing a strong dependency on data completeness.

7. **TOON_DEFAULT achieves the best weighted accuracy on optional data (69.46%)** and the largest structure awareness gain when switching to sparse data (+24.07%), but suffers the worst structure awareness on mandatory data (58.64%) and the largest aggregation drop with optional fields (−22.22%). Its performance profile is the most context-dependent of all tested formats.

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
   - Optional: TOON_DEFAULT 67.07%
   - Mandatory: JSON_COMPACT 68.82%
- Lowest accuracy drift:
   - Optional: JSON_COMPACT ↓ -2.02% ↑ 2.82%
   - Mandatory: YAML ↓ -0.79% ↑ 1.57%
- Most useful tokens:
   - Optional: XML_PRETTY 12428 / 19927 tokens
   - Mandatory: XML_PRETTY 13307 / 20456 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 76.64
   - Mandatory: JSON_COMPACT 76.62
- Lowest delta (optional-mandatory):
   - Total tokens: TOON_DEFAULT -227 tokens
   - Accuracy: XML_COMPACT -1.35%
   - Token efficiency: JSON_COMPACT 0.02

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 19927 tokens
   - Mandatory: XML_PRETTY 20456 tokens
- Highest output token drift:
   - Optional: JSON_PRETTY ↓ -96.05% ↑ 49.34%
   - Mandatory: JSON_PRETTY ↓ -25.06% ↑ 49.22%
- Lowest accuracy:
   - Optional: JSON_PRETTY 61.02%
   - Mandatory: XML_PRETTY 65.05%
- Highest accuracy drift:
   - Optional: YAML ↓ -12.56% ↑ 11.73%
   - Mandatory: XML_PRETTY ↓ -11.98% ↑ 14.05%
- Most wasted tokens:
   - Optional: XML_PRETTY 7499 / 19927 tokens
   - Mandatory: XML_PRETTY 7149 / 20456 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 45.22
   - Mandatory: XML_PRETTY 45.56
- Highest delta (optional-mandatory):
   - Total tokens: JSON_PRETTY -1148 tokens
   - Accuracy: JSON_PRETTY -5.38%
   - Token efficiency: TOON_DEFAULT 1.69

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 37017s | JSON_COMPACT ≈ 10651 | JSON_COMPACT ≈ 3321 | JSON_COMPACT ≈ 69% | JSON_COMPACT ≈ 69% | JSON_COMPACT ≈ 77 | JSON_COMPACT ≈ 76  |
| JSON_COMPACT (+87.6%) | XML_COMPACT (+23.9%) | XML_COMPACT (+32.4%) | YAML (-0.5%) | YAML (-0.5%) | XML_COMPACT (-11.6%) | XML_COMPACT (-11.0%) |
| XML_PRETTY (+91.7%) | TOON_DEFAULT (+35.6%) | YAML (+39.9%) | XML_COMPACT (-2.1%) | XML_COMPACT (-1.5%) | YAML (-15.6%) | YAML (-15.6%) |
| XML_COMPACT (+102.6%) | YAML (+37.5%) | TOON_DEFAULT (+49.6%) | JSON_PRETTY (-2.4%) | JSON_PRETTY (-1.7%) | TOON_DEFAULT (-17.3%) | TOON_DEFAULT (-18.1%) |
| YAML (+147.2%) | JSON_PRETTY (+71.6%) | JSON_PRETTY (+84.9%) | TOON_DEFAULT (-3.2%) | XML_PRETTY (-3.7%) | JSON_PRETTY (-31.0%) | JSON_PRETTY (-30.4%) |
| JSON_PRETTY (+153.7%) | XML_PRETTY (+92.1%) | XML_PRETTY (+115.3%) | XML_PRETTY (-3.8%) | TOON_DEFAULT (-4.1%) | XML_PRETTY (-40.5%) | XML_PRETTY (-40.5%) |


##### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 39891s | JSON_COMPACT ≈ 10124 | JSON_COMPACT ≈ 3374 | TOON_DEFAULT ≈ 67% | TOON_DEFAULT ≈ 69% | JSON_COMPACT ≈ 77 | JSON_COMPACT ≈ 78  |
| XML_COMPACT (+60.3%) | XML_COMPACT (+25.6%) | XML_COMPACT (+30.7%) | JSON_COMPACT (-0.4%) | JSON_COMPACT (-1.1%) | XML_COMPACT (-11.0%) | XML_COMPACT (-11.2%) |
| XML_PRETTY (+94.1%) | TOON_DEFAULT (+40.4%) | TOON_DEFAULT (+38.7%) | YAML (-0.7%) | YAML (-2.1%) | TOON_DEFAULT (-15.1%) | TOON_DEFAULT (-14.2%) |
| JSON_PRETTY (+111.8%) | YAML (+42.1%) | YAML (+43.3%) | XML_COMPACT (-1.8%) | XML_COMPACT (-2.9%) | YAML (-16.4%) | YAML (-16.8%) |
| JSON_COMPACT (+120.4%) | JSON_PRETTY (+69.2%) | JSON_PRETTY (+97.9%) | XML_PRETTY (-4.7%) | XML_PRETTY (-6.4%) | JSON_PRETTY (-31.6%) | JSON_PRETTY (-31.8%) |
| YAML (+136.8%) | XML_PRETTY (+96.8%) | XML_PRETTY (+122.2%) | JSON_PRETTY (-6.0%) | JSON_PRETTY (-7.5%) | XML_PRETTY (-41.0%) | XML_PRETTY (-41.2%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 74% | JSON_PRETTY ≈ 72% | JSON_COMPACT ≈ 67% | JSON_COMPACT ≈ 71% |
| JSON_PRETTY (-1.2%) | XML_PRETTY (-2.5%) | XML_COMPACT (0.0%) | TOON_DEFAULT (-5.6%) |
| TOON_DEFAULT (-2.7%) | YAML (-2.5%) | JSON_PRETTY (-6.4%) | YAML (-11.1%) |
| XML_COMPACT (-3.6%) | XML_COMPACT (-3.7%) | YAML (-6.4%) | XML_PRETTY (-12.7%) |
| XML_PRETTY (-4.2%) | JSON_COMPACT (-3.7%) | TOON_DEFAULT (-7.1%) | XML_COMPACT (-15.9%) |
| JSON_COMPACT (-4.9%) | TOON_DEFAULT (-13.0%) | XML_PRETTY (-12.7%) | JSON_PRETTY (-22.2%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 67% | JSON_COMPACT ≈ 83% | TOON_DEFAULT ≈ 71% | YAML ≈ 56% |
| TOON_DEFAULT (-0.6%) | TOON_DEFAULT (-0.0%) | XML_PRETTY (-7.9%) | XML_COMPACT (-0.0%) |
| JSON_COMPACT (-1.2%) | XML_COMPACT (-6.2%) | YAML (-7.9%) | JSON_COMPACT (-3.2%) |
| XML_COMPACT (-3.0%) | YAML (-7.4%) | XML_COMPACT (-7.9%) | XML_PRETTY (-3.2%) |
| JSON_PRETTY (-3.6%) | XML_PRETTY (-16.0%) | JSON_COMPACT (-9.5%) | JSON_PRETTY (-9.5%) |
| XML_PRETTY (-3.6%) | JSON_PRETTY (-16.1%) | JSON_PRETTY (-9.5%) | TOON_DEFAULT (-11.9%) |


#### 2.1.5 Conclusion

**JSON_COMPACT is the optimal format** for AI data consumption with nested structures using Claude Haiku 4.5 with extended thinking. It delivers the highest efficiency score (76.64), the lowest token cost (10,124–10,651), and the best mandatory accuracy (68.82%) — no other format matches it across all three primary metrics. The efficiency advantage over the second-best format (XML_COMPACT, 68.19) is 12%, and over the worst (XML_PRETTY, 45.22) is 70%.

**XML_COMPACT** is the strongest alternative: second-best efficiency score (67.75–68.19), best resistance to data sparsity (only −1.35% accuracy drop), and the most consistent inference output (±0.48% output token drift on optional data). The 24% token overhead vs JSON_COMPACT is the main drawback.

**YAML and TOON_DEFAULT** occupy the middle ground (efficiency scores 63–65). YAML excels at field retrieval on mandatory data (73.94%, best overall) and is the most stable format under complete data conditions (2.36% accuracy spread). TOON_DEFAULT scores highest on optional data accuracy (67.07% raw, 69.46% weighted) and achieves the largest structure awareness gain with sparse fields (+24.07%), but suffers significant aggregation volatility with optional data (−22.22%).

**Pretty-printed formats should be avoided for AI consumption**: JSON_PRETTY and XML_PRETTY provide no accuracy advantage while consuming 70–92% more tokens than their compact equivalents. XML_PRETTY has the worst efficiency score (45.22–45.56), highest wasted token count (7,149–7,499 per run), and the lowest information density (0.313–0.318 info/token).

A consistent cross-format pattern: optional/sparse data degrades raw accuracy for all formats (−1.35% to −5.38%), but improves structure awareness for compact formats. This suggests that when data is sparse, models better recognize the data's structural shape — but only in formats where absent fields are not obscured by verbose syntax.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Char/Token | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
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
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Accuracy Man (%) | Accuracy Opt (%) | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
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
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
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
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10651 | 10124 | -527 | -4.95 | 7330 | 6749 | -581 | -7.92 | 3321 | 3374 |  +53 |  +1.60 | 68.82 | 66.67 | -2.15 | 76.62 | 76.64 |  +0.02 |  +0.03 |
| JSON_PRETTY | 18275 | 17127 | -1148 | -6.28 | 12135 | 10451 | -1684 | -13.88 | 6140 | 6676 |  +536 |  +8.72 | 66.40 | 61.02 | -5.38 | 52.83 | 52.392 | -0.44 | -0.83 |
| TOON_DEFAULT | 14438 | 14211 | -227 | -1.57 | 9470 | 9531 |  +61 |  +0.65 | 4968 | 4680 | -288 | -5.81 | 65.59 | 67.07 |  +1.48 | 63.38 | 65.0765 |  +1.69 |  +2.67 |
| XML_COMPACT | 13193 | 12713 | -480 | -3.64 | 8796 | 8304 | -492 | -5.59 | 4397 | 4409 |  +12 |  +0.26 | 66.67 | 65.32 | -1.35 | 67.75 | 68.194 |  +0.45 |  +0.66 |
| XML_PRETTY | 20456 | 19927 | -529 | -2.59 | 13307 | 12429 | -878 | -6.60 | 7149 | 7498 |  +349 |  +4.88 | 65.05 | 62.37 | -2.68 | 45.56 | 45.222 | -0.34 | -0.75 |
| YAML | 14644 | 14389 | -255 | -1.74 | 9999 | 9555 | -444 | -4.44 | 4645 | 4835 |  +190 |  +4.08 | 68.28 | 66.40 | -1.88 | 64.67 | 64.091 | -0.58 | -0.89 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
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
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 85 | 83 | -2 | -2.35 | 39 | 41 |  +2 |  +5.13 | 0 | 0 | 0 | 0.00 | 68.82 | 66.67 | -2.15 |
| JSON_PRETTY | 82 | 76 | -6 | -7.32 | 42 | 48 |  +6 |  +14.29 | 0 | 0 | 0 | 0.00 | 66.40 | 61.02 | -5.38 |
| TOON_DEFAULT | 81 | 83 |  +2 |  +2.47 | 43 | 41 | -2 | -4.65 | 0 | 0 | 0 | 0.00 | 65.59 | 67.07 |  +1.48 |
| XML_COMPACT | 83 | 81 | -2 | -2.41 | 41 | 43 |  +2 |  +4.88 | 0 | 0 | 0 | 0.00 | 66.67 | 65.32 | -1.35 |
| XML_PRETTY | 81 | 77 | -4 | -4.94 | 43 | 47 |  +4 |  +9.30 | 0 | 0 | 0 | 0.00 | 65.05 | 62.37 | -2.68 |
| YAML | 85 | 82 | -3 | -3.53 | 39 | 42 |  +3 |  +7.69 | 0 | 0 | 0 | 0.00 | 68.28 | 66.40 | -1.88 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
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
- Accuracy Range: 66.67 - 68.82%
- Efficiency Score Range: 76.62 - 76.64

#### 3.1.2 Strengths

- Lowest total token cost across all formats: 10,124–10,651 tokens (47–51% fewer than XML_PRETTY)
- Highest efficiency score: 76.62–76.64 (14% ahead of 2nd-place XML_COMPACT)
- Highest accuracy on mandatory data: 68.82%
- Best aggregation on mandatory data: 71.43% (highest aggregation score of all formats and variants)
- Strong structure awareness gain with optional data: +14.82% (67.90% man → 82.72% opt)
- Most consistent output token generation on mandatory data: −0.60% to +1.19% drift

#### 3.1.3 Weaknesses

- Aggregation drops sharply with optional data: 71.43% → 52.38% (−19.05%; largest single-category drop of all formats)
- Field retrieval on mandatory data (69.09%) is below YAML (73.94%) by 4.85 points
- Structure awareness on mandatory data (67.90%) is below average for that category
- High reasoning duration variability on optional data: −38.99% to +47.03% (85.02% spread)

#### 3.1.4 Use Case Recommendation

- ✓ Use when: Token budget is the primary constraint
- ✓ Use when: Data is complete/mandatory and aggregation accuracy on dense fields is important
- ✓ Use when: Consistent, predictable output token volume is needed (mandatory variant)
- ❌ Avoid when: Data has many optional/sparse fields and aggregation is a key operation (drops to 52.38%)
- ❌ Avoid when: Human readability of the source file is also required

#### 3.1.5 Trade-offs

- JSON_COMPACT achieves the best token-to-accuracy ratio of all tested formats. The primary risk is the 19-point aggregation drop between mandatory and optional variants. If data completeness is guaranteed, JSON_COMPACT is the clear default choice. If data sparsity is variable, aggregation results become unreliable and XML_COMPACT's stability may justify its 24% token overhead.
- The efficiency advantage over JSON_PRETTY is substantial: same or better accuracy at 41% lower token cost. Whitespace formatting is pure overhead for AI consumption.

### 3.2 Detailed Analysis: JSON_PRETTY

#### 3.2.1 Performance Summary

- Token Duration Range: 85 - 94 seconds
- Token Cost Range: 17127 - 18275 tokens
- Wasted Token Range: 6140 - 6676 tokens
- Accuracy Range: 61.02 - 66.40%
- Efficiency Score Range: 52.39 - 52.83

#### 3.2.2 Strengths

- Highest field retrieval on mandatory data: 72.73% (2nd best overall; only YAML mandatory at 73.94% is higher)
- Second-best structure awareness on mandatory data: 71.60%
- Filtering shows marginal improvement with optional data: +1.59%
- Most human-readable format for developer inspection and debugging

#### 3.2.3 Weaknesses

- Second-worst efficiency score: 52.39–52.83 (32% below JSON_COMPACT)
- Uses 70% more tokens than JSON_COMPACT (17,127–18,275 vs 10,124–10,651)
- Lowest accuracy on optional data: 61.02% (worst optional accuracy across all formats)
- Extreme output token variance on optional data: −96.05% to +49.34% (highly unstable inference behavior)
- Largest accuracy drop from mandatory to optional: −5.38% (worst format for data sparsity)
- Wasted tokens increase significantly with optional data: +536 tokens (+8.72%), the highest increase of all formats

#### 3.2.4 Use Case Recommendation

- ✓ Use when: Human review alongside AI processing is required (debugging, pipeline audit)
- ✓ Use when: Data is always complete/mandatory and field retrieval or structure awareness are the primary operations
- ❌ Avoid when: Token efficiency matters — 70% overhead over JSON_COMPACT with worse accuracy
- ❌ Avoid when: Data has optional fields — largest accuracy degradation of all formats (−5.38%)
- ❌ Avoid when: Consistent, deterministic output volume is required — extreme output token variance (−96% to +49%)

#### 3.2.5 Trade-offs

- JSON_COMPACT outperforms JSON_PRETTY on accuracy, token cost, and efficiency score simultaneously even on mandatory data (68.82% vs 66.40%, 42% fewer tokens). There is no scenario where JSON_PRETTY is superior to JSON_COMPACT for AI consumption alone.
- The only justification for JSON_PRETTY is human readability of the source file. This can be addressed without penalty by storing JSON_PRETTY for human review and converting to JSON_COMPACT for AI processing.

### 3.3 Detailed Analysis: TOON_DEFAULT

#### 3.3.1 Performance Summary

- Token Duration Range: 37 - 40 seconds
- Token Cost Range: 14211 - 14438 tokens
- Wasted Token Range: 4680 - 4968 tokens
- Accuracy Range: 65.59 - 67.07%
- Efficiency Score Range: 63.38 - 65.08

#### 3.3.2 Strengths

- Highest raw accuracy on optional/sparse data: 67.07% (best optional accuracy overall)
- Highest weighted accuracy on optional data: 69.46% (best across all formats and variants)
- Largest structure awareness gain with optional data: +24.07% (58.64% man → 82.72% opt; highest gain of all formats)
- Best filtering accuracy on optional data: 71.43% (highest filtering score of all formats and variants)
- Most stable token count between variants: only −227 tokens mandatory→optional (−1.57%)

#### 3.3.3 Weaknesses

- Worst structure awareness on mandatory data: 58.64% (13+ points below most other formats on the same variant)
- Largest aggregation drop from mandatory to optional: −22.22% (65.87% → 43.65%; worst aggregation degradation)
- Moderate efficiency score (63.38–65.08): 3rd–4th place, 13–17% below JSON_COMPACT
- High accuracy variance on optional data: −8.62% to +7.01% (15.63% spread)
- Note: the pre-filled "Token Duration Range: 37–40 seconds" in section 3.3.1 appears inconsistent with the performance table in section 2.4.1, which shows 74,267 ms (mandatory) and 96,677 ms (optional). All other formats' pre-filled ranges match their performance table values exactly — verify the TOON_DEFAULT duration measurement source

#### 3.3.4 Use Case Recommendation

- ✓ Use when: Data has optional/sparse fields and filtering or structure queries are the primary operations
- ✓ Use when: Weighted accuracy (emphasizing structure awareness and field retrieval) is the key metric
- ✓ Use when: Token count stability across varying data completeness is required
- ❌ Avoid when: Aggregation on sparse/optional data is a core operation (43.65% — worst aggregation score overall)
- ❌ Avoid when: Mandatory/dense data is the primary input and structure awareness questions are frequent (58.64% — worst in category)

#### 3.3.5 Trade-offs

- TOON_DEFAULT exhibits a notable inversion: the weakest structure awareness on mandatory data but the strongest on optional data. This suggests the format's syntax helps the model recognize absent fields more readily than present ones in dense data. If data completeness is variable and structure queries matter, TOON_DEFAULT is a stronger choice than JSON_COMPACT for optional data — despite 38% higher token cost.
- For mandatory data, JSON_COMPACT is superior on every metric except filtering. TOON_DEFAULT's use case is specifically optional/sparse data with structure-heavy query profiles.

### 3.4 Detailed Analysis: XML_COMPACT

#### 3.4.1 Performance Summary

- Token Duration Range: 64 - 75 seconds
- Token Cost Range: 12712 - 13193 tokens
- Wasted Token Range: 4397 - 4409 tokens
- Accuracy Range: 65.32 - 66.67%
- Efficiency Score Range: 67.75 - 68.19

#### 3.4.2 Strengths

- Second-best efficiency score: 67.75–68.19 (behind only JSON_COMPACT)
- Lowest output token drift on optional data: −0.39% to +0.48% (most consistent inference output of all formats)
- Highest chars/token ratio: 2.52 (most tokenizer-efficient character encoding)
- Best robustness to data sparsity: only −1.35% accuracy drop mandatory→optional (smallest degradation of all formats)
- Near-zero wasted token increase between variants: +12 tokens (+0.26%)
- Consistently strong weighted accuracy: 66.59–67.98%

#### 3.4.3 Weaknesses

- Weakest aggregation on mandatory data: 55.55% (15.88 points below JSON_COMPACT mandatory)
- Field retrieval drops −6.06% with optional data (larger than JSON_COMPACT's −3.03%)
- Uses 24–26% more tokens than JSON_COMPACT (12,712–13,193 vs 10,124–10,651)
- High reasoning duration variability on mandatory data: −31.71% to +59.18% (90.89% spread)

#### 3.4.4 Use Case Recommendation

- ✓ Use when: Most consistent inference behavior is required (lowest output token drift)
- ✓ Use when: Data completeness is uncertain — most robust format under sparsity
- ✓ Use when: XML is already the standard in the data pipeline (avoids format conversion)
- ❌ Avoid when: Aggregation queries are the primary use case (55.55% mandatory — well below JSON_COMPACT's 71.43%)
- ❌ Avoid when: Token budget is tight — JSON_COMPACT achieves comparable or better accuracy at 24% lower cost

#### 3.4.5 Trade-offs

- XML_COMPACT's primary differentiator is robustness: minimal accuracy degradation with optional fields, lowest output token variance, and near-zero wasted token increase between variants. These consistency properties come at a 24% token overhead over JSON_COMPACT.
- Decision rule: if data completeness is guaranteed, prefer JSON_COMPACT; if data completeness varies unpredictably, XML_COMPACT's stability may justify the extra tokens — particularly for pipelines where consistency across runs is critical.

### 3.5 Detailed Analysis: XML_PRETTY

#### 3.5.1 Performance Summary

- Token Duration Range: 71 - 77 seconds
- Token Cost Range: 19927 - 20456 tokens
- Wasted Token Range: 7149 - 7499 tokens
- Accuracy Range: 62.37 - 65.05%
- Efficiency Score Range: 45.22 - 45.56

#### 3.5.2 Strengths

- Most human-readable format of all tested formats
- Filtering improves with optional data: +9.52% (53.97% man → 63.49% opt; largest filtering gain of all formats)
- Stable output token generation on mandatory data: −1.27% to +1.07%
- Highest absolute useful token count in raw numbers (13,307/12,428) — though a consequence of high total tokens, not information density

#### 3.5.3 Weaknesses

- Worst efficiency score overall: 45.22–45.56 (40% below JSON_COMPACT)
- Highest token cost: 19,927–20,456 tokens (92% more than JSON_COMPACT)
- Lowest accuracy on mandatory data: 65.05% (worst mandatory performance of all formats)
- Highest wasted token count: 7,149–7,499 per run (35–37% of all tokens wasted)
- Highest accuracy variance on mandatory data: −11.98% to +14.05% (26.03% spread; least consistent for mandatory)
- Worst information density: 0.313–0.318 info/token (lowest of all formats)
- Structure awareness decreases with optional data: −2.47% (opposite of compact formats)

#### 3.5.4 Use Case Recommendation

- ✓ Use when: The file must be human-readable alongside AI consumption (audit trails, debugging)
- ✓ Use when: A downstream system strictly requires pretty-printed XML format
- ❌ Avoid when: Token efficiency is relevant — worst performer by a significant margin
- ❌ Avoid when: Consistent, predictable results are needed — highest accuracy variance on mandatory data (26% spread)
- ❌ Avoid when: Data has optional fields — accuracy degrades to 62.37% (2nd worst optional performance)

#### 3.5.5 Trade-offs

- XML_PRETTY provides no accuracy advantage over XML_COMPACT while costing 52–61% more tokens. Every accuracy metric is worse or equal. The sole objective justification is human readability.
- If human-readable XML is required, consider using XML_PRETTY for human review only and XML_COMPACT for AI processing — this preserves readability without paying the token penalty.

### 3.6 Detailed Analysis: YAML

#### 3.6.1 Performance Summary

- Token Duration Range: 92 - 94 seconds
- Token Cost Range: 14389 - 14644 tokens
- Wasted Token Range: 4645 - 4835 tokens
- Accuracy Range: 66.40 - 68.28%
- Efficiency Score Range: 64.09 - 64.67

#### 3.6.2 Strengths

- Highest field retrieval accuracy on mandatory data: 73.94% (best across all formats and variants)
- Second-best raw accuracy on mandatory data: 68.28% (behind only JSON_COMPACT at 68.82%)
- Most stable mandatory accuracy across runs: −0.79% to +1.57% (2.36% spread; lowest drift of any format on mandatory data)
- Filtering improves with optional data: +3.18%
- Stable token count between variants: −255 tokens (−1.74%)
- Consistent output token generation on mandatory data: −1.97% to +1.28%

#### 3.6.3 Weaknesses

- Highest accuracy variability on optional data: −12.56% to +11.73% (24.29% spread; worst optional stability of all formats)
- Weakest filtering on mandatory data: 60.31% (tied with JSON_PRETTY; lowest mandatory filtering)
- Low chars/token ratio: 1.789–1.799 (less token-efficient character encoding than compact formats)
- Slowest inference time: 91–94 seconds (highest total duration of all formats)
- Aggregation below average: 55.56–60.32%

#### 3.6.4 Use Case Recommendation

- ✓ Use when: Field retrieval from complete/mandatory datasets is the primary operation
- ✓ Use when: Consistent, predictable results on dense data are required (most stable mandatory performance)
- ✓ Use when: YAML is already the native format of the data source (e.g., configuration files, CI/CD artifacts)
- ❌ Avoid when: Data has optional/sparse fields — highest accuracy variance with incomplete data (±12% swing)
- ❌ Avoid when: Filtering is a primary query type on complete data (weakest mandatory filtering)
- ❌ Avoid when: Inference latency is critical — slowest format at 91–94 seconds

#### 3.6.5 Trade-offs

- YAML presents a binary reliability profile: highly stable and accurate on complete/mandatory data (best field retrieval, lowest accuracy drift), but the least predictable format on sparse/optional data (largest accuracy swing of ±12%). The decision to use YAML hinges entirely on data completeness guarantees.
- With guaranteed complete data, YAML is a strong second choice after JSON_COMPACT — particularly for field retrieval workloads. With variable/optional data, the 24.29% accuracy spread introduces unacceptable variance and XML_COMPACT or JSON_COMPACT are safer alternatives.

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-17
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Extended Thinking**: on
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

- **Report Generated**: 2026-03-17
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **With the help of**: Claude Sonnet 4.6
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Related Benchmark Results**: 
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)