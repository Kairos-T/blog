---
title: Quantum TRULY Random Number Generator
date: 2023-12-03 11:01:49
categories: Quantum
tags: 
    - QRNG 
    - Quantum
---


## Introduction
In computing, we often encounter the concept of randomness — or the illusion of it. A seemingly unpredictable sequence of numbers, they are usually crafted with algorithms that in reality, follow a predictable pattern. 

Despite classical computers' capabilities (think about how you are reading this article right now!), they are bound by the deterministic nature of algorithms, and these algorithms can be reversed! 

## Quantum Computing
Quantum computers, on the other hand, are not affected by such nature. Instead, they are probabilistic, and the outcome of a quantum computation is not guaranteed.

This is due to quantum computers leveraging on the principles of quantum mechanics, and to be exact — the Heisenberg Uncertainty Principle, and the concepts of superposition and entanglement.

## Heisenberg Uncertainty Principle
The principle states that certain pairs of properties cannot be precisely measured simultaneously — the more accurately one property is measured, the less accurately the other can be determined.

In the context of qubits (while not exactly the same), we can consider the the following properties as a pair:
- State
    - Similar to 1 and 0 in classical computing, expect that there is... an infinite possibility of the states in quantum computing
- Phase
    - Angle representing how much the wave function of the qubit has rotated around the Bloch sphere (geometric representation of pure state  space of a two-level quantum mechanical systems, where points on the surface represent the possible states of a qubit).

The more precisely the state is measured, the less precisely phrase is known, and vice versa. 

Hence, this ensures that values of the qubits cannot be precisely known at any given moment, providing the foundation for unpredictability in quantum systems.

## Superposition
As mentioned, classical bits exist in states of 0 or 1. Qubits however, can exist in a superposition of both states simultaneously, meaning that it is not definitively a 0 or a 1, but a combination of both.

Thus, only when it is measured, does the specific state  from its superposition get determined, truly, randomly.

## Entanglement
Entanglement introduces an additional layer of unpredictability. It occurs when two or more qubits are correlated, such that one state of a qubit is directly related to the state of another, regardless of the distance between them.

As such, changes to one entangled qubit will instantaneously affect its entangled partner. This ensures that their states are not independently determined, but are linked in a way that defy classical intuition. Thus, this contributes to the genuine randomness observed in quantum systems.

## Quantum Random Number Generator (QRNGs)
Leveraging on the properties of quantum systems, quantum random number generators are implemented. They involve measuring quantum states, such as the polarisation of photons, or the spin of electrons, generating unpredictable sequences of numbers. 

This is unlike classical pseudo-random number generators, and these numbers are theorectically impossible to predict.

## Simple QRNG
A simple QRNG can be implemented with Qiskit, a Python toolkit for quantum computing framework by IBM. 

I will be implementing this QRNG with two possible outcomes, 0 and 1, each with the same probability of outcome.

1. Install dependencies
    ```bash
    pip3 install qiskit
    pip3 install qiskit-aer
    ```

2. Import dependencies
    ```py
    from qiskit import QuantumCircuit, execute, Aer
    ```

3. Initialise a quantum circuit (qc) with one qubit (to generate one number) and one classical bit (for storing of the outcome number)
    ```py
    qc = QuantumCircuit(1, 1)
    ```

4. Put the qubit in superposition using a Hadamard gate, hence it has equal probabilities of being measured  0 or 1.
    ```py
    qc.h(0)
    ```

5. Measure the qubit, storing it in the classical bit 
    ```py
    qc.measure(0, 0)
    ```

6. Execute the circuit once on the qasm_simulator, a simulator that mimics a quantum computer. Then, get the results. 
    ```py
    simulator = Aer.get_backend('qasm_simulator')
    job = execute(qc, simulator, shots=1)

    result = job.result()
    counts = result.get_counts(qc)

    random_number = int(list(counts.keys())[0])
    print("Random number generated:", random_number)    
    ```

7. I wrapped them in a function and had a loop:

    ```py
    print("Press enter to generate a random number or type 'exit' to quit: ")
    while True:
        user_input = input()
        if user_input == 'exit':
            break
        else:
            generate_random_number()
    ```

8. Running this program, we have 
    ```
    Press enter to generate a random number or type 'exit' to quit: 
    Random number generated: 1

    Random number generated: 1

    Random number generated: 0

    Random number generated: 1

    Random number generated: 0

    Random number generated: 1

    Random number generated: 1

    Random number generated: 1
    ```

While this might seem that there is a higher probability of generating 1s, this too, is a characteristic of true randomness, in small number of trials, exhibiting patterns that might seem skewed.

With more trials, the distribution is expected to converge towards a more balanced outcome!

## Conclusion
The ability of QRNG has pretty profound implications for various fields, such as cryptography (especially Quantum Key Distribution) and many more. 

In essence, QRNGs unlock new possibilities in various tech domains, capitalising on the inherent randomness of quantum mechanics. These possibilities are likely to be transformative :D


## References
- How do two qubit states differing by a global phase relate to each other? (n.d.). Quantum Computing Stack Exchange. [https://quantumcomputing.stackexchange.com/questions/12448/](https://quantumcomputing.stackexchange.com/questions/12448/)
- Marinescu, D. C., & Marinescu, G. M. (2012). Classical and Quantum Information Theory. Elsevier. [https://www.sciencedirect.com/topics/mathematics/bloch-sphere](https://www.sciencedirect.com/topics/mathematics/bloch-sphere)
- Stoica, O. C. (2016). On the Wavefunction Collapse. Quanta, 5(1), 19. [https://doi.org/10.12743/quanta.v5i1.40](https://doi.org/10.12743/quanta.v5i1.40).