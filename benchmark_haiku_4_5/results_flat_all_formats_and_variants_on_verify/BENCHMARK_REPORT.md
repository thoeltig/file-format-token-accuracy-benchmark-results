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
   - Optional: CSV 15867 tokens
   - Mandatory: CSV 11525 tokens
- Lowest read token cost:
   - Optional: CSV 6687 tokens
   - Mandatory: CSV 6968 tokens
- Lowest output token cost:
   - Optional: JSON_COMPACT 9070 tokens
   - Mandatory: CSV 4557 tokens
- Lowest output token cost drift:
   - Optional: YAML ↓ -0.93% ↑ 1.21%
   - Mandatory: YAML ↓ -12.04% ↑ 17.22%
- Highest accuracy:
   - Optional: XML_PRETTY 77.42%
   - Mandatory: JSON_PRETTY 78.23%
- Lowest accuracy drift:
   - Optional: TOON_DEFAULT ↓ -2.96% ↑ 4.35%
   - Mandatory: JSON_PRETTY ↓ -1.04% ↑ 1.02%
- Most useful read tokens:
   - Optional: XML_PRETTY 11672 / 15076 tokens
   - Mandatory: XML_PRETTY 12190 / 16137 tokens
- Most useful output tokens:
   - Optional: TOON_DEFAULT 9136 / 11821 tokens
   - Mandatory: JSON_PRETTY 10427 / 13328 tokens
- Highest read efficiency (%/token):
   - Optional: JSON_COMPACT 76.11
   - Mandatory: TOON_DEFAULT 82.93
- Highest output efficiency (%/token):
   - Optional: JSON_COMPACT 67.33
   - Mandatory: CSV 77.56
- Highest accuracy by char:
   - Optional: TOON_DEFAULT 92.32%
   - Mandatory: JSON_PRETTY 93.90%
- Lowest accuracy by char drift:
   - Optional: XML_COMPACT ↓ -0.39% ↑ 0.51%
   - Mandatory: JSON_COMPACT ↓ -0.42% ↑ 0.36%
- Most useful output write tokens (Acc By Char):
   - Optional: TOON_DEFAULT 10625 / 11509 tokens
   - Mandatory: JSON_PRETTY 12226 / 13020 tokens
- Highest output write efficiency (Acc By Char) (%/token):
   - Optional: JSON_COMPACT 78.02
   - Mandatory: CSV 90.97
- Lowest delta (optional-mandatory):
   - Read tokens: CSV -281 tokens
   - Output tokens: TOON_DEFAULT -464 tokens
   - Accuracy: XML_COMPACT 0.27%
   - Read efficiency: JSON_PRETTY 1.58
   - Output efficiency: TOON_DEFAULT 2.36
   - Accuracy by char: XML_COMPACT -0.02%
   - Output write efficiency (Acc By Char): TOON_DEFAULT 1.02

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 26555 tokens
   - Mandatory: JSON_PRETTY 27582 tokens
- Highest read token cost:
   - Optional: XML_PRETTY 15076 tokens
   - Mandatory: XML_PRETTY 16137 tokens
- Highest output token cost:
   - Optional: TOON_DEFAULT 11821 tokens
   - Mandatory: XML_COMPACT 13453 tokens
- Highest output token drift:
   - Optional: JSON_PRETTY ↓ -50.40% ↑ 63.06%
   - Mandatory: CSV ↓ -99.69% ↑ 193.15%
- Lowest accuracy:
   - Optional: CSV 60.49%
   - Mandatory: CSV 66.40%
- Highest accuracy drift:
   - Optional: JSON_COMPACT ↓ -9.68% ↑ 7.53%
   - Mandatory: TOON_DEFAULT ↓ -18.51% ↑ 11.11%
- Most wasted read tokens:
   - Optional: YAML 3409 / 11742 tokens
   - Mandatory: XML_PRETTY 3947 / 16137 tokens
- Most wasted output tokens:
   - Optional: CSV 3627 / 9180 tokens
   - Mandatory: XML_COMPACT 3291 / 13453 tokens
- Lowest read efficiency (%/token):
   - Optional: XML_PRETTY 55.38
   - Mandatory: XML_PRETTY 50.39
- Lowest output efficiency (%/token):
   - Optional: CSV 57.27
   - Mandatory: XML_COMPACT 52.18
- Lowest accuracy by char:
   - Optional: CSV 85.85%
   - Mandatory: CSV 86.53%
- Highest accuracy by char drift:
   - Optional: JSON_COMPACT ↓ -2.72% ↑ 1.76%
   - Mandatory: TOON_DEFAULT ↓ -4.45% ↑ 2.14%
- Most wasted output write tokens (Acc By Char):
   - Optional: CSV 7622 / 8879 tokens
   - Mandatory: XML_COMPACT 12177 / 13245 tokens
- Lowest output write efficiency (Acc By Char) (%/token):
   - Optional: TOON_DEFAULT 69.24
   - Mandatory: XML_COMPACT 62.78
