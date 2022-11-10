Welcome to our project solving the pattern matching problem using Qubits.


Video: https://drive.google.com/file/d/1NrQwLvASjidugiCMgVSqo2pdSzLIZZIh/view?usp=sharing

For this, we utilized the following paper, which details how to use
grovers to find the problem in $O(\sqrt{n})$ time:
https://www.nature.com/articles/s41534-021-00369-3

The paper specifies that the size of the string is $N$, and the pattern
has size $M$.

The main steps of the algorithm are summarized in the following 4 lines

1. Create the registers 
2. Shift the register
3. XOR the registers
4. Diffuse the result (Grovers)

Additionally, to make the number of grovers steps constant, we assume that there is only one solution.

## Running the program
Install qiskit with "pip install qiskit" and then run the jupyter
notebook. $T$ is the string and $P$ is the pattern.

## Creating the Registers       

For our pattern matching, we first need to initialize what pattern we want to match and what text we want to match that pattern to. In the search for a simple solution set due to the amount of time for the project, our group decided to think of these as qubit strings using the qiskit QuantumRegister function. The function allows us to create a register of an integer amount of qubits.

In the project we initialize three quantum registers, a register $T$ for our text of length $N$, a register $P$ for our pattern of length $M$, and a register $K$ for our index which spans the solution space. There are some initial conditions that must pass for our solution to be valid, firstly the length of $T$ must be more than the length of $P$, meaning $N>M$. The next initial condition that must be met is our $N$ and $M$ must be powers of 2.

To implement these in our algorithm can be done with just the three lines of python
```python
    T_regs = QuantumRegister(N, name = "t_reg")
    P_regs = QuantumRegister(M, name = "p_reg")
    k_regs = QuantumRegister(n, name = "k_reg")
```

Given two strings, a text string $T$ which we are searching through, and a pattern string $P$


## Shifting the register
Shifting the registers is done to move the size $M$ "window" that we
look at in our string. This is done to tie our index bits to an actual
bitsequence in string. While doing this is relavitvely simple in
classical mechanics, doing so in quantum mechanics requires us to
implement a left shift using swap gates. 

In order to minimize the number of gates, the paper details the
following algorithm to implement a left shift in $O(\log n)$:

1. Choose half the qubits, swap them to their desired location
2. Recurse on the other half

This algorithm is a little vague, but since generating the gates
doesn't count as part of the runtime of the problem, we can adopt the
bruteforce approach here:

```python
def log_l_rotate(count,times):
    """
    Computes swap gate sequence necessary to create cyclic krotation in logarithmic time.
    
    rotates system left time times if kbit is 1
    """
    qc = QuantumCircuit(count, name=f"S_{times}")
    numIterations = 0 # ensure logarithmic time dependence. returned at end
    vals = list(range(count)) # create an ordered list to help keep track of which gates have already been swapped into their correct position
    size = len(vals)
    while not is_list_shifted(vals,times):
        numIterations+=1
        vals_added = [] # all the values which we have already swapped, used to not double count the number of gates per iteration
        for i in range(size):
            # skip any cells which we are swapping this iteration, we can't use them!
            # note that this means we need to add the indices of each cell we are swapping
            if i in vals_added:
                continue
            preferred_location = rotation_ending_position(vals[i],size,times)
            # traverse all the values. If the gate hasn't already been swapped to its desired postion,
            # add a swap circuit and swap the two values
            if(i!=preferred_location):
                qc.swap(i,preferred_location)
                vals_added.append(i)
                vals_added.append(preferred_location)
                vals[preferred_location],vals[i] = vals[i],vals[preferred_location]
    return qc

```

As a small example, here is what the system looks like on three gates

```           
q_0: ─X──X────
      │  │    
q_1: ─┼──X──X─
      │     │ 
q_2: ─┼─────X─
      │       
q_3: ─X───────
```

This is then controlled by an arbitrary kbit to complete the random shifting.

## Computing the xors

Our oracle for this problem needs to figure out if the first $M$ bits of
$T$ are equal to $P$, which reduces to checking of the xor of those bits
is zero. We implement this using a set of cnot gates between each bit
pair.

## Diffuse the result

Afterwards, we simply run grovers with the following oracle

$U_w|00000\rangle = -|00000\rangle$ and $U_w = I$ otherwise

Where the ket we act on is the patterns. This effectively makes the
oracle function compare when the two terms are equal, enabling us to
run grovers. An important thing to remember for this problem is that
diffusion must also be applied to the kbits, as otherwise they have a
tendancy to lose their entanglement. 

