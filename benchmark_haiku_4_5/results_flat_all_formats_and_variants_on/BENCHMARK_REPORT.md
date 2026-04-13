# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: on
- **Data Structure**: flat
- **Formats Tested**: 7 (CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: First iteration

## Executive Summary

This benchmark evaluates token efficiency and information accuracy across 7 file formats using Claude Haiku 4.5 (claude-haiku-4-5-20251001) as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context while a format that accurately conveys information may justify higher token cost.

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
- 7 formats tested: CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML
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
| XML_COMPACT ≈ 75s | CSV ≈ 6989 | JSON_COMPACT ≈ 228 | XML_COMPACT ≈ 9086 | XML_COMPACT ≈ 9429 | CSV ≈ 16525 | JSON_PRETTY ≈ 79.30% | TOON_DEFAULT ≈ 83 | XML_COMPACT ≈ 77 | TOON_DEFAULT ≈ 79 | JSON_PRETTY ≈ 93.58% | TOON_DEFAULT ≈ 94 | XML_COMPACT ≈ 89 | TOON_DEFAULT ≈ 90 |
| CSV (+3.89%) | TOON_DEFAULT (+0.84%) | YAML (+1.46%) | CSV (+1.33%) | CSV (+1.14%) | TOON_DEFAULT (+13.27%) | TOON_DEFAULT (-2.55%) | CSV (-11.70%) | YAML (-8.93%) | CSV (-5.18%) | TOON_DEFAULT (-0.36%) | CSV (-6.13%) | CSV (-6.78%) | CSV (-0.11%) |
| YAML (+17.18%) | JSON_COMPACT (+32.61%) | TOON_DEFAULT (+26.06%) | YAML (+13.96%) | YAML (+12.26%) | XML_COMPACT (+27.81%) | XML_PRETTY (-4.30%) | JSON_COMPACT (-13.36%) | CSV (-11.53%) | XML_COMPACT (-10.35%) | XML_COMPACT (-1.18%) | JSON_COMPACT (-9.67%) | YAML (-7.36%) | XML_COMPACT (-7.80%) |
| TOON_DEFAULT (+27.39%) | XML_COMPACT (+67.31%) | CSV (+44.66%) | TOON_DEFAULT (+25.29%) | TOON_DEFAULT (+23.78%) | JSON_COMPACT (+35.72%) | XML_COMPACT (-5.10%) | XML_COMPACT (-21.65%) | TOON_DEFAULT (-13.74%) | JSON_COMPACT (-16.87%) | XML_PRETTY (-1.22%) | XML_COMPACT (-17.90%) | TOON_DEFAULT (-13.16%) | JSON_COMPACT (-12.55%) |
| XML_PRETTY (+36.22%) | YAML (+79.63%) | JSON_PRETTY (+49.78%) | XML_PRETTY (+36.89%) | XML_PRETTY (+35.55%) | YAML (+40.02%) | YAML (-5.91%) | YAML (-25.93%) | JSON_PRETTY (-21.96%) | YAML (-17.92%) | YAML (-1.52%) | YAML (-21.36%) | XML_PRETTY (-20.62%) | YAML (-14.08%) |
| JSON_PRETTY (+40.74%) | JSON_PRETTY (+104.36%) | XML_COMPACT (+50.51%) | JSON_PRETTY (+40.85%) | JSON_PRETTY (+39.34%) | JSON_PRETTY (+65.94%) | JSON_COMPACT (-7.53%) | JSON_PRETTY (-28.50%) | XML_PRETTY (-23.16%) | JSON_PRETTY (-27.50%) | JSON_COMPACT (-2.32%) | JSON_PRETTY (-26.73%) | JSON_PRETTY (-21.91%) | JSON_PRETTY (-25.76%) |
| JSON_COMPACT (+45.75%) | XML_PRETTY (+131.31%) | XML_PRETTY (+50.51%) | JSON_COMPACT (+42.34%) | JSON_COMPACT (+39.58%) | XML_PRETTY (+75.16%) | CSV (-17.47%) | XML_PRETTY (-39.89%) | JSON_COMPACT (-28.68%) | XML_PRETTY (-36.35%) | CSV (-9.33%) | XML_PRETTY (-34.61%) | JSON_COMPACT (-23.79%) | XML_PRETTY (-31.23%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 73s | CSV ≈ 6700 | CSV ≈ 231 | JSON_PRETTY ≈ 7966 | JSON_PRETTY ≈ 8302 | JSON_COMPACT ≈ 18753 | XML_PRETTY ≈ 77.96% | JSON_COMPACT ≈ 77 | JSON_PRETTY ≈ 83 | JSON_COMPACT ≈ 78 | XML_PRETTY ≈ 92.40% | CSV ≈ 91 | JSON_PRETTY ≈ 94 | JSON_COMPACT ≈ 88 |
| XML_PRETTY (+7.93%) | JSON_COMPACT (+30.57%) | XML_PRETTY (+0.58%) | JSON_COMPACT (+21.36%) | JSON_COMPACT (+20.52%) | CSV (+4.22%) | TOON_DEFAULT (-0.41%) | CSV (-4.61%) | XML_PRETTY (-8.54%) | JSON_PRETTY (-11.59%) | XML_COMPACT (-0.21%) | JSON_COMPACT (-4.09%) | XML_PRETTY (-8.98%) | CSV (-6.36%) |
| JSON_COMPACT (+13.19%) | XML_COMPACT (+63.13%) | YAML (+44.30%) | XML_PRETTY (+22.97%) | XML_PRETTY (+20.79%) | JSON_PRETTY (+15.55%) | XML_COMPACT (-0.54%) | XML_COMPACT (-9.02%) | JSON_COMPACT (-9.69%) | XML_COMPACT (-11.67%) | TOON_DEFAULT (-0.86%) | XML_COMPACT (-11.86%) | JSON_COMPACT (-9.66%) | JSON_PRETTY (-9.10%) |
| YAML (+21.03%) | TOON_DEFAULT (+72.55%) | XML_COMPACT (+45.17%) | YAML (+27.74%) | YAML (+26.59%) | YAML (+18.81%) | JSON_COMPACT (-1.62%) | TOON_DEFAULT (-11.79%) | YAML (-13.22%) | YAML (-12.31%) | JSON_COMPACT (-1.15%) | TOON_DEFAULT (-14.78%) | YAML (-12.60%) | XML_COMPACT (-10.46%) |
| TOON_DEFAULT (+28.94%) | YAML (+75.69%) | JSON_PRETTY (+45.45%) | XML_COMPACT (+40.13%) | XML_COMPACT (+38.50%) | XML_COMPACT (+19.60%) | YAML (-1.89%) | YAML (-14.03%) | XML_COMPACT (-18.64%) | TOON_DEFAULT (-14.64%) | YAML (-1.17%) | YAML (-15.83%) | XML_COMPACT (-17.66%) | YAML (-10.73%) |
| XML_COMPACT (+30.13%) | JSON_PRETTY (+99.51%) | JSON_COMPACT (+46.18%) | TOON_DEFAULT (+43.45%) | TOON_DEFAULT (+41.74%) | TOON_DEFAULT (+24.40%) | JSON_PRETTY (-3.50%) | JSON_PRETTY (-22.71%) | TOON_DEFAULT (-20.30%) | CSV (-16.45%) | JSON_PRETTY (-1.47%) | JSON_PRETTY (-22.23%) | TOON_DEFAULT (-19.69%) | TOON_DEFAULT (-13.68%) |
| CSV (+50.08%) | XML_PRETTY (+125.21%) | TOON_DEFAULT (+47.26%) | CSV (+58.34%) | CSV (+54.72%) | XML_PRETTY (+33.94%) | CSV (-17.74%) | XML_PRETTY (-27.54%) | CSV (-41.32%) | XML_PRETTY (-20.42%) | CSV (-6.38%) | XML_PRETTY (-27.83%) | CSV (-29.86%) | XML_PRETTY (-18.47%) |


#### 2.1.2 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 98.79% | JSON_PRETTY ≈ 66.67% | YAML ≈ 73.02% | TOON_DEFAULT ≈ 66.67% |
| YAML (0.00%) | XML_PRETTY (-4.94%) | XML_PRETTY (-4.76%) | JSON_PRETTY (-7.94%) |
| JSON_COMPACT (-1.21%) | XML_COMPACT (-4.94%) | TOON_DEFAULT (-7.94%) | XML_COMPACT (-12.70%) |
| XML_PRETTY (-1.21%) | TOON_DEFAULT (-9.26%) | JSON_PRETTY (-7.94%) | CSV (-19.05%) |
| TOON_DEFAULT (-4.24%) | CSV (-11.11%) | XML_COMPACT (-9.53%) | XML_PRETTY (-26.99%) |
| XML_COMPACT (-6.67%) | JSON_COMPACT (-11.11%) | JSON_COMPACT (-14.29%) | YAML (-26.99%) |
| CSV (-24.85%) | YAML (-18.52%) | CSV (-20.63%) | JSON_COMPACT (-28.57%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| XML_COMPACT ≈ 98.79% | JSON_COMPACT ≈ 74.07% | JSON_COMPACT ≈ 71.43% | XML_PRETTY ≈ 61.90% |
| XML_PRETTY (-0.61%) | TOON_DEFAULT (-1.85%) | TOON_DEFAULT (-0.00%) | JSON_COMPACT (-14.28%) |
| YAML (-1.21%) | YAML (-4.94%) | YAML (-1.59%) | TOON_DEFAULT (-14.29%) |
| JSON_PRETTY (-3.64%) | JSON_PRETTY (-7.41%) | XML_COMPACT (-4.76%) | XML_COMPACT (-15.87%) |
| TOON_DEFAULT (-4.85%) | XML_COMPACT (-7.41%) | XML_PRETTY (-6.35%) | JSON_PRETTY (-19.05%) |
| JSON_COMPACT (-8.48%) | XML_PRETTY (-14.81%) | CSV (-7.94%) | CSV (-26.98%) |
| CSV (-23.63%) | CSV (-27.16%) | JSON_PRETTY (-9.53%) | YAML (-26.98%) |


#### 2.1.3 Category Accuracy By Character Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99.71% | JSON_PRETTY ≈ 90.50% | YAML ≈ 94.55% | TOON_DEFAULT ≈ 84.40% |
| JSON_PRETTY (-0.15%) | XML_COMPACT (-0.52%) | XML_PRETTY (-0.85%) | JSON_PRETTY (-0.55%) |
| JSON_COMPACT (-0.74%) | TOON_DEFAULT (-0.67%) | TOON_DEFAULT (-1.03%) | XML_COMPACT (-3.41%) |
| XML_PRETTY (-0.75%) | XML_PRETTY (-0.72%) | XML_COMPACT (-1.75%) | XML_PRETTY (-7.31%) |
| TOON_DEFAULT (-1.56%) | JSON_COMPACT (-1.07%) | JSON_PRETTY (-2.91%) | JSON_COMPACT (-8.55%) |
| XML_COMPACT (-1.91%) | YAML (-2.34%) | CSV (-3.76%) | CSV (-9.28%) |
| CSV (-16.15%) | CSV (-2.79%) | JSON_COMPACT (-5.71%) | YAML (-9.84%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| XML_COMPACT ≈ 99.83% | JSON_PRETTY ≈ 91.60% | TOON_DEFAULT ≈ 93.63% | XML_PRETTY ≈ 79.28% |
| YAML (-0.23%) | TOON_DEFAULT (-0.28%) | JSON_COMPACT (-0.08%) | TOON_DEFAULT (-5.22%) |
| XML_PRETTY (-0.29%) | XML_COMPACT (-0.59%) | XML_COMPACT (-1.62%) | XML_COMPACT (-5.42%) |
| JSON_PRETTY (-0.99%) | CSV (-0.62%) | YAML (-1.88%) | JSON_COMPACT (-5.42%) |
| JSON_COMPACT (-2.20%) | YAML (-1.09%) | CSV (-2.04%) | JSON_PRETTY (-7.81%) |
| TOON_DEFAULT (-2.32%) | JSON_COMPACT (-1.62%) | XML_PRETTY (-2.73%) | YAML (-9.58%) |
| CSV (-11.84%) | XML_PRETTY (-2.37%) | JSON_PRETTY (-4.84%) | CSV (-10.38%) |


#### 2.1.4 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total | Accuracy By Character (%) | Useful Read Tokens (Acc By Char) | Wasted Read Tokens (Acc By Char) | Useful Output Tokens (Acc By Char) | Wasted Output Tokens (Acc By Char) | Eff Score Read (Acc By Char) | Eff Score Output (Acc By Char) | Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6989 | 9536 | 16525 | 1.444 | 74.250 | 61.83 | 4321.299 | 2667.701 | 5896.315 | 3640.018 | 73.50 | 67.77 | 74.53 | 84.25 | 5888.233 | 1100.768 | 8034.361 | 1501.972 | 88.45 | 82.72 | 89.47 |
| CSV | opt | 6700 | 12845 | 19545 | 1.429 | 101.723 | 60.22 | 4034.740 | 2665.260 | 7735.058 | 5109.609 | 73.44 | 48.66 | 65.36 | 86.02 | 5763.340 | 936.660 | 11048.983 | 1795.684 | 90.64 | 65.86 | 82.56 |
| JSON_COMPACT | man | 9268 | 13161 | 22429 | 2.149 | 104.298 | 71.77 | 6651.644 | 2616.356 | 9445.411 | 3715.256 | 72.12 | 54.64 | 65.34 | 91.26 | 8457.977 | 810.023 | 12010.425 | 1150.242 | 85.11 | 67.63 | 78.33 |
| JSON_COMPACT | opt | 8748 | 10005 | 18753 | 2.113 | 77.965 | 76.34 | 6678.223 | 2069.777 | 7638.071 | 2367.262 | 76.99 | 74.89 | 78.23 | 91.25 | 7982.550 | 765.450 | 9129.866 | 875.467 | 86.93 | 84.83 | 88.17 |
| JSON_PRETTY | man | 14283 | 13138 | 27421 | 1.694 | 103.204 | 79.30 | 11326.419 | 2956.581 | 10418.698 | 2719.635 | 59.52 | 59.78 | 56.98 | 93.58 | 13366.031 | 916.969 | 12294.852 | 843.481 | 69.04 | 69.30 | 66.50 |
| JSON_PRETTY | opt | 13367 | 8302 | 21669 | 1.680 | 64.242 | 74.46 | 9953.068 | 3413.932 | 6181.669 | 2120.331 | 59.51 | 82.92 | 69.16 | 90.93 | 12154.613 | 1212.387 | 7549.009 | 752.991 | 70.49 | 93.90 | 80.14 |
| TOON_DEFAULT | man | 7048 | 11671 | 18719 | 1.442 | 91.805 | 76.75 | 5409.340 | 1638.660 | 8957.364 | 2713.469 | 83.24 | 66.08 | 78.60 | 93.22 | 6570.146 | 477.854 | 10879.551 | 791.283 | 94.22 | 77.06 | 89.58 |
| TOON_DEFAULT | opt | 11561 | 11768 | 23329 | 1.698 | 92.155 | 77.55 | 8965.556 | 2595.445 | 9125.697 | 2641.804 | 67.92 | 66.08 | 66.78 | 91.54 | 10582.939 | 978.061 | 10771.969 | 995.531 | 77.24 | 75.41 | 76.10 |
| XML_COMPACT | man | 11693 | 9429 | 21122 | 2.366 | 73.274 | 74.20 | 8676.206 | 3016.794 | 6996.071 | 2432.596 | 65.22 | 76.60 | 70.46 | 92.40 | 10804.332 | 888.668 | 8712.088 | 716.579 | 77.35 | 88.73 | 82.59 |
| XML_COMPACT | opt | 10930 | 11498 | 22428 | 2.340 | 90.024 | 77.42 | 8462.006 | 2467.994 | 8902.009 | 2596.324 | 70.05 | 67.47 | 69.10 | 92.19 | 10076.367 | 853.633 | 10600.313 | 898.020 | 79.89 | 77.31 | 78.95 |
| XML_PRETTY | man | 16166 | 12780 | 28946 | 1.934 | 100.304 | 75.00 | 12124.500 | 4041.500 | 9585.250 | 3195.083 | 50.03 | 58.86 | 50.03 | 92.36 | 14930.918 | 1235.082 | 11803.916 | 976.417 | 61.61 | 70.44 | 61.60 |
| XML_PRETTY | opt | 15089 | 10028 | 25117 | 1.917 | 79.000 | 77.96 | 11763.384 | 3325.616 | 7818.088 | 2210.245 | 55.79 | 75.84 | 62.26 | 92.40 | 13942.236 | 1146.764 | 9266.180 | 762.153 | 65.42 | 85.47 | 71.89 |
| YAML | man | 12554 | 10585 | 23139 | 1.664 | 83.500 | 73.39 | 9213.381 | 3340.619 | 7768.332 | 2816.669 | 61.65 | 69.76 | 64.51 | 92.06 | 11557.212 | 996.788 | 9744.551 | 840.449 | 74.10 | 82.20 | 76.96 |
| YAML | opt | 11771 | 10509 | 22280 | 1.649 | 82.065 | 76.07 | 8954.200 | 2816.800 | 7994.450 | 2514.883 | 66.19 | 71.96 | 68.60 | 91.23 | 10738.683 | 1032.317 | 9587.664 | 921.669 | 76.30 | 82.06 | 78.71 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6989 | 6700 | -289 | -4.14 | 329 | 231 | -98 | -29.79 | 9207 | 12614 |  +3407 |  +37.00 | 9536 | 12844 |  +3308 |  +34.69 | 16525 | 19544 |  +3019 |  +18.27 |
| JSON_COMPACT | 9268 | 8748 | -520 | -5.61 | 228 | 338 |  +110 |  +48.25 | 12933 | 9668 | -3265 | -25.25 | 13161 | 10006 | -3155 | -23.97 | 22429 | 18754 | -3675 | -16.39 |
| JSON_PRETTY | 14283 | 13367 | -916 | -6.41 | 341 | 336 | -5 | -1.47 | 12797 | 7966 | -4831 | -37.75 | 13138 | 8302 | -4836 | -36.81 | 27421 | 21669 | -5752 | -20.98 |
| TOON_DEFAULT | 7048 | 11561 |  +4513 |  +64.03 | 287 | 340 |  +53 |  +18.47 | 11384 | 11428 |  +44 |  +0.39 | 11671 | 11768 |  +97 |  +0.83 | 18719 | 23329 |  +4610 |  +24.63 |
| XML_COMPACT | 11693 | 10930 | -763 | -6.53 | 343 | 336 | -7 | -2.04 | 9086 | 11163 |  +2077 |  +22.86 | 9429 | 11499 |  +2070 |  +21.95 | 21122 | 22429 |  +1307 |  +6.19 |
| XML_PRETTY | 16166 | 15089 | -1077 | -6.66 | 343 | 233 | -110 | -32.07 | 12438 | 9796 | -2642 | -21.24 | 12780 | 10028 | -2752 | -21.53 | 28946 | 25117 | -3829 | -13.23 |
| YAML | 12554 | 11771 | -783 | -6.24 | 231 | 333 |  +102 |  +44.16 | 10354 | 10176 | -178 | -1.72 | 10585 | 10509 | -76 | -0.72 | 23139 | 22280 | -859 | -3.71 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Before Write (ms) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) | Output (ms) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 20 | 349.450 | 0.65 | 40642 | 37620 | 0.245 | 303.39 | 37640 | 349.695 | 242.84 | 78262 |
| CSV | opt | 15 | 446.667 | 0.48 | 68590 | 41210 | 0.306 | 332.34 | 41225 | 446.973 | 265.97 | 109800 |
| JSON_COMPACT | man | 28 | 331.000 | 0.90 | 69679 | 40123 | 0.322 | 323.57 | 40151 | 331.322 | 259.04 | 109802 |
| JSON_COMPACT | opt | 10 | 874.800 | 0.32 | 41998 | 40816 | 0.237 | 329.16 | 40826 | 875.037 | 263.39 | 82813 |
| JSON_PRETTY | man | 25 | 571.320 | 0.81 | 68007 | 38019 | 0.337 | 306.60 | 38044 | 571.657 | 245.45 | 106026 |
| JSON_PRETTY | opt | 13 | 1028.231 | 0.42 | 32554 | 40608 | 0.196 | 327.48 | 40621 | 1028.427 | 262.07 | 73162 |
| TOON_DEFAULT | man | 3 | 2349.333 | 0.10 | 58363 | 37603 | 0.302 | 303.25 | 37606 | 2349.635 | 242.62 | 95966 |
| TOON_DEFAULT | opt | 5 | 2312.200 | 0.16 | 55237 | 39099 | 0.304 | 315.31 | 39104 | 2312.505 | 252.28 | 94335 |
| XML_COMPACT | man | 4 | 2923.250 | 0.13 | 38153 | 37181 | 0.244 | 299.84 | 37185 | 2923.494 | 239.90 | 75334 |
| XML_COMPACT | opt | 9 | 1214.444 | 0.29 | 57835 | 37367 | 0.299 | 301.34 | 37376 | 1214.743 | 241.13 | 95202 |
| XML_PRETTY | man | 11 | 1469.636 | 0.35 | 63055 | 39564 | 0.314 | 319.06 | 39575 | 1469.950 | 255.32 | 102619 |
| XML_PRETTY | opt | 17 | 887.588 | 0.55 | 46382 | 32583 | 0.301 | 262.77 | 32600 | 887.889 | 210.32 | 78965 |
| YAML | man | 8 | 1569.250 | 0.26 | 48750 | 39528 | 0.262 | 318.78 | 39536 | 1569.512 | 255.07 | 88278 |
| YAML | opt | 11 | 1070.091 | 0.35 | 49284 | 39265 | 0.259 | 316.65 | 39276 | 1070.350 | 253.39 | 88549 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Before Write Man (s) | Output Before Write Opt (s) | Diff (s) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 20 | 15 | -5 | -25.00 | 40.64 | 68.59 |  +27.95 |  +68.77 | 37.62 | 41.21 |  +3.59 |  +9.54 | 37.64 | 41.23 |  +3.58 |  +9.52 | 78.26 | 109.80 |  +31.54 |  +40.30 |
| JSON_COMPACT | 28 | 10 | -18 | -64.29 | 69.68 | 42.00 | -27.68 | -39.73 | 40.12 | 40.82 |  +0.69 |  +1.73 | 40.15 | 40.83 |  +0.67 |  +1.68 | 109.80 | 82.81 | -26.99 | -24.58 |
| JSON_PRETTY | 25 | 13 | -12 | -48.00 | 68.01 | 32.55 | -35.45 | -52.13 | 38.02 | 40.61 |  +2.59 |  +6.81 | 38.04 | 40.62 |  +2.58 |  +6.77 | 106.03 | 73.16 | -32.86 | -31.00 |
| TOON_DEFAULT | 3 | 5 |  +2 |  +66.67 | 58.36 | 55.24 | -3.13 | -5.36 | 37.60 | 39.10 |  +1.50 |  +3.98 | 37.61 | 39.10 |  +1.50 |  +3.98 | 95.97 | 94.34 | -1.63 | -1.70 |
| XML_COMPACT | 4 | 9 |  +5 |  +125.00 | 38.15 | 57.84 |  +19.68 |  +51.59 | 37.18 | 37.37 |  +0.19 |  +0.50 | 37.18 | 37.38 |  +0.19 |  +0.51 | 75.33 | 95.20 |  +19.87 |  +26.37 |
| XML_PRETTY | 11 | 17 |  +6 |  +54.55 | 63.05 | 46.38 | -16.67 | -26.44 | 39.56 | 32.58 | -6.98 | -17.64 | 39.58 | 32.60 | -6.97 | -17.62 | 102.62 | 78.97 | -23.65 | -23.05 |
| YAML | 8 | 11 |  +3 |  +37.50 | 48.75 | 49.28 |  +0.53 |  +1.10 | 39.53 | 39.26 | -0.26 | -0.67 | 39.54 | 39.28 | -0.26 | -0.66 | 88.28 | 88.55 |  +0.27 |  +0.31 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token | Info / Read Token (Acc By Char) | Info / Output Token (Acc By Char) | Info / Total Token (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 1.444 | 10.248 | 225.452 | 0.885 | 0.648 | 0.374 | 1.205 | 0.883 | 0.510 |
| CSV | opt | 1.429 | 10.618 | 216.129 | 0.899 | 0.469 | 0.308 | 1.284 | 0.670 | 0.440 |
| JSON_COMPACT | man | 2.149 | 13.589 | 298.968 | 0.774 | 0.545 | 0.320 | 0.985 | 0.693 | 0.407 |
| JSON_COMPACT | opt | 2.113 | 13.864 | 282.194 | 0.873 | 0.763 | 0.407 | 1.043 | 0.912 | 0.487 |
| JSON_PRETTY | man | 1.694 | 20.943 | 460.742 | 0.555 | 0.604 | 0.289 | 0.655 | 0.712 | 0.341 |
| JSON_PRETTY | opt | 1.680 | 21.184 | 431.194 | 0.557 | 0.897 | 0.344 | 0.680 | 1.095 | 0.420 |
| TOON_DEFAULT | man | 1.442 | 10.334 | 227.355 | 1.089 | 0.676 | 0.414 | 1.323 | 0.821 | 0.504 |
| TOON_DEFAULT | opt | 1.698 | 18.322 | 372.935 | 0.671 | 0.694 | 0.337 | 0.792 | 0.819 | 0.397 |
| XML_COMPACT | man | 2.366 | 17.145 | 377.194 | 0.635 | 0.787 | 0.351 | 0.790 | 0.980 | 0.437 |
| XML_COMPACT | opt | 2.340 | 17.322 | 352.581 | 0.708 | 0.673 | 0.345 | 0.843 | 0.802 | 0.411 |
| XML_PRETTY | man | 1.934 | 23.704 | 521.484 | 0.464 | 0.587 | 0.259 | 0.571 | 0.723 | 0.319 |
| XML_PRETTY | opt | 1.917 | 23.913 | 486.742 | 0.517 | 0.777 | 0.310 | 0.612 | 0.921 | 0.368 |
| YAML | man | 1.664 | 18.408 | 404.968 | 0.585 | 0.693 | 0.317 | 0.733 | 0.870 | 0.398 |
| YAML | opt | 1.649 | 18.655 | 379.710 | 0.646 | 0.724 | 0.341 | 0.775 | 0.868 | 0.409 |

#### 2.5.2 Characters And Values: Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.444 | 1.429 | -0.015 | -1.04 | 10.248 | 10.618 |  +0.370 |  +3.61 | 225.452 | 216.129 | -9.323 | -4.14 |
| JSON_COMPACT | 2.149 | 2.113 | -0.036 | -1.68 | 13.589 | 13.864 |  +0.275 |  +2.02 | 298.968 | 282.194 | -16.774 | -5.61 |
| JSON_PRETTY | 1.694 | 1.680 | -0.014 | -0.83 | 20.943 | 21.184 |  +0.241 |  +1.15 | 460.742 | 431.194 | -29.548 | -6.41 |
| TOON_DEFAULT | 1.442 | 1.698 |  +0.256 |  +17.75 | 10.334 | 18.322 |  +7.988 |  +77.30 | 227.355 | 372.935 |  +145.580 |  +64.03 |
| XML_COMPACT | 2.366 | 2.340 | -0.026 | -1.10 | 17.145 | 17.322 |  +0.177 |  +1.03 | 377.194 | 352.581 | -24.613 | -6.53 |
| XML_PRETTY | 1.934 | 1.917 | -0.017 | -0.88 | 23.704 | 23.913 |  +0.209 |  +0.88 | 521.484 | 486.742 | -34.742 | -6.66 |
| YAML | 1.664 | 1.649 | -0.015 | -0.90 | 18.408 | 18.655 |  +0.247 |  +1.34 | 404.968 | 379.710 | -25.258 | -6.24 |

#### 2.5.3 Information: Mandatory vs Optional
| Format | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 0.885 | 0.899 |  +0.014 |  +1.58 | 0.648 | 0.469 | -0.179 | -27.62 | 0.374 | 0.308 | -0.066 | -17.65 |
| JSON_COMPACT | 0.774 | 0.873 |  +0.099 |  +12.79 | 0.545 | 0.763 |  +0.218 |  +40.00 | 0.320 | 0.407 |  +0.087 |  +27.19 |
| JSON_PRETTY | 0.555 | 0.557 |  +0.002 |  +0.36 | 0.604 | 0.897 |  +0.293 |  +48.51 | 0.289 | 0.344 |  +0.055 |  +19.03 |
| TOON_DEFAULT | 1.089 | 0.671 | -0.418 | -38.38 | 0.676 | 0.694 |  +0.017 |  +2.59 | 0.414 | 0.337 | -0.078 | -18.70 |
| XML_COMPACT | 0.635 | 0.708 |  +0.073 |  +11.50 | 0.787 | 0.673 | -0.114 | -14.49 | 0.351 | 0.345 | -0.006 | -1.71 |
| XML_PRETTY | 0.464 | 0.517 |  +0.053 |  +11.42 | 0.587 | 0.777 |  +0.190 |  +32.37 | 0.259 | 0.310 |  +0.051 |  +19.69 |
| YAML | 0.585 | 0.646 |  +0.061 |  +10.43 | 0.693 | 0.724 |  +0.031 |  +4.47 | 0.317 | 0.341 |  +0.024 |  +7.57 |

#### 2.5.4 Information (Accuracy By Character): Mandatory vs Optional
| Format | Info / Read Token (Acc By Char) Man | Info / Read Token (Acc By Char) Opt | Diff | Diff (%) | Info / Output Token (Acc By Char) Man | Info / Output Token (Acc By Char)  Opt | Diff | Diff (%) | Info / Total Token (Acc By Char) Man | Info / Total Token (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.205 | 1.284 |  +0.079 |  +6.56 | 0.883 | 0.670 | -0.213 | -24.12 | 0.510 | 0.440 | -0.070 | -13.73 |
| JSON_COMPACT | 0.985 | 1.043 |  +0.058 |  +5.89 | 0.693 | 0.912 |  +0.219 |  +31.60 | 0.407 | 0.487 |  +0.080 |  +19.66 |
| JSON_PRETTY | 0.655 | 0.680 |  +0.025 |  +3.82 | 0.712 | 1.095 |  +0.383 |  +53.79 | 0.341 | 0.420 |  +0.079 |  +23.17 |
| TOON_DEFAULT | 1.323 | 0.792 | -0.531 | -40.14 | 0.821 | 0.819 | -0.002 | -0.24 | 0.504 | 0.397 | -0.106 | -21.05 |
| XML_COMPACT | 0.790 | 0.843 |  +0.053 |  +6.71 | 0.980 | 0.802 | -0.178 | -18.16 | 0.437 | 0.411 | -0.026 | -5.95 |
| XML_PRETTY | 0.571 | 0.612 |  +0.041 |  +7.18 | 0.723 | 0.921 |  +0.198 |  +27.39 | 0.319 | 0.368 |  +0.049 |  +15.36 |
| YAML | 0.733 | 0.775 |  +0.042 |  +5.73 | 0.870 | 0.868 | -0.002 | -0.23 | 0.398 | 0.409 |  +0.011 |  +2.76 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6989 | 4321 | 2668 | 9536 | 5896 | 3640 | 16525 | 10218 | 6308 | 61.83 | 73.50 | 67.77 | 74.53 | 60.79 | 72.81 | 67.08 | 73.83 |
| CSV | opt | 6700 | 4035 | 2665 | 12845 | 7735 | 5110 | 19545 | 11770 | 7775 | 60.22 | 73.44 | 48.66 | 65.36 | 59.45 | 72.93 | 48.15 | 64.85 |
| JSON_COMPACT | man | 9268 | 6652 | 2616 | 13161 | 9445 | 3715 | 22429 | 16097 | 6332 | 71.77 | 72.12 | 54.64 | 65.34 | 69.79 | 70.80 | 53.32 | 64.02 |
| JSON_COMPACT | opt | 8748 | 6678 | 2070 | 10005 | 7638 | 2367 | 18753 | 14316 | 4437 | 76.34 | 76.99 | 74.89 | 78.23 | 76.30 | 76.97 | 74.86 | 78.20 |
| JSON_PRETTY | man | 14283 | 11326 | 2957 | 13138 | 10419 | 2720 | 27421 | 21745 | 5676 | 79.30 | 59.52 | 59.78 | 56.98 | 77.39 | 58.24 | 58.50 | 55.71 |
| JSON_PRETTY | opt | 13367 | 9953 | 3414 | 8302 | 6182 | 2120 | 21669 | 16135 | 5534 | 74.46 | 59.51 | 82.92 | 69.16 | 73.38 | 58.79 | 82.20 | 68.44 |
| TOON_DEFAULT | man | 7048 | 5409 | 1639 | 11671 | 8957 | 2713 | 18719 | 14367 | 4352 | 76.75 | 83.24 | 66.08 | 78.60 | 74.09 | 81.47 | 64.31 | 76.82 |
| TOON_DEFAULT | opt | 11561 | 8966 | 2595 | 11768 | 9126 | 2642 | 23329 | 18091 | 5237 | 77.55 | 67.92 | 66.08 | 66.78 | 77.12 | 67.63 | 65.80 | 66.49 |
| XML_COMPACT | man | 11693 | 8676 | 3017 | 9429 | 6996 | 2433 | 21122 | 15672 | 5449 | 74.20 | 65.22 | 76.60 | 70.46 | 72.52 | 64.10 | 75.48 | 69.34 |
| XML_COMPACT | opt | 10930 | 8462 | 2468 | 11498 | 8902 | 2596 | 22428 | 17364 | 5064 | 77.42 | 70.05 | 67.47 | 69.10 | 76.14 | 69.19 | 66.61 | 68.25 |
| XML_PRETTY | man | 16166 | 12125 | 4042 | 12780 | 9585 | 3195 | 28946 | 21710 | 7237 | 75.00 | 50.03 | 58.86 | 50.03 | 73.78 | 49.22 | 58.05 | 49.21 |
| XML_PRETTY | opt | 15089 | 11763 | 3326 | 10028 | 7818 | 2210 | 25117 | 19581 | 5536 | 77.96 | 55.79 | 75.84 | 62.26 | 75.39 | 54.08 | 74.13 | 60.55 |
| YAML | man | 12554 | 9213 | 3341 | 10585 | 7768 | 2817 | 23139 | 16982 | 6157 | 73.39 | 61.65 | 69.76 | 64.51 | 71.26 | 60.23 | 68.34 | 63.09 |
| YAML | opt | 11771 | 8954 | 2817 | 10509 | 7994 | 2515 | 22280 | 16949 | 5332 | 76.07 | 66.19 | 71.96 | 68.60 | 75.66 | 65.92 | 71.68 | 68.33 |

#### 2.6.2 Read Tokens: Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6989 | 6700 | -289 | -4.14 | 4321 | 4034 | -287 | -6.63 | 2668 | 2666 | -2 | -0.09 | 61.83 | 60.22 | -1.61 | -2.60 | 73.50 | 73.44 | -0.06 | -0.08 | 60.79 | 59.45 | -1.34 | -2.20 | 72.81 | 72.93 |  +0.12 |  +0.17 |
| JSON_COMPACT | 9268 | 8748 | -520 | -5.61 | 6652 | 6679 |  +27 |  +0.40 | 2616 | 2069 | -547 | -20.89 | 71.77 | 76.34 |  +4.57 |  +6.37 | 72.12 | 76.99 |  +4.87 |  +6.76 | 69.79 | 76.30 |  +6.51 |  +9.33 | 70.80 | 76.97 |  +6.17 |  +8.71 |
| JSON_PRETTY | 14283 | 13367 | -916 | -6.41 | 11326 | 9953 | -1373 | -12.13 | 2957 | 3414 |  +457 |  +15.47 | 79.30 | 74.46 | -4.84 | -6.10 | 59.52 | 59.51 | -0.01 | -0.01 | 77.39 | 73.38 | -4.01 | -5.18 | 58.24 | 58.79 |  +0.55 |  +0.94 |
| TOON_DEFAULT | 7048 | 11561 |  +4513 |  +64.03 | 5409 | 8965 |  +3556 |  +65.75 | 1639 | 2596 |  +957 |  +58.38 | 76.75 | 77.55 |  +0.80 |  +1.04 | 83.24 | 67.92 | -15.33 | -18.41 | 74.09 | 77.12 |  +3.03 |  +4.09 | 81.47 | 67.63 | -13.84 | -16.99 |
| XML_COMPACT | 11693 | 10930 | -763 | -6.53 | 8676 | 8462 | -214 | -2.47 | 3017 | 2468 | -549 | -18.19 | 74.20 | 77.42 |  +3.22 |  +4.34 | 65.22 | 70.05 |  +4.83 |  +7.40 | 72.52 | 76.14 |  +3.62 |  +4.99 | 64.10 | 69.19 |  +5.09 |  +7.95 |
| XML_PRETTY | 16166 | 15089 | -1077 | -6.66 | 12125 | 11764 | -361 | -2.98 | 4042 | 3326 | -716 | -17.71 | 75.00 | 77.96 |  +2.96 |  +3.95 | 50.03 | 55.79 |  +5.76 |  +11.51 | 73.78 | 75.39 |  +1.61 |  +2.18 | 49.22 | 54.08 |  +4.86 |  +9.87 |
| YAML | 12554 | 11771 | -783 | -6.24 | 9213 | 8954 | -259 | -2.81 | 3341 | 2817 | -524 | -15.68 | 73.39 | 76.07 |  +2.68 |  +3.65 | 61.65 | 66.19 |  +4.54 |  +7.36 | 71.26 | 75.66 |  +4.40 |  +6.17 | 60.23 | 65.92 |  +5.68 |  +9.44 |

#### 2.6.3 Output Tokens: Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 9536 | 12844 |  +3308 |  +34.69 | 5896 | 7735 |  +1839 |  +31.19 | 3640 | 5110 |  +1470 |  +40.37 | 61.83 | 60.22 | -1.61 | -2.60 | 67.77 | 48.66 | -19.11 | -28.20 | 60.79 | 59.45 | -1.34 | -2.20 | 67.08 | 48.15 | -18.93 | -28.22 |
| JSON_COMPACT | 13161 | 10006 | -3155 | -23.97 | 9445 | 7638 | -1807 | -19.14 | 3715 | 2367 | -1348 | -36.29 | 71.77 | 76.34 |  +4.57 |  +6.37 | 54.64 | 74.89 |  +20.25 |  +37.06 | 69.79 | 76.30 |  +6.51 |  +9.33 | 53.32 | 74.86 |  +21.54 |  +40.40 |
| JSON_PRETTY | 13138 | 8302 | -4836 | -36.81 | 10419 | 6182 | -4237 | -40.67 | 2720 | 2121 | -599 | -22.03 | 79.30 | 74.46 | -4.84 | -6.10 | 59.78 | 82.92 |  +23.14 |  +38.71 | 77.39 | 73.38 | -4.01 | -5.18 | 58.50 | 82.20 |  +23.69 |  +40.50 |
| TOON_DEFAULT | 11671 | 11768 |  +97 |  +0.83 | 8957 | 9125 |  +168 |  +1.88 | 2713 | 2641 | -72 | -2.64 | 76.75 | 77.55 |  +0.80 |  +1.04 | 66.08 | 66.08 |  +0.01 |  +0.01 | 74.09 | 77.12 |  +3.03 |  +4.09 | 64.31 | 65.80 |  +1.49 |  +2.32 |
| XML_COMPACT | 9429 | 11499 |  +2070 |  +21.95 | 6996 | 8902 |  +1906 |  +27.24 | 2433 | 2597 |  +164 |  +6.73 | 74.20 | 77.42 |  +3.22 |  +4.34 | 76.60 | 67.47 | -9.14 | -11.93 | 72.52 | 76.14 |  +3.62 |  +4.99 | 75.48 | 66.61 | -8.87 | -11.75 |
| XML_PRETTY | 12780 | 10028 | -2752 | -21.53 | 9585 | 7818 | -1767 | -18.44 | 3195 | 2210 | -985 | -30.82 | 75.00 | 77.96 |  +2.96 |  +3.95 | 58.86 | 75.84 |  +16.98 |  +28.84 | 73.78 | 75.39 |  +1.61 |  +2.18 | 58.05 | 74.13 |  +16.08 |  +27.70 |
| YAML | 10585 | 10509 | -76 | -0.71 | 7768 | 7994 |  +226 |  +2.91 | 2817 | 2515 | -302 | -10.71 | 73.39 | 76.07 |  +2.68 |  +3.65 | 69.76 | 71.96 |  +2.20 |  +3.15 | 71.26 | 75.66 |  +4.40 |  +6.17 | 68.34 | 71.68 |  +3.35 |  +4.90 |

#### 2.6.4 Total Tokens: Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 16525 | 19544 |  +3019 |  +18.27 | 10218 | 11770 |  +1552 |  +15.19 | 6308 | 7775 |  +1467 |  +23.26 | 61.83 | 60.22 | -1.61 | -2.60 | 74.53 | 65.36 | -9.16 | -12.30 | 60.79 | 59.45 | -1.34 | -2.20 | 73.83 | 64.85 | -8.98 | -12.17 |
| JSON_COMPACT | 22429 | 18754 | -3675 | -16.39 | 16097 | 14316 | -1781 | -11.06 | 6332 | 4437 | -1895 | -29.92 | 71.77 | 76.34 |  +4.57 |  +6.37 | 65.34 | 78.23 |  +12.89 |  +19.73 | 69.79 | 76.30 |  +6.51 |  +9.33 | 64.02 | 78.20 |  +14.19 |  +22.16 |
| JSON_PRETTY | 27421 | 21669 | -5752 | -20.98 | 21745 | 16135 | -5610 | -25.80 | 5676 | 5534 | -142 | -2.50 | 79.30 | 74.46 | -4.84 | -6.10 | 56.98 | 69.16 |  +12.19 |  +21.39 | 77.39 | 73.38 | -4.01 | -5.18 | 55.71 | 68.44 |  +12.74 |  +22.87 |
| TOON_DEFAULT | 18719 | 23329 |  +4610 |  +24.63 | 14367 | 18092 |  +3725 |  +25.92 | 4352 | 5237 |  +885 |  +20.34 | 76.75 | 77.55 |  +0.80 |  +1.04 | 78.60 | 66.78 | -11.82 | -15.04 | 74.09 | 77.12 |  +3.03 |  +4.09 | 76.82 | 66.49 | -10.33 | -13.45 |
| XML_COMPACT | 21122 | 22429 |  +1307 |  +6.19 | 15672 | 17364 |  +1692 |  +10.79 | 5449 | 5064 | -385 | -7.07 | 74.20 | 77.42 |  +3.22 |  +4.34 | 70.46 | 69.10 | -1.35 | -1.92 | 72.52 | 76.14 |  +3.62 |  +4.99 | 69.34 | 68.25 | -1.09 | -1.57 |
| XML_PRETTY | 28946 | 25117 | -3829 | -13.23 | 21710 | 19582 | -2128 | -9.80 | 7237 | 5536 | -1701 | -23.50 | 75.00 | 77.96 |  +2.96 |  +3.95 | 50.03 | 62.26 |  +12.23 |  +24.45 | 73.78 | 75.39 |  +1.61 |  +2.18 | 49.21 | 60.55 |  +11.33 |  +23.03 |
| YAML | 23139 | 22280 | -859 | -3.71 | 16982 | 16949 | -33 | -0.19 | 6157 | 5331 | -826 | -13.41 | 73.39 | 76.07 |  +2.68 |  +3.65 | 64.51 | 68.60 |  +4.09 |  +6.34 | 71.26 | 75.66 |  +4.40 |  +6.17 | 63.09 | 68.33 |  +5.23 |  +8.30 |

### 2.7 Token Utilization Efficiency (Accuracy by Character)
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy by Character (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy by Character (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6989 | 5888 | 1101 | 9536 | 8034 | 1502 | 16525 | 13923 | 2603 | 84.25 | 88.45 | 82.72 | 89.47 | 85.21 | 89.09 | 83.36 | 90.11 |
| CSV | opt | 6700 | 5763 | 937 | 12845 | 11049 | 1796 | 19545 | 16812 | 2732 | 86.02 | 90.64 | 65.86 | 82.56 | 87.23 | 91.45 | 66.67 | 83.37 |
| JSON_COMPACT | man | 9268 | 8458 | 810 | 13161 | 12010 | 1150 | 22429 | 20468 | 1960 | 91.26 | 85.11 | 67.63 | 78.33 | 91.18 | 85.06 | 67.58 | 78.28 |
| JSON_COMPACT | opt | 8748 | 7983 | 765 | 10005 | 9130 | 875 | 18753 | 17112 | 1641 | 91.25 | 86.93 | 84.83 | 88.17 | 91.58 | 87.15 | 85.05 | 88.39 |
| JSON_PRETTY | man | 14283 | 13366 | 917 | 13138 | 12295 | 843 | 27421 | 25661 | 1760 | 93.58 | 69.04 | 69.30 | 66.50 | 93.30 | 68.85 | 69.11 | 66.31 |
| JSON_PRETTY | opt | 13367 | 12155 | 1212 | 8302 | 7549 | 753 | 21669 | 19704 | 1965 | 90.93 | 70.49 | 93.90 | 80.14 | 91.22 | 70.68 | 94.09 | 80.34 |
| TOON_DEFAULT | man | 7048 | 6570 | 478 | 11671 | 10880 | 791 | 18719 | 17450 | 1269 | 93.22 | 94.22 | 77.06 | 89.58 | 93.03 | 94.09 | 76.93 | 89.45 |
| TOON_DEFAULT | opt | 11561 | 10583 | 978 | 11768 | 10772 | 996 | 23329 | 21355 | 1974 | 91.54 | 77.24 | 75.41 | 76.10 | 91.96 | 77.52 | 75.69 | 76.38 |
| XML_COMPACT | man | 11693 | 10804 | 889 | 9429 | 8712 | 717 | 21122 | 19516 | 1605 | 92.40 | 77.35 | 88.73 | 82.59 | 92.38 | 77.34 | 88.72 | 82.58 |
| XML_COMPACT | opt | 10930 | 10076 | 854 | 11498 | 10600 | 898 | 22428 | 20677 | 1752 | 92.19 | 79.89 | 77.31 | 78.95 | 92.39 | 80.03 | 77.44 | 79.08 |
| XML_PRETTY | man | 16166 | 14931 | 1235 | 12780 | 11804 | 976 | 28946 | 26735 | 2212 | 92.36 | 61.61 | 70.44 | 61.60 | 92.45 | 61.67 | 70.50 | 61.66 |
| XML_PRETTY | opt | 15089 | 13942 | 1147 | 10028 | 9266 | 762 | 25117 | 23208 | 1909 | 92.40 | 65.42 | 85.47 | 71.89 | 92.20 | 65.29 | 85.33 | 71.75 |
| YAML | man | 12554 | 11557 | 997 | 10585 | 9745 | 840 | 23139 | 21302 | 1837 | 92.06 | 74.10 | 82.20 | 76.96 | 92.12 | 74.14 | 82.24 | 77.00 |
| YAML | opt | 11771 | 10739 | 1032 | 10509 | 9588 | 922 | 22280 | 20326 | 1954 | 91.23 | 76.30 | 82.06 | 78.71 | 91.58 | 76.53 | 82.30 | 78.94 |

#### 2.7.2 Read Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6989 | 6700 | -289 | -4.14 | 5888 | 5763 | -125 | -2.12 | 1101 | 937 | -164 | -14.91 | 84.25 | 86.02 |  +1.77 |  +2.10 | 88.45 | 90.64 |  +2.20 |  +2.48 | 85.21 | 87.23 |  +2.02 |  +2.37 | 89.09 | 91.45 |  +2.36 |  +2.65 |
| JSON_COMPACT | 9268 | 8748 | -520 | -5.61 | 8458 | 7983 | -475 | -5.62 | 810 | 765 | -45 | -5.50 | 91.26 | 91.25 | -0.01 | -0.01 | 85.11 | 86.93 |  +1.82 |  +2.14 | 91.18 | 91.58 |  +0.40 |  +0.44 | 85.06 | 87.15 |  +2.09 |  +2.46 |
| JSON_PRETTY | 14283 | 13367 | -916 | -6.41 | 13366 | 12155 | -1211 | -9.06 | 917 | 1212 |  +295 |  +32.22 | 93.58 | 90.93 | -2.65 | -2.83 | 69.04 | 70.49 |  +1.45 |  +2.10 | 93.30 | 91.22 | -2.08 | -2.23 | 68.85 | 70.68 |  +1.83 |  +2.66 |
| TOON_DEFAULT | 7048 | 11561 |  +4513 |  +64.03 | 6570 | 10583 |  +4013 |  +61.08 | 478 | 978 |  +500 |  +104.65 | 93.22 | 91.54 | -1.68 | -1.80 | 94.22 | 77.24 | -16.98 | -18.02 | 93.03 | 91.96 | -1.07 | -1.15 | 94.09 | 77.52 | -16.57 | -17.61 |
| XML_COMPACT | 11693 | 10930 | -763 | -6.53 | 10804 | 10076 | -728 | -6.74 | 889 | 854 | -35 | -3.94 | 92.40 | 92.19 | -0.21 | -0.23 | 77.35 | 79.89 |  +2.54 |  +3.28 | 92.38 | 92.39 |  +0.01 |  +0.01 | 77.34 | 80.03 |  +2.69 |  +3.48 |
| XML_PRETTY | 16166 | 15089 | -1077 | -6.66 | 14931 | 13942 | -989 | -6.62 | 1235 | 1147 | -88 | -7.15 | 92.36 | 92.40 |  +0.04 |  +0.04 | 61.61 | 65.42 |  +3.81 |  +6.19 | 92.45 | 92.20 | -0.25 | -0.27 | 61.67 | 65.29 |  +3.62 |  +5.87 |
| YAML | 12554 | 11771 | -783 | -6.24 | 11557 | 10738 | -819 | -7.08 | 997 | 1033 |  +36 |  +3.56 | 92.06 | 91.23 | -0.83 | -0.90 | 74.10 | 76.30 |  +2.20 |  +2.97 | 92.12 | 91.58 | -0.54 | -0.59 | 74.14 | 76.53 |  +2.39 |  +3.23 |

#### 2.7.3 Output Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 9536 | 12844 |  +3308 |  +34.69 | 8034 | 11049 |  +3015 |  +37.52 | 1502 | 1796 |  +294 |  +19.55 | 84.25 | 86.02 |  +1.77 |  +2.10 | 82.72 | 65.86 | -16.86 | -20.38 | 85.21 | 87.23 |  +2.02 |  +2.37 | 83.36 | 66.67 | -16.69 | -20.02 |
| JSON_COMPACT | 13161 | 10006 | -3155 | -23.97 | 12010 | 9129 | -2881 | -23.98 | 1150 | 875 | -275 | -23.89 | 91.26 | 91.25 | -0.01 | -0.01 | 67.63 | 84.83 |  +17.20 |  +25.43 | 91.18 | 91.58 |  +0.40 |  +0.44 | 67.58 | 85.05 |  +17.47 |  +25.85 |
| JSON_PRETTY | 13138 | 8302 | -4836 | -36.81 | 12295 | 7549 | -4746 | -38.60 | 843 | 753 | -90 | -10.73 | 93.58 | 90.93 | -2.65 | -2.83 | 69.30 | 93.90 |  +24.60 |  +35.50 | 93.30 | 91.22 | -2.08 | -2.23 | 69.11 | 94.09 |  +24.98 |  +36.14 |
| TOON_DEFAULT | 11671 | 11768 |  +97 |  +0.83 | 10880 | 10772 | -108 | -0.99 | 791 | 995 |  +204 |  +25.82 | 93.22 | 91.54 | -1.68 | -1.80 | 77.06 | 75.41 | -1.65 | -2.14 | 93.03 | 91.96 | -1.07 | -1.15 | 76.93 | 75.69 | -1.24 | -1.61 |
| XML_COMPACT | 9429 | 11499 |  +2070 |  +21.95 | 8712 | 10600 |  +1888 |  +21.67 | 717 | 898 |  +181 |  +25.31 | 92.40 | 92.19 | -0.21 | -0.23 | 88.73 | 77.31 | -11.42 | -12.87 | 92.38 | 92.39 |  +0.01 |  +0.01 | 88.72 | 77.44 | -11.28 | -12.71 |
| XML_PRETTY | 12780 | 10028 | -2752 | -21.53 | 11804 | 9266 | -2538 | -21.50 | 976 | 762 | -214 | -21.95 | 92.36 | 92.40 |  +0.04 |  +0.04 | 70.44 | 85.47 |  +15.03 |  +21.34 | 92.45 | 92.20 | -0.25 | -0.27 | 70.50 | 85.33 |  +14.84 |  +21.05 |
| YAML | 10585 | 10509 | -76 | -0.71 | 9745 | 9588 | -157 | -1.61 | 840 | 921 |  +81 |  +9.67 | 92.06 | 91.23 | -0.83 | -0.90 | 82.20 | 82.06 | -0.14 | -0.17 | 92.12 | 91.58 | -0.54 | -0.59 | 82.24 | 82.30 |  +0.05 |  +0.06 |

#### 2.7.4 Total Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 16525 | 19544 |  +3019 |  +18.27 | 13923 | 16813 |  +2890 |  +20.76 | 2603 | 2733 |  +130 |  +4.98 | 84.25 | 86.02 |  +1.77 |  +2.10 | 89.47 | 82.56 | -6.91 | -7.72 | 85.21 | 87.23 |  +2.02 |  +2.37 | 90.11 | 83.37 | -6.74 | -7.48 |
| JSON_COMPACT | 22429 | 18754 | -3675 | -16.39 | 20468 | 17112 | -3356 | -16.40 | 1960 | 1641 | -319 | -16.29 | 91.26 | 91.25 | -0.01 | -0.01 | 78.33 | 88.17 |  +9.84 |  +12.56 | 91.18 | 91.58 |  +0.40 |  +0.44 | 78.28 | 88.39 |  +10.11 |  +12.92 |
| JSON_PRETTY | 27421 | 21669 | -5752 | -20.98 | 25661 | 19704 | -5957 | -23.22 | 1760 | 1965 |  +205 |  +11.64 | 93.58 | 90.93 | -2.65 | -2.83 | 66.50 | 80.14 |  +13.65 |  +20.52 | 93.30 | 91.22 | -2.08 | -2.23 | 66.31 | 80.34 |  +14.03 |  +21.15 |
| TOON_DEFAULT | 18719 | 23329 |  +4610 |  +24.63 | 17450 | 21355 |  +3905 |  +22.38 | 1269 | 1973 |  +704 |  +55.51 | 93.22 | 91.54 | -1.68 | -1.80 | 89.58 | 76.10 | -13.47 | -15.04 | 93.03 | 91.96 | -1.07 | -1.15 | 89.45 | 76.38 | -13.06 | -14.61 |
| XML_COMPACT | 21122 | 22429 |  +1307 |  +6.19 | 19516 | 20676 |  +1160 |  +5.95 | 1605 | 1751 |  +146 |  +9.12 | 92.40 | 92.19 | -0.21 | -0.23 | 82.59 | 78.95 | -3.64 | -4.41 | 92.38 | 92.39 |  +0.01 |  +0.01 | 82.58 | 79.08 | -3.49 | -4.23 |
| XML_PRETTY | 28946 | 25117 | -3829 | -13.23 | 26735 | 23209 | -3526 | -13.19 | 2212 | 1909 | -303 | -13.68 | 92.36 | 92.40 |  +0.04 |  +0.04 | 61.60 | 71.89 |  +10.29 |  +16.70 | 92.45 | 92.20 | -0.25 | -0.27 | 61.66 | 71.75 |  +10.09 |  +16.37 |
| YAML | 23139 | 22280 | -859 | -3.71 | 21302 | 20327 | -975 | -4.58 | 1837 | 1954 |  +117 |  +6.36 | 92.06 | 91.23 | -0.83 | -0.90 | 76.96 | 78.71 |  +1.75 |  +2.27 | 92.12 | 91.58 | -0.54 | -0.59 | 77.00 | 78.94 |  +1.94 |  +2.52 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Expected Characters | Output Characters | Correct Characters | Incorrect Characters | Accuracy by Character (%) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 76.67 | 47.33 | 0.00 | 61.83 | 7777.00 | 8631.33 | 6900.33 | 1731.00 | 84.25 |
| CSV | opt | 74.67 | 49.33 | 0.00 | 60.22 | 8441.00 | 8930.33 | 7838.00 | 1092.33 | 86.02 |
| JSON_COMPACT | man | 89.00 | 35.00 | 0.00 | 71.77 | 7777.00 | 7848.67 | 7645.33 | 203.33 | 91.26 |
| JSON_COMPACT | opt | 94.67 | 29.33 | 0.00 | 76.34 | 8441.00 | 8636.67 | 8276.33 | 360.33 | 91.25 |
| JSON_PRETTY | man | 98.33 | 25.67 | 0.00 | 79.30 | 7777.00 | 7833.67 | 7658.67 | 175.00 | 93.58 |
| JSON_PRETTY | opt | 92.33 | 31.67 | 0.00 | 74.46 | 8441.00 | 8568.67 | 8262.67 | 306.00 | 90.93 |
| TOON_DEFAULT | man | 95.17 | 28.83 | 0.00 | 76.75 | 7777.00 | 8127.50 | 7461.50 | 666.00 | 93.22 |
| TOON_DEFAULT | opt | 96.17 | 27.83 | 0.00 | 77.55 | 8441.00 | 8631.50 | 8233.33 | 398.17 | 91.54 |
| XML_COMPACT | man | 92.00 | 32.00 | 0.00 | 74.20 | 7777.00 | 7879.00 | 7611.33 | 267.67 | 92.40 |
| XML_COMPACT | opt | 96.00 | 28.00 | 0.00 | 77.42 | 8441.00 | 8533.67 | 8347.00 | 186.67 | 92.19 |
| XML_PRETTY | man | 93.00 | 31.00 | 0.00 | 75.00 | 7777.00 | 7835.67 | 7594.00 | 241.67 | 92.36 |
| XML_PRETTY | opt | 96.67 | 27.33 | 0.00 | 77.96 | 8441.00 | 8557.00 | 8326.00 | 231.00 | 92.40 |
| YAML | man | 91.00 | 33.00 | 0.00 | 73.39 | 7777.00 | 7863.33 | 7617.67 | 245.67 | 92.06 |
| YAML | opt | 94.33 | 29.67 | 0.00 | 76.07 | 8441.00 | 8526.00 | 8337.67 | 188.33 | 91.23 |

#### 2.8.2 Answers: Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 76.67 | 74.67 | -2 | -2.61 | 47.33 | 49.33 |  +2 |  +4.23 | 0.00 | 0.00 | 0 | 0.00 | 61.83 | 60.22 | -1.61 |
| JSON_COMPACT | 89.00 | 94.67 |  +6 |  +6.37 | 35.00 | 29.33 | -6 | -16.20 | 0.00 | 0.00 | 0 | 0.00 | 71.77 | 76.34 |  +4.57 |
| JSON_PRETTY | 98.33 | 92.33 | -6 | -6.10 | 25.67 | 31.67 |  +6 |  +23.37 | 0.00 | 0.00 | 0 | 0.00 | 79.30 | 74.46 | -4.84 |
| TOON_DEFAULT | 95.17 | 96.17 |  +1 |  +1.05 | 28.83 | 27.83 | -1 | -3.47 | 0.00 | 0.00 | 0 | 0.00 | 76.75 | 77.55 |  +0.80 |
| XML_COMPACT | 92.00 | 96.00 |  +4 |  +4.35 | 32.00 | 28.00 | -4 | -12.50 | 0.00 | 0.00 | 0 | 0.00 | 74.20 | 77.42 |  +3.22 |
| XML_PRETTY | 93.00 | 96.67 |  +4 |  +3.95 | 31.00 | 27.33 | -4 | -11.84 | 0.00 | 0.00 | 0 | 0.00 | 75.00 | 77.96 |  +2.96 |
| YAML | 91.00 | 94.33 |  +3 |  +3.66 | 33.00 | 29.67 | -3 | -10.09 | 0.00 | 0.00 | 0 | 0.00 | 73.39 | 76.07 |  +2.68 |

#### 2.8.3 Characters: Mandatory vs Optional Data
| Format | Output Characters Man | Output Characters Opt | Diff | Diff (%) | Correct Characters Man | Correct Characters Opt | Diff | Diff (%) | Incorrect Characters Man | Incorrect Characters Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 8631.33 | 8930.33 |  +299 |  +3.46 | 6900.33 | 7838.00 |  +938 |  +13.59 | 1731.00 | 1092.33 | -639 | -36.90 | 84.25 | 86.02 |  +1.77 |
| JSON_COMPACT | 7848.67 | 8636.67 |  +788 |  +10.04 | 7645.33 | 8276.33 |  +631 |  +8.25 | 203.33 | 360.33 |  +157 |  +77.21 | 91.26 | 91.25 | -0.01 |
| JSON_PRETTY | 7833.67 | 8568.67 |  +735 |  +9.38 | 7658.67 | 8262.67 |  +604 |  +7.89 | 175.00 | 306.00 |  +131 |  +74.86 | 93.58 | 90.93 | -2.65 |
| TOON_DEFAULT | 8127.50 | 8631.50 |  +504 |  +6.20 | 7461.50 | 8233.33 |  +772 |  +10.34 | 666.00 | 398.17 | -268 | -40.22 | 93.22 | 91.54 | -1.68 |
| XML_COMPACT | 7879.00 | 8533.67 |  +655 |  +8.31 | 7611.33 | 8347.00 |  +736 |  +9.67 | 267.67 | 186.67 | -81 | -30.26 | 92.40 | 92.19 | -0.21 |
| XML_PRETTY | 7835.67 | 8557.00 |  +721 |  +9.21 | 7594.00 | 8326.00 |  +732 |  +9.64 | 241.67 | 231.00 | -11 | -4.41 | 92.36 | 92.40 |  +0.04 |
| YAML | 7863.33 | 8526.00 |  +663 |  +8.43 | 7617.67 | 8337.67 |  +720 |  +9.45 | 245.67 | 188.33 | -57 | -23.34 | 92.06 | 91.23 | -0.83 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 61.83 | 73.94 | 55.56 | 52.38 | 47.62 | 85.21 | 27.73 | 16.20 | 10.91 | 5.95 |
| CSV | opt | 60.22 | 75.15 | 46.91 | 63.49 | 34.92 | 87.23 | 28.18 | 13.68 | 13.23 | 4.36 |
| JSON_COMPACT | man | 71.77 | 97.58 | 55.55 | 58.73 | 38.10 | 91.18 | 36.59 | 16.20 | 12.23 | 4.76 |
| JSON_COMPACT | opt | 76.34 | 90.30 | 74.07 | 71.43 | 47.62 | 91.58 | 33.86 | 21.61 | 14.88 | 5.95 |
| JSON_PRETTY | man | 79.30 | 98.79 | 66.67 | 65.08 | 58.73 | 93.30 | 37.05 | 19.44 | 13.56 | 7.34 |
| JSON_PRETTY | opt | 74.46 | 95.15 | 66.67 | 61.90 | 42.86 | 91.22 | 35.68 | 19.44 | 12.90 | 5.36 |
| TOON_DEFAULT | man | 76.75 | 94.55 | 57.41 | 65.08 | 66.67 | 93.03 | 35.45 | 16.74 | 13.56 | 8.33 |
| TOON_DEFAULT | opt | 77.55 | 93.94 | 72.22 | 71.43 | 47.62 | 91.96 | 35.23 | 21.06 | 14.88 | 5.95 |
| XML_COMPACT | man | 74.20 | 92.12 | 61.73 | 63.49 | 53.97 | 92.38 | 34.55 | 18.00 | 13.23 | 6.75 |
| XML_COMPACT | opt | 77.42 | 98.79 | 66.67 | 66.67 | 46.03 | 92.39 | 37.05 | 19.45 | 13.89 | 5.76 |
| XML_PRETTY | man | 75.00 | 97.57 | 61.73 | 68.26 | 39.68 | 92.45 | 36.59 | 18.00 | 14.22 | 4.96 |
| XML_PRETTY | opt | 77.96 | 98.18 | 59.26 | 65.08 | 61.90 | 92.20 | 36.82 | 17.28 | 13.56 | 7.74 |
| YAML | man | 73.39 | 98.79 | 48.15 | 73.02 | 39.68 | 92.12 | 37.05 | 14.04 | 15.21 | 4.96 |
| YAML | opt | 76.07 | 97.58 | 69.14 | 69.84 | 34.92 | 91.58 | 36.59 | 20.16 | 14.55 | 4.36 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 73.94 | 75.15 |  +1.21 | 27.73 | 28.18 |  +0.45 |
| JSON_COMPACT | 97.58 | 90.30 | -7.27 | 36.59 | 33.86 | -2.73 |
| JSON_PRETTY | 98.79 | 95.15 | -3.64 | 37.05 | 35.68 | -1.36 |
| TOON_DEFAULT | 94.55 | 93.94 | -0.61 | 35.45 | 35.23 | -0.23 |
| XML_COMPACT | 92.12 | 98.79 |  +6.67 | 34.55 | 37.05 |  +2.50 |
| XML_PRETTY | 97.57 | 98.18 |  +0.61 | 36.59 | 36.82 |  +0.23 |
| YAML | 98.79 | 97.58 | -1.21 | 37.05 | 36.59 | -0.46 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 55.56 | 46.91 | -8.64 | 16.20 | 13.68 | -2.52 |
| JSON_COMPACT | 55.55 | 74.07 |  +18.52 | 16.20 | 21.61 |  +5.41 |
| JSON_PRETTY | 66.67 | 66.67 | 0.00 | 19.44 | 19.44 | -0.00 |
| TOON_DEFAULT | 57.41 | 72.22 |  +14.81 | 16.74 | 21.06 |  +4.32 |
| XML_COMPACT | 61.73 | 66.67 |  +4.94 | 18.00 | 19.45 |  +1.44 |
| XML_PRETTY | 61.73 | 59.26 | -2.47 | 18.00 | 17.28 | -0.72 |
| YAML | 48.15 | 69.14 |  +20.99 | 14.04 | 20.16 |  +6.12 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 52.38 | 63.49 |  +11.11 | 10.91 | 13.23 |  +2.32 |
| JSON_COMPACT | 58.73 | 71.43 |  +12.70 | 12.23 | 14.88 |  +2.65 |
| JSON_PRETTY | 65.08 | 61.90 | -3.17 | 13.56 | 12.90 | -0.66 |
| TOON_DEFAULT | 65.08 | 71.43 |  +6.35 | 13.56 | 14.88 |  +1.32 |
| XML_COMPACT | 63.49 | 66.67 |  +3.18 | 13.23 | 13.89 |  +0.66 |
| XML_PRETTY | 68.26 | 65.08 | -3.18 | 14.22 | 13.56 | -0.66 |
| YAML | 73.02 | 69.84 | -3.18 | 15.21 | 14.55 | -0.66 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 47.62 | 34.92 | -12.70 | 5.95 | 4.36 | -1.59 |
| JSON_COMPACT | 38.10 | 47.62 |  +9.52 | 4.76 | 5.95 |  +1.19 |
| JSON_PRETTY | 58.73 | 42.86 | -15.87 | 7.34 | 5.36 | -1.98 |
| TOON_DEFAULT | 66.67 | 47.62 | -19.05 | 8.33 | 5.95 | -2.38 |
| XML_COMPACT | 53.97 | 46.03 | -7.93 | 6.75 | 5.76 | -0.99 |
| XML_PRETTY | 39.68 | 61.90 |  +22.22 | 4.96 | 7.74 |  +2.77 |
| YAML | 39.68 | 34.92 | -4.76 | 4.96 | 4.36 | -0.60 |

### 2.10 Accuracy By Character Per Question Category Analysis
#### 2.10.1 Metrics
| Format | Variant | Accuracy By Character (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc By Char (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 84.25 | 83.56 | 87.71 | 90.79 | 75.12 | 85.21 | 31.33 | 25.58 | 18.91 | 9.39 |
| CSV | opt | 86.02 | 88.00 | 90.98 | 91.59 | 68.89 | 87.23 | 33.00 | 26.54 | 19.08 | 8.61 |
| JSON_COMPACT | man | 91.26 | 98.97 | 89.42 | 88.84 | 75.84 | 91.18 | 37.11 | 26.08 | 18.51 | 9.48 |
| JSON_COMPACT | opt | 91.25 | 97.64 | 89.98 | 93.54 | 73.86 | 91.58 | 36.61 | 26.25 | 19.49 | 9.23 |
| JSON_PRETTY | man | 93.58 | 99.55 | 90.50 | 91.64 | 83.85 | 93.30 | 37.33 | 26.39 | 19.09 | 10.48 |
| JSON_PRETTY | opt | 90.93 | 98.85 | 91.60 | 88.78 | 71.47 | 91.22 | 37.07 | 26.72 | 18.49 | 8.94 |
| TOON_DEFAULT | man | 93.22 | 98.14 | 89.83 | 93.52 | 84.40 | 93.03 | 36.80 | 26.20 | 19.48 | 10.55 |
| TOON_DEFAULT | opt | 91.54 | 97.51 | 91.32 | 93.63 | 74.06 | 91.96 | 36.57 | 26.63 | 19.50 | 9.26 |
| XML_COMPACT | man | 92.40 | 97.80 | 89.97 | 92.80 | 80.98 | 92.38 | 36.68 | 26.24 | 19.33 | 10.12 |
| XML_COMPACT | opt | 92.19 | 99.83 | 91.01 | 92.01 | 73.86 | 92.39 | 37.44 | 26.54 | 19.17 | 9.23 |
| XML_PRETTY | man | 92.36 | 98.95 | 89.78 | 93.70 | 77.08 | 92.45 | 37.11 | 26.18 | 19.52 | 9.63 |
| XML_PRETTY | opt | 92.40 | 99.54 | 89.23 | 90.90 | 79.28 | 92.20 | 37.33 | 26.03 | 18.94 | 9.91 |
| YAML | man | 92.06 | 99.71 | 88.16 | 94.55 | 74.56 | 92.12 | 37.39 | 25.71 | 19.70 | 9.32 |
| YAML | opt | 91.23 | 99.60 | 90.51 | 91.75 | 69.69 | 91.58 | 37.35 | 26.40 | 19.11 | 8.71 |

#### 2.10.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 83.56 | 88.00 |  +4.44 | 31.33 | 33.00 |  +1.67 |
| JSON_COMPACT | 98.97 | 97.64 | -1.33 | 37.11 | 36.61 | -0.50 |
| JSON_PRETTY | 99.55 | 98.85 | -0.71 | 37.33 | 37.07 | -0.26 |
| TOON_DEFAULT | 98.14 | 97.51 | -0.64 | 36.80 | 36.57 | -0.24 |
| XML_COMPACT | 97.80 | 99.83 |  +2.03 | 36.68 | 37.44 |  +0.76 |
| XML_PRETTY | 98.95 | 99.54 |  +0.59 | 37.11 | 37.33 |  +0.22 |
| YAML | 99.71 | 99.60 | -0.10 | 37.39 | 37.35 | -0.04 |

#### 2.10.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 87.71 | 90.98 |  +3.27 | 25.58 | 26.54 |  +0.96 |
| JSON_COMPACT | 89.42 | 89.98 |  +0.56 | 26.08 | 26.25 |  +0.17 |
| JSON_PRETTY | 90.50 | 91.60 |  +1.10 | 26.39 | 26.72 |  +0.32 |
| TOON_DEFAULT | 89.83 | 91.32 |  +1.49 | 26.20 | 26.63 |  +0.43 |
| XML_COMPACT | 89.97 | 91.01 |  +1.04 | 26.24 | 26.54 |  +0.30 |
| XML_PRETTY | 89.78 | 89.23 | -0.54 | 26.18 | 26.03 | -0.15 |
| YAML | 88.16 | 90.51 |  +2.35 | 25.71 | 26.40 |  +0.69 |

#### 2.10.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 90.79 | 91.59 |  +0.80 | 18.91 | 19.08 |  +0.17 |
| JSON_COMPACT | 88.84 | 93.54 |  +4.71 | 18.51 | 19.49 |  +0.98 |
| JSON_PRETTY | 91.64 | 88.78 | -2.86 | 19.09 | 18.49 | -0.60 |
| TOON_DEFAULT | 93.52 | 93.63 |  +0.11 | 19.48 | 19.50 |  +0.02 |
| XML_COMPACT | 92.80 | 92.01 | -0.79 | 19.33 | 19.17 | -0.16 |
| XML_PRETTY | 93.70 | 90.90 | -2.80 | 19.52 | 18.94 | -0.59 |
| YAML | 94.55 | 91.75 | -2.80 | 19.70 | 19.11 | -0.59 |

#### 2.10.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 75.12 | 68.89 | -6.23 | 9.39 | 8.61 | -0.78 |
| JSON_COMPACT | 75.84 | 73.86 | -1.99 | 9.48 | 9.23 | -0.25 |
| JSON_PRETTY | 83.85 | 71.47 | -12.38 | 10.48 | 8.94 | -1.54 |
| TOON_DEFAULT | 84.40 | 74.06 | -10.34 | 10.55 | 9.26 | -1.29 |
| XML_COMPACT | 80.98 | 73.86 | -7.12 | 10.12 | 9.23 | -0.89 |
| XML_PRETTY | 77.08 | 79.28 |  +2.19 | 9.63 | 9.91 |  +0.28 |
| YAML | 74.56 | 69.69 | -4.86 | 9.32 | 8.71 | -0.60 |

## 3. Appendices

### 3.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: on
- **Structure**: flat
- **Formats Tested**: CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML
- **Record Counts**: 31
- **Total Test Cases**: 14

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