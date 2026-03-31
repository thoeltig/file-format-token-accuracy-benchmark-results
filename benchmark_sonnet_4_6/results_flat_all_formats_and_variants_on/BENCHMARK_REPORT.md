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
- **Read Tokens**: For each data file a single read subagent is invoked with the only prompt to read the file at the provided filepath and return "Done" once finished and do nothing more. The tokens extraction script searches for the read tool use result and extracted the tokens for that action from it.
- **Output Tokens**: For each data file a three full tests subagent are invoked with all necessary files and the test setup and the instruction to write a file once with the answers. The token extraction script searches for the write tool use result and extracted the tokens for that action from it.

## 2. Results

### 2.1 TLDR: Token Efficiency Analysis

*Note: All columns ranked best-to-worst. ↑ = lower value is better (ascending). ↓ = higher value is better (descending).*

#### 2.1.1 Best results

- Lowest total token cost:
   - Optional: CSV 3926 tokens
   - Mandatory: CSV 4215 tokens
- Lowest output token cost drift:
   - Optional: CSV ↓ -0.45% ↑ 0.23%
   - Mandatory: JSON_COMPACT ↓ -0.56% ↑ 0.45%
- Highest accuracy:
   - Optional: JSON_PRETTY 98.12%
   - Mandatory: JSON_COMPACT 98.92%
- Lowest accuracy drift:
   - Optional: YAML ↓ 0.00% ↑ 0.00%
   - Mandatory: JSON_PRETTY ↓ 0.00% ↑ 0.00%
- Most useful tokens:
   - Optional: JSON_PRETTY 13160 / 13412 tokens
   - Mandatory: JSON_PRETTY 14006 / 14235 tokens
- Highest token efficiency (%/token):
   - Optional: CSV 98.28
   - Mandatory: TOON_DEFAULT 98.36
- Lowest delta (optional-mandatory):
   - Total tokens: CSV -289 tokens
   - Accuracy: YAML 0.00%
   - Token efficiency: CSV 0.09

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: JSON_PRETTY 13412 tokens
   - Mandatory: JSON_PRETTY 14235 tokens
- Highest output token drift:
   - Optional: XML_COMPACT ↓ -2.71% ↑ 1.69%
   - Mandatory: JSON_PRETTY ↓ -93.03% ↑ 48.26%
- Lowest accuracy:
   - Optional: CSV 97.58%
   - Mandatory: YAML 97.58%
- Highest accuracy drift:
   - Optional: TOON_DEFAULT ↓ -1.65% ↑ 1.65%
   - Mandatory: YAML ↓ -1.65% ↑ 2.48%
- Most wasted tokens:
   - Optional: XML_PRETTY 298 / 12314 tokens
   - Mandatory: YAML 236 / 9771 tokens
- Lowest token efficiency (%/token):
   - Optional: JSON_PRETTY 71.10
   - Mandatory: JSON_PRETTY 68.90
- Highest delta (optional-mandatory):
   - Total tokens: TOON_DEFAULT 7403 tokens
   - Accuracy: JSON_COMPACT -1.34%
   - Token efficiency: TOON_DEFAULT -22.44

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Accuracy | ↓ Wtd Accuracy | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 168s | CSV ≈ 4215 | TOON_DEFAULT ≈ 46 | JSON_COMPACT ≈ 99% | JSON_COMPACT ≈ 99% | TOON_DEFAULT ≈ 98 | TOON_DEFAULT ≈ 98 |
| JSON_PRETTY (+66.8%) | TOON_DEFAULT (+0.1%) | CSV (+24.8%) | TOON_DEFAULT (0.0%) | XML_COMPACT (-0.3%) | CSV (-0.2%) | CSV (-0.3%) |
| JSON_COMPACT (+81.4%) | JSON_COMPACT (+54.1%) | JSON_COMPACT (+53.9%) | XML_COMPACT (0.0%) | TOON_DEFAULT (-0.3%) | JSON_COMPACT (-6.7%) | JSON_COMPACT (-6.5%) |
| YAML (+84.6%) | YAML (+131.8%) | XML_COMPACT (+178.1%) | XML_PRETTY (-0.3%) | XML_PRETTY (-0.6%) | YAML (-17.3%) | YAML (-17.5%) |
| XML_PRETTY (+89.1%) | XML_COMPACT (+178.4%) | XML_PRETTY (+293.4%) | CSV (-0.3%) | CSV (-0.7%) | XML_COMPACT (-22.2%) | XML_COMPACT (-22.2%) |
| XML_COMPACT (+102.9%) | XML_PRETTY (+217.5%) | JSON_PRETTY (+402.8%) | JSON_PRETTY (-0.5%) | JSON_PRETTY (-1.1%) | XML_PRETTY (-27.2%) | XML_PRETTY (-27.3%) |
| CSV (+121.5%) | JSON_PRETTY (+237.7%) | YAML (+418.8%) | YAML (-1.3%) | YAML (-1.9%) | JSON_PRETTY (-30.0%) | JSON_PRETTY (-30.2%) |


