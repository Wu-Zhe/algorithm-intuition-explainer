# RecGPT-V2: Coordinated Agentic Intent Reasoning for Recommendation

> **Source mode:** Source-grounded  
> **Paper type:** Hybrid — method, production system, reinforcement-learning framework, and empirical evaluation  
> **Formatting note:** Mathematical expressions use portable Unicode notation, so this file renders correctly without a LaTeX or MathJax extension.

## The key intuition

**RecGPT-V2 turns recommendation into a coordinated reasoning process: compress the user’s long history once, let a planner divide the user’s possible needs among specialized experts, and let an arbiter combine their best ideas.**

Instead of:

```text
Route 1 reads the full user history
   ↓
Route 1 predicts items

Route 2 reads the same full user history
   ↓
Route 2 predicts items

Route 3 reads the same full user history
   ↓
Route 3 predicts items

Many routes repeat the same reading and reasoning
```

RecGPT-V2 does:

```text
Compress the user history once
   ↓
A global planner identifies complementary intent directions
   ↓
Specialized experts explore different directions
   ↓
A decision arbiter removes overlap and selects the best tags
   ↓
Real catalog items are retrieved and ranked
```

In simple terms:

> “One coordinator divides the work, several experts think differently, and one reviewer combines the results.”

RecGPT-V2 also upgrades three surrounding parts of the system:

```text
Fixed explanation templates
   → dynamic meta-prompts

Static supervised training
   → constrained reinforcement learning

One-shot quality scoring
   → multi-step Agent-as-a-Judge evaluation
```

---

# What RecGPT-V2 changes from RecGPT-V1

RecGPT-V1 introduced an intent-centered recommendation pipeline:

```text
User behavior
   ↓
Explicit user interests
   ↓
Predicted item tags
   ↓
Catalog retrieval
   ↓
Personalized explanations
```

RecGPT-V2 keeps this general idea but targets four practical weaknesses.

## Weakness 1: repeated computation

RecGPT-V1 was extended into multiple reasoning routes, such as routes for:

```text
general behavior
weather
seasonal events
trending topics
```

Each route could independently process the same long behavior history.

This created two kinds of waste:

```text
Representation waste:
   the same long context is encoded repeatedly

Reasoning waste:
   different routes may produce overlapping ideas
```

The paper reports an average input context near 32K tokens and a 13.46% duplication rate across reasoning routes.

## Weakness 2: repetitive explanations

Fixed prompts often generated explanations with similar wording and weak adaptation to:

```text
current weather
season
holidays
trending events
the specific audience
```

## Weakness 3: conflicting training goals

Recommendation generation needs several qualities at once:

```text
accuracy
relevance
alignment
diversity
appropriate length
clarity
timeliness
```

Simply adding these reward scores together can make an easy objective dominate a more important one.

For example:

```text
The model learns to produce very diverse tags
   ↓
But the tags become less accurate
```

## Weakness 4: shallow automated judging

RecGPT-V1’s judge predicted an overall quality result directly.

RecGPT-V2 argues that human reviewers normally evaluate several dimensions separately before forming a final judgment.

---

# The complete RecGPT-V2 flow

```text
Lifelong user behavior
+ user profile
+ real-time environment
   ↓
1. Hybrid representation inference
   Compress item and query descriptions into atomic vectors.
   Keep important natural-language context around them.

   In simple terms:
   “Shorten the history without throwing away its meaning.”

   ↓
2. Global Planner
   Analyze the compressed context once.
   Create several complementary intent personas.

   In simple terms:
   “Decide which different needs are worth exploring.”

   ↓
3. Distributed Experts
   Each expert receives one persona.
   Each generates item tags for that particular intent direction.

   In simple terms:
   “Let each expert investigate one part of the user’s needs.”

   ↓
4. Decision Arbiter
   Review all expert tags together.
   Remove duplication and choose a complementary final set.

   In simple terms:
   “Keep the best combined portfolio, not just the best individual tags.”

   ↓
5. Catalog retrieval and ranking
   Map semantic tags to real products.
   Combine them with multi-interest behavioral retrieval.
   Allocate exploration traffic under business constraints.

   ↓
6. Dynamic explanation generation
   First generate an appropriate communication style.
   Then write the recommendation explanation in that style.

   ↓
7. Agentic quality evaluation
   Specialized judges inspect different quality dimensions.
   A senior reviewer produces an overall judgment.

   ↓
8. Judge-as-a-Reward
   Distill expensive multi-agent judgments into a lightweight reward model.
   Use the reward to improve the generation policy.

   ↓
Recommended items
+ context-aware explanations
```

