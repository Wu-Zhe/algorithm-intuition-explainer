# RecGPT-V1: Intent-Centered Recommendation

> **Source mode:** Source-grounded  
> **Paper type:** Hybrid — method, production system, and empirical evaluation  
> **Formatting note:** Mathematical expressions use portable Unicode notation, so this file renders correctly without a LaTeX or MathJax extension.

## The key intuition

**RecGPT-V1 tries to recommend from an explicit understanding of what the user currently wants, rather than relying only on patterns such as “people who clicked this also clicked that.”**

Instead of:

```text
Past clicks and purchases
   ↓
Learn statistical co-occurrence patterns
   ↓
Retrieve items similar users previously consumed
```

RecGPT does:

```text
Long user behavior history
   ↓
Infer explicit user interests in natural language
   ↓
Predict the kinds of items the user may want next
   ↓
Retrieve real catalog items matching those intentions
   ↓
Explain why each item was recommended
```

In simple terms:

> “First reason about the user’s intent, then use that intent to guide retrieval.”

The system does not discard traditional collaborative recommendation. It combines LLM-derived semantic intent with existing user–item behavioral signals.

---

## The problem RecGPT-V1 is solving

Traditional recommender systems are very good at learning historical correlations:

```text
User clicked shoes
   ↓
Recommend more shoes
```

But this can miss the reason behind the behavior:

```text
The user may be:
   preparing for a marathon
   buying a gift
   replacing damaged footwear
   exploring a new style
```

Two users can click the same item for different reasons.

A system trained only to reproduce historical clicks may therefore:

- overfit narrow past preferences;
- repeatedly show similar products;
- miss new or latent interests;
- favor already-popular items;
- struggle to explain its recommendations.

RecGPT-V1 changes the intermediate representation.

Instead of representing the user only as a hidden vector, it explicitly generates:

```text
A readable interest profile
   +
A set of predicted item tags
```

Those explicit semantic objects are then connected to the production retrieval system.

---

# The complete RecGPT-V1 flow

```text
Raw lifelong user behavior
   ↓
1. Reliable behavior extraction
   Remove noisy or weak signals.
   Keep stronger signals such as searches, purchases,
   favorites, add-to-cart events, and detailed views.

   In simple terms:
   “Keep actions that more clearly reveal intent.”

   ↓
2. Hierarchical behavior compression
   Compress item details and group behavior by time,
   behavior type, and recurring item patterns.

   In simple terms:
   “Turn an enormous history into a shorter,
   information-dense story.”

   ↓
3. User-interest LLM
   Generate an explicit natural-language profile
   of the user’s current and latent interests.

   In simple terms:
   “Explain what this person may presently care about.”

   ↓
4. Item-tag LLM
   Predict semantic descriptions of items
   the user may want next.

   In simple terms:
   “Translate the user profile into shopping intentions.”

   ↓
5. User–item–tag retrieval
   Combine:
      behavioral similarity from user–item history
      +
      semantic similarity between predicted tags and items

   In simple terms:
   “Retrieve items that fit both the user’s behavior
   and the inferred intent.”

   ↓
6. Existing ranking and reranking system
   Score and order the retrieved candidates.

   ↓
7. Recommendation-explanation LLM
   Connect a selected item to a user interest
   and generate a readable explanation.

   In simple terms:
   “Tell the user why the item may be useful.”

   ↓
Recommended item + personalized explanation
```

The central contribution is the **closed intent loop**:

```text
User behavior
   ↓
User interest
   ↓
Predicted item tags
   ↓
Catalog items
   ↓
Personalized explanation
```

---

# Step 1: Compress the user’s behavior without losing the signal

Real users can have extremely long histories.

The paper reports that Taobao users have more than 37,000 historical behavior records on average, which can exceed an LLM’s usable context.

Simply truncating the history can remove important long-term interests.

RecGPT therefore performs two kinds of filtering and compression.

## Reliable behavior extraction

It emphasizes stronger indicators of intent, including:

```text
purchases
favorites
add-to-cart actions
search queries
detailed product views
review reading
```

It excludes ordinary clicks from this high-confidence set because many clicks may be accidental, shallow, or influenced by page layout.

In simple terms:

> “Not every click deserves equal trust.”

## Item-level compression

A product page may contain large amounts of text.

The system compresses each item into its core attributes:

```text
item name
category
brand
important features
```

