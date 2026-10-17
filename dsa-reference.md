# DSA reference (Pages A–D)

Scan this when a new problem feels familiar but the code will not come. Each entry is a recognition card: what the problem is really asking, the pattern name, the steps that survive a blank editor, complexity, and the mistake that usually sends the solution off course.

Code sketches are C++ and are meant to be re-derived, not memorized line by line.

## How to use this when stuck

1. Name the **data shape** before the algorithm: sorted array, linked list, tree, graph, string window, choices with overlapping subproblems.
2. Match that shape to the **pattern index** below.
3. Open the closest card. Steal the invariant (what is always true after each step), not the variable names.
4. If two patterns both fit, prefer the one whose **bottleneck** matches the constraints (n ≤ 10^5 usually forbids O(n^2); n ≤ 20 often means bitmasks or 2^n).

## Complexity cheat sheet

| Constraint on n | Usually acceptable |
| --- | --- |
| n ≤ 10 | 2^n · n!, heavy recursion |
| n ≤ 20–25 | 2^n or n · 2^n |
| n ≤ 100 | O(n^3) (Floyd, interval DP) |
| n ≤ 1e3–5e3 | O(n^2) |
| n ≤ 1e5 | O(n log n) |
| n ≤ 1e6–1e7 | O(n) or O(n log n) with a small constant |

Space limits matter as much as time. “O(1) extra space” on a linked list usually means pointer rewiring. On an array it often allows swapping inside the input.

## Pattern index

| If you notice… | Reach for | Cards |
| --- | --- | --- |
| Sorted array, “first true / last true”, monotonic predicate | Binary search on index or on answer | A3–A9, C24–C25 |
| Rotated sorted array, one pivot | Binary search; decide which half is sorted | A5 |
| Linked list, middle / cycle / nth from end | Fast and slow pointers | A10–A13, A29, D18 |
| “Next greater / previous smaller”, temperatures, spans | Monotonic stack | A15, A21, B6 |
| Grid, “minutes until”, multi-source spread | Multi-source BFS | A16 |
| Dependencies, course schedule, “order of build” | Topological sort (Kahn or DFS) | A23–A24 |
| Unweighted shortest path, layers | BFS | A25, A28, B15, D14 |
| Overlapping subproblems, “ways”, “min cost”, house row | 1D DP | B1–B5, B13, B19–B20 |
| Contiguous subarray, at most K distinct, fixed window | Sliding window | A22, B6, C26, D11 |
| k-th, top k, running median, “always extract best” | Heap | C1–C5, C25 |
| Local best never hurts the global best | Greedy + sort or exchange argument | C6–C8, C12, A27 |
| Bit appears once, parity, subset XOR | XOR / bits | C9–C11, D18 (cycle, not XOR) |
| Partition into subsets with a target sum | 0/1 knapsack DP | C15 |
| Grid/word existence | DFS backtracking | C16, B14 |
| Connect components, MST, “are they same group” | Union-Find; Kruskal | B22, C14, C18 |
| Dense shortest paths, all pairs, negative edges | Floyd / Bellman-Ford / Dijkstra | C21–C23, A28 |
| Tree shape questions (depth, diameter, path sum) | DFS returning a value to the parent | D1–D9 |
| BST inorder is sorted; LCA of BST | Inorder property or BST split | D12 |
| Permutation of 1..n, duplicate, cycle | Index-as-pointer / Floyd cycle | B10, D17–D18 |
| Parentheses, balance, min insertions | Counter or stack; never go negative | C28 |
| Prefix function, pattern inside text | KMP | D10 |
| “Minimum possible maximum” or “maximum possible minimum” | Binary search the answer | A27, C24 |

---

# Page A

## A1. Iterative pre-order traversal

**What it is.** Visit a binary tree in order **node, left, right** without recursion.

**Pattern.** Explicit stack simulating the call stack.

**Recognize when.** The interviewer bans recursion, the tree can be skewed to 1e5, or you need pre-order for serialization (the first time you see a node is when you emit it).

**Invariant.** The stack holds nodes whose own value is not yet processed, closest-to-visit on top.

**Steps.**

1. If `root` is null, return an empty list.
2. Push `root`.
3. While the stack is not empty: pop `node`, record `node->val`, then push **right first**, then **left**.
4. Pushing right before left makes left come out next, which is pre-order.

**Complexity.** O(n) time, O(h) stack space (O(n) if skewed).

**Pitfalls.** Pushing left before right silently produces node-right-left. Null children must not be pushed.

```cpp
vector<int> preorder(TreeNode* root) {
    vector<int> ans;
    if (!root) return ans;
    stack<TreeNode*> st;
    st.push(root);
    while (!st.empty()) {
        TreeNode* node = st.top(); st.pop();
        ans.push_back(node->val);
        if (node->right) st.push(node->right);
        if (node->left) st.push(node->left);
    }
    return ans;
}
```

**Related.** Inorder needs a different loop (go left, then visit). Post-order is pre-order of the mirrored tree, then reverse the result: push left then right, append, reverse.

---

## A2. Iterative inorder traversal

**What it is.** Visit **left, node, right**. On a BST this yields sorted order.

**Pattern.** Stack of ancestors; a pointer walks as far left as possible before a visit.

**Recognize when.** You need sorted BST output, k-th smallest in a BST, or validate BST while scanning inorder.

**Invariant.** Every node on the stack has had its left subtree fully explored, and has not been visited yet.

**Steps.**

1. `curr = root`.
2. While `curr` is not null or the stack is not empty:
   - While `curr` is not null: push it, go left.
   - Pop, record the value (this is the visit), then `curr = popped->right`.
3. The inner while is “descend”. The pop is “visit”. The right assignment is “explore the other side”.

**Complexity.** O(n) time, O(h) space.

**Pitfalls.** Visiting on the way down (that is pre-order). Forgetting to move to `right` after pop loops forever or skips the right subtree.

```cpp
vector<int> inorder(TreeNode* root) {
    vector<int> ans;
    stack<TreeNode*> st;
    TreeNode* curr = root;
    while (curr || !st.empty()) {
        while (curr) {
            st.push(curr);
            curr = curr->left;
        }
        curr = st.top(); st.pop();
        ans.push_back(curr->val);
        curr = curr->right;
    }
    return ans;
}
```

**Morris inorder (O(1) extra space).** If a node has a left child, find its inorder predecessor (rightmost of the left subtree) and temporarily point that predecessor’s right at the node. Second time you see the thread, you visit the node and remove the thread. Use this only when extra stack memory is forbidden; it mutates the tree temporarily and is easy to get wrong.

---

## A3. Sqrt(x) — LeetCode 69

**What it is.** Return the floor of sqrt(x) for a non-negative integer. `sqrt(8) = 2`.

**Pattern.** Binary search on the answer. The predicate “`mid * mid <= x`” is monotonic.

**Recognize when.** “Find the largest integer whose square does not exceed x”, or any “largest value that still satisfies a growing cost”.

**Steps.**

1. Special-case `x < 2`: the answer is `x`.
2. Search `lo = 1`, `hi = x / 2` (sqrt(x) never exceeds x/2 for x ≥ 2, except you can also set `hi = x`).
3. While `lo <= hi`: `mid = lo + (hi - lo) / 2`.
4. Compare using division to avoid overflow: if `mid <= x / mid`, this mid is feasible, record it, search higher (`lo = mid + 1`). Else `hi = mid - 1`.
5. The best feasible mid is the floor.

**Complexity.** O(log x) time, O(1) space.

**Pitfalls.** `mid * mid` overflows 32-bit `int`. Use `long long` or compare `mid` with `x / mid`. Returning `hi` after the loop also works if the loop invariant is set up so `hi` is the last feasible value (`while (lo <= hi)` plus update rules must match; do not mix “return lo” and “return hi” from different templates).

```cpp
int mySqrt(int x) {
    if (x < 2) return x;
    long long lo = 1, hi = x / 2, ans = 1;
    while (lo <= hi) {
        long long mid = lo + (hi - lo) / 2;
        if (mid <= x / mid) { ans = mid; lo = mid + 1; }
        else hi = mid - 1;
    }
    return (int)ans;
}
```

---

## A4. Guess Number Higher or Lower — LeetCode 374

**What it is.** Guess a hidden number in `[1, n]`. The API returns `-1` (too high), `1` (too low), or `0` (equal).

**Pattern.** Binary search where the check is an interactive predicate instead of an array lookup.

**Steps.**

1. `lo = 1`, `hi = n`.
2. `mid = lo + (hi - lo) / 2`, call `guess(mid)`.
3. `0` → return mid. `-1` → answer is below mid, `hi = mid - 1`. `1` → `lo = mid + 1`.

**Complexity.** O(log n) guesses.

**Pitfalls.** Swapping the meaning of `-1` and `1`. The problem statement defines them; do not assume “positive means too high”.

This is the same skeleton as every binary search. The only new idea is that `guess` replaces `nums[mid] < target`.

---

## A5. Search in Rotated Sorted Array — LeetCode 33

**What it is.** A sorted array was rotated some unknown number of times (`[4,5,6,7,0,1,2]`). Find `target`’s index, or `-1`. Distinct values (the duplicate variant is LC 81).

**Pattern.** Binary search in a half-sorted array. One of the two halves is always fully sorted.

**Recognize when.** “Sorted but rotated”, mountain that was cut, or “find in log n without unrotating”.

**Steps.**

1. `lo = 0`, `hi = n - 1`.
2. At `mid`, if `nums[mid] == target` return mid.
3. If `nums[lo] <= nums[mid]`, the **left half is sorted**.
   - If `target` lies in `[nums[lo], nums[mid])`, search left (`hi = mid - 1`).
   - Else search right.
4. Otherwise the **right half is sorted**.
   - If `target` lies in `(nums[mid], nums[hi]]`, search right.
   - Else search left.

**Complexity.** O(log n) time, O(1) space.

**Pitfalls.** Using `<` against the wrong endpoint. Drawing one example with the pivot on the left and one with the pivot on the right before coding. With duplicates (LC 81), when `nums[lo] == nums[mid]`, you cannot tell which side is sorted; shrink `lo++` and accept O(n) worst case.

```cpp
int search(vector<int>& nums, int target) {
    int lo = 0, hi = (int)nums.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) return mid;
        if (nums[lo] <= nums[mid]) {
            if (nums[lo] <= target && target < nums[mid]) hi = mid - 1;
            else lo = mid + 1;
        } else {
            if (nums[mid] < target && target <= nums[hi]) lo = mid + 1;
            else hi = mid - 1;
        }
    }
    return -1;
}
```

---

## A6. First Bad Version — LeetCode 278

**What it is.** Versions `1..n`. `isBadVersion(v)` becomes true at some version and stays true. Find the first bad one.

**Pattern.** Binary search for the first index where a monotonic predicate turns true. This is the lower-bound template.

**Steps.**

1. `lo = 1`, `hi = n`.
2. While `lo < hi`: `mid = lo + (hi - lo) / 2`.
3. If `isBadVersion(mid)`, the answer is mid or earlier: `hi = mid`.
4. Else it is after mid: `lo = mid + 1`.
5. Return `lo`. Loop ends when `lo == hi`, and that version is bad.

**Complexity.** O(log n) calls.

**Pitfalls.** `while (lo <= hi)` together with `hi = mid` can infinite-loop when `mid` does not move. The `lo < hi` / `hi = mid` pair is the safe “first true” template. Also `mid = (lo + hi) / 2` can overflow; use `lo + (hi - lo) / 2`.

**Template to reuse.** First position with `pred(i) == true`, assuming false…false, true…true:

```cpp
int firstTrue(int lo, int hi) { // inclusive
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (pred(mid)) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}
```

---

## A7. Find Peak Element — LeetCode 162

**What it is.** Any index `i` with `nums[i] > nums[i-1]` and `nums[i] > nums[i+1]` (ends have one neighbor). Array elements are unequal beside each other. Return any peak.

**Pattern.** Binary search on a slope. You do not need the global maximum.

**Steps.**

1. `lo = 0`, `hi = n - 1`.
2. While `lo < hi`: `mid = lo + (hi - lo) / 2`.
3. If `nums[mid] < nums[mid + 1]`, a peak exists on the right (the slope is still rising): `lo = mid + 1`.
4. Else a peak exists at mid or on the left: `hi = mid`.
5. Return `lo`.

**Why it is correct.** Walking uphill must hit a peak before the array ends, because the end is higher than its only neighbor if you arrive there still climbing.

**Complexity.** O(log n). A linear scan also works but misses the point.

**Pitfalls.** Comparing `nums[mid]` with `nums[mid - 1]` and then moving the wrong way. Neighbors outside `[0, n)` do not exist; the template above never reads `mid - 1`.

---

## A8. Find First and Last Position of Element in Sorted Array — LeetCode 34

**What it is.** Sorted `nums`, possibly with duplicates. Return `[first, last]` index of `target`, or `[-1, -1]`.

**Pattern.** Two lower-bound searches. `lower_bound(target)` and `lower_bound(target + 1) - 1`.

**Steps.**

1. Write `lowerBound(x)`: first index with `nums[i] >= x`, or `n` if none.
2. `L = lowerBound(target)`. If `L == n` or `nums[L] != target`, return `[-1, -1]`.
3. `R = lowerBound(target + 1) - 1`.
4. Return `{L, R}`.

**Complexity.** O(log n).

**Pitfalls.** One binary search that stops at any match, then scanning outward: that scan is O(n) when the array is all the same value. For `target` equal to `INT_MAX`, do not compute `target + 1`; run a second search for the last index `<= target` instead.

```cpp
int lowerBound(vector<int>& a, int x) {
    int lo = 0, hi = (int)a.size(); // hi is exclusive
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (a[mid] < x) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}
```

---

## A9. Single Element in a Sorted Array — LeetCode 540

**What it is.** Sorted array; every value appears twice except one. Return the single one. Must be O(log n). Example: `[1,1,2,3,3,4,4]` → `2`.

**Pattern.** Binary search using pair alignment. Before the single element, pairs sit on `(even, odd)` indexes. After it, pairs sit on `(odd, even)`.

**Steps.**

1. `lo = 0`, `hi = n - 1`. `n` is odd.
2. While `lo < hi`: `mid = lo + (hi - lo) / 2`. Force `mid` even: `mid -= mid & 1` (if mid is odd, step back to the pair start).
3. If `nums[mid] == nums[mid + 1]`, the single element is strictly to the right: `lo = mid + 2`.
4. Else the single element is at mid or to the left: `hi = mid`.
5. Return `nums[lo]`.

**Complexity.** O(log n) time, O(1) space. XOR of all elements is O(n) and correct but too slow for the required bound.

**Pitfalls.** Forgetting to align `mid` to an even index, so you compare the second half of a pair with the next number.

---

## A10. Intersection of Two Linked Lists — LeetCode 160

**What it is.** Two singly linked lists may merge. Return the first shared node (by reference, not by value), or null.

**Pattern.** Two pointers that switch lists. Equalizing length without measuring it.

**Why it works.** Let exclusive lengths be `a` and `b`, and shared tail `c`. Pointer A walks `a + c + b`. Pointer B walks `b + c + a`. They meet at the intersection after the same number of steps. If there is no intersection, both become null together.

**Steps.**

1. `p = headA`, `q = headB`.
2. While `p != q`: `p = p ? p->next : headB`, `q = q ? q->next : headA`.
3. Return `p`.

**Complexity.** O(a + b) time, O(1) space.

**Alternative.** Measure both lengths, advance the longer one by the difference, then walk in lockstep.

