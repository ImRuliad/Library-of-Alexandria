---
Authors: Jacob Austin, Sholto Douglas, Roy Frostig, Anselm Levskaya, Charlie Chen, Sharad Vikram, Federico Lebron, Peter Choy, Vinay Ramasesh, Albert Webson, Reiner Pope
Affiliation: Google Deepmind
Published: February 4, 2025
---
---
## Table Of Contents
















## Where Does The Time Go?
---
*Why does an algorithm take 50ms instead of 50s or 5s?*

**Computation**
A deep learning model is a bunch of matrix multiplications, each composed of floating-point multiplication and addition operations (FLOPs).
Accelerator speed determines how long these take to compute.

$$
T_{math} = \frac{computation\;FLOPs}{Acceleration\;FLOPs/s}
$$

**Communication Within A Chip**
Within an Acceleration, tensors need to be transferred between accelerator memory (HBM) and compute cores.
- HBM Bandwidth on an H100 is about [3.35 TB/s](https://www.nvidia.com/en-us/data-center/h100/)
- HBM Bandwidth on a TPU v6e is about [1.6 TB/s](https://cloud.google.com/tpu/docs/v6e)

**Communication Between Chips**
When we distribute a model across multiple accelerators, tensors frequently need to be transferred between them.
There a few options for this on hardware:
- ICI
- DCN
- PCIe

$$
T_{Comms} = \frac{Communication\;Bytes}{Network/Memory\;Bandwidth\;Bytes/s}
$$
