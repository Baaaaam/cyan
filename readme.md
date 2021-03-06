
# CyAn - Cyclus Analysis Tools

CyAn contains command line tools for post processing and analyzing Cyclus
simulation databases (http://fuelcycle.org). It is still experimental and not
all features have been validated/verified to be correct.

## Installation

Binary builds of CyAn are provided for Windows, Linux, and Mac platforms (both
32 and 64 bit) here: https://github.com/rwcarlsen/cyan/releases.  These
binaries have no external dependencies and can just be executed directly.

To build and install from source, you need a Go1.5+ toolchain.  You can get it
from http://golang.org/doc/install or you can use your favorite package
manager (if their Go package is recent enough):

```bash
# debian-based distros
apt-get install golang

# archlinux
pacman -S go

# mac OSX with macports
port install go
```

If You are using Go1.5.x, you will need to set the environment variable
``GO15VENDOREXPERIMENT=1``. You should also make a directory to use as your
GOPATH and set the GOPATH environment variable to it.  The Go tool will
install packages into this directory.  For convenience, you should also add
`$GOPATH/bin` to your PATH so that binaries from fetched packages are directly
accessible on the command line.  When you are ready, run:

```bash
go get github.com/rwcarlsen/cyan/cmd/cyan
```

## Usage

There primary command line tool is `cyan`.  The commands has various flags and
subcommands that can be viewed with the `-h` flag:

```
Usage: cyan [-db <cyclus-db>] [flags...] <subcommand> [flags...] [args...]
Computes metrics for cyclus simulation data in a sqlite database.

Options:
  -custom string
    	path to custom sql query spec file
  -db string
    	cyclus sqlite database to query
  -query
    	show query SQL for a subcommand instead of executing it
  -simid string
    	simulation id in hex (empty string defaults to first sim id in database

Sub-commands:

  [General]
    sims     list all simulations in the database
    infile   show the simulation's input file
    version  show simulation's cyclus version info
    post     post process the database
    table    show the contents of a specific table
    ts       investigate time-series data tables

  [Agents]
    agents    list all agents in the simulation
    protos    list all prototypes in the simulation
    deployed  time series total active deployments by prototype
    built     time series of new builds by prototype
    decom     time series of a decommissionings by prototype
    ages      list ages of agents at a particular time step

  [Flow]
    commods    show commodity transaction counts and quantities
    flow       time series of material transacted between agents
    flowgraph  generate a graphviz dot script of flows between agents
    trans      time series of transaction quantity over time

  [Other]
    inv      time series of inventory by prototype
    power    time series of power produced
    energy   thermal energy (J) generated between 2 timesteps
    created  material created by agents between 2 timesteps
```

Subcommands each take their own arguments and have their own help/ussage
messages.  Global arguments must be given *before* the subcommand name (e.g.
``-db``, ``-query``, etc.).  And subcommand arguments must be provided
*before* any positional arguments.  Some quick examples:

```bash
# show help/usage for the "deployed" subcommand
cyan deployed -h
# Output:
# Usage: deployed <prototype>
# Time series of a prototype's total active deployments
#   -p	plot the dat

# show a list of all agents/facilities that ever existed in the simulation
cyan -db cyclus.sqlite agents

# output a time series of active deployments for all AP1000 facilities
cyan -db cyclus.sqlite deployed AP1000

# plot a active deployments for all AP1000 facilities using gnuplot
cyan -db cyclus.sqlite deployed -p AP1000

# print the SQL query cyan uses to generate "deployed" subcommand results
cyan -db cyclus.sqlite -query deployed AP1000

# list all tables in the database
cyan -db cyclus.sqlite table

# print the content of a particular table
cyan -db cyclus.sqlite table myCustomTableName

# just post process the db (this is done automatically by other commands too)
cyan -db cyclus.sqlite post

# output a png graph of the flow of all material between agents t=2 to t=7
cyan -db cyclus.sqlite flowgraph -t1=2 -t2=7 > flow.dot
dot -Tpng -o flow.png flow.dot
```

## Cross Compilation

To cross-compile for all major architectures/OS's supported by Go, you can use
`xgo` (https://github.com/karalabe/xgo) - for example:

```
xgo -targets linux/amd64,windows/386,windows/amd64 github.com/rwcarlsen/cyan/cmd/cyan
```

And you will get 64 bit linux, 32 bit windows and 64 bit windows binaries.

