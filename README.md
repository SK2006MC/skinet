# Relativity Network

Intelligence is the process of using error to refine compression.

[![Status](https://img.shields.io/badge/status-experimental-red)](https://github.com/SK2006MC/skinet)

Relativity Network is an experimental neural architecture that replaces global optimisation (gradient descent / backpropagation) with local state reconciliation. It treats learning not as a search through weight-space driven by a distant loss function, but as a process of local state reconciliation—each neuron constantly comparing what it observes with what it remembers, and updating shared weights to resolve the discrepancy.

In this architecture, activity is the cost of surprise. Learning is the process of turning that activity into memory, eventually leading to a state of Global Silence.

---

## Philosophy

Conventional neural networks are teleological — they are driven by a distant goal (the loss function). The Relativity Network is homeostatic — it is driven by the desire for local consistency.

### Core Axioms

1. **Activity is Energy**
   - Every time a neuron fires, it consumes energy. A highly intelligent system is one that can navigate its environment with the least amount of energy.

2. **Memory is a Baseline**
   - Bias is not an offset; it is the accumulated memory of the expected environment.

3. **Silence is Convergence**
   - When the internal model perfectly matches the external world, the network goes silent.
   - Global Silence = Maximum Compression.

4. **Locality is Absolute**
   - No neuron knows the global objective. Every update is a local reconciliation between two connected neurons.

---

## Architecture

### 1. The Neuron Model (Depletion‑Mode)

Inspired by depletion‑mode transistors, the Relativity neuron is normally ON. It possesses an intrinsic tendency to fire based on its memory $b$. Incoming evidence $x$ does not create activity—it *suppresses* it. When the observed input matches memory, the neuron is silent. When the input is *below* expectation (rare), the neuron fires to signal surprise.

**Forward Pass (Observation):**

$$a = \text{ReLU}(b \cdot (b - x))$$

| Condition | Activity | Physical Meaning |
|-----------|----------|------------------|
| $x < b$ | $a > 0$ | Surprise: Environment is below expectation; neuron fires. |
| $x = b$ | $a = 0$ | Equilibrium: Environment matches memory; neuron is silent. |
| $x > b$ | $a = 0$ | Clamped: Environment exceeds expectation; neuron is suppressed. |

The factor $b$ outside the ReLU scales the surprise signal with the memory itself — a neuron that expects a large input will emit a stronger surprise when that input is missing.

### 2. The Backward Pass (Reconstruction)

Instead of propagating an error gradient, the network performs **State Reconstruction**. It treats a desired output as a boundary condition and reconstructs a compatible internal state layer-by-layer, working backward through the network.

To avoid the $O(n^3)$ cost of a pseudo‑inverse, the network maintains a **Mirrored Reciprocal Matrix** $V$ that evolves alongside the weight matrix $W$ and approximates $W^+$.

**Reconstruction Flow:**

1. **Target Input**: The layer above provides a required input $x^*_{l+1}$.
2. **Mirror Map**: The current layer finds the required activation:
   $$a^*_l = V_{l+1} \cdot x^*_{l+1}$$
3. **Invert Expression**: The neuron determines what its input should have been to produce that activation:
   $$x^*_l = b_l - \frac{\max(0, a^*_l)}{b_l}$$

### 3. The Learning Rule (Local Reconciliation)

Learning occurs when the network compares the actual weighted input $x$ with the reconstructed target input $x^*$.

**The Reconciliation Ratio:**

$$r = \frac{x^*}{x}$$

**Symmetric Weight Update:**
A connection $W_{ij}$ is updated based on the agreement between output neuron $i$ and input neuron $j$:

$$W_{ij} \leftarrow \sqrt{r_i \cdot r_j} \cdot W_{ij}$$

$$V_{ji} \leftarrow \frac{1}{\sqrt{r_i \cdot r_j}} \cdot V_{ji}$$

These multiplicative updates keep $V$ a running reciprocal of $W$ without any matrix inversion during inference.

**Memory Accumulation (Slow Timescale):**
Memory $b$ drifts slowly toward the reconstructed expectation, becoming the "best fit" for the environment:

$$b \leftarrow b + \alpha \cdot (x^* - b), \quad \text{where } \alpha \ll 1$$

---

## Stability & Constraints

To prevent the network from falling into a **Vacuum** (total suppression) or a **Supernova** (infinite activity), a local ratio between weights and memory is enforced.

**The Stability Constraint:**
Memory $b$ is kept proportional to the magnitude of incoming weights so that the neuron always operates in the "phase transition" zone:

$$b \propto \sqrt{\sum_j W^2_{ij}}$$

This ensures the "gain" of each neuron is tuned to the scale of its inputs, preventing numerical explosion and keeping Global Silence reachable.

---

## Physical & Biological Analogies

| Concept | Relativity Network | Biological / Physical Equivalent |
|---------|-------------------|----------------------------------|
| Energy | Activation $a$ | Metabolic cost of a neural spike |
| Memory | Bias $b$ | Resting membrane potential / synaptic threshold |
| Input | Weighted sum $x$ | Inhibitory / Excitatory gate voltage |
| Learning | Reconciliation | Homeostatic scaling / synaptic plasticity |
| Goal | Global Silence | Minimum energy state / predictive coding |
| Mechanism | Reciprocal $W \leftrightarrow V$ | Dendritic feedback loops |

---

## Comparison: Standard AI vs. Relativity Network

| Feature | Standard Neural Network | Relativity Network |
|---------|------------------------|--------------------|
| Objective | Minimise global loss | Reach local equilibrium |
| Signal | Additive gradients | Multiplicative ratios |
| Backward Pass | Error propagation | State reconstruction |
| Bias Role | Static offset | Dynamic memory of the environment |
| Convergence | Loss minimum | Network‑wide silence |
| Complexity | High (GPU clusters, backprop) | Low (local interactions, reciprocal) |
| Intelligence | Pattern matching | Refined compression |

---

## Open Questions & Roadmap

- **Readout Mechanism** – How to extract decisions from a network that converges to silence? Possible answer: silence on a familiar sample is the classification, or a separate decoding layer driven by unexplained variance.

- **Stability Analysis** – Tracking the condition number of $VW$ under continuous multiplicative updates, especially with batch‑to‑batch variance.

- **Sparsity** – Implementing hard‑pruning for connections where $r \approx 1$ over extended periods, reflecting "fully compressed" knowledge.

- **Dynamic Scaling** – Automating the $W/b$ ratio maintenance without hand‑tuned heuristics.

- **Benchmarking** – Testing on "Shifting Mean" tasks to demonstrate the efficiency of $b$-drift over weight‑retraining in non‑stationary environments.

- **Energy Mapping** – Quantifying "Watts per Inference" as the network converges toward silence, linking activity to actual computational cost.

---

## Summary

The Relativity Network is not an optimizer; it is an equilibrator. It posits that intelligence is not the ability to find a correct answer, but the ability to build an internal model so accurate that surprise approaches zero.

Learning is the path to silence.
