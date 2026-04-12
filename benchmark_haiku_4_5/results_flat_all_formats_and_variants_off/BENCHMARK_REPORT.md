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
   - Optional: CSV 17689 tokens
   - Mandatory: CSV 17340 tokens
- Lowest read token cost:
   - Optional: CSV 6728 tokens
   - Mandatory: CSV 7022 tokens
- Lowest output token cost:
   - Optional: TOON_DEFAULT 8984 tokens
   - Mandatory: XML_COMPACT 6976 tokens
- Lowest output token cost drift:
   - Optional: YAML ↓ -17.42% ↑ 15.86%
   - Mandatory: JSON_PRETTY ↓ -7.29% ↑ 5.35%
- Highest accuracy:
   - Optional: YAML 78.23%
   - Mandatory: JSON_PRETTY 79.57%
- Lowest accuracy drift:
   - Optional: YAML ↓ -4.13% ↑ 4.12%
   - Mandatory: JSON_PRETTY ↓ -1.68% ↑ 1.36%
- Most useful read tokens:
   - Optional: XML_PRETTY 11256 / 15117 tokens
   - Mandatory: XML_PRETTY 11748 / 16186 tokens
- Most useful output tokens:
   - Optional: YAML 10193 / 13029 tokens
   - Mandatory: JSON_COMPACT 10253 / 13489 tokens
- Highest read efficiency (%/token):
   - Optional: JSON_COMPACT 75.80
   - Mandatory: TOON_DEFAULT 80.86
- Highest output efficiency (%/token):
   - Optional: TOON_DEFAULT 71.83
   - Mandatory: XML_COMPACT 80.59
- Highest accuracy by char:
   - Optional: JSON_PRETTY 92.07%
   - Mandatory: JSON_PRETTY 92.57%
- Lowest accuracy by char drift:
   - Optional: YAML ↓ -1.68% ↑ 1.16%
   - Mandatory: JSON_COMPACT ↓ -0.65% ↑ 0.66%
- Most useful output write tokens (Acc By Char):
   - Optional: YAML 11694 / 12725 tokens
   - Mandatory: JSON_COMPACT 12127 / 13185 tokens
- Highest output write efficiency (Acc By Char) (%/token):
   - Optional: TOON_DEFAULT 83.46
   - Mandatory: XML_COMPACT 94.25
- Lowest delta (optional-mandatory):
   - Read tokens: CSV -294 tokens
   - Output tokens: CSV 643 tokens
   - Accuracy: TOON_DEFAULT -0.54%
   - Read efficiency: CSV 0.18
   - Output efficiency: CSV -4.13
   - Accuracy by char: CSV 0.12%
   - Output write efficiency (Acc By Char): CSV -2.88

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: YAML 24820 tokens
   - Mandatory: XML_PRETTY 26786 tokens
- Highest read token cost:
   - Optional: XML_PRETTY 15117 tokens
   - Mandatory: XML_PRETTY 16186 tokens
- Highest output token cost:
   - Optional: YAML 13029 tokens
   - Mandatory: JSON_COMPACT 13489 tokens
- Highest output token drift:
   - Optional: JSON_COMPACT ↓ -44.06% ↑ 59.10%
   - Mandatory: CSV ↓ -16.06% ↑ 39.46%
- Lowest accuracy:
   - Optional: CSV 63.23%
   - Mandatory: CSV 64.51%
- Highest accuracy drift:
   - Optional: CSV ↓ -15.82% ↑ 14.79%
   - Mandatory: TOON_DEFAULT ↓ -22.36% ↑ 9.36%
- Most wasted read tokens:
   - Optional: XML_PRETTY 3861 / 15117 tokens
   - Mandatory: XML_PRETTY 4438 / 16186 tokens
- Most wasted output tokens:
   - Optional: CSV 4031 / 10961 tokens
   - Mandatory: CSV 3662 / 10318 tokens
- Lowest read efficiency (%/token):
   - Optional: XML_PRETTY 53.43
   - Mandatory: XML_PRETTY 48.42
- Lowest output efficiency (%/token):
   - Optional: YAML 54.54
   - Mandatory: JSON_COMPACT 50.72
- Lowest accuracy by char:
   - Optional: CSV 85.93%
   - Mandatory: CSV 85.81%
- Highest accuracy by char drift:
   - Optional: CSV ↓ -5.24% ↑ 5.56%
   - Mandatory: XML_COMPACT ↓ -3.56% ↑ 5.25%
- Most wasted output write tokens (Acc By Char):
   - Optional: CSV 9158 / 10658 tokens
   - Mandatory: CSV 8648 / 10078 tokens
- Lowest output write efficiency (Acc By Char) (%/token):
   - Optional: YAML 63.66
   - Mandatory: JSON_COMPACT 61.37
