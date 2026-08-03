---
name: generate-simple-explanation
description: Explain algorithms, machine-learning papers, systems, formulas, experimental results, training procedures, code, and technical methods in a clear step-by-step teaching style with zero semantic redundancy. Use when the user asks for intuition, key ideas, why a method works, why it matters, practical implications, pseudocode, tensor shapes, equation meaning, benchmark or ablation interpretation, paper walkthroughs, progressive follow-up teaching such as “what does X mean?”, or a downloadable Markdown explanation or report. Route named papers through source retrieval and paper-type detection; route non-paper topics directly through the relevant workflow. Give each section a distinct job, omit sections that add no new information, save broad reports as browser-safe `.md` deliverables, and enforce portable Unicode mathematics unless the user explicitly requests LaTeX for a confirmed math-capable renderer.
---

# Generate Simple Explanation

Explain like a patient technical tutor. Optimize for understanding per sentence. Every paragraph must add information that has not already been established.

## 1. Route the Request First

Classify the request before answering:

1. **Named paper or paper-specific claim**
   - Apply the Paper Retrieval Protocol.
   - Detect the paper type when enough evidence is available.
   - Use only the paper-walkthrough components that answer the request.
   - For a broad paper walkthrough or report, create the downloadable Markdown deliverable defined in Section 11.

2. **Named algorithm, model, training rule, or technical method**
   - Start with the problem and one literal mechanism sentence.
   - Add flow, shapes, equations, examples, causal reasoning, implications, or limitations only when each contributes distinct information.
   - State what changes in practice or research when the user asks why it matters or the significance is not obvious.
   - Do not add paper source-mode or paper-type labels unless the user asks about a specific paper.

3. **Equation, symbol, loss, or mathematical object**
   - Explain its purpose, terms, shapes or units, and position in the larger flow.
   - Add a tiny example only when it demonstrates something not already evident from the equation explanation.

4. **Code or pseudocode**
   - Explain inputs, outputs, shapes, and the main operations.
   - Give conceptual pseudocode before implementation detail unless executable code is requested.
   - Do not repeat line-by-line prose when comments in the pseudocode already explain the operations.

5. **Experimental result, table, ablation, or scaling curve**
   - Explain the setup, comparison, evidence, and claim boundary.
   - State one practical or research implication when it is supported and not already obvious from the result.
   - State the headline once; do not repeat it as a second summary.
   - Use the Experimental Results Recipe in `references/style-recipes.md`.

6. **Follow-up question about an earlier explanation**
   - Answer the exact question first.
   - Reuse prior context instead of restarting the full explanation.
   - Use only the minimum progressive-depth levels needed from `references/style-recipes.md`.

## 2. Paper Retrieval Protocol

Before explaining a named paper, decide and visibly flag exactly one source mode near the start of the answer when paper-specific claims matter.

```text
Mode: Source-grounded
```

Use when the paper URL, PDF, DOI, arXiv page, abstract, or full text is available through browsing, retrieval tools, connectors, uploaded files, or pasted text that includes the relevant sections. Prefer this mode for recent, niche, result-sensitive, or citation-sensitive papers. Cite the original paper or official source when citations are supported.

```text
Mode: User-provided text
```

Use when the user supplied the relevant abstract, excerpt, notes, or document and external retrieval is unavailable or unnecessary. State that the explanation is based on the supplied material.

```text
Mode: Memory-based
```

Use only for stable, well-known papers when retrieval is unavailable, citations were not requested, and a high-level explanation is acceptable. State that precise paper details may need verification.

Apply this retrieval decision:

1. Retrieve when the user gives a URL, DOI, arXiv ID, paper title with citations requested, or a recent or niche paper.
2. Ask for the PDF, abstract, or relevant excerpt when retrieval fails and paper-specific fidelity is necessary.
3. Proceed memory-based only when the paper is stable and the user wants high-level intuition rather than precise results.
4. Use supplied text without extra retrieval when it fully supports the requested explanation.
5. Separate explicit paper claims from simplified intuition and analogy.

