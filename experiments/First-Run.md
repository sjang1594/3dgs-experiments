# 3DGS First Run

### Objective
Run the official 3DGS implementation end-to-end on a custom dataset (self-captured indoor scene) for the first time.
- Experience the **full pipeline**: COLMAP SfM → 3DGS training → point cloud output → viewer visualization
- Verify that training converges and produces visually reasonable results on a custom scene
- Obtain baseline quantitative metrics (PSNR / SSIM / LPIPS) for future comparison

### Meta
- Date: 2025-01-26
- Repo: [graphdeco-inria/gaussian-splatting](https://github.com/graphdeco-inria/gaussian-splatting)
- Dataset: `my_scene/playroom` (self-captured, 225 images)
- GPU: NVIDIA GeForce RTX 2070 Super (8GB)
- Viewer: https://antimatter15.com/splat/

### Config
```
Namespace(
  data_device='cuda',
  eval=False,            # ← BUG: no train/test split → metrics are NaN
  images='images',
  resolution=-1,         # original resolution
  sh_degree=3,           # Spherical Harmonics degree
  source_path='C:\\Users\\skcjf\\project\\gaussian-splatting\\data\\my_scene\\playroom',
  model_path='./output/858ba1ea-e',
  train_test_exp=False,
  white_background=False
)
```

### Results
- Training time: ~2 days (RTX 2070 Super)
- Iterations: 30,000 (checkpoints at 7,000 and 30,000)
- Output: `gaussian-splatting\output\858ba1ea-e\point_cloud\iteration_30000`

| Metric | Value |
|--------|-------|
| PSNR | NaN |
| SSIM | NaN |
| LPIPS | NaN |

> **Why NaN**: Trained with `eval=False`, so no train/test split occurred. All 225 images were used for training, leaving the test set empty. Metrics cannot be computed without held-out test views.

**Visual result:**
![[../img/image.png]]

**Iteration 7,000 vs 30,000 comparison:**
**Iteration 7,000 vs 30,000 comparison:**
| Iteration 7,000 | Iteration 30,000 |
|---|---|
| ![[../img/image-1.png|472x235]] | ![[../img/image-2.png|475x235]] |
- Front-facing view (looking into the room): 7k and 30k are nearly identical
- Looking up at the ceiling: visible holes/gaps → likely insufficient training views from upward angles, or densification did not cover that region adequately

### Problems Encountered
- Output folder names are random hashes (e.g. `858ba1ea-e`), so locating the actual results was initially confusing
- Did not know which file to load into the viewer — eventually found that `point_cloud/iteration_30000/point_cloud.ply` is the correct file
- No build errors; used a separate Python virtual environment
- Training took ~2 days on RTX 2070 Super, which saturated the GPU entirely (couldn't even run YouTube simultaneously)
- 4 total attempts, 3 failed/aborted before the successful run

### Output Structure
```
858ba1ea-e/           ← completed training (only successful run)
├── results.json      ← PSNR/SSIM/LPIPS (NaN — eval=False)
├── per_view.json     ← per-view metrics (empty)
├── cfg_args
├── cameras.json
├── point_cloud/
│   ├── iteration_7000/point_cloud.ply
│   └── iteration_30000/point_cloud.ply
├── train/ours_30000/ (gt: 225 images, renders: 225 images)
└── test/ours_30000/  (gt: 0, renders: 0) ← empty because eval=False

152def8b-4/  ← training interrupted (no point_cloud)
7369d8a7-b/  ← training never started
ddee61bd-8/  ← training never started
```
4 attempts total, 1 completed.

### Next Steps → [[3DGS Second Run]]
1. **Re-train with `--eval` flag** — enable train/test split to get actual PSNR/SSIM/LPIPS numbers
2. Investigate ceiling holes — adjust `densify_grad_threshold` or capture more upward-facing views
3. Test training time reduction — lower `resolution` parameter and compare quality vs speed
4. Once real metrics are available, compare against Mip-NeRF 360 indoor scene baselines