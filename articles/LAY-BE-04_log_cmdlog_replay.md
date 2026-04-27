# Backend Flow Engineering 04: From log to cmd_log: How to Preserve a Replayable Engineering Session

> Author: Darren H. Chen  
> Direction: Backend Flow / EDA tool engineering / flow infrastructure  
> demo: **LAY-BE-04_log_cmdlog_replay**  
> tags: Backend Flow, EDA, Logs, cmd_log, Replay, Tcl, Debug

Many backend issues are not caused by failure to produce a result. They are caused by failure to reconstruct how the result was produced.

Typical cases are familiar:

```text
a run worked yesterday but fails today
an interactive command fixed the issue but nobody remembers it
a report exists but the command path is unclear
a crash happened and the last valid state cannot be rebuilt
```

The core question is:

```text
Can one backend session be preserved as a replayable engineering record?
```

This article discusses the roles of `log`, `cmd.log`, summary logs, reports, `source`, and debug records.

---

## 1. A successful run is not enough

A backend run that completes once is useful. But engineering work requires more:

```text
Can it be repeated?
Can another engineer inspect it?
Can errors be localized?
Can two runs be compared?
Can the exact command sequence be reconstructed?
```

This is why logging is part of flow infrastructure, not an afterthought.

---

## 2. Different logs answer different questions

A mature backend session should not rely on a single giant output file.

Different records serve different purposes:

| Artifact | Main question answered |
|---|---|
| main log | What happened during execution? |
| stdout log | What did the terminal show? |
| cmd.log | Which Tcl commands were executed? |
| summary log | What is the high-level result? |
| reports | What were the stage outputs? |
| error trace | Where did failure occur? |

Combining these artifacts creates a reviewable engineering record.

---

## 3. The main log records process history

The main log usually includes:

```text
startup messages
version information
warnings and errors
stage messages
runtime statistics
report fragments
command echoes
```

It is best suited for understanding sequence and failure location.

When a flow fails, the first question is often:

```text
Where did the first serious error appear?
```

The main log should make that visible.

---

## 4. The command log records replay history

The command log has a different purpose.

It records the command stream.

This matters because the written script and the executed commands are not always identical. Commands may come from:

```text
startup files
main Tcl scripts
sourced sub-scripts
interactive commands
stdin
generated Tcl fragments
```

A `cmd.log` helps answer:

```text
What did the tool actually execute?
```

This is the foundation of replay.

---

## 5. Summary logs reduce review cost

Large logs are necessary, but they are not convenient for first-level review.

A summary log should answer quickly:

```text
run name
tool version
start / end time
exit status
stage list
major warnings
major errors
generated reports
output directory
```

A good summary file is not the most detailed artifact. It is the fastest artifact for triage.

---

## 6. `source` is a session entry point

In Tcl, `source` does more than read a file.

It injects a batch of commands into the current tool session:

```tcl
source ./scripts/floorplan.tcl
```

This may change:

```text
database state
current design
parameters
placement
routing
reports
log state
```

Therefore, each `source` operation should be visible in logs and protected by error handling.

---

## 7. Debug records matter

Complex Tcl scripts often include nested evaluation:

```tcl
set target_cells [get_cells -filter "ref_name =~ *BUF*"]
report_property $target_cells
```

If only the outer command is visible, debugging is harder.

A debug mode should help reveal:

```text
which branch executed
which object query returned what
which file path was computed
which variable value was used
which error stack was produced
```

In Tcl, `$::errorInfo` is especially important after failures.

---

## 8. Replay is not just sourcing cmd.log

It is tempting to say:

```text
If cmd.log exists, source it again.
```

In practice, replay also needs:

```text
the same tool version
the same run directory
the same initialization order
the same input files
the same environment variables
the same output policy
the same stage boundaries
```

A command log is necessary, but not sufficient by itself.

Replay is a session reconstruction problem.

---

## 9. Suggested demo structure

`LAY-BE-04_log_cmdlog_replay` should focus on record generation:

```text
LAY-BE-04_log_cmdlog_replay/
├─ scripts/
│  └─ run_demo04_log_replay.csh
├─ tcl/
│  ├─ demo04_main.tcl
│  └─ demo04_stage.tcl
├─ logs/
│  ├─ demo04.log
│  ├─ demo04.cmd.log
│  ├─ demo04.stdout.log
│  └─ demo04.sum.log
├─ reports/
│  └─ demo04_summary.rpt
└─ README.md
```

The demo should prove that one controlled session produces all required records.

---

## 10. Key takeaway

Logs are not passive text files. They are engineering evidence.

A backend session becomes maintainable when it leaves:

```text
process record
command record
summary record
error record
report record
```

Together, these artifacts turn a temporary run into a replayable engineering session.
