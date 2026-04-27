# Backend Flow Engineering 11: Why cell, net, pin, and port Are the Basic Units of Backend Flow Engineering

> Author: Darren H. Chen  
> Direction: Backend Flow / EDA tool engineering / flow infrastructure  
> demo: **LAY-BE-11_design_object_model**  
> tags: Backend Flow, EDA, Design Object Model, Cell, Net, Pin, Port, Tcl

After design import and link, the backend tool no longer sees the design as a collection of files. It sees an object model.

The most basic objects are:

```text
cell
net
pin
port
```

These objects form the foundation for queries, reports, checks, physical operations, timing analysis, and stage decisions.

This article explains why understanding the object model is essential for backend flow engineering.

---

## 1. Files become objects

Before import, the design exists as files:

```text
Verilog
LEF
Liberty
DEF
SDC
```

After import and link, the tool creates database objects:

```text
modules
instances
cells
nets
pins
ports
clocks
shapes
routes
properties
```

Flow scripts should operate on these objects, not only on file names.

---

## 2. Cell: the placed or placeable instance

A cell usually represents an instance in the design database.

It may be:

```text
standard cell
macro
IO cell
filler cell
tap cell
clock buffer
spare cell
```

A cell can have properties such as:

```text
name
reference master
location
orientation
placement status
area
power domain
fixed state
attributes
```

Typical operations include:

```tcl
get_cells *
get_property $cell ref_name
report_property $cell
```

Cells are central to placement, optimization, ECO, utilization, and reporting.

---

## 3. Net: the connectivity object

A net connects pins and ports.

It may represent:

```text
signal net
clock net
power net
ground net
reset net
special net
```

A net can have properties such as:

```text
name
driver
loads
net type
routing status
capacitance
fanout
shielding
weight
```

Typical queries include:

```tcl
get_nets *clk*
get_pins -of_objects [get_nets n1]
```

Nets are the link between logical connectivity and physical routing.

---

## 4. Pin: the connection point on a cell

A pin belongs to a cell instance.

It may have:

```text
name
direction
related net
physical geometry
timing role
capacitance
clock attribute
```

Pins are important because timing and routing both depend on them.

For example:

```tcl
get_pins U1/Y
get_property [get_pins U1/Y] direction
```

A timing path is often described as a sequence of pins.

---

## 5. Port: the top-level boundary object

A port is a boundary connection of the current design.

Ports represent external interface points:

```text
clock input
reset input
data input
data output
inout pad
power or ground port
```

Typical queries:

```tcl
get_ports *
get_ports clk*
get_property [get_ports rst_n] direction
```

Ports connect internal design objects to external constraints and physical IO planning.

---

## 6. Object relationships matter

The important part is not just individual object types. It is their relationships.

```text
cell owns pins
pin connects to net
net connects multiple pins and ports
port connects design boundary to net
cell points to library master
```

A simple path may be represented as:

```text
port A -> net n1 -> pin U1/A -> cell U1 -> pin U1/Y -> net n2 -> pin U2/D
```

Backend reports are built on these relationships.

---

## 7. Collections make queries composable

EDA tools often return collections rather than plain text.

A collection can be passed to another command:

```tcl
set regs [get_cells -filter "is_sequential == true"]
report_property $regs
```

This allows scripts to build object-driven logic:

```text
find objects
filter objects
inspect properties
report results
apply stage-specific operations
```

Object collections are the practical basis of maintainable backend scripts.

---

## 8. Object properties explain design state

Objects become useful when their properties are queryable.

Examples:

```text
cell placement status
cell reference name
pin direction
net fanout
port direction
clock membership
route status
violation marker
```

A useful flow reports key properties instead of relying only on visual inspection.

---

## 9. Suggested demo structure

```text
LAY-BE-11_design_object_model/
├─ scripts/
│  └─ run_object_model_demo.csh
├─ tcl/
│  ├─ 01_query_cells.tcl
│  ├─ 02_query_nets.tcl
│  ├─ 03_query_pins_ports.tcl
│  ├─ 04_report_object_properties.tcl
│  └─ 05_report_object_relationships.tcl
├─ reports/
│  ├─ cells.rpt
│  ├─ nets.rpt
│  ├─ pins.rpt
│  ├─ ports.rpt
│  └─ object_relationships.rpt
└─ README.md
```

The demo should show how cells, nets, pins, and ports are queried and related.

---

## 10. Key takeaway

Backend flow engineering is object-model engineering.

Once files have been imported and linked, the flow should reason in terms of cells, nets, pins, ports, collections, and properties.

This is the foundation for reliable queries, checks, reports, and stage control.
