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
   - Optional: JSON_COMPACT 18250 tokens
   - Mandatory: JSON_COMPACT 18446 tokens
- Lowest read token cost:
   - Optional: JSON_COMPACT 9788 tokens
   - Mandatory: JSON_COMPACT 10315 tokens
- Lowest output token cost:
   - Optional: XML_COMPACT 7356 tokens
   - Mandatory: XML_PRETTY 7518 tokens
- Lowest output token cost drift:
   - Optional: JSON_PRETTY ↓ -7.59% ↑ 14.07%
   - Mandatory: JSON_PRETTY ↓ -10.06% ↑ 8.69%
- Highest accuracy:
   - Optional: YAML 77.42%
   - Mandatory: YAML 75.80%
- Lowest accuracy drift:
   - Optional: XML_COMPACT ↓ -1.85% ↑ 3.68%
   - Mandatory: YAML ↓ -5.32% ↑ 4.26%
- Most useful read tokens:
   - Optional: XML_PRETTY 13951 / 19583 tokens
   - Mandatory: XML_PRETTY 14166 / 20114 tokens
- Most useful output tokens:
   - Optional: YAML 9984 / 12896 tokens
   - Mandatory: YAML 8417 / 11104 tokens
- Highest read efficiency (%/token):
   - Optional: JSON_COMPACT 83.29
   - Mandatory: JSON_COMPACT 79.98
- Highest output efficiency (%/token):
   - Optional: XML_COMPACT 82.03
   - Mandatory: XML_PRETTY 79.54
- Highest accuracy by char:
   - Optional: YAML 92.94%
   - Mandatory: YAML 92.87%
- Lowest accuracy by char drift:
   - Optional: JSON_COMPACT ↓ -1.69% ↑ 0.99%
   - Mandatory: YAML ↓ -0.71% ↑ 1.38%
- Most useful output write tokens (Acc By Char):
   - Optional: YAML 11674 / 12560 tokens
   - Mandatory: YAML 9999 / 10767 tokens
- Highest output write efficiency (Acc By Char) (%/token):
   - Optional: XML_COMPACT 93.69
   - Mandatory: XML_PRETTY 93.31
- Lowest delta (optional-mandatory):
   - Read tokens: TOON_DEFAULT -237 tokens
   - Output tokens: JSON_COMPACT 331 tokens
   - Accuracy: XML_PRETTY 0.81%
   - Read efficiency: JSON_PRETTY -0.05
   - Output efficiency: JSON_COMPACT 0.19
   - Accuracy by char: YAML 0.07%
   - Output write efficiency (Acc By Char): JSON_COMPACT -1.04

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 29284 tokens
   - Mandatory: JSON_PRETTY 28599 tokens
- Highest read token cost:
   - Optional: XML_PRETTY 19583 tokens
   - Mandatory: XML_PRETTY 20114 tokens
- Highest output token cost:
   - Optional: YAML 12896 tokens
   - Mandatory: YAML 11104 tokens
- Highest output token drift:
   - Optional: JSON_COMPACT ↓ -96.04% ↑ 131.71%
   - Mandatory: XML_COMPACT ↓ -37.67% ↑ 72.65%
- Lowest accuracy:
   - Optional: JSON_PRETTY 69.35%
   - Mandatory: XML_PRETTY 70.43%
- Highest accuracy drift:
   - Optional: TOON_DEFAULT ↓ -14.59% ↑ 7.56%
   - Mandatory: XML_PRETTY ↓ -15.26% ↑ 16.80%
- Most wasted read tokens:
   - Optional: XML_PRETTY 5632 / 19583 tokens
   - Mandatory: XML_PRETTY 5948 / 20114 tokens
- Most wasted output tokens:
   - Optional: JSON_PRETTY 2960 / 9659 tokens
   - Mandatory: JSON_PRETTY 2809 / 10771 tokens
- Lowest read efficiency (%/token):
   - Optional: XML_PRETTY 49.23
   - Mandatory: XML_PRETTY 46.98
- Lowest output efficiency (%/token):
   - Optional: YAML 60.98
   - Mandatory: YAML 67.63
- Lowest accuracy by char:
   - Optional: JSON_PRETTY 88.88%
   - Mandatory: XML_COMPACT 90.89%
- Highest accuracy by char drift:
   - Optional: TOON_DEFAULT ↓ -4.78% ↑ 2.37%
   - Mandatory: XML_COMPACT ↓ -3.64% ↑ 4.71%
- Most wasted output write tokens (Acc By Char):
   - Optional: JSON_PRETTY 8383 / 9431 tokens
   - Mandatory: JSON_PRETTY 9435 / 10324 tokens
- Lowest output write efficiency (Acc By Char) (%/token):
   - Optional: YAML 71.25
   - Mandatory: YAML 78.96
