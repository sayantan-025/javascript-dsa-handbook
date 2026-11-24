# 167. Two Sum II - Input Array Is Sorted

**Primary pattern:** Opposite-end two pointers  
**Platform:** LeetCode · **Difficulty:** Medium · **Topic:** Arrays and pair search  
[Problem statement](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/description/)

> **Recognition rule:** A sorted array and a target pair sum let you discard one endpoint after each failed comparison.

## 1. Problem and constraints

Given an integer array `numbers` sorted in nondecreasing order and an integer `target`, find two distinct elements that sum to `target`. Return their **1-based indices** in increasing order.

- `2 ≤ numbers.length ≤ 30,000`
- `-1000 ≤ numbers[i], target ≤ 1000`
- Exactly one valid pair of indices exists.
- The solution must use **O(1) extra space**.

| Numbers | Target | Expected output | Explanation |
|---|---:|---|---|
| `[2, 7, 11, 15]` | 9 | `[1, 2]` | `2 + 7 = 9` |
| `[2, 3, 4]` | 6 | `[1, 3]` | `2 + 4 = 6` |
| `[-1, 0]` | −1 | `[1, 2]` | `-1 + 0 = -1` |

## 2. Pattern recognition and intuition

The direct approach checks every pair. At the maximum input size, that can mean **449,985,000 comparisons**. The sorted order lets one comparison rule out several pairs at once.

Start with `i` at the first element and `j` at the last. For a fixed `i`, `numbers[j]` is the **largest** available partner. For a fixed `j`, `numbers[i]` is the **smallest** available partner.

| Current sum | Move | Why the endpoint can be discarded |
|---|---|---|
| `sum < target` | Increment `i` | Even the largest remaining partner gives a sum below `target` with `numbers[i]`. Every smaller partner also falls short. |
| `sum > target` | Decrement `j` | Even the smallest remaining partner gives a sum above `target` with `numbers[j]`. Every larger partner also exceeds it. |
| `sum === target` | Return `[i + 1, j + 1]` | The current endpoints are the required pair. |

Duplicates preserve these bounds because the array is **nondecreasing**. Negative numbers also preserve them because adding the same value to both sides of an inequality preserves the order.

## 3. Brute force

Fix each index `i` and check every later index `j`. Starting `j` at `i + 1` ensures that each pair uses distinct positions and is checked at most once.

```javascript
function twoSumBrute(numbers, target) {
  for (let i = 0; i < numbers.length - 1; i++) {
    for (let j = i + 1; j < numbers.length; j++) {
      if (numbers[i] + numbers[j] === target) {
        return [i + 1, j + 1];
      }
    }
  }
}
```

**Worst-case time:** O(n²). **Auxiliary space:** O(1).

## 4. Intermediate approach: Binary search

For each `i`, compute the complement `target - numbers[i]`. Binary search the sorted range from `i + 1` to the end. Searching only this range keeps the returned indices distinct and increasing.

```javascript
function twoSumBinary(numbers, target) {
  for (let i = 0; i < numbers.length - 1; i++) {
    const complement = target - numbers[i];
    let low = i + 1;
    let high = numbers.length - 1;

    while (low <= high) {
      const mid = low + Math.floor((high - low) / 2);

      if (numbers[mid] === complement) {
        return [i + 1, mid + 1];
      }

      if (numbers[mid] < complement) {
        low = mid + 1;
      } else {
        high = mid - 1;
      }
    }
  }
}
```

**Worst-case time:** O(n log n). **Auxiliary space:** O(1). Each starting position can still trigger a separate search.

## 5. Optimal approach and submitted solution

1. Initialize `i = 0` and `j = numbers.length - 1`.
2. While `i < j`, compute `numbers[i] + numbers[j]`.
3. If the sum is below `target`, advance `i`. If it is above `target`, move `j` back.
4. On equality, return both indices plus one.

