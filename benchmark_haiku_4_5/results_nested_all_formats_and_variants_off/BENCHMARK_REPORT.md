# File Format Token Efficiency Benchmark: Comprehensive Report
- **Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: off
- **Data Structure**: nested
- **Formats Tested**: 6 (JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML)
- **Record Counts**: 31
- **Status**: First iteration

## Executive Summary

This benchmark evaluates token efficiency and information accuracy across 6 file formats using Claude Haiku 4.5 (claude-haiku-4-5-20251001) as the inference model. The research addresses a critical but underexplored problem: **not all tokens are equally useful**. A format that uses fewer tokens but produces inaccurate results wastes both tokens and context while a format that accurately conveys information may justify higher token cost.

### Key Findings

1. **JSON_COMPACT** delivers the best overall balance of token efficiency and accuracy. It achieves the lowest read token cost (9645 to 10163 tokens) alongside the highest optional accuracy (78.07%) and a strong mandatory accuracy (75.97%) which makes it the most cost-effective format for nested data with Haiku 4.5.

2. **XML_COMPACT** dominates mandatory efficiency with the lowest total token count (18000) and the highest efficiency score (83.41) but its performance degrades significantly on optional data where output tokens increase by 75% and aggregation accuracy drops by over 30 percentage points.

3. **YAML** is the least robust format despite achieving the highest mandatory accuracy (77.42%) because its accuracy collapses by nearly 10 percentage points on optional data while read tokens stay virtually unchanged which indicates the model struggles to interpret sparse **YAML** structures even when the token cost remains stable.

4. Pretty-printed formats consistently underperform their compact counterparts in token efficiency without providing a measurable accuracy benefit. **XML_PRETTY** consumes roughly double the read tokens of **JSON_COMPACT** and ranks last in efficiency scores across both variants which demonstrates that additional whitespace and indentation actively harms the information-per-token ratio.

5. Aggregation is universally the weakest question category with accuracy ranging from 38% to 68% and every single format showing a decline from mandatory to optional data. This suggests Haiku 4.5 struggles with numerical computations on nested structures regardless of format and sparse data amplifies this weakness.

6. Field retrieval accuracy remains strong across all formats (82% to 99%) which confirms that the model reliably extracts specific values from nested data. **YAML** leads on mandatory data with 99.39% but drops to 82.42% on optional data which is the steepest decline of any format in this category.

7. Token cost and accuracy show no positive correlation. **XML_PRETTY** is the most expensive format in read tokens but consistently ranks last in accuracy while **JSON_COMPACT** is the cheapest and ranks first or second. This directly validates the report's core premise that not all tokens are equally useful and that cheaper formats can simultaneously be more accurate.

## 1. Methodology

### 1.1 Research Purpose

The underlying question: **Which file format delivers maximum information value per token consumed?**

This requires measuring:
- **Token Cost**: How many tokens does each format consume for equivalent data?
- **Information Fidelity**: How accurately can the model understand and answer questions about the data?
- **Robustness**: How consistent is performance across data variants (mandatory vs optional fields)?

### 1.2 Test Design

#### 1.2.1 Data Generation
- 6 formats tested: JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML
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
   - Optional: JSON_COMPACT 21434 tokens
   - Mandatory: XML_COMPACT 18000 tokens
- Lowest read token cost:
   - Optional: JSON_COMPACT 9645 tokens
   - Mandatory: JSON_COMPACT 10163 tokens
- Lowest output token cost:
   - Optional: YAML 8089 tokens
   - Mandatory: XML_COMPACT 5295 tokens
- Lowest output token cost drift:
   - Optional: YAML ↓ -19.23% ↑ 12.49%
   - Mandatory: JSON_COMPACT ↓ -13.38% ↑ 19.90%
- Highest accuracy:
   - Optional: JSON_COMPACT 78.07%
   - Mandatory: YAML 77.42%
- Lowest accuracy drift:
   - Optional: YAML ↓ -3.19% ↑ 2.79%
   - Mandatory: YAML ↓ -2.08% ↑ 3.13%
- Most useful read tokens:
   - Optional: XML_PRETTY 14014 / 19671 tokens
   - Mandatory: XML_PRETTY 14230 / 20204 tokens
- Most useful output tokens:
   - Optional: TOON_DEFAULT 9208 / 11912 tokens
   - Mandatory: YAML 7766 / 10031 tokens
- Highest read efficiency (%/token):
   - Optional: JSON_COMPACT 84.62
   - Mandatory: JSON_COMPACT 81.68
- Highest output efficiency (%/token):
   - Optional: XML_PRETTY 51.41
   - Mandatory: XML_COMPACT 74.73
- Lowest delta (optional-mandatory):
   - Read tokens: YAML -7 tokens
   - Output tokens: XML_PRETTY -1643 tokens
   - Accuracy: JSON_PRETTY 0.00%
   - Read efficiency: XML_COMPACT -0.42
   - Output efficiency: YAML -0.32

#### 2.1.2 Worst results

- Highest total token cost:
   - Optional: JSON_PRETTY 28491 tokens
   - Mandatory: XML_PRETTY 30149 tokens
- Highest read token cost:
   - Optional: XML_PRETTY 19671 tokens
   - Mandatory: XML_PRETTY 20204 tokens
- Highest output token cost:
   - Optional: TOON_DEFAULT 11912 tokens
   - Mandatory: YAML 10031 tokens
- Highest output token drift:
   - Optional: XML_PRETTY ↓ -40.86% ↑ 78.57%
   - Mandatory: XML_COMPACT ↓ -94.24% ↑ 80.26%
- Lowest accuracy:
   - Optional: YAML 67.47%
   - Mandatory: XML_PRETTY 70.43%
- Highest accuracy drift:
   - Optional: XML_PRETTY ↓ -9.43% ↑ 15.47%
   - Mandatory: XML_COMPACT ↓ -11.27% ↑ 11.97%
- Most wasted read tokens:
   - Optional: XML_PRETTY 5657 / 19671 tokens
   - Mandatory: XML_PRETTY 5974 / 20204 tokens
- Most wasted output tokens:
   - Optional: JSON_PRETTY 3029 / 11734 tokens
   - Mandatory: XML_PRETTY 2941 / 9945 tokens
