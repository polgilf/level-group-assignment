# Level-Balanced Student Group Assignment Optimizer

This project uses the OR-Tools library to solve a student group assignment problem. The main objective is to create groups of students while minimizing the number of repeated pairings and balancing the levels of students within groups.

## Table of Contents
- [Requirements](#requirements)
- [Usage](#usage)
- [Input Data](#input-data)
- [Output](#output)
- [Example Usage](#example-usage)
- [MILP Formulation](#milp-formulation)

## Requirements

- Python 3.x
- pandas
- OR-Tools

Install the required packages using pip:

```
pip install -r requirements.txt
```

## Usage

1. Prepare your input data in an Excel file following the format of `students_data_example.xlsx`.
2. Open `main.py` and adjust the parameters at the top of the file:
   - `session`: The session to solve. Should match a column name in the 'attendance_list' sheet.
   - `number_of_groups_per_size`: A dictionary specifying the number of groups for each group size.
   - `weight_repeated_pairings`: Weight for the repeated pairings objective function.
   - `weight_level`: Weight for the level balance objective function.
   - `name_data_file`: Name of the input Excel file.
   - `results_data_file`: Name of the output Excel file.
3. Run the script:
   ```
   python main.py
   ```

## Input Data

The input Excel file should contain three sheets:

1. `attendance_list`: A matrix with students as rows and sessions as columns. Use 1 to indicate attendance, 0 for absence.
2. `historical_pairings`: A matrix showing how many times each pair of students has worked together previously.
3. `students_level`: A list of students with their corresponding skill levels.

## Output

The script generates two output files:

1. A text file `results_{session}.txt` containing:
   - The objective function value
   - Squared sum of repeated assignments
   - Number of groups with at least one student of level 1
   - Group assignments with their levels
   - Execution time
2. An Excel file `students_group_assignments_level.xlsx` containing:
   - A sheet with the new group assignments
   - An updated historical pairings matrix

## Example Usage

1. Input: 'students_data_example.xlsx'
2. In 'main.py':
   ```python
   number_of_groups_per_size = {2: 5}  # 5 groups of 2 students
   weight_repeated_pairings = 100
   weight_level = 1
   ```
3. Run script
4. Output: 
   - 'results_S1.txt': Group assignments, levels, and objective values
   - 'students_group_assignments_level.xlsx': New groups and updated pairings

The optimizer minimizes repeated pairings and balances group levels based on historical data and student levels.

## MILP Formulation

## MILP Formulation

The MILP problem is formulated as follows:

### Sets and Parameters

- $I$: Set of students
- $G$: Set of groups
- $P$: Set of student pairs $(i,j)$ where $i,j \in I$ and $i < j$
- $s_g$: Size of group $g \in G$
- $h_{ij}$: Historical number of pairings between students $i$ and $j$
- $l_i$: Level of student $i \in I$
- $w_r$: Weight for repeated pairings objective
- $w_l$: Weight for level balance objective

### Decision Variables

- $x_{ig}$: Binary variable, 1 if student $i$ is assigned to group $g$, 0 otherwise
- $z_{ijg}$: Binary variable, 1 if both students $i$ and $j$ are assigned to group $g$, 0 otherwise
- $l_g$: Integer variable, total level of group $g$
- $y_g$: Binary variable, 1 if group $g$ has at least one student with level 1, 0 otherwise

### Auxiliary Variables

- $sqr\_sum\_rep$: Integer variable, sum of squared repeated pairings
- $groups\_with\_level\_1$: Integer variable, number of groups with at least one student of level 1

### Objective Function

Minimize the weighted sum of repeated pairings and maximize the number of groups with level 1 students:

$$
\text{Minimize} \quad w_r \cdot sqr\_sum\_rep - w_l \cdot groups\_with\_level\_1
$$

### Constraints

1. Each student is assigned to exactly one group:

   $$\sum_{g \in G} x_{ig} = 1 \quad \forall i \in I$$

2. Each group must have the specified number of students:

   $$\sum_{i \in I} x_{ig} = s_g \quad \forall g \in G$$

3. Ensure $z_{ijg}$ is 1 if both $i$ and $j$ are assigned to the same group:

   $$z_{ijg} \leq x_{ig} \quad \forall (i,j) \in P, g \in G$$
   $$z_{ijg} \leq x_{jg} \quad \forall (i,j) \in P, g \in G$$
   $$z_{ijg} \geq x_{ig} + x_{jg} - 1 \quad \forall (i,j) \in P, g \in G$$

4. Calculate the total level of each group:

   $$l_g = \sum_{i \in I} l_i \cdot x_{ig} \quad \forall g \in G$$

5. Set $y_g$ to 1 if the group has at least one student with level 1:

   $$y_g \geq l_g - \max_{g \in G}(s_g) \quad \forall g \in G$$
   $$y_g \leq l_g \quad \forall g \in G$$

6. Calculate the sum of squared repeated pairings:

   $$sqr\_sum\_rep = \sum_{(i,j) \in P} \sum_{g \in G} h_{ij}^2 \cdot z_{ijg}$$

7. Calculate the number of groups with at least one student of level 1:

   $$groups\_with\_level\_1 = \sum_{g \in G} y_g$$

8. Binary and integer constraints:

   $$x_{ig}, z_{ijg}, y_g \in \{0,1\} \quad \forall i \in I, j \in I, g \in G$$
   $$l_g, sqr\_sum\_rep, groups\_with\_level\_1 \in \mathbb{Z}^+ \quad \forall g \in G$$

This formulation allows the solver to find an optimal assignment of students to groups while minimizing repeated pairings and balancing group levels.
