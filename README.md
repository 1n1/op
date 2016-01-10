
# op - the operations tool

## Introduction

TL;DR: This tool is just a shell wrapper of wrappers, to ease systems support.

Why whould I want to use it? if you're looking for a framework to make
your systems ease of discover and manage in repeatable ways, using the shell
as in ```/bin/sh``` for the interpreter, you may want to use ```op```.

Shell variables and functions, to write system definitions and operations.

Those internal functions, includes logging, change detection and rollback
features in their implementation (alpha state).

The core functions are designed to be repeatable.

There is an HTTP notification mechanism in place, to register runs remotely.

## Getting started

In the future there will be better ways but for now this one works fine:

    root@shell:~ # mkdir -m 0700 /root/op
    root@shell:~ # cd /root/op
    root@shell:~/op # wget -q -O op 'https://inigo.me/pub/op/op' && echo OK
    OK
    root@shell:~/op # sha256sum op
    c3dafeb07aee9026ca051a82697391aeaa35dec3a8d690c60d29cfef686ce26d  op
    root@shell:~/op # chmod 0700 op
    root@shell:~/op #

From there you should be able to run ```./op``` from ```/root/op```.

## Writing a system definition

Let's create a modified file, for a custom system:

     root@shell:~/op # mkdir -m 0700 files
     root@shell:~/op # mkdir -p files/root
     root@shell:~/op # cp -a /root/.bashrc files/root/

And now, let's create a definition for this file:

     root@shell:~/op # mkdir ops
     root@shell:~/op # echo file -n /root/bashrc -m 600 -o root -g root

It should be there...

    root@shell:~/op # ./op --op --list
    root
    root@shell:~/op #

So we can run it:

    root@shell:~/op # ./op --op root -v
    root@shell:~/op # ./op --op root -v
    [DONE] Already present: /root/.bashrc
    [DONE] Already the defined ownership (root:root) for: /root/.bashrc
    [DONE] Already the defined mode (0400) for: /root/.bashrc
    [DONE] Finished OK: ./ops/root
    root@shell:~/op #

## Dealing with system changes

Days latter, we can modify our file:

    root@shell:~/op # vim files/root/.bashrc 
    root@shell:~/op # ./op --op root -v
    [CHANGE] File with differences: /root/.bashrc
    --- /root/.bashrc	2015-12-08 17:34:38.500439001 +0100
    +++ /root/op/files/root/.bashrc	2016-01-10 17:44:28.016043941 +0100
    @@ -132,8 +132,8 @@
    declare -xr HISTFILE=~/.bash_history
    declare -xr HISTCONTROL=''    # ignorespace, ignoredups, ignoreboth, erasedups
    declare -xr HISTIGNORE=''     # globs. For ex. ?:?? ignore 1 and 2 char commands
    -declare -xr HISTFILESIZE=5000            # default is 500
    -declare -xr HISTSIZE=5000                # default is 500
    +declare -xr HISTFILESIZE=10000           # default is 500
    +declare -xr HISTSIZE=10000               # default is 500
    declare -x  HISTTIMEFORMAT="%A %F %T "   # 817 Tuesday 2012-10-16 10:35:16 ls -l
    declare -x  PROMPT_COMMAND='history -a'
    
    [CHANGE] Saving rollback: /root/.bashrc
    [CHANGE] Setting up: /root/.bashrc
    [DONE] Already the defined ownership (root:root) for: /root/.bashrc
    [DONE] Already the defined mode (0400) for: /root/.bashrc
    [DONE] Finished OK: ./ops/root
    root@shell:~/op #

Woops, what did happen? well, a change was made, applied, detected, a rollback
copy saved, a diff logged, and we did get to the *Desired State*.

After the hange did apply, we can run op as many times as we want, to ensure
that the *Desired State* is already there:

    root@shell:~/op # ./op
    root@shell:~/op # ./op -v
    [DONE] Already present: /root/.bashrc
    [DONE] Already the defined ownership (root:root) for: /root/.bashrc
    [DONE] Already the defined mode (0400) for: /root/.bashrc
    root@shell:~/op # ./op -v
    [DONE] Already present: /root/.bashrc
    [DONE] Already the defined ownership (root:root) for: /root/.bashrc
    [DONE] Already the defined mode (0400) for: /root/.bashrc
    root@shell:~/op # ./op -v --op root
    [DONE] Already present: /root/.bashrc
    [DONE] Already the defined ownership (root:root) for: /root/.bashrc
    [DONE] Already the defined mode (0400) for: /root/.bashrc
    [DONE] Finished OK: ./ops/root
    root@shell:~/op # 

So you did learn what "operations" are in the op context.

## Parametrized requests

Parametrized requests, are the same than operations, but they need to be
explicitly called:

    root@shell:~/op # mkdir requests
    root@shell:~/op # echo 'echo "Rebooting: $request_args"' > requests/reboot
    root@shell:~/op # ./op request lists
    reboot
    root@shell:~/op # ./op request reboot A B C
    Rebooting: A B C
    root@shell:~/op # 

