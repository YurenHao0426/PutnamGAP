
# PutnamGAP

[![arXiv](https://img.shields.io/badge/arXiv-2508.08833-b31b1b.svg)](https://arxiv.org/abs/2508.08833)
[![GAP Code](https://img.shields.io/badge/GitHub-GAP_framework-181717?logo=github)](https://github.com/YurenHao0426/GAP)
[![Dataset Mirror](https://img.shields.io/badge/GitHub-PutnamGAP_mirror-blue?logo=github)](https://github.com/YurenHao0426/PutnamGAP)
[![Hugging Face](https://img.shields.io/badge/🤗_Dataset-PutnamGAP-yellow.svg)](https://huggingface.co/datasets/blackhao0426/PutnamGAP)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

**PutnamGAP** is a 6,306-item competition-mathematics benchmark covering every William Lowell Putnam Mathematical Competition problem from **1938 to 2024**, expanded into five mathematically equivalent variants per problem via the **GAP** (Generalization-and-Perturbation) framework. PutnamGAP is intended for stress-testing the **robustness** of large language models on advanced mathematical reasoning under semantically equivalent reformulations of the same problem.

> **Paper**: *An Investigation of Robustness of LLMs in Mathematical Reasoning: Benchmarking with Mathematically-Equivalent Transformation of Advanced Mathematical Problems* — Hao, Wan & Zhai, [arXiv:2508.08833](https://arxiv.org/abs/2508.08833)
>
> **GAP framework code & evaluation pipeline**: <https://github.com/YurenHao0426/GAP> — this repository hosts only the dataset; the variant generation pipeline, evaluation harness, structural-overlap analysis, repairability rescue runner, and Unicode → LaTeX cleaner all live in the GAP framework repo.
>
> **PutnamGAP dataset GitHub mirror** (this dataset, mirrored from Hugging Face): <https://github.com/YurenHao0426/PutnamGAP>


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

All text fields in this release have been processed through a Unicode → bare-LaTeX cleaner so that the contents are pure ASCII LaTeX. Greek letters, math operators, sub/superscripts, radical commands and ligatures have been converted to their LaTeX equivalents (e.g.\ `α` → `\alpha`, `≤` → `\leq`, `√{x+1}` → `\sqrt{x+1}`, `x₁₀` → `x_{10}`). The cleaner has been verified to:
- produce **0 non-ASCII characters** across all 1,051 files;
- introduce **0 new brace/parenthesis/bracket imbalances** beyond those already present in the source.

The cleaning, audit, brace-balance, and spot-check scripts (`unicode_clean.py`, `unicode_audit.py`, `balance_diff.py`, `spotcheck_clean.py`) live in the [GAP framework repository](https://github.com/YurenHao0426/GAP) under `analysis/`, alongside the rest of the GAP pipeline.


## Loading

The repository contains the same data in two parallel formats:

1. **`dataset.parquet`** — a flat parquet table with 35 columns. This is what the HF dataset viewer renders and what `datasets.load_dataset(...)` returns by default. To keep the schema stable across rows, the four `dict[str, str]`-with-arbitrary-keys fields (`vars`, `params`, `sci_consts`, and per-variant `map` / `metadata`) are stored as JSON-encoded strings whose names end in `_json`. Use `json.loads(...)` to recover the original dict structure.

2. **`dataset/*.json`** — 1,051 individual JSON files with the original nested structure (variants as nested dicts, rename maps as native dicts). Use this layout when running the GAP framework code directly, since the pipeline scripts expect dict access.

### Loading the parquet (default)

```python
from datasets import load_dataset
import json

ds = load_dataset("blackhao0426/PutnamGAP", split="test")
print(ds[0]["index"], ds[0]["type"])
# JSON-stringified fields
print("vars:", json.loads(ds[0]["vars_json"]))
print("DL rename map:", json.loads(ds[0]["variant_descriptive_long_map_json"]))
print("KV question:", ds[0]["variant_kernel_variant_question"][:120])
```

### Loading the JSON files (preserves nested dicts)

```python
import json
from huggingface_hub import snapshot_download
from pathlib import Path

local = snapshot_download("blackhao0426/PutnamGAP", repo_type="dataset",
                          allow_patterns="dataset/*.json")
problems = [json.load(open(p)) for p in sorted(Path(local, "dataset").glob("*.json"))]
print(f"{len(problems)} problems loaded; e.g. {problems[0]['index']}")
print("DL map:", problems[0]["variants"]["descriptive_long"]["map"])
```


## Suggested Use

- **Evaluation, not training.** PutnamGAP is an evaluation benchmark; do not include it in pre-training or fine-tuning corpora that you subsequently evaluate on.
- **Paired evaluation.** The framework's value comes from comparing accuracy on `(original, variant)` pairs, not absolute accuracy on the variants in isolation. Report McNemar tests on flip cases.
- **Mechanistic analyses.** The surface vs kernel decomposition supports mechanism-sensitive analyses such as the paired structural-overlap dichotomy and the repairability rescue protocol described in the paper. Per-cell trajectory tables and rescue results live in the GitHub companion repository.


## Important: Source Attribution

> **The original Putnam Competition problem statements and the canonical solutions distributed in this dataset are reproduced from four authoritative monographs published by the Mathematical Association of America (MAA Press), under the fair-use clause printed in the front-matter of every volume:**
>
> *"Individual readers ... are permitted to make fair use of the material, such as to copy select pages for use in teaching or research."*
>
> **All original problem statements and canonical solutions remain the intellectual property of the MAA. If you use this dataset for any research output, you MUST also cite the four MAA source books in addition to citing the GAP paper. Failure to do so misrepresents the provenance of the original problems.**

Problems and solutions from 2017 onward are included with the explicit permission of MAA.

**Takedown notice.** If you are an author, publisher, or rights-holder and you believe any portion of this release infringes your rights, please open an issue at <https://github.com/YurenHao0426/PutnamGAP/issues> or email the maintainer. The affected items will be removed promptly.


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


## License

- The **variant texts**, **rename maps**, **evaluation pipeline**, **structural-overlap analysis code**, **rescue harness**, and **Croissant metadata** are released under the [Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).
- The **original Putnam Competition problem statements and canonical solutions** remain copyrighted by the Mathematical Association of America (MAA). They are redistributed here under MAA's stated fair-use clause and only for educational and research use. **Downstream users must cite the four MAA source books listed above.**
- The cleaned LaTeX version of the original solutions is a derivative work whose changes (Unicode → LaTeX normalisation) are released under CC BY 4.0; the underlying text remains MAA-copyrighted.


## Links

- **Paper (arXiv)**: <https://arxiv.org/abs/2508.08833>
- **GAP framework code & evaluation pipeline (GitHub)**: <https://github.com/YurenHao0426/GAP>
- **Hugging Face dataset (this release)**: <https://huggingface.co/datasets/blackhao0426/PutnamGAP>
- **PutnamGAP dataset GitHub mirror**: <https://github.com/YurenHao0426/PutnamGAP>
- **Issues & contact**: <https://github.com/YurenHao0426/GAP/issues>
