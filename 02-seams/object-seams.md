# Object Seams

**Book location:** Chapter 4
**Technique:** Exploit polymorphism to substitute behavior at a place where the runtime type of an object controls which method executes

---

## Core Concept

A seam is a place where you can alter behavior in a program without editing
at that place. An object seam exists wherever a dispatching call resolves
its target at runtime based on the type of the receiver. Because the type
can be varied at the call site's enabling point — wherever the object is
constructed or injected — the call can be redirected to a test double
without touching the method body that contains it.

---

## Java — Not a Seam (No Enabling Point)

`cell` is constructed inside the method. The call to `Recalculate` is
statically bound to `FormulaCell`'s implementation. Nothing outside the
method can substitute a different behavior.

```java
public class CustomSpreadsheet extends Spreadsheet {
    public Spreadsheet buildMartSheet() {
        Cell cell = new FormulaCell(this, "A1", "=A2+A3");
        cell.Recalculate();   // NOT a seam — type fixed at construction inside this method
    }
}
```

## Java — Object Seam Created (Cell Passed In)

Moving `cell` to a parameter creates an enabling point: the argument list
of `buildMartSheet`. The caller now decides which `Recalculate` runs.

```java
public class CustomSpreadsheet extends Spreadsheet {
    public Spreadsheet buildMartSheet(Cell cell) {
        cell.Recalculate();   // IS a seam — enabling point is the argument list
    }
}
```

Test — pass any `Cell` subclass, including a sensing test double:

```java
public class TestingCell extends Cell {
    public boolean recalculateCalled = false;

    @Override
    public void Recalculate() {
        recalculateCalled = true;
    }
}

public class CustomSpreadsheetTest extends TestCase {
    public void testBuildMartSheetCallsRecalculate() {
        TestingCell cell = new TestingCell();
        CustomSpreadsheet sheet = new CustomSpreadsheet();
        sheet.buildMartSheet(cell);
        assertTrue(cell.recalculateCalled);
    }
}
```

## Java — Static Method Seam (Override via Subclass)

A `private static` call is a seam once made `protected` and overridden
in a testing subclass. The enabling point is where the subclass object
is constructed.

```java
// Production — Recalculate made protected (was private static)
public class CustomSpreadsheet extends Spreadsheet {
    public Spreadsheet buildMartSheet(Cell cell) {
        Recalculate(cell);
    }

    protected void Recalculate(Cell cell) {
        // real recalculation logic
    }
}

// Testing subclass — overrides at the enabling point
public class TestingCustomSpreadsheet extends CustomSpreadsheet {
    public boolean recalculateCalled = false;

    @Override
    protected void Recalculate(Cell cell) {
        recalculateCalled = true;
    }
}
```

---

## Ada — Not a Seam

`Formula_Cell_Type` is constructed inside the procedure body. `Recalculate`
dispatches only to that type's override. No enabling point exists.

```ada
package Spreadsheets is
   type Custom_Spreadsheet_Type is new Spreadsheet_Type with null record;
   procedure Build_Mart_Sheet (Sheet : in out Custom_Spreadsheet_Type);
end Spreadsheets;
```

```ada
package body Spreadsheets is
   procedure Build_Mart_Sheet (Sheet : in out Custom_Spreadsheet_Type) is
      Cell : Formula_Cell_Type :=
               Make_Formula_Cell (Sheet'Access, "A1", "=A2+A3");
   begin
      Recalculate (Cell);   --  NOT a seam — type is fixed at construction
   end Build_Mart_Sheet;
end Spreadsheets;
```

## Ada — Object Seam Created (Classwide Parameter)

Declaring `Cell` as `Cell_Interface'Class` creates the seam. Ada dispatches
`Recalculate (Cell)` at runtime to whichever concrete type was passed.
The enabling point is the argument list of `Build_Mart_Sheet`.

```ada
package Spreadsheets is
   type Custom_Spreadsheet_Type is new Spreadsheet_Type with null record;

   procedure Build_Mart_Sheet
     (Sheet : in out Custom_Spreadsheet_Type;
      Cell  : in out Cell_Interface'Class);   -- enabling point: argument list
end Spreadsheets;
```

```ada
package body Spreadsheets is
   procedure Build_Mart_Sheet
     (Sheet : in out Custom_Spreadsheet_Type;
      Cell  : in out Cell_Interface'Class)
   is
   begin
      Recalculate (Cell);   --  IS a seam — dispatches on Cell's runtime type
   end Build_Mart_Sheet;
end Spreadsheets;
```

## Ada — Sensing Test Type

```ada
package Testing_Cells is
   type Testing_Cell_Type is new Cell_Interface with record
      Recalculate_Called : Boolean := False;
   end record;

   overriding
   procedure Recalculate (Cell : in out Testing_Cell_Type);
end Testing_Cells;
```

```ada
package body Testing_Cells is
   overriding
   procedure Recalculate (Cell : in out Testing_Cell_Type) is
   begin
      Cell.Recalculate_Called := True;
   end Recalculate;
end Testing_Cells;
```

## Ada — Test

```ada
with Ada.Assertions; use Ada.Assertions;
with Spreadsheets;   use Spreadsheets;
with Testing_Cells;  use Testing_Cells;

procedure Test_Build_Mart_Sheet_Calls_Recalculate is
   Sheet : Custom_Spreadsheet_Type;
   Cell  : Testing_Cell_Type;        --  enabling point: we choose the type here
begin
   Build_Mart_Sheet (Sheet, Cell);

   Assert (Cell.Recalculate_Called,
           "Expected Recalculate to be called on Cell");
end Test_Build_Mart_Sheet_Calls_Recalculate;
```

