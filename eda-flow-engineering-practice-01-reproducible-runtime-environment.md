# EDA Flow Engineering Practice 01: Why the First Step of an EDA Tool Flow Is Not Design Import, but a Reproducible Runtime Environment

**Author:** Darren H. Chen  
**Series:** EDA Flow Engineering Practice  
**Topic:** EDA automation, reproducible flow, Tcl-based execution, logging, runtime context

---

## 1. Introduction

When engineers talk about an EDA implementation flow, the first step is often assumed to be design import:

```text
read netlist
read LEF / Liberty / technology files
read timing constraints
initialize design
start floorplanning, placement, CTS, routing, or timing analysis
```

This view is understandable, but it misses a more fundamental engineering problem.

In a real EDA flow, the actual first step is not importing the design. The first step is building a **reproducible runtime environment**.

Before any netlist is loaded, before any constraint file is parsed, and before any placement or routing algorithm is invoked, the tool session itself must be controlled.

Otherwise, the same script may behave differently across:

- different users,
- different directories,
- different machines,
- different shell environments,
- different GUI or batch modes,
- different startup configurations,
- different log and temporary directory settings.

In other words:

> A flow that runs once is not necessarily a reproducible flow.

---

## 2. The Hidden Problem: Runtime Context

Many EDA failures are not caused by the design database or by the algorithm itself. They are caused by uncontrolled runtime context.

Typical symptoms include:

```text
The same script works for user A but fails for user B.
The same command works in one directory but fails in another.
The flow works in GUI mode but not in batch mode.
The flow worked yesterday but fails today after the environment changed.
The log does not contain enough information to reproduce the failure.
```

At first glance, these look like tool bugs or script errors.

But very often, the root cause is that the execution context was never fully specified.

A real EDA session is affected by many inputs:

```text
Tool executable path
Tool version
Current working directory
HOME directory
Environment variables
License variables
Startup scripts
Project initialization scripts
Tcl command stream
Command-line options
Log path
Command log path
Summary log path
Temporary directory
GUI or non-GUI mode
Batch or interactive mode
```

If these inputs are implicit, the flow is fragile.

---

## 3. Why Design Import Should Not Be the First Engineering Step

Design import is important, but it assumes that the tool session is already valid.

Before importing any real design data, an engineering flow should first verify:

```text
Can the tool executable be located?
Can the tool version be checked?
Can command-line help be printed?
Can the tool run in non-GUI mode?
Can the tool execute a Tcl script?
Can the tool execute Tcl from standard input?
Can log files be generated?
Can command logs be generated?
Can summary logs be generated?
Can a dedicated temporary directory be used?
Can project initialization be explicitly loaded?
```

Only after these checks pass does it make sense to move on to design import.

Otherwise, a failure during design import may not be a design problem at all. It may be a runtime environment problem.

---

## 4. HOME Configuration vs Project Configuration

A common source of non-reproducibility is the confusion between user-level configuration and project-level configuration.

Many EDA tools support startup files under the user's HOME directory. These files are useful for personal preferences:

```text
GUI colors
Window layout
Personal aliases
Display preferences
Interactive shortcuts
```

However, project-critical settings should not depend on a user's HOME directory.

The following should be treated as project-level configuration:

```text
Library paths
Technology files
Design data locations
Run options
Algorithm switches
Output directories
Temporary directories
Flow control variables
```

A project flow that depends on a user's personal HOME setup is not portable.

A better rule is:

```text
Personal preferences belong to HOME.
Project dependencies belong to project configuration.
Production flows should explicitly source project configuration.
```

---

## 5. The Risk of Implicit Startup Files

Implicit startup files are convenient, but they are also hidden inputs.

A main flow script may look simple:

```tcl
read_netlist top.v
read_sdc top.sdc
place_opt
```

But before these commands run, the tool may have already executed startup logic such as:

```tcl
set_param some_option true
source user_init.tcl
set_app_var some_variable some_value
```

If this initialization is not visible in the main script, the flow cannot be fully understood from the script alone.

This creates several problems:

```text
The dependency is hidden.
The execution order is unclear.
The flow is harder to review.
The flow is harder to migrate.
The flow may behave differently across users.
```

For personal interactive use, implicit startup may be acceptable.

For engineering flows, explicit initialization is safer:

```tcl
source ./config/project_init.tcl
source ./config/library_setup.tcl
source ./config/run_options.tcl
```

The goal is not to write more scripts. The goal is to make dependencies visible.

---

## 6. What a Reproducible Runtime Environment Should Control

A minimal reproducible EDA runtime environment should control at least the following elements.

### 6.1 Tool Entry Point

Do not rely blindly on `PATH`.

Instead of:

```text
tool
```

prefer:

```csh
setenv EDA_TOOL_BIN /path/to/release/bin/tool
```

Then invoke:

```csh
$EDA_TOOL_BIN -version
```

This makes the tool entry point explicit.

### 6.2 Working Directory

The working directory affects relative paths, default logs, output files, and temporary data.

