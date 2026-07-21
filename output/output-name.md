# Denoising Diffusion Probabilistic Models (DDPM)

> **Source mode:** Source-grounded  
> **Paper type:** Hybrid — generative-model method and empirical evaluation  
> **Formatting note:** Mathematical expressions use portable Unicode notation, so this file renders correctly without a LaTeX or MathJax extension.

## The key intuition

**A DDPM learns to generate images by practicing one small task: remove a little Gaussian noise from a partially corrupted image.**

Instead of generating a complete image in one difficult jump:

```text
Random noise
   ↓
One large neural-network transformation
   ↓
Finished image
```

a DDPM generates gradually:

```text
Random noise
   ↓
Remove a little noise
   ↓
Remove a little more noise
   ↓
Repeat many times
   ↓
Finished image
```

The training trick is to create corrupted images automatically from real data.

```text
Real image
   ↓
Add a known amount of random noise
   ↓
Ask the network to predict that noise
```

Because the training system knows exactly which noise was added, it can create unlimited supervised denoising examples without human labels.

In simple terms:

> “Destroy images in a controlled way, then learn how to reverse the destruction one small step at a time.”

---

# The problem DDPM solves

A generative model must learn a complicated probability distribution over natural images.

A direct task such as:

```text
Input:
   random numbers

Target:
   one realistic image
```

is difficult because the model must decide everything at once:

```text
large-scale layout
object identity
shape
color
texture
fine detail
```

DDPM breaks this large problem into many easier conditional problems:

```text
Given a slightly noisy image,
what cleaner image probably came immediately before it?
```

Each reverse step is small enough to model with a Gaussian transition whose mean is predicted by a neural network.

---

# The two processes

DDPM contains two opposite Markov chains.

```text
Forward process q:
   real image → noise

Reverse process pθ:
   noise → generated image
```

A **Markov chain** means each step depends only on the current state, not the complete earlier history.

---

# Part 1: The fixed forward diffusion process

The forward process starts with a real data example **x₀**.

At each time step *t*, it adds a small amount of Gaussian noise:

**q(xₜ | xₜ₋₁) = Normal(√(1 − βₜ) xₜ₋₁, βₜI)**

The equation is trying to say:

> “Keep most of the previous image, shrink it slightly, and add a small random disturbance.”

## What each symbol means

**x₀**  
The original clean image.

**xₜ**  
The image after *t* noising steps.

**βₜ**  
The noise variance used at step *t*. A small **βₜ** means only a little information is destroyed in one step.

**I**  
The identity covariance. It means independent Gaussian noise is added to every image coordinate.

**Normal(mean, variance)**  
A Gaussian probability distribution.

## One forward step

```text
Current image xₜ₋₁
   ↓
Scale by √(1 − βₜ)
   ↓
Add Gaussian noise with variance βₜ
   ↓
Slightly noisier image xₜ
```

After a few steps, the image remains recognizable. After many steps, the signal is almost completely destroyed.

```text
x₀:
   clean image

x₁:
   almost clean

x₁₀₀:
   visibly noisy

x₅₀₀:
   mostly structure and noise

x₁₀₀₀:
   approximately pure Gaussian noise
```

The forward process is fixed. It has no learned neural-network parameters in the paper’s implementation.

In simple terms:

> “The model does not learn how to add noise. We choose that process ourselves.”

---

# Why use many small noise steps?

If the entire image were destroyed in one jump, reversing that jump would be highly ambiguous.

```text
Pure noise
   ↓
Which of millions of possible images produced it?
```

With small steps, the reverse question is easier:

```text
Slightly noisy image
   ↓
What nearby cleaner image probably produced this?
```

When the forward perturbations are small and Gaussian, the reverse transitions can also be modeled with Gaussian distributions.

---

# The cumulative noise shortcut

Define:

**αₜ = 1 − βₜ**

and:

**ᾱₜ = α₁ α₂ … αₜ**

Here **ᾱₜ** is the fraction of the original signal retained after *t* steps.

- When **ᾱₜ** is near 1, the image is mostly clean.
- When **ᾱₜ** is near 0, the image is mostly noise.

The important closed-form result is:

**q(xₜ | x₀) = Normal(√ᾱₜ x₀, (1 − ᾱₜ)I)**

Equivalently, one can sample **xₜ** directly:

**xₜ = √ᾱₜ x₀ + √(1 − ᾱₜ) ε**

where:

**ε ∼ Normal(0, I)**

This equation says:

```text
Noisy image xₜ
   =
remaining clean signal
   +
added random noise
```

The first term says:

> “Keep √ᾱₜ of the original image.”

The second term says:

> “Fill the remaining amount with known Gaussian noise.”

## Why the shortcut matters

