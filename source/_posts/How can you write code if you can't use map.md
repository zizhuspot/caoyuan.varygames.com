---
title: How can you write code if you can't use map?
date: 2023-11-06 14:04:00
categories: 
  - Backend
tags: 
  - Backend Technology Sharing
  - Go
  - data structure
  - language
  - development
  - framework
  - network
  - understand
description: Whether it is the usual development, or in the Go language technical interviews, map is very difficult to get around the topic. So, do you understand the underlying implementation mechanism of map?
cover: https://raw.githubusercontent.com/zqwuming/blogimage/img/img/5096e21c9e8d4c5e9ca92b2bd11d8393%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp
---
catalogs

1. introduction
2. map substructure
3. GET and PUT operations
4. DELETE operation
5. map expansion conditions
6. Considerations for using map
7. Postscript

## 1. Introduction

As a gopher (go as the development language of the program ape), every time in the face of the technical face of the Internet factory, will be asked to some go data structure or its unique mechanism of the underlying implementation.

For example, when I interviewed for the "Tencent TEG" department, an elegant and easy-going interviewer sitting across the video conference asked, "You usually use go language, right, so tell me what reference types are in go language?"

I thought, this is not difficult to me, so slowly answered: " **go language reference types and slice (slice), pipeline (channel), function (func), interface (interface ) and dictionary (map) ** ".

The interviewer added, "I see that you have not worked for a long time, so I won't ask you about the underlying mechanism of channel, which may be a bit difficult! In this way, you say a map of the underlying implementation, and map add and delete data and expansion of what operations".

"Hey, I'm so grumpy, it means I can't answer the question of channel, who am I looking down on?" I can't help but think that this interviewer is so ...... The bottom of the channel I understand the CSP, elegantly close the channel and other logic, if you ask about the source code implementation, I really will not!

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/e2a5fc8e8ef24fb9a7f769d257c9baee%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

However, the implementation mechanism of map in go is not very complicated, and there is a difference in design between other languages, and I spent time reviewing it before the interview. It seems that _the attentive interviewer is not only trying to find the most suitable candidate, but also the one who knows the most about them. _

## 2. map's underlying data structures

So, I'll describe the underlying implementation of a map in terms of its data structure, additions, deletions, and expansions.

First, we can find the underlying data structure code for map in the `runtime/map.go` package that comes with the go language:

```go
type hmap struct {
     count     int    
     B         uint8  
     overflow  uint16

     buckets    unsafe.Pointer
     oldbuckets unsafe.Pointer  

     extra *mapextra
     ...

 }
```

> Go maps are implemented as hmap structures. hmap records several attributes of a map, including the number of elements, the number of buckets [2^B], the addresses of the map's buckets, and the addresses of the overflow buckets [we'll talk about the concept of overflow buckets later, see below].

Among them, the structure of the extra overflow bucket is as follows, and the parameters record the address of the overflow bucket and the pointers to the upper and lower buckets, respectively:

```scss
 type mapextra struct {
     overflow    *[]*bmap
     oldoverflow *[]*bmap
     nextOverflow *bmap
 }
```

buckets records the address of the bmap that holds the map bucket (**a bucket can be thought of as a bmap bucket**). bmap is the key structure that records the actual data of the map, and its structure when it is not compiled is as follows:

```go
 type bmap struct {
     tophash [bucketCnt]uint8
 }
```

> tophash records the first 8 bits of the key in the map data, used to quickly compare whether the key exists in the map or not.

While the program is being compiled, the structure of bmap is as follows:

```go

 type bmap struct {
     tophash [8]uint8
     data    byte[1]  
     overflow uintptr
 }
```

The `overflow` in bmap records the address of the next bmap, where overflow is of type `uintptr` instead of *bmap, in order to **guarantee that the bmap does not contain any pointers at all, so as to minimize gc scanning**.

But then the overflow bucket will be missing references, and may be gc'd when used because the element is empty, so hmap adds `extra` to store the pointer to the overflow bucket.

The `data` parameter stores the real map data (k-v key-value pairs), each bmap stores 8 k-v key-value pairs, and key-value pairs in the bmap are stored **separately and consecutively** as shown below:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/85f63ffeee214f089a5315dff0db167f%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

> When there are more than 8 k-v pairs, a new bmap is generated to store the data, and the overflow records the address of the new bmap.

**summarize**

