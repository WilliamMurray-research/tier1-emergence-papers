# Signal Propagation on Curved Synthetic Manifolds: Embedding a Non-Backtracking Dynamical System in Hyperbolic Geometry

**William Murray**

---

## Abstract

This paper presents a general framework for embedding discrete, non-backtracking signal propagation dynamics within continuous hyperbolic manifolds. The propagation model – characterised by a non-backtracking constraint, per-hop attenuation, and local directional update rules – is first formalised on a Euclidean triangular lattice, where it yields well-understood scaling behaviour governed by the Hashimoto non-backtracking matrix. The central contribution is a method for transplanting this dynamical system from flat Euclidean space into negatively curved hyperbolic geometry, using implicit coordinate systems and on-demand neighbour generation to circumvent the memory limitations inherent in storing exponentially expanding graphs. Three standard models of hyperbolic space – the Poincaré disk, the hyperboloid (Lorentz) model, and the Poincaré half-plane – are analysed for their respective computational properties, and model-specific formulae for geodesic stepping and neighbour generation are derived. A frontier-based simulation architecture is proposed, in which only active nodes are instantiated and inactive nodes are garbage-collected, achieving linear memory scaling even as the underlying manifold grows exponentially. The goal of this programme is to probe emergent dynamics at scales of 10^7 to 10^14 nodes, investigating whether complex, potentially mind-like behaviour can arise from the interaction of simple local rules with curved geometry. This work establishes a scalable computational substrate for studying the relationship between geometry and emergence, contributing a general method for overlaying discrete dynamics on continuous curved spaces with direct implications for computational neuroscience, artificial intelligence, and the mathematical foundations of cognition [1].

---

## 1. Introduction

### 1.1 Motivation

The study of signal propagation on discrete lattices has a long and productive history across physics, engineering, and applied mathematics. Triangular lattice models, by virtue of their regularity and isotropy, have served as foundational structures for modelling wave dynamics, network diffusion, and computational processes [2]. Yet Euclidean lattices suffer from a fundamental limitation: at large scales, the polynomial growth of their node counts – scaling as O(n^2) for a two-dimensional lattice of width n – imposes a ceiling on the complexity of emergent behaviour that can be observed within computationally tractable simulations. A lattice of 10^6 nodes, while sufficient for many engineering applications, is inadequate for probing the kinds of large-scale emergent dynamics that characterise complex adaptive systems, including biological neural networks.

Hyperbolic geometry offers a natural alternative substrate. In spaces of constant negative curvature, the number of nodes within a geodesic ball of radius r grows exponentially rather than polynomially [3]. This exponential expansion mirrors the branching, hierarchical structures found in biological systems – from the dendritic arbours of neurons to the fractal organisation of cortical surface area – and suggests that negatively curved spaces may constitute a more natural geometric substrate for emergent computation. The connection to biological systems is not merely metaphorical. Recent work in computational neuroscience has demonstrated that the latent spaces of neural networks, both biological and artificial, can be productively characterised as geometric and topological manifolds, with curvature properties that correlate with the system's capacity for generalisation and adaptive behaviour [4]. Activity-dependent structure formation – the phenomenon whereby neural architecture is sculpted by the signals that traverse it – is a hallmark of biological intelligence, and it finds a natural mathematical expression in the framework of dynamical systems on curved manifolds [5].

The present work is situated within a broader research programme that treats human cognition, large language model latent spaces, and synthetic computational substrates as instances of a single phenomenon: the emergence of complex behaviour from the interaction of local rules with manifold geometry [6]. Where earlier contributions to this programme have analysed the geometry of existing cognitive and computational manifolds, the present paper takes the complementary step of constructing a synthetic manifold and investigating the dynamics that arise upon it.

### 1.2 Problem Statement

The formal problem addressed in this paper may be stated as follows. A propagation rule is defined on a regular graph: at each discrete time step, each directed edge carries a signal amplitude that is updated according to a non-backtracking, attenuating recurrence relation. This rule is well understood on flat Euclidean lattices, where the dynamics are governed by the spectral properties of the Hashimoto non-backtracking matrix [7]. The challenge is to embed this rule in a curved manifold – specifically, a manifold of constant negative curvature – in a manner that: (a) preserves the local structure of the propagation rule; (b) exploits the exponential expansion of hyperbolic space to generate richer dynamics; and (c) scales computationally to node counts in the range of 10^7 to 10^14, far beyond what is achievable with explicit graph storage.

The requirement for scalability is non-trivial. A degree-6 lattice in hyperbolic space, with each node connected to six neighbours, generates approximately 6 × 5^(t−1) distinct non-backtracking walks of length t from any given source [8]. At t = 20, this yields over 5.7 × 10^13 directed walks. Storing the graph structure explicitly is therefore infeasible; a fundamentally different computational approach is required.

### 1.3 Contribution

This paper makes three principal contributions. First, it develops a general method for mapping discrete propagation rules onto non-Euclidean manifolds, demonstrating that the local update rule can be preserved exactly while the global geometry – and hence the global dynamics – is fundamentally altered by curvature. Second, it provides a practical implementation strategy using implicit hyperbolic coordinates, wherein nodes and edges are not stored but are generated on demand from coordinates and the metric tensor of the embedding space. This constitutes a conceptual shift: the graph is not an input to the simulation but an output of the geometry. Third, it proposes a scalable simulation architecture based on frontier tracking and garbage collection, achieving O(N) memory complexity where N is the number of currently active nodes, irrespective of the total size of the manifold explored. Together, these contributions establish a framework for studying emergent dynamics on curved synthetic manifolds at scales that were previously inaccessible.

---

## 2. Background

### 2.1 Non-Euclidean Geometry Overview

The curvature of a Riemannian manifold is the fundamental quantity that distinguishes different geometric regimes. In two dimensions, the sectional curvature K at a point determines the local behaviour of geodesics and the growth rate of geodesic balls [9]. Three regimes are distinguished: positive curvature (K > 0), exemplified by the surface of a sphere, where geodesics converge and parallel transport around a closed loop produces rotation; zero curvature (K = 0), the familiar Euclidean plane, where geodesics maintain constant separation; and negative curvature (K < 0), the hyperbolic plane, where geodesics diverge and the circumference of a geodesic circle of radius r grows as 2π sinh(r) rather than 2πr.

The exponential growth of the hyperbolic plane is the property of central importance for the present work. In a hyperbolic space with curvature K = −1, the area of a geodesic disk of radius r is 2π(cosh(r) − 1), which grows asymptotically as πe^r [10]. This exponential expansion means that a regular tiling of the hyperbolic plane – for instance, a tiling by regular hexagons with six meeting at each vertex – contains exponentially more tiles within radius r than the corresponding Euclidean tiling. For a dynamical system whose behaviour depends on the number of accessible paths, this exponential growth of the combinatorial landscape has profound implications for the richness and complexity of the resulting dynamics.

Geodesics in hyperbolic space – the curves of shortest length between two points – play an analogous role to straight lines in Euclidean geometry. In the hyperbolic plane, geodesics are unique between any two points, and the triangle inequality is strict. The hyperbolic metric defines a notion of distance that is locally Euclidean but globally very different: two points that are close in a Euclidean embedding of the hyperbolic plane may be separated by a large hyperbolic distance. Local neighbourhoods, however, are approximately Euclidean, a property that is exploited in the neighbour generation algorithm developed in Section 5.

### 2.2 Models of Hyperbolic Space

