# ==TOC==
- Qns I had [[GenrSimAnneal#^dd1af0]]
- Traveling salesman library from whence the algorithm was copied [[GenrSimAnneal#^be161b]]
- Blog on the topic [[GenrSimAnneal#^84de39]]
- Why Scipy and R libraries don't work [[GenrSimAnneal#^1ab972]]
- Code experiments [[GenrSimAnneal#^dedbdf]]
# Qns ^dd1af0
- [ ] I don't understand why the temperature function makes this algorithm so much better than hill climbing. The R shiny app blog references this.

_____________
# Traveling salesman ^be161b
A great library for traveling salesman:
https://github.com/fillipe-gsm/python-tsp

See also this post on [traveling salesman](https://www.guru99.com/travelling-salesman-problem.html). This is where I took the traveling salesman code for my library.

keywords: library python traveling salesman trav salsmn dynamic programming dynamic_programming nphard

#building/blog  ^84de39
# Generalizing simulated annealing with continuous training
_____________
# Abstract
Simulated annealing is a powerful approach for solving comibatorial optimization problems, applicable across a wide swathe of use cases. The interfaces for existing software packages only work on problems where the inputs that are real arrays. Through our [new optimization library](https://github.com/ryu577/optimizn), we provide an interface for solving all kinds of combinatorial optimization problems with simulated annealing. Since we'll be interested in NP-hard problems where we'll never know if or when we reach an optimal solution, we introduced the concept of continuous training where we simply keep training on a regular cadence and retaining the salient aspects of previous runs to some persistent location like a disk or blob storage. 

This paper assumes basic knowledge of inheritance in Python.

# I) Introduction
Simulated annealing is a powerful optimization tool that can be used on pretty much any kind of optimization problem (we'll spell out the requirements in the next section). It is applicable to the miriad optimization problems we come across at work as data scientists for Microsofts cloud service (Azure). Especially in the context of combinatorial testing, designing experiments for testing new changes so customers are protected from any adverse effects. These problems are constrained and unconstrained combinatorial optimization problems from graph theory or extensions of other famous problems like bin-packing. Many of these problems are provably NP-hard and even if not, they are so unique that finding the best algorithms for solving each of them might not be scalable. Another characteristic about these problems tends to be their static nature. The problem statement, the parameters of the problem (like the graph for the graph optimization problems), etc. tend to stay the same for months on end. And the routines that solve them are run on a daily basis anyway. This is where the "continuous training" part comes in as we'll see in section IV.

Standard libraries in statistical languages like R or Python's Scipy have implementations of simulated annealing. However, looking deeper at the documentation and source code, these are applicable to problems where the inputs are arrays with continuous elements.

This is despite the documentation of the Scipy [anneal](https://docs.scipy.org/doc/scipy-0.14.0/reference/generated/scipy.optimize.anneal.html) method itself noting:
>"Simulated annealing is a random algorithm which uses no derivative information from the function being optimized. In practice it has been more useful in discrete optimization than continuous optimization, as there are usually better algorithms for continuous optimization problems."

The software packages I reviewed: [optim_sa in R](https://search.r-project.org/CRAN/refmans/optimization/html/optim_sa.html) and [anneal in Scipy (python)](https://docs.scipy.org/doc/scipy-0.14.0/reference/generated/scipy.optimize.anneal.html),  [dual_annealing in Scipy](https://github.com/scipy/scipy/blob/8a6f1a0621542f059a532953661cd43b8167fce0/scipy/optimize/_dual_annealing.py#L128) and  [Basin hopping](https://github.com/scipy/scipy/blob/890a1f2129313454e21280f5237ecb681f3127df/scipy/optimize/_basinhopping.py#L362) all apply to continuous optimization problems.


So, even a problem that is often used to demonstrate the effectiveness of simulated annealing, the traveling salesman problem can't be solved with the methods in these libraries since it is a combinatorial optimization problem with non-continuous inputs. 

Adding an additional layer of complexity, many of the problems we run into in our work are combinatorial constrained optimization problems, needing further customization to take the constraints into account.

This motivates the need for a library that can perform simulated annealing on all kinds of combinatorial optimization problems (constrained and un-constrained) which is where it really shines and better alternatives are often not available. Even if using the library is slightly more involved, with more responsibility lying with the user than traditional packages.

# II) When is it applicable?
For any optimization problem to be eligible for solving with simulated annealing, it only has to satisfy two requirements. You should be able to come up with an initial feasible (not optimal) solution to the problem. And you should be able to switch from any solution to another "neighboring" solution in a way that the switching strategy is capable of theoretically exploring the whole solution space if given enough time. This switching strategy is the most critical part of the algorithm and experimenting with different ways of doing this is valuable since it will have a big effect on the performance. In general, not changing the existing solution too much and even trying to greedily improve on the objective function where possible can be good strategies.

To use the library, you need to create a class in Python that represents your problem. The class should implement methods that correspond to the two primitives above, getting an initial solution and switching to a neighboring solution. Then, you make the class inherit the OptProblem class provided by the library and run the method "anneal" to optimize. We'll describe it in more detail in the next section.

# III) The code and contract
The idea of simulated annealing is very simple. It starts with some initial solution. It then considers a neighboring solution as a candidate to switch to. It switches to the candidate solution if that solution is better (has a better objective function). But even if it isn't, a coin is tossed and a switch is still made if we get a heads. This is the mechanism that helps the procedure escape local optima and hopefully find global ones. The switching to new solutions is simply repeated for a large number of iterations (typically pre-defined). 

Further, the probability with which this (switching even though the new solution is not better) happens is adjusted through the course of the iterations. Initially, this probability is high. As the iterations progress, it becomes lower in accordance with some function of the iteration count. This is the reason the procedure is called "annealing". We can consider the probability of switching to be a kind of "temperature". Then, the process of starting with a high temprature and decreasing it gradually is akin to the metallurgic process of annealing.

In order to use the interface, the user has to create a class representing the optimization problem and instantiate it. Since the logic for getting the cost associated with a candidate solution, getting a random solution and switching to a neighboring solution are specific to the particular problem being solved, these need to be implemented by the user who brings the problem. These three methods ("cost", "get_candidate" and "next_candidate") have to be implemented as methods of a class representing the problem.

Once the class and those three methods are ready, the user will have to make it inherit the class called OptProblem. Then, the "anneal" within that class would run the simulated annealing algorithm and store the best solution found over 100,000 iterations (by default). 

The basic skeleton code of the class that is to be inherited by the users object is shown below. The anneal procedure is the code that is implemented in this interface. 

In the next section, we'll show a minimal example of how to use this class on a toy problem. So far, this just involves the simulated annealing logic. The true power of the library comes from the "continuous learning" functionality, where the optimization happens everyday and never stops. We just use the best solution at any given point in time, as we need it. In many contexts, this is valuable since the problems tend to be very static, with their underlying parameters remaining unchanged over weeks and months. Moreover, the jobs that solve these problems run atleast once a day anyway. So, it makes sense to have those jobs remember their work from before and try to improve on it. To leverage the "continuous learning" part, some more requirements must be satisfied. In section V, we will review them.

```python
class OptProblem():
	def cost(self, candidate):
		''' Gets the cost for candidate solution.'''
		raise Exception("To be implemented by the child class")

	def get_candidate(self):
		''' Gets a feasible candidate.'''
		raise Exception("To be implemented by the child class")

	def next_candidate(self):
		''' Switch to the next candidate.'''
		raise Exception("To be implemented by the child class")

	def __init__(self):
		''' Initialize the problem '''
		self.candidate = self.get_candidate()
		self.current_cost = self.cost(self.candidate)
		self.best_cost = self.current_cost
		self.best_candidate = make_copy(self.candidate)

	def anneal(self, n_iter=100000):
		for i in range(n_iter):
			if i % 10000 == 0:
				print("Iteration: " + str(i) + \
					" Current best solution: " + str(self.best_cost))
			eps = 0.3 * e**(-i/n_iter)
			self.new_candidate = self.next_candidate()
			self.new_cost = self.cost(self.new_candidate)
			if self.new_cost < self.best_cost:
				self.update_best(self.new_candidate, self.new_cost)
				print("Best cost updated to:" + str(self.new_cost))
			if self.new_cost < self.current_cost or eps < uniform():
				self.update_candidate(self.new_candidate, 
									  self.new_cost)

	def update_candidate(self, candidate, cost):
		self.candidate = make_copy(candidate)
		self.current_cost = cost

	def update_best(self, candidate, cost):
		self.best_candidate = make_copy(candidate)
		self.best_cost = cost
```

# IV) Usage and sample problems
^ad6449

In this section, we'll demonstrate the interface on a sample problem, the traveling salesman.

The traveling salesman problem (TSP), often used to introduce simulated annealing to demonstrate its versatility. Given a set of cities and the distances between each of them, we have to come up with a tour that visits all cities in a way that the total distance travelled is minimized (for our salesman friend). This is known to be an NP-hard problem, meaning no polynomial time algorithm exists for solving it. If we try to solve this problem with the existing interfaces for simulated annealing, we're out of luck since its impossible to frame it as a problem in a continuous array. The solution space of the problem is best represented as an array of cities. The order in which the cities appear in the array is the order in which the tour visits them. If there are $n$ cities in our array, then the total number of possible tours is the number of permutations (we assume its possible to go from any city to any other city). And this is $n!$, too large to exhaustively enumerate even for modest $n$. 

To leverage simulated annealing, we have to come up with a strategy for an initial feasible solution and a strategy for switching solutions as mentioned in the last section. The initial solution is just a random permutation, so that should be simple enough. There can be various switching strategies that go from one tour to another. As a rule of thumb, we want the switch to take us not too far from the solution we start with. The idea being that good solutions will be in close proximity to other good solutions. 

There is an excellent [R implementation of TSP with simulated annealing](https://toddwschneider.com/posts/traveling-salesman-with-simulated-annealing-r-and-shiny/), the switching strategy is [swapping two cities](https://github.com/toddwschneider/shiny-salesman/blob/master/helpers.R). You can see that the standard [optim_sa](https://search.r-project.org/CRAN/refmans/optimization/html/optim_sa.html)method R provides was not used since it isn't applicable.

Next, let's see how to solve this problem with our library. The code below demonstrates this. First, we generate some random cities, calculate the distances between them and store in an object whose class is called "CityGraph" (all of this happens in the init method of that class).

Below that, the "TravSalsmn" class represents the problem itself (note that one can frame all kinds of problems on top of the city-graph). It implements the three methods: "cost", "get_candidate" and "next_candidate". 

- The "cost" method loops through all cities and sums the total distance between two successive ones.
- The "get_candidate" generates a random permutation of the cities (which are indexed by id's that start at $0$).
- The "next_candidate" takes two cities in the existing solution and swaps them.

```python
class CityGraph():
	def __init__(self):
		# Generate x-y coordinates of some cities.
		# Here, we just draw them from a normal dist.
		self.xs = np.random.normal(loc=0,scale=5,size=(50,2))
		self.num_cities = len(self.xs)
		self.dists = np.zeros((len(self.xs), len(self.xs)))
		# Populate the matrix of euclidean distances.
		for i in range(len(self.xs)):
			for j in range(i+1, len(self.xs)):
				dist = (self.xs[i][0]-self.xs[j][0])**2
				dist += (self.xs[i][1]-self.xs[j][1])**2
				dist = np.sqrt(dist)
				self.dists[i,j] = dist
		for i in range(len(self.xs)):
			for j in range(i):
				self.dists[i,j] = self.dists[j,i]


class TravSalsmn(OptProblem):
	"""
	Traveling salesman problem.
	"""
	def __init__(self, params):
		self.params = params
		self.name = "TravSalesman"
		super().__init__()

	def get_candidate(self):
		"""
		A candidate is going to be an array
		representing the order of cities
		visited.
		"""
		self.candidate = np.random.permutation(\
			np.arange(self.params.num_cities))
		return self.candidate

	def cost(self, candidate):
		tour_d = 0
		for i in range(1, len(candidate)):
			tour_d +=\
			 self.params.dists[candidate[i], candidate[i-1]]
		return tour_d

	def next_candidate(self, candidate):
		nu_candidate = deepcopy(candidate)
		swaps = np.random.choice(np.arange(len(candidate)),\
				size=2,replace=False)
		to_swap = nu_candidate[swaps]
		nu_candidate[swaps[0]] = to_swap[1]
		nu_candidate[swaps[1]] = to_swap[0]
		return nu_candidate
```

# V) Continuous training
Continuous training is an added, optional feature of the library. It might help to provide our own motivation for implementing it. We do think this might extend beyond our use cases to others who end up using this library as well.

Most of the problems we encounter in our work designing experiments for the cloud, in addition to being NP-hard, are also fairly static in nature. The underlying parameters that specify the problem instance (like graphs on which the problem is based) stay the same for weeks or months on end. And even when they eventually change, the changes are typically small (like the addition of some new hardware's).

Every day, we would use about 10 to 20 minutes of computing power for running jobs that run our optimization algorithm and produce the experiment design, even though the input parameters of the problem would stay the same for weeks or months. Now, 10 to 20 minutes of computing time was a trivial expense for us and it didn't make sense to invest in reducing it by only running when things changed. At the same time, it was also a waste of that computation to start from scratch every day and solve effectively the same problem that was solved yesterday most of the time. Since simulated annealing is an exploratory strategy, it'd be great to pick up where we left off or atleast take the best solution across the various runs where the underlying inputs to the algorithm were the same for the eventual use case (designing experiments). 

For NP-hard optimization problems, there is no good way to verify that a given solution is optimal. So, it doesn't make sense to ever stop looking for a better one. Eventually, the underlying inputs are going to change and the optimal solution is going to shift anyway.

This is why we added the continuous training functionality to our library. 

Note that we need to know when the inputs to the optimization problem haven't changed from the previous day and when they have. This tells us when we can build on the previous solutions and when we have to start from scratch (assuming for now we don't have a way to take the previous solution and translate to a solution for the new problem instance, taking into account how the parameters have changed). 

To fascilitate knowledge of when the underlying problem changes, we need to store the parameters of the problem somehow (for example, the graph of cities for the traveling salesman problem). This is best done as an object (instance of a class). The class can then implement an "equality" method that allows us to compare two instances of the class. This way, we can compare yesterday's object to today's object and detect when the underlying parameters of the problem have changed.

We persist three kinds of files in three different locations (these locations can be simple folders on a disk or blob storage locations if running from a cloud environment). The files in the different folders are named per the timestamps at which the method executes. This way, we can connect the objects in the different folders based on their names.

- The daily object: This is the underlying instance of the class that encodes the problem. We store the very first instance of this in a folder to begin with. From there, we only store instances when the underlying problem changes. 
- The daily optimum: We store in this folder every day (or every time the job runs). It is the best solution the simulated annealing run was able to find on that day. These come in handy since we sometimes use these as starting solutions to the runs on future days.
- The global optimum: This is the best solution we've found ever. We don't add a file to this folder everyday. We do so on the first day. And then, when we find a solution that beats the best. That way, we can always read the latest file and know that it is the best so far. We also have to write to it when the underlying inputs change.

This is demonstrated in the flowchart below.

![[ContinuousAnnealing.png]]

The mechanism that persists the objects is completely agnostic to the problem or its parameters. 

To recap, here are the requirements for using the various functionalities of the library.

For basic simulated annealing (the blue boxes in the figure below):
- Object instantiating the problem. This object should implement three methods.
	- get_cost: gets the cost associated with any candidate solution.
	- get_candidate: get a random, feasible candidate solution.
	- next_candidate: given some feasible candidate solution, switches to a neighboring solution.
For the continuous training functionality (see yellow boxes in figure below):
- There should be an object encoding problem parameters. It should be a property of the object encoding the problem (described above). The property in the latter should be named "params".
- The object encoding the parameters should have override methods for "eq" and "neq" inbuilt python methods. This is so that we can compare yesterday's parameters to today's parameters and see if the underlying problem has changed.
- Object encoding the problem should have a property called "name" which specifies the name of the problem. This is used to create a specific folder for the problem. This way, we can run multiple optimization problems from the same location.

The various components are shown in the figure below. The orange parts are implemented in the library. The blue parts need to be implemented by the user for leveraging the basic simulated annealing logic. The yellow parts are to be implemented to take advantage of the continuous training functionality.

![[SimAnnealGenContract.png]]

# VI) Some more problems
In this section, we visit some more combinatorial optimization problems that we then solve with our new simulated annealing interface. 
## VI-A) Bin packing
A classic NP-hard problem is bin packing. You are given some bins with a given volume and some objects, each of which has a given volume as well. Our task is to fit all the objects into as few bins as possible. The two basic primitives we need are coming up with an initial solution and coming up with a switching strategy. We can think of the various solutions as permutations of the items. Once we permute them, we can start fitting them into bins in order and stop once all the items are inside a bin. So like the traveling salesman problem, all we need to do is generate a random permutation for the former and a swap two random items for the latter.

## VI-B) Path cover
This problem came up in the context of designing experiments for the cloud. We are given a set of properties. For instance, hardware type, virtual machine (VM) type, OS image and so on for virtual machines running in the cloud. There is a graph representing the constraints that must be satisfied for a valid virtual machine. In particular, we have to select a hardware, a virtual machine type and an OS image to end up with a valid virtual machine. But, we can't just pick any combination of those three. The hardwares have some compatibility constraint with the virtual machines and the virtual machines have compatibility constraints with the OS-images. This is demonstrated in the figure below.

![[three_properties.png]]

We want to cover all hardware's, all VM-types and all OS-images in the smallest number of virtual machines such that the compatibility constraints encoded in the graph are satisfied. This can be thought of as a path cover problem since getting a valid combination of the three properties is like a path that starts all the way in the left-most layer (hardware) and ends in the right-most layer (OS-image). We want the smallest set of such paths that cover all the properties. Note that this isn't an NP-hard problem and we have an algorithm for the optimal solution.

The primitives we need to leverage simulated annealing are:
- Cost: This one is easy. We just count the number of paths in a solution.
- Get random solution: We go to each property and complete a path randomly (connecting to some two properties it is compatible with).
- Switch to new solution: We remove a few full path randomly (set to $4$). Then, we see how many properties are not covered as a result of this deletion. Then, we cover those properties again by completing random paths that include them at random.
In various simulated experiments, we always got to within $4$ of the optimal solution with this method within 500000 iterations of the simulated annealing.
# VII) Conclusion
Most optimization problems that exist are NP-hard. This can be said about problems arising in industry applications (like cloud computing) as well. This means there isn't even a good way to verify a given solution is optimal. Simulated annealing is a general purpose tool for solving many of these NP-hard optimization problems. We implemented a library that provides an interface fascilitating solving all of these problems. Further, we also provided a way to run the training continuously, across multiple runs and persist progress across the runs.


# References
[1] The generalized bin packing problem, Mauro Maria Baldi, Teodor Gabriel Crainic, Guido Perboli, Roberto Tadei [[gen_bin_pck_baldi.pdf]]

_________
Fin.

As of 221229, simulated annealing is implemented here: https://github.com/scipy/scipy/blob/8a6f1a0621542f059a532953661cd43b8167fce0/scipy/optimize/_dual_annealing.py

To invoke it in a way equivalent to original simulated_annealing: https://stats.stackexchange.com/questions/436052/simulated-annealing-vs-basin-hopping-algorithm

The EnergyState method has update_current, update_best methods.


There is now a take_step method.
https://docs.scipy.org/doc/scipy-0.15.1/reference/generated/scipy.optimize.basinhopping.html#scipy.optimize.basinhopping

Original anneal interface: https://docs.scipy.org/doc/scipy-0.14.0/reference/generated/scipy.optimize.anneal.html

#cs/complexity  #opt #paper #building/blog 
[[nphard]], [[blogs]]
keywords: nphard, np, complexity, cs
# Why Scipy and R libs don't work
^1ab972

The interface for simulated annealing [in R](https://search.r-project.org/CRAN/refmans/optimization/html/optim_sa.html) and [Python (Scipy)](https://docs.scipy.org/doc/scipy-0.14.0/reference/generated/scipy.optimize.anneal.html) involve passing a vector or 1-d array as input. 
	- ~~It is apparent that they are permuting the array to switch to different solutions.
		- This was actually not true. It was simply continuous inputs.
	- The switching strategy is very important in terms of the quality of solution.
- Copying solution from persistent storage should also be possible. This is because most problems are static in nature.

In Scipy, two routines available:

- [dual_annealing](https://github.com/scipy/scipy/blob/8a6f1a0621542f059a532953661cd43b8167fce0/scipy/optimize/_dual_annealing.py#L128)
	- Requires lower and upper bound arrays. Isn't clear how to get those arrays.
	- "In practice they have been used mainly in discrete rather than in continuous optimization."
- [Basin hopping](https://github.com/scipy/scipy/blob/890a1f2129313454e21280f5237ecb681f3127df/scipy/optimize/_basinhopping.py#L362)
	- Doesn't even pretend to apply to discrete space optimization problems. 
	- "Basin-hopping is a stochastic algorithm which attempts to find the global minimum of a smooth scalar function of one or more variables."


_________________
# Code experiments
^dedbdf

In the traveling salesman problem, a very good series of solutions is found shortly after switching to a random solution.

In [**66**]: tt.anneal(reset_p=1/100000)
Iteration: 0 Current best solution: 311.16501828535473
Iteration: 10000 Current best solution: 311.16501828535473
Iteration: 20000 Current best solution: 311.16501828535473
Iteration: 30000 Current best solution: 311.16501828535473
Switching to a completely random solution.
with cost: 431.5713937929245
Iteration: ==40000== Current best solution: 311.16501828535473
Best cost updated to:306.3728980626197
Best cost updated to:304.55385309578276
Best cost updated to:==302.7557039261393==
Iteration: 50000 Current best solution: 302.7557039261393
Switching to a completely random solution.
with cost: 430.48273943601924
Iteration: 60000 Current best solution: 302.7557039261393
Iteration: 70000 Current best solution: 302.7557039261393
Iteration: 80000 Current best solution: 302.7557039261393
Iteration: 90000 Current best solution: 302.7557039261393