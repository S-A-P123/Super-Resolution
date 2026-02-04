# **Satellite Image Super-Resolution: Technical Approach and Architecture**

This document provides a comprehensive overview of the methodologies, architectural choices, and technical innovations I implemented in the Advanced EDSR (Enhanced Deep Residual Networks) with Channel Attention model for $4\times$ satellite imagery super-resolution.

## 1. Core Architecture

I built the model upon a modified version of the EDSR architecture.

**Key Components:**

**Residual Channel Attention Blocks (RCAB):** Unlike standard residual blocks, I utilized RCABs which include a Channel Attention (CA) mechanism. In satellite imagery, different feature channels may represent specific sensor signatures or spectral details. CA allows the network to dynamically rescale channel-wise features, focusing the "attention" on more informative components (like building edges) while suppressing noise in flat areas (like fields or water).

**Depth and Capacity:** The network utilizes 32 blocks and 128 features. To ensure stability at this depth, I applied Residual Scaling (RES_SCALE = 0.1) to each block, preventing gradient explosion or vanishing during training.

**Global Bicubic Residual Connection:** A fundamental part of my approach is that the model does not attempt to learn the high-resolution image from scratch. Instead, it internally calculates a Bicubic baseline and focuses solely on learning the High-Frequency Residual—the difference between the math-based upscale and the truth.

## 2. Overcoming Artifacts

Standard Super-Resolution models often suffer from Checkerboard or Grid-like artifacts.

**The Problem:**

These artifacts are typically caused by PixelShuffle (Sub-pixel convolution) layers. When the convolution filters are not perfectly synchronized, the "reshuffling" of channels into pixels creates a repeating mathematical pattern that appears as a grid.

**My Solution:**

I replaced PixelShuffle with a Nearest-Neighbor Interpolation + Convolution strategy.

**Nearest-Neighbor Upscaling:** Scales the feature map spatial size without introducing new pixel values.

**Refinement Convolution**: A subsequent $3 \times 3$ convolution layer smooths the transition between pixels.
This ensures that the spatial expansion is "feature-aware" rather than purely mathematical, resulting in a smooth, grid-free output.

## 3. Optimization & Loss Functions

To achieve exact replication of ground truth, I utilize a multi-component loss function:

**Charbonnier Loss (Pixel Loss):** A robust variant of the L1 loss. I chose this because it handles outliers and extreme pixel values better than Mean Squared Error (MSE), preventing the model from becoming "blurry" in high-contrast satellite scenes.

**Perceptual Loss (VGG19):** I pass the predicted and ground-truth images through a pre-trained VGG19 network. By comparing features at Layer 18, the model is incentivized to replicate the structural "feeling" and edges of the scene, not just the raw pixel values.

**Total Variation (TV) Loss:** This acts as a regularizer to encourage piecewise smoothness. It helps eliminate random noise while keeping the boundaries of buildings and roads sharp.

## 4. Evaluation Metrics

Used two primary metrics to quantify success, measured against the High-Resolution Ground Truth:

-PSNR (Peak Signal-to-Noise Ratio)

-SSIM (Structural Similarity Index)

## 5. Technical Novelties

The novelty of this implementation lies in three specific areas:

**A. Differentiable Sharpening in-the-Loop**

Conventionally, sharpening is a post-processing step. In this approach, I integrated a Differentiable Unsharp Mask (USM) filter into the training loop. By training through the sharpening filter, the model learns to synthesize high-frequency textures that are optimized to look clear and sharp after final enhancement.

**B. Global to Local Residual Learning**

By nesting a global skip connection (Bicubic) with local skip connections (Residual Blocks), the model essentially acts as a Correction Engine. I designed it to respect the global geometry of the original low-res image while refining local textures.

**C. Satellite Specific Feature Calibration**

The use of Channel Attention without Batch Normalization is a critical choice. Removing Batch Norm (as popularized by EDSR) preserves the absolute intensity range of satellite sensors, while CA prioritizes the structural features unique to aerial perspectives.
#
**Summary:** I could achieve a PSNR score of 35db+ and SSIM score of around 0.9 with this approach. 