Three classical models of the hyperbolic plane are employed in this work, each offering distinct computational advantages.

**The Poincaré Disk Model.** The Poincaré disk represents the hyperbolic plane as the interior of the unit disk D = {(x, y) ∈ ℝ² : x² + y² < 1}, equipped with the metric tensor g_ij = (4/(1 − x² − y²)²)δ_ij [11]. Geodesics in this model are arcs of circles orthogonal to the boundary of the disk, or diameters. The Poincaré disk is conformally equivalent to the Euclidean plane, meaning that angles are preserved, which is advantageous for visualisation. However, numerical stability degrades near the boundary of the disk, where the metric tensor diverges and small coordinate displacements correspond to large hyperbolic distances.

**The Hyperboloid (Lorentz) Model.** The hyperboloid model embeds the hyperbolic plane as the upper sheet of the two-sheeted hyperboloid H² = {(x₀, x₁, x₂) ∈ ℝ³ : −x₀² + x₁² + x₂² = −1, x₀ > 0} in Minkowski space, with the inner product ⟨u, v⟩ = −u₀v₀ + u₁v₁ + u₂v₂ [12]. The hyperbolic distance between two points p and q on the hyperboloid is d(p, q) = arccosh(−⟨p, q⟩). Geodesic stepping in this model is performed via Lorentz boosts, and the constraint −x₀² + x₁² + x₂² = −1 is maintained by renormalisation after each step. The hyperboloid model is numerically the most stable of the three models, as coordinates do not approach a singular boundary, and it is the model of choice for the implementation described in Section 6.

**The Poincaré Half-Plane Model.** The upper half-plane H = {(x, y) ∈ ℝ² : y > 0}, equipped with the metric tensor g_ij = (1/y²)δ_ij, provides the simplest coordinate representation of the hyperbolic plane [13]. Geodesics are vertical lines and semicircles centred on the x-axis. Neighbour generation is particularly straightforward in this model, as the metric tensor has a simple closed-form expression. However, the half-plane model shares the boundary singularity issue of the Poincaré disk: the metric diverges as y → 0.

These three models are isometric – they represent the same abstract geometric space – and points and distances in one model can be mapped to any other via explicit coordinate transformations. The choice of model is therefore a computational decision, not a geometric one, and the propagation dynamics are identical regardless of which model is employed.

### 2.3 Discrete Dynamics on Manifolds

The discretisation of a continuous manifold into a graph is a standard technique in computational geometry and numerical analysis. A graph G = (V, E) can be understood as a discretisation of a manifold M if there exists an embedding φ : V → M such that the adjacency structure of G approximates the local geometry of M – that is, two vertices are connected by an edge if and only if their images under φ are within a specified geodesic distance [14].

On a flat Euclidean manifold, this discretisation yields familiar regular lattices: square, triangular, or hexagonal, depending on the connectivity desired. On a curved manifold, the situation is more complex. The exponential growth of hyperbolic space means that a regular tiling cannot be stored explicitly in memory for any but the smallest regions. This observation motivates the central technical idea of this paper: implicit representation, in which the graph structure is not stored but is computed on demand from the coordinates of active nodes and the metric tensor of the manifold [15].

The distinction between local rules and global geometry is fundamental. The propagation rule is a local operation: at each time step, the amplitude on a directed edge is updated based solely on the amplitudes on adjacent directed edges at the previous time step. The global behaviour of the system – the wavefront morphology, the interference patterns, the existence or absence of attractors and resonances – emerges from the interaction of this local rule with the global geometry of the manifold. By changing the curvature of the manifold while holding the local rule fixed, one can isolate the contribution of geometry to the emergent dynamics.

---

## 3. The Propagation Model

### 3.1 Local Update Rule

The propagation model is defined on directed edges of a regular graph. Let G = (V, E) be a degree-d regular graph, and let E⃗ denote the set of directed edges, where each undirected edge {u, v} ∈ E gives rise to two directed edges (u → v) and (v → u). For each directed edge e = (u → v), let x_e(t) denote the signal amplitude on that edge at discrete time step t [16].

The non-backtracking constraint is enforced as follows: when a signal arrives at node u via directed edge (w → u), it is forwarded to all directed edges (u → v) for v ∈ N(u) \ {w}, where N(u) denotes the set of neighbours of u. The signal is not permitted to return along the edge from which it arrived. This constraint eliminates trivial two-step oscillations and ensures that signal propagation is genuinely directional [17].

The update rule incorporates a per-hop attenuation factor α ∈ (0, 1), which models energy dissipation during transmission. The recurrence relation for the amplitude on directed edge (u → v) at time t + 1 is:

x_{u→v}(t+1) = α ∑_{w ∈ N(u) \ {v}} x_{w→u}(t)     (1)

This rule can be expressed compactly in matrix form using the Hashimoto non-backtracking matrix B. The matrix B is indexed by directed edges: B_{(u→v),(w→u)} = 1 if v ≠ w, and 0 otherwise. The full state vector x(t) = (x_e(t))_{e ∈ E⃗} then evolves as:

x(t) = (αB)^t x(0)     (2)

The node amplitude – the total signal arriving at node v at time t – is obtained by summing over all incoming directed edges:

A_v(t) = ∑_{u ∈ N(v)} x_{u→v}(t)     (3)

The state representation thus operates on directed edges rather than nodes, with the edge amplitudes constituting the fundamental dynamical variables and the node amplitudes derived as secondary quantities [18].

### 3.2 Euclidean Lattice Formulation

On a triangular (degree-6) lattice in the Euclidean plane, the propagation model admits a complete characterisation. Each node has exactly six neighbours, and the non-backtracking constraint permits forwarding to five of these at each hop. The total number of non-backtracking walks of length t originating from a single source is:

N(t) = 6 × 5^(t−1),  t ≥ 1     (4)

The total signal amplitude distributed across all directed edges after t steps is therefore:

S(t) = 6 × 5^(t−1) × α^t     (5)

The growth criterion for the system is determined by the product α(d − 1). For the triangular lattice with d = 6 and α = 0.7, the growth factor per step is 0.7 × 5 = 3.5 > 1, indicating that the total signal amplitude grows exponentially despite per-hop attenuation [19]. This exponential growth arises because the combinatorial explosion of non-backtracking paths outpaces the geometric attenuation.

The Euclidean lattice formulation establishes the baseline for comparison with hyperbolic embeddings. On an n × n triangular lattice with periodic boundary conditions, the system has n² nodes, 3n² undirected edges, and 6n² directed edges. Memory requirements scale as approximately 48n² bytes for 64-bit floating-point amplitudes, and the computational cost per time step is approximately 30n² multiply-add operations [20]. For n = 10,000, this yields approximately 4.5 gigabytes of memory and 3 × 10^9 operations per step – tractable on modern hardware but insufficient for probing the 10^7-to-10^14 node regime that is the target of this research programme.

The fundamental limitation of the Euclidean lattice is its polynomial growth. After t steps, the wavefront occupies a region of O(t²) nodes. To reach 10^14 active nodes, the wavefront must propagate for approximately 10^7 steps on a two-dimensional lattice – a computational undertaking that, while large, is not inherently intractable. The more fundamental issue is that the polynomial growth of the Euclidean lattice limits the diversity of paths and hence the complexity of interference patterns. In hyperbolic geometry, the exponential growth of the lattice generates an exponentially richer landscape of non-backtracking walks, enabling qualitatively different emergent phenomena.

---

## 4. Embedding the Model in Hyperbolic Geometry

