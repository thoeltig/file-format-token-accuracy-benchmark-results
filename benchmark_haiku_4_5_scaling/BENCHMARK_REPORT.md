# File Format Token Efficiency Benchmark: Comprehensive Analysis

- **Date**: 2025-12-19
- **Model**: Claude Haiku 4.5
- **Thinking**: On
- **Data Structure**: flat
- **Formats Tested**: 6 (CSV, JSON_COMPACT, JSON_PRETTY, JSONL, TOON, YAML)
- **Data**: 40 & 80 product records as flat arrays
- **Status**: First iteration

## Executive Summary

This benchmark measures read token cost across 6 file formats using Claude Haiku 4.5 as the inference model. The underlying question is straightforward: for an equivalent payload of structured data which format produces the smallest read token footprint when loaded into the model's context. Lower read token cost translates directly into more context headroom for downstream reasoning and lower per call cost at scale.

### Key Findings

1. **CSV** consumes the fewest read tokens in every cell of the matrix (4608 to 9672 tokens). It beats **JSON_COMPACT** by roughly 35% and **JSON_PRETTY** by roughly 65% for the same information.
2. With dense mandatory fields **TOON** matches **CSV** at 9760 tokens for 80 records but with optional fields **TOON** jumps to 22119 tokens which is a 127% increase. Every other format becomes 7-9% cheaper with sparse data.
3. Best tokens per character rate does not equal lowest absolute cost as **JSON_COMPACT** demonstrates. It only requires 304 tokens per 1K characters compared to **CSV** which requires 374 tokens per 1K characters but **CSV** still wins overall because it needs roughly half the characters to encode the same data.
4. **JSON_PRETTY** is the most expensive format because it costs roughly 3x **CSV** and 2x **JSON_COMPACT** for identical data. Whitespace indentation actively burns tokens with no apparent retrieval benefit at this stage of the benchmark.
5. Token cost scales near linearly with record count for all formats, structures and variants. Tokens increase between 1.94-2.04 and chars increase 1.97-2.00 from 40 to 80 records which reliably predicts that the same scale applies below and above these record counts.

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
| CSV | mandatory | 40 | 13056 | 4992 | 382.35 | 2.62 | 1089 |
| CSV | mandatory | 80 | 25694 | 9672 | 376.43 | 2.66 | 1762 |
| CSV | optional | 40 | 12334 | 4608 | 373.6 | 2.68 | 1111 |
| CSV | optional | 80 | 24412 | 8956 | 366.87 | 2.73 | 1078 |
| TOON | mandatory | 40 | 13143 | 5040 | 383.47 | 2.61 | 1240 |
| TOON | mandatory | 80 | 25861 | 9760 | 377.4 | 2.65 | 1328 |
| TOON | optional | 40 | 26134 | 10882 | 416.39 | 2.4 | 2234 |
| TOON | optional | 80 | 52166 | 22119 | 424.01 | 2.36 | 1495 |
| JSON_COMPACT | mandatory | 40 | 25763 | 7948 | 308.5 | 3.24 | 2311 |
| JSON_COMPACT | mandatory | 80 | 51361 | 15661 | 304.92 | 3.28 | 1294 |
| JSON_COMPACT | optional | 40 | 23846 | 7250 | 304.03 | 3.29 | 1124 |
| JSON_COMPACT | optional | 80 | 47620 | 14285 | 299.98 | 3.33 | 2142 |
| JSONL | mandatory | 40 | 25800 | 8109 | 314.3 | 3.18 | 1394 |
| JSONL | mandatory | 80 | 51438 | 15974 | 310.55 | 3.22 | 1444 |
| JSONL | optional | 40 | 23883 | 7403 | 309.97 | 3.23 | 1087 |
| JSONL | optional | 80 | 47697 | 14590 | 305.89 | 3.27 | 2180 |
| JSON_PRETTY | mandatory | 40 | 32245 | 14407 | 446.8 | 2.24 | 2345 |
| JSON_PRETTY | mandatory | 80 | 64323 | 29490 | 458.47 | 2.18 | 2335 |
| JSON_PRETTY | optional | 40 | 29859 | 13202 | 442.14 | 2.26 | 2215 |
| JSON_PRETTY | optional | 80 | 59602 | 26925 | 451.75 | 2.21 | 2208 |
| YAML | mandatory | 40 | 30571 | 12184 | 398.55 | 2.51 | 2536 |
| YAML | mandatory | 80 | 60969 | 24971 | 409.57 | 2.44 | 1666 |
| YAML | optional | 40 | 28324 | 11151 | 393.69 | 2.54 | 1415 |
| YAML | optional | 80 | 56530 | 22763 | 402.67 | 2.48 | 2210 |


**Note:**
- Tokens/1K Chars = read tokens per 1,000 characters (lower = more token-efficient)
- Chars/Token = characters represented per token (higher = more efficient)


### 2.2 Record Count Scaling: 40 → 80 Records

