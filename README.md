# File Format Token Accuracy Benchmark Results

Archive of benchmark results measuring token usage and retrieval accuracy across file formats for LLM consumption to find the most efficient format.

## Overview

This repository contains **benchmark results and raw data** from experiments evaluating which file formats deliver the most reliable information to LLMs with optimal token efficiency.

**To run your own benchmarks** see the [benchmark plugin repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark).

> [!NOTE]
>
> These benchmark results are specific to Claude Code using the Haiku 4.5 and Sonnet 4.6 model. They serve as a rule of thumb for choosing the best file format depending on the use case.
> However these values cannot be exactly applied to models of the same family or from other providers as token usage, accuracy and latency depend on specific model architectures and tokenizers. Also file reads will produce different characters depending on the used harness because some add marker characters, line numbers or additional information. While the relative ranking of file formats remains consistent the absolute numbers will vary.
> Especially the accuracy and output tokens results will vary because these values are bound to the model size and training, instruction interpretation and reasoning token budget.


## Initial Benchmark: Haiku 4.5 — Flat Structure, Thinking On

- **Date**: 2025-12-19
- **Model**: Claude Haiku 4.5
- **Thinking**: on
- **Structure**: flat
- **Tested Formats**: 6 (CSV, JSON Compact/Pretty, JSONL, TOON, YAML)
- **Data Variants**: Mandatory (22 fields, dense) and optional (19 mandatory + 3 optional, sparse), 40 & 80 records

### Key Findings (Summary)

| Format (mandatory, 80 records) | Read Tokens | vs CSV |
|---|---|---|
| **CSV** | 9 672 | 1.00x |
| **TOON** | 9 760 | 1.01x |
| **JSON_COMPACT** | 15 661 | 1.62x |
| **JSONL** | 15 974 | 1.65x |
| **YAML** | 24 971 | 2.58x |
| **JSON_PRETTY** | 29 490 | 3.05x |

- **CSV** is cheapest in absolute read tokens for flat mandatory data.
- **TOON** matches **CSV** on dense mandatory fields but balloons +127% when optional fields trigger its key-value fallback encoding.
- **JSON_COMPACT** and **JSONL** are interchangeable (~2% gap) and drop 7–9% in tokens when fields become sparse which is the similar to other formats except **TOON**.
- Token cost scales near linearly with record count (1.94–2.05×) across all formats.

**See [Full Report](./initital_benchmark_haiku_4_5_formats_all_variants_all_thinking_on/BENCHMARK_REPORT.md) for detailed token tables and scaling analysis.**

---

## Benchmark Results: Haiku 4.5 — Flat & Nested, Thinking On & Off

- **Date**: 2026-03-18
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: on & off
- **Structure**: flat & nested
- **Tested Formats**: 7 (CSV, JSON Compact/Pretty, TOON Default, XML Compact/Pretty, YAML)
- **Data Variants**: Mandatory (22 fields, dense) and optional (19 mandatory + 3 optional, sparse), 31 records

### Format Recommendations (Summary)

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

**See [Benchmark Report Summary](./benchmark_haiku_4_5/Benchmark_Report_Summary.md) for the full analysis.**

---

## Benchmark Results: Sonnet 4.6 — Flat & Nested, Thinking On & Off

- **Date**: 2026-03-22 & 2026-03-29
- **Model**: Claude Sonnet 4.6
- **Thinking**: on & off
- **Structure**: flat & nested
- **Tested Formats**: 8 (CSV, JSON Compact/Pretty, TOON Default/Keyfold, XML Compact/Pretty, YAML)
- **Data Variants**: Mandatory (22 fields, dense) and optional (19 mandatory + 3 optional, sparse), 31 records

### Format Recommendations (Summary)

| Scenario | Recommended Format | Eff. Score Total | Accuracy | Rationale |
|---|---|---|---|---|
| Flat structure, dense mandatory fields | **TOON_DEFAULT** (thinking on) | 95.67 | 98.92% | Matches CSV on read tokens (~3 965) with lowest flat total tokens (26 023) |
| Flat structure, sparse optional fields | **JSON_COMPACT** (thinking on) | 98.36 | 97.58% | TOON balloons to 11 320 read tokens on sparse data; JSON_COMPACT stays at 5 669 |
| Nested structure, dense mandatory fields | **TOON_DEFAULT** (thinking on) | 94.86 | 99.73% | Highest benchmark accuracy + lowest nested total tokens (32 454) |
| Nested structure, sparse optional fields | **JSON_COMPACT** (thinking on) | 98.54 | 97.85% | Only format that stays compact and accurate under both sparsity and nesting |
| Maximum accuracy, flat structure | **JSON_PRETTY** (thinking off) | 86.89 | 99.46% | Highest measured flat accuracy at the cost of ~31 800 total tokens |
| Maximum accuracy, nested structure | **TOON_DEFAULT** (thinking on) | 94.86 | 99.73% | Overall benchmark accuracy ceiling with competitive token usage |
| Token budget critical, flat structure | **CSV** (thinking on) | 89.26 | 97.58% | Lowest flat read tokens (3 922) with only a small accuracy gap |
| Token budget critical, nested structure | **JSON_COMPACT** (thinking on) | 98.54 | 97.85% | Lowest nested read tokens (6 970) without sacrificing robustness |
| Avoid in flat structure | **XML_COMPACT** / **XML_PRETTY** | 66–78 | 97–99% | 2–3× the read tokens of compact alternatives with no accuracy benefit |
| Avoid in nested structure | **XML_PRETTY** | 64–69 | 97–98% | Consistently last across all nested runs with 17 760–18 291 read tokens |

