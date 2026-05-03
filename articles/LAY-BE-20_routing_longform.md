# Backend Flow Engineering 20: Routing: What Global Route, Detail Route, and Route Optimization Solve Differently

> Author: Darren H. Chen  
> Direction: Backend Flow / Physical Implementation / EDA Tool Engineering / Tcl-Based Flow Engineering  
> demo: **LAY-BE-20_routing**  
> Tags: Backend Flow, Routing, Global Route, Detail Route, Route Optimization, DRC

Routing maps a connectivity graph to manufacturable geometry under capacity, timing, signal integrity, and design-rule constraints.

This article is written as a long-form GitHub article rather than a short README note. The goal is not to list commands, but to explain the engineering model behind the stage: what objects exist in the database, what state transition the backend tool is expected to perform, what should be measured, and how a demo can be designed so that the result is inspectable.

In a real backend project, the visible command line is only the surface. Under it, the tool is continuously converting text files, technology rules, design constraints, and physical edits into a typed implementation database. A robust flow is therefore built around three questions:

```text
1. What design state is this stage supposed to create or refine?
2. Which objects and relationships must be queried to verify that state?
3. Which reports prove that the state is safe enough for the next stage?
```

The answer to those questions is the foundation of backend flow engineering.


## 1. Conceptual Model

The important point in this topic is that a backend step is never an isolated command. It is a state transition inside a larger physical implementation system. The input state contains design objects, library models, constraints, and previous physical decisions. The output state must be both usable by the next stage and explainable by reports.

For this article, the most relevant objects are:

- `net`
- `pin`
- `routing track`
- `routing layer`
- `via`
- `global route guide`
- `detail route segment`
- `design rule violation`

These objects are not just names. They are database entries with type, ownership, geometry, connectivity, timing, and sometimes manufacturing meaning. When a script manipulates them, it is manipulating the implementation state of the chip.

A practical mental model is:

```text
source files + constraints + technology context
        |
        v
backend database objects
        |
        v
stage-specific transformation
        |
        v
reports + updated database + handoff data
```

This model is simple, but it prevents a common mistake: treating a backend flow as a sequence of text commands. A mature flow treats every stage as a controlled transition from one database state to another.


## 2. Backend Architecture View

A backend tool usually exposes the stage through Tcl or another command interface. However, the real architecture is layered. A command is parsed, resolved against the current database, checked against technology and design rules, executed by an internal engine, and finally reflected in logs, reports, and updated design objects.

For this topic, a useful architecture decomposition is:

- `routing resource model`
- `global router`
- `track assigner`
- `detail router`
- `DRC checker`
- `timing updater`
- `route optimization engine`

The flow can be visualized as:

```text
      [routing resource model]
              |
              v
      [global router]
              |
              v
      [track assigner]
              |
              v
      [detail router]
              |
              v
      [DRC checker]
              |
              v
      [timing updater]
              |
              v
      [route optimization engine]
```

This architecture view matters because many backend issues are not caused by a single bad command. They are caused by a mismatch between layers. For example, a script may request a legal operation, but the database context is incomplete. A file may be syntactically valid, but the objects created from it do not match the assumptions of the next stage. A report may look clean, but only because the relevant object collection was empty.

Therefore, a GitHub demo should not only run a command. It should also show the layer boundaries: configuration, database input, stage execution, report generation, and verification checks.


## 3. Engineering Methodology

The recommended methodology is to move from feasibility to quality. Feasibility asks whether the required objects, constraints, and database context exist. Quality asks whether the result is good enough. Mixing these two questions is a common source of confusion.

For this stage, the working rules are:

- `check routing capacity before detail route`
- `separate topology planning from legal geometry`
- `review DRC and timing together`
- `track via count and layer usage`
- `keep route status reports at every routing stage`

The general methodology is:

```text
Precheck  ->  Stage execution  ->  State query  ->  Report  ->  Comparison  ->  Next-stage gate
```

The precheck phase should verify that all required inputs and assumptions exist. The execution phase should be as deterministic as possible. The query phase should inspect the database rather than relying only on log text. The report phase should write stable files that can be compared across runs. The gate phase should decide whether the next stage is allowed to proceed.

This is especially important for GitHub-style examples. A demo that only prints a successful run log is weak evidence. A demo that prints object counts, state checks, and report summaries is much stronger.


## 4. Data Model and Object Relationships

The data model behind this topic can be understood as a graph. Objects are nodes. Connectivity, hierarchy, geometry, timing arcs, rule references, and ownership are edges. The backend tool does not operate on a flat list; it operates on a structured graph with many views.

A simplified view is:

