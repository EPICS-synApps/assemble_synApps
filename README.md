# assemble_synApps
synApps assemble script

Used to pull a user-defined set of EPICS modules at given tags and package them into a
single synApps directory that will build with a single 'make' command.

## Usage

The script can be called by itself and will use a default set of modules and tags to
build the synApps release. There are also a set of command line options to customize
the setup as needed. Or otherwise, the script itself is rather easy to find and replace
default values.

## Requirements (Linux)

Additional packages to install on Ubuntu 22.04:

    apt install -y git build-essential curl libusb-dev libusb-1.0-0-dev re2c x11proto-dev libx11-dev libxext-dev

## Example Setup on Linux

1. Create a base folder for synApps/EPICS setup:

        mkdir -p ~/epics

2. Get & build EPICS base

       cd ~/epics
       git clone --recursive -b 7.0 https://github.com/epics-base/epics-base.git base
       cd base && make distclean && make -j8

3. Get synApps

       cd ~/epics
       git clone https://github.com/BAMresearch/EPICS-synApps-assemble.git assemble_synApps

4. Build synApps

       cd ~/epics
       # make sure the source tree belongs to the current user, avoids errors around this
       sudo chown -R $(id -un).$(id -gn) .
       # assemble synApps which builds everything finally
       ./assemble_synApps/assemble_synApps --base="$(realpath base)" --dir="$(realpath synapps)"

## Command Line Interface

The command line options that assemble_synApps recognizes are:

* --config=
* --dir=
* --base=
* --set ABC=123
* --check
* --help

### config

The config option is used to provide a separate file with a list of modules and their tags.
This file should be arranged with each definition on their own line and in the form of
    MODULE_NAME=module_tag

Module definitions are read in and used instead of the default values listed in the
assemble_synApps script. This is a full replacement, not overwriting the values, so if a
module isn't defined in the config file provided, it will not use any default value, said
module just will not be downloaded.


### dir

The dir option is used to define the name of the synApps directory to be created. All modules
will be downloaded into the folder structure synApps_dir_name/support.


### base

The base option is used to define the location of the installation of EPICS base on your system
that synApps will link against.


### set

The set option is used to make individual changes to the module definitions you will be using.
Each usage of the `--set` option should be followed by a string in the form of
    MODULE_NAME=module_tag
    
These module definitions will either be created in the list of modules to download if they don't
already exist, or will overwrite an existing definition. This works regardless of if one is
using the default module definitions or if you have a config file.


### check

The check option is a sanity check before you fully run the script. It will display the values of
all the module definitions that it will be downloading.
