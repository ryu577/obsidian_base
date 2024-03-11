%%%%%% MSJAR 2019 LATEX Template. Based on ICML 2019 submission template %%%%%%%

\documentclass{article}

% Recommended, but optional, packages for figures and better typesetting:
\usepackage{microtype}
\usepackage{graphicx}
\usepackage{subfigure}
\usepackage{booktabs} % for professional tables
\usepackage{amsmath} % for equations
%\usepackage[semicolon]{natbib}

% hyperref makes hyperlinks in the resulting PDF.
% If your build breaks (sometimes temporarily if a hyperlink spans a page)
% please comment out the following usepackage line and replace
% \usepackage{icml2019} with \usepackage[nohyperref]{icml2019} above.
\usepackage[hyphens]{url}
\usepackage{breakurl}
\usepackage[breaklinks]{hyperref}
\usepackage{listings}
\usepackage{tabularx}

\usepackage{xcolor}
\definecolor{codegreen}{rgb}{0,0.6,0}
\definecolor{codegray}{rgb}{0.5,0.5,0.5}
\definecolor{codepurple}{rgb}{0.58,0,0.82}
\definecolor{backcolour}{rgb}{0.95,0.95,0.92}
\lstset{
    showstringspaces=false,
    language=Python,
    breaklines=true,
    commentstyle=\color{codegreen},
    keywordstyle=\color{magenta},
    numberstyle=\tiny\color{codegray},
    stringstyle=\color{codepurple},
    basicstyle=\ttfamily\footnotesize
}

% Attempt to make hyperref and algorithmic work together better:
\newcommand{\theHalgorithm}{\arabic{algorithm}}

\usepackage[accepted]{icml2019}

\begin{document}

\twocolumn[
\title{Optimizn: a Python Library for Developing Customized Optimization Algorithms}
\date{\vspace{-0.2in}}
\maketitle

%//////////////////// Author information ////////////////////////
% It is OKAY to include author information, even for blind
% submissions: the style file will automatically remove it for you
% unless you've provided the [accepted] option to the icml2019
% package.

% Include the name, group and email of the author.
% Use the standard \icmlauthor{Name, Group, email@email.com}{}\\
% Dont remove the curly brackets {} at the end.

% Equal contribution.
% Adding "equal" to the second bracket will create an * meaning equal contribution.
% If you are using equal contribution read further instructions aftyer square bracket ].
% Affiliations will be numbered in order of appearance
\icmlsetsymbol{equal}{*}

\begin{icmlauthorlist}
\icmlauthor{Akshay Sathiya (Microsoft Azure)}{equal}
\icmlauthor{Rohit Pandey (Microsoft Azure)}{equal}
\end{icmlauthorlist}

\vspace{0.4in}
\begin{abstract}
Optimization problems are prevalent across a wide variety of domains. These problems are often nuanced, their optimal solutions might not be efficiently obtainable, and they may require lots of time and compute resources to solve. It follows that the best course of action for solving these problems is to use general optimization algorithm paradigms to quickly and easily develop optimization algorithms that are customized to these problems and can produce solutions of decent/sufficient optimality in a reasonable amount of time. In this paper, we present \texttt{optimizn}, a Python library for developing customized optimization algorithms under general optimization algorithm paradigms (simulated annealing, branch and bound). Additionally, \texttt{optimizn} offers continuous training, with which users can run their algorithms on a regular cadence, retain the salient aspects of previous runs, and use them in subsequent runs to potentially produce solutions of greater optimality. An earlier version of this paper was also published to the Microsoft-internal Machine Learning and Data Science (MLADS) conference and Microsoft Journal of Applied Research (MSJAR) in 2023.
\\

\textbf{Keywords:} combinatorial optimization, constrained optimization, optimization algorithms, simulated annealing, branch and bound, continuous training
\end{abstract}
\vspace{0.4in}
]

% This command creates the footnote for the equal contribution*
% If you need to mention equal contribution uncoment the next line
\printAffiliationsAndNotice{\icmlEqualContribution} % Uncomment for equal contribution
\clearpage


\section{Introduction}
There are a myriad of optimization problems that arise in cloud computing at Microsoft Azure and in virtually any other industry or line of work.

One such problem encountered at Azure is the environment design problem, where environments are designed for testing internal programs and catching issues before deployment, to protect customer workloads \cite{AzQEnvDesign}. Another problem encountered in Azure is allocating virtual machines to servers with the desired configurations and resources, while using the least number of servers (leaving more for future use). 

Many optimization problems are $NP$-hard, so their optimal solutions are unlikely to be obtainable in polynomial time \cite{NPHard1}. In such cases, solutions to these problems could be determined manually and hardcoded, determined randomly, or determined with polynomial-time approximation algorithms (an approximation algorithm produces a solution whose optimality is bounded by some numeric multiple of the optimality of the optimal solution \cite{ApproxAlg}). However, if those solutions are not sufficiently optimal, it could have adverse outcomes. In the case of the virtual machine allocation problem in Azure, an insufficiently optimal solution wastes resources (the virtual machines are allocated on more servers than required). This could make it harder for some customers/users of Azure to get the compute resources they need. Additionally, in the case of the environment design problem, an insufficiently optimal set of testing configurations could prevent certain regressions/errors/bugs in internal programs from being caught in pre-production testing, which means that they could pop up in production and negatively impact customers \cite{AzQEnvDesign}.

The big idea is that a way of quickly getting sufficiently optimal solutions to these problems will help Azure be more efficient and robust, which will ultimately provide a better experience for Azure users/customers.

This paper will present a code library, created by the authors, that can be used to develop optimization algorithms that are customized to specific optimization problems and can quickly produce sufficiently optimal solutions to those problems.

This code library can help solve a variety of optimization problems in practice. It has already been used to solve the environment design problem in Azure. The environment design problem and the solution approach (which uses the code library) are presented and discussed in greater detail in other literature, written by the same authors of this paper \cite{AzQEnvDesign}.

\subsection{Solution Optimality}

As stated earlier, it is unlikely that the optimal solutions for $NP$-hard optimization problems can be obtained in polynomial time \cite{NPHard1}. In such cases, where the optimization problems in consideration are $NP$-hard, it is likely that trying to develop polynomial-time algorithms to obtain the optimal solutions is a fruitless endeavor. 

Even in cases where the optimization problems in consideration are not $NP$-hard and the optimal solutions can be obtained in polynomial time, the problems themselves are often very nuanced. In such cases, it could be very time-consuming and impractical to (for every problem in consideration) develop from scratch a problem-specific algorithm that obtains the optimal solution (especially if there are many such optimization problems to solve in a particular setting).

