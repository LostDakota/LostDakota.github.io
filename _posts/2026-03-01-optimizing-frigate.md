---
layout: post
title:  "Optimizing Frigate NVR on Consumer GPUs: A Resourceful Engineer's Guide (RTX 3050 Edition)"
date:   2026-03-01 12:00:00 -0500
categories: automation frigate
---

## The Problem: The "Coral" Tax

In the current era of "AI Everywhere," hardware has become the new bottleneck. Scalpers snatch up Google Coral TPUs the moment they restock, driving prices from $60 to $150+. Specialized NVR appliances demand a premium, and high-end CPUs capable of running multiple inference streams cost a small fortune in both silicon and electricity.

For the home lab enthusiast, this creates a frustrating barrier. We want to experiment with advanced object detection, but the "entry fee" feels artificially high.

However, many of us have older or mid-range hardware sitting idle. In our case, a localized server running an NVIDIA RTX 3050. It's not a data center A100, but it is a capable piece of silicon often overlooked for NVR tasks.

**Why are we doing this?**
1.  **Frugality:** Why spend $150 on a USB accelerator when you already own a GPU?
2.  **Sustainability:** Maximizing the lifespan of existing hardware reduces e-waste.
3.  **Creativity:** Constraints breed innovation. Learning to optimize configs teaches us more than simply swiping a credit card.

## The Resourceful Solution: TensorRT

Frigate 0.13+ introduced support for **TensorRT**, NVIDIA's high-performance deep learning inference optimizer. This was a game-changer. It allows us to offload the heavy lifting of object detection from the CPU to the GPU's dedicated Tensor cores.

The result? Your consumer-grade RTX 3050 (or even a GTX 10-series) transforms into an inference beast, easily handling 8+ camera streams with sub-10ms inference speeds.

## The Config (The "Copy-Paste" Value)

Here is how we configured our environment to maximize the RTX 3050.

### `compose.yml`

Passing the GPU to Docker is the first step. We allocate `gpu`, `compute`, and `video` capabilities to the container.

```yaml
services:
  frigate:
    image: ghcr.io/blakeblackshear/frigate:stable-tensorrt
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 'all'
              capabilities: [gpu, compute, video]
    environment:
      # Map the TensorRT model file
      - YOLO_MODELS=yolov8m:/config/model_cache/yolov8m.onnx
```

### `config.yml`

In the Frigate config, we explicitly define the detector type as `tensorrt` and point it to the specific device index.

```yaml
detectors:
  nvidia_gpu:
    type: tensorrt
    device: 0 # Maps to /dev/nvidia0

model:
  path: /config/model_cache/yolov8m.onnx # The secret sauce
  width: 640
  height: 640
  input_tensor: nchw
  input_pixel_format: rgb
```

## The Result

![Frigate Dashboard showing low CPU usage and TensorRT detection](/assets/frigate-stats.png)
*Fig 1. Frigate Dashboard showing low CPU usage and TensorRT detection (sub-10ms inference).*

**Stats:**
-   **CPU Usage:** Dropped from ~60% to ~5%.
-   **Inference Speed:** Consistent <10ms response times.
-   **Stability:** Rock solid for months.

**Conclusion:**
Use what you have. Don't buy new gear until you've squeezed every drop of performance out of the old stuff. This optimization effort demonstrates how open-source software combined with clever configuration can maximize the utility of consumer hardware, proving you don't need enterprise budgets to build enterprise-grade solutions.
