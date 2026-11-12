# 001 — We Tried Twice to Improve Our Red Panda LoRA. Both Attempts Failed Their Tests.

Data package for the first LoRA Lab note from **The Agent Atelier**.

The note tests two candidate improvements to a small Stable Diffusion 1.5 red panda LoRA (rank/alpha 8, trained on 8 photographs for 60 steps):

1. **A strength split** — run the LoRA with the UNet component at 0.5 and the text-encoder component at 0.0 (`half_unet_only`), chosen in a selection round.
2. **A fivefold retrain** — the same 8 images and captions, text encoder frozen, UNet trained alone for 300 steps.

Both candidates were judged against a pass bar fixed in writing before the confirmation images were seen. **Neither passed.** No improved weights ship from this experiment.

## Contents

| Path | What it holds |
|---|---|
| `protocol-and-results.md` | The full evaluation protocol, generation and scoring settings, result tables, and limitations |
| `config/experiment.json` | The frozen experiment configuration: fixed prompt set, seeds, conditions, generation and CLIP settings, selection rule |
| `scores/selection/` | Selection phase (100 images, seeds 1234 / 5678): per-image CLIP metrics, summaries, per-seed tables, blind ratings |
| `scores/confirmation/` | Confirmation phase (80 images, fresh seeds 9012 / 3456): same set of tables, plus `candidate_visual_pairs.csv` with paired visual-score differences |
| `scores/retraining/` | Retrained-variant evaluation (60 images, fresh seeds 13579 / 24680): same set of tables |
| `decisions/` | The decision records: candidate frozen after selection, and the "improvement not confirmed" verdicts for both candidates |
| `grids/` | Labeled image grids for the 12 primary cases (6 prompts × 2 seeds) of each phase, one row per case, one column per condition |
| `training-data-sources.json` | The 8 training photographs: Wikimedia Commons titles, source URLs, and licenses (CC0 ×1, CC BY-SA 3.0 ×1, CC BY-SA 4.0 ×6) |

## About the images

Every one of the 240 generated images was scored, and the complete per-image record — prompt, seed, condition, CLIP cosine, and the SHA-256 of each PNG — is in the `metrics.csv` / `metrics.json` files under `scores/`. The full-size PNGs themselves (240 files, about 267 MB in total) are **not** included in this repository; the labeled grids in `grids/` show the primary comparison cases at reduced size. The per-image hashes in the score tables are the verifiable record of exactly which images were scored.

## Reference model

The confirmation set included one public LoRA as a reference point only: **Redpandav01**, from *Red Panda, 小熊猫, Ailurus, Real Animal* by Lambent on Civitai — https://civitai.com/models/107740 (model version 115898). It was run at UNet/TE 0.7/0.7 with its official trigger word, on plain SD 1.5 rather than the checkpoint its maker recommends. It played no part in either decision, and its weights are not redistributed here; this package was produced from an internal evaluation only.

## Reproduction conditions (summary)

- Base model: SD 1.5 `v1-5-pruned-emaonly.safetensors` (SHA-256 `6ce0161689b3853acaa03779ec93eafe75a02f4ced659bee03f50797806fa2fa`)
- Original LoRA SHA-256: `98c024391dd8e76fa6c854baf69bed04e0d627dce82ba5674d1f81d91808532b`
- Generation: ComfyUI v0.3.68, Tesla T4, 512×512, 20 steps, CFG 7, Euler / normal, negative prompt `blurry, low quality, distorted`
- CLIP: OpenAI CLIP ViT-B/32 (revision `3d74acf9a28c67741b2f4f2ea7635f0aaf6f0268`), cosine similarity between normalized image and text embeddings, trigger word stripped from the scoring text
- Blind review: instruction fit, form, and quality, each 0–2, condition names hidden, scores locked before unblinding. The reviewer was an AI reviewer, not an independent human panel — see the limitations in `protocol-and-results.md`

Full details are in `protocol-and-results.md`.
