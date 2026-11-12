# Protocol and results

Experiment: two candidate improvements to a small SD 1.5 red panda LoRA, evaluated in three phases with strictly separated seeds. Both candidates failed a pass bar that was fixed in writing before the confirmation images were seen. No improved weights were produced.

## The LoRA under test

- Base model: Stable Diffusion 1.5, `v1-5-pruned-emaonly.safetensors`, SHA-256 `6ce0161689b3853acaa03779ec93eafe75a02f4ced659bee03f50797806fa2fa`
- LoRA: rank/alpha 8, trained for 60 steps on 8 red panda photographs from Wikimedia Commons (CC0 ×1, CC BY-SA 3.0 ×1, CC BY-SA 4.0 ×6; sources and licenses in `training-data-sources.json`), single caption style. SHA-256 `98c024391dd8e76fa6c854baf69bed04e0d627dce82ba5674d1f81d91808532b`
- "Current" in all tables means this LoRA applied at UNet 1.0 / text encoder 1.0. "Base" means the base model with the LoRA node removed entirely.

## Candidates

1. **Strength split (`half_unet_only`)** — the LoRA applied at UNet 0.5 / text encoder 0.0. It won a small selection grid over the alternatives `half` (0.5 / 0.5) and `unet_only` (1.0 / 0.0), and was frozen for independent confirmation without re-selection.
2. **Retrain (`retrained`)** — same 8 images, same captions, same base, rank/alpha 8, UNet learning rate 1e-4, batch 2, training seed 42, but UNet only (text encoder not trained) and 300 steps instead of 60. Retrained file SHA-256 `b1f3c2fd1492423c80079ab1eea4d8be09e0022a0ccd437ab886f90ef3a05e24`. This is a two-factor change (steps and frozen text encoder); the design cannot separate which factor caused what.

## Phases and seed separation

| Phase | Seeds | Conditions | Images |
|---|---|---|---:|
| Selection | 1234, 5678 | base / current / half / unet_only / half_unet_only | 100 |
| Confirmation | 9012, 3456 | base / current / half_unet_only / Redpandav01 (reference) | 80 |
| Retrain evaluation | 13579, 24680 | base / current / retrained | 60 |

Total: 240 images, 240 CLIP scores, 240 blind ratings. No seed was reused across phases.

## Prompts

Ten fixed prompts: three near-training photographic prompts (bamboo grove, branch, face close-up), three generalization prompts (snow, watercolor river, seaside at dusk), two controls (a dog, a landscape), and two trigger-ablation prompts (the bamboo and snow prompts without the trigger word). The primary comparison is the 6 photographic/scene prompts × 2 seeds = 12 matched cases per condition; controls and ablations are reported separately and never folded into the primary number. The exact prompt texts are in `config/experiment.json`.

## Generation settings

ComfyUI v0.3.68 (commit `265adad858e1f31b66cd3523a02b16f5d34ced52`), torch 2.5.1+cu121, Tesla T4. 512×512, batch 1, 20 steps, CFG 7, Euler sampler, normal scheduler, denoise 1.0, negative prompt `blurry, low quality, distorted`, no extra clip skip. Strengths: current UNet/TE 1.0/1.0, candidates as above, reference model 0.7/0.7. Within each phase, every condition shares the same prompts and seeds.

## Scoring

**CLIP.** OpenAI CLIP ViT-B/32, revision `3d74acf9a28c67741b2f4f2ea7635f0aaf6f0268`, transformers 4.46.3, CPU float32. The score is the cosine similarity between the normalized image and text embeddings (no logit scale, no softmax). The scoring text is the prompt with the leading trigger word removed, identical across conditions, so the trigger itself is never scored.

