# Engineered Dissipation for Max-Cut Quantum Annealing

This project explores Glauber-type engineered dissipation for weighted Max-Cut quantum annealing. The aim is to construct dissipative spin-flip processes that favor larger cut values using local graph information, without supplying the optimal configuration or diagonalizing the instantaneous annealing Hamiltonian.

**Status:** The theoretical construction is outlined below. Numerical implementation and benchmarking are planned in `01_three_spin_engineered_dissipation.ipynb`.

## 1. Problem Hamiltonian and Annealing Protocol

For a weighted graph with vertices $V$ and edges $E$, the cut value of a spin configuration is

```math
C(\boldsymbol{\sigma}) = \frac{1}{2}\sum_{(i,j)\in E} w_{ij}(1-\sigma_i\sigma_j),
\qquad \sigma_i\in\{-1,+1\}.
```

The corresponding quantum problem Hamiltonian is

```math
H_P = -\frac{1}{2}\sum_{(i,j)\in E} w_{ij}(I-Z_iZ_j).
```

Its ground-state configurations maximize the cut value. The identity contribution can be omitted when calculating dynamics or energy differences.

We use a transverse-field driver,

```math
H_D = -J\sum_i X_i,
```

and an annealing interpolation,

```math
H(t) = [1-s(t)]H_D+s(t)H_P,
\qquad s(0)=0,\quad s(T)=1.
```

The initial benchmark will use a linear schedule, $s(t)=t/T$, and the initial state

```math
\rho(0) = \left(|+\rangle\langle+|\right)^{\otimes N}.
```

Throughout, we set $\hbar=1$.

## 2. Local Cost Changes

Let $z$ denote a computational-basis bit string, with spin values $\sigma_i=(-1)^{z_i}$. Let $z^{(i)}$ denote the configuration obtained by flipping bit $i$.

The change in cut value is

```math
\Delta_i C(z) = C(z^{(i)})-C(z)
= \sigma_i\sum_{j\in\mathcal{N}(i)} w_{ij}\sigma_j.
```

Only edges incident on vertex $i$ contribute. Since the problem energy is $E_P(z)=-C(z)$,

```math
\Delta_i E_P(z) = -\Delta_i C(z).
```

Therefore:

- $\Delta_i C(z)>0$: the flip improves the cut and lowers the problem energy.
- $\Delta_i C(z)<0$: the flip worsens the cut.
- $\Delta_i C(z)=0$: the cut value is unchanged.

For the illustrative triangle in the handwritten notes,

```math
w_{12}=3,\qquad w_{23}=5,\qquad w_{13}=2.
```

The configuration $|001\rangle$ has cut value $7$. Flipping the first spin produces $|101\rangle$, with cut value $8$:

```math
\Delta_1 C(001) = (+1)\left[3(+1)+2(-1)\right] = 1.
```

These example weights should be distinguished from any weights used in earlier notebooks.

## 3. Glauber-Type Transition Rates

Assign each spin flip the continuous-time rate

```math
W_i(z) = \frac{\gamma_i}{1+\exp[-\beta_{\mathrm{eff}}\Delta_i C(z)]}.
```

Here, $\gamma_i>0$ sets the transition-rate scale, while $\beta_{\mathrm{eff}}\geq 0$ controls the preference for lower problem energy.

The limiting behavior is

```math
W_i(z)\approx
\begin{cases}
\gamma_i, & \beta_{\mathrm{eff}}\Delta_i C(z)\gg 1,\\
0, & \beta_{\mathrm{eff}}\Delta_i C(z)\ll -1,\\
\gamma_i/2, & \Delta_i C(z)=0.
\end{cases}
```

The last case is exact. At $\beta_{\mathrm{eff}}=0$, all flips have rate $\gamma_i/2$.

Since reversing the flip reverses the cost change,

```math
\Delta_i C(z^{(i)}) = -\Delta_i C(z),
```

the forward and reverse rates obey

```math
\frac{W_i(z)}{W_i(z^{(i)})}
= \exp[\beta_{\mathrm{eff}}\Delta_i C(z)]
= \exp[-\beta_{\mathrm{eff}}\Delta_i E_P(z)].
```

For fixed parameters, this is classical detailed balance with respect to

```math
p_\beta(z) = \frac{\exp[\beta_{\mathrm{eff}}C(z)]}{\mathcal{Z}},
\qquad
\mathcal{Z} = \sum_z \exp[\beta_{\mathrm{eff}}C(z)].
```

At finite $\beta_{\mathrm{eff}}$, unfavorable moves remain possible. Increasing $\beta_{\mathrm{eff}}$ strengthens the preference for high-cut configurations.

## 4. Two Lindblad Constructions

### Configuration-resolved jumps

A direct embedding of the classical transitions uses

```math
L_{i,z} = \sqrt{W_i(z)}\,|z^{(i)}\rangle\langle z|.
```

This construction has $N2^N$ jump operators: $24$ for three spins.

### One conditional jump per spin

Define the diagonal cost-change operator

```math
\Delta_i\widehat{C} = Z_i\sum_{j\in\mathcal{N}(i)} w_{ij}Z_j.
```

It satisfies

```math
\Delta_i\widehat{C}|z\rangle = \Delta_i C(z)|z\rangle.
```

