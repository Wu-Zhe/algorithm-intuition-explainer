# Style Recipes

Canonical presentation patterns for Generate Simple Explanation.

## Contents

1. Non-Redundant Answer Architecture
2. Intuition and Contrast
3. Forward Flow
4. Equation Explanation
5. Browser-Safe Markdown Math
6. Shape Explanation
7. Tiny Example
8. Pseudocode
9. Why It Works and So What
10. Experimental Results
11. Progressive-Depth Follow-Ups
12. Optional Final Compression
13. Downloadable Markdown Reports
14. Redundancy Audit

## 1. Non-Redundant Answer Architecture

Build the answer from distinct information slots. Select only the slots needed by the user.

| Slot | Unique question it answers |
|---|---|
| Problem | What is difficult or missing? |
| Intuition | What single mechanism changes the situation? |
| Contrast | What does the method replace? |
| Flow | In what order do operations occur? |
| Equation | What exact relationship is computed? |
| Shapes | Which dimensions must match? |
| Example | What happens for concrete values? |
| Pseudocode | How would someone execute the procedure? |
| Why | What causal link makes the mechanism useful? |
| Evidence | What was actually observed? |
| Implication / So what | What capability, decision, cost, risk, user outcome, or research direction changes? |
| Caveat | Where can the explanation or method fail? |

### One-slot rule

Assign each fact to one slot. Do not repeat it in another slot with different wording.

Examples:

- If the intuition already says the method freezes a base matrix and trains a low-rank update, the flow should show the operations, not restate that sentence.
- If shapes already establish a parameter-count reduction, do not create a second section called `Why two matrices?` that repeats the bottleneck.
- If pseudocode comments explain each step, do not add a prose line-by-line walkthrough.
- If the opening states the main takeaway, omit a closing bottom line.

### Representation gate

Include an additional representation only when it answers a different question:

```text
Flow       → What happens first, next, and last?
Equation   → What exact mathematical relation is used?
Shapes     → Why are the operations dimensionally valid?
Example    → What values appear after each operation?
Pseudocode → How is the procedure executed?
```

If two representations answer the same question, keep the clearer one.

### Length control

For a short request, prefer:

```text
one mechanism sentence
+ one supporting representation
+ one implication or caveat, if needed
```

For a broad walkthrough, add slots only when they contribute new information.

## 2. Intuition and Contrast

Combine intuition and contrast into one compact unit.

```text
<Method> replaces <old or difficult approach>
with <literal mechanism>.
```

Optionally add one problem sentence before it when the need is not obvious:

```text
The problem is <specific limitation>.
<Method> replaces <old approach> with <literal mechanism>.
```

Do not create separate `Key intuition`, `Ordinary approach`, and `Why this is different` sections when they express the same contrast.

Use a literal explanation first. Add an analogy only when it maps a genuinely unfamiliar mechanism to a familiar structure. Do not add an analogy that merely paraphrases the literal sentence.

## 3. Forward Flow

Use a flow only when operation order matters.

```text
Input
   ↓
Operation 1 → intermediate object
   ↓
Operation 2 → intermediate object
   ↓
Output
```

Keep each step operational. Do not add a repeated plain-language translation below every line when the terms were already defined.

When a step introduces a new object, define it once inside the flow:

```text
Scores = QKᵀ
   “one score for each query-key pair”
```

Do not define the same object before and after the flow.

For a user-provided flow, preserve its order and insert the requested explanation exactly where the concept occurs.

## 4. Equation Explanation

Start with the equation's purpose, then explain only the terms whose roles are not obvious.

```text
Purpose:
   <what the equation computes or balances>

Term 1:
   <mathematical term>
   <unique role>

Term 2:
   <mathematical term>
   <unique role>
```

For an optimization objective:

```text
Data-fit term:
   rewards matching the observations

Regularization term:
   penalizes an unstable or overly complex solution
```

Do not read the equation aloud symbol by symbol. Do not add a `Together` sentence when it only repeats the stated purpose.

If equations use column vectors but pseudocode uses row-major batches, state the convention once.

## 5. Browser-Safe Markdown Math

Use this recipe whenever the output is written to a `.md` file intended for a generic browser, repository viewer, note application, or Markdown renderer.

### Mandatory default

Assume that LaTeX, MathJax, and KaTeX are unavailable unless the user explicitly requests LaTeX and confirms that the target renderer supports it.

Do not write:

```text
\[ equation \]
\( inline equation \)
$$ equation $$
```

Do not use a fenced code block merely to display an equation:

```text
Z_tilde_l = g_l(A_(l-1) Q_l) + g_l(Y U_l)
```

### Required portable form

Write a standalone equation as an ordinary Markdown paragraph, preferably bold:

**Z̃ₗ = gₗ(Aₗ₋₁ Qₗ) + gₗ(Y Uₗ)**

Prefer these portable forms:

```text
LaTeX-like source     Browser-safe Markdown
Z_l                   Zₗ
Z_tilde_l             Z̃ₗ
A_{l-1}               Aₗ₋₁
A^T                   Aᵀ
(A^T A + λI)^-1       (AᵀA + λI)⁻¹
approximately equal   ≈
multiplication        ×
square root           √
```

Use italics or plain text for variables in prose:

```markdown
For layer *l*, construct the target **Z̃ₗ**.
```

Do not use inline-code formatting for mathematical variables unless referring to literal source code.

### Long equations

When Unicode would make a long equation difficult to scan:

1. Show one compact browser-safe equation.
2. Split it into named terms underneath.
3. Explain each term once.

Example:

**Z̃ₗ = input code + label code**

- **Input code:** **gₗ(Aₗ₋₁ Qₗ)** preserves information about the current representation.
- **Label code:** **gₗ(Y Uₗ)** injects information about the correct class.

### Allowed fenced blocks

Use fenced `text` blocks for:

- vertical algorithm flows,
- tensor shape tables,
- pseudocode,
- terminal output,
- actual code.

Do not use them for equation-only display.

### Pre-save validation

Before saving a Markdown file, verify:

- no `\[`, `\]`, `\(`, `\)`, or `$$` delimiters remain unless explicitly requested;
- no standalone `=` line exists outside a fenced block;
- no equation-only fenced block is used where normal Markdown would read better;
- mathematical variables are not accidentally shown as inline-code pills;
- the document remains understandable in a renderer with no math extension.

## 6. Shape Explanation

Give shapes before prose.

```text
X: N × d
W: d × h
XW: N × h
```

Then explain the one dimensional fact that matters:

```text
The inner dimension d matches, so each length-d input becomes a length-h output.
```

Add a numeric example only when symbolic shapes are not enough:

```text
X: 100 × 64
W: 64 × 256
XW: 100 × 256
```

Keep parameter arithmetic in this section when it follows directly from the shapes. Do not repeat it in a separate architecture or efficiency section.

## 7. Tiny Example

Use the smallest example that preserves the mechanism. Start with values, not another mechanism summary.

```text
Input: <small vector or 2–3 items>

Step 1:
   <operation and result>

Step 2:
   <operation and result>
```

The example must produce at least one concrete intermediate or output that was not already shown elsewhere.

Do not use a toy example that changes the actual algorithm. Do not conclude the example by restating the general mechanism unless the example revealed a non-obvious implication.

## 8. Pseudocode

```text
function algorithm_step(input):

    intermediate = simple_operation(input)  # explain only a non-obvious purpose
    output = transform(intermediate)

    return output
```

For training:

```text
for each batch:

    prediction = forward(batch)
    loss = objective(prediction, target)
    update(trainable_parameters, loss)
```

Add shapes next to variables when dimension reasoning matters.

Do not repeat pseudocode as prose. Explain only:

- a non-obvious line,
- an invariant,
- a shape change,
- a training-only behavior,
- or a simplification relative to the real implementation.

## 9. Why It Works and So What

### Why it works

Explain causes, not the operations already shown.

```text
<property created by the mechanism>
   ↓ causes
<useful effect on the next stage>
   ↓ therefore
<task-level benefit>
```

Example structure:

```text
The constraint reduces the number of free directions.
That makes the estimate less sensitive to noise.
The fitted model is therefore more stable on limited data.
```

Do not write `The algorithm works because it performs Step 1 and Step 2.` That repeats the flow rather than explaining why the steps help.

Add a non-magic caveat when appropriate:

```text
This is an empirical or conditional advantage, not a guarantee for every task.
```

### So what

Translate the task-level benefit into an external consequence when the user asks why the topic matters or the significance is not obvious:

```text
<task-level benefit>
   ↓ changes
<capability, decision, cost, risk, user outcome, or research direction>
```

Example:

```text
Greater stability with limited labels can make a model usable when collecting more labeled data is expensive.
```

Keep the implication within the available evidence. Distinguish an observed consequence from a projected one, and do not use vague claims such as `This is important` or restate the task-level benefit under a new heading.

## 10. Experimental Results

Use this recipe for benchmark tables, ablations, scaling curves, and result-heavy papers.

### Headline and evidence

State the headline once:

```text
Compared with <baseline>, <method> is better on <where>,
roughly tied on <where>,
and worse on <where>.
```

Then add only evidence that supports a different aspect of the claim.

Mention metric direction explicitly:

```text
Higher is better: accuracy, F1, BLEU, throughput
Lower is better: loss, latency, memory, error rate
```