| Format | Variant | Tokens @40 | Tokens @80 | Token Scale | Chars @40 | Chars @80 | Char Scale | Token/Char Scale Ratio |
|---|---|---|---|---|---|---|---|---|
| CSV | mandatory | 4992 | 9672 | 1.94x | 13056 | 25694 | 1.97x | 0.98 |
| CSV | optional | 4608 | 8956 | 1.94x | 12334 | 24412 | 1.98x | 0.98 |
| TOON | mandatory | 5040 | 9760 | 1.94x | 13143 | 25861 | 1.97x | 0.98 |
| TOON | optional | 10882 | 22119 | 2.03x | 26134 | 52166 | 2x | 1.01 |
| JSON_COMPACT | mandatory | 7948 | 15661 | 1.97x | 25763 | 51361 | 1.99x | 0.99 |
| JSON_COMPACT | optional | 7250 | 14285 | 1.97x | 23846 | 47620 | 2x | 0.99 |
| JSONL | mandatory | 8109 | 15974 | 1.97x | 25800 | 51438 | 1.99x | 0.99 |
| JSONL | optional | 7403 | 14590 | 1.97x | 23883 | 47697 | 2x | 0.99 |
| JSON_PRETTY | mandatory | 14407 | 29490 | 2.05x | 32245 | 64323 | 1.99x | 1.03 |
| JSON_PRETTY | optional | 13202 | 26925 | 2.04x | 29859 | 59602 | 2x | 1.02 |
| YAML | mandatory | 12184 | 24971 | 2.05x | 30571 | 60969 | 1.99x | 1.03 |
| YAML | optional | 11151 | 22763 | 2.04x | 28324 | 56530 | 2x | 1.02 |


**Note:**
- Token Scale = tokens@80 / tokens@40
- Char Scale = chars@80 / chars@40
- Token/Char Scale Ratio > 1 means tokens grow faster than characters (inefficient scaling)


### 2.3 Mandatory vs Optional Drift

| Format | Records | Mand Tokens | Opt Tokens | Token Drift % | Mand Chars | Opt Chars | Char Drift % | Token vs Char Drift Δ |
|---|---|---|---|---|---|---|---|---|
| CSV | 40 | 4992 | 4608 | -7.69% | 13056 | 12334 | -5.53% | -2.16pp |
| CSV | 80 | 9672 | 8956 | -7.4% | 25694 | 24412 | -4.99% | -2.41pp |
| TOON | 40 | 5040 | 10882 | 115.91% | 13143 | 26134 | 98.84% | 17.07pp |
| TOON | 80 | 9760 | 22119 | 126.63% | 25861 | 52166 | 101.72% | 24.91pp |
| JSON_COMPACT | 40 | 7948 | 7250 | -8.78% | 25763 | 23846 | -7.44% | -1.34pp |
| JSON_COMPACT | 80 | 15661 | 14285 | -8.79% | 51361 | 47620 | -7.28% | -1.51pp |
| JSONL | 40 | 8109 | 7403 | -8.71% | 25800 | 23883 | -7.43% | -1.28pp |
| JSONL | 80 | 15974 | 14590 | -8.66% | 51438 | 47697 | -7.27% | -1.39pp |
| JSON_PRETTY | 40 | 14407 | 13202 | -8.36% | 32245 | 29859 | -7.4% | -0.96pp |
| JSON_PRETTY | 80 | 29490 | 26925 | -8.7% | 64323 | 59602 | -7.34% | -1.36pp |
| YAML | 40 | 12184 | 11151 | -8.48% | 30571 | 28324 | -7.35% | -1.13pp |
| YAML | 80 | 24971 | 22763 | -8.84% | 60969 | 56530 | -7.28% | -1.56pp |


**Note:**
- Token Drift % = (optTokens - mandTokens) / mandTokens * 100
- Char Drift % = same for characters 
- Drift Δ (pp) = Token Drift − Char Drift
- Positive Δ = token cost grows more than character count when fields become optional


### 2.4 Format Ranking (averaged across all variants and record counts)

| Rank | Format | Avg Tokens/1K Chars | Avg Chars/Token | Avg Tokens @40 | Avg Tokens @80 |
|---|---|---|---|---|---|
| 1 | **JSON_COMPACT** | 304.36 | 3.28 | 7599 | 14973 |
| 2 | **JSONL** | 310.18 | 3.23 | 7756 | 15282 |
| 3 | **CSV** | 374.81 | 2.67 | 4800 | 9314 |
| 4 | **TOON** | 400.32 | 2.51 | 7961 | 15939.5 |
| 5 | **YAML** | 401.12 | 2.49 | 11667.5 | 23867 |
| 6 | **JSON_PRETTY** | 449.79 | 2.22 | 13804.5 | 28207.5 |


**Note:**
- Ranked by Avg Tokens/1K Chars ascending (lower = more token-efficient overall)


## 3. Conclusion & Decision Matrix

