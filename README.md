# EDA Flow Engineering Practice

Engineering notes on reproducible EDA flows, Tcl automation, runtime environments, logging systems, command streams, and APR-oriented flow infrastructure.

## Series

### EDA Flow Engineering Practice

1. [Why the Real First Step of an EDA Tool Is Not Design Import, but a Reproducible Runtime Environment](articles/01-reproducible-runtime-environment.md)
2. [The State Space Behind an EDA Tool Session: Paths, Initialization, Logs, and Command Streams](articles/02-session-state-space.md)

## Core Ideas

- A tool run is not only a command.
- An EDA session is determined by executable path, working directory, HOME, environment variables, initialization files, command streams, logs, and temporary directories.
- Reproducible EDA flows require explicit paths, explicit initialization, explicit logs, and explicit command streams.

## Planned Topics

- Tcl as the glue language of EDA automation
- log / cmd_log / sum_log and replayable engineering sessions
- project-level initialization structure
- design import preparation
- library setup
- APR flow scripting
- timing and physical implementation flow automation

## Author

Darren H. Chen  
EDA software engineering / flow automation / verification infrastructure
