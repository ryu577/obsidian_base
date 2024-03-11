[[Diffusion]], [[231230]]

# Introduction
## Naive way of doing it
Phrase to vector. Image is already a kind of vector. Then insert a deep neural network between them and simply learn the weights.
- Imperceptible parts of the image would be focussed on.

## Statistical underpinnings

Statistical physics underpinnings of the model.

- Diffusion in the sense of thermodynamics/ statistical physics.
- A blob of ink dropped into a liquid.
	- Initially maintains its structure.
	- Molecules of water collide with it.
	- Diffuses through the liquid.
	- This process is approximated well with a Markov chain.
	- Brownian motion and this is how Einstein hypothesized the existence of atoms.
- The structure of the blob is destroyed as time plays forward.
	- This gives a deeper meaning to "forward process". It's literally time playing forward.
- All possible images for a high dimensional space.
	- Most images in this space look like noise.
	- Images we'd see in real life form a cloud. The cloud has a distinct shape. We don't know what that shape is necessarily.
	- Think of that cloud diffusing into the larger space as time plays forward.

# How it physically works
- Draws on top of a Gaussian slate via reverse diffusion.
	- As opposed to a blank slate.
	- Reverse diffusion. Literally turns back time.
	- Unlike the forward step which is easy, need a neural network.
	- In small steps. Not one step going from noise to image.
		- Makes the math tractable. Diffusion process exists for any smooth distribution.

# The math
A toy model of the Math with simplifications.
- $\mu$ and $\Sigma$ are the parameters of the Gaussian.
- Predict these parameters with a neural network.
	- Any network architecture can be plugged in. It's just a mapping.
	- The network parameters can be further conditioned on additional information like the prompt.

# Further advances
- Cosine schedule instead of linear schedule for the forward process (noising the image).

# References
[1] https://www.youtube.com/watch?v=HoKDTa5jHvg
- [Original stable diffusion](https://arxiv.org/pdf/2112.10752.pdf) (Apr 2022) [[stable_diffusion0.pdf|local]] High-Resolution Image Synthesis with Latent Diffusion Models. 
	- [Diffusion models beat GANs](https://arxiv.org/pdf/2105.05233.pdf) appendix B has best explanation of what's going on.
- [Denoising Diffusion Probabilistic Models](https://arxiv.org/pdf/2006.11239.pdf) (Dec 2020) [[denoising_diffusion.pdf|local]] has the training algorithm.
- [Seminal paper](https://arxiv.org/pdf/1503.03585.pdf) 2015, [[theormodynamics_diffusion.pdf|local]] "Deep Unsupervised Learning using Nonequilibrium Thermodynamics"
- [Lilian Wang blog](https://lilianweng.github.io/posts/2021-07-11-diffusion-models/).[[DiffusionLilWang.pdf#invert|local]]
Videos:
- https://www.youtube.com/watch?v=HoKDTa5jHvg
- https://www.youtube.com/watch?v=vu6eKteJWew
	- Code: https://github.com/explainingai-code/DDPM-Pytorch
keywords: code github youtube video diffusion papers cv midjourney
# Math


$$\beta_1, \beta_2, \dots \beta_T$$
$$q(x_t|x_{t-1}) \sim \mathcal{N}(\sqrt{1-\beta_t}x_{t-1}, \beta_t I)$$
$$\begin{array}a\alpha_t= 1-\beta_t\\
\bar{\alpha_t}= \alpha_1.\alpha_2\dots \alpha_t
\end{array}$$
$$q(x_t|x_0) \sim \mathcal{N}(\sqrt{\bar{\alpha_t}}x_0, (1-\alpha_t)I)$$
$$q(x_{t-1}|x_t, x_0) \sim \mathcal{N}(\tilde{\mu}(x_t, x_0), \tilde{\beta_t}I)$$

$$\tilde{\mu}(x_t,x_0) = \frac{1}{\sqrt{\alpha_t}}\left(x_t-\frac{1-\alpha_t}{\sqrt{1-\bar{\alpha_t}}}\epsilon_t\right)$$


