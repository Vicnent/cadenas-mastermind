# cadenas-mastermind
generate 4 digits mastermind solution

### Explanation of the Mastermind Puzzle Algorithm

This Python script implements an algorithm to generate and solve a Mastermind-like puzzle, where a 4-digit code (with distinct digits, the first being non-zero) is created, and a set of clues is provided to uniquely identify the solution. The code is designed to be clear, modular, and suitable for sharing on GitHub. 
---

## Overview

The script simulates a Mastermind game where:
- A **secret code** (solution) is a 4-digit number with distinct digits (e.g., "1234"), where the first digit is not zero.
- **Clues** are generated as guesses with feedback in the form of "well-placed" (correct digit in the correct position) and "misplaced" (correct digit in the wrong position) counts.
- The algorithm ensures that the set of clues (default: 6) uniquely identifies the secret code by checking all possible 4-digit combinations.
- The output is formatted as a pandas DataFrame for a clean presentation.

The script uses random generation, permutation checking, and constraint-based filtering to create a puzzle with exactly one solution.

---

## Detailed Breakdown of the Code

### 1. **Dependencies**
```python
import random
from itertools import permutations
import pandas as pd
```
- **`random`**: Used to generate random 4-digit codes for the solution and guesses.
- **`itertools.permutations`**: Generates all possible 4-digit combinations for solution validation.
- **`pandas`**: Formats the puzzle output as a DataFrame for readable display.

---

### 2. **Function: `generate_solution()`**
```python
def generate_solution():
    while True:
        combi = ''.join(random.choices('123456789', k=1) + random.choices('0123456789', k=3))
        if len(set(combi)) == 4:
            return combi
```
- **Purpose**: Generates a random 4-digit code with distinct digits, where the first digit is non-zero (1–9).
- **How it works**:
  - Randomly selects the first digit from `'123456789'` (ensuring non-zero).
  - Selects three more digits from `'0123456789'`.
  - Concatenates them into a string (e.g., "1234").
  - Checks if all digits are unique using `set(combi)`. If not, repeats until a valid code is found.
- **Output**: A string like `"1234"`.

---

### 3. **Function: `count_well_and_misplaced(solution, guess)`**
```python
def count_well_and_misplaced(solution, guess):
    well_placed = sum(s == g for s, g in zip(solution, guess))
    remaining_sol = [s for s, g in zip(solution, guess) if s != g]
    remaining_guess = [g for s, g in zip(solution, guess) if s != g]
    misplaced = 0
    for g in remaining_guess:
        if g in remaining_sol:
            misplaced += 1
            remaining_sol.remove(g)
    return well_placed, misplaced
```
- **Purpose**: Computes the number of **well-placed** and **misplaced** digits when comparing a guess to the solution.
- **How it works**:
  - **Well-placed**: Counts digits that match in both value and position (e.g., if solution is "1234" and guess is "1243", positions 1 and 2 match, so `well_placed = 2`).
  - **Misplaced**: For digits not well-placed, checks if a guess digit appears in the solution at a different position. Removes matched digits to avoid double-counting (e.g., for "1234" vs. "1243", "3" and "4" are misplaced, so `misplaced = 2`).
- **Output**: A tuple `(well_placed, misplaced)`, e.g., `(2, 2)`.

---

### 4. **Function: `generate_random_guess(solution)`**
```python
def generate_random_guess(solution):
    while True:
        guess = ''.join(random.choices('123456789', k=1) + random.choices('0123456789', k=3))
        if guess != solution and len(set(guess)) == 4:
            return guess
```
- **Purpose**: Generates a random 4-digit guess distinct from the solution, with unique digits and a non-zero first digit.
- **How it works**:
  - Similar to `generate_solution()`, but ensures the guess is not the solution (`guess != solution`).
  - Repeats until a valid guess is found.
- **Output**: A string like `"5678"`.

---

### 5. **Function: `phrase_from_counts(guess, wp, mp)`**
```python
def phrase_from_counts(guess, wp, mp):
    if wp == 0 and mp == 0:
        return f"Avec {guess}, rien d'intéressant."
    parts = []
    if wp == 0:
        parts.append("")
    elif wp == 1:
        parts.append("1 chiffre bien placé")
    else:
        parts.append(f"{wp} chiffres bien placés")
    if mp == 0:
        parts.append("")
    elif mp == 1:
        parts.append("1 chiffre mal placé")
    else:
        parts.append(f"{mp} chiffres mal placés")
    return f"Avec {guess}, {', '.join(parts)}."
```
- **Purpose**: Formats a clue as a human-readable string based on the guess and its well-placed (`wp`) and misplaced (`mp`) counts.
- **How it works**:
  - If both `wp` and `mp` are 0, returns a message indicating no useful information.
  - Builds a list of phrases for well-placed and misplaced digits, handling singular/plural cases (e.g., "1 chiffre" vs. "2 chiffres").
  - Joins phrases with commas and includes the guess.