Introduce the positive rate operator

```math
F_i = \gamma_i\left[I+\exp(-\beta_{\mathrm{eff}}\Delta_i\widehat{C})\right]^{-1}.
```

The conditional jump is

```math
L_i = X_i\sqrt{F_i}.
```

The operator ordering matters: the rate is evaluated on the initial configuration before the spin is flipped.

Consequently,

```math
L_i|z\rangle = \sqrt{W_i(z)}\,|z^{(i)}\rangle,
\qquad L_i^\dagger L_i = F_i.
```

This construction gives $N$ jump operators. Each acts on spin $i$ and, through its rate dependence, the neighboring spins.

Because $F_i$ is diagonal, its entries can be calculated directly from local cost changes without a general matrix exponential.

## 5. Master Equation and Population Dynamics

For the conditional-jump construction, the evolution is

```math
\frac{d\rho}{dt}
= -i[H(t),\rho]
+\sum_i\left(
L_i\rho L_i^\dagger-\frac{1}{2}\{F_i,\rho\}
\right).
```

The nonnegative rates give a Lindblad generator and hence completely positive, trace-preserving evolution.

Writing $p_z=\langle z|\rho|z\rangle$, the dissipative contribution to the population equation is

```math
\left.\frac{dp_z}{dt}\right|_{\mathrm{diss}}
= \sum_i\left[
W_i(z^{(i)})p_{z^{(i)}}-W_i(z)p_z
\right].
```

The first term is population gained from neighboring configurations. The second is population lost through outgoing flips.

The transverse-field Hamiltonian adds coherent population currents that depend on off-diagonal density-matrix elements.

## 6. Why the Two Constructions Differ

Although

```math
L_i = \sum_z L_{i,z},
```

their dissipators are generally different:

```math
\mathcal{D}\left[\sum_z L_{i,z}\right](\rho)
\neq \sum_z\mathcal{D}[L_{i,z}](\rho),
```

where

```math
\mathcal{D}[L](\rho)
= L\rho L^\dagger-\frac{1}{2}\{L^\dagger L,\rho\}.
```

In general, combining two jump operators introduces cross terms:

```math
\begin{aligned}
\mathcal{D}[A+B](\rho)
={}&\mathcal{D}[A](\rho)+\mathcal{D}[B](\rho)\\
&+A\rho B^\dagger+B\rho A^\dagger\\
&-\frac{1}{2}\{A^\dagger B+B^\dagger A,\rho\}.
\end{aligned}
```

For the jumps associated with a fixed spin $i$, the two constructions have the same anticommutator contribution. Their difference comes from cross terms in the gain term, which act on quantum coherences.

Both constructions produce the same dissipative population equation, but they treat coherences differently. With a transverse-field driver, those coherence differences can subsequently affect populations and annealing performance.

The conditional construction is therefore a distinct quantum dissipative model, not merely an equivalent compression of the configuration-resolved model.

## 7. Scope and Limitations

- The rates use local changes in the fixed problem Hamiltonian $H_P$.
- Classical detailed balance for these rates does not establish thermalization to the Gibbs state of the full, noncommuting $H(t)$.
- For purely dissipative evolution restricted to diagonal states, the dynamics reduces to the classical Glauber process.
- A preference for improving flips does not guarantee monotonic improvement of the average cut or successful optimization within a finite runtime.
- In the zero-temperature limit, suppressed uphill moves can cause trapping in locally optimal configurations.
- Reducing the number of jump operators does not remove the exponential size of the full density matrix.
- No claim of quantum advantage or improved performance is made before benchmarking.

## 8. Planned Numerical Implementation

1. Construct the weighted graph, Pauli operators, $H_D$, and $H_P$.
2. Verify local cost changes against direct differences of cut values.
3. Implement both jump-operator constructions.
4. Integrate the density-matrix master equation.
5. Check trace preservation, Hermiticity, and positivity within numerical tolerance.
6. Verify the classical population equation with the driver switched off.
7. Compare closed-system annealing, local dephasing, and engineered dissipation.
8. Scan annealing time, dissipation strength, and effective inverse temperature.

### Planned observables

The expected cut value is

```math
\langle C\rangle_t = -\mathrm{Tr}[\rho(t)H_P].
```

The probability of obtaining an optimal configuration is

```math
P_{\mathrm{opt}}(t)
= \sum_{z\in\mathcal{S}_{\mathrm{opt}}}\langle z|\rho(t)|z\rangle,
```

where $\mathcal{S}_{\mathrm{opt}}$ contains all maximum-cut configurations.

The residual problem energy is

```math
E_{\mathrm{res}}(t)
= \mathrm{Tr}[\rho(t)H_P]-E_0,
```

where $E_0$ is the ground-state energy of $H_P$.

Additional diagnostics include final computational-basis populations, off-diagonal coherences, and purity:

```math
\mathcal{P}(t) = \mathrm{Tr}[\rho(t)^2].
```

For small-system benchmarking, optimal configurations and $E_0$ may be found by enumeration. This information is used only to assess performance, not to construct the dissipator.

## 9. Files

- `01_three_spin_engineered_dissipation.ipynb`: planned implementation and analysis.
- `figures/`: generated plots.
- `README.md`: model, assumptions, and implementation plan.
