
# Find the Smallest Missing Positive Integer

Given an unsorted array of integers, find the smallest positive integer not present in the array in **O(n)** time and **O(1)** extra space.

## Example

**Input**
```plaintext
orderNumbers = [3, 4, -1, 1]
```

**Output**
```plaintext
2
```

**Explanation**

We want the smallest positive missing integer.

Start with `[3, 4, -1, 1]`
- i=0: value 3 ⇒ swap with index 2 ⇒ `[-1, 4, 3, 1]`
- i=0: value -1 is out of range ⇒ move on
- i=1: value 4 ⇒ swap with index 3 ⇒ `[-1, 1, 3, 4]`
- i=1: value 1 ⇒ swap with index 0 ⇒ `[1, -1, 3, 4]`
- now 1 at index 0, 3 at 2, 4 at 3; -1 remains at index 1

Scan from index 0:  
index 0 has 1 (correct), index 1 has -1 (not 2) ⇒ missing positive is **2**

## Input Format

- An integer `n` on the first line, where 0 ≤ n ≤ 1000.  
- The next `n` lines contain an integer representing `orderNumbers[i]`.

**Example Input**
```plaintext
4
3
4
-1
1
```

## Constraints

- 0 ≤ `orderNumbers.length` ≤ 1000  
- -10⁹ ≤ `orderNumbers[i]` ≤ 10⁹ for all 0 ≤ i < `orderNumbers.length`  
- Duplicates are allowed in `orderNumbers`  
- Negative numbers and zero are allowed in `orderNumbers`

## Output Format

A single integer denoting the smallest positive order number (≥1) that does not appear in the input array.

## Sample Input 0

```plaintext
0
```

## Sample Output 0

```plaintext
1
```

## Sample Input 1

```plaintext
1
1
```

## Sample Output 1

```plaintext
2
```

---

**Note**: This is the classic "First Missing Positive" problem, solvable using the array itself as a hash table via swapping.
