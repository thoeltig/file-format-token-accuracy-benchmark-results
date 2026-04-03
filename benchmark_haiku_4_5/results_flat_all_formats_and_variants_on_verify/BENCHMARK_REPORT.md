# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: on
- **Data Structure**: flat
- **Formats Tested**: 7 (CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: Second iteration

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
   - Optional: CSV 15927 tokens
   - Mandatory: CSV 11621 tokens
- Lowest read token cost:
   - Optional: CSV 6687 tokens
   - Mandatory: CSV 6968 tokens
- Lowest output token cost:
   - Optional: JSON_COMPACT 9159 tokens
   - Mandatory: CSV 4653 tokens
- Lowest output token cost drift:
   - Optional: YAML ↓ -0.99% ↑ 1.46%
   - Mandatory: YAML ↓ -11.81% ↑ 17.87%
- Highest accuracy:
   - Optional: TOON_DEFAULT 79.84%
   - Mandatory: YAML 81.18%
- Lowest accuracy drift:
   - Optional: TOON_DEFAULT ↓ -2.02% ↑ 4.03%
   - Mandatory: JSON_PRETTY ↓ -1.00% ↑ 0.99%
- Most useful tokens:
   - Optional: XML_PRETTY 21301 / 26680 tokens
   - Mandatory: JSON_PRETTY 22390 / 27762 tokens
- Highest token efficiency (%/token):
   - Optional: JSON_COMPACT 72.55
   - Mandatory: CSV 78.34
- Lowest delta (optional-mandatory):
   - Total tokens: XML_PRETTY 620 tokens
   - Accuracy: TOON_DEFAULT 0.54%
   - Token efficiency: YAML -0.04

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 26680 tokens
   - Mandatory: JSON_PRETTY 27762 tokens
- Highest read token cost:
   - Optional: XML_PRETTY 15076 tokens
   - Mandatory: XML_PRETTY 16137 tokens
- Highest output token cost:
   - Optional: TOON_DEFAULT 11937 tokens
   - Mandatory: TOON_DEFAULT 14468 tokens
- Highest output token drift:
   - Optional: JSON_PRETTY ↓ -49.36% ↑ 61.98%
   - Mandatory: CSV ↓ -97.66% ↑ 188.78%
- Lowest accuracy:
   - Optional: CSV 62.90%
   - Mandatory: CSV 69.08%
- Highest accuracy drift:
   - Optional: JSON_COMPACT ↓ -9.38% ↑ 7.28%
   - Mandatory: TOON_DEFAULT ↓ -16.61% ↑ 10.84%
- Most wasted tokens:
   - Optional: CSV 5909 / 15927 tokens
   - Mandatory: XML_PRETTY 5465 / 26060 tokens
- Lowest token efficiency (%/token):
   - Optional: XML_PRETTY 57.92
   - Mandatory: JSON_PRETTY 56.47
- Highest delta (optional-mandatory):
   - Total tokens: CSV 4307 tokens
   - Accuracy: YAML -7.25%
   - Token efficiency: CSV -12.32

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 42s | CSV ≈ 6968 | CSV ≈ 4653 | CSV ≈ 11621 | CSV ≈ 3593 | YAML ≈ 81% | YAML ≈ 79% | CSV ≈ 78 | CSV ≈ 79 |
| XML_PRETTY (+100.3%) | TOON_DEFAULT (+0.7%) | XML_PRETTY (+113.3%) | JSON_COMPACT (+76.3%) | TOON_DEFAULT (+23.8%) | JSON_PRETTY (-0.5%) | JSON_PRETTY (-0.7%) | TOON_DEFAULT (-14.2%) | TOON_DEFAULT (-15.3%) |
| YAML (+117.3%) | JSON_COMPACT (+32.8%) | YAML (+140.3%) | TOON_DEFAULT (+84.9%) | YAML (+24.2%) | TOON_DEFAULT (-1.9%) | TOON_DEFAULT (-1.1%) | JSON_COMPACT (-15.0%) | JSON_COMPACT (-16.2%) |
| JSON_COMPACT (+122.1%) | XML_COMPACT (+67.5%) | JSON_COMPACT (+141.4%) | YAML (+104.1%) | JSON_COMPACT (+37.9%) | XML_PRETTY (-2.2%) | XML_PRETTY (-2.0%) | YAML (-17.8%) | YAML (-19.6%) |
| JSON_PRETTY (+154.8%) | YAML (+79.9%) | JSON_PRETTY (+190.3%) | XML_COMPACT (+117.9%) | JSON_PRETTY (+49.5%) | XML_COMPACT (-2.7%) | XML_COMPACT (-2.4%) | XML_COMPACT (-24.0%) | XML_COMPACT (-25.5%) |
| XML_COMPACT (+161.8%) | JSON_PRETTY (+104.6%) | XML_COMPACT (+193.3%) | XML_PRETTY (+124.3%) | XML_COMPACT (+51.5%) | JSON_COMPACT (-5.4%) | JSON_COMPACT (-4.8%) | XML_PRETTY (-25.3%) | XML_PRETTY (-26.9%) |
| CSV (+199.3%) | XML_PRETTY (+131.6%) | TOON_DEFAULT (+211.0%) | JSON_PRETTY (+138.9%) | XML_PRETTY (+52.1%) | CSV (-12.1%) | CSV (-10.1%) | JSON_PRETTY (-27.9%) | JSON_PRETTY (-29.8%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 45s | CSV ≈ 6687 | JSON_COMPACT ≈ 9159 | CSV ≈ 15927 | JSON_COMPACT ≈ 4039 | TOON_DEFAULT ≈ 80% | XML_PRETTY ≈ 80% | JSON_COMPACT ≈ 73 | JSON_COMPACT ≈ 72 |
| YAML (+52.9%) | JSON_COMPACT (+30.5%) | CSV (+0.9%) | JSON_COMPACT (+12.3%) | TOON_DEFAULT (+17.2%) | XML_PRETTY (0.0%) | TOON_DEFAULT (-0.7%) | CSV (-9.0%) | CSV (-9.1%) |
| JSON_COMPACT (+68.1%) | XML_COMPACT (+63.1%) | YAML (+1.1%) | YAML (+31.9%) | XML_COMPACT (+24.0%) | JSON_PRETTY (-1.6%) | JSON_PRETTY (-1.1%) | YAML (-11.3%) | YAML (-11.7%) |
| CSV (+69.1%) | TOON_DEFAULT (+72.6%) | JSON_PRETTY (+18.1%) | XML_COMPACT (+40.9%) | JSON_PRETTY (+30.3%) | XML_COMPACT (-2.2%) | JSON_COMPACT (-2.5%) | XML_COMPACT (-11.4%) | XML_COMPACT (-12.3%) |
| JSON_PRETTY (+89.8%) | YAML (+75.6%) | XML_COMPACT (+26.0%) | TOON_DEFAULT (+47.4%) | XML_PRETTY (+33.2%) | JSON_COMPACT (-2.4%) | XML_COMPACT (-3.1%) | TOON_DEFAULT (-12.0%) | TOON_DEFAULT (-12.6%) |
| XML_PRETTY (+93.4%) | JSON_PRETTY (+99.6%) | XML_PRETTY (+26.7%) | JSON_PRETTY (+51.7%) | YAML (+35.6%) | YAML (-5.9%) | YAML (-6.3%) | JSON_PRETTY (-15.3%) | JSON_PRETTY (-14.8%) |
| XML_COMPACT (+103.5%) | XML_PRETTY (+125.5%) | TOON_DEFAULT (+30.3%) | XML_PRETTY (+67.5%) | CSV (+46.3%) | CSV (-16.9%) | CSV (-17.1%) | XML_PRETTY (-20.2%) | XML_PRETTY (-20.2%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 100% | CSV ≈ 80% | TOON_DEFAULT ≈ 68% | XML_PRETTY ≈ 71% |
| XML_COMPACT (0.0%) | TOON_DEFAULT (-5.6%) | YAML (-0.0%) | YAML (-6.3%) |
| JSON_COMPACT (-1.2%) | XML_PRETTY (-7.4%) | JSON_PRETTY (-1.6%) | TOON_DEFAULT (-7.1%) |
| YAML (-2.4%) | YAML (-9.9%) | XML_PRETTY (-4.8%) | CSV (-11.1%) |
| TOON_DEFAULT (-8.5%) | XML_COMPACT (-11.1%) | XML_COMPACT (-4.8%) | JSON_PRETTY (-11.1%) |
| XML_PRETTY (-9.1%) | JSON_COMPACT (-12.3%) | JSON_COMPACT (-6.4%) | XML_COMPACT (-22.2%) |
| CSV (-26.7%) | JSON_PRETTY (-12.3%) | CSV (-15.9%) | JSON_COMPACT (-31.7%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| XML_COMPACT ≈ 100% | XML_PRETTY ≈ 84% | TOON_DEFAULT ≈ 67% | TOON_DEFAULT ≈ 50% |
| TOON_DEFAULT (-1.5%) | JSON_PRETTY (-0.0%) | JSON_PRETTY (-2.4%) | JSON_COMPACT (-0.8%) |
| XML_PRETTY (-3.0%) | JSON_COMPACT (-3.7%) | XML_COMPACT (-2.4%) | XML_PRETTY (-2.4%) |
| JSON_COMPACT (-7.3%) | TOON_DEFAULT (-9.3%) | YAML (-4.0%) | YAML (-4.0%) |
| JSON_PRETTY (-7.3%) | YAML (-13.6%) | JSON_COMPACT (-5.6%) | JSON_PRETTY (-4.0%) |
| YAML (-9.7%) | XML_COMPACT (-14.8%) | XML_PRETTY (-5.6%) | XML_COMPACT (-7.1%) |
| CSV (-22.4%) | CSV (-24.7%) | CSV (-10.3%) | CSV (-15.1%) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Char/Token | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6968 | 4653 | 11621 | 1.449 | 0.594 | 37.522 | 69.08 | 78.53 | 8027.557 | 3593.110 | 78.34 | 78.53 |
| CSV | opt | 6687 | 9240 | 15927 | 1.432 | 0.395 | 74.519 | 62.90 | 65.83 | 10018.292 | 5909.041 | 66.02 | 65.83 |
| JSON_COMPACT | man | 9255 | 11232 | 20487 | 2.152 | 0.370 | 90.578 | 75.81 | 65.82 | 15530.942 | 4955.725 | 66.59 | 65.82 |
| JSON_COMPACT | opt | 8727 | 9159 | 17886 | 2.118 | 0.433 | 73.863 | 77.42 | 72.41 | 13847.341 | 4038.659 | 72.55 | 72.41 |
| JSON_PRETTY | man | 14254 | 13508 | 27762 | 1.697 | 0.291 | 108.938 | 80.65 | 55.13 | 22390.322 | 5372.011 | 56.47 | 55.13 |
| JSON_PRETTY | opt | 13346 | 10821 | 24167 | 1.683 | 0.324 | 87.269 | 78.23 | 61.70 | 18906.105 | 5261.228 | 61.45 | 61.70 |
| TOON_DEFAULT | man | 7018 | 14468 | 21486 | 1.448 | 0.381 | 116.674 | 79.30 | 66.52 | 17038.002 | 4447.498 | 67.18 | 66.52 |
| TOON_DEFAULT | opt | 11540 | 11937 | 23477 | 1.701 | 0.341 | 96.267 | 79.84 | 63.29 | 18744.037 | 4732.963 | 63.86 | 63.29 |
| XML_COMPACT | man | 11672 | 13648 | 25320 | 2.370 | 0.310 | 110.065 | 78.50 | 58.48 | 19876.200 | 5443.800 | 59.50 | 58.48 |
| XML_COMPACT | opt | 10909 | 11539 | 22448 | 2.345 | 0.346 | 93.059 | 77.69 | 63.49 | 17440.110 | 5008.223 | 64.27 | 63.49 |
| XML_PRETTY | man | 16137 | 9923 | 26060 | 1.937 | 0.303 | 80.027 | 79.03 | 57.42 | 20595.481 | 5464.852 | 58.50 | 57.42 |
| XML_PRETTY | opt | 15076 | 11604 | 26680 | 1.918 | 0.299 | 93.581 | 79.84 | 57.82 | 21301.312 | 5378.688 | 57.92 | 57.82 |
| YAML | man | 12533 | 11181 | 23714 | 1.666 | 0.342 | 90.172 | 81.18 | 63.16 | 19251.296 | 4463.037 | 64.36 | 63.16 |
| YAML | opt | 11742 | 9260 | 21002 | 1.653 | 0.352 | 74.677 | 73.93 | 63.93 | 15526.779 | 5475.221 | 64.32 | 63.93 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6968 | 6687 | -281 | -4.03 | 4653 | 9241 |  +4588 |  +98.60 | 11621 | 15928 |  +4307 |  +37.06 | 69.36 | 62.64 | -6.72 | 78.53 | 65.83 | -12.70 |
| JSON_COMPACT | 9255 | 8727 | -528 | -5.71 | 11232 | 9159 | -2073 | -18.46 | 20487 | 17886 | -2601 | -12.70 | 74.71 | 77.23 |  +2.52 | 65.82 | 72.41 |  +6.59 |
| JSON_PRETTY | 14254 | 13346 | -908 | -6.37 | 13508 | 10821 | -2687 | -19.89 | 27762 | 24167 | -3595 | -12.95 | 78.73 | 78.58 | -0.15 | 55.13 | 61.70 |  +6.57 |
| TOON_DEFAULT | 7018 | 11540 |  +4522 |  +64.43 | 14468 | 11938 | -2530 | -17.49 | 21486 | 23478 |  +1992 |  +9.27 | 78.36 | 79.02 |  +0.66 | 66.52 | 63.29 | -3.23 |
| XML_COMPACT | 11672 | 10909 | -763 | -6.54 | 13648 | 11539 | -2109 | -15.45 | 25320 | 22448 | -2872 | -11.34 | 77.04 | 76.58 | -0.46 | 58.48 | 63.49 |  +5.01 |
| XML_PRETTY | 16137 | 15076 | -1061 | -6.57 | 9923 | 11604 |  +1681 |  +16.94 | 26060 | 26680 |  +620 |  +2.38 | 77.49 | 79.70 |  +2.21 | 57.42 | 57.82 |  +0.40 |
| YAML | 12533 | 11742 | -791 | -6.31 | 11181 | 9260 | -1921 | -17.18 | 23714 | 21002 | -2712 | -11.44 | 79.47 | 73.38 | -6.09 | 63.16 | 63.93 |  +0.77 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 23 | 302.957 | 0.74 | 126926 | 0.037 | 1023.60 | 126949 | 302.994 | 819.03 |
| CSV | opt | 14 | 477.643 | 0.45 | 76723 | 0.120 | 618.73 | 76737 | 477.763 | 495.08 |
| JSON_COMPACT | man | 24 | 385.625 | 0.77 | 94209 | 0.119 | 759.75 | 94233 | 385.744 | 607.96 |
| JSON_COMPACT | opt | 13 | 671.308 | 0.42 | 76265 | 0.120 | 615.04 | 76278 | 671.428 | 492.11 |
| JSON_PRETTY | man | 30 | 475.133 | 0.97 | 108073 | 0.125 | 871.55 | 108103 | 475.258 | 697.44 |
| JSON_PRETTY | opt | 36 | 370.722 | 1.16 | 86112 | 0.126 | 694.45 | 86148 | 370.848 | 555.79 |
| TOON_DEFAULT | man | 26 | 269.923 | 0.84 | 112302 | 0.129 | 905.66 | 112328 | 270.051 | 724.70 |
| TOON_DEFAULT | opt | 20 | 577.000 | 0.65 | 94789 | 0.126 | 764.42 | 94809 | 577.126 | 611.67 |
| XML_COMPACT | man | 19 | 614.316 | 0.61 | 111045 | 0.123 | 895.52 | 111064 | 614.439 | 716.54 |
| XML_COMPACT | opt | 23 | 474.304 | 0.74 | 92319 | 0.125 | 744.51 | 92342 | 474.429 | 595.76 |
| XML_PRETTY | man | 18 | 896.500 | 0.58 | 84964 | 0.117 | 685.19 | 84982 | 896.617 | 548.27 |
| XML_PRETTY | opt | 17 | 886.824 | 0.55 | 87747 | 0.132 | 707.63 | 87764 | 886.956 | 566.22 |
| YAML | man | 21 | 596.810 | 0.68 | 92165 | 0.121 | 743.27 | 92186 | 596.931 | 594.75 |
| YAML | opt | 31 | 378.774 | 1.00 | 69344 | 0.134 | 559.23 | 69375 | 378.908 | 447.58 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) | Total Man (s) | Total Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 23 | 14 | -9 | -39.13 | 126.93 | 76.72 | -50.20 | -39.55 | 126.95 | 76.74 | -50.21 | -39.55 |
| JSON_COMPACT | 24 | 13 | -11 | -45.83 | 94.21 | 76.26 | -17.94 | -19.05 | 94.23 | 76.28 | -17.96 | -19.05 |
| JSON_PRETTY | 30 | 36 |  +6 |  +20.00 | 108.07 | 86.11 | -21.96 | -20.32 | 108.10 | 86.15 | -21.95 | -20.31 |
| TOON_DEFAULT | 26 | 20 | -6 | -23.08 | 112.30 | 94.79 | -17.51 | -15.60 | 112.33 | 115.30 |  +2.97 |  +2.64 |
| XML_COMPACT | 19 | 23 |  +4 |  +21.05 | 111.05 | 92.32 | -18.73 | -16.86 | 111.06 | 92.34 | -18.72 | -16.86 |
| XML_PRETTY | 18 | 17 | -1 | -5.56 | 84.96 | 87.75 |  +2.78 |  +3.28 | 84.98 | 87.76 |  +2.78 |  +3.27 |
| YAML | 21 | 31 |  +10 |  +47.62 | 92.17 | 69.34 | -22.82 | -24.76 | 92.19 | 69.38 | -22.81 | -24.74 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Char/Token | Token/Value | Token/Object | Info/Token |
|---|---|---|---|---|---|
| CSV | man | 1.449 | 10.217 | 224.774 | 0.594 |
| CSV | opt | 1.432 | 10.597 | 215.710 | 0.395 |
| JSON_COMPACT | man | 2.152 | 13.570 | 298.548 | 0.370 |
| JSON_COMPACT | opt | 2.118 | 13.830 | 281.516 | 0.433 |
| JSON_PRETTY | man | 1.697 | 20.900 | 459.806 | 0.291 |
| JSON_PRETTY | opt | 1.683 | 21.151 | 430.516 | 0.324 |
| TOON_DEFAULT | man | 1.448 | 10.290 | 226.387 | 0.381 |
| TOON_DEFAULT | opt | 1.701 | 18.288 | 372.258 | 0.341 |
| XML_COMPACT | man | 2.370 | 17.114 | 376.516 | 0.310 |
| XML_COMPACT | opt | 2.345 | 17.288 | 351.903 | 0.346 |
| XML_PRETTY | man | 1.937 | 23.661 | 520.548 | 0.303 |
| XML_PRETTY | opt | 1.918 | 23.892 | 486.323 | 0.299 |
| YAML | man | 1.666 | 18.377 | 404.290 | 0.342 |
| YAML | opt | 1.653 | 18.609 | 378.774 | 0.352 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.449 | 1.432 | -0.017 | -1.17 | 10.217 | 10.597 |  +0.380 |  +3.72 | 224.774 | 215.710 | -9.064 | -4.03 | 0.594 | 0.395 | -0.199 | -33.50 |
| JSON_COMPACT | 2.152 | 2.118 | -0.034 | -1.58 | 13.570 | 13.830 |  +0.260 |  +1.92 | 298.548 | 281.516 | -17.032 | -5.70 | 0.370 | 0.433 |  +0.063 |  +17.03 |
| JSON_PRETTY | 1.697 | 1.683 | -0.014 | -0.82 | 20.900 | 21.151 |  +0.251 |  +1.20 | 459.806 | 430.516 | -29.290 | -6.37 | 0.291 | 0.324 |  +0.033 |  +11.34 |
| TOON_DEFAULT | 1.448 | 1.701 |  +0.253 |  +17.47 | 10.290 | 18.288 |  +7.998 |  +77.73 | 226.387 | 372.258 |  +145.871 |  +64.43 | 0.381 | 0.341 | -0.040 | -10.63 |
| XML_COMPACT | 2.370 | 2.345 | -0.025 | -1.05 | 17.114 | 17.288 |  +0.174 |  +1.02 | 376.516 | 351.903 | -24.613 | -6.54 | 0.310 | 0.346 |  +0.036 |  +11.61 |
| XML_PRETTY | 1.937 | 1.918 | -0.019 | -0.98 | 23.661 | 23.892 |  +0.231 |  +0.98 | 520.548 | 486.323 | -34.225 | -6.57 | 0.303 | 0.299 | -0.004 | -1.32 |
| YAML | 1.666 | 1.653 | -0.013 | -0.78 | 18.377 | 18.609 |  +0.232 |  +1.26 | 404.290 | 378.774 | -25.516 | -6.31 | 0.342 | 0.352 |  +0.010 |  +2.92 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| CSV | man | 11621 | 8028 | 3593 | 69.08 | 69.36 | 78.34 | 78.53 |
| CSV | opt | 15927 | 10018 | 5909 | 62.90 | 62.64 | 66.02 | 65.83 |
| JSON_COMPACT | man | 20487 | 15531 | 4956 | 75.81 | 74.71 | 66.59 | 65.82 |
| JSON_COMPACT | opt | 17886 | 13847 | 4039 | 77.42 | 77.23 | 72.55 | 72.41 |
| JSON_PRETTY | man | 27762 | 22390 | 5372 | 80.65 | 78.73 | 56.47 | 55.13 |
| JSON_PRETTY | opt | 24167 | 18906 | 5261 | 78.23 | 78.58 | 61.45 | 61.70 |
| TOON_DEFAULT | man | 21486 | 17038 | 4447 | 79.30 | 78.36 | 67.18 | 66.52 |
| TOON_DEFAULT | opt | 23477 | 18744 | 4733 | 79.84 | 79.02 | 63.86 | 63.29 |
| XML_COMPACT | man | 25320 | 19876 | 5444 | 78.50 | 77.04 | 59.50 | 58.48 |
| XML_COMPACT | opt | 22448 | 17440 | 5008 | 77.69 | 76.58 | 64.27 | 63.49 |
| XML_PRETTY | man | 26060 | 20595 | 5465 | 79.03 | 77.49 | 58.50 | 57.42 |
| XML_PRETTY | opt | 26680 | 21301 | 5379 | 79.84 | 79.70 | 57.92 | 57.82 |
| YAML | man | 23714 | 19251 | 4463 | 81.18 | 79.47 | 64.36 | 63.16 |
| YAML | opt | 21002 | 15527 | 5475 | 73.93 | 73.38 | 64.32 | 63.93 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 11621 | 15928 |  +4307 |  +37.06 | 8028 | 10019 |  +1991 |  +24.80 | 3593 | 5909 |  +2316 |  +64.46 | 69.08 | 62.90 | -6.18 | 78.34 | 66.017 | -12.32 | -15.73 |
| JSON_COMPACT | 20487 | 17886 | -2601 | -12.69 | 15531 | 13847 | -1684 | -10.84 | 4956 | 4039 | -917 | -18.50 | 75.81 | 77.42 |  +1.61 | 66.59 | 72.545 |  +5.95 |  +8.94 |
| JSON_PRETTY | 27762 | 24167 | -3595 | -12.95 | 22390 | 18906 | -3484 | -15.56 | 5372 | 5261 | -111 | -2.06 | 80.65 | 78.23 | -2.42 | 56.47 | 61.453 |  +4.98 |  +8.82 |
| TOON_DEFAULT | 21486 | 23478 |  +1992 |  +9.27 | 17038 | 18744 |  +1706 |  +10.01 | 4447 | 4732 |  +285 |  +6.42 | 79.30 | 79.84 |  +0.54 | 67.18 | 63.86150000000001 | -3.32 | -4.94 |
| XML_COMPACT | 25320 | 22448 | -2872 | -11.34 | 19876 | 17440 | -2436 | -12.26 | 5444 | 5008 | -436 | -8.00 | 78.50 | 77.69 | -0.81 | 59.50 | 64.266 |  +4.76 |  +8.01 |
| XML_PRETTY | 26060 | 26680 |  +620 |  +2.38 | 20595 | 21301 |  +706 |  +3.43 | 5465 | 5379 | -86 | -1.58 | 79.03 | 79.84 |  +0.81 | 58.50 | 57.916 | -0.58 | -1.00 |
| YAML | 23714 | 21002 | -2712 | -11.44 | 19251 | 15526 | -3725 | -19.35 | 4463 | 5475 |  +1012 |  +22.68 | 81.18 | 73.93 | -7.25 | 64.36 | 64.318 | -0.04 | -0.06 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| CSV | man | 86 | 38 | 0 | 69.08 |
| CSV | opt | 78 | 46 | 0 | 62.90 |
| JSON_COMPACT | man | 94 | 30 | 0 | 75.81 |
| JSON_COMPACT | opt | 96 | 28 | 0 | 77.42 |
| JSON_PRETTY | man | 100 | 24 | 0 | 80.65 |
| JSON_PRETTY | opt | 97 | 27 | 0 | 78.23 |
| TOON_DEFAULT | man | 98 | 26 | 0 | 79.30 |
| TOON_DEFAULT | opt | 99 | 25 | 0 | 79.84 |
| XML_COMPACT | man | 97 | 27 | 0 | 78.50 |
| XML_COMPACT | opt | 96 | 28 | 0 | 77.69 |
| XML_PRETTY | man | 98 | 26 | 0 | 79.03 |
| XML_PRETTY | opt | 99 | 25 | 0 | 79.84 |
| YAML | man | 101 | 23 | 0 | 81.18 |
| YAML | opt | 92 | 32 | 0 | 73.93 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 86 | 78 | -8 | -9.30 | 38 | 46 |  +8 |  +21.05 | 0 | 0 | 0 | 0.00 | 69.08 | 62.90 | -6.18 |
| JSON_COMPACT | 94 | 96 |  +2 |  +2.13 | 30 | 28 | -2 | -6.67 | 0 | 0 | 0 | 0.00 | 75.81 | 77.42 |  +1.61 |
| JSON_PRETTY | 100 | 97 | -3 | -3.00 | 24 | 27 |  +3 |  +12.50 | 0 | 0 | 0 | 0.00 | 80.65 | 78.23 | -2.42 |
| TOON_DEFAULT | 98 | 99 |  +1 |  +1.02 | 26 | 25 | -1 | -3.85 | 0 | 0 | 0 | 0.00 | 79.30 | 79.84 |  +0.54 |
| XML_COMPACT | 97 | 96 | -1 | -1.03 | 27 | 28 |  +1 |  +3.70 | 0 | 0 | 0 | 0.00 | 78.50 | 77.69 | -0.81 |
| XML_PRETTY | 98 | 99 |  +1 |  +1.02 | 26 | 25 | -1 | -3.85 | 0 | 0 | 0 | 0.00 | 79.03 | 79.84 |  +0.81 |
| YAML | 101 | 92 | -9 | -8.91 | 23 | 32 |  +9 |  +39.13 | 0 | 0 | 0 | 0.00 | 81.18 | 73.93 | -7.25 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| CSV | man | 69.08 | 73.33 | 80.25 | 52.38 | 60.32 |
| CSV | opt | 62.90 | 77.57 | 59.26 | 57.14 | 34.92 |
| JSON_COMPACT | man | 75.81 | 98.79 | 67.90 | 61.90 | 39.68 |
| JSON_COMPACT | opt | 77.42 | 92.73 | 80.25 | 61.90 | 49.21 |
| JSON_PRETTY | man | 80.65 | 100.00 | 67.90 | 66.66 | 60.32 |
| JSON_PRETTY | opt | 78.23 | 92.73 | 83.95 | 65.08 | 46.03 |
| TOON_DEFAULT | man | 79.30 | 91.51 | 74.69 | 68.26 | 64.28 |
| TOON_DEFAULT | opt | 79.84 | 98.48 | 74.69 | 67.46 | 50.00 |
| XML_COMPACT | man | 78.50 | 100.00 | 69.14 | 63.49 | 49.21 |
| XML_COMPACT | opt | 77.69 | 100.00 | 69.14 | 65.08 | 42.86 |
| XML_PRETTY | man | 79.03 | 90.91 | 72.84 | 63.49 | 71.43 |
| XML_PRETTY | opt | 79.84 | 96.97 | 83.95 | 61.90 | 47.62 |
| YAML | man | 81.18 | 97.57 | 70.37 | 68.25 | 65.08 |
| YAML | opt | 73.93 | 90.30 | 70.37 | 63.49 | 46.03 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 73.33 | 77.57 |  +4.24 |
| JSON_COMPACT | 98.79 | 92.73 | -6.06 |
| JSON_PRETTY | 100.00 | 92.73 | -7.27 |
| TOON_DEFAULT | 91.51 | 98.48 |  +6.97 |
| XML_COMPACT | 100.00 | 100.00 | 0.00 |
| XML_PRETTY | 90.91 | 96.97 |  +6.06 |
| YAML | 97.57 | 90.30 | -7.27 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 80.25 | 59.26 | -20.99 |
| JSON_COMPACT | 67.90 | 80.25 |  +12.35 |
| JSON_PRETTY | 67.90 | 83.95 |  +16.05 |
| TOON_DEFAULT | 74.69 | 74.69 |  +0.00 |
| XML_COMPACT | 69.14 | 69.14 | 0.00 |
| XML_PRETTY | 72.84 | 83.95 |  +11.11 |
| YAML | 70.37 | 70.37 | 0.00 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 52.38 | 57.14 |  +4.76 |
| JSON_COMPACT | 61.90 | 61.90 |  +0.00 |
| JSON_PRETTY | 66.66 | 65.08 | -1.58 |
| TOON_DEFAULT | 68.26 | 67.46 | -0.80 |
| XML_COMPACT | 63.49 | 65.08 |  +1.59 |
| XML_PRETTY | 63.49 | 61.90 | -1.59 |
| YAML | 68.25 | 63.49 | -4.76 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 60.32 | 34.92 | -25.40 |
| JSON_COMPACT | 39.68 | 49.21 |  +9.52 |
| JSON_PRETTY | 60.32 | 46.03 | -14.29 |
| TOON_DEFAULT | 64.28 | 50.00 | -14.29 |
| XML_COMPACT | 49.21 | 42.86 | -6.35 |
| XML_PRETTY | 71.43 | 47.62 | -23.81 |
| YAML | 65.08 | 46.03 | -19.04 |

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: on
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
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_flat_all_formats_and_variants_off\BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_flat_all_formats_and_variants_on\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_off\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_on_verify\BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)