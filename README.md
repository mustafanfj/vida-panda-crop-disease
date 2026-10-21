# VIDA + PANDA — Multi-Agent VLM Crop Disease Diagnosis

> **Paper:** [*A multi-agent vision-language debate framework for zero-shot crop disease diagnosis*](https://doi.org/10.3389/fpls.2026.1890016), **Frontiers in Plant Science** 17:1890016, 2026. DOI: [10.3389/fpls.2026.1890016](https://doi.org/10.3389/fpls.2026.1890016)
>
> **Authors:** Mustafa Al Juboori†, Zeeshan Abbas†, Zayed Al Aghbari, Farman Ullah, Mobeen Ur Rehman
> († equal contribution)
>
> **Institution:** United Arab Emirates University (UAEU)

---

## Framework Overview

This repository contains the complete implementation of **VIDA+PANDA**, a two-stage multi-agent Vision-Language Model framework for zero-shot crop disease diagnosis.

```
CDDM Images
    │
    ▼
┌─────────────────────────────────────────────┐
│  VIDA — Varied Independent Diagnostic       │
│  Assessment                                 │
│  6 agents × Round 1 (independent)          │
│  → GPT-4.1 judge scores RQ + Plausibility  │
│  → Softmax weights computed (T=2.0)        │
│  → Pre-debate baseline vote                 │
└──────────────────────┬──────────────────────┘
                       │ Top-3 selected
                       ▼
┌─────────────────────────────────────────────┐
│  PANDA — Peer-Anchored Named Deliberation   │
│  Architecture                               │
│  Round 2: Named peer debate                 │
│    - AGREE/DISAGREE by agent name           │
│    - New Visual Evidence required           │
│    - Influenced By field (anti-sycophancy)  │
│  Round 3: Final verdict                     │
│  → Softmax-weighted R3 consensus            │
└─────────────────────────────────────────────┘
```

## Agents

| Agent | Provider | Model String | Role |
|---|---|---|---|
| GPT-4.1 | OpenAI | `gpt-4.1` | Anchor (highest accuracy) |
| Grok-4.20-Fast | xAI | `grok-4.20-non-reasoning` | High accuracy, very stubborn |
| GPT-4.1-mini | OpenAI | `gpt-4.1-mini` | Mid-tier, most improved by debate |
| Claude Sonnet 4.5 | Anthropic | `claude-sonnet-4-5` | Most persuadable |
| Claude Haiku 4.5 | Anthropic | `claude-haiku-4-5` | Lightweight Anthropic |
| Gemma-3n-E4B | Together.ai | `google/gemma-3n-E4B-it` | Open-weight via Together.ai |

## Dataset

**CDDM** (Liu et al., 2024) — Crop Disease Diagnosis Multi-Modal benchmark  
- 54 disease categories × 16 crop species  
- Evaluation subset: **490 images**, 53 classes, stratified (80% diseased / 20% healthy), seed=42  
- Dataset path on Kaggle: `/kaggle/input/datasets/maljuboori/cddm-dataset/images`

## Notebook

**`vida_panda_cddm.ipynb`** — Single notebook, 33 cells

| Cells | Phase | Description |
|---|---|---|
| 1–9 | Setup | Install, API keys, taxonomy, dataset, prompts, parser, agents, voting, judge |
| 10 | VIDA | Round 1: all 6 agents independent, saves raw bank + checkpoint |
| 11 | VIDA | GPT-4.1 judge scoring (Reasoning Quality + Plausibility) |
| 12 | VIDA | 9-panel dashboard, metrics table |
| 13 | Config | Agent selection (auto top-3 or manual) + softmax weight computation |
| 14 | PANDA | Rounds 2 + 3 named debate, pre/post baseline comparison |
| 15 | Analysis | Influence matrix, stubbornness, flip-flop, 6-panel visualization |
| 16 | Summary | Full report with all metrics and output files |

## Kaggle Secrets Required

Add these in **Settings → Secrets** before running:

| Secret | Used for |
|---|---|
| `OPENAI_API_KEY` | GPT-4.1, GPT-4.1-mini, GPT-4.1 judge |
| `ANTHROPIC_API_KEY` | Claude Sonnet 4.5, Claude Haiku 4.5 |
| `TOGETHER_API_KEY` | Gemma-3n-E4B via Together.ai |
| `XAI_API_KEY` | Grok-4.20-Fast via xAI API |

## Key Design Decisions

**Softmax-weighted voting (T=2.0)**  
Agents vote on final consensus weighted by their R1 Combined Accuracy. Temperature 2.0 produces a ~2:1 weight ratio for a 0.8 vs 0.5 accuracy gap — meaningful but not silencing.

**Pre-debate baseline**  
The same softmax weights are applied to R1 verdicts before any debate, giving a true apples-to-apples comparison. `baseline_both_correct` vs `consensus_both_correct` is the clean measure of debate's causal effect.

**Anti-sycophancy: New Visual Evidence field**  
Agents may only change their verdict if they can cite a specific new visual observation not mentioned in R1. Verdicts that flip without new evidence are flagged as unsupported agreement.

**3-pass parser**  
1. Exact match against taxonomy labels  
2. Substring match (longest label wins)  
3. Word-overlap scoring on full response text

**Image delivery for Together.ai**  
Gemma-3n-E4B serverless endpoint does not accept inline base64. Images are uploaded to `catbox.moe` to obtain a public URL before each API call.

## Outputs

```
vida_panda_results/
├── vida_r1_results.csv       # Per-agent R1 results + judge scores
├── vida_raw_bank.json        # All raw API responses (R1, all agents)
├── vida_metrics.csv          # Per-agent VIDA metrics (9 columns)
├── vida_dashboard.png        # 9-panel VIDA dashboard
├── panda_results.csv         # Per-image PANDA results (pre/post baseline)
├── panda_transcripts.json    # Full R1/R2/R3 raw transcripts
├── influence_matrix.csv      # Who persuaded whom, how many times
└── panda_debate_analysis.png # 6-panel PANDA visualization
```

## Results (from the paper)

Zero-shot evaluation on CDDM, N = 1,219 images. *Combined* accuracy means both crop and disease are correct.

**Round 1 (VIDA, independent agents)**

| Agent | Combined accuracy |
|---|---|
| GPT-4.1 | 0.366 |
| Grok-4.20-Fast | 0.326 |
| GPT-4.1-mini | 0.303 |
| Claude Sonnet | 0.186 |
| Qwen-VL | 0.184 |
| Claude Haiku | 0.121 |
| Gemma-3n-E4B | 0.098 |

**Debate (PANDA, panel: GPT-4.1, Qwen-VL, Grok-4.20-Fast)**

| | Combined | Crop | Disease |
|---|---|---|---|
| Pre-debate weighted vote (Round 1) | 0.359 | 0.835 | 0.436 |
| Post-debate consensus (Round 3) | 0.360 | 0.829 | 0.436 |

- Debate leaves the panel's consensus unchanged (McNemar *p* = 1.000), but it substantially lifts the weaker agent: **Qwen-VL improves from 0.183 to 0.304 Combined (+0.121, Wilcoxon *p* < 0.0001)**, correcting 159 disease verdicts against 37 degraded.
- Strong agents are highly stubborn (GPT-4.1 85.8%, Grok-4.20-Fast 98.7%), and GPT-4.1's reasoning is the most cited cause of peer revisions.

See the paper for per-class results, the influence analysis and the 490-image ablation.

## Citation

```bibtex
@article{aljuboori2026vidapanda,
  title   = {A multi-agent vision-language debate framework for
             zero-shot crop disease diagnosis},
  author  = {Al Juboori, Mustafa and Abbas, Zeeshan and
             Al Aghbari, Zayed and Ullah, Farman and
             Rehman, Mobeen Ur},
  journal = {Frontiers in Plant Science},
  volume  = {17},
  pages   = {1890016},
  year    = {2026},
  doi     = {10.3389/fpls.2026.1890016},
  note    = {Co-first authors: Mustafa Al Juboori and Zeeshan Abbas}
}
```

## Contact
Corresponding author: Mobeen Ur Rehman — mobeen.rehman@uaeu.ac.ae

## License

Code released under the [MIT License](LICENSE).
