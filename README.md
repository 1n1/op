
# OP the operations tools

## Introduction

This tool is just a shell wrapper of wrappers, to ease systems support.

It provides internal functions to write system definitions and operations.

Those functions include change detection and rollback features.

You can list available support operations and requests, and execute them.

## Installation

We need to work on a proper installer.

By now you can run:

    wget -O op.tar.gz https://inigo.me/pub/op/op.v1.0.tar.gz
    tar -zxf op.tar.gz
    cd op

And you're done to start.

## Layout

You can put your infraestructure definitions (shell scripts), ready to be
sourced, under ```./ops/```

The same you can put your parametrized request (scripts), under ```./requests/```

Parametrized requests, get's it's parameters into the variable
```$request_params```. This may change in a future.

Of course requests may not need parameters, think of orders like: *deploy*,
*backup*, *reboot*, etc...

Each run, is logged into ```./logs/latest```. Also you can browse previous logs
by date, on ```./logs/```

You will find rollback files and steps, for each run, under ```./rolback/```

## Usage

Be carefull, op without arguments runs all the defined *ops*.

This shouldn't hurt, as those files should contain just system definitions.

You can call it as many times as you want, it should return OK.

To refresh your memory about ```op``` options, you can call ```./op -h```

  $ sudo ./op -h
  
    op v0.0.2  <https://inigo.me/projects/op>
  
    USAGE:
      op COMMAND [ARGUMENTS]
  
    COMMANDS:
      op OP  ..... Run only OP. By default all ops run. See 'op list'.
      request .... Runs a parametrized request. See 'request list'
      trial ...... Trial (do not apply any change, only show actions, combined)
      undo ... Runs the undo steps, and restores the latest modified ops
      help ....... Show this help message, and exit
  
      Sort format (-o) or long format (--op) as well as words (op) are valid.
  
    EXAMPLES
      op
      op op stack trial
      op op stack
      op undo
      op -v --request typeA param1 param2 ...
  
  $

Parametrized requests pass the parameters at the end of the CLI invocation.

For any other options and combinations, the order does not matter at all.

## Workflow

### Discovering systems

Lets say you work on a very big or very changing IT team. And you connect to
a server. You can discover what system definitions are there:

  ./op --op --list

As well as, what kind of customer requests are defined in the system:

  ./op --request --list

### Workflow of a system setting change

You edit inside (copy them, if needed) ./files/ the files that you want to
modify. For example:

  cp -a /etc/fstab files/etc/fstab
  vim files/etc/fstab    # do your changes there and save...

Next, you include the file from some existing operation definition, or from a
new one:

  vim ops/base

You could use the op function: ```file```, to ease logging, repeatability
and rollbacks:

  echo 'file -n /etc/fstab -o root -g root -m 644' >> ops/base

Then you can test your changes before apply them:

  ./op op base -v --trial

If everything looks ok, go ahead:

  ./op op base

### Workflow of a parametrized request

Let's say you get a request to perform the operation of type ```deploy```
with the parameter ```2.1.3```.

You could start, watching a simulation of the steps:

    ./op trial verbose request deploy 2.1.3

You will get a listing of the changes and commands that could run a real
operation. Including file diffs.

If everything looks ok, then you could proceed to run the operation:

    ./op -v -o deploy 2.1.3

Detailed operations and command outputs will be available to review:

    less logs/latest

If something goes wrong, you can always try the rollback steps:

    ./op undo

All the functions included with op (like ```user```, ```group```, ```file```,
```directory```, ```pkg```, etc) will setup the correspondent rollback actions.

For your custom scripts (those under ```./ops/``` or ```./requests/```) you can
use the provided functions ```rollback_path``` and ```rollback_cmd```. See the
examples below.

## Change and error notifications

If you want to do something with the log, each time there are changes, you
should enable notifications in the configuration file (```./op/op.conf```)

The default action, is to send the log and some system info, to the defined
email addresses. Notifications are disabled by default.

Nothing stops you to modify the ```notify_run``` function, to do whatever
you need to do with the information.

## Examples

You can browse (https://inigo.me/git/op-examples.git)[the op examples] to get
a feeling on howto use this tool.

You can take a look for example ops, and example requests there, and of course,
reuse them as dessired.


Happy operations!

