# 02 — Seams

The seam model is Feathers' most original contribution and the mental model that makes everything else in the book possible. Chapter 4 of the book introduces it; this section unpacks it and explains how the three seam types work in practice.

Once you can see code in terms of seams, you start seeing opportunities to break dependencies that weren't visible before.

---

## Files in This Section

- [What Is a Seam?](./what-is-a-seam.md) — The definition, why it matters, and enabling points
- [Object Seams](./object-seams.md) — Polymorphism as a testing tool; the most useful seam type in OO languages
- [Link Seams](./link-seams.md) — Substituting implementations at link time (C/C++) or via classpaths (Java)
- [Preprocessing Seams](./preprocessing-seams.md) — Macro-based substitution; mostly relevant in C/C++

---

## The One-Sentence Summary

**A seam is a place where you can change behavior without editing the code at that place.**

That's it. The whole seam model follows from this. The reason it matters is that when you're getting legacy code under test, you often can't edit the code you want to test — it's risky, you don't have tests yet, and changes might cascade. Seams let you alter behavior from the *outside*, leaving the production code untouched.

---

## Seam Types at a Glance

| Seam Type | Mechanism | Enabling Point | When to Use |
|---|---|---|---|
| Object | Polymorphism / subclassing | Object construction site | Default choice in OO languages |
| Link | Linker / classpath substitution | Build scripts, classpaths | When object seams aren't viable |
| Preprocessing | Compiler macros | Compiler flags / `#define` | C/C++ with pervasive dependencies |

Object seams are almost always the right first choice. Link and preprocessing seams are useful when dependencies are deeply embedded and pervasive, but they're less explicit and harder to maintain.

---

## Reading Order

Start with [What Is a Seam?](./what-is-a-seam.md), then read the seam type that's relevant to the language you're working in. The [03 — Sensing and Separation](../03-sensing-and-separation/README.md) section shows what you do *with* seams once you've found them.