**Pitfalls.** Comparing node values instead of node pointers. Cycles are out of scope for this problem; if a cycle exists this loop never ends (use Floyd separately).

---

## A11. Palindrome Linked List — LeetCode 234

**What it is.** Is the list the same forward and backward?

**Pattern.** Fast/slow to the middle, reverse the second half, compare, optionally restore.

**Steps.**

1. Slow and fast from head. Fast moves two, slow moves one. When fast finishes, slow is at the middle (for odd length, at the center node).
2. Reverse the list starting at `slow`.
3. Compare the reversed second half with the first half node by node.
4. If the caller needs the original list, reverse the second half back.

**Complexity.** O(n) time, O(1) extra space.

**Pitfalls.** Using a stack or array is correct but uses O(n) memory; mention it only as the simple version. Off-by-one on odd length: the center node has no partner and can be ignored. Reversing from the wrong node compares overlapping halves.

```cpp
ListNode* reverse(ListNode* h) {
    ListNode* prev = nullptr;
    while (h) {
        ListNode* nxt = h->next;
        h->next = prev;
        prev = h;
        h = nxt;
    }
    return prev;
}
```

---

## A12. Remove Nth Node From End of List — LeetCode 19

**What it is.** Delete the n-th node counting from the end. `n` is valid. One pass is the expected solution.

**Pattern.** Gap pointer (fast leads by n), plus a dummy head so deleting the real head is not special.

**Steps.**

1. `dummy->next = head`. `fast = slow = dummy`.
2. Advance `fast` exactly `n + 1` steps so the gap between slow and fast is n nodes of real list, and slow will stop on the **predecessor** of the target.
3. Move both until `fast` is null.
4. `slow->next = slow->next->next`.
5. Return `dummy->next`.

**Complexity.** O(n) time, O(1) space.

**Pitfalls.** Advancing fast only n steps leaves slow on the target, not before it, so you cannot delete. Without a dummy, removing the head needs a separate branch.

---

## A13. Remove Duplicates from Sorted List — LeetCode 83

**What it is.** Sorted list. Delete extra copies so each value remains once. LC 82 is the harder sibling: delete **all** nodes whose value appears more than once.

**Pattern.** Because the list is sorted, duplicates are adjacent. One pointer looks ahead.

**Steps (LC 83, keep one).**

1. `curr = head`.
2. While `curr` and `curr->next`: if values match, `curr->next = curr->next->next`. Else `curr = curr->next`.
3. Return head. Head never needs deletion because you keep the first copy.

**Steps (LC 82, keep none of a duplicated value).**

1. Dummy before head. `prev = dummy`.
2. While `prev->next`: if the next value’s run has length ≥ 2, skip the whole run. Else advance `prev` once.

**Complexity.** O(n), O(1).

**Pitfalls.** Applying the LC 83 loop to LC 82 leaves one copy of a duplicated value. Unlinking a node without keeping `next` is fine in a list if you overwrite `next` directly; do not lose the remainder.

---

## A14. Min Stack — LeetCode 155

**What it is.** Stack with `push`, `pop`, `top`, and `getMin`, all O(1).

**Pattern.** Auxiliary stack of minima, or store `{value, minSoFar}` in one stack.

**Steps.**

1. Main stack stores values. Min stack stores the minimum at or below the current height.
2. On push `x`: push `x` to main. Push `min(x, currentMin)` to the min stack (if empty, push `x`).
3. On pop: pop both.
4. `getMin` is the top of the min stack.

**Complexity.** O(1) each operation, O(n) extra space.

**Pitfalls.** Pushing onto the min stack only when `x < min` (strict) breaks when a duplicate minimum is popped: the earlier minimum disappears too early. Use `<=`. If you store pairs, one stack is enough and pop stays obviously correct.

```cpp
struct MinStack {
    stack<pair<int,int>> st; // value, min
    void push(int x) {
        int m = st.empty() ? x : min(x, st.top().second);
        st.push({x, m});
    }
    void pop() { st.pop(); }
    int top() { return st.top().first; }
    int getMin() { return st.top().second; }
};
```

---

## A15. Next Greater Element I — LeetCode 496

**What it is.** `nums1` is a subset of `nums2`. For each value in `nums1`, find the next greater value to its right in `nums2`, or `-1`.

**Pattern.** Monotonic decreasing stack (stack top is the most recent unresolved element). The general form is “next greater to the right”.

**Steps.**

1. Scan `nums2` from left to right. Stack holds indexes (or values) waiting for a greater element, and the values in the stack stay decreasing.
2. While the current value is greater than the stack top, that top’s next greater is the current value. Pop and record it in a map.
3. Push the current value.
4. Anything left in the stack has no next greater (`-1`).
5. Answer `nums1` by map lookup.

**Complexity.** O(n + m). Each element is pushed and popped once.

**Recognize again.** Daily temperatures, stock span, largest rectangle in histogram, next greater element II (circular: scan the array twice), previous smaller (scan direction or comparison flips).

**Pitfalls.** Increasing stack instead of decreasing, which answers a different question. For circular arrays, iterating `i` from `0` to `2n - 1` and indexing `i % n` is the clean version; do not push the same index twice (only push when `i < n`).

---

## A16. Rotting Oranges — LeetCode 994

**What it is.** Grid: `0` empty, `1` fresh, `2` rotten. Every minute, a rotten orange rots its 4-directional fresh neighbors. Return minutes until all fresh are rotten, or `-1`.

**Pattern.** Multi-source BFS. Time is the BFS layer.

**Steps.**

1. Push every initially rotten cell into a queue. Count fresh oranges.
2. If fresh is already 0, return 0.
3. BFS. For each layer, record the layer size first, then expand only that many cells. That layer is one minute.
4. Rot a fresh neighbor, decrement fresh, enqueue it.
5. After BFS, if fresh remains, return `-1`. Else return minutes (do not count a trailing empty layer; only increment time when the queue still has work, or start time at `-1` / subtract the last empty expansion).

**Complexity.** O(rows · cols).

**Pitfalls.** Single-source BFS from one rotten orange. Incrementing time per cell instead of per layer. Forgetting that simultaneous rotting is one minute, so the initial rotten set must all start in the queue together.

```cpp
int dirs[4][2] = {{1,0},{-1,0},{0,1},{0,-1}};
// queue all rotten; minutes++ once per layer while queue not empty
// after processing a layer, if you rotted nothing new and queue is empty, stop
```

A reliable minute count: `minutes = 0`. While queue is not empty, process one layer. If that layer rotted at least one orange, `minutes++`. At the end return `fresh == 0 ? minutes : -1`.

---

## A17. Longest Common Prefix — LeetCode 14

**What it is.** Longest string that is a prefix of every string in the array.

**Pattern.** Vertical scan, or horizontal reduction. Sorting and comparing only the first and last strings also works because those two differ as early as any pair.

**Steps (vertical).**

1. If the array is empty, return `""`.
2. For column `c` starting at 0: if any string is shorter than `c + 1`, or any character at `c` differs from `strs[0][c]`, return the prefix built so far.
3. Otherwise append that character.

**Complexity.** O(S) where S is the sum of all characters. Early exit helps.

**Pitfalls.** Assuming the shortest string is `strs[0]`. Horizontal fold (`prefix = prefix ∩ next`) is easy to write and fine.

---

## A18. Isomorphic Strings — LeetCode 205

**What it is.** `s` and `t` are isomorphic if each character in `s` maps to exactly one character in `t`, consistently, and two different characters do not share an image. `egg` and `add` are isomorphic. `foo` and `bar` are not.

**Pattern.** Bijection between alphabets. One map is not enough; the mapping must be one-to-one.

**Steps.**

1. Arrays `sToT[256]` and `tToS[256]`, initialized to a sentinel (0 means unmapped; characters are compared carefully — store the mapped char and a separate `used` flag, or store index+1).
2. Walk `i` from 0 to `n - 1`.
3. If `s[i]` is already mapped, it must map to `t[i]`. If `t[i]` is already used by some other `s` character, reject.
4. Otherwise record both directions.

**Complexity.** O(n) time, O(1) space for a fixed alphabet (256). A hash map is the same idea for Unicode.

**Pitfalls.** Mapping only `s → t` accepts `ab` → `aa` (both map to `a`). That is not one-to-one. Equal lengths are required; if lengths differ, return false immediately.

---

## A19. First unique element in a queue / stream

**What it is.** Support a stream of values and, at any time, report the first value that has occurred once so far (the earliest unique). This is the classic “first unique in a stream” design. The static version on a string is LC 387.

**Pattern.** Count map plus a queue of candidates. The queue stores values in arrival order. The front is the answer only while its count is still 1.

**Steps.**

1. `add(x)`: increment `count[x]`. Always push `x` into the queue (even if it is a repeat; stale copies get skipped later).
2. `firstUnique()`: while the queue is not empty and `count[front] != 1`, pop. Return the front, or a sentinel if empty.
3. Lazy deletion is the point. Do not scan the queue on every add.

**Complexity.** Amortized O(1) per add and per query, because each element is pushed and popped once. O(n) space.

**Static string version (LC 387).** One pass to count, second pass to return the first index whose count is 1. That is enough when the data does not arrive online.

**Pitfalls.** Erasing from the middle of a queue (queues cannot do that efficiently). Removing a value from the queue the moment it becomes non-unique requires knowing where it sits; lazy pop avoids that.

---

## A20. Filtering a stack based on original neighbors

**What it is.** A common interview variant: you are given a stack (or you are told to use a stack) of values that originally had left/right neighbors, and you must delete values that fail a condition against **those original neighbors**, not against whoever becomes adjacent after deletions.

**Pattern.** Snapshot the neighbor relation before any deletion. Index-based conditions, then filter. A stack is often the wrong structure if the condition needs both neighbors; an array plus a boolean keep-array is the direct solution. Use a stack only if the condition is “next/previous greater”, which is a monotonic-stack problem in disguise.

**Steps.**

1. Dump the stack into a vector from bottom to top so indexes match original positions. Ends have one neighbor; define the condition for them explicitly (usually they stay, or they compare against the single neighbor).
2. For each middle index `i`, evaluate the predicate using `a[i-1]` and `a[i+1]` from the **original** vector.
3. Build the kept sequence in that same order and reload the stack if the API must return a stack.

**Example predicate.** Remove `a[i]` when it is strictly smaller than both original neighbors. Ends stay. `[4, 1, 3, 2, 5]` → remove 1 (neighbors 4 and 3) but keep 2, because 2 is not smaller than both 3 and 5. Result `[4, 3, 2, 5]`.

**Pitfalls.** Popping and comparing with the new top, which changes the neighbors mid-pass and deletes a chain (the “domino” bug). Another pitfall is reversing the stack accidentally while dumping it; decide bottom vs top once and test it.

**When it actually is a stack algorithm.** “Remove adjacent pairs until none remain” (duplicate letters, LC 1047) **should** use a stack, because the neighbor changes on purpose. Read the prompt: original neighbors → snapshot. Updated neighbors → stack.

---

## A21. Daily Temperatures — LeetCode 739

**What it is.** `temperatures[i]` is the temperature on day `i`. Return an array `wait[i]` = how many days until a strictly warmer day, or 0 if none.

**Pattern.** Monotonic decreasing stack of indexes. Same engine as next greater element, but the stored answer is a distance.

**Steps.**

1. `ans` starts as all zeros. Stack is empty.
2. For `i` from 0 to `n - 1`:
   - While the stack is not empty and `temperatures[i] > temperatures[st.top()]`: `j = st.top()`, pop, `ans[j] = i - j`.
   - Push `i`.
3. Days left on the stack stay 0.

**Complexity.** O(n) time, O(n) space.

**Pitfalls.** Storing temperatures instead of indexes, then being unable to compute the day gap. Using `>=` when the problem asks for strictly warmer, which skips equal days incorrectly or resolves them too early.

---

## A22. Permutation in String — LeetCode 567

**What it is.** Return true if any substring of `s2` is an anagram of `s1` (same length, same character counts).

**Pattern.** Fixed-size sliding window plus a frequency difference. Window length is always `s1.size()`.

**Steps.**

1. If `s1` is longer than `s2`, return false.
2. Build count of `s1`. Maintain count of the current window in `s2`, or maintain one difference array.
3. A window matches when all 26 counts are equal. Track `matches`: how many letters currently agree. Updating a single letter changes at most that letter’s match state.
4. Slide: add the incoming character, remove the outgoing one, check.

**Complexity.** O(n) time, O(1) space (alphabet size 26).

**Pitfalls.** Sorting every window is O(n · k log k). Recomputing a full frequency compare from scratch each slide is still acceptable for 26 letters but hides the O(1) slide. Remember to check the first window before sliding.

```cpp
bool checkInclusion(string s1, string s2) {
    if (s1.size() > s2.size()) return false;
    int need[26] = {}, have[26] = {};
    for (char c : s1) need[c - 'a']++;
    int k = (int)s1.size();
    for (int i = 0; i < (int)s2.size(); i++) {
        have[s2[i] - 'a']++;
        if (i >= k) have[s2[i - k] - 'a']--;
        if (i >= k - 1 && equal(have, have + 26, need)) return true;
    }
    return false;
}
```

---

## A23. Topological sort

**What it is.** A linear order of vertices in a directed graph such that every edge `u → v` has `u` before `v`. Exists if and only if the graph is a DAG (no directed cycle).

**Pattern.** Kahn’s algorithm (BFS + indegree) or DFS finish times (post-order reversed).

**Recognize when.** Course prerequisites, build systems, “task A before task B”, alien dictionary order, compiling a dependency graph.

**DFS version steps.**

1. States: unvisited, in stack (gray), done (black).
2. DFS. A gray node seen again means a back edge, so a cycle: no topo order.
3. Append a node after visiting all its outgoing neighbors (post-order).
4. Reverse the recorded list. That reverse post-order is a valid topological order.

**Output is not unique.** Any order that respects edges is correct. Tests that accept any valid order must be checked that way.

**Pitfalls.** Forgetting reverse on the DFS version (you produced the opposite order). Treating an undirected edge as a dependency. Not detecting cycles and returning a partial order that the judge will reject (LC 210, Course Schedule II, wants an empty array if impossible).

See A24 for the BFS version, which is usually easier to debug.

---

## A24. Kahn’s algorithm

**What it is.** Topological sort by repeatedly removing nodes with indegree 0.

**Steps.**

1. Build adjacency list `u → v` meaning `u` precedes `v`. Compute `indegree[v]` for every node.
2. Queue every node with indegree 0.
3. While the queue is not empty: pop `u`, append `u` to the order. For each neighbor `v`, decrement `indegree[v]`. If it hits 0, push `v`.
4. If the order contains fewer than `n` nodes, there is a cycle.

**Complexity.** O(n + m) time.

**Why the cycle check works.** Every node in a cycle has indegree at least 1 forever, so it never enters the queue.

**Pitfalls.** Building the edge backwards (`v → u` when the statement says “v depends on u” / “u is a prerequisite of v”). Off-by-one if nodes are numbered `0..n-1` vs `1..n`. Using a stack instead of a queue still yields a valid topo order; BFS just produces a level-by-level one. Stable tie-breaking needs a priority queue instead of a plain queue (smallest label first).

```cpp
vector<int> topo(int n, vector<vector<int>>& edges) { // edge: prereq -> course
    vector<vector<int>> g(n);
    vector<int> indeg(n);
    for (auto& e : edges) { g[e[0]].push_back(e[1]); indeg[e[1]]++; }
    queue<int> q;
    for (int i = 0; i < n; i++) if (indeg[i] == 0) q.push(i);
    vector<int> order;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        order.push_back(u);
        for (int v : g[u]) if (--indeg[v] == 0) q.push(v);
    }
    if ((int)order.size() != n) return {};
    return order;
}
```

