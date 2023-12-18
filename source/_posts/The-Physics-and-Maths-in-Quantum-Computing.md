---
title: The Physics and Maths in Quantum Computing
date: 2023-12-18 09:39:56
categories: Quantum
tags:
  - Quantum
---

## Introduction

Quantum computing, as mentioned in my previous posts, has been a huge buzzword in the tech industry due to its potential to revolutionise some industries. But how exactly does it work? In this post, I will attempt to explain the physics and maths behind quantum computing in a simple manner.

PS: This post is still a work in progress, I'll be adding more content soon!

## Classical vs Quantum Computing

Before we dive headfirst into the concepts, let us understand the properties of classical and quantum computing.

In classical computing, information is processed using bits. In quantum computing, information is processed using quantum bits, or qubits.

### Properties

**Bits**

- **States**: Can exist in one of two states: 0 or 1
- **Discrete**: Cannot be in between 0 and 1
- **Deterministic**: Given the same conditions and inputs, the outcome will always be the same
- **Measurement**: States are measured using physical systems that have two distinct states, like voltage or current, where 0 is represented by the absence of it, and 1 is represented by the presence.
  ![Bits Measurement](../img/quantumphymath/bits.png)

**Qubits**

- **States**: Can exist in multiple states simultaneously, known as superposition.
  - Combination of both 0 and 1, denoted as |0⟩ and |1⟩. The qubit can exist in a state α|0⟩ + β|1⟩, where α and β are complex numbers.
- **Continuous**: Can be in between 0 and 1
- **Probabilistic**: Given the same conditions, the outcome will not always be the same
- **Measurement**: States are measured using quantum properties such as:
  - **Superconducting Qubits**: The direction of the current in a superconducting (zero resistance) loop
  - **Trapped Ion Qubits**: The energy levels of a trapped ion, manipulated using lasers
  - **Photonic Qubits**: The polarisation of a photon (horizontal or vertical)

## Quantum Mechanics

Now that we have a basic understanding of the differences between classical and quantum computing, we can move on to the exact physics behind quantum computing.

### Wave-Particle Duality

Wave-particle duality is a concept that describes the behaviour of particles, such as electrons and photons. It states that these particles can behave like both a wave and a particle, depending on the situation (the context in which they are observed or measured).

**Wave**

- **Interference**: Waves can interfere with each other, either constructively (when the waves are in phase) or destructively (when the waves are out of phase).
  ![Wave Interference](../img/quantumphymath/waveinterference.png)
- **Diffraction**: Waves can diffract around obstacles, causing them to spread out.
- **Superposition**: Waves can exist in multiple states simultaneously, known as superposition.
  ![Wave Superposition](../img/quantumphymath/superposition.jpg)
- **Wavelength and Frequency**: Waves have a wavelength and frequency, which are inversely proportional to each other.

**Particle**

- **Quantisation**: Particles can only exist in discrete energy levels
- **Localisation**: Particles can be localised to a specific position, meaning that they can be found at a specific point in space
- **Trajectory**: Particles have a trajectory, which can be determined using Newton's laws of motion

In essence, this duality allows quantum particles to behave like waves and particles at the same time. The dual nature is fundamental in quantum computing, with the wave-like properties being used to represent the superposition of qubits, and the particle-like properties being used to measure the qubits.

### Quantum States and Measurement

I mentioned particle-like properties are used to measure qubits. But why? When a qubit is measured, it disrupts the quantum system, bringing about the collapse of the wavefunction. Hence, its state is determined and forced into either |0⟩ or |1⟩. This is known as the **collapse of the wavefunction**.

Remember the Schrödinger's cat thought experiment? The cat inside a sealed box can be considered to be "both alive and dead", until the box is opened and the cat is observed. This is similar to the collapse of the wavefunction, where the state is determined only when the box is opened.

### Entanglement

## Quantum Gates and Circuits
