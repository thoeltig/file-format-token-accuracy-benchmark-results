# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: off
- **Data Structure**: nested
- **Formats Tested**: 6 (JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: First iteration

## Executive Summary

This benchmark evaluates token efficiency and information accuracy across 6 file formats using Claude Haiku 4.5 (claude-haiku-4-5-20251001) as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context while a format that accurately conveys information may justify higher token cost.

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

#### 1.2.1 Data Generation
- 6 formats tested: JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML
- 2 variants per format: mandatory (22 fields, dense) and optional (19 mandatory + 3 optional, sparse)
- Record Counts: 31

#### 1.2.2 Question Distribution
- 4 question categories reflecting practical use cases:
   - **Field Retrieval (55 questions, 37.50% weight):** Extract specific values from specific records
   - **Filtering (21 questions, 20.83% weight):** Count records matching criteria
   - **Aggregation (21 questions, 12.50% weight):** Sum, average, min/max calculations
   - **Structure Awareness (27 questions, 29.17% weight):** Understand data shape, organization, metadata

#### 1.2.3 Weighting Rationale
- **Field retrieval + structure awareness** = 66.67%
   - These represent the file format itself. Understanding "what data exists and how it's organized" which is fundamental to avoiding context confusion.
- **Filtering + aggregation** = 33.33%
   - These represent more the "intellectual" aspect of the model and will differ greatly depending on the model. Also if done deterministic the model still needs to do field retrieval and structure awareness on the result.

### 1.3 Metrics Definition

#### 1.3.1 Token Metrics
- **Read Tokens**: Tokens consumed reading the data file
- **Output Tokens**: Tokens consumed during inference (answering questions and creating the file content)
- **Total Tokens**: **Read Tokens** + **Output Tokens**

#### 1.3.2 Accuracy Metrics
- **Accuracy By Char**: Correct char per answers / expected characters per answer
- **Accuracy**: Correct answers / total questions
- **Weighted Accuracy**: Accuracy weighted by question category importance

#### 1.3.3 Efficiency Score
Composite metric balancing accuracy with normalized token count (favour towards accuracy). Each efficieny score has an indicator which token count was used in the calculation.
- **Normalized Tokens** = (((**Max Tokens** + 10) - **Current Tokens**) / ((**Max Tokens** + 10) - (**Min Tokens** - 10))) * 100
- **Efficiency Score**: (**Accuracy** % * 0.6666) + (**Normalized Tokens** * 0.3333)
- **Weighted Efficiency Score**: (**Weighted Accuracy** % * 0.6666) + (**Normalized Tokens** * 0.3333)

### 1.4 Token Usage Measurements

Tokens usage measured in this benchmark are no estimates but the real token usage the model used in this test. The token usage is reported to the user indirectly in the conversation transcript. Both read and output tokens are directly extracted from the transcripts of the subagents:
- **Read Tokens**: For each data file a single read subagent is invoked with the only prompt to read the file at the provided filepath and return "Done" once finished and do nothing more. The token extraction script searches for the read tool result and extracts only the read tokens of it.
- **Output Tokens**: For each data file multiple "benchmark-full-test" subagent are invoked with data, questions and answers template files and the instructions to read everything and answer all questions in a single write tool use. The token extraction script aggregates all output tokens until and including the write tool result.
   - **Output Before Write Tokens**: The output tokens which the model needed for reading the provided files and instructions.
   - **Output Write Tokens**: The output tokens the model used to create the output and write the answers file.

### 1.5 Important Note

These results are specific to Claude Code using the Claude Haiku 4.5 (claude-haiku-4-5-20251001) model. They serve as a rule of thumb for choosing the best file format depending on the use case.
However these values cannot be exactly applied to models of the same family or from other providers as token usage, accuracy and latency depend on specific model architectures and tokenizers. While the relative ranking of file formats remains consistent the absolute numbers will vary.
Especially the accuracy and output tokens results will vary because these values are bound to the model size and training, instruction interpretation and reasoning token budget.

## 2. Results

### 2.1 TLDR: Token Efficiency Analysis

*Note: All columns ranked best-to-worst. ↑ = lower value is better (ascending). ↓ = higher value is better (descending).*

#### 2.1.1 Best results

- Lowest total token cost:
   - Optional: JSON_COMPACT 21434 tokens
   - Mandatory: XML_COMPACT 18000 tokens
- Lowest read token cost:
   - Optional: JSON_COMPACT 9645 tokens
   - Mandatory: JSON_COMPACT 10163 tokens
- Lowest output token cost:
   - Optional: YAML 8089 tokens
   - Mandatory: XML_COMPACT 5295 tokens
- Lowest output token cost drift:
   - Optional: YAML ↓ -19.23% ↑ 12.49%
   - Mandatory: JSON_COMPACT ↓ -13.38% ↑ 19.90%
- Highest accuracy:
   - Optional: JSON_COMPACT 74.84%
   - Mandatory: XML_COMPACT 74.19%
- Lowest accuracy drift:
   - Optional: YAML ↓ -3.35% ↑ 2.93%
   - Mandatory: YAML ↓ -2.55% ↑ 1.83%
- Most useful read tokens:
   - Optional: XML_PRETTY 13431 / 19671 tokens
   - Mandatory: XML_PRETTY 13741 / 20204 tokens
- Most useful output tokens:
   - Optional: JSON_COMPACT 8823 / 11789 tokens
   - Mandatory: YAML 7388 / 10031 tokens
- Highest read efficiency (%/token):
   - Optional: JSON_COMPACT 83.19
   - Mandatory: JSON_COMPACT 80.59
- Highest output efficiency (%/token):
   - Optional: XML_PRETTY 67.38
   - Mandatory: XML_COMPACT 82.75
- Highest accuracy by char:
   - Optional: JSON_COMPACT 91.36%
   - Mandatory: JSON_COMPACT 93.04%
- Lowest accuracy by char drift:
   - Optional: YAML ↓ -1.18% ↑ 0.80%
   - Mandatory: JSON_PRETTY ↓ -0.94% ↑ 0.60%
- Most useful output write tokens (Acc By Char):
   - Optional: TOON_DEFAULT 10614 / 11654 tokens
   - Mandatory: YAML 9040 / 9819 tokens
- Highest output write efficiency (Acc By Char) (%/token):
   - Optional: YAML 82.07
   - Mandatory: XML_COMPACT 95.15
- Lowest delta (optional-mandatory):
   - Read tokens: YAML -7 tokens
   - Output tokens: XML_PRETTY -1643 tokens
   - Accuracy: XML_PRETTY 0.27%
   - Read efficiency: XML_COMPACT -0.29
   - Output efficiency: YAML 1.12
   - Accuracy by char: TOON_DEFAULT -1.10%
   - Output write efficiency (Acc By Char): XML_PRETTY 4.10

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: JSON_PRETTY 28491 tokens
   - Mandatory: XML_PRETTY 30149 tokens
- Highest read token cost:
   - Optional: XML_PRETTY 19671 tokens
   - Mandatory: XML_PRETTY 20204 tokens
- Highest output token cost:
   - Optional: TOON_DEFAULT 11912 tokens
   - Mandatory: YAML 10031 tokens
- Highest output token drift:
   - Optional: XML_PRETTY ↓ -40.86% ↑ 78.57%
   - Mandatory: XML_COMPACT ↓ -94.24% ↑ 80.26%
- Lowest accuracy:
   - Optional: YAML 64.25%
   - Mandatory: XML_PRETTY 68.01%
- Highest accuracy drift:
   - Optional: XML_PRETTY ↓ -7.88% ↑ 14.57%
   - Mandatory: TOON_DEFAULT ↓ -16.79% ↑ 8.74%
- Most wasted read tokens:
   - Optional: XML_PRETTY 6240 / 19671 tokens
   - Mandatory: XML_PRETTY 6463 / 20204 tokens
- Most wasted output tokens:
   - Optional: JSON_PRETTY 3249 / 11734 tokens
   - Mandatory: XML_PRETTY 3181 / 9945 tokens
- Lowest read efficiency (%/token):
   - Optional: XML_PRETTY 47.23
   - Mandatory: XML_PRETTY 45.37
- Lowest output efficiency (%/token):
   - Optional: JSON_PRETTY 57.02
   - Mandatory: XML_PRETTY 60.96
- Lowest accuracy by char:
   - Optional: XML_PRETTY 88.55%
   - Mandatory: XML_PRETTY 91.12%
- Highest accuracy by char drift:
   - Optional: JSON_PRETTY ↓ -3.66% ↑ 3.47%
   - Mandatory: XML_COMPACT ↓ -3.65% ↑ 3.80%
- Most wasted output write tokens (Acc By Char):
   - Optional: JSON_PRETTY 10316 / 11429 tokens
   - Mandatory: XML_PRETTY 8786 / 9643 tokens
- Lowest output write efficiency (Acc By Char) (%/token):
   - Optional: TOON_DEFAULT 68.94
   - Mandatory: YAML 76.51
- Highest delta (optional-mandatory):
   - Read tokens: JSON_PRETTY -925 tokens
   - Output tokens: XML_COMPACT 3993 tokens
   - Accuracy: YAML -9.40%
   - Read efficiency: YAML -6.24
   - Output efficiency: XML_COMPACT -16.25
   - Accuracy by char: YAML -3.12%
   - Output write efficiency (Acc By Char): XML_COMPACT -16.35

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy By Char | ↓ Eff Score Output Write (Acc By Char) | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 77s | JSON_COMPACT ≈ 10163 | YAML ≈ 213 | XML_COMPACT ≈ 4990 | XML_COMPACT ≈ 5295 | XML_COMPACT ≈ 18000 | JSON_COMPACT ≈ 93% | XML_COMPACT ≈ 95 | XML_COMPACT ≈ 74% | JSON_COMPACT ≈ 81 | XML_COMPACT ≈ 83 | XML_COMPACT ≈ 83 |
| XML_COMPACT (+3.9%) | XML_COMPACT (+25.0%) | XML_PRETTY (+42.1%) | TOON_DEFAULT (+72.5%) | TOON_DEFAULT (+68.4%) | JSON_COMPACT (+11.6%) | XML_COMPACT (-0.3%) | TOON_DEFAULT (-14.7%) | YAML (-0.5%) | XML_COMPACT (-9.3%) | TOON_DEFAULT (-17.8%) | JSON_COMPACT (-7.6%) |
| XML_PRETTY (+6.1%) | TOON_DEFAULT (+38.4%) | XML_COMPACT (+43.4%) | JSON_PRETTY (+79.1%) | JSON_PRETTY (+74.6%) | TOON_DEFAULT (+27.7%) | TOON_DEFAULT (-0.9%) | JSON_PRETTY (-16.3%) | JSON_COMPACT (-0.8%) | YAML (-15.4%) | JSON_PRETTY (-20.0%) | TOON_DEFAULT (-17.7%) |
| JSON_COMPACT (+6.6%) | YAML (+39.3%) | JSON_COMPACT (+43.5%) | JSON_COMPACT (+92.9%) | JSON_COMPACT (+87.5%) | YAML (+34.4%) | YAML (-1.0%) | JSON_COMPACT (-18.1%) | TOON_DEFAULT (-1.5%) | TOON_DEFAULT (-15.9%) | JSON_COMPACT (-21.9%) | YAML (-20.9%) |
| TOON_DEFAULT (+7.9%) | JSON_PRETTY (+74.0%) | JSON_PRETTY (+44.2%) | XML_PRETTY (+93.2%) | XML_PRETTY (+87.8%) | JSON_PRETTY (+49.6%) | JSON_PRETTY (-1.3%) | XML_PRETTY (-19.6%) | JSON_PRETTY (-2.3%) | JSON_PRETTY (-30.6%) | YAML (-22.2%) | JSON_PRETTY (-31.4%) |
| YAML (+11.8%) | XML_PRETTY (+98.8%) | TOON_DEFAULT (+45.3%) | YAML (+96.8%) | YAML (+89.4%) | XML_PRETTY (+67.5%) | XML_PRETTY (-1.9%) | YAML (-19.6%) | XML_PRETTY (-6.2%) | XML_PRETTY (-43.7%) | XML_PRETTY (-26.3%) | XML_PRETTY (-45.2%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy By Char | ↓ Eff Score Output Write (Acc By Char) | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 67s | JSON_COMPACT ≈ 9645 | XML_PRETTY ≈ 205 | YAML ≈ 7786 | YAML ≈ 8089 | JSON_COMPACT ≈ 21434 | JSON_COMPACT ≈ 91% | YAML ≈ 82 | JSON_COMPACT ≈ 75% | JSON_COMPACT ≈ 83 | XML_PRETTY ≈ 67 | JSON_COMPACT ≈ 74 |
| XML_PRETTY (+2.2%) | XML_COMPACT (+29.1%) | TOON_DEFAULT (+25.7%) | XML_PRETTY (+4.0%) | XML_PRETTY (+2.6%) | XML_COMPACT (+1.4%) | TOON_DEFAULT (-0.3%) | XML_PRETTY (-1.7%) | TOON_DEFAULT (-1.3%) | XML_COMPACT (-12.5%) | XML_COMPACT (-1.3%) | XML_COMPACT (-3.2%) |
| XML_COMPACT (+13.5%) | TOON_DEFAULT (+43.4%) | YAML (+47.6%) | XML_COMPACT (+15.3%) | XML_COMPACT (+14.8%) | YAML (+3.7%) | XML_COMPACT (-0.6%) | XML_COMPACT (-4.0%) | XML_COMPACT (-2.3%) | TOON_DEFAULT (-16.9%) | YAML (-2.8%) | YAML (-12.5%) |
| JSON_PRETTY (+39.9%) | YAML (+46.7%) | JSON_COMPACT (+48.7%) | JSON_PRETTY (+46.8%) | JSON_PRETTY (+45.1%) | TOON_DEFAULT (+20.1%) | JSON_PRETTY (-1.1%) | JSON_COMPACT (-15.0%) | JSON_PRETTY (-2.5%) | YAML (-25.5%) | JSON_COMPACT (-13.2%) | TOON_DEFAULT (-17.2%) |
| JSON_COMPACT (+40.8%) | JSON_PRETTY (+73.7%) | JSON_PRETTY (+48.8%) | JSON_COMPACT (+47.5%) | JSON_COMPACT (+45.8%) | XML_PRETTY (+30.5%) | YAML (-2.4%) | JSON_PRETTY (-15.6%) | XML_PRETTY (-6.6%) | JSON_PRETTY (-29.0%) | TOON_DEFAULT (-15.2%) | JSON_PRETTY (-28.5%) |
| TOON_DEFAULT (+41.0%) | XML_PRETTY (+104.0%) | XML_COMPACT (+49.8%) | TOON_DEFAULT (+49.7%) | TOON_DEFAULT (+47.3%) | JSON_PRETTY (+32.9%) | XML_PRETTY (-2.8%) | TOON_DEFAULT (-16.0%) | YAML (-10.6%) | XML_PRETTY (-43.2%) | JSON_PRETTY (-15.4%) | XML_PRETTY (-30.2%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99% | XML_COMPACT ≈ 59% | YAML ≈ 62% | XML_COMPACT ≈ 68% |
| JSON_COMPACT (-3.0%) | JSON_PRETTY (-0.0%) | JSON_PRETTY (-1.9%) | JSON_COMPACT (-8.3%) |
| TOON_DEFAULT (-5.8%) | TOON_DEFAULT (-6.0%) | JSON_COMPACT (-3.8%) | TOON_DEFAULT (-10.5%) |
| JSON_PRETTY (-7.8%) | YAML (-6.2%) | TOON_DEFAULT (-4.2%) | XML_PRETTY (-19.0%) |
| XML_COMPACT (-8.5%) | XML_PRETTY (-7.4%) | XML_COMPACT (-6.3%) | JSON_PRETTY (-19.7%) |
| XML_PRETTY (-10.9%) | JSON_COMPACT (-10.4%) | XML_PRETTY (-7.9%) | YAML (-23.8%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 98% | XML_COMPACT ≈ 63% | JSON_PRETTY ≈ 68% | JSON_COMPACT ≈ 53% |
| JSON_PRETTY (-2.4%) | TOON_DEFAULT (-2.1%) | TOON_DEFAULT (-1.6%) | TOON_DEFAULT (-8.4%) |
| XML_COMPACT (-3.6%) | XML_PRETTY (-8.6%) | JSON_COMPACT (-4.4%) | XML_PRETTY (-12.1%) |
| TOON_DEFAULT (-4.9%) | JSON_PRETTY (-9.9%) | XML_PRETTY (-6.3%) | YAML (-13.6%) |
| XML_PRETTY (-10.3%) | JSON_COMPACT (-10.4%) | YAML (-6.3%) | JSON_PRETTY (-13.7%) |
| YAML (-15.8%) | YAML (-14.8%) | XML_COMPACT (-6.4%) | XML_COMPACT (-15.2%) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy By Char (%) | Useful Output Write Tokens (Acc By Char) | Wasted Output Write Tokens (Acc By Char) | Eff Score Output Write (Acc By Char) | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10163 | 9930 | 20093 | 2.246 | 77.618 | 93.04 | 8954.73 | 669.87 | 77.89 | 73.39 | 7458.626 | 2704.374 | 7287.480 | 2642.320 | 80.59 | 64.60 | 76.49 |
| JSON_COMPACT | opt | 9645 | 11789 | 21434 | 2.219 | 92.618 | 91.36 | 10492.33 | 992.27 | 69.77 | 74.84 | 7218.318 | 2426.682 | 8823.187 | 2966.213 | 83.19 | 58.50 | 73.78 |
| JSON_PRETTY | man | 17682 | 9243 | 26925 | 1.777 | 72.069 | 91.76 | 8200.22 | 736.38 | 79.62 | 71.93 | 12718.663 | 4963.337 | 6648.634 | 2594.566 | 55.93 | 66.24 | 56.80 |
| JSON_PRETTY | opt | 16757 | 11734 | 28491 | 1.767 | 92.172 | 90.26 | 10316.12 | 1113.22 | 69.25 | 72.31 | 12116.987 | 4640.013 | 8485.096 | 3249.237 | 59.09 | 57.02 | 52.77 |
| TOON_DEFAULT | man | 14068 | 8915 | 22983 | 1.855 | 69.401 | 92.17 | 7931.87 | 673.83 | 81.14 | 72.68 | 10224.622 | 3843.378 | 6479.229 | 2435.505 | 67.81 | 67.98 | 68.10 |
| TOON_DEFAULT | opt | 13832 | 11912 | 25744 | 1.864 | 93.986 | 91.07 | 10613.53 | 1040.72 | 68.94 | 73.50 | 10166.520 | 3665.480 | 8755.259 | 3156.658 | 69.10 | 57.14 | 61.09 |
| XML_COMPACT | man | 12705 | 5295 | 18000 | 2.551 | 40.242 | 92.79 | 4630.22 | 359.78 | 95.15 | 74.19 | 9425.840 | 3279.161 | 3928.361 | 1366.640 | 73.11 | 82.75 | 82.76 |
| XML_COMPACT | opt | 12455 | 9288 | 21743 | 2.499 | 72.423 | 90.77 | 8151.60 | 828.90 | 78.80 | 72.58 | 9039.839 | 3415.161 | 6740.868 | 2546.633 | 72.83 | 66.50 | 71.43 |
| XML_PRETTY | man | 20204 | 9945 | 30149 | 1.985 | 77.762 | 91.12 | 8786.25 | 856.25 | 76.54 | 68.01 | 13740.740 | 6463.260 | 6763.424 | 3181.326 | 45.37 | 60.96 | 45.36 |
| XML_PRETTY | opt | 19671 | 8301 | 27972 | 1.974 | 65.293 | 88.55 | 7169.30 | 927.03 | 80.64 | 68.28 | 13431.359 | 6239.641 | 5668.150 | 2633.183 | 47.23 | 67.38 | 51.50 |
| YAML | man | 14155 | 10031 | 24186 | 1.808 | 79.183 | 92.07 | 9040.05 | 778.62 | 76.51 | 73.65 | 10425.158 | 3729.842 | 7388.077 | 2643.256 | 68.18 | 64.39 | 65.45 |
| YAML | opt | 14148 | 8089 | 22237 | 1.787 | 62.790 | 88.95 | 6925.65 | 860.35 | 82.07 | 64.25 | 9090.090 | 5057.910 | 5196.969 | 2891.698 | 61.94 | 65.50 | 64.53 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10163 | 9645 | -518 | -5.10 | 305 | 305 | 0 | 0.00 | 9625 | 11485 |  +1860 |  +19.32 | 9930 | 11790 |  +1860 |  +18.73 | 20093 | 21435 |  +1342 |  +6.68 |
| JSON_PRETTY | 17682 | 16757 | -925 | -5.23 | 307 | 305 | -2 | -0.65 | 8937 | 11430 |  +2493 |  +27.90 | 9243 | 11734 |  +2491 |  +26.95 | 26925 | 28491 |  +1566 |  +5.82 |
| TOON_DEFAULT | 14068 | 13832 | -236 | -1.68 | 309 | 258 | -51 | -16.50 | 8606 | 11655 |  +3049 |  +35.43 | 8915 | 11912 |  +2997 |  +33.62 | 22983 | 25744 |  +2761 |  +12.01 |
| XML_COMPACT | 12705 | 12455 | -250 | -1.97 | 305 | 307 |  +2 |  +0.66 | 4990 | 8981 |  +3991 |  +79.98 | 5295 | 9288 |  +3993 |  +75.41 | 18000 | 21743 |  +3743 |  +20.79 |
| XML_PRETTY | 20204 | 19671 | -533 | -2.64 | 302 | 205 | -97 | -32.12 | 9643 | 8097 | -1546 | -16.03 | 9945 | 8302 | -1643 | -16.52 | 30149 | 27973 | -2176 | -7.22 |
| YAML | 14155 | 14148 | -7 | -0.05 | 213 | 303 |  +90 |  +42.25 | 9819 | 7786 | -2033 | -20.70 | 10031 | 8088 | -1943 | -19.37 | 24186 | 22236 | -1950 | -8.06 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 13 | 781.769 | 0.42 | 35860 | 0.268 | 289.20 | 35873 | 782.037 | 231.44 |
| JSON_COMPACT | opt | 9 | 1071.667 | 0.29 | 38516 | 0.298 | 310.61 | 38525 | 1071.965 | 248.55 |
| JSON_PRETTY | man | 262 | 67.489 | 8.45 | 37358 | 0.239 | 301.27 | 37620 | 67.728 | 242.71 |
| JSON_PRETTY | opt | 270 | 62.063 | 8.71 | 37045 | 0.309 | 298.75 | 37315 | 62.372 | 240.74 |
| TOON_DEFAULT | man | 31 | 453.806 | 1.00 | 38540 | 0.229 | 310.81 | 38571 | 454.035 | 248.85 |
| TOON_DEFAULT | opt | 29 | 476.966 | 0.94 | 37891 | 0.321 | 305.58 | 37920 | 477.287 | 244.65 |
| XML_COMPACT | man | 13 | 977.308 | 0.42 | 41086 | 0.121 | 331.34 | 41099 | 977.429 | 265.16 |
| XML_COMPACT | opt | 12 | 1037.917 | 0.39 | 40983 | 0.219 | 330.50 | 40995 | 1038.136 | 264.48 |
| XML_PRETTY | man | 13 | 1554.154 | 0.42 | 38004 | 0.254 | 306.49 | 38017 | 1554.408 | 245.27 |
| XML_PRETTY | opt | 6 | 3278.500 | 0.19 | 40578 | 0.200 | 327.24 | 40584 | 3278.700 | 261.83 |
| YAML | man | 12 | 1179.583 | 0.39 | 37690 | 0.261 | 303.95 | 37702 | 1179.844 | 243.24 |
| YAML | opt | 11 | 1286.182 | 0.35 | 36048 | 0.216 | 290.71 | 36059 | 1286.398 | 232.64 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 13 | 9 | -4 | -30.77 | 35.86 | 38.52 |  +2.66 |  +7.41 | 35.87 | 38.52 |  +2.65 |  +7.39 |
| JSON_PRETTY | 262 | 270 |  +8 |  +3.05 | 37.36 | 37.05 | -0.31 | -0.84 | 37.62 | 37.32 | -0.30 | -0.81 |
| TOON_DEFAULT | 31 | 29 | -2 | -6.45 | 38.54 | 37.89 | -0.65 | -1.68 | 38.57 | 37.92 | -0.65 | -1.69 |
| XML_COMPACT | 13 | 12 | -1 | -7.69 | 41.09 | 40.98 | -0.10 | -0.25 | 41.10 | 40.99 | -0.10 | -0.26 |
| XML_PRETTY | 13 | 6 | -7 | -53.85 | 38.00 | 40.58 |  +2.57 |  +6.77 | 38.02 | 40.58 |  +2.57 |  +6.75 |
| YAML | 12 | 11 | -1 | -8.33 | 37.69 | 36.05 | -1.64 | -4.36 | 37.70 | 36.06 | -1.64 | -4.36 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token |
|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 2.246 | 14.902 | 327.839 | 0.722 | 0.739 | 0.365 |
| JSON_COMPACT | opt | 2.219 | 15.285 | 311.129 | 0.776 | 0.635 | 0.349 |
| JSON_PRETTY | man | 1.777 | 25.927 | 570.387 | 0.407 | 0.778 | 0.267 |
| JSON_PRETTY | opt | 1.767 | 26.556 | 540.548 | 0.432 | 0.616 | 0.254 |
| TOON_DEFAULT | man | 1.855 | 20.628 | 453.806 | 0.517 | 0.839 | 0.318 |
| TOON_DEFAULT | opt | 1.864 | 21.921 | 446.194 | 0.531 | 0.637 | 0.287 |
| XML_COMPACT | man | 2.551 | 18.629 | 409.839 | 0.584 | 1.401 | 0.412 |
| XML_COMPACT | opt | 2.499 | 19.739 | 401.774 | 0.583 | 0.781 | 0.334 |
| XML_PRETTY | man | 1.985 | 29.625 | 651.742 | 0.337 | 0.684 | 0.226 |
| XML_PRETTY | opt | 1.974 | 31.174 | 634.548 | 0.347 | 0.823 | 0.244 |
| YAML | man | 1.808 | 20.755 | 456.613 | 0.520 | 0.734 | 0.305 |
| YAML | opt | 1.787 | 22.422 | 456.387 | 0.454 | 0.794 | 0.289 |

#### 2.5.2 Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 2.246 | 2.219 | -0.027 | -1.20 | 14.902 | 15.285 |  +0.383 |  +2.57 | 327.839 | 311.129 | -16.710 | -5.10 | 0.722 | 0.776 |  +0.054 |  +7.48 | 0.739 | 0.635 | -0.104 | -14.07 | 0.365 | 0.349 | -0.016 | -4.38 |
| JSON_PRETTY | 1.777 | 1.767 | -0.010 | -0.56 | 25.927 | 26.556 |  +0.629 |  +2.43 | 570.387 | 540.548 | -29.839 | -5.23 | 0.407 | 0.432 |  +0.025 |  +6.14 | 0.778 | 0.616 | -0.162 | -20.82 | 0.267 | 0.254 | -0.013 | -4.87 |
| TOON_DEFAULT | 1.855 | 1.864 |  +0.009 |  +0.49 | 20.628 | 21.921 |  +1.293 |  +6.27 | 453.806 | 446.194 | -7.612 | -1.68 | 0.517 | 0.531 |  +0.014 |  +2.71 | 0.839 | 0.637 | -0.201 | -23.97 | 0.318 | 0.287 | -0.030 | -9.45 |
| XML_COMPACT | 2.551 | 2.499 | -0.052 | -2.04 | 18.629 | 19.739 |  +1.110 |  +5.96 | 409.839 | 401.774 | -8.065 | -1.97 | 0.584 | 0.583 | -0.001 | -0.17 | 1.401 | 0.781 | -0.620 | -44.25 | 0.412 | 0.334 | -0.078 | -18.93 |
| XML_PRETTY | 1.985 | 1.974 | -0.011 | -0.55 | 29.625 | 31.174 |  +1.549 |  +5.23 | 651.742 | 634.548 | -17.194 | -2.64 | 0.337 | 0.347 |  +0.010 |  +2.97 | 0.684 | 0.823 |  +0.139 |  +20.32 | 0.226 | 0.244 |  +0.018 |  +7.96 |
| YAML | 1.808 | 1.787 | -0.021 | -1.16 | 20.755 | 22.422 |  +1.667 |  +8.03 | 456.613 | 456.387 | -0.226 | -0.05 | 0.520 | 0.454 | -0.066 | -12.69 | 0.734 | 0.794 |  +0.060 |  +8.17 | 0.305 | 0.289 | -0.016 | -5.25 |

### 2.6 Output Write Token Utilization Efficiency (Accuracy By Char)
#### 2.6.1 Metrics
| Format | Variant | Output Write Tokens | Useful Output Write Tokens (Acc By Char) | Wasted Output Write Tokens (Acc By Char) | Accuracy by Char (%) | Eff Score Output Write (Acc By Char) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 9625 | 8955 | 670 | 93.04 | 77.89 |
| JSON_COMPACT | opt | 11485 | 10492 | 992 | 91.36 | 69.77 |
| JSON_PRETTY | man | 8937 | 8200 | 736 | 91.76 | 79.62 |
| JSON_PRETTY | opt | 11429 | 10316 | 1113 | 90.26 | 69.25 |
| TOON_DEFAULT | man | 8606 | 7932 | 674 | 92.17 | 81.14 |
| TOON_DEFAULT | opt | 11654 | 10614 | 1041 | 91.07 | 68.94 |
| XML_COMPACT | man | 4990 | 4630 | 360 | 92.79 | 95.15 |
| XML_COMPACT | opt | 8981 | 8152 | 829 | 90.77 | 78.80 |
| XML_PRETTY | man | 9643 | 8786 | 856 | 91.12 | 76.54 |
| XML_PRETTY | opt | 8096 | 7169 | 927 | 88.55 | 80.64 |
| YAML | man | 9819 | 9040 | 779 | 92.07 | 76.51 |
| YAML | opt | 7786 | 6926 | 860 | 88.95 | 82.07 |

#### 2.6.2 Mandatory vs Optional
| Format | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Useful Output Write Tokens (Acc By Char) Man | Useful Output Write Tokens (Acc By Char) Opt | Diff | Diff (%) | Wasted Output Write Tokens (Acc By Char) Man | Wasted Output Write Tokens (Acc By Char) Opt | Diff | Diff (%) | Accuracy By Char (%) Man | Accuracy By Char (%) Opt | Diff (%) | Eff Score Output Write (Acc By Char) Man | Eff Score Output Write (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 9625 | 11485 |  +1860 |  +19.32 | 8955 | 10493 |  +1538 |  +17.17 | 670 | 992 |  +322 |  +48.12 | 93.04 | 91.36 | -1.68 | -1.81 | 77.89 | 69.77 | -8.11 | -10.42 |
| JSON_PRETTY | 8937 | 11430 |  +2493 |  +27.89 | 8200 | 10316 |  +2116 |  +25.80 | 736 | 1113 |  +377 |  +51.20 | 91.76 | 90.26 | -1.50 | -1.63 | 79.62 | 69.25 | -10.37 | -13.03 |
| TOON_DEFAULT | 8606 | 11655 |  +3049 |  +35.42 | 7932 | 10614 |  +2682 |  +33.81 | 674 | 1041 |  +367 |  +54.44 | 92.17 | 91.07 | -1.10 | -1.19 | 81.14 | 68.94 | -12.20 | -15.03 |
| XML_COMPACT | 4990 | 8981 |  +3991 |  +79.97 | 4630 | 8151 |  +3521 |  +76.06 | 360 | 829 |  +469 |  +130.31 | 92.79 | 90.77 | -2.02 | -2.18 | 95.15 | 78.80 | -16.35 | -17.19 |
| XML_PRETTY | 9643 | 8097 | -1546 | -16.03 | 8786 | 7169 | -1617 | -18.40 | 856 | 927 |  +71 |  +8.27 | 91.12 | 88.55 | -2.57 | -2.82 | 76.54 | 80.64 |  +4.10 |  +5.36 |
| YAML | 9819 | 7786 | -2033 | -20.70 | 9040 | 6926 | -2114 | -23.39 | 779 | 861 |  +82 |  +10.49 | 92.07 | 88.95 | -3.12 | -3.39 | 76.51 | 82.07 |  +5.56 |  +7.27 |

### 2.7 Token Utilization Efficiency
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10163 | 7459 | 2704 | 9930 | 7287 | 2642 | 20093 | 14746 | 5347 | 73.39 | 80.59 | 64.60 | 76.49 | 70.00 | 78.33 | 62.34 | 74.23 |
| JSON_COMPACT | opt | 9645 | 7218 | 2427 | 11789 | 8823 | 2966 | 21434 | 16042 | 5393 | 74.84 | 83.19 | 58.50 | 73.78 | 72.11 | 81.37 | 56.68 | 71.96 |
| JSON_PRETTY | man | 17682 | 12719 | 4963 | 9243 | 6649 | 2595 | 26925 | 19367 | 7558 | 71.93 | 55.93 | 66.24 | 56.80 | 70.21 | 54.78 | 65.09 | 55.66 |
| JSON_PRETTY | opt | 16757 | 12117 | 4640 | 11734 | 8485 | 3249 | 28491 | 20602 | 7889 | 72.31 | 59.09 | 57.02 | 52.77 | 70.57 | 57.93 | 55.86 | 51.61 |
| TOON_DEFAULT | man | 14068 | 10225 | 3843 | 8915 | 6479 | 2436 | 22983 | 16704 | 6279 | 72.68 | 67.81 | 67.98 | 68.10 | 69.88 | 65.94 | 66.12 | 66.24 |
| TOON_DEFAULT | opt | 13832 | 10167 | 3665 | 11912 | 8755 | 3157 | 25744 | 18922 | 6822 | 73.50 | 69.10 | 57.14 | 61.09 | 72.21 | 68.24 | 56.28 | 60.23 |
| XML_COMPACT | man | 12705 | 9426 | 3279 | 5295 | 3928 | 1367 | 18000 | 13354 | 4646 | 74.19 | 73.11 | 82.75 | 82.76 | 71.47 | 71.30 | 80.93 | 80.94 |
| XML_COMPACT | opt | 12455 | 9040 | 3415 | 9288 | 6741 | 2547 | 21743 | 15781 | 5962 | 72.58 | 72.83 | 66.50 | 71.43 | 71.47 | 72.09 | 65.76 | 70.69 |
| XML_PRETTY | man | 20204 | 13741 | 6463 | 9945 | 6763 | 3181 | 30149 | 20504 | 9645 | 68.01 | 45.37 | 60.96 | 45.36 | 65.70 | 43.83 | 59.42 | 43.82 |
| XML_PRETTY | opt | 19671 | 13431 | 6240 | 8301 | 5668 | 2633 | 27972 | 19100 | 8873 | 68.28 | 47.23 | 67.38 | 51.50 | 66.85 | 46.27 | 66.43 | 50.55 |
| YAML | man | 14155 | 10425 | 3730 | 10031 | 7388 | 2643 | 24186 | 17813 | 6373 | 73.65 | 68.18 | 64.39 | 65.45 | 71.20 | 66.55 | 62.75 | 63.82 |
| YAML | opt | 14148 | 9090 | 5058 | 8089 | 5197 | 2892 | 22237 | 14287 | 7950 | 64.25 | 61.94 | 65.50 | 64.53 | 62.81 | 60.98 | 64.54 | 63.57 |

#### 2.7.2 Read Tokens Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10163 | 9645 | -518 | -5.10 | 7459 | 7219 | -240 | -3.22 | 2704 | 2426 | -278 | -10.27 | 73.39 | 74.84 |  +1.45 |  +1.98 | 80.59 | 83.19 |  +2.60 |  +3.23 | 70.00 | 72.11 |  +2.11 |  +3.01 | 78.33 | 81.37 |  +3.04 |  +3.88 |
| JSON_PRETTY | 17682 | 16757 | -925 | -5.23 | 12719 | 12117 | -602 | -4.73 | 4963 | 4640 | -323 | -6.51 | 71.93 | 72.31 |  +0.38 |  +0.53 | 55.93 | 59.09 |  +3.17 |  +5.66 | 70.21 | 70.57 |  +0.36 |  +0.51 | 54.78 | 57.93 |  +3.15 |  +5.76 |
| TOON_DEFAULT | 14068 | 13832 | -236 | -1.68 | 10225 | 10167 | -58 | -0.57 | 3843 | 3665 | -178 | -4.63 | 72.68 | 73.50 |  +0.82 |  +1.13 | 67.81 | 69.10 |  +1.29 |  +1.90 | 69.88 | 72.21 |  +2.33 |  +3.33 | 65.94 | 68.24 |  +2.30 |  +3.48 |
| XML_COMPACT | 12705 | 12455 | -250 | -1.97 | 9426 | 9040 | -386 | -4.10 | 3279 | 3415 |  +136 |  +4.15 | 74.19 | 72.58 | -1.61 | -2.17 | 73.11 | 72.83 | -0.29 | -0.39 | 71.47 | 71.47 | 0.00 | 0.00 | 71.30 | 72.09 |  +0.79 |  +1.10 |
| XML_PRETTY | 20204 | 19671 | -533 | -2.64 | 13741 | 13432 | -309 | -2.25 | 6463 | 6239 | -224 | -3.46 | 68.01 | 68.28 |  +0.27 |  +0.40 | 45.37 | 47.23 |  +1.86 |  +4.10 | 65.70 | 66.85 |  +1.15 |  +1.75 | 43.83 | 46.27 |  +2.45 |  +5.58 |
| YAML | 14155 | 14148 | -7 | -0.05 | 10425 | 9090 | -1335 | -12.81 | 3730 | 5058 |  +1328 |  +35.61 | 73.65 | 64.25 | -9.40 | -12.76 | 68.18 | 61.94 | -6.24 | -9.16 | 71.20 | 62.81 | -8.39 | -11.78 | 66.55 | 60.98 | -5.57 | -8.37 |

#### 2.7.3 Output Tokens Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 9930 | 11790 |  +1860 |  +18.73 | 7287 | 8823 |  +1536 |  +21.07 | 2642 | 2966 |  +324 |  +12.26 | 73.39 | 74.84 |  +1.45 |  +1.98 | 64.60 | 58.50 | -6.10 | -9.44 | 70.00 | 72.11 |  +2.11 |  +3.01 | 62.34 | 56.68 | -5.66 | -9.08 |
| JSON_PRETTY | 9243 | 11734 |  +2491 |  +26.95 | 6649 | 8485 |  +1836 |  +27.62 | 2595 | 3250 |  +655 |  +25.23 | 71.93 | 72.31 |  +0.38 |  +0.53 | 66.24 | 57.02 | -9.21 | -13.91 | 70.21 | 70.57 |  +0.36 |  +0.51 | 65.09 | 55.86 | -9.23 | -14.18 |
| TOON_DEFAULT | 8915 | 11912 |  +2997 |  +33.62 | 6479 | 8755 |  +2276 |  +35.13 | 2436 | 3157 |  +721 |  +29.60 | 72.68 | 73.50 |  +0.82 |  +1.13 | 67.98 | 57.14 | -10.84 | -15.95 | 69.88 | 72.21 |  +2.33 |  +3.33 | 66.12 | 56.28 | -9.84 | -14.88 |
| XML_COMPACT | 5295 | 9288 |  +3993 |  +75.40 | 3928 | 6741 |  +2813 |  +71.60 | 1367 | 2547 |  +1180 |  +86.32 | 74.19 | 72.58 | -1.61 | -2.17 | 82.75 | 66.50 | -16.25 | -19.63 | 71.47 | 71.47 | 0.00 | 0.00 | 80.93 | 65.76 | -15.17 | -18.75 |
| XML_PRETTY | 9945 | 8302 | -1643 | -16.53 | 6763 | 5668 | -1095 | -16.20 | 3181 | 2633 | -548 | -17.23 | 68.01 | 68.28 |  +0.27 |  +0.40 | 60.96 | 67.38 |  +6.43 |  +10.54 | 65.70 | 66.85 |  +1.15 |  +1.75 | 59.42 | 66.43 |  +7.01 |  +11.80 |
| YAML | 10031 | 8088 | -1943 | -19.37 | 7388 | 5197 | -2191 | -29.66 | 2643 | 2891 |  +248 |  +9.40 | 73.65 | 64.25 | -9.40 | -12.76 | 64.39 | 65.50 |  +1.12 |  +1.73 | 71.20 | 62.81 | -8.39 | -11.78 | 62.75 | 64.54 |  +1.79 |  +2.85 |

#### 2.7.4 Total Tokens Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 20093 | 21435 |  +1342 |  +6.68 | 14746 | 16041 |  +1295 |  +8.78 | 5347 | 5393 |  +46 |  +0.86 | 73.39 | 74.84 |  +1.45 |  +1.98 | 76.49 | 73.78 | -2.71 | -3.54 | 70.00 | 72.11 |  +2.11 |  +3.01 | 74.23 | 71.96 | -2.27 | -3.06 |
| JSON_PRETTY | 26925 | 28491 |  +1566 |  +5.82 | 19367 | 20602 |  +1235 |  +6.38 | 7558 | 7889 |  +331 |  +4.38 | 71.93 | 72.31 |  +0.38 |  +0.53 | 56.80 | 52.77 | -4.04 | -7.11 | 70.21 | 70.57 |  +0.36 |  +0.51 | 55.66 | 51.61 | -4.05 | -7.28 |
| TOON_DEFAULT | 22983 | 25744 |  +2761 |  +12.01 | 16704 | 18922 |  +2218 |  +13.28 | 6279 | 6822 |  +543 |  +8.65 | 72.68 | 73.50 |  +0.82 |  +1.13 | 68.10 | 61.09 | -7.02 | -10.30 | 69.88 | 72.21 |  +2.33 |  +3.33 | 66.24 | 60.23 | -6.01 | -9.07 |
| XML_COMPACT | 18000 | 21743 |  +3743 |  +20.79 | 13354 | 15781 |  +2427 |  +18.17 | 4646 | 5962 |  +1316 |  +28.33 | 74.19 | 72.58 | -1.61 | -2.17 | 82.76 | 71.43 | -11.32 | -13.68 | 71.47 | 71.47 | 0.00 | 0.00 | 80.94 | 70.69 | -10.25 | -12.66 |
| XML_PRETTY | 30149 | 27973 | -2176 | -7.22 | 20504 | 19099 | -1405 | -6.85 | 9645 | 8873 | -772 | -8.00 | 68.01 | 68.28 |  +0.27 |  +0.40 | 45.36 | 51.50 |  +6.14 |  +13.54 | 65.70 | 66.85 |  +1.15 |  +1.75 | 43.82 | 50.55 |  +6.73 |  +15.35 |
| YAML | 24186 | 22236 | -1950 | -8.06 | 17813 | 14287 | -3526 | -19.80 | 6373 | 7950 |  +1577 |  +24.74 | 73.65 | 64.25 | -9.40 | -12.76 | 65.45 | 64.53 | -0.92 | -1.41 | 71.20 | 62.81 | -8.39 | -11.78 | 63.82 | 63.57 | -0.25 | -0.39 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Accuracy by Char (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 91.00 | 33.00 | 0.00 | 73.39 | 93.04 |
| JSON_COMPACT | opt | 92.80 | 31.20 | 0.00 | 74.84 | 91.36 |
| JSON_PRETTY | man | 89.20 | 34.80 | 0.00 | 71.93 | 91.76 |
| JSON_PRETTY | opt | 89.67 | 34.33 | 0.00 | 72.31 | 90.26 |
| TOON_DEFAULT | man | 90.13 | 33.88 | -0.01 | 72.68 | 92.17 |
| TOON_DEFAULT | opt | 91.14 | 32.86 | 0.00 | 73.50 | 91.07 |
| XML_COMPACT | man | 92.00 | 32.00 | 0.00 | 74.19 | 92.79 |
| XML_COMPACT | opt | 90.00 | 34.00 | 0.00 | 72.58 | 90.77 |
| XML_PRETTY | man | 84.33 | 39.67 | 0.00 | 68.01 | 91.12 |
| XML_PRETTY | opt | 84.67 | 39.33 | 0.00 | 68.28 | 88.55 |
| YAML | man | 91.33 | 32.67 | 0.00 | 73.65 | 92.07 |
| YAML | opt | 79.67 | 44.33 | 0.00 | 64.25 | 88.95 |

#### 2.8.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Accuracy by Char (%) Man | Accuracy by Char (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 91.00 | 92.80 |  +2 |  +1.98 | 33.00 | 31.20 | -2 | -5.45 | 0.00 | 0.00 |  +0 | 0.00 | 73.39 | 74.84 |  +1.45 | 93.04 | 91.36 |  +93.04 |
| JSON_PRETTY | 89.20 | 89.67 |  +0 |  +0.53 | 34.80 | 34.33 | -0 | -1.35 | 0.00 | 0.00 | 0 | 0.00 | 71.93 | 72.31 |  +0.38 | 91.76 | 90.26 |  +91.76 |
| TOON_DEFAULT | 90.13 | 91.14 |  +1 |  +1.12 | 33.88 | 32.86 | -1 | -3.01 | -0.01 | 0.00 |  +0 | 0.00 | 72.68 | 73.50 |  +0.82 | 92.17 | 91.07 |  +92.17 |
| XML_COMPACT | 92.00 | 90.00 | -2 | -2.17 | 32.00 | 34.00 |  +2 |  +6.25 | 0.00 | 0.00 | 0 | 0.00 | 74.19 | 72.58 | -1.61 | 92.79 | 90.77 |  +92.79 |
| XML_PRETTY | 84.33 | 84.67 |  +0 |  +0.40 | 39.67 | 39.33 | -0 | -0.86 | 0.00 | 0.00 | 0 | 0.00 | 68.01 | 68.28 |  +0.27 | 91.12 | 88.55 |  +91.12 |
| YAML | 91.33 | 79.67 | -12 | -12.77 | 32.67 | 44.33 |  +12 |  +35.69 | 0.00 | 0.00 | 0 | 0.00 | 73.65 | 64.25 | -9.40 | 92.07 | 88.95 |  +92.07 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 73.39 | 96.36 | 48.89 | 58.09 | 60.00 |
| JSON_COMPACT | opt | 74.84 | 98.18 | 52.59 | 63.81 | 53.33 |
| JSON_PRETTY | man | 71.93 | 91.64 | 59.26 | 60.00 | 48.57 |
| JSON_PRETTY | opt | 72.31 | 95.76 | 53.08 | 68.25 | 39.68 |
| TOON_DEFAULT | man | 72.68 | 93.64 | 53.24 | 57.74 | 57.74 |
| TOON_DEFAULT | opt | 73.50 | 93.25 | 60.85 | 66.67 | 44.90 |
| XML_COMPACT | man | 74.19 | 90.91 | 59.26 | 55.55 | 68.25 |
| XML_COMPACT | opt | 72.58 | 94.55 | 62.96 | 61.90 | 38.10 |
| XML_PRETTY | man | 68.01 | 88.48 | 51.85 | 53.97 | 49.21 |
| XML_PRETTY | opt | 68.28 | 87.88 | 54.32 | 61.90 | 41.27 |
| YAML | man | 73.65 | 99.39 | 53.09 | 61.90 | 44.45 |
| YAML | opt | 64.25 | 82.42 | 48.15 | 61.90 | 39.68 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 96.36 | 98.18 |  +1.82 |
| JSON_PRETTY | 91.64 | 95.76 |  +4.12 |
| TOON_DEFAULT | 93.64 | 93.25 | -0.39 |
| XML_COMPACT | 90.91 | 94.55 |  +3.64 |
| XML_PRETTY | 88.48 | 87.88 | -0.61 |
| YAML | 99.39 | 82.42 | -16.97 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 48.89 | 52.59 |  +3.70 |
| JSON_PRETTY | 59.26 | 53.08 | -6.17 |
| TOON_DEFAULT | 53.24 | 60.85 |  +7.61 |
| XML_COMPACT | 59.26 | 62.96 |  +3.70 |
| XML_PRETTY | 51.85 | 54.32 |  +2.47 |
| YAML | 53.09 | 48.15 | -4.94 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 58.09 | 63.81 |  +5.71 |
| JSON_PRETTY | 60.00 | 68.25 |  +8.26 |
| TOON_DEFAULT | 57.74 | 66.67 |  +8.93 |
| XML_COMPACT | 55.55 | 61.90 |  +6.35 |
| XML_PRETTY | 53.97 | 61.90 |  +7.94 |
| YAML | 61.90 | 61.90 |  +0.00 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 60.00 | 53.33 | -6.67 |
| JSON_PRETTY | 48.57 | 39.68 | -8.89 |
| TOON_DEFAULT | 57.74 | 44.90 | -12.84 |
| XML_COMPACT | 68.25 | 38.10 | -30.16 |
| XML_PRETTY | 49.21 | 41.27 | -7.93 |
| YAML | 44.45 | 39.68 | -4.76 |

## 3. Appendices

### 3.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: off
- **Structure**: nested
- **Formats Tested**: JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML
- **Record Counts**: 31
- **Total Test Cases**: 12

### 3.2 Appendix B: Benchmark Configuration
- **Field Retrieval**: 55 questions (37.50% weight)
- **Filtering**: 21 questions (20.83% weight)
- **Aggregation**: 21 questions (12.50% weight)
- **Structure Awareness**: 27 questions (29.17% weight)

---

- **Report Generated**: 2026-04-12
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code < 2.1.86
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_flat_all_formats_and_variants_off\BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_flat_all_formats_and_variants_on\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_on\BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)