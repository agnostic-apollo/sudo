# sudo

`sudo` is a wrapper script to drop to any [supported shell](#Supported-Shells) or execute shell script files or their text passed as an argument with `superuser (root)` context in [Termux App]. Check the [Usage](#Usage) and [Command Types](#Command-Types) sections for more info on what type of commands can be run. `sudo` stands for *superuser do*.

The device must be rooted and ideally `Termux` must have been granted root permissions by your root manager app like [SuperSU] or [Magisk] for the `sudo` script to work.

Make sure to read the [Worthy Of Note](#Worthy-Of-Note) section, **specially the [RC File Variables](#rc-file-variables) section. This is very important, specially if you were previously using [termux-sudo by st42]**.

To use `sudo` with [Termux:Tasker] plugin and [RUN_COMMAND Intent], check [Termux:Tasker] `Setup Instructions` section for details on how to set them up. The [Tasker App] or your plugin host app must be granted `com.termux.permission.RUN_COMMAND` permission. The `sudo` script must be installed at `$PREFIX/bin/sudo`. The `allow-external-apps` property must also be set to `true` in `~/.termux/termux.properties` file since the `$PREFIX/bin/sudo` absolute path is outside the `~/.termux/tasker/` directory. For android `>= 10`, the [Termux App] should also be granted `Draw Over Apps` permission so that foreground commands automatically start executing without the user having to manually click the `Termux` notification in the status bar dropdown notifications list for the commands to start. Check [Templates](#Templates) section for template tasks that can be run used to run `sudo` from `Termux:Tasker` plugin and `RUN_COMMAND Intent`.

Note that this `sudo by agnostic-apollo`, [termux-sudo by st42] and [tsu by cswl] are competing packages/scripts and ideally only one of them should be used. Also note that when you install or update `tsu`, it creates a symlink at `$PREFIX/bin/sudo` for its own `sudo` command. So installing or updating `tsu` after installing `sudo by agnostic-apollo` will replace the `sudo by agnostic-apollo` script file with the `tsu` symlink and installing `sudo by agnostic-apollo` if you have already installed `tsu` will break its own `sudo` command.

If you want to run commands in `termux` user context, check [tudo].
##



### Contents
- [Dependencies](#Dependencies)
- [Downloads](#Downloads)
- [Install Instructions For Termux In Android](#Install-Instructions-For-Termux-In-Android)
- [Current Features](#Current-Features)
- [Planned Features](#Planned-Features)
- [Usage](#Usage)
- [Command Types](#Command-Types)
- [Supported Shells](#Supported-Shells)
- [Command Options](#Command-Options)
- [Shell Home](#Shell-Home)
- [Shell RC Files](#Shell-RC-Files)
- [Shell History Files](#Shell-History-Files)
- [Modifying Default Values](#Modifying-Default-Values)
- [Examples](#Examples)
- [Templates](#Templates)
- [Passing Arguments](#Passing-Arguments)
- [Issues](#Issues)
- [Worthy Of Note](#Worthy-Of-Note)
- [Tests](#Tests)
- [FAQs And FUQs](#FAQs-And-FUQs)
- [Changelog](#Changelog)
- [Contributions](#Contributions)
- [Credits](#Credits)
- [Donations](#Donations)
##



### Dependencies

- [Termux App]

Using `sudo` directly from inside `termux` terminal session does not have any specific version requirements, other than `bash` version `>= 4.1`.
However, to use `sudo` with [Termux:Tasker] plugin and [RUN_COMMAND Intent] requires the following versions to be installed. Check [Passing Arguments](#Passing-Arguments) section and [Termux:Tasker] `Setup Instructions` section for details.

- [Termux App] version `>= 0.100`
- [Tasker App] version `>= 5.9.4.beta`
- [Termux:Tasker] version `>= 0.5`
##



### Downloads

Latest version is `v0.1.0`.

- [GitHub releases](https://github.com/agnostic-apollo/sudo/releases).
##



### Install Instructions For Termux In Android

The `sudo` file should be placed in termux `bin` directory `/data/data/com.termux/files/usr/bin`.  
It should have `termux` `uid:gid` ownership and have executable `700` permission before it can be run directly without `bash`.  

1. Download the `sudo` file.  

    - Download to termux bin directory directly from github using `curl` using a non-root termux shell.  
        Run `pkg install curl` to install `curl` first.  
        - Latest release:  

          `curl -L 'https://github.com/agnostic-apollo/sudo/releases/latest/download/sudo' -o "/data/data/com.termux/files/usr/bin/sudo"`  

        - Specific release:  

          `curl -L 'https://github.com/agnostic-apollo/sudo/releases/download/v0.1.0/sudo' -o "/data/data/com.termux/files/usr/bin/sudo"`  

        - Master Branch *may be unstable*:  

          `curl -L 'https://github.com/agnostic-apollo/sudo/raw/master/sudo' -o "/data/data/com.termux/files/usr/bin/sudo"`  

    - Download `sudo` file manually from github to the android download directory and then copy it to termux bin directory.  

      You can download the `sudo` file from a github release from the `Assets` dropdown menu.  

      You can also download it from a specific github branch/tag by opening the `sudo` file from the `Code` section.  
      Right-click or hold the `Raw` button at the top and select `Download/Save link`.  

      Then copy the file to termux bin directory using `cat` command below or use a root file browser to manually place it.  

       `cat "/storage/emulated/0/Download/sudo" > "/data/data/com.termux/files/usr/bin/sudo"`  

2. Set `termux` ownership and executable permissions.  

    - If you used a `curl` or `cat` to copy the file, then use a non-root termux shell to set ownership and permissions with `chown` and `chmod` commands respectively:  

      `export termux_bin_path="/data/data/com.termux/files/usr/bin"; export owner="$(stat -c "%u" "$termux_bin_path")"; chown "$owner:$owner" "$termux_bin_path/sudo" && chmod 700 "$termux_bin_path/sudo";`  

    - If you used a root file browser to copy the file, then use `su` to start a root shell to set ownership and permissions with `chown` and `chmod` commands respectively:  

      `export termux_bin_path="/data/data/com.termux/files/usr/bin"; export owner="$(stat -c "%u" "$termux_bin_path")"; su -c "chown \"$owner:$owner\" \"$termux_bin_path/sudo\" && chmod 700 \"$termux_bin_path/sudo\"";`  

    - Or manually set them with your root file browser. You can find `termux` `uid` and `gid` by running the command `id -u` in a non-root termux shell or by checking the properties of the termux `bin` directory from your root file browser.  
##



### Current Features

- Allows dropping to an interactive shell in `termux` user context for any of the supported [Interactive Shells](#Interactive-Shells) with priority to either termux or android binary and library paths.
- Allows running single commands in `superuser (root)` context without having to start an interactive shell.
- Allows passing of script file paths or script text as arguments for any of the supported [Script Shells](#Script-Shells) to have them executed in `termux` user context without having to create physical script files first for the later case, like in `~/.termux/tasker/` directory for [Termux:Tasker].
- Automatic setup of home directories, `rc` files, `history` files and working directories with proper ownership and permissions.
- Automatic setup of the shell environment and exporting of all required variables including `LD_PRELOAD` so that termux commands work properly, specifically if being run from [Termux:Tasker] or [RUN_COMMAND Intent].
- Provides a lot of [Command Options](Command-Options) that are specifically designed for usage with [Termux:Tasker] and the [RUN_COMMAND Intent].
##



### Planned Features

`-`
##



### Usage

```
sudo is a wrapper script to drop to the supported shells or execute
shell script files or their text passed as an argument with super
user (root) context in termux.


Usage:
  sudo [command_options] su
  sudo [command_options] asu
  sudo [command_options] [-p] <command> [command_args]
  sudo [command_options] -s <core_script> [core_script_args]


Available command_options:
  [ -h | --help ]    Display this help screen.
  [ --help-extra ]   Display more help about how sudo command works.
  [ -q | --quiet ]    Set log level to 'OFF'.
  [ -v | -vv ]       Set log level to 'DEBUG', 'VERBOSE'.
  [ --version ]      Display version.
  [ -a ]             Force set priority to android paths for path
                     command type.
  [ -b ]             Go back to last activity after running core_script.
  [ -B ]             Run core_script in background.
  [ -c ]             Clear shell after running core_script.
  [ -d ]             Disable stdin for core_script.
  [ -D ]             Disable preserve environment for su.
  [ -e ]             Exit early if core_script fails.
  [ -E ]             Exec interactive shell or the path command.
  [ -f ]             Force use temp script file for core_script.
  [ -F ]             Consider core_script to be a path to script file
                     instead of script text.
  [ -H ]             Same sudo post shell home as sudo shell home.
  [ -i ]             Run interactive sudo post shell after running
                     core_script.
  [ -l ]             Go to launcher activtiy after running core_script
  [ -L ]             Export all existing paths in '$LD_LIBRARY_PATH'
                     variable.
  [ -n ]             Redirect stderr to /dev/null for core_script.
  [ -N ]             Redirect stdout and stderr to /dev/null for
                     core_script.
  [ -o ]             Redirect stderr to stdout for core_script.
  [ -O ]             Redirect stdout to stderr for core_script.
  [ -p ]             Set 'path' as command type [default].
  [ -P ]             Export all existing paths in '$PATH' variable.
  [ -r ]             Parse commands as per RUN_COMMAND intent rules.
  [ -R ]             Use root for searching and validating paths.
  [ -s ]             Set 'script' as command type.
  [ -S ]             Same sudo post shell as sudo shell.
  [ --comma-alternative=<alternative> ]
                     Comma alternative character to be used for
                     the '-r' option instead of the default.
  [ --dry-run ]
                     Do not execute sudo commands.
  [ --export-paths=<paths> ]
                     Additional paths to export in PATH variable,
                     separated with colons ':'.
  [ --export-ld-lib-paths=<paths> ]
                     Additional paths to export in LD_LIBRARY_PATH
                     variable, separated with colons ':'.
  [ --force-remount-ro ]
                     Force remount rootfs and system partitions back
                     to ro after sudo commands.
  [ --hold[=<string>] ]
                     Hold sudo from exiting until string is entered,
                     defaults to any character if string is not passed.
  [ --hold-if-fail ]
                     If '--hold' option is passed, then only hold if
                     exit code of sudo does not equal '0'.
  [ --list-interactive-shells ]
                     Display list of supported interactive shells.
  [ --list-script-shells ]
                     Display list of supported script shells.
  [ --no-create-rc ]
                     Do not create rc files automatically.
  [ --no-create-hist ]
                     Do not create history files automatically.
  [ --no-hist ]
                     Do not save history for sudo shell and sudo post
                     shell.
  [ --no-log-args ]
                     Do not log arguments and core_script content
                     when log level is '>= DEBUG'.
  [ --no-remount-ro ]
                     Do not remount rootfs and system partitions back
                     to ro after sudo commands.
  [ --keep-temp ]
                     Do not delete sudo temp directory on exit.
  [ --post-shell=<shell> ]
                     Name or absolute path for sudo post shell.
  [ --post-shell-home=<path> ]
                     Absolute path for sudo post shell home.
  [ --post-shell-options=<options> ]
                     Additional options to pass to sudo post shell.
  [ --post-shell-post-commands=<commands> ]
                     Bash commands to run after sudo post shell.
  [ --post-shell-pre-commands=<commands> ]
                     Bash commands to run before sudo post shell.
  [ --post-shell-stdin-string=<string> ]
                     String to pass as stdin to sudo post shell.
  [ --remove-prev-temp ]
                     Remove temp files and directories created on
                     previous runs of sudo command.
  [ --script-decode ]
                     Consider the core_script as base64
                     encoded that should be decoded before execution.
  [ --script-name=<name> ]
                     Filename to use for the core_script temp file
                     created in '.sudo.temp.XXXXXX' directory instead
                     of 'sudo_core_script'.
  [ --script-redirect=<mode/string> ]
                     Core_script redirect mode for stdout and stderr.
  [ --shell=<shell> ]
                     Name or absolute path for sudo shell.
  [ --shell-home=<path> ]
                     Absolute path for sudo shell home.
  [ --shell-options=<options> ]
                     Additional options to pass to sudo shell.
  [ --shell-post-commands=<commands> ]
                     Bash commands to run after sudo shell for script
                     command type.
  [ --shell-pre-commands=<commands> ]
                     Bash commands to run before sudo shell.
  [ --shell-stdin-string=<string> ]
                     String to pass as stdin to sudo shell for script
                     command type.
  [ --sleep=<seconds> ]
                     Sleep for x seconds before exiting sudo.
  [ --sleep-if-fail ]
                     If '--sleep' option is passed, then only sleep if
                     exit code of sudo does not equal '0'.
  [ --su-env-options=<options> ]
                     Additional options to pass to su that sets up the
                     sudo environment.
  [ --su-run-options=<options> ]
                     Additional options to pass to su that runs the
                     final sudo command_type command.
  [ --title=<title> ]
                     Title for sudo shell terminal.
  [ --work-dir=<path> ]
                     Absolute path for working directory.


Set log level to '>= DEBUG' to get more info when running sudo command.

Pass '--dry-run' option with log level '>= DEBUG' to see the commands
that will be run without actually executing them.

Visit https://github.com/agnostic-apollo/sudo for more help on how
sudo command works.

Supported interactive shells: `bash zsh dash sh fish python ruby pry node perl lua5.2 lua5.3 lua5.4 php python2 ksh`

Supported script shells: `bash zsh dash sh fish python ruby node perl lua5.2 lua5.3 lua5.4 php python2 ksh`


The 'su' command type drops to an interactive shell in superuser (root)
context for any of the supported interactive shells. To drop to a root
'bash' shell, just run 'sudo su'. The priority will be set to termux
bin and library paths in '$PATH' and '$LD_LIBRARY_PATH' variables.
Use the '--shell' option to set the interactive shell to use.


The 'asu' command type is the same as 'su' command type but
instead the priority will be set to android bin and library paths in
'$PATH' and '$LD_LIBRARY_PATH' variables.
Use the '--shell' option to set the interactive shell to use.


The 'path' command type runs a single command in superuser (root)
context. You can use it just by running 'sudo <command> [command_args]'
where 'command' is the executable you want to run and 'command_args'
are any optional arguments to it. The 'command' will be run within a
'bash' shell. Priority is given to termux bin and library paths unless
'command' exists in '/system' partition.
To call the 'su' binary, run the 'sudo -p su [user]' command.


The 'script' command type takes any script text or path to a script
file for any of the supported script shells referred as 'sudo shell',
and executes the script with any optional arguments with the desired
script shell. This can be done by running the
'sudo -s <core_script> [core_script_args]' command.
The 'core_script' will be considered a 'bash' script by default.
The 'core_script' will be passed to the desired shell using
process substitution or after storing the 'core_script' in a temp file
in a temp directory in 'sudo shell' home
'$HOME/.sudo.temp.XXXXXX/sudo_core_script' and passing the path to
the desired shell, where 'XXXXXX' is a randomly generated string.
The method is automatically chosen based on the script shell
capabilities. The '-f' option can be used to force the usage of a
script file. The '-F' option can passed so that the 'core_script'
is considered as a path to script file that should be passed to
'sudo shell' directly instead of considering it as a script text.
Use the '--shell' option to set the script shell to use.
Use the '--post-shell' option to set the interactive shell to use if
'-i' option is passed.


Run "exit" command of your shell to exit interactive shells and return
to the termux shell.
```
##



### Command Types


#### `su`

The `su` command type drops to an interactive shell in `superuser (root)` context for any of the supported [Interactive Shells](#Interactive-Shells). `su` stands for *substitute user* which in this case will be the `superuser (root)`. To drop to a root `bash` shell, just run `sudo su`. The priority will be set to termux bin and library paths in `$PATH` and `$LD_LIBRARY_PATH` variables. Check the [PATH and LD_LIBRARY_PATH Priorities](#path-and-ld_library_path-priorities) section for more info.

Note that `su` is just a command type and does not represent the `su` binary itself. Use the `path` command type to run the `sudo -p su [user]` command instead for calling the `su` binary.
&nbsp;&nbsp;



#### `asu`

The `asu` command type is the same as `su` command type but instead the priority will be set to android bin and library paths in `$PATH` and `$LD_LIBRARY_PATH` variables. Check the [PATH and LD_LIBRARY_PATH Priorities](path-and-ld_library_path-priorities) section for more info.
&nbsp;&nbsp;



#### `path`

The `path` command type runs a single command in `superuser (root)` context. You can use it just by running `sudo <command> [command_args]` where `command` is the executable you want to run and `command_args` are any optional arguments you want to pass to it.

The `command` will be run within a `bash` shell. Priority is given to termux bin and library paths unless `command` exists in `/system` partition. `sudo <command>` will not work if executable to be run does not have proper ownership or executable permissions set that disallows `termux` user to read or execute it if `sudo` command itself is being run from the `termux` context and `-R` option is not passed. The `command` must be an `absolute path` to an executable, or `basename` to an executable in the current directory or in a directory listed in the final `$PATH` variable that is to be exported by the `sudo` command. If it is not found, `sudo` will exit with an error.

The `path` command type is of course useful for running single commands with root context without having to drop to a root shell, but its also very useful for running commands in `/system` partition that require priorities to be set to android library paths and which fail otherwise with errors like `CANNOT LINK EXECUTABLE` and `cannot locate symbol some_symbol referenced by /lib....`. The `sudo` command will automatically detect if the `command` exists in `/system` partition and set priorities to android bin and library paths in `$PATH` and `$LD_LIBRARY_PATH` variables. So running `sudo dumpsys` will just work. You can also force setting priority to android paths by passing the `-a` option or to run a binary in `/system` partition instead of that in termux bin paths.

You can also use `sudo <command>` even if you are inside of a `sudo su` root shell and it will work without having to switch to `sudo asu` or exporting variables to change priority.

You can also run the `sudo -p su [user]` or `sudo -p /path/to/su [user]` commands to call the `su` binary for dropping to a shell for a specific user or even run a command for a specific user, like `sudo -p su -c "logcat" system`. Note that if you do not provide an absolute path to the `su` binary and just run `sudo -p su`, then the termux `su` wrapper script will be called which is stored at `$PREFIX/bin/su` which automatically tries to find the `su` binary and unsets `LD_LIBRARY_PATH` and `LD_PRELOAD` variables. You can check its contents with `cat "$PREFIX/bin/su"`. The variables will be also be unset by the `sudo` script if it detects you are trying to run a `su` binary.

Check the `-a` and `-r` command options that can be specifically used with the `path` command type.
&nbsp;&nbsp;



#### `script`

The `script` command type takes any script text or path to a script file for any of the supported [Script Shells](#Script-Shells) referred as `sudo shell`, and executes the script with any optional arguments with the desired script shell. This can be done by running the `sudo -s <core_script> [core_script_args]` command. The `core_script` will be considered a `bash` script by default.

The `script` command type is incredibly useful for usage with termux plugins like [Termux:Tasker] or [RUN_COMMAND Intent]. Currently, any script files that need to be run need to be created in `~/.termux/tasker/` directory, at least for `Termux:Tasker`. It may get inconvenient to create physical script files for each type of command you want to run. These script files are also neither part of backups of plugin host apps like Tasker and require separate backup methods and nor are part of project configs shared with other people or even between your own devices, and so the scripts need to be added manually to the `~/.termux/tasker/` directory on each device. To solve such issues and to dynamically define scripts of different interpreted languages inside your plugin host app like `Tasker` in local variables (all lowercase `%core_script`) of a task and to pass them to `Termux` as arguments instead of creating script files, the `script` command type can be used. The termux environment will also be properly loaded like setting `LD_PRELOAD` etc before running the commands.

The `core_script` will be passed to the desired shell using [Process Substitution] or after storing the `core_script` in a temp file in a temp directory in `sudo shell` home `$HOME/.sudo.temp.XXXXXX/sudo_core_script` and passing the path to the desired shell, where `XXXXXX` is a randomly generated string. The method is automatically chosen based on the script shell capabilities. The `-f` option can be used to force the usage of a script file. If the temp directory is created, it will be empty other than the `sudo_core_script` file and will be unique for each execution of the script, which the script can use for other temporary stuff without having to worry about cleanup since the temp directory will be automatically removed when `sudo` command exits unless `--keep-temp` is passed. The temp directory path will also be exported in the `$SUDO_SCRIPT_DIR` environment variable which can be used by the `core_script`, `post shell` and `--*shell-*-commands` options, like `--shell-pre-commands='cd "$SUDO_SCRIPT_DIR"'`. The `$HOME` refers to the `sudo shell` home.

For `bash zsh fish ksh python python2 ruby perl lua5.2 lua5.3 lua5.4`, process substitution is used by default and for `dash sh node php` a file is used. If the usage of process substitution is breaking for some complex scripts of some specific shell, please report the issue.

The `-F` option can be passed so that the `core_script` is considered as a path to a script file that should be passed to `sudo shell` directly instead of considering it as a script text.

The `core_script` can optionally not be passed or passed as an empty string so that other "features" of the `script` command type can still be used without calling the script shell.

It may also be important to automatically open an interactive shell after the `core_script` completes. This can be done by using the  `-i` option along with `--post-shell*` options. The `sudo post shell` can be any of the supported [Interactive Shells](#Interactive-Shells) and defaults to `bash`. The same shell as the script `sudo shell` can also be used for `sudo post shell` by passing the `-S` option as long as the `sudo shell` exists in the list of supported interactive shells. The environment variable `$SUDO_SCRIPT_EXIT_CODE` will be exported containing the exit code of the `core_script` before the interactive shell is started. Running an interactive shell will also keep the terminal session open after commands complete which is normally closed automatically when commands are run with the plugin or intents, although the `--hold` option can also be used for this.

You can define your own exit traps inside the `core_script`, but **DO NOT** define them outside it with the `--*shell-*-commands` options since `sudo` defines its own trap function `sudo_script_trap` for cleanup, killing child processes and to exit with the trap signal exit code. If you want to handle traps outside the `core_script`, then define a function named `sudo_script_custom_trap` which will automatically be called by `sudo_script_trap`. The function will be sent `TERM`, `INT`, `HUP`, `QUIT` as `$1` for the respective trap signals. For the `EXIT` signal the `$1` will not be passed. Do not `exit` inside the `sudo_script_custom_trap` function. If the `sudo_script_custom_trap` function exits with exit code `0`, then the `sudo_script_trap` will continue to exit with the original trap signal exit code. If it exits with exit code `125` `ECANCELED`, then `sudo_script_trap` will consider that as a cancellation and will just return without running any other trap commands. If any other exit code is returned, then the `sudo_script_trap` will use that as exit code instead of the original trap signal exit code.

Check the `-b`, `-B`, `-c`, `-d`, `-e`, `-E`, `-f`, `-F`, `-l`, `-n`, `N`, `-o`, `O`, `-r`, `--remove-prev-temp`, `--keep-temp`, `--shell*`, `--post-shell*`, `--script-decode`, `--script-redirect`, `--script-name` command options that can be specifically used with the `script` command type.
##



### Supported Shells

The `bash` shell is the default interactive and script shell and must exist at `$PREFIX/bin/bash` with ownership and permissions allowing `termux` user to read and execute it. The `--shell` and `--post-shell` options can be used to change the default shells. The `path` command type always uses the `bash` shell and command options are ignored. Normally, shells are not validated as the root user unless `-R` is passed so they must have proper ownership or executable permissions set that allows `termux` user to read and execute them.

The exported environmental variables `$SUDO_SHELL_PS1` and `$SUDO_POST_SHELL_PS1` can be used to change the default `$PS1` values of the shell, provided that the shell uses it. Check the [Modifying Default Values](#Modifying-Default-Values) section for more info on `sudo` environmental variables and modifying default values.


#### Interactive Shells

The supported interactive shells are: `bash zsh dash sh fish python ruby pry node perl lua5.2 lua5.3 lua5.4 php python2 ksh`

These shells can be used for the `su` and `asu` command types like `sudo --shell=<shell> su` and `sudo --shell=<shell> asu` and also as post shell for `script` command type when the `-i` option is passed like `sudo -si --post-shell=<shell> <core_script>` to start an interactive shell after script commands complete.

The `bash` shell is automatically chosen as the default interactive shell if the `--shell` or `--post-shell` options are not passed to set a specific shell. You can pass the name of a shell listed in the supported shells list like `--shell=zsh` or an absolute path like `--shell=/path/to/zsh`. The `$PREFIX/` and `~/` prefixes are also supported, like `$PREFIX/bin/zsh` or `~/zsh`.


For `perl`, the interactive shell is started using `rlwrap`, which must be installed. Use `pkg install rlwrap` to install.


#### Script Shells

The supported script shells are: `bash zsh dash sh fish python ruby node perl lua5.2 lua5.3 lua5.4 php python2 ksh`

These shells can be used for the `script` command type like `sudo -s --shell=<shell> <core_script>`.

The `bash` shell is automatically chosen as the default script shell if the `--shell` option is not passed to set a specific shell. You can pass the name of a shell listed in the supported shells list like `--shell=zsh` or an absolute path like `--shell=/path/to/zsh`. The `$PREFIX/` and `~/` prefixes are also supported, like `$PREFIX/bin/zsh` or `~/zsh`.
##



### Command Options

The `$PREFIX/` and `~/` prefixes are supported for all command options that take in absolute paths as arguments. The `$PREFIX/` is a shortcut for the termux prefix directory `/data/data/com.termux/files/usr/`. The `~/` is a shortcut for the termux home directory `/data/data/com.termux/files/home/`. Note that if the paths with shortcuts are not surrounded with single quotes, they will expanded by the local shell before being passed to the `sudo` script instead of the `sudo` script manually expanding them. Note that `~/` will expand to the shell or post shell home and not the necessarily the termux home if used inside scripts or the `*-commands` options.

It's the users responsibility to properly quote all arguments passed to command options and also for any values like paths passed inside the arguments, specifically the `*-commands` and `*-options` options, so that whitespace splitting does not occur.

Check [Arguments and Result Data Limits](#Arguments-and-Result-Data-Limits) for details on the max size of arguments that you can pass to `sudo` script, specifically the size of `core_script` and its arguments for the `script` command type.


- `-v | -vv` options can be used to increase the log level of the `sudo` command. Useful to see script progress and what commands will actually be run. You can also use log level `>= DEBUG` with the `--dry-run` option to see what commands will be run without actually executing them. `sudo` uses log levels (`OFF=0`, `NORMAL=1`, `DEBUG=2`, `VERBOSE=3`) and defaults to `NORMAL=1`, but currently does not log anything at `OFF=0` or `NORMAL=1`. Log level can also be set by exporting an `int` in `$SUDO_LOG_LEVEL` between `0-3`, like `SUDO_LOG_LEVEL=3` to set log level to `VERBOSE=3`.


- `-q | --quiet` options can be used to set the log level of the `sudo` command to `OFF=0`.


- `-a` option can be used with the `path` command type to force setting priority to android bin and library paths in `$PATH` and `$LD_LIBRARY_PATH` variables. This can be useful for cases when the `command` is an absolute path but does not exist in the `/system` partition but still needs priority to be set to android paths or if the `command` is a just the basename and you want to run the binary in `/system` partition instead of the one in termux bin path since that will be found first during the search since the `$PATH` variable will be set to priority to android paths.


- `-b` option can be used with the `script` command type mainly for when commands are to be run in a foreground terminal session from plugins. This will simulate double back button press once the `core_script` is complete to go to the last activity, first to close keyboard and second to close terminal session. Use this only for short scripts, otherwise the user may have switched from the terminal session to a different app and back buttons simulation would be done inside that app instead.


- `-B` option can be used with the `script` command type to run the `core_script` in background with `&` (not the entire `sudo` command). This can be used with the `-i` option or even with the `--shell-post-commands` option. The `pid` of the background process will be available in the `$SUDO_SCRIPT_PID` variable. Note that all child processes are killed when `sudo` exits.


- `-c` option can be used with the `script` command type mainly for when commands are to be run in a foreground terminal session from plugins and an interactive shell session needs to be opened after the `core_script` is complete with the `-i` option. This will clear the terminal session once the `core_script` is complete.


- `-d` option can be used with the `script` command type to disable `stdin` for the `core_script`. This will redirect the `stdin` to `/dev/null` and unset the `$PS1` variable so that the `core_script` can detect that the `stdin` is not available and run the script in a non-interactive mode. If the `core_script` doesn't check if `stdin` is available or not and still attempts to read, it will receive nothing as input or may even cause exceptions in some script shells if `I/O` exceptions are not handled properly. Note that when plugin commands are run in a foreground terminal session, then even though keyboard is not shown, `stdin` is available and can be requested by the script which will then open the keyboard.


- `-D` option can be used to disable preserve environment when running `su`, otherwise environment is always preserved.


- `-e` option can be used with the `script` command type to exit early if `core_script` fails due to an exit code other than `0` without running any commands meant to be run after the `core_script` like defined by `-b`, `-c`, `-i`, `-l`, `--post-shell-pre-commands` and `--post-shell-options` command options. If `-B` is passed, then this is ignored.


- `-E` option can be used to `exec` the `su` that runs the final `sudo` `command_type` command. The commands for `--hold` and  `--sleep` options and remount to `ro` commands and any other commands that need to be run after the `sudo` `command_type` command will not be run.


- `-f` option can be used with the `script` command to force usage of `$HOME/.sudo.temp.XXXXXX/sudo_core_script` temp file for storing `core_script` for debugging or if for reason the shell variant doesn't support process substitution and the `sudo` command is automatically trying to use it and is failing. It can also be used to provide a unique temp directory that can be used by the `core_script` which will automatically be deleted after execution.


- `-F` option can be used with the `script` command to consider `core_script` as a path to script file that should be passed to `sudo shell` directly instead of considering it as a script text.


- `-H` option can be used with the `script` command type with the `-i` option to use the same interactive `sudo post shell` home as the script `sudo shell` home. This is useful for situations like if you are passing a custom path for `sudo shell` home and want to use the same for `sudo post shell` home instead of the default home used by the `sudo` script. So instead of running `sudo -si --shell-home=/path/to/home --post-shell-home=/path/to/home <core_script>`, you can simple run `sudo -siH --shell-home=/path/to/home <core_script>`.


- `-i` option can be used with the `script` command to open an interactive shell after the `core_script` completes, optionally specified by `--post-shell` option. If the `--post-shell` option is not passed, then the shell defaults to `bash`.


- `-l` option can be used with the `script` command type mainly for when commands are to be run in a foreground terminal session from plugins. This will simulate home button press once the `core_script` is complete to go to the launcher activity.


- `-L` option will export all the additional paths that already exist in the `$LD_LIBRARY_PATH` variable at the moment `sudo` command is run while running shells, The default paths exported by `sudo` command will still be exported and prefixed before the additional paths. You can also use the `--shell-pre-commands` and `--post-shell-pre-commands` options to manually export the `$LD_LIBRARY_PATH` variable with a different priority as long as it doesn't break execution of the shells.


- `-n` option can be used with the `script` command type to redirect `stderr` to `/dev/null` only for the `core_script` (not the entire `sudo` command). This is a shortcut for `--script-redirect=3`.


- `-N` option can be used with the `script` command type to redirect both `stdout` and `stderr` to `/dev/null` only for the `core_script` (not the entire `sudo` command). This is a shortcut for `--script-redirect=4`.


- `-o` option can be used with the `script` command type to redirect `stderr` to `stdout` only for the `core_script` (not the entire `sudo` command). This is a shortcut for `--script-redirect=0`.


- `-O` option can be used with the `script` command type to redirect `stdout` to `stderr` only for the `core_script` (not the entire `sudo` command). This is a shortcut for `--script-redirect=1`.


- `-p` option sets `path` as the command type for `sudo` and is the default command type.


- `-P` option will export all the additional paths that already exist in the `$PATH` variable at the moment `sudo` command is run while running shells, The default paths exported by `sudo` command will still be exported and prefixed before the additional paths. You can also use the `--shell-pre-commands` and `--post-shell-pre-commands` options to manually export the `$PATH` variable with a different priority as long as it doesn't break execution of the shells.


- `-r` option will parse arguments as per `RUN_COMMAND` intent rules. This will by default replace any comma alternate characters `‚` (`#U+201A`, `&sbquo;`, `&#8218;`, `single low-9 quotation mark`) with simple commas `,` (`U+002C`, `&comma;`, `&#44;`, `comma`) found in any `command_args` for the `path` command type and in `core_script` and any `core_script_args` for the `script` command type. They will also be replaced in the `--hold`, `--post-shell-home`, `--post-shell-pre-commands`, `--post-shell-options`, `--shell-home`, `--shell-pre-commands`, `--shell-post-commands`, `--shell-options`, `--script-name`, `--su-env-options`, `--su-run-options`, `--title` and `--work-dir` command options passed **after** the `-r` option, so ideally `-r` option should be passed before any of them. You can use a different character that should be replaced using the `--comma-alternative` option. Check [Passing Arguments Using RUN_COMMAND Intent](#Passing-Arguments-Using-RUN_COMMAND-Intent) section for why this is may be required.


- `-R` option can be use to enable usage of `root` for searching and validating paths. This can be useful for cases where the `termux` user does not have the read or execute permissions to shell or other paths. Starting new `su` shells for validating paths increases execution time and hence is not done by default.


- `-s` option sets `script` as the command type for `sudo`.


- `-S` option can be used with the `script` command type with the `-i` option to use the same interactive `sudo post shell` as the script `sudo shell` as long as the `sudo shell` exists in the list of supported interactive shells. This is useful for situations like if you are running a `python` script and want to start a `python` interactive shell after the script completes instead of the likely default `bash` shell. So instead of running `sudo -si --shell=python --post-shell=python <core_script>`, you can simple run `sudo -siS --shell=python <core_script>`.


- `--comma-alternative` option can be used to set the comma alternative character to be used for the `-r` option instead of the default.


- `--dry-run` option will enable dry running of the `sudo` script. This will not execute any commands, nor will `rc` files, `history` files or `working directory` passed be created. However, the `sudo shell` home and `$HOME/.sudo.temp.XXXXXX/sudo_core_script` file will still be created if `sudo_core_script` file needs to be created. It's advisable to also pass the `-v` or `-vv` options along with this to see script progress and what commands would actually have been run. Passing `--keep-temp` may also be useful.


- `--export-paths` option can be used to set the additional paths to export in `$PATH` variable, separated with colons `:`. The string passed must not start or end with or contain two consecutive colons `:`.


- `--export-ld-lib-paths` option can be used to set the additional paths to export in `$LD_LIBRARY_PATH` variable, separated with colons `:`. The string passed must not start or end with or contain two consecutive colons `:`.


- `--force-remount-ro` will enable force remount of rootfs `/` and system `/system` partitions back to `ro` mode in `Global` namespace after `sudo` commands complete regardless of if they were mounted as `rw` or `ro` when `sudo` command was run and were mounted as `rw` due to home or working directories in those partitions.


- `--hold[=<string>]` option can be used to make `sudo` script hold the terminal and not exit until the `string` is entered. The `string` can only contain alphanumeric and punctuation characters without newlines specified by `[:alnum:]` and `[:punct:]` bash regex character classes. If only `--hold` is passed, then `sudo` will exit after any key is pressed. This is useful for cases where `sudo` is being run in a foreground terminal session, like from plugins and the terminal closes as soon as the `sudo` exits, regardless of if `sudo` failed or was successful without the user getting a chance to see the output.


- `--hold-if-fail` option can be used with the `--hold` option to only hold if exit code of `sudo` does not equal `0`.


- `--list-interactive-shells` option can be used to display the list of supported [Interactive Shells](#Interactive-Shells) and exit.


- `--list-script-shells` option can be used to display the list of supported [Script Shells](#Script-Shells) and exit.


- `--no-create-rc` option will disable automatic creation of `rc` files for `sudo shell` and `sudo post shell` if they are missing.


- `--no-create-hist` option will disable automatic creation of `history` files for `sudo shell` and `sudo post shell` if they are missing.


- `--no-hist` option will try to disable history loading and saving for `sudo shell` and `sudo post shell` depending on shell capabilities. Not all interactive shells have history support or of disabling it. The history files will also not be created automatically if they are missing.


- `--no-log-args` option can be used with the `path` or `script` command type to disable logging of arguments and `core_script` content when log level is `>= DEBUG`. This is useful in cases where the arguments or `core_script` content is too large and it "hides" the other useful log entries due to terminal session output buffer limitations.


- `--no-remount-ro` will disable remount of rootfs `/` and system `/system` partitions back to `ro` mode in `Global` namespace after `sudo` commands complete if they were mounted as `rw` when `sudo` command was run due to home or working directories in those partitions.


- `--keep-temp` option will disable automatic deletion of the sudo temp directory `$HOME/.sudo.temp.XXXXXX` on exit. This can be used to debug any temp script files created.


- `--post-shell=<shell>` option can be used with the `script` command type to pass the name or absolute path for `sudo post shell` to be used with the `script` command type and the `-i` option.


- `--post-shell-home=<path>` option can be used with the `script` command type to pass an absolute path for the `sudo post shell` home that overrides the default value.


- `--post-shell-options=<options>` option can be used with the `script` command type to set additional options to pass to `sudo post shell` while starting an interactive shell.


- `--post-shell-post-commands=<commands>` option can be used with the `script` command type to set bash commands to be run after the `sudo post shell` exits.


- `--post-shell-pre-commands=<commands>` option can be used with the `script` command type to set bash commands to be run before the `sudo post shell` is started. The commands are run after the commands that are run for the  `--shell-post-commands`, `-b`, `-l` and `-c` options.


- `--post-shell-stdin-string=<string>` ] option can be used with the `script` command type to set the string that should be passed as `stdin` to the `sudo post shell` using process substitution or herestring depending on shell capabilities. Some shells when run in interactive mode may automatically exit after running the commands received through `stdin` or may not even accept strings from `stdin`. This option is used for automated testing.


- `--remove-prev-temp` option can be used with the `script` command type to remove temp files and directories created on previous runs of `sudo` command that may have been left behind due to `sudo` being killed and not cleanly exiting, or Termux crashing or being killed by android `OOM` killer or the phone rebooting as long as the `sudo shell` home is not changed.


- `--script-decode` option can be used with the `script` command type so that the `core_script` passed is considered as a `base64` encoded string that should be decoded and stored in temp file. The temp file path is passed to the script shell. This can be useful to pass a script whose normal decoded form contains non `UTF-8` or binary data which if passed directly as an argument may be discarded by the shell if not encoded first since such data cannot be stored in bash variables. If this is passed, then `-r` option processing will be ignored for the `core_script` but not for any arguments.


- `--script-name` option can be used with the `script` command type to set the filename to use for the `core_script` temp file created in `$HOME/.sudo.temp.XXXXXX/sudo_core_script` directory instead of `sudo_core_script`. The temp file path is passed to the script shell if `-f` or `--script-decode` is passed or if the script shell doesn't support process substitution or if `core_script` passed contained non `UTF-8` or binary data.


- `--script-redirect=<mode/string>` option can be used with the `script` command type to set the redirect mode or string for `stdout` and `stderr` for the `core_script`. The following modes are supported:  

    - `0` redirect `stderr` to `stdout`. This can be used to receive both `stdout` and `stderr` in a synchronized way as `stdout`, like in `%stdout` variable for `Termux:Tasker` plugin for easier processing of result of commands.  

    - `1` redirect `stdout` to `stderr`. This can be used to receive both `stdout` and `stderr` in a synchronized way as `stderr`, like in `%stderr` variable for `Termux:Tasker` plugin for easier processing of result of commands.  

    - `2` redirect `stdout` to `/dev/null`. This can be used to ignore `stdout` output of the `core_script`.  

    - `3` redirect `stderr` to `/dev/null`. This can be used to ignore `stderr` output of the `core_script`.  

    - `4` redirect `stdout` and `stderr` to `/dev/null`. This can be used to ignore `stdout` and `stderr` output of the `core_script`.  

    - `5` redirect `stderr` to `stdout` and `stdout` to `stderr`. This can be used to swap `stdout` and `stderr` output of the `core_script`.  

    - `6` redirect `stderr` to `stdout` and `stdout` to `/dev/null`. This can be used to ignore `stdout` and to receive `stderr` output as `stdout` of the `core_script`.  

    - `*` else it is considered a string that's appended after the `core_script` and its arguments. This can be used for custom redirection, like redirection to a file and possibly used along with the `--shell-pre-commands` option if some prep is required.  

    Note that anything sent to `stdout` and `stderr` outside the `core_script` shell will still be sent to `stdout` and `stderr` and will be received in the `%stdout` and `%stderr` variables for `Termux:Tasker` plugin, so do not ignore them completely while checking for failures.  

    If you are using `SuperSU` and running commands in an interactive shell like from a foreground terminal session, then these options will not work properly. Check [Automatic redirection of stderr to stdout in SuperSU](#automatic-redirection-of-stderr-to-stdout-in-supersu) for more details.  


- `--shell=<shell>` option can be used to pass the name or absolute path for `sudo shell`. For `su` and `asu` command types, this is refers to the interactive shell. For `script` command type, this refers to script shell that should run the `core_script`. For `path` command type, this option is ignored.


- `--shell-home=<path>` option can be used to pass an absolute path for the `sudo shell` home that overrides the default value.


- `--shell-options=<options>` option can be used to set additional options to pass to `sudo shell`. For `su` and `asu` command types, these will be passed while starting an interactive shell. For `script` command type, these will be passed while starting the script shell that will be used to passed the `core_script`. For `path` command type, these options are ignored.


- `--shell-post-commands=<commands>` option can be used with the `script` command type to set bash commands to be run after the `sudo shell` running the `core_script` exits. The commands are run before the commands that are run for the  `-b`, `-l`, `-c` and `--post-shell-pre-commands` options.


- `--shell-pre-commands=<commands>` option can be used to set bash commands to be run before the `sudo shell` is started. For `su`, `asu` and `path` command types, these commands must be simple commands, (preferably one liners) where each command **must** end with a semicolon `;` since they are passed using the `-c` option to `su` that is running a `bash` shell using its `--shell` option. For the `script` command type, these can be more complicated, like a bash script itself, since they are passed to a new `bash` shell in a pseudo file. The commands are run after the `cd` command for `--work-dir` is run.


- `--shell-stdin-string=<string>` ] option can be used with the `su` `asu` and `script` command type to set the string that should be passed as `stdin` to the `sudo shell` using process substitution or herestring depending on shell capabilities. Some shells when run in interactive mode may automatically exit after running the commands received through `stdin` or may not even accept strings from `stdin`. This option is used for automated testing.


- `--sleep=<seconds>` option can be used to make `sudo` script to sleep for `x` seconds before exiting. Seconds can be an integer or floating point number that is passed to the `sleep` command. This is useful for cases where `sudo` is being run in a foreground terminal session, like from plugins and the terminal closes as soon as the `sudo` exits, regardless of if `sudo` failed or was successful without the user getting a chance to see the output.


- `--sleep-if-fail` option can be used with the `--sleep` option to only sleep if exit code of `sudo` does not equal `0`.


- `--su-env-options=<options>` option can be used to set additional options to pass to `su` that set up the `sudo` environment. The `-c` option and `user` argument is not supported. Use `sudo -p su [user]` command instead.


- `--su-run-options=<options>` option can be used to set additional options to pass to `su` that runs the final `sudo` `command_type` command. The `-c` option and `user` argument is not supported. Use `sudo -p su [user]` command instead.


- `--title=<title>` option can be used to set the title for the foreground terminal session, that is shown in the `termux` sidebar.


- `--work-dir=<path>` option can be used to set the absolute path for working directory for the `sudo shell`. The `cd` command is run before the commands passed with `--shell-pre-commands` and `--post-shell-pre-commands` options are run. The directory will be automatically created if missing.
##



### Shell Home

The default `$HOME` directory for `sudo shell` and `sudo post shell` is `/data/data/com.termux/files/home/.suroot`. The `--shell-home` and `--post-shell-home` options or the exported environmental variables `$SUDO_SHELL_HOME` and `$SUDO_POST_SHELL_HOME` can be used to change the default directory. The home directory should ideally be different from the termux home directory to keep `config`, `rc` and `history` files separate for the `root` user and the `termux` user. The home directory should also be owned by the `root` user and have `0700` permission so that `non-root` users cannot access it for security reasons and hence termux home should ideally not be used.

Check the [Modifying Default Values](#Modifying-Default-Values) section for more info on `sudo` environmental variables and modifying default values.

If the home directory is under the termux files directory, then it must not be one of the following directories: `~/.{cache,config,local,termux}` and `$PREFIX/*`.

The home directory is automatically created when `sudo` command is run if it does not exist. The `root:root` ownership and `700` permission is also set to it.

If the home or working directories are in android rootfs `/` partition or android system `/system` partition, then the respective partition is automatically remounted as `rw` in the `Global` namespace when `sudo` command is run and remounted back to `ro` before `sudo` command exits, but only if the partition was mounted as `ro` before `sudo` command was run or `--no-remount-ro` was not passed. The `--force-remount-ro` option can be passed to force remounting to `ro` regardless of partition mount state before `sudo` command was run. For android `>= 10`, do not set home or working directory in rootfs or system partition since `sudo` script will exit with error. In android `>= 10`, rootfs partition is likely a read-only system-as-root `SAR` partition and system partition is likely an `ext4` `dedup` filesystem which cannot be remounted as `rw`.

If the `-E` option is passed or an `exec` is manually done, then remounting back to `ro` will not happen.
##



### Shell RC Files

The following shell `rc` files are used for different shells depending on if `sudo shell` or `sudo post shell` home is different from termux home or shared. The `rc` files are usually unique for different shells.

- If the homes are different, then `sudo` shells and `termux` shells will have different `rc` files, stored in their own homes.

- If the homes are shared and the shell has no `--rc` param or environmental variable for `rc` files, then `sudo` shells and `termux` shells will have to share the same `RCFILE`, implied by `(shared)` and `(hard-coded)` (by the shell) columns, otherwise will be different.

For shells that do not have `rc` files have their columns set to `-`.


|  Shell  | RCFILE (different home) |   RCFILE (shared home)  |    Set Method    |
| ------- |:-----------------------:|:-----------------------:|:----------------:|
| bash    | .bashrc                 | .sudo_bashrc            | --rcfile         |
| zsh     | .zshrc                  | *(shared)*              | $ZDOTDIR         |
| dash    | .dashrc                 | .sudo_dashrc            | $ENV             |
| sh      | .shrc                   | .sudo_shrc              | $ENV             |
| fish    | .config/fish/config.fish| *(shared)*              | $XDG_CONFIG_HOME |
| python  | .pythonrc               | .sudo_pythonrc          | $PYTHONSTARTUP   |
| ruby    | .irbrc                  | *(shared)*              | *(hard-coded)*   |
| pry     | .pryrc                  | *(shared)*              | *(hard-coded)*   |
| node    | -                       | -                       | -                |
| perl    | -                       | -                       | -                |
| lua5.2  | -                       | -                       | -                |
| lua5.3  | -                       | -                       | -                |
| lua5.4  | -                       | -                       | -                |
| php     | php.ini                 | .sudo_php.ini           | -c               |
| python2 | .python2rc              | .sudo_python2rc         | $PYTHONSTARTUP   |
| ksh     | .kshrc                  | .sudo_kshrc             | $ENV             |


The `rc` file parent directory is automatically created when `sudo` command is run if it does not exist. The `root:root` ownership and `700` permission is also set to it.

The `rc` file is automatically created when `sudo` command is run if it does not exist. The `root:root` ownership and `600` permission is also set to it.

The `rc` file parent directory and `rc` file will not be created automatically if `-no-create-rc` is passed.
##



### Shell History Files


The following shell `history` files are used for different shells depending on if `sudo shell` or `sudo post shell` home is different from termux home or shared. The `history` files are usually unique for different shells.

- If the homes are different, then `sudo` shells and `termux` shells will have different `history` files, stored in their own homes.

- If the homes are shared and the shell has no environmental variable for `history` files, then `sudo` shells and `termux` shells will have to share the same `HISTFILE`, implied by `(shared)` and `(hard-coded)` (by the shell) columns, otherwise will be different.

For shells that do not have `history` files have their columns set to `-`. For shells whose history cannot be disabled have their `Disable Method` column set to `(not possible)`.

For `pry` shell, the existing history will still be loaded, but new history will not be saved.


|  Shell  |      HISTFILE (different home)|      HISTFILE (shared home)        |          Set Method         |          Disable Method             |
| ------- |:-----------------------------:|:----------------------------------:|:---------------------------:|:-----------------------------------:|
| bash    | .bash_history                 | .sudo_bash_history                 | $HISTFILE                   | HISTFILE="/dev/null"                |
| zsh     | .zsh_history                  | .sudo_zsh_history                  | $HISTFILE                   | HISTFILE="/dev/null"                |
| dash    | .dash_history                 | .sudo_dash_history                 | $HISTFILE                   | HISTFILE="/dev/null"                |
| sh      | .sh_history                   | .sudo_sh_history                   | $HISTFILE                   | HISTFILE="/dev/null"                |
| fish    | .local/share/fish/fish_history| .local/share/fish/sudo_fish_history| $fish_history               | --private                           |
| python  | .python_history               | *(shared)*                         | *(hard-coded)*              | readline.write_history_file = *None |
| ruby    | .irb_history                  | *(shared)*                         | *(hard-coded)*              | *(not possible)*                    |
| pry     | .pry_history                  | *(shared)*                         | *(hard-coded)*              | Pry.config.history_save = false     |
| node    | .node_history                 | .sudo_node_history                 | $NODE_REPL_HISTORY          | NODE_REPL_HISTORY=""                |
| perl    | .perl_history                 | .sudo_perl_history                 | `rlwrap` --history-filename | --history-filename "/dev/null"      |
| lua5.2  | -                             | -                                  | -                           | -                                   |
| lua5.3  | -                             | -                                  | -                           | -                                   |
| lua5.4  | -                             | -                                  | -                           | -                                   |
| php     | .php_history                  | *(shared)*                         | *(hard-coded)*              | *(not possible)*                    |
| python2 | -                             | -                                  | -                           | -                                   |
| ksh     | .ksh_history                  | .sudo_ksh_history                  | $HISTFILE                   | HISTFILE="/dev/null"                |


The `history` file parent directory is automatically created when `sudo` command is run if it does not exist. The `root:root` ownership and `700` permission is also set to it.

The `history` file is automatically created when `sudo` command is run if it does not exist. The `root:root` ownership and `600` permission is also set to it.

The `history` file parent directory and `history` file will not be created automatically if `-no-create-hist` or `--no-hist` is passed.
##



### Modifying Default Values

Check the [sudo.config](sudo.config) file to see the environmental variables that can be used to change the default values. If the `sudo.config` file exits at `~/.config/sudo/sudo.config`, then `sudo` will automatically source it whenever it is run. It must have `termux` user ownership or be readable by it.

You can download it from the `master` branch and set it up by running the following commands. If you are on an older version, you may want to extract it from its [release](https://github.com/agnostic-apollo/sudo/releases) instead.

```
config_directory="/data/data/com.termux/files/home/.config/sudo"
mkdir -p "$config_directory" && \
chmod 700 -R "$config_directory" && \
curl -L 'https://github.com/agnostic-apollo/sudo/raw/master/sudo.config' -o "$config_directory/sudo.config" && \
chmod 600 "$config_directory/sudo.config"
```

You can use `shell` based text editors like `nano`, `vim` or `emacs` to modify the `sudo.config` file.

`nano "/data/data/com.termux/files/home/.config/sudo/sudo.config"`

You can also use `GUI` based text editor android apps that support `SAF`. Termux provides a [Storage Access Framework (SAF)](https://wiki.termux.com/wiki/Internal_and_external_storage) file provider to allow other apps to access its `~/` home directory. However, the `$PREFIX/` directory is not accessible to other apps. The [QuickEdit] or [QuickEdit Pro] app does support `SAF` and can handle large files without crashing, however, it is closed source and its pro version without ads is paid. You can also use [Acode editor] or [Turbo Editor] if you want an open source app.

Note that the android default `SAF` `Document` file picker may not support hidden file or directories like `~/.config` which start with a dot `.`, so if you try to use it to open files for a text editor app, then that directory will not show. You can instead create a symlink for  `~/.config` at `~/config_sym` so that it is shown. Use `ln -s "/data/data/com.termux/files/home/.config" "/data/data/com.termux/files/home/config_sym"` to create it.


If you use the `bash` shell in termux terminal session, you can optionally export the environmental variables like `$SUDO_SHELL_HOME` and `$SUDO_POST_SHELL_HOME` in the `~/.bashrc` file by adding `export SUDO_SHELL_HOME="/path/to/home"` and `export SUDO_POST_SHELL_HOME="/path/to/home"` lines to it so that they are automatically set whenever you start a terminal session. However, the `~/.bashrc` and `rc` files of other shells will not be sourced if you are running commands from `Termux:Tasker` or `RUN_COMMAND Intent`, and so it is advisable to use the `sudo.config` file instead, which will be sourced in all cases, regardless of how `sudo` is run.

Note that `$SUDO_SHELL_PS1` and `$SUDO_POST_SHELL_PS1` values will not work if `$PS1` variable is overridden in `rc` files in `$PREFIX/etc/` or in `sudo shell` and `sudo post shell` homes. Check [RC File Variables](#rc-file-variables) section for more details.
##



### Examples

If you are using a foreground terminal session, then you must disable the `bash` command completion and history expansion for the current terminal session before running `sudo` commands to pass multi-line arguments by running `bind 'set disable-completion on'; set +H`. Otherwise `bash` will try to auto complete commands and search the history, and you will get prompts like `Display all x possibilities? (y or n)`.


#### `su`

- Drop to an interactive `bash` shell in `superuser (root)` context with priority set to termux bin and library paths with the default configuration.  

`sudo su`  


- Drop to an interactive `python` shell in `superuser (root)` context with priority set to termux bin and library paths.  

`sudo --shell=python --work-dir="~/" su`  


- Drop to an interactive `bash` shell in `superuser (root)` context with priority set to termux bin and library paths with `/.suroot` directory as `sudo shell` home and remount to `ro` disabled before exiting `sudo`. Since the `/.suroot` directory is in rootfs `/` partition, it will automatically be mounted as `rw` when `sudo` command is run.  

`sudo --shell-home="/.suroot" --no-remount-ro su`  


- Drop to an interactive `bash` shell in `superuser (root)` context with priority set to termux bin and library paths with `/.suroot` as the shell home and termux home as the working directory. All paths currently in `$PATH` and `$LD_LIBRARY_PATH` are also exported.  

`sudo -LP --shell-home="/.suroot" --work-dir='~/' su`  


- Drop to an interactive `bash` shell in `superuser (root)` context with priority set to termux bin and library paths, do not store history, export some additional paths in `$PATH` variable, pass additional options to the bash interactive shell starting including a different rc/init file and run some commands before running the bash shell like exporting some variables and running a script. The value of the `--shell-options` option is surrounded with double quotes and the `--init-file` option value passed in it has double quotes escaped to prevent whitespace splitting when its passed to `bash`. The `--shell-pre-commands` option is instead surrounded with single quotes as an example and so doesn't need double quotes escaped but will require single quotes in commands to be escaped. Moreover, each command in the `--shell-pre-commands` option **must** end with a semicolon `;`.  

`sudo --no-hist --export-paths="/path/to/dir1:/path/to/dir2" --shell-options="--noprofile --init-file \"path/to/file\"" --shell-pre-commands='export VARIABLE_1="VARIABLE_VALUE_1"; export VARIABLE_2="VARIABLE_VALUE_2"; /path/to/script;' su`  
##



#### `asu`

- Drop to an interactive `bash` shell in `superuser (root)` context with priority set to android bin and library paths with the default configuration.  

`sudo asu`  
##



#### `path`

- Run `top` command to show top `10` processes of any user.  

`sudo top -m 10 -n 1`  


- Run `ps` command to processes of all users.  

`sudo ps -ef`  


- Run android `dumpsys` command.  

`sudo dumpsys`  


- Run android `dumpsys` command and filter output to show only `termux` related entires.  

`sudo dumpsys | grep -a termux`  
##



#### `script`


##### `bash`

- Pass a `bash` script text surrounded with single quotes that prints the first `2` args to `stdout`. There is normally no need to pass `--shell=bash` since `bash` shell would be the default shell.  

```
sudo -s 'echo "Hi, $1 $2."' "bash" "shell"
```



- Pass a `bash` script text surrounded with single quotes that prints the first `2` args to `stdout` and start an interactive `bash` shell.  

```
sudo -si 'echo "Hi, $1 $2."' "bash" "shell"
```



- Pass a `bash` script text surrounded with single quotes that prints the first `2` args to `stdout` and start an interactive `bash` shell. The script is forcefully stored in a temp file named `sudo_test` in a temp directory `$HOME/.sudo.temp.XXXXXX`, which is also not deleted after `sudo` exits. The title of the terminal session is also set to `sudo_test`.  

```
sudo -sif --keep-temp --script-name="sudo_test" --title="sudo_test" 'echo "Hi, $1 $2."' "bash" "shell"
```



- Pass a path to `bash` script file to the `bash` shell instead of script text, with `2` args and start an interactive `bash` shell.  

```
sudo -siF '~/.termux/tasker/termux_tasker_basic_bash_test' "bash" "shell"
```



- Pass a `bash` script text surrounded with single quotes that prints the first `2` args to `stdout` and run some commands before running the script like exporting some variables. The `--shell-pre-commands` option is surrounded with single quotes and so doesn't need double quotes escaped but will require single quotes in commands to be escaped. Moreover, complex commands can be passed as argument to the `--shell-pre-commands` option, which optionally may not end with a semicolon `;`.  

```
sudo -s --shell-pre-commands='
export VARIABLE_1="VARIABLE_VALUE_1"
export VARIABLE_2="VARIABLE_VALUE_2"
' '
echo "Hi, $1 $2."
echo "VARIABLE_1=\`$VARIABLE_1\`"
echo "VARIABLE_2=\`$VARIABLE_2\`"
' "bash" "shell"
```



- Pass a `bash` script text surrounded with single quotes that prints the first `2` args to `stdout`and start an interactive `python` shell and run some commands before running the script, after running the script but before going to launcher activity, and after going to launcher activity but before starting the interactive `python` shell.  

```
sudo -sil --post-shell="python" --shell-pre-commands='echo "Running script"' --shell-post-commands='echo "Script complete\nGoing to launcher"' --shell-pre-commands='echo "Starting interactive python shell"' 'echo "Hi, $1 $2."' "bash" "shell"
```



- Pass a `bash` script text with process substitution that reads a name from `stdin` and prints it to `stdout`.  

```
sudo -s --shell=bash <(cat <<'SUDO_EOF'
echo 'What is your name?'
read name
echo "Hi, $name."
SUDO_EOF
)
```



- Pass a `bash` script text surrounded with single quotes that reads a name from `stdin` and prints it to `stdout`.  

```
sudo -s --shell=bash '
echo "What is your name?"
read name
echo "Hi, $name."
'
```



- Pass a `bash` script text surrounded with single quotes that reads a name from `stdin` and prints it to `stdout`, where the script itself also contains single quotes.  

```
sudo -s --shell=bash '
echo '\''What is your name?'\''
read name
echo "Hi, $name."
'
```



- Pass a `bash` script text with process substitution that prints the first `2` args to `stdout` and start an interactive shell if only `2` args are received, otherwise exits early with an error without starting the interactive shell afterwards. Additional arguments that will be passed to the `core_script` are passed to `sudo` after the process substitution ends.  

```
sudo -sie --shell=bash <(cat <<'SUDO_EOF'
#if argument count is not 2
if [ $# -ne 2 ]; then
    echo "Invalid argument count '$#' to 'termux_tasker_basic_bash_test'" 1>&2
    echo "$*" 1>&2
    exit 1
fi

echo "\$1=\`$1\`"
echo "\$2=\`$2\`"

exit 0
SUDO_EOF
) "hello," "termux!"
```



- Pass a `bash` script text surrounded with single quotes that redirects `stderr` of the `core_script` to `stdout` so that both `stdout` and `stderr` can be received in a synchronized way as `stdout`, like in `%stdout` variable for `Termux:Tasker` plugin for easier processing of result of commands.  

```
sudo -so 'echo stdout; echo stderr 1>&2'
```
&nbsp;&nbsp;




##### `zsh`

- Pass a `zsh` script text with process substitution that prints the first `2` args to `stdout`.  

```
sudo -s --shell=zsh <(cat <<'SUDO_EOF'
echo "Hi, $1 $2."
SUDO_EOF
) "zsh" "shell"
```



- Pass a `zsh` script text with process substitution that reads a name from `stdin` and prints it to `stdout`.  

```
sudo -s --shell=zsh <(cat <<'SUDO_EOF'
echo "What is your name?"
read name
echo "Hi, $name."
SUDO_EOF
)
```
&nbsp;&nbsp;




##### `fish`

- Pass a `fish` script text with process substitution that prints the first `2` args to `stdout`.  

```
sudo -s --shell=fish <(cat <<'SUDO_EOF'
echo "Hi, $argv[1] $argv[2]."
SUDO_EOF
) "fish" "shell"
```



- Pass a `fish` script text with process substitution that reads a name from `stdin` and prints it to `stdout`.  

```
sudo -s --shell=fish <(cat <<'SUDO_EOF'
echo "What is your name?"
read -p "" name
echo "Hi, $name."
SUDO_EOF
)
```
&nbsp;&nbsp;




##### `python`

Currently, `python` refers to `python3` and `python2` refers to `python2` in termux. Check [Termux Python Wiki](https://wiki.termux.com/wiki/Python) for more information.

- Pass a `python` script text with process substitution that prints the first `2` args to `stdout`.  

```
sudo -s --shell=python <(cat <<'SUDO_EOF'
import sys
print("Hi, %s %s." % (sys.argv[1], sys.argv[2]))
SUDO_EOF
) "python" "shell"
```



- Pass a `python` script text with process substitution that reads a name from `stdin` and prints it to `stdout`.  

```
sudo -s --shell=python <(cat <<'SUDO_EOF'
name = input("What is your name?\n")
print("Hi, %s." % name)
SUDO_EOF
)
```
&nbsp;


*The following "tests" are in solidarity with the `youtube-dl` devs, EFF and the current Github response of the "incident".*

The `youtube-dl` file is actually not a single python script text file but is a [binary file](https://github.com/ytdl-org/youtube-dl/blob/9fe50837c3e8f6c40b7bed8bf7105a868a7a678f/Makefile#L70) containing multiple python files.

- Read `python` script text from a file using `cat` and pass it with process substitution. Passing the data of the `youtube-dl` file to `sudo` script using process substitution will engage automatic `base64` encoding of the data and creation of temp script file. The `youtube-dl` generates its help output based on the named of the its own file, hence `--script-name` is passed, otherwise help with contain `sudo_core_script` entries instead. The current size of the `youtube-dl` binary is over `1MB` and so its data cannot be passed as an argument directly (after `base64` encoding) since [Arguments Data Limits](#Arguments-and-Result-Data-Limits) will cross.  

```
sudo -s --shell=python --script-name="youtube-dl" <(cat "$PREFIX/bin/youtube-dl") --help
```


- Pass a path to `python` script file to the `python` shell instead of script text, with an arg.  

```
sudo -sF --shell=python '$PREFIX/bin/youtube-dl' --help
```


- Read `python` script text from a file using `cat` in a subshell and pass it as an argument. The script size must not cross [Arguments Data Limits](#Arguments-and-Result-Data-Limits). If the script contains binary or non `UTF-8` data, then pipe the output of `cat` to `base64` and also pass the `--script-decode` option.  

```
sudo -s --shell=python "$(cat "$PREFIX/bin/bandcamp-dl")" --help
```

```
sudo -s --script-decode --shell=python "$(cat "$PREFIX/bin/bandcamp-dl" | base64)" --help
```
&nbsp;&nbsp;




##### `ruby`

- Pass a `ruby` script text with process substitution that prints the first `2` args to `stdout`.  

```
sudo -s --shell=ruby <(cat <<'SUDO_EOF'
puts "Hi, " + ARGV[0] + " " + ARGV[1] + "."
SUDO_EOF
) "ruby" "shell"
```



- Pass a `ruby` script text with process substitution that reads a name from `stdin` and prints it to `stdout`.  

```
sudo -s --shell=ruby <(cat <<'SUDO_EOF'
puts "What is your name?"
name = STDIN.gets
name = '' if name.nil?
puts "Hi, " + name.chomp.to_s + "."
SUDO_EOF
)
```
&nbsp;&nbsp;




##### `node`

- Pass a `node` script text with process substitution that prints the first `2` args to `stdout`.  

```
sudo -s --shell=node <(cat <<'SUDO_EOF'
console.log(`Hi, ${process.argv[2]} ${process.argv[3]}.`)
SUDO_EOF
) "node" "shell"
```



- Pass a `node` script text with process substitution that reads a name from `stdin` and prints it to `stdout`.  

```
sudo -s --shell=node <(cat <<'SUDO_EOF'
const readline = require('readline').createInterface({
    input: process.stdin,
    output: process.stdout
})

readline.question(`What is your name?\n`, (name) => {
    console.log(`Hi, ${name}.`)
    readline.close()
})
SUDO_EOF
)
```
&nbsp;&nbsp;




##### `perl`

- Pass a `perl` script text with process substitution that prints the first `2` args to `stdout`.  

```
sudo -s --shell=perl <(cat <<'SUDO_EOF'
print "Hi, ", $ARGV[0], " ", $ARGV[1], ".\n";
SUDO_EOF
) "perl" "shell"
```



- Pass a `perl` script text with process substitution that reads a name from `stdin` and prints it to `stdout`.  

```
sudo -s --shell=perl <(cat <<'SUDO_EOF'
print "What is your name?\n";
$name = <STDIN>;
chomp($name);
print "Hi, ", $name, ".\n";
SUDO_EOF
)
```
&nbsp;&nbsp;




##### `lua5.2`

- Pass a `lua5.2` script text with process substitution that prints the first `2` args to `stdout`.  

```
sudo -s --shell=lua5.2 <(cat <<'SUDO_EOF'
io.write('Hi, ', arg[1], ' ', arg[2], '.\n')
SUDO_EOF
) "lua5.2" "shell"
```



- Pass a `lua5.2` script text with process substitution that reads a name from `stdin` and prints it to `stdout`.  

```
sudo -s --shell=lua5.2 <(cat <<'SUDO_EOF'
io.write('What is your name?\n')
local name = io.read()
io.write('Hi, ', name, '.\n')
SUDO_EOF
)
```
&nbsp;&nbsp;




##### `lua5.3`

- Pass a `lua5.3` script text with process substitution that prints the first `2` args to `stdout`.  

```
sudo -s --shell=lua5.3 <(cat <<'SUDO_EOF'
io.write('Hi, ', arg[1], ' ', arg[2], '.\n')
SUDO_EOF
) "lua5.3" "shell"
```



- Pass a `lua5.3` script text with process substitution that reads a name from `stdin` and prints it to `stdout`.  

```
sudo -s --shell=lua5.3 <(cat <<'SUDO_EOF'
io.write('What is your name?\n')
local name = io.read()
io.write('Hi, ', name, '.\n')
SUDO_EOF
)
```
&nbsp;&nbsp;




##### `lua5.4`

- Pass a `lua5.4` script text with process substitution that prints the first `2` args to `stdout`.  

```
sudo -s --shell=lua5.4 <(cat <<'SUDO_EOF'
io.write('Hi, ', arg[1], ' ', arg[2], '.\n')
SUDO_EOF
) "lua5.4" "shell"
```



- Pass a `lua5.4` script text with process substitution that reads a name from `stdin` and prints it to `stdout`.  

```
sudo -s --shell=lua5.4 <(cat <<'SUDO_EOF'
io.write('What is your name?\n')
local name = io.read()
io.write('Hi, ', name, '.\n')
SUDO_EOF
)
```
&nbsp;&nbsp;




##### `php`

- Pass a `php` script text with process substitution that prints the first `2` args to `stdout`.  

```
sudo -s --shell=php <(cat <<'SUDO_EOF'
<?php
echo "Hi, " . $argv[1] . " " . $argv[2] . ".\n";
SUDO_EOF
) "php" "shell"
```



- Pass a `php` script text with process substitution that reads a name from `stdin` and prints it to `stdout`.  

```
sudo -s --shell=php <(cat <<'SUDO_EOF'
<?php
echo "What is your name?\n";
$name = readline();
echo "Hi, " . $name . ".\n";
SUDO_EOF
)
```
##



### Templates

#### Tasker

- `Tasks`  
    - `XML`  
        Download the [Termux Tasker Plugin Sudo Templates Task XML](templates/plugin_hosts/tasker/Termux_Tasker_Plugin_Sudo_Templates.tsk.xml) and [Termux RUN_COMMAND Intent Sudo Templates Task XML](templates/plugin_hosts/tasker/Termux_RUN_COMMAND_Intent_Sudo_Templates.tsk.xml) files to the android download directory. To download, right-click or hold the `Raw` button at the top after opening a file link and select `Download/Save link` or use `curl` from a termux shell. Then import the downloaded task files into Tasker by long pressing the `Task` tab button in Tasker home and selecting `Import Task`.  

        `curl -L 'https://github.com/agnostic-apollo/sudo/raw/master/templates/plugin_hosts/tasker/Termux_Tasker_Plugin_Sudo_Templates.tsk.xml' -o "/storage/emulated/0/Download/Termux_Tasker_Plugin_Sudo_Templates.tsk.xml"`  

        `curl -L 'https://github.com/agnostic-apollo/sudo/raw/master/templates/plugin_hosts/tasker/Termux_RUN_COMMAND_Intent_Sudo_Templates.tsk.xml' -o "/storage/emulated/0/Download/Termux_RUN_COMMAND_Intent_Sudo_Templates.tsk.xml"`  

    - `Taskernet`  
        Import `Termux Tasker Plugin Sudo Templates Task` from `Taskernet` from [here](https://taskernet.com/shares/?user=AS35m8mXdvaT1Vj8TwkSaCaoMUv220IIGtHe3pG4MymrCUhpgzrat6njEOnDVVulhAIHLi6BPUt1&id=Task%3ATermux+Tasker+Plugin+Sudo+Templates).  
        Import `Termux RUN_COMMAND Intent Sudo Templates Task` from `Taskernet` from [here](https://taskernet.com/shares/?user=AS35m8mXdvaT1Vj8TwkSaCaoMUv220IIGtHe3pG4MymrCUhpgzrat6njEOnDVVulhAIHLi6BPUt1&id=Task%3ATermux+RUN_COMMAND+Intent+Sudo+Templates).  

    Check [Termux Tasker Plugin Sudo Templates Task Info](templates/plugin_hosts/tasker/Termux_Tasker_Plugin_Sudo_Templates.tsk.md) and [Termux RUN_COMMAND Intent Sudo Templates Task Info](templates/plugin_hosts/tasker/Termux_RUN_COMMAND_Intent_Sudo_Templates.tsk.md) files for more info on the tasks.  


Termux needs to be granted `Storage` permission to allow it to access `/storage/emulated/0/Download` directory, otherwise you will get permission denied errors while running commands.
##



### Passing Arguments

The `core_script` or any other arguments passed for all the command types must be preserved in their original form and must be passed as is to `sudo` without any variable expansion or history expansion, etc.

This can be done in two ways, either using single quotes to surround the `core_script` and arguments or passing them with process substitution with a literal `cat` `heredoc`.
&nbsp;

If you are using [Termux:Tasker] plugin to run `sudo` commands, you would need to use single quotes to pass arguments, since it doesn't support process substitution. You would need to install [Termux:Tasker] version `>= 0.5` since argument parsing is broken in older versions. Check the [Passing Arguments Surrounded With Single Quotes](#Passing-Arguments-Surrounded-With-Single-Quotes) section for more details. Check the `Template 2` and `Template 3` of the [Termux Tasker Plugin Sudo Templates Task](#Templates) task for templates on how to use single quotes to pass arguments with Tasker. Basically, just set your script text to the `%core_script` variable with the `Variable Set` action and add any additional command options or arguments to the `%arguments` variable.  
&nbsp;

If you are using a foreground terminal session or scripts to run `sudo` commands, you can use single quotes to pass arguments or use process substitution. If you are using a foreground terminal session, then you must disable the `bash` command completion and history expansion for the current terminal session before running `sudo` command to pass multi-line arguments by running `bind 'set disable-completion on'; set +H`. Check the [Passing Arguments Using Process Substitution](#Passing-Arguments-Using-Process-Substitution) section for more details. Check the [Examples](#Examples) section for templates on how to use process substitution to pass arguments.
&nbsp;

If you are using [RUN_COMMAND Intent] to run `sudo` commands with Tasker using the `TermuxCommand()` function in `Tasker Function` action, you don't need to surround the `core_script` or arguments with single quotes, since arguments are split on a simple comma `,` instead. If your arguments themselves contain simple commas `,` (`U+002C`, `&comma;`, `&#44;`, `comma`), then you must replace them with the comma alternate character `‚` (`#U+201A`, `&sbquo;`, `&#8218;`, `single low-9 quotation mark`) for each argument separately before passing them to the intent action and would also need to pass the `-r` command option to `sudo`. Check the [Passing Arguments Using RUN_COMMAND Intent](#Passing-Arguments-Using-RUN_COMMAND-Intent) section for more details. Check the `Template 2` of the [Termux RUN_COMMAND Intent Sudo Templates Task](#Templates) task for a template on how to replace commas in each argument separately with Tasker.
&nbsp;

If you are using [RUN_COMMAND Intent] to run `sudo` commands with Tasker or other apps using the `am` command, like using the `Run Shell` action in Tasker, you need to surround all your arguments, like the `core_script` and all other arguments with single quotes when passing them to the `com.termux.RUN_COMMAND_ARGUMENTS` string array extra after you have escaped all the single quotes in the final value, since otherwise it may result in incorrect quoting if the arguments themselves contain single quotes. However, due to the string array extra, the arguments are still split on a simple comma `,` so if your arguments themselves contain simple commas `,` (`U+002C`, `&comma;`, `&#44;`, `comma`), then you would also have to replace them with the comma alternate character `‚` (`#U+201A`, `&sbquo;`, `&#8218;`, `single low-9 quotation mark`) for each argument separately before passing them as the argument to the extra and would also need to pass the `-r` command option to `sudo`. Check the [Passing Arguments Using RUN_COMMAND Intent](#Passing-Arguments-Using-RUN_COMMAND-Intent) section for more details. Check the `Template 3` of the [Termux RUN_COMMAND Intent Sudo Templates Task](#Templates) task for a template on how to replace commas in each argument separately and also escape single quotes in all the arguments with Tasker.
&nbsp;

Note that for [RUN_COMMAND Intent], any arguments passed to any command options or the main arguments to `sudo` should also **not** be surrounded with single or double quotes to prevent whitespace splitting in the intent action, like done for usage with `Termux:Tasker` plugin since splitting will occur on simple comma characters instead. Check the `Template 4` of the [Termux RUN_COMMAND Intent Sudo Templates Task](#Templates) task for a template for this.
&nbsp;&nbsp;



##### Passing Arguments Surrounded With Single Quotes

Any argument surrounded with single quotes is considered a literal string and variable expansion is not done. However, if an argument itself contains single quotes, then they will need to be escaped properly. You can escape them by replacing all single quotes `'` in an argument value with `'\''` **before** passing the argument surrounded with single quotes. So an argument surrounded with single quotes that would have been passed like `'some arg with single quote ' in it'` will be passed as `'some arg with single quote '\'' in it'`. This is basically 3 parts `'some arg with single quote '`, `\'` and `' in it'` but when processed, it will be considered as one single argument with the value `some arg with single quote ' in it` that is passed to `sudo`.

For `Tasker`, you can use the `Variable Search Replace` action on an `%argument` variable to escape the single quotes. Set the `Search` field to one single quote `'`, and enable `Replace Matches` toggle, and set `Replace With` field to one single quote, followed by two backslashes, followed by two single quotes `'\\''`. The double backslash is to escape the backslash character itself.

Escaping single quotes while running commands in a foreground terminal session will be much harder if there are many single quotes, same would apply for double quote surrounded strings, so use process substitution instead, check below.

The format is the following

```
sudo -s '
<core_script>
' '[some arg1]' '[some arg2]'
```
&nbsp;&nbsp;



##### Passing Arguments Using Process Substitution

[Process Substitution] can be used to pass the `core_script` and `core_script_args` for the `script` command type and to pass the `command_args` for the `path` command type when running `sudo` from a foreground terminal session or from a script.

The following is the format for passing `core_script` text.

```
sudo -s <(cat <<'SUDO_EOF'
<core_script>
SUDO_EOF
)
```

The following is the process substitution part

```
<(

)
```

Inside it there is a `cat` [Here Document]. The script text should start after a newline after the `'SUDO_EOF'` part. You can type anything as script text other than `SUDO_EOF` which when read, ends the script. The ending `SUDO_EOF` should be alone on a separate line. The starting `'SUDO_EOF'` is surrounded with single quotes so that script is considered a literal string and variable expansion, etc doesn't happen.

```
cat <<'SUDO_EOF'

SUDO_EOF
```

Basically any text you type inside the `cat` heredoc will be passed to the process substitution which will create a temporary file descriptor for it in `/proc/self/fd/<n>` and pass the path to `sudo` script so that it can read the argument text from it.

You can also read text from an existing script file using `cat` and pass that to `sudo` like the following

```
sudo -s <(cat "~/some-script")
```
&nbsp;&nbsp;



##### Passing Arguments Using RUN_COMMAND Intent

To use [RUN_COMMAND Intent] that has arguments working properly, you need to install Termux version `>= 0.100` and Tasker version `>= 5.9.4.beta`. However, leading and trailing whitespaces from arguments will be removed for Tasker version `< 5.11.1.beta` if you are using `TermuxCommand()` function, so its advisable to use a higher version or use `am` command instead.
&nbsp;

If you are using the `am` command, the format is `am startservice --user 0 -n com.termux/com.termux.app.RunCommandService -a com.termux.RUN_COMMAND --es com.termux.RUN_COMMAND_PATH '<path>' --esa com.termux.RUN_COMMAND_ARGUMENTS '<one_or_more_args_seperated_with_commas>' --es com.termux.RUN_COMMAND_WORKDIR '/data/data/com.termux/files/home' --ez com.termux.RUN_COMMAND_BACKGROUND 'false'`
&nbsp;

If you are using the `TermuxCommand()` function, the format is `TermuxCommand(path,<one_or_more_args_seperated_with_commas>,workdir,background)`. The args can be of any count, each separated with a simple comma `,`. The only condition is that in the list the first must be `path` and the last two must be `workdir` and `background` respectively.
&nbsp;

For both the `--esa com.termux.RUN_COMMAND_ARGUMENTS` string array extra and the `TermuxCommand()` function, if you want to pass an argument that itself contains a simple comma `,` (`U+002C`, `&comma;`, `&#44;`, `comma`), it must be escaped with a backslash `\,` so that the  argument isn't split into multiple arguments. The only problem is that, the arguments received by the script being executed will contain `\,` instead of `,` since the reversal isn't done as described in the [am command source](https://android.googlesource.com/platform/frameworks/base/+/21bdaf1/cmds/am/src/com/android/commands/am/Am.java#572) while converting to a string array. Tasker uses the same method. There is also no way for the `am` command or the script to know whether `\,` was done to prevent arguments splitting or `\,` was a literal string naturally part of the argument.

```
// Split on commas unless they are preceeded by an escape.
// The escape character must be escaped for the string and
// again for the regex, thus four escape characters become one.
String[] strings = value.split("(?<!\\\\),");
intent.putExtra(key, strings);
```
&nbsp;

`sudo` uses an alternative method to handle such conditions. If an argument contains a simple comma `,`, then instead of escaping them with a backslash `\,`, replace all simple commas with the comma alternate character `‚` (`#U+201A`, `&sbquo;`, `&#8218;`, `single low-9 quotation mark`). This way argument splitting will not be done. You can pass the `-r` option to `sudo` which will then parse arguments as per `RUN_COMMAND` intent rules to replace all the comma alternate characters back to simple commas. It would be unlikely for the `core_script` or the arguments to naturally contain the comma alternate characters for this to be a problem. Even if they do, they might not be significant for any script logic. If they are, then you can set a different character that should be replaced, with the `--comma-alternative` option. The `-r` and `--comma-alternative` options should ideally be the first options passed so that `sudo` replaces the alternative comma characters from all arguments passed after the options.

For `Tasker` use the `Variable Search Replace` action on an `%argument` variable to replace the simple comma characters. Set the `Search` field to one simple comma `,`, and enable `Replace Matches` toggle, and set `Replace With` field to `%comma_alternative` where the `%comma_alternative` variable must contain the comma alternate character `‚`.
##



### Issues


##### Automatic redirection of stderr to stdout in SuperSU

In `SuperSU` `v2.82` for the `script` command type, if `stdin` is available like running `su` in an interactive shell like from a foreground terminal session, then it automatically redirects `stderr` of commands to `stdout`, specially affecting the `--script-redirect` and related command options. However, if commands are run in a non-interactive shell, in the background, like from `Termux:Tasker` plugin, then `stdout` and `stderr` streams behave normally and are separate. This can be confirmed by running `(su -c 'echo 1 1>&2' 2>/dev/null)` and `(exec <&-; su -c 'echo 1 1>&2' 2>/dev/null)` in a terminal session. In the former case, `1` is still printed on the screen even though `stderr` is redirected to `/dev/null`. The later case closes the `stdin` file descriptor which makes `su` assume its running non-interactively. Running `(su -c 'echo 1 1>&2' 1>/dev/null)` also suppresses printing since it redirects `stdout` to `/dev/null` instead. Running `(bash -c 'echo 1 1>&2' 2>/dev/null)` works normally. Reopening `stdin` with hacks, inside the `su` shell doesn't work either for a few reasons, including that `stderr` redirection to `stdout` starts happening again. This seems to be an issue of the [libsuperuser](https://github.com/Chainfire/libsuperuser/blob/v1.1.0/libsuperuser/src/eu/chainfire/libsuperuser/Shell.java) itself or how the `su` binary handles streams internally and might not be solvable but if you have a solution that can be used to prevent automatic redirection, please report it.

This does not affect usage with `Termux:Tasker` in background mode. This does not apply to `Magisk`, at least the currently latest version `v21.1`. However, this may apply to other `su` implementation.
&nbsp;&nbsp;



##### su -c support

The `sudo` script requires the `-c` command option support by the `su` binary. The `su` that comes with the android studio `avd` does not support it. Other `su` implementations may not support it either.

Moreover, linux distros removed support for starting interactive shells with the `su -c` command. Check [debian su manpage](https://manpages.debian.org/testing/login/su.1.en.html). The `-c` command option info specifies that `The executed command will have no controlling terminal. This option cannot be used to execute interactive programs which need a controlling TTY.`. For more info, check [debian bug report #628843](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=628843) and [ubuntu CVE-2005-4890](https://ubuntu.com/security/CVE-2005-4890). However, this likely does not apply to android `su` implementations, at least does not apply for `SuperSU` and `Magisk` currently, so `sudo` should work fine, at least on those.
##



### Worthy Of Note


#### RC File Variables

If you don't know what `$PATH`, `$LD_LIBRARY_PATH` and `$PS1` variables or `rc` files are or don't care to find out, then just run the commands below so that `sudo` works properly, otherwise read the details below. Ignore `No such file or directory` errors when running the commands.

```
sed -i'' -E 's/^(PS1=.*)$/\(\[ -z "\$PS1" \] \|\| \[\[ "\$PS1" == '\''\\s-\\v\\\$ '\'' \]\]\) \&\& \1/' "/data/data/com.termux/files/usr/etc/bash.bashrc"
sed -i'' -E 's/^(PS1=.*)$/\(\[ -z "\$PS1" \] \|\| \[\[ "\$PS1" == '\''%m%# '\'' \]\]\) \&\& \1/' "/data/data/com.termux/files/usr/etc/zshrc"
su -c "rm \"/data/data/com.termux/files/home/.suroot/.bashrc\""
```
&nbsp;


`rc` files, short for `run commands`, are shell specific files that are run or sourced whenever an interactive shell is started to define variables and functions etc.

Make sure in your `rc` files like `~/.suroot/.bashrc` file for `bash` (where `~/.suroot` is the `sudo shell` home), the `$PATH`, `$LD_LIBRARY_PATH` and `$PS1` variables and any other variables exported by `sudo` are not overridden, **otherwise the `sudo` commands will not work properly**, since `sudo` exports its own custom variable values which will get overridden when `rc` files are sourced by any new shell started. You can run commands like `sudo -v --dry-run --shell=bash su` to see what variables are normally exported for a given shell by `sudo`.
&nbsp;


##### `$PATH` or `$LD_LIBRARY_PATH`

Check the [PATH and LD_LIBRARY_PATH Priorities](#path-and-ld_library_path-priorities) section for more info on what these variables are.

For the `$PATH` or `$LD_LIBRARY_PATH` variables, either remove their variable set lines completely or if necessary only append any required paths to the existing variable values like `export PATH="$PATH:/dir1:/dir2"` and `export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/dir1:/dir2"` instead of overriding them with `export PATH="/dir1:/dir2"` and `export LD_LIBRARY_PATH="/dir1:/dir2"` in your `rc` files.
&nbsp;


##### `$PS1`

The `$PS1` variable, short for [Prompt String 1], defines the characters you see at the start of the "line" when typing commands in shells running interactively like `bash` or `zsh`. For `termux` `bash` shell, this defaults to `$ `. For `termux` `zsh` shell, this defaults to `% `.

If you want to allow `sudo` to set its own default `$PS1` value `# ` or the one set with the `$SUDO_SHELL_PS1` or `$SUDO_POST_SHELL_PS1` variables in the `sudo.config` file, then **make sure the `$PS1` value is not overridden by the shell `rc` files, otherwise you will not be able to easily tell the difference between whether you are running a shell via `sudo` or normally as the `termux` user when you run commands like `sudo su`.** You can however check the value of `$SHLVL` to see the nested shell level, run `printenv | grep SHLVL`.

1. The `rc` files in `$PREFIX/etc/` are sourced first whenever shells are started by `sudo`. The `$PS1` value is set and exported by `sudo` before new shells are started, however, the value will get replaced if its overridden by the `rc` files when they are sourced during startup of the new shell.  

    - `bash` shell currently uses the `$PREFIX/etc/bash.bashrc` file, in which it sets the default value of `$PS1` to `\$ ` or `\[\e[0;32m\]\w\[\e[0m\] \[\e[0;97m\]\$\[\e[0m\] ` in recent versions. You need to replace the line `PS1='\$ '` or `PS1=<default>` with the conditional `([[ -z "$PS1" ]] || [[ "$PS1" == '\s-\v\$ ' ]]) && PS1=<default>` so that `$PS1` is only set if its not already set or is set to the default value internally used by `bash`. You can run the command `sed -i'' -E 's/^(PS1=.*)$/\(\[ -z "\$PS1" \] \|\| \[\[ "\$PS1" == '\''\\s-\\v\\\$ '\'' \]\]\) \&\& \1/' "/data/data/com.termux/files/usr/etc/bash.bashrc"` to automatically do it or you can do it manually by running `nano "/data/data/com.termux/files/usr/etc/bash.bashrc"`.  

    - `zsh` shell currently uses the `$PREFIX/etc/zshrc` file, in which it sets the default value of `$PS1` to `%# `. You need to replace the line `PS1='%# '` or `PS1=<default>` with the conditional `([[ -z "$PS1" ]] || [[ "$PS1" == '%m%# ' ]]) && PS1=<default>` so that `$PS1` is only set if its not already set or is set to the default value internally used by `zsh`. You can run the command `sed -i'' -E 's/^(PS1=.*)$/\(\[ -z "\$PS1" \] \|\| \[\[ "\$PS1" == '\''%m%# '\'' \]\]\) \&\& \1/' "/data/data/com.termux/files/usr/etc/zshrc"` to automatically do it or you can do it manually by running `nano "/data/data/com.termux/files/usr/etc/zshrc"`.  

2. The `rc` files in `~/.suroot` (default `sudo shell` home) are also sourced afterwards whenever shells are started by `sudo`. They must also not override the `$PS1` value. However, if you want to set a custom value in them for usage outside `sudo`, then you can add conditionals to override `$PS1` only if its not already set or is set to the default `termux` value set by `$PREFIX/etc/*` `rc` files.  

    - `bash` shell default `rc` file set by `sudo` is `~/.suroot/.bashrc`. You can for example set `$PS1` to `£ ` by adding the line `([[ -z "$PS1" ]] || [[ "$PS1" == '\$ ' ]]) && PS1='£ '` to it, where `'\$ '` is the default value for `PS1` in `$PREFIX/etc/bash.bashrc` file.  

    - `zsh` shell default `rc` file set by `sudo` is `~/.suroot/.zshrc`. You can for example set `$PS1` to `£ ` by adding the line `([[ -z "$PS1" ]] || [[ "$PS1" == '%# ' ]]) && PS1='£ '` to it, where `'%# '` is the default value for `PS1` in `$PREFIX/etc/zshrc` file.  

This will ensure that the exported `$PS1` variables will not be overridden by `rc` files for `bash` and `zsh`. For other shells that may use the `$PS1` variable, you can use similar conditionals in their `rc` files.
&nbsp;


##### `termux-sudo by st42`
If you were **previously using [termux-sudo by st42]**, then it would have automatically created the `~/.suroot/.bashrc` file with entries like the following. You should either remove those lines if you haven't exported custom values yourself or remove the file entirely if you haven't made changes to it yourself.

```
export LD_LIBRARY_PATH=$PRE/usr/lib
export PATH=/data/data/com.termux/files/usr/bin:/data/data/com.termux/files/usr/bin/applets:/system/xbin:/system/bin
export PS1="# "
```

To remove the `~/.suroot/.bashrc` file, run `su -c "rm \"/data/data/com.termux/files/home/.suroot/.bashrc\""` command.

To edit the `~/.suroot/.bashrc` file, run `su -c "nano \"/data/data/com.termux/files/home/.suroot/.bashrc\""` command. Make changes as advised above, then press `Ctrl+o` and then `Enter` to save and `Ctrl+x` to exit. You can also open the file with a text editor app with root support like [QuickEdit] or [QuickEdit Pro].
&nbsp;&nbsp;



#### Arguments and Result Data Limits

There are limits on the arguments size you can pass to commands or the full command string length that can be run, which is likely equal to `131072` bytes or `128KB` for an android device defined by `ARG_MAX` but after subtracting shell environment size, etc, it will roughly be around `120-125KB` but limits may vary for different android versions and kernels. You can check the limits for a given termux session by running `true | xargs --show-limits`. If you exceed the limit, you will get exceptions like `Argument list too long`. You can manually cross the limit by running something like `$PREFIX/bin/echo "$(head -c 131072 < /dev/zero | tr '\0' 'a')" | tr -d 'a'`, use full path of `echo`, otherwise the `echo` shell built-in will be called to which the limit does not apply since `exec` is not done.

Moreover, exchanging data between `Tasker` and `Termux:Tasker` is done using [Intents](https://developer.android.com/guide/components/activities/parcelables-and-bundles), like sending the command and receiving result of commands in `%stdout` and `%stderr`. However, android has limits on the size of *actual* data that can be sent through intents, it is roughly `500KB` on android `7` but may be different for different android versions.

Basically, make sure any data/arguments you pass to `sudo` script directly on the shell or through scripts or using the `Termux:Tasker` plugin or [RUN_COMMAND Intent] intent is less than `120KB` (or whatever you found) and any expected result sent back if using the `Termux:Tasker` plugin is less than `500KB`, but best keep it as low as possible for greater portability. If you want to exchange an even larger data between tasker and termux, use physical files instead.

The argument data limits also apply for the [RUN_COMMAND Intent] intent.
&nbsp;&nbsp;



#### PATH and LD_LIBRARY_PATH Priorities

The word executable will be used henceforth for binaries, scripts and any other executable files.

`Termux` executables currently exist at `/data/data/com.termux/files/usr/bin` and `/data/data/com.termux/files/usr/bin/applets` `Termux` libraries exist at `/data/data/com.termux/files/usr/lib`.

`Android` executables normally exist at `/system/bin` and/or `/system/xbin` `Android` libraries exist at `/system/lib` and/or `/system/lib64`.

When `sudo su` commands is run, then the termux executables paths are prepended to android executables paths in the `$PATH` variable. The termux library paths are prepended to android library paths in the `$LD_LIBRARY_PATH` variable. This gives priority to termux paths.

When `sudo asu` commands is run, then the android executables paths are prepended to termux executables paths in the `$PATH` variable. The android library paths are prepended to termux library paths in the `$LD_LIBRARY_PATH` variable. This gives priority to android paths.

The `$PATH` variable sets the paths to search for executables when commands are executed. The `$LD_LIBRARY_PATH` variable sets the paths to search for libraries for dynamic linking required by the commands that are executed. The path that appears first in both the variables is searched first and if the required binary, executable or library is found, that is the one thats used without looking further.
&nbsp;

There are a few important things to consider when using `sudo` with termux.

A executable that you want to run may exist in both termux and android executable paths but you may want to run a specific one. If you want to run the termux one instead of the the android one then run `sudo su` command and then run the command to run the executable. If you want to run the android one instead of the the termux one then run `sudo asu` command and then run the command to run the executable. However in both cases, if you write the absolute path of the executable instead of just writing its basename, the executable at the path you wrote will be executed even if the other ones path exist before in the `$PATH` variable.

Another thing to consider is that dynamic library linking errors may occur when executables try to link to the wrong library. Executables should be linked with libraries they are compatible with and that define all the needed functions needed by the executable. Executables that exist in termux executable path should ideally be linked with libraries that exist in termux library path. Executables that exist in android executable path should ideally be linked with libraries that exist in android library path.

When an executable is run, the paths in the `$LD_LIBRARY_PATH` variable are searched for the library that is required and whichever matching library is found first is used, even if that library is not compatible with the executable. If the library is indeed incompatible a linking error occurs with errors that may include words like `CANNOT LINK EXECUTABLE` and `cannot locate symbol some_symbol referenced by /lib....`.

So if an executable in android executable path tries to link with a library in termux library paths to which it is incompatible with, then a linking error will occur. This is likely to happen with some executables including the android `dumpsys` or `input` binaries among others. A linking error may occur the other way around too, when a termux executable tries to link with libraries in android library path. To prevent these linking error from occurring in most situations, separate `sudo su` and `sudo asu` commands exist, which set the correct order of paths in the `$PATH` and `$LD_LIBRARY_PATH` variables so that normally termux executables are linked with termux libraries and android executables are linked with android libraries whenever either command is run.
&nbsp;

However, another way to automatically prioritize android libraries is by running the `path` command type with `sudo <command>`. If the `command` exists in the `/system` partition, then android library paths are prepended to termux library paths in the `$LD_LIBRARY_PATH` variable automatically. An absolute path is not needed to be passed for this to work as the `$PATH_TO_EXPORT` is automatically searched. This is helpful for situations when you are already in a `sudo su` shell and do not want to shift to the `sudo asu` shell or unset `$LD_LIBRARY_PATH` to run just one executable in android executables paths.

You can also use the `tpath` and `apath` functions if they are defined in the `rc` file of your interactive shell to shift priorities.

Normally `sudo su` will work fine without problem when dropping to `sudo` shell. But if you want to specifically run an executable in the android executable paths instead of the one in termux executable paths or are getting linking errors when running android executables with `sudo su`, then try using `sudo asu` and then running the required command or use `sudo <command>` or `sudo -a <command>`.



#### `tpath` and `apath` functions

For the shells `bash zsh dash sh fish ksh`, additional functions named `tpath` and `apath` are added to their `rc` files if the `sudo` script creates the `rc` files, they are not added otherwise. You can call these functions to set priorities from inside interactive shell sessions only when running `sudo su`, `sudo asu` or `sudo -is <core_script` commands, since they depend on some environmental variables set by the `sudo` script and are not hard-coded in case of future changes.

The `tpath` function will set priority to termux bin and library paths in `$PATH` and `$LD_LIBRARY_PATH` variables.

The `apath` function will set priority to android bin and library paths in `$PATH` and `$LD_LIBRARY_PATH` variables.

The functions allows the users to quickly switch priorities without having to switch between `sudo su` and `sudo asu` shells.

If you already have an existing `rc` file for your shell like `~/.suroot/.bashrc` and want to add the functions to it. Just temporarily move (not copy) the file to somewhere else and run `sudo su` command with the optional `--shell` option, then copy the functions from the new `rc` file created by `sudo` to your old file, then remove the new file and move the old file back.



#### `export` and `unset` functions

For the `fish` shell, additional functions named `export` and `unset` are also added to the `rc` files if the `sudo` script creates its `rc` file, they are not added otherwise. The functions port the `bash` `export var=value` and `unset var` functionality respectively.
##



### Tests

Check the [sudo_tests](tests/sudo_tests) script to run automated tests for the `sudo` script command types, options and shells. Usage instructions are inside the script. There are more examples for running `sudo` inside the `sudo_tests` script that can be used by users, although may not be too user friendly to view or understand.
##



### FAQs And FUQs

Check [FAQs_And_FUQs.md](FAQs_And_FUQs.md) file for the **Frequently Asked Questions(FAQs)** and **Frequently Unasked Questions(FUQs)**.
##



### Changelog

Check [CHANGELOG.md](CHANGELOG.md) file for the **Changelog**.
##



### Contributions

`-`
##



### Credits

- [termux-sudo by st42]
- [tsu by cswl]
##



### Donations

- To donate money to support me, you can visit [here](https://github.com/agnostic-apollo/agnostic-apollo/blob/main/Donations.md) for more info.
##

[Tasker App]: https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm
[Termux App]: https://github.com/termux/termux-app
[Termux:Tasker]: https://github.com/termux/termux-tasker
[QuickEdit]: https://play.google.com/store/apps/details?id=com.rhmsoft.edit
[QuickEdit Pro]: https://play.google.com/store/apps/details?id=com.rhmsoft.edit.pro
[Acode editor]: https://github.com/deadlyjack/code-editor
[Turbo Editor]: https://github.com/vmihalachi/turbo-editor

[SuperSU]: https://forum.xda-developers.com/t/beta-2017-10-01-supersu-v2-82-sr5.2868133/
[Magisk]: https://github.com/topjohnwu/Magisk

[tudo]: https://github.com/agnostic-apollo/tudo
[RUN_COMMAND Intent]: https://github.com/termux/termux-app/blob/master/app/src/main/java/com/termux/app/RunCommandService.java
[termux-sudo by st42]: https://gitlab.com/st42/termux-sudo
[tsu by cswl]: https://github.com/cswl/tsu

[Process Substitution]: https://en.wikipedia.org/wiki/Process_substitution
[Here Document]: https://en.wikipedia.org/wiki/Here_document
[Prompt String 1]: https://www.cyberciti.biz/tips/howto-linux-unix-bash-shell-setup-prompt.html
