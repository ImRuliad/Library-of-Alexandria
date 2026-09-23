---
Authors:
  - Jacob Austin, Sholto Douglas, Roy Frostig, Anselm Levskaya, Charlie Chen, Sharad Vikram, Federico Lebron, Peter Choy, Vinay Ramasesh, Albert Webson, Reiner Pope
Affiliation: Google Deepmind
Published: February 4, 2025
Source: https://jax-ml.github.io/scaling-book/tpus/
---
---
## Table Of Contents









## What Is a TPU?
---
*A TPU is basically a compute core that specializes in matrix multiplication (called a TensorCore) attached to a stack of High-Bandwidth Memory*

![](https://jax-ml.github.io/scaling-book/assets/img/tpu-chip.png)**Figure**: *Abstract layout of a TPU TensorCore (Gray Box).

#### ***Three Components Of a TPU***

**MXU (Matrix Multiply Unit)**
- The core of the TensorCore
- Performs one of $bf16[8,128]\;@\;bf16[128,128]\;->\;f32[8,128]$ matrix multiply every 8 cycles using a systolic array
- This is about $5^{13}$ bf16 FLOPs/s per MXU at 1.5GHz on TPU v5e. Most TensorCores have 2 or 4 MXUs so total bf16 FLOPs/s is $2^{14}$
- TPUs also support lower precision matmuls with higher throughput (e.g. each TPU v5e chip can do $4^{14}$ int8 OPs/s)

 **VPU (Vector Processing Unit)**
 - Performs general math operations like ReLU activations or pointwise addition or multiplication between vectors
 - Reductions (sums) are also performed here.

**VMEM (Vector Memory)**
- On-chip scratchpad located in the TensorCore, close to the compute units
- Much smaller than HBM (128 MiB on TPU v5e)
- Much higher bandwidth than the MXU
- Operates like an L1/L2 cache but is much larger and programmer controlled.
- Data in HBM needs to be copied into VMEM before TensorCore can do any computation with it.

#### ***TPUs Are Very Very Fast at Matrix Multiplication***
- [TPU v5p]([TPU v5p  |  Google Cloud Documentation](https://docs.cloud.google.com/tpu/docs/v5p#system_architecture)) can do $2.5^{14}$ bf16 FLOPs/s per core 
- Can do $5^{14}$ bf16 FLOPs/s per chip
- A single pod of 8960 chips can do 4 bf16 exaFLOPs/s

**High Bandwidth Memory**
- Stores tensors for use by the TensorCore
- Usually has tens of gigabytes ([TPU v5p]([TPU v5p  |  Google Cloud Documentation](https://docs.cloud.google.com/tpu/docs/v5p#system_architecture)) has 16GiB of HBM)
- When needed for computation, tensors are streamed out of HBM through VMEM into the MXU
- Result is written from VMEM back to HBM
- Bandwidth between HBM and TensorCore (Through VMEM) is known as HBM bandwidth (Usually between 1-2 TB/s) and limits how fast computation can be done in memory-bound workloads

**TPU Operation Pipelining and Overlapping**
*Usually TPU operations are pipelined and overlapped*

Assume the computation $X * A -> Y$
- TPU copies chunks of matrices $A$ and $X$ from HBM into VMEM
- Chunks are then loaded into MXU
- MXU multiplies chunks of 8x128 (for $X$)
- MXU multiplies chunks of 128x128 (for $A$)
- Result is copied chunk by chunk back to HBM
- Matmul is pipelined so copied to/from VMEM are overlapped with MXU work
- Allows the MXU to continue working instead of waiting on memory transfers
- Keeps matmuls compute-bound, not memory-bound

![800x800](https://jax-ml.github.io/scaling-book/assets/img/pointwise-product.gif)

If the load from HBM to VMEM is slower than the FLOPs in the Vector Unit (or MXU) then we become Bandwidth-bound since we are starving the VPU or MXU of work.


> [!Important] Key Takeaway
> TPUs load weights from HBM into VMEM, then from VMEM into a systolic array
> - Systolic Array performs ~200 trillion multiple-adds per second.
> - The HBM ↔↔ VMEM and VMEM ↔↔ systolic array bandwidths set fundamental limits on what computations TPUs can do efficiently.


**VMEM and Arithmetic Intensity**
- Much smaller than HBM
- Much higher bandwidth than the MXU
- If an algo can fit all its I/Os in VMEM, its must less likely to hit communication bottlenecks
- This is helpful when a computation has poor arithmetic intensity
- VMEM bandwidth ~ 22x higher than HBM Bandwidth, which means an MXU op (read/write to/from VMEM) only requires an arithmetic intensity of 10-20 to achieve peak FLOPs utilization
- If we can fit our weights into VMEM instead of HBM, mat muls can be FLOPs bound at much smaller batch sizes

![](https://jax-ml.github.io/scaling-book/assets/img/tpu-bandwidth.png)

**A TPU Chip (Typically) consists of Two TPU cores which share memory and can be thought of as one large accelerator**
- v4, v5, v6 TPUs have two TPU cores and twice the FLOPs (known as Megacore)
- TPU v7 removes the megacore and instead has a high-bandwidth link between the two cores
- Older TPU chips (v3 and older) have separate memory and are considered two separate accelerators
- Inference optimized chips (TPU v5e) only have one TPU core per chip

![](https://jax-ml.github.io/scaling-book/assets/img/cores.png)

**Chips are arranged in sets of 4 in a "tray"**
- Connected to a CPU host via PCIe
- 4 Chips (8 Cores, usually treated as 4 Megacores)
- Exposed through Colab or a single TPU-VM

![](https://jax-ml.github.io/scaling-book/assets/img/pcie.png)

**PCIe bandwidth is limited**
- Like the HBM ↔↔ VMEM link, the CPU ↔↔ HBM PCIe connection has a specific bandwidth that limits how quickly you can load from host memory to HBM or vice-versa.
- PCIe bandwidth for TPU v4 is 16GiB/s each way (close to 100x slower than HBM)
- So we *can* offload data into the host (CPU) RAM, but it is very slow

## ***TPU Networking***
---
*Chips are connected to each other through the ICI network in a Pod*

- In older generation TPUs, ICI connects the 4 nearest neighbors (with edge links) to form a 2D torus.
- TPU v4 and TPU v5p are connected to nearest 6 neighbors to form a 3D torus
- These connections do not go through hosts, they are direct links between chips
- Toroidal structure reduces the max distance between any two nodes from $N$ to $N/2$
- TPUs also have a twisted torus configuration that wraps the torus in a mobius strip like topology to further reduce the average distance between nodes

![800x800](https://jax-ml.github.io/scaling-book/assets/img/ici-wraparound.png)

**TPU Pods Connected via ICI**
- The maximum pod size (called a superpod) is 16x16x16 for TPU v4
- For TPU v5p it is 16x20x28
- Large pods are composed of reconfigurable cubes of 4x4x4 chips connected by [Optical Wraparound Links](https://arxiv.org/pdf/2208.10041) 
- Optical Wraparound links can be reconfigured to connect very large topologies

![1200x640](https://jax-ml.github.io/scaling-book/assets/img/tpu-rack.png)

- Smaller topologies (2x2x1 and 2x2x2) can also be requested, but with no wraparound link
- No wraparound link typically doubles the time of most communication
- Any multiples of full cubes (4x4x4 or 4x4x8) will have wrap arounds.

![1200x640](https://jax-ml.github.io/scaling-book/assets/img/subslices.png)










## ***TPU Specs***
---
**General Figures**

| Model   | Pod Size | Host Size | HBM Capacity/chip | HBM BW/chip<br>(bytes/s) | FLOPs/s/chip<br>(bf16) | FLOPs/s/chip<br>(int8) |
| ------- | -------- | --------- | ----------------- | ------------------------ | ---------------------- | ---------------------- |
| TPU v3  | 32x32    | 4x2       | 32GB              | $9.0^{11}$               | $1.4^{14}$             | $1.4^{14}$             |
| TPU v4p | 16x16x16 | 2x2x1     | 32GB              | $1.2^{12}$               | $2.75^{14}$            | $2.75^{14}$            |
| TPU v5p | 16x20x28 | 2x2x1     | 96GB              | $2.8^{12}$               | $4.59^{14}$            | $9.18^{14}$            |
| TPU v5e | 16x16    | 4x2       | 16GB              | $8.2^{11}$               | $1.97^{14}$            | $3.94^{14}$            |
| TPU v6e | 16x16    | 4x2       | 32GB              | $1.6^{12}$               | $9.2^{14}$             | $1.84^{15}$            |
| TPU7x   | 4x4x576  | 2x2x1     | 192GB             | $7.4^{12}$               | $2.3^{15}$             | $4.61^{15}$            |

*Host size refers to the topology of TPUs connected to a single host.*
*e.g. TPU v5e has a single CPU host connected to 8 TPUs in a 4x2 topology.*

**Interconnect Figures**

| Model   | ICI BW/link<br>(one-way, byte/s) | ICI BW/link<br>(bidirectional, byte/s) |
| ------- | -------------------------------- | -------------------------------------- |
| TPU v3  | $1.0^{11}$                       | $2.0^{11}$                             |
| TPU v4p | $4.5^{10}$                       | $9.0^{10}$                             |
| TPU v5p | $9.0^{10}$                       | $1.8^{11}$                             |
| TPU v5e | $4.5^{10}$                       | $9.0^{10}$                             |
| TPU v6e | $9.0^{10}$                       | $1.8^{11}$                             |
| TPU7x   | $9.0^{10}$                       | $1.8^{11}$                             |