```javascript
/**
 * @param {number[]} numbers
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(numbers, target) {
   let i = 0;
   let j = numbers.length - 1;

   while(i < j){
       let currSum = numbers[i] + numbers[j];
       if(currSum < target){
           i++;
       }else if(currSum > target){
           j--;
       }else{
           return [i+1, j+1];
       }
   }
};
```

The submitted solution is correct and uses the optimal two-pointer approach. The guarantee of exactly one valid pair makes the return inside the loop sufficient.

## 6. Visual dry run

`numbers = [-4, -1, 0, 3, 7, 10]`, `target = 2`

The numbered columns are zero-based JavaScript indices. Bold values mark the endpoints before each comparison.

| Step | 0 | 1 | 2 | 3 | 4 | 5 | Sum | Action |
|---:|---:|---:|---:|---:|---:|---:|---:|---|
| 1 | **−4** | −1 | 0 | 3 | 7 | **10** | 6 | `6 > 2` → decrement `j` |
| 2 | **−4** | −1 | 0 | 3 | **7** | 10 | 3 | `3 > 2` → decrement `j` |
| 3 | **−4** | −1 | 0 | **3** | 7 | 10 | −1 | `-1 < 2` → increment `i` |
| 4 | −4 | **−1** | 0 | **3** | 7 | 10 | 2 | Return `[2, 4]` |

The match is at zero-based indices `1` and `3`, so the returned indices are `2` and `4`.

## 7. Why it works

**Loop invariant:** Before each comparison, both indices of the valid pair lie in the inclusive range `[i, j]`.

- **Initialization:** The first range covers the entire array.
- **Preservation:** If the sum is too small, the endpoint bound rules out every pair using `i`. If the sum is too large, the other bound rules out every pair using `j`. Each pointer move therefore keeps the valid pair inside the range.
- **Termination:** Each failed comparison shrinks the range by one. The valid pair occupies two distinct positions, so the pointers must find it before they meet.

## 8. Complexity comparison

| Approach | Worst-case time | Auxiliary space |
|---|---:|---:|
| Brute force | O(n²) | O(1) |
| Iterative binary search per index | O(n log n) | O(1) |
| Opposite-end two pointers | O(n) | O(1) |

The two-pointer solution makes one comparison if the initial endpoints match and at most `n - 1` comparisons overall. It stores two indices and the current sum.

**Why linear time is optimal in the general sorted-array model:** Consider an array with negative even values `-2m, …, -2` and positive odd values `3, 5, …, 2m + 1`. Lowering any one positive value by `1` preserves the sort order and creates exactly one pair summing to `0`. In the worst case, locating the changed position requires checking a linear number of candidates.

## 9. Edge cases

| Case | Numbers | Target | Expected |
|---|---|---:|---|
| Minimum length with equal values | `[3, 3]` | 6 | `[1, 2]` |
| Two negative values | `[-5, -3, -1]` | −8 | `[1, 2]` |
| Negative value and zero | `[-1, 0]` | −1 | `[1, 2]` |
| Duplicate zeros | `[0, 0, 1]` | 0 | `[1, 2]` |
| Match at the initial endpoints | `[2, 3, 4]` | 6 | `[1, 3]` |
| Maximum length | 29,998 zeros followed by `1, 2` | 3 | `[29999, 30000]` |

## 10. Common pitfalls and JavaScript details

- Initialize `j` to `numbers.length - 1`, the last valid JavaScript array index.
- Keep `i < j` so one position cannot be used twice.
- Move only the endpoint ruled out by the comparison. The other endpoint may still be part of the answer.
- Add `1` to both indices when returning. Array access uses zero-based indices, and the problem expects 1-based indices.
- JavaScript `Number` represents every possible sum here exactly. Each input value is between `-1000` and `1000`, so every sum is between `-2000` and `2000`.

## 11. Quick revision

> **Too small → move `i` right. Too large → move `j` left. Match → return both indices plus one.**

The current right endpoint gives the largest possible partner for `i`. The current left endpoint gives the smallest possible partner for `j`. Those bounds explain which endpoint can be discarded.

**O(n) time · O(1) auxiliary space**
