# Link Seams

**Book location:** Chapter 4
**Technique:** Substitute a different compiled unit at link time by controlling which library or classpath entry the build resolves a call against — without touching the source of the calling code

---

## Core Concept

A link seam exists wherever the build system, not the source text, decides
which compiled implementation of a name gets bound to a call site.

- The **seam** is the unresolved external reference in the compiled output.
- The **enabling point** is outside the program text entirely — in a
  makefile, linker script, classpath variable, GPR project file, or IDE
  build setting.

Because the enabling point is in the build system, link seams can break
dependencies that are otherwise unreachable: calls to third-party
libraries, hardware drivers, operating system primitives — anything whose
source you cannot modify.


## Ada — Alternate Package Body

Unlike JAVA or C++, Ada has no global functions and no separately visible linker stage. The
equivalent of a link seam is providing an **alternate package body** for
a unit whose spec the production code `with`s. The spec is the contract
and never changes. The enabling point is the GPR project file (or
`gnatmake` source search path), which controls which body is compiled
into the partition.

Like the book, the example's goal is to observe the effects of the Graphics package
without directly watching hardware image generation. Thus the stubs/graphic_sense replaces
Graphics packages when testing

## The Seam **with Graphics**
No need to address the spec

```ada
--  cross_plane_figure.adb — source unchanged between production and test builds
with Graphics;

package body Cross_Plane_Figure is
   procedure Rerender (Figure : in Figure_Type) is
   begin
      Graphics.Draw_Text
        (Figure.X, Figure.Y, Figure.Label, Figure.Clip_Len);
      Graphics.Draw_Line
        (Figure.X, Figure.Y, Figure.X + Figure.Clip_Len, Figure.Y);
      Graphics.Draw_Line
        (Figure.X, Figure.Y, Figure.X, Figure.Y + Figure.Drop_Len);
      if not Figure.Shadow_Box then
         Graphics.Draw_Line
           (Figure.X + Figure.Clip_Len, Figure.Y,
            Figure.X + Figure.Clip_Len, Figure.Y + Figure.Drop_Len);
         Graphics.Draw_Line
           (Figure.X, Figure.Y + Figure.Drop_Len,
            Figure.X + Figure.Clip_Len, Figure.Y + Figure.Drop_Len);
      end if;
   end Rerender;
end Cross_Plane_Figure;
```
## The Enabling Point  **GPR Source_Dirs**

```ada
-- production.gpr
project Production is
   for Source_Dirs use ("src");
   for Object_Dir  use "obj/production";
   for Main use ("main.adb");
end Production;
```

```ada
-- test.gpr
project Test is
   --  List stubs/ first so graphics_sense.adb shadows src/graphics.adb
   for Source_Dirs use ("stubs", "src", "test");
   for Object_Dir  use "obj/test";
   for Main use ("test_rerender.adb");
end Test;
```
The directory layout looks like this:
```
src/
  graphics.adb          ← production body (calls real hardware drivers)
  cross_plane_figure.adb
stubs/
  graphics_sense.adb    ← sensing stub body
  graphics_sep.adb      ← separation stub body
test/
  test_rerender.adb
```
GNAT takes the first body it finds in `Source_Dirs` order. Listing `stubs`
before `src` means `stubs/graphics_sense.adb` is used in the test build
and `src/graphics.adb` is used in production.

## Sensing
This takes a bit of following but it is worth it.  By itself the new graphics_sep.adb is difficult to test.
It separates the need to see the hardware images, but doesn't give any returns.
To actually see the arguments, counts, order, graphics_sense.adb 
Lets start with the production code

### Production Code

```ada
--  graphics.ads — spec, never changes
package Graphics is
   procedure Draw_Text
     (X, Y        : Integer;
      Text        : String;
      Clip_Length : Natural);

   procedure Draw_Line
     (First_X, First_Y,
      Second_X, Second_Y : Integer);
end Graphics;
```

```ada
--  graphics.adb — production body, talks to real hardware
package body Graphics is
   procedure Draw_Text
     (X, Y : Integer; Text : String; Clip_Length : Natural) is
   begin
      Hardware_Text_Driver.Render (X, Y, Text, Clip_Length);
   end Draw_Text;

   procedure Draw_Line
     (First_X, First_Y, Second_X, Second_Y : Integer) is
   begin
      Hardware_Line_Driver.Render (First_X, First_Y, Second_X, Second_Y);
   end Draw_Line;
end Graphics;
```


