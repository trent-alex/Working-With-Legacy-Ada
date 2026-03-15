# Working Effectively with Legacy Code — Companion Guide

A topic-oriented reference companion to Michael Feathers' *Working Effectively with Legacy Code* (Prentice Hall, 2004).

This repo does not reproduce the book. It organizes, explains, and cross-references the book's core ideas so you can find what you need when you're standing in front of a difficult codebase and need a technique, not a chapter.

---

## What This Is

Feathers' book is structured as a FAQ — each chapter is named after a problem ("I Can't Get This Class into a Test Harness"). That's great for browsing, but hard to use as a reference. Over time, you forget which chapter a technique lives in. You know you need to break a dependency but you can't remember the name of the pattern.

This guide reorganizes the material by topic and technique so you can navigate by concept rather than by chapter number.

---

## How to Use This Guide

**If you're new to the book**, start with the foundations:

1. [01 — Foundations](./01-foundations/README.md) — What is legacy code, why tests matter, the core algorithm
2. [02 — Seams](./02-Seams/README.md) — The single most important mental model in the book
3. [03 — Sensing and Separation](./03-sensing-and-separation/README.md) — Fakes, mocks, and why you need them
4. [04 — Adding Features Safely](./04-adding-features/README.md) — Sprout and Wrap techniques

**If you're in the middle of a problem**, go straight to:

- [05 — Breaking Dependencies](./05-breaking-dependencies/README.md) — Full catalog of 24 dependency-breaking techniques
- [06 — Understanding Legacy Code](./06-understanding/README.md) — When you don't know what the code does
- [07 — Change Strategies](./07-change-strategies/README.md) — Finding test points, pinch points, interception points
- [08 — Problem Scenarios](./08-problem-scenarios/README.md) — Mapped directly from the book's FAQ chapters

---

## Structure

```
Working with Legacy Ada/
├── 01-foundations/
├── 02-seams/
├── 03-sensing-and-separation/
├── 04-adding-features/
├── 05-breaking-dependencies/
├── 06-understanding/
├── 07-change-strategies/
├── 08-problem-scenarios/
├── 09-language-specific/
├── code-examples/
├── GLOSSARY.md
└── CONTRIBUTING.md
```

---

## A Note on the Book

Buy the book. This guide is a reference, not a replacement. Feathers walks through extended, realistic examples that no summary can substitute for. The code examples here are kept minimal and illustrative. The real teaching is in the narrative.

*Working Effectively with Legacy Code* — Michael C. Feathers, Prentice Hall, 2004. ISBN 0-13-117705-2.

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md).
