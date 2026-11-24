# furniture-true-scale

**Room photo + furniture listing photo → a generated image of your room with the item at its true, metric size.**

Course project — Deep Generative Models, HIT (lecturer: Sasha Apartsin). Solo project by Yuval.
Status: **proposal stage**. Slides: [`slides/01_proposal.pdf`](slides/01_proposal.pdf) · [`slides/01_proposal.pptx`](slides/01_proposal.pptx)

---

## 1. Motivation
Buying furniture from a photo is a size gamble: in a 2025 survey of 1,500 UK online furniture buyers, 60% had bought an item that turned out to be the wrong size and 49% one that didn't fit through the door (Fixtuur, *Returniture*, 2025). Second-hand listings often omit dimensions, and AR "view in room" only works when the retailer supplies a true-scale 3D model. Image-insertion models can place furniture into a room photo realistically — but nothing guarantees the inserted item is the right size.

## 2. Problem statement
**Given** an RGB room photo and an RGB photo of a furniture item (optionally its listed dimensions), **generate** a photorealistic image of the room containing that item at its correct metric size, standing on the floor in the right perspective.

| | |
|---|---|
| Input | room photo; furniture photo (catalog or second-hand); target floor spot; optional listed W×D×H |
| Output | generated room image at true scale; estimated W×D×H ± interval; fit check |
| Constraints | RGB only (no depth sensor), unknown camera, free GPUs |

## 3. Visual abstract
*To be added to `visuals/`.* Room: MoGe-2 → metric floor + camera. Furniture: SAM 2 → size (geometry + category prior, or listed dims). Generation: true-size 3D box on the floor → projected mask → SDXL inpainting + IP-Adapter + **our scale-exact LoRA** → output → re-measure item.

## 4. Data (planned)
- **Training — ARKitScenes** (real room scans with metric 3D boxes for 17 furniture categories): self-supervised pairs — project a furniture item's 3D box into a frame → mask; input = frame with region removed; reference = same item from another frame; target = original frame. ~200 scenes → 10–20K pairs, split by scene. *Check in week 7: frame resolution, license.*
- **Size priors — ABO** (Amazon Berkeley Objects, CC BY-NC 4.0): per-category dimension distributions.
- **Real test — collected by me:** ~20 before/after pairs (spot photographed empty, then with the real, tape-measured item) and ~30 measured items with listing-style photos. Never used for training.

## 5. Models and pipeline (planned)
| Step | Choice | Alternatives |
|---|---|---|
| Segmentation | SAM 2 | Grounded-SAM |
| Room / object geometry | MoGe-2 | Depth Pro, UniDepth V2 |
| Furniture size | geometry extents + ABO category priors; listed dims override | zero-shot VLM, trained regressor |
| Generation | SDXL-Inpainting + IP-Adapter | FLUX.1 Fill, AnyDoor |
| **Trained model** | **LoRA on the inpainting UNet, scale-exact ARKitScenes pairs** | SD 2 inpainting if memory-bound |

## 6. Evaluation (planned)
- Baselines: G0 cut & paste at scale · G1 Paint-by-Example · G2 AnyDoor · G3 SDXL-Inpaint + IP-Adapter without LoRA. Size: category mean, zero-shot VLM.
- Metrics: scale fidelity (IoU of generated item with the projected box; re-measured size error), IoU / height ratio vs real "after" photos, DINO identity similarity, KID, small blind user study, size-estimate relative error.
- Success criteria (set before results): LoRA cuts scale error vs G3 by ≥ 30% with identity similarity within 5%; end-to-end median size error ≤ 15% on real pairs; size estimate beats category mean.

## 7. Related work
See [`docs/related_work.md`](docs/related_work.md).

## 8. Reproducibility
Will be filled in as code lands: environment, data preparation, training and evaluation commands. Relative paths only; secrets go in `.env` (see `.env.example`).

## 9. Team
Yuval — everything (solo project).
