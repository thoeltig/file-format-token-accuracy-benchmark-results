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
   - **Output Before Write Tokens**: The output tokens which the model needed for reading the provided files and instructions.
   - **Output Write Tokens**: The output tokens the model used to create the output and write the answers file.

## 2. Results

### 2.1 TLDR: Token Efficiency Analysis

*Note: All columns ranked best-to-worst. ↑ = lower value is better (ascending). ↓ = higher value is better (descending).*

#### 2.1.1 Best results

- Lowest total token cost:
   - Optional: CSV 17689 tokens
   - Mandatory: CSV 17340 tokens
- Lowest read token cost:
   - Optional: CSV 6728 tokens
   - Mandatory: CSV 7022 tokens
- Lowest output token cost:
   - Optional: TOON_DEFAULT 8984 tokens
   - Mandatory: XML_COMPACT 6976 tokens
- Lowest output token cost drift:
   - Optional: YAML ↓ -17.42% ↑ 15.86%
   - Mandatory: JSON_PRETTY ↓ -7.29% ↑ 5.35%
- Highest accuracy:
   - Optional: YAML 80.38%
   - Mandatory: JSON_PRETTY 82.53%
- Lowest accuracy drift:
   - Optional: YAML ↓ -3.68% ↑ 3.33%
   - Mandatory: JSON_PRETTY ↓ -1.31% ↑ 1.62%
- Most useful read tokens:
   - Optional: XML_PRETTY 11622 / 15117 tokens
   - Mandatory: XML_PRETTY 12400 / 16186 tokens
- Most useful output tokens:
   - Optional: YAML 10473 / 13029 tokens
   - Mandatory: JSON_COMPACT 10525 / 13489 tokens
- Highest read efficiency (%/token):
   - Optional: JSON_COMPACT 77.14
   - Mandatory: TOON_DEFAULT 82.35
- Highest output efficiency (%/token):
   - Optional: TOON_DEFAULT 67.33
   - Mandatory: XML_COMPACT 64.81
- Lowest delta (optional-mandatory):
   - Read tokens: CSV -294 tokens
   - Output tokens: CSV 643 tokens
   - Accuracy: XML_PRETTY 0.27%
   - Read efficiency: JSON_PRETTY 0.23
   - Output efficiency: CSV -4.42

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: YAML 24820 tokens
   - Mandatory: XML_PRETTY 26786 tokens
- Highest read token cost:
   - Optional: XML_PRETTY 15117 tokens
   - Mandatory: XML_PRETTY 16186 tokens
- Highest output token cost:
   - Optional: YAML 13029 tokens
   - Mandatory: JSON_COMPACT 13489 tokens
- Highest output token drift:
   - Optional: JSON_COMPACT ↓ -44.06% ↑ 59.10%
   - Mandatory: CSV ↓ -16.06% ↑ 39.46%
- Lowest accuracy:
   - Optional: CSV 65.48%
   - Mandatory: CSV 67.58%
- Highest accuracy drift:
   - Optional: CSV ↓ -16.25% ↑ 13.30%
   - Mandatory: TOON_DEFAULT ↓ -22.37% ↑ 8.04%
- Most wasted read tokens:
   - Optional: XML_PRETTY 3495 / 15117 tokens
   - Mandatory: XML_PRETTY 3786 / 16186 tokens
- Most wasted output tokens:
   - Optional: CSV 3784 / 10961 tokens
   - Mandatory: CSV 3345 / 10318 tokens
- Lowest read efficiency (%/token):
   - Optional: XML_PRETTY 57.23
   - Mandatory: XML_PRETTY 53.66
- Lowest output efficiency (%/token):
   - Optional: CSV 57.49
   - Mandatory: JSON_COMPACT 54.67
