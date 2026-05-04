# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: off
- **Data Structure**: nested
- **Formats Tested**: 6 (JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: First iteration

## Executive Summary

This benchmark evaluates token efficiency and information accuracy across 6 file formats using Claude Haiku 4.5 (claude-haiku-4-5-20251001) as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context while a format that accurately conveys information may justify higher token cost.

### Key Findings

1. **JSON_COMPACT** is the most token-efficient format. It consumes the fewest read tokens across both variants (10,163 mandatory, 9,645 optional) because it encodes 2.22 to 2.25 characters per token. This is roughly 50% fewer tokens than **XML_PRETTY** which is the most expensive format. Combined with top-tier accuracy it delivers the best information value per read token (0.74 mandatory, 0.79 optional).
2. **XML_COMPACT** achieves the highest overall efficiency score for mandatory data (83.85) by combining the best accuracy by answer (75.81%) with the lowest output token count (5,295 tokens, 40.24 tokens per answer). Its character-per-token ratio of 2.55 is the highest of all formats which indicates that XML tag structure compresses well during tokenization and the dense data might also reduce the necessary reasoning the model needs to do which reduces the output tokens.
3. Most incorrect answers are near-misses and not completely wrong. Accuracy ranges from 69.62% to 75.81% for complete answers (mandatory) but from 94.18% to 96.43% by character measurement. This consistent ~20 percentage point gap shows that the majority of errors are character-level deviations in multiple places rather than entirely wrong responses.
4. **YAML** suffers the largest accuracy collapse on optional data. It drops 9.41 percentage points in accuracy by answer and 16.97 percentage points in field retrieval alone when moving from mandatory to optional data. No other format exceeds a 2.53 percentage point drop in overall accuracy between variants.
5. Pretty-printed formats deliver no accuracy benefit for their token cost. **JSON_PRETTY** and **XML_PRETTY** consistently rank in the bottom two for efficiency. They use 73% to 99% more read tokens than their compact counterparts while producing equal or lower accuracy which makes the extra whitespace purely wasteful overhead.
6. **XML_COMPACT**'s output token advantage is unstable across variants. Mandatory data produces only 5,295 output tokens but optional data inflates this to 9,288 (+75.4%) which is the largest relative increase of any format. The model's reasoning overhead appears to scale disproportionately with XML when data density decreases.
7. Aggregation is the weakest category for all formats and degrades further with optional data. Scores range from 44.45% to 68.25% (mandatory) and 38.10% to 53.33% (optional). **XML_COMPACT** drops 30.16 percentage points between variants for aggregation which confirms that computational tasks on sparse data are the primary driver of accuracy loss.

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
| JSON_PRETTY ≈ 77s | JSON_COMPACT ≈ 10163 | YAML ≈ 213 | XML_COMPACT ≈ 4990 | XML_COMPACT ≈ 5295 | XML_COMPACT ≈ 18000 | XML_COMPACT ≈ 75.81% | JSON_COMPACT ≈ 82 | XML_COMPACT ≈ 84 | XML_COMPACT ≈ 84 | YAML ≈ 96.43% | JSON_COMPACT ≈ 96 | XML_COMPACT ≈ 97 | XML_COMPACT ≈ 97 |
| XML_COMPACT (+3.89%) | XML_COMPACT (+25.01%) | XML_PRETTY (+42.12%) | TOON_DEFAULT (+72.46%) | TOON_DEFAULT (+68.36%) | JSON_COMPACT (+11.63%) | YAML (-0.54%) | XML_COMPACT (-9.15%) | TOON_DEFAULT (-17.62%) | JSON_COMPACT (-7.48%) | JSON_COMPACT (-0.20%) | XML_COMPACT (-8.76%) | TOON_DEFAULT (-14.41%) | JSON_COMPACT (-5.51%) |
| XML_PRETTY (+6.07%) | TOON_DEFAULT (+38.42%) | XML_COMPACT (+43.42%) | JSON_PRETTY (+79.09%) | JSON_PRETTY (+74.56%) | TOON_DEFAULT (+27.68%) | JSON_COMPACT (-0.81%) | YAML (-15.18%) | JSON_PRETTY (-19.70%) | TOON_DEFAULT (-17.49%) | XML_COMPACT (-0.78%) | YAML (-12.99%) | JSON_PRETTY (-16.17%) | TOON_DEFAULT (-14.29%) |
| JSON_COMPACT (+6.56%) | YAML (+39.28%) | JSON_COMPACT (+43.51%) | JSON_COMPACT (+92.88%) | JSON_COMPACT (+87.53%) | YAML (+34.37%) | TOON_DEFAULT (-1.52%) | TOON_DEFAULT (-15.65%) | JSON_COMPACT (-21.66%) | YAML (-20.64%) | TOON_DEFAULT (-1.12%) | TOON_DEFAULT (-13.48%) | JSON_COMPACT (-17.75%) | YAML (-16.92%) |
| TOON_DEFAULT (+7.89%) | JSON_PRETTY (+73.98%) | JSON_PRETTY (+44.17%) | XML_PRETTY (+93.24%) | XML_PRETTY (+87.81%) | JSON_PRETTY (+49.58%) | JSON_PRETTY (-2.26%) | JSON_PRETTY (-30.19%) | YAML (-21.90%) | JSON_PRETTY (-30.96%) | JSON_PRETTY (-1.82%) | JSON_PRETTY (-25.85%) | YAML (-18.01%) | JSON_PRETTY (-25.90%) |
| YAML (+11.79%) | XML_PRETTY (+98.80%) | TOON_DEFAULT (+45.31%) | YAML (+96.77%) | YAML (+89.45%) | XML_PRETTY (+67.49%) | XML_PRETTY (-6.19%) | XML_PRETTY (-43.13%) | XML_PRETTY (-26.00%) | XML_PRETTY (-44.61%) | XML_PRETTY (-2.25%) | XML_PRETTY (-34.44%) | XML_PRETTY (-19.22%) | XML_PRETTY (-35.29%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 67s | JSON_COMPACT ≈ 9645 | XML_PRETTY ≈ 205 | YAML ≈ 7786 | YAML ≈ 8089 | JSON_COMPACT ≈ 21434 | JSON_COMPACT ≈ 76.45% | JSON_COMPACT ≈ 84 | XML_PRETTY ≈ 68 | JSON_COMPACT ≈ 75 | XML_COMPACT ≈ 96.80% | JSON_COMPACT ≈ 97 | YAML ≈ 85 | JSON_COMPACT ≈ 88 |
| XML_PRETTY (+2.24%) | XML_COMPACT (+29.13%) | TOON_DEFAULT (+25.69%) | XML_PRETTY (+3.99%) | XML_PRETTY (+2.63%) | XML_COMPACT (+1.44%) | TOON_DEFAULT (-1.33%) | XML_COMPACT (-12.29%) | XML_COMPACT (-1.28%) | XML_COMPACT (-3.13%) | TOON_DEFAULT (-0.38%) | XML_COMPACT (-8.66%) | XML_PRETTY (-1.48%) | XML_COMPACT (-0.48%) |
| XML_COMPACT (+13.55%) | TOON_DEFAULT (+43.41%) | YAML (+47.64%) | XML_COMPACT (+15.34%) | XML_COMPACT (+14.82%) | YAML (+3.74%) | XML_COMPACT (-2.25%) | TOON_DEFAULT (-16.71%) | YAML (-2.74%) | YAML (-12.37%) | JSON_COMPACT (-0.63%) | TOON_DEFAULT (-13.37%) | XML_COMPACT (-2.40%) | YAML (-4.89%) |
| JSON_PRETTY (+39.93%) | YAML (+46.69%) | JSON_COMPACT (+48.68%) | JSON_PRETTY (+46.79%) | JSON_PRETTY (+45.07%) | TOON_DEFAULT (+20.11%) | JSON_PRETTY (-2.53%) | YAML (-25.22%) | JSON_COMPACT (-12.98%) | TOON_DEFAULT (-16.95%) | YAML (-3.79%) | YAML (-16.73%) | JSON_COMPACT (-14.12%) | TOON_DEFAULT (-13.22%) |
| JSON_COMPACT (+40.79%) | JSON_PRETTY (+73.74%) | JSON_PRETTY (+48.78%) | JSON_COMPACT (+47.50%) | JSON_COMPACT (+45.75%) | XML_PRETTY (+30.50%) | XML_PRETTY (-6.56%) | JSON_PRETTY (-28.59%) | TOON_DEFAULT (-14.95%) | JSON_PRETTY (-28.07%) | XML_PRETTY (-4.46%) | JSON_PRETTY (-25.96%) | TOON_DEFAULT (-14.47%) | XML_PRETTY (-23.25%) |
| TOON_DEFAULT (+40.96%) | XML_PRETTY (+103.95%) | XML_COMPACT (+49.76%) | TOON_DEFAULT (+49.68%) | TOON_DEFAULT (+47.27%) | JSON_PRETTY (+32.92%) | YAML (-10.59%) | XML_PRETTY (-42.68%) | JSON_PRETTY (-15.13%) | XML_PRETTY (-29.76%) | JSON_PRETTY (-4.95%) | XML_PRETTY (-35.05%) | JSON_PRETTY (-17.28%) | JSON_PRETTY (-25.24%) |


#### 2.1.2 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99.39% | JSON_PRETTY ≈ 66.67% | YAML ≈ 61.90% | XML_COMPACT ≈ 68.25% |
| JSON_COMPACT (-3.03%) | XML_COMPACT (0.00%) | JSON_PRETTY (-1.91%) | JSON_COMPACT (-8.25%) |
| TOON_DEFAULT (-5.76%) | TOON_DEFAULT (-6.02%) | JSON_COMPACT (-3.81%) | TOON_DEFAULT (-10.52%) |
| JSON_PRETTY (-7.76%) | YAML (-6.17%) | TOON_DEFAULT (-4.17%) | XML_PRETTY (-19.05%) |
| XML_COMPACT (-8.48%) | XML_PRETTY (-7.41%) | XML_COMPACT (-6.35%) | JSON_PRETTY (-19.68%) |
| XML_PRETTY (-10.91%) | JSON_COMPACT (-10.37%) | XML_PRETTY (-7.94%) | YAML (-23.81%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 98.18% | XML_COMPACT ≈ 70.37% | JSON_PRETTY ≈ 68.25% | JSON_COMPACT ≈ 53.33% |
| JSON_PRETTY (-2.43%) | TOON_DEFAULT (-2.12%) | TOON_DEFAULT (-1.59%) | TOON_DEFAULT (-8.43%) |
| XML_COMPACT (-3.64%) | XML_PRETTY (-8.64%) | JSON_COMPACT (-4.45%) | XML_PRETTY (-12.06%) |
| TOON_DEFAULT (-4.94%) | JSON_PRETTY (-9.88%) | XML_PRETTY (-6.35%) | YAML (-13.65%) |
| XML_PRETTY (-10.31%) | JSON_COMPACT (-10.37%) | YAML (-6.35%) | JSON_PRETTY (-13.65%) |
| YAML (-15.76%) | YAML (-14.81%) | XML_COMPACT (-6.35%) | XML_COMPACT (-15.24%) |


#### 2.1.3 Category Accuracy By Character Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 98.69% | JSON_PRETTY ≈ 96.42% | JSON_COMPACT ≈ 91.61% | XML_COMPACT ≈ 87.04% |
| JSON_COMPACT (-1.61%) | XML_COMPACT (-0.09%) | TOON_DEFAULT (-0.20%) | JSON_COMPACT (-2.96%) |
| TOON_DEFAULT (-3.33%) | JSON_COMPACT (-0.20%) | YAML (-0.54%) | TOON_DEFAULT (-6.24%) |
| XML_COMPACT (-3.34%) | XML_PRETTY (-0.29%) | JSON_PRETTY (-1.25%) | JSON_PRETTY (-7.29%) |
| JSON_PRETTY (-5.64%) | TOON_DEFAULT (-0.49%) | XML_COMPACT (-1.73%) | XML_PRETTY (-8.60%) |
| XML_PRETTY (-6.22%) | YAML (-0.65%) | XML_PRETTY (-2.91%) | YAML (-11.11%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 99.09% | XML_COMPACT ≈ 97.19% | JSON_PRETTY ≈ 92.04% | JSON_COMPACT ≈ 78.49% |
| JSON_COMPACT (-1.10%) | TOON_DEFAULT (-0.06%) | TOON_DEFAULT (-0.12%) | TOON_DEFAULT (-5.50%) |
| XML_COMPACT (-1.45%) | XML_PRETTY (-0.54%) | XML_COMPACT (-0.59%) | YAML (-7.18%) |
| TOON_DEFAULT (-2.32%) | YAML (-1.06%) | YAML (-1.77%) | XML_COMPACT (-7.58%) |
| YAML (-8.65%) | JSON_COMPACT (-1.72%) | JSON_COMPACT (-1.95%) | JSON_PRETTY (-7.78%) |
| XML_PRETTY (-10.44%) | JSON_PRETTY (-9.88%) | XML_PRETTY (-5.02%) | XML_PRETTY (-7.78%) |


#### 2.1.4 Conclusion

No single format dominates every metric. The optimal choice depends on the use case and data characteristics.

- For mandatory (dense) data **XML_COMPACT** leads on total efficiency score by answer accuracy (83.85) and total efficiency by character accuracy (97.07). It combines moderate read cost (12,705 tokens) with low output tokens (5,295) which suggests the model processes structured XML tags with less internal overhead when all fields are present. **JSON_COMPACT** is a close second overall (77.57 efficiency) and wins on read token economy (10,163 tokens) which makes it the better choice when minimizing input context consumption is the priority.
- For optional (sparse) data **JSON_COMPACT** takes the lead across both accuracy (76.45%) and total efficiency score (74.86). **XML_COMPACT** remains competitive on efficiency (72.52) but loses its output token advantage as the model generates 75% more output tokens compared to mandatory data. The rankings shift because **XML_COMPACT**'s efficiency gains in the mandatory scenario were heavily driven by low output tokens which is a property that does not hold up under data sparsity and is dependent on how much reasoning the model decides to do."
- **YAML** is the least stable format. While it scores competitively on mandatory data (75.27% accuracy, second for field retrieval at 99.39%) its 9.41 percentage point accuracy drop on optional data is more than triple the next worst degradation. This makes **YAML** unreliable for production contexts where data completeness cannot be guaranteed.
- Pretty-printed formats are consistently the worst performers. **JSON_PRETTY** and **XML_PRETTY** occupy the bottom two positions on nearly every efficiency ranking. The additional whitespace and indentation inflates read tokens by 73% to 99% compared to compact variants without any measurable accuracy benefit. For AI consumption pretty-printing is pure token waste.
- Category analysis reveals that format choice primarily affects retrieval and aggregation and not structure awareness. Accuracy by character for the structure awareness scores are tightly clustered (95.47% to 97.19% across all optional formats) which indicates all formats convey schema information effectively at the character level. The differentiation comes from field retrieval (82.42% to 98.18% for optional) and aggregation (38.10% to 53.33%) where the model's ability to locate and compute over values varies significantly by format.
- Use **JSON_COMPACT** as the default for nested data with the model when thinking is disabled. It delivers the lowest read token cost, the most stable accuracy across data variants and the best overall efficiency on mixed data. Consider **XML_COMPACT** only for dense, mandatory data where its output token efficiency advantage produces meaningfully higher total efficiency scores.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total | Accuracy By Character (%) | Useful Read Tokens (Acc By Char) | Wasted Read Tokens (Acc By Char) | Useful Output Tokens (Acc By Char) | Wasted Output Tokens (Acc By Char) | Eff Score Read (Acc By Char) | Eff Score Output (Acc By Char) | Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10163 | 9930 | 20093 | 2.25 | 77.62 | 75.00 | 7622 | 2541 | 7447 | 2482 | 81.67 | 65.68 | 77.57 | 96.23 | 9780 | 383 | 9555 | 374 | 95.82 | 79.83 | 91.73 |
| JSON_COMPACT | opt | 9645 | 11789 | 21434 | 2.22 | 92.62 | 76.45 | 7374 | 2271 | 9013 | 2776 | 84.27 | 59.58 | 74.86 | 96.17 | 9276 | 369 | 11338 | 452 | 97.41 | 72.72 | 88.01 |
| JSON_PRETTY | man | 17682 | 9243 | 26925 | 1.78 | 72.07 | 73.55 | 13005 | 4677 | 6798 | 2445 | 57.01 | 67.32 | 57.89 | 94.61 | 16729 | 953 | 8745 | 498 | 71.05 | 81.36 | 71.93 |
| JSON_PRETTY | opt | 16757 | 11734 | 28491 | 1.77 | 92.17 | 73.92 | 12387 | 4370 | 8674 | 3060 | 60.17 | 58.10 | 53.85 | 91.85 | 15391 | 1366 | 10778 | 956 | 72.13 | 70.05 | 65.80 |
| TOON_DEFAULT | man | 14068 | 8915 | 22983 | 1.86 | 69.40 | 74.29 | 10451 | 3617 | 6623 | 2292 | 68.89 | 69.06 | 69.18 | 95.31 | 13408 | 660 | 8497 | 418 | 82.91 | 83.08 | 83.20 |
| TOON_DEFAULT | opt | 13832 | 11912 | 25744 | 1.86 | 93.99 | 75.12 | 10391 | 3441 | 8948 | 2964 | 70.19 | 58.23 | 62.17 | 96.42 | 13337 | 495 | 11485 | 426 | 84.39 | 72.43 | 76.37 |
| XML_COMPACT | man | 12705 | 5295 | 18000 | 2.55 | 40.24 | 75.81 | 9632 | 3073 | 4014 | 1281 | 74.20 | 83.83 | 83.85 | 95.65 | 12152 | 553 | 5065 | 230 | 87.43 | 97.06 | 97.07 |
| XML_COMPACT | opt | 12455 | 9288 | 21743 | 2.50 | 72.42 | 74.20 | 9242 | 3213 | 6891 | 2396 | 73.91 | 67.59 | 72.52 | 96.80 | 12056 | 399 | 8990 | 297 | 88.98 | 82.65 | 87.59 |
| XML_PRETTY | man | 20204 | 9945 | 30149 | 1.99 | 77.76 | 69.62 | 14066 | 6138 | 6924 | 3021 | 46.44 | 62.04 | 46.44 | 94.18 | 19028 | 1176 | 9366 | 579 | 62.82 | 78.41 | 62.81 |
| XML_PRETTY | opt | 19671 | 8301 | 27972 | 1.97 | 65.29 | 69.89 | 13748 | 5923 | 5802 | 2500 | 48.30 | 68.46 | 52.58 | 92.34 | 18164 | 1507 | 7665 | 636 | 63.27 | 83.43 | 67.55 |
| YAML | man | 14155 | 10031 | 24186 | 1.81 | 79.18 | 75.27 | 10654 | 3501 | 7551 | 2481 | 69.27 | 65.47 | 66.54 | 96.43 | 13650 | 505 | 9673 | 358 | 83.38 | 79.58 | 80.65 |
| YAML | opt | 14148 | 8089 | 22237 | 1.79 | 62.79 | 65.86 | 9318 | 4830 | 5327 | 2761 | 63.02 | 66.58 | 65.61 | 93.01 | 13159 | 989 | 7523 | 565 | 81.12 | 84.68 | 83.71 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10163 | 9645 | -518 | -5.10 | 305 | 305 | 0 | 0.00 | 9625 | 11485 | +1860 | +19.32 | 9930 | 11790 | +1860 | +18.73 | 20093 | 21435 | +1342 | +6.68 |
| JSON_PRETTY | 17682 | 16757 | -925 | -5.23 | 307 | 305 | -2 | -0.65 | 8937 | 11430 | +2493 | +27.90 | 9243 | 11734 | +2491 | +26.95 | 26925 | 28491 | +1566 | +5.82 |
| TOON_DEFAULT | 14068 | 13832 | -236 | -1.68 | 309 | 258 | -51 | -16.50 | 8606 | 11655 | +3049 | +35.43 | 8915 | 11912 | +2997 | +33.62 | 22983 | 25744 | +2761 | +12.01 |
| XML_COMPACT | 12705 | 12455 | -250 | -1.97 | 305 | 307 | +2 | +0.66 | 4990 | 8981 | +3991 | +79.98 | 5295 | 9288 | +3993 | +75.41 | 18000 | 21743 | +3743 | +20.79 |
| XML_PRETTY | 20204 | 19671 | -533 | -2.64 | 302 | 205 | -97 | -32.12 | 9643 | 8097 | -1546 | -16.03 | 9945 | 8302 | -1643 | -16.52 | 30149 | 27973 | -2176 | -7.22 |
| YAML | 14155 | 14148 | -7 | -0.05 | 213 | 303 | +90 | +42.25 | 9819 | 7786 | -2033 | -20.70 | 10031 | 8088 | -1943 | -19.37 | 24186 | 22236 | -1950 | -8.06 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Before Write (ms) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) | Output (ms) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 13 | 781.77 | 0.42 | 45912 | 35860 | 0.27 | 289.19 | 35873 | 782.04 | 231.44 | 81772 |
| JSON_COMPACT | opt | 9 | 1071.67 | 0.29 | 56146 | 38516 | 0.30 | 310.61 | 38525 | 1071.96 | 248.55 | 94662 |
| JSON_PRETTY | man | 262 | 67.49 | 8.45 | 39379 | 37358 | 0.24 | 301.27 | 37620 | 67.73 | 242.71 | 76737 |
| JSON_PRETTY | opt | 270 | 62.06 | 8.71 | 57037 | 37045 | 0.31 | 298.75 | 37315 | 62.37 | 240.74 | 94082 |
| TOON_DEFAULT | man | 31 | 453.81 | 1.00 | 44249 | 38540 | 0.23 | 310.81 | 38571 | 454.04 | 248.85 | 82789 |
| TOON_DEFAULT | opt | 29 | 476.97 | 0.94 | 56881 | 37891 | 0.32 | 305.57 | 37920 | 477.29 | 244.65 | 94772 |
| XML_COMPACT | man | 13 | 977.31 | 0.42 | 38637 | 41086 | 0.12 | 331.34 | 41099 | 977.43 | 265.15 | 79723 |
| XML_COMPACT | opt | 12 | 1037.92 | 0.39 | 35363 | 40983 | 0.22 | 330.51 | 40995 | 1038.14 | 264.48 | 76345 |
| XML_PRETTY | man | 13 | 1554.15 | 0.42 | 43389 | 38004 | 0.25 | 306.48 | 38017 | 1554.41 | 245.27 | 81393 |
| XML_PRETTY | opt | 6 | 3278.50 | 0.19 | 28165 | 40578 | 0.20 | 327.24 | 40584 | 3278.70 | 261.83 | 68743 |
| YAML | man | 12 | 1179.58 | 0.39 | 48091 | 37690 | 0.26 | 303.95 | 37702 | 1179.84 | 243.24 | 85781 |
| YAML | opt | 11 | 1286.18 | 0.35 | 31187 | 36048 | 0.22 | 290.71 | 36059 | 1286.40 | 232.64 | 67236 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Before Write Man (s) | Output Before Write Opt (s) | Diff (s) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 13 | 9 | -4 | -30.77 | 45.91 | 56.15 | +10.23 | +22.29 | 35.86 | 38.52 | +2.66 | +7.41 | 35.87 | 38.52 | +2.65 | +7.39 | 81.77 | 94.66 | +12.89 | +15.76 |
| JSON_PRETTY | 262 | 270 | +8 | +3.05 | 39.38 | 57.04 | +17.66 | +44.84 | 37.36 | 37.05 | -0.31 | -0.84 | 37.62 | 37.32 | -0.30 | -0.81 | 76.74 | 94.08 | +17.35 | +22.60 |
| TOON_DEFAULT | 31 | 29 | -2 | -6.45 | 44.25 | 56.88 | +12.63 | +28.55 | 38.54 | 37.89 | -0.65 | -1.68 | 38.57 | 37.92 | -0.65 | -1.69 | 82.79 | 94.77 | +11.98 | +14.47 |
| XML_COMPACT | 13 | 12 | -1 | -7.69 | 38.64 | 35.36 | -3.27 | -8.47 | 41.09 | 40.98 | -0.10 | -0.25 | 41.10 | 40.99 | -0.10 | -0.26 | 79.72 | 76.35 | -3.38 | -4.24 |
| XML_PRETTY | 13 | 6 | -7 | -53.85 | 43.39 | 28.16 | -15.22 | -35.09 | 38.00 | 40.58 | +2.57 | +6.77 | 38.02 | 40.58 | +2.57 | +6.75 | 81.39 | 68.74 | -12.65 | -15.54 |
| YAML | 12 | 11 | -1 | -8.33 | 48.09 | 31.19 | -16.90 | -35.15 | 37.69 | 36.05 | -1.64 | -4.36 | 37.70 | 36.06 | -1.64 | -4.36 | 85.78 | 67.24 | -18.55 | -21.62 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token | Info / Read Token (Acc By Char) | Info / Output Token (Acc By Char) | Info / Total Token (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 2.25 | 14.90 | 327.84 | 0.74 | 0.76 | 0.37 | 0.95 | 0.97 | 0.48 |
| JSON_COMPACT | opt | 2.22 | 15.29 | 311.13 | 0.79 | 0.65 | 0.36 | 1.00 | 0.82 | 0.45 |
| JSON_PRETTY | man | 1.78 | 25.93 | 570.39 | 0.42 | 0.80 | 0.27 | 0.54 | 1.02 | 0.35 |
| JSON_PRETTY | opt | 1.77 | 26.56 | 540.55 | 0.44 | 0.63 | 0.26 | 0.55 | 0.78 | 0.32 |
| TOON_DEFAULT | man | 1.86 | 20.63 | 453.81 | 0.53 | 0.86 | 0.32 | 0.68 | 1.10 | 0.42 |
| TOON_DEFAULT | opt | 1.86 | 21.92 | 446.19 | 0.54 | 0.65 | 0.29 | 0.70 | 0.84 | 0.38 |
| XML_COMPACT | man | 2.55 | 18.63 | 409.84 | 0.60 | 1.43 | 0.42 | 0.75 | 1.81 | 0.53 |
| XML_COMPACT | opt | 2.50 | 19.74 | 401.77 | 0.60 | 0.80 | 0.34 | 0.78 | 1.04 | 0.45 |
| XML_PRETTY | man | 1.99 | 29.63 | 651.74 | 0.35 | 0.70 | 0.23 | 0.47 | 0.95 | 0.31 |
| XML_PRETTY | opt | 1.97 | 31.17 | 634.55 | 0.36 | 0.84 | 0.25 | 0.47 | 1.11 | 0.33 |
| YAML | man | 1.81 | 20.76 | 456.61 | 0.53 | 0.75 | 0.31 | 0.68 | 0.96 | 0.40 |
| YAML | opt | 1.79 | 22.42 | 456.39 | 0.47 | 0.81 | 0.30 | 0.66 | 1.15 | 0.42 |

#### 2.5.2 Characters And Values: Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 2.25 | 2.22 | -0.03 | -1.20 | 14.90 | 15.29 | +0.38 | +2.57 | 327.84 | 311.13 | -16.71 | -5.10 |
| JSON_PRETTY | 1.78 | 1.77 | -0.01 | -0.56 | 25.93 | 26.56 | +0.63 | +2.43 | 570.39 | 540.55 | -29.84 | -5.23 |
| TOON_DEFAULT | 1.86 | 1.86 | +0.01 | +0.49 | 20.63 | 21.92 | +1.29 | +6.27 | 453.81 | 446.19 | -7.61 | -1.68 |
| XML_COMPACT | 2.55 | 2.50 | -0.05 | -2.04 | 18.63 | 19.74 | +1.11 | +5.96 | 409.84 | 401.77 | -8.06 | -1.97 |
| XML_PRETTY | 1.99 | 1.97 | -0.01 | -0.55 | 29.63 | 31.17 | +1.55 | +5.23 | 651.74 | 634.55 | -17.19 | -2.64 |
| YAML | 1.81 | 1.79 | -0.02 | -1.16 | 20.76 | 22.42 | +1.67 | +8.03 | 456.61 | 456.39 | -0.23 | -0.05 |

#### 2.5.3 Information: Mandatory vs Optional
| Format | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 0.74 | 0.79 | +0.06 | +7.45 | 0.76 | 0.65 | -0.11 | -14.17 | 0.37 | 0.36 | -0.02 | -4.29 |
| JSON_PRETTY | 0.42 | 0.44 | +0.03 | +6.01 | 0.80 | 0.63 | -0.17 | -20.85 | 0.27 | 0.26 | -0.01 | -5.13 |
| TOON_DEFAULT | 0.53 | 0.54 | +0.02 | +2.84 | 0.86 | 0.65 | -0.21 | -24.02 | 0.32 | 0.29 | -0.03 | -9.69 |
| XML_COMPACT | 0.60 | 0.60 | 0.00 | 0.00 | 1.43 | 0.80 | -0.63 | -44.20 | 0.42 | 0.34 | -0.08 | -19.00 |
| XML_PRETTY | 0.35 | 0.36 | +0.01 | +2.90 | 0.70 | 0.84 | +0.14 | +20.29 | 0.23 | 0.25 | +0.02 | +8.23 |
| YAML | 0.53 | 0.47 | -0.07 | -12.41 | 0.75 | 0.81 | +0.06 | +8.53 | 0.31 | 0.30 | -0.02 | -4.82 |

#### 2.5.4 Information (Accuracy By Character): Mandatory vs Optional
| Format | Info / Read Token (Acc By Char) Man | Info / Read Token (Acc By Char) Opt | Diff | Diff (%) | Info / Output Token (Acc By Char) Man | Info / Output Token (Acc By Char)  Opt | Diff | Diff (%) | Info / Total Token (Acc By Char) Man | Info / Total Token (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 0.95 | 1.00 | +0.05 | +5.28 | 0.97 | 0.82 | -0.15 | -15.79 | 0.48 | 0.45 | -0.03 | -6.26 |
| JSON_PRETTY | 0.54 | 0.55 | +0.01 | +2.43 | 1.02 | 0.78 | -0.24 | -23.54 | 0.35 | 0.32 | -0.03 | -8.26 |
| TOON_DEFAULT | 0.68 | 0.70 | +0.02 | +2.95 | 1.10 | 0.84 | -0.26 | -24.00 | 0.42 | 0.38 | -0.04 | -9.48 |
| XML_COMPACT | 0.75 | 0.78 | +0.02 | +3.19 | 1.81 | 1.04 | -0.76 | -42.30 | 0.53 | 0.45 | -0.09 | -16.20 |
| XML_PRETTY | 0.47 | 0.47 | 0.00 | 0.00 | 0.95 | 1.11 | +0.17 | +17.42 | 0.31 | 0.33 | +0.02 | +5.77 |
| YAML | 0.68 | 0.66 | -0.02 | -3.52 | 0.96 | 1.15 | +0.19 | +19.67 | 0.40 | 0.42 | +0.02 | +4.76 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10163 | 7622 | 2541 | 9930 | 7447 | 2482 | 20093 | 15070 | 5023 | 75.00 | 81.67 | 65.68 | 77.57 | 72.16 | 79.78 | 63.79 | 75.68 |
| JSON_COMPACT | opt | 9645 | 7374 | 2271 | 11789 | 9013 | 2776 | 21434 | 16387 | 5048 | 76.45 | 84.27 | 59.58 | 74.86 | 74.27 | 82.81 | 58.13 | 73.41 |
| JSON_PRETTY | man | 17682 | 13005 | 4677 | 9243 | 6798 | 2445 | 26925 | 19803 | 7122 | 73.55 | 57.01 | 67.32 | 57.89 | 72.37 | 56.22 | 66.54 | 57.10 |
| JSON_PRETTY | opt | 16757 | 12387 | 4370 | 11734 | 8674 | 3060 | 28491 | 21061 | 7431 | 73.92 | 60.17 | 58.10 | 53.85 | 72.73 | 59.38 | 57.31 | 53.05 |
| TOON_DEFAULT | man | 14068 | 10451 | 3617 | 8915 | 6623 | 2292 | 22983 | 17074 | 5909 | 74.29 | 68.89 | 69.06 | 69.18 | 72.05 | 67.40 | 67.57 | 67.69 |
| TOON_DEFAULT | opt | 13832 | 10391 | 3441 | 11912 | 8948 | 2964 | 25744 | 19339 | 6405 | 75.12 | 70.19 | 58.23 | 62.17 | 74.38 | 69.69 | 57.73 | 61.68 |
| XML_COMPACT | man | 12705 | 9632 | 3073 | 5295 | 4014 | 1281 | 18000 | 13646 | 4354 | 75.81 | 74.20 | 83.83 | 83.85 | 73.63 | 72.75 | 82.38 | 82.39 |
| XML_COMPACT | opt | 12455 | 9242 | 3213 | 9288 | 6891 | 2396 | 21743 | 16133 | 5610 | 74.20 | 73.91 | 67.59 | 72.52 | 73.64 | 73.54 | 67.21 | 72.15 |
| XML_PRETTY | man | 20204 | 14066 | 6138 | 9945 | 6924 | 3021 | 30149 | 20990 | 9159 | 69.62 | 46.44 | 62.04 | 46.44 | 67.86 | 45.27 | 60.86 | 45.27 |
| XML_PRETTY | opt | 19671 | 13748 | 5923 | 8301 | 5802 | 2500 | 27972 | 19550 | 8422 | 69.89 | 48.30 | 68.46 | 52.58 | 69.01 | 47.72 | 67.88 | 52.00 |
| YAML | man | 14155 | 10654 | 3501 | 10031 | 7551 | 2481 | 24186 | 18205 | 5981 | 75.27 | 69.27 | 65.47 | 66.54 | 73.37 | 68.00 | 64.21 | 65.27 |
| YAML | opt | 14148 | 9318 | 4830 | 8089 | 5327 | 2761 | 22237 | 14645 | 7592 | 65.86 | 63.02 | 66.58 | 65.61 | 64.97 | 62.43 | 65.99 | 65.01 |

#### 2.6.2 Read Tokens: Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10163 | 9645 | -518 | -5.10 | 7622 | 7373 | -249 | -3.26 | 2541 | 2272 | -269 | -10.60 | 75.00 | 76.45 | +1.45 | 81.67 | 84.27 | +2.60 | +3.18 | 72.16 | 74.27 | +2.11 | 79.78 | 82.81 | +3.04 | +3.81 |
| JSON_PRETTY | 17682 | 16757 | -925 | -5.23 | 13005 | 12387 | -618 | -4.75 | 4677 | 4370 | -307 | -6.56 | 73.55 | 73.92 | +0.37 | 57.01 | 60.17 | +3.16 | +5.54 | 72.37 | 72.73 | +0.36 | 56.22 | 59.38 | +3.16 | +5.61 |
| TOON_DEFAULT | 14068 | 13832 | -236 | -1.68 | 10451 | 10390 | -61 | -0.58 | 3617 | 3442 | -175 | -4.85 | 74.29 | 75.12 | +0.83 | 68.89 | 70.19 | +1.30 | +1.88 | 72.05 | 74.38 | +2.33 | 67.40 | 69.69 | +2.30 | +3.41 |
| XML_COMPACT | 12705 | 12455 | -250 | -1.97 | 9632 | 9242 | -390 | -4.05 | 3073 | 3213 | +140 | +4.56 | 75.81 | 74.20 | -1.61 | 74.20 | 73.91 | -0.28 | -0.38 | 73.63 | 73.64 | +0.01 | 72.75 | 73.54 | +0.79 | +1.09 |
| XML_PRETTY | 20204 | 19671 | -533 | -2.64 | 14066 | 13748 | -318 | -2.26 | 6138 | 5923 | -215 | -3.50 | 69.62 | 69.89 | +0.27 | 46.44 | 48.30 | +1.86 | +4.00 | 67.86 | 69.01 | +1.15 | 45.27 | 47.72 | +2.45 | +5.40 |
| YAML | 14155 | 14148 | -7 | -0.05 | 10654 | 9317 | -1337 | -12.55 | 3501 | 4831 | +1330 | +37.98 | 75.27 | 65.86 | -9.41 | 69.27 | 63.02 | -6.25 | -9.03 | 73.37 | 64.97 | -8.40 | 68.00 | 62.43 | -5.58 | -8.20 |

#### 2.6.3 Output Tokens: Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 9930 | 11790 | +1860 | +18.73 | 7447 | 9013 | +1566 | +21.02 | 2482 | 2776 | +294 | +11.84 | 75.00 | 76.45 | +1.45 | 65.68 | 59.58 | -6.10 | -9.29 | 72.16 | 74.27 | +2.11 | 63.79 | 58.13 | -5.66 | -8.87 |
| JSON_PRETTY | 9243 | 11734 | +2491 | +26.95 | 6798 | 8674 | +1876 | +27.59 | 2445 | 3060 | +615 | +25.17 | 73.55 | 73.92 | +0.37 | 67.32 | 58.10 | -9.22 | -13.70 | 72.37 | 72.73 | +0.36 | 66.54 | 57.31 | -9.23 | -13.87 |
| TOON_DEFAULT | 8915 | 11912 | +2997 | +33.62 | 6623 | 8948 | +2325 | +35.11 | 2292 | 2964 | +672 | +29.31 | 74.29 | 75.12 | +0.83 | 69.06 | 58.23 | -10.84 | -15.69 | 72.05 | 74.38 | +2.33 | 67.57 | 57.73 | -9.84 | -14.56 |
| XML_COMPACT | 5295 | 9288 | +3993 | +75.40 | 4014 | 6891 | +2877 | +71.68 | 1281 | 2396 | +1115 | +87.07 | 75.81 | 74.20 | -1.61 | 83.83 | 67.59 | -16.25 | -19.38 | 73.63 | 73.64 | +0.01 | 82.38 | 67.21 | -15.17 | -18.41 |
| XML_PRETTY | 9945 | 8302 | -1643 | -16.53 | 6924 | 5802 | -1122 | -16.20 | 3021 | 2499 | -522 | -17.27 | 69.62 | 69.89 | +0.27 | 62.04 | 68.46 | +6.43 | +10.36 | 67.86 | 69.01 | +1.15 | 60.86 | 67.88 | +7.01 | +11.52 |
| YAML | 10031 | 8088 | -1943 | -19.37 | 7551 | 5328 | -2223 | -29.44 | 2481 | 2762 | +281 | +11.31 | 75.27 | 65.86 | -9.41 | 65.47 | 66.58 | +1.11 | +1.70 | 73.37 | 64.97 | -8.40 | 64.21 | 65.99 | +1.78 | +2.78 |

#### 2.6.4 Total Tokens: Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 20093 | 21435 | +1342 | +6.68 | 15070 | 16387 | +1317 | +8.74 | 5023 | 5048 | +25 | +0.49 | 75.00 | 76.45 | +1.45 | 77.57 | 74.86 | -2.71 | -3.49 | 72.16 | 74.27 | +2.11 | 75.68 | 73.41 | -2.27 | -3.00 |
| JSON_PRETTY | 26925 | 28491 | +1566 | +5.82 | 19803 | 21060 | +1257 | +6.35 | 7122 | 7431 | +309 | +4.34 | 73.55 | 73.92 | +0.37 | 57.89 | 53.85 | -4.04 | -6.98 | 72.37 | 72.73 | +0.36 | 57.10 | 53.05 | -4.05 | -7.09 |
| TOON_DEFAULT | 22983 | 25744 | +2761 | +12.01 | 17074 | 19339 | +2265 | +13.27 | 5909 | 6405 | +496 | +8.40 | 74.29 | 75.12 | +0.83 | 69.18 | 62.17 | -7.01 | -10.13 | 72.05 | 74.38 | +2.33 | 67.69 | 61.68 | -6.01 | -8.88 |
| XML_COMPACT | 18000 | 21743 | +3743 | +20.79 | 13646 | 16133 | +2487 | +18.23 | 4354 | 5609 | +1255 | +28.83 | 75.81 | 74.20 | -1.61 | 83.85 | 72.52 | -11.33 | -13.51 | 73.63 | 73.64 | +0.01 | 82.39 | 72.15 | -10.24 | -12.43 |
| XML_PRETTY | 30149 | 27973 | -2176 | -7.22 | 20990 | 19550 | -1440 | -6.86 | 9159 | 8422 | -737 | -8.04 | 69.62 | 69.89 | +0.27 | 46.44 | 52.58 | +6.14 | +13.23 | 67.86 | 69.01 | +1.15 | 45.27 | 52.00 | +6.73 | +14.86 |
| YAML | 24186 | 22236 | -1950 | -8.06 | 18205 | 14645 | -3560 | -19.55 | 5981 | 7591 | +1610 | +26.92 | 75.27 | 65.86 | -9.41 | 66.54 | 65.61 | -0.93 | -1.40 | 73.37 | 64.97 | -8.40 | 65.27 | 65.01 | -0.26 | -0.40 |

### 2.7 Token Utilization Efficiency (Accuracy by Character)
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy by Character (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy by Character (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10163 | 9780 | 383 | 9930 | 9555 | 374 | 20093 | 19335 | 757 | 96.23 | 95.82 | 79.83 | 91.73 | 94.06 | 94.38 | 78.39 | 90.28 |
| JSON_COMPACT | opt | 9645 | 9276 | 369 | 11789 | 11338 | 452 | 21434 | 20613 | 821 | 96.17 | 97.41 | 72.72 | 88.01 | 93.17 | 95.41 | 70.72 | 86.01 |
| JSON_PRETTY | man | 17682 | 16729 | 953 | 9243 | 8745 | 498 | 26925 | 25474 | 1451 | 94.61 | 71.05 | 81.36 | 71.93 | 91.81 | 69.18 | 79.50 | 70.06 |
| JSON_PRETTY | opt | 16757 | 15391 | 1366 | 11734 | 10778 | 956 | 28491 | 26169 | 2322 | 91.85 | 72.13 | 70.05 | 65.80 | 90.64 | 71.32 | 69.25 | 64.99 |
| TOON_DEFAULT | man | 14068 | 13408 | 660 | 8915 | 8497 | 418 | 22983 | 21905 | 1078 | 95.31 | 82.91 | 83.08 | 83.20 | 92.88 | 81.29 | 81.46 | 81.58 |
| TOON_DEFAULT | opt | 13832 | 13337 | 495 | 11912 | 11485 | 426 | 25744 | 24822 | 922 | 96.42 | 84.39 | 72.43 | 76.37 | 92.89 | 82.04 | 70.07 | 74.02 |
| XML_COMPACT | man | 12705 | 12152 | 553 | 5295 | 5065 | 230 | 18000 | 17217 | 783 | 95.65 | 87.43 | 97.06 | 97.07 | 93.45 | 85.96 | 95.59 | 95.61 |
| XML_COMPACT | opt | 12455 | 12056 | 399 | 9288 | 8990 | 297 | 21743 | 21047 | 696 | 96.80 | 88.98 | 82.65 | 87.59 | 92.88 | 86.37 | 80.04 | 84.97 |
| XML_PRETTY | man | 20204 | 19028 | 1176 | 9945 | 9366 | 579 | 30149 | 28394 | 1755 | 94.18 | 62.82 | 78.41 | 62.81 | 90.99 | 60.69 | 76.28 | 60.69 |
| XML_PRETTY | opt | 19671 | 18164 | 1507 | 8301 | 7665 | 636 | 27972 | 25830 | 2143 | 92.34 | 63.27 | 83.43 | 67.55 | 88.40 | 60.64 | 80.80 | 64.92 |
| YAML | man | 14155 | 13650 | 505 | 10031 | 9673 | 358 | 24186 | 23323 | 863 | 96.43 | 83.38 | 79.58 | 80.65 | 93.40 | 81.36 | 77.56 | 78.63 |
| YAML | opt | 14148 | 13159 | 989 | 8089 | 7523 | 565 | 22237 | 20682 | 1554 | 93.01 | 81.12 | 84.68 | 83.71 | 89.67 | 78.89 | 82.46 | 81.48 |

#### 2.7.2 Read Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10163 | 9645 | -518 | -5.10 | 9780 | 9276 | -504 | -5.16 | 383 | 369 | -14 | -3.59 | 96.23 | 96.17 | -0.06 | 95.82 | 97.41 | +1.59 | +1.66 | 94.06 | 93.17 | -0.89 | 94.38 | 95.41 | +1.04 | +1.10 |
| JSON_PRETTY | 17682 | 16757 | -925 | -5.23 | 16729 | 15391 | -1338 | -8.00 | 953 | 1366 | +413 | +43.30 | 94.61 | 91.85 | -2.76 | 71.05 | 72.13 | +1.07 | +1.51 | 91.81 | 90.64 | -1.17 | 69.18 | 71.32 | +2.14 | +3.09 |
| TOON_DEFAULT | 14068 | 13832 | -236 | -1.68 | 13408 | 13337 | -71 | -0.53 | 660 | 495 | -165 | -24.94 | 95.31 | 96.42 | +1.11 | 82.91 | 84.39 | +1.48 | +1.79 | 92.88 | 92.89 | +0.01 | 81.29 | 82.04 | +0.75 | +0.92 |
| XML_COMPACT | 12705 | 12455 | -250 | -1.97 | 12152 | 12056 | -96 | -0.79 | 553 | 399 | -154 | -27.87 | 95.65 | 96.80 | +1.15 | 87.43 | 88.98 | +1.55 | +1.78 | 93.45 | 92.88 | -0.57 | 85.96 | 86.37 | +0.41 | +0.47 |
| XML_PRETTY | 20204 | 19671 | -533 | -2.64 | 19028 | 18164 | -864 | -4.54 | 1176 | 1507 | +331 | +28.14 | 94.18 | 92.34 | -1.84 | 62.82 | 63.27 | +0.45 | +0.72 | 90.99 | 88.40 | -2.59 | 60.69 | 60.64 | -0.05 | -0.08 |
| YAML | 14155 | 14148 | -7 | -0.05 | 13650 | 13159 | -491 | -3.59 | 505 | 989 | +484 | +95.76 | 96.43 | 93.01 | -3.42 | 83.38 | 81.12 | -2.26 | -2.71 | 93.40 | 89.67 | -3.73 | 81.36 | 78.89 | -2.46 | -3.03 |

#### 2.7.3 Output Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 9930 | 11790 | +1860 | +18.73 | 9555 | 11337 | +1782 | +18.65 | 374 | 451 | +77 | +20.64 | 96.23 | 96.17 | -0.06 | 79.83 | 72.72 | -7.11 | -8.90 | 94.06 | 93.17 | -0.89 | 78.39 | 70.72 | -7.66 | -9.77 |
| JSON_PRETTY | 9243 | 11734 | +2491 | +26.95 | 8745 | 10778 | +2033 | +23.25 | 498 | 956 | +458 | +92.00 | 94.61 | 91.85 | -2.76 | 81.36 | 70.05 | -11.31 | -13.90 | 91.81 | 90.64 | -1.17 | 79.50 | 69.25 | -10.25 | -12.89 |
| TOON_DEFAULT | 8915 | 11912 | +2997 | +33.62 | 8497 | 11486 | +2989 | +35.18 | 418 | 426 | +8 | +2.00 | 95.31 | 96.42 | +1.11 | 83.08 | 72.43 | -10.65 | -12.82 | 92.88 | 92.89 | +0.01 | 81.46 | 70.07 | -11.38 | -13.98 |
| XML_COMPACT | 5295 | 9288 | +3993 | +75.40 | 5065 | 8991 | +3926 | +77.51 | 230 | 297 | +67 | +29.07 | 95.65 | 96.80 | +1.15 | 97.06 | 82.65 | -14.41 | -14.84 | 93.45 | 92.88 | -0.57 | 95.59 | 80.04 | -15.55 | -16.27 |
| XML_PRETTY | 9945 | 8302 | -1643 | -16.53 | 9366 | 7665 | -1701 | -18.16 | 579 | 636 | +57 | +9.86 | 94.18 | 92.34 | -1.84 | 78.41 | 83.43 | +5.02 | +6.40 | 90.99 | 88.40 | -2.59 | 76.28 | 80.80 | +4.52 | +5.93 |
| YAML | 10031 | 8088 | -1943 | -19.37 | 9673 | 7523 | -2150 | -22.23 | 358 | 565 | +207 | +57.90 | 96.43 | 93.01 | -3.42 | 79.58 | 84.68 | +5.10 | +6.41 | 93.40 | 89.67 | -3.73 | 77.56 | 82.46 | +4.90 | +6.31 |

#### 2.7.4 Total Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 20093 | 21435 | +1342 | +6.68 | 19335 | 20613 | +1278 | +6.61 | 757 | 820 | +63 | +8.38 | 96.23 | 96.17 | -0.06 | 91.73 | 88.01 | -3.72 | -4.05 | 94.06 | 93.17 | -0.89 | 90.28 | 86.01 | -4.27 | -4.73 |
| JSON_PRETTY | 26925 | 28491 | +1566 | +5.82 | 25474 | 26169 | +695 | +2.73 | 1451 | 2322 | +871 | +60.01 | 94.61 | 91.85 | -2.76 | 71.93 | 65.80 | -6.13 | -8.52 | 91.81 | 90.64 | -1.17 | 70.06 | 64.99 | -5.07 | -7.23 |
| TOON_DEFAULT | 22983 | 25744 | +2761 | +12.01 | 21905 | 24822 | +2917 | +13.32 | 1078 | 922 | -156 | -14.50 | 95.31 | 96.42 | +1.11 | 83.20 | 76.37 | -6.82 | -8.20 | 92.88 | 92.89 | +0.01 | 81.58 | 74.02 | -7.56 | -9.26 |
| XML_COMPACT | 18000 | 21743 | +3743 | +20.79 | 17217 | 21047 | +3830 | +22.24 | 783 | 696 | -87 | -11.14 | 95.65 | 96.80 | +1.15 | 97.07 | 87.59 | -9.48 | -9.77 | 93.45 | 92.88 | -0.57 | 95.61 | 84.97 | -10.63 | -11.12 |
| XML_PRETTY | 30149 | 27973 | -2176 | -7.22 | 28394 | 25830 | -2564 | -9.03 | 1755 | 2143 | +388 | +22.11 | 94.18 | 92.34 | -1.84 | 62.81 | 67.55 | +4.73 | +7.54 | 90.99 | 88.40 | -2.59 | 60.69 | 64.92 | +4.23 | +6.98 |
| YAML | 24186 | 22236 | -1950 | -8.06 | 23323 | 20682 | -2641 | -11.32 | 863 | 1554 | +691 | +80.06 | 96.43 | 93.01 | -3.42 | 80.65 | 83.71 | +3.06 | +3.79 | 93.40 | 89.67 | -3.73 | 78.63 | 81.48 | +2.85 | +3.63 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Expected Characters | Output Characters | Correct Characters | Incorrect Characters | Accuracy by Character (%) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 93 | 31 | 0 | 75.00 | 7782 | 7883 | 7585 | 298 | 96.23 |
| JSON_COMPACT | opt | 95 | 29 | 0 | 76.45 | 8451 | 8663 | 8329 | 334 | 96.17 |
| JSON_PRETTY | man | 91 | 33 | 0 | 73.55 | 7782 | 7952 | 7521 | 431 | 94.61 |
| JSON_PRETTY | opt | 92 | 32 | 0 | 73.92 | 8451 | 8833 | 8082 | 752 | 91.85 |
| TOON_DEFAULT | man | 92 | 32 | 0 | 74.29 | 7782 | 7937 | 7559 | 377 | 95.31 |
| TOON_DEFAULT | opt | 93 | 31 | 0 | 75.12 | 8451 | 8591 | 8283 | 308 | 96.42 |
| XML_COMPACT | man | 94 | 30 | 0 | 75.81 | 7782 | 7913 | 7566 | 348 | 95.65 |
| XML_COMPACT | opt | 92 | 32 | 0 | 74.20 | 8451 | 8568 | 8292 | 276 | 96.80 |
| XML_PRETTY | man | 86 | 38 | 0 | 69.62 | 7782 | 7907 | 7446 | 461 | 94.18 |
| XML_PRETTY | opt | 87 | 37 | 0 | 69.89 | 8451 | 8742 | 8065 | 677 | 92.34 |
| YAML | man | 93 | 31 | 0 | 75.27 | 7782 | 7877 | 7595 | 282 | 96.43 |
| YAML | opt | 82 | 42 | 0 | 65.86 | 8451 | 8690 | 8082 | 608 | 93.01 |

#### 2.8.2 Answers: Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 93 | 95 | +2 | +1.94 | 31 | 29 | -2 | -5.81 | 0 | 0 | 0 | 0.00 | 75.00 | 76.45 | +1.45 |
| JSON_PRETTY | 91 | 91 | 0 | +0.52 | 33 | 33 | 0 | -1.42 | 0 | 0 | 0 | 0.00 | 73.55 | 73.92 | +0.37 |
| TOON_DEFAULT | 92 | 93 | +1 | +1.10 | 32 | 31 | -1 | -3.19 | 0 | 0 | 0 | 0.00 | 74.29 | 75.12 | +0.83 |
| XML_COMPACT | 94 | 92 | -2 | -2.13 | 30 | 32 | +2 | +6.67 | 0 | 0 | 0 | 0.00 | 75.81 | 74.20 | -1.61 |
| XML_PRETTY | 86 | 86 | 0 | +0.40 | 38 | 38 | 0 | -0.89 | 0 | 0 | 0 | 0.00 | 69.62 | 69.89 | +0.27 |
| YAML | 93 | 81 | -12 | -12.54 | 31 | 43 | +12 | +37.61 | 0 | 0 | 0 | 0.00 | 75.27 | 65.86 | -9.41 |

#### 2.8.3 Characters: Mandatory vs Optional Data
| Format | Output Characters Man | Output Characters Opt | Diff | Diff (%) | Correct Characters Man | Correct Characters Opt | Diff | Diff (%) | Incorrect Characters Man | Incorrect Characters Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 7883 | 8663 | +780 | +9.90 | 7585 | 8329 | +744 | +9.81 | 298 | 334 | +36 | +12.21 | 96.23 | 96.17 | -0.06 |
| JSON_PRETTY | 7952 | 8833 | +881 | +11.08 | 7521 | 8081 | +560 | +7.45 | 431 | 752 | +321 | +74.49 | 94.61 | 91.85 | -2.76 |
| TOON_DEFAULT | 7937 | 8591 | +654 | +8.24 | 7559 | 8282 | +723 | +9.57 | 377 | 308 | -69 | -18.29 | 95.31 | 96.42 | 1.11 |
| XML_COMPACT | 7913 | 8567 | +654 | +8.27 | 7566 | 8292 | +726 | +9.60 | 348 | 276 | -72 | -20.69 | 95.65 | 96.80 | 1.15 |
| XML_PRETTY | 7907 | 8742 | +835 | +10.56 | 7446 | 8066 | +620 | +8.32 | 461 | 677 | +216 | +46.78 | 94.18 | 92.34 | -1.84 |
| YAML | 7877 | 8690 | +813 | +10.32 | 7595 | 8082 | +487 | +6.41 | 282 | 608 | +326 | +115.72 | 96.43 | 93.01 | -3.42 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 75.00 | 96.36 | 56.30 | 58.09 | 60.00 | 94.06 | 36.14 | 16.42 | 12.10 | 7.50 |
| JSON_COMPACT | opt | 76.45 | 98.18 | 60.00 | 63.81 | 53.33 | 93.17 | 36.82 | 17.50 | 13.29 | 6.67 |
| JSON_PRETTY | man | 73.55 | 91.64 | 66.67 | 60.00 | 48.57 | 91.81 | 34.36 | 19.44 | 12.50 | 6.07 |
| JSON_PRETTY | opt | 73.92 | 95.76 | 60.49 | 68.25 | 39.68 | 90.64 | 35.91 | 17.64 | 14.22 | 4.96 |
| TOON_DEFAULT | man | 74.29 | 93.64 | 60.65 | 57.74 | 57.74 | 92.88 | 35.11 | 17.69 | 12.03 | 7.22 |
| TOON_DEFAULT | opt | 75.12 | 93.25 | 68.25 | 66.67 | 44.90 | 92.89 | 34.97 | 19.91 | 13.89 | 5.61 |
| XML_COMPACT | man | 75.81 | 90.91 | 66.67 | 55.55 | 68.25 | 93.45 | 34.09 | 19.44 | 11.57 | 8.53 |
| XML_COMPACT | opt | 74.20 | 94.55 | 70.37 | 61.90 | 38.10 | 92.88 | 35.45 | 20.53 | 12.90 | 4.76 |
| XML_PRETTY | man | 69.62 | 88.48 | 59.26 | 53.97 | 49.21 | 90.99 | 33.18 | 17.28 | 11.24 | 6.15 |
| XML_PRETTY | opt | 69.89 | 87.88 | 61.73 | 61.90 | 41.27 | 88.40 | 32.96 | 18.00 | 12.90 | 5.16 |
| YAML | man | 75.27 | 99.39 | 60.49 | 61.90 | 44.45 | 93.40 | 37.27 | 17.64 | 12.90 | 5.55 |
| YAML | opt | 65.86 | 82.42 | 55.56 | 61.90 | 39.68 | 89.67 | 30.91 | 16.20 | 12.90 | 4.96 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 96.36 | 98.18 | +1.82 | 36.14 | 36.82 | +0.68 |
| JSON_PRETTY | 91.64 | 95.76 | +4.12 | 34.36 | 35.91 | +1.55 |
| TOON_DEFAULT | 93.64 | 93.25 | -0.39 | 35.11 | 34.97 | -0.15 |
| XML_COMPACT | 90.91 | 94.55 | +3.64 | 34.09 | 35.45 | +1.36 |
| XML_PRETTY | 88.48 | 87.88 | -0.61 | 33.18 | 32.96 | -0.23 |
| YAML | 99.39 | 82.42 | -16.97 | 37.27 | 30.91 | -6.36 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 56.30 | 60.00 | +3.70 | 16.42 | 17.50 | +1.08 |
| JSON_PRETTY | 66.67 | 60.49 | -6.17 | 19.44 | 17.64 | -1.80 |
| TOON_DEFAULT | 60.65 | 68.25 | +7.61 | 17.69 | 19.91 | +2.22 |
| XML_COMPACT | 66.67 | 70.37 | +3.70 | 19.44 | 20.53 | +1.08 |
| XML_PRETTY | 59.26 | 61.73 | +2.47 | 17.28 | 18.00 | +0.72 |
| YAML | 60.49 | 55.56 | -4.94 | 17.64 | 16.20 | -1.44 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 58.09 | 63.81 | +5.71 | 12.10 | 13.29 | +1.19 |
| JSON_PRETTY | 60.00 | 68.25 | +8.26 | 12.50 | 14.22 | +1.72 |
| TOON_DEFAULT | 57.74 | 66.67 | +8.93 | 12.03 | 13.89 | +1.86 |
| XML_COMPACT | 55.55 | 61.90 | +6.35 | 11.57 | 12.90 | +1.33 |
| XML_PRETTY | 53.97 | 61.90 | +7.94 | 11.24 | 12.90 | +1.65 |
| YAML | 61.90 | 61.90 | 0.00 | 12.90 | 12.90 | 0.00 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 60.00 | 53.33 | -6.67 | 7.50 | 6.67 | -0.83 |
| JSON_PRETTY | 48.57 | 39.68 | -8.89 | 6.07 | 4.96 | -1.11 |
| TOON_DEFAULT | 57.74 | 44.90 | -12.84 | 7.22 | 5.61 | -1.60 |
| XML_COMPACT | 68.25 | 38.10 | -30.16 | 8.53 | 4.76 | -3.77 |
| XML_PRETTY | 49.21 | 41.27 | -7.93 | 6.15 | 5.16 | -1.00 |
| YAML | 44.45 | 39.68 | -4.76 | 5.55 | 4.96 | -0.59 |

### 2.10 Accuracy By Character Per Question Category Analysis
#### 2.10.1 Metrics
| Format | Variant | Accuracy By Character (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc By Char (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 96.23 | 97.08 | 96.22 | 91.61 | 84.08 | 94.06 | 36.40 | 28.06 | 19.08 | 10.51 |
| JSON_COMPACT | opt | 96.17 | 97.99 | 95.47 | 90.09 | 78.49 | 93.17 | 36.75 | 27.84 | 18.77 | 9.81 |
| JSON_PRETTY | man | 94.61 | 93.05 | 96.42 | 90.36 | 79.75 | 91.81 | 34.89 | 28.12 | 18.82 | 9.97 |
| JSON_PRETTY | opt | 91.85 | 99.09 | 87.31 | 92.04 | 70.71 | 90.64 | 37.16 | 25.47 | 19.18 | 8.84 |
| TOON_DEFAULT | man | 95.31 | 95.36 | 95.93 | 91.41 | 80.80 | 92.88 | 35.76 | 27.98 | 19.04 | 10.10 |
| TOON_DEFAULT | opt | 96.42 | 96.77 | 97.13 | 91.91 | 72.99 | 92.89 | 36.29 | 28.33 | 19.15 | 9.12 |
| XML_COMPACT | man | 95.65 | 95.34 | 96.33 | 89.88 | 87.04 | 93.45 | 35.75 | 28.10 | 18.72 | 10.88 |
| XML_COMPACT | opt | 96.80 | 97.64 | 97.19 | 91.45 | 70.91 | 92.88 | 36.61 | 28.35 | 19.05 | 8.87 |
| XML_PRETTY | man | 94.18 | 92.47 | 96.13 | 88.69 | 78.44 | 90.99 | 34.67 | 28.04 | 18.48 | 9.81 |
| XML_PRETTY | opt | 92.34 | 88.64 | 96.65 | 87.02 | 70.71 | 88.40 | 33.24 | 28.19 | 18.13 | 8.84 |
| YAML | man | 96.43 | 98.69 | 95.76 | 91.07 | 75.93 | 93.40 | 37.01 | 27.93 | 18.97 | 9.49 |
| YAML | opt | 93.01 | 90.43 | 96.13 | 90.27 | 71.31 | 89.67 | 33.91 | 28.04 | 18.80 | 8.91 |

#### 2.10.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 97.08 | 97.99 | +0.91 | 36.40 | 36.75 | +0.34 |
| JSON_PRETTY | 93.05 | 99.09 | +6.04 | 34.89 | 37.16 | +2.26 |
| TOON_DEFAULT | 95.36 | 96.77 | +1.41 | 35.76 | 36.29 | +0.53 |
| XML_COMPACT | 95.34 | 97.64 | +2.30 | 35.75 | 36.61 | +0.86 |
| XML_PRETTY | 92.47 | 88.64 | -3.82 | 34.67 | 33.24 | -1.43 |
| YAML | 98.69 | 90.43 | -8.25 | 37.01 | 33.91 | -3.10 |

#### 2.10.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 96.22 | 95.47 | -0.75 | 28.06 | 27.84 | -0.22 |
| JSON_PRETTY | 96.42 | 87.31 | -9.10 | 28.12 | 25.47 | -2.66 |
| TOON_DEFAULT | 95.93 | 97.13 | +1.20 | 27.98 | 28.33 | +0.35 |
| XML_COMPACT | 96.33 | 97.19 | +0.86 | 28.10 | 28.35 | +0.25 |
| XML_PRETTY | 96.13 | 96.65 | +0.52 | 28.04 | 28.19 | +0.16 |
| YAML | 95.76 | 96.13 | +0.37 | 27.93 | 28.04 | +0.11 |

#### 2.10.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 91.61 | 90.09 | -1.52 | 19.08 | 18.77 | -0.31 |
| JSON_PRETTY | 90.36 | 92.04 | +1.68 | 18.82 | 19.18 | +0.35 |
| TOON_DEFAULT | 91.41 | 91.91 | +0.51 | 19.04 | 19.15 | +0.11 |
| XML_COMPACT | 89.88 | 91.45 | +1.57 | 18.72 | 19.05 | +0.33 |
| XML_PRETTY | 88.69 | 87.02 | -1.67 | 18.48 | 18.13 | -0.35 |
| YAML | 91.07 | 90.27 | -0.80 | 18.97 | 18.80 | -0.17 |

#### 2.10.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | 84.08 | 78.49 | -5.59 | 10.51 | 9.81 | -0.70 |
| JSON_PRETTY | 79.75 | 70.71 | -9.05 | 9.97 | 8.84 | -1.13 |
| TOON_DEFAULT | 80.80 | 72.99 | -7.82 | 10.10 | 9.12 | -0.98 |
| XML_COMPACT | 87.04 | 70.91 | -16.13 | 10.88 | 8.87 | -2.01 |
| XML_PRETTY | 78.44 | 70.71 | -7.74 | 9.81 | 8.84 | -0.97 |
| YAML | 75.93 | 71.31 | -4.61 | 9.49 | 8.91 | -0.57 |

## 3. Appendices

### 3.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: off
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
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_haiku_4_5/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)