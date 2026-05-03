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
| **TOON_KEYFOLD** | [Token-Oriented Object Notation v2.1.0](https://toonformat.dev/) with key folding enabled |
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

These results are specific to Claude Code using the Claude Sonnet 4.6 model. They serve as a rule of thumb for choosing the best file format depending on the use case.
However these values cannot be exactly applied to models of the same family or from other providers as token usage, accuracy and latency depend on specific model architectures and tokenizers. Also file reads will produce different characters depending on the used harness because some add marker characters, line numbers or additional information. While the relative ranking of file formats remains consistent the absolute numbers will vary.
Especially the accuracy and output tokens results will vary because these values are bound to the model size and training, instruction interpretation and reasoning token budget.

## 2. Simplified Results

*Note: All columns ranked best-to-worst. ↑ = lower value is better (ascending). ↓ = higher value is better (descending).*

### 2.1 Flat Structure With Thinking On

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_sonnet_4_6/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)

#### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 279s | CSV ≈ 3922 | JSON_PRETTY ≈ 201 | JSON_PRETTY ≈ 20803 | JSON_PRETTY ≈ 21004 | TOON_DEFAULT ≈ 26023 | JSON_COMPACT ≈ 98.92% | TOON_DEFAULT ≈ 98 | JSON_PRETTY ≈ 91 | TOON_DEFAULT ≈ 96 | TOON_DEFAULT ≈ 99.98% | TOON_DEFAULT ≈ 99 | JSON_PRETTY ≈ 92 | TOON_DEFAULT ≈ 96 |
| JSON_COMPACT (+8.79%) | TOON_DEFAULT (+1.10%) | TOON_DEFAULT (+26.95%) | TOON_DEFAULT (+4.81%) | TOON_DEFAULT (+5.02%) | CSV (+9.26%) | TOON_DEFAULT (0.00%) | CSV (-0.77%) | TOON_DEFAULT (-3.88%) | CSV (-6.70%) | XML_PRETTY (-0.01%) | CSV (-0.06%) | TOON_DEFAULT (-4.21%) | CSV (-5.93%) |
| YAML (+10.70%) | JSON_COMPACT (+58.01%) | CSV (+41.96%) | YAML (+10.35%) | YAML (+10.68%) | JSON_COMPACT (+17.55%) | XML_COMPACT (0.00%) | JSON_COMPACT (-7.27%) | YAML (-9.68%) | JSON_COMPACT (-10.94%) | JSON_PRETTY (-0.02%) | JSON_COMPACT (-7.29%) | YAML (-8.99%) | JSON_COMPACT (-10.92%) |
| XML_PRETTY (+13.39%) | YAML (+141.69%) | YAML (+45.11%) | JSON_COMPACT (+15.83%) | JSON_COMPACT (+16.14%) | YAML (+25.76%) | XML_PRETTY (-0.26%) | YAML (-18.87%) | JSON_COMPACT (-13.35%) | YAML (-16.98%) | YAML (-0.03%) | YAML (-17.86%) | JSON_COMPACT (-13.64%) | YAML (-15.95%) |
| TOON_DEFAULT (+20.69%) | XML_COMPACT (+191.79%) | XML_COMPACT (+45.44%) | XML_PRETTY (+16.22%) | XML_PRETTY (+16.51%) | JSON_PRETTY (+34.64%) | JSON_PRETTY (-0.53%) | XML_COMPACT (-24.37%) | XML_PRETTY (-13.85%) | JSON_PRETTY (-21.95%) | JSON_COMPACT (-0.10%) | XML_COMPACT (-24.37%) | XML_PRETTY (-13.88%) | JSON_PRETTY (-21.43%) |
| CSV (+21.24%) | XML_PRETTY (+233.68%) | XML_PRETTY (+46.77%) | CSV (+16.45%) | CSV (+16.69%) | XML_PRETTY (+44.33%) | CSV (-1.34%) | XML_PRETTY (-29.89%) | CSV (-14.80%) | XML_PRETTY (-27.79%) | XML_COMPACT (-0.27%) | XML_PRETTY (-29.51%) | CSV (-14.25%) | XML_PRETTY (-27.42%) |
| XML_COMPACT (+21.68%) | JSON_PRETTY (+257.83%) | JSON_COMPACT (+48.59%) | XML_COMPACT (+26.13%) | XML_COMPACT (+26.32%) | XML_COMPACT (+45.93%) | YAML (-1.34%) | JSON_PRETTY (-33.16%) | XML_COMPACT (-22.00%) | XML_COMPACT (-28.61%) | CSV (-0.30%) | JSON_PRETTY (-32.58%) | XML_COMPACT (-22.32%) | XML_COMPACT (-28.59%) |

#### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XML_PRETTY ≈ 265s | CSV ≈ 3633 | CSV ≈ 287 | JSON_COMPACT ≈ 18494 | JSON_COMPACT ≈ 18788 | JSON_COMPACT ≈ 24457 | JSON_PRETTY ≈ 98.12% | CSV ≈ 98 | JSON_COMPACT ≈ 98 | JSON_COMPACT ≈ 98 | JSON_COMPACT ≈ 99.88% | CSV ≈ 100 | JSON_COMPACT ≈ 100 | JSON_COMPACT ≈ 100 |
| TOON_DEFAULT (+15.66%) | JSON_COMPACT (+56.04%) | YAML (+1.74%) | XML_PRETTY (+12.59%) | XML_PRETTY (+12.40%) | CSV (+28.66%) | XML_COMPACT (-0.27%) | JSON_COMPACT (-6.45%) | XML_PRETTY (-8.71%) | CSV (-16.51%) | JSON_PRETTY (0.00%) | JSON_COMPACT (-6.51%) | XML_PRETTY (-8.58%) | CSV (-16.08%) |
| XML_COMPACT (+23.75%) | XML_COMPACT (+116.10%) | JSON_PRETTY (+2.32%) | TOON_DEFAULT (+27.68%) | TOON_DEFAULT (+27.29%) | XML_PRETTY (+35.49%) | JSON_COMPACT (-0.54%) | XML_COMPACT (-13.38%) | TOON_DEFAULT (-19.26%) | XML_PRETTY (-20.21%) | CSV (-0.01%) | XML_COMPACT (-13.57%) | TOON_DEFAULT (-18.88%) | XML_PRETTY (-19.90%) |
| JSON_PRETTY (+27.32%) | TOON_DEFAULT (+211.59%) | JSON_COMPACT (+2.56%) | JSON_PRETTY (+38.32%) | JSON_PRETTY (+37.72%) | XML_COMPACT (+38.17%) | XML_PRETTY (-0.54%) | TOON_DEFAULT (-24.95%) | JSON_PRETTY (-26.12%) | XML_COMPACT (-21.55%) | TOON_DEFAULT (-0.01%) | TOON_DEFAULT (-24.62%) | JSON_PRETTY (-26.08%) | XML_COMPACT (-21.48%) |
| YAML (+28.11%) | YAML (+217.15%) | XML_COMPACT (+2.79%) | XML_COMPACT (+38.68%) | XML_COMPACT (+38.08%) | TOON_DEFAULT (+44.07%) | TOON_DEFAULT (-0.67%) | YAML (-25.70%) | XML_COMPACT (-26.56%) | TOON_DEFAULT (-25.18%) | XML_PRETTY (-0.01%) | YAML (-25.34%) | XML_COMPACT (-26.41%) | TOON_DEFAULT (-24.72%) |
| JSON_COMPACT (+34.05%) | XML_PRETTY (+230.80%) | XML_PRETTY (+3.02%) | YAML (+39.18%) | YAML (+38.55%) | YAML (+53.55%) | CSV (-0.81%) | XML_PRETTY (-27.14%) | YAML (-27.26%) | YAML (-30.67%) | XML_COMPACT (-0.11%) | XML_PRETTY (-26.85%) | YAML (-26.75%) | YAML (-30.11%) |
| CSV (+42.39%) | JSON_PRETTY (+261.08%) | TOON_DEFAULT (+5.69%) | CSV (+48.96%) | CSV (+48.15%) | JSON_PRETTY (+59.43%) | YAML (-0.81%) | JSON_PRETTY (-30.35%) | CSV (-34.00%) | JSON_PRETTY (-33.47%) | YAML (-0.13%) | JSON_PRETTY (-30.37%) | CSV (-33.31%) | JSON_PRETTY (-33.32%) |


### 2.2 Flat Structure With Thinking Off

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_sonnet_4_6/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)

#### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 273s | CSV ≈ 6748 | YAML ≈ 198 | JSON_PRETTY ≈ 20394 | JSON_PRETTY ≈ 20597 | TOON_DEFAULT ≈ 31606 | JSON_PRETTY ≈ 99.46% | CSV ≈ 94 | JSON_PRETTY ≈ 100 | TOON_DEFAULT ≈ 87 | JSON_PRETTY ≈ 99.98% | CSV ≈ 95 | JSON_PRETTY ≈ 100 | TOON_DEFAULT ≈ 88 |
| JSON_COMPACT (+6.31%) | TOON_DEFAULT (+0.74%) | JSON_PRETTY (+2.53%) | JSON_COMPACT (+11.43%) | JSON_COMPACT (+11.79%) | JSON_PRETTY (+0.62%) | XML_COMPACT (-0.27%) | TOON_DEFAULT (-0.24%) | JSON_COMPACT (-11.52%) | JSON_PRETTY (-0.39%) | TOON_DEFAULT (-0.01%) | TOON_DEFAULT (-0.17%) | XML_PRETTY (-11.07%) | JSON_PRETTY (-0.79%) |
| XML_PRETTY (+10.35%) | JSON_COMPACT (+33.77%) | CSV (+46.54%) | XML_PRETTY (+11.53%) | XML_PRETTY (+11.87%) | JSON_COMPACT (+1.42%) | CSV (-0.53%) | JSON_COMPACT (-11.01%) | XML_PRETTY (-12.67%) | JSON_COMPACT (-2.04%) | XML_COMPACT (-0.01%) | JSON_COMPACT (-10.88%) | JSON_COMPACT (-11.14%) | JSON_COMPACT (-2.04%) |
| YAML (+16.56%) | YAML (+40.47%) | XML_PRETTY (+49.58%) | YAML (+15.60%) | YAML (+15.42%) | YAML (+5.21%) | TOON_DEFAULT (-0.54%) | YAML (-13.15%) | YAML (-14.90%) | YAML (-6.98%) | XML_PRETTY (-0.08%) | YAML (-13.00%) | YAML (-14.50%) | YAML (-6.93%) |
| XML_COMPACT (+21.98%) | JSON_PRETTY (+66.03%) | XML_COMPACT (+50.76%) | TOON_DEFAULT (+20.16%) | TOON_DEFAULT (+20.44%) | CSV (+9.87%) | JSON_COMPACT (-0.80%) | JSON_PRETTY (-20.78%) | TOON_DEFAULT (-19.40%) | CSV (-12.84%) | CSV (-0.10%) | JSON_PRETTY (-20.94%) | TOON_DEFAULT (-18.98%) | CSV (-12.81%) |
| TOON_DEFAULT (+22.54%) | XML_COMPACT (+69.59%) | JSON_COMPACT (+51.77%) | XML_COMPACT (+26.41%) | XML_COMPACT (+26.62%) | XML_PRETTY (+14.31%) | YAML (-0.80%) | XML_COMPACT (-22.11%) | XML_COMPACT (-24.97%) | XML_PRETTY (-20.06%) | YAML (-0.28%) | XML_COMPACT (-22.08%) | XML_COMPACT (-24.71%) | XML_PRETTY (-18.53%) |
| CSV (+35.33%) | XML_PRETTY (+93.94%) | TOON_DEFAULT (+52.44%) | CSV (+35.76%) | CSV (+35.84%) | XML_COMPACT (+18.72%) | XML_PRETTY (-2.42%) | XML_PRETTY (-31.43%) | CSV (-33.73%) | XML_COMPACT (-24.16%) | JSON_COMPACT (-0.29%) | XML_PRETTY (-29.88%) | CSV (-33.33%) | XML_COMPACT (-24.17%) |

#### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XML_PRETTY ≈ 281s | JSON_COMPACT ≈ 5669 | YAML ≈ 198 | XML_PRETTY ≈ 21922 | XML_PRETTY ≈ 22126 | JSON_COMPACT ≈ 28263 | CSV ≈ 98.65% | JSON_COMPACT ≈ 99 | XML_PRETTY ≈ 92 | JSON_COMPACT ≈ 99 | CSV ≈ 99.95% | JSON_COMPACT ≈ 100 | XML_PRETTY ≈ 93 | JSON_COMPACT ≈ 100 |
| JSON_PRETTY (+1.52%) | CSV (+14.48%) | XML_PRETTY (+3.20%) | JSON_PRETTY (+0.29%) | JSON_PRETTY (+0.72%) | JSON_PRETTY (+15.25%) | JSON_COMPACT (-0.53%) | CSV (-3.37%) | JSON_PRETTY (-0.98%) | JSON_PRETTY (-15.87%) | JSON_COMPACT (-0.06%) | CSV (-3.64%) | JSON_PRETTY (-0.78%) | JSON_PRETTY (-15.51%) |
| JSON_COMPACT (+3.86%) | XML_COMPACT (+38.49%) | CSV (+43.84%) | JSON_COMPACT (+1.70%) | JSON_COMPACT (+2.12%) | CSV (+16.71%) | XML_PRETTY (-0.53%) | XML_COMPACT (-11.18%) | JSON_COMPACT (-2.30%) | CSV (-16.83%) | XML_PRETTY (-0.07%) | XML_COMPACT (-9.87%) | JSON_COMPACT (-2.26%) | CSV (-16.94%) |
| TOON_DEFAULT (+10.97%) | TOON_DEFAULT (+49.71%) | XML_COMPACT (+49.75%) | YAML (+12.38%) | YAML (+12.24%) | YAML (+18.64%) | JSON_PRETTY (-0.80%) | TOON_DEFAULT (-12.98%) | YAML (-14.07%) | YAML (-19.90%) | JSON_PRETTY (-0.08%) | TOON_DEFAULT (-12.72%) | YAML (-13.21%) | YAML (-19.04%) |
| YAML (+16.34%) | YAML (+53.40%) | JSON_PRETTY (+51.43%) | TOON_DEFAULT (+14.67%) | TOON_DEFAULT (+14.97%) | TOON_DEFAULT (+20.04%) | TOON_DEFAULT (-0.80%) | YAML (-14.47%) | TOON_DEFAULT (-16.45%) | TOON_DEFAULT (-20.79%) | TOON_DEFAULT (-0.18%) | YAML (-13.68%) | TOON_DEFAULT (-16.13%) | TOON_DEFAULT (-20.44%) |
| CSV (+18.51%) | JSON_PRETTY (+81.48%) | JSON_COMPACT (+51.60%) | CSV (+19.56%) | CSV (+19.75%) | XML_PRETTY (+20.81%) | YAML (-1.61%) | JSON_PRETTY (-21.15%) | CSV (-21.05%) | XML_PRETTY (-21.40%) | XML_COMPACT (-0.18%) | JSON_PRETTY (-20.74%) | CSV (-21.11%) | XML_PRETTY (-21.15%) |
| XML_COMPACT (+18.89%) | XML_PRETTY (+112.00%) | TOON_DEFAULT (+52.11%) | XML_COMPACT (+20.18%) | XML_COMPACT (+20.41%) | XML_COMPACT (+22.04%) | XML_COMPACT (-2.41%) | XML_PRETTY (-28.83%) | XML_COMPACT (-23.52%) | XML_COMPACT (-23.94%) | YAML (-0.20%) | XML_PRETTY (-28.49%) | XML_COMPACT (-21.95%) | XML_COMPACT (-22.48%) |


