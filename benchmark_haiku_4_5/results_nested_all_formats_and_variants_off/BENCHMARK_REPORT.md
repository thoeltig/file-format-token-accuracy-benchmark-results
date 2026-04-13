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
- **Information Value**: (**Accuracy** % / **Tokens**) * 100

#### 1.3.3 Efficiency Score
Composite metric balancing accuracy with normalized token count (favour towards accuracy). Each efficieny score has an indicator which token count was used in the calculation.
- **Accuracy To Token Ratio** = 66.67 % to 33.33 %
- **Normalized Tokens** = (((**Max Tokens** + 10) - **Current Tokens**) / ((**Max Tokens** + 10) - (**Min Tokens** - 10))) * 100
- **Normalized Tokens** = (((**Max Tokens** + 10) - **Current Tokens**) / ((**Max Tokens** + 10) - (**Min Tokens** - 10))) * 100
- **Efficiency Score**: (**Accuracy** % * 0.66666) + (**Normalized Tokens** * 0.33333)
- **Weighted Efficiency Score**: (**Weighted Accuracy** % * 0.66666) + (**Normalized Tokens** * 0.33333)

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

#### 2.1.1 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 77s | JSON_COMPACT ≈ 10163 | YAML ≈ 213 | XML_COMPACT ≈ 4990 | XML_COMPACT ≈ 5295 | XML_COMPACT ≈ 18000 | XML_COMPACT ≈ 74.19% | JSON_COMPACT ≈ 81 | XML_COMPACT ≈ 83 | XML_COMPACT ≈ 83 | JSON_COMPACT ≈ 93.04% | JSON_COMPACT ≈ 94 | XML_COMPACT ≈ 95 | XML_COMPACT ≈ 95 |
| XML_COMPACT (+3.89%) | XML_COMPACT (+25.01%) | XML_PRETTY (+42.12%) | TOON_DEFAULT (+72.46%) | TOON_DEFAULT (+68.36%) | JSON_COMPACT (+11.63%) | YAML (-0.54%) | XML_COMPACT (-9.28%) | TOON_DEFAULT (-17.84%) | JSON_COMPACT (-7.57%) | XML_COMPACT (-0.25%) | XML_COMPACT (-8.73%) | TOON_DEFAULT (-14.89%) | JSON_COMPACT (-5.85%) |
| XML_PRETTY (+6.07%) | TOON_DEFAULT (+38.42%) | XML_COMPACT (+43.42%) | JSON_PRETTY (+79.09%) | JSON_PRETTY (+74.56%) | TOON_DEFAULT (+27.68%) | JSON_COMPACT (-0.80%) | YAML (-15.39%) | JSON_PRETTY (-19.95%) | TOON_DEFAULT (-17.71%) | TOON_DEFAULT (-0.87%) | TOON_DEFAULT (-13.75%) | JSON_PRETTY (-16.49%) | TOON_DEFAULT (-14.78%) |
| JSON_COMPACT (+6.56%) | YAML (+39.28%) | JSON_COMPACT (+43.51%) | JSON_COMPACT (+92.88%) | JSON_COMPACT (+87.53%) | YAML (+34.37%) | TOON_DEFAULT (-1.51%) | TOON_DEFAULT (-15.85%) | JSON_COMPACT (-21.93%) | YAML (-20.91%) | YAML (-0.97%) | YAML (-14.11%) | JSON_COMPACT (-18.34%) | YAML (-18.31%) |
| TOON_DEFAULT (+7.89%) | JSON_PRETTY (+73.98%) | JSON_PRETTY (+44.17%) | XML_PRETTY (+93.24%) | XML_PRETTY (+87.81%) | JSON_PRETTY (+49.58%) | JSON_PRETTY (-2.26%) | JSON_PRETTY (-30.60%) | YAML (-22.19%) | JSON_PRETTY (-31.36%) | JSON_PRETTY (-1.28%) | JSON_PRETTY (-26.20%) | YAML (-19.42%) | JSON_PRETTY (-26.41%) |
| YAML (+11.79%) | XML_PRETTY (+98.80%) | TOON_DEFAULT (+45.31%) | YAML (+96.77%) | YAML (+89.45%) | XML_PRETTY (+67.49%) | XML_PRETTY (-6.18%) | XML_PRETTY (-43.71%) | XML_PRETTY (-26.33%) | XML_PRETTY (-45.19%) | XML_PRETTY (-1.92%) | XML_PRETTY (-35.13%) | XML_PRETTY (-19.74%) | XML_PRETTY (-36.14%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 67s | JSON_COMPACT ≈ 9645 | XML_PRETTY ≈ 205 | YAML ≈ 7786 | YAML ≈ 8089 | JSON_COMPACT ≈ 21434 | JSON_COMPACT ≈ 74.84% | JSON_COMPACT ≈ 83 | XML_PRETTY ≈ 67 | JSON_COMPACT ≈ 74 | JSON_COMPACT ≈ 91.36% | JSON_COMPACT ≈ 94 | YAML ≈ 82 | JSON_COMPACT ≈ 85 |
| XML_PRETTY (+2.24%) | XML_COMPACT (+29.13%) | TOON_DEFAULT (+25.69%) | XML_PRETTY (+3.99%) | XML_PRETTY (+2.63%) | XML_COMPACT (+1.44%) | TOON_DEFAULT (-1.34%) | XML_COMPACT (-12.45%) | XML_COMPACT (-1.31%) | XML_COMPACT (-3.19%) | TOON_DEFAULT (-0.29%) | XML_COMPACT (-9.82%) | XML_PRETTY (-1.31%) | XML_COMPACT (-1.46%) |
| XML_COMPACT (+13.55%) | TOON_DEFAULT (+43.41%) | YAML (+47.64%) | XML_COMPACT (+15.34%) | XML_COMPACT (+14.82%) | YAML (+3.74%) | XML_COMPACT (-2.26%) | TOON_DEFAULT (-16.93%) | YAML (-2.79%) | YAML (-12.55%) | XML_COMPACT (-0.59%) | TOON_DEFAULT (-14.21%) | XML_COMPACT (-4.08%) | YAML (-4.49%) |
| JSON_PRETTY (+39.93%) | YAML (+46.69%) | JSON_COMPACT (+48.68%) | JSON_PRETTY (+46.79%) | JSON_PRETTY (+45.07%) | TOON_DEFAULT (+20.11%) | JSON_PRETTY (-2.53%) | YAML (-25.54%) | JSON_COMPACT (-13.18%) | TOON_DEFAULT (-17.21%) | JSON_PRETTY (-1.10%) | YAML (-16.77%) | JSON_COMPACT (-15.20%) | TOON_DEFAULT (-14.15%) |
| JSON_COMPACT (+40.79%) | JSON_PRETTY (+73.74%) | JSON_PRETTY (+48.78%) | JSON_COMPACT (+47.50%) | JSON_COMPACT (+45.75%) | XML_PRETTY (+30.50%) | XML_PRETTY (-6.56%) | JSON_PRETTY (-28.96%) | TOON_DEFAULT (-15.20%) | JSON_PRETTY (-28.48%) | YAML (-2.41%) | JSON_PRETTY (-24.57%) | JSON_PRETTY (-15.84%) | XML_PRETTY (-23.33%) |
| TOON_DEFAULT (+40.96%) | XML_PRETTY (+103.95%) | XML_COMPACT (+49.76%) | TOON_DEFAULT (+49.68%) | TOON_DEFAULT (+47.27%) | JSON_PRETTY (+32.92%) | YAML (-10.59%) | XML_PRETTY (-43.23%) | JSON_PRETTY (-15.37%) | XML_PRETTY (-30.20%) | XML_PRETTY (-2.81%) | XML_PRETTY (-35.52%) | TOON_DEFAULT (-16.00%) | JSON_PRETTY (-23.66%) |


#### 2.1.2 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99.39% | XML_COMPACT ≈ 59.26% | YAML ≈ 61.90% | XML_COMPACT ≈ 68.25% |
| JSON_COMPACT (-3.03%) | JSON_PRETTY (-0.00%) | JSON_PRETTY (-1.91%) | JSON_COMPACT (-8.25%) |
| TOON_DEFAULT (-5.76%) | TOON_DEFAULT (-6.02%) | JSON_COMPACT (-3.81%) | TOON_DEFAULT (-10.52%) |
| JSON_PRETTY (-7.76%) | YAML (-6.17%) | TOON_DEFAULT (-4.17%) | XML_PRETTY (-19.05%) |
| XML_COMPACT (-8.48%) | XML_PRETTY (-7.41%) | XML_COMPACT (-6.35%) | JSON_PRETTY (-19.68%) |
| XML_PRETTY (-10.91%) | JSON_COMPACT (-10.37%) | XML_PRETTY (-7.94%) | YAML (-23.81%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 98.18% | XML_COMPACT ≈ 62.96% | JSON_PRETTY ≈ 68.25% | JSON_COMPACT ≈ 53.33% |
| JSON_PRETTY (-2.43%) | TOON_DEFAULT (-2.11%) | TOON_DEFAULT (-1.59%) | TOON_DEFAULT (-8.43%) |
| XML_COMPACT (-3.64%) | XML_PRETTY (-8.64%) | JSON_COMPACT (-4.45%) | XML_PRETTY (-12.06%) |
| TOON_DEFAULT (-4.94%) | JSON_PRETTY (-9.88%) | XML_PRETTY (-6.35%) | YAML (-13.65%) |
| XML_PRETTY (-10.31%) | JSON_COMPACT (-10.37%) | YAML (-6.35%) | JSON_PRETTY (-13.65%) |
| YAML (-15.76%) | YAML (-14.81%) | XML_COMPACT (-6.35%) | XML_COMPACT (-15.24%) |


#### 2.1.3 Category Accuracy By Character Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99.59% | XML_PRETTY ≈ 89.58% | JSON_COMPACT ≈ 92.03% | XML_COMPACT ≈ 87.91% |
| JSON_COMPACT (-0.75%) | JSON_PRETTY (-0.11%) | TOON_DEFAULT (-0.07%) | JSON_COMPACT (-2.51%) |
| TOON_DEFAULT (-1.59%) | XML_COMPACT (-0.74%) | YAML (-0.34%) | TOON_DEFAULT (-5.60%) |
| XML_COMPACT (-2.17%) | YAML (-1.13%) | JSON_PRETTY (-1.02%) | JSON_PRETTY (-6.78%) |
| JSON_PRETTY (-2.35%) | TOON_DEFAULT (-1.47%) | XML_COMPACT (-1.40%) | XML_PRETTY (-8.20%) |
| XML_PRETTY (-2.72%) | JSON_COMPACT (-1.63%) | XML_PRETTY (-2.56%) | YAML (-10.52%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 99.44% | TOON_DEFAULT ≈ 89.21% | JSON_PRETTY ≈ 92.75% | JSON_COMPACT ≈ 78.35% |
| JSON_COMPACT (-0.18%) | XML_COMPACT (-0.15%) | TOON_DEFAULT (-0.10%) | TOON_DEFAULT (-5.02%) |
| XML_COMPACT (-0.98%) | XML_PRETTY (-0.80%) | XML_COMPACT (-0.53%) | YAML (-6.28%) |
| TOON_DEFAULT (-1.30%) | YAML (-1.85%) | YAML (-1.64%) | JSON_PRETTY (-6.91%) |
| XML_PRETTY (-4.08%) | JSON_COMPACT (-3.62%) | JSON_COMPACT (-1.67%) | XML_COMPACT (-6.99%) |
| YAML (-4.10%) | JSON_PRETTY (-4.94%) | XML_PRETTY (-4.55%) | XML_PRETTY (-7.09%) |


#### 2.1.4 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total | Accuracy By Character (%) | Useful Read Tokens (Acc By Char) | Wasted Read Tokens (Acc By Char) | Useful Output Tokens (Acc By Char) | Wasted Output Tokens (Acc By Char) | Eff Score Read (Acc By Char) | Eff Score Output (Acc By Char) | Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10163 | 9930 | 20093 | 2.246 | 77.618 | 73.39 | 7458.626 | 2704.374 | 7287.480 | 2642.320 | 80.60 | 64.61 | 76.50 | 93.04 | 9455.655 | 707.345 | 9238.686 | 691.114 | 93.69 | 77.71 | 89.60 |
| JSON_COMPACT | opt | 9645 | 11789 | 21434 | 2.219 | 92.618 | 74.84 | 7218.318 | 2426.682 | 8823.187 | 2966.213 | 83.19 | 58.51 | 73.79 | 91.36 | 8811.672 | 833.328 | 10770.796 | 1018.604 | 94.21 | 69.52 | 84.80 |
| JSON_PRETTY | man | 17682 | 9243 | 26925 | 1.777 | 72.069 | 71.93 | 12718.663 | 4963.337 | 6648.634 | 2594.566 | 55.93 | 66.24 | 56.81 | 91.76 | 16225.003 | 1456.997 | 8481.560 | 761.640 | 69.15 | 79.46 | 70.03 |
| JSON_PRETTY | opt | 16757 | 11734 | 28491 | 1.767 | 92.172 | 72.31 | 12116.987 | 4640.013 | 8485.096 | 3249.237 | 59.10 | 57.03 | 52.77 | 90.26 | 15124.868 | 1632.132 | 10591.409 | 1142.924 | 71.06 | 68.99 | 64.74 |
| TOON_DEFAULT | man | 14068 | 8915 | 22983 | 1.855 | 69.401 | 72.68 | 10224.622 | 3843.378 | 6479.229 | 2435.505 | 67.82 | 67.99 | 68.11 | 92.17 | 12966.476 | 1101.524 | 8216.710 | 698.024 | 80.81 | 80.98 | 81.10 |
| TOON_DEFAULT | opt | 13832 | 11912 | 25744 | 1.864 | 93.986 | 73.50 | 10166.520 | 3665.480 | 8755.259 | 3156.658 | 69.11 | 57.15 | 61.09 | 91.07 | 12596.802 | 1235.198 | 10848.182 | 1063.734 | 80.82 | 68.86 | 72.81 |
| XML_COMPACT | man | 12705 | 5295 | 18000 | 2.551 | 40.242 | 74.19 | 9425.840 | 3279.161 | 3928.361 | 1366.640 | 73.12 | 82.75 | 82.77 | 92.79 | 11788.970 | 916.030 | 4913.231 | 381.769 | 85.52 | 95.15 | 95.17 |
| XML_COMPACT | opt | 12455 | 9288 | 21743 | 2.499 | 72.423 | 72.58 | 9039.839 | 3415.161 | 6740.868 | 2546.633 | 72.83 | 66.51 | 71.44 | 90.77 | 11305.403 | 1149.597 | 8430.264 | 857.236 | 84.96 | 78.63 | 83.57 |
| XML_PRETTY | man | 20204 | 9945 | 30149 | 1.985 | 77.762 | 68.01 | 13740.740 | 6463.260 | 6763.424 | 3181.326 | 45.37 | 60.96 | 45.37 | 91.12 | 18409.885 | 1794.115 | 9061.656 | 883.094 | 60.78 | 76.37 | 60.77 |
| XML_PRETTY | opt | 19671 | 8301 | 27972 | 1.974 | 65.293 | 68.28 | 13431.359 | 6239.641 | 5668.150 | 2633.183 | 47.23 | 67.39 | 51.51 | 88.55 | 17418.671 | 2252.330 | 7350.830 | 950.503 | 60.74 | 80.90 | 65.02 |
| YAML | man | 14155 | 10031 | 24186 | 1.808 | 79.183 | 73.65 | 10425.158 | 3729.842 | 7388.077 | 2643.256 | 68.19 | 64.39 | 65.46 | 92.07 | 13032.509 | 1122.492 | 9235.848 | 795.485 | 80.47 | 76.67 | 77.74 |
| YAML | opt | 14148 | 8089 | 22237 | 1.787 | 62.790 | 64.25 | 9090.090 | 5057.910 | 5196.969 | 2891.698 | 61.95 | 65.51 | 64.53 | 88.95 | 12584.646 | 1563.354 | 7194.869 | 893.798 | 78.41 | 81.98 | 81.00 |

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
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Before Write (ms) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) | Output (ms) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 13 | 781.769 | 0.42 | 45912 | 35860 | 0.268 | 289.20 | 35873 | 782.037 | 231.44 | 81772 |
| JSON_COMPACT | opt | 9 | 1071.667 | 0.29 | 56146 | 38516 | 0.298 | 310.61 | 38525 | 1071.965 | 248.55 | 94662 |
| JSON_PRETTY | man | 262 | 67.489 | 8.45 | 39379 | 37358 | 0.239 | 301.27 | 37620 | 67.728 | 242.71 | 76737 |
| JSON_PRETTY | opt | 270 | 62.063 | 8.71 | 57037 | 37045 | 0.309 | 298.75 | 37315 | 62.372 | 240.74 | 94082 |
| TOON_DEFAULT | man | 31 | 453.806 | 1.00 | 44249 | 38540 | 0.229 | 310.81 | 38571 | 454.035 | 248.85 | 82789 |
| TOON_DEFAULT | opt | 29 | 476.966 | 0.94 | 56881 | 37891 | 0.321 | 305.58 | 37920 | 477.287 | 244.65 | 94772 |
| XML_COMPACT | man | 13 | 977.308 | 0.42 | 38637 | 41086 | 0.121 | 331.34 | 41099 | 977.429 | 265.16 | 79723 |
| XML_COMPACT | opt | 12 | 1037.917 | 0.39 | 35363 | 40983 | 0.219 | 330.50 | 40995 | 1038.136 | 264.48 | 76345 |
| XML_PRETTY | man | 13 | 1554.154 | 0.42 | 43389 | 38004 | 0.254 | 306.49 | 38017 | 1554.408 | 245.27 | 81393 |
| XML_PRETTY | opt | 6 | 3278.500 | 0.19 | 28165 | 40578 | 0.200 | 327.24 | 40584 | 3278.700 | 261.83 | 68743 |
| YAML | man | 12 | 1179.583 | 0.39 | 48091 | 37690 | 0.261 | 303.95 | 37702 | 1179.844 | 243.24 | 85781 |
| YAML | opt | 11 | 1286.182 | 0.35 | 31187 | 36048 | 0.216 | 290.71 | 36059 | 1286.398 | 232.64 | 67236 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Before Write Man (s) | Output Before Write Opt (s) | Diff (s) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 13 | 9 | -4 | -30.77 | 45.91 | 56.15 |  +10.23 |  +22.29 | 35.86 | 38.52 |  +2.66 |  +7.41 | 35.87 | 38.52 |  +2.65 |  +7.39 | 81.77 | 94.66 |  +12.89 |  +15.76 |
| JSON_PRETTY | 262 | 270 |  +8 |  +3.05 | 39.38 | 57.04 |  +17.66 |  +44.84 | 37.36 | 37.05 | -0.31 | -0.84 | 37.62 | 37.32 | -0.30 | -0.81 | 76.74 | 94.08 |  +17.35 |  +22.60 |
| TOON_DEFAULT | 31 | 29 | -2 | -6.45 | 44.25 | 56.88 |  +12.63 |  +28.55 | 38.54 | 37.89 | -0.65 | -1.68 | 38.57 | 37.92 | -0.65 | -1.69 | 82.79 | 94.77 |  +11.98 |  +14.47 |
| XML_COMPACT | 13 | 12 | -1 | -7.69 | 38.64 | 35.36 | -3.27 | -8.47 | 41.09 | 40.98 | -0.10 | -0.25 | 41.10 | 40.99 | -0.10 | -0.26 | 79.72 | 76.34 | -3.38 | -4.24 |
| XML_PRETTY | 13 | 6 | -7 | -53.85 | 43.39 | 28.16 | -15.22 | -35.09 | 38.00 | 40.58 |  +2.57 |  +6.77 | 38.02 | 40.58 |  +2.57 |  +6.75 | 81.39 | 68.74 | -12.65 | -15.54 |
| YAML | 12 | 11 | -1 | -8.33 | 48.09 | 31.19 | -16.90 | -35.15 | 37.69 | 36.05 | -1.64 | -4.36 | 37.70 | 36.06 | -1.64 | -4.36 | 85.78 | 67.24 | -18.55 | -21.62 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token | Info / Read Token (Acc By Char) | Info / Output Token (Acc By Char) | Info / Total Token (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 2.246 | 14.902 | 327.839 | 0.722 | 0.739 | 0.365 | 0.915 | 0.937 | 0.463 |
| JSON_COMPACT | opt | 2.219 | 15.285 | 311.129 | 0.776 | 0.635 | 0.349 | 0.947 | 0.775 | 0.426 |
| JSON_PRETTY | man | 1.777 | 25.927 | 570.387 | 0.407 | 0.778 | 0.267 | 0.519 | 0.993 | 0.341 |
| JSON_PRETTY | opt | 1.767 | 26.556 | 540.548 | 0.432 | 0.616 | 0.254 | 0.539 | 0.769 | 0.317 |
| TOON_DEFAULT | man | 1.855 | 20.628 | 453.806 | 0.517 | 0.839 | 0.318 | 0.655 | 1.064 | 0.403 |
| TOON_DEFAULT | opt | 1.864 | 21.921 | 446.194 | 0.531 | 0.637 | 0.287 | 0.658 | 0.790 | 0.357 |
| XML_COMPACT | man | 2.551 | 18.629 | 409.839 | 0.584 | 1.401 | 0.412 | 0.730 | 1.752 | 0.516 |
| XML_COMPACT | opt | 2.499 | 19.739 | 401.774 | 0.583 | 0.781 | 0.334 | 0.729 | 0.977 | 0.417 |
| XML_PRETTY | man | 1.985 | 29.625 | 651.742 | 0.337 | 0.684 | 0.226 | 0.451 | 0.916 | 0.302 |
| XML_PRETTY | opt | 1.974 | 31.174 | 634.548 | 0.347 | 0.823 | 0.244 | 0.450 | 1.067 | 0.317 |
| YAML | man | 1.808 | 20.755 | 456.613 | 0.520 | 0.734 | 0.305 | 0.650 | 0.918 | 0.381 |
| YAML | opt | 1.787 | 22.422 | 456.387 | 0.454 | 0.794 | 0.289 | 0.629 | 1.100 | 0.400 |

#### 2.5.2 Characters And Values: Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 2.246 | 2.219 | -0.027 | -1.20 | 14.902 | 15.285 |  +0.383 |  +2.57 | 327.839 | 311.129 | -16.710 | -5.10 |
| JSON_PRETTY | 1.777 | 1.767 | -0.010 | -0.56 | 25.927 | 26.556 |  +0.629 |  +2.43 | 570.387 | 540.548 | -29.839 | -5.23 |
| TOON_DEFAULT | 1.855 | 1.864 |  +0.009 |  +0.49 | 20.628 | 21.921 |  +1.293 |  +6.27 | 453.806 | 446.194 | -7.612 | -1.68 |
| XML_COMPACT | 2.551 | 2.499 | -0.052 | -2.04 | 18.629 | 19.739 |  +1.110 |  +5.96 | 409.839 | 401.774 | -8.065 | -1.97 |
| XML_PRETTY | 1.985 | 1.974 | -0.011 | -0.55 | 29.625 | 31.174 |  +1.549 |  +5.23 | 651.742 | 634.548 | -17.194 | -2.64 |
| YAML | 1.808 | 1.787 | -0.021 | -1.16 | 20.755 | 22.422 |  +1.667 |  +8.03 | 456.613 | 456.387 | -0.226 | -0.05 |

#### 2.5.3 Information: Mandatory vs Optional
| Format | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 0.722 | 0.776 |  +0.054 |  +7.48 | 0.739 | 0.635 | -0.104 | -14.07 | 0.365 | 0.349 | -0.016 | -4.38 |
| JSON_PRETTY | 0.407 | 0.432 |  +0.025 |  +6.14 | 0.778 | 0.616 | -0.162 | -20.82 | 0.267 | 0.254 | -0.013 | -4.87 |
| TOON_DEFAULT | 0.517 | 0.531 |  +0.014 |  +2.71 | 0.839 | 0.637 | -0.201 | -23.97 | 0.318 | 0.287 | -0.030 | -9.45 |
| XML_COMPACT | 0.584 | 0.583 | -0.001 | -0.17 | 1.401 | 0.781 | -0.620 | -44.25 | 0.412 | 0.334 | -0.078 | -18.93 |
| XML_PRETTY | 0.337 | 0.347 |  +0.010 |  +2.97 | 0.684 | 0.823 |  +0.139 |  +20.32 | 0.226 | 0.244 |  +0.018 |  +7.96 |
| YAML | 0.520 | 0.454 | -0.066 | -12.69 | 0.734 | 0.794 |  +0.060 |  +8.17 | 0.305 | 0.289 | -0.016 | -5.25 |

#### 2.5.4 Information (Accuracy By Character): Mandatory vs Optional
| Format | Info / Read Token (Acc By Char) Man | Info / Read Token (Acc By Char) Opt | Diff | Diff (%) | Info / Output Token (Acc By Char) Man | Info / Output Token (Acc By Char)  Opt | Diff | Diff (%) | Info / Total Token (Acc By Char) Man | Info / Total Token (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 0.915 | 0.947 |  +0.032 |  +3.50 | 0.937 | 0.775 | -0.162 | -17.29 | 0.463 | 0.426 | -0.037 | -7.99 |
| JSON_PRETTY | 0.519 | 0.539 |  +0.020 |  +3.85 | 0.993 | 0.769 | -0.224 | -22.56 | 0.341 | 0.317 | -0.024 | -7.04 |
| TOON_DEFAULT | 0.655 | 0.658 |  +0.003 |  +0.46 | 1.064 | 0.790 | -0.274 | -25.80 | 0.403 | 0.357 | -0.046 | -11.54 |
| XML_COMPACT | 0.730 | 0.729 | -0.001 | -0.14 | 1.752 | 0.977 | -0.775 | -44.24 | 0.516 | 0.417 | -0.099 | -19.19 |
| XML_PRETTY | 0.451 | 0.450 | -0.001 | -0.22 | 0.916 | 1.067 |  +0.151 |  +16.48 | 0.302 | 0.317 |  +0.015 |  +4.97 |
| YAML | 0.650 | 0.629 | -0.021 | -3.23 | 0.918 | 1.100 |  +0.182 |  +19.83 | 0.381 | 0.400 |  +0.019 |  +4.99 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10163 | 7459 | 2704 | 9930 | 7287 | 2642 | 20093 | 14746 | 5347 | 73.39 | 80.60 | 64.61 | 76.50 | 70.00 | 78.34 | 62.35 | 74.24 |
| JSON_COMPACT | opt | 9645 | 7218 | 2427 | 11789 | 8823 | 2966 | 21434 | 16042 | 5393 | 74.84 | 83.19 | 58.51 | 73.79 | 72.11 | 81.37 | 56.69 | 71.97 |
| JSON_PRETTY | man | 17682 | 12719 | 4963 | 9243 | 6649 | 2595 | 26925 | 19367 | 7558 | 71.93 | 55.93 | 66.24 | 56.81 | 70.21 | 54.78 | 65.09 | 55.66 |
| JSON_PRETTY | opt | 16757 | 12117 | 4640 | 11734 | 8485 | 3249 | 28491 | 20602 | 7889 | 72.31 | 59.10 | 57.03 | 52.77 | 70.57 | 57.94 | 55.87 | 51.61 |
| TOON_DEFAULT | man | 14068 | 10225 | 3843 | 8915 | 6479 | 2436 | 22983 | 16704 | 6279 | 72.68 | 67.82 | 67.99 | 68.11 | 69.88 | 65.95 | 66.12 | 66.24 |
| TOON_DEFAULT | opt | 13832 | 10167 | 3665 | 11912 | 8755 | 3157 | 25744 | 18922 | 6822 | 73.50 | 69.11 | 57.15 | 61.09 | 72.21 | 68.25 | 56.29 | 60.23 |
| XML_COMPACT | man | 12705 | 9426 | 3279 | 5295 | 3928 | 1367 | 18000 | 13354 | 4646 | 74.19 | 73.12 | 82.75 | 82.77 | 71.47 | 71.31 | 80.94 | 80.95 |
| XML_COMPACT | opt | 12455 | 9040 | 3415 | 9288 | 6741 | 2547 | 21743 | 15781 | 5962 | 72.58 | 72.83 | 66.51 | 71.44 | 71.47 | 72.09 | 65.77 | 70.70 |
| XML_PRETTY | man | 20204 | 13741 | 6463 | 9945 | 6763 | 3181 | 30149 | 20504 | 9645 | 68.01 | 45.37 | 60.96 | 45.37 | 65.70 | 43.83 | 59.42 | 43.83 |
| XML_PRETTY | opt | 19671 | 13431 | 6240 | 8301 | 5668 | 2633 | 27972 | 19100 | 8873 | 68.28 | 47.23 | 67.39 | 51.51 | 66.85 | 46.28 | 66.44 | 50.55 |
| YAML | man | 14155 | 10425 | 3730 | 10031 | 7388 | 2643 | 24186 | 17813 | 6373 | 73.65 | 68.19 | 64.39 | 65.46 | 71.20 | 66.56 | 62.76 | 63.83 |
| YAML | opt | 14148 | 9090 | 5058 | 8089 | 5197 | 2892 | 22237 | 14287 | 7950 | 64.25 | 61.95 | 65.51 | 64.53 | 62.81 | 60.99 | 64.55 | 63.57 |

#### 2.6.2 Read Tokens: Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10163 | 9645 | -518 | -5.10 | 7459 | 7219 | -240 | -3.22 | 2704 | 2426 | -278 | -10.27 | 73.39 | 74.84 |  +1.45 |  +1.98 | 80.60 | 83.19 |  +2.60 |  +3.22 | 70.00 | 72.11 |  +2.11 |  +3.01 | 78.34 | 81.37 |  +3.04 |  +3.88 |
| JSON_PRETTY | 17682 | 16757 | -925 | -5.23 | 12719 | 12117 | -602 | -4.73 | 4963 | 4640 | -323 | -6.51 | 71.93 | 72.31 |  +0.38 |  +0.53 | 55.93 | 59.10 |  +3.17 |  +5.66 | 70.21 | 70.57 |  +0.36 |  +0.51 | 54.78 | 57.94 |  +3.16 |  +5.76 |
| TOON_DEFAULT | 14068 | 13832 | -236 | -1.68 | 10225 | 10167 | -58 | -0.57 | 3843 | 3665 | -178 | -4.63 | 72.68 | 73.50 |  +0.82 |  +1.13 | 67.82 | 69.11 |  +1.29 |  +1.90 | 69.88 | 72.21 |  +2.33 |  +3.33 | 65.95 | 68.25 |  +2.30 |  +3.48 |
| XML_COMPACT | 12705 | 12455 | -250 | -1.97 | 9426 | 9040 | -386 | -4.10 | 3279 | 3415 |  +136 |  +4.15 | 74.19 | 72.58 | -1.61 | -2.17 | 73.12 | 72.83 | -0.28 | -0.39 | 71.47 | 71.47 | 0.00 | 0.00 | 71.31 | 72.09 |  +0.79 |  +1.11 |
| XML_PRETTY | 20204 | 19671 | -533 | -2.64 | 13741 | 13432 | -309 | -2.25 | 6463 | 6239 | -224 | -3.46 | 68.01 | 68.28 |  +0.27 |  +0.40 | 45.37 | 47.23 |  +1.86 |  +4.10 | 65.70 | 66.85 |  +1.15 |  +1.75 | 43.83 | 46.28 |  +2.45 |  +5.58 |
| YAML | 14155 | 14148 | -7 | -0.05 | 10425 | 9090 | -1335 | -12.81 | 3730 | 5058 |  +1328 |  +35.61 | 73.65 | 64.25 | -9.40 | -12.76 | 68.19 | 61.95 | -6.25 | -9.16 | 71.20 | 62.81 | -8.39 | -11.78 | 66.56 | 60.99 | -5.57 | -8.37 |

#### 2.6.3 Output Tokens: Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 9930 | 11790 |  +1860 |  +18.73 | 7287 | 8823 |  +1536 |  +21.07 | 2642 | 2966 |  +324 |  +12.26 | 73.39 | 74.84 |  +1.45 |  +1.98 | 64.61 | 58.51 | -6.10 | -9.44 | 70.00 | 72.11 |  +2.11 |  +3.01 | 62.35 | 56.69 | -5.66 | -9.08 |
| JSON_PRETTY | 9243 | 11734 |  +2491 |  +26.95 | 6649 | 8485 |  +1836 |  +27.62 | 2595 | 3250 |  +655 |  +25.23 | 71.93 | 72.31 |  +0.38 |  +0.53 | 66.24 | 57.03 | -9.21 | -13.91 | 70.21 | 70.57 |  +0.36 |  +0.51 | 65.09 | 55.87 | -9.23 | -14.17 |
| TOON_DEFAULT | 8915 | 11912 |  +2997 |  +33.62 | 6479 | 8755 |  +2276 |  +35.13 | 2436 | 3157 |  +721 |  +29.60 | 72.68 | 73.50 |  +0.82 |  +1.13 | 67.99 | 57.15 | -10.84 | -15.95 | 69.88 | 72.21 |  +2.33 |  +3.33 | 66.12 | 56.29 | -9.84 | -14.88 |
| XML_COMPACT | 5295 | 9288 |  +3993 |  +75.40 | 3928 | 6741 |  +2813 |  +71.60 | 1367 | 2547 |  +1180 |  +86.32 | 74.19 | 72.58 | -1.61 | -2.17 | 82.75 | 66.51 | -16.25 | -19.63 | 71.47 | 71.47 | 0.00 | 0.00 | 80.94 | 65.77 | -15.17 | -18.75 |
| XML_PRETTY | 9945 | 8302 | -1643 | -16.53 | 6763 | 5668 | -1095 | -16.20 | 3181 | 2633 | -548 | -17.23 | 68.01 | 68.28 |  +0.27 |  +0.40 | 60.96 | 67.39 |  +6.43 |  +10.54 | 65.70 | 66.85 |  +1.15 |  +1.75 | 59.42 | 66.44 |  +7.01 |  +11.80 |
| YAML | 10031 | 8088 | -1943 | -19.37 | 7388 | 5197 | -2191 | -29.66 | 2643 | 2891 |  +248 |  +9.40 | 73.65 | 64.25 | -9.40 | -12.76 | 64.39 | 65.51 |  +1.12 |  +1.73 | 71.20 | 62.81 | -8.39 | -11.78 | 62.76 | 64.55 |  +1.79 |  +2.85 |

#### 2.6.4 Total Tokens: Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 20093 | 21435 |  +1342 |  +6.68 | 14746 | 16041 |  +1295 |  +8.78 | 5347 | 5393 |  +46 |  +0.86 | 73.39 | 74.84 |  +1.45 |  +1.98 | 76.50 | 73.79 | -2.71 | -3.54 | 70.00 | 72.11 |  +2.11 |  +3.01 | 74.24 | 71.97 | -2.27 | -3.05 |
| JSON_PRETTY | 26925 | 28491 |  +1566 |  +5.82 | 19367 | 20602 |  +1235 |  +6.38 | 7558 | 7889 |  +331 |  +4.38 | 71.93 | 72.31 |  +0.38 |  +0.53 | 56.81 | 52.77 | -4.04 | -7.10 | 70.21 | 70.57 |  +0.36 |  +0.51 | 55.66 | 51.61 | -4.05 | -7.28 |
| TOON_DEFAULT | 22983 | 25744 |  +2761 |  +12.01 | 16704 | 18922 |  +2218 |  +13.28 | 6279 | 6822 |  +543 |  +8.65 | 72.68 | 73.50 |  +0.82 |  +1.13 | 68.11 | 61.09 | -7.02 | -10.30 | 69.88 | 72.21 |  +2.33 |  +3.33 | 66.24 | 60.23 | -6.01 | -9.07 |
| XML_COMPACT | 18000 | 21743 |  +3743 |  +20.79 | 13354 | 15781 |  +2427 |  +18.17 | 4646 | 5962 |  +1316 |  +28.33 | 74.19 | 72.58 | -1.61 | -2.17 | 82.77 | 71.44 | -11.33 | -13.68 | 71.47 | 71.47 | 0.00 | 0.00 | 80.95 | 70.70 | -10.25 | -12.66 |
| XML_PRETTY | 30149 | 27973 | -2176 | -7.22 | 20504 | 19099 | -1405 | -6.85 | 9645 | 8873 | -772 | -8.00 | 68.01 | 68.28 |  +0.27 |  +0.40 | 45.37 | 51.51 |  +6.14 |  +13.54 | 65.70 | 66.85 |  +1.15 |  +1.75 | 43.83 | 50.55 |  +6.73 |  +15.35 |
| YAML | 24186 | 22236 | -1950 | -8.06 | 17813 | 14287 | -3526 | -19.80 | 6373 | 7950 |  +1577 |  +24.74 | 73.65 | 64.25 | -9.40 | -12.76 | 65.46 | 64.53 | -0.93 | -1.41 | 71.20 | 62.81 | -8.39 | -11.78 | 63.83 | 63.57 | -0.25 | -0.40 |

### 2.7 Token Utilization Efficiency (Accuracy by Character)
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy by Character (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy by Character (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10163 | 9456 | 707 | 9930 | 9239 | 691 | 20093 | 18694 | 1398 | 93.04 | 93.69 | 77.71 | 89.60 | 92.57 | 93.38 | 77.39 | 89.29 |
| JSON_COMPACT | opt | 9645 | 8812 | 833 | 11789 | 10771 | 1019 | 21434 | 19582 | 1852 | 91.36 | 94.21 | 69.52 | 84.80 | 90.95 | 93.93 | 69.25 | 84.53 |
| JSON_PRETTY | man | 17682 | 16225 | 1457 | 9243 | 8482 | 762 | 26925 | 24707 | 2219 | 91.76 | 69.15 | 79.46 | 70.03 | 91.66 | 69.08 | 79.39 | 69.96 |
| JSON_PRETTY | opt | 16757 | 15125 | 1632 | 11734 | 10591 | 1143 | 28491 | 25716 | 2775 | 90.26 | 71.06 | 68.99 | 64.74 | 90.13 | 70.98 | 68.91 | 64.65 |
| TOON_DEFAULT | man | 14068 | 12966 | 1102 | 8915 | 8217 | 698 | 22983 | 21183 | 1800 | 92.17 | 80.81 | 80.98 | 81.10 | 91.90 | 80.63 | 80.80 | 80.92 |
| TOON_DEFAULT | opt | 13832 | 12597 | 1235 | 11912 | 10848 | 1064 | 25744 | 23445 | 2299 | 91.07 | 80.82 | 68.86 | 72.81 | 91.30 | 80.97 | 69.01 | 72.96 |
| XML_COMPACT | man | 12705 | 11789 | 916 | 5295 | 4913 | 382 | 18000 | 16702 | 1298 | 92.79 | 85.52 | 95.15 | 95.17 | 92.31 | 85.20 | 94.83 | 94.84 |
| XML_COMPACT | opt | 12455 | 11305 | 1150 | 9288 | 8430 | 857 | 21743 | 19736 | 2007 | 90.77 | 84.96 | 78.63 | 83.57 | 91.03 | 85.13 | 78.81 | 83.74 |
| XML_PRETTY | man | 20204 | 18410 | 1794 | 9945 | 9062 | 883 | 30149 | 27472 | 2677 | 91.12 | 60.78 | 76.37 | 60.77 | 91.05 | 60.73 | 76.32 | 60.73 |
| XML_PRETTY | opt | 19671 | 17419 | 2252 | 8301 | 7351 | 951 | 27972 | 24770 | 3203 | 88.55 | 60.74 | 80.90 | 65.02 | 88.83 | 60.93 | 81.09 | 65.21 |
| YAML | man | 14155 | 13033 | 1122 | 10031 | 9236 | 795 | 24186 | 22268 | 1918 | 92.07 | 80.47 | 76.67 | 77.74 | 91.93 | 80.38 | 76.58 | 77.65 |
| YAML | opt | 14148 | 12585 | 1563 | 8089 | 7195 | 894 | 22237 | 19780 | 2457 | 88.95 | 78.41 | 81.98 | 81.00 | 89.23 | 78.60 | 82.16 | 81.19 |

#### 2.7.2 Read Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10163 | 9645 | -518 | -5.10 | 9456 | 8812 | -644 | -6.81 | 707 | 833 |  +126 |  +17.82 | 93.04 | 91.36 | -1.68 | -1.81 | 93.69 | 94.21 |  +0.51 |  +0.55 | 92.57 | 90.95 | -1.62 | -1.75 | 93.38 | 93.93 |  +0.55 |  +0.59 |
| JSON_PRETTY | 17682 | 16757 | -925 | -5.23 | 16225 | 15125 | -1100 | -6.78 | 1457 | 1632 |  +175 |  +12.02 | 91.76 | 90.26 | -1.50 | -1.63 | 69.15 | 71.06 |  +1.91 |  +2.77 | 91.66 | 90.13 | -1.53 | -1.67 | 69.08 | 70.98 |  +1.89 |  +2.74 |
| TOON_DEFAULT | 14068 | 13832 | -236 | -1.68 | 12966 | 12596 | -370 | -2.85 | 1102 | 1236 |  +134 |  +12.13 | 92.17 | 91.07 | -1.10 | -1.19 | 80.81 | 80.82 |  +0.01 |  +0.01 | 91.90 | 91.30 | -0.60 | -0.65 | 80.63 | 80.97 |  +0.34 |  +0.43 |
| XML_COMPACT | 12705 | 12455 | -250 | -1.97 | 11789 | 11305 | -484 | -4.10 | 916 | 1150 |  +234 |  +25.50 | 92.79 | 90.77 | -2.02 | -2.18 | 85.52 | 84.96 | -0.56 | -0.65 | 92.31 | 91.03 | -1.28 | -1.39 | 85.20 | 85.13 | -0.06 | -0.08 |
| XML_PRETTY | 20204 | 19671 | -533 | -2.64 | 18410 | 17419 | -991 | -5.38 | 1794 | 2252 |  +458 |  +25.54 | 91.12 | 88.55 | -2.57 | -2.82 | 60.78 | 60.74 | -0.03 | -0.06 | 91.05 | 88.83 | -2.22 | -2.44 | 60.73 | 60.93 |  +0.20 |  +0.33 |
| YAML | 14155 | 14148 | -7 | -0.05 | 13033 | 12585 | -448 | -3.44 | 1122 | 1563 |  +441 |  +39.29 | 92.07 | 88.95 | -3.12 | -3.39 | 80.47 | 78.41 | -2.06 | -2.56 | 91.93 | 89.23 | -2.70 | -2.94 | 80.38 | 78.60 | -1.78 | -2.21 |

#### 2.7.3 Output Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 9930 | 11790 |  +1860 |  +18.73 | 9239 | 10771 |  +1532 |  +16.58 | 691 | 1018 |  +327 |  +47.39 | 93.04 | 91.36 | -1.68 | -1.81 | 77.71 | 69.52 | -8.19 | -10.54 | 92.57 | 90.95 | -1.62 | -1.75 | 77.39 | 69.25 | -8.15 | -10.53 |
| JSON_PRETTY | 9243 | 11734 |  +2491 |  +26.95 | 8482 | 10592 |  +2110 |  +24.87 | 762 | 1143 |  +381 |  +50.04 | 91.76 | 90.26 | -1.50 | -1.63 | 79.46 | 68.99 | -10.47 | -13.17 | 91.66 | 90.13 | -1.53 | -1.67 | 79.39 | 68.91 | -10.49 | -13.21 |
| TOON_DEFAULT | 8915 | 11912 |  +2997 |  +33.62 | 8217 | 10848 |  +2631 |  +32.02 | 698 | 1064 |  +366 |  +52.39 | 92.17 | 91.07 | -1.10 | -1.19 | 80.98 | 68.86 | -12.12 | -14.97 | 91.90 | 91.30 | -0.60 | -0.65 | 80.80 | 69.01 | -11.79 | -14.59 |
| XML_COMPACT | 5295 | 9288 |  +3993 |  +75.40 | 4913 | 8430 |  +3517 |  +71.59 | 382 | 857 |  +475 |  +124.47 | 92.79 | 90.77 | -2.02 | -2.18 | 95.15 | 78.63 | -16.52 | -17.36 | 92.31 | 91.03 | -1.28 | -1.39 | 94.83 | 78.81 | -16.03 | -16.90 |
| XML_PRETTY | 9945 | 8302 | -1643 | -16.53 | 9062 | 7351 | -1711 | -18.88 | 883 | 950 |  +67 |  +7.63 | 91.12 | 88.55 | -2.57 | -2.82 | 76.37 | 80.90 |  +4.53 |  +5.94 | 91.05 | 88.83 | -2.22 | -2.44 | 76.32 | 81.09 |  +4.77 |  +6.24 |
| YAML | 10031 | 8088 | -1943 | -19.37 | 9236 | 7195 | -2041 | -22.10 | 795 | 893 |  +98 |  +12.37 | 92.07 | 88.95 | -3.12 | -3.39 | 76.67 | 81.98 |  +5.30 |  +6.92 | 91.93 | 89.23 | -2.70 | -2.94 | 76.58 | 82.16 |  +5.58 |  +7.29 |

#### 2.7.4 Total Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 20093 | 21435 |  +1342 |  +6.68 | 18694 | 19582 |  +888 |  +4.75 | 1398 | 1851 |  +453 |  +32.44 | 93.04 | 91.36 | -1.68 | -1.81 | 89.60 | 84.80 | -4.80 | -5.35 | 92.57 | 90.95 | -1.62 | -1.75 | 89.29 | 84.53 | -4.75 | -5.33 |
| JSON_PRETTY | 26925 | 28491 |  +1566 |  +5.82 | 24707 | 25717 |  +1010 |  +4.09 | 2219 | 2775 |  +556 |  +25.08 | 91.76 | 90.26 | -1.50 | -1.63 | 70.03 | 64.74 | -5.29 | -7.55 | 91.66 | 90.13 | -1.53 | -1.67 | 69.96 | 64.65 | -5.31 | -7.59 |
| TOON_DEFAULT | 22983 | 25744 |  +2761 |  +12.01 | 21183 | 23445 |  +2262 |  +10.68 | 1800 | 2299 |  +499 |  +27.74 | 92.17 | 91.07 | -1.10 | -1.19 | 81.10 | 72.81 | -8.30 | -10.23 | 91.90 | 91.30 | -0.60 | -0.65 | 80.92 | 72.96 | -7.96 | -9.84 |
| XML_COMPACT | 18000 | 21743 |  +3743 |  +20.79 | 16702 | 19735 |  +3033 |  +18.16 | 1298 | 2007 |  +709 |  +54.63 | 92.79 | 90.77 | -2.02 | -2.18 | 95.17 | 83.57 | -11.60 | -12.19 | 92.31 | 91.03 | -1.28 | -1.39 | 94.84 | 83.74 | -11.11 | -11.71 |
| XML_PRETTY | 30149 | 27973 | -2176 | -7.22 | 27472 | 24770 | -2702 | -9.84 | 2677 | 3203 |  +526 |  +19.63 | 91.12 | 88.55 | -2.57 | -2.82 | 60.77 | 65.02 |  +4.25 |  +6.99 | 91.05 | 88.83 | -2.22 | -2.44 | 60.73 | 65.21 |  +4.48 |  +7.38 |
| YAML | 24186 | 22236 | -1950 | -8.06 | 22268 | 19779 | -2489 | -11.18 | 1918 | 2457 |  +539 |  +28.11 | 92.07 | 88.95 | -3.12 | -3.39 | 77.74 | 81.00 |  +3.26 |  +4.19 | 91.93 | 89.23 | -2.70 | -2.94 | 77.65 | 81.19 |  +3.54 |  +4.56 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Expected Characters | Output Characters | Correct Characters | Incorrect Characters | Accuracy by Character (%) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 91.00 | 33.00 | 0.00 | 73.39 | 7777.00 | 7882.60 | 7579.60 | 303.00 | 93.04 |
| JSON_COMPACT | opt | 92.80 | 31.20 | 0.00 | 74.84 | 8441.00 | 8663.00 | 8318.60 | 344.40 | 91.36 |
| JSON_PRETTY | man | 89.20 | 34.80 | 0.00 | 71.93 | 7777.00 | 7952.00 | 7516.40 | 435.60 | 91.76 |
| JSON_PRETTY | opt | 89.67 | 34.33 | 0.00 | 72.31 | 8441.00 | 8833.33 | 8071.67 | 761.67 | 90.26 |
| TOON_DEFAULT | man | 90.13 | 33.88 | -0.01 | 72.68 | 7777.00 | 7936.75 | 7554.38 | 382.38 | 92.17 |
| TOON_DEFAULT | opt | 91.14 | 32.86 | 0.00 | 73.50 | 8441.00 | 8591.00 | 8272.57 | 318.43 | 91.07 |
| XML_COMPACT | man | 92.00 | 32.00 | 0.00 | 74.19 | 7777.00 | 7913.33 | 7560.67 | 352.67 | 92.79 |
| XML_COMPACT | opt | 90.00 | 34.00 | 0.00 | 72.58 | 8441.00 | 8567.67 | 8282.00 | 285.67 | 90.77 |
| XML_PRETTY | man | 84.33 | 39.67 | 0.00 | 68.01 | 7777.00 | 7907.00 | 7440.67 | 466.33 | 91.12 |
| XML_PRETTY | opt | 84.67 | 39.33 | 0.00 | 68.28 | 8441.00 | 8742.33 | 8055.33 | 687.00 | 88.55 |
| YAML | man | 91.33 | 32.67 | 0.00 | 73.65 | 7777.00 | 7876.67 | 7590.00 | 286.67 | 92.07 |
| YAML | opt | 79.67 | 44.33 | 0.00 | 64.25 | 8441.00 | 8689.67 | 8071.67 | 618.00 | 88.95 |

#### 2.8.2 Answers: Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 91.00 | 92.80 |  +2 |  +1.98 | 33.00 | 31.20 | -2 | -5.45 | 0.00 | 0.00 |  +0 | 0.00 | 73.39 | 74.84 |  +1.45 |
| JSON_PRETTY | 89.20 | 89.67 |  +0 |  +0.53 | 34.80 | 34.33 | -0 | -1.35 | 0.00 | 0.00 | 0 | 0.00 | 71.93 | 72.31 |  +0.38 |
| TOON_DEFAULT | 90.13 | 91.14 |  +1 |  +1.12 | 33.88 | 32.86 | -1 | -3.01 | -0.01 | 0.00 |  +0 | 0.00 | 72.68 | 73.50 |  +0.82 |
| XML_COMPACT | 92.00 | 90.00 | -2 | -2.17 | 32.00 | 34.00 |  +2 |  +6.25 | 0.00 | 0.00 | 0 | 0.00 | 74.19 | 72.58 | -1.61 |
| XML_PRETTY | 84.33 | 84.67 |  +0 |  +0.40 | 39.67 | 39.33 | -0 | -0.86 | 0.00 | 0.00 | 0 | 0.00 | 68.01 | 68.28 |  +0.27 |
| YAML | 91.33 | 79.67 | -12 | -12.77 | 32.67 | 44.33 |  +12 |  +35.69 | 0.00 | 0.00 | 0 | 0.00 | 73.65 | 64.25 | -9.40 |

#### 2.8.3 Characters: Mandatory vs Optional Data
| Format | Output Characters Man | Output Characters Opt | Diff | Diff (%) | Correct Characters Man | Correct Characters Opt | Diff | Diff (%) | Incorrect Characters Man | Incorrect Characters Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7882.60 | 8663.00 |  +780 |  +9.90 | 7579.60 | 8318.60 |  +739 |  +9.75 | 303.00 | 344.40 |  +41 |  +13.66 | 93.04 | 91.36 | -1.68 |
| JSON_PRETTY | 7952.00 | 8833.33 |  +881 |  +11.08 | 7516.40 | 8071.67 |  +555 |  +7.39 | 435.60 | 761.67 |  +326 |  +74.85 | 91.76 | 90.26 | -1.50 |
| TOON_DEFAULT | 7936.75 | 8591.00 |  +654 |  +8.24 | 7554.38 | 8272.57 |  +718 |  +9.51 | 382.38 | 318.43 | -64 | -16.72 | 92.17 | 91.07 | -1.10 |
| XML_COMPACT | 7913.33 | 8567.67 |  +654 |  +8.27 | 7560.67 | 8282.00 |  +721 |  +9.54 | 352.67 | 285.67 | -67 | -19.00 | 92.79 | 90.77 | -2.02 |
| XML_PRETTY | 7907.00 | 8742.33 |  +835 |  +10.56 | 7440.67 | 8055.33 |  +615 |  +8.26 | 466.33 | 687.00 |  +221 |  +47.32 | 91.12 | 88.55 | -2.57 |
| YAML | 7876.67 | 8689.67 |  +813 |  +10.32 | 7590.00 | 8071.67 |  +482 |  +6.35 | 286.67 | 618.00 |  +331 |  +115.58 | 92.07 | 88.95 | -3.12 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 73.39 | 96.36 | 48.89 | 58.09 | 60.00 | 92.57 | 36.14 | 14.26 | 12.10 | 7.50 |
| JSON_COMPACT | opt | 74.84 | 98.18 | 52.59 | 63.81 | 53.33 | 90.95 | 36.82 | 15.34 | 13.29 | 6.67 |
| JSON_PRETTY | man | 71.93 | 91.64 | 59.26 | 60.00 | 48.57 | 91.66 | 34.36 | 17.28 | 12.50 | 6.07 |
| JSON_PRETTY | opt | 72.31 | 95.76 | 53.08 | 68.25 | 39.68 | 90.13 | 35.91 | 15.48 | 14.22 | 4.96 |
| TOON_DEFAULT | man | 72.68 | 93.64 | 53.24 | 57.74 | 57.74 | 91.90 | 35.11 | 15.53 | 12.03 | 7.22 |
| TOON_DEFAULT | opt | 73.50 | 93.25 | 60.85 | 66.67 | 44.90 | 91.30 | 34.97 | 17.74 | 13.89 | 5.61 |
| XML_COMPACT | man | 74.19 | 90.91 | 59.26 | 55.55 | 68.25 | 92.31 | 34.09 | 17.28 | 11.57 | 8.53 |
| XML_COMPACT | opt | 72.58 | 94.55 | 62.96 | 61.90 | 38.10 | 91.03 | 35.45 | 18.36 | 12.90 | 4.76 |
| XML_PRETTY | man | 68.01 | 88.48 | 51.85 | 53.97 | 49.21 | 91.05 | 33.18 | 15.12 | 11.24 | 6.15 |
| XML_PRETTY | opt | 68.28 | 87.88 | 54.32 | 61.90 | 41.27 | 88.83 | 32.96 | 15.84 | 12.90 | 5.16 |
| YAML | man | 73.65 | 99.39 | 53.09 | 61.90 | 44.45 | 91.93 | 37.27 | 15.48 | 12.90 | 5.55 |
| YAML | opt | 64.25 | 82.42 | 48.15 | 61.90 | 39.68 | 89.23 | 30.91 | 14.04 | 12.90 | 4.96 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 96.36 | 98.18 |  +1.82 | 36.14 | 36.82 |  +0.68 |
| JSON_PRETTY | 91.64 | 95.76 |  +4.12 | 34.36 | 35.91 |  +1.55 |
| TOON_DEFAULT | 93.64 | 93.25 | -0.39 | 35.11 | 34.97 | -0.15 |
| XML_COMPACT | 90.91 | 94.55 |  +3.64 | 34.09 | 35.45 |  +1.36 |
| XML_PRETTY | 88.48 | 87.88 | -0.61 | 33.18 | 32.96 | -0.23 |
| YAML | 99.39 | 82.42 | -16.97 | 37.27 | 30.91 | -6.36 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 48.89 | 52.59 |  +3.70 | 14.26 | 15.34 |  +1.08 |
| JSON_PRETTY | 59.26 | 53.08 | -6.17 | 17.28 | 15.48 | -1.80 |
| TOON_DEFAULT | 53.24 | 60.85 |  +7.61 | 15.53 | 17.74 |  +2.22 |
| XML_COMPACT | 59.26 | 62.96 |  +3.70 | 17.28 | 18.36 |  +1.08 |
| XML_PRETTY | 51.85 | 54.32 |  +2.47 | 15.12 | 15.84 |  +0.72 |
| YAML | 53.09 | 48.15 | -4.94 | 15.48 | 14.04 | -1.44 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 58.09 | 63.81 |  +5.71 | 12.10 | 13.29 |  +1.19 |
| JSON_PRETTY | 60.00 | 68.25 |  +8.26 | 12.50 | 14.22 |  +1.72 |
| TOON_DEFAULT | 57.74 | 66.67 |  +8.93 | 12.03 | 13.89 |  +1.86 |
| XML_COMPACT | 55.55 | 61.90 |  +6.35 | 11.57 | 12.90 |  +1.33 |
| XML_PRETTY | 53.97 | 61.90 |  +7.94 | 11.24 | 12.90 |  +1.65 |
| YAML | 61.90 | 61.90 |  +0.00 | 12.90 | 12.90 | 0.00 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 60.00 | 53.33 | -6.67 | 7.50 | 6.67 | -0.83 |
| JSON_PRETTY | 48.57 | 39.68 | -8.89 | 6.07 | 4.96 | -1.11 |
| TOON_DEFAULT | 57.74 | 44.90 | -12.84 | 7.22 | 5.61 | -1.60 |
| XML_COMPACT | 68.25 | 38.10 | -30.16 | 8.53 | 4.76 | -3.77 |
| XML_PRETTY | 49.21 | 41.27 | -7.93 | 6.15 | 5.16 | -1.00 |
| YAML | 44.45 | 39.68 | -4.76 | 5.55 | 4.96 | -0.59 |

### 2.10 Accuracy By Character Per Question Category Analysis
#### 2.10.1 Metrics
| Format | Variant | Accuracy By Character (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc By Char (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 93.04 | 98.84 | 87.96 | 92.03 | 85.40 | 92.57 | 37.07 | 25.65 | 19.17 | 10.68 |
| JSON_COMPACT | opt | 91.36 | 99.26 | 85.59 | 91.08 | 78.35 | 90.95 | 37.22 | 24.96 | 18.97 | 9.79 |
| JSON_PRETTY | man | 91.76 | 97.24 | 89.48 | 91.02 | 81.13 | 91.66 | 36.46 | 26.10 | 18.96 | 10.14 |
| JSON_PRETTY | opt | 90.26 | 99.44 | 84.27 | 92.75 | 71.45 | 90.13 | 37.29 | 24.58 | 19.32 | 8.93 |
| TOON_DEFAULT | man | 92.17 | 98.00 | 88.11 | 91.96 | 82.31 | 91.90 | 36.75 | 25.70 | 19.16 | 10.29 |
| TOON_DEFAULT | opt | 91.07 | 98.14 | 89.21 | 92.65 | 73.33 | 91.30 | 36.80 | 26.02 | 19.30 | 9.17 |
| XML_COMPACT | man | 92.79 | 97.42 | 88.85 | 90.64 | 87.91 | 92.31 | 36.53 | 25.91 | 18.88 | 10.99 |
| XML_COMPACT | opt | 90.77 | 98.46 | 89.06 | 92.22 | 71.36 | 91.03 | 36.92 | 25.98 | 19.21 | 8.92 |
| XML_PRETTY | man | 91.12 | 96.86 | 89.58 | 89.47 | 79.71 | 91.05 | 36.32 | 26.13 | 18.64 | 9.96 |
| XML_PRETTY | opt | 88.55 | 95.36 | 88.41 | 88.20 | 71.26 | 88.83 | 35.76 | 25.79 | 18.38 | 8.91 |
| YAML | man | 92.07 | 99.59 | 88.46 | 91.69 | 77.39 | 91.93 | 37.35 | 25.80 | 19.10 | 9.68 |
| YAML | opt | 88.95 | 95.34 | 87.36 | 91.11 | 72.07 | 89.23 | 35.75 | 25.48 | 18.98 | 9.01 |

#### 2.10.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 98.84 | 99.26 |  +0.42 | 37.07 | 37.22 |  +0.16 |
| JSON_PRETTY | 97.24 | 99.44 |  +2.20 | 36.46 | 37.29 |  +0.83 |
| TOON_DEFAULT | 98.00 | 98.14 |  +0.14 | 36.75 | 36.80 |  +0.05 |
| XML_COMPACT | 97.42 | 98.46 |  +1.04 | 36.53 | 36.92 |  +0.39 |
| XML_PRETTY | 96.86 | 95.36 | -1.51 | 36.32 | 35.76 | -0.57 |
| YAML | 99.59 | 95.34 | -4.24 | 37.35 | 35.75 | -1.59 |

#### 2.10.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 87.96 | 85.59 | -2.37 | 25.65 | 24.96 | -0.69 |
| JSON_PRETTY | 89.48 | 84.27 | -5.20 | 26.10 | 24.58 | -1.52 |
| TOON_DEFAULT | 88.11 | 89.21 |  +1.10 | 25.70 | 26.02 |  +0.32 |
| XML_COMPACT | 88.85 | 89.06 |  +0.22 | 25.91 | 25.98 |  +0.06 |
| XML_PRETTY | 89.58 | 88.41 | -1.17 | 26.13 | 25.79 | -0.34 |
| YAML | 88.46 | 87.36 | -1.09 | 25.80 | 25.48 | -0.32 |

#### 2.10.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 92.03 | 91.08 | -0.95 | 19.17 | 18.97 | -0.20 |
| JSON_PRETTY | 91.02 | 92.75 |  +1.73 | 18.96 | 19.32 |  +0.36 |
| TOON_DEFAULT | 91.96 | 92.65 |  +0.69 | 19.16 | 19.30 |  +0.14 |
| XML_COMPACT | 90.64 | 92.22 |  +1.58 | 18.88 | 19.21 |  +0.33 |
| XML_PRETTY | 89.47 | 88.20 | -1.27 | 18.64 | 18.38 | -0.26 |
| YAML | 91.69 | 91.11 | -0.58 | 19.10 | 18.98 | -0.12 |

#### 2.10.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 85.40 | 78.35 | -7.05 | 10.68 | 9.79 | -0.88 |
| JSON_PRETTY | 81.13 | 71.45 | -9.68 | 10.14 | 8.93 | -1.21 |
| TOON_DEFAULT | 82.31 | 73.33 | -8.98 | 10.29 | 9.17 | -1.12 |
| XML_COMPACT | 87.91 | 71.36 | -16.55 | 10.99 | 8.92 | -2.06 |
| XML_PRETTY | 79.71 | 71.26 | -8.45 | 9.96 | 8.91 | -1.06 |
| YAML | 77.39 | 72.07 | -5.32 | 9.68 | 9.01 | -0.67 |

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

- **Report Generated**: 2026-04-13
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code < 2.1.86
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)