##### Optional

| ↑ Total Duration | ↑ Total Tokens | ↑ Wasted Tokens | ↓ Acc | ↓ Wtd Acc | ↓ Eff Score | ↓ Wtd Eff Score |
|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 160s | CSV ≈ 3926 | CSV ≈ 95 | JSON_PRETTY ≈ 98% | JSON_PRETTY ≈ 98% | CSV ≈ 98 | CSV ≈ 98 |
| XML_PRETTY (+65.8%) | JSON_COMPACT (+51.9%) | JSON_COMPACT (+51.9%) | XML_COMPACT (0.0%) | XML_COMPACT (0.0%) | JSON_COMPACT (-6.0%) | JSON_COMPACT (-6.1%) |
| XML_COMPACT (+105.2%) | XML_COMPACT (+107.5%) | XML_COMPACT (+61.2%) | CSV (-0.5%) | TOON_DEFAULT (-0.4%) | XML_COMPACT (-12.1%) | XML_COMPACT (-12.0%) |
| JSON_PRETTY (+111.1%) | TOON_DEFAULT (+196.0%) | JSON_PRETTY (+165.4%) | JSON_COMPACT (-0.5%) | YAML (-0.5%) | TOON_DEFAULT (-22.7%) | TOON_DEFAULT (-22.6%) |
| YAML (+112.4%) | YAML (+200.9%) | TOON_DEFAULT (+196.0%) | TOON_DEFAULT (-0.5%) | CSV (-0.6%) | YAML (-23.3%) | YAML (-23.2%) |
| CSV (+119.4%) | XML_PRETTY (+213.6%) | YAML (+200.9%) | XML_PRETTY (-0.5%) | XML_PRETTY (-0.7%) | XML_PRETTY (-24.8%) | XML_PRETTY (-24.8%) |
| JSON_COMPACT (+122.3%) | JSON_PRETTY (+241.6%) | XML_PRETTY (+213.6%) | YAML (-0.5%) | JSON_COMPACT (-0.7%) | JSON_PRETTY (-27.6%) | JSON_PRETTY (-27.6%) |


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
| CSV | man | 3922 | 293 | 4215 | 2.574 | 2.340 | 2.363 | 98.65 | 97.98 | 4158.097 | 56.902 | 98.19 | 97.98 |
| CSV | opt | 3633 | 293 | 3926 | 2.636 | 2.485 | 2.366 | 97.58 | 98.49 | 3831.316 | 95.017 | 98.28 | 98.49 |
| JSON_COMPACT | man | 6197 | 299 | 6496 | 3.214 | 1.523 | 2.409 | 98.92 | 91.84 | 6425.514 | 70.153 | 91.75 | 91.84 |
| JSON_COMPACT | opt | 5669 | 294 | 5963 | 3.261 | 1.636 | 2.374 | 97.58 | 92.47 | 5819.020 | 144.313 | 92.36 | 92.47 |
| JSON_PRETTY | man | 14034 | 201 | 14235 | 1.724 | 0.691 | 1.621 | 98.39 | 68.56 | 14005.817 | 229.184 | 68.90 | 68.56 |
| JSON_PRETTY | opt | 13118 | 294 | 13412 | 1.712 | 0.732 | 2.368 | 98.12 | 71.33 | 13159.528 | 252.139 | 71.10 | 71.33 |
| TOON_DEFAULT | man | 3965 | 255 | 4220 | 2.563 | 2.344 | 2.058 | 98.92 | 98.24 | 4174.589 | 45.578 | 98.36 | 98.24 |
| TOON_DEFAULT | opt | 11320 | 303 | 11623 | 1.734 | 0.839 | 2.446 | 97.58 | 76.23 | 11342.049 | 281.284 | 75.92 | 76.23 |
| XML_COMPACT | man | 11444 | 292 | 11736 | 2.417 | 0.843 | 2.358 | 98.92 | 76.43 | 11609.581 | 126.752 | 76.53 | 76.43 |
| XML_COMPACT | opt | 7851 | 295 | 8146 | 3.258 | 1.205 | 2.379 | 98.12 | 86.63 | 7992.855 | 153.145 | 86.40 | 86.63 |
| XML_PRETTY | man | 13087 | 295 | 13382 | 2.389 | 0.737 | 2.379 | 98.66 | 71.44 | 13202.681 | 179.319 | 71.57 | 71.44 |
| XML_PRETTY | opt | 12018 | 296 | 12314 | 2.407 | 0.792 | 2.384 | 97.58 | 74.04 | 12015.676 | 297.991 | 73.92 | 74.04 |
| YAML | man | 9479 | 292 | 9771 | 2.203 | 0.999 | 2.352 | 97.58 | 81.00 | 9534.217 | 236.450 | 81.30 | 81.00 |
| YAML | opt | 11522 | 292 | 11814 | 1.684 | 0.826 | 2.355 | 97.58 | 75.61 | 11528.101 | 285.899 | 75.37 | 75.61 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Tokens Man | Tokens Opt | Diff | Diff (%) | Accuracy Man (%) | Accuracy Opt (%) | Diff (%) | Wtd Accuracy Man (%) | Wtd Accuracy Opt (%) | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Wtd Eff Score Man | Wtd Eff Score Opt | Diff |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 4215 | 3926 | -289 | -6.86 | 98.65 | 97.58 | -1.07 | 98.35 | 97.89 | -0.46 | 98.19 | 98.28 |  +0.09 | 97.98 | 98.49 |  +0.52 |
| JSON_COMPACT | 6496 | 5964 | -532 | -8.19 | 98.92 | 97.58 | -1.34 | 99.04 | 97.73 | -1.31 | 91.75 | 92.36 |  +0.61 | 91.84 | 92.47 |  +0.63 |
| JSON_PRETTY | 14235 | 13412 | -823 | -5.78 | 98.39 | 98.12 | -0.27 | 97.90 | 98.45 |  +0.55 | 68.90 | 71.10 |  +2.20 | 68.56 | 71.33 |  +2.78 |
| TOON_DEFAULT | 4220 | 11623 |  +7403 |  +175.43 | 98.92 | 97.58 | -1.34 | 98.74 | 98.02 | -0.72 | 98.36 | 75.92 | -22.44 | 98.24 | 76.23 | -22.01 |
| XML_COMPACT | 11736 | 8146 | -3590 | -30.59 | 98.92 | 98.12 | -0.80 | 98.78 | 98.45 | -0.33 | 76.53 | 86.40 |  +9.87 | 76.43 | 86.63 |  +10.20 |
| XML_PRETTY | 13382 | 12314 | -1068 | -7.98 | 98.66 | 97.58 | -1.08 | 98.48 | 97.76 | -0.72 | 71.57 | 73.92 |  +2.35 | 71.44 | 74.04 |  +2.60 |
| YAML | 9771 | 11814 |  +2043 |  +20.91 | 97.58 | 97.58 | 0.00 | 97.15 | 97.92 |  +0.77 | 81.30 | 75.37 | -5.94 | 81.00 | 75.61 | -5.40 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output (ms) | Output (tokens/ms) | Rate (ms/question) | Total (ms) | Total (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 17 | 230.706 | 0.55 | 371149 | 0.001 | 2993.13 | 371166 | 230.707 | 2394.62 |
| CSV | opt | 24 | 151.375 | 0.77 | 351118 | 0.001 | 2831.60 | 351142 | 151.376 | 2265.43 |
| JSON_COMPACT | man | 12 | 516.417 | 0.39 | 303939 | 0.001 | 2451.12 | 303951 | 516.418 | 1960.97 |
| JSON_COMPACT | opt | 14 | 404.929 | 0.45 | 355690 | 0.001 | 2868.47 | 355704 | 404.930 | 2294.86 |
| JSON_PRETTY | man | 18 | 779.667 | 0.58 | 279380 | 0.001 | 2253.06 | 279398 | 779.668 | 1802.57 |
| JSON_PRETTY | opt | 23 | 570.348 | 0.74 | 337845 | 0.001 | 2724.56 | 337868 | 570.349 | 2179.79 |
| TOON_DEFAULT | man | 39 | 101.667 | 1.26 | 337186 | 0.001 | 2719.24 | 337225 | 101.668 | 2175.65 |
| TOON_DEFAULT | opt | 21 | 539.048 | 0.68 | 306906 | 0.001 | 2475.05 | 306927 | 539.049 | 1980.18 |
| XML_COMPACT | man | 32 | 357.625 | 1.03 | 339955 | 0.001 | 2741.57 | 339987 | 357.626 | 2193.46 |
| XML_COMPACT | opt | 91 | 86.275 | 2.94 | 328348 | 0.001 | 2647.97 | 328439 | 86.276 | 2118.96 |
| XML_PRETTY | man | 69 | 189.667 | 2.23 | 316790 | 0.001 | 2554.76 | 316859 | 189.668 | 2044.25 |
| XML_PRETTY | opt | 27 | 445.111 | 0.87 | 265342 | 0.001 | 2139.85 | 265369 | 445.112 | 1712.06 |
| YAML | man | 27 | 351.074 | 0.87 | 309263 | 0.001 | 2494.06 | 309290 | 351.075 | 1995.42 |
| YAML | opt | 30 | 384.067 | 0.97 | 339934 | 0.001 | 2741.41 | 339964 | 384.068 | 2193.32 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) | Total Man (s) | Total Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 17 | 24 |  +7 |  +41.18 | 371.15 | 351.12 | -20.03 | -5.40 | 371.17 | 351.14 | -20.02 | -5.39 |
| JSON_COMPACT | 12 | 14 |  +2 |  +16.67 | 303.94 | 355.69 |  +51.75 |  +17.03 | 303.95 | 355.70 |  +51.75 |  +17.03 |
| JSON_PRETTY | 18 | 23 |  +5 |  +27.78 | 279.38 | 337.85 |  +58.47 |  +20.93 | 279.40 | 337.87 |  +58.47 |  +20.93 |
| TOON_DEFAULT | 39 | 21 | -18 | -46.15 | 337.19 | 306.91 | -30.28 | -8.98 | 337.23 | 329.73 | -7.49 | -2.22 |
| XML_COMPACT | 32 | 91 |  +59 |  +184.38 | 339.95 | 328.35 | -11.61 | -3.41 | 339.99 | 328.44 | -11.55 | -3.40 |
| XML_PRETTY | 69 | 27 | -42 | -60.87 | 316.79 | 265.34 | -51.45 | -16.24 | 316.86 | 265.37 | -51.49 | -16.25 |
| YAML | 27 | 30 |  +3 |  +11.11 | 309.26 | 339.93 |  +30.67 |  +9.92 | 309.29 | 339.96 |  +30.67 |  +9.92 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Char/Token | Token/Value | Token/Object | Info/Token |
|---|---|---|---|---|---|
| CSV | man | 2.574 | 5.751 | 126.516 | 2.340 |
| CSV | opt | 2.636 | 5.758 | 117.194 | 2.485 |
| JSON_COMPACT | man | 3.214 | 9.087 | 199.903 | 1.523 |
| JSON_COMPACT | opt | 3.261 | 8.984 | 182.871 | 1.636 |
| JSON_PRETTY | man | 1.724 | 20.578 | 452.710 | 0.691 |
| JSON_PRETTY | opt | 1.712 | 20.789 | 423.161 | 0.732 |
| TOON_DEFAULT | man | 2.563 | 5.814 | 127.903 | 2.344 |
| TOON_DEFAULT | opt | 1.734 | 17.940 | 365.161 | 0.839 |
| XML_COMPACT | man | 2.417 | 16.780 | 369.161 | 0.843 |
| XML_COMPACT | opt | 3.258 | 12.442 | 253.258 | 1.205 |
| XML_PRETTY | man | 2.389 | 19.189 | 422.161 | 0.737 |
| XML_PRETTY | opt | 2.407 | 19.046 | 387.677 | 0.792 |
| YAML | man | 2.203 | 13.899 | 305.774 | 0.999 |
| YAML | opt | 1.684 | 18.260 | 371.677 | 0.826 |