- Lowest read efficiency (%/token):
   - Optional: XML_PRETTY 51.41
   - Mandatory: XML_PRETTY 49.33
- Lowest output efficiency (%/token):
   - Optional: JSON_PRETTY 59.87
   - Mandatory: XML_PRETTY 63.36
- Highest delta (optional-mandatory):
   - Read tokens: JSON_PRETTY -925 tokens
   - Output tokens: XML_COMPACT 3993 tokens
   - Accuracy: YAML -9.95%
   - Read efficiency: YAML -6.95
   - Output efficiency: XML_COMPACT -14.78

#### 2.1.3 Format Ranking

##### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 77s | JSON_COMPACT ≈ 10163 | YAML ≈ 213 | XML_COMPACT ≈ 4990 | XML_COMPACT ≈ 5295 | XML_COMPACT ≈ 18000 | YAML ≈ 77% | JSON_COMPACT ≈ 82 | XML_COMPACT ≈ 83 | XML_COMPACT ≈ 83 |
| XML_COMPACT (+3.9%) | XML_COMPACT (+25.0%) | XML_PRETTY (+42.1%) | TOON_DEFAULT (+72.5%) | TOON_DEFAULT (+68.4%) | JSON_COMPACT (+11.6%) | XML_COMPACT (-1.1%) | XML_COMPACT (-8.5%) | TOON_DEFAULT (-15.2%) | JSON_COMPACT (-6.5%) |
| XML_PRETTY (+6.1%) | TOON_DEFAULT (+38.4%) | XML_COMPACT (+43.4%) | JSON_PRETTY (+79.1%) | JSON_PRETTY (+74.6%) | TOON_DEFAULT (+27.7%) | JSON_COMPACT (-1.5%) | YAML (-12.6%) | JSON_PRETTY (-18.0%) | TOON_DEFAULT (-15.1%) |
| JSON_COMPACT (+6.6%) | YAML (+39.3%) | JSON_COMPACT (+43.5%) | JSON_COMPACT (+92.9%) | JSON_COMPACT (+87.5%) | YAML (+34.4%) | TOON_DEFAULT (-1.5%) | TOON_DEFAULT (-13.6%) | YAML (-18.5%) | YAML (-17.4%) |
| TOON_DEFAULT (+7.9%) | JSON_PRETTY (+74.0%) | JSON_PRETTY (+44.2%) | XML_PRETTY (+93.2%) | XML_PRETTY (+87.8%) | JSON_PRETTY (+49.6%) | JSON_PRETTY (-3.2%) | JSON_PRETTY (-27.6%) | JSON_COMPACT (-19.3%) | JSON_PRETTY (-28.2%) |
| YAML (+11.8%) | XML_PRETTY (+98.8%) | TOON_DEFAULT (+45.3%) | YAML (+96.8%) | YAML (+89.4%) | XML_PRETTY (+67.5%) | XML_PRETTY (-7.0%) | XML_PRETTY (-39.6%) | XML_PRETTY (-24.0%) | XML_PRETTY (-40.9%) |


##### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 67s | JSON_COMPACT ≈ 9645 | XML_PRETTY ≈ 205 | YAML ≈ 7786 | YAML ≈ 8089 | JSON_COMPACT ≈ 21434 | JSON_COMPACT ≈ 78% | JSON_COMPACT ≈ 85 | XML_PRETTY ≈ 70 | JSON_COMPACT ≈ 76 |
| XML_PRETTY (+2.2%) | XML_COMPACT (+29.1%) | TOON_DEFAULT (+25.7%) | XML_PRETTY (+4.0%) | XML_PRETTY (+2.6%) | XML_COMPACT (+1.4%) | TOON_DEFAULT (-0.8%) | XML_COMPACT (-12.2%) | XML_COMPACT (-1.3%) | XML_COMPACT (-4.1%) |
| XML_COMPACT (+13.5%) | TOON_DEFAULT (+43.4%) | YAML (+47.6%) | XML_COMPACT (+15.3%) | XML_COMPACT (+14.8%) | YAML (+3.7%) | XML_COMPACT (-3.3%) | TOON_DEFAULT (-14.7%) | YAML (-2.7%) | YAML (-12.3%) |
| JSON_PRETTY (+39.9%) | YAML (+46.7%) | JSON_COMPACT (+48.7%) | JSON_PRETTY (+46.8%) | JSON_PRETTY (+45.1%) | TOON_DEFAULT (+20.1%) | JSON_PRETTY (-3.9%) | YAML (-23.9%) | JSON_COMPACT (-10.3%) | TOON_DEFAULT (-14.7%) |
| JSON_COMPACT (+40.8%) | JSON_PRETTY (+73.7%) | JSON_PRETTY (+48.8%) | JSON_COMPACT (+47.5%) | JSON_COMPACT (+45.8%) | XML_PRETTY (+30.5%) | XML_PRETTY (-6.8%) | JSON_PRETTY (-27.0%) | TOON_DEFAULT (-11.7%) | JSON_PRETTY (-26.4%) |
| TOON_DEFAULT (+41.0%) | XML_PRETTY (+104.0%) | XML_COMPACT (+49.8%) | TOON_DEFAULT (+49.7%) | TOON_DEFAULT (+47.3%) | JSON_PRETTY (+32.9%) | YAML (-10.6%) | XML_PRETTY (-39.2%) | JSON_PRETTY (-13.9%) | XML_PRETTY (-27.4%) |


#### 2.1.4 Category Accuracy Ranking

##### Mandatory

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| YAML ≈ 99% | YAML ≈ 70% | YAML ≈ 62% | XML_COMPACT ≈ 68% |
| JSON_COMPACT (-3.0%) | JSON_PRETTY (-0.7%) | JSON_PRETTY (-1.9%) | JSON_COMPACT (-8.3%) |
| TOON_DEFAULT (-5.8%) | XML_COMPACT (-1.2%) | JSON_COMPACT (-3.8%) | TOON_DEFAULT (-10.5%) |
| JSON_PRETTY (-7.8%) | TOON_DEFAULT (-2.3%) | TOON_DEFAULT (-4.2%) | XML_PRETTY (-19.0%) |
| XML_COMPACT (-8.5%) | XML_PRETTY (-7.4%) | XML_COMPACT (-6.3%) | JSON_PRETTY (-19.7%) |
| XML_PRETTY (-10.9%) | JSON_COMPACT (-9.6%) | XML_PRETTY (-7.9%) | YAML (-23.8%) |


