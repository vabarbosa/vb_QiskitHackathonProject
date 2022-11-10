Welcome to our project solving the LCS problem using Qubits.

We used a paper that claimed to say that this algorithm for the LCS problem was actually a speed up to the classical solution

The main steps of the algorithm are summarized in the following 4 lines
1. Create the registers 
2. Shift the register
3. XOR the registers
4. Diffuse the result (Grovers)

## Creating the Registers       
Given two strings, a text string T which we are searching through, and a pattern string P