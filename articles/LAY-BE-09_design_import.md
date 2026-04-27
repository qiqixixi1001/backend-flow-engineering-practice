# Backend Flow Engineering 09: Design Import Is Not File Reading: It Builds the First Semantic Layer of the Design Database

> Author: Darren H. Chen  
> Direction: Backend Flow / EDA tool engineering / flow infrastructure  
> demo: **LAY-BE-09_design_import**  
> tags: Backend Flow, EDA, Design Import, Design Database, Verilog, LEF, Liberty, DEF

Design import is often described as reading input files:

```tcl
read_verilog top.v
read_lef tech.lef
read_liberty slow.lib
read_def floorplan.def
```

That description is incomplete.

Design import is the process of building the first semantic layer of the backend database. The tool must parse input data, create objects, resolve references, attach views, and prepare later stages to work on a consistent design model.

---

## 1. Reading a file is not the same as understanding it

A parser can read syntax successfully while the design is still not usable.

For backend work, the tool must answer:

```text
What is the top design?
Which modules are defined?
Which instances are standard cells?
Which instances are macros?
Which library masters match these references?
Which ports and nets exist?
Which constraints refer to valid objects?
Which physical objects are already present?
```

If these questions are not resolved, import is incomplete.

---

## 2. Verilog import creates logical objects

When Verilog is imported, the tool may create:

```text
modules
instances
nets
ports
pins
hierarchy
connectivity
```

But this is still a logical representation.

An instance such as:

```verilog
NAND2X1 U10 (.A(a), .B(b), .Y(y));
```

must later be connected to a library master named `NAND2X1`.

Until that happens, it is only a reference by name.

---

## 3. Library import creates master definitions

Library import creates the definitions that design instances need:

```text
standard cell masters
macro abstracts
timing views
power views
technology layers
site rules
routing rules
```

The design database depends on these definitions.

If a Verilog instance references a cell not found in the project library, the tool cannot fully interpret that instance.

---

## 4. DEF import may create physical state

DEF can add:

```text
die area
rows
components
placement status
pins
blockages
special nets
routing
```

Depending on the stage, DEF can represent a floorplan, a placed design, or a routed design.

Therefore, importing DEF is not just loading geometry. It may restore a stage of physical design state.

---

## 5. Import creates dependency between views

A meaningful backend object often requires multiple views.

Example:

```text
Instance U1
  logical source: Verilog
  physical master: LEF
  timing view: Liberty
  placement state: DEF
```

If one view is missing or inconsistent, the object is only partially valid.

Design import must expose such partial states early.

---

## 6. Common import failure types

Useful reports should distinguish failure categories:

```text
syntax parse failure
missing input file
missing library master
missing physical abstract
missing timing view
pin mismatch
port mismatch
top module not found
duplicate definition
inconsistent units
layer mismatch
```

Different failures require different fixes.

A generic “import failed” message is not enough for engineering review.

---

## 7. Import should be stage-controlled

Design import should be treated as a stage with clear inputs and outputs.

Inputs:

```text
tool environment
project library
Verilog netlist
constraints
optional DEF
import configuration
```

Outputs:

```text
import log
object summary
missing reference report
top design report
port / net / instance summary
format consistency report
```

This makes import reviewable.

---

## 8. Suggested demo structure

```text
LAY-BE-09_design_import/
├─ data/
│  ├─ netlist/
│  ├─ lef/
│  ├─ liberty/
│  └─ def/
├─ scripts/
│  └─ run_design_import.csh
├─ tcl/
│  ├─ 01_precheck_inputs.tcl
│  ├─ 02_load_libraries.tcl
│  ├─ 03_import_design.tcl
│  ├─ 04_report_import_objects.tcl
│  └─ 05_report_import_issues.tcl
├─ reports/
└─ README.md
```

The demo should focus on import observability:

```text
which files were read
which top was selected
which objects were created
which references were unresolved
which reports were generated
```

---

## 9. Key takeaway

Design import is not file reading.

It is the first stage where external design formats become internal backend database objects.

A good import stage makes that transformation visible and checkable.
