# Phone photo → studio packshot (Neural Wave 2026)

Team **AACtivEtion Function**'s pipeline for the Sartiq challenge at the Neural Wave 2026 hackathon (USI Lugano, 16–18 October 2026).

**Task:** turn amateur smartphone photos of a garment into a clean e-commerce flat-lay packshot on a white background. Submissions were scored on similarity to a hidden studio reference: colour 40%, LPIPS 25%, CLIP 15%, silhouette IoU 15%, SSIM 5%.

## Approach

1. **Multi-view input.** The front photo plus up to N detail or back shots go to a hosted image-editing model, with a packshot prompt that tells it to keep exact colour, texture, print and proportions.
2. **Several candidates per garment.** Each garment is generated K times on its own, and candidates are cached so a rerun never pays twice.
3. **Consensus selection.** Each candidate is scored on:
   - **Colour drift:** distance in CIELAB from the group's median garment colour, so random colour drift from the generator cancels out
   - **Symmetry:** left–right overlap of the segmented garment mask
   - **Tilt:** angle of the mask's principal axis (via PCA)

   The candidate with the lowest combined penalty is kept, and manual overrides are possible.
4. **Robust submission.** Garments that fail fall back to an earlier run, so no garment scores zero.

## Files

| File | What it does |
|---|---|
| `pipeline.py` | End-to-end pipeline: multi-view generation, candidate caching, consensus selection, fallback filling |
| `colour_tools.py` | Lab-space colour analysis: diagnose the generator's colour bias, calibrate and apply corrections, combine AI and phone-photo colour estimates, select candidates |
| `post_experiment.py` | Experiment testing whether cheap post-processing (e.g. removing low-frequency shading) raises the official score |
| `make_ours_manifest.py` | Builds a manifest from the team's own on-site captures (best front shot first, then detail shots) |

## Running it

These scripts plug into the organisers' starter repository ([shootify-io/neural-wave-2026](https://github.com/shootify-io/neural-wave-2026)). It provides the hosted-model wrapper (`baselines/baseline_nanobanana.py`) and the segmenter (`sartiq_score`), which are not included here.

1. Clone the starter repo and follow its setup steps.
2. Copy the files from this repository into its root.
3. Set your fal.ai key as `FAL_KEY`, then run:

```bash
.venv/bin/python pipeline.py --manifest ../captures/ours.jsonl --dataset ../captures \
  --out submissions_final --team aactivetion_function --candidates 3 --max-inputs 3
```

## My contribution
Pipeline code, running the generation pipeline, parameter sensitivity analysis and the submissions.

## Team
Qi An · Arianna Silvestri · Eduardo Trabattoni · Andrea Gemetti · Mahdi Ameridaragheh

## Notes
The challenge dataset, the organisers' scorer and the baseline code are not included, in line with the hackathon terms. This repository holds only the team's own code.
