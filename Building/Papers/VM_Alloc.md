# Problem
We're given a bunch of VMSKUs. Each of them has a certain number of cores and a reliability of the request (probability that a request for that VMSKU will succeed). There is also the price of each VMSKU. This is shown for three VMSKUs below.

| VMSKU | Cores | Cost | Reliability |
| ---- | ---- | ---- | ---- |
| $v_1$ | $c_1$ | $m_1$ | $p_1$ |
| $v_2$ | $c_2$ | $m_2$ | $p_2$ |
| $v_3$ | $c_3$ | $m_3$ | $p_3$ |
> Table-1: The different VMSKUs and their characteristics.

The customer wants to create a cluster of machines. She wants the cluster to have some amount of total resources. For now, let's keep it simple and assume she only cares about the total number of cores. She needs a total of $c$ cores. Further, her budget is $m$. We provision the VMs on her behalf in a way as to satisfy her requirements. How many VMs of each type should we provision?

# The levers
We can choose any number of VMs of each type. Say we try to provision $n_1$ VMs of type-1, $n_2$ of type-2 and $n_3$ of type-3. Because each request we make has some probability of failure, the actual number of VMSKUs of type-1 will be a random number. Let's call this $N_1$ (random numbers are typically capitalized). Note that we must have $N_1 \leq n_1$ and so on. We can now add those two new columns to table-1 above. Let's add these two new variables to table-1.

| VMSKU | Cores | Cost | Reliability | NumAllocated | NumRealized |
| ---- | ---- | ---- | ---- | ---- | ---- |
| $v_1$ | $c_1$ | $m_1$ | $p_1$ | $n_1$ | $N_1$ |
| $v_2$ | $c_2$ | $m_2$ | $p_2$ | $n_2$ | $N_2$ |
| $v_3$ | $c_3$ | $m_3$ | $p_3$ | $n_3$ | $N_3$ |
> Table-2: The different VMSKUs along with their characteristics and the levers we can control. We can choose $n_1$, $n_2$ and $n_3$.
>Note: The tables above are shown for $3$ VMSKUs for demonstration. This can easily be extended to $k$ VMSKUs.

# Math Formulation
It's easy to see that the $N_i$ are binomial random variables with parameters $(n_i, p_i)$. We mentioned that the budget we have to work with is $m$. We want to guarantee that we won't cross that budget no matter what. At most, we'll get everything we asked for, which is $n_1$ type-1 VMs and so on. This gives us the budget constraint (assume $k$ VMSKUs):

