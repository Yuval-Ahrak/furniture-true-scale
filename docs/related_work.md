# Related work — true-scale furniture insertion

Literature review for the proposal (November 2026). Grouped by approach; each entry ends with what it means for this project.

## 0. Reference-based object insertion (the generation step)
- **Paint by Example** — Yang et al., 2022, [arXiv:2211.13227](https://arxiv.org/abs/2211.13227). Exemplar-guided inpainting trained self-supervised; inserts an object from a reference image into a masked region.
- **AnyDoor** — Chen et al., CVPR 2024, [arXiv:2307.09481](https://arxiv.org/abs/2307.09481). Regenerates a box-marked region of a scene using the target object as template; trained on image pairs of the same object from videos.
- **IP-Adapter** — Ye et al., 2023, [arXiv:2308.06721](https://arxiv.org/abs/2308.06721). Lightweight image-prompt adapter (decoupled cross-attention) for frozen text-to-image diffusion models; works with inpainting.

→ *For us:* baselines G1–G3 and the base model. None of them gives the box a metric meaning — the box size is the user's guess.

## 1. Metric monocular depth / geometry
- **Depth Pro** — Bochkovskii et al., ICLR 2025, [arXiv:2410.02073](https://arxiv.org/abs/2410.02073). Zero-shot metric depth with absolute scale and no camera intrinsics needed; estimates focal length from the image; 2.25 MP depth map in ~0.3 s.
- **MoGe-2** — Wang et al., NeurIPS 2025, [arXiv:2507.02546](https://arxiv.org/abs/2507.02546). Metric-scale 3D point map (plus normals and FOV) from one image; extends affine-invariant MoGe to metric scale.
- **Honey, I Shrunk the Arc de Triomphe!** — Xiangli et al., 2026, [arXiv:2606.02379](https://arxiv.org/abs/2606.02379). Shows "scale collapse": metric geometry models systematically shrink far / wide structures; blames training data (LiDAR, short-range indoor, synthetic) and fine-tunes MoGe-2 on in-the-wild metric data.

→ *For us:* the geometry baseline (B1) and a feature source for the size network. Useless on white-background catalog shots (no scene), and possibly biased on wide shots.

## 2. Monocular 3D object detection
- **Omni3D / Cube R-CNN** — Brazil et al., CVPR 2023, [arXiv:2207.10660](https://arxiv.org/abs/2207.10660). 234K images, 98 categories; Cube R-CNN predicts 3D location, **size** and rotation from one image across domains, using a "virtual depth" to handle varying intrinsics.

→ *For us:* baseline B3 for furniture classes it knows. Trained on scan datasets (SUN RGB-D, ARKitScenes, Hypersim, Objectron…), not on listing photos.

## 3. VLM quantitative spatial reasoning
- **SpatialVLM** — Chen et al., CVPR 2024, [arXiv:2401.12168](https://arxiv.org/abs/2401.12168). Synthesizes internet-scale metric spatial VQA (2B QA pairs from 10M images) and co-trains a VLM; enables metric distance/size answers.
- **Q-Spatial Bench + SpatialPrompt** — Liao et al., EMNLP 2024, [arXiv:2409.09788](https://arxiv.org/abs/2409.09788). 271 human-annotated quantitative questions; prompting VLMs to reason via reference objects raises success rates by 20–40 points without fine-tuning.
- **VSI-Bench (Thinking in Space)** — Yang et al., CVPR 2025, [arXiv:2412.14171](https://arxiv.org/abs/2412.14171). Video benchmark incl. object-size estimation scored by mean relative accuracy (MRA).
- **Ill-Posed by Design (Metric VQA)** — Meivar, Perek, Shvartzman, Schwartz, Avidan, 2026, [arXiv:2606.24335](https://arxiv.org/abs/2606.24335). Most directly relevant. Uses monocular metric object-size estimation as a diagnostic: 10,813 queries (Objectron) + 331 tape-measured in-the-wild scenes, 12 open VLMs. Finds that **object identity (category prior) dominates**, scene geometry is largely unused, the largest VLMs still trail a text-only LLM in the wild, and LoRA fine-tuning makes the task learnable but does not make models use scene geometry.

→ *For us:* baseline B2, and the motivation for the evidence-sensitivity evaluation (within-category correlation, scale counterfactuals). Their in-the-wild split shows tape-measured test sets are an accepted protocol.

## 4. Furniture insertion into room images
- **RoomEditor++** — Wang et al., 2025, [arXiv:2512.17573](https://arxiv.org/abs/2512.17573). Parameter-sharing dual diffusion backbone (U-Net or DiT) for furniture insertion; RoomBench++ dataset (112,851 train / 1,832 test pairs). Does not address real-world dimensions.
- **GOPI** — Zhai et al., 2026, [arXiv:2608.06836](https://arxiv.org/abs/2608.06836). Single-view RGB-D furniture insertion; states that with 2D mask conditioning the physical scale of inserted furniture "cannot be uniquely determined", so infers a plausible 3D pose first, then conditions generation on its projection.

→ *For us:* the demo stage. Our predicted metric box can supply exactly the missing scale; GOPI's projected-geometry conditioning is the template for the demo.

## 5. Data
- **ARKitScenes** — Baruch et al., NeurIPS 2021 Datasets & Benchmarks, [arXiv:2111.08897](https://arxiv.org/abs/2111.08897). 5,048 captures of 1,661 scenes; manually labeled oriented 3D bounding boxes for 17 room-defining furniture categories. → *For us:* scale-exact self-supervised training pairs.
- **ABO** — Collins et al., CVPR 2022, [arXiv:2110.06199](https://arxiv.org/abs/2110.06199). 147,702 Amazon listings, 398,212 catalog images, 7,953 artist-made PBR meshes, up to 18 metadata attributes incl. dimensions; CC BY-NC 4.0.
- **D-Extract** — Amazon Science. Extracts dimension attributes from product images via OCR — shows retailers value dimensions, but only works when numbers are printed on the image.

→ *For us:* ABO gives per-category size priors for the size estimate.

## Gap
Insertion models make the furniture look right, but nobody checks that it is the right size: reference-based inpainting has no metric notion of the box, RoomEditor++ ignores real dimensions, and GOPI needs RGB-D and still calls scale underdetermined. This project inserts furniture at metric scale from two RGB photos and measures scale fidelity in the generated images.
