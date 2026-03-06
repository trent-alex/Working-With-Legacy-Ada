# What Is Legacy Code?

**Book location:** Preface, Chapter 1
**Related:** [Working with Feedback](./working-with-feedback.md), [Glossary — Legacy Code](../GLOSSARY.md#legacy-code)

---

## The Common Definition vs. The Working Definition

The industry definition of legacy code is roughly *difficult-to-change code we don't understand* — tangled structure, missing documentation, etc

Feathers proposes a different definition: **legacy code is code without tests**.

This is deliberately precise, and it initially sounds too simple. What do tests have to do with whether code is bad?

Test are about providing *feedback* — the ability to verify, quickly and objectively, that your change did what you intended and didn't break what you didn't touch. Without that feedback, every change is a leap of faith. The quality of the leap depends on your skill, your understanding of the code, your memory of what changed last week, and a dozen other factors that vary unpredictably.

With tests, confident change becomes a repeatable process.

---

## The Practical Consequence

If legacy code is code without tests, then the path forward is clear: the work of dealing with legacy code is the work of incrementally adding tests. Not a big-bang rewrite, not a comprehensive test suite before you're allowed to change anything — just enough tests around the specific thing you need to change, right now, so you can change it with confidence.

That's the thing the entire book is built around.

---

## See Also

- [The Legacy Code Algorithm](./the-legacy-code-algorithm.md) — the repeatable process for applying this in practice
- [Working with Feedback](./working-with-feedback.md) — what unit tests actually provide
- [Characterization Tests](../06-understanding/characterization-tests.md) — how to write tests for code you don't fully understand
