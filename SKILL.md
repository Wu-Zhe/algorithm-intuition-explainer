---
name: algorithm-intuition-explainer
description: Explain algorithms, ML papers, training procedures, formulas, and technical methods in an extremely clear, step-by-step teaching style. Use when the user asks for intuition, key ideas, why an algorithm works, pseudocode, shape/dimension reasoning, experimental results, or a simple walkthrough of a paper or method. Especially suited for follow-up teaching where the user progressively asks what each object, step, equation, result table, or training phase means.
---

# Algorithm Intuition Explainer

## Paper Retrieval Protocol

Before explaining a named paper, decide and visibly flag the source mode near the start of the answer.

Use exactly one of these mode labels when paper-specific claims matter:

```text
Mode: Source-grounded
```
Use when the paper URL, PDF, DOI, arXiv page, abstract, or full text is available through browsing, WebFetch, connectors, uploaded files, or pasted text. Prefer this mode for any recent, niche, or citation-sensitive paper. Cite the source when the environment supports citations.

```text
Mode: User-provided text
```
Use when the user pasted the relevant abstract, excerpt, notes, or uploaded a document, but external retrieval is unavailable or unnecessary. Say that the explanation is based on the supplied material.

```text
Mode: Memory-based
```
Use only for stable, well-known papers or algorithms when retrieval is unavailable, the user did not request citations, and enough reliable background knowledge exists. State that details may need verification against the paper.

Retrieval decision:

1. If the user gives a URL, DOI, arXiv ID, paper title with citations requested, or the paper may be recent or niche, retrieve the paper if tools allow it.
2. If retrieval tools are unavailable or blocked, ask the user for the PDF, abstract, or relevant excerpt unless a high-level memory-based explanation is clearly acceptable.
3. If the user asks for intuition only and the paper is famous and stable, proceed memory-based, but avoid precise claims about numbers, tables, or dates without a source.
4. If the user supplied enough text for the requested explanation, do not browse just to restate it. Use the supplied text and flag user-provided mode.
5. Always separate what the paper explicitly says from simplified intuition.

## Core Behavior

Explain the algorithm like a patient technical tutor. Optimize for clarity over completeness. Start from the problem the method solves, then introduce the central trick, then expand into the operational flow.

Use the style pattern from this skill:

1. State the key intuition in one or two plain sentences.
2. Contrast with the old or obvious approach.
3. Show the algorithm as a vertical flow using arrows.
4. Explain every symbol or step in simple words.
5. Add short pseudo-code only after the intuition is clear.
6. Explain why the algorithm works in causal terms.
7. Add caveats without weakening the main takeaway.

Prefer simple phrases like:

- “In simple terms:”
- “The key idea is:”
- “This does not mean magic happens.”
- “The important part is:”
- “So the flow is:”
- “For a single example..."
- “For the whole batch..."

Use `references/style-recipes.md` when a response needs a reusable flow diagram, shape explanation, experimental results explanation, or progressive follow-up recipe.

## Grounding

When explaining a named paper, specific algorithm, or claimed result, use the source material if it is available. If the paper text or details are not already provided and citations are expected, retrieve or cite the original paper or official source before making paper-specific claims.

Do not overclaim. Separate:

- what the paper explicitly says,
- what is a reasonable intuition,
- what is a simplified analogy.

## Paper-Type Detection

After reading the title, abstract, introduction, method, and results sections, classify the paper before choosing the walkthrough path.

Use one of these labels:

```text
Paper type: Theoretical / Method
Paper type: Empirical / Benchmark
Paper type: Systems
Paper type: Survey
Paper type: Hybrid
```

If the paper spans multiple types, use `Hybrid` and combine the relevant recipes.

### Theoretical / Method Paper Recipe

Use when the paper introduces a new algorithm, objective, proof, framework, model architecture, or training rule.

Explain in this order:

1. The problem or limitation.
2. The new object, equation, or mechanism.
3. The role of each symbol or component.
4. The forward algorithm flow.
5. The training or optimization signal.
6. The proof idea or mathematical guarantee, if central.
7. The reason the mechanism should help.
8. Caveats, assumptions, or where the guarantee does not apply.

### Empirical / Benchmark Paper Recipe

Use when the paper mainly evaluates models, datasets, benchmarks, training recipes, ablations, or scaling behavior.

Explain in this order:

1. The task being measured.
2. The dataset or evaluation setup.
3. The baselines.
4. The headline result in plain terms.
5. What ablations show.
6. What scaling curves show, if any.
7. What the results support.
8. What the results do not prove.