#### 2.5.2 Mandatory vs Optional
| Format | Char/Token Man | Char/Token Opt | Diff | Diff (%) | Token/Value Man | Token/Value Opt | Diff | Diff (%) | Token/Object Man | Token/Object Opt | Diff | Diff (%) | Info/Token Man | Info/Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 2.574 | 2.636 |  +0.062 |  +2.41 | 5.751 | 5.758 |  +0.007 |  +0.12 | 126.516 | 117.194 | -9.322 | -7.37 | 2.340 | 2.485 |  +0.145 |  +6.20 |
| JSON_COMPACT | 3.214 | 3.261 |  +0.047 |  +1.46 | 9.087 | 8.984 | -0.103 | -1.13 | 199.903 | 182.871 | -17.032 | -8.52 | 1.523 | 1.636 |  +0.113 |  +7.42 |
| JSON_PRETTY | 1.724 | 1.712 | -0.012 | -0.70 | 20.578 | 20.789 |  +0.211 |  +1.03 | 452.710 | 423.161 | -29.549 | -6.53 | 0.691 | 0.732 |  +0.041 |  +5.93 |
| TOON_DEFAULT | 2.563 | 1.734 | -0.829 | -32.34 | 5.814 | 17.940 |  +12.126 |  +208.57 | 127.903 | 365.161 |  +237.258 |  +185.50 | 2.344 | 0.839 | -1.505 | -64.19 |
| XML_COMPACT | 2.417 | 3.258 |  +0.841 |  +34.80 | 16.780 | 12.442 | -4.338 | -25.85 | 369.161 | 253.258 | -115.903 | -31.40 | 0.843 | 1.205 |  +0.362 |  +42.94 |
| XML_PRETTY | 2.389 | 2.407 |  +0.018 |  +0.75 | 19.189 | 19.046 | -0.143 | -0.75 | 422.161 | 387.677 | -34.484 | -8.17 | 0.737 | 0.792 |  +0.055 |  +7.46 |
| YAML | 2.203 | 1.684 | -0.519 | -23.56 | 13.899 | 18.260 |  +4.361 |  +31.38 | 305.774 | 371.677 |  +65.903 |  +21.55 | 0.999 | 0.826 | -0.173 | -17.32 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Total Tokens | Useful Tokens | Wasted Tokens | Accuracy (%) | Wtd Accuracy (%) | Eff Score | Wtd Eff Score |
|---|---|---|---|---|---|---|---|---|
| CSV | man | 4215 | 4158 | 57 | 98.65 | 98.35 | 98.19 | 97.98 |
| CSV | opt | 3926 | 3831 | 95 | 97.58 | 97.89 | 98.28 | 98.49 |
| JSON_COMPACT | man | 6496 | 6426 | 70 | 98.92 | 99.04 | 91.75 | 91.84 |
| JSON_COMPACT | opt | 5963 | 5819 | 144 | 97.58 | 97.73 | 92.36 | 92.47 |
| JSON_PRETTY | man | 14235 | 14006 | 229 | 98.39 | 97.90 | 68.90 | 68.56 |
| JSON_PRETTY | opt | 13412 | 13160 | 252 | 98.12 | 98.45 | 71.10 | 71.33 |
| TOON_DEFAULT | man | 4220 | 4175 | 46 | 98.92 | 98.74 | 98.36 | 98.24 |
| TOON_DEFAULT | opt | 11623 | 11342 | 281 | 97.58 | 98.02 | 75.92 | 76.23 |
| XML_COMPACT | man | 11736 | 11610 | 127 | 98.92 | 98.78 | 76.53 | 76.43 |
| XML_COMPACT | opt | 8146 | 7993 | 153 | 98.12 | 98.45 | 86.40 | 86.63 |
| XML_PRETTY | man | 13382 | 13203 | 179 | 98.66 | 98.48 | 71.57 | 71.44 |
| XML_PRETTY | opt | 12314 | 12016 | 298 | 97.58 | 97.76 | 73.92 | 74.04 |
| YAML | man | 9771 | 9534 | 236 | 97.58 | 97.15 | 81.30 | 81.00 |
| YAML | opt | 11814 | 11528 | 286 | 97.58 | 97.92 | 75.37 | 75.61 |