## 3. Handle Short or Partial Excerpts Safely

Do not assume a short excerpt reveals the whole paper's primary contribution.

When only a partial excerpt is available:

1. Classify the **excerpt content** first.
2. Classify the paper only when the excerpt contains strong evidence about its primary contribution.
3. Mark any paper classification as provisional.
4. Use `Paper type: Undetermined from excerpt` when evidence is insufficient.
5. Explain only what the excerpt supports; do not invent missing method, evaluation, or system details.

Use these labels when needed:

```text
Paper type: Theoretical / Method — provisional
Paper type: Empirical / Benchmark — provisional
Paper type: Systems — provisional
Paper type: Survey — provisional
Paper type: Hybrid — provisional
Paper type: Undetermined from excerpt
```

Evidence hints:

- New objective, theorem, architecture, algorithm, or training rule → likely Theoretical / Method.
- Datasets, metrics, baselines, benchmark tables, ablations, or scaling results → likely Empirical / Benchmark.
- Runtime, memory, throughput, compiler, networking, storage, accelerator, or deployment design → likely Systems.
- Taxonomy, literature organization, method families, or open problems → likely Survey.

Classify by the paper's **primary contribution**, not merely by the supplied section.

## 4. Detect the Paper Type

After reading enough of the title, abstract, introduction, method, system design, and results, select one label:

```text
Paper type: Theoretical / Method
Paper type: Empirical / Benchmark
Paper type: Systems
Paper type: Survey
Paper type: Hybrid
```

Use `Hybrid` when two or more contribution types are central rather than incidental.

Treat each list below as an ordering guide, not a requirement to create a separate section for every item. Merge adjacent items when they communicate one idea, and omit items outside the user's scope.

### Theoretical / Method

Explain in this order:

1. Problem or limitation.
2. New mechanism, objective, proof object, or architecture.
3. Role of components or symbols not already defined.
4. Forward algorithm flow, when order matters.
5. Training or optimization signal, when requested or central.
6. Proof intuition or guarantee, when central.
7. Causal reason the method should help.
8. Practical or research implication, when requested or not obvious.
9. Assumptions, caveats, and where guarantees do not apply.

### Empirical / Benchmark

Explain in this order:

1. Task, dataset, and evaluation setup.
2. Baselines and metric direction.
3. Headline comparison.
4. Ablation or scaling evidence that adds a different claim.
5. What the results support and do not prove.
6. What decision, capability, or research direction the bounded result may affect, when useful.

### Systems

Explain in this order:

1. Operational bottleneck and system boundary.
2. Architecture components and request or dataflow.
3. The scheduling, storage, memory, networking, compiler, or hardware technique.
4. Reliability or failure behavior, when relevant.
5. Evaluation workloads, metrics, tradeoffs, and deployment caveats.

### Survey

Explain in this order:

1. Field-level question and taxonomy.
2. Major method families compared along consistent dimensions.
3. Open problems.
4. Beginner-friendly reading path.

### Hybrid

Combine only the relevant recipes. Clearly separate:

- how the method or system works,
- how it was evaluated,
- what the evidence supports,
- what changes in practice or research if that evidence holds.

Do not repeat the method description while interpreting the experiments.

## 5. Core Teaching Workflow

For any technical explanation:

1. Identify one central mechanism sentence.
2. Decide the smallest set of representations needed to answer the request.
3. Assign each selected representation one exclusive role:
   - **Intuition:** the literal mechanism and essential contrast.
   - **Flow:** operation order and intermediate outputs.
   - **Equation:** exact mathematical relationship.
   - **Shapes:** dimensional compatibility.
   - **Example:** one concrete execution.
   - **Pseudocode:** executable conceptual procedure.
   - **Why it works:** cause-and-effect reasoning and assumptions.
   - **Evidence:** observed support.
   - **Implication / So what:** an external consequence for a capability, decision, cost, risk, user outcome, or research direction.
   - **Limitations:** failure conditions, tradeoffs, or scope boundaries.
