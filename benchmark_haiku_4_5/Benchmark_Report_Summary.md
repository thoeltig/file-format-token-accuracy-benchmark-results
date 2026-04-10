# File Format Token Efficiency Benchmark: Comprehensive Report

## 1. Methodology

The underlying question: **Which file format delivers maximum information value per token consumed?**

This requires measuring:
- **Token Cost**: How many tokens does each format consume for equivalent data?
- **Information Fidelity**: How accurately can the model understand and answer questions about the data?
- **Robustness**: How consistent is performance across data variants (mandatory vs optional fields)?

### 1.1 Metric Definitions

**Token Metrics:**
- `Read Tokens`: Tokens consumed reading the data file
- `Output Tokens`: Tokens consumed during inference
   - `Output Before Write Tokens` = The output tokens before the write tool was used (understanding instructions and files)
   - `Output Write Tokens` = The output tokens used to write the answers (answering questions and creating the file content)
- `Total Tokens`: Read tokens + output tokens

**Accuracy Metrics:**
- `Accuracy`: Correct answers / total questions

**Efficiency Score:**
- Composite metric balancing accuracy with normalized token cost (weighted towards accuracy)
- `NormalizedTokenCost` = (((MaxTotalTokens + 10) − CurrentTotalTokens) / ((MaxTotalTokens + 10) − (MinTotalTokens − 10))) × 100
- `EfficiencyScore` = (Accuracy% × 0.7) + (NormalizedTokenCost × 0.3)

### 1.2 Formats Tested

