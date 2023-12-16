# I) Introduction
Imagine a program that takes some input, performs some computation in the most efficient way possible and produces an output. We generally care about the amount of computation the program has to perform as a function of how large the inputs are. If the amount of computation grows in a way that some polynomial function will always be larger than it, the problem is considered "easy". And if it grows so fast that it out paces any polynomial function imaginable, the problem is "hard". And then there are problems which not only can we not solve efficiently, but if an oracle gave the solution to us, we couldn't even tell if it was the right solution efficiently. These kinds of problems is called "NP-hard". Some optimization problems typically fall in this category. An example is finding the best move in a given position of a chess game. If someone just told you what it was, there would be no way to verify it.

It's important to be able to tell weather a problem is NP-hard. Then, we don't have to worry about trying to find an optimal solution because its futile and be content with heuristics that find "good" solutions.

Most problems one would encounter in the wild should be NP-hard. Also, there is a phenomenon where problem that arise in industry settings "grow up". When they first gets formulated, they often maps to some nice, famous problem with an efficient solution. But as time goes on, more and more factors must be taken into account to keep up with business needs, making the problem more complicated, more ugly. Its often the case that a problem starts off as "easy" problem and then graduates to a "hard" and then "NP-hard" problem. In this blog, we'll go through this evolution with a problem that has broad applicability in software testing. 

# II) The combinatorial testing problem
Many software products are meant to run on not just one, but a diverse set of environments. A website runs on many browsers, a software library like Numpy or Scipy runs on many operating systems and Python environments, software agents that run on the machines in a cloud system will need to work well on the various hardware’s and other configurations found in the cloud fleet they serve, operating systems like windows and mac-os are deployed to a diverse set of machines from laptops to desktops, etc.

We want to design experiments where new changes (like new software builds, hardware models, features, etc.) are A-B tested against the appropriate last known good (LKG) versions. It's important that these bits are exposed to as much production diversity as possible so there are no surprises in production and real customers are protected from any negative impact these changes might have (by catching and fixing issues in the pre-prod environment). Ideally, we'd like to cover all valid combinations of the important properties (or at least the ones seen in production). But, our pre-prod test environment is typically very small compared to the production fleet. So, we revise our ambitions down by insisting only that all levels of the various properties are covered. The properties have various feasibility constraints that can be expressed in terms of graphs.

