# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: off
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
   - Optional: JSON_COMPACT 21522 tokens
   - Mandatory: XML_COMPACT 18168 tokens
- Lowest read token cost:
   - Optional: JSON_COMPACT 9645 tokens
   - Mandatory: JSON_COMPACT 10163 tokens
- Lowest output token cost:
   - Optional: YAML 8195 tokens
   - Mandatory: XML_COMPACT 5463 tokens
- Lowest output token cost drift:
   - Optional: YAML ↓ -20.26% ↑ 12.57%
   - Mandatory: JSON_COMPACT ↓ -12.67% ↑ 19.53%
- Highest accuracy:
   - Optional: JSON_COMPACT 78.07%
   - Mandatory: YAML 77.42%
- Lowest accuracy drift:
   - Optional: YAML ↓ -3.19% ↑ 2.79%
   - Mandatory: YAML ↓ -2.08% ↑ 3.13%
- Most useful tokens:
   - Optional: JSON_PRETTY 21273 / 28673 tokens
   - Mandatory: XML_PRETTY 21312 / 30260 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 76.31
   - Mandatory: XML_COMPACT 83.41
- Lowest delta (optional-mandatory):
   - Total tokens: JSON_COMPACT 1312 tokens
   - Accuracy: JSON_PRETTY 0.00%
   - Token efficiency: JSON_COMPACT -1.78

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: JSON_PRETTY 28673 tokens
   - Mandatory: XML_PRETTY 30260 tokens
- Highest read token cost:
   - Optional: XML_PRETTY 19671 tokens
   - Mandatory: XML_PRETTY 20204 tokens
- Highest output token cost:
   - Optional: TOON_DEFAULT 12878 tokens
   - Mandatory: YAML 10150 tokens
- Highest output token drift:
   - Optional: XML_PRETTY ↓ -40.84% ↑ 77.69%
   - Mandatory: XML_COMPACT ↓ -90.50% ↑ 76.78%
- Lowest accuracy:
   - Optional: YAML 67.47%
   - Mandatory: XML_PRETTY 70.43%
- Highest accuracy drift:
   - Optional: XML_PRETTY ↓ -9.43% ↑ 15.47%
   - Mandatory: XML_COMPACT ↓ -11.27% ↑ 11.97%
- Most wasted tokens:
   - Optional: XML_PRETTY 8083 / 28106 tokens
   - Mandatory: XML_PRETTY 8948 / 30260 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 55.23
   - Mandatory: XML_PRETTY 49.33
