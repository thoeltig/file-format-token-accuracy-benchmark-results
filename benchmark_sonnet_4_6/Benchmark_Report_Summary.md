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




### 2.2 Flat Structure With Thinking Off

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_sonnet_4_6/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)




### 2.3 Nested Structure With Thinking On

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_sonnet_4_6/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)

*CSV excluded — flat-only format, does not support nested structures.*




### 2.4 Nested Structure With Thinking Off

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_sonnet_4_6/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)

*CSV excluded — flat-only format, does not support nested structures.*




## 3. Conclusion & Decision Matrix

### 3.1 Cross-Configuration Findings

<ADD_FININGS_HERE>

### 3.2 Decision Matrix

| Scenario | Recommended Format | Eff. Score Total | Accuracy | Rationale |
|---|---|---|---|---|
| Flat structure, dense mandatory fields |  |  |  |  |
| Flat structure, sparse optional fields |  |  |  |  |
| Nested structure, dense mandatory fields |  |  |  |  |
| Nested structure, sparse optional fields |  |  |  |  |
| Maximum accuracy required, flat structure |  |  |  |  |
| Maximum accuracy required, nested structure |  |  |  |  |
| Token budget critical, flat structure |  |  |  |  |
| Token budget critical, nested structure |  |  |  |  |
| Avoid in flat structure |  |  |  |  |
| Avoid in nested structure |  |  |  |  |

### 3.3 Real-World Impact

<ADD_FININGS_HERE>

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