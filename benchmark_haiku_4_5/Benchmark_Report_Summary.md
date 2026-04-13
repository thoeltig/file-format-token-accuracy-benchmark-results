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
| **TOON_DEFAULT** | [Token-Oriented Object Notation](https://toonformat.dev/) with key folding disabled — nested structures expanded as-is |
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
However these values cannot be exactly applied to models of the same family or from other providers as token usage, accuracy and latency depend on specific model architectures and tokenizers. While the relative ranking of file formats remains consistent the absolute numbers will vary.
Especially the accuracy and output tokens results will vary because these values are bound to the model size and training, instruction interpretation and reasoning token budget.

## 2. Simplified Results

*Note: All columns ranked best-to-worst. ↑ = lower value is better (ascending). ↓ = higher value is better (descending).*

### 2.1 Flat Structure With Thinking On

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)

#### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XML_COMPACT ≈ 75s | CSV ≈ 6989 | JSON_COMPACT ≈ 228 | XML_COMPACT ≈ 9086 | XML_COMPACT ≈ 9429 | CSV ≈ 16525 | JSON_PRETTY ≈ 79.30% | TOON_DEFAULT ≈ 83 | XML_COMPACT ≈ 77 | TOON_DEFAULT ≈ 79 | JSON_PRETTY ≈ 93.58% | TOON_DEFAULT ≈ 94 | XML_COMPACT ≈ 89 | TOON_DEFAULT ≈ 90 |
| CSV (+3.89%) | TOON_DEFAULT (+0.84%) | YAML (+1.46%) | CSV (+1.33%) | CSV (+1.14%) | TOON_DEFAULT (+13.27%) | TOON_DEFAULT (-2.55%) | CSV (-11.70%) | YAML (-8.93%) | CSV (-5.18%) | TOON_DEFAULT (-0.36%) | CSV (-6.13%) | CSV (-6.78%) | CSV (-0.11%) |
| YAML (+17.18%) | JSON_COMPACT (+32.61%) | TOON_DEFAULT (+26.06%) | YAML (+13.96%) | YAML (+12.26%) | XML_COMPACT (+27.81%) | XML_PRETTY (-4.30%) | JSON_COMPACT (-13.36%) | CSV (-11.53%) | XML_COMPACT (-10.35%) | XML_COMPACT (-1.18%) | JSON_COMPACT (-9.67%) | YAML (-7.36%) | XML_COMPACT (-7.80%) |
| TOON_DEFAULT (+27.39%) | XML_COMPACT (+67.31%) | CSV (+44.66%) | TOON_DEFAULT (+25.29%) | TOON_DEFAULT (+23.78%) | JSON_COMPACT (+35.72%) | XML_COMPACT (-5.10%) | XML_COMPACT (-21.65%) | TOON_DEFAULT (-13.74%) | JSON_COMPACT (-16.87%) | XML_PRETTY (-1.22%) | XML_COMPACT (-17.90%) | TOON_DEFAULT (-13.16%) | JSON_COMPACT (-12.55%) |
| XML_PRETTY (+36.22%) | YAML (+79.63%) | JSON_PRETTY (+49.78%) | XML_PRETTY (+36.89%) | XML_PRETTY (+35.55%) | YAML (+40.02%) | YAML (-5.91%) | YAML (-25.93%) | JSON_PRETTY (-21.96%) | YAML (-17.92%) | YAML (-1.52%) | YAML (-21.36%) | XML_PRETTY (-20.62%) | YAML (-14.08%) |
| JSON_PRETTY (+40.74%) | JSON_PRETTY (+104.36%) | XML_COMPACT (+50.51%) | JSON_PRETTY (+40.85%) | JSON_PRETTY (+39.34%) | JSON_PRETTY (+65.94%) | JSON_COMPACT (-7.53%) | JSON_PRETTY (-28.50%) | XML_PRETTY (-23.16%) | JSON_PRETTY (-27.50%) | JSON_COMPACT (-2.32%) | JSON_PRETTY (-26.73%) | JSON_PRETTY (-21.91%) | JSON_PRETTY (-25.76%) |
| JSON_COMPACT (+45.75%) | XML_PRETTY (+131.31%) | XML_PRETTY (+50.51%) | JSON_COMPACT (+42.34%) | JSON_COMPACT (+39.58%) | XML_PRETTY (+75.16%) | CSV (-17.47%) | XML_PRETTY (-39.89%) | JSON_COMPACT (-28.68%) | XML_PRETTY (-36.35%) | CSV (-9.33%) | XML_PRETTY (-34.61%) | JSON_COMPACT (-23.79%) | XML_PRETTY (-31.23%) |

#### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 73s | CSV ≈ 6700 | CSV ≈ 231 | JSON_PRETTY ≈ 7966 | JSON_PRETTY ≈ 8302 | JSON_COMPACT ≈ 18753 | XML_PRETTY ≈ 77.96% | JSON_COMPACT ≈ 77 | JSON_PRETTY ≈ 83 | JSON_COMPACT ≈ 78 | XML_PRETTY ≈ 92.40% | CSV ≈ 91 | JSON_PRETTY ≈ 94 | JSON_COMPACT ≈ 88 |
| XML_PRETTY (+7.93%) | JSON_COMPACT (+30.57%) | XML_PRETTY (+0.58%) | JSON_COMPACT (+21.36%) | JSON_COMPACT (+20.52%) | CSV (+4.22%) | TOON_DEFAULT (-0.41%) | CSV (-4.61%) | XML_PRETTY (-8.54%) | JSON_PRETTY (-11.59%) | XML_COMPACT (-0.21%) | JSON_COMPACT (-4.09%) | XML_PRETTY (-8.98%) | CSV (-6.36%) |
| JSON_COMPACT (+13.19%) | XML_COMPACT (+63.13%) | YAML (+44.30%) | XML_PRETTY (+22.97%) | XML_PRETTY (+20.79%) | JSON_PRETTY (+15.55%) | XML_COMPACT (-0.54%) | XML_COMPACT (-9.02%) | JSON_COMPACT (-9.69%) | XML_COMPACT (-11.67%) | TOON_DEFAULT (-0.86%) | XML_COMPACT (-11.86%) | JSON_COMPACT (-9.66%) | JSON_PRETTY (-9.10%) |
| YAML (+21.03%) | TOON_DEFAULT (+72.55%) | XML_COMPACT (+45.17%) | YAML (+27.74%) | YAML (+26.59%) | YAML (+18.81%) | JSON_COMPACT (-1.62%) | TOON_DEFAULT (-11.79%) | YAML (-13.22%) | YAML (-12.31%) | JSON_COMPACT (-1.15%) | TOON_DEFAULT (-14.78%) | YAML (-12.60%) | XML_COMPACT (-10.46%) |
| TOON_DEFAULT (+28.94%) | YAML (+75.69%) | JSON_PRETTY (+45.45%) | XML_COMPACT (+40.13%) | XML_COMPACT (+38.50%) | XML_COMPACT (+19.60%) | YAML (-1.89%) | YAML (-14.03%) | XML_COMPACT (-18.64%) | TOON_DEFAULT (-14.64%) | YAML (-1.17%) | YAML (-15.83%) | XML_COMPACT (-17.66%) | YAML (-10.73%) |
| XML_COMPACT (+30.13%) | JSON_PRETTY (+99.51%) | JSON_COMPACT (+46.18%) | TOON_DEFAULT (+43.45%) | TOON_DEFAULT (+41.74%) | TOON_DEFAULT (+24.40%) | JSON_PRETTY (-3.50%) | JSON_PRETTY (-22.71%) | TOON_DEFAULT (-20.30%) | CSV (-16.45%) | JSON_PRETTY (-1.47%) | JSON_PRETTY (-22.23%) | TOON_DEFAULT (-19.69%) | TOON_DEFAULT (-13.68%) |
| CSV (+50.08%) | XML_PRETTY (+125.21%) | TOON_DEFAULT (+47.26%) | CSV (+58.34%) | CSV (+54.72%) | XML_PRETTY (+33.94%) | CSV (-17.74%) | XML_PRETTY (-27.54%) | CSV (-41.32%) | XML_PRETTY (-20.42%) | CSV (-6.38%) | XML_PRETTY (-27.83%) | CSV (-29.86%) | XML_PRETTY (-18.47%) |


### 2.2 Flat Structure With Thinking Off

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)

#### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 76s | CSV ≈ 7022 | CSV ≈ 241 | XML_COMPACT ≈ 6673 | XML_COMPACT ≈ 6976 | CSV ≈ 17340 | JSON_PRETTY ≈ 79.57% | TOON_DEFAULT ≈ 81 | XML_COMPACT ≈ 81 | TOON_DEFAULT ≈ 77 | JSON_PRETTY ≈ 92.57% | TOON_DEFAULT ≈ 93 | XML_COMPACT ≈ 94 | CSV ≈ 91 |
| XML_COMPACT (+5.78%) | TOON_DEFAULT (+2.28%) | TOON_DEFAULT (+7.02%) | YAML (+35.93%) | YAML (+34.36%) | TOON_DEFAULT (+8.40%) | JSON_COMPACT (-3.56%) | CSV (-6.92%) | YAML (-15.17%) | CSV (-1.33%) | XML_PRETTY (-0.43%) | CSV (-3.74%) | YAML (-12.78%) | TOON_DEFAULT (-1.20%) |
| CSV (+6.64%) | JSON_COMPACT (+32.38%) | JSON_PRETTY (+25.38%) | CSV (+51.01%) | CSV (+47.91%) | XML_COMPACT (+9.15%) | TOON_DEFAULT (-5.82%) | JSON_COMPACT (-7.33%) | XML_PRETTY (-21.62%) | XML_COMPACT (-2.98%) | JSON_COMPACT (-0.59%) | JSON_COMPACT (-7.92%) | XML_PRETTY (-19.14%) | XML_COMPACT (-2.01%) |
| XML_PRETTY (+13.92%) | XML_COMPACT (+70.18%) | YAML (+25.38%) | XML_PRETTY (+54.29%) | XML_PRETTY (+51.95%) | YAML (+26.57%) | XML_PRETTY (-6.99%) | XML_COMPACT (-23.03%) | JSON_PRETTY (-24.55%) | JSON_COMPACT (-16.21%) | TOON_DEFAULT (-0.70%) | XML_COMPACT (-18.34%) | CSV (-22.09%) | YAML (-13.56%) |
| TOON_DEFAULT (+25.23%) | YAML (+79.07%) | XML_COMPACT (+25.80%) | TOON_DEFAULT (+70.19%) | TOON_DEFAULT (+66.50%) | JSON_COMPACT (+31.40%) | XML_COMPACT (-8.60%) | YAML (-25.74%) | CSV (-26.50%) | YAML (-16.74%) | YAML (-0.83%) | YAML (-20.49%) | TOON_DEFAULT (-24.82%) | JSON_COMPACT (-16.64%) |
| JSON_PRETTY (+26.80%) | JSON_PRETTY (+103.80%) | JSON_COMPACT (+26.35%) | JSON_PRETTY (+74.95%) | JSON_PRETTY (+71.68%) | JSON_PRETTY (+51.60%) | YAML (-8.60%) | JSON_PRETTY (-26.21%) | TOON_DEFAULT (-27.07%) | JSON_PRETTY (-29.09%) | XML_COMPACT (-1.11%) | JSON_PRETTY (-26.47%) | JSON_PRETTY (-26.29%) | JSON_PRETTY (-29.83%) |
| JSON_COMPACT (+40.46%) | XML_PRETTY (+130.50%) | XML_PRETTY (+26.49%) | JSON_COMPACT (+97.57%) | JSON_COMPACT (+93.36%) | XML_PRETTY (+54.48%) | CSV (-15.06%) | XML_PRETTY (-40.12%) | JSON_COMPACT (-37.06%) | XML_PRETTY (-37.39%) | CSV (-6.76%) | XML_PRETTY (-33.88%) | JSON_COMPACT (-34.89%) | XML_PRETTY (-32.09%) |

#### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 74s | CSV ≈ 6728 | JSON_PRETTY ≈ 231 | TOON_DEFAULT ≈ 8675 | TOON_DEFAULT ≈ 8984 | CSV ≈ 17689 | YAML ≈ 78.23% | JSON_COMPACT ≈ 76 | TOON_DEFAULT ≈ 72 | JSON_COMPACT ≈ 80 | JSON_PRETTY ≈ 92.07% | CSV ≈ 91 | TOON_DEFAULT ≈ 83 | JSON_COMPACT ≈ 91 |
| XML_PRETTY (+0.30%) | JSON_COMPACT (+30.32%) | XML_PRETTY (+30.74%) | XML_PRETTY (+2.68%) | XML_PRETTY (+2.51%) | JSON_COMPACT (+3.60%) | JSON_PRETTY (-1.78%) | CSV (-0.46%) | XML_PRETTY (-0.44%) | CSV (-6.65%) | YAML (-0.17%) | JSON_COMPACT (-4.09%) | XML_PRETTY (-1.32%) | CSV (-1.36%) |
| JSON_COMPACT (+2.14%) | XML_COMPACT (+66.19%) | CSV (+31.43%) | JSON_COMPACT (+6.66%) | JSON_COMPACT (+6.38%) | TOON_DEFAULT (+16.98%) | XML_COMPACT (-1.89%) | XML_COMPACT (-9.59%) | JSON_COMPACT (-2.86%) | TOON_DEFAULT (-11.58%) | JSON_COMPACT (-0.93%) | XML_COMPACT (-13.99%) | JSON_COMPACT (-3.08%) | TOON_DEFAULT (-9.60%) |
| JSON_PRETTY (+14.79%) | TOON_DEFAULT (+74.03%) | YAML (+31.60%) | JSON_PRETTY (+17.35%) | JSON_PRETTY (+15.88%) | XML_COMPACT (+26.89%) | JSON_COMPACT (-3.71%) | YAML (-10.76%) | JSON_PRETTY (-7.13%) | XML_COMPACT (-16.73%) | XML_PRETTY (-1.39%) | YAML (-15.26%) | JSON_PRETTY (-7.55%) | XML_COMPACT (-16.56%) |
| CSV (+16.87%) | YAML (+75.25%) | JSON_COMPACT (+31.95%) | CSV (+22.86%) | CSV (+22.01%) | JSON_PRETTY (+34.58%) | XML_PRETTY (-3.77%) | TOON_DEFAULT (-14.80%) | XML_COMPACT (-13.30%) | JSON_PRETTY (-22.65%) | TOON_DEFAULT (-1.47%) | TOON_DEFAULT (-15.90%) | XML_COMPACT (-14.11%) | JSON_PRETTY (-20.62%) |
| XML_COMPACT (+18.40%) | JSON_PRETTY (+99.09%) | XML_COMPACT (+33.77%) | XML_COMPACT (+26.30%) | XML_COMPACT (+25.39%) | XML_PRETTY (+37.52%) | TOON_DEFAULT (-5.02%) | JSON_PRETTY (-19.77%) | CSV (-23.30%) | YAML (-25.65%) | XML_COMPACT (-1.66%) | JSON_PRETTY (-21.37%) | CSV (-15.82%) | XML_PRETTY (-23.67%) |
| YAML (+40.42%) | XML_PRETTY (+124.69%) | TOON_DEFAULT (+33.84%) | YAML (+46.69%) | YAML (+45.02%) | YAML (+40.31%) | CSV (-15.00%) | XML_PRETTY (-29.51%) | YAML (-24.07%) | XML_PRETTY (-26.63%) | CSV (-6.14%) | XML_PRETTY (-29.07%) | YAML (-23.70%) | YAML (-24.69%) |


### 2.3 Nested Structure With Thinking On

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)

*CSV excluded — flat-only format, does not support nested structures.*

#### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT ≈ 69s | JSON_COMPACT ≈ 10315 | JSON_COMPACT ≈ 336 | XML_PRETTY ≈ 7175 | XML_PRETTY ≈ 7518 | JSON_COMPACT ≈ 18446 | YAML ≈ 75.80% | JSON_COMPACT ≈ 80 | XML_PRETTY ≈ 80 | JSON_COMPACT ≈ 81 | YAML ≈ 92.87% | JSON_COMPACT ≈ 92 | XML_PRETTY ≈ 93 | JSON_COMPACT ≈ 93 |
| XML_PRETTY (+2.18%) | XML_COMPACT (+24.56%) | YAML (+0.50%) | JSON_COMPACT (+8.64%) | JSON_COMPACT (+8.16%) | XML_COMPACT (+18.26%) | JSON_PRETTY (-1.88%) | XML_COMPACT (-10.87%) | JSON_COMPACT (-1.53%) | XML_COMPACT (-13.18%) | TOON_DEFAULT (-0.68%) | XML_COMPACT (-8.91%) | JSON_COMPACT (-2.93%) | XML_COMPACT (-10.94%) |
| TOON_DEFAULT (+6.95%) | TOON_DEFAULT (+36.66%) | TOON_DEFAULT (+1.74%) | TOON_DEFAULT (+16.27%) | TOON_DEFAULT (+15.52%) | TOON_DEFAULT (+23.50%) | JSON_COMPACT (-3.22%) | YAML (-13.39%) | TOON_DEFAULT (-5.77%) | TOON_DEFAULT (-17.33%) | JSON_PRETTY (-1.48%) | TOON_DEFAULT (-12.33%) | TOON_DEFAULT (-4.63%) | TOON_DEFAULT (-13.14%) |
| XML_COMPACT (+8.00%) | YAML (+38.69%) | XML_PRETTY (+1.88%) | XML_COMPACT (+20.16%) | XML_COMPACT (+19.27%) | YAML (+37.75%) | XML_COMPACT (-4.02%) | TOON_DEFAULT (-16.46%) | XML_COMPACT (-6.73%) | YAML (-23.25%) | XML_PRETTY (-1.76%) | YAML (-12.57%) | XML_COMPACT (-6.86%) | YAML (-21.14%) |
| YAML (+31.78%) | JSON_PRETTY (+72.84%) | XML_COMPACT (+2.58%) | JSON_PRETTY (+43.88%) | JSON_PRETTY (+43.27%) | XML_PRETTY (+49.79%) | TOON_DEFAULT (-4.70%) | JSON_PRETTY (-29.14%) | JSON_PRETTY (-14.73%) | XML_PRETTY (-35.92%) | JSON_COMPACT (-1.89%) | JSON_PRETTY (-25.94%) | JSON_PRETTY (-14.85%) | XML_PRETTY (-29.58%) |
| JSON_PRETTY (+34.89%) | XML_PRETTY (+95.00%) | JSON_PRETTY (+33.04%) | YAML (+50.05%) | YAML (+47.71%) | JSON_PRETTY (+55.04%) | XML_PRETTY (-5.37%) | XML_PRETTY (-41.26%) | YAML (-14.97%) | JSON_PRETTY (-36.65%) | XML_COMPACT (-1.98%) | XML_PRETTY (-34.13%) | YAML (-15.33%) | JSON_PRETTY (-32.50%) |

#### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XML_COMPACT ≈ 64s | JSON_COMPACT ≈ 9788 | JSON_PRETTY ≈ 228 | XML_COMPACT ≈ 7011 | XML_COMPACT ≈ 7356 | JSON_COMPACT ≈ 18250 | YAML ≈ 77.42% | JSON_COMPACT ≈ 83 | XML_COMPACT ≈ 82 | JSON_COMPACT ≈ 83 | YAML ≈ 92.94% | JSON_COMPACT ≈ 94 | XML_COMPACT ≈ 94 | JSON_COMPACT ≈ 94 |
| XML_PRETTY (+21.07%) | XML_COMPACT (+26.36%) | JSON_COMPACT (+47.44%) | JSON_COMPACT (+15.90%) | JSON_COMPACT (+15.04%) | XML_COMPACT (+8.07%) | TOON_DEFAULT (-0.94%) | XML_COMPACT (-11.48%) | JSON_COMPACT (-4.29%) | XML_COMPACT (-6.84%) | JSON_COMPACT (-1.37%) | XML_COMPACT (-9.49%) | JSON_COMPACT (-4.41%) | XML_COMPACT (-5.39%) |
| JSON_PRETTY (+31.75%) | TOON_DEFAULT (+41.59%) | YAML (+47.58%) | XML_PRETTY (+33.45%) | JSON_PRETTY (+31.31%) | TOON_DEFAULT (+42.74%) | JSON_COMPACT (-2.42%) | YAML (-14.56%) | XML_PRETTY (-13.87%) | TOON_DEFAULT (-27.05%) | TOON_DEFAULT (-1.61%) | YAML (-13.60%) | XML_PRETTY (-11.11%) | TOON_DEFAULT (-25.10%) |
| JSON_COMPACT (+37.52%) | YAML (+43.57%) | XML_PRETTY (+51.10%) | JSON_PRETTY (+34.52%) | XML_PRETTY (+31.88%) | JSON_PRETTY (+45.52%) | XML_COMPACT (-4.30%) | TOON_DEFAULT (-14.56%) | JSON_PRETTY (-15.18%) | YAML (-29.56%) | XML_COMPACT (-2.33%) | TOON_DEFAULT (-14.07%) | JSON_PRETTY (-11.84%) | YAML (-26.84%) |
| YAML (+47.75%) | JSON_PRETTY (+72.65%) | XML_COMPACT (+51.24%) | TOON_DEFAULT (+68.86%) | TOON_DEFAULT (+65.74%) | YAML (+47.67%) | XML_PRETTY (-6.18%) | JSON_PRETTY (-32.03%) | TOON_DEFAULT (-22.72%) | JSON_PRETTY (-34.60%) | XML_PRETTY (-2.76%) | JSON_PRETTY (-26.18%) | TOON_DEFAULT (-21.77%) | JSON_PRETTY (-28.45%) |
| TOON_DEFAULT (+51.18%) | XML_PRETTY (+100.07%) | TOON_DEFAULT (+54.50%) | YAML (+79.14%) | YAML (+75.33%) | XML_PRETTY (+60.46%) | JSON_PRETTY (-8.07%) | XML_PRETTY (-40.89%) | YAML (-25.66%) | XML_PRETTY (-42.95%) | JSON_PRETTY (-4.06%) | XML_PRETTY (-34.43%) | YAML (-23.87%) | XML_PRETTY (-36.25%) |


