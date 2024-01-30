---
layout: single
title:  "Longest Common Subsequence"
date:   2024-01-29 16:40:00 +0530
categories: programming
---

Dynamic programming has always been elusive, every program I practiced seems unique and despite many tutorials and hours, never got a grip on it.  I **think** I may have found the elixir with this resource
[Dynamic Programming is not Black Magic](https://qsantos.fr/2024/01/04/dynamic-programming-is-not-black-magic/). The author went through "Edit Distance"/"Levenstein distance" program in detail, improving in-steps from a simpler recursive solution to a time optimized iterative solution. At the end of the article, there was suggestion on follow-up programs that can use dynamic programming. 

**Longest Common Subsequence(LCS)** is another program that can be solved using dynamic programming, I have seen this problem solved via iterative solution, but have never understood the process of arriving at that, this is the process of discovery.

LCS description from Google:
```
A longest common subsequence (LCS) is the longest subsequence common to all sequences in a set of sequences (often just two sequences). It differs from the longest common substring: unlike substrings, subsequences are not required to occupy consecutive positions within the original sequences.
```

An example for clarity:
```
S1  = [2,3,4,1,1,3,4]
S2  = [1,3,4,2,1,3]
LCS = [3,4,1,3]
```

Thinking about this problem at a high level:
- If either S1 or S2 is empty, the LCS will be empty
  ```
  if len(S1) == 0 or len(s2) == 0:
      return []
  ```
- If we pick the first elements from both lists and if they are same we can include that in our final output and we can exclude that element and proceed with rest of elemens in both lists
  ```
  if s1[0] == s2[0]:
      return [s1[0]] + lcs(s1[1:], s2[1:])
  ```
- When the first elements are not same, we have two possibilities
  - keep the first sequence as-is and then start from second element of second sequence
  - start from second element in first sequence and keep the second sequence as-is
  - We have to explore both, but we can only keep one version at the end, how do we pick which version to keep?
    - Referring to our problem statement, we get a clue, whatever is the **longest** of the two.
  ```
  if s1[0] != s2[0]:
      l1 = lcs(s1[1:], s2)
      l2 = lcs(s1, s2[1:])
      if len(l1) > len(l2):
          return l1
      return l2
  ```

# Version 1 - Recursive solution
Gluing all the above steps together we get our first version.
```
def lcs(s1, s2):
    if len(s1) == 0  or len(s2) == 0:
        return []
    if s1[0] == s2[0]:
        return [s1[0]] + lcs(s1[1:], s2[1:])
    else:
        l1 = lcs(s1[1:], s2)
        l2 = lcs(s1, s2[1:])
        if len(l1) > len(l2):
            return l1
        return l2
```

Running this through inputs of different sizes, we can see how it does as input grows.

Sequences of length 7 & 6, merely milliescons
```
# print(lcs([2,3,4,1,1,3,4], [1,3,4,2,1,3]))

# ❯ time python3 lcs.py
# [3, 4, 1, 3]
# python3 lcs.py  0.01s user 0.00s system 92% cpu 0.018 total
```

Sequences of length 14 & 12, slightly more than doubled, still good
```
# print(lcs([2,3,4,1,1,3,4]*2, [1,3,4,2,1,3]*2))

# ❯ time python3 lcs.py 
# [3, 4, 1, 1, 3, 4, 2, 1, 3]
# python3 lcs.py  0.03s user 0.01s system 94% cpu 0.044 total
```

Sequence lengths tripled, but the time is in seconds now.
```
# print(lcs([2,3,4,1,1,3,4]*3, [1,3,4,2,1,3]*3))

# ❯ time python3 lcs.py 
# [3, 4, 1, 1, 3, 4, 2, 3, 1, 3, 4, 2, 1, 3]
# python3 lcs.py  2.35s user 0.01s system 99% cpu 2.373 total
```

Went to the obvious next step, this is where I had to step away from staring, more than **9 minutes**
```
# print(lcs([2,3,4,1,1,3,4]*4, [1,3,4,2,1,3]*4))
# ❯ time python3 lcs.py 
# [3, 4, 1, 1, 3, 4, 2, 3, 1, 3, 4, 2, 3, 1, 3, 4, 2, 1, 3]
# python3 lcs.py  538.97s user 1.28s system 99% cpu 9:00.71 total
```

We are doing a brute-force recursive calls without any optimization, next step is to reduce number of calls by using off-the-shelf cache.

# Version 2 - Recursion with cache

```
from functools import cache

def lcs_v2(s1, s2):
    @cache
    def lcs(idx1, idx2):
        if idx1 == len(s1) or idx2 == len(s2):
            return []
        if s1[idx1] == s2[idx2]:
            return [s1[idx1]] + lcs(idx1+1, idx2+1)
        else:
            l1 = lcs(idx1+1, idx2)
            l2 = lcs(idx1, idx2+1)
            if len(l1) > len(l2):
                return l1
            return l2
    
    return lcs(0,0)
```

With this one, we can see a huge improvement, **9+ minutes** to **26 milliseconds**

```
# print(lcs_v2([2,3,4,1,1,3,4]*4, [1,3,4,2,1,3]*4))

# ❯ time python3 lcs.py 
# [3, 4, 1, 1, 3, 4, 2, 3, 1, 3, 4, 2, 3, 1, 3, 4, 2, 1, 3]
# python3 lcs.py  0.02s user 0.01s system 92% cpu 0.026 total
```

# Version 3 - Recursion with our own cache

This is an important step in coming up with an iterative version, building our own cache helps us to figure what an iterative appraoch would look like and whether to start from the beginning of the list or the ending of the list.

```
def lcs_v3(s1, s2):
    s1_len = len(s1)
    s2_len = len(s2)
    cache = [[None]*(s2_len+1) for _ in range(s1_len+1)]
    def lcs(idx1, idx2):
        if cache[idx1][idx2] is not None:
            return cache[idx1][idx2]
        if idx1 == s1_len or idx2 == s2_len:
            cache[idx1][idx2] = []
        elif s1[idx1] == s2[idx2]:
            cache[idx1][idx2] = [s1[idx1]] + lcs(idx1+1, idx2+1)
        else:
            l1 = lcs(idx1+1, idx2)
            l2 = lcs(idx1, idx2+1)
            if len(l1) > len(l2):
                cache[idx1][idx2] = l1
            else:
                cache[idx1][idx2] = l2
        return cache[idx1][idx2]
    
    return lcs(0,0)
```

We were able to keep the same time with our own cache.
```
# print(lcs_v3([2,3,4,1,1,3,4]*5, [1,3,4,2,1,3]*5))

# ❯ time python3 lcs.py 
# [3, 4, 1, 1, 3, 4, 2, 3, 1, 3, 4, 2, 3, 1, 3, 4, 2, 3, 1, 3, 4, 2, 1, 3]
# python3 lcs.py  0.02s user 0.01s system 92% cpu 0.031 total
```

# Version 4 - Iterative version

As mentioned in the referenced article, this step requires little bit more deliberation and had to work through some examples and think how the recursive functions are working to figure out that for this problem.
```
def lcs_v4(s1, s2):
    s1_len = len(s1)
    s2_len = len(s2)

    cache = [[None]*(s2_len+1) for _ in range(s1_len+1)]
    for idx1 in range(s1_len, -1, -1):
        for idx2 in range(s2_len, -1, -1):
            if idx1 == s1_len or idx2 == s2_len:
                cache[idx1][idx2] = []
            elif s1[idx1] == s2[idx2]:
                cache[idx1][idx2] = [s1[idx1]] + cache[idx1+1][idx2+1]
            else:
                l1 = cache[idx1+1][idx2]
                l2 = cache[idx1][idx2+1]
                if len(l1) > len(l2):
                    cache[idx1][idx2] = l1
                else:
                    cache[idx1][idx2] = l2
    return cache[0][0]
```

Initial sequence that we used for testing
```
# print(lcs_v4([2,3,4,1,1,3,4], [1,3,4,2,1,3]))

# ❯ time  python3 lcs.py  
# [3, 4, 1, 3]
# python3 lcs.py  0.01s user 0.01s system 92% cpu 0.021 total
```

Almost negligible spike(21 milliseconds to 26 milliseconds) even with the size of the inputs increased by a factor of **5**.
```
print(lcs_v4([2,3,4,1,1,3,4]*5, [1,3,4,2,1,3]*5))

# ❯ time  python3 lcs.py  
# [3, 4, 1, 1, 3, 4, 2, 3, 1, 3, 4, 2, 3, 1, 3, 4, 2, 3, 1, 3, 4, 2, 1, 3]
# python3 lcs.py  0.02s user 0.01s system 92% cpu 0.026 total
```

## What Next
Word Wrap / Line Wrap program