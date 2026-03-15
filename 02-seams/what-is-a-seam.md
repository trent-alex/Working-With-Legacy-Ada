# What Is a Seam?

**Book location:** Chapter 4
**Related:** [Object Seams](./object-seams.md), [Link Seams](./link-seams.md), [Preprocessing Seams](./preprocessing-seams.md), [Glossary — Seam](../GLOSSARY.md#seam)

---

## The Definition

> A seam is a place where you can alter behavior in your program without editing in that place.

This is Feathers' formal definition, and the italics are his. The "without editing in that place" is what makes it powerful. In legacy code work, you often can't safely edit the code you want to test — you don't have tests yet, you don't fully understand the effects. Seams let you change what code *does* from somewhere else.

---

## Why Seams Matter for Testing

The challenge of testing legacy code is usually not that the logic is hard to understand. It's that the code can't easily be instantiated in a test harness. It might depend on a database connection established in a constructor. It might write to a file system as a side effect. It might call a global function that makes a network request.

To test such code, you need a way to substitute those dependencies with something controllable. Seams are where that substitution happens.

---

## Enabling Points

Every seam has an **enabling point**: a place, separate from the seam itself, where you can make the decision to use different behavior.

This is the part people miss. Finding a seam isn't enough — you need a place to *act on it* without modifying the code under test. If there's nowhere to make that decision from the outside, it's not a seam.

For example: a virtual method call is a seam. The enabling point is the place where the object is constructed — that's where you decide whether to create the production class or a testing subclass. If object construction is buried inside the method you're testing with `new ConcreteClass()`, the call may be a seam in principle, but you have no enabling point — you can't substitute behavior without editing the method itself.

---

## A Concrete Example

Consider this Java code:

```java
// Not a seam — no enabling point
public Spreadsheet buildMartSheet() {
    Cell cell = new FormulaCell(this, "A1", "=A2+A3");
    cell.Recalculate();
    // ...
}
```

The call to `cell.Recalculate()` looks like it could be varied — `Cell` has subclasses. But the cell is created inside the method. You can't change which `Recalculate` is called without editing the method. No enabling point, no seam.

Now contrast:

```java
// An object seam — enabling point is the method argument
public Spreadsheet buildMartSheet(Cell cell) {
    cell.Recalculate();
    // ...
}
```

Now `cell` is passed in. You can call `buildMartSheet` in a test with any `Cell` subclass you want, including a testing subclass that doesn't actually recalculate anything. The enabling point is the argument list of `buildMartSheet`.

The code in `buildMartSheet` didn't change at all between these two versions. The difference is entirely in whether there's a place outside the method to vary the behavior.

---

## The Three Seam Types

The physical mechanism of substitution differs depending on the language and the seam type:

- **Object seams** — Exploit polymorphism. You pass a different object, or construct a testing subclass. Most useful in OO languages. See [Object Seams](./object-seams.md).
- **Link seams** — Substitute a library or class at link time (C/C++) or by manipulating the classpath (Java). The enabling point is the build configuration. See [Link Seams](./link-seams.md).
- **Preprocessing seams** — Use compiler macros to replace function calls before compilation. Mostly a C/C++ technique. The enabling point is a compiler define. See [Preprocessing Seams](./preprocessing-seams.md).

---

## Learning to See Seams

The practical skill is learning to read code and ask: *where can behavior be varied from the outside?* Every virtual method call, every interface, every constructor argument, every import statement is a potential seam. The question is whether there's an enabling point that makes it exploitable.

When you get used to seeing code this way, untestable code starts to look different. Instead of a wall, you see a surface with cracks — places where small changes can create enough leverage to get tests in place.
