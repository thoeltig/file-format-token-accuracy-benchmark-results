# File Format Token Efficiency Benchmark: Comprehensive Report

## 1. Methodology

### 1.1 Research Purpose

The underlying question: **Which file format delivers maximum information value per token consumed?**

This requires measuring:
- **Token Cost**: How many tokens does each format consume for equivalent data?
- **Information Fidelity**: How accurately can the model understand and answer questions about the data?
- **Robustness**: How consistent is performance across data variants (mandatory vs optional fields)?

### 1.2 Test Design

#### 1.2.1 Data Generation
- 2 variants per format: mandatory (22 fields, dense) and optional (19 mandatory + 3 optional, sparse)
- Record Counts: 31

| Formats | Description |
|---|---|
| **CSV** | Comma-separated values — flat structure only |
| **JSON_COMPACT** | Minified JSON (no whitespace or newlines) |
| **JSON_PRETTY** | Standard indented JSON |
| **TOON_DEFAULT** | [Token-Oriented Object Notation v2.1.0](https://toonformat.dev/) with key folding disabled — nested structures expanded as-is |
| **XML_COMPACT** | Minified XML (no whitespace or indentation) |
| **XML_PRETTY** | Standard indented XML |
| **YAML** | Standard YAML with hierarchical indentation |

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
Composite metric balancing accuracy with normalized token count (favour towards accuracy). Each efficieny score has an indicator which token count was used in the calculation.
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

## 2. Simplified Results

*Note: All columns ranked best-to-worst. ↑ = lower value is better (ascending). ↓ = higher value is better (descending).*

### 2.1 Flat Structure With Thinking On

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)

#### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| CSV ≈ 75s | TOON_DEFAULT ≈ 7048 | JSON_COMPACT ≈ 228 | CSV ≈ 8957 | CSV ≈ 9256 | CSV ≈ 16749 | JSON_PRETTY ≈ 80.91% | TOON_DEFAULT ≈ 86 | CSV ≈ 79 | CSV ≈ 84 | JSON_PRETTY ≈ 97.83% | TOON_DEFAULT ≈ 95 | XML_COMPACT ≈ 92 | CSV ≈ 95 |
| XML_COMPACT (+0.63%) | CSV (+6.31%) | YAML (+1.46%) | XML_COMPACT (+1.44%) | XML_COMPACT (+1.86%) | TOON_DEFAULT (+11.76%) | TOON_DEFAULT (-2.55%) | CSV (-3.68%) | XML_COMPACT (-1.41%) | TOON_DEFAULT (-4.58%) | JSON_COMPACT (-0.36%) | CSV (-1.60%) | CSV (-2.10%) | TOON_DEFAULT (-5.77%) |
| YAML (+17.93%) | JSON_COMPACT (+31.50%) | TOON_DEFAULT (+26.06%) | YAML (+15.60%) | YAML (+14.35%) | XML_COMPACT (+26.10%) | XML_PRETTY (-4.30%) | JSON_COMPACT (-13.34%) | YAML (-10.10%) | XML_COMPACT (-14.41%) | XML_PRETTY (-0.85%) | JSON_COMPACT (-4.86%) | YAML (-6.71%) | XML_COMPACT (-9.55%) |
| TOON_DEFAULT (+28.20%) | XML_COMPACT (+65.91%) | CSV (+31.63%) | TOON_DEFAULT (+27.10%) | TOON_DEFAULT (+26.08%) | JSON_COMPACT (+33.91%) | CSV (-4.84%) | XML_COMPACT (-21.80%) | TOON_DEFAULT (-14.77%) | JSON_COMPACT (-20.57%) | YAML (-0.89%) | XML_COMPACT (-14.74%) | TOON_DEFAULT (-16.59%) | JSON_COMPACT (-12.77%) |
| XML_PRETTY (+37.08%) | YAML (+78.12%) | JSON_PRETTY (+49.78%) | XML_PRETTY (+38.86%) | XML_PRETTY (+38.07%) | YAML (+38.15%) | XML_COMPACT (-5.10%) | YAML (-26.10%) | JSON_PRETTY (-22.77%) | YAML (-21.60%) | XML_COMPACT (-1.13%) | YAML (-17.88%) | XML_PRETTY (-19.74%) | YAML (-15.18%) |
| JSON_PRETTY (+41.63%) | JSON_PRETTY (+102.65%) | XML_COMPACT (+50.51%) | JSON_PRETTY (+42.88%) | JSON_PRETTY (+41.94%) | JSON_PRETTY (+63.72%) | YAML (-5.91%) | JSON_PRETTY (-28.87%) | XML_PRETTY (-23.93%) | JSON_PRETTY (-30.82%) | CSV (-5.43%) | JSON_PRETTY (-23.91%) | JSON_PRETTY (-21.26%) | JSON_PRETTY (-26.87%) |
| JSON_COMPACT (+46.68%) | XML_PRETTY (+129.37%) | XML_PRETTY (+50.51%) | JSON_COMPACT (+44.40%) | JSON_COMPACT (+42.18%) | XML_PRETTY (+72.82%) | JSON_COMPACT (-7.52%) | XML_PRETTY (-40.25%) | JSON_COMPACT (-29.28%) | XML_PRETTY (-39.18%) | TOON_DEFAULT (-5.59%) | XML_PRETTY (-31.75%) | JSON_COMPACT (-21.65%) | XML_PRETTY (-31.85%) |

#### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 73s | CSV ≈ 7225 | XML_PRETTY ≈ 232 | JSON_PRETTY ≈ 7966 | JSON_PRETTY ≈ 8302 | CSV ≈ 18282 | XML_PRETTY ≈ 79.57% | CSV ≈ 82 | JSON_PRETTY ≈ 84 | JSON_COMPACT ≈ 80 | XML_COMPACT ≈ 97.93% | CSV ≈ 95 | JSON_PRETTY ≈ 98 | CSV ≈ 92 |
| XML_PRETTY (+7.93%) | JSON_COMPACT (+21.08%) | CSV (+30.56%) | JSON_COMPACT (+21.36%) | JSON_COMPACT (+20.52%) | JSON_COMPACT (+2.58%) | TOON_DEFAULT (-0.40%) | JSON_COMPACT (-3.08%) | XML_PRETTY (-8.43%) | CSV (-2.21%) | YAML (-0.02%) | JSON_COMPACT (-4.51%) | XML_PRETTY (-9.06%) | JSON_COMPACT (-0.03%) |
| JSON_COMPACT (+13.19%) | XML_COMPACT (+51.28%) | YAML (+43.47%) | XML_PRETTY (+22.97%) | XML_PRETTY (+20.79%) | JSON_PRETTY (+18.53%) | XML_COMPACT (-0.54%) | XML_COMPACT (-11.96%) | JSON_COMPACT (-9.56%) | JSON_PRETTY (-11.55%) | XML_PRETTY (-0.51%) | XML_COMPACT (-11.47%) | JSON_COMPACT (-9.93%) | JSON_PRETTY (-8.24%) |
| CSV (+19.75%) | TOON_DEFAULT (+60.01%) | XML_COMPACT (+44.33%) | YAML (+27.74%) | YAML (+26.59%) | YAML (+21.87%) | JSON_COMPACT (-1.61%) | TOON_DEFAULT (-14.66%) | YAML (-13.04%) | XML_COMPACT (-11.67%) | JSON_PRETTY (-1.36%) | YAML (-14.70%) | YAML (-11.41%) | YAML (-9.09%) |
| YAML (+21.03%) | YAML (+62.92%) | JSON_PRETTY (+44.62%) | CSV (+34.99%) | CSV (+33.18%) | XML_COMPACT (+22.68%) | YAML (-1.88%) | YAML (-16.81%) | XML_COMPACT (-18.40%) | YAML (-12.28%) | JSON_COMPACT (-1.98%) | TOON_DEFAULT (-15.54%) | XML_COMPACT (-16.92%) | XML_COMPACT (-9.51%) |
| TOON_DEFAULT (+28.94%) | JSON_PRETTY (+85.01%) | JSON_COMPACT (+45.34%) | XML_COMPACT (+40.13%) | XML_COMPACT (+38.50%) | TOON_DEFAULT (+27.61%) | JSON_PRETTY (-3.50%) | JSON_PRETTY (-25.27%) | CSV (-20.01%) | TOON_DEFAULT (-14.63%) | TOON_DEFAULT (-2.38%) | JSON_PRETTY (-21.74%) | CSV (-17.09%) | TOON_DEFAULT (-13.91%) |
| XML_COMPACT (+30.13%) | XML_PRETTY (+108.84%) | TOON_DEFAULT (+46.41%) | TOON_DEFAULT (+43.45%) | TOON_DEFAULT (+41.74%) | XML_PRETTY (+37.39%) | CSV (-6.18%) | XML_PRETTY (-30.11%) | TOON_DEFAULT (-20.03%) | XML_PRETTY (-20.41%) | CSV (-3.87%) | XML_PRETTY (-27.73%) | TOON_DEFAULT (-20.04%) | XML_PRETTY (-17.87%) |


### 2.2 Flat Structure With Thinking Off

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)

#### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 76s | CSV ≈ 7063 | TOON_DEFAULT ≈ 258 | XML_COMPACT ≈ 6673 | XML_COMPACT ≈ 6976 | CSV ≈ 17225 | JSON_PRETTY ≈ 81.18% | CSV ≈ 84 | XML_COMPACT ≈ 82 | CSV ≈ 85 | JSON_COMPACT ≈ 97.50% | TOON_DEFAULT ≈ 94 | XML_COMPACT ≈ 96 | CSV ≈ 93 |
| XML_COMPACT (+5.78%) | TOON_DEFAULT (+1.68%) | CSV (+16.50%) | YAML (+35.93%) | YAML (+34.36%) | TOON_DEFAULT (+9.13%) | JSON_COMPACT (-3.56%) | TOON_DEFAULT (-1.93%) | YAML (-14.97%) | TOON_DEFAULT (-7.86%) | JSON_PRETTY (-0.51%) | CSV (-2.20%) | YAML (-11.51%) | XML_COMPACT (-2.42%) |
| CSV (+8.43%) | JSON_COMPACT (+31.62%) | JSON_PRETTY (+17.15%) | CSV (+47.78%) | CSV (+45.67%) | XML_COMPACT (+9.88%) | CSV (-4.03%) | JSON_COMPACT (-9.07%) | CSV (-16.18%) | XML_COMPACT (-10.58%) | YAML (-1.29%) | JSON_COMPACT (-4.57%) | XML_PRETTY (-18.48%) | TOON_DEFAULT (-3.23%) |
| XML_PRETTY (+13.92%) | XML_COMPACT (+69.19%) | YAML (+17.15%) | XML_PRETTY (+54.29%) | XML_PRETTY (+51.95%) | YAML (+27.41%) | TOON_DEFAULT (-5.82%) | XML_COMPACT (-24.30%) | XML_PRETTY (-21.33%) | JSON_COMPACT (-22.46%) | XML_PRETTY (-1.95%) | XML_COMPACT (-16.74%) | CSV (-20.70%) | YAML (-12.53%) |
| TOON_DEFAULT (+25.23%) | YAML (+78.03%) | XML_COMPACT (+17.54%) | TOON_DEFAULT (+70.19%) | TOON_DEFAULT (+66.50%) | JSON_COMPACT (+32.28%) | XML_PRETTY (-6.99%) | YAML (-26.93%) | JSON_PRETTY (-24.22%) | YAML (-22.98%) | XML_COMPACT (-3.01%) | YAML (-17.88%) | JSON_PRETTY (-24.77%) | JSON_COMPACT (-14.75%) |
| JSON_PRETTY (+26.80%) | JSON_PRETTY (+102.62%) | JSON_COMPACT (+18.06%) | JSON_PRETTY (+74.95%) | JSON_PRETTY (+71.68%) | JSON_PRETTY (+52.61%) | XML_COMPACT (-8.60%) | JSON_PRETTY (-27.43%) | TOON_DEFAULT (-26.71%) | JSON_PRETTY (-34.04%) | TOON_DEFAULT (-4.80%) | JSON_PRETTY (-23.89%) | TOON_DEFAULT (-25.83%) | JSON_PRETTY (-28.28%) |
| JSON_COMPACT (+40.46%) | XML_PRETTY (+129.17%) | XML_PRETTY (+18.19%) | JSON_COMPACT (+97.57%) | JSON_COMPACT (+93.36%) | XML_PRETTY (+55.51%) | YAML (-8.60%) | XML_PRETTY (-40.92%) | JSON_COMPACT (-36.58%) | XML_PRETTY (-41.59%) | CSV (-8.52%) | XML_PRETTY (-32.00%) | JSON_COMPACT (-32.43%) | XML_PRETTY (-31.19%) |

#### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 74s | CSV ≈ 6795 | JSON_PRETTY ≈ 231 | TOON_DEFAULT ≈ 8675 | TOON_DEFAULT ≈ 8984 | JSON_COMPACT ≈ 18326 | YAML ≈ 79.84% | CSV ≈ 82 | TOON_DEFAULT ≈ 73 | JSON_COMPACT ≈ 80 | YAML ≈ 98.41% | CSV ≈ 95 | TOON_DEFAULT ≈ 87 | JSON_COMPACT ≈ 94 |
| XML_PRETTY (+0.30%) | JSON_COMPACT (+29.04%) | CSV (+29.29%) | XML_PRETTY (+2.68%) | XML_PRETTY (+2.51%) | CSV (+1.74%) | JSON_PRETTY (-1.77%) | JSON_COMPACT (-6.07%) | XML_PRETTY (-0.44%) | CSV (-3.89%) | XML_COMPACT (-0.55%) | JSON_COMPACT (-4.99%) | XML_PRETTY (-1.01%) | CSV (-3.58%) |
| JSON_COMPACT (+2.14%) | XML_COMPACT (+64.55%) | XML_PRETTY (+30.74%) | JSON_COMPACT (+6.66%) | JSON_COMPACT (+6.38%) | TOON_DEFAULT (+12.92%) | XML_COMPACT (-1.88%) | XML_COMPACT (-15.00%) | JSON_COMPACT (-2.82%) | TOON_DEFAULT (-11.35%) | JSON_PRETTY (-1.15%) | XML_COMPACT (-12.83%) | JSON_COMPACT (-2.61%) | TOON_DEFAULT (-9.51%) |
| JSON_PRETTY (+14.79%) | TOON_DEFAULT (+72.32%) | YAML (+31.60%) | JSON_PRETTY (+17.35%) | JSON_PRETTY (+15.88%) | XML_COMPACT (+22.49%) | JSON_COMPACT (-3.71%) | YAML (-16.11%) | JSON_PRETTY (-7.01%) | XML_COMPACT (-16.35%) | JSON_COMPACT (-2.17%) | YAML (-14.71%) | JSON_PRETTY (-6.86%) | XML_COMPACT (-14.16%) |
| XML_COMPACT (+18.40%) | YAML (+73.52%) | JSON_COMPACT (+31.95%) | XML_COMPACT (+26.30%) | XML_COMPACT (+25.39%) | JSON_PRETTY (+29.90%) | XML_PRETTY (-3.77%) | TOON_DEFAULT (-19.83%) | XML_COMPACT (-13.10%) | JSON_PRETTY (-22.15%) | XML_PRETTY (-2.75%) | TOON_DEFAULT (-16.63%) | XML_COMPACT (-11.44%) | JSON_PRETTY (-19.64%) |
| CSV (+26.29%) | JSON_PRETTY (+97.13%) | XML_COMPACT (+33.77%) | CSV (+33.16%) | CSV (+31.90%) | XML_PRETTY (+32.75%) | TOON_DEFAULT (-5.02%) | JSON_PRETTY (-24.47%) | CSV (-21.61%) | YAML (-25.08%) | TOON_DEFAULT (-3.17%) | JSON_PRETTY (-21.48%) | CSV (-18.72%) | YAML (-22.59%) |
| YAML (+40.42%) | XML_PRETTY (+122.47%) | TOON_DEFAULT (+33.84%) | YAML (+46.69%) | YAML (+45.02%) | YAML (+35.44%) | CSV (-6.72%) | XML_PRETTY (-33.53%) | YAML (-23.72%) | XML_PRETTY (-26.07%) | CSV (-5.53%) | XML_PRETTY (-29.01%) | YAML (-21.41%) | XML_PRETTY (-22.71%) |


### 2.3 Nested Structure With Thinking On

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)

*CSV excluded — flat-only format, does not support nested structures.*

#### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT ≈ 69s | JSON_COMPACT ≈ 10315 | JSON_COMPACT ≈ 336 | XML_PRETTY ≈ 7175 | XML_PRETTY ≈ 7518 | JSON_COMPACT ≈ 18446 | YAML ≈ 77.42% | JSON_COMPACT ≈ 81 | XML_PRETTY ≈ 81 | JSON_COMPACT ≈ 82 | JSON_PRETTY ≈ 96.67% | JSON_COMPACT ≈ 92 | XML_PRETTY ≈ 95 | JSON_COMPACT ≈ 93 |
| XML_PRETTY (+2.18%) | XML_COMPACT (+24.56%) | YAML (+0.50%) | JSON_COMPACT (+8.64%) | JSON_COMPACT (+8.16%) | XML_COMPACT (+18.26%) | JSON_PRETTY (-1.88%) | XML_COMPACT (-10.73%) | JSON_COMPACT (-1.51%) | XML_COMPACT (-13.01%) | YAML (-0.40%) | XML_COMPACT (-7.23%) | TOON_DEFAULT (-5.10%) | XML_COMPACT (-9.29%) |
| TOON_DEFAULT (+6.95%) | TOON_DEFAULT (+36.66%) | TOON_DEFAULT (+1.74%) | TOON_DEFAULT (+16.27%) | TOON_DEFAULT (+15.52%) | TOON_DEFAULT (+23.50%) | JSON_COMPACT (-3.23%) | YAML (-13.21%) | TOON_DEFAULT (-5.70%) | TOON_DEFAULT (-17.11%) | TOON_DEFAULT (-2.16%) | YAML (-9.88%) | JSON_COMPACT (-5.32%) | TOON_DEFAULT (-11.25%) |
| XML_COMPACT (+8.00%) | YAML (+38.69%) | XML_PRETTY (+1.88%) | XML_COMPACT (+20.16%) | XML_COMPACT (+19.27%) | YAML (+37.75%) | XML_COMPACT (-4.03%) | TOON_DEFAULT (-16.24%) | XML_COMPACT (-6.64%) | YAML (-22.94%) | XML_PRETTY (-2.41%) | TOON_DEFAULT (-10.42%) | XML_COMPACT (-7.52%) | YAML (-18.51%) |
| YAML (+31.78%) | JSON_PRETTY (+72.84%) | XML_COMPACT (+2.58%) | JSON_PRETTY (+43.88%) | JSON_PRETTY (+43.27%) | XML_PRETTY (+49.79%) | TOON_DEFAULT (-4.71%) | JSON_PRETTY (-28.75%) | JSON_PRETTY (-14.53%) | XML_PRETTY (-35.45%) | XML_COMPACT (-3.79%) | JSON_PRETTY (-21.92%) | JSON_PRETTY (-13.03%) | XML_PRETTY (-27.14%) |
| JSON_PRETTY (+34.89%) | XML_PRETTY (+95.00%) | JSON_PRETTY (+33.04%) | YAML (+50.05%) | YAML (+47.71%) | JSON_PRETTY (+55.04%) | XML_PRETTY (-5.38%) | XML_PRETTY (-40.71%) | YAML (-14.76%) | JSON_PRETTY (-36.16%) | JSON_COMPACT (-6.05%) | XML_PRETTY (-31.67%) | YAML (-14.82%) | JSON_PRETTY (-28.55%) |

#### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XML_COMPACT ≈ 64s | JSON_COMPACT ≈ 9788 | JSON_PRETTY ≈ 228 | XML_COMPACT ≈ 7011 | XML_COMPACT ≈ 7356 | JSON_COMPACT ≈ 18250 | YAML ≈ 79.03% | JSON_COMPACT ≈ 84 | XML_COMPACT ≈ 83 | JSON_COMPACT ≈ 84 | YAML ≈ 98.14% | JSON_COMPACT ≈ 97 | XML_COMPACT ≈ 96 | JSON_COMPACT ≈ 97 |
| XML_PRETTY (+21.07%) | XML_COMPACT (+26.36%) | JSON_COMPACT (+47.44%) | JSON_COMPACT (+15.90%) | JSON_COMPACT (+15.04%) | XML_COMPACT (+8.07%) | TOON_DEFAULT (-0.94%) | XML_COMPACT (-11.34%) | JSON_COMPACT (-4.24%) | XML_COMPACT (-6.75%) | TOON_DEFAULT (-1.43%) | XML_COMPACT (-9.07%) | JSON_COMPACT (-4.48%) | XML_COMPACT (-5.07%) |
| JSON_PRETTY (+31.75%) | TOON_DEFAULT (+41.59%) | YAML (+47.58%) | XML_PRETTY (+33.45%) | JSON_PRETTY (+31.31%) | TOON_DEFAULT (+42.74%) | JSON_COMPACT (-2.42%) | YAML (-14.37%) | XML_PRETTY (-13.69%) | TOON_DEFAULT (-26.71%) | JSON_COMPACT (-2.93%) | YAML (-12.18%) | JSON_PRETTY (-10.58%) | TOON_DEFAULT (-23.27%) |
| JSON_COMPACT (+37.52%) | YAML (+43.57%) | XML_PRETTY (+51.10%) | JSON_PRETTY (+34.52%) | XML_PRETTY (+31.88%) | JSON_PRETTY (+45.52%) | XML_COMPACT (-4.30%) | TOON_DEFAULT (-14.38%) | JSON_PRETTY (-14.98%) | YAML (-29.18%) | XML_COMPACT (-3.63%) | TOON_DEFAULT (-12.52%) | XML_PRETTY (-11.08%) | YAML (-25.09%) |
| YAML (+47.75%) | JSON_PRETTY (+72.65%) | XML_COMPACT (+51.24%) | TOON_DEFAULT (+68.86%) | TOON_DEFAULT (+65.74%) | YAML (+47.67%) | XML_PRETTY (-6.18%) | JSON_PRETTY (-31.61%) | TOON_DEFAULT (-22.42%) | JSON_PRETTY (-34.15%) | JSON_PRETTY (-4.00%) | JSON_PRETTY (-24.41%) | TOON_DEFAULT (-20.16%) | JSON_PRETTY (-26.63%) |
| TOON_DEFAULT (+51.18%) | XML_PRETTY (+100.07%) | TOON_DEFAULT (+54.50%) | YAML (+79.14%) | YAML (+75.33%) | XML_PRETTY (+60.46%) | JSON_PRETTY (-8.06%) | XML_PRETTY (-40.37%) | YAML (-25.33%) | XML_PRETTY (-42.41%) | XML_PRETTY (-4.45%) | XML_PRETTY (-33.66%) | YAML (-22.33%) | XML_PRETTY (-35.43%) |


### 2.4 Nested Structure With Thinking Off

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)

*CSV excluded — flat-only format, does not support nested structures.*

#### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 77s | JSON_COMPACT ≈ 10163 | YAML ≈ 213 | XML_COMPACT ≈ 4990 | XML_COMPACT ≈ 5295 | XML_COMPACT ≈ 18000 | XML_COMPACT ≈ 75.81% | JSON_COMPACT ≈ 82 | XML_COMPACT ≈ 84 | XML_COMPACT ≈ 84 | YAML ≈ 96.43% | JSON_COMPACT ≈ 96 | XML_COMPACT ≈ 97 | XML_COMPACT ≈ 97 |
| XML_COMPACT (+3.89%) | XML_COMPACT (+25.01%) | XML_PRETTY (+42.12%) | TOON_DEFAULT (+72.46%) | TOON_DEFAULT (+68.36%) | JSON_COMPACT (+11.63%) | YAML (-0.54%) | XML_COMPACT (-9.15%) | TOON_DEFAULT (-17.62%) | JSON_COMPACT (-7.48%) | JSON_COMPACT (-0.20%) | XML_COMPACT (-8.76%) | TOON_DEFAULT (-14.41%) | JSON_COMPACT (-5.51%) |
| XML_PRETTY (+6.07%) | TOON_DEFAULT (+38.42%) | XML_COMPACT (+43.42%) | JSON_PRETTY (+79.09%) | JSON_PRETTY (+74.56%) | TOON_DEFAULT (+27.68%) | JSON_COMPACT (-0.81%) | YAML (-15.18%) | JSON_PRETTY (-19.70%) | TOON_DEFAULT (-17.49%) | XML_COMPACT (-0.78%) | YAML (-12.99%) | JSON_PRETTY (-16.17%) | TOON_DEFAULT (-14.29%) |
| JSON_COMPACT (+6.56%) | YAML (+39.28%) | JSON_COMPACT (+43.51%) | JSON_COMPACT (+92.88%) | JSON_COMPACT (+87.53%) | YAML (+34.37%) | TOON_DEFAULT (-1.52%) | TOON_DEFAULT (-15.65%) | JSON_COMPACT (-21.66%) | YAML (-20.64%) | TOON_DEFAULT (-1.12%) | TOON_DEFAULT (-13.48%) | JSON_COMPACT (-17.75%) | YAML (-16.92%) |
| TOON_DEFAULT (+7.89%) | JSON_PRETTY (+73.98%) | JSON_PRETTY (+44.17%) | XML_PRETTY (+93.24%) | XML_PRETTY (+87.81%) | JSON_PRETTY (+49.58%) | JSON_PRETTY (-2.26%) | JSON_PRETTY (-30.19%) | YAML (-21.90%) | JSON_PRETTY (-30.96%) | JSON_PRETTY (-1.82%) | JSON_PRETTY (-25.85%) | YAML (-18.01%) | JSON_PRETTY (-25.90%) |
| YAML (+11.79%) | XML_PRETTY (+98.80%) | TOON_DEFAULT (+45.31%) | YAML (+96.77%) | YAML (+89.45%) | XML_PRETTY (+67.49%) | XML_PRETTY (-6.19%) | XML_PRETTY (-43.13%) | XML_PRETTY (-26.00%) | XML_PRETTY (-44.61%) | XML_PRETTY (-2.25%) | XML_PRETTY (-34.44%) | XML_PRETTY (-19.22%) | XML_PRETTY (-35.29%) |

#### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 67s | JSON_COMPACT ≈ 9645 | XML_PRETTY ≈ 205 | YAML ≈ 7786 | YAML ≈ 8089 | JSON_COMPACT ≈ 21434 | JSON_COMPACT ≈ 76.45% | JSON_COMPACT ≈ 84 | XML_PRETTY ≈ 68 | JSON_COMPACT ≈ 75 | XML_COMPACT ≈ 96.80% | JSON_COMPACT ≈ 97 | YAML ≈ 85 | JSON_COMPACT ≈ 88 |
| XML_PRETTY (+2.24%) | XML_COMPACT (+29.13%) | TOON_DEFAULT (+25.69%) | XML_PRETTY (+3.99%) | XML_PRETTY (+2.63%) | XML_COMPACT (+1.44%) | TOON_DEFAULT (-1.33%) | XML_COMPACT (-12.29%) | XML_COMPACT (-1.28%) | XML_COMPACT (-3.13%) | TOON_DEFAULT (-0.38%) | XML_COMPACT (-8.66%) | XML_PRETTY (-1.48%) | XML_COMPACT (-0.48%) |
| XML_COMPACT (+13.55%) | TOON_DEFAULT (+43.41%) | YAML (+47.64%) | XML_COMPACT (+15.34%) | XML_COMPACT (+14.82%) | YAML (+3.74%) | XML_COMPACT (-2.25%) | TOON_DEFAULT (-16.71%) | YAML (-2.74%) | YAML (-12.37%) | JSON_COMPACT (-0.63%) | TOON_DEFAULT (-13.37%) | XML_COMPACT (-2.40%) | YAML (-4.89%) |
| JSON_PRETTY (+39.93%) | YAML (+46.69%) | JSON_COMPACT (+48.68%) | JSON_PRETTY (+46.79%) | JSON_PRETTY (+45.07%) | TOON_DEFAULT (+20.11%) | JSON_PRETTY (-2.53%) | YAML (-25.22%) | JSON_COMPACT (-12.98%) | TOON_DEFAULT (-16.95%) | YAML (-3.79%) | YAML (-16.73%) | JSON_COMPACT (-14.12%) | TOON_DEFAULT (-13.22%) |
| JSON_COMPACT (+40.79%) | JSON_PRETTY (+73.74%) | JSON_PRETTY (+48.78%) | JSON_COMPACT (+47.50%) | JSON_COMPACT (+45.75%) | XML_PRETTY (+30.50%) | XML_PRETTY (-6.56%) | JSON_PRETTY (-28.59%) | TOON_DEFAULT (-14.95%) | JSON_PRETTY (-28.07%) | XML_PRETTY (-4.46%) | JSON_PRETTY (-25.96%) | TOON_DEFAULT (-14.47%) | XML_PRETTY (-23.25%) |
| TOON_DEFAULT (+40.96%) | XML_PRETTY (+103.95%) | XML_COMPACT (+49.76%) | TOON_DEFAULT (+49.68%) | TOON_DEFAULT (+47.27%) | JSON_PRETTY (+32.92%) | YAML (-10.59%) | XML_PRETTY (-42.68%) | JSON_PRETTY (-15.13%) | XML_PRETTY (-29.76%) | JSON_PRETTY (-4.95%) | XML_PRETTY (-35.05%) | JSON_PRETTY (-17.28%) | JSON_PRETTY (-25.24%) |