- Highest delta (optional-mandatory):
   - Total tokens: XML_COMPACT 3753 tokens
   - Accuracy: YAML -9.95%
   - Token efficiency: XML_COMPACT -10.42

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 40s | JSON_COMPACT ≈ 10163 | XML_COMPACT ≈ 5463 | XML_COMPACT ≈ 18168 | XML_COMPACT ≈ 4298 | YAML ≈ 77% | YAML ≈ 76% | XML_COMPACT ≈ 83 | XML_COMPACT ≈ 82 |
| JSON_PRETTY (+91.5%) | XML_COMPACT (+25.0%) | TOON_DEFAULT (+64.9%) | JSON_COMPACT (+11.2%) | JSON_COMPACT (+13.0%) | XML_COMPACT (-1.1%) | XML_COMPACT (-1.9%) | JSON_COMPACT (-6.4%) | JSON_COMPACT (-6.9%) |
| XML_COMPACT (+98.3%) | TOON_DEFAULT (+38.4%) | JSON_PRETTY (+71.8%) | TOON_DEFAULT (+27.0%) | YAML (+27.7%) | JSON_COMPACT (-1.5%) | TOON_DEFAULT (-2.0%) | TOON_DEFAULT (-14.9%) | TOON_DEFAULT (-14.9%) |
| XML_PRETTY (+102.4%) | YAML (+39.3%) | JSON_COMPACT (+83.9%) | YAML (+33.8%) | TOON_DEFAULT (+29.3%) | TOON_DEFAULT (-1.5%) | JSON_COMPACT (-2.8%) | YAML (-17.3%) | YAML (-16.9%) |
| JSON_COMPACT (+103.4%) | JSON_PRETTY (+74.0%) | XML_PRETTY (+84.1%) | JSON_PRETTY (+49.0%) | JSON_PRETTY (+62.5%) | JSON_PRETTY (-3.2%) | JSON_PRETTY (-3.0%) | JSON_PRETTY (-28.2%) | JSON_PRETTY (-27.8%) |
| YAML (+113.3%) | XML_PRETTY (+98.8%) | YAML (+85.8%) | XML_PRETTY (+66.6%) | XML_PRETTY (+108.2%) | XML_PRETTY (-7.0%) | XML_PRETTY (-7.3%) | XML_PRETTY (-40.9%) | XML_PRETTY (-41.1%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 61s | JSON_COMPACT ≈ 9645 | YAML ≈ 8195 | JSON_COMPACT ≈ 21522 | JSON_COMPACT ≈ 4720 | JSON_COMPACT ≈ 78% | TOON_DEFAULT ≈ 77% | JSON_COMPACT ≈ 76 | JSON_COMPACT ≈ 75 |
| YAML (+11.0%) | XML_COMPACT (+29.1%) | XML_PRETTY (+2.9%) | XML_COMPACT (+1.8%) | XML_COMPACT (+17.4%) | TOON_DEFAULT (-0.8%) | JSON_COMPACT (-0.8%) | XML_COMPACT (-4.4%) | XML_COMPACT (-3.2%) |
| XML_PRETTY (+13.5%) | TOON_DEFAULT (+43.4%) | XML_COMPACT (+15.5%) | YAML (+3.8%) | TOON_DEFAULT (+28.5%) | XML_COMPACT (-3.3%) | XML_COMPACT (-2.9%) | YAML (-12.4%) | YAML (-11.4%) |
| XML_COMPACT (+26.1%) | YAML (+46.7%) | JSON_COMPACT (+44.9%) | TOON_DEFAULT (+24.1%) | YAML (+54.0%) | JSON_PRETTY (-3.9%) | JSON_PRETTY (-4.2%) | TOON_DEFAULT (-17.5%) | TOON_DEFAULT (-16.3%) |
| JSON_PRETTY (+55.8%) | JSON_PRETTY (+73.7%) | JSON_PRETTY (+45.4%) | XML_PRETTY (+30.6%) | JSON_PRETTY (+56.8%) | XML_PRETTY (-6.8%) | XML_PRETTY (-6.6%) | JSON_PRETTY (-26.8%) | JSON_PRETTY (-26.7%) |
| JSON_COMPACT (+56.3%) | XML_PRETTY (+104.0%) | TOON_DEFAULT (+57.1%) | JSON_PRETTY (+33.2%) | XML_PRETTY (+71.3%) | YAML (-10.6%) | YAML (-10.1%) | XML_PRETTY (-27.6%) | XML_PRETTY (-27.1%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99% | YAML ≈ 70% | YAML ≈ 62% | XML_COMPACT ≈ 68% |
| JSON_COMPACT (-3.0%) | JSON_PRETTY (-0.7%) | JSON_PRETTY (-1.9%) | JSON_COMPACT (-8.3%) |
| TOON_DEFAULT (-5.8%) | XML_COMPACT (-1.2%) | JSON_COMPACT (-3.8%) | TOON_DEFAULT (-10.5%) |
| JSON_PRETTY (-7.8%) | TOON_DEFAULT (-2.3%) | TOON_DEFAULT (-4.2%) | XML_PRETTY (-19.0%) |
| XML_COMPACT (-8.5%) | XML_PRETTY (-7.4%) | XML_COMPACT (-6.3%) | JSON_PRETTY (-19.7%) |
| XML_PRETTY (-10.9%) | JSON_COMPACT (-9.6%) | XML_PRETTY (-7.9%) | YAML (-23.8%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 98% | TOON_DEFAULT ≈ 78% | JSON_PRETTY ≈ 68% | JSON_COMPACT ≈ 53% |
| JSON_PRETTY (-2.4%) | XML_COMPACT (-4.9%) | TOON_DEFAULT (-1.6%) | TOON_DEFAULT (-8.4%) |
| XML_COMPACT (-3.6%) | JSON_COMPACT (-10.4%) | JSON_COMPACT (-4.4%) | XML_PRETTY (-12.1%) |
| TOON_DEFAULT (-4.7%) | XML_PRETTY (-11.1%) | XML_PRETTY (-6.3%) | YAML (-13.6%) |
| XML_PRETTY (-9.7%) | YAML (-14.8%) | YAML (-6.3%) | JSON_PRETTY (-13.7%) |
| YAML (-15.8%) | JSON_PRETTY (-16.1%) | XML_COMPACT (-6.4%) | XML_COMPACT (-15.2%) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Char/Token | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10163 | 10048 | 20211 | 2.246 | 0.376 | 81.031 | 75.97 | 76.33 | 15354.145 | 4856.655 | 78.09 | 76.33 |
| JSON_COMPACT | opt | 9645 | 11877 | 21522 | 2.219 | 0.363 | 95.785 | 78.07 | 75.17 | 16802.538 | 4719.862 | 76.31 | 75.17 |
| JSON_PRETTY | man | 17682 | 9383 | 27065 | 1.777 | 0.274 | 75.673 | 74.19 | 59.20 | 20079.820 | 6985.580 | 59.87 | 59.20 |
| JSON_PRETTY | opt | 16757 | 11916 | 28673 | 1.767 | 0.259 | 96.099 | 74.19 | 55.12 | 21272.746 | 7400.587 | 55.89 | 55.12 |
| TOON_DEFAULT | man | 14068 | 9010 | 23078 | 1.855 | 0.331 | 72.660 | 75.91 | 69.76 | 17518.333 | 5559.434 | 70.95 | 69.76 |
| TOON_DEFAULT | opt | 13832 | 12878 | 26710 | 1.864 | 0.293 | 103.852 | 77.30 | 62.90 | 20646.572 | 6063.094 | 62.93 | 62.90 |
| XML_COMPACT | man | 12705 | 5463 | 18168 | 2.551 | 0.420 | 44.054 | 76.34 | 82.02 | 13869.197 | 4298.470 | 83.41 | 82.02 |
| XML_COMPACT | opt | 12455 | 9466 | 21921 | 2.499 | 0.341 | 76.335 | 74.73 | 72.73 | 16381.190 | 5539.310 | 72.99 | 72.73 |
| XML_PRETTY | man | 20204 | 10056 | 30260 | 1.985 | 0.233 | 81.097 | 70.43 | 48.28 | 21312.118 | 8947.882 | 49.33 | 48.28 |
| XML_PRETTY | opt | 19671 | 8435 | 28106 | 1.974 | 0.253 | 68.024 | 71.24 | 54.84 | 20022.714 | 8083.286 | 55.23 | 54.84 |
| YAML | man | 14155 | 10150 | 24305 | 1.808 | 0.319 | 81.855 | 77.42 | 68.15 | 18816.931 | 5488.069 | 68.97 | 68.15 |
| YAML | opt | 14148 | 8195 | 22343 | 1.787 | 0.302 | 66.089 | 67.47 | 66.63 | 15074.822 | 7268.178 | 66.86 | 66.63 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10163 | 9645 | -518 | -5.10 | 10048 | 11878 |  +1830 |  +18.21 | 20211 | 21523 |  +1312 |  +6.49 | 73.45 | 76.44 |  +2.99 | 76.33 | 75.17 | -1.16 |
| JSON_PRETTY | 17682 | 16757 | -925 | -5.23 | 9383 | 11916 |  +2533 |  +27.00 | 27065 | 28673 |  +1608 |  +5.94 | 73.24 | 73.09 | -0.15 | 59.20 | 55.12 | -4.09 |
| TOON_DEFAULT | 14068 | 13832 | -236 | -1.68 | 9010 | 12878 |  +3868 |  +42.93 | 23078 | 26710 |  +3632 |  +15.74 | 74.21 | 77.26 |  +3.05 | 69.76 | 62.90 | -6.86 |
| XML_COMPACT | 12705 | 12455 | -250 | -1.97 | 5463 | 9466 |  +4003 |  +73.27 | 18168 | 21921 |  +3753 |  +20.66 | 74.35 | 74.36 |  +0.01 | 82.02 | 72.73 | -9.29 |
| XML_PRETTY | 20204 | 19671 | -533 | -2.64 | 10056 | 8435 | -1621 | -16.12 | 30260 | 28106 | -2154 | -7.12 | 68.94 | 70.68 |  +1.74 | 48.28 | 54.84 |  +6.55 |
| YAML | 14155 | 14148 | -7 | -0.05 | 10150 | 8195 | -1955 | -19.26 | 24305 | 22343 | -1962 | -8.07 | 76.25 | 67.13 | -9.12 | 68.15 | 66.63 | -1.52 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 13 | 781.769 | 0.42 | 81772 | 0.123 | 659.45 | 81785 | 781.892 | 527.65 |
| JSON_COMPACT | opt | 9 | 1071.667 | 0.29 | 94662 | 0.125 | 763.40 | 94671 | 1071.792 | 610.78 |
| JSON_PRETTY | man | 262 | 67.489 | 8.45 | 76737 | 0.122 | 618.85 | 76999 | 67.611 | 496.77 |
| JSON_PRETTY | opt | 270 | 62.063 | 8.71 | 94082 | 0.127 | 758.73 | 94352 | 62.190 | 608.72 |
| TOON_DEFAULT | man | 31 | 453.806 | 1.00 | 82789 | 0.108 | 667.66 | 82820 | 453.914 | 534.32 |
| TOON_DEFAULT | opt | 29 | 476.966 | 0.94 | 101511 | 0.126 | 818.63 | 101540 | 477.092 | 655.09 |
| XML_COMPACT | man | 13 | 977.308 | 0.42 | 79723 | 0.069 | 642.93 | 79736 | 977.377 | 514.43 |
| XML_COMPACT | opt | 12 | 1037.917 | 0.39 | 76345 | 0.124 | 615.69 | 76357 | 1038.041 | 492.63 |
| XML_PRETTY | man | 13 | 1554.154 | 0.42 | 81393 | 0.124 | 656.40 | 81406 | 1554.278 | 525.20 |
| XML_PRETTY | opt | 6 | 3278.500 | 0.19 | 68743 | 0.123 | 554.38 | 68749 | 3278.623 | 443.54 |
| YAML | man | 12 | 1179.583 | 0.39 | 85781 | 0.118 | 691.78 | 85793 | 1179.701 | 553.51 |
| YAML | opt | 11 | 1286.182 | 0.35 | 67236 | 0.122 | 542.22 | 67247 | 1286.304 | 433.85 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) | Total Man (s) | Total Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 13 | 9 | -4 | -30.77 | 81.77 | 94.66 |  +12.89 |  +15.76 | 81.79 | 94.67 |  +12.89 |  +15.76 |
| JSON_PRETTY | 262 | 270 |  +8 |  +3.05 | 76.74 | 94.08 |  +17.35 |  +22.60 | 77.00 | 94.35 |  +17.35 |  +22.54 |
| TOON_DEFAULT | 31 | 29 | -2 | -6.45 | 82.79 | 101.51 |  +18.72 |  +22.61 | 82.82 | 103.17 |  +20.35 |  +24.57 |
| XML_COMPACT | 13 | 12 | -1 | -7.69 | 79.72 | 76.34 | -3.38 | -4.24 | 79.74 | 76.36 | -3.38 | -4.24 |
| XML_PRETTY | 13 | 6 | -7 | -53.85 | 81.39 | 68.74 | -12.65 | -15.54 | 81.41 | 68.75 | -12.66 | -15.55 |
| YAML | 12 | 11 | -1 | -8.33 | 85.78 | 67.24 | -18.55 | -21.62 | 85.79 | 67.25 | -18.55 | -21.62 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Char/Token | Token/Value | Token/Object | Info/Token |
|---|---|---|---|---|---|
| JSON_COMPACT | man | 2.246 | 14.902 | 327.839 | 0.376 |
| JSON_COMPACT | opt | 2.219 | 15.285 | 311.129 | 0.363 |
| JSON_PRETTY | man | 1.777 | 25.927 | 570.387 | 0.274 |
| JSON_PRETTY | opt | 1.767 | 26.556 | 540.548 | 0.259 |
| TOON_DEFAULT | man | 1.855 | 20.628 | 453.806 | 0.331 |
| TOON_DEFAULT | opt | 1.864 | 21.921 | 446.194 | 0.293 |
| XML_COMPACT | man | 2.551 | 18.629 | 409.839 | 0.420 |
| XML_COMPACT | opt | 2.499 | 19.739 | 401.774 | 0.341 |
| XML_PRETTY | man | 1.985 | 29.625 | 651.742 | 0.233 |
| XML_PRETTY | opt | 1.974 | 31.174 | 634.548 | 0.253 |
| YAML | man | 1.808 | 20.755 | 456.613 | 0.319 |
| YAML | opt | 1.787 | 22.422 | 456.387 | 0.302 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 2.246 | 2.219 | -0.027 | -1.20 | 14.902 | 15.285 |  +0.383 |  +2.57 | 327.839 | 311.129 | -16.710 | -5.10 | 0.376 | 0.363 | -0.013 | -3.46 |
| JSON_PRETTY | 1.777 | 1.767 | -0.010 | -0.56 | 25.927 | 26.556 |  +0.629 |  +2.43 | 570.387 | 540.548 | -29.839 | -5.23 | 0.274 | 0.259 | -0.015 | -5.47 |
| TOON_DEFAULT | 1.855 | 1.864 |  +0.009 |  +0.49 | 20.628 | 21.921 |  +1.293 |  +6.27 | 453.806 | 446.194 | -7.612 | -1.68 | 0.331 | 0.293 | -0.037 | -11.35 |
| XML_COMPACT | 2.551 | 2.499 | -0.052 | -2.04 | 18.629 | 19.739 |  +1.110 |  +5.96 | 409.839 | 401.774 | -8.065 | -1.97 | 0.420 | 0.341 | -0.079 | -18.81 |
| XML_PRETTY | 1.985 | 1.974 | -0.011 | -0.55 | 29.625 | 31.174 |  +1.549 |  +5.23 | 651.742 | 634.548 | -17.194 | -2.64 | 0.233 | 0.253 |  +0.020 |  +8.58 |
| YAML | 1.808 | 1.787 | -0.021 | -1.16 | 20.755 | 22.422 |  +1.667 |  +8.03 | 456.613 | 456.387 | -0.226 | -0.05 | 0.319 | 0.302 | -0.017 | -5.33 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 20211 | 15354 | 4857 | 75.97 | 73.45 | 78.09 | 76.33 |
| JSON_COMPACT | opt | 21522 | 16803 | 4720 | 78.07 | 76.44 | 76.31 | 75.17 |
| JSON_PRETTY | man | 27065 | 20080 | 6986 | 74.19 | 73.24 | 59.87 | 59.20 |
| JSON_PRETTY | opt | 28673 | 21273 | 7401 | 74.19 | 73.09 | 55.89 | 55.12 |
| TOON_DEFAULT | man | 23078 | 17518 | 5559 | 75.91 | 74.21 | 70.95 | 69.76 |
| TOON_DEFAULT | opt | 26710 | 20647 | 6063 | 77.30 | 77.26 | 62.93 | 62.90 |
| XML_COMPACT | man | 18168 | 13869 | 4298 | 76.34 | 74.35 | 83.41 | 82.02 |
| XML_COMPACT | opt | 21921 | 16381 | 5539 | 74.73 | 74.36 | 72.99 | 72.73 |
| XML_PRETTY | man | 30260 | 21312 | 8948 | 70.43 | 68.94 | 49.33 | 48.28 |
| XML_PRETTY | opt | 28106 | 20023 | 8083 | 71.24 | 70.68 | 55.23 | 54.84 |
| YAML | man | 24305 | 18817 | 5488 | 77.42 | 76.25 | 68.97 | 68.15 |
| YAML | opt | 22343 | 15075 | 7268 | 67.47 | 67.13 | 66.86 | 66.63 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 20211 | 21523 |  +1312 |  +6.49 | 15354 | 16802 |  +1448 |  +9.43 | 4857 | 4720 | -137 | -2.82 | 75.97 | 78.07 |  +2.10 | 78.09 | 76.315 | -1.78 | -2.28 |
| JSON_PRETTY | 27065 | 28673 |  +1608 |  +5.94 | 20080 | 21273 |  +1193 |  +5.94 | 6986 | 7401 |  +415 |  +5.94 | 74.19 | 74.19 | 0.00 | 59.87 | 55.888 | -3.98 | -6.65 |
| TOON_DEFAULT | 23078 | 26710 |  +3632 |  +15.74 | 17518 | 20646 |  +3128 |  +17.86 | 5559 | 6063 |  +504 |  +9.06 | 75.91 | 77.30 |  +1.39 | 70.95 | 62.9285 | -8.02 | -11.31 |
| XML_COMPACT | 18168 | 21921 |  +3753 |  +20.66 | 13869 | 16381 |  +2512 |  +18.11 | 4298 | 5539 |  +1241 |  +28.87 | 76.34 | 74.73 | -1.61 | 83.41 | 72.991 | -10.42 | -12.49 |
| XML_PRETTY | 30260 | 28106 | -2154 | -7.12 | 21312 | 20023 | -1289 | -6.05 | 8948 | 8083 | -865 | -9.66 | 70.43 | 71.24 |  +0.81 | 49.33 | 55.228 |  +5.90 |  +11.97 |
| YAML | 24305 | 22343 | -1962 | -8.07 | 18817 | 15075 | -3742 | -19.89 | 5488 | 7268 |  +1780 |  +32.44 | 77.42 | 67.47 | -9.95 | 68.97 | 66.863 | -2.11 | -3.05 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| JSON_COMPACT | man | 94 | 30 | 0 | 75.97 |
| JSON_COMPACT | opt | 97 | 27 | 0 | 78.07 |
| JSON_PRETTY | man | 92 | 32 | 0 | 74.19 |
| JSON_PRETTY | opt | 92 | 32 | 0 | 74.19 |
| TOON_DEFAULT | man | 94 | 30 | 0 | 75.91 |
| TOON_DEFAULT | opt | 96 | 28 | 0 | 77.30 |
| XML_COMPACT | man | 95 | 29 | 0 | 76.34 |
| XML_COMPACT | opt | 93 | 31 | 0 | 74.73 |
| XML_PRETTY | man | 87 | 37 | 0 | 70.43 |
| XML_PRETTY | opt | 88 | 36 | 0 | 71.24 |
| YAML | man | 96 | 28 | 0 | 77.42 |
| YAML | opt | 84 | 40 | 0 | 67.47 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 94 | 97 |  +3 |  +3.19 | 30 | 27 | -3 | -10.00 | 0 | 0 | 0 | 0.00 | 75.97 | 78.07 |  +2.10 |
| JSON_PRETTY | 92 | 92 | 0 | 0.00 | 32 | 32 | 0 | 0.00 | 0 | 0 | 0 | 0.00 | 74.19 | 74.19 | 0.00 |
| TOON_DEFAULT | 94 | 96 |  +2 |  +2.13 | 30 | 28 | -2 | -6.67 | 0 | 0 | 0 | 0.00 | 75.91 | 77.30 |  +1.39 |
| XML_COMPACT | 95 | 93 | -2 | -2.11 | 29 | 31 |  +2 |  +6.90 | 0 | 0 | 0 | 0.00 | 76.34 | 74.73 | -1.61 |
| XML_PRETTY | 87 | 88 |  +1 |  +1.15 | 37 | 36 | -1 | -2.70 | 0 | 0 | 0 | 0.00 | 70.43 | 71.24 |  +0.81 |
| YAML | 96 | 84 | -12 | -12.50 | 28 | 40 |  +12 |  +42.86 | 0 | 0 | 0 | 0.00 | 77.42 | 67.47 | -9.95 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 75.97 | 96.36 | 60.74 | 58.09 | 60.00 |
| JSON_COMPACT | opt | 78.07 | 98.18 | 67.41 | 63.81 | 53.33 |
| JSON_PRETTY | man | 74.19 | 91.64 | 69.63 | 60.00 | 48.57 |
| JSON_PRETTY | opt | 74.19 | 95.76 | 61.73 | 68.25 | 39.68 |
| TOON_DEFAULT | man | 75.91 | 93.64 | 68.06 | 57.74 | 57.74 |
| TOON_DEFAULT | opt | 77.30 | 93.51 | 77.78 | 66.67 | 44.90 |
| XML_COMPACT | man | 76.34 | 90.91 | 69.14 | 55.55 | 68.25 |
| XML_COMPACT | opt | 74.73 | 94.55 | 72.84 | 61.90 | 38.10 |
| XML_PRETTY | man | 70.43 | 88.48 | 62.96 | 53.97 | 49.21 |
| XML_PRETTY | opt | 71.24 | 88.48 | 66.67 | 61.90 | 41.27 |
| YAML | man | 77.42 | 99.39 | 70.37 | 61.90 | 44.45 |
| YAML | opt | 67.47 | 82.42 | 62.96 | 61.90 | 39.68 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 96.36 | 98.18 |  +1.82 |
| JSON_PRETTY | 91.64 | 95.76 |  +4.12 |
| TOON_DEFAULT | 93.64 | 93.51 | -0.13 |
| XML_COMPACT | 90.91 | 94.55 |  +3.64 |
| XML_PRETTY | 88.48 | 88.48 | 0.00 |
| YAML | 99.39 | 82.42 | -16.97 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 60.74 | 67.41 |  +6.67 |
| JSON_PRETTY | 69.63 | 61.73 | -7.90 |
| TOON_DEFAULT | 68.06 | 77.78 |  +9.72 |
| XML_COMPACT | 69.14 | 72.84 |  +3.70 |
| XML_PRETTY | 62.96 | 66.67 |  +3.70 |
| YAML | 70.37 | 62.96 | -7.41 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 58.09 | 63.81 |  +5.71 |
| JSON_PRETTY | 60.00 | 68.25 |  +8.26 |
| TOON_DEFAULT | 57.74 | 66.67 |  +8.93 |
| XML_COMPACT | 55.55 | 61.90 |  +6.35 |
| XML_PRETTY | 53.97 | 61.90 |  +7.94 |
| YAML | 61.90 | 61.90 |  +0.00 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 60.00 | 53.33 | -6.67 |
| JSON_PRETTY | 48.57 | 39.68 | -8.89 |
| TOON_DEFAULT | 57.74 | 44.90 | -12.84 |
| XML_COMPACT | 68.25 | 38.10 | -30.16 |
| XML_PRETTY | 49.21 | 41.27 | -7.93 |
| YAML | 44.45 | 39.68 | -4.76 |

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: off
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
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_on_verify\BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)