4. Explain each object at first use and refer back to it briefly later.
5. Stop when the user's question is answered.

Do not treat every representation as mandatory. Include a second representation only when it answers a different question from the first.

Do not confuse `Why it works` with `So what`. The former explains the internal causal chain; the latter explains the external consequence. Include an implication when the user asks why the topic matters or when its significance would otherwise remain unclear. Label an implication as projected with language such as `can`, `could`, or `may` unless the source directly measured it.

Use `references/style-recipes.md` as the canonical source for presentation patterns, including non-redundant answer architecture, flows, equations, shapes, examples, pseudocode, implications, experimental results, follow-ups, and browser-safe Markdown mathematics.

## 6. Non-Redundancy Contract

Enforce semantic, not merely verbal, non-redundancy.

- **One idea, one home:** explain a fact fully in one location. Later mentions may point back to it but must not re-explain it.
- **Rephrasing is still repetition:** a new heading, analogy, equation, diagram, or summary does not justify repeating the same claim.
- **Representations need distinct jobs:** keep both a flow and pseudocode only when the flow teaches conceptual order and the pseudocode teaches execution detail.
- **Merge overlapping sections:** combine intuition with contrast, shapes with parameter arithmetic, or mechanism with component roles when separate sections would repeat content.
- **Delete empty transitions:** remove sentences that only announce or recap the next section.
- **Avoid summary echoes:** do not add a bottom line when the opening already states the takeaway. For long answers, place the takeaway either at the start or the end, not both.
- **Use short back-references:** write `This is the same update shown in the flow above` rather than restating the update.
- **Prefer omission over coverage theater:** leave out a requested format only when it would duplicate another format, and briefly state the omitted format's unique information inline if needed.

Apply this novelty test to every paragraph, list item, and section:

> Does this add a new operation, dimension, example result, causal reason, evidence item, exception, or practical implication?

If the answer is no, delete or merge it.

## 7. Grounding and Claim Boundaries

Prefer the original paper, official documentation, or primary source for named methods and claimed results.

Distinguish clearly among:

- **Explicit claim:** stated or demonstrated by the source.
- **Reasonable intuition:** a causal interpretation supported by the mechanism.
- **Simplified analogy:** a teaching device that is not a literal implementation description.

Do not overclaim from a benchmark table, ablation, or one dataset. State what the evidence supports and what it does not prove.

Do not present an intuitive interpretation of a factor, dimension, head, neuron, or latent direction as uniquely identifiable unless the source supports that interpretation.

## 8. Style Rules

- Keep sentences short and concrete.
- Avoid dense paragraphs before the intuition is established.
- Use vertical code-block flows only when sequence order is easier to understand visually.
- Explain symbols at first use.
- Translate equation terms into plain language without narrating the equation symbol by symbol.
- State vector or batch orientation when equations and pseudocode use different conventions.
- Answer ordering questions directly: `No. The order is forward, not backward.`
- Answer dimension questions with shapes first, then a numerical example only when it adds clarity.
- Preserve a user-provided flow and insert the requested explanation at the correct point.
- Use phrases such as `In simple terms:` and `For a tiny example:` only when they introduce new information.
- Avoid unnecessary source-mode or paper-type labels for ordinary non-paper concepts.
- Prefer precise comparisons such as `a 256× reduction` or `1/256 as many trainable parameters` over ambiguous wording such as `256 times fewer`.

## 9. Mandatory Browser-Safe Markdown Formatting

When creating or updating a `.md` file for generic browser or Markdown viewing, enforce the Browser-Safe Markdown Math recipe in `references/style-recipes.md`.

Unless the user explicitly requests LaTeX and confirms a math-capable renderer such as MathJax or KaTeX:

