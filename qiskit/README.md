# Undergraduate Qiskit Tutorial

**Format:** Undergraduate hands-on tutorial | **Platform:** Google Colab or Jupyter | **Duration:** 3 hours

## Learning outcomes

By the end of this lab, students will be able to create quantum circuits, manipulate qubits with gates, visualize statevectors and Bloch spheres, simulate measurement, construct a Bell state, implement a half adder, and compare a small QFT circuit with its Fourier matrix.

## How to run

Open [Google Colab](https://colab.research.google.com/), create a notebook, and run the Python blocks below in order. Run the installation block once per fresh Colab runtime.

# Undergraduate Qiskit Tutorial
3-hour beginner lab. Run cells sequentially in Google Colab.

```python
%pip install -q qiskit qiskit-aer matplotlib pylatexenc
```

```python
import numpy as np
from qiskit import QuantumCircuit
from qiskit.quantum_info import Statevector, Operator
from qiskit_aer import AerSimulator
from qiskit.visualization import plot_histogram, plot_bloch_multivector
from IPython.display import display
sim = AerSimulator()
```

## 1. Basis state and X gate

```python
qc=QuantumCircuit(1)
display(plot_bloch_multivector(Statevector.from_instruction(qc)))
qc.x(0)
print(Statevector.from_instruction(qc).data)
display(qc.draw("mpl"))
```

## 2. Hadamard and measurement

```python
qc=QuantumCircuit(1,1)
qc.h(0)
print(Statevector.from_instruction(qc.remove_final_measurements(inplace=False)).probabilities_dict())
qc.measure(0,0)
counts=sim.run(qc,shots=1024).result().get_counts()
print(counts)
display(plot_histogram(counts))
```

## 3. Gate matrices and order

```python
qc=QuantumCircuit(1)
qc.h(0);qc.z(0)
print("Matrix H then Z:\n",Operator(qc).data)
print("State:",Statevector.from_instruction(qc).data)
display(plot_bloch_multivector(Statevector.from_instruction(qc)))
```

## 4. Bell entanglement

```python
bell=QuantumCircuit(2)
bell.h(0);bell.cx(0,1)
print(Statevector.from_instruction(bell).data)
display(bell.draw("mpl"))
meas=bell.copy();meas.measure_all()
counts=sim.run(meas,shots=1024).result().get_counts()
print(counts);display(plot_histogram(counts))
```

## 5. Quantum half adder
Qubit 0=A, 1=B, 2=carry (initially 0). Compute carry before changing B into the sum.

```python
def half_adder(a,b):
    qc=QuantumCircuit(3,2)
    if a:qc.x(0)
    if b:qc.x(1)
    qc.ccx(0,1,2)
    qc.cx(0,1)
    qc.measure(1,0) # sum
    qc.measure(2,1) # carry
    return qc
for a in (0,1):
    for b in (0,1):
        qc=half_adder(a,b)
        counts=sim.run(qc,shots=128).result().get_counts()
        print(f"A={a}, B={b}, carry/sum={counts}")
display(half_adder(1,1).draw("mpl"))
```

## 6. Quantum Fourier Transform
For N=2^n, the positive-sign QFT matrix is F[k,x]=exp(2πikx/N)/sqrt(N).

```python
def qft(n):
    qc=QuantumCircuit(n)
    for j in range(n):
        qc.h(j)
        for k in range(j+1,n):
            qc.cp(np.pi/2**(k-j),k,j)
    for j in range(n//2):
        qc.swap(j,n-1-j)
    return qc
n=2;N=2**n
indices=np.arange(N)
F=np.exp(2j*np.pi*np.outer(indices,indices)/N)/np.sqrt(N)
U=Operator(qft(n)).data
print("QFT equals Fourier matrix:",np.allclose(U,F))
print("QFT inverse check:",np.allclose(U.conj().T@U,np.eye(N)))
display(qft(n).draw("mpl"))
```

## Checkpoints
1. Why does H|0> give approximately equal 0 and 1 counts?
2. What happens if you apply H twice?
3. Show the half-adder truth table.
4. Verify IQFT(QFT(|01>)) = |01>.
5. Explain why the normalized Fourier matrix is unitary.

**Submission:** executed notebook, screenshots of plots, and short answers.

## Student submission

Submit the executed notebook or a report containing circuit diagrams, statevectors, histograms, the half-adder truth table, and written checkpoint answers.
