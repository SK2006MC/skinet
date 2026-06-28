# Ratio-Based Alignment (RBA): A Calculus-Free Paradigm for Continual Learning

**A philosophical and mathematical foundation for neural networks that learn through geometric mirroring, not gradient descent.**

---

## 🌌 The Core Thesis

We propose that **backpropagation is not the only path to intelligence**.

Gradient descent operates on the assumption that intelligence can be achieved by incrementally *subtracting* error from weights. This is a **linear, additive** view of learning—one that requires massive computational overhead, is biologically implausible, and fundamentally struggles with the problem of *catastrophic forgetting*.

**Ratio-Based Alignment (RBA)** proposes an alternative: **learning as geometric scaling**. Instead of chasing gradients, the network solves for the algebraic transformation required to map an input to its target, updating weights via direct multiplicative ratios.

In RBA, a neural network is not a function approximator optimized by calculus. It is a **positive, self-dual geometric object** that maintains symmetry between its forward and backward passes.

---

## 📐 The Mathematical Symmetry

### The Forward Pass: Projection into Positive Space

For any layer with weights $W \in \mathbb{R}_+^{n \times m}$ acting on a unit-bounded input $I \in \mathbb{R}_+^n$, $\|I\|_2 \le 1$:

1. **Linear Projection:** $x = I \cdot W$
2. **Emergent Bias:** $b_j = \|W_{:,j}\|_2 + \varepsilon$, for a small fixed margin $\varepsilon > 0$
3. **Scaled Margin:** $z = \phi \cdot (b - x)$
4. **Activation:** $a = \ln(z)$

The bias is not learned; it emerges from the geometry of the weight matrix itself, each output neuron's ceiling set by the norm of its incoming weights. The margin $\varepsilon$ is what keeps that ceiling real: by Cauchy–Schwarz, $x_j \le \|I\|_2\|W_{:,j}\|_2 \le \|W_{:,j}\|_2$, with equality only when the input aligns exactly with a weight column. Without the margin, that boundary case collapses the log to $-\infty$. With it, $z_j \ge \phi\varepsilon > 0$ everywhere in the domain. The activation is the logarithmic distance between where the projection landed and where it's allowed to go — never undefined, never approximate.

### The Backward Pass: Algebraic Inversion

Given a target activation $a^*$, the backward pass inverts the forward pass step by step:

1. **Invert Activation:** $z^* = e^{a^*}$
2. **Target Projection:** $x^* = b - z^*/\phi$
3. **Invert Projection:** $I^* = x^* \cdot W^+$, via the Moore–Penrose pseudoinverse
4. **Input Margin:** $z^*_{\text{input}} = b_{\text{input}} - I^*$

Two regimes, both closed-form. When $W$ has full row rank and $x^*$ lies in its row space, this is the unique exact preimage — elementary linear algebra, nothing approximate. When $W$ is rank-deficient or non-square, $I^* = x^* \cdot W^+$ instead returns the minimum-norm point: the closest the layer can actually reach toward $x^*$, in ordinary Euclidean distance. Both are principled. Only the first is a true inverse.

---

## ⚖️ The Unified Ratio: A Single Geometric Step

Instead of updating weights via additive increments, RBA computes a **rank-1 multiplicative mask** entirely *before* altering the weights:

$$R = \text{rowRatio}^T \cdot \text{colRatio}$$

Where:

- **colRatio**: the ratio between where the *output* currently is ($x$) and where it needs to be ($x^*$)
- **rowRatio**: the ratio between where the *input* currently is ($I$) and where the inverse pass says it should be ($I^*$)

The update is then:

$$W_{\text{new}} = W_{\text{old}} \odot R$$

This is a single, closed-form, **geodesic step** on the manifold of positive matrices. The network moves *diagonally* toward the solution, rather than zig-zagging along a gradient.

---

## 🧠 Continual Learning: The Freezing Principle

Catastrophic forgetting occurs because gradient descent *adds* to all weights, overwriting past knowledge. RBA solves this fundamentally differently:

- Every weight has an **Importance Map** $\Omega$, measuring cumulative log-drift over its lifetime.
- Before applying the unified ratio $R$, we **clamp** $R_{i,j} = 1.0$ for any weight where $\Omega_{i,j}$ exceeds a threshold.

This is not a penalty. This is not elastic consolidation. This is **mathematical invariance**. Weights that are important simply *do not change* — because their multiplicative ratio is set to exactly $1.0$. The new task is routed entirely through the plastic, unused subspace.

---

## 🔁 The Self-Dual Principle