In the go implementation of map, the structure represented is hmap, and hmap maintains a number of bucket buckets (i.e., bmap buckets, which are subsequently called buckets for the sake of uniformity). For intuitive understanding, we refine the key fields in hmap to graphical patterns:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/5096e21c9e8d4c5e9ca92b2bd11d8393~tplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp) Each element of the `buckets` array is a bmap structure. Each `bmap` bucket holds 8 k-v pairs, and if there are more than 8, a new bmap will be generated, and the overflow will record the address of the new bmap.

## 3. GET and PUT operations

After understanding the data structure of a map, we will learn how to access the data in a map.

## 3.1 GET Getting Data

Assuming that B=4, i.e. the number of buckets is `2^B = 16`, suppose we want to get the value corresponding to key5 from the map.

```go
m := make(map[string]string, 0)
...

fmt.Println(m[key5])
```

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/6d4c84a8a39a4127989e8ca328c4b88c%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

As shown above, the get process is divided into. 1:

1. Calculate the hash value of k5 [64-bit operating system, the result has 64 bits]. 2;
2. determine which bucket is in the last B bit of the **hash value **, 0100 is converted to decimal 4, so it is in bucket 4. 3. determine which bucket is in the first 8 bits of the hash value and the `tophash value;
3. according to the hash value of the first 8 bits and `tophash` for comparison, quickly determine in which bucket position, the next step. 4. the query key5 and the `tophash` to the next step;
4. compare the key5 of the query with the key5 of the `bmap`, and get the corresponding value5 if it matches exactly. 5. if no key5 is found in the current bucket, then get the corresponding value5;
5. if not found in the current bucket, go to the next overflow bucket connected by `overflow`, and repeat steps 3-4.

### Repeat steps 3-4. 3.2 Storing Data in the PUT

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/4dd5ce329ba140b99eb579a7cc3a8566%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

The map assignment can be divided into the following steps (assuming we add key6):

* Determine which bucket it is by the last B bits of the hash value of key6;
* Iterate over the current bucket, compare the first 8 bits of the hash value of the key with `tophash` to prevent the key from being repeated, and then find the location where the elements are stored, i.e., insert the data at the first empty position of the **bmap**;
** If the current bucket is full, a new `bmap` bucket is created to insert the elements, then the address of the new bmap is recorded with `overflow` and a reference to the new bucket is added to the `extra` pointer array in the hmap.

### 3.3 About hash conflicts

**A hash conflict occurs when two different keys fall into the same bucket. When the bucket is not full, it is inserted using open addressing to find the first empty space from front to back. When the bmap bucket already has 8 k-v key-value pairs, a new overflow bucket (bmap) is created and the address of the new bmap is recorded by `overflow` and a reference to the new bucket is added to the hmap's `extra` pointer array.

## 4. The DELETE operation

When a map is deleted:

* If the deleted element is a value type, such as int, float, bool, string, and array, the map's memory is not automatically freed;
* If the element is a reference type, such as pointer, slice, map, and chan, part of the map memory will be freed, but the memory freed is the memory occupied by the reference type of the child element, and the memory occupied by the map itself is not affected;
* The memory will be released only after map is set to nil, and the memory will be reclaimed in the next GC.

So, ** map will not release memory immediately after deletion**, let's verify this.

### 4.1 When map is a basic type

```scss

var intMap map[int]int
var cnt = 8192
​

func printMemStats() {
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
​
    log.Printf("Alloc = %vKB, TotalAlloc = %vKB, Sys = %vKB, NumGC = %v\n",
              m.Alloc/1024, m.TotalAlloc/1024, m.Sys/1024, m.NumGC)
}
​
func initMap() {
    intMap = make(map[int]int, cnt)
    for i := 0; i < cnt; i++ {
       intMap[i] = i
   }
}
​

