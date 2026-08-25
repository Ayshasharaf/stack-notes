# Time & Space Complexity — Simple Guide

---

## What is Time Complexity?

**How many steps does your code take as input grows?**

You don't count seconds. You count how many times the code *runs* relative to the input size `n`.

```java
// n = 5 → runs 5 times
// n = 1000 → runs 1000 times
for (int i = 0; i < n; i++) {
    System.out.println(i);
}
```

---

## What is Space Complexity?

**How much extra memory does your code use as input grows?**

```java
// You created an array of size n → space grows with n
int[] arr = new int[n];

// You just created 3 variables → always 3, doesn't matter if n = 5 or 5000
int l = 0, r = 0, sum = 0;
```

---

## Big O Notation

Big O describes the **worst case** and **ignores small details**.

- Drop constants → `O(2n)` becomes `O(n)`
- Drop smaller terms → `O(n² + n)` becomes `O(n²)`

---

## The Common Complexities (Fastest → Slowest)

### O(1) — Constant
**Does not matter how big n is, always same steps.**

```java
int x = nums[0];        // just grab index 0
int y = nums[n - 1];    // just grab last index
```

```
n = 10     → 1 step
n = 10000  → 1 step
```

---

### O(log n) — Logarithmic
**Cuts the problem in half each time.**

```java
// Binary Search
int l = 0, r = n - 1;
while (l <= r) {
    int mid = (l + r) / 2;   // half each time
    ...
}
```

```
n = 8    → 3 steps   (8 → 4 → 2 → 1)
n = 1000 → ~10 steps
n = 1M   → ~20 steps
```

Very fast. Seen in: Binary Search, Balanced BST operations.

---

### O(n) — Linear
**One pass through the input.**

```java
for (int i = 0; i < n; i++) {
    // do something once per element
}
```

```
n = 10    → 10 steps
n = 1000  → 1000 steps
```

Seen in: Two Pointers, Sliding Window, simple loops.

---

### O(n log n) — Linearithmic
**Sorting is the classic example.**

```java
Arrays.sort(nums);   // O(n log n)
```

```
n = 8    → ~24 steps
n = 1000 → ~10000 steps
```

Seen in: Merge Sort, most sorting algorithms, sort + linear pass problems.

---

### O(n²) — Quadratic
**Loop inside a loop.**

```java
for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) {
        // runs n * n times
    }
}
```

```
n = 10   → 100 steps
n = 1000 → 1,000,000 steps
```

Seen in: Brute force 3Sum, bubble sort, naive string matching.

---

### O(n³) — Cubic
**Three nested loops. Avoid if possible.**

```java
for (int i ...)
    for (int j ...)
        for (int k ...)   // n * n * n
```

```
n = 10   → 1,000 steps
n = 100  → 1,000,000 steps
```

Seen in: Brute force 3Sum, matrix multiplication.

---

### O(2ⁿ) — Exponential
**Doubles with every extra element. Very slow.**

```
n = 10  → 1024 steps
n = 30  → 1,000,000,000 steps
```

Seen in: Recursive subsets, some backtracking.

---

## Visual Size Comparison

```
O(1)        ████
O(log n)    ████░
O(n)        ████████░░
O(n log n)  ████████████░░
O(n²)       ██████████████████████░░░░░░░░
O(2ⁿ)       off the chart 🚀
```

---

## Space Complexity Examples

### O(1) Space — no extra memory
```java
// Two pointers — just 2 variables
int l = 0, r = n - 1;
```

### O(n) Space — extra array or list
```java
// Storing results
List<Integer> res = new ArrayList<>();  // can grow to size n

// Frequency array
int[] freq = new int[26];               // fixed 26, so actually O(1)
int[] freq = new int[n];               // grows with n → O(n)
```

### O(n²) Space — 2D grid or matrix
```java
int[][] dp = new int[n][n];   // n * n memory
```

---


## LeetCode Complexity Targets

| Problem Type | Target Time | Target Space |
|---|---|---|
| Simple loop | O(n) | O(1) |
| Two pointers | O(n) | O(1) |
| Binary search | O(log n) | O(1) |
| Sorting + loop | O(n log n) | O(1) |
| HashMap lookup | O(n) | O(n) |
| Nested loops | O(n²) | O(1) |

---

## The One Rule to Remember

> **If you see one loop → O(n). Loop inside loop → O(n²). Each time you halve the problem → O(log n).**

