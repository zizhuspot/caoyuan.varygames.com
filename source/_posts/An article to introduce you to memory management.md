---
title: An article to introduce you to memory management
date: 2023-11-06 03:04:00
categories: 
  - Backend
tags: 
  - Backend
  - Interview
  - candidate
  - knowledge
  - development
  - framework
  - Computer Composition Principles
  - Memory management
description: Memory management, an inescapable topic for developers in the process of program writing and tuning, is also a must-know computer knowledge towards senior programmers. Experienced interviewers will look at a candidate's skill level in terms of their mastery of memory management.
cover: https://raw.githubusercontent.com/zqwuming/blogimage/img/img/c957903820544270a9faa96e14005d9d%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp
---
Catalog

>

1. Introduction
2. Virtual Memory
3. Memory Management
4. Escape Analysis
5. Summary

## 1. Introduction

Memory management is a topic that developers can't avoid in the process of writing and tuning programs, and it's also a computer knowledge that must be understood by senior programmers.

Experienced interviewers will look at a candidate's skill level in terms of how well he or she masters memory management. The knowledge involved may include operating systems, principles of computer composition, and the underlying implementation of programming languages.

When it comes to memory, which is actually storage, we can look at von. Neumann's computer structure to understand the concept of memory:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/af282acb032a448db07e7b78ca08b048%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

As you can see, memory is an integral part of a computer. Memory management, in fact, is the management of memory storage space.

Next, we will learn about memory management in terms of memory classification, and memory space allocation in the Go language, combined with common escape analysis scenarios.

## 2. Virtual Memory

## 2.1 The Difference Between Virtual Memory and Physical Memory

As we all know, computer memory was very small in the past, and we had a very limited scope of physical addressing when running computer programs.

For example, on a 32-bit machine, the addressing range is only 2 to the 32nd power, that is, 4G, and this is fixed for the program, so we can imagine that if every computer process is allocated 4G of physical memory, it will consume too much resources.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/c957903820544270a9faa96e14005d9d%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

Resource utilization is also a huge problem. Processes that are not allocated a resource have to wait for it, and when a process is finished, the waiting process is loaded into memory, and this frequent loading is also very inefficient.

Moreover, since the instructions are accessible to the physical memory, any process can modify the data of other processes in the memory or even modify the data in the kernel address space, which is very unsafe.

Due to the high resource consumption, low utilization and insecurity when physical memory is used. Therefore, **Virtual Memory** was introduced.

Virtual memory is a technique for memory management in computer systems, ** by allocating virtual logical memory addresses so that each application program thinks it has continuously available memory space. ** In reality, these memory spaces are usually multiple fragments of physical memory that are partitioned, with some temporarily stored on external disk storage for data exchange when needed.

### 2.2 Virtual Memory Conversion

Since computers use virtual memory, how do we get the real physical memory addresses? The answer is memory mapping, i.e. how to convert virtual addresses (also known as logical addresses) into physical addresses.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/a9ad96e8700243fc934533467d40a769%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

Under the Linux operating system, memory is first managed in two ways, page storage management and segmented storage management, where:

* Page-based storage effectively addresses memory fragmentation and improves memory utilization;
* Segmented storage management reflects the logical structure of the program and facilitates segment sharing;

In layman's terms there are two units of memory, one is paging and the other is segmentation. ** Paging is the process of cutting up the entire virtual and physical memory space into many fixed-size chunks ** with mapping between virtual and physical addresses through ** page tables **:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/6938f7fab54747baa593d3c773245001%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

Paged memory is pre-divided, so it doesn't create memory fragments with very small gaps, and is more efficiently utilized when allocated.

This is not the case with segmentation, which is based on the logic of the program, and since program attributes can vary greatly, the size of the segments can also vary. In segment management, virtual and physical addresses are mapped through a **segment table**:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/1f2ca2495b474ece81bc940e3d70e3b4%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

It is easy to see that the slice of segmented memory management is not uniform, but is allocated according to the memory occupied by different programs. The problem brought about by this is that, assuming that after the memory of program 1 (1G) is used up and released, another program 4 (assuming that the memory needs to be 1000M) is loaded into the physical memory and may still have 24M of memory left, if there are a large number of such memory fragments in the system, then the overall memory utilization will be very low.

Thus, the segment-and-page memory management approach emerged, which combines the above two methods of memory management: i.e., **first divide the user program into a number of segments, assign a segment name to each segment, and then divide each segment into a number of pages**.

In a segment-and-page system, in order to realize the translation from logical address to physical address, the system needs to be configured with both segment and page tables, and the segment and page tables are utilized for mapping from user address to physical memory space. The system creates a segment table for each process and a page table on each segment. The segment table contains the segment number, page table length, and page table start address, and the page table contains the page number and block number.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/42af202fc5e1400593debfb8a0f96c0c%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

