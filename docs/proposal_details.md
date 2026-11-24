# Proposal details — TrueScale

These details back the 5-slide proposal. The slides are deliberately compact.

## 1. Geometry → generation interface
- **Room:** MoGe-2 → metric point map + camera intrinsics. Floor plane fitted with RANSAC. Optional global rescale from one user-entered known length (wall or door). This corrects global scale only, not local distortion; local errors are measured separately.
- **Furniture:** given W×D×H → 3D box. The user sets the orientation; the box rests on the floor at the clicked point.
- **Projection:** pinhole camera model → silhouette of the 3D box in the image → binary mask.
- **What the diffusion model receives:**
  - The binary mask, through the inpainting UNet's mask channel. This is the only geometric input in the primary setup.
  - The reference cut-out, through IP-Adapter, which carries identity.
  - Training and inference use the same conditioning.
- **Stretch option**, used only if mask compliance is low at the week-8 go/no-go: add the projected box edges as a ControlNet condition.

## 2. LoRA training
- **Input:** ARKitScenes frame with the item's projected-box region masked, plus that mask, plus a reference crop of the same item from another frame (IP-Adapter).
- **Target:** the original frame.
- **Loss:** standard denoising MSE, weighted to the mask.
- **Why this should help scale:** in every target, the real item fills its annotated metric box. A model that shrinks or stretches the item inside the mask is therefore penalised by the reconstruction loss.
- **What the loss does not guarantee:** a low loss does not by itself mean a low physical size error.
  - Checkpoints are selected by validation mask compliance.
  - Scale fidelity is evaluated independently.
- **Setup:** LoRA rank ~16 on UNet attention layers; base model frozen.
  - F1 = SDXL-Inpaint + IP-Adapter (primary).
  - F2 = SD 1.5-Inpaint + IP-Adapter (resource-constrained alternative, same protocol).

## 3. Dataset construction
**Planned data yield (to be measured):**

| Stage | Planned measurement |
|---|---|
| Raw rooms | ~100 (raw ARKitScenes: 640×480 RGB, intrinsics, per-frame poses, 3D box labels) |
| Candidate furniture instances | to be measured |
| Valid reference–target pairs | to be measured |
| Pairs after quality filtering | target ~5,000 |
| Train / validation / internal test | ~70 / 15 / 15 % of rooms |

**Filters:**
- The box covers at least 5% of the frame.
- The item is at most 30% occluded.
- The reference view is at least 30° from the target view, and the item is fully visible in it.

**Leakage control:**
- Splits are made by room.
- All reference–target pairs are built within one partition.
- Duplicate frames and repeated object instances are checked across partitions, where instance identifiers exist.

**Manual inspection:** 100 random pairs. A pair is rejected for any of the following:
- wrong identity
- misaligned box
- wrong projection
- reference not visible
- excessive occlusion
- no floor contact
- corrupted masked region

The rejection rate is reported.

**Known limitation:** ARKitScenes-derived pairs give controlled geometric supervision, but scans are not consumer phone photos. The real-world test is therefore separate and is never used for training or model selection.

**License:** CC BY-NC-SA 4.0. The project is non-commercial, cites the dataset, and shares derived data under the same license.

## 4. Real-world measurement procedure (pilot, ~20 items, sofas and tables first)
1. Fix the phone on a tripod: position, orientation and zoom.
2. Photograph the empty spot.
3. Mark the item's footprint and orientation on the floor with tape.
4. Run every method with identical inputs.
5. Place the real item on the marks.
6. Photograph again without touching the camera.
7. Register the generated image and the real image on static background features.
8. Segment the item in both images with SAM 2, then check by hand.
9. Compute E = |w_gen − w_real| / w_real, and the same for height. Report per item.
10. Report the median, with an item-level bootstrap confidence interval. Registration and segmentation uncertainty are noted per item.

**What E measures:** projected size consistency under an identical camera and placement. In this controlled setup it is a proxy for physical size error. Mask IoU is a separate measure: it checks whether the model follows the conditioning, not the item's physical size.

## 5. Fair comparison
All methods get the same inputs:
- room image
- reference image
- target dimensions
- placement
- projected mask, wherever the method supports a mask

The methods are G0 cut-and-paste at scale, Paint-by-Example, AnyDoor and E1–E5. Unsupported conditioning is documented.

The improvement over baselines is computed as:

(E_best baseline − E_ours) / E_best baseline

## 6. Risks
| Risk | Detection | Response |
|---|---|---|
| MoGe-2 scale inaccurate | room-length error | known-length calibration |
| Floor estimation fails | plane residual | exclude scene or correct manually |
| Diffusion ignores the mask | mask compliance | box-edge conditioning (ControlNet) |
| LoRA does not improve size | validation compliance / E | keep the baseline study as the result |
| SDXL exceeds GPU | memory use | lower resolution / batch, or SD 1.5 |
| Data yield too low | valid-pair count | fewer categories or more rooms |
| Identity drift | DINO + qualitative examples | report failures |
| Background altered | PSNR outside the mask | paste back unmasked pixels |

## 7. Priority order
1. End-to-end geometric insertion.
2. Off-the-shelf study.
3. LoRA.
4. Real-world test.
5. Dimension estimation from photos, only if time allows.

**Minimum outcome:** a pipeline that takes a room photo, a reference image, dimensions and a placement spot, and produces a geometry-conditioned insertion. It is evaluated against at least two reference-guided baselines. LoRA improvement is a research objective, not a prerequisite.
