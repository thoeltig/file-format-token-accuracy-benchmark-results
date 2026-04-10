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

1. **JSON_COMPACT** delivers the best overall efficiency by combining the lowest total token cost (18,250 opt / 18,446 man) with competitive accuracy (~76–79%) which results in the highest total efficiency score of 85.29 (optional). No other format comes close to this balance of cost and quality.

2. **YAML** achieves the highest accuracy across both variants (80.64% opt / 78.76% man) and dominates field retrieval at 97–99% but this comes at the cost of the highest output token consumption (12,896 opt / 11,104 man) which drags its total efficiency score down to 62–65.

3. Pretty-printed formats consistently underperform their compact counterparts in both token efficiency and accuracy. **XML_PRETTY** ranks last in total efficiency (51.58 opt / 55.88 man) and wastes the most read tokens (5,160–5,352) while **JSON_PRETTY** shows the lowest accuracy for optional data (72.85%). The added whitespace increases token cost without helping the model understand the data better.

4. Aggregation is the weakest category for all formats with accuracy ranging from 43% to 71% which suggests that Haiku 4.5 struggles with numerical computation regardless of how the data is presented. This contrasts sharply with field retrieval where most formats score above 85%.

5. **TOON_DEFAULT** exhibits the most unstable behavior between data variants. It shows the largest accuracy swing (+5.78%) alongside a dramatic output token increase of +40% when switching from mandatory to optional data. This makes it the least predictable format for production workloads despite achieving the second-highest optional accuracy (80.24%).

6. **XML_COMPACT** offers the densest character packing at ~2.52 characters per read token which makes it the most structurally efficient format for encoding information. Combined with the lowest output tokens (7,356 opt) and a respectable total efficiency score of 78.85 it represents a strong middle-ground choice between **JSON_COMPACT** and **YAML**.

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
- **Accuracy**: Correct answers / total questions
- **Weighted Accuracy**: Accuracy weighted by question category importanc

#### 1.3.3 Efficiency Score
Composite metric balancing accuracy with normalized token count (favour towards accuracy). Each efficieny score has an indicator which token count was used in the calculation.
- **Normalized Tokens** = (((**Max Tokens** + 10) - **Curren Tokens**) / ((**Max Tokens** + 10) - (**Min Tokens** - 10))) * 100
- **Efficiency Score**: (**Accuracy** % x 0.7) + (**Normalized Tokens** * 0.3)
- **Weighted Efficiency Score**: (**Weighted Accuracy** % x 0.7) + (**Normalized Tokens** * 0.3)

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
   - Optional: YAML 80.64%
   - Mandatory: YAML 78.76%
- Lowest accuracy drift:
   - Optional: XML_COMPACT ↓ -1.79% ↑ 3.56%
   - Mandatory: YAML ↓ -2.73% ↑ 2.40%
- Most useful read tokens:
   - Optional: XML_PRETTY 14423 / 19583 tokens
   - Mandatory: XML_PRETTY 14762 / 20114 tokens
- Most useful output tokens:
   - Optional: YAML 10400 / 12896 tokens
   - Mandatory: YAML 8746 / 11104 tokens
- Highest read efficiency (%/token):
   - Optional: JSON_COMPACT 85.29
   - Mandatory: JSON_COMPACT 81.51
- Highest output efficiency (%/token):
   - Optional: XML_COMPACT 75.37
   - Mandatory: XML_PRETTY 51.40
- Lowest delta (optional-mandatory):
   - Read tokens: TOON_DEFAULT -237 tokens
   - Output tokens: JSON_COMPACT 331 tokens
   - Accuracy: XML_PRETTY 0.26%
   - Read efficiency: JSON_PRETTY -0.13
   - Output efficiency: JSON_COMPACT 0.97

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
   - Optional: JSON_PRETTY 72.85%
   - Mandatory: XML_PRETTY 73.39%
- Highest accuracy drift:
   - Optional: JSON_PRETTY ↓ -11.43% ↑ 10.71%
   - Mandatory: XML_PRETTY ↓ -13.19% ↑ 15.38%
- Most wasted read tokens:
   - Optional: XML_PRETTY 5160 / 19583 tokens
   - Mandatory: XML_PRETTY 5352 / 20114 tokens
- Most wasted output tokens:
   - Optional: JSON_PRETTY 2622 / 9659 tokens
   - Mandatory: JSON_PRETTY 2490 / 10771 tokens
- Lowest read efficiency (%/token):
   - Optional: XML_PRETTY 53.12
   - Mandatory: XML_PRETTY 51.40
- Lowest output efficiency (%/token):
   - Optional: YAML 64.88
   - Mandatory: JSON_PRETTY 70.51
