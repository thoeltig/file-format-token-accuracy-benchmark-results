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

1. **CSV** is the cheapest format but also the least accurate. Despite having the lowest read tokens (6700–6989) and the lowest total tokens for mandatory data (16525) **CSV** consistently ranks last in accuracy at roughly 63% in both variants. The tokens saved are offset by the highest proportion of wasted tokens which makes **CSV** a poor choice when information fidelity matters.

2. **TOON_DEFAULT** and **JSON_COMPACT** deliver the best overall efficiency. **TOON_DEFAULT** leads the mandatory efficiency score at 80.39 while **JSON_COMPACT** leads the optional efficiency score at 80.30. Both formats achieve strong accuracy (around 79–80%) at moderate token cost which places them at the top of the composite efficiency ranking.

3. **JSON_PRETTY** achieves the highest single-variant accuracy (82.80% mandatory) but pays a steep token premium. It consumes 27421 total tokens for mandatory data which is 66% more than the cheapest format and its efficiency score drops to 61.66 as a result. High accuracy alone does not guarantee high information value per token.

4. Aggregation is the universally weakest question category. No format exceeds 67% accuracy on aggregation for mandatory data and several drop below 40%. This indicates that Haiku 4.5 struggles with numerical reasoning regardless of how the data is formatted which suggests a model capability limitation rather than a format encoding issue.

5. **XML_PRETTY** is consistently the most expensive format across both variants (25117–28946 total tokens) while offering only mid-range accuracy. Its high read token cost (15089–16166) combined with moderate accuracy (78–80%) results in the lowest efficiency scores overall which makes it the least cost-effective choice.

6. Most formats improve slightly on optional (sparse) data but **JSON_PRETTY** degrades by 6.46 percentage points. This makes **JSON_PRETTY** the least robust format across data variants. In contrast **TOON_DEFAULT** shows near-zero accuracy variation (0.27%) between mandatory and optional data which makes it the most stable format. Its +64% read token increase for optional data is caused by the **TOON_DEFAULT** encoding itself: when data is sparse rather than uniformly filled the format cannot use its collapsed writing style and instead expands each record individually which inflates token cost without affecting accuracy.

7. Field retrieval accuracy is high (90%+ for most formats) while structure awareness and filtering show much greater format sensitivity. This confirms that how well a format conveys structural relationships and enables record-level reasoning matters more than raw value extraction when differentiating format quality.

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
   - Optional: XML_PRETTY 80.11%
   - Mandatory: JSON_PRETTY 82.80%
- Lowest accuracy drift:
   - Optional: JSON_PRETTY ↓ -1.76% ↑ 3.52%
   - Mandatory: JSON_COMPACT ↓ -0.36% ↑ 0.73%
- Most useful read tokens:
   - Optional: XML_PRETTY 12088 / 15089 tokens
   - Mandatory: XML_PRETTY 12603 / 16166 tokens
- Most useful output tokens:
   - Optional: TOON_DEFAULT 9395 / 11768 tokens
   - Mandatory: JSON_PRETTY 10879 / 13138 tokens
- Highest read efficiency (%/token):
   - Optional: JSON_COMPACT 79.19
   - Mandatory: TOON_DEFAULT 84.57
- Highest output efficiency (%/token):
   - Optional: JSON_PRETTY 62.32
   - Mandatory: XML_COMPACT 68.37
- Lowest delta (optional-mandatory):
   - Read tokens: CSV -289 tokens
   - Output tokens: YAML -76 tokens
   - Accuracy: TOON_DEFAULT 0.27%
   - Read efficiency: CSV 0.54
   - Output efficiency: TOON_DEFAULT -0.29

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
   - Optional: CSV 63.18%
   - Mandatory: CSV 63.71%
- Highest accuracy drift:
   - Optional: JSON_COMPACT ↓ -10.81% ↑ 7.43%
   - Mandatory: CSV ↓ -18.99% ↑ 15.19%
- Most wasted read tokens:
   - Optional: JSON_PRETTY 3163 / 13367 tokens
   - Mandatory: XML_PRETTY 3563 / 16166 tokens
- Most wasted output tokens:
   - Optional: CSV 4729 / 12845 tokens
   - Mandatory: CSV 3461 / 9536 tokens
- Lowest read efficiency (%/token):
   - Optional: XML_PRETTY 59.52
   - Mandatory: XML_PRETTY 54.60
- Lowest output efficiency (%/token):
   - Optional: CSV 51.89
   - Mandatory: JSON_COMPACT 58.23