- Highest delta (optional-mandatory):
   - Read tokens: TOON_DEFAULT 4527 tokens
   - Output tokens: XML_COMPACT 4290 tokens
   - Accuracy: YAML 7.26%
   - Read efficiency: TOON_DEFAULT -16.28
   - Output efficiency: JSON_COMPACT 19.06
   - Accuracy by char: XML_PRETTY -1.46%
   - Output write efficiency (Acc By Char): XML_COMPACT -22.56

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy By Char | ↓ Eff Score Output Write (Acc By Char) | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 76s | CSV ≈ 7022 | CSV ≈ 241 | XML_COMPACT ≈ 6673 | XML_COMPACT ≈ 6976 | CSV ≈ 17340 | JSON_PRETTY ≈ 93% | XML_COMPACT ≈ 94 | JSON_PRETTY ≈ 80% | TOON_DEFAULT ≈ 81 | XML_COMPACT ≈ 81 | TOON_DEFAULT ≈ 77 |
| XML_COMPACT (+5.8%) | TOON_DEFAULT (+2.3%) | TOON_DEFAULT (+7.0%) | YAML (+35.9%) | YAML (+34.4%) | TOON_DEFAULT (+8.4%) | XML_PRETTY (-0.4%) | YAML (-12.8%) | JSON_COMPACT (-3.6%) | CSV (-6.9%) | YAML (-15.2%) | CSV (-1.3%) |
| CSV (+6.6%) | JSON_COMPACT (+32.4%) | JSON_PRETTY (+25.4%) | CSV (+51.0%) | CSV (+47.9%) | XML_COMPACT (+9.1%) | JSON_COMPACT (-0.6%) | XML_PRETTY (-19.1%) | TOON_DEFAULT (-5.8%) | JSON_COMPACT (-7.3%) | XML_PRETTY (-21.6%) | XML_COMPACT (-3.0%) |
| XML_PRETTY (+13.9%) | XML_COMPACT (+70.2%) | YAML (+25.4%) | XML_PRETTY (+54.3%) | XML_PRETTY (+52.0%) | YAML (+26.6%) | TOON_DEFAULT (-0.7%) | CSV (-22.4%) | XML_PRETTY (-7.0%) | XML_COMPACT (-23.0%) | JSON_PRETTY (-24.5%) | JSON_COMPACT (-16.2%) |
| TOON_DEFAULT (+25.2%) | YAML (+79.1%) | XML_COMPACT (+25.8%) | TOON_DEFAULT (+70.2%) | TOON_DEFAULT (+66.5%) | JSON_COMPACT (+31.4%) | YAML (-0.8%) | TOON_DEFAULT (-25.1%) | XML_COMPACT (-8.6%) | YAML (-25.7%) | CSV (-26.5%) | YAML (-16.7%) |
| JSON_PRETTY (+26.8%) | JSON_PRETTY (+103.8%) | JSON_COMPACT (+26.4%) | JSON_PRETTY (+75.0%) | JSON_PRETTY (+71.7%) | JSON_PRETTY (+51.6%) | XML_COMPACT (-1.1%) | JSON_PRETTY (-26.3%) | YAML (-8.6%) | JSON_PRETTY (-26.2%) | TOON_DEFAULT (-27.1%) | JSON_PRETTY (-29.1%) |
| JSON_COMPACT (+40.5%) | XML_PRETTY (+130.5%) | XML_PRETTY (+26.5%) | JSON_COMPACT (+97.6%) | JSON_COMPACT (+93.4%) | XML_PRETTY (+54.5%) | CSV (-6.8%) | JSON_COMPACT (-34.9%) | CSV (-15.1%) | XML_PRETTY (-40.1%) | JSON_COMPACT (-37.1%) | XML_PRETTY (-37.4%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy By Char | ↓ Eff Score Output Write (Acc By Char) | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 74s | CSV ≈ 6728 | JSON_PRETTY ≈ 231 | TOON_DEFAULT ≈ 8675 | TOON_DEFAULT ≈ 8984 | CSV ≈ 17689 | JSON_PRETTY ≈ 92% | TOON_DEFAULT ≈ 83 | YAML ≈ 78% | JSON_COMPACT ≈ 76 | TOON_DEFAULT ≈ 72 | JSON_COMPACT ≈ 80 |
| XML_PRETTY (+0.3%) | JSON_COMPACT (+30.3%) | XML_PRETTY (+30.7%) | XML_PRETTY (+2.7%) | XML_PRETTY (+2.5%) | JSON_COMPACT (+3.6%) | YAML (-0.2%) | XML_PRETTY (-1.4%) | JSON_PRETTY (-1.8%) | CSV (-0.5%) | XML_PRETTY (-0.4%) | CSV (-6.6%) |
| JSON_COMPACT (+2.1%) | XML_COMPACT (+66.2%) | CSV (+31.4%) | JSON_COMPACT (+6.7%) | JSON_COMPACT (+6.4%) | TOON_DEFAULT (+17.0%) | JSON_COMPACT (-0.9%) | JSON_COMPACT (-3.1%) | XML_COMPACT (-1.9%) | XML_COMPACT (-9.6%) | JSON_COMPACT (-2.9%) | TOON_DEFAULT (-11.6%) |
| JSON_PRETTY (+14.8%) | TOON_DEFAULT (+74.0%) | YAML (+31.6%) | JSON_PRETTY (+17.3%) | JSON_PRETTY (+15.9%) | XML_COMPACT (+26.9%) | XML_PRETTY (-1.4%) | JSON_PRETTY (-8.0%) | JSON_COMPACT (-3.7%) | YAML (-10.8%) | JSON_PRETTY (-7.1%) | XML_COMPACT (-16.7%) |
| CSV (+16.9%) | YAML (+75.3%) | JSON_COMPACT (+31.9%) | CSV (+22.9%) | CSV (+22.0%) | JSON_PRETTY (+34.6%) | TOON_DEFAULT (-1.5%) | XML_COMPACT (-14.1%) | XML_PRETTY (-3.8%) | TOON_DEFAULT (-14.8%) | XML_COMPACT (-13.3%) | JSON_PRETTY (-22.7%) |
| XML_COMPACT (+18.4%) | JSON_PRETTY (+99.1%) | XML_COMPACT (+33.8%) | XML_COMPACT (+26.3%) | XML_COMPACT (+25.4%) | XML_PRETTY (+37.5%) | XML_COMPACT (-1.7%) | CSV (-15.9%) | TOON_DEFAULT (-5.0%) | JSON_PRETTY (-19.8%) | CSV (-23.3%) | YAML (-25.7%) |
| YAML (+40.4%) | XML_PRETTY (+124.7%) | TOON_DEFAULT (+33.8%) | YAML (+46.7%) | YAML (+45.0%) | YAML (+40.3%) | CSV (-6.1%) | YAML (-23.7%) | CSV (-15.0%) | XML_PRETTY (-29.5%) | YAML (-24.1%) | XML_PRETTY (-26.6%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 98% | JSON_PRETTY ≈ 70% | JSON_PRETTY ≈ 67% | JSON_PRETTY ≈ 59% |
| YAML (-1.2%) | JSON_COMPACT (-1.9%) | XML_PRETTY (-3.2%) | XML_COMPACT (-1.6%) |
| JSON_PRETTY (-1.2%) | TOON_DEFAULT (-4.9%) | JSON_COMPACT (-3.6%) | CSV (-5.4%) |
| XML_PRETTY (-2.4%) | XML_COMPACT (-13.6%) | CSV (-4.8%) | TOON_DEFAULT (-9.5%) |
| TOON_DEFAULT (-5.9%) | CSV (-17.8%) | YAML (-4.8%) | YAML (-9.5%) |
| XML_COMPACT (-9.7%) | XML_PRETTY (-18.5%) | TOON_DEFAULT (-6.4%) | XML_PRETTY (-11.1%) |
| CSV (-22.5%) | YAML (-28.4%) | XML_COMPACT (-9.5%) | JSON_COMPACT (-18.3%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99% | XML_COMPACT ≈ 72% | YAML ≈ 73% | JSON_COMPACT ≈ 52% |
| XML_COMPACT (-1.8%) | XML_PRETTY (-1.2%) | JSON_PRETTY (-6.4%) | XML_PRETTY (0.0%) |
| JSON_PRETTY (-2.7%) | JSON_PRETTY (-6.4%) | CSV (-10.2%) | TOON_DEFAULT (-4.8%) |
| JSON_COMPACT (-4.5%) | YAML (-7.4%) | JSON_COMPACT (-11.1%) | JSON_PRETTY (-4.8%) |
| TOON_DEFAULT (-6.1%) | TOON_DEFAULT (-9.9%) | XML_COMPACT (-11.1%) | YAML (-6.3%) |
| XML_PRETTY (-8.5%) | JSON_COMPACT (-11.6%) | TOON_DEFAULT (-12.2%) | XML_COMPACT (-11.1%) |
| CSV (-20.1%) | CSV (-19.8%) | XML_PRETTY (-14.3%) | CSV (-16.2%) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy By Char (%) | Useful Output Write Tokens (Acc By Char) | Wasted Output Write Tokens (Acc By Char) | Eff Score Output Write (Acc By Char) | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7022 | 10318 | 17340 | 1.438 | 81.271 | 85.81 | 8647.59 | 1430.01 | 73.11 | 64.51 | 4529.892 | 2492.108 | 6656.271 | 3661.929 | 75.26 | 59.23 | 76.30 |
| CSV | opt | 6728 | 10961 | 17689 | 1.423 | 85.950 | 85.93 | 9158.25 | 1499.55 | 70.23 | 63.23 | 4254.114 | 2473.886 | 6930.893 | 4030.507 | 75.44 | 55.09 | 74.21 |
| JSON_COMPACT | man | 9296 | 13489 | 22785 | 2.143 | 106.327 | 91.98 | 12127.10 | 1057.40 | 61.37 | 76.01 | 7065.890 | 2230.110 | 10252.609 | 3235.891 | 74.93 | 50.72 | 64.79 |
| JSON_COMPACT | opt | 8768 | 9558 | 18326 | 2.108 | 74.621 | 91.14 | 8433.18 | 819.82 | 80.87 | 74.52 | 6533.914 | 2234.086 | 7122.473 | 2435.327 | 75.80 | 69.78 | 79.50 |
| JSON_PRETTY | man | 14311 | 11977 | 26288 | 1.691 | 94.153 | 92.57 | 10807.55 | 867.45 | 69.46 | 79.57 | 11387.263 | 2923.737 | 9529.834 | 2446.833 | 59.67 | 60.81 | 54.83 |
| JSON_PRETTY | opt | 13395 | 10411 | 23806 | 1.677 | 82.097 | 92.07 | 9372.73 | 807.27 | 76.76 | 76.45 | 10240.478 | 3154.522 | 7959.210 | 2451.790 | 60.81 | 66.72 | 61.49 |
| TOON_DEFAULT | man | 7182 | 11615 | 18797 | 1.415 | 91.593 | 91.87 | 10434.14 | 923.36 | 70.62 | 73.75 | 5296.725 | 1885.275 | 8566.063 | 3048.937 | 80.86 | 58.77 | 77.33 |
| TOON_DEFAULT | opt | 11709 | 8984 | 20693 | 1.677 | 69.959 | 90.60 | 7859.55 | 815.45 | 83.46 | 73.21 | 8572.159 | 3136.841 | 6577.309 | 2406.859 | 64.58 | 71.83 | 70.29 |
| XML_COMPACT | man | 11950 | 6976 | 18926 | 2.315 | 53.817 | 91.46 | 6103.43 | 569.90 | 94.25 | 70.97 | 8480.915 | 3469.085 | 4950.867 | 2025.133 | 62.24 | 80.59 | 75.02 |
| XML_COMPACT | opt | 11181 | 11266 | 22447 | 2.288 | 88.360 | 90.41 | 9905.92 | 1050.74 | 71.69 | 76.34 | 8535.575 | 2645.425 | 8600.210 | 2665.457 | 68.52 | 62.28 | 66.20 |
| XML_PRETTY | man | 16186 | 10600 | 26786 | 1.931 | 83.032 | 92.14 | 9486.73 | 809.27 | 76.21 | 72.58 | 11747.799 | 4438.201 | 7693.722 | 2906.611 | 48.42 | 63.17 | 48.42 |
| XML_PRETTY | opt | 15117 | 9210 | 24327 | 1.913 | 71.836 | 90.68 | 8077.47 | 830.20 | 82.32 | 74.46 | 11256.118 | 3860.882 | 6857.518 | 2352.149 | 53.43 | 71.52 | 58.33 |
| YAML | man | 12574 | 9373 | 21947 | 1.661 | 73.153 | 91.74 | 8321.74 | 749.26 | 82.20 | 70.97 | 8923.768 | 3650.232 | 6651.782 | 2720.885 | 60.05 | 68.36 | 64.38 |
| YAML | opt | 11791 | 13029 | 24820 | 1.646 | 102.621 | 91.90 | 11694.27 | 1030.72 | 63.66 | 78.23 | 9224.099 | 2566.901 | 10192.587 | 2836.413 | 67.64 | 54.54 | 59.11 |

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
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 28 | 250.786 | 0.90 | 34797 | 0.290 | 280.62 | 34825 | 251.076 | 224.68 |
| CSV | opt | 10 | 672.800 | 0.32 | 36882 | 0.289 | 297.43 | 36892 | 673.089 | 238.01 |
| JSON_COMPACT | man | 36 | 258.222 | 1.16 | 35225 | 0.374 | 284.07 | 35261 | 258.596 | 227.49 |
| JSON_COMPACT | opt | 9 | 974.222 | 0.29 | 35563 | 0.260 | 286.80 | 35572 | 974.482 | 229.50 |
| JSON_PRETTY | man | 13 | 1100.846 | 0.42 | 35817 | 0.326 | 288.85 | 35830 | 1101.172 | 231.16 |
| JSON_PRETTY | opt | 15 | 893.000 | 0.48 | 37347 | 0.273 | 301.19 | 37362 | 893.273 | 241.05 |
| TOON_DEFAULT | man | 27 | 266.000 | 0.87 | 38129 | 0.298 | 307.49 | 38156 | 266.298 | 246.17 |
| TOON_DEFAULT | opt | 27 | 433.667 | 0.87 | 37047 | 0.237 | 298.77 | 37074 | 433.904 | 239.19 |
| XML_COMPACT | man | 16 | 746.875 | 0.52 | 47309 | 0.141 | 381.53 | 47325 | 747.016 | 305.32 |
| XML_COMPACT | opt | 15 | 745.400 | 0.48 | 35007 | 0.313 | 282.32 | 35022 | 745.713 | 225.95 |
| XML_PRETTY | man | 18 | 899.222 | 0.58 | 36393 | 0.283 | 293.49 | 36411 | 899.505 | 234.91 |
| XML_PRETTY | opt | 9 | 1679.667 | 0.29 | 36231 | 0.246 | 292.18 | 36240 | 1679.913 | 233.80 |
| YAML | man | 11 | 1143.091 | 0.35 | 34805 | 0.261 | 280.69 | 34816 | 1143.352 | 224.62 |
| YAML | opt | 8 | 1473.875 | 0.26 | 37991 | 0.335 | 306.38 | 37999 | 1474.210 | 245.15 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 28 | 10 | -18 | -64.29 | 34.80 | 36.88 |  +2.08 |  +5.99 | 34.83 | 36.89 |  +2.07 |  +5.93 |
| JSON_COMPACT | 36 | 9 | -27 | -75.00 | 35.22 | 35.56 |  +0.34 |  +0.96 | 35.26 | 35.57 |  +0.31 |  +0.88 |
| JSON_PRETTY | 13 | 15 |  +2 |  +15.38 | 35.82 | 37.35 |  +1.53 |  +4.27 | 35.83 | 37.36 |  +1.53 |  +4.28 |
| TOON_DEFAULT | 27 | 27 | 0 | 0.00 | 38.13 | 37.05 | -1.08 | -2.84 | 38.16 | 37.07 | -1.08 | -2.83 |
| XML_COMPACT | 16 | 15 | -1 | -6.25 | 47.31 | 35.01 | -12.30 | -26.00 | 47.33 | 35.02 | -12.30 | -26.00 |
| XML_PRETTY | 18 | 9 | -9 | -50.00 | 36.39 | 36.23 | -0.16 | -0.45 | 36.41 | 36.24 | -0.17 | -0.47 |
| YAML | 11 | 8 | -3 | -27.27 | 34.80 | 37.99 |  +3.19 |  +9.15 | 34.82 | 38.00 |  +3.18 |  +9.14 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token |
|---|---|---|---|---|---|---|---|
| CSV | man | 1.438 | 10.296 | 226.516 | 0.919 | 0.625 | 0.372 |
| CSV | opt | 1.423 | 10.662 | 217.032 | 0.940 | 0.577 | 0.357 |
| JSON_COMPACT | man | 2.143 | 13.630 | 299.871 | 0.818 | 0.564 | 0.334 |
| JSON_COMPACT | opt | 2.108 | 13.895 | 282.839 | 0.850 | 0.780 | 0.407 |
| JSON_PRETTY | man | 1.691 | 20.984 | 461.645 | 0.556 | 0.664 | 0.303 |
| JSON_PRETTY | opt | 1.677 | 21.228 | 432.097 | 0.571 | 0.734 | 0.321 |
| TOON_DEFAULT | man | 1.415 | 10.531 | 231.677 | 1.027 | 0.636 | 0.393 |
| TOON_DEFAULT | opt | 1.677 | 18.556 | 377.710 | 0.625 | 0.839 | 0.356 |
| XML_COMPACT | man | 2.315 | 17.522 | 385.484 | 0.594 | 1.017 | 0.375 |
| XML_COMPACT | opt | 2.288 | 17.719 | 360.677 | 0.683 | 0.678 | 0.340 |
| XML_PRETTY | man | 1.931 | 23.733 | 522.129 | 0.448 | 0.685 | 0.271 |
| XML_PRETTY | opt | 1.913 | 23.957 | 487.645 | 0.493 | 0.808 | 0.306 |
| YAML | man | 1.661 | 18.437 | 405.613 | 0.564 | 0.757 | 0.323 |
| YAML | opt | 1.646 | 18.686 | 380.355 | 0.663 | 0.600 | 0.315 |

#### 2.5.2 Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.438 | 1.423 | -0.015 | -1.04 | 10.296 | 10.662 |  +0.366 |  +3.55 | 226.516 | 217.032 | -9.484 | -4.19 | 0.919 | 0.940 |  +0.021 |  +2.29 | 0.625 | 0.577 | -0.048 | -7.68 | 0.372 | 0.357 | -0.015 | -4.03 |
| JSON_COMPACT | 2.143 | 2.108 | -0.035 | -1.63 | 13.630 | 13.895 |  +0.265 |  +1.94 | 299.871 | 282.839 | -17.032 | -5.68 | 0.818 | 0.850 |  +0.032 |  +3.91 | 0.564 | 0.780 |  +0.216 |  +38.30 | 0.334 | 0.407 |  +0.073 |  +21.86 |
| JSON_PRETTY | 1.691 | 1.677 | -0.014 | -0.83 | 20.984 | 21.228 |  +0.244 |  +1.16 | 461.645 | 432.097 | -29.548 | -6.40 | 0.556 | 0.571 |  +0.015 |  +2.70 | 0.664 | 0.734 |  +0.070 |  +10.54 | 0.303 | 0.321 |  +0.018 |  +5.94 |
| TOON_DEFAULT | 1.415 | 1.677 |  +0.262 |  +18.52 | 10.531 | 18.556 |  +8.025 |  +76.20 | 231.677 | 377.710 |  +146.033 |  +63.03 | 1.027 | 0.625 | -0.402 | -39.14 | 0.636 | 0.839 |  +0.203 |  +31.92 | 0.393 | 0.356 | -0.037 | -9.54 |
| XML_COMPACT | 2.315 | 2.288 | -0.027 | -1.17 | 17.522 | 17.719 |  +0.197 |  +1.12 | 385.484 | 360.677 | -24.807 | -6.44 | 0.594 | 0.683 |  +0.089 |  +14.98 | 1.017 | 0.678 | -0.339 | -33.33 | 0.375 | 0.340 | -0.035 | -9.33 |
| XML_PRETTY | 1.931 | 1.913 | -0.018 | -0.93 | 23.733 | 23.957 |  +0.224 |  +0.94 | 522.129 | 487.645 | -34.484 | -6.60 | 0.448 | 0.493 |  +0.045 |  +10.04 | 0.685 | 0.808 |  +0.123 |  +17.96 | 0.271 | 0.306 |  +0.035 |  +12.92 |
| YAML | 1.661 | 1.646 | -0.015 | -0.90 | 18.437 | 18.686 |  +0.249 |  +1.35 | 405.613 | 380.355 | -25.258 | -6.23 | 0.564 | 0.663 |  +0.099 |  +17.55 | 0.757 | 0.600 | -0.157 | -20.74 | 0.323 | 0.315 | -0.008 | -2.48 |

### 2.6 Output Write Token Utilization Efficiency (Accuracy By Char)
#### 2.6.1 Metrics
| Format | Variant | Output Write Tokens | Useful Output Write Tokens (Acc By Char) | Wasted Output Write Tokens (Acc By Char) | Accuracy by Char (%) | Eff Score Output Write (Acc By Char) |
|---|---|---|---|---|---|---|
| CSV | man | 10078 | 8648 | 1430 | 85.81 | 73.11 |
| CSV | opt | 10658 | 9158 | 1500 | 85.93 | 70.23 |
| JSON_COMPACT | man | 13185 | 12127 | 1057 | 91.98 | 61.37 |
| JSON_COMPACT | opt | 9253 | 8433 | 820 | 91.14 | 80.87 |
| JSON_PRETTY | man | 11675 | 10808 | 867 | 92.57 | 69.46 |
| JSON_PRETTY | opt | 10180 | 9373 | 807 | 92.07 | 76.76 |
| TOON_DEFAULT | man | 11358 | 10434 | 923 | 91.87 | 70.62 |
| TOON_DEFAULT | opt | 8675 | 7860 | 815 | 90.60 | 83.46 |
| XML_COMPACT | man | 6673 | 6103 | 570 | 91.46 | 94.25 |
| XML_COMPACT | opt | 10957 | 9906 | 1051 | 90.41 | 71.69 |
| XML_PRETTY | man | 10296 | 9487 | 809 | 92.14 | 76.21 |
| XML_PRETTY | opt | 8908 | 8077 | 830 | 90.68 | 82.32 |
| YAML | man | 9071 | 8322 | 749 | 91.74 | 82.20 |
| YAML | opt | 12725 | 11694 | 1031 | 91.90 | 63.66 |

#### 2.6.2 Mandatory vs Optional
| Format | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Useful Output Write Tokens (Acc By Char) Man | Useful Output Write Tokens (Acc By Char) Opt | Diff | Diff (%) | Wasted Output Write Tokens (Acc By Char) Man | Wasted Output Write Tokens (Acc By Char) Opt | Diff | Diff (%) | Accuracy By Char (%) Man | Accuracy By Char (%) Opt | Diff (%) | Eff Score Output Write (Acc By Char) Man | Eff Score Output Write (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 10078 | 10658 |  +580 |  +5.76 | 8648 | 9159 |  +511 |  +5.90 | 1430 | 1500 |  +70 |  +4.86 | 85.81 | 85.93 |  +0.12 |  +0.14 | 73.11 | 70.23 | -2.88 | -3.94 |
| JSON_COMPACT | 13185 | 9254 | -3932 | -29.82 | 12127 | 8433 | -3694 | -30.46 | 1057 | 819 | -238 | -22.48 | 91.98 | 91.14 | -0.84 | -0.91 | 61.37 | 80.87 |  +19.50 |  +31.78 |
| JSON_PRETTY | 11675 | 10180 | -1495 | -12.81 | 10808 | 9373 | -1435 | -13.28 | 867 | 807 | -60 | -6.94 | 92.57 | 92.07 | -0.50 | -0.54 | 69.46 | 76.76 |  +7.30 |  +10.50 |
| TOON_DEFAULT | 11358 | 8676 | -2683 | -23.62 | 10434 | 7859 | -2575 | -24.67 | 923 | 815 | -108 | -11.69 | 91.87 | 90.60 | -1.27 | -1.38 | 70.62 | 83.46 |  +12.84 |  +18.19 |
| XML_COMPACT | 6673 | 10956 |  +4283 |  +64.19 | 6103 | 9905 |  +3802 |  +62.31 | 570 | 1051 |  +481 |  +84.36 | 91.46 | 90.41 | -1.05 | -1.15 | 94.25 | 71.69 | -22.56 | -23.94 |
| XML_PRETTY | 10296 | 8908 | -1388 | -13.48 | 9487 | 8078 | -1409 | -14.85 | 809 | 830 |  +21 |  +2.59 | 92.14 | 90.68 | -1.46 | -1.58 | 76.21 | 82.32 |  +6.11 |  +8.02 |
| YAML | 9071 | 12725 |  +3654 |  +40.28 | 8322 | 11695 |  +3373 |  +40.53 | 749 | 1030 |  +281 |  +37.58 | 91.74 | 91.90 |  +0.16 |  +0.17 | 82.20 | 63.66 | -18.54 | -22.56 |

### 2.7 Token Utilization Efficiency
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7022 | 4530 | 2492 | 10318 | 6656 | 3662 | 17340 | 11186 | 6154 | 64.51 | 75.26 | 59.23 | 76.30 | 63.26 | 74.43 | 58.40 | 75.46 |
| CSV | opt | 6728 | 4254 | 2474 | 10961 | 6931 | 4031 | 17689 | 11185 | 6504 | 63.23 | 75.44 | 55.09 | 74.21 | 62.47 | 74.94 | 54.59 | 73.71 |
| JSON_COMPACT | man | 9296 | 7066 | 2230 | 13489 | 10253 | 3236 | 22785 | 17318 | 5466 | 76.01 | 74.93 | 50.72 | 64.79 | 75.01 | 74.27 | 50.05 | 64.13 |
| JSON_COMPACT | opt | 8768 | 6534 | 2234 | 9558 | 7122 | 2435 | 18326 | 13656 | 4669 | 74.52 | 75.80 | 69.78 | 79.50 | 72.53 | 74.47 | 68.45 | 78.17 |
| JSON_PRETTY | man | 14311 | 11387 | 2924 | 11977 | 9530 | 2447 | 26288 | 20917 | 5371 | 79.57 | 59.67 | 60.81 | 54.83 | 78.12 | 58.70 | 59.84 | 53.87 |
| JSON_PRETTY | opt | 13395 | 10240 | 3155 | 10411 | 7959 | 2452 | 23806 | 18200 | 5606 | 76.45 | 60.81 | 66.72 | 61.49 | 75.12 | 59.92 | 65.83 | 60.60 |
| TOON_DEFAULT | man | 7182 | 5297 | 1885 | 11615 | 8566 | 3049 | 18797 | 13863 | 4934 | 73.75 | 80.86 | 58.77 | 77.33 | 72.42 | 79.97 | 57.89 | 76.44 |
| TOON_DEFAULT | opt | 11709 | 8572 | 3137 | 8984 | 6577 | 2407 | 20693 | 15149 | 5544 | 73.21 | 64.58 | 71.83 | 70.29 | 71.63 | 63.53 | 70.78 | 69.24 |
| XML_COMPACT | man | 11950 | 8481 | 3469 | 6976 | 4951 | 2025 | 18926 | 13432 | 5494 | 70.97 | 62.24 | 80.59 | 75.02 | 68.79 | 60.79 | 79.13 | 73.57 |
| XML_COMPACT | opt | 11181 | 8536 | 2645 | 11266 | 8600 | 2665 | 22447 | 17136 | 5311 | 76.34 | 68.52 | 62.28 | 66.20 | 75.53 | 67.98 | 61.74 | 65.66 |
| XML_PRETTY | man | 16186 | 11748 | 4438 | 10600 | 7694 | 2907 | 26786 | 19442 | 7345 | 72.58 | 48.42 | 63.17 | 48.42 | 70.21 | 46.84 | 61.59 | 46.84 |
| XML_PRETTY | opt | 15117 | 11256 | 3861 | 9210 | 6858 | 2352 | 24327 | 18114 | 6213 | 74.46 | 53.43 | 71.52 | 58.33 | 73.40 | 52.72 | 70.81 | 57.62 |
| YAML | man | 12574 | 8924 | 3650 | 9373 | 6652 | 2721 | 21947 | 15576 | 6371 | 70.97 | 60.05 | 68.36 | 64.38 | 67.65 | 57.83 | 66.15 | 62.17 |
| YAML | opt | 11791 | 9224 | 2567 | 13029 | 10193 | 2836 | 24820 | 19417 | 5403 | 78.23 | 67.64 | 54.54 | 59.11 | 76.96 | 66.79 | 53.70 | 58.26 |

#### 2.7.2 Read Tokens Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7022 | 6728 | -294 | -4.19 | 4530 | 4254 | -276 | -6.09 | 2492 | 2474 | -18 | -0.73 | 64.51 | 63.23 | -1.28 | -1.98 | 75.26 | 75.44 |  +0.18 |  +0.24 | 63.26 | 62.47 | -0.79 | -1.25 | 74.43 | 74.94 |  +0.51 |  +0.68 |
| JSON_COMPACT | 9296 | 8768 | -528 | -5.68 | 7066 | 6534 | -532 | -7.53 | 2230 | 2234 |  +4 |  +0.18 | 76.01 | 74.52 | -1.49 | -1.96 | 74.93 | 75.80 |  +0.86 |  +1.15 | 75.01 | 72.53 | -2.48 | -3.31 | 74.27 | 74.47 |  +0.20 |  +0.27 |
| JSON_PRETTY | 14311 | 13395 | -916 | -6.40 | 11387 | 10240 | -1147 | -10.07 | 2924 | 3155 |  +231 |  +7.89 | 79.57 | 76.45 | -3.12 | -3.92 | 59.67 | 60.81 |  +1.14 |  +1.91 | 78.12 | 75.12 | -3.00 | -3.84 | 58.70 | 59.92 |  +1.22 |  +2.08 |
| TOON_DEFAULT | 7182 | 11709 |  +4527 |  +63.03 | 5297 | 8572 |  +3275 |  +61.84 | 1885 | 3137 |  +1252 |  +66.40 | 73.75 | 73.21 | -0.54 | -0.73 | 80.86 | 64.58 | -16.28 | -20.13 | 72.42 | 71.63 | -0.79 | -1.09 | 79.97 | 63.53 | -16.45 | -20.56 |
| XML_COMPACT | 11950 | 11181 | -769 | -6.44 | 8481 | 8536 |  +55 |  +0.64 | 3469 | 2645 | -824 | -23.74 | 70.97 | 76.34 |  +5.37 |  +7.57 | 62.24 | 68.52 |  +6.28 |  +10.10 | 68.79 | 75.53 |  +6.74 |  +9.80 | 60.79 | 67.98 |  +7.20 |  +11.84 |
| XML_PRETTY | 16186 | 15117 | -1069 | -6.60 | 11748 | 11256 | -492 | -4.19 | 4438 | 3861 | -577 | -13.01 | 72.58 | 74.46 |  +1.88 |  +2.59 | 48.42 | 53.43 |  +5.01 |  +10.35 | 70.21 | 73.40 |  +3.19 |  +4.54 | 46.84 | 52.72 |  +5.89 |  +12.57 |
| YAML | 12574 | 11791 | -783 | -6.23 | 8924 | 9224 |  +300 |  +3.37 | 3650 | 2567 | -1083 | -29.68 | 70.97 | 78.23 |  +7.26 |  +10.23 | 60.05 | 67.64 |  +7.59 |  +12.65 | 67.65 | 76.96 |  +9.31 |  +13.76 | 57.83 | 66.79 |  +8.96 |  +15.49 |

#### 2.7.3 Output Tokens Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 10318 | 10961 |  +643 |  +6.23 | 6656 | 6931 |  +275 |  +4.13 | 3662 | 4031 |  +369 |  +10.06 | 64.51 | 63.23 | -1.28 | -1.98 | 59.23 | 55.09 | -4.13 | -6.98 | 63.26 | 62.47 | -0.79 | -1.25 | 58.40 | 54.59 | -3.81 | -6.52 |
| JSON_COMPACT | 13489 | 9558 | -3931 | -29.14 | 10253 | 7123 | -3130 | -30.53 | 3236 | 2435 | -801 | -24.74 | 76.01 | 74.52 | -1.49 | -1.96 | 50.72 | 69.78 |  +19.06 |  +37.58 | 75.01 | 72.53 | -2.48 | -3.31 | 50.05 | 68.45 |  +18.40 |  +36.77 |
| JSON_PRETTY | 11977 | 10411 | -1566 | -13.07 | 9530 | 7959 | -1571 | -16.48 | 2447 | 2452 |  +5 |  +0.20 | 79.57 | 76.45 | -3.12 | -3.92 | 60.81 | 66.72 |  +5.91 |  +9.72 | 78.12 | 75.12 | -3.00 | -3.84 | 59.84 | 65.83 |  +5.99 |  +10.01 |
| TOON_DEFAULT | 11615 | 8984 | -2631 | -22.65 | 8566 | 6577 | -1989 | -23.22 | 3049 | 2407 | -642 | -21.06 | 73.75 | 73.21 | -0.54 | -0.73 | 58.77 | 71.83 |  +13.06 |  +22.23 | 72.42 | 71.63 | -0.79 | -1.09 | 57.89 | 70.78 |  +12.90 |  +22.28 |
| XML_COMPACT | 6976 | 11266 |  +4290 |  +61.49 | 4951 | 8600 |  +3649 |  +73.71 | 2025 | 2665 |  +640 |  +31.62 | 70.97 | 76.34 |  +5.37 |  +7.57 | 80.59 | 62.28 | -18.31 | -22.72 | 68.79 | 75.53 |  +6.74 |  +9.80 | 79.13 | 61.74 | -17.39 | -21.98 |
| XML_PRETTY | 10600 | 9209 | -1391 | -13.12 | 7694 | 6858 | -836 | -10.87 | 2907 | 2353 | -554 | -19.07 | 72.58 | 74.46 |  +1.88 |  +2.59 | 63.17 | 71.52 |  +8.35 |  +13.22 | 70.21 | 73.40 |  +3.19 |  +4.54 | 61.59 | 70.81 |  +9.22 |  +14.97 |
| YAML | 9373 | 13029 |  +3656 |  +39.01 | 6652 | 10193 |  +3541 |  +53.23 | 2721 | 2837 |  +116 |  +4.25 | 70.97 | 78.23 |  +7.26 |  +10.23 | 68.36 | 54.54 | -13.81 | -20.21 | 67.65 | 76.96 |  +9.31 |  +13.76 | 66.15 | 53.70 | -12.45 | -18.82 |

#### 2.7.4 Total Tokens Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 17340 | 17689 |  +349 |  +2.01 | 11186 | 11185 | -1 | -0.01 | 6154 | 6504 |  +350 |  +5.69 | 64.51 | 63.23 | -1.28 | -1.98 | 76.30 | 74.21 | -2.08 | -2.73 | 63.26 | 62.47 | -0.79 | -1.25 | 75.46 | 73.71 | -1.76 | -2.33 |
| JSON_COMPACT | 22785 | 18326 | -4459 | -19.57 | 17318 | 13656 | -3662 | -21.15 | 5466 | 4669 | -797 | -14.57 | 76.01 | 74.52 | -1.49 | -1.96 | 64.79 | 79.50 |  +14.71 |  +22.70 | 75.01 | 72.53 | -2.48 | -3.31 | 64.13 | 78.17 |  +14.05 |  +21.90 |
| JSON_PRETTY | 26288 | 23806 | -2482 | -9.44 | 20917 | 18200 | -2717 | -12.99 | 5371 | 5607 |  +236 |  +4.39 | 79.57 | 76.45 | -3.12 | -3.92 | 54.83 | 61.49 |  +6.66 |  +12.14 | 78.12 | 75.12 | -3.00 | -3.84 | 53.87 | 60.60 |  +6.74 |  +12.51 |
| TOON_DEFAULT | 18797 | 20693 |  +1896 |  +10.09 | 13863 | 15150 |  +1287 |  +9.28 | 4934 | 5543 |  +609 |  +12.35 | 73.75 | 73.21 | -0.54 | -0.73 | 77.33 | 70.29 | -7.04 | -9.10 | 72.42 | 71.63 | -0.79 | -1.09 | 76.44 | 69.24 | -7.20 | -9.42 |
| XML_COMPACT | 18926 | 22447 |  +3521 |  +18.60 | 13432 | 17136 |  +3704 |  +27.58 | 5494 | 5311 | -183 | -3.34 | 70.97 | 76.34 |  +5.37 |  +7.57 | 75.02 | 66.20 | -8.82 | -11.75 | 68.79 | 75.53 |  +6.74 |  +9.80 | 73.57 | 65.66 | -7.90 | -10.74 |
| XML_PRETTY | 26786 | 24326 | -2460 | -9.18 | 19442 | 18114 | -1328 | -6.83 | 7345 | 6213 | -1132 | -15.41 | 72.58 | 74.46 |  +1.88 |  +2.59 | 48.42 | 58.33 |  +9.91 |  +20.48 | 70.21 | 73.40 |  +3.19 |  +4.54 | 46.84 | 57.62 |  +10.79 |  +23.03 |
| YAML | 21947 | 24820 |  +2873 |  +13.09 | 15576 | 19417 |  +3841 |  +24.66 | 6371 | 5403 | -968 | -15.19 | 70.97 | 78.23 |  +7.26 |  +10.23 | 64.38 | 59.11 | -5.28 | -8.20 | 67.65 | 76.96 |  +9.31 |  +13.76 | 62.17 | 58.26 | -3.91 | -6.29 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Accuracy by Char (%) |
|---|---|---|---|---|---|---|
| CSV | man | 80.00 | 44.00 | 0.00 | 64.51 | 85.81 |
| CSV | opt | 78.40 | 45.60 | -0.00 | 63.23 | 85.93 |
| JSON_COMPACT | man | 94.25 | 29.75 | 0.00 | 76.01 | 91.98 |
| JSON_COMPACT | opt | 92.40 | 31.60 | -0.00 | 74.52 | 91.14 |
| JSON_PRETTY | man | 98.67 | 25.33 | 0.00 | 79.57 | 92.57 |
| JSON_PRETTY | opt | 94.80 | 29.20 | 0.00 | 76.45 | 92.07 |
| TOON_DEFAULT | man | 91.44 | 32.56 | 0.00 | 73.75 | 91.87 |
| TOON_DEFAULT | opt | 90.78 | 33.22 | 0.00 | 73.21 | 90.60 |
| XML_COMPACT | man | 88.00 | 36.00 | 0.00 | 70.97 | 91.46 |
| XML_COMPACT | opt | 94.67 | 29.33 | 0.00 | 76.34 | 90.41 |
| XML_PRETTY | man | 90.00 | 34.00 | 0.00 | 72.58 | 92.14 |
| XML_PRETTY | opt | 92.33 | 31.67 | 0.00 | 74.46 | 90.68 |
| YAML | man | 88.00 | 36.00 | 0.00 | 70.97 | 91.74 |
| YAML | opt | 97.00 | 27.00 | 0.00 | 78.23 | 91.90 |

#### 2.8.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Accuracy by Char (%) Man | Accuracy by Char (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 80.00 | 78.40 | -2 | -2.00 | 44.00 | 45.60 |  +2 |  +3.64 | 0.00 | -0.00 | -0 | 0.00 | 64.51 | 63.23 | -1.28 | 85.81 | 85.93 |  +85.81 |
| JSON_COMPACT | 94.25 | 92.40 | -2 | -1.96 | 29.75 | 31.60 |  +2 |  +6.22 | 0.00 | -0.00 | -0 | 0.00 | 76.01 | 74.52 | -1.49 | 91.98 | 91.14 |  +91.98 |
| JSON_PRETTY | 98.67 | 94.80 | -4 | -3.92 | 25.33 | 29.20 |  +4 |  +15.28 | 0.00 | 0.00 |  +0 | 0.00 | 79.57 | 76.45 | -3.12 | 92.57 | 92.07 |  +92.57 |
| TOON_DEFAULT | 91.44 | 90.78 | -1 | -0.72 | 32.56 | 33.22 |  +1 |  +2.03 | 0.00 | 0.00 | 0 | 0.00 | 73.75 | 73.21 | -0.54 | 91.87 | 90.60 |  +91.87 |
| XML_COMPACT | 88.00 | 94.67 |  +7 |  +7.58 | 36.00 | 29.33 | -7 | -18.53 | 0.00 | 0.00 | 0 | 0.00 | 70.97 | 76.34 |  +5.37 | 91.46 | 90.41 |  +91.46 |
| XML_PRETTY | 90.00 | 92.33 |  +2 |  +2.59 | 34.00 | 31.67 | -2 | -6.85 | 0.00 | 0.00 | 0 | 0.00 | 72.58 | 74.46 |  +1.88 | 92.14 | 90.68 |  +92.14 |
| YAML | 88.00 | 97.00 |  +9 |  +10.23 | 36.00 | 27.00 | -9 | -25.00 | 0.00 | 0.00 | 0 | 0.00 | 70.97 | 78.23 |  +7.26 | 91.74 | 91.90 |  +91.74 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| CSV | man | 64.51 | 75.64 | 52.59 | 61.90 | 53.33 |
| CSV | opt | 63.23 | 79.27 | 51.85 | 62.86 | 36.19 |
| JSON_COMPACT | man | 76.01 | 98.18 | 68.52 | 63.09 | 40.48 |
| JSON_COMPACT | opt | 74.52 | 94.91 | 60.00 | 61.90 | 52.38 |
| JSON_PRETTY | man | 79.57 | 96.97 | 70.37 | 66.67 | 58.73 |
| JSON_PRETTY | opt | 76.45 | 96.73 | 65.19 | 66.67 | 47.62 |
| TOON_DEFAULT | man | 73.75 | 92.32 | 65.43 | 60.32 | 49.21 |
| TOON_DEFAULT | opt | 73.21 | 93.33 | 61.73 | 60.84 | 47.62 |
| XML_COMPACT | man | 70.97 | 88.49 | 56.79 | 57.14 | 57.14 |
| XML_COMPACT | opt | 76.34 | 97.58 | 71.61 | 61.90 | 41.27 |
| XML_PRETTY | man | 72.58 | 95.76 | 51.85 | 63.49 | 47.62 |
| XML_PRETTY | opt | 74.46 | 90.91 | 70.37 | 58.73 | 52.38 |
| YAML | man | 70.97 | 96.97 | 41.97 | 61.90 | 49.21 |
| YAML | opt | 78.23 | 99.39 | 64.20 | 73.02 | 46.03 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 75.64 | 79.27 |  +3.64 |
| JSON_COMPACT | 98.18 | 94.91 | -3.27 |
| JSON_PRETTY | 96.97 | 96.73 | -0.24 |
| TOON_DEFAULT | 92.32 | 93.33 |  +1.01 |
| XML_COMPACT | 88.49 | 97.58 |  +9.09 |
| XML_PRETTY | 95.76 | 90.91 | -4.85 |
| YAML | 96.97 | 99.39 |  +2.42 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 52.59 | 51.85 | -0.74 |
| JSON_COMPACT | 68.52 | 60.00 | -8.52 |
| JSON_PRETTY | 70.37 | 65.19 | -5.18 |
| TOON_DEFAULT | 65.43 | 61.73 | -3.70 |
| XML_COMPACT | 56.79 | 71.61 |  +14.82 |
| XML_PRETTY | 51.85 | 70.37 |  +18.52 |
| YAML | 41.97 | 64.20 |  +22.23 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 61.90 | 62.86 |  +0.95 |
| JSON_COMPACT | 63.09 | 61.90 | -1.19 |
| JSON_PRETTY | 66.67 | 66.67 | -0.00 |
| TOON_DEFAULT | 60.32 | 60.84 |  +0.53 |
| XML_COMPACT | 57.14 | 61.90 |  +4.76 |
| XML_PRETTY | 63.49 | 58.73 | -4.77 |
| YAML | 61.90 | 73.02 |  +11.11 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 53.33 | 36.19 | -17.14 |
| JSON_COMPACT | 40.48 | 52.38 |  +11.90 |
| JSON_PRETTY | 58.73 | 47.62 | -11.11 |
| TOON_DEFAULT | 49.21 | 47.62 | -1.59 |
| XML_COMPACT | 57.14 | 41.27 | -15.87 |
| XML_PRETTY | 47.62 | 52.38 |  +4.76 |
| YAML | 49.21 | 46.03 | -3.17 |

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

- **Report Generated**: 2026-04-12
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code < 2.1.86
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_flat_all_formats_and_variants_on\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_off\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_on_verify\BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)