##### Optional

| ↓ Field Retrieval | ↓ Structure Awareness | ↓ Filtering | ↓ Aggregation |
|---|---|---|---|
| JSON_COMPACT ≈ 98% | TOON_DEFAULT ≈ 78% | JSON_PRETTY ≈ 68% | JSON_COMPACT ≈ 53% |
| JSON_PRETTY (-2.4%) | XML_COMPACT (-4.9%) | TOON_DEFAULT (-1.6%) | TOON_DEFAULT (-8.4%) |
| XML_COMPACT (-3.6%) | JSON_COMPACT (-10.4%) | JSON_COMPACT (-4.4%) | XML_PRETTY (-12.1%) |
| TOON_DEFAULT (-4.7%) | XML_PRETTY (-11.1%) | XML_PRETTY (-6.3%) | YAML (-13.6%) |
| XML_PRETTY (-9.7%) | YAML (-14.8%) | YAML (-6.3%) | JSON_PRETTY (-13.7%) |
| YAML (-15.8%) | JSON_PRETTY (-16.1%) | XML_COMPACT (-6.4%) | XML_COMPACT (-15.2%) |


#### 2.1.5 Conclusion

**JSON_COMPACT** is the clear winner for nested data with Haiku 4.5 when thinking is disabled. It ranks first in read token efficiency across both variants. It achieves the highest optional accuracy (78.07%) and delivers the best overall efficiency score for optional data (76.16). Its compact encoding packs the most characters per read token (2.2+) which translates directly into higher information density and fewer wasted tokens.

**XML_COMPACT** is a strong contender for mandatory-only workloads where it achieves the highest efficiency score (83.41) thanks to remarkably low output tokens (5295) but this advantage disappears with optional data as output tokens nearly double and accuracy slightly declines. Any system that may encounter sparse or optional fields should avoid relying on **XML_COMPACT**'s mandatory-only performance.

**YAML** presents a cautionary result. It achieves top mandatory accuracy (77.42%) and the lowest accuracy drift on mandatory data but it suffers the largest accuracy collapse when optional fields are introduced (nearly 10 percentage points). This fragility makes **YAML** a risky choice for production workloads where data completeness varies despite its human-readable appeal.

Pretty-printed formats (**JSON_PRETTY** and **XML_PRETTY**) offer no accuracy advantage over their compact equivalents while consuming 60% to 100% more read tokens. **JSON_PRETTY** maintains perfectly stable accuracy across variants (74.19% both) but at a substantial token cost premium. **XML_PRETTY** consistently ranks last in both efficiency scores and accuracy which makes it the worst overall choice.

The category-level analysis reveals that all formats struggle with aggregation tasks (38% to 68% accuracy) and that every format shows declining aggregation accuracy on optional data. This performance floor appears to be a model-level limitation of Haiku 4.5 on nested structures rather than a format-specific issue since even the best-performing format (**XML_COMPACT** at 68.25% mandatory) drops to 38.10% on optional data. Field retrieval remains the strongest category across all formats (82% to 99%) which confirms that the model can reliably parse and extract values from nested structures regardless of serialization format.

For practical applications using Haiku 4.5 with nested data and thinking disabled **JSON_COMPACT** should be the default choice. It provides the lowest read token cost, competitive accuracy, strong robustness across data variants and the best information value per token. Only in mandatory-only scenarios with tight token budgets does **XML_COMPACT** offer a meaningful alternative due to its exceptionally low output token count.

### 2.2 Comprehensive Benchmark Metrics
| Format | Variant | Read Tokens | Output Tokens | Total Tokens | Char / Read Token | Output Write Tokens / Answer | Accuracy (%) | Useful Read Tokens | Wasted Read Tokens | Useful Output Tokens | Wasted Output Tokens | Eff Score Read | Eff Score Output | Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10163 | 9930 | 20093 | 2.246 | 77.618 | 75.97 | 7720.831 | 2442.169 | 7543.669 | 2386.131 | 81.68 | 67.29 | 78.00 |
| JSON_COMPACT | opt | 9645 | 11789 | 21434 | 2.219 | 92.618 | 78.07 | 7529.852 | 2115.149 | 9203.985 | 2585.415 | 84.62 | 62.40 | 76.16 |
| JSON_PRETTY | man | 17682 | 9243 | 26925 | 1.777 | 72.069 | 74.19 | 13118.276 | 4563.724 | 6857.530 | 2385.670 | 59.11 | 68.39 | 59.91 |
| JSON_PRETTY | opt | 16757 | 11734 | 28491 | 1.767 | 92.172 | 74.19 | 12432.018 | 4324.982 | 8705.702 | 3028.631 | 61.74 | 59.87 | 56.04 |
| TOON_DEFAULT | man | 14068 | 8915 | 22983 | 1.855 | 69.401 | 75.91 | 10679.019 | 3388.981 | 6767.174 | 2147.559 | 70.57 | 70.72 | 70.83 |
| TOON_DEFAULT | opt | 13832 | 11912 | 25744 | 1.864 | 93.986 | 77.30 | 10692.136 | 3139.864 | 9207.912 | 2704.005 | 72.21 | 61.44 | 64.99 |
| XML_COMPACT | man | 12705 | 5295 | 18000 | 2.551 | 40.242 | 76.34 | 9698.997 | 3006.003 | 4042.203 | 1252.797 | 74.73 | 83.40 | 83.41 |
| XML_COMPACT | opt | 12455 | 9288 | 21743 | 2.499 | 72.423 | 74.73 | 9307.622 | 3147.378 | 6940.549 | 2346.951 | 74.31 | 68.62 | 73.06 |
| XML_PRETTY | man | 20204 | 9945 | 30149 | 1.985 | 77.762 | 70.43 | 14229.677 | 5974.323 | 7004.087 | 2940.663 | 49.33 | 63.36 | 49.33 |
| XML_PRETTY | opt | 19671 | 8301 | 27972 | 1.974 | 65.293 | 71.24 | 14013.620 | 5657.380 | 5913.870 | 2387.463 | 51.41 | 69.55 | 55.26 |
| YAML | man | 14155 | 10031 | 24186 | 1.808 | 79.183 | 77.42 | 10958.801 | 3196.199 | 7766.258 | 2265.075 | 71.38 | 67.96 | 68.92 |
| YAML | opt | 14148 | 8089 | 22237 | 1.787 | 62.790 | 67.47 | 9545.656 | 4602.344 | 5457.424 | 2631.243 | 64.43 | 67.64 | 66.76 |

