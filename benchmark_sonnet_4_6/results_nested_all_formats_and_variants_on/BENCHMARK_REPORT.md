# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Sonnet 4.6
- **Thinking**: on (medium)
- **Data Structure**: nested
- **Formats Tested**: 7 (JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, TOON_KEYFOLD, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: First iteration

## Executive Summary

This benchmark evaluates token efficiency and information accuracy across 7 file formats using Sonnet 4.6 as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context while a format that accurately conveys information may justify higher token cost.

### Key Findings

1. **TOON_DEFAULT** leads the dense mandatory variant with 32454 total tokens, 99.73% accuracy and a total efficiency score of 94.86. **JSON_COMPACT** leads the sparse optional variant with 30448 total tokens, 97.85% accuracy and a total efficiency score of 98.54. The adaptive encoding of **TOON** collapses into **YAML** style key value pairs when flat records become sparse or objects are nested which removes most of the tabular token advantage that makes **TOON** competitive on flat and dense data. But the **YAML** style nested structure seems to be easier to understand for the model which results in less output tokens even if the read tokens increased compared to **JSON_COMPACT**.
2. **TOON_KEYFOLD** does not improve on **TOON_DEFAULT** in this benchmark because of the tested object shape which only has one field valid for keyfolding. On the optional variant it consumes 35425 total tokens (+7.05% vs its own mandatory run and +6.99% vs **TOON_DEFAULT** optional) while accuracy drops to 96.50%. Keyfolding compacts object wrappers but does not offset the output token growth the model produces when reasoning over sparse data.
3. **XML_PRETTY** needs 43981 total tokens in mandatory (+35.52% vs **TOON_DEFAULT**) at 97.58% accuracy. **JSON_PRETTY** needs 39127 tokens at the same 97.58%. Both rank last or second last on every efficiency score column and should be avoided for LLM consumption.
4. **JSON_COMPACT** delivers 3.19 to 3.23 characters per read token and wins the read ranking in both variants. **XML_COMPACT** reaches slightly higher raw density (3.37 to 3.41) but the overhead from the tags still pushes its total read tokens 35% above **JSON_COMPACT**.
5. Aggregation accuracy collapses under sparsity for every format. Each format loses between 6.35 and 12.70 percentage points on aggregation questions when moving from mandatory to optional data. This is a model behaviour with sparse data not a format property and it dominates the overall accuracy drop observed on the optional variant.
6. **YAML** is the most sparsity sensitive format on read tokens because it drops from 14475 to 11404 read tokens (21.22%) between mandatory and optional due to omitted null valued keys. The same sparsity costs **YAML** 3.22 percentage points of accuracy which is the largest accuracy regression in the benchmark.
7. Every format and variant scores between 99.25% and 100.00% accuracy by character. The complete answer accuracy spread (96.50% to 99.73%) is therefore driven by a small number of answers with minor character errors rather than by fully wrong responses. The format choice should be based on token cost because accuracy alone is effectively a tiebreaker at this precision.

## 1. Methodology

### 1.1 Research Purpose

The underlying question: **Which file format delivers maximum information value per token consumed?**

This requires measuring:
- **Token Cost**: How many tokens does each format consume for equivalent data?
- **Information Fidelity**: How accurately can the model understand and answer questions about the data?
- **Robustness**: How consistent is performance across data variants (mandatory vs optional fields)?

### 1.2 Test Design

#### 1.2.1 Data Generation
- 7 formats tested: JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, TOON_KEYFOLD, XML_COMPACT, XML_PRETTY, YAML
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

These results are specific to Claude Code using the Sonnet 4.6 model. They serve as a rule of thumb for choosing the best file format depending on the use case.
However these values cannot be exactly applied to models of the same family or from other providers as token usage, accuracy and latency depend on specific model architectures and tokenizers. Also file reads will produce different characters depending on the used harness because some add marker characters, line numbers or additional information. While the relative ranking of file formats remains consistent the absolute numbers will vary.
Especially the accuracy and output tokens results will vary because these values are bound to the model size and training, instruction interpretation and reasoning token budget.

## 2. Results

### 2.1 TLDR: Token Efficiency Analysis

*Note: All columns ranked best-to-worst. ↑ = lower value is better (ascending). ↓ = higher value is better (descending).*

#### 2.1.1 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 263s | JSON_COMPACT ≈ 7498 | XML_COMPACT ≈ 198 | TOON_DEFAULT ≈ 20713 | TOON_DEFAULT ≈ 21014 | TOON_DEFAULT ≈ 32454 | TOON_DEFAULT ≈ 99.73% | JSON_COMPACT ≈ 98 | TOON_DEFAULT ≈ 100 | TOON_DEFAULT ≈ 95 | TOON_DEFAULT ≈ 100.00% | JSON_COMPACT ≈ 98 | TOON_DEFAULT ≈ 100 | TOON_DEFAULT ≈ 95 |
| TOON_KEYFOLD (+7.71%) | XML_COMPACT (+35.56%) | YAML (0.00%) | TOON_KEYFOLD (+4.26%) | TOON_KEYFOLD (+3.75%) | XML_COMPACT (+1.25%) | YAML (0.00%) | XML_COMPACT (-8.40%) | TOON_KEYFOLD (-5.98%) | XML_COMPACT (-2.18%) | JSON_COMPACT (-0.04%) | XML_COMPACT (-8.15%) | TOON_KEYFOLD (-5.30%) | XML_COMPACT (-1.28%) |
| JSON_PRETTY (+12.42%) | TOON_KEYFOLD (+50.56%) | JSON_COMPACT (+2.86%) | XML_COMPACT (+8.61%) | XML_COMPACT (+8.00%) | TOON_KEYFOLD (+1.96%) | JSON_COMPACT (-1.08%) | TOON_DEFAULT (-11.14%) | XML_COMPACT (-11.91%) | TOON_KEYFOLD (-2.60%) | YAML (-0.04%) | TOON_KEYFOLD (-11.53%) | XML_COMPACT (-11.03%) | TOON_KEYFOLD (-1.89%) |
| XML_COMPACT (+14.36%) | TOON_DEFAULT (+52.57%) | TOON_KEYFOLD (+4.03%) | YAML (+10.77%) | YAML (+10.13%) | JSON_COMPACT (+2.92%) | TOON_KEYFOLD (-1.34%) | TOON_KEYFOLD (-11.60%) | YAML (-13.71%) | JSON_COMPACT (-3.22%) | XML_PRETTY (-0.31%) | TOON_DEFAULT (-11.75%) | YAML (-13.71%) | JSON_COMPACT (-2.48%) |
| YAML (+19.80%) | YAML (+93.05%) | XML_PRETTY (+46.56%) | JSON_PRETTY (+11.25%) | JSON_PRETTY (+11.07%) | YAML (+15.91%) | XML_COMPACT (-1.61%) | YAML (-20.29%) | JSON_PRETTY (-16.42%) | YAML (-13.39%) | XML_COMPACT (-0.32%) | YAML (-20.84%) | JSON_PRETTY (-15.46%) | YAML (-13.39%) |
| XML_PRETTY (+22.96%) | JSON_PRETTY (+110.55%) | JSON_PRETTY (+49.41%) | XML_PRETTY (+22.62%) | XML_PRETTY (+22.25%) | JSON_PRETTY (+20.56%) | JSON_PRETTY (-2.15%) | JSON_PRETTY (-25.71%) | XML_PRETTY (-31.56%) | JSON_PRETTY (-18.81%) | TOON_KEYFOLD (-0.34%) | JSON_PRETTY (-25.24%) | XML_PRETTY (-30.27%) | JSON_PRETTY (-17.80%) |
| JSON_COMPACT (+23.07%) | XML_PRETTY (+143.95%) | TOON_DEFAULT (+51.60%) | JSON_COMPACT (+24.08%) | JSON_COMPACT (+23.27%) | XML_PRETTY (+35.52%) | XML_PRETTY (-2.15%) | XML_PRETTY (-33.26%) | JSON_COMPACT (-32.22%) | XML_PRETTY (-31.40%) | JSON_PRETTY (-0.75%) | XML_PRETTY (-32.42%) | JSON_COMPACT (-31.47%) | XML_PRETTY (-30.05%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 279s | JSON_COMPACT ≈ 6970 | JSON_PRETTY ≈ 201 | TOON_DEFAULT ≈ 21613 | TOON_DEFAULT ≈ 21911 | JSON_COMPACT ≈ 30448 | JSON_COMPACT ≈ 97.85% | JSON_COMPACT ≈ 99 | TOON_DEFAULT ≈ 92 | JSON_COMPACT ≈ 99 | JSON_PRETTY ≈ 99.88% | JSON_COMPACT ≈ 100 | TOON_DEFAULT ≈ 94 | JSON_COMPACT ≈ 100 |
| JSON_COMPACT (+9.79%) | XML_COMPACT (+38.94%) | XML_PRETTY (+45.18%) | YAML (+5.20%) | YAML (+5.12%) | TOON_DEFAULT (+8.73%) | XML_PRETTY (0.00%) | XML_COMPACT (-8.28%) | YAML (-8.02%) | TOON_DEFAULT (-7.36%) | JSON_COMPACT (-0.01%) | XML_COMPACT (-8.03%) | YAML (-7.82%) | TOON_DEFAULT (-6.59%) |
| TOON_KEYFOLD (+10.93%) | TOON_KEYFOLD (+58.45%) | YAML (+47.34%) | JSON_COMPACT (+7.26%) | JSON_COMPACT (+7.15%) | YAML (+13.10%) | JSON_PRETTY (-0.27%) | TOON_KEYFOLD (-13.07%) | JSON_COMPACT (-10.16%) | YAML (-10.86%) | XML_COMPACT (-0.07%) | TOON_KEYFOLD (-12.16%) | JSON_COMPACT (-10.66%) | YAML (-10.02%) |
| JSON_PRETTY (+11.73%) | TOON_DEFAULT (+60.62%) | JSON_COMPACT (+47.51%) | JSON_PRETTY (+8.61%) | JSON_PRETTY (+8.05%) | TOON_KEYFOLD (+16.35%) | XML_COMPACT (-0.27%) | TOON_DEFAULT (-13.33%) | JSON_PRETTY (-11.73%) | TOON_KEYFOLD (-13.34%) | XML_PRETTY (-0.07%) | TOON_DEFAULT (-12.48%) | JSON_PRETTY (-11.99%) | TOON_KEYFOLD (-12.43%) |
| YAML (+12.26%) | YAML (+63.62%) | XML_COMPACT (+48.01%) | TOON_KEYFOLD (+11.40%) | TOON_KEYFOLD (+11.28%) | XML_COMPACT (+17.79%) | TOON_DEFAULT (-1.08%) | YAML (-14.13%) | TOON_KEYFOLD (-17.45%) | XML_COMPACT (-13.70%) | TOON_DEFAULT (-0.08%) | YAML (-13.24%) | TOON_KEYFOLD (-17.02%) | XML_COMPACT (-13.38%) |
| XML_PRETTY (+15.99%) | JSON_PRETTY (+113.17%) | TOON_DEFAULT (+48.34%) | XML_PRETTY (+14.29%) | XML_PRETTY (+14.06%) | JSON_PRETTY (+26.55%) | YAML (-1.34%) | JSON_PRETTY (-23.71%) | XML_PRETTY (-20.74%) | JSON_PRETTY (-20.36%) | TOON_KEYFOLD (-0.27%) | JSON_PRETTY (-23.20%) | XML_PRETTY (-21.05%) | JSON_PRETTY (-19.90%) |
| XML_COMPACT (+17.98%) | XML_PRETTY (+154.81%) | TOON_KEYFOLD (+51.49%) | XML_COMPACT (+19.76%) | XML_COMPACT (+19.49%) | XML_PRETTY (+40.41%) | TOON_KEYFOLD (-1.35%) | XML_PRETTY (-32.18%) | XML_COMPACT (-29.24%) | XML_PRETTY (-30.71%) | YAML (-0.30%) | XML_PRETTY (-31.79%) | XML_COMPACT (-29.17%) | XML_PRETTY (-30.34%) |


#### 2.1.2 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 100.00% | JSON_COMPACT ≈ 100.00% | YAML ≈ 100.00% | TOON_DEFAULT ≈ 100.00% |
| JSON_PRETTY (0.00%) | TOON_DEFAULT (0.00%) | TOON_DEFAULT (-1.59%) | XML_PRETTY (0.00%) |
| TOON_DEFAULT (0.00%) | XML_PRETTY (0.00%) | JSON_COMPACT (-4.76%) | YAML (0.00%) |
| TOON_KEYFOLD (0.00%) | YAML (0.00%) | TOON_KEYFOLD (-4.76%) | TOON_KEYFOLD (-1.59%) |
| XML_PRETTY (-0.61%) | XML_COMPACT (-1.23%) | XML_COMPACT (-4.76%) | XML_COMPACT (-1.59%) |
| YAML (-0.61%) | TOON_KEYFOLD (-2.47%) | JSON_PRETTY (-6.35%) | JSON_COMPACT (-3.17%) |
| XML_COMPACT (-1.21%) | JSON_PRETTY (-3.70%) | XML_PRETTY (-12.70%) | JSON_PRETTY (-3.17%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 100.00% | JSON_PRETTY ≈ 100.00% | TOON_DEFAULT ≈ 100.00% | XML_PRETTY ≈ 92.07% |
| JSON_PRETTY (0.00%) | JSON_COMPACT (-1.23%) | YAML (0.00%) | JSON_COMPACT (-1.59%) |
| TOON_DEFAULT (0.00%) | XML_COMPACT (-1.23%) | JSON_COMPACT (-1.59%) | JSON_PRETTY (-1.59%) |
| TOON_KEYFOLD (0.00%) | XML_PRETTY (-1.23%) | TOON_KEYFOLD (-1.59%) | TOON_DEFAULT (-1.59%) |
| XML_COMPACT (0.00%) | YAML (-6.17%) | XML_COMPACT (-3.17%) | TOON_KEYFOLD (-1.59%) |
| XML_PRETTY (0.00%) | TOON_DEFAULT (-7.41%) | XML_PRETTY (-3.17%) | XML_COMPACT (-1.59%) |
| YAML (0.00%) | TOON_KEYFOLD (-7.41%) | JSON_PRETTY (-4.76%) | YAML (-4.76%) |


#### 2.1.3 Category Accuracy By Character Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 100.00% | JSON_COMPACT ≈ 100.00% | YAML ≈ 100.00% | TOON_DEFAULT ≈ 100.00% |
| JSON_PRETTY (0.00%) | TOON_DEFAULT (0.00%) | TOON_DEFAULT (-0.30%) | XML_PRETTY (0.00%) |
| TOON_DEFAULT (0.00%) | XML_PRETTY (0.00%) | JSON_COMPACT (-0.89%) | YAML (0.00%) |
| TOON_KEYFOLD (0.00%) | YAML (0.00%) | TOON_KEYFOLD (-0.89%) | TOON_KEYFOLD (-0.62%) |
| YAML (-0.11%) | XML_COMPACT (-0.06%) | XML_COMPACT (-0.89%) | XML_COMPACT (-0.62%) |
| XML_COMPACT (-0.66%) | TOON_KEYFOLD (-0.55%) | JSON_PRETTY (-1.19%) | JSON_PRETTY (-1.03%) |
| XML_PRETTY (-0.69%) | JSON_PRETTY (-1.23%) | XML_PRETTY (-2.38%) | JSON_COMPACT (-1.23%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 100.00% | JSON_PRETTY ≈ 100.00% | TOON_DEFAULT ≈ 100.00% | XML_PRETTY ≈ 96.76% |
| JSON_PRETTY (0.00%) | JSON_COMPACT (-0.03%) | YAML (0.00%) | JSON_COMPACT (-2.21%) |
| TOON_DEFAULT (0.00%) | XML_COMPACT (-0.15%) | JSON_COMPACT (-0.29%) | JSON_PRETTY (-2.21%) |
| TOON_KEYFOLD (0.00%) | TOON_DEFAULT (-0.18%) | TOON_KEYFOLD (-0.29%) | TOON_DEFAULT (-2.21%) |
| XML_COMPACT (0.00%) | XML_PRETTY (-0.22%) | XML_COMPACT (-0.59%) | TOON_KEYFOLD (-2.21%) |
| XML_PRETTY (0.00%) | TOON_KEYFOLD (-0.54%) | XML_PRETTY (-0.59%) | XML_COMPACT (-2.21%) |
| YAML (0.00%) | YAML (-0.55%) | JSON_PRETTY (-0.88%) | YAML (-3.42%) |


#### 2.1.4 Conclusion

- For mandatory data **TOON_DEFAULT** is the strongest choice. It produces the lowest output tokens (21014), the lowest total tokens (32454), the highest accuracy (99.73%), a perfect 100.00% accuracy by character and the best combined total efficiency score (94.86). Its read cost is not the lowest but the output token savings dominate the total cost because output tokens outweigh read tokens by roughly 2 to 1 on every format in this benchmark. The total tokens difference between **JSON_COMPACT** and **TOON_DEFAULT** is only 2.92% because **JSON_COMAPCT** used 23.27% more output tokens and 52.57% less read tokens.
- For optional data **JSON_COMPACT** wins because it used less output tokens for sparse data compared to dense data. It reaches the lowest read tokens (6970), the lowest total tokens (30448) and the best total efficiency score (98.54). **TOON_DEFAULT** falls to second on totals because its output tokens grow by 4.27% from mandatory to optional while its accuracy drops 2.96 percentage points. **TOON_DEFAULT** still uses the least output tokens overall but **JSON_COMPACT** used only 7.15% more than it comapred to 23.27% more on the dense data which results in **JSON_COMPACT** using the least total tokens while **TOON_DEFAULT** used 8.73% more total tokens.
- **JSON_COMPACT** has the smallest accuracy regression between variants (0.80 percentage points) and therefore the most predictable behaviour across data shapes. **YAML** ties **TOON_DEFAULT** for highest mandatory accuracy (99.73%) but regresses the most in optional (3.22 points) which makes it the least robust of the top accuracy formats. **XML_PRETTY** is the only format whose accuracy improves slightly on the optional variant (+0.27 points) but it pays for this with the highest token cost in both variants and never reaches the top half of any efficiency score ranking. **TOON_KEYFOLD** is consistently worse than **TOON_DEFAULT** in this nested benchmark on both accuracy and output tokens which might be due to the format pattern breaking when single field objects are collapsed.
- Use **TOON_DEFAULT** when records are uniformly populated and output token cost dominates. Use **JSON_COMPACT** when records contain optional fields, when read token cost dominates or when a single format must serve mixed data shapes. Avoid **JSON_PRETTY** and **XML_PRETTY** for LLM input because they add 20% to 36% total tokens with no accuracy return. Prefer **TOON_DEFAULT** over **TOON_KEYFOLD** because keyfolding did not pay off in either variant of this benchmark and the pattern break might hurt accuracy.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total | Accuracy By Character (%) | Useful Read Tokens (Acc By Char) | Wasted Read Tokens (Acc By Char) | Useful Output Tokens (Acc By Char) | Wasted Output Tokens (Acc By Char) | Eff Score Read (Acc By Char) | Eff Score Output (Acc By Char) | Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 7498 | 25905 | 33403 | 3.19 | 207.26 | 98.65 | 7397 | 101 | 25555 | 350 | 97.52 | 67.61 | 91.81 | 99.96 | 7495 | 3 | 25894 | 10 | 98.39 | 68.48 | 92.68 |
| JSON_COMPACT | opt | 6970 | 23478 | 30448 | 3.23 | 186.95 | 97.85 | 6820 | 150 | 22973 | 505 | 98.54 | 82.67 | 98.54 | 99.87 | 6961 | 9 | 23447 | 31 | 99.88 | 84.02 | 99.89 |
| JSON_PRETTY | man | 15787 | 23340 | 39127 | 2.09 | 185.84 | 97.58 | 15405 | 382 | 22775 | 565 | 72.44 | 83.37 | 77.02 | 99.25 | 15669 | 118 | 23165 | 175 | 73.56 | 84.49 | 78.13 |
| JSON_PRETTY | opt | 14858 | 23674 | 38532 | 2.11 | 189.30 | 97.58 | 14498 | 360 | 23101 | 573 | 75.17 | 81.23 | 78.48 | 99.88 | 14840 | 18 | 23646 | 28 | 76.71 | 82.76 | 80.01 |
| TOON_DEFAULT | man | 11440 | 21014 | 32454 | 2.37 | 167.04 | 99.73 | 11409 | 31 | 20957 | 57 | 86.65 | 99.76 | 94.86 | 100.00 | 11440 | 0 | 21014 | 0 | 86.83 | 99.94 | 95.04 |
| TOON_DEFAULT | opt | 11195 | 21911 | 33106 | 2.39 | 174.30 | 96.77 | 10833 | 362 | 21203 | 708 | 85.40 | 92.02 | 91.29 | 99.80 | 11173 | 22 | 21867 | 44 | 87.42 | 94.04 | 93.31 |
| TOON_KEYFOLD | man | 11289 | 21803 | 33092 | 2.38 | 174.16 | 98.39 | 11107 | 182 | 21452 | 351 | 86.20 | 93.79 | 92.40 | 99.66 | 11251 | 38 | 21729 | 74 | 87.05 | 94.64 | 93.25 |
| TOON_KEYFOLD | opt | 11044 | 24381 | 35425 | 2.41 | 194.17 | 96.50 | 10657 | 387 | 23528 | 853 | 85.66 | 75.97 | 85.40 | 99.61 | 11001 | 43 | 24286 | 95 | 87.74 | 78.04 | 87.47 |
| XML_COMPACT | man | 10164 | 22696 | 32860 | 3.37 | 181.43 | 98.12 | 9973 | 191 | 22269 | 427 | 89.33 | 87.87 | 92.79 | 99.68 | 10131 | 33 | 22623 | 73 | 90.37 | 88.91 | 93.83 |
| XML_COMPACT | opt | 9684 | 26182 | 35866 | 3.41 | 208.75 | 97.58 | 9450 | 234 | 25548 | 634 | 90.38 | 65.12 | 85.04 | 99.81 | 9666 | 18 | 26132 | 50 | 91.87 | 66.60 | 86.52 |
| XML_PRETTY | man | 18291 | 25690 | 43981 | 2.32 | 204.83 | 97.58 | 17848 | 443 | 25068 | 622 | 65.08 | 68.28 | 65.08 | 99.69 | 18234 | 57 | 25610 | 80 | 66.49 | 69.68 | 66.48 |
| XML_PRETTY | opt | 17760 | 24992 | 42752 | 2.32 | 199.20 | 97.85 | 17378 | 382 | 24455 | 537 | 66.82 | 72.94 | 68.28 | 99.81 | 17726 | 34 | 24945 | 47 | 68.13 | 74.25 | 69.59 |
| YAML | man | 14475 | 23143 | 37618 | 1.84 | 185.04 | 99.73 | 14436 | 39 | 23080 | 62 | 77.73 | 86.08 | 82.16 | 99.96 | 14469 | 6 | 23133 | 9 | 77.89 | 86.23 | 82.31 |
| YAML | opt | 11404 | 23033 | 34437 | 2.30 | 183.36 | 96.51 | 11006 | 398 | 22229 | 804 | 84.61 | 84.64 | 87.84 | 99.58 | 11356 | 48 | 22936 | 97 | 86.66 | 86.68 | 89.88 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7498 | 6970 | -528 | -7.04 | 204 | 296 | +92 | +45.10 | 25701 | 23182 | -2519 | -9.80 | 25905 | 23478 | -2427 | -9.37 | 33403 | 30448 | -2955 | -8.85 |
| JSON_PRETTY | 15787 | 14858 | -929 | -5.88 | 296 | 200 | -96 | -32.43 | 23044 | 23473 | +429 | +1.86 | 23340 | 23674 | +334 | +1.43 | 39127 | 38532 | -595 | -1.52 |
| TOON_DEFAULT | 11440 | 11195 | -245 | -2.14 | 301 | 298 | -3 | -1.00 | 20713 | 21613 | +900 | +4.35 | 21014 | 21911 | +897 | +4.27 | 32454 | 33106 | +652 | +2.01 |
| TOON_KEYFOLD | 11289 | 11044 | -245 | -2.17 | 206 | 304 | +98 | +47.57 | 21596 | 24077 | +2481 | +11.49 | 21803 | 24382 | +2579 | +11.83 | 33092 | 35426 | +2334 | +7.05 |
| XML_COMPACT | 10164 | 9684 | -480 | -4.72 | 198 | 297 | +99 | +50.00 | 22498 | 25885 | +3387 | +15.05 | 22696 | 26182 | +3486 | +15.36 | 32860 | 35866 | +3006 | +9.15 |
| XML_PRETTY | 18291 | 17760 | -531 | -2.90 | 291 | 292 | +1 | +0.34 | 25399 | 24701 | -698 | -2.75 | 25690 | 24992 | -698 | -2.72 | 43981 | 42752 | -1229 | -2.79 |
| YAML | 14475 | 11404 | -3071 | -21.22 | 198 | 295 | +97 | +48.99 | 22944 | 22737 | -207 | -0.90 | 23143 | 23033 | -110 | -0.48 | 37618 | 34437 | -3181 | -8.46 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Before Write (ms) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) | Output (ms) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 22 | 340.82 | 0.71 | 259358 | 64156 | 0.40 | 517.39 | 64178 | 341.22 | 414.05 | 323514 |
| JSON_COMPACT | opt | 20 | 348.50 | 0.65 | 237622 | 69106 | 0.34 | 557.31 | 69126 | 348.84 | 445.97 | 306728 |
| JSON_PRETTY | man | 311 | 50.76 | 10.03 | 230440 | 65068 | 0.35 | 524.74 | 65379 | 51.12 | 421.80 | 295508 |
| JSON_PRETTY | opt | 366 | 40.60 | 11.81 | 251188 | 60978 | 0.39 | 491.76 | 61344 | 40.98 | 395.77 | 312166 |
| TOON_DEFAULT | man | 36 | 317.78 | 1.16 | 201764 | 61105 | 0.34 | 492.78 | 61141 | 318.12 | 394.46 | 262869 |
| TOON_DEFAULT | opt | 30 | 373.17 | 0.97 | 219887 | 59498 | 0.36 | 479.82 | 59528 | 373.53 | 384.05 | 279384 |
| TOON_KEYFOLD | man | 12 | 940.75 | 0.39 | 215383 | 67762 | 0.32 | 546.47 | 67774 | 941.07 | 437.25 | 283145 |
| TOON_KEYFOLD | opt | 30 | 368.13 | 0.97 | 242454 | 67459 | 0.36 | 544.02 | 67489 | 368.49 | 435.41 | 309913 |
| XML_COMPACT | man | 35 | 290.40 | 1.13 | 238617 | 62006 | 0.36 | 500.05 | 62041 | 290.76 | 400.26 | 300623 |
| XML_COMPACT | opt | 10 | 968.40 | 0.32 | 271328 | 58287 | 0.44 | 470.06 | 58297 | 968.84 | 376.11 | 329615 |
| XML_PRETTY | man | 29 | 630.72 | 0.94 | 253745 | 69478 | 0.37 | 560.31 | 69507 | 631.09 | 448.43 | 323223 |
| XML_PRETTY | opt | 30 | 592.00 | 0.97 | 261736 | 62333 | 0.40 | 502.69 | 62363 | 592.40 | 402.34 | 324069 |
| YAML | man | 28 | 516.96 | 0.90 | 236909 | 78001 | 0.29 | 629.04 | 78029 | 517.26 | 503.41 | 314910 |
| YAML | opt | 44 | 259.18 | 1.42 | 250468 | 63159 | 0.36 | 509.35 | 63203 | 259.54 | 407.76 | 313627 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Before Write Man (s) | Output Before Write Opt (s) | Diff (s) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 22 | 20 | -2 | -9.09 | 259.36 | 237.62 | -21.74 | -8.38 | 64.16 | 69.11 | +4.95 | +7.72 | 64.18 | 69.13 | +4.95 | +7.71 | 323.51 | 306.73 | -16.79 | -5.19 |
| JSON_PRETTY | 311 | 366 | +55 | +17.68 | 230.44 | 251.19 | +20.75 | +9.00 | 65.07 | 60.98 | -4.09 | -6.29 | 65.38 | 61.34 | -4.03 | -6.17 | 295.51 | 312.17 | +16.66 | +5.64 |
| TOON_DEFAULT | 36 | 30 | -6 | -16.67 | 201.76 | 219.89 | +18.12 | +8.98 | 61.11 | 59.50 | -1.61 | -2.63 | 61.14 | 59.53 | -1.61 | -2.64 | 262.87 | 279.38 | +16.52 | +6.28 |
| TOON_KEYFOLD | 12 | 30 | +18 | +150.00 | 215.38 | 242.45 | +27.07 | +12.57 | 67.76 | 67.46 | -0.30 | -0.45 | 67.77 | 67.49 | -0.28 | -0.42 | 283.14 | 309.91 | +26.77 | +9.45 |
| XML_COMPACT | 35 | 10 | -25 | -71.43 | 238.62 | 271.33 | +32.71 | +13.71 | 62.01 | 58.29 | -3.72 | -6.00 | 62.04 | 58.30 | -3.74 | -6.03 | 300.62 | 329.61 | +28.99 | +9.64 |
| XML_PRETTY | 29 | 30 | +1 | +3.45 | 253.74 | 261.74 | +7.99 | +3.15 | 69.48 | 62.33 | -7.14 | -10.28 | 69.51 | 62.36 | -7.14 | -10.28 | 323.22 | 324.07 | +0.85 | +0.26 |
| YAML | 28 | 44 | +16 | +57.14 | 236.91 | 250.47 | +13.56 | +5.72 | 78.00 | 63.16 | -14.84 | -19.03 | 78.03 | 63.20 | -14.83 | -19.00 | 314.91 | 313.63 | -1.28 | -0.41 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token | Info / Read Token (Acc By Char) | Info / Output Token (Acc By Char) | Info / Total Token (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 3.19 | 10.99 | 241.87 | 1.32 | 0.38 | 0.30 | 1.33 | 0.39 | 0.30 |
| JSON_COMPACT | opt | 3.23 | 11.05 | 224.84 | 1.40 | 0.42 | 0.32 | 1.43 | 0.43 | 0.33 |
| JSON_PRETTY | man | 2.09 | 23.15 | 509.26 | 0.62 | 0.42 | 0.25 | 0.63 | 0.43 | 0.25 |
| JSON_PRETTY | opt | 2.11 | 23.55 | 479.29 | 0.66 | 0.41 | 0.25 | 0.67 | 0.42 | 0.26 |
| TOON_DEFAULT | man | 2.37 | 16.77 | 369.03 | 0.87 | 0.48 | 0.31 | 0.87 | 0.48 | 0.31 |
| TOON_DEFAULT | opt | 2.39 | 17.74 | 361.13 | 0.86 | 0.44 | 0.29 | 0.89 | 0.46 | 0.30 |
| TOON_KEYFOLD | man | 2.38 | 16.55 | 364.16 | 0.87 | 0.45 | 0.30 | 0.88 | 0.46 | 0.30 |
| TOON_KEYFOLD | opt | 2.41 | 17.50 | 356.26 | 0.87 | 0.40 | 0.27 | 0.90 | 0.41 | 0.28 |
| XML_COMPACT | man | 3.37 | 14.90 | 327.87 | 0.97 | 0.43 | 0.30 | 0.98 | 0.44 | 0.30 |
| XML_COMPACT | opt | 3.41 | 15.35 | 312.39 | 1.01 | 0.37 | 0.27 | 1.03 | 0.38 | 0.28 |
| XML_PRETTY | man | 2.32 | 26.82 | 590.03 | 0.53 | 0.38 | 0.22 | 0.55 | 0.39 | 0.23 |
| XML_PRETTY | opt | 2.32 | 28.15 | 572.90 | 0.55 | 0.39 | 0.23 | 0.56 | 0.40 | 0.23 |
| YAML | man | 1.84 | 21.22 | 466.94 | 0.69 | 0.43 | 0.27 | 0.69 | 0.43 | 0.27 |
| YAML | opt | 2.30 | 18.07 | 367.87 | 0.85 | 0.42 | 0.28 | 0.87 | 0.43 | 0.29 |

#### 2.5.2 Characters And Values: Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 3.19 | 3.23 | +0.04 | +1.13 | 10.99 | 11.05 | +0.05 | +0.47 | 241.87 | 224.84 | -17.03 | -7.04 |
| JSON_PRETTY | 2.09 | 2.11 | +0.01 | +0.53 | 23.15 | 23.55 | +0.40 | +1.72 | 509.26 | 479.29 | -29.97 | -5.88 |
| TOON_DEFAULT | 2.37 | 2.39 | +0.02 | +1.01 | 16.77 | 17.74 | +0.97 | +5.77 | 369.03 | 361.13 | -7.90 | -2.14 |
| TOON_KEYFOLD | 2.38 | 2.41 | +0.02 | +1.05 | 16.55 | 17.50 | +0.95 | +5.73 | 364.16 | 356.26 | -7.90 | -2.17 |
| XML_COMPACT | 3.37 | 3.41 | +0.04 | +1.07 | 14.90 | 15.35 | +0.44 | +2.98 | 327.87 | 312.39 | -15.48 | -4.72 |
| XML_PRETTY | 2.32 | 2.32 | 0.00 | 0.00 | 26.82 | 28.15 | +1.33 | +4.94 | 590.03 | 572.90 | -17.13 | -2.90 |
| YAML | 1.84 | 2.30 | +0.47 | +25.40 | 21.22 | 18.07 | -3.15 | -14.85 | 466.94 | 367.87 | -99.06 | -21.22 |

#### 2.5.3 Information: Mandatory vs Optional
| Format | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 1.32 | 1.40 | +0.09 | +6.69 | 0.38 | 0.42 | +0.04 | +9.45 | 0.30 | 0.32 | +0.03 | +8.81 |
| JSON_PRETTY | 0.62 | 0.66 | +0.04 | +6.31 | 0.42 | 0.41 | -0.01 | -1.44 | 0.25 | 0.25 | 0.00 | 0.00 |
| TOON_DEFAULT | 0.87 | 0.86 | -0.01 | -0.92 | 0.48 | 0.44 | -0.03 | -6.95 | 0.31 | 0.29 | -0.02 | -4.89 |
| TOON_KEYFOLD | 0.87 | 0.87 | 0.00 | 0.00 | 0.45 | 0.40 | -0.05 | -12.20 | 0.30 | 0.27 | -0.02 | -8.42 |
| XML_COMPACT | 0.97 | 1.01 | +0.04 | +4.46 | 0.43 | 0.37 | -0.06 | -13.66 | 0.30 | 0.27 | -0.03 | -9.03 |
| XML_PRETTY | 0.53 | 0.55 | +0.02 | +3.38 | 0.38 | 0.39 | +0.01 | +3.16 | 0.22 | 0.23 | +0.01 | +3.15 |
| YAML | 0.69 | 0.85 | +0.16 | +22.79 | 0.43 | 0.42 | -0.01 | -2.78 | 0.27 | 0.28 | +0.02 | +5.66 |

#### 2.5.4 Information (Accuracy By Character): Mandatory vs Optional
| Format | Info / Read Token (Acc By Char) Man | Info / Read Token (Acc By Char) Opt | Diff | Diff (%) | Info / Output Token (Acc By Char) Man | Info / Output Token (Acc By Char)  Opt | Diff | Diff (%) | Info / Total Token (Acc By Char) Man | Info / Total Token (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 1.33 | 1.43 | +0.10 | +7.50 | 0.39 | 0.43 | +0.04 | +10.10 | 0.30 | 0.33 | +0.03 | +9.70 |
| JSON_PRETTY | 0.63 | 0.67 | +0.04 | +6.84 | 0.43 | 0.42 | 0.00 | 0.00 | 0.25 | 0.26 | +0.01 | +1.97 |
| TOON_DEFAULT | 0.87 | 0.89 | +0.02 | +1.95 | 0.48 | 0.46 | -0.02 | -4.41 | 0.31 | 0.30 | -0.01 | -2.27 |
| TOON_KEYFOLD | 0.88 | 0.90 | +0.02 | +2.15 | 0.46 | 0.41 | -0.05 | -10.50 | 0.30 | 0.28 | -0.02 | -6.64 |
| XML_COMPACT | 0.98 | 1.03 | +0.05 | +5.10 | 0.44 | 0.38 | -0.06 | -13.21 | 0.30 | 0.28 | -0.02 | -8.25 |
| XML_PRETTY | 0.55 | 0.56 | +0.02 | +3.12 | 0.39 | 0.40 | +0.01 | +2.84 | 0.23 | 0.23 | +0.01 | +2.64 |
| YAML | 0.69 | 0.87 | +0.18 | +26.34 | 0.43 | 0.43 | 0.00 | 0.00 | 0.27 | 0.29 | +0.02 | +8.65 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 7498 | 7397 | 101 | 25905 | 25555 | 350 | 33403 | 32952 | 451 | 98.65 | 97.52 | 67.61 | 91.81 | 98.61 | 97.49 | 67.58 | 91.78 |
| JSON_COMPACT | opt | 6970 | 6820 | 150 | 23478 | 22973 | 505 | 30448 | 29793 | 655 | 97.85 | 98.54 | 82.67 | 98.54 | 98.12 | 98.72 | 82.85 | 98.72 |
| JSON_PRETTY | man | 15787 | 15405 | 382 | 23340 | 22775 | 565 | 39127 | 38180 | 947 | 97.58 | 72.44 | 83.37 | 77.02 | 97.20 | 72.19 | 83.12 | 76.76 |
| JSON_PRETTY | opt | 14858 | 14498 | 360 | 23674 | 23101 | 573 | 38532 | 37600 | 932 | 97.58 | 75.17 | 81.23 | 78.48 | 97.82 | 75.33 | 81.39 | 78.64 |
| TOON_DEFAULT | man | 11440 | 11409 | 31 | 21014 | 20957 | 57 | 32454 | 32366 | 88 | 99.73 | 86.65 | 99.76 | 94.86 | 99.67 | 86.61 | 99.72 | 94.82 |
| TOON_DEFAULT | opt | 11195 | 10833 | 362 | 21911 | 21203 | 708 | 33106 | 32036 | 1069 | 96.77 | 85.40 | 92.02 | 91.29 | 96.65 | 85.32 | 91.94 | 91.21 |
| TOON_KEYFOLD | man | 11289 | 11107 | 182 | 21803 | 21452 | 351 | 33092 | 32559 | 533 | 98.39 | 86.20 | 93.79 | 92.40 | 98.09 | 86.00 | 93.59 | 92.20 |
| TOON_KEYFOLD | opt | 11044 | 10657 | 387 | 24381 | 23528 | 853 | 35425 | 34185 | 1240 | 96.50 | 85.66 | 75.97 | 85.40 | 96.32 | 85.54 | 75.85 | 85.28 |
| XML_COMPACT | man | 10164 | 9973 | 191 | 22696 | 22269 | 427 | 32860 | 32242 | 618 | 98.12 | 89.33 | 87.87 | 92.79 | 98.00 | 89.25 | 87.79 | 92.71 |
| XML_COMPACT | opt | 9684 | 9450 | 234 | 26182 | 25548 | 634 | 35866 | 34998 | 868 | 97.58 | 90.38 | 65.12 | 85.04 | 97.79 | 90.52 | 65.26 | 85.18 |
| XML_PRETTY | man | 18291 | 17848 | 443 | 25690 | 25068 | 622 | 43981 | 42917 | 1064 | 97.58 | 65.08 | 68.28 | 65.08 | 97.13 | 64.78 | 67.98 | 64.78 |
| XML_PRETTY | opt | 17760 | 17378 | 382 | 24992 | 24455 | 537 | 42752 | 41833 | 919 | 97.85 | 66.82 | 72.94 | 68.28 | 97.99 | 66.92 | 73.03 | 68.37 |
| YAML | man | 14475 | 14436 | 39 | 23143 | 23080 | 62 | 37618 | 37516 | 102 | 99.73 | 77.73 | 86.08 | 82.16 | 99.77 | 77.76 | 86.10 | 82.19 |
| YAML | opt | 11404 | 11006 | 398 | 23033 | 22229 | 804 | 34437 | 33235 | 1202 | 96.51 | 84.61 | 84.64 | 87.84 | 96.61 | 84.68 | 84.70 | 87.90 |

#### 2.6.2 Read Tokens: Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7498 | 6970 | -528 | -7.04 | 7397 | 6820 | -577 | -7.80 | 101 | 150 | +49 | +48.15 | 98.65 | 97.85 | -0.80 | 97.52 | 98.54 | +1.02 | +1.04 | 98.61 | 98.12 | -0.49 | 97.49 | 98.72 | +1.22 | +1.26 |
| JSON_PRETTY | 15787 | 14858 | -929 | -5.88 | 15405 | 14498 | -907 | -5.88 | 382 | 360 | -22 | -5.89 | 97.58 | 97.58 | 0.00 | 72.44 | 75.17 | +2.73 | +3.77 | 97.20 | 97.82 | +0.62 | 72.19 | 75.33 | +3.14 | +4.36 |
| TOON_DEFAULT | 11440 | 11195 | -245 | -2.14 | 11409 | 10833 | -576 | -5.05 | 31 | 362 | +331 | +1066.81 | 99.73 | 96.77 | -2.96 | 86.65 | 85.40 | -1.25 | -1.45 | 99.67 | 96.65 | -3.02 | 86.61 | 85.32 | -1.29 | -1.49 |
| TOON_KEYFOLD | 11289 | 11044 | -245 | -2.17 | 11107 | 10657 | -450 | -4.05 | 182 | 387 | +205 | +112.52 | 98.39 | 96.50 | -1.89 | 86.20 | 85.66 | -0.54 | -0.63 | 98.09 | 96.32 | -1.77 | 86.00 | 85.54 | -0.46 | -0.53 |
| XML_COMPACT | 10164 | 9684 | -480 | -4.72 | 9973 | 9450 | -523 | -5.25 | 191 | 234 | +43 | +22.65 | 98.12 | 97.58 | -0.54 | 89.33 | 90.38 | +1.05 | +1.18 | 98.00 | 97.79 | -0.21 | 89.25 | 90.52 | +1.27 | +1.42 |
| XML_PRETTY | 18291 | 17760 | -531 | -2.90 | 17848 | 17378 | -470 | -2.63 | 443 | 382 | -61 | -13.73 | 97.58 | 97.85 | +0.27 | 65.08 | 66.82 | +1.74 | +2.68 | 97.13 | 97.99 | +0.86 | 64.78 | 66.92 | +2.13 | +3.29 |
| YAML | 14475 | 11404 | -3071 | -21.22 | 14436 | 11006 | -3430 | -23.76 | 39 | 398 | +359 | +920.30 | 99.73 | 96.51 | -3.22 | 77.73 | 84.61 | +6.88 | +8.85 | 99.77 | 96.61 | -3.16 | 77.76 | 84.68 | +6.92 | +8.90 |

#### 2.6.3 Output Tokens: Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 25905 | 23478 | -2427 | -9.37 | 25555 | 22973 | -2582 | -10.10 | 350 | 505 | +155 | +44.30 | 98.65 | 97.85 | -0.80 | 67.61 | 82.67 | +15.06 | +22.27 | 98.61 | 98.12 | -0.49 | 67.58 | 82.85 | +15.27 | +22.59 |
| JSON_PRETTY | 23340 | 23674 | +334 | +1.43 | 22775 | 23101 | +326 | +1.43 | 565 | 573 | +8 | +1.43 | 97.58 | 97.58 | 0.00 | 83.37 | 81.23 | -2.14 | -2.57 | 97.20 | 97.82 | +0.62 | 83.12 | 81.39 | -1.73 | -2.08 |
| TOON_DEFAULT | 21014 | 21911 | +897 | +4.27 | 20957 | 21203 | +246 | +1.17 | 57 | 708 | +651 | +1142.06 | 99.73 | 96.77 | -2.96 | 99.76 | 92.02 | -7.73 | -7.75 | 99.67 | 96.65 | -3.02 | 99.72 | 91.94 | -7.78 | -7.80 |
| TOON_KEYFOLD | 21803 | 24382 | +2579 | +11.83 | 21452 | 23528 | +2076 | +9.68 | 351 | 853 | +502 | +143.11 | 98.39 | 96.50 | -1.89 | 93.79 | 75.97 | -17.83 | -19.01 | 98.09 | 96.32 | -1.77 | 93.59 | 75.85 | -17.75 | -18.96 |
| XML_COMPACT | 22696 | 26182 | +3486 | +15.36 | 22269 | 25548 | +3279 | +14.72 | 427 | 634 | +207 | +48.46 | 98.12 | 97.58 | -0.54 | 87.87 | 65.12 | -22.76 | -25.90 | 98.00 | 97.79 | -0.21 | 87.79 | 65.26 | -22.54 | -25.67 |
| XML_PRETTY | 25690 | 24992 | -698 | -2.72 | 25068 | 24455 | -613 | -2.45 | 622 | 538 | -84 | -13.56 | 97.58 | 97.85 | +0.27 | 68.28 | 72.94 | +4.66 | +6.83 | 97.13 | 97.99 | +0.86 | 67.98 | 73.03 | +5.06 | +7.44 |
| YAML | 23143 | 23033 | -110 | -0.48 | 23080 | 22229 | -851 | -3.69 | 62 | 803 | +741 | +1195.73 | 99.73 | 96.51 | -3.22 | 86.08 | 84.64 | -1.44 | -1.67 | 99.77 | 96.61 | -3.16 | 86.10 | 84.70 | -1.40 | -1.63 |

#### 2.6.4 Total Tokens: Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 33403 | 30448 | -2955 | -8.85 | 32952 | 29794 | -3158 | -9.58 | 451 | 655 | +204 | +45.17 | 98.65 | 97.85 | -0.80 | 91.81 | 98.54 | +6.73 | +7.33 | 98.61 | 98.12 | -0.49 | 91.78 | 98.72 | +6.94 | +7.56 |
| JSON_PRETTY | 39127 | 38532 | -595 | -1.52 | 38180 | 37599 | -581 | -1.52 | 947 | 933 | -14 | -1.52 | 97.58 | 97.58 | 0.00 | 77.02 | 78.48 | +1.46 | +1.90 | 97.20 | 97.82 | +0.62 | 76.76 | 78.64 | +1.88 | +2.45 |
| TOON_DEFAULT | 32454 | 33106 | +652 | +2.01 | 32366 | 32036 | -330 | -1.02 | 88 | 1070 | +982 | +1115.55 | 99.73 | 96.77 | -2.96 | 94.86 | 91.29 | -3.58 | -3.77 | 99.67 | 96.65 | -3.02 | 94.82 | 91.21 | -3.62 | -3.81 |
| TOON_KEYFOLD | 33092 | 35426 | +2334 | +7.05 | 32559 | 34186 | +1627 | +5.00 | 533 | 1240 | +707 | +132.67 | 98.39 | 96.50 | -1.89 | 92.40 | 85.40 | -7.00 | -7.57 | 98.09 | 96.32 | -1.77 | 92.20 | 85.28 | -6.92 | -7.50 |
| XML_COMPACT | 32860 | 35866 | +3006 | +9.15 | 32242 | 34997 | +2755 | +8.55 | 618 | 868 | +250 | +40.48 | 98.12 | 97.58 | -0.54 | 92.79 | 85.04 | -7.75 | -8.35 | 98.00 | 97.79 | -0.21 | 92.71 | 85.18 | -7.53 | -8.12 |
| XML_PRETTY | 43981 | 42752 | -1229 | -2.79 | 42917 | 41833 | -1084 | -2.52 | 1064 | 919 | -145 | -13.64 | 97.58 | 97.85 | +0.27 | 65.08 | 68.28 | +3.20 | +4.92 | 97.13 | 97.99 | +0.86 | 64.78 | 68.37 | +3.59 | +5.55 |
| YAML | 37618 | 34437 | -3181 | -8.46 | 37516 | 33235 | -4281 | -11.41 | 102 | 1202 | +1100 | +1078.70 | 99.73 | 96.51 | -3.22 | 82.16 | 87.84 | +5.68 | +6.91 | 99.77 | 96.61 | -3.16 | 82.19 | 87.90 | +5.72 | +6.95 |

### 2.7 Token Utilization Efficiency (Accuracy by Character)
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy by Character (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy by Character (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 7498 | 7495 | 3 | 25905 | 25894 | 10 | 33403 | 33389 | 13 | 99.96 | 98.39 | 68.48 | 92.68 | 99.66 | 98.19 | 68.28 | 92.48 |
| JSON_COMPACT | opt | 6970 | 6961 | 9 | 23478 | 23447 | 31 | 30448 | 30408 | 40 | 99.87 | 99.88 | 84.02 | 99.89 | 99.25 | 99.47 | 83.60 | 99.47 |
| JSON_PRETTY | man | 15787 | 15669 | 118 | 23340 | 23165 | 175 | 39127 | 38834 | 293 | 99.25 | 73.56 | 84.49 | 78.13 | 99.27 | 73.57 | 84.50 | 78.14 |
| JSON_PRETTY | opt | 14858 | 14840 | 18 | 23674 | 23646 | 28 | 38532 | 38486 | 46 | 99.88 | 76.71 | 82.76 | 80.01 | 99.14 | 76.21 | 82.27 | 79.52 |
| TOON_DEFAULT | man | 11440 | 11440 | 0 | 21014 | 21014 | 0 | 32454 | 32454 | 0 | 100.00 | 86.83 | 99.94 | 95.04 | 99.94 | 86.79 | 99.90 | 95.00 |
| TOON_DEFAULT | opt | 11195 | 11173 | 22 | 21911 | 21867 | 44 | 33106 | 33039 | 66 | 99.80 | 87.42 | 94.04 | 93.31 | 99.26 | 87.06 | 93.68 | 92.95 |
| TOON_KEYFOLD | man | 11289 | 11251 | 38 | 21803 | 21729 | 74 | 33092 | 32979 | 113 | 99.66 | 87.05 | 94.64 | 93.25 | 99.58 | 87.00 | 94.59 | 93.19 |
| TOON_KEYFOLD | opt | 11044 | 11001 | 43 | 24381 | 24286 | 95 | 35425 | 35287 | 138 | 99.61 | 87.74 | 78.04 | 87.47 | 99.10 | 87.40 | 77.70 | 87.13 |
| XML_COMPACT | man | 10164 | 10131 | 33 | 22696 | 22623 | 73 | 32860 | 32755 | 105 | 99.68 | 90.37 | 88.91 | 93.83 | 99.47 | 90.23 | 88.77 | 93.69 |
| XML_COMPACT | opt | 9684 | 9666 | 18 | 26182 | 26132 | 50 | 35866 | 35798 | 68 | 99.81 | 91.87 | 66.60 | 86.52 | 99.16 | 91.43 | 66.17 | 86.09 |
| XML_PRETTY | man | 18291 | 18234 | 57 | 25690 | 25610 | 80 | 43981 | 43845 | 136 | 99.69 | 66.49 | 69.68 | 66.48 | 99.25 | 66.19 | 69.39 | 66.19 |
| XML_PRETTY | opt | 17760 | 17726 | 34 | 24992 | 24945 | 47 | 42752 | 42671 | 81 | 99.81 | 68.13 | 74.25 | 69.59 | 99.41 | 67.86 | 73.98 | 69.32 |
| YAML | man | 14475 | 14469 | 6 | 23143 | 23133 | 9 | 37618 | 37603 | 15 | 99.96 | 77.89 | 86.23 | 82.31 | 99.96 | 77.89 | 86.23 | 82.31 |
| YAML | opt | 11404 | 11356 | 48 | 23033 | 22936 | 97 | 34437 | 34292 | 145 | 99.58 | 86.66 | 86.68 | 89.88 | 99.00 | 86.27 | 86.30 | 89.50 |

#### 2.7.2 Read Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7498 | 6970 | -528 | -7.04 | 7495 | 6961 | -534 | -7.13 | 3 | 9 | +6 | +202.07 | 99.96 | 99.87 | -0.09 | 98.39 | 99.88 | +1.49 | +1.52 | 99.66 | 99.25 | -0.41 | 98.19 | 99.47 | +1.28 | +1.30 |
| JSON_PRETTY | 15787 | 14858 | -929 | -5.88 | 15669 | 14841 | -828 | -5.29 | 118 | 17 | -101 | -85.23 | 99.25 | 99.88 | +0.63 | 73.56 | 76.71 | +3.15 | +4.28 | 99.27 | 99.14 | -0.13 | 73.57 | 76.21 | +2.64 | +3.59 |
| TOON_DEFAULT | 11440 | 11195 | -245 | -2.14 | 11440 | 11173 | -267 | -2.34 | 0 | 22 | +22 | 0.00 | 100.00 | 99.80 | -0.20 | 86.83 | 87.42 | +0.59 | +0.67 | 99.94 | 99.26 | -0.68 | 86.79 | 87.06 | +0.27 | +0.31 |
| TOON_KEYFOLD | 11289 | 11044 | -245 | -2.17 | 11251 | 11001 | -250 | -2.22 | 38 | 43 | +5 | +12.34 | 99.66 | 99.61 | -0.05 | 87.05 | 87.74 | +0.69 | +0.79 | 99.58 | 99.10 | -0.48 | 87.00 | 87.40 | +0.40 | +0.46 |
| XML_COMPACT | 10164 | 9684 | -480 | -4.72 | 10131 | 9665 | -466 | -4.60 | 33 | 19 | -14 | -42.80 | 99.68 | 99.81 | +0.13 | 90.37 | 91.87 | +1.50 | +1.66 | 99.47 | 99.16 | -0.31 | 90.23 | 91.43 | +1.20 | +1.33 |
| XML_PRETTY | 18291 | 17760 | -531 | -2.90 | 18234 | 17726 | -508 | -2.79 | 57 | 34 | -23 | -40.28 | 99.69 | 99.81 | +0.12 | 66.49 | 68.13 | +1.64 | +2.47 | 99.25 | 99.41 | +0.16 | 66.19 | 67.86 | +1.67 | +2.52 |
| YAML | 14475 | 11404 | -3071 | -21.22 | 14469 | 11356 | -3113 | -21.52 | 6 | 48 | +42 | +701.78 | 99.96 | 99.58 | -0.38 | 77.89 | 86.66 | +8.77 | +11.26 | 99.96 | 99.00 | -0.96 | 77.89 | 86.27 | +8.39 | +10.77 |

#### 2.7.3 Output Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 25905 | 23478 | -2427 | -9.37 | 25894 | 23447 | -2447 | -9.45 | 10 | 30 | +20 | +201.59 | 99.96 | 99.87 | -0.09 | 68.48 | 84.02 | +15.53 | +22.68 | 99.66 | 99.25 | -0.41 | 68.28 | 83.60 | +15.32 | +22.43 |
| JSON_PRETTY | 23340 | 23674 | +334 | +1.43 | 23165 | 23645 | +480 | +2.07 | 175 | 28 | -147 | -83.80 | 99.25 | 99.88 | +0.63 | 84.49 | 82.76 | -1.72 | -2.04 | 99.27 | 99.14 | -0.13 | 84.50 | 82.27 | -2.23 | -2.64 |
| TOON_DEFAULT | 21014 | 21911 | +897 | +4.27 | 21014 | 21867 | +853 | +4.06 | 0 | 44 | +44 | 0.00 | 100.00 | 99.80 | -0.20 | 99.94 | 94.04 | -5.89 | -5.90 | 99.94 | 99.26 | -0.68 | 99.90 | 93.68 | -6.21 | -6.22 |
| TOON_KEYFOLD | 21803 | 24382 | +2579 | +11.83 | 21729 | 24287 | +2558 | +11.77 | 74 | 95 | +21 | +28.32 | 99.66 | 99.61 | -0.05 | 94.64 | 78.04 | -16.60 | -17.54 | 99.58 | 99.10 | -0.48 | 94.59 | 77.70 | -16.89 | -17.86 |
| XML_COMPACT | 22696 | 26182 | +3486 | +15.36 | 22623 | 26132 | +3509 | +15.51 | 73 | 50 | -23 | -31.35 | 99.68 | 99.81 | +0.13 | 88.91 | 66.60 | -22.31 | -25.09 | 99.47 | 99.16 | -0.31 | 88.77 | 66.17 | -22.60 | -25.46 |
| XML_PRETTY | 25690 | 24992 | -698 | -2.72 | 25610 | 24944 | -666 | -2.60 | 80 | 48 | -32 | -40.19 | 99.69 | 99.81 | +0.12 | 69.68 | 74.25 | +4.56 | +6.55 | 99.25 | 99.41 | +0.16 | 69.39 | 73.98 | +4.59 | +6.61 |
| YAML | 23143 | 23033 | -110 | -0.48 | 23133 | 22936 | -197 | -0.85 | 9 | 96 | +87 | +972.00 | 99.96 | 99.58 | -0.38 | 86.23 | 86.68 | +0.45 | +0.53 | 99.96 | 99.00 | -0.96 | 86.23 | 86.30 | +0.07 | +0.08 |

#### 2.7.4 Total Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 33403 | 30448 | -2955 | -8.85 | 33389 | 30408 | -2981 | -8.93 | 13 | 39 | +26 | +201.70 | 99.96 | 99.87 | -0.09 | 92.68 | 99.89 | +7.21 | +7.78 | 99.66 | 99.25 | -0.41 | 92.48 | 99.47 | +6.99 | +7.56 |
| JSON_PRETTY | 39127 | 38532 | -595 | -1.52 | 38834 | 38486 | -348 | -0.90 | 293 | 46 | -247 | -84.37 | 99.25 | 99.88 | +0.63 | 78.13 | 80.01 | +1.88 | +2.41 | 99.27 | 99.14 | -0.13 | 78.14 | 79.52 | +1.38 | +1.76 |
| TOON_DEFAULT | 32454 | 33106 | +652 | +2.01 | 32454 | 33039 | +585 | +1.80 | 0 | 66 | +66 | 0.00 | 100.00 | 99.80 | -0.20 | 95.04 | 93.31 | -1.74 | -1.83 | 99.94 | 99.26 | -0.68 | 95.00 | 92.95 | -2.06 | -2.16 |
| TOON_KEYFOLD | 33092 | 35426 | +2334 | +7.05 | 32979 | 35287 | +2308 | +7.00 | 113 | 139 | +26 | +22.70 | 99.66 | 99.61 | -0.05 | 93.25 | 87.47 | -5.77 | -6.19 | 99.58 | 99.10 | -0.48 | 93.19 | 87.13 | -6.06 | -6.50 |
| XML_COMPACT | 32860 | 35866 | +3006 | +9.15 | 32755 | 35798 | +3043 | +9.29 | 105 | 68 | -37 | -35.24 | 99.68 | 99.81 | +0.13 | 93.83 | 86.52 | -7.31 | -7.79 | 99.47 | 99.16 | -0.31 | 93.69 | 86.09 | -7.60 | -8.11 |
| XML_PRETTY | 43981 | 42752 | -1229 | -2.79 | 43845 | 42671 | -1174 | -2.68 | 136 | 81 | -55 | -40.52 | 99.69 | 99.81 | +0.12 | 66.48 | 69.59 | +3.10 | +4.67 | 99.25 | 99.41 | +0.16 | 66.19 | 69.32 | +3.13 | +4.73 |
| YAML | 37618 | 34437 | -3181 | -8.46 | 37603 | 34292 | -3311 | -8.80 | 15 | 145 | +130 | +863.91 | 99.96 | 99.58 | -0.38 | 82.31 | 89.88 | +7.57 | +9.20 | 99.96 | 99.00 | -0.96 | 82.31 | 89.50 | +7.18 | +8.73 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Expected Characters | Output Characters | Correct Characters | Incorrect Characters | Accuracy by Character (%) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 122 | 2 | 0 | 98.65 | 7782 | 7782 | 7779 | 3 | 99.96 |
| JSON_COMPACT | opt | 121 | 3 | 0 | 97.85 | 8451 | 8453 | 8442 | 11 | 99.87 |
| JSON_PRETTY | man | 121 | 3 | 0 | 97.58 | 7782 | 7838 | 7778 | 60 | 99.25 |
| JSON_PRETTY | opt | 121 | 3 | 0 | 97.58 | 8451 | 8452 | 8442 | 10 | 99.88 |
| TOON_DEFAULT | man | 124 | 0 | 0 | 99.73 | 7782 | 7782 | 7782 | 0 | 100.00 |
| TOON_DEFAULT | opt | 120 | 4 | 0 | 96.77 | 8451 | 8460 | 8443 | 17 | 99.80 |
| TOON_KEYFOLD | man | 122 | 2 | 0 | 98.39 | 7782 | 7799 | 7773 | 27 | 99.66 |
| TOON_KEYFOLD | opt | 120 | 4 | 0 | 96.50 | 8451 | 8476 | 8443 | 33 | 99.61 |
| XML_COMPACT | man | 122 | 2 | 0 | 98.12 | 7782 | 7792 | 7767 | 25 | 99.68 |
| XML_COMPACT | opt | 121 | 3 | 0 | 97.58 | 8451 | 8459 | 8442 | 16 | 99.81 |
| XML_PRETTY | man | 121 | 3 | 0 | 97.58 | 7782 | 7790 | 7766 | 24 | 99.69 |
| XML_PRETTY | opt | 121 | 3 | 0 | 97.85 | 8451 | 8461 | 8445 | 16 | 99.81 |
| YAML | man | 124 | 0 | 0 | 99.73 | 7782 | 7784 | 7780 | 3 | 99.96 |
| YAML | opt | 120 | 4 | 0 | 96.51 | 8451 | 8477 | 8441 | 36 | 99.58 |

#### 2.8.2 Answers: Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 122 | 121 | -1 | -0.82 | 2 | 3 | +1 | +50.00 | 0 | 0 | 0 | 0.00 | 98.65 | 97.85 | -0.80 |
| JSON_PRETTY | 121 | 121 | 0 | 0.00 | 3 | 3 | 0 | 0.00 | 0 | 0 | 0 | 0.00 | 97.58 | 97.58 | 0.00 |
| TOON_DEFAULT | 124 | 120 | -4 | -2.96 | 0 | 4 | +4 | 0.00 | 0 | 0 | 0 | 0.00 | 99.73 | 96.77 | -2.96 |
| TOON_KEYFOLD | 122 | 120 | -2 | -1.91 | 2 | 4 | +2 | +116.50 | 0 | 0 | 0 | 0.00 | 98.39 | 96.50 | -1.89 |
| XML_COMPACT | 122 | 121 | -1 | -0.55 | 2 | 3 | +1 | +33.50 | 0 | 0 | 0 | 0.00 | 98.12 | 97.58 | -0.54 |
| XML_PRETTY | 121 | 121 | 0 | +0.27 | 3 | 3 | 0 | -11.00 | 0 | 0 | 0 | 0.00 | 97.58 | 97.85 | +0.27 |
| YAML | 124 | 120 | -4 | -3.23 | 0 | 4 | +4 | 0.00 | 0 | 0 | 0 | 0.00 | 99.73 | 96.51 | -3.22 |

#### 2.8.3 Characters: Mandatory vs Optional Data
| Format | Output Characters Man | Output Characters Opt | Diff | Diff (%) | Correct Characters Man | Correct Characters Opt | Diff | Diff (%) | Incorrect Characters Man | Incorrect Characters Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7782 | 8453 | +671 | +8.62 | 7779 | 8442 | +663 | +8.53 | 3 | 11 | +8 | +255.57 | 99.96 | 99.87 | -0.09 |
| JSON_PRETTY | 7838 | 8452 | +614 | +7.83 | 7778 | 8442 | +664 | +8.53 | 60 | 10 | -50 | -82.78 | 99.25 | 99.88 | 0.63 |
| TOON_DEFAULT | 7782 | 8460 | +678 | +8.71 | 7782 | 8443 | +661 | +8.50 | 0 | 17 | +17 | 0.00 | 100.00 | 99.80 | -0.20 |
| TOON_KEYFOLD | 7799 | 8476 | +677 | +8.68 | 7773 | 8443 | +670 | +8.62 | 27 | 34 | +7 | +24.69 | 99.66 | 99.61 | -0.05 |
| XML_COMPACT | 7792 | 8459 | +667 | +8.56 | 7767 | 8442 | +675 | +8.69 | 25 | 16 | -9 | -34.67 | 99.68 | 99.81 | 0.13 |
| XML_PRETTY | 7790 | 8461 | +671 | +8.62 | 7766 | 8445 | +679 | +8.75 | 24 | 16 | -8 | -33.33 | 99.69 | 99.81 | 0.12 |
| YAML | 7784 | 8477 | +693 | +8.90 | 7780 | 8441 | +661 | +8.49 | 3 | 35 | +32 | +1077.80 | 99.96 | 99.58 | -0.38 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 98.65 | 100.00 | 100.00 | 95.24 | 96.83 | 99.66 | 37.50 | 29.17 | 19.84 | 12.10 |
| JSON_COMPACT | opt | 97.85 | 100.00 | 98.77 | 98.41 | 90.48 | 99.25 | 37.50 | 28.81 | 20.50 | 11.31 |
| JSON_PRETTY | man | 97.58 | 100.00 | 96.30 | 93.65 | 96.83 | 99.27 | 37.50 | 28.09 | 19.51 | 12.10 |
| JSON_PRETTY | opt | 97.58 | 100.00 | 100.00 | 95.24 | 90.48 | 99.14 | 37.50 | 29.17 | 19.84 | 11.31 |
| TOON_DEFAULT | man | 99.73 | 100.00 | 100.00 | 98.41 | 100.00 | 99.94 | 37.50 | 29.17 | 20.50 | 12.50 |
| TOON_DEFAULT | opt | 96.77 | 100.00 | 92.59 | 100.00 | 90.48 | 99.26 | 37.50 | 27.01 | 20.83 | 11.31 |
| TOON_KEYFOLD | man | 98.39 | 100.00 | 97.53 | 95.24 | 98.41 | 99.58 | 37.50 | 28.45 | 19.84 | 12.30 |
| TOON_KEYFOLD | opt | 96.50 | 100.00 | 92.59 | 98.41 | 90.48 | 99.10 | 37.50 | 27.01 | 20.50 | 11.31 |
| XML_COMPACT | man | 98.12 | 98.79 | 98.77 | 95.24 | 98.41 | 99.47 | 37.05 | 28.81 | 19.84 | 12.30 |
| XML_COMPACT | opt | 97.58 | 100.00 | 98.77 | 96.83 | 90.48 | 99.16 | 37.50 | 28.81 | 20.17 | 11.31 |
| XML_PRETTY | man | 97.58 | 99.39 | 100.00 | 87.30 | 100.00 | 99.25 | 37.27 | 29.17 | 18.19 | 12.50 |
| XML_PRETTY | opt | 97.85 | 100.00 | 98.77 | 96.83 | 92.07 | 99.41 | 37.50 | 28.81 | 20.17 | 11.51 |
| YAML | man | 99.73 | 99.39 | 100.00 | 100.00 | 100.00 | 99.96 | 37.27 | 29.17 | 20.83 | 12.50 |
| YAML | opt | 96.51 | 100.00 | 93.83 | 100.00 | 87.30 | 99.00 | 37.50 | 27.37 | 20.83 | 10.91 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| JSON_PRETTY | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| TOON_DEFAULT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| TOON_KEYFOLD | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| XML_COMPACT | 98.79 | 100.00 | +1.21 | 37.05 | 37.50 | +0.45 |
| XML_PRETTY | 99.39 | 100.00 | +0.61 | 37.27 | 37.50 | +0.23 |
| YAML | 99.39 | 100.00 | +0.61 | 37.27 | 37.50 | +0.23 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 100.00 | 98.77 | -1.23 | 29.17 | 28.81 | -0.36 |
| JSON_PRETTY | 96.30 | 100.00 | +3.70 | 28.09 | 29.17 | +1.08 |
| TOON_DEFAULT | 100.00 | 92.59 | -7.41 | 29.17 | 27.01 | -2.16 |
| TOON_KEYFOLD | 97.53 | 92.59 | -4.94 | 28.45 | 27.01 | -1.44 |
| XML_COMPACT | 98.77 | 98.77 | 0.00 | 28.81 | 28.81 | 0.00 |
| XML_PRETTY | 100.00 | 98.77 | -1.23 | 29.17 | 28.81 | -0.36 |
| YAML | 100.00 | 93.83 | -6.17 | 29.17 | 27.37 | -1.80 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 95.24 | 98.41 | +3.17 | 19.84 | 20.50 | +0.66 |
| JSON_PRETTY | 93.65 | 95.24 | +1.58 | 19.51 | 19.84 | +0.33 |
| TOON_DEFAULT | 98.41 | 100.00 | +1.59 | 20.50 | 20.83 | +0.33 |
| TOON_KEYFOLD | 95.24 | 98.41 | +3.17 | 19.84 | 20.50 | +0.66 |
| XML_COMPACT | 95.24 | 96.83 | +1.59 | 19.84 | 20.17 | +0.33 |
| XML_PRETTY | 87.30 | 96.83 | +9.53 | 18.19 | 20.17 | +1.98 |
| YAML | 100.00 | 100.00 | 0.00 | 20.83 | 20.83 | 0.00 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 96.83 | 90.48 | -6.35 | 12.10 | 11.31 | -0.79 |
| JSON_PRETTY | 96.83 | 90.48 | -6.35 | 12.10 | 11.31 | -0.79 |
| TOON_DEFAULT | 100.00 | 90.48 | -9.52 | 12.50 | 11.31 | -1.19 |
| TOON_KEYFOLD | 98.41 | 90.48 | -7.93 | 12.30 | 11.31 | -0.99 |
| XML_COMPACT | 98.41 | 90.48 | -7.93 | 12.30 | 11.31 | -0.99 |
| XML_PRETTY | 100.00 | 92.07 | -7.93 | 12.50 | 11.51 | -0.99 |
| YAML | 100.00 | 87.30 | -12.70 | 12.50 | 10.91 | -1.59 |

### 2.10 Accuracy By Character Per Question Category Analysis
#### 2.10.1 Metrics
| Format | Variant | Accuracy By Character (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc By Char (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 99.96 | 100.00 | 100.00 | 99.11 | 98.77 | 99.66 | 37.50 | 29.17 | 20.65 | 12.35 |
| JSON_COMPACT | opt | 99.87 | 100.00 | 99.97 | 99.71 | 94.55 | 99.25 | 37.50 | 29.16 | 20.77 | 11.82 |
| JSON_PRETTY | man | 99.25 | 100.00 | 98.77 | 98.81 | 98.97 | 99.27 | 37.50 | 28.81 | 20.59 | 12.37 |
| JSON_PRETTY | opt | 99.88 | 100.00 | 100.00 | 99.12 | 94.55 | 99.14 | 37.50 | 29.17 | 20.65 | 11.82 |
| TOON_DEFAULT | man | 100.00 | 100.00 | 100.00 | 99.70 | 100.00 | 99.94 | 37.50 | 29.17 | 20.77 | 12.50 |
| TOON_DEFAULT | opt | 99.80 | 100.00 | 99.82 | 100.00 | 94.55 | 99.26 | 37.50 | 29.11 | 20.83 | 11.82 |
| TOON_KEYFOLD | man | 99.66 | 100.00 | 99.45 | 99.11 | 99.38 | 99.58 | 37.50 | 29.01 | 20.65 | 12.42 |
| TOON_KEYFOLD | opt | 99.61 | 100.00 | 99.46 | 99.71 | 94.55 | 99.10 | 37.50 | 29.01 | 20.77 | 11.82 |
| XML_COMPACT | man | 99.68 | 99.34 | 99.94 | 99.11 | 99.38 | 99.47 | 37.25 | 29.15 | 20.64 | 12.42 |
| XML_COMPACT | opt | 99.81 | 100.00 | 99.85 | 99.41 | 94.55 | 99.16 | 37.50 | 29.13 | 20.71 | 11.82 |
| XML_PRETTY | man | 99.69 | 99.31 | 100.00 | 97.62 | 100.00 | 99.25 | 37.24 | 29.17 | 20.34 | 12.50 |
| XML_PRETTY | opt | 99.81 | 100.00 | 99.78 | 99.41 | 96.76 | 99.41 | 37.50 | 29.10 | 20.71 | 12.10 |
| YAML | man | 99.96 | 99.89 | 100.00 | 100.00 | 100.00 | 99.96 | 37.46 | 29.17 | 20.83 | 12.50 |
| YAML | opt | 99.58 | 100.00 | 99.45 | 100.00 | 93.34 | 99.00 | 37.50 | 29.01 | 20.83 | 11.67 |

#### 2.10.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| JSON_PRETTY | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| TOON_DEFAULT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| TOON_KEYFOLD | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| XML_COMPACT | 99.34 | 100.00 | +0.66 | 37.25 | 37.50 | +0.25 |
| XML_PRETTY | 99.31 | 100.00 | +0.69 | 37.24 | 37.50 | +0.26 |
| YAML | 99.89 | 100.00 | +0.11 | 37.46 | 37.50 | +0.04 |

#### 2.10.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 100.00 | 99.97 | -0.03 | 29.17 | 29.16 | -0.01 |
| JSON_PRETTY | 98.77 | 100.00 | +1.23 | 28.81 | 29.17 | +0.36 |
| TOON_DEFAULT | 100.00 | 99.82 | -0.18 | 29.17 | 29.11 | -0.06 |
| TOON_KEYFOLD | 99.45 | 99.46 | +0.01 | 29.01 | 29.01 | 0.00 |
| XML_COMPACT | 99.94 | 99.85 | -0.09 | 29.15 | 29.13 | -0.02 |
| XML_PRETTY | 100.00 | 99.78 | -0.22 | 29.17 | 29.10 | -0.07 |
| YAML | 100.00 | 99.45 | -0.55 | 29.17 | 29.01 | -0.16 |

#### 2.10.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 99.11 | 99.71 | +0.60 | 20.65 | 20.77 | +0.12 |
| JSON_PRETTY | 98.81 | 99.12 | +0.31 | 20.59 | 20.65 | +0.06 |
| TOON_DEFAULT | 99.70 | 100.00 | +0.30 | 20.77 | 20.83 | +0.06 |
| TOON_KEYFOLD | 99.11 | 99.71 | +0.60 | 20.65 | 20.77 | +0.12 |
| XML_COMPACT | 99.11 | 99.41 | +0.31 | 20.64 | 20.71 | +0.07 |
| XML_PRETTY | 97.62 | 99.41 | +1.79 | 20.34 | 20.71 | +0.37 |
| YAML | 100.00 | 100.00 | 0.00 | 20.83 | 20.83 | 0.00 |

#### 2.10.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 98.77 | 94.55 | -4.22 | 12.35 | 11.82 | -0.53 |
| JSON_PRETTY | 98.97 | 94.55 | -4.42 | 12.37 | 11.82 | -0.55 |
| TOON_DEFAULT | 100.00 | 94.55 | -5.45 | 12.50 | 11.82 | -0.68 |
| TOON_KEYFOLD | 99.38 | 94.55 | -4.83 | 12.42 | 11.82 | -0.60 |
| XML_COMPACT | 99.38 | 94.55 | -4.83 | 12.42 | 11.82 | -0.60 |
| XML_PRETTY | 100.00 | 96.76 | -3.24 | 12.50 | 12.10 | -0.40 |
| YAML | 100.00 | 93.34 | -6.66 | 12.50 | 11.67 | -0.83 |

## 3. Appendices

### 3.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Sonnet 4.6
- **Thinking**: on (medium)
- **Structure**: nested
- **Formats Tested**: JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, TOON_KEYFOLD, XML_COMPACT, XML_PRETTY, YAML
- **Record Counts**: 31
- **Total Test Cases**: 14

### 3.2 Appendix B: Benchmark Configuration
- **Field Retrieval**: 55 questions (37.50% weight)
- **Filtering**: 21 questions (20.83% weight)
- **Aggregation**: 21 questions (12.50% weight)
- **Structure Awareness**: 27 questions (29.17% weight)

---

- **Report Generated**: 2026-04-20
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code 2.1.80
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_sonnet_4_6/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_sonnet_4_6/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_sonnet_4_6/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)