---

# Part 1: Hybrid Representation Inference

## The problem

A long user history may contain thousands of product titles and search queries.

If each item is written in full:

```text
item title with many tokens
item attributes with many tokens
search phrase with many tokens
...
```

the LLM must repeatedly process a huge sequence.

Attention computation also grows rapidly with input length.

## The core compression trick

RecGPT-V2 converts each item or search entity into a dense embedding:

**h = f_embed(x)**

where:

- **x** is the entity’s original text;
- **f_embed** is a pretrained embedding model;
- **h** is a fixed-length semantic vector.

A small adaptor then maps the embedding into the LLM’s hidden space:

**z = W₂ σ(W₁h + b₁) + b₂**

where:

- **W₁** and **W₂** are learned projection matrices;
- **b₁** and **b₂** are biases;
- **σ** is a nonlinear activation;
- **z** is the atomic representation inserted into the LLM context.

The full product title can then be replaced by a single conceptual token:

```text
Long product description
   ↓ embedding model
Dense semantic vector
   ↓ adaptor
[entity]
```

In simple terms:

> “Replace a paragraph about a product with one learned packet of meaning.”

## Why it is called a hybrid representation

The entire prompt is not converted into vectors.

RecGPT-V2 mixes two forms:

```text
Natural language:
   user attributes
   timestamps
   weather
   seasonal context
   instructions

Atomic representations:
   product descriptions
   historical search entities
```

This preserves human-readable structure while compressing the most repetitive parts.

## How the adaptor learns

The main LLM remains frozen during the adaptor-training phase.

Only the small adaptor is trained.

Two kinds of tasks teach it.

### Self-perception tasks

The system asks questions about a compressed entity:

```text
What type of product is this?
What attributes does it have?
Who might use it?
What season is it suitable for?
```

The adaptor must preserve enough information for the frozen LLM to answer.

### Production-oriented tasks

The compressed representation is used in actual recommendation tasks such as:

```text
user-interest mining
item-tag prediction
```

The compressed prompt is trained to produce responses similar to those generated from the full-text prompt.

In simple terms:

> “The atomic vector is good only if the LLM can still reason as though it had read the original text.”

## Why freeze the LLM?

```text
Frozen LLM
   ↓
preserves general language and reasoning ability

Small trainable adaptor
   ↓
learns how to translate recommendation entities
into the LLM’s existing semantic space
```

This makes the representation layer easier to replace or reuse with different embedding models and LLM backbones.

---

# Part 2: Hierarchical Multi-Agent System

The central reasoning architecture is:

```text
Global Planner
   ↓
Distributed Experts
   ↓
Decision Arbiter
```

This is the main algorithmic change in RecGPT-V2.

---

## Step 1: Build the shared context

The planner receives three kinds of information:

**C = {B, U, E}**

where:

- **B** is the compressed behavioral history;
- **U** is the user profile;
- **E** is the environmental context.

The user profile includes:

```text
Static attributes:
   relatively stable user information

Dynamic interests:
   patterns inferred from recent and long-term behavior
```

The environmental context can include:

```text
weather
season
holidays
trending events
```

In simple terms:

> “Understand both who the user usually is and what may matter right now.”

---

## Step 2: The Global Planner creates personas

The planner produces *K* complementary personas:

**{p₁, p₂, …, pₖ} = f_planner(C)**

A persona is not necessarily a demographic identity.

It is a specialized reasoning role focused on one facet of the user’s potential intent.

For example:

```text
User context:
- cooling weather
- upcoming Halloween
- historical fitness interest
- purchases for children

Planner personas:
- ladies’ fashion expert
- children’s products expert
- health and fitness expert
```

The important part is that the planner processes the shared context once.

It explicitly asks experts to explore different semantic regions.

In simple terms:

> “Do not let every expert independently rediscover the same user story.”

---

## Step 3: Experts generate item tags

Each persona is sent to one expert:

**Tₖ = f_expert(pₖ)**

where:

- **pₖ** is one planner-generated persona;
- **Tₖ** is that expert’s set of predicted item tags.

Example:

```text
Fashion expert:
- wool-blend cardigan
- autumn ankle boots

Children’s-products expert:
- children’s hydrating lotion
- Halloween costume

Health expert:
- adjustable dumbbell set
- indoor exercise mat
```

The experts run in parallel, but their assignments are coordinated.

In simple terms:

> “Parallel thinking is useful when the thinkers are given different jobs.”

---

## Step 4: The Decision Arbiter combines the experts

The candidate pool is the union of all expert tags:

**T_all = T₁ ∪ T₂ ∪ … ∪ Tₖ**

The arbiter then produces:

**T_final = f_arbiter(T_all, C)**

It evaluates the tags jointly rather than treating each one independently.

The arbiter considers qualities such as:

```text
behavioral relevance
consistency with the user profile
specificity
validity
complementarity
lack of duplication
```

Why joint evaluation matters:

```text
Three individually good tags
may all describe nearly the same product need.

A jointly good tag set
covers several useful needs without becoming random.
```

In simple terms:

> “Select the best team of recommendations, not merely the highest-scoring individuals.”

---

# Part 3: Training the experts

The experts are trained in two stages.

```text
Supervised fine-tuning
   ↓
Constrained reinforcement learning
```

---

## Stage 1: Supervised fine-tuning

For each planner persona, the system identifies future user interactions that semantically match that persona.

Those interactions become training targets.

The expert learns:

```text
Given this persona,
predict item tags consistent with it.
```

General instruction-following data is mixed with recommendation-specific data so the model retains broad language and reasoning ability.

In simple terms:

> “First teach the expert what a reasonable answer looks like.”

---

## Stage 2: Reinforcement learning

The paper uses Group Relative Policy Optimization, or GRPO.

For one input, the policy generates a group of candidate outputs.

The candidates are compared using rewards, and outputs better than the group average receive a positive learning signal.

A KL penalty discourages the updated policy from moving too far from the supervised model.

In simple terms:

> “Generate several answers, reward the relatively better ones, but do not let the model change recklessly.”

---

# The key training trick: Constrained Reward Shaping

The item-tag task uses four reward dimensions.

## Accuracy reward

**R_acc**

This asks:

> “Do the generated tags cover categories the user actually interacts with?”

## Alignment reward

**R_align**

This asks:

> “Are the tags relevant to the assigned persona and consistent with human quality standards?”

## Diversity reward

**R_div**

This asks:

> “Are the tags semantically different enough to cover multiple needs?”

## Length reward

**R_len**

This asks:

> “Are the tags detailed enough to be meaningful but short enough to work well in retrieval?”

---

## Why not simply add the rewards?

A simple weighted sum would look conceptually like:

**R_sum = aR_acc + bR_align + cR_div + dR_len**

The problem is that gradients from different rewards can conflict.

For example:

```text
Increase diversity
   ↓
produce more unusual tags
   ↓
accuracy may fall
```

or:

```text
Increase accuracy
   ↓
repeat safe popular categories
   ↓
diversity may fall
```

A large combined score can hide a serious failure in one critical dimension.

---

## RecGPT-V2’s constrained reward

RecGPT-V2 treats accuracy as the main objective and the other rewards as minimum conditions:

**R_total = R_acc × 𝕀[R_align ≥ τ_align] × 𝕀[R_div ≥ τ_div] × 𝕀[R_len ≥ τ_len]**

Here:

- **𝕀[condition]** is 1 when the condition is satisfied and 0 otherwise;
- **τ_align**, **τ_div**, and **τ_len** are minimum thresholds.

