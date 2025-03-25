---
page_ref: "@ARK_PROJECT__VARIANT@/agnostic-apollo/sudo/docs/@ARK_DOC__VERSION@/developer/guides/su-sub-process-communication/index.md"
---

# `su` Sub Process Communication Guide

<!-- @ARK_DOCS__HEADER_PLACEHOLDER@ -->

The following guide details how the communication between `su` sub processes and the main `sudo` script process happens and related issues.

### Contents

- [Send Data To `su` process](#send-data-to-su-process)
- [Receive Data From `su` process](#receive-data-from-su-process)
- [Send And Receive Data From `su` process](#send-and-receive-data-from-su-process)
- [Additional Rationale](#additional-rationale)

---

&nbsp;





# Send Data To `su` process

Data like `sudo -s <command>` (`$SUDO_SCRIPT_COMMAND_TO_RUN`) cannot be passed to the bash shell inside `su` using process substitution directly since the `self` in `proc/self/fd/#` passed by process substitution is for the `sudo` script process and not for the `bash` process inside `su`. So when bash shell inside `su` tries to read from it, then there will be errors like `/proc/self/fd/63: No such file or directory`.

```shell
/sbin/su --shell="${TERMUX__PREFIX:-$PREFIX}/bin/bash" --preserve-environment -c "bash --noprofile --norc" <(echo 'echo 1')`
bash: /proc/self/fd/63: No such file or directory
```

Hence, we manually create a file descriptor and pass its path to `su` or to be more specific to the `bash` shell for the `path` and `script` commands. The method that is used for passing data to `su` using file descriptors in `sudo` script is the following.

1. Get an unsed file descriptor for the current `sudo` script process and `fd` path by calling `sudo_get_unsed_file_descriptor_and_path()`. The fd path is manually set to `/proc/$BASHPID/fd/<unsed_fd_number>` since that is where the `fd` are created in Android. `$$` returns pid of current process, but preferable `$BASHPID` should be used as `$$` will give wrong results in subshells. Normally unused fd number start at `3`, and its path will be `/proc/$BASHPID/fd/3`.
2. Then run something like `eval "exec $fd<" <(printf "%s" "string")`, where `$fd` expands to `3`.
3. This will create a pseudo pipe file descriptor and attach `stdout` of `printf` to the file descriptor `3`. This basically creates the file descriptor and "saves" the output of `printf` before `su` is started so `/proc/self/fd` issues do not occur.
4. We pass this path to whatever requires it inside `su`, like `bash` or `cat` for it to read. This is done by hardcoding path in the command run with `su` or exporting a variable for the path. The path will of course be for current `sudo` script `bash` process and not for the `bash` process inside `su` because of `$$`/`$BASHPID`, but since `bash` is being run with root it will be able to read from it. The file descriptor will be a read only stream like in pipes. seeking forward/backward is not a possibility and once data has been read by another process, it is gone and cannot be read again from the start.

Credits and details: https://stackoverflow.com/a/20018118/14686958 by Jo So

A basic version not dependent on `sudo` script functions/variables is the following. Errors and `trap` are not handled.

```shell
(
get_unsed_file_descriptor() {
    local __fd=2 max=256; while ((++__fd < max)); do ! true <&"$__fd" && break; done 2>/dev/null || return $?; printf -v "$1" "%s" "$__fd"
}

# Get first unsued file descriptor and export its path for `su`.
get_unsed_file_descriptor "SUDO_SU_SUB_PROCESS__SEND_COMM__FD"
export SUDO_SU_SUB_PROCESS__SEND_COMM__FD_PATH="/proc/$BASHPID/fd/$SUDO_SU_SUB_PROCESS__SEND_COMM__FD"

send_data="I think, therefore I am"

# Use process substitution to start a process that writes send_data to
# `SUDO_SU_SUB_PROCESS__SEND_COMM__FD` and then exits.
eval "exec $SUDO_SU_SUB_PROCESS__SEND_COMM__FD<" <(printf '%s' "$send_data")

# Start an `su` process in a subshell and read send_data from
# `SUDO_SU_SUB_PROCESS__SEND_COMM__FD` with `cat`
# and process it, like writing to a file or optionally printing
# processed data back to stdout.
su_output=$(unset LD_PRELOAD; su --preserve-environment -c 'send_data="$(cat "$SUDO_SU_SUB_PROCESS__SEND_COMM__FD_PATH")" && echo "${send_data%,*}"' 2>&1 < /dev/null)
echo "PROCESSED_DATA:'$su_output'"
#I think

# Close send fd.
exec {SUDO_SU_SUB_PROCESS__SEND_COMM__FD}<&-
)
```

An advance version dependent on `sudo` script functions/variables is the following. Errors and trap are handled.

```shell
sudo_get_unsed_file_descriptor_and_path SUDO_SU_SUB_PROCESS__SEND_COMM__FD SUDO_SU_SUB_PROCESS__SEND_COMM__FD_PATH "to write data" || return $?
export SUDO_SU_SUB_PROCESS__SEND_COMM__FD_PATH

send_data="I think, therefore I am"

# Use process substitution to start a process that writes send_data to
# `SUDO_SU_SUB_PROCESS__SEND_COMM__FD` and then exits.
eval "exec $SUDO_SU_SUB_PROCESS__SEND_COMM__FD<" <(printf '%s' "$send_data")

# Start an `su` process in a subshell and read send_data from
# `SUDO_SU_SUB_PROCESS__SEND_COMM__FD` with `cat`
# and process it, like writing to a file or optionally printing
# processed data back to stdout.
sudo_unset_pre_su_variables
su_output=$($SU_ENV_COMMAND 'send_data="$(cat "$SUDO_SU_SUB_PROCESS__SEND_COMM__FD_PATH")" && echo "${send_data%,*}"' 2>&1 < /dev/null)
sudo_set_post_su_variables
echo "PROCESSED_DATA:'$su_output'"
#I think

# Close send fd.
exec {SUDO_SU_SUB_PROCESS__SEND_COMM__FD}<&-
```

---

&nbsp;





# Receive Data From `su` process

To receive data from `su` process, the `sudo_su_sub_process__receive_comm__start()` function can be used, like it is for `sudo_setup_sudo_shell_home_and_working_environment_wrapper()` that gets `$SUDO_TEMP_DIRECTORY` from `su` process. The same process of using process substitution is used to open a `fd` as detailed in [Send Data To `su` process](#send-data-to-su-process), however, inside the process substitution referred as the relay process, instead of `printf`, we start a `sleep` process in background and wait on it to keep the current process substitution process alive until it is manually killed after `su` process ends and communication is no longer required by calling the `sudo_su_sub_process__receive_comm__stop()` function. A trap is also setup inside the process substitution to also kill the background `sleep` process when the process substitution process is sent a kill signal by `sudo_su_sub_process__receive_comm__stop()`.

The following procedure needs to be used.
1. Call `set_sudo_traps` to setup a trap for `sudo_trap` that calls `sudo_su_sub_process__receive_comm__stop` and `sudo_su_sub_process__receive_comm__cleanup()` if `sudo` script exits early on errors or receives a kill signal.
2. Call `sudo_su_sub_process__receive_comm__start` to start the receive communication by setting up the process substitution and fds.
3. Run `su` command that writes to the `$SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH` variable exported by `sudo_su_sub_process__receive_comm__start()`.
4. Call `sudo_su_sub_process__receive_comm__stop` after `su` process exits to kill the process substitution process.
5. Read data from `$SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH` with `cat`, etc sent by `su` command.
6. Call `sudo_su_sub_process__receive_comm__cleanup()` to close the `$SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD` opened for process substitution.

The `sudo_su_sub_process__receive_comm__stop()` function must be called before reading data from `$SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH`, otherwise script will hang indefinitely.

If the process substitution process is not kept alive with a background `sleep` process and waiting on it, then older devices like below Android `6` and `7`, possibly depending on kernel version, will fail with errors like `/proc/6026/fd/3: Text file busy` when `su` process attempts to write to `$SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH`. So the process substitution process must only be killed after `su` process has exited and sent all its data.

An advance version dependent on `sudo` script functions/variables is the following. Errors and trap are handled.

```shell
# Set traps to run commands before exiting sudo.
set_sudo_traps "sudo_trap" || return $?

# Start relay process.
sudo_su_sub_process__receive_comm__start || return $?

sudo_unset_pre_su_variables
su_output=$($SU_ENV_COMMAND 'receive_data="I think, therefore I am"; echo "$receive_data" > "$SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH"' 2>&1 < /dev/null)
return_value=$?
sudo_set_post_su_variables
[ -n "$su_output" ] && echo "$su_output"
if [ $return_value -ne 0 ]; then
    sudo_log_errors "Failure while running su sub process"
    return $return_value
fi

# Stop relay process.
sudo_su_sub_process__receive_comm__stop || return $?

if [ ! -e "$SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH" ]; then
    sudo_log_errors "The SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH '$SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH' does not exist that needs to be used to read RECEIVE_DATA"
    return 1
fi

RECEIVE_DATA="$(cat "$SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH")"
return_value=$?
if [ $return_value -ne 0 ]; then
    sudo_log_errors "Failure to read RECEIVE_DATA from SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH '$SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH'"
    return $return_value
fi

echo "RECEIVE_DATA='$RECEIVE_DATA'"
#I think, therefore I am

# Close receive fd.
sudo_su_sub_process__receive_comm__cleanup || return $?
```

---

&nbsp;





# Send And Receive Data From `su` process

This is a combination of [Send Data To `su` process](#send-data-to-su-process) and [Receive Data From `su` process](#receive-data-from-su-process).

An advance version dependent on `sudo` script functions/variables is the following. Errors and trap are handled.

```shell
# Set traps to run commands before exiting sudo.
set_sudo_traps "sudo_trap" || return $?

# Get first unsued file descriptor and export its path for `su`.
sudo_get_unsed_file_descriptor_and_path SUDO_SU_SUB_PROCESS__SEND_COMM__FD \
    SUDO_SU_SUB_PROCESS__SEND_COMM__FD_PATH "to send data" || return $?
export SUDO_SU_SUB_PROCESS__SEND_COMM__FD_PATH

send_data="I think, therefore I am"

# Use process substitution to start a process that writes send_data to
# `SUDO_SU_SUB_PROCESS__SEND_COMM__FD` and then exits.
eval "exec $SUDO_SU_SUB_PROCESS__SEND_COMM__FD<" <(printf '%s' "$send_data")


# Start relay process.
sudo_su_sub_process__receive_comm__start || return $?


sudo_unset_pre_su_variables
su_output=$($SU_ENV_COMMAND 'send_data="$(cat "$SUDO_SU_SUB_PROCESS__SEND_COMM__FD_PATH")" && receive_data="${send_data%,*}" && echo "$receive_data" > "$SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH"' 2>&1 < /dev/null)
return_value=$?
sudo_set_post_su_variables
[ -n "$su_output" ] && echo "$su_output"
if [ $return_value -ne 0 ]; then
    sudo_log_errors "Failure while running su sub process"
    return $return_value
fi

# Stop relay process.
sudo_su_sub_process__receive_comm__stop || return $?

if [ ! -e "$SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH" ]; then
    sudo_log_errors "The SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH '$SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH' does not exist that needs to be used to read PROCESSED_DATA"
    return 1
fi

PROCESSED_DATA="$(cat "$SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH")"
return_value=$?
if [ $return_value -ne 0 ]; then
    sudo_log_errors "Failure to read PROCESSED_DATA from SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH '$SUDO_SU_SUB_PROCESS__RECEIVE_COMM__FD_PATH'"
    return $return_value
fi

echo "PROCESSED_DATA='$PROCESSED_DATA'"
#I think

# Close receive fd.
sudo_su_sub_process__receive_comm__cleanup || return $?

# Close send fd.
exec {SUDO_SU_SUB_PROCESS__SEND_COMM__FD}<&-
```

---

&nbsp;





# Additional Rationale

There are other reasons as well why the above method is being used other than the `/proc/self/fd` issues.

1. The above method is faster than `heredocs` and `herestrings` since data is processed only in memory and does not create temp files on the disk increasing execution time like the later. The data is of course buffered with a likely limit of `64KB` but that's still large enough not to cause problems for average script text passing.

2. The temp files created by `heredocs` and `herestrings` are in `$TMPDIR` which in termux's case is `$TERMUX__PREFIX/tmp`. The `$TMPDIR` is relatively less secure since non-root processes can access it and ideally shouldn't be used due to potential risk of privilege escalation or private data leakage. Although any process in Termux context could get `su` access if termux app has been granted root access but other apps granted SAF access can by default only access `$TERMUX__HOME` and not `$TERMUX__PREFIX` by default, and cannot access any root owned directories as Termux app process cannot access them in its `ContentProvider`. You can confirm usage of `$TMPDIR` by running:
```shell
# For herestrings.
sleep 3 <<<"here string" & lsof -w -p $! | grep 0r

# For heredocs.
sleep 3 <<EOF &
here doc
EOF
lsof -w -p $! | grep 0r
```

3. The script command type may require `stdin` to be usable by the scripts. If `heredocs` and `herestrings` are used, then they will be read by the `bash` shell through the `stdin` which will prevent the script shells to use it for user input and they may even read characters from the script itself as `stdin` in some cases. Hence process substitution should be used which leaves `stdin` free, but of course that is not possible due to `/proc/self/fd` issues due to `su` usage.

The other alternative is to create another temp file in `$SUDO_TEMP_DIRECTORY` and store the `$SUDO_SCRIPT_COMMAND_TO_RUN` in it and pass the path to `bash`. This will of course increase execution time significantly due to creation of the `$SUDO_TEMP_DIRECTORY` directory on every run even if `sudo_script__core_script` temp file is not to be created and also due to another call to the `su` shell for the creation of another temp file as current `sudo` script process won't be able to access `$SUDO_TEMP_DIRECTORY`, unless file is created in `$TMPDIR`, creating security issues.

Another hacky alternative that will work is to solve the `/proc/self/fd` issue itself that can be done by guessing the real path of fd inside the `su` shell and pass that to `bash` instead. That can be done by running:

```shell
fd_number="$(echo <(echo "") | sed -E 's|/proc/self/fd/([0-9]+)|\1|')"
/sbin/su --shell="${TERMUX__PREFIX:-$PREFIX}/bin/bash" --preserve-environment -c "bash_shell_pid=$$; "'su_shell_pid=$(pgrep -P $bash_shell_pid); bash /proc/$su_shell_pid/fd/'"$fd_number" <(echo 'echo 1')
```

Use the path to the `su` installed on your device if its not at `/sbin/su`. The first line will just get the `fd` number that is chosen by default by `bash` when it uses process substitution, which is likely going to be `63`. The second line is the shorter version of `$SU_RUN_COMMAND` used by the `sudo` script. The single and double quoting is extremely important to make sure some things are run in the local `sudo` script `bash` shell and others inside the `su` shell. Note that the local `bash` shell is of the `sudo` script and is different from the `bash` shell that is inside the `su` shell. First we get store the pid of the local `bash` shell in `$bash_shell_pid`. Note the double quotes so that `$$` expands locally. Then we get pid of `su` shell by finding the process whose parent pid matches the `$bash_shell_pid`. Since there are no background processes run by the `sudo` script, `pgrep` should return a single pid. Then we pass the `/proc/su_shell_pid/fd/<fd_number>` path to `bash` directly that the `<(echo 'echo 1')` process substitution will create. The `$su_shell_pid` is inside single quotes so it expands inside the `su` shell and `$fd_number` is inside double quotes so it expands locally. Despite the `/proc/self/fd/63` argument passed due to process substitution, bash will read from the path manually passed. Note that we cannot create the path using `$PPID` variable because multiple processes are created by the `su` shell and `$PPID` will not match `$su_shell_pid`. If the `$TERMUX__PREFIX/bin/su` is directly used by running `su --shell...` instead of full path, then another process is created because `su` in Termux `bin` directory is actually a wrapper script and process substitution path will be created for it instead, it may even give errors while executing commands. This method is however not used because its a bit hacky but could work, but would require multiple device and multiple `su` implementation testing. Another reason this isn't used is because normally, the `sudo` binary of linux distros will close all open file descriptors other than standard input, standard output and standard error when its called for security reasons to prevent child processes from getting access to file descriptors of the parent processes that were not intended for it. Some `su` binaries behave in the same way. There are however not closed by SuperSU `v2.82` as shown by the above test and neither by Magisk `v21.1` but other android `su` implementations may close the file descriptors, breaking this method.

To monitor the file descriptors `63` opened by child processes, run the following commands:
```shell
# Drop to a root shell so that root processes can also be monitored.
sudo su
# Run a background infinite loop that passes a comma separated list of
# pids returned by ps to ls using brace expansion that will list all
# files in `/proc/{pids}/fd` paths and then grep only fds matching `63`
# and output their pid to stdout which is captured by `$pid`.
# If $pid is set, then ps is used to display pid,ppid,cmd of all pids stored in the $pid variable
(while true; do pid="$(eval "ls --color=never -d -1 /proc/{$(ps --no-headers -o pid -g | sed -z -e 's/\n/,/g' -e 's/ //g' -e 's/,$//')}/fd/* 2>/dev/null" | grep "/fd/63" | sed -E 's|/proc/([0-9]+)/fd/.*|\1|g')"; [ -n "$pid" ] && ps -wwo pid,ppid,cmd $(echo "$pid" | sed -z -E 's/[ \n\t]+/ /'); done) &
# Store the pid of the background process so that it can be killed later.
while_pid=$!
# Then drop to another root shell.
sudo su
# Then run the su command with an added sleep command for background
# monitoring to work, you may want to increase the sleep time.
# Ideally two commands would be shown by ps that created fd `63`, one
# for the local bash interactive shell of the sudo su command and the
# other for the newly created su shell since the fd will be copied
# during the fork.
fd_number="$(echo <(echo "") | sed -E 's|/proc/self/fd/([0-9]+)|\1|')"
/sbin/su --shell="${TERMUX__PREFIX:-$PREFIX}/bin/bash" --preserve-environment -c "bash_shell_pid=$$; "'su_shell_pid=$(pgrep -P $bash_shell_pid); sleep 0.3; bash /proc/$su_shell_pid/fd/'"$fd_number" <(echo 'echo 1')
# Exit the second root shell once done.
exit
# Kill background process, do not leave it running since it will
# consume lots of resources and may slow down the system.
kill $while_pid
```
There are probably other methods using `lsof` etc.

---

&nbsp;