Without this formula, training at time step 800 would require applying 800 noising operations.

Instead:

```text
Clean image x₀
   ↓
Choose any time t
   ↓
Sample one Gaussian noise tensor ε
   ↓
Compute xₜ directly
```

In simple terms:

> “Jump directly to any corruption level without simulating all previous steps.”

---

# Part 2: The learned reverse process

Generation begins with:

**x_T ∼ Normal(0, I)**

This is pure Gaussian noise.

The model learns transitions:

**pθ(xₜ₋₁ | xₜ) = Normal(μθ(xₜ, t), Σₜ)**

The equation is trying to say:

> “Given the current noisy image and its noise level, predict a Gaussian distribution for the slightly cleaner image.”

Here:

**θ**  
The neural-network parameters.

**μθ(xₜ, t)**  
The predicted mean of the previous, cleaner state.

**Σₜ**  
The reverse-step variance. In the paper’s main implementation, this variance is fixed rather than learned.

---

# The key parameterization: predict the noise

The network could directly predict the cleaner image or the reverse-process mean.

Ho et al. instead train **εθ(xₜ, t)** to predict the Gaussian noise **ε** that was added.

```text
Input to network:
   noisy image xₜ
   time step t

Output from network:
   estimated noise εθ(xₜ, t)
```

In simple terms:

> “Look at the corrupted image and tell me which part is noise.”

Once the noise is estimated, the algorithm can subtract the appropriate amount and compute the mean of the next cleaner state.

---

# Why predicting noise is enough

Recall:

**xₜ = √ᾱₜ x₀ + √(1 − ᾱₜ) ε**

If the network knows:

```text
xₜ
t
estimated ε
```

then it can infer the clean-signal component hidden inside **xₜ**.

Conceptually:

```text
Noisy observation
   −
Estimated noise contribution
   ↓
Estimate of the underlying clean signal
```

The paper shows that this parameterization turns the reverse-process objective into a form closely related to denoising score matching across multiple noise scales.

---

# The simplified training loss

**L_simple = Eₓ₀,ε,t [ ‖ε − εθ(xₜ, t)‖² ]**

with:

**xₜ = √ᾱₜ x₀ + √(1 − ᾱₜ) ε**

The loss says:

> “Make the network’s predicted noise match the actual noise that was added.”

## The target

**ε**  
The exact random noise sampled during corruption.

## The prediction

**εθ(xₜ, t)**  
The network’s estimate of that noise.

## The error

**‖ε − εθ(xₜ, t)‖²**  
The squared difference between the true and predicted noise.

```text
Known added noise
   ↓ compare
Predicted noise
   ↓
Mean-squared error
```

---

# The complete training flow

```text
Training dataset of clean images
   ↓
Sample a clean image x₀

   ↓
Sample a random time step t
   The network must learn every noise level.

   ↓
Sample Gaussian noise ε

   ↓
Create xₜ directly
   xₜ = √ᾱₜ x₀ + √(1 − ᾱₜ) ε

   ↓
Give xₜ and t to the neural network

   ↓
Network predicts εθ(xₜ, t)

   ↓
Compare predicted noise with actual noise
   loss = ‖ε − εθ(xₜ, t)‖²

   ↓
Update θ with gradient descent

   ↓
Repeat
```

The model does **not** run the full reverse chain during each training update.

It trains on one randomly selected time step per example.

In simple terms:

> “At every update, quiz the model on one randomly chosen difficulty level.”

---

# Conceptual training pseudocode

```text
repeat until training converges:

    x₀ = sample_from_dataset()

    t = random_integer(1, T)

    ε = standard_gaussian_noise(shape_of=x₀)

    xₜ =
        sqrt(alpha_bar[t]) * x₀
        +
        sqrt(1 - alpha_bar[t]) * ε

    predicted_noise = network(xₜ, t)

    loss = squared_error(ε, predicted_noise)

    update_network_parameters(loss)
```

---

# The sampling equation

At generation time, the model starts from Gaussian noise and repeatedly applies:

**xₜ₋₁ = 1/√αₜ · (xₜ − βₜ/√(1 − ᾱₜ) · εθ(xₜ, t)) + σₜz**

where:

- **z ∼ Normal(0, I)** when *t* is greater than 1;
- **z = 0** at the final step;
- **σₜ** controls the random variation added during the reverse step.

The equation performs three operations.

## 1. Estimate and remove noise

**xₜ − βₜ/√(1 − ᾱₜ) · εθ(xₜ, t)**

> “Use the neural network’s noise estimate to move toward a cleaner image.”

## 2. Correct the scale

**1/√αₜ**

This compensates for the shrinking applied during the forward process.

## 3. Add reverse-process randomness