The equation says:

```text
Check alignment
Check diversity
Check length
   ↓
If every condition passes:
   use the accuracy reward

If any condition fails:
   total reward becomes zero
```

In simple terms:

> “The model may optimize accuracy only inside the region where the other requirements are acceptable.”

This changes the learning problem from:

```text
Trade every quality against every other quality
```

to:

```text
First enter the acceptable region
   ↓
Then improve the main objective
```

The paper reports that this constrained version trained more stably than direct reward summation and reached a higher tag-prediction HR@30 in its experiment.

---

# Part 4: Mapping the final tags to real items

The agents generate semantic tags, not catalog item IDs.

RecGPT-V2 retains the user–item–tag retrieval foundation from RecGPT-V1 and extends it with multi-interest user encoding.

## Multi-interest user representation

Instead of compressing the user into one vector, the user encoder learns several interest vectors:

```text
User behavior
   ↓
Interest vector 1
Interest vector 2
...
Interest vector K
```

Each vector can specialize in a different preference facet.

Candidate items are compared with:

```text
tag representations
+
multiple user-interest representations
```

In simple terms:

> “A person does not have only one taste, so retrieval should not force every preference into one vector.”

## Exploration and exploitation

RecGPT-V2 distinguishes:

```text
Utility channel:
   reliable items supported by established behavioral models

Cognitive channel:
   exploratory items suggested by LLM-derived intent
```

Traffic allocation is formulated as a quadratic-programming problem.

Its practical role is:

> Allocate enough exposure to exploratory recommendations to improve discovery, while protecting short-term business performance.

---

# Part 5: Dynamic explanation generation

RecGPT-V1 generated explanations directly from a fixed prompt.

RecGPT-V2 separates the task into two stages.

```text
User interest
+ item attributes
+ current context
   ↓
Stage 1: generate a style guideline
   ↓
Stage 2: generate the explanation using that style
```

---

## Stage 1: Style synthesis

**g = f_meta(U, I, S)**

where:

- **U** is the user interest;
- **I** is the item information;
- **S** is the situational context;
- **g** is a generated style instruction.

The style can specify:

```text
tone
target audience
emotional effect
rhetorical device
degree of playfulness
seasonal framing
```

For a children’s toy near a holiday, the generated guideline might request a playful, visual, parent-oriented caption.

## Stage 2: Explanation generation

**e = f_exp(g, U, I, S)**

where **e** is the final explanation.

In simple terms:

> “First decide how this particular recommendation should be communicated; then write it.”

This is called **meta-prompting** because one model-generated instruction guides another generation step.

---

# Improving explanations with constrained RL

The explanation policy uses two main rewards.

## Alignment reward

**R_align**

This measures qualities such as:

```text
relevance
factuality
clarity
safety
timeliness
informativeness
attractiveness
```

## Diversity reward

**R_div**

The system keeps a buffer of recent explanations and rewards language that is less repetitive.

For explanations, RecGPT-V2 uses:

**R_total = R_align × 𝕀[R_div ≥ τ_div]**

This says:

> “Optimize human-aligned quality, but only when the explanation is sufficiently diverse.”

The paper reports that RecGPT-V2 improved both explanation diversity and human-rated quality compared with RecGPT-V1 in the evaluated setup.

---

# Part 6: Agent-as-a-Judge

Recommendation outputs cannot be evaluated reliably with exact string matching.

A tag or explanation may be good even when it differs from the reference wording.

RecGPT-V2 evaluates generation in stages.

```text
Generated item tags or explanations
   ↓
Specialized sub-evaluators
   each checks one quality dimension
   ↓
Senior Reviewer
   combines all dimension-level judgments
   ↓
Superior / Average / Bad
```

## Specialized sub-evaluators

For item tags, dimensions can include:

```text
relevance
consistency
specificity
validity
```

For explanations, dimensions can include:

```text
relevance
factuality
clarity
safety
timeliness
informativeness
attractiveness
```

Each evaluator focuses on one question.

In simple terms:

> “Do not ask one judge to think about everything at once.”

