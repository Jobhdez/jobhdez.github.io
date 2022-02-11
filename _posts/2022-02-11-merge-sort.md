---
layout: post
title: "The Mergesort Algorithm"
author: Job Hernandez
---
# Mergesort work

## How does mergesort work?

Mergesort is implemented using a technique called Divide and Conquer. In the mergesort algorithm you Divide the problem by creating two sub-arrays and you Conquer by sorting both of these sub-arrays with mergesort and, finally, you combine these two sorted sub-arrays. Divide and Conquer is recursive in nature.

Suppose you have two piles of cards both of which are sorted. In both piles of cards the smallest cards are on the top. The auxiliary procedure $$ merge $$ works by placing the smallest card from these two piles into the output pile until one pile is empty and then placing the remaining cards from the other pile on top of faced down.

## Loop Invariant
The loop invariant for the mergesort algorithm is the following. 

At each iteration the sub-array $$ A[$$ p...k - 1 $$] $$ consists of the $$ k - p $$ smallest elements from the sub-arrays $$ L[$$ 1 ... n1 + 1 $$] $$ and $$ R[$$ 1 ... n2 + 1 $$] $$. Furthermore, $$ L[$$ i $$] $$ and $$ R[$$ i $$] $$ are the smallest elements that have not been placed in the output pile.

## Initialization
Prior to the first iteration the sub-array $$A[$$ p ... k-1 $$] $$ contains the first $$ k - p $$ smallest elements of the sub-array $$ L $$ and $$ R $$. Since prior to the first iteration of the loop $$ k = p = 0 $$ then it contains the 0th smallest elements -- trivially, of course. And since $$ i = j = 0 $$ the the $$L[$$i$$]$$ and $$R[$$j$$]$$ are the smallest elements -- i.e., cards have not been placed in the output pile.

## Maintenance
Suppose that $$ L[$$ i $$] $$ is less than or equal to $$ R[$$ j $$]$$. Then this element is the smallest element of the two piles. We know that the sub-array $$ A[$$ p...k - 1 $$] $$ contains the smallest $$ k - p $$ elements of the two sub-arrays. So when we $$ L[$$ i $$] = A[$$ k $$] $$ then $$ A[$$ p...k $$] $$ consists of $$ k - p + 1 $$ of the two sub-arrays. Incrementing $$ i $$ and $$ k $$ restablishes the loop invariant.

## Termination
By the loop invariant the sub-array $$ A[$$ p...k - 1 $$] $$ which is  $$ A[$$ p ... r $$] $$ contains the smallest $$ k - p $$ elements of the two sub-arrays.

# Code

Here is the implementation of the mergesort algorithm in CLRS:

{% highlight python %}

import math

def merge_sort(lst, start, end):
    """
    Divides the lst into two sublists and sorts them recursively and then
    merges the two sublists.
   
    @param lst: then given list that needs to be sorted
    @param start: the start of the first sublist 
    @param end: the end of the second sublist 
    @return: a sorted list

    """
    if start < end:
        mid = (start + end) // 2
        merge_sort(lst, start, mid)
        merge_sort(lst, mid+1, end)
        merge(lst, start, mid, end)

    return lst
    
def merge(lst, left, middle, right):
    """
    Combines two sorted lists
    
    @param lst: the list to be sorted
    @param left: the left start of the first sublist
    @param middle: the mid point of the two sublists
    @param right: the end of the second sublist 
    @return: a sorted list
    """
    n1 = middle - left + 1
    n2 = right - middle

    l_lst = [0] * (n1 + 1)
    r_lst = [0] * (n2 + 1)

    for i in range(0,n1):
        l_lst[i] = lst[left + i]

    for j in range(0,n2):
        r_lst[j] = lst[middle + 1 + j]

    l_lst[n1] = math.inf
    r_lst[n2] = math.inf

    i = 0
    j = 0

    for k in range(left, right+1):
        if l_lst[i] <= r_lst[j]:
            lst[k] = l_lst[i]
            i = i + 1
        else:
            lst[k] = r_lst[j]
            j = j + 1

    
        
lst = [3,6,4,1,2,5]
print(merge_sort(lst, 0, 5))
{% endhighlight %}