**See [Benchmark Report Summary](./benchmark_sonnet_4_6/Benchmark_Report_Summary.md) for the full analysis.**

## Repository Structure

### Benchmark Runs

Each benchmark run directory contains:

```
benchmark/
├── data/                            # Raw test data files in all formats
│   └── {format}/
│       ├── *.csv, *.json, *.yaml    # Data files tested
├── questions/                       # Test question sets
├── answers_validation/              # Expected answers for validation
├── subagent_outputs/                # Raw agent responses (3 runs per variant)
├── results/                         # Validation results per format
├── metadata.json                    # Dataset characteristics
├── metrics.json                     # Token and accuracy metrics
├── analytics_results.json           # Final rankings and insights
└── BENCHMARK_REPORT.md              # Comprehensive analysis and findings
```

**Key Files per Benchmark**:
- `data/` - Raw data files used in testing
- `subagent_outputs/` - Raw LLM responses for reproducibility
- `metadata.json` - Dataset characteristics and generation parameters
- `agent_ids.json` - Contains testConfiguration (formats, variants, model, thinking) and all readonly and full test agent IDs for metrics extraction
- `metrics.json` - Combines input and output metrics extracted from agent transcripts
- `analytics_results.json` - Final analysis output with efficiency rankings and key insights
- `BENCHMARK_REPORT.md` - Complete analysis, recommendations, and methodology

## Running Benchmarks

To generate new benchmark results:

1. Clone the [benchmark plugin repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)
2. Follow setup instructions there
3. Run `/benchmark` command with desired parameters
4. Results are generated in the specified output directory

## Data File Structure

Each benchmark generates standardized test data:

- **31 records** (standardized across benchmarks)
- **22 fields** (19 mandatory + 3 optional)
- **Product dataset** (consistent across formats for fair comparison)
- **Multiple variants**:
  - Flat and nested structures
  - Mandatory fields only
  - Sparse data (optional fields)

All data is formatted identically in each file format to ensure apples-to-apples comparison.

### Formats Tested

| Format | Description |
|--------|-------------|
| **CSV** | Comma-separated values (tabular, row-per-record) |
| **JSON Pretty** | Standard indented JSON |
| **JSON Compact** | Minified JSON (no whitespace or newlines) |
| **XML Pretty** | Standard indented XML |
| **XML Compact** | Minified XML (no whitespace or indentation) |
| **TOON Default** | [Token-Oriented Object Notation v2.1.0](https://toonformat.dev/) with key folding disabled which expands the nested structures fully (default) |
| **TOON Keyfold** | [Token-Oriented Object Notation v2.1.0](https://toonformat.dev/) with key folding enabled which collapses single-key object chains into dotted paths (`a.b.c: value`) |
| **YAML** | Standard YAML with hierarchical indentation |

*Note: All formats are tested in both flat and nested structures except CSV which is flat only.*

Format implementation details can be found in ["Format Specifics" section in the benchmark plugin README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics).

## Methodology

### Evaluation Criteria

1. **Token Efficiency** - Total tokens consumed per format
2. **Information Accuracy** - Weighted accuracy (66.7% retrieval/structure, 33.3% filtering/aggregation)
3. **Consistency** - Performance across data variants (mandatory vs optional fields)
4. **Robustness** - Scaling behavior with data volume

### Question Weighting

- **Field Retrieval (37.5%)** - Extract specific values
- **Structure Awareness (29.2%)** - Understand data organization
- **Filtering (20.8%)** - Count matching criteria
- **Aggregation (12.5%)** - Sum/average across records

Weighted accuracy prioritizes **understanding data organization** over model reasoning capability.

---

## License

See root [LICENSE](./LICENSE) for details.

## Related

- **Benchmark Plugin**: [file-format-token-accuracy-benchmark](https://github.com/thoeltig/file-format-token-accuracy-benchmark)
- **Author**: [Thore Höltig](https://github.com/thoeltig)