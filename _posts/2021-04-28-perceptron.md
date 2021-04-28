---
title: Quantum perceptron
date: April, 09 2021
author: Nathan Habib
mathjax: true
---

# Implementation of the Quantum perceptron as found in [Simulating a perceptron on a quantum computer](https://arxiv.org/pdf/1412.3635.pdf)

## Imports


```python
import numpy as np
from numpy import pi

import matplotlib.pyplot as plt

# importing Qiskit
import qiskit
from qiskit import QuantumCircuit, Aer
from qiskit.visualization import plot_histogram
```

## Creating the gates we will need

We need to make a Unitary gate that encode the perceptron output into the phase.

That means:
$$
\vert{\psi}\rangle \longrightarrow e^{2\pi\phi}\vert{\psi}\rangle
$$
Where $\vert{\psi}\rangle$ is our input state, ($\psi \in \{0, 1\}$), and $2\pi\phi$ is the angle by which our phase has been shifted. The output of our perceptron is either $1$ or $0$ depending on wether $\phi$ is greater than $0.5$ or not (The phase shifted by more than half a turn).

### Apply a rotation of $k\pi$

First, this gate applies a __controlled__ global phase shift of $k\pi$ where $k$ is _the control_count_ argument.


```python
def Cglobal_phase_shift(quantum_circuit, theta, qubit, control_qubit, control_count):
    quantum_circuit.cp(theta * control_count, control_qubit, qubit)
    quantum_circuit.cx(control_qubit, qubit)
    quantum_circuit.cp(theta * control_count, control_qubit, qubit)
    quantum_circuit.cx(control_qubit, qubit)
    return quantum_circuit.to_gate(label="U0")
```

### Apply a rotation of $\frac{2\pi w_k}{2n}$ 

To use our perceptron, we need to be able to encode a phase $\phi$ into our input (a quantum state).
For that we use the gate defined below.

This gate is the same as:
$$
\begin{align}
U_k &=
\begin{pmatrix}
e^{(-2\pi w_k) / 2n} & 0 \\
0    & e^{(2\pi w_k) / 2n}
\end{pmatrix}
\end{align}
$$
where $n$ is our input size.

The gate $U_k$ is applied to the $k^{th}$ input, along with a global phase shift of $\pi$.
In the end, all of the gates have received a global phase shift of $\pi$ plus their respective shift depending on the weight associating with them ($w_k$).


```python
def perceptron_U(quantum_circuit, w, n, qubit):
    theta = (2 * pi * w) / (2 * n)
    quantum_circuit.u1(theta, qubit)
    quantum_circuit.x(qubit)
    quantum_circuit.u1(-theta, qubit)
    quantum_circuit.x(qubit)
    
    return quantum_circuit.to_gate(label="U" + str(qubit+1))
```

We will not be using the gate defined above because we need it to be __controlled__. That means we want it to activate only if another bit is set to $1$.

To be able To be used with the Quantum Phase Estimation algorithm, we need a controlled version of our Unitary, this is the one defined below.
We also need to be able to apply one gate multiple time efficiently that is why we have a control_count argument.
This argument will multiply our rotation by a certain amount (always an integer) so that for example, instead of applying our gate two times, we just apply one gate but with two times the rotation.


```python
def Cperceptron_U(quantum_circuit, w, n, qubit, control_qbit, control_count):
    theta = ((2 * pi * w) / (2 * n)) * control_count
    quantum_circuit.cp(theta, control_qbit, qubit)
    quantum_circuit.cx(control_qbit, qubit)
    quantum_circuit.cp(-theta, control_qbit, qubit)
    quantum_circuit.cx(control_qbit, qubit)
    return quantum_circuit.to_gate(label="U" + str(qubit+1))
```

## Adding the Quantum Phase Estimation algorithm

### Circuit parameters

#### How to find the weights by hand:

Suppose we want to reproduce the not function. That is, when input is 1 we want 0, and when input is 0 we want 1.

To do this we will want a weight associated with our input that produces a phase shift greater than pi when our input is 0 and lower when the input is 1.


We use two Unitary gates on our input, the first one shifts the phase by pi radian and the second by either $\frac{-2\pi w_1}{2n}$ or $\frac{2\pi w_1}{2n}$