### 2.3 Nested Structure With Thinking On

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_sonnet_4_6/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)

*CSV excluded — flat-only format, does not support nested structures.*

#### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 263s | JSON_COMPACT ≈ 7498 | XML_COMPACT ≈ 198 | TOON_DEFAULT ≈ 20713 | TOON_DEFAULT ≈ 21014 | TOON_DEFAULT ≈ 32454 | TOON_DEFAULT ≈ 99.73% | JSON_COMPACT ≈ 98 | TOON_DEFAULT ≈ 100 | TOON_DEFAULT ≈ 95 | TOON_DEFAULT ≈ 100.00% | JSON_COMPACT ≈ 98 | TOON_DEFAULT ≈ 100 | TOON_DEFAULT ≈ 95 |
| TOON_KEYFOLD (+7.71%) | XML_COMPACT (+35.56%) | YAML (0.00%) | TOON_KEYFOLD (+4.26%) | TOON_KEYFOLD (+3.75%) | XML_COMPACT (+1.25%) | YAML (0.00%) | XML_COMPACT (-8.40%) | TOON_KEYFOLD (-5.98%) | XML_COMPACT (-2.18%) | JSON_COMPACT (-0.04%) | XML_COMPACT (-8.15%) | TOON_KEYFOLD (-5.30%) | XML_COMPACT (-1.28%) |
| JSON_PRETTY (+12.42%) | TOON_KEYFOLD (+50.56%) | JSON_COMPACT (+2.86%) | XML_COMPACT (+8.61%) | XML_COMPACT (+8.00%) | TOON_KEYFOLD (+1.96%) | JSON_COMPACT (-1.08%) | TOON_DEFAULT (-11.14%) | XML_COMPACT (-11.91%) | TOON_KEYFOLD (-2.60%) | YAML (-0.04%) | TOON_KEYFOLD (-11.53%) | XML_COMPACT (-11.03%) | TOON_KEYFOLD (-1.89%) |
| XML_COMPACT (+14.36%) | TOON_DEFAULT (+52.57%) | TOON_KEYFOLD (+4.03%) | YAML (+10.77%) | YAML (+10.13%) | JSON_COMPACT (+2.92%) | TOON_KEYFOLD (-1.34%) | TOON_KEYFOLD (-11.60%) | YAML (-13.71%) | JSON_COMPACT (-3.22%) | XML_PRETTY (-0.31%) | TOON_DEFAULT (-11.75%) | YAML (-13.71%) | JSON_COMPACT (-2.48%) |
| YAML (+19.80%) | YAML (+93.05%) | XML_PRETTY (+46.56%) | JSON_PRETTY (+11.25%) | JSON_PRETTY (+11.07%) | YAML (+15.91%) | XML_COMPACT (-1.61%) | YAML (-20.29%) | JSON_PRETTY (-16.42%) | YAML (-13.39%) | XML_COMPACT (-0.32%) | YAML (-20.84%) | JSON_PRETTY (-15.46%) | YAML (-13.39%) |
| XML_PRETTY (+22.96%) | JSON_PRETTY (+110.55%) | JSON_PRETTY (+49.41%) | XML_PRETTY (+22.62%) | XML_PRETTY (+22.25%) | JSON_PRETTY (+20.56%) | JSON_PRETTY (-2.15%) | JSON_PRETTY (-25.71%) | XML_PRETTY (-31.56%) | JSON_PRETTY (-18.81%) | TOON_KEYFOLD (-0.34%) | JSON_PRETTY (-25.24%) | XML_PRETTY (-30.27%) | JSON_PRETTY (-17.80%) |
| JSON_COMPACT (+23.07%) | XML_PRETTY (+143.95%) | TOON_DEFAULT (+51.60%) | JSON_COMPACT (+24.08%) | JSON_COMPACT (+23.27%) | XML_PRETTY (+35.52%) | XML_PRETTY (-2.15%) | XML_PRETTY (-33.26%) | JSON_COMPACT (-32.22%) | XML_PRETTY (-31.40%) | JSON_PRETTY (-0.75%) | XML_PRETTY (-32.42%) | JSON_COMPACT (-31.47%) | XML_PRETTY (-30.05%) |

#### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 279s | JSON_COMPACT ≈ 6970 | JSON_PRETTY ≈ 201 | TOON_DEFAULT ≈ 21613 | TOON_DEFAULT ≈ 21911 | JSON_COMPACT ≈ 30448 | JSON_COMPACT ≈ 97.85% | JSON_COMPACT ≈ 99 | TOON_DEFAULT ≈ 92 | JSON_COMPACT ≈ 99 | JSON_PRETTY ≈ 99.88% | JSON_COMPACT ≈ 100 | TOON_DEFAULT ≈ 94 | JSON_COMPACT ≈ 100 |
| JSON_COMPACT (+9.79%) | XML_COMPACT (+38.94%) | XML_PRETTY (+45.18%) | YAML (+5.20%) | YAML (+5.12%) | TOON_DEFAULT (+8.73%) | XML_PRETTY (0.00%) | XML_COMPACT (-8.28%) | YAML (-8.02%) | TOON_DEFAULT (-7.36%) | JSON_COMPACT (-0.01%) | XML_COMPACT (-8.03%) | YAML (-7.82%) | TOON_DEFAULT (-6.59%) |
| TOON_KEYFOLD (+10.93%) | TOON_KEYFOLD (+58.45%) | YAML (+47.34%) | JSON_COMPACT (+7.26%) | JSON_COMPACT (+7.15%) | YAML (+13.10%) | JSON_PRETTY (-0.27%) | TOON_KEYFOLD (-13.07%) | JSON_COMPACT (-10.16%) | YAML (-10.86%) | XML_COMPACT (-0.07%) | TOON_KEYFOLD (-12.16%) | JSON_COMPACT (-10.66%) | YAML (-10.02%) |
| JSON_PRETTY (+11.73%) | TOON_DEFAULT (+60.62%) | JSON_COMPACT (+47.51%) | JSON_PRETTY (+8.61%) | JSON_PRETTY (+8.05%) | TOON_KEYFOLD (+16.35%) | XML_COMPACT (-0.27%) | TOON_DEFAULT (-13.33%) | JSON_PRETTY (-11.73%) | TOON_KEYFOLD (-13.34%) | XML_PRETTY (-0.07%) | TOON_DEFAULT (-12.48%) | JSON_PRETTY (-11.99%) | TOON_KEYFOLD (-12.43%) |
| YAML (+12.26%) | YAML (+63.62%) | XML_COMPACT (+48.01%) | TOON_KEYFOLD (+11.40%) | TOON_KEYFOLD (+11.28%) | XML_COMPACT (+17.79%) | TOON_DEFAULT (-1.08%) | YAML (-14.13%) | TOON_KEYFOLD (-17.45%) | XML_COMPACT (-13.70%) | TOON_DEFAULT (-0.08%) | YAML (-13.24%) | TOON_KEYFOLD (-17.02%) | XML_COMPACT (-13.38%) |
| XML_PRETTY (+15.99%) | JSON_PRETTY (+113.17%) | TOON_DEFAULT (+48.34%) | XML_PRETTY (+14.29%) | XML_PRETTY (+14.06%) | JSON_PRETTY (+26.55%) | YAML (-1.34%) | JSON_PRETTY (-23.71%) | XML_PRETTY (-20.74%) | JSON_PRETTY (-20.36%) | TOON_KEYFOLD (-0.27%) | JSON_PRETTY (-23.20%) | XML_PRETTY (-21.05%) | JSON_PRETTY (-19.90%) |
| XML_COMPACT (+17.98%) | XML_PRETTY (+154.81%) | TOON_KEYFOLD (+51.49%) | XML_COMPACT (+19.76%) | XML_COMPACT (+19.49%) | XML_PRETTY (+40.41%) | TOON_KEYFOLD (-1.35%) | XML_PRETTY (-32.18%) | XML_COMPACT (-29.24%) | XML_PRETTY (-30.71%) | YAML (-0.30%) | XML_PRETTY (-31.79%) | XML_COMPACT (-29.17%) | XML_PRETTY (-30.34%) |


### 2.4 Nested Structure With Thinking Off

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_sonnet_4_6/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)

*CSV excluded — flat-only format, does not support nested structures.*

#### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 292s | JSON_COMPACT ≈ 7498 | TOON_DEFAULT ≈ 205 | JSON_PRETTY ≈ 21490 | JSON_PRETTY ≈ 21787 | JSON_COMPACT ≈ 34929 | TOON_DEFAULT ≈ 99.19% | JSON_COMPACT ≈ 98 | JSON_PRETTY ≈ 97 | JSON_COMPACT ≈ 91 | TOON_DEFAULT ≈ 99.99% | JSON_COMPACT ≈ 98 | JSON_PRETTY ≈ 99 | JSON_COMPACT ≈ 92 |
| XML_PRETTY (+5.77%) | XML_COMPACT (+35.56%) | JSON_COMPACT (+43.58%) | YAML (+7.87%) | YAML (+7.76%) | YAML (+0.57%) | XML_COMPACT (0.00%) | XML_COMPACT (-7.67%) | YAML (-7.87%) | YAML (-0.43%) | XML_COMPACT (-0.01%) | XML_COMPACT (-7.76%) | YAML (-9.09%) | YAML (-0.41%) |
| YAML (+8.12%) | TOON_KEYFOLD (+50.56%) | YAML (+43.90%) | XML_PRETTY (+10.20%) | XML_PRETTY (+10.08%) | TOON_DEFAULT (+4.73%) | YAML (-0.27%) | TOON_DEFAULT (-11.51%) | XML_PRETTY (-11.49%) | TOON_DEFAULT (-4.84%) | XML_PRETTY (-0.03%) | TOON_DEFAULT (-11.58%) | XML_PRETTY (-12.10%) | TOON_DEFAULT (-4.95%) |
| TOON_DEFAULT (+12.41%) | TOON_DEFAULT (+52.57%) | JSON_PRETTY (+44.72%) | TOON_DEFAULT (+16.03%) | TOON_DEFAULT (+15.39%) | XML_COMPACT (+5.35%) | JSON_COMPACT (-0.54%) | TOON_KEYFOLD (-11.97%) | TOON_DEFAULT (-17.78%) | XML_COMPACT (-5.53%) | YAML (-0.03%) | TOON_KEYFOLD (-11.75%) | TOON_DEFAULT (-18.96%) | XML_COMPACT (-5.64%) |
| XML_COMPACT (+15.53%) | YAML (+55.36%) | XML_PRETTY (+46.50%) | TOON_KEYFOLD (+20.83%) | TOON_KEYFOLD (+20.59%) | TOON_KEYFOLD (+7.54%) | XML_PRETTY (-1.07%) | YAML (-12.33%) | TOON_KEYFOLD (-25.59%) | TOON_KEYFOLD (-8.93%) | JSON_COMPACT (-0.33%) | YAML (-12.22%) | TOON_KEYFOLD (-26.33%) | TOON_KEYFOLD (-8.72%) |
| TOON_KEYFOLD (+16.29%) | JSON_PRETTY (+110.55%) | XML_COMPACT (+47.15%) | XML_COMPACT (+22.53%) | XML_COMPACT (+22.24%) | JSON_PRETTY (+7.57%) | TOON_KEYFOLD (-1.34%) | JSON_PRETTY (-27.18%) | XML_COMPACT (-26.85%) | JSON_PRETTY (-10.73%) | TOON_KEYFOLD (-0.93%) | JSON_PRETTY (-25.58%) | XML_COMPACT (-27.84%) | JSON_PRETTY (-9.14%) |
| JSON_COMPACT (+24.40%) | XML_PRETTY (+143.95%) | TOON_KEYFOLD (+50.08%) | JSON_COMPACT (+26.27%) | JSON_COMPACT (+25.90%) | XML_PRETTY (+21.03%) | JSON_PRETTY (-3.76%) | XML_PRETTY (-32.89%) | JSON_COMPACT (-32.06%) | XML_PRETTY (-23.66%) | JSON_PRETTY (-1.46%) | XML_PRETTY (-32.10%) | JSON_COMPACT (-32.80%) | XML_PRETTY (-22.89%) |

#### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 323s | JSON_COMPACT ≈ 6970 | XML_COMPACT ≈ 204 | JSON_PRETTY ≈ 24887 | JSON_PRETTY ≈ 25184 | JSON_COMPACT ≈ 32254 | JSON_PRETTY ≈ 98.66% | JSON_COMPACT ≈ 99 | JSON_PRETTY ≈ 79 | JSON_COMPACT ≈ 99 | JSON_PRETTY ≈ 99.92% | JSON_COMPACT ≈ 100 | JSON_PRETTY ≈ 80 | JSON_COMPACT ≈ 100 |
| JSON_PRETTY (+0.22%) | TOON_KEYFOLD (+58.45%) | TOON_KEYFOLD (+2.95%) | YAML (+0.25%) | YAML (+0.23%) | YAML (+13.62%) | JSON_COMPACT (-0.81%) | TOON_DEFAULT (-13.33%) | JSON_COMPACT (-1.42%) | YAML (-13.63%) | JSON_COMPACT (-0.04%) | TOON_KEYFOLD (-12.19%) | YAML (-0.51%) | YAML (-12.76%) |
| TOON_KEYFOLD (+0.33%) | TOON_DEFAULT (+60.62%) | YAML (+43.86%) | JSON_COMPACT (+0.41%) | JSON_COMPACT (+0.40%) | TOON_KEYFOLD (+13.85%) | XML_COMPACT (-0.81%) | TOON_KEYFOLD (-13.61%) | YAML (-2.03%) | TOON_KEYFOLD (-14.57%) | XML_PRETTY (-0.05%) | TOON_DEFAULT (-12.47%) | JSON_COMPACT (-0.77%) | TOON_KEYFOLD (-13.15%) |
| TOON_DEFAULT (+1.92%) | YAML (+63.62%) | JSON_COMPACT (+45.17%) | XML_COMPACT (+1.86%) | XML_COMPACT (+1.47%) | TOON_DEFAULT (+15.31%) | XML_PRETTY (-1.88%) | YAML (-13.96%) | XML_COMPACT (-3.44%) | TOON_DEFAULT (-15.23%) | TOON_DEFAULT (-0.09%) | YAML (-13.08%) | XML_COMPACT (-2.80%) | TOON_DEFAULT (-14.34%) |
| JSON_COMPACT (+2.27%) | XML_COMPACT (+79.25%) | JSON_PRETTY (+45.83%) | TOON_KEYFOLD (+2.33%) | TOON_KEYFOLD (+1.95%) | XML_COMPACT (+17.97%) | TOON_DEFAULT (-1.89%) | XML_COMPACT (-16.48%) | TOON_KEYFOLD (-6.16%) | XML_COMPACT (-17.02%) | XML_COMPACT (-0.09%) | XML_COMPACT (-16.29%) | TOON_KEYFOLD (-3.92%) | XML_COMPACT (-16.82%) |
| XML_COMPACT (+2.32%) | JSON_PRETTY (+153.77%) | TOON_DEFAULT (+47.46%) | XML_PRETTY (+3.23%) | XML_PRETTY (+3.21%) | JSON_PRETTY (+32.92%) | YAML (-1.89%) | JSON_PRETTY (-31.42%) | XML_PRETTY (-7.61%) | JSON_PRETTY (-30.63%) | YAML (-0.09%) | JSON_PRETTY (-31.51%) | XML_PRETTY (-6.00%) | JSON_PRETTY (-30.73%) |
| XML_PRETTY (+4.62%) | XML_PRETTY (+154.81%) | XML_PRETTY (+47.79%) | TOON_DEFAULT (+3.25%) | TOON_DEFAULT (+3.23%) | XML_PRETTY (+35.65%) | TOON_KEYFOLD (-2.96%) | XML_PRETTY (-32.91%) | TOON_DEFAULT (-7.64%) | XML_PRETTY (-34.49%) | TOON_KEYFOLD (-0.35%) | XML_PRETTY (-31.76%) | TOON_DEFAULT (-6.06%) | XML_PRETTY (-33.32%) |


