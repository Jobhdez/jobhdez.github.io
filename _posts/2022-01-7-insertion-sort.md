---
layout: post
title: "The Insertion Sort Algorithm"
author: Job Hernandez
---
# How does the insertion sort algorithm works?

As you can see in the below insertion sort implementation the list INTEGERS is sorted in place; that is, without creating an additional list which would require additional space.

At each iteration of the for loop $$ key $$ holds the current value which in the beginning is initialized to $$ i = 1 $$; moreover, in the beginning $$ j = i - 1 $$.

Essentially, this algorithm sorts the input by shifting, through the while loop, $$ integers[i-1] $$ , $$ integers[i-2] $$, $$ integers[i-3] $$ and so one by one position to the right until $$ key $$ is inserted in the correct position.

### Loop Invariant
A loop invariant is a pattern that holds before the first iteration, is maintained at each iteration and it holds when the loop terminates. In this case, the loop invariant is the observation that in the beginning of each iteration the subarray $$ integers[$$ 0 ... i - 1 $$] $$ is sorted comprised of the original elements i.e., $$ 0 ... i - 1 $$ in the list $$ integers $$.

#### Initialization 
Before the first iteration $$ i = 1 $$; consequently, the subarray $$ integers[0 ...i - 1] $$ is sorted since $$ 0 ... i - 1 = 0 $$ so the subarray $$ integers[0] $$ is sorted.

#### Maintenance 
As I said above, the algorithm works by shifting the elements $$ i - 1 $$, $$ i - 2 $$, $$ i - 3 $$ and so on by one position to the right until it finds the correct position of $$ i $$; so, the subarray comprised of $$ 1 ... i $$ is sorted.

#### Termination
The sublist comprised of the original elements $$ 0 ... n $$ is now sorted. Where $$ n $$ is the length of $$ integers $$.

### Here is the implementation in Python

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

class test_insertion_sort(unittest2.TestCase):

    def test_algorithm(self):
        """
        Test if the insertion sort algorithm works.
        """
        self.assertEqual([2,3,1,5,4,8,7], [1,2,3,4,5,7,8])

if __name__ =='__main__':
    unittest2.main()
{% endhighlight %}
            