- Highest delta (optional-mandatory):
   - Read tokens: TOON_DEFAULT 4522 tokens
   - Output tokens: CSV 4622 tokens
   - Accuracy: YAML -6.72%
   - Read efficiency: TOON_DEFAULT -15.19
   - Output efficiency: CSV -20.29
   - Accuracy by char: YAML -3.89%
   - Output write efficiency (Acc By Char): CSV -16.64

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy By Char | ↓ Eff Score Output Write (Acc By Char) | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|
| XML_PRETTY ≈ 85s | CSV ≈ 6968 | CSV ≈ 204 | CSV ≈ 4353 | CSV ≈ 4557 | CSV ≈ 11525 | JSON_PRETTY ≈ 94% | CSV ≈ 91 | JSON_PRETTY ≈ 78% | TOON_DEFAULT ≈ 83 | CSV ≈ 78 | CSV ≈ 78 |
| YAML (+8.5%) | TOON_DEFAULT (+0.7%) | XML_COMPACT (+2.3%) | XML_PRETTY (+118.1%) | XML_PRETTY (+115.0%) | TOON_DEFAULT (+67.5%) | YAML (-0.3%) | XML_PRETTY (-16.1%) | YAML (-0.5%) | CSV (-7.7%) | XML_PRETTY (-16.1%) | TOON_DEFAULT (-12.4%) |
| JSON_COMPACT (+10.9%) | JSON_COMPACT (+32.8%) | JSON_COMPACT (+4.2%) | YAML (+147.5%) | YAML (+143.0%) | JSON_COMPACT (+76.7%) | TOON_DEFAULT (-0.6%) | YAML (-20.0%) | TOON_DEFAULT (-2.0%) | JSON_COMPACT (-11.8%) | YAML (-20.0%) | JSON_COMPACT (-17.6%) |
| TOON_DEFAULT (+18.1%) | XML_COMPACT (+67.5%) | XML_PRETTY (+48.9%) | JSON_COMPACT (+150.4%) | JSON_COMPACT (+143.8%) | YAML (+104.8%) | XML_PRETTY (-1.7%) | JSON_COMPACT (-21.7%) | XML_COMPACT (-2.7%) | XML_COMPACT (-20.3%) | JSON_COMPACT (-23.9%) | YAML (-22.6%) |
| JSON_PRETTY (+27.2%) | YAML (+79.9%) | YAML (+48.9%) | TOON_DEFAULT (+175.1%) | TOON_DEFAULT (+169.6%) | XML_COMPACT (+118.0%) | JSON_COMPACT (-1.9%) | TOON_DEFAULT (-25.0%) | XML_PRETTY (-2.7%) | YAML (-22.2%) | TOON_DEFAULT (-26.8%) | XML_COMPACT (-28.5%) |
| XML_COMPACT (+30.7%) | JSON_PRETTY (+104.6%) | JSON_PRETTY (+51.0%) | JSON_PRETTY (+199.1%) | JSON_PRETTY (+192.5%) | XML_PRETTY (+125.0%) | XML_COMPACT (-2.0%) | JSON_PRETTY (-28.7%) | JSON_COMPACT (-4.8%) | JSON_PRETTY (-29.1%) | JSON_PRETTY (-29.8%) | XML_PRETTY (-30.7%) |
| CSV (+49.4%) | XML_PRETTY (+131.6%) | TOON_DEFAULT (+51.4%) | XML_COMPACT (+204.2%) | XML_COMPACT (+195.2%) | JSON_PRETTY (+139.3%) | CSV (-7.4%) | XML_COMPACT (-31.0%) | CSV (-11.8%) | XML_PRETTY (-39.2%) | XML_COMPACT (-32.7%) | JSON_PRETTY (-32.7%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy By Char | ↓ Eff Score Output Write (Acc By Char) | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 69s | CSV ≈ 6687 | JSON_COMPACT ≈ 208 | YAML ≈ 8803 | JSON_COMPACT ≈ 9070 | CSV ≈ 15867 | TOON_DEFAULT ≈ 92% | JSON_COMPACT ≈ 78 | XML_PRETTY ≈ 77% | JSON_COMPACT ≈ 76 | JSON_COMPACT ≈ 67 | JSON_COMPACT ≈ 70 |
| JSON_COMPACT (+10.0%) | JSON_COMPACT (+30.5%) | XML_PRETTY (+0.5%) | JSON_COMPACT (+0.7%) | YAML (+0.4%) | JSON_COMPACT (+12.2%) | XML_COMPACT (-0.4%) | YAML (-1.1%) | TOON_DEFAULT (-0.1%) | CSV (-3.3%) | YAML (-4.2%) | CSV (-8.1%) |
| CSV (+10.6%) | XML_COMPACT (+63.1%) | CSV (+44.5%) | CSV (+0.9%) | CSV (+1.2%) | YAML (+31.4%) | XML_PRETTY (-0.8%) | CSV (-4.7%) | JSON_PRETTY (-1.6%) | XML_COMPACT (-9.4%) | JSON_PRETTY (-7.4%) | XML_COMPACT (-12.5%) |
| JSON_PRETTY (+24.2%) | TOON_DEFAULT (+72.6%) | JSON_PRETTY (+45.6%) | JSON_PRETTY (+17.3%) | JSON_PRETTY (+17.2%) | XML_COMPACT (+40.6%) | JSON_COMPACT (-1.0%) | JSON_PRETTY (-6.9%) | XML_COMPACT (-1.6%) | TOON_DEFAULT (-11.0%) | XML_PRETTY (-10.3%) | YAML (-12.8%) |
| XML_PRETTY (+26.5%) | YAML (+75.6%) | YAML (+47.7%) | XML_COMPACT (+26.0%) | XML_COMPACT (+25.7%) | TOON_DEFAULT (+47.2%) | JSON_PRETTY (-1.3%) | XML_COMPACT (-9.7%) | JSON_COMPACT (-2.4%) | YAML (-17.5%) | XML_COMPACT (-11.4%) | TOON_DEFAULT (-14.2%) |
| XML_COMPACT (+33.1%) | JSON_PRETTY (+99.6%) | XML_COMPACT (+48.2%) | XML_PRETTY (+28.0%) | XML_PRETTY (+26.6%) | JSON_PRETTY (+51.1%) | YAML (-2.6%) | XML_PRETTY (-10.8%) | YAML (-6.5%) | JSON_PRETTY (-20.7%) | TOON_DEFAULT (-12.2%) | JSON_PRETTY (-17.4%) |
| TOON_DEFAULT (+36.7%) | XML_PRETTY (+125.5%) | TOON_DEFAULT (+49.6%) | TOON_DEFAULT (+30.7%) | TOON_DEFAULT (+30.3%) | XML_PRETTY (+67.4%) | CSV (-6.5%) | TOON_DEFAULT (-11.2%) | CSV (-16.9%) | XML_PRETTY (-27.2%) | CSV (-14.9%) | XML_PRETTY (-23.5%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 100% | CSV ≈ 68% | TOON_DEFAULT ≈ 68% | XML_PRETTY ≈ 71% |
| XML_COMPACT (0.0%) | TOON_DEFAULT (-6.8%) | YAML (-0.0%) | YAML (-6.3%) |
| JSON_COMPACT (-1.2%) | JSON_COMPACT (-11.1%) | JSON_PRETTY (-1.6%) | TOON_DEFAULT (-7.1%) |
| YAML (-2.4%) | JSON_PRETTY (-11.1%) | XML_PRETTY (-4.8%) | CSV (-11.1%) |
| TOON_DEFAULT (-8.8%) | XML_PRETTY (-11.1%) | XML_COMPACT (-4.8%) | JSON_PRETTY (-11.1%) |
| XML_PRETTY (-9.1%) | XML_COMPACT (-12.3%) | JSON_COMPACT (-6.4%) | XML_COMPACT (-22.2%) |
| CSV (-26.7%) | YAML (-13.6%) | CSV (-15.9%) | JSON_COMPACT (-31.7%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| XML_COMPACT ≈ 100% | JSON_PRETTY ≈ 73% | TOON_DEFAULT ≈ 67% | TOON_DEFAULT ≈ 50% |
| TOON_DEFAULT (-1.5%) | XML_PRETTY (0.0%) | JSON_PRETTY (-2.4%) | JSON_COMPACT (-0.8%) |
| XML_PRETTY (-3.0%) | JSON_COMPACT (-3.7%) | XML_COMPACT (-2.4%) | XML_PRETTY (-2.4%) |
| JSON_COMPACT (-7.3%) | TOON_DEFAULT (-9.9%) | YAML (-4.0%) | YAML (-4.0%) |
| JSON_PRETTY (-7.3%) | XML_COMPACT (-12.3%) | JSON_COMPACT (-5.6%) | JSON_PRETTY (-4.0%) |
| YAML (-9.7%) | YAML (-16.1%) | XML_PRETTY (-5.6%) | XML_COMPACT (-7.1%) |
| CSV (-23.0%) | CSV (-23.5%) | CSV (-10.3%) | CSV (-15.1%) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy By Char (%) | Useful Output Write Tokens (Acc By Char) | Wasted Output Write Tokens (Acc By Char) | Eff Score Output Write (Acc By Char) | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6968 | 4557 | 11525 | 1.449 | 35.108 | 86.53 | 3766.94 | 586.39 | 90.97 | 66.40 | 4626.752 | 2341.248 | 3026.069 | 1531.264 | 76.57 | 77.56 | 77.57 |
| CSV | opt | 6687 | 9180 | 15867 | 1.432 | 71.602 | 85.85 | 7622.34 | 1256.33 | 74.34 | 60.49 | 4044.966 | 2642.034 | 5552.781 | 3626.886 | 73.62 | 57.27 | 64.63 |
| JSON_COMPACT | man | 9255 | 11113 | 20368 | 2.152 | 87.903 | 92.05 | 10033.45 | 866.55 | 71.24 | 73.39 | 6792.245 | 2462.756 | 8155.586 | 2957.081 | 73.18 | 59.03 | 63.90 |
| JSON_COMPACT | opt | 8727 | 9070 | 17797 | 2.118 | 71.462 | 91.28 | 8088.63 | 772.71 | 78.02 | 75.00 | 6545.250 | 2181.750 | 6802.250 | 2267.417 | 76.11 | 67.33 | 70.30 |
| JSON_PRETTY | man | 14254 | 13328 | 27582 | 1.697 | 105.003 | 93.90 | 12226.09 | 794.24 | 64.89 | 78.23 | 11150.904 | 3103.096 | 10426.755 | 2901.578 | 58.81 | 54.42 | 52.17 |
| JSON_PRETTY | opt | 13346 | 10628 | 23974 | 1.683 | 83.261 | 91.03 | 9398.24 | 926.09 | 72.62 | 75.81 | 10117.603 | 3228.397 | 8056.834 | 2570.833 | 60.39 | 62.36 | 58.04 |
| TOON_DEFAULT | man | 7018 | 12285 | 19303 | 1.448 | 96.579 | 93.30 | 11173.45 | 802.38 | 68.22 | 76.21 | 5348.418 | 1669.582 | 9362.208 | 2922.542 | 82.93 | 56.76 | 67.99 |
| TOON_DEFAULT | opt | 11540 | 11821 | 23361 | 1.701 | 92.815 | 92.32 | 10625.11 | 883.89 | 69.24 | 77.29 | 8919.266 | 2620.734 | 9136.193 | 2684.474 | 67.74 | 59.13 | 60.29 |
| XML_COMPACT | man | 11672 | 13453 | 25125 | 2.370 | 106.812 | 91.94 | 12177.15 | 1067.52 | 62.78 | 75.54 | 8817.029 | 2854.971 | 10162.648 | 3290.685 | 66.11 | 52.18 | 55.47 |
| XML_COMPACT | opt | 10909 | 11402 | 22311 | 2.345 | 89.460 | 91.92 | 10196.69 | 896.31 | 70.46 | 75.81 | 8270.113 | 2638.887 | 8643.604 | 2758.063 | 68.97 | 59.62 | 61.48 |
| XML_PRETTY | man | 16137 | 9799 | 25936 | 1.937 | 76.578 | 92.18 | 8753.11 | 742.56 | 76.35 | 75.54 | 12189.890 | 3947.110 | 7402.416 | 2396.917 | 50.39 | 65.11 | 53.79 |
| XML_PRETTY | opt | 15076 | 11479 | 26555 | 1.918 | 90.882 | 91.55 | 10317.07 | 952.26 | 69.58 | 77.42 | 11671.839 | 3404.161 | 8886.784 | 2591.883 | 55.38 | 60.42 | 53.76 |
| YAML | man | 12533 | 11076 | 23609 | 1.666 | 86.874 | 93.61 | 10083.98 | 688.35 | 72.74 | 77.69 | 9736.888 | 2796.112 | 8604.944 | 2471.056 | 64.51 | 62.03 | 60.05 |
| YAML | opt | 11742 | 9110 | 20852 | 1.653 | 70.989 | 89.72 | 7897.75 | 904.91 | 77.19 | 70.97 | 8333.297 | 3408.703 | 6465.603 | 2644.730 | 62.81 | 64.50 | 61.28 |

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
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 23 | 302.957 | 0.74 | 52278 | 0.083 | 421.59 | 52301 | 303.040 | 337.42 |
| CSV | opt | 14 | 477.643 | 0.45 | 39364 | 0.226 | 317.45 | 39378 | 477.869 | 254.05 |
| JSON_COMPACT | man | 24 | 385.625 | 0.77 | 39580 | 0.275 | 319.19 | 39604 | 385.900 | 255.51 |
| JSON_COMPACT | opt | 13 | 671.308 | 0.42 | 37630 | 0.235 | 303.47 | 37643 | 671.543 | 242.86 |
| JSON_PRETTY | man | 30 | 475.133 | 0.97 | 39401 | 0.330 | 317.75 | 39431 | 475.463 | 254.40 |
| JSON_PRETTY | opt | 36 | 370.722 | 1.16 | 40232 | 0.257 | 324.45 | 40268 | 370.979 | 259.79 |
| TOON_DEFAULT | man | 26 | 269.923 | 0.84 | 38099 | 0.313 | 307.25 | 38125 | 270.236 | 245.97 |
| TOON_DEFAULT | opt | 20 | 577.000 | 0.65 | 37064 | 0.311 | 298.90 | 37084 | 577.312 | 239.25 |
| XML_COMPACT | man | 19 | 614.316 | 0.61 | 37753 | 0.351 | 304.46 | 37772 | 614.667 | 243.69 |
| XML_COMPACT | opt | 23 | 474.304 | 0.74 | 36545 | 0.304 | 294.72 | 36568 | 474.608 | 235.92 |
| XML_PRETTY | man | 18 | 896.500 | 0.58 | 39623 | 0.240 | 319.54 | 39641 | 896.740 | 255.75 |
| XML_PRETTY | opt | 17 | 886.824 | 0.55 | 30600 | 0.368 | 246.78 | 30617 | 887.192 | 197.53 |
| YAML | man | 21 | 596.810 | 0.68 | 38458 | 0.280 | 310.15 | 38479 | 597.090 | 248.25 |
| YAML | opt | 31 | 378.774 | 1.00 | 30155 | 0.292 | 243.19 | 30186 | 379.066 | 194.75 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 23 | 14 | -9 | -39.13 | 52.28 | 39.36 | -12.91 | -24.70 | 52.30 | 39.38 | -12.92 | -24.71 |
| JSON_COMPACT | 24 | 13 | -11 | -45.83 | 39.58 | 37.63 | -1.95 | -4.93 | 39.60 | 37.64 | -1.96 | -4.95 |
| JSON_PRETTY | 30 | 36 |  +6 |  +20.00 | 39.40 | 40.23 |  +0.83 |  +2.11 | 39.43 | 40.27 |  +0.84 |  +2.12 |
| TOON_DEFAULT | 26 | 20 | -6 | -23.08 | 38.10 | 37.06 | -1.03 | -2.72 | 38.12 | 37.08 | -1.04 | -2.73 |
| XML_COMPACT | 19 | 23 |  +4 |  +21.05 | 37.75 | 36.54 | -1.21 | -3.20 | 37.77 | 36.57 | -1.20 | -3.19 |
| XML_PRETTY | 18 | 17 | -1 | -5.56 | 39.62 | 30.60 | -9.02 | -22.77 | 39.64 | 30.62 | -9.02 | -22.76 |
| YAML | 21 | 31 |  +10 |  +47.62 | 38.46 | 30.15 | -8.30 | -21.59 | 38.48 | 30.19 | -8.29 | -21.55 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token |
|---|---|---|---|---|---|---|---|
| CSV | man | 1.449 | 10.217 | 224.774 | 0.953 | 1.457 | 0.576 |
| CSV | opt | 1.432 | 10.597 | 215.710 | 0.905 | 0.659 | 0.381 |
| JSON_COMPACT | man | 2.152 | 13.570 | 298.548 | 0.793 | 0.660 | 0.360 |
| JSON_COMPACT | opt | 2.118 | 13.830 | 281.516 | 0.859 | 0.827 | 0.421 |
| JSON_PRETTY | man | 1.697 | 20.900 | 459.806 | 0.549 | 0.587 | 0.284 |
| JSON_PRETTY | opt | 1.683 | 21.151 | 430.516 | 0.568 | 0.713 | 0.316 |
| TOON_DEFAULT | man | 1.448 | 10.290 | 226.387 | 1.086 | 0.632 | 0.397 |
| TOON_DEFAULT | opt | 1.701 | 18.288 | 372.258 | 0.670 | 0.656 | 0.331 |
| XML_COMPACT | man | 2.370 | 17.114 | 376.516 | 0.647 | 0.561 | 0.301 |
| XML_COMPACT | opt | 2.345 | 17.288 | 351.903 | 0.695 | 0.665 | 0.340 |
| XML_PRETTY | man | 1.937 | 23.661 | 520.548 | 0.468 | 0.771 | 0.291 |
| XML_PRETTY | opt | 1.918 | 23.892 | 486.323 | 0.514 | 0.674 | 0.292 |
| YAML | man | 1.666 | 18.377 | 404.290 | 0.620 | 0.701 | 0.329 |
| YAML | opt | 1.653 | 18.609 | 378.774 | 0.604 | 0.779 | 0.340 |

#### 2.5.2 Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.449 | 1.432 | -0.017 | -1.17 | 10.217 | 10.597 |  +0.380 |  +3.72 | 224.774 | 215.710 | -9.064 | -4.03 | 0.953 | 0.905 | -0.048 | -5.04 | 1.457 | 0.659 | -0.798 | -54.77 | 0.576 | 0.381 | -0.195 | -33.85 |
| JSON_COMPACT | 2.152 | 2.118 | -0.034 | -1.58 | 13.570 | 13.830 |  +0.260 |  +1.92 | 298.548 | 281.516 | -17.032 | -5.70 | 0.793 | 0.859 |  +0.066 |  +8.32 | 0.660 | 0.827 |  +0.167 |  +25.30 | 0.360 | 0.421 |  +0.061 |  +16.94 |
| JSON_PRETTY | 1.697 | 1.683 | -0.014 | -0.82 | 20.900 | 21.151 |  +0.251 |  +1.20 | 459.806 | 430.516 | -29.290 | -6.37 | 0.549 | 0.568 |  +0.019 |  +3.46 | 0.587 | 0.713 |  +0.126 |  +21.47 | 0.284 | 0.316 |  +0.032 |  +11.27 |
| TOON_DEFAULT | 1.448 | 1.701 |  +0.253 |  +17.47 | 10.290 | 18.288 |  +7.998 |  +77.73 | 226.387 | 372.258 |  +145.871 |  +64.43 | 1.086 | 0.670 | -0.416 | -38.31 | 0.632 | 0.656 |  +0.024 |  +3.88 | 0.397 | 0.331 | -0.066 | -16.73 |
| XML_COMPACT | 2.370 | 2.345 | -0.025 | -1.05 | 17.114 | 17.288 |  +0.174 |  +1.02 | 376.516 | 351.903 | -24.613 | -6.54 | 0.647 | 0.695 |  +0.048 |  +7.42 | 0.561 | 0.665 |  +0.104 |  +18.54 | 0.301 | 0.340 |  +0.039 |  +12.96 |
| XML_PRETTY | 1.937 | 1.918 | -0.019 | -0.98 | 23.661 | 23.892 |  +0.231 |  +0.98 | 520.548 | 486.323 | -34.225 | -6.57 | 0.468 | 0.514 |  +0.046 |  +9.83 | 0.771 | 0.674 | -0.097 | -12.58 | 0.291 | 0.292 |  +0.001 |  +0.34 |
| YAML | 1.666 | 1.653 | -0.013 | -0.78 | 18.377 | 18.609 |  +0.232 |  +1.26 | 404.290 | 378.774 | -25.516 | -6.31 | 0.620 | 0.604 | -0.016 | -2.58 | 0.701 | 0.779 |  +0.078 |  +11.13 | 0.329 | 0.340 |  +0.011 |  +3.34 |

### 2.6 Output Write Token Utilization Efficiency (Accuracy By Char)
#### 2.6.1 Metrics
| Format | Variant | Output Write Tokens | Useful Output Write Tokens (Acc By Char) | Wasted Output Write Tokens (Acc By Char) | Accuracy by Char (%) | Eff Score Output Write (Acc By Char) |
|---|---|---|---|---|---|---|
| CSV | man | 4353 | 3767 | 586 | 86.53 | 90.97 |
| CSV | opt | 8879 | 7622 | 1256 | 85.85 | 74.34 |
| JSON_COMPACT | man | 10900 | 10033 | 867 | 92.05 | 71.24 |
| JSON_COMPACT | opt | 8861 | 8089 | 773 | 91.28 | 78.02 |
| JSON_PRETTY | man | 13020 | 12226 | 794 | 93.90 | 64.89 |
| JSON_PRETTY | opt | 10324 | 9398 | 926 | 91.03 | 72.62 |
| TOON_DEFAULT | man | 11976 | 11173 | 802 | 93.30 | 68.22 |
| TOON_DEFAULT | opt | 11509 | 10625 | 884 | 92.32 | 69.24 |
| XML_COMPACT | man | 13245 | 12177 | 1068 | 91.94 | 62.78 |
| XML_COMPACT | opt | 11093 | 10197 | 896 | 91.92 | 70.46 |
| XML_PRETTY | man | 9496 | 8753 | 743 | 92.18 | 76.35 |
| XML_PRETTY | opt | 11269 | 10317 | 952 | 91.55 | 69.58 |
| YAML | man | 10772 | 10084 | 688 | 93.61 | 72.74 |
| YAML | opt | 8803 | 7898 | 905 | 89.72 | 77.19 |

#### 2.6.2 Mandatory vs Optional
| Format | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Useful Output Write Tokens (Acc By Char) Man | Useful Output Write Tokens (Acc By Char) Opt | Diff | Diff (%) | Wasted Output Write Tokens (Acc By Char) Man | Wasted Output Write Tokens (Acc By Char) Opt | Diff | Diff (%) | Accuracy By Char (%) Man | Accuracy By Char (%) Opt | Diff (%) | Eff Score Output Write (Acc By Char) Man | Eff Score Output Write (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 4353 | 8878 |  +4525 |  +103.96 | 3767 | 7622 |  +3855 |  +102.35 | 586 | 1256 |  +670 |  +114.32 | 86.53 | 85.85 | -0.68 | -0.79 | 90.97 | 74.34 | -16.64 | -18.29 |
| JSON_COMPACT | 10900 | 8861 | -2039 | -18.70 | 10033 | 8088 | -1945 | -19.38 | 867 | 773 | -94 | -10.82 | 92.05 | 91.28 | -0.77 | -0.84 | 71.24 | 78.02 |  +6.78 |  +9.52 |
| JSON_PRETTY | 13020 | 10324 | -2696 | -20.71 | 12226 | 9398 | -2828 | -23.13 | 794 | 926 |  +132 |  +16.61 | 93.90 | 91.03 | -2.87 | -3.06 | 64.89 | 72.62 |  +7.73 |  +11.91 |
| TOON_DEFAULT | 11976 | 11509 | -467 | -3.90 | 11173 | 10625 | -548 | -4.91 | 802 | 884 |  +82 |  +10.16 | 93.30 | 92.32 | -0.98 | -1.05 | 68.22 | 69.24 |  +1.02 |  +1.49 |
| XML_COMPACT | 13245 | 11093 | -2152 | -16.25 | 12177 | 10197 | -1980 | -16.26 | 1068 | 897 | -171 | -16.03 | 91.94 | 91.92 | -0.02 | -0.02 | 62.78 | 70.46 |  +7.68 |  +12.24 |
| XML_PRETTY | 9496 | 11270 |  +1774 |  +18.68 | 8753 | 10317 |  +1564 |  +17.87 | 743 | 953 |  +210 |  +28.22 | 92.18 | 91.55 | -0.63 | -0.68 | 76.35 | 69.58 | -6.76 | -8.86 |
| YAML | 10772 | 8802 | -1970 | -18.29 | 10084 | 7898 | -2186 | -21.68 | 688 | 905 |  +217 |  +31.48 | 93.61 | 89.72 | -3.89 | -4.16 | 72.74 | 77.19 |  +4.45 |  +6.12 |

### 2.7 Token Utilization Efficiency
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6968 | 4627 | 2341 | 4557 | 3026 | 1531 | 11525 | 7653 | 3873 | 66.40 | 76.57 | 77.56 | 77.57 | 65.76 | 76.14 | 77.13 | 77.14 |
| CSV | opt | 6687 | 4045 | 2642 | 9180 | 5553 | 3627 | 15867 | 9598 | 6269 | 60.49 | 73.62 | 57.27 | 64.63 | 59.53 | 72.98 | 56.63 | 63.99 |
| JSON_COMPACT | man | 9255 | 6792 | 2463 | 11113 | 8156 | 2957 | 20368 | 14948 | 5420 | 73.39 | 73.18 | 59.03 | 63.90 | 71.47 | 71.90 | 57.75 | 62.62 |
| JSON_COMPACT | opt | 8727 | 6545 | 2182 | 9070 | 6802 | 2267 | 17797 | 13348 | 4449 | 75.00 | 76.11 | 67.33 | 70.30 | 73.99 | 75.44 | 66.66 | 69.63 |
| JSON_PRETTY | man | 14254 | 11151 | 3103 | 13328 | 10427 | 2902 | 27582 | 21578 | 6005 | 78.23 | 58.81 | 54.42 | 52.17 | 75.49 | 56.98 | 52.59 | 50.34 |
| JSON_PRETTY | opt | 13346 | 10118 | 3228 | 10628 | 8057 | 2571 | 23974 | 18174 | 5799 | 75.81 | 60.39 | 62.36 | 58.04 | 75.33 | 60.07 | 62.04 | 57.72 |
| TOON_DEFAULT | man | 7018 | 5348 | 1670 | 12285 | 9362 | 2923 | 19303 | 14711 | 4592 | 76.21 | 82.93 | 56.76 | 67.99 | 74.28 | 81.64 | 55.48 | 66.70 |
| TOON_DEFAULT | opt | 11540 | 8919 | 2621 | 11821 | 9136 | 2684 | 23361 | 18055 | 5305 | 77.29 | 67.74 | 59.13 | 60.29 | 75.60 | 66.61 | 58.00 | 59.17 |
| XML_COMPACT | man | 11672 | 8817 | 2855 | 13453 | 10163 | 3291 | 25125 | 18980 | 6146 | 75.54 | 66.11 | 52.18 | 55.47 | 73.08 | 64.47 | 50.54 | 53.83 |
| XML_COMPACT | opt | 10909 | 8270 | 2639 | 11402 | 8644 | 2758 | 22311 | 16914 | 5397 | 75.81 | 68.97 | 59.62 | 61.48 | 74.06 | 67.80 | 58.45 | 60.32 |
| XML_PRETTY | man | 16137 | 12190 | 3947 | 9799 | 7402 | 2397 | 25936 | 19592 | 6344 | 75.54 | 50.39 | 65.11 | 53.79 | 72.81 | 48.57 | 63.29 | 51.97 |
| XML_PRETTY | opt | 15076 | 11672 | 3404 | 11479 | 8887 | 2592 | 26555 | 20559 | 5996 | 77.42 | 55.38 | 60.42 | 53.76 | 76.46 | 54.74 | 59.78 | 53.12 |
| YAML | man | 12533 | 9737 | 2796 | 11076 | 8605 | 2471 | 23609 | 18342 | 5267 | 77.69 | 64.51 | 62.03 | 60.05 | 74.78 | 62.57 | 60.09 | 58.11 |
| YAML | opt | 11742 | 8333 | 3409 | 9110 | 6466 | 2645 | 20852 | 14799 | 6053 | 70.97 | 62.81 | 64.50 | 61.28 | 69.41 | 61.77 | 63.46 | 60.24 |

#### 2.7.2 Read Tokens Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6968 | 6687 | -281 | -4.03 | 4627 | 4045 | -582 | -12.57 | 2341 | 2642 |  +301 |  +12.85 | 66.40 | 60.49 | -5.91 | -8.90 | 76.57 | 73.62 | -2.95 | -3.85 | 65.76 | 59.53 | -6.23 | -9.47 | 76.14 | 72.98 | -3.16 | -4.15 |
| JSON_COMPACT | 9255 | 8727 | -528 | -5.71 | 6792 | 6545 | -247 | -3.64 | 2463 | 2182 | -281 | -11.41 | 73.39 | 75.00 |  +1.61 |  +2.19 | 73.18 | 76.11 |  +2.93 |  +4.01 | 71.47 | 73.99 |  +2.52 |  +3.53 | 71.90 | 75.44 |  +3.54 |  +4.92 |
| JSON_PRETTY | 14254 | 13346 | -908 | -6.37 | 11151 | 10118 | -1033 | -9.27 | 3103 | 3228 |  +125 |  +4.04 | 78.23 | 75.81 | -2.42 | -3.09 | 58.81 | 60.39 |  +1.58 |  +2.69 | 75.49 | 75.33 | -0.16 | -0.21 | 56.98 | 60.07 |  +3.09 |  +5.42 |
| TOON_DEFAULT | 7018 | 11540 |  +4522 |  +64.43 | 5348 | 8919 |  +3571 |  +66.77 | 1670 | 2621 |  +951 |  +56.96 | 76.21 | 77.29 |  +1.08 |  +1.42 | 82.93 | 67.74 | -15.19 | -18.32 | 74.28 | 75.60 |  +1.32 |  +1.78 | 81.64 | 66.61 | -15.04 | -18.42 |
| XML_COMPACT | 11672 | 10909 | -763 | -6.54 | 8817 | 8270 | -547 | -6.20 | 2855 | 2639 | -216 | -7.57 | 75.54 | 75.81 |  +0.27 |  +0.36 | 66.11 | 68.97 |  +2.86 |  +4.33 | 73.08 | 74.06 |  +0.98 |  +1.34 | 64.47 | 67.80 |  +3.34 |  +5.18 |
| XML_PRETTY | 16137 | 15076 | -1061 | -6.57 | 12190 | 11672 | -518 | -4.25 | 3947 | 3404 | -543 | -13.76 | 75.54 | 77.42 |  +1.88 |  +2.49 | 50.39 | 55.38 |  +4.99 |  +9.90 | 72.81 | 76.46 |  +3.65 |  +5.01 | 48.57 | 54.74 |  +6.17 |  +12.70 |
| YAML | 12533 | 11742 | -791 | -6.31 | 9737 | 8333 | -1404 | -14.42 | 2796 | 3409 |  +613 |  +21.91 | 77.69 | 70.97 | -6.72 | -8.65 | 64.51 | 62.81 | -1.70 | -2.63 | 74.78 | 69.41 | -5.37 | -7.18 | 62.57 | 61.77 | -0.80 | -1.27 |

#### 2.7.3 Output Tokens Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 4557 | 9179 |  +4622 |  +101.43 | 3026 | 5553 |  +2527 |  +83.50 | 1531 | 3627 |  +2096 |  +136.88 | 66.40 | 60.49 | -5.91 | -8.90 | 77.56 | 57.27 | -20.29 | -26.16 | 65.76 | 59.53 | -6.23 | -9.47 | 77.13 | 56.63 | -20.50 | -26.58 |
| JSON_COMPACT | 11113 | 9070 | -2043 | -18.38 | 8156 | 6803 | -1353 | -16.59 | 2957 | 2267 | -690 | -23.32 | 73.39 | 75.00 |  +1.61 |  +2.19 | 59.03 | 67.33 |  +8.30 |  +14.06 | 71.47 | 73.99 |  +2.52 |  +3.53 | 57.75 | 66.66 |  +8.91 |  +15.42 |
| JSON_PRETTY | 13328 | 10627 | -2701 | -20.26 | 10427 | 8057 | -2370 | -22.73 | 2902 | 2571 | -331 | -11.40 | 78.23 | 75.81 | -2.42 | -3.09 | 54.42 | 62.36 |  +7.94 |  +14.59 | 75.49 | 75.33 | -0.16 | -0.21 | 52.59 | 62.04 |  +9.45 |  +17.96 |
| TOON_DEFAULT | 12285 | 11821 | -464 | -3.78 | 9362 | 9136 | -226 | -2.41 | 2923 | 2685 | -238 | -8.14 | 76.21 | 77.29 |  +1.08 |  +1.42 | 56.76 | 59.13 |  +2.36 |  +4.16 | 74.28 | 75.60 |  +1.32 |  +1.78 | 55.48 | 58.00 |  +2.52 |  +4.55 |
| XML_COMPACT | 13453 | 11401 | -2052 | -15.25 | 10163 | 8644 | -1519 | -14.95 | 3291 | 2758 | -533 | -16.18 | 75.54 | 75.81 |  +0.27 |  +0.36 | 52.18 | 59.62 |  +7.44 |  +14.25 | 73.08 | 74.06 |  +0.98 |  +1.34 | 50.54 | 58.45 |  +7.91 |  +15.65 |
| XML_PRETTY | 9799 | 11478 |  +1679 |  +17.14 | 7402 | 8886 |  +1484 |  +20.05 | 2397 | 2592 |  +195 |  +8.13 | 75.54 | 77.42 |  +1.88 |  +2.49 | 65.11 | 60.42 | -4.69 | -7.20 | 72.81 | 76.46 |  +3.65 |  +5.01 | 63.29 | 59.78 | -3.51 | -5.54 |
| YAML | 11076 | 9110 | -1966 | -17.75 | 8605 | 6466 | -2139 | -24.86 | 2471 | 2645 |  +174 |  +7.03 | 77.69 | 70.97 | -6.72 | -8.65 | 62.03 | 64.50 |  +2.47 |  +3.99 | 74.78 | 69.41 | -5.37 | -7.18 | 60.09 | 63.46 |  +3.37 |  +5.61 |

#### 2.7.4 Total Tokens Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 11525 | 15866 |  +4341 |  +37.67 | 7653 | 9598 |  +1945 |  +25.41 | 3873 | 6269 |  +2396 |  +61.87 | 66.40 | 60.49 | -5.91 | -8.90 | 77.57 | 64.63 | -12.94 | -16.68 | 65.76 | 59.53 | -6.23 | -9.47 | 77.14 | 63.99 | -13.15 | -17.05 |
| JSON_COMPACT | 20368 | 17797 | -2571 | -12.62 | 14948 | 13348 | -1600 | -10.71 | 5420 | 4449 | -971 | -17.91 | 73.39 | 75.00 |  +1.61 |  +2.19 | 63.90 | 70.30 |  +6.40 |  +10.02 | 71.47 | 73.99 |  +2.52 |  +3.53 | 62.62 | 69.63 |  +7.01 |  +11.19 |
| JSON_PRETTY | 27582 | 23973 | -3609 | -13.08 | 21578 | 18175 | -3403 | -15.77 | 6005 | 5800 | -205 | -3.42 | 78.23 | 75.81 | -2.42 | -3.09 | 52.17 | 58.04 |  +5.87 |  +11.25 | 75.49 | 75.33 | -0.16 | -0.21 | 50.34 | 57.72 |  +7.38 |  +14.65 |
| TOON_DEFAULT | 19303 | 23361 |  +4058 |  +21.02 | 14711 | 18056 |  +3345 |  +22.74 | 4592 | 5305 |  +713 |  +15.53 | 76.21 | 77.29 |  +1.08 |  +1.42 | 67.99 | 60.29 | -7.69 | -11.31 | 74.28 | 75.60 |  +1.32 |  +1.78 | 66.70 | 59.17 | -7.53 | -11.29 |
| XML_COMPACT | 25125 | 22310 | -2815 | -11.20 | 18980 | 16914 | -2066 | -10.88 | 6146 | 5397 | -749 | -12.18 | 75.54 | 75.81 |  +0.27 |  +0.36 | 55.47 | 61.48 |  +6.02 |  +10.85 | 73.08 | 74.06 |  +0.98 |  +1.34 | 53.83 | 60.32 |  +6.49 |  +12.05 |
| XML_PRETTY | 25936 | 26554 |  +618 |  +2.38 | 19592 | 20558 |  +966 |  +4.93 | 6344 | 5996 | -348 | -5.49 | 75.54 | 77.42 |  +1.88 |  +2.49 | 53.79 | 53.76 | -0.03 | -0.05 | 72.81 | 76.46 |  +3.65 |  +5.01 | 51.97 | 53.12 |  +1.15 |  +2.21 |
| YAML | 23609 | 20852 | -2757 | -11.68 | 18342 | 14799 | -3543 | -19.32 | 5267 | 6053 |  +786 |  +14.93 | 77.69 | 70.97 | -6.72 | -8.65 | 60.05 | 61.28 |  +1.24 |  +2.06 | 74.78 | 69.41 | -5.37 | -7.18 | 58.11 | 60.24 |  +2.14 |  +3.68 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Accuracy by Char (%) |
|---|---|---|---|---|---|---|
| CSV | man | 82.33 | 41.67 | 0.00 | 66.40 | 86.53 |
| CSV | opt | 75.00 | 49.00 | 0.00 | 60.49 | 85.85 |
| JSON_COMPACT | man | 91.00 | 33.00 | 0.00 | 73.39 | 92.05 |
| JSON_COMPACT | opt | 93.00 | 31.00 | 0.00 | 75.00 | 91.28 |
| JSON_PRETTY | man | 97.00 | 27.00 | 0.00 | 78.23 | 93.90 |
| JSON_PRETTY | opt | 94.00 | 30.00 | 0.00 | 75.81 | 91.03 |
| TOON_DEFAULT | man | 94.50 | 29.50 | 0.00 | 76.21 | 93.30 |
| TOON_DEFAULT | opt | 95.83 | 28.17 | 0.00 | 77.29 | 92.32 |
| XML_COMPACT | man | 93.67 | 30.33 | 0.00 | 75.54 | 91.94 |
| XML_COMPACT | opt | 94.00 | 30.00 | 0.00 | 75.81 | 91.92 |
| XML_PRETTY | man | 93.67 | 30.33 | 0.00 | 75.54 | 92.18 |
| XML_PRETTY | opt | 96.00 | 28.00 | 0.00 | 77.42 | 91.55 |
| YAML | man | 96.33 | 27.67 | 0.00 | 77.69 | 93.61 |
| YAML | opt | 88.00 | 36.00 | 0.00 | 70.97 | 89.72 |

#### 2.8.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Accuracy by Char (%) Man | Accuracy by Char (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 82.33 | 75.00 | -7 | -8.90 | 41.67 | 49.00 |  +7 |  +17.59 | 0.00 | 0.00 | 0 | 0.00 | 66.40 | 60.49 | -5.91 | 86.53 | 85.85 |  +86.53 |
| JSON_COMPACT | 91.00 | 93.00 |  +2 |  +2.20 | 33.00 | 31.00 | -2 | -6.06 | 0.00 | 0.00 | 0 | 0.00 | 73.39 | 75.00 |  +1.61 | 92.05 | 91.28 |  +92.05 |
| JSON_PRETTY | 97.00 | 94.00 | -3 | -3.09 | 27.00 | 30.00 |  +3 |  +11.11 | 0.00 | 0.00 | 0 | 0.00 | 78.23 | 75.81 | -2.42 | 93.90 | 91.03 |  +93.90 |
| TOON_DEFAULT | 94.50 | 95.83 |  +1 |  +1.41 | 29.50 | 28.17 | -1 | -4.51 | 0.00 | 0.00 | 0 | 0.00 | 76.21 | 77.29 |  +1.08 | 93.30 | 92.32 |  +93.30 |
| XML_COMPACT | 93.67 | 94.00 |  +0 |  +0.35 | 30.33 | 30.00 | -0 | -1.09 | 0.00 | 0.00 | 0 | 0.00 | 75.54 | 75.81 |  +0.27 | 91.94 | 91.92 |  +91.94 |
| XML_PRETTY | 93.67 | 96.00 |  +2 |  +2.49 | 30.33 | 28.00 | -2 | -7.68 | 0.00 | 0.00 | 0 | 0.00 | 75.54 | 77.42 |  +1.88 | 92.18 | 91.55 |  +92.18 |
| YAML | 96.33 | 88.00 | -8 | -8.65 | 27.67 | 36.00 |  +8 |  +30.10 | 0.00 | 0.00 | 0 | 0.00 | 77.69 | 70.97 | -6.72 | 93.61 | 89.72 |  +93.61 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| CSV | man | 66.40 | 73.33 | 67.90 | 52.38 | 60.32 |
| CSV | opt | 60.49 | 76.97 | 49.38 | 57.14 | 34.92 |
| JSON_COMPACT | man | 73.39 | 98.79 | 56.79 | 61.90 | 39.68 |
| JSON_COMPACT | opt | 75.00 | 92.73 | 69.14 | 61.90 | 49.21 |
| JSON_PRETTY | man | 78.23 | 100.00 | 56.79 | 66.66 | 60.32 |
| JSON_PRETTY | opt | 75.81 | 92.73 | 72.84 | 65.08 | 46.03 |
| TOON_DEFAULT | man | 76.21 | 91.21 | 61.11 | 68.26 | 64.28 |
| TOON_DEFAULT | opt | 77.29 | 98.48 | 62.96 | 67.46 | 50.00 |
| XML_COMPACT | man | 75.54 | 100.00 | 55.55 | 63.49 | 49.21 |
| XML_COMPACT | opt | 75.81 | 100.00 | 60.49 | 65.08 | 42.86 |
| XML_PRETTY | man | 75.54 | 90.91 | 56.79 | 63.49 | 71.43 |
| XML_PRETTY | opt | 77.42 | 96.97 | 72.84 | 61.90 | 47.62 |
| YAML | man | 77.69 | 97.57 | 54.32 | 68.25 | 65.08 |
| YAML | opt | 70.97 | 90.30 | 56.79 | 63.49 | 46.03 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 73.33 | 76.97 |  +3.64 |
| JSON_COMPACT | 98.79 | 92.73 | -6.06 |
| JSON_PRETTY | 100.00 | 92.73 | -7.27 |
| TOON_DEFAULT | 91.21 | 98.48 |  +7.27 |
| XML_COMPACT | 100.00 | 100.00 | 0.00 |
| XML_PRETTY | 90.91 | 96.97 |  +6.06 |
| YAML | 97.57 | 90.30 | -7.27 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 67.90 | 49.38 | -18.52 |
| JSON_COMPACT | 56.79 | 69.14 |  +12.35 |
| JSON_PRETTY | 56.79 | 72.84 |  +16.05 |
| TOON_DEFAULT | 61.11 | 62.96 |  +1.85 |
| XML_COMPACT | 55.55 | 60.49 |  +4.94 |
| XML_PRETTY | 56.79 | 72.84 |  +16.05 |
| YAML | 54.32 | 56.79 |  +2.47 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 52.38 | 57.14 |  +4.76 |
| JSON_COMPACT | 61.90 | 61.90 |  +0.00 |
| JSON_PRETTY | 66.66 | 65.08 | -1.58 |
| TOON_DEFAULT | 68.26 | 67.46 | -0.80 |
| XML_COMPACT | 63.49 | 65.08 |  +1.59 |
| XML_PRETTY | 63.49 | 61.90 | -1.59 |
| YAML | 68.25 | 63.49 | -4.76 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 60.32 | 34.92 | -25.40 |
| JSON_COMPACT | 39.68 | 49.21 |  +9.52 |
| JSON_PRETTY | 60.32 | 46.03 | -14.29 |
| TOON_DEFAULT | 64.28 | 50.00 | -14.29 |
| XML_COMPACT | 49.21 | 42.86 | -6.35 |
| XML_PRETTY | 71.43 | 47.62 | -23.81 |
| YAML | 65.08 | 46.03 | -19.04 |

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

- **Report Generated**: 2026-04-12
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code < 2.1.86
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_flat_all_formats_and_variants_off\BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_flat_all_formats_and_variants_on\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_off\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_on_verify\BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)