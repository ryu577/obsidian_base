This is an incomplete draft. The complete version: https://medium.com/@rohitpandey576/p-np-would-mean-were-a-bunch-of-dumb-apes-20c6e50f0ba3

**I) Introduction**

You might have heard about the famous P=NP conjecture and the fact that there is a million dollar prize for proving or disproving it.It is concerned with the different problems we can encounter and how many of these we can solve with our computers and clever algorithms. There are many online posts explaining these terms. I would often read one of those posts, think I understood the concept only to not be able to recall the distinctions a month later.I decided to sit down and understand this in a way that it would stick.And my goal with this post is that a reader with an understanding of high school math be similarly able to recite these concepts decades after reading this. A key gap in other posts I think I filled here is accompanying concrete examples. 


**II) Big-O notations**

Best to jump straight into an example. Consider the function y=x. Imagine it’s “competing” against another equation, y=a x^2 + b x + c x (with a>0). We want to see which of the two takes a larger value for x values that are >0. 

  

The concept of complexity can easily be extended to problems with many inputs (and not just one). For example in graph theory, the size of the graph is defined by the number of edges and the number of vertices. In this case, we have the run-time of the algorithm growing with two variables and not just one. But the 

  

-   Generalizes readily to multiple dimensions. Example, graphs.
-   Ex: algorithm for path cover.

  

**III) The class P**

The book, “Introduction to algorithms” by Cormen et.al. defines in chapter 34 the class P of all problems that are solvable in polynomial time in the size of inputs. This would mean the problem of sorting an array is in P.

  

However, per most textbooks dedicated to complexity theory, the class (or set) P is defined as the set of all “decision problems” that are solvable in time polynomial with respect to their inputs. Meaning if you plot the size of the input on the x-axis and the run-time of the program that solves the problem on the y-axis, you’ll get a function that can be dominated by some polynomial (or is O(n^k) for some k). The “decision problems” referenced in the definition are problems that have a yes/ no output. It makes defining relationships between various classes we’re going to define convenient. 



This is a little annoying however, since most problems of interest are more than decision problems. For instance, consider sorting an array of numbers. There is no binary decision here, just a sorted list to be returned. And so, this problem is technically not in the set P even though there are many efficient algorithms that can solve it in O(n^2) time. In the book, “Computational complexity: a modern approach” by Arora and Barak, this is listed as a “criticism” of the way the class P is defined. 



But, a lot of times, for any given problem, we can find a decision problem in P that is equivalent to our non-decision problem. For instance, for the problem of sorting an array, the best algorithm for counting the number of inversions of the array we know is via merge sort (we’re making an assumption something more efficient doesn’t exist). To find the number of inversions then, you need to sort the array. Now we can construct a decision problem by setting a threshold on the number of inversions, asking if it’s greater than some threshold value. And the most efficient way to answer this would be to sort the array and find the number of inversions in the process, then compare them to the threshold.

  

  

  

**IV) Beyond P: “Not P”?**

The next class we consider is called NP (the title of this section was a joke, it doesn’t actually stand for “not P” but “non-deterministic polynomial time”).

  

We still stay within the paradigm of decision problems, but since we’re venturing outside class P, we consider including problems that _can’t_ be solved in polynomial time. So what’s the next best thing? What if someone handed you a solution to the problem. Is it at least possible to verify the solution in polynomial time? The set of decision problems for which this is true are in the class NP. 

  

The “non-deterministic polynomial” *vaguely* comes from non-deterministically having a solution handed to us and seeing if we got lucky. 

  

It is clear that the set P lies completely inside this new set NP. If we can solve a problem in polynomial time, we can simply solve it and then compare the result to the candidate solution we got (the results are just binary, since these are decision problems).

  

Today, we have many problems we can verify but not solve in polynomial time. These tend to be decision problems. Graph theory is full of these kinds of problems. A good example is the traveling salesman problem (its “decision flavor”). Given a set of cities and the distances between each pair, we need to tell the salesman the order in which he should visit the cities so that the total distance he travels is minimized. 

  

This is the optimization version of the problem and not in NP since its not a decision problem. To convert it into a decision problem, (a general trick), we compare the objective function to some threshold value. Here, we would say something like, “is there a tour that visits all cities and has a total length less than 1000 km”. If someone gave us a tour, we could easily trace it out, get its total length and see if this was less than 1000 km. So, this problem is in NP. So far, we have two sets sitting something like this:

  

<figure 1>
  

**V) NP-complete**

A key concept in the description of the NP-complete class is reducibility. We use it all the time when solving problems. For instance, a lazy and totally acceptable way to find the median of an array is to sort it first and then take the middle element (or the mean of the two middle elements if it has even entries). We have reduced the problem of finding the median to the problem of sorting. If the reduction is so expensive that it takes more than polynomial time, this of course defeats the purpose. We are only interested in “polynomial time transformations”. 

  

If problem A is reducible to problem B, then B is at least as hard as A. We are thinking of hardness here in very binary terms. If a problem belongs to the set P, it is easy and hard otherwise. 

  

_______

In 1978, Cook and Levin proved something quite incredible. They showed that all problems in the set NP could be reduced to one problem in polynomial time. This one problem was the 'Boolean satisfiability' problem. It's basically the same as solving a system of equations, only the equations have to be in terms of Boolean variables with OR conditions between them.

  

Its clear that the Boolean satisfiability problem is in No since we can plug a solution into the system of equations and check if its satisfied. Proving that all problems in ND can be reduced to this problem in polynomial time was a heavy weight to lift. But then, others started showing that there were problems that broken satisfiability it self could be reduced to in polynomial time. Which meant that those problems could also be used to solve any other problem in the set NP. This growing

set of problems which riled all the problems in NP was given the name Np-complete.

  

  

  

  

**VII) Bringing it all together in the famous diagram**

All of these sets are 

  

  

  

[https://en.wikipedia.org/wiki/NP-intermediate](https://en.wikipedia.org/wiki/NP-intermediate)

  

[https://cstheory.stackexchange.com/questions/79/problems-between-p-and-npc/6384#6384](https://cstheory.stackexchange.com/questions/79/problems-between-p-and-npc/6384#6384)

  

  

  
221014

#building/blog, #cs,  #cs/complexity,
keywords: np, nphard, cs, complexity

List of complexity classes: 

[https://en.wikipedia.org/wiki/List_of_complexity_classes](https://en.wikipedia.org/wiki/List_of_complexity_classes)

  

[https://en.wikipedia.org/wiki/NP-](https://en.wikipedia.org/wiki/NP-easy)easy

  

**V) NP-complete**

A key concept in the description of the NP-complete class is reducibility. We use it all the time when solving problems. For instance, a lazy and totally acceptable way to find the median of an array is to sort it first and then take the middle element (or the mean of the two middle elements if it has even entries). We have reduced the problem of finding the median to the problem of sorting. If the reduction is so expensive that it takes more than polynomial time, this of course defeats the purpose. We are only interested in “polynomial time transformations”. 

  

If problem A is reducible to problem B, then B is at least as hard as A. We are thinking of hardness here in very binary terms. If a problem belongs to the set P, it is easy and hard otherwise.

___________________

