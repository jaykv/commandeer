[![GitHub Workflow Status (with branch)](https://img.shields.io/github/actions/workflow/status/jaykv/cliffy/python-app.yaml?branch=main)](https://github.com/jaykv/cliffy/actions)
[![PyPI](https://img.shields.io/pypi/v/cliffy)](https://pypi.org/project/cliffy/)
![GitHub](https://img.shields.io/github/license/jaykv/cliffy)

# cliffy :mountain:
YAML-defined CLI generator and manager for python

## Features
* Build and generate YAML-defined CLIs
* Dynamic abstractions to rapidly build and test CLIs
* Manage CLIs- load, list, update, and remove
* Built-in shell and Python script support
* Supports Jinja2-templating to create a flexible command flow

## Install
`pip install cliffy`

## Usage
`cli <command>`
* `init <cli name>`: Generate a template CLI manifest
* `load <manifest>`: Add a new CLI based on the manifest
* `render <manifest>`: Render the YAML manifest into executable code
* `list` or `ls`: Ouput a list of loaded CLIs 
* `update <cli name>`: Reloads a CLI
* `disable <cli name>`: Disable a CLI
* `enable <cli name>`: Enable a disabled CLI
* `remove <cli name>` or `rm <cli name>`: Remove a loaded CLI

### Basic Example

1. Define a manifest
```yaml
# hello.yaml
name: hello
version: 0.1.0

commands:
  bash: $echo "hello from bash"
  python: print("hello from python")
```

2. Load CLI
```
$ cli load hello.yaml
```

3. Run CLI directly

`hello -h`

![hello-demo](docs/images/hello.png)

For more examples, check [examples](examples/) directory.

## Manifest template
Generated by `cli init`:
```yaml
# cliffy v1 template
manifestVersion: v1

# The name of the CLI
# This will be used as the script name when invoking the CLI from the command line.
name: cliffy 

# The version of the CLI
# This should follow the standard semantic versioning format (e.g., 'MAJOR.MINOR.PATCH').
version: 0.1.0

# List of external CLI manifest paths to include into the main manifest
# Performs a deep merge of manifests sequentially in the order given to assemble a merged manifest
# and finally, deep merges the merged manifest with the main manifest.
includes: []

# List of Python dependencies required for the CLI
# Validated on CLI load and update
# Supports basic requirements specifier syntax.
requires: []

# A mapping defining manifest variables that can be referenced in any other blocks
# Environments variables can be used in this section with ${some_env_var} for dynamic parsing
# Supports jinja2 formatted expressions as values
# Interpolate defined vars in other blocks jinja2-styled {{ var_name }}.
vars:
    default_mood: happy

# A string block or list of strings containing any module imports
# These can be used to import any python modules that the CLI depends on.
imports:
    - import os
    - |
        from collections import defaultdict
        import re

# A list containing any helper functions
# Each element of the list can be a separate function
# These functions should be defined as strings that can be executed by the Python interpreter.
functions:
    - |
        def greet_name(name: str):
            print("hello " + name)

# A mapping containing any shared type definitions
# These types can be referenced by name in the args section to provide type annotations for params and options defined in the args section.
types:
    Language: str = typer.Option("english", "-l", help="Language to greet in", prompt=True)

# A mapping containing the arguments and options for each command
# Each key in the mapping should correspond to a command in the commands section
# The value should be a list of mappings representing the params and options for that command.
args:
    world: [--name|-n: str!]                      # a REQUIRED option
    greet.all: 
        - names: str!                             # a REQUIRED param as denoted by the ! at the end
        - mood: str = "{{default_mood}}"          # an OPTIONAL param that uses a manifest var as default
        - --language: Language                    # an option with a default that uses Language type as arg definition

# A mapping containing the command definitions for the CLI
# Each command should have a unique key- which can be either a group command or nested subcommands
# Nested subcommands are joined by '.' in between each level
# A special (*) wildcard can be used to spread the subcommand to all group-level commands
# The value is the python code to run when the command is called OR a list of bash commands to run (prefixed with $).
commands:
    # this is a parent command that will get invoked with: hello world
    world: 
        - |
            """
            Help text for list
            """
            greet_name("world")
        - $ echo "i can also mix-and-match this command script to run bash commands"
    
    # this is a nested command that will get invoked with: hello greet all
    greet.all: 
        - help: Help text for list.all       # you can also define help text like this
        - $ echo "hello all"                 # this is a bash command that will get converted to python subprocess call
        - print("greetings from python")     # this python code will get directly invoked

```
For a barebones template: `cli init --raw`

## Development
```
poetry shell
cli -h
```