## II-A) Azure architecture
For the purpose of this blog, we need to know that there are node-level properties and VM-level properties we need to account for. The nodes are basically servers that are spread across data-centers in various regions. Our customers (in Azure compute) rent virtual machines (VMs). These run on the nodes and there can be more than one running per node, each with its own partition of the resources like memory, disk, etc. The VMs are completely isolated from each other and also the host. This is visualized in the picture below where the larger rectangle represents the node and the smaller colored shapes inside it represent virtual machines. Virtual machines come in various types (represented by shapes and colors) and multiple customers could be running the same kind of virtual machine on the node while some run other kinds.
![[VMsOnNode.png#invert | 300]]
> Figure 1: The blue rectangle represents the node or server in Azure, a physical concept. The colored shapes inside represent virtual machines, a virtual concept.
# III) The evolution of the problem
## III-A) Cover two properties
At this point, the problem is an infant. We wanted to cover just two properties, the hardware model of the node and the virtual machine type. Further, there was a constraint in our environment where the node could not run different types of VMs. It could run multiple VMs, but they all had to be the same type. We'll call this the homegenity constraint. Also, not all hardware's are compatible with every possible virtual machine type. For example, a certain model of Dell hardware might only be able to run A-series and C-series VMs. These compatibility constraints can be encoded in a graph where we list all the hardware's on the left and all the virtual machine types on the right. Then, we connect a hardware and virtual machine type if the two are compatible with an edge. A toy example is shown below.


![[hw_vm_props.png#invert | 300]]
> Figure 2: A bipartite graph with two properties, hardware model on the left and VM type on the right

Under these conditions, we wanted to cover all the hardware models and all the virtual machine types in the smallest possible number of nodes. Since a node can have only one hardware, and per the homogenity constraint only one kind of VM type can run on it, picking an edge in the compatibility graph we drew above completely specifies a node in our experiment design. So to minimize the number of nodes such that all hardware models and virtual machine types are covered, we need a minimal set of edges in the graph that cover all the nodes. This is the famous min-edge-cover problem (see section I-A [here](https://medium.com/experience-stack/using-graph-theory-to-design-experiments-145f24875281)). And there are many efficient algorithms for solving it, it is firmly in the "easy" bucket.

## III-B) Cover n properties (bipartite graph)
The next layer of complexity was, predictably, adding more properties. But without losing a key property of the graph. That key property is that the graph was [bi-partite](https://en.wikipedia.org/wiki/Bipartite_graph). This means that the vertices of the graph can be divided into two sets such that all edges go only between those two sets. The graph in figure 2 was an obvious example. The hardware models on the left were the first set and the virtual machine types on the right were the second set. It's possible to add more properties beyond hardware model and VM-type while maintaining this property. For example, let's say we add a third property called OS-image (the operating system running within the VM, say Linux, Windows server, etc.). And there are constraints between the OS image and the virtual machine type, also encoded in the graph shown below in figure-3.
![[ThreeProperties.png#invert | 300]]
> Figure 3: We added a third property, the OS image of the VM. And there are some constraints in terms of which images are compatible with which VM-types. But the graph is still bipartite.

While it might not be immediately obvious at first glance, the graph in figure-3 is also bipartite. We can think of the two sets we want to divide the vertices into as colors. So, set-1 is the color red and set-2 is the color blue. It is possible to paint all the hardware's (left-most layer) blue, all the VM-types (middle layer) red and then all the OS-images (right most layer) blue again. In this way, we can add $n$ layers or properties (instead of just 3). But as long as new edges are introduced between successive layers like we did above (no layers between OS image and hardware were introduced, only to the previous layer, VM-type), the graph remains bi-partite. For covering all the properties (all hardwares, VM-SKUs, OS-images) in the smallest number of nodes, we now observe that a combination of the three (Dell, A-series and Linux for example) specify a node. And this can be thought of as a path in the graph from the left-most layer to the right-most layer. Which enables us to invoke yet another well known algorithm, the min-path cover (though some adjustments are needed to the well known version). For details, see section II [here](https://medium.com/experience-stack/using-graph-theory-to-design-experiments-145f24875281). The problem still maps to a famous "easy" problem, although the connection is harder to see this time. We can think of the problem now as a baby.

## III-C) Cover n properties (k-partite graph)
Alas, all good things must come to an end. And the nice property that allowed us to keep our graph bipartite was doomed from the time it was first recognized. We soon started getting properties that were not connected to just the previous property (in terms of constraints), but multiple properties. This is shown in the figure below. We still have three properties, but the graph isn't bi-partite anymore because of the connections between the third and first layer (OS-images with hardwares).

![[ThreePropertiesNonBipartite.png#invert | 300]]
> Figure 4: We still have three properties: hardware, VM type and OS image. But now there are connections (constraints) between any two properties. OS-image is no longer connected to just VM-type but also to hardware mode.

What will completely specify a valid node in our experiment now is not a path, but a clique. For a node to be assigned a hardware, a VM type to run and an OS-image on that VM, all three need to be compatible with each other. In other words, they need to be connected in the graph. And a set of vertices such that any two are connected in a graph is called a "clique" (think of a group of friends in high school where any two in the group are friends with each other). So, the problem goes from a min-path cover to a [min-clique cover](https://en.wikipedia.org/wiki/Clique_cover). And this is known to be an NP-hard problem. Our problem has now graduated and is a difficult teenager now. 
## III-D) Constrained optimization problem (homogeneous nodes)
There are all kinds of other requirements that keep coming in for our experiment design service. So far, we have been minimizing the number of nodes in our experiment. This is for certain kinds of shorter experiments. Therer are other kinds of experiments where we're given the number of nodes we can use (say 300) up-front. Similar to before, we still have some properties and a graph representing the compatibility constraints between the properties (like in figure-4). We still would like to cover all the properties. But now, we'd like the precentage representation of different hardware's to be as close as possible to some target distribution (ex: we want 30% of the nodes to be HP, Dell to be 50% and Microsoft to be 20%). This target distribution might have come from the representations in the fleet, for example. And we might have similar target distributions for the VM-types and OS-images. We then get a constrained optimization problem:

>Objective fn:
>	Minimize distance between target and actual distributions.
>Constraints:
>    All properties in the graph should be covered.
>    All n available testing servers should be utilized.
>    The graph constraints should be respected.

This constrained optimization problem is also NP-hard. Also, it no longer maps to some famous problem. It is a unique beast. We can think of our problem having reached adult-hood now.

# IV) One algorithm to rule them all: simulated annealing
In the previous section, we saw how the problem of designing experiments for testing changes quickly went from a cute problem that can be solved with famous algorithms in graph theory to NP-hard as layers of complexity were added and finally something that can't even be mapped to a famous problem. And this is what the evolution trajectory of most practical problems looks like. Wouldn't it be nice if we had a methodology that could give us good solutions for all of these problems? Maybe not optimal ones, but good ones.

There are many such heuristics with broad applicability across problems. One of them is simulated annealing. To apply it to a problem, all we need is that we should be able to generate a feasible solution (not optimal) to the problem and we should have some way to switch from one solution to a neighboring solution. The smaller the changes this "switching strategy" makes, the better (akin to evolution where small mutations pair with survival of the fittest).

The methods for simulated annealing exposed in libraries like R and Python-SciPy did not apply to combinatorial problems like the ones described here, only to problems over continuous variables. So, we created a [new library](https://github.com/ryu577/optimizn) that can apply this and other famous heuristics like branch and bound to any combinatorial optimization problem. Further, since a lot of these problems are NP-hard meaning we can't even tell when an optimal solution is found, we added a "continuous training" functionality to the library where it keeps searching for the optimal solution and persists state across runs so it can pick up where it left off. But that is a topic for another blog (watch this space).

_______________________
#building/blog #cs/complexity 
keywords: np, nphard, cs, complexity, blog
[[nphard]]

Fishers geometric model.

> Keywords: blog nphard np pnp azure envdes experimentdesign experiment_design environment

