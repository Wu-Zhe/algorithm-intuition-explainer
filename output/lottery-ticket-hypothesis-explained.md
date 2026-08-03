# The Lottery Ticket Hypothesis — Simple Explanation

**Paper:** Jonathan Frankle and Michael Carbin, *The Lottery Ticket Hypothesis: Finding Sparse, Trainable Neural Networks* (ICLR 2019)  
**Source:** [arXiv PDF](https://arxiv.org/pdf/1803.03635)  
**Source mode:** Source-grounded  
**Paper type:** Hybrid — Method + Empirical

## The central idea

A randomly initialized neural network may contain a much smaller subnetwork that can train successfully on its own—but only if it keeps the right subset of connections and their original initial weights.

The authors call such a subnetwork a **winning ticket**.

Importantly, a winning ticket is not merely a small architecture. It is the combination:

**winning ticket = connection pattern + original initialization**

Randomizing the surviving weights often destroys its advantage.

## The problem

Pruning can remove over 90% of a trained network's parameters without seriously reducing accuracy. But this does not mean we can simply initialize the resulting sparse architecture randomly and train it from scratch: sparse networks often optimize poorly.

The paper asks:

> Was the large network truly necessary, or did it merely help us locate a small, unusually trainable subnetwork?

## How they find winning tickets

Let **θ₀** be the dense network's initial weights and **m** a binary mask indicating which weights remain.

The sparse network is:

**f(x; m ⊙ θ₀)**

Here, **m ⊙ θ₀** keeps weights where the mask is 1 and permanently removes weights where it is 0.

The search procedure is:

```text
Randomly initialize a dense network → save θ₀
                    ↓
Train the complete network
                    ↓
Remove the smallest-magnitude weights
                    ↓
Reset every surviving weight to its value in θ₀
                    ↓
Train this sparse network from the beginning
```

The reset is the paper's crucial step. Conventional pruning normally keeps the surviving weights' trained values and fine-tunes them. This experiment rewinds them all the way to their initial values.

### Iterative pruning

Rather than remove most weights at once, the authors repeat the cycle:

```text
train → prune → reset → train → prune → reset → ...
```

If 20% of the remaining weights are pruned each round, the fraction left after *k* rounds is **0.8ᵏ**. After ten rounds, about **10.7%** remain.

Iterative pruning generally discovers smaller successful subnetworks than one-shot pruning, but it is expensive because the network must be trained repeatedly.

## What qualifies as a winning ticket?

Suppose the original network reaches test accuracy **a** after **j** training iterations. A sparse subnetwork is a winning ticket if it:

- uses substantially fewer parameters;
- reaches at least accuracy **a**;
- requires no more than **j** iterations;
- starts from its corresponding values in the original initialization.

This definition combines model size, accuracy, and optimization speed. It is stronger than merely showing that a trained model can be compressed.

## What the experiments found

The authors tested fully connected and convolutional networks on MNIST and CIFAR-10, including LeNet, several VGG-style networks, VGG-19, and ResNet-18.

Their main observations were:

- They consistently found subnetworks with roughly **10–20% or less** of the original weights that matched the dense model's accuracy.
- Moderate pruning sometimes made winning tickets learn faster and achieve slightly higher test accuracy.
- Excessive pruning eventually reduced both training speed and accuracy.
- Keeping the sparse structure but randomly reinitializing its weights usually performed much worse at high sparsity. This supports the importance of the original initialization.
- Iterative pruning found smaller winning tickets than one-shot pruning.

The relationship therefore looks like a hill:

```text
No pruning          Moderate pruning          Extreme pruning
too much capacity → best performance region → too little capacity
```

Moderate pruning may remove unnecessary or harmful degrees of freedom. Beyond a threshold, the network no longer has enough useful connections.

## Why might winning tickets exist?

The paper does not establish a definitive mechanism, but offers a plausible interpretation.

A large random network samples many weights and therefore implicitly contains many possible subnetworks. A few of those subnetworks may begin in regions of parameter space where gradient-based optimization works especially well.

Training the dense model reveals which connections become important. Magnitude pruning then uses that information to search backward for a promising subnetwork.

This does **not** mean the winning weights were already close to their trained values. The paper reports that they can move farther during training than other weights. Their benefit may instead be that their starting configuration creates favorable optimization dynamics.

## Hypothesis versus conjecture

The paper makes two claims with different evidential status:

- **Lottery ticket hypothesis:** Dense, randomly initialized networks contain sparse, well-initialized subnetworks that can match the dense network when trained independently. The experiments directly support this for the tested settings.
- **Lottery ticket conjecture:** SGD succeeds on overparameterized networks because it finds and trains one of these subnetworks. The authors explicitly present this as an untested explanation, not an experimental conclusion.

That distinction is easy to miss: demonstrating that winning tickets exist does not prove that ordinary dense-network training operates by selecting one.

## Important limitations

The original work does not provide a cheap way to train small networks:

- Finding a ticket first requires training the dense network—often repeatedly.
- The experiments cover MNIST and CIFAR-10, not large-scale datasets such as ImageNet.
- The masks are unstructured: individual weights are removed irregularly, so fewer parameters do not automatically translate into proportional hardware speedups.
- Results on deeper networks depend strongly on learning rate. VGG-19 and ResNet-18 required learning-rate warmup or modified learning rates; the method could not find ResNet-18 tickets at its original learning rate of 0.1.
- The evidence establishes existence in the tested architectures, not that every architecture, task, or initialization contains a practically useful ticket.