---

## A25. DFS and BFS

**What they are.** Two ways to walk a graph or tree. DFS goes deep (stack or recursion). BFS goes layer by layer (queue).

**Use BFS when.**

- Unweighted shortest path (fewest edges, fewest steps, minimum knight moves).
- Multi-source spread (rotting oranges, walls and gates).
- Level order of a tree.

**Use DFS when.**

- Detecting a cycle in a directed graph (colors).
- Topological post-order.
- Connected components, flood fill, islands (either works; DFS is shorter).
- Backtracking (permutations, word search, maze with choices you must undo).
- Tree answers that combine children’s return values (height, diameter).

**Graph DFS must mark visited when you first see the node**, before pushing neighbors, or an undirected edge pair will enqueue the same node twice and explode. For cycle detection in directed graphs, a simple visited set is not enough: you need three colors so a back edge into the current stack is different from an edge into an already finished node (that cross edge is fine).

**BFS shortest path sketch.**

```cpp
vector<int> dist(n, -1);
queue<int> q;
dist[src] = 0; q.push(src);
while (!q.empty()) {
    int u = q.front(); q.pop();
    for (int v : g[u]) if (dist[v] == -1) {
        dist[v] = dist[u] + 1;
        q.push(v);
    }
}
```

**Pitfalls.** Recursive DFS on a chain of 1e5 nodes blows the call stack; use an explicit stack or raise the limit only in contests where that is allowed. BFS with a `visited` mark **after** dequeue (instead of on enqueue) turns the queue into an exponential duplicate factory. On grids, decide 4-direction vs 8-direction before coding.

**Parent / path reconstruction.** Store `parent[v] = u` when you first discover `v`. Walk parents from the target back to the source, then reverse.

---

## A26. AVL rotations (LL, RR, LR, RL)

**What it is.** An AVL tree is a BST whose balance factor at every node is in `{-1, 0, 1}`. Balance factor = height(left) − height(right). Insertions and deletions restore balance with rotations.

**Pattern.** Fix the **lowest** unbalanced node on the insertion path. The shape of the three nodes (node, child, grandchild) names the case.

**Heights.** Null height is `-1` or `0` depending on the convention you pick. Stick to one. With null height `-1`, a leaf has height 0 and balance 0.

**LL (left-left).** Inserted into the left subtree of the left child. Balance of node is `+2`, balance of left child is `≥ 0`. **Right rotation** on the node.

```
        z                y
       /                / \
      y       →        x   z
     /
    x
```

Right rotation on `z`: `y` becomes root, `z` takes `y`’s right subtree as its left.

**RR (right-right).** Mirror of LL. Balance of node is `-2`, right child balance `≤ 0`. **Left rotation** on the node.

**LR (left-right).** Inserted into the right subtree of the left child. Node balance `+2`, left child balance `< 0`. **Left rotation on the left child**, then **right rotation on the node**.

**RL (right-left).** Node balance `-2`, right child balance `> 0`. **Right rotation on the right child**, then **left rotation on the node**.

**Memory hook.** The first letter is where the imbalance leans (L means heavy on the left). The second letter says whether a preliminary rotation is needed. Same two letters (LL, RR): one rotation, opposite direction of the letters (LL → right rotate). Different letters (LR, RL): rotate the child first to turn it into LL or RR, then do that case.

**After every rotation.** Recompute heights bottom-up: child first, then parent. A single insert needs at most two rotations (one double rotation) to restore the whole tree; you can stop at the first fix. Deletion may require fixes all the way to the root.

**Pitfalls.** Updating height before rewiring pointers. Rotating the wrong node (a higher ancestor) when a lower one is the real imbalance. Forgetting that LR is not “a single magical rotation”: it is two single rotations. BST inorder must be unchanged by every rotation; test that.

```cpp
struct Node { int key, height; Node *l, *r; };
int h(Node* n) { return n ? n->height : -1; }
void pull(Node* n) { n->height = 1 + max(h(n->l), h(n->r)); }
int bal(Node* n) { return n ? h(n->l) - h(n->r) : 0; }

Node* rotR(Node* z) {          // LL fix
    Node* y = z->l;
    z->l = y->r; y->r = z;
    pull(z); pull(y);
    return y;
}
Node* rotL(Node* z) {          // RR fix
    Node* y = z->r;
    z->r = y->l; y->l = z;
    pull(z); pull(y);
    return y;
}
// after recursive insert returns the new subtree root:
// b = bal(node)
// if (b > 1 && key < node->l->key) return rotR(node);          // LL
// if (b < -1 && key > node->r->key) return rotL(node);         // RR
// if (b > 1 && key > node->l->key) { node->l = rotL(node->l); return rotR(node); } // LR
// if (b < -1 && key < node->r->key) { node->r = rotR(node->r); return rotL(node); } // RL
```

---

## A27. Minimize Maximum Pair Sum in Array — LeetCode 1877

**What it is.** Pair up all `2n` elements. Pair sum is `nums[i] + nums[j]`. Minimize the maximum pair sum.

**Pattern.** Greedy sort + pair ends. Also a classic “minimize the maximum”, which you can binary-search, but the sort proof is direct.

**Why pairing smallest with largest works.** Suppose an optimal pairing matches the largest element with something that is not the smallest. Swapping partners so the largest meets the smallest cannot increase the maximum pair sum: the largest element’s pair only gets smaller or stays, and the displaced partner is now with a smaller mate. Repeat.

**Steps.**

1. Sort ascending.
2. Pair `nums[i]` with `nums[n - 1 - i]` for `i` in `0 .. n/2 - 1` (here `n` is the full length).
3. The answer is the max of those sums.

**Complexity.** O(n log n) for the sort.

**Pitfalls.** Pairing adjacent elements after sorting (`1+2`, `3+4`) maximizes the chance that two large values meet. That is the opposite goal.

**Generalize.** “Minimize the maximum” often allows binary search on the answer when a direct pairing proof is hard: search `mid` = allowed maximum, greedily test whether a pairing exists under that cap.

---

## A28. Shortest path algorithms

Pick by the edge weights and by whether you need one source or all pairs.

| Algorithm | Graph | Time (typical) | Use |
| --- | --- | --- | --- |
| BFS | unweighted, or equal weights | O(n + m) | fewest edges |
| 0-1 BFS | edges weight 0 or 1 | O(n + m) | deque: 0-weight to front |
| Dijkstra | non-negative weights | O(m log n) with a heap | one source, GPS-style |
| Bellman-Ford | negative edges ok | O(n · m) | one source, or detect negative cycle |
| Floyd-Warshall | any weights, no negative cycle for distances | O(n^3) | all pairs, n ≤ ~400 |
| DAG shortest path | directed acyclic | O(n + m) | relax in topological order |

**Parent pointers** reconstruct the path. **Distance initialized to +infinity**, source to 0. Do not use a visited array that blocks re-processing in Bellman-Ford or SPFA; Dijkstra may settle a node once **only because weights are non-negative**.

**Negative cycle.** If a node on the path can loop and reduce distance forever, no shortest path exists. Bellman-Ford’s extra nth pass detects this. Floyd detects it when some `dist[i][i] < 0`.

Cards C21–C23 expand Dijkstra, Bellman-Ford, and Floyd.

---

## A29. Split Linked List in Parts — LeetCode 725

**What it is.** Split the list into `k` consecutive parts. Sizes differ by at most 1. Earlier parts take the extra nodes. `k` may exceed the length; extra parts are null.

**Pattern.** Count, then compute base size and remainder. Walk once, cutting links.

**Steps.**

1. Count `n`.
2. `base = n / k`, `extra = n % k`. The first `extra` parts have length `base + 1`. The rest have length `base`.
3. For each part `i`: if length is 0, store null. Else the current node is the part head. Walk `len - 1` steps, cut `curr->next = nullptr`, and continue from the saved next.

**Complexity.** O(n + k) time. The answer array has size k.

**Pitfalls.** Giving later parts the extra node (the problem wants earlier parts longer). Forgetting to null-terminate a part, so parts overlap and share tails. When `k > n`, `base` is 0 and only the first `n` parts get one node.

---

## A30. Longest Mountain in Array — LeetCode 845

**What it is.** A mountain is a subarray that strictly increases, then strictly decreases, with length at least 3. Return the longest such length, or 0.

**Pattern.** One pass tracking the current uphill run and downhill run. Alternatively, for each peak expand with two pointers (O(n) total if you only expand from strict peaks).

**Steps (state machine).**

1. Walk `i` from 1 to `n - 1`, with `up = 0`, `down = 0`, `best = 0`.
2. If `a[i] > a[i - 1]`: if `down > 0`, a previous mountain just ended, so reset `up = 0`. Then `up++`, `down = 0`.
3. Else if `a[i] < a[i - 1]`: `down++` only if `up > 0` (you cannot start a mountain by descending).
4. Else (equal): reset `up = down = 0`. A plateau kills the mountain.
5. Whenever `up > 0 && down > 0`, `best = max(best, up + down + 1)`.

**Complexity.** O(n) time, O(1) space.

**Pitfalls.** Counting a purely increasing array as a mountain. Allowing plateaus (`==`) inside the slope. Starting `down` before any `up`. The `+1` is the peak itself; `up` and `down` are edge counts, not node counts.

**Example.** `[2,1,4,7,3,2,5]`: the mountain `[1,4,7,3,2]` has length 5.

---

# Page B

## B1. Fibonacci Sequence — LeetCode 509

**What it is.** `F(0) = 0`, `F(1) = 1`, `F(n) = F(n-1) + F(n-2)`.

**Pattern.** The smallest DP. Also matrix exponentiation or fast doubling if n is huge.

**Steps (iterative).** Keep only `a = F(i-2)`, `b = F(i-1)`. For `i` from 2 to n: `c = a + b`, then shift. Return `b` after n steps (handle n = 0).

**Complexity.** O(n) time, O(1) space. Naive recursion is O(φ^n) and exists to illustrate why caching matters.

**Pitfalls.** Recursion without memo on n = 45 already times out. Integer overflow: LeetCode 509 fits in 32-bit for its constraints; larger n needs big integers or `long long`.

```cpp
int fib(int n) {
    if (n < 2) return n;
    int a = 0, b = 1;
    for (int i = 2; i <= n; i++) { int c = a + b; a = b; b = c; }
    return b;
}
```

---

## B2. Climbing Stairs — LeetCode 70

**What it is.** Each step you climb 1 or 2 stairs. How many distinct ways to reach `n`?

**Pattern.** Fibonacci in disguise. `ways(n) = ways(n-1) + ways(n-2)`, with `ways(1) = 1`, `ways(2) = 2` (or `ways(0) = 1`).

**Recognize when.** “Ordered ways to sum to n with coins {1, 2}” where order matters (1 then 2 is different from 2 then 1). That is permutation-style counting, which this recurrence already handles. Unordered combinations are a different DP (coin change combinations).

**Steps.** Same loop as Fibonacci. Answer for n is `F(n+1)` if `F(1) = 1`, `F(2) = 1`.

**Pitfalls.** Mixing this up with min-cost climbing (optimization, not counting) and with coin change (unordered, and usually a minimum count).

---

## B3. Min Cost Climbing Stairs — LeetCode 746

**What it is.** `cost[i]` is paid when you step **on** stair `i`. From there you may jump 1 or 2. You may start at index 0 or 1. Reach “past the last stair”. Return the minimum cost.

**Pattern.** 1D DP. `dp[i]` = min cost to stand at i, already having paid `cost[i]`, or alternatively min cost to reach i before paying. Pick one definition and do not mix.

**Definition used here.** `dp[i]` = min cost to reach index i (not yet paying `cost[i]`). The top is index `n` (one past the end).

**Steps.**

1. `dp[0] = dp[1] = 0` (you can start at either for free).
2. For `i` from 2 to n: `dp[i] = min(dp[i-1] + cost[i-1], dp[i-2] + cost[i-2])`.
3. Return `dp[n]`.

You only need two rolling variables.

**Complexity.** O(n) time, O(1) space.

**Pitfalls.** Paying the cost of the destination past the end (there is no stair there). Charging `cost[i]` when arriving at i and also when leaving i.

---

## B4. House Robber — LeetCode 198

**What it is.** You cannot rob two adjacent houses. Maximize the sum.

**Pattern.** Linear DP with a skip/take choice. `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`.

**Meaning.** `dp[i]` is the best loot using the prefix `nums[0..i]`.

**Steps.**

1. If one house, return it.
2. `prev2 = nums[0]`, `prev1 = max(nums[0], nums[1])`.
3. For `i` from 2 to n-1: `cur = max(prev1, prev2 + nums[i])`, then shift.
4. Return `prev1`.

**Complexity.** O(n), O(1).

**Pitfalls.** `prev2 + nums[i]` uses the house two back, which is non-adjacent, so it is legal. Adding to `prev1` is the bug. A left-to-right “take it whenever you can” greedy fails on `[2, 1, 1, 2]`: it takes the first 2 and the later 1 (total 3). Robbing both ends yields 4. Trust the recurrence.

---

## B5. House Robber II — LeetCode 213

**What it is.** Houses form a circle. The first and last are adjacent, so they cannot both be robbed.

**Pattern.** Reduce a circular DP to two linear DPs.

**Steps.**

1. If `n == 1`, return `nums[0]`.
2. Answer = max( rob linear range `[0, n-2]` , rob linear range `[1, n-1]` ).
3. Use the House Robber I function on each slice.

**Why this covers every optimal plan.** Any optimal plan omits at least one of {first, last}. The better of “omit last” and “omit first” includes that plan. Plans that omit both are considered inside both ranges and simply will not win unless they are optimal.

**Pitfalls.** Running House Robber I on the whole array. Special-casing n = 1 incorrectly by taking `max` of two empty ranges. Overlapping both ends in a third case is unnecessary.

---

## B6. Sliding Window Maximum — LeetCode 239

**What it is.** Array and window size `k`. Return the max of every contiguous window of length k.

**Pattern.** Monotonic deque of indexes, values decreasing from front to back. Front is the index of the current window max.

**Steps.**

1. Deque will store indexes. For each `i`:
2. Pop front while `dq.front() <= i - k` (outside the window).
3. Pop back while `nums[dq.back()] <= nums[i]` (those can never be the max while `i` is in the window; they are smaller and older).
4. Push `i`.
5. If `i >= k - 1`, record `nums[dq.front()]`.

**Complexity.** O(n). Each index enters and leaves the deque once.

**Pitfalls.** Storing values without indexes, so you cannot expire the left edge. Using `<` instead of `<=` when popping the back, which keeps useless duplicates. A heap also works but is O(n log k) and needs lazy deletion of indexes that slid out.

---

## B7. Container With Most Water — LeetCode 11

**What it is.** Vertical lines at x = 0..n-1 with heights `height[i]`. Pick two lines; the area is `min(height[i], height[j]) * (j - i)`. Maximize area.

**Pattern.** Two pointers at the ends. Move the shorter side inward.

**Why moving the shorter side is safe.** Area is limited by the short line. Moving the tall side inward cannot beat the current area for this short line, because width shrinks and the min cannot increase above the short line. So the short line is finished; advance it.

**Steps.**

1. `l = 0`, `r = n - 1`, `best = 0`.
2. While `l < r`: `best = max(best, min(h[l], h[r]) * (r - l))`. If `h[l] < h[r]` then `l++`, else `r--`.