## 3. Conclusion & Decision Matrix

### 3.1 Cross-Configuration Findings

1. **JSON_COMPACT** is the most robust format across all eight runs. It ranks first on total efficiency score in five of eight configurations (all four optional variants plus nested mandatory with thinking off) and never drops below third place. Its read tokens stay inside a narrow band of 5669 to 9027 and accuracy stays between 97.58% and 98.66%. **JSON_COMPACT** is the safest default when the data shape is unknown in advance.
2. **TOON_DEFAULT** dominates dense tabular data but collapses on sparse data. On flat mandatory data **TOON** consumes 3965 read tokens which is close to **CSV** (3922) while reaching 98.92% accuracy. But for sparse data **TOON**'s adaptive encoding shifts from a **CSV** like tabular layout to a **YAML** like key value layout. Read tokens jump from 3965 to 11320 on flat data which is an increase of +185% and the total efficiency score falls from 95.67 to 73.59. The format is therefore only attractive when every record shares the exact same filled fields.
3. **TOON_KEYFOLD** offers no measurable benefit on the tested schema. Compared against **TOON_DEFAULT** on nested data the key folding saves only 151 read tokens (11289 vs 11440) while accuracy drops from 99.73% to 98.39% on the mandatory variant. The schema under test contains few single value wrapper objects so key folding has little to compress. Key folding should only be enabled on schemas dominated by nested wrappers around a single field but accuracy and output token usage should be monitored because the drop in accuracy might be due to the pattern break between **YAML** like key value layout and the collapsed single field objects which could be harder to parse for the model.
4. Thinking increases accuracy by roughly 0.3 to 2 percentage points on most formats while adding 10% to 30% more total tokens. For flat mandatory data **JSON_PRETTY** with thinking off reaches the highest measured accuracy of the entire benchmark (99.46%) while **TOON_DEFAULT** with thinking on reaches 98.92%. For nested mandatory data the picture reverses because **TOON_DEFAULT** with thinking on hits 99.73% accuracy which is the overall benchmark ceiling. All formats in both structures and thinking modes scored above 96% on accuracy by answer and above 98% on accuracy by character which makes token cost the main concern when picking a data format for LLM consumption.
5. **XML_PRETTY** is the weakest format for nested data in every run because its total efficiency score ranges from 64.55 to 69.72 across the four nested configurations. Read tokens sit around 17760 to 18291 which is roughly 2.4x the read tokens **JSON_COMPACT** used. Accuracy is on par with the other formats so the additional tokens are pure overhead. **XML_COMPACT** shows the same pattern on flat data where it ranks last twice.
6. Accuracy by character remains above 98% in every format and configuration. The lowest value recorded is 98.53% (**JSON_PRETTY** nested mandatory with thinking off) and most formats exceed 99.7%. This confirms that wrong answers are never fully wrong but just partial character drifts or missing words. The format choice mainly defines how many tokens are required and not how accurate the result will be.
7. For nested mandatory data read tokens span 7498 to 18291 which is a 2.4x spread while output tokens span 21014 to 27431 which is only a 1.3x spread. Output token differences between formats are small because the written answer file is the same regardless of the input format. The lever that matters is input compactness.
8. Pretty formats cost 2x to 3x more read tokens without accuracy gains. **JSON_PRETTY** uses between 2.3x and 3.6x the read tokens of **JSON_COMPACT** while accuracy differs by less than one percentage point in either direction. **XML_PRETTY** vs **XML_COMPACT** shows the same pattern. Pretty formatting is a human concern and adds no value for model consumption.
9. **CSV** produces the lowest read token counts on flat mandatory data (3922 on, 6748 off) but its accuracy trails the best format by 0.3 to 1.3 percentage points. **CSV** also cannot represent nested structures at all so its applicability is limited to flat tables with uniform column presence.