## Senior Reviewer

The Senior Reviewer combines the dimension-specific results.

The paper uses three quality levels:

```text
Superior:
   strong across most or all dimensions

Average:
   meets the basic standard

Bad:
   fails at least one critical requirement
```

The decision logic is roughly:

```text
Any critical defect?
   ↓ yes
Bad

No critical defect?
   ↓
Check how many dimensions are strongly positive
   ↓
Average or Superior
```

This mirrors a structured human review process more closely than a single direct score.

---

# Part 7: Judge-as-a-Reward

The multi-agent judge is informative but expensive.

It also produces discrete labels, while reinforcement learning benefits from dense scalar rewards.

RecGPT-V2 therefore distills the judge into a smaller reward model.

```text
Agent-as-a-Judge
   ↓
Produces Superior / Average / Bad preferences
   ↓
Train a lightweight reward model
   ↓
Reward model outputs a continuous score
   ↓
Use that score during reinforcement learning
```

The reward model learns the ordering:

**Superior ≻ Average ≻ Bad**

using listwise learning-to-rank.

Why listwise training helps:

```text
Pointwise training:
   judge each output independently

Listwise training:
   learn the relative ordering among several quality levels
```

In simple terms:

> “Teach the reward model not only what is good, but what is better than what.”

---

# The self-improvement flywheel

The pieces form a loop:

```text
Policy generates item tags and explanations
   ↓
Agent-as-a-Judge evaluates several quality dimensions
   ↓
Judge-as-a-Reward converts judgments into dense scores
   ↓
Reinforcement learning updates the policy
   ↓
The improved policy generates better outputs
```

The authors describe this as a flywheel.

Human annotation is still required to establish and refresh the quality standard, even though the loop can automate much of the later iteration.

---

# A tiny end-to-end example

Suppose the system sees:

```text
User:
- 35 years old
- lives in a region where the weather is cooling
- has historical interest in fitness
- has purchased children’s products

Context:
- Halloween is approaching
```

## 1. Compress the history

```text
Full product and search descriptions
   ↓
Atomic entity representations
   ↓
Shorter hybrid context
```

## 2. Planner decomposes the intent

```text
Persona 1:
   autumn fashion expert

Persona 2:
   children’s seasonal-products expert

Persona 3:
   home-fitness expert
```

## 3. Experts predict tags

```text
Fashion expert:
   wool-blend cardigan

Children’s expert:
   children’s moisturizing lotion
   Halloween costume

Fitness expert:
   adjustable dumbbell set
```

## 4. Arbiter selects a complementary set

The arbiter may retain:

```text
wool-blend cardigan
Halloween costume
adjustable dumbbell set
```

because these cover different plausible needs.

## 5. Retrieval finds real catalog items

```text
Semantic tags
   ↓ tag and item towers
Concrete product candidates
   ↓ ranking system
Final products
```

## 6. Meta-prompting generates explanations

For the cardigan:

```text
Style guideline:
   warm, seasonal, concise

Explanation:
   “A light wool blend can add warmth as autumn temperatures begin to drop.”
```

This is a simplified illustration of the reported mechanism, not a literal trace of one production request.

---

# Why RecGPT-V2 works intuitively

## 1. It removes duplicated reading

The long behavior history is compressed and interpreted once by the planner.

Experts do not each need to re-read the same 32K-token context independently.

## 2. It creates deliberate cognitive diversity

Experts are assigned complementary personas.

Diversity is planned rather than left to chance.

```text
Uncoordinated parallelism:
   several agents may repeat the same idea

Coordinated parallelism:
   each agent explores a different useful direction
```

## 3. It separates generation from selection

Experts are allowed to explore.

The arbiter is responsible for consistency, relevance, and redundancy removal.

This division makes the overall task easier:

```text
Experts:
   generate possibilities

Arbiter:
   construct a coherent final set
```

## 4. Constraints protect critical qualities

The reinforcement-learning objective does not allow an easy secondary metric to compensate for a failed critical requirement.

The system must meet the minimum quality gates before receiving the main reward.

## 5. Evaluation follows the structure of the task