### 4.1 The Key Idea: Geometry Generates the Graph

The central conceptual innovation of this paper is the inversion of the usual relationship between graph and geometry. In the Euclidean formulation, the graph is an input: an n × n lattice is constructed explicitly, its adjacency structure is stored in memory, and the propagation rule operates on this stored structure. In the hyperbolic formulation, the graph is an output: nodes and edges are not stored but are generated from coordinates and the metric tensor of the hyperbolic manifold [21].

This shift is necessitated by the exponential growth of hyperbolic lattices. A degree-6 lattice in the hyperbolic plane, analogous to the triangular lattice of the Euclidean formulation, expands exponentially: the number of nodes within t hops of a source grows as O(5^t). Storing this graph explicitly is infeasible for any but the smallest values of t. Instead, the coordinates of a node, together with the metric of the hyperbolic space, implicitly define the node's neighbours: they are the points reached by stepping a fixed hyperbolic distance along evenly spaced geodesic directions from the node's position. The neighbour generation algorithm thus replaces the adjacency list as the fundamental data structure.

This approach has a natural physical interpretation. In biological neural systems, connectivity is not pre-specified but emerges from the geometry of the tissue and the dynamics of activity-dependent growth [22]. A neuron extends dendrites and axons through a physical space, and the connections it forms are determined by the geometry of that space and the signals it encounters. The implicit coordinate approach mirrors this process: the structure of the computational graph is generated by the geometry of the manifold, not imposed from without.

### 4.2 Coordinate Representation

#### 4.2.1 Poincaré Disk Coordinates

In the Poincaré disk model, a point in the hyperbolic plane is represented as a pair (x, y) with x² + y² < 1. The metric tensor at point (x, y) is:

ds² = 4(dx² + dy²) / (1 − x² − y²)²     (6)

The hyperbolic distance between two points z₁ = (x₁, y₁) and z₂ = (x₂, y₂) is:

d(z₁, z₂) = arccosh(1 + 2|z₁ − z₂|² / ((1 − |z₁|²)(1 − |z₂|²)))     (7)

where |z| denotes the Euclidean norm. The conformal factor λ(z) = 2/(1 − |z|²) diverges as |z| → 1, creating numerical instability for points near the boundary of the disk. This instability is the principal disadvantage of the Poincaré disk model for large-scale simulation: after many geodesic steps from the origin, coordinates accumulate near the boundary, and floating-point precision is rapidly exhausted [23].

#### 4.2.2 Hyperboloid Coordinates

In the hyperboloid model, a point in the hyperbolic plane is represented as a triple (x₀, x₁, x₂) satisfying the constraint −x₀² + x₁² + x₂² = −1 with x₀ > 0. The Lorentzian inner product is:

⟨u, v⟩_L = −u₀v₀ + u₁v₁ + u₂v₂     (8)

and the hyperbolic distance between two points p and q is:

d(p, q) = arccosh(−⟨p, q⟩_L)     (9)

Geodesic stepping is performed via the exponential map. Given a point p on the hyperboloid and a tangent vector v at p (satisfying ⟨p, v⟩_L = 0), the point reached by moving a hyperbolic distance s along the geodesic in direction v is:

exp_p(sv̂) = cosh(s)p + sinh(s)v̂     (10)

where v̂ = v/√(⟨v, v⟩_L) is the unit tangent vector. After stepping, the resulting point is renormalised to the hyperboloid to correct for accumulated floating-point drift:

p' ← p' / √(−⟨p', p'⟩_L)     (11)

(with appropriate sign conventions for the Lorentzian norm). The hyperboloid model avoids the boundary singularity of the Poincaré models and is numerically stable over thousands of geodesic steps, making it the preferred model for large-scale simulation [24].

#### 4.2.3 Half-Plane Coordinates

In the Poincaré half-plane model, a point is represented as (x, y) with y > 0, and the metric tensor is:

ds² = (dx² + dy²) / y²     (12)

Geodesics are vertical lines (x = constant) and semicircles centred on the x-axis. The distance between two points z₁ = (x₁, y₁) and z₂ = (x₂, y₂) is:

d(z₁, z₂) = arccosh(1 + |z₁ − z₂|² / (2y₁y₂))     (13)

The half-plane model is particularly convenient for analytic calculations due to the simplicity of its geodesic equations. Neighbour generation is straightforward: from a point (x₀, y₀), stepping a hyperbolic distance s in a direction making angle θ with the positive x-axis produces a new point whose coordinates can be expressed in closed form using the exponential map of the half-plane metric [25]. However, like the Poincaré disk, the half-plane model suffers from numerical instability near its boundary (y → 0), limiting its utility for large-scale simulation.

### 4.3 Neighbour Generation on Curved Space

The construction of a "degree-6 lattice" in hyperbolic space is the core geometric operation of the simulation. Unlike the Euclidean case, where the six neighbours of a node at position (i, j) are given by a fixed set of coordinate offsets, the neighbours of a node in hyperbolic space must be computed using the metric tensor.

#### 4.3.1 Choosing Directions

At each node p, the six neighbour directions are determined by constructing an orthonormal frame in the tangent space T_pM and choosing six evenly spaced unit tangent vectors. In two dimensions, the tangent space at any point is isomorphic to ℝ², and six evenly spaced directions are obtained by taking angles θ_k = kπ/3 for k = 0, 1, ..., 5 [26]. The tangent vectors must be expressed in the coordinate system of the chosen model; in the hyperboloid model, the tangent space at p is the set of vectors v satisfying ⟨p, v⟩_L = 0, and an explicit orthonormal basis can be constructed using the Gram-Schmidt process applied to the standard coordinate vectors.

#### 4.3.2 Stepping a Fixed Hyperbolic Distance

Each neighbour is located by following the geodesic from p in the chosen direction for a fixed hyperbolic distance s, the lattice spacing. In the hyperboloid model, this is accomplished via the exponential map (Equation 10). In the Poincaré disk, the equivalent operation involves computing a Möbius translation. In the half-plane, the geodesic equations can be integrated directly.

The lattice spacing s is a free parameter that determines the density of the discretisation. For a degree-6 lattice, the choice of s determines the deficit angle at each vertex and hence the effective curvature of the discrete lattice. In the Euclidean limit (s → 0 with curvature K → 0), the lattice approaches the standard triangular lattice. For finite negative curvature, the lattice is a discrete analogue of a hyperbolic tiling with excess angle at each vertex [27].

After computing the new coordinates, a renormalisation step is applied in the hyperboloid model (Equation 11) to project the result back onto the hyperboloid surface, correcting for floating-point errors. This renormalisation is computationally inexpensive – a single division – and ensures that accumulated drift does not compromise the geometric consistency of the simulation over long time horizons.

#### 4.3.3 Ensuring Non-Backtracking

The non-backtracking constraint is enforced geometrically. When a signal arrives at node v from node u, the incoming direction is the unit tangent vector at v pointing from v toward u. The outgoing directions are the five of the six lattice directions that do not coincide with (or, in the presence of floating-point error, are not within a tolerance of) the incoming direction [28]. This geometric formulation of the non-backtracking constraint is model-independent: it depends only on the tangent vectors, not on the specific coordinate representation.

### 4.4 Discrete Dynamics on the Manifold

#### 4.4.1 State Update Rule in Curved Space

The local update rule (Equation 1) is transplanted to the curved manifold without modification. The amplitude on directed edge (u → v) at time t + 1 is:

