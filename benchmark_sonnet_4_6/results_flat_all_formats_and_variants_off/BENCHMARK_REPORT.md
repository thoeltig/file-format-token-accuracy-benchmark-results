# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Sonnet 4.6
- **Thinking**: off
- **Data Structure**: flat
- **Formats Tested**: 7 (CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: First iteration

## Executive Summary

This benchmark evaluates token efficiency and information accuracy across 7 file formats using Sonnet 4.6 as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context while a format that accurately conveys information may justify higher token cost.

### Key Findings

1. No format produced catastrophically wrong answers and accuracy by character stayed above 99.69% for every format and variant combination. Complete answer accuracy ranged from 96.24% (**XML_COMPACT** optional) to 99.46% (**JSON_PRETTY** mandatory). This means most errors were small character level mistakes rather than fully wrong answers so format choice affects token cost far more than answer correctness at this scale.
2. **TOON_DEFAULT** wins total token efficiency on dense mandatory data. It produced the lowest total token count (31606) and the highest weighted efficiency score for total tokens used (87.24). **TOON** uses adaptive encoding that shifts between a **CSV** like tabular layout for dense records and a **YAML** like key value layout for sparse records. On the optional variant this shift triggers and read tokens grow by 24.85% (from 6798 to 8487) which lands it close to **YAML** (8696). The tabular advantage is therefore specific to dense data and the format accepts a higher cost on sparse data in exchange for unambiguous per record representation.
3. The read tokens of **JSON_COMPACT** dropped by 37.20% from mandatory to optional (9027 to 5669) because absent fields can be omitted from each object while the structure stays parseable. The resulting total efficiency score of 98.71 is the highest value measured across any format or variant in this benchmark.
4. **XML_PRETTY** mandatory consumed 13087 read tokens which is 93.94% more than **CSV** while delivering the lowest mandatory accuracy at 97.04%. **XML_COMPACT** optional produced the lowest overall accuracy (96.24%) and the highest count of wasted output tokens (1002).
5. Pretty-printed formats like **JSON_PRETTY** (+67%) and **XML_PRETTY** (+94% to +97%) consume a more read tokens than **CSV** while their accuracy by character only differs by 0.08% to 0.10%. Their is no real benefit in using pretty-printed formats in any agentic workflow because they requiere more tokens and fill up the context faster. If users choose to use pretty-printed formats they pay the extra cost just for their own ability to read the data easier.
6. Aggregation is the weakest question category and degrades the most under sparse data. Every format lost accuracy on aggregation when moving from mandatory to optional with drops ranging from 3.17 percentage points (**CSV**) to 11.11 percentage points (**XML_COMPACT**). Field retrieval by contrast held at or near 100% for every format and both variants.
7. **CSV**'s complete answer accuracy changed by only 0.28 percentage points between mandatory and optional, its read tokens varied by 3.82% and it produced the lowest count of wasted character tokens on optional data (3). The cost is output efficiency where **CSV** ranks last on mandatory data (output efficiency score 66.00) because the model spends more output tokens reasoning about the tabular layout during answering.

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
| JSON_PRETTY ≈ 273s | CSV ≈ 6748 | YAML ≈ 198 | JSON_PRETTY ≈ 20394 | JSON_PRETTY ≈ 20597 | TOON_DEFAULT ≈ 31606 | JSON_PRETTY ≈ 99.46% | CSV ≈ 94 | JSON_PRETTY ≈ 100 | TOON_DEFAULT ≈ 87 | JSON_PRETTY ≈ 99.98% | CSV ≈ 95 | JSON_PRETTY ≈ 100 | TOON_DEFAULT ≈ 88 |
| JSON_COMPACT (+6.31%) | TOON_DEFAULT (+0.74%) | JSON_PRETTY (+2.53%) | JSON_COMPACT (+11.43%) | JSON_COMPACT (+11.79%) | JSON_PRETTY (+0.62%) | XML_COMPACT (-0.27%) | TOON_DEFAULT (-0.24%) | JSON_COMPACT (-11.52%) | JSON_PRETTY (-0.39%) | TOON_DEFAULT (-0.01%) | TOON_DEFAULT (-0.17%) | XML_PRETTY (-11.07%) | JSON_PRETTY (-0.79%) |
| XML_PRETTY (+10.35%) | JSON_COMPACT (+33.77%) | CSV (+46.54%) | XML_PRETTY (+11.53%) | XML_PRETTY (+11.87%) | JSON_COMPACT (+1.42%) | CSV (-0.53%) | JSON_COMPACT (-11.01%) | XML_PRETTY (-12.67%) | JSON_COMPACT (-2.04%) | XML_COMPACT (-0.01%) | JSON_COMPACT (-10.88%) | JSON_COMPACT (-11.14%) | JSON_COMPACT (-2.04%) |
| YAML (+16.56%) | YAML (+40.47%) | XML_PRETTY (+49.58%) | YAML (+15.60%) | YAML (+15.42%) | YAML (+5.21%) | TOON_DEFAULT (-0.54%) | YAML (-13.15%) | YAML (-14.90%) | YAML (-6.98%) | XML_PRETTY (-0.08%) | YAML (-13.00%) | YAML (-14.50%) | YAML (-6.93%) |
| XML_COMPACT (+21.98%) | JSON_PRETTY (+66.03%) | XML_COMPACT (+50.76%) | TOON_DEFAULT (+20.16%) | TOON_DEFAULT (+20.44%) | CSV (+9.87%) | JSON_COMPACT (-0.80%) | JSON_PRETTY (-20.78%) | TOON_DEFAULT (-19.40%) | CSV (-12.84%) | CSV (-0.10%) | JSON_PRETTY (-20.94%) | TOON_DEFAULT (-18.98%) | CSV (-12.81%) |
| TOON_DEFAULT (+22.54%) | XML_COMPACT (+69.59%) | JSON_COMPACT (+51.77%) | XML_COMPACT (+26.41%) | XML_COMPACT (+26.62%) | XML_PRETTY (+14.31%) | YAML (-0.80%) | XML_COMPACT (-22.11%) | XML_COMPACT (-24.97%) | XML_PRETTY (-20.06%) | YAML (-0.28%) | XML_COMPACT (-22.08%) | XML_COMPACT (-24.71%) | XML_PRETTY (-18.53%) |
| CSV (+35.33%) | XML_PRETTY (+93.94%) | TOON_DEFAULT (+52.44%) | CSV (+35.76%) | CSV (+35.84%) | XML_COMPACT (+18.72%) | XML_PRETTY (-2.42%) | XML_PRETTY (-31.43%) | CSV (-33.73%) | XML_COMPACT (-24.16%) | JSON_COMPACT (-0.29%) | XML_PRETTY (-29.88%) | CSV (-33.33%) | XML_COMPACT (-24.17%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XML_PRETTY ≈ 281s | JSON_COMPACT ≈ 5669 | YAML ≈ 198 | XML_PRETTY ≈ 21922 | XML_PRETTY ≈ 22126 | JSON_COMPACT ≈ 28263 | CSV ≈ 98.65% | JSON_COMPACT ≈ 99 | XML_PRETTY ≈ 92 | JSON_COMPACT ≈ 99 | CSV ≈ 99.95% | JSON_COMPACT ≈ 100 | XML_PRETTY ≈ 93 | JSON_COMPACT ≈ 100 |
| JSON_PRETTY (+1.52%) | CSV (+14.48%) | XML_PRETTY (+3.20%) | JSON_PRETTY (+0.29%) | JSON_PRETTY (+0.72%) | JSON_PRETTY (+15.25%) | JSON_COMPACT (-0.53%) | CSV (-3.37%) | JSON_PRETTY (-0.98%) | JSON_PRETTY (-15.87%) | JSON_COMPACT (-0.06%) | CSV (-3.64%) | JSON_PRETTY (-0.78%) | JSON_PRETTY (-15.51%) |
| JSON_COMPACT (+3.86%) | XML_COMPACT (+38.49%) | CSV (+43.84%) | JSON_COMPACT (+1.70%) | JSON_COMPACT (+2.12%) | CSV (+16.71%) | XML_PRETTY (-0.53%) | XML_COMPACT (-11.18%) | JSON_COMPACT (-2.30%) | CSV (-16.83%) | XML_PRETTY (-0.07%) | XML_COMPACT (-9.87%) | JSON_COMPACT (-2.26%) | CSV (-16.94%) |
| TOON_DEFAULT (+10.97%) | TOON_DEFAULT (+49.71%) | XML_COMPACT (+49.75%) | YAML (+12.38%) | YAML (+12.24%) | YAML (+18.64%) | JSON_PRETTY (-0.80%) | TOON_DEFAULT (-12.98%) | YAML (-14.07%) | YAML (-19.90%) | JSON_PRETTY (-0.08%) | TOON_DEFAULT (-12.72%) | YAML (-13.21%) | YAML (-19.04%) |
| YAML (+16.34%) | YAML (+53.40%) | JSON_PRETTY (+51.43%) | TOON_DEFAULT (+14.67%) | TOON_DEFAULT (+14.97%) | TOON_DEFAULT (+20.04%) | TOON_DEFAULT (-0.80%) | YAML (-14.47%) | TOON_DEFAULT (-16.45%) | TOON_DEFAULT (-20.79%) | TOON_DEFAULT (-0.18%) | YAML (-13.68%) | TOON_DEFAULT (-16.13%) | TOON_DEFAULT (-20.44%) |
| CSV (+18.51%) | JSON_PRETTY (+81.48%) | JSON_COMPACT (+51.60%) | CSV (+19.56%) | CSV (+19.75%) | XML_PRETTY (+20.81%) | YAML (-1.61%) | JSON_PRETTY (-21.15%) | CSV (-21.05%) | XML_PRETTY (-21.40%) | XML_COMPACT (-0.18%) | JSON_PRETTY (-20.74%) | CSV (-21.11%) | XML_PRETTY (-21.15%) |
| XML_COMPACT (+18.89%) | XML_PRETTY (+112.00%) | TOON_DEFAULT (+52.11%) | XML_COMPACT (+20.18%) | XML_COMPACT (+20.41%) | XML_COMPACT (+22.04%) | XML_COMPACT (-2.41%) | XML_PRETTY (-28.83%) | XML_COMPACT (-23.52%) | XML_COMPACT (-23.94%) | YAML (-0.20%) | XML_PRETTY (-28.49%) | XML_COMPACT (-21.95%) | XML_COMPACT (-22.48%) |


#### 2.1.2 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| CSV ≈ 100.00% | JSON_PRETTY ≈ 100.00% | JSON_COMPACT ≈ 100.00% | JSON_PRETTY ≈ 98.41% |
| JSON_PRETTY (0.00%) | TOON_DEFAULT (0.00%) | CSV (-1.59%) | XML_COMPACT (0.00%) |
| TOON_DEFAULT (0.00%) | YAML (0.00%) | JSON_PRETTY (-1.59%) | YAML (0.00%) |
| XML_COMPACT (0.00%) | CSV (-1.23%) | XML_COMPACT (-1.59%) | CSV (-1.59%) |
| XML_PRETTY (0.00%) | XML_COMPACT (-1.23%) | TOON_DEFAULT (-3.17%) | JSON_COMPACT (-1.59%) |
| JSON_COMPACT (-0.61%) | JSON_COMPACT (-2.47%) | XML_PRETTY (-3.17%) | TOON_DEFAULT (-1.59%) |
| YAML (-0.61%) | XML_PRETTY (-8.64%) | YAML (-4.76%) | XML_PRETTY (-1.59%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| CSV ≈ 100.00% | CSV ≈ 100.00% | JSON_PRETTY ≈ 100.00% | CSV ≈ 93.65% |
| JSON_COMPACT (0.00%) | JSON_COMPACT (0.00%) | TOON_DEFAULT (0.00%) | JSON_COMPACT (-3.17%) |
| JSON_PRETTY (0.00%) | XML_PRETTY (0.00%) | YAML (0.00%) | JSON_PRETTY (-3.17%) |
| TOON_DEFAULT (0.00%) | JSON_PRETTY (-2.47%) | CSV (-1.59%) | TOON_DEFAULT (-3.17%) |
| XML_COMPACT (0.00%) | TOON_DEFAULT (-2.47%) | JSON_COMPACT (-1.59%) | XML_PRETTY (-3.17%) |
| XML_PRETTY (0.00%) | XML_COMPACT (-4.94%) | XML_PRETTY (-1.59%) | YAML (-4.76%) |
| YAML (0.00%) | YAML (-4.94%) | XML_COMPACT (-3.17%) | XML_COMPACT (-6.35%) |


#### 2.1.3 Category Accuracy By Character Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| CSV ≈ 100.00% | JSON_PRETTY ≈ 100.00% | JSON_COMPACT ≈ 100.00% | JSON_PRETTY ≈ 99.38% |
| JSON_PRETTY (0.00%) | TOON_DEFAULT (0.00%) | CSV (-0.30%) | XML_COMPACT (0.00%) |
| TOON_DEFAULT (0.00%) | YAML (0.00%) | JSON_PRETTY (-0.30%) | YAML (0.00%) |
| XML_COMPACT (0.00%) | JSON_COMPACT (-0.03%) | XML_COMPACT (-0.30%) | JSON_COMPACT (-0.41%) |
| XML_PRETTY (0.00%) | XML_COMPACT (-0.03%) | TOON_DEFAULT (-0.59%) | CSV (-0.62%) |
| JSON_COMPACT (-0.68%) | XML_PRETTY (-0.12%) | XML_PRETTY (-0.59%) | TOON_DEFAULT (-0.62%) |
| YAML (-0.69%) | CSV (-0.15%) | YAML (-0.89%) | XML_PRETTY (-0.62%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| CSV ≈ 100.00% | CSV ≈ 100.00% | JSON_PRETTY ≈ 100.00% | CSV ≈ 97.37% |
| JSON_COMPACT (0.00%) | JSON_COMPACT (0.00%) | TOON_DEFAULT (0.00%) | JSON_COMPACT (-2.82%) |
| JSON_PRETTY (0.00%) | XML_PRETTY (0.00%) | YAML (0.00%) | JSON_PRETTY (-2.82%) |
| TOON_DEFAULT (0.00%) | JSON_PRETTY (-0.04%) | CSV (-0.29%) | TOON_DEFAULT (-2.82%) |
| XML_COMPACT (0.00%) | XML_COMPACT (-0.16%) | JSON_COMPACT (-0.29%) | XML_PRETTY (-2.82%) |
| XML_PRETTY (0.00%) | YAML (-0.24%) | XML_COMPACT (-0.59%) | YAML (-3.43%) |
| YAML (0.00%) | TOON_DEFAULT (-0.24%) | XML_PRETTY (-1.18%) | XML_COMPACT (-4.03%) |


#### 2.1.4 Conclusion

- On mandatory data **TOON_DEFAULT** achieves the best total token efficiency score (87.24) by combining compactness close to **CSV** with an explicit schema per record. **JSON_PRETTY** wins on pure output efficiency (99.59) and raw accuracy (99.46%) at the cost of the highest mandatory read token count among non **XML** formats. **JSON_COMPACT** offers the most balanced mandatory profile without leading any single metric.
- On optional data **JSON_COMPACT** becomes the dominant choice across almost every efficiency metric because it omits absent fields entirely and keeps structure parseable. **TOON** loses its tabular advantage here because its adaptive encoding switches to a **YAML** like key value layout on sparse records which increases read tokens by 24.85% relative to its mandatory variant. **CSV** remains the most predictable format across both variants with the smallest variance in accuracy and token count.
- The **XML** formats occupy the bottom of both mandatory and optional rankings. They pay the largest read token cost and also produce the weakest accuracy scores especially in the optional variant where **XML_COMPACT** drops to 96.24% complete answer accuracy. **YAML** sits in the middle of both rankings without standing out on any specific metric.
- Accuracy by character remains above 99.69% for every combination tested so no format produces answers that are fundamentally wrong at this record count. The meaningful tradeoff is therefore tokens spent per correct answer rather than correctness itself. For dense tabular data **TOON_DEFAULT** is the recommended choice, for sparse data with optional fields **JSON_COMPACT** is recommended and **CSV** is the safe default when robustness across unknown data shapes matters more than peak efficiency.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total | Accuracy By Character (%) | Useful Read Tokens (Acc By Char) | Wasted Read Tokens (Acc By Char) | Useful Output Tokens (Acc By Char) | Wasted Output Tokens (Acc By Char) | Eff Score Read (Acc By Char) | Eff Score Output (Acc By Char) | Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6748 | 27978 | 34726 | 1.50 | 223.29 | 98.93 | 6676 | 72 | 27679 | 299 | 94.41 | 66.00 | 76.04 | 99.88 | 6740 | 8 | 27944 | 34 | 95.04 | 66.63 | 76.67 |
| CSV | opt | 6490 | 26495 | 32985 | 1.48 | 211.38 | 98.65 | 6402 | 88 | 26138 | 358 | 95.38 | 72.49 | 82.10 | 99.95 | 6487 | 3 | 26482 | 13 | 96.24 | 73.36 | 82.97 |
| JSON_COMPACT | man | 9027 | 23026 | 32053 | 2.21 | 183.28 | 98.66 | 8906 | 121 | 22718 | 309 | 84.01 | 88.12 | 85.46 | 99.69 | 8999 | 28 | 22955 | 71 | 84.70 | 88.81 | 86.14 |
| JSON_COMPACT | opt | 5669 | 22594 | 28263 | 3.26 | 179.79 | 98.12 | 5562 | 107 | 22169 | 425 | 98.70 | 89.71 | 98.71 | 99.89 | 5663 | 6 | 22569 | 25 | 99.88 | 90.89 | 99.89 |
| JSON_PRETTY | man | 11204 | 20597 | 31801 | 2.16 | 164.47 | 99.46 | 11143 | 61 | 20486 | 111 | 74.79 | 99.59 | 86.90 | 99.98 | 11202 | 2 | 20593 | 4 | 75.14 | 99.94 | 87.24 |
| JSON_PRETTY | opt | 10288 | 22285 | 32573 | 2.18 | 177.30 | 97.85 | 10067 | 221 | 21806 | 479 | 77.82 | 90.92 | 83.05 | 99.87 | 10275 | 13 | 22256 | 29 | 79.17 | 92.27 | 84.40 |
| TOON_DEFAULT | man | 6798 | 24808 | 31606 | 1.50 | 197.63 | 98.92 | 6725 | 73 | 24540 | 268 | 94.18 | 80.27 | 87.24 | 99.97 | 6796 | 2 | 24801 | 7 | 94.88 | 80.97 | 87.94 |
| TOON_DEFAULT | opt | 8487 | 25439 | 33926 | 2.31 | 202.73 | 97.85 | 8305 | 182 | 24892 | 547 | 85.89 | 76.71 | 78.19 | 99.77 | 8467 | 20 | 25381 | 59 | 87.17 | 77.99 | 79.47 |
| XML_COMPACT | man | 11444 | 26079 | 37523 | 2.42 | 207.91 | 99.19 | 11351 | 93 | 25868 | 211 | 73.53 | 74.72 | 66.16 | 99.97 | 11441 | 3 | 26071 | 8 | 74.05 | 75.24 | 66.68 |
| XML_COMPACT | opt | 7851 | 26642 | 34493 | 3.26 | 212.47 | 96.24 | 7556 | 295 | 25641 | 1002 | 87.67 | 70.22 | 75.08 | 99.77 | 7833 | 18 | 26581 | 61 | 90.02 | 72.57 | 77.43 |
| XML_PRETTY | man | 13087 | 23041 | 36128 | 2.39 | 183.43 | 97.04 | 12700 | 387 | 22359 | 682 | 64.74 | 86.97 | 69.74 | 99.90 | 13074 | 13 | 23018 | 23 | 66.64 | 88.88 | 71.65 |
| XML_PRETTY | opt | 12018 | 22126 | 34144 | 2.41 | 176.79 | 98.12 | 11792 | 226 | 21710 | 416 | 70.25 | 91.81 | 77.59 | 99.88 | 12004 | 14 | 22099 | 27 | 71.42 | 92.99 | 78.76 |
| YAML | man | 9479 | 23774 | 33253 | 2.20 | 190.13 | 98.66 | 9352 | 127 | 23455 | 319 | 81.99 | 84.75 | 81.15 | 99.70 | 9451 | 28 | 23702 | 71 | 82.68 | 85.45 | 81.84 |
| YAML | opt | 8696 | 24835 | 33531 | 2.23 | 198.69 | 97.04 | 8439 | 257 | 24100 | 735 | 84.42 | 78.90 | 79.07 | 99.75 | 8674 | 22 | 24773 | 62 | 86.22 | 80.70 | 80.88 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6748 | 6490 | -258 | -3.82 | 290 | 285 | -5 | -1.72 | 27688 | 26211 | -1477 | -5.33 | 27978 | 26495 | -1483 | -5.30 | 34726 | 32985 | -1741 | -5.01 |
| JSON_COMPACT | 9027 | 5669 | -3358 | -37.20 | 300 | 300 | 0 | 0.00 | 22726 | 22294 | -432 | -1.90 | 23026 | 22594 | -432 | -1.88 | 32053 | 28263 | -3790 | -11.82 |
| JSON_PRETTY | 11204 | 10288 | -916 | -8.18 | 203 | 300 | +97 | +47.78 | 20394 | 21985 | +1591 | +7.80 | 20597 | 22285 | +1688 | +8.20 | 31801 | 32573 | +772 | +2.43 |
| TOON_DEFAULT | 6798 | 8487 | +1689 | +24.85 | 301 | 300 | -1 | -0.33 | 24507 | 25139 | +632 | +2.58 | 24808 | 25439 | +631 | +2.54 | 31606 | 33926 | +2320 | +7.34 |
| XML_COMPACT | 11444 | 7851 | -3593 | -31.40 | 298 | 296 | -2 | -0.67 | 25781 | 26346 | +565 | +2.19 | 26079 | 26642 | +563 | +2.16 | 37523 | 34493 | -3030 | -8.08 |
| XML_PRETTY | 13087 | 12018 | -1069 | -8.17 | 296 | 204 | -92 | -31.08 | 22746 | 21922 | -824 | -3.62 | 23041 | 22126 | -915 | -3.97 | 36128 | 34144 | -1984 | -5.49 |
| YAML | 9479 | 8696 | -783 | -8.26 | 198 | 198 | 0 | 0.00 | 23576 | 24637 | +1061 | +4.50 | 23774 | 24835 | +1061 | +4.46 | 33253 | 33531 | +278 | +0.84 |

### 2.4 Drift over multiple runs

*Note: Drift = The distance from average to the lowest or highest value. Spread = Distance between lowest to highest value (larger = less predictable results across runs).*

#### 2.4.1 Drift per Format and Variant
| Format | Variant | Runs | Output Tokens Total | Drift | Spread | Accuracy (%) | Drift (pp) | Spread (pp) | Accuracy By Character (%) | Drift (pp) | Spread (pp) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 3 | 27978 | -1304/+1063 | 2367 | 98.93 | -0.55/+1.08 | 1.63 | 99.88 | -0.18/+0.12 | 0.30 |
| CSV | opt | 3 | 26495 | -2703/+1404 | 4107 | 98.65 | -1.08/+0.55 | 1.63 | 99.95 | -0.07/+0.03 | 0.10 |
| JSON_COMPACT | man | 3 | 23026 | -4297/+8296 | 12593 | 98.66 | -2.73/+1.36 | 4.09 | 99.69 | -0.61/+0.31 | 0.92 |
| JSON_COMPACT | opt | 3 | 22594 | -2901/+2693 | 5594 | 98.12 | -0.55/+0.28 | 0.83 | 99.89 | -0.01/0.00 | 0.01 |
| JSON_PRETTY | man | 3 | 20597 | -659/+1300 | 1959 | 99.46 | -0.27/+0.54 | 0.81 | 99.98 | -0.02/+0.02 | 0.04 |
| JSON_PRETTY | opt | 3 | 22285 | -1767/+3394 | 5161 | 97.85 | -0.28/+0.55 | 0.83 | 99.87 | -0.01/+0.02 | 0.03 |
| TOON_DEFAULT | man | 3 | 24808 | -933/+1382 | 2315 | 98.92 | -0.54/+0.27 | 0.81 | 99.97 | -0.02/+0.02 | 0.04 |
| TOON_DEFAULT | opt | 3 | 25439 | -1394/+801 | 2195 | 97.85 | -0.28/+0.55 | 0.83 | 99.77 | -0.23/+0.12 | 0.35 |
| XML_COMPACT | man | 3 | 26079 | -939/+1244 | 2183 | 99.19 | 0.00/0.00 | 0.00 | 99.97 | -0.02/+0.02 | 0.04 |
| XML_COMPACT | opt | 3 | 26642 | -1340/+991 | 2331 | 96.24 | -1.96/+2.23 | 4.19 | 99.77 | -0.09/+0.12 | 0.21 |
| XML_PRETTY | man | 3 | 23041 | -979/+548 | 1528 | 97.04 | -0.28/+0.56 | 0.84 | 99.90 | -0.02/+0.01 | 0.03 |
| XML_PRETTY | opt | 3 | 22126 | -2697/+4516 | 7213 | 98.12 | -0.55/+0.28 | 0.83 | 99.88 | -0.03/+0.01 | 0.04 |
| YAML | man | 3 | 23774 | -1374/+1353 | 2727 | 98.66 | -1.09/+1.36 | 2.45 | 99.70 | -0.57/+0.30 | 0.87 |
| YAML | opt | 3 | 24835 | -1902/+1316 | 3219 | 97.04 | -1.10/+1.39 | 2.49 | 99.75 | -0.13/+0.14 | 0.27 |

#### 2.4.2 Spread: Mandatory vs Optional
| Format | Output Tokens Total Spread Man | Output Tokens Total Spread Opt | Diff | Acc Spread Man (pp) | Acc Spread Opt (pp) | Diff (pp) | Acc By Char Spread Man (pp) | Acc By Char Spread Opt (pp) | Diff (pp) |
|---|---|---|---|---|---|---|---|---|---|
| CSV | 2367 | 4107 | +1740 | 1.63 | 1.63 | 0.00 | 0.30 | 0.10 | -0.20 |
| JSON_COMPACT | 12593 | 5594 | -6999 | 4.09 | 0.83 | -3.26 | 0.92 | 0.01 | -0.91 |
| JSON_PRETTY | 1959 | 5161 | +3202 | 0.81 | 0.83 | +0.02 | 0.04 | 0.03 | -0.01 |
| TOON_DEFAULT | 2315 | 2195 | -119 | 0.81 | 0.83 | +0.02 | 0.04 | 0.35 | +0.31 |
| XML_COMPACT | 2183 | 2331 | +148 | 0.00 | 4.19 | +4.19 | 0.04 | 0.21 | +0.17 |
| XML_PRETTY | 1528 | 7213 | +5685 | 0.84 | 0.83 | -0.01 | 0.03 | 0.04 | +0.01 |
| YAML | 2727 | 3219 | +492 | 2.45 | 2.49 | +0.04 | 0.87 | 0.27 | -0.60 |

### 2.5 Performance
#### 2.5.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Before Write (ms) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) | Output (ms) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 31 | 217.68 | 1.00 | 294296 | 74754 | 0.37 | 602.85 | 74785 | 218.05 | 482.48 | 369050 |
| CSV | opt | 9 | 721.11 | 0.29 | 276091 | 56578 | 0.46 | 456.27 | 56587 | 721.57 | 365.08 | 332669 |
| JSON_COMPACT | man | 28 | 322.39 | 0.90 | 229416 | 60482 | 0.38 | 487.76 | 60510 | 322.77 | 390.39 | 289898 |
| JSON_COMPACT | opt | 32 | 177.16 | 1.03 | 229832 | 61707 | 0.36 | 497.64 | 61739 | 177.52 | 398.32 | 291539 |
| JSON_PRETTY | man | 20 | 560.20 | 0.65 | 190946 | 81750 | 0.25 | 659.27 | 81770 | 560.45 | 527.55 | 272696 |
| JSON_PRETTY | opt | 18 | 571.56 | 0.58 | 227171 | 57799 | 0.38 | 466.12 | 57817 | 571.94 | 373.01 | 284970 |
| TOON_DEFAULT | man | 36 | 188.83 | 1.16 | 263121 | 71035 | 0.35 | 572.86 | 71071 | 189.18 | 458.52 | 334156 |
| TOON_DEFAULT | opt | 71 | 119.54 | 2.29 | 253367 | 58135 | 0.43 | 468.83 | 58206 | 119.97 | 375.52 | 311501 |
| XML_COMPACT | man | 36 | 317.89 | 1.16 | 269388 | 63247 | 0.41 | 510.06 | 63283 | 318.30 | 408.28 | 332635 |
| XML_COMPACT | opt | 31 | 253.26 | 1.00 | 275798 | 57918 | 0.46 | 467.08 | 57949 | 253.71 | 373.86 | 333716 |
| XML_PRETTY | man | 37 | 353.70 | 1.19 | 226726 | 74194 | 0.31 | 598.34 | 74231 | 354.01 | 478.91 | 300920 |
| XML_PRETTY | opt | 37 | 324.81 | 1.19 | 222643 | 58058 | 0.38 | 468.21 | 58095 | 325.19 | 374.81 | 280701 |
| YAML | man | 10 | 947.90 | 0.32 | 253987 | 63856 | 0.37 | 514.97 | 63866 | 948.27 | 412.04 | 317843 |
| YAML | opt | 23 | 378.09 | 0.74 | 266774 | 59799 | 0.41 | 482.25 | 59822 | 378.50 | 385.95 | 326573 |

#### 2.5.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Before Write Man (s) | Output Before Write Opt (s) | Diff (s) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 31 | 9 | -22 | -70.97 | 294.30 | 276.09 | -18.20 | -6.19 | 74.75 | 56.58 | -18.18 | -24.31 | 74.79 | 56.59 | -18.20 | -24.33 | 369.05 | 332.67 | -36.38 | -9.86 |
| JSON_COMPACT | 28 | 32 | +4 | +14.29 | 229.42 | 229.83 | +0.42 | +0.18 | 60.48 | 61.71 | +1.23 | +2.03 | 60.51 | 61.74 | +1.23 | +2.03 | 289.90 | 291.54 | +1.64 | +0.57 |
| JSON_PRETTY | 20 | 18 | -2 | -10.00 | 190.95 | 227.17 | +36.23 | +18.97 | 81.75 | 57.80 | -23.95 | -29.30 | 81.77 | 57.82 | -23.95 | -29.29 | 272.70 | 284.97 | +12.27 | +4.50 |
| TOON_DEFAULT | 36 | 71 | +35 | +97.22 | 263.12 | 253.37 | -9.75 | -3.71 | 71.04 | 58.13 | -12.90 | -18.16 | 71.07 | 58.21 | -12.87 | -18.10 | 334.16 | 311.50 | -22.65 | -6.78 |
| XML_COMPACT | 36 | 31 | -5 | -13.89 | 269.39 | 275.80 | +6.41 | +2.38 | 63.25 | 57.92 | -5.33 | -8.43 | 63.28 | 57.95 | -5.33 | -8.43 | 332.64 | 333.72 | +1.08 | +0.32 |
| XML_PRETTY | 37 | 37 | 0 | 0.00 | 226.73 | 222.64 | -4.08 | -1.80 | 74.19 | 58.06 | -16.14 | -21.75 | 74.23 | 58.10 | -16.14 | -21.74 | 300.92 | 280.70 | -20.22 | -6.72 |
| YAML | 10 | 23 | +13 | +130.00 | 253.99 | 266.77 | +12.79 | +5.03 | 63.86 | 59.80 | -4.06 | -6.35 | 63.87 | 59.82 | -4.04 | -6.33 | 317.84 | 326.57 | +8.73 | +2.75 |

### 2.6 Structural Efficiency
#### 2.6.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token | Info / Read Token (Acc By Char) | Info / Output Token (Acc By Char) | Info / Total Token (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 1.50 | 9.89 | 217.68 | 1.47 | 0.35 | 0.28 | 1.48 | 0.36 | 0.29 |
| CSV | opt | 1.48 | 10.29 | 209.36 | 1.52 | 0.37 | 0.30 | 1.54 | 0.38 | 0.30 |
| JSON_COMPACT | man | 2.21 | 13.24 | 291.19 | 1.09 | 0.43 | 0.31 | 1.10 | 0.43 | 0.31 |
| JSON_COMPACT | opt | 3.26 | 8.98 | 182.87 | 1.73 | 0.43 | 0.35 | 1.76 | 0.44 | 0.35 |
| JSON_PRETTY | man | 2.16 | 16.43 | 361.42 | 0.89 | 0.48 | 0.31 | 0.89 | 0.49 | 0.31 |
| JSON_PRETTY | opt | 2.18 | 16.30 | 331.87 | 0.95 | 0.44 | 0.30 | 0.97 | 0.45 | 0.31 |
| TOON_DEFAULT | man | 1.50 | 9.97 | 219.29 | 1.46 | 0.40 | 0.31 | 1.47 | 0.40 | 0.32 |
| TOON_DEFAULT | opt | 2.31 | 13.45 | 273.77 | 1.15 | 0.39 | 0.29 | 1.18 | 0.39 | 0.29 |
| XML_COMPACT | man | 2.42 | 16.78 | 369.16 | 0.87 | 0.38 | 0.26 | 0.87 | 0.38 | 0.27 |
| XML_COMPACT | opt | 3.26 | 12.44 | 253.26 | 1.23 | 0.36 | 0.28 | 1.27 | 0.37 | 0.29 |
| XML_PRETTY | man | 2.39 | 19.19 | 422.16 | 0.74 | 0.42 | 0.27 | 0.76 | 0.43 | 0.28 |
| XML_PRETTY | opt | 2.41 | 19.05 | 387.68 | 0.82 | 0.44 | 0.29 | 0.83 | 0.45 | 0.29 |
| YAML | man | 2.20 | 13.90 | 305.77 | 1.04 | 0.42 | 0.30 | 1.05 | 0.42 | 0.30 |
| YAML | opt | 2.23 | 13.78 | 280.52 | 1.12 | 0.39 | 0.29 | 1.15 | 0.40 | 0.30 |

#### 2.6.2 Characters And Values: Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.50 | 1.48 | -0.02 | -1.34 | 9.89 | 10.29 | +0.39 | +3.95 | 217.68 | 209.36 | -8.32 | -3.82 |
| JSON_COMPACT | 2.21 | 3.26 | +1.06 | +47.82 | 13.24 | 8.98 | -4.25 | -32.12 | 291.19 | 182.87 | -108.32 | -37.20 |
| JSON_PRETTY | 2.16 | 2.18 | +0.02 | +1.06 | 16.43 | 16.30 | -0.12 | -0.75 | 361.42 | 331.87 | -29.55 | -8.18 |
| TOON_DEFAULT | 1.50 | 2.31 | +0.82 | +54.72 | 9.97 | 13.45 | +3.48 | +34.93 | 219.29 | 273.77 | +54.48 | +24.85 |
| XML_COMPACT | 2.42 | 3.26 | +0.84 | +34.80 | 16.78 | 12.44 | -4.34 | -25.85 | 369.16 | 253.26 | -115.90 | -31.40 |
| XML_PRETTY | 2.39 | 2.41 | +0.02 | +0.75 | 19.19 | 19.05 | -0.14 | -0.75 | 422.16 | 387.68 | -34.48 | -8.17 |
| YAML | 2.20 | 2.23 | +0.03 | +1.32 | 13.90 | 13.78 | -0.12 | -0.85 | 305.77 | 280.52 | -25.26 | -8.26 |

#### 2.6.3 Information: Mandatory vs Optional
| Format | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.47 | 1.52 | +0.05 | +3.68 | 0.35 | 0.37 | +0.02 | +5.08 | 0.28 | 0.30 | +0.01 | +4.91 |
| JSON_COMPACT | 1.09 | 1.73 | +0.64 | +58.37 | 0.43 | 0.43 | +0.01 | +1.40 | 0.31 | 0.35 | +0.04 | +12.66 |
| JSON_PRETTY | 0.89 | 0.95 | +0.06 | +7.09 | 0.48 | 0.44 | -0.04 | -9.11 | 0.31 | 0.30 | -0.01 | -4.15 |
| TOON_DEFAULT | 1.46 | 1.15 | -0.30 | -20.76 | 0.40 | 0.39 | -0.01 | -3.51 | 0.31 | 0.29 | -0.03 | -7.99 |
| XML_COMPACT | 0.87 | 1.23 | +0.36 | +41.41 | 0.38 | 0.36 | -0.02 | -5.00 | 0.26 | 0.28 | +0.02 | +5.68 |
| XML_PRETTY | 0.74 | 0.82 | +0.07 | +10.12 | 0.42 | 0.44 | +0.02 | +5.23 | 0.27 | 0.29 | +0.02 | +6.69 |
| YAML | 1.04 | 1.12 | +0.08 | +7.20 | 0.42 | 0.39 | -0.02 | -5.78 | 0.30 | 0.29 | -0.01 | -2.69 |

#### 2.6.4 Information (Accuracy By Character): Mandatory vs Optional
| Format | Info / Read Token (Acc By Char) Man | Info / Read Token (Acc By Char) Opt | Diff | Diff (%) | Info / Output Token (Acc By Char) Man | Info / Output Token (Acc By Char)  Opt | Diff | Diff (%) | Info / Total Token (Acc By Char) Man | Info / Total Token (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.48 | 1.54 | +0.06 | +4.05 | 0.36 | 0.38 | +0.02 | +5.60 | 0.29 | 0.30 | +0.02 | +5.21 |
| JSON_COMPACT | 1.10 | 1.76 | +0.66 | +59.60 | 0.43 | 0.44 | +0.01 | +2.08 | 0.31 | 0.35 | +0.04 | +13.50 |
| JSON_PRETTY | 0.89 | 0.97 | +0.08 | +8.86 | 0.49 | 0.45 | -0.04 | -7.63 | 0.31 | 0.31 | -0.01 | -2.23 |
| TOON_DEFAULT | 1.47 | 1.18 | -0.30 | -20.05 | 0.40 | 0.39 | -0.01 | -2.73 | 0.32 | 0.29 | -0.02 | -6.96 |
| XML_COMPACT | 0.87 | 1.27 | +0.40 | +45.42 | 0.38 | 0.37 | -0.01 | -2.35 | 0.27 | 0.29 | +0.02 | +8.65 |
| XML_PRETTY | 0.76 | 0.83 | +0.07 | +8.91 | 0.43 | 0.45 | +0.02 | +3.92 | 0.28 | 0.29 | +0.02 | +5.78 |
| YAML | 1.05 | 1.15 | +0.09 | +9.03 | 0.42 | 0.40 | -0.02 | -4.06 | 0.30 | 0.30 | 0.00 | 0.00 |

### 2.7 Token Utilization Efficiency
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6748 | 6676 | 72 | 27978 | 27679 | 299 | 34726 | 34354 | 372 | 98.93 | 94.41 | 66.00 | 76.04 | 98.91 | 94.39 | 65.98 | 76.02 |
| CSV | opt | 6490 | 6402 | 88 | 26495 | 26138 | 358 | 32985 | 32540 | 445 | 98.65 | 95.38 | 72.49 | 82.10 | 98.87 | 95.52 | 72.64 | 82.25 |
| JSON_COMPACT | man | 9027 | 8906 | 121 | 23026 | 22718 | 309 | 32053 | 31624 | 430 | 98.66 | 84.01 | 88.12 | 85.46 | 98.66 | 84.01 | 88.12 | 85.46 |
| JSON_COMPACT | opt | 5669 | 5562 | 107 | 22594 | 22169 | 425 | 28263 | 27732 | 531 | 98.12 | 98.70 | 89.71 | 98.71 | 98.48 | 98.94 | 89.95 | 98.95 |
| JSON_PRETTY | man | 11204 | 11143 | 61 | 20597 | 20486 | 111 | 31801 | 31629 | 172 | 99.46 | 74.79 | 99.59 | 86.90 | 99.47 | 74.80 | 99.60 | 86.90 |
| JSON_PRETTY | opt | 10288 | 10067 | 221 | 22285 | 21806 | 479 | 32573 | 31873 | 700 | 97.85 | 77.82 | 90.92 | 83.05 | 98.09 | 77.98 | 91.08 | 83.21 |
| TOON_DEFAULT | man | 6798 | 6725 | 73 | 24808 | 24540 | 268 | 31606 | 31265 | 341 | 98.92 | 94.18 | 80.27 | 87.24 | 98.94 | 94.19 | 80.28 | 87.25 |
| TOON_DEFAULT | opt | 8487 | 8305 | 182 | 25439 | 24892 | 547 | 33926 | 33197 | 729 | 97.85 | 85.89 | 76.71 | 78.19 | 98.09 | 86.05 | 76.87 | 78.35 |
| XML_COMPACT | man | 11444 | 11351 | 93 | 26079 | 25868 | 211 | 37523 | 37219 | 304 | 99.19 | 73.53 | 74.72 | 66.16 | 99.11 | 73.48 | 74.67 | 66.11 |
| XML_COMPACT | opt | 7851 | 7556 | 295 | 26642 | 25641 | 1002 | 34493 | 33196 | 1297 | 96.24 | 87.67 | 70.22 | 75.08 | 96.31 | 87.72 | 70.27 | 75.12 |
| XML_PRETTY | man | 13087 | 12700 | 387 | 23041 | 22359 | 682 | 36128 | 35059 | 1069 | 97.04 | 64.74 | 86.97 | 69.74 | 96.42 | 64.32 | 86.56 | 69.33 |
| XML_PRETTY | opt | 12018 | 11792 | 226 | 22126 | 21710 | 416 | 34144 | 33502 | 642 | 98.12 | 70.25 | 91.81 | 77.59 | 98.48 | 70.49 | 92.05 | 77.83 |
| YAML | man | 9479 | 9352 | 127 | 23774 | 23455 | 319 | 33253 | 32807 | 446 | 98.66 | 81.99 | 84.75 | 81.15 | 98.58 | 81.93 | 84.70 | 81.09 |
| YAML | opt | 8696 | 8439 | 257 | 24835 | 24100 | 735 | 33531 | 32538 | 993 | 97.04 | 84.42 | 78.90 | 79.07 | 97.17 | 84.50 | 78.98 | 79.16 |

#### 2.7.2 Read Tokens: Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6748 | 6490 | -258 | -3.82 | 6676 | 6403 | -273 | -4.10 | 72 | 87 | +15 | +21.40 | 98.93 | 98.65 | -0.28 | 94.41 | 95.38 | +0.97 | +1.03 | 98.91 | 98.87 | -0.04 | 94.39 | 95.52 | +1.13 | +1.20 |
| JSON_COMPACT | 9027 | 5669 | -3358 | -37.20 | 8906 | 5562 | -3344 | -37.54 | 121 | 107 | -14 | -11.89 | 98.66 | 98.12 | -0.54 | 84.01 | 98.70 | +14.69 | +17.48 | 98.66 | 98.48 | -0.18 | 84.01 | 98.94 | +14.93 | +17.77 |
| JSON_PRETTY | 11204 | 10288 | -916 | -8.18 | 11143 | 10066 | -1077 | -9.66 | 61 | 222 | +161 | +263.43 | 99.46 | 97.85 | -1.61 | 74.79 | 77.82 | +3.03 | +4.05 | 99.47 | 98.09 | -1.38 | 74.80 | 77.98 | +3.18 | +4.26 |
| TOON_DEFAULT | 6798 | 8487 | +1689 | +24.85 | 6725 | 8305 | +1580 | +23.49 | 73 | 182 | +109 | +149.39 | 98.92 | 97.85 | -1.07 | 94.18 | 85.89 | -8.28 | -8.80 | 98.94 | 98.09 | -0.85 | 94.19 | 86.05 | -8.14 | -8.64 |
| XML_COMPACT | 11444 | 7851 | -3593 | -31.40 | 11351 | 7555 | -3796 | -33.44 | 93 | 296 | +203 | +217.74 | 99.19 | 96.24 | -2.95 | 73.53 | 87.67 | +14.13 | +19.22 | 99.11 | 96.31 | -2.80 | 73.48 | 87.72 | +14.24 | +19.37 |
| XML_PRETTY | 13087 | 12018 | -1069 | -8.17 | 12700 | 11792 | -908 | -7.15 | 387 | 226 | -161 | -41.71 | 97.04 | 98.12 | +1.08 | 64.74 | 70.25 | +5.51 | +8.51 | 96.42 | 98.48 | +2.06 | 64.32 | 70.49 | +6.16 | +9.58 |
| YAML | 9479 | 8696 | -783 | -8.26 | 9352 | 8439 | -913 | -9.77 | 127 | 257 | +130 | +102.66 | 98.66 | 97.04 | -1.62 | 81.99 | 84.42 | +2.43 | +2.96 | 98.58 | 97.17 | -1.41 | 81.93 | 84.50 | +2.57 | +3.14 |

#### 2.7.3 Output Tokens: Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 27978 | 26495 | -1483 | -5.30 | 27679 | 26138 | -1541 | -5.57 | 299 | 357 | +58 | +19.51 | 98.93 | 98.65 | -0.28 | 66.00 | 72.49 | +6.49 | +9.84 | 98.91 | 98.87 | -0.04 | 65.98 | 72.64 | +6.65 | +10.08 |
| JSON_COMPACT | 23026 | 22594 | -432 | -1.88 | 22718 | 22169 | -549 | -2.41 | 309 | 425 | +116 | +37.61 | 98.66 | 98.12 | -0.54 | 88.12 | 89.71 | +1.59 | +1.80 | 98.66 | 98.48 | -0.18 | 88.12 | 89.95 | +1.83 | +2.07 |
| JSON_PRETTY | 20597 | 22285 | +1688 | +8.20 | 20486 | 21806 | +1320 | +6.44 | 111 | 479 | +368 | +331.45 | 99.46 | 97.85 | -1.61 | 99.59 | 90.92 | -8.68 | -8.71 | 99.47 | 98.09 | -1.38 | 99.60 | 91.08 | -8.52 | -8.56 |
| TOON_DEFAULT | 24808 | 25439 | +631 | +2.54 | 24540 | 24892 | +352 | +1.44 | 268 | 547 | +279 | +104.11 | 98.92 | 97.85 | -1.07 | 80.27 | 76.71 | -3.56 | -4.43 | 98.94 | 98.09 | -0.85 | 80.28 | 76.87 | -3.41 | -4.25 |
| XML_COMPACT | 26079 | 26642 | +563 | +2.16 | 25868 | 25641 | -227 | -0.88 | 211 | 1002 | +791 | +374.65 | 99.19 | 96.24 | -2.95 | 74.72 | 70.22 | -4.50 | -6.03 | 99.11 | 96.31 | -2.80 | 74.67 | 70.27 | -4.40 | -5.90 |
| XML_PRETTY | 23041 | 22126 | -915 | -3.97 | 22359 | 21710 | -649 | -2.90 | 682 | 416 | -266 | -39.01 | 97.04 | 98.12 | +1.08 | 86.97 | 91.81 | +4.84 | +5.57 | 96.42 | 98.48 | +2.06 | 86.56 | 92.05 | +5.50 | +6.35 |
| YAML | 23774 | 24835 | +1061 | +4.46 | 23455 | 24099 | +644 | +2.75 | 319 | 736 | +417 | +130.58 | 98.66 | 97.04 | -1.62 | 84.75 | 78.90 | -5.86 | -6.91 | 98.58 | 97.17 | -1.41 | 84.70 | 78.98 | -5.72 | -6.75 |

#### 2.7.4 Total Tokens: Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 34726 | 32985 | -1741 | -5.01 | 34354 | 32540 | -1814 | -5.28 | 372 | 446 | +74 | +19.82 | 98.93 | 98.65 | -0.28 | 76.04 | 82.10 | +6.07 | +7.98 | 98.91 | 98.87 | -0.04 | 76.02 | 82.25 | +6.23 | +8.19 |
| JSON_COMPACT | 32053 | 28263 | -3790 | -11.83 | 31624 | 27732 | -3892 | -12.31 | 430 | 532 | +102 | +23.68 | 98.66 | 98.12 | -0.54 | 85.46 | 98.71 | +13.25 | +15.51 | 98.66 | 98.48 | -0.18 | 85.46 | 98.95 | +13.50 | +15.79 |
| JSON_PRETTY | 31801 | 32573 | +772 | +2.43 | 31629 | 31872 | +243 | +0.77 | 172 | 701 | +529 | +307.32 | 99.46 | 97.85 | -1.61 | 86.90 | 83.05 | -3.85 | -4.43 | 99.47 | 98.09 | -1.38 | 86.90 | 83.21 | -3.69 | -4.25 |
| TOON_DEFAULT | 31606 | 33926 | +2320 | +7.34 | 31265 | 33197 | +1932 | +6.18 | 341 | 729 | +388 | +113.80 | 98.92 | 97.85 | -1.07 | 87.24 | 78.19 | -9.05 | -10.37 | 98.94 | 98.09 | -0.85 | 87.25 | 78.35 | -8.90 | -10.20 |
| XML_COMPACT | 37523 | 34493 | -3030 | -8.07 | 37219 | 33196 | -4023 | -10.81 | 304 | 1297 | +993 | +326.65 | 99.19 | 96.24 | -2.95 | 66.16 | 75.08 | +8.92 | +13.48 | 99.11 | 96.31 | -2.80 | 66.11 | 75.12 | +9.02 | +13.64 |
| XML_PRETTY | 36128 | 34144 | -1984 | -5.49 | 35059 | 33502 | -1557 | -4.44 | 1069 | 642 | -427 | -39.99 | 97.04 | 98.12 | +1.08 | 69.74 | 77.59 | +7.85 | +11.25 | 96.42 | 98.48 | +2.06 | 69.33 | 77.83 | +8.50 | +12.26 |
| YAML | 33253 | 33531 | +278 | +0.84 | 32807 | 32538 | -269 | -0.82 | 446 | 993 | +547 | +122.63 | 98.66 | 97.04 | -1.62 | 81.15 | 79.07 | -2.08 | -2.56 | 98.58 | 97.17 | -1.41 | 81.09 | 79.16 | -1.94 | -2.39 |

### 2.8 Token Utilization Efficiency (Accuracy by Character)
#### 2.8.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy by Character (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy by Character (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6748 | 6740 | 8 | 27978 | 27944 | 34 | 34726 | 34684 | 42 | 99.88 | 95.04 | 66.63 | 76.67 | 99.74 | 94.95 | 66.54 | 76.58 |
| CSV | opt | 6490 | 6487 | 3 | 26495 | 26482 | 13 | 32985 | 32969 | 16 | 99.95 | 96.24 | 73.36 | 82.97 | 99.61 | 96.02 | 73.13 | 82.74 |
| JSON_COMPACT | man | 9027 | 8999 | 28 | 23026 | 22955 | 71 | 32053 | 31954 | 99 | 99.69 | 84.70 | 88.81 | 86.14 | 99.61 | 84.65 | 88.75 | 86.09 |
| JSON_COMPACT | opt | 5669 | 5663 | 6 | 22594 | 22569 | 25 | 28263 | 28232 | 31 | 99.89 | 99.88 | 90.89 | 99.89 | 99.26 | 99.46 | 90.47 | 99.47 |
| JSON_PRETTY | man | 11204 | 11202 | 2 | 20597 | 20593 | 4 | 31801 | 31795 | 6 | 99.98 | 75.14 | 99.94 | 87.24 | 99.86 | 75.06 | 99.86 | 87.16 |
| JSON_PRETTY | opt | 10288 | 10275 | 13 | 22285 | 22256 | 29 | 32573 | 32531 | 42 | 99.87 | 79.17 | 92.27 | 84.40 | 99.31 | 78.79 | 91.89 | 84.02 |
| TOON_DEFAULT | man | 6798 | 6796 | 2 | 24808 | 24801 | 7 | 31606 | 31597 | 9 | 99.97 | 94.88 | 80.97 | 87.94 | 99.73 | 94.72 | 80.81 | 87.78 |
| TOON_DEFAULT | opt | 8487 | 8467 | 20 | 25439 | 25381 | 59 | 33926 | 33848 | 78 | 99.77 | 87.17 | 77.99 | 79.47 | 99.25 | 86.83 | 77.65 | 79.12 |
| XML_COMPACT | man | 11444 | 11441 | 3 | 26079 | 26071 | 8 | 37523 | 37512 | 11 | 99.97 | 74.05 | 75.24 | 66.68 | 99.85 | 73.97 | 75.16 | 66.60 |
| XML_COMPACT | opt | 7851 | 7833 | 18 | 26642 | 26581 | 61 | 34493 | 34414 | 79 | 99.77 | 90.02 | 72.57 | 77.43 | 98.99 | 89.50 | 72.05 | 76.91 |
| XML_PRETTY | man | 13087 | 13074 | 13 | 23041 | 23018 | 23 | 36128 | 36092 | 36 | 99.90 | 66.64 | 88.88 | 71.65 | 99.69 | 66.50 | 88.74 | 71.51 |
| XML_PRETTY | opt | 12018 | 12004 | 14 | 22126 | 22099 | 27 | 34144 | 34103 | 41 | 99.88 | 71.42 | 92.99 | 78.76 | 99.08 | 70.89 | 92.45 | 78.23 |
| YAML | man | 9479 | 9451 | 28 | 23774 | 23702 | 71 | 33253 | 33153 | 100 | 99.70 | 82.68 | 85.45 | 81.84 | 99.48 | 82.53 | 85.30 | 81.69 |
| YAML | opt | 8696 | 8674 | 22 | 24835 | 24773 | 62 | 33531 | 33447 | 84 | 99.75 | 86.22 | 80.70 | 80.88 | 99.17 | 85.84 | 80.32 | 80.49 |

#### 2.8.2 Read Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6748 | 6490 | -258 | -3.82 | 6740 | 6487 | -253 | -3.76 | 8 | 3 | -5 | -60.66 | 99.88 | 99.95 | +0.07 | 95.04 | 96.24 | +1.20 | +1.27 | 99.74 | 99.61 | -0.13 | 94.95 | 96.02 | +1.07 | +1.13 |
| JSON_COMPACT | 9027 | 5669 | -3358 | -37.20 | 8999 | 5663 | -3336 | -37.07 | 28 | 6 | -22 | -77.67 | 99.69 | 99.89 | +0.20 | 84.70 | 99.88 | +15.18 | +17.92 | 99.61 | 99.26 | -0.35 | 84.65 | 99.46 | +14.82 | +17.50 |
| JSON_PRETTY | 11204 | 10288 | -916 | -8.18 | 11202 | 10275 | -927 | -8.28 | 2 | 13 | +11 | +556.65 | 99.98 | 99.87 | -0.11 | 75.14 | 79.17 | +4.03 | +5.37 | 99.86 | 99.31 | -0.55 | 75.06 | 78.79 | +3.74 | +4.98 |
| TOON_DEFAULT | 6798 | 8487 | +1689 | +24.85 | 6796 | 8468 | +1672 | +24.60 | 2 | 19 | +17 | +874.05 | 99.97 | 99.77 | -0.20 | 94.88 | 87.17 | -7.70 | -8.12 | 99.73 | 99.25 | -0.48 | 94.72 | 86.83 | -7.89 | -8.33 |
| XML_COMPACT | 11444 | 7851 | -3593 | -31.40 | 11441 | 7833 | -3608 | -31.53 | 3 | 18 | +15 | +487.47 | 99.97 | 99.77 | -0.20 | 74.05 | 90.02 | +15.97 | +21.56 | 99.85 | 98.99 | -0.86 | 73.97 | 89.50 | +15.53 | +20.99 |
| XML_PRETTY | 13087 | 12018 | -1069 | -8.17 | 13074 | 12004 | -1070 | -8.19 | 13 | 14 | +1 | +10.27 | 99.90 | 99.88 | -0.02 | 66.64 | 71.42 | +4.78 | +7.17 | 99.69 | 99.08 | -0.61 | 66.50 | 70.89 | +4.38 | +6.59 |
| YAML | 9479 | 8696 | -783 | -8.26 | 9451 | 8675 | -776 | -8.21 | 28 | 21 | -7 | -23.92 | 99.70 | 99.75 | +0.05 | 82.68 | 86.22 | +3.54 | +4.28 | 99.48 | 99.17 | -0.31 | 82.53 | 85.84 | +3.30 | +4.00 |

#### 2.8.3 Output Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 27978 | 26495 | -1483 | -5.30 | 27944 | 26482 | -1462 | -5.23 | 34 | 14 | -20 | -59.78 | 99.88 | 99.95 | +0.07 | 66.63 | 73.36 | +6.72 | +10.09 | 99.74 | 99.61 | -0.13 | 66.54 | 73.13 | +6.59 | +9.91 |
| JSON_COMPACT | 23026 | 22594 | -432 | -1.88 | 22955 | 22569 | -386 | -1.68 | 71 | 24 | -47 | -65.53 | 99.69 | 99.89 | +0.20 | 88.81 | 90.89 | +2.08 | +2.34 | 99.61 | 99.26 | -0.35 | 88.75 | 90.47 | +1.71 | +1.93 |
| JSON_PRETTY | 20597 | 22285 | +1688 | +8.20 | 20593 | 22256 | +1663 | +8.08 | 4 | 29 | +25 | +621.28 | 99.98 | 99.87 | -0.11 | 99.94 | 92.27 | -7.68 | -7.68 | 99.86 | 99.31 | -0.55 | 99.86 | 91.89 | -7.97 | -7.98 |
| TOON_DEFAULT | 24808 | 25439 | +631 | +2.54 | 24801 | 25381 | +580 | +2.34 | 7 | 58 | +51 | +729.54 | 99.97 | 99.77 | -0.20 | 80.97 | 77.99 | -2.98 | -3.68 | 99.73 | 99.25 | -0.48 | 80.81 | 77.65 | -3.16 | -3.91 |
| XML_COMPACT | 26079 | 26642 | +563 | +2.16 | 26071 | 26581 | +510 | +1.96 | 8 | 61 | +53 | +668.16 | 99.97 | 99.77 | -0.20 | 75.24 | 72.57 | -2.67 | -3.55 | 99.85 | 98.99 | -0.86 | 75.16 | 72.05 | -3.11 | -4.14 |
| XML_PRETTY | 23041 | 22126 | -915 | -3.97 | 23018 | 22099 | -919 | -3.99 | 23 | 27 | +4 | +15.26 | 99.90 | 99.88 | -0.02 | 88.88 | 92.99 | +4.11 | +4.62 | 99.69 | 99.08 | -0.61 | 88.74 | 92.45 | +3.72 | +4.19 |
| YAML | 23774 | 24835 | +1061 | +4.46 | 23702 | 24772 | +1070 | +4.52 | 71 | 62 | -9 | -13.01 | 99.70 | 99.75 | +0.05 | 85.45 | 80.70 | -4.75 | -5.55 | 99.48 | 99.17 | -0.31 | 85.30 | 80.32 | -4.98 | -5.84 |

#### 2.8.4 Total Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 34726 | 32985 | -1741 | -5.01 | 34684 | 32969 | -1715 | -4.95 | 42 | 17 | -25 | -59.95 | 99.88 | 99.95 | +0.07 | 76.67 | 82.97 | +6.30 | +8.22 | 99.74 | 99.61 | -0.13 | 76.58 | 82.74 | +6.17 | +8.05 |
| JSON_COMPACT | 32053 | 28263 | -3790 | -11.83 | 31954 | 28232 | -3722 | -11.65 | 99 | 31 | -68 | -68.97 | 99.69 | 99.89 | +0.20 | 86.14 | 99.89 | +13.75 | +15.96 | 99.61 | 99.26 | -0.35 | 86.09 | 99.47 | +13.38 | +15.54 |
| JSON_PRETTY | 31801 | 32573 | +772 | +2.43 | 31795 | 32531 | +736 | +2.31 | 6 | 42 | +36 | +599.75 | 99.98 | 99.87 | -0.11 | 87.24 | 84.40 | -2.85 | -3.26 | 99.86 | 99.31 | -0.55 | 87.16 | 84.02 | -3.14 | -3.60 |
| TOON_DEFAULT | 31606 | 33926 | +2320 | +7.34 | 31597 | 33849 | +2252 | +7.13 | 9 | 78 | +69 | +761.66 | 99.97 | 99.77 | -0.20 | 87.94 | 79.47 | -8.47 | -9.63 | 99.73 | 99.25 | -0.48 | 87.78 | 79.12 | -8.65 | -9.86 |
| XML_COMPACT | 37523 | 34493 | -3030 | -8.07 | 37512 | 34414 | -3098 | -8.26 | 11 | 79 | +68 | +618.89 | 99.97 | 99.77 | -0.20 | 66.68 | 77.43 | +10.75 | +16.12 | 99.85 | 98.99 | -0.86 | 66.60 | 76.91 | +10.31 | +15.48 |
| XML_PRETTY | 36128 | 34144 | -1984 | -5.49 | 36092 | 34103 | -1989 | -5.51 | 36 | 41 | +5 | +13.46 | 99.90 | 99.88 | -0.02 | 71.65 | 78.76 | +7.11 | +9.93 | 99.69 | 99.08 | -0.61 | 71.51 | 78.23 | +6.72 | +9.40 |
| YAML | 33253 | 33531 | +278 | +0.84 | 33153 | 33447 | +294 | +0.89 | 100 | 84 | -16 | -15.93 | 99.70 | 99.75 | +0.05 | 81.84 | 80.88 | -0.97 | -1.18 | 99.48 | 99.17 | -0.31 | 81.69 | 80.49 | -1.20 | -1.48 |

### 2.9 Answer Per Format Breakdown
#### 2.9.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Expected Characters | Output Characters | Correct Characters | Incorrect Characters | Accuracy by Character (%) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 123 | 1 | 0 | 98.93 | 7782 | 7785 | 7776 | 9 | 99.88 |
| CSV | opt | 122 | 2 | 0 | 98.65 | 8451 | 8451 | 8447 | 5 | 99.95 |
| JSON_COMPACT | man | 122 | 2 | 0 | 98.66 | 7782 | 7790 | 7766 | 24 | 99.69 |
| JSON_COMPACT | opt | 122 | 2 | 0 | 98.12 | 8451 | 8452 | 8443 | 9 | 99.89 |
| JSON_PRETTY | man | 123 | 1 | 0 | 99.46 | 7782 | 7782 | 7781 | 1 | 99.98 |
| JSON_PRETTY | opt | 121 | 3 | 0 | 97.85 | 8451 | 8453 | 8442 | 11 | 99.87 |
| TOON_DEFAULT | man | 123 | 1 | 0 | 98.92 | 7782 | 7782 | 7779 | 3 | 99.97 |
| TOON_DEFAULT | opt | 121 | 3 | 0 | 97.85 | 8451 | 8462 | 8442 | 20 | 99.77 |
| XML_COMPACT | man | 123 | 1 | 0 | 99.19 | 7782 | 7782 | 7779 | 3 | 99.97 |
| XML_COMPACT | opt | 119 | 5 | 0 | 96.24 | 8451 | 8455 | 8436 | 19 | 99.77 |
| XML_PRETTY | man | 120 | 4 | 0 | 97.04 | 7782 | 7783 | 7775 | 8 | 99.90 |
| XML_PRETTY | opt | 122 | 2 | 0 | 98.12 | 8451 | 8452 | 8442 | 10 | 99.88 |
| YAML | man | 122 | 2 | 0 | 98.66 | 7782 | 7790 | 7767 | 23 | 99.70 |
| YAML | opt | 120 | 4 | 0 | 97.04 | 8451 | 8462 | 8441 | 21 | 99.75 |

#### 2.9.2 Answers: Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 123 | 123 | 0 | -0.28 | 1 | 1 | 0 | +34.00 | 0 | 0 | 0 | 0.00 | 98.93 | 98.65 | -0.28 |
| JSON_COMPACT | 122 | 121 | -1 | -0.54 | 2 | 3 | +1 | +33.00 | 0 | 0 | 0 | 0.00 | 98.66 | 98.12 | -0.54 |
| JSON_PRETTY | 123 | 121 | -2 | -1.63 | 1 | 3 | +2 | +200.00 | 0 | 0 | 0 | 0.00 | 99.46 | 97.85 | -1.61 |
| TOON_DEFAULT | 123 | 122 | -1 | -1.09 | 1 | 2 | +1 | +134.00 | 0 | 0 | 0 | 0.00 | 98.92 | 97.85 | -1.07 |
| XML_COMPACT | 123 | 119 | -4 | -2.98 | 1 | 5 | +4 | +367.00 | 0 | 0 | 0 | 0.00 | 99.19 | 96.24 | -2.95 |
| XML_PRETTY | 120 | 121 | +1 | +1.12 | 4 | 3 | -1 | -33.50 | 0 | 0 | 0 | 0.00 | 97.04 | 98.12 | +1.08 |
| YAML | 122 | 120 | -2 | -1.64 | 2 | 4 | +2 | +100.00 | 0 | 0 | 0 | 0.00 | 98.66 | 97.04 | -1.62 |

#### 2.9.3 Characters: Mandatory vs Optional Data
| Format | Output Characters Man | Output Characters Opt | Diff | Diff (%) | Correct Characters Man | Correct Characters Opt | Diff | Diff (%) | Incorrect Characters Man | Incorrect Characters Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7785 | 8451 | +666 | +8.55 | 7776 | 8446 | +670 | +8.62 | 9 | 5 | -4 | -48.14 | 99.88 | 99.95 | 0.07 |
| JSON_COMPACT | 7790 | 8452 | +662 | +8.50 | 7766 | 8443 | +677 | +8.72 | 24 | 9 | -15 | -61.11 | 99.69 | 99.89 | 0.20 |
| JSON_PRETTY | 7782 | 8453 | +671 | +8.62 | 7781 | 8443 | +662 | +8.50 | 1 | 10 | +9 | +933.40 | 99.98 | 99.87 | -0.11 |
| TOON_DEFAULT | 7782 | 8462 | +680 | +8.74 | 7779 | 8442 | +663 | +8.52 | 3 | 20 | +17 | +566.67 | 99.97 | 99.77 | -0.20 |
| XML_COMPACT | 7782 | 8455 | +673 | +8.65 | 7779 | 8436 | +657 | +8.45 | 3 | 19 | +16 | +544.43 | 99.97 | 99.77 | -0.20 |
| XML_PRETTY | 7783 | 8452 | +669 | +8.59 | 7775 | 8441 | +666 | +8.57 | 8 | 10 | +2 | +29.16 | 99.90 | 99.88 | -0.02 |
| YAML | 7790 | 8462 | +672 | +8.63 | 7767 | 8442 | +675 | +8.69 | 23 | 20 | -3 | -11.59 | 99.70 | 99.75 | 0.05 |

### 2.10 Accuracy Per Question Category Analysis
#### 2.10.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 98.93 | 100.00 | 98.77 | 98.41 | 96.83 | 99.74 | 37.50 | 28.81 | 20.50 | 12.10 |
| CSV | opt | 98.65 | 100.00 | 100.00 | 98.41 | 93.65 | 99.61 | 37.50 | 29.17 | 20.50 | 11.70 |
| JSON_COMPACT | man | 98.66 | 99.39 | 97.53 | 100.00 | 96.83 | 99.61 | 37.27 | 28.45 | 20.83 | 12.10 |
| JSON_COMPACT | opt | 98.12 | 100.00 | 100.00 | 98.41 | 90.48 | 99.26 | 37.50 | 29.17 | 20.50 | 11.31 |
| JSON_PRETTY | man | 99.46 | 100.00 | 100.00 | 98.41 | 98.41 | 99.86 | 37.50 | 29.17 | 20.50 | 12.30 |
| JSON_PRETTY | opt | 97.85 | 100.00 | 97.53 | 100.00 | 90.48 | 99.31 | 37.50 | 28.45 | 20.83 | 11.31 |
| TOON_DEFAULT | man | 98.92 | 100.00 | 100.00 | 96.83 | 96.83 | 99.73 | 37.50 | 29.17 | 20.17 | 12.10 |
| TOON_DEFAULT | opt | 97.85 | 100.00 | 97.53 | 100.00 | 90.48 | 99.25 | 37.50 | 28.45 | 20.83 | 11.31 |
| XML_COMPACT | man | 99.19 | 100.00 | 98.77 | 98.41 | 98.41 | 99.85 | 37.50 | 28.81 | 20.50 | 12.30 |
| XML_COMPACT | opt | 96.24 | 100.00 | 95.06 | 96.83 | 87.30 | 98.99 | 37.50 | 27.73 | 20.17 | 10.91 |
| XML_PRETTY | man | 97.04 | 100.00 | 91.36 | 96.83 | 96.83 | 99.69 | 37.50 | 26.65 | 20.17 | 12.10 |
| XML_PRETTY | opt | 98.12 | 100.00 | 100.00 | 98.41 | 90.48 | 99.08 | 37.50 | 29.17 | 20.50 | 11.31 |
| YAML | man | 98.66 | 99.39 | 100.00 | 95.24 | 98.41 | 99.48 | 37.27 | 29.17 | 19.84 | 12.30 |
| YAML | opt | 97.04 | 100.00 | 95.06 | 100.00 | 88.89 | 99.17 | 37.50 | 27.73 | 20.83 | 11.11 |

#### 2.10.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| JSON_COMPACT | 99.39 | 100.00 | +0.61 | 37.27 | 37.50 | +0.23 |
| JSON_PRETTY | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| TOON_DEFAULT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| XML_COMPACT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| XML_PRETTY | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| YAML | 99.39 | 100.00 | +0.61 | 37.27 | 37.50 | +0.23 |

#### 2.10.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 98.77 | 100.00 | +1.23 | 28.81 | 29.17 | +0.36 |
| JSON_COMPACT | 97.53 | 100.00 | +2.47 | 28.45 | 29.17 | +0.72 |
| JSON_PRETTY | 100.00 | 97.53 | -2.47 | 29.17 | 28.45 | -0.72 |
| TOON_DEFAULT | 100.00 | 97.53 | -2.47 | 29.17 | 28.45 | -0.72 |
| XML_COMPACT | 98.77 | 95.06 | -3.70 | 28.81 | 27.73 | -1.08 |
| XML_PRETTY | 91.36 | 100.00 | +8.64 | 26.65 | 29.17 | +2.52 |
| YAML | 100.00 | 95.06 | -4.94 | 29.17 | 27.73 | -1.44 |

#### 2.10.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 98.41 | 98.41 | 0.00 | 20.50 | 20.50 | 0.00 |
| JSON_COMPACT | 100.00 | 98.41 | -1.59 | 20.83 | 20.50 | -0.33 |
| JSON_PRETTY | 98.41 | 100.00 | +1.59 | 20.50 | 20.83 | +0.33 |
| TOON_DEFAULT | 96.83 | 100.00 | +3.17 | 20.17 | 20.83 | +0.66 |
| XML_COMPACT | 98.41 | 96.83 | -1.59 | 20.50 | 20.17 | -0.33 |
| XML_PRETTY | 96.83 | 98.41 | +1.59 | 20.17 | 20.50 | +0.33 |
| YAML | 95.24 | 100.00 | +4.76 | 19.84 | 20.83 | +0.99 |

#### 2.10.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 96.83 | 93.65 | -3.17 | 12.10 | 11.70 | -0.40 |
| JSON_COMPACT | 96.83 | 90.48 | -6.35 | 12.10 | 11.31 | -0.79 |
| JSON_PRETTY | 98.41 | 90.48 | -7.93 | 12.30 | 11.31 | -0.99 |
| TOON_DEFAULT | 96.83 | 90.48 | -6.35 | 12.10 | 11.31 | -0.79 |
| XML_COMPACT | 98.41 | 87.30 | -11.11 | 12.30 | 10.91 | -1.39 |
| XML_PRETTY | 96.83 | 90.48 | -6.35 | 12.10 | 11.31 | -0.79 |
| YAML | 98.41 | 88.89 | -9.52 | 12.30 | 11.11 | -1.19 |

### 2.11 Accuracy By Character Per Question Category Analysis
#### 2.11.1 Metrics
| Format | Variant | Accuracy By Character (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc By Char (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 99.88 | 100.00 | 99.85 | 99.70 | 98.77 | 99.74 | 37.50 | 29.13 | 20.77 | 12.35 |
| CSV | opt | 99.95 | 100.00 | 100.00 | 99.71 | 97.37 | 99.61 | 37.50 | 29.17 | 20.77 | 12.17 |
| JSON_COMPACT | man | 99.69 | 99.32 | 99.97 | 100.00 | 98.97 | 99.61 | 37.25 | 29.16 | 20.83 | 12.37 |
| JSON_COMPACT | opt | 99.89 | 100.00 | 100.00 | 99.71 | 94.55 | 99.26 | 37.50 | 29.17 | 20.77 | 11.82 |
| JSON_PRETTY | man | 99.98 | 100.00 | 100.00 | 99.70 | 99.38 | 99.86 | 37.50 | 29.17 | 20.77 | 12.42 |
| JSON_PRETTY | opt | 99.87 | 100.00 | 99.96 | 100.00 | 94.55 | 99.31 | 37.50 | 29.16 | 20.83 | 11.82 |
| TOON_DEFAULT | man | 99.97 | 100.00 | 100.00 | 99.41 | 98.77 | 99.73 | 37.50 | 29.17 | 20.71 | 12.35 |
| TOON_DEFAULT | opt | 99.77 | 100.00 | 99.76 | 100.00 | 94.55 | 99.25 | 37.50 | 29.10 | 20.83 | 11.82 |
| XML_COMPACT | man | 99.97 | 100.00 | 99.97 | 99.70 | 99.38 | 99.85 | 37.50 | 29.16 | 20.77 | 12.42 |
| XML_COMPACT | opt | 99.77 | 100.00 | 99.84 | 99.41 | 93.34 | 98.99 | 37.50 | 29.12 | 20.71 | 11.67 |
| XML_PRETTY | man | 99.90 | 100.00 | 99.88 | 99.41 | 98.77 | 99.69 | 37.50 | 29.13 | 20.71 | 12.35 |
| XML_PRETTY | opt | 99.88 | 100.00 | 100.00 | 98.82 | 94.55 | 99.08 | 37.50 | 29.17 | 20.59 | 11.82 |
| YAML | man | 99.70 | 99.31 | 100.00 | 99.11 | 99.38 | 99.48 | 37.24 | 29.17 | 20.65 | 12.42 |
| YAML | opt | 99.75 | 100.00 | 99.76 | 100.00 | 93.94 | 99.17 | 37.50 | 29.10 | 20.83 | 11.74 |

#### 2.11.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| JSON_COMPACT | 99.32 | 100.00 | +0.68 | 37.25 | 37.50 | +0.25 |
| JSON_PRETTY | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| TOON_DEFAULT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| XML_COMPACT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| XML_PRETTY | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| YAML | 99.31 | 100.00 | +0.69 | 37.24 | 37.50 | +0.26 |

#### 2.11.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 99.85 | 100.00 | +0.15 | 29.13 | 29.17 | +0.04 |
| JSON_COMPACT | 99.97 | 100.00 | +0.03 | 29.16 | 29.17 | +0.01 |
| JSON_PRETTY | 100.00 | 99.96 | -0.04 | 29.17 | 29.16 | -0.01 |
| TOON_DEFAULT | 100.00 | 99.76 | -0.24 | 29.17 | 29.10 | -0.07 |
| XML_COMPACT | 99.97 | 99.84 | -0.13 | 29.16 | 29.12 | -0.04 |
| XML_PRETTY | 99.88 | 100.00 | +0.12 | 29.13 | 29.17 | +0.04 |
| YAML | 100.00 | 99.76 | -0.24 | 29.17 | 29.10 | -0.07 |

#### 2.11.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 99.70 | 99.71 | 0.00 | 20.77 | 20.77 | 0.00 |
| JSON_COMPACT | 100.00 | 99.71 | -0.29 | 20.83 | 20.77 | -0.06 |
| JSON_PRETTY | 99.70 | 100.00 | +0.30 | 20.77 | 20.83 | +0.06 |
| TOON_DEFAULT | 99.41 | 100.00 | +0.59 | 20.71 | 20.83 | +0.12 |
| XML_COMPACT | 99.70 | 99.41 | -0.29 | 20.77 | 20.71 | -0.06 |
| XML_PRETTY | 99.41 | 98.82 | -0.59 | 20.71 | 20.59 | -0.12 |
| YAML | 99.11 | 100.00 | +0.89 | 20.65 | 20.83 | +0.18 |

#### 2.11.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 98.77 | 97.37 | -1.40 | 12.35 | 12.17 | -0.17 |
| JSON_COMPACT | 98.97 | 94.55 | -4.42 | 12.37 | 11.82 | -0.55 |
| JSON_PRETTY | 99.38 | 94.55 | -4.83 | 12.42 | 11.82 | -0.60 |
| TOON_DEFAULT | 98.77 | 94.55 | -4.22 | 12.35 | 11.82 | -0.53 |
| XML_COMPACT | 99.38 | 93.34 | -6.05 | 12.42 | 11.67 | -0.76 |
| XML_PRETTY | 98.77 | 94.55 | -4.22 | 12.35 | 11.82 | -0.53 |
| YAML | 99.38 | 93.94 | -5.44 | 12.42 | 11.74 | -0.68 |

## 3. Appendices

### 3.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Sonnet 4.6
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

- **Report Generated**: 2026-06-21
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code 2.1.80
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_sonnet_4_6/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_sonnet_4_6/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_sonnet_4_6/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)