**Complexity.** O(n) time, O(1) space.

**Pitfalls.** Brute force pairs are O(n^2). Moving both pointers every iteration skips candidates. When heights are equal, moving either side is fine for correctness of this problem.

---

## B8. Bubble sort and selection sort

**Bubble sort.** Repeatedly sweep left to right, swapping adjacent inversions. After pass `i`, the last `i` elements are in final position (if you stop the inner loop early). Best case O(n) with a “did I swap?” flag on an already sorted array. Worst and average O(n^2). Stable. In-place.

**Selection sort.** For index `i`, scan `i..n-1`, swap the minimum into `i`. Always O(n^2), even if sorted. In-place. Unstable if you swap (the swapped element can jump over equals). At most n swaps, which matters when writes are expensive.

**When either shows up in interviews.** Usually as a warmup, or as the contrast against merge/quick/heap (O(n log n)). Do not use them as the solution to a “sort then scan” problem unless n is tiny; use `std::sort`.

**Insertion sort** is the other O(n^2) classic and the one that is actually used: it is O(n + inversions), stable, and fast on nearly sorted data. Timsort and introsort use it for small runs.

---

## B9. Sort Colors — LeetCode 75

**What it is.** Array of 0, 1, 2 only. Sort it in-place. This is the Dutch national flag problem.

**Pattern.** Three-way partition with pointers `lo`, `mid`, `hi`.

**Invariant.**

- `[0, lo)` are 0s
- `[lo, mid)` are 1s
- `[mid, hi]` are unknown
- `(hi, n)` are 2s

**Steps.**

1. `lo = mid = 0`, `hi = n - 1`.
2. While `mid <= hi`:
   - `nums[mid] == 0`: swap with `lo`, `lo++`, `mid++`.
   - `nums[mid] == 1`: `mid++`.
   - `nums[mid] == 2`: swap with `hi`, `hi--`. Do **not** increment `mid`; the swapped-in value is unexamined.

**Complexity.** O(n) time, O(1) space. Counting sort (count 0/1/2, rewrite) is also O(n) and easier; the pointer version is what “one pass, constant space, in place” usually means.

**Pitfalls.** Incrementing `mid` after a swap with `hi`. Using `mid < hi` and leaving `nums[hi]` unprocessed.

---

## B10. Find All Numbers Disappeared in an Array — LeetCode 448

**What it is.** `nums` has length n and values in `1..n`. Some numbers appear twice, some never. Return the missing ones. Follow-up wants O(1) extra space (the output array does not count).

**Pattern.** Use the index as a hash. The value `x` belongs at index `x - 1`. Mark presence by negating `nums[x - 1]`.

**Steps.**

1. For each `x` in the array, let `idx = abs(x) - 1`. If `nums[idx] > 0`, set `nums[idx] = -nums[idx]`.
2. A second pass: if `nums[i] > 0`, then `i + 1` never appeared.
3. The signs are the marks. Absolute value restores the original number when you need it as an index.

**Complexity.** O(n) time, O(1) extra space.

**Alternative.** Cyclic swaps until `nums[i] == i + 1` or the target slot already holds the right value. Then a scan finds mismatches. Same pattern as “missing and duplicate” problems.

**Pitfalls.** Negating twice (toggling) instead of negating only if positive, which loses the mark when a duplicate is seen. Index `x` instead of `x - 1` when values are 1-based.

---

## B11. 3Sum Closest — LeetCode 16

**What it is.** Find three integers whose sum is as close as possible to `target`. Return that sum. Exactly one solution is typical in the statement’s guarantee, but the algorithm does not need uniqueness.

**Pattern.** Sort, then two pointers inside a fixed left index. Same skeleton as 3Sum (LC 15), except you track the closest sum instead of exact zeros.

**Steps.**

1. Sort. `best` = sum of the first three (or a large sentinel difference).
2. For `i` from 0 to n-3:
   - `l = i + 1`, `r = n - 1`.
   - While `l < r`: `sum = a[i] + a[l] + a[r]`. If `abs(sum - target)` is smaller, update `best`.
   - If `sum < target`, `l++` (need a larger sum). If `sum > target`, `r--`. If equal, return `target` immediately.
3. Return `best`.

**Complexity.** O(n^2).

**Pitfalls.** Returning the difference instead of the sum. Updating the best only on exact logic copied from 3Sum (skipping duplicates is optional here; it does not change the answer). Overflow is unlikely at LeetCode constraints but `long long` sums are a cheap habit.

---

## B12. GCD of two numbers

**What it is.** Greatest common divisor. Euclidean algorithm: `gcd(a, b) = gcd(b, a % b)`, and `gcd(a, 0) = a`.

**In C++.** `std::gcd(a, b)` from `<numeric>` (C++17).

**GCD of an entire array.** Fold: `g = 0; for (int x : a) g = std::gcd(g, x);`. This is the true common divisor of every element.

**LeetCode 1979** asks only for `gcd(min element, max element)`. That is a different problem. Do not substitute it for the fold unless the statement says so.

**LCM.** `lcm(a, b) = a / gcd(a, b) * b`. Divide before multiplying or 64-bit values overflow. `lcm` of an array can overflow even when the mathematical answer fits a bigger type; check constraints.

**Complexity.** O(log min(a, b)).

**Pitfalls.** `gcd` of negatives: define the result as non-negative. Recursive `%` with negative numbers in C++ truncates toward zero; prefer non-negative inputs. `a / gcd * b` still overflows if you do it in `int`.

```cpp
int gcd(int a, int b) { while (b) { int t = a % b; a = b; b = t; } return a; }
```

---

## B13. Coin Change — LeetCode 322

**What it is.** Coin values (unlimited supply) and an amount. Fewest coins that sum to the amount, or `-1` if impossible.

**Pattern.** Unbounded knapsack / bottom-up DP. `dp[x]` = fewest coins to make amount `x`.

**Steps.**

1. `dp[0] = 0`. Every other `dp[x] = infinity` (use `amount + 1` as infinity; it cannot be a real answer).
2. For `x` from 1 to amount: for each coin `c` if `c <= x`, `dp[x] = min(dp[x], dp[x - c] + 1)`.
3. If `dp[amount]` is still infinity, return `-1`.

**Complexity.** O(amount · coins) time, O(amount) space.

**This is not the combination-count problem.** LC 518 counts unordered combinations and iterates coins outside, amount inside, so each combination is counted once. LC 322’s loop order does not matter for a minimum. If the question is “number of ways, order matters”, it becomes permutation counting (like climbing stairs with arbitrary steps).

**Pitfalls.** Initializing `dp` to 0, so the min never moves. Using `INT_MAX` then adding 1, which overflows. Greedy “largest coin first” is not correct in general. Coins `[1, 5, 6]` and amount `10`: greedy takes `6 + 1 + 1 + 1 + 1` (5 coins) while `5 + 5` uses 2. Use the DP unless the prompt proves the coin system is canonical.

---

## B14. Rat in a Maze

**What it is.** `n × n` grid of open cells (1) and walls (0). Rat starts at `(0,0)` and must reach `(n-1, n-1)`. Moves are usually only right and down, sometimes 4 directions. Return any path, or all paths, as sequences of moves (`DDRR`) or as a solution matrix.

**Pattern.** DFS backtracking. Mark the cell visited when you enter, unmark when you leave, so a cell is not reused in the same path.

**Steps.**

1. If start or end is blocked, return no path.
2. `dfs(r, c, path)`:
   - Out of bounds, wall, or already in this path: return.
   - If `(r, c)` is the end: record `path`, return.
   - Mark visited. Try the four directions in the required order (lexicographic order of direction letters matters if the judge wants sorted paths: D, L, R, U is a common required order).
   - Unmark visited.
3. The path string appends the direction letter before the call and pops after.

**Complexity.** Worst case exponential in the number of open cells. Fine for n ≤ 10 or so, which is the usual constraint.

**Pitfalls.** Forgetting to unmark (then only one path is explored, and even that can get stuck). Allowing a revisit, which cycles. Printing the path in reverse. If moves are only right and down, visited marks are unnecessary because you cannot loop; the problem gets easier and DP can count paths.

**Difference from word search.** Same backtracking shape. Maze asks for a route to a fixed cell. Word search asks whether a string can be spelled by adjacent letters.

---

## B15. Steps by Knight (minimum knight moves)

**What it is.** Chessboard, start square, target square. A knight moves in the eight L shapes `(±1, ±2)` and `(±2, ±1)`. Return the minimum number of moves. Infinite chessboard variants also exist (LC 1197, Minimum Knight Moves).

**Pattern.** BFS on an unweighted graph. Each square is a node; each legal move is an edge of length 1.

**Steps.**

1. Queue of `{row, col, dist}`. Mark visited on enqueue.
2. If the popped square is the target, return dist.
3. Push all eight moves that stay inside the board and are unvisited, with `dist + 1`.

**Complexity.** O(rows · cols) for a finite board.

**Infinite board (LC 1197).** Shift the start to the origin and search only a box a little larger than the target (`max(|tx|, |ty|) + 2` is a safe bound). Known short cases that punish a too-tight bound or a greedy “always reduce the larger coordinate” rule: `(1, 1)` takes 2 moves, `(2, 2)` takes 4, `(5, 5)` takes 4. BFS inside that box gets these right. A knight’s eight moves are `(±1, ±2)` and `(±2, ±1)`, not the eight adjacent king moves.

**Pitfalls.** DFS “first time you reach the target”, which is not shortest. Marking visited on dequeue. Treating the knight as a king (8 adjacent squares) or as a limited set of 2 moves.

---

## B16. Check If a String Is a Valid Sequence from Root to Leaves — LeetCode 1430

**What it is.** Array `arr` should match some root-to-**leaf** path exactly: same values, same length, and the last node is a leaf.

**Pattern.** DFS carrying the index into `arr`.

**Steps.**

1. `dfs(node, i)`:
   - Null node: false.
   - `i` out of range, or `node->val != arr[i]`: false.
   - If `node` is a leaf: return `i == arr.size() - 1`.
   - Else return `dfs(left, i+1)` or `dfs(right, i+1)`.
2. Start `dfs(root, 0)`.

**Pitfalls.** Accepting a path that matches the array but stops at an internal node. Continuing past the array length. Comparing only values and not the exact end.

A leaf is a node with no left and no right child. A null child is not a leaf.

---

## B17. Binary Tree Longest Consecutive Sequence — LeetCode 298

**What it is.** Longest downward path (parent to child only) where each next value is exactly `parent + 1`. Return the length in nodes.

**Pattern.** DFS that passes the parent’s value and the length of the consecutive run ending at the parent.

**Steps.**

1. `dfs(node, parentVal, length)` returns nothing; it updates a global best. Or return the best in this subtree.
2. If `node->val == parentVal + 1`, `length++`, else `length = 1`.
3. `best = max(best, length)`.
4. Recurse to children with `(node->val, length)`.

**Complexity.** O(n).

**Pitfalls.** Allowing child-to-parent or any-direction paths (that harder problem is LC 549). Counting edges instead of nodes. Treating “increasing by any positive amount” as consecutive; the problem wants `+1` exactly.

LC 298 is the downward-only version. If the prompt says “either direction”, switch to 549: each node returns the longest increasing and decreasing chain downward, and the answer through that node adds the two directions carefully so directions differ.

---

## B18. Count Nodes Equal to Sum of Descendants — LeetCode 1973

**What it is.** Count nodes whose value equals the sum of values in their proper descendants (not including themselves). Leaves have descendant sum 0, so a leaf counts only if its value is 0.

**Pattern.** Post-order DFS. The function returns the sum of the subtree **including** the node. The parent uses children’s returned sums as the descendant sum.

**Steps.**

1. `dfs(node)` returns subtree sum including node.
2. Null returns 0.
3. `left = dfs(node->left)`, `right = dfs(node->right)`.
4. If `node->val == left + right`, increment the answer.
5. Return `node->val + left + right`.

**Complexity.** O(n). Use `long long` for the returned subtree sums. With up to `1e5` nodes and values up to `1e5`, the sum does not fit in 32-bit `int`.

**Pitfalls.** Comparing against the sum that already includes the node (`left + right + val == val` only when descendants sum to 0). Forgetting that “descendants” excludes the node.

---

## B19. Jump Game — LeetCode 55

**What it is.** `nums[i]` is the maximum forward jump length from i. Start at 0. Can you reach the last index?

**Pattern.** Greedy reach. Track the farthest index you can touch.

**Steps.**

1. `far = 0`.
2. For `i` from 0 to `n - 1`: if `i > far`, return false (you never reached i). `far = max(far, i + nums[i])`. If `far >= n - 1`, return true.
3. Return true.

**Complexity.** O(n).

**Pitfalls.** DFS of every jump, which is exponential. Confusing this with Jump Game II (that one asks for the minimum number of jumps, not a boolean). A zero at index i blocks progress only if `far` cannot get past i.

---

## B20. Jump Game II — LeetCode 45

**What it is.** Same jump rule. Return the minimum number of jumps to reach the last index. You can assume it is reachable.

**Pattern.** Greedy BFS layers. Each “layer” is the range of indexes reachable with exactly `jumps` jumps. Inside the current layer, compute how far the next layer extends.

**Steps.**

1. `jumps = 0`, `curEnd = 0`, `far = 0`.
2. For `i` from 0 to `n - 2` (stop before the last index; you do not need to jump from there):
   - `far = max(far, i + nums[i])`.
   - If `i == curEnd`: you must jump now, `jumps++`, `curEnd = far`.
3. Return `jumps`.

**Why it is BFS.** `curEnd` is the end of the current queue layer. When you finish the layer, one jump takes you to the farthest anything in that layer could see.

**Complexity.** O(n) time, O(1) space. DP `dp[i] = min jumps to i` is O(n^2) and correct but slower.

**Pitfalls.** Incrementing jumps on every index. Looping through `i < n` and counting an extra jump when `i == n - 1` coincides with `curEnd`.

---

## B21. Minimum Cost to Cut a Stick — LeetCode 1547

**What it is.** Stick of length `n`. Cuts must be made at positions in `cuts`. The cost of a cut is the length of the current segment you cut. Order the cuts to minimize total cost.

**Pattern.** Interval DP. Add the ends `0` and `n`, sort the cut points. `dp[l][r]` is the min cost to perform every cut strictly between `cuts[l]` and `cuts[r]`.

**Steps.**

1. Insert `0` and `n` into the cut list. Sort. Let there be `m` points now (original cuts plus 2).
2. `dp[i][j]` = 0 if there is no cut strictly between i and j (`j == i + 1`).
3. For interval length from 2 up to `m - 1`: for each `i`, `j = i + len`. Cost to separate `i` from `j` is `cuts[j] - cuts[i]`, plus the min over the first cut `k` in `(i, j)` of `dp[i][k] + dp[k][j]`.
4. Answer is `dp[0][m - 1]`.

**Meaning of the added length.** Whenever you cut a segment whose endpoints are currently `cuts[i]` and `cuts[j]`, you pay `cuts[j] - cuts[i]`, regardless of which inner cut you choose. Then that segment splits into two independent subproblems.

**Complexity.** O(m^3) time, O(m^2) space, m ≤ 102 because cuts are at most 100.

**Pitfalls.** Forgetting to add `0` and `n`. Sorting is required; the interval DP assumes left-to-right order. Computing cost as the original `n` on every cut overcharges after the stick has split.

---

## B22. Union-Find and Kruskal’s algorithm

**Union-Find (DSU).** Tracks disjoint sets.

