# Combinatorics Mastery Roadmap
## From Beginner to Advanced

---

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Core Patterns Overview](#core-patterns-overview)
4. [Learning Path](#learning-path)
5. [Pattern-Based Problem Groups](#pattern-based-problem-groups)
6. [Practice Strategy](#practice-strategy)

---

## Introduction

This roadmap is designed to take you from an absolute beginner in combinatorics to mastering all 62 LeetCode problems. The problems are organized by patterns and concepts, with a carefully structured learning progression.

**Total Problems:** 62
**Estimated Time:** 8-12 weeks (depending on your pace)
**Prerequisites Time:** 1-2 weeks

---

## Prerequisites

Before starting the LeetCode problems, ensure you understand these fundamental concepts:

### 1. Basic Mathematics (Week 1)
- **Factorials**: n! = n × (n-1) × ... × 2 × 1
- **Permutations**: nPr = n!/(n-r)!
- **Combinations**: nCr = n!/(r!(n-r)!)
- **Basic probability**: P(A) = favorable outcomes / total outcomes

### 2. Programming Fundamentals (Week 1)
- Modular arithmetic: (a + b) % MOD, (a × b) % MOD
- Handling large numbers with MOD = 10^9 + 7
- Basic recursion and memoization
- Dynamic programming basics

### 3. Essential Mathematical Concepts (Week 2)
- **Pascal's Triangle**: Understanding nCr relationships
- **Inclusion-Exclusion Principle**: Counting with overlaps
- **Stars and Bars**: Distributing identical items
- **Catalan Numbers**: Sequence counting (1, 1, 2, 5, 14, 42...)
- **Derangements**: Permutations with no fixed points
- **GCD/LCM**: Greatest common divisor and least common multiple

### Study Resources for Prerequisites:
- Khan Academy: Combinatorics & Probability
- "Concrete Mathematics" by Knuth (Chapters 1-2)
- YouTube: "Combinatorics" by TheTrevTutor

---

## Core Patterns Overview

### Pattern 1: Basic Counting (12 problems)
**Difficulty:** Easy to Medium
**Core Concept:** Direct application of counting principles
**Key Techniques:** Multiplication principle, addition principle, basic combinatorics

### Pattern 2: Dynamic Programming with Counting (15 problems)
**Difficulty:** Medium to Hard
**Core Concept:** Build solutions incrementally using previous states
**Key Techniques:** Bottom-up DP, memoization, state transitions

### Pattern 3: Mathematical Formula Derivation (8 problems)
**Difficulty:** Medium to Hard
**Core Concept:** Find closed-form mathematical solutions
**Key Techniques:** Pattern recognition, mathematical proofs, formula optimization

### Pattern 4: Combinatorics with Constraints (10 problems)
**Difficulty:** Medium to Hard
**Core Concept:** Count with restrictions and conditions
**Key Techniques:** Inclusion-exclusion, constraint satisfaction, conditional counting

### Pattern 5: Advanced Combinatorics (9 problems)
**Difficulty:** Hard
**Core Concept:** Complex mathematical structures
**Key Techniques:** Stirling numbers, Bell numbers, advanced number theory

### Pattern 6: Modular Arithmetic Heavy (8 problems)
**Difficulty:** Hard
**Core Concept:** Large number computations with modulo
**Key Techniques:** Modular inverse, Fermat's little theorem, Chinese remainder theorem

---

## Learning Path

### Phase 1: Foundation (Weeks 1-2)
**Goal:** Master basic counting and simple DP
**Time:** 2 weeks

#### Week 1: Introduction to Counting
**Problems to Solve:** 5

1. **1863. Sum of All Subset XOR Totals** (Easy - 90.1%)
   - **Why start here:** Introduces subset enumeration
   - **Concepts:** Backtracking, bit manipulation basics
   - **Time:** 30-45 mins

2. **2928. Distribute Candies Among Children I** (Easy - 76.1%)
   - **Why:** Basic stars and bars introduction
   - **Concepts:** Simple distribution, constraint satisfaction
   - **Time:** 30-45 mins

3. **3461. Check If Digits Are Equal in String After Operations I** (Easy - 82.5%)
   - **Why:** Warm-up for pattern recognition
   - **Concepts:** Parity, counting operations
   - **Time:** 20-30 mins

4. **1641. Count Sorted Vowel Strings** (Medium - 79.2%)
   - **Why:** Classic DP counting problem
   - **Concepts:** DP with small state space, stars and bars
   - **Time:** 1-1.5 hours

5. **2221. Find Triangular Sum of an Array** (Medium - 82.0%)
   - **Why:** Pascal's triangle application
   - **Concepts:** Pattern recognition, simulation
   - **Time:** 45 mins - 1 hour

**Week 1 Focus:**
- Understand the difference between permutations and combinations
- Master basic DP state definition
- Practice modular arithmetic

#### Week 2: Grid Paths and Basic DP
**Problems to Solve:** 4

6. **62. Unique Paths** (Medium - 66.5%)
   - **Why:** Classic DP problem, builds intuition
   - **Concepts:** 2D DP, combinatorial interpretation (nCr)
   - **Time:** 1-1.5 hours

7. **3128. Right Triangles** (Medium - 48.3%)
   - **Why:** Counting with geometric constraints
   - **Concepts:** Multiplication principle, optimization
   - **Time:** 1 hour

8. **2063. Vowels of All Substrings** (Medium - 55.2%)
   - **Why:** Contribution technique introduction
   - **Concepts:** Mathematical counting, avoiding brute force
   - **Time:** 1-1.5 hours

9. **3179. Find the N-th Value After K Seconds** (Medium - 53.8%)
   - **Why:** Prefix sum with DP
   - **Concepts:** Iterative DP, pattern observation
   - **Time:** 1 hour

**Week 2 Focus:**
- Visualize DP states
- Understand contribution technique
- Practice avoiding TLE with mathematical shortcuts

---

### Phase 2: Intermediate (Weeks 3-5)
**Goal:** Master constrained counting and mathematical formulas
**Time:** 3 weeks

#### Week 3: Permutations and Arrangements
**Problems to Solve:** 5

10. **634. Find the Derangement of An Array** (Medium - 41.7%)
    - **Why:** Introduces derangement formula
    - **Concepts:** Inclusion-exclusion, recursive formulas
    - **Time:** 1.5-2 hours

11. **1621. Number of Sets of K Non-Overlapping Line Segments** (Medium - 45.5%)
    - **Why:** Stars and bars with constraints
    - **Concepts:** Combinatorial identities
    - **Time:** 2 hours

12. **2400. Number of Ways to Reach a Position After Exactly k Steps** (Medium - 36.8%)
    - **Why:** 1D random walk problem
    - **Concepts:** Combinatorics, parity constraints
    - **Time:** 1.5 hours

13. **2929. Distribute Candies Among Children II** (Medium - 55.7%)
    - **Why:** Extends problem 2928 with limits
    - **Concepts:** Inclusion-exclusion on distributions
    - **Time:** 2 hours

14. **3577. Count the Number of Computer Unlocking Permutations** (Medium - 59.0%)
    - **Why:** Permutation with structure
    - **Concepts:** Constraint-based counting
    - **Time:** 1.5 hours

#### Week 4: Advanced DP and String Counting
**Problems to Solve:** 5

15. **2597. The Number of Beautiful Subsets** (Medium - 50.9%)
    - **Why:** Subset counting with constraints
    - **Concepts:** Backtracking with pruning, DP optimization
    - **Time:** 2 hours

16. **2638. Count the Number of K-Free Subsets** (Medium - 47.2%)
    - **Why:** Similar to above with different constraints
    - **Concepts:** Subset DP, constraint handling
    - **Time:** 1.5 hours

17. **2539. Count the Number of Good Subsequences** (Medium - 48.4%)
    - **Why:** Subsequence counting with frequency
    - **Concepts:** Frequency-based DP
    - **Time:** 2 hours

18. **3247. Number of Subsequences with Odd Sum** (Medium - 47.5%)
    - **Why:** Parity-based counting
    - **Concepts:** Mathematical observation, parity DP
    - **Time:** 1 hour

19. **3428. Maximum and Minimum Sums of at Most Size K Subsequences** (Medium - 21.6%)
    - **Why:** Optimization with combinatorics
    - **Concepts:** Greedy + counting
    - **Time:** 2.5 hours

#### Week 5: Modular Arithmetic Intensive
**Problems to Solve:** 4

20. **1201. Ugly Number III** (Medium - 31.2%)
    - **Why:** Inclusion-exclusion with LCM
    - **Concepts:** Binary search, GCD/LCM, set theory
    - **Time:** 2 hours

21. **2930. Number of Strings Which Can Be Rearranged to Contain Substring** (Medium - 56.9%)
    - **Why:** Inclusion-exclusion on strings
    - **Concepts:** Complement counting
    - **Time:** 2-2.5 hours

22. **2963. Count the Number of Good Partitions** (Hard - 48.9%)
    - **Why:** Partition counting introduction
    - **Concepts:** DP with modular arithmetic
    - **Time:** 2.5 hours

23. **920. Number of Music Playlists** (Hard - 60.0%)
    - **Why:** Classic hard combinatorics problem
    - **Concepts:** DP with inclusion-exclusion
    - **Time:** 3 hours

---

### Phase 3: Advanced (Weeks 6-8)
**Goal:** Master complex combinatorics and hard problems
**Time:** 3 weeks

#### Week 6: Trees and Recursive Structures
**Problems to Solve:** 4

24. **1569. Number of Ways to Reorder Array to Get Same BST** (Hard - 53.9%)
    - **Why:** Tree + combinatorics
    - **Concepts:** Catalan-like counting, tree properties
    - **Time:** 3-4 hours

25. **1916. Count Ways to Build Rooms in an Ant Colony** (Hard - 51.0%)
    - **Why:** Tree DP with factorials
    - **Concepts:** Tree structure, multinomial coefficients
    - **Time:** 3-4 hours

26. **1866. Number of Ways to Rearrange Sticks With K Sticks Visible** (Hard - 60.4%)
    - **Why:** Stirling numbers of first kind
    - **Concepts:** Advanced recurrence relations
    - **Time:** 3-4 hours

27. **3154. Find Number of Ways to Reach the K-th Stair** (Hard - 37.4%)
    - **Why:** Complex state DP
    - **Concepts:** State compression, careful transitions
    - **Time:** 3 hours

#### Week 7: Probability and Expected Value
**Problems to Solve:** 4

28. **458. Poor Pigs** (Hard - 59.1%)
    - **Why:** Information theory in combinatorics
    - **Concepts:** Base conversion, logarithmic thinking
    - **Time:** 2-3 hours (requires mathematical insight)

29. **1467. Probability of a Two Boxes Having The Same Number of Distinct Balls** (Hard - 60.9%)
    - **Why:** Hypergeometric distribution
    - **Concepts:** Probability + combinatorics
    - **Time:** 4 hours

30. **1643. Kth Smallest Instructions** (Hard - 44.6%)
    - **Why:** Lexicographic ordering with combinations
    - **Concepts:** Combinatorial ranking
    - **Time:** 3 hours

31. **1359. Count All Valid Pickup and Delivery Options** (Hard - 64.9%)
    - **Why:** Classic hard pattern
    - **Concepts:** Catalan number variant
    - **Time:** 2.5-3 hours

#### Week 8: Number Theory Integration
**Problems to Solve:** 5

32. **1735. Count Ways to Make Array With Product** (Hard - 54.3%)
    - **Why:** Prime factorization + DP
    - **Concepts:** Number theory, stars and bars
    - **Time:** 4 hours

33. **2338. Count the Number of Ideal Arrays** (Hard - 56.9%)
    - **Why:** Multiplicative function + stars and bars
    - **Concepts:** Number theory, divisibility
    - **Time:** 3-4 hours

34. **2514. Count Anagrams** (Hard - 36.9%)
    - **Why:** Multinomial coefficients
    - **Concepts:** Permutations with repetition
    - **Time:** 2.5 hours

35. **3116. Kth Smallest Amount With Single Denomination Combination** (Hard - 19.9%)
    - **Why:** Inclusion-exclusion + binary search
    - **Concepts:** Complex set theory
    - **Time:** 4-5 hours

36. **3312. Sorted GCD Pair Queries** (Hard - 22.3%)
    - **Why:** Number theory + combinatorics
    - **Concepts:** GCD properties, frequency counting
    - **Time:** 4 hours

---

### Phase 4: Expert (Weeks 9-12)
**Goal:** Tackle the hardest problems with advanced techniques
**Time:** 4 weeks

#### Week 9-10: Competition-Level Problems
**Problems to Solve:** 8

37. **1830. Minimum Number of Operations to Make String Sorted** (Hard - 50.7%)
    - **Why:** Permutation ranking with factoradic
    - **Concepts:** Advanced permutation theory
    - **Time:** 4-5 hours

38. **2842. Count K-Subsequences of a String With Maximum Beauty** (Hard - 30.1%)
    - **Why:** Greedy + combinatorics
    - **Concepts:** Optimal selection with counting
    - **Time:** 3-4 hours

39. **2912. Number of Ways to Reach Destination in the Grid** (Hard - 58.4%)
    - **Why:** Grid DP with obstacles
    - **Concepts:** Path counting with constraints
    - **Time:** 3 hours

40. **2927. Distribute Candies Among Children III** (Hard - 57.2%)
    - **Why:** Most complex distribution problem
    - **Concepts:** Advanced stars and bars
    - **Time:** 3-4 hours

41. **2954. Count the Number of Infection Sequences** (Hard - 37.2%)
    - **Why:** Multinomial + tree properties
    - **Concepts:** Sequence counting
    - **Time:** 4-5 hours

42. **3250. Find the Count of Monotonic Pairs I** (Hard - 47.5%)
    - **Why:** DP with monotonicity constraints
    - **Concepts:** State space optimization
    - **Time:** 3-4 hours

43. **3251. Find the Count of Monotonic Pairs II** (Hard - 24.5%)
    - **Why:** Optimization of problem 3250
    - **Concepts:** Advanced DP optimization
    - **Time:** 4-5 hours

44. **3317. Find the Number of Possible Ways for an Event** (Hard - 34.8%)
    - **Why:** Multi-stage combinatorics
    - **Concepts:** Complex counting with stages
    - **Time:** 4 hours

#### Week 11-12: Extreme Challenges
**Problems to Solve:** 10

45. **3272. Find the Count of Good Integers** (Hard - 69.5%)
    - **Why:** Digit DP with palindrome constraints
    - **Concepts:** Advanced digit DP
    - **Time:** 3-4 hours

46. **3343. Count Number of Balanced Permutations** (Hard - 49.2%)
    - **Why:** Permutation with balance constraints
    - **Concepts:** Complex constraint handling
    - **Time:** 4-5 hours

47. **3352. Count K-Reducible Numbers Less Than N** (Hard - 27.5%)
    - **Why:** Binary + digit DP
    - **Concepts:** Binary operations, recursion depth
    - **Time:** 5 hours

48. **3405. Count the Number of Arrays with K Matching Adjacent Elements** (Hard - 58.4%)
    - **Why:** Adjacent constraint counting
    - **Concepts:** DP with adjacency rules
    - **Time:** 3-4 hours

49. **3426. Manhattan Distances of All Arrangements of Pieces** (Hard - 35.1%)
    - **Why:** Geometry + counting
    - **Concepts:** Manhattan distance, contribution technique
    - **Time:** 4-5 hours

50. **3470. Permutations IV** (Hard - 33.8%)
    - **Why:** Advanced permutation properties
    - **Concepts:** Specialized permutation theory
    - **Time:** 4-5 hours

51. **3395. Subsequences with a Unique Middle Mode I** (Hard - 20.9%)
    - **Why:** Statistical mode + subsequences
    - **Concepts:** Complex constraint satisfaction
    - **Time:** 5-6 hours

52. **3416. Subsequences with a Unique Middle Mode II** (Hard - 13.4%)
    - **Why:** Extreme optimization challenge
    - **Concepts:** Advanced optimization techniques
    - **Time:** 6+ hours

53. **3463. Check If Digits Are Equal in String After Operations II** (Hard - 14.2%)
    - **Why:** Complex transformation counting
    - **Concepts:** Operation sequence analysis
    - **Time:** 5-6 hours

54. **3518. Smallest Palindromic Rearrangement II** (Hard - 14.2%)
    - **Why:** Lexicographic + palindrome
    - **Concepts:** Greedy with palindrome constraints
    - **Time:** 5 hours

#### Final Week: Ultra-Hard Problems
**Problems to Solve:** 8

55. **3539. Find Sum of Array Product of Magical Sequences** (Hard - 62.0%)
    - **Why:** Product sum with sequences
    - **Concepts:** Mathematical derivation
    - **Time:** 4 hours

56. **3621. Number of Integers With Popcount-Depth Equal to K I** (Hard - 22.3%)
    - **Why:** Bit manipulation + recursion depth
    - **Concepts:** Binary properties
    - **Time:** 5 hours

57. **3725. Count Ways to Choose Coprime Integers from Rows** (Hard - 48.3%)
    - **Why:** Number theory + constraints
    - **Concepts:** Coprimality, Chinese remainder theorem
    - **Time:** 4-5 hours

58. **3757. Number of Effective Subsequences** (Hard - 30.3%)
    - **Why:** Effectiveness metric + counting
    - **Concepts:** Custom constraint design
    - **Time:** 5 hours

59. **3821. Find Nth Smallest Integer With K One Bits** (Hard - 33.8%)
    - **Why:** Binary search + combinatorics
    - **Concepts:** Ranking with bit constraints
    - **Time:** 4 hours

60. **1643. Kth Smallest Instructions** (Hard - 44.6%)
    - **Why:** Combinatorial ranking (if not done earlier)
    - **Concepts:** Lexicographic ordering
    - **Time:** 3 hours

61. **920. Number of Music Playlists** (Hard - 60.0%)
    - **Why:** Review of complex DP (if not mastered)
    - **Concepts:** Reinforcement
    - **Time:** 2 hours

62. **Any remaining problem from the list**
    - **Why:** Fill gaps in knowledge
    - **Time:** Variable

---

## Pattern-Based Problem Groups

### Group 1: Stars and Bars (Distribution Problems)
**Core Formula:** Distributing n identical items into k bins
- Formula: C(n+k-1, k-1)

**Problems:**
1. 2928. Distribute Candies Among Children I (Easy)
2. 2929. Distribute Candies Among Children II (Medium)
3. 2927. Distribute Candies Among Children III (Hard)
4. 1641. Count Sorted Vowel Strings (Medium)
5. 1735. Count Ways to Make Array With Product (Hard)
6. 2338. Count the Number of Ideal Arrays (Hard)

**Key Learning:**
- Basic stars and bars formula
- Handling upper/lower bounds with inclusion-exclusion
- Combining with other constraints

---

### Group 2: Grid Paths
**Core Concept:** Counting paths in 2D grids with/without obstacles

**Problems:**
1. 62. Unique Paths (Medium)
2. 2912. Number of Ways to Reach Destination in the Grid (Hard)
3. 1643. Kth Smallest Instructions (Hard)
4. 3154. Find Number of Ways to Reach the K-th Stair (Hard)

**Key Learning:**
- DP on grids: dp[i][j] = dp[i-1][j] + dp[i][j-1]
- Mathematical formula: C(m+n-2, m-1) for m×n grid
- Lexicographic ranking for paths

---

### Group 3: Permutations and Derangements
**Core Concept:** Counting arrangements with constraints

**Problems:**
1. 634. Find the Derangement of An Array (Medium)
2. 1830. Minimum Number of Operations to Make String Sorted (Hard)
3. 2514. Count Anagrams (Hard)
4. 3343. Count Number of Balanced Permutations (Hard)
5. 3470. Permutations IV (Hard)
6. 3577. Count the Number of Computer Unlocking Permutations (Medium)

**Derangement Formula:**
- D(n) = n! × Σ((-1)^k / k!) for k=0 to n
- Or: D(n) = (n-1) × (D(n-1) + D(n-2))

---

### Group 4: Subset and Subsequence Counting
**Core Concept:** Count subsets/subsequences with properties

**Problems:**
1. 1863. Sum of All Subset XOR Totals (Easy)
2. 2597. The Number of Beautiful Subsets (Medium)
3. 2638. Count the Number of K-Free Subsets (Medium)
4. 2539. Count the Number of Good Subsequences (Medium)
5. 3247. Number of Subsequences with Odd Sum (Medium)
6. 3395. Subsequences with a Unique Middle Mode I (Hard)
7. 3416. Subsequences with a Unique Middle Mode II (Hard)
8. 3757. Number of Effective Subsequences (Hard)

**Key Technique:**
- Use DP or backtracking with pruning
- Contribution technique: count how many subsets each element appears in
- Bit manipulation for small n

---

### Group 5: Tree-Based Combinatorics
**Core Concept:** Counting with tree structures

**Problems:**
1. 1569. Number of Ways to Reorder Array to Get Same BST (Hard)
2. 1916. Count Ways to Build Rooms in an Ant Colony (Hard)
3. 1866. Number of Ways to Rearrange Sticks With K Sticks Visible (Hard)
4. 2954. Count the Number of Infection Sequences (Hard)

**Key Concepts:**
- Catalan numbers: C(n) = (2n)! / ((n+1)! × n!)
- Stirling numbers of first kind
- Tree DP with combinatorial coefficients

---

### Group 6: Number Theory + Combinatorics
**Core Concept:** GCD, LCM, prime factorization with counting

**Problems:**
1. 1201. Ugly Number III (Medium)
2. 1735. Count Ways to Make Array With Product (Hard)
3. 2338. Count the Number of Ideal Arrays (Hard)
4. 3116. Kth Smallest Amount With Single Denomination Combination (Hard)
5. 3312. Sorted GCD Pair Queries (Hard)
6. 3725. Count Ways to Choose Coprime Integers from Rows (Hard)

**Key Formulas:**
- Inclusion-Exclusion with LCM
- Euler's totient function
- Chinese Remainder Theorem

---

### Group 7: Digit DP
**Core Concept:** Count numbers with digit constraints

**Problems:**
1. 3461. Check If Digits Are Equal in String After Operations I (Easy)
2. 3463. Check If Digits Are Equal in String After Operations II (Hard)
3. 3272. Find the Count of Good Integers (Hard)
4. 3352. Count K-Reducible Numbers Less Than N (Hard)
5. 3518. Smallest Palindromic Rearrangement II (Hard)
6. 3621. Number of Integers With Popcount-Depth Equal to K I (Hard)
7. 3821. Find Nth Smallest Integer With K One Bits (Hard)

**Standard Template:**
```python
def digit_dp(pos, tight, state):
    if pos == n:
        return 1 if valid(state) else 0
    limit = digit[pos] if tight else 9
    result = 0
    for d in range(0, limit + 1):
        result += digit_dp(pos + 1, tight and (d == limit), new_state)
    return result
```

---

### Group 8: Inclusion-Exclusion Principle
**Core Concept:** Count by adding/subtracting overlaps

**Problems:**
1. 1201. Ugly Number III (Medium)
2. 634. Find the Derangement of An Array (Medium)
3. 2929. Distribute Candies Among Children II (Medium)
4. 2930. Number of Strings Which Can Be Rearranged to Contain Substring (Medium)
5. 920. Number of Music Playlists (Hard)
6. 3116. Kth Smallest Amount With Single Denomination Combination (Hard)

**Formula:**
|A ∪ B ∪ C| = |A| + |B| + |C| - |A ∩ B| - |A ∩ C| - |B ∩ C| + |A ∩ B ∩ C|

---

### Group 9: Catalan Numbers and Variants
**Core Concept:** Recursive structures, balanced sequences

**Problems:**
1. 1359. Count All Valid Pickup and Delivery Options (Hard)
2. 1569. Number of Ways to Reorder Array to Get Same BST (Hard)
3. 2221. Find Triangular Sum of an Array (Medium)

**Catalan Number:**
- C(n) = C(2n, n) / (n + 1)
- Applications: Binary trees, parentheses, paths

---

### Group 10: Modular Arithmetic Heavy
**Core Concept:** Large computations with MOD

**Problems:**
1. 920. Number of Music Playlists (Hard)
2. 1830. Minimum Number of Operations to Make String Sorted (Hard)
3. 2514. Count Anagrams (Hard)
4. 2842. Count K-Subsequences of a String With Maximum Beauty (Hard)
5. 3317. Find the Number of Possible Ways for an Event (Hard)
6. 3343. Count Number of Balanced Permutations (Hard)

**Essential Techniques:**
- Modular inverse: a^(-1) ≡ a^(MOD-2) (mod MOD) [Fermat's Little Theorem]
- Precompute factorials and inverse factorials
- Careful with overflow in intermediate steps

---

## Practice Strategy

### Daily Routine
1. **Read problem** (5 mins)
2. **Identify pattern** (10 mins)
3. **Plan approach** (15 mins)
4. **Code solution** (30-90 mins)
5. **Debug and optimize** (15-30 mins)
6. **Review editorial** (20 mins)
7. **Note key insights** (10 mins)

### Weekly Review
- Review all problems solved that week
- Identify weak patterns
- Redo 2-3 difficult problems

### When Stuck
1. Try to identify the pattern (check this document)
2. Work through small examples by hand
3. Look for mathematical patterns in the examples
4. Check hints or partial solutions (not full solutions)
5. Take a break and return fresh

### Mastery Checklist
For each problem, ensure you can:
- [ ] Identify the pattern immediately
- [ ] Explain the approach to someone else
- [ ] Code the solution without looking at notes
- [ ] Solve similar problems quickly
- [ ] Optimize time and space complexity

---

## Key Formulas Reference

### 1. Combinations and Permutations
```
nCr = n! / (r! × (n-r)!)
nPr = n! / (n-r)!
```

### 2. Stars and Bars
```
Distribute n identical items into k bins:
= C(n + k - 1, k - 1)
```

### 3. Derangements
```
D(n) = n! × Σ((-1)^k / k!) for k = 0 to n
D(n) = (n-1) × (D(n-1) + D(n-2))
```

### 4. Catalan Numbers
```
C(n) = C(2n, n) / (n + 1)
C(n) = Σ(C(i) × C(n-1-i)) for i = 0 to n-1
```

### 5. Inclusion-Exclusion (3 sets)
```
|A ∪ B ∪ C| = |A| + |B| + |C| - |A ∩ B| - |A ∩ C| - |B ∩ C| + |A ∩ B ∩ C|
```

### 6. Modular Inverse
```
a^(-1) ≡ a^(MOD-2) (mod MOD)  [when MOD is prime]
```

### 7. Grid Paths
```
Paths from (0,0) to (m,n) = C(m+n, m)
```

---

## Additional Resources

### Online Judges
- LeetCode (primary)
- Codeforces (Combinatorics tag)
- AtCoder (Math/Combinatorics problems)

### Books
1. "Concrete Mathematics" - Graham, Knuth, Patashnik
2. "Generatingfunctionology" - Herbert Wilf
3. "A Path to Combinatorics for Undergraduates" - Titu Andreescu

### Video Courses
1. MIT OCW: Mathematics for Computer Science (Counting sections)
2. Coursera: Introduction to Discrete Mathematics for Computer Science
3. YouTube: MIT 6.042J (Counting and Probability)

---

## Final Tips

1. **Don't rush:** Combinatorics requires deep understanding, not just pattern matching
2. **Draw examples:** Always work through small examples by hand
3. **Prove to yourself:** Make sure you understand WHY a formula works
4. **Build gradually:** Don't jump to hard problems too quickly
5. **Track progress:** Use the accompanying spreadsheet to monitor your journey
6. **Review regularly:** Combinatorics formulas are easy to forget
7. **Join communities:** Discuss problems on LeetCode forums, Reddit r/leetcode
8. **Compete regularly:** Participate in LeetCode weekly contests

**Remember:** The goal isn't just to solve 62 problems, but to develop an intuition for combinatorial thinking that will serve you in any algorithmic challenge.

Good luck on your journey to mastering combinatorics!
