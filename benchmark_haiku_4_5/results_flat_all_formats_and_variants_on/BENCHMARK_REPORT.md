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
   - Optional: JSON_COMPACT 18753 tokens
   - Mandatory: CSV 16525 tokens
- Lowest read token cost:
   - Optional: CSV 6700 tokens
   - Mandatory: CSV 6989 tokens
- Lowest output token cost:
   - Optional: JSON_PRETTY 8302 tokens
   - Mandatory: XML_COMPACT 9429 tokens
- Lowest output token cost drift:
   - Optional: CSV ↓ -3.13% ↑ 2.21%
   - Mandatory: JSON_COMPACT ↓ -9.44% ↑ 7.51%
- Highest accuracy:
   - Optional: XML_PRETTY 77.96%
   - Mandatory: JSON_PRETTY 79.30%
- Lowest accuracy drift:
   - Optional: XML_PRETTY ↓ -3.80% ↑ 2.41%
   - Mandatory: JSON_COMPACT ↓ 0.00% ↑ 0.00%
- Most useful read tokens:
   - Optional: XML_PRETTY 11763 / 15089 tokens
   - Mandatory: XML_PRETTY 12125 / 16166 tokens
- Most useful output tokens:
   - Optional: TOON_DEFAULT 9126 / 11768 tokens
   - Mandatory: JSON_PRETTY 10419 / 13138 tokens
- Highest read efficiency (%/token):
   - Optional: JSON_COMPACT 76.99
   - Mandatory: TOON_DEFAULT 83.23
- Highest output efficiency (%/token):
   - Optional: JSON_PRETTY 82.91
   - Mandatory: XML_COMPACT 76.59
- Highest accuracy by char:
   - Optional: XML_PRETTY 92.40%
   - Mandatory: JSON_PRETTY 93.58%
- Lowest accuracy by char drift:
   - Optional: XML_COMPACT ↓ -0.67% ↑ 0.55%
   - Mandatory: YAML ↓ -0.50% ↑ 0.58%
- Most useful output write tokens (Acc By Char):
   - Optional: CSV 10850 / 12614 tokens
   - Mandatory: JSON_PRETTY 11976 / 12797 tokens
- Highest output write efficiency (Acc By Char) (%/token):
   - Optional: JSON_PRETTY 93.89
   - Mandatory: XML_COMPACT 88.76
- Lowest delta (optional-mandatory):
   - Read tokens: CSV -289 tokens
   - Output tokens: YAML -76 tokens
   - Accuracy: TOON_DEFAULT 0.80%
   - Read efficiency: JSON_PRETTY -0.01
   - Output efficiency: TOON_DEFAULT 0.01
   - Accuracy by char: JSON_COMPACT -0.01%
   - Output write efficiency (Acc By Char): YAML 0.42

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 25117 tokens
   - Mandatory: XML_PRETTY 28946 tokens
- Highest read token cost:
   - Optional: XML_PRETTY 15089 tokens
   - Mandatory: XML_PRETTY 16166 tokens
- Highest output token cost:
   - Optional: CSV 12845 tokens
   - Mandatory: JSON_COMPACT 13161 tokens
- Highest output token drift:
   - Optional: JSON_COMPACT ↓ -42.41% ↑ 49.43%
   - Mandatory: CSV ↓ -39.52% ↑ 33.02%
- Lowest accuracy:
   - Optional: CSV 60.22%
   - Mandatory: CSV 61.83%
- Highest accuracy drift:
   - Optional: JSON_COMPACT ↓ -11.27% ↑ 7.75%
   - Mandatory: CSV ↓ -19.13% ↑ 14.78%
- Most wasted read tokens:
   - Optional: JSON_PRETTY 3414 / 13367 tokens
   - Mandatory: XML_PRETTY 4042 / 16166 tokens
- Most wasted output tokens:
   - Optional: CSV 5110 / 12845 tokens
   - Mandatory: JSON_COMPACT 3715 / 13161 tokens
- Lowest read efficiency (%/token):
   - Optional: XML_PRETTY 55.79
   - Mandatory: XML_PRETTY 50.03
- Lowest output efficiency (%/token):
   - Optional: CSV 48.65
   - Mandatory: JSON_COMPACT 54.63
- Lowest accuracy by char:
   - Optional: CSV 86.02%
   - Mandatory: CSV 84.25%
- Highest accuracy by char drift:
   - Optional: JSON_COMPACT ↓ -2.35% ↑ 2.15%
   - Mandatory: CSV ↓ -6.94% ↑ 4.77%
- Most wasted output write tokens (Acc By Char):
   - Optional: CSV 10850 / 12614 tokens
   - Mandatory: CSV 7757 / 9207 tokens
- Lowest output write efficiency (Acc By Char) (%/token):
   - Optional: CSV 65.26
   - Mandatory: JSON_COMPACT 67.01