For flat tabular data **CSV** is the best choice on raw token cost. **TOON** with mandatory data is competitive with **CSV** but its adaptive encoding breaks down sharply once optional fields enter the schema. The **JSON** family covers the middle ground with **JSON_COMPACT** and **JSONL** acting as token-efficient structured options and **JSON_PRETTY** as a format to avoid for read-heavy workloads. **YAML** sits in the expensive tier without offering a measurable token advantage over **JSON** variants.


### 3.1 Cross-Configuration Findings

- **TOON**'s adaptive encoding flips between a **CSV**-like layout for mandatory data with uniform fileds and a **YAML**-like layout for optional data with varying fields. In the tabular layout it uses ~14.7 characters per value and in the key-value layout it jumps to ~32.2 characters per value. The two layouts produce a 2.1x token gap on otherwise comparable data.
- The format with the lowest tokens per 1K characters does not produce the lowest absolute token cost. **JSON_COMPACT** tokenizes 23% more efficiently per character than **CSV** (304 vs 374 tokens per 1K chars) but **CSV** files contain roughly half the characters of **JSON_COMPACT** files. **CSV** uses 38-41% fewer tokens overall.
- **CSV**, **JSON_COMPACT**, **JSONL**, **JSON_PRETTY** and **YAML** all drop 7-9% in tokens when data becomes optional while characters drop 5-8% which is a 1-2 percentage point difference. **TOON** instead increases by 116-127% in tokens and 99-102% in characters which is a 17-25 percentage point divergence driven by the encoding switch.
- Scaling from 40 to 80 records is essentially linear with **JSON_PRETTY** and **YAML** tokens slightly growing faster than characters (1.02 to 1.03 ratio) while **CSV**, **JSON_COMPACT** and **JSONL** scale slightly sub-linearly (0.98 to 0.99). The differences are small enough that format choice should not depend on dataset size.
- Per-call latency varies between 1078 ms and 2536 ms but variance within a single format and variant is on the same order as variance between formats. Latency is not a reliable tiebreaker at the file sizes tested.
- **JSONL** and **JSON_COMPACT** behave almost identically but **JSONL** adds a small overhead (~2% more tokens, ~0.1% more characters) from per-record line wrapping. The two formats are interchangeable from a token efficiency standpoint so the choice should be made on streaming or parsing requirements.


### 3.2 Decision Matrix

| Scenario | Recommended Format | Rationale |
|---|---|---|
| Flat structure, dense mandatory fields | **CSV** or **TOON** (mandatory) | Both consume ~9700 tokens at 80 records which is roughly 38% of **JSON_COMPACT** and 33% of **JSON_PRETTY** for the same data |
| Flat structure, sparse optional fields | **CSV** | **CSV** stays cheapest at 4608 to 8956 tokens but **TOON** must be avoided here because adaptive encoding more than doubles its cost |
| Token budget critical, flat structure | **CSV** | Lowest absolute token cost |
| Avoid in flat structure | **JSON_PRETTY** and **TOON** (optional) | **JSON_PRETTY** costs ~3x **CSV** with no observed benefit and **TOON**-optional doubles relative to **TOON**-mandatory due to the encoding fallback |


### 3.3 Real-World Impact

For a single read the token costs span almost a factor of three.

| Format (mandatory, 80 records) | Read tokens | Multiplier vs CSV |
|---|---|---|
| CSV | 9672 | 1.00x |
| TOON | 9760 | 1.01x |
| JSON_COMPACT | 15661 | 1.62x |
| JSONL | 15974 | 1.65x |
| YAML | 24971 | 2.58x |
| JSON_PRETTY | 29490 | 3.05x |

- Format choice directly determines how much data the model can hold in context. A 200K context budget fits roughly 20 reads of a 10K token **CSV** file but only 6 reads of the equivalent **JSON_PRETTY** file. Furthermore as the token count increases the model becomes more likely to get lost in the middle and start hallucinating data.
- A team benchmarking **TOON** on a dense schema will see **CSV**-like efficiency and might standardize on it. However should the schema gain even a single optional field the per-read cost can more than double without any code changes. Teams considering **TOON** should either commit to dense schemas or rigorously test both encoder branches before adoption.
- With read durations ranging from 1.0 to 2.5 seconds per file the latency differences are too insignificant to justify a format recommendation based on speed alone. Instead the format choice should be driven by token efficiency.


## 4. Appendix

- **Test Date**: 2025-12-19
- **Model**: Claude Haiku 4.5
- **Thinking**: on
- **Structure**: flat
- **Formats Tested**: CSV, JSON_COMPACT, JSON_PRETTY, JSONL, TOON, YAML
- **Record Counts**: 40 & 80

---

- **Report Generated**: 2026-05-04
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code 2.0.74
- **Data Source**: `metrics.json` & `metadata.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/LICENSE)
- **Other Benchmark Reports**:
   - [Report - Haiku 4.5 Summary](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_haiku_4_5/Benchmark_Report_Summary.md)
   - [Report - Sonnet 4.6 Summary](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/blob/develop/benchmark_sonnet_4_6/Benchmark_Report_Summary.md)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)