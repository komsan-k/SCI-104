# Tutorial: Principles of Quantum Bits (Qubits)

**Level:** Undergraduate beginner  
**Duration:** 3 hours  
**Environment:** Google Colab or Jupyter Notebook  
**Prerequisites:** Basic Python, binary numbers, vectors, complex numbers

## Learning outcomes
Students will be able to: (1) contrast classical bits and qubits; (2) use Dirac notation and state vectors; (3) calculate measurement probabilities; (4) explain Bloch-sphere coordinates and relative phase; (5) apply basic gates; (6) demonstrate interference and measurement in Qiskit.

## 1. Classical bits and qubits
A classical bit is 0 or 1. A pure qubit is a normalized vector in a two-dimensional complex Hilbert space:

$$|\psi\rangle=\alpha|0\rangle+\beta|1\rangle,\quad \alpha,\beta\in\mathbb C,\quad |\alpha|^2+|\beta|^2=1.$$

The computational basis states are

$$|0\rangle=\begin{bmatrix}1\\0\end{bmatrix},\qquad |1\rangle=\begin{bmatrix}0\\1\end{bmatrix}.$$

A qubit does **not** reveal both amplitudes in a single measurement. Computational-basis measurement yields 0 with probability $|\alpha|^2$ and 1 with probability $|\beta|^2$.

### Worked example

$$|\psi\rangle=\frac{\sqrt3}{2}|0\rangle+\frac12|1\rangle.$$

Thus $P(0)=3/4$ and $P(1)=1/4$.

**Checkpoint 1:** Find the probabilities for $(|0\rangle+i|1\rangle)/\sqrt2$.

## 2. Bloch sphere and phase
Ignoring physically irrelevant global phase, every pure qubit has the form

$$|\psi\rangle=\cos\left(\frac\theta2\right)|0\rangle+e^{i\phi}\sin\left(\frac\theta2\right)|1\rangle.$$

The Bloch vector is $(\sin\theta\cos\phi,\sin\theta\sin\phi,\cos\theta)$.

- $|0\rangle$: north pole, $\theta=0$.
- $|1\rangle$: south pole, $\theta=\pi$.
- $|+\rangle=(|0\rangle+|1\rangle)/\sqrt2$: positive X axis.
- $|-\rangle=(|0\rangle-|1\rangle)/\sqrt2$: negative X axis.
- $(|0\rangle+i|1\rangle)/\sqrt2$: positive Y axis.

**Relative phase** affects subsequent interference even though it does not change computational-basis probabilities by itself. A global phase $e^{i\gamma}|\psi\rangle$ is not observable.

## 3. Quantum gates
Quantum gates are unitary: $U^\dagger U=I$.

$$X=\begin{bmatrix}0&1\\1&0\end{bmatrix},\quad Z=\begin{bmatrix}1&0\\0&-1\end{bmatrix},\quad H=\frac1{\sqrt2}\begin{bmatrix}1&1\\1&-1\end{bmatrix}.$$

- X exchanges $|0\rangle$ and $|1\rangle$.
- Z changes the relative phase of $|1\rangle$.
- H maps $|0\rangle\mapsto|+\rangle$ and $|1\rangle\mapsto|-\rangle$.

A sequence of gates acts from right to left in matrix notation. For example, $HZH|0\rangle=X|0\rangle=|1\rangle$.

## 4. Online setup
Open <https://colab.research.google.com/> and create a Python notebook. Install:

```python
%pip install -q qiskit qiskit-aer matplotlib pylatexenc
```

Then import:

```python
import numpy as np
from qiskit import QuantumCircuit
from qiskit.quantum_info import Statevector, Operator
from qiskit_aer import AerSimulator
from qiskit.visualization import plot_bloch_multivector, plot_histogram
```

## 5. Experiment 1: Computational basis states

```python
qc0 = QuantumCircuit(1)
print('State |0>:', Statevector.from_instruction(qc0).data)

qc1 = QuantumCircuit(1)
qc1.x(0)
print('State |1>:', Statevector.from_instruction(qc1).data)
display(qc1.draw('mpl'))
display(plot_bloch_multivector(Statevector.from_instruction(qc1)))
```

**Checkpoint 2:** Explain why the X gate is sometimes called a quantum NOT gate.

