# The suitcase shuffle problem
________________
A particular flavor of the bin packing problem came up in the context of cloud computing.

- We have already fit a certain number of VMs into a certain number of nodes.
- Keeping the same number of nodes, we want to see if even more VMs can be fit by moving things around.
________
## Toy problem
230114: Christened this problem the "suitcase reshuffling problem". I have a set of suit cases with their capacities. Some items already in the suit cases. I want to move them around so that I can fit more items.

Toy example:

Suitcase-1: capacity - 13; items: (7, 5)
Suitcase-2: capacity - 11; items: (4,6)
We want to fit an item of 2 units. Currently not possible. So, swap items.

(7,6) and (5,4). Now, an item of 2 can be fit in the second suitcase.


_______________
## Question on cs.stackexchange:

There is a problem I recently encountered in my work which is related to the knapsack and bin packing problems. But I couldn't find the exact problem anywhere. 

Say you have some suitcases. Each of them has its own capacity (volume). You also have some items with their volumes, already fitted inside the suitcases. There isn't enough space to fit any additional items in any of the suitcases. But we'd still like to accommodate more items (we have an infinite number of items of each kind outside the suitcases). This can be done by shuffling the items among the suitcases until we get enough space in one of them to fit additional items. 

Let's take an example with two suitcases. The capacity of the first one is 13 and that of the second one is 11. The first one has two items with volumes 7 and 5. The second one also has two items with volumes 4 and 6. Both the suitcases have a volume of 1 remaining. 

The kinds of items available to us have volumes 7, 5, 6, 4 and 2. The total spare capacity across the two suitcases is 2. So, our only hope is to fit an additional item of size 2 (since we can't remove existing items).

We currently can't fit this item in any of the suitcases. But if we swap the positions of the items with capacities 5 and 6, the first suitcase now has no capacity and the second one has a spare capacity of 2. So we can now fit an extra item of size 2 in the second suitcase. 

![[suit_case_reshfl_toy.png | 400]]

In general, we'd like to fit as much additional volume as we can. My question is three-fold:

1) Has this problem been studied?
2) Is it possible to prove it is NP-hard?
3) What are some good ways to solve it? Can it be somehow reduced to the knapsack or bin-packing problems or their variants?

[Question on cs](https://cs.stackexchange.com/questions/156856/has-this-problem-related-to-bin-packing-and-knapsack-been-studied)
___________________
## Solving on a computer
Simulated annealing is applicable to this problem. 

## Feasible Solution
A solution is an array of arrays. One array for each suitcase. The volumes of the items currently in that suitcase are the elements of the array. The last element of each array is the "empty item" or the amount of spare space remaining in that suit case.

## Switching strategy
The switching strategy can be swapping two items that are not on the same suitcase. It it makes a suit case overflow, we can either reject that swap, or continue swapping till none of the suitcases overflow.

## Objective fn
We prefer solutions where there is more contiguous, empty space in the suitcases. A few suitcases should contain all the empty space. If we rank them descending by empty space, the distribution should look very skewed. One idea is to take the max of empty space across the suitcases as the objective function. At some point, we start trying to empty the second largest and so on.

Another objective function could be the sum of the two largest. Or the k-largest. 

_______________
# References
[1] The generalized bin packing problem, Mauro Maria Baldi, Teodor Gabriel Crainic, Guido Perboli, Roberto Tadei [[gen_bin_pck_baldi.pdf]]
[2] The stochastic generalized bin packing problem https://www.sciencedirect.com/science/article/pii/S0166218X11004719

_________
#paper #cs/complexity 
keywords: np, nphard, cs, complexity

keywords: knapsack binpack binpacking nphard np suitcase shuffle packing msft microsoft azure allocator
# Generalization
Most general problem:

- Items to fit into the bins are multi-dimensional. 
- Bins are also multi-dimensional and there are multiple types.

- [ ] How do we come up with a toy problem where the optimal solution is known?
	- [ ] Should probably frame it so that its equivalent to original bin-packing.


## Creating toy instances


# Links

See: [[KnapSk_BinPk]]
[[nphard]]
[[blogs]]