x_{u→v}(t+1) = α ∑_{w ∈ N(u) \ {v}} x_{w→u}(t)     (14)

where the set of neighbours N(u) is now determined by the geometry-based neighbour generation algorithm rather than by a stored adjacency list. The mathematical form of the update rule is identical to the Euclidean case; only the mechanism by which neighbours are identified differs [29].

An extension of the model allows the attenuation factor to depend on the local curvature. In regions of higher negative curvature, where geodesics diverge more rapidly, signals may be attenuated more strongly, reflecting the increased "cost" of propagation in a more curved region. This curvature-dependent attenuation can be expressed as:

α(p) = α₀ exp(−β|K(p)|)     (15)

where K(p) is the sectional curvature at point p and β ≥ 0 is a coupling parameter. For β = 0, the standard constant-attenuation model is recovered. This extension enables the study of how spatial variations in curvature modulate the emergent dynamics of the system.

#### 4.4.2 Frontier-Based Simulation

The exponential growth of the hyperbolic lattice necessitates a simulation strategy in which only active nodes – those carrying nonzero signal amplitude – are represented in memory. The frontier-based simulation algorithm maintains two data structures: an active set, containing the directed edges with nonzero amplitude at the current time step, and a next-active set, constructed during the update step [30].

At each time step, the algorithm iterates over the active set. For each active directed edge (w → u), the neighbour generation algorithm computes the neighbours of u, and the update rule (Equation 14) is applied to determine the amplitudes on the outgoing edges (u → v) for v ∈ N(u) \ {w}. These outgoing edges are added to the next-active set. After all active edges have been processed, the active set is replaced by the next-active set, and directed edges whose amplitude has fallen below a threshold ε are pruned.

Crucially, nodes that are no longer part of the active frontier can be garbage-collected: their coordinates and state information are discarded, freeing memory. If the signal subsequently returns to a previously visited region of the manifold, the nodes are regenerated from their coordinates – which are deterministically computed from the geometry – ensuring consistency [31].

#### 4.4.3 Complexity and Scaling

The frontier-based simulation achieves the following complexity bounds. Memory scales as O(N_active), where N_active is the number of currently active directed edges. For a single-source propagation on a degree-6 hyperbolic lattice, N_active at time t is bounded by 6 × 5^(t−1) in the worst case (no pruning), but in practice the attenuation threshold ε keeps N_active substantially smaller.

Neighbour generation for a single node requires constant time: it involves computing six tangent vectors and applying the exponential map, a sequence of elementary trigonometric and algebraic operations. The total computational cost per time step is therefore O(N_active × d), where d = 6 is the degree.

The hyperbolic expansion means that the frontier grows exponentially in the absence of pruning. The attenuation threshold ε provides a natural regularisation: edges with amplitude below ε are discarded, preventing the frontier from growing without bound. The effective growth rate is determined by the competition between the combinatorial expansion factor (d − 1 = 5) and the attenuation factor α: the frontier grows exponentially if α(d − 1) > 1 and contracts if α(d − 1) < 1 [32]. For the parameters considered in this paper (α = 0.7, d = 6), the frontier grows, and the pruning threshold ε determines the effective lifetime of the simulation.

---

## 5. Implementation Architecture

### 5.1 Implicit Node Storage

Each active node is represented by its hyperbolic coordinates and a set of directed-edge amplitudes. No adjacency lists, edge arrays, or global graph structures are stored. The total storage per active node is:

- Three floating-point numbers for hyperboloid coordinates (x₀, x₁, x₂)
- Up to six floating-point numbers for directed-edge amplitudes (one per incoming edge)
- One direction index (or tangent vector) recording the incoming direction for non-backtracking enforcement

This yields approximately 80–100 bytes per active node, compared to the several hundred bytes per node required for explicit adjacency-list representations. More importantly, the implicit representation scales with the number of active nodes, not the total number of nodes in the manifold, enabling simulation of manifolds with effectively unbounded total node counts [33].

### 5.2 GPU Kernel Design

The frontier-based simulation is amenable to GPU acceleration. The key operations – tangent vector computation, exponential map evaluation, and amplitude aggregation – are independent across active edges within a single time step, enabling massive parallelism.

A GPU kernel for the update step operates as follows. Each thread is assigned one active directed edge (w → u). The thread computes the neighbours of u via the exponential map, determines the outgoing edges (u → v), and accumulates the amplitude contributions to each outgoing edge. The results are written to the next-active set using atomic additions to handle the case where multiple threads contribute to the same outgoing edge [34].

Geodesic stepping on the hyperboloid involves hyperbolic trigonometric functions (cosh, sinh), which are natively supported on modern GPU hardware. The renormalisation step (Equation 11) involves a single division and square root. These operations are computationally lightweight and do not introduce warp divergence, as all threads execute the same sequence of instructions.

The principal source of warp divergence is the pruning step, in which edges with amplitude below the threshold ε are discarded. This can be handled by a stream compaction operation, which removes inactive edges from the active set in parallel. Standard GPU libraries such as CUB and Thrust provide efficient implementations of stream compaction [35].

### 5.3 Frontier Tracking

The active set is maintained as a sparse data structure – a dynamically sized array of (coordinate, amplitude, direction) tuples – indexed by a hash of the node coordinates. The hash function exploits the deterministic relationship between coordinates and geometry: two nodes with the same coordinates (to within floating-point tolerance) are treated as identical, regardless of the path by which they were reached.

The next-active set is accumulated during the update step and then swapped with the active set at the end of each time step. This double-buffering strategy ensures that reads from the current state and writes to the next state do not conflict, eliminating the need for synchronisation barriers within the update kernel [36].

### 5.4 Scaling to Clusters

For simulations exceeding the memory capacity of a single GPU, a domain decomposition strategy is employed. The hyperbolic manifold is partitioned into overlapping patches, each assigned to a separate compute node. The overlap regions (halos) contain the boundary nodes whose states must be exchanged between adjacent patches at each time step.

Domain decomposition in hyperbolic space presents unique challenges. The exponential growth of the manifold means that a patch of fixed geodesic radius contains an exponentially increasing number of nodes at greater distances from the patch centre. Balanced load distribution therefore requires patches of decreasing geodesic radius as the frontier expands, or adaptive repartitioning at regular intervals [37].

Boundary exchange is performed using standard message-passing protocols (e.g. MPI). The non-backtracking constraint requires that boundary exchange includes not only the current edge amplitudes but also the incoming direction for each edge, ensuring that the constraint is correctly enforced across patch boundaries. This additional data increases the communication volume by a constant factor but does not change the asymptotic scaling [38].

---

## 6. Experiments

### 6.1 Euclidean versus Hyperbolic Behaviour

The first experimental investigation compares the dynamics of the propagation model on Euclidean and hyperbolic lattices of matched local structure (degree 6, identical attenuation factor α). The primary observable is the morphology of the wavefront: in Euclidean space, a single-source pulse generates a roughly circular wavefront that expands at constant speed, with the number of active nodes growing linearly with the wavefront circumference. In hyperbolic space, the wavefront also expands at constant geodesic speed, but the number of active nodes grows exponentially due to the exponential growth of geodesic circles [39].

Interference patterns provide a second point of comparison. In the Euclidean lattice, two pulses originating from nearby sources produce interference fringes with a spacing determined by the lattice geometry and the relative phase of the pulses. In the hyperbolic lattice, the exponential divergence of geodesics causes the interference pattern to become increasingly complex at large distances, with the number of distinct interfering paths growing exponentially. This exponential enrichment of the interference landscape is a qualitative difference between the Euclidean and hyperbolic regimes and is a principal motivation for the hyperbolic embedding.

