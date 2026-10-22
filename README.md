# Open-System Quantum Annealing

Numerical exploration of finite-time quantum annealing for weighted Max-Cut problems, connecting classical optimization with Ising spin models and open quantum dynamics.

The project begins with isolated-system evolution and extends to density-matrix dynamics with local Markovian dephasing. A planned research direction is to investigate whether engineered dissipation can improve final ground-state preparation within a limited annealing duration.

## 1. From Weighted Max-Cut to an Ising Hamiltonian

For a graph with edge set $E$ and weights $w_{ij}$, the weighted Max-Cut objective is

$$
C(\mathbf{z}) = \sum_{(i,j)\in E}\frac{w_{ij}}{2}(1-z_i z_j), \qquad z_i\in\{-1,+1\}.
$$

Each variable specifies which of the two partitions contains a vertex. An edge contributes its weight when its endpoints belong to opposite partitions.

Replacing the classical variables by Pauli operators gives

$$
\hat{C} = \sum_{(i,j)\in E}\frac{w_{ij}}{2}(I-Z_iZ_j).
$$

The problem Hamiltonian is

$$
H_P = -\hat{C} = -\frac{1}{2}\sum_{(i,j)\in E}w_{ij}I+\frac{1}{2}\sum_{(i,j)\in E}w_{ij}Z_iZ_j.
$$

Consequently,

$$
E_{\min} = -C_{\max}.
$$

Positive edge weights correspond to antiferromagnetic Ising couplings. The constant energy offset is retained so that problem energy equals the negative cut value.

Graph connectivity determines which spins interact. Distances in a graph drawing do not determine interaction strength unless a spatial dependence is explicitly specified.

**Max-Cut is the optimization problem; quantum annealing is the dynamical method used to seek its solution.**

## 2. Quantum Annealing Protocol

The driver Hamiltonian is

$$
H_D = -\sum_{i=0}^{N-1}X_i.
$$

Its ground state is the product state

$$
|\psi_0\rangle = |+\rangle^{\otimes N}, \qquad |+\rangle = \frac{|0\rangle+|1\rangle}{\sqrt{2}}.
$$

A linear schedule interpolates between the driver and problem Hamiltonians:

$$
H(s) = (1-s)H_D+sH_P, \qquad s=\frac{t}{T}\in[0,1].
$$

Here, $T$ is the total annealing duration.

The objective is to obtain a high final probability of measuring an optimal bitstring. Finite-duration evolution can produce excitations, so successful optimization is evaluated from the final state.

The computational-basis convention is

$$
Z_i|b_i\rangle = (-1)^{b_i}|b_i\rangle.
$$

Spin 0 is represented by the leftmost bit in a displayed bitstring.

## 3. Models Studied

### Three-spin weighted triangle

| Edge | Weight |
|---|---:|
| $(0,1)$ | 1 |
| $(1,2)$ | 2 |
| $(0,2)$ | 3 |

For this instance,

$$
C_{\max}=5, \qquad E_{\min}=-5.
$$

The optimal basis states are

$$
|001\rangle, \qquad |110\rangle.
$$

These complementary bitstrings describe the same partition with its labels exchanged. Both are included when calculating optimization success.

### Five-spin nearest-neighbor open chain

The chain contains four unit-weight edges:

$$
E = \{(0,1),(1,2),(2,3),(3,4)\}.
$$

All four antiferromagnetic bonds can be satisfied simultaneously:

$$
C_{\max}=4, \qquad E_{\min}=-4.
$$

The optimal states are

$$
|01010\rangle, \qquad |10101\rangle.
$$

This provides an unfrustrated reference model.

### Five-spin fully connected model

Every distinct pair interacts with unit weight:



```math
E = \left\lbrace (i,j) \mid 0 \leq i < j \leq 4 \right\rbrace.
```

There are ten edges. This uniform all-to-all model includes nearest-neighbor and longer-range pairs, without distance-dependent decay.

For a bitstring with Hamming weight $k$, meaning $k$ entries equal to one,

$$
C = k(5-k).
$$

Therefore,

$$
C_{\max}=6, \qquad E_{\min}=-6.
$$

The ground-state subspace contains the 20 bitstrings with Hamming weight 2 or 3. These correspond to 10 partitions when complementary bitstrings are identified.

The model is frustrated: it is impossible to place every connected pair in opposite partitions.

## 4. Isolated-System Dynamics

Closed-system evolution follows the time-dependent Schrödinger equation, with $\hbar=1$:

$$
i\frac{d}{dt}|\psi(t)\rangle = H(t)|\psi(t)\rangle.
$$

Using normalized time,

$$
\frac{d}{ds}|\psi(s)\rangle = -iT H(s)|\psi(s)\rangle.
$$

Different annealing durations are compared to examine their effect on final success probability and energy.

## 5. Density-Matrix Formulation

The isolated dynamics can also be written using the von Neumann equation:

$$
\frac{d\rho}{dt} = -i[H(t),\rho(t)].
$$

The initial density matrix is

$$
\rho(0) = |\psi_0\rangle\langle\psi_0|.
$$

For isolated evolution, an initially pure state remains pure. Agreement between state-vector and density-matrix calculations provides a consistency check before introducing dissipation.

## 6. Open-System Extension: Local Dephasing

The current open-system model uses independent local dephasing channels:

$$
\frac{d\rho}{dt} = -i[H(t),\rho]+\gamma_\phi\sum_i\left(Z_i\rho Z_i-\rho\right).
$$

The corresponding jump operators are

$$
L_i = \sqrt{\gamma_\phi}\,Z_i.
$$

Here, $\rho$ is the reduced density matrix of the spins. Environmental degrees of freedom are not explicitly included in its indices; their modeled influence enters through the dissipator.

