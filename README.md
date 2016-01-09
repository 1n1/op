
# op - the operations tool

## Introduction

TL;DR: This tool is just a shell wrapper of wrappers, to ease systems support.

Why whould I want to use it? if you're looking for a framework to make
your systems ease of discover and manage in repeatable ways, using the shell
as in ```/bin/sh``` for the interpreter, you may want to use ```op```.

Shell variables and functions, to write system definitions and operations.

Those internal functions, includes logging, change detection and rollback
features in their implementation (alpha state).

System definitions (stack, files, users, etc) go as "operations", this is
sourced primitives stored under ```./ops/*```.

Parametrized system change requests, go under ```./requests/*```.

You can list available support operations and requests, and of course,
execute them.

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

You connect to a new system (lets say, srv1539) and you want to discover what
it does because you did get an alert:

    cd /root/op
    ./op --op list

Magic? no, you simple get what was defined previosuly. Which maybe nothing.

If you get a response, and you want to know each item of the list in detail,
just check the file on ```./ops/filename```, they should be easy primitives
to understand. This is /bin/sh with a few core functions.

Here you can see one, as an example:


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

From there, we can say that the master files, used in that operation, are:

    /root/op/files/etc/logrotate.d/nginx
    /root/op/files/etc/nginx/conf.d/default.conf
    /root/op/files/etc/nginx/nginx_signing.key

5 minutes latter, you get a request of "do_whatever_buz" with parameters "foo"
and "bar" on another server (lets say, customer57os26srv34)... what a mess... easy:

    cd /root/op
    ./op --request do_whatever_buz foo bar

All op invocations (except the help) are logged, including command output, file diffs,
user/sudo-user calling the program, etc...

Optionally, if you setup an MTA in the host where ```op``` is called, you
can send output of changes and errors by mail.

This script is just a convention to call to other custom scripts, the same you
maybe calling to Ansible, or Puppet, or a Nagios check, or a complex SQL, on those
custom scripts, below the shell umbrella. The purposse is to simply have them
called on a repeatable, auditable and easy to remember or discover way.

## Examples

You can browse [the op-examples](https://github.com/1n1/op-examples) to get
a feeling on howto use this tool.

Happy operations!