- **Output**: A string like `"Avec 5678, 1 chiffre bien placé, 2 chiffres mal placés."`.

---

### 6. **Function: `generate_puzzle(solution, num_clues=4)`**
```python
def generate_puzzle(solution, num_clues=4):
    clues = []
    guesses = set()
    attempts = 0
    while len(clues) < num_clues and attempts < 1000:
        attempts += 1
        guess = generate_random_guess(solution)
        if guess in guesses:
            continue
        guesses.add(guess)
        wp, mp = count_well_and_misplaced(solution, guess)
        # Contraintes imposées
        if wp == 0 and mp > 2:
            continue
        if mp == 0 and wp > 2:
            continue
        if wp + mp > 3:
            continue
        phrase = phrase_from_counts(guess, wp, mp)
        clues.append((guess, wp, mp, phrase))
    return clues
```
- **Purpose**: Generates a list of clues (guesses with their well-placed and misplaced counts) for the given solution.
- **How it works**:
  - Iterates until `num_clues` (default: 4) valid clues are generated or 1000 attempts are made.
  - Generates a unique guess using `generate_random_guess()`.
  - Skips guesses already used (tracked in `guesses` set).
  - Computes `wp` and `mp` for the guess.
  - Applies constraints to filter guesses:
    - Excludes guesses with `wp = 0` and `mp > 2`.
    - Excludes guesses with `mp = 0` and `wp > 2`.
    - Excludes guesses where `wp + mp > 3`.
  - Formats each valid guess into a clue using `phrase_from_counts()`.
  - Stores each clue as a tuple `(guess, wp, mp, phrase)`.
- **Output**: A list of tuples, e.g., `[("5678", 1, 2, "Avec 5678, 1 chiffre bien placé, 2 chiffres mal placés."), ...]`.

---

### 7. **Function: `find_possible_solutions(clues)`**
```python
def find_possible_solutions(clues):
    all_combinations = [''.join(p) for p in permutations('0123456789', 4) if p[0] != '0']
    valid = []
    for combi in all_combinations:
        if all(count_well_and_misplaced(combi, guess) == (wp, mp) for guess, wp, mp, _ in clues):
            valid.append(combi)
    return valid
```
- **Purpose**: Identifies all 4-digit codes that satisfy the given clues.
- **How it works**:
  - Generates all possible 4-digit combinations with distinct digits and non-zero first digit using `permutations('0123456789', 4)`.
  - Filters out combinations starting with '0'.
  - For each combination, checks if it satisfies all clues by comparing `count_well_and_misplaced(combi, guess)` with the clue's `(wp, mp)`.
  - Collects valid combinations in the `valid` list.
- **Output**: A list of valid codes, e.g., `["1234"]`.

---

### 8. **Main Loop: Puzzle Generation and Validation**
```python
while True:
    solution = generate_solution()
    clues = generate_puzzle(solution, num_clues=6)
    candidates = find_possible_solutions(clues)
    if len(candidates) == 1:
        result = [f"✅ Solution : {solution}"]
        for _, _, _, phrase in clues:
            result.append(phrase)
        break
```
- **Purpose**: Generates a puzzle with exactly one solution.
- **How it works**:
  - Repeatedly generates a random solution and its clues (default: 6 clues).
  - Checks if the clues uniquely identify the solution by calling `find_possible_solutions()`.
  - If exactly one candidate is found (`len(candidates) == 1`), constructs the result list with the solution and clue phrases.
  - Breaks the loop when a valid puzzle is found.
- **Output**: A list like `["✅ Solution : 1234", "Avec 5678, 1 chiffre bien placé, 2 chiffres mal placés.", ...]`.

---

### 9. **Output Formatting**
```python
df = pd.DataFrame(result, columns=["Énigme"])
print(df.to_string(index=False, justify='left'))
```
- **Purpose**: Displays the puzzle in a clean, tabular format.
- **How it works**:
  - Creates a pandas DataFrame with the result list under the column "Énigme".
  - Uses `to_string(index=False, justify='left')` to print without indices and with left-aligned text.
- **Output Example**:
  ```
  Énigme
  ✅ Solution : 1234
  Avec 5678, 1 chiffre bien placé, 2 chiffres mal placés.
  Avec 9012, 0 chiffres bien placés, 2 chiffres mal placés.
  Avec 3456, 2 chiffres bien placés, 0 chiffres mal placés.
  Avec 7890, 0 chiffres bien placés, 1 chiffre mal placé.
  Avec 2345, 2 chiffres bien placés, 1 chiffre mal placé.
  Avec 6789, 1 chiffre bien placé, 1 chiffre mal placé.
  ```