## 6. Experiment 2: Hadamard superposition

```python
qc = QuantumCircuit(1)
qc.h(0)
state = Statevector.from_instruction(qc)
print('Amplitudes:', state.data)
print('Probabilities:', state.probabilities_dict())
display(qc.draw('mpl'))
display(plot_bloch_multivector(state))
```

Expected amplitudes: $[1/\sqrt2,1/\sqrt2]$; expected probabilities: 0.5 each.

## 7. Experiment 3: Measurement statistics

```python
qc = QuantumCircuit(1, 1)
qc.h(0)
qc.measure(0, 0)
simulator = AerSimulator()
for shots in [100, 1000, 10000]:
    counts = simulator.run(qc, shots=shots).result().get_counts()
    print(shots, counts)
    display(plot_histogram(counts))
```

**Checkpoint 3:** Why do counts fluctuate between runs? Why do relative frequencies usually get closer to 0.5 as shots increase?

## 8. Experiment 4: Relative phase and interference
Compare the states $|+\rangle$ and $|-\rangle$. Both give 50/50 measurement probabilities in the Z basis, but another H gate distinguishes them.

```python
for apply_z in [False, True]:
    qc = QuantumCircuit(1)
    qc.h(0)
    if apply_z:
        qc.z(0)
    print('After H' + ('ZH' if apply_z else '') + ':',
          Statevector.from_instruction(qc).data)
    qc.h(0)
    print('After final H:', Statevector.from_instruction(qc).probabilities_dict())
    display(qc.draw('mpl'))
```

The H-H circuit ends in $|0\rangle$; the H-Z-H circuit ends in $|1\rangle$ (ideal conditions). This is a simple example of **interference**.

## 9. Experiment 5: Bloch-sphere rotations

```python
from math import pi
for angle in [0, pi/4, pi/2, pi]:
    qc = QuantumCircuit(1)
    qc.ry(angle, 0)
    state = Statevector.from_instruction(qc)
    print('theta =', angle, 'probabilities:', state.probabilities_dict())
    display(plot_bloch_multivector(state))
```

**Checkpoint 4:** Predict $P(1)$ at $\theta=\pi/2$ and $\theta=\pi$ before running the cell.

## 10. Experiment 6: Gate matrices and reversibility

```python
qc = QuantumCircuit(1)
qc.h(0)
qc.z(0)
qc.h(0)
print('HZH matrix:\n', Operator(qc).data)
print('Equals X?', np.allclose(Operator(qc).data, np.array([[0,1],[1,0]])))
print('Unitary?', np.allclose(Operator(qc).data.conj().T @ Operator(qc).data, np.eye(2)))
```

## 11. Common misconceptions
1. **A qubit is not simply a classical bit with an unknown value.** It can have coherent relative phase and exhibit interference.
2. **Superposition is not a way to read all answers at once.** Measurements return classical outcomes.
3. **A global phase is not measurable**, but relative phase can affect interference.
4. **Measurement probabilities depend on the basis.** A state that is definite in one basis can be random in another.
5. **Not every qubit state is pure.** Noise and incomplete information may require a density-matrix description.

## 12. Exercises
1. Normalize $2|0\rangle+i|1\rangle$ and calculate $P(0)$ and $P(1)$.
2. Show by matrix multiplication that $H^2=I$.
3. Prepare $|-\rangle$ using X and H gates.
4. Compare $HZH$ and X using Qiskit `Operator`.
5. Use `ry` to prepare a state with $P(1)=0.25$. Determine the rotation angle.
6. Explain why a Z gate changes $|+\rangle$ even though it does not change its Z-basis probabilities.

<!---
## 13. Suggested assessment (100 marks)
| Item | Marks |
|---|---:|
| Mathematical representation and normalization | 20 |
| State preparation and Bloch visualization | 20 |
| Gate operations and matrices | 20 |
| Measurement statistics | 15 |
| Phase and interference explanation | 15 |
| Code quality and interpretation | 10 |

**Submission:** Executed notebook, circuit diagrams, measurement plots, answers to checkpoints, and a short conclusion.
--->
## Further reading
- Qiskit documentation: <https://docs.quantum.ibm.com/>
- IBM Quantum Learning: <https://quantum.cloud.ibm.com/learning>
- Google Colab: <https://colab.research.google.com/>
