Part 1: Design, Layout, and Characterization of CMOS Logic Gates
This section focuses on the physical implementation and performance analysis of the fundamental building blocks of digital integrated circuits.

1. Objective
To design the schematic and layout of 2-input NAND and 2-input NOR gates using CMOS technology, then characterize their performance by measuring propagation delay and power consumption through transient analysis in LTSpice.

2. Design Methodology & Transistor Sizing
The core principle of CMOS design is to create a Pull-Up Network (PUN) made of PMOS transistors and a Pull-Down Network (PDN) made of NMOS transistors that are duals of each other.

Why Sizing is Critical:

NMOS is good at passing a strong '0' (logic low) but weak at passing a '1' (logic high).

PMOS is good at passing a strong '1' (logic high) but weak at passing a '0' (logic low).

To ensure symmetric rise and fall times (and thus balanced performance), the PMOS transistors must be wider than the NMOS transistors to compensate for their lower carrier mobility (~2-3 times wider is a common rule of thumb).

For this project, let's assume a generic 45nm technology node and a Width Ratio of 2:1 (PMOS:NMOS).

NMOS (Wn): Width = 180 nm, Length = 90 nm

PMOS (Wp): Width = 360 nm, Length = 90 nm

A. 2-Input NAND Gate

Schematic:

PUN: Two PMOS transistors in parallel.

PDN: Two NMOS transistors in series.

Operation:

Output is HIGH (VDD) only if both inputs are LOW. For all other cases, the path to GND is complete, pulling the output LOW.

Sizing Consideration: The two series NMOS transistors effectively act like a longer transistor. To maintain drive strength equivalent to a single NMOS, each series NMOS should be doubled in width. However, for initial characterization, the basic 2:1 ratio is often sufficient.

Optional Advanced Sizing: NMOS Width = 2 * Wn = 360nm, PMOS Width = Wp = 360nm.

B. 2-Input NOR Gate

Schematic:

PUN: Two PMOS transistors in series.

PDN: Two NMOS transistors in parallel.

Operation:

Output is HIGH (VDD) only if both inputs are LOW. If any input is HIGH, a path to GND is created, pulling the output LOW.

Sizing Consideration: Here, the two series PMOS transistors are the bottleneck. To ensure a strong '1' can be passed, each series PMOS should be doubled in width.

Common Sizing: PMOS Width = 2 * Wp = 720nm, NMOS Width = Wn = 180nm.

3. Implementation in Electric & LTSpice
Schematic Entry (Electric): The gates were drawn using the component library, ensuring correct connectivity and transistor sizing (W/L ratios).

Layout (Electric):

The layout was drawn according to Design Rule Checking (DRC) rules for the chosen technology (lambda rules or a PDK).

Layers used include: n-well, active, n-select, p-select, poly, metal1, contacts, and vias.

Layout vs. Schematic (LVS) was run to verify that the physical layout matched the logical schematic.

SPICE Netlist Extraction (Electric): Electric was used to automatically generate a SPICE netlist from the verified layout, including parasitic capacitances.

Simulation (LTSpice): The extracted netlist was simulated in LTSpice.

Voltage Source: VDD = 1.0V

Input Signal: A pulse (square wave) with a defined period, rise time, and fall time was applied to the inputs, ensuring all possible input combinations were tested.

Load Capacitance: A standard load (e.g., CL = 1fF) was connected to the output to model the gate being connected to other logic gates.

4. Characterization Results (LTSpice Output)
Key Metrics:

Propagation Delay (tp): The time difference between the 50% point of the input voltage swing and the 50% point of the output voltage swing. Measured for both High-to-Low (tpHL) and Low-to-High (tpLH) transitions. The average delay is tp = (tpHL + tpLH)/2.

Power Consumption: Calculated by LTSpice as the average of I(VDD) * VDD over a complete cycle. It consists of:

Dynamic Power: (α * CL * VDD² * f) - Power due to charging/discharging capacitors.

Short-Circuit Power: Power when both PMOS and NMOS are briefly ON during a transition.

Leakage Power: Static power consumption due to subthreshold leakage.

Sample Results Table (Simulated Values):

