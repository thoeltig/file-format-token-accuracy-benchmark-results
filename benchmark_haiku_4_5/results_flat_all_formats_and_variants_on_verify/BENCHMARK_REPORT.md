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

1. **CSV** achieves the highest total efficiency scores in both mandatory (78.16) and optional (81.33) variants. Its low read token cost (7695 mandatory, 7438 optional) combined with competitive accuracy creates the best cost-to-value ratio across all formats.
2. Accuracy by answer spread across formats is narrow but efficiency spread is wide. Complete answer accuracy ranges from 75.00% to 79.84% (mandatory) and 72.04% to 79.03% (optional) which is a spread of roughly 5 to 7 percentage points. However efficiency scores span from 53.26 to 78.16 (mandatory) and 55.80 to 81.33 (optional) which indicates token cost is the dominant differentiator and not accuracy.
3. Accuracy by character reveals that most errors are minor compared to what the accuracy by answer indicates. Complete answer accuracy averages around 77% but character-level accuracy reaches 94% to 98% across all formats. This 17 to 21 percentage point gap means the model frequently produces answers that are only partially wrong (e.g. off by a digit in aggregation, missing words) rather than entirely incorrect.
4. **TOON_DEFAULT**'s adaptive encoding creates a significant robustness penalty for sparse data. **TOON_DEFAULT** read tokens increase by +64.43% from mandatory (7018) to optional (11540) because sparse data triggers its key-value fallback encoding. Every other format reduces read tokens by 3% to 7% with optional data. This encoding switch pushes **TOON_DEFAULT** from the cheapest read format (mandatory) to the 4th most expensive (optional) which erases its dense-data advantage.
5. Pretty-printed formats consistently waste tokens without providing accuracy gains. **JSON_PRETTY** and **XML_PRETTY** consume 54% to 130% more read tokens than their compact counterparts. **JSON_PRETTY** achieves the highest mandatory accuracy (79.84%) but only 2.15 percentage points above **CSV** while consuming 85% more total tokens. **XML_PRETTY** shows no accuracy advantage over **XML_COMPACT** in mandatory data (both 77.15%) despite using 38% more total tokens.
6. Aggregation is the hardest question category and creates the largest accuracy variance between formats. Aggregation accuracy ranges from 39.68% (**JSON_COMPACT** mandatory) to 71.43% (**XML_PRETTY** mandatory) which is a 31.75 percentage point spread while field retrieval spans 90.91% to 100.00%. The model struggles with numerical computation regardless of format but the quality of number representation in each format noticeably affects aggregation performance.
7. **CSV** and **YAML** are the least robust formats when transitioning from dense to sparse data. **CSV** drops 5.65 percentage points in accuracy (77.69% to 72.04%) and **YAML** drops 6.72 points (79.30% to 72.58%) when optional fields are introduced. **JSON_COMPACT**, **TOON_DEFAULT**, **XML_COMPACT** and **XML_PRETTY** maintain or improve accuracy with optional data. Formats that rely on positional structure (**CSV**) or indentation-based grouping (**YAML**) appear more sensitive to missing values.

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
| XML_PRETTY ≈ 85s | TOON_DEFAULT ≈ 7024 | XML_COMPACT ≈ 209 | XML_PRETTY ≈ 9496 | XML_PRETTY ≈ 9799 | CSV ≈ 18813 | JSON_PRETTY ≈ 79.84% | TOON_DEFAULT ≈ 85 | XML_PRETTY ≈ 79 | CSV ≈ 78 | XML_COMPACT ≈ 98.31% | TOON_DEFAULT ≈ 97 | XML_PRETTY ≈ 91 | CSV ≈ 91 |
| CSV (+6.53%) | CSV (+9.56%) | JSON_COMPACT (+1.92%) | YAML (+13.44%) | YAML (+13.03%) | TOON_DEFAULT (+2.63%) | YAML (-0.54%) | CSV (-2.88%) | YAML (-10.38%) | TOON_DEFAULT (-1.90%) | JSON_PRETTY (-0.21%) | CSV (-0.59%) | YAML (-8.39%) | TOON_DEFAULT (-3.68%) |
| YAML (+8.48%) | JSON_COMPACT (+31.77%) | CSV (+44.73%) | CSV (+13.90%) | JSON_COMPACT (+13.40%) | JSON_COMPACT (+8.27%) | CSV (-2.15%) | JSON_COMPACT (-11.68%) | CSV (-12.14%) | JSON_COMPACT (-8.27%) | CSV (-0.65%) | JSON_COMPACT (-6.58%) | CSV (-8.55%) | JSON_COMPACT (-5.20%) |
| JSON_COMPACT (+10.88%) | XML_COMPACT (+66.18%) | XML_PRETTY (+45.53%) | JSON_COMPACT (+14.79%) | CSV (+13.45%) | YAML (+25.50%) | TOON_DEFAULT (-2.15%) | XML_COMPACT (-20.36%) | JSON_COMPACT (-14.36%) | YAML (-17.05%) | JSON_COMPACT (-0.78%) | XML_COMPACT (-15.18%) | JSON_COMPACT (-8.61%) | YAML (-15.93%) |
| TOON_DEFAULT (+18.09%) | YAML (+78.44%) | YAML (+45.53%) | TOON_DEFAULT (+26.12%) | TOON_DEFAULT (+25.36%) | XML_COMPACT (+33.56%) | XML_COMPACT (-2.69%) | YAML (-22.37%) | TOON_DEFAULT (-23.29%) | XML_COMPACT (-24.71%) | YAML (-0.90%) | YAML (-19.06%) | TOON_DEFAULT (-20.37%) | XML_COMPACT (-20.25%) |
| JSON_PRETTY (+27.20%) | JSON_PRETTY (+102.95%) | JSON_PRETTY (+47.60%) | JSON_PRETTY (+37.12%) | JSON_PRETTY (+36.01%) | XML_PRETTY (+37.87%) | XML_PRETTY (-2.69%) | JSON_PRETTY (-29.33%) | JSON_PRETTY (-31.45%) | XML_PRETTY (-27.83%) | TOON_DEFAULT (-3.47%) | JSON_PRETTY (-25.09%) | JSON_PRETTY (-26.68%) | XML_PRETTY (-25.81%) |
| XML_COMPACT (+30.70%) | XML_PRETTY (+129.76%) | TOON_DEFAULT (+48.04%) | XML_COMPACT (+39.48%) | XML_COMPACT (+37.29%) | JSON_PRETTY (+46.62%) | JSON_COMPACT (-4.84%) | XML_PRETTY (-39.51%) | XML_COMPACT (-34.91%) | JSON_PRETTY (-31.86%) | XML_PRETTY (-3.98%) | XML_PRETTY (-34.81%) | XML_COMPACT (-27.57%) | JSON_PRETTY (-28.47%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 69s | CSV ≈ 7438 | CSV ≈ 207 | YAML ≈ 8803 | CSV ≈ 9064 | CSV ≈ 16502 | XML_PRETTY ≈ 79.03% | CSV ≈ 80 | JSON_COMPACT ≈ 84 | CSV ≈ 81 | XML_COMPACT ≈ 98.16% | CSV ≈ 93 | JSON_COMPACT ≈ 98 | CSV ≈ 95 |
| CSV (+3.87%) | JSON_COMPACT (+17.33%) | JSON_COMPACT (+0.48%) | CSV (+0.61%) | JSON_COMPACT (+0.06%) | JSON_COMPACT (+7.85%) | TOON_DEFAULT (-0.13%) | JSON_COMPACT (-2.08%) | YAML (-3.55%) | JSON_COMPACT (-1.03%) | TOON_DEFAULT (-0.04%) | JSON_COMPACT (-1.72%) | YAML (-1.32%) | JSON_COMPACT (-0.83%) |
| JSON_COMPACT (+9.98%) | XML_COMPACT (+46.67%) | XML_PRETTY (+0.96%) | JSON_COMPACT (+0.67%) | YAML (+0.51%) | YAML (+26.36%) | JSON_PRETTY (-1.61%) | XML_COMPACT (-11.38%) | CSV (-3.56%) | YAML (-15.62%) | XML_PRETTY (-0.81%) | XML_COMPACT (-9.16%) | CSV (-3.14%) | YAML (-11.56%) |
| JSON_PRETTY (+24.18%) | TOON_DEFAULT (+55.15%) | JSON_PRETTY (+46.30%) | JSON_PRETTY (+17.29%) | JSON_PRETTY (+17.25%) | XML_COMPACT (+35.20%) | XML_COMPACT (-1.61%) | TOON_DEFAULT (-13.03%) | JSON_PRETTY (-13.33%) | XML_COMPACT (-17.04%) | JSON_PRETTY (-1.03%) | TOON_DEFAULT (-11.66%) | JSON_PRETTY (-11.71%) | XML_COMPACT (-14.06%) |
| XML_PRETTY (+26.54%) | YAML (+57.87%) | YAML (+48.39%) | XML_COMPACT (+26.02%) | XML_COMPACT (+25.79%) | TOON_DEFAULT (+41.56%) | JSON_COMPACT (-2.42%) | YAML (-19.23%) | XML_PRETTY (-19.69%) | TOON_DEFAULT (-19.70%) | JSON_COMPACT (-1.55%) | YAML (-14.59%) | XML_COMPACT (-17.00%) | TOON_DEFAULT (-17.42%) |
| XML_COMPACT (+33.13%) | JSON_PRETTY (+79.43%) | XML_COMPACT (+48.87%) | XML_PRETTY (+28.02%) | XML_PRETTY (+26.64%) | JSON_PRETTY (+45.28%) | YAML (-6.45%) | JSON_PRETTY (-22.52%) | XML_COMPACT (-20.27%) | JSON_PRETTY (-23.18%) | YAML (-3.02%) | JSON_PRETTY (-19.45%) | XML_PRETTY (-18.15%) | JSON_PRETTY (-20.07%) |
| TOON_DEFAULT (+36.69%) | XML_PRETTY (+102.69%) | TOON_DEFAULT (+50.32%) | TOON_DEFAULT (+30.74%) | TOON_DEFAULT (+30.41%) | XML_PRETTY (+60.92%) | CSV (-6.99%) | XML_PRETTY (-29.09%) | TOON_DEFAULT (-22.86%) | XML_PRETTY (-31.39%) | CSV (-6.21%) | XML_PRETTY (-26.08%) | TOON_DEFAULT (-20.27%) | XML_PRETTY (-28.10%) |


#### 2.1.2 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 100.00% | CSV ≈ 70.37% | TOON_DEFAULT ≈ 68.26% | XML_PRETTY ≈ 71.43% |
| XML_COMPACT (0.00%) | TOON_DEFAULT (-2.47%) | YAML (0.00%) | YAML (-6.35%) |
| CSV (-1.21%) | JSON_PRETTY (-6.17%) | JSON_PRETTY (-1.59%) | TOON_DEFAULT (-7.14%) |
| JSON_COMPACT (-1.21%) | JSON_COMPACT (-6.18%) | XML_PRETTY (-4.76%) | JSON_PRETTY (-11.11%) |
| YAML (-2.43%) | XML_PRETTY (-6.18%) | XML_COMPACT (-4.77%) | CSV (-20.63%) |
| TOON_DEFAULT (-8.79%) | XML_COMPACT (-7.41%) | JSON_COMPACT (-6.35%) | XML_COMPACT (-22.22%) |
| XML_PRETTY (-9.09%) | YAML (-8.64%) | CSV (-9.53%) | JSON_COMPACT (-31.74%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| XML_COMPACT ≈ 100.00% | JSON_PRETTY ≈ 80.25% | TOON_DEFAULT ≈ 67.46% | CSV ≈ 53.97% |
| TOON_DEFAULT (-1.52%) | XML_PRETTY (0.00%) | JSON_PRETTY (-2.38%) | TOON_DEFAULT (-3.97%) |
| XML_PRETTY (-3.03%) | JSON_COMPACT (-3.70%) | XML_COMPACT (-2.38%) | JSON_COMPACT (-4.76%) |
| JSON_COMPACT (-7.27%) | TOON_DEFAULT (-9.88%) | YAML (-3.97%) | XML_PRETTY (-6.35%) |
| JSON_PRETTY (-7.27%) | CSV (-11.11%) | JSON_COMPACT (-5.55%) | YAML (-7.93%) |
| YAML (-9.70%) | XML_COMPACT (-12.35%) | XML_PRETTY (-5.55%) | JSON_PRETTY (-7.94%) |
| CSV (-15.15%) | YAML (-16.05%) | CSV (-7.14%) | XML_COMPACT (-11.11%) |


#### 2.1.3 Category Accuracy By Character Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 100.00% | XML_COMPACT ≈ 98.22% | JSON_PRETTY ≈ 93.75% | YAML ≈ 85.39% |
| XML_COMPACT (0.00%) | JSON_COMPACT (-0.61%) | TOON_DEFAULT (-0.60%) | XML_PRETTY (-0.21%) |
| CSV (-0.82%) | JSON_PRETTY (-0.75%) | YAML (-1.49%) | TOON_DEFAULT (-0.72%) |
| JSON_COMPACT (-1.05%) | CSV (-0.81%) | CSV (-2.38%) | JSON_PRETTY (-3.08%) |
| YAML (-1.60%) | YAML (-0.92%) | XML_PRETTY (-2.68%) | CSV (-5.76%) |
| TOON_DEFAULT (-6.35%) | TOON_DEFAULT (-2.10%) | JSON_COMPACT (-3.87%) | XML_COMPACT (-9.67%) |
| XML_PRETTY (-7.18%) | XML_PRETTY (-2.39%) | XML_COMPACT (-5.95%) | JSON_COMPACT (-11.52%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| XML_COMPACT ≈ 100.00% | XML_PRETTY ≈ 98.52% | TOON_DEFAULT ≈ 92.04% | CSV ≈ 76.57% |
| TOON_DEFAULT (-0.46%) | JSON_PRETTY (-0.52%) | XML_COMPACT (-1.47%) | JSON_COMPACT (-0.81%) |
| JSON_PRETTY (-2.60%) | TOON_DEFAULT (-0.53%) | YAML (-1.48%) | XML_PRETTY (-2.63%) |
| JSON_COMPACT (-2.69%) | XML_COMPACT (-0.72%) | JSON_PRETTY (-2.06%) | TOON_DEFAULT (-2.83%) |
| XML_PRETTY (-2.76%) | YAML (-0.75%) | XML_PRETTY (-2.95%) | JSON_PRETTY (-3.84%) |
| YAML (-6.74%) | JSON_COMPACT (-1.50%) | CSV (-4.43%) | YAML (-4.44%) |
| CSV (-12.92%) | CSV (-1.56%) | JSON_COMPACT (-4.43%) | XML_COMPACT (-4.65%) |


#### 2.1.4 Conclusion

The rankings reveal that no single format dominates across all metrics. The optimal choice depends on the use case and what constraint matters most.

- For minimal total token cost with good accuracy **CSV** is a good choice because it ranks 1st in total efficiency score for both variants (78.16 mandatory, 81.33 optional). It has the lowest or near-lowest read tokens and achieves mid-range accuracy. Its main weakness is robustness because accuracy drops 5.65 percentage points with sparse data and it ranks last in filtering accuracy for mandatory data (58.73%). **CSV** works best when data is dense and complete.
- With the highest read token efficiency with dense data **TOON_DEFAULT** is a good alternative to **CSV**. **TOON_DEFAULT** achieves the lowest read tokens for mandatory data (7018, 9% lower than **CSV**) and the highest read efficiency score (85.09). It also leads in filtering accuracy for both variants (68.26% mandatory, 67.46% optional). **TOON_DEFAULT**'s adaptive encoding is a critical caveat because it inflates read tokens by 64% when data becomes sparse which collapses its read efficiency advantage. **TOON_DEFAULT** is the best choice when data completeness is guaranteed.
- For maximum accuracy regardless of cost choose **JSON_PRETTY** or **XML_PRETTY**. **JSON_PRETTY** achieves the highest mandatory accuracy (79.84%) with perfect field retrieval (100%). **XML_PRETTY** leads optional accuracy (79.03%) and mandatory aggregation (71.43%). Both formats are the most expensive in read tokens. They suit scenarios where accuracy is critical and token budget is flexible.
- For balanced accuracy and robustness across data variants **JSON_COMPACT** or **XML_COMPACT** are a good choice. **JSON_COMPACT** actually improves accuracy with optional data (+1.61 points) and ranks 2nd in read tokens and total efficiency for optional data (80.49). **XML_COMPACT** shows the most stable accuracy across variants (77.15% to 77.42%, only +0.27 points change) and achieves the highest accuracy by character overall (98.31% mandatory, 98.16% optional). These formats handle varying data completeness without degradation.
- **YAML** occupies a middle ground without a clear niche. It achieves the 2nd highest mandatory accuracy (79.30%) but suffers the largest accuracy drop with sparse data (6.72 points). Its token cost is moderate but offers no clear advantage over **JSON_COMPACT** for structured data exchange.
- Accuracy by character is remarkably consistent across formats. Accuracy by character clusters between 92% and 98% while accuracy by answer varies from 72% to 80% between formats. This consistency suggests that format choice affects whether the model gets an answer fully correct and not whether it fundamentally misunderstands the data. The practical impact depends on whether downstream consumers need exact matches or can tolerate near-correct values.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total | Accuracy By Character (%) | Useful Read Tokens (Acc By Char) | Wasted Read Tokens (Acc By Char) | Useful Output Tokens (Acc By Char) | Wasted Output Tokens (Acc By Char) | Eff Score Read (Acc By Char) | Eff Score Output (Acc By Char) | Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7695 | 11118 | 18813 | 1.31 | 87.22 | 77.69 | 5978 | 1717 | 8637 | 2480 | 82.64 | 69.53 | 78.16 | 97.66 | 7515 | 180 | 10858 | 260 | 95.95 | 82.84 | 91.47 |
| CSV | opt | 7438 | 9064 | 16502 | 1.29 | 71.43 | 72.04 | 5358 | 2080 | 6530 | 2534 | 79.81 | 81.28 | 81.33 | 91.95 | 6839 | 599 | 8334 | 730 | 93.08 | 94.56 | 94.60 |
| JSON_COMPACT | man | 9255 | 11113 | 20368 | 2.15 | 87.90 | 75.00 | 6941 | 2314 | 8335 | 2778 | 75.15 | 67.77 | 71.69 | 97.53 | 9026 | 229 | 10838 | 274 | 90.17 | 82.79 | 86.71 |
| JSON_COMPACT | opt | 8727 | 9070 | 17797 | 2.12 | 71.46 | 76.61 | 6686 | 2041 | 6948 | 2121 | 78.15 | 84.29 | 80.49 | 96.61 | 8431 | 296 | 8762 | 307 | 91.49 | 97.62 | 93.82 |
| JSON_PRETTY | man | 14254 | 13328 | 27582 | 1.70 | 105.00 | 79.84 | 11380 | 2874 | 10641 | 2687 | 60.14 | 54.25 | 53.26 | 98.10 | 13983 | 271 | 13075 | 253 | 72.31 | 66.42 | 65.43 |
| JSON_PRETTY | opt | 13346 | 10628 | 23974 | 1.68 | 83.26 | 77.42 | 10332 | 3014 | 8228 | 2400 | 61.84 | 73.05 | 62.48 | 97.13 | 12963 | 383 | 10323 | 305 | 74.97 | 86.19 | 75.62 |
| TOON_DEFAULT | man | 7024 | 12285 | 19308 | 1.45 | 96.58 | 77.69 | 5457 | 1567 | 9544 | 2741 | 85.09 | 60.70 | 76.67 | 94.84 | 6661 | 362 | 11651 | 634 | 96.52 | 72.14 | 88.10 |
| TOON_DEFAULT | opt | 11540 | 11821 | 23361 | 1.70 | 92.82 | 78.90 | 9105 | 2435 | 9327 | 2494 | 69.41 | 65.02 | 65.31 | 98.12 | 11323 | 217 | 11598 | 222 | 82.23 | 77.83 | 78.12 |
| XML_COMPACT | man | 11672 | 13453 | 25125 | 2.37 | 106.81 | 77.15 | 9005 | 2667 | 10379 | 3074 | 67.76 | 51.51 | 58.84 | 98.31 | 11475 | 197 | 13226 | 227 | 81.87 | 65.61 | 72.95 |
| XML_COMPACT | opt | 10909 | 11402 | 22311 | 2.35 | 89.46 | 77.42 | 8446 | 2463 | 8827 | 2574 | 70.73 | 67.20 | 67.47 | 98.16 | 10708 | 201 | 11192 | 210 | 84.56 | 81.03 | 81.30 |
| XML_PRETTY | man | 16137 | 9799 | 25936 | 1.94 | 76.58 | 77.15 | 12450 | 3687 | 7560 | 2239 | 51.47 | 79.13 | 56.41 | 94.33 | 15222 | 915 | 9244 | 556 | 62.92 | 90.59 | 67.86 |
| XML_PRETTY | opt | 15076 | 11479 | 26555 | 1.92 | 90.88 | 79.03 | 11915 | 3161 | 9072 | 2407 | 56.60 | 67.69 | 55.80 | 97.35 | 14676 | 400 | 11174 | 304 | 68.81 | 79.90 | 68.02 |
| YAML | man | 12533 | 11076 | 23609 | 1.67 | 86.87 | 79.30 | 9939 | 2594 | 8783 | 2293 | 66.06 | 70.91 | 64.83 | 97.41 | 12208 | 325 | 10789 | 287 | 78.13 | 82.99 | 76.90 |
| YAML | opt | 11742 | 9110 | 20852 | 1.65 | 70.99 | 72.58 | 8522 | 3220 | 6612 | 2498 | 64.46 | 81.29 | 68.63 | 95.14 | 11171 | 571 | 8668 | 443 | 79.50 | 96.33 | 83.67 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7695 | 7438 | -257 | -3.34 | 302 | 207 | -95 | -31.46 | 10816 | 8857 | -1959 | -18.11 | 11118 | 9064 | -2054 | -18.47 | 18813 | 16502 | -2311 | -12.28 |
| JSON_COMPACT | 9255 | 8727 | -528 | -5.71 | 213 | 209 | -4 | -1.88 | 10900 | 8861 | -2039 | -18.71 | 11113 | 9070 | -2043 | -18.38 | 20368 | 17797 | -2571 | -12.62 |
| JSON_PRETTY | 14254 | 13346 | -908 | -6.37 | 308 | 303 | -5 | -1.62 | 13020 | 10324 | -2696 | -20.71 | 13328 | 10627 | -2701 | -20.27 | 27582 | 23973 | -3609 | -13.08 |
| TOON_DEFAULT | 7024 | 11541 | +4517 | +64.31 | 309 | 312 | +3 | +0.97 | 11976 | 11509 | -467 | -3.90 | 12285 | 11821 | -464 | -3.78 | 19308 | 23360 | +4052 | +20.99 |
| XML_COMPACT | 11672 | 10909 | -763 | -6.54 | 209 | 309 | +100 | +47.85 | 13245 | 11093 | -2152 | -16.25 | 13453 | 11401 | -2052 | -15.25 | 25125 | 22310 | -2815 | -11.20 |
| XML_PRETTY | 16137 | 15076 | -1061 | -6.57 | 304 | 210 | -94 | -30.92 | 9496 | 11270 | +1774 | +18.68 | 9799 | 11478 | +1679 | +17.13 | 25936 | 26554 | +618 | +2.38 |
| YAML | 12533 | 11742 | -791 | -6.31 | 304 | 308 | +4 | +1.32 | 10772 | 8802 | -1970 | -18.29 | 11076 | 9110 | -1966 | -17.75 | 23609 | 20852 | -2757 | -11.68 |

### 2.4 Drift over multiple runs

*Note: Drift = The distance from average to the lowest or highest value (absolute distance in tokens and perentage points). Spread = Distance between lowest to highest value (larger = less predictable results across runs).*

#### 2.4.1 Drift per Format and Variant
| Format | Variant | Runs | Output Tokens Total | Drift | Spread | Accuracy (%) | Drift (pp) | Spread (pp) | Accuracy By Character (%) | Drift (pp) | Spread (pp) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 3 | 11118 | -2081/+3062 | 5143 | 77.69 | -7.53/+4.57 | 12.10 | 97.66 | -1.21/+1.11 | 2.32 |
| CSV | opt | 3 | 9064 | -3417/+1851 | 5268 | 72.04 | -8.33/+6.19 | 14.52 | 91.95 | -2.60/+3.77 | 6.37 |
| JSON_COMPACT | man | 3 | 11113 | -2221/+1508 | 3729 | 75.00 | -2.42/+1.61 | 4.04 | 97.53 | -0.03/+0.05 | 0.08 |
| JSON_COMPACT | opt | 3 | 9070 | -4004/+4016 | 8020 | 76.61 | -7.26/+5.65 | 12.92 | 96.61 | -3.09/+1.56 | 4.65 |
| JSON_PRETTY | man | 3 | 13328 | -1418/+2564 | 3983 | 79.84 | -0.81/+0.81 | 1.61 | 98.10 | -0.47/+0.36 | 0.83 |
| JSON_PRETTY | opt | 3 | 10628 | -5356/+6702 | 12058 | 77.42 | -3.23/+3.23 | 6.46 | 97.13 | -1.92/+1.15 | 3.07 |
| TOON_DEFAULT | man | 5 | 12285 | -2065/+2155 | 4220 | 77.69 | -14.79/+8.60 | 23.39 | 94.84 | -6.11/+4.05 | 10.16 |
| TOON_DEFAULT | opt | 6 | 11821 | -2702/+2504 | 5206 | 78.90 | -2.29/+3.36 | 5.65 | 98.12 | -0.95/+0.43 | 1.38 |
| XML_COMPACT | man | 3 | 13453 | -1699/+2272 | 3971 | 77.15 | -2.15/+2.69 | 4.85 | 98.31 | -0.33/+0.50 | 0.84 |
| XML_COMPACT | opt | 3 | 11402 | -1593/+2749 | 4342 | 77.42 | -2.42/+4.03 | 6.46 | 98.16 | -0.49/+0.96 | 1.45 |
| XML_PRETTY | man | 3 | 9799 | -1279/+2133 | 3412 | 77.15 | -2.15/+2.69 | 4.85 | 94.33 | -1.39/+1.97 | 3.36 |
| XML_PRETTY | opt | 3 | 11479 | -2435/+4236 | 6670 | 79.03 | -4.03/+4.03 | 8.06 | 97.35 | -1.44/+1.89 | 3.33 |
| YAML | man | 3 | 11076 | -1334/+1907 | 3241 | 79.30 | -3.49/+5.38 | 8.87 | 97.41 | -0.72/+0.47 | 1.19 |
| YAML | opt | 3 | 9110 | -85/+110 | 195 | 72.58 | -4.03/+4.03 | 8.06 | 95.14 | -2.12/+2.78 | 4.90 |

#### 2.4.2 Spread: Mandatory vs Optional
| Format | Output Tokens Total Spread Man | Output Tokens Total Spread Opt | Diff | Acc Spread Man (pp) | Acc Spread Opt (pp) | Diff (pp) | Acc By Char Spread Man (pp) | Acc By Char Spread Opt (pp) | Diff (pp) |
|---|---|---|---|---|---|---|---|---|---|
| CSV | 5143 | 5268 | +125 | 12.10 | 14.52 | +2.42 | 2.32 | 6.37 | +4.05 |
| JSON_COMPACT | 3729 | 8020 | +4291 | 4.04 | 12.92 | +8.88 | 0.08 | 4.65 | +4.57 |
| JSON_PRETTY | 3983 | 12058 | +8076 | 1.61 | 6.46 | +4.84 | 0.83 | 3.07 | +2.24 |
| TOON_DEFAULT | 4220 | 5206 | +986 | 23.39 | 5.65 | -17.74 | 10.16 | 1.38 | -8.77 |
| XML_COMPACT | 3971 | 4342 | +370 | 4.85 | 6.46 | +1.61 | 0.84 | 1.45 | +0.62 |
| XML_PRETTY | 3412 | 6670 | +3258 | 4.85 | 8.06 | +3.22 | 3.36 | 3.33 | -0.03 |
| YAML | 3241 | 195 | -3046 | 8.87 | 8.06 | -0.81 | 1.19 | 4.90 | +3.71 |

### 2.5 Performance
#### 2.5.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Before Write (ms) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) | Output (ms) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 12 | 641.25 | 0.39 | 55603 | 34909 | 0.31 | 281.52 | 34921 | 641.56 | 225.30 | 90512 |
| CSV | opt | 12 | 619.83 | 0.39 | 40469 | 31556 | 0.28 | 254.48 | 31568 | 620.11 | 203.66 | 72025 |
| JSON_COMPACT | man | 24 | 385.63 | 0.77 | 54629 | 39580 | 0.28 | 319.19 | 39604 | 385.90 | 255.51 | 94209 |
| JSON_COMPACT | opt | 13 | 671.31 | 0.42 | 38635 | 37630 | 0.24 | 303.47 | 37643 | 671.54 | 242.86 | 76265 |
| JSON_PRETTY | man | 30 | 475.13 | 0.97 | 68671 | 39401 | 0.33 | 317.75 | 39431 | 475.46 | 254.39 | 108073 |
| JSON_PRETTY | opt | 36 | 370.72 | 1.16 | 45880 | 40232 | 0.26 | 324.45 | 40268 | 370.98 | 259.79 | 86112 |
| TOON_DEFAULT | man | 22 | 319.25 | 0.71 | 62236 | 38099 | 0.31 | 307.25 | 38121 | 319.56 | 245.94 | 100335 |
| TOON_DEFAULT | opt | 21 | 562.93 | 0.68 | 57725 | 37064 | 0.31 | 298.90 | 37085 | 563.24 | 239.26 | 94789 |
| XML_COMPACT | man | 19 | 614.32 | 0.61 | 73292 | 37753 | 0.35 | 304.46 | 37772 | 614.67 | 243.69 | 111045 |
| XML_COMPACT | opt | 23 | 474.30 | 0.74 | 55775 | 36545 | 0.30 | 294.72 | 36568 | 474.61 | 235.92 | 92319 |
| XML_PRETTY | man | 18 | 896.50 | 0.58 | 45341 | 39623 | 0.24 | 319.54 | 39641 | 896.74 | 255.75 | 84964 |
| XML_PRETTY | opt | 17 | 886.82 | 0.55 | 57146 | 30600 | 0.37 | 246.77 | 30617 | 887.19 | 197.53 | 87747 |
| YAML | man | 21 | 596.81 | 0.68 | 53707 | 38458 | 0.28 | 310.15 | 38479 | 597.09 | 248.25 | 92165 |
| YAML | opt | 31 | 378.77 | 1.00 | 39189 | 30155 | 0.29 | 243.19 | 30186 | 379.07 | 194.75 | 69344 |

#### 2.5.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Before Write Man (s) | Output Before Write Opt (s) | Diff (s) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 12 | 12 | 0 | 0.00 | 55.60 | 40.47 | -15.13 | -27.22 | 34.91 | 31.56 | -3.35 | -9.60 | 34.92 | 31.57 | -3.35 | -9.60 | 90.51 | 72.02 | -18.49 | -20.42 |
| JSON_COMPACT | 24 | 13 | -11 | -45.83 | 54.63 | 38.64 | -15.99 | -29.28 | 39.58 | 37.63 | -1.95 | -4.93 | 39.60 | 37.64 | -1.96 | -4.95 | 94.21 | 76.26 | -17.94 | -19.05 |
| JSON_PRETTY | 30 | 36 | +6 | +20.00 | 68.67 | 45.88 | -22.79 | -33.19 | 39.40 | 40.23 | +0.83 | +2.11 | 39.43 | 40.27 | +0.84 | +2.12 | 108.07 | 86.11 | -21.96 | -20.32 |
| TOON_DEFAULT | 22 | 21 | -1 | -4.55 | 62.24 | 57.72 | -4.51 | -7.25 | 38.10 | 37.06 | -1.03 | -2.72 | 38.12 | 37.09 | -1.04 | -2.72 | 100.34 | 94.79 | -5.55 | -5.53 |
| XML_COMPACT | 19 | 23 | +4 | +21.05 | 73.29 | 55.77 | -17.52 | -23.90 | 37.75 | 36.54 | -1.21 | -3.20 | 37.77 | 36.57 | -1.20 | -3.19 | 111.05 | 92.32 | -18.73 | -16.86 |
| XML_PRETTY | 18 | 17 | -1 | -5.56 | 45.34 | 57.15 | +11.81 | +26.04 | 39.62 | 30.60 | -9.02 | -22.77 | 39.64 | 30.62 | -9.02 | -22.76 | 84.96 | 87.75 | +2.78 | +3.28 |
| YAML | 21 | 31 | +10 | +47.62 | 53.71 | 39.19 | -14.52 | -27.03 | 38.46 | 30.15 | -8.30 | -21.59 | 38.48 | 30.19 | -8.29 | -21.55 | 92.17 | 69.34 | -22.82 | -24.76 |

### 2.6 Structural Efficiency
#### 2.6.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token | Info / Read Token (Acc By Char) | Info / Output Token (Acc By Char) | Info / Total Token (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 1.31 | 11.28 | 248.23 | 1.01 | 0.70 | 0.41 | 1.27 | 0.88 | 0.52 |
| CSV | opt | 1.29 | 11.79 | 239.94 | 0.97 | 0.80 | 0.44 | 1.24 | 1.01 | 0.56 |
| JSON_COMPACT | man | 2.15 | 13.57 | 298.55 | 0.81 | 0.68 | 0.37 | 1.05 | 0.88 | 0.48 |
| JSON_COMPACT | opt | 2.12 | 13.83 | 281.52 | 0.88 | 0.85 | 0.43 | 1.11 | 1.07 | 0.54 |
| JSON_PRETTY | man | 1.70 | 20.90 | 459.81 | 0.56 | 0.60 | 0.29 | 0.69 | 0.74 | 0.36 |
| JSON_PRETTY | opt | 1.68 | 21.15 | 430.52 | 0.58 | 0.73 | 0.32 | 0.73 | 0.91 | 0.41 |
| TOON_DEFAULT | man | 1.45 | 10.30 | 226.57 | 1.11 | 0.63 | 0.40 | 1.35 | 0.77 | 0.49 |
| TOON_DEFAULT | opt | 1.70 | 18.29 | 372.26 | 0.68 | 0.67 | 0.34 | 0.85 | 0.83 | 0.42 |
| XML_COMPACT | man | 2.37 | 17.11 | 376.52 | 0.66 | 0.57 | 0.31 | 0.84 | 0.73 | 0.39 |
| XML_COMPACT | opt | 2.35 | 17.29 | 351.90 | 0.71 | 0.68 | 0.35 | 0.90 | 0.86 | 0.44 |
| XML_PRETTY | man | 1.94 | 23.66 | 520.55 | 0.48 | 0.79 | 0.30 | 0.59 | 0.96 | 0.36 |
| XML_PRETTY | opt | 1.92 | 23.89 | 486.32 | 0.52 | 0.69 | 0.30 | 0.65 | 0.85 | 0.37 |
| YAML | man | 1.67 | 18.38 | 404.29 | 0.63 | 0.72 | 0.34 | 0.78 | 0.88 | 0.41 |
| YAML | opt | 1.65 | 18.61 | 378.77 | 0.62 | 0.80 | 0.35 | 0.81 | 1.04 | 0.46 |

#### 2.6.2 Characters And Values: Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.31 | 1.29 | -0.02 | -1.83 | 11.28 | 11.79 | +0.51 | +4.48 | 248.23 | 239.94 | -8.29 | -3.34 |
| JSON_COMPACT | 2.15 | 2.12 | -0.03 | -1.58 | 13.57 | 13.83 | +0.26 | +1.92 | 298.55 | 281.52 | -17.03 | -5.70 |
| JSON_PRETTY | 1.70 | 1.68 | -0.01 | -0.82 | 20.90 | 21.15 | +0.25 | +1.20 | 459.81 | 430.52 | -29.29 | -6.37 |
| TOON_DEFAULT | 1.45 | 1.70 | +0.25 | +17.55 | 10.30 | 18.29 | +7.99 | +77.59 | 226.57 | 372.26 | +145.69 | +64.31 |
| XML_COMPACT | 2.37 | 2.35 | -0.02 | -1.05 | 17.11 | 17.29 | +0.17 | +1.02 | 376.52 | 351.90 | -24.61 | -6.54 |
| XML_PRETTY | 1.94 | 1.92 | -0.02 | -0.98 | 23.66 | 23.89 | +0.23 | +0.98 | 520.55 | 486.32 | -34.23 | -6.57 |
| YAML | 1.67 | 1.65 | -0.01 | -0.78 | 18.38 | 18.61 | +0.23 | +1.26 | 404.29 | 378.77 | -25.52 | -6.31 |

#### 2.6.3 Information: Mandatory vs Optional
| Format | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.01 | 0.97 | -0.04 | -4.06 | 0.70 | 0.80 | +0.10 | +13.73 | 0.41 | 0.44 | +0.02 | +5.81 |
| JSON_COMPACT | 0.81 | 0.88 | +0.07 | +8.40 | 0.68 | 0.85 | +0.17 | +25.19 | 0.37 | 0.43 | +0.06 | +16.85 |
| JSON_PRETTY | 0.56 | 0.58 | +0.02 | +3.57 | 0.60 | 0.73 | +0.13 | +21.54 | 0.29 | 0.32 | +0.03 | +11.76 |
| TOON_DEFAULT | 1.11 | 0.68 | -0.42 | -38.16 | 0.63 | 0.67 | +0.04 | +5.54 | 0.40 | 0.34 | -0.06 | -15.92 |
| XML_COMPACT | 0.66 | 0.71 | +0.05 | +7.41 | 0.57 | 0.68 | +0.11 | +18.50 | 0.31 | 0.35 | +0.04 | +13.03 |
| XML_PRETTY | 0.48 | 0.52 | +0.05 | +9.62 | 0.79 | 0.69 | -0.10 | -12.58 | 0.30 | 0.30 | 0.00 | 0.00 |
| YAML | 0.63 | 0.62 | -0.02 | -2.37 | 0.72 | 0.80 | +0.08 | +11.31 | 0.34 | 0.35 | +0.01 | +3.57 |

#### 2.6.4 Information (Accuracy By Character): Mandatory vs Optional
| Format | Info / Read Token (Acc By Char) Man | Info / Read Token (Acc By Char) Opt | Diff | Diff (%) | Info / Output Token (Acc By Char) Man | Info / Output Token (Acc By Char)  Opt | Diff | Diff (%) | Info / Total Token (Acc By Char) Man | Info / Total Token (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.27 | 1.24 | -0.03 | -2.60 | 0.88 | 1.01 | +0.14 | +15.49 | 0.52 | 0.56 | +0.04 | +7.32 |
| JSON_COMPACT | 1.05 | 1.11 | +0.05 | +5.03 | 0.88 | 1.07 | +0.19 | +21.30 | 0.48 | 0.54 | +0.06 | +13.36 |
| JSON_PRETTY | 0.69 | 0.73 | +0.04 | +5.81 | 0.74 | 0.91 | +0.18 | +24.18 | 0.36 | 0.41 | +0.05 | +13.76 |
| TOON_DEFAULT | 1.35 | 0.85 | -0.50 | -37.04 | 0.77 | 0.83 | +0.06 | +7.51 | 0.49 | 0.42 | -0.07 | -14.46 |
| XML_COMPACT | 0.84 | 0.90 | +0.06 | +6.89 | 0.73 | 0.86 | +0.13 | +17.78 | 0.39 | 0.44 | +0.05 | +12.53 |
| XML_PRETTY | 0.59 | 0.65 | +0.06 | +10.43 | 0.96 | 0.85 | -0.11 | -11.94 | 0.36 | 0.37 | 0.00 | 0.00 |
| YAML | 0.78 | 0.81 | +0.03 | +4.25 | 0.88 | 1.04 | +0.17 | +18.77 | 0.41 | 0.46 | +0.04 | +10.41 |

### 2.7 Token Utilization Efficiency
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7695 | 5978 | 1717 | 11118 | 8637 | 2480 | 18813 | 14616 | 4197 | 77.69 | 82.64 | 69.53 | 78.16 | 76.16 | 81.62 | 68.51 | 77.14 |
| CSV | opt | 7438 | 5358 | 2080 | 9064 | 6530 | 2534 | 16502 | 11888 | 4614 | 72.04 | 79.81 | 81.28 | 81.33 | 71.29 | 79.31 | 80.78 | 80.83 |
| JSON_COMPACT | man | 9255 | 6941 | 2314 | 11113 | 8335 | 2778 | 20368 | 15276 | 5092 | 75.00 | 75.15 | 67.77 | 71.69 | 73.63 | 74.24 | 66.86 | 70.78 |
| JSON_COMPACT | opt | 8727 | 6686 | 2041 | 9070 | 6948 | 2121 | 17797 | 13634 | 4163 | 76.61 | 78.15 | 84.29 | 80.49 | 76.15 | 77.85 | 83.98 | 80.18 |
| JSON_PRETTY | man | 14254 | 11380 | 2874 | 13328 | 10641 | 2687 | 27582 | 22022 | 5561 | 79.84 | 60.14 | 54.25 | 53.26 | 77.65 | 58.68 | 52.79 | 51.80 |
| JSON_PRETTY | opt | 13346 | 10332 | 3014 | 10628 | 8228 | 2400 | 23974 | 18560 | 5413 | 77.42 | 61.84 | 73.05 | 62.48 | 77.50 | 61.89 | 73.10 | 62.53 |
| TOON_DEFAULT | man | 7024 | 5457 | 1567 | 12285 | 9544 | 2741 | 19308 | 15001 | 4308 | 77.69 | 85.09 | 60.70 | 76.67 | 76.26 | 84.14 | 59.75 | 75.72 |
| TOON_DEFAULT | opt | 11540 | 9105 | 2435 | 11821 | 9327 | 2494 | 23361 | 18432 | 4929 | 78.90 | 69.41 | 65.02 | 65.31 | 77.76 | 68.65 | 64.26 | 64.55 |
| XML_COMPACT | man | 11672 | 9005 | 2667 | 13453 | 10379 | 3074 | 25125 | 19384 | 5741 | 77.15 | 67.76 | 51.51 | 58.84 | 75.24 | 66.49 | 50.24 | 57.57 |
| XML_COMPACT | opt | 10909 | 8446 | 2463 | 11402 | 8827 | 2574 | 22311 | 17273 | 5038 | 77.42 | 70.73 | 67.20 | 67.47 | 76.22 | 69.93 | 66.40 | 66.67 |
| XML_PRETTY | man | 16137 | 12450 | 3687 | 9799 | 7560 | 2239 | 25936 | 20010 | 5926 | 77.15 | 51.47 | 79.13 | 56.41 | 74.97 | 50.02 | 77.68 | 54.95 |
| XML_PRETTY | opt | 15076 | 11915 | 3161 | 11479 | 9072 | 2407 | 26555 | 20986 | 5569 | 79.03 | 56.60 | 67.69 | 55.80 | 78.62 | 56.32 | 67.42 | 55.53 |
| YAML | man | 12533 | 9939 | 2594 | 11076 | 8783 | 2293 | 23609 | 18722 | 4887 | 79.30 | 66.06 | 70.91 | 64.83 | 76.94 | 64.48 | 69.34 | 63.25 |
| YAML | opt | 11742 | 8522 | 3220 | 9110 | 6612 | 2498 | 20852 | 15135 | 5718 | 72.58 | 64.46 | 81.29 | 68.63 | 71.57 | 63.79 | 80.62 | 67.95 |

#### 2.7.2 Read Tokens: Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7695 | 7438 | -257 | -3.34 | 5978 | 5358 | -620 | -10.37 | 1717 | 2080 | +363 | +21.14 | 77.69 | 72.04 | -5.65 | 82.64 | 79.81 | -2.83 | -3.42 | 76.16 | 71.29 | -4.87 | 81.62 | 79.31 | -2.31 | -2.83 |
| JSON_COMPACT | 9255 | 8727 | -528 | -5.71 | 6941 | 6686 | -255 | -3.68 | 2314 | 2041 | -273 | -11.78 | 75.00 | 76.61 | +1.61 | 75.15 | 78.15 | +3.00 | +3.99 | 73.63 | 76.15 | +2.52 | 74.24 | 77.85 | +3.61 | +4.86 |
| JSON_PRETTY | 14254 | 13346 | -908 | -6.37 | 11380 | 10332 | -1048 | -9.21 | 2874 | 3014 | +140 | +4.87 | 79.84 | 77.42 | -2.42 | 60.14 | 61.84 | +1.70 | +2.83 | 77.65 | 77.50 | -0.15 | 58.68 | 61.89 | +3.21 | +5.48 |
| TOON_DEFAULT | 7024 | 11541 | +4517 | +64.30 | 5457 | 9106 | +3649 | +66.86 | 1567 | 2435 | +868 | +55.39 | 77.69 | 78.90 | +1.21 | 85.09 | 69.41 | -15.68 | -18.42 | 76.26 | 77.76 | +1.50 | 84.14 | 68.65 | -15.48 | -18.40 |
| XML_COMPACT | 11672 | 10909 | -763 | -6.54 | 9005 | 8446 | -559 | -6.21 | 2667 | 2463 | -204 | -7.64 | 77.15 | 77.42 | +0.27 | 67.76 | 70.73 | +2.97 | +4.38 | 75.24 | 76.22 | +0.98 | 66.49 | 69.93 | +3.44 | +5.17 |
| XML_PRETTY | 16137 | 15076 | -1061 | -6.57 | 12450 | 11915 | -535 | -4.30 | 3687 | 3161 | -526 | -14.26 | 77.15 | 79.03 | +1.88 | 51.47 | 56.60 | +5.13 | +9.96 | 74.97 | 78.62 | +3.65 | 50.02 | 56.32 | +6.31 | +12.61 |
| YAML | 12533 | 11742 | -791 | -6.31 | 9939 | 8523 | -1416 | -14.25 | 2594 | 3219 | +625 | +24.11 | 79.30 | 72.58 | -6.72 | 66.06 | 64.46 | -1.59 | -2.41 | 76.94 | 71.57 | -5.37 | 64.48 | 63.79 | -0.69 | -1.07 |

#### 2.7.3 Output Tokens: Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 11118 | 9064 | -2054 | -18.47 | 8637 | 6529 | -2108 | -24.40 | 2480 | 2534 | +54 | +2.18 | 77.69 | 72.04 | -5.65 | 69.53 | 81.28 | +11.76 | +16.91 | 76.16 | 71.29 | -4.87 | 68.51 | 80.78 | +12.28 | +17.92 |
| JSON_COMPACT | 11113 | 9070 | -2043 | -18.38 | 8335 | 6949 | -1386 | -16.63 | 2778 | 2121 | -657 | -23.64 | 75.00 | 76.61 | +1.61 | 67.77 | 84.29 | +16.52 | +24.37 | 73.63 | 76.15 | +2.52 | 66.86 | 83.98 | +17.13 | +25.61 |
| JSON_PRETTY | 13328 | 10627 | -2701 | -20.26 | 10641 | 8228 | -2413 | -22.68 | 2687 | 2400 | -287 | -10.69 | 79.84 | 77.42 | -2.42 | 54.25 | 73.05 | +18.80 | +34.66 | 77.65 | 77.50 | -0.15 | 52.79 | 73.10 | +20.32 | +38.49 |
| TOON_DEFAULT | 12285 | 11821 | -464 | -3.78 | 9544 | 9326 | -218 | -2.28 | 2741 | 2494 | -247 | -9.00 | 77.69 | 78.90 | +1.21 | 60.70 | 65.02 | +4.31 | +7.11 | 76.26 | 77.76 | +1.50 | 59.75 | 64.26 | +4.51 | +7.54 |
| XML_COMPACT | 13453 | 11401 | -2052 | -15.25 | 10379 | 8827 | -1552 | -14.95 | 3074 | 2574 | -500 | -16.25 | 77.15 | 77.42 | +0.27 | 51.51 | 67.20 | +15.69 | +30.46 | 75.24 | 76.22 | +0.98 | 50.24 | 66.40 | +16.16 | +32.17 |
| XML_PRETTY | 9799 | 11478 | +1679 | +17.14 | 7560 | 9071 | +1511 | +19.99 | 2239 | 2407 | +168 | +7.50 | 77.15 | 79.03 | +1.88 | 79.13 | 67.69 | -11.44 | -14.46 | 74.97 | 78.62 | +3.65 | 77.68 | 67.42 | -10.26 | -13.21 |
| YAML | 11076 | 9110 | -1966 | -17.75 | 8783 | 6612 | -2171 | -24.72 | 2293 | 2498 | +205 | +8.95 | 79.30 | 72.58 | -6.72 | 70.91 | 81.29 | +10.38 | +14.64 | 76.94 | 71.57 | -5.37 | 69.34 | 80.62 | +11.28 | +16.27 |

#### 2.7.4 Total Tokens: Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 18813 | 16502 | -2311 | -12.28 | 14616 | 11888 | -2728 | -18.66 | 4197 | 4614 | +417 | +9.93 | 77.69 | 72.04 | -5.65 | 78.16 | 81.33 | +3.17 | +4.06 | 76.16 | 71.29 | -4.87 | 77.14 | 80.83 | +3.69 | +4.79 |
| JSON_COMPACT | 20368 | 17797 | -2571 | -12.62 | 15276 | 13634 | -1642 | -10.75 | 5092 | 4163 | -929 | -18.25 | 75.00 | 76.61 | +1.61 | 71.69 | 80.49 | +8.79 | +12.27 | 73.63 | 76.15 | +2.52 | 70.78 | 80.18 | +9.40 | +13.28 |
| JSON_PRETTY | 27582 | 23973 | -3609 | -13.08 | 22022 | 18561 | -3461 | -15.72 | 5561 | 5414 | -147 | -2.65 | 79.84 | 77.42 | -2.42 | 53.26 | 62.48 | +9.22 | +17.32 | 77.65 | 77.50 | -0.15 | 51.80 | 62.53 | +10.74 | +20.73 |
| TOON_DEFAULT | 19308 | 23360 | +4052 | +20.99 | 15001 | 18432 | +3431 | +22.87 | 4308 | 4929 | +621 | +14.43 | 77.69 | 78.90 | +1.21 | 76.67 | 65.31 | -11.36 | -14.82 | 76.26 | 77.76 | +1.50 | 75.72 | 64.55 | -11.17 | -14.75 |
| XML_COMPACT | 25125 | 22310 | -2815 | -11.20 | 19384 | 17273 | -2111 | -10.89 | 5741 | 5038 | -703 | -12.25 | 77.15 | 77.42 | +0.27 | 58.84 | 67.47 | +8.63 | +14.67 | 75.24 | 76.22 | +0.98 | 57.57 | 66.67 | +9.11 | +15.82 |
| XML_PRETTY | 25936 | 26554 | +618 | +2.38 | 20010 | 20986 | +976 | +4.88 | 5926 | 5568 | -358 | -6.04 | 77.15 | 79.03 | +1.88 | 56.41 | 55.80 | -0.60 | -1.07 | 74.97 | 78.62 | +3.65 | 54.95 | 55.53 | +0.58 | +1.05 |
| YAML | 23609 | 20852 | -2757 | -11.68 | 18722 | 15135 | -3587 | -19.16 | 4887 | 5718 | +831 | +17.00 | 79.30 | 72.58 | -6.72 | 64.83 | 68.63 | +3.80 | +5.86 | 76.94 | 71.57 | -5.37 | 63.25 | 67.95 | +4.70 | +7.43 |

### 2.8 Token Utilization Efficiency (Accuracy by Character)
#### 2.8.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy by Character (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy by Character (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7695 | 7515 | 180 | 11118 | 10858 | 260 | 18813 | 18372 | 440 | 97.66 | 95.95 | 82.84 | 91.47 | 94.59 | 93.91 | 80.79 | 89.42 |
| CSV | opt | 7438 | 6839 | 599 | 9064 | 8334 | 730 | 16502 | 15174 | 1328 | 91.95 | 93.08 | 94.56 | 94.60 | 88.76 | 90.96 | 92.43 | 92.48 |
| JSON_COMPACT | man | 9255 | 9026 | 229 | 11113 | 10838 | 274 | 20368 | 19865 | 503 | 97.53 | 90.17 | 82.79 | 86.71 | 93.53 | 87.51 | 80.12 | 84.05 |
| JSON_COMPACT | opt | 8727 | 8431 | 296 | 9070 | 8762 | 307 | 17797 | 17193 | 603 | 96.61 | 91.49 | 97.62 | 93.82 | 92.51 | 88.75 | 94.89 | 91.09 |
| JSON_PRETTY | man | 14254 | 13983 | 271 | 13328 | 13075 | 253 | 27582 | 27058 | 524 | 98.10 | 72.31 | 66.42 | 65.43 | 95.75 | 70.74 | 64.85 | 63.86 |
| JSON_PRETTY | opt | 13346 | 12963 | 383 | 10628 | 10323 | 305 | 23974 | 23286 | 688 | 97.13 | 74.97 | 86.19 | 75.62 | 92.94 | 72.18 | 83.40 | 72.83 |
| TOON_DEFAULT | man | 7024 | 6661 | 362 | 12285 | 11651 | 634 | 19308 | 18312 | 996 | 94.84 | 96.52 | 72.14 | 88.10 | 93.15 | 95.40 | 71.01 | 86.98 |
| TOON_DEFAULT | opt | 11540 | 11323 | 217 | 11821 | 11598 | 222 | 23361 | 22921 | 439 | 98.12 | 82.23 | 77.83 | 78.12 | 94.30 | 79.68 | 75.28 | 75.57 |
| XML_COMPACT | man | 11672 | 11475 | 197 | 13453 | 13226 | 227 | 25125 | 24701 | 425 | 98.31 | 81.87 | 65.61 | 72.95 | 93.91 | 78.94 | 62.68 | 70.01 |
| XML_COMPACT | opt | 10909 | 10708 | 201 | 11402 | 11192 | 210 | 22311 | 21900 | 411 | 98.16 | 84.56 | 81.03 | 81.30 | 93.88 | 81.70 | 78.17 | 78.45 |
| XML_PRETTY | man | 16137 | 15222 | 915 | 9799 | 9244 | 556 | 25936 | 24466 | 1471 | 94.33 | 62.92 | 90.59 | 67.86 | 92.37 | 61.62 | 89.28 | 66.55 |
| XML_PRETTY | opt | 15076 | 14676 | 400 | 11479 | 11174 | 304 | 26555 | 25851 | 704 | 97.35 | 68.81 | 79.90 | 68.02 | 93.00 | 65.91 | 77.00 | 65.11 |
| YAML | man | 12533 | 12208 | 325 | 11076 | 10789 | 287 | 23609 | 22998 | 611 | 97.41 | 78.13 | 82.99 | 76.90 | 95.18 | 76.64 | 81.50 | 75.41 |
| YAML | opt | 11742 | 11171 | 571 | 9110 | 8668 | 443 | 20852 | 19839 | 1013 | 95.14 | 79.50 | 96.33 | 83.67 | 91.37 | 76.99 | 93.82 | 81.15 |

#### 2.8.2 Read Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7695 | 7438 | -257 | -3.34 | 7515 | 6839 | -676 | -8.99 | 180 | 599 | +419 | +232.61 | 97.66 | 91.95 | -5.71 | 95.95 | 93.08 | -2.87 | -2.99 | 94.59 | 88.76 | -5.83 | 93.91 | 90.96 | -2.95 | -3.14 |
| JSON_COMPACT | 9255 | 8727 | -528 | -5.71 | 9026 | 8431 | -595 | -6.59 | 229 | 296 | +67 | +29.37 | 97.53 | 96.61 | -0.92 | 90.17 | 91.49 | +1.31 | +1.46 | 93.53 | 92.51 | -1.02 | 87.51 | 88.75 | +1.25 | +1.43 |
| JSON_PRETTY | 14254 | 13346 | -908 | -6.37 | 13983 | 12963 | -1020 | -7.30 | 271 | 383 | +112 | +41.40 | 98.10 | 97.13 | -0.97 | 72.31 | 74.97 | +2.67 | +3.69 | 95.75 | 92.94 | -2.81 | 70.74 | 72.18 | +1.44 | +2.04 |
| TOON_DEFAULT | 7024 | 11541 | +4517 | +64.30 | 6661 | 11323 | +4662 | +69.99 | 362 | 217 | -145 | -40.18 | 94.84 | 98.12 | +3.28 | 96.52 | 82.23 | -14.30 | -14.81 | 93.15 | 94.30 | +1.15 | 95.40 | 79.68 | -15.72 | -16.48 |
| XML_COMPACT | 11672 | 10909 | -763 | -6.54 | 11475 | 10709 | -766 | -6.68 | 197 | 200 | +3 | +1.76 | 98.31 | 98.16 | -0.15 | 81.87 | 84.56 | +2.69 | +3.28 | 93.91 | 93.88 | -0.03 | 78.94 | 81.70 | +2.76 | +3.50 |
| XML_PRETTY | 16137 | 15076 | -1061 | -6.57 | 15222 | 14676 | -546 | -3.58 | 915 | 400 | -515 | -56.33 | 94.33 | 97.35 | +3.02 | 62.92 | 68.81 | +5.89 | +9.35 | 92.37 | 93.00 | +0.63 | 61.62 | 65.91 | +4.29 | +6.97 |
| YAML | 12533 | 11742 | -791 | -6.31 | 12208 | 11171 | -1037 | -8.49 | 325 | 571 | +246 | +75.71 | 97.41 | 95.14 | -2.27 | 78.13 | 79.50 | +1.37 | +1.76 | 95.18 | 91.37 | -3.81 | 76.64 | 76.99 | +0.35 | +0.45 |

#### 2.8.3 Output Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 11118 | 9064 | -2054 | -18.47 | 10858 | 8335 | -2523 | -23.24 | 260 | 729 | +469 | +180.58 | 97.66 | 91.95 | -5.71 | 82.84 | 94.56 | +11.72 | +14.15 | 94.59 | 88.76 | -5.83 | 80.79 | 92.43 | +11.64 | +14.40 |
| JSON_COMPACT | 11113 | 9070 | -2043 | -18.38 | 10838 | 8762 | -2076 | -19.15 | 274 | 307 | +33 | +12.04 | 97.53 | 96.61 | -0.92 | 82.79 | 97.62 | +14.83 | +17.91 | 93.53 | 92.51 | -1.02 | 80.12 | 94.89 | +14.76 | +18.43 |
| JSON_PRETTY | 13328 | 10627 | -2701 | -20.26 | 13075 | 10323 | -2752 | -21.05 | 253 | 305 | +52 | +20.46 | 98.10 | 97.13 | -0.97 | 66.42 | 86.19 | +19.77 | +29.76 | 95.75 | 92.94 | -2.81 | 64.85 | 83.40 | +18.54 | +28.59 |
| TOON_DEFAULT | 12285 | 11821 | -464 | -3.78 | 11651 | 11599 | -52 | -0.45 | 634 | 222 | -412 | -64.93 | 94.84 | 98.12 | +3.28 | 72.14 | 77.83 | +5.70 | +7.89 | 93.15 | 94.30 | +1.15 | 71.01 | 75.28 | +4.28 | +6.02 |
| XML_COMPACT | 13453 | 11401 | -2052 | -15.25 | 13226 | 11192 | -2034 | -15.38 | 227 | 209 | -18 | -7.74 | 98.31 | 98.16 | -0.15 | 65.61 | 81.03 | +15.41 | +23.49 | 93.91 | 93.88 | -0.03 | 62.68 | 78.17 | +15.49 | +24.71 |
| XML_PRETTY | 9799 | 11478 | +1679 | +17.14 | 9244 | 11175 | +1931 | +20.89 | 556 | 305 | -251 | -45.22 | 94.33 | 97.35 | +3.02 | 90.59 | 79.90 | -10.68 | -11.79 | 92.37 | 93.00 | +0.63 | 89.28 | 77.00 | -12.28 | -13.75 |
| YAML | 11076 | 9110 | -1966 | -17.75 | 10789 | 8667 | -2122 | -19.66 | 287 | 443 | +156 | +54.32 | 97.41 | 95.14 | -2.27 | 82.99 | 96.33 | +13.35 | +16.08 | 95.18 | 91.37 | -3.81 | 81.50 | 93.82 | +12.32 | +15.12 |

#### 2.8.4 Total Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 18813 | 16502 | -2311 | -12.28 | 18372 | 15173 | -3199 | -17.41 | 440 | 1328 | +888 | +201.86 | 97.66 | 91.95 | -5.71 | 91.47 | 94.60 | +3.13 | +3.42 | 94.59 | 88.76 | -5.83 | 89.42 | 92.48 | +3.05 | +3.41 |
| JSON_COMPACT | 20368 | 17797 | -2571 | -12.62 | 19865 | 17194 | -2671 | -13.45 | 503 | 603 | +100 | +19.93 | 97.53 | 96.61 | -0.92 | 86.71 | 93.82 | +7.11 | +8.20 | 93.53 | 92.51 | -1.02 | 84.05 | 91.09 | +7.04 | +8.38 |
| JSON_PRETTY | 27582 | 23973 | -3609 | -13.08 | 27058 | 23285 | -3773 | -13.94 | 524 | 688 | +164 | +31.29 | 98.10 | 97.13 | -0.97 | 65.43 | 75.62 | +10.19 | +15.57 | 95.75 | 92.94 | -2.81 | 63.86 | 72.83 | +8.96 | +14.03 |
| TOON_DEFAULT | 19308 | 23360 | +4052 | +20.99 | 18312 | 22922 | +4610 | +25.17 | 996 | 439 | -557 | -55.94 | 94.84 | 98.12 | +3.28 | 88.10 | 78.12 | -9.98 | -11.33 | 93.15 | 94.30 | +1.15 | 86.98 | 75.57 | -11.40 | -13.11 |
| XML_COMPACT | 25125 | 22310 | -2815 | -11.20 | 24701 | 21900 | -2801 | -11.34 | 425 | 411 | -14 | -3.32 | 98.31 | 98.16 | -0.15 | 72.95 | 81.30 | +8.35 | +11.45 | 93.91 | 93.88 | -0.03 | 70.01 | 78.45 | +8.43 | +12.04 |
| XML_PRETTY | 25936 | 26554 | +618 | +2.38 | 24466 | 25851 | +1385 | +5.66 | 1471 | 704 | -767 | -52.13 | 94.33 | 97.35 | +3.02 | 67.86 | 68.02 | +0.16 | +0.23 | 92.37 | 93.00 | +0.63 | 66.55 | 65.11 | -1.44 | -2.16 |
| YAML | 23609 | 20852 | -2757 | -11.68 | 22998 | 19839 | -3159 | -13.73 | 611 | 1013 | +402 | +65.79 | 97.41 | 95.14 | -2.27 | 76.90 | 83.67 | +6.76 | +8.80 | 95.18 | 91.37 | -3.81 | 75.41 | 81.15 | +5.74 | +7.61 |

### 2.9 Answer Per Format Breakdown
#### 2.9.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Expected Characters | Output Characters | Correct Characters | Incorrect Characters | Accuracy by Character (%) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 96 | 28 | 0 | 77.69 | 7782 | 7824 | 7640 | 184 | 97.66 |
| CSV | opt | 89 | 35 | 0 | 72.04 | 8451 | 8757 | 8048 | 709 | 91.95 |
| JSON_COMPACT | man | 93 | 31 | 0 | 75.00 | 7782 | 7828 | 7635 | 193 | 97.53 |
| JSON_COMPACT | opt | 95 | 29 | 0 | 76.61 | 8451 | 8595 | 8301 | 293 | 96.61 |
| JSON_PRETTY | man | 99 | 25 | 0 | 79.84 | 7782 | 7811 | 7663 | 148 | 98.10 |
| JSON_PRETTY | opt | 96 | 28 | 0 | 77.42 | 8451 | 8584 | 8337 | 248 | 97.13 |
| TOON_DEFAULT | man | 96 | 28 | 0 | 77.69 | 7782 | 7977 | 7559 | 418 | 94.84 |
| TOON_DEFAULT | opt | 98 | 26 | 0 | 78.90 | 8451 | 8509 | 8349 | 160 | 98.12 |
| XML_COMPACT | man | 96 | 28 | 0 | 77.15 | 7782 | 7808 | 7676 | 132 | 98.31 |
| XML_COMPACT | opt | 96 | 28 | 0 | 77.42 | 8451 | 8489 | 8333 | 156 | 98.16 |
| XML_PRETTY | man | 96 | 28 | 0 | 77.15 | 7782 | 7975 | 7521 | 454 | 94.33 |
| XML_PRETTY | opt | 98 | 26 | 0 | 79.03 | 8451 | 8541 | 8314 | 227 | 97.35 |
| YAML | man | 98 | 26 | 0 | 79.30 | 7782 | 7843 | 7640 | 203 | 97.41 |
| YAML | opt | 90 | 34 | 0 | 72.58 | 8451 | 8585 | 8166 | 419 | 95.14 |

#### 2.9.2 Answers: Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 96 | 89 | -7 | -7.29 | 28 | 35 | +7 | +25.00 | 0 | 0 | 0 | 0.00 | 77.69 | 72.04 | -5.65 |
| JSON_COMPACT | 93 | 95 | +2 | +2.15 | 31 | 29 | -2 | -6.45 | 0 | 0 | 0 | 0.00 | 75.00 | 76.61 | +1.61 |
| JSON_PRETTY | 99 | 96 | -3 | -3.03 | 25 | 28 | +3 | +12.00 | 0 | 0 | 0 | 0.00 | 79.84 | 77.42 | -2.42 |
| TOON_DEFAULT | 96 | 98 | +2 | +1.56 | 28 | 27 | -1 | -5.36 | 0 | 0 | 0 | 0.00 | 77.69 | 78.90 | +1.21 |
| XML_COMPACT | 96 | 96 | 0 | +0.34 | 28 | 28 | 0 | -1.18 | 0 | 0 | 0 | 0.00 | 77.15 | 77.42 | +0.27 |
| XML_PRETTY | 96 | 98 | +2 | +2.43 | 28 | 26 | -2 | -8.32 | 0 | 0 | 0 | 0.00 | 77.15 | 79.03 | +1.88 |
| YAML | 98 | 90 | -8 | -8.50 | 26 | 34 | +8 | +32.04 | 0 | 0 | 0 | 0.00 | 79.30 | 72.58 | -6.72 |

#### 2.9.3 Characters: Mandatory vs Optional Data
| Format | Output Characters Man | Output Characters Opt | Diff | Diff (%) | Correct Characters Man | Correct Characters Opt | Diff | Diff (%) | Incorrect Characters Man | Incorrect Characters Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7824 | 8757 | +933 | +11.92 | 7640 | 8048 | +408 | +5.34 | 184 | 709 | +525 | +285.33 | 97.66 | 91.95 | -5.71 |
| JSON_COMPACT | 7828 | 8594 | +766 | +9.79 | 7635 | 8301 | +666 | +8.72 | 193 | 293 | +100 | +51.99 | 97.53 | 96.61 | -0.92 |
| JSON_PRETTY | 7811 | 8584 | +773 | +9.90 | 7663 | 8337 | +674 | +8.79 | 148 | 247 | +99 | +67.12 | 98.10 | 97.13 | -0.97 |
| TOON_DEFAULT | 7977 | 8509 | +532 | +6.67 | 7559 | 8349 | +789 | +10.44 | 418 | 161 | -257 | -61.60 | 94.84 | 98.12 | 3.28 |
| XML_COMPACT | 7808 | 8489 | +681 | +8.72 | 7676 | 8333 | +657 | +8.56 | 132 | 156 | +24 | +17.93 | 98.31 | 98.16 | -0.15 |
| XML_PRETTY | 7975 | 8541 | +566 | +7.09 | 7521 | 8313 | +792 | +10.53 | 454 | 227 | -227 | -49.93 | 94.33 | 97.35 | 3.02 |
| YAML | 7843 | 8585 | +742 | +9.46 | 7640 | 8166 | +526 | +6.88 | 203 | 419 | +216 | +106.40 | 97.41 | 95.14 | -2.27 |

### 2.10 Accuracy Per Question Category Analysis
#### 2.10.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 77.69 | 98.79 | 70.37 | 58.73 | 50.79 | 94.59 | 37.05 | 20.52 | 12.24 | 6.35 |
| CSV | opt | 72.04 | 84.85 | 69.14 | 60.32 | 53.97 | 88.76 | 31.82 | 20.16 | 12.56 | 6.74 |
| JSON_COMPACT | man | 75.00 | 98.79 | 64.20 | 61.90 | 39.68 | 93.53 | 37.05 | 18.72 | 12.90 | 4.96 |
| JSON_COMPACT | opt | 76.61 | 92.73 | 76.55 | 61.90 | 49.21 | 92.51 | 34.77 | 22.33 | 12.90 | 6.15 |
| JSON_PRETTY | man | 79.84 | 100.00 | 64.20 | 66.66 | 60.32 | 95.75 | 37.50 | 18.72 | 13.89 | 7.54 |
| JSON_PRETTY | opt | 77.42 | 92.73 | 80.25 | 65.08 | 46.03 | 92.94 | 34.77 | 23.41 | 13.56 | 5.76 |
| TOON_DEFAULT | man | 77.69 | 91.21 | 67.90 | 68.26 | 64.29 | 93.15 | 34.21 | 19.81 | 14.22 | 8.03 |
| TOON_DEFAULT | opt | 78.90 | 98.49 | 70.37 | 67.46 | 50.00 | 94.30 | 36.93 | 20.53 | 14.06 | 6.25 |
| XML_COMPACT | man | 77.15 | 100.00 | 62.96 | 63.49 | 49.21 | 93.91 | 37.50 | 18.36 | 13.23 | 6.15 |
| XML_COMPACT | opt | 77.42 | 100.00 | 67.90 | 65.08 | 42.86 | 93.88 | 37.50 | 19.80 | 13.56 | 5.36 |
| XML_PRETTY | man | 77.15 | 90.91 | 64.20 | 63.49 | 71.43 | 92.37 | 34.09 | 18.72 | 13.23 | 8.93 |
| XML_PRETTY | opt | 79.03 | 96.97 | 80.25 | 61.90 | 47.62 | 93.00 | 36.36 | 23.41 | 12.90 | 5.95 |
| YAML | man | 79.30 | 97.57 | 61.73 | 68.25 | 65.08 | 95.18 | 36.59 | 18.00 | 14.22 | 8.13 |
| YAML | opt | 72.58 | 90.30 | 64.20 | 63.49 | 46.03 | 91.37 | 33.87 | 18.72 | 13.23 | 5.76 |

#### 2.10.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 98.79 | 84.85 | -13.94 | 37.05 | 31.82 | -5.23 |
| JSON_COMPACT | 98.79 | 92.73 | -6.06 | 37.05 | 34.77 | -2.27 |
| JSON_PRETTY | 100.00 | 92.73 | -7.27 | 37.50 | 34.77 | -2.73 |
| TOON_DEFAULT | 91.21 | 98.49 | +7.27 | 34.21 | 36.93 | +2.73 |
| XML_COMPACT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| XML_PRETTY | 90.91 | 96.97 | +6.06 | 34.09 | 36.36 | +2.27 |
| YAML | 97.57 | 90.30 | -7.27 | 36.59 | 33.87 | -2.73 |

#### 2.10.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 70.37 | 69.14 | -1.23 | 20.52 | 20.16 | -0.36 |
| JSON_COMPACT | 64.20 | 76.55 | +12.35 | 18.72 | 22.33 | +3.60 |
| JSON_PRETTY | 64.20 | 80.25 | +16.05 | 18.72 | 23.41 | +4.69 |
| TOON_DEFAULT | 67.90 | 70.37 | +2.47 | 19.81 | 20.53 | +0.72 |
| XML_COMPACT | 62.96 | 67.90 | +4.94 | 18.36 | 19.80 | +1.44 |
| XML_PRETTY | 64.20 | 80.25 | +16.05 | 18.72 | 23.41 | +4.69 |
| YAML | 61.73 | 64.20 | +2.47 | 18.00 | 18.72 | +0.72 |

#### 2.10.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 58.73 | 60.32 | +1.59 | 12.24 | 12.56 | +0.33 |
| JSON_COMPACT | 61.90 | 61.90 | 0.00 | 12.90 | 12.90 | 0.00 |
| JSON_PRETTY | 66.66 | 65.08 | -1.58 | 13.89 | 13.56 | -0.33 |
| TOON_DEFAULT | 68.26 | 67.46 | -0.80 | 14.22 | 14.06 | -0.16 |
| XML_COMPACT | 63.49 | 65.08 | +1.59 | 13.23 | 13.56 | +0.33 |
| XML_PRETTY | 63.49 | 61.90 | -1.59 | 13.23 | 12.90 | -0.33 |
| YAML | 68.25 | 63.49 | -4.76 | 14.22 | 13.23 | -0.99 |

#### 2.10.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 50.79 | 53.97 | +3.17 | 6.35 | 6.74 | +0.39 |
| JSON_COMPACT | 39.68 | 49.21 | +9.52 | 4.96 | 6.15 | +1.19 |
| JSON_PRETTY | 60.32 | 46.03 | -14.29 | 7.54 | 5.76 | -1.78 |
| TOON_DEFAULT | 64.29 | 50.00 | -14.29 | 8.03 | 6.25 | -1.78 |
| XML_COMPACT | 49.21 | 42.86 | -6.35 | 6.15 | 5.36 | -0.79 |
| XML_PRETTY | 71.43 | 47.62 | -23.81 | 8.93 | 5.95 | -2.97 |
| YAML | 65.08 | 46.03 | -19.04 | 8.13 | 5.76 | -2.38 |

### 2.11 Accuracy By Character Per Question Category Analysis
#### 2.11.1 Metrics
| Format | Variant | Accuracy By Character (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc By Char (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 97.66 | 99.18 | 97.41 | 91.37 | 79.63 | 94.59 | 37.19 | 28.41 | 19.03 | 9.95 |
| CSV | opt | 91.95 | 87.08 | 96.95 | 87.61 | 76.57 | 88.76 | 32.66 | 28.28 | 18.25 | 9.57 |
| JSON_COMPACT | man | 97.53 | 98.95 | 97.61 | 89.88 | 73.87 | 93.53 | 37.11 | 28.47 | 18.72 | 9.23 |
| JSON_COMPACT | opt | 96.61 | 97.31 | 97.01 | 87.61 | 75.76 | 92.51 | 36.49 | 28.30 | 18.25 | 9.47 |
| JSON_PRETTY | man | 98.10 | 100.00 | 97.48 | 93.75 | 82.31 | 95.75 | 37.50 | 28.43 | 19.53 | 10.29 |
| JSON_PRETTY | opt | 97.13 | 97.40 | 98.00 | 89.97 | 72.73 | 92.94 | 36.52 | 28.58 | 18.74 | 9.09 |
| TOON_DEFAULT | man | 94.84 | 93.65 | 96.13 | 93.15 | 84.67 | 93.15 | 35.12 | 28.04 | 19.41 | 10.59 |
| TOON_DEFAULT | opt | 98.12 | 99.54 | 97.98 | 92.04 | 73.74 | 94.30 | 37.32 | 28.58 | 19.18 | 9.22 |
| XML_COMPACT | man | 98.31 | 100.00 | 98.22 | 87.80 | 75.72 | 93.91 | 37.50 | 28.65 | 18.29 | 9.47 |
| XML_COMPACT | opt | 98.16 | 100.00 | 97.80 | 90.56 | 71.92 | 93.88 | 37.50 | 28.52 | 18.87 | 8.99 |
| XML_PRETTY | man | 94.33 | 92.82 | 95.83 | 91.07 | 85.18 | 92.37 | 34.81 | 27.95 | 18.97 | 10.65 |
| XML_PRETTY | opt | 97.35 | 97.24 | 98.52 | 89.09 | 73.94 | 93.00 | 36.46 | 28.73 | 18.56 | 9.24 |
| YAML | man | 97.41 | 98.40 | 97.31 | 92.26 | 85.39 | 95.18 | 36.90 | 28.38 | 19.22 | 10.67 |
| YAML | opt | 95.14 | 93.26 | 97.77 | 90.56 | 72.12 | 91.37 | 34.97 | 28.52 | 18.87 | 9.02 |

#### 2.11.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 99.18 | 87.08 | -12.10 | 37.19 | 32.66 | -4.54 |
| JSON_COMPACT | 98.95 | 97.31 | -1.64 | 37.11 | 36.49 | -0.62 |
| JSON_PRETTY | 100.00 | 97.40 | -2.60 | 37.50 | 36.52 | -0.98 |
| TOON_DEFAULT | 93.65 | 99.54 | +5.89 | 35.12 | 37.32 | +2.21 |
| XML_COMPACT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| XML_PRETTY | 92.82 | 97.24 | +4.42 | 34.81 | 36.46 | +1.66 |
| YAML | 98.40 | 93.26 | -5.14 | 36.90 | 34.97 | -1.93 |

#### 2.11.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 97.41 | 96.95 | -0.46 | 28.41 | 28.28 | -0.13 |
| JSON_COMPACT | 97.61 | 97.01 | -0.60 | 28.47 | 28.30 | -0.17 |
| JSON_PRETTY | 97.48 | 98.00 | +0.52 | 28.43 | 28.58 | +0.15 |
| TOON_DEFAULT | 96.13 | 97.98 | +1.86 | 28.04 | 28.58 | +0.54 |
| XML_COMPACT | 98.22 | 97.80 | -0.43 | 28.65 | 28.52 | -0.13 |
| XML_PRETTY | 95.83 | 98.52 | +2.69 | 27.95 | 28.73 | +0.79 |
| YAML | 97.31 | 97.77 | +0.46 | 28.38 | 28.52 | +0.13 |

#### 2.11.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 91.37 | 87.61 | -3.76 | 19.03 | 18.25 | -0.78 |
| JSON_COMPACT | 89.88 | 87.61 | -2.27 | 18.72 | 18.25 | -0.47 |
| JSON_PRETTY | 93.75 | 89.97 | -3.78 | 19.53 | 18.74 | -0.79 |
| TOON_DEFAULT | 93.15 | 92.04 | -1.12 | 19.41 | 19.18 | -0.23 |
| XML_COMPACT | 87.80 | 90.56 | +2.77 | 18.29 | 18.87 | +0.58 |
| XML_PRETTY | 91.07 | 89.09 | -1.98 | 18.97 | 18.56 | -0.41 |
| YAML | 92.26 | 90.56 | -1.70 | 19.22 | 18.87 | -0.35 |

#### 2.11.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 79.63 | 76.57 | -3.06 | 9.95 | 9.57 | -0.38 |
| JSON_COMPACT | 73.87 | 75.76 | +1.89 | 9.23 | 9.47 | +0.24 |
| JSON_PRETTY | 82.31 | 72.73 | -9.58 | 10.29 | 9.09 | -1.20 |
| TOON_DEFAULT | 84.67 | 73.74 | -10.93 | 10.59 | 9.22 | -1.37 |
| XML_COMPACT | 75.72 | 71.92 | -3.80 | 9.47 | 8.99 | -0.48 |
| XML_PRETTY | 85.18 | 73.94 | -11.24 | 10.65 | 9.24 | -1.41 |
| YAML | 85.39 | 72.12 | -13.27 | 10.67 | 9.02 | -1.66 |

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

- **Report Generated**: 2026-06-21
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code 2.1.73
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_haiku_4_5/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_haiku_4_5/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_haiku_4_5/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_haiku_4_5/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)