Do not assume the direction when the metric is specialized; verify it from the source.

### Benchmark tables

Explain table structure only when the user needs help reading it:

```text
Rows: methods
Columns: datasets, tasks, or metrics
```

Do not separately restate best, tied, and weak values after the comparison sentence has already covered them.

### Ablations

```text
Full method
   ↓ remove or replace one component
Ablated method
   ↓ compare the result
```

State the distinct inference:

- A drop supports that the component helped in this setup.
- Little change suggests the component may not be essential in this setup.

Do not repeat the full-method description. Do not conclude universal necessity or uselessness from one ablation.

### Scaling curves

Explain only the observed curve behavior:

```text
Steep improvement → scaling helps strongly in the measured range
Flat curve        → the setup has saturated
Noisy curve       → the result may be unstable or under-measured
Crossing curves   → the preferred method depends on scale
```

### So what

When significance is requested or not obvious, add one bounded consequence:

```text
Because <observed result>, <stakeholder> can consider <decision or capability> within <tested scope>.
```

Do not present a projected deployment, cost, safety, or user impact as directly measured unless the source evaluated it.

### Claim boundary

Finish with one boundary statement:

```text
The results support <directly observed claim>,
but do not establish <broader claim>.
```

Do not add a second result summary after the boundary.

## 11. Progressive-Depth Follow-Ups

Answer the exact question first. Select only the levels needed; do not automatically include all five.

### Level 1: Direct meaning

```text
<X> means <one plain sentence>.
```

### Level 2: Position in the flow

```text
Previous step
   ↓
<X>
   ↓
Next step
```

### Level 3: Tiny example

```text
For a tiny example:
   <small concrete example>
```

### Level 4: Why it matters / So what

```text
<X> creates <property>, so the next step can <useful effect>.
This matters because it changes <capability, decision, cost, risk, user outcome, or research direction>.
```

Use the second sentence only when it adds an external consequence rather than restating the useful effect.

### Level 5: Technical detail

Add equations, shapes, implementation details, or caveats only when the user asks or they are necessary to resolve the question.

Do not restart the original paper or algorithm explanation. Use a short back-reference when context is needed.

## 12. Optional Final Compression

Do not add a `Bottom line` by default.

Use one final compression sentence only when:

- the answer is long and no equivalent takeaway appeared at the start, or
- the user explicitly asks for a summary.

When a final compression is useful, replace the opening takeaway with it rather than using both.

```text
<Method> uses <central mechanism> to address <original limitation>.
```

If the implication is the requested takeaway, use this form instead:

```text
<Finding or method> matters because <claim-bounded external consequence>.
```

## 13. Downloadable Markdown Reports

Create a downloadable Markdown artifact for a broad paper walkthrough, comprehensive technical report, or explicit Markdown-file request. Keep narrow answers inline unless the user requests a file.

### Artifact structure

Use only the sections selected for the report, in this order when applicable:

```text
# <Descriptive report title>

Mode: <source mode>
Paper type: <type>

<problem and central mechanism>
<flow, components, or equations>
<evidence and results>
<why it works>
<practical implication>
<limitations and claim boundary>
<direct source links>
```

The artifact must be understandable without the conversation. Do not include process notes, retrieval logs, internal tool names, or a message telling the reader to consult the chat.

### Filename and location

- Use a descriptive lowercase hyphenated filename ending in `.md`.
- Save it in the environment's designated user-facing output or deliverable directory.
- Do not expose an intermediate, scratch, cache, or temporary path as the deliverable.

### Verification

Read the file back before returning it. Verify:

- the report was not truncated;
- headings and fenced blocks are balanced;
- direct source links are preserved;
- generic Markdown can render all mathematics;
- the file contains the same claims and boundaries as the chat report.

### Handoff

Place exactly one artifact link after the report, as the final response element:

```markdown
[Download the Markdown report](/absolute/path/to/report.md)
```

Do not follow the link with another recap, offer, or question.

## 14. Redundancy Audit

Before sending, inspect every paragraph, list item, heading, diagram, and code block.

Delete or merge an element unless it adds at least one of:

- a new operation,
- a new dimension or unit,
- a concrete intermediate or result,
- a causal reason,
- evidence,
- an exception or limitation,
- a practical implication.

Check these common failures:

- intuition repeated as contrast;
- mechanism repeated in flow captions;
- flow repeated in pseudocode prose;
- shapes repeated in a separate efficiency section;
- example followed by a general recap;
- `why it works` restating the steps;
- `so what` restating a technical benefit without an external consequence;
- result headline repeated after table interpretation;
- conclusion echoing the opening.

If removing a section preserves all unique information, remove it.