func delMapKey() {
    for i := 0; i < cnt; i++ {
       delete(intMap, i)
   }
}
​
func main() {
    printMemStats()
​
    initMap()
    log.Println("after initMap, len(map) =", len(intMap))

    runtime.GC()
    printMemStats()
​
    delMapKey()
    log.Println("after delMapKey, len(map) =", len(intMap))
​
    runtime.GC()
    printMemStats()
​
    intMap = nil
    runtime.GC()
    printMemStats()
}
```

The final printout is:

> Alloc = 108, TotalAlloc = 108, Sys = 6292, NumGC = 0
> after initMap, len(map) = 8192
> Alloc = 410, TotalAlloc = 424, Sys = 6867, NumGC = 1
> after delMapKey, len(map) = 0
> Alloc = 410, TotalAlloc = 425, Sys = 6931, NumGC = 2
> Alloc = 99, TotalAlloc = 427, Sys = 6931, NumGC = 3

Where Alloc is the memory space occupied by the current heap object in KB, TotalAlloc is the cumulative space allocated for heap objects, Sys is the memory space occupied by the operating system, and NumGC is the number of garbage collection attempts. It can be clearly seen that ** when map deletes a basic type element, no matter how much GC is done, the memory is not freed. **

### 4.2 When map values are reference types

```go
type Person struct {
    Name string
    Age  int
}
var intMap map[int]*Person
​
func initMap() {
    intMap = make(map[int]*Person, cnt)
    for i := 0; i < cnt; i++ {
       intMap[i] = &Person{
          Name: "zhangsan",
          Age:  20,
      }
   }
}
```

Change the value type of map to a reference type, leave the rest of the code unchanged, and run the result again:

> Alloc = 108, TotalAlloc = 108, Sys = 6036, NumGC = 0
> after initMap, len(map) = 8192
> Alloc = 601, TotalAlloc = 615, Sys = 6867, NumGC = 1
> after delMapKey, len(map) = 0
> Alloc = 409, TotalAlloc = 615, Sys = 6931, NumGC = 2
> Alloc = 99, TotalAlloc = 618, Sys = 6931, NumGC = 3

Comparing the two examples, we can find that when the map value is a basic type, the deletion of the key will not release space; while when the map value is a reference type, it will release part of the space (space on the heap of the value), but the memory occupied by the map itself is not reduced, why is this?

This is because the bottom of map will only increase or decrease the number of bmap buckets after ** applying for them**. The delete operation of the map will only set the data on the bmap to nil (if the value is of pointer type, the pointer object will be reclaimed in the next GC), but the memory occupied by the bucket itself remains unchanged, so the memory occupied by the map itself will not be reduced because of the deleted key.

In particular, if there is an empty space in the bmap bucket after a key is deleted, and a new key is added later, the empty space in the bucket may be filled**, and the memory footprint of the map remains unchanged.

### 4.3 How to Solve Memory Leakage Caused by a Map

When a map is frequently added or deleted, or too many keys are added (triggering a map expansion), even if ** these keys are deleted, the memory space will still be unavailable, thus triggering a memory leak. ** This can be solved by the following method.

We can use the following methods to solve the problem:

** Set the map that is no longer in use to nil, or restart the system periodically to allow the map to be reallocated;
* When the value of a map consumes too much memory, change the value to a pointer;
* Periodically copy the elements of the map to another map.

Generally speaking, most of the Internet businesses with high traffic impact are ToC scenarios, and the on-line frequency is very high. Some businesses may go online several times a day and restart and recover before the problem is exposed, which is not a big problem 🐶.

## 5. map expansion conditions

## 5.1 Expansion of the same capacity

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/7e62339e501c41f19cee9914e23038e3%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

Since map constantly puts and deletes elements, there may be a lot of intermittent empty bits in the bucket, which can cause bmap to overflow the bucket by a lot, resulting in longer scan times. So, this kind of expansion is actually a kind of collation, organizing the data of the backward bits to the front. ** With equal expansion, the elements are rearranged, but the buckets are not swapped, and the number of element buckets does not increase. **

### 5.2 Two-volume expansion

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/26c90c2ccd704c6ebaafd3fe153011da%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

When the bucket array is not enough, the expansion will occur, then, ** elements will be rearranged, bmap bucket increase, the key elements inside the bucket may also be migrated **.

As shown in the figure, before expansion B = 2, after expansion B = 3, assuming that an element key hash value of the last three for 101, then by the introduction of the above can be seen, before expansion, hash value of the last two to determine the number of buckets. That is, ** before expansion, the last two digits of an element are 01, and the element is in bucket 1; after expansion, the last three digits of an element are 101, and the element needs to be migrated to bucket 5**.

### 5.3 Conditions for expansion to occur

#### 1) Expansion condition 1: loading factor > 6.5

Under normal circumstances, if there are no overflow buckets, there are at most 8 elements in a bucket. When the average number of elements in each bucket exceeds 6.5, it means that the capacity of the current bucket is almost full and needs to be expanded.

> loadFactor = number of elements in map / number of current buckets in map, i.e. loadFactor = count / (2^B)
> From the formula, **loadFactor is the average number of elements in each bucket in the current buckets**.

#### 2) Expansion Condition 2: Excessive number of overflow buckets

* When B < 15, if the number of overflowed bmaps reaches 2^B;

* When B >= 15, if the number of overflow's bmap exceeds 2^15, then start to expand the capacity.

The reason for too many overflow buckets is that a large number of keys in the map have the same hash value after the B-bit, which makes individual bmap buckets keep inserting new data, which leads to longer and longer chains of overflow buckets.

Thus, when the map is being added, deleted, modified and checked, the scanning speed will become slower and slower. When expanding the capacity, you can rearrange the **elements of these overflow buckets to make the position of the elements in the bucket more average**, so as to improve the efficiency of scanning.

#### 3) Details when expanding

1. In our hmap structure, there is an oldbuckets, when the expansion occurs, the old data will be stored in this first and then expand the buckets, at this time, the capacity of the buckets is twice the oldbuckets. 2. map expansion is an incremental increase in the capacity of the oldbuckets, so that the oldbuckets can be expanded;
2. map expansion is an incremental expansion, not a full expansion, and every time the map is deleted or modified, the operation of migrating from oldbuckets to buckets will be triggered. This is because full expansion with a large number of keys will consume a lot of resources and cause the program to stall. 3;
3. Before the migration is complete, each get or put traverses the oldbuckets before traversing the buckets.

## 6. Notes on map

#### 1) Do not address elements

** As the elements of a map grow, the map bottom may reallocate space, resulting in invalid addresses before ** Let's look at an example:

```go
type Student struct {
    Name string
    Age  int
}
​
func f1() {
    m := map[int]Student{
        1: Student{Age: 15, Name: "jack"},
        2: Student{Age: 16, Name: "danny"},
        3: Student{Age: 17, Name: "andy"},
    }
    m[1].Name = "JACK"
}
```

A compilation error occurs in this case because map cannot be addressed. That is, you can get m[1], but you cannot make any changes to its value. If you want to modify value, you can use value with a pointer as follows:

```css
func f2() {
    m := map[int]*Student{
        1: &Student{Age: 15, Name: "jack"},
        2: &Student{Age: 16, Name: "danny"},
        3: &Student{Age: 17, Name: "andy"},
    }
    m[1].Name = "JACK"
}
```

#### 2) Thread unsafe

Suppose the number of barrels of a map is 4, i.e., B=2, and the number of current elements is also 4, i.e., the barrel is full. At this time, two goroutines (g1 and g2) read and write to this map, g1 inserts key1 and g2 reads key2, the following may happen:

1. g2 calculates the hash value of key2 [1101..... .101], B=2, and determines the bucket number to be 1;

2. g1 adds key1, which triggers the expansion condition, increasing B to 3 and expanding the number of buckets to 8. 3;
3. The key in the map starts to migrate, assuming that the migration is completed soon, and key2 is migrated from bucket 1 to bucket 5. 4. g2 is migrated from bucket 1 to bucket 5;

4. g2 traverses from bucket 1 and fails to get data!

So, when manipulating a map, you can **use Go's own sync.RWMutex locks, or use sync.Map (which supports concurrent and shared locks) to ensure thread safety**.

## 7. Postscript

After answering these questions, the interviewer's face remained unperturbed, and he said lightly, "Why don't we move on to talk about channel?"

So, I disliked all the points I had learned, and the interviewer seemed to recognize my level, so he didn't give me a hard time anymore and started the next topic.

After the interview, I couldn't help but think: "It's worthy of being a Penguin factory, it seems that the candidates are all strong!"

Nowadays, in the background of the Internet market is so cold, we want to meet the technical side of the big Internet factories, we must not only have a wide application of the technology we have learned, but also have to spend some effort on the cognition of the underlying mechanism.

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/daf762c3893a4e0196b21f49bb6cf6e3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

After all, anyone can memorize the eight-stud text, anyone can brush the algorithmic questions, the interviewer in the screening also had to increase the difficulty, in order to test the candidate's breadth and depth of knowledge! The gap between you and others may be the distance of a map.