During address translation, the segment table is used to find the page table address, then the page table is used to get the page frame number, and finally the physical address.

The mapping of virtual memory to physical memory is managed at the operating system level. When we are developing, the memory management involved is often only the work that the software program has to do when it calls the virtual memory:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/2c25f47d44174ab69b75120cb4fd50c1%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

Next, we analyze memory management in software development in terms of what constitutes virtual memory.

## 3. Memory Management

The program is divided into five parts on the virtual memory: stack area, heap area, data area, global data area, and code segment.

Memory management, on the other hand, is the rationalization of the use of memory space, mainly the allocation and use of the two important areas of the heap (Heap) and stack (Stack).

### 3.1 Heap and Stack

There are two important address spaces in virtual memory, the heap and the stack. For underlying programming languages such as C++, the memory space on the stack is managed by the compiler, while the memory space on the heap needs to be managed manually by the programmer for allocation and reclamation.

In **Go, the memory space on the stack is also managed by the compiler, while the memory space on the heap is managed by both the compiler and the garbage collector for allocation and reclamation**, which brings great convenience to us programmers.

The overhead of allocating and reclaiming memory on the stack is very low, requiring only 2 instructions: PUSH and POP. PUSH presses the data into the stack and POP frees the space, consuming only the time it takes to copy the data into memory.

When allocating memory on the heap, it is not only slower to allocate, but also more difficult to garbage collect. For example, Go has used the three-color marking method + mixed write barrier technique to do garbage collection since 1.8. Overall, heap memory allocation leads to much higher overhead than stack memory allocation.