```ada
--  cross_plane_figure.ads
package Cross_Plane_Figure is
   type Figure_Type is tagged private;
   procedure Rerender (Figure : in Figure_Type);
private
   type Figure_Type is tagged record
      X, Y       : Integer;
      Label      : String (1 .. 64);
      Clip_Len   : Natural;
      Drop_Len   : Natural;
      Shadow_Box : Boolean;
   end record;
end Cross_Plane_Figure;
```



### Separation Stub Body
If you only want to separate the code from Graphics.

```ada
--  stubs/graphics_sep.adb — no hardware dependency
package body Graphics is
   procedure Draw_Text
     (X, Y : Integer; Text : String; Clip_Length : Natural) is
   begin
      null;
   end Draw_Text;

   procedure Draw_Line
     (First_X, First_Y, Second_X, Second_Y : Integer) is
   begin
      null;
   end Draw_Line;
end Graphics;
```

### Sensing Stub Body
If you want to verify behavior

Sensing state lives in the package body — package-private. Test accessors
are exposed through a child package compiled only in test builds.

```ada
--  stubs/graphics_sense.adb
with Ada.Containers.Vectors;

package body Graphics is

   type Line_Action_Type is record
      First_X, First_Y, Second_X, Second_Y : Integer;
   end record;

   package Line_Vectors is new
     Ada.Containers.Vectors (Positive, Line_Action_Type);

   Recorded_Lines : Line_Vectors.Vector; --private to body
   Label_Count    : Natural := 0;       --private to body

   procedure Draw_Text
     (X, Y : Integer; Text : String; Clip_Length : Natural) is
   begin
      Label_Count := Label_Count + 1;
   end Draw_Text;

   procedure Draw_Line
     (First_X, First_Y, Second_X, Second_Y : Integer) is
   begin
      Recorded_Lines.Append ((First_X, First_Y, Second_X, Second_Y));
   end Draw_Line;

end Graphics;
```
**Recorded_Lines** and **Label_Count** are declared inside package body and hidden. 
A child package can see these values.

```ada
--  stubs/graphics-test_support.ads — child package, test builds only
package Graphics.Test_Support is
   function  Line_Count               return Natural;
   function  Label_Count              return Natural;
   function  Get_Line (N : Positive)  return Line_Action_Type;
   procedure Reset;
end Graphics.Test_Support;
```

```ada
--  stubs/graphics-test_support.adb
package body Graphics.Test_Support is

   function Line_Count return Natural is
   begin
      return Natural (Recorded_Lines.Length);
   end Line_Count;

   function Label_Count return Natural is
   begin
      return Graphics.Label_Count;
   end Label_Count;

   function Get_Line (N : Positive) return Line_Action_Type is
   begin
      return Recorded_Lines (N);
   end Get_Line;

   procedure Reset is
   begin
      Recorded_Lines.Clear;
      Label_Count := 0;
   end Reset;

end Graphics.Test_Support;
```
Print statements here could reveal the returns but Feather's of internal
tests is a better standard. 

### Test Procedure

```ada
with Ada.Assertions;        use Ada.Assertions;
with Cross_Plane_Figure;    use Cross_Plane_Figure;
with Graphics.Test_Support;

procedure Test_Rerender is
   Figure : Figure_Type :=
     (X => 0, Y => 0,
      Label      => "simple" & (7 .. 64 => ' '),
      Clip_Len   => 6,
      Drop_Len   => 4,
      Shadow_Box => False);
begin
   Graphics.Test_Support.Reset;

   Rerender (Figure);

   --  1 label + 4 lines for a non-shadow-box figure
   Assert (Graphics.Test_Support.Label_Count = 1,
           "Expected 1 Draw_Text call, got" &
           Graphics.Test_Support.Label_Count'Image);
   Assert (Graphics.Test_Support.Line_Count = 4,
           "Expected 4 Draw_Line calls, got" &
           Graphics.Test_Support.Line_Count'Image);

   declare
      First_Line : constant Line_Action_Type :=
                     Graphics.Test_Support.Get_Line (1);
   begin
      Assert (First_Line.First_X = 0 and First_Line.First_Y = 0,
              "Expected first Draw_Line to start at origin");
   end;
end Test_Rerender;
```


