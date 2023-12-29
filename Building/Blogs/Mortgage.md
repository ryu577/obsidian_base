keywords: mortgage bank of america interest loan

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
$$u = ar\left(\frac{(1+r)^t}{(1+r)^t-1}\right) = \frac{ar}{1-(1+r)^{-t}}\tag{1}$$
# If u is fixed
$$(a+r)^t-1 = \frac{ar}{u-ar}$$
$$(1+r)^t = \frac{u}{u-ar}$$
$$t = \log_{1+r}\left(\frac{u}{u-ar}\right)$$
# Frequent compounding
If we compound $x$ times:
$$\left(\frac{u}{x}-\frac{ar}{x}\right)\left(\frac{\left(1+\frac{r}{x}\right)^{t_1}-1}{\frac{r}{x}}\right)$$
$u$ is given by equation (1)

$$\left(1+\frac{r}{x}\right)^{t_1}=\frac{u}{u-ar}$$
$$t_1 = \log_{\left(1+\frac{r}{x}\right)}\left(\frac{u}{u-ar}\right)$$
$$=\log_{\left(1+\frac{r}{x}\right)}\left(\frac{\left(\frac{ar}{1-(1+r)^{-t}}\right)}{\frac{ar}{1-(1+r)^{-t}}-ar}\right)$$

- [ ] Share this to r/investing on reddit once done.
# Bard output
1. Making informed decisions: Mortgages involve complex calculations like interest rates, loan terms, and monthly payments. Grasping these concepts empowers you to compare different loan options, understand the true cost of borrowing, and choose the mortgage that best suits your financial situation. Imagine it like navigating a financial jungle — proper math skills are your machete, helping you blaze a path towards the most advantageous loan.

2. Avoiding financial pitfalls: Blindly trusting loan officers or online calculators can be risky. By understanding the math, you can independently verify calculations, identify hidden fees, and avoid predatory lending practices. It’s like having a financial X-ray vision, allowing you to see through the fine print and make informed decisions that protect your long-term financial health.

3. Building financial resilience: Mortgages are long-term commitments. Knowing how interest accrues, how principal payments affect your balance, and how unexpected events like rising interest rates can impact your payments empowers you to plan ahead, adjust your budget accordingly, and weather financial storms with greater confidence. It’s like building a sturdy financial house — the math lays the foundation for a secure and stable future.