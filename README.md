# Algorithm Intuition Explainer

Current release: **0.3.0**

A reusable ChatGPT Skill for explaining algorithms, machine-learning papers, systems, equations, code, and experimental results in a clear progressive teaching style.

## Installation

1. Obtain the packaged `skill.zip` file.
2. Import it through the Skills interface available in your ChatGPT environment.
3. Confirm that **Algorithm Intuition Explainer** appears in your Skill library.

Exact interface locations may vary by ChatGPT surface and release.

## Activation

The Skill can activate automatically when a request matches its description, including:

- explaining a paper or technical method,
- asking for the intuition behind an algorithm,
- walking through an equation or tensor shape,
- interpreting benchmark tables or ablations,
- generating conceptual pseudocode,
- asking progressive follow-ups such as “what does X mean?”

It may also be selected explicitly from the available Skills when the interface supports explicit selection.

## Example Prompts

### Papers

```text
Explain the key intuition in this paper: <URL>
```

```text
Walk me through the method and experiments in this paper without assuming I know the notation.
```

```text
Explain the attention mechanism in “Attention Is All You Need.”
```

### Algorithms and methods

```text
Explain ridge regression intuitively and show why the penalty helps.
```

```text
Show the training flow for a two-layer MLP using this algorithm.
```

### Equations and shapes

```text
Explain every term in this loss function in plain language.
```

```text
What are the dimensions of Q, K, V, and the attention matrix?
```

### Results

```text
Explain this benchmark table, including where the method wins and where it does not.
```

```text
What does this ablation actually support?
```

### Follow-up teaching

```text
What does “random projection” mean in the flow you just showed?
```

```text
Why does this step happen before the output layer?
```

## Source Handling

For named papers, the Skill distinguishes among:

- source-grounded explanations,
- explanations based on user-provided text,
- memory-based high-level explanations.

Short excerpts are classified cautiously. The Skill should use a provisional or undetermined paper type when the excerpt does not reveal the paper’s primary contribution.

## Package Contents

```text
algorithm-intuition-explainer/
├── SKILL.md
├── README.md
├── CHANGELOG.md
├── agents/
│   └── openai.yaml
└── references/
    ├── style-recipes.md
    └── evaluation-cases.md
```

## Design Notes

- `SKILL.md` controls routing, grounding, and required behavior.
- `references/style-recipes.md` is the canonical source for response templates.
- `references/evaluation-cases.md` contains regression cases for maintaining quality as the Skill evolves.

## Limitations

- A short excerpt may not support confident paper classification.
- Simplified analogies are not literal implementation descriptions.
- Precise benchmark claims require the relevant paper sections or another primary source.
- The Skill improves explanation structure; it does not replace source verification for recent or niche work.
