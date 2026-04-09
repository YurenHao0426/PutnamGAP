---
license: cc-by-4.0
language:
- en
pretty_name: PutnamGAP
size_categories:
- 1K<n<10K
task_categories:
- text-generation
- question-answering
tags:
- mathematics
- benchmark
- robustness
- evaluation
- putnam
- competition-mathematics
- llm-evaluation
- gap-framework
- equivalence-preserving
- stress-test
configs:
- config_name: default
  data_files:
  - split: test
    path: dataset/*.json
---

# PutnamGAP

[![arXiv](https://img.shields.io/badge/arXiv-2508.08833-b31b1b.svg)](https://arxiv.org/abs/2508.08833)
[![GitHub](https://img.shields.io/badge/GitHub-YurenHao0426%2FPutnamGAP-blue?logo=github)](https://github.com/YurenHao0426/PutnamGAP)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Hugging Face](https://img.shields.io/badge/🤗-Hugging%20Face-yellow.svg)](https://huggingface.co/datasets/blackhao0426/PutnamGAP)

**PutnamGAP** is a 6,306-item competition-mathematics benchmark covering every William Lowell Putnam Mathematical Competition problem from **1938 to 2024**, expanded into five mathematically equivalent variants per problem via the **GAP** (Generalization-and-Perturbation) framework. PutnamGAP is intended for stress-testing the **robustness** of large language models on advanced mathematical reasoning under semantically equivalent reformulations of the same problem.

> **Paper**: *An Investigation of Robustness of LLMs in Mathematical Reasoning: Benchmarking with Mathematically-Equivalent Transformation of Advanced Mathematical Problems* — Hao, Wan & Zhai, [arXiv:2508.08833](https://arxiv.org/abs/2508.08833)
>
> **Code & pipeline**: <https://github.com/YurenHao0426/PutnamGAP>

---

## What is in the dataset

For each of the 1,051 original Putnam problems we provide:

| Field | Description |
|---|---|
| `index` | Canonical problem id, e.g. `1987-B-2` (year-section-problem) |
| `type` | Topical category (`ALG`, `ANA`, `NT`, `COMB`, `GEO`) |
| `tag` | Free-form sub-tags |
| `difficulty` | Difficulty proxy from problem index (1–2 Easy / 3–4 Medium / 5–6 Hard / 7–8 ExtraHard) |
| `problem_type` | `proof` or `calculation` |
| `question` | Original LaTeX problem statement |
| `solution` | Original LaTeX canonical solution |
| `vars` / `params` | Free / fixed identifiers extracted from the problem |
| `variants` | Five aligned variants — see below |

The `variants` object contains:

| Variant key | Description |
|---|---|
| `descriptive_long` (`DL`) | Variables renamed to single descriptive English phrases (e.g.\ `population_density`). |
| `descriptive_long_confusing` (`DLC`) | Variables renamed to 2–5 unrelated concatenated nouns. |
| `descriptive_long_misleading` (`DLM`) | Variables renamed to mathematically suggestive but semantically misleading names. |
| `garbled_string` (`GS`) | Variables renamed to 4–16 character random alphanumeric hashes. |
| `kernel_variant` (`KV`) | Numeric / parametric slots resampled while preserving the original proof skeleton; the canonical solution is regenerated and verified by a 5-judge LLM ensemble with explicit repair loops. |

Each surface variant additionally exposes a deterministic **rename map** (`variants[v].map`) from canonical variable names to variant variable names. The kernel variant carries provenance metadata in `variants.kernel_variant.metadata`.

**1,051 originals × (1 + 5 variants) = 6,306 items.**

### Cleaning

All text fields in this release have been processed through a Unicode → bare-LaTeX cleaner so that the contents are pure ASCII LaTeX. Greek letters, math operators, sub/superscripts, radical commands and ligatures have been converted to their LaTeX equivalents (e.g.\ `α` → `\alpha`, `≤` → `\leq`, `√{x+1}` → `\sqrt{x+1}`, `x₁₀` → `x_{10}`). The cleaner script is available under `tools/unicode_clean.py` and is reproducible from the included `tools/unicode_audit.py`. The cleaner has been verified to:
- produce **0 non-ASCII characters** across all 1,051 files;
- introduce **0 new brace/parenthesis/bracket imbalances** beyond those already present in the source.

---

## Loading

```python
from datasets import load_dataset

ds = load_dataset("blackhao0426/PutnamGAP", split="test")
print(ds[0]["index"], ds[0]["type"])
print("Variants:", list(ds[0]["variants"].keys()))
```

Or load directly from the JSON files:

```python
import json
from pathlib import Path
problems = [json.load(open(p)) for p in Path("dataset").glob("*.json")]
print(f"{len(problems)} problems loaded")
```

---

## Suggested Use

- **Evaluation, not training.** PutnamGAP is an evaluation benchmark; do not include it in pre-training or fine-tuning corpora that you subsequently evaluate on.
- **Paired evaluation.** The framework's value comes from comparing accuracy on `(original, variant)` pairs, not absolute accuracy on the variants in isolation. Report McNemar tests on flip cases.
- **Mechanistic analyses.** The surface vs kernel decomposition supports mechanism-sensitive analyses such as the paired structural-overlap dichotomy and the repairability rescue protocol described in the paper. Per-cell trajectory tables and rescue results live in the GitHub companion repository.

---

## Important: Source Attribution

> **The original Putnam Competition problem statements and the canonical solutions distributed in this dataset are reproduced from four authoritative monographs published by the Mathematical Association of America (MAA Press), under the fair-use clause printed in the front-matter of every volume:**
>
> *"Individual readers ... are permitted to make fair use of the material, such as to copy select pages for use in teaching or research."*
>
> **All original problem statements and canonical solutions remain the intellectual property of the MAA. If you use this dataset for any research output, you MUST also cite the four MAA source books in addition to citing the GAP paper. Failure to do so misrepresents the provenance of the original problems.**

Problems and solutions from 2017 onward are included with the explicit permission of MAA.

If you are an author, publisher, or rights-holder and you believe any portion of this release infringes your rights, please open an issue at <https://github.com/YurenHao0426/PutnamGAP/issues> or email the maintainer; the affected items will be removed promptly.

> **侵删**: 本数据集包含的所有 Putnam 原题和官方解答均为美国数学协会 (MAA) 出版的四本权威 problem-book 的复制品，按 MAA fair-use 条款用于学术研究。原题与解答的版权归 MAA 所有。任何使用本数据集的下游工作必须同时引用四本 MAA 原始书目（见下方 BibTeX）以及本论文。若有版权方认为本发布涉及侵权，请通过 GitHub Issue 或邮件联系维护者，相关条目将立即移除。

---

## Citation

If you use PutnamGAP, you **must** cite **all five** entries below: the GAP framework paper **and** the four MAA Putnam source books that the original problems and solutions are reproduced from. Citing fewer is a misrepresentation of the dataset's provenance.

In-text example:

> "We evaluate on PutnamGAP \cite{hao2025gap, putnamI, putnamII, putnamIII, putnamIV}."

Full BibTeX (copy the entire block — all five entries are mandatory):

```bibtex
@article{hao2025gap,
  title   = {An Investigation of Robustness of {LLM}s in Mathematical Reasoning:
             Benchmarking with Mathematically-Equivalent Transformation of
             Advanced Mathematical Problems},
  author  = {Hao, Yuren and Wan, Xiang and Zhai, ChengXiang},
  journal = {arXiv preprint arXiv:2508.08833},
  year    = {2025},
  url     = {https://arxiv.org/abs/2508.08833}
}

@book{putnamI,
  title     = {The William Lowell Putnam Mathematical Competition:
               Problems and Solutions 1938--1964},
  author    = {Gleason, A. M. and Greenwood, R. E. and Kelly, L. M.},
  publisher = {Mathematical Association of America},
  year      = {1980},
  series    = {MAA Problem Books},
  volume    = {1},
  address   = {Washington, DC},
  note      = {673\,pp; reprinted by AMS/MAA Press}
}

@book{putnamII,
  title     = {The William Lowell Putnam Mathematical Competition:
               Problems and Solutions 1965--1984},
  author    = {Alexanderson, Gerald L. and Klosinski, Leonard F. and
               Larson, Loren C.},
  publisher = {Mathematical Association of America},
  year      = {1985},
  series    = {MAA Problem Books},
  volume    = {30},
  address   = {Washington, DC},
  note      = {Reprinted by AMS/MAA Press}
}

@book{putnamIII,
  title     = {The William Lowell Putnam Mathematical Competition 1985--2000:
               Problems, Solutions and Commentary},
  author    = {Kedlaya, Kiran S. and Poonen, Bjorn and Vakil, Ravi},
  publisher = {Mathematical Association of America},
  year      = {2002},
  series    = {MAA Problem Books},
  volume    = {33},
  address   = {Washington, DC},
  note      = {Reprinted by AMS/MAA Press}
}

@book{putnamIV,
  title     = {The William Lowell Putnam Mathematical Competition 2001--2016:
               Problems, Solutions and Commentary},
  author    = {Kedlaya, Kiran S. and Kane, Daniel M. and Kane, Jonathan M. and
               O'Dorney, Evan M.},
  publisher = {American Mathematical Society (MAA Press)},
  year      = {2020},
  series    = {MAA Problem Books},
  volume    = {37},
  address   = {Providence, RI},
  note      = {Softcover and e-book versions available}
}
```

> **Reminder.** The four `putnamI`–`putnamIV` entries are not optional or supplementary; the original problem statements and canonical solutions in this dataset are reproduced from those four MAA monographs under the MAA fair-use clause, and the IP belongs to the Mathematical Association of America. Any downstream use of PutnamGAP that omits the four MAA citations misrepresents the dataset's provenance.

---

## License

- The **variant texts**, **rename maps**, **evaluation pipeline**, **structural-overlap analysis code**, **rescue harness**, and **Croissant metadata** are released under the [Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).
- The **original Putnam Competition problem statements and canonical solutions** remain copyrighted by the Mathematical Association of America (MAA). They are redistributed here under MAA's stated fair-use clause and only for educational and research use. **Downstream users must cite the four MAA source books listed above.**
- The cleaned LaTeX version of the original solutions is a derivative work whose changes (Unicode → LaTeX normalisation) are released under CC BY 4.0; the underlying text remains MAA-copyrighted.

---

## Links

- **Paper (arXiv)**: <https://arxiv.org/abs/2508.08833>
- **Code & pipeline (GitHub)**: <https://github.com/YurenHao0426/PutnamGAP>
- **Hugging Face dataset**: <https://huggingface.co/datasets/blackhao0426/PutnamGAP>
- **Issues & contact**: <https://github.com/YurenHao0426/PutnamGAP/issues>
