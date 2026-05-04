# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: off
- **Data Structure**: flat
- **Formats Tested**: 7 (CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: First iteration

## Executive Summary

This benchmark evaluates token efficiency and information accuracy across 7 file formats using Claude Haiku 4.5 (claude-haiku-4-5-20251001) as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context while a format that accurately conveys information may justify higher token cost.

### Key Findings

1. With the lowest read tokens (7063 mandatory, 6795 optional) and lowest total tokens (17225 mandatory, 18645 optional) **CSV** consistently minimizes token cost which results in the best token efficiency overall. It achieves 1.43 characters per read token and only 10.36 read tokens per value which makes it the densest encoding tested.
2. **JSON_PRETTY** achieves the highest accuracy on mandatory data (81.18%) but requires a lot of token. It consumes roughly double the read tokens of **CSV** (14311 vs 7063) resulting in the lowest total efficiency score (55.89). **YAML** takes the accuracy lead at 79.84% for optional data.
3. Accuracy by character reveals that most errors are minor which is not expected if the accuracy was only judged by the correct answers. The gap between complete answer accuracy and by-character accuracy ranges from 11 percentage points (**CSV** mandatory: 77.15% vs 88.98%) to nearly 20 percentage points (**JSON_COMPACT** mandatory: 77.62% vs 97.50%). **JSON_COMPACT** and **YAML** produce answers that are character-level precise even when counted as "wrong" by exact match which indicates the model understands the data but occasionally misses words, makes small formatting or rounding mistakes.
4. **TOON**'s adaptive encoding creates a dramatic token penalty when data becomes sparse. Read tokens jump from 7182 (mandatory) to 11709 (optional) which is a 63% increase that is 10x larger than any other format's mandatory-to-optional swing. This happens because **TOON** falls back from its compact tabular layout to explicit key-value pairs when fields are missing and it looses the header-based compression that makes it competitive with **CSV** on dense data.
5. Aggregation is the weakest category for the model across all formats regardless of encoding. Accuracy ranges from 40.48% (**JSON_COMPACT** mandatory) to 68.26% (**CSV** mandatory) which is far below field retrieval (88-99%) and structure awareness (49-79%). This pattern confirms that computational reasoning and not data comprehension is the bottleneck for the model running without thinking.
6. **XML_COMPACT** produces a unique output token profile. It uses the fewest output tokens on mandatory data (6976) despite having relatively high read tokens (11950). This suggests the model can process XML's explicit structure quickly but the format's high read token cost (2.32 chars per read token) limits its overall efficiency.
7. No single format dominates all metrics simultaneously: **CSV** wins on token cost and total efficiency scores and **JSON_PRETTY** wins on accuracy. **JSON_COMPACT** wins on by-character accuracy for mandatory data (97.50%) and total efficiency for optional data. **YAML** wins on accuracy by character for optional data (98.41%). The optimal choice depends on whether the use case prioritizes minimizing token spend, maximizing answer correctness or balancing both.

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
| YAML ≈ 76s | CSV ≈ 7063 | TOON_DEFAULT ≈ 258 | XML_COMPACT ≈ 6673 | XML_COMPACT ≈ 6976 | CSV ≈ 17225 | JSON_PRETTY ≈ 81.18% | CSV ≈ 84 | XML_COMPACT ≈ 82 | CSV ≈ 85 | JSON_COMPACT ≈ 97.50% | TOON_DEFAULT ≈ 94 | XML_COMPACT ≈ 96 | CSV ≈ 93 |
| XML_COMPACT (+5.78%) | TOON_DEFAULT (+1.68%) | CSV (+16.50%) | YAML (+35.93%) | YAML (+34.36%) | TOON_DEFAULT (+9.13%) | JSON_COMPACT (-3.56%) | TOON_DEFAULT (-1.93%) | YAML (-14.97%) | TOON_DEFAULT (-7.86%) | JSON_PRETTY (-0.51%) | CSV (-2.20%) | YAML (-11.51%) | XML_COMPACT (-2.42%) |
| CSV (+8.43%) | JSON_COMPACT (+31.62%) | JSON_PRETTY (+17.15%) | CSV (+47.78%) | CSV (+45.67%) | XML_COMPACT (+9.88%) | CSV (-4.03%) | JSON_COMPACT (-9.07%) | CSV (-16.18%) | XML_COMPACT (-10.58%) | YAML (-1.29%) | JSON_COMPACT (-4.57%) | XML_PRETTY (-18.48%) | TOON_DEFAULT (-3.23%) |
| XML_PRETTY (+13.92%) | XML_COMPACT (+69.19%) | YAML (+17.15%) | XML_PRETTY (+54.29%) | XML_PRETTY (+51.95%) | YAML (+27.41%) | TOON_DEFAULT (-5.82%) | XML_COMPACT (-24.30%) | XML_PRETTY (-21.33%) | JSON_COMPACT (-22.46%) | XML_PRETTY (-1.95%) | XML_COMPACT (-16.74%) | CSV (-20.70%) | YAML (-12.53%) |
| TOON_DEFAULT (+25.23%) | YAML (+78.03%) | XML_COMPACT (+17.54%) | TOON_DEFAULT (+70.19%) | TOON_DEFAULT (+66.50%) | JSON_COMPACT (+32.28%) | XML_PRETTY (-6.99%) | YAML (-26.93%) | JSON_PRETTY (-24.22%) | YAML (-22.98%) | XML_COMPACT (-3.01%) | YAML (-17.88%) | JSON_PRETTY (-24.77%) | JSON_COMPACT (-14.75%) |
| JSON_PRETTY (+26.80%) | JSON_PRETTY (+102.62%) | JSON_COMPACT (+18.06%) | JSON_PRETTY (+74.95%) | JSON_PRETTY (+71.68%) | JSON_PRETTY (+52.61%) | XML_COMPACT (-8.60%) | JSON_PRETTY (-27.43%) | TOON_DEFAULT (-26.71%) | JSON_PRETTY (-34.04%) | TOON_DEFAULT (-4.80%) | JSON_PRETTY (-23.89%) | TOON_DEFAULT (-25.83%) | JSON_PRETTY (-28.28%) |
| JSON_COMPACT (+40.46%) | XML_PRETTY (+129.17%) | XML_PRETTY (+18.19%) | JSON_COMPACT (+97.57%) | JSON_COMPACT (+93.36%) | XML_PRETTY (+55.51%) | YAML (-8.60%) | XML_PRETTY (-40.92%) | JSON_COMPACT (-36.58%) | XML_PRETTY (-41.59%) | CSV (-8.52%) | XML_PRETTY (-32.00%) | JSON_COMPACT (-32.43%) | XML_PRETTY (-31.19%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 74s | CSV ≈ 6795 | JSON_PRETTY ≈ 231 | TOON_DEFAULT ≈ 8675 | TOON_DEFAULT ≈ 8984 | JSON_COMPACT ≈ 18326 | YAML ≈ 79.84% | CSV ≈ 82 | TOON_DEFAULT ≈ 73 | JSON_COMPACT ≈ 80 | YAML ≈ 98.41% | CSV ≈ 95 | TOON_DEFAULT ≈ 87 | JSON_COMPACT ≈ 94 |
| XML_PRETTY (+0.30%) | JSON_COMPACT (+29.04%) | CSV (+29.29%) | XML_PRETTY (+2.68%) | XML_PRETTY (+2.51%) | CSV (+1.74%) | JSON_PRETTY (-1.77%) | JSON_COMPACT (-6.07%) | XML_PRETTY (-0.44%) | CSV (-3.89%) | XML_COMPACT (-0.55%) | JSON_COMPACT (-4.99%) | XML_PRETTY (-1.01%) | CSV (-3.58%) |
| JSON_COMPACT (+2.14%) | XML_COMPACT (+64.55%) | XML_PRETTY (+30.74%) | JSON_COMPACT (+6.66%) | JSON_COMPACT (+6.38%) | TOON_DEFAULT (+12.92%) | XML_COMPACT (-1.88%) | XML_COMPACT (-15.00%) | JSON_COMPACT (-2.82%) | TOON_DEFAULT (-11.35%) | JSON_PRETTY (-1.15%) | XML_COMPACT (-12.83%) | JSON_COMPACT (-2.61%) | TOON_DEFAULT (-9.51%) |
| JSON_PRETTY (+14.79%) | TOON_DEFAULT (+72.32%) | YAML (+31.60%) | JSON_PRETTY (+17.35%) | JSON_PRETTY (+15.88%) | XML_COMPACT (+22.49%) | JSON_COMPACT (-3.71%) | YAML (-16.11%) | JSON_PRETTY (-7.01%) | XML_COMPACT (-16.35%) | JSON_COMPACT (-2.17%) | YAML (-14.71%) | JSON_PRETTY (-6.86%) | XML_COMPACT (-14.16%) |
| XML_COMPACT (+18.40%) | YAML (+73.52%) | JSON_COMPACT (+31.95%) | XML_COMPACT (+26.30%) | XML_COMPACT (+25.39%) | JSON_PRETTY (+29.90%) | XML_PRETTY (-3.77%) | TOON_DEFAULT (-19.83%) | XML_COMPACT (-13.10%) | JSON_PRETTY (-22.15%) | XML_PRETTY (-2.75%) | TOON_DEFAULT (-16.63%) | XML_COMPACT (-11.44%) | JSON_PRETTY (-19.64%) |
| CSV (+26.29%) | JSON_PRETTY (+97.13%) | XML_COMPACT (+33.77%) | CSV (+33.16%) | CSV (+31.90%) | XML_PRETTY (+32.75%) | TOON_DEFAULT (-5.02%) | JSON_PRETTY (-24.47%) | CSV (-21.61%) | YAML (-25.08%) | TOON_DEFAULT (-3.17%) | JSON_PRETTY (-21.48%) | CSV (-18.72%) | YAML (-22.59%) |
| YAML (+40.42%) | XML_PRETTY (+122.47%) | TOON_DEFAULT (+33.84%) | YAML (+46.69%) | YAML (+45.02%) | YAML (+35.44%) | CSV (-6.72%) | XML_PRETTY (-33.53%) | YAML (-23.72%) | XML_PRETTY (-26.07%) | CSV (-5.53%) | XML_PRETTY (-29.01%) | YAML (-21.41%) | XML_PRETTY (-22.71%) |


#### 2.1.2 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 98.18% | JSON_PRETTY ≈ 77.78% | JSON_PRETTY ≈ 66.67% | CSV ≈ 68.26% |
| YAML (-1.21%) | JSON_COMPACT (-1.85%) | XML_PRETTY (-3.17%) | JSON_PRETTY (-9.53%) |
| JSON_PRETTY (-1.21%) | TOON_DEFAULT (-4.94%) | JSON_COMPACT (-3.57%) | XML_COMPACT (-11.11%) |
| CSV (-2.42%) | XML_COMPACT (-13.58%) | YAML (-4.76%) | TOON_DEFAULT (-19.05%) |
| XML_PRETTY (-2.42%) | CSV (-18.52%) | CSV (-6.35%) | YAML (-19.05%) |
| TOON_DEFAULT (-5.86%) | XML_PRETTY (-18.52%) | TOON_DEFAULT (-6.35%) | XML_PRETTY (-20.63%) |
| XML_COMPACT (-9.69%) | YAML (-28.40%) | XML_COMPACT (-9.53%) | JSON_COMPACT (-27.78%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99.39% | XML_COMPACT ≈ 79.01% | YAML ≈ 73.02% | JSON_COMPACT ≈ 52.38% |
| XML_COMPACT (-1.82%) | XML_PRETTY (-1.23%) | JSON_PRETTY (-6.35%) | XML_PRETTY (0.00%) |
| JSON_PRETTY (-2.67%) | JSON_PRETTY (-6.42%) | JSON_COMPACT (-11.11%) | TOON_DEFAULT (-4.76%) |
| JSON_COMPACT (-4.48%) | YAML (-7.41%) | XML_COMPACT (-11.11%) | CSV (-4.76%) |
| TOON_DEFAULT (-6.06%) | TOON_DEFAULT (-9.88%) | CSV (-11.11%) | JSON_PRETTY (-4.76%) |
| XML_PRETTY (-8.49%) | CSV (-11.11%) | TOON_DEFAULT (-12.17%) | YAML (-6.35%) |
| CSV (-9.69%) | JSON_COMPACT (-11.61%) | XML_PRETTY (-14.29%) | XML_COMPACT (-11.11%) |


#### 2.1.3 Category Accuracy By Character Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 98.37% | JSON_PRETTY ≈ 98.69% | XML_PRETTY ≈ 92.86% | CSV ≈ 85.18% |
| JSON_COMPACT (-0.49%) | JSON_COMPACT (-0.41%) | TOON_DEFAULT (-0.79%) | XML_COMPACT (-2.06%) |
| XML_PRETTY (-1.61%) | XML_COMPACT (-1.57%) | CSV (-1.19%) | JSON_PRETTY (-6.38%) |
| JSON_PRETTY (-2.64%) | YAML (-3.11%) | JSON_COMPACT (-1.78%) | XML_PRETTY (-7.82%) |
| CSV (-3.13%) | XML_PRETTY (-3.24%) | JSON_PRETTY (-1.79%) | TOON_DEFAULT (-7.94%) |
| TOON_DEFAULT (-5.08%) | TOON_DEFAULT (-5.62%) | YAML (-2.63%) | YAML (-8.36%) |
| XML_COMPACT (-6.67%) | CSV (-12.88%) | XML_COMPACT (-4.46%) | JSON_COMPACT (-11.73%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99.85% | XML_PRETTY ≈ 98.45% | YAML ≈ 92.33% | XML_PRETTY ≈ 75.96% |
| XML_COMPACT (-0.81%) | YAML (-0.11%) | JSON_PRETTY (-0.65%) | JSON_COMPACT (-0.32%) |
| JSON_PRETTY (-1.91%) | XML_COMPACT (-0.18%) | CSV (-1.48%) | JSON_PRETTY (-0.69%) |
| JSON_COMPACT (-3.27%) | JSON_PRETTY (-0.80%) | TOON_DEFAULT (-3.05%) | TOON_DEFAULT (-1.10%) |
| TOON_DEFAULT (-5.20%) | CSV (-1.10%) | JSON_COMPACT (-3.13%) | CSV (-2.83%) |
| XML_PRETTY (-6.29%) | JSON_COMPACT (-1.56%) | XML_PRETTY (-4.43%) | YAML (-4.44%) |
| CSV (-11.02%) | TOON_DEFAULT (-1.73%) | XML_COMPACT (-6.49%) | XML_COMPACT (-7.68%) |


#### 2.1.4 Conclusion

The ranking tables reveals that the format choice depends on the use case and constraints.

- **CSV** achieves a high total efficiency scores (84.73 mandatory, 77.10 optional) because its read tokens are minimal compared to most other formats. **JSON_COMPACT** trades higher read tokens (+32% over **CSV**) for stronger accuracy by character (97.50% mandatory, 96.24% optional) and achieves the best total efficiency on optional data (80.22 and 93.63 by character). Both formats deliver high information value per token without sacrificing meaningful accuracy.
- **JSON_PRETTY** produces the highest accuracy by answer on mandatory data (81.18%) and **YAML** leads on optional data (79.84%) but both pay for it with excessive read tokens (14311 and 12574 mandatory). Their total efficiency scores fall to 55.89 and 65.26 respectively. These formats are appropriate when answer correctness matters more than token budget for example in scenarios where wrong answers trigger expensive downstream corrections.
- **TOON_DEFAULT** matches **CSV**'s read token cost on dense mandatory data (7182 vs 7063) and achieves a strong read efficiency score (82.17) but its adaptive encoding causes a 63% token increase on sparse optional data which destroys the efficiency advantage. This volatility makes **TOON_DEFAULT** unpredictable for mixed workloads. **XML_COMPACT** shows an interesting output token advantage (fewest output tokens on mandatory data at 6976) but its high read token cost (11950) limits total efficiency. **XML_PRETTY** consistently ranks last or second-to-last on both token cost and efficiency scores.
- Field retrieval accuracy exceeds 88% across all formats which confirms that the model can reliably extract specific values from flat data regardless of encoding. The differentiating categories are structure awareness and aggregation. **YAML**'s structure awareness drops to 49.38% on mandatory data (the lowest recorded value) but recovers to 71.60% on optional data which is a 22 percentage point swing that suggests **YAML**'s indentation-based structure becomes harder to parse when more data is present. Aggregation accuracy peaks at 68.26% (**CSV** mandatory) and drops to 40.48% (**JSON_COMPACT** mandatory) which indicates it is a model capability limitation rather than a format-dependent outcome.
- Some formats show a small accuracy drop (1 to 4 percentage points) when moving from mandatory to optional data which is expected because sparse records introduce ambiguity about absent versus missing values. Notable exceptions are **XML_COMPACT** (+5.38%), **XML_PRETTY** (+1.88%) and **YAML** (+7.26%) which improve on optional data. This improvement correlates with these formats using explicit field labeling per value (**XML** tags, **YAML** keys) which makes the presence or absence of each field unambiguous even when records are incomplete.
- **CSV** is the default choice when token budget is the primary constraint. **JSON_COMPACT** is preferred when accuracy by character matters and the slight token premium is acceptable. **JSON_PRETTY** should be reserved for cases where accuracy by answer is critical and the token overhead (roughly 50% more than **CSV**) is tolerable. **TOON_DEFAULT** should only be used when the data is guaranteed to be dense because its token cost becomes unpredictable with sparse data.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total | Accuracy By Character (%) | Useful Read Tokens (Acc By Char) | Wasted Read Tokens (Acc By Char) | Useful Output Tokens (Acc By Char) | Wasted Output Tokens (Acc By Char) | Eff Score Read (Acc By Char) | Eff Score Output (Acc By Char) | Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7063 | 10162 | 17225 | 1.43 | 79.53 | 77.15 | 5449 | 1614 | 7840 | 2322 | 83.78 | 68.46 | 84.73 | 88.98 | 6285 | 778 | 9042 | 1120 | 91.67 | 76.34 | 92.62 |
| CSV | opt | 6795 | 11850 | 18645 | 1.41 | 93.16 | 73.12 | 4969 | 1826 | 8665 | 3185 | 82.04 | 57.16 | 77.10 | 92.88 | 6311 | 484 | 11006 | 844 | 95.22 | 70.33 | 90.28 |
| JSON_COMPACT | man | 9296 | 13489 | 22785 | 2.14 | 106.33 | 77.62 | 7216 | 2080 | 10470 | 3019 | 76.19 | 51.80 | 65.70 | 97.50 | 9064 | 232 | 13151 | 337 | 89.44 | 65.05 | 78.96 |
| JSON_COMPACT | opt | 8768 | 9558 | 18326 | 2.11 | 74.62 | 76.13 | 6675 | 2093 | 7276 | 2281 | 77.06 | 70.86 | 80.22 | 96.24 | 8438 | 330 | 9198 | 359 | 90.47 | 84.27 | 93.63 |
| JSON_PRETTY | man | 14311 | 11977 | 26288 | 1.69 | 94.15 | 81.18 | 11618 | 2693 | 9723 | 2254 | 60.80 | 61.89 | 55.89 | 96.99 | 13880 | 431 | 11616 | 360 | 71.34 | 72.43 | 66.43 |
| JSON_PRETTY | opt | 13395 | 10411 | 23806 | 1.68 | 82.10 | 78.07 | 10457 | 2938 | 8128 | 2283 | 61.97 | 67.80 | 62.45 | 97.26 | 13028 | 367 | 10126 | 285 | 74.76 | 80.59 | 75.24 |
| TOON_DEFAULT | man | 7182 | 11615 | 18797 | 1.42 | 91.59 | 75.36 | 5412 | 1770 | 8753 | 2862 | 82.17 | 59.85 | 78.07 | 92.70 | 6658 | 524 | 10767 | 848 | 93.73 | 71.41 | 89.63 |
| TOON_DEFAULT | opt | 11709 | 8984 | 20693 | 1.68 | 69.96 | 74.82 | 8761 | 2948 | 6722 | 2262 | 65.77 | 72.91 | 71.11 | 95.24 | 11152 | 557 | 8557 | 428 | 79.39 | 86.53 | 84.73 |
| XML_COMPACT | man | 11950 | 6976 | 18926 | 2.32 | 53.82 | 72.58 | 8673 | 3277 | 5063 | 1913 | 63.43 | 81.67 | 75.77 | 94.49 | 11292 | 658 | 6592 | 384 | 78.03 | 96.28 | 90.37 |
| XML_COMPACT | opt | 11181 | 11266 | 22447 | 2.29 | 88.36 | 77.96 | 8717 | 2464 | 8783 | 2483 | 69.74 | 63.37 | 67.11 | 97.86 | 10942 | 239 | 11025 | 241 | 83.00 | 76.63 | 80.37 |
| XML_PRETTY | man | 16186 | 10600 | 26786 | 1.93 | 83.03 | 74.19 | 12008 | 4178 | 7864 | 2736 | 49.50 | 64.25 | 49.49 | 95.55 | 15466 | 720 | 10129 | 472 | 63.74 | 78.49 | 63.73 |
| XML_PRETTY | opt | 15117 | 9210 | 24327 | 1.91 | 71.84 | 76.07 | 11500 | 3617 | 7006 | 2204 | 54.54 | 72.60 | 59.31 | 95.66 | 14461 | 656 | 8810 | 400 | 67.59 | 85.66 | 72.36 |
| YAML | man | 12574 | 9373 | 21947 | 1.66 | 73.15 | 72.58 | 9126 | 3448 | 6803 | 2570 | 61.22 | 69.44 | 65.26 | 96.21 | 12097 | 477 | 9017 | 355 | 76.97 | 85.19 | 81.01 |
| YAML | opt | 11791 | 13029 | 24820 | 1.65 | 102.62 | 79.84 | 9414 | 2377 | 10402 | 2627 | 68.83 | 55.62 | 60.10 | 98.41 | 11604 | 187 | 12822 | 207 | 81.21 | 68.00 | 72.48 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7063 | 6795 | -268 | -3.79 | 300 | 299 | -1 | -0.33 | 9862 | 11551 | +1689 | +17.13 | 10162 | 11850 | +1688 | +16.61 | 17225 | 18645 | +1420 | +8.24 |
| JSON_COMPACT | 9296 | 8768 | -528 | -5.68 | 304 | 305 | +1 | +0.33 | 13185 | 9254 | -3931 | -29.81 | 13489 | 9558 | -3931 | -29.14 | 22785 | 18326 | -4459 | -19.57 |
| JSON_PRETTY | 14311 | 13395 | -916 | -6.40 | 302 | 231 | -71 | -23.51 | 11675 | 10180 | -1495 | -12.81 | 11977 | 10411 | -1566 | -13.08 | 26288 | 23806 | -2482 | -9.44 |
| TOON_DEFAULT | 7182 | 11709 | +4527 | +63.03 | 258 | 310 | +52 | +20.16 | 11358 | 8676 | -2682 | -23.61 | 11615 | 8984 | -2631 | -22.65 | 18797 | 20693 | +1896 | +10.09 |
| XML_COMPACT | 11950 | 11181 | -769 | -6.44 | 303 | 309 | +6 | +1.98 | 6673 | 10956 | +4283 | +64.18 | 6976 | 11266 | +4290 | +61.50 | 18926 | 22447 | +3521 | +18.60 |
| XML_PRETTY | 16186 | 15117 | -1069 | -6.60 | 304 | 302 | -2 | -0.66 | 10296 | 8908 | -1388 | -13.48 | 10600 | 9209 | -1391 | -13.12 | 26786 | 24326 | -2460 | -9.18 |
| YAML | 12574 | 11791 | -783 | -6.23 | 302 | 304 | +2 | +0.66 | 9071 | 12725 | +3654 | +40.28 | 9373 | 13029 | +3656 | +39.01 | 21947 | 24820 | +2873 | +13.09 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Before Write (ms) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) | Output (ms) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 92 | 76.77 | 2.97 | 48734 | 33545 | 0.29 | 270.52 | 33637 | 77.07 | 217.01 | 82279 |
| CSV | opt | 15 | 453.00 | 0.48 | 64296 | 29220 | 0.40 | 235.65 | 29235 | 453.40 | 188.61 | 93516 |
| JSON_COMPACT | man | 36 | 258.22 | 1.16 | 71360 | 35225 | 0.37 | 284.07 | 35261 | 258.60 | 227.49 | 106585 |
| JSON_COMPACT | opt | 9 | 974.22 | 0.29 | 40073 | 35563 | 0.26 | 286.80 | 35572 | 974.48 | 229.50 | 75636 |
| JSON_PRETTY | man | 13 | 1100.85 | 0.42 | 60406 | 35817 | 0.33 | 288.85 | 35830 | 1101.17 | 231.16 | 96223 |
| JSON_PRETTY | opt | 15 | 893.00 | 0.48 | 47655 | 37347 | 0.27 | 301.19 | 37362 | 893.27 | 241.05 | 85002 |
| TOON_DEFAULT | man | 27 | 266.00 | 0.87 | 56899 | 38129 | 0.30 | 307.49 | 38156 | 266.30 | 246.17 | 95028 |
| TOON_DEFAULT | opt | 27 | 433.67 | 0.87 | 37002 | 37047 | 0.24 | 298.77 | 37074 | 433.90 | 239.19 | 74050 |
| XML_COMPACT | man | 16 | 746.88 | 0.52 | 32957 | 47309 | 0.14 | 381.52 | 47325 | 747.02 | 305.32 | 80267 |
| XML_COMPACT | opt | 15 | 745.40 | 0.48 | 52664 | 35007 | 0.31 | 282.31 | 35022 | 745.71 | 225.95 | 87671 |
| XML_PRETTY | man | 18 | 899.22 | 0.58 | 50051 | 36393 | 0.28 | 293.49 | 36411 | 899.51 | 234.91 | 86445 |
| XML_PRETTY | opt | 9 | 1679.67 | 0.29 | 38045 | 36231 | 0.25 | 292.19 | 36240 | 1679.91 | 233.81 | 74275 |
| YAML | man | 11 | 1143.09 | 0.35 | 41079 | 34805 | 0.26 | 280.69 | 34816 | 1143.35 | 224.62 | 75884 |
| YAML | opt | 8 | 1473.88 | 0.26 | 65990 | 37991 | 0.34 | 306.38 | 37999 | 1474.21 | 245.15 | 103981 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Before Write Man (s) | Output Before Write Opt (s) | Diff (s) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) | Output Man (s) | Output Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 92 | 15 | -77 | -83.70 | 48.73 | 64.30 | +15.56 | +31.93 | 33.54 | 29.22 | -4.32 | -12.89 | 33.64 | 29.24 | -4.40 | -13.09 | 82.28 | 93.52 | +11.24 | +13.66 |
| JSON_COMPACT | 36 | 9 | -27 | -75.00 | 71.36 | 40.07 | -31.29 | -43.84 | 35.22 | 35.56 | +0.34 | +0.96 | 35.26 | 35.57 | +0.31 | +0.88 | 106.59 | 75.64 | -30.95 | -29.04 |
| JSON_PRETTY | 13 | 15 | +2 | +15.38 | 60.41 | 47.66 | -12.75 | -21.11 | 35.82 | 37.35 | +1.53 | +4.27 | 35.83 | 37.36 | +1.53 | +4.28 | 96.22 | 85.00 | -11.22 | -11.66 |
| TOON_DEFAULT | 27 | 27 | 0 | 0.00 | 56.90 | 37.00 | -19.90 | -34.97 | 38.13 | 37.05 | -1.08 | -2.84 | 38.16 | 37.07 | -1.08 | -2.83 | 95.03 | 74.05 | -20.98 | -22.08 |
| XML_COMPACT | 16 | 15 | -1 | -6.25 | 32.96 | 52.66 | +19.71 | +59.79 | 47.31 | 35.01 | -12.30 | -26.00 | 47.33 | 35.02 | -12.30 | -26.00 | 80.27 | 87.67 | +7.40 | +9.22 |
| XML_PRETTY | 18 | 9 | -9 | -50.00 | 50.05 | 38.04 | -12.01 | -23.99 | 36.39 | 36.23 | -0.16 | -0.45 | 36.41 | 36.24 | -0.17 | -0.47 | 86.44 | 74.28 | -12.17 | -14.08 |
| YAML | 11 | 8 | -3 | -27.27 | 41.08 | 65.99 | +24.91 | +60.64 | 34.81 | 37.99 | +3.19 | +9.15 | 34.82 | 38.00 | +3.18 | +9.14 | 75.88 | 103.98 | +28.10 | +37.03 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token | Info / Read Token (Acc By Char) | Info / Output Token (Acc By Char) | Info / Total Token (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 1.43 | 10.36 | 227.84 | 1.09 | 0.76 | 0.45 | 1.26 | 0.88 | 0.52 |
| CSV | opt | 1.41 | 10.77 | 219.19 | 1.08 | 0.62 | 0.39 | 1.37 | 0.78 | 0.50 |
| JSON_COMPACT | man | 2.14 | 13.63 | 299.87 | 0.84 | 0.57 | 0.34 | 1.05 | 0.72 | 0.43 |
| JSON_COMPACT | opt | 2.11 | 13.90 | 282.84 | 0.87 | 0.80 | 0.42 | 1.10 | 1.01 | 0.53 |
| JSON_PRETTY | man | 1.69 | 20.98 | 461.65 | 0.57 | 0.68 | 0.31 | 0.68 | 0.81 | 0.37 |
| JSON_PRETTY | opt | 1.68 | 21.23 | 432.10 | 0.58 | 0.75 | 0.33 | 0.73 | 0.93 | 0.41 |
| TOON_DEFAULT | man | 1.42 | 10.53 | 231.68 | 1.05 | 0.65 | 0.40 | 1.29 | 0.80 | 0.49 |
| TOON_DEFAULT | opt | 1.68 | 18.56 | 377.71 | 0.64 | 0.86 | 0.36 | 0.81 | 1.09 | 0.46 |
| XML_COMPACT | man | 2.32 | 17.52 | 385.48 | 0.61 | 1.04 | 0.38 | 0.79 | 1.36 | 0.50 |
| XML_COMPACT | opt | 2.29 | 17.72 | 360.68 | 0.70 | 0.69 | 0.35 | 0.88 | 0.87 | 0.44 |
| XML_PRETTY | man | 1.93 | 23.73 | 522.13 | 0.46 | 0.70 | 0.28 | 0.59 | 0.90 | 0.36 |
| XML_PRETTY | opt | 1.91 | 23.96 | 487.65 | 0.50 | 0.83 | 0.31 | 0.63 | 1.04 | 0.39 |
| YAML | man | 1.66 | 18.44 | 405.61 | 0.58 | 0.77 | 0.33 | 0.77 | 1.03 | 0.44 |
| YAML | opt | 1.65 | 18.69 | 380.36 | 0.68 | 0.61 | 0.32 | 0.84 | 0.76 | 0.40 |

#### 2.5.2 Characters And Values: Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.43 | 1.41 | -0.02 | -1.40 | 10.36 | 10.77 | +0.41 | +3.99 | 227.84 | 219.19 | -8.65 | -3.79 |
| JSON_COMPACT | 2.14 | 2.11 | -0.03 | -1.63 | 13.63 | 13.90 | +0.26 | +1.94 | 299.87 | 282.84 | -17.03 | -5.68 |
| JSON_PRETTY | 1.69 | 1.68 | -0.01 | -0.83 | 20.98 | 21.23 | +0.24 | +1.16 | 461.65 | 432.10 | -29.55 | -6.40 |
| TOON_DEFAULT | 1.42 | 1.68 | +0.26 | +18.52 | 10.53 | 18.56 | +8.03 | +76.20 | 231.68 | 377.71 | +146.03 | +63.03 |
| XML_COMPACT | 2.32 | 2.29 | -0.03 | -1.17 | 17.52 | 17.72 | +0.20 | +1.12 | 385.48 | 360.68 | -24.81 | -6.44 |
| XML_PRETTY | 1.93 | 1.91 | -0.02 | -0.93 | 23.73 | 23.96 | +0.22 | +0.94 | 522.13 | 487.65 | -34.48 | -6.60 |
| YAML | 1.66 | 1.65 | -0.02 | -0.90 | 18.44 | 18.69 | +0.25 | +1.35 | 405.61 | 380.36 | -25.26 | -6.23 |

#### 2.5.3 Information: Mandatory vs Optional
| Format | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.09 | 1.08 | -0.02 | -1.47 | 0.76 | 0.62 | -0.14 | -18.71 | 0.45 | 0.39 | -0.06 | -12.50 |
| JSON_COMPACT | 0.84 | 0.87 | +0.03 | +3.95 | 0.57 | 0.80 | +0.22 | +38.61 | 0.34 | 0.42 | +0.07 | +21.70 |
| JSON_PRETTY | 0.57 | 0.58 | +0.02 | +2.82 | 0.68 | 0.75 | +0.07 | +10.62 | 0.31 | 0.33 | +0.02 | +6.15 |
| TOON_DEFAULT | 1.05 | 0.64 | -0.41 | -39.08 | 0.65 | 0.86 | +0.21 | +31.90 | 0.40 | 0.36 | -0.04 | -9.46 |
| XML_COMPACT | 0.61 | 0.70 | +0.09 | +14.83 | 1.04 | 0.69 | -0.35 | -33.46 | 0.38 | 0.35 | -0.04 | -9.40 |
| XML_PRETTY | 0.46 | 0.50 | +0.04 | +9.83 | 0.70 | 0.83 | +0.13 | +18.00 | 0.28 | 0.31 | +0.04 | +13.00 |
| YAML | 0.58 | 0.68 | +0.10 | +17.33 | 0.77 | 0.61 | -0.16 | -20.80 | 0.33 | 0.32 | -0.01 | -2.72 |

#### 2.5.4 Information (Accuracy By Character): Mandatory vs Optional
| Format | Info / Read Token (Acc By Char) Man | Info / Read Token (Acc By Char) Opt | Diff | Diff (%) | Info / Output Token (Acc By Char) Man | Info / Output Token (Acc By Char)  Opt | Diff | Diff (%) | Info / Total Token (Acc By Char) Man | Info / Total Token (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.26 | 1.37 | +0.11 | +8.49 | 0.88 | 0.78 | -0.09 | -10.50 | 0.52 | 0.50 | -0.02 | -3.68 |
| JSON_COMPACT | 1.05 | 1.10 | +0.05 | +4.67 | 0.72 | 1.01 | +0.28 | +39.28 | 0.43 | 0.53 | +0.10 | +22.66 |
| JSON_PRETTY | 0.68 | 0.73 | +0.05 | +7.08 | 0.81 | 0.93 | +0.12 | +15.31 | 0.37 | 0.41 | +0.04 | +10.84 |
| TOON_DEFAULT | 1.29 | 0.81 | -0.48 | -37.03 | 0.80 | 1.09 | +0.29 | +36.59 | 0.49 | 0.46 | -0.03 | -6.18 |
| XML_COMPACT | 0.79 | 0.88 | +0.08 | +10.62 | 1.36 | 0.87 | -0.49 | -35.87 | 0.50 | 0.44 | -0.06 | -12.63 |
| XML_PRETTY | 0.59 | 0.63 | +0.04 | +7.29 | 0.90 | 1.04 | +0.14 | +15.32 | 0.36 | 0.39 | +0.04 | +10.08 |
| YAML | 0.77 | 0.84 | +0.07 | +9.15 | 1.03 | 0.76 | -0.27 | -26.41 | 0.44 | 0.40 | -0.04 | -9.59 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7063 | 5449 | 1614 | 10162 | 7840 | 2322 | 17225 | 13289 | 3936 | 77.15 | 83.78 | 68.46 | 84.73 | 74.29 | 81.88 | 66.55 | 82.82 |
| CSV | opt | 6795 | 4969 | 1826 | 11850 | 8665 | 3185 | 18645 | 13633 | 5012 | 73.12 | 82.04 | 57.16 | 77.10 | 72.28 | 81.48 | 56.60 | 76.54 |
| JSON_COMPACT | man | 9296 | 7216 | 2080 | 13489 | 10470 | 3019 | 22785 | 17685 | 5099 | 77.62 | 76.19 | 51.80 | 65.70 | 77.17 | 75.89 | 51.50 | 65.40 |
| JSON_COMPACT | opt | 8768 | 6675 | 2093 | 9558 | 7276 | 2281 | 18326 | 13951 | 4374 | 76.13 | 77.06 | 70.86 | 80.22 | 74.69 | 76.10 | 69.90 | 79.26 |
| JSON_PRETTY | man | 14311 | 11618 | 2693 | 11977 | 9723 | 2254 | 26288 | 21340 | 4947 | 81.18 | 60.80 | 61.89 | 55.89 | 80.28 | 60.20 | 61.29 | 55.29 |
| JSON_PRETTY | opt | 13395 | 10457 | 2938 | 10411 | 8128 | 2283 | 23806 | 18585 | 5221 | 78.07 | 61.97 | 67.80 | 62.45 | 77.29 | 61.45 | 67.28 | 61.93 |
| TOON_DEFAULT | man | 7182 | 5412 | 1770 | 11615 | 8753 | 2862 | 18797 | 14165 | 4632 | 75.36 | 82.17 | 59.85 | 78.07 | 74.58 | 81.65 | 59.33 | 77.55 |
| TOON_DEFAULT | opt | 11709 | 8761 | 2948 | 8984 | 6722 | 2262 | 20693 | 15483 | 5211 | 74.82 | 65.77 | 72.91 | 71.11 | 73.79 | 65.08 | 72.23 | 70.43 |
| XML_COMPACT | man | 11950 | 8673 | 3277 | 6976 | 5063 | 1913 | 18926 | 13736 | 5190 | 72.58 | 63.43 | 81.67 | 75.77 | 70.95 | 62.34 | 80.58 | 74.68 |
| XML_COMPACT | opt | 11181 | 8717 | 2464 | 11266 | 8783 | 2483 | 22447 | 17499 | 4947 | 77.96 | 69.74 | 63.37 | 67.11 | 77.70 | 69.56 | 63.19 | 66.93 |
| XML_PRETTY | man | 16186 | 12008 | 4178 | 10600 | 7864 | 2736 | 26786 | 19873 | 6914 | 74.19 | 49.50 | 64.25 | 49.49 | 72.37 | 48.28 | 63.03 | 48.28 |
| XML_PRETTY | opt | 15117 | 11500 | 3617 | 9210 | 7006 | 2204 | 24327 | 18505 | 5821 | 76.07 | 54.54 | 72.60 | 59.31 | 75.56 | 54.20 | 72.26 | 58.97 |
| YAML | man | 12574 | 9126 | 3448 | 9373 | 6803 | 2570 | 21947 | 15929 | 6018 | 72.58 | 61.22 | 69.44 | 65.26 | 69.81 | 59.37 | 67.59 | 63.41 |
| YAML | opt | 11791 | 9414 | 2377 | 13029 | 10402 | 2627 | 24820 | 19816 | 5004 | 79.84 | 68.83 | 55.62 | 60.10 | 79.12 | 68.35 | 55.14 | 59.62 |

#### 2.6.2 Read Tokens: Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7063 | 6795 | -268 | -3.79 | 5449 | 4968 | -481 | -8.82 | 1614 | 1827 | +213 | +13.17 | 77.15 | 73.12 | -4.03 | 83.78 | 82.04 | -1.74 | -2.07 | 74.29 | 72.28 | -2.01 | 81.88 | 81.48 | -0.39 | -0.48 |
| JSON_COMPACT | 9296 | 8768 | -528 | -5.68 | 7216 | 6676 | -540 | -7.49 | 2080 | 2092 | +12 | +0.60 | 77.62 | 76.13 | -1.49 | 76.19 | 77.06 | +0.88 | +1.15 | 77.17 | 74.69 | -2.48 | 75.89 | 76.10 | +0.22 | +0.29 |
| JSON_PRETTY | 14311 | 13395 | -916 | -6.40 | 11618 | 10458 | -1160 | -9.99 | 2693 | 2937 | +244 | +9.07 | 81.18 | 78.07 | -3.11 | 60.80 | 61.97 | +1.17 | +1.93 | 80.28 | 77.29 | -2.99 | 60.20 | 61.45 | +1.25 | +2.08 |
| TOON_DEFAULT | 7182 | 11709 | +4527 | +63.03 | 5412 | 8760 | +3348 | +61.87 | 1770 | 2949 | +1179 | +66.59 | 75.36 | 74.82 | -0.54 | 82.17 | 65.77 | -16.39 | -19.95 | 74.58 | 73.79 | -0.79 | 81.65 | 65.08 | -16.56 | -20.28 |
| XML_COMPACT | 11950 | 11181 | -769 | -6.44 | 8673 | 8716 | +43 | +0.50 | 3277 | 2465 | -812 | -24.79 | 72.58 | 77.96 | +5.38 | 63.43 | 69.74 | +6.31 | +9.95 | 70.95 | 77.70 | +6.75 | 62.34 | 69.56 | +7.22 | +11.59 |
| XML_PRETTY | 16186 | 15117 | -1069 | -6.60 | 12008 | 11499 | -509 | -4.24 | 4178 | 3618 | -560 | -13.41 | 74.19 | 76.07 | +1.88 | 49.50 | 54.54 | +5.04 | +10.18 | 72.37 | 75.56 | +3.19 | 48.28 | 54.20 | +5.91 | +12.25 |
| YAML | 12574 | 11791 | -783 | -6.23 | 9126 | 9414 | +288 | +3.15 | 3448 | 2377 | -1071 | -31.05 | 72.58 | 79.84 | +7.26 | 61.22 | 68.83 | +7.61 | +12.44 | 69.81 | 79.12 | +9.31 | 59.37 | 68.35 | +8.98 | +15.13 |

#### 2.6.3 Output Tokens: Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 10162 | 11850 | +1688 | +16.61 | 7840 | 8665 | +825 | +10.52 | 2322 | 3185 | +863 | +37.18 | 77.15 | 73.12 | -4.03 | 68.46 | 57.16 | -11.30 | -16.51 | 74.29 | 72.28 | -2.01 | 66.55 | 56.60 | -9.95 | -14.96 |
| JSON_COMPACT | 13489 | 9558 | -3931 | -29.14 | 10470 | 7277 | -3193 | -30.50 | 3019 | 2282 | -737 | -24.42 | 77.62 | 76.13 | -1.49 | 51.80 | 70.86 | +19.06 | +36.81 | 77.17 | 74.69 | -2.48 | 51.50 | 69.90 | +18.40 | +35.74 |
| JSON_PRETTY | 11977 | 10411 | -1566 | -13.07 | 9723 | 8128 | -1595 | -16.40 | 2254 | 2283 | +29 | +1.29 | 81.18 | 78.07 | -3.11 | 61.89 | 67.80 | +5.92 | +9.56 | 80.28 | 77.29 | -2.99 | 61.29 | 67.28 | +6.00 | +9.78 |
| TOON_DEFAULT | 11615 | 8984 | -2631 | -22.65 | 8753 | 6722 | -2031 | -23.20 | 2862 | 2262 | -600 | -20.95 | 75.36 | 74.82 | -0.54 | 59.85 | 72.91 | +13.06 | +21.83 | 74.58 | 73.79 | -0.79 | 59.33 | 72.23 | +12.90 | +21.74 |
| XML_COMPACT | 6976 | 11266 | +4290 | +61.49 | 5063 | 8783 | +3720 | +73.47 | 1913 | 2483 | +570 | +29.80 | 72.58 | 77.96 | +5.38 | 81.67 | 63.37 | -18.30 | -22.41 | 70.95 | 77.70 | +6.75 | 80.58 | 63.19 | -17.39 | -21.58 |
| XML_PRETTY | 10600 | 9209 | -1391 | -13.12 | 7864 | 7005 | -859 | -10.92 | 2736 | 2204 | -532 | -19.45 | 74.19 | 76.07 | +1.88 | 64.25 | 72.60 | +8.35 | +12.99 | 72.37 | 75.56 | +3.19 | 63.03 | 72.26 | +9.22 | +14.63 |
| YAML | 9373 | 13029 | +3656 | +39.01 | 6803 | 10403 | +3600 | +52.91 | 2570 | 2627 | +57 | +2.20 | 72.58 | 79.84 | +7.26 | 69.44 | 55.62 | -13.82 | -19.90 | 69.81 | 79.12 | +9.31 | 67.59 | 55.14 | -12.45 | -18.42 |

#### 2.6.4 Total Tokens: Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 17225 | 18645 | +1420 | +8.24 | 13289 | 13633 | +344 | +2.59 | 3936 | 5012 | +1076 | +27.33 | 77.15 | 73.12 | -4.03 | 84.73 | 77.10 | -7.63 | -9.00 | 74.29 | 72.28 | -2.01 | 82.82 | 76.54 | -6.28 | -7.58 |
| JSON_COMPACT | 22785 | 18326 | -4459 | -19.57 | 17685 | 13951 | -3734 | -21.11 | 5099 | 4374 | -725 | -14.21 | 77.62 | 76.13 | -1.49 | 65.70 | 80.22 | +14.52 | +22.10 | 77.17 | 74.69 | -2.48 | 65.40 | 79.26 | +13.86 | +21.19 |
| JSON_PRETTY | 26288 | 23806 | -2482 | -9.44 | 21340 | 18585 | -2755 | -12.91 | 4947 | 5220 | +273 | +5.52 | 81.18 | 78.07 | -3.11 | 55.89 | 62.45 | +6.56 | +11.74 | 80.28 | 77.29 | -2.99 | 55.29 | 61.93 | +6.64 | +12.01 |
| TOON_DEFAULT | 18797 | 20693 | +1896 | +10.09 | 14165 | 15482 | +1317 | +9.30 | 4632 | 5211 | +579 | +12.50 | 75.36 | 74.82 | -0.54 | 78.07 | 71.11 | -6.96 | -8.91 | 74.58 | 73.79 | -0.79 | 77.55 | 70.43 | -7.12 | -9.19 |
| XML_COMPACT | 18926 | 22447 | +3521 | +18.60 | 13736 | 17499 | +3763 | +27.39 | 5190 | 4948 | -242 | -4.67 | 72.58 | 77.96 | +5.38 | 75.77 | 67.11 | -8.66 | -11.43 | 70.95 | 77.70 | +6.75 | 74.68 | 66.93 | -7.75 | -10.37 |
| XML_PRETTY | 26786 | 24326 | -2460 | -9.18 | 19873 | 18506 | -1367 | -6.88 | 6914 | 5822 | -1092 | -15.80 | 74.19 | 76.07 | +1.88 | 49.49 | 59.31 | +9.81 | +19.82 | 72.37 | 75.56 | +3.19 | 48.28 | 58.97 | +10.68 | +22.13 |
| YAML | 21947 | 24820 | +2873 | +13.09 | 15929 | 19816 | +3887 | +24.40 | 6018 | 5004 | -1014 | -16.85 | 72.58 | 79.84 | +7.26 | 65.26 | 60.10 | -5.16 | -7.90 | 69.81 | 79.12 | +9.31 | 63.41 | 59.62 | -3.79 | -5.98 |

### 2.7 Token Utilization Efficiency (Accuracy by Character)
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy by Character (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy by Character (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 7063 | 6285 | 778 | 10162 | 9042 | 1120 | 17225 | 15327 | 1898 | 88.98 | 91.67 | 76.34 | 92.62 | 90.49 | 92.67 | 77.35 | 93.62 |
| CSV | opt | 6795 | 6311 | 484 | 11850 | 11006 | 844 | 18645 | 17317 | 1328 | 92.88 | 95.22 | 70.33 | 90.28 | 89.77 | 93.14 | 68.26 | 88.20 |
| JSON_COMPACT | man | 9296 | 9064 | 232 | 13489 | 13151 | 337 | 22785 | 22215 | 570 | 97.50 | 89.44 | 65.05 | 78.96 | 93.53 | 86.79 | 62.40 | 76.31 |
| JSON_COMPACT | opt | 8768 | 8438 | 330 | 9558 | 9198 | 359 | 18326 | 17637 | 689 | 96.24 | 90.47 | 84.27 | 93.63 | 92.52 | 87.99 | 81.79 | 91.15 |
| JSON_PRETTY | man | 14311 | 13880 | 431 | 11977 | 11616 | 360 | 26288 | 25496 | 791 | 96.99 | 71.34 | 72.43 | 66.43 | 93.51 | 69.02 | 70.11 | 64.11 |
| JSON_PRETTY | opt | 13395 | 13028 | 367 | 10411 | 10126 | 285 | 23806 | 23154 | 652 | 97.26 | 74.76 | 80.59 | 75.24 | 93.72 | 72.40 | 78.23 | 72.88 |
| TOON_DEFAULT | man | 7182 | 6658 | 524 | 11615 | 10767 | 848 | 18797 | 17425 | 1372 | 92.70 | 93.73 | 71.41 | 89.63 | 90.97 | 92.57 | 70.26 | 88.48 |
| TOON_DEFAULT | opt | 11709 | 11152 | 557 | 8984 | 8557 | 428 | 20693 | 19708 | 985 | 95.24 | 79.39 | 86.53 | 84.73 | 91.67 | 77.01 | 84.15 | 82.35 |
| XML_COMPACT | man | 11950 | 11292 | 658 | 6976 | 6592 | 384 | 18926 | 17883 | 1043 | 94.49 | 78.03 | 96.28 | 90.37 | 91.52 | 76.05 | 94.30 | 88.39 |
| XML_COMPACT | opt | 11181 | 10942 | 239 | 11266 | 11025 | 241 | 22447 | 21966 | 480 | 97.86 | 83.00 | 76.63 | 80.37 | 92.22 | 79.24 | 72.87 | 76.61 |
| XML_PRETTY | man | 16186 | 15466 | 720 | 10600 | 10129 | 472 | 26786 | 25594 | 1192 | 95.55 | 63.74 | 78.49 | 63.73 | 93.14 | 62.13 | 76.88 | 62.13 |
| XML_PRETTY | opt | 15117 | 14461 | 656 | 9210 | 8810 | 400 | 24327 | 23271 | 1056 | 95.66 | 67.59 | 85.66 | 72.36 | 91.61 | 64.89 | 82.96 | 69.67 |
| YAML | man | 12574 | 12097 | 477 | 9373 | 9017 | 355 | 21947 | 21115 | 832 | 96.21 | 76.97 | 85.19 | 81.01 | 93.17 | 74.94 | 83.17 | 78.98 |
| YAML | opt | 11791 | 11604 | 187 | 13029 | 12822 | 207 | 24820 | 24425 | 395 | 98.41 | 81.21 | 68.00 | 72.48 | 94.30 | 78.47 | 65.26 | 69.74 |

#### 2.7.2 Read Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 7063 | 6795 | -268 | -3.79 | 6285 | 6312 | +27 | +0.42 | 778 | 483 | -295 | -37.86 | 88.98 | 92.88 | +3.90 | 91.67 | 95.22 | +3.55 | +3.87 | 90.49 | 89.77 | -0.72 | 92.67 | 93.14 | +0.47 | +0.51 |
| JSON_COMPACT | 9296 | 8768 | -528 | -5.68 | 9064 | 8439 | -625 | -6.90 | 232 | 329 | +97 | +41.93 | 97.50 | 96.24 | -1.26 | 89.44 | 90.47 | +1.03 | +1.15 | 93.53 | 92.52 | -1.01 | 86.79 | 87.99 | +1.20 | +1.38 |
| JSON_PRETTY | 14311 | 13395 | -916 | -6.40 | 13880 | 13028 | -852 | -6.14 | 431 | 367 | -64 | -14.79 | 96.99 | 97.26 | +0.27 | 71.34 | 74.76 | +3.42 | +4.80 | 93.51 | 93.72 | +0.21 | 69.02 | 72.40 | +3.38 | +4.90 |
| TOON_DEFAULT | 7182 | 11709 | +4527 | +63.03 | 6658 | 11152 | +4494 | +67.50 | 524 | 557 | +33 | +6.31 | 92.70 | 95.24 | +2.54 | 93.73 | 79.39 | -14.34 | -15.30 | 90.97 | 91.67 | +0.70 | 92.57 | 77.01 | -15.57 | -16.82 |
| XML_COMPACT | 11950 | 11181 | -769 | -6.44 | 11292 | 10942 | -350 | -3.10 | 658 | 239 | -419 | -63.70 | 94.49 | 97.86 | +3.37 | 78.03 | 83.00 | +4.97 | +6.37 | 91.52 | 92.22 | +0.70 | 76.05 | 79.24 | +3.19 | +4.19 |
| XML_PRETTY | 16186 | 15117 | -1069 | -6.60 | 15466 | 14461 | -1005 | -6.50 | 720 | 656 | -64 | -8.92 | 95.55 | 95.66 | +0.11 | 63.74 | 67.59 | +3.86 | +6.05 | 93.14 | 91.61 | -1.53 | 62.13 | 64.89 | +2.77 | +4.45 |
| YAML | 12574 | 11791 | -783 | -6.23 | 12097 | 11603 | -494 | -4.08 | 477 | 188 | -289 | -60.60 | 96.21 | 98.41 | +2.20 | 76.97 | 81.21 | +4.24 | +5.51 | 93.17 | 94.30 | +1.13 | 74.94 | 78.47 | +3.53 | +4.70 |

#### 2.7.3 Output Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 10162 | 11850 | +1688 | +16.61 | 9042 | 11006 | +1964 | +21.72 | 1120 | 844 | -276 | -24.65 | 88.98 | 92.88 | +3.90 | 76.34 | 70.33 | -6.01 | -7.88 | 90.49 | 89.77 | -0.72 | 77.35 | 68.26 | -9.09 | -11.76 |
| JSON_COMPACT | 13489 | 9558 | -3931 | -29.14 | 13151 | 9198 | -3953 | -30.06 | 337 | 359 | +22 | +6.58 | 97.50 | 96.24 | -1.26 | 65.05 | 84.27 | +19.22 | +29.54 | 93.53 | 92.52 | -1.01 | 62.40 | 81.79 | +19.38 | +31.06 |
| JSON_PRETTY | 11977 | 10411 | -1566 | -13.07 | 11616 | 10126 | -1490 | -12.83 | 360 | 285 | -75 | -20.90 | 96.99 | 97.26 | +0.27 | 72.43 | 80.59 | +8.17 | +11.28 | 93.51 | 93.72 | +0.21 | 70.11 | 78.23 | +8.13 | +11.60 |
| TOON_DEFAULT | 11615 | 8984 | -2631 | -22.65 | 10767 | 8556 | -2211 | -20.53 | 848 | 428 | -420 | -49.56 | 92.70 | 95.24 | +2.54 | 71.41 | 86.53 | +15.12 | +21.17 | 90.97 | 91.67 | +0.70 | 70.26 | 84.15 | +13.89 | +19.77 |
| XML_COMPACT | 6976 | 11266 | +4290 | +61.49 | 6592 | 11025 | +4433 | +67.25 | 384 | 241 | -143 | -37.32 | 94.49 | 97.86 | +3.37 | 96.28 | 76.63 | -19.64 | -20.40 | 91.52 | 92.22 | +0.70 | 94.30 | 72.87 | -21.42 | -22.72 |
| XML_PRETTY | 10600 | 9209 | -1391 | -13.12 | 10129 | 8810 | -1319 | -13.02 | 472 | 400 | -72 | -15.26 | 95.55 | 95.66 | +0.11 | 78.49 | 85.66 | +7.17 | +9.13 | 93.14 | 91.61 | -1.53 | 76.88 | 82.96 | +6.08 | +7.90 |
| YAML | 9373 | 13029 | +3656 | +39.01 | 9017 | 12821 | +3804 | +42.19 | 355 | 207 | -148 | -41.71 | 96.21 | 98.41 | +2.20 | 85.19 | 68.00 | -17.19 | -20.18 | 93.17 | 94.30 | +1.13 | 83.17 | 65.26 | -17.90 | -21.53 |

#### 2.7.4 Total Tokens (Accuracy by Character): Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy by Character (%) Man | Wtd Accuracy by Character (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 17225 | 18645 | +1420 | +8.24 | 15327 | 17318 | +1991 | +12.99 | 1898 | 1327 | -571 | -30.07 | 88.98 | 92.88 | +3.90 | 92.62 | 90.28 | -2.34 | -2.53 | 90.49 | 89.77 | -0.72 | 93.62 | 88.20 | -5.42 | -5.79 |
| JSON_COMPACT | 22785 | 18326 | -4459 | -19.57 | 22215 | 17637 | -4578 | -20.61 | 570 | 689 | +119 | +20.95 | 97.50 | 96.24 | -1.26 | 78.96 | 93.63 | +14.67 | +18.58 | 93.53 | 92.52 | -1.01 | 76.31 | 91.15 | +14.84 | +19.44 |
| JSON_PRETTY | 26288 | 23806 | -2482 | -9.44 | 25496 | 23153 | -2343 | -9.19 | 791 | 652 | -139 | -17.57 | 96.99 | 97.26 | +0.27 | 66.43 | 75.24 | +8.81 | +13.27 | 93.51 | 93.72 | +0.21 | 64.11 | 72.88 | +8.77 | +13.69 |
| TOON_DEFAULT | 18797 | 20693 | +1896 | +10.09 | 17425 | 19708 | +2283 | +13.10 | 1372 | 985 | -387 | -28.22 | 92.70 | 95.24 | +2.54 | 89.63 | 84.73 | -4.90 | -5.47 | 90.97 | 91.67 | +0.70 | 88.48 | 82.35 | -6.13 | -6.93 |
| XML_COMPACT | 18926 | 22447 | +3521 | +18.60 | 17883 | 21966 | +4083 | +22.83 | 1043 | 481 | -562 | -53.93 | 94.49 | 97.86 | +3.37 | 90.37 | 80.37 | -10.00 | -11.07 | 91.52 | 92.22 | +0.70 | 88.39 | 76.61 | -11.78 | -13.33 |
| XML_PRETTY | 26786 | 24326 | -2460 | -9.18 | 25594 | 23271 | -2323 | -9.08 | 1192 | 1056 | -136 | -11.43 | 95.55 | 95.66 | +0.11 | 63.73 | 72.36 | +8.63 | +13.54 | 93.14 | 91.61 | -1.53 | 62.13 | 69.67 | +7.54 | +12.13 |
| YAML | 21947 | 24820 | +2873 | +13.09 | 21115 | 24425 | +3310 | +15.68 | 832 | 395 | -437 | -52.54 | 96.21 | 98.41 | +2.20 | 81.01 | 72.48 | -8.53 | -10.53 | 93.17 | 94.30 | +1.13 | 78.98 | 69.74 | -9.24 | -11.70 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) | Expected Characters | Output Characters | Correct Characters | Incorrect Characters | Accuracy by Character (%) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 96 | 28 | 0 | 77.15 | 7782 | 8213 | 7271 | 942 | 88.98 |
| CSV | opt | 91 | 33 | 0 | 73.12 | 8451 | 8724 | 8097 | 627 | 92.88 |
| JSON_COMPACT | man | 96 | 28 | 0 | 77.62 | 7782 | 7829 | 7633 | 196 | 97.50 |
| JSON_COMPACT | opt | 94 | 30 | 0 | 76.13 | 8451 | 8577 | 8253 | 324 | 96.24 |
| JSON_PRETTY | man | 101 | 23 | 0 | 81.18 | 7782 | 7843 | 7606 | 237 | 96.99 |
| JSON_PRETTY | opt | 97 | 27 | 0 | 78.07 | 8451 | 8567 | 8330 | 238 | 97.26 |
| TOON_DEFAULT | man | 93 | 31 | 0 | 75.36 | 7782 | 8149 | 7528 | 621 | 92.70 |
| TOON_DEFAULT | opt | 93 | 31 | 0 | 74.82 | 8451 | 8646 | 8232 | 414 | 95.24 |
| XML_COMPACT | man | 90 | 34 | 0 | 72.58 | 7782 | 7982 | 7539 | 442 | 94.49 |
| XML_COMPACT | opt | 97 | 27 | 0 | 77.96 | 8451 | 8524 | 8341 | 183 | 97.86 |
| XML_PRETTY | man | 92 | 32 | 0 | 74.19 | 7782 | 7906 | 7554 | 352 | 95.55 |
| XML_PRETTY | opt | 94 | 30 | 0 | 76.07 | 8451 | 8629 | 8251 | 378 | 95.66 |
| YAML | man | 90 | 34 | 0 | 72.58 | 7782 | 7921 | 7619 | 302 | 96.21 |
| YAML | opt | 99 | 25 | 0 | 79.84 | 8451 | 8482 | 8347 | 135 | 98.41 |

#### 2.8.2 Answers: Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 96 | 91 | -5 | -5.21 | 28 | 33 | +5 | +17.86 | 0 | 0 | 0 | 0.00 | 77.15 | 73.12 | -4.03 |
| JSON_COMPACT | 96 | 94 | -2 | -1.93 | 28 | 30 | +2 | +6.61 | 0 | 0 | 0 | 0.00 | 77.62 | 76.13 | -1.49 |
| JSON_PRETTY | 101 | 97 | -4 | -3.83 | 23 | 27 | +4 | +16.83 | 0 | 0 | 0 | 0.00 | 81.18 | 78.07 | -3.11 |
| TOON_DEFAULT | 93 | 92 | -1 | -0.71 | 31 | 32 | +1 | +2.13 | 0 | 0 | 0 | 0.00 | 75.36 | 74.82 | -0.54 |
| XML_COMPACT | 90 | 97 | +7 | +7.41 | 34 | 27 | -7 | -19.62 | 0 | 0 | 0 | 0.00 | 72.58 | 77.96 | +5.38 |
| XML_PRETTY | 92 | 94 | +2 | +2.53 | 32 | 30 | -2 | -7.28 | 0 | 0 | 0 | 0.00 | 74.19 | 76.07 | +1.88 |
| YAML | 90 | 99 | +9 | +10.00 | 34 | 25 | -9 | -26.47 | 0 | 0 | 0 | 0.00 | 72.58 | 79.84 | +7.26 |

#### 2.8.3 Characters: Mandatory vs Optional Data
| Format | Output Characters Man | Output Characters Opt | Diff | Diff (%) | Correct Characters Man | Correct Characters Opt | Diff | Diff (%) | Incorrect Characters Man | Incorrect Characters Opt | Diff | Diff (%) | Accuracy by Character (%) Man | Accuracy by Character (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 8213 | 8724 | +511 | +6.22 | 7271 | 8098 | +827 | +11.37 | 942 | 626 | -316 | -33.51 | 88.98 | 92.88 | 3.90 |
| JSON_COMPACT | 7829 | 8577 | +748 | +9.56 | 7633 | 8253 | +620 | +8.12 | 196 | 325 | +129 | +65.66 | 97.50 | 96.24 | -1.26 |
| JSON_PRETTY | 7843 | 8568 | +725 | +9.24 | 7606 | 8330 | +724 | +9.52 | 237 | 238 | +1 | +0.39 | 96.99 | 97.26 | 0.27 |
| TOON_DEFAULT | 8149 | 8647 | +498 | +6.11 | 7528 | 8232 | +704 | +9.35 | 621 | 414 | -207 | -33.26 | 92.70 | 95.24 | 2.54 |
| XML_COMPACT | 7982 | 8524 | +542 | +6.79 | 7539 | 8341 | +802 | +10.64 | 442 | 182 | -260 | -58.75 | 94.49 | 97.86 | 3.37 |
| XML_PRETTY | 7906 | 8629 | +723 | +9.15 | 7554 | 8251 | +697 | +9.23 | 352 | 378 | +26 | +7.39 | 95.55 | 95.66 | 0.11 |
| YAML | 7921 | 8483 | +562 | +7.09 | 7619 | 8347 | +728 | +9.56 | 302 | 136 | -166 | -55.08 | 96.21 | 98.41 | 2.20 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 77.15 | 95.76 | 59.26 | 60.32 | 68.26 | 90.49 | 35.91 | 17.28 | 12.57 | 8.53 |
| CSV | opt | 73.12 | 89.70 | 67.90 | 61.90 | 47.62 | 89.77 | 33.63 | 19.80 | 12.89 | 5.95 |
| JSON_COMPACT | man | 77.62 | 98.18 | 75.93 | 63.10 | 40.48 | 93.53 | 36.82 | 22.15 | 13.15 | 5.06 |
| JSON_COMPACT | opt | 76.13 | 94.91 | 67.41 | 61.90 | 52.38 | 92.52 | 35.59 | 19.66 | 12.90 | 6.55 |
| JSON_PRETTY | man | 81.18 | 96.97 | 77.78 | 66.67 | 58.73 | 93.51 | 36.36 | 22.69 | 13.89 | 7.34 |
| JSON_PRETTY | opt | 78.07 | 96.73 | 72.59 | 66.67 | 47.62 | 93.72 | 36.27 | 21.18 | 13.89 | 5.95 |
| TOON_DEFAULT | man | 75.36 | 92.32 | 72.84 | 60.32 | 49.21 | 90.97 | 34.62 | 21.24 | 12.57 | 6.15 |
| TOON_DEFAULT | opt | 74.82 | 93.33 | 69.14 | 60.84 | 47.62 | 91.67 | 35.00 | 20.16 | 12.68 | 5.95 |
| XML_COMPACT | man | 72.58 | 88.49 | 64.20 | 57.14 | 57.14 | 91.52 | 33.18 | 18.72 | 11.91 | 7.14 |
| XML_COMPACT | opt | 77.96 | 97.58 | 79.01 | 61.90 | 41.27 | 92.22 | 36.59 | 23.05 | 12.90 | 5.16 |
| XML_PRETTY | man | 74.19 | 95.76 | 59.26 | 63.49 | 47.62 | 93.14 | 35.91 | 17.28 | 13.23 | 5.95 |
| XML_PRETTY | opt | 76.07 | 90.91 | 77.78 | 58.73 | 52.38 | 91.61 | 34.09 | 22.69 | 12.23 | 6.55 |
| YAML | man | 72.58 | 96.97 | 49.38 | 61.90 | 49.21 | 93.17 | 36.36 | 14.40 | 12.89 | 6.15 |
| YAML | opt | 79.84 | 99.39 | 71.60 | 73.02 | 46.03 | 94.30 | 37.27 | 20.88 | 15.21 | 5.75 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 95.76 | 89.70 | -6.06 | 35.91 | 33.63 | -2.28 |
| JSON_COMPACT | 98.18 | 94.91 | -3.27 | 36.82 | 35.59 | -1.23 |
| JSON_PRETTY | 96.97 | 96.73 | -0.24 | 36.36 | 36.27 | -0.09 |
| TOON_DEFAULT | 92.32 | 93.33 | +1.01 | 34.62 | 35.00 | +0.38 |
| XML_COMPACT | 88.49 | 97.58 | +9.09 | 33.18 | 36.59 | +3.41 |
| XML_PRETTY | 95.76 | 90.91 | -4.85 | 35.91 | 34.09 | -1.82 |
| YAML | 96.97 | 99.39 | +2.42 | 36.36 | 37.27 | +0.91 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 59.26 | 67.90 | +8.64 | 17.28 | 19.80 | +2.52 |
| JSON_COMPACT | 75.93 | 67.41 | -8.52 | 22.15 | 19.66 | -2.48 |
| JSON_PRETTY | 77.78 | 72.59 | -5.19 | 22.69 | 21.18 | -1.51 |
| TOON_DEFAULT | 72.84 | 69.14 | -3.71 | 21.24 | 20.16 | -1.08 |
| XML_COMPACT | 64.20 | 79.01 | +14.82 | 18.72 | 23.05 | +4.33 |
| XML_PRETTY | 59.26 | 77.78 | +18.52 | 17.28 | 22.69 | +5.41 |
| YAML | 49.38 | 71.60 | +22.22 | 14.40 | 20.88 | +6.48 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 60.32 | 61.90 | +1.59 | 12.57 | 12.89 | +0.33 |
| JSON_COMPACT | 63.10 | 61.90 | -1.19 | 13.15 | 12.90 | -0.25 |
| JSON_PRETTY | 66.67 | 66.67 | 0.00 | 13.89 | 13.89 | 0.00 |
| TOON_DEFAULT | 60.32 | 60.84 | +0.53 | 12.57 | 12.68 | +0.11 |
| XML_COMPACT | 57.14 | 61.90 | +4.76 | 11.91 | 12.90 | +0.99 |
| XML_PRETTY | 63.49 | 58.73 | -4.77 | 13.23 | 12.23 | -0.99 |
| YAML | 61.90 | 73.02 | +11.11 | 12.89 | 15.21 | +2.32 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 68.26 | 47.62 | -20.64 | 8.53 | 5.95 | -2.58 |
| JSON_COMPACT | 40.48 | 52.38 | +11.90 | 5.06 | 6.55 | +1.49 |
| JSON_PRETTY | 58.73 | 47.62 | -11.11 | 7.34 | 5.95 | -1.39 |
| TOON_DEFAULT | 49.21 | 47.62 | -1.59 | 6.15 | 5.95 | -0.20 |
| XML_COMPACT | 57.14 | 41.27 | -15.87 | 7.14 | 5.16 | -1.98 |
| XML_PRETTY | 47.62 | 52.38 | +4.76 | 5.95 | 6.55 | +0.60 |
| YAML | 49.21 | 46.03 | -3.17 | 6.15 | 5.75 | -0.40 |

### 2.10 Accuracy By Character Per Question Category Analysis
#### 2.10.1 Metrics
| Format | Variant | Accuracy By Character (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) | Wtd Acc By Char (%) | Wtd Field Retrieval (%) | Wtd Structure Awareness (%) | Wtd Filtering (%) | Wtd Aggregation (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 88.98 | 95.24 | 85.80 | 91.67 | 85.18 | 90.49 | 35.72 | 25.03 | 19.10 | 10.65 |
| CSV | opt | 92.88 | 88.83 | 97.35 | 90.86 | 73.13 | 89.77 | 33.31 | 28.39 | 18.93 | 9.14 |
| JSON_COMPACT | man | 97.50 | 97.89 | 98.28 | 91.07 | 73.46 | 93.53 | 36.71 | 28.67 | 18.97 | 9.18 |
| JSON_COMPACT | opt | 96.24 | 96.58 | 96.90 | 89.20 | 75.64 | 92.52 | 36.22 | 28.26 | 18.58 | 9.46 |
| JSON_PRETTY | man | 96.99 | 95.73 | 98.69 | 91.07 | 78.81 | 93.51 | 35.90 | 28.78 | 18.97 | 9.85 |
| JSON_PRETTY | opt | 97.26 | 97.94 | 97.66 | 91.68 | 75.27 | 93.72 | 36.73 | 28.48 | 19.10 | 9.41 |
| TOON_DEFAULT | man | 92.70 | 93.30 | 93.07 | 92.06 | 77.25 | 90.97 | 34.99 | 27.15 | 19.18 | 9.66 |
| TOON_DEFAULT | opt | 95.24 | 94.65 | 96.72 | 89.28 | 74.86 | 91.67 | 35.49 | 28.21 | 18.60 | 9.36 |
| XML_COMPACT | man | 94.49 | 91.71 | 97.12 | 88.39 | 83.13 | 91.52 | 34.39 | 28.33 | 18.41 | 10.39 |
| XML_COMPACT | opt | 97.86 | 99.04 | 98.27 | 85.84 | 68.28 | 92.22 | 37.14 | 28.66 | 17.88 | 8.54 |
| XML_PRETTY | man | 95.55 | 96.76 | 95.45 | 92.86 | 77.37 | 93.14 | 36.28 | 27.84 | 19.35 | 9.67 |
| XML_PRETTY | opt | 95.66 | 93.56 | 98.45 | 87.91 | 75.96 | 91.61 | 35.08 | 28.71 | 18.31 | 9.50 |
| YAML | man | 96.21 | 98.37 | 95.57 | 90.22 | 76.83 | 93.17 | 36.89 | 27.88 | 18.80 | 9.60 |
| YAML | opt | 98.41 | 99.85 | 98.34 | 92.33 | 71.52 | 94.30 | 37.44 | 28.68 | 19.23 | 8.94 |

#### 2.10.2 Field Retrieval: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 95.24 | 88.83 | -6.41 | 35.72 | 33.31 | -2.40 |
| JSON_COMPACT | 97.89 | 96.58 | -1.30 | 36.71 | 36.22 | -0.49 |
| JSON_PRETTY | 95.73 | 97.94 | +2.21 | 35.90 | 36.73 | +0.83 |
| TOON_DEFAULT | 93.30 | 94.65 | +1.35 | 34.99 | 35.49 | +0.51 |
| XML_COMPACT | 91.71 | 99.04 | +7.33 | 34.39 | 37.14 | +2.75 |
| XML_PRETTY | 96.76 | 93.56 | -3.21 | 36.28 | 35.08 | -1.20 |
| YAML | 98.37 | 99.85 | +1.48 | 36.89 | 37.44 | +0.55 |

#### 2.10.3 Structure Awareness: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 85.80 | 97.35 | +11.55 | 25.03 | 28.39 | +3.37 |
| JSON_COMPACT | 98.28 | 96.90 | -1.38 | 28.67 | 28.26 | -0.40 |
| JSON_PRETTY | 98.69 | 97.66 | -1.03 | 28.78 | 28.48 | -0.30 |
| TOON_DEFAULT | 93.07 | 96.72 | +3.65 | 27.15 | 28.21 | +1.06 |
| XML_COMPACT | 97.12 | 98.27 | +1.15 | 28.33 | 28.66 | +0.33 |
| XML_PRETTY | 95.45 | 98.45 | +3.00 | 27.84 | 28.71 | +0.87 |
| YAML | 95.57 | 98.34 | +2.77 | 27.88 | 28.68 | +0.81 |

#### 2.10.4 Filtering: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 91.67 | 90.86 | -0.81 | 19.10 | 18.93 | -0.17 |
| JSON_COMPACT | 91.07 | 89.20 | -1.87 | 18.97 | 18.58 | -0.39 |
| JSON_PRETTY | 91.07 | 91.68 | +0.61 | 18.97 | 19.10 | +0.13 |
| TOON_DEFAULT | 92.06 | 89.28 | -2.78 | 19.18 | 18.60 | -0.58 |
| XML_COMPACT | 88.39 | 85.84 | -2.55 | 18.41 | 17.88 | -0.53 |
| XML_PRETTY | 92.86 | 87.91 | -4.95 | 19.35 | 18.31 | -1.03 |
| YAML | 90.22 | 92.33 | +2.11 | 18.80 | 19.23 | +0.44 |

#### 2.10.5 Aggregation: Mandatory vs Optional

| Format | Man (%) | Opt (%) | Diff (%) | Wdt Man (%) | Wdt Opt (%) | Diff (%) |
|---|---|---|---|---|---|---|
| CSV | 85.18 | 73.13 | -12.05 | 10.65 | 9.14 | -1.51 |
| JSON_COMPACT | 73.46 | 75.64 | +2.18 | 9.18 | 9.46 | +0.27 |
| JSON_PRETTY | 78.81 | 75.27 | -3.53 | 9.85 | 9.41 | -0.44 |
| TOON_DEFAULT | 77.25 | 74.86 | -2.39 | 9.66 | 9.36 | -0.29 |
| XML_COMPACT | 83.13 | 68.28 | -14.84 | 10.39 | 8.54 | -1.85 |
| XML_PRETTY | 77.37 | 75.96 | -1.41 | 9.67 | 9.50 | -0.17 |
| YAML | 76.83 | 71.52 | -5.31 | 9.60 | 8.94 | -0.66 |

## 3. Appendices

### 3.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
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

- **Report Generated**: 2026-04-14
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code 2.1.73
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_haiku_4_5/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_haiku_4_5/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_haiku_4_5/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)