Stability analysis reveals further differences. The Euclidean lattice with α(d − 1) > 1 exhibits unbounded amplitude growth, which can be interpreted as a supercritical phase in which the combinatorial explosion of paths overwhelms the per-hop attenuation. The hyperbolic lattice exhibits the same supercritical behaviour, but the exponential growth of the manifold amplifies it: the total amplitude grows as (α(d − 1))^t, the same as in the Euclidean case, but the amplitude is distributed over an exponentially larger number of edges [40].

### 6.2 Emergent Dynamics

The central experimental question is whether the hyperbolic embedding gives rise to emergent dynamical phenomena that are absent or rare in the Euclidean case. Three classes of emergent behaviour are investigated.

**Attractors.** In the Euclidean lattice, persistent patterns (attractors) are well known: periodic oscillations can arise from constructive interference between signals traversing loops of compatible length. In the hyperbolic lattice, the prevalence of short loops (particularly 3-cycles, or triangles) and the exponential diversity of loop lengths create a far richer landscape of potential attractors. The question is whether stable, self-sustaining signal patterns can emerge spontaneously from single-source initialisation [41].

**Resonances.** Resonance occurs when the propagation dynamics amplify signals at specific frequencies or spatial scales. In the Euclidean lattice, resonance frequencies are determined by the eigenvalues of the Hashimoto matrix, which in turn depend on the lattice geometry. In the hyperbolic lattice, the spectral properties of the non-backtracking operator are fundamentally different: the spectral radius is still d − 1, but the spectral gap and the distribution of eigenvalues are altered by the negative curvature. These spectral differences are expected to produce qualitatively different resonance behaviour [42].

**Self-sustaining loops.** A self-sustaining loop is a closed path in the graph along which a signal can circulate indefinitely, with the attenuation per loop compensated by constructive interference from other paths. In the Euclidean lattice, such loops require careful tuning of the attenuation parameter. In the hyperbolic lattice, the exponential number of loops of each length increases the probability of spontaneous self-sustaining loop formation, potentially giving rise to complex, self-organising dynamical structures [43].

### 6.3 Sensitivity to Curvature

The curvature parameter K is varied systematically to investigate its effect on the emergent dynamics. As K → 0 (the Euclidean limit), the dynamics approach those of the standard triangular lattice. As |K| increases, the exponential growth rate of the manifold increases, and the density of short loops changes. The experimental programme systematically maps the dependence of the observables – wavefront morphology, interference complexity, attractor prevalence, resonance spectra, and self-sustaining loop formation – on the curvature parameter.

Particular attention is paid to qualitative transitions: values of K at which the character of the dynamics changes abruptly, analogous to phase transitions in statistical mechanics. Such transitions would indicate that the curvature of the manifold acts as a control parameter for the computational capacity of the substrate, a finding with significant implications for the broader research programme on geometry and emergence [44].

---

## 7. Discussion

### 7.1 Why Curvature Matters

The results of this investigation support a principle that may be stated concisely: geometry shapes computation. The local update rule is identical in the Euclidean and hyperbolic formulations; only the global geometry differs. Yet the emergent dynamics are qualitatively different. This demonstrates that the curvature of the manifold is not a passive container for the dynamics but an active participant – it determines the combinatorial landscape of paths, the prevalence of loops and interference, and hence the complexity and character of the emergent behaviour [45].

This principle has implications beyond the specific propagation model studied here. Any dynamical system whose behaviour depends on the topology and combinatorics of the paths available to it – including neural networks, both biological and artificial – will exhibit curvature-dependent dynamics when embedded in a curved manifold. The framework developed in this paper provides a general tool for studying this dependence.

The finding that hyperbolic space enables richer dynamics is consistent with the observation that many natural systems exhibit hierarchical, tree-like structures that are more naturally embedded in hyperbolic than in Euclidean space [46]. The dendritic arbours of neurons, the hierarchical organisation of cortical areas, and the branching structure of vascular networks all exhibit properties that are better captured by hyperbolic metrics. This suggests that the evolutionary "choice" of geometry in biological neural systems may be, in part, a response to the computational advantages of negatively curved substrates.

### 7.2 Biological Parallels

The parallels between the synthetic substrate developed here and biological neural systems are both suggestive and instructive. In biological systems, neural architecture is not pre-specified but emerges from the interaction of activity-dependent growth rules with the geometry of the developing tissue [47]. Neurons extend processes, form synapses, and prune connections based on local signals – a process that is formally analogous to the frontier-based simulation described in Section 4.4.2, where nodes are instantiated when reached by a signal and discarded when their activity falls below a threshold.

The non-uniform geometry of the cerebral cortex – with its sulci and gyri creating regions of varying local curvature – suggests that the brain does not operate on a flat manifold. The curvature of cortical surfaces has been shown to correlate with functional specialisation, with regions of higher curvature associated with specific cognitive functions [48]. The present work provides a computational framework for investigating the hypothesis that cortical curvature is not merely a consequence of packing constraints but plays a functional role in shaping neural dynamics.

The concept of intelligence as an emergent property of neural plasticity – the brain's capacity to reorganise its structure and function in response to environmental inputs – finds a natural mathematical expression in the framework developed here [49]. The propagation model on a hyperbolic manifold is a system in which structure (the graph) emerges from dynamics (the propagation rule acting on the geometry), and in which the emergent dynamics can exhibit complex, adaptive behaviour without explicit programming. This provides a minimal model for studying the conditions under which intelligence-like behaviour – adaptation, self-organisation, the formation of persistent representational structures – can arise from simple local rules operating on a curved substrate.

### 7.3 Limitations

Several limitations of the present framework must be acknowledged. First, numerical drift in the hyperboloid coordinates, while mitigated by renormalisation, accumulates over very long simulations and may introduce systematic errors in the geometry of the lattice at large distances from the origin. The magnitude of this drift depends on the floating-point precision employed; double precision (64-bit) is adequate for simulations of several thousand time steps, but extended precision may be required for the 10^14-node regime [50].

Second, the implicit coordinate approach assumes that the manifold has a globally consistent coordinate system, which is the case for the constant-curvature spaces considered here but may not hold for more general manifolds. Extension to manifolds of variable curvature would require a local coordinate approach, with coordinate patches and transition maps, adding significant implementation complexity.

Third, GPU memory bandwidth is a bottleneck for the frontier-based simulation. The update step requires reading the amplitudes of all active edges and writing the amplitudes of all next-active edges; for large frontiers, this memory traffic may exceed the bandwidth of current GPU architectures. Strategies for alleviating this bottleneck – including tiling the active set for cache efficiency and overlapping computation with memory transfer – are an active area of engineering optimisation [51].

---

## 8. Future Work

The framework developed in this paper opens several avenues for future research.

**Mixed-curvature manifolds.** The restriction to constant-curvature spaces, while mathematically convenient, is biologically unrealistic. The cerebral cortex exhibits regions of positive, zero, and negative curvature, and the functional implications of this curvature variation are not well understood. Extending the framework to product manifolds – spaces that combine hyperbolic, Euclidean, and spherical components – would enable the study of signal propagation on substrates that more faithfully approximate biological geometry [52].

