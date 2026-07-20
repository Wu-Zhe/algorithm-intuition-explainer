# Attention Mechanism in *Attention Is All You Need*

```text
Mode: Source-grounded
Paper type: Hybrid
Focus: Theoretical / Method walkthrough of the attention mechanism
```

## The key intuition

**Attention lets each token decide which other tokens are useful, then build a new representation by mixing information from those tokens.**

Instead of:

```text
Process a sentence only through a step-by-step chain
token 1 → token 2 → token 3 → ...
```

the Transformer does:

```text
Let every token compare itself directly with every relevant token
token 1 ↔ token 2 ↔ token 3 ↔ ...
```

In simple terms:

> “Look at the whole context, decide what matters, and collect the useful information.”

The paper’s attention function takes a **query** and a set of **key–value pairs**. It gives each value a weight based on how well its key matches the query, then returns a weighted mixture of the values.

---

## What goes into attention?

For every token, the model creates three vectors:

```text
Query Q:
   “What information am I looking for?”

Key K:
   “What kind of information do I contain?”

Value V:
   “What information should I pass along?”
```

These are not three different words or three copies with fixed meanings.

They are three **learned views** of the same token representation:

```text
Q = XW_Q
K = XW_K
V = XW_V
```

where:

```text
X:
   the current token representations

W_Q, W_K, W_V:
   learned projection matrices
```

In self-attention, `Q`, `K`, and `V` all come from the same sequence.

---

## The full attention flow

```text
Input token vectors X
   ↓
Create queries, keys, and values
   Q = XW_Q
   K = XW_K
   V = XW_V

   In simple terms:
   “Give every token a question, a label for matching,
   and information it can share.”

   ↓
Compare every query with every key
   scores = QKᵀ

   In simple terms:
   “For each token, score how relevant every other token is.”

   ↓
Scale the scores
   scaled_scores = scores / sqrt(d_k)

   In simple terms:
   “Keep the scores from becoming too large.”

   ↓
Apply an optional mask
   masked_scores = apply_mask(scaled_scores)

   In simple terms:
   “Block connections that are not allowed.”

   ↓
Apply softmax
   attention_weights = softmax(masked_scores)

   In simple terms:
   “Turn the scores into proportions that add up to 1.”

   ↓
Mix the values
   output = attention_weights × V

   In simple terms:
   “Collect more information from highly relevant tokens
   and less from irrelevant tokens.”

   ↓
Context-aware token vectors
```

---

## The central equation

The paper calls this **scaled dot-product attention**:

\[
\operatorname{Attention}(Q,K,V)
=
\operatorname{softmax}
\left(
\frac{QK^\top}{\sqrt{d_k}}
\right)V
\]

The equation is trying to:

> Produce a context-aware vector for each query by selecting and mixing the available value vectors.

### Part 1: compare queries and keys

\[
QK^\top
\]

This says:

> “Compare every query with every key.”

A large dot product means the query and key point in similar learned directions, so the corresponding token may be relevant.

### Part 2: scale the comparisons

\[
\frac{QK^\top}{\sqrt{d_k}}
\]

This says:

> “Reduce the magnitude of the scores before softmax.”

When `d_k` is large, a dot product adds many products together and can become large in magnitude. Large scores can make softmax extremely sharp, leaving very small gradients for most choices. Dividing by `sqrt(d_k)` keeps the score scale better behaved.

### Part 3: turn scores into weights

\[
\operatorname{softmax}(\cdots)
\]

This says:

> “Convert relevance scores into nonnegative weights that add up to 1.”

For one query, the result might be:

```text
token A: 0.05
token B: 0.70
token C: 0.20
token D: 0.05
```

The query will borrow most heavily from token B.

### Part 4: mix the values

\[
\operatorname{softmax}(\cdots)V
\]

This says:

> “Use the attention weights to take a weighted average of the value vectors.”

Together:

> “Find the relevant tokens, then collect their information.”

---

## A tiny example

Consider:

```text
The animal did not cross the street because it was tired.
```

Focus on the token:

```text
it
```

Its query is compared with keys from all tokens:

```text
it → The
it → animal
it → did
it → street
it → tired
...
```

Suppose the learned scores become:

```text
animal:  3.2
street:  0.7
tired:   1.4
others:  smaller values
```

After scaling and softmax, the weights might look like:

```text
animal: 0.62
street: 0.08
tired:  0.20
others: 0.10 total
```