- Highest delta (optional-mandatory):
   - Read tokens: JSON_PRETTY -929 tokens
   - Output tokens: TOON_DEFAULT 3506 tokens
   - Accuracy: TOON_DEFAULT 5.78%
   - Read efficiency: TOON_DEFAULT 4.73
   - Output efficiency: TOON_DEFAULT -9.58

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT ≈ 69s | JSON_COMPACT ≈ 10315 | JSON_COMPACT ≈ 336 | XML_PRETTY ≈ 7175 | XML_PRETTY ≈ 7518 | JSON_COMPACT ≈ 18446 | YAML ≈ 79% | JSON_COMPACT ≈ 82 | XML_PRETTY ≈ 81 | JSON_COMPACT ≈ 83 |
| XML_PRETTY (+2.2%) | XML_COMPACT (+24.6%) | YAML (+0.5%) | JSON_COMPACT (+8.6%) | JSON_COMPACT (+8.2%) | XML_COMPACT (+18.3%) | JSON_PRETTY (-1.9%) | XML_COMPACT (-10.2%) | JSON_COMPACT (-0.9%) | XML_COMPACT (-12.2%) |
| TOON_DEFAULT (+7.0%) | TOON_DEFAULT (+36.7%) | TOON_DEFAULT (+1.7%) | TOON_DEFAULT (+16.3%) | TOON_DEFAULT (+15.5%) | TOON_DEFAULT (+23.5%) | JSON_COMPACT (-3.0%) | YAML (-11.7%) | TOON_DEFAULT (-4.7%) | TOON_DEFAULT (-15.4%) |
| XML_COMPACT (+8.0%) | YAML (+38.7%) | XML_PRETTY (+1.9%) | XML_COMPACT (+20.2%) | XML_COMPACT (+19.3%) | YAML (+37.8%) | TOON_DEFAULT (-4.3%) | TOON_DEFAULT (-14.6%) | XML_COMPACT (-6.0%) | YAML (-20.4%) |
| YAML (+31.8%) | JSON_PRETTY (+72.8%) | XML_COMPACT (+2.6%) | JSON_PRETTY (+43.9%) | JSON_PRETTY (+43.3%) | XML_PRETTY (+49.8%) | XML_COMPACT (-4.3%) | JSON_PRETTY (-25.8%) | YAML (-12.6%) | XML_PRETTY (-32.3%) |
| JSON_PRETTY (+34.9%) | XML_PRETTY (+95.0%) | JSON_PRETTY (+33.0%) | YAML (+50.1%) | YAML (+47.7%) | JSON_PRETTY (+55.0%) | XML_PRETTY (-5.4%) | XML_PRETTY (-36.9%) | JSON_PRETTY (-12.6%) | JSON_PRETTY (-32.5%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|
| XML_COMPACT ≈ 64s | JSON_COMPACT ≈ 9788 | JSON_PRETTY ≈ 228 | XML_COMPACT ≈ 7011 | XML_COMPACT ≈ 7356 | JSON_COMPACT ≈ 18250 | YAML ≈ 81% | JSON_COMPACT ≈ 85 | XML_COMPACT ≈ 83 | JSON_COMPACT ≈ 85 |
| XML_PRETTY (+21.1%) | XML_COMPACT (+26.4%) | JSON_COMPACT (+47.4%) | JSON_COMPACT (+15.9%) | JSON_COMPACT (+15.0%) | XML_COMPACT (+8.1%) | TOON_DEFAULT (-0.4%) | XML_COMPACT (-11.6%) | JSON_COMPACT (-2.2%) | XML_COMPACT (-7.6%) |
| JSON_PRETTY (+31.7%) | TOON_DEFAULT (+41.6%) | YAML (+47.6%) | XML_PRETTY (+33.4%) | JSON_PRETTY (+31.3%) | TOON_DEFAULT (+42.7%) | JSON_COMPACT (-1.6%) | TOON_DEFAULT (-12.8%) | XML_PRETTY (-12.6%) | TOON_DEFAULT (-23.8%) |
| JSON_COMPACT (+37.5%) | YAML (+43.6%) | XML_PRETTY (+51.1%) | JSON_PRETTY (+34.5%) | XML_PRETTY (+31.9%) | JSON_PRETTY (+45.5%) | XML_COMPACT (-5.1%) | YAML (-13.2%) | JSON_PRETTY (-13.1%) | YAML (-26.4%) |
| YAML (+47.7%) | JSON_PRETTY (+72.7%) | XML_COMPACT (+51.2%) | TOON_DEFAULT (+68.9%) | TOON_DEFAULT (+65.7%) | YAML (+47.7%) | XML_PRETTY (-7.0%) | JSON_PRETTY (-29.2%) | TOON_DEFAULT (-18.7%) | JSON_PRETTY (-31.5%) |
| TOON_DEFAULT (+51.2%) | XML_PRETTY (+100.1%) | TOON_DEFAULT (+54.5%) | YAML (+79.1%) | YAML (+75.3%) | XML_PRETTY (+60.5%) | JSON_PRETTY (-7.8%) | XML_PRETTY (-37.7%) | YAML (-21.7%) | XML_PRETTY (-39.5%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 98% | JSON_PRETTY ≈ 72% | JSON_COMPACT ≈ 67% | JSON_COMPACT ≈ 71% |
| JSON_PRETTY (-1.2%) | XML_PRETTY (-2.5%) | XML_COMPACT (0.0%) | TOON_DEFAULT (-5.6%) |
| TOON_DEFAULT (-6.4%) | YAML (-2.5%) | JSON_PRETTY (-6.4%) | YAML (-11.1%) |
| XML_PRETTY (-9.1%) | XML_COMPACT (-3.7%) | YAML (-6.4%) | XML_PRETTY (-12.7%) |
| XML_COMPACT (-9.7%) | JSON_COMPACT (-3.7%) | TOON_DEFAULT (-7.1%) | XML_COMPACT (-15.9%) |
| JSON_COMPACT (-12.7%) | TOON_DEFAULT (-13.0%) | XML_PRETTY (-12.7%) | JSON_PRETTY (-22.2%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99% | JSON_COMPACT ≈ 83% | TOON_DEFAULT ≈ 71% | YAML ≈ 56% |
| TOON_DEFAULT (-3.0%) | TOON_DEFAULT (-0.0%) | XML_PRETTY (-7.9%) | XML_COMPACT (-0.0%) |
| JSON_COMPACT (-5.5%) | XML_COMPACT (-6.2%) | YAML (-7.9%) | JSON_COMPACT (-3.2%) |
| JSON_PRETTY (-9.1%) | YAML (-7.4%) | XML_COMPACT (-7.9%) | XML_PRETTY (-3.2%) |
| XML_PRETTY (-10.3%) | XML_PRETTY (-16.0%) | JSON_COMPACT (-9.5%) | JSON_PRETTY (-9.5%) |
| XML_COMPACT (-12.1%) | JSON_PRETTY (-16.1%) | JSON_PRETTY (-9.5%) | TOON_DEFAULT (-11.9%) |


#### 2.1.5 Conclusion

The benchmark reveals a clear trade-off between token cost and accuracy.

**JSON_COMPACT** dominates the efficiency rankings because it consumes 35–38% fewer total tokens than the next cheapest format while maintaining accuracy in the 76–79% range. **XML_COMPACT** occupies a similar niche with slightly higher read tokens but the lowest output token cost of any format which makes it particularly attractive when output tokens are more expensive. Both compact formats benefit from high character density per read token (2.2–2.5), meaning the tokenizer encodes their syntax more efficiently than verbose alternatives.

**YAML** consistently tops the accuracy rankings across both variants and excels at field retrieval (97–99%) which is the highest-weighted category. However it pays for this accuracy with the highest output token consumption. The model appears to "think harder" when processing **YAML** which produces more output tokens to arrive at more correct answers. Whether this trade-off is worthwhile depends on whether accuracy or token cost matters more for a given use case.

The pretty-printed formats add significant token overhead through whitespace and indentation without improving accuracy. **JSON_PRETTY** consumes 45–55% more total tokens than **JSON_COMPACT** while actually scoring lower on accuracy for optional data. **XML_PRETTY** is the worst performer overall which positions it last in total efficiency for both variants. **TOON_DEFAULT** is harder to categorize because it achieves strong optional accuracy (80.24%) but shows the most erratic behavior between variants with output tokens jumping 40% from mandatory to optional data.

**YAML** dominates field retrieval across all conditions. **JSON_COMPACT** and **TOON_DEFAULT** lead structure awareness for optional data. Filtering results are mixed across formats with no clear winner. Aggregation is universally weak (43–71%) which indicates that Haiku 4.5's ability to perform numerical calculations is limited regardless of input format. This suggests that aggregation accuracy is more of a model capability constraint than a format readability issue.

Robustness analysis shows that most formats handle optional fields gracefully and read tokens decrease slightly as expected due to fewer populated fields. The notable exception is **TOON_DEFAULT** where output tokens spike dramatically (+40%) for optional data. **JSON_COMPACT** is the most robust format overall with total token variance of just 1% between variants.

For Haiku 4.5 with nested data and thinking enabled **JSON_COMPACT** is the recommended default format. It offers the best efficiency score, the most predictable token usage and accuracy that falls within 2 percentage points of **YAML**. If maximum accuracy is critical and token budget allows it then **YAML** is the better choice. **XML_COMPACT** serves as a viable alternative when output token cost is the primary concern. Pretty-printed formats should be avoided for machine consumption as they provide no accuracy benefit while substantially increasing cost.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10315 | 8131 | 18446 | 2.213 | 62.866 | 75.81 | 7819.802 | 2495.199 | 6164.364 | 1966.969 | 81.51 | 80.01 | 82.51 |
| JSON_COMPACT | opt | 9788 | 8462 | 18250 | 2.186 | 65.535 | 79.03 | 7735.456 | 2052.544 | 6687.519 | 1774.481 | 85.29 | 80.98 | 85.29 |
| JSON_PRETTY | man | 17828 | 10771 | 28599 | 1.762 | 83.255 | 76.88 | 13706.166 | 4121.834 | 8280.489 | 2490.178 | 60.47 | 70.51 | 55.70 |
| JSON_PRETTY | opt | 16899 | 9659 | 26558 | 1.752 | 76.059 | 72.85 | 12310.921 | 4588.079 | 7036.581 | 2622.419 | 60.35 | 72.01 | 58.42 |
| TOON_DEFAULT | man | 14096 | 8685 | 22781 | 1.851 | 67.281 | 74.46 | 10495.882 | 3600.118 | 6466.603 | 2218.064 | 69.60 | 76.92 | 69.80 |
| TOON_DEFAULT | opt | 13859 | 12191 | 26050 | 1.860 | 95.478 | 80.24 | 11120.462 | 2738.538 | 9782.126 | 2408.958 | 74.33 | 67.34 | 64.97 |
| XML_COMPACT | man | 12848 | 8966 | 21814 | 2.522 | 69.530 | 74.46 | 9566.621 | 3281.379 | 6676.332 | 2290.001 | 73.22 | 75.83 | 72.42 |
| XML_COMPACT | opt | 12368 | 7356 | 19724 | 2.517 | 56.543 | 75.54 | 9342.787 | 3025.213 | 5556.471 | 1799.196 | 75.37 | 82.84 | 78.85 |
| XML_PRETTY | man | 20114 | 7518 | 27632 | 1.993 | 57.866 | 73.39 | 14761.665 | 5352.335 | 5517.216 | 2000.451 | 51.40 | 80.70 | 55.88 |
| XML_PRETTY | opt | 19583 | 9701 | 29284 | 1.982 | 75.456 | 73.65 | 14422.880 | 5160.120 | 7144.418 | 2556.082 | 53.12 | 72.41 | 51.58 |
| YAML | man | 14306 | 11104 | 25410 | 1.789 | 86.828 | 78.76 | 11267.406 | 3038.594 | 8745.773 | 2358.560 | 72.00 | 70.53 | 65.67 |
| YAML | opt | 14053 | 12896 | 26949 | 1.799 | 101.293 | 80.64 | 11332.339 | 2720.661 | 10399.603 | 2496.730 | 74.05 | 64.88 | 62.81 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10315 | 9788 | -527 | -5.11 | 336 | 336 |  +331 |  +98.51 | 7795 | 8126 |  +331 |  +4.25 | 8131 | 8462 |  +331 |  +4.07 | 18446 | 18250 | -196 | -1.06 |
| JSON_PRETTY | 17828 | 16899 | -929 | -5.21 | 447 | 228 | -892 | -199.55 | 10324 | 9432 | -1112 | -10.77 | 10771 | 9659 | -1112 | -10.32 | 28599 | 26558 | -2041 | -7.14 |
| TOON_DEFAULT | 14096 | 13859 | -237 | -1.68 | 342 | 352 |  +3497 |  +1022.51 | 8343 | 11840 |  +3506 |  +42.02 | 8685 | 12191 |  +3506 |  +40.37 | 22781 | 26050 |  +3269 |  +14.35 |
| XML_COMPACT | 12848 | 12368 | -480 | -3.74 | 345 | 345 | -1610 | -466.67 | 8622 | 7012 | -1611 | -18.68 | 8966 | 7355 | -1611 | -17.97 | 21814 | 19723 | -2091 | -9.59 |
| XML_PRETTY | 20114 | 19583 | -531 | -2.64 | 342 | 344 |  +2181 |  +637.72 | 7175 | 9356 |  +2183 |  +30.43 | 7518 | 9701 |  +2183 |  +29.04 | 27632 | 29284 |  +1652 |  +5.98 |
| YAML | 14306 | 14053 | -253 | -1.77 | 338 | 336 |  +1794 |  +530.77 | 10767 | 12561 |  +1792 |  +16.64 | 11104 | 12896 |  +1792 |  +16.14 | 25410 | 26949 |  +1539 |  +6.06 |

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
| JSON_COMPACT | man | 2.213 | 15.125 | 332.742 | 0.735 | 0.932 | 0.411 |
| JSON_COMPACT | opt | 2.186 | 15.512 | 315.742 | 0.807 | 0.934 | 0.433 |
| JSON_PRETTY | man | 1.762 | 26.141 | 575.097 | 0.431 | 0.714 | 0.269 |
| JSON_PRETTY | opt | 1.752 | 26.781 | 545.129 | 0.431 | 0.754 | 0.274 |
| TOON_DEFAULT | man | 1.851 | 20.669 | 454.710 | 0.528 | 0.859 | 0.327 |
| TOON_DEFAULT | opt | 1.860 | 21.964 | 447.065 | 0.579 | 0.697 | 0.311 |
| XML_COMPACT | man | 2.522 | 18.839 | 414.452 | 0.580 | 0.830 | 0.341 |
| XML_COMPACT | opt | 2.517 | 19.601 | 398.968 | 0.611 | 1.027 | 0.383 |
| XML_PRETTY | man | 1.993 | 29.493 | 648.839 | 0.365 | 0.976 | 0.266 |
| XML_PRETTY | opt | 1.982 | 31.035 | 631.710 | 0.376 | 0.759 | 0.252 |
| YAML | man | 1.789 | 20.977 | 461.484 | 0.551 | 0.709 | 0.310 |
| YAML | opt | 1.799 | 22.271 | 453.323 | 0.574 | 0.625 | 0.299 |

#### 2.5.2 Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 2.213 | 2.186 | -0.027 | -1.22 | 15.125 | 15.512 |  +0.387 |  +2.56 | 332.742 | 315.742 | -17.000 | -5.11 | 0.735 | 0.807 |  +0.072 |  +9.80 | 0.932 | 0.934 |  +0.002 |  +0.21 | 0.411 | 0.433 |  +0.022 |  +5.35 |
| JSON_PRETTY | 1.762 | 1.752 | -0.010 | -0.57 | 26.141 | 26.781 |  +0.640 |  +2.45 | 575.097 | 545.129 | -29.968 | -5.21 | 0.431 | 0.431 | 0.000 | 0.00 | 0.714 | 0.754 |  +0.040 |  +5.60 | 0.269 | 0.274 |  +0.005 |  +1.86 |
| TOON_DEFAULT | 1.851 | 1.860 |  +0.009 |  +0.49 | 20.669 | 21.964 |  +1.295 |  +6.27 | 454.710 | 447.065 | -7.645 | -1.68 | 0.528 | 0.579 |  +0.051 |  +9.66 | 0.859 | 0.697 | -0.162 | -18.87 | 0.327 | 0.311 | -0.016 | -4.74 |
| XML_COMPACT | 2.522 | 2.517 | -0.005 | -0.20 | 18.839 | 19.601 |  +0.762 |  +4.04 | 414.452 | 398.968 | -15.484 | -3.74 | 0.580 | 0.611 |  +0.031 |  +5.34 | 0.830 | 1.027 |  +0.197 |  +23.73 | 0.341 | 0.383 |  +0.042 |  +12.32 |
| XML_PRETTY | 1.993 | 1.982 | -0.011 | -0.55 | 29.493 | 31.035 |  +1.542 |  +5.23 | 648.839 | 631.710 | -17.129 | -2.64 | 0.365 | 0.376 |  +0.011 |  +3.01 | 0.976 | 0.759 | -0.217 | -22.23 | 0.266 | 0.252 | -0.014 | -5.26 |
| YAML | 1.789 | 1.799 |  +0.010 |  +0.56 | 20.977 | 22.271 |  +1.294 |  +6.17 | 461.484 | 453.323 | -8.161 | -1.77 | 0.551 | 0.574 |  +0.023 |  +4.17 | 0.709 | 0.625 | -0.084 | -11.85 | 0.310 | 0.299 | -0.011 | -3.55 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10315 | 7820 | 2495 | 8131 | 6164 | 1967 | 18446 | 13984 | 4462 | 75.81 | 81.51 | 80.01 | 82.51 | 74.44 | 80.55 | 79.06 | 81.55 |
| JSON_COMPACT | opt | 9788 | 7735 | 2053 | 8462 | 6688 | 1774 | 18250 | 14423 | 3827 | 79.03 | 85.29 | 80.98 | 85.29 | 78.80 | 85.13 | 80.82 | 85.13 |
| JSON_PRETTY | man | 17828 | 13706 | 4122 | 10771 | 8280 | 2490 | 28599 | 21987 | 6612 | 76.88 | 60.47 | 70.51 | 55.70 | 75.74 | 59.68 | 69.71 | 54.90 |
| JSON_PRETTY | opt | 16899 | 12311 | 4588 | 9659 | 7037 | 2622 | 26558 | 19348 | 7210 | 72.85 | 60.35 | 72.01 | 58.42 | 71.96 | 59.72 | 71.38 | 57.80 |
| TOON_DEFAULT | man | 14096 | 10496 | 3600 | 8685 | 6467 | 2218 | 22781 | 16962 | 5818 | 74.46 | 69.60 | 76.92 | 69.80 | 71.94 | 67.84 | 75.16 | 68.03 |
| TOON_DEFAULT | opt | 13859 | 11120 | 2739 | 12191 | 9782 | 2409 | 26050 | 20903 | 5147 | 80.24 | 74.33 | 67.34 | 64.97 | 80.60 | 74.59 | 67.59 | 65.22 |
| XML_COMPACT | man | 12848 | 9567 | 3281 | 8966 | 6676 | 2290 | 21814 | 16243 | 5571 | 74.46 | 73.22 | 75.83 | 72.42 | 73.59 | 72.61 | 75.22 | 71.81 |
| XML_COMPACT | opt | 12368 | 9343 | 3025 | 7356 | 5556 | 1799 | 19724 | 14899 | 4824 | 75.54 | 75.37 | 82.84 | 78.85 | 75.23 | 75.15 | 82.62 | 78.63 |
| XML_PRETTY | man | 20114 | 14762 | 5352 | 7518 | 5517 | 2000 | 27632 | 20279 | 7353 | 73.39 | 51.40 | 80.70 | 55.88 | 71.93 | 50.38 | 79.68 | 54.86 |
| XML_PRETTY | opt | 19583 | 14423 | 5160 | 9701 | 7144 | 2556 | 29284 | 21567 | 7716 | 73.65 | 53.12 | 72.41 | 51.58 | 72.63 | 52.41 | 71.69 | 50.87 |
| YAML | man | 14306 | 11267 | 3039 | 11104 | 8746 | 2359 | 25410 | 20013 | 5397 | 78.76 | 72.00 | 70.53 | 65.67 | 76.86 | 70.67 | 69.20 | 64.34 |
| YAML | opt | 14053 | 11332 | 2721 | 12896 | 10400 | 2497 | 26949 | 21732 | 5217 | 80.64 | 74.05 | 64.88 | 62.81 | 79.41 | 73.19 | 64.02 | 61.95 |

#### 2.6.2 Read Tokens Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10315 | 9788 | -527 | -5.11 | 7820 | 7736 | -84 | -1.08 | 2495 | 2052 | -443 | -17.74 | 75.81 | 79.03 |  +3.22 |  +4.25 | 81.51 | 85.29 |  +3.78 |  +4.64 | 74.44 | 78.80 |  +4.36 |  +5.86 | 80.55 | 85.13 |  +4.58 |  +5.69 |
| JSON_PRETTY | 17828 | 16899 | -929 | -5.21 | 13706 | 12311 | -1395 | -10.18 | 4122 | 4588 |  +466 |  +11.31 | 76.88 | 72.85 | -4.03 | -5.24 | 60.47 | 60.35 | -0.13 | -0.21 | 75.74 | 71.96 | -3.78 | -4.99 | 59.68 | 59.72 |  +0.05 |  +0.08 |
| TOON_DEFAULT | 14096 | 13859 | -237 | -1.68 | 10496 | 11121 |  +625 |  +5.95 | 3600 | 2738 | -862 | -23.93 | 74.46 | 80.24 |  +5.78 |  +7.76 | 69.60 | 74.33 |  +4.73 |  +6.80 | 71.94 | 80.60 |  +8.66 |  +12.04 | 67.84 | 74.59 |  +6.75 |  +9.95 |
| XML_COMPACT | 12848 | 12368 | -480 | -3.74 | 9567 | 9343 | -224 | -2.34 | 3281 | 3025 | -256 | -7.81 | 74.46 | 75.54 |  +1.08 |  +1.45 | 73.22 | 75.37 |  +2.15 |  +2.93 | 73.59 | 75.23 |  +1.64 |  +2.23 | 72.61 | 75.15 |  +2.54 |  +3.50 |
| XML_PRETTY | 20114 | 19583 | -531 | -2.64 | 14762 | 14423 | -339 | -2.29 | 5352 | 5160 | -192 | -3.59 | 73.39 | 73.65 |  +0.26 |  +0.35 | 51.40 | 53.12 |  +1.72 |  +3.35 | 71.93 | 72.63 |  +0.70 |  +0.97 | 50.38 | 52.41 |  +2.03 |  +4.03 |
| YAML | 14306 | 14053 | -253 | -1.77 | 11267 | 11332 |  +65 |  +0.58 | 3039 | 2721 | -318 | -10.46 | 78.76 | 80.64 |  +1.88 |  +2.39 | 72.00 | 74.05 |  +2.05 |  +2.85 | 76.86 | 79.41 |  +2.55 |  +3.32 | 70.67 | 73.19 |  +2.52 |  +3.56 |

#### 2.6.3 Output Tokens Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 8131 | 8462 |  +331 |  +4.07 | 6164 | 6687 |  +523 |  +8.49 | 1967 | 1775 | -192 | -9.79 | 75.81 | 79.03 |  +3.22 |  +4.25 | 80.01 | 80.98 |  +0.97 |  +1.21 | 74.44 | 78.80 |  +4.36 |  +5.86 | 79.06 | 80.82 |  +1.77 |  +2.24 |
| JSON_PRETTY | 10771 | 9659 | -1112 | -10.32 | 8280 | 7036 | -1244 | -15.02 | 2490 | 2622 |  +132 |  +5.31 | 76.88 | 72.85 | -4.03 | -5.24 | 70.51 | 72.01 |  +1.50 |  +2.13 | 75.74 | 71.96 | -3.78 | -4.99 | 69.71 | 71.38 |  +1.67 |  +2.40 |
| TOON_DEFAULT | 8685 | 12191 |  +3506 |  +40.37 | 6467 | 9783 |  +3316 |  +51.27 | 2218 | 2409 |  +191 |  +8.61 | 74.46 | 80.24 |  +5.78 |  +7.76 | 76.92 | 67.34 | -9.58 | -12.45 | 71.94 | 80.60 |  +8.66 |  +12.04 | 75.16 | 67.59 | -7.56 | -10.06 |
| XML_COMPACT | 8966 | 7355 | -1611 | -17.96 | 6676 | 5556 | -1120 | -16.77 | 2290 | 1799 | -491 | -21.43 | 74.46 | 75.54 |  +1.08 |  +1.45 | 75.83 | 82.84 |  +7.01 |  +9.25 | 73.59 | 75.23 |  +1.64 |  +2.23 | 75.22 | 82.62 |  +7.41 |  +9.85 |
| XML_PRETTY | 7518 | 9701 |  +2183 |  +29.03 | 5517 | 7144 |  +1627 |  +29.49 | 2000 | 2556 |  +556 |  +27.78 | 73.39 | 73.65 |  +0.26 |  +0.35 | 80.70 | 72.41 | -8.30 | -10.28 | 71.93 | 72.63 |  +0.70 |  +0.97 | 79.68 | 71.69 | -7.99 | -10.03 |
| YAML | 11104 | 12896 |  +1792 |  +16.14 | 8746 | 10400 |  +1654 |  +18.91 | 2359 | 2497 |  +138 |  +5.86 | 78.76 | 80.64 |  +1.88 |  +2.39 | 70.53 | 64.88 | -5.65 | -8.01 | 76.86 | 79.41 |  +2.55 |  +3.32 | 69.20 | 64.02 | -5.18 | -7.48 |

#### 2.6.4 Total Tokens Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 18446 | 18250 | -196 | -1.06 | 13984 | 14423 |  +439 |  +3.14 | 4462 | 3827 | -635 | -14.23 | 75.81 | 79.03 |  +3.22 |  +4.25 | 82.51 | 85.29 |  +2.79 |  +3.38 | 74.44 | 78.80 |  +4.36 |  +5.86 | 81.55 | 85.13 |  +3.58 |  +4.40 |
| JSON_PRETTY | 28599 | 26558 | -2041 | -7.14 | 21987 | 19348 | -2639 | -12.00 | 6612 | 7210 |  +598 |  +9.05 | 76.88 | 72.85 | -4.03 | -5.24 | 55.70 | 58.42 |  +2.72 |  +4.88 | 75.74 | 71.96 | -3.78 | -4.99 | 54.90 | 57.80 |  +2.89 |  +5.27 |
| TOON_DEFAULT | 22781 | 26050 |  +3269 |  +14.35 | 16962 | 20902 |  +3940 |  +23.23 | 5818 | 5147 | -671 | -11.53 | 74.46 | 80.24 |  +5.78 |  +7.76 | 69.80 | 64.97 | -4.83 | -6.92 | 71.94 | 80.60 |  +8.66 |  +12.04 | 68.03 | 65.22 | -2.81 | -4.13 |
| XML_COMPACT | 21814 | 19723 | -2091 | -9.58 | 16243 | 14899 | -1344 | -8.27 | 5571 | 4824 | -747 | -13.41 | 74.46 | 75.54 |  +1.08 |  +1.45 | 72.42 | 78.85 |  +6.43 |  +8.88 | 73.59 | 75.23 |  +1.64 |  +2.23 | 71.81 | 78.63 |  +6.82 |  +9.50 |
| XML_PRETTY | 27632 | 29284 |  +1652 |  +5.98 | 20279 | 21567 |  +1288 |  +6.35 | 7353 | 7716 |  +363 |  +4.94 | 73.39 | 73.65 |  +0.26 |  +0.35 | 55.88 | 51.58 | -4.30 | -7.70 | 71.93 | 72.63 |  +0.70 |  +0.97 | 54.86 | 50.87 | -3.99 | -7.28 |
| YAML | 25410 | 26949 |  +1539 |  +6.06 | 20013 | 21732 |  +1719 |  +8.59 | 5397 | 5217 | -180 | -3.33 | 78.76 | 80.64 |  +1.88 |  +2.39 | 65.67 | 62.81 | -2.86 | -4.36 | 76.86 | 79.41 |  +2.55 |  +3.32 | 64.34 | 61.95 | -2.39 | -3.72 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| JSON_COMPACT | man | 94 | 30 | 0 | 75.81 |
| JSON_COMPACT | opt | 98 | 26 | 0 | 79.03 |
| JSON_PRETTY | man | 95 | 29 | 0 | 76.88 |
| JSON_PRETTY | opt | 90 | 34 | 0 | 72.85 |
| TOON_DEFAULT | man | 92 | 32 | 0 | 74.46 |
| TOON_DEFAULT | opt | 100 | 25 | 0 | 80.24 |
| XML_COMPACT | man | 92 | 32 | 0 | 74.46 |
| XML_COMPACT | opt | 94 | 30 | 0 | 75.54 |
| XML_PRETTY | man | 91 | 33 | 0 | 73.39 |
| XML_PRETTY | opt | 91 | 33 | 0 | 73.65 |
| YAML | man | 98 | 26 | 0 | 78.76 |
| YAML | opt | 100 | 24 | 0 | 80.64 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 94 | 98 |  +4 |  +4.26 | 30 | 26 | -4 | -13.33 | 0 | 0 | 0 | 0.00 | 75.81 | 79.03 |  +3.22 |
| JSON_PRETTY | 95 | 90 | -5 | -5.26 | 29 | 34 |  +5 |  +17.24 | 0 | 0 | 0 | 0.00 | 76.88 | 72.85 | -4.03 |
| TOON_DEFAULT | 92 | 100 |  +8 |  +8.70 | 32 | 25 | -7 | -21.88 | 0 | 0 | 0 | 0.00 | 74.46 | 80.24 |  +5.78 |
| XML_COMPACT | 92 | 94 |  +2 |  +2.17 | 32 | 30 | -2 | -6.25 | 0 | 0 | 0 | 0.00 | 74.46 | 75.54 |  +1.08 |
| XML_PRETTY | 91 | 91 | 0 | 0.00 | 33 | 33 | 0 | 0.00 | 0 | 0 | 0 | 0.00 | 73.39 | 73.65 |  +0.26 |
| YAML | 98 | 100 |  +2 |  +2.04 | 26 | 24 | -2 | -7.69 | 0 | 0 | 0 | 0.00 | 78.76 | 80.64 |  +1.88 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 75.81 | 84.85 | 67.90 | 66.67 | 71.43 |
| JSON_COMPACT | opt | 79.03 | 93.94 | 82.72 | 61.91 | 52.38 |
| JSON_PRETTY | man | 76.88 | 96.36 | 71.60 | 60.31 | 49.20 |
| JSON_PRETTY | opt | 72.85 | 90.30 | 66.67 | 61.90 | 46.03 |
| TOON_DEFAULT | man | 74.46 | 91.21 | 58.64 | 59.52 | 65.87 |
| TOON_DEFAULT | opt | 80.24 | 96.36 | 82.72 | 71.43 | 43.65 |
| XML_COMPACT | man | 74.46 | 87.88 | 67.90 | 66.67 | 55.55 |
| XML_COMPACT | opt | 75.54 | 87.27 | 76.54 | 63.49 | 55.55 |
| XML_PRETTY | man | 73.39 | 88.48 | 69.14 | 53.97 | 58.73 |
| XML_PRETTY | opt | 73.65 | 89.09 | 66.67 | 63.49 | 52.38 |
| YAML | man | 78.76 | 97.58 | 69.14 | 60.31 | 60.32 |
| YAML | opt | 80.64 | 99.39 | 75.31 | 63.49 | 55.56 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 84.85 | 93.94 |  +9.09 |
| JSON_PRETTY | 96.36 | 90.30 | -6.06 |
| TOON_DEFAULT | 91.21 | 96.36 |  +5.15 |
| XML_COMPACT | 87.88 | 87.27 | -0.61 |
| XML_PRETTY | 88.48 | 89.09 |  +0.61 |
| YAML | 97.58 | 99.39 |  +1.82 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 67.90 | 82.72 |  +14.82 |
| JSON_PRETTY | 71.60 | 66.67 | -4.94 |
| TOON_DEFAULT | 58.64 | 82.72 |  +24.07 |
| XML_COMPACT | 67.90 | 76.54 |  +8.64 |
| XML_PRETTY | 69.14 | 66.67 | -2.47 |
| YAML | 69.14 | 75.31 |  +6.17 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 66.67 | 61.91 | -4.76 |
| JSON_PRETTY | 60.31 | 61.90 |  +1.59 |
| TOON_DEFAULT | 59.52 | 71.43 |  +11.91 |
| XML_COMPACT | 66.67 | 63.49 | -3.18 |
| XML_PRETTY | 53.97 | 63.49 |  +9.52 |
| YAML | 60.31 | 63.49 |  +3.18 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 71.43 | 52.38 | -19.05 |
| JSON_PRETTY | 49.20 | 46.03 | -3.17 |
| TOON_DEFAULT | 65.87 | 43.65 | -22.22 |
| XML_COMPACT | 55.55 | 55.55 |  +0.00 |
| XML_PRETTY | 58.73 | 52.38 | -6.35 |
| YAML | 60.32 | 55.56 | -4.76 |

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: on
- **Structure**: nested
- **Formats Tested**: JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML
- **Record Counts**: 31
- **Total Test Cases**: 12

### 4.2 Appendix B: Benchmark Configuration
- **Field Retrieval**: 55 questions (37.50% weight)
- **Filtering**: 21 questions (20.83% weight)
- **Aggregation**: 21 questions (12.50% weight)
- **Structure Awareness**: 27 questions (29.17% weight)

---

- **Report Generated**: 2026-04-09
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