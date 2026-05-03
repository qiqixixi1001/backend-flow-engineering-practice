# Backend Flow Engineering 09: Design Import Is Not File Reading: It Builds the First Semantic Layer of the Design Database

> Author: Darren H. Chen  
> Direction: Backend Flow / Physical Implementation / EDA Tool Engineering / Tcl-Based Flow Engineering  
> demo: **LAY-BE-09_design_import**  
> Tags: Backend Flow, EDA, Design Import, Netlist, DEF, Library, Database Semantics

Design import is often described as reading files, but that description is too shallow. Import is the first semantic construction step of the backend database. The tool is not only parsing text; it is creating modules, instances, nets, ports, pins, library references, physical components, and initial relationships between views.

If import is treated as a simple file I/O step, serious errors may be missed. A file can be syntactically valid but semantically incompatible with the rest of the project. The import stage must therefore be designed to prove that the first database layer has been built correctly.

This article is written as a long-form engineering article rather than a short README note. The goal is not to list a few steps, but to explain the engineering model behind the stage: what problem the stage solves, what internal state it creates or verifies, what report evidence should be produced, and how a small demo can be designed so that the result is inspectable.

A backend flow is not only a command sequence. It is a state transition system. Each stage starts from an input state, applies a controlled operation, and leaves an output state for the next stage. The quality of the whole flow depends on whether these state transitions are explicit, observable, and reviewable.


## 1. Conceptual Model

The concept is semantic lifting. Source files are text. The backend database is typed and relational. Import lifts text into typed objects. Verilog becomes modules, instances, ports, and nets. Liberty becomes timing libraries and cell arcs. LEF becomes layers, sites, cell abstracts, obstructions, and pin shapes. DEF becomes floorplan, rows, components, pins, and physical nets.

The success of import should be measured by database evidence, not only by the absence of parser errors. A robust import stage asks whether the expected objects exist, whether counts are reasonable, and whether unresolved references are reported.

A useful way to reason about the stage is:

```text
inputs + assumptions
        |
        v
controlled backend tool operation
        |
        v
observable state + reports + next-stage readiness
```

This model prevents a common mistake: judging a backend stage only by whether a command returned without a fatal error. A clean return is useful, but it is not sufficient. The stage must also leave enough evidence to prove that the expected state was created.


## 2. Backend Architecture View

The import architecture usually includes file inventory, parser, object creation, name resolver, database insertion, and import report. File inventory checks paths. Parsers read standard formats. Object creation builds internal representations. Name resolver attaches references to existing objects. Database insertion updates the design state. Reports expose what happened.

Each imported format should have a separate status. Combining all formats into one pass or fail result hides useful information. LEF may pass while DEF fails. Verilog may load while link fails. Liberty may parse while timing arcs are incomplete. The report must preserve this distinction.

A generic architecture view is:

```text
[launcher / Tcl entry]
        |
        v
[configuration and precheck layer]
        |
        v
[backend database or runtime context]
        |
        v
[stage-specific engine]
        |
        v
[reports, logs, and state checks]
```

The important engineering principle is separation of responsibility. The launcher should not hide design assumptions. The configuration layer should not silently change database state. The stage engine should not be treated as a black box without reports. The report layer should not rely only on human memory.


## 3. Engineering Methodology

The recommended methodology is to move from visibility to control. Visibility means the flow can show its inputs, state, actions, and outputs. Control means the flow can reject unsafe conditions before they damage the run state. In backend engineering, visibility without control creates long logs but weak reliability. Control without visibility creates rigid scripts that are difficult to debug. A mature flow needs both.

The practical sequence is:

```text
precheck -> execute stage -> query state -> write reports -> review gate -> next stage
```

The precheck phase validates assumptions. The execution phase performs the intended state transition. The query phase inspects the database or runtime state. The report phase converts internal state into durable evidence. The review gate decides whether the next stage is allowed to proceed.

This methodology is intentionally conservative. Backend stages are highly coupled. A weak assumption in an early stage can become a timing, routing, physical verification, or ECO problem much later. The earlier the flow records and checks state, the cheaper debugging becomes.


For this article, the most important working rules are:

```text
1. Make hidden assumptions visible.
2. Convert every critical state into a reportable fact.
3. Separate setup, execution, query, and review.
4. Treat warnings as engineering signals, not background noise.
5. Keep the demo small enough to understand and strict enough to be useful.
```

This is also the reason each demo in this series is designed to be independent. A demo should carry its own configuration, scripts, sample data where needed, logs, reports, temporary directory, and README files. Independence makes the example easier to review and prevents one demo from silently depending on the side effects of another.


## 4. Data Model and Object Relationships

The data model after import is a partially connected graph. Some nodes come from libraries. Some come from the design netlist. Some come from physical data. Some edges are complete, while others are waiting for the link stage.

A good demo should show this partial nature. Import is not the end of design understanding. It is the beginning. The link stage and later checks are required to prove that the objects belong to one coherent design context.

A practical debugging question is:

```text
Which state did this stage promise to create, and which report proves that it was created?
```

When this question cannot be answered, the flow is not yet engineered. It may still run, but it is not ready to be used as a repeatable technical asset.


## 5. Demo Design

The paired demo is:

```text
LAY-BE-09_design_import
```

The demo should be self-contained. It should use a local directory structure, local configuration, local Tcl entry point, local logs, local reports, and local output directories. It should not depend on files generated by another demo. The purpose is to make the stage understandable from the directory alone.

A recommended directory structure is:

```text
LAY-BE-09_design_import/
  README.md
  README.zh-CN.md
  config/
  scripts/
  tcl/
  data/
  logs/
  reports/
  tmp/
  output/
```

