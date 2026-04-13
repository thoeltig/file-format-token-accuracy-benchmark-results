# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: off
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
| YAML ≈ 76s | CSV ≈ 7022 | CSV ≈ 241 | XML_COMPACT ≈ 6673 | XML_COMPACT ≈ 6976 | CSV ≈ 17340 | JSON_PRETTY ≈ 79.57% | TOON_DEFAULT ≈ 81 | XML_COMPACT ≈ 81 | TOON_DEFAULT ≈ 77 | JSON_PRETTY ≈ 92.57% | TOON_DEFAULT ≈ 93 | XML_COMPACT ≈ 94 | CSV ≈ 91 |
| XML_COMPACT (+5.78%) | TOON_DEFAULT (+2.28%) | TOON_DEFAULT (+7.02%) | YAML (+35.93%) | YAML (+34.36%) | TOON_DEFAULT (+8.40%) | JSON_COMPACT (-3.56%) | CSV (-6.92%) | YAML (-15.17%) | CSV (-1.33%) | XML_PRETTY (-0.43%) | CSV (-3.74%) | YAML (-12.78%) | TOON_DEFAULT (-1.20%) |
| CSV (+6.64%) | JSON_COMPACT (+32.38%) | JSON_PRETTY (+25.38%) | CSV (+51.01%) | CSV (+47.91%) | XML_COMPACT (+9.15%) | TOON_DEFAULT (-5.82%) | JSON_COMPACT (-7.33%) | XML_PRETTY (-21.62%) | XML_COMPACT (-2.98%) | JSON_COMPACT (-0.59%) | JSON_COMPACT (-7.92%) | XML_PRETTY (-19.14%) | XML_COMPACT (-2.01%) |
| XML_PRETTY (+13.92%) | XML_COMPACT (+70.18%) | YAML (+25.38%) | XML_PRETTY (+54.29%) | XML_PRETTY (+51.95%) | YAML (+26.57%) | XML_PRETTY (-6.99%) | XML_COMPACT (-23.03%) | JSON_PRETTY (-24.55%) | JSON_COMPACT (-16.21%) | TOON_DEFAULT (-0.70%) | XML_COMPACT (-18.34%) | CSV (-22.09%) | YAML (-13.56%) |
| TOON_DEFAULT (+25.23%) | YAML (+79.07%) | XML_COMPACT (+25.80%) | TOON_DEFAULT (+70.19%) | TOON_DEFAULT (+66.50%) | JSON_COMPACT (+31.40%) | XML_COMPACT (-8.60%) | YAML (-25.74%) | CSV (-26.50%) | YAML (-16.74%) | YAML (-0.83%) | YAML (-20.49%) | TOON_DEFAULT (-24.82%) | JSON_COMPACT (-16.64%) |
| JSON_PRETTY (+26.80%) | JSON_PRETTY (+103.80%) | JSON_COMPACT (+26.35%) | JSON_PRETTY (+74.95%) | JSON_PRETTY (+71.68%) | JSON_PRETTY (+51.60%) | YAML (-8.60%) | JSON_PRETTY (-26.21%) | TOON_DEFAULT (-27.07%) | JSON_PRETTY (-29.09%) | XML_COMPACT (-1.11%) | JSON_PRETTY (-26.47%) | JSON_PRETTY (-26.29%) | JSON_PRETTY (-29.83%) |
| JSON_COMPACT (+40.46%) | XML_PRETTY (+130.50%) | XML_PRETTY (+26.49%) | JSON_COMPACT (+97.57%) | JSON_COMPACT (+93.36%) | XML_PRETTY (+54.48%) | CSV (-15.06%) | XML_PRETTY (-40.12%) | JSON_COMPACT (-37.06%) | XML_PRETTY (-37.39%) | CSV (-6.76%) | XML_PRETTY (-33.88%) | JSON_COMPACT (-34.89%) | XML_PRETTY (-32.09%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 74s | CSV ≈ 6728 | JSON_PRETTY ≈ 231 | TOON_DEFAULT ≈ 8675 | TOON_DEFAULT ≈ 8984 | CSV ≈ 17689 | YAML ≈ 78.23% | JSON_COMPACT ≈ 76 | TOON_DEFAULT ≈ 72 | JSON_COMPACT ≈ 80 | JSON_PRETTY ≈ 92.07% | CSV ≈ 91 | TOON_DEFAULT ≈ 83 | JSON_COMPACT ≈ 91 |
| XML_PRETTY (+0.30%) | JSON_COMPACT (+30.32%) | XML_PRETTY (+30.74%) | XML_PRETTY (+2.68%) | XML_PRETTY (+2.51%) | JSON_COMPACT (+3.60%) | JSON_PRETTY (-1.78%) | CSV (-0.46%) | XML_PRETTY (-0.44%) | CSV (-6.65%) | YAML (-0.17%) | JSON_COMPACT (-4.09%) | XML_PRETTY (-1.32%) | CSV (-1.36%) |
| JSON_COMPACT (+2.14%) | XML_COMPACT (+66.19%) | CSV (+31.43%) | JSON_COMPACT (+6.66%) | JSON_COMPACT (+6.38%) | TOON_DEFAULT (+16.98%) | XML_COMPACT (-1.89%) | XML_COMPACT (-9.59%) | JSON_COMPACT (-2.86%) | TOON_DEFAULT (-11.58%) | JSON_COMPACT (-0.93%) | XML_COMPACT (-13.99%) | JSON_COMPACT (-3.08%) | TOON_DEFAULT (-9.60%) |
| JSON_PRETTY (+14.79%) | TOON_DEFAULT (+74.03%) | YAML (+31.60%) | JSON_PRETTY (+17.35%) | JSON_PRETTY (+15.88%) | XML_COMPACT (+26.89%) | JSON_COMPACT (-3.71%) | YAML (-10.76%) | JSON_PRETTY (-7.13%) | XML_COMPACT (-16.73%) | XML_PRETTY (-1.39%) | YAML (-15.26%) | JSON_PRETTY (-7.55%) | XML_COMPACT (-16.56%) |
| CSV (+16.87%) | YAML (+75.25%) | JSON_COMPACT (+31.95%) | CSV (+22.86%) | CSV (+22.01%) | JSON_PRETTY (+34.58%) | XML_PRETTY (-3.77%) | TOON_DEFAULT (-14.80%) | XML_COMPACT (-13.30%) | JSON_PRETTY (-22.65%) | TOON_DEFAULT (-1.47%) | TOON_DEFAULT (-15.90%) | XML_COMPACT (-14.11%) | JSON_PRETTY (-20.62%) |
| XML_COMPACT (+18.40%) | JSON_PRETTY (+99.09%) | XML_COMPACT (+33.77%) | XML_COMPACT (+26.30%) | XML_COMPACT (+25.39%) | XML_PRETTY (+37.52%) | TOON_DEFAULT (-5.02%) | JSON_PRETTY (-19.77%) | CSV (-23.30%) | YAML (-25.65%) | XML_COMPACT (-1.66%) | JSON_PRETTY (-21.37%) | CSV (-15.82%) | XML_PRETTY (-23.67%) |
| YAML (+40.42%) | XML_PRETTY (+124.69%) | TOON_DEFAULT (+33.84%) | YAML (+46.69%) | YAML (+45.02%) | YAML (+40.31%) | CSV (-15.00%) | XML_PRETTY (-29.51%) | YAML (-24.07%) | XML_PRETTY (-26.63%) | CSV (-6.14%) | XML_PRETTY (-29.07%) | YAML (-23.70%) | YAML (-24.69%) |


#### 2.1.2 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 98.18% | JSON_PRETTY ≈ 70.37% | JSON_PRETTY ≈ 66.67% | JSON_PRETTY ≈ 58.73% |
| YAML (-1.21%) | JSON_COMPACT (-1.85%) | XML_PRETTY (-3.17%) | XML_COMPACT (-1.59%) |
| JSON_PRETTY (-1.21%) | TOON_DEFAULT (-4.94%) | JSON_COMPACT (-3.57%) | CSV (-5.40%) |
| XML_PRETTY (-2.42%) | XML_COMPACT (-13.58%) | CSV (-4.76%) | TOON_DEFAULT (-9.52%) |
| TOON_DEFAULT (-5.86%) | CSV (-17.78%) | YAML (-4.76%) | YAML (-9.52%) |
| XML_COMPACT (-9.69%) | XML_PRETTY (-18.52%) | TOON_DEFAULT (-6.35%) | XML_PRETTY (-11.11%) |
| CSV (-22.54%) | YAML (-28.40%) | XML_COMPACT (-9.53%) | JSON_COMPACT (-18.25%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99.39% | XML_COMPACT ≈ 71.61% | YAML ≈ 73.02% | JSON_COMPACT ≈ 52.38% |
| XML_COMPACT (-1.82%) | XML_PRETTY (-1.24%) | JSON_PRETTY (-6.35%) | XML_PRETTY (0.00%) |
| JSON_PRETTY (-2.67%) | JSON_PRETTY (-6.42%) | CSV (-10.16%) | TOON_DEFAULT (-4.76%) |
| JSON_COMPACT (-4.48%) | YAML (-7.41%) | JSON_COMPACT (-11.11%) | JSON_PRETTY (-4.76%) |
| TOON_DEFAULT (-6.06%) | TOON_DEFAULT (-9.88%) | XML_COMPACT (-11.11%) | YAML (-6.35%) |
| XML_PRETTY (-8.49%) | JSON_COMPACT (-11.61%) | TOON_DEFAULT (-12.17%) | XML_COMPACT (-11.11%) |
| CSV (-20.12%) | CSV (-19.76%) | XML_PRETTY (-14.29%) | CSV (-16.19%) |


#### 2.1.3 Category Accuracy By Character Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99.22% | JSON_COMPACT ≈ 91.15% | XML_PRETTY ≈ 93.39% | XML_COMPACT ≈ 84.00% |
| JSON_COMPACT (-0.34%) | TOON_DEFAULT (-0.17%) | TOON_DEFAULT (-0.81%) | JSON_PRETTY (-3.62%) |
| JSON_PRETTY (-0.61%) | JSON_PRETTY (-0.77%) | JSON_PRETTY (-1.64%) | XML_PRETTY (-4.91%) |
| XML_PRETTY (-0.79%) | XML_COMPACT (-1.41%) | JSON_COMPACT (-1.68%) | TOON_DEFAULT (-5.20%) |
| TOON_DEFAULT (-2.19%) | XML_PRETTY (-2.62%) | CSV (-2.15%) | YAML (-5.82%) |
| XML_COMPACT (-3.20%) | CSV (-2.82%) | YAML (-2.33%) | CSV (-6.41%) |
| CSV (-13.59%) | YAML (-3.56%) | XML_COMPACT (-4.18%) | JSON_COMPACT (-8.72%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99.95% | XML_COMPACT ≈ 90.99% | YAML ≈ 93.07% | XML_PRETTY ≈ 76.18% |
| XML_COMPACT (-0.30%) | CSV (-0.55%) | JSON_PRETTY (-0.62%) | JSON_COMPACT (-0.05%) |
| JSON_PRETTY (-0.91%) | JSON_PRETTY (-0.74%) | CSV (-0.72%) | JSON_PRETTY (-0.34%) |
| JSON_COMPACT (-1.34%) | YAML (-0.83%) | TOON_DEFAULT (-2.84%) | TOON_DEFAULT (-0.95%) |
| TOON_DEFAULT (-2.26%) | XML_PRETTY (-0.85%) | JSON_COMPACT (-2.94%) | YAML (-4.30%) |
| XML_PRETTY (-2.81%) | JSON_COMPACT (-2.58%) | XML_PRETTY (-4.12%) | XML_COMPACT (-7.51%) |
| CSV (-12.10%) | TOON_DEFAULT (-2.61%) | XML_COMPACT (-5.87%) | CSV (-7.51%) |


#### 2.1.4 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total | Accuracy By Character (%) | Useful Read Tokens (Acc By Char) | Wasted Read Tokens (Acc By Char) | Useful Output Tokens (Acc By Char) | Wasted Output Tokens (Acc By Char) | Eff Score Read (Acc By Char) | Eff Score Output (Acc By Char) | Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7022 | 10318 | 17340 | 1.438 | 81.271 | 64.51 | 4529.892 | 2492.108 | 6656.271 | 3661.929 | 75.27 | 59.23 | 76.30 | 85.81 | 6025.578 | 996.422 | 8854.047 | 1464.153 | 89.47 | 73.43 | 90.50 |
| CSV | opt | 6728 | 10961 | 17689 | 1.423 | 85.950 | 63.23 | 4254.114 | 2473.886 | 6930.893 | 4030.507 | 75.45 | 55.10 | 74.22 | 85.93 | 5781.370 | 946.630 | 9419.131 | 1542.269 | 90.58 | 70.23 | 89.35 |
| JSON_COMPACT | man | 9296 | 13489 | 22785 | 2.143 | 106.327 | 76.01 | 7065.890 | 2230.110 | 10252.609 | 3235.891 | 74.94 | 50.72 | 64.80 | 91.98 | 8550.461 | 745.539 | 12406.722 | 1081.778 | 85.59 | 61.37 | 75.45 |
| JSON_COMPACT | opt | 8768 | 9558 | 18326 | 2.108 | 74.621 | 74.52 | 6533.914 | 2234.086 | 7122.473 | 2435.327 | 75.80 | 69.79 | 79.51 | 91.14 | 7991.155 | 776.845 | 8710.979 | 846.821 | 86.88 | 80.87 | 90.59 |
| JSON_PRETTY | man | 14311 | 11977 | 26288 | 1.691 | 94.153 | 79.57 | 11387.263 | 2923.737 | 9529.834 | 2446.833 | 59.67 | 60.81 | 54.84 | 92.57 | 13247.693 | 1063.307 | 11086.801 | 889.866 | 68.34 | 69.48 | 63.50 |
| JSON_PRETTY | opt | 13395 | 10411 | 23806 | 1.677 | 82.097 | 76.45 | 10240.478 | 3154.522 | 7959.210 | 2451.790 | 60.82 | 66.72 | 61.50 | 92.07 | 12332.777 | 1062.224 | 9585.408 | 825.592 | 71.23 | 77.13 | 71.91 |
| TOON_DEFAULT | man | 7182 | 11615 | 18797 | 1.415 | 91.593 | 73.75 | 5296.725 | 1885.275 | 8566.063 | 3048.937 | 80.87 | 58.78 | 77.33 | 91.87 | 6598.103 | 583.897 | 10670.700 | 944.300 | 92.95 | 70.86 | 89.41 |
| TOON_DEFAULT | opt | 11709 | 8984 | 20693 | 1.677 | 69.959 | 73.21 | 8572.159 | 3136.841 | 6577.309 | 2406.859 | 64.59 | 71.84 | 70.30 | 90.60 | 10608.354 | 1100.646 | 8139.655 | 844.512 | 76.18 | 83.43 | 81.89 |
| XML_COMPACT | man | 11950 | 6976 | 18926 | 2.315 | 53.817 | 70.97 | 8480.915 | 3469.085 | 4950.867 | 2025.133 | 62.25 | 80.59 | 75.03 | 91.46 | 10929.470 | 1020.530 | 6380.250 | 595.750 | 75.91 | 94.25 | 88.69 |
| XML_COMPACT | opt | 11181 | 11266 | 22447 | 2.288 | 88.360 | 76.34 | 8535.575 | 2645.425 | 8600.210 | 2665.457 | 68.53 | 62.29 | 66.21 | 90.41 | 10108.742 | 1072.258 | 10185.290 | 1080.377 | 77.91 | 71.67 | 75.59 |
| XML_PRETTY | man | 16186 | 10600 | 26786 | 1.931 | 83.032 | 72.58 | 11747.799 | 4438.201 | 7693.722 | 2906.611 | 48.42 | 63.17 | 48.42 | 92.14 | 14913.780 | 1272.220 | 9767.147 | 833.186 | 61.46 | 76.21 | 61.46 |
| XML_PRETTY | opt | 15117 | 9210 | 24327 | 1.913 | 71.836 | 74.46 | 11256.118 | 3860.882 | 6857.518 | 2352.149 | 53.43 | 71.52 | 58.34 | 90.68 | 13708.096 | 1408.904 | 8351.326 | 858.341 | 64.25 | 82.34 | 69.15 |
| YAML | man | 12574 | 9373 | 21947 | 1.661 | 73.153 | 70.97 | 8923.768 | 3650.232 | 6651.782 | 2720.885 | 60.05 | 68.37 | 64.39 | 91.74 | 11535.388 | 1038.612 | 8598.485 | 774.182 | 73.90 | 82.21 | 78.24 |
| YAML | opt | 11791 | 13029 | 24820 | 1.646 | 102.621 | 78.23 | 9224.099 | 2566.901 | 10192.587 | 2836.413 | 67.64 | 54.55 | 59.11 | 91.90 | 10835.929 | 955.071 | 11973.651 | 1055.349 | 76.76 | 63.66 | 68.22 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7022 | 6728 | -294 | -4.19 | 241 | 304 |  +63 |  +26.14 | 10078 | 10658 |  +580 |  +5.76 | 10318 | 10961 |  +643 |  +6.23 | 17340 | 17689 |  +349 |  +2.01 |
| JSON_COMPACT | 9296 | 8768 | -528 | -5.68 | 304 | 305 |  +1 |  +0.33 | 13185 | 9254 | -3931 | -29.81 | 13489 | 9558 | -3931 | -29.14 | 22785 | 18326 | -4459 | -19.57 |
| JSON_PRETTY | 14311 | 13395 | -916 | -6.40 | 302 | 231 | -71 | -23.51 | 11675 | 10180 | -1495 | -12.81 | 11977 | 10411 | -1566 | -13.08 | 26288 | 23806 | -2482 | -9.44 |
| TOON_DEFAULT | 7182 | 11709 |  +4527 |  +63.03 | 258 | 310 |  +52 |  +20.16 | 11358 | 8676 | -2682 | -23.61 | 11615 | 8984 | -2631 | -22.65 | 18797 | 20693 |  +1896 |  +10.09 |
| XML_COMPACT | 11950 | 11181 | -769 | -6.44 | 303 | 309 |  +6 |  +1.98 | 6673 | 10956 |  +4283 |  +64.18 | 6976 | 11266 |  +4290 |  +61.50 | 18926 | 22447 |  +3521 |  +18.60 |
| XML_PRETTY | 16186 | 15117 | -1069 | -6.60 | 304 | 302 | -2 | -0.66 | 10296 | 8908 | -1388 | -13.48 | 10600 | 9209 | -1391 | -13.12 | 26786 | 24326 | -2460 | -9.18 |
| YAML | 12574 | 11791 | -783 | -6.23 | 302 | 304 |  +2 |  +0.66 | 9071 | 12725 |  +3654 |  +40.28 | 9373 | 13029 |  +3656 |  +39.01 | 21947 | 24820 |  +2873 |  +13.09 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Before Write (ms) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) | Output (ms) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 28 | 250.786 | 0.90 | 46128 | 34797 | 0.290 | 280.62 | 34825 | 251.076 | 224.68 | 80925 |
| CSV | opt | 10 | 672.800 | 0.32 | 49658 | 36882 | 0.289 | 297.43 | 36892 | 673.089 | 238.01 | 86539 |
| JSON_COMPACT | man | 36 | 258.222 | 1.16 | 71360 | 35225 | 0.374 | 284.07 | 35261 | 258.596 | 227.49 | 106585 |
| JSON_COMPACT | opt | 9 | 974.222 | 0.29 | 40073 | 35563 | 0.260 | 286.80 | 35572 | 974.482 | 229.50 | 75636 |
| JSON_PRETTY | man | 13 | 1100.846 | 0.42 | 60406 | 35817 | 0.326 | 288.85 | 35830 | 1101.172 | 231.16 | 96223 |
| JSON_PRETTY | opt | 15 | 893.000 | 0.48 | 47655 | 37347 | 0.273 | 301.19 | 37362 | 893.273 | 241.05 | 85002 |
| TOON_DEFAULT | man | 27 | 266.000 | 0.87 | 56899 | 38129 | 0.298 | 307.49 | 38156 | 266.298 | 246.17 | 95028 |
| TOON_DEFAULT | opt | 27 | 433.667 | 0.87 | 37002 | 37047 | 0.237 | 298.77 | 37074 | 433.904 | 239.19 | 74050 |
| XML_COMPACT | man | 16 | 746.875 | 0.52 | 32957 | 47309 | 0.141 | 381.53 | 47325 | 747.016 | 305.32 | 80267 |
| XML_COMPACT | opt | 15 | 745.400 | 0.48 | 52664 | 35007 | 0.313 | 282.32 | 35022 | 745.713 | 225.95 | 87671 |
| XML_PRETTY | man | 18 | 899.222 | 0.58 | 50051 | 36393 | 0.283 | 293.49 | 36411 | 899.505 | 234.91 | 86445 |
| XML_PRETTY | opt | 9 | 1679.667 | 0.29 | 38045 | 36231 | 0.246 | 292.18 | 36240 | 1679.913 | 233.80 | 74275 |
| YAML | man | 11 | 1143.091 | 0.35 | 41079 | 34805 | 0.261 | 280.69 | 34816 | 1143.352 | 224.62 | 75884 |
| YAML | opt | 8 | 1473.875 | 0.26 | 65990 | 37991 | 0.335 | 306.38 | 37999 | 1474.210 | 245.15 | 103981 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Before Write Man (s) | Output Before Write Opt (s) | Diff (s) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 28 | 10 | -18 | -64.29 | 46.13 | 49.66 |  +3.53 |  +7.65 | 34.80 | 36.88 |  +2.08 |  +5.99 | 34.83 | 36.89 |  +2.07 |  +5.93 | 80.93 | 86.54 |  +5.61 |  +6.94 |
| JSON_COMPACT | 36 | 9 | -27 | -75.00 | 71.36 | 40.07 | -31.29 | -43.84 | 35.22 | 35.56 |  +0.34 |  +0.96 | 35.26 | 35.57 |  +0.31 |  +0.88 | 106.58 | 75.64 | -30.95 | -29.04 |
| JSON_PRETTY | 13 | 15 |  +2 |  +15.38 | 60.41 | 47.66 | -12.75 | -21.11 | 35.82 | 37.35 |  +1.53 |  +4.27 | 35.83 | 37.36 |  +1.53 |  +4.28 | 96.22 | 85.00 | -11.22 | -11.66 |
| TOON_DEFAULT | 27 | 27 | 0 | 0.00 | 56.90 | 37.00 | -19.90 | -34.97 | 38.13 | 37.05 | -1.08 | -2.84 | 38.16 | 37.07 | -1.08 | -2.83 | 95.03 | 74.05 | -20.98 | -22.08 |
| XML_COMPACT | 16 | 15 | -1 | -6.25 | 32.96 | 52.66 |  +19.71 |  +59.79 | 47.31 | 35.01 | -12.30 | -26.00 | 47.33 | 35.02 | -12.30 | -26.00 | 80.27 | 87.67 |  +7.40 |  +9.22 |
| XML_PRETTY | 18 | 9 | -9 | -50.00 | 50.05 | 38.04 | -12.01 | -23.99 | 36.39 | 36.23 | -0.16 | -0.45 | 36.41 | 36.24 | -0.17 | -0.47 | 86.44 | 74.28 | -12.17 | -14.08 |
| YAML | 11 | 8 | -3 | -27.27 | 41.08 | 65.99 |  +24.91 |  +60.64 | 34.80 | 37.99 |  +3.19 |  +9.15 | 34.82 | 38.00 |  +3.18 |  +9.14 | 75.88 | 103.98 |  +28.10 |  +37.03 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token | Info / Read Token (Acc By Char) | Info / Output Token (Acc By Char) | Info / Total Token (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 1.438 | 10.296 | 226.516 | 0.919 | 0.625 | 0.372 | 1.222 | 0.832 | 0.495 |
| CSV | opt | 1.423 | 10.662 | 217.032 | 0.940 | 0.577 | 0.357 | 1.277 | 0.784 | 0.486 |
| JSON_COMPACT | man | 2.143 | 13.630 | 299.871 | 0.818 | 0.564 | 0.334 | 0.989 | 0.682 | 0.404 |
| JSON_COMPACT | opt | 2.108 | 13.895 | 282.839 | 0.850 | 0.780 | 0.407 | 1.039 | 0.954 | 0.497 |
| JSON_PRETTY | man | 1.691 | 20.984 | 461.645 | 0.556 | 0.664 | 0.303 | 0.647 | 0.773 | 0.352 |
| JSON_PRETTY | opt | 1.677 | 21.228 | 432.097 | 0.571 | 0.734 | 0.321 | 0.687 | 0.884 | 0.387 |
| TOON_DEFAULT | man | 1.415 | 10.531 | 231.677 | 1.027 | 0.636 | 0.393 | 1.279 | 0.793 | 0.489 |
| TOON_DEFAULT | opt | 1.677 | 18.556 | 377.710 | 0.625 | 0.839 | 0.356 | 0.774 | 1.039 | 0.441 |
| XML_COMPACT | man | 2.315 | 17.522 | 385.484 | 0.594 | 1.017 | 0.375 | 0.765 | 1.311 | 0.483 |
| XML_COMPACT | opt | 2.288 | 17.719 | 360.677 | 0.683 | 0.678 | 0.340 | 0.809 | 0.803 | 0.403 |
| XML_PRETTY | man | 1.931 | 23.733 | 522.129 | 0.448 | 0.685 | 0.271 | 0.569 | 0.869 | 0.344 |
| XML_PRETTY | opt | 1.913 | 23.957 | 487.645 | 0.493 | 0.808 | 0.306 | 0.600 | 0.985 | 0.373 |
| YAML | man | 1.661 | 18.437 | 405.613 | 0.564 | 0.757 | 0.323 | 0.730 | 0.979 | 0.418 |
| YAML | opt | 1.646 | 18.686 | 380.355 | 0.663 | 0.600 | 0.315 | 0.779 | 0.705 | 0.370 |

#### 2.5.2 Characters And Values: Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.438 | 1.423 | -0.015 | -1.04 | 10.296 | 10.662 |  +0.366 |  +3.55 | 226.516 | 217.032 | -9.484 | -4.19 |
| JSON_COMPACT | 2.143 | 2.108 | -0.035 | -1.63 | 13.630 | 13.895 |  +0.265 |  +1.94 | 299.871 | 282.839 | -17.032 | -5.68 |
| JSON_PRETTY | 1.691 | 1.677 | -0.014 | -0.83 | 20.984 | 21.228 |  +0.244 |  +1.16 | 461.645 | 432.097 | -29.548 | -6.40 |
| TOON_DEFAULT | 1.415 | 1.677 |  +0.262 |  +18.52 | 10.531 | 18.556 |  +8.025 |  +76.20 | 231.677 | 377.710 |  +146.033 |  +63.03 |
| XML_COMPACT | 2.315 | 2.288 | -0.027 | -1.17 | 17.522 | 17.719 |  +0.197 |  +1.12 | 385.484 | 360.677 | -24.807 | -6.44 |
| XML_PRETTY | 1.931 | 1.913 | -0.018 | -0.93 | 23.733 | 23.957 |  +0.224 |  +0.94 | 522.129 | 487.645 | -34.484 | -6.60 |
| YAML | 1.661 | 1.646 | -0.015 | -0.90 | 18.437 | 18.686 |  +0.249 |  +1.35 | 405.613 | 380.355 | -25.258 | -6.23 |

#### 2.5.3 Information: Mandatory vs Optional
| Format | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 0.919 | 0.940 |  +0.021 |  +2.29 | 0.625 | 0.577 | -0.048 | -7.68 | 0.372 | 0.357 | -0.015 | -4.03 |
| JSON_COMPACT | 0.818 | 0.850 |  +0.032 |  +3.91 | 0.564 | 0.780 |  +0.216 |  +38.30 | 0.334 | 0.407 |  +0.073 |  +21.86 |
| JSON_PRETTY | 0.556 | 0.571 |  +0.015 |  +2.70 | 0.664 | 0.734 |  +0.070 |  +10.54 | 0.303 | 0.321 |  +0.018 |  +5.94 |
| TOON_DEFAULT | 1.027 | 0.625 | -0.402 | -39.14 | 0.636 | 0.839 |  +0.203 |  +31.92 | 0.393 | 0.356 | -0.037 | -9.54 |
| XML_COMPACT | 0.594 | 0.683 |  +0.089 |  +14.98 | 1.017 | 0.678 | -0.339 | -33.33 | 0.375 | 0.340 | -0.035 | -9.33 |
| XML_PRETTY | 0.448 | 0.493 |  +0.045 |  +10.04 | 0.685 | 0.808 |  +0.123 |  +17.96 | 0.271 | 0.306 |  +0.035 |  +12.92 |
| YAML | 0.564 | 0.663 |  +0.099 |  +17.55 | 0.757 | 0.600 | -0.157 | -20.74 | 0.323 | 0.315 | -0.008 | -2.48 |

#### 2.5.4 Information (Accuracy By Character): Mandatory vs Optional
| Format | Info / Read Token (Acc By Char) Man | Info / Read Token (Acc By Char) Opt | Diff | Diff (%) | Info / Output Token (Acc By Char) Man | Info / Output Token (Acc By Char)  Opt | Diff | Diff (%) | Info / Total Token (Acc By Char) Man | Info / Total Token (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.222 | 1.277 |  +0.055 |  +4.50 | 0.832 | 0.784 | -0.048 | -5.77 | 0.495 | 0.486 | -0.009 | -1.82 |
| JSON_COMPACT | 0.989 | 1.039 |  +0.050 |  +5.06 | 0.682 | 0.954 |  +0.272 |  +39.88 | 0.404 | 0.497 |  +0.093 |  +23.02 |
| JSON_PRETTY | 0.647 | 0.687 |  +0.040 |  +6.18 | 0.773 | 0.884 |  +0.111 |  +14.36 | 0.352 | 0.387 |  +0.035 |  +9.94 |
| TOON_DEFAULT | 1.279 | 0.774 | -0.505 | -39.48 | 0.793 | 1.039 |  +0.246 |  +31.02 | 0.489 | 0.441 | -0.048 | -9.92 |
| XML_COMPACT | 0.765 | 0.809 |  +0.044 |  +5.75 | 1.311 | 0.803 | -0.508 | -38.75 | 0.483 | 0.403 | -0.080 | -16.56 |
| XML_PRETTY | 0.569 | 0.600 |  +0.031 |  +5.45 | 0.869 | 0.985 |  +0.116 |  +13.35 | 0.344 | 0.373 |  +0.029 |  +8.43 |
| YAML | 0.730 | 0.779 |  +0.049 |  +6.71 | 0.979 | 0.705 | -0.274 | -27.99 | 0.418 | 0.370 | -0.048 | -11.48 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7022 | 4530 | 2492 | 10318 | 6656 | 3662 | 17340 | 11186 | 6154 | 64.51 | 75.27 | 59.23 | 76.30 | 63.26 | 74.44 | 58.40 | 75.47 |
| CSV | opt | 6728 | 4254 | 2474 | 10961 | 6931 | 4031 | 17689 | 11185 | 6504 | 63.23 | 75.45 | 55.10 | 74.22 | 62.47 | 74.94 | 54.59 | 73.71 |
| JSON_COMPACT | man | 9296 | 7066 | 2230 | 13489 | 10253 | 3236 | 22785 | 17318 | 5466 | 76.01 | 74.94 | 50.72 | 64.80 | 75.01 | 74.27 | 50.06 | 64.13 |
| JSON_COMPACT | opt | 8768 | 6534 | 2234 | 9558 | 7122 | 2435 | 18326 | 13656 | 4669 | 74.52 | 75.80 | 69.79 | 79.51 | 72.53 | 74.48 | 68.46 | 78.18 |
| JSON_PRETTY | man | 14311 | 11387 | 2924 | 11977 | 9530 | 2447 | 26288 | 20917 | 5371 | 79.57 | 59.67 | 60.81 | 54.84 | 78.12 | 58.71 | 59.84 | 53.87 |
| JSON_PRETTY | opt | 13395 | 10240 | 3155 | 10411 | 7959 | 2452 | 23806 | 18200 | 5606 | 76.45 | 60.82 | 66.72 | 61.50 | 75.12 | 59.93 | 65.83 | 60.61 |
| TOON_DEFAULT | man | 7182 | 5297 | 1885 | 11615 | 8566 | 3049 | 18797 | 13863 | 4934 | 73.75 | 80.87 | 58.78 | 77.33 | 72.42 | 79.98 | 57.89 | 76.45 |
| TOON_DEFAULT | opt | 11709 | 8572 | 3137 | 8984 | 6577 | 2407 | 20693 | 15149 | 5544 | 73.21 | 64.59 | 71.84 | 70.30 | 71.63 | 63.53 | 70.79 | 69.24 |
| XML_COMPACT | man | 11950 | 8481 | 3469 | 6976 | 4951 | 2025 | 18926 | 13432 | 5494 | 70.97 | 62.25 | 80.59 | 75.03 | 68.79 | 60.79 | 79.14 | 73.57 |
| XML_COMPACT | opt | 11181 | 8536 | 2645 | 11266 | 8600 | 2665 | 22447 | 17136 | 5311 | 76.34 | 68.53 | 62.29 | 66.21 | 75.53 | 67.99 | 61.75 | 65.67 |
| XML_PRETTY | man | 16186 | 11748 | 4438 | 10600 | 7694 | 2907 | 26786 | 19442 | 7345 | 72.58 | 48.42 | 63.17 | 48.42 | 70.21 | 46.84 | 61.59 | 46.84 |
| XML_PRETTY | opt | 15117 | 11256 | 3861 | 9210 | 6858 | 2352 | 24327 | 18114 | 6213 | 74.46 | 53.43 | 71.52 | 58.34 | 73.40 | 52.73 | 70.82 | 57.63 |
| YAML | man | 12574 | 8924 | 3650 | 9373 | 6652 | 2721 | 21947 | 15576 | 6371 | 70.97 | 60.05 | 68.37 | 64.39 | 67.65 | 57.84 | 66.15 | 62.18 |
| YAML | opt | 11791 | 9224 | 2567 | 13029 | 10193 | 2836 | 24820 | 19417 | 5403 | 78.23 | 67.64 | 54.55 | 59.11 | 76.96 | 66.80 | 53.70 | 58.27 |

#### 2.6.2 Read Tokens: Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7022 | 6728 | -294 | -4.19 | 4530 | 4254 | -276 | -6.09 | 2492 | 2474 | -18 | -0.73 | 64.51 | 63.23 | -1.28 | -1.98 | 75.27 | 75.45 |  +0.18 |  +0.24 | 63.26 | 62.47 | -0.79 | -1.25 | 74.44 | 74.94 |  +0.51 |  +0.68 |
| JSON_COMPACT | 9296 | 8768 | -528 | -5.68 | 7066 | 6534 | -532 | -7.53 | 2230 | 2234 |  +4 |  +0.18 | 76.01 | 74.52 | -1.49 | -1.96 | 74.94 | 75.80 |  +0.86 |  +1.15 | 75.01 | 72.53 | -2.48 | -3.31 | 74.27 | 74.48 |  +0.20 |  +0.27 |
| JSON_PRETTY | 14311 | 13395 | -916 | -6.40 | 11387 | 10240 | -1147 | -10.07 | 2924 | 3155 |  +231 |  +7.89 | 79.57 | 76.45 | -3.12 | -3.92 | 59.67 | 60.82 |  +1.14 |  +1.91 | 78.12 | 75.12 | -3.00 | -3.84 | 58.71 | 59.93 |  +1.22 |  +2.08 |
| TOON_DEFAULT | 7182 | 11709 |  +4527 |  +63.03 | 5297 | 8572 |  +3275 |  +61.84 | 1885 | 3137 |  +1252 |  +66.40 | 73.75 | 73.21 | -0.54 | -0.73 | 80.87 | 64.59 | -16.28 | -20.13 | 72.42 | 71.63 | -0.79 | -1.09 | 79.98 | 63.53 | -16.45 | -20.56 |
| XML_COMPACT | 11950 | 11181 | -769 | -6.44 | 8481 | 8536 |  +55 |  +0.64 | 3469 | 2645 | -824 | -23.74 | 70.97 | 76.34 |  +5.37 |  +7.57 | 62.25 | 68.53 |  +6.28 |  +10.10 | 68.79 | 75.53 |  +6.74 |  +9.80 | 60.79 | 67.99 |  +7.20 |  +11.84 |
| XML_PRETTY | 16186 | 15117 | -1069 | -6.60 | 11748 | 11256 | -492 | -4.19 | 4438 | 3861 | -577 | -13.01 | 72.58 | 74.46 |  +1.88 |  +2.59 | 48.42 | 53.43 |  +5.01 |  +10.35 | 70.21 | 73.40 |  +3.19 |  +4.54 | 46.84 | 52.73 |  +5.89 |  +12.57 |
| YAML | 12574 | 11791 | -783 | -6.23 | 8924 | 9224 |  +300 |  +3.37 | 3650 | 2567 | -1083 | -29.68 | 70.97 | 78.23 |  +7.26 |  +10.23 | 60.05 | 67.64 |  +7.59 |  +12.65 | 67.65 | 76.96 |  +9.31 |  +13.76 | 57.84 | 66.80 |  +8.96 |  +15.49 |

#### 2.6.3 Output Tokens: Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 10318 | 10961 |  +643 |  +6.23 | 6656 | 6931 |  +275 |  +4.13 | 3662 | 4031 |  +369 |  +10.06 | 64.51 | 63.23 | -1.28 | -1.98 | 59.23 | 55.10 | -4.14 | -6.98 | 63.26 | 62.47 | -0.79 | -1.25 | 58.40 | 54.59 | -3.81 | -6.52 |
| JSON_COMPACT | 13489 | 9558 | -3931 | -29.14 | 10253 | 7123 | -3130 | -30.53 | 3236 | 2435 | -801 | -24.74 | 76.01 | 74.52 | -1.49 | -1.96 | 50.72 | 69.79 |  +19.06 |  +37.58 | 75.01 | 72.53 | -2.48 | -3.31 | 50.06 | 68.46 |  +18.40 |  +36.77 |
| JSON_PRETTY | 11977 | 10411 | -1566 | -13.07 | 9530 | 7959 | -1571 | -16.48 | 2447 | 2452 |  +5 |  +0.20 | 79.57 | 76.45 | -3.12 | -3.92 | 60.81 | 66.72 |  +5.91 |  +9.72 | 78.12 | 75.12 | -3.00 | -3.84 | 59.84 | 65.83 |  +5.99 |  +10.01 |
| TOON_DEFAULT | 11615 | 8984 | -2631 | -22.65 | 8566 | 6577 | -1989 | -23.22 | 3049 | 2407 | -642 | -21.06 | 73.75 | 73.21 | -0.54 | -0.73 | 58.78 | 71.84 |  +13.06 |  +22.23 | 72.42 | 71.63 | -0.79 | -1.09 | 57.89 | 70.79 |  +12.90 |  +22.28 |
| XML_COMPACT | 6976 | 11266 |  +4290 |  +61.49 | 4951 | 8600 |  +3649 |  +73.71 | 2025 | 2665 |  +640 |  +31.62 | 70.97 | 76.34 |  +5.37 |  +7.57 | 80.59 | 62.29 | -18.31 | -22.72 | 68.79 | 75.53 |  +6.74 |  +9.80 | 79.14 | 61.75 | -17.40 | -21.98 |
| XML_PRETTY | 10600 | 9209 | -1391 | -13.12 | 7694 | 6858 | -836 | -10.87 | 2907 | 2353 | -554 | -19.07 | 72.58 | 74.46 |  +1.88 |  +2.59 | 63.17 | 71.52 |  +8.35 |  +13.22 | 70.21 | 73.40 |  +3.19 |  +4.54 | 61.59 | 70.82 |  +9.22 |  +14.97 |
| YAML | 9373 | 13029 |  +3656 |  +39.01 | 6652 | 10193 |  +3541 |  +53.23 | 2721 | 2837 |  +116 |  +4.25 | 70.97 | 78.23 |  +7.26 |  +10.23 | 68.37 | 54.55 | -13.82 | -20.21 | 67.65 | 76.96 |  +9.31 |  +13.76 | 66.15 | 53.70 | -12.45 | -18.82 |

#### 2.6.4 Total Tokens: Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 17340 | 17689 |  +349 |  +2.01 | 11186 | 11185 | -1 | -0.01 | 6154 | 6504 |  +350 |  +5.69 | 64.51 | 63.23 | -1.28 | -1.98 | 76.30 | 74.22 | -2.08 | -2.73 | 63.26 | 62.47 | -0.79 | -1.25 | 75.47 | 73.71 | -1.76 | -2.33 |
| JSON_COMPACT | 22785 | 18326 | -4459 | -19.57 | 17318 | 13656 | -3662 | -21.15 | 5466 | 4669 | -797 | -14.57 | 76.01 | 74.52 | -1.49 | -1.96 | 64.80 | 79.51 |  +14.71 |  +22.70 | 75.01 | 72.53 | -2.48 | -3.31 | 64.13 | 78.18 |  +14.05 |  +21.90 |
| JSON_PRETTY | 26288 | 23806 | -2482 | -9.44 | 20917 | 18200 | -2717 | -12.99 | 5371 | 5607 |  +236 |  +4.39 | 79.57 | 76.45 | -3.12 | -3.92 | 54.84 | 61.50 |  +6.66 |  +12.14 | 78.12 | 75.12 | -3.00 | -3.84 | 53.87 | 60.61 |  +6.74 |  +12.51 |
| TOON_DEFAULT | 18797 | 20693 |  +1896 |  +10.09 | 13863 | 15150 |  +1287 |  +9.28 | 4934 | 5543 |  +609 |  +12.35 | 73.75 | 73.21 | -0.54 | -0.73 | 77.33 | 70.30 | -7.04 | -9.10 | 72.42 | 71.63 | -0.79 | -1.09 | 76.45 | 69.24 | -7.20 | -9.42 |
| XML_COMPACT | 18926 | 22447 |  +3521 |  +18.60 | 13432 | 17136 |  +3704 |  +27.58 | 5494 | 5311 | -183 | -3.34 | 70.97 | 76.34 |  +5.37 |  +7.57 | 75.03 | 66.21 | -8.82 | -11.75 | 68.79 | 75.53 |  +6.74 |  +9.80 | 73.57 | 65.67 | -7.90 | -10.74 |
| XML_PRETTY | 26786 | 24326 | -2460 | -9.18 | 19442 | 18114 | -1328 | -6.83 | 7345 | 6213 | -1132 | -15.41 | 72.58 | 74.46 |  +1.88 |  +2.59 | 48.42 | 58.34 |  +9.91 |  +20.48 | 70.21 | 73.40 |  +3.19 |  +4.54 | 46.84 | 57.63 |  +10.79 |  +23.03 |
| YAML | 21947 | 24820 |  +2873 |  +13.09 | 15576 | 19417 |  +3841 |  +24.66 | 6371 | 5403 | -968 | -15.19 | 70.97 | 78.23 |  +7.26 |  +10.23 | 64.39 | 59.11 | -5.28 | -8.20 | 67.65 | 76.96 |  +9.31 |  +13.76 | 62.18 | 58.27 | -3.91 | -6.29 |

### 2.7 Token Utilization Efficiency (Accuracy by Character)
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy by Character (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy by Character (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7022 | 6026 | 996 | 10318 | 8854 | 1464 | 17340 | 14880 | 2461 | 85.81 | 89.47 | 73.43 | 90.50 | 86.58 | 89.98 | 73.95 | 91.02 |
| CSV | opt | 6728 | 5781 | 947 | 10961 | 9419 | 1542 | 17689 | 15201 | 2489 | 85.93 | 90.58 | 70.23 | 89.35 | 87.15 | 91.40 | 71.05 | 90.17 |
| JSON_COMPACT | man | 9296 | 8550 | 746 | 13489 | 12407 | 1082 | 22785 | 20957 | 1827 | 91.98 | 85.59 | 61.37 | 75.45 | 92.17 | 85.71 | 61.50 | 75.57 |
| JSON_COMPACT | opt | 8768 | 7991 | 777 | 9558 | 8711 | 847 | 18326 | 16702 | 1624 | 91.14 | 86.88 | 80.87 | 90.59 | 91.05 | 86.82 | 80.81 | 90.53 |
| JSON_PRETTY | man | 14311 | 13248 | 1063 | 11977 | 11087 | 890 | 26288 | 24334 | 1953 | 92.57 | 68.34 | 69.48 | 63.50 | 92.50 | 68.30 | 69.43 | 63.46 |
| JSON_PRETTY | opt | 13395 | 12333 | 1062 | 10411 | 9585 | 826 | 23806 | 21918 | 1888 | 92.07 | 71.23 | 77.13 | 71.91 | 92.20 | 71.32 | 77.22 | 72.00 |
| TOON_DEFAULT | man | 7182 | 6598 | 584 | 11615 | 10671 | 944 | 18797 | 17269 | 1528 | 91.87 | 92.95 | 70.86 | 89.41 | 92.06 | 93.07 | 70.98 | 89.54 |
| TOON_DEFAULT | opt | 11709 | 10608 | 1101 | 8984 | 8140 | 845 | 20693 | 18748 | 1945 | 90.60 | 76.18 | 83.43 | 81.89 | 90.61 | 76.19 | 83.44 | 81.90 |
| XML_COMPACT | man | 11950 | 10929 | 1021 | 6976 | 6380 | 596 | 18926 | 17310 | 1616 | 91.46 | 75.91 | 94.25 | 88.69 | 91.26 | 75.77 | 94.12 | 88.55 |
| XML_COMPACT | opt | 11181 | 10109 | 1072 | 11266 | 10185 | 1080 | 22447 | 20294 | 2153 | 90.41 | 77.91 | 71.67 | 75.59 | 90.66 | 78.08 | 71.83 | 75.76 |
| XML_PRETTY | man | 16186 | 14914 | 1272 | 10600 | 9767 | 833 | 26786 | 24681 | 2105 | 92.14 | 61.46 | 76.21 | 61.46 | 92.08 | 61.42 | 76.17 | 61.42 |
| XML_PRETTY | opt | 15117 | 13708 | 1409 | 9210 | 8351 | 858 | 24327 | 22059 | 2267 | 90.68 | 64.25 | 82.34 | 69.15 | 90.77 | 64.31 | 82.40 | 69.21 |
| YAML | man | 12574 | 11535 | 1039 | 9373 | 8598 | 774 | 21947 | 20134 | 1813 | 91.74 | 73.90 | 82.21 | 78.24 | 91.50 | 73.74 | 82.05 | 78.08 |
| YAML | opt | 11791 | 10836 | 955 | 13029 | 11974 | 1055 | 24820 | 22810 | 2010 | 91.90 | 76.76 | 63.66 | 68.22 | 92.15 | 76.92 | 63.83 | 68.39 |

#### 2.7.2 Read Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7022 | 6728 | -294 | -4.19 | 6026 | 5782 | -244 | -4.05 | 996 | 946 | -50 | -5.00 | 85.81 | 85.93 |  +0.12 |  +0.14 | 89.47 | 90.58 |  +1.11 |  +1.25 | 86.58 | 87.15 |  +0.57 |  +0.66 | 89.98 | 91.40 |  +1.41 |  +1.57 |
| JSON_COMPACT | 9296 | 8768 | -528 | -5.68 | 8550 | 7991 | -559 | -6.54 | 746 | 777 |  +31 |  +4.20 | 91.98 | 91.14 | -0.84 | -0.91 | 85.59 | 86.88 |  +1.30 |  +1.52 | 92.17 | 91.05 | -1.12 | -1.22 | 85.71 | 86.82 |  +1.11 |  +1.30 |
| JSON_PRETTY | 14311 | 13395 | -916 | -6.40 | 13248 | 12333 | -915 | -6.91 | 1063 | 1062 | -1 | -0.10 | 92.57 | 92.07 | -0.50 | -0.54 | 68.34 | 71.23 |  +2.89 |  +4.23 | 92.50 | 92.20 | -0.30 | -0.32 | 68.30 | 71.32 |  +3.02 |  +4.42 |
| TOON_DEFAULT | 7182 | 11709 |  +4527 |  +63.03 | 6598 | 10608 |  +4010 |  +60.78 | 584 | 1101 |  +517 |  +88.48 | 91.87 | 90.60 | -1.27 | -1.38 | 92.95 | 76.18 | -16.77 | -18.04 | 92.06 | 90.61 | -1.45 | -1.58 | 93.07 | 76.19 | -16.89 | -18.14 |
| XML_COMPACT | 11950 | 11181 | -769 | -6.44 | 10929 | 10108 | -821 | -7.51 | 1021 | 1073 |  +52 |  +5.07 | 91.46 | 90.41 | -1.05 | -1.15 | 75.91 | 77.91 |  +2.00 |  +2.64 | 91.26 | 90.66 | -0.60 | -0.66 | 75.77 | 78.08 |  +2.30 |  +3.04 |
| XML_PRETTY | 16186 | 15117 | -1069 | -6.60 | 14914 | 13708 | -1206 | -8.08 | 1272 | 1409 |  +137 |  +10.75 | 92.14 | 90.68 | -1.46 | -1.58 | 61.46 | 64.25 |  +2.79 |  +4.53 | 92.08 | 90.77 | -1.31 | -1.42 | 61.42 | 64.31 |  +2.89 |  +4.70 |
| YAML | 12574 | 11791 | -783 | -6.23 | 11535 | 10836 | -699 | -6.06 | 1039 | 955 | -84 | -8.04 | 91.74 | 91.90 |  +0.16 |  +0.17 | 73.90 | 76.76 |  +2.86 |  +3.87 | 91.50 | 92.15 |  +0.65 |  +0.71 | 73.74 | 76.92 |  +3.19 |  +4.32 |

#### 2.7.3 Output Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 10318 | 10961 |  +643 |  +6.23 | 8854 | 9419 |  +565 |  +6.38 | 1464 | 1542 |  +78 |  +5.34 | 85.81 | 85.93 |  +0.12 |  +0.14 | 73.43 | 70.23 | -3.20 | -4.36 | 86.58 | 87.15 |  +0.57 |  +0.66 | 73.95 | 71.05 | -2.90 | -3.92 |
| JSON_COMPACT | 13489 | 9558 | -3931 | -29.14 | 12407 | 8711 | -3696 | -29.79 | 1082 | 847 | -235 | -21.72 | 91.98 | 91.14 | -0.84 | -0.91 | 61.37 | 80.87 |  +19.50 |  +31.77 | 92.17 | 91.05 | -1.12 | -1.22 | 61.50 | 80.81 |  +19.31 |  +31.40 |
| JSON_PRETTY | 11977 | 10411 | -1566 | -13.07 | 11087 | 9586 | -1501 | -13.54 | 890 | 826 | -64 | -7.22 | 92.57 | 92.07 | -0.50 | -0.54 | 69.48 | 77.13 |  +7.66 |  +11.02 | 92.50 | 92.20 | -0.30 | -0.32 | 69.43 | 77.22 |  +7.79 |  +11.22 |
| TOON_DEFAULT | 11615 | 8984 | -2631 | -22.65 | 10671 | 8140 | -2531 | -23.72 | 944 | 844 | -100 | -10.57 | 91.87 | 90.60 | -1.27 | -1.38 | 70.86 | 83.43 |  +12.58 |  +17.75 | 92.06 | 90.61 | -1.45 | -1.58 | 70.98 | 83.44 |  +12.46 |  +17.55 |
| XML_COMPACT | 6976 | 11266 |  +4290 |  +61.49 | 6380 | 10185 |  +3805 |  +59.64 | 596 | 1081 |  +485 |  +81.31 | 91.46 | 90.41 | -1.05 | -1.15 | 94.25 | 71.67 | -22.59 | -23.97 | 91.26 | 90.66 | -0.60 | -0.66 | 94.12 | 71.83 | -22.29 | -23.68 |
| XML_PRETTY | 10600 | 9209 | -1391 | -13.12 | 9767 | 8351 | -1416 | -14.50 | 833 | 858 |  +25 |  +3.02 | 92.14 | 90.68 | -1.46 | -1.58 | 76.21 | 82.34 |  +6.12 |  +8.03 | 92.08 | 90.77 | -1.31 | -1.42 | 76.17 | 82.40 |  +6.22 |  +8.17 |
| YAML | 9373 | 13029 |  +3656 |  +39.01 | 8598 | 11973 |  +3375 |  +39.26 | 774 | 1055 |  +281 |  +36.33 | 91.74 | 91.90 |  +0.16 |  +0.17 | 82.21 | 63.66 | -18.55 | -22.56 | 91.50 | 92.15 |  +0.65 |  +0.71 | 82.05 | 63.83 | -18.22 | -22.21 |

#### 2.7.4 Total Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 17340 | 17689 |  +349 |  +2.01 | 14880 | 15201 |  +321 |  +2.16 | 2461 | 2489 |  +28 |  +1.15 | 85.81 | 85.93 |  +0.12 |  +0.14 | 90.50 | 89.35 | -1.15 | -1.27 | 86.58 | 87.15 |  +0.57 |  +0.66 | 91.02 | 90.17 | -0.85 | -0.93 |
| JSON_COMPACT | 22785 | 18326 | -4459 | -19.57 | 20957 | 16702 | -4255 | -20.30 | 1827 | 1623 | -204 | -11.15 | 91.98 | 91.14 | -0.84 | -0.91 | 75.45 | 90.59 |  +15.14 |  +20.07 | 92.17 | 91.05 | -1.12 | -1.22 | 75.57 | 90.53 |  +14.95 |  +19.79 |
| JSON_PRETTY | 26288 | 23806 | -2482 | -9.44 | 24334 | 21918 | -2416 | -9.93 | 1953 | 1888 | -65 | -3.35 | 92.57 | 92.07 | -0.50 | -0.54 | 63.50 | 71.91 |  +8.41 |  +13.24 | 92.50 | 92.20 | -0.30 | -0.32 | 63.46 | 72.00 |  +8.54 |  +13.46 |
| TOON_DEFAULT | 18797 | 20693 |  +1896 |  +10.09 | 17269 | 18748 |  +1479 |  +8.57 | 1528 | 1945 |  +417 |  +27.29 | 91.87 | 90.60 | -1.27 | -1.38 | 89.41 | 81.89 | -7.52 | -8.41 | 92.06 | 90.61 | -1.45 | -1.58 | 89.54 | 81.90 | -7.64 | -8.54 |
| XML_COMPACT | 18926 | 22447 |  +3521 |  +18.60 | 17310 | 20294 |  +2984 |  +17.24 | 1616 | 2152 |  +536 |  +33.19 | 91.46 | 90.41 | -1.05 | -1.15 | 88.69 | 75.59 | -13.10 | -14.77 | 91.26 | 90.66 | -0.60 | -0.66 | 88.55 | 75.76 | -12.80 | -14.45 |
| XML_PRETTY | 26786 | 24326 | -2460 | -9.18 | 24681 | 22059 | -2622 | -10.62 | 2105 | 2267 |  +162 |  +7.69 | 92.14 | 90.68 | -1.46 | -1.58 | 61.46 | 69.15 |  +7.69 |  +12.51 | 92.08 | 90.77 | -1.31 | -1.42 | 61.42 | 69.21 |  +7.79 |  +12.68 |
| YAML | 21947 | 24820 |  +2873 |  +13.09 | 20134 | 22810 |  +2676 |  +13.29 | 1813 | 2011 |  +198 |  +10.90 | 91.74 | 91.90 |  +0.16 |  +0.17 | 78.24 | 68.22 | -10.01 | -12.80 | 91.50 | 92.15 |  +0.65 |  +0.71 | 78.08 | 68.39 | -9.68 | -12.40 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Expected Characters | Output Characters | Correct Characters | Incorrect Characters | Accuracy by Character (%) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 80.00 | 44.00 | 0.00 | 64.51 | 7777.00 | 8311.40 | 7128.40 | 1183.00 | 85.81 |
| CSV | opt | 78.40 | 45.60 | -0.00 | 63.23 | 8441.00 | 9043.40 | 7758.20 | 1285.20 | 85.93 |
| JSON_COMPACT | man | 94.25 | 29.75 | 0.00 | 76.01 | 7777.00 | 7828.75 | 7628.25 | 200.50 | 91.98 |
| JSON_COMPACT | opt | 92.40 | 31.60 | -0.00 | 74.52 | 8441.00 | 8577.00 | 8242.80 | 334.20 | 91.14 |
| JSON_PRETTY | man | 98.67 | 25.33 | 0.00 | 79.57 | 7777.00 | 7842.67 | 7601.00 | 241.67 | 92.57 |
| JSON_PRETTY | opt | 94.80 | 29.20 | 0.00 | 76.45 | 8441.00 | 8567.40 | 8319.80 | 247.60 | 92.07 |
| TOON_DEFAULT | man | 91.44 | 32.56 | 0.00 | 73.75 | 7777.00 | 8148.67 | 7523.00 | 625.67 | 91.87 |
| TOON_DEFAULT | opt | 90.78 | 33.22 | 0.00 | 73.21 | 8441.00 | 8646.22 | 8222.11 | 424.11 | 90.60 |
| XML_COMPACT | man | 88.00 | 36.00 | 0.00 | 70.97 | 7777.00 | 7981.67 | 7534.33 | 447.33 | 91.46 |
| XML_COMPACT | opt | 94.67 | 29.33 | 0.00 | 76.34 | 8441.00 | 8524.00 | 8331.33 | 192.67 | 90.41 |
| XML_PRETTY | man | 90.00 | 34.00 | 0.00 | 72.58 | 7777.00 | 7906.00 | 7548.67 | 357.33 | 92.14 |
| XML_PRETTY | opt | 92.33 | 31.67 | 0.00 | 74.46 | 8441.00 | 8629.33 | 8241.00 | 388.33 | 90.68 |
| YAML | man | 88.00 | 36.00 | 0.00 | 70.97 | 7777.00 | 7920.67 | 7614.00 | 306.67 | 91.74 |
| YAML | opt | 97.00 | 27.00 | 0.00 | 78.23 | 8441.00 | 8482.33 | 8337.00 | 145.33 | 91.90 |

#### 2.8.2 Answers: Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 80.00 | 78.40 | -2 | -2.00 | 44.00 | 45.60 |  +2 |  +3.64 | 0.00 | -0.00 | -0 | 0.00 | 64.51 | 63.23 | -1.28 |
| JSON_COMPACT | 94.25 | 92.40 | -2 | -1.96 | 29.75 | 31.60 |  +2 |  +6.22 | 0.00 | -0.00 | -0 | 0.00 | 76.01 | 74.52 | -1.49 |
| JSON_PRETTY | 98.67 | 94.80 | -4 | -3.92 | 25.33 | 29.20 |  +4 |  +15.28 | 0.00 | 0.00 |  +0 | 0.00 | 79.57 | 76.45 | -3.12 |
| TOON_DEFAULT | 91.44 | 90.78 | -1 | -0.72 | 32.56 | 33.22 |  +1 |  +2.03 | 0.00 | 0.00 | 0 | 0.00 | 73.75 | 73.21 | -0.54 |
| XML_COMPACT | 88.00 | 94.67 |  +7 |  +7.58 | 36.00 | 29.33 | -7 | -18.53 | 0.00 | 0.00 | 0 | 0.00 | 70.97 | 76.34 |  +5.37 |
| XML_PRETTY | 90.00 | 92.33 |  +2 |  +2.59 | 34.00 | 31.67 | -2 | -6.85 | 0.00 | 0.00 | 0 | 0.00 | 72.58 | 74.46 |  +1.88 |
| YAML | 88.00 | 97.00 |  +9 |  +10.23 | 36.00 | 27.00 | -9 | -25.00 | 0.00 | 0.00 | 0 | 0.00 | 70.97 | 78.23 |  +7.26 |

#### 2.8.3 Characters: Mandatory vs Optional Data
| Format | Output Characters Man | Output Characters Opt | Diff | Diff (%) | Correct Characters Man | Correct Characters Opt | Diff | Diff (%) | Incorrect Characters Man | Incorrect Characters Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 8311.40 | 9043.40 |  +732 |  +8.81 | 7128.40 | 7758.20 |  +630 |  +8.84 | 1183.00 | 1285.20 |  +102 |  +8.64 | 85.81 | 85.93 |  +0.12 |
| JSON_COMPACT | 7828.75 | 8577.00 |  +748 |  +9.56 | 7628.25 | 8242.80 |  +615 |  +8.06 | 200.50 | 334.20 |  +134 |  +66.68 | 91.98 | 91.14 | -0.84 |
| JSON_PRETTY | 7842.67 | 8567.40 |  +725 |  +9.24 | 7601.00 | 8319.80 |  +719 |  +9.46 | 241.67 | 247.60 |  +6 |  +2.46 | 92.57 | 92.07 | -0.50 |
| TOON_DEFAULT | 8148.67 | 8646.22 |  +498 |  +6.11 | 7523.00 | 8222.11 |  +699 |  +9.29 | 625.67 | 424.11 | -202 | -32.21 | 91.87 | 90.60 | -1.27 |
| XML_COMPACT | 7981.67 | 8524.00 |  +542 |  +6.79 | 7534.33 | 8331.33 |  +797 |  +10.58 | 447.33 | 192.67 | -255 | -56.93 | 91.46 | 90.41 | -1.05 |
| XML_PRETTY | 7906.00 | 8629.33 |  +723 |  +9.15 | 7548.67 | 8241.00 |  +692 |  +9.17 | 357.33 | 388.33 |  +31 |  +8.68 | 92.14 | 90.68 | -1.46 |
| YAML | 7920.67 | 8482.33 |  +562 |  +7.09 | 7614.00 | 8337.00 |  +723 |  +9.50 | 306.67 | 145.33 | -161 | -52.61 | 91.74 | 91.90 |  +0.16 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 64.51 | 75.64 | 52.59 | 61.90 | 53.33 | 86.58 | 28.36 | 15.34 | 12.89 | 6.67 |
| CSV | opt | 63.23 | 79.27 | 51.85 | 62.86 | 36.19 | 87.15 | 29.73 | 15.12 | 13.09 | 4.52 |
| JSON_COMPACT | man | 76.01 | 98.18 | 68.52 | 63.09 | 40.48 | 92.17 | 36.82 | 19.98 | 13.14 | 5.06 |
| JSON_COMPACT | opt | 74.52 | 94.91 | 60.00 | 61.90 | 52.38 | 91.05 | 35.59 | 17.50 | 12.90 | 6.55 |
| JSON_PRETTY | man | 79.57 | 96.97 | 70.37 | 66.67 | 58.73 | 92.50 | 36.36 | 20.52 | 13.89 | 7.34 |
| JSON_PRETTY | opt | 76.45 | 96.73 | 65.19 | 66.67 | 47.62 | 92.20 | 36.27 | 19.01 | 13.89 | 5.95 |
| TOON_DEFAULT | man | 73.75 | 92.32 | 65.43 | 60.32 | 49.21 | 92.06 | 34.62 | 19.08 | 12.57 | 6.15 |
| TOON_DEFAULT | opt | 73.21 | 93.33 | 61.73 | 60.84 | 47.62 | 90.61 | 35.00 | 18.00 | 12.68 | 5.95 |
| XML_COMPACT | man | 70.97 | 88.49 | 56.79 | 57.14 | 57.14 | 91.26 | 33.18 | 16.56 | 11.91 | 7.14 |
| XML_COMPACT | opt | 76.34 | 97.58 | 71.61 | 61.90 | 41.27 | 90.66 | 36.59 | 20.88 | 12.90 | 5.16 |
| XML_PRETTY | man | 72.58 | 95.76 | 51.85 | 63.49 | 47.62 | 92.08 | 35.91 | 15.12 | 13.23 | 5.95 |
| XML_PRETTY | opt | 74.46 | 90.91 | 70.37 | 58.73 | 52.38 | 90.77 | 34.09 | 20.52 | 12.23 | 6.55 |
| YAML | man | 70.97 | 96.97 | 41.97 | 61.90 | 49.21 | 91.50 | 36.36 | 12.24 | 12.89 | 6.15 |
| YAML | opt | 78.23 | 99.39 | 64.20 | 73.02 | 46.03 | 92.15 | 37.27 | 18.72 | 15.21 | 5.75 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 75.64 | 79.27 |  +3.64 | 28.36 | 29.73 |  +1.36 |
| JSON_COMPACT | 98.18 | 94.91 | -3.27 | 36.82 | 35.59 | -1.23 |
| JSON_PRETTY | 96.97 | 96.73 | -0.24 | 36.36 | 36.27 | -0.09 |
| TOON_DEFAULT | 92.32 | 93.33 |  +1.01 | 34.62 | 35.00 |  +0.38 |
| XML_COMPACT | 88.49 | 97.58 |  +9.09 | 33.18 | 36.59 |  +3.41 |
| XML_PRETTY | 95.76 | 90.91 | -4.85 | 35.91 | 34.09 | -1.82 |
| YAML | 96.97 | 99.39 |  +2.42 | 36.36 | 37.27 |  +0.91 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 52.59 | 51.85 | -0.74 | 15.34 | 15.12 | -0.22 |
| JSON_COMPACT | 68.52 | 60.00 | -8.52 | 19.98 | 17.50 | -2.48 |
| JSON_PRETTY | 70.37 | 65.19 | -5.18 | 20.52 | 19.01 | -1.51 |
| TOON_DEFAULT | 65.43 | 61.73 | -3.70 | 19.08 | 18.00 | -1.08 |
| XML_COMPACT | 56.79 | 71.61 |  +14.82 | 16.56 | 20.88 |  +4.32 |
| XML_PRETTY | 51.85 | 70.37 |  +18.52 | 15.12 | 20.52 |  +5.40 |
| YAML | 41.97 | 64.20 |  +22.23 | 12.24 | 18.72 |  +6.48 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 61.90 | 62.86 |  +0.95 | 12.89 | 13.09 |  +0.20 |
| JSON_COMPACT | 63.09 | 61.90 | -1.19 | 13.14 | 12.90 | -0.25 |
| JSON_PRETTY | 66.67 | 66.67 | -0.00 | 13.89 | 13.89 | -0.00 |
| TOON_DEFAULT | 60.32 | 60.84 |  +0.53 | 12.57 | 12.68 |  +0.11 |
| XML_COMPACT | 57.14 | 61.90 |  +4.76 | 11.91 | 12.90 |  +0.99 |
| XML_PRETTY | 63.49 | 58.73 | -4.77 | 13.23 | 12.23 | -0.99 |
| YAML | 61.90 | 73.02 |  +11.11 | 12.89 | 15.21 |  +2.32 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 53.33 | 36.19 | -17.14 | 6.67 | 4.52 | -2.14 |
| JSON_COMPACT | 40.48 | 52.38 |  +11.90 | 5.06 | 6.55 |  +1.49 |
| JSON_PRETTY | 58.73 | 47.62 | -11.11 | 7.34 | 5.95 | -1.39 |
| TOON_DEFAULT | 49.21 | 47.62 | -1.59 | 6.15 | 5.95 | -0.20 |
| XML_COMPACT | 57.14 | 41.27 | -15.87 | 7.14 | 5.16 | -1.98 |
| XML_PRETTY | 47.62 | 52.38 |  +4.76 | 5.95 | 6.55 |  +0.60 |
| YAML | 49.21 | 46.03 | -3.17 | 6.15 | 5.75 | -0.40 |

### 2.10 Accuracy By Character Per Question Category Analysis
#### 2.10.1 Metrics
| Format | Variant | Accuracy By Character (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc By Char (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 85.81 | 85.63 | 88.33 | 91.24 | 77.59 | 86.58 | 32.11 | 25.76 | 19.01 | 9.70 |
| CSV | opt | 85.93 | 87.85 | 90.44 | 92.35 | 68.67 | 87.15 | 32.94 | 26.38 | 19.24 | 8.58 |
| JSON_COMPACT | man | 91.98 | 98.88 | 91.15 | 91.71 | 75.27 | 92.17 | 37.08 | 26.59 | 19.10 | 9.41 |
| JSON_COMPACT | opt | 91.14 | 98.60 | 88.41 | 90.13 | 76.12 | 91.05 | 36.98 | 25.79 | 18.78 | 9.51 |
| JSON_PRETTY | man | 92.57 | 98.62 | 90.38 | 91.75 | 80.38 | 92.50 | 36.98 | 26.36 | 19.11 | 10.05 |
| JSON_PRETTY | opt | 92.07 | 99.03 | 90.25 | 92.44 | 75.83 | 92.20 | 37.14 | 26.32 | 19.26 | 9.48 |
| TOON_DEFAULT | man | 91.87 | 97.03 | 90.98 | 92.57 | 78.80 | 92.06 | 36.39 | 26.54 | 19.28 | 9.85 |
| TOON_DEFAULT | opt | 90.60 | 97.69 | 88.38 | 90.23 | 75.23 | 90.61 | 36.63 | 25.78 | 18.80 | 9.40 |
| XML_COMPACT | man | 91.46 | 96.03 | 89.74 | 89.21 | 84.00 | 91.26 | 36.01 | 26.17 | 18.58 | 10.50 |
| XML_COMPACT | opt | 90.41 | 99.64 | 90.99 | 87.20 | 68.67 | 90.66 | 37.37 | 26.54 | 18.16 | 8.59 |
| XML_PRETTY | man | 92.14 | 98.43 | 88.53 | 93.39 | 79.09 | 92.08 | 36.91 | 25.82 | 19.46 | 9.88 |
| XML_PRETTY | opt | 90.68 | 97.14 | 90.14 | 88.94 | 76.18 | 90.77 | 36.43 | 26.29 | 18.53 | 9.52 |
| YAML | man | 91.74 | 99.22 | 87.59 | 91.06 | 78.18 | 91.50 | 37.21 | 25.55 | 18.97 | 9.77 |
| YAML | opt | 91.90 | 99.95 | 90.17 | 93.07 | 71.87 | 92.15 | 37.48 | 26.30 | 19.39 | 8.98 |

#### 2.10.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 85.63 | 87.85 |  +2.22 | 32.11 | 32.94 |  +0.83 |
| JSON_COMPACT | 98.88 | 98.60 | -0.28 | 37.08 | 36.98 | -0.10 |
| JSON_PRETTY | 98.62 | 99.03 |  +0.42 | 36.98 | 37.14 |  +0.16 |
| TOON_DEFAULT | 97.03 | 97.69 |  +0.65 | 36.39 | 36.63 |  +0.25 |
| XML_COMPACT | 96.03 | 99.64 |  +3.62 | 36.01 | 37.37 |  +1.36 |
| XML_PRETTY | 98.43 | 97.14 | -1.30 | 36.91 | 36.43 | -0.48 |
| YAML | 99.22 | 99.95 |  +0.72 | 37.21 | 37.48 |  +0.27 |

#### 2.10.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 88.33 | 90.44 |  +2.11 | 25.76 | 26.38 |  +0.62 |
| JSON_COMPACT | 91.15 | 88.41 | -2.74 | 26.59 | 25.79 | -0.80 |
| JSON_PRETTY | 90.38 | 90.25 | -0.12 | 26.36 | 26.32 | -0.04 |
| TOON_DEFAULT | 90.98 | 88.38 | -2.59 | 26.54 | 25.78 | -0.76 |
| XML_COMPACT | 89.74 | 90.99 |  +1.26 | 26.17 | 26.54 |  +0.37 |
| XML_PRETTY | 88.53 | 90.14 |  +1.61 | 25.82 | 26.29 |  +0.47 |
| YAML | 87.59 | 90.17 |  +2.58 | 25.55 | 26.30 |  +0.75 |

#### 2.10.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 91.24 | 92.35 |  +1.11 | 19.01 | 19.24 |  +0.23 |
| JSON_COMPACT | 91.71 | 90.13 | -1.58 | 19.10 | 18.78 | -0.33 |
| JSON_PRETTY | 91.75 | 92.44 |  +0.70 | 19.11 | 19.26 |  +0.15 |
| TOON_DEFAULT | 92.57 | 90.23 | -2.35 | 19.28 | 18.80 | -0.49 |
| XML_COMPACT | 89.21 | 87.20 | -2.01 | 18.58 | 18.16 | -0.42 |
| XML_PRETTY | 93.39 | 88.94 | -4.44 | 19.46 | 18.53 | -0.93 |
| YAML | 91.06 | 93.07 |  +2.01 | 18.97 | 19.39 |  +0.42 |

#### 2.10.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 77.59 | 68.67 | -8.92 | 9.70 | 8.58 | -1.12 |
| JSON_COMPACT | 75.27 | 76.12 |  +0.85 | 9.41 | 9.51 |  +0.11 |
| JSON_PRETTY | 80.38 | 75.83 | -4.54 | 10.05 | 9.48 | -0.57 |
| TOON_DEFAULT | 78.80 | 75.23 | -3.57 | 9.85 | 9.40 | -0.45 |
| XML_COMPACT | 84.00 | 68.67 | -15.33 | 10.50 | 8.59 | -1.91 |
| XML_PRETTY | 79.09 | 76.18 | -2.91 | 9.88 | 9.52 | -0.36 |
| YAML | 78.18 | 71.87 | -6.30 | 9.77 | 8.98 | -0.79 |

## 3. Appendices

### 3.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: off
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