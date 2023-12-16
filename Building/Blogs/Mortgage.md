$a$: Principal remaining.
$t$: Number of periods.
$u$: Due per period.

| Interest | Principal | Total |  
| -------- | -------- | -------- |  
| $ar$ | $u-ar$ | $u$ |  
| $(a-(u-ar))r$ | $u-(a-(u-ar))r = (u-ar)(1+r)$ | $u$ |


Principal paid in period $i$ becomes: $(u-ar)(1+r)^{i-1}$
$$\sum\limits_{i=1}^t (u-ar)(1+r)^{i-1} = a$$
$$(u-ar)\left(\frac{(1+r)^t-1}{r}\right) = a$$
# If t is fixed
$$u-ar = \frac{ar}{(1+r)^t-1}$$
$$u = ar\left(\frac{(1+r)^t}{(1+r)^t-1}\right) = \frac{ar}{1-(1+r)^{-t}}$$
# If u is fixed
$$(a+r)^t-1 = \frac{ar}{u-ar}$$
$$(1+r)^t = \frac{u}{u-ar}$$
$$t = \log_{1+r}\left(\frac{u}{u-ar}\right)$$
