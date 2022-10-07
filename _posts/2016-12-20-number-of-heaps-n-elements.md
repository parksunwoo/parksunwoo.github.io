---
title: Number of (binary) heaps on N elements
categories:
  - Dev
tags:
  - algorithm
  - heap
last_modified_at: 2016-12-20T09:00:00-00:00
---

Given an array 1 to N , how many permutations of it will be Heap of N! possible permutations.

---

**Approach:**  
  
**Recurrence equation:**  
**T(n) = T(k) \* T(n-k-1) \* (n-1)Ck where k = number of nodes on left**  
  
**We always have to take care that the Max-Height(left-subtree) - Max-Height(right-subtree) <= 1.**  
  
**number of leafs in a heap of size n = Math.floor((n+1)/2).**  
  
**T(1) = 1**  
**T(2) = 1**  
**T(3) = 2**  
  
**T(4) = 3C2 \* T(2) \* T(1) = 3**  
**T(5) = 4C3 \* T(3) \* T(1) = 8**  
**T(6) = 5C3 \* T(3) \* T(2) = 20**  
**T(7) = 6C3 \* T(3) \* T(3) = 80**  
  
**T(8) = 7C4 \* T(4) \* T(3) = 210**  
**T(9) = 8C5 \* T(5) \* T(3) = 896**  
**T(10) = 9C6 \* T(6) \* T(3) = 3360**  
**T(11) = 10C7 \* T(7) \* T(3) = 19200**  
**T(12) = 11C7 \* T(7) \* T(4) = 79200**  
**T(13) = 12C7 \* T(7) \* T(5) = 506880**  
**T(14) = 13C7 \* T(7) \* T(6) = 2745600**  
**T(15) = 14C7 \* T(7) \* T(7) = 21964800**

**\[참고\]** 

**http://karmaandcoding.blogspot.kr/2012/02/number-of-min-heaps-from-array-of-size.html**
