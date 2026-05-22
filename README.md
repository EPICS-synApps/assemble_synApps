# assemble_synApps

synApps assemble script

Used to pull a user-defined set of EPICS modules at given tags and package them into a
single synApps directory that will build with a single `make` command.


## Prerequisites

- **Perl** 5.12 or later
- **Git**
- **curl** (for downloading AllenBradley and ULDAQ tarballs)
- **make** and a **C/C++ compiler** (for building ULDAQ on Linux)
- A **built EPICS base** installation


## Quick Start

```bash
# Clone the assemble script
git clone https://github.com/EPICS-synApps/assemble_synApps.git
cd assemble_synApps

# Assemble synApps using the default module set
./assemble_synApps --base=/path/to/epics/base

# Build synApps
cd git/support
make -j4
```


## Command Line Options

```
./assemble_synApps [options]
```

| Option | Description |
|--------|-------------|
| `--base=<path>` | Path to a built EPICS base installation |
| `--config=<file>` | Use an external config file instead of the default module list |
| `--dir=<name>` | Name of the output directory (default: `git`) |
| `--set MODULE=TAG` | Override a single module's tag (repeatable) |
| `--update` | Skip modules already at the correct tag |
| `--check` | Display module list without downloading |
| `--quiet` | Suppress progress messages and external tool output |
| `--verbose` | Show full output from git and other external commands |
| `--help` | Print usage information |


### base

The base option defines the location of the EPICS base installation that synApps will
link against. The path must point to a fully built EPICS base directory (containing
`include/epicsVersion.h`).

```bash
./assemble_synApps --base=/APSshare/epics/base-7.0.8
```


### config

The config option provides a separate file with a list of modules and their tags. The
file should have one definition per line in the form `MODULE_NAME=module_tag`.

```
# myconfig.txt
ASYN=R4-44-2
AUTOSAVE=R5-11
BUSY=R1-7-4
CALC=R3-7-5
SSCAN=R2-11-6
DEVIOCSTATS=3.1.16
SNCSEQ=R2-2-9
STD=R3-6-4
STREAM=2.8.24
XXX=R6-3
```

```bash
./assemble_synApps --base=/path/to/base --config=myconfig.txt
```

Config file definitions **completely replace** the default module list. If a module is
not listed in the config file, it will not be downloaded. Blank lines and lines starting
with `#` are ignored. Leading and trailing whitespace around keys and values is trimmed.


### dir

The dir option defines the name of the synApps directory to be created. All modules will
be downloaded into `<dir>/support/`.

```bash
./assemble_synApps --base=/path/to/base --dir=synApps_R6-3
# Creates synApps_R6-3/support/
```

The default directory name is `git`.


### set

The set option makes individual changes to the module definitions. Each usage should be
followed by a string in the form `MODULE_NAME=module_tag`. These definitions will either
create a new entry or overwrite an existing one. This works with both the default module
list and a config file.

```bash
# Override specific module versions
./assemble_synApps --base=/path/to/base \
    --set ASYN=R4-45 \
    --set MOTOR=R7-3-1

# Remove a module by setting its tag to an empty string
./assemble_synApps --base=/path/to/base --set GALIL= --set DXP=

# Add extra areaDetector submodules (space-separated list)
./assemble_synApps --base=/path/to/base \
    --set "AREA_DETECTOR_SUBMODULES=ADProsilica ADPointGrey ADPilatus"
```


### update

The update option skips modules that are already cloned at the correct tag. This is
useful for two scenarios:

**Resuming after a failure**: If the script fails partway through (e.g., a network
error during cloning), re-running with `--update` will skip the modules that were
already successfully cloned and pick up where it left off.

```bash
# Initial run fails partway through
./assemble_synApps --base=/path/to/base
# ... network error on module 20 of 40 ...

# Resume -- skips the first 19 modules
./assemble_synApps --base=/path/to/base --update
```

**Adding modules to an existing deployment**: If you have a working synApps and want
to add a new module or change one module's version, `--update` avoids re-cloning and
re-patching all the unchanged modules.

```bash
# Add a new module to an existing synApps
./assemble_synApps --base=/path/to/base --set OPCUA=v0.9.3 --update

# Change one module's version
./assemble_synApps --base=/path/to/base --set ASYN=R4-45 --update
```

When `--update` is set:

- Modules already at the requested tag are skipped entirely (no `git clean`, no
  re-patching).
- Modules at a different tag are updated to the new tag and re-patched.
- New modules not yet cloned are cloned and patched normally.
- The `configure/RELEASE` file is always regenerated to ensure consistency.
- AllenBradley is skipped if its directory already exists.
- The ULDAQ library build (inside measComp) is skipped if already built.

Without `--update`, the script restores every existing module to a clean state
(`git stash`, `git clean -fdx`, `git checkout`) and re-applies all patches. This
is the default behavior and ensures a fully clean assembly.


### check

The check option is a dry run. It displays the module definitions that would be used
without downloading anything.

```bash
./assemble_synApps --base=/path/to/base --check
```


### quiet / verbose

These options control the amount of output produced by the script.

- `--quiet` suppresses progress messages ("Grabbing X at tag Y") and redirects
  stdout from external tools (git, curl, tar, make) to `/dev/null`. Errors and
  warnings are always shown on stderr.

- `--verbose` shows full output from git commands (removes the default `-q` flag).
  All progress messages are shown.

These flags are mutually exclusive.

```bash
# Silent operation -- only errors/warnings shown
./assemble_synApps --base=/path/to/base --quiet

# Full git output for debugging
./assemble_synApps --base=/path/to/base --verbose
```


## Re-running the Script

The script can be safely re-run against an existing synApps directory:

- The `support/` repository is not re-cloned if it already exists.
- Without `--update`, each module is restored to a clean state and all patches are
  re-applied. This wipes any local changes or build artifacts in module directories.
- With `--update`, only new or changed modules are touched. Existing modules at the
  correct tag are left as-is, preserving build artifacts.
- The `configure/RELEASE` file is always regenerated from scratch.
- `make release` is run at the end to propagate paths to all modules.


## Examples

```bash
# Basic assembly with default modules
./assemble_synApps --base=/usr/local/epics/base-7.0.8

# Custom directory name
./assemble_synApps --base=/path/to/base --dir=synApps_R6-3

# Minimal synApps using a config file
./assemble_synApps --base=/path/to/base --config=minimal.txt

# Remove modules you don't need
./assemble_synApps --base=/path/to/base --set GALIL= --set DXP= --set XSPRESS3=

# Preview what would be assembled
./assemble_synApps --base=/path/to/base --set ASYN=R4-45 --check

# Quiet assembly for CI/automated builds
./assemble_synApps --base=/path/to/base --quiet

# Resume a failed assembly
./assemble_synApps --base=/path/to/base --update

# Add a module to an existing deployment
./assemble_synApps --base=/path/to/base --set OPCUA=v0.9.3 --update
```
