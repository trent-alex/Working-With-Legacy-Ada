# 01 — Foundations

The conceptual bedrock. B Feathers covers this material in Part I of the book (*The Mechanics of Change*, Chapters 1–5). It's short, and most developers skip it to get to the practical stuff. Don't. The seam model is lit, err i mean it changes how you look at code manipulations.

---

## Files in This Section

- [What Is Legacy Code?](./what-is-legacy-code.md) — The working definition this entire guide is built on
- [Four Reasons to Change Software](./four-reasons-to-change-software.md) — The taxonomy of change and why it matters


---

## The Core Argument

The book's central claim is simple: **code without tests is legacy code**, regardless of how recently it was written or how clean it looks. This is a provocative definition, and Feathers earns it.

The reason tests are the dividing line isn't moral — it's practical. Without tests, you can't verify that a change preserved existing behavior. That means every change carries unknown risk. You can be skilled, careful, and experienced, and you still can't know. Tests are the mechanism that converts unknown risk into known risk.

Everything else in the book — seams, fakes, dependency-breaking techniques — is in service of one goal: getting code under test so you can change it confidently.

---

## Reading Order

Feathers recommends reading this section in order. Each file here sets up vocabulary and concepts that later sections depend on. The [Glossary](../GLOSSARY.md) is also worth skimming before going deeper.
