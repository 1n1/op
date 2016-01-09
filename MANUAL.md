    
     op - the operations tool.
    
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
    
     +------------------------------+
     | Initialization and variables |
     +------------------------------+
    
     When you call 'op' without arguments, it sources all operations stored in
     a relative subdirectory from the invocation dir, under ./ops/*
    
     Those sourced operations, can reuse this program variables and functions.
    
     There are switches to call specific ./ops/ files (-o) and specific requests
     under ./requests/ (-r). We will reference those custom files as "recipes"
     from now on.
    
     The modes -e and -u, are set at initialization, recipes will be affected
     by such behavior.
    
     By default we're quiet, with the program output. Use -v to see verbose
     messages.
    
     Use --trial on the command line, for test without applying commands.
    
     By now, only for root can run op. This may change in a future.
    
     The program initializes the following variables, that you can latter
     reuse in your recipes.
    
     * version: The program version.
     * op: contains the executable name, used on messages, etc.
     * work_dir: The directory where the program was invocated.
     * files: The main directory for files used in recipes.
     * rollback_dir: The rollback files main directory.
     * lock: The file used to block against parallel sessions.
     * verbose: The verbosity switch (bool, default disabled, 0).
     * trial: The trial switch (bool, default disabled, 0).
     * nocolor: The colors switch (boold, default disabled, 0).
     * hostname: It contains the hostname at our launch time.
    
     The following foreground color variables, are initialized for reusing:
    
         normal, red, green, yellow, blue, magenta, cyan, white
    
    
     The help message, is stored in the variable: op_help
    
     +---------------+
     | Logging files |
     +---------------+
    
     Each run, gets a uniq identifier, stored in the variable: run_id
    
     This is used to manage the symlinks, to the latest log and the latest
     rollback directories.
    
     The variables 'logs' and 'logfile' will point to our current log during
     execution. Mostly for internal use.
    
     +-------------------------+
     | Notifications variables |
     +-------------------------+
    
     Notifications are controlled by the boolean: notify_enabled
    
     Set to 1 to notify run results, set to 0 to disable notifications.
    
     Mail notifications are configured using:
    
     * mail_to: main recipient
     * mail_cc: addresses with a copy
    
     The notify_tpl() funtion, emits the mail body sent.
    
     Edit all of them to your needs if you enable notifications.
    
    
     +--------------------------------------------------+
     | Functions: Output, messages and general purpose. |
     +--------------------------------------------------+
    
    
     FUNCTION now - prints a timestamp intended for messages.
    
     Precision is nanoseconds, this may change for portability in a future.
    
     No arguments.
    
     USAGE
       now
    
    
     FUNCTION log - dispatchs a message to the log.
    
     In the process, colors are remved from the message.
    
     USAGE
       log MSG
    
    
     FUNCTION assert - finishes a step as [DONE].
    
     USAGE
       assert MSG
    
    
     FUNCTION error - triggers an ERROR with the given message..
    
     Errors trigger notifications if they are enabled. Do not stop execution.
    
     USAGE
       error MSG
    
    
     FUNCTION die - triggers a fatal ERROR, finishing the execution.
    
     USAGE
       die MSG
    
    
     FUNCTION info - dispatch a INFO message.
    
     USAGE
       info MSG
    
    
     FUNTION report - reports a CHANGE.
    
     Runs with changes are notified (if enabled).
    
     USAGE
       report MSG
    
    
     FUNCTION cmd - runs the given command, aware of the TRIAL mode (-t).
    
     Prefix cp, rm, mv. find, etc commands with 'cmd ' in front of them.
    
     USAGE
       cmd CMD ARG1 ARG2
    
    
     FUNCTION dir_is_empty - true if the given dir is empty.
    
     USAGE
       dir_is_empty DIR
    
    
     +------------------------+
     | Notification functions |
     +------------------------+
    
    
     FUNCTION notify - triggers the notification of the current run.
    
     Notifications are send from an EXIT trap. This function, will trigger
     them if they are enabled:
    
     USAGE
       notify
    
    
     FUNCTION notify_run - function to run notifications on EXIT trap
    
     No arguments required.
    
     USAGE
       notify_run
    
    
     +------------------------+
     | Rolling-back functions |
     +------------------------+
    
    
     FUNCTION rollback_path - Save modified files and directories.
    
     See undo_last_changes too.
    
     USAGE
       rollback_path /path/to/file
    
    
     FUNCTION rollback_cmd - Function used to save modified steps.
    
     Usually you register a command to do the opposite than the previous or
     the next command.
    
     See undo_last_changes too.
    
     USAGE
       rollback_cmd mycommand --to-do-the-opossite-step
    
    
     FUNCTION undo_last_files - Rollback the last filesystem changeset
    
     Well, really print the files found, fixme.
    
     USAGE
       undo_last_files
    
    
     FUNCTION undo_last_commands - Rollback the last commands changeset
    
     Well, really print the code that should run, fixme.
    
     USAGE
       undo_last_commands
    
    
     FUNCTION undo_last_changes - Generic rollback invocation
    
     Important: The filesystem is restored first, and the commands run after.
    
     USAGE
       undo_last_changes
    
    
     +------------------------+
     | File related functions |
     +------------------------+
    
    
     FUNCTION permissions - Sets the given permissions.
    
     The content is ignored. Argument order matters.
    
     USAGE
       permissions PATH OWNER GROUP MODE
    
     EXAMPLES
       permissions /path/to/dir  root root  750
       permissions /path/to/file oper oper 0644
    
    
     FUNCTION directory - Ensures a directory
    
     USAGE
       directory -n /path/to/dir -o root -g root -m 700
       directory name /path/to/dir ensure absent
    
    
     FUNCTION file - Ensures a file from op files
    
     USAGE
      file -n /path/to/file -o root -g root -m 400 -t "echo trigger command"
      file -n /path/to/file -e absent
    
    
     +------------------------------+
     | Package management functions |
     +------------------------------+
    
    
     FUNCTION pkg - ensure given (.deb) packages status
    
     Argument order matters, first the orders, last the packages.
    
     USAGE
       pkg present pkg1 pkg2
       pkg absent pkg1 pkg2
       pkg trigger whatever absent pkg1
    
    
     FUNCTION aptkey - ensure an apt key
    
     FIXME needs review, works for my usage, rollback maybe broken
    
     USAGE
       aptkey present /path/to/filename.key  # ex: nginx_signing.key
       aptkey absent keyid                   # ex: 7BD9BF62
    
    
     +----------------------------+
     | Users and groups functions |
     +----------------------------+
    
    
     FUNCTION group - ensure a system group
    
     USAGE
       group name oper gid 2222 members oper
       group name oper ensure absent
    
    
     FUNCTION user - ensure the given user configuration
    
     TO-DO Rreview and add more options.
    
     USAGE
       user name oper groups oper,wheel pass 'PLAINTEXT'
       user name oper ensure absent
    