The total shift then is:

$$
i\pi - \frac{2\pi w_1}{2n} = 2i\pi (\frac{w_1 - 1}{2}) \text{ when } x = 0\\
\text{and}\\
i\pi + \frac{2\pi w_1}{2n} = 2i\pi (\frac{w_1 + 1}{2}) \text{ when } x = 1\\
\text{We take, } \frac{w_1 - 1}{2} = \phi_0\\
\text{and } \frac{w_1 + 1}{2} = \phi_1
$$

Now remember we want the phase shift to be greater than half a turn when $x = 0$ than means that $\phi_0$ needs to be greater than $\frac{1}{2}$.

For the same reason, we want $\phi_1$ to be lower than $\frac{1}{2}$.

__Important:__ To measure to phase shift need to make use of the quantum fourrier transform, however in qiksit this algorithm and its inverse are swapped.
It doesnt matter as long as we stay coherent in the way we handle the QTF and its inverse. Because of the way the QFT works we also need to stay coherent on the direction
in which we rotate our phase. For this reason we need to rotate our phases the other way around, for that we only need to take the oppopsite of $\phi_0$ and $\phi_1$.

Knowing all of this we can graph $\phi_0$ and $\phi_1$ with $w_1$ as variable to see what values of $w_1$ will fulfill our needs.


```python
w = np.linspace(-1.5, 1.5, 10)
phi0 = (-w + 1) / 2
phi1 = (-w - 1) / 2

fig = plt.figure()
ax = fig.add_subplot(1, 1, 1)
ax.spines['left'].set_position('center')
ax.spines['bottom'].set_position('center')
ax.spines['right'].set_color('none')
ax.spines['top'].set_color('none')
ax.xaxis.set_ticks_position('bottom')
ax.yaxis.set_ticks_position('left')

plt.plot(w, phi0, "r")
plt.plot(w, phi1, "b");
```


![png](/assets/images/output_15_0.png)


We can then determine than $w_1$ need to be greather than between $-1$ and $0$ we therefore take $w_k = -0.5$. for us to simulate the not function.

__note:__ because we are talking about rotations around the unit circle, the weights are modulo 1.

#### Determine the precision

In our case, we chose a weight equal to -0.5, when we do the calculation we see that the rotation will then be a quarter of a turn and three quarter of a turn depending on
the value of x.


```python
τ = 2# percision of the QPE
n = 1 # size of our input vector
weights = [-0.5]
input_vec = [1]
```

### Putting it all together


```python
# This is our full circuit for our neuron
full = QuantumCircuit(τ + n, 2)

# apply H gates on measuring bits of the QPE
for qubit in range(τ):
    full.h(qubit)
    
# Setups our inputs
for i in range(len(input_vec)):
    if input_vec[i]:
        full.x(τ + i)

# Apply the Controlled Unitaries
for source in range(n):
    for control_bit in reversed(range(0, τ)):
        print(control_bit)
        full.append(Cglobal_phase_shift(QuantumCircuit(n + τ), pi, source + τ, control_bit, τ -control_bit), range(n + τ))
        u = Cperceptron_U(QuantumCircuit(τ + n), weights[source], n, source + τ, control_bit, τ - control_bit)
        full.append(u, range(τ + n))
    full.barrier()

# Apply the inverse QFT
inv_qft = qiskit.circuit.library.QFT(τ, inverse=True)
full.append(inv_qft, range(τ))
full.barrier()
# measure the most significant bit
full.measure(range(2), range(2))

# draw the decomposed circuit
full.decompose().draw('mpl')
```

    1
    0





![png](/assets/images/output_20_1.png)



## Running the circuit


```python
backend_sim = Aer.get_backend('qasm_simulator')
job_sim = qiskit.execute(full, backend_sim, shots=1024)
```

The bit to be read is the last one, (most significative bit).


```python
# Grab the results from the job.
result_sim = job_sim.result()
counts = result_sim.get_counts(full)
print(counts)
plot_histogram(counts)
```

    {'11': 1024}





![png](/assets/images/output_24_1.png)




```python

```