- Highest delta (optional-mandatory):
   - Read tokens: TOON_DEFAULT 4513 tokens
   - Output tokens: JSON_PRETTY -4836 tokens
   - Accuracy: JSON_PRETTY -4.84%
   - Read efficiency: TOON_DEFAULT -15.32
   - Output efficiency: JSON_PRETTY 23.14
   - Accuracy by char: JSON_PRETTY -2.65%
   - Output write efficiency (Acc By Char): JSON_PRETTY 24.59

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy By Char | ↓ Eff Score Output Write (Acc By Char) | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|
| XML_COMPACT ≈ 75s | CSV ≈ 6989 | JSON_COMPACT ≈ 228 | XML_COMPACT ≈ 9086 | XML_COMPACT ≈ 9429 | CSV ≈ 16525 | JSON_PRETTY ≈ 94% | XML_COMPACT ≈ 89 | JSON_PRETTY ≈ 79% | TOON_DEFAULT ≈ 83 | XML_COMPACT ≈ 77 | TOON_DEFAULT ≈ 79 |
| CSV (+3.9%) | TOON_DEFAULT (+0.8%) | YAML (+1.5%) | CSV (+1.3%) | CSV (+1.1%) | TOON_DEFAULT (+13.3%) | TOON_DEFAULT (-0.4%) | CSV (-6.9%) | TOON_DEFAULT (-2.5%) | CSV (-11.7%) | YAML (-8.9%) | CSV (-5.2%) |
| YAML (+17.2%) | JSON_COMPACT (+32.6%) | TOON_DEFAULT (+26.1%) | YAML (+14.0%) | YAML (+12.3%) | XML_COMPACT (+27.8%) | XML_COMPACT (-1.2%) | YAML (-8.0%) | XML_PRETTY (-4.3%) | JSON_COMPACT (-13.4%) | CSV (-11.5%) | XML_COMPACT (-10.4%) |
| TOON_DEFAULT (+27.4%) | XML_COMPACT (+67.3%) | CSV (+44.7%) | TOON_DEFAULT (+25.3%) | TOON_DEFAULT (+23.8%) | JSON_COMPACT (+35.7%) | XML_PRETTY (-1.2%) | TOON_DEFAULT (-13.5%) | XML_COMPACT (-5.1%) | XML_COMPACT (-21.7%) | TOON_DEFAULT (-13.7%) | JSON_COMPACT (-16.9%) |
| XML_PRETTY (+36.2%) | YAML (+79.6%) | JSON_PRETTY (+49.8%) | XML_PRETTY (+36.9%) | XML_PRETTY (+35.5%) | YAML (+40.0%) | YAML (-1.5%) | XML_PRETTY (-20.6%) | YAML (-5.9%) | YAML (-25.9%) | JSON_PRETTY (-22.0%) | YAML (-17.9%) |
| JSON_PRETTY (+40.7%) | JSON_PRETTY (+104.4%) | XML_COMPACT (+50.5%) | JSON_PRETTY (+40.8%) | JSON_PRETTY (+39.3%) | JSON_PRETTY (+65.9%) | JSON_COMPACT (-2.3%) | JSON_PRETTY (-21.9%) | JSON_COMPACT (-7.5%) | JSON_PRETTY (-28.5%) | XML_PRETTY (-23.2%) | JSON_PRETTY (-27.5%) |
| JSON_COMPACT (+45.8%) | XML_PRETTY (+131.3%) | XML_PRETTY (+50.5%) | JSON_COMPACT (+42.3%) | JSON_COMPACT (+39.6%) | XML_PRETTY (+75.2%) | CSV (-9.3%) | JSON_COMPACT (-24.5%) | CSV (-17.5%) | XML_PRETTY (-39.9%) | JSON_COMPACT (-28.7%) | XML_PRETTY (-36.3%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy By Char | ↓ Eff Score Output Write (Acc By Char) | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 73s | CSV ≈ 6700 | CSV ≈ 231 | JSON_PRETTY ≈ 7966 | JSON_PRETTY ≈ 8302 | JSON_COMPACT ≈ 18753 | XML_PRETTY ≈ 92% | JSON_PRETTY ≈ 94 | XML_PRETTY ≈ 78% | JSON_COMPACT ≈ 77 | JSON_PRETTY ≈ 83 | JSON_COMPACT ≈ 78 |
| XML_PRETTY (+7.9%) | JSON_COMPACT (+30.6%) | XML_PRETTY (+0.6%) | JSON_COMPACT (+21.4%) | JSON_COMPACT (+20.5%) | CSV (+4.2%) | XML_COMPACT (-0.2%) | XML_PRETTY (-9.6%) | TOON_DEFAULT (-0.4%) | CSV (-4.6%) | XML_PRETTY (-8.5%) | JSON_PRETTY (-11.6%) |
| JSON_COMPACT (+13.2%) | XML_COMPACT (+63.1%) | YAML (+44.3%) | XML_PRETTY (+23.0%) | XML_PRETTY (+20.8%) | JSON_PRETTY (+15.5%) | TOON_DEFAULT (-0.9%) | JSON_COMPACT (-9.7%) | XML_COMPACT (-0.5%) | XML_COMPACT (-9.0%) | JSON_COMPACT (-9.7%) | XML_COMPACT (-11.7%) |
| YAML (+21.0%) | TOON_DEFAULT (+72.6%) | XML_COMPACT (+45.2%) | YAML (+27.7%) | YAML (+26.6%) | YAML (+18.8%) | JSON_COMPACT (-1.2%) | YAML (-12.6%) | JSON_COMPACT (-1.6%) | TOON_DEFAULT (-11.8%) | YAML (-13.2%) | YAML (-12.3%) |
| TOON_DEFAULT (+28.9%) | YAML (+75.7%) | JSON_PRETTY (+45.5%) | XML_COMPACT (+40.1%) | XML_COMPACT (+38.5%) | XML_COMPACT (+19.6%) | YAML (-1.2%) | XML_COMPACT (-17.7%) | YAML (-1.9%) | YAML (-14.0%) | XML_COMPACT (-18.6%) | TOON_DEFAULT (-14.6%) |
| XML_COMPACT (+30.1%) | JSON_PRETTY (+99.5%) | JSON_COMPACT (+46.2%) | TOON_DEFAULT (+43.5%) | TOON_DEFAULT (+41.7%) | TOON_DEFAULT (+24.4%) | JSON_PRETTY (-1.5%) | TOON_DEFAULT (-19.7%) | JSON_PRETTY (-3.5%) | JSON_PRETTY (-22.7%) | TOON_DEFAULT (-20.3%) | CSV (-16.4%) |
| CSV (+50.1%) | XML_PRETTY (+125.2%) | TOON_DEFAULT (+47.3%) | CSV (+58.3%) | CSV (+54.7%) | XML_PRETTY (+33.9%) | CSV (-6.4%) | CSV (-30.5%) | CSV (-17.7%) | XML_PRETTY (-27.5%) | CSV (-41.3%) | XML_PRETTY (-20.4%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 99% | JSON_PRETTY ≈ 67% | YAML ≈ 73% | TOON_DEFAULT ≈ 67% |
| YAML (0.0%) | XML_PRETTY (-4.9%) | XML_PRETTY (-4.8%) | JSON_PRETTY (-7.9%) |
| JSON_COMPACT (-1.2%) | XML_COMPACT (-4.9%) | TOON_DEFAULT (-7.9%) | XML_COMPACT (-12.7%) |
| XML_PRETTY (-1.2%) | TOON_DEFAULT (-9.3%) | JSON_PRETTY (-7.9%) | CSV (-19.0%) |
| TOON_DEFAULT (-4.2%) | CSV (-11.1%) | XML_COMPACT (-9.5%) | XML_PRETTY (-27.0%) |
| XML_COMPACT (-6.7%) | JSON_COMPACT (-11.1%) | JSON_COMPACT (-14.3%) | YAML (-27.0%) |
| CSV (-24.8%) | YAML (-18.5%) | CSV (-20.6%) | JSON_COMPACT (-28.6%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| XML_COMPACT ≈ 99% | JSON_COMPACT ≈ 74% | JSON_COMPACT ≈ 71% | XML_PRETTY ≈ 62% |
| XML_PRETTY (-0.6%) | TOON_DEFAULT (-1.9%) | TOON_DEFAULT (-0.0%) | JSON_COMPACT (-14.3%) |
| YAML (-1.2%) | YAML (-4.9%) | YAML (-1.6%) | TOON_DEFAULT (-14.3%) |
| JSON_PRETTY (-3.6%) | JSON_PRETTY (-7.4%) | XML_COMPACT (-4.8%) | XML_COMPACT (-15.9%) |
| TOON_DEFAULT (-4.8%) | XML_COMPACT (-7.4%) | XML_PRETTY (-6.4%) | JSON_PRETTY (-19.0%) |
| JSON_COMPACT (-8.5%) | XML_PRETTY (-14.8%) | CSV (-7.9%) | CSV (-27.0%) |
| CSV (-23.6%) | CSV (-27.2%) | JSON_PRETTY (-9.5%) | YAML (-27.0%) |


#### 2.1.5 Conclusion

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy By Char (%) | Useful Output Write Tokens (Acc By Char) | Wasted Output Write Tokens (Acc By Char) | Eff Score Output Write (Acc By Char) | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6989 | 9536 | 16525 | 1.444 | 74.250 | 84.25 | 7756.90 | 1450.10 | 82.67 | 61.83 | 4321.299 | 2667.701 | 5896.315 | 3640.018 | 73.50 | 67.76 | 74.52 |
| CSV | opt | 6700 | 12845 | 19545 | 1.429 | 101.723 | 86.02 | 10850.28 | 1763.39 | 65.26 | 60.22 | 4034.740 | 2665.260 | 7735.058 | 5109.609 | 73.44 | 48.65 | 65.36 |
| JSON_COMPACT | man | 9268 | 13161 | 22429 | 2.149 | 104.298 | 91.26 | 11802.66 | 1130.34 | 67.01 | 71.77 | 6651.644 | 2616.356 | 9445.411 | 3715.256 | 72.11 | 54.63 | 65.33 |
| JSON_COMPACT | opt | 8748 | 10005 | 18753 | 2.113 | 77.965 | 91.25 | 8821.75 | 845.92 | 84.82 | 76.34 | 6678.223 | 2069.777 | 7638.071 | 2367.262 | 76.99 | 74.88 | 78.22 |
| JSON_PRETTY | man | 14283 | 13138 | 27421 | 1.694 | 103.204 | 93.58 | 11975.74 | 821.59 | 69.29 | 79.30 | 11326.419 | 2956.581 | 10418.698 | 2719.635 | 59.51 | 59.77 | 56.97 |
| JSON_PRETTY | opt | 13367 | 8302 | 21669 | 1.680 | 64.242 | 90.93 | 7243.48 | 722.52 | 93.89 | 74.46 | 9953.068 | 3413.932 | 6181.669 | 2120.331 | 59.51 | 82.91 | 69.16 |
| TOON_DEFAULT | man | 7048 | 11671 | 18719 | 1.442 | 91.805 | 93.22 | 10612.01 | 771.82 | 76.77 | 76.75 | 5409.340 | 1638.660 | 8957.364 | 2713.469 | 83.23 | 66.07 | 78.59 |
| TOON_DEFAULT | opt | 11561 | 11768 | 23329 | 1.698 | 92.155 | 91.54 | 10460.58 | 966.75 | 75.41 | 77.55 | 8965.556 | 2595.445 | 9125.697 | 2641.804 | 67.91 | 66.08 | 66.77 |
| XML_COMPACT | man | 11693 | 9429 | 21122 | 2.366 | 73.274 | 92.40 | 8395.46 | 690.54 | 88.76 | 74.20 | 8676.206 | 3016.794 | 6996.071 | 2432.596 | 65.21 | 76.59 | 70.45 |
| XML_COMPACT | opt | 10930 | 11498 | 22428 | 2.340 | 90.024 | 92.19 | 10291.17 | 871.83 | 77.28 | 77.42 | 8462.006 | 2467.994 | 8902.009 | 2596.324 | 70.04 | 67.46 | 69.10 |
| XML_PRETTY | man | 16166 | 12780 | 28946 | 1.934 | 100.304 | 92.36 | 11487.43 | 950.24 | 70.44 | 75.00 | 12124.500 | 4041.500 | 9585.250 | 3195.083 | 50.03 | 58.86 | 50.02 |
| XML_PRETTY | opt | 15089 | 10028 | 25117 | 1.917 | 79.000 | 92.40 | 9051.50 | 744.50 | 84.88 | 77.96 | 11763.384 | 3325.616 | 7818.088 | 2210.245 | 55.79 | 75.83 | 62.25 |
| YAML | man | 12554 | 10585 | 23139 | 1.664 | 83.500 | 92.06 | 9531.89 | 822.11 | 81.61 | 73.39 | 9213.381 | 3340.619 | 7768.332 | 2816.669 | 61.65 | 69.75 | 64.51 |
| YAML | opt | 11771 | 10509 | 22280 | 1.649 | 82.065 | 91.23 | 9283.57 | 892.43 | 82.03 | 76.07 | 8954.200 | 2816.800 | 7994.450 | 2514.883 | 66.19 | 71.95 | 68.59 |

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
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 20 | 349.450 | 0.65 | 37620 | 0.245 | 303.39 | 37640 | 349.695 | 242.84 |
| CSV | opt | 15 | 446.667 | 0.48 | 41210 | 0.306 | 332.34 | 41225 | 446.973 | 265.97 |
| JSON_COMPACT | man | 28 | 331.000 | 0.90 | 40123 | 0.322 | 323.57 | 40151 | 331.322 | 259.04 |
| JSON_COMPACT | opt | 10 | 874.800 | 0.32 | 40816 | 0.237 | 329.16 | 40826 | 875.037 | 263.39 |
| JSON_PRETTY | man | 25 | 571.320 | 0.81 | 38019 | 0.337 | 306.60 | 38044 | 571.657 | 245.45 |
| JSON_PRETTY | opt | 13 | 1028.231 | 0.42 | 40608 | 0.196 | 327.48 | 40621 | 1028.427 | 262.07 |
| TOON_DEFAULT | man | 3 | 2349.333 | 0.10 | 37603 | 0.302 | 303.25 | 37606 | 2349.635 | 242.62 |
| TOON_DEFAULT | opt | 5 | 2312.200 | 0.16 | 39099 | 0.304 | 315.31 | 39104 | 2312.505 | 252.28 |
| XML_COMPACT | man | 4 | 2923.250 | 0.13 | 37181 | 0.244 | 299.84 | 37185 | 2923.494 | 239.90 |
| XML_COMPACT | opt | 9 | 1214.444 | 0.29 | 37367 | 0.299 | 301.34 | 37376 | 1214.743 | 241.13 |
| XML_PRETTY | man | 11 | 1469.636 | 0.35 | 39564 | 0.314 | 319.06 | 39575 | 1469.950 | 255.32 |
| XML_PRETTY | opt | 17 | 887.588 | 0.55 | 32583 | 0.301 | 262.77 | 32600 | 887.889 | 210.32 |
| YAML | man | 8 | 1569.250 | 0.26 | 39528 | 0.262 | 318.78 | 39536 | 1569.512 | 255.07 |
| YAML | opt | 11 | 1070.091 | 0.35 | 39265 | 0.259 | 316.65 | 39276 | 1070.350 | 253.39 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 20 | 15 | -5 | -25.00 | 37.62 | 41.21 |  +3.59 |  +9.54 | 37.64 | 41.23 |  +3.58 |  +9.52 |
| JSON_COMPACT | 28 | 10 | -18 | -64.29 | 40.12 | 40.82 |  +0.69 |  +1.73 | 40.15 | 40.83 |  +0.67 |  +1.68 |
| JSON_PRETTY | 25 | 13 | -12 | -48.00 | 38.02 | 40.61 |  +2.59 |  +6.81 | 38.04 | 40.62 |  +2.58 |  +6.77 |
| TOON_DEFAULT | 3 | 5 |  +2 |  +66.67 | 37.60 | 39.10 |  +1.50 |  +3.98 | 37.61 | 39.10 |  +1.50 |  +3.98 |
| XML_COMPACT | 4 | 9 |  +5 |  +125.00 | 37.18 | 37.37 |  +0.19 |  +0.50 | 37.18 | 37.38 |  +0.19 |  +0.51 |
| XML_PRETTY | 11 | 17 |  +6 |  +54.55 | 39.56 | 32.58 | -6.98 | -17.64 | 39.58 | 32.60 | -6.97 | -17.62 |
| YAML | 8 | 11 |  +3 |  +37.50 | 39.53 | 39.26 | -0.26 | -0.67 | 39.54 | 39.28 | -0.26 | -0.66 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token |
|---|---|---|---|---|---|---|---|
| CSV | man | 1.444 | 10.248 | 225.452 | 0.885 | 0.648 | 0.374 |
| CSV | opt | 1.429 | 10.618 | 216.129 | 0.899 | 0.469 | 0.308 |
| JSON_COMPACT | man | 2.149 | 13.589 | 298.968 | 0.774 | 0.545 | 0.320 |
| JSON_COMPACT | opt | 2.113 | 13.864 | 282.194 | 0.873 | 0.763 | 0.407 |
| JSON_PRETTY | man | 1.694 | 20.943 | 460.742 | 0.555 | 0.604 | 0.289 |
| JSON_PRETTY | opt | 1.680 | 21.184 | 431.194 | 0.557 | 0.897 | 0.344 |
| TOON_DEFAULT | man | 1.442 | 10.334 | 227.355 | 1.089 | 0.676 | 0.414 |
| TOON_DEFAULT | opt | 1.698 | 18.322 | 372.935 | 0.671 | 0.694 | 0.337 |
| XML_COMPACT | man | 2.366 | 17.145 | 377.194 | 0.635 | 0.787 | 0.351 |
| XML_COMPACT | opt | 2.340 | 17.322 | 352.581 | 0.708 | 0.673 | 0.345 |
| XML_PRETTY | man | 1.934 | 23.704 | 521.484 | 0.464 | 0.587 | 0.259 |
| XML_PRETTY | opt | 1.917 | 23.913 | 486.742 | 0.517 | 0.777 | 0.310 |
| YAML | man | 1.664 | 18.408 | 404.968 | 0.585 | 0.693 | 0.317 |
| YAML | opt | 1.649 | 18.655 | 379.710 | 0.646 | 0.724 | 0.341 |

#### 2.5.2 Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.444 | 1.429 | -0.015 | -1.04 | 10.248 | 10.618 |  +0.370 |  +3.61 | 225.452 | 216.129 | -9.323 | -4.14 | 0.885 | 0.899 |  +0.014 |  +1.58 | 0.648 | 0.469 | -0.179 | -27.62 | 0.374 | 0.308 | -0.066 | -17.65 |
| JSON_COMPACT | 2.149 | 2.113 | -0.036 | -1.68 | 13.589 | 13.864 |  +0.275 |  +2.02 | 298.968 | 282.194 | -16.774 | -5.61 | 0.774 | 0.873 |  +0.099 |  +12.79 | 0.545 | 0.763 |  +0.218 |  +40.00 | 0.320 | 0.407 |  +0.087 |  +27.19 |
| JSON_PRETTY | 1.694 | 1.680 | -0.014 | -0.83 | 20.943 | 21.184 |  +0.241 |  +1.15 | 460.742 | 431.194 | -29.548 | -6.41 | 0.555 | 0.557 |  +0.002 |  +0.36 | 0.604 | 0.897 |  +0.293 |  +48.51 | 0.289 | 0.344 |  +0.055 |  +19.03 |
| TOON_DEFAULT | 1.442 | 1.698 |  +0.256 |  +17.75 | 10.334 | 18.322 |  +7.988 |  +77.30 | 227.355 | 372.935 |  +145.580 |  +64.03 | 1.089 | 0.671 | -0.418 | -38.38 | 0.676 | 0.694 |  +0.017 |  +2.59 | 0.414 | 0.337 | -0.078 | -18.70 |
| XML_COMPACT | 2.366 | 2.340 | -0.026 | -1.10 | 17.145 | 17.322 |  +0.177 |  +1.03 | 377.194 | 352.581 | -24.613 | -6.53 | 0.635 | 0.708 |  +0.073 |  +11.50 | 0.787 | 0.673 | -0.114 | -14.49 | 0.351 | 0.345 | -0.006 | -1.71 |
| XML_PRETTY | 1.934 | 1.917 | -0.017 | -0.88 | 23.704 | 23.913 |  +0.209 |  +0.88 | 521.484 | 486.742 | -34.742 | -6.66 | 0.464 | 0.517 |  +0.053 |  +11.42 | 0.587 | 0.777 |  +0.190 |  +32.37 | 0.259 | 0.310 |  +0.051 |  +19.69 |
| YAML | 1.664 | 1.649 | -0.015 | -0.90 | 18.408 | 18.655 |  +0.247 |  +1.34 | 404.968 | 379.710 | -25.258 | -6.24 | 0.585 | 0.646 |  +0.061 |  +10.43 | 0.693 | 0.724 |  +0.031 |  +4.47 | 0.317 | 0.341 |  +0.024 |  +7.57 |

### 2.6 Output Write Token Utilization Efficiency (Accuracy By Char)
#### 2.6.1 Metrics
| Format | Variant | Output Write Tokens | Useful Output Write Tokens (Acc By Char) | Wasted Output Write Tokens (Acc By Char) | Accuracy by Char (%) | Eff Score Output Write (Acc By Char) |
|---|---|---|---|---|---|---|
| CSV | man | 9207 | 7757 | 1450 | 84.25 | 82.67 |
| CSV | opt | 12614 | 10850 | 1763 | 86.02 | 65.26 |
| JSON_COMPACT | man | 12933 | 11803 | 1130 | 91.26 | 67.01 |
| JSON_COMPACT | opt | 9668 | 8822 | 846 | 91.25 | 84.82 |
| JSON_PRETTY | man | 12797 | 11976 | 822 | 93.58 | 69.29 |
| JSON_PRETTY | opt | 7966 | 7243 | 723 | 90.93 | 93.89 |
| TOON_DEFAULT | man | 11384 | 10612 | 772 | 93.22 | 76.77 |
| TOON_DEFAULT | opt | 11427 | 10461 | 967 | 91.54 | 75.41 |
| XML_COMPACT | man | 9086 | 8395 | 691 | 92.40 | 88.76 |
| XML_COMPACT | opt | 11163 | 10291 | 872 | 92.19 | 77.28 |
| XML_PRETTY | man | 12438 | 11487 | 950 | 92.36 | 70.44 |
| XML_PRETTY | opt | 9796 | 9052 | 744 | 92.40 | 84.88 |
| YAML | man | 10354 | 9532 | 822 | 92.06 | 81.61 |
| YAML | opt | 10176 | 9284 | 892 | 91.23 | 82.03 |

#### 2.6.2 Mandatory vs Optional
| Format | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Useful Output Write Tokens (Acc By Char) Man | Useful Output Write Tokens (Acc By Char) Opt | Diff | Diff (%) | Wasted Output Write Tokens (Acc By Char) Man | Wasted Output Write Tokens (Acc By Char) Opt | Diff | Diff (%) | Accuracy By Char (%) Man | Accuracy By Char (%) Opt | Diff (%) | Eff Score Output Write (Acc By Char) Man | Eff Score Output Write (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 9207 | 12614 |  +3407 |  +37.00 | 7757 | 10850 |  +3093 |  +39.88 | 1450 | 1763 |  +313 |  +21.61 | 84.25 | 86.02 |  +1.77 |  +2.10 | 82.67 | 65.26 | -17.41 | -21.06 |
| JSON_COMPACT | 12933 | 9668 | -3265 | -25.25 | 11803 | 8822 | -2981 | -25.26 | 1130 | 846 | -284 | -25.17 | 91.26 | 91.25 | -0.01 | -0.01 | 67.01 | 84.82 |  +17.81 |  +26.58 |
| JSON_PRETTY | 12797 | 7966 | -4831 | -37.75 | 11976 | 7244 | -4732 | -39.51 | 822 | 723 | -99 | -12.05 | 93.58 | 90.93 | -2.65 | -2.83 | 69.29 | 93.89 |  +24.59 |  +35.49 |
| TOON_DEFAULT | 11384 | 11428 |  +44 |  +0.38 | 10612 | 10461 | -151 | -1.43 | 772 | 967 |  +195 |  +25.25 | 93.22 | 91.54 | -1.68 | -1.80 | 76.77 | 75.41 | -1.36 | -1.77 |
| XML_COMPACT | 9086 | 11163 |  +2077 |  +22.86 | 8395 | 10291 |  +1896 |  +22.58 | 691 | 872 |  +181 |  +26.24 | 92.40 | 92.19 | -0.21 | -0.23 | 88.76 | 77.28 | -11.47 | -12.93 |
| XML_PRETTY | 12438 | 9796 | -2642 | -21.24 | 11487 | 9051 | -2436 | -21.21 | 950 | 744 | -206 | -21.66 | 92.36 | 92.40 |  +0.04 |  +0.04 | 70.44 | 84.88 |  +14.44 |  +20.50 |
| YAML | 10354 | 10176 | -178 | -1.72 | 9532 | 9284 | -248 | -2.61 | 822 | 892 |  +70 |  +8.56 | 92.06 | 91.23 | -0.83 | -0.90 | 81.61 | 82.03 |  +0.42 |  +0.51 |

### 2.7 Token Utilization Efficiency
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6989 | 4321 | 2668 | 9536 | 5896 | 3640 | 16525 | 10218 | 6308 | 61.83 | 73.50 | 67.76 | 74.52 | 60.79 | 72.80 | 67.07 | 73.83 |
| CSV | opt | 6700 | 4035 | 2665 | 12845 | 7735 | 5110 | 19545 | 11770 | 7775 | 60.22 | 73.44 | 48.65 | 65.36 | 59.45 | 72.92 | 48.14 | 64.84 |
| JSON_COMPACT | man | 9268 | 6652 | 2616 | 13161 | 9445 | 3715 | 22429 | 16097 | 6332 | 71.77 | 72.11 | 54.63 | 65.33 | 69.79 | 70.79 | 53.31 | 64.01 |
| JSON_COMPACT | opt | 8748 | 6678 | 2070 | 10005 | 7638 | 2367 | 18753 | 14316 | 4437 | 76.34 | 76.99 | 74.88 | 78.22 | 76.30 | 76.96 | 74.85 | 78.20 |
| JSON_PRETTY | man | 14283 | 11326 | 2957 | 13138 | 10419 | 2720 | 27421 | 21745 | 5676 | 79.30 | 59.51 | 59.77 | 56.97 | 77.39 | 58.24 | 58.50 | 55.70 |
| JSON_PRETTY | opt | 13367 | 9953 | 3414 | 8302 | 6182 | 2120 | 21669 | 16135 | 5534 | 74.46 | 59.51 | 82.91 | 69.16 | 73.38 | 58.78 | 82.19 | 68.44 |
| TOON_DEFAULT | man | 7048 | 5409 | 1639 | 11671 | 8957 | 2713 | 18719 | 14367 | 4352 | 76.75 | 83.23 | 66.07 | 78.59 | 74.09 | 81.46 | 64.30 | 76.81 |
| TOON_DEFAULT | opt | 11561 | 8966 | 2595 | 11768 | 9126 | 2642 | 23329 | 18091 | 5237 | 77.55 | 67.91 | 66.08 | 66.77 | 77.12 | 67.62 | 65.79 | 66.49 |
| XML_COMPACT | man | 11693 | 8676 | 3017 | 9429 | 6996 | 2433 | 21122 | 15672 | 5449 | 74.20 | 65.21 | 76.59 | 70.45 | 72.52 | 64.09 | 75.47 | 69.33 |
| XML_COMPACT | opt | 10930 | 8462 | 2468 | 11498 | 8902 | 2596 | 22428 | 17364 | 5064 | 77.42 | 70.04 | 67.46 | 69.10 | 76.14 | 69.19 | 66.61 | 68.24 |
| XML_PRETTY | man | 16166 | 12125 | 4042 | 12780 | 9585 | 3195 | 28946 | 21710 | 7237 | 75.00 | 50.03 | 58.86 | 50.02 | 73.78 | 49.22 | 58.04 | 49.21 |
| XML_PRETTY | opt | 15089 | 11763 | 3326 | 10028 | 7818 | 2210 | 25117 | 19581 | 5536 | 77.96 | 55.79 | 75.83 | 62.25 | 75.39 | 54.07 | 74.12 | 60.54 |
| YAML | man | 12554 | 9213 | 3341 | 10585 | 7768 | 2817 | 23139 | 16982 | 6157 | 73.39 | 61.65 | 69.75 | 64.51 | 71.26 | 60.23 | 68.33 | 63.09 |
| YAML | opt | 11771 | 8954 | 2817 | 10509 | 7994 | 2515 | 22280 | 16949 | 5332 | 76.07 | 66.19 | 71.95 | 68.59 | 75.66 | 65.91 | 71.68 | 68.32 |

#### 2.7.2 Read Tokens Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6989 | 6700 | -289 | -4.14 | 4321 | 4034 | -287 | -6.63 | 2668 | 2666 | -2 | -0.09 | 61.83 | 60.22 | -1.61 | -2.60 | 73.50 | 73.44 | -0.06 | -0.08 | 60.79 | 59.45 | -1.34 | -2.20 | 72.80 | 72.92 |  +0.12 |  +0.17 |
| JSON_COMPACT | 9268 | 8748 | -520 | -5.61 | 6652 | 6679 |  +27 |  +0.40 | 2616 | 2069 | -547 | -20.89 | 71.77 | 76.34 |  +4.57 |  +6.37 | 72.11 | 76.99 |  +4.87 |  +6.76 | 69.79 | 76.30 |  +6.51 |  +9.33 | 70.79 | 76.96 |  +6.17 |  +8.71 |
| JSON_PRETTY | 14283 | 13367 | -916 | -6.41 | 11326 | 9953 | -1373 | -12.13 | 2957 | 3414 |  +457 |  +15.47 | 79.30 | 74.46 | -4.84 | -6.10 | 59.51 | 59.51 | -0.01 | -0.01 | 77.39 | 73.38 | -4.01 | -5.18 | 58.24 | 58.78 |  +0.55 |  +0.94 |
| TOON_DEFAULT | 7048 | 11561 |  +4513 |  +64.03 | 5409 | 8965 |  +3556 |  +65.75 | 1639 | 2596 |  +957 |  +58.38 | 76.75 | 77.55 |  +0.80 |  +1.04 | 83.23 | 67.91 | -15.32 | -18.41 | 74.09 | 77.12 |  +3.03 |  +4.09 | 81.46 | 67.62 | -13.84 | -16.99 |
| XML_COMPACT | 11693 | 10930 | -763 | -6.53 | 8676 | 8462 | -214 | -2.47 | 3017 | 2468 | -549 | -18.19 | 74.20 | 77.42 |  +3.22 |  +4.34 | 65.21 | 70.04 |  +4.83 |  +7.40 | 72.52 | 76.14 |  +3.62 |  +4.99 | 64.09 | 69.19 |  +5.09 |  +7.95 |
| XML_PRETTY | 16166 | 15089 | -1077 | -6.66 | 12125 | 11764 | -361 | -2.98 | 4042 | 3326 | -716 | -17.71 | 75.00 | 77.96 |  +2.96 |  +3.95 | 50.03 | 55.79 |  +5.76 |  +11.51 | 73.78 | 75.39 |  +1.61 |  +2.18 | 49.22 | 54.07 |  +4.86 |  +9.87 |
| YAML | 12554 | 11771 | -783 | -6.24 | 9213 | 8954 | -259 | -2.81 | 3341 | 2817 | -524 | -15.68 | 73.39 | 76.07 |  +2.68 |  +3.65 | 61.65 | 66.19 |  +4.54 |  +7.36 | 71.26 | 75.66 |  +4.40 |  +6.17 | 60.23 | 65.91 |  +5.68 |  +9.44 |

#### 2.7.3 Output Tokens Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 9536 | 12844 |  +3308 |  +34.69 | 5896 | 7735 |  +1839 |  +31.19 | 3640 | 5110 |  +1470 |  +40.37 | 61.83 | 60.22 | -1.61 | -2.60 | 67.76 | 48.65 | -19.11 | -28.20 | 60.79 | 59.45 | -1.34 | -2.20 | 67.07 | 48.14 | -18.93 | -28.22 |
| JSON_COMPACT | 13161 | 10006 | -3155 | -23.97 | 9445 | 7638 | -1807 | -19.14 | 3715 | 2367 | -1348 | -36.29 | 71.77 | 76.34 |  +4.57 |  +6.37 | 54.63 | 74.88 |  +20.25 |  +37.06 | 69.79 | 76.30 |  +6.51 |  +9.33 | 53.31 | 74.85 |  +21.54 |  +40.40 |
| JSON_PRETTY | 13138 | 8302 | -4836 | -36.81 | 10419 | 6182 | -4237 | -40.67 | 2720 | 2121 | -599 | -22.03 | 79.30 | 74.46 | -4.84 | -6.10 | 59.77 | 82.91 |  +23.14 |  +38.71 | 77.39 | 73.38 | -4.01 | -5.18 | 58.50 | 82.19 |  +23.69 |  +40.50 |
| TOON_DEFAULT | 11671 | 11768 |  +97 |  +0.83 | 8957 | 9125 |  +168 |  +1.88 | 2713 | 2641 | -72 | -2.64 | 76.75 | 77.55 |  +0.80 |  +1.04 | 66.07 | 66.08 |  +0.01 |  +0.01 | 74.09 | 77.12 |  +3.03 |  +4.09 | 64.30 | 65.79 |  +1.49 |  +2.32 |
| XML_COMPACT | 9429 | 11499 |  +2070 |  +21.95 | 6996 | 8902 |  +1906 |  +27.24 | 2433 | 2597 |  +164 |  +6.73 | 74.20 | 77.42 |  +3.22 |  +4.34 | 76.59 | 67.46 | -9.14 | -11.93 | 72.52 | 76.14 |  +3.62 |  +4.99 | 75.47 | 66.61 | -8.87 | -11.75 |
| XML_PRETTY | 12780 | 10028 | -2752 | -21.53 | 9585 | 7818 | -1767 | -18.44 | 3195 | 2210 | -985 | -30.82 | 75.00 | 77.96 |  +2.96 |  +3.95 | 58.86 | 75.83 |  +16.98 |  +28.84 | 73.78 | 75.39 |  +1.61 |  +2.18 | 58.04 | 74.12 |  +16.08 |  +27.69 |
| YAML | 10585 | 10509 | -76 | -0.71 | 7768 | 7994 |  +226 |  +2.91 | 2817 | 2515 | -302 | -10.71 | 73.39 | 76.07 |  +2.68 |  +3.65 | 69.75 | 71.95 |  +2.20 |  +3.15 | 71.26 | 75.66 |  +4.40 |  +6.17 | 68.33 | 71.68 |  +3.35 |  +4.90 |

#### 2.7.4 Total Tokens Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 16525 | 19544 |  +3019 |  +18.27 | 10218 | 11770 |  +1552 |  +15.19 | 6308 | 7775 |  +1467 |  +23.26 | 61.83 | 60.22 | -1.61 | -2.60 | 74.52 | 65.36 | -9.16 | -12.29 | 60.79 | 59.45 | -1.34 | -2.20 | 73.83 | 64.84 | -8.98 | -12.17 |
| JSON_COMPACT | 22429 | 18754 | -3675 | -16.39 | 16097 | 14316 | -1781 | -11.06 | 6332 | 4437 | -1895 | -29.92 | 71.77 | 76.34 |  +4.57 |  +6.37 | 65.33 | 78.22 |  +12.89 |  +19.74 | 69.79 | 76.30 |  +6.51 |  +9.33 | 64.01 | 78.20 |  +14.19 |  +22.16 |
| JSON_PRETTY | 27421 | 21669 | -5752 | -20.98 | 21745 | 16135 | -5610 | -25.80 | 5676 | 5534 | -142 | -2.50 | 79.30 | 74.46 | -4.84 | -6.10 | 56.97 | 69.16 |  +12.18 |  +21.39 | 77.39 | 73.38 | -4.01 | -5.18 | 55.70 | 68.44 |  +12.74 |  +22.87 |
| TOON_DEFAULT | 18719 | 23329 |  +4610 |  +24.63 | 14367 | 18092 |  +3725 |  +25.92 | 4352 | 5237 |  +885 |  +20.34 | 76.75 | 77.55 |  +0.80 |  +1.04 | 78.59 | 66.77 | -11.82 | -15.04 | 74.09 | 77.12 |  +3.03 |  +4.09 | 76.81 | 66.49 | -10.33 | -13.45 |
| XML_COMPACT | 21122 | 22429 |  +1307 |  +6.19 | 15672 | 17364 |  +1692 |  +10.79 | 5449 | 5064 | -385 | -7.07 | 74.20 | 77.42 |  +3.22 |  +4.34 | 70.45 | 69.10 | -1.35 | -1.92 | 72.52 | 76.14 |  +3.62 |  +4.99 | 69.33 | 68.24 | -1.09 | -1.57 |
| XML_PRETTY | 28946 | 25117 | -3829 | -13.23 | 21710 | 19582 | -2128 | -9.80 | 7237 | 5536 | -1701 | -23.50 | 75.00 | 77.96 |  +2.96 |  +3.95 | 50.02 | 62.25 |  +12.23 |  +24.45 | 73.78 | 75.39 |  +1.61 |  +2.18 | 49.21 | 60.54 |  +11.33 |  +23.03 |
| YAML | 23139 | 22280 | -859 | -3.71 | 16982 | 16949 | -33 | -0.19 | 6157 | 5331 | -826 | -13.41 | 73.39 | 76.07 |  +2.68 |  +3.65 | 64.51 | 68.59 |  +4.09 |  +6.34 | 71.26 | 75.66 |  +4.40 |  +6.17 | 63.09 | 68.32 |  +5.23 |  +8.29 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Accuracy by Char (%) |
|---|---|---|---|---|---|---|
| CSV | man | 76.67 | 47.33 | 0.00 | 61.83 | 84.25 |
| CSV | opt | 74.67 | 49.33 | 0.00 | 60.22 | 86.02 |
| JSON_COMPACT | man | 89.00 | 35.00 | 0.00 | 71.77 | 91.26 |
| JSON_COMPACT | opt | 94.67 | 29.33 | 0.00 | 76.34 | 91.25 |
| JSON_PRETTY | man | 98.33 | 25.67 | 0.00 | 79.30 | 93.58 |
| JSON_PRETTY | opt | 92.33 | 31.67 | 0.00 | 74.46 | 90.93 |
| TOON_DEFAULT | man | 95.17 | 28.83 | 0.00 | 76.75 | 93.22 |
| TOON_DEFAULT | opt | 96.17 | 27.83 | 0.00 | 77.55 | 91.54 |
| XML_COMPACT | man | 92.00 | 32.00 | 0.00 | 74.20 | 92.40 |
| XML_COMPACT | opt | 96.00 | 28.00 | 0.00 | 77.42 | 92.19 |
| XML_PRETTY | man | 93.00 | 31.00 | 0.00 | 75.00 | 92.36 |
| XML_PRETTY | opt | 96.67 | 27.33 | 0.00 | 77.96 | 92.40 |
| YAML | man | 91.00 | 33.00 | 0.00 | 73.39 | 92.06 |
| YAML | opt | 94.33 | 29.67 | 0.00 | 76.07 | 91.23 |

#### 2.8.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Accuracy by Char (%) Man | Accuracy by Char (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 76.67 | 74.67 | -2 | -2.61 | 47.33 | 49.33 |  +2 |  +4.23 | 0.00 | 0.00 | 0 | 0.00 | 61.83 | 60.22 | -1.61 | 84.25 | 86.02 |  +84.25 |
| JSON_COMPACT | 89.00 | 94.67 |  +6 |  +6.37 | 35.00 | 29.33 | -6 | -16.20 | 0.00 | 0.00 | 0 | 0.00 | 71.77 | 76.34 |  +4.57 | 91.26 | 91.25 |  +91.26 |
| JSON_PRETTY | 98.33 | 92.33 | -6 | -6.10 | 25.67 | 31.67 |  +6 |  +23.37 | 0.00 | 0.00 | 0 | 0.00 | 79.30 | 74.46 | -4.84 | 93.58 | 90.93 |  +93.58 |
| TOON_DEFAULT | 95.17 | 96.17 |  +1 |  +1.05 | 28.83 | 27.83 | -1 | -3.47 | 0.00 | 0.00 | 0 | 0.00 | 76.75 | 77.55 |  +0.80 | 93.22 | 91.54 |  +93.22 |
| XML_COMPACT | 92.00 | 96.00 |  +4 |  +4.35 | 32.00 | 28.00 | -4 | -12.50 | 0.00 | 0.00 | 0 | 0.00 | 74.20 | 77.42 |  +3.22 | 92.40 | 92.19 |  +92.40 |
| XML_PRETTY | 93.00 | 96.67 |  +4 |  +3.95 | 31.00 | 27.33 | -4 | -11.84 | 0.00 | 0.00 | 0 | 0.00 | 75.00 | 77.96 |  +2.96 | 92.36 | 92.40 |  +92.36 |
| YAML | 91.00 | 94.33 |  +3 |  +3.66 | 33.00 | 29.67 | -3 | -10.09 | 0.00 | 0.00 | 0 | 0.00 | 73.39 | 76.07 |  +2.68 | 92.06 | 91.23 |  +92.06 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| CSV | man | 61.83 | 73.94 | 55.56 | 52.38 | 47.62 |
| CSV | opt | 60.22 | 75.15 | 46.91 | 63.49 | 34.92 |
| JSON_COMPACT | man | 71.77 | 97.58 | 55.55 | 58.73 | 38.10 |
| JSON_COMPACT | opt | 76.34 | 90.30 | 74.07 | 71.43 | 47.62 |
| JSON_PRETTY | man | 79.30 | 98.79 | 66.67 | 65.08 | 58.73 |
| JSON_PRETTY | opt | 74.46 | 95.15 | 66.67 | 61.90 | 42.86 |
| TOON_DEFAULT | man | 76.75 | 94.55 | 57.41 | 65.08 | 66.67 |
| TOON_DEFAULT | opt | 77.55 | 93.94 | 72.22 | 71.43 | 47.62 |
| XML_COMPACT | man | 74.20 | 92.12 | 61.73 | 63.49 | 53.97 |
| XML_COMPACT | opt | 77.42 | 98.79 | 66.67 | 66.67 | 46.03 |
| XML_PRETTY | man | 75.00 | 97.57 | 61.73 | 68.26 | 39.68 |
| XML_PRETTY | opt | 77.96 | 98.18 | 59.26 | 65.08 | 61.90 |
| YAML | man | 73.39 | 98.79 | 48.15 | 73.02 | 39.68 |
| YAML | opt | 76.07 | 97.58 | 69.14 | 69.84 | 34.92 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 73.94 | 75.15 |  +1.21 |
| JSON_COMPACT | 97.58 | 90.30 | -7.27 |
| JSON_PRETTY | 98.79 | 95.15 | -3.64 |
| TOON_DEFAULT | 94.55 | 93.94 | -0.61 |
| XML_COMPACT | 92.12 | 98.79 |  +6.67 |
| XML_PRETTY | 97.57 | 98.18 |  +0.61 |
| YAML | 98.79 | 97.58 | -1.21 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 55.56 | 46.91 | -8.64 |
| JSON_COMPACT | 55.55 | 74.07 |  +18.52 |
| JSON_PRETTY | 66.67 | 66.67 | 0.00 |
| TOON_DEFAULT | 57.41 | 72.22 |  +14.81 |
| XML_COMPACT | 61.73 | 66.67 |  +4.94 |
| XML_PRETTY | 61.73 | 59.26 | -2.47 |
| YAML | 48.15 | 69.14 |  +20.99 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 52.38 | 63.49 |  +11.11 |
| JSON_COMPACT | 58.73 | 71.43 |  +12.70 |
| JSON_PRETTY | 65.08 | 61.90 | -3.17 |
| TOON_DEFAULT | 65.08 | 71.43 |  +6.35 |
| XML_COMPACT | 63.49 | 66.67 |  +3.18 |
| XML_PRETTY | 68.26 | 65.08 | -3.18 |
| YAML | 73.02 | 69.84 | -3.18 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 47.62 | 34.92 | -12.70 |
| JSON_COMPACT | 38.10 | 47.62 |  +9.52 |
| JSON_PRETTY | 58.73 | 42.86 | -15.87 |
| TOON_DEFAULT | 66.67 | 47.62 | -19.05 |
| XML_COMPACT | 53.97 | 46.03 | -7.93 |
| XML_PRETTY | 39.68 | 61.90 |  +22.22 |
| YAML | 39.68 | 34.92 | -4.76 |

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
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_off\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_on_verify\BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)