- Highest delta (optional-mandatory):
   - Read tokens: JSON_PRETTY -929 tokens
   - Output tokens: TOON_DEFAULT 3506 tokens
   - Accuracy: TOON_DEFAULT 5.38%
   - Read efficiency: TOON_DEFAULT 4.35
   - Output efficiency: TOON_DEFAULT -11.55
   - Accuracy by char: JSON_PRETTY -2.51%
   - Output write efficiency (Acc By Char): TOON_DEFAULT -15.69

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy By Char | ↓ Eff Score Output Write (Acc By Char) | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT ≈ 69s | JSON_COMPACT ≈ 10315 | JSON_COMPACT ≈ 336 | XML_PRETTY ≈ 7175 | XML_PRETTY ≈ 7518 | JSON_COMPACT ≈ 18446 | YAML ≈ 93% | XML_PRETTY ≈ 93 | YAML ≈ 76% | JSON_COMPACT ≈ 80 | XML_PRETTY ≈ 80 | JSON_COMPACT ≈ 81 |
| XML_PRETTY (+2.2%) | XML_COMPACT (+24.6%) | YAML (+0.5%) | JSON_COMPACT (+8.6%) | JSON_COMPACT (+8.2%) | XML_COMPACT (+18.3%) | TOON_DEFAULT (-0.7%) | JSON_COMPACT (-3.0%) | JSON_PRETTY (-1.9%) | XML_COMPACT (-10.9%) | JSON_COMPACT (-1.5%) | XML_COMPACT (-13.2%) |
| TOON_DEFAULT (+7.0%) | TOON_DEFAULT (+36.7%) | TOON_DEFAULT (+1.7%) | TOON_DEFAULT (+16.3%) | TOON_DEFAULT (+15.5%) | TOON_DEFAULT (+23.5%) | JSON_PRETTY (-1.5%) | TOON_DEFAULT (-4.6%) | JSON_COMPACT (-3.2%) | YAML (-13.4%) | TOON_DEFAULT (-5.8%) | TOON_DEFAULT (-17.3%) |
| XML_COMPACT (+8.0%) | YAML (+38.7%) | XML_PRETTY (+1.9%) | XML_COMPACT (+20.2%) | XML_COMPACT (+19.3%) | YAML (+37.8%) | XML_PRETTY (-1.8%) | XML_COMPACT (-6.9%) | XML_COMPACT (-4.0%) | TOON_DEFAULT (-16.5%) | XML_COMPACT (-6.7%) | YAML (-23.2%) |
| YAML (+31.8%) | JSON_PRETTY (+72.8%) | XML_COMPACT (+2.6%) | JSON_PRETTY (+43.9%) | JSON_PRETTY (+43.3%) | XML_PRETTY (+49.8%) | JSON_COMPACT (-1.9%) | JSON_PRETTY (-14.4%) | TOON_DEFAULT (-4.7%) | JSON_PRETTY (-29.1%) | JSON_PRETTY (-14.7%) | XML_PRETTY (-35.9%) |
| JSON_PRETTY (+34.9%) | XML_PRETTY (+95.0%) | JSON_PRETTY (+33.0%) | YAML (+50.1%) | YAML (+47.7%) | JSON_PRETTY (+55.0%) | XML_COMPACT (-2.0%) | YAML (-15.4%) | XML_PRETTY (-5.4%) | XML_PRETTY (-41.3%) | YAML (-15.0%) | JSON_PRETTY (-36.7%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy By Char | ↓ Eff Score Output Write (Acc By Char) | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|
| XML_COMPACT ≈ 64s | JSON_COMPACT ≈ 9788 | JSON_PRETTY ≈ 228 | XML_COMPACT ≈ 7011 | XML_COMPACT ≈ 7356 | JSON_COMPACT ≈ 18250 | YAML ≈ 93% | XML_COMPACT ≈ 94 | YAML ≈ 77% | JSON_COMPACT ≈ 83 | XML_COMPACT ≈ 82 | JSON_COMPACT ≈ 83 |
| XML_PRETTY (+21.1%) | XML_COMPACT (+26.4%) | JSON_COMPACT (+47.4%) | JSON_COMPACT (+15.9%) | JSON_COMPACT (+15.0%) | XML_COMPACT (+8.1%) | JSON_COMPACT (-1.4%) | JSON_COMPACT (-4.5%) | TOON_DEFAULT (-0.9%) | XML_COMPACT (-11.5%) | JSON_COMPACT (-4.3%) | XML_COMPACT (-6.8%) |
| JSON_PRETTY (+31.7%) | TOON_DEFAULT (+41.6%) | YAML (+47.6%) | XML_PRETTY (+33.4%) | JSON_PRETTY (+31.3%) | TOON_DEFAULT (+42.7%) | TOON_DEFAULT (-1.6%) | XML_PRETTY (-11.1%) | JSON_COMPACT (-2.4%) | YAML (-14.6%) | XML_PRETTY (-13.9%) | TOON_DEFAULT (-27.1%) |
| JSON_COMPACT (+37.5%) | YAML (+43.6%) | XML_PRETTY (+51.1%) | JSON_PRETTY (+34.5%) | XML_PRETTY (+31.9%) | JSON_PRETTY (+45.5%) | XML_COMPACT (-2.3%) | JSON_PRETTY (-12.4%) | XML_COMPACT (-4.3%) | TOON_DEFAULT (-14.6%) | JSON_PRETTY (-15.2%) | YAML (-29.6%) |
| YAML (+47.7%) | JSON_PRETTY (+72.7%) | XML_COMPACT (+51.2%) | TOON_DEFAULT (+68.9%) | TOON_DEFAULT (+65.7%) | YAML (+47.7%) | XML_PRETTY (-2.8%) | TOON_DEFAULT (-21.8%) | XML_PRETTY (-6.2%) | JSON_PRETTY (-32.0%) | TOON_DEFAULT (-22.7%) | JSON_PRETTY (-34.6%) |
| TOON_DEFAULT (+51.2%) | XML_PRETTY (+100.1%) | TOON_DEFAULT (+54.5%) | YAML (+79.1%) | YAML (+75.3%) | XML_PRETTY (+60.5%) | JSON_PRETTY (-4.1%) | YAML (-23.9%) | JSON_PRETTY (-8.1%) | XML_PRETTY (-40.9%) | YAML (-25.7%) | XML_PRETTY (-43.0%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 98% | JSON_PRETTY ≈ 58% | JSON_COMPACT ≈ 67% | JSON_COMPACT ≈ 71% |
| JSON_PRETTY (-1.2%) | YAML (-2.5%) | XML_COMPACT (0.0%) | TOON_DEFAULT (-5.6%) |
| TOON_DEFAULT (-6.4%) | XML_COMPACT (-2.5%) | JSON_PRETTY (-6.4%) | YAML (-11.1%) |
| XML_PRETTY (-9.1%) | XML_PRETTY (-2.5%) | YAML (-6.4%) | XML_PRETTY (-12.7%) |
| XML_COMPACT (-9.7%) | JSON_COMPACT (-4.9%) | TOON_DEFAULT (-7.1%) | XML_COMPACT (-15.9%) |
| JSON_COMPACT (-12.7%) | TOON_DEFAULT (-14.8%) | XML_PRETTY (-12.7%) | JSON_PRETTY (-22.2%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99% | XML_COMPACT ≈ 65% | TOON_DEFAULT ≈ 71% | YAML ≈ 56% |
| TOON_DEFAULT (-3.0%) | TOON_DEFAULT (-0.0%) | XML_PRETTY (-7.9%) | XML_COMPACT (-0.0%) |
| JSON_COMPACT (-5.5%) | JSON_COMPACT (-1.2%) | YAML (-7.9%) | JSON_COMPACT (-3.2%) |
| JSON_PRETTY (-9.1%) | YAML (-4.9%) | XML_COMPACT (-7.9%) | XML_PRETTY (-3.2%) |
| XML_PRETTY (-10.3%) | XML_PRETTY (-9.9%) | JSON_COMPACT (-9.5%) | JSON_PRETTY (-9.5%) |
| XML_COMPACT (-12.1%) | JSON_PRETTY (-14.8%) | JSON_PRETTY (-9.5%) | TOON_DEFAULT (-11.9%) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy By Char (%) | Useful Output Write Tokens (Acc By Char) | Wasted Output Write Tokens (Acc By Char) | Eff Score Output Write (Acc By Char) | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10315 | 8131 | 18446 | 2.213 | 62.866 | 90.98 | 7092.19 | 703.14 | 90.55 | 72.58 | 7486.627 | 2828.373 | 5901.721 | 2229.612 | 79.98 | 78.32 | 81.09 |
| JSON_COMPACT | opt | 9788 | 8462 | 18250 | 2.186 | 65.535 | 91.57 | 7441.28 | 685.05 | 89.51 | 75.00 | 7341.000 | 2447.000 | 6346.500 | 2115.500 | 83.29 | 78.51 | 83.30 |
| JSON_PRETTY | man | 17828 | 10771 | 28599 | 1.762 | 83.255 | 91.39 | 9434.80 | 888.87 | 79.89 | 73.92 | 13178.458 | 4649.542 | 7961.677 | 2808.990 | 56.67 | 67.82 | 51.37 |
| JSON_PRETTY | opt | 16899 | 9659 | 26558 | 1.752 | 76.059 | 88.88 | 8382.57 | 1048.76 | 82.07 | 69.35 | 11719.456 | 5179.544 | 6698.516 | 2960.484 | 56.62 | 69.57 | 54.48 |
| TOON_DEFAULT | man | 14096 | 8685 | 22781 | 1.851 | 67.281 | 92.19 | 7691.26 | 651.58 | 88.98 | 71.10 | 10022.256 | 4073.744 | 6174.798 | 2509.868 | 66.81 | 74.94 | 67.03 |
| TOON_DEFAULT | opt | 13859 | 12191 | 26050 | 1.860 | 95.478 | 91.33 | 10812.86 | 1026.47 | 73.29 | 76.48 | 10599.363 | 3259.637 | 9323.740 | 2867.343 | 71.16 | 63.39 | 60.76 |
| XML_COMPACT | man | 12848 | 8966 | 21814 | 2.522 | 69.530 | 90.89 | 7836.23 | 785.43 | 86.91 | 71.78 | 9222.294 | 3625.706 | 6436.034 | 2530.299 | 71.29 | 74.18 | 70.40 |
| XML_COMPACT | opt | 12368 | 7356 | 19724 | 2.517 | 56.543 | 90.61 | 6352.97 | 658.36 | 93.69 | 73.12 | 9043.482 | 3324.518 | 5378.464 | 1977.203 | 73.73 | 82.03 | 77.60 |
| XML_PRETTY | man | 20114 | 7518 | 27632 | 1.993 | 57.866 | 91.11 | 6537.45 | 637.89 | 93.31 | 70.43 | 14166.290 | 5947.710 | 5294.693 | 2222.974 | 46.98 | 79.54 | 51.96 |
| XML_PRETTY | opt | 19583 | 9701 | 29284 | 1.982 | 75.456 | 90.18 | 8437.69 | 918.81 | 83.26 | 71.24 | 13950.929 | 5632.071 | 6910.636 | 2789.864 | 49.23 | 70.65 | 47.52 |
| YAML | man | 14306 | 11104 | 25410 | 1.789 | 86.828 | 92.87 | 9999.00 | 767.66 | 78.96 | 75.80 | 10843.948 | 3462.052 | 8417.084 | 2687.249 | 69.27 | 67.63 | 62.24 |
| YAML | opt | 14053 | 12896 | 26949 | 1.799 | 101.293 | 92.94 | 11673.57 | 886.76 | 71.25 | 77.42 | 10879.833 | 3173.167 | 9984.341 | 2911.992 | 71.17 | 60.98 | 58.68 |

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
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 24 | 429.792 | 0.77 | 38410 | 0.203 | 309.76 | 38434 | 429.995 | 247.96 |
| JSON_COMPACT | opt | 9 | 1087.556 | 0.29 | 39495 | 0.206 | 318.51 | 39504 | 1087.762 | 254.87 |
| JSON_PRETTY | man | 267 | 66.772 | 8.61 | 36625 | 0.282 | 295.36 | 36892 | 67.054 | 238.01 |
| JSON_PRETTY | opt | 272 | 62.129 | 8.77 | 36507 | 0.258 | 294.41 | 36779 | 62.387 | 237.28 |
| TOON_DEFAULT | man | 10 | 1409.600 | 0.32 | 39156 | 0.218 | 315.77 | 39166 | 1409.818 | 252.68 |
| TOON_DEFAULT | opt | 20 | 692.950 | 0.65 | 39438 | 0.309 | 318.05 | 39458 | 693.260 | 254.57 |
| XML_COMPACT | man | 11 | 1168.000 | 0.35 | 41724 | 0.207 | 336.49 | 41735 | 1168.207 | 269.26 |
| XML_COMPACT | opt | 9 | 1374.222 | 0.29 | 39280 | 0.178 | 316.77 | 39289 | 1374.400 | 253.48 |
| XML_PRETTY | man | 11 | 1828.545 | 0.35 | 47627 | 0.151 | 384.09 | 47638 | 1828.696 | 307.34 |
| XML_PRETTY | opt | 11 | 1780.273 | 0.35 | 35179 | 0.266 | 283.70 | 35190 | 1780.539 | 227.03 |
| YAML | man | 12 | 1192.167 | 0.39 | 36643 | 0.294 | 295.51 | 36655 | 1192.461 | 236.48 |
| YAML | opt | 8 | 1756.625 | 0.26 | 32619 | 0.385 | 263.05 | 32627 | 1757.010 | 210.49 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 24 | 9 | -15 | -62.50 | 38.41 | 39.50 |  +1.08 |  +2.82 | 38.43 | 39.50 |  +1.07 |  +2.78 |
| JSON_PRETTY | 267 | 272 |  +5 |  +1.87 | 36.63 | 36.51 | -0.12 | -0.32 | 36.89 | 36.78 | -0.11 | -0.31 |
| TOON_DEFAULT | 10 | 20 |  +10 |  +100.00 | 39.16 | 39.44 |  +0.28 |  +0.72 | 39.17 | 39.46 |  +0.29 |  +0.75 |
| XML_COMPACT | 11 | 9 | -2 | -18.18 | 41.72 | 39.28 | -2.44 | -5.86 | 41.74 | 39.29 | -2.45 | -5.86 |
| XML_PRETTY | 11 | 11 | 0 | 0.00 | 47.63 | 35.18 | -12.45 | -26.14 | 47.64 | 35.19 | -12.45 | -26.13 |
| YAML | 12 | 8 | -4 | -33.33 | 36.64 | 32.62 | -4.02 | -10.98 | 36.66 | 32.63 | -4.03 | -10.99 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token |
|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 2.213 | 15.125 | 332.742 | 0.704 | 0.893 | 0.393 |
| JSON_COMPACT | opt | 2.186 | 15.512 | 315.742 | 0.766 | 0.886 | 0.411 |
| JSON_PRETTY | man | 1.762 | 26.141 | 575.097 | 0.415 | 0.686 | 0.258 |
| JSON_PRETTY | opt | 1.752 | 26.781 | 545.129 | 0.410 | 0.718 | 0.261 |
| TOON_DEFAULT | man | 1.851 | 20.669 | 454.710 | 0.504 | 0.820 | 0.312 |
| TOON_DEFAULT | opt | 1.860 | 21.964 | 447.065 | 0.552 | 0.664 | 0.297 |
| XML_COMPACT | man | 2.522 | 18.839 | 414.452 | 0.559 | 0.801 | 0.329 |
| XML_COMPACT | opt | 2.517 | 19.601 | 398.968 | 0.591 | 0.994 | 0.371 |
| XML_PRETTY | man | 1.993 | 29.493 | 648.839 | 0.350 | 0.937 | 0.255 |
| XML_PRETTY | opt | 1.982 | 31.035 | 631.710 | 0.364 | 0.734 | 0.243 |
| YAML | man | 1.789 | 20.977 | 461.484 | 0.530 | 0.683 | 0.298 |
| YAML | opt | 1.799 | 22.271 | 453.323 | 0.551 | 0.600 | 0.287 |

#### 2.5.2 Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 2.213 | 2.186 | -0.027 | -1.22 | 15.125 | 15.512 |  +0.387 |  +2.56 | 332.742 | 315.742 | -17.000 | -5.11 | 0.704 | 0.766 |  +0.062 |  +8.81 | 0.893 | 0.886 | -0.007 | -0.78 | 0.393 | 0.411 |  +0.018 |  +4.58 |
| JSON_PRETTY | 1.762 | 1.752 | -0.010 | -0.57 | 26.141 | 26.781 |  +0.640 |  +2.45 | 575.097 | 545.129 | -29.968 | -5.21 | 0.415 | 0.410 | -0.005 | -1.20 | 0.686 | 0.718 |  +0.032 |  +4.66 | 0.258 | 0.261 |  +0.003 |  +1.16 |
| TOON_DEFAULT | 1.851 | 1.860 |  +0.009 |  +0.49 | 20.669 | 21.964 |  +1.295 |  +6.27 | 454.710 | 447.065 | -7.645 | -1.68 | 0.504 | 0.552 |  +0.048 |  +9.52 | 0.820 | 0.664 | -0.156 | -18.97 | 0.312 | 0.297 | -0.015 | -4.81 |
| XML_COMPACT | 2.522 | 2.517 | -0.005 | -0.20 | 18.839 | 19.601 |  +0.762 |  +4.04 | 414.452 | 398.968 | -15.484 | -3.74 | 0.559 | 0.591 |  +0.032 |  +5.72 | 0.801 | 0.994 |  +0.193 |  +24.09 | 0.329 | 0.371 |  +0.042 |  +12.77 |
| XML_PRETTY | 1.993 | 1.982 | -0.011 | -0.55 | 29.493 | 31.035 |  +1.542 |  +5.23 | 648.839 | 631.710 | -17.129 | -2.64 | 0.350 | 0.364 |  +0.014 |  +4.00 | 0.937 | 0.734 | -0.203 | -21.66 | 0.255 | 0.243 | -0.012 | -4.71 |
| YAML | 1.789 | 1.799 |  +0.010 |  +0.56 | 20.977 | 22.271 |  +1.294 |  +6.17 | 461.484 | 453.323 | -8.161 | -1.77 | 0.530 | 0.551 |  +0.021 |  +3.96 | 0.683 | 0.600 | -0.083 | -12.15 | 0.298 | 0.287 | -0.011 | -3.69 |

### 2.6 Output Write Token Utilization Efficiency (Accuracy By Char)
#### 2.6.1 Metrics
| Format | Variant | Output Write Tokens | Useful Output Write Tokens (Acc By Char) | Wasted Output Write Tokens (Acc By Char) | Accuracy by Char (%) | Eff Score Output Write (Acc By Char) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 7795 | 7092 | 703 | 90.98 | 90.55 |
| JSON_COMPACT | opt | 8126 | 7441 | 685 | 91.57 | 89.51 |
| JSON_PRETTY | man | 10324 | 9435 | 889 | 91.39 | 79.89 |
| JSON_PRETTY | opt | 9431 | 8383 | 1049 | 88.88 | 82.07 |
| TOON_DEFAULT | man | 8343 | 7691 | 652 | 92.19 | 88.98 |
| TOON_DEFAULT | opt | 11839 | 10813 | 1026 | 91.33 | 73.29 |
| XML_COMPACT | man | 8622 | 7836 | 785 | 90.89 | 86.91 |
| XML_COMPACT | opt | 7011 | 6353 | 658 | 90.61 | 93.69 |
| XML_PRETTY | man | 7175 | 6537 | 638 | 91.11 | 93.31 |
| XML_PRETTY | opt | 9357 | 8438 | 919 | 90.18 | 83.26 |
| YAML | man | 10767 | 9999 | 768 | 92.87 | 78.96 |
| YAML | opt | 12560 | 11674 | 887 | 92.94 | 71.25 |

#### 2.6.2 Mandatory vs Optional
| Format | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Useful Output Write Tokens (Acc By Char) Man | Useful Output Write Tokens (Acc By Char) Opt | Diff | Diff (%) | Wasted Output Write Tokens (Acc By Char) Man | Wasted Output Write Tokens (Acc By Char) Opt | Diff | Diff (%) | Accuracy By Char (%) Man | Accuracy By Char (%) Opt | Diff (%) | Eff Score Output Write (Acc By Char) Man | Eff Score Output Write (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7795 | 8126 |  +331 |  +4.25 | 7092 | 7441 |  +349 |  +4.92 | 703 | 685 | -18 | -2.57 | 90.98 | 91.57 |  +0.59 |  +0.65 | 90.55 | 89.51 | -1.04 | -1.15 |
| JSON_PRETTY | 10324 | 9432 | -892 | -8.64 | 9435 | 8383 | -1052 | -11.15 | 889 | 1049 |  +160 |  +17.99 | 91.39 | 88.88 | -2.51 | -2.75 | 79.89 | 82.07 |  +2.19 |  +2.74 |
| TOON_DEFAULT | 8343 | 11840 |  +3497 |  +41.91 | 7691 | 10813 |  +3122 |  +40.59 | 652 | 1027 |  +375 |  +57.50 | 92.19 | 91.33 | -0.86 | -0.93 | 88.98 | 73.29 | -15.69 | -17.63 |
| XML_COMPACT | 8622 | 7012 | -1610 | -18.68 | 7836 | 6353 | -1483 | -18.93 | 785 | 658 | -127 | -16.19 | 90.89 | 90.61 | -0.28 | -0.31 | 86.91 | 93.69 |  +6.77 |  +7.80 |
| XML_PRETTY | 7175 | 9356 |  +2181 |  +30.40 | 6537 | 8437 |  +1900 |  +29.07 | 638 | 919 |  +281 |  +44.03 | 91.11 | 90.18 | -0.93 | -1.02 | 93.31 | 83.26 | -10.05 | -10.77 |
| YAML | 10767 | 12561 |  +1794 |  +16.66 | 9999 | 11674 |  +1675 |  +16.75 | 768 | 887 |  +119 |  +15.51 | 92.87 | 92.94 |  +0.07 |  +0.08 | 78.96 | 71.25 | -7.71 | -9.76 |

### 2.7 Token Utilization Efficiency
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10315 | 7487 | 2828 | 8131 | 5902 | 2230 | 18446 | 13388 | 5058 | 72.58 | 79.98 | 78.32 | 81.09 | 70.12 | 78.34 | 76.68 | 79.45 |
| JSON_COMPACT | opt | 9788 | 7341 | 2447 | 8462 | 6347 | 2116 | 18250 | 13688 | 4563 | 75.00 | 83.29 | 78.51 | 83.30 | 73.39 | 82.22 | 77.43 | 82.22 |
| JSON_PRETTY | man | 17828 | 13178 | 4650 | 10771 | 7962 | 2809 | 28599 | 21140 | 7459 | 73.92 | 56.67 | 67.82 | 51.37 | 71.77 | 55.24 | 66.39 | 49.94 |
| JSON_PRETTY | opt | 16899 | 11719 | 5180 | 9659 | 6699 | 2960 | 26558 | 18418 | 8140 | 69.35 | 56.62 | 69.57 | 54.48 | 67.27 | 55.23 | 68.19 | 53.09 |
| TOON_DEFAULT | man | 14096 | 10022 | 4074 | 8685 | 6175 | 2510 | 22781 | 16197 | 6584 | 71.10 | 66.81 | 74.94 | 67.03 | 67.44 | 64.38 | 72.51 | 64.59 |
| TOON_DEFAULT | opt | 13859 | 10599 | 3260 | 12191 | 9324 | 2867 | 26050 | 19923 | 6127 | 76.48 | 71.16 | 63.39 | 60.76 | 75.56 | 70.55 | 62.78 | 60.15 |
| XML_COMPACT | man | 12848 | 9222 | 3626 | 8966 | 6436 | 2530 | 21814 | 15658 | 6156 | 71.78 | 71.29 | 74.18 | 70.40 | 69.99 | 70.09 | 72.99 | 69.21 |
| XML_COMPACT | opt | 12368 | 9043 | 3325 | 7356 | 5378 | 1977 | 19724 | 14422 | 5302 | 73.12 | 73.73 | 82.03 | 77.60 | 71.98 | 72.97 | 81.27 | 76.84 |
| XML_PRETTY | man | 20114 | 14166 | 5948 | 7518 | 5295 | 2223 | 27632 | 19461 | 8171 | 70.43 | 46.98 | 79.54 | 51.96 | 67.97 | 45.34 | 77.90 | 50.32 |
| XML_PRETTY | opt | 19583 | 13951 | 5632 | 9701 | 6911 | 2790 | 29284 | 20862 | 8422 | 71.24 | 49.23 | 70.65 | 47.52 | 69.39 | 48.00 | 69.42 | 46.29 |
| YAML | man | 14306 | 10844 | 3462 | 11104 | 8417 | 2687 | 25410 | 19261 | 6149 | 75.80 | 69.27 | 67.63 | 62.24 | 72.90 | 67.34 | 65.70 | 60.30 |
| YAML | opt | 14053 | 10880 | 3173 | 12896 | 9984 | 2912 | 26949 | 20864 | 6085 | 77.42 | 71.17 | 60.98 | 58.68 | 75.09 | 69.61 | 59.42 | 57.12 |

#### 2.7.2 Read Tokens Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10315 | 9788 | -527 | -5.11 | 7487 | 7341 | -146 | -1.95 | 2828 | 2447 | -381 | -13.49 | 72.58 | 75.00 |  +2.42 |  +3.33 | 79.98 | 83.29 |  +3.31 |  +4.14 | 70.12 | 73.39 |  +3.27 |  +4.66 | 78.34 | 82.22 |  +3.88 |  +4.95 |
| JSON_PRETTY | 17828 | 16899 | -929 | -5.21 | 13178 | 11719 | -1459 | -11.07 | 4650 | 5180 |  +530 |  +11.40 | 73.92 | 69.35 | -4.57 | -6.18 | 56.67 | 56.62 | -0.05 | -0.10 | 71.77 | 67.27 | -4.50 | -6.27 | 55.24 | 55.23 | -0.01 | -0.01 |
| TOON_DEFAULT | 14096 | 13859 | -237 | -1.68 | 10022 | 10599 |  +577 |  +5.76 | 4074 | 3260 | -814 | -19.98 | 71.10 | 76.48 |  +5.38 |  +7.57 | 66.81 | 71.16 |  +4.35 |  +6.51 | 67.44 | 75.56 |  +8.12 |  +12.04 | 64.38 | 70.55 |  +6.18 |  +9.59 |
| XML_COMPACT | 12848 | 12368 | -480 | -3.74 | 9222 | 9043 | -179 | -1.94 | 3626 | 3325 | -301 | -8.31 | 71.78 | 73.12 |  +1.34 |  +1.87 | 71.29 | 73.73 |  +2.44 |  +3.42 | 69.99 | 71.98 |  +1.99 |  +2.84 | 70.09 | 72.97 |  +2.87 |  +4.10 |
| XML_PRETTY | 20114 | 19583 | -531 | -2.64 | 14166 | 13951 | -215 | -1.52 | 5948 | 5632 | -316 | -5.31 | 70.43 | 71.24 |  +0.81 |  +1.15 | 46.98 | 49.23 |  +2.25 |  +4.79 | 67.97 | 69.39 |  +1.42 |  +2.09 | 45.34 | 48.00 |  +2.66 |  +5.86 |
| YAML | 14306 | 14053 | -253 | -1.77 | 10844 | 10880 |  +36 |  +0.33 | 3462 | 3173 | -289 | -8.34 | 75.80 | 77.42 |  +1.62 |  +2.14 | 69.27 | 71.17 |  +1.89 |  +2.74 | 72.90 | 75.09 |  +2.19 |  +3.00 | 67.34 | 69.61 |  +2.28 |  +3.38 |

#### 2.7.3 Output Tokens Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 8131 | 8462 |  +331 |  +4.07 | 5902 | 6347 |  +445 |  +7.54 | 2230 | 2116 | -114 | -5.12 | 72.58 | 75.00 |  +2.42 |  +3.33 | 78.32 | 78.51 |  +0.19 |  +0.24 | 70.12 | 73.39 |  +3.27 |  +4.66 | 76.68 | 77.43 |  +0.75 |  +0.98 |
| JSON_PRETTY | 10771 | 9659 | -1112 | -10.32 | 7962 | 6699 | -1263 | -15.86 | 2809 | 2960 |  +151 |  +5.39 | 73.92 | 69.35 | -4.57 | -6.18 | 67.82 | 69.57 |  +1.75 |  +2.58 | 71.77 | 67.27 | -4.50 | -6.27 | 66.39 | 68.19 |  +1.80 |  +2.71 |
| TOON_DEFAULT | 8685 | 12191 |  +3506 |  +40.37 | 6175 | 9324 |  +3149 |  +51.00 | 2510 | 2867 |  +357 |  +14.24 | 71.10 | 76.48 |  +5.38 |  +7.57 | 74.94 | 63.39 | -11.55 | -15.41 | 67.44 | 75.56 |  +8.12 |  +12.04 | 72.51 | 62.78 | -9.72 | -13.41 |
| XML_COMPACT | 8966 | 7355 | -1611 | -17.96 | 6436 | 5378 | -1058 | -16.43 | 2530 | 1977 | -553 | -21.86 | 71.78 | 73.12 |  +1.34 |  +1.87 | 74.18 | 82.03 |  +7.85 |  +10.58 | 69.99 | 71.98 |  +1.99 |  +2.84 | 72.99 | 81.27 |  +8.28 |  +11.34 |
| XML_PRETTY | 7518 | 9701 |  +2183 |  +29.03 | 5295 | 6911 |  +1616 |  +30.52 | 2223 | 2790 |  +567 |  +25.50 | 70.43 | 71.24 |  +0.81 |  +1.15 | 79.54 | 70.65 | -8.88 | -11.17 | 67.97 | 69.39 |  +1.42 |  +2.09 | 77.90 | 69.42 | -8.48 | -10.88 |
| YAML | 11104 | 12896 |  +1792 |  +16.14 | 8417 | 9984 |  +1567 |  +18.62 | 2687 | 2912 |  +225 |  +8.36 | 75.80 | 77.42 |  +1.62 |  +2.14 | 67.63 | 60.98 | -6.66 | -9.84 | 72.90 | 75.09 |  +2.19 |  +3.00 | 65.70 | 59.42 | -6.28 | -9.55 |

#### 2.7.4 Total Tokens Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 18446 | 18250 | -196 | -1.06 | 13388 | 13687 |  +299 |  +2.23 | 5058 | 4563 | -495 | -9.80 | 72.58 | 75.00 |  +2.42 |  +3.33 | 81.09 | 83.30 |  +2.20 |  +2.72 | 70.12 | 73.39 |  +3.27 |  +4.66 | 79.45 | 82.22 |  +2.77 |  +3.49 |
| JSON_PRETTY | 28599 | 26558 | -2041 | -7.14 | 21140 | 18418 | -2722 | -12.88 | 7459 | 8140 |  +681 |  +9.14 | 73.92 | 69.35 | -4.57 | -6.18 | 51.37 | 54.48 |  +3.11 |  +6.05 | 71.77 | 67.27 | -4.50 | -6.27 | 49.94 | 53.09 |  +3.15 |  +6.32 |
| TOON_DEFAULT | 22781 | 26050 |  +3269 |  +14.35 | 16197 | 19923 |  +3726 |  +23.00 | 6584 | 6127 | -457 | -6.94 | 71.10 | 76.48 |  +5.38 |  +7.57 | 67.03 | 60.76 | -6.27 | -9.36 | 67.44 | 75.56 |  +8.12 |  +12.04 | 64.59 | 60.15 | -4.45 | -6.88 |
| XML_COMPACT | 21814 | 19723 | -2091 | -9.58 | 15658 | 14422 | -1236 | -7.90 | 6156 | 5302 | -854 | -13.88 | 71.78 | 73.12 |  +1.34 |  +1.87 | 70.40 | 77.60 |  +7.20 |  +10.22 | 69.99 | 71.98 |  +1.99 |  +2.84 | 69.21 | 76.84 |  +7.63 |  +11.02 |
| XML_PRETTY | 27632 | 29284 |  +1652 |  +5.98 | 19461 | 20862 |  +1401 |  +7.20 | 8171 | 8422 |  +251 |  +3.07 | 70.43 | 71.24 |  +0.81 |  +1.15 | 51.96 | 47.52 | -4.44 | -8.55 | 67.97 | 69.39 |  +1.42 |  +2.09 | 50.32 | 46.29 | -4.03 | -8.02 |
| YAML | 25410 | 26949 |  +1539 |  +6.06 | 19261 | 20864 |  +1603 |  +8.32 | 6149 | 6085 | -64 | -1.04 | 75.80 | 77.42 |  +1.62 |  +2.14 | 62.24 | 58.68 | -3.56 | -5.72 | 72.90 | 75.09 |  +2.19 |  +3.00 | 60.30 | 57.12 | -3.18 | -5.27 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Accuracy by Char (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 90.00 | 34.00 | 0.00 | 72.58 | 90.98 |
| JSON_COMPACT | opt | 93.00 | 31.00 | 0.00 | 75.00 | 91.57 |
| JSON_PRETTY | man | 91.67 | 32.33 | 0.00 | 73.92 | 91.39 |
| JSON_PRETTY | opt | 86.00 | 38.00 | 0.00 | 69.35 | 88.88 |
| TOON_DEFAULT | man | 88.17 | 35.83 | 0.00 | 71.10 | 92.19 |
| TOON_DEFAULT | opt | 94.83 | 29.17 | 0.00 | 76.48 | 91.33 |
| XML_COMPACT | man | 89.00 | 35.00 | 0.00 | 71.78 | 90.89 |
| XML_COMPACT | opt | 90.67 | 33.33 | 0.00 | 73.12 | 90.61 |
| XML_PRETTY | man | 87.33 | 36.67 | 0.00 | 70.43 | 91.11 |
| XML_PRETTY | opt | 88.33 | 35.67 | 0.00 | 71.24 | 90.18 |
| YAML | man | 94.00 | 30.00 | 0.00 | 75.80 | 92.87 |
| YAML | opt | 96.00 | 28.00 | 0.00 | 77.42 | 92.94 |

#### 2.8.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Accuracy by Char (%) Man | Accuracy by Char (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 90.00 | 93.00 |  +3 |  +3.33 | 34.00 | 31.00 | -3 | -8.82 | 0.00 | 0.00 | 0 | 0.00 | 72.58 | 75.00 |  +2.42 | 90.98 | 91.57 |  +90.98 |
| JSON_PRETTY | 91.67 | 86.00 | -6 | -6.19 | 32.33 | 38.00 |  +6 |  +17.54 | 0.00 | 0.00 | 0 | 0.00 | 73.92 | 69.35 | -4.57 | 91.39 | 88.88 |  +91.39 |
| TOON_DEFAULT | 88.17 | 94.83 |  +7 |  +7.55 | 35.83 | 29.17 | -7 | -18.59 | 0.00 | 0.00 | 0 | 0.00 | 71.10 | 76.48 |  +5.38 | 92.19 | 91.33 |  +92.19 |
| XML_COMPACT | 89.00 | 90.67 |  +2 |  +1.88 | 35.00 | 33.33 | -2 | -4.77 | 0.00 | 0.00 | 0 | 0.00 | 71.78 | 73.12 |  +1.34 | 90.89 | 90.61 |  +90.89 |
| XML_PRETTY | 87.33 | 88.33 |  +1 |  +1.15 | 36.67 | 35.67 | -1 | -2.73 | 0.00 | 0.00 | 0 | 0.00 | 70.43 | 71.24 |  +0.81 | 91.11 | 90.18 |  +91.11 |
| YAML | 94.00 | 96.00 |  +2 |  +2.13 | 30.00 | 28.00 | -2 | -6.67 | 0.00 | 0.00 | 0 | 0.00 | 75.80 | 77.42 |  +1.62 | 92.87 | 92.94 |  +92.87 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 72.58 | 84.85 | 53.09 | 66.67 | 71.43 |
| JSON_COMPACT | opt | 75.00 | 93.94 | 64.20 | 61.91 | 52.38 |
| JSON_PRETTY | man | 73.92 | 96.36 | 58.03 | 60.31 | 49.20 |
| JSON_PRETTY | opt | 69.35 | 90.30 | 50.61 | 61.90 | 46.03 |
| TOON_DEFAULT | man | 71.10 | 91.21 | 43.21 | 59.52 | 65.87 |
| TOON_DEFAULT | opt | 76.48 | 96.36 | 65.43 | 71.43 | 43.65 |
| XML_COMPACT | man | 71.78 | 87.88 | 55.55 | 66.67 | 55.55 |
| XML_COMPACT | opt | 73.12 | 87.27 | 65.43 | 63.49 | 55.55 |
| XML_PRETTY | man | 70.43 | 88.48 | 55.55 | 53.97 | 58.73 |
| XML_PRETTY | opt | 71.24 | 89.09 | 55.56 | 63.49 | 52.38 |
| YAML | man | 75.80 | 97.58 | 55.56 | 60.31 | 60.32 |
| YAML | opt | 77.42 | 99.39 | 60.49 | 63.49 | 55.56 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 84.85 | 93.94 |  +9.09 |
| JSON_PRETTY | 96.36 | 90.30 | -6.06 |
| TOON_DEFAULT | 91.21 | 96.36 |  +5.15 |
| XML_COMPACT | 87.88 | 87.27 | -0.61 |
| XML_PRETTY | 88.48 | 89.09 |  +0.61 |
| YAML | 97.58 | 99.39 |  +1.82 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 53.09 | 64.20 |  +11.11 |
| JSON_PRETTY | 58.03 | 50.61 | -7.41 |
| TOON_DEFAULT | 43.21 | 65.43 |  +22.22 |
| XML_COMPACT | 55.55 | 65.43 |  +9.88 |
| XML_PRETTY | 55.55 | 55.56 |  +0.00 |
| YAML | 55.56 | 60.49 |  +4.93 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 66.67 | 61.91 | -4.76 |
| JSON_PRETTY | 60.31 | 61.90 |  +1.59 |
| TOON_DEFAULT | 59.52 | 71.43 |  +11.91 |
| XML_COMPACT | 66.67 | 63.49 | -3.18 |
| XML_PRETTY | 53.97 | 63.49 |  +9.52 |
| YAML | 60.31 | 63.49 |  +3.18 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 71.43 | 52.38 | -19.05 |
| JSON_PRETTY | 49.20 | 46.03 | -3.17 |
| TOON_DEFAULT | 65.87 | 43.65 | -22.22 |
| XML_COMPACT | 55.55 | 55.55 |  +0.00 |
| XML_PRETTY | 58.73 | 52.38 | -6.35 |
| YAML | 60.32 | 55.56 | -4.76 |

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
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)