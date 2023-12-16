# Notes
- Spun up from [[Pt_process]]
- Published to Medium.
# Eulers number, the exponential distribution and compound interest

There are many interesting events that happen at points in time. For example, the arriavals of busses at a bus stop, accidents on a highway, goals scored in a soccer (football) game. The processes that model such point in time events are called "point processes". An important consideration in these processes is how long it takes from one event to the next. For example, given you just missed a bus, how long will you have to wait for the next bus? This time is a random variable and the choice of the random variable specifies the point process. One choice for this random variable is something that isn't very random (deterministic numbers are just a special case of random ones). Busses arriving on a punctual schedule, every 10 minutes for example. This might sound like the simplest possible point process, but there is one even simpler. And it arises when the times between events follow an exponential distribution (the process is called the Poisson process). It's called the exponential distribution for good reason. It is tied to Euler's number, $e$ and compound interest. In this article, we'll see the connection.

# Compound interest
Say you deposit 1\$ in the bank. The interest rate is $x$ per year. At the end of the year, your balance will be $(1+x)$. To get more money, you ask the bank to pay you the interest monthly instead of yearly. Since the rate is $x$ per year, the interest you'll earn in a month will be $\frac{x}{12}$. And you immediately re-invest the interest. So for the second month, your investment becomes $(1+x/12)$ and this in turn grows by a factor of $(1+x/12)$ meaning the amount after $2$ months is $(1+x/12)^2$. Repeating this for $12$ months, your balance at the end of the year will be $(1+x/12)^{12}$. Using the Binomial theorem, this new balance at the end of the year is:

$$\begin{align}B = {12 \choose 0} \left(\frac{x}{12}\right)^0 + {12 \choose 1} \left(\frac{x}{12}\right)^1 + {12 \choose 2} \left(\frac{x}{12}\right)^2 + \dots + {12 \choose 12} \left(\frac{x}{12}\right)^{12}\\
= 1 + x + {12 \choose 2} \left(\frac{x}{12}\right)^2 + \dots
\end{align}
$$

We can see that this is more than the $(1+x)$ we ended up with before. This makes sense since we were getting interest throughout the months and the interest was re-invested and earning further interest on top. But why stop at $12$ intervals? You want to compound as frequently as possible. Every millisecond if the bank will allow it. Instead of $12$ intervals, we generalize to $n$ intervals and make $n$ really large. After every interval, our balance grows further by $(1+x/n)^n$. And at the end of the year the amount we'll have,

$$B = \lim_{n \to \infty}\left(1+\frac{x}{n}\right)^n \tag{1}$$
Expanding this out with the binomial theorem, 
$$\begin{align}B = \lim_{n \to \infty} \left(1+\frac{x}{n}\right)^n\\ 
 =\lim_{n \to \infty}  1 + {n \choose 1}\left(\frac x n\right) + {n \choose 2}\left(\frac x n\right)^2 + \dots\\
 =\lim_{n \to \infty} 1+ n \left(\frac x n\right) + \frac{n(n-1)}{2!}\left(\frac x n\right)^2 + \frac{n(n-1)(n-2)}{3!}\left(\frac x n\right)^3 + \dots
\end{align}$$
As $n$ becomes larger, the $n-1$, $n-2$, etc. are practically the same as $n$. So, all those terms involving $n$ cancel out between the numerators and denominators (since we have $n \to \infty$) and we're left with:

$$B(x) = 1+x+\frac{x^2}{2!}+\frac{x^3}{3!} + \dots$$
If we differentiate $B(x)$ with respect to $x$, we get $B(x)$ back. If we plug in $x=1$, we get a very special number. Can you guess? It's readily apparent from the first two terms that this number is greater than $2$. 

$$B(1) = 1+\frac{1}{1!}+\frac{1}{2!}+\frac{1}{3!} + \dots = e$$
We've just re-discovered the famous Eulers number, $e=2.71828\dots$. And it turns out, $B(x)=e^x$. This wasn't immediately obvious to me, but we can see this by going back to equation (1). 
$$\begin{align}B(x) = \lim_{n \to \infty}\left(1+\frac{x}{n}\right)^n\\
=> B(1) = e =\lim_{n \to \infty} \left(1+\frac{1}{n}\right)^n
\end{align}$$
We have $e$ in the second equation, but not in the first one. The $\frac{x}{n}$ term inside the bracket is kind of getting in the way of that. To clean it up, let's change up the variables by defining:

$$\frac 1 t = \frac{x}{n}$$
This will make equation (1):
$$\begin{align}B(x) = \lim_{t \to \infty}\left(1+\frac{1}{t}\right)^{xt}\\
= \left(\lim_{t \to \infty}\left(1+\frac{1}{t}\right)^{t}\right)^x\\
= e^x
\end{align}$$
Note that taking the $x$ outside the limit like we did above is allowed for continuous functions. 

# The exponential distribution
So that was compound interest and the motivation for the number $e$. How does all this relate to point processes and the exponential distribution? The exponential distribution works in continuous time and models the time until some event (like a car accident). 

The best way to understand it is to think of the limit of tossing coins. 

The claim to fame of the exponential distribution is the fact that it is memory less. In fact, it is the only continuous distribution that is memory less. If you're waiting for a bus whose time until arrival is exponentially distributed, then it doesn't matter how long you've waited already.
The distribution of the additional time you have to wait is exactly the same weather you just arrived or have been waiting for ten hours. This property makes the exponential distribution very easy to work with.
Its easier to understand this property when we make things discrete. Instead of waiting in continuous time for a bus to arrive, imagine tossing a coin every minute and waiting to see a heads. The number of tails we'll see before we see the first heads is a discrete random variable since it can take only non negative integer values (unlike the bus arrival time which can be any real number like 3.4 minutes). This discrete distribution is called the Geometric distribution.
A real world scenario where the Geometric distribution applies perfectly is a slot machine in a casino which a gambler keeps playing until he hits a jackpot.
Every spin of the machine is independent of the spins in the past. Which means that this Geometric distribution is also memory less.


The coin has a probability $p$ of heads. We start tossing this coin. What is the probability that we haven't seen a heads after $k$ tosses? The probability is:

$$P(X>k) = (1-p)^k$$


$$\begin{align}P(T>t) = S_T(t) = \lim_{p \to 0, d\to 0} (1-p)^{\left[\frac t d\right]}\\
 = \lim_{p \to 0, d\to 0} (1-p)^{\frac t d - \left\{\frac t d \right\}}\\
 = \lim_{p \to 0, d\to 0} (1-p)^{\frac t d} (1-p)^{- \left\{\frac t d \right\}}\\
 = \lim_{p \to 0, d\to 0} (1-p)^{\frac t d} \lim_{p \to 0, d\to 0}(1-p)^{- \left\{\frac t d \right\}}
\end{align}$$
The second limit just becomes $1$.
$$S_T(t) = \lim_{p \to 0, d\to 0} (1-p)^{\frac t d} \tag{3}$$
This limit is interesting only when $p$ and $d$ decrease to $0$ together in a linear relationship with each other. Because both are going to zero together, the line has to have an intercept of $0$. Let's say the line is:

$$d = \frac p \lambda$$
This is the step most people have trouble with. Why all of a sudden this linear equation? Where did it come from? If we ask what the double limit in equation (3) equals, the answer is going to be that "it depends". Depends on the relationship between $p$ and $d$. For one thing, we know that $p$ and $d$ are approaching $0$ together. So the relationship between them must pass through $(0,0)$. Next, we need to pick the functional form between them. And it is up to us what to pick. But if we pick anything but a linear relationship, we get a trivial answer (like $0$ or $1$) and don't get an interesting continuous distribution.

Going with the linear relationship, equation (3) above becomes:
$$S_T(t) = \lim_{d\to 0} (1-\lambda d)^{\frac t d}$$
We need another substitution to make this align with equation (1):
$$f = \frac 1 d$$
$$\begin{align}S_T(t) = \lim_{f \to \infty} \left(1+\frac {(-\lambda)}{ f}\right)^{ft}\\
 = \left(\lim_{f \to \infty} \left(1+\frac {(-\lambda)}{ f}\right)^f\right)^{t}\\
 = (e^{-\lambda})^t = e^{-\lambda t}
\end{align}$$
Which is the survival function of the exponential distrubution.

# References
[1] https://www.cantorsparadise.com/the-history-of-eulers-number-e-8c982994a39b

#math/prob #math/prob/exponential #math 