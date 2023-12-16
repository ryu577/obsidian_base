# Why FFT only works for highly composite inputs

Many people consider complex numbers an esoteric concept, showing up in the depths of mathematics and removed from practical applications. 

Now, consider that the most influential algorithm in computer vision is convolutional neural networks. So, an algorithm that performs convolutions as efficiently as possible is important. And in order to develop the most efficient algorithm, you need to invoke complex numbers. Via the fast fourier transform (FFT). 

The foothold complex numbers have on the field of algorithm development is mostly via the FFT algorithm, which has unreasonable influence to the extent that some call it the most important algorithm of the century ex: [Veretasium]. We'll spend a little time motivating it and then explore an annoying peculiarity it has, getting an appreciation of its workings in the process.

# I) Motivation: polynomials and convolution
Before we obsess about performing a fourier transform fast, let's get an appreciation for why we'd even care. 

Chapter 30 of the book "Introduction to algorithms" by Cormen et.al. [clr_text_book] is titled "Polynomials and the FFT". To get an appreciation of the FFT from an algorithmic perspective, we need to start with polynomials. A one dimensional polynomial is a function in $x$ of the form:

$$f(x) = a_0 +a_1x+a_2 x^2+a_3 x^3+\dots + a_{n-1} x^{n-1}$$
The equation above can lead to lines, parabolas, ellipses, etc. Since the highest power of $x$ is $n-1$, this would be an $n-1$ degree polynomial if $a_{n-1} \neq 0$. Now, how do we represent a one dimensional polynomial in the memory of a computer? The simplest idea would be to take the coefficients, $a_0, a_1, \dots a_{n-1}$ and put them in an array since the coefficients completely specify the polynomial. This is called the coefficient representation. In an object oriented programming paradigm, we could write a class whose instances (objects) are defined by the coefficient array. Now that we've expressed polynomials in a computer, we want to be able to do arithmetic operations on them. Addition is quite simple. You just add the coefficient vectors together. If the arrays corresponding to one of the polynomials is larger than the other, no problem. Make them the same size by padding zeros to the shorter one. Subtraction is really just addition once you negate all coefficients of the second polynomial. Let's ignore division because dividing two polynomials leads most of the time to a creature that isn't even a polynomial. Mutiplication is very interesting (neither trivial nor impossible) and we'll motivate the FFT with this.

Let's say we want to multiply two polynomials - the first polynomial given by the coefficients: $a_0, a_1, a_2$ and the second one by: $b_0, b_1, b_2$.  Multiplying them together produces the polynomial:

$$
\begin{align}
m(x) = (a_0+a_1x+a_2x^2)(b_0+b_1x+b_2x^2)\\
= a_0b_0+(a_0b_1+b_0a_1)x+(a_0b_2+a_1b_1+a_2b_1)x^2\\
+(a_1b_2+a_2b_1)x^3+a_2b_2x^4\\
= c_0+c_1c+c_2x^2+c_3x^3+c_4x_4
\end{align}
$$
This resulting polynomial has coefficients $c_0, c_1, c_2, c_3, c_4$ where:

$$
\begin{align}
c_0=a_0b_0\\
c_1 = a_0b_1+b_0a_1\\
c_2 = a_0b_2+a_1b_1+a_2b_0\\
c_3 = a_1b_2+a_2b_1\\
c_4=a_2b_2
\end{align}\tag{1}
$$
The equations above seem messy, but they follow simply from the distributive property of multiplication. This means each term in the first polynomial is multiplied to every term in the second polynomial, which suggests a double for-loop, one loop over the terms of the first polynomial and one over the second one. 

This is implemented in the Python code below that takes two arrays representing the coefficients of the polynomials to be multiplied as input and produces the coefficient array of the multiplied polynomial as output. 