| Format ID | Description |
|---|---|
| **CSV** | Comma-separated values — flat structure only |
| **JSON_COMPACT** | Minified JSON (no whitespace or newlines) |
| **JSON_PRETTY** | Standard indented JSON |
| **XML_COMPACT** | Minified XML (no whitespace or indentation) |
| **XML_PRETTY** | Standard indented XML |
| **TOON_DEFAULT** | [Token-Oriented Object Notation](https://toonformat.dev/) with key folding disabled — nested structures expanded as-is |
| **YAML** | Standard YAML with hierarchical indentation |

## 2. Simplified Results

> [!NOTE]
>
> These benchmark results are specific to Claude Code using the Haiku 4.5 model. They serve as a rule of thumb for choosing the best file format for your use case.
> However these values cannot be directly applied to models from other providers as token usage and latency depend on specific model architectures and tokenizers. While the relative ranking of file formats remains consistent the absolute number of tokens will vary.
>
> All columns ranked best-to-worst: 
> - ↑ = lower value is better (ascending)
> - ↓ = higher value is better (descending)
>
> See [§1.1](#11-metric-definitions) for formulas

### 2.1 Flat Structure With Thinking On

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)

#### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|
| XML_COMPACT ≈ 75s | CSV ≈ 6989 | JSON_COMPACT ≈ 228 | XML_COMPACT ≈ 9086 | XML_COMPACT ≈ 9429 | CSV ≈ 16525 | JSON_PRETTY ≈ 83% | TOON_DEFAULT ≈ 85 | XML_COMPACT ≈ 79 | TOON_DEFAULT ≈ 80 |
| CSV (+3.9%) | TOON_DEFAULT (+0.8%) | YAML (+1.5%) | CSV (+1.3%) | CSV (+1.1%) | TOON_DEFAULT (+13.3%) | TOON_DEFAULT (-3.2%) | JSON_COMPACT (-12.5%) | YAML (-7.5%) | CSV (-7.2%) |
| YAML (+17.2%) | JSON_COMPACT (+32.6%) | TOON_DEFAULT (+26.1%) | YAML (+14.0%) | YAML (+12.3%) | XML_COMPACT (+27.8%) | XML_PRETTY (-4.8%) | CSV (-12.9%) | TOON_DEFAULT (-12.1%) | XML_COMPACT (-9.1%) |
| TOON_DEFAULT (+27.4%) | XML_COMPACT (+67.3%) | CSV (+44.7%) | TOON_DEFAULT (+25.3%) | TOON_DEFAULT (+23.8%) | JSON_COMPACT (+35.7%) | XML_COMPACT (-5.4%) | XML_COMPACT (-19.2%) | CSV (-12.9%) | YAML (-15.4%) |
| XML_PRETTY (+36.2%) | YAML (+79.6%) | JSON_PRETTY (+49.8%) | XML_PRETTY (+36.9%) | XML_PRETTY (+35.5%) | YAML (+40.0%) | YAML (-5.6%) | YAML (-22.6%) | JSON_PRETTY (-18.4%) | JSON_COMPACT (-15.6%) |
| JSON_PRETTY (+40.7%) | JSON_PRETTY (+104.4%) | XML_COMPACT (+50.5%) | JSON_PRETTY (+40.8%) | JSON_PRETTY (+39.3%) | JSON_PRETTY (+65.9%) | JSON_COMPACT (-8.3%) | JSON_PRETTY (-24.4%) | XML_PRETTY (-20.4%) | JSON_PRETTY (-23.3%) |
| JSON_COMPACT (+45.8%) | XML_PRETTY (+131.3%) | XML_PRETTY (+50.5%) | JSON_COMPACT (+42.3%) | JSON_COMPACT (+39.6%) | XML_PRETTY (+75.2%) | CSV (-19.1%) | XML_PRETTY (-35.4%) | JSON_COMPACT (-25.9%) | XML_PRETTY (-32.1%) |

#### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 73s | CSV ≈ 6700 | CSV ≈ 231 | JSON_PRETTY ≈ 7966 | JSON_PRETTY ≈ 8302 | JSON_COMPACT ≈ 18753 | XML_PRETTY ≈ 80% | JSON_COMPACT ≈ 79 | JSON_PRETTY ≈ 83 | JSON_COMPACT ≈ 80 |
| XML_PRETTY (+7.9%) | JSON_COMPACT (+30.6%) | XML_PRETTY (+0.6%) | JSON_COMPACT (+21.4%) | JSON_COMPACT (+20.5%) | CSV (+4.2%) | TOON_DEFAULT (-0.3%) | CSV (-6.3%) | XML_PRETTY (-7.0%) | XML_COMPACT (-11.0%) |
| JSON_COMPACT (+13.2%) | XML_COMPACT (+63.1%) | YAML (+44.3%) | XML_PRETTY (+23.0%) | XML_PRETTY (+20.8%) | JSON_PRETTY (+15.5%) | JSON_COMPACT (-0.5%) | XML_COMPACT (-8.7%) | JSON_COMPACT (-7.3%) | JSON_PRETTY (-11.6%) |
| YAML (+21.0%) | TOON_DEFAULT (+72.6%) | XML_COMPACT (+45.2%) | YAML (+27.7%) | YAML (+26.6%) | YAML (+18.8%) | XML_COMPACT (-0.5%) | TOON_DEFAULT (-11.0%) | YAML (-11.4%) | YAML (-11.8%) |
| TOON_DEFAULT (+28.9%) | YAML (+75.7%) | JSON_PRETTY (+45.5%) | XML_COMPACT (+40.1%) | XML_COMPACT (+38.5%) | XML_COMPACT (+19.6%) | YAML (-1.9%) | YAML (-13.3%) | XML_COMPACT (-16.1%) | TOON_DEFAULT (-13.5%) |
| XML_COMPACT (+30.1%) | JSON_PRETTY (+99.5%) | JSON_COMPACT (+46.2%) | TOON_DEFAULT (+43.5%) | TOON_DEFAULT (+41.7%) | TOON_DEFAULT (+24.4%) | JSON_PRETTY (-3.8%) | JSON_PRETTY (-21.3%) | TOON_DEFAULT (-17.5%) | CSV (-16.7%) |
| CSV (+50.1%) | XML_PRETTY (+125.2%) | TOON_DEFAULT (+47.3%) | CSV (+58.3%) | CSV (+54.7%) | XML_PRETTY (+33.9%) | CSV (-16.9%) | XML_PRETTY (-24.8%) | CSV (-37.8%) | XML_PRETTY (-18.6%) |


### 2.2 Flat Structure With Thinking Off

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)

#### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 76s | CSV ≈ 7022 | CSV ≈ 241 | XML_COMPACT ≈ 6673 | XML_COMPACT ≈ 6976 | CSV ≈ 17340 | JSON_PRETTY ≈ 83% | TOON_DEFAULT ≈ 82 | XML_COMPACT ≈ 81 | TOON_DEFAULT ≈ 79 |
| XML_COMPACT (+5.8%) | TOON_DEFAULT (+2.3%) | TOON_DEFAULT (+7.0%) | YAML (+35.9%) | YAML (+34.4%) | TOON_DEFAULT (+8.4%) | JSON_COMPACT (-4.5%) | JSON_COMPACT (-7.1%) | YAML (-11.5%) | CSV (-2.4%) |
| CSV (+6.6%) | JSON_COMPACT (+32.4%) | JSON_PRETTY (+25.4%) | CSV (+51.0%) | CSV (+47.9%) | XML_COMPACT (+9.1%) | TOON_DEFAULT (-5.7%) | CSV (-7.3%) | XML_PRETTY (-17.7%) | XML_COMPACT (-3.6%) |
| XML_PRETTY (+13.9%) | XML_COMPACT (+70.2%) | YAML (+25.4%) | XML_PRETTY (+54.3%) | XML_PRETTY (+52.0%) | YAML (+26.6%) | XML_PRETTY (-5.9%) | XML_COMPACT (-21.3%) | JSON_PRETTY (-20.4%) | YAML (-13.6%) |
| TOON_DEFAULT (+25.2%) | YAML (+79.1%) | XML_COMPACT (+25.8%) | TOON_DEFAULT (+70.2%) | TOON_DEFAULT (+66.5%) | JSON_COMPACT (+31.4%) | YAML (-6.7%) | YAML (-21.6%) | TOON_DEFAULT (-23.2%) | JSON_COMPACT (-14.9%) |
| JSON_PRETTY (+26.8%) | JSON_PRETTY (+103.8%) | JSON_COMPACT (+26.4%) | JSON_PRETTY (+75.0%) | JSON_PRETTY (+71.7%) | JSON_PRETTY (+51.6%) | XML_COMPACT (-9.1%) | JSON_PRETTY (-22.6%) | CSV (-23.9%) | JSON_PRETTY (-25.0%) |
| JSON_COMPACT (+40.5%) | XML_PRETTY (+130.5%) | XML_PRETTY (+26.5%) | JSON_COMPACT (+97.6%) | JSON_COMPACT (+93.4%) | XML_PRETTY (+54.5%) | CSV (-15.0%) | XML_PRETTY (-34.8%) | JSON_COMPACT (-32.8%) | XML_PRETTY (-32.2%) |

#### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|
| TOON_DEFAULT ≈ 74s | CSV ≈ 6728 | JSON_PRETTY ≈ 231 | TOON_DEFAULT ≈ 8675 | TOON_DEFAULT ≈ 8984 | CSV ≈ 17689 | YAML ≈ 80% | JSON_COMPACT ≈ 77 | TOON_DEFAULT ≈ 74 | JSON_COMPACT ≈ 80 |
| XML_PRETTY (+0.3%) | JSON_COMPACT (+30.3%) | XML_PRETTY (+30.7%) | XML_PRETTY (+2.7%) | XML_PRETTY (+2.5%) | JSON_COMPACT (+3.6%) | JSON_PRETTY (-1.7%) | CSV (-1.7%) | XML_PRETTY (-0.5%) | CSV (-7.2%) |
| JSON_COMPACT (+2.1%) | XML_COMPACT (+66.2%) | CSV (+31.4%) | JSON_COMPACT (+6.7%) | JSON_COMPACT (+6.4%) | TOON_DEFAULT (+17.0%) | XML_COMPACT (-1.9%) | XML_COMPACT (-8.2%) | JSON_COMPACT (-2.9%) | TOON_DEFAULT (-9.9%) |
| JSON_PRETTY (+14.8%) | TOON_DEFAULT (+74.0%) | YAML (+31.6%) | JSON_PRETTY (+17.3%) | JSON_PRETTY (+15.9%) | XML_COMPACT (+26.9%) | XML_PRETTY (-3.5%) | YAML (-9.0%) | JSON_PRETTY (-6.2%) | XML_COMPACT (-14.6%) |
| CSV (+16.9%) | YAML (+75.3%) | JSON_COMPACT (+31.9%) | CSV (+22.9%) | CSV (+22.0%) | JSON_PRETTY (+34.6%) | JSON_COMPACT (-3.8%) | TOON_DEFAULT (-12.7%) | XML_COMPACT (-11.7%) | JSON_PRETTY (-19.8%) |
| XML_COMPACT (+18.4%) | JSON_PRETTY (+99.1%) | XML_COMPACT (+33.8%) | XML_COMPACT (+26.3%) | XML_COMPACT (+25.4%) | XML_PRETTY (+37.5%) | TOON_DEFAULT (-4.5%) | JSON_PRETTY (-17.1%) | YAML (-20.9%) | YAML (-22.3%) |
| YAML (+40.4%) | XML_PRETTY (+124.7%) | TOON_DEFAULT (+33.8%) | YAML (+46.7%) | YAML (+45.0%) | YAML (+40.3%) | CSV (-14.9%) | XML_PRETTY (-25.8%) | CSV (-22.2%) | XML_PRETTY (-23.4%) |


### 2.3 Nested Structure With Thinking On

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)

*CSV excluded — flat-only format, does not support nested structures.*

#### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|
| JSON_COMPACT ≈ 69s | JSON_COMPACT ≈ 10315 | JSON_COMPACT ≈ 336 | XML_PRETTY ≈ 7175 | XML_PRETTY ≈ 7518 | JSON_COMPACT ≈ 18446 | YAML ≈ 79% | JSON_COMPACT ≈ 82 | XML_PRETTY ≈ 81 | JSON_COMPACT ≈ 83 |
| XML_PRETTY (+2.2%) | XML_COMPACT (+24.6%) | YAML (+0.5%) | JSON_COMPACT (+8.6%) | JSON_COMPACT (+8.2%) | XML_COMPACT (+18.3%) | JSON_PRETTY (-1.9%) | XML_COMPACT (-10.2%) | JSON_COMPACT (-0.9%) | XML_COMPACT (-12.2%) |
| TOON_DEFAULT (+7.0%) | TOON_DEFAULT (+36.7%) | TOON_DEFAULT (+1.7%) | TOON_DEFAULT (+16.3%) | TOON_DEFAULT (+15.5%) | TOON_DEFAULT (+23.5%) | JSON_COMPACT (-3.0%) | YAML (-11.7%) | TOON_DEFAULT (-4.7%) | TOON_DEFAULT (-15.4%) |
| XML_COMPACT (+8.0%) | YAML (+38.7%) | XML_PRETTY (+1.9%) | XML_COMPACT (+20.2%) | XML_COMPACT (+19.3%) | YAML (+37.8%) | TOON_DEFAULT (-4.3%) | TOON_DEFAULT (-14.6%) | XML_COMPACT (-6.0%) | YAML (-20.4%) |
| YAML (+31.8%) | JSON_PRETTY (+72.8%) | XML_COMPACT (+2.6%) | JSON_PRETTY (+43.9%) | JSON_PRETTY (+43.3%) | XML_PRETTY (+49.8%) | XML_COMPACT (-4.3%) | JSON_PRETTY (-25.8%) | YAML (-12.6%) | XML_PRETTY (-32.3%) |
| JSON_PRETTY (+34.9%) | XML_PRETTY (+95.0%) | JSON_PRETTY (+33.0%) | YAML (+50.1%) | YAML (+47.7%) | JSON_PRETTY (+55.0%) | XML_PRETTY (-5.4%) | XML_PRETTY (-36.9%) | JSON_PRETTY (-12.6%) | JSON_PRETTY (-32.5%) |

#### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|
| XML_COMPACT ≈ 64s | JSON_COMPACT ≈ 9788 | JSON_PRETTY ≈ 228 | XML_COMPACT ≈ 7011 | XML_COMPACT ≈ 7356 | JSON_COMPACT ≈ 18250 | YAML ≈ 81% | JSON_COMPACT ≈ 85 | XML_COMPACT ≈ 83 | JSON_COMPACT ≈ 85 |
| XML_PRETTY (+21.1%) | XML_COMPACT (+26.4%) | JSON_COMPACT (+47.4%) | JSON_COMPACT (+15.9%) | JSON_COMPACT (+15.0%) | XML_COMPACT (+8.1%) | TOON_DEFAULT (-0.4%) | XML_COMPACT (-11.6%) | JSON_COMPACT (-2.2%) | XML_COMPACT (-7.6%) |
| JSON_PRETTY (+31.7%) | TOON_DEFAULT (+41.6%) | YAML (+47.6%) | XML_PRETTY (+33.4%) | JSON_PRETTY (+31.3%) | TOON_DEFAULT (+42.7%) | JSON_COMPACT (-1.6%) | TOON_DEFAULT (-12.8%) | XML_PRETTY (-12.6%) | TOON_DEFAULT (-23.8%) |
| JSON_COMPACT (+37.5%) | YAML (+43.6%) | XML_PRETTY (+51.1%) | JSON_PRETTY (+34.5%) | XML_PRETTY (+31.9%) | JSON_PRETTY (+45.5%) | XML_COMPACT (-5.1%) | YAML (-13.2%) | JSON_PRETTY (-13.1%) | YAML (-26.4%) |
| YAML (+47.7%) | JSON_PRETTY (+72.7%) | XML_COMPACT (+51.2%) | TOON_DEFAULT (+68.9%) | TOON_DEFAULT (+65.7%) | YAML (+47.7%) | XML_PRETTY (-7.0%) | JSON_PRETTY (-29.2%) | TOON_DEFAULT (-18.7%) | JSON_PRETTY (-31.5%) |
| TOON_DEFAULT (+51.2%) | XML_PRETTY (+100.1%) | TOON_DEFAULT (+54.5%) | YAML (+79.1%) | YAML (+75.3%) | XML_PRETTY (+60.5%) | JSON_PRETTY (-7.8%) | XML_PRETTY (-37.7%) | YAML (-21.7%) | XML_PRETTY (-39.5%) |


### 2.4 Nested Structure With Thinking Off

[Full report with complete breakdowns](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/benchmark_haiku_4_5/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)

*CSV excluded — flat-only format, does not support nested structures.*

