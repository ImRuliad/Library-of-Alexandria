## Table Of Contents
- [[#***The Basics of Transformer Inference***|The Basics of Transformer Inference]]
- [[#***The KV Cache***|The KV Cache]]
- [[#***What Do We Actually Want To Optimize?***|What Do We Actually Want To Optimize?]]






## ***The Basics of Transformer Inference***
---
*We have a trained transformer and we want to generate new sequences.*

Sampling is conceptually simple
- We put a sequence in
- Transformer spits a sequence out

This sequence is defined as
$$
logp(next\;token_i\;|\;previous\;tokens)
$$
In other words, this is a log-probabilities for all possible next tokens....
- We can sample from this distribution of all next possible tokens to obtain a new token.
- We can append this token and then repeat the process.
- We thus obtain a sequence of tokens which is the continuation of the prompt.

![Naive Sampling from a Transformer](https://jax-ml.github.io/scaling-book/assets/img/naive-inference.png)
*Figure: naive sampling from a Transformer. The blue logits give us a distribution over the next token that we can sample from. Each step re-processes the entire prefix, leading to an O(n^2) runtime for the algorithm.*

This is called Naive Transformer Sampling.
- It works
- It is never done in practice

Its never done in practice because re-processing the entire sequence every token generation is extremely inefficient.

$$
FFW = O(n^2)
$$
$$
Attention\;Mechanism = O(n^3)
$$
## ***The KV Cache***
---
*How do we avoid the above? How can we avoid re-processing the entire sequence each time we want to generate a new token?*

Instead of doing the full forward pass each time, we can save some intermediate activations from each forward pass that lets us avoid re-processing previous tokens.
- A given token only attends to previous tokens during dot product attention
- We can write each tokens key and value projections into a new data structure
- Once key / value projections are saved for previous tokens, we can compute the dot products without performing any new FLOPs on the earlier tokens.

$$
q_i\;*\;k_j
$$
This is called a KV Cache.

With this in mind, inference has to key parts:
- **Prefill**
- **Generation**

**Prefill**
Given a long prompt, process all tokens in the prompt at the same time and save the resulting activations (specifically the key-value projections) in a KV Cache.
We also save the logits for the last token.

**Generation**
Given a KV Cache and previous logits, incrementally sample on token from the logits, feed that token back into the transformer, and produce a new set of logits for the next step.
We also append the KV activations for that new token to the KV cache. We repeat this until we hit a special EOS token or reach some maximum length limit.


![[Pasted image 20260918083003.png]]

*figure: a diagram of efficient Transformer sampling with a KV cache. Prefill processes our prompt and saves all the per-token key-value activations in a cache. Generation takes this cache (and the last-token logits), samples a new token, and passes that new token through the model, attending to the KV cache and saving the new tokens key-value projections back to the cache. This is an O(n) algorithm in the MLP block.*








## ***What Do We Actually Want To Optimize?***
---
