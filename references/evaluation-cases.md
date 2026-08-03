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
11. Browser-Safe Markdown File
12. Non-Redundant Method Explanation
13. So-What Explanation
14. Downloadable Markdown Paper Report


## Global Non-Redundancy Criteria

Apply these checks to every case:

- Give each section a distinct informational role.
- Treat a paraphrase, analogy, diagram, equation, or summary as redundant when it communicates no new operation, dimension, result, cause, evidence, exception, or implication.
- Merge overlapping sections rather than preserving a fixed template.
- Omit the conclusion when it repeats the opening takeaway.
- Use only the minimum progressive-depth levels needed for a follow-up.
- When significance is requested or not obvious, state one claim-bounded external consequence for a capability, decision, cost, risk, user outcome, or research direction.

**Must not**

- Repeat the central mechanism under multiple headings.
- Restate a flow as prose after showing it visually.
- Explain pseudocode line by line when comments already do so.
- Add a recap after every example or section.
- Label a mechanism recap or vague claim of importance as `So what`.

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
- State one decision or research implication supported by the results.
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
- Connect lower memory traffic to a bounded practical capability, such as longer sequences or larger batches on the tested hardware.

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
- State one practical or research consequence that follows within that boundary.

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
- Explain what design or research decision the ablation may inform.

## 11. Browser-Safe Markdown File

**Prompt**

```text
Explain the key equation in this paper and write the result to output-paper-name.md.
```

**Expected behavior**

- Treat generic Markdown rendering as having no LaTeX, MathJax, or KaTeX support.
- Use normal Markdown plus Unicode notation for mathematical displays, such as `Z̃ₗ`, `Aₗ₋₁`, `Aᵀ`, `⁻¹`, `λ`, and `≈`.
- Show standalone equations as ordinary Markdown paragraphs, preferably bold.
- Use fenced `text` blocks only for flows, shapes, or pseudocode.
- Use italics or plain text for mathematical variables in prose rather than inline-code pills.
- Validate the saved file for browser-safe rendering before returning it.

**Must not**

- Use `\[...\]`, `\(...\)`, or `$$...$$` unless the user explicitly requested LaTeX for a confirmed compatible renderer.
- Put a mathematical display in a code block merely as a fallback.
- Leave a standalone `=` line outside a fenced block, because it may become a Setext heading.
- Produce notation such as `Z_tilde_l` or `A_(l-1)` when readable Unicode notation is available.

## 12. Non-Redundant Method Explanation

**Prompt**

```text
Explain how LoRA fine-tunes a large model. Include intuition, a flow, shapes for a 4096 × 4096 matrix with rank 8, a tiny example, pseudocode, why it works, and limitations.
```

**Expected behavior**

- State the frozen-base-plus-low-rank-update mechanism once.
- Use the flow only for operation order.
- Keep shape arithmetic and parameter reduction in one section.
- Use the tiny example for concrete intermediate values rather than another general explanation.
- Use pseudocode for execution and explain only non-obvious conventions, such as row-versus-column orientation.
- Explain why LoRA can help using causal or empirical reasoning, without repeating `ΔW = BA`.
- State limitations without restating the method.
- Omit a closing bottom line when the opening already provides the takeaway.

**Must not**

- Create separate sections for `Key intuition`, `Ordinary fine-tuning`, and `Why two matrices?` when they repeat the same contrast.
- Call the rank dimensions uniquely interpretable features without qualification.
- Repeat the parameter count outside the shape section.
- Follow the example or pseudocode with a prose recap of the same operations.
- Add a conclusion that echoes the first paragraph.

## 13. So-What Explanation

**Conversation context**

The user already understands the mechanism and main result.

**Prompt**

```text
So what? Why should anyone care?
```

**Expected behavior**

- Answer with the external consequence first.
- Connect the established mechanism or evidence to a capability, decision, cost, risk, user outcome, or research direction.
- Distinguish a directly observed consequence from a projected one.
- Keep the implication within the source's claim boundary.
- Use only a short causal back-reference when needed.

**Must not**

- Restart the full mechanism or results explanation.
- Treat `why it works` and `why it matters` as interchangeable.
- Say only that the topic is `important`, `significant`, or `useful`.
- Invent real-world impact that the mechanism or evidence does not support.

## 14. Downloadable Markdown Paper Report

**Prompt**

```text
Give me a full explanation of the paper “Building a Production Shopping Agent at Scale.”
```

**Expected behavior**

- Produce the source-grounded paper report requested in chat.
- Save a self-contained copy as `building-a-production-shopping-agent-at-scale.md` in the environment's designated user-facing deliverable directory.
- Preserve the report's source mode, paper type, evidence, practical implications, claim boundaries, and direct source links in the file.
- Apply browser-safe Markdown formatting and verify the saved file by reading it back.
- End the response with exactly one clickable local-file link labeled `Download the Markdown report`.

**Must not**

- Save the deliverable only in a scratch, cache, or temporary directory.
- Require the reader to consult the chat to understand the file.
- End the response without the artifact link or place additional prose after it.
- Generate a file automatically for every narrow follow-up question.