#### Mandatory

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|
| JSON_PRETTY ≈ 77s | JSON_COMPACT ≈ 10163 | YAML ≈ 213 | XML_COMPACT ≈ 4990 | XML_COMPACT ≈ 5295 | XML_COMPACT ≈ 18000 | YAML ≈ 77% | JSON_COMPACT ≈ 82 | XML_COMPACT ≈ 83 | XML_COMPACT ≈ 83 |
| XML_COMPACT (+3.9%) | XML_COMPACT (+25.0%) | XML_PRETTY (+42.1%) | TOON_DEFAULT (+72.5%) | TOON_DEFAULT (+68.4%) | JSON_COMPACT (+11.6%) | XML_COMPACT (-1.1%) | XML_COMPACT (-8.5%) | TOON_DEFAULT (-15.2%) | JSON_COMPACT (-6.5%) |
| XML_PRETTY (+6.1%) | TOON_DEFAULT (+38.4%) | XML_COMPACT (+43.4%) | JSON_PRETTY (+79.1%) | JSON_PRETTY (+74.6%) | TOON_DEFAULT (+27.7%) | JSON_COMPACT (-1.5%) | YAML (-12.6%) | JSON_PRETTY (-18.0%) | TOON_DEFAULT (-15.1%) |
| JSON_COMPACT (+6.6%) | YAML (+39.3%) | JSON_COMPACT (+43.5%) | JSON_COMPACT (+92.9%) | JSON_COMPACT (+87.5%) | YAML (+34.4%) | TOON_DEFAULT (-1.5%) | TOON_DEFAULT (-13.6%) | YAML (-18.5%) | YAML (-17.4%) |
| TOON_DEFAULT (+7.9%) | JSON_PRETTY (+74.0%) | JSON_PRETTY (+44.2%) | XML_PRETTY (+93.2%) | XML_PRETTY (+87.8%) | JSON_PRETTY (+49.6%) | JSON_PRETTY (-3.2%) | JSON_PRETTY (-27.6%) | JSON_COMPACT (-19.3%) | JSON_PRETTY (-28.2%) |
| YAML (+11.8%) | XML_PRETTY (+98.8%) | TOON_DEFAULT (+45.3%) | YAML (+96.8%) | YAML (+89.4%) | XML_PRETTY (+67.5%) | XML_PRETTY (-7.0%) | XML_PRETTY (-39.6%) | XML_PRETTY (-24.0%) | XML_PRETTY (-40.9%) |

#### Optional

| ↑ Total Duration | ↑ Read Tokens | ↑ Output Before Write Tokens | ↑ Output Write Tokens | ↑ Output Tokens | ↑ Total Tokens | ↓ Accuracy | ↓ Eff Score Read | ↓ Eff Score Output | ↓ Eff Score Total |
|---|---|---|---|---|---|---|---|---|---|
| YAML ≈ 67s | JSON_COMPACT ≈ 9645 | XML_PRETTY ≈ 205 | YAML ≈ 7786 | YAML ≈ 8089 | JSON_COMPACT ≈ 21434 | JSON_COMPACT ≈ 78% | JSON_COMPACT ≈ 85 | XML_PRETTY ≈ 70 | JSON_COMPACT ≈ 76 |
| XML_PRETTY (+2.2%) | XML_COMPACT (+29.1%) | TOON_DEFAULT (+25.7%) | XML_PRETTY (+4.0%) | XML_PRETTY (+2.6%) | XML_COMPACT (+1.4%) | TOON_DEFAULT (-0.8%) | XML_COMPACT (-12.2%) | XML_COMPACT (-1.3%) | XML_COMPACT (-4.1%) |
| XML_COMPACT (+13.5%) | TOON_DEFAULT (+43.4%) | YAML (+47.6%) | XML_COMPACT (+15.3%) | XML_COMPACT (+14.8%) | YAML (+3.7%) | XML_COMPACT (-3.3%) | TOON_DEFAULT (-14.7%) | YAML (-2.7%) | YAML (-12.3%) |
| JSON_PRETTY (+39.9%) | YAML (+46.7%) | JSON_COMPACT (+48.7%) | JSON_PRETTY (+46.8%) | JSON_PRETTY (+45.1%) | TOON_DEFAULT (+20.1%) | JSON_PRETTY (-3.9%) | YAML (-23.9%) | JSON_COMPACT (-10.3%) | TOON_DEFAULT (-14.7%) |
| JSON_COMPACT (+40.8%) | JSON_PRETTY (+73.7%) | JSON_PRETTY (+48.8%) | JSON_COMPACT (+47.5%) | JSON_COMPACT (+45.8%) | XML_PRETTY (+30.5%) | XML_PRETTY (-6.8%) | JSON_PRETTY (-27.0%) | TOON_DEFAULT (-11.7%) | JSON_PRETTY (-26.4%) |
| TOON_DEFAULT (+41.0%) | XML_PRETTY (+104.0%) | XML_COMPACT (+49.8%) | TOON_DEFAULT (+49.7%) | TOON_DEFAULT (+47.3%) | JSON_PRETTY (+32.9%) | YAML (-10.6%) | XML_PRETTY (-39.2%) | JSON_PRETTY (-13.9%) | XML_PRETTY (-27.4%) |


