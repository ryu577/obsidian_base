Statistics and ML
- Hypothesis testing.
	- T-test, ANOVA, F-test, Chi-squared.
- How to evaluate the performance of models.
	- Precision, recall, AUROC. Wikipedia article on ROC curve.
- Linear regression
	- How to do it from scratch.
	- R^2, p-values of coefficients.
	- Duplicating the data - what will it do to the above?
	- Extension to probabilities: Logistic regression.
- ML concepts
	- Boosting and bagging; random forests are bagged.
	- Basics of neural networks: back-propagation, convolution (multiplication of polynomials), how they can represent any function (universal theorem).

Algorithms
- Introduction to Algorithms, CLR.
	- Big-O notation (chapter 3).
- DFS, BFS, graph traversal.
	- Counting clouds, detonate bombs.
	- Backtracking.
		- Generate permutations, letter combination of phone number.
	- Trees are special case of graphs.
		- In-order, post-order and pre-order are all DFS.
- Two pointer trick.
	- Container with most water.
	- Largest palindromic sub-string.
- Data structures
	- queue, stack.
	- Heap (priority queue)
		- Implementing it with arrays vs trees.
		- Meeting rooms I and II.
- Sorting, binary search.
	- Median of two sorted arrays.
- Dynamic programming.
	- Min coins to make change.
	- Longest increasing sub-sequence.
		- Geeks for geeks page

Data architecture, system design.
- SQL - inner joins, outer joins, left-outer joins.
	- Most important skill.
	- Put some thought into implementing from scratch.
- CAP theorem.
- ACID and BASE philosophies.
- REST API, HTTP, trigger based, cron based.
- Micro-services.

![[1698939535094.gif|400]]


[Consistency](https://en.wikipedia.org/wiki/Consistency_model "Consistency model"
Every read receives the most recent write or an error.

[Availability](https://en.wikipedia.org/wiki/Availability "Availability")
Every request receives a (non-error) response, without the guarantee that it contains the most recent write.

[Partition tolerance](https://en.wikipedia.org/wiki/Network_partitioning "Network partitioning")
The system continues to operate despite an arbitrary number of messages being dropped (or delayed) by the network between nodes.

ACID vs BASE

https://medium.com/javarevisited/how-to-crack-system-design-interviews-in-2022-tips-questions-and-resources-fcad05e2dab

Load balancer, Caching.

[[230831]]