---

## Algorithm Flow

1. **Generate a solution**: Create a random 4-digit code with unique digits (e.g., "1234").
2. **Generate clues**: Produce 6 guesses with constraints to ensure varied and solvable clues.
3. **Validate uniqueness**: Check if the clues uniquely identify the solution by testing all possible 4-digit codes.
4. **Repeat if necessary**: If multiple or no solutions are found, restart with a new solution and clues.
5. **Output**: Present the solution and clues in a readable DataFrame.

---

## Key Features

- **Randomized Puzzle Generation**: Ensures varied puzzles each run.
- **Constraint-Based Clues**: Filters guesses to avoid trivial or overly complex clues, enhancing solvability.
- **Uniqueness Guarantee**: Ensures the puzzle has exactly one solution.
- **Readable Output**: Uses pandas for clean presentation, suitable for sharing or logging.
- **Modular Design**: Functions are well-separated, making the code reusable and maintainable.

---

## Potential Improvements

- **Customizable Constraints**: Allow users to modify clue constraints (e.g., allow `wp + mp > 3`).
- **Optimization**: Use more efficient algorithms for `find_possible_solutions()` (e.g., constraint propagation instead of brute-force permutation).
- **Interactive Mode**: Add a user interface to input guesses and get feedback.
- **Error Handling**: Add checks for edge cases, like failure to generate clues within 1000 attempts.

---

## Usage

To run the script:
1. Ensure Python 3.x and pandas are installed (`pip install pandas`).
2. Copy the code into a `.py` file or Jupyter notebook.
3. Execute to generate a unique Mastermind puzzle.

This algorithm is ideal for educational purposes, puzzle enthusiasts, or as a foundation for more complex game implementations.

---

```python
import random
from itertools import permutations
import pandas as pd

def generate_solution():
    while True:
        combi = ''.join(random.choices('123456789', k=1) + random.choices('0123456789', k=3))
        if len(set(combi)) == 4:
            return combi

def count_well_and_misplaced(solution, guess):
    well_placed = sum(s == g for s, g in zip(solution, guess))
    remaining_sol = [s for s, g in zip(solution, guess) if s != g]
    remaining_guess = [g for s, g in zip(solution, guess) if s != g]
    misplaced = 0
    for g in remaining_guess:
        if g in remaining_sol:
            misplaced += 1
            remaining_sol.remove(g)
    return well_placed, misplaced

def generate_random_guess(solution):
    while True:
        guess = ''.join(random.choices('123456789', k=1) + random.choices('0123456789', k=3))
        if guess != solution and len(set(guess)) == 4:
            return guess

def phrase_from_counts(guess, wp, mp):
    if wp == 0 and mp == 0:
        return f"Avec {guess}, rien d'intéressant."
    parts = []
    if wp == 0:
        parts.append("")
    elif wp == 1:
        parts.append("1 chiffre bien placé")
    else:
        parts.append(f"{wp} chiffres bien placés")
    if mp == 0:
        parts.append("")
    elif mp == 1:
        parts.append("1 chiffre mal placé")
    else:
        parts.append(f"{mp} chiffres mal placés")
    return f"Avec {guess}, {', '.join(parts)}."

def generate_puzzle(solution, num_clues=4):
    clues = []
    guesses = set()
    attempts = 0
    while len(clues) < num_clues and attempts < 1000:
        attempts += 1
        guess = generate_random_guess(solution)
        if guess in guesses:
            continue
        guesses.add(guess)
        wp, mp = count_well_and_misplaced(solution, guess)
        if wp == 0 and mp > 2:
            continue
        if mp == 0 and wp > 2:
            continue
        if wp + mp > 3:
            continue
        phrase = phrase_from_counts(guess, wp, mp)
        clues.append((guess, wp, mp, phrase))
    return clues

def find_possible_solutions(clues):
    all_combinations = [''.join(p) for p in permutations('0123456789', 4) if p[0] != '0']
    valid = []
    for combi in all_combinations:
        if all(count_well_and_misplaced(combi, guess) == (wp, mp) for guess, wp, mp, _ in clues):
            valid.append(combi)
    return valid

while True:
    solution = generate_solution()
    clues = generate_puzzle(solution, num_clues=6)
    candidates = find_possible_solutions(clues)
    if len(candidates) == 1:
        result = [f"✅ Solution : {solution}"]
        for _, _, _, phrase in clues:
            result.append(phrase)
        break

df = pd.DataFrame(result, columns=["Énigme"])
print(df.to_string(index=False, justify='left'))
```
