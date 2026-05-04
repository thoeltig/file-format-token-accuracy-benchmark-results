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

1. **TOON_DEFAULT** is the most token-efficient format on dense data but the most volatile across variants. On mandatory data it consumes the fewest read tokens (7,048) which beats **CSV** by 6%. On optional/sparse data its adaptive switch from tabular to key-value encoding inflates read tokens by 64% (to 11,561) which is the largest variant swing of any format.
2. **CSV** achieves the best overall efficiency scores (84.02 mandatory, 78.05 optional on total tokens) by pairing low read tokens with low output tokens even though its accuracy by character is not the best compared to the other formats.
3. **JSON_PRETTY** reaches the highest accuracy by answer (80.91% mandatory) and accuracy by character (97.83% mandatory) but pays a steep token premium. Its total token consumption on mandatory data (27,421) is 64% higher than **CSV** which produces the worst total efficiency score in this benchmark (58.13).
4. Accuracy by character is substantially higher than accuracy by answer across every format (92 to 98% vs. 73 to 81%). This gap reveals that most errors are partial which means the model typically retrieves the right structure and most of the answer but gets individual characters, words or numeric digits wrong.
5. Aggregation is the weakest question category for every format. Accuracy ranges from 34.92% (**YAML** optional) to 76.19% (**CSV** mandatory) which is far below field retrieval which peaks at 98.79%. This confirms that computational reasoning over structured data is the primary bottleneck for the model and not data parsing.
6. Five of seven formats show equal or higher accuracy on optional/sparse data most likely because they contain fewer values (631 vs. 682). The likely explanation is that fewer fields per record reduce the information density the model must track which lowers retrieval and counting errors. **CSV** and **JSON_PRETTY** are the two exceptions that lose accuracy on optional data.
7. **XML** formats deliver the most stable accuracy between variants. **XML_COMPACT** improves by +3.22 percentage points and **XML_PRETTY** by +2.96 on optional data which are the smallest absolute swings in the benchmark. **JSON_COMPACT** shows the largest positive shift (+4.57 pp) while **JSON_PRETTY** drops the most (−4.84 pp).

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
| CSV ≈ 75s | TOON_DEFAULT ≈ 7048 | JSON_COMPACT ≈ 228 | CSV ≈ 8957 | CSV ≈ 9256 | CSV ≈ 16749 | JSON_PRETTY ≈ 80.91% | TOON_DEFAULT ≈ 86 | CSV ≈ 79 | CSV ≈ 84 | JSON_PRETTY ≈ 97.83% | TOON_DEFAULT ≈ 95 | XML_COMPACT ≈ 92 | CSV ≈ 95 |
| XML_COMPACT (+0.63%) | CSV (+6.31%) | YAML (+1.46%) | XML_COMPACT (+1.44%) | XML_COMPACT (+1.86%) | TOON_DEFAULT (+11.76%) | TOON_DEFAULT (-2.55%) | CSV (-3.68%) | XML_COMPACT (-1.41%) | TOON_DEFAULT (-4.58%) | JSON_COMPACT (-0.36%) | CSV (-1.60%) | CSV (-2.10%) | TOON_DEFAULT (-5.77%) |
| YAML (+17.93%) | JSON_COMPACT (+31.50%) | TOON_DEFAULT (+26.06%) | YAML (+15.60%) | YAML (+14.35%) | XML_COMPACT (+26.10%) | XML_PRETTY (-4.30%) | JSON_COMPACT (-13.34%) | YAML (-10.10%) | XML_COMPACT (-14.41%) | XML_PRETTY (-0.85%) | JSON_COMPACT (-4.86%) | YAML (-6.71%) | XML_COMPACT (-9.55%) |
| TOON_DEFAULT (+28.20%) | XML_COMPACT (+65.91%) | CSV (+31.63%) | TOON_DEFAULT (+27.10%) | TOON_DEFAULT (+26.08%) | JSON_COMPACT (+33.91%) | CSV (-4.84%) | XML_COMPACT (-21.80%) | TOON_DEFAULT (-14.77%) | JSON_COMPACT (-20.57%) | YAML (-0.89%) | XML_COMPACT (-14.74%) | TOON_DEFAULT (-16.59%) | JSON_COMPACT (-12.77%) |
| XML_PRETTY (+37.08%) | YAML (+78.12%) | JSON_PRETTY (+49.78%) | XML_PRETTY (+38.86%) | XML_PRETTY (+38.07%) | YAML (+38.15%) | XML_COMPACT (-5.10%) | YAML (-26.10%) | JSON_PRETTY (-22.77%) | YAML (-21.60%) | XML_COMPACT (-1.13%) | YAML (-17.88%) | XML_PRETTY (-19.74%) | YAML (-15.18%) |
| JSON_PRETTY (+41.63%) | JSON_PRETTY (+102.65%) | XML_COMPACT (+50.51%) | JSON_PRETTY (+42.88%) | JSON_PRETTY (+41.94%) | JSON_PRETTY (+63.72%) | YAML (-5.91%) | JSON_PRETTY (-28.87%) | XML_PRETTY (-23.93%) | JSON_PRETTY (-30.82%) | CSV (-5.43%) | JSON_PRETTY (-23.91%) | JSON_PRETTY (-21.26%) | JSON_PRETTY (-26.87%) |
| JSON_COMPACT (+46.68%) | XML_PRETTY (+129.37%) | XML_PRETTY (+50.51%) | JSON_COMPACT (+44.40%) | JSON_COMPACT (+42.18%) | XML_PRETTY (+72.82%) | JSON_COMPACT (-7.52%) | XML_PRETTY (-40.25%) | JSON_COMPACT (-29.28%) | XML_PRETTY (-39.18%) | TOON_DEFAULT (-5.59%) | XML_PRETTY (-31.75%) | JSON_COMPACT (-21.65%) | XML_PRETTY (-31.85%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 73s | CSV ≈ 7225 | XML_PRETTY ≈ 232 | JSON_PRETTY ≈ 7966 | JSON_PRETTY ≈ 8302 | CSV ≈ 18282 | XML_PRETTY ≈ 79.57% | CSV ≈ 82 | JSON_PRETTY ≈ 84 | JSON_COMPACT ≈ 80 | XML_COMPACT ≈ 97.93% | CSV ≈ 95 | JSON_PRETTY ≈ 98 | CSV ≈ 92 |
| XML_PRETTY (+7.93%) | JSON_COMPACT (+21.08%) | CSV (+30.56%) | JSON_COMPACT (+21.36%) | JSON_COMPACT (+20.52%) | JSON_COMPACT (+2.58%) | TOON_DEFAULT (-0.40%) | JSON_COMPACT (-3.08%) | XML_PRETTY (-8.43%) | CSV (-2.21%) | YAML (-0.02%) | JSON_COMPACT (-4.51%) | XML_PRETTY (-9.06%) | JSON_COMPACT (-0.03%) |
| JSON_COMPACT (+13.19%) | XML_COMPACT (+51.28%) | YAML (+43.47%) | XML_PRETTY (+22.97%) | XML_PRETTY (+20.79%) | JSON_PRETTY (+18.53%) | XML_COMPACT (-0.54%) | XML_COMPACT (-11.96%) | JSON_COMPACT (-9.56%) | JSON_PRETTY (-11.55%) | XML_PRETTY (-0.51%) | XML_COMPACT (-11.47%) | JSON_COMPACT (-9.93%) | JSON_PRETTY (-8.24%) |
| CSV (+19.75%) | TOON_DEFAULT (+60.01%) | XML_COMPACT (+44.33%) | YAML (+27.74%) | YAML (+26.59%) | YAML (+21.87%) | JSON_COMPACT (-1.61%) | TOON_DEFAULT (-14.66%) | YAML (-13.04%) | XML_COMPACT (-11.67%) | JSON_PRETTY (-1.36%) | YAML (-14.70%) | YAML (-11.41%) | YAML (-9.09%) |
| YAML (+21.03%) | YAML (+62.92%) | JSON_PRETTY (+44.62%) | CSV (+34.99%) | CSV (+33.18%) | XML_COMPACT (+22.68%) | YAML (-1.88%) | YAML (-16.81%) | XML_COMPACT (-18.40%) | YAML (-12.28%) | JSON_COMPACT (-1.98%) | TOON_DEFAULT (-15.54%) | XML_COMPACT (-16.92%) | XML_COMPACT (-9.51%) |
| TOON_DEFAULT (+28.94%) | JSON_PRETTY (+85.01%) | JSON_COMPACT (+45.34%) | XML_COMPACT (+40.13%) | XML_COMPACT (+38.50%) | TOON_DEFAULT (+27.61%) | JSON_PRETTY (-3.50%) | JSON_PRETTY (-25.27%) | CSV (-20.01%) | TOON_DEFAULT (-14.63%) | TOON_DEFAULT (-2.38%) | JSON_PRETTY (-21.74%) | CSV (-17.09%) | TOON_DEFAULT (-13.91%) |
| XML_COMPACT (+30.13%) | XML_PRETTY (+108.84%) | TOON_DEFAULT (+46.41%) | TOON_DEFAULT (+43.45%) | TOON_DEFAULT (+41.74%) | XML_PRETTY (+37.39%) | CSV (-6.18%) | XML_PRETTY (-30.11%) | TOON_DEFAULT (-20.03%) | XML_PRETTY (-20.41%) | CSV (-3.87%) | XML_PRETTY (-27.73%) | TOON_DEFAULT (-20.04%) | XML_PRETTY (-17.87%) |


#### 2.1.2 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 98.79% | JSON_PRETTY ≈ 74.08% | YAML ≈ 73.02% | CSV ≈ 76.19% |
| YAML (0.00%) | XML_COMPACT (-4.94%) | XML_PRETTY (-4.76%) | TOON_DEFAULT (-9.52%) |
| JSON_COMPACT (-1.21%) | XML_PRETTY (-4.94%) | TOON_DEFAULT (-7.94%) | JSON_PRETTY (-17.46%) |
| XML_PRETTY (-1.21%) | CSV (-7.41%) | JSON_PRETTY (-7.94%) | XML_COMPACT (-22.22%) |
| TOON_DEFAULT (-4.24%) | TOON_DEFAULT (-9.26%) | XML_COMPACT (-9.53%) | XML_PRETTY (-36.51%) |
| XML_COMPACT (-6.67%) | JSON_COMPACT (-11.11%) | JSON_COMPACT (-14.29%) | YAML (-36.51%) |
| CSV (-10.91%) | YAML (-18.52%) | CSV (-15.88%) | JSON_COMPACT (-38.09%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| XML_COMPACT ≈ 98.79% | JSON_COMPACT ≈ 81.48% | JSON_COMPACT ≈ 71.43% | XML_PRETTY ≈ 61.90% |
| XML_PRETTY (-0.61%) | TOON_DEFAULT (-1.85%) | TOON_DEFAULT (0.00%) | JSON_COMPACT (-14.28%) |
| YAML (-1.21%) | YAML (-4.94%) | YAML (-1.59%) | TOON_DEFAULT (-14.29%) |
| JSON_PRETTY (-3.64%) | JSON_PRETTY (-7.41%) | XML_COMPACT (-4.76%) | XML_COMPACT (-15.87%) |
| TOON_DEFAULT (-4.85%) | XML_COMPACT (-7.41%) | XML_PRETTY (-6.35%) | CSV (-17.46%) |
| CSV (-8.48%) | CSV (-12.35%) | CSV (-7.94%) | JSON_PRETTY (-19.05%) |
| JSON_COMPACT (-8.48%) | XML_PRETTY (-14.82%) | JSON_PRETTY (-9.53%) | YAML (-26.98%) |


#### 2.1.3 Category Accuracy By Character Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99.47% | XML_PRETTY ≈ 98.31% | YAML ≈ 94.05% | CSV ≈ 88.06% |
| JSON_COMPACT (-0.50%) | JSON_PRETTY (-0.27%) | XML_PRETTY (-0.89%) | TOON_DEFAULT (-5.04%) |
| JSON_PRETTY (-0.89%) | XML_COMPACT (-0.45%) | TOON_DEFAULT (-1.04%) | JSON_PRETTY (-5.35%) |
| XML_PRETTY (-3.14%) | JSON_COMPACT (-0.78%) | XML_COMPACT (-1.78%) | XML_COMPACT (-8.23%) |
| XML_COMPACT (-3.34%) | CSV (-1.92%) | JSON_PRETTY (-3.27%) | XML_PRETTY (-12.55%) |
| TOON_DEFAULT (-3.88%) | YAML (-2.16%) | CSV (-3.87%) | JSON_COMPACT (-13.99%) |
| CSV (-12.00%) | TOON_DEFAULT (-7.96%) | JSON_COMPACT (-6.25%) | YAML (-15.22%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| XML_COMPACT ≈ 99.65% | YAML ≈ 98.14% | TOON_DEFAULT ≈ 92.92% | XML_PRETTY ≈ 79.19% |
| YAML (-0.53%) | TOON_DEFAULT (-0.06%) | JSON_COMPACT (0.00%) | JSON_COMPACT (-5.25%) |
| XML_PRETTY (-0.61%) | JSON_COMPACT (-0.18%) | XML_COMPACT (-1.77%) | XML_COMPACT (-5.45%) |
| JSON_PRETTY (-2.90%) | CSV (-0.44%) | YAML (-2.06%) | TOON_DEFAULT (-5.55%) |
| JSON_COMPACT (-4.99%) | JSON_PRETTY (-0.52%) | XML_PRETTY (-2.95%) | CSV (-7.47%) |
| TOON_DEFAULT (-5.93%) | XML_COMPACT (-0.58%) | JSON_PRETTY (-5.31%) | JSON_PRETTY (-8.08%) |
| CSV (-8.57%) | XML_PRETTY (-1.19%) | CSV (-6.79%) | YAML (-10.10%) |


#### 2.1.4 Conclusion

The rankings reveal a fundamental tension between token cost and accuracy that no single format resolves perfectly.

- Read token efficiency does not predict accuracy. **TOON_DEFAULT** and **CSV** consume the fewest read tokens on mandatory data (7,048 and 7,493) but rank 2nd and 5th in accuracy (78.36% and 76.07%). **JSON_PRETTY** consumes the second most read tokens (14,283) but achieves the highest accuracy (80.91%). The correlation between fewer tokens and better comprehension is weak at best which means optimizing purely for token count might not maximize information fidelity.
- The efficiency score favors accuracy over token count but **CSV** and **TOON_DEFAULT** use so few tokens that they dominate the composite efficiency rankings even when their accuracy trails by several percentage points. **CSV** leads on total efficiency score (84.02 mandatory) and **TOON_DEFAULT** leads on read efficiency score (85.54 mandatory). For use cases where cost or context budget is the binding constraint these two formats offer the best return per token. For use cases where answer correctness matters more than budget **JSON_PRETTY** is the strongest choice despite its token overhead.
- Accuracy by character tells a different story than accuracy by answer. When measured by character correctness the gap between formats narrows and the spread is only 5.6 percentage points on mandatory data (92.24% to 97.83%) versus 7.52 pp on accuracy by answer. **CSV** is the outlier with its 92.40% character accuracy being notably lower than the 96 to 98% range of most other formats. This suggests **CSV**'s lack of explicit field names causes errors that propagate across multiple characters and words while formats with named fields (**JSON**, **XML**, **YAML**) produce more singular mistakes.
- **TOON_DEFAULT**'s adaptive encoding is a double-edged sword. On dense mandatory data it achieves **CSV**-like token efficiency with better accuracy. On sparse optional data the fallback to key-value pairs pushes its read tokens up by 64% which eliminates its efficiency advantage. For applications where data completeness varies **TOON_DEFAULT** introduces unpredictable token costs that make capacity planning harder. **CSV** and **YAML** remain more stable across data shapes with read token variation under 7%.
- Aggregation accuracy is the decisive weakness of the model regardless of format. The best aggregation result (**CSV** mandatory at 76.19%) falls below the worst field retrieval result (**CSV** mandatory at 87.88%). This is a model limitation and not a format limitation. Formats that expose data in a way that facilitates counting and arithmetic (**CSV** with its tabular layout) show a modest advantage but the gap is inconsistent across variants.
- **CSV** offers the best balance of low token cost and acceptable accuracy when data is dense and field structure is consistent. **JSON_PRETTY** is worth its token premium when maximizing answer correctness is the priority. **TOON_DEFAULT** is the strongest choice when the data is known to be dense and complete but should be avoided if data sparsity is unpredictable. **XML** and **YAML** occupy the middle ground without excelling in either dimension.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total | Accuracy By Character (%) | Useful Read Tokens (Acc By Char) | Wasted Read Tokens (Acc By Char) | Useful Output Tokens (Acc By Char) | Wasted Output Tokens (Acc By Char) | Eff Score Read (Acc By Char) | Eff Score Output (Acc By Char) | Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7493 | 9256 | 16749 | 1.35 | 72.23 | 76.07 | 5700 | 1793 | 7041 | 2215 | 82.39 | 78.79 | 84.02 | 92.40 | 6924 | 569 | 8553 | 703 | 93.27 | 89.68 | 94.91 |
| CSV | opt | 7225 | 11057 | 18282 | 1.33 | 86.72 | 73.39 | 5302 | 1923 | 8114 | 2942 | 81.58 | 67.19 | 78.05 | 94.06 | 6796 | 429 | 10400 | 657 | 95.36 | 80.97 | 91.83 |
| JSON_COMPACT | man | 9268 | 13161 | 22429 | 2.15 | 104.30 | 73.39 | 6802 | 2466 | 9659 | 3502 | 74.13 | 55.72 | 66.74 | 97.47 | 9034 | 234 | 12828 | 333 | 90.18 | 71.77 | 82.79 |
| JSON_COMPACT | opt | 8748 | 10005 | 18753 | 2.11 | 77.97 | 77.96 | 6820 | 1928 | 7800 | 2205 | 79.07 | 75.97 | 79.81 | 95.95 | 8394 | 354 | 9600 | 405 | 91.06 | 87.96 | 91.80 |
| JSON_PRETTY | man | 14283 | 13138 | 27421 | 1.69 | 103.20 | 80.91 | 11556 | 2727 | 10630 | 2508 | 60.85 | 60.85 | 58.13 | 97.83 | 13973 | 310 | 12853 | 285 | 72.13 | 72.13 | 69.41 |
| JSON_PRETTY | opt | 13367 | 8302 | 21669 | 1.68 | 64.24 | 76.07 | 10168 | 3199 | 6315 | 1987 | 60.96 | 83.99 | 70.60 | 96.57 | 12909 | 458 | 8017 | 285 | 74.63 | 97.66 | 84.26 |
| TOON_DEFAULT | man | 7048 | 11671 | 18719 | 1.44 | 91.81 | 78.36 | 5523 | 1525 | 9145 | 2526 | 85.54 | 67.15 | 80.17 | 92.24 | 6501 | 547 | 10765 | 906 | 94.79 | 76.41 | 89.43 |
| TOON_DEFAULT | opt | 11561 | 11768 | 23329 | 1.70 | 92.16 | 79.17 | 9153 | 2408 | 9316 | 2451 | 69.61 | 67.16 | 68.13 | 95.55 | 11047 | 514 | 11244 | 524 | 80.53 | 78.08 | 79.05 |
| XML_COMPACT | man | 11693 | 9429 | 21122 | 2.37 | 73.27 | 75.81 | 8864 | 2829 | 7148 | 2281 | 66.89 | 77.68 | 71.92 | 96.70 | 11307 | 386 | 9118 | 311 | 80.82 | 91.60 | 85.84 |
| XML_COMPACT | opt | 10930 | 11498 | 22428 | 2.34 | 90.02 | 79.03 | 8638 | 2292 | 9087 | 2411 | 71.82 | 68.54 | 70.50 | 97.93 | 10704 | 226 | 11260 | 238 | 84.42 | 81.14 | 83.10 |
| XML_PRETTY | man | 16166 | 12780 | 28946 | 1.93 | 100.30 | 76.61 | 12385 | 3781 | 9791 | 2989 | 51.11 | 59.94 | 51.10 | 96.98 | 15678 | 488 | 12394 | 386 | 64.69 | 73.52 | 64.68 |
| XML_PRETTY | opt | 15089 | 10028 | 25117 | 1.92 | 79.00 | 79.57 | 12006 | 3083 | 7980 | 2049 | 57.01 | 76.91 | 63.52 | 97.42 | 14700 | 389 | 9770 | 259 | 68.91 | 88.81 | 75.42 |
| YAML | man | 12554 | 10585 | 23139 | 1.66 | 83.50 | 75.00 | 9416 | 3139 | 7939 | 2646 | 63.21 | 70.83 | 65.87 | 96.94 | 12170 | 384 | 10261 | 324 | 77.84 | 85.46 | 80.50 |
| YAML | opt | 11771 | 10509 | 22280 | 1.65 | 82.07 | 77.69 | 9145 | 2626 | 8165 | 2345 | 67.86 | 73.04 | 70.01 | 97.91 | 11525 | 246 | 10290 | 220 | 81.34 | 86.52 | 83.49 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7493 | 7225 | -268 | -3.58 | 300 | 304 | +4 | +1.33 | 8957 | 10754 | +1797 | +20.06 | 9256 | 11056 | +1800 | +19.45 | 16749 | 18281 | +1532 | +9.15 |
| JSON_COMPACT | 9268 | 8748 | -520 | -5.61 | 228 | 338 | +110 | +48.25 | 12933 | 9668 | -3265 | -25.25 | 13161 | 10006 | -3155 | -23.97 | 22429 | 18754 | -3675 | -16.39 |
| JSON_PRETTY | 14283 | 13367 | -916 | -6.41 | 341 | 336 | -5 | -1.47 | 12797 | 7966 | -4831 | -37.75 | 13138 | 8302 | -4836 | -36.81 | 27421 | 21669 | -5752 | -20.98 |
| TOON_DEFAULT | 7048 | 11561 | +4513 | +64.03 | 287 | 340 | +53 | +18.47 | 11384 | 11428 | +44 | +0.39 | 11671 | 11768 | +97 | +0.83 | 18719 | 23329 | +4610 | +24.63 |
| XML_COMPACT | 11693 | 10930 | -763 | -6.53 | 343 | 336 | -7 | -2.04 | 9086 | 11163 | +2077 | +22.86 | 9429 | 11499 | +2070 | +21.95 | 21122 | 22429 | +1307 | +6.19 |
| XML_PRETTY | 16166 | 15089 | -1077 | -6.66 | 343 | 233 | -110 | -32.07 | 12438 | 9796 | -2642 | -21.24 | 12780 | 10028 | -2752 | -21.53 | 28946 | 25117 | -3829 | -13.23 |
| YAML | 12554 | 11771 | -783 | -6.24 | 231 | 333 | +102 | +44.16 | 10354 | 10176 | -178 | -1.72 | 10585 | 10509 | -76 | -0.72 | 23139 | 22280 | -859 | -3.71 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Before Write (ms) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) | Output (ms) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 30 | 249.77 | 0.97 | 39443 | 35416 | 0.25 | 285.61 | 35446 | 250.02 | 228.68 | 74859 |
| CSV | opt | 29 | 249.14 | 0.94 | 50036 | 37574 | 0.29 | 303.02 | 37603 | 249.42 | 242.60 | 87610 |
| JSON_COMPACT | man | 28 | 331.00 | 0.90 | 69679 | 40123 | 0.32 | 323.57 | 40151 | 331.32 | 259.04 | 109802 |
| JSON_COMPACT | opt | 10 | 874.80 | 0.32 | 41998 | 40816 | 0.24 | 329.16 | 40826 | 875.04 | 263.39 | 82813 |
| JSON_PRETTY | man | 25 | 571.32 | 0.81 | 68007 | 38019 | 0.34 | 306.60 | 38044 | 571.66 | 245.45 | 106026 |
| JSON_PRETTY | opt | 13 | 1028.23 | 0.42 | 32554 | 40608 | 0.20 | 327.48 | 40621 | 1028.43 | 262.07 | 73162 |
| TOON_DEFAULT | man | 3 | 2349.33 | 0.10 | 58363 | 37603 | 0.30 | 303.25 | 37606 | 2349.64 | 242.62 | 95966 |
| TOON_DEFAULT | opt | 5 | 2312.20 | 0.16 | 55237 | 39099 | 0.30 | 315.31 | 39104 | 2312.50 | 252.28 | 94335 |
| XML_COMPACT | man | 4 | 2923.25 | 0.13 | 38153 | 37181 | 0.24 | 299.85 | 37185 | 2923.49 | 239.90 | 75334 |
| XML_COMPACT | opt | 9 | 1214.44 | 0.29 | 57835 | 37367 | 0.30 | 301.35 | 37376 | 1214.74 | 241.14 | 95202 |
| XML_PRETTY | man | 11 | 1469.64 | 0.35 | 63055 | 39564 | 0.31 | 319.06 | 39575 | 1469.95 | 255.32 | 102619 |
| XML_PRETTY | opt | 17 | 887.59 | 0.55 | 46382 | 32583 | 0.30 | 262.77 | 32600 | 887.89 | 210.32 | 78965 |
| YAML | man | 8 | 1569.25 | 0.26 | 48750 | 39528 | 0.26 | 318.77 | 39536 | 1569.51 | 255.07 | 88278 |
| YAML | opt | 11 | 1070.09 | 0.35 | 49284 | 39265 | 0.26 | 316.65 | 39276 | 1070.35 | 253.39 | 88549 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Before Write Man (s) | Output Before Write Opt (s) | Diff (s) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 30 | 29 | -1 | -3.33 | 39.44 | 50.04 | +10.59 | +26.86 | 35.42 | 37.57 | +2.16 | +6.09 | 35.45 | 37.60 | +2.16 | +6.08 | 74.86 | 87.61 | +12.75 | +17.03 |
| JSON_COMPACT | 28 | 10 | -18 | -64.29 | 69.68 | 42.00 | -27.68 | -39.73 | 40.12 | 40.82 | +0.69 | +1.73 | 40.15 | 40.83 | +0.68 | +1.68 | 109.80 | 82.81 | -26.99 | -24.58 |
| JSON_PRETTY | 25 | 13 | -12 | -48.00 | 68.01 | 32.55 | -35.45 | -52.13 | 38.02 | 40.61 | +2.59 | +6.81 | 38.04 | 40.62 | +2.58 | +6.77 | 106.03 | 73.16 | -32.86 | -31.00 |
| TOON_DEFAULT | 3 | 5 | +2 | +66.67 | 58.36 | 55.24 | -3.13 | -5.36 | 37.60 | 39.10 | +1.50 | +3.98 | 37.61 | 39.10 | +1.50 | +3.98 | 95.97 | 94.34 | -1.63 | -1.70 |
| XML_COMPACT | 4 | 9 | +5 | +125.00 | 38.15 | 57.84 | +19.68 | +51.59 | 37.18 | 37.37 | +0.19 | +0.50 | 37.18 | 37.38 | +0.19 | +0.51 | 75.33 | 95.20 | +19.87 | +26.37 |
| XML_PRETTY | 11 | 17 | +6 | +54.55 | 63.06 | 46.38 | -16.67 | -26.44 | 39.56 | 32.58 | -6.98 | -17.64 | 39.58 | 32.60 | -6.97 | -17.62 | 102.62 | 78.97 | -23.65 | -23.05 |
| YAML | 8 | 11 | +3 | +37.50 | 48.75 | 49.28 | +0.53 | +1.10 | 39.53 | 39.26 | -0.26 | -0.67 | 39.54 | 39.28 | -0.26 | -0.66 | 88.28 | 88.55 | +0.27 | +0.31 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token | Info / Read Token (Acc By Char) | Info / Output Token (Acc By Char) | Info / Total Token (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 1.35 | 10.99 | 241.71 | 1.01 | 0.82 | 0.45 | 1.23 | 1.00 | 0.55 |
| CSV | opt | 1.33 | 11.45 | 233.07 | 1.02 | 0.66 | 0.40 | 1.30 | 0.85 | 0.52 |
| JSON_COMPACT | man | 2.15 | 13.59 | 298.97 | 0.79 | 0.56 | 0.33 | 1.05 | 0.74 | 0.44 |
| JSON_COMPACT | opt | 2.11 | 13.86 | 282.19 | 0.89 | 0.78 | 0.42 | 1.10 | 0.96 | 0.51 |
| JSON_PRETTY | man | 1.69 | 20.94 | 460.74 | 0.57 | 0.62 | 0.30 | 0.69 | 0.75 | 0.36 |
| JSON_PRETTY | opt | 1.68 | 21.18 | 431.19 | 0.57 | 0.92 | 0.35 | 0.72 | 1.16 | 0.45 |
| TOON_DEFAULT | man | 1.44 | 10.33 | 227.36 | 1.11 | 0.69 | 0.42 | 1.31 | 0.81 | 0.50 |
| TOON_DEFAULT | opt | 1.70 | 18.32 | 372.94 | 0.69 | 0.71 | 0.34 | 0.83 | 0.86 | 0.42 |
| XML_COMPACT | man | 2.37 | 17.15 | 377.19 | 0.65 | 0.80 | 0.36 | 0.83 | 1.03 | 0.46 |
| XML_COMPACT | opt | 2.34 | 17.32 | 352.58 | 0.72 | 0.69 | 0.35 | 0.90 | 0.85 | 0.44 |
| XML_PRETTY | man | 1.93 | 23.70 | 521.48 | 0.47 | 0.60 | 0.27 | 0.60 | 0.76 | 0.34 |
| XML_PRETTY | opt | 1.92 | 23.91 | 486.74 | 0.53 | 0.79 | 0.32 | 0.65 | 0.97 | 0.39 |
| YAML | man | 1.66 | 18.41 | 404.97 | 0.60 | 0.71 | 0.32 | 0.77 | 0.92 | 0.42 |
| YAML | opt | 1.65 | 18.66 | 379.71 | 0.66 | 0.74 | 0.35 | 0.83 | 0.93 | 0.44 |

#### 2.5.2 Characters And Values: Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.35 | 1.33 | -0.02 | -1.56 | 10.99 | 11.45 | +0.46 | +4.21 | 241.71 | 233.07 | -8.65 | -3.58 |
| JSON_COMPACT | 2.15 | 2.11 | -0.04 | -1.68 | 13.59 | 13.86 | +0.28 | +2.02 | 298.97 | 282.19 | -16.77 | -5.61 |
| JSON_PRETTY | 1.69 | 1.68 | -0.01 | -0.83 | 20.94 | 21.18 | +0.24 | +1.15 | 460.74 | 431.19 | -29.55 | -6.41 |
| TOON_DEFAULT | 1.44 | 1.70 | +0.26 | +17.75 | 10.33 | 18.32 | +7.99 | +77.30 | 227.36 | 372.94 | +145.58 | +64.03 |
| XML_COMPACT | 2.37 | 2.34 | -0.03 | -1.10 | 17.15 | 17.32 | +0.18 | +1.03 | 377.19 | 352.58 | -24.61 | -6.53 |
| XML_PRETTY | 1.93 | 1.92 | -0.02 | -0.88 | 23.70 | 23.91 | +0.21 | +0.88 | 521.48 | 486.74 | -34.74 | -6.66 |
| YAML | 1.66 | 1.65 | -0.01 | -0.90 | 18.41 | 18.66 | +0.25 | +1.34 | 404.97 | 379.71 | -25.26 | -6.24 |

#### 2.5.3 Information: Mandatory vs Optional
| Format | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.01 | 1.02 | 0.00 | 0.00 | 0.82 | 0.66 | -0.16 | -19.22 | 0.45 | 0.40 | -0.05 | -11.67 |
| JSON_COMPACT | 0.79 | 0.89 | +0.10 | +12.50 | 0.56 | 0.78 | +0.22 | +39.61 | 0.33 | 0.42 | +0.09 | +27.22 |
| JSON_PRETTY | 0.57 | 0.57 | 0.00 | 0.00 | 0.62 | 0.92 | +0.30 | +48.70 | 0.30 | 0.35 | +0.06 | +18.98 |
| TOON_DEFAULT | 1.11 | 0.69 | -0.43 | -38.40 | 0.69 | 0.71 | +0.02 | +2.61 | 0.42 | 0.34 | -0.08 | -18.89 |
| XML_COMPACT | 0.65 | 0.72 | +0.07 | +11.57 | 0.80 | 0.69 | -0.12 | -14.55 | 0.36 | 0.35 | -0.01 | -1.95 |
| XML_PRETTY | 0.47 | 0.53 | +0.05 | +11.18 | 0.60 | 0.79 | +0.19 | +32.39 | 0.27 | 0.32 | +0.05 | +19.62 |
| YAML | 0.60 | 0.66 | +0.06 | +10.55 | 0.71 | 0.74 | +0.03 | +4.23 | 0.32 | 0.35 | +0.02 | +7.72 |

#### 2.5.4 Information (Accuracy By Character): Mandatory vs Optional
| Format | Info / Read Token (Acc By Char) Man | Info / Read Token (Acc By Char) Opt | Diff | Diff (%) | Info / Output Token (Acc By Char) Man | Info / Output Token (Acc By Char)  Opt | Diff | Diff (%) | Info / Total Token (Acc By Char) Man | Info / Total Token (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.23 | 1.30 | +0.07 | +5.60 | 1.00 | 0.85 | -0.15 | -14.73 | 0.55 | 0.52 | -0.04 | -6.70 |
| JSON_COMPACT | 1.05 | 1.10 | +0.04 | +4.28 | 0.74 | 0.96 | +0.22 | +29.42 | 0.44 | 0.51 | +0.08 | +17.70 |
| JSON_PRETTY | 0.69 | 0.72 | +0.04 | +5.40 | 0.75 | 1.16 | +0.42 | +56.11 | 0.36 | 0.45 | +0.09 | +24.93 |
| TOON_DEFAULT | 1.31 | 0.83 | -0.48 | -36.90 | 0.81 | 0.86 | +0.04 | +5.30 | 0.50 | 0.42 | -0.08 | -16.67 |
| XML_COMPACT | 0.83 | 0.90 | +0.07 | +8.34 | 1.03 | 0.85 | -0.17 | -16.96 | 0.46 | 0.44 | -0.02 | -4.59 |
| XML_PRETTY | 0.60 | 0.65 | +0.05 | +7.67 | 0.76 | 0.97 | +0.21 | +27.93 | 0.34 | 0.39 | +0.05 | +15.82 |
| YAML | 0.77 | 0.83 | +0.06 | +7.77 | 0.92 | 0.93 | +0.02 | +1.75 | 0.42 | 0.44 | +0.02 | +4.77 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7493 | 5700 | 1793 | 9256 | 7041 | 2215 | 16749 | 12741 | 4008 | 76.07 | 82.39 | 78.79 | 84.02 | 73.82 | 80.89 | 77.29 | 82.52 |
| CSV | opt | 7225 | 5302 | 1923 | 11057 | 8114 | 2942 | 18282 | 13417 | 4865 | 73.39 | 81.58 | 67.19 | 78.05 | 72.81 | 81.19 | 66.80 | 77.66 |
| JSON_COMPACT | man | 9268 | 6802 | 2466 | 13161 | 9659 | 3502 | 22429 | 16460 | 5968 | 73.39 | 74.13 | 55.72 | 66.74 | 71.95 | 73.17 | 54.76 | 65.78 |
| JSON_COMPACT | opt | 8748 | 6820 | 1928 | 10005 | 7800 | 2205 | 18753 | 14620 | 4133 | 77.96 | 79.07 | 75.97 | 79.81 | 78.47 | 79.41 | 76.31 | 80.15 |
| JSON_PRETTY | man | 14283 | 11556 | 2727 | 13138 | 10630 | 2508 | 27421 | 22187 | 5235 | 80.91 | 60.85 | 60.85 | 58.13 | 79.55 | 59.94 | 59.94 | 57.22 |
| JSON_PRETTY | opt | 13367 | 10168 | 3199 | 8302 | 6315 | 1987 | 21669 | 16484 | 5185 | 76.07 | 60.96 | 83.99 | 70.60 | 75.54 | 60.61 | 83.64 | 70.24 |
| TOON_DEFAULT | man | 7048 | 5523 | 1525 | 11671 | 9145 | 2526 | 18719 | 14668 | 4051 | 78.36 | 85.54 | 67.15 | 80.17 | 76.25 | 84.13 | 65.75 | 78.77 |
| TOON_DEFAULT | opt | 11561 | 9153 | 2408 | 11768 | 9316 | 2451 | 23329 | 18469 | 4859 | 79.17 | 69.61 | 67.16 | 68.13 | 79.29 | 69.69 | 67.24 | 68.21 |
| XML_COMPACT | man | 11693 | 8864 | 2829 | 9429 | 7148 | 2281 | 21122 | 16012 | 5109 | 75.81 | 66.89 | 77.68 | 71.92 | 74.68 | 66.14 | 76.92 | 71.16 |
| XML_COMPACT | opt | 10930 | 8638 | 2292 | 11498 | 9087 | 2411 | 22428 | 17725 | 4703 | 79.03 | 71.82 | 68.54 | 70.50 | 78.30 | 71.33 | 68.05 | 70.01 |
| XML_PRETTY | man | 16166 | 12385 | 3781 | 12780 | 9791 | 2989 | 28946 | 22176 | 6771 | 76.61 | 51.11 | 59.94 | 51.10 | 75.94 | 50.66 | 59.49 | 50.65 |
| XML_PRETTY | opt | 15089 | 12006 | 3083 | 10028 | 7980 | 2049 | 25117 | 19986 | 5131 | 79.57 | 57.01 | 76.91 | 63.52 | 77.56 | 55.67 | 75.57 | 62.18 |
| YAML | man | 12554 | 9416 | 3139 | 10585 | 7939 | 2646 | 23139 | 17354 | 5785 | 75.00 | 63.21 | 70.83 | 65.87 | 73.42 | 62.16 | 69.78 | 64.82 |
| YAML | opt | 11771 | 9145 | 2626 | 10509 | 8165 | 2345 | 22280 | 17310 | 4971 | 77.69 | 67.86 | 73.04 | 70.01 | 77.83 | 67.95 | 73.13 | 70.10 |

#### 2.6.2 Read Tokens: Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7493 | 7225 | -268 | -3.58 | 5700 | 5303 | -397 | -6.97 | 1793 | 1922 | +129 | +7.22 | 76.07 | 73.39 | -2.68 | 82.39 | 81.58 | -0.81 | -0.98 | 73.82 | 72.81 | -1.01 | 80.89 | 81.19 | +0.30 | +0.38 |
| JSON_COMPACT | 9268 | 8748 | -520 | -5.61 | 6802 | 6820 | +18 | +0.27 | 2466 | 1928 | -538 | -21.82 | 73.39 | 77.96 | +4.57 | 74.13 | 79.07 | +4.94 | +6.67 | 71.95 | 78.47 | +6.52 | 73.17 | 79.41 | +6.24 | +8.53 |
| JSON_PRETTY | 14283 | 13367 | -916 | -6.41 | 11556 | 10168 | -1388 | -12.01 | 2727 | 3199 | +472 | +17.31 | 80.91 | 76.07 | -4.84 | 60.85 | 60.96 | +0.11 | +0.19 | 79.55 | 75.54 | -4.01 | 59.94 | 60.61 | +0.67 | +1.11 |
| TOON_DEFAULT | 7048 | 11561 | +4513 | +64.03 | 5523 | 9153 | +3630 | +65.73 | 1525 | 2408 | +883 | +57.90 | 78.36 | 79.17 | +0.81 | 85.54 | 69.61 | -15.92 | -18.61 | 76.25 | 79.29 | +3.04 | 84.13 | 69.69 | -14.44 | -17.16 |
| XML_COMPACT | 11693 | 10930 | -763 | -6.53 | 8864 | 8638 | -226 | -2.56 | 2829 | 2292 | -537 | -18.96 | 75.81 | 79.03 | +3.22 | 66.89 | 71.82 | +4.93 | +7.37 | 74.68 | 78.30 | +3.62 | 66.14 | 71.33 | +5.20 | +7.86 |
| XML_PRETTY | 16166 | 15089 | -1077 | -6.66 | 12385 | 12007 | -378 | -3.06 | 3781 | 3082 | -699 | -18.48 | 76.61 | 79.57 | +2.96 | 51.11 | 57.01 | +5.90 | +11.55 | 75.94 | 77.56 | +1.62 | 50.66 | 55.67 | +5.01 | +9.88 |
| YAML | 12554 | 11771 | -783 | -6.24 | 9416 | 9145 | -271 | -2.87 | 3139 | 2627 | -512 | -16.32 | 75.00 | 77.69 | +2.69 | 63.21 | 67.86 | +4.65 | +7.35 | 73.42 | 77.83 | +4.41 | 62.16 | 67.95 | +5.80 | +9.32 |

#### 2.6.3 Output Tokens: Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 9256 | 11056 | +1800 | +19.45 | 7041 | 8114 | +1073 | +15.24 | 2215 | 2942 | +727 | +32.83 | 76.07 | 73.39 | -2.68 | 78.79 | 67.19 | -11.60 | -14.73 | 73.82 | 72.81 | -1.01 | 77.29 | 66.80 | -10.49 | -13.57 |
| JSON_COMPACT | 13161 | 10006 | -3155 | -23.97 | 9659 | 7801 | -1858 | -19.24 | 3502 | 2205 | -1297 | -37.03 | 73.39 | 77.96 | +4.57 | 55.72 | 75.97 | +20.25 | +36.34 | 71.95 | 78.47 | +6.52 | 54.76 | 76.31 | +21.55 | +39.35 |
| JSON_PRETTY | 13138 | 8302 | -4836 | -36.81 | 10630 | 6315 | -4315 | -40.59 | 2508 | 1987 | -521 | -20.79 | 80.91 | 76.07 | -4.84 | 60.85 | 83.99 | +23.14 | +38.03 | 79.55 | 75.54 | -4.01 | 59.94 | 83.64 | +23.69 | +39.53 |
| TOON_DEFAULT | 11671 | 11768 | +97 | +0.83 | 9145 | 9316 | +171 | +1.87 | 2526 | 2452 | -74 | -2.95 | 78.36 | 79.17 | +0.81 | 67.15 | 67.16 | +0.01 | +0.02 | 76.25 | 79.29 | +3.04 | 65.75 | 67.24 | +1.50 | +2.28 |
| XML_COMPACT | 9429 | 11499 | +2070 | +21.95 | 7148 | 9087 | +1939 | +27.13 | 2281 | 2411 | +130 | +5.72 | 75.81 | 79.03 | +3.22 | 77.68 | 68.54 | -9.14 | -11.76 | 74.68 | 78.30 | +3.62 | 76.92 | 68.05 | -8.87 | -11.53 |
| XML_PRETTY | 12780 | 10028 | -2752 | -21.53 | 9791 | 7980 | -1811 | -18.50 | 2989 | 2048 | -941 | -31.47 | 76.61 | 79.57 | +2.96 | 59.94 | 76.91 | +16.98 | +28.33 | 75.94 | 77.56 | +1.62 | 59.49 | 75.57 | +16.08 | +27.04 |
| YAML | 10585 | 10509 | -76 | -0.71 | 7939 | 8165 | +226 | +2.85 | 2646 | 2344 | -302 | -11.40 | 75.00 | 77.69 | +2.69 | 70.83 | 73.04 | +2.21 | +3.11 | 73.42 | 77.83 | +4.41 | 69.78 | 73.13 | +3.35 | +4.80 |

#### 2.6.4 Total Tokens: Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 16749 | 18281 | +1532 | +9.15 | 12741 | 13417 | +676 | +5.30 | 4008 | 4865 | +857 | +21.37 | 76.07 | 73.39 | -2.68 | 84.02 | 78.05 | -5.97 | -7.10 | 73.82 | 72.81 | -1.01 | 82.52 | 77.66 | -4.86 | -5.88 |
| JSON_COMPACT | 22429 | 18754 | -3675 | -16.39 | 16460 | 14620 | -1840 | -11.18 | 5968 | 4133 | -1835 | -30.75 | 73.39 | 77.96 | +4.57 | 66.74 | 79.81 | +13.08 | +19.59 | 71.95 | 78.47 | +6.52 | 65.78 | 80.15 | +14.38 | +21.85 |
| JSON_PRETTY | 27421 | 21669 | -5752 | -20.98 | 22187 | 16484 | -5703 | -25.70 | 5235 | 5186 | -49 | -0.94 | 80.91 | 76.07 | -4.84 | 58.13 | 70.60 | +12.47 | +21.45 | 79.55 | 75.54 | -4.01 | 57.22 | 70.24 | +13.02 | +22.76 |
| TOON_DEFAULT | 18719 | 23329 | +4610 | +24.63 | 14668 | 18469 | +3801 | +25.91 | 4051 | 4860 | +809 | +19.96 | 78.36 | 79.17 | +0.81 | 80.17 | 68.13 | -12.04 | -15.01 | 76.25 | 79.29 | +3.04 | 78.77 | 68.21 | -10.55 | -13.39 |
| XML_COMPACT | 21122 | 22429 | +1307 | +6.19 | 16012 | 17725 | +1713 | +10.70 | 5109 | 4703 | -406 | -7.95 | 75.81 | 79.03 | +3.22 | 71.92 | 70.50 | -1.42 | -1.97 | 74.68 | 78.30 | +3.62 | 71.16 | 70.01 | -1.15 | -1.62 |
| XML_PRETTY | 28946 | 25117 | -3829 | -13.23 | 22176 | 19986 | -2190 | -9.88 | 6771 | 5132 | -1639 | -24.21 | 76.61 | 79.57 | +2.96 | 51.10 | 63.52 | +12.42 | +24.31 | 75.94 | 77.56 | +1.62 | 50.65 | 62.18 | +11.53 | +22.76 |
| YAML | 23139 | 22280 | -859 | -3.71 | 17354 | 17309 | -45 | -0.26 | 5785 | 4971 | -814 | -14.07 | 75.00 | 77.69 | +2.69 | 65.87 | 70.01 | +4.14 | +6.28 | 73.42 | 77.83 | +4.41 | 64.82 | 70.10 | +5.28 | +8.15 |

### 2.7 Token Utilization Efficiency (Accuracy by Character)
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy by Character (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy by Character (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7493 | 6924 | 569 | 9256 | 8553 | 703 | 16749 | 15476 | 1273 | 92.40 | 93.27 | 89.68 | 94.91 | 90.71 | 92.15 | 88.55 | 93.78 |
| CSV | opt | 7225 | 6796 | 429 | 11057 | 10400 | 657 | 18282 | 17196 | 1086 | 94.06 | 95.36 | 80.97 | 91.83 | 89.56 | 92.36 | 77.97 | 88.83 |
| JSON_COMPACT | man | 9268 | 9034 | 234 | 13161 | 12828 | 333 | 22429 | 21861 | 567 | 97.47 | 90.18 | 71.77 | 82.79 | 93.11 | 87.27 | 68.86 | 79.88 |
| JSON_COMPACT | opt | 8748 | 8394 | 354 | 10005 | 9600 | 405 | 18753 | 17994 | 760 | 95.95 | 91.06 | 87.96 | 91.80 | 92.67 | 88.88 | 85.77 | 89.62 |
| JSON_PRETTY | man | 14283 | 13973 | 310 | 13138 | 12853 | 285 | 27421 | 26826 | 595 | 97.83 | 72.13 | 72.13 | 69.41 | 94.80 | 70.11 | 70.11 | 67.39 |
| JSON_PRETTY | opt | 13367 | 12909 | 458 | 8302 | 8017 | 285 | 21669 | 20926 | 743 | 96.57 | 74.63 | 97.66 | 84.26 | 91.90 | 71.51 | 94.55 | 81.15 |
| TOON_DEFAULT | man | 7048 | 6501 | 547 | 11671 | 10765 | 906 | 18719 | 17266 | 1453 | 92.24 | 94.79 | 76.41 | 89.43 | 91.95 | 94.60 | 76.21 | 89.23 |
| TOON_DEFAULT | opt | 11561 | 11047 | 514 | 11768 | 11244 | 524 | 23329 | 22290 | 1038 | 95.55 | 80.53 | 78.08 | 79.05 | 92.32 | 78.38 | 75.93 | 76.90 |
| XML_COMPACT | man | 11693 | 11307 | 386 | 9429 | 9118 | 311 | 21122 | 20425 | 697 | 96.70 | 80.82 | 91.60 | 85.84 | 93.79 | 78.88 | 89.66 | 83.90 |
| XML_COMPACT | opt | 10930 | 10704 | 226 | 11498 | 11260 | 238 | 22428 | 21964 | 464 | 97.93 | 84.42 | 81.14 | 83.10 | 94.03 | 81.82 | 78.54 | 80.50 |
| XML_PRETTY | man | 16166 | 15678 | 488 | 12780 | 12394 | 386 | 28946 | 28072 | 874 | 96.98 | 64.69 | 73.52 | 64.68 | 93.65 | 62.47 | 71.30 | 62.46 |
| XML_PRETTY | opt | 15089 | 14700 | 389 | 10028 | 9770 | 259 | 25117 | 24469 | 648 | 97.42 | 68.91 | 88.81 | 75.42 | 94.05 | 66.66 | 86.57 | 73.17 |
| YAML | man | 12554 | 12170 | 384 | 10585 | 10261 | 324 | 23139 | 22431 | 708 | 96.94 | 77.84 | 85.46 | 80.50 | 94.05 | 75.91 | 83.53 | 78.57 |
| YAML | opt | 11771 | 11525 | 246 | 10509 | 10290 | 220 | 22280 | 21815 | 466 | 97.91 | 81.34 | 86.52 | 83.49 | 93.35 | 78.30 | 83.48 | 80.45 |

#### 2.7.2 Read Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7493 | 7225 | -268 | -3.58 | 6924 | 6796 | -128 | -1.84 | 569 | 429 | -140 | -24.66 | 92.40 | 94.06 | +1.66 | 93.27 | 95.36 | +2.08 | +2.23 | 90.71 | 89.56 | -1.15 | 92.15 | 92.36 | +0.21 | +0.23 |
| JSON_COMPACT | 9268 | 8748 | -520 | -5.61 | 9034 | 8394 | -640 | -7.08 | 234 | 354 | +120 | +51.20 | 97.47 | 95.95 | -1.52 | 90.18 | 91.06 | +0.88 | +0.98 | 93.11 | 92.67 | -0.44 | 87.27 | 88.88 | +1.60 | +1.84 |
| JSON_PRETTY | 14283 | 13367 | -916 | -6.41 | 13973 | 12908 | -1065 | -7.62 | 310 | 459 | +149 | +47.92 | 97.83 | 96.57 | -1.26 | 72.13 | 74.63 | +2.50 | +3.47 | 94.80 | 91.90 | -2.90 | 70.11 | 71.51 | +1.41 | +2.01 |
| TOON_DEFAULT | 7048 | 11561 | +4513 | +64.03 | 6501 | 11046 | +4545 | +69.92 | 547 | 515 | -32 | -5.93 | 92.24 | 95.55 | +3.31 | 94.79 | 80.53 | -14.25 | -15.04 | 91.95 | 92.32 | +0.37 | 94.60 | 78.38 | -16.22 | -17.14 |
| XML_COMPACT | 11693 | 10930 | -763 | -6.53 | 11307 | 10704 | -603 | -5.34 | 386 | 226 | -160 | -41.35 | 96.70 | 97.93 | +1.23 | 80.82 | 84.42 | +3.60 | +4.46 | 93.79 | 94.03 | +0.24 | 78.88 | 81.82 | +2.94 | +3.73 |
| XML_PRETTY | 16166 | 15089 | -1077 | -6.66 | 15678 | 14700 | -978 | -6.24 | 488 | 389 | -99 | -20.27 | 96.98 | 97.42 | +0.44 | 64.69 | 68.91 | +4.22 | +6.53 | 93.65 | 94.05 | +0.40 | 62.47 | 66.66 | +4.20 | +6.72 |
| YAML | 12554 | 11771 | -783 | -6.24 | 12170 | 11525 | -645 | -5.30 | 384 | 246 | -138 | -35.97 | 96.94 | 97.91 | +0.97 | 77.84 | 81.34 | +3.50 | +4.50 | 94.05 | 93.35 | -0.70 | 75.91 | 78.30 | +2.39 | +3.15 |

#### 2.7.3 Output Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 9256 | 11056 | +1800 | +19.45 | 8553 | 10400 | +1847 | +21.60 | 703 | 656 | -47 | -6.65 | 92.40 | 94.06 | +1.66 | 89.68 | 80.97 | -8.71 | -9.71 | 90.71 | 89.56 | -1.15 | 88.55 | 77.97 | -10.58 | -11.95 |
| JSON_COMPACT | 13161 | 10006 | -3155 | -23.97 | 12828 | 9600 | -3228 | -25.16 | 333 | 405 | +72 | +21.70 | 97.47 | 95.95 | -1.52 | 71.77 | 87.96 | +16.19 | +22.56 | 93.11 | 92.67 | -0.44 | 68.86 | 85.77 | +16.91 | +24.55 |
| JSON_PRETTY | 13138 | 8302 | -4836 | -36.81 | 12853 | 8017 | -4836 | -37.63 | 285 | 285 | 0 | -0.12 | 97.83 | 96.57 | -1.26 | 72.13 | 97.66 | +25.53 | +35.39 | 94.80 | 91.90 | -2.90 | 70.11 | 94.55 | +24.43 | +34.85 |
| TOON_DEFAULT | 11671 | 11768 | +97 | +0.83 | 10765 | 11244 | +479 | +4.45 | 906 | 524 | -382 | -42.16 | 92.24 | 95.55 | +3.31 | 76.41 | 78.08 | +1.68 | +2.20 | 91.95 | 92.32 | +0.37 | 76.21 | 75.93 | -0.28 | -0.37 |
| XML_COMPACT | 9429 | 11499 | +2070 | +21.95 | 9118 | 11261 | +2143 | +23.50 | 311 | 238 | -73 | -23.51 | 96.70 | 97.93 | +1.23 | 91.60 | 81.14 | -10.46 | -11.42 | 93.79 | 94.03 | +0.24 | 89.66 | 78.54 | -11.12 | -12.41 |
| XML_PRETTY | 12780 | 10028 | -2752 | -21.53 | 12394 | 9769 | -2625 | -21.18 | 386 | 259 | -127 | -32.96 | 96.98 | 97.42 | +0.44 | 73.52 | 88.81 | +15.30 | +20.81 | 93.65 | 94.05 | +0.40 | 71.30 | 86.57 | +15.27 | +21.42 |
| YAML | 10585 | 10509 | -76 | -0.71 | 10261 | 10290 | +29 | +0.28 | 324 | 220 | -104 | -32.18 | 96.94 | 97.91 | +0.97 | 85.46 | 86.52 | +1.06 | +1.24 | 94.05 | 93.35 | -0.70 | 83.53 | 83.48 | -0.05 | -0.06 |

#### 2.7.4 Total Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 16749 | 18281 | +1532 | +9.15 | 15476 | 17195 | +1719 | +11.11 | 1273 | 1086 | -187 | -14.69 | 92.40 | 94.06 | +1.66 | 94.91 | 91.83 | -3.07 | -3.24 | 90.71 | 89.56 | -1.15 | 93.78 | 88.83 | -4.95 | -5.28 |
| JSON_COMPACT | 22429 | 18754 | -3675 | -16.39 | 21861 | 17994 | -3867 | -17.69 | 567 | 759 | +192 | +33.87 | 97.47 | 95.95 | -1.52 | 82.79 | 91.80 | +9.01 | +10.89 | 93.11 | 92.67 | -0.44 | 79.88 | 89.62 | +9.73 | +12.19 |
| JSON_PRETTY | 27421 | 21669 | -5752 | -20.98 | 26826 | 20925 | -5901 | -22.00 | 595 | 743 | +148 | +24.91 | 97.83 | 96.57 | -1.26 | 69.41 | 84.26 | +14.86 | +21.40 | 94.80 | 91.90 | -2.90 | 67.39 | 81.15 | +13.76 | +20.42 |
| TOON_DEFAULT | 18719 | 23329 | +4610 | +24.63 | 17266 | 22290 | +5024 | +29.10 | 1453 | 1039 | -414 | -28.52 | 92.24 | 95.55 | +3.31 | 89.43 | 79.05 | -10.37 | -11.60 | 91.95 | 92.32 | +0.37 | 89.23 | 76.90 | -12.33 | -13.82 |
| XML_COMPACT | 21122 | 22429 | +1307 | +6.19 | 20425 | 21964 | +1539 | +7.54 | 697 | 464 | -233 | -33.39 | 96.70 | 97.93 | +1.23 | 85.84 | 83.10 | -2.75 | -3.20 | 93.79 | 94.03 | +0.24 | 83.90 | 80.50 | -3.41 | -4.06 |
| XML_PRETTY | 28946 | 25117 | -3829 | -13.23 | 28072 | 24469 | -3603 | -12.83 | 874 | 648 | -226 | -25.88 | 96.98 | 97.42 | +0.44 | 64.68 | 75.42 | +10.74 | +16.60 | 93.65 | 94.05 | +0.40 | 62.46 | 73.17 | +10.71 | +17.15 |
| YAML | 23139 | 22280 | -859 | -3.71 | 22431 | 21815 | -616 | -2.75 | 708 | 466 | -242 | -34.24 | 96.94 | 97.91 | +0.97 | 80.50 | 83.49 | +2.99 | +3.71 | 94.05 | 93.35 | -0.70 | 78.57 | 80.45 | +1.88 | +2.39 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Expected Characters | Output Characters | Correct Characters | Incorrect Characters | Accuracy by Character (%) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 94 | 30 | 0 | 76.07 | 7782 | 8103 | 7481 | 623 | 92.40 |
| CSV | opt | 91 | 33 | 0 | 73.39 | 8451 | 8686 | 8168 | 518 | 94.06 |
| JSON_COMPACT | man | 91 | 33 | 0 | 73.39 | 7782 | 7849 | 7650 | 198 | 97.47 |
| JSON_COMPACT | opt | 97 | 27 | 0 | 77.96 | 8451 | 8637 | 8286 | 350 | 95.95 |
| JSON_PRETTY | man | 100 | 24 | 0 | 80.91 | 7782 | 7834 | 7664 | 170 | 97.83 |
| JSON_PRETTY | opt | 94 | 30 | 0 | 76.07 | 8451 | 8569 | 8273 | 296 | 96.57 |
| TOON_DEFAULT | man | 97 | 27 | 0 | 78.36 | 7782 | 8128 | 7467 | 661 | 92.24 |
| TOON_DEFAULT | opt | 98 | 26 | 0 | 79.17 | 8451 | 8632 | 8243 | 388 | 95.55 |
| XML_COMPACT | man | 94 | 30 | 0 | 75.81 | 7782 | 7879 | 7616 | 263 | 96.70 |
| XML_COMPACT | opt | 98 | 26 | 0 | 79.03 | 8451 | 8534 | 8357 | 177 | 97.93 |
| XML_PRETTY | man | 95 | 29 | 0 | 76.61 | 7782 | 7836 | 7599 | 237 | 96.98 |
| XML_PRETTY | opt | 99 | 25 | 0 | 79.57 | 8451 | 8557 | 8336 | 221 | 97.42 |
| YAML | man | 93 | 31 | 0 | 75.00 | 7782 | 7863 | 7623 | 241 | 96.94 |
| YAML | opt | 96 | 28 | 0 | 77.69 | 8451 | 8526 | 8348 | 178 | 97.91 |

#### 2.8.2 Answers: Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 94 | 91 | -3 | -3.54 | 30 | 33 | +3 | +11.10 | 0 | 0 | 0 | 0.00 | 76.07 | 73.39 | -2.68 |
| JSON_COMPACT | 91 | 97 | +6 | +6.23 | 33 | 27 | -6 | -17.18 | 0 | 0 | 0 | 0.00 | 73.39 | 77.96 | +4.57 |
| JSON_PRETTY | 100 | 94 | -6 | -6.00 | 24 | 30 | +6 | +25.00 | 0 | 0 | 0 | 0.00 | 80.91 | 76.07 | -4.84 |
| TOON_DEFAULT | 97 | 98 | +1 | +1.03 | 27 | 26 | -1 | -3.70 | 0 | 0 | 0 | 0.00 | 78.36 | 79.17 | +0.81 |
| XML_COMPACT | 94 | 98 | +4 | +4.26 | 30 | 26 | -4 | -13.33 | 0 | 0 | 0 | 0.00 | 75.81 | 79.03 | +3.22 |
| XML_PRETTY | 95 | 99 | +4 | +3.86 | 29 | 25 | -4 | -12.66 | 0 | 0 | 0 | 0.00 | 76.61 | 79.57 | +2.96 |
| YAML | 93 | 96 | +3 | +3.58 | 31 | 28 | -3 | -10.74 | 0 | 0 | 0 | 0.00 | 75.00 | 77.69 | +2.69 |

#### 2.8.3 Characters: Mandatory vs Optional Data
| Format | Output Characters Man | Output Characters Opt | Diff | Diff (%) | Correct Characters Man | Correct Characters Opt | Diff | Diff (%) | Incorrect Characters Man | Incorrect Characters Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 8103 | 8686 | +583 | +7.19 | 7481 | 8168 | +687 | +9.19 | 623 | 519 | -104 | -16.75 | 92.40 | 94.06 | 1.66 |
| JSON_COMPACT | 7849 | 8637 | +788 | +10.04 | 7650 | 8286 | +636 | +8.31 | 198 | 350 | +152 | +76.77 | 97.47 | 95.95 | -1.52 |
| JSON_PRETTY | 7834 | 8569 | +735 | +9.38 | 7664 | 8273 | +609 | +7.95 | 170 | 296 | +126 | +74.12 | 97.83 | 96.57 | -1.26 |
| TOON_DEFAULT | 8128 | 8632 | +504 | +6.20 | 7467 | 8244 | +777 | +10.40 | 661 | 388 | -273 | -41.28 | 92.24 | 95.55 | 3.31 |
| XML_COMPACT | 7879 | 8534 | +655 | +8.31 | 7616 | 8357 | +741 | +9.73 | 263 | 177 | -86 | -32.70 | 96.70 | 97.93 | 1.23 |
| XML_PRETTY | 7836 | 8557 | +721 | +9.21 | 7599 | 8336 | +737 | +9.70 | 237 | 221 | -16 | -6.61 | 96.98 | 97.42 | 0.44 |
| YAML | 7863 | 8526 | +663 | +8.43 | 7623 | 8348 | +725 | +9.51 | 241 | 179 | -62 | -25.86 | 96.94 | 97.91 | 0.97 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 76.07 | 87.88 | 66.66 | 57.14 | 76.19 | 90.71 | 32.95 | 19.44 | 11.90 | 9.52 |
| CSV | opt | 73.39 | 90.30 | 69.14 | 63.49 | 44.44 | 89.56 | 33.86 | 20.16 | 13.23 | 5.56 |
| JSON_COMPACT | man | 73.39 | 97.58 | 62.96 | 58.73 | 38.10 | 93.11 | 36.59 | 18.36 | 12.23 | 4.76 |
| JSON_COMPACT | opt | 77.96 | 90.30 | 81.48 | 71.43 | 47.62 | 92.67 | 33.86 | 23.77 | 14.88 | 5.95 |
| JSON_PRETTY | man | 80.91 | 98.79 | 74.08 | 65.08 | 58.73 | 94.80 | 37.05 | 21.60 | 13.56 | 7.34 |
| JSON_PRETTY | opt | 76.07 | 95.15 | 74.08 | 61.90 | 42.86 | 91.90 | 35.68 | 21.61 | 12.90 | 5.36 |
| TOON_DEFAULT | man | 78.36 | 94.55 | 64.81 | 65.08 | 66.67 | 91.95 | 35.46 | 18.90 | 13.56 | 8.33 |
| TOON_DEFAULT | opt | 79.17 | 93.94 | 79.63 | 71.43 | 47.62 | 92.32 | 35.23 | 23.23 | 14.88 | 5.95 |
| XML_COMPACT | man | 75.81 | 92.12 | 69.14 | 63.49 | 53.97 | 93.79 | 34.55 | 20.16 | 13.23 | 6.75 |
| XML_COMPACT | opt | 79.03 | 98.79 | 74.08 | 66.67 | 46.03 | 94.03 | 37.05 | 21.61 | 13.89 | 5.76 |
| XML_PRETTY | man | 76.61 | 97.57 | 69.14 | 68.26 | 39.68 | 93.65 | 36.59 | 20.16 | 14.22 | 4.96 |
| XML_PRETTY | opt | 79.57 | 98.18 | 66.67 | 65.08 | 61.90 | 94.05 | 36.82 | 19.44 | 13.56 | 7.74 |
| YAML | man | 75.00 | 98.79 | 55.56 | 73.02 | 39.68 | 94.05 | 37.05 | 16.20 | 15.21 | 4.96 |
| YAML | opt | 77.69 | 97.58 | 76.54 | 69.84 | 34.92 | 93.35 | 36.59 | 22.33 | 14.55 | 4.36 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 87.88 | 90.30 | +2.42 | 32.95 | 33.86 | +0.91 |
| JSON_COMPACT | 97.58 | 90.30 | -7.27 | 36.59 | 33.86 | -2.73 |
| JSON_PRETTY | 98.79 | 95.15 | -3.64 | 37.05 | 35.68 | -1.36 |
| TOON_DEFAULT | 94.55 | 93.94 | -0.61 | 35.46 | 35.23 | -0.23 |
| XML_COMPACT | 92.12 | 98.79 | +6.67 | 34.55 | 37.05 | +2.50 |
| XML_PRETTY | 97.57 | 98.18 | +0.61 | 36.59 | 36.82 | +0.23 |
| YAML | 98.79 | 97.58 | -1.21 | 37.05 | 36.59 | -0.46 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 66.66 | 69.14 | +2.47 | 19.44 | 20.16 | +0.72 |
| JSON_COMPACT | 62.96 | 81.48 | +18.52 | 18.36 | 23.77 | +5.41 |
| JSON_PRETTY | 74.08 | 74.08 | 0.00 | 21.60 | 21.61 | 0.00 |
| TOON_DEFAULT | 64.81 | 79.63 | +14.82 | 18.90 | 23.23 | +4.33 |
| XML_COMPACT | 69.14 | 74.08 | +4.94 | 20.16 | 21.61 | +1.44 |
| XML_PRETTY | 69.14 | 66.67 | -2.47 | 20.16 | 19.44 | -0.72 |
| YAML | 55.56 | 76.54 | +20.99 | 16.20 | 22.33 | +6.13 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 57.14 | 63.49 | +6.35 | 11.90 | 13.23 | +1.32 |
| JSON_COMPACT | 58.73 | 71.43 | +12.70 | 12.23 | 14.88 | +2.65 |
| JSON_PRETTY | 65.08 | 61.90 | -3.17 | 13.56 | 12.90 | -0.66 |
| TOON_DEFAULT | 65.08 | 71.43 | +6.35 | 13.56 | 14.88 | +1.32 |
| XML_COMPACT | 63.49 | 66.67 | +3.18 | 13.23 | 13.89 | +0.66 |
| XML_PRETTY | 68.26 | 65.08 | -3.18 | 14.22 | 13.56 | -0.66 |
| YAML | 73.02 | 69.84 | -3.18 | 15.21 | 14.55 | -0.66 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 76.19 | 44.44 | -31.75 | 9.52 | 5.56 | -3.97 |
| JSON_COMPACT | 38.10 | 47.62 | +9.52 | 4.76 | 5.95 | +1.19 |
| JSON_PRETTY | 58.73 | 42.86 | -15.87 | 7.34 | 5.36 | -1.98 |
| TOON_DEFAULT | 66.67 | 47.62 | -19.05 | 8.33 | 5.95 | -2.38 |
| XML_COMPACT | 53.97 | 46.03 | -7.93 | 6.75 | 5.76 | -0.99 |
| XML_PRETTY | 39.68 | 61.90 | +22.22 | 4.96 | 7.74 | +2.77 |
| YAML | 39.68 | 34.92 | -4.76 | 4.96 | 4.36 | -0.60 |

### 2.10 Accuracy By Character Per Question Category Analysis
#### 2.10.1 Metrics
| Format | Variant | Accuracy By Character (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc By Char (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 92.40 | 87.47 | 96.39 | 90.18 | 88.06 | 90.71 | 32.80 | 28.11 | 18.78 | 11.01 |
| CSV | opt | 94.06 | 91.08 | 97.70 | 86.14 | 71.72 | 89.56 | 34.15 | 28.50 | 17.94 | 8.97 |
| JSON_COMPACT | man | 97.47 | 98.97 | 97.53 | 87.80 | 74.07 | 93.11 | 37.11 | 28.45 | 18.29 | 9.26 |
| JSON_COMPACT | opt | 95.95 | 94.66 | 97.96 | 92.92 | 73.94 | 92.67 | 35.50 | 28.58 | 19.36 | 9.24 |
| JSON_PRETTY | man | 97.83 | 98.58 | 98.04 | 90.78 | 82.72 | 94.80 | 36.97 | 28.59 | 18.91 | 10.34 |
| JSON_PRETTY | opt | 96.57 | 96.74 | 97.63 | 87.61 | 71.11 | 91.90 | 36.28 | 28.47 | 18.25 | 8.89 |
| TOON_DEFAULT | man | 92.24 | 95.59 | 90.35 | 93.01 | 83.03 | 91.95 | 35.85 | 26.35 | 19.37 | 10.38 |
| TOON_DEFAULT | opt | 95.55 | 93.72 | 98.08 | 92.92 | 73.64 | 92.32 | 35.15 | 28.61 | 19.36 | 9.20 |
| XML_COMPACT | man | 96.70 | 96.13 | 97.86 | 92.26 | 79.84 | 93.79 | 36.05 | 28.54 | 19.22 | 9.98 |
| XML_COMPACT | opt | 97.93 | 99.65 | 97.56 | 91.15 | 73.74 | 94.03 | 37.37 | 28.45 | 18.99 | 9.22 |
| XML_PRETTY | man | 96.98 | 96.33 | 98.31 | 93.15 | 75.52 | 93.65 | 36.13 | 28.68 | 19.41 | 9.44 |
| XML_PRETTY | opt | 97.42 | 99.03 | 96.95 | 89.97 | 79.19 | 94.05 | 37.14 | 28.28 | 18.74 | 9.90 |
| YAML | man | 96.94 | 99.47 | 96.16 | 94.05 | 72.84 | 94.05 | 37.30 | 28.05 | 19.59 | 9.10 |
| YAML | opt | 97.91 | 99.12 | 98.14 | 90.86 | 69.09 | 93.35 | 37.17 | 28.62 | 18.93 | 8.63 |

#### 2.10.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 87.47 | 91.08 | +3.61 | 32.80 | 34.15 | +1.35 |
| JSON_COMPACT | 98.97 | 94.66 | -4.31 | 37.11 | 35.50 | -1.62 |
| JSON_PRETTY | 98.58 | 96.74 | -1.84 | 36.97 | 36.28 | -0.69 |
| TOON_DEFAULT | 95.59 | 93.72 | -1.87 | 35.85 | 35.15 | -0.70 |
| XML_COMPACT | 96.13 | 99.65 | +3.51 | 36.05 | 37.37 | +1.32 |
| XML_PRETTY | 96.33 | 99.03 | +2.70 | 36.13 | 37.14 | +1.01 |
| YAML | 99.47 | 99.12 | -0.35 | 37.30 | 37.17 | -0.14 |

#### 2.10.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 96.39 | 97.70 | +1.31 | 28.11 | 28.50 | +0.38 |
| JSON_COMPACT | 97.53 | 97.96 | +0.43 | 28.45 | 28.58 | +0.13 |
| JSON_PRETTY | 98.04 | 97.63 | -0.41 | 28.59 | 28.47 | -0.12 |
| TOON_DEFAULT | 90.35 | 98.08 | +7.73 | 26.35 | 28.61 | +2.26 |
| XML_COMPACT | 97.86 | 97.56 | -0.30 | 28.54 | 28.45 | -0.09 |
| XML_PRETTY | 98.31 | 96.95 | -1.36 | 28.68 | 28.28 | -0.40 |
| YAML | 96.16 | 98.14 | +1.99 | 28.05 | 28.62 | +0.58 |

#### 2.10.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 90.18 | 86.14 | -4.04 | 18.78 | 17.94 | -0.84 |
| JSON_COMPACT | 87.80 | 92.92 | +5.12 | 18.29 | 19.36 | +1.07 |
| JSON_PRETTY | 90.78 | 87.61 | -3.16 | 18.91 | 18.25 | -0.65 |
| TOON_DEFAULT | 93.01 | 92.92 | -0.08 | 19.37 | 19.36 | -0.01 |
| XML_COMPACT | 92.26 | 91.15 | -1.11 | 19.22 | 18.99 | -0.23 |
| XML_PRETTY | 93.15 | 89.97 | -3.18 | 19.41 | 18.74 | -0.66 |
| YAML | 94.05 | 90.86 | -3.19 | 19.59 | 18.93 | -0.66 |

#### 2.10.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 88.06 | 71.72 | -16.34 | 11.01 | 8.97 | -2.04 |
| JSON_COMPACT | 74.07 | 73.94 | -0.13 | 9.26 | 9.24 | -0.02 |
| JSON_PRETTY | 82.72 | 71.11 | -11.60 | 10.34 | 8.89 | -1.45 |
| TOON_DEFAULT | 83.03 | 73.64 | -9.39 | 10.38 | 9.20 | -1.18 |
| XML_COMPACT | 79.84 | 73.74 | -6.10 | 9.98 | 9.22 | -0.76 |
| XML_PRETTY | 75.52 | 79.19 | +3.67 | 9.44 | 9.90 | +0.46 |
| YAML | 72.84 | 69.09 | -3.75 | 9.10 | 8.63 | -0.47 |

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

- **Report Generated**: 2026-04-14
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code 2.1.73
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_haiku_4_5/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_haiku_4_5/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_haiku_4_5/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)