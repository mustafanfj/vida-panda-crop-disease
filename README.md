# VIDA + PANDA

**Multi-agent vision-language debate for zero-shot crop disease diagnosis.**

Reproducibility code for the paper **[A multi-agent vision-language debate framework for zero-shot crop disease diagnosis](https://doi.org/10.3389/fpls.2026.1890016)**, *Frontiers in Plant Science* 17:1890016, 2026.

**M. Al Juboori**†, Z. Abbas†, Z. Al Aghbari, F. Ullah, M. Ur Rehman. († equal contribution) United Arab Emirates University.

The framework runs a pool of heterogeneous vision-language models (VLMs) through two stages:

- **VIDA** (Visual Independent Diagnosis Arena) — Round 1. Every agent diagnoses each image independently (crop + disease), with no peer communication. A **non-participant GPT-5 judge** scores each agent's reasoning quality and plausibility.
- **PANDA** (Peer-Assisted Named Debate Arena) — Rounds 2–3. The top-3 agents by Round-1 combined accuracy engage in named, evidence-grounded debate; a performance-weighted vote produces the final consensus. The pipeline records every verdict change, who influenced whom, and whether debate helped or hurt.

Dataset identity is withheld from all prompts, and an anti-sycophancy rule allows a verdict change only when new visual evidence is explicitly cited.

## Results

Zero-shot evaluation on CDDM, N = 1,219 images. *Combined* accuracy means both crop and disease are correct.

| Round 1 agent | Combined accuracy |
|---|---:|
| GPT-4.1 | 0.366 |
| Grok-4.20-Fast | 0.326 |
| GPT-4.1-mini | 0.303 |
| Claude Sonnet 4.5 | 0.186 |
| Qwen-VL | 0.184 |
| Claude Haiku 4.5 | 0.121 |
| Gemma-3n-E4B | 0.098 |

| Debate panel (GPT-4.1, Qwen-VL, Grok-4.20-Fast) | Combined | Crop | Disease |
|---|---:|---:|---:|
| Pre-debate weighted vote | 0.359 | 0.835 | 0.436 |
| Post-debate consensus (Round 3) | 0.360 | 0.829 | 0.436 |

- Debate leaves the panel's consensus unchanged (McNemar *p* = 1.000) but substantially lifts the weaker agent: **Qwen-VL improves from 0.183 to 0.304 Combined (+12.1 points, Wilcoxon *p* < 0.0001)**, correcting 159 disease verdicts against 37 degraded.
- Strong agents are highly stubborn (GPT-4.1 85.8%, Grok-4.20-Fast 98.7%), and GPT-4.1's reasoning is the most cited cause of peer revisions.

## Repository structure

```
.
├── notebooks/
│   ├── 1_vida_tournament.ipynb               # VIDA Round-1 tournament (7 agents) → r1_results.csv, r1_raw_bank.json
│   ├── 1b_vida_analysis_from_frozen_r1.ipynb # Loads a frozen R1 run; GPT-5 judge + dashboard + small-sample robustness (no re-inference)
│   └── 2_panda_debate.ipynb                  # PANDA 3-round debate (top-3 agents) → debate_results.csv, transcripts, analysis figure
├── data/        # place the CDDM images + (optionally) the released frozen-R1 files here — see data/README.md
├── results/     # pipeline outputs are written here
├── requirements.txt
├── .env.example
├── CITATION.cff
└── LICENSE
```

## Agents and providers

| Agent | Model ID | Provider | API key |
|---|---|---|---|
| GPT-4.1 | `gpt-4.1` | OpenAI | `OPENAI_API_KEY` |
| GPT-4.1-mini | `gpt-4.1-mini` | OpenAI | `OPENAI_API_KEY` |
| Grok-4.20-Fast | `grok-4.20-non-reasoning` | xAI | `XAI_API_KEY` |
| Claude-Sonnet | `claude-sonnet-4-5` | Anthropic | `ANTHROPIC_API_KEY` |
| Claude-Haiku | `claude-haiku-4-5` | Anthropic | `ANTHROPIC_API_KEY` |
| Qwen-VL | `Qwen/Qwen3.5-9B` | Together.ai | `TOGETHER_API_KEY` |
| Gemma-3n-E4B | `google/gemma-3n-E4B-it` | Together.ai | `TOGETHER_API_KEY` |
| **Judge** | `gpt-5` | OpenAI | `OPENAI_API_KEY` |

## Setup

```bash
pip install -r requirements.txt
```

The notebooks were developed on **Kaggle** and read API keys via `kaggle_secrets.UserSecretsClient` when available, falling back to environment variables otherwise. To run outside Kaggle, export the keys first:

```bash
cp .env.example .env     # then fill in your keys
export $(grep -v '^#' .env | xargs)
```

Only set the keys for the providers you actually run. `kaggle_secrets` ships with the Kaggle runtime and is not a pip package; outside Kaggle the environment-variable path is used automatically.

## Dataset

The pipeline uses the **CDDM** benchmark (Liu et al., 2024). See `data/README.md` for where to obtain it and how the evaluation subset is sampled (`MAX_PER_CLASS = 25`, giving ~1,219 images across 53 classes and 16 crops). Set the dataset path at the top of each notebook (`DATASET_ROOT` / `R1_DATASET_DIR`).

## Reproducing the results

**Option A — full pipeline (re-runs all inference; incurs API cost):**

1. **`notebooks/1_vida_tournament.ipynb`** — runs all 7 agents on the sampled images. Writes `r1_results.csv` (per-agent predictions) and `r1_raw_bank.json` (raw responses) to `results/`.
2. **`notebooks/2_panda_debate.ipynb`** — reads the Round-1 bank, runs the 3-round debate for the top-3 agents, and writes `debate_results.csv`, `debate_transcripts.json`, and the PANDA analysis figure. Debate resumes from a checkpoint if interrupted.

**Option B — analysis only (no new inference):**

Place a frozen `r1_results.csv` / `r1_raw_bank.json` in `data/` and run **`notebooks/1b_vida_analysis_from_frozen_r1.ipynb`**. It applies the GPT-5 judge (judge calls only), regenerates the VIDA dashboard, and computes the stratified 490-image small-sample robustness check — without repeating the expensive Round-1 tournament. This is the cheapest way to reproduce the VIDA figures/tables.

### Key configuration flags

- `MAX_PER_CLASS = 25` — images sampled per class for the evaluation set.
- `MENTION_DATASET_NAME = False` — withholds the dataset name from every prompt (prevents dataset-identity leakage).
- Judge calls use `max_completion_tokens` with `reasoning_effort="minimal"` and a grow-and-retry loop (required for GPT-5).
- Qwen3.5 via Together requires `extra_body={"chat_template_kwargs": {"enable_thinking": False}}`, injected automatically for `qwen3.5/3.6` models.

### Outputs

| File | Produced by | Contents |
|---|---|---|
| `r1_results.csv` | notebook 1 | per-agent, per-image crop/disease predictions, confidence, correctness |
| `r1_raw_bank.json` | notebook 1 | raw model responses for Round 1 |
| `debate_results.csv` | notebook 2 | per-image R1→R3 verdicts, consensus, baseline, influence, changes |
| `debate_transcripts.json` | notebook 2 | full R2/R3 debate text per image |
| VIDA dashboard / PANDA analysis `.png` | 1b / 2 | the figures used in the paper |

## Citation

```bibtex
@article{aljuboori2026vidapanda,
  title   = {A multi-agent vision-language debate framework for zero-shot crop disease diagnosis},
  author  = {Al Juboori, Mustafa and Abbas, Zeeshan and Al Aghbari, Zayed and Ullah, Farman and Rehman, Mobeen Ur},
  journal = {Frontiers in Plant Science},
  volume  = {17},
  pages   = {1890016},
  year    = {2026},
  doi     = {10.3389/fpls.2026.1890016}
}
```

## License

Released under the MIT License (see `LICENSE`). The CDDM dataset is distributed by its original authors under its own terms.
