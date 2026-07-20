# Forward Projection in *Closed-Form Feedback-Free Learning with Forward Projection*

> **Formatting note:** Equations use fenced plain-text notation so this file renders correctly in Markdown viewers that do not support LaTeX.

```text
Mode: Source-grounded
Paper type: Hybrid
Focus: The Forward Projection training algorithm
```

## The key intuition

**Forward Projection trains each neural-network layer by giving it a locally generated, label-informed target, then solving directly for the weights that reproduce that target.**

Instead of:

```text
Run the whole network
   ↓
Measure the final error
   ↓
Send corrections backward
   ↓
Update every layer a little
   ↓
Repeat for many epochs
```

it does:

```text
Create a useful target for layer 1
   ↓
Solve layer 1 directly
   ↓
Create a useful target for layer 2
   ↓
Solve layer 2 directly
   ↓
Continue forward
```

The paper calls this algorithm **Forward Projection**. Its hidden-layer targets combine randomized nonlinear projections of the current layer input and the correct label. Each layer is then fit locally by closed-form ridge regression, without receiving an error signal from later layers.

---

## The problem being solved

For a hidden layer, we normally do not know what its neurons *should* output.

We know:

```text
Input X
Correct label Y
```

But we do not directly know:

```text
Ideal hidden representation H1
Ideal hidden representation H2
```

Backpropagation solves this by calculating how the final error depends on every earlier weight.

Forward Projection takes a different approach:

> “Instead of discovering the perfect hidden target by sending errors backward, construct a useful hidden target directly from the input and label.”

The method is intentionally stricter than many local-learning approaches: the weights entering a layer are fit without feedback from that layer’s outputs or from downstream layers.

---

## The complete training flow

For a network with two hidden layers:

```text
Training data X + labels Y
   ↓
Generate target T1 for hidden layer 1
   T1 contains:
      a random nonlinear code of X
      +
      a random nonlinear code of Y

   ↓
Fit W1 with ridge regression
   Find W1 such that:
      XW1 ≈ T1

   In simple terms:
   “Make layer 1 reproduce the input-and-label clue.”

   ↓
Compute the real hidden activation
   Z1 = XW1
   H1 = activation(Z1)

   ↓
Generate target T2 for hidden layer 2
   T2 contains:
      a random nonlinear code of H1
      +
      a random nonlinear code of Y

   ↓
Fit W2 with ridge regression
   Find W2 such that:
      H1W2 ≈ T2

   In simple terms:
   “Make layer 2 reproduce a new clue built from
   the improved representation and the label.”

   ↓
Compute the second hidden activation
   Z2 = H1W2
   H2 = activation(Z2)

   ↓
Fit the final readout
   Find Wout such that:
      H2Wout ≈ Y

   ↓
Trained network
```

The order is **forward, layer by layer**. The first layer must be fitted before the second layer because the second target depends on the first layer’s activations.

---

## Step 1: Generate a local target

For layer `l`, the paper defines the target pre-activation as:

```text
Z_tilde_l = g_l(A_(l-1) Q_l) + g_l(Y U_l)
```

The equation is trying to:

> Create a hidden-layer target that contains both information about the current input and information about the correct answer.

This target-generation rule is the central idea.

### The input part

```text
g_l(A_(l-1) Q_l)
```

This says:

> “Preserve a coded version of what is entering the layer.”

Here:

```text
A_(l-1):
   activations entering the current layer

Q_l:
   a fixed random projection matrix

g_l:
   an element-wise nonlinear coding function
```

The random matrix `Q_l` is selected before training and is not learned.

This term helps prevent every example with the same label from being forced into exactly the same hidden target.

In simple terms:

> “Do not forget the individual input.”

### The label part

```text
g_l(Y U_l)
```

This says:

> “Add a coded version of the correct answer.”

Here:

```text
Y:
   the training labels, usually one-hot vectors

U_l:
   a fixed random label-projection matrix
```

Examples with the same label receive the same label-code contribution for that layer.

In simple terms:

> “Move examples from the same class toward some shared structure.”

### Together

```text
Z_tilde_l =
    g_l(A_(l-1) Q_l)   # retain input information
  + g_l(Y U_l)         # inject label information
```

Together they say:

> “Construct a target that remembers the example while also pointing toward its class.”

The resulting target is a useful heuristic representation rather than the uniquely correct hidden representation.

---

## Step 2: Fit the layer with ridge regression

Once the target `Z_tilde_l` has been created, Forward Projection chooses `W_l` so that:

```text
A_(l-1) W_l ≈ Z_tilde_l
```

The closed-form solution is:

```text
W_l = (A_(l-1)^T A_(l-1) + λI)^-1 A_(l-1)^T Z_tilde_l
```

This is ridge regression.

The first part says:

```text
A_(l-1)^T A_(l-1)
```

> “Summarize how the incoming features relate to one another.”

The second part says:

```text
A_(l-1)^T Z_tilde_l
```

> “Summarize how the incoming features relate to the desired target.”

The regularization term says:

```text
λI
```

> “Avoid an unstable solution with excessively large weights.”

The inverse and multiplication say:

> “Directly calculate the weights that best reproduce the clue.”

Unlike gradient descent, this does not make thousands of small updates. It solves the local fitting problem once after accumulating the required statistics.

---

## Step 3: Compute the actual hidden representation

After solving for `W_l`:

```text
Z_l = A_(l-1) W_l
```

and then:

```text
A_l = f_l(Z_l)
```

Here:

```text
Z_l:
   the raw linear output, or pre-activation potential

f_l:
   the actual neural-network activation function

A_l:
   the representation passed into the next layer
```

For example, if `f_l` is ReLU:

```text
f_l(z) = max(0, z)
```

In simple terms:

> “First reproduce the local clue as closely as possible, then transform it into the features used by the next layer.”

A subtle but important detail is that the target coding function `g_l` does not have to equal the neural activation `f_l`.

---

## Shapes

Suppose layer `l` has:

```text
N:
   number of training examples

m_(l-1):
   number of inputs to the layer

m_l:
   number of neurons in the current layer

C:
   number of output classes
```

Then:

```text
A_(l-1): N × m_(l-1)
Y:       N × C

Q_l:     m_(l-1) × m_l
U_l:     C × m_l
```

The random input code has shape:

```text
A_(l-1) Q_l:
(N × m_(l-1))(m_(l-1) × m_l)
= N × m_l
```

The random label code has shape:

```text
Y U_l:
(N × C)(C × m_l)
= N × m_l
```

Therefore:

```text
T_l = Z̃_l:
N × m_l
```

The learned layer weight has shape:

```text
W_l:
m_(l-1) × m_l
```

So:

```text
A_(l-1) W_l:
N × m_l
```

which has the same shape as the target.

---

## Tiny example

Suppose:

```text
100 training examples
4 input features
2 classes
5 neurons in hidden layer 1
```

Then:

```text
X: 100 × 4
Y: 100 × 2

Q1: 4 × 5
U1: 2 × 5
```

Generate the input code:

```text
XQ1:
(100 × 4)(4 × 5)
= 100 × 5
```

Generate the label code:

```text
YU1:
(100 × 2)(2 × 5)
= 100 × 5
```

Add them:

```text
T1 = g(XQ1) + g(YU1)

T1:
100 × 5
```

Then solve:

```text
W1:
4 × 5

so that:

XW1 ≈ T1
```

For one example, the target might conceptually look like:

```text
input code:
[ 0.8, -1.0, 0.5, 0.2, -0.6 ]

label code:
[ 1.0,  0.4, 0.7, -0.3, 0.9 ]

combined target:
[ 1.8, -0.6, 1.2, -0.1, 0.3 ]
```

The exact numbers are arbitrary because the projections are random. What matters is that the same fixed coding system is used throughout training.

---

## Why does this work intuitively?

### 1. The target is learnable from the current input

The layer is not asked to reproduce a target made only from the label.

The target also contains:

```text
g_l(A_(l-1) Q_l)
```

which is directly constructed from the layer’s own input.

That gives the regression problem a strong input-predictable component.

In simple terms:

> “A large part of the clue is something the layer can already infer from what it sees.”

### 2. The target contains task information

The term:

```text
g_l(Y U_l)
```

places information about the correct label inside the target.

Therefore, the fitted representation is not merely a random transformation of the input. It is encouraged to contain structure useful for the task.

In simple terms:

> “The layer preserves the input, but bends its representation toward the correct answer.”

### 3. Each later layer starts from a better representation

After layer 1 has been fitted:

```text
raw input X
   ↓
label-informed H1
```

Layer 2 constructs its target from `H_1`, not from the original raw input:

```text
label-informed H1
   ↓
new label-informed target T2
   ↓
more task-oriented H2
```

The representation is expected to move progressively from predominantly input information in early layers toward greater label information in later layers.

### 4. The nonlinear activation creates new features

Without an activation function, stacking several fitted linear layers would still reduce to one linear transformation.

The activation:

```text
A_l = f_l(A_(l-1) W_l)
```

changes the feature space between regression steps.

This lets later layers fit relationships that were not linearly available in the original input.