**Blind review.** Every image was scored for instruction fit, form (subject anatomy/structure), and quality, each 0 = bad, 1 = mixed, 2 = good, with condition names and all numbers hidden. For landscapes, form was scored 2 unless there was an obvious structural failure. In the confirmation phase, all 80 images were viewed on 10 comparison sheets and the ratings were locked (hash-recorded) before the conditions and CLIP scores were unblinded. The reviewer was an AI reviewer, not an independent human panel; see Limitations.

**Pass bar (fixed before confirmation images were seen).** A candidate is supported only if all of these hold: primary CLIP beats both current and base by more than +0.001; the difference is positive on both seeds against both comparisons; the control-group mean stays within −0.01 of current; and there is no systematic severe visual regression. The same bar was applied to the retrained variant.

**Reference intervals.** 95% intervals are percentile bootstrap intervals (5,000 resamples, random seed 8841) over prompt-level mean differences (each prompt's 2-seed average is one cluster). The sample is small; these are reference intervals, not significance tests and not proof of a general effect.

## Results

### Selection (seeds 1234, 5678)

The frozen candidate `half_unet_only` scored primary CLIP 0.3631, +0.0039 against current and +0.0037 against base. It led the runner-up by 0.0016, more than the 0.001 tie width, and its control delta (−0.0059) stayed inside the hold threshold, so it was frozen for confirmation. Two warning signs were already visible: the per-seed deltas against current disagreed in sign, and its blind form scores were below current.

### Confirmation (fresh seeds 9012, 3456) — primary group, 12 cases

| Condition | Mean CLIP | Δ vs current | Δ vs base | Blind total /6 | Form /2 | Fit /2 |
|---|---:|---:|---:|---:|---:|---:|
| base | 0.3635 | +0.0013 | 0 | 5.08 | 1.33 | 1.92 |
| current | 0.3623 | 0 | −0.0013 | 5.25 | 1.50 | 1.83 |
| half_unet_only | 0.3592 | −0.0031 | −0.0044 | 5.08 | 1.33 | 1.75 |
| Redpandav01 (reference) | 0.3635 | +0.0013 | −0.0000 | 5.42 | 1.92 | 1.58 |

The candidate's 95% reference interval was [−0.0121, +0.0045] against current and [−0.0105, +0.0014] against base. It won 6 of 12 paired cases against current and 3 of 12 against base. Per-seed deltas flipped sign (seed 9012: +0.0072 vs current; seed 3456: −0.0134). **Verdict: improvement not confirmed** — it fails even a simple mean-improvement rule, not only the stricter pre-registered bar. The frozen decision record is `decisions/confirmation_decision.json`.

Candidate results by prompt group:

| Group | n | Mean CLIP | Δ vs current | Δ vs base |
|---|---:|---:|---:|---:|
| Near-training | 6 | 0.3466 | −0.0062 | +0.0017 |
| Generalization | 6 | 0.3718 | −0.0000 | −0.0105 |
| Control | 4 | 0.3240 | −0.0023 | +0.0003 |
| Trigger ablation | 4 | 0.3514 | +0.0060 | +0.0012 |
| All (diagnostic) | 20 | 0.3506 | −0.0011 | −0.0023 |

Controls stayed within threshold and the candidate produced no form score of 0, but on the primary group its form and instruction fit were both below current; a quality-only gain would not have counted as improvement in any case. Recurring defects across conditions included multiple or overlapping bodies, finger-like artifacts, still poses in snow with oversized bodies, and missing river or dusk-seaside content.

### Retrain evaluation (fresh seeds 13579, 24680) — primary group, 12 cases

| Condition | Mean CLIP | Δ vs current | Δ vs base | Blind total /6 | Form /2 | Fit /2 |
|---|---:|---:|---:|---:|---:|---:|
| base | 0.3472 | −0.0015 | 0 | 4.67 | 1.17 | 1.75 |
| current | 0.3487 | 0 | +0.0015 | 4.50 | 1.17 | 1.58 |
| retrained | 0.3384 | −0.0104 | −0.0089 | 4.92 | 1.58 | 1.33 |

The retrained variant's 95% reference interval was [−0.0219, +0.0013] against current and [−0.0259, +0.0067] against base. It won 5 of 12 cases against current and 4 of 12 against base, and its CLIP delta against current was negative on both seeds. **Verdict: improvement not confirmed** (`decisions/retraining_decision.json`).

Retrained variant by prompt group:

| Group | n | Mean CLIP | Δ vs current | Δ vs base |
|---|---:|---:|---:|---:|
| Near-training | 6 | 0.3382 | −0.0058 | −0.0091 |
| Generalization | 6 | 0.3385 | −0.0150 | −0.0087 |
| Control | 4 | 0.3144 | −0.0033 | +0.0031 |
| Trigger ablation | 4 | 0.3550 | +0.0111 | +0.0089 |
| All (diagnostic) | 20 | 0.3369 | −0.0047 | −0.0029 |

Visually, form and quality did improve (form 1.58 vs 1.17 for current on the primary group; quality 2.00 vs 1.75; zero form scores of 0, against one each for base and current). Instruction fit fell (1.33 vs 1.58): a requested close-up kept coming back as a full body on a branch, a requested walking pose kept coming back as a still pose in a tree, and the dusk seaside was weak. A form-for-obedience trade is not an overall improvement, so the retrained weights stay labeled experimental and unconfirmed. No strength search and no second retrain were run.

### The reference model

One public LoRA was run in the confirmation set as a reference point only: **Redpandav01**, from *Red Panda, 小熊猫, Ailurus, Real Animal* by Lambent — https://civitai.com/models/107740, model version 115898 (v1.0). File `Redpandav01.safetensors` (37,865,674 bytes), SHA-256 `C1707274C1D8E7D181F51CF4B5F36C897E1B03DA8A0FEDCB8B75902508A58182`. It was run at UNet/TE 0.7/0.7, the middle of its maker's recommended 0.6–0.8 range, and only the leading trigger was replaced with its official trigger `redpanda`; prompt content, negative prompt, controls, and ablation texts were otherwise unchanged. Its metadata describes training on 93 images at dim 32 / alpha 16 for 16,740 steps with clip skip 2, and its page recommends realistic checkpoints; this run used plain SD 1.5 with the fixed settings above, which is not the maker's preferred environment. Its primary CLIP matched base, its form scores were the best of the set, and its instruction fit was the weakest. None of its numbers were used in either candidate decision, and no conclusion about the model in general should be drawn from this run. The weights were obtained for internal evaluation only and are not redistributed in this package.

## The uncomfortable baseline

On the confirmation set, the base model with no LoRA attached scored 0.3635, level with the reference model and a hair above the LoRA under test (0.3623). At 8 images and 60 steps, the LoRA's contribution is small, and a small contribution leaves weight-side tuning with very little to work on. This is why the follow-up experiment changes the data — more training images and varied captions — rather than the weights, with fresh evaluation seeds fixed in advance and the same pass bar.

## Limitations

- The blind review was performed by an AI reviewer, not by independent humans at full resolution. A scorer may also be able to guess a condition from image style despite blinding.
- Samples are small (12 primary paired cases per condition). Reference intervals that include zero are not proof that no effect exists, and two-seed agreement is weak evidence either way.
- Whether the retraining images were byte-identical to the original training downloads is unverified; sources and captions matched.
- CLIP cosine does not measure anatomy, memorization, or individual identity. No direct measurement of training-image memorization was made.
- Results are specific to this base model, this prompt set, and these settings; they do not generalize to other checkpoints, wider prompt sets, or many-seed evaluations.
- GPU cost in money terms was not recorded.

## Audit summary

All 240 images were accounted for in each phase: file decoding, dimensions, and hashes verified; every prompt/seed combination present for every condition; CLIP scores matched to images; blind ratings locked by hash before unblinding; the frozen selection record was not modified after selection. Failed smoke-test generations (2 images in the selection phase) were excluded before scoring and are not in any table.
