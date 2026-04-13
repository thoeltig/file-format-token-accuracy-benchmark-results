# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: on
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
| JSON_COMPACT ≈ 69s | JSON_COMPACT ≈ 10315 | JSON_COMPACT ≈ 336 | XML_PRETTY ≈ 7175 | XML_PRETTY ≈ 7518 | JSON_COMPACT ≈ 18446 | YAML ≈ 75.80% | JSON_COMPACT ≈ 80 | XML_PRETTY ≈ 80 | JSON_COMPACT ≈ 81 | YAML ≈ 92.87% | JSON_COMPACT ≈ 92 | XML_PRETTY ≈ 93 | JSON_COMPACT ≈ 93 |
| XML_PRETTY (+2.18%) | XML_COMPACT (+24.56%) | YAML (+0.50%) | JSON_COMPACT (+8.64%) | JSON_COMPACT (+8.16%) | XML_COMPACT (+18.26%) | JSON_PRETTY (-1.88%) | XML_COMPACT (-10.87%) | JSON_COMPACT (-1.53%) | XML_COMPACT (-13.18%) | TOON_DEFAULT (-0.68%) | XML_COMPACT (-8.91%) | JSON_COMPACT (-2.93%) | XML_COMPACT (-10.94%) |
| TOON_DEFAULT (+6.95%) | TOON_DEFAULT (+36.66%) | TOON_DEFAULT (+1.74%) | TOON_DEFAULT (+16.27%) | TOON_DEFAULT (+15.52%) | TOON_DEFAULT (+23.50%) | JSON_COMPACT (-3.22%) | YAML (-13.39%) | TOON_DEFAULT (-5.77%) | TOON_DEFAULT (-17.33%) | JSON_PRETTY (-1.48%) | TOON_DEFAULT (-12.33%) | TOON_DEFAULT (-4.63%) | TOON_DEFAULT (-13.14%) |
| XML_COMPACT (+8.00%) | YAML (+38.69%) | XML_PRETTY (+1.88%) | XML_COMPACT (+20.16%) | XML_COMPACT (+19.27%) | YAML (+37.75%) | XML_COMPACT (-4.02%) | TOON_DEFAULT (-16.46%) | XML_COMPACT (-6.73%) | YAML (-23.25%) | XML_PRETTY (-1.76%) | YAML (-12.57%) | XML_COMPACT (-6.86%) | YAML (-21.14%) |
| YAML (+31.78%) | JSON_PRETTY (+72.84%) | XML_COMPACT (+2.58%) | JSON_PRETTY (+43.88%) | JSON_PRETTY (+43.27%) | XML_PRETTY (+49.79%) | TOON_DEFAULT (-4.70%) | JSON_PRETTY (-29.14%) | JSON_PRETTY (-14.73%) | XML_PRETTY (-35.92%) | JSON_COMPACT (-1.89%) | JSON_PRETTY (-25.94%) | JSON_PRETTY (-14.85%) | XML_PRETTY (-29.58%) |
| JSON_PRETTY (+34.89%) | XML_PRETTY (+95.00%) | JSON_PRETTY (+33.04%) | YAML (+50.05%) | YAML (+47.71%) | JSON_PRETTY (+55.04%) | XML_PRETTY (-5.37%) | XML_PRETTY (-41.26%) | YAML (-14.97%) | JSON_PRETTY (-36.65%) | XML_COMPACT (-1.98%) | XML_PRETTY (-34.13%) | YAML (-15.33%) | JSON_PRETTY (-32.50%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XML_COMPACT ≈ 64s | JSON_COMPACT ≈ 9788 | JSON_PRETTY ≈ 228 | XML_COMPACT ≈ 7011 | XML_COMPACT ≈ 7356 | JSON_COMPACT ≈ 18250 | YAML ≈ 77.42% | JSON_COMPACT ≈ 83 | XML_COMPACT ≈ 82 | JSON_COMPACT ≈ 83 | YAML ≈ 92.94% | JSON_COMPACT ≈ 94 | XML_COMPACT ≈ 94 | JSON_COMPACT ≈ 94 |
| XML_PRETTY (+21.07%) | XML_COMPACT (+26.36%) | JSON_COMPACT (+47.44%) | JSON_COMPACT (+15.90%) | JSON_COMPACT (+15.04%) | XML_COMPACT (+8.07%) | TOON_DEFAULT (-0.94%) | XML_COMPACT (-11.48%) | JSON_COMPACT (-4.29%) | XML_COMPACT (-6.84%) | JSON_COMPACT (-1.37%) | XML_COMPACT (-9.49%) | JSON_COMPACT (-4.41%) | XML_COMPACT (-5.39%) |
| JSON_PRETTY (+31.75%) | TOON_DEFAULT (+41.59%) | YAML (+47.58%) | XML_PRETTY (+33.45%) | JSON_PRETTY (+31.31%) | TOON_DEFAULT (+42.74%) | JSON_COMPACT (-2.42%) | YAML (-14.56%) | XML_PRETTY (-13.87%) | TOON_DEFAULT (-27.05%) | TOON_DEFAULT (-1.61%) | YAML (-13.60%) | XML_PRETTY (-11.11%) | TOON_DEFAULT (-25.10%) |
| JSON_COMPACT (+37.52%) | YAML (+43.57%) | XML_PRETTY (+51.10%) | JSON_PRETTY (+34.52%) | XML_PRETTY (+31.88%) | JSON_PRETTY (+45.52%) | XML_COMPACT (-4.30%) | TOON_DEFAULT (-14.56%) | JSON_PRETTY (-15.18%) | YAML (-29.56%) | XML_COMPACT (-2.33%) | TOON_DEFAULT (-14.07%) | JSON_PRETTY (-11.84%) | YAML (-26.84%) |
| YAML (+47.75%) | JSON_PRETTY (+72.65%) | XML_COMPACT (+51.24%) | TOON_DEFAULT (+68.86%) | TOON_DEFAULT (+65.74%) | YAML (+47.67%) | XML_PRETTY (-6.18%) | JSON_PRETTY (-32.03%) | TOON_DEFAULT (-22.72%) | JSON_PRETTY (-34.60%) | XML_PRETTY (-2.76%) | JSON_PRETTY (-26.18%) | TOON_DEFAULT (-21.77%) | JSON_PRETTY (-28.45%) |
| TOON_DEFAULT (+51.18%) | XML_PRETTY (+100.07%) | TOON_DEFAULT (+54.50%) | YAML (+79.14%) | YAML (+75.33%) | XML_PRETTY (+60.46%) | JSON_PRETTY (-8.07%) | XML_PRETTY (-40.89%) | YAML (-25.66%) | XML_PRETTY (-42.95%) | JSON_PRETTY (-4.06%) | XML_PRETTY (-34.43%) | YAML (-23.87%) | XML_PRETTY (-36.25%) |


#### 2.1.2 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 97.58% | JSON_PRETTY ≈ 58.03% | JSON_COMPACT ≈ 66.67% | JSON_COMPACT ≈ 71.43% |
| JSON_PRETTY (-1.21%) | YAML (-2.47%) | XML_COMPACT (0.00%) | TOON_DEFAULT (-5.56%) |
| TOON_DEFAULT (-6.36%) | XML_COMPACT (-2.47%) | JSON_PRETTY (-6.35%) | YAML (-11.11%) |
| XML_PRETTY (-9.09%) | XML_PRETTY (-2.47%) | YAML (-6.35%) | XML_PRETTY (-12.69%) |
| XML_COMPACT (-9.70%) | JSON_COMPACT (-4.94%) | TOON_DEFAULT (-7.14%) | XML_COMPACT (-15.87%) |
| JSON_COMPACT (-12.73%) | TOON_DEFAULT (-14.82%) | XML_PRETTY (-12.70%) | JSON_PRETTY (-22.22%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99.39% | XML_COMPACT ≈ 65.43% | TOON_DEFAULT ≈ 71.43% | YAML ≈ 55.56% |
| TOON_DEFAULT (-3.03%) | TOON_DEFAULT (-0.00%) | XML_PRETTY (-7.94%) | XML_COMPACT (-0.00%) |
| JSON_COMPACT (-5.45%) | JSON_COMPACT (-1.24%) | YAML (-7.94%) | JSON_COMPACT (-3.18%) |
| JSON_PRETTY (-9.09%) | YAML (-4.94%) | XML_COMPACT (-7.94%) | XML_PRETTY (-3.18%) |
| XML_PRETTY (-10.30%) | XML_PRETTY (-9.88%) | JSON_COMPACT (-9.52%) | JSON_PRETTY (-9.52%) |
| XML_COMPACT (-12.12%) | JSON_PRETTY (-14.82%) | JSON_PRETTY (-9.53%) | TOON_DEFAULT (-11.90%) |


#### 2.1.3 Category Accuracy By Character Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99.10% | XML_PRETTY ≈ 88.94% | JSON_COMPACT ≈ 94.07% | TOON_DEFAULT ≈ 86.69% |
| JSON_PRETTY (-0.39%) | YAML (-0.07%) | XML_COMPACT (-0.69%) | JSON_COMPACT (-2.30%) |
| TOON_DEFAULT (-2.04%) | JSON_COMPACT (-0.79%) | TOON_DEFAULT (-2.06%) | YAML (-3.57%) |
| XML_PRETTY (-2.57%) | XML_COMPACT (-0.95%) | YAML (-2.65%) | XML_PRETTY (-4.42%) |
| XML_COMPACT (-3.85%) | JSON_PRETTY (-1.27%) | JSON_PRETTY (-4.23%) | XML_COMPACT (-5.96%) |
| JSON_COMPACT (-5.40%) | TOON_DEFAULT (-2.22%) | XML_PRETTY (-5.50%) | JSON_PRETTY (-8.13%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99.92% | TOON_DEFAULT ≈ 90.06% | TOON_DEFAULT ≈ 93.62% | JSON_COMPACT ≈ 79.28% |
| TOON_DEFAULT (-1.33%) | YAML (-0.48%) | YAML (-0.55%) | YAML (-0.45%) |
| JSON_COMPACT (-1.90%) | XML_COMPACT (-0.99%) | JSON_COMPACT (-0.93%) | XML_PRETTY (-1.13%) |
| JSON_PRETTY (-2.77%) | XML_PRETTY (-1.64%) | XML_PRETTY (-2.25%) | XML_COMPACT (-2.10%) |
| XML_COMPACT (-3.15%) | JSON_COMPACT (-2.97%) | XML_COMPACT (-3.78%) | JSON_PRETTY (-5.70%) |
| XML_PRETTY (-4.74%) | JSON_PRETTY (-3.39%) | JSON_PRETTY (-8.23%) | TOON_DEFAULT (-7.61%) |


#### 2.1.4 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total | Accuracy By Character (%) | Useful Read Tokens (Acc By Char) | Wasted Read Tokens (Acc By Char) | Useful Output Tokens (Acc By Char) | Wasted Output Tokens (Acc By Char) | Eff Score Read (Acc By Char) | Eff Score Output (Acc By Char) | Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10315 | 8131 | 18446 | 2.213 | 62.866 | 72.58 | 7486.627 | 2828.373 | 5901.721 | 2229.612 | 79.99 | 78.33 | 81.10 | 90.98 | 9384.587 | 930.413 | 7397.887 | 733.446 | 92.26 | 90.59 | 93.36 |
| JSON_COMPACT | opt | 9788 | 8462 | 18250 | 2.186 | 65.535 | 75.00 | 7341.000 | 2447.000 | 6346.500 | 2115.500 | 83.30 | 78.51 | 83.30 | 91.57 | 8962.872 | 825.128 | 7748.653 | 713.347 | 94.35 | 89.56 | 94.35 |
| JSON_PRETTY | man | 17828 | 10771 | 28599 | 1.762 | 83.255 | 73.92 | 13178.458 | 4649.542 | 7961.677 | 2808.990 | 56.68 | 67.83 | 51.38 | 91.39 | 16293.009 | 1534.991 | 9843.313 | 927.354 | 68.32 | 79.47 | 63.02 |
| JSON_PRETTY | opt | 16899 | 9659 | 26558 | 1.752 | 76.059 | 69.35 | 11719.456 | 5179.544 | 6698.516 | 2960.484 | 56.62 | 69.58 | 54.48 | 88.88 | 15019.831 | 1879.169 | 8584.919 | 1074.081 | 69.64 | 82.60 | 67.50 |
| TOON_DEFAULT | man | 14096 | 8685 | 22781 | 1.851 | 67.281 | 71.10 | 10022.256 | 4073.744 | 6174.798 | 2509.868 | 66.82 | 74.95 | 67.04 | 92.19 | 12995.102 | 1100.898 | 8006.394 | 678.273 | 80.88 | 89.01 | 81.10 |
| TOON_DEFAULT | opt | 13859 | 12191 | 26050 | 1.860 | 95.478 | 76.48 | 10599.363 | 3259.637 | 9323.740 | 2867.343 | 71.17 | 63.40 | 60.77 | 91.33 | 12657.425 | 1201.575 | 11134.117 | 1056.967 | 81.07 | 73.30 | 70.67 |
| XML_COMPACT | man | 12848 | 8966 | 21814 | 2.522 | 69.530 | 71.78 | 9222.294 | 3625.706 | 6436.034 | 2530.299 | 71.30 | 74.19 | 70.41 | 90.89 | 11677.547 | 1170.453 | 8149.500 | 816.833 | 84.03 | 86.93 | 83.15 |
| XML_COMPACT | opt | 12368 | 7356 | 19724 | 2.517 | 56.543 | 73.12 | 9043.482 | 3324.518 | 5378.464 | 1977.203 | 73.73 | 82.04 | 77.61 | 90.61 | 11206.645 | 1161.355 | 6664.970 | 690.697 | 85.39 | 93.70 | 89.27 |
| XML_PRETTY | man | 20114 | 7518 | 27632 | 1.993 | 57.866 | 70.43 | 14166.290 | 5947.710 | 5294.693 | 2222.974 | 46.98 | 79.54 | 51.96 | 91.11 | 18325.865 | 1788.135 | 6849.346 | 668.321 | 60.77 | 93.33 | 65.75 |
| XML_PRETTY | opt | 19583 | 9701 | 29284 | 1.982 | 75.456 | 71.24 | 13950.929 | 5632.071 | 6910.636 | 2789.864 | 49.24 | 70.66 | 47.52 | 90.18 | 17659.949 | 1923.051 | 8747.911 | 952.589 | 61.86 | 83.29 | 60.15 |
| YAML | man | 14306 | 11104 | 25410 | 1.789 | 86.828 | 75.80 | 10843.948 | 3462.052 | 8417.084 | 2687.249 | 69.28 | 67.64 | 62.24 | 92.87 | 13285.982 | 1020.018 | 10312.594 | 791.739 | 80.66 | 79.02 | 73.62 |
| YAML | opt | 14053 | 12896 | 26949 | 1.799 | 101.293 | 77.42 | 10879.833 | 3173.167 | 9984.341 | 2911.992 | 71.17 | 60.98 | 58.68 | 92.94 | 13060.858 | 992.142 | 11985.852 | 910.481 | 81.52 | 71.33 | 69.03 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10315 | 9788 | -527 | -5.11 | 336 | 336 | 0 | 0.00 | 7795 | 8126 |  +331 |  +4.25 | 8131 | 8462 |  +331 |  +4.07 | 18446 | 18250 | -196 | -1.06 |
| JSON_PRETTY | 17828 | 16899 | -929 | -5.21 | 447 | 228 | -219 | -48.99 | 10324 | 9432 | -892 | -8.64 | 10771 | 9659 | -1112 | -10.32 | 28599 | 26558 | -2041 | -7.14 |
| TOON_DEFAULT | 14096 | 13859 | -237 | -1.68 | 342 | 352 |  +10 |  +2.92 | 8343 | 11840 |  +3497 |  +41.92 | 8685 | 12191 |  +3506 |  +40.37 | 22781 | 26050 |  +3269 |  +14.35 |
| XML_COMPACT | 12848 | 12368 | -480 | -3.74 | 345 | 345 | 0 | 0.00 | 8622 | 7012 | -1610 | -18.67 | 8966 | 7355 | -1611 | -17.97 | 21814 | 19723 | -2091 | -9.59 |
| XML_PRETTY | 20114 | 19583 | -531 | -2.64 | 342 | 344 |  +2 |  +0.58 | 7175 | 9356 |  +2181 |  +30.40 | 7518 | 9701 |  +2183 |  +29.04 | 27632 | 29284 |  +1652 |  +5.98 |
| YAML | 14306 | 14053 | -253 | -1.77 | 338 | 336 | -2 | -0.59 | 10767 | 12561 |  +1794 |  +16.66 | 11104 | 12896 |  +1792 |  +16.14 | 25410 | 26949 |  +1539 |  +6.06 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Before Write (ms) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) | Output (ms) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 24 | 429.792 | 0.77 | 31018 | 38410 | 0.203 | 309.76 | 38434 | 429.995 | 247.96 | 69429 |
| JSON_COMPACT | opt | 9 | 1087.556 | 0.29 | 48428 | 39495 | 0.206 | 318.51 | 39504 | 1087.762 | 254.87 | 87923 |
| JSON_PRETTY | man | 267 | 66.772 | 8.61 | 57029 | 36625 | 0.282 | 295.36 | 36892 | 67.054 | 238.01 | 93654 |
| JSON_PRETTY | opt | 272 | 62.129 | 8.77 | 47729 | 36507 | 0.258 | 294.41 | 36779 | 62.387 | 237.28 | 84236 |
| TOON_DEFAULT | man | 10 | 1409.600 | 0.32 | 35101 | 39156 | 0.218 | 315.77 | 39166 | 1409.818 | 252.68 | 74257 |
| TOON_DEFAULT | opt | 20 | 692.950 | 0.65 | 57218 | 39438 | 0.309 | 318.05 | 39458 | 693.260 | 254.57 | 96657 |
| XML_COMPACT | man | 11 | 1168.000 | 0.35 | 33261 | 41724 | 0.207 | 336.49 | 41735 | 1168.207 | 269.26 | 74985 |
| XML_COMPACT | opt | 9 | 1374.222 | 0.29 | 24657 | 39280 | 0.178 | 316.77 | 39289 | 1374.400 | 253.48 | 63937 |
| XML_PRETTY | man | 11 | 1828.545 | 0.35 | 23314 | 47627 | 0.151 | 384.09 | 47638 | 1828.696 | 307.34 | 70941 |
| XML_PRETTY | opt | 11 | 1780.273 | 0.35 | 42231 | 35179 | 0.266 | 283.70 | 35190 | 1780.539 | 227.03 | 77410 |
| YAML | man | 12 | 1192.167 | 0.39 | 54849 | 36643 | 0.294 | 295.51 | 36655 | 1192.461 | 236.48 | 91492 |
| YAML | opt | 8 | 1756.625 | 0.26 | 61846 | 32619 | 0.385 | 263.05 | 32627 | 1757.010 | 210.49 | 94465 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Before Write Man (s) | Output Before Write Opt (s) | Diff (s) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 24 | 9 | -15 | -62.50 | 31.02 | 48.43 |  +17.41 |  +56.13 | 38.41 | 39.50 |  +1.08 |  +2.82 | 38.43 | 39.50 |  +1.07 |  +2.78 | 69.43 | 87.92 |  +18.49 |  +26.64 |
| JSON_PRETTY | 267 | 272 |  +5 |  +1.87 | 57.03 | 47.73 | -9.30 | -16.31 | 36.63 | 36.51 | -0.12 | -0.32 | 36.89 | 36.78 | -0.11 | -0.31 | 93.65 | 84.24 | -9.42 | -10.06 |
| TOON_DEFAULT | 10 | 20 |  +10 |  +100.00 | 35.10 | 57.22 |  +22.12 |  +63.01 | 39.16 | 39.44 |  +0.28 |  +0.72 | 39.17 | 39.46 |  +0.29 |  +0.75 | 74.26 | 96.66 |  +22.40 |  +30.17 |
| XML_COMPACT | 11 | 9 | -2 | -18.18 | 33.26 | 24.66 | -8.60 | -25.87 | 41.72 | 39.28 | -2.44 | -5.86 | 41.74 | 39.29 | -2.45 | -5.86 | 74.99 | 63.94 | -11.05 | -14.73 |
| XML_PRETTY | 11 | 11 | 0 | 0.00 | 23.31 | 42.23 |  +18.92 |  +81.14 | 47.63 | 35.18 | -12.45 | -26.14 | 47.64 | 35.19 | -12.45 | -26.13 | 70.94 | 77.41 |  +6.47 |  +9.12 |
| YAML | 12 | 8 | -4 | -33.33 | 54.85 | 61.85 |  +7.00 |  +12.76 | 36.64 | 32.62 | -4.02 | -10.98 | 36.66 | 32.63 | -4.03 | -10.99 | 91.49 | 94.47 |  +2.97 |  +3.25 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token | Info / Read Token (Acc By Char) | Info / Output Token (Acc By Char) | Info / Total Token (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 2.213 | 15.125 | 332.742 | 0.704 | 0.893 | 0.393 | 0.882 | 1.119 | 0.493 |
| JSON_COMPACT | opt | 2.186 | 15.512 | 315.742 | 0.766 | 0.886 | 0.411 | 0.936 | 1.082 | 0.502 |
| JSON_PRETTY | man | 1.762 | 26.141 | 575.097 | 0.415 | 0.686 | 0.258 | 0.513 | 0.849 | 0.320 |
| JSON_PRETTY | opt | 1.752 | 26.781 | 545.129 | 0.410 | 0.718 | 0.261 | 0.526 | 0.920 | 0.335 |
| TOON_DEFAULT | man | 1.851 | 20.669 | 454.710 | 0.504 | 0.820 | 0.312 | 0.654 | 1.063 | 0.405 |
| TOON_DEFAULT | opt | 1.860 | 21.964 | 447.065 | 0.552 | 0.664 | 0.297 | 0.659 | 0.793 | 0.355 |
| XML_COMPACT | man | 2.522 | 18.839 | 414.452 | 0.559 | 0.801 | 0.329 | 0.707 | 1.014 | 0.417 |
| XML_COMPACT | opt | 2.517 | 19.601 | 398.968 | 0.591 | 0.994 | 0.371 | 0.733 | 1.232 | 0.459 |
| XML_PRETTY | man | 1.993 | 29.493 | 648.839 | 0.350 | 0.937 | 0.255 | 0.453 | 1.212 | 0.330 |
| XML_PRETTY | opt | 1.982 | 31.035 | 631.710 | 0.364 | 0.734 | 0.243 | 0.461 | 0.930 | 0.308 |
| YAML | man | 1.789 | 20.977 | 461.484 | 0.530 | 0.683 | 0.298 | 0.649 | 0.836 | 0.365 |
| YAML | opt | 1.799 | 22.271 | 453.323 | 0.551 | 0.600 | 0.287 | 0.661 | 0.721 | 0.345 |

#### 2.5.2 Characters And Values: Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 2.213 | 2.186 | -0.027 | -1.22 | 15.125 | 15.512 |  +0.387 |  +2.56 | 332.742 | 315.742 | -17.000 | -5.11 |
| JSON_PRETTY | 1.762 | 1.752 | -0.010 | -0.57 | 26.141 | 26.781 |  +0.640 |  +2.45 | 575.097 | 545.129 | -29.968 | -5.21 |
| TOON_DEFAULT | 1.851 | 1.860 |  +0.009 |  +0.49 | 20.669 | 21.964 |  +1.295 |  +6.27 | 454.710 | 447.065 | -7.645 | -1.68 |
| XML_COMPACT | 2.522 | 2.517 | -0.005 | -0.20 | 18.839 | 19.601 |  +0.762 |  +4.04 | 414.452 | 398.968 | -15.484 | -3.74 |
| XML_PRETTY | 1.993 | 1.982 | -0.011 | -0.55 | 29.493 | 31.035 |  +1.542 |  +5.23 | 648.839 | 631.710 | -17.129 | -2.64 |
| YAML | 1.789 | 1.799 |  +0.010 |  +0.56 | 20.977 | 22.271 |  +1.294 |  +6.17 | 461.484 | 453.323 | -8.161 | -1.77 |

#### 2.5.3 Information: Mandatory vs Optional
| Format | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 0.704 | 0.766 |  +0.062 |  +8.81 | 0.893 | 0.886 | -0.007 | -0.78 | 0.393 | 0.411 |  +0.018 |  +4.58 |
| JSON_PRETTY | 0.415 | 0.410 | -0.005 | -1.20 | 0.686 | 0.718 |  +0.032 |  +4.66 | 0.258 | 0.261 |  +0.003 |  +1.16 |
| TOON_DEFAULT | 0.504 | 0.552 |  +0.048 |  +9.52 | 0.820 | 0.664 | -0.156 | -18.97 | 0.312 | 0.297 | -0.015 | -4.81 |
| XML_COMPACT | 0.559 | 0.591 |  +0.032 |  +5.72 | 0.801 | 0.994 |  +0.193 |  +24.09 | 0.329 | 0.371 |  +0.042 |  +12.77 |
| XML_PRETTY | 0.350 | 0.364 |  +0.014 |  +4.00 | 0.937 | 0.734 | -0.203 | -21.66 | 0.255 | 0.243 | -0.012 | -4.71 |
| YAML | 0.530 | 0.551 |  +0.021 |  +3.96 | 0.683 | 0.600 | -0.083 | -12.15 | 0.298 | 0.287 | -0.011 | -3.69 |

#### 2.5.4 Information (Accuracy By Character): Mandatory vs Optional
| Format | Info / Read Token (Acc By Char) Man | Info / Read Token (Acc By Char) Opt | Diff | Diff (%) | Info / Output Token (Acc By Char) Man | Info / Output Token (Acc By Char)  Opt | Diff | Diff (%) | Info / Total Token (Acc By Char) Man | Info / Total Token (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 0.882 | 0.936 |  +0.054 |  +6.12 | 1.119 | 1.082 | -0.037 | -3.31 | 0.493 | 0.502 |  +0.009 |  +1.83 |
| JSON_PRETTY | 0.513 | 0.526 |  +0.013 |  +2.53 | 0.849 | 0.920 |  +0.071 |  +8.36 | 0.320 | 0.335 |  +0.015 |  +4.69 |
| TOON_DEFAULT | 0.654 | 0.659 |  +0.005 |  +0.76 | 1.063 | 0.793 | -0.270 | -25.36 | 0.405 | 0.355 | -0.050 | -12.35 |
| XML_COMPACT | 0.707 | 0.733 |  +0.026 |  +3.68 | 1.014 | 1.232 |  +0.218 |  +21.50 | 0.417 | 0.459 |  +0.042 |  +10.07 |
| XML_PRETTY | 0.453 | 0.461 |  +0.008 |  +1.77 | 1.212 | 0.930 | -0.282 | -23.27 | 0.330 | 0.308 | -0.022 | -6.67 |
| YAML | 0.649 | 0.661 |  +0.012 |  +1.85 | 0.836 | 0.721 | -0.115 | -13.76 | 0.365 | 0.345 | -0.020 | -5.48 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10315 | 7487 | 2828 | 8131 | 5902 | 2230 | 18446 | 13388 | 5058 | 72.58 | 79.99 | 78.33 | 81.10 | 70.12 | 78.35 | 76.69 | 79.46 |
| JSON_COMPACT | opt | 9788 | 7341 | 2447 | 8462 | 6347 | 2116 | 18250 | 13688 | 4563 | 75.00 | 83.30 | 78.51 | 83.30 | 73.39 | 82.23 | 77.44 | 82.23 |
| JSON_PRETTY | man | 17828 | 13178 | 4650 | 10771 | 7962 | 2809 | 28599 | 21140 | 7459 | 73.92 | 56.68 | 67.83 | 51.38 | 71.77 | 55.24 | 66.39 | 49.94 |
| JSON_PRETTY | opt | 16899 | 11719 | 5180 | 9659 | 6699 | 2960 | 26558 | 18418 | 8140 | 69.35 | 56.62 | 69.58 | 54.48 | 67.27 | 55.24 | 68.19 | 53.09 |
| TOON_DEFAULT | man | 14096 | 10022 | 4074 | 8685 | 6175 | 2510 | 22781 | 16197 | 6584 | 71.10 | 66.82 | 74.95 | 67.04 | 67.44 | 64.38 | 72.51 | 64.60 |
| TOON_DEFAULT | opt | 13859 | 10599 | 3260 | 12191 | 9324 | 2867 | 26050 | 19923 | 6127 | 76.48 | 71.17 | 63.40 | 60.77 | 75.56 | 70.56 | 62.79 | 60.15 |
| XML_COMPACT | man | 12848 | 9222 | 3626 | 8966 | 6436 | 2530 | 21814 | 15658 | 6156 | 71.78 | 71.30 | 74.19 | 70.41 | 69.99 | 70.10 | 73.00 | 69.21 |
| XML_COMPACT | opt | 12368 | 9043 | 3325 | 7356 | 5378 | 1977 | 19724 | 14422 | 5302 | 73.12 | 73.73 | 82.04 | 77.61 | 71.98 | 72.97 | 81.28 | 76.84 |
| XML_PRETTY | man | 20114 | 14166 | 5948 | 7518 | 5295 | 2223 | 27632 | 19461 | 8171 | 70.43 | 46.98 | 79.54 | 51.96 | 67.97 | 45.34 | 77.90 | 50.32 |
| XML_PRETTY | opt | 19583 | 13951 | 5632 | 9701 | 6911 | 2790 | 29284 | 20862 | 8422 | 71.24 | 49.24 | 70.66 | 47.52 | 69.39 | 48.00 | 69.43 | 46.29 |
| YAML | man | 14306 | 10844 | 3462 | 11104 | 8417 | 2687 | 25410 | 19261 | 6149 | 75.80 | 69.28 | 67.64 | 62.24 | 72.90 | 67.34 | 65.71 | 60.31 |
| YAML | opt | 14053 | 10880 | 3173 | 12896 | 9984 | 2912 | 26949 | 20864 | 6085 | 77.42 | 71.17 | 60.98 | 58.68 | 75.09 | 69.62 | 59.43 | 57.13 |

#### 2.6.2 Read Tokens: Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10315 | 9788 | -527 | -5.11 | 7487 | 7341 | -146 | -1.95 | 2828 | 2447 | -381 | -13.49 | 72.58 | 75.00 |  +2.42 |  +3.33 | 79.99 | 83.30 |  +3.31 |  +4.14 | 70.12 | 73.39 |  +3.27 |  +4.66 | 78.35 | 82.23 |  +3.88 |  +4.95 |
| JSON_PRETTY | 17828 | 16899 | -929 | -5.21 | 13178 | 11719 | -1459 | -11.07 | 4650 | 5180 |  +530 |  +11.40 | 73.92 | 69.35 | -4.57 | -6.18 | 56.68 | 56.62 | -0.05 | -0.10 | 71.77 | 67.27 | -4.50 | -6.27 | 55.24 | 55.24 | -0.01 | -0.01 |
| TOON_DEFAULT | 14096 | 13859 | -237 | -1.68 | 10022 | 10599 |  +577 |  +5.76 | 4074 | 3260 | -814 | -19.98 | 71.10 | 76.48 |  +5.38 |  +7.57 | 66.82 | 71.17 |  +4.35 |  +6.51 | 67.44 | 75.56 |  +8.12 |  +12.04 | 64.38 | 70.56 |  +6.18 |  +9.59 |
| XML_COMPACT | 12848 | 12368 | -480 | -3.74 | 9222 | 9043 | -179 | -1.94 | 3626 | 3325 | -301 | -8.31 | 71.78 | 73.12 |  +1.34 |  +1.87 | 71.30 | 73.73 |  +2.44 |  +3.42 | 69.99 | 71.98 |  +1.99 |  +2.84 | 70.10 | 72.97 |  +2.87 |  +4.10 |
| XML_PRETTY | 20114 | 19583 | -531 | -2.64 | 14166 | 13951 | -215 | -1.52 | 5948 | 5632 | -316 | -5.31 | 70.43 | 71.24 |  +0.81 |  +1.15 | 46.98 | 49.24 |  +2.25 |  +4.79 | 67.97 | 69.39 |  +1.42 |  +2.09 | 45.34 | 48.00 |  +2.66 |  +5.86 |
| YAML | 14306 | 14053 | -253 | -1.77 | 10844 | 10880 |  +36 |  +0.33 | 3462 | 3173 | -289 | -8.34 | 75.80 | 77.42 |  +1.62 |  +2.14 | 69.28 | 71.17 |  +1.90 |  +2.74 | 72.90 | 75.09 |  +2.19 |  +3.00 | 67.34 | 69.62 |  +2.28 |  +3.38 |

#### 2.6.3 Output Tokens: Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 8131 | 8462 |  +331 |  +4.07 | 5902 | 6347 |  +445 |  +7.54 | 2230 | 2116 | -114 | -5.12 | 72.58 | 75.00 |  +2.42 |  +3.33 | 78.33 | 78.51 |  +0.19 |  +0.24 | 70.12 | 73.39 |  +3.27 |  +4.66 | 76.69 | 77.44 |  +0.75 |  +0.98 |
| JSON_PRETTY | 10771 | 9659 | -1112 | -10.32 | 7962 | 6699 | -1263 | -15.86 | 2809 | 2960 |  +151 |  +5.39 | 73.92 | 69.35 | -4.57 | -6.18 | 67.83 | 69.58 |  +1.75 |  +2.58 | 71.77 | 67.27 | -4.50 | -6.27 | 66.39 | 68.19 |  +1.80 |  +2.71 |
| TOON_DEFAULT | 8685 | 12191 |  +3506 |  +40.37 | 6175 | 9324 |  +3149 |  +51.00 | 2510 | 2867 |  +357 |  +14.24 | 71.10 | 76.48 |  +5.38 |  +7.57 | 74.95 | 63.40 | -11.55 | -15.41 | 67.44 | 75.56 |  +8.12 |  +12.04 | 72.51 | 62.79 | -9.72 | -13.41 |
| XML_COMPACT | 8966 | 7355 | -1611 | -17.96 | 6436 | 5378 | -1058 | -16.43 | 2530 | 1977 | -553 | -21.86 | 71.78 | 73.12 |  +1.34 |  +1.87 | 74.19 | 82.04 |  +7.85 |  +10.58 | 69.99 | 71.98 |  +1.99 |  +2.84 | 73.00 | 81.28 |  +8.28 |  +11.34 |
| XML_PRETTY | 7518 | 9701 |  +2183 |  +29.03 | 5295 | 6911 |  +1616 |  +30.52 | 2223 | 2790 |  +567 |  +25.50 | 70.43 | 71.24 |  +0.81 |  +1.15 | 79.54 | 70.66 | -8.88 | -11.17 | 67.97 | 69.39 |  +1.42 |  +2.09 | 77.90 | 69.43 | -8.48 | -10.88 |
| YAML | 11104 | 12896 |  +1792 |  +16.14 | 8417 | 9984 |  +1567 |  +18.62 | 2687 | 2912 |  +225 |  +8.36 | 75.80 | 77.42 |  +1.62 |  +2.14 | 67.64 | 60.98 | -6.66 | -9.84 | 72.90 | 75.09 |  +2.19 |  +3.00 | 65.71 | 59.43 | -6.28 | -9.55 |

#### 2.6.4 Total Tokens: Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 18446 | 18250 | -196 | -1.06 | 13388 | 13687 |  +299 |  +2.23 | 5058 | 4563 | -495 | -9.80 | 72.58 | 75.00 |  +2.42 |  +3.33 | 81.10 | 83.30 |  +2.21 |  +2.72 | 70.12 | 73.39 |  +3.27 |  +4.66 | 79.46 | 82.23 |  +2.77 |  +3.49 |
| JSON_PRETTY | 28599 | 26558 | -2041 | -7.14 | 21140 | 18418 | -2722 | -12.88 | 7459 | 8140 |  +681 |  +9.14 | 73.92 | 69.35 | -4.57 | -6.18 | 51.38 | 54.48 |  +3.11 |  +6.05 | 71.77 | 67.27 | -4.50 | -6.27 | 49.94 | 53.09 |  +3.15 |  +6.31 |
| TOON_DEFAULT | 22781 | 26050 |  +3269 |  +14.35 | 16197 | 19923 |  +3726 |  +23.00 | 6584 | 6127 | -457 | -6.94 | 71.10 | 76.48 |  +5.38 |  +7.57 | 67.04 | 60.77 | -6.27 | -9.36 | 67.44 | 75.56 |  +8.12 |  +12.04 | 64.60 | 60.15 | -4.45 | -6.88 |
| XML_COMPACT | 21814 | 19723 | -2091 | -9.58 | 15658 | 14422 | -1236 | -7.90 | 6156 | 5302 | -854 | -13.88 | 71.78 | 73.12 |  +1.34 |  +1.87 | 70.41 | 77.61 |  +7.20 |  +10.22 | 69.99 | 71.98 |  +1.99 |  +2.84 | 69.21 | 76.84 |  +7.63 |  +11.03 |
| XML_PRETTY | 27632 | 29284 |  +1652 |  +5.98 | 19461 | 20862 |  +1401 |  +7.20 | 8171 | 8422 |  +251 |  +3.07 | 70.43 | 71.24 |  +0.81 |  +1.15 | 51.96 | 47.52 | -4.44 | -8.55 | 67.97 | 69.39 |  +1.42 |  +2.09 | 50.32 | 46.29 | -4.03 | -8.02 |
| YAML | 25410 | 26949 |  +1539 |  +6.06 | 19261 | 20864 |  +1603 |  +8.32 | 6149 | 6085 | -64 | -1.04 | 75.80 | 77.42 |  +1.62 |  +2.14 | 62.24 | 58.68 | -3.56 | -5.72 | 72.90 | 75.09 |  +2.19 |  +3.00 | 60.31 | 57.13 | -3.18 | -5.27 |

### 2.7 Token Utilization Efficiency (Accuracy by Character)
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy by Character (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy by Character (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10315 | 9385 | 930 | 8131 | 7398 | 733 | 18446 | 16782 | 1664 | 90.98 | 92.26 | 90.59 | 93.36 | 90.99 | 92.26 | 90.60 | 93.37 |
| JSON_COMPACT | opt | 9788 | 8963 | 825 | 8462 | 7749 | 713 | 18250 | 16712 | 1538 | 91.57 | 94.35 | 89.56 | 94.35 | 91.39 | 94.23 | 89.44 | 94.23 |
| JSON_PRETTY | man | 17828 | 16293 | 1535 | 10771 | 9843 | 927 | 28599 | 26136 | 2462 | 91.39 | 68.32 | 79.47 | 63.02 | 91.12 | 68.14 | 79.29 | 62.84 |
| JSON_PRETTY | opt | 16899 | 15020 | 1879 | 9659 | 8585 | 1074 | 26558 | 23605 | 2953 | 88.88 | 69.64 | 82.60 | 67.50 | 88.70 | 69.52 | 82.48 | 67.38 |
| TOON_DEFAULT | man | 14096 | 12995 | 1101 | 8685 | 8006 | 678 | 22781 | 21001 | 1779 | 92.19 | 80.88 | 89.01 | 81.10 | 91.69 | 80.55 | 88.68 | 80.77 |
| TOON_DEFAULT | opt | 13859 | 12657 | 1202 | 12191 | 11134 | 1057 | 26050 | 23792 | 2259 | 91.33 | 81.07 | 73.30 | 70.67 | 91.71 | 81.32 | 73.55 | 70.92 |
| XML_COMPACT | man | 12848 | 11678 | 1170 | 8966 | 8150 | 817 | 21814 | 19827 | 1987 | 90.89 | 84.03 | 86.93 | 83.15 | 90.92 | 84.06 | 86.95 | 83.17 |
| XML_COMPACT | opt | 12368 | 11207 | 1161 | 7356 | 6665 | 691 | 19724 | 17872 | 1852 | 90.61 | 85.39 | 93.70 | 89.27 | 90.64 | 85.42 | 93.72 | 89.28 |
| XML_PRETTY | man | 20114 | 18326 | 1788 | 7518 | 6849 | 668 | 27632 | 25175 | 2456 | 91.11 | 60.77 | 93.33 | 65.75 | 90.87 | 60.61 | 93.17 | 65.59 |
| XML_PRETTY | opt | 19583 | 17660 | 1923 | 9701 | 8748 | 953 | 29284 | 26408 | 2876 | 90.18 | 61.86 | 83.29 | 60.15 | 90.28 | 61.93 | 83.35 | 60.22 |
| YAML | man | 14306 | 13286 | 1020 | 11104 | 10313 | 792 | 25410 | 23599 | 1812 | 92.87 | 80.66 | 79.02 | 73.62 | 92.52 | 80.42 | 78.79 | 73.39 |
| YAML | opt | 14053 | 13061 | 992 | 12896 | 11986 | 910 | 26949 | 25047 | 1903 | 92.94 | 81.52 | 71.33 | 69.03 | 92.85 | 81.46 | 71.27 | 68.97 |

#### 2.7.2 Read Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10315 | 9788 | -527 | -5.11 | 9385 | 8963 | -422 | -4.49 | 930 | 825 | -105 | -11.32 | 90.98 | 91.57 |  +0.59 |  +0.65 | 92.26 | 94.35 |  +2.09 |  +2.27 | 90.99 | 91.39 |  +0.40 |  +0.44 | 92.26 | 94.23 |  +1.97 |  +2.13 |
| JSON_PRETTY | 17828 | 16899 | -929 | -5.21 | 16293 | 15020 | -1273 | -7.81 | 1535 | 1879 |  +344 |  +22.42 | 91.39 | 88.88 | -2.51 | -2.75 | 68.32 | 69.64 |  +1.32 |  +1.93 | 91.12 | 88.70 | -2.42 | -2.66 | 68.14 | 69.52 |  +1.38 |  +2.03 |
| TOON_DEFAULT | 14096 | 13859 | -237 | -1.68 | 12995 | 12657 | -338 | -2.60 | 1101 | 1202 |  +101 |  +9.14 | 92.19 | 91.33 | -0.86 | -0.93 | 80.88 | 81.07 |  +0.19 |  +0.23 | 91.69 | 91.71 |  +0.02 |  +0.02 | 80.55 | 81.32 |  +0.78 |  +0.96 |
| XML_COMPACT | 12848 | 12368 | -480 | -3.74 | 11678 | 11207 | -471 | -4.03 | 1170 | 1161 | -9 | -0.78 | 90.89 | 90.61 | -0.28 | -0.31 | 84.03 | 85.39 |  +1.36 |  +1.62 | 90.92 | 90.64 | -0.28 | -0.31 | 84.06 | 85.42 |  +1.36 |  +1.62 |
| XML_PRETTY | 20114 | 19583 | -531 | -2.64 | 18326 | 17660 | -666 | -3.63 | 1788 | 1923 |  +135 |  +7.55 | 91.11 | 90.18 | -0.93 | -1.02 | 60.77 | 61.86 |  +1.09 |  +1.79 | 90.87 | 90.28 | -0.59 | -0.65 | 60.61 | 61.93 |  +1.32 |  +2.17 |
| YAML | 14306 | 14053 | -253 | -1.77 | 13286 | 13061 | -225 | -1.69 | 1020 | 992 | -28 | -2.73 | 92.87 | 92.94 |  +0.07 |  +0.08 | 80.66 | 81.52 |  +0.86 |  +1.07 | 92.52 | 92.85 |  +0.33 |  +0.36 | 80.42 | 81.46 |  +1.03 |  +1.29 |

#### 2.7.3 Output Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 8131 | 8462 |  +331 |  +4.07 | 7398 | 7749 |  +351 |  +4.74 | 733 | 713 | -20 | -2.74 | 90.98 | 91.57 |  +0.59 |  +0.65 | 90.59 | 89.56 | -1.03 | -1.14 | 90.99 | 91.39 |  +0.40 |  +0.44 | 90.60 | 89.44 | -1.16 | -1.28 |
| JSON_PRETTY | 10771 | 9659 | -1112 | -10.32 | 9843 | 8585 | -1258 | -12.78 | 927 | 1074 |  +147 |  +15.83 | 91.39 | 88.88 | -2.51 | -2.75 | 79.47 | 82.60 |  +3.13 |  +3.93 | 91.12 | 88.70 | -2.42 | -2.66 | 79.29 | 82.48 |  +3.19 |  +4.02 |
| TOON_DEFAULT | 8685 | 12191 |  +3506 |  +40.37 | 8006 | 11134 |  +3128 |  +39.07 | 678 | 1057 |  +379 |  +55.85 | 92.19 | 91.33 | -0.86 | -0.93 | 89.01 | 73.30 | -15.71 | -17.65 | 91.69 | 91.71 |  +0.02 |  +0.02 | 88.68 | 73.55 | -15.12 | -17.06 |
| XML_COMPACT | 8966 | 7355 | -1611 | -17.96 | 8150 | 6665 | -1485 | -18.22 | 817 | 691 | -126 | -15.44 | 90.89 | 90.61 | -0.28 | -0.31 | 86.93 | 93.70 |  +6.77 |  +7.78 | 90.92 | 90.64 | -0.28 | -0.31 | 86.95 | 93.72 |  +6.77 |  +7.78 |
| XML_PRETTY | 7518 | 9701 |  +2183 |  +29.03 | 6849 | 8748 |  +1899 |  +27.72 | 668 | 952 |  +284 |  +42.56 | 91.11 | 90.18 | -0.93 | -1.02 | 93.33 | 83.29 | -10.04 | -10.76 | 90.87 | 90.28 | -0.59 | -0.65 | 93.17 | 83.35 | -9.82 | -10.54 |
| YAML | 11104 | 12896 |  +1792 |  +16.14 | 10313 | 11986 |  +1673 |  +16.22 | 792 | 911 |  +119 |  +14.99 | 92.87 | 92.94 |  +0.07 |  +0.08 | 79.02 | 71.33 | -7.69 | -9.73 | 92.52 | 92.85 |  +0.33 |  +0.36 | 78.79 | 71.27 | -7.52 | -9.54 |

#### 2.7.4 Total Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 18446 | 18250 | -196 | -1.06 | 16782 | 16711 | -71 | -0.42 | 1664 | 1539 | -125 | -7.54 | 90.98 | 91.57 |  +0.59 |  +0.65 | 93.36 | 94.35 |  +0.98 |  +1.06 | 90.99 | 91.39 |  +0.40 |  +0.44 | 93.37 | 94.23 |  +0.86 |  +0.92 |
| JSON_PRETTY | 28599 | 26558 | -2041 | -7.14 | 26136 | 23604 | -2532 | -9.69 | 2462 | 2953 |  +491 |  +19.94 | 91.39 | 88.88 | -2.51 | -2.75 | 63.02 | 67.50 |  +4.48 |  +7.11 | 91.12 | 88.70 | -2.42 | -2.66 | 62.84 | 67.38 |  +4.54 |  +7.23 |
| TOON_DEFAULT | 22781 | 26050 |  +3269 |  +14.35 | 21001 | 23791 |  +2790 |  +13.29 | 1779 | 2258 |  +479 |  +26.95 | 92.19 | 91.33 | -0.86 | -0.93 | 81.10 | 70.67 | -10.43 | -12.86 | 91.69 | 91.71 |  +0.02 |  +0.02 | 80.77 | 70.92 | -9.85 | -12.19 |
| XML_COMPACT | 21814 | 19723 | -2091 | -9.58 | 19827 | 17872 | -1955 | -9.86 | 1987 | 1852 | -135 | -6.81 | 90.89 | 90.61 | -0.28 | -0.31 | 83.15 | 89.27 |  +6.12 |  +7.36 | 90.92 | 90.64 | -0.28 | -0.31 | 83.17 | 89.28 |  +6.12 |  +7.36 |
| XML_PRETTY | 27632 | 29284 |  +1652 |  +5.98 | 25175 | 26408 |  +1233 |  +4.90 | 2456 | 2875 |  +419 |  +17.07 | 91.11 | 90.18 | -0.93 | -1.02 | 65.75 | 60.15 | -5.60 | -8.52 | 90.87 | 90.28 | -0.59 | -0.65 | 65.59 | 60.22 | -5.37 | -8.19 |
| YAML | 25410 | 26949 |  +1539 |  +6.06 | 23599 | 25047 |  +1448 |  +6.14 | 1812 | 1903 |  +91 |  +5.01 | 92.87 | 92.94 |  +0.07 |  +0.08 | 73.62 | 69.03 | -4.59 | -6.24 | 92.52 | 92.85 |  +0.33 |  +0.36 | 73.39 | 68.97 | -4.42 | -6.02 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Expected Characters | Output Characters | Correct Characters | Incorrect Characters | Accuracy by Character (%) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 90.00 | 34.00 | 0.00 | 72.58 | 7777.00 | 8113.33 | 7344.00 | 769.33 | 90.98 |
| JSON_COMPACT | opt | 93.00 | 31.00 | 0.00 | 75.00 | 8441.00 | 8718.33 | 8285.33 | 433.00 | 91.57 |
| JSON_PRETTY | man | 91.67 | 32.33 | 0.00 | 73.92 | 7777.00 | 7857.00 | 7590.00 | 267.00 | 91.39 |
| JSON_PRETTY | opt | 86.00 | 38.00 | 0.00 | 69.35 | 8441.00 | 8687.67 | 8168.67 | 519.00 | 88.88 |
| TOON_DEFAULT | man | 88.17 | 35.83 | 0.00 | 71.10 | 7777.00 | 7966.00 | 7520.67 | 445.33 | 92.19 |
| TOON_DEFAULT | opt | 94.83 | 29.17 | 0.00 | 76.48 | 8441.00 | 8584.50 | 8288.83 | 295.67 | 91.33 |
| XML_COMPACT | man | 89.00 | 35.00 | 0.00 | 71.78 | 7777.00 | 8009.67 | 7427.00 | 582.67 | 90.89 |
| XML_COMPACT | opt | 90.67 | 33.33 | 0.00 | 73.12 | 8441.00 | 8722.00 | 8230.00 | 492.00 | 90.61 |
| XML_PRETTY | man | 87.33 | 36.67 | 0.00 | 70.43 | 7777.00 | 7992.00 | 7525.00 | 467.00 | 91.11 |
| XML_PRETTY | opt | 88.33 | 35.67 | 0.00 | 71.24 | 8441.00 | 8721.33 | 8158.67 | 562.67 | 90.18 |
| YAML | man | 94.00 | 30.00 | 0.00 | 75.80 | 7777.00 | 7895.67 | 7593.00 | 302.67 | 92.87 |
| YAML | opt | 96.00 | 28.00 | 0.00 | 77.42 | 8441.00 | 8510.33 | 8342.00 | 168.33 | 92.94 |

#### 2.8.2 Answers: Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 90.00 | 93.00 |  +3 |  +3.33 | 34.00 | 31.00 | -3 | -8.82 | 0.00 | 0.00 | 0 | 0.00 | 72.58 | 75.00 |  +2.42 |
| JSON_PRETTY | 91.67 | 86.00 | -6 | -6.19 | 32.33 | 38.00 |  +6 |  +17.54 | 0.00 | 0.00 | 0 | 0.00 | 73.92 | 69.35 | -4.57 |
| TOON_DEFAULT | 88.17 | 94.83 |  +7 |  +7.55 | 35.83 | 29.17 | -7 | -18.59 | 0.00 | 0.00 | 0 | 0.00 | 71.10 | 76.48 |  +5.38 |
| XML_COMPACT | 89.00 | 90.67 |  +2 |  +1.88 | 35.00 | 33.33 | -2 | -4.77 | 0.00 | 0.00 | 0 | 0.00 | 71.78 | 73.12 |  +1.34 |
| XML_PRETTY | 87.33 | 88.33 |  +1 |  +1.15 | 36.67 | 35.67 | -1 | -2.73 | 0.00 | 0.00 | 0 | 0.00 | 70.43 | 71.24 |  +0.81 |
| YAML | 94.00 | 96.00 |  +2 |  +2.13 | 30.00 | 28.00 | -2 | -6.67 | 0.00 | 0.00 | 0 | 0.00 | 75.80 | 77.42 |  +1.62 |

#### 2.8.3 Characters: Mandatory vs Optional Data
| Format | Output Characters Man | Output Characters Opt | Diff | Diff (%) | Correct Characters Man | Correct Characters Opt | Diff | Diff (%) | Incorrect Characters Man | Incorrect Characters Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 8113.33 | 8718.33 |  +605 |  +7.46 | 7344.00 | 8285.33 |  +941 |  +12.82 | 769.33 | 433.00 | -336 | -43.72 | 90.98 | 91.57 |  +0.59 |
| JSON_PRETTY | 7857.00 | 8687.67 |  +831 |  +10.57 | 7590.00 | 8168.67 |  +579 |  +7.62 | 267.00 | 519.00 |  +252 |  +94.38 | 91.39 | 88.88 | -2.51 |
| TOON_DEFAULT | 7966.00 | 8584.50 |  +619 |  +7.76 | 7520.67 | 8288.83 |  +768 |  +10.21 | 445.33 | 295.67 | -150 | -33.61 | 92.19 | 91.33 | -0.86 |
| XML_COMPACT | 8009.67 | 8722.00 |  +712 |  +8.89 | 7427.00 | 8230.00 |  +803 |  +10.81 | 582.67 | 492.00 | -91 | -15.56 | 90.89 | 90.61 | -0.28 |
| XML_PRETTY | 7992.00 | 8721.33 |  +729 |  +9.13 | 7525.00 | 8158.67 |  +634 |  +8.42 | 467.00 | 562.67 |  +96 |  +20.49 | 91.11 | 90.18 | -0.93 |
| YAML | 7895.67 | 8510.33 |  +615 |  +7.78 | 7593.00 | 8342.00 |  +749 |  +9.86 | 302.67 | 168.33 | -134 | -44.38 | 92.87 | 92.94 |  +0.07 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 72.58 | 84.85 | 53.09 | 66.67 | 71.43 | 90.99 | 31.82 | 15.48 | 13.89 | 8.93 |
| JSON_COMPACT | opt | 75.00 | 93.94 | 64.20 | 61.91 | 52.38 | 91.39 | 35.23 | 18.72 | 12.90 | 6.55 |
| JSON_PRETTY | man | 73.92 | 96.36 | 58.03 | 60.31 | 49.20 | 91.12 | 36.14 | 16.92 | 12.57 | 6.15 |
| JSON_PRETTY | opt | 69.35 | 90.30 | 50.61 | 61.90 | 46.03 | 88.70 | 33.86 | 14.76 | 12.90 | 5.75 |
| TOON_DEFAULT | man | 71.10 | 91.21 | 43.21 | 59.52 | 65.87 | 91.69 | 34.20 | 12.60 | 12.40 | 8.23 |
| TOON_DEFAULT | opt | 76.48 | 96.36 | 65.43 | 71.43 | 43.65 | 91.71 | 36.14 | 19.08 | 14.88 | 5.46 |
| XML_COMPACT | man | 71.78 | 87.88 | 55.55 | 66.67 | 55.55 | 90.92 | 32.95 | 16.20 | 13.89 | 6.95 |
| XML_COMPACT | opt | 73.12 | 87.27 | 65.43 | 63.49 | 55.55 | 90.64 | 32.73 | 19.08 | 13.23 | 6.94 |
| XML_PRETTY | man | 70.43 | 88.48 | 55.55 | 53.97 | 58.73 | 90.87 | 33.18 | 16.20 | 11.24 | 7.34 |
| XML_PRETTY | opt | 71.24 | 89.09 | 55.56 | 63.49 | 52.38 | 90.28 | 33.41 | 16.20 | 13.23 | 6.55 |
| YAML | man | 75.80 | 97.58 | 55.56 | 60.31 | 60.32 | 92.52 | 36.59 | 16.20 | 12.57 | 7.54 |
| YAML | opt | 77.42 | 99.39 | 60.49 | 63.49 | 55.56 | 92.85 | 37.27 | 17.65 | 13.23 | 6.94 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 84.85 | 93.94 |  +9.09 | 31.82 | 35.23 |  +3.41 |
| JSON_PRETTY | 96.36 | 90.30 | -6.06 | 36.14 | 33.86 | -2.27 |
| TOON_DEFAULT | 91.21 | 96.36 |  +5.15 | 34.20 | 36.14 |  +1.93 |
| XML_COMPACT | 87.88 | 87.27 | -0.61 | 32.95 | 32.73 | -0.23 |
| XML_PRETTY | 88.48 | 89.09 |  +0.61 | 33.18 | 33.41 |  +0.23 |
| YAML | 97.58 | 99.39 |  +1.82 | 36.59 | 37.27 |  +0.68 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 53.09 | 64.20 |  +11.11 | 15.48 | 18.72 |  +3.24 |
| JSON_PRETTY | 58.03 | 50.61 | -7.41 | 16.92 | 14.76 | -2.16 |
| TOON_DEFAULT | 43.21 | 65.43 |  +22.22 | 12.60 | 19.08 |  +6.48 |
| XML_COMPACT | 55.55 | 65.43 |  +9.88 | 16.20 | 19.08 |  +2.88 |
| XML_PRETTY | 55.55 | 55.56 |  +0.00 | 16.20 | 16.20 | -0.00 |
| YAML | 55.56 | 60.49 |  +4.93 | 16.20 | 17.65 |  +1.45 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 66.67 | 61.91 | -4.76 | 13.89 | 12.90 | -0.99 |
| JSON_PRETTY | 60.31 | 61.90 |  +1.59 | 12.57 | 12.90 |  +0.33 |
| TOON_DEFAULT | 59.52 | 71.43 |  +11.91 | 12.40 | 14.88 |  +2.48 |
| XML_COMPACT | 66.67 | 63.49 | -3.18 | 13.89 | 13.23 | -0.66 |
| XML_PRETTY | 53.97 | 63.49 |  +9.52 | 11.24 | 13.23 |  +1.98 |
| YAML | 60.31 | 63.49 |  +3.18 | 12.57 | 13.23 |  +0.66 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 71.43 | 52.38 | -19.05 | 8.93 | 6.55 | -2.38 |
| JSON_PRETTY | 49.20 | 46.03 | -3.17 | 6.15 | 5.75 | -0.40 |
| TOON_DEFAULT | 65.87 | 43.65 | -22.22 | 8.23 | 5.46 | -2.78 |
| XML_COMPACT | 55.55 | 55.55 |  +0.00 | 6.95 | 6.94 | -0.00 |
| XML_PRETTY | 58.73 | 52.38 | -6.35 | 7.34 | 6.55 | -0.79 |
| YAML | 60.32 | 55.56 | -4.76 | 7.54 | 6.94 | -0.60 |

### 2.10 Accuracy By Character Per Question Category Analysis
#### 2.10.1 Metrics
| Format | Variant | Accuracy By Character (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc By Char (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 90.98 | 93.70 | 88.16 | 94.07 | 84.39 | 90.99 | 35.14 | 25.71 | 19.60 | 10.55 |
| JSON_COMPACT | opt | 91.57 | 98.02 | 87.10 | 92.70 | 79.28 | 91.39 | 36.76 | 25.41 | 19.31 | 9.91 |
| JSON_PRETTY | man | 91.39 | 98.71 | 87.68 | 89.84 | 78.55 | 91.12 | 37.02 | 25.57 | 18.71 | 9.82 |
| JSON_PRETTY | opt | 88.88 | 97.15 | 86.67 | 85.40 | 73.58 | 88.70 | 36.43 | 25.28 | 17.79 | 9.20 |
| TOON_DEFAULT | man | 92.19 | 97.05 | 86.73 | 92.01 | 86.69 | 91.69 | 36.39 | 25.29 | 19.17 | 10.84 |
| TOON_DEFAULT | opt | 91.33 | 98.59 | 90.06 | 93.62 | 71.67 | 91.71 | 36.97 | 26.27 | 19.51 | 8.96 |
| XML_COMPACT | man | 90.89 | 95.24 | 87.99 | 93.39 | 80.73 | 90.92 | 35.72 | 25.66 | 19.45 | 10.09 |
| XML_COMPACT | opt | 90.61 | 96.77 | 89.08 | 89.84 | 77.18 | 90.64 | 36.29 | 25.98 | 18.72 | 9.65 |
| XML_PRETTY | man | 91.11 | 96.52 | 88.94 | 88.57 | 82.27 | 90.87 | 36.20 | 25.94 | 18.45 | 10.28 |
| XML_PRETTY | opt | 90.18 | 95.17 | 88.43 | 91.38 | 78.15 | 90.28 | 35.69 | 25.79 | 19.04 | 9.77 |
| YAML | man | 92.87 | 99.10 | 88.87 | 91.43 | 83.11 | 92.52 | 37.16 | 25.92 | 19.05 | 10.39 |
| YAML | opt | 92.94 | 99.92 | 89.58 | 93.07 | 78.84 | 92.85 | 37.47 | 26.13 | 19.39 | 9.86 |

#### 2.10.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 93.70 | 98.02 |  +4.32 | 35.14 | 36.76 |  +1.62 |
| JSON_PRETTY | 98.71 | 97.15 | -1.56 | 37.02 | 36.43 | -0.59 |
| TOON_DEFAULT | 97.05 | 98.59 |  +1.54 | 36.39 | 36.97 |  +0.58 |
| XML_COMPACT | 95.24 | 96.77 |  +1.52 | 35.72 | 36.29 |  +0.57 |
| XML_PRETTY | 96.52 | 95.17 | -1.35 | 36.20 | 35.69 | -0.51 |
| YAML | 99.10 | 99.92 |  +0.82 | 37.16 | 37.47 |  +0.31 |

#### 2.10.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 88.16 | 87.10 | -1.06 | 25.71 | 25.41 | -0.30 |
| JSON_PRETTY | 87.68 | 86.67 | -1.00 | 25.57 | 25.28 | -0.29 |
| TOON_DEFAULT | 86.73 | 90.06 |  +3.33 | 25.29 | 26.27 |  +0.97 |
| XML_COMPACT | 87.99 | 89.08 |  +1.08 | 25.66 | 25.98 |  +0.32 |
| XML_PRETTY | 88.94 | 88.43 | -0.52 | 25.94 | 25.79 | -0.15 |
| YAML | 88.87 | 89.58 |  +0.71 | 25.92 | 26.13 |  +0.21 |

#### 2.10.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 94.07 | 92.70 | -1.38 | 19.60 | 19.31 | -0.28 |
| JSON_PRETTY | 89.84 | 85.40 | -4.44 | 18.71 | 17.79 | -0.92 |
| TOON_DEFAULT | 92.01 | 93.62 |  +1.61 | 19.17 | 19.51 |  +0.34 |
| XML_COMPACT | 93.39 | 89.84 | -3.55 | 19.45 | 18.72 | -0.74 |
| XML_PRETTY | 88.57 | 91.38 |  +2.81 | 18.45 | 19.04 |  +0.58 |
| YAML | 91.43 | 93.07 |  +1.64 | 19.05 | 19.39 |  +0.34 |

#### 2.10.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 84.39 | 79.28 | -5.10 | 10.55 | 9.91 | -0.64 |
| JSON_PRETTY | 78.55 | 73.58 | -4.97 | 9.82 | 9.20 | -0.62 |
| TOON_DEFAULT | 86.69 | 71.67 | -15.01 | 10.84 | 8.96 | -1.88 |
| XML_COMPACT | 80.73 | 77.18 | -3.55 | 10.09 | 9.65 | -0.44 |
| XML_PRETTY | 82.27 | 78.15 | -4.12 | 10.28 | 9.77 | -0.52 |
| YAML | 83.11 | 78.84 | -4.28 | 10.39 | 9.86 | -0.53 |

## 3. Appendices

### 3.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: on
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