### 2.3 Format Robustness: Mandatory vs Optional
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Output Before Write Tokens Man | Output Before Write Tokens Opt | Diff | Diff (%) | Output Write Tokens Man | Output Write Tokens Opt | Diff | Diff (%) | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10163 | 9645 | -518 | -5.10 | 305 | 305 |  +1860 |  +609.84 | 9625 | 11485 |  +1860 |  +19.32 | 9930 | 11790 |  +1860 |  +18.73 | 20093 | 21435 |  +1342 |  +6.68 |
| JSON_PRETTY | 17682 | 16757 | -925 | -5.23 | 307 | 305 |  +2493 |  +812.05 | 8937 | 11430 |  +2491 |  +27.87 | 9243 | 11734 |  +2491 |  +26.95 | 26925 | 28491 |  +1566 |  +5.82 |
| TOON_DEFAULT | 14068 | 13832 | -236 | -1.68 | 309 | 258 |  +3049 |  +986.73 | 8606 | 11655 |  +2997 |  +34.82 | 8915 | 11912 |  +2997 |  +33.62 | 22983 | 25744 |  +2761 |  +12.01 |
| XML_COMPACT | 12705 | 12455 | -250 | -1.97 | 305 | 307 |  +3991 |  +1308.52 | 4990 | 8981 |  +3993 |  +80.02 | 5295 | 9288 |  +3993 |  +75.41 | 18000 | 21743 |  +3743 |  +20.79 |
| XML_PRETTY | 20204 | 19671 | -533 | -2.64 | 302 | 205 | -1546 | -511.92 | 9643 | 8097 | -1643 | -17.04 | 9945 | 8302 | -1643 | -16.52 | 30149 | 27973 | -2176 | -7.22 |
| YAML | 14155 | 14148 | -7 | -0.05 | 213 | 303 | -2033 | -954.46 | 9819 | 7786 | -1943 | -19.79 | 10031 | 8088 | -1943 | -19.37 | 24186 | 22236 | -1950 | -8.06 |

### 2.4 Performance
#### 2.4.1 Metrics
| Format | Variant | Read (ms) | Read (tokens/ms) | Rate (ms/record) | Output Write (ms) | Output Write (tokens/ms) | Rate (ms/question) | Read + Output Write (ms) | Read + Output Write (tokens/ms) | Rate (ms/record+question) |
|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 13 | 781.769 | 0.42 | 35860 | 0.268 | 289.20 | 35873 | 782.037 | 231.44 |
| JSON_COMPACT | opt | 9 | 1071.667 | 0.29 | 38516 | 0.298 | 310.61 | 38525 | 1071.965 | 248.55 |
| JSON_PRETTY | man | 262 | 67.489 | 8.45 | 37358 | 0.239 | 301.27 | 37620 | 67.728 | 242.71 |
| JSON_PRETTY | opt | 270 | 62.063 | 8.71 | 37045 | 0.309 | 298.75 | 37315 | 62.372 | 240.74 |
| TOON_DEFAULT | man | 31 | 453.806 | 1.00 | 38540 | 0.229 | 310.81 | 38571 | 454.035 | 248.85 |
| TOON_DEFAULT | opt | 29 | 476.966 | 0.94 | 37891 | 0.321 | 305.58 | 37920 | 477.287 | 244.65 |
| XML_COMPACT | man | 13 | 977.308 | 0.42 | 41086 | 0.121 | 331.34 | 41099 | 977.429 | 265.16 |
| XML_COMPACT | opt | 12 | 1037.917 | 0.39 | 40983 | 0.219 | 330.50 | 40995 | 1038.136 | 264.48 |
| XML_PRETTY | man | 13 | 1554.154 | 0.42 | 38004 | 0.254 | 306.49 | 38017 | 1554.408 | 245.27 |
| XML_PRETTY | opt | 6 | 3278.500 | 0.19 | 40578 | 0.200 | 327.24 | 40584 | 3278.700 | 261.83 |
| YAML | man | 12 | 1179.583 | 0.39 | 37690 | 0.261 | 303.95 | 37702 | 1179.844 | 243.24 |
| YAML | opt | 11 | 1286.182 | 0.35 | 36048 | 0.216 | 290.71 | 36059 | 1286.398 | 232.64 |

#### 2.4.2 Mandatory vs Optional
| Format | Read Man (ms) | Read Opt (ms) | Diff (ms) | Diff (%) | Output Write Man (s) | Output Write Opt (s) | Diff (s) | Diff (%) | Read + Output Write Man (s) | Read + Output Write Opt (s) | Diff (s) | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 13 | 9 | -4 | -30.77 | 35.86 | 38.52 |  +2.66 |  +7.41 | 35.87 | 38.52 |  +2.65 |  +7.39 |
| JSON_PRETTY | 262 | 270 |  +8 |  +3.05 | 37.36 | 37.05 | -0.31 | -0.84 | 37.62 | 37.32 | -0.30 | -0.81 |
| TOON_DEFAULT | 31 | 29 | -2 | -6.45 | 38.54 | 37.89 | -0.65 | -1.68 | 38.57 | 37.92 | -0.65 | -1.69 |
| XML_COMPACT | 13 | 12 | -1 | -7.69 | 41.09 | 40.98 | -0.10 | -0.25 | 41.10 | 40.99 | -0.10 | -0.26 |
| XML_PRETTY | 13 | 6 | -7 | -53.85 | 38.00 | 40.58 |  +2.57 |  +6.77 | 38.02 | 40.58 |  +2.57 |  +6.75 |
| YAML | 12 | 11 | -1 | -8.33 | 37.69 | 36.05 | -1.64 | -4.36 | 37.70 | 36.06 | -1.64 | -4.36 |

