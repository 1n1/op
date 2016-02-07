
#  op - *The* operations tool

This program, aims to ease systems definition and maintenance.

For more information, see: https://github.com/1n1/op

##  Initialization

When you call ```op``` without arguments, it sources all operations stored
in a relative subdirectory from the invocation dir, under ```./ops/*```.

Those sourced operations, can reuse this program variables and functions.

There are switches to call specific ```./ops/``` files (```-o```) and
specific requests under ```./requests/``` with (```-r```). We will reference
those custom files as "recipes" from now on.

By default we're quiet, with the program output. Use ```-v``` to see verbose
messages.

Use ```--trial``` on the command line, for test without applying commands.

The modes ```-e``` and ```-u```, are set at initialization, recipes will be
affected by such behavior.

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
* ```nocolor```: The colors switch (boold, default disabled, 0).
* ```hostname```: It contains the hostname at our launch time.

##  Color variables

The following foreground color variables, are initialized for reusing:

    normal, red, green, yellow, blue, magenta, cyan, white
###  Logging variables

Each run, gets a uniq identifier, stored in the variable: ```run_id```

This is used to manage the symlinks, to the latest log and the latest
rollback directories.

The variables ```logs``` and ```logfile``` will point to our current
log during execution. For internal use.

###  Notifications variables


Notifications are controlled by the boolean: ```notify_enabled```

Set to 1 to notify run results, set to 0 to disable notifications.

We start counting, like if there were no changes or errors...
HTTP notifications are configured using:

```http_url```: URL to use for the request
```http_headers```: Extra headers in curl format
```http_options```: All of the previous together

Use ```http_options``` to tune any extra curl option you may need.

##  General purpose functions

Functions for Output, messages and general purpose

###  function now

Prints a timestamp intended for messages. Precision is nanoseconds,
this may change for portability in a future.

No arguments.

    USAGE
      now

###  function log

Dispatchs a message to the log.

In the process, colors are remved from the message.

    USAGE
      log MSG

###  function assert

Finishes a step as [DONE].

    USAGE
      assert MSG

###  function error

Triggers an ERROR with the given message..

Errors trigger notifications if they are enabled. Do not stop execution.

    USAGE
      error MSG

###  function die

Triggers a fatal ERROR, finishing the execution.

    USAGE
      die MSG

###  function info

Dispatch a INFO message.

    USAGE
      info MSG

###  FUNTION report

Reports a CHANGE.

Runs with changes are notified (if enabled).

    USAGE
      report MSG

###  function cmd

Runs the given command, unless we're in TRIAL mode (-t).

Prefix cp, rm, mv. find, etc commands with 'cmd ' in front of them.

    USAGE
      cmd CMD ARG1 ARG2

###  function dir_is_empty

True if the given dir is empty.

    USAGE
      dir_is_empty DIR

##  Notification functions

Functions related to change detection notifications.

###  function notify

Triggers the notification of the current run.

Notifications are send from an EXIT trap. This function, will trigger
them if they are enabled:

    USAGE
      notify

###  function notify_run

 Function to trigger notifications at the EXIT trap

No arguments required.

    USAGE
      notify_run

##  Rolling back functions

Functions related to change detection notifications.

###  function rollback_path

Save modified files and directories.

See undo_last_changes too.

    USAGE
      rollback_path /path/to/file

###  function rollback_cmd

Function used to save modified steps.

Usually you register a command to do the opposite than the previous or
the next command.

See undo_last_changes too.

    USAGE
      rollback_cmd mycommand --to-do-the-opossite-step

###  function undo_last_files

Rollback the last filesystem changeset

Well, really print the files found, fixme.

    USAGE
      undo_last_files

###  function undo_last_commands

Rollback the last commands changeset

Well, really print the code that should run, fixme.

    USAGE
      undo_last_commands

###  function undo_last_changes

Generic rollback invocation

Important: The filesystem is restored first, and the commands run after.

    USAGE
      undo_last_changes

##  File functions

Functions related to file and directory manipulations

###  function permissions

Sets the given permissions on a file.

The content is ignored. Argument order matters.

    USAGE
      permissions PATH OWNER GROUP MODE

    EXAMPLES
      permissions /path/to/dir  root root  750
      permissions /path/to/file oper oper 0644

###  function directory

Ensures a directory

    USAGE
      directory -n /path/to/dir -o root -g root -m 700
      directory name /path/to/dir ensure absent

###  function file

Ensures a file from op files

    USAGE
     file -n /path/to/file -o root -g root -m 400 -t "echo trigger command"
     file -n /path/to/file -e absent

##  Package management functions

Functions related to package management.

###  function pkg

Ensure given (debian) packages status

Argument order matters, first the orders, last the packages.

    USAGE
      pkg present pkg1 pkg2
      pkg absent pkg1 pkg2
      pkg trigger whatever absent pkg1

###  function aptkey

Ensure an apt key

FIXME needs review, works for my usage, rollback maybe broken

    USAGE
      aptkey present /path/to/filename.key  # ex: nginx_signing.key
      aptkey absent keyid                   # ex: 7BD9BF62

##  Users and groups functions

Functions related to system users and groups

###  function group

Ensure a system group

    USAGE
      group name oper gid 2222 members oper
      group name oper ensure absent

###  function user

Ensure the given user configuration

TO-DO Rreview and add more options.

    USAGE
      user name oper groups oper,wheel shell /bin/bash pass 'PLAINTEXT'
      user name oper ensure absent

###  function doc

Query and extract the nested documentation.

    USAGE
      doc [topics|TOPIC]

    EXAMPLES
      doc topics        # List them all
      doc function      # Request all topics matching function
      doc function doc  # Show this help
###  function doc_title

Create a markdown title of the given level (1 to 6)

    USAGE
      doc_title 4 "My title"

###  function run_op_init

This is an internal function, to initialize runs.

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

###  function op_help

Prints the ```op``` help message and exits the program.

If a numeric argument is given, exits with that status.

If more arguments ar given, print them before the help meassage.

    USAGE
      ophelp
      >&2 op_help 4 "Exiting with error 4"

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

