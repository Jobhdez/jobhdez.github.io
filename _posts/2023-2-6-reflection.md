---
layout: post
title: "Lessons from studying Lisp"
author: Job Hernandez
---


## Intro
I have been programming in Common Lisp and Scheme for a while and I feel like I have wasted my time but I have a lot of good memories with my past experiences with Common Lisp and Scheme. This is a reflection of my experiences.

## Lesson 1: Thinking recursively
I failed at learning programming a few times in the beginning of my programming journey but thankfully I stumbled across “The Little Schemer” by Felleisen which was how I learned Scheme and started learning how to program. “The Little Schemer” is a book about learning how to think recursively. After reading the first 4 or 5 chapters I started expressing myself recursively in Scheme while writing a very little DSL. In every Scheme or Common Lisp project I worked on since then I have implemented recursive algorithms – recursion is a pretty natural way to do things in Lispy languages. So, I think this is the first lesson I learned from writing programs in Scheme/Lisp and studying Lisp books. A lot of programmers have great difficulty with recursion but working with Scheme taught me how to express myself recursively so I feel very grateful.

## Lesson 2: Tail Recursion
After I studied “The Little Schemer” I started studying “Structure and Interpretation of Computer Programs” and this is the book that really taught me how to program. SICP is a beautiful book; reading this book has been one of the best intellectual experiences I have had. I remember crying of joy several times and it blew my mind several times as well. One concept I learned from reading this book, that not many programmers realize, is tail recursion. That is, it is possible to write a recursive procedure that generates an iterative process. In most programming languages a recursive procedure generates a recursive process whose time complexity is O(n) and whose space complexity is O(n); but in Scheme, a recursive procedure may generate an iterative process – i.e., a process whose time complexity is also O(n) but whose space complexity is O(1). The real insight here is that for a given procedure that generates a recursive process one can also write a procedure that generates an iterative process. In Scheme there are no looping constructs for instance.
## Lesson 3: Appreciation for computer science
Another lesson I learned from studying SICP is a love for mathematics and computer science; for some reason, it made me love mathematics more and it helped me realize how deep computer science is. The way the authors of SICP described higher order procedures – e.g., how higher order procedures elevate the conceptual level of a given program – was a tremendously meaningful experience for me. So, in other words Scheme and SICP helped me appreciate computer science. In addition, my SICP studies helped me understand that interpreters/DSLs and compilers are the ultimate abstraction which is key for controlling complexity.

## Conclusion
There are more lessons such as learning to think in a functional way which is also a big lesson but I think the lessons I described above made me a better programmer. But I feel like I have not explored all the power that Lisp gives you; for example, I have not used macros a lot in my programs so I think I am going to stick with Lisp for some time still. I am not sure whether to use Common Lisp or Racket but I need to keep exploring this language.

Thanks!

Happy Programming