In simple terms:

> “Keep what identifies the product; remove decorative detail.”

## Sequence-level compression

The system groups behavior using time and behavior type, then reorganizes repeated item patterns.

Conceptually:

```text
Raw sequence:
   thousands of individual actions

   ↓ group by time and action type

Daily / monthly / yearly behavior groups

   ↓ merge recurring item patterns

Compressed behavioral sequence
```

The paper reports that this compression increases the fraction of user histories fitting within a 128K-token context from 88% to 98%, while improving interest-inference efficiency by 29%.

The important part is:

> Compression is not the final recommendation algorithm. It makes the intent-reasoning step computationally possible.

---

# Step 2: Generate an explicit user-interest profile

The user-interest model, called **LLM_UI**, receives:

```text
User attributes
   +
Compressed reliable behavior sequence
   +
Prompt instructions
```

and generates a set of explicit interests.

Conceptually:

**Iᵤ = LLM_UI(Aᵤ, Bᵤ | P_UI)**

where:

- **Aᵤ** is information about user *u*;
- **Bᵤ** is the compressed behavior history;
- **P_UI** is the interest-mining prompt;
- **Iᵤ** is the generated interest profile.

The equation is trying to say:

> “Use the user’s attributes and reliable behavior to write down the interests that best explain the observed actions.”

A simplified output could look like:

```text
Current interests:
- beginner-friendly trail running
- lightweight outdoor equipment
- products suitable for hot weather
- value-conscious sports purchases
```

This is more interpretable than an anonymous embedding vector.

Experts and automated judges can inspect whether the generated profile is:

```text
supported by behavior
specific enough to be useful
diverse rather than repetitive
reasonable rather than speculative
```

---

# Step 3: Predict item tags from the inferred interests

The user-interest profile is still too abstract to retrieve concrete catalog items.

RecGPT therefore uses a second model, **LLM_IT**, to predict item tags.

Conceptually:

**Tᵤ = LLM_IT(Aᵤ, Iᵤ, Sᵤ | P_IT)**

where:

- **Aᵤ** is user information;
- **Iᵤ** is the inferred interest profile;
- **Sᵤ** is the multi-behavior interaction sequence;
- **P_IT** is the item-tag prediction prompt;
- **Tᵤ** is a set of predicted item tags.

Suppose the interest profile says:

```text
The user is beginning trail running
and prefers lightweight equipment.
```

The item-tag model might produce:

```text
lightweight trail-running shoes
breathable running hydration vest
beginner trail gaiters
compact soft water flask
quick-dry running shirt
```

In simple terms:

> “The interest model says what the user cares about; the tag model says what kinds of products could satisfy it.”

The tags describe desired items semantically. They are not necessarily exact catalog names.

That creates the next problem:

```text
Predicted semantic tag
   ≠
Specific item ID in the live catalog
```

The retrieval module bridges this gap.

---

# Step 4: Retrieve real items with three towers

RecGPT introduces a **User–Item–Tag Retrieval Framework**.

It has three representation towers:

```text
User tower
   represents behavioral preference

Item tower
   represents each catalog item

Tag tower
   represents the LLM-predicted semantic intent
```

## User tower

The user tower encodes user identity and interaction sequences.

It learns from collaborative behavior:

```text
Which users interacted with which items?
```

Its strength is exploiting established patterns.

## Item tower

The item tower encodes features such as:

```text
item ID
category
brand
price
sales
other product attributes
```

The same item representation must work with both user and tag signals.

## Tag tower

The tag tower encodes the semantic item descriptions generated by **LLM_IT**.

Its strength is exploring items that match inferred intent, even when they are not obvious from historical co-occurrence.

---

## Two objectives train the retrieval space

The retrieval system combines two kinds of learning.

### Collaborative objective

**L_col**

This says:

> “Place users near items they historically interacted with.”

It preserves the useful behavior patterns already learned by conventional recommendation.

### Semantic objective

The semantic objective has two parts:

**L_sem = αL_tag + (1 − α)L_cate**

where:

- **L_tag** aligns predicted tags with matching items;
- **L_cate** encourages useful category-level distinctions;
- **α** controls the balance between the two semantic losses.

The full objective is:

**L_TAR = L_col + αL_tag + (1 − α)L_cate**

The equation is trying to say:

> “Learn an item space that is useful both for historical user behavior and for LLM-generated semantic intent.”