### 2.4 Nested Structure With Thinking Off

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)

*CSV excluded — flat-only format, does not support nested structures.*

#### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 77s | JSON_COMPACT ≈ 10163 | YAML ≈ 213 | XML_COMPACT ≈ 4990 | XML_COMPACT ≈ 5295 | XML_COMPACT ≈ 18000 | XML_COMPACT ≈ 74.19% | JSON_COMPACT ≈ 81 | XML_COMPACT ≈ 83 | XML_COMPACT ≈ 83 | JSON_COMPACT ≈ 93.04% | JSON_COMPACT ≈ 94 | XML_COMPACT ≈ 95 | XML_COMPACT ≈ 95 |
| XML_COMPACT (+3.89%) | XML_COMPACT (+25.01%) | XML_PRETTY (+42.12%) | TOON_DEFAULT (+72.46%) | TOON_DEFAULT (+68.36%) | JSON_COMPACT (+11.63%) | YAML (-0.54%) | XML_COMPACT (-9.28%) | TOON_DEFAULT (-17.84%) | JSON_COMPACT (-7.57%) | XML_COMPACT (-0.25%) | XML_COMPACT (-8.73%) | TOON_DEFAULT (-14.89%) | JSON_COMPACT (-5.85%) |
| XML_PRETTY (+6.07%) | TOON_DEFAULT (+38.42%) | XML_COMPACT (+43.42%) | JSON_PRETTY (+79.09%) | JSON_PRETTY (+74.56%) | TOON_DEFAULT (+27.68%) | JSON_COMPACT (-0.80%) | YAML (-15.39%) | JSON_PRETTY (-19.95%) | TOON_DEFAULT (-17.71%) | TOON_DEFAULT (-0.87%) | TOON_DEFAULT (-13.75%) | JSON_PRETTY (-16.49%) | TOON_DEFAULT (-14.78%) |
| JSON_COMPACT (+6.56%) | YAML (+39.28%) | JSON_COMPACT (+43.51%) | JSON_COMPACT (+92.88%) | JSON_COMPACT (+87.53%) | YAML (+34.37%) | TOON_DEFAULT (-1.51%) | TOON_DEFAULT (-15.85%) | JSON_COMPACT (-21.93%) | YAML (-20.91%) | YAML (-0.97%) | YAML (-14.11%) | JSON_COMPACT (-18.34%) | YAML (-18.31%) |
| TOON_DEFAULT (+7.89%) | JSON_PRETTY (+73.98%) | JSON_PRETTY (+44.17%) | XML_PRETTY (+93.24%) | XML_PRETTY (+87.81%) | JSON_PRETTY (+49.58%) | JSON_PRETTY (-2.26%) | JSON_PRETTY (-30.60%) | YAML (-22.19%) | JSON_PRETTY (-31.36%) | JSON_PRETTY (-1.28%) | JSON_PRETTY (-26.20%) | YAML (-19.42%) | JSON_PRETTY (-26.41%) |
| YAML (+11.79%) | XML_PRETTY (+98.80%) | TOON_DEFAULT (+45.31%) | YAML (+96.77%) | YAML (+89.45%) | XML_PRETTY (+67.49%) | XML_PRETTY (-6.18%) | XML_PRETTY (-43.71%) | XML_PRETTY (-26.33%) | XML_PRETTY (-45.19%) | XML_PRETTY (-1.92%) | XML_PRETTY (-35.13%) | XML_PRETTY (-19.74%) | XML_PRETTY (-36.14%) |

#### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total | ↓ Accuracy By Character | ↓ Eff Score Read (Acc By Char) | ↓ Eff Score Output (Acc By Char) | ↓ Eff Score Total (Acc By Char) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 67s | JSON_COMPACT ≈ 9645 | XML_PRETTY ≈ 205 | YAML ≈ 7786 | YAML ≈ 8089 | JSON_COMPACT ≈ 21434 | JSON_COMPACT ≈ 74.84% | JSON_COMPACT ≈ 83 | XML_PRETTY ≈ 67 | JSON_COMPACT ≈ 74 | JSON_COMPACT ≈ 91.36% | JSON_COMPACT ≈ 94 | YAML ≈ 82 | JSON_COMPACT ≈ 85 |
| XML_PRETTY (+2.24%) | XML_COMPACT (+29.13%) | TOON_DEFAULT (+25.69%) | XML_PRETTY (+3.99%) | XML_PRETTY (+2.63%) | XML_COMPACT (+1.44%) | TOON_DEFAULT (-1.34%) | XML_COMPACT (-12.45%) | XML_COMPACT (-1.31%) | XML_COMPACT (-3.19%) | TOON_DEFAULT (-0.29%) | XML_COMPACT (-9.82%) | XML_PRETTY (-1.31%) | XML_COMPACT (-1.46%) |
| XML_COMPACT (+13.55%) | TOON_DEFAULT (+43.41%) | YAML (+47.64%) | XML_COMPACT (+15.34%) | XML_COMPACT (+14.82%) | YAML (+3.74%) | XML_COMPACT (-2.26%) | TOON_DEFAULT (-16.93%) | YAML (-2.79%) | YAML (-12.55%) | XML_COMPACT (-0.59%) | TOON_DEFAULT (-14.21%) | XML_COMPACT (-4.08%) | YAML (-4.49%) |
| JSON_PRETTY (+39.93%) | YAML (+46.69%) | JSON_COMPACT (+48.68%) | JSON_PRETTY (+46.79%) | JSON_PRETTY (+45.07%) | TOON_DEFAULT (+20.11%) | JSON_PRETTY (-2.53%) | YAML (-25.54%) | JSON_COMPACT (-13.18%) | TOON_DEFAULT (-17.21%) | JSON_PRETTY (-1.10%) | YAML (-16.77%) | JSON_COMPACT (-15.20%) | TOON_DEFAULT (-14.15%) |
| JSON_COMPACT (+40.79%) | JSON_PRETTY (+73.74%) | JSON_PRETTY (+48.78%) | JSON_COMPACT (+47.50%) | JSON_COMPACT (+45.75%) | XML_PRETTY (+30.50%) | XML_PRETTY (-6.56%) | JSON_PRETTY (-28.96%) | TOON_DEFAULT (-15.20%) | JSON_PRETTY (-28.48%) | YAML (-2.41%) | JSON_PRETTY (-24.57%) | JSON_PRETTY (-15.84%) | XML_PRETTY (-23.33%) |
| TOON_DEFAULT (+40.96%) | XML_PRETTY (+103.95%) | XML_COMPACT (+49.76%) | TOON_DEFAULT (+49.68%) | TOON_DEFAULT (+47.27%) | JSON_PRETTY (+32.92%) | YAML (-10.59%) | XML_PRETTY (-43.23%) | JSON_PRETTY (-15.37%) | XML_PRETTY (-30.20%) | XML_PRETTY (-2.81%) | XML_PRETTY (-35.52%) | TOON_DEFAULT (-16.00%) | JSON_PRETTY (-23.66%) |


## 3. Conclusion & Decision Matrix

### 3.1 Cross-Configuration Findings

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

### 3.2 Decision Matrix

| Scenario | Recommended Format | Eff. Score Total | Accuracy | Rationale |
|---|---|---|---|---|
| Flat structure, dense mandatory fields |  |  |  |  |
| Flat structure, sparse optional fields |  |  |  |  |
| Nested structure, any field density |  |  |  |  |
| Maximum accuracy required, flat structure |  |  |  |  |
| Maximum accuracy required, nested structure |  |  |  |  |
| Token budget critical, flat only |   |  |  |  |
| Avoid in all use cases |   |  |  |  |

### 3.3 Real-World Impact

<ADD_CONTENT_HERE>Analysis here</ADD_CONTENT_HERE>

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

- **Report Generated**: 2026-04-13
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code < 2.1.86
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Full Benchmark Reports**: 
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)