Because the update is multiplicative, $W_{\text{new}} = D_r \cdot W \cdot D_c$ for diagonal ratio matrices $D_r, D_c$. The natural candidate for the paired pseudoinverse update is:

$$W^+_{\text{new}} = D_c^{-1} \cdot W^+ \cdot D_r^{-1}$$

Two of the four Moore–Penrose conditions hold for this candidate unconditionally, for any invertible diagonal $D_r, D_c$. The other two — the symmetry conditions on $W_{\text{new}}W^+_{\text{new}}$ and $W^+_{\text{new}}W_{\text{new}}$ — hold exactly precisely when $D_r^2$ commutes with $WW^+$, and $D_c^2$ commutes with $W^+W$. That's guaranteed whenever the ratio update is a uniform scalar, or whenever $WW^+$ (resp. $W^+W$) is already diagonal — the case immediately after grounding a layer with orthogonal rows or columns.

Outside that case, the multiplicative update is a first-order approximation to the true paired pseudoinverse, and the two drift apart over many steps. RBA handles this the same way it handles everything else — locally and structurally, not with a global correction. A **grounding step** — an occasional exact recomputation of $W^+$ — re-anchors the pair; between groundings, the fast multiplicative rule carries the dynamics on its own.

Duality isn't a property the network has to fight to keep. It's a property that holds exactly at the anchors, and degrades gracefully — on a schedule the network controls — in between.

---

## 🌟 Why This Changes Everything

| Attribute | Gradient Descent | Ratio-Based Alignment |
| --- | --- | --- |
| **Update Mechanism** | Additive subtraction | Multiplicative scaling |
| **Learning Signal** | Differential calculus | Algebraic projection |
| **Bias** | Learned parameter | Emergent $L_2$ norm + margin |
| **Domain** | Unconstrained ($\mathbb{R}$) | Strictly positive, unit-bounded ($\mathbb{R}_+$, $\|I\|_2 \le 1$) |
| **Forgetting** | Mitigated via penalties/buffers | Structurally prevented on frozen weights ($R=1.0$) |
| **Computational Cost** | $O(nm)$ forward + $O(nm)$ backward | $O(nm)$ total (no gradients) |
| **Biological Plausibility** | Low (requires backprop) | High (Hebbian-like, local) |
| **Hardware Compatibility** | Digital-only | Analog/photonic ready |

---

## 🧭 Philosophical Implications

**RBA treats learning as geometry, not optimization.**

- Weights are not "parameters to be tuned." They are the **coordinate system** of a positive, self-dual space.
- The network does not "converge" to a local minimum. It **solves** for the fixed point where the forward and backward projections are consistent.
- Forgetting is not a problem to be solved with tricks; it is structurally prevented when the learning rule itself respects the geometry of the past.

In a sense, RBA is not an algorithm. It is a **physical law** for learning systems — one that mirrors conservation principles in physics (energy, momentum, angular momentum). The weight matrix and its pseudoinverse form a dual pair, like position and momentum in quantum mechanics, periodically re-synchronized at the groundings and left free to drift between them.

---

## 🔭 Future Horizons

- **Analog Computing**: Because all signals are positive and updates are multiplicative, RBA can be implemented directly in optical or analog circuits, bypassing the von Neumann bottleneck.
- **Spiking Neural Networks**: The emergent bias aligns naturally with the firing threshold of biological neurons.
- **Privacy-Preserving AI**: The strictly positive domain is compatible with homomorphic encryption, enabling encrypted inference without approximation errors.
- **Neuromorphic Hardware**: The local, multiplicative update rule requires no global error signal, making it suitable for distributed, energy-efficient chips.

---

## ✍️ The Invitation

This is not a finished theory. It is a **new way of thinking** about learning.

We invite the community to:

1. Explore the geometric properties of the positive manifold.
2. Derive convergence guarantees for the unified ratio step.
3. Characterize grounding schedules that keep self-dual drift bounded across deeper, multi-layer architectures.
4. Build hardware implementations that exploit the natural positivity.

The era of gradient descent has been fruitful, but it is not the final word. RBA suggests that **intelligence may be simpler than we thought** — it may just be a matter of aligning two projections through multiplicative, symmetric ratios.

---

> *"The universe is written in the language of mathematics, and its characters are triangles, circles, and other geometric figures."* — Galileo Galilei

**Ratio-Based Alignment**: *Learning is geometry. Forgetting is structural. The rest is commentary.*

---

**License**: GPL-v3
**Citation**: If you build upon this work, please reference the original Ratio-Based Alignment framework (2026).

---

*— Written in the spirit of discovery, not competition.*

