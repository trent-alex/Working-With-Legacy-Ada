# Preprocessing Seams

**Book location:**  — Chapter 4

**Technique:** A compiler flag, preprocessor define, or build-system variable that selects which text expansion (or file inclusion) the compiler actually sees.

Truth is advertising, Feather's does not prioritize preprocessing seams.  It is already close to link seams. 
However in our recent builds, I noticed using compilation flags for different environments so why not include one for testing.
Feathers summarises the trade-off plainly: preprocessing seams are powerful and compensate for some of C/C++'s testing obstacles, but tests that depend on them can be harder to maintain than object seams, so they are best reserved for cases where dependencies are pervasive and no better alternative exists.

---


## Concept

A **preprocessing seam** exploits the fact that some languages (chiefly C and C++) run a text-substitution stage *before* the compiler touches the source. 

Ada has a specific tool for preprocessing `gnatprep`
Reference Adacore documentation at [AdaCore-GNATPREP](https://learn.adacore.com/courses/GNAT_Toolchain_Intro/chapters/gnat_tools.html#gnatprep)

```
account_update.adb  →  gnatprep          →  filtered source
with --$ directives    -DTESTING=True        no --$ lines remain
                            ↑                       ↓
                       flag: TESTING          gnatcc (compiler)
                       (same .adb,            sees clean Ada
                        two runs)

production build  ──┐
TESTING not set     ├──→ gnatprep
                    │    compiler never sees --$ lines
test build      ────┘
TESTING=True
```




## Ada Translation

Ada has no macro preprocessor in its standard form, but it provides two mechanisms that achieve the same testing goal more safely:

| C/C++ mechanism | Ada equivalent |
|---|---|
| `#define` macro replacement | Generic subprogram instantiation or a renamed subprogram |
| `#include "localdefs.h"` | `gnatprep` conditional inclusion **or** a GPR scenario variable selecting an alternate package body |
| `-DTESTING` compile flag | `gnatprep` `-D` switch **or** a GPR `external` scenario variable |

The GPR scenario variable approach (alternate package bodies) is the most idiomatic and is covered in the **Link Seams** file. This file focuses on the `gnatprep` path because it is the closest structural analogue to the C preprocessor seam.

---

### Package specification — `Account_Update` (unchanged in both modes)

The spec declares `DB_Update` as a normal procedure. Nothing here changes between production and test builds.

```ada
-- account_update.ads
with DFHLItem;
with DHLSRecord;

package Account_Update is

   MAX_ITEMS      : constant := 100;
   MASTER_ACCOUNT : constant := 0;

   -- Production signature — identical to the C extern
   procedure DB_Update
     (Account_No : Integer;
      Item       : DFHLItem.Item_Access);

   procedure Account_Update_Op
     (Account_No : Integer;
      Record_Ptr : DHLSRecord.Record_Access;
      Activated  : Boolean);

end Account_Update;
```

---

### Production body — `account_update.adb`

`gnatprep` directives appear as specially formatted comments (`--$`). In a normal production build they are stripped and `DB_Update` calls the real library.

```ada
-- account_update.adb
with DB_Library;   -- real database binding

package body Account_Update is

--$ if not TESTING

   procedure DB_Update
     (Account_No : Integer;
      Item       : DFHLItem.Item_Access) is
   begin
      DB_Library.DB_Update (Account_No, Item);
   end DB_Update;

--$ else

   Last_Item       : DFHLItem.Item_Access := null;
   Last_Account_No : Integer              := -1;

   procedure DB_Update
     (Account_No : Integer;
      Item       : DFHLItem.Item_Access) is
   begin
      Last_Item       := Item;
      Last_Account_No := Account_No;
   end DB_Update;

   function Get_Last_Account_No return Integer             is (Last_Account_No);
   function Get_Last_Item       return DFHLItem.Item_Access is (Last_Item);

--$ end if

   -----------------------------------------------------------------------
   procedure Account_Update_Op
     (Account_No : Integer;
      Record_Ptr : DHLSRecord.Record_Access;
      Activated  : Boolean) is
   begin
      if Activated then
         if Record_Ptr.Date_Stamped
           and then Record_Ptr.Quantity > MAX_ITEMS
         then
            DB_Update (Account_No, Record_Ptr.Item);
         else
            DB_Update (Account_No, Record_Ptr.Backup_Item);
         end if;
      end if;
      DB_Update (MASTER_ACCOUNT, Record_Ptr.Item);
   end Account_Update_Op;

end Account_Update;
```

**Enabling point:** pass `-DTESTING=True` to `gnatprep` before compilation, or add a `gnatprep` pre-build step in your GPR file:

```
-- myproject.gpr (relevant excerpt)
package Builder is
   for Default_Switches ("Ada") use ("-gnatprep", "-DTESTING=True");
end Builder;
```

In a production build, omit the `-DTESTING` switch entirely (or set it to `False`).

---

### Ada test — no external test library required

```ada
-- test_account_update.adb
with Ada.Assertions;  use Ada.Assertions;
with Ada.Text_IO;     use Ada.Text_IO;
with Account_Update;
with DFHLItem;
with DHLSRecord;

procedure Test_Account_Update is

   Item_A   : aliased DFHLItem.Item_Type   := (others => <>);
   Backup_A : aliased DFHLItem.Item_Type   := (others => <>);

   Record_A : aliased DHLSRecord.Record_Type :=
     (Date_Stamped => True,
      Quantity     => Account_Update.MAX_ITEMS + 1,
      Item         => Item_A'Access,
      Backup_Item  => Backup_A'Access);

begin
   --  ── Test 1: activated + quantity over threshold → primary item 
   Account_Update.Account_Update_Op
     (Account_No => 42,
      Record_Ptr => Record_A'Access,
      Activated  => True);

   --  The stub captures the *last* call, which is always the MASTER_ACCOUNT
   --  call at the bottom of Account_Update_Op.
   Assert (Account_Update.Get_Last_Account_No = Account_Update.MASTER_ACCOUNT,
           "Expected final DB_Update to use MASTER_ACCOUNT");
   Assert (Account_Update.Get_Last_Item = Item_A'Access,
           "Expected final DB_Update to use primary item");

   --  Test 2: not activated → no conditional DB_Update calls 
   declare
      Record_B : aliased DHLSRecord.Record_Type :=
        (Date_Stamped => False,
         Quantity     => 0,
         Item         => Item_A'Access,
         Backup_Item  => Backup_A'Access);
   begin
      Account_Update.Account_Update_Op
        (Account_No => 99,
         Record_Ptr => Record_B'Access,
         Activated  => False);

      --  Not activated: only the unconditional tail call to MASTER_ACCOUNT fires
      Assert (Account_Update.Get_Last_Account_No = Account_Update.MASTER_ACCOUNT,
              "Not activated: expected MASTER_ACCOUNT tail call");
   end;

   Put_Line ("All preprocessing seam tests passed.");
end Test_Account_Update;
```
---

## Ada Notes

### `gnatprep` vs. GPR scenario variables

`gnatprep` is a text-level tool (GNAT-specific, not part of the Ada standard)

GPR scenario variables (used in the Link Seams file) achieve the same goal at the *build system* level: the entire package body file is swapped, rather than sections within a single file. 

---

## Ravenscar Profile Implications

Preprocessing seams interact with Ravenscar in two ways.

### Sensing variables and task safety

In the test stub above, `Last_Item` and `Last_Account_No` are plain package-level variables. Under a single-task test harness this is fine. If the test harness exercises the unit under test from multiple tasks — for example, a hardware-in-the-loop rig — these variables must be protected:

```ada
protected type Stub_State is
   procedure Set (Account_No : Integer; Item : DFHLItem.Item_Access);
   function  Last_Account return Integer;
   function  Last_Item    return DFHLItem.Item_Access;
private
   Saved_Account : Integer                := -1;
   Saved_Item    : DFHLItem.Item_Access   := null;
end Stub_State;

Stub : Stub_State;  -- library-level; satisfies No_Local_Protected_Objects
```

The Ravenscar profile prohibits `No_Local_Protected_Objects`, so the protected object must be declared at library level, not inside a subprogram or block.

### `gnatprep` and the Ravenscar pragma

The `pragma Restrictions (Ravenscar)` must appear in every compilation unit of the production partition. When using `gnatprep` to gate the test stub, ensure the `--$ if TESTING` block does not inadvertently suppress or duplicate that pragma:

```ada
--$ if not TESTING
pragma Restrictions (Ravenscar);
--$ end if
```

### Generic instantiation and Ravenscar

The generic approach described above is fully Ravenscar-compatible: generic instantiation is a pure compile-time operation. Neither instantiation (production or test) introduces any dynamic dispatch, heap allocation, or runtime type information that would conflict with the profile. The production instantiation carries the full Ravenscar restrictions; the test instantiation, compiled into a separate test partition, may omit them.

### Dynamic allocation in stubs

Ravenscar forbids dynamic memory allocation (`No_Implicit_Heap_Allocations`, `No_Allocators`). Test stubs that cache items via access values must store them in pre-allocated fixed-size arrays rather than growing containers:

```ada
MAX_CALLS : constant := 32;

type Call_Record is record
   Account_No : Integer                := -1;
   Item       : DFHLItem.Item_Access   := null;
end record;

type Call_Log_Array is array (1 .. MAX_CALLS) of Call_Record;

protected type Stub_Log is
   procedure Record_Call (Account_No : Integer; Item : DFHLItem.Item_Access);
   function  Call_Count return Natural;
   function  Nth_Call (N : Positive) return Call_Record;
private
   Log   : Call_Log_Array;
   Count : Natural := 0;
end Stub_Log;
```

This pattern applies regardless of whether the seam is implemented via `gnatprep` or a generic the constraint is on the stub's internal storage, not on the seam mechanism itself.
