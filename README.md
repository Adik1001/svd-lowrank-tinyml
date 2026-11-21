# Low-Rank SVD Compression for TinyML Edge Devices

**Paper:** [Theoretical Foundations of Matrix Low-Rank Approximation and Operator Norm Error Bounds on Binary Digit Spaces](Salamat_SVD_Low_Rank_Approximation_TinyML.pdf) (PDF, 17 pages)
**Author:** Adilzhan Salamat · Pioneer Research Institute, 2026

Microcontrollers used for TinyML have very little SRAM, so images and weight matrices have to be compressed before they can be stored and processed on the device. This project studies how far a binary image can be compressed with truncated Singular Value Decomposition (SVD) before its structure breaks, and how to know the worst-case error **without reconstructing the image**.

## Main idea

A rank-*k* SVD approximation stores `k(m + n + 1)` numbers instead of `m × n`. By the Eckart–Young–Mirsky theorem, the truncation is the best possible rank-*k* approximation, and its **operator-norm (spectral) error is exactly the first discarded singular value, σ<sub>k+1</sub>**. A device can therefore read the worst-case distortion straight from the precomputed singular values.

The paper argues for the operator norm instead of the Frobenius norm: the Frobenius norm averages error over the whole image and can hide a single erased stroke, while the operator norm measures the worst direction.

## Method

1. Build 100 × 100 binary matrices for the handwritten digits "0" and "1".
2. Decompose each with full-rank SVD (`numpy.linalg.svd`).
3. Sweep the rank *k* from 100 down to 1, reconstruct the approximation each time, and measure the operator-norm error (`ord=2`) and the memory footprint.
4. Compare the measured error with the theoretical bound σ<sub>k+1</sub>.

## Results

**The theoretical bound matches the measured error to machine precision (Δ ≈ 10⁻¹⁶):**

| Digit | Rank *k* | Measured operator-norm error | σ<sub>k+1</sub> |
|---|---|---|---|
| "0" | 1 | 23.0926 | 23.0926 |
| "0" | 2 | 16.8579 | 16.8579 |
| "0" | 3 | 10.0919 | 10.0919 |
| "0" | 5 | 5.1820 | 5.1820 |
| "0" | 10 | 2.4170 | 2.4170 |
| "0" | 15 | 1.7162 | 1.7162 |
| "1" | 1 | 0.0000 | 0.0000 |

**Memory saved on a 100 × 100 grid (10,000 values uncompressed):**

| Rank *k* | Values stored | Memory saved |
|---|---|---|
| 15 | 3,015 | 69.8% |
| 10 | 2,010 | 79.9% |
| 5 | 1,005 | 90.0% |
| 3 | 603 | 94.0% |
| 2 | 402 | 96.0% |
| 1 | 201 | 98.0% |

- **Digit "1"** is a straight vertical line, so it is exactly a rank-1 matrix: 98% smaller with zero error.
- **Digit "0"** has curved strokes, so its singular values decay slowly. **Rank 5 is the knee point**: 90% memory saved with operator-norm error 5.18. At rank 2 or below the loop breaks apart into blocks and the image is no longer recognizable.

## Files

- `Salamat_SVD_Low_Rank_Approximation_TinyML.pdf`: the full paper, with hand-worked SVD examples, the error-bound proofs, figures and references.
