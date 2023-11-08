---
title: Sorting Algorithm Questions
date: 2023-11-07 03:04:00
categories: 
  - Algorithm
tags: 
  - sorting algorithm
  - power button
  - questions
  - recognize
  - development
  - array
  - companies
  - frequency
description: How do you brush up on your power button? It is recommended to start with sorting questions, from sorting a chained table -> the Kth largest element of an array, etc. This article takes you through the sorting high frequency interview questions of the big Internet companies.
cover: https://raw.githubusercontent.com/zqwuming/blogimage/img/img/2226192700.webp
---
Table of Contents

1. Common Sorting Algorithms
2. Getting Started
  3. Sorting Chained Tables
  4. Merge intervals
5. Advanced Topics:
  6. kth largest element in an array
  7. Finding the median of two positively ordered arrays
8. Summary

## 1. Common Sorting Algorithms

Common sorting algorithms include Insertion Sort [O(n^2)], Summation Sort [O(nlogn)], Heap Sort [O(nlogn)], and Quick Sort [O(nlogn) -> O(n^2)].

If you are not familiar with the sorting algorithms, you can learn them on [Visualgo Algorithmic Data Structures](https://visualgo.net/en/sorting"), which has videos and algorithm definitions, so I won't repeat them in the article.

## 2. Introductory topics

#### 1) Sorted Chained Lists

Power button topic 148:

![](https://s2.loli.net/2023/11/07/n7rJVAGXWEs8fbg.webp)

This is leetcode question 148, which is simple: sort an unordered linked table, with the advanced requirement: sort the linked table in `O(nlogn)` time complexity and constant-level space complexity.

Among the sorting algorithms **The algorithms with O(nlongn) time complexity are subsumption sort, heap sort, and fast sort**, but the worst time complexity of fast sort is O(n^2) which is not applicable in this problem, and heap sort is a better sorting algorithm for chained lists compared to subsumption sort.

So, we use the subsumption sort to realize this question, which is based on the idea of **Partition Recursion** with the following steps:

1. divide the chain table from the middle node into two, you can use fast and slow pointer way to realize;
2. sort the two sub-chained tables separately;
3. the sorted list for the merger, you can get a complete sorted list.

The Go code is as follows:

```css

func sortList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
    // 快慢指针，找到链表中间节点
    slow, fast := head, head
    for fast.Next != nil && fast.Next.Next != nil {
        fast = fast.Next.Next
        slow = slow.Next
    }
    next := slow.Next
    slow.Next = nil
    return mergeTwoList(sortList(head), sortList(next))
}
​
// mergeTwoList 合并两个子链表
func mergeTwoList(h1, h2 *ListNode) *ListNode {
    pre := new(ListNode)
    hair := pre
    for h1 != nil && h2 != nil {
        if h1.Val < h2.Val {
            pre.Next = h1
            h1 = h1.Next
        } else {
            pre.Next = h2
            h2 = h2.Next
        }
        pre = pre.Next
    }
    if h1 != nil {
        pre.Next = h1
    }
    if h2 != nil {
        pre.Next = h2
    }
    return hair.Next
}
```

The code is very simple and efficient to write in recursive fashion:

![](https://s2.loli.net/2023/11/07/itN1aSVIZf6WJOs.webp)

#### 2) Merging intervals

Force Buckle Topic 56:

![](https://s2.loli.net/2023/11/07/PhBNYRkCbdumDXT.webp)

The general idea is to merge an array of intervals so that there are no overlapping intervals after the merge. Suppose there are intervals [1,3] and [2,6], which should be merged into [1,6].

The idea of solving this problem is to compare the 2nd element of all the subarrays with the 1st element of the array that follows, and if there is a crossover, find a larger element to be the 2nd element of the new array. That is, when comparing `[1, 3] &#x548C; [2, 6]`, we first compare 3 is greater than 2, so we can see that there is a crossover between the two arrays, and 6 is greater than 3, so the 2nd element of the new array is 6 => `[1, 6]`.

How can we be sure that the first element is unique? For example, if we have 2 arrays: `[3, 5], [1, 8]`, we'll have to determine the size of the first element when we merge them, so for simplicity we can sort all the arrays first, making sure that the array with the smaller first element comes first. In Go, you can implement array sorting with simple functions:

```go
sort.Slice(arr, func(i, j int) bool {
    return arr[i][0] < arr[j][0]
})
```

Next, we can write the full code:

```go
import "sort"
​
func merge(intervals [][]int) [][]int {

    sort.Slice(intervals, func(i, j int)bool{
        return intervals[i][0] < intervals[j][0]
    })
    var res [][]int
    for _, interval := range intervals{
        var size = len(res)

        if size > 0 && interval[0] -1][1] {
            res[size-1][1] = max(res[size-1][1], interval[1])
        }else{
            res = append(res, interval)
        }
    }
    return res
}
​
func max(x int, y int) int {
    if x < y {
        return y
    }
    return x
}
```

More than 94% efficient golang commits:

![](https://s2.loli.net/2023/11/07/pAWK23rjuYncdFM.webp)

## 3. Advanced Topics

## 1) Finding the Kth largest element of an array

Power Buckle topic 215:

![](https://s2.loli.net/2023/11/07/C1Ls9EgmNOtFqTk.webp)

K largest element of the array, this question is the Internet factory interview questions, I have been tested in the byte, Baidu interview, so you must be proficient.

The meaning of the topic is very simple, is to find the kth largest element of the array sorted, such as [1,2,3,4,5], k = 2, then the second largest element is 4. If there is a topic with the same elements, we do not have to pay attention to, for example, [5,5,4,3,1], k = 2, then the second largest element is 5, rather than 4.

**The advanced requirement is**: design and implement an algorithm with `O(n)` time complexity to solve this problem.

This problem is a classic one in sorting algorithms, in which, if we don't require time complexity, we can just use all kinds of sorting algorithms to first sort the array and then get the kth largest element. However, whether in interviews or in daily use, we need to pursue the optimal time complexity of the algorithm, and efficient sorting algorithms are heap sort (nlogn), subsumption sort (nlogn) and quick sort (nlogn -> n^2).

How to satisfy the time complexity O(n)?

**Quick sort**

The answer is mentioned in 9.2 of Introduction to Algorithms, Third Edition, where the `randomized_select` algorithm is a selection algorithm that expects linear time, and is modeled after fast sort, where the idea is still recursive partitioning. Unlike fast sort, however, the `randomized_select` algorithm deals only with one side of the partition, which we describe in code:

```css
func findKthLargest(nums []int, k int) int {
    rand.Seed(time.Now().UnixNano())
    return quickSelect(nums, 0, len(nums)-1, len(nums)-k)
}
​
func quickSelect(a []int, l, r, index int) int {
    q := partition(a, l, r)
    if q == index {
        return a[q]
    } else if q < index {
        // 根据partition元素，只处理划分的一边
        return quickSelect(a, q + 1, r, index)
    }
    return quickSelect(a, l, q - 1, index)
}
​
func partition(a []int, l, r int) int {
    // 选取随机数作为边界元素
    i := rand.Intn(r-l+1) + l
    a[i], a[r] = a[r], a[i]

    x := a[r]
    i := l - 1
    for j := l; j < r; j++ {
        if a[j] i++
            a[i], a[j] = a[j], a[i]
        }
    }
    a[i+1], a[r] = a[r], a[i+1]
    return i + 1
}
```

Submission of results:

![](https://s2.loli.net/2023/11/07/3bJOeIBUTMpFdHL.webp)

We found that the optimized algorithm randomized_select time complexity for fast sort exceeds 99% of users.

**Heap Sort**

Next we implement it again with heap sort, which is a sorting algorithm designed to take advantage of the data structure **heap**. A heap is a structure that approximates a complete binary tree and simultaneously satisfies the heap form: i.e., a child node's key value is always less than (or greater than) the key value of its parent node:

* Small top heap: each node's value is less than or equal to the value of its child node;
* big top heap: each node's value is greater than or equal to the value of its child node.

This question is to find the largest kth number, so we build a big top heap, each time we find the largest number, remove the largest number (implementation: the top of the heap and the end of the heap elements are interchanged, and then the size of the heap will be subtracted by one), and then find the k-1th number, the code is as follows:

```css
func findKthLargest(nums []int, k int) int {
    heapSize := len(nums)
    buildMaxHeap(nums, heapSize)
    for i := len(nums) - 1; i >= len(nums)+1-k; i-- {
        nums[0], nums[i] = nums[i], nums[0]
        heapSize--
        maxHeapify(nums, 0, heapSize)
    }
    return nums[0]
}
​
func buildMaxHeap(nums []int, heapSize int) {
    for i := heapSize / 2; i >= 0; i-- {
        maxHeapify(nums, i, heapSize)
    }
}
​
func maxHeapify(nums []int, i, headSize int) {
    l, r, largest := i*2+1, i*2+2, i
    if l < headSize && nums[l] > nums[largest] {
        largest = l
    }
    if r < headSize && nums[r] > nums[largest] {
        largest = r
    }
    if largest != i {
        nums[i], nums[largest] = nums[largest], nums[i]
        maxHeapify(nums, largest, headSize)
    }
}
```

As you can see, heap sort has a time complexity of O(nlogn), which is a bit more time-consuming than `randomized_select` for fast sort:

![](https://s2.loli.net/2023/11/07/yBus9PRrZmHxjQA.webp)

### 2) Find the median of two positively ordered arrays

Force Buckle topic 4:

![](https://s2.loli.net/2023/11/07/PMi7WGlSxc5aIem.webp)

The general idea of the question is to find the median of two ordered arrays nums1 and nums2, and requires a time complexity of , which assumes that the two arrays will not be empty at the same time.

For this problem, the first and easiest way to think of is to merge the two arrays and take out the median. However, the operation of merging the arrays has a complexity of m+n, which does not meet the requirements of the question. Seeing the time complexity of log, we associate it with **binary search**. 1.

1. When the total number of two arrays is odd, the middle number is the median; when the total number is even, the mean of the two middle numbers is the median. 2;
2. know the length of the array, to find the median of the two arrays combined, only need to bisect the search to find the middle position of the array of 1 or 2 elements;
3. As the location of the middle number to determine, so the two arrays only need to ** bisect the search for an array ** can be, in order to minimize the complexity of our search for the length of the smaller arrays;
4. the most critical: bisection cut should be how to determine whether the end, we assume that there is an array A and array B (length of m and n, respectively), the first bisection cut on the array A, that cut point i, and j [due to i + j = (m + n)/2, so j = (m + n)/2 - i] need to be satisfied, * * A [i], B [j] the left-hand side of the elements should be smaller than the right-hand side of the elements, which can be guaranteed that i and j are one of the median elements**.

The graphical representation is as follows:

![](https://s2.loli.net/2023/11/07/ZH5C7TeK3zfhYxt.webp)

1 two-point split:

![](https://s2.loli.net/2023/11/07/lqZH839X1b7JfjK.webp)

2 two-point splits:

![](https://s2.loli.net/2023/11/07/psIZbPR1iuvBkjX.webp)

After splitting again, left = 3, right = 3, and the dichotomous cut of array A is complete. At this point, the cut subscripts for arrays A and B are i = left = 3, j = mid - left = 2, respectively.

![](https://s2.loli.net/2023/11/07/3Xx7IjZ1WMgAif5.webp)

The median result will only come from these `i, j, i-1, j-1` numbers. 1. when the total number of arrays is odd, the median is the larger of the left partition (i.e. i-1 and j-1):

1. when the total number of arrays is odd, the median is the larger of the left partition (i.e., i-1 and j-1);
2. when the total number of arrays is even, the median is the average of the largest number in the left partition and the smallest number in the right partition.

In addition, we have to consider boundary cases, such as when i, j is 0, the subscript i-1 or j-1 does not exist; when i, j is m, n, the subscript i or j does not exist. Since the title states that m and n will not be 0 at the same time, at least one of i and j must exist.

The Go code is as follows:

```go
func findMedianSortedArrays(nums1, nums2 []int) float64 {

   m, n := len(nums1), len(nums2)
   if m > n {
      return findMedianSortedArrays(nums2, nums1)
   }
   left, right, mid := 0, m, (m+n+1)/2
   for left < right {
      i := (left + right + 1) / 2
      j := mid - i

      if nums1[i-1] > nums2[j] {
         right = i - 1
      } else {
         left = i
      }
   }
   i, j := left, mid-left
   var leftMax float64
   if i > 0 && j > 0 {
      leftMax = float64(max(nums1[i-1], nums2[j-1]))
   } else if i > 0 {
      leftMax = float64(nums1[i-1])
   } else {
      leftMax = float64(nums2[j-1])
   }
   if (m+n)%2 == 1 {
      return leftMax
   }
   var rightMin float64
   if i < m && j < n {
      rightMin = float64(min(nums1[i], nums2[j]))
   } else if i < m {
      rightMin = float64(nums1[i])
   } else {
      rightMin = float64(nums2[j])
   }
   return (rightMin+leftMax)/2
}
​
func max(a, b int) int {
    if a>b {
        return a
    }
    return b
}
​
func min(a, b int) int {
    if areturn a
    }
    return b
}
```

Submission of results:

![](https://s2.loli.net/2023/11/07/uRUK9QIeg1Dk6br.webp)

## 4. Summary

A few of the classic questions in the Power Buckle Sorting category are above, and their probability of being encountered in interviews or written exams is very high. In Huawei's machine test questions, the frequency is also very high, except for the last question, which is a bit less difficult, the rest of the questions basically appear at least once in 10 interviews.

