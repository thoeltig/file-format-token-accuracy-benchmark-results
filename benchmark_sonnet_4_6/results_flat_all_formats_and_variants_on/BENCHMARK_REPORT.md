# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Sonnet 4.6
- **Thinking**: on (medium effort)
- **Data Structure**: flat
- **Formats Tested**: 7 (CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: First iteration

## Executive Summary

This benchmark evaluates token efficiency and information accuracy across 7 file formats using Sonnet 4.6 as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context while a format that accurately conveys information may justify higher token cost.

### Key Findings

1. **TOON_DEFAULT** dominates dense mandatory records with the lowest total tokens (26023) and the highest character accuracy (99.98%). **JSON_COMPACT** dominates sparse optional records with the lowest total tokens (24457) and the top total efficiency score (98.36). The winning format flips entirely depending on whether optional fields are present.
2. **TOON**'s adaptive encoding has a steep cost on sparse data. On mandatory records **TOON** uses only 3965 read tokens which is effectively tied with **CSV** (3922). On optional records **TOON** jumps to 11320 read tokens which is an increase of 185.50% because the layout shifts from the tabular **CSV** style to the **YAML** style key value form to express missing fields. Total tokens grow by 35.40% across the same transition while every other format except **YAML** stays flatter or even shrinks.
3. **JSON_PRETTY** consumes 2.26x the read tokens of **JSON_COMPACT** on mandatory data and still loses 0.53 points of answer accuracy on the same variant. **XML_PRETTY** shows the same pattern against **XML_COMPACT**. Added whitespace produces no measurable comprehension benefit for the model and wastes read budget.
4. Full answer accuracy spans only 1.61 points (97.31% to 98.92%) and character accuracy stays above 99.68% in every cell of the matrix. Format choice should therefore be driven almost entirely by token cost and robustness not by raw correctness.
5. Errors are typically minor character drift rather than structural misreads. **TOON_DEFAULT** on mandatory data produced one fully incorrect answer but only 2 wrong characters out of 7782. **CSV** on mandatory data produced 3 incorrect answers with 25 wrong characters. The gap between answer accuracy and character accuracy shows that failures are almost always off by a digit or a single word due to a hallucinated or missed field.
6. Filtering is the weakest and most format sensitive question category. On mandatory data filtering accuracy ranges from 87.30% (**YAML**) to 98.41% (**JSON_COMPACT**) which is a spread of 11.11 points. Field retrieval and structure awareness spread by less than 5 points on the same data. Formats without explicit per field labels on every row (**YAML**, **CSV**) perform worst here when records are dense.
7. Aggregation regresses on optional data for every format. All seven formats lose between 4.22 and 5.45 points of character accuracy on aggregation questions when moving from mandatory to optional. The regression is driven by null handling rather than by any single format's weakness and represents a cross cutting risk for numeric workloads with sparse inputs.

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
| JSON_PRETTY ≈ 279s | CSV ≈ 3922 | JSON_PRETTY ≈ 201 | JSON_PRETTY ≈ 20803 | JSON_PRETTY ≈ 21004 | TOON_DEFAULT ≈ 26023 | JSON_COMPACT ≈ 98.92% | TOON_DEFAULT ≈ 98 | JSON_PRETTY ≈ 91 | TOON_DEFAULT ≈ 96 | TOON_DEFAULT ≈ 99.98% | TOON_DEFAULT ≈ 99 | JSON_PRETTY ≈ 92 | TOON_DEFAULT ≈ 96 |
| JSON_COMPACT (+8.79%) | TOON_DEFAULT (+1.10%) | TOON_DEFAULT (+26.95%) | TOON_DEFAULT (+4.81%) | TOON_DEFAULT (+5.02%) | CSV (+9.26%) | TOON_DEFAULT (0.00%) | CSV (-0.77%) | TOON_DEFAULT (-3.88%) | CSV (-6.70%) | XML_PRETTY (-0.01%) | CSV (-0.06%) | TOON_DEFAULT (-4.21%) | CSV (-5.93%) |
| YAML (+10.70%) | JSON_COMPACT (+58.01%) | CSV (+41.96%) | YAML (+10.35%) | YAML (+10.68%) | JSON_COMPACT (+17.55%) | XML_COMPACT (0.00%) | JSON_COMPACT (-7.27%) | YAML (-9.68%) | JSON_COMPACT (-10.94%) | JSON_PRETTY (-0.02%) | JSON_COMPACT (-7.29%) | YAML (-8.99%) | JSON_COMPACT (-10.92%) |
| XML_PRETTY (+13.39%) | YAML (+141.69%) | YAML (+45.11%) | JSON_COMPACT (+15.83%) | JSON_COMPACT (+16.14%) | YAML (+25.76%) | XML_PRETTY (-0.26%) | YAML (-18.87%) | JSON_COMPACT (-13.35%) | YAML (-16.98%) | YAML (-0.03%) | YAML (-17.86%) | JSON_COMPACT (-13.64%) | YAML (-15.95%) |
| TOON_DEFAULT (+20.69%) | XML_COMPACT (+191.79%) | XML_COMPACT (+45.44%) | XML_PRETTY (+16.22%) | XML_PRETTY (+16.51%) | JSON_PRETTY (+34.64%) | JSON_PRETTY (-0.53%) | XML_COMPACT (-24.37%) | XML_PRETTY (-13.85%) | JSON_PRETTY (-21.95%) | JSON_COMPACT (-0.10%) | XML_COMPACT (-24.37%) | XML_PRETTY (-13.88%) | JSON_PRETTY (-21.43%) |
| CSV (+21.24%) | XML_PRETTY (+233.68%) | XML_PRETTY (+46.77%) | CSV (+16.45%) | CSV (+16.69%) | XML_PRETTY (+44.33%) | CSV (-1.34%) | XML_PRETTY (-29.89%) | CSV (-14.80%) | XML_PRETTY (-27.79%) | XML_COMPACT (-0.27%) | XML_PRETTY (-29.51%) | CSV (-14.25%) | XML_PRETTY (-27.42%) |
| XML_COMPACT (+21.68%) | JSON_PRETTY (+257.83%) | JSON_COMPACT (+48.59%) | XML_COMPACT (+26.13%) | XML_COMPACT (+26.32%) | XML_COMPACT (+45.93%) | YAML (-1.34%) | JSON_PRETTY (-33.16%) | XML_COMPACT (-22.00%) | XML_COMPACT (-28.61%) | CSV (-0.30%) | JSON_PRETTY (-32.58%) | XML_COMPACT (-22.32%) | XML_COMPACT (-28.59%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XML_PRETTY ≈ 265s | CSV ≈ 3633 | CSV ≈ 287 | JSON_COMPACT ≈ 18494 | JSON_COMPACT ≈ 18788 | JSON_COMPACT ≈ 24457 | JSON_PRETTY ≈ 98.12% | CSV ≈ 98 | JSON_COMPACT ≈ 98 | JSON_COMPACT ≈ 98 | JSON_COMPACT ≈ 99.88% | CSV ≈ 100 | JSON_COMPACT ≈ 100 | JSON_COMPACT ≈ 100 |
| TOON_DEFAULT (+15.66%) | JSON_COMPACT (+56.04%) | YAML (+1.74%) | XML_PRETTY (+12.59%) | XML_PRETTY (+12.40%) | CSV (+28.66%) | XML_COMPACT (-0.27%) | JSON_COMPACT (-6.45%) | XML_PRETTY (-8.71%) | CSV (-16.51%) | JSON_PRETTY (0.00%) | JSON_COMPACT (-6.51%) | XML_PRETTY (-8.58%) | CSV (-16.08%) |
| XML_COMPACT (+23.75%) | XML_COMPACT (+116.10%) | JSON_PRETTY (+2.32%) | TOON_DEFAULT (+27.68%) | TOON_DEFAULT (+27.29%) | XML_PRETTY (+35.49%) | JSON_COMPACT (-0.54%) | XML_COMPACT (-13.38%) | TOON_DEFAULT (-19.26%) | XML_PRETTY (-20.21%) | CSV (-0.01%) | XML_COMPACT (-13.57%) | TOON_DEFAULT (-18.88%) | XML_PRETTY (-19.90%) |
| JSON_PRETTY (+27.32%) | TOON_DEFAULT (+211.59%) | JSON_COMPACT (+2.56%) | JSON_PRETTY (+38.32%) | JSON_PRETTY (+37.72%) | XML_COMPACT (+38.17%) | XML_PRETTY (-0.54%) | TOON_DEFAULT (-24.95%) | JSON_PRETTY (-26.12%) | XML_COMPACT (-21.55%) | TOON_DEFAULT (-0.01%) | TOON_DEFAULT (-24.62%) | JSON_PRETTY (-26.08%) | XML_COMPACT (-21.48%) |
| YAML (+28.11%) | YAML (+217.15%) | XML_COMPACT (+2.79%) | XML_COMPACT (+38.68%) | XML_COMPACT (+38.08%) | TOON_DEFAULT (+44.07%) | TOON_DEFAULT (-0.67%) | YAML (-25.70%) | XML_COMPACT (-26.56%) | TOON_DEFAULT (-25.18%) | XML_PRETTY (-0.01%) | YAML (-25.34%) | XML_COMPACT (-26.41%) | TOON_DEFAULT (-24.72%) |
| JSON_COMPACT (+34.05%) | XML_PRETTY (+230.80%) | XML_PRETTY (+3.02%) | YAML (+39.18%) | YAML (+38.55%) | YAML (+53.55%) | CSV (-0.81%) | XML_PRETTY (-27.14%) | YAML (-27.26%) | YAML (-30.67%) | XML_COMPACT (-0.11%) | XML_PRETTY (-26.85%) | YAML (-26.75%) | YAML (-30.11%) |
| CSV (+42.39%) | JSON_PRETTY (+261.08%) | TOON_DEFAULT (+5.69%) | CSV (+48.96%) | CSV (+48.15%) | JSON_PRETTY (+59.43%) | YAML (-0.81%) | JSON_PRETTY (-30.35%) | CSV (-34.00%) | JSON_PRETTY (-33.47%) | YAML (-0.13%) | JSON_PRETTY (-30.37%) | CSV (-33.31%) | JSON_PRETTY (-33.32%) |


#### 2.1.2 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 100.00% | CSV ≈ 100.00% | JSON_COMPACT ≈ 98.41% | JSON_PRETTY ≈ 100.00% |
| TOON_DEFAULT (0.00%) | JSON_COMPACT (0.00%) | JSON_PRETTY (-1.59%) | XML_COMPACT (0.00%) |
| XML_PRETTY (0.00%) | TOON_DEFAULT (0.00%) | XML_COMPACT (-3.17%) | TOON_DEFAULT (-0.79%) |
| YAML (0.00%) | XML_COMPACT (0.00%) | TOON_DEFAULT (-3.97%) | CSV (-1.59%) |
| CSV (-0.61%) | XML_PRETTY (0.00%) | XML_PRETTY (-4.76%) | XML_PRETTY (-1.59%) |
| JSON_COMPACT (-0.61%) | YAML (0.00%) | CSV (-9.52%) | YAML (-1.59%) |
| XML_COMPACT (-0.61%) | JSON_PRETTY (-4.94%) | YAML (-11.11%) | JSON_COMPACT (-3.17%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| CSV ≈ 100.00% | TOON_DEFAULT ≈ 99.38% | JSON_COMPACT ≈ 100.00% | JSON_COMPACT ≈ 90.48% |
| JSON_COMPACT (0.00%) | JSON_PRETTY (-0.62%) | JSON_PRETTY (0.00%) | JSON_PRETTY (0.00%) |
| JSON_PRETTY (0.00%) | XML_PRETTY (-1.85%) | XML_COMPACT (0.00%) | XML_COMPACT (0.00%) |
| TOON_DEFAULT (0.00%) | YAML (-1.85%) | CSV (-1.59%) | XML_PRETTY (0.00%) |
| XML_COMPACT (0.00%) | CSV (-1.85%) | XML_PRETTY (-1.59%) | CSV (-1.59%) |
| XML_PRETTY (0.00%) | XML_COMPACT (-1.85%) | YAML (-1.59%) | YAML (-1.59%) |
| YAML (0.00%) | JSON_COMPACT (-3.09%) | TOON_DEFAULT (-2.38%) | TOON_DEFAULT (-2.38%) |


#### 2.1.3 Category Accuracy By Character Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 100.00% | CSV ≈ 100.00% | JSON_COMPACT ≈ 99.70% | JSON_PRETTY ≈ 100.00% |
| TOON_DEFAULT (0.00%) | JSON_COMPACT (0.00%) | JSON_PRETTY (-0.30%) | TOON_DEFAULT (-0.31%) |
| XML_PRETTY (0.00%) | TOON_DEFAULT (0.00%) | XML_COMPACT (-0.60%) | XML_COMPACT (-0.41%) |
| YAML (0.00%) | XML_COMPACT (0.00%) | TOON_DEFAULT (-0.74%) | CSV (-0.62%) |
| JSON_COMPACT (-0.23%) | XML_PRETTY (0.00%) | XML_PRETTY (-0.89%) | XML_PRETTY (-0.62%) |
| CSV (-0.69%) | YAML (0.00%) | CSV (-1.79%) | YAML (-0.62%) |
| XML_COMPACT (-0.69%) | JSON_PRETTY (-0.06%) | YAML (-2.08%) | JSON_COMPACT (-1.23%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| CSV ≈ 100.00% | JSON_PRETTY ≈ 99.98% | JSON_COMPACT ≈ 100.00% | JSON_COMPACT ≈ 94.55% |
| JSON_COMPACT (0.00%) | CSV (-0.01%) | JSON_PRETTY (0.00%) | JSON_PRETTY (0.00%) |
| JSON_PRETTY (0.00%) | JSON_COMPACT (-0.02%) | XML_COMPACT (0.00%) | XML_COMPACT (0.00%) |
| TOON_DEFAULT (0.00%) | XML_PRETTY (-0.02%) | CSV (-0.29%) | XML_PRETTY (0.00%) |
| XML_COMPACT (0.00%) | TOON_DEFAULT (-0.02%) | XML_PRETTY (-0.29%) | TOON_DEFAULT (-0.11%) |
| XML_PRETTY (0.00%) | XML_COMPACT (-0.22%) | YAML (-0.29%) | CSV (-0.41%) |
| YAML (0.00%) | YAML (-0.22%) | TOON_DEFAULT (-0.44%) | YAML (-0.61%) |


#### 2.1.4 Conclusion

- On mandatory records **TOON_DEFAULT** is the recommended format. It posts the top total efficiency score (95.67), the top character accuracy (99.98%) and reads in only 3965 tokens which is within 1.10% of **CSV**. **CSV** itself is a reasonable second choice when read cost matters more than filtering accuracy but its 88.89% filtering score on mandatory data is a tangible weakness.
- On optional records **JSON_COMPACT** is the recommended format. It posts the top total efficiency score (98.36), the lowest total tokens (24457) and the lowest output tokens (18788). **TOON_DEFAULT** falls to fourth place on this variant because its encoding switch inflates read tokens by 185.50% relative to its own mandatory baseline.
- Pretty-printed **JSON** and **XML** should be avoided in token-sensitive pipelines regardless of variant. They cost 2x to 3x more read tokens than their compact siblings and deliver no accuracy advantage.
- If the density of the incoming data cannot be predicted **JSON_COMPACT** is the safer default. Its penalty on mandatory data is bounded (10.94% below **TOON** on total efficiency score) while **TOON**'s penalty on optional data is severe (25.18% below **JSON_COMPACT** on the same metric).
- Aggregation over optional numeric fields should be treated as the riskiest operation in this benchmark. Every format regressed by at least 4 points on that subcategory and no format choice solved the problem.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total | Accuracy By Character (%) | Useful Read Tokens (Acc By Char) | Wasted Read Tokens (Acc By Char) | Useful Output Tokens (Acc By Char) | Wasted Output Tokens (Acc By Char) | Eff Score Read (Acc By Char) | Eff Score Output (Acc By Char) | Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 3922 | 24510 | 28432 | 2.57 | 195.36 | 97.58 | 3827 | 95 | 23917 | 593 | 97.43 | 77.31 | 89.26 | 99.68 | 3909 | 13 | 24432 | 78 | 98.83 | 78.71 | 90.66 |
| CSV | opt | 3633 | 27835 | 31468 | 2.64 | 222.16 | 97.31 | 3535 | 98 | 27086 | 749 | 98.17 | 64.91 | 82.13 | 99.87 | 3628 | 5 | 27799 | 36 | 99.88 | 66.62 | 83.83 |
| JSON_COMPACT | man | 6197 | 24394 | 30591 | 3.21 | 194.32 | 98.92 | 6130 | 67 | 24131 | 263 | 91.05 | 78.63 | 85.21 | 99.88 | 6190 | 7 | 24365 | 29 | 91.69 | 79.27 | 85.85 |
| JSON_COMPACT | opt | 5669 | 18788 | 24457 | 3.26 | 149.15 | 97.58 | 5532 | 137 | 18334 | 455 | 91.84 | 98.35 | 98.36 | 99.88 | 5662 | 7 | 18766 | 23 | 93.38 | 99.88 | 99.90 |
| JSON_PRETTY | man | 14034 | 21004 | 35038 | 1.72 | 167.76 | 98.39 | 13808 | 226 | 20666 | 338 | 65.63 | 90.74 | 74.67 | 99.96 | 14028 | 6 | 20995 | 8 | 66.67 | 91.79 | 75.72 |
| JSON_PRETTY | opt | 13118 | 25875 | 38993 | 1.71 | 206.30 | 98.12 | 12871 | 247 | 25388 | 486 | 68.38 | 72.66 | 65.44 | 99.88 | 13102 | 16 | 25844 | 31 | 69.55 | 73.83 | 66.61 |
| TOON_DEFAULT | man | 3965 | 22058 | 26023 | 2.56 | 175.83 | 98.92 | 3922 | 43 | 21820 | 238 | 98.19 | 87.22 | 95.67 | 99.98 | 3964 | 1 | 22054 | 4 | 98.89 | 87.93 | 96.38 |
| TOON_DEFAULT | opt | 11320 | 23916 | 35236 | 1.73 | 190.43 | 97.45 | 11031 | 289 | 23306 | 610 | 73.68 | 79.41 | 73.59 | 99.87 | 11305 | 15 | 23885 | 31 | 75.29 | 81.02 | 75.21 |
| XML_COMPACT | man | 11444 | 26531 | 37975 | 2.42 | 211.60 | 98.92 | 11320 | 124 | 26244 | 287 | 74.26 | 70.78 | 68.30 | 99.71 | 11411 | 33 | 26454 | 77 | 74.79 | 71.30 | 68.83 |
| XML_COMPACT | opt | 7851 | 25943 | 33794 | 3.26 | 206.84 | 97.85 | 7682 | 169 | 25385 | 558 | 85.04 | 72.23 | 77.16 | 99.77 | 7833 | 18 | 25883 | 60 | 86.32 | 73.51 | 78.44 |
| XML_PRETTY | man | 13087 | 24471 | 37558 | 2.39 | 194.97 | 98.66 | 12912 | 175 | 24143 | 328 | 68.83 | 78.18 | 69.08 | 99.97 | 13083 | 4 | 24464 | 7 | 69.71 | 79.05 | 69.95 |
| XML_PRETTY | opt | 12018 | 21119 | 33137 | 2.41 | 167.93 | 97.58 | 11727 | 291 | 20608 | 511 | 71.53 | 89.78 | 78.49 | 99.87 | 12002 | 16 | 21091 | 27 | 73.06 | 91.31 | 80.01 |
| YAML | man | 9479 | 23247 | 32726 | 2.20 | 185.13 | 97.58 | 9250 | 229 | 22685 | 563 | 79.65 | 81.96 | 79.43 | 99.95 | 9474 | 5 | 23236 | 12 | 81.23 | 83.54 | 81.01 |
| YAML | opt | 11522 | 26031 | 37553 | 1.68 | 207.58 | 97.31 | 11212 | 310 | 25331 | 700 | 72.94 | 71.54 | 68.19 | 99.75 | 11493 | 29 | 25966 | 65 | 74.57 | 73.17 | 69.82 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 3922 | 3633 | -289 | -7.37 | 285 | 287 | +2 | +0.70 | 24225 | 27548 | +3323 | +13.72 | 24510 | 27835 | +3325 | +13.57 | 28432 | 31468 | +3036 | +10.68 |
| JSON_COMPACT | 6197 | 5669 | -528 | -8.52 | 299 | 295 | -4 | -1.34 | 24096 | 18494 | -5602 | -23.25 | 24394 | 18788 | -5606 | -22.98 | 30591 | 24457 | -6134 | -20.05 |
| JSON_PRETTY | 14034 | 13118 | -916 | -6.53 | 201 | 294 | +93 | +46.27 | 20803 | 25581 | +4778 | +22.97 | 21004 | 25875 | +4871 | +23.19 | 35038 | 38993 | +3955 | +11.29 |
| TOON_DEFAULT | 3965 | 11320 | +7355 | +185.50 | 255 | 303 | +48 | +18.82 | 21803 | 23613 | +1810 | +8.30 | 22058 | 23916 | +1858 | +8.42 | 26023 | 35236 | +9213 | +35.40 |
| XML_COMPACT | 11444 | 7851 | -3593 | -31.40 | 292 | 295 | +3 | +1.03 | 26239 | 25648 | -591 | -2.25 | 26531 | 25943 | -588 | -2.22 | 37975 | 33794 | -4181 | -11.01 |
| XML_PRETTY | 13087 | 12018 | -1069 | -8.17 | 295 | 296 | +1 | +0.34 | 24176 | 20823 | -3353 | -13.87 | 24471 | 21119 | -3352 | -13.70 | 37558 | 33137 | -4421 | -11.77 |
| YAML | 9479 | 11522 | +2043 | +21.55 | 292 | 292 | 0 | 0.00 | 22956 | 25740 | +2784 | +12.13 | 23247 | 26031 | +2784 | +11.98 | 32726 | 37553 | +4827 | +14.75 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Before Write (ms) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) | Output (ms) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 17 | 230.71 | 0.55 | 270125 | 68588 | 0.35 | 553.13 | 68605 | 231.06 | 442.61 | 338713 |
| CSV | opt | 24 | 151.38 | 0.77 | 309458 | 68350 | 0.40 | 551.21 | 68374 | 151.78 | 441.12 | 377808 |
| JSON_COMPACT | man | 12 | 516.42 | 0.39 | 239555 | 64384 | 0.37 | 519.23 | 64396 | 516.79 | 415.46 | 303939 |
| JSON_COMPACT | opt | 14 | 404.93 | 0.45 | 288416 | 67274 | 0.28 | 542.53 | 67288 | 405.20 | 434.12 | 355690 |
| JSON_PRETTY | man | 18 | 779.67 | 0.58 | 211833 | 67547 | 0.31 | 544.73 | 67565 | 779.98 | 435.90 | 279380 |
| JSON_PRETTY | opt | 23 | 570.35 | 0.74 | 275091 | 62754 | 0.41 | 506.08 | 62777 | 570.76 | 405.01 | 337845 |
| TOON_DEFAULT | man | 39 | 101.67 | 1.26 | 276682 | 60505 | 0.36 | 487.94 | 60544 | 102.03 | 390.61 | 337186 |
| TOON_DEFAULT | opt | 21 | 539.05 | 0.68 | 244726 | 62181 | 0.38 | 501.46 | 62202 | 539.43 | 401.30 | 306906 |
| XML_COMPACT | man | 32 | 357.63 | 1.03 | 276020 | 63935 | 0.41 | 515.60 | 63967 | 358.04 | 412.69 | 339955 |
| XML_COMPACT | opt | 91 | 86.28 | 2.94 | 269443 | 58905 | 0.44 | 475.04 | 58996 | 86.71 | 380.62 | 328348 |
| XML_PRETTY | man | 69 | 189.67 | 2.23 | 246315 | 70475 | 0.34 | 568.35 | 70544 | 190.01 | 455.12 | 316790 |
| XML_PRETTY | opt | 27 | 445.11 | 0.87 | 207095 | 58246 | 0.36 | 469.73 | 58273 | 445.47 | 375.95 | 265342 |
| YAML | man | 27 | 351.07 | 0.87 | 237535 | 71728 | 0.32 | 578.45 | 71755 | 351.39 | 462.94 | 309263 |
| YAML | opt | 30 | 384.07 | 0.97 | 278291 | 61643 | 0.42 | 497.12 | 61673 | 384.49 | 397.89 | 339934 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Before Write Man (s) | Output Before Write Opt (s) | Diff (s) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 17 | 24 | +7 | +41.18 | 270.13 | 309.46 | +39.33 | +14.56 | 68.59 | 68.35 | -0.24 | -0.35 | 68.60 | 68.37 | -0.23 | -0.34 | 338.71 | 377.81 | +39.10 | +11.54 |
| JSON_COMPACT | 12 | 14 | +2 | +16.67 | 239.55 | 288.42 | +48.86 | +20.40 | 64.38 | 67.27 | +2.89 | +4.49 | 64.40 | 67.29 | +2.89 | +4.49 | 303.94 | 355.69 | +51.75 | +17.03 |
| JSON_PRETTY | 18 | 23 | +5 | +27.78 | 211.83 | 275.09 | +63.26 | +29.86 | 67.55 | 62.75 | -4.79 | -7.10 | 67.57 | 62.78 | -4.79 | -7.09 | 279.38 | 337.85 | +58.47 | +20.93 |
| TOON_DEFAULT | 39 | 21 | -18 | -46.15 | 276.68 | 244.73 | -31.96 | -11.55 | 60.50 | 62.18 | +1.68 | +2.77 | 60.54 | 62.20 | +1.66 | +2.74 | 337.19 | 306.91 | -30.28 | -8.98 |
| XML_COMPACT | 32 | 91 | +59 | +184.38 | 276.02 | 269.44 | -6.58 | -2.38 | 63.93 | 58.90 | -5.03 | -7.87 | 63.97 | 59.00 | -4.97 | -7.77 | 339.96 | 328.35 | -11.61 | -3.41 |
| XML_PRETTY | 69 | 27 | -42 | -60.87 | 246.32 | 207.10 | -39.22 | -15.92 | 70.47 | 58.25 | -12.23 | -17.35 | 70.54 | 58.27 | -12.27 | -17.39 | 316.79 | 265.34 | -51.45 | -16.24 |
| YAML | 27 | 30 | +3 | +11.11 | 237.53 | 278.29 | +40.76 | +17.16 | 71.73 | 61.64 | -10.09 | -14.06 | 71.76 | 61.67 | -10.08 | -14.05 | 309.26 | 339.93 | +30.67 | +9.92 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token | Info / Read Token (Acc By Char) | Info / Output Token (Acc By Char) | Info / Total Token (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 2.57 | 5.75 | 126.52 | 2.49 | 0.40 | 0.34 | 2.54 | 0.41 | 0.35 |
| CSV | opt | 2.64 | 5.76 | 117.19 | 2.68 | 0.35 | 0.31 | 2.75 | 0.36 | 0.32 |
| JSON_COMPACT | man | 3.21 | 9.09 | 199.90 | 1.60 | 0.41 | 0.32 | 1.61 | 0.41 | 0.33 |
| JSON_COMPACT | opt | 3.26 | 8.98 | 182.87 | 1.72 | 0.52 | 0.40 | 1.76 | 0.53 | 0.41 |
| JSON_PRETTY | man | 1.72 | 20.58 | 452.71 | 0.70 | 0.47 | 0.28 | 0.71 | 0.48 | 0.28 |
| JSON_PRETTY | opt | 1.71 | 20.79 | 423.16 | 0.75 | 0.38 | 0.25 | 0.76 | 0.39 | 0.26 |
| TOON_DEFAULT | man | 2.56 | 5.81 | 127.90 | 2.50 | 0.45 | 0.38 | 2.52 | 0.45 | 0.38 |
| TOON_DEFAULT | opt | 1.73 | 17.94 | 365.16 | 0.86 | 0.41 | 0.28 | 0.88 | 0.42 | 0.28 |
| XML_COMPACT | man | 2.42 | 16.78 | 369.16 | 0.86 | 0.37 | 0.26 | 0.87 | 0.38 | 0.26 |
| XML_COMPACT | opt | 3.26 | 12.44 | 253.26 | 1.25 | 0.38 | 0.29 | 1.27 | 0.39 | 0.30 |
| XML_PRETTY | man | 2.39 | 19.19 | 422.16 | 0.75 | 0.40 | 0.26 | 0.76 | 0.41 | 0.27 |
| XML_PRETTY | opt | 2.41 | 19.05 | 387.68 | 0.81 | 0.46 | 0.29 | 0.83 | 0.47 | 0.30 |
| YAML | man | 2.20 | 13.90 | 305.77 | 1.03 | 0.42 | 0.30 | 1.05 | 0.43 | 0.31 |
| YAML | opt | 1.68 | 18.26 | 371.68 | 0.85 | 0.37 | 0.26 | 0.87 | 0.38 | 0.27 |

#### 2.5.2 Characters And Values: Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 2.57 | 2.64 | +0.06 | +2.41 | 5.75 | 5.76 | +0.01 | +0.12 | 126.52 | 117.19 | -9.32 | -7.37 |
| JSON_COMPACT | 3.21 | 3.26 | +0.05 | +1.46 | 9.09 | 8.98 | -0.10 | -1.13 | 199.90 | 182.87 | -17.03 | -8.52 |
| JSON_PRETTY | 1.72 | 1.71 | -0.01 | -0.70 | 20.58 | 20.79 | +0.21 | +1.03 | 452.71 | 423.16 | -29.55 | -6.53 |
| TOON_DEFAULT | 2.56 | 1.73 | -0.83 | -32.34 | 5.81 | 17.94 | +12.13 | +208.57 | 127.90 | 365.16 | +237.26 | +185.50 |
| XML_COMPACT | 2.42 | 3.26 | +0.84 | +34.80 | 16.78 | 12.44 | -4.34 | -25.85 | 369.16 | 253.26 | -115.90 | -31.40 |
| XML_PRETTY | 2.39 | 2.41 | +0.02 | +0.75 | 19.19 | 19.05 | -0.14 | -0.75 | 422.16 | 387.68 | -34.48 | -8.17 |
| YAML | 2.20 | 1.68 | -0.52 | -23.56 | 13.90 | 18.26 | +4.36 | +31.38 | 305.77 | 371.68 | +65.90 | +21.55 |

#### 2.5.3 Information: Mandatory vs Optional
| Format | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 2.49 | 2.68 | +0.19 | +7.68 | 0.40 | 0.35 | -0.05 | -12.06 | 0.34 | 0.31 | -0.03 | -9.91 |
| JSON_COMPACT | 1.60 | 1.72 | +0.13 | +7.83 | 0.41 | 0.52 | +0.11 | +27.83 | 0.32 | 0.40 | +0.08 | +23.53 |
| JSON_PRETTY | 0.70 | 0.75 | +0.05 | +6.70 | 0.47 | 0.38 | -0.09 | -19.02 | 0.28 | 0.25 | -0.03 | -10.32 |
| TOON_DEFAULT | 2.50 | 0.86 | -1.63 | -65.49 | 0.45 | 0.41 | -0.04 | -9.15 | 0.38 | 0.28 | -0.10 | -27.11 |
| XML_COMPACT | 0.86 | 1.25 | +0.38 | +44.21 | 0.37 | 0.38 | 0.00 | 0.00 | 0.26 | 0.29 | +0.03 | +11.54 |
| XML_PRETTY | 0.75 | 0.81 | +0.06 | +7.69 | 0.40 | 0.46 | +0.06 | +14.64 | 0.26 | 0.29 | +0.03 | +11.79 |
| YAML | 1.03 | 0.85 | -0.18 | -17.88 | 0.42 | 0.37 | -0.05 | -10.95 | 0.30 | 0.26 | -0.04 | -13.09 |

#### 2.5.4 Information (Accuracy By Character): Mandatory vs Optional
| Format | Info / Read Token (Acc By Char) Man | Info / Read Token (Acc By Char) Opt | Diff | Diff (%) | Info / Output Token (Acc By Char) Man | Info / Output Token (Acc By Char)  Opt | Diff | Diff (%) | Info / Total Token (Acc By Char) Man | Info / Total Token (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 2.54 | 2.75 | +0.21 | +8.14 | 0.41 | 0.36 | -0.05 | -11.79 | 0.35 | 0.32 | -0.03 | -9.69 |
| JSON_COMPACT | 1.61 | 1.76 | +0.15 | +9.31 | 0.41 | 0.53 | +0.12 | +30.07 | 0.33 | 0.41 | +0.08 | +25.15 |
| JSON_PRETTY | 0.71 | 0.76 | +0.05 | +6.88 | 0.48 | 0.39 | -0.09 | -18.91 | 0.28 | 0.26 | -0.03 | -10.18 |
| TOON_DEFAULT | 2.52 | 0.88 | -1.64 | -65.03 | 0.45 | 0.42 | -0.04 | -7.73 | 0.38 | 0.28 | -0.10 | -26.30 |
| XML_COMPACT | 0.87 | 1.27 | +0.40 | +45.92 | 0.38 | 0.39 | +0.01 | +2.39 | 0.26 | 0.30 | +0.03 | +12.17 |
| XML_PRETTY | 0.76 | 0.83 | +0.07 | +8.77 | 0.41 | 0.47 | +0.06 | +15.65 | 0.27 | 0.30 | +0.03 | +13.16 |
| YAML | 1.05 | 0.87 | -0.19 | -17.84 | 0.43 | 0.38 | -0.05 | -10.93 | 0.31 | 0.27 | -0.04 | -12.79 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 3922 | 3827 | 95 | 24510 | 23917 | 593 | 28432 | 27744 | 688 | 97.58 | 97.43 | 77.31 | 89.26 | 97.26 | 97.22 | 77.10 | 89.05 |
| CSV | opt | 3633 | 3535 | 98 | 27835 | 27086 | 749 | 31468 | 30622 | 846 | 97.31 | 98.17 | 64.91 | 82.13 | 97.56 | 98.34 | 65.08 | 82.29 |
| JSON_COMPACT | man | 6197 | 6130 | 67 | 24394 | 24131 | 263 | 30591 | 30261 | 330 | 98.92 | 91.05 | 78.63 | 85.21 | 99.04 | 91.13 | 78.71 | 85.29 |
| JSON_COMPACT | opt | 5669 | 5532 | 137 | 18788 | 18334 | 455 | 24457 | 23865 | 592 | 97.58 | 91.84 | 98.35 | 98.36 | 97.73 | 91.94 | 98.45 | 98.46 |
| JSON_PRETTY | man | 14034 | 13808 | 226 | 21004 | 20666 | 338 | 35038 | 34474 | 564 | 98.39 | 65.63 | 90.74 | 74.67 | 97.90 | 65.30 | 90.42 | 74.35 |
| JSON_PRETTY | opt | 13118 | 12871 | 247 | 25875 | 25388 | 486 | 38993 | 38260 | 733 | 98.12 | 68.38 | 72.66 | 65.44 | 98.45 | 68.60 | 72.88 | 65.66 |
| TOON_DEFAULT | man | 3965 | 3922 | 43 | 22058 | 21820 | 238 | 26023 | 25742 | 281 | 98.92 | 98.19 | 87.22 | 95.67 | 98.74 | 98.07 | 87.10 | 95.55 |
| TOON_DEFAULT | opt | 11320 | 11031 | 289 | 23916 | 23306 | 610 | 35236 | 34338 | 899 | 97.45 | 73.68 | 79.41 | 73.59 | 97.84 | 73.94 | 79.67 | 73.85 |
| XML_COMPACT | man | 11444 | 11320 | 124 | 26531 | 26244 | 287 | 37975 | 37565 | 410 | 98.92 | 74.26 | 70.78 | 68.30 | 98.78 | 74.17 | 70.68 | 68.21 |
| XML_COMPACT | opt | 7851 | 7682 | 169 | 25943 | 25385 | 558 | 33794 | 33067 | 727 | 97.85 | 85.04 | 72.23 | 77.16 | 98.09 | 85.20 | 72.39 | 77.32 |
| XML_PRETTY | man | 13087 | 12912 | 175 | 24471 | 24143 | 328 | 37558 | 37055 | 503 | 98.66 | 68.83 | 78.18 | 69.08 | 98.48 | 68.71 | 78.06 | 68.96 |
| XML_PRETTY | opt | 12018 | 11727 | 291 | 21119 | 20608 | 511 | 33137 | 32335 | 802 | 97.58 | 71.53 | 89.78 | 78.49 | 97.76 | 71.65 | 89.90 | 78.61 |
| YAML | man | 9479 | 9250 | 229 | 23247 | 22685 | 563 | 32726 | 31934 | 792 | 97.58 | 79.65 | 81.96 | 79.43 | 97.15 | 79.37 | 81.67 | 79.14 |
| YAML | opt | 11522 | 11212 | 310 | 26031 | 25331 | 700 | 37553 | 36543 | 1010 | 97.31 | 72.94 | 71.54 | 68.19 | 97.56 | 73.11 | 71.71 | 68.36 |

#### 2.6.2 Read Tokens: Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 3922 | 3633 | -289 | -7.37 | 3827 | 3535 | -292 | -7.63 | 95 | 98 | +3 | +2.96 | 97.58 | 97.31 | -0.27 | 97.43 | 98.17 | +0.75 | +0.76 | 97.26 | 97.56 | +0.30 | 97.22 | 98.34 | +1.12 | +1.16 |
| JSON_COMPACT | 6197 | 5669 | -528 | -8.52 | 6130 | 5532 | -598 | -9.76 | 67 | 137 | +70 | +104.87 | 98.92 | 97.58 | -1.34 | 91.05 | 91.84 | +0.79 | +0.87 | 99.04 | 97.73 | -1.31 | 91.13 | 91.94 | +0.81 | +0.89 |
| JSON_PRETTY | 14034 | 13118 | -916 | -6.53 | 13808 | 12871 | -937 | -6.78 | 226 | 247 | +21 | +9.15 | 98.39 | 98.12 | -0.27 | 65.63 | 68.38 | +2.75 | +4.19 | 97.90 | 98.45 | +0.55 | 65.30 | 68.60 | +3.30 | +5.05 |
| TOON_DEFAULT | 3965 | 11320 | +7355 | +185.50 | 3922 | 11031 | +7109 | +181.26 | 43 | 289 | +246 | +571.72 | 98.92 | 97.45 | -1.47 | 98.19 | 73.68 | -24.51 | -24.96 | 98.74 | 97.84 | -0.90 | 98.07 | 73.94 | -24.13 | -24.60 |
| XML_COMPACT | 11444 | 7851 | -3593 | -31.40 | 11320 | 7682 | -3638 | -32.14 | 124 | 169 | +45 | +36.45 | 98.92 | 97.85 | -1.07 | 74.26 | 85.04 | +10.78 | +14.52 | 98.78 | 98.09 | -0.69 | 74.17 | 85.20 | +11.03 | +14.88 |
| XML_PRETTY | 13087 | 12018 | -1069 | -8.17 | 12912 | 11728 | -1184 | -9.17 | 175 | 290 | +115 | +65.98 | 98.66 | 97.58 | -1.08 | 68.83 | 71.53 | +2.70 | +3.92 | 98.48 | 97.76 | -0.72 | 68.71 | 71.65 | +2.94 | +4.28 |
| YAML | 9479 | 11522 | +2043 | +21.55 | 9250 | 11212 | +1962 | +21.22 | 229 | 310 | +81 | +35.17 | 97.58 | 97.31 | -0.27 | 79.65 | 72.94 | -6.71 | -8.43 | 97.15 | 97.56 | +0.41 | 79.37 | 73.11 | -6.26 | -7.89 |

#### 2.6.3 Output Tokens: Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 24510 | 27835 | +3325 | +13.57 | 23917 | 27086 | +3169 | +13.25 | 593 | 749 | +156 | +26.24 | 97.58 | 97.31 | -0.27 | 77.31 | 64.91 | -12.40 | -16.04 | 97.26 | 97.56 | +0.30 | 77.10 | 65.08 | -12.02 | -15.60 |
| JSON_COMPACT | 24394 | 18788 | -5606 | -22.98 | 24131 | 18334 | -5797 | -24.02 | 263 | 454 | +191 | +72.71 | 98.92 | 97.58 | -1.34 | 78.63 | 98.35 | +19.72 | +25.08 | 99.04 | 97.73 | -1.31 | 78.71 | 98.45 | +19.74 | +25.07 |
| JSON_PRETTY | 21004 | 25875 | +4871 | +23.19 | 20666 | 25389 | +4723 | +22.85 | 338 | 486 | +148 | +43.87 | 98.39 | 98.12 | -0.27 | 90.74 | 72.66 | -18.09 | -19.93 | 97.90 | 98.45 | +0.55 | 90.42 | 72.88 | -17.54 | -19.40 |
| TOON_DEFAULT | 22058 | 23916 | +1858 | +8.42 | 21820 | 23307 | +1487 | +6.81 | 238 | 610 | +372 | +156.15 | 98.92 | 97.45 | -1.47 | 87.22 | 79.41 | -7.81 | -8.96 | 98.74 | 97.84 | -0.90 | 87.10 | 79.67 | -7.43 | -8.53 |
| XML_COMPACT | 26531 | 25943 | -588 | -2.22 | 26244 | 25384 | -860 | -3.28 | 287 | 558 | +271 | +94.51 | 98.92 | 97.85 | -1.07 | 70.78 | 72.23 | +1.45 | +2.05 | 98.78 | 98.09 | -0.69 | 70.68 | 72.39 | +1.70 | +2.41 |
| XML_PRETTY | 24471 | 21119 | -3352 | -13.70 | 24143 | 20608 | -3535 | -14.64 | 328 | 511 | +183 | +55.84 | 98.66 | 97.58 | -1.08 | 78.18 | 89.78 | +11.60 | +14.84 | 98.48 | 97.76 | -0.72 | 78.06 | 89.90 | +11.85 | +15.17 |
| YAML | 23247 | 26031 | +2784 | +11.98 | 22685 | 25331 | +2646 | +11.67 | 563 | 701 | +138 | +24.45 | 97.58 | 97.31 | -0.27 | 81.96 | 71.54 | -10.42 | -12.71 | 97.15 | 97.56 | +0.41 | 81.67 | 71.71 | -9.96 | -12.20 |

#### 2.6.4 Total Tokens: Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 28432 | 31468 | +3036 | +10.68 | 27744 | 30622 | +2878 | +10.37 | 688 | 846 | +158 | +23.03 | 97.58 | 97.31 | -0.27 | 89.26 | 82.13 | -7.13 | -7.99 | 97.26 | 97.56 | +0.30 | 89.05 | 82.29 | -6.75 | -7.58 |
| JSON_COMPACT | 30591 | 24457 | -6134 | -20.05 | 30261 | 23866 | -6395 | -21.13 | 330 | 591 | +261 | +79.24 | 98.92 | 97.58 | -1.34 | 85.21 | 98.36 | +13.15 | +15.44 | 99.04 | 97.73 | -1.31 | 85.29 | 98.46 | +13.17 | +15.45 |
| JSON_PRETTY | 35038 | 38993 | +3955 | +11.29 | 34474 | 38260 | +3786 | +10.98 | 564 | 733 | +169 | +29.96 | 98.39 | 98.12 | -0.27 | 74.67 | 65.44 | -9.24 | -12.37 | 97.90 | 98.45 | +0.55 | 74.35 | 65.66 | -8.69 | -11.69 |
| TOON_DEFAULT | 26023 | 35236 | +9213 | +35.40 | 25742 | 34338 | +8596 | +33.39 | 281 | 898 | +617 | +219.74 | 98.92 | 97.45 | -1.47 | 95.67 | 73.59 | -22.08 | -23.08 | 98.74 | 97.84 | -0.90 | 95.55 | 73.85 | -21.70 | -22.71 |
| XML_COMPACT | 37975 | 33794 | -4181 | -11.01 | 37565 | 33067 | -4498 | -11.97 | 410 | 726 | +316 | +77.18 | 98.92 | 97.85 | -1.07 | 68.30 | 77.16 | +8.86 | +12.98 | 98.78 | 98.09 | -0.69 | 68.21 | 77.32 | +9.12 | +13.37 |
| XML_PRETTY | 37558 | 33137 | -4421 | -11.77 | 37055 | 32335 | -4720 | -12.74 | 503 | 802 | +299 | +59.37 | 98.66 | 97.58 | -1.08 | 69.08 | 78.49 | +9.41 | +13.61 | 98.48 | 97.76 | -0.72 | 68.96 | 78.61 | +9.64 | +13.99 |
| YAML | 32726 | 37553 | +4827 | +14.75 | 31934 | 36543 | +4609 | +14.43 | 792 | 1010 | +218 | +27.55 | 97.58 | 97.31 | -0.27 | 79.43 | 68.19 | -11.23 | -14.14 | 97.15 | 97.56 | +0.41 | 79.14 | 68.36 | -10.78 | -13.62 |

### 2.7 Token Utilization Efficiency (Accuracy by Character)
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy by Character (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy by Character (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 3922 | 3909 | 13 | 24510 | 24432 | 78 | 28432 | 28341 | 91 | 99.68 | 98.83 | 78.71 | 90.66 | 99.23 | 98.53 | 78.41 | 90.36 |
| CSV | opt | 3633 | 3628 | 5 | 27835 | 27799 | 36 | 31468 | 31427 | 41 | 99.87 | 99.88 | 66.62 | 83.83 | 99.20 | 99.43 | 66.17 | 83.39 |
| JSON_COMPACT | man | 6197 | 6190 | 7 | 24394 | 24365 | 29 | 30591 | 30555 | 37 | 99.88 | 91.69 | 79.27 | 85.85 | 99.70 | 91.57 | 79.15 | 85.73 |
| JSON_COMPACT | opt | 5669 | 5662 | 7 | 18788 | 18766 | 23 | 24457 | 24428 | 29 | 99.88 | 93.38 | 99.88 | 99.90 | 99.31 | 93.00 | 99.50 | 99.52 |
| JSON_PRETTY | man | 14034 | 14028 | 6 | 21004 | 20995 | 8 | 35038 | 35024 | 14 | 99.96 | 66.67 | 91.79 | 75.72 | 99.86 | 66.61 | 91.72 | 75.65 |
| JSON_PRETTY | opt | 13118 | 13102 | 16 | 25875 | 25844 | 31 | 38993 | 38946 | 47 | 99.88 | 69.55 | 73.83 | 66.61 | 99.31 | 69.17 | 73.45 | 66.23 |
| TOON_DEFAULT | man | 3965 | 3964 | 1 | 22058 | 22054 | 4 | 26023 | 26018 | 5 | 99.98 | 98.89 | 87.93 | 96.38 | 99.75 | 98.74 | 87.78 | 96.22 |
| TOON_DEFAULT | opt | 11320 | 11305 | 15 | 23916 | 23885 | 31 | 35236 | 35190 | 46 | 99.87 | 75.29 | 81.02 | 75.21 | 99.20 | 74.85 | 80.58 | 74.76 |
| XML_COMPACT | man | 11444 | 11411 | 33 | 26531 | 26454 | 77 | 37975 | 37865 | 110 | 99.71 | 74.79 | 71.30 | 68.83 | 99.51 | 74.66 | 71.17 | 68.69 |
| XML_COMPACT | opt | 7851 | 7833 | 18 | 25943 | 25883 | 60 | 33794 | 33716 | 78 | 99.77 | 86.32 | 73.51 | 78.44 | 99.25 | 85.98 | 73.16 | 78.10 |
| XML_PRETTY | man | 13087 | 13083 | 4 | 24471 | 24464 | 7 | 37558 | 37547 | 11 | 99.97 | 69.71 | 79.05 | 69.95 | 99.68 | 69.51 | 78.86 | 69.76 |
| XML_PRETTY | opt | 12018 | 12002 | 16 | 21119 | 21091 | 27 | 33137 | 33094 | 43 | 99.87 | 73.06 | 91.31 | 80.01 | 99.25 | 72.65 | 90.90 | 79.60 |
| YAML | man | 9479 | 9474 | 5 | 23247 | 23236 | 12 | 32726 | 32710 | 16 | 99.95 | 81.23 | 83.54 | 81.01 | 99.43 | 80.89 | 83.19 | 80.66 |
| YAML | opt | 11522 | 11493 | 29 | 26031 | 25966 | 65 | 37553 | 37459 | 94 | 99.75 | 74.57 | 73.17 | 69.82 | 99.11 | 74.14 | 72.74 | 69.39 |

#### 2.7.2 Read Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 3922 | 3633 | -289 | -7.37 | 3909 | 3628 | -281 | -7.19 | 13 | 5 | -8 | -60.21 | 99.68 | 99.87 | +0.19 | 98.83 | 99.88 | +1.05 | +1.06 | 99.23 | 99.20 | -0.03 | 98.53 | 99.43 | +0.91 | +0.92 |
| JSON_COMPACT | 6197 | 5669 | -528 | -8.52 | 6190 | 5663 | -527 | -8.52 | 7 | 6 | -1 | -9.04 | 99.88 | 99.88 | 0.00 | 91.69 | 93.38 | +1.69 | +1.84 | 99.70 | 99.31 | -0.39 | 91.57 | 93.00 | +1.43 | +1.56 |
| JSON_PRETTY | 14034 | 13118 | -916 | -6.53 | 14028 | 13102 | -926 | -6.60 | 6 | 16 | +10 | +168.80 | 99.96 | 99.88 | -0.08 | 66.67 | 69.55 | +2.88 | +4.32 | 99.86 | 99.31 | -0.55 | 66.61 | 69.17 | +2.56 | +3.85 |
| TOON_DEFAULT | 3965 | 11320 | +7355 | +185.50 | 3964 | 11305 | +7341 | +185.19 | 1 | 15 | +14 | +1392.30 | 99.98 | 99.87 | -0.11 | 98.89 | 75.29 | -23.60 | -23.86 | 99.75 | 99.20 | -0.55 | 98.74 | 74.85 | -23.89 | -24.20 |
| XML_COMPACT | 11444 | 7851 | -3593 | -31.40 | 11411 | 7833 | -3578 | -31.35 | 33 | 18 | -15 | -45.85 | 99.71 | 99.77 | +0.06 | 74.79 | 86.32 | +11.53 | +15.42 | 99.51 | 99.25 | -0.26 | 74.66 | 85.98 | +11.32 | +15.16 |
| XML_PRETTY | 13087 | 12018 | -1069 | -8.17 | 13083 | 12002 | -1081 | -8.26 | 4 | 16 | +12 | +292.42 | 99.97 | 99.87 | -0.10 | 69.71 | 73.06 | +3.35 | +4.81 | 99.68 | 99.25 | -0.43 | 69.51 | 72.65 | +3.13 | +4.51 |
| YAML | 9479 | 11522 | +2043 | +21.55 | 9474 | 11493 | +2019 | +21.31 | 5 | 29 | +24 | +481.32 | 99.95 | 99.75 | -0.20 | 81.23 | 74.57 | -6.67 | -8.21 | 99.43 | 99.11 | -0.32 | 80.89 | 74.14 | -6.75 | -8.34 |

#### 2.7.3 Output Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 24510 | 27835 | +3325 | +13.57 | 24432 | 27799 | +3367 | +13.78 | 78 | 36 | -42 | -54.16 | 99.68 | 99.87 | +0.19 | 78.71 | 66.62 | -12.10 | -15.37 | 99.23 | 99.20 | -0.03 | 78.41 | 66.17 | -12.25 | -15.62 |
| JSON_COMPACT | 24394 | 18788 | -5606 | -22.98 | 24365 | 18766 | -5599 | -22.98 | 29 | 22 | -7 | -23.20 | 99.88 | 99.88 | 0.00 | 79.27 | 99.88 | +20.61 | +26.00 | 99.70 | 99.31 | -0.39 | 79.15 | 99.50 | +20.35 | +25.71 |
| JSON_PRETTY | 21004 | 25875 | +4871 | +23.19 | 20995 | 25843 | +4848 | +23.09 | 8 | 31 | +23 | +283.11 | 99.96 | 99.88 | -0.08 | 91.79 | 73.83 | -17.96 | -19.57 | 99.86 | 99.31 | -0.55 | 91.72 | 73.45 | -18.27 | -19.92 |
| TOON_DEFAULT | 22058 | 23916 | +1858 | +8.42 | 22054 | 23885 | +1831 | +8.30 | 4 | 31 | +27 | +666.98 | 99.98 | 99.87 | -0.11 | 87.93 | 81.02 | -6.91 | -7.85 | 99.75 | 99.20 | -0.55 | 87.78 | 80.58 | -7.20 | -8.20 |
| XML_COMPACT | 26531 | 25943 | -588 | -2.22 | 26454 | 25883 | -571 | -2.16 | 77 | 60 | -17 | -22.43 | 99.71 | 99.77 | +0.06 | 71.30 | 73.51 | +2.20 | +3.09 | 99.51 | 99.25 | -0.26 | 71.17 | 73.16 | +1.99 | +2.80 |
| XML_PRETTY | 24471 | 21119 | -3352 | -13.70 | 24464 | 21092 | -3372 | -13.79 | 7 | 27 | +20 | +287.33 | 99.97 | 99.87 | -0.10 | 79.05 | 91.31 | +12.26 | +15.51 | 99.68 | 99.25 | -0.43 | 78.86 | 90.90 | +12.04 | +15.27 |
| YAML | 23247 | 26031 | +2784 | +11.98 | 23236 | 25967 | +2731 | +11.75 | 12 | 65 | +53 | +445.45 | 99.95 | 99.75 | -0.20 | 83.54 | 73.17 | -10.37 | -12.41 | 99.43 | 99.11 | -0.32 | 83.19 | 72.74 | -10.45 | -12.56 |

#### 2.7.4 Total Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 28432 | 31468 | +3036 | +10.68 | 28341 | 31427 | +3086 | +10.89 | 91 | 41 | -50 | -55.03 | 99.68 | 99.87 | +0.19 | 90.66 | 83.83 | -6.83 | -7.53 | 99.23 | 99.20 | -0.03 | 90.36 | 83.39 | -6.97 | -7.72 |
| JSON_COMPACT | 30591 | 24457 | -6134 | -20.05 | 30555 | 24428 | -6127 | -20.05 | 37 | 30 | -7 | -19.89 | 99.88 | 99.88 | 0.00 | 85.85 | 99.90 | +14.05 | +16.36 | 99.70 | 99.31 | -0.39 | 85.73 | 99.52 | +13.79 | +16.08 |
| JSON_PRETTY | 35038 | 38993 | +3955 | +11.29 | 35024 | 38946 | +3922 | +11.20 | 14 | 47 | +33 | +234.11 | 99.96 | 99.88 | -0.08 | 75.72 | 66.61 | -9.11 | -12.03 | 99.86 | 99.31 | -0.55 | 75.65 | 66.23 | -9.42 | -12.46 |
| TOON_DEFAULT | 26023 | 35236 | +9213 | +35.40 | 26018 | 35191 | +9173 | +35.25 | 5 | 46 | +41 | +812.04 | 99.98 | 99.87 | -0.11 | 96.38 | 75.21 | -21.17 | -21.97 | 99.75 | 99.20 | -0.55 | 96.22 | 74.76 | -21.47 | -22.31 |
| XML_COMPACT | 37975 | 33794 | -4181 | -11.01 | 37865 | 33716 | -4149 | -10.96 | 110 | 78 | -32 | -29.46 | 99.71 | 99.77 | +0.06 | 68.83 | 78.44 | +9.62 | +13.97 | 99.51 | 99.25 | -0.26 | 68.69 | 78.10 | +9.40 | +13.69 |
| XML_PRETTY | 37558 | 33137 | -4421 | -11.77 | 37547 | 33094 | -4453 | -11.86 | 11 | 43 | +32 | +289.19 | 99.97 | 99.87 | -0.10 | 69.95 | 80.01 | +10.06 | +14.38 | 99.68 | 99.25 | -0.43 | 69.76 | 79.60 | +9.84 | +14.10 |
| YAML | 32726 | 37553 | +4827 | +14.75 | 32710 | 37459 | +4749 | +14.52 | 16 | 94 | +78 | +484.50 | 99.95 | 99.75 | -0.20 | 81.01 | 69.82 | -11.19 | -13.81 | 99.43 | 99.11 | -0.32 | 80.66 | 69.39 | -11.27 | -13.97 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Expected Characters | Output Characters | Correct Characters | Incorrect Characters | Accuracy by Character (%) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 121 | 3 | 0 | 97.58 | 7782 | 7790 | 7765 | 25 | 99.68 |
| CSV | opt | 121 | 3 | 0 | 97.31 | 8451 | 8452 | 8441 | 11 | 99.87 |
| JSON_COMPACT | man | 123 | 1 | 0 | 98.92 | 7782 | 7787 | 7778 | 9 | 99.88 |
| JSON_COMPACT | opt | 121 | 3 | 0 | 97.58 | 8451 | 8452 | 8442 | 10 | 99.88 |
| JSON_PRETTY | man | 122 | 2 | 0 | 98.39 | 7782 | 7782 | 7779 | 3 | 99.96 |
| JSON_PRETTY | opt | 122 | 2 | 0 | 98.12 | 8451 | 8452 | 8442 | 10 | 99.88 |
| TOON_DEFAULT | man | 123 | 1 | 0 | 98.92 | 7782 | 7782 | 7780 | 2 | 99.98 |
| TOON_DEFAULT | opt | 121 | 3 | 0 | 97.45 | 8451 | 8453 | 8442 | 11 | 99.87 |
| XML_COMPACT | man | 123 | 1 | 0 | 98.92 | 7782 | 7790 | 7767 | 23 | 99.71 |
| XML_COMPACT | opt | 121 | 3 | 0 | 97.85 | 8451 | 8462 | 8442 | 20 | 99.77 |
| XML_PRETTY | man | 122 | 2 | 0 | 98.66 | 7782 | 7782 | 7780 | 2 | 99.97 |
| XML_PRETTY | opt | 121 | 3 | 0 | 97.58 | 8451 | 8452 | 8441 | 11 | 99.87 |
| YAML | man | 121 | 3 | 0 | 97.58 | 7782 | 7782 | 7778 | 4 | 99.95 |
| YAML | opt | 121 | 3 | 0 | 97.31 | 8451 | 8462 | 8441 | 21 | 99.75 |

#### 2.8.2 Answers: Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 121 | 121 | 0 | -0.27 | 3 | 3 | 0 | +11.00 | 0 | 0 | 0 | 0.00 | 97.58 | 97.31 | -0.27 |
| JSON_COMPACT | 123 | 121 | -2 | -1.36 | 1 | 3 | +2 | +167.00 | 0 | 0 | 0 | 0.00 | 98.92 | 97.58 | -1.34 |
| JSON_PRETTY | 122 | 122 | 0 | -0.27 | 2 | 2 | 0 | +16.50 | 0 | 0 | 0 | 0.00 | 98.39 | 98.12 | -0.27 |
| TOON_DEFAULT | 123 | 121 | -2 | -1.50 | 1 | 3 | +2 | +184.00 | 0 | 0 | 0 | 0.00 | 98.92 | 97.45 | -1.47 |
| XML_COMPACT | 123 | 122 | -1 | -1.09 | 1 | 2 | +1 | +134.00 | 0 | 0 | 0 | 0.00 | 98.92 | 97.85 | -1.07 |
| XML_PRETTY | 122 | 121 | -1 | -1.09 | 2 | 3 | +1 | +66.50 | 0 | 0 | 0 | 0.00 | 98.66 | 97.58 | -1.08 |
| YAML | 121 | 121 | 0 | -0.27 | 3 | 3 | 0 | +11.00 | 0 | 0 | 0 | 0.00 | 97.58 | 97.31 | -0.27 |

#### 2.8.3 Characters: Mandatory vs Optional Data
| Format | Output Characters Man | Output Characters Opt | Diff | Diff (%) | Correct Characters Man | Correct Characters Opt | Diff | Diff (%) | Incorrect Characters Man | Incorrect Characters Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7790 | 8452 | +662 | +8.50 | 7765 | 8441 | +676 | +8.70 | 25 | 11 | -14 | -54.67 | 99.68 | 99.87 | 0.19 |
| JSON_COMPACT | 7787 | 8452 | +665 | +8.54 | 7778 | 8442 | +664 | +8.53 | 9 | 10 | +1 | +11.11 | 99.88 | 99.88 | 0.00 |
| JSON_PRETTY | 7782 | 8452 | +670 | +8.61 | 7779 | 8443 | +664 | +8.53 | 3 | 9 | +6 | +211.13 | 99.96 | 99.88 | -0.08 |
| TOON_DEFAULT | 7782 | 8453 | +671 | +8.63 | 7780 | 8442 | +662 | +8.50 | 2 | 12 | +10 | +483.30 | 99.98 | 99.87 | -0.11 |
| XML_COMPACT | 7790 | 8462 | +672 | +8.63 | 7767 | 8442 | +675 | +8.69 | 23 | 20 | -3 | -14.49 | 99.71 | 99.77 | 0.06 |
| XML_PRETTY | 7782 | 8452 | +670 | +8.61 | 7780 | 8442 | +662 | +8.50 | 2 | 10 | +8 | +416.70 | 99.97 | 99.87 | -0.10 |
| YAML | 7782 | 8462 | +680 | +8.74 | 7778 | 8441 | +663 | +8.52 | 4 | 21 | +17 | +433.32 | 99.95 | 99.75 | -0.20 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 97.58 | 99.39 | 100.00 | 88.89 | 98.41 | 99.23 | 37.27 | 29.17 | 18.52 | 12.30 |
| CSV | opt | 97.31 | 100.00 | 97.53 | 98.41 | 88.89 | 99.20 | 37.50 | 28.45 | 20.50 | 11.11 |
| JSON_COMPACT | man | 98.92 | 99.39 | 100.00 | 98.41 | 96.83 | 99.70 | 37.27 | 29.17 | 20.50 | 12.10 |
| JSON_COMPACT | opt | 97.58 | 100.00 | 96.30 | 100.00 | 90.48 | 99.31 | 37.50 | 28.09 | 20.83 | 11.31 |
| JSON_PRETTY | man | 98.39 | 100.00 | 95.06 | 96.83 | 100.00 | 99.86 | 37.50 | 27.73 | 20.17 | 12.50 |
| JSON_PRETTY | opt | 98.12 | 100.00 | 98.77 | 100.00 | 90.48 | 99.31 | 37.50 | 28.81 | 20.83 | 11.31 |
| TOON_DEFAULT | man | 98.92 | 100.00 | 100.00 | 94.45 | 99.21 | 99.75 | 37.50 | 29.17 | 19.67 | 12.40 |
| TOON_DEFAULT | opt | 97.45 | 100.00 | 99.38 | 97.62 | 88.10 | 99.20 | 37.50 | 28.99 | 20.34 | 11.01 |
| XML_COMPACT | man | 98.92 | 99.39 | 100.00 | 95.24 | 100.00 | 99.51 | 37.27 | 29.17 | 19.84 | 12.50 |
| XML_COMPACT | opt | 97.85 | 100.00 | 97.53 | 100.00 | 90.48 | 99.25 | 37.50 | 28.45 | 20.83 | 11.31 |
| XML_PRETTY | man | 98.66 | 100.00 | 100.00 | 93.65 | 98.41 | 99.68 | 37.50 | 29.17 | 19.51 | 12.30 |
| XML_PRETTY | opt | 97.58 | 100.00 | 97.53 | 98.41 | 90.48 | 99.25 | 37.50 | 28.45 | 20.50 | 11.31 |
| YAML | man | 97.58 | 100.00 | 100.00 | 87.30 | 98.41 | 99.43 | 37.50 | 29.17 | 18.18 | 12.30 |
| YAML | opt | 97.31 | 100.00 | 97.53 | 98.41 | 88.89 | 99.11 | 37.50 | 28.45 | 20.50 | 11.11 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 99.39 | 100.00 | +0.61 | 37.27 | 37.50 | +0.23 |
| JSON_COMPACT | 99.39 | 100.00 | +0.61 | 37.27 | 37.50 | +0.23 |
| JSON_PRETTY | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| TOON_DEFAULT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| XML_COMPACT | 99.39 | 100.00 | +0.61 | 37.27 | 37.50 | +0.23 |
| XML_PRETTY | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| YAML | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 100.00 | 97.53 | -2.47 | 29.17 | 28.45 | -0.72 |
| JSON_COMPACT | 100.00 | 96.30 | -3.70 | 29.17 | 28.09 | -1.08 |
| JSON_PRETTY | 95.06 | 98.77 | +3.71 | 27.73 | 28.81 | +1.08 |
| TOON_DEFAULT | 100.00 | 99.38 | -0.62 | 29.17 | 28.99 | -0.18 |
| XML_COMPACT | 100.00 | 97.53 | -2.47 | 29.17 | 28.45 | -0.72 |
| XML_PRETTY | 100.00 | 97.53 | -2.47 | 29.17 | 28.45 | -0.72 |
| YAML | 100.00 | 97.53 | -2.47 | 29.17 | 28.45 | -0.72 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 88.89 | 98.41 | +9.52 | 18.52 | 20.50 | +1.98 |
| JSON_COMPACT | 98.41 | 100.00 | +1.59 | 20.50 | 20.83 | +0.33 |
| JSON_PRETTY | 96.83 | 100.00 | +3.17 | 20.17 | 20.83 | +0.66 |
| TOON_DEFAULT | 94.45 | 97.62 | +3.17 | 19.67 | 20.34 | +0.66 |
| XML_COMPACT | 95.24 | 100.00 | +4.76 | 19.84 | 20.83 | +0.99 |
| XML_PRETTY | 93.65 | 98.41 | +4.76 | 19.51 | 20.50 | +0.99 |
| YAML | 87.30 | 98.41 | +11.11 | 18.18 | 20.50 | +2.32 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 98.41 | 88.89 | -9.52 | 12.30 | 11.11 | -1.19 |
| JSON_COMPACT | 96.83 | 90.48 | -6.35 | 12.10 | 11.31 | -0.79 |
| JSON_PRETTY | 100.00 | 90.48 | -9.52 | 12.50 | 11.31 | -1.19 |
| TOON_DEFAULT | 99.21 | 88.10 | -11.11 | 12.40 | 11.01 | -1.39 |
| XML_COMPACT | 100.00 | 90.48 | -9.52 | 12.50 | 11.31 | -1.19 |
| XML_PRETTY | 98.41 | 90.48 | -7.93 | 12.30 | 11.31 | -0.99 |
| YAML | 98.41 | 88.89 | -9.52 | 12.30 | 11.11 | -1.19 |

### 2.10 Accuracy By Character Per Question Category Analysis
#### 2.10.1 Metrics
| Format | Variant | Accuracy By Character (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc By Char (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 99.68 | 99.31 | 100.00 | 97.91 | 99.38 | 99.23 | 37.24 | 29.17 | 20.40 | 12.42 |
| CSV | opt | 99.87 | 100.00 | 99.98 | 99.71 | 94.14 | 99.20 | 37.50 | 29.16 | 20.77 | 11.77 |
| JSON_COMPACT | man | 99.88 | 99.77 | 100.00 | 99.70 | 98.77 | 99.70 | 37.42 | 29.17 | 20.77 | 12.35 |
| JSON_COMPACT | opt | 99.88 | 100.00 | 99.97 | 100.00 | 94.55 | 99.31 | 37.50 | 29.16 | 20.83 | 11.82 |
| JSON_PRETTY | man | 99.96 | 100.00 | 99.94 | 99.40 | 100.00 | 99.86 | 37.50 | 29.15 | 20.71 | 12.50 |
| JSON_PRETTY | opt | 99.88 | 100.00 | 99.98 | 100.00 | 94.55 | 99.31 | 37.50 | 29.16 | 20.83 | 11.82 |
| TOON_DEFAULT | man | 99.98 | 100.00 | 100.00 | 98.96 | 99.69 | 99.75 | 37.50 | 29.17 | 20.62 | 12.46 |
| TOON_DEFAULT | opt | 99.87 | 100.00 | 99.96 | 99.56 | 94.44 | 99.20 | 37.50 | 29.16 | 20.74 | 11.81 |
| XML_COMPACT | man | 99.71 | 99.31 | 100.00 | 99.11 | 99.59 | 99.51 | 37.24 | 29.17 | 20.65 | 12.45 |
| XML_COMPACT | opt | 99.77 | 100.00 | 99.76 | 100.00 | 94.55 | 99.25 | 37.50 | 29.10 | 20.83 | 11.82 |
| XML_PRETTY | man | 99.97 | 100.00 | 100.00 | 98.81 | 99.38 | 99.68 | 37.50 | 29.17 | 20.59 | 12.42 |
| XML_PRETTY | opt | 99.87 | 100.00 | 99.97 | 99.71 | 94.55 | 99.25 | 37.50 | 29.16 | 20.77 | 11.82 |
| YAML | man | 99.95 | 100.00 | 100.00 | 97.62 | 99.38 | 99.43 | 37.50 | 29.17 | 20.34 | 12.42 |
| YAML | opt | 99.75 | 100.00 | 99.76 | 99.71 | 93.94 | 99.11 | 37.50 | 29.10 | 20.77 | 11.74 |

#### 2.10.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 99.31 | 100.00 | +0.69 | 37.24 | 37.50 | +0.26 |
| JSON_COMPACT | 99.77 | 100.00 | +0.23 | 37.42 | 37.50 | +0.08 |
| JSON_PRETTY | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| TOON_DEFAULT | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| XML_COMPACT | 99.31 | 100.00 | +0.69 | 37.24 | 37.50 | +0.26 |
| XML_PRETTY | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |
| YAML | 100.00 | 100.00 | 0.00 | 37.50 | 37.50 | 0.00 |

#### 2.10.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 100.00 | 99.98 | -0.02 | 29.17 | 29.16 | -0.01 |
| JSON_COMPACT | 100.00 | 99.97 | -0.03 | 29.17 | 29.16 | -0.01 |
| JSON_PRETTY | 99.94 | 99.98 | +0.04 | 29.15 | 29.16 | +0.01 |
| TOON_DEFAULT | 100.00 | 99.96 | -0.04 | 29.17 | 29.16 | -0.01 |
| XML_COMPACT | 100.00 | 99.76 | -0.24 | 29.17 | 29.10 | -0.07 |
| XML_PRETTY | 100.00 | 99.97 | -0.03 | 29.17 | 29.16 | -0.01 |
| YAML | 100.00 | 99.76 | -0.24 | 29.17 | 29.10 | -0.07 |

#### 2.10.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 97.91 | 99.71 | +1.79 | 20.40 | 20.77 | +0.37 |
| JSON_COMPACT | 99.70 | 100.00 | +0.30 | 20.77 | 20.83 | +0.06 |
| JSON_PRETTY | 99.40 | 100.00 | +0.60 | 20.71 | 20.83 | +0.12 |
| TOON_DEFAULT | 98.96 | 99.56 | +0.60 | 20.62 | 20.74 | +0.12 |
| XML_COMPACT | 99.11 | 100.00 | +0.89 | 20.65 | 20.83 | +0.18 |
| XML_PRETTY | 98.81 | 99.71 | +0.90 | 20.59 | 20.77 | +0.18 |
| YAML | 97.62 | 99.71 | +2.09 | 20.34 | 20.77 | +0.43 |

#### 2.10.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 99.38 | 94.14 | -5.24 | 12.42 | 11.77 | -0.65 |
| JSON_COMPACT | 98.77 | 94.55 | -4.22 | 12.35 | 11.82 | -0.53 |
| JSON_PRETTY | 100.00 | 94.55 | -5.45 | 12.50 | 11.82 | -0.68 |
| TOON_DEFAULT | 99.69 | 94.44 | -5.25 | 12.46 | 11.81 | -0.66 |
| XML_COMPACT | 99.59 | 94.55 | -5.04 | 12.45 | 11.82 | -0.63 |
| XML_PRETTY | 99.38 | 94.55 | -4.83 | 12.42 | 11.82 | -0.60 |
| YAML | 99.38 | 93.94 | -5.44 | 12.42 | 11.74 | -0.68 |

## 3. Appendices

### 3.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Sonnet 4.6
- **Thinking**: on (medium effort)
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

- **Report Generated**: 2026-04-20
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code 2.1.80
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_sonnet_4_6/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_sonnet_4_6/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_sonnet_4_6/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)