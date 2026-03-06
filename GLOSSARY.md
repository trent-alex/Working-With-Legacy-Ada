# Glossary

Key terms used throughout *Working Effectively with Legacy Code* and this guide. These definitions matter — Feathers uses them precisely, and the techniques only make sense once the vocabulary is shared.

---

## Characterization Test

A test that documents the *actual, current behavior* of a piece of code — not what it was supposed to do, not what you think it does, but what it demonstrably does right now. Characterization tests are your safety net before touching legacy code. You write them by running the code, observing its output, and writing assertions that match that output.

The algorithm:
1. Use the code in a test harness.
2. Write an assertion you know will fail.
3. Let the failure tell you what the behavior actually is.
4. Update the test to expect the real behavior.
5. Repeat until the behavior is documented.

See: [06 — Characterization Tests](./06-understanding/characterization-tests.md)

---

## Cover and Modify

The opposite of *Edit and Pray*. Before changing code, you cover it with tests. Then you modify with confidence, using the tests to detect unintended behavior changes. This is the foundational working style the entire book is built around.

---

## Edit and Pray

The default mode in legacy systems — make a change, hope nothing breaks, deploy carefully, debug production. Feathers uses this term as a foil to *Cover and Modify*.

---

## Enabling Point

Every seam has an enabling point: a place *outside* the code you want to change where you can make the decision to use different behavior. For an object seam, the enabling point might be where the object is constructed. For a link seam, it might be a build script. For a preprocessing seam, it might be a compiler flag.

You cannot have a seam without an enabling point. If there's no place to swap the behavior from the outside, it's not a seam.

See: [02 — Seams](./02-seams/what-is-a-seam.md)

---

## Fake Object

A class you write for testing purposes that implements the same interface as a real dependency, but with a simplified, controllable implementation. Unlike mocks, fakes aren't necessarily set up with expectations — they just behave in a simple, predictable way that makes testing possible.

Example: a fake database that stores records in memory instead of making real SQL calls.

See: [03 — Fake Objects](./03-sensing-and-separation/fake-objects.md)

---

## Interception Point

A place in a call chain where you can put a test to detect changes in behavior. Feathers distinguishes between narrow interception points (close to the change, easier to set up, faster feedback) and higher-level interception points (further from the change, covering more behavior, but slower and harder to maintain). The goal is usually to find the narrowest interception point that still gives you the coverage you need.

See: [07 — Interception Points](./07-change-strategies/interception-points.md)

---

## Legacy Code

In the common industry sense: difficult-to-change code we don't understand. In Feathers' precise definition: **code without tests**. The definition that matters for this book is his: the absence of tests is what makes code risky to change, regardless of how clean or messy it is.

---

## Mock Object

A fake object that also makes assertions — it verifies that it was called in the expected way. Mocks are used when you need to *sense* whether something happened (a method was called, a value was passed) in addition to simply substituting a dependency. Distinguished from a plain fake by the fact that it can cause a test to fail based on interaction, not just return values.

See: [03 — Mock Objects](./03-sensing-and-separation/mock-objects.md)

---

## Monster Method

A method so large and tangled that it's nearly impossible to test or change safely. Feathers categorizes them as *bulleted methods* (a sequence of loosely related chunks, often separated by blank lines or comments) and *snarled methods* (one large, deeply nested block of logic). Different strategies apply to each.

See: [08 — Working with Monster Methods](./08-problem-scenarios/working-with-monster-methods.md)

---

## Pinch Point

A narrow place in a call chain through which many effects must pass — a natural chokepoint. Tests placed at a pinch point cover a large area of code with relatively few tests. Pinch points are useful for getting legacy code under coarse-grained control before finer-grained unit tests exist.

Think of a pinch point as an encapsulation boundary that exists in the code whether you planned it that way or not.

See: [07 — Pinch Points](./07-change-strategies/pinch-points.md)

---

## Seam

**A seam is a place where you can alter the behavior of a program without editing in that place.**

This is the single most important concept in the book. Seams are how you get untestable code under test without making risky changes. Every technique in the dependency-breaking catalog is essentially a way of finding or creating a seam.

Three types exist: object seams, link seams, and preprocessing seams. Object seams are the most useful in OO languages.

See: [02 — What Is a Seam?](./02-seams/what-is-a-seam.md)

---

## Sensing

One of two primary reasons for breaking dependencies in tests. You *sense* when you need to verify that a dependency was called correctly, received the right arguments, or produced a specific side effect that your production code can't directly expose. Sensing is often achieved with mock objects.

Contrasted with *separation*.

See: [03 — Sensing and Separation](./03-sensing-and-separation/README.md)

---

## Separation

One of two primary reasons for breaking dependencies in tests. You *separate* when you need to isolate the code under test from a dependency that makes testing difficult — a database, a network call, a GUI. The dependency isn't the thing you're testing; it's just in the way.

Contrasted with *sensing*.

See: [03 — Sensing and Separation](./03-sensing-and-separation/README.md)

---

## Sprout Method / Sprout Class

Techniques for adding new, tested functionality to a class you can't yet put under test. Rather than modifying the existing method (risky without tests), you write a new method or class that contains only the new behavior, test that in isolation, then call it from the original code with minimal modification.

See: [04 — Sprout Method](./04-adding-features/sprout-method.md), [04 — Sprout Class](./04-adding-features/sprout-class.md)

---

## Test Harness

The testing code and infrastructure that lets you run a piece of software in isolation. This includes the test framework (JUnit, NUnit, etc.), any setup/teardown, and the fake or mock objects that stand in for real dependencies.

---

## Testing Subclass

A subclass created specifically to expose or override behavior for testing. A common pattern: you can't test a private method directly, so you create a subclass that makes it protected, or overrides a factory method to return a fake object.

---

## Unit Test

A test that runs in less than 1/10th of a second and is small enough to help localize failures when they occur. Feathers is strict about this definition: a test that hits a database, file system, or network is not a unit test regardless of what framework runs it. Speed and isolation are what make unit tests useful as a development tool.

---

## Wrap Method / Wrap Class

Techniques for adding new, tested behavior around an existing method or class without modifying its internals. Wrap Method renames the original and replaces it with a new method that calls both the original and the new behavior. Wrap Class (decorator-style) wraps an object at a higher level.

See: [04 — Wrap Method](./04-adding-features/wrap-method.md), [04 — Wrap Class](./04-adding-features/wrap-class.md)