## 3. Conclusion & Decision Matrix

### 3.1 Cross-Configuration Findings

#### 3.1.1 Read Token Cost Is Driven by Format Verbosity with One Notable Exception

Across all configurations the read token ranking follows a predictable pattern dictated by format verbosity. **CSV** is consistently the cheapest for flat structures at roughly 7K tokens and **JSON_COMPACT** dominates nested structures at roughly 10K tokens. **XML_PRETTY** sits at the expensive end everywhere because it consumes 100–131% more read tokens than the cheapest option in any given configuration.

The one exception is **TOON_DEFAULT**. On flat mandatory data it trails **CSV** by only 0.8–2.3%, making it the second most token-efficient flat format. On flat optional data however its read token count increases by roughly 64% (from ~7.1K to ~11.6K) which drops it from second place to fourth or fifth. This happens because **TOON_DEFAULT**'s collapsed encoding relies on uniformly populated records to merge field definitions. When the structure is sparse rather than union-filled the encoder cannot collapse records in the same compact manner and instead expands each one individually which eliminates the token advantage that makes **TOON_DEFAULT** competitive on dense data.

#### 3.1.2 Output Write Tokens Correlate Positively with Accuracy

In seven of eight configurations the format achieving the highest accuracy also ranks in the upper half of output write token consumption. The pattern is most visible with **YAML** in nested structures where it leads accuracy in three of four configurations (78.8–80.6%) while simultaneously producing the highest output write token counts (10,767–12,561). **JSON_PRETTY** shows the same behavior in flat mandatory where it leads accuracy at roughly 83% and ranks second highest in output write tokens (11,675–12,797). On the other side of the spectrum **XML_COMPACT** frequently produces the fewest output write tokens (lowest in four of eight configurations at values as low as 4,990) while placing in the lower half for accuracy.

The most striking example comes from **YAML** in the nested optional configuration. With thinking on **YAML** achieves the best accuracy at 80.6% and simultaneously produces the highest output at 12,561 output write tokens. With thinking off **YAML** collapses to the worst accuracy at 67.5% and its output write tokens drop to 7,786 which is the lowest of all formats in that configuration. Both metrics move together which reinforces the hypothesis that output volume and data comprehension are linked.

A plausible explanation would be that when the model comprehends the input data effectively it produces more thorough and detailed answers which naturally consume more tokens but in this benchmark the expected answers are fixed. So in theory if 100% accuracy would be achieved in all formats then the needed output write tokens should have an equal baseline amount over all formats which is equal to the tokens used to generate the answers. The remaining difference per format would discribe how much the model needed or decided to reason about the questions and data in the specific format. This would also outline if models tend to give up earlier if it struggles with a format which would produce partially or completely wrong answers or if the complexetity of a format forces more reasoning which in turn results in correct answers. This would mean that low output token counts are not necessarily a sign of efficiency but may instead reflect uncertainty or shallow data comprehension. The pattern holds more reliably at the top of the accuracy ranking than at the bottom since **CSV** occasionally produces high output write tokens despite poor accuracy (notably 12,614 in flat optional with thinking on at only 63.2% accuracy) which suggests that high output alone does not guarantee quality.

#### 3.1.3 Thinking Mode Normalizes Output Behavior Across Formats

With thinking enabled the spread in output write tokens between the cheapest and most expensive format stays moderate at roughly 40–50%. With thinking disabled this spread nearly doubles to approximately 98% in the most extreme case (flat mandatory where **XML_COMPACT** drops to 6,673 tokens while **JSON_COMPACT** rises to 13,185). The effect is most visible on **XML_COMPACT** which sees its output write tokens drop by up to 27% when thinking is turned off while **JSON_COMPACT**'s output barely changes at under 2%. This suggests that the thinking phase acts as an equalizer that normalizes the model's answer generation regardless of input format. Without it the model's output behavior becomes significantly more dependent on how the data was encoded.

#### 3.1.4 Reasoning Overhead Is Format-Independent

The "Output Before Write" tokens which represent the model's reasoning and planning before it calls the write tool are remarkably stable across formats within any given configuration. They typically vary by around 25–50% from lowest to highest which is modest compared to the 2x spread seen in output write tokens. One outlier is **JSON_PRETTY** in nested mandatory with thinking on where it reaches 447 tokens while other formats in the same configuration cluster between 336 and 345. This indicates that the format of the input data primarily impacts the answer generation phase rather than the comprehension and planning phase. The model appears to invest a roughly constant amount of reasoning effort regardless of whether it reads **CSV**, **JSON**, **XML**, **YAML** or **TOON**.

#### 3.1.5 CSV Offers Token Savings at a Steep Accuracy Cost