### 2.5 Structural Efficiency
#### 2.5.1 Metrics
| Format | Variant | Chars / Read Token | Read Tokens / Value | Read Tokens / Object | Info / Read Token | Info / Output Token | Info / Total Token |
|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 2.246 | 14.902 | 327.839 | 0.748 | 0.765 | 0.378 |
| JSON_COMPACT | opt | 2.219 | 15.285 | 311.129 | 0.809 | 0.662 | 0.364 |
| JSON_PRETTY | man | 1.777 | 25.927 | 570.387 | 0.420 | 0.803 | 0.276 |
| JSON_PRETTY | opt | 1.767 | 26.556 | 540.548 | 0.443 | 0.632 | 0.260 |
| TOON_DEFAULT | man | 1.855 | 20.628 | 453.806 | 0.540 | 0.876 | 0.332 |
| TOON_DEFAULT | opt | 1.864 | 21.921 | 446.194 | 0.559 | 0.671 | 0.302 |
| XML_COMPACT | man | 2.551 | 18.629 | 409.839 | 0.601 | 1.442 | 0.424 |
| XML_COMPACT | opt | 2.499 | 19.739 | 401.774 | 0.600 | 0.805 | 0.344 |
| XML_PRETTY | man | 1.985 | 29.625 | 651.742 | 0.349 | 0.708 | 0.234 |
| XML_PRETTY | opt | 1.974 | 31.174 | 634.548 | 0.362 | 0.858 | 0.255 |
| YAML | man | 1.808 | 20.755 | 456.613 | 0.547 | 0.772 | 0.320 |
| YAML | opt | 1.787 | 22.422 | 456.387 | 0.477 | 0.834 | 0.303 |

#### 2.5.2 Mandatory vs Optional
| Format | Chars / Read Token Man | Chars / Read Token Opt | Diff | Diff (%) | Read Tokens / Value Man | Read Tokens / Value Opt | Diff | Diff (%) | Read Tokens / Object Man | Read Tokens / Object Opt | Diff | Diff (%) | Info / Read Token Man | Info / Read Token Opt | Diff | Diff (%) | Info / Output Token Man | Info / Output Token Opt | Diff | Diff (%) | Info / Total Token Man | Info / Total Token Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 2.246 | 2.219 | -0.027 | -1.20 | 14.902 | 15.285 |  +0.383 |  +2.57 | 327.839 | 311.129 | -16.710 | -5.10 | 0.748 | 0.809 |  +0.061 |  +8.16 | 0.765 | 0.662 | -0.103 | -13.46 | 0.378 | 0.364 | -0.014 | -3.70 |
| JSON_PRETTY | 1.777 | 1.767 | -0.010 | -0.56 | 25.927 | 26.556 |  +0.629 |  +2.43 | 570.387 | 540.548 | -29.839 | -5.23 | 0.420 | 0.443 |  +0.023 |  +5.48 | 0.803 | 0.632 | -0.171 | -21.30 | 0.276 | 0.260 | -0.016 | -5.80 |
| TOON_DEFAULT | 1.855 | 1.864 |  +0.009 |  +0.49 | 20.628 | 21.921 |  +1.293 |  +6.27 | 453.806 | 446.194 | -7.612 | -1.68 | 0.540 | 0.559 |  +0.019 |  +3.52 | 0.876 | 0.671 | -0.205 | -23.46 | 0.332 | 0.302 | -0.029 | -8.90 |
| XML_COMPACT | 2.551 | 2.499 | -0.052 | -2.04 | 18.629 | 19.739 |  +1.110 |  +5.96 | 409.839 | 401.774 | -8.065 | -1.97 | 0.601 | 0.600 | -0.001 | -0.17 | 1.442 | 0.805 | -0.637 | -44.17 | 0.424 | 0.344 | -0.080 | -18.87 |
| XML_PRETTY | 1.985 | 1.974 | -0.011 | -0.55 | 29.625 | 31.174 |  +1.549 |  +5.23 | 651.742 | 634.548 | -17.194 | -2.64 | 0.349 | 0.362 |  +0.013 |  +3.72 | 0.708 | 0.858 |  +0.150 |  +21.19 | 0.234 | 0.255 |  +0.021 |  +8.97 |
| YAML | 1.808 | 1.787 | -0.021 | -1.16 | 20.755 | 22.422 |  +1.667 |  +8.03 | 456.613 | 456.387 | -0.226 | -0.05 | 0.547 | 0.477 | -0.070 | -12.80 | 0.772 | 0.834 |  +0.062 |  +8.03 | 0.320 | 0.303 | -0.017 | -5.31 |

