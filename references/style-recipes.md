# Style Recipes

Canonical presentation patterns for the Algorithm Intuition Explainer.

## Contents

1. Intuition and Contrast
2. Forward Flow
3. Equation Explanation
4. Shape Explanation
5. Tiny Example
6. Pseudocode
7. Why It Works
8. Experimental Results
9. Progressive-Depth Follow-Ups
10. Bottom Line

## 1. Intuition and Contrast

Start with one memorable mechanism.

```text
The key intuition is:

<method> works by <simple mechanism>,
so it does not have to <hard thing>.

Instead of:
   <old or difficult approach>

It does:
   <new approach>
```

Use a literal explanation first. Add analogy only after the literal mechanism is clear.

## 2. Forward Flow

```text
Input + any supervision or context
   ↓
Step 1
   <what is computed>

   In simple terms:
   “<plain translation>”

   ↓
Step 2
   <what is computed>

   In simple terms:
   “<plain translation>”

   ↓
Output
```

For a user-provided flow, preserve its order and insert the new explanation exactly where the concept occurs.

## 3. Equation Explanation

Explain the whole purpose, then each part.

```text
The equation is trying to:
   <one-sentence purpose>

Term 1:
   <mathematical term>
   “<plain meaning>”

Term 2:
   <mathematical term>
   “<plain meaning>”

Together:
   “<combined tradeoff or operation>”
```

For an optimization objective:

```text
The first term says:
   “Fit the data.”

The second term says:
   “Avoid an unstable or overly complex solution.”
```

## 4. Shape Explanation

Give shapes before prose.

```text
If:
   X has shape N × d
   W has shape d × h

Then:
   XW has shape N × h

For each example:
   a length-d input becomes a length-h vector.
```

Then give a numeric example:

```text
N = 100 examples
d = 64 input features
h = 256 hidden units

X: 100 × 64
W: 64 × 256
XW: 100 × 256
```

## 5. Tiny Example

Use the smallest example that preserves the mechanism.

```text
For a tiny example:
   <2–3 items or one small vector>

Step 1:
   <simple operation>

Step 2:
   <simple result>
```

Do not use a toy example that changes the actual algorithm.

## 6. Pseudocode

```text
function algorithm_step(input):

    # Explain what is being created.
    intermediate = simple_operation(input)

    # Explain why the next operation is useful.
    output = transform(intermediate)

    return output
```

For training:

```text
for each layer / step / batch:

    make the local object or prediction
    compare it with the target or objective
    update or solve the parameters
    pass the improved representation forward
```

Add shapes next to variables when dimension reasoning matters.

## 7. Why It Works

Explain causal links, not slogans.

```text
The algorithm works because:

1. <mechanism> preserves or extracts <needed information>.
2. <mechanism> injects or computes <task-relevant signal>.
3. The next stage receives an easier, cleaner, or better-conditioned problem.
```

Add the non-magic caveat when appropriate:

```text
This does not mean the intermediate target is perfect.
It means the target is useful enough to guide the next step.
```

## 8. Experimental Results

Use this recipe for benchmark tables, ablations, scaling curves, and result-heavy papers.

### Start with the headline

```text
The main result is:

<method> performs better on <task>,
especially under <condition>.
```

### Benchmark tables

Translate the table as comparisons, not a number dump.

```text
Rows:
   methods being compared

Columns:
   datasets, tasks, or metrics

Best values:
   where the method wins

Close values:
   where it is competitive

Weak values:
   where it struggles
```

Then summarize:

```text
Compared with <baseline>, the method is better on <where>,
roughly tied on <where>,
and worse on <where>.

In simple terms:
“The method helps most in <condition>, not everywhere.”
```

Mention metric direction explicitly:

```text
Higher is better: accuracy, F1, BLEU, throughput
Lower is better: loss, latency, memory, error rate
```

Do not assume the direction when the metric is specialized; verify it from the source.

### Ablations

```text
Full method
   ↓ remove or replace one component
Ablated method
   ↓ compare the result
```

Explain:

```text
The ablation asks:
“What happens if we remove this piece?”

If the score drops:
“This component was doing useful work.”

If the score barely changes:
“This component may not be essential in this setup.”
```

Do not conclude that a component is universally useless from one ablation.

### Scaling curves

```text
Increase data / parameters / compute / context / workers
   ↓
Measure whether performance improves,
plateaus,
becomes unstable,
or changes relative to a baseline.
```

Interpret the shape:

```text
Steep improvement:
   scaling helps strongly

Flat curve:
   the measured setup has saturated

Noisy curve:
   the result may be unstable or under-measured

Crossing curves:
   one method is better in the small regime,
   another in the large regime
```

### Claim boundary

Always finish result interpretation with:

```text
So the results support:
   <what the evidence directly shows>

They do not prove:
   <broader claim requiring more evidence>
```

## 9. Progressive-Depth Follow-Ups

Do not repeat the entire paper or algorithm summary. Zoom into the requested object.

### Level 1: Direct meaning

```text
<X> means:
<one plain sentence>
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

### Level 4: Why it matters

```text
The important part is:
<X> helps the algorithm <useful effect>,
so the next step can <be easier, more stable, or better informed>.
```

### Level 5: Technical detail

Only then add equations, shapes, implementation details, or caveats.

### Follow-up answer pattern

```text
<Direct answer: Yes / No / definition>.

<X> is:
   <plain definition>

In the flow:

<previous object>
   ↓
<X>
   ↓
<next object>

For a tiny example:
   <example>

Why it matters:
   <one causal sentence>
```

## 10. Bottom Line

End with one memorable sentence:

```text
Bottom line:
<method> works by <central mechanism>,
which makes <original hard problem> easier.
```