## 3. Conclusion & Decision Matrix

### 3.1 Cross-Configuration Findings

- **CSV** and **TOON_DEFAULT** dominate flat read tokens at approximately 7,000 to 7,500 tokens. **JSON_COMPACT** dominates nested read tokens at approximately 10,000 tokens. The most token-hungry format is **XML_PRETTY** which consumes roughly 2x the read tokens of the leader in every configuration.
- **JSON_PRETTY** consistently achieves the highest accuracy by answer in flat structures at approximately 81%. **YAML** leads in nested accuracy at 77% to 79% with thinking enabled. Neither format ranks well in efficiency because their token overhead outweighs the accuracy advantage in the composite score.
- All formats achieve 92% to 98% accuracy by charcter compared to 70% to 81% accuracy by answer. This gap indicates that the majority of incorrect answers differ from the expected answer by only a few characters rather than being entirely wrong. Aggregation and filtering questions are the primary source of complete answer failures because a single wrong digit or mising word invalidates the entire response.
- The ranking order of formats stays largely stable between thinking on and off. Thinking on occasionally boosts accuracy by 1 to 3 percentage points but the additional reasoning tokens sometimes offset that gain in the efficiency score. The practical impact of thinking mode is smaller than the impact of format choice.
- For flat mandatory (dense) data **TOON_DEFAULT** matches **CSV** at approximately 7,000 read tokens because its tabular mode places field names once in a header row. For optional (sparse) data **TOON_DEFAULT** jumps to approximately 11,500 read tokens as it falls back to explicit key-value pairs per record to represent missing fields without ambiguity. This adaptive behavior makes **TOON_DEFAULT** competitive with **CSV** on dense data and competitive with **YAML** on sparse data but it does not outperform the specialist in either variant or structure.
- **JSON_PRETTY** and **XML_PRETTY** consume 1.5x to 2x the read tokens of their compact counterparts. Indentation and newlines add tokens that carry zero informational value for the model and only serve human readability. **JSON_PRETTY** partially compensates through higher accuracy in flat structures but **XML_PRETTY** fails to compensate in any configuration.
- Moving from flat to nested structure increases read tokens by 30% to 60% for most formats and drops accuracy by 2 to 5 percentage points on average. Nested hierarchy adds structural tokens (braces, indentation, repeated parent keys) that dilute the data to token ratio.
- **CSV** achieves the best or second best efficiency in every flat configuration and its inability to represent nested structures seems to be its only weakness. When data is flat **CSV** is the strongest choice for dense fields.
- **XML_COMPACT** becomes a top 2 contender in nested structures while only middling flat performance. Its explicit open and close tags appear to help the model maintain context when navigating hierarchical data and the absence of whitespace keeps tokens low.
- **JSON_COMPACT** is the most versatile format overall because it ranks in the top 3 for efficiency across all 8 configurations. It never wins in flat mandatory compared to **CSV** and **TOON_DEFAULT** but it never catastrophically fails in any configuration either. For teams that need a single format across mixed workloads **JSON_COMPACT** offers the most consistent performance.

### 3.2 Decision Matrix