### 2.6 Token Utilization Efficiency
#### 2.6.1 Metrics
| Format | Variant | Read Tokens | Useful Read Tokens | Wasted Read Tokens | Output Tokens | Useful Output Tokens | Wasted Output Tokens | Total Tokens | Useful Total Tokens | Wasted Total Tokens | Accuracy (%) | Eff Score Read | Eff Score Output | Eff Score Total | Wtd Accuracy (%) | Wtd Eff Score Read | Wtd Eff Score Output | Wtd Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 10163 | 7721 | 2442 | 9930 | 7544 | 2386 | 20093 | 15265 | 4828 | 75.97 | 81.68 | 67.29 | 78.00 | 73.45 | 79.92 | 65.53 | 76.23 |
| JSON_COMPACT | opt | 9645 | 7530 | 2115 | 11789 | 9204 | 2585 | 21434 | 16734 | 4701 | 78.07 | 84.62 | 62.40 | 76.16 | 76.44 | 83.48 | 61.26 | 75.02 |
| JSON_PRETTY | man | 17682 | 13118 | 4564 | 9243 | 6858 | 2386 | 26925 | 19976 | 6949 | 74.19 | 59.11 | 68.39 | 59.91 | 73.24 | 58.45 | 67.73 | 59.24 |
| JSON_PRETTY | opt | 16757 | 12432 | 4325 | 11734 | 8706 | 3029 | 28491 | 21138 | 7354 | 74.19 | 61.74 | 59.87 | 56.04 | 73.09 | 60.97 | 59.10 | 55.27 |
| TOON_DEFAULT | man | 14068 | 10679 | 3389 | 8915 | 6767 | 2148 | 22983 | 17446 | 5537 | 75.91 | 70.57 | 70.72 | 70.83 | 74.21 | 69.38 | 69.53 | 69.64 |
| TOON_DEFAULT | opt | 13832 | 10692 | 3140 | 11912 | 9208 | 2704 | 25744 | 19900 | 5844 | 77.30 | 72.21 | 61.44 | 64.99 | 77.26 | 72.18 | 61.41 | 64.97 |
| XML_COMPACT | man | 12705 | 9699 | 3006 | 5295 | 4042 | 1253 | 18000 | 13741 | 4259 | 76.34 | 74.73 | 83.40 | 83.41 | 74.35 | 73.34 | 82.01 | 82.02 |
| XML_COMPACT | opt | 12455 | 9308 | 3147 | 9288 | 6941 | 2347 | 21743 | 16248 | 5494 | 74.73 | 74.31 | 68.62 | 73.06 | 74.36 | 74.06 | 68.36 | 72.80 |
| XML_PRETTY | man | 20204 | 14230 | 5974 | 9945 | 7004 | 2941 | 30149 | 21234 | 8915 | 70.43 | 49.33 | 63.36 | 49.33 | 68.94 | 48.29 | 62.32 | 48.28 |
| XML_PRETTY | opt | 19671 | 14014 | 5657 | 8301 | 5914 | 2387 | 27972 | 19927 | 8045 | 71.24 | 51.41 | 69.55 | 55.26 | 70.68 | 51.02 | 69.16 | 54.87 |
| YAML | man | 14155 | 10959 | 3196 | 10031 | 7766 | 2265 | 24186 | 18725 | 5461 | 77.42 | 71.38 | 67.96 | 68.92 | 76.25 | 70.56 | 67.14 | 68.10 |
| YAML | opt | 14148 | 9546 | 4602 | 8089 | 5457 | 2631 | 22237 | 15003 | 7234 | 67.47 | 64.43 | 67.64 | 66.76 | 67.13 | 64.19 | 67.40 | 66.52 |

#### 2.6.2 Read Tokens Mandatory vs Optional Data
| Format | Read Tokens Man | Read Tokens Opt | Diff | Diff (%) | Useful Read Tokens Man | Useful Read Tokens Opt | Diff | Diff (%) | Wasted Read Tokens Man | Wasted Read Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Read Man | Eff Score Read Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Read Man | Wtd Eff Score Read Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 10163 | 9645 | -518 | -5.10 | 7721 | 7530 | -191 | -2.47 | 2442 | 2115 | -327 | -13.39 | 75.97 | 78.07 |  +2.10 |  +2.76 | 81.68 | 84.62 |  +2.94 |  +3.60 | 73.45 | 76.44 |  +2.99 |  +4.07 | 79.92 | 83.48 |  +3.56 |  +4.46 |
| JSON_PRETTY | 17682 | 16757 | -925 | -5.23 | 13118 | 12432 | -686 | -5.23 | 4564 | 4325 | -239 | -5.23 | 74.19 | 74.19 | 0.00 | 0.00 | 59.11 | 61.74 |  +2.62 |  +4.44 | 73.24 | 73.09 | -0.15 | -0.20 | 58.45 | 60.97 |  +2.52 |  +4.31 |
| TOON_DEFAULT | 14068 | 13832 | -236 | -1.68 | 10679 | 10692 |  +13 |  +0.12 | 3389 | 3140 | -249 | -7.35 | 75.91 | 77.30 |  +1.39 |  +1.83 | 70.57 | 72.21 |  +1.64 |  +2.33 | 74.21 | 77.26 |  +3.05 |  +4.11 | 69.38 | 72.18 |  +2.80 |  +4.04 |
| XML_COMPACT | 12705 | 12455 | -250 | -1.97 | 9699 | 9308 | -391 | -4.04 | 3006 | 3147 |  +141 |  +4.70 | 76.34 | 74.73 | -1.61 | -2.11 | 74.73 | 74.31 | -0.42 | -0.56 | 74.35 | 74.36 |  +0.01 |  +0.01 | 73.34 | 74.06 |  +0.72 |  +0.98 |
| XML_PRETTY | 20204 | 19671 | -533 | -2.64 | 14230 | 14014 | -216 | -1.52 | 5974 | 5657 | -317 | -5.31 | 70.43 | 71.24 |  +0.81 |  +1.15 | 49.33 | 51.41 |  +2.08 |  +4.21 | 68.94 | 70.68 |  +1.74 |  +2.52 | 48.29 | 51.02 |  +2.73 |  +5.65 |
| YAML | 14155 | 14148 | -7 | -0.05 | 10959 | 9546 | -1413 | -12.89 | 3196 | 4602 |  +1406 |  +44.00 | 77.42 | 67.47 | -9.95 | -12.85 | 71.38 | 64.43 | -6.95 | -9.73 | 76.25 | 67.13 | -9.12 | -11.96 | 70.56 | 64.19 | -6.36 | -9.02 |