- `find(x)` returns the representative of x’s set. Path compression: `parent[x] = find(parent[x])`.
- `unite(a, b)` links the two roots. Union by rank or by size keeps trees shallow.
- Together these are essentially inverse-Ackermann, which is constant for all practical n.

**Use DSU when.** “Are a and b connected after these merges?”, number of components, cycle detection in an undirected graph (an edge whose ends already share a parent closes a cycle), accounts merging, LC 547, 684, 721, 1319.

```cpp
struct DSU {
    vector<int> p, r;
    DSU(int n) : p(n), r(n, 0) { iota(p.begin(), p.end(), 0); }
    int find(int x) { return p[x] == x ? x : p[x] = find(p[x]); }
    bool unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return false;
        if (r[a] < r[b]) swap(a, b);
        p[b] = a;
        if (r[a] == r[b]) r[a]++;
        return true;
    }
};
```

**Kruskal’s MST.**

1. Sort undirected edges by increasing weight.
2. Add an edge if its endpoints are in different components (`unite` returns true).
3. Stop after `n - 1` edges. If you cannot, the graph is disconnected (no spanning tree; a minimum spanning forest is what you have).

**Why it is correct.** Cut property: the lightest edge across any cut is in some MST. Kruskal adds the globally lightest edge that does not form a cycle, which is exactly that idea.

**Complexity.** O(m log m) for the sort, plus nearly O(m) DSU.

**Pitfalls.** Using Kruskal on a directed graph. Forgetting path compression and timing out on n = 1e5 with a skewed tree. Sorting descending builds a maximum spanning tree, which is sometimes what you want (negate weights or flip the comparator on purpose).

Prim and Kruskal both produce an MST. Kruskal is better when the graph is sparse and already given as an edge list. Prim is natural on dense graphs with an adjacency matrix.

---

## B23. Boundary traversal of a binary tree

**What it is.** Print the boundary anti-clockwise (the usual convention):

1. Left boundary top-down, excluding leaves.
2. All leaves left to right.
3. Right boundary bottom-up, excluding leaves.

The root is not repeated if it is also a leaf. Leaves on the left or right boundary are printed in the leaf pass only, so they are not duplicated.

**Pattern.** Three separate walks. Do not try to do this with one clever traversal.

**Steps.**

1. If root is null, empty result. Add root if it is not a leaf (or always add root, and skip it later in both boundary walks and the leaf walk — pick one policy).
2. **Left boundary.** Start at `root->left`. While the node exists: if it is not a leaf, record it. Prefer going left; if no left child, go right. Stop at a leaf (do not record it).
3. **Leaves.** DFS or inorder. A leaf has neither child. Record it. This includes leaves that sit on the boundary.
4. **Right boundary.** Collect into a temporary list the same way (prefer right, else left, skip leaves), then append that list **reversed**.

**Pitfalls.** Recording a leaf during the left-boundary walk and again during the leaf walk. Walking the right boundary top-down into the answer (the required order is bottom-up). On a one-sided tree, “prefer left else right” is what keeps the boundary on the outer silhouette; always-left will fall off.

**Clockwise** is the mirror: right boundary first, leaves right-to-left, left boundary bottom-up. Confirm the required direction in the prompt.

---

## B24. Check whether a graph is bipartite

**What it is.** Nodes can be colored with two colors so every edge connects different colors. Equivalently, no odd-length cycle.

**Pattern.** BFS or DFS 2-coloring. Unvisited components each need their own start.

**Steps.**

1. `color[i] = -1` means uncolored. Colors are 0 and 1.
2. For each node `s` still uncolored: color it 0, BFS/DFS.
3. For a neighbor `v`: if uncolored, color it `1 - color[u]` and continue. If colored and `color[v] == color[u]`, return false.
4. If you finish all components, return true.

**Complexity.** O(n + m).

**Pitfalls.** Checking only one component. Self-loops make a graph non-bipartite. Undirected edges stored once but traversed only one way: store both directions. In an undirected graph the parent appears as a neighbor; coloring logic handles it because the parent has the opposite color, so it will not false-alarm. Do not special-case the parent unless your graph is directed.

LC 785 is this problem.

```cpp
bool isBipartite(vector<vector<int>>& g) {
    int n = (int)g.size();
    vector<int> color(n, -1);
    for (int s = 0; s < n; s++) {
        if (color[s] != -1) continue;
        queue<int> q; q.push(s); color[s] = 0;
        while (!q.empty()) {
            int u = q.front(); q.pop();
            for (int v : g[u]) {
                if (color[v] == -1) { color[v] = color[u] ^ 1; q.push(v); }
                else if (color[v] == color[u]) return false;
            }
        }
    }
    return true;
}
```

---

## B25. Longest Increasing Subsequence — LeetCode 300

**What it is.** Longest subsequence (not necessarily contiguous) that is strictly increasing. Return its length.

**Pattern.** Patience sorting: `tails[len]` = smallest possible tail of any increasing subsequence of length `len + 1`.

**Steps.**

1. `tails` is a vector, initially empty.
2. For each `x`: if `x` is greater than every tail, append it (LIS grew). Else replace the first tail that is `≥ x` (`lower_bound`).
3. The length of `tails` is the LIS length. `tails` itself is **not** the subsequence.

**Why replacement is correct.** A smaller tail at the same length can only help future extensions. Binary search works because `tails` stays sorted.

**Complexity.** O(n log n). The DP `dp[i] = max LIS ending at i` is O(n^2) and is the right fallback if you must reconstruct under teaching constraints or if n ≤ 2000.

**Reconstructing one LIS.** O(n^2) DP plus predecessor indexes is the straightforward way. The O(n log n) version can reconstruct too, but you must store for each element which pile/length it achieved and then walk backward; do not read `tails` left to right as the answer.

**Pitfalls.** Using `upper_bound` for a strictly increasing LIS (that allows equals). Non-decreasing LIS uses `upper_bound`. Subarray vs subsequence: if the problem wants contiguous, this is the wrong tool (use Kadane-style or a simple streak).

```cpp
int lengthOfLIS(vector<int>& a) {
    vector<int> tails;
    for (int x : a) {
        auto it = lower_bound(tails.begin(), tails.end(), x);
        if (it == tails.end()) tails.push_back(x);
        else *it = x;
    }
    return (int)tails.size();
}
```

---

# Page C

## C1. Heap and priority queue

**What it is.** A binary heap is a complete binary tree stored in an array. For a min-heap, every parent is ≤ its children. The root is the minimum. `std::priority_queue<int>` in C++ is a **max-heap** by default.

**Index rules (0-based).** Parent of `i` is `(i - 1) / 2`. Children are `2i + 1` and `2i + 2`.

**Operations.**

- Push: place at the end, sift up. O(log n).
- Pop root: move last element to root, sift down. O(log n).
- Peek: O(1).

**C++ habits.**

```cpp
priority_queue<int> maxh;
priority_queue<int, vector<int>, greater<int>> minh;
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> dij;
```

**Recognize when.** Repeatedly need the current min or max, merge k sorted lists, top k elements, running median (two heaps), Dijkstra, schedule the next event.

**Pitfalls.** Assuming `priority_queue` is a min-heap. No decrease-key in the STL heap: for Dijkstra, push a new pair and ignore stale popped pairs (`if (d != dist[u]) continue`). Building a heap from an existing array is O(n) (`make_heap` / heapify), not O(n log n).

---

## C2. Heap sort

**Steps.**

1. Heapify the array into a max-heap, bottom-up from the last parent `n/2 - 1` down to 0. O(n).
2. For `end` from `n - 1` down to 1: swap `a[0]` (current max) with `a[end]`. Sift down the root on the range `[0, end)`.
3. The array grows a sorted suffix.

**Complexity.** O(n log n) time, O(1) extra space. Unstable.

**Sift down.** At index `i`, while a child exists, let `j` be the larger child. If `a[i] >= a[j]`, stop. Else swap with `j` and continue at `j`.

**Pitfalls.** Sift down on the full array after the swap, which pulls the already sorted suffix back in. Heap sort is a good worst-case sort; introsort (what `std::sort` often is) falls back to it when quicksort recurses too deep. In interviews, calling `std::sort` is fine unless the question is to implement the heap.

---

## C3. Squares of a Sorted Array — LeetCode 977

**What it is.** Sorted array, possibly with negatives. Return `nums[i]^2` sorted ascending.

**Pattern.** Two pointers at the ends. The largest square is at one of the ends, because squares grow as values move away from zero.

**Steps.**

1. `l = 0`, `r = n - 1`, fill an answer array from the **back**.
2. If `abs(a[l]) > abs(a[r])`, put `a[l]^2` at the write position and `l++`. Else put `a[r]^2` and `r--`.

**Complexity.** O(n). Sorting the squares is O(n log n) and correct but misses the point.

**Pitfalls.** Filling the answer from the front (you discover large squares first). `a[i] * a[i]` overflows 32-bit `int` once `|a[i]|` exceeds `46340`. LeetCode 977’s values fit in `int` (`±1e4`). Use `long long` when you reuse the pattern on larger values.

---

## C4. Unsorted array to a max-heap

**What it is.** Rearrange an array so it satisfies the max-heap property. Do not fully sort it.

**Pattern.** Bottom-up heapify. Same first step as heap sort.

**Steps.**

1. Last non-leaf index is `(n / 2) - 1`.
2. For `i` from that index down to 0, sift down at `i`.
3. After this, `a[0]` is the maximum, and every parent ≥ children. The array is not sorted.

**Why O(n).** Most nodes are near the leaves and sift a short distance. The sum of costs is linear.

**Pitfalls.** Calling sift-up on every element from left to right, which is a correct heap but O(n log n). Looping `i` upward from 0, so parents are fixed before children and then violated.

---

## C5. Kth smallest or largest element

**What it is.** Return the k-th largest (LC 215) or k-th smallest. Duplicates: this is order statistic on the multiset, not distinct values, unless the prompt says distinct.

**Three tools.**

| Method | Time | When |
| --- | --- | --- |
| Sort and index | O(n log n) | n is small, or you need the array sorted anyway |
| Min-heap of size k for k-th largest | O(n log k) | k ≪ n, and you stream values |
| Quickselect | average O(n), worst O(n^2) | typical interview follow-up for linear average |

**Heap for k-th largest.** Keep a min-heap of the k largest seen. If the new value is bigger than the heap top, pop and push. The top is the k-th largest. For k-th smallest, keep a max-heap of size k.

**Quickselect.** Partition like quicksort (Lomuto or Hoare). If the pivot lands on index `n - k` (for k-th largest in ascending order), return it. If the pivot index is larger than the target, recurse left; else right. Only one side.

**Pitfalls.** Off-by-one: k-th largest is index `n - k` after ascending sort, not index `k`. A max-heap of size k does not give you the k-th largest at the top; the top is the largest. Quickselect worst case is sorted input with bad pivots; randomize the pivot if that matters.

```cpp
int findKthLargest(vector<int>& a, int k) {
    priority_queue<int, vector<int>, greater<int>> h;
    for (int x : a) {
        h.push(x);
        if ((int)h.size() > k) h.pop();
    }
    return h.top();
}
```

---

## C6. Greedy algorithm problems

**What greedy means.** Build the answer one local choice at a time, and prove that choice can be part of some optimal solution (exchange argument or staying ahead).

**Greedy is appropriate when you can say one of these out loud.**

- Interval scheduling: sort by end time, take the interval that finishes first.
- Jump game farthest reach.
- Huffman coding: merge the two lightest nodes.
- Fractional knapsack: value/weight ratio. (0/1 knapsack is DP, not this.)
- Pair smallest with largest to minimize the max pair sum.
- Gas stations / circular tour: if the total gas ≥ total cost, a start exists; the correct start is just after the prefix that goes most negative.

**Greedy is a trap when.**

- 0/1 choices have overlap (knapsack, house robber without the DP).
- Coin systems are not canonical.
- “Local densest substring” style choices block a better global layout (burst balloons, matrix chain, stick cuts) → interval DP.

**How to practice the proof.** Assume an optimal solution disagrees with your first choice. Swap or replace to make it agree without getting worse. If you cannot write that swap, do not ship the greedy.

---

## C7. Best Time to Buy and Sell Stock II — LeetCode 122

**What it is.** Prices by day. You may complete as many buys and sells as you want, but you hold at most one share. Maximize profit.

**Pattern.** Greedy: take every upward difference. Each increasing adjacent pair is profit you can capture, and skipping a day in a rising run never helps.

**Steps.** Sum `max(0, prices[i] - prices[i-1])` over all i.

**Why.** A rise from a to c through b equals `(b - a) + (c - b)` when both differences are positive. Valleys and peaks fall out automatically.

**Complexity.** O(n).

**Pitfalls.** Applying the one-transaction solution (LC 121, track min price and max profit) which undercounts. Holding state machines are for the limited-transaction versions (LC 123, LC 188) and for the cooldown version (LC 309). If a fee exists (LC 714), the adjacent-difference trick needs adjustment; DP or a modified greedy is safer.

---

## C8. Two City Scheduling — LeetCode 1029

**What it is.** `2n` people. Each has a cost to city A and city B: `costs[i] = [aCost, bCost]`. Send exactly n people to each city. Minimize total cost.

**Pattern.** Greedy by the **difference** `aCost - bCost`. The company of “how much extra A costs versus B”.

**Steps.**

1. Sort people by `aCost - bCost` ascending. The front of the list saves the most by going to A (or loses the least).
2. Send the first n people to A, the rest to B.
3. Sum the corresponding costs.

**Alternative view.** Send everyone to A, then choose n people to “refund” by switching to B. The refund of switching person i is `aCost - bCost`. You want the n largest refunds, which is the same sort.

**Complexity.** O(n log n).

**Pitfalls.** Sorting by `aCost` alone, which ignores how cheap B was. Sending each person to their cheaper city, then fixing the imbalance: possible but messier, and the fix-up is exactly this difference sort.

---

## C9. XOR operations

Properties that solve whole families of problems:

- `x ^ x = 0`, `x ^ 0 = x`, XOR is commutative and associative.
- If every number appears twice except one, the XOR of all of them is the single number.
- `x ^ y` is the bitwise difference. `x ^ y == 0` iff `x == y`.
- Prefix XOR: `prefix[i] ^ prefix[j]` is the XOR of the subarray `(j, i]`. A subarray XOR equals `k` when two prefixes differ by `k`. Count with a map, same shape as subarray sum equals k.
- Swap with XOR (`a ^= b; b ^= a; a ^= b`) works only when `a` and `b` are distinct objects. Do not use it on the same variable or overlapping references.
- `x & (x - 1)` clears the lowest set bit. `x & -x` isolates it (two’s complement).

**When XOR is the wrong tool.** You need the actual missing number and values can lie outside `1..n` with extras; then XOR still works for “one missing” if you also XOR the expected range. Two missing numbers need more than one XOR (split by a set bit). Order, positions, and counts above 2 usually want a map or index marking.

---

## C10. Single Number — LeetCode 136

**What it is.** Every element appears twice except one. Return it. Linear time, constant space.

**Steps.** `ans = 0`; for each `x`, `ans ^= x`; return `ans`.

**Extensions.**

- LC 137: others appear three times. Bit count mod 3, or a two-variable bit state machine.
- LC 260: two unique numbers. XOR everything, isolate one differing bit, split into two groups, XOR each group.

**Pitfalls.** Using a set and toggling presence: correct but O(n) space. Summing and subtracting the set of unique values: overflow and extra space.

---

## C11. Number of 1 Bits — LeetCode 191

**What it is.** Hamming weight of an unsigned integer: how many bits are 1.

