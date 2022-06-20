---
layout: post
title: "Sorting Algorithms"
author: Job Hernandez
---
An algorithm is a sequence of steps that transform a given input to a corresponding output --- i.e., an algorithm is a computational procedure [1]. An algorithm is a mathematical object about which we can reason and prove properties; for example, some properties include how fast the algorithm is and how much memory the algorithm consumes. Without algorithms such as the RSA algorithm -- which is the basis for encryption -- the internet, as we know it, would not be possible.

In this essay we will focus on the sorting problem. Cormen [1] defined the sorting problem as follows:

input: a sequence of $$ n $$ numbers $$ $$ a_1 $$, $$ a_2 $$, $$ \dots $$, $$ a_n $$
output: a permutation $$ a_1 $$, $$ a_2 $$, $$\dots$$, $$ a_n $$ such that
$$ a_1 $$ $$\leq$$ $$a_2$$ $$\leq$$ $$\dots$$ $$\leq$$ $$a_n$$.


# Sorting Algorithms

As mentioned above, we can prove mathematical properties of algorithms. In addition to how fast or how much memory an algorithm consumes we can prove the correctness of an algorithm; an algorithm is correct if it computes the correct output for every input in finite time [1].

To prove the correctness of an algorithm we must figure out its loop invariant. A loop invariant is a pattern or property that holds before the first iteration, is maintained at each iteration, and holds after the loop terminates. This will be discussed shortly.

## Mergesort

Mergesort is an effecient sorting algorithm. It is implemented using a technique called Divide and Conquer. In the $$mergesort$$ algorithm you Divide the problem by creating two sub-arrays and you Conquer by sorting both of these sub-arrays and, finally, you combine these two sorted sub-arrays. Divide and Conquer is recursive in nature.

Suppose you have two piles of cards both of which are sorted. In both piles of cards the smallest cards are on the top. The auxiliary procedure $$ merge $$ works by placing the smallest card from these two piles into the output pile until one pile is empty and then placing the remaining cards from the other pile on top of faced down.

The loop invariant for the mergesort algorithm is the following. 

At each iteration the sub-array $$A$$ containing the elements $$ p \dots k - 1 $$ $$ consists of the $$ k - p $$ smallest elements from the sub-arrays $$ L $$ containing the elements $$ 1 \dots n1 + 1 $$ and $$ R $$ containing the elements $$ 1 \dots n2 + 1 $$. Furthermore, $$ L[$$ i $$] $$ and $$ R[$$ i $$] $$ are the smallest elements that have not been placed in the output pile.

Prior to the first iteration the sub-array $$A$$ containing the elements $$ p \dots k-1 $$  contains the first $$ k - p $$ smallest elements of the sub-array $$ L $$ and $$ R $$. Since prior to the first iteration of the loop $$ k = p = 0 $$ then it contains the $$ 0_th $$ smallest elements -- trivially, of course. And since $$ i = j = 0 $$  $$L[i]$$ and $$R[j]$$ are the smallest elements -- i.e., cards have not been placed in the output pile.

Suppose that $$ L[i] $$ is less than or equal to $$ R[j]$$. Then this element is the smallest element of the two piles. We know that the sub-array $$ A $$ containing $$ p \dots k - 1 $$ contains the smallest $$ k - p $$ elements of the two sub-arrays. So, when we $$ L[i] = A[k] $$ then the array $$ A $$ $$ p \dots  $$ consists of $$ k - p + 1 $$ of the two sub-arrays. Incrementing $$ i $$ and $$ k $$ restablishes the loop invariant.

By the loop invariant the sub-array $$ A $$ contains $$ p \dots k - 1 $$ which is the array consisting of $$ p \dots r $$  which contains the smallest $$ k - p $$ elements of the two sub-arrays.

### Code

Here is the implementation of the mergesort algorithm in CLRS:

{% highlight python %}

import math

def merge_sort(A, p, r):
    """
    Divides A into two sub-arrays and sorts them recursively and then
    merges the two sub-arrays.
   
    @param A: the given array that needs to be sorted
    @param p: the start of the first sub-aray 
    @param r: the end of the second sub-array
    @return: a sorted array

    """
    if p < r:
        mid = (p + r) // 2
        merge_sort(A, p, mid)
        merge_sort(A, mid+1, r)
        merge(A, p, mid, r)

    return A
    
def merge(A, p, q, r):
    """
    Combines two sorted lists
    
    @param A: the array to be sorted
    @param p: the left start of the first sub-array
    @param q: the mid point of the two sub-arrays
    @param r: the end of the second sub-array
    @return: a sorted array
    """
    n1 = q - p + 1
    n2 = r - q

    L = [0] * (n1 + 1)
    R = [0] * (n2 + 1)

    for i in range(0,n1):
        L[i] = A[p + i]

    for j in range(0,n2):
        R[j] = A[q + 1 + j]

    L[n1] = math.inf
    R[n2] = math.inf

    i = 0
    j = 0

    for k in range(p, r+1):
        if L[i] <= R[j]:
            A[k] = L[i]
            i = i + 1
        else:
            A[k] = R[j]
            j = j + 1

    
        
A = [3, 6, 4, 1, 2, 5]
print(merge_sort(A, 0, 5))
{% endhighlight %}


## Insertion Sort

As you can see in the below insertion sort implementation the array INTEGERS is sorted in place; that is, without creating additional space.

At each iteration of the for loop $$ key $$ holds the current value which in the beginning is initialized to $$ i = 1 $$; moreover, in the beginning $$ j = i - 1 $$.

Essentially, this algorithm sorts the input by shifting, through the while loop, $$ integers[i-1] $$ , $$ integers[i-2] $$, $$ integers[i-3] $$ and so one by one position to the right until $$ key $$ is inserted in the correct position.
 
Before the first iteration $$ i = 1 $$; consequently, the subarray $$ integers $$ containing the elements $$ 0 \dots i - 1  $$ is sorted since $$ 0 ... i - 1 = 0 $$ so the subarray $$ integers[0] $$ is sorted.

As I said above, the algorithm works by shifting the elements $$ i - 1 $$, $$ i - 2 $$, $$ i - 3 $$ and so on by one position to the right until it finds the correct position of $$ i $$; so, the subarray comprised of $$ 1 ... i $$ is sorted. This is demonstrates that the loop invariant is maintained.

The loop invariant holds when the loop terminates becasue the sub-array comprised of the original elements $$ 0 ... n $$ is now sorted. Where $$ n $$ is the length of $$ integers $$.

### Code

{% highlight python %}
import unittest2

def insertion_sort(integers):
    """
    Sorts the list INTEGERS.

    :param integers: list of integers
    :returns: sorted list of integers
    """
    for i in range(1, len(integers)):
        key = integers[i]
        j = i - 1
        while j > -1 and integers[j] > key:
            integers[j+1] = integers[j]
            j -= 1
        integers[j+1] = key

    return integers
{% endhighlight %}