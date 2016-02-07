
# op - *The* operations tool


## Introduction

TL;DR: This tool is just a shell wrapper, to ease systems support.

```op``` is just a single ```/bin/sh``` script.

It expects a directory layout, with definitions inside, and declares a few
shell functions.

You can **reuse** those functions, to write system definitions and operations,
in that same directory tree. They, are designed to be **repeatable**.

The ```op``` <abbr title="Domain Specific Language">D.S.L.</abbr> functions,
include **change detection**, **logging** and **rollback** features in their
implementation. Rollback features are in alpha state.

The change detection feature, includes an optional HTTP **notification mechanism**
in place, for convenience.


## Why whould I want to use ```op```?

If you're looking for a framework, to make your systems easier to
**discover** and **manage**, in **repeatable** ways, using the shell as
in ```/bin/sh``` for the interpreter, you may want to use ```op```.

It's fast, a low dependency chain is required, the learning curve is almost
null if you know sh scripting, and you can reuse
[available recipes](https://github.com/1n1/op-examples)!.

If you're looking for a <abbr title="Domain Specific Language">DSL</abbr> to
get to **desired state** on basic system operations (permissions, file,
directory, package, etc..) ```op``` maybe your mod-friendly tool fo choice!


## Installation

You can install ```op``` anywhere in your PATH. It's just a script.

In the future there will be better ways, by now you can run:

    wget -q -O op 'https://inigo.me/pub/op/op'
    # less op
    mv op /usr/local/bin/op
    chmod 0755 /usr/local/bin/op

From this point, any system user will be able to run ```op``` and ```op doc```,
and you're done to start creating or reusing definitions.


## Usage

For a guide with examples step by step, check the
[op cookbook](https://github.com/1n1/op/blob/master/COOKBOOK.md)

To hint the usage at any time in the command line, use ```op help```:


      op - the operations tool
      
      USAGE:
        op COMMAND [ARGUMENTS]
      
      COMMANDS:
        play NAME ............. Play only NAME. See 'play list'.
        request NAME [ARGS] ... Runs a parametrized request. See 'request list'
        trial ................. Trial mode (do not apply any change)
        gen PATH [--perms] .... Print file rules for given path, or perms with -p
        import PATH ........... Imports a given path, into the op tree
        undo .................. Rollback steps, and restore the modified files
        doc FILE [topics|TOPIC] Show detailed documentation, and exit
        help .................. Show this help message, and exit
      
      Sort format (-p) or long format (--play) as well as words (play) are valid.
       
      EXAMPLES
        op --play --list
        op play stack-network trial verbose
        op -p stack-network
        op undo
        op -v --request typeA param1 param2
        op import /etc/hosts
        op --gen /etc/network --perms
        op --doc op function log

## Reference manual

See the [op manual](https://github.com/1n1/op/blob/master/MANUAL.md) online,
for a complete reference.

You can access the same information at any time, from the command line, using
the ```--doc``` switch:

     op --doc
     op --doc op topics
     op --doc op function log

The ```--doc``` switch also work with your definitions, so you can generate
markdown from comments on code:

     op doc play/mystack | markdown

## Contributing

See the
[contributing guide](https://github.com/1n1/op/blob/master/CONTRIBUTING.md).


## License

See the MIT style [LICENSE](https://github.com/1n1/op/blob/master/LICENSE),
included with the source repository.