**σₜz**

This keeps generation probabilistic.

Different random seeds can therefore produce different images.

---

# The complete sampling flow

```text
Sample pure Gaussian noise x_T
   ↓
For t = T down to 1:

   Predict noise εθ(xₜ, t)

   ↓
   Compute the mean of the cleaner state

   ↓
   Add the prescribed random variation
   except at the final step

   ↓
   Produce xₜ₋₁

   ↓
Return x₀
```

In simple terms:

> “Start with static and repeatedly ask the network what noise can be safely removed next.”

---

# Conceptual sampling pseudocode

```text
x = standard_gaussian_noise(image_shape)

for t from T down to 1:

    predicted_noise = network(x, t)

    reverse_mean =
        (1 / sqrt(alpha[t]))
        *
        (
            x
            -
            beta[t]
            / sqrt(1 - alpha_bar[t])
            * predicted_noise
        )

    if t > 1:
        z = standard_gaussian_noise(image_shape)
    else:
        z = 0

    x = reverse_mean + sigma[t] * z

return x
```

---

# A tiny one-dimensional example

Suppose:

**x₀ = 2**

At some time step:

**√ᾱₜ = 0.8**

**√(1 − ᾱₜ) = 0.6**

Sample noise:

**ε = −0.5**

Then:

**xₜ = 0.8 × 2 + 0.6 × (−0.5)**

**xₜ = 1.6 − 0.3 = 1.3**

The network receives:

```text
noisy value: 1.3
time step: t
```

Its target is:

```text
actual noise: −0.5
```

If it predicts **−0.4**, the squared error is:

**(−0.5 − (−0.4))² = 0.01**

This toy example preserves the actual training mechanism, although real images contain many dimensions.

---

# Image shapes

Suppose a batch contains:

```text
N images
C color channels
H image rows
W image columns
```

Then:

```text
x₀: N × C × H × W
ε:  N × C × H × W
xₜ: N × C × H × W
```

The network predicts:

```text
εθ(xₜ, t):
   N × C × H × W
```

For CIFAR-10 with a batch of 128:

```text
x₀: 128 × 3 × 32 × 32
ε:  128 × 3 × 32 × 32
xₜ: 128 × 3 × 32 × 32
prediction: 128 × 3 × 32 × 32
```

---

# How the network knows the noise level

The same network is reused at every time step.

It must distinguish between:

```text
an almost-clean image
and
an almost-pure-noise image
```

The time step *t* is embedded into a vector and supplied to the network.

The paper uses sinusoidal time-step embeddings.

```text
Time step t
   ↓
Time embedding
   ↓
Injected into the U-Net
   ↓
Denoising behavior adapts to the current noise level
```

In simple terms:

> “Tell the denoiser how damaged the image currently is.”

---

# The neural-network architecture

The paper uses a U-Net-like architecture.

```text
High resolution
   ↓
Downsample to capture large-scale structure
   ↓
Process low-resolution global features
   ↓
Upsample
   ↓
Combine with earlier high-resolution features
```

This helps the model use both:

```text
global understanding:
   object layout and large shapes

local understanding:
   edges, textures, and fine details
```

The paper also uses self-attention at selected feature-map resolutions.

---

# Why DDPM works intuitively

## 1. It turns generation into supervised denoising

The procedure creates the corruption and therefore knows the exact answer.

```text
Generate noise ε
   ↓
Use ε to corrupt x₀
   ↓
ε becomes the training target
```

## 2. Each reverse step is easier than full generation

```text
Very hard:
   noise → complete image

Easier:
   noisy image → slightly cleaner image
```

## 3. Random time sampling covers the whole reverse chain

Over many updates, one network learns:

```text
fine-detail cleanup at small t
mid-level reconstruction at medium t
large-scale structure recovery at large t
```

## 4. The closed-form forward process keeps training efficient

A training example at any time step can be created in one operation.

## 5. Noise prediction gives a simple target

Gaussian noise has a known scale and distribution, so mean-squared error is straightforward to optimize.

## 6. Repeated denoising creates progressive structure

Large-scale features tend to emerge before fine details during reverse generation.

---

# Connection to score matching

A probability distribution has a score:

**∇ₓ log p(x)**

This vector points toward directions where data density increases.

For a noisy image, a score model asks:

> “Which small change would make this image look more like a sample from the noisy data distribution?”

The DDPM noise predictor is proportional to a score estimate at each noise level.

```text
Predicted noise
   ↓
indicates a direction away from corruption
   ↓
acts like a learned direction toward more probable images
```

The paper connects noise-prediction training to denoising score matching and reverse sampling to annealed Langevin dynamics.

---

# Connection to variational inference

The intermediate noisy states:

```text
x₁, x₂, …, x_T
```

act as latent variables between data and pure noise.

Training begins from a variational upper bound on negative log likelihood.

The bound compares:

```text
the true reverse conditional implied by the forward process
versus
the learned reverse conditional
```

Because both are Gaussian, the terms can be calculated in closed form.

The paper’s simplified objective removes the original weighting and trains the noise predictor with an unweighted mean-squared error.

The empirical tradeoff was:

```text
True variational bound:
   better likelihood / codelength

Simplified noise-prediction objective:
   better visual sample quality
```

---

# What “probabilistic” means

The reverse process predicts a distribution, not one forced answer.

```text
Predicted mean
   +
Gaussian random variation
   ↓
One possible cleaner state
```

A noisy state may plausibly develop into more than one image.

---

# The noise schedule

The paper uses **T = 1000** diffusion steps.

Its forward variances increase linearly:

```text
β₁ = 0.0001
   ↓
...
   ↓
β_T = 0.02
```

The cumulative effect makes **x_T** close to a standard Gaussian.

This schedule is chosen by the practitioner rather than learned in the main model.

---

# What the experiments support

## CIFAR-10

The paper reports:

| Metric | DDPM result | Direction |
|---|---:|---|
| Inception Score | 9.46 | higher is better |
| FID against training data | 3.17 | lower is better |
| FID against test data | 5.24 | lower is better |

At publication time, the reported FID of 3.17 was state of the art for unconditional CIFAR-10 generation.

## Larger images

On 256 × 256 LSUN datasets, the paper reports sample quality comparable to ProgressiveGAN.

## Parameterization ablation

The strongest evaluated sample-quality combination was:

```text
predict noise ε
+
use fixed reverse variances
+
train with the simplified objective
```

Learning reverse variances was unstable or produced worse samples in the reported setup.

## Likelihood versus sample quality

The true variational bound produced better negative log likelihood.

The simplified objective produced better-looking samples.

---

# So the results support

```text
- diffusion probabilistic models can generate high-quality images;
- predicting noise is an effective reverse-process parameterization;
- a simple denoising MSE can improve perceptual sample quality;
- iterative Gaussian denoising can compete with strong generative models;
- diffusion supports progressive generation and compression interpretations.
```

# The results do not prove

```text
- that diffusion is always better than GANs, VAEs, flows,
  or autoregressive models;
- that 1000 sampling steps are necessary or optimal;
- that the original noise schedule is optimal;
- that better FID guarantees better human usefulness;
- that the model learns unbiased or factual representations;
- that the same settings are optimal for all data types.
```

---

# Important limitations

## Sampling is slow

The original sampling algorithm runs:

```text
x_T → x_T₋₁ → … → x₁ → x₀
```

With **T = 1000**, one image needs roughly 1000 network evaluations.

## The forward process is hand-designed

The variance schedule and step count are chosen manually.

## The simple loss is not the exact likelihood objective

The simplified MSE changes the weighting of the variational-bound terms.

## The model reflects its training data

It can reproduce dataset biases or be used to generate misleading content.

## Reverse transitions are approximate

Small denoising errors can accumulate across many steps.

## Metrics are incomplete

FID and Inception Score do not fully measure:

```text
semantic correctness
fairness
memorization
human preference
downstream usefulness
```

---

# Training versus sampling

```text
Training:
   start from a real image
   create one random noisy version
   predict the known noise
   update the network

Sampling:
   start from pure noise
   run every reverse step in order
   produce one image
```

Training is parallel across examples and time steps.

Sampling is sequential across diffusion time.

---

# A compact mental model

```text
Forward process:
   known destruction

Training:
   learn to recognize the destruction

Reverse process:
   repeatedly undo predicted destruction

Result:
   generate structured data from noise
```

---

# Bottom line

**DDPM works by turning image generation into a long sequence of small denoising decisions: corrupt real images with known Gaussian noise during training, teach a network to predict that noise at every corruption level, and run those learned corrections backward from pure noise to a new image.**

The memorable version:

```text
Forward diffusion asks:
   “How do I gradually erase an image?”

Training asks:
   “Can the network identify what was added?”

Reverse diffusion asks:
   “Can I remove the predicted noise one step at a time?”

Generation is:
   repeated learned denoising from random noise
```

---

## Primary source

Jonathan Ho, Ajay Jain, and Pieter Abbeel, [*Denoising Diffusion Probabilistic Models*](https://arxiv.org/abs/2006.11239), NeurIPS 2020.

Key sections used:

- Section 2: diffusion-model background
- Section 3.2: reverse-process noise prediction
- Section 3.4: simplified training objective
- Algorithm 1: training
- Algorithm 2: sampling
- Section 4: experiments and ablations