| Scenario | Recommended Format | Eff. Score Total | Accuracy | Rationale |
|---|---|---|---|---|
| Flat structure, dense mandatory fields | **CSV** | 84 to 85 | 76 to 77% | Lowest total tokens (~17k), top efficiency across thinking modes |
| Flat structure, sparse optional fields | **JSON_COMPACT** | ~80 | 76 to 78% | Best efficiency when optional fields create sparsity, **CSV** accuracy drops to ~73% on sparse data |
| Nested structure, dense mandatory fields | **JSON_COMPACT** | 78 to 82 | 74 to 75% | Fewest nested read tokens (~10k), top efficiency with thinking on |
| Nested structure, sparse optional fields | **JSON_COMPACT** | 75 to 84 | 76% | Dominant efficiency leader, omits missing keys entirely to keep tokens low |
| Maximum accuracy required, flat structure | **JSON_PRETTY** | 56 to 58 | ~81% | Highest flat accuracy at significant token cost (+64% total tokens over **CSV**) |
| Maximum accuracy required, nested structure | **YAML** | 59 to 67 | 75 to 79% | Best nested accuracy with thinking on, indentation aids hierarchical parsing |
| Token budget critical, flat structure | **CSV** | 84 to 85 | 76 to 77% | Fewest read tokens (~7k), fewest total tokens (~17k) |
| Token budget critical, nested structure | **JSON_COMPACT** | 78 to 82 | 74 to 76% | Fewest nested read tokens (~10k), fewest nested total tokens (~18 to 20k) |
| Avoid in flat structure | **XML_PRETTY** | 49 to 51 | 74 to 77% | Worst efficiency, 2x the read tokens of **CSV** for no accuracy advantage |
| Avoid in nested structure | **XML_PRETTY** | 46 to 47 | 68 to 70% | Consistently last in efficiency and among the lowest in accuracy across nested configurations |

### 3.3 Real-World Impact

- Switching from **XML_PRETTY** to **CSV** for flat data reduces read tokens from approximately 16,000 to 7,000 per file. That is a 56% reduction per read operation. For workflows that read data files thousands of times per day (automated agents, batch processing pipelines, RAG retrieval) this translates directly into lower API costs and faster response times. The same switch from **XML_PRETTY** to **JSON_COMPACT** in nested data saves approximately 50% of read tokens.
- The highest accuracy format (**JSON_PRETTY** at 81% flat) costs 64% more total tokens than the most efficient format (**CSV** at 84 to 85 efficiency score, 76 to 77% accuracy). The accuracy difference is roughly 4 to 5 percentage points for a 64% increase in token spend. Accuracy by character narrows this gap further because most **CSV** "errors" differ by only a few characters. Teams should decide whether those extra percentage points of complete answer accuracy justify the token budget increase for their specific use case.
- The efficiency gap between the best and worst format within a single configuration ranges from 35 to 45 points on the efficiency score. The gap between thinking on and off for the same format is typically 1 to 5 points. Optimizing format selection provides a larger return than toggling extended thinking for this model.
- **TOON_DEFAULT** occupies a practical middle ground for mixed density data. In workflows where some files have all fields populated and others have sparse optional fields **TOON_DEFAULT** avoids the need to maintain two format pipelines. Its adaptive encoding automatically shifts between **CSV**-like tabular layout for dense records and **YAML**-like key-value layout for sparse records. The tradeoff is that it never quite matches **CSV** efficiency on dense data or **JSON_COMPACT** efficiency on sparse or nested data. It was not tested but is worth mentioning that **TOON_DEFAULT** also supports a tabular layout for objects with nested array of objects where data is dense which outperforms **JSON_COMPACT** depending on the object shape. Choose **TOON_DEFAULT** if the data is dense and the data shape benfits from the format otherwise use **CSV** and **JSON_COMPACT**. 
- The accuracy drop from flat to nested structures (2 to 5 percentage points) combined with the token increase (30% to 60%) means that nested data files are inherently more expensive and less reliable to query. Where possible flattening nested structures before feeding them to the model will improve both accuracy and cost. When nesting is unavoidable **JSON_COMPACT** or **XML_COMPACT** should be preferred over other formats.
- **XML_PRETTY** should be avoided as a model input format. Across all 8 configurations **XML_PRETTY** ranks last or second to last in efficiency and frequently last in accuracy for nested structures. Its verbose tag structure and indentation consume tokens without aiding model comprehension. If **XML** is required for compatibility reasons **XML_COMPACT** provides meaningfully better results by eliminating whitespace overhead.

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure

- **Test Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Extended Thinking**: on & off
- **Structure**: flat & nested
- **Variant**: mandatory (22 fields, dense) and optional (19 mandatory + 3 optional, sparse)
- **Formats Tested**: CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML
- **Record Counts**: 31

### 4.2 Appendix B: Benchmark Configuration
- **Field Retrieval**: 55 questions (37.50% weight)
- **Filtering**: 21 questions (20.83% weight)
- **Aggregation**: 21 questions (12.50% weight)
- **Structure Awareness**: 27 questions (29.17% weight)

---

- **Report Generated**: 2026-04-14
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code 2.1.73
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Full Benchmark Reports**: 
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)