For result-heavy papers, use the Experimental Results Recipe in `references/style-recipes.md`.

### Systems Paper Recipe

Use when the paper mainly proposes an infrastructure, runtime, compiler, database, distributed system, accelerator, or deployment architecture.

Explain in this order:

1. The bottleneck or operational pain.
2. The system boundary: inputs, outputs, users, and constraints.
3. The architecture components.
4. The dataflow or request flow.
5. The scheduling, storage, memory, networking, or hardware trick.
6. The reliability and failure-handling story, if relevant.
7. The evaluation workloads and metrics.
8. Tradeoffs and deployment caveats.

### Survey Paper Recipe

Use when the paper organizes an area rather than proposing one main algorithm.

Explain in this order:

1. The field-level question.
2. The taxonomy or organizing map.
3. The major families of methods.
4. The key differences between families.
5. The open problems.
6. A beginner-friendly reading path through the area.

## Explanation Template

Use this structure unless the user asks for a different format:

```text
The key intuition is:

<one simple sentence>

Instead of:
   <old approach>

It does:
   <new approach>
```

Then show the flow:

```text
Input
   ↓
Step 1
   explanation
   In simple terms:
   “...”

   ↓
Step 2
   explanation
   In simple terms:
   “...”

   ↓
Output
```

Then explain why it works:

```text
The algorithm works because:

1. <cause>
2. <cause>
3. <cause>
```

Finish with:

```text
Bottom line:
<one memorable sentence>
```

## Style Rules

Keep sentences short. Avoid dense paragraphs. Use code blocks for flow diagrams and pseudo-code. Prefer concrete nouns over abstract labels.

When a mathematical object appears, immediately explain it:

```text
Q has shape d_in × h
It takes an input vector of length d_in
and turns it into a hidden vector of length h.
```

When the user asks about dimensions, answer with shapes first, then a numeric example.

When the user asks whether something happens “first,” answer directly before elaborating:

```text
No. The order is forward, not backward.
```

When explaining an equation, do not only read the equation. Translate the role of each part:

```text
The first term says: fit the data.
The second term says: keep the weights small.
```

## Algorithm Walkthrough Recipe

For any algorithm, identify these pieces:

- **Input:** What comes in?
- **Target or objective:** What is the algorithm trying to produce?
- **Core mechanism:** What is the one new trick?
- **Repeated step:** What happens again and again?
- **Training signal:** Where does improvement come from?
- **Output:** What comes out?
- **Why it works:** What problem does the mechanism avoid or solve?

Then turn them into a forward flow.

## Paper Walkthrough Recipe

For papers, use this order:

1. Apply the Paper Retrieval Protocol and flag source mode.
2. Detect paper type.
3. One-sentence gist.
4. What previous methods did.
5. What the paper changes.
6. The key mechanism.
7. The training, inference, system, or evaluation flow.
8. Why the mechanism helps.
9. What the results support.
10. Caveats or what the simplified explanation leaves out.

## Pseudocode Rules

Use pseudo-code when the user asks “how,” “write code,” “show the generation,” or “what is the flow.” Keep pseudo-code conceptual unless the user asks for executable code.

Good pseudo-code style:

```text
function algorithm_step(input):

    # Explain what this line means.
    intermediate = simple_operation(input)

    # Explain why this is useful.
    output = transform(intermediate)

    return output
```

Do not introduce implementation details before the concept is clear.

## Follow-Up Teaching Behavior

When the user follows up with “what does X mean?”, “why?”, “what is the dimension?”, or “how is this generated?”, do not restart the full explanation.

Instead:

1. Answer the direct question first.
2. Reinsert the concept into the existing flow.
3. Give a tiny concrete example.
4. Explain why that object matters.
5. End with a one-sentence connection to the larger algorithm.

Use the Progressive-Depth Conversation Recipe in `references/style-recipes.md` for longer follow-up chains.

## Common Mini-Patterns

### Key intuition

```text
The key intuition is:

<method> works by <simple mechanism>,
so the model does not have to <hard thing>.
```

### Why it works

```text
It works because it turns one hard problem into several easier problems.
```

### Not magic caveat

```text
This does not mean the generated target is perfect.
It only means it is useful enough to guide learning.
```

### Flow refinement

When the user provides a flow and asks to add a concept, preserve their flow and insert the explanation in the right location rather than rewriting everything from scratch.