The first term says:

> “Do not throw away collaborative evidence.”

The remaining terms say:

> “Also make semantically appropriate tags land near the right products.”

---

# Step 5: Fuse behavior and intent during retrieval

At inference time, RecGPT combines the user representation and the tag representation.

**h_fuse = βhᵤ + (1 − β)hₜ**

where:

- **hᵤ** is the user-tower representation;
- **hₜ** is the tag-tower representation;
- **β** controls how much the system trusts behavioral history versus semantic intent.

The same idea can be written at the score level:

**ŷ_final = βŷ_col + (1 − β)ŷ_sem**

The first score says:

> “Is this item behaviorally likely for the user?”

The second score says:

> “Does this item semantically fit the inferred intent?”

Together:

> “Prefer items supported by behavior, semantic intent, or both.”

## What β means

When **β is large**:

```text
More weight on collaborative history
   ↓
safer and more familiar recommendations
```

When **β is small**:

```text
More weight on semantic intent
   ↓
more exploration of latent or emerging interests
```

This provides a controllable exploration–exploitation tradeoff.

The important part is:

> RecGPT is not “an LLM that directly outputs product IDs.” The LLM supplies semantic intent, while a production retrieval model maps that intent to catalog items efficiently.

---

# Step 6: Rank candidates with the existing system

The tri-tower model retrieves a manageable set of candidate items.

Those candidates then pass through the platform’s downstream ranking and reranking stack.

```text
Huge item catalog
   ↓ tri-tower retrieval
Small candidate set
   ↓ existing ranker
Ordered recommendations
```

This design is operationally important.

RecGPT changes candidate generation without requiring the platform to replace every downstream ranking component.

In simple terms:

> “Use LLM reasoning to widen and improve the candidate pool, then reuse the mature ranking machinery.”

---

# Step 7: Generate personalized explanations

A third model, **LLM_RE**, generates a user-facing explanation.

Its inputs include:

```text
User interest
   +
Recommended item information
   +
Date or contextual information
```

The model follows two conceptual steps:

```text
Understand the connection
   ↓
Generate a short, conversational explanation
```

For example:

```text
Interest:
   lightweight beginner trail-running equipment

Item:
   breathable hydration vest

Explanation:
   “A lightweight vest like this can carry water
   without adding much bulk on shorter trail runs.”
```

The paper describes an offline production strategy that generates and caches interest–item–explanation mappings.

Why offline?

```text
LLM generation is relatively expensive
   ↓
Generate explanations before the request
   ↓
Store them in a lookup table
   ↓
Serve them with low online latency
```

The explanation model is instructed not to invent a personal connection when one is unsupported. In that case, it can describe the item’s inherent qualities instead.

---

# How the LLM modules are trained

General-purpose LLMs do not automatically understand a platform’s product taxonomy, behavior semantics, quality rules, or operational goals.

RecGPT uses a progressive alignment process.

## Stage 1: Curriculum-based task preparation

For user-interest mining, the model first learns smaller prerequisite tasks such as:

```text
extract key information
understand product attributes
analyze user profiles
perform causal reasoning
```

These tasks are ordered from simpler prerequisites to more complex recommendation reasoning.

In simple terms:

> “Teach the component skills before asking for the full job.”

## Stage 2: Reasoning-enhanced pre-alignment

A stronger reasoning model generates candidate training examples.

Human experts curate the examples before they are used to fine-tune the production model.

```text
Strong teacher model
   ↓
Generate reasoned examples
   ↓
Human quality filtering
   ↓
Fine-tune the task model
```

In simple terms:

> “Distill expensive reasoning into a model designed for production.”

## Stage 3: Self-training evolution

The aligned model generates additional examples itself.

A quality-control system accepts useful outputs and rejects weak ones.

```text
Current model
   ↓
Generate new candidate training data
   ↓
Human / LLM judge filters the data
   ↓
Retrain the model
   ↓
Improved model
```

This creates an iterative improvement loop.

## Incremental learning

For item-tag prediction, online feedback is periodically cleaned, balanced, and used to update the model.

This helps the system respond to:

```text
new products
new trends
seasonal changes
shifting user interests
```

The paper still identifies periodic supervised updates as a limitation because they cannot fully adapt continuously.

---

# The Human–LLM cooperative judge

RecGPT uses generated text at several stages:

```text
user interests
item tags
recommendation explanations
```

