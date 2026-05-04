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

1. **JSON_COMPACT** is the most token-efficient format overall. It consumes the fewest total tokens (18,250 to 18,446) and achieves the highest total efficiency scores (82 to 84) across both data variants. Its read token count is 25% to 95% lower than every other format tested.
2. **YAML** achieves the highest accuracy (77.42% mandatory, 79.03% optional) and dominates field retrieval (97 to 99%). However its output token consumption is the highest (11,104 to 12,896) which erodes its efficiency score and pushes it to mid-range in overall rankings.
3. Most errors are minor character level deviations and not completely wrong answers. Accuracy by character ranges from 90% to 98% across all formats while accuracy by answer ranges from 70% to 79%. This 15 to 20 percentage point gap indicates that the model frequently produces nearly correct answers with small mistakes in specific characters.
4. Pretty-printed formats (**JSON_PRETTY**, **XML_PRETTY**) consistently rank worst for token efficiency. **XML_PRETTY** consumes 95% more read tokens than **JSON_COMPACT** on mandatory data while delivering lower accuracy. The whitespace and indentation that aids human readability translates directly into wasted tokens without improving model comprehension.
5. Aggregation is the weakest category for all formats and degrades further on optional data. Every format shows a decline in aggregation accuracy when switching from mandatory to optional data (up to 22 percentage points for **TOON_DEFAULT**). Sparse data structures make numerical computations harder for the model regardless of format.
6. **XML_COMPACT** shows the most stable output token behavior across variants. While **TOON_DEFAULT** output tokens spike by 40% and **YAML** by 16% on optional data **XML_COMPACT** actually decreases by 18% which makes it the most predictable format for inference cost planning.
7. **TOON_DEFAULT** exhibits the largest accuracy swing between variants (+5.38%). This is driven by a +22.22 percentage point jump in structure awareness on optional data which suggests the format becomes easier to parse when data is sparser. This also means **TOON_DEFAULT** is the least predictable format for accuracy.

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
Composite metric balancing accuracy with normalized token count (favor towards accuracy). Each efficiency score has an indicator which token count was used in the calculation.
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
However these values cannot be exactly applied to models of the same family or from other providers as token usage, accuracy and latency depend on specific model architectures and tokenizers. Also file reads will produce different characters depending on the used harness because some add marker characters, line numbers or additional information. While the relative ranking of file formats remains consistent the absolute numbers will vary.
Especially the accuracy and output tokens results will vary because these values are bound to the model size and training, instruction interpretation and reasoning token budget.

## 2. Results

### 2.1 TLDR: Token Efficiency Analysis

*Note: All columns ranked best-to-worst. ↑ = lower value is better (ascending). ↓ = higher value is better (descending).*