### Dephasing-rate convention

For a single spin under the dissipator alone,

$$
\frac{d\rho_{01}}{dt} = -2\gamma_\phi\rho_{01}.
$$

Thus, $\gamma_\phi$ is the coefficient multiplying each dissipative term, while the single-spin off-diagonal decay rate is $2\gamma_\phi$.

### Physical interpretation

The noise dephases in the computational $Z$ basis. During annealing, this generally differs from the instantaneous energy eigenbasis because of the transverse-field driver.

The dissipator leaves computational-basis populations unchanged directly, but modifies the coherences involved in Hamiltonian-driven population transfer.

This model describes dephasing rather than thermal relaxation or cooling. No bath temperature is assigned in this phenomenological implementation.

In normalized time, the equation becomes

$$
\frac{d\rho}{ds} = T\left[-i[H(s),\rho]+\gamma_\phi\sum_i\left(Z_i\rho Z_i-\rho\right)\right].
$$

## 7. Performance Measures

### Mean cut value


```math
\langle C\rangle(t) = \mathrm{Tr}[\rho(t)\hat{C}].
```

This is the average cut value obtained by computational-basis measurement.

### Problem-Hamiltonian energy

$$
E_P(t) = \mathrm{Tr}[\rho(t)H_P] = -\langle C\rangle(t).
$$

### Final success probability

$$
P_{\mathrm{GS}}(T) = \mathrm{Tr}[\rho(T)\Pi_{\mathrm{GS}}].
$$

The projector $\Pi_{\mathrm{GS}}$ includes every ground state of $H_P$. This accounts for degeneracy and gives the probability of measuring an optimal bitstring.

### Final residual energy

$$
\epsilon_{\mathrm{res}} = E_P(T)-E_{\min}.
$$

A residual energy close to zero indicates an average energy close to the optimum.

### Purity

$$
\mathcal{P}(t) = \mathrm{Tr}[\rho(t)^2].
$$

Purity equals one for a pure state. For an $N$-spin maximally mixed state, it equals $2^{-N}$.

### Interpreting energy plots

The instantaneous total energy and the problem energy are distinct during the ramp:

$$
E_{\mathrm{tot}}(s) = \mathrm{Tr}[\rho(s)H(s)], \qquad E_P(s) = \mathrm{Tr}[\rho(s)H_P].
$$

They coincide at $s=1$, when $H(1)=H_P$.

The exact optimum is calculated independently by enumerating the small computational basis. It provides a benchmark against which the dynamical results are evaluated.

## 8. Numerical Implementation

The notebooks use:

- **NumPy:** arrays, spin operators, and linear algebra;
- **SciPy:** time integration with `solve_ivp`;
- **Matplotlib:** plots of dynamics and final performance;
- **Jupyter notebooks:** explanations, executable code, and outputs.

The supplied simulations use the `DOP853` integration method with controlled numerical tolerances.

The notebooks cover the following progression:

1. Three-spin closed-system annealing.
2. Five-spin nearest-neighbor chain.
3. Five-spin fully connected model.
4. Three-spin density-matrix evolution.
5. Three-spin annealing with local dephasing.

## 9. Numerical Validation

Checks include:

- agreement between state-vector and density-matrix results for isolated dynamics;
- recovery of isolated dynamics when $\gamma_\phi=0$;
- preservation of state normalization and density-matrix trace;
- density-matrix Hermiticity;
- positivity within numerical tolerance;
- comparison with independently enumerated optimal cuts.

Small negative density-matrix eigenvalues at the integration-error scale are monitored as numerical errors. Significant violations require checking solver tolerances or the implementation.

## 10. Units

Energies are measured in a reference scale $E_0$, and times in $\hbar/E_0$.

The conversion to physical units is

$$
t_{\mathrm{physical}} = \frac{\hbar}{E_0}t.
$$

The physical dephasing coefficient is

$$
\gamma_{\phi,\mathrm{physical}} = \frac{E_0}{\hbar}\gamma_\phi.
$$

For example, a numerical duration $T=10$ corresponds to $10\hbar/E_0$. Its value in seconds depends on the chosen energy scale.

The symbol $T$ denotes annealing duration, not temperature.

## 11. Running the Notebooks

Install the required packages:

```bash
python -m pip install numpy scipy matplotlib jupyter ipykernel
```

Then:

1. Open a notebook in VS Code or Jupyter.
2. Select the Python environment containing these packages.
3. Run the cells in order.
4. Inspect the numerical checks before interpreting the plots.

The main adjustable parameters are:

- the graph edge list and weights;
- annealing duration $T$;
- dephasing coefficient $\gamma_\phi$;
- solver tolerances and output sampling.

When changing the graph, recalculate the exact optimal cut and its degeneracy.

## 12. Current Scope and Research Direction

The completed simulations establish small-system baselines for isolated annealing and local dephasing. They illustrate how connectivity, frustration, degeneracy, and decoherence enter the dynamical optimization problem.

Planned extensions include:

- thermal initial states;
- a specified thermal-relaxation model;
- engineered population-transfer channels with nonnegative, bounded rates;
- local or few-spin jump operators that reduce reliance on instantaneous many-body spectral information;
- comparisons at fixed annealing duration and under explicit coupling constraints.

The central research question is:

> Under what conditions can engineered Markovian dissipation improve final ground-subspace occupation during finite-time quantum annealing?

A related theoretical objective is to investigate how a framework designed for prescribed thermal populations can be adapted to an endpoint ground-state objective.

The present examples provide implementation and validation benchmarks. Performance on difficult instances, scalability, and benefits from engineered dissipation remain subjects for further investigation.

## Author

**Saikat Mistry**

Research interests: nonequilibrium quantum dynamics, open quantum systems, engineered dissipation, and quantum annealing.