#### 2.6.3 Output Tokens Mandatory vs Optional Data
| Format | Output Tokens Man | Output Tokens Opt | Diff | Diff (%) | Useful Output Tokens Man | Useful Output Tokens Opt | Diff | Diff (%) | Wasted Output Tokens Man | Wasted Output Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Output Man | Eff Score Output Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Output Man | Wtd Eff Score Output Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 9930 | 11790 |  +1860 |  +18.73 | 7544 | 9204 |  +1660 |  +22.01 | 2386 | 2585 |  +199 |  +8.35 | 75.97 | 78.07 |  +2.10 |  +2.76 | 67.29 | 62.40 | -4.89 | -7.27 | 73.45 | 76.44 |  +2.99 |  +4.07 | 65.53 | 61.26 | -4.27 | -6.51 |
| JSON_PRETTY | 9243 | 11734 |  +2491 |  +26.95 | 6858 | 8706 |  +1848 |  +26.95 | 2386 | 3029 |  +643 |  +26.95 | 74.19 | 74.19 | 0.00 | 0.00 | 68.39 | 59.87 | -8.52 | -12.46 | 73.24 | 73.09 | -0.15 | -0.20 | 67.73 | 59.10 | -8.63 | -12.74 |
| TOON_DEFAULT | 8915 | 11912 |  +2997 |  +33.62 | 6767 | 9208 |  +2441 |  +36.07 | 2148 | 2704 |  +556 |  +25.91 | 75.91 | 77.30 |  +1.39 |  +1.83 | 70.72 | 61.44 | -9.28 | -13.12 | 74.21 | 77.26 |  +3.05 |  +4.11 | 69.53 | 61.41 | -8.12 | -11.67 |
| XML_COMPACT | 5295 | 9288 |  +3993 |  +75.40 | 4042 | 6940 |  +2898 |  +71.71 | 1253 | 2347 |  +1094 |  +87.32 | 76.34 | 74.73 | -1.61 | -2.11 | 83.40 | 68.62 | -14.78 | -17.73 | 74.35 | 74.36 |  +0.01 |  +0.01 | 82.01 | 68.36 | -13.65 | -16.64 |
| XML_PRETTY | 9945 | 8302 | -1643 | -16.53 | 7004 | 5914 | -1090 | -15.57 | 2941 | 2388 | -553 | -18.81 | 70.43 | 71.24 |  +0.81 |  +1.15 | 63.36 | 69.55 |  +6.19 |  +9.77 | 68.94 | 70.68 |  +1.74 |  +2.52 | 62.32 | 69.16 |  +6.84 |  +10.97 |
| YAML | 10031 | 8088 | -1943 | -19.37 | 7766 | 5457 | -2309 | -29.73 | 2265 | 2631 |  +366 |  +16.17 | 77.42 | 67.47 | -9.95 | -12.85 | 67.96 | 67.64 | -0.32 | -0.47 | 76.25 | 67.13 | -9.12 | -11.96 | 67.14 | 67.40 |  +0.26 |  +0.39 |

#### 2.6.4 Total Tokens Mandatory vs Optional Data
| Format | Total Tokens Man | Total Tokens Opt | Diff | Diff (%) | Useful Total Tokens Man | Useful Total Tokens Opt | Diff | Diff (%) | Wasted Total Tokens Man | Wasted Total Tokens Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) | Eff Score Total Man | Eff Score Total Opt | Diff | Diff (%) | Wtd Accuracy (%) Man | Wtd Accuracy (%) Opt | Diff (%) | Wtd Eff Score Total Man | Wtd Eff Score Total Opt | Diff | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 20093 | 21435 |  +1342 |  +6.68 | 15265 | 16734 |  +1469 |  +9.63 | 4828 | 4700 | -128 | -2.65 | 75.97 | 78.07 |  +2.10 |  +2.76 | 78.00 | 76.16 | -1.84 | -2.36 | 73.45 | 76.44 |  +2.99 |  +4.07 | 76.23 | 75.02 | -1.21 | -1.59 |
| JSON_PRETTY | 26925 | 28491 |  +1566 |  +5.82 | 19976 | 21138 |  +1162 |  +5.82 | 6949 | 7353 |  +404 |  +5.82 | 74.19 | 74.19 | 0.00 | 0.00 | 59.91 | 56.04 | -3.86 | -6.45 | 73.24 | 73.09 | -0.15 | -0.20 | 59.24 | 55.27 | -3.97 | -6.69 |
| TOON_DEFAULT | 22983 | 25744 |  +2761 |  +12.01 | 17446 | 19900 |  +2454 |  +14.07 | 5537 | 5844 |  +307 |  +5.55 | 75.91 | 77.30 |  +1.39 |  +1.83 | 70.83 | 64.99 | -5.83 | -8.24 | 74.21 | 77.26 |  +3.05 |  +4.11 | 69.64 | 64.97 | -4.67 | -6.71 |
| XML_COMPACT | 18000 | 21743 |  +3743 |  +20.79 | 13741 | 16248 |  +2507 |  +18.24 | 4259 | 5495 |  +1236 |  +29.01 | 76.34 | 74.73 | -1.61 | -2.11 | 83.41 | 73.06 | -10.35 | -12.41 | 74.35 | 74.36 |  +0.01 |  +0.01 | 82.02 | 72.80 | -9.22 | -11.24 |
| XML_PRETTY | 30149 | 27973 | -2176 | -7.22 | 21234 | 19928 | -1306 | -6.15 | 8915 | 8045 | -870 | -9.76 | 70.43 | 71.24 |  +0.81 |  +1.15 | 49.33 | 55.26 |  +5.93 |  +12.03 | 68.94 | 70.68 |  +1.74 |  +2.52 | 48.28 | 54.87 |  +6.58 |  +13.63 |
| YAML | 24186 | 22236 | -1950 | -8.06 | 18725 | 15003 | -3722 | -19.88 | 5461 | 7233 |  +1772 |  +32.45 | 77.42 | 67.47 | -9.95 | -12.85 | 68.92 | 66.76 | -2.16 | -3.13 | 76.25 | 67.13 | -9.12 | -11.96 | 68.10 | 66.52 | -1.58 | -2.32 |

### 2.7 Answer Per Format Breakdown
#### 2.7.1 Metrics
| Format | Variant | Correct Answers | Incorrect Answers | No Answers | Accuracy (%) |
|---|---|---|---|---|---|
| JSON_COMPACT | man | 94 | 30 | 0 | 75.97 |
| JSON_COMPACT | opt | 97 | 27 | 0 | 78.07 |
| JSON_PRETTY | man | 92 | 32 | 0 | 74.19 |
| JSON_PRETTY | opt | 92 | 32 | 0 | 74.19 |
| TOON_DEFAULT | man | 94 | 30 | 0 | 75.91 |
| TOON_DEFAULT | opt | 96 | 28 | 0 | 77.30 |
| XML_COMPACT | man | 95 | 29 | 0 | 76.34 |
| XML_COMPACT | opt | 93 | 31 | 0 | 74.73 |
| XML_PRETTY | man | 87 | 37 | 0 | 70.43 |
| XML_PRETTY | opt | 88 | 36 | 0 | 71.24 |
| YAML | man | 96 | 28 | 0 | 77.42 |
| YAML | opt | 84 | 40 | 0 | 67.47 |

