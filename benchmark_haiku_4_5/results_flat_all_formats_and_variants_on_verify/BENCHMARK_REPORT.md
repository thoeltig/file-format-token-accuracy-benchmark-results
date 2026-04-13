# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: on
- **Data Structure**: flat
- **Formats Tested**: 7 (CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: Second iteration

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
| XML_PRETTY ≈ 85s | CSV ≈ 6968 | CSV ≈ 204 | CSV ≈ 4353 | CSV ≈ 4557 | CSV ≈ 11525 | JSON_PRETTY ≈ 78.23% | TOON_DEFAULT ≈ 83 | CSV ≈ 78 | CSV ≈ 78 | JSON_PRETTY ≈ 93.90% | TOON_DEFAULT ≈ 94 | CSV ≈ 91 | CSV ≈ 91 |
| YAML (+8.48%) | TOON_DEFAULT (+0.72%) | XML_COMPACT (+2.29%) | XML_PRETTY (+118.12%) | XML_PRETTY (+115.02%) | TOON_DEFAULT (+67.48%) | YAML (-0.54%) | CSV (-7.67%) | XML_PRETTY (-16.05%) | TOON_DEFAULT (-12.35%) | YAML (-0.29%) | CSV (-4.60%) | XML_PRETTY (-16.24%) | TOON_DEFAULT (-12.76%) |
| JSON_COMPACT (+10.88%) | JSON_COMPACT (+32.82%) | JSON_COMPACT (+4.25%) | YAML (+147.45%) | YAML (+143.04%) | JSON_COMPACT (+76.72%) | TOON_DEFAULT (-2.02%) | JSON_COMPACT (-11.76%) | YAML (-20.02%) | JSON_COMPACT (-17.62%) | TOON_DEFAULT (-0.60%) | JSON_COMPACT (-9.23%) | YAML (-20.16%) | JSON_COMPACT (-16.10%) |
| TOON_DEFAULT (+18.09%) | XML_COMPACT (+67.51%) | XML_PRETTY (+48.86%) | JSON_COMPACT (+150.38%) | JSON_COMPACT (+143.84%) | YAML (+104.84%) | XML_COMPACT (-2.69%) | XML_COMPACT (-20.29%) | JSON_COMPACT (-23.89%) | YAML (-22.59%) | XML_PRETTY (-1.72%) | XML_COMPACT (-18.33%) | JSON_COMPACT (-21.44%) | YAML (-22.34%) |
| JSON_PRETTY (+27.20%) | YAML (+79.87%) | YAML (+48.86%) | TOON_DEFAULT (+175.10%) | TOON_DEFAULT (+169.56%) | XML_COMPACT (+118.00%) | XML_PRETTY (-2.69%) | YAML (-22.22%) | TOON_DEFAULT (-26.81%) | XML_COMPACT (-28.49%) | JSON_COMPACT (-1.85%) | YAML (-20.36%) | TOON_DEFAULT (-25.08%) | XML_COMPACT (-27.02%) |
| XML_COMPACT (+30.70%) | JSON_PRETTY (+104.56%) | JSON_PRETTY (+50.98%) | JSON_PRETTY (+199.09%) | JSON_PRETTY (+192.46%) | XML_PRETTY (+125.04%) | JSON_COMPACT (-4.84%) | JSON_PRETTY (-29.09%) | JSON_PRETTY (-29.83%) | XML_PRETTY (-30.66%) | XML_COMPACT (-1.96%) | JSON_PRETTY (-26.58%) | JSON_PRETTY (-28.70%) | XML_PRETTY (-28.70%) |
| CSV (+49.39%) | XML_PRETTY (+131.59%) | TOON_DEFAULT (+51.43%) | XML_COMPACT (+204.24%) | XML_COMPACT (+195.20%) | JSON_PRETTY (+139.32%) | CSV (-11.83%) | XML_PRETTY (-39.24%) | XML_COMPACT (-32.71%) | JSON_PRETTY (-32.75%) | CSV (-7.37%) | XML_PRETTY (-34.82%) | XML_COMPACT (-30.62%) | JSON_PRETTY (-31.19%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 69s | CSV ≈ 6687 | JSON_COMPACT ≈ 208 | YAML ≈ 8803 | JSON_COMPACT ≈ 9070 | CSV ≈ 15867 | XML_PRETTY ≈ 77.42% | JSON_COMPACT ≈ 76 | JSON_COMPACT ≈ 67 | JSON_COMPACT ≈ 70 | TOON_DEFAULT ≈ 92.32% | CSV ≈ 91 | JSON_COMPACT ≈ 78 | CSV ≈ 82 |
| JSON_COMPACT (+9.98%) | JSON_COMPACT (+30.51%) | XML_PRETTY (+0.48%) | JSON_COMPACT (+0.67%) | YAML (+0.45%) | JSON_COMPACT (+12.16%) | TOON_DEFAULT (-0.13%) | CSV (-3.28%) | YAML (-4.20%) | CSV (-8.07%) | XML_COMPACT (-0.40%) | JSON_COMPACT (-3.93%) | YAML (-1.51%) | JSON_COMPACT (-0.47%) |
| CSV (+10.64%) | XML_COMPACT (+63.14%) | CSV (+44.48%) | CSV (+0.86%) | CSV (+1.21%) | YAML (+31.42%) | JSON_PRETTY (-1.61%) | XML_COMPACT (-9.38%) | JSON_PRETTY (-7.38%) | XML_COMPACT (-12.54%) | XML_PRETTY (-0.77%) | XML_COMPACT (-11.95%) | CSV (-5.13%) | YAML (-9.51%) |
| JSON_PRETTY (+24.18%) | TOON_DEFAULT (+72.57%) | JSON_PRETTY (+45.60%) | JSON_PRETTY (+17.29%) | JSON_PRETTY (+17.18%) | XML_COMPACT (+40.61%) | XML_COMPACT (-1.61%) | TOON_DEFAULT (-11.00%) | XML_PRETTY (-10.26%) | YAML (-12.83%) | JSON_COMPACT (-1.04%) | TOON_DEFAULT (-14.10%) | JSON_PRETTY (-7.26%) | XML_COMPACT (-11.42%) |
| XML_PRETTY (+26.54%) | YAML (+75.59%) | YAML (+47.68%) | XML_COMPACT (+26.02%) | XML_COMPACT (+25.71%) | TOON_DEFAULT (+47.23%) | JSON_COMPACT (-2.42%) | YAML (-17.47%) | XML_COMPACT (-11.45%) | TOON_DEFAULT (-14.24%) | JSON_PRETTY (-1.29%) | YAML (-16.80%) | XML_COMPACT (-10.01%) | TOON_DEFAULT (-13.76%) |
| XML_COMPACT (+33.13%) | JSON_PRETTY (+99.58%) | XML_COMPACT (+48.16%) | XML_PRETTY (+28.02%) | XML_PRETTY (+26.56%) | JSON_PRETTY (+51.09%) | YAML (-6.45%) | JSON_PRETTY (-20.65%) | TOON_DEFAULT (-12.18%) | JSON_PRETTY (-17.45%) | YAML (-2.60%) | JSON_PRETTY (-22.08%) | XML_PRETTY (-10.67%) | JSON_PRETTY (-16.38%) |
| TOON_DEFAULT (+36.69%) | XML_PRETTY (+125.45%) | TOON_DEFAULT (+49.60%) | TOON_DEFAULT (+30.74%) | TOON_DEFAULT (+30.33%) | XML_PRETTY (+67.36%) | CSV (-16.93%) | XML_PRETTY (-27.24%) | CSV (-14.94%) | XML_PRETTY (-23.53%) | CSV (-6.47%) | XML_PRETTY (-28.42%) | TOON_DEFAULT (-11.56%) | XML_PRETTY (-22.52%) |


#### 2.1.2 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 100.00% | CSV ≈ 67.90% | TOON_DEFAULT ≈ 68.26% | XML_PRETTY ≈ 71.43% |
| XML_COMPACT (0.00%) | TOON_DEFAULT (-6.79%) | YAML (-0.00%) | YAML (-6.35%) |
| JSON_COMPACT (-1.21%) | JSON_COMPACT (-11.11%) | JSON_PRETTY (-1.59%) | TOON_DEFAULT (-7.14%) |
| YAML (-2.43%) | JSON_PRETTY (-11.11%) | XML_PRETTY (-4.76%) | CSV (-11.11%) |
| TOON_DEFAULT (-8.79%) | XML_PRETTY (-11.11%) | XML_COMPACT (-4.77%) | JSON_PRETTY (-11.11%) |
| XML_PRETTY (-9.09%) | XML_COMPACT (-12.35%) | JSON_COMPACT (-6.35%) | XML_COMPACT (-22.22%) |
| CSV (-26.67%) | YAML (-13.58%) | CSV (-15.87%) | JSON_COMPACT (-31.74%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| XML_COMPACT ≈ 100.00% | JSON_PRETTY ≈ 72.84% | TOON_DEFAULT ≈ 67.46% | TOON_DEFAULT ≈ 50.00% |
| TOON_DEFAULT (-1.52%) | XML_PRETTY (0.00%) | JSON_PRETTY (-2.38%) | JSON_COMPACT (-0.79%) |
| XML_PRETTY (-3.03%) | JSON_COMPACT (-3.70%) | XML_COMPACT (-2.38%) | XML_PRETTY (-2.38%) |
| JSON_COMPACT (-7.27%) | TOON_DEFAULT (-9.88%) | YAML (-3.97%) | YAML (-3.96%) |
| JSON_PRETTY (-7.27%) | XML_COMPACT (-12.35%) | JSON_COMPACT (-5.55%) | JSON_PRETTY (-3.97%) |
| YAML (-9.70%) | YAML (-16.05%) | XML_PRETTY (-5.55%) | XML_COMPACT (-7.14%) |
| CSV (-23.03%) | CSV (-23.46%) | CSV (-10.32%) | CSV (-15.08%) |


#### 2.1.3 Category Accuracy By Character Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 100.00% | CSV ≈ 92.07% | JSON_PRETTY ≈ 94.18% | YAML ≈ 86.60% |
| XML_COMPACT (0.00%) | TOON_DEFAULT (-0.72%) | TOON_DEFAULT (-0.43%) | XML_PRETTY (-0.45%) |
| JSON_COMPACT (-0.73%) | JSON_COMPACT (-0.73%) | YAML (-1.27%) | TOON_DEFAULT (-0.73%) |
| YAML (-1.03%) | XML_COMPACT (-2.71%) | XML_PRETTY (-2.38%) | JSON_PRETTY (-3.01%) |
| TOON_DEFAULT (-3.07%) | JSON_PRETTY (-2.80%) | JSON_COMPACT (-3.54%) | CSV (-3.09%) |
| XML_PRETTY (-3.54%) | YAML (-3.38%) | CSV (-4.55%) | XML_COMPACT (-9.46%) |
| CSV (-16.21%) | XML_PRETTY (-3.62%) | XML_COMPACT (-5.24%) | JSON_COMPACT (-11.12%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| XML_COMPACT ≈ 100.00% | XML_COMPACT ≈ 90.91% | TOON_DEFAULT ≈ 92.86% | JSON_COMPACT ≈ 76.18% |
| TOON_DEFAULT (-0.17%) | TOON_DEFAULT (-0.11%) | YAML (-1.32%) | XML_PRETTY (-1.81%) |
| XML_PRETTY (-0.95%) | XML_PRETTY (-0.12%) | XML_COMPACT (-1.38%) | TOON_DEFAULT (-2.13%) |
| JSON_COMPACT (-1.58%) | JSON_COMPACT (-0.41%) | JSON_PRETTY (-1.96%) | JSON_PRETTY (-2.98%) |
| JSON_PRETTY (-1.80%) | CSV (-0.44%) | XML_PRETTY (-2.81%) | CSV (-3.68%) |
| YAML (-3.78%) | JSON_PRETTY (-0.52%) | CSV (-3.38%) | XML_COMPACT (-3.68%) |
| CSV (-12.71%) | YAML (-2.35%) | JSON_COMPACT (-4.18%) | YAML (-3.78%) |


#### 2.1.4 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total | Accuracy By Character (%) | Useful Read Tokens (Acc By Char) | Wasted Read Tokens (Acc By Char) | Useful Output Tokens (Acc By Char) | Wasted Output Tokens (Acc By Char) | Eff Score Read (Acc By Char) | Eff Score Output (Acc By Char) | Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6968 | 4557 | 11525 | 1.449 | 35.108 | 66.40 | 4626.752 | 2341.248 | 3026.069 | 1531.264 | 76.58 | 77.56 | 77.58 | 86.53 | 6029.410 | 938.590 | 3943.460 | 613.873 | 90.00 | 90.98 | 91.00 |
| CSV | opt | 6687 | 9180 | 15867 | 1.432 | 71.602 | 60.49 | 4044.966 | 2642.034 | 5552.781 | 3626.886 | 73.62 | 57.27 | 64.64 | 85.85 | 5740.790 | 946.211 | 7880.744 | 1298.923 | 90.53 | 74.18 | 81.54 |
| JSON_COMPACT | man | 9255 | 11113 | 20368 | 2.152 | 87.903 | 73.39 | 6792.245 | 2462.756 | 8155.586 | 2957.081 | 73.19 | 59.03 | 63.91 | 92.05 | 8519.227 | 735.773 | 10229.210 | 883.457 | 85.63 | 71.47 | 76.34 |
| JSON_COMPACT | opt | 8727 | 9070 | 17797 | 2.118 | 71.462 | 75.00 | 6545.250 | 2181.750 | 6802.250 | 2267.417 | 76.12 | 67.33 | 70.31 | 91.28 | 7966.006 | 760.994 | 8278.792 | 790.875 | 86.97 | 78.19 | 81.16 |
| JSON_PRETTY | man | 14254 | 13328 | 27582 | 1.697 | 105.003 | 78.23 | 11150.904 | 3103.096 | 10426.755 | 2901.578 | 58.82 | 54.42 | 52.17 | 93.90 | 13384.506 | 869.494 | 12515.305 | 813.028 | 69.26 | 64.87 | 62.62 |
| JSON_PRETTY | opt | 13346 | 10628 | 23974 | 1.683 | 83.261 | 75.81 | 10117.603 | 3228.397 | 8056.834 | 2570.833 | 60.40 | 62.36 | 58.04 | 91.03 | 12148.864 | 1197.136 | 9674.365 | 953.302 | 70.55 | 72.51 | 68.19 |
| TOON_DEFAULT | man | 7018 | 12285 | 19303 | 1.448 | 96.579 | 76.21 | 5348.418 | 1669.582 | 9362.208 | 2922.542 | 82.94 | 56.77 | 67.99 | 93.30 | 6547.794 | 470.206 | 11461.672 | 823.079 | 94.33 | 68.16 | 79.39 |
| TOON_DEFAULT | opt | 11540 | 11821 | 23361 | 1.701 | 92.815 | 77.29 | 8919.266 | 2620.734 | 9136.193 | 2684.474 | 67.74 | 59.13 | 60.30 | 92.32 | 10653.728 | 886.272 | 10912.839 | 907.827 | 77.76 | 69.15 | 70.32 |
| XML_COMPACT | man | 11672 | 13453 | 25125 | 2.370 | 106.812 | 75.54 | 8817.029 | 2854.971 | 10162.648 | 3290.685 | 66.11 | 52.19 | 55.47 | 91.94 | 10731.237 | 940.763 | 12368.994 | 1084.339 | 77.04 | 63.12 | 66.41 |
| XML_COMPACT | opt | 10909 | 11402 | 22311 | 2.345 | 89.460 | 75.81 | 8270.113 | 2638.887 | 8643.604 | 2758.063 | 68.98 | 59.63 | 61.49 | 91.92 | 10027.553 | 881.447 | 10480.412 | 921.255 | 79.72 | 70.37 | 72.23 |
| XML_PRETTY | man | 16137 | 9799 | 25936 | 1.937 | 76.578 | 75.54 | 12189.890 | 3947.110 | 7402.416 | 2396.917 | 50.40 | 65.11 | 53.79 | 92.18 | 14875.087 | 1261.913 | 9033.025 | 766.308 | 61.49 | 76.21 | 64.89 |
| XML_PRETTY | opt | 15076 | 11479 | 26555 | 1.918 | 90.882 | 77.42 | 11671.839 | 3404.161 | 8886.784 | 2591.883 | 55.38 | 60.43 | 53.76 | 91.55 | 13802.078 | 1273.922 | 10508.720 | 969.947 | 64.80 | 69.85 | 63.18 |
| YAML | man | 12533 | 11076 | 23609 | 1.666 | 86.874 | 77.69 | 9736.888 | 2796.112 | 8604.944 | 2471.056 | 64.51 | 62.03 | 60.05 | 93.61 | 11732.141 | 800.859 | 10368.244 | 707.756 | 75.13 | 72.64 | 70.67 |
| YAML | opt | 11742 | 9110 | 20852 | 1.653 | 70.989 | 70.97 | 8333.297 | 3408.703 | 6465.603 | 2644.730 | 62.82 | 64.50 | 61.29 | 89.72 | 10534.922 | 1207.078 | 8173.791 | 936.542 | 75.32 | 77.00 | 73.79 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6968 | 6687 | -281 | -4.03 | 204 | 301 |  +97 |  +47.55 | 4353 | 8878 |  +4525 |  +103.95 | 4557 | 9179 |  +4622 |  +101.43 | 11525 | 15866 |  +4341 |  +37.67 |
| JSON_COMPACT | 9255 | 8727 | -528 | -5.71 | 213 | 209 | -4 | -1.88 | 10900 | 8861 | -2039 | -18.71 | 11113 | 9070 | -2043 | -18.38 | 20368 | 17797 | -2571 | -12.62 |
| JSON_PRETTY | 14254 | 13346 | -908 | -6.37 | 308 | 303 | -5 | -1.62 | 13020 | 10324 | -2696 | -20.71 | 13328 | 10627 | -2701 | -20.27 | 27582 | 23973 | -3609 | -13.08 |
| TOON_DEFAULT | 7018 | 11540 |  +4522 |  +64.43 | 309 | 312 |  +3 |  +0.97 | 11976 | 11509 | -467 | -3.90 | 12285 | 11821 | -464 | -3.78 | 19303 | 23361 |  +4058 |  +21.02 |
| XML_COMPACT | 11672 | 10909 | -763 | -6.54 | 209 | 309 |  +100 |  +47.85 | 13245 | 11093 | -2152 | -16.25 | 13453 | 11401 | -2052 | -15.25 | 25125 | 22310 | -2815 | -11.20 |
| XML_PRETTY | 16137 | 15076 | -1061 | -6.57 | 304 | 210 | -94 | -30.92 | 9496 | 11270 |  +1774 |  +18.68 | 9799 | 11478 |  +1679 |  +17.13 | 25936 | 26554 |  +618 |  +2.38 |
| YAML | 12533 | 11742 | -791 | -6.31 | 304 | 308 |  +4 |  +1.32 | 10772 | 8802 | -1970 | -18.29 | 11076 | 9110 | -1966 | -17.75 | 23609 | 20852 | -2757 | -11.68 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Before Write (ms) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) | Output (ms) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 23 | 302.957 | 0.74 | 74648 | 52278 | 0.083 | 421.59 | 52301 | 303.040 | 337.42 | 126926 |
| CSV | opt | 14 | 477.643 | 0.45 | 37359 | 39364 | 0.226 | 317.45 | 39378 | 477.869 | 254.05 | 76723 |
| JSON_COMPACT | man | 24 | 385.625 | 0.77 | 54629 | 39580 | 0.275 | 319.19 | 39604 | 385.900 | 255.51 | 94209 |
| JSON_COMPACT | opt | 13 | 671.308 | 0.42 | 38635 | 37630 | 0.235 | 303.47 | 37643 | 671.543 | 242.86 | 76265 |
| JSON_PRETTY | man | 30 | 475.133 | 0.97 | 68671 | 39401 | 0.330 | 317.75 | 39431 | 475.463 | 254.40 | 108073 |
| JSON_PRETTY | opt | 36 | 370.722 | 1.16 | 45880 | 40232 | 0.257 | 324.45 | 40268 | 370.979 | 259.79 | 86112 |
| TOON_DEFAULT | man | 26 | 269.923 | 0.84 | 62236 | 38099 | 0.313 | 307.25 | 38125 | 270.236 | 245.97 | 100335 |
| TOON_DEFAULT | opt | 20 | 577.000 | 0.65 | 57725 | 37064 | 0.311 | 298.90 | 37084 | 577.312 | 239.25 | 94789 |
| XML_COMPACT | man | 19 | 614.316 | 0.61 | 73292 | 37753 | 0.351 | 304.46 | 37772 | 614.667 | 243.69 | 111045 |
| XML_COMPACT | opt | 23 | 474.304 | 0.74 | 55775 | 36545 | 0.304 | 294.72 | 36568 | 474.608 | 235.92 | 92319 |
| XML_PRETTY | man | 18 | 896.500 | 0.58 | 45341 | 39623 | 0.240 | 319.54 | 39641 | 896.740 | 255.75 | 84964 |
| XML_PRETTY | opt | 17 | 886.824 | 0.55 | 57146 | 30600 | 0.368 | 246.78 | 30617 | 887.192 | 197.53 | 87747 |
| YAML | man | 21 | 596.810 | 0.68 | 53707 | 38458 | 0.280 | 310.15 | 38479 | 597.090 | 248.25 | 92165 |
| YAML | opt | 31 | 378.774 | 1.00 | 39189 | 30155 | 0.292 | 243.19 | 30186 | 379.066 | 194.75 | 69344 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Before Write Man (s) | Output Before Write Opt (s) | Diff (s) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 23 | 14 | -9 | -39.13 | 74.65 | 37.36 | -37.29 | -49.95 | 52.28 | 39.36 | -12.91 | -24.70 | 52.30 | 39.38 | -12.92 | -24.71 | 126.93 | 76.72 | -50.20 | -39.55 |
| JSON_COMPACT | 24 | 13 | -11 | -45.83 | 54.63 | 38.63 | -15.99 | -29.28 | 39.58 | 37.63 | -1.95 | -4.93 | 39.60 | 37.64 | -1.96 | -4.95 | 94.21 | 76.26 | -17.94 | -19.05 |
| JSON_PRETTY | 30 | 36 |  +6 |  +20.00 | 68.67 | 45.88 | -22.79 | -33.19 | 39.40 | 40.23 |  +0.83 |  +2.11 | 39.43 | 40.27 |  +0.84 |  +2.12 | 108.07 | 86.11 | -21.96 | -20.32 |
| TOON_DEFAULT | 26 | 20 | -6 | -23.08 | 62.24 | 57.72 | -4.51 | -7.25 | 38.10 | 37.06 | -1.03 | -2.72 | 38.12 | 37.08 | -1.04 | -2.73 | 100.34 | 94.79 | -5.55 | -5.53 |
| XML_COMPACT | 19 | 23 |  +4 |  +21.05 | 73.29 | 55.77 | -17.52 | -23.90 | 37.75 | 36.54 | -1.21 | -3.20 | 37.77 | 36.57 | -1.20 | -3.19 | 111.05 | 92.32 | -18.73 | -16.86 |
| XML_PRETTY | 18 | 17 | -1 | -5.56 | 45.34 | 57.15 |  +11.81 |  +26.04 | 39.62 | 30.60 | -9.02 | -22.77 | 39.64 | 30.62 | -9.02 | -22.76 | 84.96 | 87.75 |  +2.78 |  +3.28 |
| YAML | 21 | 31 |  +10 |  +47.62 | 53.71 | 39.19 | -14.52 | -27.03 | 38.46 | 30.15 | -8.30 | -21.59 | 38.48 | 30.19 | -8.29 | -21.55 | 92.17 | 69.34 | -22.82 | -24.76 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token | Info / Read Token (Acc By Char) | Info / Output Token (Acc By Char) | Info / Total Token (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 1.449 | 10.217 | 224.774 | 0.953 | 1.457 | 0.576 | 1.242 | 1.899 | 0.751 |
| CSV | opt | 1.432 | 10.597 | 215.710 | 0.905 | 0.659 | 0.381 | 1.284 | 0.935 | 0.541 |
| JSON_COMPACT | man | 2.152 | 13.570 | 298.548 | 0.793 | 0.660 | 0.360 | 0.995 | 0.828 | 0.452 |
| JSON_COMPACT | opt | 2.118 | 13.830 | 281.516 | 0.859 | 0.827 | 0.421 | 1.046 | 1.006 | 0.513 |
| JSON_PRETTY | man | 1.697 | 20.900 | 459.806 | 0.549 | 0.587 | 0.284 | 0.659 | 0.705 | 0.340 |
| JSON_PRETTY | opt | 1.683 | 21.151 | 430.516 | 0.568 | 0.713 | 0.316 | 0.682 | 0.857 | 0.380 |
| TOON_DEFAULT | man | 1.448 | 10.290 | 226.387 | 1.086 | 0.632 | 0.397 | 1.329 | 0.774 | 0.487 |
| TOON_DEFAULT | opt | 1.701 | 18.288 | 372.258 | 0.670 | 0.656 | 0.331 | 0.800 | 0.784 | 0.395 |
| XML_COMPACT | man | 2.370 | 17.114 | 376.516 | 0.647 | 0.561 | 0.301 | 0.788 | 0.683 | 0.366 |
| XML_COMPACT | opt | 2.345 | 17.288 | 351.903 | 0.695 | 0.665 | 0.340 | 0.843 | 0.806 | 0.412 |
| XML_PRETTY | man | 1.937 | 23.661 | 520.548 | 0.468 | 0.771 | 0.291 | 0.571 | 0.941 | 0.355 |
| XML_PRETTY | opt | 1.918 | 23.892 | 486.323 | 0.514 | 0.674 | 0.292 | 0.607 | 0.798 | 0.345 |
| YAML | man | 1.666 | 18.377 | 404.290 | 0.620 | 0.701 | 0.329 | 0.747 | 0.845 | 0.397 |
| YAML | opt | 1.653 | 18.609 | 378.774 | 0.604 | 0.779 | 0.340 | 0.764 | 0.985 | 0.430 |

#### 2.5.2 Characters And Values: Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.449 | 1.432 | -0.017 | -1.17 | 10.217 | 10.597 |  +0.380 |  +3.72 | 224.774 | 215.710 | -9.064 | -4.03 |
| JSON_COMPACT | 2.152 | 2.118 | -0.034 | -1.58 | 13.570 | 13.830 |  +0.260 |  +1.92 | 298.548 | 281.516 | -17.032 | -5.70 |
| JSON_PRETTY | 1.697 | 1.683 | -0.014 | -0.82 | 20.900 | 21.151 |  +0.251 |  +1.20 | 459.806 | 430.516 | -29.290 | -6.37 |
| TOON_DEFAULT | 1.448 | 1.701 |  +0.253 |  +17.47 | 10.290 | 18.288 |  +7.998 |  +77.73 | 226.387 | 372.258 |  +145.871 |  +64.43 |
| XML_COMPACT | 2.370 | 2.345 | -0.025 | -1.05 | 17.114 | 17.288 |  +0.174 |  +1.02 | 376.516 | 351.903 | -24.613 | -6.54 |
| XML_PRETTY | 1.937 | 1.918 | -0.019 | -0.98 | 23.661 | 23.892 |  +0.231 |  +0.98 | 520.548 | 486.323 | -34.225 | -6.57 |
| YAML | 1.666 | 1.653 | -0.013 | -0.78 | 18.377 | 18.609 |  +0.232 |  +1.26 | 404.290 | 378.774 | -25.516 | -6.31 |

#### 2.5.3 Information: Mandatory vs Optional
| Format | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 0.953 | 0.905 | -0.048 | -5.04 | 1.457 | 0.659 | -0.798 | -54.77 | 0.576 | 0.381 | -0.195 | -33.85 |
| JSON_COMPACT | 0.793 | 0.859 |  +0.066 |  +8.32 | 0.660 | 0.827 |  +0.167 |  +25.30 | 0.360 | 0.421 |  +0.061 |  +16.94 |
| JSON_PRETTY | 0.549 | 0.568 |  +0.019 |  +3.46 | 0.587 | 0.713 |  +0.126 |  +21.47 | 0.284 | 0.316 |  +0.032 |  +11.27 |
| TOON_DEFAULT | 1.086 | 0.670 | -0.416 | -38.31 | 0.632 | 0.656 |  +0.024 |  +3.88 | 0.397 | 0.331 | -0.066 | -16.73 |
| XML_COMPACT | 0.647 | 0.695 |  +0.048 |  +7.42 | 0.561 | 0.665 |  +0.104 |  +18.54 | 0.301 | 0.340 |  +0.039 |  +12.96 |
| XML_PRETTY | 0.468 | 0.514 |  +0.046 |  +9.83 | 0.771 | 0.674 | -0.097 | -12.58 | 0.291 | 0.292 |  +0.001 |  +0.34 |
| YAML | 0.620 | 0.604 | -0.016 | -2.58 | 0.701 | 0.779 |  +0.078 |  +11.13 | 0.329 | 0.340 |  +0.011 |  +3.34 |

#### 2.5.4 Information (Accuracy By Character): Mandatory vs Optional
| Format | Info / Read Token (Acc By Char) Man | Info / Read Token (Acc By Char) Opt | Diff | Diff (%) | Info / Output Token (Acc By Char) Man | Info / Output Token (Acc By Char)  Opt | Diff | Diff (%) | Info / Total Token (Acc By Char) Man | Info / Total Token (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.242 | 1.284 |  +0.042 |  +3.38 | 1.899 | 0.935 | -0.964 | -50.76 | 0.751 | 0.541 | -0.210 | -27.96 |
| JSON_COMPACT | 0.995 | 1.046 |  +0.051 |  +5.13 | 0.828 | 1.006 |  +0.178 |  +21.50 | 0.452 | 0.513 |  +0.061 |  +13.50 |
| JSON_PRETTY | 0.659 | 0.682 |  +0.023 |  +3.49 | 0.705 | 0.857 |  +0.152 |  +21.56 | 0.340 | 0.380 |  +0.040 |  +11.76 |
| TOON_DEFAULT | 1.329 | 0.800 | -0.529 | -39.80 | 0.774 | 0.784 |  +0.011 |  +1.42 | 0.487 | 0.395 | -0.092 | -18.79 |
| XML_COMPACT | 0.788 | 0.843 |  +0.055 |  +6.98 | 0.683 | 0.806 |  +0.123 |  +18.01 | 0.366 | 0.412 |  +0.046 |  +12.57 |
| XML_PRETTY | 0.571 | 0.607 |  +0.036 |  +6.30 | 0.941 | 0.798 | -0.143 | -15.20 | 0.355 | 0.345 | -0.010 | -2.82 |
| YAML | 0.747 | 0.764 |  +0.017 |  +2.28 | 0.845 | 0.985 |  +0.140 |  +16.57 | 0.397 | 0.430 |  +0.033 |  +8.31 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6968 | 4627 | 2341 | 4557 | 3026 | 1531 | 11525 | 7653 | 3873 | 66.40 | 76.58 | 77.56 | 77.58 | 65.76 | 76.15 | 77.14 | 77.15 |
| CSV | opt | 6687 | 4045 | 2642 | 9180 | 5553 | 3627 | 15867 | 9598 | 6269 | 60.49 | 73.62 | 57.27 | 64.64 | 59.53 | 72.98 | 56.63 | 64.00 |
| JSON_COMPACT | man | 9255 | 6792 | 2463 | 11113 | 8156 | 2957 | 20368 | 14948 | 5420 | 73.39 | 73.19 | 59.03 | 63.91 | 71.47 | 71.91 | 57.76 | 62.63 |
| JSON_COMPACT | opt | 8727 | 6545 | 2182 | 9070 | 6802 | 2267 | 17797 | 13348 | 4449 | 75.00 | 76.12 | 67.33 | 70.31 | 73.99 | 75.44 | 66.66 | 69.64 |
| JSON_PRETTY | man | 14254 | 11151 | 3103 | 13328 | 10427 | 2902 | 27582 | 21578 | 6005 | 78.23 | 58.82 | 54.42 | 52.17 | 75.49 | 56.99 | 52.60 | 50.35 |
| JSON_PRETTY | opt | 13346 | 10118 | 3228 | 10628 | 8057 | 2571 | 23974 | 18174 | 5799 | 75.81 | 60.40 | 62.36 | 58.04 | 75.33 | 60.08 | 62.04 | 57.72 |
| TOON_DEFAULT | man | 7018 | 5348 | 1670 | 12285 | 9362 | 2923 | 19303 | 14711 | 4592 | 76.21 | 82.94 | 56.77 | 67.99 | 74.28 | 81.65 | 55.48 | 66.71 |
| TOON_DEFAULT | opt | 11540 | 8919 | 2621 | 11821 | 9136 | 2684 | 23361 | 18055 | 5305 | 77.29 | 67.74 | 59.13 | 60.30 | 75.60 | 66.61 | 58.00 | 59.17 |
| XML_COMPACT | man | 11672 | 8817 | 2855 | 13453 | 10163 | 3291 | 25125 | 18980 | 6146 | 75.54 | 66.11 | 52.19 | 55.47 | 73.08 | 64.47 | 50.55 | 53.83 |
| XML_COMPACT | opt | 10909 | 8270 | 2639 | 11402 | 8644 | 2758 | 22311 | 16914 | 5397 | 75.81 | 68.98 | 59.63 | 61.49 | 74.06 | 67.81 | 58.46 | 60.32 |
| XML_PRETTY | man | 16137 | 12190 | 3947 | 9799 | 7402 | 2397 | 25936 | 19592 | 6344 | 75.54 | 50.40 | 65.11 | 53.79 | 72.81 | 48.58 | 63.29 | 51.97 |
| XML_PRETTY | opt | 15076 | 11672 | 3404 | 11479 | 8887 | 2592 | 26555 | 20559 | 5996 | 77.42 | 55.38 | 60.43 | 53.76 | 76.46 | 54.74 | 59.79 | 53.12 |
| YAML | man | 12533 | 9737 | 2796 | 11076 | 8605 | 2471 | 23609 | 18342 | 5267 | 77.69 | 64.51 | 62.03 | 60.05 | 74.78 | 62.57 | 60.09 | 58.11 |
| YAML | opt | 11742 | 8333 | 3409 | 9110 | 6466 | 2645 | 20852 | 14799 | 6053 | 70.97 | 62.82 | 64.50 | 61.29 | 69.41 | 61.78 | 63.47 | 60.25 |

#### 2.6.2 Read Tokens: Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6968 | 6687 | -281 | -4.03 | 4627 | 4045 | -582 | -12.57 | 2341 | 2642 |  +301 |  +12.85 | 66.40 | 60.49 | -5.91 | -8.90 | 76.58 | 73.62 | -2.95 | -3.85 | 65.76 | 59.53 | -6.23 | -9.47 | 76.15 | 72.98 | -3.16 | -4.16 |
| JSON_COMPACT | 9255 | 8727 | -528 | -5.71 | 6792 | 6545 | -247 | -3.64 | 2463 | 2182 | -281 | -11.41 | 73.39 | 75.00 |  +1.61 |  +2.19 | 73.19 | 76.12 |  +2.93 |  +4.01 | 71.47 | 73.99 |  +2.52 |  +3.53 | 71.91 | 75.44 |  +3.54 |  +4.92 |
| JSON_PRETTY | 14254 | 13346 | -908 | -6.37 | 11151 | 10118 | -1033 | -9.27 | 3103 | 3228 |  +125 |  +4.04 | 78.23 | 75.81 | -2.42 | -3.09 | 58.82 | 60.40 |  +1.58 |  +2.69 | 75.49 | 75.33 | -0.16 | -0.21 | 56.99 | 60.08 |  +3.09 |  +5.42 |
| TOON_DEFAULT | 7018 | 11540 |  +4522 |  +64.43 | 5348 | 8919 |  +3571 |  +66.77 | 1670 | 2621 |  +951 |  +56.96 | 76.21 | 77.29 |  +1.08 |  +1.42 | 82.94 | 67.74 | -15.20 | -18.32 | 74.28 | 75.60 |  +1.32 |  +1.78 | 81.65 | 66.61 | -15.04 | -18.42 |
| XML_COMPACT | 11672 | 10909 | -763 | -6.54 | 8817 | 8270 | -547 | -6.20 | 2855 | 2639 | -216 | -7.57 | 75.54 | 75.81 |  +0.27 |  +0.36 | 66.11 | 68.98 |  +2.86 |  +4.33 | 73.08 | 74.06 |  +0.98 |  +1.34 | 64.47 | 67.81 |  +3.34 |  +5.18 |
| XML_PRETTY | 16137 | 15076 | -1061 | -6.57 | 12190 | 11672 | -518 | -4.25 | 3947 | 3404 | -543 | -13.76 | 75.54 | 77.42 |  +1.88 |  +2.49 | 50.40 | 55.38 |  +4.99 |  +9.90 | 72.81 | 76.46 |  +3.65 |  +5.01 | 48.58 | 54.74 |  +6.17 |  +12.70 |
| YAML | 12533 | 11742 | -791 | -6.31 | 9737 | 8333 | -1404 | -14.42 | 2796 | 3409 |  +613 |  +21.91 | 77.69 | 70.97 | -6.72 | -8.65 | 64.51 | 62.82 | -1.70 | -2.63 | 74.78 | 69.41 | -5.37 | -7.18 | 62.57 | 61.78 | -0.80 | -1.27 |

#### 2.6.3 Output Tokens: Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 4557 | 9179 |  +4622 |  +101.43 | 3026 | 5553 |  +2527 |  +83.50 | 1531 | 3627 |  +2096 |  +136.88 | 66.40 | 60.49 | -5.91 | -8.90 | 77.56 | 57.27 | -20.29 | -26.16 | 65.76 | 59.53 | -6.23 | -9.47 | 77.14 | 56.63 | -20.50 | -26.58 |
| JSON_COMPACT | 11113 | 9070 | -2043 | -18.38 | 8156 | 6803 | -1353 | -16.59 | 2957 | 2267 | -690 | -23.32 | 73.39 | 75.00 |  +1.61 |  +2.19 | 59.03 | 67.33 |  +8.30 |  +14.06 | 71.47 | 73.99 |  +2.52 |  +3.53 | 57.76 | 66.66 |  +8.91 |  +15.42 |
| JSON_PRETTY | 13328 | 10627 | -2701 | -20.26 | 10427 | 8057 | -2370 | -22.73 | 2902 | 2571 | -331 | -11.40 | 78.23 | 75.81 | -2.42 | -3.09 | 54.42 | 62.36 |  +7.94 |  +14.59 | 75.49 | 75.33 | -0.16 | -0.21 | 52.60 | 62.04 |  +9.45 |  +17.96 |
| TOON_DEFAULT | 12285 | 11821 | -464 | -3.78 | 9362 | 9136 | -226 | -2.41 | 2923 | 2685 | -238 | -8.14 | 76.21 | 77.29 |  +1.08 |  +1.42 | 56.77 | 59.13 |  +2.36 |  +4.16 | 74.28 | 75.60 |  +1.32 |  +1.78 | 55.48 | 58.00 |  +2.52 |  +4.55 |
| XML_COMPACT | 13453 | 11401 | -2052 | -15.25 | 10163 | 8644 | -1519 | -14.95 | 3291 | 2758 | -533 | -16.18 | 75.54 | 75.81 |  +0.27 |  +0.36 | 52.19 | 59.63 |  +7.44 |  +14.25 | 73.08 | 74.06 |  +0.98 |  +1.34 | 50.55 | 58.46 |  +7.91 |  +15.65 |
| XML_PRETTY | 9799 | 11478 |  +1679 |  +17.14 | 7402 | 8886 |  +1484 |  +20.05 | 2397 | 2592 |  +195 |  +8.13 | 75.54 | 77.42 |  +1.88 |  +2.49 | 65.11 | 60.43 | -4.69 | -7.20 | 72.81 | 76.46 |  +3.65 |  +5.01 | 63.29 | 59.79 | -3.51 | -5.54 |
| YAML | 11076 | 9110 | -1966 | -17.75 | 8605 | 6466 | -2139 | -24.86 | 2471 | 2645 |  +174 |  +7.03 | 77.69 | 70.97 | -6.72 | -8.65 | 62.03 | 64.50 |  +2.47 |  +3.99 | 74.78 | 69.41 | -5.37 | -7.18 | 60.09 | 63.47 |  +3.37 |  +5.61 |

#### 2.6.4 Total Tokens: Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 11525 | 15866 |  +4341 |  +37.67 | 7653 | 9598 |  +1945 |  +25.41 | 3873 | 6269 |  +2396 |  +61.87 | 66.40 | 60.49 | -5.91 | -8.90 | 77.58 | 64.64 | -12.94 | -16.68 | 65.76 | 59.53 | -6.23 | -9.47 | 77.15 | 64.00 | -13.16 | -17.05 |
| JSON_COMPACT | 20368 | 17797 | -2571 | -12.62 | 14948 | 13348 | -1600 | -10.71 | 5420 | 4449 | -971 | -17.91 | 73.39 | 75.00 |  +1.61 |  +2.19 | 63.91 | 70.31 |  +6.40 |  +10.02 | 71.47 | 73.99 |  +2.52 |  +3.53 | 62.63 | 69.64 |  +7.01 |  +11.20 |
| JSON_PRETTY | 27582 | 23973 | -3609 | -13.08 | 21578 | 18175 | -3403 | -15.77 | 6005 | 5800 | -205 | -3.42 | 78.23 | 75.81 | -2.42 | -3.09 | 52.17 | 58.04 |  +5.87 |  +11.25 | 75.49 | 75.33 | -0.16 | -0.21 | 50.35 | 57.72 |  +7.38 |  +14.65 |
| TOON_DEFAULT | 19303 | 23361 |  +4058 |  +21.02 | 14711 | 18056 |  +3345 |  +22.74 | 4592 | 5305 |  +713 |  +15.53 | 76.21 | 77.29 |  +1.08 |  +1.42 | 67.99 | 60.30 | -7.69 | -11.32 | 74.28 | 75.60 |  +1.32 |  +1.78 | 66.71 | 59.17 | -7.53 | -11.29 |
| XML_COMPACT | 25125 | 22310 | -2815 | -11.20 | 18980 | 16914 | -2066 | -10.88 | 6146 | 5397 | -749 | -12.18 | 75.54 | 75.81 |  +0.27 |  +0.36 | 55.47 | 61.49 |  +6.02 |  +10.84 | 73.08 | 74.06 |  +0.98 |  +1.34 | 53.83 | 60.32 |  +6.49 |  +12.06 |
| XML_PRETTY | 25936 | 26554 |  +618 |  +2.38 | 19592 | 20558 |  +966 |  +4.93 | 6344 | 5996 | -348 | -5.49 | 75.54 | 77.42 |  +1.88 |  +2.49 | 53.79 | 53.76 | -0.03 | -0.05 | 72.81 | 76.46 |  +3.65 |  +5.01 | 51.97 | 53.12 |  +1.15 |  +2.21 |
| YAML | 23609 | 20852 | -2757 | -11.68 | 18342 | 14799 | -3543 | -19.32 | 5267 | 6053 |  +786 |  +14.93 | 77.69 | 70.97 | -6.72 | -8.65 | 60.05 | 61.29 |  +1.23 |  +2.06 | 74.78 | 69.41 | -5.37 | -7.18 | 58.11 | 60.25 |  +2.13 |  +3.67 |

### 2.7 Token Utilization Efficiency (Accuracy by Character)
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy by Character (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy by Character (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6968 | 6029 | 939 | 4557 | 3943 | 614 | 11525 | 9973 | 1552 | 86.53 | 90.00 | 90.98 | 91.00 | 87.39 | 90.57 | 91.56 | 91.57 |
| CSV | opt | 6687 | 5741 | 946 | 9180 | 7881 | 1299 | 15867 | 13622 | 2245 | 85.85 | 90.53 | 74.18 | 81.54 | 86.82 | 91.18 | 74.83 | 82.19 |
| JSON_COMPACT | man | 9255 | 8519 | 736 | 11113 | 10229 | 883 | 20368 | 18748 | 1619 | 92.05 | 85.63 | 71.47 | 76.34 | 92.18 | 85.71 | 71.56 | 76.43 |
| JSON_COMPACT | opt | 8727 | 7966 | 761 | 9070 | 8279 | 791 | 17797 | 16245 | 1552 | 91.28 | 86.97 | 78.19 | 81.16 | 91.30 | 86.98 | 78.20 | 81.18 |
| JSON_PRETTY | man | 14254 | 13385 | 869 | 13328 | 12515 | 813 | 27582 | 25900 | 1683 | 93.90 | 69.26 | 64.87 | 62.62 | 93.61 | 69.07 | 64.68 | 62.43 |
| JSON_PRETTY | opt | 13346 | 12149 | 1197 | 10628 | 9674 | 953 | 23974 | 21823 | 2150 | 91.03 | 70.55 | 72.51 | 68.19 | 91.28 | 70.71 | 72.68 | 68.36 |
| TOON_DEFAULT | man | 7018 | 6548 | 470 | 12285 | 11462 | 823 | 19303 | 18009 | 1293 | 93.30 | 94.33 | 68.16 | 79.39 | 93.26 | 94.31 | 68.14 | 79.36 |
| TOON_DEFAULT | opt | 11540 | 10654 | 886 | 11821 | 10913 | 908 | 23361 | 21567 | 1794 | 92.32 | 77.76 | 69.15 | 70.32 | 92.52 | 77.89 | 69.28 | 70.45 |
| XML_COMPACT | man | 11672 | 10731 | 941 | 13453 | 12369 | 1084 | 25125 | 23100 | 2025 | 91.94 | 77.04 | 63.12 | 66.41 | 91.73 | 76.90 | 62.98 | 66.27 |
| XML_COMPACT | opt | 10909 | 10028 | 881 | 11402 | 10480 | 921 | 22311 | 20508 | 1803 | 91.92 | 79.72 | 70.37 | 72.23 | 92.14 | 79.86 | 70.51 | 72.38 |
| XML_PRETTY | man | 16137 | 14875 | 1262 | 9799 | 9033 | 766 | 25936 | 23908 | 2028 | 92.18 | 61.49 | 76.21 | 64.89 | 91.86 | 61.27 | 75.99 | 64.67 |
| XML_PRETTY | opt | 15076 | 13802 | 1274 | 11479 | 10509 | 970 | 26555 | 24311 | 2244 | 91.55 | 64.80 | 69.85 | 63.18 | 91.68 | 64.89 | 69.93 | 63.27 |
| YAML | man | 12533 | 11732 | 801 | 11076 | 10368 | 708 | 23609 | 22100 | 1509 | 93.61 | 75.13 | 72.64 | 70.67 | 93.15 | 74.82 | 72.34 | 70.36 |
| YAML | opt | 11742 | 10535 | 1207 | 9110 | 8174 | 937 | 20852 | 18709 | 2144 | 89.72 | 75.32 | 77.00 | 73.79 | 90.03 | 75.52 | 77.21 | 73.99 |

#### 2.7.2 Read Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6968 | 6687 | -281 | -4.03 | 6029 | 5740 | -289 | -4.79 | 939 | 947 |  +8 |  +0.81 | 86.53 | 85.85 | -0.68 | -0.79 | 90.00 | 90.53 |  +0.54 |  +0.60 | 87.39 | 86.82 | -0.57 | -0.65 | 90.57 | 91.18 |  +0.61 |  +0.67 |
| JSON_COMPACT | 9255 | 8727 | -528 | -5.71 | 8519 | 7966 | -553 | -6.49 | 736 | 761 |  +25 |  +3.43 | 92.05 | 91.28 | -0.77 | -0.84 | 85.63 | 86.97 |  +1.34 |  +1.57 | 92.18 | 91.30 | -0.88 | -0.95 | 85.71 | 86.98 |  +1.27 |  +1.48 |
| JSON_PRETTY | 14254 | 13346 | -908 | -6.37 | 13385 | 12149 | -1236 | -9.23 | 869 | 1197 |  +328 |  +37.70 | 93.90 | 91.03 | -2.87 | -3.06 | 69.26 | 70.55 |  +1.28 |  +1.85 | 93.61 | 91.28 | -2.33 | -2.49 | 69.07 | 70.71 |  +1.64 |  +2.38 |
| TOON_DEFAULT | 7018 | 11540 |  +4522 |  +64.43 | 6548 | 10654 |  +4106 |  +62.71 | 470 | 886 |  +416 |  +88.52 | 93.30 | 92.32 | -0.98 | -1.05 | 94.33 | 77.76 | -16.57 | -17.57 | 93.26 | 92.52 | -0.74 | -0.79 | 94.31 | 77.89 | -16.41 | -17.40 |
| XML_COMPACT | 11672 | 10909 | -763 | -6.54 | 10731 | 10027 | -704 | -6.56 | 941 | 882 | -59 | -6.30 | 91.94 | 91.92 | -0.02 | -0.02 | 77.04 | 79.72 |  +2.67 |  +3.47 | 91.73 | 92.14 |  +0.41 |  +0.45 | 76.90 | 79.86 |  +2.96 |  +3.85 |
| XML_PRETTY | 16137 | 15076 | -1061 | -6.57 | 14875 | 13802 | -1073 | -7.21 | 1262 | 1274 |  +12 |  +0.95 | 92.18 | 91.55 | -0.63 | -0.68 | 61.49 | 64.80 |  +3.31 |  +5.39 | 91.86 | 91.68 | -0.18 | -0.20 | 61.27 | 64.89 |  +3.61 |  +5.90 |
| YAML | 12533 | 11742 | -791 | -6.31 | 11732 | 10535 | -1197 | -10.20 | 801 | 1207 |  +406 |  +50.71 | 93.61 | 89.72 | -3.89 | -4.16 | 75.13 | 75.32 |  +0.19 |  +0.25 | 93.15 | 90.03 | -3.12 | -3.35 | 74.82 | 75.52 |  +0.70 |  +0.94 |

#### 2.7.3 Output Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 4557 | 9179 |  +4622 |  +101.43 | 3943 | 7880 |  +3937 |  +99.86 | 614 | 1299 |  +685 |  +111.57 | 86.53 | 85.85 | -0.68 | -0.79 | 90.98 | 74.18 | -16.80 | -18.47 | 87.39 | 86.82 | -0.57 | -0.65 | 91.56 | 74.83 | -16.73 | -18.27 |
| JSON_COMPACT | 11113 | 9070 | -2043 | -18.38 | 10229 | 8279 | -1950 | -19.07 | 883 | 790 | -93 | -10.48 | 92.05 | 91.28 | -0.77 | -0.84 | 71.47 | 78.19 |  +6.71 |  +9.39 | 92.18 | 91.30 | -0.88 | -0.95 | 71.56 | 78.20 |  +6.64 |  +9.28 |
| JSON_PRETTY | 13328 | 10627 | -2701 | -20.26 | 12515 | 9674 | -2841 | -22.70 | 813 | 953 |  +140 |  +17.25 | 93.90 | 91.03 | -2.87 | -3.06 | 64.87 | 72.51 |  +7.64 |  +11.78 | 93.61 | 91.28 | -2.33 | -2.49 | 64.68 | 72.68 |  +8.00 |  +12.37 |
| TOON_DEFAULT | 12285 | 11821 | -464 | -3.78 | 11462 | 10913 | -549 | -4.79 | 823 | 908 |  +85 |  +10.30 | 93.30 | 92.32 | -0.98 | -1.05 | 68.16 | 69.15 |  +0.99 |  +1.45 | 93.26 | 92.52 | -0.74 | -0.79 | 68.14 | 69.28 |  +1.15 |  +1.69 |
| XML_COMPACT | 13453 | 11401 | -2052 | -15.25 | 12369 | 10480 | -1889 | -15.27 | 1084 | 921 | -163 | -15.04 | 91.94 | 91.92 | -0.02 | -0.02 | 63.12 | 70.37 |  +7.24 |  +11.48 | 91.73 | 92.14 |  +0.41 |  +0.45 | 62.98 | 70.51 |  +7.53 |  +11.96 |
| XML_PRETTY | 9799 | 11478 |  +1679 |  +17.14 | 9033 | 10509 |  +1476 |  +16.34 | 766 | 970 |  +204 |  +26.58 | 92.18 | 91.55 | -0.63 | -0.68 | 76.21 | 69.85 | -6.36 | -8.35 | 91.86 | 91.68 | -0.18 | -0.20 | 75.99 | 69.93 | -6.06 | -7.97 |
| YAML | 11076 | 9110 | -1966 | -17.75 | 10368 | 8174 | -2194 | -21.17 | 708 | 937 |  +229 |  +32.31 | 93.61 | 89.72 | -3.89 | -4.16 | 72.64 | 77.00 |  +4.36 |  +6.00 | 93.15 | 90.03 | -3.12 | -3.35 | 72.34 | 77.21 |  +4.87 |  +6.74 |

#### 2.7.4 Total Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 11525 | 15866 |  +4341 |  +37.67 | 9973 | 13622 |  +3649 |  +36.59 | 1552 | 2245 |  +693 |  +44.63 | 86.53 | 85.85 | -0.68 | -0.79 | 91.00 | 81.54 | -9.45 | -10.39 | 87.39 | 86.82 | -0.57 | -0.65 | 91.57 | 82.19 | -9.38 | -10.24 |
| JSON_COMPACT | 20368 | 17797 | -2571 | -12.62 | 18748 | 16244 | -2504 | -13.35 | 1619 | 1552 | -67 | -4.16 | 92.05 | 91.28 | -0.77 | -0.84 | 76.34 | 81.16 |  +4.82 |  +6.31 | 92.18 | 91.30 | -0.88 | -0.95 | 76.43 | 81.18 |  +4.74 |  +6.21 |
| JSON_PRETTY | 27582 | 23973 | -3609 | -13.08 | 25900 | 21823 | -4077 | -15.74 | 1683 | 2151 |  +468 |  +27.80 | 93.90 | 91.03 | -2.87 | -3.06 | 62.62 | 68.19 |  +5.57 |  +8.89 | 93.61 | 91.28 | -2.33 | -2.49 | 62.43 | 68.36 |  +5.93 |  +9.50 |
| TOON_DEFAULT | 19303 | 23361 |  +4058 |  +21.02 | 18009 | 21566 |  +3557 |  +19.75 | 1293 | 1794 |  +501 |  +38.73 | 93.30 | 92.32 | -0.98 | -1.05 | 79.39 | 70.32 | -9.07 | -11.42 | 93.26 | 92.52 | -0.74 | -0.79 | 79.36 | 70.45 | -8.91 | -11.22 |
| XML_COMPACT | 25125 | 22310 | -2815 | -11.20 | 23100 | 20508 | -2592 | -11.22 | 2025 | 1803 | -222 | -10.98 | 91.94 | 91.92 | -0.02 | -0.02 | 66.41 | 72.23 |  +5.82 |  +8.77 | 91.73 | 92.14 |  +0.41 |  +0.45 | 66.27 | 72.38 |  +6.11 |  +9.22 |
| XML_PRETTY | 25936 | 26554 |  +618 |  +2.38 | 23908 | 24311 |  +403 |  +1.68 | 2028 | 2244 |  +216 |  +10.63 | 92.18 | 91.55 | -0.63 | -0.68 | 64.89 | 63.18 | -1.70 | -2.62 | 91.86 | 91.68 | -0.18 | -0.20 | 64.67 | 63.27 | -1.40 | -2.17 |
| YAML | 23609 | 20852 | -2757 | -11.68 | 22100 | 18708 | -3392 | -15.35 | 1509 | 2144 |  +635 |  +42.08 | 93.61 | 89.72 | -3.89 | -4.16 | 70.67 | 73.79 |  +3.12 |  +4.42 | 93.15 | 90.03 | -3.12 | -3.35 | 70.36 | 73.99 |  +3.64 |  +5.17 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Expected Characters | Output Characters | Correct Characters | Incorrect Characters | Accuracy by Character (%) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 82.33 | 41.67 | 0.00 | 66.40 | 7777.00 | 8403.33 | 7113.33 | 1290.00 | 86.53 |
| CSV | opt | 75.00 | 49.00 | 0.00 | 60.49 | 8441.00 | 9012.00 | 7718.00 | 1294.00 | 85.85 |
| JSON_COMPACT | man | 91.00 | 33.00 | 0.00 | 73.39 | 7777.00 | 7828.33 | 7630.33 | 198.00 | 92.05 |
| JSON_COMPACT | opt | 93.00 | 31.00 | 0.00 | 75.00 | 8441.00 | 8594.67 | 8291.33 | 303.33 | 91.28 |
| JSON_PRETTY | man | 97.00 | 27.00 | 0.00 | 78.23 | 7777.00 | 7811.33 | 7658.00 | 153.33 | 93.90 |
| JSON_PRETTY | opt | 94.00 | 30.00 | 0.00 | 75.81 | 8441.00 | 8584.33 | 8326.67 | 257.67 | 91.03 |
| TOON_DEFAULT | man | 94.50 | 29.50 | 0.00 | 76.21 | 7777.00 | 7977.00 | 7554.17 | 422.83 | 93.30 |
| TOON_DEFAULT | opt | 95.83 | 28.17 | 0.00 | 77.29 | 8441.00 | 8509.00 | 8338.67 | 170.33 | 92.32 |
| XML_COMPACT | man | 93.67 | 30.33 | 0.00 | 75.54 | 7777.00 | 7808.33 | 7671.00 | 137.33 | 91.94 |
| XML_COMPACT | opt | 94.00 | 30.00 | 0.00 | 75.81 | 8441.00 | 8489.00 | 8323.00 | 166.00 | 91.92 |
| XML_PRETTY | man | 93.67 | 30.33 | 0.00 | 75.54 | 7777.00 | 7975.00 | 7516.33 | 458.67 | 92.18 |
| XML_PRETTY | opt | 96.00 | 28.00 | 0.00 | 77.42 | 8441.00 | 8540.67 | 8303.67 | 237.00 | 91.55 |
| YAML | man | 96.33 | 27.67 | 0.00 | 77.69 | 7777.00 | 7843.00 | 7635.33 | 207.67 | 93.61 |
| YAML | opt | 88.00 | 36.00 | 0.00 | 70.97 | 8441.00 | 8585.00 | 8156.33 | 428.67 | 89.72 |

#### 2.8.2 Answers: Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 82.33 | 75.00 | -7 | -8.90 | 41.67 | 49.00 |  +7 |  +17.59 | 0.00 | 0.00 | 0 | 0.00 | 66.40 | 60.49 | -5.91 |
| JSON_COMPACT | 91.00 | 93.00 |  +2 |  +2.20 | 33.00 | 31.00 | -2 | -6.06 | 0.00 | 0.00 | 0 | 0.00 | 73.39 | 75.00 |  +1.61 |
| JSON_PRETTY | 97.00 | 94.00 | -3 | -3.09 | 27.00 | 30.00 |  +3 |  +11.11 | 0.00 | 0.00 | 0 | 0.00 | 78.23 | 75.81 | -2.42 |
| TOON_DEFAULT | 94.50 | 95.83 |  +1 |  +1.41 | 29.50 | 28.17 | -1 | -4.51 | 0.00 | 0.00 | 0 | 0.00 | 76.21 | 77.29 |  +1.08 |
| XML_COMPACT | 93.67 | 94.00 |  +0 |  +0.35 | 30.33 | 30.00 | -0 | -1.09 | 0.00 | 0.00 | 0 | 0.00 | 75.54 | 75.81 |  +0.27 |
| XML_PRETTY | 93.67 | 96.00 |  +2 |  +2.49 | 30.33 | 28.00 | -2 | -7.68 | 0.00 | 0.00 | 0 | 0.00 | 75.54 | 77.42 |  +1.88 |
| YAML | 96.33 | 88.00 | -8 | -8.65 | 27.67 | 36.00 |  +8 |  +30.10 | 0.00 | 0.00 | 0 | 0.00 | 77.69 | 70.97 | -6.72 |

#### 2.8.3 Characters: Mandatory vs Optional Data
| Format | Output Characters Man | Output Characters Opt | Diff | Diff (%) | Correct Characters Man | Correct Characters Opt | Diff | Diff (%) | Incorrect Characters Man | Incorrect Characters Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 8403.33 | 9012.00 |  +609 |  +7.24 | 7113.33 | 7718.00 |  +605 |  +8.50 | 1290.00 | 1294.00 |  +4 |  +0.31 | 86.53 | 85.85 | -0.68 |
| JSON_COMPACT | 7828.33 | 8594.67 |  +766 |  +9.79 | 7630.33 | 8291.33 |  +661 |  +8.66 | 198.00 | 303.33 |  +105 |  +53.20 | 92.05 | 91.28 | -0.77 |
| JSON_PRETTY | 7811.33 | 8584.33 |  +773 |  +9.90 | 7658.00 | 8326.67 |  +669 |  +8.73 | 153.33 | 257.67 |  +104 |  +68.04 | 93.90 | 91.03 | -2.87 |
| TOON_DEFAULT | 7977.00 | 8509.00 |  +532 |  +6.67 | 7554.17 | 8338.67 |  +784 |  +10.38 | 422.83 | 170.33 | -253 | -59.72 | 93.30 | 92.32 | -0.98 |
| XML_COMPACT | 7808.33 | 8489.00 |  +681 |  +8.72 | 7671.00 | 8323.00 |  +652 |  +8.50 | 137.33 | 166.00 |  +29 |  +20.87 | 91.94 | 91.92 | -0.02 |
| XML_PRETTY | 7975.00 | 8540.67 |  +566 |  +7.09 | 7516.33 | 8303.67 |  +787 |  +10.47 | 458.67 | 237.00 | -222 | -48.33 | 92.18 | 91.55 | -0.63 |
| YAML | 7843.00 | 8585.00 |  +742 |  +9.46 | 7635.33 | 8156.33 |  +521 |  +6.82 | 207.67 | 428.67 |  +221 |  +106.42 | 93.61 | 89.72 | -3.89 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 66.40 | 73.33 | 67.90 | 52.38 | 60.32 | 87.39 | 27.50 | 19.80 | 10.91 | 7.54 |
| CSV | opt | 60.49 | 76.97 | 49.38 | 57.14 | 34.92 | 86.82 | 28.86 | 14.40 | 11.90 | 4.37 |
| JSON_COMPACT | man | 73.39 | 98.79 | 56.79 | 61.90 | 39.68 | 92.18 | 37.05 | 16.56 | 12.90 | 4.96 |
| JSON_COMPACT | opt | 75.00 | 92.73 | 69.14 | 61.90 | 49.21 | 91.30 | 34.77 | 20.17 | 12.90 | 6.15 |
| JSON_PRETTY | man | 78.23 | 100.00 | 56.79 | 66.66 | 60.32 | 93.61 | 37.50 | 16.56 | 13.89 | 7.54 |
| JSON_PRETTY | opt | 75.81 | 92.73 | 72.84 | 65.08 | 46.03 | 91.28 | 34.77 | 21.24 | 13.56 | 5.76 |
| TOON_DEFAULT | man | 76.21 | 91.21 | 61.11 | 68.26 | 64.28 | 93.26 | 34.21 | 17.82 | 14.22 | 8.03 |
| TOON_DEFAULT | opt | 77.29 | 98.48 | 62.96 | 67.46 | 50.00 | 92.52 | 36.93 | 18.36 | 14.05 | 6.25 |
| XML_COMPACT | man | 75.54 | 100.00 | 55.55 | 63.49 | 49.21 | 91.73 | 37.50 | 16.20 | 13.23 | 6.15 |
| XML_COMPACT | opt | 75.81 | 100.00 | 60.49 | 65.08 | 42.86 | 92.14 | 37.50 | 17.64 | 13.56 | 5.36 |
| XML_PRETTY | man | 75.54 | 90.91 | 56.79 | 63.49 | 71.43 | 91.86 | 34.09 | 16.56 | 13.23 | 8.93 |
| XML_PRETTY | opt | 77.42 | 96.97 | 72.84 | 61.90 | 47.62 | 91.68 | 36.36 | 21.24 | 12.90 | 5.95 |
| YAML | man | 77.69 | 97.57 | 54.32 | 68.25 | 65.08 | 93.15 | 36.59 | 15.84 | 14.22 | 8.13 |
| YAML | opt | 70.97 | 90.30 | 56.79 | 63.49 | 46.03 | 90.03 | 33.87 | 16.56 | 13.23 | 5.76 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 73.33 | 76.97 |  +3.64 | 27.50 | 28.86 |  +1.36 |
| JSON_COMPACT | 98.79 | 92.73 | -6.06 | 37.05 | 34.77 | -2.27 |
| JSON_PRETTY | 100.00 | 92.73 | -7.27 | 37.50 | 34.77 | -2.73 |
| TOON_DEFAULT | 91.21 | 98.48 |  +7.27 | 34.21 | 36.93 |  +2.73 |
| XML_COMPACT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| XML_PRETTY | 90.91 | 96.97 |  +6.06 | 34.09 | 36.36 |  +2.27 |
| YAML | 97.57 | 90.30 | -7.27 | 36.59 | 33.87 | -2.73 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 67.90 | 49.38 | -18.52 | 19.80 | 14.40 | -5.40 |
| JSON_COMPACT | 56.79 | 69.14 |  +12.35 | 16.56 | 20.17 |  +3.61 |
| JSON_PRETTY | 56.79 | 72.84 |  +16.05 | 16.56 | 21.24 |  +4.68 |
| TOON_DEFAULT | 61.11 | 62.96 |  +1.85 | 17.82 | 18.36 |  +0.54 |
| XML_COMPACT | 55.55 | 60.49 |  +4.94 | 16.20 | 17.64 |  +1.44 |
| XML_PRETTY | 56.79 | 72.84 |  +16.05 | 16.56 | 21.24 |  +4.68 |
| YAML | 54.32 | 56.79 |  +2.47 | 15.84 | 16.56 |  +0.72 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 52.38 | 57.14 |  +4.76 | 10.91 | 11.90 |  +0.99 |
| JSON_COMPACT | 61.90 | 61.90 |  +0.00 | 12.90 | 12.90 | 0.00 |
| JSON_PRETTY | 66.66 | 65.08 | -1.58 | 13.89 | 13.56 | -0.33 |
| TOON_DEFAULT | 68.26 | 67.46 | -0.80 | 14.22 | 14.05 | -0.16 |
| XML_COMPACT | 63.49 | 65.08 |  +1.59 | 13.23 | 13.56 |  +0.33 |
| XML_PRETTY | 63.49 | 61.90 | -1.59 | 13.23 | 12.90 | -0.33 |
| YAML | 68.25 | 63.49 | -4.76 | 14.22 | 13.23 | -0.99 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 60.32 | 34.92 | -25.40 | 7.54 | 4.37 | -3.17 |
| JSON_COMPACT | 39.68 | 49.21 |  +9.52 | 4.96 | 6.15 |  +1.19 |
| JSON_PRETTY | 60.32 | 46.03 | -14.29 | 7.54 | 5.76 | -1.78 |
| TOON_DEFAULT | 64.28 | 50.00 | -14.29 | 8.03 | 6.25 | -1.78 |
| XML_COMPACT | 49.21 | 42.86 | -6.35 | 6.15 | 5.36 | -0.79 |
| XML_PRETTY | 71.43 | 47.62 | -23.81 | 8.93 | 5.95 | -2.97 |
| YAML | 65.08 | 46.03 | -19.04 | 8.13 | 5.76 | -2.38 |

### 2.10 Accuracy By Character Per Question Category Analysis
#### 2.10.1 Metrics
| Format | Variant | Accuracy By Character (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc By Char (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 86.53 | 83.79 | 92.07 | 89.63 | 83.51 | 87.39 | 31.42 | 26.85 | 18.67 | 10.44 |
| CSV | opt | 85.85 | 87.29 | 90.47 | 89.47 | 72.50 | 86.82 | 32.73 | 26.38 | 18.64 | 9.06 |
| JSON_COMPACT | man | 92.05 | 99.27 | 91.33 | 90.64 | 75.48 | 92.18 | 37.22 | 26.64 | 18.88 | 9.44 |
| JSON_COMPACT | opt | 91.28 | 98.42 | 90.50 | 88.68 | 76.18 | 91.30 | 36.91 | 26.40 | 18.47 | 9.52 |
| JSON_PRETTY | man | 93.90 | 100.00 | 89.27 | 94.18 | 83.59 | 93.61 | 37.50 | 26.04 | 19.62 | 10.45 |
| JSON_PRETTY | opt | 91.03 | 98.20 | 90.39 | 90.90 | 73.20 | 91.28 | 36.83 | 26.36 | 18.94 | 9.15 |
| TOON_DEFAULT | man | 93.30 | 96.93 | 91.35 | 93.75 | 85.87 | 93.26 | 36.35 | 26.64 | 19.53 | 10.73 |
| TOON_DEFAULT | opt | 92.32 | 99.83 | 90.80 | 92.86 | 74.05 | 92.52 | 37.44 | 26.48 | 19.35 | 9.25 |
| XML_COMPACT | man | 91.94 | 100.00 | 89.36 | 88.94 | 77.14 | 91.73 | 37.50 | 26.06 | 18.53 | 9.64 |
| XML_COMPACT | opt | 91.92 | 100.00 | 90.91 | 91.48 | 72.49 | 92.14 | 37.50 | 26.51 | 19.06 | 9.06 |
| XML_PRETTY | man | 92.18 | 96.46 | 88.45 | 91.80 | 86.15 | 91.86 | 36.17 | 25.80 | 19.13 | 10.77 |
| XML_PRETTY | opt | 91.55 | 99.05 | 90.79 | 90.05 | 74.37 | 91.68 | 37.14 | 26.48 | 18.76 | 9.30 |
| YAML | man | 93.61 | 98.97 | 88.69 | 92.91 | 86.60 | 93.15 | 37.11 | 25.87 | 19.35 | 10.82 |
| YAML | opt | 89.72 | 96.22 | 88.56 | 91.53 | 72.39 | 90.03 | 36.08 | 25.83 | 19.07 | 9.05 |

#### 2.10.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 83.79 | 87.29 |  +3.50 | 31.42 | 32.73 |  +1.31 |
| JSON_COMPACT | 99.27 | 98.42 | -0.84 | 37.22 | 36.91 | -0.31 |
| JSON_PRETTY | 100.00 | 98.20 | -1.80 | 37.50 | 36.83 | -0.67 |
| TOON_DEFAULT | 96.93 | 99.83 |  +2.90 | 36.35 | 37.44 |  +1.09 |
| XML_COMPACT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| XML_PRETTY | 96.46 | 99.05 |  +2.59 | 36.17 | 37.14 |  +0.97 |
| YAML | 98.97 | 96.22 | -2.75 | 37.11 | 36.08 | -1.03 |

#### 2.10.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 92.07 | 90.47 | -1.60 | 26.85 | 26.38 | -0.47 |
| JSON_COMPACT | 91.33 | 90.50 | -0.83 | 26.64 | 26.40 | -0.24 |
| JSON_PRETTY | 89.27 | 90.39 |  +1.12 | 26.04 | 26.36 |  +0.33 |
| TOON_DEFAULT | 91.35 | 90.80 | -0.55 | 26.64 | 26.48 | -0.16 |
| XML_COMPACT | 89.36 | 90.91 |  +1.55 | 26.06 | 26.51 |  +0.45 |
| XML_PRETTY | 88.45 | 90.79 |  +2.34 | 25.80 | 26.48 |  +0.68 |
| YAML | 88.69 | 88.56 | -0.13 | 25.87 | 25.83 | -0.04 |

#### 2.10.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 89.63 | 89.47 | -0.16 | 18.67 | 18.64 | -0.03 |
| JSON_COMPACT | 90.64 | 88.68 | -1.96 | 18.88 | 18.47 | -0.41 |
| JSON_PRETTY | 94.18 | 90.90 | -3.28 | 19.62 | 18.94 | -0.69 |
| TOON_DEFAULT | 93.75 | 92.86 | -0.90 | 19.53 | 19.35 | -0.19 |
| XML_COMPACT | 88.94 | 91.48 |  +2.54 | 18.53 | 19.06 |  +0.53 |
| XML_PRETTY | 91.80 | 90.05 | -1.75 | 19.13 | 18.76 | -0.37 |
| YAML | 92.91 | 91.53 | -1.38 | 19.35 | 19.07 | -0.28 |

#### 2.10.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 83.51 | 72.50 | -11.02 | 10.44 | 9.06 | -1.38 |
| JSON_COMPACT | 75.48 | 76.18 |  +0.70 | 9.44 | 9.52 |  +0.08 |
| JSON_PRETTY | 83.59 | 73.20 | -10.40 | 10.45 | 9.15 | -1.30 |
| TOON_DEFAULT | 85.87 | 74.05 | -11.83 | 10.73 | 9.25 | -1.48 |
| XML_COMPACT | 77.14 | 72.49 | -4.65 | 9.64 | 9.06 | -0.58 |
| XML_PRETTY | 86.15 | 74.37 | -11.78 | 10.77 | 9.30 | -1.47 |
| YAML | 86.60 | 72.39 | -14.21 | 10.82 | 9.05 | -1.78 |

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