#### 2.6.2 Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Tokens Man | Useful Tokens Opt | Diff | Diff (%) | Wasted Tokens Man | Wasted Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Man | Eff Score Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 4215 | 3926 | -289 | -6.85 | 4158 | 3831 | -327 | -7.86 | 57 | 95 |  +38 |  +66.87 | 98.65 | 97.58 | -1.07 | 98.19 | 98.277 |  +0.09 |  +0.09 |
| JSON_COMPACT | 6496 | 5964 | -532 | -8.19 | 6426 | 5820 | -606 | -9.44 | 70 | 144 |  +74 |  +105.94 | 98.92 | 97.58 | -1.34 | 91.75 | 92.36 |  +0.61 |  +0.66 |
| JSON_PRETTY | 14235 | 13412 | -823 | -5.78 | 14006 | 13160 | -846 | -6.04 | 229 | 252 |  +23 |  +10.02 | 98.39 | 98.12 | -0.27 | 68.90 | 71.104 |  +2.20 |  +3.20 |
| TOON_DEFAULT | 4220 | 11623 |  +7403 |  +175.43 | 4175 | 11342 |  +7167 |  +171.68 | 46 | 282 |  +236 |  +512.41 | 98.92 | 97.58 | -1.34 | 98.36 | 75.9205 | -22.44 | -22.82 |
| XML_COMPACT | 11736 | 8146 | -3590 | -30.59 | 11610 | 7993 | -3617 | -31.15 | 127 | 153 |  +26 |  +20.78 | 98.92 | 98.12 | -0.80 | 76.53 | 86.399 |  +9.87 |  +12.89 |
| XML_PRETTY | 13382 | 12314 | -1068 | -7.98 | 13203 | 12016 | -1187 | -8.99 | 179 | 298 |  +119 |  +66.30 | 98.66 | 97.58 | -1.08 | 71.57 | 73.916 |  +2.35 |  +3.28 |
| YAML | 9771 | 11814 |  +2043 |  +20.91 | 9534 | 11528 |  +1994 |  +20.91 | 236 | 285 |  +49 |  +20.95 | 97.58 | 97.58 | 0.00 | 81.30 | 75.367 | -5.94 | -7.30 |

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