### 5. Ridge regression gives a stable direct solution

The regularization parameter `λ` discourages huge weights and improves the conditioning of the matrix solve.

In simple terms:

> “Match the target, but do not overreact to the training data.”

---

## Is the target the perfect hidden representation?

No.

This does not mean the random target is mathematically guaranteed to be the unique best internal representation.

It means:

```text
The target retains input information
   +
The target contains label information
   +
The target can be fitted locally
   =
A useful supervised hidden representation
```

The paper provides a theoretical comparison in a simplified two-layer setting under specific assumptions. That is not a universal proof that every Forward Projection network will outperform backpropagation or every other model.

---

## What does “single pass” mean?

The precise interpretation is:

> **One pass over the dataset for each layer, with one closed-form weight solve per layer.**

It does not mean that all layers can be solved simultaneously in one traversal.

For a two-hidden-layer network:

```text
Pass through data to collect layer-1 statistics
   ↓
Solve W1
   ↓
Use W1 to produce H1
   ↓
Pass through data to collect layer-2 statistics
   ↓
Solve W2
   ↓
Fit final readout
```

The matrices needed for ridge regression can be accumulated batch by batch:

```text
A_(l-1)^T A_(l-1)
```

and:

```text
A_(l-1)^T Z_tilde_l
```

Therefore, the full activity and target matrices do not necessarily have to be retained in memory.

---

## Training versus inference

The labels are needed to construct the hidden targets **during training**:

```text
Training:
   input X + correct label Y
   ↓
   generate local targets
   ↓
   solve weights
```

After training, inference is an ordinary forward pass:

```text
New input X
   ↓
H1 = activation(XW1)
   ↓
H2 = activation(H1W2)
   ↓
prediction = H2Wout
```

The label is not needed at inference time.

---

## Conceptual pseudocode

```text
function train_forward_projection(X, Y, hidden_sizes, lambda):

    A = X

    for each hidden layer l:

        # Fixed random projections.
        Q_l = random_matrix(
            input_dimension(A),
            hidden_sizes[l]
        )

        U_l = random_matrix(
            label_dimension(Y),
            hidden_sizes[l]
        )

        # Construct a local target containing
        # both input information and label information.
        T_l = g_l(A × Q_l) + g_l(Y × U_l)

        # Accumulate sufficient statistics.
        input_gram = transpose(A) × A
        input_target = transpose(A) × T_l

        # Solve the local ridge-regression problem once.
        W_l = inverse(
            input_gram + lambda × identity
        ) × input_target

        # Produce the representation used by the next layer.
        Z_l = A × W_l
        A = activation_l(Z_l)

        save W_l, Q_l, U_l

    # Fit the final representation directly to the labels.
    W_output = ridge_regression(A, Y, lambda)

    return hidden weights and W_output
```

A streaming implementation replaces the full matrix products with batchwise accumulation:

```text
S_l = 0
R_l = 0

for each batch (A_batch, Y_batch):

    T_batch =
        g_l(A_batch × Q_l)
        +
        g_l(Y_batch × U_l)

    S_l += transpose(A_batch) × A_batch
    R_l += transpose(A_batch) × T_batch

W_l = inverse(S_l + lambda × identity) × R_l
```

---

## What the experiments support

Across the evaluated MLP, convolutional, sequence, and limited Transformer settings, Forward Projection was generally stronger than the other evaluated feedback-free or local-learning alternatives, although standard backpropagation remained stronger in several conventional full-data settings.

The paper reports particularly strong few-shot results on its chest X-ray and retinal OCT tasks. These are results on the studied datasets and architectures, not evidence that the method will always dominate backpropagation in low-data problems.

For one three-hidden-layer Fashion-MNIST MLP experiment, the authors report a 66× wall-clock training speedup. That number is an example from a particular architecture, implementation, device, and training configuration—not a universal speedup factor.

---

## Important limitations

The closed-form solve requires matrix inversion or an equivalent linear-system solution. For a dense width-`m` layer, this can become expensive for very wide layers even though it is performed only once per layer.

One-step learning through matrix inversion is not necessarily biologically plausible, despite removing backward neural communication.

The random matrices `Q_l` and `U_l` are not improved during standard closed-form training. The representation improves because each successive target is constructed from a previously fitted activation—not because the random clue generator itself learns.

---

## Bottom line

**Forward Projection works by giving every hidden layer a target that mixes “what this layer sees” with “what the correct answer is,” then using ridge regression to solve that layer directly before moving forward.**

---

## Primary source

O’Shea, R. and Rajendran, B., *Closed-Form Feedback-Free Learning with Forward Projection*, arXiv:2501.16476.
