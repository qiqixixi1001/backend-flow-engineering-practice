# Backend Flow Engineering 05: Why Backend Tcl Scripts Must Be Checked Before Execution

> Author: Darren H. Chen  
> Direction: Backend Flow / EDA tool engineering / flow infrastructure  
> demo: **LAY-BE-05_tcl_script_precheck**  
> tags: Backend Flow, EDA, Tcl, Precheck, Error Handling, Flow Engineering

A backend Tcl script is not just a text file. It is often a state-changing entry point into a live design database.

Running:

```tcl
source run.tcl
```

may import a design, load libraries, change the current design, initialize floorplan data, modify placement, generate routes, or overwrite reports.

This is why serious backend Tcl scripts should be checked before execution.

---

## 1. Tcl scripts modify session state

A backend script may execute commands such as:

```tcl
import_verilog ./data/top.v
link_project top
init_floorplan
place_optimize
route
report_timing
export_def ./out/top.def
```

These commands are not harmless text operations. They can change:

```text
design database
library context
current design
floorplan state
placement state
routing state
timing scenario
output files
temporary files
```

If the script fails halfway, the session may be partially modified.

---

## 2. Runtime errors can appear late

Many flow errors do not appear at the first line.

A script may run successfully for many commands and fail later:

```text
library setup succeeds
design import succeeds
link partially succeeds
floorplan starts
one parameter fails
session is now half-changed
```

After that, it may be unclear whether the session can continue safely.

This is why precheck is valuable.

---

## 3. Script checking has several layers

Checking a backend Tcl script is not only syntax validation.

A useful precheck covers:

| Layer | Example questions |
|---|---|
| syntax | Are braces, quotes, and blocks valid? |
| command | Are the required commands available? |
| parameter | Are command options supported? |
| path | Do referenced files exist? |
| environment | Are required variables set? |
| context | Is the session in the right state? |
| side effect | Could the script overwrite or modify important state? |

The most backend-specific layers are context and side effects.

---

## 4. Context checking is essential

Some commands exist, but only make sense when the session is ready.

For example:

```tcl
get_cells *
```

requires a design database.

```tcl
report_timing
```

requires linked design data, timing libraries, and constraints.

Therefore, a script should not only ask:

```text
Does the command exist?
```

It should also ask:

```text
Is the current session ready for this command?
```

---

## 5. File and environment precheck

A basic precheck layer can be implemented with Tcl procedures:

```tcl
proc require_env {name} {
    if {![info exists ::env($name)]} {
        error "Missing required environment variable: $name"
    }
}

proc require_file {path} {
    if {![file exists $path]} {
        error "Required file does not exist: $path"
    }
}

proc require_dir {path} {
    if {![file isdirectory $path]} {
        error "Required directory does not exist: $path"
    }
}
```

Then the main script can check:

```tcl
require_env LAY_PROJECT_ROOT
require_dir  $env(LAY_PROJECT_ROOT)
require_file $env(LAY_PROJECT_ROOT)/config/project_init.tcl
```

These checks prevent many avoidable failures.

---

## 6. Stage boundaries and `catch`

Each major stage should have an error boundary:

```tcl
proc run_stage {stage_name script_path policy} {
    puts "STAGE_BEGIN: $stage_name"

    if {![file exists $script_path]} {
        error "Stage script not found: $script_path"
    }

    set rc [catch {
        source $script_path
    } errMsg]

    if {$rc != 0} {
        puts "STAGE_ERROR: $stage_name"
        puts "ERROR: $errMsg"
        puts "ERRORINFO: $::errorInfo"

        if {$policy eq "fail-fast"} {
            error "Stage failed: $stage_name"
        } else {
            puts "WARNING: continuing after stage failure"
        }
    }

    puts "STAGE_END: $stage_name"
}
```

This makes failure behavior explicit.

---

## 7. Fail-fast vs continue-on-error

Not every failure has the same severity.

| Policy | Suitable for |
|---|---|
| fail-fast | library loading, design import, link, floorplan, routing, export |
| continue-on-error | optional reports, debug dumps, non-critical summaries |

A flow should not blindly continue after critical state-changing stages fail.

---

## 8. Precheck reports

Precheck results should be written to files, not only printed.

Example output:

```text
[PASS] LAY_PROJECT_ROOT is set
[PASS] log directory is writable
[PASS] project_init.tcl exists
[FAIL] tech.lef is missing
[BLOCK] design import should not continue
```

A dedicated report makes failure review faster than searching a large log.

---

## 9. Demo focus

`LAY-BE-05_tcl_script_precheck` should demonstrate:

```text
required environment checks
required file checks
required directory checks
stage-level source handling
catch-based error reporting
fail-fast and continue-on-error policy
precheck report generation
```

The goal is not to run a full design. The goal is to show a safe script execution structure.

---

## 10. Key takeaway

A backend Tcl script should not be treated as a casual script.

It is a state-changing procedure against a live tool session.

A mature flow checks before execution, controls errors during execution, and records results after execution.