**CSV** achieves the lowest flat read token count yet it consistently suffers the worst accuracy across all flat configurations at 15–19% below the best format. Its output token behavior is inconsistent since it sometimes produces among the lowest output (flat mandatory with thinking on at 9,207) and other times produces the highest output (flat optional with thinking on at 12,614) without any accuracy improvement. This disconnect between output volume and accuracy suggests that **CSV**'s problem is not insufficient effort by the model but rather a fundamental difficulty in reliably extracting and cross-referencing information from the flat tabular format. While **CSV** remains viable when an accuracy floor of 63–68% is operationally acceptable its token savings are effectively meaningless for any task requiring reliable data comprehension.

#### 3.1.6 XML_PRETTY Has No Defensible Use Case

**XML_PRETTY** finishes last or near-last in weighted efficiency score across all eight configurations. It occasionally achieves competitive accuracy (it ranks first in flat optional with thinking on at 80%) but the extreme read token overhead of 100–131% above the cheapest format prevents this accuracy from translating into a viable efficiency score. No configuration in this benchmark produces a scenario where **XML_PRETTY** is the recommended choice.

#### 3.1.7 Open Questions for Further Research

The accuracy-output token correlation is the strongest signal that emerged from this cross-configuration analysis but it remains unclear whether this is a property of Haiku 4.5 specifically or a general pattern across model families. Running the same benchmark against Sonnet and Opus would clarify whether larger models exhibit the same format-dependent output behavior or whether additional capacity reduces the sensitivity to input format.

**YAML**'s accuracy collapse in nested optional with thinking off (from 80.6% to 67.5%) is a significant outlier that deserves targeted investigation. It is the only configuration where the top-performing nested format drops to last place and it raises the question of whether **YAML**'s nested accuracy depends specifically on the thinking phase to resolve optional field ambiguity or whether this is a statistical anomaly that would disappear over repeated runs.

The **TOON_DEFAULT** sparse data penalty also raises the question of whether alternative **TOON_DEFAULT** encoding strategies (such as explicit null markers or partial key folding) could recover the token advantage on optional fields without requiring uniform record structures. This would require testing modified **TOON_DEFAULT** encoders rather than additional benchmark runs.

Finally the thinking mode normalization effect on output tokens deserves targeted investigation. A test run with varying thinking budget allocations could reveal whether there is a minimum thinking threshold below which format-dependent output divergence appears and above which it stabilizes.

### 3.2 Decision Matrix

| Scenario | Recommended Format | Eff. Score Total | Accuracy | Rationale |
|---|---|---|---|---|
| Flat structure, dense mandatory fields | **TOON_DEFAULT** | 79–80 | 77–80% | Lowest flat read token count (~7.0–7.2K) and highest flat efficiency score across both thinking modes |
| Flat structure, sparse optional fields | **JSON_COMPACT** | ~80 | 77–80% | **TOON_DEFAULT** read token count rises ~64% on sparse optional schemas while **JSON_COMPACT** remains stable at ~8.7–8.8K |
| Nested structure, any field density | **JSON_COMPACT** | 76–85 | 74–80% | Lowest nested read token count (~9.8–10.3K) and highest nested efficiency score in three of four configurations. **XML_COMPACT** leads nested mandatory with thinking off but **JSON_COMPACT** is a close second and more consistent overall |
| Maximum accuracy required, flat structure | **JSON_PRETTY** | 59–61 | ~83% | Highest flat accuracy at 82.5–82.8% across both thinking modes at the cost of a ~103% read token premium over **TOON_DEFAULT** |
| Maximum accuracy required, nested structure | **YAML** (thinking on only) | 63–66 | ~79–81% | **YAML** achieves highest nested accuracy (78.8–80.6%) with thinking on. Without thinking **YAML**'s accuracy collapses to 67% on optional data so **JSON_COMPACT** becomes the safer choice when thinking cannot be guaranteed |
| Token budget critical, flat only | **CSV** | 67–77 | 63–68% | Lowest flat read token count (~7K) but only viable when an accuracy floor of 63–68% is operationally acceptable |
| Avoid in all use cases | **XML_PRETTY** | 49–65 | 69–78% | Last or near-last efficiency score in all eight configurations with no accuracy advantage that justifies the token overhead |

### 3.3 Real-World Impact

The benchmark results translate into concrete guidance for anyone building systems that feed structured data to language models. The core insight is that format choice alone can cause up to a 2x difference in token consumption and up to 19 percentage points in accuracy on identical data which makes it one of the highest-leverage optimizations available before touching prompts or model selection.

#### 3.3.1 Context Windows and Cost Management

