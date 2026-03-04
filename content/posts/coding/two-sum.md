---
title: "Two Sum"
date: 2026-03-03T14:20:47-06:00
draft: false
toc: false
images:
tags:
  - untagged
---

Today we are given a problem "Two Sum".

The question states:

> Given an array of integers `nums` and an integer `target`, return indices of the two numbers such that they add up to `target`.
>
> You may assume that each input would have exactly one solution, and you may not use the same element twice.
>
> You can return the answer in any order.

## Examples

**Example 1:**

- **Input:** `nums = [2,7,11,15]`, `target = 9`
- **Output:** `[0,1]`
- **Explanation:** Because `nums[0] + nums[1] == 9`, we return `[0, 1]`.

**Example 2:**

- **Input:** `nums = [3,2,4]`, `target = 6`
- **Output:** `[1,2]`

**Example 3:**

- **Input:** `nums = [3,3]`, `target = 6`
- **Output:** `[0,1]`

## Constraints

- `2 <= nums.length <= 10^4`
- `-10^9 <= nums[i] <= 10^9`
- `-10^9 <= target <= 10^9`
- Only one valid answer exists.

**Follow-up:** Can you come up with an algorithm that is less than O(n²) time complexity?

---

## Brute force approach

We can try doing 2 for loops nested within each other:

```cpp
for (int i = 0; i < n - 1; i++) {
    for (int j = i + 1; j < n; j++) {
        if (nums[i] + nums[j] == target) {
            return {i, j};
        }
    }
}
```

This is fairly simple: we have a for loop inside a for loop. The inner loop goes through all the numbers (e.g. 1–9) while the outer loop stays on 1 until the inner loop is finished. When it's finished, the outer loop goes to 2 and the inner loop runs again. Although this may seem the easiest approach, it is not the most efficient, since we are going through the array twice and comparing every pair.

This works but we can do better the above is brute force with time complexity O(n²) because we iterate through the array twice (nested for loop).

---

## Hash map approach

We can be more efficient by building a `numMap` to keep track of numbers we've seen and checking for the complement as we go:

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> numMap;
        int n = nums.size();
        for (int i = 0; i < n; i++) {
            int complement = target - nums[i];
            if (numMap.count(complement)) {
                return {numMap[complement], i};
            }
            numMap[nums[i]] = i;
        }
        return {};
    }
};
```

Here we create a map and use the idea: **complement = target - current number**. With that, we are not looking for an exact match in the array we subtract the target from the current number and check our map for that complement. If it exists, we've found the two indices and return the solution.
