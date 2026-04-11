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

1. **CSV** is the cheapest format but the least accurate. Despite consuming the fewest read tokens across both variants **CSV** consistently ranks last in accuracy (69.08% mandatory, 62.90% optional). Its output tokens more than double for optional data (+101.43%) while accuracy simultaneously drops which delivers the worst cost-to-value ratio when sparse data is involved.

2. **TOON_DEFAULT** is the most robust format across data variants with only a 0.54% accuracy difference between mandatory and optional. However its read tokens increase by 64.43% for optional data because the **TOON_DEFAULT** encoding switches from a collapsed tabular style (header row with comma-separated values) to expanded key-value pairs when the structure is sparsely filled and field presence varies between records.

3. **YAML** achieves the highest mandatory accuracy (81.18%) but suffers the steepest accuracy drop for optional data (-7.25%). This makes it an unreliable choice when data completeness cannot be guaranteed despite its strong performance on dense and uniform structures.

4. Output tokens correlate with accuracy which reveals that the model invests more reasoning effort into structured formats. **CSV** mandatory uses only 4,557 output tokens which is roughly half of any other format and this correlates with its lowest accuracy. The model appears to underinvest in reasoning when the format looks deceptively simple while explicitly structured formats like **JSON**, **XML** and **TOON_DEFAULT** elicit 9,800 to 13,400 output tokens of deeper analytical engagement that translates into higher accuracy.

5. **JSON_COMPACT** delivers the best overall efficiency for optional data (efficiency score 72.47) by combining moderate token cost with competitive accuracy (77.42%) while **JSON_PRETTY** leads mandatory accuracy after **YAML** at 80.65% but at the highest total token cost (27,582 tokens).

6. Aggregation is universally the hardest question category and degrades most severely for optional data. Accuracy drops an average of roughly 15 percentage points from mandatory to optional across all formats with some formats losing over 20 points (**CSV** -25.40%, **XML_PRETTY** -23.81%). This indicates that sparse data structures make mathematical reasoning significantly more difficult regardless of format.

7. Most formats consume fewer output tokens for optional data (15-20% reduction) because fewer fields produce shorter answers. The two notable exceptions are **CSV** (+101.43%) and **XML_PRETTY** (+17.14%) which both expand their output when handling sparse data. **CSV**'s output explosion is particularly striking because it doubles its token expenditure without any accuracy benefit which suggests the model compensates for structural ambiguity with verbosity rather than precision.

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
   - Optional: CSV 15867 tokens
   - Mandatory: CSV 11525 tokens
- Lowest read token cost:
   - Optional: CSV 6687 tokens
   - Mandatory: CSV 6968 tokens
- Lowest output token cost:
   - Optional: JSON_COMPACT 9070 tokens
   - Mandatory: CSV 4557 tokens
- Lowest output token cost drift:
   - Optional: YAML ↓ -0.93% ↑ 1.21%
   - Mandatory: YAML ↓ -12.04% ↑ 17.22%
- Highest accuracy:
   - Optional: TOON_DEFAULT 79.84%
   - Mandatory: YAML 81.18%
- Lowest accuracy drift:
   - Optional: TOON_DEFAULT ↓ -2.02% ↑ 4.03%
   - Mandatory: JSON_PRETTY ↓ -1.00% ↑ 0.99%
- Most useful read tokens:
   - Optional: XML_PRETTY 12037 / 15076 tokens
   - Mandatory: XML_PRETTY 12753 / 16137 tokens
- Most useful output tokens:
   - Optional: TOON_DEFAULT 9438 / 11821 tokens
   - Mandatory: JSON_PRETTY 10749 / 13328 tokens
- Highest read efficiency (%/token):
   - Optional: JSON_COMPACT 77.70
   - Mandatory: TOON_DEFAULT 84.43
- Highest output efficiency (%/token):
   - Optional: JSON_COMPACT 77.70
   - Mandatory: CSV 77.43
- Highest accuracy by char:
   - Optional: XML_PRETTY 95.06%
   - Mandatory: YAML 83.97%
- Lowest accuracy by char drift:
   - Optional: XML_PRETTY ↓ -4.31% ↑ 2.90%
   - Mandatory: YAML ↓ -4.12% ↑ 2.54%
- Most useful output write tokens (Acc By Char):
   - Optional: XML_PRETTY 10713 / 11269 tokens
   - Mandatory: XML_COMPACT 11119 / 13245 tokens
- Highest output write efficiency (Acc By Char) (%/token):
   - Optional: JSON_PRETTY 75.70
   - Mandatory: CSV 77.88
- Lowest delta (optional-mandatory):
   - Read tokens: CSV -281 tokens
   - Output tokens: TOON_DEFAULT -464 tokens
   - Accuracy: TOON_DEFAULT 0.54%
   - Read efficiency: JSON_PRETTY 1.18
   - Output efficiency: YAML 1.18
   - Accuracy by char: XML_COMPACT 1.51%
   - Output write efficiency (Acc By Char): XML_PRETTY 3.94

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: XML_PRETTY 26555 tokens
   - Mandatory: JSON_PRETTY 27582 tokens
- Highest read token cost:
   - Optional: XML_PRETTY 15076 tokens
   - Mandatory: XML_PRETTY 16137 tokens
- Highest output token cost:
   - Optional: TOON_DEFAULT 11821 tokens
   - Mandatory: XML_COMPACT 13453 tokens
- Highest output token drift:
   - Optional: JSON_PRETTY ↓ -50.40% ↑ 63.06%
   - Mandatory: CSV ↓ -99.69% ↑ 193.15%
- Lowest accuracy:
   - Optional: CSV 62.90%
   - Mandatory: CSV 69.08%
- Highest accuracy drift:
   - Optional: JSON_COMPACT ↓ -9.38% ↑ 7.28%
   - Mandatory: TOON_DEFAULT ↓ -16.61% ↑ 10.84%
- Most wasted read tokens:
   - Optional: YAML 3061 / 11742 tokens
   - Mandatory: XML_PRETTY 3384 / 16137 tokens
- Most wasted output tokens:
   - Optional: CSV 3406 / 9180 tokens
   - Mandatory: XML_COMPACT 2892 / 13453 tokens
- Lowest read efficiency (%/token):
   - Optional: XML_PRETTY 59.28
   - Mandatory: XML_PRETTY 55.35
- Lowest output efficiency (%/token):
   - Optional: CSV 59.28
   - Mandatory: XML_COMPACT 56.60
- Lowest accuracy by char:
   - Optional: CSV 57.27%
   - Mandatory: CSV 68.44%
- Highest accuracy by char drift:
   - Optional: JSON_COMPACT ↓ -24.45% ↑ 12.93%
   - Mandatory: TOON_DEFAULT ↓ -20.03% ↑ 25.69%
- Most wasted output write tokens (Acc By Char):
   - Optional: CSV 5085 / 8879 tokens
   - Mandatory: TOON_DEFAULT 9176 / 11976 tokens
- Lowest output write efficiency (Acc By Char) (%/token):
   - Optional: CSV 55.49
   - Mandatory: TOON_DEFAULT 59.06