$$n_1 m_1 +n_2 m_2 + \dots n_k m_k \leq m \tag{1}$$
For the case of 3 VMSKUs, this can be visualized as in figure-1 below.
![[Screenshot 2024-03-10 at 8.56.11 PM.png#invert|300]]
> Figure-1

Further, the actual number of VMSKUs of type-1 we'll get is $N_1$ and so the number of cores: $N_1 c_1$. So, the total number of cores we'll end up with (a random number): $N_1 c_1+N_2 c_2+\dots N_k c_k$. The customer wanted $c$ total cores in their cluster. We want to minimize the probability that this requirement won't be achieved. So, our objective function is to minimize:
$$P(N_1 c_1+N_2 c_2+\dots N_k c_k < c) \tag{2}$$
Again, we can visualize this condition on cores for the case of three VMSKUs.
![[Screenshot 2024-03-10 at 8.57.52 PM.png#invert]]
Hence, we have a constrained optimization problem with equation (2) as the objective function and equation (1) as the constraint.
# Solving the problem
We can solve this constrained optimization problem in a brute force manner. 
1) Identify all points, $n_1, n_2, \dots n_k$ that satisfy equation (1). Store all such tuples in an array (array-1).
2) For each tuple in array-1 above, find the probability in equation (2).
3) Pick the tuple from array-1 that has the lowest probability. This represents the number of VMSKUs of each type we initialize.
This algorithm is implemented in Python and the code included in appendix-1. This code can be run directly. In lines 57 through 61 (right after the
```python
if __name__ == "__main__"
```
block), the parameters from table-1 can be changed and the optimal tuple as well as the corresponding minimal probability is printed out.

# Experiments
## Experiment 1
Say we have 4 VMSKUs. The table below shows the various parameters.

| VMSKU | Cores | Cost | Reliability |
| ---- | ---- | ---- | ---- |
| $v_1$ | $1$ | $1.2$ | $.99$ |
| $v_2$ | $2$ | $1.9$ | $.8$ |
| $v_3$ | $3$ | $2.7$ | $.77$ |
| $v_4$ | $2$ | $2.2$ | $.84$ |
Also, the number of cores required is $c=8$ and our budget is $m=10.4$.
The optimal allocation is $7$ VMs of type-1, $1$ VM of type-2 and $0$ each of types 3 and 4. The probability of failure is $0.254$.
We can see that the most money we can spend is $1.2 \times 7+1.9 = 10.3$ which is just below our budget. Since the probability of success for the first VMSKU was so high at $.99$, it makes sense that the model doubled down on that one, asking for $7$ of them. 
## Experiment 2
Now, let's make that probability slightly lower at $p_1 = 0.9$. 
Now the optimal provisioning becomes requesting $4$ VMs of type-2 and $1$ VM of type-3. The probability of failure now becomes $0.37$. Since the reliability of the first VMSKU was lowered and its cost per core is high, the model moved away from it.
## Experiment 3
Finally, let's increase our budget. Instead of $m=10.4$, let's make it $m=13$. Our optimal allocation now becomes $n_1=1,n_2=5,n_3=0,n_4=1$ and the probability of failure to get $c=8$ cores is now $0.114$.

# Efficiency considerations
The code in appendix-1 as it stands isn't very efficient and doesn't scale well with the number of VMSKUs (see [4](https://math.stackexchange.com/questions/4878197/estimating-the-number-of-integer-tuples-that-satisfy-a-linear-inequality/4878678#4878678)). This can easily be rectified. For calculating the objective function, we don't have to loop over all points that satisfy equation (2). Instead, we can approximate the multivariate Binomial with a multivariate Gaussian and calculate its probability mass beyond the hyper-plane. This is discussed in [2](https://math.stackexchange.com/questions/4878751/area-of-a-multivariate-gaussian-beyond-a-hyperplane). Further, we don't need to evaluate all the points that satisfy equation (1). It can be shown that it one tuple of $n_i$'s is greater than or equal to another tuple for every $i$, it's guaranteed to have a better objective function (given by (2)). So, we can eliminate a lot of points from consideration this way.

Further, notice that the code uses a depth first approach on a conceptual tree that builds the tuples. This tree can be actively pruned according to the criterion of the branch and bound algorithm, [6](https://en.wikipedia.org/wiki/Branch_and_bound#:~:text=A%20best%2Dfirst%20branch%20and,and%20its%20descendant%20A*%20search.) to get us a much more efficient heuristic.

# References
[1](https://math.stackexchange.com/questions/4878096/approximating-the-probability-that-a-linear-combination-of-binomials-will-be-les?noredirect=1#comment10404452_4878096) Approximate probability for a linear combination of Binomials.
[2](https://math.stackexchange.com/questions/4878751/area-of-a-multivariate-gaussian-beyond-a-hyperplane?noredirect=1#comment10404450_4878751) Multivariate Gaussian beyond hyperplane.
[3](https://cs.stackexchange.com/questions/167010/regular-branch-and-bound-vs-integer-programming-branch-and-bound) Various flavors of branch and bound.
[4](https://math.stackexchange.com/questions/4878197/estimating-the-number-of-integer-tuples-that-satisfy-a-linear-inequality/4878678#4878678) Estimating linear tuples that satisfy linear inequality.
[5](http://web.tecnico.ulisboa.pt/mcasquilho/compute/_linpro/TaylorB_module_c.pdf) Branch and bound for integer programming, "Integer programming the branch and bound method".
[6](https://en.wikipedia.org/wiki/Branch_and_bound#:~:text=A%20best%2Dfirst%20branch%20and,and%20its%20descendant%20A*%20search.) Wikipedia article on Branch and bound.

# Appendix-1
```python
import numpy as np
from scipy.stats import binom


class VMAlloc():
    def __init__(self, m, m1):
        self.m = m
        self.m1 = m1
        self.n = len(m)
        self.constr_sum = 0
        self.maxs = []
        for i in range(self.n):
            self.maxs.append(int(m1/m[i]))
        self.pts = []

    def explore(self, lvl=0, arr=[]):
        if lvl >= self.n:
            print(arr)
            return
        for i in range(int(self.m1/self.m[lvl])+1):
            arr.append(i)
            self.explore(lvl+1, arr)
            arr.pop()

    def explore2(self, lvl=0, arr=[]):
        if self.constr_sum > self.m1:
            return
        elif lvl >= self.n:
            print(arr)
            self.pts.append(np.copy(arr))
            return
        for i in range(int(self.m1/self.m[lvl])+1):
            arr.append(i)
            self.constr_sum += i*self.m[lvl]
            self.explore2(lvl+1, arr)
            arr.pop()
            self.constr_sum -= i*self.m[lvl]


def binom_pmf(a=np.array([1, 1, 1]),
              maxs=np.array([3, 3, 3]),
              ps=np.array([.2, .3, .4])):
    prod1 = 1
    for i in range(len(a)):
        prod1 *= binom.pmf(a[i], maxs[i], ps[i])
    return prod1


def binom_sf(as1, maxs, ps):
    sum1 = 0
    for a in as1:
        sum1 += binom_pmf(a, maxs, ps)
    return sum1


if __name__ == "__main__":
    m = np.array([1.2, 1.9, 2.7, 2.2])
    m1 = 10.4
    c = np.array([1, 2, 3, 2])
    c1 = 8
    probs = np.array([.99, .8, .77, .84])

    vma = VMAlloc(m, m1)
    vma.explore2()

    print("################")
    vma1 = VMAlloc(c, c1)
    vma1.explore2()

    # Now get the best tuple of (n1, n2, n3).
    minn = 1
    winner = []
    for a1 in vma.pts:
        sum1 = binom_sf(vma1.pts, a1, probs)
        if sum1 < minn:
            minn = sum1
            winner = a1

    print("winner:")
    print(winner)
    print(minn)

```


