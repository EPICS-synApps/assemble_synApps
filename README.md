# assemble_synApps
synApps assemble script

Used to pull a user-defined set of EPICS modules at given tags and package them into a
single synApps directory that will build with a single 'make' command.

## usage

The script can be called by itself and will use a default set of modules and tags to
build the synApps release. There are also a set of command line options to customize
the setup as needed. Or otherwise, the script itself is rather easy to find and replace
default values.

### Command Line Interface

The command line options that assemble_synApps recognizes are:

* --config=
* --dir=
* --base=
* --set ABC=123
* --check
* --help

#### config

The config option is used to provide a separate file with a list of modules and their tags.
This file should be arranged with each definition on their own line and in the form of
    MODULE_NAME=module_tag

Module definitions are read in and used instead of the default values listed in the
assemble_synApps script. This is a full replacement, not overwriting the values, so if a
module isn't defined in the config file provided, it will not use any default value, said
module just will not be downloaded.


#### dir

The dir option is used to define the name of the synApps directory to be created. All modules
will be downloaded into the folder structure synApps_dir_name/support.


#### base

The base option is used to define the location of the installation of EPICS base on your system
that synApps will link against.


#### set

The set option is used to make individual changes to the module definitions you will be using.
Each usage of the `--set` option should be followed by a string in the form of
    MODULE_NAME=module_tag
    
These module definitions will either be created in the list of modules to download if they don't
already exist, or will overwrite an existing definition. This works regardless of if one is
using the default module definitions or if you have a config file.


#### check

The check option is a sanity check before you fully run the script. It will display the values of
all the module definitions that it will be downloading.
