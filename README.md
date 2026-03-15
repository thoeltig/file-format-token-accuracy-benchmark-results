# File Format Token Accuracy Benchmark Results

Archive of benchmark results measuring token usage and retrieval accuracy across file formats for LLM consumption to find the most efficient format.

## Overview

This repository contains **benchmark results and raw data** from experiments evaluating which file formats deliver the most reliable information to LLMs with optimal token efficiency.

**To run your own benchmarks**, see the [benchmark plugin repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark).

## Initial Benchmark Results

- **Date**: December 19, 2025
- **Model**: Claude 4.5 Haiku
- **Extended Thinking**: Off
- **Tested Formats**: 7 (CSV, JSON Compact/Pretty, JSONL, TOON, Markdown, YAML)
- **Data Variants**: Mandatory and optional fields, 40 & 80 record datasets

### Key Findings

- **CSV**: Unbeatable for dense, mandatory data (70.98% weighted accuracy @ 9,008 tokens). Accuracy collapses ~15% with sparse data.
- **JSON Compact**: Recommended baseline (70.12% weighted accuracy, 15,957 tokens). Only format that uses fewer tokens with optional fields.
- **YAML**: Highest accuracy (71.96% weighted) but at 2.62x token cost compared to CSV.
- **Markdown**: Catastrophic failure (24.67% weighted accuracy despite token efficiency). Unreliable format.
- **TOON**: Competitive on dense data but token cost explodes 2.17x with optional fields.

### Format Recommendations (Summary)

| Scenario | Format | Notes |
|----------|--------|-------|
| **Default choice** | JSON Compact | Best balance of accuracy and token efficiency |
| **Dense, complete data** | CSV | Unbeatable tokens/accuracy ratio (9,008 tokens @ 71% weighted) |
| **Maximum accuracy** | YAML | 71.96% weighted, but 2.62x token cost vs CSV |
| **Sparse/optional fields** | JSON Compact | Only format reducing tokens with optional data |
| **Never use** | Markdown | 24.67% accuracy = 76% token waste |

**See [Full Report](./initital_benchmark_haiku_4_5_formats_all_variants_all_extended_thinking_off/BENCHMARK_REPORT.md) for detailed decision framework and trade-off analysis.**

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
- `metadata.json`: Dataset characteristics and generation parameters
- `agent_ids.json`: Contains testConfiguration (formats, variants, model, thinking) and all readonly and full test agent IDs for metrics extraction
- `metrics.json`: Combines input and output metrics extracted from agent transcripts
- `analytics_results.json`: Final analysis output with efficiency rankings and key insights
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
| **TOON Unsafe** | [Token-Oriented Object Notation](https://toonformat.dev/) with key folding disabled which expands the nested structures fully (default) |
| **TOON Safe** | [Token-Oriented Object Notation](https://toonformat.dev/) with key folding enabled which collapses single-key object chains into dotted paths (`a.b.c: value`) |
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