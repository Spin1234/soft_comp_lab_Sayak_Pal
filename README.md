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

Output:
--- Textbook TSP implementation ---

**Chromosome example from book:** 101011001110001
**Decoded Tour:** a -> d -> b -> c -> e -> a
**Calculated Fitness (Reward):** 220 (Expected: 222)

| Chromosome       | Tour      | Fitness |
|------------------|-----------|---------|
| 100011010111010  | e-d-c-a-b | 143     |
| 001001000110101  | b-c-a-d-e | 141     |
| 110010011000110  | a-c-d-b-e | 166     |
| 000111011110100  | a-b-d-c-e | 222     |
| 010110110101001  | c-a-b-d-e | 159     |
| 110010000011110  | a-c-b-d-e | 156     |
| 100001001000101  | e-b-c-a-d | 141     |
| 010101000010110  | c-a-b-d-e | 159     |
| 011000001011010  | d-a-b-e-c | 207     |
| 010101011111111  | c-a-d-b-e | 167     |

---
