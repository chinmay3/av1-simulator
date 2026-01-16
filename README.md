# AV1-Inspired Video Codec Simulator

This repository contains an AV1-inspired video codec simulator that implements the core ideas of modern video compression: spatial prediction, temporal prediction, transform coding, quantization, entropy coding, and end-to-end decoding.  
This is not a full AV1 implementation. The goal is correctness, clarity, and visibility into each stage of the pipeline.

## Pipeline

Raw YUV  
→ Encode  
→ AV1S bitstream  
→ Decode  
→ Reconstructed video

## Part 1 — Input & YUV Handling
Reads raw YUV420 video and parses frames using provided width, height, and frame rate.  
Separates luma and chroma planes and prepares frames for block-based processing.

## Part 2 — Adaptive Block Partitioning
Splits each frame into variable-sized blocks.  
Large blocks are used in low-detail regions, and smaller blocks are used near edges and high-detail areas.  
This block structure is reused by all later stages.

## Part 3 — Intra Prediction
Performs spatial prediction using previously decoded neighboring pixels.  
Implemented prediction modes:
- DC
- Vertical
- Horizontal
- Angular

The best mode is selected per block based on prediction error.

## Part 4 — Inter Prediction
Performs temporal prediction using previously decoded frames.
- Motion estimation searches for matching blocks
- Motion vectors describe block displacement
- Motion-compensated prediction is generated

This stage exploits temporal redundancy between frames.

## Part 5 — Residual Computation
Computes the residual:  
`residual = original − prediction`  
Most residual values are near zero when prediction is effective.

## Part 6 — Transform & Quantization
Applies a block-wise transform to residuals, followed by quantization.  
This stage:
- Concentrates energy into low frequencies
- Discards visually less important detail
- Introduces controlled loss

## Part 7 — Entropy Coding
Serializes and entropy-codes transformed coefficients and metadata into a compact `.av1s` bitstream.  
This is where actual on-disk compression occurs.

## Part 8 — Decoder: Bitstream Parsing
Parses the `.av1s` bitstream and reconstructs encoder decisions:
- Block structure
- Prediction modes
- Motion vectors
- Quantized coefficients

No searching is performed during decoding.

## Part 9 — Inverse Transform & Reconstruction
Performs:
- Inverse quantization
- Inverse transform
- Residual addition to predictions

Frames are reconstructed sequentially to produce decoded output.

## Part 10 — Output & Evaluation
Outputs:
- Reconstructed YUV
- Optional MP4 preview

Metrics:
- Compression ratio
- Bitrate
- PSNR / SSIM (luma)

## Results
Metric | Value
---|---
Input format | Raw YUV420
Resolution | 352×288
Frame rate | 30 fps
Input size | 45.6 MB
Compressed size | 3.75 MB
Compression ratio | 12.7×

## Limitations
- Not AV1 spec-compliant
- Simplified prediction and rate control
- No hardware acceleration
- Intended for learning and visualization

## Usage

Encode:
```bash
python3 -m scripts.encode input.yuv output.av1s \
  --yuv-width 352 --yuv-height 288 --yuv-fps 30
```

Decode:
```bash
python3 -m scripts.decode output.av1s recon.yuv
```

## Repository Structure

```
scripts/
  encode.py
  decode.py
  intra_predict.py
  inter_predict.py
  transform.py
  entropy.py
```

## Notes
This project focuses on understanding how modern codecs work, not on matching production AV1 performance.