- Do not use LaTeX delimiters such as `\(...\)`, `\[...\]`, or `$$...$$`.
- Do not place mathematical equations in fenced code blocks. Reserve fenced blocks for flows, shapes, pseudocode, terminal output, and actual code.
- Write standalone equations as normal Markdown paragraphs, preferably bold, using portable Unicode notation.
- Use readable symbols such as `Z̃ₗ`, `Aₗ₋₁`, `Aᵀ`, `(...)⁻¹`, `λ`, `≈`, `×`, and `√` where practical.
- Use italics or plain text for mathematical variables. Do not render a variable such as *l* as an inline-code pill unless it is literally a program identifier.
- Never put a standalone `=` line outside a fenced block; Markdown may interpret it as a Setext heading underline.
- Break equations into named terms and bullet explanations when a long expression becomes hard to read in Unicode.

Before saving the file, verify that it renders without a LaTeX extension and contains no accidental heading markers or equation-only code fences.

## 10. Pseudocode and Code Behavior

Use pseudocode when the user asks how something is generated, trained, updated, or executed.

Keep pseudocode conceptual unless executable code is requested. Include:

- input and output,
- important dimensions,
- comments only where the operation's purpose is not already obvious,
- the connection between each line and the algorithm flow when that mapping adds information.

Do not repeat the pseudocode as a prose walkthrough. Explain only non-obvious lines, invariants, shape changes, or implementation choices.

When executable code is requested, provide a runnable minimal implementation and state which paper details are simplified or omitted.

## 11. Downloadable Markdown Report

Create a `.md` deliverable by default for:

- broad paper walkthroughs,
- comprehensive technical reports,
- or any request that explicitly asks for a report or downloadable Markdown file.

Do not create a file for a narrow explanation or follow-up unless the user requests one.

For a qualifying report:

1. Write a self-contained Markdown copy of the complete report to the environment's designated user-facing output or deliverable directory. Use the workspace's `outputs/` directory when that is the established convention.
2. Name it with a descriptive lowercase hyphenated slug, such as `building-a-production-shopping-agent-at-scale.md`.
3. Include the title, source mode, paper type when applicable, explanation, evidence, practical implications, claim boundaries, and direct source links needed to understand the report without the chat transcript.
4. Apply the Browser-Safe Markdown Math rules in Section 9. Keep citations as ordinary Markdown links to the original sources.
5. Read the saved file back and verify that it is non-empty, complete, and free of accidental LaTeX-only notation, malformed fences, or truncated sections.
6. End the chat response with one clickable local-file link labeled `Download the Markdown report`. Make this link the final element of the response.

The file may intentionally mirror the report shown in chat; do not add a second summary merely to introduce the download.

Use the Downloadable Markdown Report recipe in `references/style-recipes.md` for the artifact structure and handoff pattern.

## 12. Reference Files

Read only the relevant reference:

- `references/style-recipes.md` — canonical non-redundant output patterns for explanations, results, shapes, equations, pseudocode, and follow-up teaching.
- `references/evaluation-cases.md` — regression tests for paper types, excerpts, non-paper topics, follow-ups, Markdown portability, and redundancy control.

## 13. Final Quality Check

Before answering, verify:

1. Did the answer route correctly as paper, algorithm, equation, code, result, or follow-up?
2. If paper-specific, is the source mode accurate?
3. If only an excerpt is available, is classification provisional or undetermined?
4. Does the answer establish intuition before unnecessary implementation detail?
5. Does every selected section have a distinct job?
6. Is each object, operation, or claim explained only once?
7. Do equations, flows, examples, and pseudocode add different information rather than paraphrasing one another?
8. Does the causal explanation add reasons rather than repeat mechanics?
9. Are experimental claims bounded by the available evidence?
10. Did the answer avoid restarting the full explanation for a narrow follow-up?
11. Can any sentence or section be deleted without losing unique information? If yes, delete it.
12. Is a conclusion omitted when it would duplicate the opening takeaway?
13. When significance was requested or not obvious, did the answer state a claim-bounded practical or research implication, distinguish measured from projected impact, and avoid repeating the mechanism or result?
14. If this was a broad report, was a self-contained `.md` file saved in the designated deliverable directory and linked as the final element of the response?
15. If a `.md` file was created, was it read back and verified for completeness, balanced fences, working source links, and browser-safe notation without LaTeX support?
