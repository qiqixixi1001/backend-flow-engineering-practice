# Backend Flow Engineering 01: Why the First Step Is Not Design Import, but a Reproducible Runtime Environment

> Author: Darren H. Chen  
> Direction: Backend Flow / EDA tool engineering / flow infrastructure  
> demo: **LAY-BE-01_reproducible_environment**  
> tags: Backend Flow, EDA, APR, Reproducible Environment, Tcl, Session

When engineers discuss backend implementation, the conversation usually starts with design data:

```text
Verilog netlist
SDC constraints
LEF / Liberty libraries
DEF floorplan
placement
routing
timing reports
```

That sequence is familiar, but it hides a more basic requirement. Before a backend tool can import a design in a reliable way, the runtime environment must first be controlled.

A backend flow is not executed in an abstract space. It is executed inside a concrete session, with a concrete tool binary, a concrete working directory, a concrete set of environment variables, a concrete license setup, and a concrete logging structure. If these conditions are not fixed, the same script can behave differently across users, machines, directories, or tool releases.

This article explains why a reproducible runtime environment is the first engineering object in a backend flow.

---

## 1. A backend flow starts before `import_design`

A typical first script often looks like this:

```tcl
read_verilog ./data/top.v
read_sdc     ./data/top.sdc
link_design  top
```

From a flow perspective, this looks like the beginning. From a system perspective, many decisions have already been made before these commands run:

```text
Which executable is launched?
Which version is used?
What is the current working directory?
Where is HOME?
Which init scripts are loaded?
Where are logs written?
Where are temporary files written?
How is the command stream captured?
```

If these values are implicit, the flow is not yet an engineering asset. It is only a local run.

---

## 2. The runtime environment is part of the input

A reproducible backend run depends on more than design files. It depends on environment state.

The practical input set should be understood as:

```text
DesignData
LibraryData
ConstraintData
ToolBinary
ToolVersion
RunDirectory
EnvironmentVariables
InitScripts
LogPolicy
TemporaryDirectory
ExecutionMode
```

This means the environment is not a background condition. It is part of the flow input.

If a timing report changes after moving the same script to another machine, the cause may not be the netlist or the SDC. It may be a different library path, a different tool binary, a different initialization file, or a different working directory.

---

## 3. Tool path must be explicit

Using a command such as:

```bash
tool run.tcl
```

relies on `PATH`. That is convenient for interactive work, but risky for a stable flow.

A more controlled pattern is:

```csh
setenv LAY_BACKEND_TOOL_NAME AP
setenv LAY_BACKEND_TOOL_BIN  /proj0/apoka/bin/rhel8-64/AP
```

Then every run can record:

```csh
$LAY_BACKEND_TOOL_BIN -version
```

The flow should never rely only on the shell resolving the tool name. The executable path is a high-impact state variable.

---

## 4. Working directory must be fixed

Relative paths are useful only when the working directory is known.

```tcl
source ./config/project_init.tcl
read_verilog ./data/top.v
```

These commands mean different things if the current directory changes.

A repeatable run should establish a project root first:

```csh
set LAY_PROJECT_ROOT = /path/to/project
cd "$LAY_PROJECT_ROOT"
```

or pass an explicit working directory to the tool if supported:

```csh
$LAY_BACKEND_TOOL_BIN -wd "$LAY_PROJECT_ROOT" ...
```

This makes every relative path interpretable.

---

## 5. Initialization must be visible

Backend tools often load startup files from several places:

```text
release default configuration
user HOME configuration
project-level configuration
explicit Tcl source files
GUI preference files
```

For interactive use, this is convenient. For engineering use, hidden initialization is dangerous.

A project flow should prefer explicit initialization:

```tcl
source $env(LAY_PROJECT_INIT_TCL)
```

The session should show exactly which initialization file is used and in which order it is loaded.

---

## 6. Logs are not optional

A backend run that only prints to terminal is not repeatable enough.

A minimal session should produce several types of output:

| Artifact | Purpose |
|---|---|
| stdout log | Capture terminal output |
| main log | Record tool runtime messages |
| command log | Capture the executed command stream |
| summary log | Provide a short run summary |
| report directory | Store stage reports |
| temporary directory | Isolate intermediate files |

These files are not merely debugging aids. They are the evidence that turns a run into a reviewable engineering event.

---

## 7. A minimal runtime model

A backend session can be modeled as:

```text
Session = F(
    ToolBinary,
    ToolVersion,
    WorkingDirectory,
    EnvironmentVariables,
    InitScripts,
    CommandStream,
    LogSystem,
    TempDirectory,
    ExecutionMode
)
```

Design import is only one later event in this session. The first task is to make this function controlled.

---

## 8. Suggested demo structure

For `LAY-BE-01_reproducible_environment`, the directory should focus on session startup rather than design import:

```text
LAY-BE-01_reproducible_environment/
├─ config/
│  └─ laycore_env.csh
├─ scripts/
│  ├─ run_demo01_session_runner.csh
│  └─ clean.csh
├─ tcl/
│  └─ demo01_session_runner.tcl
├─ logs/
├─ reports/
├─ tmp/
└─ README.md
```

The demo should verify:

```text
tool path is fixed
version is recorded
working directory is controlled
project initialization is explicit
logs are generated
temporary files are isolated
summary report is written
```

It should not import a real design. Its only purpose is to prove that a backend session can start in a controlled way.

---

## 9. Key takeaway

The first step in backend flow engineering is not reading a netlist.

The first step is building a reproducible runtime environment.

Only after the environment is controlled does design import become meaningful, reviewable, and repeatable.