- Highest delta (optional-mandatory):
   - Read tokens: TOON_DEFAULT 4522 tokens
   - Output tokens: CSV 4622 tokens
   - Accuracy: YAML -7.25%
   - Read efficiency: TOON_DEFAULT -13.95
   - Output efficiency: CSV -19.04
   - Accuracy by char: XML_PRETTY 13.79%
   - Output write efficiency (Acc By Char): CSV -22.39

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy By Char | ↓ Eff Score Output Write (Acc By Char) | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|
| XML_PRETTY ≈ 85s | CSV ≈ 6968 | CSV ≈ 204 | CSV ≈ 4353 | CSV ≈ 4557 | CSV ≈ 11525 | YAML ≈ 84% | CSV ≈ 78 | YAML ≈ 81% | TOON_DEFAULT ≈ 84 | CSV ≈ 78 | CSV ≈ 78 |
| YAML (+8.5%) | TOON_DEFAULT (+0.7%) | XML_COMPACT (+2.3%) | XML_PRETTY (+118.1%) | XML_PRETTY (+115.0%) | TOON_DEFAULT (+67.5%) | XML_COMPACT (-0.0%) | XML_PRETTY (-9.7%) | JSON_PRETTY (-0.5%) | CSV (-8.3%) | XML_PRETTY (-12.4%) | TOON_DEFAULT (-9.4%) |
| JSON_COMPACT (+10.9%) | JSON_COMPACT (+32.8%) | JSON_COMPACT (+4.2%) | YAML (+147.5%) | YAML (+143.0%) | JSON_COMPACT (+76.7%) | JSON_PRETTY (-0.5%) | YAML (-12.6%) | TOON_DEFAULT (-1.9%) | JSON_COMPACT (-11.3%) | YAML (-15.7%) | JSON_COMPACT (-15.0%) |
| TOON_DEFAULT (+18.1%) | XML_COMPACT (+67.5%) | XML_PRETTY (+48.9%) | JSON_COMPACT (+150.4%) | JSON_COMPACT (+143.8%) | YAML (+104.8%) | XML_PRETTY (-2.7%) | JSON_COMPACT (-19.9%) | XML_PRETTY (-2.2%) | XML_COMPACT (-18.1%) | JSON_COMPACT (-20.6%) | YAML (-18.0%) |
| JSON_PRETTY (+27.2%) | YAML (+79.9%) | YAML (+48.9%) | TOON_DEFAULT (+175.1%) | TOON_DEFAULT (+169.6%) | XML_COMPACT (+118.0%) | TOON_DEFAULT (-7.3%) | JSON_PRETTY (-22.3%) | XML_COMPACT (-2.7%) | YAML (-19.1%) | TOON_DEFAULT (-22.3%) | XML_COMPACT (-24.0%) |
| XML_COMPACT (+30.7%) | JSON_PRETTY (+104.6%) | JSON_PRETTY (+51.0%) | JSON_PRETTY (+199.1%) | JSON_PRETTY (+192.5%) | XML_PRETTY (+125.0%) | JSON_COMPACT (-7.6%) | XML_COMPACT (-22.8%) | JSON_COMPACT (-5.4%) | JSON_PRETTY (-26.0%) | JSON_PRETTY (-25.3%) | XML_PRETTY (-25.4%) |
| CSV (+49.4%) | XML_PRETTY (+131.6%) | TOON_DEFAULT (+51.4%) | XML_COMPACT (+204.2%) | XML_COMPACT (+195.2%) | JSON_PRETTY (+139.3%) | CSV (-15.5%) | TOON_DEFAULT (-24.2%) | CSV (-12.1%) | XML_PRETTY (-34.4%) | XML_COMPACT (-27.7%) | JSON_PRETTY (-27.9%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy By Char | ↓ Eff Score Output Write (Acc By Char) | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 69s | CSV ≈ 6687 | JSON_COMPACT ≈ 208 | YAML ≈ 8803 | JSON_COMPACT ≈ 9070 | CSV ≈ 15867 | XML_PRETTY ≈ 95% | JSON_PRETTY ≈ 76 | TOON_DEFAULT ≈ 80% | JSON_COMPACT ≈ 78 | JSON_COMPACT ≈ 70 | JSON_COMPACT ≈ 72 |
| JSON_COMPACT (+10.0%) | JSON_COMPACT (+30.5%) | XML_PRETTY (+0.5%) | JSON_COMPACT (+0.7%) | YAML (+0.4%) | JSON_COMPACT (+12.2%) | JSON_PRETTY (-2.3%) | JSON_COMPACT (-0.3%) | XML_PRETTY (0.0%) | CSV (-4.8%) | YAML (-3.7%) | CSV (-9.1%) |
| CSV (+10.6%) | XML_COMPACT (+63.1%) | CSV (+44.5%) | CSV (+0.9%) | CSV (+1.2%) | YAML (+31.4%) | TOON_DEFAULT (-5.3%) | XML_PRETTY (-1.9%) | JSON_PRETTY (-1.6%) | XML_COMPACT (-8.7%) | JSON_PRETTY (-6.3%) | YAML (-11.2%) |
| JSON_PRETTY (+24.2%) | TOON_DEFAULT (+72.6%) | JSON_PRETTY (+45.6%) | JSON_PRETTY (+17.3%) | JSON_PRETTY (+17.2%) | XML_COMPACT (+40.6%) | JSON_COMPACT (-9.3%) | YAML (-3.6%) | XML_COMPACT (-2.2%) | TOON_DEFAULT (-9.3%) | XML_PRETTY (-8.6%) | XML_COMPACT (-11.4%) |
| XML_PRETTY (+26.5%) | YAML (+75.6%) | YAML (+47.7%) | XML_COMPACT (+26.0%) | XML_COMPACT (+25.7%) | TOON_DEFAULT (+47.2%) | XML_COMPACT (-9.6%) | TOON_DEFAULT (-7.8%) | JSON_COMPACT (-2.4%) | YAML (-15.4%) | TOON_DEFAULT (-10.1%) | TOON_DEFAULT (-12.0%) |
| XML_COMPACT (+33.1%) | JSON_PRETTY (+99.6%) | XML_COMPACT (+48.2%) | XML_PRETTY (+28.0%) | XML_PRETTY (+26.6%) | JSON_PRETTY (+51.1%) | YAML (-13.2%) | XML_COMPACT (-10.1%) | YAML (-5.9%) | JSON_PRETTY (-18.1%) | XML_COMPACT (-10.4%) | JSON_PRETTY (-15.1%) |
| TOON_DEFAULT (+36.7%) | XML_PRETTY (+125.5%) | TOON_DEFAULT (+49.6%) | TOON_DEFAULT (+30.7%) | TOON_DEFAULT (+30.3%) | XML_PRETTY (+67.4%) | CSV (-37.8%) | CSV (-26.7%) | CSV (-16.9%) | XML_PRETTY (-23.7%) | CSV (-15.1%) | XML_PRETTY (-20.2%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_PRETTY ≈ 100% | CSV ≈ 80% | TOON_DEFAULT ≈ 68% | XML_PRETTY ≈ 71% |
| XML_COMPACT (0.0%) | TOON_DEFAULT (-5.6%) | YAML (-0.0%) | YAML (-6.3%) |
| JSON_COMPACT (-1.2%) | XML_PRETTY (-7.4%) | JSON_PRETTY (-1.6%) | TOON_DEFAULT (-7.1%) |
| YAML (-2.4%) | YAML (-9.9%) | XML_PRETTY (-4.8%) | CSV (-11.1%) |
| TOON_DEFAULT (-8.5%) | XML_COMPACT (-11.1%) | XML_COMPACT (-4.8%) | JSON_PRETTY (-11.1%) |
| XML_PRETTY (-9.1%) | JSON_COMPACT (-12.3%) | JSON_COMPACT (-6.4%) | XML_COMPACT (-22.2%) |
| CSV (-26.7%) | JSON_PRETTY (-12.3%) | CSV (-15.9%) | JSON_COMPACT (-31.7%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| XML_COMPACT ≈ 100% | XML_PRETTY ≈ 84% | TOON_DEFAULT ≈ 67% | TOON_DEFAULT ≈ 50% |
| TOON_DEFAULT (-1.5%) | JSON_PRETTY (-0.0%) | JSON_PRETTY (-2.4%) | JSON_COMPACT (-0.8%) |
| XML_PRETTY (-3.0%) | JSON_COMPACT (-3.7%) | XML_COMPACT (-2.4%) | XML_PRETTY (-2.4%) |
| JSON_COMPACT (-7.3%) | TOON_DEFAULT (-9.3%) | YAML (-4.0%) | YAML (-4.0%) |
| JSON_PRETTY (-7.3%) | YAML (-13.6%) | JSON_COMPACT (-5.6%) | JSON_PRETTY (-4.0%) |
| YAML (-9.7%) | XML_COMPACT (-14.8%) | XML_PRETTY (-5.6%) | XML_COMPACT (-7.1%) |
| CSV (-22.4%) | CSV (-24.7%) | CSV (-10.3%) | CSV (-15.1%) |


#### 2.1.5 Conclusion

The benchmark reveals that token cost and accuracy are not simply inversely correlated. **CSV** is the cheapest format to read but the least accurate and when data becomes sparse its output tokens double without any accuracy improvement. This pattern points to a deeper finding about how format structure influences model reasoning behavior.

Formats with explicit structural markup (**JSON**, **XML**, **TOON**, **YAML**) consistently elicit higher output token counts which reflect the model's reasoning investment through thinking tokens. **CSV**'s mandatory output of only 4,557 tokens is roughly half of every other format and this shallow reasoning directly maps to its accuracy floor. The model appears to treat **CSV** as deceptively simple and fails to engage the deeper analysis that structured formats naturally prompt. When **CSV** encounters optional sparse data the model compensates by producing more verbose output (+101%) but this verbosity does not translate into better answers. The issue is insufficient structural cues for reliable data navigation.

**TOON_DEFAULT** stands out as the most accuracy-robust format (0.54% mandatory-to-optional delta) but pays for this with a 64.43% read token increase on optional data. This happens because the **TOON_DEFAULT** encoding switches from a collapsed tabular representation where field names appear once as a header and records follow as comma-separated rows to expanded key-value pairs when the structure is sparsely filled and not every record contains the same fields. The encoding cannot collapse heterogeneous records into a uniform row format and must instead spell out each field name per record. Despite this token cost increase **TOON_DEFAULT** maintains near-identical accuracy which suggests its encoding is structurally clear enough for the model to parse reliably in both modes.

**YAML** achieves the highest mandatory accuracy (81.18%) but drops the most for optional data (-7.25%) which makes it a strong choice only when data completeness is guaranteed. **JSON_COMPACT** emerges as the best all-around performer for optional data by balancing moderate token cost with solid accuracy (77.42%) and the highest efficiency score (72.47). **JSON_PRETTY** trades efficiency for accuracy on mandatory data (80.65%) but at a steep token premium (27,582 total tokens).

Aggregation questions expose a universal weakness: all formats lose accuracy when data becomes sparse with an average drop of roughly 15 percentage points. This category-specific degradation suggests that the challenge lies not in the format itself but in the increased cognitive load of performing mathematical operations over incomplete data structures where null or missing values must be tracked.

The core takeaway is that the cheapest format is not the most efficient one. Efficiency must account for information fidelity per token spent and by that measure formats that invest tokens in explicit structure consistently outperform the minimalist **CSV** approach. For Haiku 4.5 on flat data with thinking enabled **JSON_COMPACT** offers the best general-purpose efficiency while **TOON_DEFAULT** offers the most predictable accuracy across varying data completeness levels.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy By Char (%) | Useful Output Write Tokens (Acc By Char) | Wasted Output Write Tokens (Acc By Char) | Eff Score Output Write (Acc By Char) | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6968 | 4557 | 11525 | 1.449 | 35.108 | 68.44 | 2979.42 | 1373.91 | 77.88 | 69.08 | 4813.494 | 2154.506 | 3148.206 | 1409.127 | 77.43 | 78.32 | 78.34 |
| CSV | opt | 6687 | 9180 | 15867 | 1.432 | 71.602 | 57.27 | 5084.81 | 3793.85 | 55.49 | 62.90 | 4206.123 | 2480.877 | 5774.011 | 3405.656 | 74.00 | 59.28 | 65.91 |
| JSON_COMPACT | man | 9255 | 11113 | 20368 | 2.152 | 87.903 | 76.37 | 8324.33 | 2575.67 | 62.35 | 75.81 | 7016.216 | 2238.785 | 8424.513 | 2688.154 | 74.90 | 62.16 | 66.55 |
| JSON_COMPACT | opt | 8727 | 9070 | 17797 | 2.118 | 71.462 | 85.75 | 7598.59 | 1262.74 | 75.48 | 77.42 | 6756.443 | 1970.557 | 7021.736 | 2047.931 | 77.70 | 69.80 | 72.47 |
| JSON_PRETTY | man | 14254 | 13328 | 27582 | 1.697 | 105.003 | 83.52 | 10874.58 | 2145.75 | 60.53 | 80.65 | 11495.851 | 2758.149 | 10749.301 | 2579.032 | 62.45 | 58.50 | 56.47 |
| JSON_PRETTY | opt | 13346 | 10628 | 23974 | 1.683 | 83.261 | 92.80 | 9580.98 | 743.35 | 75.70 | 78.23 | 10440.576 | 2905.424 | 8314.024 | 2313.643 | 63.63 | 65.40 | 61.51 |
| TOON_DEFAULT | man | 7018 | 12285 | 19303 | 1.448 | 96.579 | 76.62 | 9175.88 | 2799.95 | 59.06 | 79.30 | 5565.274 | 1452.726 | 9741.807 | 2542.944 | 84.43 | 60.88 | 70.98 |
| TOON_DEFAULT | opt | 11540 | 11821 | 23361 | 1.701 | 92.815 | 89.77 | 10331.63 | 1177.37 | 69.77 | 79.84 | 9213.536 | 2326.464 | 9437.620 | 2383.047 | 70.48 | 62.73 | 63.78 |
| XML_COMPACT | man | 11672 | 13453 | 25125 | 2.370 | 106.812 | 83.95 | 11118.90 | 2125.77 | 60.11 | 78.50 | 9162.520 | 2509.480 | 10560.866 | 2892.467 | 69.13 | 56.60 | 59.55 |
| XML_COMPACT | opt | 10909 | 11402 | 22311 | 2.345 | 89.460 | 85.46 | 9480.08 | 1612.92 | 68.09 | 77.69 | 8475.202 | 2433.798 | 8857.955 | 2543.712 | 70.98 | 62.56 | 64.24 |
| XML_PRETTY | man | 16137 | 9799 | 25936 | 1.937 | 76.578 | 81.27 | 7717.13 | 1778.54 | 70.30 | 79.03 | 12753.071 | 3383.929 | 7744.413 | 2054.920 | 55.35 | 68.60 | 58.41 |
| XML_PRETTY | opt | 15076 | 11479 | 26555 | 1.918 | 90.882 | 95.06 | 10712.63 | 556.71 | 74.25 | 79.84 | 12036.678 | 3039.322 | 9164.568 | 2314.099 | 59.28 | 63.82 | 57.82 |
| YAML | man | 12533 | 11076 | 23609 | 1.666 | 86.874 | 83.97 | 9045.53 | 1726.81 | 68.08 | 81.18 | 10174.289 | 2358.711 | 8991.497 | 2084.503 | 68.28 | 66.04 | 64.26 |
| YAML | opt | 11742 | 9110 | 20852 | 1.653 | 70.989 | 81.90 | 7209.38 | 1593.28 | 72.97 | 73.93 | 8680.861 | 3061.139 | 6735.269 | 2375.064 | 65.71 | 67.22 | 64.33 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6968 | 6687 | -281 | -4.03 | 204 | 301 |  +4525 |  +2218.14 | 4353 | 8878 |  +4622 |  +106.18 | 4557 | 9179 |  +4622 |  +101.43 | 11525 | 15866 |  +4341 |  +37.67 |
| JSON_COMPACT | 9255 | 8727 | -528 | -5.71 | 213 | 209 | -2039 | -957.28 | 10900 | 8861 | -2043 | -18.74 | 11113 | 9070 | -2043 | -18.38 | 20368 | 17797 | -2571 | -12.62 |
| JSON_PRETTY | 14254 | 13346 | -908 | -6.37 | 308 | 303 | -2696 | -875.32 | 13020 | 10324 | -2701 | -20.75 | 13328 | 10627 | -2701 | -20.27 | 27582 | 23973 | -3609 | -13.08 |
| TOON_DEFAULT | 7018 | 11540 |  +4522 |  +64.43 | 309 | 312 | -467 | -151.13 | 11976 | 11509 | -464 | -3.87 | 12285 | 11821 | -464 | -3.78 | 19303 | 23361 |  +4058 |  +21.02 |
| XML_COMPACT | 11672 | 10909 | -763 | -6.54 | 209 | 309 | -2152 | -1029.67 | 13245 | 11093 | -2052 | -15.49 | 13453 | 11401 | -2052 | -15.25 | 25125 | 22310 | -2815 | -11.20 |
| XML_PRETTY | 16137 | 15076 | -1061 | -6.57 | 304 | 210 |  +1774 |  +583.55 | 9496 | 11270 |  +1679 |  +17.68 | 9799 | 11478 |  +1679 |  +17.13 | 25936 | 26554 |  +618 |  +2.38 |
| YAML | 12533 | 11742 | -791 | -6.31 | 304 | 308 | -1970 | -648.03 | 10772 | 8802 | -1966 | -18.25 | 11076 | 9110 | -1966 | -17.75 | 23609 | 20852 | -2757 | -11.68 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 23 | 302.957 | 0.74 | 52278 | 0.083 | 421.59 | 52301 | 303.040 | 337.42 |
| CSV | opt | 14 | 477.643 | 0.45 | 39364 | 0.226 | 317.45 | 39378 | 477.869 | 254.05 |
| JSON_COMPACT | man | 24 | 385.625 | 0.77 | 39580 | 0.275 | 319.19 | 39604 | 385.900 | 255.51 |
| JSON_COMPACT | opt | 13 | 671.308 | 0.42 | 37630 | 0.235 | 303.47 | 37643 | 671.543 | 242.86 |
| JSON_PRETTY | man | 30 | 475.133 | 0.97 | 39401 | 0.330 | 317.75 | 39431 | 475.463 | 254.40 |
| JSON_PRETTY | opt | 36 | 370.722 | 1.16 | 40232 | 0.257 | 324.45 | 40268 | 370.979 | 259.79 |
| TOON_DEFAULT | man | 26 | 269.923 | 0.84 | 38099 | 0.313 | 307.25 | 38125 | 270.236 | 245.97 |
| TOON_DEFAULT | opt | 20 | 577.000 | 0.65 | 37064 | 0.311 | 298.90 | 37084 | 577.312 | 239.25 |
| XML_COMPACT | man | 19 | 614.316 | 0.61 | 37753 | 0.351 | 304.46 | 37772 | 614.667 | 243.69 |
| XML_COMPACT | opt | 23 | 474.304 | 0.74 | 36545 | 0.304 | 294.72 | 36568 | 474.608 | 235.92 |
| XML_PRETTY | man | 18 | 896.500 | 0.58 | 39623 | 0.240 | 319.54 | 39641 | 896.740 | 255.75 |
| XML_PRETTY | opt | 17 | 886.824 | 0.55 | 30600 | 0.368 | 246.78 | 30617 | 887.192 | 197.53 |
| YAML | man | 21 | 596.810 | 0.68 | 38458 | 0.280 | 310.15 | 38479 | 597.090 | 248.25 |
| YAML | opt | 31 | 378.774 | 1.00 | 30155 | 0.292 | 243.19 | 30186 | 379.066 | 194.75 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 23 | 14 | -9 | -39.13 | 52.28 | 39.36 | -12.91 | -24.70 | 52.30 | 39.38 | -12.92 | -24.71 |
| JSON_COMPACT | 24 | 13 | -11 | -45.83 | 39.58 | 37.63 | -1.95 | -4.93 | 39.60 | 37.64 | -1.96 | -4.95 |
| JSON_PRETTY | 30 | 36 |  +6 |  +20.00 | 39.40 | 40.23 |  +0.83 |  +2.11 | 39.43 | 40.27 |  +0.84 |  +2.12 |
| TOON_DEFAULT | 26 | 20 | -6 | -23.08 | 38.10 | 37.06 | -1.03 | -2.72 | 38.12 | 37.08 | -1.04 | -2.73 |
| XML_COMPACT | 19 | 23 |  +4 |  +21.05 | 37.75 | 36.54 | -1.21 | -3.20 | 37.77 | 36.57 | -1.20 | -3.19 |
| XML_PRETTY | 18 | 17 | -1 | -5.56 | 39.62 | 30.60 | -9.02 | -22.77 | 39.64 | 30.62 | -9.02 | -22.76 |
| YAML | 21 | 31 |  +10 |  +47.62 | 38.46 | 30.15 | -8.30 | -21.59 | 38.48 | 30.19 | -8.29 | -21.55 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token |
|---|---|---|---|---|---|---|---|
| CSV | man | 1.449 | 10.217 | 224.774 | 0.991 | 1.516 | 0.599 |
| CSV | opt | 1.432 | 10.597 | 215.710 | 0.941 | 0.685 | 0.396 |
| JSON_COMPACT | man | 2.152 | 13.570 | 298.548 | 0.819 | 0.682 | 0.372 |
| JSON_COMPACT | opt | 2.118 | 13.830 | 281.516 | 0.887 | 0.854 | 0.435 |
| JSON_PRETTY | man | 1.697 | 20.900 | 459.806 | 0.566 | 0.605 | 0.292 |
| JSON_PRETTY | opt | 1.683 | 21.151 | 430.516 | 0.586 | 0.736 | 0.326 |
| TOON_DEFAULT | man | 1.448 | 10.290 | 226.387 | 1.130 | 0.657 | 0.414 |
| TOON_DEFAULT | opt | 1.701 | 18.288 | 372.258 | 0.692 | 0.678 | 0.342 |
| XML_COMPACT | man | 2.370 | 17.114 | 376.516 | 0.673 | 0.583 | 0.312 |
| XML_COMPACT | opt | 2.345 | 17.288 | 351.903 | 0.712 | 0.681 | 0.348 |
| XML_PRETTY | man | 1.937 | 23.661 | 520.548 | 0.490 | 0.806 | 0.305 |
| XML_PRETTY | opt | 1.918 | 23.892 | 486.323 | 0.530 | 0.696 | 0.301 |
| YAML | man | 1.666 | 18.377 | 404.290 | 0.648 | 0.733 | 0.344 |
| YAML | opt | 1.653 | 18.609 | 378.774 | 0.630 | 0.811 | 0.355 |

#### 2.5.2 Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 1.449 | 1.432 | -0.017 | -1.17 | 10.217 | 10.597 |  +0.380 |  +3.72 | 224.774 | 215.710 | -9.064 | -4.03 | 0.991 | 0.941 | -0.050 | -5.05 | 1.516 | 0.685 | -0.831 | -54.82 | 0.599 | 0.396 | -0.203 | -33.89 |
| JSON_COMPACT | 2.152 | 2.118 | -0.034 | -1.58 | 13.570 | 13.830 |  +0.260 |  +1.92 | 298.548 | 281.516 | -17.032 | -5.70 | 0.819 | 0.887 |  +0.068 |  +8.30 | 0.682 | 0.854 |  +0.172 |  +25.22 | 0.372 | 0.435 |  +0.063 |  +16.94 |
| JSON_PRETTY | 1.697 | 1.683 | -0.014 | -0.82 | 20.900 | 21.151 |  +0.251 |  +1.20 | 459.806 | 430.516 | -29.290 | -6.37 | 0.566 | 0.586 |  +0.020 |  +3.53 | 0.605 | 0.736 |  +0.131 |  +21.65 | 0.292 | 0.326 |  +0.034 |  +11.64 |
| TOON_DEFAULT | 1.448 | 1.701 |  +0.253 |  +17.47 | 10.290 | 18.288 |  +7.998 |  +77.73 | 226.387 | 372.258 |  +145.871 |  +64.43 | 1.130 | 0.692 | -0.438 | -38.76 | 0.657 | 0.678 |  +0.021 |  +3.19 | 0.414 | 0.342 | -0.072 | -17.39 |
| XML_COMPACT | 2.370 | 2.345 | -0.025 | -1.05 | 17.114 | 17.288 |  +0.174 |  +1.02 | 376.516 | 351.903 | -24.613 | -6.54 | 0.673 | 0.712 |  +0.039 |  +5.79 | 0.583 | 0.681 |  +0.098 |  +16.81 | 0.312 | 0.348 |  +0.036 |  +11.54 |
| XML_PRETTY | 1.937 | 1.918 | -0.019 | -0.98 | 23.661 | 23.892 |  +0.231 |  +0.98 | 520.548 | 486.323 | -34.225 | -6.57 | 0.490 | 0.530 |  +0.040 |  +8.16 | 0.806 | 0.696 | -0.110 | -13.65 | 0.305 | 0.301 | -0.004 | -1.31 |
| YAML | 1.666 | 1.653 | -0.013 | -0.78 | 18.377 | 18.609 |  +0.232 |  +1.26 | 404.290 | 378.774 | -25.516 | -6.31 | 0.648 | 0.630 | -0.018 | -2.78 | 0.733 | 0.811 |  +0.078 |  +10.64 | 0.344 | 0.355 |  +0.011 |  +3.20 |

### 2.6 Output Write Token Utilization Efficiency (Accuracy By Char)
#### 2.6.1 Metrics
| Format | Variant | Output Write Tokens | Useful Output Write Tokens (Acc By Char) | Wasted Output Write Tokens (Acc By Char) | Accuracy by Char (%) | Eff Score Output Write (Acc By Char) |
|---|---|---|---|---|---|---|
| CSV | man | 4353 | 2979 | 1374 | 68.44 | 77.88 |
| CSV | opt | 8879 | 5085 | 3794 | 57.27 | 55.49 |
| JSON_COMPACT | man | 10900 | 8324 | 2576 | 76.37 | 62.35 |
| JSON_COMPACT | opt | 8861 | 7599 | 1263 | 85.75 | 75.48 |
| JSON_PRETTY | man | 13020 | 10875 | 2146 | 83.52 | 60.53 |
| JSON_PRETTY | opt | 10324 | 9581 | 743 | 92.80 | 75.70 |
| TOON_DEFAULT | man | 11976 | 9176 | 2800 | 76.62 | 59.06 |
| TOON_DEFAULT | opt | 11509 | 10332 | 1177 | 89.77 | 69.77 |
| XML_COMPACT | man | 13245 | 11119 | 2126 | 83.95 | 60.11 |
| XML_COMPACT | opt | 11093 | 9480 | 1613 | 85.46 | 68.09 |
| XML_PRETTY | man | 9496 | 7717 | 1779 | 81.27 | 70.30 |
| XML_PRETTY | opt | 11269 | 10713 | 557 | 95.06 | 74.25 |
| YAML | man | 10772 | 9046 | 1727 | 83.97 | 68.08 |
| YAML | opt | 8803 | 7209 | 1593 | 81.90 | 72.97 |

#### 2.6.2 Read Tokens Mandatory vs Optional Data
| Format | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Useful Output Write Tokens (Acc By Char) Man | Useful Output Write Tokens (Acc By Char) Opt | Diff | Diff (%) | Wasted Output Write Tokens (Acc By Char) Man | Wasted Output Write Tokens (Acc By Char) Opt | Diff | Diff (%) | Accuracy By Char (%) Man | Accuracy By Char (%) Opt | Diff (%) | Eff Score Output Write (Acc By Char) Man | Eff Score Output Write (Acc By Char) Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 4353 | 8878 |  +4525 |  +103.96 | 2979 | 5084 |  +2105 |  +70.67 | 1374 | 3794 |  +2420 |  +176.12 | 68.44 | 57.27 | -11.17 | -16.32 | 77.88 | 55.49 | -22.39 | -28.75 |
| JSON_COMPACT | 10900 | 8861 | -2039 | -18.70 | 8324 | 7598 | -726 | -8.72 | 2576 | 1263 | -1313 | -50.97 | 76.37 | 85.75 |  +9.38 |  +12.28 | 62.35 | 75.48 |  +13.13 |  +21.06 |
| JSON_PRETTY | 13020 | 10324 | -2696 | -20.71 | 10875 | 9581 | -1294 | -11.90 | 2146 | 744 | -1402 | -65.35 | 83.52 | 92.80 |  +9.28 |  +11.11 | 60.53 | 75.70 |  +15.17 |  +25.07 |
| TOON_DEFAULT | 11976 | 11509 | -467 | -3.90 | 9176 | 10332 |  +1156 |  +12.60 | 2800 | 1177 | -1623 | -57.95 | 76.62 | 89.77 |  +13.15 |  +17.16 | 59.06 | 69.77 |  +10.71 |  +18.13 |
| XML_COMPACT | 13245 | 11093 | -2152 | -16.25 | 11119 | 9480 | -1639 | -14.74 | 2126 | 1613 | -513 | -24.12 | 83.95 | 85.46 |  +1.51 |  +1.80 | 60.11 | 68.09 |  +7.98 |  +13.28 |
| XML_PRETTY | 9496 | 11270 |  +1774 |  +18.68 | 7717 | 10712 |  +2995 |  +38.82 | 1779 | 557 | -1222 | -68.68 | 81.27 | 95.06 |  +13.79 |  +16.97 | 70.30 | 74.25 |  +3.94 |  +5.61 |
| YAML | 10772 | 8802 | -1970 | -18.29 | 9046 | 7210 | -1836 | -20.30 | 1727 | 1593 | -134 | -7.73 | 83.97 | 81.90 | -2.07 | -2.47 | 68.08 | 72.97 |  +4.89 |  +7.19 |

### 2.7 Token Utilization Efficiency
#### 2.7.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | man | 6968 | 4813 | 2155 | 4557 | 3148 | 1409 | 11525 | 7962 | 3564 | 69.08 | 77.43 | 78.32 | 78.34 | 69.36 | 77.63 | 78.52 | 78.53 |
| CSV | opt | 6687 | 4206 | 2481 | 9180 | 5774 | 3406 | 15867 | 9980 | 5887 | 62.90 | 74.00 | 59.28 | 65.91 | 62.64 | 73.82 | 59.10 | 65.73 |
| JSON_COMPACT | man | 9255 | 7016 | 2239 | 11113 | 8425 | 2688 | 20368 | 15441 | 4927 | 75.81 | 74.90 | 62.16 | 66.55 | 74.71 | 74.13 | 61.40 | 65.78 |
| JSON_COMPACT | opt | 8727 | 6756 | 1971 | 9070 | 7022 | 2048 | 17797 | 13778 | 4018 | 77.42 | 77.70 | 69.80 | 72.47 | 77.23 | 77.57 | 69.66 | 72.34 |
| JSON_PRETTY | man | 14254 | 11496 | 2758 | 13328 | 10749 | 2579 | 27582 | 22245 | 5337 | 80.65 | 62.45 | 58.50 | 56.47 | 78.73 | 61.11 | 57.16 | 55.13 |
| JSON_PRETTY | opt | 13346 | 10441 | 2905 | 10628 | 8314 | 2314 | 23974 | 18755 | 5219 | 78.23 | 63.63 | 65.40 | 61.51 | 78.58 | 63.88 | 65.65 | 61.76 |
| TOON_DEFAULT | man | 7018 | 5565 | 1453 | 12285 | 9742 | 2543 | 19303 | 15307 | 3996 | 79.30 | 84.43 | 60.88 | 70.98 | 78.36 | 83.77 | 60.22 | 70.32 |
| TOON_DEFAULT | opt | 11540 | 9214 | 2326 | 11821 | 9438 | 2383 | 23361 | 18651 | 4710 | 79.84 | 70.48 | 62.73 | 63.78 | 79.02 | 69.91 | 62.16 | 63.21 |
| XML_COMPACT | man | 11672 | 9163 | 2509 | 13453 | 10561 | 2892 | 25125 | 19723 | 5402 | 78.50 | 69.13 | 56.60 | 59.55 | 77.04 | 68.10 | 55.57 | 58.53 |
| XML_COMPACT | opt | 10909 | 8475 | 2434 | 11402 | 8858 | 2544 | 22311 | 17333 | 4978 | 77.69 | 70.98 | 62.56 | 64.24 | 76.58 | 70.20 | 61.78 | 63.46 |
| XML_PRETTY | man | 16137 | 12753 | 3384 | 9799 | 7744 | 2055 | 25936 | 20497 | 5439 | 79.03 | 55.35 | 68.60 | 58.41 | 77.49 | 54.27 | 67.52 | 57.33 |
| XML_PRETTY | opt | 15076 | 12037 | 3039 | 11479 | 9165 | 2314 | 26555 | 21201 | 5353 | 79.84 | 59.28 | 63.82 | 57.82 | 79.70 | 59.18 | 63.72 | 57.73 |
| YAML | man | 12533 | 10174 | 2359 | 11076 | 8991 | 2085 | 23609 | 19166 | 4443 | 81.18 | 68.28 | 66.04 | 64.26 | 79.47 | 67.08 | 64.84 | 63.06 |
| YAML | opt | 11742 | 8681 | 3061 | 9110 | 6735 | 2375 | 20852 | 15416 | 5436 | 73.93 | 65.71 | 67.22 | 64.33 | 73.38 | 65.32 | 66.84 | 63.94 |

#### 2.7.2 Read Tokens Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 6968 | 6687 | -281 | -4.03 | 4813 | 4206 | -607 | -12.62 | 2155 | 2481 |  +326 |  +15.14 | 69.08 | 62.90 | -6.18 | -8.95 | 77.43 | 74.00 | -3.44 | -4.44 | 69.36 | 62.64 | -6.72 | -9.69 | 77.63 | 73.82 | -3.81 | -4.91 |
| JSON_COMPACT | 9255 | 8727 | -528 | -5.71 | 7016 | 6756 | -260 | -3.70 | 2239 | 1971 | -268 | -11.98 | 75.81 | 77.42 |  +1.61 |  +2.12 | 74.90 | 77.70 |  +2.80 |  +3.74 | 74.71 | 77.23 |  +2.52 |  +3.37 | 74.13 | 77.57 |  +3.44 |  +4.64 |
| JSON_PRETTY | 14254 | 13346 | -908 | -6.37 | 11496 | 10441 | -1055 | -9.18 | 2758 | 2905 |  +147 |  +5.34 | 80.65 | 78.23 | -2.42 | -3.00 | 62.45 | 63.63 |  +1.18 |  +1.89 | 78.73 | 78.58 | -0.15 | -0.19 | 61.11 | 63.88 |  +2.77 |  +4.53 |
| TOON_DEFAULT | 7018 | 11540 |  +4522 |  +64.43 | 5565 | 9213 |  +3648 |  +65.56 | 1453 | 2327 |  +874 |  +60.13 | 79.30 | 79.84 |  +0.54 |  +0.68 | 84.43 | 70.48 | -13.95 | -16.52 | 78.36 | 79.02 |  +0.66 |  +0.84 | 83.77 | 69.91 | -13.86 | -16.55 |
| XML_COMPACT | 11672 | 10909 | -763 | -6.54 | 9163 | 8476 | -687 | -7.50 | 2509 | 2433 | -76 | -3.02 | 78.50 | 77.69 | -0.81 | -1.03 | 69.13 | 70.98 |  +1.85 |  +2.68 | 77.04 | 76.58 | -0.46 | -0.60 | 68.10 | 70.20 |  +2.09 |  +3.08 |
| XML_PRETTY | 16137 | 15076 | -1061 | -6.57 | 12753 | 12037 | -716 | -5.62 | 3384 | 3039 | -345 | -10.18 | 79.03 | 79.84 |  +0.81 |  +1.02 | 55.35 | 59.28 |  +3.93 |  +7.10 | 77.49 | 79.70 |  +2.21 |  +2.85 | 54.27 | 59.18 |  +4.91 |  +9.04 |
| YAML | 12533 | 11742 | -791 | -6.31 | 10174 | 8681 | -1493 | -14.68 | 2359 | 3061 |  +702 |  +29.78 | 81.18 | 73.93 | -7.25 | -8.93 | 68.28 | 65.71 | -2.57 | -3.76 | 79.47 | 73.38 | -6.09 | -7.66 | 67.08 | 65.32 | -1.76 | -2.62 |

#### 2.7.3 Output Tokens Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 4557 | 9179 |  +4622 |  +101.43 | 3148 | 5774 |  +2626 |  +83.41 | 1409 | 3406 |  +1997 |  +141.70 | 69.08 | 62.90 | -6.18 | -8.95 | 78.32 | 59.28 | -19.04 | -24.31 | 69.36 | 62.64 | -6.72 | -9.69 | 78.52 | 59.10 | -19.42 | -24.73 |
| JSON_COMPACT | 11113 | 9070 | -2043 | -18.38 | 8425 | 7022 | -1403 | -16.65 | 2688 | 2048 | -640 | -23.82 | 75.81 | 77.42 |  +1.61 |  +2.12 | 62.16 | 69.80 |  +7.63 |  +12.28 | 74.71 | 77.23 |  +2.52 |  +3.37 | 61.40 | 69.66 |  +8.27 |  +13.47 |
| JSON_PRETTY | 13328 | 10627 | -2701 | -20.26 | 10749 | 8314 | -2435 | -22.66 | 2579 | 2314 | -265 | -10.29 | 80.65 | 78.23 | -2.42 | -3.00 | 58.50 | 65.40 |  +6.90 |  +11.80 | 78.73 | 78.58 | -0.15 | -0.19 | 57.16 | 65.65 |  +8.49 |  +14.86 |
| TOON_DEFAULT | 12285 | 11821 | -464 | -3.78 | 9742 | 9438 | -304 | -3.12 | 2543 | 2383 | -160 | -6.29 | 79.30 | 79.84 |  +0.54 |  +0.68 | 60.88 | 62.73 |  +1.86 |  +3.05 | 78.36 | 79.02 |  +0.66 |  +0.84 | 60.22 | 62.16 |  +1.94 |  +3.22 |
| XML_COMPACT | 13453 | 11401 | -2052 | -15.25 | 10561 | 8858 | -1703 | -16.12 | 2892 | 2543 | -349 | -12.06 | 78.50 | 77.69 | -0.81 | -1.03 | 56.60 | 62.56 |  +5.97 |  +10.54 | 77.04 | 76.58 | -0.46 | -0.60 | 55.57 | 61.78 |  +6.21 |  +11.17 |
| XML_PRETTY | 9799 | 11478 |  +1679 |  +17.14 | 7744 | 9164 |  +1420 |  +18.34 | 2055 | 2314 |  +259 |  +12.61 | 79.03 | 79.84 |  +0.81 |  +1.02 | 68.60 | 63.82 | -4.78 | -6.97 | 77.49 | 79.70 |  +2.21 |  +2.85 | 67.52 | 63.72 | -3.80 | -5.63 |
| YAML | 11076 | 9110 | -1966 | -17.75 | 8991 | 6735 | -2256 | -25.09 | 2085 | 2376 |  +291 |  +13.94 | 81.18 | 73.93 | -7.25 | -8.93 | 66.04 | 67.22 |  +1.18 |  +1.79 | 79.47 | 73.38 | -6.09 | -7.66 | 64.84 | 66.84 |  +2.00 |  +3.08 |

#### 2.7.4 Total Tokens Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 11525 | 15866 |  +4341 |  +37.67 | 7962 | 9980 |  +2018 |  +25.35 | 3564 | 5887 |  +2323 |  +65.18 | 69.08 | 62.90 | -6.18 | -8.95 | 78.34 | 65.91 | -12.43 | -15.86 | 69.36 | 62.64 | -6.72 | -9.69 | 78.53 | 65.73 | -12.81 | -16.31 |
| JSON_COMPACT | 20368 | 17797 | -2571 | -12.62 | 15441 | 13778 | -1663 | -10.77 | 4927 | 4019 | -908 | -18.44 | 75.81 | 77.42 |  +1.61 |  +2.12 | 66.55 | 72.47 |  +5.92 |  +8.90 | 74.71 | 77.23 |  +2.52 |  +3.37 | 65.78 | 72.34 |  +6.56 |  +9.98 |
| JSON_PRETTY | 27582 | 23973 | -3609 | -13.08 | 22245 | 18754 | -3491 | -15.69 | 5337 | 5219 | -118 | -2.21 | 80.65 | 78.23 | -2.42 | -3.00 | 56.47 | 61.51 |  +5.04 |  +8.92 | 78.73 | 78.58 | -0.15 | -0.19 | 55.13 | 61.76 |  +6.63 |  +12.02 |
| TOON_DEFAULT | 19303 | 23361 |  +4058 |  +21.02 | 15307 | 18651 |  +3344 |  +21.85 | 3996 | 4710 |  +714 |  +17.86 | 79.30 | 79.84 |  +0.54 |  +0.68 | 70.98 | 63.78 | -7.19 | -10.14 | 78.36 | 79.02 |  +0.66 |  +0.84 | 70.32 | 63.21 | -7.11 | -10.11 |
| XML_COMPACT | 25125 | 22310 | -2815 | -11.20 | 19723 | 17333 | -2390 | -12.12 | 5402 | 4978 | -424 | -7.86 | 78.50 | 77.69 | -0.81 | -1.03 | 59.55 | 64.24 |  +4.69 |  +7.87 | 77.04 | 76.58 | -0.46 | -0.60 | 58.53 | 63.46 |  +4.93 |  +8.42 |
| XML_PRETTY | 25936 | 26554 |  +618 |  +2.38 | 20497 | 21201 |  +704 |  +3.43 | 5439 | 5354 | -85 | -1.57 | 79.03 | 79.84 |  +0.81 |  +1.02 | 58.41 | 57.82 | -0.59 | -1.00 | 77.49 | 79.70 |  +2.21 |  +2.85 | 57.33 | 57.73 |  +0.39 |  +0.69 |
| YAML | 23609 | 20852 | -2757 | -11.68 | 19166 | 15416 | -3750 | -19.56 | 4443 | 5436 |  +993 |  +22.35 | 81.18 | 73.93 | -7.25 | -8.93 | 64.26 | 64.33 |  +0.07 |  +0.11 | 79.47 | 73.38 | -6.09 | -7.66 | 63.06 | 63.94 |  +0.88 |  +1.40 |

### 2.8 Answer Per Format Breakdown
#### 2.8.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| CSV | man | 86 | 38 | 0 | 69.08 |
| CSV | opt | 78 | 46 | 0 | 62.90 |
| JSON_COMPACT | man | 94 | 30 | 0 | 75.81 |
| JSON_COMPACT | opt | 96 | 28 | 0 | 77.42 |
| JSON_PRETTY | man | 100 | 24 | 0 | 80.65 |
| JSON_PRETTY | opt | 97 | 27 | 0 | 78.23 |
| TOON_DEFAULT | man | 98 | 26 | 0 | 79.30 |
| TOON_DEFAULT | opt | 99 | 25 | 0 | 79.84 |
| XML_COMPACT | man | 97 | 27 | 0 | 78.50 |
| XML_COMPACT | opt | 96 | 28 | 0 | 77.69 |
| XML_PRETTY | man | 98 | 26 | 0 | 79.03 |
| XML_PRETTY | opt | 99 | 25 | 0 | 79.84 |
| YAML | man | 101 | 23 | 0 | 81.18 |
| YAML | opt | 92 | 32 | 0 | 73.93 |

#### 2.8.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV | 86 | 78 | -8 | -9.30 | 38 | 46 |  +8 |  +21.05 | 0 | 0 | 0 | 0.00 | 69.08 | 62.90 | -6.18 |
| JSON_COMPACT | 94 | 96 |  +2 |  +2.13 | 30 | 28 | -2 | -6.67 | 0 | 0 | 0 | 0.00 | 75.81 | 77.42 |  +1.61 |
| JSON_PRETTY | 100 | 97 | -3 | -3.00 | 24 | 27 |  +3 |  +12.50 | 0 | 0 | 0 | 0.00 | 80.65 | 78.23 | -2.42 |
| TOON_DEFAULT | 98 | 99 |  +1 |  +1.02 | 26 | 25 | -1 | -3.85 | 0 | 0 | 0 | 0.00 | 79.30 | 79.84 |  +0.54 |
| XML_COMPACT | 97 | 96 | -1 | -1.03 | 27 | 28 |  +1 |  +3.70 | 0 | 0 | 0 | 0.00 | 78.50 | 77.69 | -0.81 |
| XML_PRETTY | 98 | 99 |  +1 |  +1.02 | 26 | 25 | -1 | -3.85 | 0 | 0 | 0 | 0.00 | 79.03 | 79.84 |  +0.81 |
| YAML | 101 | 92 | -9 | -8.91 | 23 | 32 |  +9 |  +39.13 | 0 | 0 | 0 | 0.00 | 81.18 | 73.93 | -7.25 |

### 2.9 Accuracy Per Question Category Analysis
#### 2.9.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| CSV | man | 69.08 | 73.33 | 80.25 | 52.38 | 60.32 |
| CSV | opt | 62.90 | 77.57 | 59.26 | 57.14 | 34.92 |
| JSON_COMPACT | man | 75.81 | 98.79 | 67.90 | 61.90 | 39.68 |
| JSON_COMPACT | opt | 77.42 | 92.73 | 80.25 | 61.90 | 49.21 |
| JSON_PRETTY | man | 80.65 | 100.00 | 67.90 | 66.66 | 60.32 |
| JSON_PRETTY | opt | 78.23 | 92.73 | 83.95 | 65.08 | 46.03 |
| TOON_DEFAULT | man | 79.30 | 91.51 | 74.69 | 68.26 | 64.28 |
| TOON_DEFAULT | opt | 79.84 | 98.48 | 74.69 | 67.46 | 50.00 |
| XML_COMPACT | man | 78.50 | 100.00 | 69.14 | 63.49 | 49.21 |
| XML_COMPACT | opt | 77.69 | 100.00 | 69.14 | 65.08 | 42.86 |
| XML_PRETTY | man | 79.03 | 90.91 | 72.84 | 63.49 | 71.43 |
| XML_PRETTY | opt | 79.84 | 96.97 | 83.95 | 61.90 | 47.62 |
| YAML | man | 81.18 | 97.57 | 70.37 | 68.25 | 65.08 |
| YAML | opt | 73.93 | 90.30 | 70.37 | 63.49 | 46.03 |

#### 2.9.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 73.33 | 77.57 |  +4.24 |
| JSON_COMPACT | 98.79 | 92.73 | -6.06 |
| JSON_PRETTY | 100.00 | 92.73 | -7.27 |
| TOON_DEFAULT | 91.51 | 98.48 |  +6.97 |
| XML_COMPACT | 100.00 | 100.00 | 0.00 |
| XML_PRETTY | 90.91 | 96.97 |  +6.06 |
| YAML | 97.57 | 90.30 | -7.27 |

#### 2.9.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 80.25 | 59.26 | -20.99 |
| JSON_COMPACT | 67.90 | 80.25 |  +12.35 |
| JSON_PRETTY | 67.90 | 83.95 |  +16.05 |
| TOON_DEFAULT | 74.69 | 74.69 |  +0.00 |
| XML_COMPACT | 69.14 | 69.14 | 0.00 |
| XML_PRETTY | 72.84 | 83.95 |  +11.11 |
| YAML | 70.37 | 70.37 | 0.00 |

#### 2.9.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 52.38 | 57.14 |  +4.76 |
| JSON_COMPACT | 61.90 | 61.90 |  +0.00 |
| JSON_PRETTY | 66.66 | 65.08 | -1.58 |
| TOON_DEFAULT | 68.26 | 67.46 | -0.80 |
| XML_COMPACT | 63.49 | 65.08 |  +1.59 |
| XML_PRETTY | 63.49 | 61.90 | -1.59 |
| YAML | 68.25 | 63.49 | -4.76 |

#### 2.9.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| CSV | 60.32 | 34.92 | -25.40 |
| JSON_COMPACT | 39.68 | 49.21 |  +9.52 |
| JSON_PRETTY | 60.32 | 46.03 | -14.29 |
| TOON_DEFAULT | 64.28 | 50.00 | -14.29 |
| XML_COMPACT | 49.21 | 42.86 | -6.35 |
| XML_PRETTY | 71.43 | 47.62 | -23.81 |
| YAML | 65.08 | 46.03 | -19.04 |

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

- **Report Generated**: 2026-04-11
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code < 2.1.86
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_flat_all_formats_and_variants_off\BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_flat_all_formats_and_variants_on\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_off\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_on_verify\BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)