

## What Dynamic Programming really is (minus the mystical fog)

DP = **structured caching of repeated subproblems**.

If your solution keeps recomputing the same stuff (usually via recursion), DP says:

* “Stop being dramatic”
* **store answers**
* **reuse them**

Two core properties (if these are true, DP is usually viable):

1. **Optimal substructure**: best answer for a problem uses best answers of smaller parts
2. **Overlapping subproblems**: the same smaller parts show up again and again

---

## The DP “spotting” checklist for LeetCode

If a problem asks for any of these, your DP radar should turn on:

### A) The question type screams DP

* **max/min**: “maximum profit”, “minimum cost”, “shortest/least”
* **counting**: “how many ways”, “number of subsets”, “distinct ways”
* **boolean**: “can we do it?”, “is it possible?”
* **strings**: subsequence/substring alignment, edits, matches
* **partitioning**: split into segments with best score/cost
* **paths**: grid traversal with constraints

### B) The constraints hint at DP

* `n` up to **$10^2, 10^3, 10^4$** often suggests **O(n^2)** or **O(n log n)** or **O(n)** DP
* 2D inputs (grid, two strings): often **O(n*m)** DP
* subset sum / knapsack: **sum** up to ~$10^4-10^5$ suggests **O(n*sum)** DP

### C) Wording patterns (classic bait)

* “choose / don’t choose”
* “at each step you can…”
* “depends on previous…”
* “you may take 1 or 2…”
* “longest …”, “minimum …”
* “ways to reach …”
* “with cooldown / fee / k transactions” (stocks = DP factory)

---

## The only DP recipe you actually need

### Step 1) Define the **state**

A state answers: **what info do I need to uniquely describe a subproblem?**

Examples:

* `dp[i]` = best answer up to index `i`
* `dp[i][j]` = best answer for first `i` chars of A and first `j` chars of B
* `dp[i][w]` = can we achieve sum `w` using first `i` items

Rule: **state must be minimal but sufficient**.

### Step 2) Write the **transition**

“How do I compute this state from smaller states?”

Typical shapes:

* **take / skip**:
$$
  dp[i] = \max(dp[i-1], dp[i-2] + a[i])
$$
* **pick an option among many**:
$$
  dp[i] = \min_{k < i}(dp[k] + cost(k \to i))
$$
* **match / mismatch (strings)**:
$$
  dp[i][j] =
  \begin{cases}
  dp[i-1][j-1] + 1 & \text{if } A[i-1]=B[j-1] \
  \max(dp[i-1][j], dp[i][j-1]) & \text{otherwise}
  \end{cases}
$$
### Step 3) Set **base cases**

What are the smallest problems you already know?

* `dp[0] = ...`
* empty string / zero items / sum 0
* edges of grid

### Step 4) Choose **top-down** or **bottom-up**

* **Top-down (memoization)**: recursion + cache (easy to derive)
* **Bottom-up (tabulation)**: iterative fill (often faster / simpler memory)

### Step 5) Complexity check

Count number of states × work per transition.

---

## DP patterns you’ll see constantly

### 1) 1D DP (prefix DP)

**Used for**: climbing stairs, house robber, min cost, LIS variants
State: `dp[i]` uses a few previous values.

### 2) 2D DP grid

**Used for**: unique paths, min path sum, obstacles
State: `dp[r][c]` from top/left (or more).

### 3) Knapsack / subset sum DP

**Used for**: can/can’t reach sum, max value under weight, partition equal subset sum
State often: `dp[i][w]` or optimized to `dp[w]`.

### 4) String DP

**Used for**: LCS, edit distance, regex matching DP-ish, palindrome DP
State: `dp[i][j]` on prefixes or intervals.

### 5) Interval DP

**Used for**: “burst balloons”, matrix chain multiplication, merging stones
State: `dp[l][r]` = best answer on subarray `l..r`, transition tries a split `k`.

### 6) “DP with mode” (extra dimension)

**Used for**: stock trading, cooldowns, constraints, “k times”, “must pick exactly …”
State: `dp[i][state]` where state is like holding/not holding, transactions used, etc.

---

## Worked examples (the ones that teach the brain, not just the hands)

### Example 1: Climbing Stairs (count ways)

**Problem**: ways to reach step `n` with 1 or 2 steps.

**State**: `dp[i]` = number of ways to reach step `i`
**Transition**: to reach `i`, you came from `i-1` or `i-2`
$$
dp[i] = dp[i-1] + dp[i-2]
$$
**Base**: `dp[0]=1` (one way to stand still), `dp[1]=1`
**Answer**: `dp[n]`

Space-optimized:

* keep just last two values.

---

### Example 2: House Robber (max with adjacency constraint)

**Problem**: max money, cannot rob adjacent houses.

**State**: `dp[i]` = max money using houses `0..i`
At house `i`:

* skip it: `dp[i-1]`
* take it: `dp[i-2] + nums[i]`
$$
dp[i] = \max(dp[i-1], dp[i-2] + nums[i])
$$
**Base**:

* `dp[0]=nums[0]`
* `dp[1]=max(nums[0], nums[1])`

This is the “take/skip” DP template in disguise.

---

### Example 3: Coin Change (min coins)

**Problem**: minimum coins to make amount `A` (unlimited coins).

**State**: `dp[x]` = min coins to make amount `x`
**Transition**: last coin could be `c`, so previous is `x-c`
$$
dp[x] = \min_{c \in coins,, x-c \ge 0}(dp[x-c] + 1)
$$
**Base**:

* `dp[0]=0`
* initialize others to “infinity”

If `dp[A]` stays infinity, answer is `-1`.

This is “min over options” DP.

---

### Example 4: Longest Common Subsequence (LCS)

**Problem**: length of longest subsequence common to two strings `A, B`.

**State**: `dp[i][j]` = LCS length for `A[0..i-1]` and `B[0..j-1]`
**Transition**:

* if chars match: take diagonal + 1
* else: best of skipping one char from either string
$$
dp[i][j] =
\begin{cases}
dp[i-1][j-1] + 1 & A[i-1]=B[j-1] \
\max(dp[i-1][j], dp[i][j-1]) & \text{otherwise}
\end{cases}
$$
**Base**: row 0 and col 0 are 0.

This is the blueprint behind a ton of string DP.

---

## Memoization vs Tabulation (how to pick fast)

### Top-down (memo)

Good when:

* recursion is natural
* many states never get visited
* easier to derive quickly

### Bottom-up (tab)

Good when:

* you can order states cleanly
* need speed and avoid recursion depth
* want easy space optimization

---

## DP pitfalls (aka how humans sabotage themselves)

* **State missing info** → forces hacks later → wrong answers
* **Wrong base cases** → everything becomes nonsense but looks “almost right”
* **Bad iteration order** (bottom-up) → using states not computed yet
* **Infinity handling** wrong for min DP (overflow or accidental minima)
* **Double counting** in “count ways” problems (order vs combination matters)

  * “combinations” usually loop coin-first
  * “permutations” usually loop amount-first

---

## Templates you can reuse

### Memoization skeleton

```cpp
// dp[state] = cached answer, initialize to "unknown"
int solve(state) {
    if (base case) return base;
    if (memo[state] != UNKNOWN) return memo[state];

    int ans = ...; // min/max/count
    for (option : options) {
        ans = combine(ans, solve(smaller_state));
    }
    return memo[state] = ans;
}
```

### Bottom-up skeleton

```cpp
// build dp from smallest -> largest
dp[base] = ...;
for (state in increasing order) {
    for (option : options) {
        dp[state] = combine(dp[state], dp[prev_state] + cost);
    }
}
```

---

## Summary tables (since humans love pretending tables equal clarity)

### 1) “What pattern is this?” DP identification table

| Problem vibe                   | Common DP state       | Typical transition | Examples                                       |
| ------------------------------ | --------------------- | ------------------ | ---------------------------------------------- |
| Count ways to reach/compose    | `dp[i]`               | sum of previous    | Climbing Stairs, Decode Ways                   |
| Max/min with local constraint  | `dp[i]`               | take/skip          | House Robber, Delete and Earn                  |
| Grid paths/cost                | `dp[r][c]`            | from neighbors     | Unique Paths, Min Path Sum                     |
| Choose items under limit       | `dp[w]` or `dp[i][w]` | take/skip / max    | 0/1 Knapsack, Partition Equal Subset Sum       |
| Unlimited choices to reach sum | `dp[x]`               | min/sum over `x-c` | Coin Change, Combination Sum IV                |
| Two strings alignment          | `dp[i][j]`            | diag / up / left   | LCS, Edit Distance                             |
| Best over subarray splits      | `dp[l][r]`            | try split `k`      | Burst Balloons, Matrix Chain                   |
| Extra “mode” needed            | `dp[i][mode]`         | state machine      | Best Time to Buy/Sell Stock (k, cooldown, fee) |

---

### 2) DP build checklist (do this every time)

| Step       | What you must write down      | Quick sanity test                        |
| ---------- | ----------------------------- | ---------------------------------------- |
| State      | what subproblem means         | does it uniquely define the subproblem?  |
| Transition | how state uses smaller ones   | are smaller indices strictly smaller?    |
| Base       | smallest cases                | do they match reality (empty, 0, edges)? |
| Order      | top-down or bottom-up order   | no “future” reads in tabulation          |
| Complexity | #states × cost per state      | does it fit constraints?                 |
| Optimize   | reduce dimensions if possible | do you only need previous row/2 states?  |

That’s DP: not magic, just organized laziness.