The new representation for `it` becomes a weighted mixture of the corresponding value vectors.

In simple terms:

> `it` becomes “it, understood using information mostly associated with animal.”

This is a teaching example, not a guarantee that a real trained head will produce exactly these weights or this linguistic interpretation.

---

## Shapes

Let:

```text
N       = number of tokens
d_model = model representation size
d_k     = query/key size
d_v     = value size
```

For one attention head:

```text
X:   N × d_model

W_Q: d_model × d_k
W_K: d_model × d_k
W_V: d_model × d_v
```

Therefore:

```text
Q = XW_Q: N × d_k
K = XW_K: N × d_k
V = XW_V: N × d_v
```

Now compare every query with every key:

```text
QKᵀ:
(N × d_k)(d_k × N)
= N × N
```

Why `N × N`?

Because every one of the `N` query tokens receives one score for each of the `N` key tokens.

Then:

```text
attention_weights: N × N
V:                 N × d_v

attention_weights × V:
(N × N)(N × d_v)
= N × d_v
```

So attention outputs one new vector for every input token.

### Numeric example

Suppose:

```text
N = 5 tokens
d_model = 512
d_k = 64
d_v = 64
```

Then:

```text
X:                 5 × 512
Q, K:              5 × 64
V:                 5 × 64
QKᵀ:               5 × 5
attention weights: 5 × 5
output:            5 × 64
```

Each row of the `5 × 5` attention matrix answers:

> “For this token, how much attention should go to each of the five tokens?”

---

## Why use multiple heads?

A single attention head creates one weighted mixture for each token.

That can be restrictive because one mixture has to combine every useful relationship at once.

**Multi-head attention runs several smaller attention operations in parallel.**

```text
Input X
   ↓
Head 1:
   create Q₁, K₁, V₁
   run scaled dot-product attention

Head 2:
   create Q₂, K₂, V₂
   run scaled dot-product attention

...

Head h:
   create Qₕ, Kₕ, Vₕ
   run scaled dot-product attention

   ↓
Concatenate all head outputs
   ↓
Apply one final learned projection
   ↓
Multi-head attention output
```

The paper defines:

\[
\operatorname{MultiHead}(Q,K,V)
=
\operatorname{Concat}(\text{head}_1,\ldots,\text{head}_h)W^O
\]

with:

\[
\text{head}_i
=
\operatorname{Attention}
(QW_i^Q,KW_i^K,VW_i^V)
\]

In simple terms:

> “Use several different learned ways of looking at the same sequence, then combine what they found.”

One head might become useful for nearby relationships, while another might become useful for longer-range relationships. This is an intuition, not a rule assigning a fixed linguistic job to each head.

In the original Transformer configuration:

```text
d_model = 512
h       = 8 heads
d_k     = 64 per head
d_v     = 64 per head
```

The eight `64`-dimensional head outputs are concatenated back into `512` dimensions before the final output projection.

---

## Where attention is used in the Transformer

The original Transformer uses multi-head attention in three ways.

### 1. Encoder self-attention

```text
Q, K, V all come from the encoder sequence.
```

Every input token can attend to every input token.

```text
input token representations
   ↓
encoder self-attention
   ↓
context-aware input representations
```

In simple terms:

> “Each input word reads the other input words.”

### 2. Masked decoder self-attention

```text
Q, K, V all come from the decoder sequence,
but future positions are blocked.
```

When predicting token `t`, the model may use tokens up to `t`, but not later tokens.

```text
previous output tokens
   ↓
masked self-attention
   ↓
representation that cannot see future output tokens
```

In simple terms:

> “Use the past, but do not cheat by looking at the future.”

### 3. Encoder–decoder attention

```text
Q comes from the decoder.
K and V come from the encoder output.
```

The decoder asks which input positions are useful for producing the current output.

```text
decoder state supplies Q
encoder output supplies K and V
   ↓
encoder–decoder attention
   ↓
decoder representation informed by the input
```

In simple terms:

> “While generating the output, look back at the relevant parts of the input.”

---

## Why is a mask applied before softmax?

Suppose the decoder is predicting position 4.

It may use:

```text
positions 1, 2, 3, and the current shifted position
```

It must not use:

```text
positions 5, 6, 7, ...
```

The blocked scores are set to a very negative value, conceptually `−∞`, before softmax:

```text
allowed score     → normal number
forbidden score   → −∞
```

After softmax:

```text
allowed connection   → possible positive weight
forbidden connection → weight 0
```

The important part is:

> Masking changes which information paths are allowed; it does not change the basic attention calculation.

---

## How does attention learn?

The attention weights are not stored as permanent rules.

They are calculated again for every input from the current `Q` and `K`.

What training learns are the projection matrices:

```text
W_Q
W_K
W_V
W_O
```

During normal Transformer training:

```text
Input sequence
   ↓
Compute attention and model predictions
   ↓
Compare predictions with target tokens
   ↓
Backpropagate the prediction error
   ↓
Update W_Q, W_K, W_V, W_O and other model parameters
```

Over training, the projections become useful because they produce queries and keys whose similarities help the model reduce its prediction loss.

In simple terms:

> “The model learns what questions to ask, what labels to match against, and what information to pass.”

---

## Why the mechanism works intuitively

The attention mechanism works because:

### 1. It creates direct communication paths

Any token can use information from any other allowed token in one attention step.

A recurrent model may need information to travel through many intermediate states. Attention creates a shorter path between distant positions.

### 2. It makes context dependent

A token does not have one fixed meaning inside the layer.

Its new representation depends on the other tokens present and on the learned attention weights.

```text
same token
+ different surrounding tokens
= different context-aware representation
```

### 3. It is selective

Softmax lets the model emphasize some value vectors and suppress others.

The model does not have to treat every token equally.

### 4. It processes token comparisons in parallel

During training, the queries for all positions can be compared with all keys using matrix multiplication.

The encoder does not need to wait for token 1 before processing token 2 in the way a standard recurrent chain does.

### 5. Multiple heads preserve several views

Different learned projections let the model create several attention patterns before combining them.

This reduces the pressure on one weighted average to capture every useful relationship.

---

## Conceptual pseudocode

```text
function scaled_dot_product_attention(X, mask):

    # X contains one vector for each token.
    # Shape: N × d_model

    Q = X × W_Q
    K = X × W_K
    V = X × W_V

    # Compare every query token with every key token.
    # Shape: N × N
    scores = Q × transpose(K)

    # Keep score magnitudes well behaved.
    scores = scores / sqrt(d_k)

    # Prevent forbidden connections, such as looking into the future.
    if mask exists:
        scores[forbidden_positions] = negative_infinity

    # Convert each row into weights that add up to 1.
    weights = softmax(scores, across_keys)

    # Collect information from the value vectors.
    # Shape: N × d_v
    output = weights × V

    return output
```

Multi-head attention:

```text
function multi_head_attention(X, mask):

    head_outputs = []

    for each head i:

        # Each head has its own learned projections.
        head_i = scaled_dot_product_attention_with_head_i(X, mask)

        append head_i to head_outputs

    combined = concatenate(head_outputs)

    # Mix information from all heads and return to d_model dimensions.
    output = combined × W_O

    return output
```

---

## What the simple explanation leaves out

### Attention does not know word order by itself

Without additional position information, self-attention treats token relationships without inherently knowing which token came first. The Transformer adds positional encodings to token embeddings before the attention stack.

### Attention is not the whole Transformer layer

The original architecture also uses:

```text
residual connections
layer normalization
position-wise feed-forward networks
embeddings
positional encodings
```

The core attention operation ends with the weighted mixture of values, but the surrounding components are important for the full model.

### Attention weights are not guaranteed explanations

A large attention weight shows that one value contributed strongly through that head at that layer. It does not automatically prove a human-interpretable reason or a causal explanation for the final prediction.

### Standard full attention has quadratic pairwise comparisons

The `N × N` score matrix compares every token with every token. This is powerful, but its memory and computation grow quickly with sequence length.

### A weighted average can blur information

Attention mixes value vectors. Multi-head attention helps provide several different mixtures, but it does not remove every possible information bottleneck.

---

## Bottom line

**Attention works by letting each token ask what it needs, match that request against what other tokens offer, and build a new representation from the most useful information.**

The memorable version:

```text
Query:
   “What am I looking for?”

Key:
   “How well do I match that request?”

Value:
   “What information will I contribute if selected?”
```

---

## Primary source

Vaswani, A. et al. (2017), [*Attention Is All You Need*](https://arxiv.org/abs/1706.03762), especially Sections 3.2–3.2.3 and Section 4.