- Highest delta (optional-mandatory):
   - Read tokens: TOON_DEFAULT 4527 tokens
   - Output tokens: XML_COMPACT 4290 tokens
   - Accuracy: XML_COMPACT 5.11%
   - Read efficiency: TOON_DEFAULT -15.02
   - Output efficiency: JSON_COMPACT 17.06

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 76s | CSV ≈ 7022 | CSV ≈ 241 | XML_COMPACT ≈ 6673 | XML_COMPACT ≈ 6976 | CSV ≈ 17340 | JSON_PRETTY ≈ 83% | TOON_DEFAULT ≈ 82 | XML_COMPACT ≈ 81 | TOON_DEFAULT ≈ 79 |
| XML_COMPACT (+5.8%) | TOON_DEFAULT (+2.3%) | TOON_DEFAULT (+7.0%) | YAML (+35.9%) | YAML (+34.4%) | TOON_DEFAULT (+8.4%) | JSON_COMPACT (-4.5%) | JSON_COMPACT (-7.1%) | YAML (-11.5%) | CSV (-2.4%) |
| CSV (+6.6%) | JSON_COMPACT (+32.4%) | JSON_PRETTY (+25.4%) | CSV (+51.0%) | CSV (+47.9%) | XML_COMPACT (+9.1%) | TOON_DEFAULT (-5.7%) | CSV (-7.3%) | XML_PRETTY (-17.7%) | XML_COMPACT (-3.6%) |
| XML_PRETTY (+13.9%) | XML_COMPACT (+70.2%) | YAML (+25.4%) | XML_PRETTY (+54.3%) | XML_PRETTY (+52.0%) | YAML (+26.6%) | XML_PRETTY (-5.9%) | XML_COMPACT (-21.3%) | JSON_PRETTY (-20.4%) | YAML (-13.6%) |
| TOON_DEFAULT (+25.2%) | YAML (+79.1%) | XML_COMPACT (+25.8%) | TOON_DEFAULT (+70.2%) | TOON_DEFAULT (+66.5%) | JSON_COMPACT (+31.4%) | YAML (-6.7%) | YAML (-21.6%) | TOON_DEFAULT (-23.2%) | JSON_COMPACT (-14.9%) |
| JSON_PRETTY (+26.8%) | JSON_PRETTY (+103.8%) | JSON_COMPACT (+26.4%) | JSON_PRETTY (+75.0%) | JSON_PRETTY (+71.7%) | JSON_PRETTY (+51.6%) | XML_COMPACT (-9.1%) | JSON_PRETTY (-22.6%) | CSV (-23.9%) | JSON_PRETTY (-25.0%) |
| JSON_COMPACT (+40.5%) | XML_PRETTY (+130.5%) | XML_PRETTY (+26.5%) | JSON_COMPACT (+97.6%) | JSON_COMPACT (+93.4%) | XML_PRETTY (+54.5%) | CSV (-15.0%) | XML_PRETTY (-34.8%) | JSON_COMPACT (-32.8%) | XML_PRETTY (-32.2%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 74s | CSV ≈ 6728 | JSON_PRETTY ≈ 231 | TOON_DEFAULT ≈ 8675 | TOON_DEFAULT ≈ 8984 | CSV ≈ 17689 | YAML ≈ 80% | JSON_COMPACT ≈ 77 | TOON_DEFAULT ≈ 74 | JSON_COMPACT ≈ 80 |
| XML_PRETTY (+0.3%) | JSON_COMPACT (+30.3%) | XML_PRETTY (+30.7%) | XML_PRETTY (+2.7%) | XML_PRETTY (+2.5%) | JSON_COMPACT (+3.6%) | JSON_PRETTY (-1.7%) | CSV (-1.7%) | XML_PRETTY (-0.5%) | CSV (-7.2%) |
| JSON_COMPACT (+2.1%) | XML_COMPACT (+66.2%) | CSV (+31.4%) | JSON_COMPACT (+6.7%) | JSON_COMPACT (+6.4%) | TOON_DEFAULT (+17.0%) | XML_COMPACT (-1.9%) | XML_COMPACT (-8.2%) | JSON_COMPACT (-2.9%) | TOON_DEFAULT (-9.9%) |
| JSON_PRETTY (+14.8%) | TOON_DEFAULT (+74.0%) | YAML (+31.6%) | JSON_PRETTY (+17.3%) | JSON_PRETTY (+15.9%) | XML_COMPACT (+26.9%) | XML_PRETTY (-3.5%) | YAML (-9.0%) | JSON_PRETTY (-6.2%) | XML_COMPACT (-14.6%) |
| CSV (+16.9%) | YAML (+75.3%) | JSON_COMPACT (+31.9%) | CSV (+22.9%) | CSV (+22.0%) | JSON_PRETTY (+34.6%) | JSON_COMPACT (-3.8%) | TOON_DEFAULT (-12.7%) | XML_COMPACT (-11.7%) | JSON_PRETTY (-19.8%) |
| XML_COMPACT (+18.4%) | JSON_PRETTY (+99.1%) | XML_COMPACT (+33.8%) | XML_COMPACT (+26.3%) | XML_COMPACT (+25.4%) | XML_PRETTY (+37.5%) | TOON_DEFAULT (-4.5%) | JSON_PRETTY (-17.1%) | YAML (-20.9%) | YAML (-22.3%) |
| YAML (+40.4%) | XML_PRETTY (+124.7%) | TOON_DEFAULT (+33.8%) | YAML (+46.7%) | YAML (+45.0%) | YAML (+40.3%) | CSV (-14.9%) | XML_PRETTY (-25.8%) | CSV (-22.2%) | XML_PRETTY (-23.4%) |


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
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7022 | 10318 | 17340 | 1.438 | 81.271 | 67.58 | 4745.468 | 2276.532 | 6973.040 | 3345.160 | 76.34 | 61.91 | 77.27 |
| CSV | opt | 6728 | 10961 | 17689 | 1.423 | 85.950 | 65.48 | 4405.494 | 2322.506 | 7177.525 | 3783.875 | 75.80 | 57.49 | 74.70 |
| JSON_COMPACT | man | 9296 | 13489 | 22785 | 2.143 | 106.327 | 78.03 | 7253.669 | 2042.331 | 10525.077 | 2963.423 | 76.46 | 54.67 | 67.33 |
| JSON_COMPACT | opt | 8768 | 9558 | 18326 | 2.108 | 74.621 | 76.61 | 6717.165 | 2050.835 | 7322.231 | 2235.569 | 77.14 | 71.72 | 80.47 |
| JSON_PRETTY | man | 14311 | 11977 | 26288 | 1.691 | 94.153 | 82.53 | 11810.868 | 2500.132 | 9884.343 | 2092.324 | 63.74 | 64.76 | 59.38 |
| JSON_PRETTY | opt | 13395 | 10411 | 23806 | 1.677 | 82.097 | 78.71 | 10543.204 | 2851.796 | 8194.498 | 2216.502 | 63.96 | 69.28 | 64.57 |
| TOON_DEFAULT | man | 7182 | 11615 | 18797 | 1.415 | 91.593 | 76.88 | 5521.522 | 1660.478 | 8929.612 | 2685.388 | 82.35 | 62.47 | 79.17 |
| TOON_DEFAULT | opt | 11709 | 8984 | 20693 | 1.677 | 69.959 | 75.90 | 8887.131 | 2821.869 | 6818.983 | 2165.184 | 67.33 | 73.86 | 72.47 |
| XML_COMPACT | man | 11950 | 6976 | 18926 | 2.315 | 53.817 | 73.39 | 8770.105 | 3179.895 | 5119.686 | 1856.314 | 64.81 | 81.33 | 76.32 |
| XML_COMPACT | opt | 11181 | 11266 | 22447 | 2.288 | 88.360 | 78.50 | 8777.085 | 2403.915 | 8843.549 | 2422.118 | 70.82 | 65.20 | 68.73 |
| XML_PRETTY | man | 16186 | 10600 | 26786 | 1.931 | 83.032 | 76.61 | 12400.095 | 3785.905 | 8120.915 | 2479.418 | 53.66 | 66.94 | 53.66 |
| XML_PRETTY | opt | 15117 | 9210 | 24327 | 1.913 | 71.836 | 76.88 | 11621.950 | 3495.050 | 7080.392 | 2129.275 | 57.23 | 73.51 | 61.64 |
| YAML | man | 12574 | 9373 | 21947 | 1.661 | 73.153 | 75.81 | 9532.349 | 3041.651 | 7105.419 | 2267.248 | 64.53 | 72.02 | 68.44 |
| YAML | opt | 11791 | 13029 | 24820 | 1.646 | 102.621 | 80.38 | 9477.606 | 2313.394 | 10472.710 | 2556.290 | 70.21 | 58.42 | 62.53 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7022 | 6728 | -294 | -4.19 | 241 | 304 |  +580 |  +240.66 | 10078 | 10658 |  +643 |  +6.38 | 10318 | 10961 |  +643 |  +6.23 | 17340 | 17689 |  +349 |  +2.01 |
| JSON_COMPACT | 9296 | 8768 | -528 | -5.68 | 304 | 305 | -3931 | -1293.09 | 13185 | 9254 | -3931 | -29.81 | 13489 | 9558 | -3931 | -29.14 | 22785 | 18326 | -4459 | -19.57 |
| JSON_PRETTY | 14311 | 13395 | -916 | -6.40 | 302 | 231 | -1495 | -495.03 | 11675 | 10180 | -1566 | -13.41 | 11977 | 10411 | -1566 | -13.08 | 26288 | 23806 | -2482 | -9.44 |
| TOON_DEFAULT | 7182 | 11709 |  +4527 |  +63.03 | 258 | 310 | -2682 | -1039.53 | 11358 | 8676 | -2631 | -23.16 | 11615 | 8984 | -2631 | -22.65 | 18797 | 20693 |  +1896 |  +10.09 |
| XML_COMPACT | 11950 | 11181 | -769 | -6.44 | 303 | 309 |  +4283 |  +1413.53 | 6673 | 10956 |  +4290 |  +64.29 | 6976 | 11266 |  +4290 |  +61.50 | 18926 | 22447 |  +3521 |  +18.60 |
| XML_PRETTY | 16186 | 15117 | -1069 | -6.60 | 304 | 302 | -1388 | -456.58 | 10296 | 8908 | -1391 | -13.51 | 10600 | 9209 | -1391 | -13.12 | 26786 | 24326 | -2460 | -9.18 |
| YAML | 12574 | 11791 | -783 | -6.23 | 302 | 304 |  +3654 |  +1209.93 | 9071 | 12725 |  +3656 |  +40.30 | 9373 | 13029 |  +3656 |  +39.01 | 21947 | 24820 |  +2873 |  +13.09 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 28 | 250.786 | 0.90 | 34797 | 0.290 | 280.62 | 34825 | 251.076 | 224.68 |
| CSV | opt | 10 | 672.800 | 0.32 | 36882 | 0.289 | 297.43 | 36892 | 673.089 | 238.01 |
| JSON_COMPACT | man | 36 | 258.222 | 1.16 | 35225 | 0.374 | 284.07 | 35261 | 258.596 | 227.49 |
| JSON_COMPACT | opt | 9 | 974.222 | 0.29 | 35563 | 0.260 | 286.80 | 35572 | 974.482 | 229.50 |
| JSON_PRETTY | man | 13 | 1100.846 | 0.42 | 35817 | 0.326 | 288.85 | 35830 | 1101.172 | 231.16 |
| JSON_PRETTY | opt | 15 | 893.000 | 0.48 | 37347 | 0.273 | 301.19 | 37362 | 893.273 | 241.05 |
| TOON_DEFAULT | man | 27 | 266.000 | 0.87 | 38129 | 0.298 | 307.49 | 38156 | 266.298 | 246.17 |
| TOON_DEFAULT | opt | 27 | 433.667 | 0.87 | 37047 | 0.237 | 298.77 | 37074 | 433.904 | 239.19 |
| XML_COMPACT | man | 16 | 746.875 | 0.52 | 47309 | 0.141 | 381.53 | 47325 | 747.016 | 305.32 |
| XML_COMPACT | opt | 15 | 745.400 | 0.48 | 35007 | 0.313 | 282.32 | 35022 | 745.713 | 225.95 |
| XML_PRETTY | man | 18 | 899.222 | 0.58 | 36393 | 0.283 | 293.49 | 36411 | 899.505 | 234.91 |
| XML_PRETTY | opt | 9 | 1679.667 | 0.29 | 36231 | 0.246 | 292.18 | 36240 | 1679.913 | 233.80 |
| YAML | man | 11 | 1143.091 | 0.35 | 34805 | 0.261 | 280.69 | 34816 | 1143.352 | 224.62 |
| YAML | opt | 8 | 1473.875 | 0.26 | 37991 | 0.335 | 306.38 | 37999 | 1474.210 | 245.15 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 28 | 10 | -18 | -64.29 | 34.80 | 36.88 |  +2.08 |  +5.99 | 34.83 | 36.89 |  +2.07 |  +5.93 |
| JSON_COMPACT | 36 | 9 | -27 | -75.00 | 35.22 | 35.56 |  +0.34 |  +0.96 | 35.26 | 35.57 |  +0.31 |  +0.88 |
| JSON_PRETTY | 13 | 15 |  +2 |  +15.38 | 35.82 | 37.35 |  +1.53 |  +4.27 | 35.83 | 37.36 |  +1.53 |  +4.28 |
| TOON_DEFAULT | 27 | 27 | 0 | 0.00 | 38.13 | 37.05 | -1.08 | -2.84 | 38.16 | 37.07 | -1.08 | -2.83 |
| XML_COMPACT | 16 | 15 | -1 | -6.25 | 47.31 | 35.01 | -12.30 | -26.00 | 47.33 | 35.02 | -12.30 | -26.00 |
| XML_PRETTY | 18 | 9 | -9 | -50.00 | 36.39 | 36.23 | -0.16 | -0.45 | 36.41 | 36.24 | -0.17 | -0.47 |
| YAML | 11 | 8 | -3 | -27.27 | 34.80 | 37.99 |  +3.19 |  +9.15 | 34.82 | 38.00 |  +3.18 |  +9.14 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token |
|---|---|---|---|---|---|---|---|
| CSV | man | 1.438 | 10.296 | 226.516 | 0.962 | 0.655 | 0.390 |
| CSV | opt | 1.423 | 10.662 | 217.032 | 0.973 | 0.597 | 0.370 |
| JSON_COMPACT | man | 2.143 | 13.630 | 299.871 | 0.839 | 0.578 | 0.342 |
| JSON_COMPACT | opt | 2.108 | 13.895 | 282.839 | 0.874 | 0.802 | 0.418 |
| JSON_PRETTY | man | 1.691 | 20.984 | 461.645 | 0.577 | 0.689 | 0.314 |
| JSON_PRETTY | opt | 1.677 | 21.228 | 432.097 | 0.588 | 0.756 | 0.331 |
| TOON_DEFAULT | man | 1.415 | 10.531 | 231.677 | 1.070 | 0.663 | 0.409 |
| TOON_DEFAULT | opt | 1.677 | 18.556 | 377.710 | 0.648 | 0.870 | 0.369 |
| XML_COMPACT | man | 2.315 | 17.522 | 385.484 | 0.614 | 1.052 | 0.388 |
| XML_COMPACT | opt | 2.288 | 17.719 | 360.677 | 0.702 | 0.697 | 0.350 |
| XML_PRETTY | man | 1.931 | 23.733 | 522.129 | 0.473 | 0.723 | 0.286 |
| XML_PRETTY | opt | 1.913 | 23.957 | 487.645 | 0.509 | 0.835 | 0.316 |
| YAML | man | 1.661 | 18.437 | 405.613 | 0.603 | 0.809 | 0.345 |
| YAML | opt | 1.646 | 18.686 | 380.355 | 0.682 | 0.617 | 0.324 |

#### 2.5.2 Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.438 | 1.423 | -0.015 | -1.04 | 10.296 | 10.662 |  +0.366 |  +3.55 | 226.516 | 217.032 | -9.484 | -4.19 | 0.962 | 0.973 |  +0.011 |  +1.14 | 0.655 | 0.597 | -0.058 | -8.85 | 0.390 | 0.370 | -0.020 | -5.13 |
| JSON_COMPACT | 2.143 | 2.108 | -0.035 | -1.63 | 13.630 | 13.895 |  +0.265 |  +1.94 | 299.871 | 282.839 | -17.032 | -5.68 | 0.839 | 0.874 |  +0.035 |  +4.17 | 0.578 | 0.802 |  +0.224 |  +38.75 | 0.342 | 0.418 |  +0.076 |  +22.22 |
| JSON_PRETTY | 1.691 | 1.677 | -0.014 | -0.83 | 20.984 | 21.228 |  +0.244 |  +1.16 | 461.645 | 432.097 | -29.548 | -6.40 | 0.577 | 0.588 |  +0.011 |  +1.91 | 0.689 | 0.756 |  +0.067 |  +9.72 | 0.314 | 0.331 |  +0.017 |  +5.41 |
| TOON_DEFAULT | 1.415 | 1.677 |  +0.262 |  +18.52 | 10.531 | 18.556 |  +8.025 |  +76.20 | 231.677 | 377.710 |  +146.033 |  +63.03 | 1.070 | 0.648 | -0.422 | -39.44 | 0.663 | 0.870 |  +0.207 |  +31.22 | 0.409 | 0.369 | -0.040 | -9.89 |
| XML_COMPACT | 2.315 | 2.288 | -0.027 | -1.17 | 17.522 | 17.719 |  +0.197 |  +1.12 | 385.484 | 360.677 | -24.807 | -6.44 | 0.614 | 0.702 |  +0.088 |  +14.33 | 1.052 | 0.697 | -0.355 | -33.75 | 0.388 | 0.350 | -0.038 | -9.79 |
| XML_PRETTY | 1.931 | 1.913 | -0.018 | -0.93 | 23.733 | 23.957 |  +0.224 |  +0.94 | 522.129 | 487.645 | -34.484 | -6.60 | 0.473 | 0.509 |  +0.036 |  +7.61 | 0.723 | 0.835 |  +0.112 |  +15.49 | 0.286 | 0.316 |  +0.030 |  +10.49 |
| YAML | 1.661 | 1.646 | -0.015 | -0.90 | 18.437 | 18.686 |  +0.249 |  +1.35 | 405.613 | 380.355 | -25.258 | -6.23 | 0.603 | 0.682 |  +0.079 |  +13.10 | 0.809 | 0.617 | -0.192 | -23.73 | 0.345 | 0.324 | -0.021 | -6.09 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7022 | 4745 | 2277 | 10318 | 6973 | 3345 | 17340 | 11719 | 5622 | 67.58 | 76.34 | 61.91 | 77.27 | 67.37 | 76.20 | 61.76 | 77.13 |
| CSV | opt | 6728 | 4405 | 2323 | 10961 | 7178 | 3784 | 17689 | 11583 | 6106 | 65.48 | 75.80 | 57.49 | 74.70 | 65.49 | 75.81 | 57.49 | 74.70 |
| JSON_COMPACT | man | 9296 | 7254 | 2042 | 13489 | 10525 | 2963 | 22785 | 17779 | 5006 | 78.03 | 76.46 | 54.67 | 67.33 | 77.71 | 76.24 | 54.44 | 67.11 |
| JSON_COMPACT | opt | 8768 | 6717 | 2051 | 9558 | 7322 | 2236 | 18326 | 14039 | 4286 | 76.61 | 77.14 | 71.72 | 80.47 | 75.34 | 76.25 | 70.83 | 79.58 |
| JSON_PRETTY | man | 14311 | 11811 | 2500 | 11977 | 9884 | 2092 | 26288 | 21695 | 4592 | 82.53 | 63.74 | 64.76 | 59.38 | 82.08 | 63.42 | 64.44 | 59.07 |
| JSON_PRETTY | opt | 13395 | 10543 | 2852 | 10411 | 8194 | 2217 | 23806 | 18738 | 5068 | 78.71 | 63.96 | 69.28 | 64.57 | 78.15 | 63.57 | 68.88 | 64.18 |
| TOON_DEFAULT | man | 7182 | 5522 | 1660 | 11615 | 8930 | 2685 | 18797 | 14451 | 4346 | 76.88 | 82.35 | 62.47 | 79.17 | 76.63 | 82.17 | 62.29 | 78.99 |
| TOON_DEFAULT | opt | 11709 | 8887 | 2822 | 8984 | 6819 | 2165 | 20693 | 15706 | 4987 | 75.90 | 67.33 | 73.86 | 72.47 | 75.19 | 66.83 | 73.36 | 71.98 |
| XML_COMPACT | man | 11950 | 8770 | 3180 | 6976 | 5120 | 1856 | 18926 | 13890 | 5036 | 73.39 | 64.81 | 81.33 | 76.32 | 72.03 | 63.86 | 80.38 | 75.36 |
| XML_COMPACT | opt | 11181 | 8777 | 2404 | 11266 | 8844 | 2422 | 22447 | 17621 | 4826 | 78.50 | 70.82 | 65.20 | 68.73 | 78.42 | 70.77 | 65.15 | 68.68 |
| XML_PRETTY | man | 16186 | 12400 | 3786 | 10600 | 8121 | 2479 | 26786 | 20521 | 6265 | 76.61 | 53.66 | 66.94 | 53.66 | 75.61 | 52.96 | 66.24 | 52.96 |
| XML_PRETTY | opt | 15117 | 11622 | 3495 | 9210 | 7080 | 2129 | 24327 | 18702 | 5624 | 76.88 | 57.23 | 73.51 | 61.64 | 76.51 | 56.97 | 73.25 | 61.38 |
| YAML | man | 12574 | 9532 | 3042 | 9373 | 7105 | 2267 | 21947 | 16638 | 5309 | 75.81 | 64.53 | 72.02 | 68.44 | 74.13 | 63.35 | 70.84 | 67.26 |
| YAML | opt | 11791 | 9478 | 2313 | 13029 | 10473 | 2556 | 24820 | 19950 | 4870 | 80.38 | 70.21 | 58.42 | 62.53 | 79.84 | 69.83 | 58.04 | 62.15 |

#### 2.6.2 Read Tokens Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7022 | 6728 | -294 | -4.19 | 4745 | 4405 | -340 | -7.16 | 2277 | 2323 |  +46 |  +2.02 | 67.58 | 65.48 | -2.10 | -3.11 | 76.34 | 75.80 | -0.54 | -0.71 | 67.37 | 65.49 | -1.88 | -2.79 | 76.20 | 75.81 | -0.39 | -0.51 |
| JSON_COMPACT | 9296 | 8768 | -528 | -5.68 | 7254 | 6717 | -537 | -7.40 | 2042 | 2051 |  +9 |  +0.42 | 78.03 | 76.61 | -1.42 | -1.82 | 76.46 | 77.14 |  +0.68 |  +0.89 | 77.71 | 75.34 | -2.37 | -3.05 | 76.24 | 76.25 |  +0.01 |  +0.02 |
| JSON_PRETTY | 14311 | 13395 | -916 | -6.40 | 11811 | 10543 | -1268 | -10.73 | 2500 | 2852 |  +352 |  +14.07 | 82.53 | 78.71 | -3.82 | -4.63 | 63.74 | 63.96 |  +0.23 |  +0.35 | 82.08 | 78.15 | -3.93 | -4.79 | 63.42 | 63.57 |  +0.15 |  +0.23 |
| TOON_DEFAULT | 7182 | 11709 |  +4527 |  +63.03 | 5522 | 8888 |  +3366 |  +60.95 | 1660 | 2821 |  +1161 |  +69.96 | 76.88 | 75.90 | -0.98 | -1.27 | 82.35 | 67.33 | -15.02 | -18.23 | 76.63 | 75.19 | -1.44 | -1.88 | 82.17 | 66.83 | -15.34 | -18.66 |
| XML_COMPACT | 11950 | 11181 | -769 | -6.44 | 8770 | 8777 |  +7 |  +0.08 | 3180 | 2404 | -776 | -24.40 | 73.39 | 78.50 |  +5.11 |  +6.96 | 64.81 | 70.82 |  +6.01 |  +9.27 | 72.03 | 78.42 |  +6.39 |  +8.87 | 63.86 | 70.77 |  +6.91 |  +10.82 |
| XML_PRETTY | 16186 | 15117 | -1069 | -6.60 | 12400 | 11622 | -778 | -6.28 | 3786 | 3495 | -291 | -7.68 | 76.61 | 76.88 |  +0.27 |  +0.35 | 53.66 | 57.23 |  +3.57 |  +6.66 | 75.61 | 76.51 |  +0.90 |  +1.19 | 52.96 | 56.97 |  +4.01 |  +7.58 |
| YAML | 12574 | 11791 | -783 | -6.23 | 9532 | 9477 | -55 | -0.57 | 3042 | 2314 | -728 | -23.94 | 75.81 | 80.38 |  +4.57 |  +6.03 | 64.53 | 70.21 |  +5.68 |  +8.80 | 74.13 | 79.84 |  +5.71 |  +7.70 | 63.35 | 69.83 |  +6.48 |  +10.22 |

#### 2.6.3 Output Tokens Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 10318 | 10961 |  +643 |  +6.23 | 6973 | 7177 |  +204 |  +2.93 | 3345 | 3784 |  +439 |  +13.12 | 67.58 | 65.48 | -2.10 | -3.11 | 61.91 | 57.49 | -4.42 | -7.15 | 67.37 | 65.49 | -1.88 | -2.79 | 61.76 | 57.49 | -4.27 | -6.91 |
| JSON_COMPACT | 13489 | 9558 | -3931 | -29.14 | 10525 | 7322 | -3203 | -30.43 | 2963 | 2235 | -728 | -24.56 | 78.03 | 76.61 | -1.42 | -1.82 | 54.67 | 71.72 |  +17.06 |  +31.20 | 77.71 | 75.34 | -2.37 | -3.05 | 54.44 | 70.83 |  +16.39 |  +30.11 |
| JSON_PRETTY | 11977 | 10411 | -1566 | -13.07 | 9884 | 8194 | -1690 | -17.10 | 2092 | 2216 |  +124 |  +5.94 | 82.53 | 78.71 | -3.82 | -4.63 | 64.76 | 69.28 |  +4.52 |  +6.97 | 82.08 | 78.15 | -3.93 | -4.79 | 64.44 | 68.88 |  +4.44 |  +6.89 |
| TOON_DEFAULT | 11615 | 8984 | -2631 | -22.65 | 8930 | 6819 | -2111 | -23.64 | 2685 | 2165 | -520 | -19.37 | 76.88 | 75.90 | -0.98 | -1.27 | 62.47 | 73.86 |  +11.40 |  +18.24 | 76.63 | 75.19 | -1.44 | -1.88 | 62.29 | 73.36 |  +11.07 |  +17.78 |
| XML_COMPACT | 6976 | 11266 |  +4290 |  +61.49 | 5120 | 8844 |  +3724 |  +72.73 | 1856 | 2422 |  +566 |  +30.49 | 73.39 | 78.50 |  +5.11 |  +6.96 | 81.33 | 65.20 | -16.12 | -19.82 | 72.03 | 78.42 |  +6.39 |  +8.87 | 80.38 | 65.15 | -15.23 | -18.94 |
| XML_PRETTY | 10600 | 9209 | -1391 | -13.12 | 8121 | 7080 | -1041 | -12.81 | 2479 | 2129 | -350 | -14.12 | 76.61 | 76.88 |  +0.27 |  +0.35 | 66.94 | 73.51 |  +6.58 |  +9.82 | 75.61 | 76.51 |  +0.90 |  +1.19 | 66.24 | 73.25 |  +7.02 |  +10.59 |
| YAML | 9373 | 13029 |  +3656 |  +39.01 | 7105 | 10472 |  +3367 |  +47.39 | 2267 | 2556 |  +289 |  +12.75 | 75.81 | 80.38 |  +4.57 |  +6.03 | 72.02 | 58.42 | -13.59 | -18.88 | 74.13 | 79.84 |  +5.71 |  +7.70 | 70.84 | 58.04 | -12.80 | -18.06 |

#### 2.6.4 Total Tokens Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 17340 | 17689 |  +349 |  +2.01 | 11719 | 11584 | -135 | -1.16 | 5622 | 6107 |  +485 |  +8.62 | 67.58 | 65.48 | -2.10 | -3.11 | 77.27 | 74.70 | -2.58 | -3.33 | 67.37 | 65.49 | -1.88 | -2.79 | 77.13 | 74.70 | -2.42 | -3.14 |
| JSON_COMPACT | 22785 | 18326 | -4459 | -19.57 | 17779 | 14040 | -3739 | -21.03 | 5006 | 4287 | -719 | -14.37 | 78.03 | 76.61 | -1.42 | -1.82 | 67.33 | 80.47 |  +13.14 |  +19.51 | 77.71 | 75.34 | -2.37 | -3.05 | 67.11 | 79.58 |  +12.47 |  +18.58 |
| JSON_PRETTY | 26288 | 23806 | -2482 | -9.44 | 21695 | 18737 | -2958 | -13.63 | 4592 | 5068 |  +476 |  +10.36 | 82.53 | 78.71 | -3.82 | -4.63 | 59.38 | 64.57 |  +5.19 |  +8.74 | 82.08 | 78.15 | -3.93 | -4.79 | 59.07 | 64.18 |  +5.11 |  +8.66 |
| TOON_DEFAULT | 18797 | 20693 |  +1896 |  +10.09 | 14451 | 15706 |  +1255 |  +8.68 | 4346 | 4987 |  +641 |  +14.75 | 76.88 | 75.90 | -0.98 | -1.27 | 79.17 | 72.47 | -6.69 | -8.46 | 76.63 | 75.19 | -1.44 | -1.88 | 78.99 | 71.98 | -7.02 | -8.88 |
| XML_COMPACT | 18926 | 22447 |  +3521 |  +18.60 | 13890 | 17621 |  +3731 |  +26.86 | 5036 | 4826 | -210 | -4.17 | 73.39 | 78.50 |  +5.11 |  +6.96 | 76.32 | 68.73 | -7.58 | -9.93 | 72.03 | 78.42 |  +6.39 |  +8.87 | 75.36 | 68.68 | -6.69 | -8.87 |
| XML_PRETTY | 26786 | 24326 | -2460 | -9.18 | 20521 | 18702 | -1819 | -8.86 | 6265 | 5624 | -641 | -10.23 | 76.61 | 76.88 |  +0.27 |  +0.35 | 53.66 | 61.64 |  +7.98 |  +14.88 | 75.61 | 76.51 |  +0.90 |  +1.19 | 52.96 | 61.38 |  +8.42 |  +15.91 |
| YAML | 21947 | 24820 |  +2873 |  +13.09 | 16638 | 19951 |  +3313 |  +19.91 | 5309 | 4870 | -439 | -8.27 | 75.81 | 80.38 |  +4.57 |  +6.03 | 68.44 | 62.53 | -5.91 | -8.63 | 74.13 | 79.84 |  +5.71 |  +7.70 | 67.26 | 62.15 | -5.11 | -7.60 |

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

- **Report Generated**: 2026-04-09
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code < 2.1.86
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_flat_all_formats_and_variants_on\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_off\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_on_verify\BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)