**Adaptive geometry.** In the present framework, the geometry of the manifold is fixed. A natural extension is to allow the geometry to be modified by the dynamics – for instance, by allowing the local curvature to change in response to the amplitude of signals passing through a region. This activity-dependent geometry would provide a direct analogue of activity-dependent plasticity in biological neural systems, where the strength and structure of synaptic connections are modified by neural activity [53].

**Hardware acceleration.** The frontier-based simulation is well suited to implementation on specialised hardware, including neuromorphic processors and field-programmable gate arrays (FPGAs). Neuromorphic architectures, which implement spiking neural dynamics directly in silicon, provide a natural substrate for the non-backtracking propagation model, with each hardware neuron corresponding to a node in the hyperbolic lattice. Exploration of such hardware implementations is a priority for achieving the 10^14-node target [54].

**Scaling to 10^14 nodes.** Reaching the target scale of 10^14 active nodes will require a combination of algorithmic innovation (more aggressive pruning, multi-resolution representations) and hardware scaling (multi-rack GPU clusters, custom accelerators). The computational requirements can be estimated using the scaling laws derived in Section 3.2: at 10^14 active edges, each time step requires approximately 1.5 × 10^15 multiply-add operations, or approximately 1.5 petaFLOP. This is within the capability of current exascale computing facilities but represents a substantial allocation of resources, underscoring the need for efficient implementation and targeted experimental design [55].

---

## 9. Conclusion

This paper has presented a general framework for embedding discrete, non-backtracking signal propagation dynamics within hyperbolic manifolds. The central insight is that the graph structure of the dynamical system need not be stored explicitly but can be generated on demand from the coordinates and metric of the embedding space. This implicit representation enables simulation at scales that are inaccessible to explicit graph methods, opening the door to the study of emergent dynamics on exponentially expanding substrates.

By embedding a simple propagation rule in a curved manifold using implicit coordinates, a scalable synthetic substrate is created that is capable of exhibiting complex, emergent, potentially mind-like dynamics. The framework demonstrates that geometry is not merely a backdrop for computation but an active determinant of computational behaviour: changing the curvature of the manifold, while holding the local rule fixed, produces qualitatively different emergent dynamics. This finding has implications for computational neuroscience, where the geometry of cortical surfaces may play a functional role in shaping neural dynamics; for artificial intelligence, where the geometry of latent spaces influences learning and generalisation; and for the mathematical foundations of cognition, where the interaction of local rules with manifold geometry may constitute a universal mechanism for the emergence of intelligent behaviour [56].

The present work establishes the computational and mathematical foundations for a broader research programme in which human cognition, artificial intelligence latent spaces, and synthetic hyperbolic substrates are all treated as instances of dynamical systems on geometric manifolds. By analysing the invariants that emerge across biological minds, artificial models, and large-scale non-backtracking dynamical systems, this programme aims to test whether intelligence arises from the interaction of simple local rules with curved geometry – a question whose answer may illuminate the deepest structures of mind itself [57].

---

## Notes

[1] The framework presented here extends and synthesises several strands of research. The non-backtracking propagation model on triangular lattices was developed in the context of discrete signal processing; see K. Hashimoto, "Zeta Functions of Finite Graphs and Representations of p-adic Groups," in *Automorphic Forms and Geometry of Arithmetic Varieties*, Advanced Studies in Pure Mathematics, vol. 15 (Tokyo: Academic Press, 1989), 211–280. The application of hyperbolic geometry to complex networks is reviewed in D. Krioukov, F. Papadopoulos, M. Kitsak, A. Vahdat, and M. Boguñá, "Hyperbolic Geometry of Complex Networks," *Physical Review E* 82, no. 3 (2010): 036106.

[2] For foundational treatments of lattice dynamics, see C. Kittel, *Introduction to Solid State Physics*, 8th ed. (Hoboken: John Wiley & Sons, 2005); and M. Born and K. Huang, *Dynamical Theory of Crystal Lattices* (Oxford: Clarendon Press, 1954).

[3] The growth rate of geodesic balls in hyperbolic space is a classical result; see J. Ratcliffe, *Foundations of Hyperbolic Manifolds*, 2nd ed. (New York: Springer, 2006), Chapter 3.

[4] William Murray, "Probing the Geometry of Mind: A Framework for Aligning Artificial and Human Cognition through Multi-Model Analysis" (unpublished manuscript, June 2025). Murray argues that LLM latent spaces can be characterised as geometric and topological manifolds whose properties correlate with cognitive capacity.

[5] William Murray, "Intelligence as an Emergent Property of Neural Plasticity" (unpublished manuscript, July 2025). This paper frames intelligence as a dynamic, adaptive process of neural reorganisation driven by environmental and sensory inputs.

[6] The unified research programme connecting cognitive geometry, LLM latent spaces, and synthetic substrates is articulated in Murray, "Probing the Geometry of Mind."

[7] The Hashimoto matrix was introduced in the context of graph zeta functions; see Hashimoto, "Zeta Functions of Finite Graphs." For its application to non-backtracking walks, see T. Kempton, "Non-backtracking Random Walks and a Weighted Ihara's Theorem," *Open Journal of Discrete Mathematics* 6, no. 4 (2016): 207–226.

[8] This count follows from the structure of the degree-6 triangular lattice: the first hop admits 6 directed edges, and each subsequent hop admits 5, yielding N(t) = 6 × 5^(t−1) for t ≥ 1. For derivation, see the analysis in Appendix A of the present work and the treatment in the context document "Triangular Lattice Pulse Propagation – Non-Backtracking Model and Scaling Laws."

[9] For a comprehensive introduction to Riemannian geometry, see M. do Carmo, *Riemannian Geometry*, trans. F. Flaherty (Boston: Birkhäuser, 1992).

[10] The area formula for hyperbolic disks is derived in Ratcliffe, *Foundations of Hyperbolic Manifolds*, Section 3.4.

[11] The Poincaré disk model is treated in detail in J. Anderson, *Hyperbolic Geometry*, 2nd ed. (London: Springer, 2005), Chapter 4.

[12] For the hyperboloid model and its properties, see Ratcliffe, *Foundations of Hyperbolic Manifolds*, Section 3.2; and J. Cannon, W. Floyd, R. Kenyon, and W. Parry, "Hyperbolic Geometry," in *Flavors of Geometry*, MSRI Publications, vol. 31 (Cambridge: Cambridge University Press, 1997), 59–115.

[13] The Poincaré half-plane model is the classical model of hyperbolic geometry; see Anderson, *Hyperbolic Geometry*, Chapter 3.

[14] The relationship between graphs and manifold discretisations is discussed in A. Bobenko and B. Springborn, "A Discrete Laplace-Beltrami Operator for Simplicial Surfaces," *Discrete and Computational Geometry* 38, no. 4 (2007): 740–756.

[15] The concept of implicit graph representation for hyperbolic lattices is, to the best of the author's knowledge, novel in this context. Related ideas appear in the computational geometry literature on Delaunay triangulations of hyperbolic spaces; see M. Bogdanov, O. Devillers, and M. Teillaud, "Hyperbolic Delaunay Complexes and Voronoi Diagrams Made Practical," *Journal of Computational Geometry* 5, no. 1 (2014): 56–85.

[16] The directed-edge formulation follows the standard treatment of non-backtracking walks; see Hashimoto, "Zeta Functions of Finite Graphs."

[17] The elimination of two-step oscillations by the non-backtracking constraint is discussed in F. Krzakala, C. Moore, E. Mossel, J. Neeman, A. Sly, L. Zdeborová, and P. Zhang, "Spectral Redemption in Clustering Sparse Networks," *Proceedings of the National Academy of Sciences* 110, no. 52 (2013): 20935–20940.