#### 2.7.2 Mandatory vs Optional Data
| Format | Correct Man | Correct Opt | Diff | Diff (%) | Incorrect Man | Incorrect Opt | Diff | Diff (%) | No Answers Man | No Answers Opt | Diff | Diff (%) | Accuracy (%) Man | Accuracy (%) Opt | Diff (%) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT | 94 | 97 |  +3 |  +3.19 | 30 | 27 | -3 | -10.00 | 0 | 0 | 0 | 0.00 | 75.97 | 78.07 |  +2.10 |
| JSON_PRETTY | 92 | 92 | 0 | 0.00 | 32 | 32 | 0 | 0.00 | 0 | 0 | 0 | 0.00 | 74.19 | 74.19 | 0.00 |
| TOON_DEFAULT | 94 | 96 |  +2 |  +2.13 | 30 | 28 | -2 | -6.67 | 0 | 0 | 0 | 0.00 | 75.91 | 77.30 |  +1.39 |
| XML_COMPACT | 95 | 93 | -2 | -2.11 | 29 | 31 |  +2 |  +6.90 | 0 | 0 | 0 | 0.00 | 76.34 | 74.73 | -1.61 |
| XML_PRETTY | 87 | 88 |  +1 |  +1.15 | 37 | 36 | -1 | -2.70 | 0 | 0 | 0 | 0.00 | 70.43 | 71.24 |  +0.81 |
| YAML | 96 | 84 | -12 | -12.50 | 28 | 40 |  +12 |  +42.86 | 0 | 0 | 0 | 0.00 | 77.42 | 67.47 | -9.95 |

### 2.8 Accuracy Per Question Category Analysis
#### 2.8.1 Metrics
| Format | Variant | Accuracy (%) | Field Retrieval (%) | Structure Awareness (%) | Filtering (%) | Aggregation (%) |
|---|---|---|---|---|---|---|
| JSON_COMPACT | man | 75.97 | 96.36 | 60.74 | 58.09 | 60.00 |
| JSON_COMPACT | opt | 78.07 | 98.18 | 67.41 | 63.81 | 53.33 |
| JSON_PRETTY | man | 74.19 | 91.64 | 69.63 | 60.00 | 48.57 |
| JSON_PRETTY | opt | 74.19 | 95.76 | 61.73 | 68.25 | 39.68 |
| TOON_DEFAULT | man | 75.91 | 93.64 | 68.06 | 57.74 | 57.74 |
| TOON_DEFAULT | opt | 77.30 | 93.51 | 77.78 | 66.67 | 44.90 |
| XML_COMPACT | man | 76.34 | 90.91 | 69.14 | 55.55 | 68.25 |
| XML_COMPACT | opt | 74.73 | 94.55 | 72.84 | 61.90 | 38.10 |
| XML_PRETTY | man | 70.43 | 88.48 | 62.96 | 53.97 | 49.21 |
| XML_PRETTY | opt | 71.24 | 88.48 | 66.67 | 61.90 | 41.27 |
| YAML | man | 77.42 | 99.39 | 70.37 | 61.90 | 44.45 |
| YAML | opt | 67.47 | 82.42 | 62.96 | 61.90 | 39.68 |

#### 2.8.2 Field Retrieval: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 96.36 | 98.18 |  +1.82 |
| JSON_PRETTY | 91.64 | 95.76 |  +4.12 |
| TOON_DEFAULT | 93.64 | 93.51 | -0.13 |
| XML_COMPACT | 90.91 | 94.55 |  +3.64 |
| XML_PRETTY | 88.48 | 88.48 | 0.00 |
| YAML | 99.39 | 82.42 | -16.97 |

#### 2.8.3 Structure Awareness: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 60.74 | 67.41 |  +6.67 |
| JSON_PRETTY | 69.63 | 61.73 | -7.90 |
| TOON_DEFAULT | 68.06 | 77.78 |  +9.72 |
| XML_COMPACT | 69.14 | 72.84 |  +3.70 |
| XML_PRETTY | 62.96 | 66.67 |  +3.70 |
| YAML | 70.37 | 62.96 | -7.41 |

#### 2.8.4 Filtering: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 58.09 | 63.81 |  +5.71 |
| JSON_PRETTY | 60.00 | 68.25 |  +8.26 |
| TOON_DEFAULT | 57.74 | 66.67 |  +8.93 |
| XML_COMPACT | 55.55 | 61.90 |  +6.35 |
| XML_PRETTY | 53.97 | 61.90 |  +7.94 |
| YAML | 61.90 | 61.90 |  +0.00 |

#### 2.8.5 Aggregation: Mandatory vs Optional

| Format | Mand (%) | Opt (%) | Diff (%) |
|---|---|---|---|
| JSON_COMPACT | 60.00 | 53.33 | -6.67 |
| JSON_PRETTY | 48.57 | 39.68 | -8.89 |
| TOON_DEFAULT | 57.74 | 44.90 | -12.84 |
| XML_COMPACT | 68.25 | 38.10 | -30.16 |
| XML_PRETTY | 49.21 | 41.27 | -7.93 |
| YAML | 44.45 | 39.68 | -4.76 |

## 4. Appendices

### 4.1 Appendix A: Test Infrastructure
- **Test Date**: 2026-03-22
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **Thinking**: off
- **Structure**: nested
- **Formats Tested**: JSON_COMPACT, JSON_PRETTY, TOON_DEFAULT, XML_COMPACT, XML_PRETTY, YAML
- **Record Counts**: 31
- **Total Test Cases**: 12

### 4.2 Appendix B: Benchmark Configuration
- **Field Retrieval**: 55 questions (37.50% weight)
- **Filtering**: 21 questions (20.83% weight)
- **Aggregation**: 21 questions (12.50% weight)
- **Structure Awareness**: 27 questions (29.17% weight)

---

- **Report Generated**: 2026-04-09
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code < 2.1.86
- **Data Source**: `analytics_results.json`
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Related Benchmark Results**:
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_flat_all_formats_and_variants_off\BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_flat_all_formats_and_variants_on\BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/results_nested_all_formats_and_variants_on\BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)