# DVSCP -- Chinese Version (Simplified)
## Dynamic Vector Semantic Compression Protocol

**Author:** Masato Amano (M AI-studio)
**Status:** Patent Application Pending
**Version:** v1.0 / 2026-03-18

> Japanese repository: [dvscp-concept-ja](https://github.com/masatoamano1967-dotcom/dvscp-concept-ja)
> English repository: [dvscp-concept-en](https://github.com/masatoamano1967-dotcom/dvscp-concept-en)

---

## Overview

DVSCP is a communication protocol that extracts only the 'semantic skeleton' from text, compresses it into binary data, and delegates reconstruction to an AI on the receiving side.

```
[Original Chinese Text]
    |
    v (jieba segmentation + CC-CEDICT, no AI)
[Semantic Skeleton Packets]
    |
    v (Any LLM)
[Reconstructed Simplified Chinese]
```

**Core principle:** Transmit only what the receiving AI cannot infer from context.

---

## Chinese-Specific Design

### Word Segmentation
Chinese text has no spaces between words. jieba handles segmentation automatically with rule-based algorithms -- no AI required.

### Coordinate System
CC-CEDICT (~120,000 entries) provides stable X/Y coordinates via entry line numbers and POS bands:
- Noun band: Y = 0-63
- Verb band: Y = 64-127
- Adjective band: Y = 128-191
- Other band: Y = 192-255

### Negation (DIR_R -- highest priority)
Chinese negation words always transmitted as anchors:
- 不 / 没 / 无 / 非 -- semantic reversal
- 但 / 然而 / 尽管 -- contrast/adversative

### Always Skipped
Structural particles (的/地/得), aspect particles (了/着/过), and sentence-final particles (吗/呢/吧) carry no semantic weight and are always omitted.

---

## Benchmark Results (Measured / 2026-03-18)

| # | Original text | Original | Compressed | Reduction |
|---|---|---|---|---|
| 1 | 科学家悄悄地做出了重大发现。 | 42B | 13B | **69.0%** |
| 2 | 她微笑着，但眼中充满了泪水。 | 42B | 18B | **57.1%** |
| 3 | 他从不放弃，即使一切看起来都没有希望。 | 57B | 27B | **52.6%** |
| 4 | 老人沿着河边慢慢地走，回忆着他的青春。 | 57B | 24B | **57.9%** |
| 5 | 尽管危险，她还是毫不犹豫地向前迈进。 | 54B | 21B | **61.1%** |
| **Total** | | **252B** | **103B** | **57.9%** |

> Note: Chinese UTF-8 encodes each character as 3 bytes. A V5 packet is also
> 3 bytes, so single-character words yield ~0% per-token reduction. This is a
> UTF-8 encoding property, not a theoretical limitation of DVSCP.

---

## Multi-AI Reconstruction Test

| Test | ChatGPT | Gemini | Perplexity | Meaning | Emotion |
|---|---|---|---|---|---|
| 1 | 科学家悄悄地做出了一个重大发现。 | 科学家悄悄地做出了重大发现。 | 科学家悄悄地做出了重大发现。 | OK | OK |
| 2 | 她微笑着，但眼中充满了泪水。 | 她微笑着，但眼中却充满了泪水。 | 她微笑着，但眼中却充满了泪水。 | OK | OK |
| 3 | 他从不放弃，即使一切看起来都没有希望。 | 他从不放弃，即使一切看起来都毫无希望。 | 他从不放弃，即使一切看起来都没有希望。 | OK | OK |
| 4 | 老人沿着河边慢慢走着，回忆着他的青春。 | 老人沿着河边慢慢走着，回忆着他的青春。 | 老人沿着河边慢慢走着，回忆着自己的青春。 | OK | OK |
| 5 | 尽管危险，她还是毫不犹豫地向前迈进。 | 尽管危险，她还是毫不犹豫地向前迈进。 | 尽管危险，她还是毫不犹豫地向前迈进。 | OK | OK |

**Meaning preserved: 5/5 | Emotion preserved: 5/5 | NEG/contrast preserved: 5/5**

> Test 5: All 3 AIs produced identical output -- perfect score.
> Synonym enrichment (却/毫无/自己) by Gemini/Perplexity is meaning-equivalent or stronger.

See [sample_results.md](./sample_results.md) for full analysis.

---

## Three-Language Comparison

| Metric | Japanese v5.1 | English v1.0 | Chinese v1.0 |
|---|---|---|---|
| Tokenizer | MeCab | spaCy | jieba |
| Coordinate system | LaBSE+UMAP | WordNet synset | CC-CEDICT ID |
| Avg compression | ~77-85% | 70.5% | **57.9%** |
| Meaning preservation | 100% | 100% | **100%** |
| Emotion preservation | 100% | 100% | **100%** |
| Encoding | Rule-based | Rule-based | Rule-based |

All three languages achieve 100% meaning and emotion preservation,
demonstrating that **DVSCP's semantic skeleton principle is language-independent**.

---

## Repository Structure

```
dvscp-concept-zh/
  README.md            <- This file
  THEORY.md            <- Theoretical background
  sample_results.md    <- Full reconstruction experiment results
  dvscp_interface.py   <- Public interface definition (implementation not disclosed)
  LICENSE
```
## 路线图 — 语音DVSCP（下一代）

目前正在开发集成音频维度的下一代版本。

| 轴 | 说明 |
|---|---|
| PITCH | 基频轮廓 |
| INTON | 语调模式 |
| EMOT | 情感极性 |
| 惯性 | 韵律连续性（可预测时跳过） |

**压缩率提升：** 语音DVSCP已实证将1分钟音频从约1.9MB压缩至582B，压缩率超过99%。

文本DVSCP（日语/英语/中文）构成该统一架构的语义层。
语音DVSCP是主发明，文本DVSCP作为其补充。

*语音DVSCP — 专利申请中 / M AI-studio*
---

## License

Copyright 2026 Masato Amano. All Rights Reserved.
This repository contains intellectual property subject to a pending patent application.
Reference for research purposes is permitted.
Implementation or redistribution requires explicit written permission.
