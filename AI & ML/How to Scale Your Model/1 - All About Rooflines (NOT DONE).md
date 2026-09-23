---
Authors: Jacob Austin, Sholto Douglas, Roy Frostig, Anselm Levskaya, Charlie Chen, Sharad Vikram, Federico Lebron, Peter Choy, Vinay Ramasesh, Albert Webson, Reiner Pope
Affiliation: Google Deepmind
Published: February 4, 2025
Source: https://jax-ml.github.io/scaling-book/tpus/
---
---
## Table Of Contents
[[#***Where Does The Time Go?***|Where Does The Time Go?]]
- [[#### **Computation** | Computation]]
- [[#### **Communication Within A Chip** | Communication Within A Chip]]
- [[#### **Communication Between Chips** | Communication Between Chips]]
- [[#### **Compute Bound** | Compute Bound]]
- [[#### **Communication Bound** | Communication Bound]]
- [[#### **Arithmetic Intensity** | Arithmetic Intensity]]
[[#***Visualizing Rooflines*** | Visualizing Rooflines]]

---


## ***Where Does The Time Go?***
---
*Why does an algorithm take 50ms instead of 50s or 5s?*

#### **Computation**
A deep learning model is a bunch of matrix multiplications, each composed of floating-point multiplication and addition operations (FLOPs).
Accelerator speed determines how long these take to compute.

$$
T_{math} = \frac{computation\;FLOPs}{Acceleration\;FLOPs/s}
$$

#### **Communication Within A Chip**
Within an Acceleration, tensors need to be transferred between accelerator memory (HBM) and compute cores.
- HBM Bandwidth on an H100 is about [3.35 TB/s](https://www.nvidia.com/en-us/data-center/h100/)
- HBM Bandwidth on a TPU v6e is about [1.6 TB/s](https://cloud.google.com/tpu/docs/v6e)

#### **Communication Between Chips**
When we distribute a model across multiple accelerators, tensors frequently need to be transferred between them.
There a few options for this on hardware:
- ICI
- DCN
- PCIe

$$
T_{Comms} = \frac{Communication\;Bytes}{Network/Memory\;Bandwidth\;Bytes/s}
$$

Typically, but not always, computation within a single chip can be overlapped with communication within a chip and between chips.
This means that we can:
- Lower-bound training and inference time by using the maximum of computation and communication time.
- Upper-bound with their sum.

If optimizing against the maximum, then the lower and upper bounds differ by at most a factor of 2.

$$
T_{lower} = max(T_{math}, T_{comms})
$$
$$
T_{upper} = T_{math}+T_{comms}
$$

#### **Compute Bound**
> [!Important] Compute Bound
> Assuming perfect communication and computation overlap, when $$T_{math} > T_{comms}$$ we see full utilization from our hardware. We call this being "Compute Bound".

#### **Communication Bound**

> [!Important] Communication Bound
> When $$T_{comms} > T_{math}$$ we tend to call this "Communication Bound".
> And at least some fraction of our accelerator FLOPs/s are being wasted waiting for data to be passed around.

#### **Arithmetic Intensity**
*One way to tell if an operation will be Communication or Compute Bound is to look at its Arithmetic Intensity*

The arithmetic intensity of an algorithm is given by the ratio of the total FLOPs it performs to the number if bytes it needs to communicate, either within a chip or between chips.
$$
Arithmetic\;Intesity = \frac{Computation\;FLOPs}{Communication\;Bytes}
$$


Arithmetic Intensity measures the FLOPs per byte of a given operation.
- When Arithmetic Intensity is high $T_{math}$ is large compared to $T_{comms}$ and we typically use most of the available FLOPs
- When Arithmetic Intensity is low $T_{comms}$ is large compared to $T_{math}$ and we spend more time on comms and waste FLOPs.
- The point where this crossover happens is called "Peak Arithmetic Intensity".


NOT DONE


## ***Visualizing Rooflines***
---
*We can visualize the tradeoff between memory and compute using a roofline plot, which plots peak achievable FLOPs/s (throughput) of an algorithm on hardware (Y-Axis) against the arithmetic intensity of that algorithm (X-Axis).*

![800x800](https://jax-ml.github.io/scaling-book/assets/img/roofline-improved-1400.webp)

**Figure:** *an example roofline plot showing two algorithms with different arithmetic intensities (Algo 1 and Algo 2) and their corresponding theoretical peak throughput under different bandwidths (BW1 and BW2).*
- *In the red area, an algorithm is bandwidth bound at both bandwidths and is wasting some fraction of the hardware's peak FLOPs/s. 
- *The yellow area is bandwidth-bound only at the lower bandwidth (BW1). 
- *The green area is compute-bound at all bandwidths. 
- *Here, we are using the peak FLOPs/s of the accelerator and increasing bandwidth or improving intensity yield no benefit.*

As arithmetic intensity moves left to right, we initially see a linear increase in the performance of our algorithm (in FLOPs/s) until we git the critical arithmetic intensity of the hardware.
- Any algorithm with a lower intensity will be bandwidth (BW) bound and limited by peak memory bandwidth (In Red).
- Any algorithm to the right will fully utilize our FLOPs (In Green).

In the image above we can see that
- Algo 1 is comms-bound and uses only a fraction of the total hardware FLOPs/s
- Algo 2 is compute-bound.

We can improve the performance of an algorithm by increasing its arithmetic intensity of by increasing the memory bandwidth available to it (Moving from BW1 to BW2).

## ***Matrix Multiplication***
---

Given two matrices $X$ and $Y$ we can write $X * Y -> Z$ 
- $X$ has the shape $bf16[B,D]$
- $Y$ has the shape $bf16[D,F]$
- $Z$ has the shape $bf16[B,F]$

To do the matmul we need to load $DF+2BD$ bytes, perform $2BDF$ FLOPs, and write $2BF$ bytes back, thus:

$$
Intensity(matmul) = \frac{2BDF}{2BD+2DF+2BF} = \frac{BDF}{BD+DF+BF}
$$
We can get a nice simplification if we assume our batch size $B$ is small relative to $D$ and $F$ 
$$
\frac{BDF}{BD+DF+BF} = \frac{BDF}{DF} = B
$$
$$
Intensity(matmul)>Intensity(TPU) => B > \frac{1.97^{14}}{8.20^{11}}=240
$$
This is a reasonable assumption for Transformer matmuls since we...
- Typically have a local (per-replica) token batch size $B < 1024$ (not sequences).
- $D$ and $F$ > $8000$
Thus, we generally become compute-bound when our per-replica batch size is greater than 240 tokens.