A multi-dimensional output is judged by multiple dimension-specific evaluators.

This makes the feedback more interpretable and easier to distill into training signals.

## 6. Expensive reasoning is used selectively

```text
LLMs:
   planning, intent reasoning, tag generation, explanation

Specialized retrieval infrastructure:
   search a massive catalog efficiently

Lightweight reward model:
   provide frequent training feedback
```

In simple terms:

> “Use the expensive model for meaning and coordination; use specialized systems for scale.”

---

# What the experiments support

## Representation and systems efficiency

The paper reports that the combined optimization pipeline:

- compressed user behavior contexts substantially;
- improved Model FLOPs Utilization from 11.56% to 17.04%;
- reduced GPU consumption by 60%;
- increased prefill QPS and decoding throughput in the reported serving setup.

These measurements support the claim that the compressed representation and serving architecture improved the efficiency of this particular deployment.

They do not establish the same savings for every LLM, accelerator, context length, or recommender system.

## Intent coverage and tag prediction

The paper reports:

| Method | HR@30 |
|---|---:|
| RecGPT-V1 | 26.29% |
| RecGPT-V2 base model | 23.08% |
| RecGPT-V2 SFT | 29.20% |
| RecGPT-V2 GRPO with summed rewards | 27.38% |
| RecGPT-V2 GRPO with constrained rewards | 32.60% |

Higher HR@30 means that the predicted tags more often matched categories in the user’s later interactions.

The table suggests:

```text
Domain supervision helps.
Naively summing rewards can hurt.
Constrained reward shaping performed best in this experiment.
```

The paper also reports that exclusive recall across reasoning routes increased from 9.39% to 10.99%, supporting the claim that coordinated experts produced more non-overlapping coverage.

## Explanation generation

The paper reports:

| Method | Diversity | Human-rated quality |
|---|---:|---:|
| RecGPT-V1 | 0.631 | 36.03% |
| RecGPT-V2 | 0.677 | 40.73% |

The authors interpret this as a 7.3% relative diversity improvement and a 13.0% relative quality improvement.

These results support the combination of meta-prompting and preference-aware RL in the studied evaluation.

They do not isolate exactly how much improvement came from each individual component unless supported by additional ablations.

## Agent-as-a-Judge

The paper compares the V1 and V2 judges against human labels.

The V2 process generally improves agreement, particularly for the fine-tuned Qwen judge used for explanation assessment.

The gains are smaller than the recommendation-quality gains, so the result is best interpreted as a measured alignment improvement rather than a solved evaluation problem.

## Online A/B test

The report describes a two-week Taobao homepage experiment with RecGPT-V1 as the control.

Each control and experimental group received 1% of total platform traffic.

Reported relative changes were:

| Scenario | IPV | CTR | TV | GMV | ATC | NER | LT-14 | LT-30 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Item | +3.64% | +3.01% | +2.11% | +3.39% | +3.47% | +11.46% | — | — |
| Feed | +1.29% | +1.50% | +0.34% | +1.53% | +0.99% | +4.49% | +0.04% | +0.05% |

The large NER increase supports the claim that the system exposed users to more novel items in this deployment.

The engagement and transaction gains support practical usefulness relative to the RecGPT-V1 baseline during the experiment.

They do not prove:

```text
- that every user benefited equally;
- that novelty always improved satisfaction;
- that the method transfers unchanged to another marketplace;
- that the LLM’s inferred personas are the user’s true internal motives;
- that the observed gains persist indefinitely;
- that the system is free from popularity, merchant, demographic, or model biases.
```

---

# Important limitations

## The intent decomposition can still be wrong

The planner may create personas that are plausible but unsupported.

A mistaken plan can propagate through all experts.

```text
Incorrect persona
   ↓
Incorrect tags
   ↓
Irrelevant item retrieval
```

## Compression can discard details

Atomic representations reduce context length, but no finite vector is guaranteed to preserve every detail in the original text.

Rare attributes may be lost or blurred.

## The system is complex

RecGPT-V2 requires:

```text
embedding models
adaptors
a planner
expert agents
an arbiter
retrieval towers
ranking infrastructure
meta-prompt generation
explanation generation
sub-evaluators
a senior judge
reward-model distillation
reinforcement-learning pipelines
```

Each additional module introduces maintenance and failure modes.

## Hard reward thresholds can be brittle

A constraint just below its threshold can zero out the entire reward, while a nearly identical score just above the threshold passes.

Thresholds may need careful tuning and monitoring.

## Automated judges remain imperfect

Decomposed evaluation can improve agreement without guaranteeing truth.

The judge may inherit:

```text
annotation bias
model bias
outdated quality standards
blind spots shared by its training data
```

Human review remains important.

## Online metrics are not complete welfare measures

CTR, item-page views, transaction value, and novelty are useful operational metrics.

They do not fully measure:

```text
long-term satisfaction
decision quality
unwanted persuasion
fairness among sellers
privacy
user autonomy
```

---

# Conceptual pseudocode

```text
function recommend_with_recgpt_v2(user, environment):

    # 1. Compress long item and search descriptions.
    hybrid_context = atomize_entities(
        user.behavior_history,
        user.profile,
        environment
    )

    # 2. Read the shared context once and divide the reasoning work.
    personas = global_planner(hybrid_context)

    expert_tag_sets = []

    # 3. Each expert explores one complementary intent direction.
    for persona in personas:
        tags = expert_agent(persona)
        expert_tag_sets.append(tags)

    # 4. Select a coherent, non-redundant final tag set.
    final_tags = decision_arbiter(
        expert_tag_sets,
        hybrid_context
    )

    # 5. Map semantic tags and multiple user interests to catalog items.
    cognitive_candidates = retrieve_from_tags(final_tags)
    utility_candidates = retrieve_from_behavior(user)

    # 6. Balance exploration with established utility.
    candidates = allocate_and_merge(
        cognitive_candidates,
        utility_candidates
    )

    ranked_items = production_ranker(user, candidates)

    # 7. Generate context-specific communication styles and explanations.
    results = []

    for item in ranked_items:
        style = meta_prompt_generator(
            user.interests,
            item.attributes,
            environment
        )

        explanation = explanation_model(
            style,
            user.interests,
            item.attributes,
            environment
        )

        results.append((item, explanation))

    return results
```

Training the expert policy:

```text
for each planner persona:

    # Supervised stage.
    learn to predict persona-aligned item tags

    # Reinforcement-learning stage.
    generate a group of candidate tag sets

    for each candidate:
        compute accuracy
        compute human-alignment score
        compute diversity
        compute length quality

        if alignment, diversity, and length pass:
            reward = accuracy
        else:
            reward = 0

    update the policy toward better candidates
    while limiting deviation from the supervised model
```

Judge and reward flywheel:

```text
policy generates outputs
   ↓
dimension-specific judge agents evaluate them
   ↓
senior reviewer assigns Superior / Average / Bad
   ↓
distill rankings into a scalar reward model
   ↓
use the reward model to train the policy
   ↓
repeat with refreshed human supervision
```

---

# Bottom line

**RecGPT-V2 works by compressing the user’s history once, coordinating several specialized intent experts through a planner and arbiter, and using structured evaluation plus constrained reinforcement learning to improve both recommendations and explanations.**

The memorable version:

```text
The planner asks:
   “Which different needs should we explore?”

The experts ask:
   “What products fit my assigned need?”

The arbiter asks:
   “Which combined set is relevant, diverse, and non-redundant?”

The judge asks:
   “Does every important quality dimension pass?”

The reward model asks:
   “How can that judgment become a practical learning signal?”
```

---

## Primary source

Chao Yi et al., [*RecGPT-V2 Technical Report*](https://arxiv.org/abs/2512.14503), arXiv:2512.14503, submitted December 16, 2025.

Key sections used:

- Section 2.1: Hybrid Representation Inference
- Section 2.2: Hierarchical Multi-Agent System
- Section 3: Dynamic Explanation Generation
- Section 4: Agentic Judge Framework
- Section 5: Online experiments and case study