These outputs cannot be judged only by exact string matching.

A generated interest may be phrased differently while still being correct.

The paper therefore introduces task-specific LLM judges trained from human annotations.

## Judge flow

```text
Generated task outputs
   ↓
Human labels based on quality criteria
   ↓
Judge data buffer
   ↓
Train a task-specific LLM judge
   ↓
Use the judge for data filtering and evaluation
   ↓
Periodically compare the judge with humans
   ↓
Realign when performance drifts
```

Different tasks use different criteria.

Examples include:

```text
User interests:
   willingness
   reasonableness

Item tags:
   relevance
   consistency
   specificity
   validity

Explanations:
   relevance
   factuality
   clarity
   safety
```

The important part is:

> “LLM-as-a-judge” is not treated as permanently trustworthy.

Human reviewers remain in the loop because:

- domain knowledge changes;
- new products appear;
- user behavior shifts;
- business quality standards evolve;
- a static judge can become biased or stale.

---

# A tiny end-to-end example

Suppose a user recently:

```text
searched for “beginner trail routes”
bought ordinary running socks
viewed lightweight hiking shoes
saved a compact water bottle
```

## 1. Behavior compression

```text
Recent high-intent outdoor fitness activity:
- beginner trail search
- lightweight footwear exploration
- hydration accessory interest
```

## 2. User-interest mining

```text
The user may be preparing to begin recreational trail running
and prefers lightweight, beginner-friendly equipment.
```

## 3. Item-tag prediction

```text
lightweight trail-running shoes
beginner hydration vest
quick-dry trail socks
compact soft flask
```

## 4. Retrieval

```text
Collaborative signal:
   items chosen by users with similar behavior

Semantic signal:
   catalog items close to the predicted tags

Combined result:
   items supported by history, intent, or both
```

## 5. Explanation

```text
“This lightweight vest may suit shorter beginner trail runs
where you want to carry water without a bulky backpack.”
```

This example illustrates the mechanism. It is not a literal trace from the paper’s production system.

---

# Why the approach works intuitively

## 1. It creates an explicit semantic bridge

Traditional recommendation jumps directly from behavior to item scores.

RecGPT inserts interpretable stages:

```text
Behavior
   ↓
Interest
   ↓
Desired item type
   ↓
Actual item
```

Each stage transforms an ambiguous problem into a more specific one.

## 2. It combines exploitation with exploration

Collaborative signals answer:

> “What is already likely based on behavior?”

Semantic signals answer:

> “What else makes sense given the inferred intent?”

The fusion mechanism keeps the system from relying entirely on either one.

## 3. It can use world and product knowledge

An LLM can connect concepts that may have weak co-occurrence in the interaction logs.

For example:

```text
new parent
   ↓
sleep disruption
   ↓
quiet, low-light, time-saving product needs
```

The connection may be semantically reasonable even before enough users have produced a strong click pattern.

## 4. Intermediate outputs can be inspected

A poor recommendation can be traced backward:

```text
Was the user interest wrong?
Was the predicted tag too broad?
Did retrieval map the tag incorrectly?
Was the final explanation unsupported?
```

This enables process-level supervision instead of evaluating only the final click.

## 5. The system preserves production efficiency

The LLMs generate compact interests and tags.

A specialized retrieval model still performs large-scale nearest-neighbor candidate retrieval.

Explanations can be produced offline and cached.

In simple terms:

> “Use expensive reasoning where it adds meaning; use efficient retrieval where scale matters.”

---

# What the reported results support

The paper evaluates RecGPT in the “Guess What You Like” scenario on the Taobao homepage.

It reports the following improvements over the baseline in its online A/B test:

| Metric | Meaning | Reported improvement |
|---|---|---:|
| DT | user dwell time | +4.82% |
| EICD | diversity of exposed item categories | +0.11% |
| CICD | diversity of clicked item categories | +6.96% |
| IPV | item-page views | +9.47% |
| CTR | click-through rate | +6.33% |
| DCAU | daily users making a recommendation click | +3.72% |
| ATC | add-to-cart actions | +3.91% |

The paper also reports:

- behavior compression increased 128K-context coverage from 88% to 98%;
- behavior compression improved interest-inference efficiency by 29%;
- production optimizations using FP8 quantization and KV caching improved inference speed by 57%;
- incremental learning improved item-tag HR@30 by 1.05% in the reported setup.

