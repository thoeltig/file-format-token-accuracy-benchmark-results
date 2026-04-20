# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-29
- **Model**: Sonnet 4.6
- **Thinking**: off
- **Data Structure**: nested
- **Formats Tested**: 7 (JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, TOON_KEYFOLD, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: First iteration

## Executive Summary

This benchmark evaluates token efficiency and information accuracy across 7 file formats using Sonnet 4.6 as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context while a format that accurately conveys information may justify higher token cost.

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
- 7 formats tested: JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, TOON_KEYFOLD, XML_COMPACT, XML_PRETTY, YAML
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

These results are specific to Claude Code using the Sonnet 4.6 model. They serve as a rule of thumb for choosing the best file format depending on the use case.
However these values cannot be exactly applied to models of the same family or from other providers as token usage, accuracy and latency depend on specific model architectures and tokenizers. Also file reads will produce different characters depending on the used harness because some add marker characters, line numbers or additional information. While the relative ranking of file formats remains consistent the absolute numbers will vary.
Especially the accuracy and output tokens results will vary because these values are bound to the model size and training, instruction interpretation and reasoning token budget.

## 2. Results

### 2.1 TLDR: Token Efficiency Analysis

*Note: All columns ranked best-to-worst. ↑ = lower value is better (ascending). ↓ = higher value is better (descending).*

#### 2.1.1 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 292s | JSON_COMPACT ≈ 7498 | TOON_DEFAULT ≈ 205 | JSON_PRETTY ≈ 21490 | JSON_PRETTY ≈ 21787 | JSON_COMPACT ≈ 34929 | TOON_DEFAULT ≈ 99.19% | JSON_COMPACT ≈ 98 | JSON_PRETTY ≈ 97 | JSON_COMPACT ≈ 91 | TOON_DEFAULT ≈ 99.99% | JSON_COMPACT ≈ 98 | JSON_PRETTY ≈ 99 | JSON_COMPACT ≈ 92 |
| XML_PRETTY (+5.77%) | XML_COMPACT (+35.56%) | JSON_COMPACT (+43.58%) | YAML (+7.87%) | YAML (+7.76%) | YAML (+0.57%) | XML_COMPACT (0.00%) | XML_COMPACT (-7.67%) | YAML (-7.87%) | YAML (-0.43%) | XML_COMPACT (-0.01%) | XML_COMPACT (-7.76%) | YAML (-9.09%) | YAML (-0.41%) |
| YAML (+8.12%) | TOON_KEYFOLD (+50.56%) | YAML (+43.90%) | XML_PRETTY (+10.20%) | XML_PRETTY (+10.08%) | TOON_DEFAULT (+4.73%) | YAML (-0.27%) | TOON_DEFAULT (-11.51%) | XML_PRETTY (-11.49%) | TOON_DEFAULT (-4.84%) | XML_PRETTY (-0.03%) | TOON_DEFAULT (-11.58%) | XML_PRETTY (-12.10%) | TOON_DEFAULT (-4.95%) |
| TOON_DEFAULT (+12.41%) | TOON_DEFAULT (+52.57%) | JSON_PRETTY (+44.72%) | TOON_DEFAULT (+16.03%) | TOON_DEFAULT (+15.39%) | XML_COMPACT (+5.35%) | JSON_COMPACT (-0.54%) | TOON_KEYFOLD (-11.97%) | TOON_DEFAULT (-17.78%) | XML_COMPACT (-5.53%) | YAML (-0.03%) | TOON_KEYFOLD (-11.75%) | TOON_DEFAULT (-18.96%) | XML_COMPACT (-5.64%) |
| XML_COMPACT (+15.53%) | YAML (+55.36%) | XML_PRETTY (+46.50%) | TOON_KEYFOLD (+20.83%) | TOON_KEYFOLD (+20.59%) | TOON_KEYFOLD (+7.54%) | XML_PRETTY (-1.07%) | YAML (-12.33%) | TOON_KEYFOLD (-25.59%) | TOON_KEYFOLD (-8.93%) | JSON_COMPACT (-0.33%) | YAML (-12.22%) | TOON_KEYFOLD (-26.33%) | TOON_KEYFOLD (-8.72%) |
| TOON_KEYFOLD (+16.29%) | JSON_PRETTY (+110.55%) | XML_COMPACT (+47.15%) | XML_COMPACT (+22.53%) | XML_COMPACT (+22.24%) | JSON_PRETTY (+7.57%) | TOON_KEYFOLD (-1.34%) | JSON_PRETTY (-27.18%) | XML_COMPACT (-26.85%) | JSON_PRETTY (-10.73%) | TOON_KEYFOLD (-0.93%) | JSON_PRETTY (-25.58%) | XML_COMPACT (-27.84%) | JSON_PRETTY (-9.14%) |
| JSON_COMPACT (+24.40%) | XML_PRETTY (+143.95%) | TOON_KEYFOLD (+50.08%) | JSON_COMPACT (+26.27%) | JSON_COMPACT (+25.90%) | XML_PRETTY (+21.03%) | JSON_PRETTY (-3.76%) | XML_PRETTY (-32.89%) | JSON_COMPACT (-32.06%) | XML_PRETTY (-23.66%) | JSON_PRETTY (-1.46%) | XML_PRETTY (-32.10%) | JSON_COMPACT (-32.80%) | XML_PRETTY (-22.89%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 323s | JSON_COMPACT ≈ 6970 | XML_COMPACT ≈ 204 | JSON_PRETTY ≈ 24887 | JSON_PRETTY ≈ 25184 | JSON_COMPACT ≈ 32254 | JSON_PRETTY ≈ 98.66% | JSON_COMPACT ≈ 99 | JSON_PRETTY ≈ 79 | JSON_COMPACT ≈ 99 | JSON_PRETTY ≈ 99.92% | JSON_COMPACT ≈ 100 | JSON_PRETTY ≈ 80 | JSON_COMPACT ≈ 100 |
| JSON_PRETTY (+0.22%) | TOON_KEYFOLD (+58.45%) | TOON_KEYFOLD (+2.95%) | YAML (+0.25%) | YAML (+0.23%) | YAML (+13.62%) | JSON_COMPACT (-0.81%) | TOON_DEFAULT (-13.33%) | JSON_COMPACT (-1.42%) | YAML (-13.63%) | JSON_COMPACT (-0.04%) | TOON_KEYFOLD (-12.19%) | YAML (-0.51%) | YAML (-12.76%) |
| TOON_KEYFOLD (+0.33%) | TOON_DEFAULT (+60.62%) | YAML (+43.86%) | JSON_COMPACT (+0.41%) | JSON_COMPACT (+0.40%) | TOON_KEYFOLD (+13.85%) | XML_COMPACT (-0.81%) | TOON_KEYFOLD (-13.61%) | YAML (-2.03%) | TOON_KEYFOLD (-14.57%) | XML_PRETTY (-0.05%) | TOON_DEFAULT (-12.47%) | JSON_COMPACT (-0.77%) | TOON_KEYFOLD (-13.15%) |
| TOON_DEFAULT (+1.92%) | YAML (+63.62%) | JSON_COMPACT (+45.17%) | XML_COMPACT (+1.86%) | XML_COMPACT (+1.47%) | TOON_DEFAULT (+15.31%) | XML_PRETTY (-1.88%) | YAML (-13.96%) | XML_COMPACT (-3.44%) | TOON_DEFAULT (-15.23%) | TOON_DEFAULT (-0.09%) | YAML (-13.08%) | XML_COMPACT (-2.80%) | TOON_DEFAULT (-14.34%) |
| JSON_COMPACT (+2.27%) | XML_COMPACT (+79.25%) | JSON_PRETTY (+45.83%) | TOON_KEYFOLD (+2.33%) | TOON_KEYFOLD (+1.95%) | XML_COMPACT (+17.97%) | TOON_DEFAULT (-1.89%) | XML_COMPACT (-16.48%) | TOON_KEYFOLD (-6.16%) | XML_COMPACT (-17.02%) | XML_COMPACT (-0.09%) | XML_COMPACT (-16.29%) | TOON_KEYFOLD (-3.92%) | XML_COMPACT (-16.82%) |
| XML_COMPACT (+2.32%) | JSON_PRETTY (+153.77%) | TOON_DEFAULT (+47.46%) | XML_PRETTY (+3.23%) | XML_PRETTY (+3.21%) | JSON_PRETTY (+32.92%) | YAML (-1.89%) | JSON_PRETTY (-31.42%) | XML_PRETTY (-7.61%) | JSON_PRETTY (-30.63%) | YAML (-0.09%) | JSON_PRETTY (-31.51%) | XML_PRETTY (-6.00%) | JSON_PRETTY (-30.73%) |
| XML_PRETTY (+4.62%) | XML_PRETTY (+154.81%) | XML_PRETTY (+47.79%) | TOON_DEFAULT (+3.25%) | TOON_DEFAULT (+3.23%) | XML_PRETTY (+35.65%) | TOON_KEYFOLD (-2.96%) | XML_PRETTY (-32.91%) | TOON_DEFAULT (-7.64%) | XML_PRETTY (-34.49%) | TOON_KEYFOLD (-0.35%) | XML_PRETTY (-31.76%) | TOON_DEFAULT (-6.06%) | XML_PRETTY (-33.32%) |


#### 2.1.2 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 100.00% | TOON_DEFAULT ≈ 100.00% | YAML ≈ 98.41% | JSON_COMPACT ≈ 100.00% |
| TOON_DEFAULT (0.00%) | XML_COMPACT (0.00%) | XML_COMPACT (-1.59%) | JSON_PRETTY (0.00%) |
| TOON_KEYFOLD (0.00%) | XML_PRETTY (0.00%) | TOON_KEYFOLD (-3.17%) | TOON_DEFAULT (0.00%) |
| XML_COMPACT (0.00%) | JSON_COMPACT (-1.23%) | JSON_COMPACT (-3.18%) | XML_COMPACT (-1.59%) |
| XML_PRETTY (0.00%) | YAML (-1.23%) | TOON_DEFAULT (-3.18%) | XML_PRETTY (-1.59%) |
| YAML (0.00%) | TOON_KEYFOLD (-3.70%) | XML_PRETTY (-7.94%) | TOON_KEYFOLD (-3.17%) |
| JSON_COMPACT (-0.61%) | JSON_PRETTY (-7.41%) | JSON_PRETTY (-15.87%) | YAML (-3.17%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 100.00% | JSON_PRETTY ≈ 100.00% | JSON_COMPACT ≈ 100.00% | JSON_PRETTY ≈ 92.07% |
| JSON_PRETTY (0.00%) | XML_PRETTY (0.00%) | JSON_PRETTY (0.00%) | JSON_COMPACT (-1.59%) |
| TOON_DEFAULT (0.00%) | XML_COMPACT (-1.23%) | TOON_DEFAULT (0.00%) | XML_PRETTY (-1.59%) |
| TOON_KEYFOLD (0.00%) | JSON_COMPACT (-2.47%) | XML_COMPACT (0.00%) | TOON_DEFAULT (-1.59%) |
| XML_COMPACT (0.00%) | YAML (-3.70%) | YAML (-3.17%) | TOON_KEYFOLD (-3.18%) |
| XML_PRETTY (0.00%) | TOON_KEYFOLD (-6.17%) | TOON_KEYFOLD (-6.35%) | XML_COMPACT (-3.18%) |
| YAML (0.00%) | TOON_DEFAULT (-7.41%) | XML_PRETTY (-9.53%) | YAML (-3.18%) |


#### 2.1.3 Category Accuracy By Character Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 100.00% | TOON_DEFAULT ≈ 100.00% | YAML ≈ 99.70% | JSON_COMPACT ≈ 100.00% |
| TOON_DEFAULT (0.00%) | XML_COMPACT (0.00%) | XML_COMPACT (-0.30%) | JSON_PRETTY (0.00%) |
| TOON_KEYFOLD (0.00%) | XML_PRETTY (0.00%) | JSON_COMPACT (-0.60%) | TOON_DEFAULT (0.00%) |
| XML_COMPACT (0.00%) | YAML (-0.03%) | TOON_DEFAULT (-0.60%) | XML_COMPACT (-0.62%) |
| XML_PRETTY (0.00%) | JSON_COMPACT (-0.09%) | TOON_KEYFOLD (-0.60%) | XML_PRETTY (-0.62%) |
| YAML (0.00%) | TOON_KEYFOLD (-1.56%) | XML_PRETTY (-1.49%) | YAML (-1.03%) |
| JSON_COMPACT (-0.69%) | JSON_PRETTY (-2.46%) | JSON_PRETTY (-2.98%) | TOON_KEYFOLD (-1.23%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 100.00% | JSON_PRETTY ≈ 100.00% | JSON_COMPACT ≈ 100.00% | JSON_PRETTY ≈ 95.96% |
| JSON_PRETTY (0.00%) | XML_PRETTY (0.00%) | JSON_PRETTY (0.00%) | TOON_DEFAULT (-0.61%) |
| TOON_DEFAULT (0.00%) | JSON_COMPACT (-0.03%) | TOON_DEFAULT (0.00%) | JSON_COMPACT (-1.41%) |
| TOON_KEYFOLD (0.00%) | XML_COMPACT (-0.09%) | XML_COMPACT (0.00%) | XML_PRETTY (-1.41%) |
| XML_COMPACT (0.00%) | YAML (-0.09%) | YAML (-0.59%) | YAML (-1.82%) |
| XML_PRETTY (0.00%) | TOON_DEFAULT (-0.15%) | TOON_KEYFOLD (-1.18%) | TOON_KEYFOLD (-2.02%) |
| YAML (0.00%) | TOON_KEYFOLD (-0.56%) | XML_PRETTY (-1.77%) | XML_COMPACT (-2.02%) |


#### 2.1.4 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total | Accuracy By Character (%) | Useful Read Tokens (Acc By Char) | Wasted Read Tokens (Acc By Char) | Useful Output Tokens (Acc By Char) | Wasted Output Tokens (Acc By Char) | Eff Score Read (Acc By Char) | Eff Score Output (Acc By Char) | Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 7498 | 27431 | 34929 | 3.19 | 218.84 | 98.65 | 7397 | 101 | 27060 | 370 | 97.52 | 65.83 | 91.33 | 99.66 | 7473 | 25 | 27337 | 93 | 98.19 | 66.50 | 92.00 |
| JSON_COMPACT | opt | 6970 | 25284 | 32254 | 3.23 | 201.52 | 97.85 | 6820 | 150 | 24740 | 544 | 98.54 | 77.93 | 98.54 | 99.88 | 6962 | 8 | 25253 | 30 | 99.89 | 79.28 | 99.89 |
| JSON_PRETTY | man | 15787 | 21787 | 37574 | 2.09 | 173.31 | 95.43 | 15066 | 721 | 20791 | 996 | 71.01 | 96.89 | 81.53 | 98.53 | 15555 | 232 | 21467 | 320 | 73.08 | 98.96 | 83.60 |
| JSON_PRETTY | opt | 17688 | 25184 | 42872 | 1.77 | 200.70 | 98.66 | 17451 | 237 | 24847 | 337 | 67.57 | 79.05 | 68.35 | 99.92 | 17674 | 14 | 25164 | 20 | 68.41 | 79.89 | 69.19 |
| TOON_DEFAULT | man | 11440 | 25140 | 36580 | 2.37 | 201.09 | 99.19 | 11347 | 93 | 24936 | 204 | 86.29 | 79.67 | 86.91 | 99.99 | 11439 | 1 | 25137 | 3 | 86.83 | 80.20 | 87.45 |
| TOON_DEFAULT | opt | 11195 | 25997 | 37192 | 2.39 | 207.23 | 96.77 | 10833 | 362 | 25157 | 840 | 85.40 | 73.01 | 83.53 | 99.83 | 11176 | 19 | 25952 | 44 | 87.44 | 75.05 | 85.57 |
| TOON_KEYFOLD | man | 11289 | 26274 | 37563 | 2.38 | 209.41 | 97.85 | 11046 | 243 | 25709 | 565 | 85.84 | 72.10 | 83.17 | 99.06 | 11183 | 106 | 26027 | 247 | 86.65 | 72.91 | 83.98 |
| TOON_KEYFOLD | opt | 11044 | 25676 | 36720 | 2.41 | 205.37 | 95.70 | 10569 | 475 | 24572 | 1104 | 85.13 | 74.19 | 84.18 | 99.57 | 10997 | 47 | 25566 | 110 | 87.71 | 76.77 | 86.76 |
| XML_COMPACT | man | 10164 | 26633 | 36797 | 3.37 | 212.35 | 99.19 | 10082 | 82 | 26417 | 216 | 90.04 | 70.88 | 86.28 | 99.98 | 10162 | 2 | 26628 | 5 | 90.57 | 71.41 | 86.81 |
| XML_COMPACT | opt | 12494 | 25554 | 38048 | 2.64 | 204.44 | 97.85 | 12225 | 269 | 25005 | 549 | 82.30 | 76.33 | 81.77 | 99.83 | 12473 | 21 | 25511 | 43 | 83.62 | 77.66 | 83.09 |
| XML_PRETTY | man | 18291 | 23983 | 42274 | 2.32 | 190.99 | 98.12 | 17947 | 344 | 23532 | 451 | 65.44 | 85.76 | 69.72 | 99.96 | 18284 | 7 | 23974 | 10 | 66.67 | 86.99 | 70.95 |
| XML_PRETTY | opt | 17760 | 25993 | 43753 | 2.32 | 207.19 | 96.78 | 17188 | 572 | 25156 | 837 | 66.11 | 73.04 | 64.55 | 99.87 | 17737 | 23 | 25959 | 34 | 68.17 | 75.10 | 66.61 |
| YAML | man | 11649 | 23478 | 35127 | 2.28 | 186.96 | 98.92 | 11523 | 126 | 23224 | 254 | 85.50 | 89.27 | 90.94 | 99.96 | 11644 | 5 | 23468 | 9 | 86.19 | 89.96 | 91.63 |
| YAML | opt | 11404 | 25243 | 36647 | 2.30 | 201.21 | 96.77 | 11036 | 368 | 24427 | 815 | 84.78 | 77.45 | 85.11 | 99.83 | 11385 | 19 | 25200 | 43 | 86.82 | 79.49 | 87.15 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7498 | 6970 | -528 | -7.04 | 294 | 295 | +1 | +0.34 | 27136 | 24988 | -2148 | -7.92 | 27431 | 25284 | -2147 | -7.83 | 34929 | 32254 | -2675 | -7.66 |
| JSON_PRETTY | 15787 | 17688 | +1901 | +12.04 | 297 | 297 | 0 | 0.00 | 21490 | 24887 | +3397 | +15.81 | 21787 | 25184 | +3397 | +15.59 | 37574 | 42872 | +5298 | +14.10 |
| TOON_DEFAULT | 11440 | 11195 | -245 | -2.14 | 205 | 300 | +95 | +46.34 | 24935 | 25697 | +762 | +3.06 | 25140 | 25997 | +857 | +3.41 | 36580 | 37192 | +612 | +1.67 |
| TOON_KEYFOLD | 11289 | 11044 | -245 | -2.17 | 308 | 210 | -98 | -31.82 | 25966 | 25466 | -500 | -1.93 | 26274 | 25676 | -598 | -2.28 | 37563 | 36720 | -843 | -2.24 |
| XML_COMPACT | 10164 | 12494 | +2330 | +22.92 | 302 | 204 | -98 | -32.45 | 26331 | 25350 | -981 | -3.73 | 26633 | 25554 | -1079 | -4.05 | 36797 | 38048 | +1251 | +3.40 |
| XML_PRETTY | 18291 | 17760 | -531 | -2.90 | 300 | 301 | +1 | +0.33 | 23683 | 25692 | +2009 | +8.48 | 23983 | 25993 | +2010 | +8.38 | 42274 | 43753 | +1479 | +3.50 |
| YAML | 11649 | 11404 | -245 | -2.10 | 295 | 293 | -2 | -0.68 | 23183 | 24950 | +1767 | +7.62 | 23478 | 25243 | +1765 | +7.52 | 35127 | 36647 | +1520 | +4.33 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Before Write (ms) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) | Output (ms) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 35 | 214.23 | 1.13 | 297392 | 65783 | 0.41 | 530.51 | 65818 | 214.64 | 424.63 | 363175 |
| JSON_COMPACT | opt | 19 | 366.84 | 0.61 | 270071 | 60679 | 0.41 | 489.35 | 60698 | 367.25 | 391.60 | 330751 |
| JSON_PRETTY | man | 319 | 49.49 | 10.29 | 218730 | 73203 | 0.29 | 590.35 | 73522 | 49.78 | 474.34 | 291933 |
| JSON_PRETTY | opt | 331 | 53.44 | 10.68 | 259327 | 64776 | 0.38 | 522.39 | 65107 | 53.82 | 420.05 | 324103 |
| TOON_DEFAULT | man | 54 | 211.85 | 1.74 | 258201 | 69965 | 0.36 | 564.23 | 70019 | 212.21 | 451.74 | 328166 |
| TOON_DEFAULT | opt | 40 | 279.88 | 1.29 | 269460 | 60162 | 0.43 | 485.18 | 60202 | 280.30 | 388.40 | 329622 |
| TOON_KEYFOLD | man | 35 | 322.54 | 1.13 | 255274 | 84221 | 0.31 | 679.20 | 84256 | 322.85 | 543.59 | 339495 |
| TOON_KEYFOLD | opt | 41 | 269.37 | 1.32 | 263619 | 60859 | 0.42 | 490.80 | 60900 | 269.78 | 392.90 | 324478 |
| XML_COMPACT | man | 110 | 92.40 | 3.55 | 269346 | 67910 | 0.39 | 547.66 | 68020 | 92.79 | 438.84 | 337256 |
| XML_COMPACT | opt | 116 | 107.71 | 3.74 | 271317 | 59599 | 0.43 | 480.64 | 59715 | 108.13 | 385.26 | 330916 |
| XML_PRETTY | man | 37 | 494.35 | 1.19 | 241360 | 67412 | 0.35 | 543.65 | 67449 | 494.70 | 435.15 | 308772 |
| XML_PRETTY | opt | 42 | 422.86 | 1.35 | 279603 | 58734 | 0.44 | 473.66 | 58776 | 423.29 | 379.20 | 338337 |
| YAML | man | 85 | 137.05 | 2.74 | 244358 | 71271 | 0.33 | 574.77 | 71356 | 137.37 | 460.36 | 315630 |
| YAML | opt | 81 | 140.79 | 2.61 | 263992 | 59415 | 0.42 | 479.15 | 59496 | 141.21 | 383.85 | 323407 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Before Write Man (s) | Output Before Write Opt (s) | Diff (s) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 35 | 19 | -16 | -45.71 | 297.39 | 270.07 | -27.32 | -9.19 | 65.78 | 60.68 | -5.10 | -7.76 | 65.82 | 60.70 | -5.12 | -7.78 | 363.17 | 330.75 | -32.42 | -8.93 |
| JSON_PRETTY | 319 | 331 | +12 | +3.76 | 218.73 | 259.33 | +40.60 | +18.56 | 73.20 | 64.78 | -8.43 | -11.51 | 73.52 | 65.11 | -8.41 | -11.45 | 291.93 | 324.10 | +32.17 | +11.02 |
| TOON_DEFAULT | 54 | 40 | -14 | -25.93 | 258.20 | 269.46 | +11.26 | +4.36 | 69.96 | 60.16 | -9.80 | -14.01 | 70.02 | 60.20 | -9.82 | -14.02 | 328.17 | 329.62 | +1.46 | +0.44 |
| TOON_KEYFOLD | 35 | 41 | +6 | +17.14 | 255.27 | 263.62 | +8.34 | +3.27 | 84.22 | 60.86 | -23.36 | -27.74 | 84.26 | 60.90 | -23.36 | -27.72 | 339.50 | 324.48 | -15.02 | -4.42 |
| XML_COMPACT | 110 | 116 | +6 | +5.45 | 269.35 | 271.32 | +1.97 | +0.73 | 67.91 | 59.60 | -8.31 | -12.24 | 68.02 | 59.72 | -8.30 | -12.21 | 337.26 | 330.92 | -6.34 | -1.88 |
| XML_PRETTY | 37 | 42 | +5 | +13.51 | 241.36 | 279.60 | +38.24 | +15.84 | 67.41 | 58.73 | -8.68 | -12.87 | 67.45 | 58.78 | -8.67 | -12.86 | 308.77 | 338.34 | +29.56 | +9.57 |
| YAML | 85 | 81 | -4 | -4.71 | 244.36 | 263.99 | +19.63 | +8.03 | 71.27 | 59.42 | -11.86 | -16.64 | 71.36 | 59.50 | -11.86 | -16.62 | 315.63 | 323.41 | +7.78 | +2.46 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token | Info / Read Token (Acc By Char) | Info / Output Token (Acc By Char) | Info / Total Token (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 3.19 | 10.99 | 241.87 | 1.32 | 0.36 | 0.28 | 1.33 | 0.36 | 0.28 |
| JSON_COMPACT | opt | 3.23 | 11.05 | 224.84 | 1.40 | 0.39 | 0.30 | 1.43 | 0.40 | 0.31 |
| JSON_PRETTY | man | 2.09 | 23.15 | 509.26 | 0.60 | 0.44 | 0.25 | 0.62 | 0.45 | 0.26 |
| JSON_PRETTY | opt | 1.77 | 28.03 | 570.58 | 0.56 | 0.39 | 0.23 | 0.56 | 0.40 | 0.23 |
| TOON_DEFAULT | man | 2.37 | 16.77 | 369.03 | 0.87 | 0.40 | 0.27 | 0.87 | 0.40 | 0.27 |
| TOON_DEFAULT | opt | 2.39 | 17.74 | 361.13 | 0.86 | 0.37 | 0.26 | 0.89 | 0.38 | 0.27 |
| TOON_KEYFOLD | man | 2.38 | 16.55 | 364.16 | 0.87 | 0.37 | 0.26 | 0.88 | 0.38 | 0.26 |
| TOON_KEYFOLD | opt | 2.41 | 17.50 | 356.26 | 0.87 | 0.37 | 0.26 | 0.90 | 0.39 | 0.27 |
| XML_COMPACT | man | 3.37 | 14.90 | 327.87 | 0.98 | 0.37 | 0.27 | 0.98 | 0.38 | 0.27 |
| XML_COMPACT | opt | 2.64 | 19.80 | 403.03 | 0.78 | 0.38 | 0.26 | 0.80 | 0.39 | 0.26 |
| XML_PRETTY | man | 2.32 | 26.82 | 590.03 | 0.54 | 0.41 | 0.23 | 0.55 | 0.42 | 0.24 |
| XML_PRETTY | opt | 2.32 | 28.15 | 572.90 | 0.55 | 0.37 | 0.22 | 0.56 | 0.38 | 0.23 |
| YAML | man | 2.28 | 17.08 | 375.77 | 0.85 | 0.42 | 0.28 | 0.86 | 0.43 | 0.28 |
| YAML | opt | 2.30 | 18.07 | 367.87 | 0.85 | 0.38 | 0.26 | 0.88 | 0.40 | 0.27 |

#### 2.5.2 Characters And Values: Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 3.19 | 3.23 | +0.04 | +1.13 | 10.99 | 11.05 | +0.05 | +0.47 | 241.87 | 224.84 | -17.03 | -7.04 |
| JSON_PRETTY | 2.09 | 1.77 | -0.33 | -15.57 | 23.15 | 28.03 | +4.88 | +21.10 | 509.26 | 570.58 | +61.32 | +12.04 |
| TOON_DEFAULT | 2.37 | 2.39 | +0.02 | +1.01 | 16.77 | 17.74 | +0.97 | +5.77 | 369.03 | 361.13 | -7.90 | -2.14 |
| TOON_KEYFOLD | 2.38 | 2.41 | +0.02 | +1.05 | 16.55 | 17.50 | +0.95 | +5.73 | 364.16 | 356.26 | -7.90 | -2.17 |
| XML_COMPACT | 3.37 | 2.64 | -0.73 | -21.66 | 14.90 | 19.80 | +4.90 | +32.86 | 327.87 | 403.03 | +75.16 | +22.92 |
| XML_PRETTY | 2.32 | 2.32 | 0.00 | 0.00 | 26.82 | 28.15 | +1.33 | +4.94 | 590.03 | 572.90 | -17.13 | -2.90 |
| YAML | 2.28 | 2.30 | +0.02 | +0.92 | 17.08 | 18.07 | +0.99 | +5.81 | 375.77 | 367.87 | -7.90 | -2.10 |

#### 2.5.3 Information: Mandatory vs Optional
| Format | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 1.32 | 1.40 | +0.09 | +6.69 | 0.36 | 0.39 | +0.03 | +7.50 | 0.28 | 0.30 | +0.02 | +7.45 |
| JSON_PRETTY | 0.60 | 0.56 | -0.05 | -7.62 | 0.44 | 0.39 | -0.05 | -10.50 | 0.25 | 0.23 | -0.02 | -9.45 |
| TOON_DEFAULT | 0.87 | 0.86 | 0.00 | 0.00 | 0.40 | 0.37 | -0.02 | -5.82 | 0.27 | 0.26 | -0.01 | -4.06 |
| TOON_KEYFOLD | 0.87 | 0.87 | 0.00 | 0.00 | 0.37 | 0.37 | 0.00 | 0.00 | 0.26 | 0.26 | 0.00 | 0.00 |
| XML_COMPACT | 0.98 | 0.78 | -0.19 | -19.77 | 0.37 | 0.38 | +0.01 | +2.96 | 0.27 | 0.26 | -0.01 | -4.81 |
| XML_PRETTY | 0.54 | 0.55 | +0.01 | +1.68 | 0.41 | 0.37 | -0.04 | -9.05 | 0.23 | 0.22 | -0.01 | -4.74 |
| YAML | 0.85 | 0.85 | 0.00 | 0.00 | 0.42 | 0.38 | -0.04 | -9.03 | 0.28 | 0.26 | -0.02 | -6.38 |

#### 2.5.4 Information (Accuracy By Character): Mandatory vs Optional
| Format | Info / Read Token (Acc By Char) Man | Info / Read Token (Acc By Char) Opt | Diff | Diff (%) | Info / Output Token (Acc By Char) Man | Info / Output Token (Acc By Char)  Opt | Diff | Diff (%) | Info / Total Token (Acc By Char) Man | Info / Total Token (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 1.33 | 1.43 | +0.10 | +7.83 | 0.36 | 0.40 | +0.03 | +8.82 | 0.28 | 0.31 | +0.03 | +8.77 |
| JSON_PRETTY | 0.62 | 0.56 | -0.06 | -9.46 | 0.45 | 0.40 | -0.05 | -12.17 | 0.26 | 0.23 | -0.03 | -11.07 |
| TOON_DEFAULT | 0.87 | 0.89 | +0.02 | +2.06 | 0.40 | 0.38 | -0.01 | -3.52 | 0.27 | 0.27 | -0.01 | -1.83 |
| TOON_KEYFOLD | 0.88 | 0.90 | +0.03 | +2.85 | 0.38 | 0.39 | +0.01 | +2.92 | 0.26 | 0.27 | +0.01 | +2.65 |
| XML_COMPACT | 0.98 | 0.80 | -0.18 | -18.80 | 0.38 | 0.39 | +0.02 | +4.27 | 0.27 | 0.26 | -0.01 | -3.68 |
| XML_PRETTY | 0.55 | 0.56 | +0.02 | +2.93 | 0.42 | 0.38 | -0.03 | -7.91 | 0.24 | 0.23 | -0.01 | -3.39 |
| YAML | 0.86 | 0.88 | +0.02 | +1.98 | 0.43 | 0.40 | -0.03 | -7.28 | 0.28 | 0.27 | -0.01 | -4.56 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 7498 | 7397 | 101 | 27431 | 27060 | 370 | 34929 | 34457 | 472 | 98.65 | 97.52 | 65.83 | 91.33 | 98.42 | 97.36 | 65.67 | 91.18 |
| JSON_COMPACT | opt | 6970 | 6820 | 150 | 25284 | 24740 | 544 | 32254 | 31560 | 693 | 97.85 | 98.54 | 77.93 | 98.54 | 98.09 | 98.70 | 78.09 | 98.70 |
| JSON_PRETTY | man | 15787 | 15066 | 721 | 21787 | 20791 | 996 | 37574 | 35857 | 1717 | 95.43 | 71.01 | 96.89 | 81.53 | 94.21 | 70.19 | 96.08 | 80.72 |
| JSON_PRETTY | opt | 17688 | 17451 | 237 | 25184 | 24847 | 337 | 42872 | 42298 | 574 | 98.66 | 67.57 | 79.05 | 68.35 | 99.01 | 67.81 | 79.29 | 68.58 |
| TOON_DEFAULT | man | 11440 | 11347 | 93 | 25140 | 24936 | 204 | 36580 | 36283 | 296 | 99.19 | 86.29 | 79.67 | 86.91 | 99.01 | 86.17 | 79.55 | 86.79 |
| TOON_DEFAULT | opt | 11195 | 10833 | 362 | 25997 | 25157 | 840 | 37192 | 35990 | 1201 | 96.77 | 85.40 | 73.01 | 83.53 | 96.65 | 85.32 | 72.93 | 83.45 |
| TOON_KEYFOLD | man | 11289 | 11046 | 243 | 26274 | 25709 | 565 | 37563 | 36755 | 808 | 97.85 | 85.84 | 72.10 | 83.17 | 97.53 | 85.63 | 71.89 | 82.96 |
| TOON_KEYFOLD | opt | 11044 | 10569 | 475 | 25676 | 24572 | 1104 | 36720 | 35141 | 1579 | 95.70 | 85.13 | 74.19 | 84.18 | 95.49 | 84.99 | 74.05 | 84.04 |
| XML_COMPACT | man | 10164 | 10082 | 82 | 26633 | 26417 | 216 | 36797 | 36499 | 298 | 99.19 | 90.04 | 70.88 | 86.28 | 99.14 | 90.01 | 70.85 | 86.25 |
| XML_COMPACT | opt | 12494 | 12225 | 269 | 25554 | 25005 | 549 | 38048 | 37230 | 818 | 97.85 | 82.30 | 76.33 | 81.77 | 98.25 | 82.57 | 76.60 | 82.04 |
| XML_PRETTY | man | 18291 | 17947 | 344 | 23983 | 23532 | 451 | 42274 | 41480 | 795 | 98.12 | 65.44 | 85.76 | 69.72 | 97.82 | 65.24 | 85.56 | 69.52 |
| XML_PRETTY | opt | 17760 | 17188 | 572 | 25993 | 25156 | 837 | 43753 | 42344 | 1409 | 96.78 | 66.11 | 73.04 | 64.55 | 96.83 | 66.14 | 73.07 | 64.58 |
| YAML | man | 11649 | 11523 | 126 | 23478 | 23224 | 254 | 35127 | 34747 | 379 | 98.92 | 85.50 | 89.27 | 90.94 | 98.91 | 85.49 | 89.26 | 90.93 |
| YAML | opt | 11404 | 11036 | 368 | 25243 | 24427 | 815 | 36647 | 35463 | 1184 | 96.77 | 84.78 | 77.45 | 85.11 | 96.87 | 84.85 | 77.52 | 85.17 |

#### 2.6.2 Read Tokens: Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7498 | 6970 | -528 | -7.04 | 7397 | 6820 | -577 | -7.80 | 101 | 150 | +49 | +48.15 | 98.65 | 97.85 | -0.80 | 97.52 | 98.54 | +1.02 | +1.04 | 98.42 | 98.09 | -0.33 | 97.36 | 98.70 | +1.33 | +1.37 |
| JSON_PRETTY | 15787 | 17688 | +1901 | +12.04 | 15066 | 17451 | +2385 | +15.83 | 721 | 237 | -484 | -67.19 | 95.43 | 98.66 | +3.23 | 71.01 | 67.57 | -3.43 | -4.84 | 94.21 | 99.01 | +4.80 | 70.19 | 67.81 | -2.39 | -3.40 |
| TOON_DEFAULT | 11440 | 11195 | -245 | -2.14 | 11347 | 10833 | -514 | -4.53 | 93 | 362 | +269 | +289.18 | 99.19 | 96.77 | -2.42 | 86.29 | 85.40 | -0.89 | -1.04 | 99.01 | 96.65 | -2.36 | 86.17 | 85.32 | -0.85 | -0.99 |
| TOON_KEYFOLD | 11289 | 11044 | -245 | -2.17 | 11046 | 10569 | -477 | -4.32 | 243 | 475 | +232 | +95.55 | 97.85 | 95.70 | -2.15 | 85.84 | 85.13 | -0.71 | -0.83 | 97.53 | 95.49 | -2.04 | 85.63 | 84.99 | -0.64 | -0.75 |
| XML_COMPACT | 10164 | 12494 | +2330 | +22.92 | 10082 | 12226 | +2144 | +21.26 | 82 | 268 | +186 | +227.19 | 99.19 | 97.85 | -1.34 | 90.04 | 82.30 | -7.74 | -8.60 | 99.14 | 98.25 | -0.89 | 90.01 | 82.57 | -7.44 | -8.27 |
| XML_PRETTY | 18291 | 17760 | -531 | -2.90 | 17947 | 17188 | -759 | -4.23 | 344 | 572 | +228 | +66.28 | 98.12 | 96.78 | -1.34 | 65.44 | 66.11 | +0.67 | +1.02 | 97.82 | 96.83 | -0.99 | 65.24 | 66.14 | +0.90 | +1.38 |
| YAML | 11649 | 11404 | -245 | -2.10 | 11523 | 11035 | -488 | -4.23 | 126 | 369 | +243 | +192.49 | 98.92 | 96.77 | -2.15 | 85.50 | 84.78 | -0.71 | -0.83 | 98.91 | 96.87 | -2.04 | 85.49 | 84.85 | -0.64 | -0.75 |

#### 2.6.3 Output Tokens: Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 27431 | 25284 | -2147 | -7.83 | 27060 | 24740 | -2320 | -8.57 | 370 | 543 | +173 | +46.83 | 98.65 | 97.85 | -0.80 | 65.83 | 77.93 | +12.10 | +18.39 | 98.42 | 98.09 | -0.33 | 65.67 | 78.09 | +12.42 | +18.91 |
| JSON_PRETTY | 21787 | 25184 | +3397 | +15.59 | 20791 | 24846 | +4055 | +19.50 | 996 | 338 | -658 | -66.08 | 95.43 | 98.66 | +3.23 | 96.89 | 79.05 | -17.84 | -18.41 | 94.21 | 99.01 | +4.80 | 96.08 | 79.29 | -16.79 | -17.48 |
| TOON_DEFAULT | 25140 | 25997 | +857 | +3.41 | 24936 | 25157 | +221 | +0.89 | 204 | 840 | +636 | +311.79 | 99.19 | 96.77 | -2.42 | 79.67 | 73.01 | -6.66 | -8.36 | 99.01 | 96.65 | -2.36 | 79.55 | 72.93 | -6.62 | -8.32 |
| TOON_KEYFOLD | 26274 | 25676 | -598 | -2.28 | 25709 | 24572 | -1137 | -4.42 | 565 | 1104 | +539 | +95.43 | 97.85 | 95.70 | -2.15 | 72.10 | 74.19 | +2.09 | +2.89 | 97.53 | 95.49 | -2.04 | 71.89 | 74.05 | +2.16 | +3.00 |
| XML_COMPACT | 26633 | 25554 | -1079 | -4.05 | 26417 | 25005 | -1412 | -5.35 | 216 | 550 | +334 | +154.49 | 99.19 | 97.85 | -1.34 | 70.88 | 76.33 | +5.46 | +7.70 | 99.14 | 98.25 | -0.89 | 70.85 | 76.60 | +5.75 | +8.12 |
| XML_PRETTY | 23983 | 25993 | +2010 | +8.38 | 23532 | 25156 | +1624 | +6.90 | 451 | 837 | +386 | +85.61 | 98.12 | 96.78 | -1.34 | 85.76 | 73.04 | -12.72 | -14.83 | 97.82 | 96.83 | -0.99 | 85.56 | 73.07 | -12.49 | -14.60 |
| YAML | 23478 | 25243 | +1765 | +7.52 | 23224 | 24427 | +1203 | +5.18 | 254 | 816 | +562 | +221.17 | 98.92 | 96.77 | -2.15 | 89.27 | 77.45 | -11.82 | -13.24 | 98.91 | 96.87 | -2.04 | 89.26 | 77.52 | -11.75 | -13.16 |

#### 2.6.4 Total Tokens: Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 34929 | 32254 | -2675 | -7.66 | 34457 | 31560 | -2897 | -8.41 | 472 | 694 | +222 | +47.02 | 98.65 | 97.85 | -0.80 | 91.33 | 98.54 | +7.21 | +7.89 | 98.42 | 98.09 | -0.33 | 91.18 | 98.70 | +7.52 | +8.25 |
| JSON_PRETTY | 37574 | 42872 | +5298 | +14.10 | 35857 | 42298 | +6441 | +17.96 | 1717 | 574 | -1143 | -66.55 | 95.43 | 98.66 | +3.23 | 81.53 | 68.35 | -13.18 | -16.16 | 94.21 | 99.01 | +4.80 | 80.72 | 68.58 | -12.13 | -15.03 |
| TOON_DEFAULT | 36580 | 37192 | +612 | +1.67 | 36283 | 35990 | -293 | -0.81 | 296 | 1201 | +905 | +305.74 | 99.19 | 96.77 | -2.42 | 86.91 | 83.53 | -3.38 | -3.89 | 99.01 | 96.65 | -2.36 | 86.79 | 83.45 | -3.34 | -3.85 |
| TOON_KEYFOLD | 37563 | 36720 | -843 | -2.24 | 36755 | 35141 | -1614 | -4.39 | 808 | 1579 | +771 | +95.46 | 97.85 | 95.70 | -2.15 | 83.17 | 84.18 | +1.01 | +1.21 | 97.53 | 95.49 | -2.04 | 82.96 | 84.04 | +1.08 | +1.30 |
| XML_COMPACT | 36797 | 38048 | +1251 | +3.40 | 36499 | 37230 | +731 | +2.00 | 298 | 818 | +520 | +174.49 | 99.19 | 97.85 | -1.34 | 86.28 | 81.77 | -4.51 | -5.23 | 99.14 | 98.25 | -0.89 | 86.25 | 82.04 | -4.21 | -4.89 |
| XML_PRETTY | 42274 | 43753 | +1479 | +3.50 | 41480 | 42345 | +865 | +2.08 | 795 | 1409 | +614 | +77.24 | 98.12 | 96.78 | -1.34 | 69.72 | 64.55 | -5.17 | -7.42 | 97.82 | 96.83 | -0.99 | 69.52 | 64.58 | -4.94 | -7.10 |
| YAML | 35127 | 36647 | +1520 | +4.33 | 34747 | 35463 | +716 | +2.06 | 379 | 1183 | +804 | +212.22 | 98.92 | 96.77 | -2.15 | 90.94 | 85.11 | -5.83 | -6.41 | 98.91 | 96.87 | -2.04 | 90.93 | 85.17 | -5.76 | -6.33 |

### 2.7 Token Utilization Efficiency (Accuracy by Character)
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy by Character (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy by Character (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 7498 | 7473 | 25 | 27431 | 27337 | 93 | 34929 | 34810 | 119 | 99.66 | 98.19 | 66.50 | 92.00 | 99.53 | 98.10 | 66.41 | 91.92 |
| JSON_COMPACT | opt | 6970 | 6962 | 8 | 25284 | 25253 | 30 | 32254 | 32215 | 39 | 99.88 | 99.89 | 79.28 | 99.89 | 99.31 | 99.51 | 78.90 | 99.51 |
| JSON_PRETTY | man | 15787 | 15555 | 232 | 21787 | 21467 | 320 | 37574 | 37022 | 552 | 98.53 | 73.08 | 98.96 | 83.60 | 98.60 | 73.12 | 99.01 | 83.64 |
| JSON_PRETTY | opt | 17688 | 17674 | 14 | 25184 | 25164 | 20 | 42872 | 42838 | 34 | 99.92 | 68.41 | 79.89 | 69.19 | 99.50 | 68.13 | 79.61 | 68.91 |
| TOON_DEFAULT | man | 11440 | 11439 | 1 | 25140 | 25137 | 3 | 36580 | 36576 | 4 | 99.99 | 86.83 | 80.20 | 87.45 | 99.81 | 86.71 | 80.08 | 87.33 |
| TOON_DEFAULT | opt | 11195 | 11176 | 19 | 25997 | 25952 | 44 | 37192 | 37128 | 63 | 99.83 | 87.44 | 75.05 | 85.57 | 99.37 | 87.13 | 74.75 | 85.26 |
| TOON_KEYFOLD | man | 11289 | 11183 | 106 | 26274 | 26027 | 247 | 37563 | 37210 | 353 | 99.06 | 86.65 | 72.91 | 83.98 | 99.21 | 86.75 | 73.01 | 84.08 |
| TOON_KEYFOLD | opt | 11044 | 10997 | 47 | 25676 | 25566 | 110 | 36720 | 36562 | 158 | 99.57 | 87.71 | 76.77 | 86.76 | 98.83 | 87.22 | 76.27 | 86.27 |
| XML_COMPACT | man | 10164 | 10162 | 2 | 26633 | 26628 | 5 | 36797 | 36790 | 7 | 99.98 | 90.57 | 71.41 | 86.81 | 99.80 | 90.45 | 71.29 | 86.69 |
| XML_COMPACT | opt | 12494 | 12473 | 21 | 25554 | 25511 | 43 | 38048 | 37984 | 65 | 99.83 | 83.62 | 77.66 | 83.09 | 99.22 | 83.21 | 77.25 | 82.68 |
| XML_PRETTY | man | 18291 | 18284 | 7 | 23983 | 23974 | 10 | 42274 | 42257 | 17 | 99.96 | 66.67 | 86.99 | 70.95 | 99.55 | 66.40 | 86.71 | 70.67 |
| XML_PRETTY | opt | 17760 | 17737 | 23 | 25993 | 25959 | 34 | 43753 | 43696 | 57 | 99.87 | 68.17 | 75.10 | 66.61 | 98.95 | 67.56 | 74.49 | 66.00 |
| YAML | man | 11649 | 11644 | 5 | 23478 | 23468 | 9 | 35127 | 35113 | 14 | 99.96 | 86.19 | 89.96 | 91.63 | 99.80 | 86.08 | 89.86 | 91.52 |
| YAML | opt | 11404 | 11385 | 19 | 25243 | 25200 | 43 | 36647 | 36584 | 62 | 99.83 | 86.82 | 79.49 | 87.15 | 99.12 | 86.35 | 79.02 | 86.67 |

#### 2.7.2 Read Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7498 | 6970 | -528 | -7.04 | 7473 | 6962 | -511 | -6.84 | 25 | 8 | -17 | -68.52 | 99.66 | 99.88 | +0.22 | 98.19 | 99.89 | +1.70 | +1.73 | 99.53 | 99.31 | -0.22 | 98.10 | 99.51 | +1.41 | +1.43 |
| JSON_PRETTY | 15787 | 17688 | +1901 | +12.04 | 15555 | 17674 | +2119 | +13.62 | 232 | 14 | -218 | -93.93 | 98.53 | 99.92 | +1.39 | 73.08 | 68.41 | -4.66 | -6.38 | 98.60 | 99.50 | +0.90 | 73.12 | 68.13 | -4.99 | -6.82 |
| TOON_DEFAULT | 11440 | 11195 | -245 | -2.14 | 11439 | 11176 | -263 | -2.30 | 1 | 19 | +18 | +1788.80 | 99.99 | 99.83 | -0.16 | 86.83 | 87.44 | +0.61 | +0.71 | 99.81 | 99.37 | -0.44 | 86.71 | 87.13 | +0.43 | +0.49 |
| TOON_KEYFOLD | 11289 | 11044 | -245 | -2.17 | 11183 | 10997 | -186 | -1.67 | 106 | 47 | -59 | -55.31 | 99.06 | 99.57 | +0.51 | 86.65 | 87.71 | +1.06 | +1.22 | 99.21 | 98.83 | -0.38 | 86.75 | 87.22 | +0.47 | +0.54 |
| XML_COMPACT | 10164 | 12494 | +2330 | +22.92 | 10162 | 12473 | +2311 | +22.74 | 2 | 21 | +19 | +960.35 | 99.98 | 99.83 | -0.15 | 90.57 | 83.62 | -6.95 | -7.67 | 99.80 | 99.22 | -0.58 | 90.45 | 83.21 | -7.23 | -8.00 |
| XML_PRETTY | 18291 | 17760 | -531 | -2.90 | 18284 | 17737 | -547 | -2.99 | 7 | 23 | +16 | +225.31 | 99.96 | 99.87 | -0.09 | 66.67 | 68.17 | +1.50 | +2.25 | 99.55 | 98.95 | -0.60 | 66.40 | 67.56 | +1.16 | +1.75 |
| YAML | 11649 | 11404 | -245 | -2.10 | 11644 | 11384 | -260 | -2.23 | 5 | 20 | +15 | +294.54 | 99.96 | 99.83 | -0.13 | 86.19 | 86.82 | +0.63 | +0.73 | 99.80 | 99.12 | -0.68 | 86.08 | 86.35 | +0.27 | +0.31 |

#### 2.7.3 Output Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 27431 | 25284 | -2147 | -7.83 | 27337 | 25253 | -2084 | -7.62 | 93 | 30 | -63 | -67.66 | 99.66 | 99.88 | +0.22 | 66.50 | 79.28 | +12.78 | +19.22 | 99.53 | 99.31 | -0.22 | 66.41 | 78.90 | +12.49 | +18.81 |
| JSON_PRETTY | 21787 | 25184 | +3397 | +15.59 | 21467 | 25164 | +3697 | +17.22 | 320 | 20 | -300 | -93.79 | 98.53 | 99.92 | +1.39 | 98.96 | 79.89 | -19.07 | -19.27 | 98.60 | 99.50 | +0.90 | 99.01 | 79.61 | -19.39 | -19.59 |
| TOON_DEFAULT | 25140 | 25997 | +857 | +3.41 | 25137 | 25952 | +815 | +3.24 | 3 | 45 | +42 | +1389.33 | 99.99 | 99.83 | -0.16 | 80.20 | 75.05 | -5.15 | -6.42 | 99.81 | 99.37 | -0.44 | 80.08 | 74.75 | -5.34 | -6.66 |
| TOON_KEYFOLD | 26274 | 25676 | -598 | -2.28 | 26027 | 25566 | -461 | -1.77 | 247 | 110 | -137 | -55.29 | 99.06 | 99.57 | +0.51 | 72.91 | 76.77 | +3.86 | +5.29 | 99.21 | 98.83 | -0.38 | 73.01 | 76.27 | +3.27 | +4.47 |
| XML_COMPACT | 26633 | 25554 | -1079 | -4.05 | 26628 | 25511 | -1117 | -4.19 | 5 | 43 | +38 | +762.30 | 99.98 | 99.83 | -0.15 | 71.41 | 77.66 | +6.25 | +8.75 | 99.80 | 99.22 | -0.58 | 71.29 | 77.25 | +5.96 | +8.36 |
| XML_PRETTY | 23983 | 25993 | +2010 | +8.38 | 23974 | 25959 | +1985 | +8.28 | 10 | 34 | +24 | +241.98 | 99.96 | 99.87 | -0.09 | 86.99 | 75.10 | -11.89 | -13.67 | 99.55 | 98.95 | -0.60 | 86.71 | 74.49 | -12.23 | -14.10 |
| YAML | 23478 | 25243 | +1765 | +7.52 | 23468 | 25199 | +1731 | +7.38 | 9 | 43 | +34 | +372.47 | 99.96 | 99.83 | -0.13 | 89.96 | 79.49 | -10.47 | -11.64 | 99.80 | 99.12 | -0.68 | 89.86 | 79.02 | -10.84 | -12.07 |

#### 2.7.4 Total Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 34929 | 32254 | -2675 | -7.66 | 34810 | 32215 | -2595 | -7.45 | 119 | 39 | -80 | -67.27 | 99.66 | 99.88 | +0.22 | 92.00 | 99.89 | +7.89 | +8.57 | 99.53 | 99.31 | -0.22 | 91.92 | 99.51 | +7.59 | +8.26 |
| JSON_PRETTY | 37574 | 42872 | +5298 | +14.10 | 37022 | 42838 | +5816 | +15.71 | 552 | 34 | -518 | -93.85 | 98.53 | 99.92 | +1.39 | 83.60 | 69.19 | -14.40 | -17.23 | 98.60 | 99.50 | +0.90 | 83.64 | 68.91 | -14.73 | -17.61 |
| TOON_DEFAULT | 36580 | 37192 | +612 | +1.67 | 36576 | 37128 | +552 | +1.51 | 4 | 64 | +60 | +1489.20 | 99.99 | 99.83 | -0.16 | 87.45 | 85.57 | -1.88 | -2.15 | 99.81 | 99.37 | -0.44 | 87.33 | 85.26 | -2.06 | -2.36 |
| TOON_KEYFOLD | 37563 | 36720 | -843 | -2.24 | 37210 | 36562 | -648 | -1.74 | 353 | 158 | -195 | -55.30 | 99.06 | 99.57 | +0.51 | 83.98 | 86.76 | +2.78 | +3.31 | 99.21 | 98.83 | -0.38 | 84.08 | 86.27 | +2.19 | +2.60 |
| XML_COMPACT | 36797 | 38048 | +1251 | +3.40 | 36790 | 37984 | +1194 | +3.25 | 7 | 64 | +57 | +818.90 | 99.98 | 99.83 | -0.15 | 86.81 | 83.09 | -3.72 | -4.29 | 99.80 | 99.22 | -0.58 | 86.69 | 82.68 | -4.01 | -4.62 |
| XML_PRETTY | 42274 | 43753 | +1479 | +3.50 | 42257 | 43696 | +1439 | +3.40 | 17 | 57 | +40 | +235.11 | 99.96 | 99.87 | -0.09 | 70.95 | 66.61 | -4.34 | -6.12 | 99.55 | 98.95 | -0.60 | 70.67 | 66.00 | -4.68 | -6.62 |
| YAML | 35127 | 36647 | +1520 | +4.33 | 35113 | 36585 | +1472 | +4.19 | 14 | 62 | +48 | +344.63 | 99.96 | 99.83 | -0.13 | 91.63 | 87.15 | -4.48 | -4.89 | 99.80 | 99.12 | -0.68 | 91.52 | 86.67 | -4.85 | -5.30 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Expected Characters | Output Characters | Correct Characters | Incorrect Characters | Accuracy by Character (%) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 122 | 2 | 0 | 98.65 | 7782 | 7791 | 7765 | 26 | 99.66 |
| JSON_COMPACT | opt | 121 | 3 | 0 | 97.85 | 8451 | 8452 | 8442 | 10 | 99.88 |
| JSON_PRETTY | man | 118 | 6 | 0 | 95.43 | 7782 | 7894 | 7777 | 117 | 98.53 |
| JSON_PRETTY | opt | 122 | 2 | 0 | 98.66 | 8451 | 8452 | 8445 | 7 | 99.92 |
| TOON_DEFAULT | man | 123 | 1 | 0 | 99.19 | 7782 | 7782 | 7781 | 1 | 99.99 |
| TOON_DEFAULT | opt | 120 | 4 | 0 | 96.77 | 8451 | 8457 | 8443 | 14 | 99.83 |
| TOON_KEYFOLD | man | 121 | 3 | 0 | 97.85 | 7782 | 7846 | 7771 | 75 | 99.06 |
| TOON_KEYFOLD | opt | 119 | 5 | 0 | 95.70 | 8451 | 8477 | 8441 | 37 | 99.57 |
| XML_COMPACT | man | 123 | 1 | 0 | 99.19 | 7782 | 7782 | 7780 | 2 | 99.98 |
| XML_COMPACT | opt | 121 | 3 | 0 | 97.85 | 8451 | 8456 | 8442 | 14 | 99.83 |
| XML_PRETTY | man | 122 | 2 | 0 | 98.12 | 7782 | 7782 | 7779 | 3 | 99.96 |
| XML_PRETTY | opt | 120 | 4 | 0 | 96.78 | 8451 | 8452 | 8441 | 11 | 99.87 |
| YAML | man | 123 | 1 | 0 | 98.92 | 7782 | 7782 | 7779 | 3 | 99.96 |
| YAML | opt | 120 | 4 | 0 | 96.77 | 8451 | 8456 | 8441 | 14 | 99.83 |

#### 2.8.2 Answers: Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 122 | 121 | -1 | -0.82 | 2 | 3 | +1 | +50.00 | 0 | 0 | 0 | 0.00 | 98.65 | 97.85 | -0.80 |
| JSON_PRETTY | 118 | 122 | +4 | +3.39 | 6 | 2 | -4 | -66.67 | 0 | 0 | 0 | 0.00 | 95.43 | 98.66 | +3.23 |
| TOON_DEFAULT | 123 | 120 | -3 | -2.44 | 1 | 4 | +3 | +300.00 | 0 | 0 | 0 | 0.00 | 99.19 | 96.77 | -2.42 |
| TOON_KEYFOLD | 121 | 118 | -3 | -2.20 | 3 | 6 | +3 | +88.67 | 0 | 0 | 0 | 0.00 | 97.85 | 95.70 | -2.15 |
| XML_COMPACT | 123 | 121 | -2 | -1.36 | 1 | 3 | +2 | +167.00 | 0 | 0 | 0 | 0.00 | 99.19 | 97.85 | -1.34 |
| XML_PRETTY | 122 | 120 | -2 | -1.37 | 2 | 4 | +2 | +83.50 | 0 | 0 | 0 | 0.00 | 98.12 | 96.78 | -1.34 |
| YAML | 123 | 120 | -3 | -2.17 | 1 | 4 | +3 | +267.00 | 0 | 0 | 0 | 0.00 | 98.92 | 96.77 | -2.15 |

#### 2.8.3 Characters: Mandatory vs Optional Data
| Format | Output Characters Man | Output Characters Opt | Diff | Diff (%) | Correct Characters Man | Correct Characters Opt | Diff | Diff (%) | Incorrect Characters Man | Incorrect Characters Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7791 | 8452 | +661 | +8.48 | 7765 | 8442 | +677 | +8.71 | 26 | 10 | -16 | -61.54 | 99.66 | 99.88 | 0.22 |
| JSON_PRETTY | 7894 | 8452 | +558 | +7.06 | 7777 | 8445 | +668 | +8.59 | 117 | 7 | -110 | -94.30 | 98.53 | 99.92 | 1.39 |
| TOON_DEFAULT | 7782 | 8457 | +675 | +8.67 | 7781 | 8443 | +662 | +8.50 | 1 | 14 | +13 | +1333.30 | 99.99 | 99.83 | -0.16 |
| TOON_KEYFOLD | 7846 | 8477 | +631 | +8.05 | 7771 | 8441 | +670 | +8.62 | 75 | 37 | -38 | -51.11 | 99.06 | 99.57 | 0.51 |
| XML_COMPACT | 7782 | 8456 | +674 | +8.66 | 7780 | 8442 | +662 | +8.50 | 2 | 14 | +12 | +616.65 | 99.98 | 99.83 | -0.15 |
| XML_PRETTY | 7782 | 8452 | +670 | +8.61 | 7779 | 8441 | +662 | +8.51 | 3 | 11 | +8 | +266.67 | 99.96 | 99.87 | -0.09 |
| YAML | 7782 | 8456 | +674 | +8.66 | 7779 | 8442 | +663 | +8.52 | 3 | 14 | +11 | +366.67 | 99.96 | 99.83 | -0.13 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 98.65 | 99.39 | 98.77 | 95.24 | 100.00 | 99.53 | 37.27 | 28.81 | 19.84 | 12.50 |
| JSON_COMPACT | opt | 97.85 | 100.00 | 97.53 | 100.00 | 90.48 | 99.31 | 37.50 | 28.45 | 20.83 | 11.31 |
| JSON_PRETTY | man | 95.43 | 100.00 | 92.59 | 82.54 | 100.00 | 98.60 | 37.50 | 27.01 | 17.20 | 12.50 |
| JSON_PRETTY | opt | 98.66 | 100.00 | 100.00 | 100.00 | 92.07 | 99.50 | 37.50 | 29.17 | 20.83 | 11.51 |
| TOON_DEFAULT | man | 99.19 | 100.00 | 100.00 | 95.24 | 100.00 | 99.81 | 37.50 | 29.17 | 19.84 | 12.50 |
| TOON_DEFAULT | opt | 96.77 | 100.00 | 92.59 | 100.00 | 90.48 | 99.37 | 37.50 | 27.01 | 20.83 | 11.31 |
| TOON_KEYFOLD | man | 97.85 | 100.00 | 96.30 | 95.24 | 96.83 | 99.21 | 37.50 | 28.09 | 19.84 | 12.10 |
| TOON_KEYFOLD | opt | 95.70 | 100.00 | 93.83 | 93.65 | 88.89 | 98.83 | 37.50 | 27.37 | 19.51 | 11.11 |
| XML_COMPACT | man | 99.19 | 100.00 | 100.00 | 96.83 | 98.41 | 99.80 | 37.50 | 29.17 | 20.17 | 12.30 |
| XML_COMPACT | opt | 97.85 | 100.00 | 98.77 | 100.00 | 88.89 | 99.22 | 37.50 | 28.81 | 20.83 | 11.11 |
| XML_PRETTY | man | 98.12 | 100.00 | 100.00 | 90.47 | 98.41 | 99.55 | 37.50 | 29.17 | 18.85 | 12.30 |
| XML_PRETTY | opt | 96.78 | 100.00 | 100.00 | 90.47 | 90.48 | 98.95 | 37.50 | 29.17 | 18.85 | 11.31 |
| YAML | man | 98.92 | 100.00 | 98.77 | 98.41 | 96.83 | 99.80 | 37.50 | 28.81 | 20.50 | 12.10 |
| YAML | opt | 96.77 | 100.00 | 96.30 | 96.83 | 88.89 | 99.12 | 37.50 | 28.09 | 20.17 | 11.11 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 99.39 | 100.00 | +0.61 | 37.27 | 37.50 | +0.23 |
| JSON_PRETTY | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| TOON_DEFAULT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| TOON_KEYFOLD | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| XML_COMPACT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| XML_PRETTY | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| YAML | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 98.77 | 97.53 | -1.24 | 28.81 | 28.45 | -0.36 |
| JSON_PRETTY | 92.59 | 100.00 | +7.41 | 27.01 | 29.17 | +2.16 |
| TOON_DEFAULT | 100.00 | 92.59 | -7.41 | 29.17 | 27.01 | -2.16 |
| TOON_KEYFOLD | 96.30 | 93.83 | -2.47 | 28.09 | 27.37 | -0.72 |
| XML_COMPACT | 100.00 | 98.77 | -1.23 | 29.17 | 28.81 | -0.36 |
| XML_PRETTY | 100.00 | 100.00 | 0.00 | 29.17 | 29.17 | 0.00 |
| YAML | 98.77 | 96.30 | -2.47 | 28.81 | 28.09 | -0.72 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 95.24 | 100.00 | +4.76 | 19.84 | 20.83 | +0.99 |
| JSON_PRETTY | 82.54 | 100.00 | +17.46 | 17.20 | 20.83 | +3.63 |
| TOON_DEFAULT | 95.24 | 100.00 | +4.76 | 19.84 | 20.83 | +0.99 |
| TOON_KEYFOLD | 95.24 | 93.65 | -1.59 | 19.84 | 19.51 | -0.33 |
| XML_COMPACT | 96.83 | 100.00 | +3.17 | 20.17 | 20.83 | +0.66 |
| XML_PRETTY | 90.47 | 90.47 | 0.00 | 18.85 | 18.85 | 0.00 |
| YAML | 98.41 | 96.83 | -1.59 | 20.50 | 20.17 | -0.33 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 100.00 | 90.48 | -9.52 | 12.50 | 11.31 | -1.19 |
| JSON_PRETTY | 100.00 | 92.07 | -7.93 | 12.50 | 11.51 | -0.99 |
| TOON_DEFAULT | 100.00 | 90.48 | -9.52 | 12.50 | 11.31 | -1.19 |
| TOON_KEYFOLD | 96.83 | 88.89 | -7.94 | 12.10 | 11.11 | -0.99 |
| XML_COMPACT | 98.41 | 88.89 | -9.52 | 12.30 | 11.11 | -1.19 |
| XML_PRETTY | 98.41 | 90.48 | -7.93 | 12.30 | 11.31 | -0.99 |
| YAML | 96.83 | 88.89 | -7.94 | 12.10 | 11.11 | -0.99 |

### 2.10 Accuracy By Character Per Question Category Analysis
#### 2.10.1 Metrics
| Format | Variant | Accuracy By Character (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc By Char (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 99.66 | 99.31 | 99.91 | 99.11 | 100.00 | 99.53 | 37.24 | 29.14 | 20.64 | 12.50 |
| JSON_COMPACT | opt | 99.88 | 100.00 | 99.97 | 100.00 | 94.55 | 99.31 | 37.50 | 29.16 | 20.83 | 11.82 |
| JSON_PRETTY | man | 98.53 | 100.00 | 97.54 | 96.72 | 100.00 | 98.60 | 37.50 | 28.45 | 20.15 | 12.50 |
| JSON_PRETTY | opt | 99.92 | 100.00 | 100.00 | 100.00 | 95.96 | 99.50 | 37.50 | 29.17 | 20.83 | 12.00 |
| TOON_DEFAULT | man | 99.99 | 100.00 | 100.00 | 99.11 | 100.00 | 99.81 | 37.50 | 29.17 | 20.64 | 12.50 |
| TOON_DEFAULT | opt | 99.83 | 100.00 | 99.85 | 100.00 | 95.35 | 99.37 | 37.50 | 29.12 | 20.83 | 11.92 |
| TOON_KEYFOLD | man | 99.06 | 100.00 | 98.44 | 99.11 | 98.77 | 99.21 | 37.50 | 28.72 | 20.65 | 12.35 |
| TOON_KEYFOLD | opt | 99.57 | 100.00 | 99.44 | 98.82 | 93.94 | 98.83 | 37.50 | 29.00 | 20.59 | 11.74 |
| XML_COMPACT | man | 99.98 | 100.00 | 100.00 | 99.40 | 99.38 | 99.80 | 37.50 | 29.17 | 20.71 | 12.42 |
| XML_COMPACT | opt | 99.83 | 100.00 | 99.91 | 100.00 | 93.94 | 99.22 | 37.50 | 29.14 | 20.83 | 11.74 |
| XML_PRETTY | man | 99.96 | 100.00 | 100.00 | 98.21 | 99.38 | 99.55 | 37.50 | 29.17 | 20.46 | 12.42 |
| XML_PRETTY | opt | 99.87 | 100.00 | 100.00 | 98.23 | 94.55 | 98.95 | 37.50 | 29.17 | 20.46 | 11.82 |
| YAML | man | 99.96 | 100.00 | 99.97 | 99.70 | 98.97 | 99.80 | 37.50 | 29.16 | 20.77 | 12.37 |
| YAML | opt | 99.83 | 100.00 | 99.91 | 99.41 | 94.14 | 99.12 | 37.50 | 29.14 | 20.71 | 11.77 |

#### 2.10.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 99.31 | 100.00 | +0.69 | 37.24 | 37.50 | +0.26 |
| JSON_PRETTY | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| TOON_DEFAULT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| TOON_KEYFOLD | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| XML_COMPACT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| XML_PRETTY | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| YAML | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |

#### 2.10.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 99.91 | 99.97 | +0.06 | 29.14 | 29.16 | +0.02 |
| JSON_PRETTY | 97.54 | 100.00 | +2.46 | 28.45 | 29.17 | +0.72 |
| TOON_DEFAULT | 100.00 | 99.85 | -0.15 | 29.17 | 29.12 | -0.05 |
| TOON_KEYFOLD | 98.44 | 99.44 | +0.99 | 28.72 | 29.00 | +0.29 |
| XML_COMPACT | 100.00 | 99.91 | -0.09 | 29.17 | 29.14 | -0.03 |
| XML_PRETTY | 100.00 | 100.00 | 0.00 | 29.17 | 29.17 | 0.00 |
| YAML | 99.97 | 99.91 | -0.06 | 29.16 | 29.14 | -0.02 |

#### 2.10.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 99.11 | 100.00 | +0.89 | 20.64 | 20.83 | +0.19 |
| JSON_PRETTY | 96.72 | 100.00 | +3.28 | 20.15 | 20.83 | +0.68 |
| TOON_DEFAULT | 99.11 | 100.00 | +0.89 | 20.64 | 20.83 | +0.19 |
| TOON_KEYFOLD | 99.11 | 98.82 | -0.28 | 20.65 | 20.59 | -0.06 |
| XML_COMPACT | 99.40 | 100.00 | +0.60 | 20.71 | 20.83 | +0.12 |
| XML_PRETTY | 98.21 | 98.23 | +0.02 | 20.46 | 20.46 | +0.01 |
| YAML | 99.70 | 99.41 | -0.29 | 20.77 | 20.71 | -0.06 |

#### 2.10.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 100.00 | 94.55 | -5.45 | 12.50 | 11.82 | -0.68 |
| JSON_PRETTY | 100.00 | 95.96 | -4.04 | 12.50 | 12.00 | -0.50 |
| TOON_DEFAULT | 100.00 | 95.35 | -4.65 | 12.50 | 11.92 | -0.58 |
| TOON_KEYFOLD | 98.77 | 93.94 | -4.82 | 12.35 | 11.74 | -0.60 |
| XML_COMPACT | 99.38 | 93.94 | -5.44 | 12.42 | 11.74 | -0.68 |
| XML_PRETTY | 99.38 | 94.55 | -4.83 | 12.42 | 11.82 | -0.60 |
| YAML | 98.97 | 94.14 | -4.83 | 12.37 | 11.77 | -0.60 |

## 3. Appendices

### 3.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-29
- **Model**: Sonnet 4.6
- **Thinking**: off
- **Structure**: nested
- **Formats Tested**: JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, TOON_KEYFOLD, XML_COMPACT, XML_PRETTY, YAML
- **Record Counts**: 31
- **Total Test Cases**: 14

### 3.2 Appendix B: Benchmark Configuration
- **Field Retrieval**: 55 questions (37.50% weight)
- **Filtering**: 21 questions (20.83% weight)
- **Aggregation**: 21 questions (12.50% weight)
- **Structure Awareness**: 27 questions (29.17% weight)

---

- **Report Generated**: 2026-04-20
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code 2.1.80
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