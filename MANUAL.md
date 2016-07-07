
#  op - *The* operations tool

This program, helps to make systems definitions **reproducible** and to
help on system's maintenance and discovery.

##  Initialization

When you call ```op``` without arguments, it sources all definitions stored
in a relative subdirectory from the invocation dir, under ```./play/*```.

Those sourced operations, can reuse the ```op``` variables and functions.

There are switches to call specific ```./play/``` files (```-p```) and
specific requests under ```./requests/``` with (```-r```). 

By default we're quiet, with the program output. Use ```-v``` to see verbose
messages.

Use ```--trial``` on the command line, for test without applying commands.

The modes ```-e``` and ```-u```, are set at initialization, recipes will be
affected by such behavior, as they are sourced.

##  Program variables

The program initializes the following variables, that you can latter
reuse in your recipes.

* ```version```: The program version.
* ```op```: contains the executable name, used on messages, etc.
* ```work_dir```: The directory where the program was invocated.
* ```files```: The main directory for files used in recipes.
* ```rollback_dir```: The rollback files main directory.
* ```lock```: The file used to block against parallel sessions.
* ```verbose```: The verbosity switch (bool, default disabled, 0).
* ```trial```: The trial switch (bool, default disabled, 0).
* ```nocolor```: The colors switch (bool, default disabled, 0).
* ```hostname```: It contains the hostname at our launch time.
* ```pkgtool```: Package system (apt, yum).


##  Color variables

The following foreground color variables, are initialized for reusing:

    normal, red, green, yellow, blue, magenta, cyan, white
###  Logging variables

Each run gets a uniq identifier, stored in the variable: ```run_id```

The variables ```logs``` and ```logfile``` will point to our current
log during execution. For internal use.

###  Notifications variables


Notifications are controlled by the boolean: ```notify_enabled```

Set to 1 to notify run results, set to 0 to disable notifications.

HTTP notifications are configured using:

```http_url```: URL to use for the request
```http_headers```: Extra headers in curl format
```http_options```: All of the previous together

Use ```http_options``` to tune any extra curl option you may need.

##  General purpose functions

Functions for output, messages and general purpose

###  function now

Prints a timestamp intended for messages.

No arguments.

    USAGE
      now

###  function log

Dispatchs a message to the log file.

In the process, colors are removed from the message.

    USAGE
      log MSG

###  function assert

Finish a step as [DONE].

    USAGE
      assert MSG

###  function error

Triggers an ERROR with the given message.

Errors trigger notifications, if they are enabled, but do no not stop
the execution.

    USAGE
      error MSG

###  function die

Triggers a fatal ERROR, finishing the execution.

    USAGE
      die MSG

###  function info

Dispatch an INFO message.

    USAGE
      info MSG

###  function report

Reports a CHANGE.

Any CHANGE will trigger a notification of the run log, if notifications are
enabled.

    USAGE
      report MSG

###  function cmd

Runs the given command, unless we're in TRIAL mode (```-t```).

Prefix your cp, rm, mv. find, etc commands with 'cmd ' in front of them.

    USAGE
      cmd CMD ARG1 ARG2

###  function dir_is_empty

True if the given DIR is empty.

    USAGE
      dir_is_empty DIR

##  Notification functions

Functions related to notifications.

###  function notify

Triggers the notification of the current run log.

Notifications are send from an EXIT trap.

This function, will trigger there are changes.

    USAGE
      notify

###  function notify_run

Function to trigger notifications at the EXIT trap.

No arguments required. Internal use.

    USAGE
      notify_run

##  Rolling back functions

Functions related to saving rollbacks.

###  function rollback_path

Save modified files and directories.

See undo_last_changes too.

    USAGE
      rollback_path /path/to/file

###  function rollback_cmd

Function used to save modified steps.

Usually you register a command, to do the opposite than the previous or
the next command.

See undo_last_changes too.

    USAGE
      rollback_cmd COMMAND ARGS

