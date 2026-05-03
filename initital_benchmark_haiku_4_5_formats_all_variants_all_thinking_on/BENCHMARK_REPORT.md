# File Format Token Efficiency Benchmark: Comprehensive Analysis

- **Date**: 2025-12-19
- **Model**: Claude Haiku 4.5
- **Thinking**: On
- **Data Structure**: flat
- **Formats Tested**: 6 (CSV, JSON_COMPACT, JSON_PRETTY, JSONL, TOON, YAML)
- **Data**: 40 & 80 product records as flat arrays
- **Status**: First iteration

## Executive Summary

This benchmark evaluates token efficiency and information accuracy across 6 file formats using Claude Haiku 4.5 as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context while a format that accurately conveys information may justify higher token cost.

### Key Findings

<ADD_FINDINGS_HERE>Add 3-5 major findings as a quick overview, details come later</ADD_FINDINGS_HERE>

## 1. Methodology

### 1.1 Research Purpose

The underlying question: **Which file format delivers maximum information value per token consumed?**

This requires measuring:
- **Token Cost**: How many tokens does each format consume for equivalent data?
- **Robustness**: How consistent is performance across data variants (mandatory vs optional fields)?

### 1.2 Test Design

- 6 formats tested: CSV, JSON_COMPACT, JSON_PRETTY, JSONL, TOON, YAML
- 2 variants per format: mandatory (22 fields, dense) and optional (19 mandatory + 3 optional, sparse)
- 2 record counts: 40 and 80 records

### 1.3 Token Usage Measurements

Tokens usage measured in this benchmark are no estimates but the real token usage the model used in this test. The token usage is reported to the user indirectly in the conversation transcript. Both read and output tokens are directly extracted from the transcripts of the subagents:
- **Read Tokens**: For each data file a single read subagent is invoked with the only prompt to read the file at the provided filepath and return "Done" once finished and do nothing more. The token extraction script searches for the read tool result and extracts only the read tokens of it.

### 1.4 Important Note

These results are specific to Claude Code using the Claude Haiku 4.5 model. They serve as a rule of thumb for choosing the best file format depending on the use case.
However these values cannot be exactly applied to models of the same family or from other providers as token usage and latency depend on specific model architectures and tokenizers. Also file reads will produce different characters depending on the used harness because some add marker characters, line numbers or additional information. While the relative ranking of file formats remains consistent the absolute numbers will vary.

## 2. Results

### 2.1 Token Efficiency: All Combinations

| Format | Variant | Records | Chars | Read Tokens | Tokens/1K Chars | Chars/Token | Read Duration (ms) |
|---|---|---|---|---|---|---|---|
| csv | mandatory | 40 | 13056 | 4992 | 382.35 | 2.62 | 1089 |
| csv | mandatory | 80 | 25694 | 9672 | 376.43 | 2.66 | 1762 |
| csv | optional | 40 | 12334 | 4608 | 373.6 | 2.68 | 1111 |
| csv | optional | 80 | 24412 | 8956 | 366.87 | 2.73 | 1078 |
| toon | mandatory | 40 | 13143 | 5040 | 383.47 | 2.61 | 1240 |
| toon | mandatory | 80 | 25861 | 9760 | 377.4 | 2.65 | 1328 |
| toon | optional | 40 | 26134 | 10882 | 416.39 | 2.4 | 2234 |
| toon | optional | 80 | 52166 | 22119 | 424.01 | 2.36 | 1495 |
| json_compact | mandatory | 40 | 25763 | 7948 | 308.5 | 3.24 | 2311 |
| json_compact | mandatory | 80 | 51361 | 15661 | 304.92 | 3.28 | 1294 |
| json_compact | optional | 40 | 23846 | 7250 | 304.03 | 3.29 | 1124 |
| json_compact | optional | 80 | 47620 | 14285 | 299.98 | 3.33 | 2142 |
| jsonl | mandatory | 40 | 25800 | 8109 | 314.3 | 3.18 | 1394 |
| jsonl | mandatory | 80 | 51438 | 15974 | 310.55 | 3.22 | 1444 |
| jsonl | optional | 40 | 23883 | 7403 | 309.97 | 3.23 | 1087 |
| jsonl | optional | 80 | 47697 | 14590 | 305.89 | 3.27 | 2180 |
| json_pretty | mandatory | 40 | 32245 | 14407 | 446.8 | 2.24 | 2345 |
| json_pretty | mandatory | 80 | 64323 | 29490 | 458.47 | 2.18 | 2335 |
| json_pretty | optional | 40 | 29859 | 13202 | 442.14 | 2.26 | 2215 |
| json_pretty | optional | 80 | 59602 | 26925 | 451.75 | 2.21 | 2208 |
| yaml | mandatory | 40 | 30571 | 12184 | 398.55 | 2.51 | 2536 |
| yaml | mandatory | 80 | 60969 | 24971 | 409.57 | 2.44 | 1666 |
| yaml | optional | 40 | 28324 | 11151 | 393.69 | 2.54 | 1415 |
| yaml | optional | 80 | 56530 | 22763 | 402.67 | 2.48 | 2210 |


**Note:**
- Tokens/1K Chars = read tokens per 1,000 characters (lower = more token-efficient)
- Chars/Token = characters represented per token (higher = more efficient)


### 2.2 Record Count Scaling: 40 → 80 Records