**Steps (Brian Kernighan).** While `n`: `n &= n - 1`, count++. Each step removes one set bit. Loops once per set bit.

**Library.** `popcount` / `__builtin_popcount` / `std::popcount` (C++20). Fine unless the question forbids it.

**Pitfalls.** Signed integers and arithmetic right shift (`>>` fills with the sign bit), so the loop `n >>= 1` may never end on a negative `int`. The problem uses unsigned. `n & 1` then shift is the simple 32-iteration version.

---

## C12. Array Partition — LeetCode 561

**What it is.** `2n` integers. Partition into n pairs. Sum the **minimum** of each pair. Maximize that sum.

**Pattern.** Sort and pair adjacent elements. The optimal sum is `a[0] + a[2] + a[4] + ...` after sorting.

**Why.** In the sorted array, pairing `a[2i]` with `a[2i+1]` means the min of the pair is `a[2i]`. Any pairing that “wastes” a larger number as a minimum can be swapped into this adjacent pairing without decreasing the sum of mins.

**Complexity.** O(n log n).

**Pitfalls.** Pairing smallest with largest (that is the minimize-max-pair-sum pattern, a different objective). Summing the larger of each pair.

---

## C13. Partition List — LeetCode 86

**What it is.** Linked list and value `x`. Reorder so every node `< x` comes before every node `≥ x`. Relative order inside each group stays the same (stable).

**Pattern.** Two dummy lists, then join.

**Steps.**

1. `less` chain and `ge` chain, each with a dummy tail pointer.
2. Walk the original list once. Append the node to the matching chain. Detach it (`node->next = nullptr`) as you go, or fix the tail at the end.
3. `lessTail->next = geHead`. `geTail->next = nullptr` so you do not form a cycle back into `less`.
4. Return `lessDummy->next`.

**Complexity.** O(n), O(1) extra.

**Pitfalls.** Leaving `geTail->next` pointing into a node that was later moved to the less-chain, which cycles. Unstable rearrangements (array partition) fail the “preserve relative order” tests. Values equal to `x` go to the second part, not the first.

---

## C14. Spanning tree

**What it is.** A subset of edges that connects all vertices and contains no cycle. A tree on n vertices has exactly `n - 1` edges. A minimum spanning tree is a spanning tree of minimum total edge weight.

**Facts worth memorizing.**

- Exists iff the undirected graph is connected.
- Adding any missing edge to a tree creates exactly one cycle.
- Removing any tree edge disconnects it.
- All MSTs have the same total weight; the edge set is unique only if all edge weights are distinct.
- Algorithms: Kruskal (sort edges + DSU), Prim (grow a fringe with a heap), reverse-delete (drop heaviest edges that are not bridges).

**Counting spanning trees** of a general graph is Kirchhoff’s matrix-tree theorem, not an interview coding problem. Cayley’s formula: `n^(n-2)` spanning trees on the complete graph K_n.

**Pitfalls.** Searching for an MST with Dijkstra. Dijkstra minimizes path lengths from a source; those edges do not form an MST. BFS spanning tree is a valid spanning tree of an unweighted connected graph, but it is not minimum when weights vary.

---

## C15. Partition Equal Subset Sum — LeetCode 416

**What it is.** Can the array be split into two subsets with equal sum?

**Pattern.** 0/1 knapsack. Total sum must be even. Target = `sum / 2`. Ask whether a subset sums to target.

**Steps.**

1. `sum` of all elements. If odd, return false.
2. `target = sum / 2`. Boolean `dp[x]` = “some subset sums to x”. `dp[0] = true`.
3. For each number `num`: for `x` from `target` down to `num`: `dp[x] = dp[x] || dp[x - num]`.
4. Return `dp[target]`.

**Why downward.** Each number may be used once. Updating high to low means `dp[x - num]` still refers to subsets that do not include the current number. Updating low to high reuses `num` many times (that is the unbounded/coin version).

**Complexity.** O(n · target) time, O(target) space. Target is at most `sum/2`. If `sum` is huge (values 1e9, n = 100) this DP does not fit; then meet-in-the-middle or another constraint is in play. LC 416 has n ≤ 200 and sum up to 2e4, so it fits.

**Pitfalls.** Looping `x` upward. Forgetting the odd-sum early exit. Using int sums that overflow when you only needed a boolean.

---

## C16. Word Search — LeetCode 79

**What it is.** Grid of letters. Does `word` appear as a path of adjacent cells (4 directions)? A cell may not be reused in the same path.

**Pattern.** DFS backtracking from every cell that matches `word[0]`.

**Steps.**

1. For each cell, `dfs(r, c, k)` tries to match `word[k]` here.
2. Bounds fail, letter mismatch: false. If `k == word.size() - 1`: true.
3. Mark the cell (change to `#` or a visited flag). Try 4 neighbors at `k + 1`. Unmark.
4. If any start cell returns true, the answer is true.

**Complexity.** O(rows · cols · 4^L) worst case, L = word length. Constraints are small.

**Pruning.** If the board does not contain enough of some letter, return false before searching. Matching from both ends is unnecessary at this size.

**Pitfalls.** Marking visited and not restoring, so later start positions see a damaged board. Reusing a cell. Comparing the whole remaining string instead of one character index (correct but allocates). Word Search II (LC 212) is a different pattern: put all words in a trie and DFS the board once.

---

## C17. Rearrange Array Elements by Sign — LeetCode 2149

**What it is.** Equal number of positive and negative integers. Rearrange so signs alternate, starting with positive, and the relative order inside positives and inside negatives is preserved.

**Pattern.** Stable split into two lists, then interleave. Or one pass with two write pointers if you build a new array: `pos` writes at `0, 2, 4, ...`, `neg` writes at `1, 3, 5, ...`.

**Steps.**

1. `ans` of size n. `i = 0` for the next positive slot, `j = 1` for the next negative slot.
2. For each value: if positive, `ans[i] = x`, `i += 2`. Else `ans[j] = x`, `j += 2`.

**Complexity.** O(n) time, O(n) space. In-place stable rearrangement is much harder and not required.

**Pitfalls.** Sorting by sign, which destroys relative order (`3, 1, -2, -5` should become `3, -2, 1, -5`, not `1, -5, 3, -2`). The follow-up where counts are unequal (LC 2149 discussion variant) needs the two-list approach: place the extras at the end after alternation runs out.

---

## C18. Kruskal’s algorithm

Same algorithm as B22, written as a checklist you can recode from memory.

1. Each vertex starts as its own component.
2. Sort edges by weight ascending.
3. For each edge `u—v` of weight `w`: if `find(u) != find(v)`, add `w` to the MST cost and `unite(u, v)`.
4. After `n - 1` successful unions, stop. Total cost is the MST weight.

**Second-best MST** (a common follow-up). Build the MST, then for each non-tree edge, the best swap is that edge replacing the heaviest tree edge on the path between its ends. Compute path-max with binary lifting, or a simpler O(n^2) DP on the tree if n is small.

**Maximum spanning tree.** Sort descending, or negate weights and negate the result.

---

## C19. Prim’s algorithm

**What it is.** Grow one component until it covers the graph. Always add the cheapest edge that leaves the component.

**Steps (lazy heap version).**

1. `inMST[v] = false`, `dist[v] = infinity`, `dist[start] = 0`.
2. Min-heap of `{cost, node}`. Push `{0, start}`.
3. While the heap is not empty: pop `{w, u}`. If `u` is already in the MST, skip. Mark `u`. Add `w` to the answer.
4. For each edge `u—v` of weight `e`: if `v` is not in the MST, push `{e, v}`.
5. If you marked fewer than n nodes, the graph is disconnected.

**Complexity.** O(m log m) with a binary heap and lazy duplicates. O(n^2) with a dense scan for the next closest vertex, which is better when m is about n^2.

**Difference from Dijkstra.** Dijkstra pushes `dist[u] + e` (path cost). Prim pushes `e` (edge cost to join the tree). The code looks similar and that similarity causes bugs. Write the meaning in a comment above the push.

**Pitfalls.** Marking a node in the MST when it is first pushed. A later cheaper edge can still be the right connection. With non-negative weights, the first time you **pop** a node from a min-heap you have its cheapest link. Skip pops of nodes already in the tree. That lazy pattern is the safe one with `std::priority_queue`, which has no decrease-key.

---

## C20. Tries

**What it is.** A tree of prefixes. Each edge is a character. Nodes store “is this the end of a word?” and children, usually an array of 26 pointers or a hash map.

**Operations.** Insert, search exact word, search prefix. All O(length of the word).

**Recognize when.** Many prefix queries, autocomplete, word search II, replace words with roots (LC 648), maximum XOR of two numbers (binary trie of bits), counting pairs with a given XOR prefix.

```cpp
struct Trie {
    struct Node { Node* ch[26]{}; bool end = false; };
    Node* root = new Node();
    void insert(const string& s) {
        Node* n = root;
        for (char c : s) {
            int i = c - 'a';
            if (!n->ch[i]) n->ch[i] = new Node();
            n = n->ch[i];
        }
        n->end = true;
    }
    bool search(const string& s, bool prefixOnly = false) {
        Node* n = root;
        for (char c : s) {
            int i = c - 'a';
            if (!n->ch[i]) return false;
            n = n->ch[i];
        }
        return prefixOnly || n->end;
    }
};
```

**Pitfalls.** `search("app")` returning true because `"apple"` was inserted. That is a prefix query; exact search must check `end`. Forgetting `end` on a word that is also a prefix of another (`app` and `apple` both need their own end flags). Binary XOR trie uses bit 31 down to 0, and at each bit you prefer the child that matches the opposite bit if you want maximum XOR.

**Memory.** 26 pointers per node is the fast interview version. A map is fine for sparse or Unicode alphabets.

---

## C21. Dijkstra’s algorithm

**What it is.** Single-source shortest paths on a graph with **non-negative** edge weights.

**Steps.**

1. `dist[src] = 0`, all others infinity.
2. Min-heap of `{dist, node}`. Push the source.
3. Pop `{d, u}`. If `d != dist[u]`, this is a stale entry; skip. (This replaces decrease-key.)
4. For each edge `u → v` with weight `w`: if `dist[u] + w < dist[v]`, update `dist[v]` and push `{dist[v], v}`.
5. The first time you pop a node with `d == dist[u]`, that distance is final.

**Complexity.** O(m log n) with a binary heap.

**Pitfalls.** Negative edges: a later negative edge can improve a node you already settled. Use Bellman-Ford. Forgetting the stale-entry check and also marking visited too early. Using an unordered `priority_queue` (max-heap) so you expand the farthest node first. On an undirected graph, add both directions.

**Path.** Store `parent[v] = u` when you relax v. If you need the count of shortest paths, also relax equality: when `dist[u] + w == dist[v]`, add the ways.

---

## C22. Bellman-Ford

**What it is.** Single-source shortest paths that **allow negative weights**, and can detect a negative cycle reachable from the source.

**Steps.**

1. `dist[src] = 0`, others infinity.
2. Repeat `n - 1` times: for every edge `u → v` with weight `w`, if `dist[u]` is finite and `dist[u] + w < dist[v]`, update `dist[v]`.
3. One more pass over all edges. If any distance still improves, a negative cycle is reachable.

**Why n − 1 rounds.** A simple shortest path has at most n − 1 edges. Each round extends the number of edges you are willing to use by one, in the worst order of relaxation.

**Complexity.** O(n · m). Use it when m is small or negative edges exist. If n = 1e5, this is too slow; the graph had better be a DAG or non-negative.

**SPFA** is Bellman-Ford with a queue (only re-queue nodes whose distance improved). Average case is faster, worst case is still O(n · m). It is more code for the same guarantees.

**Pitfalls.** Stopping at n − 2 rounds. Treating “infinity + w” as a real relaxation because you used a sentinel that overflows (`1e9 + 1e9`). Use a check `dist[u] < INF`. Detecting a negative cycle that is **not** reachable from the source: only matters if the problem asks “anywhere in the graph”. Then you can add a dummy source connected to all nodes with weight 0, or start the extra logic from every component.

---

## C23. Floyd-Warshall

**What it is.** All-pairs shortest paths. Works with negative edges. Detects negative cycles if any `dist[i][i]` becomes negative.

**Steps.**

1. `dist[i][i] = 0`. `dist[i][j] = w` for each edge, infinity if no edge. If multiple edges, keep the minimum.
2. For `k` from 0 to n-1 (the intermediate node): for every `i`, for every `j`: `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`.
3. `k` must be the **outer** loop. It means “paths that are allowed to pass through nodes `0..k`”.

**Complexity.** O(n^3) time, O(n^2) space. Practical for n ≤ 400, sometimes 500.

**Path reconstruction.** `next[i][j] = j` initially for existing edges. When you improve via `k`, set `next[i][j] = next[i][k]`. Follow `next` from i to j.

**Pitfalls.** Putting `k` in the inner loop, which uses updated intermediates out of the induction order and is wrong. Overflow on `INF + INF`. Using Floyd on n = 1e5. For an unweighted graph, n BFS runs are O(n(n+m)) and can be better when the graph is sparse.

**Transitive closure** is the boolean version: `reach[i][j] |= reach[i][k] && reach[k][j]`. Same loop order.

---

## C24. Find the Smallest Divisor Given a Threshold — LeetCode 1283

**What it is.** Find the smallest positive integer divisor `d` such that the sum over `i` of `ceil(nums[i] / d)` is ≤ `threshold`.

**Pattern.** Binary search on the answer. As `d` grows, the sum of ceilings shrinks (monotonic).

**Steps.**

1. `lo = 1`, `hi = max(nums)` (a divisor larger than every element gives sum = n, and the problem guarantees a solution, with `threshold ≥ n`).
2. While `lo < hi`: `mid = lo + (hi - lo) / 2`.
3. If `sum of ceil(a[i] / mid) <= threshold`, try smaller: `hi = mid`. Else `lo = mid + 1`.
4. Return `lo`.

**Ceil without floats.** `ceil(a / d) = (a + d - 1) / d` for positive integers.

**Complexity.** O(n log M) where M is the max element.

**Pitfalls.** Binary searching the array indexes. Using floating `ceil` and getting precision errors. `while (lo <= hi)` mixed with `hi = mid` can loop forever; stay on the first-true template from A6.

This same template solves capacity to ship packages, koko eating bananas, split array largest sum, and minimize maximum pair distance: define `ok(mid)`, search the boundary.

---

## C25. Find K Closest Elements — LeetCode 658

**What it is.** Sorted array, return the k elements closest to `x`, in ascending order. Distance is absolute value. Ties break toward the smaller element.

**Pattern.** Binary search the start of the window of length k. You want the leftmost `i` such that the window `a[i .. i+k-1]` is better than the window shifted right.

**Steps.**

1. Search `lo = 0`, `hi = n - k` for the starting index.
2. `mid` is a candidate start. Compare `x - a[mid]` with `a[mid + k] - x`.
3. If `x - a[mid] > a[mid + k] - x`, the element just left of the window is farther than the element just right of it, so the start must move right: `lo = mid + 1`.
4. Else the better start is mid or to the left: `hi = mid`.
5. The window `a[lo .. lo+k)` is the answer, already sorted.

**Why the comparison.** Window starting at `mid` excludes `a[mid+k]` and includes `a[mid]`. If `a[mid]` is strictly farther from `x` than `a[mid+k]`, that start is worse than the start at `mid+1`. On a tie (`equal` distances), the smaller element should win, and `>` (not `>=`) keeps the left element.

**Alternative.** Binary search the insertion point of `x`, then expand two pointers outward k times. O(log n + k). Easier to invent under pressure, slightly more care on ties.