## 3. Format-Specific Analysis
### 3.1 Detailed Analysis: CSV

#### 3.1.1 Performance Summary

- Token Duration Range: 351 - 371 seconds
- Token Cost Range: 3926 - 4215 tokens
- Wasted Token Range: 57 - 95 tokens
- Accuracy Range: 97.58 - 98.65%
- Efficiency Score Range: 98.19 - 98.28

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

- Token Duration Range: 304 - 356 seconds
- Token Cost Range: 5963 - 6496 tokens
- Wasted Token Range: 70 - 144 tokens
- Accuracy Range: 97.58 - 98.92%
- Efficiency Score Range: 91.75 - 92.36

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

- Token Duration Range: 279 - 338 seconds
- Token Cost Range: 13412 - 14235 tokens
- Wasted Token Range: 229 - 252 tokens
- Accuracy Range: 98.12 - 98.39%
- Efficiency Score Range: 68.90 - 71.10

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

- Token Duration Range: 160 - 168 seconds
- Token Cost Range: 4220 - 11623 tokens
- Wasted Token Range: 46 - 281 tokens
- Accuracy Range: 97.58 - 98.92%
- Efficiency Score Range: 75.92 - 98.36

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

- Token Duration Range: 328 - 340 seconds
- Token Cost Range: 8146 - 11736 tokens
- Wasted Token Range: 127 - 153 tokens
- Accuracy Range: 98.12 - 98.92%
- Efficiency Score Range: 76.53 - 86.40

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

- Token Duration Range: 265 - 317 seconds
- Token Cost Range: 12314 - 13382 tokens
- Wasted Token Range: 179 - 298 tokens
- Accuracy Range: 97.58 - 98.66%
- Efficiency Score Range: 71.57 - 73.92

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

- Token Duration Range: 309 - 340 seconds
- Token Cost Range: 9771 - 11814 tokens
- Wasted Token Range: 236 - 286 tokens
- Accuracy Range: 97.58 - 97.58%
- Efficiency Score Range: 75.37 - 81.30

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

- **Report Generated**: 2026-03-31
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code < 2.1.86
   - Note: All tests run with the same read tool overhead (line number format + system reminder)
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)