In applications where structured data competes for context window space with instructions, conversation history or retrieved documents every token saved on data encoding frees capacity for other content. **JSON_COMPACT** and **TOON_DEFAULT** on dense flat data consume roughly half the read tokens of **XML_PRETTY** or **JSON_PRETTY** for the same information. For a system that loads multiple data files per request this difference compounds quickly and can determine whether the full context fits within the model's window or requires truncation. Since API pricing scales with token count the format choice also has a direct cost impact at scale.

#### 3.3.2 RAG Pipelines and Knowledge Bases

Retrieval-augmented generation systems that store structured records (product catalogs, configuration databases, user profiles) benefit most from this benchmark. If the retrieved records have a consistent schema with all fields populated then **TOON_DEFAULT** offers the best token efficiency for flat records and **JSON_COMPACT** for nested ones. If the schema contains optional fields that are frequently empty the data becomes sparse and **JSON_COMPACT** is the safer default across both flat and nested structures because its token cost remains stable regardless of field density.

#### 3.3.3 AI-Assisted Code and Configuration Tools

Tools that feed configuration files, schema definitions or structured metadata to a model should prefer **JSON_COMPACT** for nested configurations (which most config files are) since it achieves the lowest read token cost and the highest efficiency score across all nested configurations. **YAML** is a common choice for human-authored config files and it performs well on accuracy (especially with thinking enabled) but its token cost runs 38–47% higher than **JSON_COMPACT** in nested structures. If the config files are already in **YAML** and converting them adds too much engineering overhead then the accuracy advantage may justify keeping **YAML** as-is rather than adding a conversion step.

#### 3.3.4 Data Extraction and Analysis Tasks

When the primary goal is accurate extraction of values from structured data (field retrieval, filtering, aggregation) the accuracy differences between formats matter more than token savings. **JSON_PRETTY** achieves the highest flat accuracy at roughly 83% which is 3–6 percentage points above the efficiency-optimized formats. For tasks where a wrong answer is costlier than extra tokens (financial data, medical records, compliance checks) accepting the token premium of **JSON_PRETTY** or **YAML** is the rational trade-off. For bulk processing where occasional errors are tolerable and throughput matters **JSON_COMPACT** provides the best balance of accuracy and cost.

#### 3.3.5 Thinking Mode Considerations

Systems that allow configuring extended thinking should account for its impact on format sensitivity. With thinking enabled the model's output behavior is more consistent across formats which makes the format choice less critical for output costs. With thinking disabled the output token spread between formats nearly doubles which means a suboptimal format choice has a larger penalty. If thinking is disabled for latency or cost reasons the format choice becomes more consequential and defaults should lean toward **JSON_COMPACT** as the most stable all-around performer.

#### 3.3.6 When Not to Optimize

For one-off queries, small payloads or prototyping where the data fits comfortably within the context window and cost is negligible then the format choice does not meaningfully affect outcomes. At small scales the absolute token differences shrink to noise. The optimization becomes relevant when structured data is a recurring and significant portion of the context such as in production pipelines, automated agents or high-volume API integrations.

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

**Question Distribution:**

- **Field Retrieval:** Extract specific values from specific records (55 questions, 37.50% weight)
- **Filtering:** Count records matching criteria (21 questions, 20.83% weight)
- **Aggregation:** Sum, average, min/max calculations (21 questions, 12.50% weight)
- **Structure Awareness:** Understand data shape, organization, metadata (27 questions, 29.17% weight)

**Weighting Rationale:**

- Field retrieval + structure awareness = 66.67%
   - These represent the file format itself. Understanding "what data exists and how it's organized" which is fundamental to avoiding context confusion.
- Filtering + aggregation = 33.33%
   - These represent more the "intellectual" aspect of the model and will differ greatly depending on the model. Also if done deterministic the model still needs to do field retrieval and structure awareness on the result.

---

- **Report Generated**: 2026-04-09
- **Written by**: [Thore Höltig](https://github.com/thoeltig)
- **Test run in**: Claude Code < 2.1.86
- **Publication**: Open source research in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results)
- **Licensed under**: [CC BY 4.0](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/LICENSE)
- **Full Benchmark Reports**: 
   - [Report - flat structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_flat_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - flat structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_flat_all_formats_and_variants_off/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking on](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_nested_all_formats_and_variants_on/BENCHMARK_REPORT.md)
   - [Report - nested structure & thinking off](https://github.com/thoeltig/file-format-token-accuracy-benchmark-results/tree/feature/benchmark_haiku_4_5_flat_all_formats_and_variants_off/benchmark_haiku_4_5/results_nested_all_formats_and_variants_off/BENCHMARK_REPORT.md)
- **Format Specifics**: [README](https://github.com/thoeltig/file-format-token-accuracy-benchmark#format-specifics)
- **Benchmark Tool**: Claude Code Plugin in [GitHub repository](https://github.com/thoeltig/file-format-token-accuracy-benchmark)