- Highest delta (optional-mandatory):
   - Read tokens: TOON_DEFAULT 4513 tokens
   - Output tokens: JSON_PRETTY -4836 tokens
   - Accuracy: JSON_PRETTY -6.46%
   - Read efficiency: TOON_DEFAULT -14.08
   - Output efficiency: JSON_PRETTY 19.21

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|
| XML_COMPACT ≈ 75s | CSV ≈ 6989 | JSON_COMPACT ≈ 228 | XML_COMPACT ≈ 9086 | XML_COMPACT ≈ 9429 | CSV ≈ 16525 | JSON_PRETTY ≈ 83% | TOON_DEFAULT ≈ 85 | XML_COMPACT ≈ 79 | TOON_DEFAULT ≈ 80 |
| CSV (+3.9%) | TOON_DEFAULT (+0.8%) | YAML (+1.5%) | CSV (+1.3%) | CSV (+1.1%) | TOON_DEFAULT (+13.3%) | TOON_DEFAULT (-3.2%) | JSON_COMPACT (-12.5%) | YAML (-7.5%) | CSV (-7.2%) |
| YAML (+17.2%) | JSON_COMPACT (+32.6%) | TOON_DEFAULT (+26.1%) | YAML (+14.0%) | YAML (+12.3%) | XML_COMPACT (+27.8%) | XML_PRETTY (-4.8%) | CSV (-12.9%) | TOON_DEFAULT (-12.1%) | XML_COMPACT (-9.1%) |
| TOON_DEFAULT (+27.4%) | XML_COMPACT (+67.3%) | CSV (+44.7%) | TOON_DEFAULT (+25.3%) | TOON_DEFAULT (+23.8%) | JSON_COMPACT (+35.7%) | XML_COMPACT (-5.4%) | XML_COMPACT (-19.2%) | CSV (-12.9%) | YAML (-15.4%) |
| XML_PRETTY (+36.2%) | YAML (+79.6%) | JSON_PRETTY (+49.8%) | XML_PRETTY (+36.9%) | XML_PRETTY (+35.5%) | YAML (+40.0%) | YAML (-5.6%) | YAML (-22.6%) | JSON_PRETTY (-18.4%) | JSON_COMPACT (-15.6%) |
| JSON_PRETTY (+40.7%) | JSON_PRETTY (+104.4%) | XML_COMPACT (+50.5%) | JSON_PRETTY (+40.8%) | JSON_PRETTY (+39.3%) | JSON_PRETTY (+65.9%) | JSON_COMPACT (-8.3%) | JSON_PRETTY (-24.4%) | XML_PRETTY (-20.4%) | JSON_PRETTY (-23.3%) |
| JSON_COMPACT (+45.8%) | XML_PRETTY (+131.3%) | XML_PRETTY (+50.5%) | JSON_COMPACT (+42.3%) | JSON_COMPACT (+39.6%) | XML_PRETTY (+75.2%) | CSV (-19.1%) | XML_PRETTY (-35.4%) | JSON_COMPACT (-25.9%) | XML_PRETTY (-32.1%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 73s | CSV ≈ 6700 | CSV ≈ 231 | JSON_PRETTY ≈ 7966 | JSON_PRETTY ≈ 8302 | JSON_COMPACT ≈ 18753 | XML_PRETTY ≈ 80% | JSON_COMPACT ≈ 79 | JSON_PRETTY ≈ 83 | JSON_COMPACT ≈ 80 |
| XML_PRETTY (+7.9%) | JSON_COMPACT (+30.6%) | XML_PRETTY (+0.6%) | JSON_COMPACT (+21.4%) | JSON_COMPACT (+20.5%) | CSV (+4.2%) | TOON_DEFAULT (-0.3%) | CSV (-6.3%) | XML_PRETTY (-7.0%) | XML_COMPACT (-11.0%) |
| JSON_COMPACT (+13.2%) | XML_COMPACT (+63.1%) | YAML (+44.3%) | XML_PRETTY (+23.0%) | XML_PRETTY (+20.8%) | JSON_PRETTY (+15.5%) | JSON_COMPACT (-0.5%) | XML_COMPACT (-8.7%) | JSON_COMPACT (-7.3%) | JSON_PRETTY (-11.6%) |
| YAML (+21.0%) | TOON_DEFAULT (+72.6%) | XML_COMPACT (+45.2%) | YAML (+27.7%) | YAML (+26.6%) | YAML (+18.8%) | XML_COMPACT (-0.5%) | TOON_DEFAULT (-11.0%) | YAML (-11.4%) | YAML (-11.8%) |
| TOON_DEFAULT (+28.9%) | YAML (+75.7%) | JSON_PRETTY (+45.5%) | XML_COMPACT (+40.1%) | XML_COMPACT (+38.5%) | XML_COMPACT (+19.6%) | YAML (-1.9%) | YAML (-13.3%) | XML_COMPACT (-16.1%) | TOON_DEFAULT (-13.5%) |
| XML_COMPACT (+30.1%) | JSON_PRETTY (+99.5%) | JSON_COMPACT (+46.2%) | TOON_DEFAULT (+43.5%) | TOON_DEFAULT (+41.7%) | TOON_DEFAULT (+24.4%) | JSON_PRETTY (-3.8%) | JSON_PRETTY (-21.3%) | TOON_DEFAULT (-17.5%) | CSV (-16.7%) |
| CSV (+50.1%) | XML_PRETTY (+125.2%) | TOON_DEFAULT (+47.3%) | CSV (+58.3%) | CSV (+54.7%) | XML_PRETTY (+33.9%) | CSV (-16.9%) | XML_PRETTY (-24.8%) | CSV (-37.8%) | XML_PRETTY (-18.6%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 99% | JSON_PRETTY ≈ 81% | YAML ≈ 73% | TOON_DEFAULT ≈ 67% |
| YAML (-0.6%) | XML_COMPACT (-6.2%) | XML_PRETTY (-4.8%) | JSON_PRETTY (-7.9%) |
| JSON_COMPACT (-1.8%) | XML_PRETTY (-6.2%) | TOON_DEFAULT (-7.9%) | XML_COMPACT (-12.7%) |
| XML_PRETTY (-1.8%) | TOON_DEFAULT (-11.1%) | JSON_PRETTY (-7.9%) | CSV (-19.0%) |
| TOON_DEFAULT (-4.8%) | JSON_COMPACT (-13.6%) | XML_COMPACT (-9.5%) | XML_PRETTY (-27.0%) |
| XML_COMPACT (-6.7%) | YAML (-16.0%) | JSON_COMPACT (-14.3%) | YAML (-27.0%) |
| CSV (-25.5%) | CSV (-17.3%) | CSV (-20.6%) | JSON_COMPACT (-28.6%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| XML_COMPACT ≈ 99% | JSON_COMPACT ≈ 88% | JSON_COMPACT ≈ 71% | XML_PRETTY ≈ 62% |
| XML_PRETTY (-0.6%) | TOON_DEFAULT (-4.9%) | TOON_DEFAULT (-0.0%) | JSON_COMPACT (-14.3%) |
| YAML (-1.2%) | YAML (-8.6%) | YAML (-1.6%) | TOON_DEFAULT (-14.3%) |
| JSON_PRETTY (-3.6%) | XML_COMPACT (-11.1%) | XML_COMPACT (-4.8%) | XML_COMPACT (-15.9%) |
| TOON_DEFAULT (-4.8%) | JSON_PRETTY (-12.3%) | XML_PRETTY (-6.4%) | JSON_PRETTY (-19.0%) |
| JSON_COMPACT (-7.9%) | XML_PRETTY (-18.5%) | CSV (-7.9%) | CSV (-27.0%) |
| CSV (-23.6%) | CSV (-27.2%) | JSON_PRETTY (-9.5%) | YAML (-27.0%) |


#### 2.1.5 Conclusion

The benchmark reveals a clear tension between token economy and information fidelity that makes naive "fewest tokens wins" reasoning misleading. **CSV** consumes the fewest read tokens but wastes the most output tokens due to consistently poor accuracy around 63% which results in roughly 36% of all tokens being spent on incorrect answers. On the other end **XML_PRETTY** delivers competitive accuracy (78–80%) but at such extreme token cost (up to 28946 total) that its efficiency score ranks last.

The most effective formats occupy the middle ground. **TOON_DEFAULT** stands out for mandatory data with the highest efficiency score (80.39) which combines strong accuracy (79.57%) with the second-lowest total token cost (18719). It also demonstrates exceptional robustness with only a 0.27% accuracy shift between mandatory and optional variants. **JSON_COMPACT** dominates the optional data scenario with the highest efficiency score (80.30) and the lowest total token cost (18753) while achieving 79.57% accuracy. Both formats deliver approximately 80% of the information value of the best-accuracy format (**JSON_PRETTY** at 82.80%) while using 30–40% fewer tokens.

The category-level analysis reveals that format choice primarily affects structure awareness and filtering performance where accuracy ranges span 20+ percentage points across formats. Field retrieval is largely format-insensitive above 90% for all structured formats and aggregation remains weak across the board regardless of format. This suggests that the practical impact of format selection is concentrated in how well a format communicates data relationships and record boundaries rather than individual field values.

A notable asymmetry exists in how formats handle the mandatory-to-optional transition. Formats like **JSON_COMPACT** and **XML_PRETTY** improve in accuracy when moving to optional (sparse) data while **JSON_PRETTY** degrades significantly (−6.46%). This indicates that data density interacts with format structure in non-trivial ways and the best format for one data profile may not be optimal for another.

For practitioners using Haiku 4.5 with flat data structures and thinking enabled the recommendation is to prefer **TOON_DEFAULT** or **JSON_COMPACT** as they provide the best balance of token efficiency and accuracy. **CSV** should be avoided for LLM consumption despite its minimal token footprint and **XML_PRETTY** should be avoided due to its disproportionate token overhead relative to its accuracy gains.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6989 | 9536 | 16525 | 1.444 | 74.250 | 63.71 | 4452.692 | 2536.308 | 6075.598 | 3460.735 | 73.65 | 68.49 | 74.57 |
| CSV | opt | 6700 | 12845 | 19545 | 1.429 | 101.723 | 63.18 | 4233.060 | 2466.940 | 8115.261 | 4729.406 | 74.19 | 51.89 | 66.92 |
| JSON_COMPACT | man | 9268 | 13161 | 22429 | 2.149 | 104.298 | 74.46 | 6900.953 | 2367.047 | 9799.433 | 3361.234 | 73.97 | 58.23 | 67.86 |
| JSON_COMPACT | opt | 8748 | 10005 | 18753 | 2.113 | 77.965 | 79.57 | 6960.784 | 1787.216 | 7961.243 | 2044.090 | 79.19 | 77.29 | 80.30 |
| JSON_PRETTY | man | 14283 | 13138 | 27421 | 1.694 | 103.204 | 82.80 | 11826.324 | 2456.676 | 10878.540 | 2259.793 | 63.95 | 64.18 | 61.66 |
| JSON_PRETTY | opt | 13367 | 8302 | 21669 | 1.680 | 64.242 | 76.34 | 10204.368 | 3162.632 | 6337.747 | 1964.253 | 62.32 | 83.39 | 71.01 |
| TOON_DEFAULT | man | 7048 | 11671 | 18719 | 1.442 | 91.805 | 79.57 | 5608.094 | 1439.906 | 9286.482 | 2384.351 | 84.57 | 69.12 | 80.39 |
| TOON_DEFAULT | opt | 11561 | 11768 | 23329 | 1.698 | 92.155 | 79.84 | 9230.302 | 2330.698 | 9395.172 | 2372.328 | 70.48 | 68.83 | 69.46 |
| XML_COMPACT | man | 11693 | 9429 | 21122 | 2.366 | 73.274 | 77.42 | 9052.721 | 2640.279 | 7299.674 | 2128.993 | 68.37 | 78.62 | 73.09 |
| XML_COMPACT | opt | 10930 | 11498 | 22428 | 2.340 | 90.024 | 79.57 | 8697.001 | 2232.999 | 9149.224 | 2349.109 | 72.29 | 69.97 | 71.44 |
| XML_PRETTY | man | 16166 | 12780 | 28946 | 1.934 | 100.304 | 77.96 | 12603.014 | 3562.986 | 9963.548 | 2816.785 | 54.60 | 62.55 | 54.60 |
| XML_PRETTY | opt | 15089 | 10028 | 25117 | 1.917 | 79.000 | 80.11 | 12087.798 | 3001.202 | 8033.698 | 1994.635 | 59.52 | 77.56 | 65.33 |
| YAML | man | 12554 | 10585 | 23139 | 1.664 | 83.500 | 77.15 | 9685.411 | 2868.589 | 8166.328 | 2418.672 | 65.46 | 72.75 | 68.03 |
| YAML | opt | 11771 | 10509 | 22280 | 1.649 | 82.065 | 78.23 | 9208.453 | 2562.547 | 8221.451 | 2287.882 | 68.69 | 73.88 | 70.86 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6989 | 6700 | -289 | -4.14 | 329 | 231 |  +3407 |  +1035.56 | 9207 | 12614 |  +3308 |  +35.93 | 9536 | 12844 |  +3308 |  +34.69 | 16525 | 19544 |  +3019 |  +18.27 |
| JSON_COMPACT | 9268 | 8748 | -520 | -5.61 | 228 | 338 | -3265 | -1432.02 | 12933 | 9668 | -3155 | -24.39 | 13161 | 10006 | -3155 | -23.97 | 22429 | 18754 | -3675 | -16.39 |
| JSON_PRETTY | 14283 | 13367 | -916 | -6.41 | 341 | 336 | -4831 | -1416.72 | 12797 | 7966 | -4836 | -37.79 | 13138 | 8302 | -4836 | -36.81 | 27421 | 21669 | -5752 | -20.98 |
| TOON_DEFAULT | 7048 | 11561 |  +4513 |  +64.03 | 287 | 340 |  +44 |  +15.33 | 11384 | 11428 |  +97 |  +0.85 | 11671 | 11768 |  +97 |  +0.83 | 18719 | 23329 |  +4610 |  +24.63 |
| XML_COMPACT | 11693 | 10930 | -763 | -6.53 | 343 | 336 |  +2077 |  +605.54 | 9086 | 11163 |  +2070 |  +22.78 | 9429 | 11499 |  +2070 |  +21.95 | 21122 | 22429 |  +1307 |  +6.19 |
| XML_PRETTY | 16166 | 15089 | -1077 | -6.66 | 343 | 233 | -2642 | -770.26 | 12438 | 9796 | -2752 | -22.13 | 12780 | 10028 | -2752 | -21.53 | 28946 | 25117 | -3829 | -13.23 |
| YAML | 12554 | 11771 | -783 | -6.24 | 231 | 333 | -178 | -77.06 | 10354 | 10176 | -76 | -0.73 | 10585 | 10509 | -76 | -0.72 | 23139 | 22280 | -859 | -3.71 |

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
| CSV | man | 1.444 | 10.248 | 225.452 | 0.912 | 0.668 | 0.386 |
| CSV | opt | 1.429 | 10.618 | 216.129 | 0.943 | 0.492 | 0.323 |
| JSON_COMPACT | man | 2.149 | 13.589 | 298.968 | 0.803 | 0.566 | 0.332 |
| JSON_COMPACT | opt | 2.113 | 13.864 | 282.194 | 0.910 | 0.795 | 0.424 |
| JSON_PRETTY | man | 1.694 | 20.943 | 460.742 | 0.580 | 0.630 | 0.302 |
| JSON_PRETTY | opt | 1.680 | 21.184 | 431.194 | 0.571 | 0.920 | 0.352 |
| TOON_DEFAULT | man | 1.442 | 10.334 | 227.355 | 1.129 | 0.700 | 0.429 |
| TOON_DEFAULT | opt | 1.698 | 18.322 | 372.935 | 0.691 | 0.715 | 0.347 |
| XML_COMPACT | man | 2.366 | 17.145 | 377.194 | 0.662 | 0.821 | 0.367 |
| XML_COMPACT | opt | 2.340 | 17.322 | 352.581 | 0.728 | 0.692 | 0.355 |
| XML_PRETTY | man | 1.934 | 23.704 | 521.484 | 0.482 | 0.610 | 0.269 |
| XML_PRETTY | opt | 1.917 | 23.913 | 486.742 | 0.531 | 0.799 | 0.319 |
| YAML | man | 1.664 | 18.408 | 404.968 | 0.615 | 0.729 | 0.333 |
| YAML | opt | 1.649 | 18.655 | 379.710 | 0.665 | 0.744 | 0.351 |

#### 2.5.2 Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.444 | 1.429 | -0.015 | -1.04 | 10.248 | 10.618 |  +0.370 |  +3.61 | 225.452 | 216.129 | -9.323 | -4.14 | 0.912 | 0.943 |  +0.031 |  +3.40 | 0.668 | 0.492 | -0.176 | -26.35 | 0.386 | 0.323 | -0.063 | -16.32 |
| JSON_COMPACT | 2.149 | 2.113 | -0.036 | -1.68 | 13.589 | 13.864 |  +0.275 |  +2.02 | 298.968 | 282.194 | -16.774 | -5.61 | 0.803 | 0.910 |  +0.107 |  +13.33 | 0.566 | 0.795 |  +0.229 |  +40.46 | 0.332 | 0.424 |  +0.092 |  +27.71 |
| JSON_PRETTY | 1.694 | 1.680 | -0.014 | -0.83 | 20.943 | 21.184 |  +0.241 |  +1.15 | 460.742 | 431.194 | -29.548 | -6.41 | 0.580 | 0.571 | -0.009 | -1.55 | 0.630 | 0.920 |  +0.290 |  +46.03 | 0.302 | 0.352 |  +0.050 |  +16.56 |
| TOON_DEFAULT | 1.442 | 1.698 |  +0.256 |  +17.75 | 10.334 | 18.322 |  +7.988 |  +77.30 | 227.355 | 372.935 |  +145.580 |  +64.03 | 1.129 | 0.691 | -0.438 | -38.80 | 0.700 | 0.715 |  +0.014 |  +2.00 | 0.429 | 0.347 | -0.083 | -19.21 |
| XML_COMPACT | 2.366 | 2.340 | -0.026 | -1.10 | 17.145 | 17.322 |  +0.177 |  +1.03 | 377.194 | 352.581 | -24.613 | -6.53 | 0.662 | 0.728 |  +0.066 |  +9.97 | 0.821 | 0.692 | -0.129 | -15.71 | 0.367 | 0.355 | -0.012 | -3.27 |
| XML_PRETTY | 1.934 | 1.917 | -0.017 | -0.88 | 23.704 | 23.913 |  +0.209 |  +0.88 | 521.484 | 486.742 | -34.742 | -6.66 | 0.482 | 0.531 |  +0.049 |  +10.17 | 0.610 | 0.799 |  +0.189 |  +30.98 | 0.269 | 0.319 |  +0.050 |  +18.59 |
| YAML | 1.664 | 1.649 | -0.015 | -0.90 | 18.408 | 18.655 |  +0.247 |  +1.34 | 404.968 | 379.710 | -25.258 | -6.24 | 0.615 | 0.665 |  +0.050 |  +8.13 | 0.729 | 0.744 |  +0.015 |  +2.06 | 0.333 | 0.351 |  +0.018 |  +5.41 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6989 | 4453 | 2536 | 9536 | 6076 | 3461 | 16525 | 10528 | 5997 | 63.71 | 73.65 | 68.49 | 74.57 | 63.32 | 73.38 | 68.22 | 74.30 |
| CSV | opt | 6700 | 4233 | 2467 | 12845 | 8115 | 4729 | 19545 | 12348 | 7196 | 63.18 | 74.19 | 51.89 | 66.92 | 63.41 | 74.36 | 52.05 | 67.08 |
| JSON_COMPACT | man | 9268 | 6901 | 2367 | 13161 | 9799 | 3361 | 22429 | 16700 | 5728 | 74.46 | 73.97 | 58.23 | 67.86 | 73.39 | 73.22 | 57.48 | 67.11 |
| JSON_COMPACT | opt | 8748 | 6961 | 1787 | 10005 | 7961 | 2044 | 18753 | 14922 | 3831 | 79.57 | 79.19 | 77.29 | 80.30 | 80.50 | 79.84 | 77.94 | 80.95 |
| JSON_PRETTY | man | 14283 | 11826 | 2457 | 13138 | 10879 | 2260 | 27421 | 22705 | 4716 | 82.80 | 63.95 | 64.18 | 61.66 | 81.94 | 63.34 | 63.58 | 61.06 |
| JSON_PRETTY | opt | 13367 | 10204 | 3163 | 8302 | 6338 | 1964 | 21669 | 16542 | 5127 | 76.34 | 62.32 | 83.39 | 71.01 | 75.90 | 62.01 | 83.08 | 70.70 |
| TOON_DEFAULT | man | 7048 | 5608 | 1440 | 11671 | 9286 | 2384 | 18719 | 14895 | 3824 | 79.57 | 84.57 | 69.12 | 80.39 | 77.87 | 83.38 | 67.93 | 79.20 |
| TOON_DEFAULT | opt | 11561 | 9230 | 2331 | 11768 | 9395 | 2372 | 23329 | 18625 | 4703 | 79.84 | 70.48 | 68.83 | 69.46 | 80.19 | 70.73 | 69.08 | 69.70 |
| XML_COMPACT | man | 11693 | 9053 | 2640 | 9429 | 7300 | 2129 | 21122 | 16352 | 4769 | 77.42 | 68.37 | 78.62 | 73.09 | 76.71 | 67.88 | 78.12 | 72.59 |
| XML_COMPACT | opt | 10930 | 8697 | 2233 | 11498 | 9149 | 2349 | 22428 | 17846 | 4582 | 79.57 | 72.29 | 69.97 | 71.44 | 79.02 | 71.91 | 69.58 | 71.06 |
| XML_PRETTY | man | 16166 | 12603 | 3563 | 12780 | 9964 | 2817 | 28946 | 22567 | 6380 | 77.96 | 54.60 | 62.55 | 54.60 | 77.74 | 54.45 | 62.40 | 54.44 |
| XML_PRETTY | opt | 15089 | 12088 | 3001 | 10028 | 8034 | 1995 | 25117 | 20121 | 4996 | 80.11 | 59.52 | 77.56 | 65.33 | 78.28 | 58.23 | 76.28 | 64.05 |
| YAML | man | 12554 | 9685 | 2869 | 10585 | 8166 | 2419 | 23139 | 17852 | 5287 | 77.15 | 65.46 | 72.75 | 68.03 | 76.30 | 64.86 | 72.16 | 67.44 |
| YAML | opt | 11771 | 9208 | 2563 | 10509 | 8221 | 2288 | 22280 | 17430 | 4850 | 78.23 | 68.69 | 73.88 | 70.86 | 78.55 | 68.92 | 74.11 | 71.08 |

#### 2.6.2 Read Tokens Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6989 | 6700 | -289 | -4.14 | 4453 | 4233 | -220 | -4.93 | 2536 | 2467 | -69 | -2.74 | 63.71 | 63.18 | -0.53 | -0.83 | 73.65 | 74.19 |  +0.54 |  +0.74 | 63.32 | 63.41 |  +0.09 |  +0.14 | 73.38 | 74.36 |  +0.98 |  +1.33 |
| JSON_COMPACT | 9268 | 8748 | -520 | -5.61 | 6901 | 6961 |  +60 |  +0.87 | 2367 | 1787 | -580 | -24.50 | 74.46 | 79.57 |  +5.11 |  +6.86 | 73.97 | 79.19 |  +5.22 |  +7.06 | 73.39 | 80.50 |  +7.11 |  +9.69 | 73.22 | 79.84 |  +6.62 |  +9.04 |
| JSON_PRETTY | 14283 | 13367 | -916 | -6.41 | 11826 | 10204 | -1622 | -13.72 | 2457 | 3163 |  +706 |  +28.73 | 82.80 | 76.34 | -6.46 | -7.80 | 63.95 | 62.32 | -1.63 | -2.54 | 81.94 | 75.90 | -6.04 | -7.37 | 63.34 | 62.01 | -1.33 | -2.10 |
| TOON_DEFAULT | 7048 | 11561 |  +4513 |  +64.03 | 5608 | 9230 |  +3622 |  +64.59 | 1440 | 2331 |  +891 |  +61.86 | 79.57 | 79.84 |  +0.27 |  +0.34 | 84.57 | 70.48 | -14.08 | -16.65 | 77.87 | 80.19 |  +2.32 |  +2.98 | 83.38 | 70.73 | -12.65 | -15.17 |
| XML_COMPACT | 11693 | 10930 | -763 | -6.53 | 9053 | 8697 | -356 | -3.93 | 2640 | 2233 | -407 | -15.43 | 77.42 | 79.57 |  +2.15 |  +2.78 | 68.37 | 72.29 |  +3.92 |  +5.73 | 76.71 | 79.02 |  +2.31 |  +3.01 | 67.88 | 71.91 |  +4.03 |  +5.94 |
| XML_PRETTY | 16166 | 15089 | -1077 | -6.66 | 12603 | 12088 | -515 | -4.09 | 3563 | 3001 | -562 | -15.77 | 77.96 | 80.11 |  +2.15 |  +2.76 | 54.60 | 59.52 |  +4.91 |  +8.99 | 77.74 | 78.28 |  +0.54 |  +0.69 | 54.45 | 58.23 |  +3.78 |  +6.95 |
| YAML | 12554 | 11771 | -783 | -6.24 | 9685 | 9208 | -477 | -4.92 | 2869 | 2563 | -306 | -10.67 | 77.15 | 78.23 |  +1.08 |  +1.40 | 65.46 | 68.69 |  +3.23 |  +4.94 | 76.30 | 78.55 |  +2.25 |  +2.95 | 64.86 | 68.92 |  +4.05 |  +6.25 |

#### 2.6.3 Output Tokens Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 9536 | 12844 |  +3308 |  +34.69 | 6076 | 8116 |  +2040 |  +33.57 | 3461 | 4730 |  +1269 |  +36.66 | 63.71 | 63.18 | -0.53 | -0.83 | 68.49 | 51.89 | -16.60 | -24.24 | 63.32 | 63.41 |  +0.09 |  +0.14 | 68.22 | 52.05 | -16.17 | -23.70 |
| JSON_COMPACT | 13161 | 10006 | -3155 | -23.97 | 9799 | 7961 | -1838 | -18.76 | 3361 | 2044 | -1317 | -39.19 | 74.46 | 79.57 |  +5.11 |  +6.86 | 58.23 | 77.29 |  +19.06 |  +32.73 | 73.39 | 80.50 |  +7.11 |  +9.69 | 57.48 | 77.94 |  +20.46 |  +35.59 |
| JSON_PRETTY | 13138 | 8302 | -4836 | -36.81 | 10879 | 6338 | -4541 | -41.74 | 2260 | 1964 | -296 | -13.08 | 82.80 | 76.34 | -6.46 | -7.80 | 64.18 | 83.39 |  +19.21 |  +29.93 | 81.94 | 75.90 | -6.04 | -7.37 | 63.58 | 83.08 |  +19.50 |  +30.68 |
| TOON_DEFAULT | 11671 | 11768 |  +97 |  +0.83 | 9286 | 9395 |  +109 |  +1.17 | 2384 | 2372 | -12 | -0.50 | 79.57 | 79.84 |  +0.27 |  +0.34 | 69.12 | 68.83 | -0.29 | -0.41 | 77.87 | 80.19 |  +2.32 |  +2.98 | 67.93 | 69.08 |  +1.15 |  +1.69 |
| XML_COMPACT | 9429 | 11499 |  +2070 |  +21.95 | 7300 | 9150 |  +1850 |  +25.34 | 2129 | 2349 |  +220 |  +10.34 | 77.42 | 79.57 |  +2.15 |  +2.78 | 78.62 | 69.97 | -8.65 | -11.00 | 76.71 | 79.02 |  +2.31 |  +3.01 | 78.12 | 69.58 | -8.54 | -10.93 |
| XML_PRETTY | 12780 | 10028 | -2752 | -21.53 | 9964 | 8034 | -1930 | -19.37 | 2817 | 1995 | -822 | -29.19 | 77.96 | 80.11 |  +2.15 |  +2.76 | 62.55 | 77.56 |  +15.01 |  +23.99 | 77.74 | 78.28 |  +0.54 |  +0.69 | 62.40 | 76.28 |  +13.88 |  +22.25 |
| YAML | 10585 | 10509 | -76 | -0.71 | 8166 | 8221 |  +55 |  +0.68 | 2419 | 2288 | -131 | -5.41 | 77.15 | 78.23 |  +1.08 |  +1.40 | 72.75 | 73.88 |  +1.13 |  +1.55 | 76.30 | 78.55 |  +2.25 |  +2.95 | 72.16 | 74.11 |  +1.95 |  +2.70 |

#### 2.6.4 Total Tokens Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 16525 | 19544 |  +3019 |  +18.27 | 10528 | 12348 |  +1820 |  +17.29 | 5997 | 7196 |  +1199 |  +20.00 | 63.71 | 63.18 | -0.53 | -0.83 | 74.57 | 66.92 | -7.65 | -10.26 | 63.32 | 63.41 |  +0.09 |  +0.14 | 74.30 | 67.08 | -7.22 | -9.71 |
| JSON_COMPACT | 22429 | 18754 | -3675 | -16.39 | 16700 | 14922 | -1778 | -10.65 | 5728 | 3831 | -1897 | -33.12 | 74.46 | 79.57 |  +5.11 |  +6.86 | 67.86 | 80.30 |  +12.44 |  +18.33 | 73.39 | 80.50 |  +7.11 |  +9.69 | 67.11 | 80.95 |  +13.84 |  +20.62 |
| JSON_PRETTY | 27421 | 21669 | -5752 | -20.98 | 22705 | 16542 | -6163 | -27.14 | 4716 | 5126 |  +410 |  +8.70 | 82.80 | 76.34 | -6.46 | -7.80 | 61.66 | 71.01 |  +9.35 |  +15.16 | 81.94 | 75.90 | -6.04 | -7.37 | 61.06 | 70.70 |  +9.64 |  +15.79 |
| TOON_DEFAULT | 18719 | 23329 |  +4610 |  +24.63 | 14895 | 18626 |  +3731 |  +25.05 | 3824 | 4703 |  +879 |  +22.98 | 79.57 | 79.84 |  +0.27 |  +0.34 | 80.39 | 69.46 | -10.93 | -13.59 | 77.87 | 80.19 |  +2.32 |  +2.98 | 79.20 | 69.70 | -9.49 | -11.98 |
| XML_COMPACT | 21122 | 22429 |  +1307 |  +6.19 | 16352 | 17846 |  +1494 |  +9.14 | 4769 | 4582 | -187 | -3.92 | 77.42 | 79.57 |  +2.15 |  +2.78 | 73.09 | 71.44 | -1.65 | -2.25 | 76.71 | 79.02 |  +2.31 |  +3.01 | 72.59 | 71.06 | -1.53 | -2.11 |
| XML_PRETTY | 28946 | 25117 | -3829 | -13.23 | 22567 | 20122 | -2445 | -10.83 | 6380 | 4996 | -1384 | -21.69 | 77.96 | 80.11 |  +2.15 |  +2.76 | 54.60 | 65.33 |  +10.74 |  +19.67 | 77.74 | 78.28 |  +0.54 |  +0.69 | 54.44 | 64.05 |  +9.61 |  +17.65 |
| YAML | 23139 | 22280 | -859 | -3.71 | 17852 | 17430 | -422 | -2.36 | 5287 | 4850 | -437 | -8.26 | 77.15 | 78.23 |  +1.08 |  +1.40 | 68.03 | 70.86 |  +2.83 |  +4.15 | 76.30 | 78.55 |  +2.25 |  +2.95 | 67.44 | 71.08 |  +3.64 |  +5.40 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| CSV | man | 79 | 45 | 0 | 63.71 |
| CSV | opt | 78 | 46 | 0 | 63.18 |
| JSON_COMPACT | man | 92 | 32 | 0 | 74.46 |
| JSON_COMPACT | opt | 99 | 25 | 0 | 79.57 |
| JSON_PRETTY | man | 103 | 21 | 0 | 82.80 |
| JSON_PRETTY | opt | 95 | 29 | 0 | 76.34 |
| TOON_DEFAULT | man | 99 | 25 | 0 | 79.57 |
| TOON_DEFAULT | opt | 99 | 25 | 0 | 79.84 |
| XML_COMPACT | man | 96 | 28 | 0 | 77.42 |
| XML_COMPACT | opt | 99 | 25 | 0 | 79.57 |
| XML_PRETTY | man | 97 | 27 | 0 | 77.96 |
| XML_PRETTY | opt | 99 | 25 | 0 | 80.11 |
| YAML | man | 96 | 28 | 0 | 77.15 |
| YAML | opt | 97 | 27 | 0 | 78.23 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 79 | 78 | -1 | -1.27 | 45 | 46 |  +1 |  +2.22 | 0 | 0 | 0 | 0.00 | 63.71 | 63.18 | -0.53 |
| JSON_COMPACT | 92 | 99 |  +7 |  +7.61 | 32 | 25 | -7 | -21.88 | 0 | 0 | 0 | 0.00 | 74.46 | 79.57 |  +5.11 |
| JSON_PRETTY | 103 | 95 | -8 | -7.77 | 21 | 29 |  +8 |  +38.10 | 0 | 0 | 0 | 0.00 | 82.80 | 76.34 | -6.46 |
| TOON_DEFAULT | 99 | 99 | 0 | 0.00 | 25 | 25 | 0 | 0.00 | 0 | 0 | 0 | 0.00 | 79.57 | 79.84 |  +0.27 |
| XML_COMPACT | 96 | 99 |  +3 |  +3.13 | 28 | 25 | -3 | -10.71 | 0 | 0 | 0 | 0.00 | 77.42 | 79.57 |  +2.15 |
| XML_PRETTY | 97 | 99 |  +2 |  +2.06 | 27 | 25 | -2 | -7.41 | 0 | 0 | 0 | 0.00 | 77.96 | 80.11 |  +2.15 |
| YAML | 96 | 97 |  +1 |  +1.04 | 28 | 27 | -1 | -3.57 | 0 | 0 | 0 | 0.00 | 77.15 | 78.23 |  +1.08 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| CSV | man | 63.71 | 73.94 | 64.20 | 52.38 | 47.62 |
| CSV | opt | 63.18 | 75.15 | 60.49 | 63.49 | 34.92 |
| JSON_COMPACT | man | 74.46 | 97.58 | 67.90 | 58.73 | 38.10 |
| JSON_COMPACT | opt | 79.57 | 90.91 | 87.66 | 71.43 | 47.62 |
| JSON_PRETTY | man | 82.80 | 99.39 | 81.48 | 65.08 | 58.73 |
| JSON_PRETTY | opt | 76.34 | 95.15 | 75.31 | 61.90 | 42.86 |
| TOON_DEFAULT | man | 79.57 | 94.55 | 70.37 | 65.08 | 66.67 |
| TOON_DEFAULT | opt | 79.84 | 93.94 | 82.72 | 71.43 | 47.62 |
| XML_COMPACT | man | 77.42 | 92.73 | 75.31 | 63.49 | 53.97 |
| XML_COMPACT | opt | 79.57 | 98.79 | 76.55 | 66.67 | 46.03 |
| XML_PRETTY | man | 77.96 | 97.57 | 75.31 | 68.26 | 39.68 |
| XML_PRETTY | opt | 80.11 | 98.18 | 69.14 | 65.08 | 61.90 |
| YAML | man | 77.15 | 98.79 | 65.43 | 73.02 | 39.68 |
| YAML | opt | 78.23 | 97.58 | 79.01 | 69.84 | 34.92 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 73.94 | 75.15 |  +1.21 |
| JSON_COMPACT | 97.58 | 90.91 | -6.67 |
| JSON_PRETTY | 99.39 | 95.15 | -4.24 |
| TOON_DEFAULT | 94.55 | 93.94 | -0.61 |
| XML_COMPACT | 92.73 | 98.79 |  +6.06 |
| XML_PRETTY | 97.57 | 98.18 |  +0.61 |
| YAML | 98.79 | 97.58 | -1.21 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 64.20 | 60.49 | -3.70 |
| JSON_COMPACT | 67.90 | 87.66 |  +19.76 |
| JSON_PRETTY | 81.48 | 75.31 | -6.17 |
| TOON_DEFAULT | 70.37 | 82.72 |  +12.34 |
| XML_COMPACT | 75.31 | 76.55 |  +1.24 |
| XML_PRETTY | 75.31 | 69.14 | -6.17 |
| YAML | 65.43 | 79.01 |  +13.58 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 52.38 | 63.49 |  +11.11 |
| JSON_COMPACT | 58.73 | 71.43 |  +12.70 |
| JSON_PRETTY | 65.08 | 61.90 | -3.17 |
| TOON_DEFAULT | 65.08 | 71.43 |  +6.35 |
| XML_COMPACT | 63.49 | 66.67 |  +3.18 |
| XML_PRETTY | 68.26 | 65.08 | -3.18 |
| YAML | 73.02 | 69.84 | -3.18 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 47.62 | 34.92 | -12.70 |
| JSON_COMPACT | 38.10 | 47.62 |  +9.52 |
| JSON_PRETTY | 58.73 | 42.86 | -15.87 |
| TOON_DEFAULT | 66.67 | 47.62 | -19.05 |
| XML_COMPACT | 53.97 | 46.03 | -7.93 |
| XML_PRETTY | 39.68 | 61.90 |  +22.22 |
| YAML | 39.68 | 34.92 | -4.76 |

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: on
- **Structure**: flat
- **Formats Tested**: CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML
- **Record Counts**: 31
- **Total Test Cases**: 14

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
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_off\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_on_verify\BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)