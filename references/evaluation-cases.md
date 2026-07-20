# Evaluation Cases

Use these cases as regression tests when updating the Skill. Evaluate behavior and structure, not exact wording.

## Contents

1. Theoretical / Method Paper
2. Empirical / Benchmark Paper
3. Systems Paper
4. Survey Paper
5. Hybrid Paper
6. Short Excerpt
7. Non-Paper Algorithm
8. Equation Follow-Up
9. Shape Follow-Up
10. Experimental Results Follow-Up

## 1. Theoretical / Method Paper

**Prompt**

```text
Explain the attention mechanism in “Attention Is All You Need” intuitively.
```

**Expected behavior**

- Use a source mode because a named paper is requested.
- Classify as Theoretical / Method or Hybrid, depending on response scope.
- Start from the sequence-modeling limitation.
- Explain query, key, value, scores, scaling, softmax, and weighted values.
- Show a forward flow and at least one tiny token example.
- Explain why direct token-to-token paths and parallelism help.

**Must not**

- Begin with tensor algebra before giving the intuition.
- Describe query/key/value only as dictionary definitions without showing their interaction.
- Claim every attention head learns a fixed linguistic role.

## 2. Empirical / Benchmark Paper

**Prompt**

```text
Explain the conclusions from this benchmark paper, including its tables and ablations.
```

**Expected behavior**

- Identify tasks, datasets, metrics, and baselines.
- State whether higher or lower is better for each important metric.
- Summarize where the method wins, ties, and loses.
- Explain each ablation as a component-removal or replacement test.
- End with supported and unsupported claims.

**Must not**

- List every number without interpretation.
- Treat one benchmark win as universal superiority.
- Treat a small ablation difference as proof that a component never matters.

## 3. Systems Paper

**Prompt**

```text
Explain how FlashAttention works and why it is faster.
```

**Expected behavior**

- Start from memory traffic or data-movement cost rather than only arithmetic count.
- Define system boundary and exact output equivalence.
- Show blockwise dataflow and the memory-saving idea.
- Separate algorithmic equivalence from hardware efficiency.
- Explain evaluation metrics such as runtime, memory, and throughput.

**Must not**

- Say it changes the mathematical attention result when discussing exact FlashAttention variants.
- Explain only the attention equation and omit the systems bottleneck.

## 4. Survey Paper

**Prompt**

```text
Explain the taxonomy in this survey and give me a reading path.
```

**Expected behavior**

- Identify the field-level organizing question.
- Explain major families and dimensions of comparison.
- Contrast families using the same comparison axes.
- Surface open problems.
- Give a beginner-friendly reading order.

**Must not**

- Pretend the survey proposes one central algorithm.
- Turn the answer into a chronological list without explaining the taxonomy.

## 5. Hybrid Paper

**Prompt**

```text
Explain both the method and experimental evidence in this paper.
```

**Expected behavior**

- Use `Paper type: Hybrid` when both contribution types are central.
- Separate mechanism explanation from evidence explanation.
- Use the method flow first, then the experimental-results recipe.
- State what the experiments support and what remains uncertain.

**Must not**

- Mix method steps and benchmark numbers into one confusing sequence.

## 6. Short Excerpt

**Prompt**

```text
Explain this excerpt:
“We evaluate the proposed architecture on three benchmarks and report accuracy, latency, and memory.”
```

**Expected behavior**

- Use `Mode: User-provided text`.
- Use `Paper type: Undetermined from excerpt`, or a clearly provisional classification.
- Explain that the excerpt describes evaluation but does not establish the whole paper type.
- Explain only the supported evaluation concepts.

**Must not**

- Confidently classify the entire paper as Empirical / Benchmark.
- Invent the architecture, datasets, or numerical results.

## 7. Non-Paper Algorithm

**Prompt**

```text
Explain ridge regression intuitively.
```

**Expected behavior**

- Do not show paper source-mode or paper-type labels.
- Start with “fit the data, but keep weights small.”
- Translate both objective terms into plain language.
- Explain why regularization stabilizes the solution.
- Give a tiny example or comparison with ordinary least squares.

**Must not**

- Route through paper classification.
- Present only the closed-form equation.

## 8. Equation Follow-Up

**Conversation context**

The user was shown scaled dot-product attention.

**Prompt**

```text
Why divide by sqrt(d_k)?
```

**Expected behavior**

- Answer directly before restating attention.
- Explain that dot-product magnitude grows with dimension.
- Connect large scores to softmax saturation and weak gradients.
- Give a small intuitive comparison.
- Reinsert scaling between score computation and softmax.

**Must not**

- Restart the entire Transformer explanation.

## 9. Shape Follow-Up

**Prompt**

```text
What is the dimension of the attention score matrix?
```

**Expected behavior**

- Give shapes first.
- Show `Q: N × d_k`, `K: N × d_k`, and `QK^T: N × N`.
- Explain that each row compares one token with all tokens.
- Give a numeric example such as five tokens producing a `5 × 5` matrix.

## 10. Experimental Results Follow-Up

**Prompt**

```text
What does this ablation prove?
```

**Expected behavior**

- Avoid the word “prove” unless a formal proof exists.
- Explain what component was removed or changed.
- State what the observed difference supports in this setup.
- State alternative explanations and limits.
- Connect the ablation back to the full method.