In practice, optimal solutions are not always required and solutions of decent optimality (``good" solutions) would suffice, and are preferable, if they can be obtained relatively quickly.

\subsection{Static Problems}

The context and parameters of these optimization problems might not change much from day to day. For instance, in the environment design problem, the compute resources (hardware models, virtual machine types, etc.) available in the Azure fleet and their compatibility relationships do not change often, meaning that the environment design problem itself is relatively static \cite{AzQEnvDesign}. 

If an optimization algorithm that solves a relatively static optimization problem requires lots of compute time and resources, then it would be convenient if the algorithm could be run across discontinuous intervals of time, as compute time and resources are available. If the problem context and parameters have not changed from the previous run(s), the next run can start from where the previous run(s) left off instead of starting ground zero. This allows the algorithm to build on the progress made by the previous run(s) and make the most of the available compute time and resources to find the most optimal solution, even if the compute time and resources are not available in one, large continuous block of time. 

\subsection{Optimization Library}

This calls for a general method of developing optimization algorithms that can quickly give good/decently optimal solutions. Additionally, these algorithms should be able to run across discontinuous intervals of time to make the most of available compute time and resources and obtain the most optimal solution possible under those limitations.

This paper presents the answer to that call, a Python library called \texttt{optimizn} \cite{Optimizn}. The current offerings of \texttt{optimizn} are the following.

\begin{itemize}
  \item Simulated annealing
  \item Branch and bound
  \item Continuous training
\end{itemize}

Simulated annealing and branch and bound are both general optimization algorithm paradigms that can be customized to solve specific optimization algorithms. At their core, these algorithms consider several different solutions in the solution space of the problem they are solving, and keep track of the most optimal solution they find. 

Continuous training allows both these algorithms to run, save their state (problem parameters, most optimal solution observed in latest run, and most optimal solution observed over all runs under same problem parameters), and resume running from that state later. This allows the algorithms to find the most optimal solutions possible in situations where compute resources and time are not readily available/are only available in disjoint intervals.

This library can help solve several optimization problems inside and outside of Azure.

\section{Background Information}

\subsection{$P/NP$ Complexity Theory}

In computer science, problems are classified based on how ``hard" they are to solve through $P/NP$ complexity theory.

Under $P/NP$ complexity theory, computers are represented as either deterministic or non-deterministic Turing machines \cite{NPHard1}. Deterministic Turing machines follow one sequence of execution to obtain a solution to a problem, and non-deterministic Turing machines follow multiple sequences simultaneously to obtain a solution to a problem (essentially, the sequence leading to the solution is always chosen) \cite{NPHard1}. Between the two types of Turing machines, most modern computers (at the time of this writing) are closer to deterministic Turing machines since they follow a single sequence of execution to arrive at a solution, rather than exploring multiple sequences of execution at the same time. Hence, this paper considers real-world computers as deterministic Turing machines.

Problems are placed in complexity classes based on how long it would take for a particular type of Turing machine to solve it (their ``hard"-ness). The complexity classes are as follows \cite{NPHard1}.

\begin{itemize}
  \item $P$: decision problems (for which the answer is yes/no) that are solvable in polynomial time by a deterministic Turing machine.
  \item $NP$: decision problems that are solvable in polynomial time by a non-deterministic Turing machine. Includes problems in $P$, $P \subseteq NP$. Problems in $NP$ are at least as hard as the problems in $P$.
  \item $NP$-Complete ($NPC$): decision problems in $NP$ that any problem in $NP$ ($NPC \subseteq NP$) can be reduced to (the inputs of problem $p_1 \in NP$ can be converted, in polynomial time, to the inputs of problem $p_2 \in NPC$, and the answer to $p_2$ is the answer to $p_1$). $NP$-complete problems are at least as hard as any other problem in $NP$.
  \item $NP$-Hard ($NPH$): problems (not only decision problems) that may or may not be solvable in polynomial time by a non-deterministic Turing machine ($NPC \subset NPH$). $NP$-hard problems are at least as hard as the $NP$-complete problems.
\end{itemize}

If $P = NP$, then $NPC \in P$ and the least hard $NP$-hard problems (the $NP$-complete problems) are solvable in polynomial time by a deterministic Turing machine. Additionally, if there is overlap between the problems in $P$ and the $NP$-complete problems, $NPC \cap P \neq \emptyset$, then $P = NP$, since all problems in $NP$ can be reduced to the $NP$-complete problems that are also in $P$, so all $NP$ problems can be solved by a deterministic Turing machine in polynomial time. 

Since it has not yet been proven that $P = NP$ \cite{NPHard1}, there is likely no overlap between the $NP$-complete problems and the problems in $P$ ($NPC \cap P = \emptyset$), so it is very likely that $NP$-hard problems are not solvable in polynomial time by deterministic Turing machines. Thus, $NP$-hard problems are very unlikely to be solvable in polynomial time by most modern computers (at the time of this writing). It is very likely that solving $NP$-hard problems requires exponential time or worse \cite{NPHard1}.

It follows that the optimal solutions to $NP$-hard optimization problems cannot be obtained ``efficiently" (in polynomial time \cite{ExpDesign}). Hence, the need arises in practice for optimization algorithms that can get good solutions of sufficient optimality in limited amounts of time.

\subsection{Simulated Annealing}

One general optimization algorithm paradigm is simulated annealing. The term ``simulated annealing" is derived from the process of physical annealing, where a material is heated and then cooled over time so it can be altered to fit a desired structure \cite{SimulatedAnnealing1}. In simulated annealing, the material to be altered is an initial solution, the heating and cooling over time is a series of modifications made to the initial solution (each modification producing a new and potentially more optimal solution), and the desired structure of the material is the optimal solution to the optimization problem. The most optimal solution seen across the series of modifications is the result of the simulated annealing algorithm.

The \texttt{optimizn} library's implementation of simulated annealing is based on the simulated annealing implementation for the traveling salesman problem in \textit{The Traveling Salesman with Simulated Annealing, R, and Shiny} \cite{SimulatedAnnealing2, SimulatedAnnealing3}. 

The customizable components of simulated annealing are as follows \cite{SimulatedAnnealing1, SimulatedAnnealing2, SimulatedAnnealing3}. 
\begin{itemize}
  \item An initial solution, or a means of obtaining one given the optimization problem parameters (through a polynomial-time approximation algorithm, for instance). 
  \item A method of generating a new solution from a current solution (a neighboring solution). A neighboring solution can be generated by partially changing the current solution in some way.
  \item A number of iterations, to dictate the number of solution switches the algorithm considers before termination.
  \item An objective function, which accepts a solution as input and provides a cost value as output. It is assumed that the optimization problem is a minimization problem, so lower cost values correspond to more optimal solutions. In the event that the optimization problem is a maximization problem, the cost value can simply be negated \cite{MinToMax}.
\end{itemize}

The \texttt{optimizn} implementation features two additional components, the reset probability and the reset solution method.
\begin{itemize}
  \item A reset probability, under which the algorithm will continue from a new solution (that is not related to/dependent on the current solution) by setting the current solution to this new solution (resetting the current solution). This new solution is generated by the following method.
  \item A method of producing a new solution that is not related to/dependent on the current solution, for when the algorithm resets the current solution under the reset probability.
\end{itemize}

These two additional components help the simulated annealing algorithm visit different parts of the solution space, which could yield solutions that are more optimal than those already seen.

Given these components, customized to a specific optimization problem, the \texttt{optimizn} library's simulated annealing algorithm works like so. 

The algorithm starts at the initial solution (using the initial solution provided, or generating it through the means provided). This initial solution becomes the current solution.

For the specified number of iterations, a neighboring solution is generated from the current solution. 

However, under the reset probability, the current solution will instead switch to a new solution, which is generated by the reset solution method. Otherwise, the neighboring solution is considered.

If the cost of the neighboring solution is less than the cost of the current solution (under the objective function), then the neighboring solution becomes the new current solution. 

Otherwise, the neighboring solution becomes the new current solution under the probability $\epsilon$, given by the following equation, where $c_n$ is the cost of the neighboring solution and $c_c$ is the cost of the current solution.

$$\epsilon = e^{(c_n - c_c)/\text{temperature}}$$

The temperature represents an exploration-exploitation tradeoff \cite{SimulatedAnnealing1}. Higher temperature values prioritize exploration, where the algorithm has a a higher probability of switching to suboptimal solutions in hopes of finding more optimal solutions that are reachable through those suboptimal solutions (open to other parts of the solution space). Lower temperature values prioritize exploitation, where the algorithm has a lower probability of switching to suboptimal solutions, and staying in the area of the current solution (staying in the same part of the solution space).

The temperature value decreases over the iterations. In the \texttt{optimizn} implementation of simulated annealing, the temperature is determined by the following function, based on the Sigmoid function \cite{Sigmoid}, where $a$ is the amplitude, $c$ is the center, and $w$ is the width of the modified Sigmoid function \cite{SimulatedAnnealing3}. In the \textit{The Traveling Salesman with Simulated Annealing, R, and Shiny} implementation, $x$ is the total number of iterations \cite{SimulatedAnnealing3}, but in the \texttt{optimizn} implementation, $x$ is the number of iterations since the last time the current solution was reset. In the \texttt{optimizn} implementation, $a = 4000$, $c = 0$, $w = 3000$ by default. These default values are used in the \textit{The Traveling Salesman with Simulated Annealing, R, and Shiny} implementation and has shown good results for their use case/demo \cite{SimulatedAnnealing2, SimulatedAnnealing3}, so they will be used in the \texttt{optimizn} simulated annealing implementation as well.

$$S(x, a, c, w) = \frac{a}{1 + e^{(x - c)/w}}$$

This process repeats for the desired number of iterations or until a desired amount of time has elapsed (whichever comes first). As the iterations progress, the algorithm keeps track of the most optimal solution observed and the cost of that solution under the objective function. These are the final results of the simulated annealing algorithm.

\subsection{Branch and Bound}

Another general optimization algorithm paradigm is branch and bound. Branch and bound represents the solution space as a tree structure (where subtrees represent constrained solution spaces and constrained versions of the original optimization problem) and searches this solution tree structure for the optimal solution, keeping track of the most optimal solution observed and pruning out subtrees from the search if they will not yield a solution more optimal than the most optimal solution observed in the search so far \cite{BranchAndBound1}.

The \texttt{optimizn} library's branch and bound implementation is based on the pseudocode and explanation of eager branch and bound presented in \textit{Branch and Bound Algorithms - Principles and Examples.} \cite{BranchAndBound1} and the demonstration of branch and bound shown in \textit{7.2 0/1 Knapsack using Branch and Bound} \cite{BranchAndBound2}. 

The customizable components of branch and bound are as follows \cite{BranchAndBound1, BranchAndBound2}.
\begin{itemize}
  \item An initial solution, or a means of obtaining one given the optimization problem parameters (e.g. a polynomial-time approximation algorithm). This initial solution corresponds to the entire solution space, with no additional constraints on the original optimization problem. This initial solution helps the branch and bound algorithm prune out suboptimal subtrees from the start (the pruned subtrees would not yield a solution more optimal than the initial solution).
  \item A method of generating the root node solution of the solution tree structure. The solution tree structure is grown from this root node solution using the following method. 
  \item A method of generating other solutions from a current solution (branching), to grow the solution tree structure. Each of the other solutions retains the properties of the current solution, but also has additional properties that make it correspond to a more constrained solution space and a more constrained version of the original optimization problem.
  \item A method of determining if a solution is complete or incomplete/partial. A complete solution corresponds to a fully constrained solution space and fully constrained version of the optimization problem, for which it is the only solution. Complete solutions solve the original problem (adhering to all of its constraints) and cannot be built/expanded further through branching, since the corresponding constrained solution space and constrained version of the optimization problem cannot be constrained further. Since complete solutions cannot be further built/expanded by branching, it follows that complete solutions are represented in the solution tree structure by the leaf nodes. Incomplete/partial solutions do not solve the problem in their current form and must be built/expanded further through branching to yield any complete solutions to the problem.   
  \item A method of completing an incomplete solution (e.g. a polynomial-time approximation algorithm). In the solution tree structure, the leaf nodes are complete solutions. In some situations, reaching the leaf nodes takes a very long time, and some sufficiently optimal solutions can be found before a leaf node is reached by completing a partial solution using this method.
  \item An objective function, which accepts a solution as input and provides a cost value as output. It is assumed that the optimization problem is a minimization problem, so lower cost values correspond to more optimal solutions. In the event that the optimization problem is a maximization problem, the cost value can simply be negated \cite{MinToMax}.
  \item A lower bound function, which accepts a solution as input and provides the lowest cost value attainable from the solution itself as well as all other solutions in its solution space/subtree (the solution's lower bound). Used to prune subtrees from the search for the optimal solution if they do not contain a solution more optimal than the most optimal solution seen so far in the search. 
\end{itemize}

The \texttt{optimizn} implementation features an additional component, a method to check if a solution is feasible or infeasible. 

\begin{itemize}
  \item A method of determining if a solution is feasible or infeasible. Feasible solutions are within the constraints of the original problem and either are, or may yield through branching, complete solutions. Infeasible solutions break/violate the constraints of the original problem, so they are not complete solutions, nor can they yield any complete solutions through branching.
  
  Note that this concept of a feasible solution, presented in this paper and used in the \texttt{optimizn} library, differs from the concept of a feasible solution in \textit{Branch and Bound Algorithms - Principles and Examples.}, where feasible solutions are solutions that solve the original problem (adhering to all its constraints) and are represented by leaf nodes of the solution tree structure \cite{BranchAndBound1} (essentially what was described earlier as complete solutions).
\end{itemize}

This additional component ensures that infeasible solutions produced by the branching method are not considered in the branch and bound algorithm.

Given these components, customized to a specific optimization problem, the \texttt{optimizn} library's branch and bound algorithm works like so.

The algorithm starts at the initial solution (using the initial solution provided, or generating it through the means provided). At the start of the algorithm, this initial solution is considered the be most optimal solution observed so far. This helps the algorithm prune out suboptimal subtrees (which will not yield a solution more optimal than this initial solution) early on. This ensures that the algorithm produces a solution that is at least as optimal as the initial solution (the algorithm can return the initial solution or a solution that is more optimal than the initial solution).

Then, the algorithm generates the root node solution and adds it to a priority queue, with its priority being its lower bound. 

While the queue is not empty (or until a specified number of iterations has been reached or until a certain amount of time has elapsed), the algorithm iteratively takes the solution with the lowest lower bound off the priority queue and processes it by doing the following.

The algorithm compares the lower bound of the current solution to the cost of the most optimal solution seen so far. If this lower bound is not less than the cost of the most optimal solution seen so far, then the current solution itself as well as all others in its solution space (the solutions obtained from branching on the current solution, and branching on those solutions, and so on) will not have lower cost than the most optimal solution seen so far. The algorithm will not get a more optimal solution from searching in that part of the solution space, so it will no longer consider this solution, and the others in its solution space, and will move on to the next solution in the priority queue.

If the lower bound is less than the cost of the most optimal solution seen so far, then it could yield a solution more optimal than the solutions seen so far. The algorithm branches this solution, and does the following for each of the new solutions.

If the new solution is infeasible, then it is discarded and the next new solution is considered. 

If the new solution is feasible and complete, then the cost is computed for it and compared to the cost of the most optimal solution seen so far. If the cost of the new solution is less than the cost of the most optimal solution seen so far, then the most optimal solution and its cost are updated to the new solution and its cost, respectively. 

If the new solution is feasible and incomplete/partial, then it and its lower bound are added to the priority queue. Additionally, the new solution could be completed in some way (e.g. a polynomial-time approximation algorithm) and the completed new solution can be checked against the most optimal solution. If the cost of the completed new solution is less than the cost of the most optimal solution, then the most optimal solution and its cost can be updated to the completed new solution and the cost of the completed new solution, respectively. The completed new solution is not added to the priority queue.

The advantage of doing this is that good/decently optimal solutions can be found earlier in the algorithm, in case it takes a long time for the algorithm to reach the leaf nodes of the solution space tree, which correspond to complete solutions. The disadvantage of doing this is that the algorithm does not perform as many iterations under a given amount of time, since completing a partial solution adds time to each iteration. It follows that the algorithm would be less likely to reach the leaf nodes under a given amount of time.

Let's refer to the branch and bound algorithm without the completion of feasible, partial solutions as traditional branch and bound and let's refer to the approach where the feasible, partial solutions are completed and evaluated against the most optimal solution as the modified branch and bound algorithm. This completion and evaluation of feasible, partial solutions that characterizes the modified branch and bound algorithm is based on the demonstration of branch and bound shown in \textit{7.2 0/1 Knapsack using Branch and Bound} \cite{BranchAndBound2}.

The process repeats until the priority queue is empty, the desired number of iterations has been reached, or the desired amount of time has elapsed (whichever comes first). The most optimal solution and its cost are kept track of as the algorithm processes solutions. These are the final results of the branch and bound algorithm.

\section{Related Works}

\subsection{Simulated Annealing}

The following software packages for simulated annealing have been reviewed: \texttt{optim\_sa} in R \cite{OptimSAR}, \texttt{anneal} in SciPy \cite{SciPyAnneal}, \texttt{dual\_annealing} in SciPy \cite{SciPyDualAnnealing}, and \texttt{basinhopping} in SciPy \cite{SciPyBasinHopping}.

Libraries in statistical languages like R or Python's Scipy have implementations of simulated annealing. However, looking deeper at the documentation and source code, these are applicable to problems where the inputs
are arrays with continuous elements.

This is despite the documentation of the Scipy anneal method itself noting: ``Simulated annealing is a random algorithm which uses no derivative information from the function being optimized. In practice it has been more useful in discrete optimization than continuous optimization, as there are usually better algorithms for continuous optimization problems" \cite{SciPyAnneal}.

So, even a problem that is often used to demonstrate the effectiveness of simulated annealing, the traveling salesman problem, can't be solved with the methods in these libraries since it is a combinatorial optimization problem with non-continuous inputs.

Adding an additional layer of complexity, many optimization problems that are present in practice are combinatorial constrained optimization problems, needing further customization to take the constraints into account.

This motivates the need for a library that can perform simulated annealing on all kinds of combinatorial optimization problems (constrained and un-constrained), which is where it really shines and better alternatives are often not available, even if using the library is slightly more involved, with more responsibility lying with the user than traditional packages.

Hence, the \texttt{optimizn} library offers simulated annealing, where the core components discussed in the previous section can be implemented by the user for their specific optimization problem, regardless of the nature of their inputs (discrete/continuous).

\subsection{Branch and Bound}

The following software packages for branch and bound have been reviewed: \texttt{PuLP} \cite{PuLP1, PuLP2}, \texttt{PyBnB} \cite{PyBnB}.

The \texttt{PuLP} package uses branch and bound to solve constrained optimization problems that can be expressed in an integer programming context \cite{PuLP1}. However, there may be some optimization problems that are easier to express in other ways. This motivates the need for a library with a more general approach to branch and bound, even if using the library is slightly more involved.

The \texttt{PyBnB} package offers a general branch and bound implementation, where the user implements certain functions, like \texttt{branch}, \texttt{objective}, \texttt{bound}, and others, based on the specifics of their optimization problem \cite{PyBnB}. Additionally, \texttt{PyBnB} gives the user the chance to specify how their problem state can be saved and loaded so execution of their branch and bound algorithm can be resumed from where a previous execution left off (this is essentially continuous training). 

Even though \texttt{PyBnB} addresses the same needs as \texttt{optimizn}, it only offers support for branch and bound, and does not offer support for other general optimization algorithm paradigms, like simulated annealing.

Hence, \texttt{optimizn} also contains a general branch and bound implementation, in addition to simulated annealing, with continuous training for both. This is done in the spirit of providing support for a wider variety of general optimization algorithm paradigms under a single library. This gives users more freedom when developing algorithms to solve their optimization problems.

\section{Code and Contracts}

There are three main classes in the \texttt{optimizn} library, which are listed below.

\begin{itemize}
  \item \texttt{OptProblem}: the class for all optimization problems. The user must implement/override certain methods in this class as needed for their specific optimization problem. Contains the logic for continuous training. 
  \item \texttt{SimAnnealProblem}: the class for optimization problems solved using simulated annealing, extends the \texttt{OptProblem} class. The user must create a class for their specific optimization problem that extends this class and implement/override certain methods as needed for the specific optimization problem. Contains the logic for running simulated annealing in general, using the methods customized by the user.
  \item \texttt{BnBProblem}: the class for optimization problems solved using branch and bound, extends the \texttt{OptProblem} class. The user must create a class for their specific optimization problem that extends this class and implement/override certain methods as needed for the specific optimization problem. Contains the logic for running branch and bound in general, using the methods customized by the user. The user can also specify whether they would like to run traditional branch and bound or modified branch and bound.
\end{itemize} 

To implement an optimization algorithm, the user creates a class for their optimization problem (referred to as the ``optimization problem class") which extends either \texttt{SimAnnealProblem} or \texttt{BnBProblem} class, based on which optimization algorithm they want to implement (simulated annealing or branch and bound, respectively). 

Both the \texttt{SimAnnealProblem} and \texttt{BnBProblem} classes extend the \texttt{OptProblem} class. The optimization problem class inherits the method that executes the general optimization algorithm (simulated annealing or branch and bound), but the user must implement other inherited methods (as needed for their specific optimization problem) that are used in the execution of the general optimization algorithm.

To run their optimization algorithm, the user can simply instantiate their optimization problem class and call the solver method inherited from the \texttt{SimAnnealProblem} or \texttt{BnBProblem} class. The problem parameters and instance of the optimization problem class (which contains the most optimal solution found and the cost of that solution) can be saved by continuous training for future use/reference.

The code and contract explanations for the \texttt{optimizn} library's simulated annealing, branch and bound, and continuous training offerings are shown in the following subsections.

\subsection{Optimization Problem Class with Continuous Training}

The code for the \texttt{OptProblem} class and continuous training are shown below. 

\begin{lstlisting}
# Copyright (c) Microsoft Corporation.
# Licensed under the MIT License.

import pickle
import os
from datetime import datetime


class OptProblem():
  def __init__(self):
    ''' Initialize the problem '''
    self.init_time = datetime.now()
    self.init_secs = int(self.init_time.timestamp())
    self.best_solution = self.get_candidate()
    self.best_cost = self.cost(self.best_solution)
    print(f'Initial solution: {self.best_solution}')
    print(f'Initial solution cost: {self.best_cost}')
    self.name = self.__class__.__name__
    if not hasattr(self, 'params'):
      raise Exception('All problem class instances must have a "params" attribute, which is an object that contains the input parameters to the problem class')

  def get_candidate(self):
    ''' Gets a feasible candidate.'''
    raise Exception("Not implemented")

  def cost(self, sol):
    ''' Gets the cost for candidate solution.'''
    raise Exception("Not implemented")

  def cost_delta(self, cost1, cost2):
    return cost1 - cost2

  def persist(self):
    create_folders(self.name)
    existing_obj = load_latest_pckl("Data//" + self.name + "//DailyObj")
    self.obj_changed = (existing_obj != self.params)
    if self.obj_changed or existing_obj is None:
      # Write the latest input object that has changed.
      f_name = "Data//" + self.name + "//DailyObj//" + str(self.init_secs) + ".obj"
      file1 = open(f_name, 'wb')
      pickle.dump(self.params, file1)
      print("Wrote to DailyObj")
    # Write the optimization object.
    f_name = "Data//" + self.name + "//DailyOpt//" + str(self.init_secs) + ".obj"
    file1 = open(f_name, 'wb')
    pickle.dump(self, file1)
    print("Wrote to DailyOpt")

    # Now check if the current best is better than the global best
    existing_best = load_latest_pckl("Data//" + self.name + "//GlobalOpt")
    if existing_best is None or self.best_cost > existing_best.best_cost or self.obj_changed:
      f_name = "Data//" + self.name + "//GlobalOpt//" + str(self.init_secs) + ".obj"
      file1 = open(f_name, 'wb')
      pickle.dump(self, file1)
      print("Wrote to GlobalOpt")


def create_folders(name):
  if not os.path.exists("Data//"):
    os.mkdir("Data//")
  if not os.path.exists("Data//" + name + "//"):
    os.mkdir("Data//" + name + "//")
  if not os.path.exists("Data//" + name + "//DailyObj//"):
    os.mkdir("Data//" + name + "//DailyObj//")
  if not os.path.exists("Data//" + name + "//DailyOpt//"):
    os.mkdir("Data//" + name + "//DailyOpt//")
  if not os.path.exists("Data//" + name + "//GlobalOpt//"):
    os.mkdir("Data//" + name + "//GlobalOpt//")


def load_latest_pckl(path1="Data/DailyObj"):
  if not os.path.exists(path1):
    return None
  msh_files = os.listdir(path1)
  msh_files = [i for i in msh_files if not i.startswith('.')]
  msh_files = sorted(msh_files)
  if len(msh_files) > 0:
    latest_file = msh_files[len(msh_files)-1]
    filepath = path1 + "//" + latest_file
    if os.path.getsize(filepath) == 0:
      print('File located at', filepath, 'is empty')
    else:
      filehandler = open(filepath, 'rb')
      existing_obj = pickle.load(filehandler)
      return existing_obj
  return None
\end{lstlisting}

The \texttt{OptProblem} class contains logic required for all optimization problems and continuous training.

The user is required to implement the following methods (which are inherited from the \texttt{OptProblem} class and correspond to customizable components of both simulated annealing and branch and bound) in their optimization problem class.
\begin{itemize}
\item \texttt{get\_candidate}: the method of producing and returning the initial solution.
\item \texttt{cost}: the method of computing the value of the objective function (cost) for a given solution.
\item \texttt{cost\_delta}: the method of computing the difference between two cost values. Defaults to the difference of the first and second cost values. If the difference is positive, the first cost value is greater, if the difference is negative, the second cost value is greater, and if the difference is zero, then they are equal. This method can be overridden by the user, if needed.
\item \texttt{persist}: the method of saving the optimization problem parameters and optimization problem class instance (with the problem state, most optimal solution found, and cost of the most optimal solution found). This method is already implemented by default, but can be overridden by the user, if needed.
\end{itemize}

The \texttt{OptProblem} class contains the continuous training logic in the \texttt{persist} method, which is called at the end of the solver functions for the \texttt{SimAnnealProblem} or \texttt{BnBProblem} classes. In order to use continuous training in their optimization algorithms, the user must set the \texttt{params} attribute of their optimization problem class instance to a dictionary, an instance of a custom parameters class, or any other object that holds the parameters of the problem. Additionally, they must call the \texttt{persist} method to save the optimization problem parameters and the instance of the optimization problem class that is solving that optimization problem.

The \texttt{persist} method performs continuous training like so. First, it creates (if it does not already exist) a directory called \texttt{Data}, and it creates (if it does not already exist) a subdirectory for the specific optimization problem class (which extends either the \texttt{SimAnnealProblem} or \texttt{BnBProblem} class). The name of the subdirectory is the name of the problem class itself (e.g. \texttt{TravelingSalesmanProblem}). Then, three subdirectories are created (if they do not already exist) in the optimization problem class subdirectory. These three directories are called \texttt{DailyObj}, \texttt{DailyOpt}, and \texttt{GlobalOpt}.

The \texttt{DailyObj} directory contains the problem parameters (the value of the \texttt{params} attribute in the optimization problem class) for the most recent run of the optimization algorithm. After a run of the optimization algorithm, the problem parameters are saved into the \texttt{DailyObj} directory if it is empty (no previous runs) or if the problem parameters are different than those of the most recent previous run of the optimization algorithm. 

The \texttt{DailyOpt} directory contains the instance of the optimization problem class for the most recent run of the optimization algorithm. After a run of the optimization algorithm, the instance of the optimization problem class (which contains, as attributes, the most optimal solution it found and the cost of that solution) is saved to the \texttt{DailyOpt} directory.

The \texttt{GlobalOpt} directory contains the instance of the optimization problem class with the most optimal solution observed across all runs of the optimization algorithm. If the problem parameters of the current run of the optimization algorithm are different than those of the most recent previous run, or if there was no previous run, or if the most optimal solution found by the current run is more optimal than the most optimal solution found across all previous runs (under the same problem parameters), then the instance of the optimization problem class is saved to the \texttt{GlobalOpt} folder as well.

To continue running an optimization algorithm from where it left off, the current parameters of the optimization problem can be compared to the most recently saved parameters, which can be loaded from the \texttt{DailyObj} directory. If the parameters are equal, then the optimization problem has not changed since the most recent run of the optimization algorithm, so the instance of the corresponding optimization problem class can be loaded from either the \texttt{DailyOpt} or the \texttt{GlobalOpt} directories and ran again. The result will be a solution of equal or greater optimality than the most optimal solution observed in the previous run(s) of the optimization algorithm. The most recently saved optimization problem parameters and optimization problem class instances can be loaded using the \texttt{load\_latest\_pckl} function.

The continuous training logic is implemented for saving/loading optimization algorithms locally (using the \texttt{pickle} module \cite{pickle}) by default, but can be overridden by the user if they want to save/load data to/from other sources.

\subsection{Simulated Annealing Class}

The code for the \texttt{SimAnnealProblem} class and simulated annealing, based on the simulated annealing implementation for the traveling salesman problem in \textit{The Traveling Salesman with Simulated Annealing, R, and Shiny} \cite{SimulatedAnnealing2, SimulatedAnnealing3}, is shown below. The code of the \textit{The Traveling Salesman with Simulated Annealing, R, and Shiny} simulated annealing implementation is licensed under the MIT License \cite{SimulatedAnnealing4}. The \texttt{optimizn} library is also licensed under the MIT License \cite{Optimizn2}, and the \texttt{optimizn} repository contains the original license text for the \textit{The Traveling Salesman with Simulated Annealing, R, and Shiny} simulated annealing code \cite{Optimizn3}. 

\begin{lstlisting}
# Copyright (c) Microsoft Corporation.
# Licensed under the MIT License.

from numpy.random import uniform
import numpy as np
from copy import deepcopy
from optimizn.combinatorial.opt_problem import OptProblem
import time


class SimAnnealProblem(OptProblem):
  def __init__(self):
    ''' Initialize the problem '''
    super().__init__()
    self.candidate = make_copy(self.best_solution)
    self.current_cost = self.best_cost

  def next_candidate(self):
    ''' Switch to the next candidate.'''
    raise Exception("Not implemented")

  def reset_candidate(self):
    '''
    Returns a new solution for when the candidate solution is reset.
    Defaults to get_candidate but can be overridden if needed
    '''
    return self.get_candidate()

  def anneal(self, n_iter=100000, reset_p=1/10000, time_limit=3600):
    '''
    This simulated annealing implementation is based on the code and explanation of simulated annealing presented in the following sources.

    One key difference between this simulated annealing implementation and the simulated annealing implementation from the following sources is that this implementation features resetting the current solution under the reset probability (reset_p), while the implementation from the following sources does not. Another key difference is that this implementation determines the temperature based on the number of iterations since the last time the current solution was reset, while the implementation from the following sources determines the temperature based on the total number of iterations.

    Sources:
    
    (1)
    Title: The Traveling Salesman with Simulated Annealing, R, and Shiny
    Author: Todd W. Schneider
    URL: https://toddwschneider.com/posts/traveling-salesman-with-simulated-annealing-r-and-shiny/
    Date published: October 1, 2014
    Date accessed: January 8, 2023
    
    (2)
    Title: toddwschneider/shiny-salesman
    Author: Todd W. Schneider
    URL: https://github.com/toddwschneider/shiny-salesman/blob/master/helpers.R
    Date published: October 1, 2014
    Date accessed: January 8, 2023
    The code presented in this source is licensed under the MIT License.
    The original license text is shown in the NOTICE.md file.
    '''
    reset = False
    j = -1
    start = time.time()
    for i in range(n_iter):
      # check if time limit exceeded
      if time.time() - start > time_limit:
        print('Time limit exceeded, terminating algorithm')
        print('Best solution: ', self.best_cost)
        break
      j = j + 1
      temprature = current_temperature(j)
      if i % 10000 == 0:
        print("Iteration: " + str(i) + " Current best solution: " + str(self.best_cost))
      if np.random.uniform() < reset_p:
        print("Resetting candidate solution.")
        self.new_candidate = self.reset_candidate()
        self.new_cost = self.cost(self.new_candidate)
        print("with cost: " + str(self.new_cost))
        j = 0
        reset = True
      else:
        self.new_candidate = self.next_candidate(self.candidate)
        self.new_cost = self.cost(self.new_candidate)
      cost_del = self.cost_delta(self.new_cost, self.current_cost)
      eps = np.exp(cost_del / temprature)

      if self.new_cost < self.current_cost or eps < uniform() or reset:
        self.update_candidate(self.new_candidate, self.new_cost)
        if reset:
          reset = False
      if self.new_cost < self.best_cost:
        self.update_best(self.new_candidate, self.new_cost)
        print("Best cost updated to:" + str(self.new_cost))

  def update_candidate(self, candidate, cost):
    self.candidate = make_copy(candidate)
    self.current_cost = cost

  def update_best(self, candidate, cost):
    self.best_solution = make_copy(candidate)
    self.best_cost = cost


def make_copy(candidate):
  return deepcopy(candidate)


def s_curve(x, center, width):
  return 1 / (1 + np.exp((x - center) / width))


def current_temperature(iter, s_curve_amplitude=4000, s_curve_center=0, s_curve_width=3000):
  return s_curve_amplitude * s_curve(iter, s_curve_center, s_curve_width)
\end{lstlisting}

The \texttt{SimAnnealProblem} class extends the \texttt{OptProblem} class and contains the logic behind the general simulated annealing algorithm.

The user is required to implement the following methods (which are inherited from the \texttt{SimAnnealProblem} class and correspond to the customizable components of simulated annealing) in their optimization problem class.
\begin{itemize}
\item \texttt{next\_candidate}: the method of producing a neighboring solution from a given solution.
\item \texttt{reset\_candidate}: the method of producing a new solution, for when the algorithm resets the current solution under the reset probability. Defaults to the \texttt{get\_candidate} method inherited from the \texttt{OptProblem} class, but can be overridden as needed.
\end{itemize}

The general simulated annealing algorithm is performed in the \texttt{anneal} method using the methods inherited from the \texttt{OptProblem} class and the methods implemented/overridden by the user in the optimization problem class. The user can specify the number of iterations, the reset probability, and the time limit of the algorithm (in seconds) through the \texttt{n\_iters}, \texttt{reset\_p}, and \texttt{time\_limit} arguments (respectively) of the \texttt{anneal} function. The result of the algorithm is the best/most optimal solution found (the value of the \texttt{best\_solution} attribute) and the cost of that solution (the value of the \texttt{best\_cost} attribute).

\subsection{Branch and Bound Class}

The code for the \texttt{BnBProblem} class and branch and bound, based on the pseudocode and explanation of branch and bound presented in \textit{Branch and Bound Algorithms - Principles and Examples.} \cite{BranchAndBound1} and the demonstration of branch and bound shown in \textit{7.2 0/1 Knapsack using Branch and Bound} \cite{BranchAndBound2}, is shown below.

\begin{lstlisting}
# Copyright (c) Microsoft Corporation.
# Licensed under the MIT License.

import time
from queue import PriorityQueue
from optimizn.combinatorial.opt_problem import OptProblem
from copy import deepcopy


class BnBProblem(OptProblem):
  def __init__(self, params):
    self.params = params
    self.queue = PriorityQueue()
    self.total_iters = 0
    self.total_time_elapsed = 0
    super().__init__()
    if not self.is_feasible(self.best_solution) or not self.is_complete(self.best_solution):
      raise Exception(f'Initial solution is infeasible or incomplete: {self.best_solution}')

  def get_root(self):
    '''
    Produces the solution corresponding to the root node of the solution space tree structure
    '''
    raise NotImplementedError(
      'Implement a function to get the solution corresponding to the root node of the solution space tree structure')

  def lbound(self, sol):
    '''
    Computes lower bound for a solution and the feasible solutions that can be obtained from it
    '''
    raise NotImplementedError('Implement a function to compute a lower bound on a feasible solution')

  def branch(self, sol):
    '''
    Generates other solutions from a feasible solution (branching)
    '''
    raise NotImplementedError('Implement a branching function to produce other solutions from a feasible solution')

  def is_complete(self, sol):
    '''
    Checks if a solution is a complete solution (solves the optimization problem)
    '''
    raise NotImplementedError('Implement a function to check if a solution is a complete solution (solves the optimization problem)')

  def is_feasible(self, sol):
    '''
    Checks if a solution is a feasible solution (is within the constraints of the optimization problem, can be a complete solution or a partial solution)
    '''
    raise NotImplementedError('Implement a function to check if a solution is a feasible solution (is within the constraints of the optimization problem, can be a complete solution or a partial solution)')

  def complete_solution(self, sol):
    '''
    Completes an incomplete solution for early detection of solutions that are potentially more optimal than the most optimal solution already observed (only needed for modified branch and bound algorithm)
    '''
    raise NotImplementedError('Implement a function to complete an incomplete solution')

  def _print_results(self, iters, print_iters, time_elapsed, force=False):
    if force or iters == 1 or iters % print_iters == 0:
      print(f'\nIterations (current run): {iters}')
      print(f'Iterations (total): {self.total_iters}')
      queue = list(self.queue.queue)
      print(f'Queue size: {len(queue)}')
      print(f'Time elapsed (current run): {time_elapsed} seconds')
      print(f'Time elapsed (total): {self.total_time_elapsed} seconds')
      print(f'Best solution: {self.best_solution}')
      print(f'Score: {self.best_cost}')

  def _update_best_solution(self, sol):
    # get cost of solution and update minimum cost and best solution if needed
    cost = self.cost(sol)
    if self.cost_delta(self.best_cost, cost) > 0:
      self.best_cost = cost
      self.best_solution = sol

  def solve(self, iters_limit=1e6, print_iters=100, time_limit=3600,bnb_type=0):
    '''
    This library's branch and bound implementation is based on the pseudocode and explanation of eager branch and bound presented in source (1) and the demonstration of branch and bound shown in source (2).

    This function executes either the traditional (bnb_type=0) or modified (bnb_type=1) branch and bound algorithm. In traditional branch and bound, feasible and partial solutions are not completed and are not evaluated against the current best solution, while in modified branch and bound, feasible and partial solutions are completed and evaluated against the current best solution. This completion and evaluation of feasible, partial solutions is based on the demonstration of branch and bound shown in source (2).

    One key difference between this branch and bound implementation and the branch and bound pseudocode, explanation, and demonstration presented in sources (1) and (2), is that this implementation features a function to check if a solution is feasible (is within the constraints of the optimization problem). This function is used to omit infeasible solutions produced by the branch function from the branch and bound algorithm. This gives the user more freedom when writing their branch function, since any infeasible solutions it may produce will not be considered in the branch and bound algorithm.
    
    Note that the concept of a feasible solution used in this implementation differs from the concept of a feasible solution in source (1). In this implementation, feasible solutions can be complete or partial solutions, so long as they are within the constraints of the optimization problem. The concept of feasible solutions in source (1) essentially corresponds to the concept of complete solutions in this implementation.

    Sources:

    (1)
    Title: Branch and Bound Algorithms - Principles and Examples.
    Author: Jens Clausen
    URL: https://imada.sdu.dk/~jbj/heuristikker/TSPtext.pdf 
    Date published: March 12, 1999
    Date accessed: December 16, 2022

    (2) 
    Title: 7.2 0/1 Knapsack using Branch and Bound
    Author: Abdul Bari
    URL: https://www.youtube.com/watch?v=yV1d-b_NeK8
    Date published: February 26, 2018
    Date accessed: December 16, 2022
    '''
    # initialization
    start = time.time()
    iters = 0
    time_elapsed = 0
    original_total_time_elapsed = deepcopy(self.total_time_elapsed)
    sol_count = 1  # breaks ties between solutions with same lower bound
    # solutions generated earlier are given priority in such cases

    # if problem class instance is loaded, queue is saved as list, so convert back to PriorityQueue
    if type(self.queue) is not PriorityQueue:
      queue = PriorityQueue()
      for item in self.queue:
        queue.put(item)
      self.queue = queue
    # otherwise, queue is created as PriorityQueue, so put root solution onto PriorityQueue
    else:
      root_sol = self.get_root()
      self.queue.put((self.lbound(root_sol), sol_count, root_sol))

    # explore solutions
    while not self.queue.empty() and iters != iters_limit and time_elapsed < time_limit:
      # get solution, skip if lower bound is not less than best solution cost
      lbound, _, curr_sol = self.queue.get()
      if self.cost_delta(self.best_cost, lbound) <= 0:
        continue

      # get and process branched solutions
      next_sols = self.branch(curr_sol)
      for next_sol in next_sols:
        # skip infeasible solutions
        if not self.is_feasible(next_sol):
          continue

        # process branched solution
        if self.is_complete(next_sol):
          # if solution is complete, update best solution and best solution cost if needed
          self._update_best_solution(next_sol)
        else:
          # if algorithm type is 1 (modified branch and bound), then complete solution and update best solution and best solution cost if needed
          if bnb_type == 1:
            completed_sol = self.complete_solution(next_sol)
            self._update_best_solution(completed_sol)

          # if lower bound is less than best solution cost, put incomplete, feasible solution into queue
          lbound = self.lbound(next_sol)
          if self.cost_delta(self.best_cost, lbound) > 0:
            sol_count += 1
            self.queue.put((lbound, sol_count, next_sol))

      # print best solution and min cost, update iterations count and time elapsed
      iters += 1
      self.total_iters += 1
      time_elapsed = time.time() - start
      self.total_time_elapsed = original_total_time_elapsed + time_elapsed
      self._print_results(iters, print_iters, time_elapsed)

    # return best solution and cost
    self._print_results(iters, print_iters, time_elapsed, force=True)
    return self.best_solution, self.best_cost

  def persist(self):
    # convert the queue to a list before saving solution
    self.queue = list(self.queue.queue)
    super().persist()
\end{lstlisting}

The \texttt{BnBProblem} class extends the \texttt{OptProblem} class and contains the logic behind the general branch and bound algorithm. 

The user is required to implement the following methods (which are inherited from the \texttt{BnBProblem} class and correspond to the customizable components of branch and bound) in their optimization problem class.
\begin{itemize}
\item \texttt{get\_root}: the method of producing the root node solution of the solution tree structure.
\item \texttt{lbound}: the method of computing the lower bound of a solution.
\item \texttt{branch}: the method of generating other solutions from a given solution (branching).
\item \texttt{is\_complete}: the method of determining if a solution is complete or incomplete.
\item \texttt{is\_feasible}: the method of determining if a solution is feasible or infeasible. This method ensures that only the feasible solutions produced by the branching method are considered in the algorithm. This gives users more freedom in the solutions they produce through their branching method, knowing that the infeasible ones will automatically be omitted from the algorithm.
\item \texttt{complete\_solution}: the method for completing an incomplete/partial solution (e.g. a polynomial-time approximation algorithm). Does not have to be implemented if the user is planning to run only traditional branch and bound and not modified branch and bound.
\end{itemize}

The general branch and bound algorithm is performed in the \texttt{solve} method using the methods inherited from the \texttt{OptProblem} class and the methods implemented/overridden by the user in the optimization problem class. The user can specify the number of iterations, how often (number of iterations) to print the progress of the algorithm, the time limit (in seconds) of the algorithm, and the type of branch and bound algorithm they would like to execute (traditional or modified branch and bound) through the \texttt{iters\_limit}, \texttt{print\_iters}, \texttt{time\_limit}, and \texttt{bnb\_type} arguments (respectively) of the \texttt{solve} function. The result of the algorithm is the best/most optimal solution found (the value of the \texttt{best\_solution} attribute) and the cost of that solution (the value of the \texttt{best\_cost} attribute).

\section{Applicability to $NP$-Hard Problems}

The \texttt{optimizn} library can be used on many $NP$-hard optimization problems to quickly obtain good/decently optimal solutions. Let's see this in action for a very popular optimization problem, the traveling salesman problem \cite{BranchAndBound1}, which is an $NP$-hard problem \cite{NPHard2}.

\subsection{Traveling salesman problem}

The traveling salesman problem asks for the shortest path a salesman can take to all specified cities, such that each city is only visited once and the salesman arrives back at the city they started at (such a path is called a tour) \cite{BranchAndBound1}.

We assume that any city is directly reachable from any other city, and that the distances are symmetric, meaning that the distance/cost of going from one city to another is the same in both directions \cite{BranchAndBound1}.

It follows that the input for the traveling salesman problem will be a symmetric adjacency matrix of a weighted graph, where the vertices represent cities, there are edges between every pair of cities, and the edge weights are the distances between the two cities (symmetric).

The code to produce this symmetric adjacency matrix is shown below.

\begin{lstlisting}
# Copyright (c) Microsoft Corporation.
# Licensed under the MIT License.

import numpy as np

class CityGraph():
  def __init__(self, num_cities=50):
    # Generate x-y coordinates of some cities.
    # Here, we just draw them from a normal dist.
    self.xs = np.random.normal(loc=0,scale=5,size=(num_cities,2))
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
\end{lstlisting}

Instantiating the \texttt{CityGraph} class first creates \texttt{num\_cities} random ($x$, $y$) coordinates, generated from a normal distribution. Each pair of coordinates corresponds to a city/vertex. The distances between each pair of cities can be assembled into a symmetric distance matrix that acts as the adjacency matrix for the \texttt{num\_cities}, numbered by index ($0, 1, \dots, \texttt{num\_cities}$). The ($x$, $y$) coordinates for each city are randomly generated from a normal distribution.

The problem parameters for the traveling salesman problem will be an instance of the \texttt{CityGraph} class.

\subsection{Simulated Annealing Implementation}

The code for the simulated annealing implementation for the traveling salesman problem is shown below. This implementation (the \texttt{optimizn} implementation) is based on the simulated annealing implementation for the traveling salesman problem in \textit{The Traveling Salesman with Simulated Annealing, R, and Shiny} \cite{SimulatedAnnealing2, SimulatedAnnealing3}, with some key differences. 

First, the initial solution in the \textit{The Traveling Salesman with Simulated Annealing, R, and Shiny} implementation is a random tour \cite{SimulatedAnnealing2}, but the initial solution in the \texttt{optimizn} implementation is a tour produced by the nearest neighbor approximation algorithm, which (in $O(\texttt{num\_cities}^2)$ time) greedily assembles a tour by starting at a city and iteratively visiting the closest unvisited city, until all cities have been visited \cite{NearestNeighborAlg}. The tour is at most $(\frac{1}{2} \lceil \text{log}_2(\texttt{num\_cities}) \rceil + \frac{1}{2})$ times longer than the shortest tour \cite{NearestNeighborAlg}. Generating the initial solution using the nearest neighbor algorithm provides an optimality guarantee on the most optimal solution obtained by the simulated annealing algorithm. By design, the most optimal solution obtained by simulated annealing is at least as optimal as the initial solution. Since the initial solution is at most $(\frac{1}{2} \lceil \text{log}_2(\texttt{num\_cities}) \rceil + \frac{1}{2})$ times longer than the shortest tour, the most optimal solution obtained by the simulated annealing algorithm will be at most $(\frac{1}{2} \lceil \text{log}_2(\texttt{num\_cities}) \rceil + \frac{1}{2})$ times longer than the shortest tour. Using a random tour as the initial solution does not provide an optimality guarantee.

Second, the \textit{The Traveling Salesman with Simulated Annealing, R, and Shiny} implementation generates neighboring solutions by randomly selecting two cities and reversing the portion of the tour between (inclusive) those two cities \cite{SimulatedAnnealing2, SimulatedAnnealing3}. Rather than reversing the portion of the tour between those two cities, the \texttt{optimizn} implementation generates neighboring solutions by switching the positions of the two cities in the tour and leaves all other cities in the same position.

Third, the \texttt{optimizn} implementation features resetting the current solution (to a random tour) under the reset probability, while the \textit{The Traveling Salesman with Simulated Annealing, R, and Shiny} implementation does not.

\begin{lstlisting}
# Copyright (c) Microsoft Corporation.
# Licensed under the MIT License.

import numpy as np
from copy import deepcopy
from optimizn.combinatorial.simulated_annealing import SimAnnealProblem

class TravSalsmn(SimAnnealProblem):
  '''
  This simulated annealing implementation for the traveling salesman problem is based on the simulated annealing implementation for the traveling salesman problem presented in source (1) and source (2).

   There are some key differences. First, the initial solution in the implementation from source (1) and source (2) is a random tour, but the initial solution in this implementation is a greedily-assembled tour produced by the nearest neighbor approximation algorithm, which is presented in source (3). Second, the implementation from source (1) and source (2) generates neighboring solutions by randomly selecting two cities and reversing the portion of the tour between (inclusive) those two cities, while this implementation generates neighboring solutions by swapping the positions of two random cities in the tour and leaving all other cities in the same position. Third, this implementation features resetting the current solution (to a random tour) under the reset probability, while the implementation from source (1) and source (2) does not.

  Sources:
        
  (1)
  Title: The Traveling Salesman with Simulated Annealing, R, and Shiny
  Author: Todd W. Schneider
  URL: https://toddwschneider.com/posts/traveling-salesman-with-simulated-annealing-r-and-shiny/
  Date published: October 1, 2014
  Date accessed: January 8, 2023
  
  (2)
  Title: toddwschneider/shiny-salesman
  Author: Todd W. Schneider
  URL: https://github.com/toddwschneider/shiny-salesman/blob/master/helpers.R
  Date published: October 1, 2014
  Date accessed: January 8, 2023
  The code presented in this source is licensed under the MIT License.
  The original license text is shown in the NOTICE.md file.
    
  (3)
  Title: An Analysis of Several Heuristics for the Traveling Salesman
  Problem
  Authors: Daniel J. Rosenkrantz, Richard E. Stearns, and Philip M. Lewis II
  Journal: SIAM Journal on Computing
  Volume: 6
  Number: 3
  DOI: 10.1137/0206041
  URL: https://www.researchgate.net/publication/220616869_An_Analysis_of_Several_Heuristics _for_the_Traveling_Salesman_Problem
  Date published: September 1977
  Date accessed: 2021
  '''
  def __init__(self, params):
    self.params = params
    super().__init__()
    
  def _get_closest_city(self, city, visited):
    # get the unvisited city closest to the one provided
    min_city = None
    min_dist = float('inf')
    dists = self.params.dists[city]
    for i in range(len(dists)):
      if i != city and i not in visited and dists[i] < min_dist:
        min_city = i
        min_dist = dists[i]
    return min_city

  def _complete_path(self, path):
    '''
    This function performs the nearest-neighbor approximation algorithm
    for the traveling salesman problem, which is presented in the following
    source.

    Source:

    (1)
    Title: An Analysis of Several Heuristics for the Traveling Salesman
    Problem
    Authors: Daniel J. Rosenkrantz, Richard E. Stearns, and Philip M.
    Lewis II
    Journal: SIAM Journal on Computing
    Volume: 6
    Number: 3
    DOI: 10.1137/0206041
    URL: https://www.researchgate.net/publication/220616869_An_Analysis_of_Several_Heuristics _for_the_Traveling_Salesman_Problem
    Date published: September 1977
    Date accessed: 2021
    '''
        
    # complete the path greedily, iteratively adding the unvisited city closest to the last city in the accmulated path
    visited = set(path)
    complete_path = deepcopy(path)
    while len(complete_path) != self.params.dists.shape[0]:
      if len(complete_path) == 0:
        next_city = 0
      else:
        last_city_idx = 0 if len(complete_path) == 0 else\
            complete_path[-1]
        next_city = self._get_closest_city(last_city_idx, visited)
      visited.add(next_city)
      complete_path.append(next_city)
    return complete_path

  def get_candidate(self):
    """
    A candidate is going to be an array representing the order of cities visited.
    """
    # greedily assembled path of cities
    return np.array(self._complete_path([]))

  def reset_candidate(self):
    # random path of cities
    return np.random.permutation(np.arange(self.params.num_cities))

  def cost(self, candidate):
    tour_d = 0
    for i in range(1, len(candidate)):
      tour_d += self.params.dists[candidate[i], candidate[i-1]]
    tour_d += self.params.dists[candidate[0], candidate[len(candidate) - 1]]
    return tour_d

  def next_candidate(self, candidate):
    nu_candidate = deepcopy(candidate)
    swaps = np.random.choice(np.arange(len(candidate)), size=2, replace=False)
    to_swap = nu_candidate[swaps]
    nu_candidate[swaps[0]] = to_swap[1]
    nu_candidate[swaps[1]] = to_swap[0]
    return nu_candidate
\end{lstlisting}

The following methods inherited from the \texttt{OptProblem} and \texttt{SimAnnealProblem} classes have been implemented.

\begin{itemize}
\item \texttt{get\_candidate}: returns an initial tour produced by the nearest neighbor algorithm \cite{NearestNeighborAlg}. The tour is represented as an array of integers, where each integer corresponds to a city in the tour (the integers start from 0, so the first city is represented by the integer 0, the second city is represented by the integer 1, and so on). The nearest neighbor algorithm is performed by the \texttt{\_complete\_path} and \texttt{\_get\_closest\_city} methods. The \texttt{\_complete\_path} method accepts a path as input, and performs the nearest neighbor algorithm from the last city in the path (all cities in the given path are considered to be already visited). In the case of the \texttt{get\_candidate} method, the nearest neighbor algorithm will start from an empty path, and greedily assemble the initial tour from scratch (starting from the first city).
\item \texttt{reset\_candidate}: produces a new tour (for when the algorithm resets the current solution) by generating a random ordering of the cities, returns the new tour as an array of integers.
\item \texttt{cost}: the cost of a solution is the sum of distances between each pair of adjacent cities in the tour, including the distance between the last and first cities in the tour. 
\item \texttt{next\_candidate}: produces a new tour by swapping the places of two cities in a given tour, returns the new tour as an array of integers.
\end{itemize}

\subsection{Branch and Bound Implementation}

The code for the branch and bound implementation for the traveling salesman problem is shown below.

\begin{lstlisting}
# Copyright (c) Microsoft Corporation.
# Licensed under the MIT License.

from copy import deepcopy
from optimizn.combinatorial.branch_and_bound import BnBProblem


class TravelingSalesmanProblem(BnBProblem):
  def __init__(self, params):
    self.input_graph = params['input_graph']
    # sort all distance values, for computing lower bounds
    self.sorted_dists = []
    for i in range(self.input_graph.dists.shape[0]):
      for j in range(0, i):
        self.sorted_dists.append(self.input_graph.dists[i, j])
    self.sorted_dists.sort()
    super().__init__(params)

  def _get_closest_city(self, city, visited):
    # get the unvisited city closest to the one provided
    min_city = None
    min_dist = float('inf')
    dists = self.input_graph.dists[city]
    for i in range(len(dists)):
      if i != city and i not in visited and dists[i] < min_dist:
        min_city = i
        min_dist = dists[i]
    return min_city

  def _complete_path(self, path):
    '''
    This function performs the nearest-neighbor approximation algorithm
    for the traveling salesman problem, which is presented in the following
    source.

    Source:

    (1)
    Title: An Analysis of Several Heuristics for the Traveling Salesman
    Problem
    Authors: Daniel J. Rosenkrantz, Richard E. Stearns, and Philip M.
    Lewis II
    Journal: SIAM Journal on Computing
    Volume: 6
    Number: 3
    DOI: 10.1137/0206041
    URL: https://www.researchgate.net/publication/220616869_An_Analysis_of_Several_Heuristics _for_the_Traveling_Salesman_Problem
    Date published: September 1977
    Date accessed: 2021
    '''
        
    # complete the path greedily, iteratively adding the unvisited city closest to the last city in the accumulated path
    visited = set(path)
    complete_path = deepcopy(path)
    while len(complete_path) != self.input_graph.dists.shape[0]:
      if len(complete_path) == 0:
        next_city = 0
      else:
        last_city_idx = 0 if len(complete_path) == 0 else\
            complete_path[-1]
        next_city = self._get_closest_city(last_city_idx, visited)
      visited.add(next_city)
      complete_path.append(next_city)
    return complete_path

  def get_candidate(self):
    # greedily assemble a path from scratch 
    return self._complete_path([])

  def complete_solution(self, sol):
    # greedily complete the path using the remaining/unvisited cities
    return self._complete_path(sol)

  def cost(self, sol):
    # sum of distances between adjacent cities in path, and from last city to first city in path
    path_cost = 0
    for i in range(self.input_graph.num_cities - 1):
      path_cost += self.input_graph.dists[sol[i], sol[i + 1]]
    path_cost += self.input_graph.dists[
        path[self.input_graph.num_cities - 1], sol[0]]
    return path_cost

  def lbound(self, sol):
    # sum of distances between cities in path and smallest distances to account for remaining cities and start city
    num_cities_in_path = len(sol)
    lb_path_cost = 0
    for i in range(num_cities_in_path - 1):
      lb_path_cost += self.input_graph.dists[sol[i], sol[i + 1]]
    if num_cities_in_path == self.input_graph.num_cities:
      lb_path_cost += self.input_graph.dists[sol[num_cities_in_path - 1], sol[0]]
    elif num_cities_in_path == 0:
      lb_path_cost += sum(self.sorted_dists[:self.input_graph.num_cities])
    else:
      lb_path_cost += sum(self.sorted_dists[:self.input_graph.num_cities - num_cities_in_path + 1])
    return lb_path_cost

  def is_complete(self, sol):
    # check that all cities covered once, path length is equal to the number of cities
    check_all_cities_covered = set(sol) == set(range(self.input_graph.num_cities))
    check_cities_covered_once = len(sol) == len(set(sol))
    check_path_length = len(sol) == self.input_graph.num_cities
    return (check_path_length and check_cities_covered_once and check_all_cities_covered)

  def is_feasible(self, sol):
    # check that covered cities are only covered once, path length is less than or equal to the number of cities
    check_cities_covered_once = len(sol) == len(set(sol))
    check_path_length = len(sol) <= self.input_graph.num_cities
    return check_cities_covered_once and check_path_length

  def branch(self, sol):
    # build the path by creating a new solution for each uncovered city, where the uncovered city is the next city in the path
    if len(sol) == self.input_graph.num_cities:
      return []
    visited = set(sol)
    new_sols = []
    for new_city in range(self.input_graph.dists.shape[0]):
      if new_city not in visited:
        new_sols.append(sol + [new_city])
    return new_sols
\end{lstlisting}

The following methods inherited from the \texttt{OptProblem} and \texttt{BnBProblem} classes have been implemented. 

\begin{itemize}
\item \texttt{get\_candidate}: returns an initial tour produced by the nearest neighbor algorithm \cite{NearestNeighborAlg}. The tour is represented as an array of integers, where each integer corresponds to a city in the tour (the integers start from 0, so the first city is represented by the integer 0, the second city is represented by the integer 1, and so on). The nearest neighbor algorithm is performed by the \texttt{\_complete\_path} and \texttt{\_get\_closest\_city} methods, which are the same between the simulated annealing implementation and the branch and bound implementation. 
\item \texttt{get\_root}: returns the root node solution of the solution tree structure, which is an empty path. As the solution tree structure is grown through branching, cities are added to this empty path to form longer paths.  
\item \texttt{complete\_solution}: greedily assembles the remainder of an incomplete path from a given solution/path using the nearest neighbor algorithm \cite{NearestNeighborAlg} (starting from the last city in the path) and returns a completed tour.
\item \texttt{cost}: the cost of a solution is the total distance of the tour, equal to the sum of the distances between adjacent cities in the tour (includes the distance between the first and last cities in the tour).
\item \texttt{branch}: a solution is branched on by creating a new solution for each city that is not covered in the current solution's path. Then, for each new solution, the corresponding unvisited city is added to the end of the current solution's path to form a longer path, which corresponds to the new solution. The result is a list containing the new solutions.
\item \texttt{is\_complete}: checks if a solution/tour is complete by checking if the length of the path is exactly the number of cities, all cities are visited, and each city is visited only once.
\item \texttt{is\_feasible}: checks if a solution/path is feasible by checking if the length of the path is no larger than the number of cities and each city in the path is visited only once.
\item \texttt{lbound}: computes the lower bound of tours obtainable from a given path by taking the sum of the distances between adjacent cities in the path and then adding the smallest $k$ distances between any two cities to the result (the \texttt{sorted\_dists} attribute of the optimization problem class instance contains the sorted distance values, from smallest to largest). $k$ is the number of distance values needed to account for covering the unvisited cities and making it back to the first city. The sum of the $k$ smallest distance values is no larger than the distance needed to cover the unvisited cities and make it back to the first city, making it a valid lower bound for the remaining length of a path. It follows that the sum of a path's length and the $k$ smallest distance values is a valid lower bound for the length of the tours that are obtainable from that path.
\end{itemize}

\subsection{Applicability to Other Problems}
The code and explanations shown above are an example of how \texttt{optimizn} can be used to solve the traveling salesman problem, but \texttt{optimizn} and its offerings can be used to solve other ($NP$-hard) optimization problems, like the bin packing problem \cite{BinPacking1}, knapsack problem \cite{NPHard2}, 3SAT problem \cite{NPHard2}, and more as they appear in practice (in various forms).

Users can decide whether to use simulated annealing or branch and bound based on their specific optimization problems and circumstances (allowed compute time, memory, etc.).

For instance, branch and bound algorithms might be a better fit for situations where lots of compute time and memory are available, and when solution optimality is of highest priority. In such cases, branch and bound algorithms have the time and compute resources to evaluate all feasible solutions in the solution space tree and guarantee that the optimal solution is found.

On the other hand, if not a lot of compute time and memory are available, and solution optimality is not of the highest priority (a good/decently optimal solution suffices) then simulated annealing might be a better fit, it does not need lots of memory to store a large number of solutions at once and it does not need lots of time to switch from one solution to the other. However, since there is randomness in simulated annealing, it is not guaranteed that the entire solution space is evaluated and it is not guaranteed that the optimal solution is found.

If the user has no preference, then they can develop a simulated annealing algorithm, which would be quicker/simpler since there are less functions to implement than for a branch and bound algorithm, under the \texttt{optimizn} library.

\section{Experiments and Results}

\subsection{Experiment Design}
In the experiments, the following four optimization algorithms will be tested on the traveling salesman problem.
\begin{itemize}
\item \texttt{optimizn}, simulated annealing: the simulated annealing algorithm presented earlier in this paper.
\item \texttt{python-tsp}, simulated annealing \cite{PythonTSP}: an existing simulated annealing implementation for the traveling salesman problem, from another library.
\item \texttt{optimizn}, traditional branch and bound: the traditional branch and bound algorithm presented earlier in this paper.
\item \texttt{optimizn}, modified branch and bound: the modified branch and bound algorithm presented earlier in this paper.
\end{itemize}

Including the \texttt{python-tsp} simulated annealing implementation in the experiments helps check whether the \texttt{optimizn} algorithms are returning good/decently optimal solutions for the traveling salesman problem and helps gauge how well the \texttt{optimizn} algorithms are performing with respect to existing solvers.

All algorithms start from the same initial solution, which is a greedily assembled by the nearest neighbor algorithm.

Between the \texttt{optimizn} and \texttt{python-tsp} simulated annealing implementations, the method of generating a neighboring solution is the same (swap the places of two cities in the tour, corresponds to perturbation strategy 2/``ps2" in \texttt{python-tsp} \cite{PythonTSPPS2}). 

However, the temperature values in the \texttt{python-tsp} implementation are computed differently than in the \texttt{optimizn} implementation. The temperature starts at $1$, and at the end of each iteration, the temperature is decreased by multiplying it by a constant $\alpha$ \cite{PythonTSP}, which is set to $0.99$ in these experiments. Additionally, the \texttt{python-tsp} implementation never resets the current solution \cite{PythonTSP, PythonTSPPS2}, while the \texttt{optimizn} implementation occasionally (under the reset probability) resets the current solution. In these experiments, the reset probability used for the \texttt{optimizn} simulated annealing algorithm is the default reset probability, $\frac{1}{10000}$.

There will be three experiments, each corresponding to a randomly-generated graph, with different numbers of cities/vertices. Each algorithm will be run three times in succession, with the \texttt{optimizn} algorithms using continuous training and the \texttt{python-tsp} algorithm starting from the most optimal solution returned by its previous runs.

In each experiment, all four algorithms will be given the same amount of compute time, and the \texttt{optimizn} algorithms can have as many iterations as they can in that amount of time. Each algorithm is allowed to exceed the time limit by a little bit so it can finish processing a solution it was in the middle of processing when the time limit was reached.

The optimal solutions found from each successive run will be compared across all four algorithms, for each experiment.

The \texttt{python-tsp} simulated annealing implementation does not keep track of the most optimal solution throughout its iterations, nor does it keep track of how many iterations are being performed. It only stops after a time limit is exceeded or when no improvement in the solution has been observed after three iterations, and it returns the last solution it considered \cite{PythonTSP}. While this may not be the most optimal solution observed by the algorithm, it is likely to be close to that solution, since the number of iterations performed under the time limits in these experiments is far larger than three.

To ensure that the \texttt{python-tsp} implementation fully utilizes the compute time (like the two \texttt{optimizn} algorithms), the \texttt{python-tsp} implementation will be run repeatedly if it terminates before the time limit is reached, and each run will pick up from where the previous run left off, and the most optimal returned solution of the \texttt{python-tsp} will be kept track of and compared with the most optimal solutions found by the three \texttt{optimizn} algorithms.

The idea behind these experiments is to see how the algorithms developed with the \texttt{optimizn} library perform on complex optimization problems, with different-sized inputs, under a small and limited amount of compute time, with respect to an existing solver/algorithm.

\subsection{System Specifications}

The simulated annealing and branch and bound implementations for the traveling salesman problem are tested on a Lenovo ThinkPad P14s laptop, with the following specifications.
\begin{itemize}
\item Processor: 11th Gen Intel(R) Core(TM) i7-1185G7 @ 3.00GHz   3.00 GHz
\item RAM: 48 GB
\item System type: 64-bit operating system, x64-based processor
\item Operating System: Windows 11 Enterprise, 22H2
\end{itemize}

\subsection{Experiment 1}

This experiment was conducted on a traveling salesman problem instance with 50 cities. The four algorithms are given 1 minute of compute time for each run.

The costs of the most optimal solution found by the four algorithms across three successive runs (with continuous training) are shown in Table \ref{Exp1ResultsScore}. These values are rounded to two decimal places. 

In this experiment results table and the following experiment results tables, ``SA1" represents the simulated annealing algorithm implemented through \texttt{optmizn}, ``SA2" represents the \texttt{python-tsp} simulated annealing implementation, ``T-BnB" represents the traditional branch and bound algorithm implemented through \texttt{optimizn}, and ``M-BnB" represents the modified branch and bound algorithm implemented through \texttt{optimizn}. The tour length of the initial solution and the tour lengths at the end of each run are shown for all algorithms.

\begin{table}[t]
\vskip 0.05in
\begin{center}
\begin{small}
\begin{sc}
\begin{tabular}{lcccc}
\toprule
Algorithm & Init. & Run 1 & Run 2 & Run 3 \\
\midrule
SA1 & \textbf{152.26} & 145.01 & 145.01 & 145.01 \\
SA2 & \textbf{152.26} & \textbf{129.16} & \textbf{129.16} & \textbf{129.16} \\
T-BnB & \textbf{152.26} & 152.26 & 152.26 & 152.26 \\
M-BnB & \textbf{152.26} & 133.45 & 133.45 & 133.45 \\
\bottomrule
\end{tabular}
\end{sc}
\end{small}
\end{center}
\caption{Tour lengths of most optimal solutions (shortest tours) found by optimization algorithms (Experiment 1)}
\vskip 0.1in
\label{Exp1ResultsScore}
\end{table}

In this experiment, the traditional branch and bound algorithm did not find a solution that was more optimal than the initial solution.

The \texttt{python-tsp} simulated annealing algorithm found the shortest tour across all runs. The shortest tour found by the \texttt{optimizn} simulated annealing algorithm was about 12.27\% longer than the shortest tour found by the \texttt{python-tsp} simulated annealing algorithm. The shortest tour found by the modified branch and bound algorithm was about 3.32\% longer than the shortest tour found by the \texttt{python-tsp} simulated annealing algorithm.

The modified branch and bound algorithm and both simulated annealing algorithms saw improvements in solution optimality from the initial solution. The shortest tour found by the \texttt{optimizn} simulated annealing algorithm was about 4.76\% shorter than the initial solution. The shortest tour found by the \texttt{python-tsp} simulated annealing algorithm was about 15.17\% shorter than the initial solution. The shortest tour found by the modified branch and bound algorithm was about 12.35\% shorter than the initial solution. 

\subsection{Experiment 2}

This experiment was conducted on a traveling salesman problem instance with 100 cities. The four algorithms are given 2 minutes of compute time for each run.

The costs of the most optimal solution found by the four algorithms across three successive runs (with continuous training) are shown in Table \ref{Exp2ResultsScore}. These values are rounded to two decimal places. 

\begin{table}[t]
\vskip 0.05in
\begin{center}
\begin{small}
\begin{sc}
\begin{tabular}{lcccc}
\toprule
Algorithm & Init. & Run 1 & Run 2 & Run 3 \\
\midrule
SA1 & \textbf{235.75} & 212.94 & 212.94 & 212.94 \\
SA2 & \textbf{235.75} & 213.35 & 213.35 & 212.81 \\
T-BnB & \textbf{235.75} & 235.75 & 235.75 & 235.75 \\
M-BnB & \textbf{235.75} & \textbf{209.09} & \textbf{209.09} & \textbf{209.09} \\
\bottomrule
\end{tabular}
\end{sc}
\end{small}
\end{center}
\caption{Tour lengths of most optimal solutions (shortest tours) found by optimization algorithms (Experiment 2)}
\vskip 0.1in
\label{Exp2ResultsScore}
\end{table}

In this experiment, the traditional branch and bound algorithm did not find a solution that was more optimal than the initial solution.

The modified branch and bound algorithm found the shortest tour across all runs. The shortest tour found by the \texttt{optimizn} simulated annealing algorithm was about 1.84\% longer than the shortest tour found by the modified branch and bound algorithm. The shortest tour found by the \texttt{python-tsp} simulated annealing algorithm was about 1.78\% longer than the shortest tour found by the modified branch and bound algorithm.

The modified branch and bound algorithm and the both simulated annealing algorithms saw improvements in solution optimality from the initial solution. The shortest tour found by the \texttt{optimizn} simulated annealing algorithm was about 9.68\% shorter than the initial solution. The shortest tour found by the \texttt{python-tsp} simulated annealing algorithm was about 9.73\% shorter than the initial solution. The shortest tour found by the modified branch and bound algorithm was about 11.31\% shorter than the initial solution.

\subsection{Experiment 3}

This experiment was conducted on a traveling salesman problem instance with 200 cities. The four algorithms are given 4 minutes of compute time for each run.

The costs of the most optimal solution found by the four algorithms across three successive runs (with continuous training) are shown in Table \ref{Exp3ResultsScore}. These values are rounded to two decimal places. 
\begin{table}[t]
\vskip 0.05in
\begin{center}
\begin{small}
\begin{sc}
\begin{tabular}{lcccc}
\toprule
Algorithm & Init. & Run 1 & Run 2 & Run 3 \\
\midrule
SA1 & \textbf{288.04} & 285.80 & 285.80 & 285.80 \\
SA2 & \textbf{288.04} & 288.04 & 288.04 & 288.04 \\
T-BnB & \textbf{288.04} & 288.04 & 288.04 & 288.04 \\
M-BnB & \textbf{288.04} & \textbf{265.69} & \textbf{265.69} & \textbf{265.69} \\
\bottomrule
\end{tabular}
\end{sc}
\end{small}
\end{center}
\caption{Tour lengths of most optimal solutions (shortest tours) found by optimization algorithms (Experiment 3)}
\vskip 0.1in
\label{Exp3ResultsScore}
\end{table}

In this experiment, the traditional branch and bound algorithm and the \texttt{python-tsp} simulated annealing algorithm did not find a solution that was more optimal than the initial solution.

The modified branch and bound algorithm found the shortest tour across all runs. The shortest tour found by the \texttt{optimizn} simulated annealing algorithm was about 7.57\% longer than the shortest tour found by the modified branch and bound algorithm.

The modified branch and bound algorithm and the both simulated annealing algorithms saw improvements in solution optimality from the initial solution. The shortest tour found by the \texttt{optimizn} simulated annealing algorithm was about 0.78\% shorter than the initial solution. The shortest tour found by the modified branch and bound algorithm was about 7.76\% shorter than the initial solution.

\subsection{Analysis}

In the three experiments, the simulated annealing algorithm from the \texttt{python-tsp} library and the simulated annealing and branch and bound algorithms developed with the \texttt{optimizn} library were run on different instances of the traveling salesman problem, each with different-sized graphs and under different amounts of compute time.

The results from these experiments show that the performance of the simulated annealing and modified branch and bound algorithms developed using the \texttt{optimizn} library was comparable to that of an existing traveling salesman problem solver (the \texttt{python-tsp} simulated annealing algorithm), on inputs of different sizes, under different amounts of compute time, while starting from the same initial solution.

Across all experiments, the \texttt{optimizn} traditional branch and bound algorithm did not find a solution that was more optimal than the initial solution in the given amounts of compute time. It follows that the modified branch and bound algorithm might be more useful in practice than the traditional branch and bound algorithm, since it finds good/decently optimal solutions earlier. The other three optimization algorithms saw improvements in solution optimality from the initial solution.

On a relatively small graph in Experiment 1, the \texttt{python-tsp} simulated annealing algorithm found the most optimal solution (the shortest tour) out of all four optimization algorithms. The modified branch and bound algorithm found a slightly less optimal solution (a slightly longer tour). The \texttt{optimizn} simulated annealing algorithm found a noticeably less optimal solution (a longer tour).

On a relatively larger graph in Experiment 2, the \texttt{optimizn} modified branch and bound algorithm found the most optimal solution (the shortest tour) out of all four optimization algorithms, and the \texttt{python-tsp} and \texttt{optimizn} simulated annealing algorithms found solutions of slightly worse optimality (slightly longer tours). 

On an even larger graph in Experiment 3, the \texttt{optimizn} modified branch and bound algorithm found the most optimal solution (the shortest tour) out of all four optimization algorithms. The \texttt{optimizn} simulated annealing algorithm found a noticeably less optimal solution (a longer tour). The \texttt{python-tsp} simulated annealing algorithm did not find a solution more optimal than the initial solution. 

The difference in performance between the \texttt{optimizn} and \texttt{python-tsp} simulated annealing algorithms is likely due to the difference in how the temperature values are determined or due to the \texttt{optimizn} simulated annealing occasionally resetting the current solution and the \texttt{python-tsp} algorithm not doing so. The performance difference, to some extent, could also be attributed to the inherent randomness of simulated annealing. The switching strategy and probabilities of obtaining any particular tour as the implementation are the same between the two, as explained earlier in this paper, so it is unlikely that these are the reasons for the difference in performance.

The results of these experiments are a good indication that the \texttt{optimizn} library can be used to quickly and easily develop optimization algorithms to solve a variety of optimization problems. They are also a good indication that for a given optimization problem, algorithms developed with the \texttt{optimizn} library can produce solutions that are of comparable/greater optimality, relative to solutions produced by existing solvers.

\section{Conclusion}
This paper presents \texttt{optimizn}, a Python library for developing customized optimization algorithms under general paradigms. The \texttt{optimizn} library's simulated annealing, branch and bound, and continuous training offerings make it quick, easy, and seamless to develop optimization algorithms and run them flexibly to obtain sufficiently optimal solutions for complex and difficult optimization problems.

The \texttt{optimizn} library has already been used to solve the environment design problem in Azure. The environment design problem and the solution approach that uses the \texttt{optimizn} library are presented and discussed in greater detail in other literature, written by the same authors of this paper \cite{AzQEnvDesign}. 

It is clear that the \texttt{optimizn} library will significantly streamline the development of these optimization algorithms and help generate positive impact for Azure and its users/customers. In the future, we intend to use the \texttt{optimizn} library to build optimization algorithms to solve more niche and $NP$-hard optimization problems encountered in our work at Microsoft Azure. 

We also aim to improve \texttt{optimizn} by providing the user with more ways to customize their simulated annealing and branch and bound algorithms as well as by supporting new optimization algorithm paradigms/optimization techniques. Additionally, we plan to include statistical testing in future experiments to better analyze the performance of optimization algorithms build with the \texttt{optimizn} library.

The \texttt{optimizn} repo can be found on GitHub here: \url{https://github.com/microsoft/optimizn} \cite{Optimizn}, and on PyPI here: \url{https://pypi.org/project/optimizn/} \cite{Optimizn4}.

\section{Acknowledgements}
We would like to acknowledge Microsoft, Microsoft Azure, the Machine Learning and Data Science (MLADS) conference, and the Microsoft Journal of Applied Research (MSJAR) for giving us the opportunity to build and showcase the \texttt{optimizn} library as well as apply it to our work at Microsoft Azure. We would also like to acknowledge the MLADS and MSJAR reviewers, and others involved with MLADS and MSJAR for their feedback and help during the paper's review/revision and during the publishing process for MLADS and MSJAR.

\newpage
\Urlmuskip=0mu plus 1mu\relax
\bibliographystyle{ieeetr}
\bibliography{example_paper}

\end{document}


% MSJAR template is based on ICML 2019 template in order to 
% make it easier for authors to submit to MSJAR.

% This document was modified from the file originally made available by
% Pat Langley and Andrea Danyluk for ICML-2K. This version was created
% by Iain Murray in 2018, and modified by Alexandre Bouchard in
% 2019. Previous contributors include Dan Roy, Lise Getoor and Tobias
% Scheffer, which was slightly modified from the 2010 version by
% Thorsten Joachims & Johannes Fuernkranz, slightly modified from the
% 2009 version by Kiri Wagstaff and Sam Roweis's 2008 version, which is
% slightly modified from Prasad Tadepalli's 2007 version which is a
% lightly changed version of the previous year's version by Andrew
% Moore, which was in turn edited from those of Kristian Kersting and
% Codrina Lauth. Alex Smola contributed to the algorithmic style files.