### 3.2 Decision Matrix

| Scenario | Recommended Format | Efficiency Score Total | Accuracy | Rationale |
|---|---|---|---|---|
| Flat structure, dense mandatory fields | **TOON_DEFAULT** (thinking on) | 95.67 | 98.92% | Matches **CSV** on read tokens at 3965 while producing the lowest total tokens of the run at 26023 |
| Flat structure, sparse optional fields | **JSON_COMPACT** (thinking on) | 98.36 | 97.58% | **TOON** loses its tabular shape and balloons to 11320 read tokens while **JSON_COMPACT** stays at 5669 |
| Nested structure, dense mandatory fields | **TOON_DEFAULT** (thinking on) | 94.86 | 99.73% | Highest benchmark accuracy combined with the lowest nested total tokens at 32454. **JSON_COMPACT** trailing close behing it. |
| Nested structure, sparse optional fields | **JSON_COMPACT** (thinking on) | 98.54 | 97.85% | Only format that stays compact and accurate under both sparsity and nesting |
| Maximum accuracy required, flat structure | **JSON_PRETTY** (thinking off) | 86.89 | 99.46% | Highest measured flat accuracy of the benchmark at the cost of 31801 total tokens |
| Maximum accuracy required, nested structure | **TOON_DEFAULT** (thinking on) | 94.86 | 99.73% | Overall benchmark ceiling for accuracy paired with competitive token usage |
| Token budget critical, flat structure | **CSV** (thinking on) | 89.26 | 97.58% | Lowest read tokens of any format at 3922 with only a small accuracy gap to the leaders |
| Token budget critical, nested structure | **JSON_COMPACT** (thinking on) | 98.54 | 97.85% | Lowest nested read tokens at 6970 without sacrificing robustness |
| Avoid in flat structure | **XML_COMPACT** and **XML_PRETTY** | 66.16 to 78.49 | 97.04% to 99.19% | Read tokens 2x to 3x the compact alternatives with no matching accuracy benefit |
| Avoid in nested structure | **XML_PRETTY** | 64.55 to 69.72 | 96.78% to 98.12% | Consistently last place across all four nested runs with 17760 to 18291 read tokens |

### 3.3 Real-World Impact

Projecting the per run total token counts to a workload of 1000 queries on 31 record datasets yields the following savings:
- Flat mandatory, thinking on: Switching from **XML_COMPACT** (37975 total tokens) to **TOON_DEFAULT** (26023) saves roughly 12 million tokens per 1000 queries (31.5% reduction).
- Flat optional, thinking on: Switching from **JSON_PRETTY** (38993) to **JSON_COMPACT** (24457) saves roughly 14.5 million tokens per 1000 queries (37.3% reduction).
- Nested mandatory, thinking on: Switching from **XML_PRETTY** (43981) to **TOON_DEFAULT** (32454) saves roughly 11.5 million tokens per 1000 queries (26.2% reduction).
- Nested optional, thinking on: Switching from **XML_PRETTY** (42752) to **JSON_COMPACT** (30448) saves roughly 12.3 million tokens per 1000 queries (28.8% reduction).

The benchmark results highlight three simple rules for production use:
1. If the data schema is dense and every record fills every field use **TOON_DEFAULT**. Expected savings are 10% to 30% total tokens versus **JSON_COMPACT** with similar accuracy.
2. If any field can be absent in any record or the data schema is unknown use **JSON_COMPACT**. It is a safe default choice and the only format that stays compact under sparsity because **TOON**'s adaptive encoding expands into key value form whenever optional fields appear.
3. Pretty formats (**JSON_PRETTY**, **XML_PRETTY**) should only be stored on disk when humans also need to read them. They add no accuracy benefit when consumed by a model and cost 2x to 3x more read tokens than their compact counterparts.

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure

- **Test Date**: 2026-03-22 & 2026-03-29
- **Model**: Sonnet 4.6
- **Extended Thinking**: on & off
- **Structure**: flat & nested
- **Variant**: mandatory (22 fields, dense) and optional (19 mandatory + 3 optional, sparse)
- **Formats Tested**: CSV, JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, TOON_KEYFOLD, XML_COMPACT, XML_PRETTY, YAML
- **Record Counts**: 31

### 4.2 Appendix B: Benchmark Configuration
- **Field Retrieval**: 55 questions (37.50% weight)
- **Filtering**: 21 questions (20.83% weight)
- **Aggregation**: 21 questions (12.50% weight)
- **Structure Awareness**: 27 questions (29.17% weight)

---

- **Report Generated**: 2026-04-16
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code 2.1.80
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_sonnet_4_6/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_sonnet_4_6/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_sonnet_4_6/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_sonnet_4_6/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)