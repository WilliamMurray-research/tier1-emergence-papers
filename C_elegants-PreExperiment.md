# **A Dynamical Manifold Framework for Modelling the *C. elegans* Nervous System**  
### *Pre‑Experiment Paper*

---

# **Abstract**

The nematode *Caenorhabditis elegans* provides a uniquely tractable system for studying how behaviour emerges from neural structure. Despite its small size and fully mapped connectome, existing computational models typically focus on circuit‑level motifs or graph‑based simulations and do not capture the global geometry or temporal organisation of the nervous system. Here, we introduce a **dynamical manifold framework** in which the *C. elegans* connectome is embedded into a hyperbolic manifold and neural activity evolves under nonlinear recurrent dynamics. Behavioural motifs correspond to **coherent regimes**—stable interference patterns in the temporal evolution of the global state. This paper establishes the theoretical foundation, computational architecture, and experimental design for the forthcoming empirical study. The goal is not to present results but to define a rigorous, falsifiable modelling framework grounded in temporal geometry and emergence theory.

---

# **1. Introduction**

Understanding how behaviour arises from neural structure remains a central challenge in neuroscience. The nematode *C. elegans*, with its 302 neurons and fully mapped connectome, offers an unparalleled opportunity to study organism‑scale neural dynamics. Yet most computational models fall into two categories: **circuit‑level simulations** that focus on local motifs, and **graph‑based dynamical systems** that treat the connectome as a static adjacency structure. Neither approach captures the **global geometry** of the nervous system or the **temporal organisation** of its dynamics.

This paper introduces a **dynamical manifold framework** for modelling the *C. elegans* nervous system. The central idea is that neural activity traces a trajectory on a **curved geometric manifold**, and behavioural motifs correspond to **coherent regimes**—stable interference patterns in the system’s temporal geometry.

This is a **pre‑experiment paper**. Its purpose is to:

1. Present the theoretical foundation for the dynamical manifold approach.  
2. Define the computational architecture.  
3. Specify the experimental plan for the empirical paper.  

The next paper will present full simulation results, perturbation analyses, and phase diagrams.

---

# **2. Theoretical Framework**

## **2.1. Two‑Tier Emergence**

The framework builds on a two‑tier model of emergence:

- **First‑order emergence**: functional coherence arising from local interactions.  
- **Second‑order emergence**: invariants over coherence regimes.

*C. elegans* is expected to exhibit first‑order emergence only. This paper does not claim or imply second‑order emergence in a 302‑neuron system.

## **2.2. Temporal Interference**

Temporal Interference theory models cognition as **temporal geometry**:  
the system’s state evolves along trajectories whose interference patterns encode functional structure.

In this view:

- Behaviour corresponds to **stable interference regimes**.  
- Transitions correspond to **regime transitions**.  
- Perturbations reveal **invariants** of the temporal geometry.

## **2.3. Hyperbolic Dynamics**

Hyperbolic space provides a natural substrate for representing hierarchical, tree‑like, and sparse connectivity patterns. The *C. elegans* connectome exhibits:

- hierarchical organisation,  
- heterogeneous connectivity,  
- sparse long‑range structure.

Embedding the connectome into \(\mathbb{H}^3\) captures these properties and provides a geometric foundation for the dynamical model.

## **2.4. Relationship to Prior Work**

The **Hyperbolic Dynamics** paper introduced a generative model in which geometry produces the graph.  
Here, the direction is reversed: the biological graph is **embedded into** a manifold.  
This is an interpretive rather than generative use of geometry.

The **Temporal Interference** paper predicts that coherent regimes are structural features of the temporal geometry and should be **stable across clustering granularities**.  
This prediction is explicitly tested in the empirical paper via K‑sweeps.

## **2.5. Node‑State vs Edge‑State Models**

The Hyperbolic Dynamics paper used **edge‑state dynamics** to enforce non‑backtracking propagation.  
Here, we use a **node‑state model** because neurons—not synapses—are the primary state‑bearing units in biological literature.  
This is a deliberate modelling choice appropriate for *C. elegans*.

---

# **3. Computational Architecture**

## **3.1. Connectome Representation**

The nervous system is represented as a weighted directed graph  
\[
G = (V, E, W),
\]  
with adjacency matrix \(A\). Neurons are classified into sensory, interneuron, and motor groups.

## **3.2. Hyperbolic Embedding**

Each neuron is embedded into the hyperboloid model of \(\mathbb{H}^3\).  
We use **Sarkar’s hyperbolic multidimensional scaling (HMDS)**, which is well‑suited for sparse graphs and preserves shortest‑path structure (Sarkar, 2011).

The embedding minimises distortion between graph distances and hyperbolic distances.

## **3.3. Neural Dynamics**

Neural activity evolves according to:
\[
x(t+1) = \tanh(\lambda A x(t) + \beta x(t)).
\]

The self‑persistence term \(\beta x(t)\) is kept separate from the adjacency term rather than absorbed into the diagonal of \(A\).  
This allows **independent control of self‑persistence** during parameter sweeps (§4.4), enabling a cleaner phase‑diagram analysis.