| Format | Variant | Tokens @40 | Tokens @80 | Token Scale | Chars @40 | Chars @80 | Char Scale | Token/Char Scale Ratio |
|---|---|---|---|---|---|---|---|---|
| csv | mandatory | 4992 | 9672 | 1.94x | 13056 | 25694 | 1.97x | 0.98 |
| csv | optional | 4608 | 8956 | 1.94x | 12334 | 24412 | 1.98x | 0.98 |
| toon | mandatory | 5040 | 9760 | 1.94x | 13143 | 25861 | 1.97x | 0.98 |
| toon | optional | 10882 | 22119 | 2.03x | 26134 | 52166 | 2x | 1.01 |
| json_compact | mandatory | 7948 | 15661 | 1.97x | 25763 | 51361 | 1.99x | 0.99 |
| json_compact | optional | 7250 | 14285 | 1.97x | 23846 | 47620 | 2x | 0.99 |
| jsonl | mandatory | 8109 | 15974 | 1.97x | 25800 | 51438 | 1.99x | 0.99 |
| jsonl | optional | 7403 | 14590 | 1.97x | 23883 | 47697 | 2x | 0.99 |
| json_pretty | mandatory | 14407 | 29490 | 2.05x | 32245 | 64323 | 1.99x | 1.03 |
| json_pretty | optional | 13202 | 26925 | 2.04x | 29859 | 59602 | 2x | 1.02 |
| yaml | mandatory | 12184 | 24971 | 2.05x | 30571 | 60969 | 1.99x | 1.03 |
| yaml | optional | 11151 | 22763 | 2.04x | 28324 | 56530 | 2x | 1.02 |


**Note:**
- Token Scale = tokens@80 / tokens@40
- Char Scale = chars@80 / chars@40
- Token/Char Scale Ratio > 1 means tokens grow faster than characters (inefficient scaling)


### 2.3 Mandatory vs Optional Drift

| Format | Records | Mand Tokens | Opt Tokens | Token Drift % | Mand Chars | Opt Chars | Char Drift % | Token vs Char Drift Δ |
|---|---|---|---|---|---|---|---|---|
| csv | 40 | 4992 | 4608 | -7.69% | 13056 | 12334 | -5.53% | -2.16pp |
| csv | 80 | 9672 | 8956 | -7.4% | 25694 | 24412 | -4.99% | -2.41pp |
| toon | 40 | 5040 | 10882 | 115.91% | 13143 | 26134 | 98.84% | 17.07pp |
| toon | 80 | 9760 | 22119 | 126.63% | 25861 | 52166 | 101.72% | 24.91pp |
| json_compact | 40 | 7948 | 7250 | -8.78% | 25763 | 23846 | -7.44% | -1.34pp |
| json_compact | 80 | 15661 | 14285 | -8.79% | 51361 | 47620 | -7.28% | -1.51pp |
| jsonl | 40 | 8109 | 7403 | -8.71% | 25800 | 23883 | -7.43% | -1.28pp |
| jsonl | 80 | 15974 | 14590 | -8.66% | 51438 | 47697 | -7.27% | -1.39pp |
| json_pretty | 40 | 14407 | 13202 | -8.36% | 32245 | 29859 | -7.4% | -0.96pp |
| json_pretty | 80 | 29490 | 26925 | -8.7% | 64323 | 59602 | -7.34% | -1.36pp |
| yaml | 40 | 12184 | 11151 | -8.48% | 30571 | 28324 | -7.35% | -1.13pp |
| yaml | 80 | 24971 | 22763 | -8.84% | 60969 | 56530 | -7.28% | -1.56pp |


**Note:**
- Token Drift % = (optTokens - mandTokens) / mandTokens * 100
- Char Drift % = same for characters 
- Drift Δ (pp) = Token Drift − Char Drift
- Positive Δ = token cost grows more than character count when fields become optional


### 2.4 Format Ranking (averaged across all variants and record counts)

| Rank | Format | Avg Tokens/1K Chars | Avg Chars/Token | Avg Tokens @40 | Avg Tokens @80 |
|---|---|---|---|---|---|
| 1 | json_compact | 304.36 | 3.28 | 7599 | 14973 |
| 2 | jsonl | 310.18 | 3.23 | 7756 | 15282 |
| 3 | csv | 374.81 | 2.67 | 4800 | 9314 |
| 4 | toon | 400.32 | 2.51 | 7961 | 15939.5 |
| 5 | yaml | 401.12 | 2.49 | 11667.5 | 23867 |
| 6 | json_pretty | 449.79 | 2.22 | 13804.5 | 28207.5 |


**Note:**
- Ranked by Avg Tokens/1K Chars ascending (lower = more token-efficient overall)


## 3. Conclusion & Decision Matrix

<ADD_CONTENT_HERE>


### 3.1 Cross-Configuration Findings

<ADD_CONTENT_HERE>


### 3.2 Decision Matrix

| Scenario | Recommended Format | Rationale |
|---|---|---|
| Flat structure, dense mandatory fields |  |  |
| Flat structure, sparse optional fields |  |  |
| Nested structure, dense mandatory fields |  |  |
| Nested structure, sparse optional fields |  |  |
| Maximum accuracy required, flat structure |  |  |
| Maximum accuracy required, nested structure |  |  |
| Token budget critical, flat structure |  |  |
| Token budget critical, nested structure |  |  |
| Avoid in flat structure |  |  |
| Avoid in nested structure |  |  |


### 3.3 Real-World Impact

<ADD_CONTENT_HERE>


## 4. Appendices

- **Test Date**: 2025-12-19
- **Model**: Claude Haiku 4.5
- **Thinking**: on
- **Structure**: flat
- **Formats Tested**: CSV, JSON_COMPACT, JSON_PRETTY, JSONL, TOON, YAML
- **Record Counts**: 40 & 80

---

- **Report Generated**: 2026-01-25
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code 2.0.74
- **Data Source**: `metrics.json` & `metadata.json`
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)