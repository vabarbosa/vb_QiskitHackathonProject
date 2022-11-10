Welcome to our project solving the pattern matching problem using Qubits.

We used a paper that claimed to say that this algorithm for the pattern matching problem was actually a speed up to the classical solution

The main steps of the algorithm are summarized in the following 4 lines
1. Create the registers 
2. Shift the register
3. XOR the registers
4. Diffuse the result (Grovers)

## Creating the Registers       
For our pattern matching, we first need to initialize what pattern we want to match and what text we want to match that pattern to. In the search for a simple solution set due to the amount of time for the project, our group decided to think of these as qubit strings using the qiskit QuantumRegister function. The function allows us to create a register of an integer amount of qubits.

In the project we initialize three quantum registers, a register T for our text of length N, a register P for our pattern of length M, and a register K for our index which spans the solution space. There are some initial conditions that must pass for our solution to be valid, firstly the length of T must be more than the length of P, meaning N>M. The next initial condition that must be met is our N and M must be powers of 2.

To implement these in our algorithm can be done with just the three lines of python
```python
    T_regs = QuantumRegister(N, name = "t_reg")
    P_regs = QuantumRegister(M, name = "p_reg")
    k_regs = QuantumRegister(n, name = "k_reg")
```