That's it.

## Layout

System definitions (stack, files, users, etc) go as "operations", this is
sourced primitives stored under ```./ops/*```.

Parametrized system change requests, go under ```./requests/*```.

Files to be controlled or distributed, go under ```./files/*```.

You can list available support operations and requests, and of course,
execute them.

So typically you wil work with a tree like this:

    root@shell:~/op # tree
    .
    ├── files
    │   └── root
    ├── op
    ├── ops
    │   └── root
    └── requests
        └── reboot
    
    4 directories, 3 files
    root@shell:~/op # 

When ```op```runs, a new log is created, you can access througt ```./logs/latest```:

    root@shell:~/op # ./op
    root@shell:~/op # tree
    .
    ├── files
    │   └── root
    ├── logs
    │   ├── 2016
    │   │   └── 01
    │   │       └── 10
    │   │           └── 18-15-05.031234472.log
    │   └── latest -> /root/op/logs/2016/01/10/18-15-05.031234472.log
    ├── op
    ├── ops
    │   └── root
    └── requests
        └── reboot
    
    8 directories, 5 files
    root@shell:~/op # 

Also, when there are changes, rollback files are created:

    root@shell:~/op # echo "alias o='./op'" >> files/root/.bashrc
    root@shell:~/op # ./op
    [CHANGE] File with differences: /root/.bashrc
    --- /root/.bashrc	2016-01-10 18:12:27.776034157 +0100
    +++ /root/op/files/root/.bashrc	2016-01-10 18:17:44.952032310 +0100
    @@ -184,3 +184,4 @@
    chmod 0700 /root/op/op
    }
    
    +alias o='./op'
    [CHANGE] Saving rollback: /root/.bashrc
    [CHANGE] Setting up: /root/.bashrc
    root@shell:~/op # 

They will be available under ```./rollback/latest/```:

    root@shell:~/op # tree
    .
    ├── files
    │   └── root
    ├── logs
    │   ├── 2016
    │   │   └── 01
    │   │       └── 10
    │   │           ├── 18-15-05.031234472.log
    │   │           └── 18-17-48.549559851.log
    │   └── latest -> /root/op/logs/2016/01/10/18-17-48.549559851.log
    ├── op
    ├── ops
    │   └── root
    ├── requests
    │   └── reboot
    └── rollback
        ├── latest -> /root/op/rollback/shell.20160110181748547788051.30923
        └── shell.20160110181748547788051.30923
            └── root
    
    12 directories, 6 files
    root@shell:~/op # 

If there are commands logged as rollback, they will be writen in
```./rollback.undo/latest```.

## Why this software?

Unix-like system integrators and supporters, tend to create custom system
scripts.

Those scripts, tend to finish using random naming conventions, and living in
places like /usr/local/bin/do_something, ~/bin/do_it, or worse places like
/opt/org/app/foo/components/version/bin/do_that, etc...

Also, many time you repeat patterns... steps... procedures... or you need to
remember options for those operations, that you no use often enough.

Or let's say you need to create systems, to transfer to operations.

Then, if you reflect on any of the previous scenarios, this wraper may become
helpful for you.

## Recipes

You can reuse the internal functions and variables od ```op``` in your recipes.

Those core functions, will ensure and respect the message levels, trial modes,
and the overall repeatability.

    info "Starting the nginx setup"
    
    file -o root -g root -m 644 -n '/etc/logrotate.d/nginx'
    
    directory -o root -g root -m 755 -n '/etc'    
    directory -o root -g root -m 755 -n '/etc/nginx'
    directory -o root -g root -m 755 -n '/etc/nginx/conf.d'
    
    file -o root -g root -m 644 -n '/etc/nginx/conf.d/default.conf' \
         --trigger 'service nginx restart'
    
    aptkey present '/etc/nginx/nginx_signing.key' -t 'apt-get update'
    
    pkg present nginx
    
    info "Finished the nginx setup"

You can discover those functions in the ```op``` [MANUAL](MANUAL.txt).

All op invocations (except the help) are logged, including command output,
file diffs, user/sudo-user calling the program, etc...

Optionally, you can send notifications of changes, errors, or even normal
runs, and the log file by HTTP. See internal variables for this.

This script is just a convention to call to other custom scripts, the same you
maybe calling to Ansible, or Puppet, or a Nagios check, or a complex SQL, on
those custom scripts, below the shell umbrella.

The purposse is to simply have them called on a repeatable, auditable and easy
to remember, or discover, way.

## Examples

You can continue reading the [WORKFLOW](./WORKFLOW.md) to get more
instructions, and the [MANUAL](MANUAL.txt) while writing your own recipes.

You can browse [the op-examples](https://github.com/1n1/op-examples) to get
a feeling on howto use this tool.

Happy operations!

