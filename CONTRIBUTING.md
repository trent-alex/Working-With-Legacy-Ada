# Contributing

Contributions are welcome. This guide exists to be useful to working developers, so practical improvements — clearer explanations, better examples, corrections — are the highest priority.

---

## What Helps

**Clarifying an explanation** — If something reads as confusing or imprecise, open a PR with a clearer version. Explaining *why* a technique works (not just how) is particularly valuable.

**Adding code examples** — The `code-examples/` directory holds illustrative snippets. Examples should show the *before and after* of applying a technique, kept short enough to understand without surrounding context. Java is the reference language since it's what Feathers uses throughout the book. Additional languages are welcome.

**Fixing cross-reference links** — Many files reference each other. If a link is broken or points to the wrong place, fix it.

**Flagging inaccuracies** — If something in this guide contradicts what the book actually says, open an issue. The book's content takes precedence. This guide is a companion, not a rewrite.

---

## What Doesn't Help

**Reproducing extended passages from the book.** This guide summarizes and explains — it doesn't transcribe. Significant quotes are avoided for copyright reasons. Explaining a concept in your own words, with your own example, is both legally safer and often more useful anyway.

**Opinionated additions not grounded in the book.** The value of this repo is that it maps reliably to Feathers' ideas. If you have a strong opinion about how to handle legacy code that diverges significantly from the book, that's better suited to a blog post than a PR here.

---

## File Format

Each technique or concept file follows a consistent structure:

```markdown
# Technique Name

**Book location:** Chapter X / Part Y
**Related techniques:** [Link](./link.md), [Link](./link.md)

## The Problem It Solves

One paragraph: what situation calls for this technique?

## How It Works

Explanation of the technique. No bulleted lists — prose.

## Example

Before and after code, kept short.

## Trade-offs

Honest discussion of when this technique is the right choice and when it isn't.

## See Also

Links to related files in this repo.
```

---

## Pull Request Process

1. Fork the repo and create a branch.
2. Make your changes. Keep commits focused — one topic per commit.
3. Open a PR with a description of what you changed and why.
4. If you're adding a new file, make sure it's linked from the relevant `README.md`.

There's no formal review SLA. This is a small project maintained on a best-effort basis.
