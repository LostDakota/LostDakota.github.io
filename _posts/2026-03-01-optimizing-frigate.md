---
layout: post
title:  "Optimizing Frigate NVR on Consumer GPUs: A Scrappy Engineer's Guide (RTX 3050 Edition)"
date:   2026-03-01 12:00:00 -0500
categories: automation frigate
---

## The Problem: The "Coral" Tax

Everyone says you *need* a Google Coral TPU for Frigate. But they are scalped, expensive, or out of stock.

Meanwhile, we have a perfectly good NVIDIA GPU (RTX 3050) sitting in the server. Why let it idle while the CPU cries?

In the realm of AI network video recorders (NVR), the use of advanced hardware is often a prerequisite for optimal performance and efficiency. Traditional solutions frequently rely on expensive Coral TPUs or colossal CPUs, both of which may not align with budget constraints or existing infrastructure setups.

## The Scrappy Solution: TensorRT

Despite the initial obstacles posed by traditional AI NVR hardware requirements, there exists an alternative approach – leveraging consumer-grade GPUs such as the RTX 3050.

Frigate 0.13+ supports **TensorRT**. It turns your NVIDIA card into an inference beast. You don't need an A100. Even a GTX 10-series or RTX 3050 (like ours) crushes 8+ camera streams.

## The Config (The "Copy-Paste" Value)

Here is how we configured our environment to utilize the RTX 3050.

### `compose.yml`

Passing the GPU to Docker is the first step.

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
      - YOLO_MODELS=yolov8m:/config/model_cache/yolov8m.onnx
```

### `config.yml`

The specific detector block maps the device.

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

**Stats:** My CPU usage dropped from 60% to 5%. Inference speeds hit <10ms.

**Conclusion:** Use what you have. Don't buy new gear until you've squeezed every drop out of the old stuff. This optimization effort demonstrates how open-source software combined with clever configuration can maximize the utility of consumer hardware.