```text
logical hierarchy ---- owns ---- instances / modules
        |                         |
        |                         v
        |                    pins / ports ---- connect ---- nets
        |                         |                         |
        v                         v                         v
constraints              timing arcs / checks          physical routes
        |                         |                         |
        v                         v                         v
reports                  timing paths                  DRC / PV results
```

The most important engineering implication is that every stage must preserve cross-view consistency. A physical change may affect timing. A timing fix may affect routing. A routing fix may affect physical verification. A low-power or hierarchical constraint may affect all of them.

The right way to debug is not to ask only, "Which command failed?" The better question is:

```text
Which view of the design became inconsistent with another view?
```


## 5. Demo Design

The paired demo is:

```text
LAY-BE-20_routing
```

The demo should be self-contained. It should use a local directory structure, local sample data where possible, local Tcl scripts, local report files, and a local run manifest. It should not depend on files from another demo. This makes the example easy to copy, review, and rerun.

A recommended directory structure is:

```text
LAY-BE-20_routing/
  README.md
  config/
    env.csh
    design_config.tcl
  data/
    README.md
  scripts/
    run_demo.csh
  tcl/
    run_demo.tcl
    precheck.tcl
    report_utils.tcl
  logs/
  reports/
  output/
  tmp/
```

The demo should produce at least three categories of output:

```text
1. A main report describing the stage result.
2. A check report describing whether expected objects and files exist.
3. A log summary that separates warnings, errors, and important state messages.
```

The purpose is not to mimic a full production chip flow. The purpose is to build a minimal, inspectable engineering unit that demonstrates the principle.


## 6. What to Check in Reports

A useful report should be written for humans first and scripts second. It should make the stage decision visible: what was checked, what passed, what failed, and what should be reviewed before continuing.

For this topic, the report should include:

- `routed net percentage`
- `open and short count`
- `DRC count by rule type`
- `via count`
- `layer usage`
- `post-route timing trend`

A recommended report skeleton is:

```text
# Stage Report
Generated: <timestamp>
Demo: LAY-BE-20_routing

## Input Summary
- design files
- technology/library files
- constraint files
- configuration variables

## Object Summary
- object class
- count
- key properties
- suspicious empty collections

## Stage Result
- executed operations
- pass/fail status
- warnings/errors

## Next-Stage Gate
- ready: yes/no
- reason
- required fixes
```

The next-stage gate is important. Backend flow engineering is not only about producing a result; it is about deciding whether that result is safe to consume.


## 7. Common Pitfalls

The most common pitfalls are:

- `global-route congestion is ignored until detail-route failure`
- `shorts/opens are treated as isolated issues`
- `route timing is compared against pre-route estimates without context`
- `signal integrity is considered only after signoff extraction`

Most of these problems come from treating the flow as a command recipe rather than a database engineering system. A command recipe may work once. A database-aware flow can be debugged, compared, and transferred.

A useful debugging sequence is:

```text
1. Check whether the expected input files exist.
2. Check whether the expected database objects were created.
3. Check whether object counts match the assumption.
4. Check whether the stage changed the expected objects.
5. Check whether the reports describe the change clearly.
6. Check whether the next stage can consume the result.
```

This sequence is slower than guessing, but it produces durable engineering knowledge.


## 8. GitHub Documentation Style

For GitHub, the article should be connected with executable artifacts. A good repository page should not be only a theory note and should not be only a script dump. It should connect explanation, demo input, demo output, and engineering interpretation.

A recommended GitHub layout is:

```text
docs/articles/20_routing.md
examples/LAY-BE-20_routing/
examples/LAY-BE-20_routing/README.md
examples/LAY-BE-20_routing/reports/
```

The article explains why the stage matters. The example directory shows how the stage is represented as files and reports. The report directory shows what the user should inspect after a run.

This separation is useful because backend work has two audiences:

```text
- readers who want to understand the method;
- engineers who want to reproduce the run.
```

A long-form article serves the first audience. A demo README serves the second audience. Both are needed.


## 9. Engineering Takeaways

The central takeaway is:

> Routing maps a connectivity graph to manufacturable geometry under capacity, timing, signal integrity, and design-rule constraints.

A backend flow becomes reliable when each stage has a clear state model, clear input assumptions, clear object queries, clear reports, and clear next-stage gates. The more complex the design becomes, the less effective ad-hoc scripting becomes. What scales is not command memorization; what scales is a structured engineering method.

For this topic, the practical rules are:

```text
- understand the database objects before editing them;
- check feasibility before judging quality;
- write reports that explain the stage state;
- compare runs through stable output files;
- treat the demo as a small but complete engineering unit.
```

A backend tool can execute commands, but engineering judgment comes from how we model the state, inspect the result, and preserve the reasoning path from input to output.
