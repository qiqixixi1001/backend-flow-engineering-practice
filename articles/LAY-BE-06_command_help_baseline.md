# Backend Flow Engineering 06: Why a Mature Backend Flow Needs a Command Help Baseline

> Author: Darren H. Chen  
> Direction: Backend Flow / EDA tool engineering / flow infrastructure  
> demo: **LAY-BE-06_command_help_baseline**  
> tags: Backend Flow, EDA, Tcl, Command Help, Baseline, Version Control

When engineers begin writing backend scripts, they often start from an existing flow file and modify it.

That is practical, but it is not enough for long-term flow ownership.

A stable backend flow needs to know what the tool actually exposes in the current environment:

```text
Which commands exist?
Which options are supported?
Which commands belong to which stage?
Which commands need design context?
Which commands are available in batch mode?
Which interfaces changed between releases?
```

That reference is called a command help baseline.

---

## 1. What is a command help baseline?

A command help baseline is a recorded view of the tool command interface under a specific condition:

```text
tool version
run mode
platform
license setup
loaded extensions
session context
```

It records command inventory, key command help, command families, failed help queries, and environment metadata.

It is not a copy of the official manual. It is a snapshot of what the current environment actually exposes.

---

## 2. Why reference scripts are not enough

A reference script shows how one run was written.

It does not necessarily answer:

```text
Why was this command used?
What other options exist?
Is the option still supported?
Does the command require linked design data?
Is the command available in batch mode?
What changed in the new release?
```

Without a baseline, script development becomes trial-driven.

With a baseline, command usage can be checked against an explicit interface reference.

---

## 3. First question: does the command exist?

The simplest inventory is:

```tcl
info commands
```

Then commands can be filtered by pattern:

```tcl
info commands *import*
info commands *link*
info commands get_*
info commands report_*
info commands *route*
```

This creates the first fact base:

```text
Which command names are visible in this session?
```

---

## 4. Second question: are the options known?

Command existence is not enough.

Backend failures often occur at the option level:

```text
option renamed
option removed
option requires another option
option is invalid in this mode
option requires design context
```

A help baseline should collect help for key commands:

```tcl
help import
help link_project
help get_cells
help report_timing
help route
help export_def
```

Exact command names vary by tool, but the purpose is stable: record the parameter interface.

---

## 5. Command family classification

A flat command list is hard to use. Commands should be grouped by flow role.

| Family | Typical purpose |
|---|---|
| session | startup, exit, logs, parameters |
| library | technology and library setup |
| import | design data reading |
| link | database binding and reference resolution |
| query | cell, net, pin, port, property access |
| floorplan | die, core, rows, macro and IO setup |
| timing | constraints and timing reports |
| placement | placement and physical optimization |
| clock | clock tree and clock rules |
| routing | global/detail routing and route repair |
| report | summaries and diagnostics |
| export | DEF, GDS, Verilog, SDC and reports |
| ECO | controlled design modification |
| debug | error, replay, trace, inspection |

This turns command inventory into a flow map.

---

## 6. Version binding is mandatory

A command baseline must record tool version.

Backend tool releases may change:

```text
command names
option names
default values
help text
warning behavior
report format
command availability
```

A baseline should start with:

```text
Tool Version:
Platform:
Run Mode:
Date:
Command Inventory Source:
```

Without version metadata, comparison is weak.

---

## 7. Mode binding is also important

The same tool may expose different behavior in:

```text
GUI mode
batch mode
shell mode
stdin mode
no-design mode
design-loaded mode
```

A no-design baseline should be separated from a design-loaded baseline.

```text
no-design baseline: command existence and help
design-loaded baseline: object query and report behavior
```

Mixing them creates confusion.

---

## 8. Record help failures

Failed help queries are useful.

They may indicate:

```text
command unavailable
help not supported
wrong mode
license limitation
missing plugin
context requirement
```

A baseline should include a file such as:

```text
missing_or_failed_help.rpt
```

Failure is data, not noise.

---

## 9. Suggested demo structure

```text
LAY-BE-06_command_help_baseline/
├─ scripts/
│  └─ run_command_baseline.csh
├─ tcl/
│  ├─ collect_command_inventory.tcl
│  ├─ collect_help_baseline.tcl
│  └─ classify_command_family.tcl
├─ reports/
│  ├─ 01_tool_version.rpt
│  ├─ 02_all_commands.rpt
│  ├─ 03_command_family.rpt
│  ├─ 04_help_summary.rpt
│  └─ 05_missing_or_failed_help.rpt
├─ logs/
└─ README.md
```

The demo should observe the command interface without running high-impact design operations.

---

## 10. Key takeaway

A command help baseline is the engineering reference surface for a backend tool interface.

It supports:

```text
script checking
version upgrade review
team handoff
command family mapping
parameter inspection
flow maintenance
```

A mature backend flow should not rely only on memory or copied scripts. It should have a command interface baseline.
