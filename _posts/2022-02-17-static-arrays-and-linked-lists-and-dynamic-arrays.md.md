---
layout: post
title: "Static Arrays, Linked Lists, and Dynamic Arrays"
date: 2022-02-17
categories: algorithms
excerpt_separator: <!--more-->
---

What is difference between the data structures: static arrays, linked lists, and dynamic arrays? Why are dynamic arrays so much more useful, and how are they implemented?
<!--more-->Let's begin by looking at static arrays.

### Static Array

Static arrays are a data structure that has a fixed length n, storing n elements in a sequential manner. For example, a static array with size 2, that stores the values 1 and 2 would look like this:

> [1, 2]

Each element in a static array are stored using the same amount of bits as a word (the number of bits the processor uses, which is 64 bits for modern processors.)

Static arrays are stored in one contiguous chunk of memory, and since each element has equal size, this allows for random access to the array elements in constant time. To access the i<sup>th</sup> element of the array we would access a specific address calculated by the following equation:

> i<sup>th</sup> address = start address + i * element size

This means the time complexity for accessing an element and changing the value of an element in a static array is O(1). All you have to do to change the value is access it, then set the value if needed.

Static arrays are great when the elements do not change. But what if a new element is to be added to or deleted? The size of a static array is constant, so a new memory location with the corresponding size has to be allocated, and the index of elements may change, requiring them to be shifted. Static arrays are bad when the elements may change. Linked lists might do better.

### Linked Lists

A linked list is a data structure made of group of static arrays of size 2. These static arrays are called nodes, and store a value in the first index, and a pointer to the next node in the second index. A head pointer of the linked list points to the first node to start the chain. A tail pointer could be added to point to the end of the linked list, although it is not necessary. 

These nodes do not have to be stored in a contiguous chunk of memory, because each node has a pointer that points to the address of the next node. This means every time a new element is to be added or deleted at the start of the sequence, there is no need to allocate new memory space to the entire array. It also only takes O(1) time, because it just modifies the head pointer (and create a size 2 static array if adding). 

However, the downside of linked list is that it takes O(n) time to access elements that are not the start of the sequence. To access the 10<sup>th</sup> element it takes 10 pointer hops to find the element. Linked lists only allow for sequential access, unlike static arrays. Is there a data structure that gets the best of both worlds? 

### Dynamic Arrays

A dynamic array is a static array that has relaxed constraints on its size. A static array has its size to be equal to the number of elements it stores. A dynamic array is allocated more memory than needed at the time to anticipate for extra elements. This reduces the number of times the array has to be copied over to a new larger chunk of contiguous memory.

When the dynamic array does become full, the array has to be copied to a larger chunk of contiguous memory with the size `some_constant * current_size`. If this constant is 2 and the current size is 16, the array does not have to be copied again until the size reaches 32. While the operation of adding an element when the array is full is costly, it is not for every other instant. It has a amortized (averaged out over sequence of operations) complexity of O(1). 

Most programming languages have built in dynamic arrays, like python lists and vectors in C++ because of its efficiency.

### Summary

Static Array: <br>
- Access: O(1)
- add_at_start: O(n)
- add_at_end: O(n)
- add_at_i: O(n)

Linked List: <br>
- Access: O(n)
- add_at_start: O(1)
- add_at_end: O(n)
- add_at_i: O(n)

Dynamic Array: <br>
- Access: O(1)
- add_at_start: O(n)
- add_at_end: O(1)
- add_at_i: O(n)

This [MIT opencourseware video](https://www.youtube.com/watch?v=CHhwJjR0mZA&list=PLUl4u3cNGP63EdVPNLG3ToM6LaEUuStEY&index=2&ab_channel=MITOpenCourseWare) on the topic is highly recommended for further information.