###  function undo_last_files

Rollback the last filesystem changeset

Well, really print the files found, FIXME.

    USAGE
      undo_last_files

###  function undo_last_commands

Rollback the last commands changeset

Well, really print the code that should run, FIXME.

    USAGE
      undo_last_commands

###  function undo_last_changes

Generic rollback invocation

Important: The filesystem is restored first, and the commands run after.

    USAGE
      undo_last_changes

##  File functions

Functions related to file and directory definitions

###  function permissions

Sets the given permissions on a file.

The content is ignored. Argument order matters.

    USAGE
      permissions PATH OWNER GROUP MODE

    EXAMPLES
      permissions /path/to/dir  root root  750
      permissions /path/to/file oper oper 0644

###  function directory

Ensures a directory definition

    USAGE
      directory -n /path/to/dir -o root -g root -m 700
      directory name /path/to/dir ensure absent --trigger notify

###  function file

Ensures a file from op files

    USAGE
     file -n /path/to/file -o root -g root -m 400 -t "echo trigger command"
     file -n /path/to/file -e absent --trigger '/root/myscript.sh'

##  Package management functions

Functions related to package management.

###  function pkg

Ensure given packages (debian/redhat) status

Argument order matters, first the orders, last the packages.

    USAGE
      pkg present pkg1 pkg2
      pkg absent pkg1 pkg2
      pkg trigger auditscript.sh absent pkg1

###  function aptkey

Ensure an apt key

Works for my usage, rollback maybe broken. FIXME

    USAGE
      aptkey present /path/to/filename.key  # ex: nginx_signing.key
      aptkey absent keyid                   # ex: 7BD9BF62

##  Users and groups functions

Functions related to system users and groups.

###  function group

Ensure a system group

    USAGE
      group name oper gid 2222 members oper
      group name oper ensure absent

###  function user

Ensure the given user configuration

    USAGE
      user name oper groups oper,wheel shell /bin/bash pass 'PLAINTEXT'
      user name oper ensure absent

###  function doc

Query and extract the nested documentation.

    USAGE
      doc FILE [topics|TOPIC]

    EXAMPLES
      doc play/stage1 topics     # List them all
      doc op function            # Request all topics matching 'function'
      doc op function doc        # Show this help message

###  function doc_title

Create a markdown title of the given level (1 to 6)

    USAGE
      doc_title 4 "My title"

###  function run_op_init

This is an internal function, to initialize runs.

You should not call it.

It setups logging, blocks parallel runs, and initializes the following
facts (that you can reuse):

    USAGE
      run_op_init

###  function run_op

This is an internal function, to launch operations or requests.

    USAGE
      run_op "$@"

###  function import_path

Imports the given path to the op ```./files/``` tree.

The given argument, should be an **absolute** path.

    USAGE
      import_path PATH

###  function gen

Given a path, generates ```op``` rules for that files to STDOUT

If a second parameter of ```-p```, ```-perms```, ```--perms```, ```perms```,
is given, then will generate rules only for permisions (not for content).

The firs level will be removed in the case of paths starting by ```files/```.

Use with common sense, only for normal files (config files, aplication
files) and directories.

Not tested against special files (links, devices, sockets, etc).

    USAGE
      gen files/etc/newtree    # generate rules for file()
      gen /usr/bin --perms     # generate rules for permissions()

###  help
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
      doc FILE [topics|NAME]  Show detailed documentation, and exit
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

###  See also

See the project page at: http://1n1.github.io/op/

And the source repository at: https://github.com/1n1/op

##  License

    Copyright (C) 2015, Iñigo Tejedor Arrondo <http://inigo.me/>

    All rights reserved.

    Permission to use, copy, modify, and/or distribute this software for any
    purpose with or without fee is hereby granted, provided that the above
    copyright notice and this permission notice appear in all copies.

    THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
    WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
    MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
    ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
    WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
    ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
    OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.