**Pitfalls.** Sorting by distance, which scrambles ascending order and is O(n log n). Expanding until you have k elements but breaking ties toward the larger value.

---

## C26. Longest Substring Without Repeating Characters — LeetCode 3

**What it is.** Longest contiguous substring whose characters are all unique.

**Pattern.** Variable sliding window. The window `[l, r]` always has unique characters. `r` only moves forward. `l` jumps forward to kill a duplicate.

**Steps.**

1. Map `last[char] = -1` or the last index where it appeared.
2. `l = 0`, `best = 0`.
3. For `r` from 0 to n-1: if `s[r]` was seen at index `p` and `p >= l`, move `l` to `p + 1`. Update `last[s[r]] = r`. `best = max(best, r - l + 1)`.

**Complexity.** O(n) time, O(alphabet) space.

**Pitfalls.** Moving `l` backward (never). Using a set and deleting one by one from the left is also O(n) and fine; the index map is the cleaner version. Subsequence instead of substring: then the answer is the number of distinct characters, a different problem. Resetting the whole map when you see a duplicate instead of shrinking only past that character, which drops valid windows.

---

## C27. Sieve of Eratosthenes

**What it is.** All prime numbers up to `n`, or a boolean array `isPrime[0..n]`.

**Steps.**

1. `isPrime[0] = isPrime[1] = false`. Everything from 2 to n starts true.
2. For `p` from 2 to `sqrt(n)`: if `isPrime[p]`, mark multiples `p*p, p*p+p, p*p+2p, ...` as not prime. Multiples below `p*p` were already marked by smaller primes.
3. The true indexes are the primes.

**Complexity.** O(n log log n) time, O(n) space.

**Linear sieve.** If you need the smallest prime factor of every number (useful for fast factorization of many queries), the linear sieve marks each composite exactly once and stores `spf[i]`. Then factoring `x` is O(log x) by dividing out `spf`.

**Pitfalls.** Looping `p` all the way to `n` (correct but slower; the inner loop already covers it). Using `int` for `p * p` when `n` is near 1e7–1e8 and `p` exceeds 46340; compute the bound in `long long` or loop `p <= n / p`. Marking from `2*p` is fine, just more writes.

**Prime test for one number.** If you have one query and n ≤ 1e12, trial division up to sqrt(n) is enough. Sieve is for many numbers or all numbers in a range.

---

## C28. Minimum Add to Make Parentheses Valid — LeetCode 921

**What it is.** String of `'('` and `')'` only. Return the minimum number of insertions to make it valid.

**Pattern.** Balance counter. `open` unmatched `'('`. `insert` is the number of `'('` you had to imagine in front of extra `')'`.

**Steps.**

1. For each character: if `'('`, `open++`. If `')'`: if `open > 0`, `open--` (match it), else `insert++` (this close had no partner).
2. Answer is `insert + open` (each leftover open needs a close).

**Complexity.** O(n), O(1).

**A stack of indexes** is the right upgrade when you must **remove** invalid parentheses or return the string (LC 1249, 301), because you need positions. For a pure count, the counter is enough.

**Pitfalls.** Returning only `open` and ignoring already-seen unmatched closes. Treating `[]` and `{}` the same without separate counters; if multiple bracket types exist, one integer balance is insufficient (use a stack). “Minimum remove” vs “minimum add” are different outputs.

---

## C29. Maximum Product Subarray — LeetCode 152

**What it is.** Contiguous subarray with the largest product.

**Pattern.** Kadane, but track both the max and the min product ending here, because a negative times a negative becomes large.

**Steps.**

1. `maxEnd = minEnd = ans = nums[0]`.
2. For each next `x`:
   - Candidates are `x`, `maxEnd * x`, `minEnd * x`.
   - New max is the largest of the three. New min is the smallest.
   - Because you need both old values, compute them into temporaries before assigning.
3. `ans = max(ans, newMax)`.

**Complexity.** O(n) time, O(1) space.

**Pitfalls.** Plain Kadane for sums ignores the sign flip. Always include `x` itself as a candidate so a zero or a lone negative can start a new subarray. Keep running products in `long long`: LeetCode guarantees the **answer** fits in 32-bit `int`, but a running product of many twos does not. The final `ans` can be cast back to `int`.

---

## C30. Minimum Initial Energy to Finish Tasks — LeetCode 1665

**What it is.** Each task has `actual` energy it consumes and `minimum` energy you must have **before** starting it. Completing a task subtracts `actual`. Find the minimum starting energy so that some order finishes every task. Energy never goes negative at a start check; you need `energy >= minimum` before the task, then `energy -= actual`.

**Pattern.** Greedy sort. Do first the task that forces the largest **leftover requirement** `minimum - actual` (how much energy must remain after you pay the cost).

**Steps.**

1. Sort tasks by `minimum - actual` descending. Ties do not matter.
2. Walk in that order, tracking current energy spent planning from the start:
   - Let `start` be the energy at the beginning (unknown). It is easier to simulate the minimum energy required so far.
   - `have` starts at 0 and represents extra energy you currently hold above 0 in a backward way. Standard simulation:
3. Concrete simulation used by most solutions:
   - `ans = 0` (energy required at the start), `curr = 0` (energy you have as you go, if you started with `ans`… the usual code is):

```cpp
int minimumEffort(vector<vector<int>>& tasks) {
    sort(tasks.begin(), tasks.end(), [](auto& A, auto& B) {
        return (A[1] - A[0]) > (B[1] - B[0]);
    });
    int ans = 0, curr = 0; // curr is energy at this moment if we started with `ans`
    for (auto& t : tasks) {
        int actual = t[0], mn = t[1];
        if (curr < mn) {
            ans += mn - curr;
            curr = mn;
        }
        curr -= actual;
    }
    return ans;
}
```

**Meaning.** Whenever the energy you are holding is below `minimum`, you must have started with more. Add that gap to the initial energy and to the current energy, then pay `actual`.

**Why this order.** Saving tasks with a high `minimum - actual` for later is worse: those tasks demand a large bankroll relative to what they consume, so they should be done while your energy is still high. Doing a task that consumes almost all of its minimum first wastes the surplus smaller tasks could have used.

**Pitfalls.** Sorting by `minimum` descending alone, which fails when a high minimum also consumes a lot (`minimum - actual` is small). Sorting by `actual` alone.

---

## C31. Sum of Beauty of All Substrings — LeetCode 1781

**What it is.** Beauty of a string is (highest frequency of any character) − (lowest frequency among characters that appear at least once). Sum the beauty of every substring.

**Pattern.** Because the alphabet is 26, you can afford O(26) work per substring, or smarter: fix the start (or end) and extend while maintaining frequencies.

**Steps.**

1. `ans = 0`.
2. For each start `i`: frequency array of 26 zeros. For each end `j` from `i` to `n - 1`: increment `freq[s[j]]`. Scan the 26 counts, track max and the min among non-zero counts. Add `max - min` to `ans`.

**Complexity.** O(n^2 · 26) time, O(26) extra space. n ≤ 500 on this problem, so n^2 · 26 is about 6e6 operations.

**Pitfalls.** Including zeros in the minimum, which makes beauty equal the max frequency and is wrong. Trying a full O(n) formula too early; the constraints were chosen so the quadratic scan is the intended solution. Beauty of a single character is `1 - 1 = 0`.

---

# Page D

## D1. Maximum Depth of Binary Tree — LeetCode 104

**What it is.** Number of nodes on the longest root-to-leaf path. An empty tree has depth 0. A single node has depth 1.

**Pattern.** DFS returns `1 + max(left, right)`. BFS level count is equivalent.

```cpp
int maxDepth(TreeNode* root) {
    if (!root) return 0;
    return 1 + max(maxDepth(root->left), maxDepth(root->right));
}
```

**Pitfalls.** Returning the max of the children without the `+1`. Counting edges (then the answer is one smaller; the LeetCode problem counts nodes). Confusing this with diameter, which does not have to pass through the root.

---

## D2. Symmetric Tree — LeetCode 101

**What it is.** The tree is a mirror of itself around the center.

**Pattern.** Compare two nodes at a time: the left child of one against the right child of the other.

**Steps.** `same(a, b)` is true when both are null, false when one is null or values differ, otherwise `same(a->left, b->right) && same(a->right, b->left)`. Start with `same(root->left, root->right)`. A null root is symmetric.

**Pitfalls.** Comparing `a->left` with `b->left` (that checks identical structure, i.e. Same Tree, not a mirror).

---

## D3. Path Sum — LeetCode 112

**What it is.** Does any root-to-**leaf** path have values summing to `targetSum`?

**Steps.** Subtract `node->val` as you go. At a leaf, return whether the remaining target is 0. A null node returns false (it is not a successful empty path, except if the whole tree is null: return false).

**Pitfalls.** Returning true on an internal node whose prefix already equals the target. The path must end at a leaf. LC 113 asks for all such paths (push/pop the node on the way down). LC 437 counts paths that may start and end anywhere and needs prefix sums on the DFS path.

---

## D4. Invert Binary Tree — LeetCode 226

**What it is.** Swap every node’s left and right child.

**Steps.** If null, return null. Swap children, then invert each child (or recurse first, then swap; both work). Return the same root.

```cpp
TreeNode* invertTree(TreeNode* root) {
    if (!root) return nullptr;
    swap(root->left, root->right);
    invertTree(root->left);
    invertTree(root->right);
    return root;
}
```

**Pitfalls.** Swapping values instead of child pointers when the structure is what must mirror. Losing a child by overwriting `left` before saving `right`.

---

## D5. Same Tree — LeetCode 100

**What it is.** Two trees are identical in structure and values.

**Steps.** Both null → true. One null or values differ → false. Else left subtrees match and right subtrees match.

**Pitfalls.** Comparing only preorder sequences without null markers: different shapes can share a preorder listing. If you serialize, include nulls.

---

## D6. Balanced Binary Tree — LeetCode 110

**What it is.** A tree is height-balanced if, for **every** node, the heights of the left and right subtrees differ by at most 1.

**Pattern.** Post-order. Return the height, or a sentinel `-1` if this subtree is already unbalanced. One O(n) pass.

**Steps.**

1. Null returns height 0.
2. `L = dfs(left)`, `R = dfs(right)`.
3. If either is `-1`, or `abs(L - R) > 1`, return `-1`.
4. Else return `1 + max(L, R)`.
5. The tree is balanced if the root call is not `-1`.

**Pitfalls.** Only checking the root’s two heights. Computing height separately at every node, which is O(n^2) on a skewed tree. Using depth from the root instead of subtree height.

---

## D7. Count Good Nodes in Binary Tree — LeetCode 1448

**What it is.** A node is good if its value is ≥ every value on the path from the root to that node.

**Pattern.** DFS passing `maxSoFar`.

**Steps.** `dfs(node, mx)`. If null, return 0. `good = node->val >= mx ? 1 : 0`. Update `mx = max(mx, node->val)`. Return `good + dfs(left, mx) + dfs(right, mx)`. Start with `mx` equal to the root value, or with `INT_MIN` so the root counts.

**Pitfalls.** Comparing only with the parent, not with the max on the whole path. A smaller parent with a large grandparent still blocks the node.

---

## D8. Diameter of Binary Tree — LeetCode 543

**What it is.** The longest path between any two nodes, measured in **edges**. The path does not need to pass through the root.

**Pattern.** Post-order. The function returns how many nodes are on the longest downward chain starting at this node. The best path through this node, measured in edges, is `leftChain + rightChain`. Update a global best at every node, because the diameter may sit entirely inside one subtree.

**Definition used below.** Null returns 0. A leaf returns 1. `L + R` is the edge count of the path that goes down the left chain, through the node, and down the right chain.

```cpp
int best = 0; // edges
int dfs(TreeNode* n) {          // returns nodes in longest downward chain
    if (!n) return 0;
    int L = dfs(n->left), R = dfs(n->right);
    best = max(best, L + R);    // edges through n
    return 1 + max(L, R);
}
```

A downward chain of `L` nodes on the left and `R` nodes on the right joins through the current node in `L + R` edges. A single node (a leaf) has `L = R = 0`, so it contributes diameter 0, and the function returns 1.

**Pitfalls.** Returning `1 + L + R` as the diameter (that counts nodes, but this problem wants edges). Computing only `height(left) + height(right)` at the root and ignoring a longer path in a subtree. The global `best` must be updated inside every call.

---

## D9. Binary Tree Zigzag Level Order Traversal — LeetCode 103

**What it is.** Level order, but alternate direction each level. Level 0 left to right, level 1 right to left, and so on.

**Pattern.** Normal BFS. Reverse every odd level, or push into the level array from the back using a direction flag.

**Steps.**

1. Queue the root. `leftToRight = true`.
2. For each layer, snapshot `sz = queue.size()`. Fill a vector of size `sz`.
3. Pop `i` from 0 to `sz - 1`. Write the value at index `i` or at index `sz - 1 - i` depending on direction.
4. Always enqueue left child then right child (the index trick handles visual order). Toggle direction.

**Pitfalls.** Enqueueing right before left on odd levels **and** reversing, which double-flips. Using a deque to push front/back is fine too; just do not also reverse.

---

## D10. KMP (Knuth-Morris-Pratt)

**What it is.** Find occurrences of pattern `p` inside text `t` in O(|t| + |p|), by never re-comparing characters you already know match.

**The prefix function (also called LPS, pi, failure function).** `pi[i]` is the longest proper prefix of `p[0..i]` that is also a suffix of `p[0..i]`. Proper means not the whole string.

**Building `pi`.**

1. `pi[0] = 0`. `len = 0` (length of the current matching prefix).
2. For `i` from 1 to `m - 1`:
   - While `len > 0` and `p[i] != p[len]`, fall back: `len = pi[len - 1]`.
   - If `p[i] == p[len]`, `len++`.
   - `pi[i] = len`.

**Searching.**

1. `j = 0` (matched length into `p`).
2. For `i` over the text: while `j > 0` and `t[i] != p[j]`, `j = pi[j - 1]`. If `t[i] == p[j]`, `j++`. If `j == m`, record a match ending at `i`, then `j = pi[j - 1]` to allow overlaps.

**Why the fallback is correct.** If you have matched `j` characters and the next fails, the next smaller border of the matched portion is `pi[j - 1]`. You skip the comparisons that the border guarantees.

**Recognize when.** Pattern matching with overlaps (`aaa` inside `aaaa`), or “longest prefix that is a suffix”, or string compression problems that are secretly the prefix function (LC 1392, longest happy prefix). For a single search in an interview, `std::string::find` is acceptable unless they asked you to implement KMP. For many patterns, switch to Aho-Corasick (trie + failure links), which is KMP generalized.

**Pitfalls.** Setting `pi[0] = -1` and `pi[0] = 0` in the same code. Off-by-one when falling back (`pi[j]` vs `pi[j - 1]`). Forgetting that after a full match you must fallback, or overlapping matches are lost.

```cpp
vector<int> prefixFunction(const string& p) {
    int m = (int)p.size();
    vector<int> pi(m);
    for (int i = 1, len = 0; i < m; i++) {
        while (len > 0 && p[i] != p[len]) len = pi[len - 1];
        if (p[i] == p[len]) len++;
        pi[i] = len;
    }
    return pi;
}
```

---

## D11. Count substrings with most 3s and 2s, and the repeating-character window

Two different window problems travel together in notes. Keep them separate.

### Longest substring without repeating characters

This is C26 / LC 3. Variable window, last-seen index, `l` jumps to `last[c] + 1`. Answer is the max window length.