Running 
```
gprbuild -P test.gpr 
```
kicks off the seam

With the changed GPR
---

## Ada Notes

**`with` clause is the seam.**  In Ada, `with Graphics` in
`cross_plane_figure.adb` is the seam and the GPR `Source_Dirs` order is
the enabling point. In both cases the source of the calling unit is
unchanged between production and test builds — which is exactly what the
book's definition of a link seam requires.

**One spec, multiple bodies.** Ada's separate compilation model
guarantees that every body is compiled against its spec. Swapping bodies
at the build level cannot silently omit a subprogram — the compiler
rejects any body that fails to satisfy its spec. 

**Child package for test accessors.** Sensing state (`Recorded_Lines`,
`Label_Count`) is in the package body — invisible to callers. Exposing it
for tests requires `Graphics.Test_Support`, compiled only in test builds.
The production spec stays entirely clean.

**No source editing.** `cross_plane_figure.adb` is byte-for-byte identical
in production and test builds. The seam is purely in the GPR project file.

---

## Ravenscar Implications

**The enabling point is outside the runtime — no profile impact.** Swapping
a package body via GPR `Source_Dirs` is a compile-time decision. The
Ravenscar profile restricts runtime constructs (tasking, allocation, etc.);
it places no restriction on which compilation units are included in the
partition. The seam mechanism itself is fully Ravenscar-compatible.

**Sensing stub uses `Ada.Containers.Vectors` — forbidden under Ravenscar.**
`Line_Vectors.Vector` and `Append` call the heap allocator internally. If
the test partition runs with `pragma Restrictions (Ravenscar)` active — for
example in a hardware-in-the-loop test where the profile is enforced
throughout — replace the vector with a fixed-size array:

```ada
--  Ravenscar-safe sensing inside graphics_sense.adb
Max_Lines  : constant := 64;

type Line_Array_Type is array (1 .. Max_Lines) of Line_Action_Type;

Recorded_Lines : Line_Array_Type;
Line_Count     : Natural := 0;

procedure Draw_Line
  (First_X, First_Y, Second_X, Second_Y : Integer) is
begin
   if Line_Count < Max_Lines then
      Line_Count := Line_Count + 1;
      Recorded_Lines (Line_Count) :=
        (First_X, First_Y, Second_X, Second_Y);
   end if;
end Draw_Line;
```

Size `Max_Lines` to the worst-case number of draw calls in a single test
invocation — a deliberate design decision that Ravenscar forces to be
explicit.

**Stub body-level state is not task-safe.** `Recorded_Lines`, `Line_Count`,
and `Label_Count` are plain package-body variables — shared mutable state
with no concurrency protection. If `Rerender` is called from multiple tasks
(possible in a Ravenscar partition), their writes will race. Wrap the stub
state in a protected object:

```ada
protected type Stub_Log_Type is
   procedure Record_Line  (Action : Line_Action_Type);
   procedure Record_Label;
   function  Line_Count   return Natural;
   function  Label_Count  return Natural;
   function  Get_Line (N : Positive) return Line_Action_Type;
   procedure Reset;
private
   Lines       : Line_Array_Type;
   N_Lines     : Natural := 0;
   N_Labels    : Natural := 0;
end Stub_Log_Type;

--  Must be at library level — satisfies No_Local_Protected_Objects
Stub_Log : Stub_Log_Type;
```

Under Ravenscar: at most one *entry* per protected object — procedures and
functions are unrestricted — so all operations above are valid. `Stub_Log`
is declared at package-body level, which satisfies
`No_Local_Protected_Objects`.

**Test partition versus production partition.** In most projects, the test
partition does not carry `pragma Restrictions (Ravenscar)`. Use
`Ada.Containers.Vectors` freely in test-only code. Apply the bounded
container replacements described above only when the test partition itself
must run under the profile — hardware-in-the-loop or on-target testing
scenarios where the full Ravenscar runtime is the only runtime available.