## So the evidence supports

```text
In the evaluated Taobao deployment:
- intent-centered retrieval improved several engagement metrics;
- clicked-category diversity increased;
- the framework could be integrated into a large production pipeline;
- compression, caching, and lightweight serving made deployment feasible.
```

## The evidence does not prove

```text
- that the same percentage gains will transfer to every platform;
- that every generated interest is a true psychological explanation;
- that LLM-based intent reasoning always beats collaborative filtering;
- that increased clicks necessarily imply long-term user welfare;
- that the framework is free from popularity, merchant, or model biases.
```

The reported metrics are specific to the paper’s traffic allocation, product environment, baseline, model versions, and evaluation period.

---

# Important limitations

## Ultra-long histories remain difficult

The compression method handles most histories, but the paper reports that roughly 2% still exceed the 128K-token limit.

Compression can also remove details that later turn out to matter.

## Generated intent is an inference, not ground truth

A profile can be plausible without being the user’s actual motivation.

The model may:

```text
overinterpret weak behavior
confuse a gift purchase with personal preference
retain an outdated interest
invent an unsupported connection
```

## Several stages can propagate errors

```text
Incorrect interest profile
   ↓
Incorrect item tags
   ↓
Poor retrieval
   ↓
Misleading explanation
```

Interpretable intermediate outputs help diagnose this problem, but do not eliminate it.

## The system is operationally complex

It requires:

```text
multiple task-specific LLMs
data curation
judge models
human review
retrieval-model training
offline explanation production
caching and refresh pipelines
```

## The judge can drift

A judge trained on old behavior, products, or business rules may become unreliable.

The human-in-the-loop process reduces this risk but increases maintenance cost.

## Training is not yet unified

The paper trains several generation tasks separately and relies mainly on supervised learning with periodic updates.

It identifies reinforcement-learning-based multi-objective joint optimization as future work.

---

# Conceptual pseudocode

```text
function recommend_with_recgpt(user):

    # 1. Extract the behavior most likely to reflect real intent.
    reliable_history = filter_reliable_behaviors(
        user.full_behavior_history
    )

    # 2. Compress the history so it fits in the LLM context.
    compressed_history = hierarchical_compress(
        reliable_history
    )

    # 3. Generate an explicit user-interest profile.
    interests = user_interest_llm(
        user.attributes,
        compressed_history
    )

    # 4. Predict semantic descriptions of desired items.
    tags = item_tag_llm(
        user.attributes,
        interests,
        reliable_history
    )

    # 5. Produce behavioral and semantic retrieval vectors.
    user_vector = user_tower(user, reliable_history)
    tag_vector = tag_tower(tags)

    # 6. Balance established behavior with inferred intent.
    fused_vector =
        beta * user_vector
        + (1 - beta) * tag_vector

    # 7. Retrieve concrete catalog items.
    candidates = retrieve_nearest_items(
        fused_vector,
        item_tower_index
    )

    # 8. Reuse the platform's ranking and reranking system.
    ranked_items = rank_and_rerank(
        user,
        candidates
    )

    # 9. Attach a cached or generated explanation.
    for item in ranked_items:
        explanation = explanation_lookup_or_generate(
            interests,
            item
        )

    return ranked_items_with_explanations
```

Training-time quality loop:

```text
for each RecGPT generation task:

    create or collect candidate outputs

    human experts label a representative sample

    train a task-specific LLM judge

    use the judge to filter self-generated training data

    fine-tune the task model on approved data

    periodically compare judge decisions with humans

    if judge quality falls:
        collect fresh labels
        rebalance the data
        realign the judge
```

---

# Bottom line

**RecGPT-V1 works by converting user behavior into explicit intent, converting intent into predicted item descriptions, and combining those semantic signals with conventional behavioral retrieval to find and explain real products.**

The memorable version:

```text
Behavior says:
   “What did the user do?”

The interest LLM asks:
   “What might those actions mean?”

The tag LLM asks:
   “What kinds of items could satisfy that intent?”

The retrieval model asks:
   “Which real catalog products match both
   the intent and the behavioral evidence?”
```

---

## Primary source

Chao Yi et al., [*RecGPT Technical Report*](https://arxiv.org/abs/2507.22879), 2025. The report was later referred to as **RecGPT-V1** in the RecGPT-V2 and RecGPT-V3 technical reports.