### Substrings whose most frequent character is constrained (the “3 twos and 2 threes” style)

The wording on the sheet is ambiguous. The usual problem it refers to is one of these, and the technique is the same:

- Count substrings that contain at most `k` of a given character.
- Count substrings where the count of `'2'` is at most 3 and the count of `'3'` is at most 2 (or similar caps).
- LC 395-style “at least K repeating” is harder (divide and conquer or sliding window on the number of distinct letters). Do not force that into a single simple window.

**At most K of certain characters.**

1. Window `[l, r]`, maintain counts.
2. Advance `r`, add `s[r]`.
3. While the window violates the cap, remove `s[l]` and `l++`.
4. Every window `[l, r]` that is valid means **all** substrings ending at `r` and starting in `[l, r]` are valid if the property is “at most” (shrinking from the left only makes counts smaller). Add `(r - l + 1)` to the answer.

**Exactly K.** `count(at most K) - count(at most K - 1)`. This subtraction trick is the standard way to turn “at most” windows into “exactly”.

**Recognize when.** The constraint is about counts inside a contiguous slice, and adding a character on the right only moves the feasibility in one direction. If feasibility is not monotonic, sliding window does not apply; use prefix counts plus a formula instead.

**Pitfalls.** Adding `1` per valid window instead of `r - l + 1` when the question asks for the **number of substrings**, not the longest length. Using the exactly-K window without the subtraction and trying to move `l` in both directions.

---

## D12. BST: search, validity, LCA

### Search (LC 700)

If the node is null or equal to the target, return it. If `target < node->val`, search left, else search right. O(h).

### Validity (LC 98)

Every node must be strictly greater than all nodes in its left subtree and strictly less than all nodes in its right subtree. Checking only the parent is wrong: a node can be greater than its parent but still smaller than its grandparent on the wrong side.

**Method.** Pass an open range `(low, high)`. Root starts with `(LLONG_MIN, LLONG_MAX)` or nullable bounds. Left recursion tightens `high` to `node->val`. Right recursion tightens `low` to `node->val`.

**Inorder method.** Inorder of a BST is strictly sorted. Walk inorder and check each value is greater than the previous. Store `prev` as a pointer or a long long initialized below every legal value.

### Lowest common ancestor in a BST (LC 235)

If both targets are `< node`, LCA is in the left subtree. If both are `>`, LCA is in the right. Otherwise this node is the split point, so it is the LCA. This uses the BST property and is O(h).

### LCA in a general binary tree (LC 236)

You do **not** have ordering. DFS returns a pointer.

- If the node is null or equals either target, return the node.
- `L = dfs(left)`, `R = dfs(right)`.
- If both `L` and `R` are non-null, this node is the LCA.
- Else return whichever child result is non-null.

This assumes both targets exist. If one might be missing, you need a second check that both were seen.

**Pitfalls.** Using the BST split rule on a general binary tree. Using the general LCA recursion on a BST is correct but slower to write and O(n) instead of O(h). Allowing duplicates if the problem says strict BST; decide whether equals go left.

---

## D13. Graph theory (the pieces that show up in coding problems)

Keep a small vocabulary so problem statements map to algorithms.

- **Undirected vs directed.** Undirected edges are stored twice if you use adjacency lists.
- **Degree / indegree / outdegree.** Kahn’s algorithm consumes indegree.
- **Path vs walk vs cycle.** A simple path does not repeat vertices. Shortest paths in unweighted graphs are simple.
- **Connected vs strongly connected.** Undirected: a path between every pair. Directed: a directed path each way. Algorithms differ (Kosaraju / Tarjan for SCCs).
- **Tree.** Connected undirected graph with `n - 1` edges, or any connected graph with no cycle.
- **Bipartite.** Two colors, no odd cycle. See B24.
- **DAG.** Directed, no cycle. Topological order exists. DP on the nodes in topo order is the standard “longest path in a DAG”.
- **Component.** DFS/BFS flood fill, or DSU for undirected merge-only connectivity.
- **Bridge / articulation point.** Edges or vertices whose removal increases the number of components. Tarjan with discovery times and low links. Needed for “critical connections” (LC 1192).
- **Euler path.** Visits every **edge** once. Directed: the graph is weakly connected (ignoring isolated vertices) and every vertex has equal in and out degree, except possibly one start (`out = in + 1`) and one end (`in = out + 1`). Hierholzer’s algorithm: stack, greedy walk, splice tours.
- **Hamilton path.** Visits every **vertex** once. NP-hard. Only attempt it for n ≤ 20 with DP over subsets.

**Representation choice.**

| Graph | Representation |
| --- | --- |
| n ≤ 500, dense, Floyd or checking every pair | adjacency matrix |
| sparse, n = 1e5 | adjacency list `vector<vector<pair<int,int>>>` |
| dynamic connectivity, undirected, no deletions | DSU |
| grid | implicit graph, neighbors are the four cells |

**Pitfalls.** Recursion depth on a line graph of 1e5 nodes. `vector<vector<int>>` when you needed weights. Counting m for undirected graphs after inserting both directions (do not divide by two if the input already lists each edge once and you only iterate the input for Kruskal).

---

## D14. Graph traversal (DFS / BFS)

Operational checklist. The concepts are in A25.

**Building the graph from an edge list.**

```cpp
vector<vector<int>> g(n);
for (auto& e : edges) {
    int u = e[0], v = e[1];
    g[u].push_back(v);
    g[v].push_back(u); // omit this line if the edge is directed
}
```

**Connected components (undirected).**

```cpp
vector<int> seen(n);
int components = 0;
function<void(int)> dfs = [&](int u) {
    seen[u] = 1;
    for (int v : g[u]) if (!seen[v]) dfs(v);
};
for (int i = 0; i < n; i++) if (!seen[i]) { components++; dfs(i); }
```

**Grid flood fill.** Same DFS/BFS. Skip cells outside the board, cells that are not land, and cells already visited. Mark visited immediately. Count the number of times you start a new fill: that is the number of islands (LC 200).

**0-1 BFS.** When every edge weighs 0 or 1, use a deque. Relaxing a 0-weight edge pushes the neighbor to the **front**. A 1-weight edge pushes to the **back**. Distances are still non-decreasing from front to back. O(n + m).

**Multi-source.** Put all sources in the queue at distance 0 before the loop. That is how you compute distance to the nearest source.

**Directed cycle check.** `color`: 0 unseen, 1 on the current path, 2 finished. Seeing color 1 again means a cycle.

**Pitfalls.** `seen[v] = true` after the recursive call returns, which allows a node to be entered through two paths and explodes. For grids, forgetting the visited mark and also walking back to the parent. BFS distance stored in a separate array initialized to -1 doubles as the visited mark.

---

## D15. Clone Graph — LeetCode 133

**What it is.** Deep-copy an undirected connected graph. Each node has a value and a list of neighbors. The copy must have the same structure, with new node objects.

**Pattern.** DFS or BFS plus a hash map from original node pointer to the clone. The map is both the visited set and the way you reuse a clone when a neighbor points back.

**Steps.**

1. If the start node is null, return null.
2. `clone(node)`: if `node` is already in the map, return the clone. Else create `Node(node->val)`, store it in the map **before** walking neighbors (cycles), then for each neighbor push `clone(neighbor)` into the copy’s neighbor list.
3. Return the clone of the start.

**Complexity.** O(n + m).

**Pitfalls.** Creating a new node every time you see an edge, so shared neighbors become duplicates and cycles never terminate. Using the node value as the map key when values are not unique (the problem says values are unique on LC 133, but pointer keys are still the safer habit).

---

## D16. All Paths From Source to Target — LeetCode 797

**What it is.** Directed acyclic graph of `n` nodes, edges only go from a smaller index to a larger one (so it is a DAG). Return every path from `0` to `n - 1`.

**Pattern.** DFS backtracking. Push the node, recurse to neighbors, pop the node. Because it is a DAG you do not need a visited set to prevent cycles. If the graph might have cycles, a visited set is mandatory or you loop forever, and “all paths” can be exponential anyway.

**Steps.**

1. `path = {0}`. `dfs(u)`: if `u == n - 1`, record a copy of `path` and return.
2. For each neighbor: push, dfs, pop.

**Complexity.** Output-sensitive. The number of paths can be exponential (a layered DAG). Time is proportional to the total length of all paths.

**Pitfalls.** Recording `path` by reference without copying, so later pops destroy stored answers. Marking nodes visited globally in a DAG and therefore missing paths that share a vertex (sharing a vertex is legal; only a cycle is a problem).

---

## D17. Reverse a permutation / inverse permutation

Two operations share this name. Identify which one the prompt wants.

### Reverse the array

The sequence is a permutation of `1..n` or of `0..n-1`, but the task is only to reverse order. Two pointers, swap toward the center. O(n) time, O(1) space.

If it must stay a permutation of the same set, reversal does that automatically.

### Inverse permutation

Given `p` where `p[i]` is the image of `i` (0-based), build `inv` so `inv[p[i]] = i`. Then `inv[p[i]]` restores `i`. One pass, O(n).

```cpp
vector<int> inverse(const vector<int>& p) {
    vector<int> inv(p.size());
    for (int i = 0; i < (int)p.size(); i++) inv[p[i]] = i;
    return inv;
}
```

**Apply a permutation k times.** If k is small, repeat. If k is large, decompose into cycles. On a cycle of length L, each element moves `k % L` steps along the cycle. Follow the cycle into a temporary buffer, write the rotated values, mark the cycle visited so you do not redo it.

**Next permutation** (LC 31) is a different algorithm: find the longest non-increasing suffix, swap the pivot with the next larger element in the suffix, reverse the suffix. Know the name so you do not confuse it with “inverse”.

**Pitfalls.** Treating `p` as 1-based while indexing `inv` as 0-based (`inv[p[i] - 1] = i + 1`). Following cycles without a visited array and rotating the same cycle once per element. In-place cycle following loses values if you do not save the overwritten element; use a buffer unless you are careful with the “start value” technique.

---

## D18. Find the Duplicate Number — LeetCode 287

**What it is.** `n + 1` integers, each in `1..n`. Exactly one value is repeated (it may repeat more than once). Find it. Requirements that matter: do not modify the array, O(1) extra space. (If modification is allowed, index marking from B10 is easier.)

**Pattern.** Floyd cycle detection. Treat the array as a function `next(i) = nums[i]`. Because the range is `1..n` and there are `n + 1` entries, following indexes from 0 stays inside the array, and the duplicate value is the entrance of the cycle.

**Why there is a cycle.** `nums[i]` is a pointer from index `i` to index `nums[i]`. Indexes are `0..n` and values are `1..n`, so every node has outdegree 1, and node 0 points somewhere in `1..n`. A duplicate value means two indexes point at the same next index, which creates a cycle. The cycle entrance is that duplicated value.

**Steps.**

1. `slow = nums[0]`, `fast = nums[0]`.
2. Do `slow = nums[slow]`, `fast = nums[nums[fast]]` until `slow == fast`. They meet inside the cycle.
3. `slow = nums[0]`. Move both one step at a time: `slow = nums[slow]`, `fast = nums[fast]`, until they meet. That meeting point is the duplicate.
4. Return it.

**Complexity.** O(n) time, O(1) space.

**Alternative if you may use O(n) memory or the array is not index-linked.** Binary search the value: count how many numbers are `≤ mid`. If the count is `> mid`, the duplicate is in `1..mid`. This also finds the duplicate, uses O(1) space, and does not rely on the cycle story. It needs the “exactly the values lie in 1..n with one duplicate” guarantee.

**Pitfalls.** Returning the meeting point of the first phase (that point is some node inside the cycle, not necessarily the entrance). Starting fast and slow at 0 vs `nums[0]` inconsistently. Using index marking after the interviewer said the array is read-only. XOR does not work: the duplicate can appear more than twice, and the range argument is fragile here.

```cpp
int findDuplicate(vector<int>& nums) {
    int slow = nums[0], fast = nums[0];
    do {
        slow = nums[slow];
        fast = nums[nums[fast]];
    } while (slow != fast);
    slow = nums[0];
    while (slow != fast) {
        slow = nums[slow];
        fast = nums[fast];
    }
    return slow;
}
```

---

# Blank-editor templates

Copy the shape, then rename variables to match the problem.

## Binary search: first true

```cpp
int lo = L, hi = R;          // inclusive, pred(R) is true
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (pred(mid)) hi = mid;
    else lo = mid + 1;
}
// lo is the first index where pred is true
```

## Monotonic stack: next greater to the right

```cpp
vector<int> nextGreater(vector<int>& a) {
    int n = (int)a.size();
    vector<int> ans(n, -1);
    stack<int> st;
    for (int i = 0; i < n; i++) {
        while (!st.empty() && a[i] > a[st.top()]) {
            ans[st.top()] = a[i];
            st.pop();
        }
        st.push(i);
    }
    return ans;
}
```

## 0/1 knapsack boolean

```cpp
// dp[x] = can we make sum x using each num at most once
vector<char> dp(target + 1);
dp[0] = 1;
for (int num : nums) {
    for (int x = target; x >= num; --x)
        dp[x] = dp[x] || dp[x - num];
}
```

## Fast and slow pointers

Middle: `while (fast && fast->next) { slow = slow->next; fast = fast->next->next; }`. Slow stops at the middle. For “middle of two centers, the left one”, start `fast` one ahead or stop when you need the predecessor: `while (fast->next && fast->next->next)`.

Cycle: if they meet, there is a cycle. Entrance: reset one pointer to the head, then move both one step.

## Graph BFS skeleton

```cpp
queue<int> q;
vector<int> dist(n, -1);
dist[src] = 0; q.push(src);
while (!q.empty()) {
    int u = q.front(); q.pop();
    for (int v : g[u]) if (dist[v] == -1) {
        dist[v] = dist[u] + 1;
        q.push(v);
    }
}
```

## Interval DP skeleton

```cpp
// dp[i][j] = best cost to solve the open interval (i, j)
// need points sorted, including sentinels at both ends
for (int len = 2; len < m; len++) {
    for (int i = 0; i + len < m; i++) {
        int j = i + len;
        dp[i][j] = INF;
        for (int k = i + 1; k < j; k++)
            dp[i][j] = min(dp[i][j], dp[i][k] + dp[k][j] + cost(i, j));
    }
}
```

---

# What to try, in order, when nothing matches

1. Write the brute force in a comment. Name its complexity. The optimized solution is usually that search with a pruned state, a better order, or a data structure that makes the inner check O(1) or O(log n).
2. Constraints pick the family. `n ≤ 20` → subsets or meet-in-the-middle. `n ≤ 100` and pairs → O(n^3) interval or Floyd. `n ≤ 1e5` → sort, heap, binary search, hash, DSU, two pointers, monotonic stack, linear DP.
3. Sorted input → two pointers or binary search before DP.
4. “Minimum maximum” or “maximum minimum” → binary search the answer, then a greedy `ok(mid)`.
5. Contiguous + add/remove at the ends only → sliding window. If the window answer does not move monotonically, it is not a sliding window; think prefixes.
6. Tree, “return something to my parent” → post-order DFS. If you also need a global best path, update it at each node from the children’s return values (diameter pattern).
7. Graph with dependencies → topo. Graph with “fewest steps” → BFS. Graph with non-negative weights → Dijkstra. Negative weights → Bellman-Ford. All pairs and n is small → Floyd.
8. Duplicates in `1..n` or a missing number → index as hash, or Floyd if the array is read-only.
9. After you have a candidate algorithm, state the invariant in one sentence. If you cannot, you do not have the algorithm yet.
