# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Sonnet 4.6
- **Thinking**: on (medium effort)
- **Data Structure**: flat
- **Formats Tested**: 7 (CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: First iteration

## Executive Summary

This benchmark evaluates token efficiency and information accuracy across 7 file formats using Sonnet 4.6 as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context, while a format that accurately conveys information may justify higher token cost.

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
   - Optional: CSV 23086 tokens
   - Mandatory: TOON_DEFAULT 26162 tokens
- Lowest read token cost:
   - Optional: CSV 3633 tokens
   - Mandatory: CSV 3922 tokens
- Lowest output token cost:
   - Optional: JSON_COMPACT 18882 tokens
   - Mandatory: JSON_PRETTY 21118 tokens
- Lowest output token cost drift:
   - Optional: XML_COMPACT ↓ -1.48% ↑ 2.71%
   - Mandatory: CSV ↓ -1.66% ↑ 2.85%
- Highest accuracy:
   - Optional: JSON_PRETTY 98.12%
   - Mandatory: JSON_COMPACT 98.92%
- Lowest accuracy drift:
   - Optional: YAML ↓ 0.00% ↑ 0.00%
   - Mandatory: JSON_PRETTY ↓ 0.00% ↑ 0.00%
- Most useful tokens:
   - Optional: JSON_PRETTY 38313 / 39047 tokens
   - Mandatory: XML_COMPACT 37565 / 37975 tokens
- Highest token efficiency (%/token):
   - Optional: CSV 98.29
   - Mandatory: TOON_DEFAULT 93.45
- Lowest delta (optional-mandatory):
   - Total tokens: JSON_PRETTY 3895 tokens
   - Accuracy: YAML 0.00%
   - Token efficiency: XML_COMPACT 7.05

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: JSON_PRETTY 39047 tokens
   - Mandatory: XML_COMPACT 37975 tokens
- Highest read token cost:
   - Optional: JSON_PRETTY 13118 tokens
   - Mandatory: JSON_PRETTY 14034 tokens
- Highest output token cost:
   - Optional: YAML 26093 tokens
   - Mandatory: CSV 29855 tokens
- Highest output token drift:
   - Optional: CSV ↓ -98.06% ↑ 60.46%
   - Mandatory: TOON_DEFAULT ↓ -97.94% ↑ 25.10%
- Lowest accuracy:
   - Optional: CSV 97.58%
   - Mandatory: YAML 97.58%
- Highest accuracy drift:
   - Optional: TOON_DEFAULT ↓ -1.65% ↑ 1.65%
   - Mandatory: YAML ↓ -1.65% ↑ 2.48%
- Most wasted tokens:
   - Optional: YAML 910 / 37615 tokens
   - Mandatory: YAML 796 / 32884 tokens
- Lowest token efficiency (%/token):
   - Optional: JSON_PRETTY 68.70
   - Mandatory: XML_COMPACT 71.28
- Highest delta (optional-mandatory):
   - Total tokens: CSV -10692 tokens
   - Accuracy: JSON_COMPACT -1.34%
   - Token efficiency: CSV 19.32

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 279s | CSV ≈ 3922 | JSON_PRETTY ≈ 21118 | TOON_DEFAULT ≈ 26162 | TOON_DEFAULT ≈ 283 | JSON_COMPACT ≈ 99% | JSON_COMPACT ≈ 99% | TOON_DEFAULT ≈ 93 | TOON_DEFAULT ≈ 93 |
| JSON_COMPACT (+8.8%) | TOON_DEFAULT (+1.1%) | TOON_DEFAULT (+5.1%) | JSON_COMPACT (+17.6%) | JSON_COMPACT (+17.6%) | TOON_DEFAULT (0.0%) | XML_COMPACT (-0.3%) | JSON_COMPACT (-9.2%) | JSON_COMPACT (-9.0%) |
| YAML (+10.7%) | JSON_COMPACT (+58.0%) | YAML (+10.8%) | YAML (+25.7%) | XML_COMPACT (+45.2%) | XML_COMPACT (0.0%) | TOON_DEFAULT (-0.3%) | YAML (-14.5%) | YAML (-14.7%) |
| XML_PRETTY (+13.4%) | YAML (+141.7%) | JSON_COMPACT (+16.3%) | CSV (+29.1%) | CSV (+61.4%) | XML_PRETTY (-0.3%) | XML_PRETTY (-0.6%) | CSV (-15.5%) | CSV (-15.6%) |
| TOON_DEFAULT (+20.7%) | XML_COMPACT (+191.8%) | XML_PRETTY (+16.5%) | JSON_PRETTY (+34.4%) | XML_PRETTY (+78.8%) | CSV (-0.3%) | CSV (-0.7%) | JSON_PRETTY (-18.5%) | JSON_PRETTY (-18.7%) |
| XML_COMPACT (+21.7%) | XML_PRETTY (+233.7%) | XML_COMPACT (+25.6%) | XML_PRETTY (+44.1%) | JSON_PRETTY (+100.3%) | JSON_PRETTY (-0.5%) | JSON_PRETTY (-1.1%) | XML_PRETTY (-23.4%) | XML_PRETTY (-23.4%) |
| CSV (+32.8%) | JSON_PRETTY (+257.8%) | CSV (+41.4%) | XML_COMPACT (+45.2%) | YAML (+181.6%) | YAML (-1.3%) | YAML (-1.9%) | XML_COMPACT (-23.7%) | XML_COMPACT (-23.7%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Tokens | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| XML_PRETTY ≈ 265s | CSV ≈ 3633 | JSON_COMPACT ≈ 18882 | CSV ≈ 23086 | CSV ≈ 559 | JSON_PRETTY ≈ 98% | JSON_PRETTY ≈ 98% | CSV ≈ 98 | CSV ≈ 99 |
| TOON_DEFAULT (+15.7%) | JSON_COMPACT (+56.0%) | CSV (+3.0%) | JSON_COMPACT (+6.3%) | JSON_COMPACT (+6.3%) | XML_COMPACT (0.0%) | XML_COMPACT (0.0%) | JSON_COMPACT (-2.8%) | JSON_COMPACT (-2.9%) |
| XML_COMPACT (+23.8%) | XML_COMPACT (+116.1%) | XML_PRETTY (+12.4%) | XML_PRETTY (+44.0%) | XML_COMPACT (+14.1%) | CSV (-0.5%) | TOON_DEFAULT (-0.4%) | XML_PRETTY (-19.4%) | XML_PRETTY (-19.5%) |
| JSON_PRETTY (+27.3%) | TOON_DEFAULT (+211.6%) | TOON_DEFAULT (+27.1%) | XML_COMPACT (+46.9%) | JSON_PRETTY (+31.4%) | JSON_COMPACT (-0.5%) | YAML (-0.5%) | XML_COMPACT (-20.3%) | XML_COMPACT (-20.2%) |
| YAML (+28.1%) | YAML (+217.1%) | JSON_PRETTY (+37.3%) | TOON_DEFAULT (+53.0%) | XML_PRETTY (+44.0%) | TOON_DEFAULT (-0.5%) | CSV (-0.6%) | TOON_DEFAULT (-23.4%) | TOON_DEFAULT (-23.2%) |
| CSV (+32.3%) | XML_PRETTY (+230.8%) | XML_COMPACT (+38.1%) | YAML (+62.9%) | TOON_DEFAULT (+53.0%) | XML_PRETTY (-0.5%) | XML_PRETTY (-0.7%) | YAML (-27.8%) | YAML (-27.7%) |
| JSON_COMPACT (+34.0%) | JSON_PRETTY (+261.1%) | YAML (+38.2%) | JSON_PRETTY (+69.1%) | YAML (+62.9%) | YAML (-0.5%) | JSON_COMPACT (-0.7%) | JSON_PRETTY (-30.1%) | JSON_PRETTY (-30.0%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| CSV ≈ 100% | CSV ≈ 100% | JSON_COMPACT ≈ 98% | CSV ≈ 100% |
| JSON_PRETTY (0.0%) | JSON_COMPACT (0.0%) | JSON_PRETTY (-1.6%) | JSON_PRETTY (0.0%) |
| TOON_DEFAULT (0.0%) | TOON_DEFAULT (0.0%) | XML_COMPACT (-3.2%) | XML_COMPACT (0.0%) |
| XML_PRETTY (0.0%) | XML_COMPACT (0.0%) | TOON_DEFAULT (-4.0%) | TOON_DEFAULT (-0.8%) |
| YAML (0.0%) | XML_PRETTY (0.0%) | XML_PRETTY (-4.8%) | XML_PRETTY (-1.6%) |
| JSON_COMPACT (-0.6%) | YAML (0.0%) | CSV (-6.4%) | YAML (-1.6%) |
| XML_COMPACT (-0.6%) | JSON_PRETTY (-4.9%) | YAML (-11.1%) | JSON_COMPACT (-3.2%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| CSV ≈ 100% | TOON_DEFAULT ≈ 100% | CSV ≈ 100% | JSON_COMPACT ≈ 90% |
| JSON_COMPACT (0.0%) | JSON_PRETTY (-1.2%) | JSON_COMPACT (0.0%) | JSON_PRETTY (0.0%) |
| JSON_PRETTY (0.0%) | XML_COMPACT (-1.2%) | JSON_PRETTY (0.0%) | XML_COMPACT (0.0%) |
| TOON_DEFAULT (0.0%) | YAML (-1.2%) | XML_COMPACT (0.0%) | XML_PRETTY (0.0%) |
| XML_COMPACT (0.0%) | XML_PRETTY (-2.5%) | XML_PRETTY (-1.6%) | CSV (-1.6%) |
| XML_PRETTY (0.0%) | CSV (-2.5%) | YAML (-1.6%) | YAML (-1.6%) |
| YAML (0.0%) | JSON_COMPACT (-3.7%) | TOON_DEFAULT (-2.4%) | TOON_DEFAULT (-2.4%) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total | Char/Token | Info/Token | Token/Answer | Accuracy (%) | Wtd Accuracy (%) | Used Tokens | Wasted Tokens | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 3922 | 29855 | 33777 | 2.574 | 0.292 | 240.769 | 98.65 | 78.76 | 33321.339 | 455.994 | 78.97 | 78.76 |
| CSV | opt | 3633 | 19453 | 23086 | 2.636 | 0.423 | 156.876 | 97.58 | 98.50 | 22526.994 | 558.673 | 98.29 | 98.50 |
| JSON_COMPACT | man | 6197 | 24559 | 30756 | 3.214 | 0.322 | 198.056 | 98.92 | 84.91 | 30423.835 | 332.165 | 84.83 | 84.91 |
| JSON_COMPACT | opt | 5669 | 18882 | 24551 | 3.261 | 0.397 | 152.272 | 97.58 | 95.64 | 23956.541 | 594.126 | 95.54 | 95.64 |
| JSON_PRETTY | man | 14034 | 21118 | 35152 | 1.724 | 0.280 | 170.306 | 98.39 | 75.86 | 34586.053 | 565.947 | 76.20 | 75.86 |
| JSON_PRETTY | opt | 13118 | 25929 | 39047 | 1.712 | 0.251 | 209.102 | 98.12 | 68.93 | 38312.590 | 734.077 | 68.70 | 68.93 |
| TOON_DEFAULT | man | 3965 | 22197 | 26162 | 2.563 | 0.378 | 179.004 | 98.92 | 93.33 | 25878.956 | 282.544 | 93.45 | 93.33 |
| TOON_DEFAULT | opt | 11320 | 23997 | 35317 | 1.734 | 0.276 | 193.520 | 97.58 | 75.64 | 34461.841 | 854.659 | 75.33 | 75.64 |
| XML_COMPACT | man | 11444 | 26531 | 37975 | 2.417 | 0.260 | 213.960 | 98.92 | 71.18 | 37564.870 | 410.130 | 71.28 | 71.18 |
| XML_COMPACT | opt | 7851 | 26069 | 33920 | 3.258 | 0.289 | 210.237 | 98.12 | 78.56 | 33282.631 | 637.702 | 78.33 | 78.56 |
| XML_PRETTY | man | 13087 | 24607 | 37694 | 2.389 | 0.262 | 198.441 | 98.66 | 71.50 | 37188.572 | 505.095 | 71.62 | 71.50 |
| XML_PRETTY | opt | 12018 | 21227 | 33245 | 2.407 | 0.294 | 171.183 | 97.58 | 79.34 | 32440.146 | 804.521 | 79.22 | 79.34 |
| YAML | man | 9479 | 23405 | 32884 | 2.203 | 0.297 | 188.747 | 97.58 | 79.59 | 32087.882 | 795.785 | 79.89 | 79.59 |
| YAML | opt | 11522 | 26093 | 37615 | 1.684 | 0.259 | 210.427 | 97.58 | 71.25 | 36704.717 | 910.283 | 71.01 | 71.25 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 3922 | 3633 | -289 | -7.37 | 29855 | 19452 | -10403 | -34.85 | 33777 | 23085 | -10692 | -31.65 | 98.35 | 97.89 | -0.46 | 78.76 | 98.50 |  +19.75 |
| JSON_COMPACT | 6197 | 5669 | -528 | -8.52 | 24559 | 18882 | -5677 | -23.12 | 30756 | 24551 | -6205 | -20.17 | 99.04 | 97.73 | -1.31 | 84.91 | 95.64 |  +10.73 |
| JSON_PRETTY | 14034 | 13118 | -916 | -6.53 | 21118 | 25929 |  +4811 |  +22.78 | 35152 | 39047 |  +3895 |  +11.08 | 97.90 | 98.45 |  +0.55 | 75.86 | 68.93 | -6.93 |
| TOON_DEFAULT | 3965 | 11320 |  +7355 |  +185.50 | 22197 | 23997 |  +1800 |  +8.11 | 26162 | 35317 |  +9155 |  +34.99 | 98.74 | 98.02 | -0.72 | 93.33 | 75.64 | -17.69 |
| XML_COMPACT | 11444 | 7851 | -3593 | -31.40 | 26531 | 26069 | -462 | -1.74 | 37975 | 33920 | -4055 | -10.68 | 98.78 | 98.45 | -0.33 | 71.18 | 78.56 |  +7.38 |
| XML_PRETTY | 13087 | 12018 | -1069 | -8.17 | 24607 | 21227 | -3380 | -13.74 | 37694 | 33245 | -4449 | -11.80 | 98.48 | 97.76 | -0.72 | 71.50 | 79.34 |  +7.85 |
| YAML | 9479 | 11522 |  +2043 |  +21.55 | 23405 | 26093 |  +2688 |  +11.48 | 32884 | 37615 |  +4731 |  +14.39 | 97.15 | 97.92 |  +0.77 | 79.59 | 71.25 | -8.34 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 17 | 230.706 | 0.55 | 371149 | 0.080 | 2993.13 | 371166 | 230.786 | 2394.62 |
| CSV | opt | 24 | 151.375 | 0.77 | 351118 | 0.055 | 2831.60 | 351142 | 151.430 | 2265.43 |
| JSON_COMPACT | man | 12 | 516.417 | 0.39 | 303939 | 0.081 | 2451.12 | 303951 | 516.498 | 1960.97 |
| JSON_COMPACT | opt | 14 | 404.929 | 0.45 | 355690 | 0.053 | 2868.47 | 355704 | 404.982 | 2294.86 |
| JSON_PRETTY | man | 18 | 779.667 | 0.58 | 279380 | 0.076 | 2253.06 | 279398 | 779.743 | 1802.57 |
| JSON_PRETTY | opt | 23 | 570.348 | 0.74 | 337845 | 0.077 | 2724.56 | 337868 | 570.425 | 2179.79 |
| TOON_DEFAULT | man | 39 | 101.667 | 1.26 | 337186 | 0.066 | 2719.24 | 337225 | 101.733 | 2175.65 |
| TOON_DEFAULT | opt | 21 | 539.048 | 0.68 | 306906 | 0.078 | 2475.05 | 306927 | 539.126 | 1980.18 |
| XML_COMPACT | man | 32 | 357.625 | 1.03 | 339955 | 0.078 | 2741.57 | 339987 | 357.703 | 2193.46 |
| XML_COMPACT | opt | 91 | 86.275 | 2.94 | 328348 | 0.079 | 2647.97 | 328439 | 86.354 | 2118.96 |
| XML_PRETTY | man | 69 | 189.667 | 2.23 | 316790 | 0.078 | 2554.76 | 316859 | 189.745 | 2044.25 |
| XML_PRETTY | opt | 27 | 445.111 | 0.87 | 265342 | 0.080 | 2139.85 | 265369 | 445.191 | 1712.06 |
| YAML | man | 27 | 351.074 | 0.87 | 309263 | 0.076 | 2494.06 | 309290 | 351.150 | 1995.42 |
| YAML | opt | 30 | 384.067 | 0.97 | 339934 | 0.077 | 2741.41 | 339964 | 384.144 | 2193.32 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) | Total Man (s) | Total Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 17 | 24 |  +7 |  +41.18 | 371.15 | 351.12 | -20.03 | -5.40 | 371.17 | 351.14 | -20.02 | -5.39 |
| JSON_COMPACT | 12 | 14 |  +2 |  +16.67 | 303.94 | 355.69 |  +51.75 |  +17.03 | 303.95 | 355.70 |  +51.75 |  +17.03 |
| JSON_PRETTY | 18 | 23 |  +5 |  +27.78 | 279.38 | 337.85 |  +58.47 |  +20.93 | 279.40 | 337.87 |  +58.47 |  +20.93 |
| TOON_DEFAULT | 39 | 21 | -18 | -46.15 | 337.19 | 306.91 | -30.28 | -8.98 | 337.23 | 306.93 | -30.30 | -8.98 |
| XML_COMPACT | 32 | 91 |  +59 |  +184.38 | 339.95 | 328.35 | -11.61 | -3.41 | 339.99 | 328.44 | -11.55 | -3.40 |
| XML_PRETTY | 69 | 27 | -42 | -60.87 | 316.79 | 265.34 | -51.45 | -16.24 | 316.86 | 265.37 | -51.49 | -16.25 |
| YAML | 27 | 30 |  +3 |  +11.11 | 309.26 | 339.93 |  +30.67 |  +9.92 | 309.29 | 339.96 |  +30.67 |  +9.92 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Char/Token | Token/Value | Token/Object | Info/Token |
|---|---|---|---|---|---|
| CSV | man | 2.574 | 5.751 | 126.516 | 0.292 |
| CSV | opt | 2.636 | 5.758 | 117.194 | 0.423 |
| JSON_COMPACT | man | 3.214 | 9.087 | 199.903 | 0.322 |
| JSON_COMPACT | opt | 3.261 | 8.984 | 182.871 | 0.397 |
| JSON_PRETTY | man | 1.724 | 20.578 | 452.710 | 0.280 |
| JSON_PRETTY | opt | 1.712 | 20.789 | 423.161 | 0.251 |
| TOON_DEFAULT | man | 2.563 | 5.814 | 127.903 | 0.378 |
| TOON_DEFAULT | opt | 1.734 | 17.940 | 365.161 | 0.276 |
| XML_COMPACT | man | 2.417 | 16.780 | 369.161 | 0.260 |
| XML_COMPACT | opt | 3.258 | 12.442 | 253.258 | 0.289 |
| XML_PRETTY | man | 2.389 | 19.189 | 422.161 | 0.262 |
| XML_PRETTY | opt | 2.407 | 19.046 | 387.677 | 0.294 |
| YAML | man | 2.203 | 13.899 | 305.774 | 0.297 |
| YAML | opt | 1.684 | 18.260 | 371.677 | 0.259 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 2.574 | 2.636 |  +0.062 |  +2.41 | 5.751 | 5.758 |  +0.007 |  +0.12 | 126.516 | 117.194 | -9.322 | -7.37 | 0.292 | 0.423 |  +0.131 |  +44.86 |
| JSON_COMPACT | 3.214 | 3.261 |  +0.047 |  +1.46 | 9.087 | 8.984 | -0.103 | -1.13 | 199.903 | 182.871 | -17.032 | -8.52 | 0.322 | 0.397 |  +0.075 |  +23.29 |
| JSON_PRETTY | 1.724 | 1.712 | -0.012 | -0.70 | 20.578 | 20.789 |  +0.211 |  +1.03 | 452.710 | 423.161 | -29.549 | -6.53 | 0.280 | 0.251 | -0.029 | -10.36 |
| TOON_DEFAULT | 2.563 | 1.734 | -0.829 | -32.34 | 5.814 | 17.940 |  +12.126 |  +208.57 | 127.903 | 365.161 |  +237.258 |  +185.50 | 0.378 | 0.276 | -0.102 | -26.98 |
| XML_COMPACT | 2.417 | 3.258 |  +0.841 |  +34.80 | 16.780 | 12.442 | -4.338 | -25.85 | 369.161 | 253.258 | -115.903 | -31.40 | 0.260 | 0.289 |  +0.029 |  +11.15 |
| XML_PRETTY | 2.389 | 2.407 |  +0.018 |  +0.75 | 19.189 | 19.046 | -0.143 | -0.75 | 422.161 | 387.677 | -34.484 | -8.17 | 0.262 | 0.294 |  +0.032 |  +12.21 |
| YAML | 2.203 | 1.684 | -0.519 | -23.56 | 13.899 | 18.260 |  +4.361 |  +31.38 | 305.774 | 371.677 |  +65.903 |  +21.55 | 0.297 | 0.259 | -0.038 | -12.79 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| CSV | man | 33777 | 33321 | 456 | 98.65 | 98.35 | 78.97 | 78.76 |
| CSV | opt | 23086 | 22527 | 559 | 97.58 | 97.89 | 98.29 | 98.50 |
| JSON_COMPACT | man | 30756 | 30424 | 332 | 98.92 | 99.04 | 84.83 | 84.91 |
| JSON_COMPACT | opt | 24551 | 23957 | 594 | 97.58 | 97.73 | 95.54 | 95.64 |
| JSON_PRETTY | man | 35152 | 34586 | 566 | 98.39 | 97.90 | 76.20 | 75.86 |
| JSON_PRETTY | opt | 39047 | 38313 | 734 | 98.12 | 98.45 | 68.70 | 68.93 |
| TOON_DEFAULT | man | 26162 | 25879 | 283 | 98.92 | 98.74 | 93.45 | 93.33 |
| TOON_DEFAULT | opt | 35317 | 34462 | 855 | 97.58 | 98.02 | 75.33 | 75.64 |
| XML_COMPACT | man | 37975 | 37565 | 410 | 98.92 | 98.78 | 71.28 | 71.18 |
| XML_COMPACT | opt | 33920 | 33283 | 638 | 98.12 | 98.45 | 78.33 | 78.56 |
| XML_PRETTY | man | 37694 | 37189 | 505 | 98.66 | 98.48 | 71.62 | 71.50 |
| XML_PRETTY | opt | 33245 | 32440 | 805 | 97.58 | 97.76 | 79.22 | 79.34 |
| YAML | man | 32884 | 32088 | 796 | 97.58 | 97.15 | 79.89 | 79.59 |
| YAML | opt | 37615 | 36705 | 910 | 97.58 | 97.92 | 71.01 | 71.25 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 33777 | 23085 | -10692 | -31.65 | 33321 | 22527 | -10794 | -32.40 | 456 | 559 |  +103 |  +22.52 | 98.65 | 97.58 | -1.07 | 78.97 | 98.287 |  +19.32 |  +24.47 |
| JSON_COMPACT | 30756 | 24551 | -6205 | -20.18 | 30424 | 23957 | -6467 | -21.26 | 332 | 594 |  +262 |  +78.90 | 98.92 | 97.58 | -1.34 | 84.83 | 95.537 |  +10.71 |  +12.63 |
| JSON_PRETTY | 35152 | 39047 |  +3895 |  +11.08 | 34586 | 38313 |  +3727 |  +10.77 | 566 | 734 |  +168 |  +29.70 | 98.39 | 98.12 | -0.27 | 76.20 | 68.703 | -7.50 | -9.84 |
| TOON_DEFAULT | 26162 | 35317 |  +9155 |  +34.99 | 25879 | 34462 |  +8583 |  +33.17 | 283 | 855 |  +572 |  +202.16 | 98.92 | 97.58 | -1.34 | 93.45 | 75.327 | -18.12 | -19.39 |
| XML_COMPACT | 37975 | 33920 | -4055 | -10.68 | 37565 | 33283 | -4282 | -11.40 | 410 | 638 |  +228 |  +55.51 | 98.92 | 98.12 | -0.80 | 71.28 | 78.326 |  +7.05 |  +9.89 |
| XML_PRETTY | 37694 | 33245 | -4449 | -11.80 | 37189 | 32441 | -4748 | -12.77 | 505 | 804 |  +299 |  +59.29 | 98.66 | 97.58 | -1.08 | 71.62 | 79.216 |  +7.59 |  +10.60 |
| YAML | 32884 | 37615 |  +4731 |  +14.39 | 32088 | 36705 |  +4617 |  +14.39 | 796 | 910 |  +114 |  +14.38 | 97.58 | 97.58 | 0.00 | 79.89 | 71.012 | -8.88 | -11.12 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| CSV | man | 122 | 2 | 0 | 98.65 |
| CSV | opt | 121 | 3 | 0 | 97.58 |
| JSON_COMPACT | man | 123 | 1 | 0 | 98.92 |
| JSON_COMPACT | opt | 121 | 3 | 0 | 97.58 |
| JSON_PRETTY | man | 122 | 2 | 0 | 98.39 |
| JSON_PRETTY | opt | 122 | 2 | 0 | 98.12 |
| TOON_DEFAULT | man | 123 | 1 | 0 | 98.92 |
| TOON_DEFAULT | opt | 121 | 3 | 0 | 97.58 |
| XML_COMPACT | man | 123 | 1 | 0 | 98.92 |
| XML_COMPACT | opt | 122 | 2 | 0 | 98.12 |
| XML_PRETTY | man | 122 | 2 | 0 | 98.66 |
| XML_PRETTY | opt | 121 | 3 | 0 | 97.58 |
| YAML | man | 121 | 3 | 0 | 97.58 |
| YAML | opt | 121 | 3 | 0 | 97.58 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 122 | 121 | -1 | -0.82 | 2 | 3 |  +1 |  +50.00 | 0 | 0 | 0 | 0.00 | 98.65 | 97.58 | -1.07 |
| JSON_COMPACT | 123 | 121 | -2 | -1.63 | 1 | 3 |  +2 |  +200.00 | 0 | 0 | 0 | 0.00 | 98.92 | 97.58 | -1.34 |
| JSON_PRETTY | 122 | 122 | 0 | 0.00 | 2 | 2 | 0 | 0.00 | 0 | 0 | 0 | 0.00 | 98.39 | 98.12 | -0.27 |
| TOON_DEFAULT | 123 | 121 | -2 | -1.63 | 1 | 3 |  +2 |  +200.00 | 0 | 0 | 0 | 0.00 | 98.92 | 97.58 | -1.34 |
| XML_COMPACT | 123 | 122 | -1 | -0.81 | 1 | 2 |  +1 |  +100.00 | 0 | 0 | 0 | 0.00 | 98.92 | 98.12 | -0.80 |
| XML_PRETTY | 122 | 121 | -1 | -0.82 | 2 | 3 |  +1 |  +50.00 | 0 | 0 | 0 | 0.00 | 98.66 | 97.58 | -1.08 |
| YAML | 121 | 121 | 0 | 0.00 | 3 | 3 | 0 | 0.00 | 0 | 0 | 0 | 0.00 | 97.58 | 97.58 | 0.00 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| CSV | man | 98.65 | 100.00 | 100.00 | 92.06 | 100.00 |
| CSV | opt | 97.58 | 100.00 | 97.53 | 100.00 | 88.89 |
| JSON_COMPACT | man | 98.92 | 99.39 | 100.00 | 98.41 | 96.83 |
| JSON_COMPACT | opt | 97.58 | 100.00 | 96.30 | 100.00 | 90.48 |
| JSON_PRETTY | man | 98.39 | 100.00 | 95.06 | 96.83 | 100.00 |
| JSON_PRETTY | opt | 98.12 | 100.00 | 98.77 | 100.00 | 90.48 |
| TOON_DEFAULT | man | 98.92 | 100.00 | 100.00 | 94.45 | 99.21 |
| TOON_DEFAULT | opt | 97.58 | 100.00 | 100.00 | 97.62 | 88.10 |
| XML_COMPACT | man | 98.92 | 99.39 | 100.00 | 95.24 | 100.00 |
| XML_COMPACT | opt | 98.12 | 100.00 | 98.77 | 100.00 | 90.48 |
| XML_PRETTY | man | 98.66 | 100.00 | 100.00 | 93.65 | 98.41 |
| XML_PRETTY | opt | 97.58 | 100.00 | 97.53 | 98.41 | 90.48 |
| YAML | man | 97.58 | 100.00 | 100.00 | 87.30 | 98.41 |
| YAML | opt | 97.58 | 100.00 | 98.77 | 98.41 | 88.89 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 100.00 | 100.00 | 0.00 |
| JSON_COMPACT | 99.39 | 100.00 |  +0.61 |
| JSON_PRETTY | 100.00 | 100.00 | 0.00 |
| TOON_DEFAULT | 100.00 | 100.00 | 0.00 |
| XML_COMPACT | 99.39 | 100.00 |  +0.61 |
| XML_PRETTY | 100.00 | 100.00 | 0.00 |
| YAML | 100.00 | 100.00 | 0.00 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 100.00 | 97.53 | -2.47 |
| JSON_COMPACT | 100.00 | 96.30 | -3.70 |
| JSON_PRETTY | 95.06 | 98.77 |  +3.71 |
| TOON_DEFAULT | 100.00 | 100.00 | 0.00 |
| XML_COMPACT | 100.00 | 98.77 | -1.23 |
| XML_PRETTY | 100.00 | 97.53 | -2.47 |
| YAML | 100.00 | 98.77 | -1.23 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 92.06 | 100.00 |  +7.94 |
| JSON_COMPACT | 98.41 | 100.00 |  +1.59 |
| JSON_PRETTY | 96.83 | 100.00 |  +3.17 |
| TOON_DEFAULT | 94.45 | 97.62 |  +3.17 |
| XML_COMPACT | 95.24 | 100.00 |  +4.76 |
| XML_PRETTY | 93.65 | 98.41 |  +4.76 |
| YAML | 87.30 | 98.41 |  +11.11 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 100.00 | 88.89 | -11.11 |
| JSON_COMPACT | 96.83 | 90.48 | -6.35 |
| JSON_PRETTY | 100.00 | 90.48 | -9.52 |
| TOON_DEFAULT | 99.21 | 88.10 | -11.11 |
| XML_COMPACT | 100.00 | 90.48 | -9.52 |
| XML_PRETTY | 98.41 | 90.48 | -7.93 |
| YAML | 98.41 | 88.89 | -9.52 |

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Sonnet 4.6
- **Thinking**: on (medium effort)
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

- **Report Generated**: 2026-04-01
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code < 2.1.86
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)