The demo should not try to be a production tapeout flow. It should be a minimal engineering microscope. It should expose one concept clearly, generate stable reports, and make the difference between success, warning, and failure visible.

The demo should produce at least three categories of evidence:

```text
1. Input evidence: what files, variables, and assumptions were used.
2. State evidence: what runtime or database state was created.
3. Review evidence: what report should be inspected before moving on.
```


## 6. What to Check in Reports

For this stage, the report checklist should include:

- Each input file is listed with pass or fail status.
- Each import action writes an independent result.
- Expected top-level module is present.
- Instance, net, pin, and port counts are reported after import when possible.
- DEF or floorplan data is checked only after required logical context exists.

The strongest reports are compact and explicit. They should be short enough to read quickly, but structured enough to support comparison across runs. A report that only says "done" is not useful. A report that names the checked assumptions and records pass or fail status is useful.

A good report also distinguishes between missing inputs, unsupported commands, empty object collections, and real design errors. These cases require different debugging actions.


## 7. Common Pitfalls

Common pitfalls include:

- Treating parser success as database success.
- Importing files in an order that leaves physical data without logical anchors.
- Not separating LEF, Liberty, Verilog, and DEF status in reports.
- Ignoring empty object counts after import.
- Failing to capture import warnings in a compact report.

The deeper issue behind these pitfalls is state ambiguity. Backend problems become expensive when the engineer cannot tell whether the failure came from the design, the library, the tool context, the command interface, or the run environment. The purpose of the demo is to reduce that ambiguity.


## 8. Engineering Takeaways

The key takeaway is that backend flow quality comes from controlled state transitions, not from long scripts. A script is only one part of the system. The real system includes configuration, runtime context, design database, library context, constraints, reports, logs, and handoff artifacts.

For this stage, the engineering discipline can be summarized as:

```text
Do not only run the command.
Define the expected state.
Generate evidence for that state.
Review the evidence before moving forward.
```

This is the difference between a command sequence and a backend flow engineering practice. The former may work once. The latter can be reviewed, repeated, compared, debugged, and reused.


## Architecture Deep Dive: What the Stage Really Owns

This stage owns **import as semantic translation from files into typed database state**. In a production backend flow, this ownership must be stated explicitly because many failures look similar at the log level but come from different architectural layers. A missing object, an empty collection, a mismatched unit, and an unsupported option can all appear as a short tool message. The engineering question is not only "what failed", but "which layer owned the assumption that failed".

The main architectural objects for this stage are:

```text
readers, parsers, object creators, name tables, unit conversion, initial physical/timing state
```

These objects should not be treated as incidental details. They form the interface contract of the stage. When a stage is executed, it consumes some of these objects, creates or refines others, and produces evidence that the next stage can trust. The more explicitly this contract is written, the easier it becomes to debug the flow when a later stage fails.

A useful architecture rule is to separate **representation**, **interpretation**, and **evidence**:

```text
representation  ->  interpretation  ->  evidence
files / params      database state       reports / logs / checkpoints
```

Representation is what the engineer writes or receives: scripts, libraries, constraints, layout views, or configuration files. Interpretation is the state created inside the EDA tool after those inputs are processed. Evidence is the external proof that interpretation happened as expected. Many backend problems occur because engineers check representation but never check interpretation. For example, a file can exist but still fail to create the required database objects; a constraint can be sourced but still not apply to the intended object set; a physical edit can be legal locally but harmful to timing or routing globally.

The architecture of a reliable flow therefore does not stop at command execution. It must also include state queries, report generation, and review gates. In this series, every demo is designed around that idea: a stage is only complete when it leaves behind enough evidence to explain what changed.

## Methodology Playbook

For this topic, the recommended methodology is:

```text
validate after each import; query object counts; isolate format errors before linking
```

This methodology can be applied at three levels.

At the **script level**, every important operation should be surrounded by clear input checks and output checks. The script should not depend on unstated shell state, invisible tool settings, or manual interpretation of long logs. It should print what it is about to do, execute the stage, query the resulting state, and write a compact report.

At the **database level**, the flow should avoid confusing names with objects. A name is only a textual handle. The real question is whether the EDA tool has created the intended cell, net, pin, port, layer, row, path, domain, or physical shape object. This distinction is especially important in hierarchical designs, ECO work, low-power implementation, and PV handoff, where names can be rewritten, flattened, uniquified, scoped, or transformed across formats.

At the **review level**, the team should define a small number of gates. A gate is not just a milestone. It is a decision point backed by reports. A useful gate says: these inputs were used, this state was created, these checks passed, these warnings remain, and this is why the next stage is safe. Without review gates, a backend flow easily becomes a long chain of commands where the first real failure is discovered too late.

A strong playbook also records negative evidence. Empty reports, missing object counts, unsupported commands, ignored constraints, and unresolved references should not be hidden. They are often the first sign that the flow is running under different assumptions from the engineer's mental model.

## Design Review Questions

Before accepting this stage as healthy, review the following question:

> What new database objects exist after each imported file?

This question is intentionally practical. It forces the stage to be judged by evidence rather than by confidence. In backend implementation, confidence without evidence is fragile. Evidence without interpretation is noise. The target is a flow where every major stage produces evidence that is compact enough to review and precise enough to drive the next engineering action.

A reviewer should also ask:

```text
1. What state did this stage promise to create?
2. Which input assumptions were required?
3. Which report proves that the state exists?
4. Which warnings are acceptable, and which must block the next step?
5. What downstream stage will fail first if this stage is wrong?
```

These five questions turn a demo into an engineering method. They also make the article useful beyond the small example, because the same reasoning can be reused in a real backend project with commercial libraries, large netlists, multiple corners, hierarchical blocks, and signoff handoff requirements.