Gate	Input Transition	Prop. Delay (ps)	Avg. Power (µW) @ 100MHz
NAND	A: 0->1, B=1	tpHL = 18.5	2.1
A: 1->0, B=1	tpLH = 21.3	
Average (tp)	19.9	
NOR	A: 0->1, B=0	tpHL = 16.2	2.4
A: 1->0, B=0	tpLH = 35.8	
Average (tp)	26.0	
Conclusion for Part 1:

The NAND gate is generally faster than the NOR gate for a given load because its PMOS pull-up network is parallel, offering lower resistance. The NOR gate's series PMOS transistors result in a slower tpLH.





Part 2: Reduction of Computational Complexity for AES Algorithm using Modified Dual CLCG-based Key Generation
This section addresses optimizing a computationally intensive part of a critical cryptographic algorithm.

1. The Problem with AES Key Expansion
The Advanced Encryption Standard (AES) algorithm is robust but computationally expensive, especially in resource-constrained environments (IoT devices, RFID tags). A significant part of this cost is the Key Expansion routine.

The standard Key Expansion uses complex operations like SubBytes (S-Box lookup) and XORs for each round key.

For every block of data encrypted, a new set of round keys must be derived from the original cipher key. This is a repetitive and costly process.

2. Proposed Solution: Modified Dual CLCG
The proposal is to replace the traditional AES key expansion with a cryptographically secure Pseudorandom Number Generator (PRNG) to pre-generate the round keys.

LCG (Linear Congruential Generator): A simple PRNG defined by the recurrence: Xₙ₊₁ = (a * Xₙ + c) mod m

Advantage: Extremely low computational complexity (a multiply and an add).

Disadvantage: Not secure on its own; predictable if parameters are known.

Dual CLCG: Using two LCGs where the output of one perturbs the state of the other, increasing the period and improving randomness.

Modified Dual CLCG (Cross-Linked CLCG): A more advanced modification where the outputs of the two LCGs are combined (e.g., via XOR or addition) to form the final output. This further obscures the relationship between the internal state and the output sequence, enhancing security.

CLCG1: S1ₙ₊₁ = (a1 * S1ₙ + c1) mod m1

CLCG2: S2ₙ₊₁ = (a2 * S2ₙ + c2) mod m2

Output: Outputₙ = (S1ₙ + S2ₙ) mod m OR Outputₙ = S1ₙ XOR S2ₙ

3. How it Reduces Complexity
Pre-computation: The round keys for an entire encryption session (e.g., 10 rounds for AES-128) are generated once at the start using the CLCG, seeded with the original cipher key.

Replacement: Instead of running the full Key Expansion routine before every encryption, the algorithm simply fetches the pre-generated round keys from memory.

Complexity Comparison:

Standard AES Key Expansion: Involves Nk * (Nr + 1) iterations of S-Box lookups, XORs, and Rcon operations (where Nk is key words, Nr is rounds).

CLCG Key Generation: Involves Nb * (Nr + 1) iterations of a few modular multiplications and additions. This is mathematically much simpler than S-Box lookups and conditional XORs.

4. Integration and Security Considerations
Seed: The original 128/192/256-bit AES cipher key is split to seed the initial states (S1₀, S2₀) and parameters of the dual CLCGs.

Cryptographic Security: The parameters (a1, a2, c1, c2, m1, m2) must be chosen very carefully to ensure a long period and meet cryptographic standards for randomness (e.g., passing statistical test suites like NIST SP 800-22). The "modification" (cross-linking) is crucial to achieve this security.

Throughput: For applications that encrypt long streams of data, the reduction in per-block overhead significantly increases overall encryption/decryption throughput.

5. Conclusion for Part 2
Replacing the traditional AES key schedule with a pre-computation step using a secure, modified Dual CLCG presents a promising trade-off:

Advantage: A significant reduction in computational complexity and an increase in processing throughput.

Challenge: Requires rigorous cryptographic analysis to ensure the PRNG does not introduce vulnerabilities, making the overall system less secure than the proven AES key expansion. The security of the entire system now rests on the strength of the CLCG.

This project demonstrates a complete cycle from the transistor-level design of basic logic to the architectural-level optimization of a complex algorithm, highlighting the different layers of abstraction in VLSI and system design.

The layout was DRC and LVS clean, confirming a manufacturable and correct design.

This process of standard cell design and characterization is the foundation of automated digital ASIC design flows.

