## ASSIGNMENT 5
### Q1)Write a python function to realizing the logical AND function through Hebb learning.

Code:
```python
# Training data (bipolar)
inputs = [
    (1, 1),
    (1, -1),
    (-1, 1),
    (-1, -1)
]

targets = [1, -1, -1, -1]

# Initial weights and bias
w1 = 0
w2 = 0
b = 0

print("Training Start:\n")

# Training using Hebb rule
for i in range(len(inputs)):
    x1, x2 = inputs[i]
    y = targets[i]

    # Using your formulas
    delta_w1 = x1 * y      # Δw1 = x1 * y
    delta_w2 = x2 * y      # Δw2 = x2 * y
    delta_b = y            # Δb = y

    w1 = w1 + delta_w1     # w1(new) = w1(old) + Δw1
    w2 = w2 + delta_w2     # w2(new) = w2(old) + Δw2
    b = b + delta_b        # b(new) = b(old) + Δb

    print(f"Step {i+1}: w1={w1}, w2={w2}, b={b}")

print("\nFinal Weights:")
print("w1 =", w1)
print("w2 =", w2)
print("b  =", b)

# User input
print("\nEnter input values (only 1 or -1):")
x1 = int(input("Enter x1: "))
x2 = int(input("Enter x2: "))

# Net calculation
net = x1 * w1 + x2 * w2 + b

# Activation (sign function)
if net >= 0:
    output = 1
else:
    output = -1

print("\nOutput (Target Value):", output)
```

### Output:
<img width="352" height="372" alt="image" src="https://github.com/user-attachments/assets/f5d6d7c3-3952-444a-9735-c6c53cb70af8" />


### Q2) Write a python function to realizing the logical AND function by a perceptron.

Code:
```python
inputs = [
    (1, 1),
    (1, -1),
    (-1, 1),
    (-1, -1)
]

targets = [1, -1, -1, -1]

# Initial weights
w1 = 0
w2 = 0
b = 0

alpha = 1

print("EPOCH 1:\n")

# Training
for i in range(4):
    x1, x2 = inputs[i]
    t = targets[i]

    # Net input
    y_in = b + x1*w1 + x2*w2

    # Activation
    if y_in > 0:
        y = 1
    elif y_in == 0:
        y = 0
    else:
        y = -1

    # Update only if incorrect
    if y != t:
        delta_w1 = alpha * t * x1
        delta_w2 = alpha * t * x2
        delta_b = alpha * t

        w1 = w1 + delta_w1
        w2 = w2 + delta_w2
        b = b + delta_b

    print(f"Row {i+1} -> w1={w1}, w2={w2}, b={b}")

print("\nFinal Weights:")
print("w1 =", w1, "w2 =", w2, "b =", b)

# ------------------ USER TESTING ------------------

print("\nEnter input (only 1 or -1):")
x1 = int(input("Enter x1: "))
x2 = int(input("Enter x2: "))

# Net input for testing
y_in = b + x1*w1 + x2*w2

# Activation
if y_in > 0:
    output = 1
elif y_in == 0:
    output = 0
else:
    output = -1

print("\nFinal Output:", output)
```
### Output:
<img width="290" height="324" alt="image" src="https://github.com/user-attachments/assets/70bf41ee-bcc6-438f-b7ed-3ac6c572e63c" />


## Assignments-6
Elementary Search Techniques (Travelling Salesperson Problem)

Code:
```python
import random
nodes_list = ['a', 'b', 'c', 'd', 'e']

reward_matrix = [
  #  a  b   c   d   e
    [0, 43, 0, 39, 44],   # a
    [43, 0, 44, 45, 35],  # b
    [0, 44, 0, 42, 48],   # c
    [39, 45, 42, 0, 23],  # d
    [44, 35, 48, 23, 0]   # e
]

binary_to_node = {
  "000": 0, # a
  "001": 1, # b
  "010": 2, # c
  "011": 3, # d
  "100": 4  # e
}

def decode_chromosome(chromosome_str):
    """
    Decodes a 15-bit binary string into a valid TSP tour of 5 nodes
    using the sequential repair logic from Fig 12.7.
    """

    # Track visited status of nodes 0 to 4, because initially all nodes are not visited, something like this:
    # visited = { 'a': False, 'b': False, 'c': False, 'd': False, 'e': False }

    visited = [False] * 5
    tour = []

    # Split the 15-bit chromosome into five 3-bit chunks, total 5 chunks
    chunks = [chromosome_str[i:i+3] for i in range(0, 15, 3)]

    for chunk in chunks:
        node = binary_to_node.get(chunk, None)

        # If it's a valid node code and hasn't been visited yet
        if node is not None and not visited[node]:
            tour.append(node)
            visited[node] = True
        else:
            # Code is invalid OR node is already visited.
            # Select the next available unvisited node in circular order (a -> b -> c -> d -> e)
            for offset in range(5):
                # If the current chunk was valid, start searching from it, else start from 0
                start_node = node if node is not None else 0
                candidate = (start_node + offset) % 5
                if not visited[candidate]:
                    tour.append(candidate)
                    visited[candidate] = True
                    break

    return tour




def calculate_fitness(tour):
    """
    Calculates total reward of a tour.
    Includes the returning edge to make it a circular tour (cycle).
    That is why the decoded_letters[0] is added to the end of the tour.
    """
    total_reward = 0
    num_nodes = len(tour)

    for i in range(num_nodes):
        current_node = tour[i]
        next_node = tour[(i + 1) % num_nodes] # Circular loop back to start
        total_reward += reward_matrix[current_node][next_node]

    return total_reward

def generate_initial_population(pop_size=10):
    population = []
    for i in range(pop_size):
        # Generate a random 15-bit binary string
        chromosome = "".join(str(random.randint(0, 1)) for _ in range(15))
        #     return population
        tour = decode_chromosome(str(chromosome))
        actualtour = "-".join(nodes_list[i] for i in tour)
        fitness = calculate_fitness(tour)
        population.append((chromosome, actualtour, fitness))
    return population

if __name__ == "__main__":
    # Test text book example: chr = 101 011 001 110 001
    # According to textbook, this must decode to: a -> d -> b -> c -> e

    book_chromosome = "101011001110001"
    decoded_indices = decode_chromosome(book_chromosome)
    decoded_letters = [nodes_list[idx] for idx in decoded_indices]
    fitness_val = calculate_fitness(decoded_indices)

    print("--- Textbook TSP implementation ---\n")
    print(f"Chromosome example from book: {book_chromosome}")
    print(f"Decoded Tour: {' -> '.join(decoded_letters)} -> {decoded_letters[0]}")
    print(f"Calculated Fitness (Reward): {fitness_val} (Expected: 222)")
    # print(generate_initial_population())

from tabulate import tabulate

headers = ["Chromosome", "Tour", "Fitness"]

# Uses 'grid' style for a bordered table
print(tabulate(generate_initial_population(), headers=headers, tablefmt="grid"))
print("-" * 38 + "\n")
```

### Output:

<img width="532" height="626" alt="image" src="https://github.com/user-attachments/assets/da89ee27-77d5-4b9b-a65f-ecf9da656ee4" />
