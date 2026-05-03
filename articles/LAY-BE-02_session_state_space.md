# 02. EDA Tool Startup as a Session State Space

> Author: Darren H. Chen  
> Direction: Backend Flow / Physical Implementation / EDA Tool Engineering / Tcl-Based Flow Engineering  
> demo: **LAY-BE-02_session_state_space**  
> Tags: Backend Flow, EDA, Session State, Runtime Context, Environment Variables, Flow Engineering

A backend session is not a simple process launch. It is a state space. The visible command line is only one coordinate. Other coordinates include the working directory, startup files, loaded libraries, loaded design data, scenario settings, current design context, selected objects, tool parameters, report paths, and previous commands.

When two engineers run the same script and get different results, the root cause is often hidden in this state space. The script text is identical, but the session state is not. A mature backend flow therefore needs a way to describe, snapshot, and compare session state before the design stage becomes complex.

A backend flow is not only a command sequence. It is a state transition system. Each stage starts from an input state, applies a controlled operation, and leaves an output state for the next stage. The quality of the whole flow depends on whether these state transitions are explicit, observable, and reviewable.

## 1. Conceptual Model

A session state space is the complete set of variables and internal tool states that can influence command behavior. Some states are external, such as shell variables and file paths. Some are internal, such as loaded libraries, active design context, current scenario, tool parameters, and selected object collections. A command does not run in isolation; it runs inside this state.

This model explains why a minimal startup demo is valuable. It does not need to optimize placement or route wires. It needs to reveal whether the session can be observed. If the state cannot be observed at startup, it will be much harder to debug after design import, placement, clock tree, routing, or signoff handoff.

A useful way to reason about the stage is:

```text
inputs + assumptions
        |
        v
controlled EDA tool operation
        |
        v
observable state + reports + next-stage readiness
```

This model prevents a common mistake: judging a backend stage only by whether a command returned without a fatal error. A clean return is useful, but it is not sufficient. The stage must also leave enough evidence to prove that the expected state was created.

## 2. Backend Architecture View

The session architecture can be viewed as concentric layers. The outer layer is the operating system environment. Inside it sits the tool process. Inside the tool process sits the Tcl interpreter. Inside the Tcl interpreter sits the backend database context. Inside that context sit design objects, libraries, constraints, reports, and selected objects.

A state snapshot should not attempt to dump everything. It should capture stable evidence: executable, version probe, working directory, startup source list, important environment variables, known output paths, loaded state indicators, and command availability. These are enough to explain most early-stage run differences.

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

The data model for session state can be represented as a key-value graph. `PATH`, license variables, home directory, tool path, project root, report directory, and temporary directory are external keys. Loaded library count, design context, current scenario, and key command availability are internal keys.

The value of this model is comparison. A session state report generated on Monday and another generated on Tuesday can be compared line by line. If a failure appears only on Tuesday, the state difference becomes the first debugging target.

A practical debugging question is:

```text
Which state did this stage promise to create, and which report proves that it was created?
```

When this question cannot be answered, the flow is not yet engineered. It may still run, but it is not ready to be used as a repeatable technical asset.

## 5. Demo Design

The paired demo is:

```text
LAY-BE-02_session_state_space
```

The demo should be self-contained. It should use a local directory structure, local configuration, local Tcl entry point, local logs, local reports, and local output directories. It should not depend on files generated by another demo. The purpose is to make the stage understandable from the directory alone.

A recommended directory structure is:

```text
LAY-BE-02_session_state_space/
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

- Important shell variables are captured.
- Output directories are fixed before design commands run.
- Key command families are probed.
- No design context is assumed before import and link.
- The state report is stable enough to compare across machines.

The strongest reports are compact and explicit. They should be short enough to read quickly, but structured enough to support comparison across runs. A report that only says "done" is not useful. A report that names the checked assumptions and records pass or fail status is useful.

A good report also distinguishes between missing inputs, unsupported commands, empty object collections, and real design errors. These cases require different debugging actions.

## 7. Common Pitfalls

Common pitfalls include:

- Assuming a tool session starts from an empty state.
- Changing parameters in one stage and forgetting their effect on later stages.
- Mixing graphical exploration with scripted batch runs without recording state.
- Treating a command failure as a syntax issue when the real issue is missing context.
- Not recording license-related environment variables when capacity behavior changes.

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

This stage owns **session state as the hidden input to every later stage**. In a production backend flow, this ownership must be stated explicitly because many failures look similar at the log level but come from different architectural layers. A missing object, an empty collection, a mismatched unit, and an unsupported option can all appear as a short tool message. The engineering question is not only "what failed", but "which layer owned the assumption that failed".

The main architectural objects for this stage are:

```text
environment variables, initialization files, license context, path resolution, working directory, history, run metadata
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
model startup as a state vector; compare state across machines; detect personal settings before they change project behavior
```

This methodology can be applied at three levels.

At the **script level**, every important operation should be surrounded by clear input checks and output checks. The script should not depend on unstated shell state, invisible tool settings, or manual interpretation of long logs. It should print what it is about to do, execute the stage, query the resulting state, and write a compact report.

At the **database level**, the flow should avoid confusing names with objects. A name is only a textual handle. The real question is whether the EDA tool has created the intended cell, net, pin, port, layer, row, path, domain, or physical shape object. This distinction is especially important in hierarchical designs, ECO work, low-power implementation, and PV handoff, where names can be rewritten, flattened, uniquified, scoped, or transformed across formats.

At the **review level**, the team should define a small number of gates. A gate is not just a milestone. It is a decision point backed by reports. A useful gate says: these inputs were used, this state was created, these checks passed, these warnings remain, and this is why the next stage is safe. Without review gates, a backend flow easily becomes a long chain of commands where the first real failure is discovered too late.

A strong playbook also records negative evidence. Empty reports, missing object counts, unsupported commands, ignored constraints, and unresolved references should not be hidden. They are often the first sign that the flow is running under different assumptions from the engineer's mental model.

## Design Review Questions

Before accepting this stage as healthy, review the following question:

> Can we compare two sessions and explain why they are equivalent or different?

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