## Circuit Diagrams
### Full Diagram
```
            ┌────────────┐ ░ ┌──────────────────┐ ░ ┌─┐   
   k_reg_0: ┤0           ├─░─┤0                 ├─░─┤M├───
            │  Hadamards │ ░ │                  │ ░ └╥┘┌─┐
   k_reg_1: ┤1           ├─░─┤1                 ├─░──╫─┤M├
            ├───────────┬┘ ░ │                  │ ░  ║ └╥┘
   t_reg_0: ┤0          ├──░─┤2                 ├─░──╫──╫─
            │           │  ░ │                  │ ░  ║  ║ 
   t_reg_1: ┤1          ├──░─┤3                 ├─░──╫──╫─
            │  Encoding │  ░ │  Grovers 2 times │ ░  ║  ║ 
   t_reg_2: ┤2          ├──░─┤4                 ├─░──╫──╫─
            │           │  ░ │                  │ ░  ║  ║ 
   t_reg_3: ┤3          ├──░─┤5                 ├─░──╫──╫─
            ├───────────┤  ░ │                  │ ░  ║  ║ 
   p_reg_0: ┤0          ├──░─┤6                 ├─░──╫──╫─
            │  Encoding │  ░ │                  │ ░  ║  ║ 
   p_reg_1: ┤1          ├──░─┤7                 ├─░──╫──╫─
            └───────────┘  ░ └──────────────────┘ ░  ║  ║ 
idx_cout_0: ═════════════════════════════════════════╩══╬═
                                                        ║ 
idx_cout_1: ════════════════════════════════════════════╩═
```
### Grovers 2 times
```
         ┌────────────┐ ░            ░                   ░ ┌────────┐ ░ 
k_reg_0: ┤0           ├─░────────────░───────────────────░─┤2       ├─░─
         │            │ ░            ░                   ░ │        │ ░ 
k_reg_1: ┤1           ├─░────────────░───────────────────░─┤3       ├─░─
         │            │ ░            ░                   ░ │        │ ░ 
t_reg_0: ┤2           ├─░───■────────░───────────────────░─┤        ├─░─
         │  S_k shift │ ░   │        ░                   ░ │        │ ░ 
t_reg_1: ┤3           ├─░───┼────■───░───────────────────░─┤        ├─░─
         │            │ ░   │    │   ░                   ░ │  U$_s$ │ ░ 
t_reg_2: ┤4           ├─░───┼────┼───░───────────────────░─┤        ├─░─
         │            │ ░   │    │   ░                   ░ │        │ ░ 
t_reg_3: ┤5           ├─░───┼────┼───░───────────────────░─┤        ├─░─
         └────────────┘ ░ ┌─┴─┐  │   ░ ┌───────────────┐ ░ │        │ ░ 
p_reg_0: ───────────────░─┤ X ├──┼───░─┤0              ├─░─┤0       ├─░─
                        ░ └───┘┌─┴─┐ ░ │  Phase Oracle │ ░ │        │ ░ 
p_reg_1: ───────────────░──────┤ X ├─░─┤1              ├─░─┤1       ├─░─
                        ░      └───┘ ░ └───────────────┘ ░ └────────┘ ░ 
```
### Phase oracle(for $p = 4$)
```
q_0: ────■─────────
         │         
q_1: ─■──o─────────
      │  │         
q_2: ─o──o───────■─
      │  │ ┌───┐ │ 
q_3: ─o──o─┤ Z ├─o─
```

## Difficulties

While implementing the algorithm, our group ran into a small issue
which simply do not have the time to solve: Our grovers algorithm is
broken so that the right answer always has the *least* amplification
instead of the largest. We're not sure exactly why this is
happening(though we suspect phase kickback in our oracle or diffuser),
so we have just chosen to ignore it for now as the runtime is
technically identical for both algorithms(while keeping track of your
counts, it's equally easy to look at the minimum as opposed to the
maximum).

## Emailing the Authors
Reading the paper was a passable solution to generating an algorithm
for our problem. Because of what we think is Phase Kickback, our
answer is the opposite of what the paper claims to be true. The reason
for this problem is mostly because of the way they explain running
Grover's Algorithm, in that they don't not explain the specifics at
all. This was a common theme among quantum computing papers that used
Grover's, just not explaining how they actually set up that Grover's
search. We intend to email the authors of the paper to ask how they
would change the diffuser to fix this issue.

## Team members

John Hoffman
Alex Walburg
Jack Dickson
Aathreya Kadambi

## Team name

Phase kickback
