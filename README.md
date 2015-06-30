
# OP the operations tool

## Introduction

This tool is just a shell wrapper of wrappers, to ease systems support.

It provides internal functions to write system definitions and operations.

Those internal functions, includes logging,  change detection and rollback
features in their implementation (alpha state).

You can list available support operations and requests, and of course,execute
them.

## Why this software?

Unix-like system integrators and supporters, tend to create custom system
scripts.

Those scripts use to finish in /usr/local/bin/do_something, ~/bin/do_it, etc...

Now you're an operator. You connect to a new system (lets say, srv1539) and you
want to discover what it does:

    cd /root/op
    ./op --op list

If you want to know what each item of the list does, just check the file
on ```./ops/filename```, they should be easy primitives to understand. Here
you can see an example:


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

5 minuttes latter, you get a request of "do_whatever_buz" with parameters "foo"
and "bar" on another server (lets say, customer57os26srv34)... easy:

    cd /root/op
    ./op --request do_whatever_buz foo bar

All op invocations (but help) are logged, including command output, file diffs,
user/sudo-user calling the program, etc...

Optionally, if you setup an mta in the host where ```op``` is called, you
can send output of changes and errors by mail.

## Examples

You can browse [the op-examples](https://github.com/1n1/op-examples) to get
a feeling on howto use this tool.

You can take a look for example ops, and example requests there, and of course,
reuse them as dessired.

Happy operations!