## Ada — Subtype Override Seam (Static Method Equivalent)

When a call is not yet dispatching on a parameter, promote it to a
primitive of the spreadsheet tagged type and override in a testing child.
The enabling point becomes wherever the child type object is constructed.

```ada
--  Production: Recalculate is a primitive — now overridable
package Spreadsheets is
   type Custom_Spreadsheet_Type is new Spreadsheet_Type with null record;

   procedure Build_Mart_Sheet
     (Sheet : in out Custom_Spreadsheet_Type;
      Cell  : in out Cell_Interface'Class);

   procedure Recalculate
     (Sheet : in out Custom_Spreadsheet_Type;
      Cell  : in out Cell_Interface'Class);
end Spreadsheets;
```

```ada
--  Testing child type: overrides Recalculate
package Testing_Spreadsheets is
   type Testing_Custom_Spreadsheet_Type is
     new Custom_Spreadsheet_Type with record
      Recalculate_Called : Boolean := False;
   end record;

   overriding
   procedure Recalculate
     (Sheet : in out Testing_Custom_Spreadsheet_Type;
      Cell  : in out Cell_Interface'Class);
end Testing_Spreadsheets;
```

```ada
package body Testing_Spreadsheets is
   overriding
   procedure Recalculate
     (Sheet : in out Testing_Custom_Spreadsheet_Type;
      Cell  : in out Cell_Interface'Class)
   is
   begin
      Sheet.Recalculate_Called := True;
   end Recalculate;
end Testing_Spreadsheets;
```

---

## Ada Notes

**`Cell_Interface'Class` is the seam carrier.** Declaring the parameter
as a classwide type is what creates the object seam. A non-classwide
parameter (`Cell_Interface` without `'Class`) is statically bound — no
dispatch, no seam.

**Enabling point is always the call site.** The production code inside
`Build_Mart_Sheet` is identical before and after the refactor. Only what
is passed to it changes. This mirrors Java exactly.

**`overriding` is enforced, not advisory.** Ada 2005 requires `overriding`
on any subprogram that overrides a parent's primitive. If the parent
signature changes and the override no longer matches, the compiler rejects
it at the declaration — stronger than Java's optional `@Override`.

**Record fields for sensing.** A record component with a default
initializer (`Recalculate_Called : Boolean := False`) guarantees clean
state on each object construction — no equivalent of a forgotten reset
between tests. Java testing subclasses use public boolean fields for the
same purpose.

**No protected/private on dispatch.** Java uses `protected` to give a
testing subclass access to override a method. In Ada any tagged type in a
package that `with`s the parent can extend and override its primitives —
control access through which packages are compiled into the test partition,
not through visibility keywords.

---

## Ravenscar Implications

**Dynamic dispatch is fully Ravenscar-compliant.** The profile places no
restriction on dispatching through classwide types. `Recalculate (Cell)` on
a `Cell_Interface'Class` parameter works unchanged in a Ravenscar
partition. Ada's OO dispatch mechanism is fully available in
high-integrity real-time code.

**`Testing_Cell_Type` has no Ravenscar concerns.** The sensing type
contains only a `Boolean` field — no heap allocation, no protected type,
no task type. It can be declared as a local `aliased` variable in a
sequential test procedure with no restriction impact.

**Object scope under `No_Task_Hierarchy`.** Ravenscar requires all tasks
to be at library level. A task that calls `Build_Mart_Sheet` must receive
its `Cell_Interface'Class` argument from an object whose lifetime spans
the task's execution. Declare the cell object in a library-level package,
not inside a procedure body:

```ada
--  Library-level — satisfies No_Task_Hierarchy scope requirement
package System_State is
   Active_Cell : aliased Formula_Cell_Type;
end System_State;

task body Spreadsheet_Task is
begin
   loop
      Build_Mart_Sheet (Sheet, System_State.Active_Cell);
      delay until Next_Release;
   end loop;
end Spreadsheet_Task;
```

**Sensing across a task boundary requires a protected object.**
If a test needs to verify that `Recalculate_Called` was set *by a task*,
the boolean must be read after a synchronization point. Wrap it in a
protected object at library level:

```ada
protected type Sensing_Guard_Type is
   procedure Set_Called;
   function  Was_Called return Boolean;
private
   Called : Boolean := False;
end Sensing_Guard_Type;

--  Library-level — satisfies No_Local_Protected_Objects
Recalculate_Sense : Sensing_Guard_Type;
```

The testing cell's `Recalculate` calls `Recalculate_Sense.Set_Called`,
and the test reads `Recalculate_Sense.Was_Called` after the task has
finished or after an explicit synchronization barrier.

**`No_Local_Protected_Objects` does not affect plain records.** The
`Testing_Cell_Type` and `Testing_Custom_Spreadsheet_Type` records hold
only `Boolean` fields — not protected types. Local declarations are fine
in a sequential test harness. The restriction applies only to protected
objects; plain tagged records are unrestricted.

**Subtype override seam and embedded protected objects.** If the
production spreadsheet type embeds a protected object for thread-safe
cell coordination, that object must be at library level and cannot be a
local component of the testing subtype. In that case, hold an `access` to
a library-level protected object in the record rather than embedding one
directly.

**Test partition versus production partition.** Apply `pragma Restrictions
(Ravenscar)` only to the production build. The test partition — a separate
executable in the GPR project — need not carry this restriction.
Sequential test procedures can use local `aliased` declarations and
unrestricted containers. Keep the two partitions cleanly separate in the
GPR file to avoid the Ravenscar restrictions bleeding into test code.
