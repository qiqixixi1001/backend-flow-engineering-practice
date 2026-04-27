# Backend Flow Engineering 07: How Project Library Manages Technology and Standard Cell Context

> Author: Darren H. Chen  
> Direction: Backend Flow / EDA tool engineering / flow infrastructure  
> demo: **LAY-BE-07_project_library**  
> tags: Backend Flow, EDA, Project Library, LEF, Liberty, Technology File, Library Context

Before a backend tool can truly understand a design, it must understand the technology and library context in which the design exists.

A Verilog netlist can say:

```verilog
INVX1 U1 (.A(a), .Y(n1));
DFFQX1 U2 (.D(n1), .CK(clk), .Q(q));
```

But it does not tell the tool:

```text
How large is INVX1?
Where are its pins?
What is its timing arc?
Can it be placed in this row?
What are its power pins?
What metal layers are available?
What routing rules apply?
```

Those answers come from the project library.

---

## 1. Library context comes before design interpretation

Without library context, a netlist is only a connection graph of names.

With library context, each instance can be connected to physical, timing, and rule information.

The backend tool needs to know:

```text
standard cell masters
macro abstracts
pin geometries
site and row rules
routing layers
via rules
timing arcs
power models
operating conditions
```

Only then can the design become a physical implementation object.

---

## 2. Project Library is not one file

A real library context usually includes multiple model types.

| Model | Typical content | Used by |
|---|---|---|
| Technology file | layers, units, rules, vias | floorplan, routing, DRC |
| LEF | abstracts, pins, blockages, sites | placement and routing |
| Liberty | timing, power, functions, constraints | timing and optimization |
| GDS / OASIS | final layout geometry | stream-out and signoff |
| RC model | interconnect parasitics | timing and extraction |
| EM / IR rule | current and voltage limits | power network analysis |
| Antenna rule | antenna limits | routing checks and repair |

Project Library organizes these files into one coherent context.

---

## 3. Libraries become internal models

Library files are not merely read and forgotten.

They are converted into internal objects.

A LEF standard cell may become:

```text
cell master
cell boundary
pin shapes
routing obstruction
site compatibility
power and ground pins
```

A Liberty cell may become:

```text
function
timing arcs
setup / hold constraints
pin direction
capacitance tables
power models
cell attributes
```

A technology rule may become:

```text
routing layer
cut layer
preferred direction
min width
min spacing
via rule
manufacturing grid
```

This conversion is why library loading is a database construction step.

---

## 4. Name consistency is critical

Library problems often start with names.

Examples:

```text
cell exists in Liberty but not in LEF
cell exists in LEF but not in Liberty
netlist references a missing master
GDS cell name differs from LEF name
pin names differ across views
corner libraries use inconsistent naming
```

If names do not align, link and downstream stages will fail or become unreliable.

Project Library must make logical, physical, timing, and layout names consistent.

---

## 5. View consistency is also critical

One standard cell may have several views:

| View | Source | Meaning |
|---|---|---|
| logical | Verilog / function | logic behavior |
| physical abstract | LEF | placement and routing view |
| timing | Liberty | delay and constraints |
| power | Liberty / power model | power analysis |
| layout | GDS / OASIS | final geometry |

All views must describe the same cell.

If not, errors can appear later as routing failures, timing gaps, pin mismatches, or signoff discrepancies.

---

## 6. Library verification should happen early

Loading library files successfully does not prove they are consistent.

Useful checks include:

```text
cell name matching
LEF / Liberty matching
pin name matching
pin direction matching
site compatibility
cell height consistency
power / ground pin completeness
timing arc presence
clock pin definition
macro blockage completeness
GDS / LEF boundary consistency
```

Finding these issues before design import saves time later.

---

## 7. Project Library and Design Database

The relationship can be summarized as:

```text
Project Library: defines the available world
Design Database: instantiates objects in that world
```

Example:

```text
Library: INVX1 master cell
Design: U123 instance of INVX1
```

The library explains what INVX1 is. The design database explains where U123 is and how it is connected.

---

## 8. Suggested demo structure

```text
LAY-BE-07_project_library/
├─ data/
│  ├─ tech/
│  ├─ lef/
│  ├─ liberty/
│  └─ netlist/
├─ scripts/
│  └─ run_library_check.csh
├─ tcl/
│  ├─ 01_load_technology.tcl
│  ├─ 02_load_lef.tcl
│  ├─ 03_load_liberty.tcl
│  ├─ 04_verify_library.tcl
│  └─ 05_report_library_summary.tcl
├─ reports/
│  ├─ technology_layer_summary.rpt
│  ├─ loaded_libraries.rpt
│  ├─ library_cell_summary.rpt
│  ├─ lef_liberty_match.rpt
│  └─ library_verification.rpt
└─ README.md
```

The demo should prove that libraries can be loaded, queried, checked, and summarized.

---

## 9. Key takeaway

Project Library is not path management.

It is the engineering layer that converts technology and cell files into a usable backend context.

If this layer is weak, every downstream result becomes questionable.
