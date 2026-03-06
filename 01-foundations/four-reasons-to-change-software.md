# Four Reasons to Change Software

**Book location:** Chapter 1
**Related:** [The Legacy Code Algorithm](./the-legacy-code-algorithm.md)

---

## The Taxonomy

Feathers identifies four reasons we change software:

1. **Adding a feature** — the system needs to do something it doesn't currently do
2. **Fixing a bug** — the system does something it shouldn't
3. **Improving the design** — refactoring to make the code easier to work with, without changing behavior
4. **Optimizing resource usage** — changing performance characteristics without changing behavior

---

## Why This Matters

The distinction between these four categories sounds academic until you notice how often they get confused in practice.

Fixing a bug and adding a feature look identical at the code level — both involve writing new code. The difference is about *intent and verification*. When fixing a bug, you're trying to restore behavior the system was supposed to have. When adding a feature, you're introducing behavior that never existed. The risk profile is different, and the tests you need are different.

---

## Behavior

The concept of *behavior* is central to everything that follows. A system's behavior is what it does: the outputs it produces for given inputs, the side effects it triggers, the things it stores and retrieves.

When you add a feature, you're adding behavior. When you fix a bug, you're correcting behavior. When you refactor, you're keeping behavior identical while changing structure. The entire discipline of testing in this book is about one thing: detecting unintended behavior changes.

This is why the question "did I break anything?" is so hard to answer without tests. You can read code and reason about behavior, but behavior is an empirical fact — it's what the code *does*, not what you think it does.

---

## Risky Changes

Any change risks introducing bugs. Feathers notes that the risk increases with:

- The size of the change
- How much of the existing code the change touches
- How well you understand the code being changed
- Whether tests exist to catch unexpected effects