#### 2.1.1 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT ≈ 69s | JSON_COMPACT ≈ 10315 | JSON_COMPACT ≈ 336 | XML_PRETTY ≈ 7175 | XML_PRETTY ≈ 7518 | JSON_COMPACT ≈ 18446 | YAML ≈ 77.42% | JSON_COMPACT ≈ 81 | XML_PRETTY ≈ 81 | JSON_COMPACT ≈ 82 | JSON_PRETTY ≈ 96.67% | JSON_COMPACT ≈ 92 | XML_PRETTY ≈ 95 | JSON_COMPACT ≈ 93 |
| XML_PRETTY (+2.18%) | XML_COMPACT (+24.56%) | YAML (+0.50%) | JSON_COMPACT (+8.64%) | JSON_COMPACT (+8.16%) | XML_COMPACT (+18.26%) | JSON_PRETTY (-1.88%) | XML_COMPACT (-10.73%) | JSON_COMPACT (-1.51%) | XML_COMPACT (-13.01%) | YAML (-0.40%) | XML_COMPACT (-7.23%) | TOON_DEFAULT (-5.10%) | XML_COMPACT (-9.29%) |
| TOON_DEFAULT (+6.95%) | TOON_DEFAULT (+36.66%) | TOON_DEFAULT (+1.74%) | TOON_DEFAULT (+16.27%) | TOON_DEFAULT (+15.52%) | TOON_DEFAULT (+23.50%) | JSON_COMPACT (-3.23%) | YAML (-13.21%) | TOON_DEFAULT (-5.70%) | TOON_DEFAULT (-17.11%) | TOON_DEFAULT (-2.16%) | YAML (-9.88%) | JSON_COMPACT (-5.32%) | TOON_DEFAULT (-11.25%) |
| XML_COMPACT (+8.00%) | YAML (+38.69%) | XML_PRETTY (+1.88%) | XML_COMPACT (+20.16%) | XML_COMPACT (+19.27%) | YAML (+37.75%) | XML_COMPACT (-4.03%) | TOON_DEFAULT (-16.24%) | XML_COMPACT (-6.64%) | YAML (-22.94%) | XML_PRETTY (-2.41%) | TOON_DEFAULT (-10.42%) | XML_COMPACT (-7.52%) | YAML (-18.51%) |
| YAML (+31.78%) | JSON_PRETTY (+72.84%) | XML_COMPACT (+2.58%) | JSON_PRETTY (+43.88%) | JSON_PRETTY (+43.27%) | XML_PRETTY (+49.79%) | TOON_DEFAULT (-4.71%) | JSON_PRETTY (-28.75%) | JSON_PRETTY (-14.53%) | XML_PRETTY (-35.45%) | XML_COMPACT (-3.79%) | JSON_PRETTY (-21.92%) | JSON_PRETTY (-13.03%) | XML_PRETTY (-27.14%) |
| JSON_PRETTY (+34.89%) | XML_PRETTY (+95.00%) | JSON_PRETTY (+33.04%) | YAML (+50.05%) | YAML (+47.71%) | JSON_PRETTY (+55.04%) | XML_PRETTY (-5.38%) | XML_PRETTY (-40.71%) | YAML (-14.76%) | JSON_PRETTY (-36.16%) | JSON_COMPACT (-6.05%) | XML_PRETTY (-31.67%) | YAML (-14.82%) | JSON_PRETTY (-28.55%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XML_COMPACT ≈ 64s | JSON_COMPACT ≈ 9788 | JSON_PRETTY ≈ 228 | XML_COMPACT ≈ 7011 | XML_COMPACT ≈ 7356 | JSON_COMPACT ≈ 18250 | YAML ≈ 79.03% | JSON_COMPACT ≈ 84 | XML_COMPACT ≈ 83 | JSON_COMPACT ≈ 84 | YAML ≈ 98.14% | JSON_COMPACT ≈ 97 | XML_COMPACT ≈ 96 | JSON_COMPACT ≈ 97 |
| XML_PRETTY (+21.07%) | XML_COMPACT (+26.36%) | JSON_COMPACT (+47.44%) | JSON_COMPACT (+15.90%) | JSON_COMPACT (+15.04%) | XML_COMPACT (+8.07%) | TOON_DEFAULT (-0.94%) | XML_COMPACT (-11.34%) | JSON_COMPACT (-4.24%) | XML_COMPACT (-6.75%) | TOON_DEFAULT (-1.43%) | XML_COMPACT (-9.07%) | JSON_COMPACT (-4.48%) | XML_COMPACT (-5.07%) |
| JSON_PRETTY (+31.75%) | TOON_DEFAULT (+41.59%) | YAML (+47.58%) | XML_PRETTY (+33.45%) | JSON_PRETTY (+31.31%) | TOON_DEFAULT (+42.74%) | JSON_COMPACT (-2.42%) | YAML (-14.37%) | XML_PRETTY (-13.69%) | TOON_DEFAULT (-26.71%) | JSON_COMPACT (-2.93%) | YAML (-12.18%) | JSON_PRETTY (-10.58%) | TOON_DEFAULT (-23.27%) |
| JSON_COMPACT (+37.52%) | YAML (+43.57%) | XML_PRETTY (+51.10%) | JSON_PRETTY (+34.52%) | XML_PRETTY (+31.88%) | JSON_PRETTY (+45.52%) | XML_COMPACT (-4.30%) | TOON_DEFAULT (-14.38%) | JSON_PRETTY (-14.98%) | YAML (-29.18%) | XML_COMPACT (-3.63%) | TOON_DEFAULT (-12.52%) | XML_PRETTY (-11.08%) | YAML (-25.09%) |
| YAML (+47.75%) | JSON_PRETTY (+72.65%) | XML_COMPACT (+51.24%) | TOON_DEFAULT (+68.86%) | TOON_DEFAULT (+65.74%) | YAML (+47.67%) | XML_PRETTY (-6.18%) | JSON_PRETTY (-31.61%) | TOON_DEFAULT (-22.42%) | JSON_PRETTY (-34.15%) | JSON_PRETTY (-4.00%) | JSON_PRETTY (-24.41%) | TOON_DEFAULT (-20.16%) | JSON_PRETTY (-26.63%) |
| TOON_DEFAULT (+51.18%) | XML_PRETTY (+100.07%) | TOON_DEFAULT (+54.50%) | YAML (+79.14%) | YAML (+75.33%) | XML_PRETTY (+60.46%) | JSON_PRETTY (-8.06%) | XML_PRETTY (-40.37%) | YAML (-25.33%) | XML_PRETTY (-42.41%) | XML_PRETTY (-4.45%) | XML_PRETTY (-33.66%) | YAML (-22.33%) | XML_PRETTY (-35.43%) |


#### 2.1.2 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 97.58% | JSON_PRETTY ≈ 65.43% | JSON_COMPACT ≈ 66.67% | JSON_COMPACT ≈ 71.43% |
| JSON_PRETTY (-1.21%) | XML_COMPACT (-2.47%) | XML_COMPACT (0.00%) | TOON_DEFAULT (-5.56%) |
| TOON_DEFAULT (-6.36%) | XML_PRETTY (-2.47%) | JSON_PRETTY (-6.35%) | YAML (-11.11%) |
| XML_PRETTY (-9.09%) | YAML (-2.47%) | YAML (-6.35%) | XML_PRETTY (-12.69%) |
| XML_COMPACT (-9.70%) | JSON_COMPACT (-4.94%) | TOON_DEFAULT (-7.14%) | XML_COMPACT (-15.87%) |
| JSON_COMPACT (-12.73%) | TOON_DEFAULT (-14.82%) | XML_PRETTY (-12.70%) | JSON_PRETTY (-22.22%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99.39% | XML_COMPACT ≈ 72.84% | TOON_DEFAULT ≈ 71.43% | YAML ≈ 55.56% |
| TOON_DEFAULT (-3.03%) | TOON_DEFAULT (0.00%) | XML_PRETTY (-7.94%) | XML_COMPACT (0.00%) |
| JSON_COMPACT (-5.45%) | JSON_COMPACT (-1.23%) | YAML (-7.94%) | JSON_COMPACT (-3.18%) |
| JSON_PRETTY (-9.09%) | YAML (-4.94%) | XML_COMPACT (-7.94%) | XML_PRETTY (-3.18%) |
| XML_PRETTY (-10.30%) | XML_PRETTY (-9.87%) | JSON_COMPACT (-9.52%) | JSON_PRETTY (-9.52%) |
| XML_COMPACT (-12.12%) | JSON_PRETTY (-14.82%) | JSON_PRETTY (-9.53%) | TOON_DEFAULT (-11.90%) |


#### 2.1.3 Category Accuracy By Character Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 97.24% | JSON_PRETTY ≈ 97.45% | JSON_COMPACT ≈ 93.75% | TOON_DEFAULT ≈ 85.49% |
| JSON_PRETTY (-0.35%) | JSON_COMPACT (-0.44%) | XML_COMPACT (-0.89%) | JSON_COMPACT (-2.37%) |
| TOON_DEFAULT (-3.55%) | YAML (-1.18%) | TOON_DEFAULT (-2.23%) | YAML (-3.81%) |
| XML_PRETTY (-4.52%) | XML_PRETTY (-1.45%) | YAML (-2.98%) | XML_PRETTY (-4.63%) |
| XML_COMPACT (-6.81%) | TOON_DEFAULT (-1.85%) | JSON_PRETTY (-4.76%) | XML_COMPACT (-5.86%) |
| JSON_COMPACT (-14.85%) | XML_COMPACT (-2.25%) | XML_PRETTY (-5.95%) | JSON_PRETTY (-8.13%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99.82% | TOON_DEFAULT ≈ 98.02% | TOON_DEFAULT ≈ 92.93% | JSON_COMPACT ≈ 79.39% |
| JSON_PRETTY (-3.17%) | YAML (-0.42%) | YAML (-0.59%) | YAML (-0.46%) |
| TOON_DEFAULT (-3.28%) | XML_COMPACT (-0.65%) | JSON_COMPACT (-0.89%) | XML_PRETTY (-1.62%) |
| JSON_COMPACT (-4.42%) | XML_PRETTY (-1.73%) | XML_PRETTY (-2.36%) | XML_COMPACT (-2.42%) |
| XML_COMPACT (-7.62%) | JSON_COMPACT (-2.22%) | XML_COMPACT (-4.13%) | JSON_PRETTY (-6.26%) |
| XML_PRETTY (-8.21%) | JSON_PRETTY (-4.87%) | JSON_PRETTY (-9.15%) | TOON_DEFAULT (-8.08%) |


#### 2.1.4 Conclusion

The rankings reveal a consistent tradeoff between accuracy and token cost that no single format resolves perfectly.

- **JSON_COMPACT** is the recommended default for nested data because it leads in total efficiency score for both mandatory (82.17) and optional (84.38) data. It achieves this through the lowest read token count (9,788 to 10,315) and moderate output tokens which it combines with mid-range accuracy. Its characters-per-read-token ratio of 2.2 shows that the tokenizer encodes compact **JSON** more densely than any format except **XML_COMPACT** (2.52). The efficiency lead holds across all three token-based efficiency scores (read, output, total).
- **YAML** is the best choice when accuracy is the primary concern and token budget is secondary. It ranks first in accuracy for both variants and achieves the highest accuracy by character overall (96.27% to 98.14%). **YAML** excels at field retrieval (97 to 99%) because its key-value structure maps directly to how the model retrieves individual data points. The cost is a 38% to 48% increase in total tokens compared to **JSON_COMPACT** which is driven almost entirely by higher output tokens during inference.
- **XML_COMPACT** occupies a useful middle ground. It ranks second in total tokens for optional data (only 8% behind **JSON_COMPACT**) and shows the most stable output token behavior between variants. For workloads where predictable inference cost matters more than peak accuracy **XML_COMPACT** is worth considering. Its high characters-per-read-token ratio (2.52) makes it the most structurally dense format at the read stage.
- Pretty-printed formats offer no measurable benefit for model consumption. **JSON_PRETTY** and **XML_PRETTY** consistently rank last in efficiency metrics. **XML_PRETTY** uses nearly double the read tokens of **JSON_COMPACT** (20,114 vs 10,315 on mandatory data) without compensating through better accuracy. **JSON_PRETTY** achieves the highest accuracy by character on mandatory data (96.67%) but the difference over **YAML** (96.27%) does not justify the 72% read token premium. The indentation and whitespace that makes these formats human-readable only adds overhead for the model.
- The accuracy by character view significantly narrows the gap between formats. When measured by complete answer correctness the spread from worst to best is 8 percentage points (70.97% to 79.03% on optional). When measured by character the spread compresses to under 5 percentage points (93.69% to 98.14%). This means the format choice affects whether the model gets answers exactly right but most formats produce answers that are close to correct. In applications that tolerate minor output variations (like approximate lookups or fuzzy matching) the format choice matters less than these accuracy numbers suggest.
- Aggregation remains a model bottleneck that format choice cannot solve. The best aggregation accuracy on optional data is 55.56% (**YAML**) and no format exceeds 71.43% even on mandatory. This is a reasoning limitation of the model and not a format parsing issue.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total | Accuracy By Character (%) | Useful Read Tokens (Acc By Char) | Wasted Read Tokens (Acc By Char) | Useful Output Tokens (Acc By Char) | Wasted Output Tokens (Acc By Char) | Eff Score Read (Acc By Char) | Eff Score Output (Acc By Char) | Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10315 | 8131 | 18446 | 2.21 | 62.87 | 74.19 | 7653 | 2662 | 6033 | 2099 | 81.06 | 79.40 | 82.17 | 90.62 | 9347 | 968 | 7369 | 763 | 92.02 | 90.35 | 93.12 |
| JSON_COMPACT | opt | 9788 | 8462 | 18250 | 2.19 | 65.54 | 76.61 | 7499 | 2289 | 6483 | 1979 | 84.37 | 79.59 | 84.38 | 95.21 | 9319 | 469 | 8057 | 405 | 96.77 | 91.99 | 96.78 |
| JSON_PRETTY | man | 17828 | 10771 | 28599 | 1.76 | 83.26 | 75.54 | 13467 | 4361 | 8136 | 2635 | 57.76 | 68.91 | 52.46 | 96.67 | 17234 | 594 | 10412 | 359 | 71.84 | 82.99 | 66.54 |
| JSON_PRETTY | opt | 16899 | 9659 | 26558 | 1.75 | 76.06 | 70.97 | 11993 | 4906 | 6855 | 2804 | 57.70 | 70.66 | 55.56 | 94.14 | 15909 | 990 | 9093 | 566 | 73.15 | 86.11 | 71.01 |
| TOON_DEFAULT | man | 14096 | 8685 | 22781 | 1.85 | 67.28 | 72.71 | 10249 | 3847 | 6315 | 2370 | 67.89 | 76.03 | 68.11 | 94.51 | 13322 | 774 | 8208 | 477 | 82.43 | 90.56 | 82.65 |
| TOON_DEFAULT | opt | 13859 | 12191 | 26050 | 1.86 | 95.48 | 78.09 | 10822 | 3037 | 9520 | 2671 | 72.24 | 64.47 | 61.84 | 96.71 | 13403 | 456 | 11790 | 401 | 84.66 | 76.89 | 74.25 |
| XML_COMPACT | man | 12848 | 8966 | 21814 | 2.52 | 69.53 | 73.39 | 9429 | 3419 | 6580 | 2386 | 72.37 | 75.26 | 71.48 | 92.88 | 11933 | 915 | 8328 | 638 | 85.36 | 88.26 | 84.47 |
| XML_COMPACT | opt | 12368 | 7356 | 19724 | 2.52 | 56.54 | 74.73 | 9243 | 3125 | 5497 | 1859 | 74.81 | 83.11 | 78.68 | 94.51 | 11689 | 679 | 6952 | 404 | 88.00 | 96.30 | 91.87 |
| XML_PRETTY | man | 20114 | 7518 | 27632 | 1.99 | 57.87 | 72.04 | 14490 | 5624 | 5416 | 2102 | 48.06 | 80.62 | 53.04 | 94.26 | 18959 | 1155 | 7086 | 432 | 62.87 | 95.43 | 67.85 |
| XML_PRETTY | opt | 19583 | 9701 | 29284 | 1.98 | 75.46 | 72.85 | 14266 | 5317 | 7067 | 2634 | 50.31 | 71.73 | 48.60 | 93.69 | 18347 | 1236 | 9088 | 612 | 64.20 | 85.63 | 62.49 |
| YAML | man | 14306 | 11104 | 25410 | 1.79 | 86.83 | 77.42 | 11076 | 3230 | 8597 | 2507 | 70.36 | 68.72 | 63.32 | 96.27 | 13772 | 534 | 10690 | 414 | 82.92 | 81.29 | 75.89 |
| YAML | opt | 14053 | 12896 | 26949 | 1.80 | 101.29 | 79.03 | 11106 | 2947 | 10192 | 2704 | 72.25 | 62.06 | 59.76 | 98.14 | 13792 | 261 | 12656 | 240 | 84.99 | 74.80 | 72.50 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10315 | 9788 | -527 | -5.11 | 336 | 336 | 0 | 0.00 | 7795 | 8126 | +331 | +4.25 | 8131 | 8462 | +331 | +4.07 | 18446 | 18250 | -196 | -1.06 |
| JSON_PRETTY | 17828 | 16899 | -929 | -5.21 | 447 | 228 | -219 | -48.99 | 10324 | 9432 | -892 | -8.64 | 10771 | 9659 | -1112 | -10.32 | 28599 | 26558 | -2041 | -7.14 |
| TOON_DEFAULT | 14096 | 13859 | -237 | -1.68 | 342 | 352 | +10 | +2.92 | 8343 | 11840 | +3497 | +41.92 | 8685 | 12191 | +3506 | +40.37 | 22781 | 26050 | +3269 | +14.35 |
| XML_COMPACT | 12848 | 12368 | -480 | -3.74 | 345 | 345 | 0 | 0.00 | 8622 | 7012 | -1610 | -18.67 | 8966 | 7355 | -1611 | -17.97 | 21814 | 19723 | -2091 | -9.59 |
| XML_PRETTY | 20114 | 19583 | -531 | -2.64 | 342 | 344 | +2 | +0.58 | 7175 | 9356 | +2181 | +30.40 | 7518 | 9701 | +2183 | +29.04 | 27632 | 29284 | +1652 | +5.98 |
| YAML | 14306 | 14053 | -253 | -1.77 | 338 | 336 | -2 | -0.59 | 10767 | 12561 | +1794 | +16.66 | 11104 | 12896 | +1792 | +16.14 | 25410 | 26949 | +1539 | +6.06 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Before Write (ms) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) | Output (ms) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 24 | 429.79 | 0.77 | 31018 | 38410 | 0.20 | 309.76 | 38434 | 429.99 | 247.96 | 69429 |
| JSON_COMPACT | opt | 9 | 1087.56 | 0.29 | 48428 | 39495 | 0.21 | 318.51 | 39504 | 1087.76 | 254.86 | 87923 |
| JSON_PRETTY | man | 267 | 66.77 | 8.61 | 57029 | 36625 | 0.28 | 295.36 | 36892 | 67.05 | 238.01 | 93654 |
| JSON_PRETTY | opt | 272 | 62.13 | 8.77 | 47729 | 36507 | 0.26 | 294.41 | 36779 | 62.39 | 237.28 | 84236 |
| TOON_DEFAULT | man | 10 | 1409.60 | 0.32 | 35101 | 39156 | 0.22 | 315.77 | 39166 | 1409.82 | 252.68 | 74257 |
| TOON_DEFAULT | opt | 20 | 692.95 | 0.65 | 57218 | 39438 | 0.31 | 318.05 | 39458 | 693.26 | 254.57 | 96657 |
| XML_COMPACT | man | 11 | 1168.00 | 0.35 | 33261 | 41724 | 0.21 | 336.48 | 41735 | 1168.21 | 269.26 | 74985 |
| XML_COMPACT | opt | 9 | 1374.22 | 0.29 | 24657 | 39280 | 0.18 | 316.77 | 39289 | 1374.40 | 253.48 | 63937 |
| XML_PRETTY | man | 11 | 1828.55 | 0.35 | 23314 | 47627 | 0.15 | 384.09 | 47638 | 1828.70 | 307.34 | 70941 |
| XML_PRETTY | opt | 11 | 1780.27 | 0.35 | 42231 | 35179 | 0.27 | 283.70 | 35190 | 1780.54 | 227.03 | 77410 |
| YAML | man | 12 | 1192.17 | 0.39 | 54849 | 36643 | 0.29 | 295.51 | 36655 | 1192.46 | 236.48 | 91492 |
| YAML | opt | 8 | 1756.63 | 0.26 | 61846 | 32619 | 0.39 | 263.06 | 32627 | 1757.01 | 210.50 | 94465 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Before Write Man (s) | Output Before Write Opt (s) | Diff (s) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 24 | 9 | -15 | -62.50 | 31.02 | 48.43 | +17.41 | +56.13 | 38.41 | 39.50 | +1.09 | +2.82 | 38.43 | 39.50 | +1.07 | +2.78 | 69.43 | 87.92 | +18.49 | +26.64 |
| JSON_PRETTY | 267 | 272 | +5 | +1.87 | 57.03 | 47.73 | -9.30 | -16.31 | 36.63 | 36.51 | -0.12 | -0.32 | 36.89 | 36.78 | -0.11 | -0.31 | 93.65 | 84.24 | -9.42 | -10.06 |
| TOON_DEFAULT | 10 | 20 | +10 | +100.00 | 35.10 | 57.22 | +22.12 | +63.01 | 39.16 | 39.44 | +0.28 | +0.72 | 39.17 | 39.46 | +0.29 | +0.75 | 74.26 | 96.66 | +22.40 | +30.17 |
| XML_COMPACT | 11 | 9 | -2 | -18.18 | 33.26 | 24.66 | -8.60 | -25.87 | 41.72 | 39.28 | -2.44 | -5.86 | 41.74 | 39.29 | -2.45 | -5.86 | 74.99 | 63.94 | -11.05 | -14.73 |
| XML_PRETTY | 11 | 11 | 0 | 0.00 | 23.31 | 42.23 | +18.92 | +81.14 | 47.63 | 35.18 | -12.45 | -26.14 | 47.64 | 35.19 | -12.45 | -26.13 | 70.94 | 77.41 | +6.47 | +9.12 |
| YAML | 12 | 8 | -4 | -33.33 | 54.85 | 61.85 | +7.00 | +12.76 | 36.64 | 32.62 | -4.02 | -10.98 | 36.66 | 32.63 | -4.03 | -10.99 | 91.49 | 94.47 | +2.97 | +3.25 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token | Info / Read Token (Acc By Char) | Info / Output Token (Acc By Char) | Info / Total Token (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 2.21 | 15.13 | 332.74 | 0.72 | 0.91 | 0.40 | 0.88 | 1.11 | 0.49 |
| JSON_COMPACT | opt | 2.19 | 15.51 | 315.74 | 0.78 | 0.91 | 0.42 | 0.97 | 1.13 | 0.52 |
| JSON_PRETTY | man | 1.76 | 26.14 | 575.10 | 0.42 | 0.70 | 0.26 | 0.54 | 0.90 | 0.34 |
| JSON_PRETTY | opt | 1.75 | 26.78 | 545.13 | 0.42 | 0.74 | 0.27 | 0.56 | 0.98 | 0.35 |
| TOON_DEFAULT | man | 1.85 | 20.67 | 454.71 | 0.52 | 0.84 | 0.32 | 0.67 | 1.09 | 0.42 |
| TOON_DEFAULT | opt | 1.86 | 21.96 | 447.07 | 0.56 | 0.68 | 0.30 | 0.70 | 0.84 | 0.38 |
| XML_COMPACT | man | 2.52 | 18.84 | 414.45 | 0.57 | 0.82 | 0.34 | 0.72 | 1.04 | 0.43 |
| XML_COMPACT | opt | 2.52 | 19.60 | 398.97 | 0.60 | 1.02 | 0.38 | 0.76 | 1.29 | 0.48 |
| XML_PRETTY | man | 1.99 | 29.49 | 648.84 | 0.36 | 0.96 | 0.26 | 0.47 | 1.25 | 0.34 |
| XML_PRETTY | opt | 1.98 | 31.04 | 631.71 | 0.37 | 0.75 | 0.25 | 0.48 | 0.97 | 0.32 |
| YAML | man | 1.79 | 20.98 | 461.48 | 0.54 | 0.70 | 0.31 | 0.67 | 0.87 | 0.38 |
| YAML | opt | 1.80 | 22.27 | 453.32 | 0.56 | 0.61 | 0.29 | 0.70 | 0.76 | 0.36 |

#### 2.5.2 Characters And Values: Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 2.21 | 2.19 | -0.03 | -1.22 | 15.13 | 15.51 | +0.39 | +2.56 | 332.74 | 315.74 | -17.00 | -5.11 |
| JSON_PRETTY | 1.76 | 1.75 | -0.01 | -0.57 | 26.14 | 26.78 | +0.64 | +2.45 | 575.10 | 545.13 | -29.97 | -5.21 |
| TOON_DEFAULT | 1.85 | 1.86 | +0.01 | +0.49 | 20.67 | 21.96 | +1.29 | +6.27 | 454.71 | 447.07 | -7.64 | -1.68 |
| XML_COMPACT | 2.52 | 2.52 | 0.00 | 0.00 | 18.84 | 19.60 | +0.76 | +4.04 | 414.45 | 398.97 | -15.48 | -3.74 |
| XML_PRETTY | 1.99 | 1.98 | -0.01 | -0.55 | 29.49 | 31.04 | +1.54 | +5.23 | 648.84 | 631.71 | -17.13 | -2.64 |
| YAML | 1.79 | 1.80 | +0.01 | +0.56 | 20.98 | 22.27 | +1.29 | +6.17 | 461.48 | 453.32 | -8.16 | -1.77 |

#### 2.5.3 Information: Mandatory vs Optional
| Format | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 0.72 | 0.78 | +0.06 | +8.90 | 0.91 | 0.91 | -0.01 | -0.77 | 0.40 | 0.42 | +0.02 | +4.48 |
| JSON_PRETTY | 0.42 | 0.42 | 0.00 | 0.00 | 0.70 | 0.74 | +0.03 | +4.85 | 0.26 | 0.27 | 0.00 | 0.00 |
| TOON_DEFAULT | 0.52 | 0.56 | +0.05 | +9.11 | 0.84 | 0.68 | -0.16 | -19.09 | 0.32 | 0.30 | -0.02 | -4.86 |
| XML_COMPACT | 0.57 | 0.60 | +0.03 | +5.78 | 0.82 | 1.02 | +0.20 | +24.05 | 0.34 | 0.38 | +0.04 | +12.80 |
| XML_PRETTY | 0.36 | 0.37 | +0.01 | +3.91 | 0.96 | 0.75 | -0.21 | -21.61 | 0.26 | 0.25 | -0.01 | -4.60 |
| YAML | 0.54 | 0.56 | +0.02 | +3.88 | 0.70 | 0.61 | -0.08 | -12.05 | 0.31 | 0.29 | -0.01 | -3.93 |

#### 2.5.4 Information (Accuracy By Character): Mandatory vs Optional
| Format | Info / Read Token (Acc By Char) Man | Info / Read Token (Acc By Char) Opt | Diff | Diff (%) | Info / Output Token (Acc By Char) Man | Info / Output Token (Acc By Char)  Opt | Diff | Diff (%) | Info / Total Token (Acc By Char) Man | Info / Total Token (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 0.88 | 0.97 | +0.09 | +10.69 | 1.11 | 1.13 | +0.01 | +0.99 | 0.49 | 0.52 | +0.03 | +6.31 |
| JSON_PRETTY | 0.54 | 0.56 | +0.02 | +2.77 | 0.90 | 0.98 | +0.08 | +8.57 | 0.34 | 0.35 | +0.02 | +4.73 |
| TOON_DEFAULT | 0.67 | 0.70 | +0.03 | +4.18 | 1.09 | 0.84 | -0.25 | -22.95 | 0.42 | 0.38 | -0.04 | -9.52 |
| XML_COMPACT | 0.72 | 0.76 | +0.04 | +5.67 | 1.04 | 1.29 | +0.25 | +24.03 | 0.43 | 0.48 | +0.05 | +12.44 |
| XML_PRETTY | 0.47 | 0.48 | +0.01 | +1.92 | 1.25 | 0.97 | -0.29 | -22.97 | 0.34 | 0.32 | -0.02 | -6.16 |
| YAML | 0.67 | 0.70 | +0.02 | +3.71 | 0.87 | 0.76 | -0.11 | -12.23 | 0.38 | 0.36 | -0.02 | -3.96 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10315 | 7653 | 2662 | 8131 | 6033 | 2099 | 18446 | 13685 | 4761 | 74.19 | 81.06 | 79.40 | 82.17 | 72.28 | 79.79 | 78.13 | 80.90 |
| JSON_COMPACT | opt | 9788 | 7499 | 2289 | 8462 | 6483 | 1979 | 18250 | 13981 | 4269 | 76.61 | 84.37 | 79.59 | 84.38 | 75.55 | 83.67 | 78.88 | 83.67 |
| JSON_PRETTY | man | 17828 | 13467 | 4361 | 10771 | 8136 | 2635 | 28599 | 21603 | 6995 | 75.54 | 57.76 | 68.91 | 52.46 | 73.94 | 56.69 | 67.84 | 51.39 |
| JSON_PRETTY | opt | 16899 | 11993 | 4906 | 9659 | 6855 | 2804 | 26558 | 18848 | 7710 | 70.97 | 57.70 | 70.66 | 55.56 | 69.43 | 56.68 | 69.63 | 54.54 |
| TOON_DEFAULT | man | 14096 | 10249 | 3847 | 8685 | 6315 | 2370 | 22781 | 16564 | 6217 | 72.71 | 67.89 | 76.03 | 68.11 | 69.60 | 65.82 | 73.95 | 66.04 |
| TOON_DEFAULT | opt | 13859 | 10822 | 3037 | 12191 | 9520 | 2671 | 26050 | 20343 | 5708 | 78.09 | 72.24 | 64.47 | 61.84 | 77.72 | 72.00 | 64.23 | 61.59 |
| XML_COMPACT | man | 12848 | 9429 | 3419 | 8966 | 6580 | 2386 | 21814 | 16010 | 5805 | 73.39 | 72.37 | 75.26 | 71.48 | 72.15 | 71.54 | 74.44 | 70.65 |
| XML_COMPACT | opt | 12368 | 9243 | 3125 | 7356 | 5497 | 1859 | 19724 | 14739 | 4984 | 74.73 | 74.81 | 83.11 | 78.68 | 74.15 | 74.42 | 82.72 | 78.29 |
| XML_PRETTY | man | 20114 | 14490 | 5624 | 7518 | 5416 | 2102 | 27632 | 19906 | 7726 | 72.04 | 48.06 | 80.62 | 53.04 | 70.13 | 46.79 | 79.34 | 51.76 |
| XML_PRETTY | opt | 19583 | 14266 | 5317 | 9701 | 7067 | 2634 | 29284 | 21333 | 7950 | 72.85 | 50.31 | 71.73 | 48.60 | 71.55 | 49.44 | 70.87 | 47.73 |
| YAML | man | 14306 | 11076 | 3230 | 11104 | 8597 | 2507 | 25410 | 19673 | 5738 | 77.42 | 70.36 | 68.72 | 63.32 | 75.06 | 68.78 | 67.15 | 61.75 |
| YAML | opt | 14053 | 11106 | 2947 | 12896 | 10192 | 2704 | 26949 | 21298 | 5651 | 79.03 | 72.25 | 62.06 | 59.76 | 77.25 | 71.06 | 60.87 | 58.57 |

#### 2.6.2 Read Tokens: Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10315 | 9788 | -527 | -5.11 | 7653 | 7499 | -154 | -2.01 | 2662 | 2289 | -373 | -14.01 | 74.19 | 76.61 | +2.42 | 81.06 | 84.37 | +3.31 | +4.09 | 72.28 | 75.55 | +3.27 | 79.79 | 83.67 | +3.88 | +4.86 |
| JSON_PRETTY | 17828 | 16899 | -929 | -5.21 | 13467 | 11993 | -1474 | -10.95 | 4361 | 4906 | +545 | +12.50 | 75.54 | 70.97 | -4.57 | 57.76 | 57.70 | -0.05 | -0.09 | 73.94 | 69.43 | -4.51 | 56.69 | 56.68 | -0.01 | -0.02 |
| TOON_DEFAULT | 14096 | 13859 | -237 | -1.68 | 10249 | 10822 | +573 | +5.59 | 3847 | 3037 | -810 | -21.06 | 72.71 | 78.09 | +5.38 | 67.89 | 72.24 | +4.35 | +6.41 | 69.60 | 77.72 | +8.12 | 65.82 | 72.00 | +6.18 | +9.38 |
| XML_COMPACT | 12848 | 12368 | -480 | -3.74 | 9429 | 9242 | -187 | -1.98 | 3419 | 3126 | -293 | -8.58 | 73.39 | 74.73 | +1.34 | 72.37 | 74.81 | +2.44 | +3.37 | 72.15 | 74.15 | +2.00 | 71.54 | 74.42 | +2.88 | +4.02 |
| XML_PRETTY | 20114 | 19583 | -531 | -2.64 | 14490 | 14266 | -224 | -1.55 | 5624 | 5317 | -307 | -5.46 | 72.04 | 72.85 | +0.81 | 48.06 | 50.31 | +2.25 | +4.68 | 70.13 | 71.55 | +1.42 | 46.79 | 49.44 | +2.66 | +5.68 |
| YAML | 14306 | 14053 | -253 | -1.77 | 11076 | 11106 | +30 | +0.27 | 3230 | 2947 | -283 | -8.77 | 77.42 | 79.03 | +1.61 | 70.36 | 72.25 | +1.89 | +2.68 | 75.06 | 77.25 | +2.19 | 68.78 | 71.06 | +2.27 | +3.31 |

#### 2.6.3 Output Tokens: Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 8131 | 8462 | +331 | +4.07 | 6033 | 6483 | +450 | +7.46 | 2099 | 1980 | -119 | -5.69 | 74.19 | 76.61 | +2.42 | 79.40 | 79.59 | +0.19 | +0.23 | 72.28 | 75.55 | +3.27 | 78.13 | 78.88 | +0.75 | +0.96 |
| JSON_PRETTY | 10771 | 9659 | -1112 | -10.32 | 8136 | 6855 | -1281 | -15.75 | 2635 | 2805 | +170 | +6.43 | 75.54 | 70.97 | -4.57 | 68.91 | 70.66 | +1.75 | +2.54 | 73.94 | 69.43 | -4.51 | 67.84 | 69.63 | +1.79 | +2.64 |
| TOON_DEFAULT | 8685 | 12191 | +3506 | +40.37 | 6315 | 9520 | +3205 | +50.76 | 2370 | 2671 | +301 | +12.70 | 72.71 | 78.09 | +5.38 | 76.03 | 64.47 | -11.55 | -15.19 | 69.60 | 77.72 | +8.12 | 73.95 | 64.23 | -9.72 | -13.15 |
| XML_COMPACT | 8966 | 7355 | -1611 | -17.96 | 6580 | 5496 | -1084 | -16.47 | 2386 | 1859 | -527 | -22.09 | 73.39 | 74.73 | +1.34 | 75.26 | 83.11 | +7.85 | +10.43 | 72.15 | 74.15 | +2.00 | 74.44 | 82.72 | +8.29 | +11.13 |
| XML_PRETTY | 7518 | 9701 | +2183 | +29.03 | 5416 | 7067 | +1651 | +30.49 | 2102 | 2634 | +532 | +25.30 | 72.04 | 72.85 | +0.81 | 80.62 | 71.73 | -8.88 | -11.02 | 70.13 | 71.55 | +1.42 | 79.34 | 70.87 | -8.48 | -10.68 |
| YAML | 11104 | 12896 | +1792 | +16.14 | 8597 | 10192 | +1595 | +18.55 | 2507 | 2704 | +197 | +7.86 | 77.42 | 79.03 | +1.61 | 68.72 | 62.06 | -6.66 | -9.70 | 75.06 | 77.25 | +2.19 | 67.15 | 60.87 | -6.28 | -9.35 |

#### 2.6.4 Total Tokens: Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 18446 | 18250 | -196 | -1.06 | 13685 | 13981 | +296 | +2.16 | 4761 | 4269 | -492 | -10.34 | 74.19 | 76.61 | +2.42 | 82.17 | 84.38 | +2.21 | +2.68 | 72.28 | 75.55 | +3.27 | 80.90 | 83.67 | +2.77 | +3.43 |
| JSON_PRETTY | 28599 | 26558 | -2041 | -7.14 | 21603 | 18848 | -2755 | -12.75 | 6995 | 7710 | +715 | +10.22 | 75.54 | 70.97 | -4.57 | 52.46 | 55.56 | +3.11 | +5.92 | 73.94 | 69.43 | -4.51 | 51.39 | 54.54 | +3.15 | +6.12 |
| TOON_DEFAULT | 22781 | 26050 | +3269 | +14.35 | 16564 | 20343 | +3779 | +22.81 | 6217 | 5708 | -509 | -8.19 | 72.71 | 78.09 | +5.38 | 68.11 | 61.84 | -6.27 | -9.21 | 69.60 | 77.72 | +8.12 | 66.04 | 61.59 | -4.45 | -6.73 |
| XML_COMPACT | 21814 | 19723 | -2091 | -9.58 | 16010 | 14740 | -1270 | -7.93 | 5805 | 4984 | -821 | -14.14 | 73.39 | 74.73 | +1.34 | 71.48 | 78.68 | +7.20 | +10.07 | 72.15 | 74.15 | +2.00 | 70.65 | 78.29 | +7.64 | +10.81 |
| XML_PRETTY | 27632 | 29284 | +1652 | +5.98 | 19906 | 21333 | +1427 | +7.17 | 7726 | 7951 | +225 | +2.91 | 72.04 | 72.85 | +0.81 | 53.04 | 48.60 | -4.44 | -8.38 | 70.13 | 71.55 | +1.42 | 51.76 | 47.73 | -4.03 | -7.79 |
| YAML | 25410 | 26949 | +1539 | +6.06 | 19673 | 21298 | +1625 | +8.26 | 5738 | 5652 | -86 | -1.51 | 77.42 | 79.03 | +1.61 | 63.32 | 59.76 | -3.57 | -5.63 | 75.06 | 77.25 | +2.19 | 61.75 | 58.57 | -3.18 | -5.15 |

### 2.7 Token Utilization Efficiency (Accuracy by Character)
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy by Character (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy by Character (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10315 | 9347 | 968 | 8131 | 7369 | 763 | 18446 | 16716 | 1730 | 90.62 | 92.02 | 90.35 | 93.12 | 89.11 | 91.01 | 89.35 | 92.12 |
| JSON_COMPACT | opt | 9788 | 9319 | 469 | 8462 | 8057 | 405 | 18250 | 17376 | 874 | 95.21 | 96.77 | 91.99 | 96.78 | 92.81 | 95.17 | 90.39 | 95.18 |
| JSON_PRETTY | man | 17828 | 17234 | 594 | 10771 | 10412 | 359 | 28599 | 27646 | 952 | 96.67 | 71.84 | 82.99 | 66.54 | 92.97 | 69.38 | 80.53 | 64.08 |
| JSON_PRETTY | opt | 16899 | 15909 | 990 | 9659 | 9093 | 566 | 26558 | 25002 | 1556 | 94.14 | 73.15 | 86.11 | 71.01 | 90.00 | 70.39 | 83.35 | 68.25 |
| TOON_DEFAULT | man | 14096 | 13322 | 774 | 8685 | 8208 | 477 | 22781 | 21530 | 1251 | 94.51 | 82.43 | 90.56 | 82.65 | 92.77 | 81.27 | 89.40 | 81.49 |
| TOON_DEFAULT | opt | 13859 | 13403 | 456 | 12191 | 11790 | 401 | 26050 | 25193 | 857 | 96.71 | 84.66 | 76.89 | 74.25 | 93.06 | 82.22 | 74.45 | 71.82 |
| XML_COMPACT | man | 12848 | 11933 | 915 | 8966 | 8328 | 638 | 21814 | 20261 | 1553 | 92.88 | 85.36 | 88.26 | 84.47 | 90.97 | 84.09 | 86.98 | 83.20 |
| XML_COMPACT | opt | 12368 | 11689 | 679 | 7356 | 6952 | 404 | 19724 | 18641 | 1083 | 94.51 | 88.00 | 96.30 | 91.87 | 91.09 | 85.72 | 94.02 | 89.59 |
| XML_PRETTY | man | 20114 | 18959 | 1155 | 7518 | 7086 | 432 | 27632 | 26046 | 1586 | 94.26 | 62.87 | 95.43 | 67.85 | 91.16 | 60.81 | 93.36 | 65.78 |
| XML_PRETTY | opt | 19583 | 18347 | 1236 | 9701 | 9088 | 612 | 29284 | 27436 | 1848 | 93.69 | 64.20 | 85.63 | 62.49 | 91.02 | 62.42 | 83.85 | 60.71 |
| YAML | man | 14306 | 13772 | 534 | 11104 | 10690 | 414 | 25410 | 24463 | 948 | 96.27 | 82.92 | 81.29 | 75.89 | 93.67 | 81.19 | 79.55 | 74.16 |
| YAML | opt | 14053 | 13792 | 261 | 12896 | 12656 | 240 | 26949 | 26448 | 501 | 98.14 | 84.99 | 74.80 | 72.50 | 95.00 | 82.89 | 72.70 | 70.40 |

#### 2.7.2 Read Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10315 | 9788 | -527 | -5.11 | 9347 | 9319 | -28 | -0.30 | 968 | 469 | -499 | -51.52 | 90.62 | 95.21 | +4.59 | 92.02 | 96.77 | +4.76 | +5.17 | 89.11 | 92.81 | +3.70 | 91.01 | 95.17 | +4.16 | +4.58 |
| JSON_PRETTY | 17828 | 16899 | -929 | -5.21 | 17234 | 15908 | -1326 | -7.69 | 594 | 991 | +397 | +66.77 | 96.67 | 94.14 | -2.53 | 71.84 | 73.15 | +1.31 | +1.82 | 92.97 | 90.00 | -2.97 | 69.38 | 70.39 | +1.01 | +1.46 |
| TOON_DEFAULT | 14096 | 13859 | -237 | -1.68 | 13322 | 13403 | +81 | +0.61 | 774 | 456 | -318 | -41.07 | 94.51 | 96.71 | +2.20 | 82.43 | 84.66 | +2.23 | +2.71 | 92.77 | 93.06 | +0.29 | 81.27 | 82.22 | +0.96 | +1.18 |
| XML_COMPACT | 12848 | 12368 | -480 | -3.74 | 11933 | 11689 | -244 | -2.05 | 915 | 679 | -236 | -25.77 | 92.88 | 94.51 | +1.63 | 85.36 | 88.00 | +2.63 | +3.09 | 90.97 | 91.09 | +0.12 | 84.09 | 85.72 | +1.63 | +1.93 |
| XML_PRETTY | 20114 | 19583 | -531 | -2.64 | 18959 | 18347 | -612 | -3.23 | 1155 | 1236 | +81 | +7.03 | 94.26 | 93.69 | -0.57 | 62.87 | 64.20 | +1.33 | +2.12 | 91.16 | 91.02 | -0.14 | 60.81 | 62.42 | +1.62 | +2.66 |
| YAML | 14306 | 14053 | -253 | -1.77 | 13772 | 13791 | +19 | +0.14 | 534 | 262 | -272 | -50.98 | 96.27 | 98.14 | +1.87 | 82.92 | 84.99 | +2.06 | +2.49 | 93.67 | 95.00 | +1.33 | 81.19 | 82.89 | +1.70 | +2.10 |

#### 2.7.3 Output Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 8131 | 8462 | +331 | +4.07 | 7369 | 8057 | +688 | +9.34 | 763 | 406 | -357 | -46.84 | 90.62 | 95.21 | +4.59 | 90.35 | 91.99 | +1.63 | +1.81 | 89.11 | 92.81 | +3.70 | 89.35 | 90.39 | +1.04 | +1.16 |
| JSON_PRETTY | 10771 | 9659 | -1112 | -10.32 | 10412 | 9093 | -1319 | -12.67 | 359 | 566 | +207 | +57.76 | 96.67 | 94.14 | -2.53 | 82.99 | 86.11 | +3.11 | +3.75 | 92.97 | 90.00 | -2.97 | 80.53 | 83.35 | +2.82 | +3.50 |
| TOON_DEFAULT | 8685 | 12191 | +3506 | +40.37 | 8208 | 11790 | +3582 | +43.64 | 477 | 401 | -76 | -15.87 | 94.51 | 96.71 | +2.20 | 90.56 | 76.89 | -13.67 | -15.10 | 92.77 | 93.06 | +0.29 | 89.40 | 74.45 | -14.94 | -16.72 |
| XML_COMPACT | 8966 | 7355 | -1611 | -17.96 | 8328 | 6952 | -1376 | -16.52 | 638 | 403 | -235 | -36.77 | 92.88 | 94.51 | +1.63 | 88.26 | 96.30 | +8.04 | +9.11 | 90.97 | 91.09 | +0.12 | 86.98 | 94.02 | +7.03 | +8.09 |
| XML_PRETTY | 7518 | 9701 | +2183 | +29.03 | 7086 | 9088 | +2002 | +28.26 | 432 | 613 | +181 | +41.80 | 94.26 | 93.69 | -0.57 | 95.43 | 85.63 | -9.80 | -10.27 | 91.16 | 91.02 | -0.14 | 93.36 | 83.85 | -9.52 | -10.19 |
| YAML | 11104 | 12896 | +1792 | +16.14 | 10690 | 12656 | +1966 | +18.39 | 414 | 240 | -174 | -42.11 | 96.27 | 98.14 | +1.87 | 81.29 | 74.80 | -6.49 | -7.98 | 93.67 | 95.00 | +1.33 | 79.55 | 72.70 | -6.85 | -8.61 |

#### 2.7.4 Total Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 18446 | 18250 | -196 | -1.06 | 16716 | 17376 | +660 | +3.95 | 1730 | 874 | -856 | -49.49 | 90.62 | 95.21 | +4.59 | 93.12 | 96.78 | +3.65 | +3.92 | 89.11 | 92.81 | +3.70 | 92.12 | 95.18 | +3.06 | +3.32 |
| JSON_PRETTY | 28599 | 26558 | -2041 | -7.14 | 27646 | 25001 | -2645 | -9.57 | 952 | 1556 | +604 | +63.44 | 96.67 | 94.14 | -2.53 | 66.54 | 71.01 | +4.47 | +6.71 | 92.97 | 90.00 | -2.97 | 64.08 | 68.25 | +4.17 | +6.51 |
| TOON_DEFAULT | 22781 | 26050 | +3269 | +14.35 | 21530 | 25193 | +3663 | +17.01 | 1251 | 857 | -394 | -31.46 | 94.51 | 96.71 | +2.20 | 82.65 | 74.25 | -8.39 | -10.15 | 92.77 | 93.06 | +0.29 | 81.49 | 71.82 | -9.67 | -11.86 |
| XML_COMPACT | 21814 | 19723 | -2091 | -9.58 | 20261 | 18641 | -1620 | -8.00 | 1553 | 1083 | -470 | -30.29 | 92.88 | 94.51 | +1.63 | 84.47 | 91.87 | +7.39 | +8.75 | 90.97 | 91.09 | +0.12 | 83.20 | 89.59 | +6.38 | +7.67 |
| XML_PRETTY | 27632 | 29284 | +1652 | +5.98 | 26046 | 27436 | +1390 | +5.34 | 1586 | 1848 | +262 | +16.50 | 94.26 | 93.69 | -0.57 | 67.85 | 62.49 | -5.36 | -7.90 | 91.16 | 91.02 | -0.14 | 65.78 | 60.71 | -5.07 | -7.71 |
| YAML | 25410 | 26949 | +1539 | +6.06 | 24463 | 26449 | +1986 | +8.12 | 948 | 501 | -447 | -47.10 | 96.27 | 98.14 | +1.87 | 75.89 | 72.50 | -3.39 | -4.47 | 93.67 | 95.00 | +1.33 | 74.16 | 70.40 | -3.75 | -5.06 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Expected Characters | Output Characters | Correct Characters | Incorrect Characters | Accuracy by Character (%) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 92 | 32 | 0 | 74.19 | 7782 | 8113 | 7349 | 764 | 90.62 |
| JSON_COMPACT | opt | 95 | 29 | 0 | 76.61 | 8451 | 8718 | 8295 | 423 | 95.21 |
| JSON_PRETTY | man | 94 | 30 | 0 | 75.54 | 7782 | 7857 | 7595 | 262 | 96.67 |
| JSON_PRETTY | opt | 88 | 36 | 0 | 70.97 | 8451 | 8688 | 8179 | 509 | 94.14 |
| TOON_DEFAULT | man | 90 | 34 | 0 | 72.71 | 7782 | 7966 | 7526 | 440 | 94.51 |
| TOON_DEFAULT | opt | 97 | 27 | 0 | 78.09 | 8451 | 8585 | 8299 | 286 | 96.71 |
| XML_COMPACT | man | 91 | 33 | 0 | 73.39 | 7782 | 8010 | 7432 | 578 | 92.88 |
| XML_COMPACT | opt | 93 | 31 | 0 | 74.73 | 8451 | 8722 | 8240 | 482 | 94.51 |
| XML_PRETTY | man | 89 | 35 | 0 | 72.04 | 7782 | 7992 | 7530 | 462 | 94.26 |
| XML_PRETTY | opt | 90 | 34 | 0 | 72.85 | 8451 | 8721 | 8169 | 553 | 93.69 |
| YAML | man | 96 | 28 | 0 | 77.42 | 7782 | 7896 | 7598 | 298 | 96.27 |
| YAML | opt | 98 | 26 | 0 | 79.03 | 8451 | 8510 | 8352 | 158 | 98.14 |

#### 2.8.2 Answers: Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 92 | 95 | +3 | +3.26 | 32 | 29 | -3 | -9.37 | 0 | 0 | 0 | 0.00 | 74.19 | 76.61 | +2.42 |
| JSON_PRETTY | 94 | 88 | -6 | -6.03 | 30 | 36 | +6 | +18.90 | 0 | 0 | 0 | 0.00 | 75.54 | 70.97 | -4.57 |
| TOON_DEFAULT | 90 | 97 | +7 | +7.40 | 34 | 27 | -7 | -19.59 | 0 | 0 | 0 | 0.00 | 72.71 | 78.09 | +5.38 |
| XML_COMPACT | 91 | 93 | +2 | +1.84 | 33 | 31 | -2 | -5.06 | 0 | 0 | 0 | 0.00 | 73.39 | 74.73 | +1.34 |
| XML_PRETTY | 89 | 90 | +1 | +1.12 | 35 | 34 | -1 | -2.86 | 0 | 0 | 0 | 0.00 | 72.04 | 72.85 | +0.81 |
| YAML | 96 | 98 | +2 | +2.08 | 28 | 26 | -2 | -7.14 | 0 | 0 | 0 | 0.00 | 77.42 | 79.03 | +1.61 |

#### 2.8.3 Characters: Mandatory vs Optional Data
| Format | Output Characters Man | Output Characters Opt | Diff | Diff (%) | Correct Characters Man | Correct Characters Opt | Diff | Diff (%) | Incorrect Characters Man | Incorrect Characters Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 8113 | 8718 | +605 | +7.46 | 7349 | 8295 | +946 | +12.88 | 764 | 423 | -341 | -44.68 | 90.62 | 95.21 | 4.59 |
| JSON_PRETTY | 7857 | 8688 | +831 | +10.57 | 7595 | 8179 | +584 | +7.68 | 262 | 509 | +247 | +94.27 | 96.67 | 94.14 | -2.53 |
| TOON_DEFAULT | 7966 | 8585 | +619 | +7.76 | 7526 | 8299 | +773 | +10.27 | 440 | 285 | -155 | -35.15 | 94.51 | 96.71 | 2.20 |
| XML_COMPACT | 8010 | 8722 | +712 | +8.89 | 7432 | 8240 | +808 | +10.87 | 578 | 482 | -96 | -16.55 | 92.88 | 94.51 | 1.63 |
| XML_PRETTY | 7992 | 8721 | +729 | +9.13 | 7530 | 8169 | +639 | +8.48 | 462 | 553 | +91 | +19.62 | 94.26 | 93.69 | -0.57 |
| YAML | 7896 | 8511 | +615 | +7.78 | 7598 | 8352 | +754 | +9.92 | 298 | 159 | -139 | -46.76 | 96.27 | 98.14 | 1.87 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 74.19 | 84.85 | 60.50 | 66.67 | 71.43 | 89.11 | 31.82 | 17.64 | 13.89 | 8.93 |
| JSON_COMPACT | opt | 76.61 | 93.94 | 71.61 | 61.91 | 52.38 | 92.81 | 35.23 | 20.88 | 12.90 | 6.55 |
| JSON_PRETTY | man | 75.54 | 96.36 | 65.43 | 60.31 | 49.20 | 92.97 | 36.14 | 19.08 | 12.57 | 6.15 |
| JSON_PRETTY | opt | 70.97 | 90.30 | 58.02 | 61.90 | 46.03 | 90.00 | 33.86 | 16.92 | 12.90 | 5.75 |
| TOON_DEFAULT | man | 72.71 | 91.21 | 50.62 | 59.52 | 65.87 | 92.77 | 34.20 | 14.76 | 12.40 | 8.23 |
| TOON_DEFAULT | opt | 78.09 | 96.36 | 72.84 | 71.43 | 43.65 | 93.06 | 36.14 | 21.25 | 14.88 | 5.46 |
| XML_COMPACT | man | 73.39 | 87.88 | 62.96 | 66.67 | 55.55 | 90.97 | 32.95 | 18.36 | 13.89 | 6.95 |
| XML_COMPACT | opt | 74.73 | 87.27 | 72.84 | 63.49 | 55.55 | 91.09 | 32.73 | 21.25 | 13.23 | 6.94 |
| XML_PRETTY | man | 72.04 | 88.48 | 62.96 | 53.97 | 58.73 | 91.16 | 33.18 | 18.36 | 11.24 | 7.34 |
| XML_PRETTY | opt | 72.85 | 89.09 | 62.97 | 63.49 | 52.38 | 91.02 | 33.41 | 18.36 | 13.23 | 6.55 |
| YAML | man | 77.42 | 97.58 | 62.96 | 60.31 | 60.32 | 93.67 | 36.59 | 18.36 | 12.57 | 7.54 |
| YAML | opt | 79.03 | 99.39 | 67.90 | 63.49 | 55.56 | 95.00 | 37.27 | 19.81 | 13.23 | 6.94 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 84.85 | 93.94 | +9.09 | 31.82 | 35.23 | +3.41 |
| JSON_PRETTY | 96.36 | 90.30 | -6.06 | 36.14 | 33.86 | -2.27 |
| TOON_DEFAULT | 91.21 | 96.36 | +5.15 | 34.20 | 36.14 | +1.93 |
| XML_COMPACT | 87.88 | 87.27 | -0.61 | 32.95 | 32.73 | -0.23 |
| XML_PRETTY | 88.48 | 89.09 | +0.61 | 33.18 | 33.41 | +0.23 |
| YAML | 97.58 | 99.39 | +1.82 | 36.59 | 37.27 | +0.68 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 60.50 | 71.61 | +11.11 | 17.64 | 20.88 | +3.24 |
| JSON_PRETTY | 65.43 | 58.02 | -7.41 | 19.08 | 16.92 | -2.16 |
| TOON_DEFAULT | 50.62 | 72.84 | +22.22 | 14.76 | 21.25 | +6.49 |
| XML_COMPACT | 62.96 | 72.84 | +9.88 | 18.36 | 21.25 | +2.88 |
| XML_PRETTY | 62.96 | 62.97 | 0.00 | 18.36 | 18.36 | 0.00 |
| YAML | 62.96 | 67.90 | +4.94 | 18.36 | 19.81 | +1.44 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 66.67 | 61.91 | -4.76 | 13.89 | 12.90 | -0.99 |
| JSON_PRETTY | 60.31 | 61.90 | +1.59 | 12.57 | 12.90 | +0.33 |
| TOON_DEFAULT | 59.52 | 71.43 | +11.91 | 12.40 | 14.88 | +2.48 |
| XML_COMPACT | 66.67 | 63.49 | -3.18 | 13.89 | 13.23 | -0.66 |
| XML_PRETTY | 53.97 | 63.49 | +9.52 | 11.24 | 13.23 | +1.98 |
| YAML | 60.31 | 63.49 | +3.18 | 12.57 | 13.23 | +0.66 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 71.43 | 52.38 | -19.05 | 8.93 | 6.55 | -2.38 |
| JSON_PRETTY | 49.20 | 46.03 | -3.17 | 6.15 | 5.75 | -0.40 |
| TOON_DEFAULT | 65.87 | 43.65 | -22.22 | 8.23 | 5.46 | -2.78 |
| XML_COMPACT | 55.55 | 55.55 | 0.00 | 6.95 | 6.94 | 0.00 |
| XML_PRETTY | 58.73 | 52.38 | -6.35 | 7.34 | 6.55 | -0.79 |
| YAML | 60.32 | 55.56 | -4.76 | 7.54 | 6.94 | -0.60 |

### 2.10 Accuracy By Character Per Question Category Analysis
#### 2.10.1 Metrics
| Format | Variant | Accuracy By Character (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc By Char (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 90.62 | 82.39 | 97.00 | 93.75 | 83.13 | 89.11 | 30.89 | 28.29 | 19.53 | 10.39 |
| JSON_COMPACT | opt | 95.21 | 95.41 | 95.79 | 92.04 | 79.39 | 92.81 | 35.78 | 27.94 | 19.17 | 9.92 |
| JSON_PRETTY | man | 96.67 | 96.89 | 97.45 | 88.99 | 77.37 | 92.97 | 36.34 | 28.42 | 18.54 | 9.67 |
| JSON_PRETTY | opt | 94.14 | 96.65 | 93.14 | 83.78 | 73.13 | 90.00 | 36.24 | 27.17 | 17.45 | 9.14 |
| TOON_DEFAULT | man | 94.51 | 93.70 | 95.60 | 91.52 | 85.49 | 92.77 | 35.14 | 27.88 | 19.06 | 10.69 |
| TOON_DEFAULT | opt | 96.71 | 96.54 | 98.02 | 92.93 | 71.32 | 93.06 | 36.20 | 28.59 | 19.36 | 8.91 |
| XML_COMPACT | man | 92.88 | 90.43 | 95.20 | 92.86 | 79.63 | 90.97 | 33.91 | 27.76 | 19.34 | 9.95 |
| XML_COMPACT | opt | 94.51 | 92.21 | 97.37 | 88.79 | 76.97 | 91.09 | 34.58 | 28.40 | 18.50 | 9.62 |
| XML_PRETTY | man | 94.26 | 92.73 | 95.99 | 87.80 | 80.86 | 91.16 | 34.77 | 28.00 | 18.29 | 10.11 |
| XML_PRETTY | opt | 93.69 | 91.61 | 96.29 | 90.56 | 77.78 | 91.02 | 34.35 | 28.08 | 18.87 | 9.72 |
| YAML | man | 96.27 | 97.24 | 96.27 | 90.77 | 81.69 | 93.67 | 36.47 | 28.08 | 18.91 | 10.21 |
| YAML | opt | 98.14 | 99.82 | 97.60 | 92.33 | 78.93 | 95.00 | 37.43 | 28.47 | 19.24 | 9.87 |

#### 2.10.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 82.39 | 95.41 | +13.02 | 30.89 | 35.78 | +4.88 |
| JSON_PRETTY | 96.89 | 96.65 | -0.24 | 36.34 | 36.24 | -0.09 |
| TOON_DEFAULT | 93.70 | 96.54 | +2.85 | 35.14 | 36.20 | +1.07 |
| XML_COMPACT | 90.43 | 92.21 | +1.77 | 33.91 | 34.58 | +0.66 |
| XML_PRETTY | 92.73 | 91.61 | -1.12 | 34.77 | 34.35 | -0.42 |
| YAML | 97.24 | 99.82 | +2.58 | 36.47 | 37.43 | +0.97 |

#### 2.10.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 97.00 | 95.79 | -1.21 | 28.29 | 27.94 | -0.35 |
| JSON_PRETTY | 97.45 | 93.14 | -4.30 | 28.42 | 27.17 | -1.26 |
| TOON_DEFAULT | 95.60 | 98.02 | +2.42 | 27.88 | 28.59 | +0.71 |
| XML_COMPACT | 95.20 | 97.37 | +2.17 | 27.76 | 28.40 | +0.63 |
| XML_PRETTY | 95.99 | 96.29 | +0.29 | 28.00 | 28.08 | +0.08 |
| YAML | 96.27 | 97.60 | +1.33 | 28.08 | 28.47 | +0.39 |

#### 2.10.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 93.75 | 92.04 | -1.71 | 19.53 | 19.17 | -0.36 |
| JSON_PRETTY | 88.99 | 83.78 | -5.21 | 18.54 | 17.45 | -1.08 |
| TOON_DEFAULT | 91.52 | 92.93 | +1.41 | 19.06 | 19.36 | +0.30 |
| XML_COMPACT | 92.86 | 88.79 | -4.06 | 19.34 | 18.50 | -0.84 |
| XML_PRETTY | 87.80 | 90.56 | +2.76 | 18.29 | 18.87 | +0.58 |
| YAML | 90.77 | 92.33 | +1.56 | 18.91 | 19.24 | +0.33 |

#### 2.10.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 83.13 | 79.39 | -3.73 | 10.39 | 9.92 | -0.47 |
| JSON_PRETTY | 77.37 | 73.13 | -4.24 | 9.67 | 9.14 | -0.53 |
| TOON_DEFAULT | 85.49 | 71.32 | -14.18 | 10.69 | 8.91 | -1.77 |
| XML_COMPACT | 79.63 | 76.97 | -2.66 | 9.95 | 9.62 | -0.33 |
| XML_PRETTY | 80.86 | 77.78 | -3.09 | 10.11 | 9.72 | -0.39 |
| YAML | 81.69 | 78.93 | -2.75 | 10.21 | 9.87 | -0.34 |

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

- **Report Generated**: 2026-04-14
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code 2.1.73
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_haiku_4_5/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_haiku_4_5/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_haiku_4_5/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)