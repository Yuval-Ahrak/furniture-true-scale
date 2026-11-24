# furniture-true-scale

**Room photo + furniture listing photo → a generated image of your room with the item at its true, metric size.**

Course project — Deep Generative Models, HIT (lecturer: Sasha Apartsin). Solo project by Yuval.
Status: **proposal stage**. Slides: [`slides/01_proposal.pdf`](slides/01_proposal.pdf) · [`slides/01_proposal.pptx`](slides/01_proposal.pptx)

---

## 1. Motivation
People buying furniture online — especially second-hand — judge from photos and must imagine how the item will look in their room. In a 2025 survey of 1,500 UK online furniture buyers, 60% reported receiving an item that was the wrong size (Fixtuur, *Returniture*, Oct 2025). Image-insertion models make furniture look plausible, but nothing guarantees it is the right size. TrueScale is a decision-support preview, not a guarantee of physical fit.

## 2. Problem statement
**Given** a room photo, a furniture reference photo and the furniture's dimensions W×D×H, **generate** a photorealistic image with the furniture at that metric size, standing on the floor at a chosen spot, with correct perspective.

| | |
|---|---|
| Input | room photo · furniture photo · W×D×H · floor spot · optional one known room length (calibration) |
| Output | room image with the item at true scale; pixels outside the mask unchanged; same item, same shape |
| Modes | **primary:** dimensions given · **extension:** dimensions estimated from the photo, evaluated separately |
| Assumptions | placement area visible, floor identifiable, item rests on the floor, selected categories |

**Research questions:** RQ1 — how well do existing reference-guided insertion methods keep a given metric size? RQ2 — does a mask projected from the metric 3D box improve scale fidelity? RQ3 — does LoRA fine-tuning on metric-box examples improve it further?

Full method, data-yield, measurement-protocol and risk details: [`docs/proposal_details.md`](docs/proposal_details.md).

## 3. Pipeline
1. **Room geometry:** MoGe-2 metric point map + intrinsics → floor plane (RANSAC) → optional rescale from one known length.
2. **Furniture:** SAM 2 cut-out (for IP-Adapter) · given W×D×H → oriented 3D box.
3. **Placement:** box on the floor at the clicked spot → pinhole projection → 2D mask.
4. **Generation:** SDXL-Inpainting + IP-Adapter + **our LoRA** → output image; re-measure the item as a check.

**LoRA:** input = frame with item region masked + true-size box mask + reference crop; target = original frame; loss = standard denoising MSE weighted to the mask; rank ~16 on UNet attention layers. F1 = SDXL base, F2 = SD 1.5 base.
**Reused:** MoGe-2, SAM 2, IP-Adapter, SD models. **Mine:** placement & projection, data pipeline, LoRA training, evaluation.
**Fallbacks:** user-entered room length · SD 1.5 if SDXL runs out of memory · baseline study stands if LoRA gives no gain.

## 4. Data
- **Training pairs, derived from real ARKitScenes scans (not rendered):** raw release has 640×480 RGB, intrinsics, per-frame poses and 3D box labels; CC BY-NC-SA 4.0. Filters: box ≥ 5% of frame, ≤ 30% occluded, reference view ≥ 30° apart and fully visible; 100 random pairs inspected by hand.
- **Splits by room:** train ~70% (~100 rooms → ~5K pairs) · validation ~15% · internal test ~15% · real-world test separate.
- **Real-world test (pilot):** ~20 items, tape-measured W×D×H, a reference photo of each, and before/after photos from a fixed tripod with the item's spot marked by tape. Never used for training or model selection.
- **ABO:** only for the dimension-estimation extension (category size priors).

## 5. Evaluation
- **Main:** projected size error E = |w_gen − w_real| / w_real (and height) against the real after-photo from the same camera and spot — a proxy for physical size error; median over items.
- Image-space silhouette IoU vs the real after-photo · mask compliance on unseen ARKitScenes rooms · MoGe-2 room-scale error with/without calibration · DINO similarity + visual check for identity.
- **Baselines:** G0 cut & paste at scale · G1 Paint-by-Example · G2 AnyDoor · G3 SDXL-Inpaint + IP-Adapter. **Experiments:** E1 SDXL-Inpaint + rough box · E2 + IP-Adapter · E3 + projected metric mask · E4 + LoRA (F1) · E5 SD 1.5 + IP-Adapter + metric mask + LoRA (F2).
- **Proposed targets:** median E ≤ 15% on the real-world test; ≥ 30% lower median E than the strongest off-the-shelf baseline; extension beats the category-mean baseline. Bootstrap CIs at item level.
- **Minimum success:** working pipeline with given dimensions + quantitative baseline study; a null LoRA result is still a valid finding.

## 6. Milestones
wk 7 data access + end-to-end on 3 items · wk 8 baselines · wk 9 interim · wk 10–11 LoRA · wk 12 evaluation + buffer · wk 13 final · repo +2 weeks.

## 7. Related work
See [`docs/related_work.md`](docs/related_work.md).

## 8. Reproducibility
Will be filled in as code lands: environment, data preparation, training and evaluation commands. Relative paths only; secrets go in `.env` (see `.env.example`).

## 9. Team
Yuval — everything (solo project).