Use an explicit project root:

```csh
set ROOT_DIR = /path/to/project
cd "$ROOT_DIR"
```

If the tool supports it, also pass the working directory as a command-line option.

### 6.3 Project Initialization

Do not rely on automatic startup discovery.

Use explicit initialization:

```tcl
source $env(PROJECT_INIT_TCL)
```

The path can be passed from the shell:

```csh
setenv PROJECT_INIT_TCL "$ROOT_DIR/config/project_init.tcl"
```

### 6.4 Logging

A reproducible flow should capture:

```text
stdout log
main log
command log
summary log
error or crash information
```

The command log is especially important because it records the actual Tcl command sequence executed by the tool.

### 6.5 Temporary Directory

Temporary directories should be isolated per run:

```text
tmp/run_name.tmp
```

This avoids contamination from old data, concurrent runs, or failed sessions.

---

## 7. Why Command Logs Matter

Many engineers only inspect the main log.

However, in EDA flow debugging, the command log is often more important.

The main log answers:

```text
What did the tool print?
```

The command log answers:

```text
What did the tool actually execute?
```

These are not always the same.

The actual command stream may include:

```text
Commands from the main Tcl file
Commands from startup scripts
Commands from explicitly sourced files
Commands generated by scripts
Commands entered interactively
Commands triggered through GUI operations
```

Without a command log, it is difficult to replay or audit a session.

With a command log, the flow becomes more observable and easier to debug.

---

## 8. A Recommended Runtime Directory Structure

A clean runtime environment can be organized like this:

```text
eda_d01_env/
├─ config/
│  ├─ project_init.tcl
│  ├─ library_setup.tcl
│  └─ run_options.tcl
├─ generated_tcl/
│  └─ run.tcl
├─ logs/
│  ├─ run.log
│  ├─ run.cmd.log
│  ├─ run.sum.log
│  └─ run.stdout.log
├─ scripts/
│  └─ run_all.csh
├─ tmp/
│  └─ run.tmp/
└─ notes/
   └─ summary.md
```

Each directory has a clear role:

| Directory | Purpose |
|---|---|
| `config/` | Project-level initialization files |
| `generated_tcl/` | Generated Tcl command streams |
| `logs/` | Observable outputs and replay material |
| `scripts/` | Shell entry points |
| `tmp/` | Isolated temporary data |
| `notes/` | Run notes and engineering summaries |

---

## 9. A Generic csh-Based Execution Pattern

Many legacy and production EDA environments still use `csh` or `tcsh`.

A generic pattern looks like this:

```csh
#!/bin/csh -f

set nonomatch

set ROOT_DIR = /path/to/project
set LOG_DIR = "$ROOT_DIR/logs"
set TMP_DIR = "$ROOT_DIR/tmp"
set TCL_DIR = "$ROOT_DIR/generated_tcl"

mkdir -p "$LOG_DIR"
mkdir -p "$TMP_DIR"
mkdir -p "$TCL_DIR"

setenv EDA_TOOL_BIN /path/to/tool
setenv PROJECT_INIT_TCL "$ROOT_DIR/config/project_init.tcl"

cat >! "$TCL_DIR/run.tcl" << EOF_TCL
puts "RUN_BEGIN"

source \$env(PROJECT_INIT_TCL)

# Real design flow commands start here.

puts "RUN_END"
exit
EOF_TCL

$EDA_TOOL_BIN \
    -wd "$ROOT_DIR" \
    -batch "$TCL_DIR/run.tcl" \
    -session DEMO_SESSION \
    -log "$LOG_DIR/run.log" \
    -cmd_log "$LOG_DIR/run.cmd.log" \
    -sum_log "$LOG_DIR/run.sum.log" \
    -tmp_dir "$TMP_DIR/run.tmp" \
    >&! "$LOG_DIR/run.stdout.log"
```

The exact options vary across tools. The engineering pattern remains the same:

```text
Explicit tool path
Explicit working directory
Explicit initialization
Explicit command stream
Explicit logs
Explicit temporary directory
```

---

## 10. From Tool Usage to Engineering Infrastructure

Learning individual EDA commands is necessary, but not sufficient.

A real engineering flow also requires:

```text
Runtime context control
User environment isolation
Project-level initialization
Command stream recording
Log management
Failure diagnosis
Session replay
Batch execution
Portability across users and machines
```

This is the difference between using a tool and building a flow.

A tool command may solve a local task.

A reproducible flow becomes an engineering asset.

---

## 11. Conclusion

The main point of this article is simple:

> The first step of an EDA tool flow is not design import. The first step is controlling the tool session.

A mature EDA flow should make the following explicit:

```text
Tool path
Tool version
Working directory
Project initialization
Environment variables
Command stream
Logs
Temporary directory
Execution mode
```

Only after the runtime environment is stable does it make sense to import the design.

Otherwise, later failures may be caused not by the design or the algorithm, but by an uncontrolled session state.

Reproducible EDA automation starts with reproducible tool invocation.
