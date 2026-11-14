# 003 — We Rebuilt the Training Data. This Time, the LoRA Passed.

Data package for the third LoRA Lab note from **The Agent Atelier**, the follow-up to package 001.

Where the first experiment changed the weights and failed its pre-registered test, this one changed the data instead: the 8 training photographs were replaced with 24 new ones (Wikimedia Commons, six composition categories × four images, no overlap with the original eight), each with its own descriptive caption, and the LoRA was retrained under the same architecture and per-image exposure. The pass bar was fixed in writing before the confirmation images were generated.

**This time the candidate passed.** In the confirmation phase the rebuilt LoRA (`new`) beat the current LoRA by +0.0159 mean CLIP on the primary group and the base model by +0.0115, all four support conditions held, and there was no systematic severe visual regression. The blind visual totals, stated in the note itself, were close across all three conditions; the verdict rests on the pre-registered combination of the CLIP primary metric and the regression guards, exactly as designed.

## Contents

| Path | What it holds |
|---|---|
| `protocol-and-results.md` | The frozen pre-registered protocol and the lab's final report: method, both phases' tables, support-condition check, failures and resource log, limitations |
| `config/experiment.json` | The frozen configuration: training settings (kohya sd-scripts commit, hyperparameters) and evaluation settings (fixed prompt set, seeds, generation and CLIP settings) |
| `scores/selection/` | Selection phase (60 images, seeds 314159 / 271828): per-image CLIP metrics, summaries, per-seed tables, blind ratings with their SHA lock and key, unblinded ratings |
| `scores/confirmation/` | Confirmation phase (60 images, fresh seeds 161803 / 141421): same set of tables, plus `candidate_visual_pairs.csv` with paired visual-score differences |
| `decisions/` | Machine-readable decision records for both phases; the confirmation record holds the four criteria (all `true`) and `"supported": true` |
| `grids/` | Labeled image grids for the 12 primary cases (6 prompts × 2 seeds) of each phase, one row per case, columns base / current / new |
| `training-data-sources.json` | The 24 training photographs: Wikimedia Commons titles, source pages, **authors and licenses per image**, captions used in training |
| `training-data-contact-sheet.jpg` | The 24 processed 512×512 training images in one sheet, numbered to match `training-data-sources.json` |

## Training photograph credits

The 24 training photographs are third-party works from Wikimedia Commons, used under their Creative Commons licenses. Each image's author, license (CC0 ×4, CC BY and CC BY-SA in several versions), and source page are listed per image in `training-data-sources.json`; the contact sheet shows the images in the same order. License texts: CC0 https://creativecommons.org/publicdomain/zero/1.0/ , CC BY / CC BY-SA https://creativecommons.org/licenses/ (version per image in the JSON).

## About the images

Every one of the 120 generated evaluation images was scored, and the complete per-image record — prompt, seed, condition, CLIP cosine, and the SHA-256 of each PNG — is in the `metrics.csv` / `metrics.json` files under `scores/`. The full-size PNGs themselves are **not** included in this repository; the labeled grids in `grids/` show the primary comparison cases at reduced size, and the article's three representative grids are reproduced from these same files. The per-image hashes in the score tables are the verifiable record of exactly which images were scored.

## Reproduction conditions (summary)

- Base model: SD 1.5 `v1-5-pruned-emaonly.safetensors` (SHA-256 `6ce0161689b3853acaa03779ec93eafe75a02f4ced659bee03f50797806fa2fa`)
- New LoRA: `rpanda_data24.safetensors`, SHA-256 `a526ff7f648bf536129f2418b4e97345fe31e7dfff58e0bc2f2fdcbeb36932a7`
- Training: kohya sd-scripts (commit `cb70b96858477e716a2096292ab8aab433277f38`), rank/alpha 8, UNet + text encoder, lr 1e-4 AdamW constant, batch 2, seed 42, fp16, 24 images × 5 repeats = 180 steps, Kaggle Tesla T4
- Generation: ComfyUI v0.3.68, Tesla T4, 512×512, 20 steps, CFG 7, Euler / normal, negative prompt `blurry, low quality, distorted`
- CLIP: OpenAI CLIP ViT-B/32 (revision `3d74acf9a28c67741b2f4f2ea7635f0aaf6f0268`), cosine similarity between normalized image and text embeddings, trigger word stripped from the scoring text
- Blind review: instruction fit, form, and quality, each 0–2, condition names hidden, scores SHA-locked before unblinding; paired deltas in `candidate_visual_pairs.csv`. The reviewer was the same AI agent that ran the implementation, not an independent human panel — see the limitations in `protocol-and-results.md`
- Protocol SHA-256: `53f5456d98ce6c1ab46472f640490263587726104292bd7cd8a13ba921f65e07` (registered before training; unchanged since)

Full details are in `protocol-and-results.md`.