## **3.4. Global State Representation**

The global state trajectory is projected into three dimensions using PCA:
\[
s(t) \in \mathbb{R}^3.
\]

This provides a low‑dimensional representation of the system’s temporal geometry.

## **3.5. Coherent Regime Detection**

Sliding windows of the global state are clustered into coherent regimes.

For this pre‑experiment paper, we fix:
\[
K = 6.
\]

This is **not** a biological claim.  
It is a provisional choice to demonstrate the pipeline.

In the empirical paper, **K will be swept** (e.g., \(K = 3 \dots 12\)) and regime stability will be evaluated using:

- silhouette scores,  
- cluster persistence,  
- transition‑graph consistency.

## **3.6. Behavioural Readout**

Motor neuron activity is aggregated into a scalar motor drive:
\[
u(t) = \frac{1}{|M|} \sum_{i \in M} x_i(t).
\]

Behaviour is classified as:

- **Forward** if \(u(t) > 0.3\)  
- **Reverse** if \(u(t) < -0.3\)  
- **Pause** otherwise  

This provides a coarse behavioural proxy for regime mapping.

---

# **4. Experimental Design**

## **4.1. Experiment 1 — Baseline Dynamics**

**Goal:** Identify coherent regimes in the unperturbed system.  
**Prediction:**  
If coherent regimes reflect genuine temporal geometry, they should be **stable across K** and exhibit **structured transitions**.

## **4.2. Experiment 2 — Behavioural Motif Mapping**

**Goal:** Map coherent regimes to behavioural classes.  
**Prediction:**  
Forward and reverse behaviours should correspond to **topologically distinct basins** in the PCA trajectory, reflecting the separation of AVA/AVB motor circuits in the hyperbolic embedding.

## **4.3. Experiment 3 — Perturbation Analysis**

### **Neuron Ablation**
Prediction:  
Ablation of key interneurons (e.g., AVA, AVB) should collapse or merge specific regimes.

### **Synaptic Scaling**
Prediction:  
Moderate scaling should preserve regime structure; large scaling should increase transition entropy.

### **Noise Injection**
Prediction:  
Noise should shorten dwell times but preserve core regimes.

## **4.4. Experiment 4 — Parameter Sweeps**

Sweep:

- \(\lambda\) (global coupling)  
- \(\beta\) (self‑persistence)  
- curvature \(K\)  

Prediction:  
Hyperbolic geometry should **shift the phase boundary** relative to a flat‑graph model.  
Specifically, more negative curvature should enlarge the stable region by increasing separation between dynamical basins.

## **4.5. Dashboard as an Experimental Instrument**

The dashboard provides:

- **PCA trajectory**  
- **Regime strip**  
- **Behaviour strip**  
- **Metrics panel**, including:  
  - regime entropy  
  - transition entropy  
  - regime dwell times  
  - behaviour frequencies  
  - perturbation parameters  

These metrics are defined here so the empirical paper can report against them.

---

# **5. Expected Outcomes (Derived Predictions)**

## **5.1. Regime Stability Across K**

Temporal Interference predicts that coherent regimes are structural features of the temporal geometry.  
Therefore, regime identities should remain stable across K‑sweeps.

## **5.2. Behavioural Correspondence**

Forward and reverse behaviours should map to distinct coherent regimes because:

- the hyperbolic embedding separates AVA‑dominated and AVB‑dominated circuits,  
- these circuits drive opposite motor outputs,  
- their separation creates distinct attractor basins.

## **5.3. Robustness Under Perturbation**

If coherent regimes reflect genuine dynamical structure, they should:

- persist under mild perturbations,  
- degrade smoothly under strong perturbations,  
- exhibit predictable changes in transition entropy.

## **5.4. Phase Diagram Structure**

The system should exhibit:

- a stable worm‑like region in parameter space,  
- a chaotic region at high \(\lambda\),  
- a degenerate region at low \(\lambda\),  
- **and a curvature‑dependent shift in the stability boundary**,  
  reflecting the role of hyperbolic geometry in separating dynamical basins.

---

# **6. Discussion**

This framework introduces a new modelling paradigm for small nervous systems.  
By combining hyperbolic geometry, nonlinear dynamics, and temporal interference, it provides a unified account of:

- structural organisation,  
- dynamical behaviour,  
- behavioural motifs,  
- robustness and invariants.

The approach differs from traditional circuit models by emphasising **global temporal geometry** rather than local interactions alone.

Limitations include:

- simplified neuron model,  
- coarse behavioural mapping,  
- no biomechanics in v1.

Future work will incorporate:

- geodesic updates,  
- learning rules,  
- biomechanical coupling,  
- scaling to larger organisms,  
- synthetic agents.

---

# **7. Conclusion**

This pre‑experiment paper establishes the theoretical and computational foundation for a dynamical manifold model of the *C. elegans* nervous system. The next paper will present full simulation results, perturbation analyses, and phase diagrams. Together, these works form the basis of a new approach to modelling biological and synthetic cognition.

---