> To learn more about Go garbage collection, check out this article: [Go Language Garbage Collection](https://link.juejin.cn?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzI5Nzk2MDgwNg%3D%3D% 26mid%3D2247484217%26idx%3D1%26sn%3Dd464e95dcec6252b5e00a5f634a46892%26chksm%26 3Decac5730dbdbde26005ba864ab324a3a2c9b90f42c5ce037eb4f5eee36a4e55a4385421d9dd3%26scene%3D21%23wechat_redirect "http://mp.weixin.qq. com/s?__biz=MzI5Nzk2MDgwNg==&mid=2247484217&idx=1&sn=d464e95dcec6252b5e00a5f634a46892&chksm= ecac5730dbdbde26005ba864ab324a3a2c9b90f42c5ce037eb4f5eee36a4e55a4385421d9dd3&scene=21#wechat_redirect")

### 3.2 Stack memory allocation

#### 1) Memory allocation challenges

* Memory space is requested by user programs like C/C++, which may make frequent memory requests and reclaims, but every time a memory allocation requires a system call (i.e., memory can only be requested by entering the kernel state), it results in low performance.
* In addition to this, there may be multiple threads (there are also co-threads in Go) accessing the same address space, which will definitely require locking the memory, and will bring more overhead.
* Initially, the heap memory is a contiguous block of memory, but as the system continues to request and reclaim memory, many memory fragments may be created, resulting in a less efficient use of memory.

In order to cope with the above three most common problems when a program performs memory allocation, Go language has made some improvements by combining Google's TCMalloc (ThreadCacheMalloc) memory reclamation method. At the same time, TCMalloc and Go memory allocation will introduce ** thread cache (mcentral of P), central cache (mcentral) and page heap (mheap) ** three components for hierarchical management of memory. This is shown in the figure:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/1f325f633ae545b9891820aebc2e1c41%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

Thread cache belongs to each individual thread or co-thread, which stores the memory block span used by each thread, and since the size of the memory block varies, there are hundreds of memory block class span classes, which manage different sizes of memory space (e.g., 8KB, 16KB, 32KB, ...). The Since there is no multi-threading involved, there is no need to use mutual exclusion locks to protect the memory to minimize the performance loss caused by lock contention.

When the space of the thread cache is not enough, the center cache is used as the allocation of memory for small objects. The center cache corresponds to each span class of the thread cache one by one, and there are two memory blocks in each span class of the center cache, storing the allocated memory space and the full memory space respectively, in order to improve the efficiency of memory allocation. If the center cache is not satisfied, it is like page heap for space request.

In order to improve the utilization of space, when encountered in the medium-large object (> = 32KB) allocation, the memory allocator will choose the page heap directly for allocation.

The core of Go language memory allocation is the use of **multilevel caching** to categorize objects based on size and implement different allocation policies according to the category. As shown in the above figure, the application will request memory space from different components according to the size of the object (Tiny or Large and medium).

#### 2) Stack Memory Allocation

The memory in the stack area is usually allocated and released automatically by the compiler. Generally speaking, the stack area stores function inputs and local variables, which will be created with the creation of the function, and perish with the return of the function, and generally will not exist in the program for a long time.

This linear memory allocation strategy is extremely efficient, but engineers often have no control over the allocation of stack memory, which is basically done by the compiler.

The stack space contains two important global variables at runtime, `runtime.stackpool` and `runtime.stackLarge`, which represent the global stack cache, which allocates less than 32KB of memory, and the large stack cache, which allocates more than 32KB of stack space:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/f11965a043f8481ba6696c74f58d090b%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

When allocating stack space, depending on the size of the thread cache and the request stack, the Go language allocates stack space in three different ways:

1. if the stack space is small, allocate memory using the global stack cache or a fixed-size free chain table on the thread cache;
2. if the stack is large, it is allocated from the global `runtime.stackLarge` stack cache;
3. if the stack is large and `runtime.stackLarge` is insufficient, request a sufficiently sized piece of memory on the heap.

Since Go 1.4, the minimum stack memory size is 2KB, which is the size of a goroutine program. So when the number of goroutines in a program exceeds the maximum amount of stack memory that can be allocated, it will be allocated on the heap. In other words, although Go can allocate an unlimited number of goroutines with the go keyword, performance-wise it is better not to allocate more goroutines than the maximum amount of stack space.

Assuming that the maximum stack memory is 8MB, it is best to allocate no more than 4000 goroutines (8MB/2KB).

## 4. Escape analysis

## 4.1 How Go does escape analysis

In programming languages like C and C++, which require manual memory management, the allocation of objects or structures to the stack or heap is up to the engineers, which poses a challenge: how to accurately allocate a reasonable amount of space for each variable, so as to improve the efficiency of the whole program and the efficiency of memory usage. However, this manual allocation of memory in C and C++ leads to the following two problems:

1. Objects that don't need to be allocated on the heap are allocated on the heap - wasting memory space;
2. objects that need to be allocated on the heap are allocated on the stack - creating wild pointers and compromising memory safety;

Compared to wild pointers, wasted memory space is a minor issue. In C, it is a common error for a variable on the stack to be returned to the caller as a return value by a function, in the code shown below, the variable `i` on the stack is returned incorrectly:

`` arduino
int *dangling_pointer() {
   int i = 2;
   return &i.
}
```

When the `dangling_pointer` function returns, its local variables are reclaimed by the compiler (the mechanism of space on the stack), and the caller acquires dangerous wild pointers. If there are a lot of illegal pointer values inside the program, it is more difficult to find and locate them in large projects.

> When an object is freed or reclaimed, but no modifications are made to the pointer so that it still points to a reclaimed memory address, the pointer is called a wild pointer, or a dangling pointer, or a lost pointer. --wikipedia

So, in Go, how does the compiler know whether a variable needs to be allocated on the heap or the stack to avoid this problem?

**The way the compiler decides where to allocate memory is called escape analysis**. Escape analysis is done by the compiler and works during the compilation phase. In compiler optimization, escape analysis is the method used to determine the dynamic scope of pointers.The compiler of the Go language uses escape analysis to determine which variables should be allocated on the stack and which should be allocated on the heap.

This includes memory that is implicitly allocated using methods such as `new`, `make`, and literals. The Go language's escape analysis follows the following two invariants:

1. pointers to stack objects cannot exist in the heap;
2. a pointer to a stack object cannot survive a stack object reclamation.

What does this mean? Let's translate:

* First, if a pointer to the heap points to a stack object, then the stack object's memory needs to be allocated to the heap;
* If the pointer survives after the stack object is reclaimed, then the object can only be allocated to the heap.

When we perform memory allocation, the compiler will follow the above two principles to allocate memory to either the stack or the heap for the variables or objects we request.

In other words, when we allocate memory in a way that violates one of these two principles, a variable that was intended for the stack may "escape" to the heap, which is called a memory escape. A program with a large number of memory escapes is bound to have unintended negative consequences, such as slow garbage collection and memory overflows.

### 4.2 Four Escape Scenarios

In Go, memory escapes on the stack can occur due to the following four scenarios.

**1. Pointer escapes

Pointer escape is easy to understand, when we create an object in a function, the life cycle of the object ends with the end of the function, and that's when the object's memory is allocated on the stack.

And if a pointer to an object is returned, in this case, the function exits, but the pointer is still there, the object's memory can not be reclaimed with the end of the function, so it can only be allocated on the heap.
```

```go
package main

type User struct {
    ID     int64
    Name   string
    Avatar string
}

func GetUserInfo() *User {
    return &User{
        ID: 666666,
        Name: "sim lou",
        Avatar: "https://www.baidu.com/avatar/666666",
    }
}

func main() {
    u := GetUserInfo()
    println(u.Name)
}
```

In the above example, if the `User` object had been returned instead of the object pointer `*User`, it would have been a local variable and would have been allocated on the stack; conversely, the pointer, being a reference, would have continued to be used in the main function, and thus memory would have been allocated only to the heap.

We can look at variable escapes with the compiler command `go build -gcflags -m main.go`:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/b93cc2d097d54f8e80c89cde05d00e0a%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

`&User{...} escapes to heap` That means the object escaped to the heap.

**interface{} Dynamic type escaping

In Go, a null interface, `interface{}`, can represent an arbitrary type. If a function argument is an interface{}, it is difficult to determine the exact type of the argument during compilation, and it will escape. For example, the Println function has an interface{} null input:

```go
func Println(a . .interface{}) (n int, err error)
```

This returns a User object, which also escapes, but the escape node is when the fmt.Println function is used:

```go
func GetUserInfo() User {
    return User{
        ID: 666666,
        Name: "sim lou",
        Avatar: "https://www.baidu.com/avatar/666666",
    }
}

func main() {
    u := GetUserInfo()
    fmt.Println(u.Name) 
}
```

**3. Insufficient stack space

The operating system has a size limit on the amount of stack space used by kernel threads, usually 8 MB on 64-bit Linux systems.You can use the ulimit -a command to see how much memory the stack is allowed to take up on your machine.

```css
root@cvm_172_16_10_34:~ # ulimit -a
-s: stack size (kbytes)             8192
-u: processes                       655360
-n: file descriptors                655360
```

Because the stack space is usually small, improper implementation of recursive functions can lead to stack overflow.

For Go, the runtime tries to dynamically allocate stack space when the goroutine needs it, and the initial stack size of a goroutine is 2 KB. When the goroutine is dispatched, it binds to the kernel thread for execution, and the size of the stack does not exceed the operating system's limit.

For the Go compiler, local variables that exceed a certain size will escape to the heap, and the size limit may be different for different Go versions. Let's do an experiment (note that when allocating int[], int takes up 8 bytes, so 8192 ints is 64 KB):

```go
package main

import "math/rand"

func generate8191() {
    nums := make([]int, 8192) 
    for i := 0; i 8192; i++ {
        nums[i] = rand.Int()
    }
}

func generate8192() {
    nums := make([]int, 8193) 
    for i := 0; i 8193; i++ {
        nums[i] = rand.Int()
    }
}

func generate(n int) {
    nums := make([]int, n) 
    for i := 0; i func main() {
    generate8191()
    generate8192()
    generate(1)
}
```

The compilation results are as follows:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/e6d2d75636734c378b7a770d1b6492d4%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

It can be noticed that `make([]int, 8192)` does not escape, and `make([]int, 8193)` and `make([]int, n)` escape to the heap. In other words, when the memory occupied by a slice exceeds a certain size, or the current slice length cannot be determined, the memory occupied by the object will be allocated on the heap.

**4. closures**

> The combination of a function and a reference to its surrounding state (lexical environment) bound together (or the function surrounded by references) is a closure. That is, a closure lets you access the scope of an inner function within the scope of its outer function.
> - Closures

Memory escapes also occur in the Go language when using closure functions. Look at a sample code:

```go
package main

func Increase() func() int {
    n := 0
    return func() int {
       n++
       return n
  }
}
func main() {
    in := Increase()
    fmt.Println(in()) 
    fmt.Println(in()) 
}
```

`Increase()` The return value of the function is a closure that accesses the external variable n, which will remain until in is destroyed. Obviously, the memory occupied by the variable n cannot be reclaimed with the exit of Increase(), so it will escape to the heap.

### 4.3 Performance Improvement with Escape Analysis

**Passing a value vs. passing a pointer

Passing a value copies the entire object, while passing a pointer only copies the pointer address, which points to the same object. Passing pointers reduces the copying of values, but causes memory allocations to escape to the heap, increasing the burden on garbage collection (GC). In scenarios where objects are created and deleted frequently, the GC overhead caused by passing pointers can have a serious impact on performance.

In general, pass pointers for structures that require modification of the original object's value, or occupy a relatively large amount of memory. For a read-only structure with a small memory footprint, passing a value directly can achieve better performance.

## 5. Summary

Memory allocation is the core logic of runtime memory management. The Go program's runtime memory allocator uses an allocation strategy similar to `TCMalloc` to classify objects according to their size, and designs a multi-layer cache component to improve the performance of the memory allocator. Understanding the design and implementation principles of the Go memory allocator can help us understand the different choices made by different programming languages when designing memory allocators.

Stack memory is an important memory space in an application, which can support local local variables and function calls. Variables in the stack space are created and destroyed along with the stack, and this part of the memory space doesn't require too much intervention and management by engineers. Modern programming languages reduce our workload through escape analysis, and understanding the allocation of the stack space can be of great help in understanding the runtime of the Go language.