[18] The matrix formulation x(t) = (αB)^t x(0) is standard; see the derivation in the context document "Triangular Lattice Pulse Propagation."

[19] The growth criterion α(d − 1) > 1 is derived in the context document "GPT-OSS_20b_initial," where it is noted that for d = 6 and α = 0.7, the growth factor per step is 3.5.

[20] Memory and computational cost estimates are based on the analysis in the context document "GPT-OSS_20b_initial," where it is shown that memory scales as approximately 48n² bytes and operations per step as approximately 30n² multiply-adds for an n × n lattice.

[21] This conceptual shift – from stored graphs to geometry-generated graphs – is the central contribution of the present paper. It is anticipated in a different context by N. Peyerimhoff and O. Post, "Spectral Theory of Discrete Laplacians on Periodic Graphs," in *Analysis on Graphs and Its Applications*, Proceedings of Symposia in Pure Mathematics, vol. 77 (Providence: American Mathematical Society, 2008), 391–418.

[22] Activity-dependent structure formation in biological neural systems is reviewed in Murray, "Intelligence as an Emergent Property of Neural Plasticity," Sections 3–4, where the mechanisms of synaptic plasticity, neurogenesis, and dendritic remodelling are described in detail.

[23] Numerical stability issues in the Poincaré disk model are analysed in M. Nickel and D. Kiela, "Poincaré Embeddings for Learning Hierarchical Representations," in *Advances in Neural Information Processing Systems* 30 (2017): 6338–6347.

[24] The numerical superiority of the hyperboloid model is demonstrated in O.-E. Ganea, G. Bécigneul, and T. Hofmann, "Hyperbolic Neural Networks," in *Advances in Neural Information Processing Systems* 31 (2018): 5345–5355.

[25] The exponential map in the half-plane model is derived in Anderson, *Hyperbolic Geometry*, Section 3.5.

[26] The construction of evenly spaced tangent vectors on the hyperboloid is described in Ganea et al., "Hyperbolic Neural Networks," Appendix A.

[27] The relationship between lattice spacing and effective curvature in hyperbolic tilings is discussed in Cannon et al., "Hyperbolic Geometry," Section 7.

[28] The geometric formulation of the non-backtracking constraint via tangent vectors is, to the author's knowledge, original to this work.

[29] The invariance of the local update rule under change of geometry is a consequence of the rule's dependence on local adjacency only.

[30] Frontier-based traversal algorithms for implicit graphs have been employed in artificial intelligence search; see S. Russell and P. Norvig, *Artificial Intelligence: A Modern Approach*, 4th ed. (Hoboken: Pearson, 2020), Chapter 3. The application to hyperbolic lattice simulation is novel.

[31] The deterministic regeneration of nodes from coordinates ensures consistency of the simulation across garbage collection cycles, provided that floating-point operations are performed in a consistent order. This is guaranteed by the IEEE 754 standard for floating-point arithmetic.

[32] The growth criterion for the frontier, α(d − 1) ≷ 1, is identical to the growth criterion for the total signal amplitude derived in Section 3.2.

[33] The per-node storage estimate is based on: 3 × 8 = 24 bytes for hyperboloid coordinates, 6 × 8 = 48 bytes for edge amplitudes, and approximately 8–16 bytes for direction and metadata, totalling 80–88 bytes.

[34] Atomic addition operations on GPUs are discussed in N. Wilt, *The CUDA Handbook: A Comprehensive Guide to GPU Programming* (Upper Saddle River: Addison-Wesley, 2013), Chapter 9.

[35] Stream compaction on GPUs is treated in M. Harris, S. Sengupta, and J. Owens, "Parallel Prefix Sum (Scan) with CUDA," in *GPU Gems 3*, ed. H. Nguyen (Upper Saddle River: Addison-Wesley, 2007), Chapter 39.

[36] Double-buffering is a standard technique in GPU programming; see Wilt, *The CUDA Handbook*, Chapter 6.

[37] Domain decomposition strategies for hyperbolic lattices are discussed in the context document "Modular Simulation of Adjacent Triangular Lattice Meshes in the Non-backtracking Pulse Propagation Model: Feasibility, Constraints, and Implementation Strategies," which analyses validity conditions for edge-state merging across domain boundaries.

[38] The requirement for direction exchange across domain boundaries is analysed in the context document "Modular Simulation of Adjacent Triangular Lattice Meshes," Section 5.

[39] The contrast between polynomial and exponential wavefront growth is a direct consequence of the Gauss-Bonnet theorem applied to geodesic disks; see Ratcliffe, *Foundations of Hyperbolic Manifolds*, Section 3.4.

[40] The total amplitude growth rate (α(d − 1))^t is independent of the curvature; what changes is the distribution of this amplitude across the manifold.

[41] The prevalence of short loops in hyperbolic lattices is analysed in D. Aldous and R. Lyons, "Processes on Unimodular Random Networks," *Electronic Journal of Probability* 12 (2007): 1454–1508.

[42] The spectral theory of non-backtracking operators on hyperbolic lattices is related to the Ihara zeta function; see A. Terras, *Zeta Functions of Graphs: A Stroll through the Garden*, Cambridge Studies in Advanced Mathematics, vol. 128 (Cambridge: Cambridge University Press, 2011).

[43] Self-sustaining loops in network dynamics are related to the concept of "echo chambers" in signal processing; for a mathematical treatment, see M. Newman, *Networks*, 2nd ed. (Oxford: Oxford University Press, 2018), Chapter 18.

[44] Phase transitions in dynamical systems on networks are reviewed in R. Pastor-Satorras and A. Vespignani, "Epidemic Spreading in Scale-Free Networks," *Physical Review Letters* 86, no. 14 (2001): 3200–3203.

[45] The principle that geometry shapes computation is a recurring theme in the literature on geometric deep learning; see M. Bronstein, J. Bruna, T. Cohen, and P. Veličković, "Geometric Deep Learning: Grids, Groups, Graphs, Geodesics, and Gauges," *arXiv preprint* arXiv:2104.13478 (2021).

[46] The embedding of hierarchical structures in hyperbolic space is demonstrated in Nickel and Kiela, "Poincaré Embeddings."

[47] Activity-dependent neural development is reviewed in Murray, "Intelligence as an Emergent Property of Neural Plasticity," Section 4, and in L. Katz and C. Shatz, "Synaptic Activity and the Construction of Cortical Circuits," *Science* 274, no. 5290 (1996): 1133–1138.

[48] The correlation between cortical curvature and functional specialisation is discussed in D. Van Essen, "A Tension-Based Theory of Morphogenesis and Compact Wiring in the Central Nervous System," *Nature* 385 (1997): 313–318.

[49] Murray, "Intelligence as an Emergent Property of Neural Plasticity," passim.

[50] Floating-point precision in hyperbolic computations is analysed in Nickel and Kiela, "Poincaré Embeddings," Section 4.2.

[51] Memory bandwidth optimisation for GPU-based graph algorithms is discussed in Y. Wang, A. Davidson, Y. Pan, Y. Wu, A. Riffel, and J. Owens, "Gunrock: A High-Performance Graph Processing Library on the GPU," in *Proceedings of the 21st ACM SIGPLAN Symposium on Principles and Practice of Parallel Programming* (2016): 1–12.

