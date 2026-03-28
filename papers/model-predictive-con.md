# Algebraic Handedness: Why Some Mathematical Structures Can Never Be Mirrored

*Original: Chirality and Racemization on Isotopy Classes of Quasigroups*

**arXiv**: [https://arxiv.org/abs/2603.2243](https://arxiv.org/abs/2603.2243)  
**Read on PaperGlide**: [https://paperglide.net/papers/m2m7cvex](https://paperglide.net/papers/m2m7cvex)

---

**TL;DR:** Just like your left and right hands are mirror images that can't be perfectly stacked, some mathematical structures called quasigroups have a "handedness" that prevents them from ever matching their own reflection. Researchers discovered that while most of these structures naturally flip and mix with their mirror versions over time, certain rare examples are mathematically locked into one shape and can never swap. This discovery matters because it uses pure math to explain the same kind of stability seen in "left-handed" or "right-handed" molecules in chemistry, proving that handedness is a fundamental property of logic itself.

---

## Paper Primer

### Order‑7 Quasigroup Defies Mirror Symmetry 
A non‑commutative quasigroup on seven elements **has no mirror‑isotopism**, so its isotopy class is *structurally chiral* and the induced Markov rate **k([Q])** is exactly **zero**.

### Why You Should Care About Algebraic Handedness 
If you’ve ever wondered why some molecules stay “left‑handed” while others flip to a racemic mix, you’re looking at the same abstract tension—only now it lives in the world of **quasigroups**. Your algebraic toolbox may be full of Latin squares, but without a notion of *handedness* you’ve missed a whole dynamical layer that mirrors chemistry’s enantiomer race.

### What the Literature Gets Wrong 
- **No gauge‑theoretic view of quasigroup re‑coordination.** Existing isotopy theory treats relabelings as ad‑hoc, not as a symmetry that can be factored out of dynamics. 
- **Racemization never modeled.** Prior work on quasigroups focuses on structural classification; none describe a *time‑evolution* that mixes a structure with its mirror. 
- **Chiral stability lacks a structural test.** Researchers have no algebraic criterion to decide whether a quasigroup can ever flip to its opposite. 
- **Commutativity is assumed harmless.** Many papers overlook that a commutative operation automatically kills any chance of structural chirality. 
- **No concrete, minimal counterexample.** The smallest known chiral Latin squares were large; no order‑7 example existed to demonstrate that chirality isn’t pathological.

### What the Authors Actually Did (And Why It’s Clever) 
They built a **two‑state continuous‑time Markov model** whose state space is the set of all quasigroup tables on a fixed finite set **X**. The only allowed jump is from a quasigroup **Q** to its *mirror* **Q-mirror** (the opposite operation). By insisting that transition rates respect the **isotopy gauge symmetry**, the whole high‑dimensional process collapses to a **single scalar rate** **k([Q])** for each isotopy class **[Q]**. 

Then they asked: *When is **k([Q])** forced to be zero?* The answer is strikingly simple—**exactly when**Q**has no mirror‑isotopism** (no element of **G** sends **Q** to **Q-mirror**). They proved the equivalence, introduced the term **structurally chiral isotopy class**, and supplied a concrete order‑7 Latin square that satisfies the condition. 

That’s it. That’s the core idea.

### How the Theory Works, Step by Step 

| Step | Action | Novelty |
|------|--------|---------|
| **1. Define the gauge group** | **G = Sym(X)³** acts on a quasigroup **Q** by **(α, β, γ) · Q** with **x * y = γ(α⁻¹(x) · β⁻¹(y))**. | — |
| **2. Introduce the mirror** | **Q-mirror = (X, diamond)** where **x diamond y = y · x** (the opposite operation). | — |
| **3. Impose gauge‑invariant rates** | Choose a **conjugation‑invariant weight** **weight function w: G → [0, ∞)** and set 

r(Q\to R)=\sum_{g\in\operatorname{Mir}(Q)} w(g)\,\mathbf 1_{\{R=g\cdot Q^{\#}\}}.
| **[NEW]** – rates depend only on *mirror‑isotopisms* of **Q**. |
| **4. Collapse to isotopy classes** | The total outbound rate from **[Q]** is 

k([Q])=\sum_{g\in\operatorname{Mir}(Q)} w(g).
The generator on the quotient is **L_bar f_bar([Q]) = k([Q]) [f_bar([Q-mirror]) - f_bar([Q])]**. | **[NEW]** – a *two‑state* CTMC for each pair **the set containing the isomorphism class [Q] and its mirror class [Q-mirror]**. |
| **5. Characterize chiral stability** | **rate k([Q]) = 0 if and only if the set of mirror autotopisms Mir(Q) is empty, which implies** no isotopism **the transition from Q to Q-mirror** exists. The class is then called **structurally chiral**. | **[NEW]** – algebraic obstruction replaces dynamical assumptions. |
| **6. Verify with a concrete Latin square** | The order‑7 table (see below) has **the set of mirror autotopisms Mir(Q) is empty**, so **k([Q])=0** and the class never racemizes. | — |

**Key technical ingredients** 
- **Isotopy‑invariant observables** **f** satisfy **f(g · Q) = f(Q)**. 
- **Mirror‑only transitions** guarantee the process never jumps to a non‑mirror isotopy class. 
- **Symmetric racemization** ( **k([Q]) = k([Q-mirror])**) yields the racemic equilibrium **Probability that the state at time t is [Q] approaches 1/2**. 

### Numbers That Matter 

| Quasigroup type | Mirror‑isotopism set **the set of mirror autotopisms Mir(Q)** | Weight sum **k([Q])** (choose **weight function w is identically 1**) | Racemization outcome |
|-----------------|---------------------------------------------|----------------------------------------|----------------------|
| **Order‑7 chiral example** (non‑commutative) | **empty set** | **0** | **Never** reaches mirror class (structurally stable). |
| **Any commutative quasigroup** | Non‑empty (contains identity) | **>0** (at least **weight of the identity element w(id)**) | Converges to racemic equilibrium (racemizes). |
| **Generic non‑commutative isotopic pair** | At least one element (e.g., a non‑trivial autotopism) | **>0** (sum of **w** over those elements) | Racemizes with rate proportional to **k([Q])**. |

*Baseline*: In the absence of the structural criterion, one would assume every quasigroup can flip, i.e. **k([Q])>0** for all **[Q]**. The order‑7 example **breaks** that baseline with a **100 % reduction** in mirror‑transition rate.

### The Twist No One Saw Coming 
You’d expect *commutativity* to be the only obstacle to chirality, but the paper shows the opposite: **every commutative quasigroup is *forced* to racemize**, because its mirror is identical and the identity isotopism already provides a non‑zero rate. Conversely, a tiny **order‑7 non‑commutative** Latin square can be *completely immune* to mirror moves—its chirality is baked into the algebraic structure, not into any external energy barrier. This flips the chemical intuition that “larger, more complex molecules are needed for stable handedness” on its head.

### When This Won’t Work 

- **Mirror choice matters.** The whole theory hinges on picking the *opposite* parastrophe as the mirror; a different parastrophe could resurrect mirror‑isotopisms. 
- **Weight function is arbitrary.** If you set **w(g)=0** for all **g**, every class becomes chiral; if you give huge weight to the identity, even mildly asymmetric quasigroups racemize instantly. The framework does not prescribe a canonical **w**. 
- **Finite‑set restriction.** All proofs assume **the size of set X is finite**; extending to infinite loops or topological quasigroups is non‑trivial. 
- **Only mirror‑only jumps.** Realistic dynamics might allow other structural mutations (e.g., changing a single entry); the model deliberately excludes them. 
- **No algorithmic shortcut.** Detecting **the set of Mirror parastrophes of Q is empty** currently relies on combinatorial invariants (intercalate counts) or exhaustive search, which may be infeasible for large **|X|**.

### Bottom Line 
By treating **isotopy as a gauge symmetry** and **mirror parastrophy as handedness reversal**, the authors turned a bewildering space of quasigroup tables into a **clean two‑state Markov process** whose fate is decided by a single algebraic check: *does a mirror‑isotopism exist?* The order‑7 example proves that **structural chirality is real, not pathological**, and that **racemization is the default unless an explicit algebraic obstruction blocks it**. This bridges abstract algebra, dynamical systems, and the chemistry of enantiomers, opening a path to classify chiral quasigroup families and to explore “handedness” in any setting where isotopy‑type gauge freedoms appear.

---

### Hands, Molecules, and the Mystery of Handedness 

Our everyday world already knows “handedness.” A left hand cannot be superimposed on a right hand, just as a molecule’s two mirror images—*enantiomers*—cannot be rotated to match each other. This **chirality** is more than a visual quirk; it can dictate dramatically different biological outcomes.

Consider the drug **thalidomide**. It exists as two enantiomers, labeled **(R)** and **(S)**. Early clinical use distributed a *racemic* mixture (both forms together). Researchers later realized that the **(R)** form largely provides the desired sedative effect, while the **(S)** form is responsible for severe birth defects. 

In the human body, however, the two forms interconvert rapidly—a process called **racemization**. If a pure **(R)** dose is injected, within hours the bloodstream contains roughly equal amounts of **(R)** and **(S)** because the molecule flips its handedness in physiological conditions. Thus, the “handedness label” can be **lost dynamically**, motivating the search for an abstract framework where handedness may be preserved or erased.

--- 

### Isotopy: Relabeling the Coordinates of a Quasigroup 

In algebra we work with a **quasigroup**: a binary operation “ ***** ” defined on a fixed set **X**. The operation’s multiplication table lists outcomes **x * y = z**. Crucially, we can **relabel** the three roles—left input, right input, and output—independently without changing the underlying computational structure. 

The collection of all such independent relabelings forms the group 

 **G is defined as the product of three symmetric groups: Sym(X) × Sym(X) × Sym(X)** 

where: 

- **Sym(X) (the symmetric group of set X)** – the set of all permutations (bijections) of the underlying set **X**; 
- the first factor permutes left inputs, the second permutes right inputs, and the third permutes outputs; 
- the Cartesian product **Cartesian product (cross product)** bundles the three permutations into a single transformation. 

An element **The triplet (α, β, γ) is an element of group G** acts on a product entry **x * y = z** by **relabeling** it to 

 **α(x) *' β(y) = γ(z)**. 

All tables related by such a transformation are **isotopic**. Because the transformation merely changes “coordinates,” isotopy is interpreted as a **gauge symmetry**: different tables represent the same abstract quasigroup, just expressed in different labels. This symmetry does **not** alter algebraic properties such as where associativity fails; it only shuffles the presentation.

--- 

### Mirror Parastrophy: Flipping Handedness 

Just as a left hand becomes a right hand when mirrored, a quasigroup has a **mirror (or opposite) operation**, denoted **Q#**. Formally, if **x * y = z** holds in **Q**, then in the mirrored quasigroup **Q#** we may have 

 **y *# x = z**. 

Thus the roles of the two inputs are swapped while the output stays the same, capturing a **handedness reversal**. In the chemical analogy, moving from a molecule to its mirror image corresponds to passing from **Q** to **Q#**. This operation will later serve as the sole “physical” transition in our dynamical model.

--- 

### A Gauge‑Invariant Two‑State Markov Model 

To study how handedness can change over time, imagine a **continuous‑time Markov process** whose state space consists of *all* quasigroup structures on **X**. The only allowed transition is a **mirror move**: 


Q \;\leftrightarrow\; Q^{\#}.


The generator of this process has off‑diagonal entries 

 **k([Q])** 

where: 

- **k([Q])** – the transition rate associated with the **isotopy class** **[Q]** (all tables isotopic to one another); 
- the rate depends **only** on the class, not on the particular representative, embodying a **gauge‑invariance hypothesis**; 
- all other entries are zero because no other moves are permitted. 

If **k([Q]) = 0**, the system never flips handedness for that class, representing **dynamical chiral stability**—the algebraic analogue of a chemically stable enantiomer.

--- 

### Descending to Isotopy Classes: Racemic Equilibria and Chiral Stability 

Because the transition rate respects isotopy, the dynamics **projects** onto the quotient space of isotopy classes, **Q(X)/G**. On each unordered pair **the set containing the isotopy class [Q] and its mirror image [Q#]** the induced process is a genuine two‑state Markov chain with rate **k([Q])**. 

- When **k([Q]) > 0**, the chain eventually reaches a **racemic equilibrium**, assigning equal stationary probability to **[Q]** and **the mirror image isotopy class [Q#]**. 
- When **k([Q]) = 0**, the pair never interconverts; the system remains locked in its original handedness, mirroring a chemically protected enantiomer. 

A concrete **order‑7** quasigroup class exhibits **k([Q]) > 0**, demonstrating that structurally chiral classes do exist. Conversely, classes lacking any admissible mirror‑isotopism satisfy **k([Q]) = 0**, guaranteeing chiral stability. 

**Outlook:** By engineering the algebraic environment so that **k([Q])** is forced to vanish—or at least become negligibly small—we obtain the abstract counterpart of preserving a single enantiomer in a biochemical setting. This bridge between molecular chirality and quasigroup isotopy opens a pathway for transferring intuition across chemistry and algebra.

### From Intuition to Formalism: A Continuous‑Time Markov View of Quasigroup Racemization 

Imagine a quasigroup as a “molecule” that can flip between a left‑handed form and its mirror image. The stochastic dynamics we study describe random jumps from one quasigroup to another, but only when the jump corresponds to taking a mirror and then possibly relabeling the elements. By insisting that the dynamics respect all possible relabelings (the **gauge freedom**) we can collapse the huge state space onto a tiny quotient that only remembers *which isotopy class* the system occupies.

---

### 1. The gauge group and its action 

The relabeling symmetry is captured by the Cartesian product of three copies of the symmetric group on the underlying set **X**:

 **G is defined as the product of three symmetric groups: Sym(X) × Sym(X) × Sym(X)** 

**Where:** 
- **Sym(X)** – all permutations of the finite set **X**. 
- The three factors act on the *left input*, *right input*, and *output* of the quasigroup operation, respectively. 

An element **g = (α, β, γ) in G** transforms a quasigroup **Q = (X, ·)** into a new quasigroup **g · Q = (X, *)** via 

 **x * y:= γ(α⁻¹(x) · β⁻¹(y))** 

**Where:** 
- **α⁻¹, β⁻¹** – undo the input relabelings before applying the original product. 
- **γ** – relabels the resulting output. 
- The operation **·** is the original quasigroup multiplication. 

**Q. What does “gauge freedom’’ mean in this algebraic setting?** 
A. It means we may arbitrarily permute the names of the elements of **X** on the left input, right input, and output without changing the intrinsic algebraic structure; such permutations are exactly the elements of **G**.

This formal action refines the earlier notion of isotopy as a gauge symmetry: two quasigroups related by some **g in G** are considered the same *up to relabeling*.

---

### 2. Mirror (parastrophe) and its compatibility with isotopy 

The mirror of a quasigroup swaps the order of multiplication:

 **Q#:= (X, ⋄), where x ⋄ y:= y · x** 

**Where:** 
- **Q#** – the *parastrophe* (mirror) of **Q**. 
- **⋄** – the flipped product, exchanging left and right arguments. 

The mirror operation is involutive: applying it twice returns the original quasigroup, **(Q#)# = Q**.

When we apply an isotopy **g = (α, β, γ)** and then mirror, the result is the same as first mirroring and then applying the *swapped* isotopy 

 **g#:= (β, α, γ)** 

The compatibility is expressed by the lemma 

 **(g · Q)# = g# · (Q#)** 

**Where:** 
- **g · Q** – isotopy action defined above. 
- **()^\#** – mirror operation. 
- **g#** – isotopy with the two input permutations exchanged. 

**Q. How does the mirror operation interact with isotopy?** 
A. Applying an isotopy to a quasigroup and then mirroring is the same as first mirroring and then applying the isotopy with the two input permutations swapped (the element **g#**). 

Because of this compatibility, the mirror map descends to the quotient space of isotopy classes **Q(X)/G**.

---

### 3. Observables that respect the gauge symmetry 

A function **function f from the set of quasigroups Q(X) to the real numbers** is **isotopy‑invariant** if 

 **f(g · Q) = f(Q) for all g in G** 

**Where:** 
- **f** – observable defined on the set of all quasigroups. 
- **g · Q** – isotopy action. 
- The equality must hold for every relabeling **g**. 

Such observables depend only on the isotopy class, not on the particular representative.

---

### 4. The underlying continuous‑time Markov process 

We consider a Markov process **(Q_t) for time t ≥ 0** on the full state space **Q(X)** with transition rates **rate r(Q → R) ≥ 0**. Its infinitesimal generator acts on functions **f** as 

 **(Lf)(Q) = Σ(for all R in Q(X)) of r(Q → R) · (f(R) - f(Q))** 

**Where:** 
- **the set of all quasigroups Q(X)** – the set of all quasigroups on **X**. 
- **rate r(Q → R)** – instantaneous rate of jumping from **Q** to **R**. 
- **f(R)-f(Q)** – change in the observable caused by the jump. 

**Q. What does isotopy invariance of an observable imply for its evolution under**L**?** 
A. If **f** does not change under any isotopy, then **L** will map **f** to another isotopy‑invariant function, because the rates will respect the same symmetry (see the next section).

---

### 5. Structural assumptions that enforce a two‑state reduction 

To obtain a dynamics that only flips between a class and its mirror, we impose three conditions on the rates:

1. **Gauge invariance of rates** 

 **rate(g·Q → g·R) = rate(Q → R) for all g in G** 

 **Where:** 
 - The left‑hand side is the rate after relabeling both source and target. 
 - Equality guarantees that the dynamics are blind to the choice of coordinates.

2. **Mirror‑only transitions** 

 **rate(Q → R) = 0 unless R is in the set G·Q#** 

 **Where:** 
 - **the set G·Q#** – the orbit of the mirror under all isotopies. 
 - The rule forces every jump to land in the *mirror orbit*.

3. **Class‑dependent total rate** 

 **k([Q]) = Σ(R in G·Q#) of rate(Q → R)** 

 **Where:** 
 - **[Q]** – the isotopy class of **Q**. 
 - **k([Q])** is the *effective* rate at which the process leaves class **[Q]** toward its mirror. 
 - By assumption, **k([Q])** depends only on the class, not on the specific representative.

**Q. Why restrict transitions to the mirror class?** 
A. Because we want a dynamics that can be described solely by whether a quasigroup is in a given isotopy class or its mirror, yielding a two‑state Markov chain after quotienting.

These three assumptions guarantee that the original Markov process projects cleanly onto the quotient **Q(X)/G**.

---

### 6. Projection onto the isotopy quotient 

Because the rates are gauge‑invariant and only connect a class to the orbit of its mirror, the Markov semigroup preserves isotopy‑invariant functions. Consequently the process descends to the quotient, which now has only two states: the class **[Q]** and its mirror **[Q_mirror]**.

The induced generator **L_bar** acting on functions **f_bar: Q(X)/G → ℝ** is 

 **(L_bar f_bar)([Q]) = k([Q]) · (f_bar([Q_mirror]) - f_bar([Q]))** 

**Where:** 
- **f_bar** – observable defined on isotopy classes. 
- **k([Q])** – total mirror‑transition rate defined above. 
- The expression measures the net flow from **[Q]** to its mirror.

**Q. What does the induced generator tell us about the system’s behaviour?** 
A. It shows that, after factoring out gauge freedom, the dynamics reduce to a simple birth‑death‑like process that only flips between a class and its mirror at rate **k([Q])**.

Thus any isotopy‑invariant observable evolves according to this two‑state rule.

---

### 7. Symmetric racemization and the equilibrium distribution 

If the effective rates are symmetric,

 **k([Q]) = k([Q#])** 

the induced two‑state chain has equal forward and backward rates. Its unique stationary distribution is uniform:

 **limit as t approaches infinity of Pr([Q_t] = [Q]) = limit as t approaches infinity of Pr([Q_t] = [Q#]) = 1/2** 

**Where:** 
- **Pr([Q_t] = [Q])** – probability that at time **t** the process is in class **[Q]**. 
- The limit as **t approaches infinity** gives the long‑run (stationary) probabilities. 

**Q. What does “racemic equilibrium’’ mean for isotopy classes?** 
A. It means that, after a long time, the probability of finding the system in a given class is exactly the same as the probability of finding it in the mirror class; the two states are statistically indistinguishable. 

This mirrors the chemical notion of a racemic mixture, where left‑ and right‑handed enantiomers are present in equal amounts.

---

### 8. When racemization is blocked: structural obstructions 

A special situation occurs when the total mirror‑transition rate vanishes for a class:

 **k([Q]) = 0.** 

In this case the induced generator **L-bar** is identically zero on **[Q]**; the process never leaves that isotopy class. Algebraically, this reflects a **dynamically stable chiral class**: the mirror cannot be reached by any allowed transition.

The obstruction stems from the internal symmetry group of the quasigroup. If no autotopism, paratopism, or other canonical self‑equivalence maps **Q** to its mirror, then every gauge‑invariant rate **rate of transition from Q to R** with **R is an element of the orbit of Q-sharp under group G** must be zero, forcing **k([Q])** to vanish.

**Q. How can algebraic structure prevent mirror transitions?** 
A. If no internal symmetry (e.g., autotopism) maps **Q** to its mirror, then the gauge‑invariant rate **rate of transition from Q to R** must be zero for all **R** in the mirror orbit, making **k([Q])** vanish.

Identifying such obstructions is a separate structural problem, inviting deeper study of the quasigroup’s internal symmetry groups.

### Autotopism Group Atp(Q): Intrinsic Symmetries of a Quasigroup 

**Key idea:** The autotopism group collects exactly those triples of permutations that leave the multiplication table of a quasigroup unchanged. 

Let  **X** be a finite set and  **Q is defined by set X and operation dot** a quasigroup. 
The **isotopy group** is the Cartesian product of three copies of the symmetric group on  **X**: 

 **G is the direct product of the symmetric group of X with itself three times** 

Where: 
- **the symmetric group of X** — all bijections of  **X** (the full permutation group) 
- The three factors act respectively on the left argument, the right argument, and the product of the quasigroup 

An **autotopism** is a triple **the triple (α, β, γ) is an element of G** satisfying 

 **the autotopism group of Q is the set of triples (α, β, γ) in G such that α(x) dot β(y) = γ(x dot y) for all x, y in X** 

Where: 
- **α, β, γ** — permutations from the three copies of **the symmetric group of X** 
- **α(x) dot β(y)** — apply **α** to the left input, **β** to the right input, then multiply in  **Q** 
- **γ(x dot y)** — apply **γ** to the original product 

In words, each element of  **the autotopism group of Q** is an **internal symmetry**: after permuting the two inputs and finally the output, the original multiplication table is recovered. 

**Concrete example.** 
Take **X is the set {0, 1}** with the quasigroup operation given by 


\begin{array}{c|cc}
\cdot & 0 & 1\\\hline
0 & 0 & 1\\
1 & 1 & 0
\end{array}


The permutation **σ** that swaps 0 and 1 satisfies **the triple (σ, σ, σ) is an element of the autotopism group of Q**; swapping both arguments and the product leaves the table invariant. 

*Why it matters:* Later we will draw admissible **mirror transitions** from this intrinsic symmetry set, so **the autotopism group of Q** is the algebraic reservoir for those moves.

--- 

### Mirror Quasigroup  **Q-sharp** and the Set of Mirror‑Isotopisms  **the mirror group of Q** 

**Key idea:** A mirror‑isotopism is a symmetry that maps a quasigroup to its *reflected* version, i.e. the opposite quasigroup obtained by swapping the order of multiplication. 

Define the **mirror (opposite) quasigroup** **Q-sharp is defined by set X and operation diamond** by 

 **x diamond y is defined as y dot x** 

Where: 
- **x diamond y** — the new product obtained by reversing the original order 
- **y dot x** — the original product with arguments swapped 

Thus **Q-sharp** is the algebraic analogue of looking at  **Q** in a mirror. 

A **mirror‑isotopism** is a triple **the triple (α, β, γ) is an element of G** that is an isotopism from  **Q** to  **Q-sharp**: 

 **α(x) diamond β(y) = γ(x dot y) for all x, y in X** 

Where: 
- **α, β, γ** — permutations from the three copies of **the symmetric group of X** 
- **α(x) diamond β(y)** — apply **α** to the left input, **β** to the right, then use the mirror product **diamond** 
- **γ(x dot y)** — apply **γ** to the original product 

Using the definition of **diamond**, this condition is equivalent to 

 **β(y) dot α(x) = γ(x dot y) for all x, y in X** 

Where the order of the first two permutations is reversed compared with an autotopism. 

Collect all such triples in the **mirror‑isotopism set** 

 **the mirror group of Q is the set of isotopisms from Q to Q-sharp, defined as triples (α, β, γ) in G such that β(y) dot α(x) = γ(x dot y)** 

Where: 
- **the set of isotopisms from Q to Q-sharp** — the set of isotopisms from  **Q** to its mirror 
- The condition inside the braces is exactly the mirror‑isotopism equation 

**Example.** 
If a quasigroup’s multiplication table is symmetric under transposition (i.e. the Latin square equals its transpose), then the identity triple **the identity triple (id, id, id)** belongs to  **the mirror group of Q**. 

**Empty case.** Many quasigroups admit no such triple; then **the mirror group of Q is empty**. This emptiness will later be linked to a vanishing mirror transition rate.

--- 

### Symmetry‑Generated Mirror Transition Mechanism 

**Key idea:** Mirror jumps are allowed **only** when they are realized by a mirror‑isotopism, and the jump intensity is dictated by a weight attached to that symmetry. 

Fix a non‑negative weight function 

 **weight function w from G to the non-negative real numbers** 

that is a **class function** (conjugacy‑invariant): 

 **w(h g h-inverse) = w(g) for all g, h in G** 

Where: 
- **w(g)** — the intrinsic “strength” of the symmetry **g** 
- **h g h-inverse** — conjugation of **g** by any **h is an element of G** (a change of basis) 

For each quasigroup  **Q** define the **mirror transition rate** to a target quasigroup  **R** as 

 **the rate r(Q to R) is the sum over all g in the mirror group of Q of w(g) times the indicator function that R equals g acting on Q-sharp** 

Where: 
- **sum over all g in the mirror group of Q** — sum over all mirror‑isotopisms of  **Q** 
- **w(g)** — weight of the particular symmetry **g** 
- **indicator function that R equals g acting on Q-sharp** — indicator that **R** is exactly the image of the mirror **Q-sharp** under **g** 

In plain language: from  **Q** we may jump **only** to those isotopes of its mirror that are reachable via a mirror‑isotopism, and the jump intensity is the weight attached to that symmetry. 

Because **w** is a class function, the construction is **gauge invariant**: conjugating the whole configuration by any **h is an element of G** leaves all rates unchanged. Moreover, the definition enforces the **mirror‑only property**—no transition can land outside the orbit **the orbit of Q-sharp under G**. 

*Illustrative scenario.* 
If **the mirror group of Q contains only the element g** consists of a single element and **w(g) = λ**, then the only admissible mirror move is **Q transitions to g acting on Q-sharp**, occurring at rate **λ**.

--- 

### Theorem 3.6: Structural Vanishing of the Mirror Rate 

**Key statement.** Under the symmetry‑generated mechanism, the following three conditions are equivalent:

1. **k([Q]) = 0** (the induced mirror‑transition rate on the isotopy class of  **Q** vanishes); 
2. **the mirror group of Q is empty** (no mirror‑isotopism exists); 
3. **the equivalence class of Q is not equal to the equivalence class of Q-sharp** and the mirror class is dynamically inaccessible by intrinsic symmetry. 

*Derivation.* 
The induced mirror‑transition rate on isotopy classes is 

 **the total rate k([Q]) is the sum over R in the orbit of Q-sharp of r(Q to R), which equals the sum over g in the mirror group of Q of w(g)** 

Where: 
- The first equality sums the pointwise rates over all isotopes of the mirror; 
- The second equality uses the definition of **r** and the fact that each **g is an element of the mirror group of Q** produces exactly one such isotope **g acting on Q-sharp**. 

Since every weight **w(g)** is non‑negative, the sum can be zero **iff** the index set **the mirror group of Q** is empty. Hence (1) ⇔ (2). Condition (3) follows because an empty **the mirror group of Q** means no symmetry can map **Q** to its mirror, so the two isotopy classes remain distinct and the mirror class is never reached.

**Analogy.** 
Think of **Q** as a chiral molecule (e.g., a left‑handed screw). If the molecule possesses no internal symmetry that can re‑orient it into its mirror image, then there is no admissible “flip” – mathematically, **the mirror group of Q is empty**. Consequently the stochastic process never jumps to the mirror class, and the mirror rate **k([Q])** is zero.

**Definition 3.7 (Structurally chiral isotopy class).** 
An isotopy class **[Q]** is called **structurally chiral** (with respect to the chosen mirror) precisely when **the mirror group of Q is empty**.

**Remark 3.8.** 
The same reasoning holds for any fixed parastrophe **Q-pi**, not just the opposite **Q-sharp**; the corresponding set of **π** ‑isotopisms replaces **the mirror group of Q** in the equivalences.

### Purpose and Overview 

The goal of this final analytical segment is to **formalize the effective mirror transition rate** on isotopy classes and to prove that **chiral quasigroups actually exist**. We will see how the algebraic set of mirror‑isotopisms  **the mirror group of Q** completely determines the class‑dependent rate **k([Q])**, and we will illustrate the theory with a concrete **order‑7** quasigroup whose structure guarantees chirality.

--- 

### Lemma 3.13 – Gauge Invariance of Symmetry‑Generated Transitions 

**Statement.** Under Assumption 3.11, for any symmetry element **h is an element of G** and any quasigroups **Q,R**,
r(h \cdot Q \to h \cdot R) = r(Q \to R). 

**Why it matters.** This lemma shows that the transition probability **r** does not depend on a global relabeling of the underlying set; the dynamics respect the group action **G**.

**Key algebraic facts used in the proof.**

1. **Conjugation of mirror‑isotopisms** 
 \operatorname{Mir}(h \cdot Q) = h \operatorname{Mir}(Q) h^{-1}. 
 *Where:* 
 - **the mirror group of h acting on Q** – mirror‑isotopisms of the transformed quasigroup. 
 - **h** – an element of the symmetry group **G**. 
 - **the mirror group of Q** – mirror‑isotopisms of the original quasigroup. 
 - **h-inverse** – inverse of **h** in **G**. 

 *Explanation:* Conjugating every mirror‑isotopism by **h** yields exactly the mirror‑isotopisms of the **h** ‑shifted quasigroup.

2. **Conjugation invariance of the weight function** 
 w(hgh^{-1}) = w(g). 
 *Where:* 
 - **weight function w** – the non‑negative weight assigned to a group element. 
 - **g** – an arbitrary element of **G**. 
 - **h g h-inverse** – the conjugate of **g** by **h**. 

 *Explanation:* The weight depends only on the conjugacy class, so moving **g** around by **h** leaves its weight unchanged.

3. **Compatibility of the action with conjugation** 
 (hgh^{-1}) \cdot (h \cdot Q^{\#}) = h \cdot (g \cdot Q^{\#}). 
 *Where:* 
 - **Q-sharp** – the transpose (mirror) of **Q**. 
 - **dot** – the group action on quasigroups. 

 *Explanation:* Applying a conjugated symmetry to the mirrored quasigroup is the same as first acting by **g** on **Q-sharp** and then applying **h**.

**Putting it together.** The transition rate is defined by 
r(Q \to R)=\sum_{g\in\operatorname{Mir}(Q)} w(g)\,\mathbf 1_{\{R=g\cdot Q^{\#}\}}. 
Substituting the three identities above replaces every occurrence of **Q,R,g** with **h acting on Q, h acting on R, and h g h-inverse**, leaving the sum unchanged. Hence **the rate from h·Q to h·R equals the rate from Q to R**.

--- 

### Proposition 3.14 – Explicit Formula for the Mirror Rate 

**Statement.** Under Assumption 3.11, the effective mirror rate on an isotopy class **[Q]** is 
k([Q]) = \sum_{g \in \mathrm{Mir}(Q)} w(g). 

**Derivation.** 

1. By definition, the class‑dependent rate aggregates all transitions from **Q** to any mirrored image reachable under the group action: 
 k([Q]) = \sum_{R \in G \cdot Q^{\#}} r(Q \to R). 
 *Where:* 
 - **the orbit of Q-sharp under G** – the orbit of the transpose **Q-sharp** under **G**. 
 - **rate of transition from Q to R** – transition probability from **Q** to a specific target **R**. 

2. Inserting the definition of **r** gives a double sum: 
 k([Q]) = \sum_{R \in G \cdot Q^{\#}} \sum_{g \in \mathrm{Mir}(Q)} w(g)\,\mathbf 1_{\{R = g \cdot Q^{\#}\}}. 
 *Where:* 
 - **indicator function that R equals g acting on Q-sharp** – indicator that is **1** exactly when **R** equals the image of **Q-sharp** under **g**. 

3. Because each distinct mirror‑isotopism **g** produces a unique target **R equals g acting on Q-sharp**, the outer sum counts each **g** once and collapses to a single sum: 
 k([Q]) = \sum_{g \in \mathrm{Mir}(Q)} w(g). 

Thus the effective mirror rate is simply the total weight of all mirror‑isotopisms of **Q**.

--- 

### Theorem 3.15 – Structural Vanishing Criterion 

**Statement.** Under Assumption 3.11, the following are equivalent: 

1. **k([Q]) = 0**; 
2. **the mirror group of Q is empty**; 
3. No isotopism maps **Q** to its mirror **Q-sharp**. 

**Reasoning.** Proposition 3.14 tells us that **k([Q])** equals the sum of non‑negative weights **w(g)** over **the mirror group of Q**. A sum of non‑negative terms can be zero only if the index set is empty, establishing (1)⇔(2). By definition, **the mirror group of Q** consists precisely of those isotopisms sending **Q** to **Q-sharp**, so (2)⇔(3).

**Definition 3.16 (Structurally chiral isotopy class).** 
An isotopy class **[Q]** is *structurally chiral* when **the mirror group of Q is empty**. In this case the effective mirror rate vanishes, i.e. **k([Q])=0**.

**Remark 3.17.** 
If **the mirror group of Q is not empty**, then **Q** is isotopic to its mirror, the class **[Q]** can transition to **the equivalence class of Q-sharp** under the symmetry‑generated dynamics, and the system is dynamically racemic. Conversely, structural chirality blocks any mirror transition at the algebraic level, guaranteeing permanent handedness.

--- 

### Proposition 3.18 – Commutative Quasigroups Are Not Structurally Chiral 

If a quasigroup **Q is defined by set X and operation dot** satisfies **x dot y = y dot x** for all **x, y are elements of X**, then its transpose **Q-sharp** coincides with **Q**. Consequently the identity isotopism belongs to **the mirror group of Q**, so **the mirror group of Q is not empty** and **[Q]** cannot be structurally chiral. Moreover, Assumption 3.11 guarantees **w(id) is greater than 0**, implying **k([Q])>0**.

--- 

### Example 3.19 – An Order‑7 Structurally Chiral Quasigroup 

**The quasigroup.** Let **X is the set {1,..., 7}** and define the operation **dot** by the Cayley table shown in Figure 12. 

![](crops/figure12) 
*Figure 12 – Cayley table of the order‑7 quasigroup.*

**Why it is not isotopic to its transpose.** 
Consider the Latin square representation of the table. An *intercalate* is a **2 by 2** subsquare whose four entries form a mini‑Latin square. For each cell we count how many intercalates contain it. This multiset of counts is invariant under any isotopy because isotopy merely permutes rows, columns, and symbols independently.

In this table there exists a **row** (row 1) that contains two cells— **(1,2)** and **(1,5)** —each belonging to **three** distinct intercalates. No **column** anywhere in the square has two cells with count three. Since isotopy cannot turn a row into a column, the transpose (which swaps rows and columns) cannot be reached by any isotopism. Hence **Mir(Q) is the empty set**.

**Intercalate count example.** 
Cell **(1,2)** participates in three intercalates, namely the subsquares with corners 
 **the set of cells {(1,2), (1,5), (4,2), (4,5)}**, 
 **the set of cells {(1,2), (1,6), (5,2), (5,6)}**, and 
 **the set of cells {(1,2), (1,7), (6,2), (6,7)}**. 

Because **Mir(Q) is the empty set**, Proposition 3.14 gives 
k([Q]) = \sum_{g\in\emptyset} w(g) = 0, 
so the isotopy class **[Q]** is **structurally chiral**.

--- 

### Conclusion – Linking Algebraic Obstruction to Dynamical Chirality 

Lemma 3.13 guarantees that mirror‑generated transition rates respect the global symmetry **G**. Proposition 3.14 reduces the effective mirror rate **k([Q])** to the simple weight sum **Σ(g in Mir(Q)) w(g)**. Theorem 3.15 then translates the vanishing of this sum into the purely algebraic condition **Mir(Q) is the empty set**, which we call **structural chirality**. 

The order‑7 quasigroup of Example 3.19 demonstrates that such chiral isotopy classes are not merely pathological; the intercalate‑distribution argument provides a concrete, combinatorial certificate of **Mir(Q) is the empty set**. 

In the dynamical picture each isotopy class **[Q]** participates in a two‑state continuous‑time Markov chain **the set containing [Q] and [Q#]** with transition rate **k([Q])**. When **k([Q])=0** the chain is frozen, yielding permanent chiral stability; when **k([Q])>0** the system inevitably racemizes. This completes the algebraic characterization of when handedness can persist under the gauge‑invariant dynamics.