[Github gist](https://gist.github.com/ryu577/df0c062c84ca56c8d67fe375c2dcc637) and [code](https://github.com/ryu577/algorithms/blob/master/algorith/polynoms/mult.py)
```python
def polynom_mult(a, b):
	"""
	Multiplies two polynomials given in coefficient form.
	The input arrays a and b represent the coefficients.
	"""
	# The output polynomial initialized to
	# an array of zeros.
	c = [0]*(len(a)+len(b)-1)
	
	# This double loop is the distributive law
	# of multiplication.
	# Every combination of elements of the two
	# arrays get multiplied.
	for i in range(len(b)):
		for j in range(len(a)):
			c[i+j] += a[i]*b[j]
	return c
```
The function above is nice and simple, just two loops. If the first polynomial is of degree $n$ (meaning array $a$ has $n+1$ elements including the constant) and the second of degree $m$, the number of computations (multiplications, additions, etc.) the algorithm performs would roughly be $c_1+c_2. (n \times m)$ for some constants $c_1$ and $c_2$ (because of the two loops). Another way of saying this is that the time complexity of the algoritm is $O(n m)$ (since the time taken on any computer by it will be proportional to the number of computations performed).

For the special case $n=m$, this becomes $O(n^2)$, meaning the running time scales quadratically with the size of the inputs. To keep things simple, we will assume this special case going forward, but we can easily generalize to $n$ and $m$ in section III.

Note that the Python code above simply takes two arrays as inputs and produces a single array as output. There is no explicit context on polynomials in the code itself. And indeed, there is a name for the operation that takes the vectors $[a_0, a_1, a_2]$ and $[b_0, b_1, b_2]$ as inputs and produces the vector $[c_0, c_1, c_2, c_3, c_4]$ as output outside the realm of polynomials. It's called a 'convolution'. Apart from being equivalent to polynomial multiplication, it is used in probability theory for determining the distribution of the sum of two random variables and is also a fundamental operation in convolutional neural networks, the current cutting edge model in computer vision.

To recap, with the coefficient representation of a polynomial, we could perform addition and subtraction trivially, but multiplication was a bit harder. Is there some other way of expressing a polynomial in computer memory that would make multiplication easy?

Indeed, there is. If we took any $n$ inputs to the polynomial, $x_0, x_1, \dots x_{n-1}$, and evaluated the polynomial at each of them (getting $y_0, y_1, \dots y_{n-1}$), then the pairs: 

$$(x_0,y_0), (x_1, y_1), \dots (x_{n-1},y_{n-1})$$ 
would also completely specify the polynomial (called the point-value representation for obvious reasons). This is because plugging each of those $x_i$ values into the polynomial and equating it to $y_i$ produces one equation in the $n$ coefficients, $a_0, a_1, \dots a_{n-1}$ (which are unknown here). Since there are $n$ equations and $n$ unkonwns, we can retreive all the coefficients from these equations, hence fully specifying the polynomial.

If we're given two polynomials in this representation (both with degree $n$ and with the same $x_i$'s'), multiplying them becomes trivial. If the first polynomial is $y_0$ at $x_0$ and the second polynomial is $y_0'$ at $x_0$, then what is the value of the polynomial that is their product at $x_0$? Its simply $y_0 y_0'$. Similarly, we can get the value of the product at all $n$ $x_i$'s. 

But there is a problem with this picture. The polynomial we get from multiplying the two $n$ degree polynomials is of degree $2n$ (because the $x^n$ terms of both will get multiplied to yield a degree $2n$ term). So, we need $2n$ point-values. But if we stored just $n$ for both of the input polynomials, we don't have enough terms for their product, which requires $2n$ values.

What we need is to be able to get the point-value form efficiently from the coefficient form for any number of points we'd like. Then, we can just generate $2n$ point-values for both polynomials and multiply the $y_i$'s of both polynomials point-wise for each $i$ to get the point-value representation of the product polynomial. 

Let's say we're in the coefficient form of a polynomial and want to get the point-value form at the points: $x_0, x_1, \dots x_{n-1}$. So, we want to find:
$$
\begin{align}
y_0 = a_0 + a_1x_0+a_2x_0^2+\dots +a_nx_0^{n-1} \\
y_1 = a_0 + a_1x_1+a_2x_1^2+\dots +a_nx_1^{n-1} \\
\vdots\\
y_{n-1} = a_0 + a_1x_{n-1}+a_2x_{n-1}^2+\dots +a_{n-1}x_{n-1}^{n-1}
\end{align}
$$
So, we have about $n^2$ multiplications between the coefficients and the powers of $x_i$'s to be performed. Meaning, this algorithms run time will scale as $O(n^2)$. Once we get this point-value representation for both polynomials, doing the actual polynomial multiplication is just another $n$ multiplications which don't change the complexity, $O(n^2)$.

Since doing the convolution directly using the Python algorithm from earlier was also an $O(n^2)$ operation, this moving to the point-value representation trick hasn't done much for us so far.

But there is another card we can play. The set of $x_i$'s can be chosen in a way that the moving to the point-value representation becomes much more efficient. The algorithm that works with the special $x_i$ values that give us a big performance boost is the Fast Fourier Transform (FFT). We'll delve into this algorithm in the next section. As as a teaser, those special $x_i$ values are complex numbers. After that, we'll pick polynomial multiplication back up.

To recap this section, we described one dimensional polynomials (in only one variable, $x$), considered storing them on a computer and how multiplying them was equivalent to one-dimensional convolution of their coefficients. The dominant computer vision algorithm for a while has been convolutional neural networks. And the fundamental operation there is thinking of the image as a two dimensional array of pixel values and doing a convolution with another, smaller two dimensional array (called the kernel). This convolution operation on the image is useful for purposes besides machine learning like detecting edges, blurring, etc. This is covered very well in the video by 3 Blue 1 Brown [3b1b_convolution].

The image can actually be thought of as a polynomial in two variables ($x$ and $y$) and so can the kernel. The convolution operation is then simply multiplying the polynomial corresponding to the image with that corresponding to the kernel. But that is a topic for another day.

# II) The algorithm in the book
In the previous section, we described how our goal was to convert a polynomial represented as an array of coefficients to point-value representation efficiently. The straightforward method involved simply evaluating the polynomial at all the points, and the run time of this scaled as $O(n^2)$ with the number of coefficients. This was for a general set of $x_i$'s however and we promised there would be some set of $x_i$'s for which it would become more efficient. And those special values are the $n$th roots of unity. The solutions to the equation:
$$x^n = 1$$
The algorithm in chapter 30 of the [clr_text_book] does this, but with the caveat that the input array *must* be of a size that is a power of $2$. This is a bit of a bummer because most inputs you'd encounter in the real world you'd expect to *not* be powers of $2$. 

They say that algorithms that can handle non powers of 2 sized arrays exist, but are beyond the scope of the book. Then, in the exercises, they ask about a corresponding algorithm that would work on powers of $3$ instead (but don't ask for an implementation). And this gives a hint as to how to get to a more general algorithm. For the time being, we'll assume that $n$ is a power of $2$ and describe the algorithm in the book with some pictures in this section (this section should be considered a supplement to the book itself). Then, we will describe what prevents us from passing non-power of $2$ inputs to the algorithm. Finally, we'll generalize to an algorithm that works on inputs that are sized as powers of 3 and describe how a bunch of such algorithms can be combined into a more general algorithm that can work on any input that is a highly composite number. For most applications of FFT, we have some freedom in choosing the size of the input. And since we can always find a highly composite number very close to any number, this suffices.
## II-A) A crash course in the roots of unity
If you're already familiar with the $n$-th roots of units, you can skip ahead to the sub-section on the halving lemma.

### The number i
There is no real number such that multiplying with itself will result in $-1$. At first, people were content to say that the concept of square roots is simply not applicable to $-1$ and other negative numbers. But even while trying to find even the *real* roots of general cubic equations, the square root of $-1$ kept popping up (it turns out, it would cancel out eventually to produce a real number). And people eventually said "if the square root of $-1$ doesn't exist on the real number line, doesn't mean it doesn't exist anywhere". They created a new number, $i$ explicitly defined to satisfy the property that multiplying with itself would lead to $-1$. And since it was something that existed outside the domain of "real" numbers, it was called an "imaginary" number. The truth is that all numbers (real or complex) are imaginary constructs we invented to understand the world.

### The complex plane
Once you have the new number $i$, you can multiply it with $2$ and get a new number, $2i$. In fact, for every real number, we can multiply it with $i$ and get a whole new number line made of complex numberes. We can put the original number line on the x-axis of a 2-d plane and the new complex numbers on the y-axis. This special kind of 2-d plane is called the "complex plane". Note that multiplying $0$ with $i$ leads to $0$ just like with any other number. So, $0$ is where the two lines intersect. A point that has coordinates $(x,y)$ on this 2-d plane can be written as: $x+iy$ as shown.

![[cmplx_plane.png#invert | 400]]

### The unit circle
While the complex plane is fascinating in its entirity, we'll focus only on the points that lie on the unit circle, centered at $0$. The distance to each point from the origin (also the center of the circle) is $1$. Any point on the circle (for example the red point in the figure below) is completely specified by the angle, $\theta$ it makes with the x-axis. It can be seen that the green and red points are a part of a right angled triangle with hypotenuse $1$ (the radius of the circle). So, the x-coordinate of the red point is the base of that right triangle, $\cos(\theta)$ and the y-coordinate is the perpendicular of the right triangle which is $\sin(\theta)$. So, the point itself is represented by the complex number: $\cos(\theta)+i\sin(\theta)$. And by Euler's famous identity, this is nothing but $e^{i\theta}$. 

![[unit_circle2.png#invert | 500]]
So we've established that any point on the circle is specified by $e^{i \theta}$. Now, what happens when we raise this number to the power $n$?

$$(e^{i\theta})^n = e^{in\theta} \tag{2}$$
We get a new point on the same circle described by the angle, $n\theta$.

### The roots of unity
Now, let's go back to the problem of finding the $n$-th roots of unity. The values that satisfy the equation:

$$x^n=1$$
Let's start with the right hand side. It's just a $1$. But we want to speak only in terms of points on the unit circle (of the form $e^{i\theta}$)?  What values of $\theta$ lead to $1$? Of course, $\theta=0$ satisfies this requirement since any number raise to the power of $0$ is $1$ and $i$ multiplied by $0$ is $0$. So, $x=e^{i 0} = 1$ is always a solution. But there are more.

Now, imagine starting at $\theta=0$ and gradually increasing $\theta$. As we do this, we move around the circle. 
![[around_circle.gif#invert | 300]]

What happens at θ = 2𝜋? We complete one full revolution around it and find ourselves back at $1$. 
So, $e^{i 2\pi}$ is also equal to $1$. As we keep increasing θ beyond this, we take another loop and at θ = 2𝜋+2𝜋=4𝜋, we end up back at $1$. The pattern here is that all integer multiples of 2𝜋 end up at $1$. So,

$$e^{2ik\pi}=1\;\; \forall \; k \in \Bbb{Z}$$
the equation we wanted to solve becomes:
$$x^n = e^{2 i k\pi}$$
Taking the $n$-th root of both sides:
$$x = e^{\frac{2ik\pi}{n}}$$
There now seem to be a (countably) infinite number of solutions for our original equation. So, we went from just one solution ($x=1$) to infinite now. But if you delve into them a little more, you'll find there aren't really infinite distinct solutions. You get $n$ distinct solutions and then they start repeating. The first solution is $x=1$ corresponding to $k=0$. The second is $\omega_n=e^{\frac{2i\pi}{n}}$ corresponding to $k=1$. This is called the "principal root of unity" and all others are integer powers of this. The third is $e^{\frac{4\pi i}{n}} = \omega_n^2$ for $k=2$ and so on. Once we get to $k=n$, we get $e^{\frac{2\pi n}{n}} = e^{2 \pi} = 1$ (same as when $k=0$). And with $k=n+1$, its the same number as for $k=1$. So, only values of $k$ ranging from $0$ to $n-1$ produce distinct roots that we didn't see before. 

Let's consider the case of $n=3$. In other words, we're now looking to find the cube roots of unity, the numbers that satisfy:

$$x^3=1$$
Plugging into $$e^{\frac{2\pi i k}{3}}$$, we get the following 3 distinct values for $k=0,1,2$ and then, they just start repeating.  

![[cube_roots.png#invert | 300]]
### The halving lemma
Now, we get to the most important concept that is central to how we're able to obtain the speedup in the fast fourier transform and also to the question around why the FFT only works on highly composite inputs. 

If you take the $n$-th roots of unity when $n$ happens to be even and square all of them, you get the $n/2$-th roots of unity. But from the previous section, we know there are exactly $n$ distinct $n$-th roots and $n/2$ distinct $n/2$-th roots of unity. So when we square the $n$-th roots, only half of them end up being distinct numbers and the other half just wrap around and repeat the existing ones. And it is the introduction of this redundancy that helps us save some computation and speed things up. 

This is demonstrated below for the case of $n=8$. Squaring each of the $8$-th roots of unity brings us to four distinct points left standing, the fourth roots of unity.

![[halving_lemma.gif#invert | 300]]


## II-B) Divide and conquer
Now, let's see how the halving lemma is actually utilized to create an efficient algorithm. Remember, the problem statememnt of the FFT is really simple. Given $n$ coefficients of a polynomial (stored in an array), evaluate the polynomial at each of the $n$-th roots of unity. 

Thinking first of the naive way to do this that takes $O(n^2)$ computations, we can visualize these as a table with the $n$ coefficients (the inputs) placed along the columns and the $n$ roots of unity placed along the rows of the table. In the i,j cell of the table, we raise the corresponding $\omega$ of that row to the power $j$ and multiply with $a_i$, the corresponding coefficient of the column the cell is in. Then, we sum across all rows to get one element per column (see the green parts of the figure below). These $n$ elements are packaged into the output vector, $y$. The whole process is visualized in the figure below.

![[fft_calc_naive.png#invert | 300]]

To improve on the $O(n^2)$ computations, we use a trick that leans on the halving lemma from the previous section and helps split the problem into smaller sub-problems. This is the divide and conquer strategy that shows up in many CS algorithms like merge sort, binary search in a sorted array, etc.

Since the lemma involved squaring all the $n$ roots of unity, we try to express the equations of interest in terms of only the squares of the original input. All the evaluations we'd like to do are polynomials at each of the $n$ roots of unity of the form:

$$A(x) = a_0+a_1x+a_2x^2+\dots a_{n-1}x^{n-1}$$
Can we modify this equation such that it only involves polynomials in $x^2$? Let's pick the low hanging fruit first. The coefficients that already have $x^2$ and its powers, which are just the evenly indexed coefficients. Separating those and the remaining (which are the odd indexed coefficients) out, we can write the original polynomial:

$$
\begin{align}
A(x) = (a_0 + a_2 x^2 + a_4 x^4 + \dots + a_{n-2}x^{n-2}) +\\(a_1x+a_3x^3+\dots a_{n-1}x^{n-1})\\
\end{align}
$$
Now, with the motivation that we're looking to split the original polynomial into smaller polynomials so we can divide and conquer,

$$
\begin{align}
A(x) = (a_0 + a_2 (x^2)^1 + a_4 (x^2)^2 + \dots + a_{n-2}x^{(n/2-1)^2}) +\\x(a_1+a_3 (x^2)^1+a_5(x^2)^2\dots + a_{n-1}x^{(n/2-1)^2})\\
\end{align}
$$
So, we've succeeded in splitting the original polynomial into two polynomials. And both those polynomials involve only powers of $x^2$. Since the first polynomial has even indexed terms from the original polynomial, we can call it $A_0(x)$ (indices such that remainder when divided by $2$ is $0$) and similarly the second one can be called $A_1(x)$. This gives us:

$$A(x) = A_0(x^2)+xA_1(x^2)\tag{3}$$
Where:
$$\begin{align}
A_0(x) = a_0+a_2x+a_4x^2+\dots + a_{n-2}x^{n/2-1}\\
A_1(x) = a_1+a_3x+a_5x^2+\dots + a_{n-1}x^{n/2-1}
\end{align}\tag{4}$$
So, the original problem of evaluating $A(x)$ at the $n$ roots of unity, $\omega_n^0, \omega_n^1, \dots \omega_n^{n-1}$ reduces to evaluating the polynomials $A_0(x)$ and $A_1(x)$ at the squares of the $n$ roots of unity. Both of the new polynomials are smaller and have $\frac{n}{2}$ coefficients. And now, the halving lemma from the previous section comes in. When we square each of the $n$ roots of unity, the first $\frac{n}{2}$ happen to be the distinct $\frac{n}{2}$-roots of unity. The remaining $\frac{n}{2}$ of them just loop around and repeat as shown in figure-x. So, we don't have to evaluate the polynomials for them again. Hence, the number of points where we have to evaluate the polynomial also gets cut in half too. And since the squares of the $n$ roots are the $\frac{n}{2}$-roots, the two sub-problems involve evaluating polynomials with $\frac n 2$ coefficients at the $\frac n 2$ roots of unity and are hence also discrete Fourier transform problems, just with different (smaller) coefficient arrays. The divide step is now complete. But now that we know how to divide, we don't stop at just one division. Instead, we keep going until we hit the base case, which is an array of length one. At that point, the Fourier transform is the element in the array itself. The process of dividing the problem is core to the speed up the FFT brings. This is visualized in the figure below to further strengthen the intuition. We set up for the $n \times n$ multiplications like in figure-y. But then, we split the $n \times n$ matrix into two smaller matrices, each of size $n \times n/2$. The red parts of the matrices are just repetitions of the blue parts, so they can be discarded. This makes the two smaller matrices $n/2 \times n/2$.

![[fft_split_2.png#invert | 300]]
For getting the blue part of the original matrix, the output values from $0 \to n/2-1$, we can just use the blue parts of the two split matrices via equation (3). For getting the yellow part of the original matrix, the output values from $n/2 \to n-1$, we can use the equations below:


$$\begin{align}
y[n/2+k] = A(\omega_n^{n/2+k})\\
 = A_0(\omega_n^{n+2k})+\omega_n^{n/2+k}A_1(\omega_n^{n+2k})\\
 = A_0(\omega_n^{n} \omega_n^{2k})+\omega_n^{n/2}\omega_n^kA_1(\omega_n^{n}\omega_n^{2k})
\end{align}
$$
Now, by definition,
$$\begin{align}
\omega_n = e^{2\pi i/n}\\
\omega_n^{n/2} = e^{\pi i} = -1\\
\omega_n^n = e^{2\pi i} = 1
\end{align}$$
Plugging these into equation (4),
$$\begin{align}
y[n/2+k] = A_0(\omega_n^{2k})-\omega_n^kA_1(\omega_n^{2k})\\
 = y_0[k]-\omega_n^k y_1[k]
\end{align} \tag{4}$$

With this context on how the divide and conquer works, we can introduce the Python code (note: the initial skeleton of this was written by ChatGPT-3, the remainder translated from the pseudo code in chapter 30 of the [clr_text_book]). 

[github_gist](https://gist.github.com/ryu577/84c2e48289fe0912b4db95154c284c73)
```python
import numpy as np

def fft_2(a):
	"""
	Perform a fast Fourier transform
	on the array, a. The size of the array
	has to be a power of 2. Otherwise, it will
	yield incorrect results.
	""" 
	n = len(a)
	if n == 1:
		return a
	# The complex roots of unity. omega is just 1
	# to start with and will be repeatedly multiplied
	# by the principal root, omega_n to get the others.
	omega_n = np.exp(1j*2*np.pi/n)
	omega_k = 1+0j
	# Extact the even indices from array and perform FFT on them.
	# x[::2] means start and the beginning and go in steps of 2.
	y_0 = fft_2(a[::2]) 
	# Extract the odd indices from array and perform FFT on them.
	# x[1::2] means start at index 1 and go in steps of 2.
	y_1 = fft_2(a[1::2]) 
	# The result array.
	y = [0]*n
	# The step that merges the two inputs into the
	# final answer. It takes O(n) time since
	# it is just a single loop.
	for k in range(n//2):
		y[k] = y_0[k]+omega_k*y_1[k]
		y[k+n//2] = y_0[k]-omega_k*y_1[k]
		omega_k = omega_k*omega_n
	
	return y
```

Line 10 is the base-case of the recursion, an array of size 1. Here, the discrete fourier transform is  simply the array itself.

Lines 20 and 23 leverage the observation that we can split the FFT into more FFT's that are half the size.

The for loop in line 29 then combines the two smaller FFT's and produces the larger FFT per equations (3) (line 30) and (4) (line 31).

## II-C) Complexity analysis
To appreciate the speedup this approach is bringing us over the simpler naive implementation, let's consider performing the discrete fourier transform of an $8$ element array. To count all the operations that need to happen without the fancy divide and conquer, let's bring back the matrix from before. 
![[fft_calc_naive.png#invert | 300]]
There are a total of 64 cells in the matrix above and all of them need at least one multiplication (multiplying the coefficient with the complex number). So, that's at least 64 multiplications (the complex roots need to be calculated as well, but we'll ignore that as any approach would require it). To add them up and get the output $y$ vector, we'll need to sum each row. Summing 8 numbers requires 7 additions, so that makes for an additional $7 \times 8 = 56$ additions. So, the naive approach has 64 coefficient-complex number multiplications and 56 additions. Let's see how the divide and conquer fares.

The way the algorithm in the "fft_2" routine pans out is shown below. We start out with the 8 element array and want to get the DFT. What we reference as $y_0$ and $y_1$ in the code, we call $y_l$ and $y_r$ in the diagram below for left and right arrays (since that's how they're arranged in the recursion tree). When $y_l$ is split into two smaller arrays, they get the names $y_{ll}$ and $y_{lr}$ etc. We see there are just three recursion levels where arithmetic operations are happening. This isn't surprising since $\log_2(8)=3$. On the left of the diagram, we see that 8 multiplications and 4 additions happen at each layer in the recursion tree. This makes for a total of 24 additions and 12 multiplications. Contrast this with the 64 multiplications and 56 additions for the naive approach. 

![[fft_run_time.png#invert | 300]]
In general, the problem size at each iteration gets split in two sub-problems half the size of the original one. And we then require one for-loop (or $O(n)$ computations) to combine the outputs from the two sub-problems and solve the original problem of interest. Let's say that total operations required for an input array of size $n$ is $T(n)$. Every time we split into two sub-problems, we need to do $2T(n/2)$ work to solve each of them and then $O(n)$ work to combine them, meaning $c.n+d$ for some constants, $c$ and $d$. This can be expressed:

> [[Recurrence]] has the method for solving recursion tree for merge sort.

$$\begin{align}
T(n)=2T(\frac{n}{2} )+(cn+d)\\
T(n)= 2[2T(\frac{n}{4})+\color{red}{c \frac n2}+d] +c.n+d \\= 4T(\frac{n}{4})+\color{red}2cn+(1+2).d\\
\end{align}
$$
Continuing with this pattern,
$$ \begin{align}
T(n)=8T(\frac{n}{8})+\color{red}3cn+(1+2+2^2)d\\
\vdots\\
T(n)= 2^kT(\frac{n}{2^k})+\color{red}kcn + (1+2+\dots 2^{k-1})d
\end{align}
$$

We stop when $\frac{n}{2^k}=1$ and $\therefore n=2^k$. This means:

$$k = \log_2(n)$$

At that stage, we get $T(1)$ which is zero additions or multiplications. Plugging into the equation above we get:

$$\begin{align}T(n) = cn \log_2(n) + d(2^{\log_2(n)}-1)\\
 = c n \log_2(n)+d(n-1)
\end{align}$$
Which is $O(n\log(n))$ computations.

# III) Back to polynomials
First, let's tie up the lose end from section I and describe how this new divide and conquer algorithm is going to help perform polynomial multiplication much faster than the double for-loop.   Let's say we're given two polynomials expressed as coefficient arrays. And the sizes of those arrays are $n$ and $m$. The two polynomial will look something like:

$$\begin{align}
A(x) = a_0+a_1x+a_2x^2+\dots a_{n-1}x^{n-1}\\
B(x) = b_0+a_1x+b_2x^2+\dots b_{m-1}x^{m-1}
\end{align}$$
The degree (highest power of $x$) of the first polynomial is $n-1$ and that of the second polynomial is $n-2$. So, the product polynomial, $C(x)$ will have degree $n+m-2$ and look something like:
$$C(x) = c_0+c_1x+c_2x^2+\dots c_{n+m-2}x^{n+m-2}$$
There are a total of $n+m-1$ coefficients. 

To do the multiplication, we want to first convert the two input polynomials to point-value form. Once there, as long as the points at which the two are evaluated are the same, we can multiply point-wise to get the values of $C(x)$. To specify the $n+m-1$ coefficients of $C(x)$, we'll need $n+m-1$ points to evaluate them at. So, we want $A(x)$ and $B(x)$ to also be calculated at $n+m-1$ points. But, they have just $n$ and $m$ coefficients respectively and applying the Fourier transforms to them will evaluate them at the $n$ and $m$-th roots of unity respectively. So, we make them both have $n+m-1$ coefficients instead. The way to do this is to pretend they are both polynomials of degree $n+m-1$. By just tagging on zero coefficients at the end. So, the two polynomials become:

$$\begin{align}
A(x) = a_0+a_1x+a_2x^2+\dots a_{n-1}x^{n-1}+0x^n+\dots+0x^{n+m-2}\\
B(x) = b_0+a_1x+b_2x^2+\dots b_{m-1}x^{m-1}+0x^m+\dots+0x^{n+m-2}
\end{align}$$
Just pass these larger, zero-padded arrays to the FFT routine and it'll calculate the values of the two polynomials at each of the $n+m-1$-th roots of unity. These steps will take $O((n+m)\log(n+m))$ time. 

Then, multiply these two sets of $n+m-1$ values point-wise, which takes $O(n+m)$ time. And now that we have the point-values of $C(x)$ at those $n+m-1$ roots of unity, we can go back to the coefficient representation of that polynomial. This last step is called the inverse Fourier transform (since it does the opposite thing a Fourier transform does). And per theorem 30.7 of the [clr_text_book], this can also be done in $O((n+m)\log(n+m))$ time. In fact, the FFT routine needs to be modified only slightly to convert it into an inverse FFT routine.

The overall process takes  $O((n+m)\log(n+m))$ time, which is better than the $O(nm)$ of the naive method of doing this.

These two approaches are shown pictorially in figure 30.1 of the [clr_text_book] (they consider the case $n=m$). I've shamelessly pasted that figure here.

![[clr_polynom_shortcut.png | 500]]

But aren't we forgetting that the efficient implementation we have only works on powers of $2$? What if $n+m-1$ is not a power of $2$? One solution would be to find the next higher number that is a power of $2$. If we get lucky, that could be quite close. But what if the next power of $2$ is very far away? And why are we stuck with this restriction anyway?

# IV)  Why powers of 2?
At first glance, it would seem like the whole divide and conquer paradigm is the reason we're stuck with powers of $2$. That is until you realize that the [clt_text_book] is full of divide and conquer algorithms. Examples are binary search, sorting algorithms like merge sort and quick sort among many otheres. And none of them have the restriction of only working on inputs that are powers of $2$. Even if you implement them assuming the input size is going to be a power of $2$ and then give them arrays of sizes that are not powers of $2$, they just work for them too. In fact, the FFT is the only algorithm in that book that doesn't generalize to non-powers of $2$. What happens if you just give the fft_2 routine an array that isn't correctly sized? Nothing breaks, no explosion happens and no error is thrown. But, sadly, the values produced are incorrect. They won't match up with what you would get from the naive method.

The reason for the limitation is the use of the halving lemma from section II. If you take the $n$-th roots of unity when $n$ is even and square them, you will get the $n/2$ roots of unity and the last $n/2$ values will nicely fold into the first $n/2$. But if $n$ isn't even, this whole thing breaks down. First, $n/2$ isn't even an integer. And squaring the complex numbers doesn't even result in the $[n/2]$ roots of unity or anything. Which is why assuming that when $n$ isn't even will just lead to a wrong answer. And because we want to be able to do the split all the way to the base-case, $n$ must be a power of $2$ for the fft_2 routine to work.

But, what if $n$ isn't quite a power of $2$, but a near miss. Say we have: $n=3*2^{10}$. Of course, we could just try to evaluate the FFT at $n=4*2^{10}$, but even this would be unncessary if we modify the fft_2 routine very slightly and having a kind of "eject button" when the power of $2$ assumption is broken. As we split the $3*2^{10}$-sized array two ways recursively, we'll keep getting even array sizes until we reach $6$. Then, when we split that further into two calls on arrays of size $3$, we just default to the naive approach of just evaluating the polynomials. All we need is an "if" condition at the very top checking if $n$ is a power of $2$ and falling back to the naive routine if so.

But what if we had an array whose input size was $n = 3^7*2^2$? Once we ran out of two way splits (which'll happen very quickly), we'd end up with two arrays each sized $3^7$ which are quite large. At that point, you'll be wishing there was a routine that worked on powers of $3$ inputs instead of powers of $2$. And in-fact, there is. Just as we can choose the number of splits for other divide and conquer algorithms, we can do the same for the FFT. In the next section, we'll cover a version of the FFT, fft_3 which works on powers of $3$. Now, we can have a master routine that passes the array to the fft_2 as long as the input is even, to fft_3 if its a multiple of $3$ and to the naive implementation as the final fall-back. But then we can have versions for the first $m$ prime numbers, fft_2, fft_3, fft_5, fft_7, fft_11 and so on. Now, as long as we get an input array size that is highly composite, we're still going to get all the benefit of the FFT. And it is easy to find a highly composite number that is only slightly larger than any $n$, which suffices for our polynomial multiplication and most other use-cases.

# IV) An FFT for powers of 3
Finally, let's cover an FFT algorithm that works on powers of $3$ instead of $2$. The approach can obviously be extended to any number (but we'd want to use it for primes). As discussed in the previous section, the reason the fft_2 routine gave incorrect answers for input sizes not powers of $2$ was the halving lemma. If we want it to work for powers of $3$, we need a corresponding "trisecting lemma". 

If $n$ is a power of $3$ and you take the $n$-th roots of unity and cube them all, you get the $n/3$-th roots of unity. The other $2n/3$ roots just wrap around and overlap with the $n/3$ roots.

This will give us the ability to split the FFT into three sub-problems. But, we'd also like to know how to combine them into the FFT array we want. To generalize equation (4) to the case of a three-way split, this is how the Math works out:

![[powr_of_3.png#invert | 400]]

Where in the last step we used the fact that:

$$e^{8 \pi i /3} = e^{6 \pi i /3} e^{2\pi i /3} = e^{2\pi i /3}$$
With all of these facts, we have the fft_3 algorithm:
```python
def fft_3(x):
	"""
	Perform a fast Fourier transform
	on the given array of complex numbers.
	""" 
	n = len(x)
	if n == 1: 
		return x
	# The complex roots of unity.
	omega_n = np.exp(1j*2*np.pi/n)
	omega = 1+0j
	# Extact the indices from array that are powers of 3.
	y_0 = fft_3(x[::3]) 
	# Extract the indices from array that are of the form 3x+1.
	y_1 = fft_3(x[1::3])
	# Extract the indices from array that are of the form 3x+2.
	y_2 = fft_3(x[2::3]) 
	# The result array.
	y = [0]*n
	# The step that merges the two inputs into the
	# final answer. This step takes O(n) time since
	# it is just a single loop.
	for k in range(n//3):
		y[k] = y_0[k]+omega*y_1[k]+omega**2*y_2[k]
		y[k+n//3] = \
			y_0[k]+np.exp(2*np.pi*1j/3)*omega*y_1[k]+\
			np.exp(4*np.pi*1j/3)*omega**2*y_2[k]
		y[k+(2*n)//3] = \
			y_0[k]+np.exp(4*np.pi*1j/3)*omega*y_1[k]+\
			np.exp(2*np.pi*1j/3)*omega**2*y_2[k]
		omega = omega*omega_n
	
	return y
```

___________________________
# Fin


My answer on this: https://math.stackexchange.com/a/4606378/155881

- Why does it not work for any $n$? Merge sort is also divide and conquer and it does.
- Halving lemma is the bottleneck.
- Show an example with 16.

# IV) Does it matter?
- In most applications, we have some freedom in choosing $n$.
	- For polynomial multiplication/ convolution, padding by 0 doesn't matter? See cs.stackexchange qn
- If it is only powers of 2, we might find ourselves very far from the closest one.
- But we can also create one for powers of 3. Then powers of 5 and so on. Now, we can find a highly composite number close to $n$.
- Algorithm starts splitting in 2's, then in 3's, then in 5's and finally uses the naive, $O(n^2)$ approach.
- Methods to do it when inputs are large primes.

# V) Convolutional neural networks
The most successful algorithm in computer vision recently has been convolutional neural networks (CNN's). The fundamental operation here is convolution of an image with a "kernel matrix".

boox/daily/221216
![[2d_fft.png#invert | 400]]


# V) FT: Splitting a signal into frequencies
While we motivated the Fourier transform here through an algorithmic angle, that's just an incidental application. Something of a side-benefit, a hobby. The main job of the Fourier transform is to decompose a signal into its constituent frequencies. 


<TODO: Add python code>

https://www.wolframalpha.com/input?i2d=true&i=Integrate%5Bcos%5C%2840%295+x%5C%2841%29cos%5C%2840%292x%2B1%5C%2841%29%2C%7Bx%2C-2000000%2C2000000%7D%5D


it's original motivation comes from finding the frequencies a signal is composed of.

The general Fourier transform X(ω) of a function x(t), on which the DFT is based is defined as:

$$X(\omega) = \int\limits_{-\infty}^\infty x(t) e^{-i \omega t} dt$$

So, it's simply 

In the discrete case, this integral becomes a polynomial.

The most common use of a Fourier transform is to discern the frequencies a signal is composed of. The signal could be measurements from a seismograph (or network of them) - devices that record vibrations in the ground. The vibrations could be from natural phenomenon like earthquakes and volcanoes or man-made phenomenon like underground nuclear tests. And the frequency spectrum of the signal(s) could help tell those apart.

boox/daily/221209
![[fft_polynom.png#invert | 500]]

#building/blog #convolution 

Note from Remarkable 1 (Daily/221104)

![[fft_3_outline.png#invert | 400]]



[[Convolution]]

References:
[divide and conquer - Why does merge sort work for any $n$, but the basic FFT algorithm only for powers of $2$? - Computer Science Stack Exchange](https://cs.stackexchange.com/questions/155685/why-does-merge-sort-work-for-any-n-but-the-basic-fft-algorithm-only-for-power/155692?noredirect=1#comment327582_155692)

[3B1B what is a Fourier transform?] https://www.youtube.com/watch?v=spUNpyF58BY&vl=en

[[1312.5851] Fast Training of Convolutional Networks through FFTs (arxiv.org)](https://arxiv.org/abs/1312.5851 "https://arxiv.org/abs/1312.5851")

[cormen] Introduction to algorithms by Cormen et.al.
[Veretasium] 

[3b1b_convolution]  

Created: 221211
#building/blog #convolution #cs/algo/fft #math 
[[Convolution]]

Other links:
[The beginning of the end for Convolutional Neural Networks? (analyticsindiamag.com)](https://analyticsindiamag.com/the-beginning-of-the-end-for-convolutional-neural-networks/ "https://analyticsindiamag.com/the-beginning-of-the-end-for-convolutional-neural-networks/")

[[blogs]]

# Graveyard
![[fft_recursn.png#invert | 500]]

boox/daily/221209
![[fft_algo.png#invert | 400]]


Boox/Daily/221225
![[fft_problem.png#invert | 400]]

Boox/Daily/221225
![[fft_recur_matr-min.png#invert | 400]]

Regarding the speedup:

![[fft_recursn2.png#invert | 400]]
![[fft_3_algo.png#invert]]