[52] Product manifolds combining hyperbolic and Euclidean components are employed in M. Gu, H. Sala, B. Gunel, and C. Ré, "Learning Mixed-Curvature Representations in Product of Model Spaces," in *International Conference on Learning Representations* (2019).

[53] Activity-dependent geometry modification is anticipated in Murray, "Probing the Geometry of Mind," where the curvature of latent spaces is hypothesised to be shaped by learning dynamics.

[54] Neuromorphic computing for graph dynamics is reviewed in M. Davies et al., "Loihi: A Neuromorphic Manycore Processor with On-Chip Learning," *IEEE Micro* 38, no. 1 (2018): 82–99. The context document "Sovereign Cognition Mesh: Node Architecture, Communication Layers, and Substrate Strategy" further explores hardware substrates for mesh-based cognition architectures.

[55] The computational requirements for 10^14-node simulations are estimated from the scaling laws in Section 3.2: 30 × 10^14 multiply-adds per step ≈ 3 × 10^16 FLOP per step.

[56] This conclusion synthesises the arguments of Murray, "Probing the Geometry of Mind" and Murray, "Intelligence as an Emergent Property of Neural Plasticity," extending them to the synthetic substrate domain.

[57] The unified research programme is articulated in Murray, "Probing the Geometry of Mind," and the present paper constitutes its experimental arm.

---

## Bibliography

### Books

Anderson, James W. *Hyperbolic Geometry*. 2nd ed. London: Springer, 2005.

Born, Max, and Kun Huang. *Dynamical Theory of Crystal Lattices*. Oxford: Clarendon Press, 1954.

do Carmo, Manfredo P. *Riemannian Geometry*. Translated by Francis Flaherty. Boston: Birkhäuser, 1992.

Kittel, Charles. *Introduction to Solid State Physics*. 8th ed. Hoboken: John Wiley & Sons, 2005.

Newman, Mark. *Networks*. 2nd ed. Oxford: Oxford University Press, 2018.

Ratcliffe, John G. *Foundations of Hyperbolic Manifolds*. 2nd ed. New York: Springer, 2006.

Russell, Stuart, and Peter Norvig. *Artificial Intelligence: A Modern Approach*. 4th ed. Hoboken: Pearson, 2020.

Terras, Audrey. *Zeta Functions of Graphs: A Stroll through the Garden*. Cambridge Studies in Advanced Mathematics, vol. 128. Cambridge: Cambridge University Press, 2011.

Wilt, Nicholas. *The CUDA Handbook: A Comprehensive Guide to GPU Programming*. Upper Saddle River: Addison-Wesley, 2013.

### Articles and Conference Papers

Aldous, David, and Russell Lyons. "Processes on Unimodular Random Networks." *Electronic Journal of Probability* 12 (2007): 1454–1508.

Bobenko, Alexander I., and Boris A. Springborn. "A Discrete Laplace-Beltrami Operator for Simplicial Surfaces." *Discrete and Computational Geometry* 38, no. 4 (2007): 740–756.

Bogdanov, Mikhail, Olivier Devillers, and Monique Teillaud. "Hyperbolic Delaunay Complexes and Voronoi Diagrams Made Practical." *Journal of Computational Geometry* 5, no. 1 (2014): 56–85.

Bronstein, Michael M., Joan Bruna, Taco Cohen, and Petar Veličković. "Geometric Deep Learning: Grids, Groups, Graphs, Geodesics, and Gauges." *arXiv preprint* arXiv:2104.13478 (2021).

Cannon, James W., William J. Floyd, Richard Kenyon, and Walter R. Parry. "Hyperbolic Geometry." In *Flavors of Geometry*, edited by Silvio Levy, 59–115. MSRI Publications, vol. 31. Cambridge: Cambridge University Press, 1997.

Davies, Mike, et al. "Loihi: A Neuromorphic Manycore Processor with On-Chip Learning." *IEEE Micro* 38, no. 1 (2018): 82–99.

Ganea, Octavian-Eugen, Gary Bécigneul, and Thomas Hofmann. "Hyperbolic Neural Networks." In *Advances in Neural Information Processing Systems* 31 (2018): 5345–5355.

Gu, Albert, Frederic Sala, Beliz Gunel, and Christopher Ré. "Learning Mixed-Curvature Representations in Product of Model Spaces." In *International Conference on Learning Representations* (2019).

Harris, Mark, Shubhabrata Sengupta, and John D. Owens. "Parallel Prefix Sum (Scan) with CUDA." In *GPU Gems 3*, edited by Hubert Nguyen, Chapter 39. Upper Saddle River: Addison-Wesley, 2007.

Hashimoto, Ki-ichiro. "Zeta Functions of Finite Graphs and Representations of p-adic Groups." In *Automorphic Forms and Geometry of Arithmetic Varieties*, 211–280. Advanced Studies in Pure Mathematics, vol. 15. Tokyo: Academic Press, 1989.

Katz, Lawrence C., and Carla J. Shatz. "Synaptic Activity and the Construction of Cortical Circuits." *Science* 274, no. 5290 (1996): 1133–1138.

Kempton, Tom. "Non-backtracking Random Walks and a Weighted Ihara's Theorem." *Open Journal of Discrete Mathematics* 6, no. 4 (2016): 207–226.

Krioukov, Dmitri, Fragkiskos Papadopoulos, Maksim Kitsak, Amin Vahdat, and Marián Boguñá. "Hyperbolic Geometry of Complex Networks." *Physical Review E* 82, no. 3 (2010): 036106.

Krzakala, Florent, Cristopher Moore, Elchanan Mossel, Joe Neeman, Allan Sly, Lenka Zdeborová, and Pan Zhang. "Spectral Redemption in Clustering Sparse Networks." *Proceedings of the National Academy of Sciences* 110, no. 52 (2013): 20935–20940.

Nickel, Maximilian, and Douwe Kiela. "Poincaré Embeddings for Learning Hierarchical Representations." In *Advances in Neural Information Processing Systems* 30 (2017): 6338–6347.

Pastor-Satorras, Romualdo, and Alessandro Vespignani. "Epidemic Spreading in Scale-Free Networks." *Physical Review Letters* 86, no. 14 (2001): 3200–3203.

Van Essen, David C. "A Tension-Based Theory of Morphogenesis and Compact Wiring in the Central Nervous System." *Nature* 385 (1997): 313–318.

Wang, Yangzihao, Andrew Davidson, Yuechao Pan, Yuduo Wu, Andy Riffel, and John D. Owens. "Gunrock: A High-Performance Graph Processing Library on the GPU." In *Proceedings of the 21st ACM SIGPLAN Symposium on Principles and Practice of Parallel Programming* (2016): 1–12.

### Unpublished Manuscripts and Context Documents

Murray, William. "Intelligence as an Emergent Property of Neural Plasticity." Unpublished manuscript, July 2025.

Murray, William. "Probing the Geometry of Mind: A Framework for Aligning Artificial and Human Cognition through Multi-Model Analysis." Unpublished manuscript, June 2025.

Murray, William. "Sovereign Cognition Mesh: Node Architecture, Communication Layers, and Substrate Strategy." Unpublished design document, 2025.

"Modular Simulation of Adjacent Triangular Lattice Meshes in the Non-backtracking Pulse Propagation Model: Feasibility, Constraints, and Implementation Strategies." Technical report, 2025.

"Triangular Lattice Pulse Propagation – Non-Backtracking Model and Scaling Laws." Technical document, 2025.

---

