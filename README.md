# Backtracking

A problem is suitable for backtracking when you can:
1) Incrementally build a solution to the problem (called a partial candidate solution)
2) Verify that a partial candidate solution can be rejected *before* it is finished

A classic example is n-Queens. Given an `n x n` chessboard, place `n` queens so that no queens
are on the same row, column, or diagonal. 

In general, *all* backtracking algorithms can be thought of as DFS on a tree, where the root
is an empty solution, internal nodes are partial candidate solutions, and leaves are full
(but not necessarily valid) solutions. Thus, we can write the following as a general form
of backtracking (where we wish to return a list containing all solutions --- this can be modified
to exit early and return the first valid solution).

```python
def backtracking(problem):
    def children(candidate):
        # yields the next partial candidate solutions from candidate
    def reject(candidate):
        # returns True if the partial candidate solution can be rejected, else False
    def accept(candidate):
        # returns True if the partial candidate solution cannot be rejected
        # AND the candidate is a full solution, else False

    result = [] # stores all solutions 
    def dfs(node):
        if reject(node):
            return
        if accept(node):
            result.append(node)
            return
        for child in children(node):
            dfs(child)

    root = ... # root node, whose form will depend on the problem and implementation
    dfs(root)
    return result
```

For example, we can encode solutions the n-Queens problem as an array: say the length `n`
array `board` has `board[i] = j` to indicate that the `i`th row has a queen on the `j`th column
(this is simply a more memory-efficient way to describe the chessboard than a 2-dimensional array,
with the added benefit that, by design, no queens are placed on the same row). Then the above may
look something like

```python
from itertools import combinations

def n_queens_bt(n):
    def children(candidate):
        yield from (candidate + [i] for i in range(n))

    def reject(candidate):
        same_col = (len(candidate) > len(set(candidate)))
        same_diag = any(x1 - y1 == x2 - y2 for (x1, y1), (x2, y2) in combinations(enumerate(candidate), 2))
        same_antidiag = any(x1 + y1 == x2 + y2 for (x1, y1), (x2, y2) in combinations(enumerate(candidate), 2))
        return same_col or same_diag or same_antidiag

    def accept(candidate):
        return not reject(candidate) and len(candidate) == n

    result = []  # stores all solutions 
    def dfs(node):
        if reject(node):
            return
        if accept(node):
            result.append(node)
            return
        for child in children(node):
            dfs(child)

    root = []
    dfs(root)
    yield from result
```

Note that there is an interplay between the `children` function and the `reject` function --- that is, our children
function never generates two partial candidate solutions with queens on the same row, so `reject` does not need
to test for this. We could also modify the `children` function to not generate partial candidate solutions with
queens on the same column, the same diagonal, the same anti-diagonal, etc.

Finally, notice that if we define reject as:

```python
def reject(candidate):
    return False
```

then our dfs will necessarily explore the entire tree (note that we would need to modify `accept` as well,
in this case, since it calls `reject`). In that case, it is just conventional brute-force, exploring all possible
solutions. In the case of n-Queens, it is identical to the following:

```python
from itertools import product

def n_queens_bf(n):
    boards = product(range(n), repeat=n)
    def reject(board):
        same_col = (len(board) > len(set(board)))
        same_diag = any(x1 - y1 == x2 - y2 for (x1, y1), (x2, y2) in combinations(enumerate(board), 2))
        same_antidiag = any(x1 + y1 == x2 + y2 for (x1, y1), (x2, y2) in combinations(enumerate(board), 2))
        return same_col or same_diag or same_antidiag
    yield from (board for board in boards if not reject(board))
```
