# Backend Flow Engineering 10: From Import to Link: Why link_project Determines Whether the Tool Understands the Design

> Author: Darren H. Chen  
> Direction: Backend Flow / EDA tool engineering / flow infrastructure  
> demo: **LAY-BE-10_import_link_current_design**  
> tags: Backend Flow, EDA, Design Import, link_project, Current Design, Project Library, Database

After design import, many engineers assume the design is ready.

That is not always true.

Import can create raw objects and references, but linking determines whether those references are resolved against the project library and whether the tool can truly understand the design.

This article explains the path from import to link and the role of current design context.

---

## 1. Import creates references

When a Verilog netlist is imported, the tool may see:

```text
module top
instance U1 of INVX1
instance U2 of DFFQX1
net n1 connecting U1/Y to U2/D
```

But `INVX1` and `DFFQX1` are still names that must be matched to library definitions.

The imported design has references. Linking resolves them.

---

## 2. Linking binds design to library

A link step answers:

```text
Does every standard cell instance have a master?
Does every macro instance have a physical abstract?
Are pin names consistent?
Is the top module known?
Are unresolved references allowed?
Are black boxes expected or accidental?
Are timing views available?
```

If this step fails, the design database is not fully usable.

---

## 3. link_project is a semantic validation point

The command name varies by tool, but the concept is common:

```text
link the imported design against the project library
```

This is the point where raw design data becomes a connected design model.

Before link:

```text
instances may be unresolved
library views may be unattached
timing arcs may be unavailable
physical masters may be missing
```

After successful link:

```text
instances can point to masters
pins can be checked against views
queries become meaningful
reports become more reliable
```

---

## 4. Current design matters

Many backend commands depend on a current design context:

```tcl
current_design top
get_cells *
report_timing
report_utilization
```

If the current design is not set or points to the wrong scope, later commands can fail or return misleading results.

A stable flow should explicitly set and report current design state.

Example checks:

```text
current design exists
top name matches expectation
instance count is nonzero
port count is nonzero
linked cell count matches expectations
unresolved count is zero or justified
```

---

## 5. Common link issues

Link failures often come from library context, not only from netlist syntax.

Common issues include:

```text
missing standard cell master
missing macro abstract
Liberty / LEF mismatch
pin name mismatch
wrong top module
duplicate module definition
black box not configured
library corner not loaded
case-sensitive name mismatch
```

A good flow should report these separately.

---

## 6. Link reports are required

After link, the flow should generate reports such as:

```text
link_summary.rpt
unresolved_references.rpt
current_design.rpt
library_binding_summary.rpt
missing_master_cells.rpt
pin_mismatch.rpt
blackbox_summary.rpt
```

These reports explain whether the tool has enough information to continue.

---

## 7. Link is the gate before physical stages

Physical stages should not start if the design is not linked.

For example:

```text
floorplan should not start before top is valid
placement should not start before masters are resolved
routing should not start before physical abstracts are available
timing should not start before timing views and constraints are valid
```

Link is therefore a stage gate.

---

## 8. Suggested demo structure

```text
LAY-BE-10_import_link_current_design/
├─ data/
│  ├─ netlist/
│  ├─ lef/
│  └─ liberty/
├─ scripts/
│  └─ run_import_link.csh
├─ tcl/
│  ├─ 01_load_project_library.tcl
│  ├─ 02_import_netlist.tcl
│  ├─ 03_link_project.tcl
│  ├─ 04_set_current_design.tcl
│  └─ 05_report_link_status.tcl
├─ reports/
│  ├─ current_design.rpt
│  ├─ link_summary.rpt
│  └─ unresolved_references.rpt
└─ README.md
```

The demo should intentionally distinguish:

```text
import succeeded
link succeeded
current design is valid
```

These are related but not identical.

---

## 9. Key takeaway

Import tells the tool what design data exists.

Link tells the tool whether that design data can be understood in the project library context.

Current design tells later commands which design scope they are operating on.

A mature backend flow treats all three as explicit checks.
