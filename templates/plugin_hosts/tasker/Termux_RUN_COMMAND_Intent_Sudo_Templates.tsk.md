# Termux RUN COMMAND Intent Sudo Templates

&nbsp;
## Export Info:
**Tasker Version:** `5.11.7.beta`  
**Timestamp:** `2020-12-14 15.54.33`  
&nbsp;



&nbsp;
## Profile Names:
**Count:** `0`




## Scene Names:
**Count:** `0`




## Task Names:
**Count:** `1`

- `Termux RUN_COMMAND Intent Sudo Templates`
##
&nbsp;



&nbsp;
## Profiles Info:
&nbsp;



&nbsp;
## Tasks Info:
&nbsp;

### Task 1
**Name:** `Termux RUN_COMMAND Intent Sudo Templates`  
**ID:** `999`  
**Collision Handling:** `Abort New Task`  
**Keep Device Awake:** `false`  

#### Help:

A task that provides templates for running `sudo` script commands with the `RUN_COMMAND` intent as the `root (superuser)` user in Termux. The commands are run using the Tasker `TermuxCommand()` function of the `Tasker Function` action and with the `am` command with the `Run Shell` action and are referred as intent actions in this task. This task requires Termux:Tasker version `>= 0.5`. Tasker must be granted `com.termux.permission.RUN_COMMAND` permission. The `sudo` script must be installed at `$PREFIX/bin/sudo`. The `allow-external-apps` property must also be set to `true` in `~/.termux/termux.properties` file, otherwise any commands received via the `RUN_COMMAND` intent will not be executed by Termux. For android `>= 10`, Termux must also be granted `Draw Over Apps` permissions so that foreground commands automatically start executing without the user having to manually click the `Termux` notification in the status bar dropdown notifications list for the commands to start. The device must be rooted and ideally `Termux` must have been granted root permissions by your root manager app like `SuperSU` or `Magisk` for the `sudo` script to work.

Check [Termux:Tasker Github](https://github.com/termux/termux-tasker) and [RunCommand Intent](https://github.com/termux/termux-app/wiki/RUN_COMMAND-Intent) for more details on `RUN_COMMAND` intent configuration.

Check [sudo](https://github.com/agnostic-apollo/sudo) for more details for the `sudo` script.


The result of commands is not received back when commands are run with the `RUN_COMMAND` intent, hence all templates run their commands in the foreground terminal session so that the results can be viewed. The `--sleep=3` command option is also passed to `sudo` so that it sleeps for `3` seconds before exiting. If this is not passed, the terminal would immediately close after `sudo` executes its commands without giving the user a chance to view the results in the terminal session. Moreover, a `Wait` action of `2` seconds is also added to the task after each template so that the templates execute in order and don't start until the previous one has at least ideally started executing.

The default value of the `%comma_alternative` variable in this task is set to `‚` (`#U+201A`, `&sbquo;`, `&#8218;`, `single low-9 quotation mark`). This is the same character that is replaced with the simple comma `,` (`U+002C`, `&comma;`, `&#44;`, `comma`) by the `sudo` script by default when the `-r` option is passed. You can use a different character that should be replaced using the `--comma-alternative` option.


Template 1 runs the `$PREFIX/bin/sudo --sleep=3 dumpsys -l` command in the foreground using `TermuxCommand()` function as a template for the `path` `command_type` to list all android services. The arguments do not contain any simple comma characters and hence can be passed directly without having to replace them with `%comma_alternative` characters.


Template 2 runs the `$PREFIX/bin/sudo -sr --sleep=3 '%core_script' '%argument_1' '%argument_2'` command in the foreground using `TermuxCommand()` function as a template for the `script` `command_type` with `bash` as the `sudo shell`, which would be chosen automatically by default without having to pass the `--shell` option. The `termux_tasker_basic_bash_test` `bash` script text is passed as the `core_script` argument with 2 complex dynamic args that may contain simple comma characters. The `-s` command option is passed to set `command_type` to `script`. The `-r` command option is passed so that `sudo` parses the arguments as per `RUN_COMMAND` intent rules. The `sudo` executable is passed to the intent using the `%executable` variable. The `%core_script`, `%argument_1` and `%argument_2` variables are used to store the dynamic values to be sent as `$1`, `$2` and `$3` respectively to `sudo`. They are first all set in the `%arguments` variable separated by simple comma characters, which is passed to the intent. The `%core_script`, `%argument_1` and `%argument_2` variables may contain any type of characters, even a simple comma, but simple commas must be replaced with the `%comma_alternative` characters before the intent action is run so that their values are not split into multiple arguments by the intent. To replace the simple commas, `Variable Search Replace` action is run to replace all simple commas `,` with the `%comma_alternative` variable value. The `Variable Search Replace` action must be used separately for each argument variable before adding it to the `%arguments` variable.  Do not set multiple arguments in the same variable and use `Variable Search Replace` action on it since that will result in incorrect argument splitting. This template shows how you can dynamically create intent commands at runtime using variables and send them via the `RUN_COMMAND` intent to Termux for execution, including passing the script text itself without having to create a physical file in `~/.termux/tasker/` directory.


Template 3 is almost the same as Template 2, but it runs the `$PREFIX/bin/sudo -sr --shell=python --sleep=3 '%core_script' '%argument_1' '%argument_2'` command in the foreground using `am` command as a template for the `script` `command_type` with `python` as the `sudo shell`. The `termux_tasker_basic_python_test` `python` script text is passed as the `core_script` argument with 2 complex dynamic args that may contain simple comma characters. However, as already mentioned that this template uses the `am` command, the `%arguments` variable is passed to the `com.termux.RUN_COMMAND_ARGUMENTS` string array extra surrounded with single quotes inside a shell and hence any single quotes inside the `%arguments` variable also need to be escaped before running the intent command to prevent incorrect quoting. To escape the single quotes, `Variable Search Replace` action is run to replace all single quotes `'` with one single quote, followed by one backslash, followed by two single quotes `'\''`. So `%arguments` surrounded with single quotes that would have been passed like `'some arg with single quote ' in it'` will be passed as `'some arg with single quote '\'' in it'`. This is basically 3 parts `'some arg with single quote '`, `\'` and `' in it'` but when processed, it will be considered as one single argument with the value `some arg with single quote ' in it` that is passed as the `com.termux.RUN_COMMAND_ARGUMENTS` string array extra value. The `Variable Search Replace` action does not need to be used separately for each argument variable before adding it to the `%arguments` variable, running it on only the final `%arguments` variable would be fine, unlike how its done on individual argument variables like `%argument_1`, etc for usage with `Termux:Tasker` plugin app.


Template 4 runs `$PREFIX/bin/sudo --shell-pre-commands="echo 'starting sudo shell';" --title='sudo' su` in the foreground using `TermuxCommand()` function as a template for the `su` `command_type` to start an interactive `bash` shell with priority to termux binaries and libraries. The `--shell-pre-commands` option is passed to run some commands before starting the interactive `bash` shell and its argument is **not** passed surrounded with single or double quotes to prevent whitespace splitting in the intent action, like done for usage with `Termux:Tasker` plugin app since splitting will occur on simple comma characters instead. If there are going to be simple comma characters in arguments to the command options or in the main arguments to `sudo`, you must replace them with `%comma_alternative` variable value and pass the `-r` option. The `--title` option is passed to set the title of the terminal.


The `$PREFIX/` is a shortcut for the termux prefix directory `/data/data/com.termux/files/usr/`. The `~/` is a shortcut for the termux home directory `/data/data/com.termux/files/home/`. These shortcuts can be used in any path arguments to the `sudo` command.


The `%command_failed` variable will be set if the intent action failed, this is detected by whether `%err` or `%errmsg` is set by the intent action. If you run multiple intent actions in the same task or are using `Local Variable Passthrough`, then you must clear the `%command_failed` variable and optionally the `%errmsg` variable with the `Variable Clear` action before running each intent action, in case they were already set, like by a previously failed intent action after which the task was not stopped.
##


**Parameters:** `-`


**Returns:** `-`


**Control:**

```
version_name: 0.1.0
```
##
&nbsp;



&nbsp;
## Code Description:
&nbsp;

``````
Task Name: Termux RUN_COMMAND Intent Sudo Templates

Actions:
    <A task that provides templates for running `sudo` script commands with the `RUN_COMMAND` intent as the `root (superuser)` user in Termux. The commands are run using the Tasker `TermuxCommand()` function of the `Tasker Function` action and with the `am` command with the `Run Shell` action and are referred as intent actions in this task. This task requires Termux:Tasker version `>= 0.5`. Tasker must be granted `com.termux.permission.RUN_COMMAND` permission. The `sudo` script must be installed at `$PREFIX/bin/sudo`. The `allow-external-apps` property must also be set to `true` in `~/.termux/termux.properties` file, otherwise any commands received via the `RUN_COMMAND` intent will not be executed by Termux. For android `>= 10`, Termux must also be granted `Draw Over Apps` permissions so that foreground commands automatically start executing without the user having to manually click the `Termux` notification in the status bar dropdown notifications list for the commands to start. The device must be rooted and ideally `Termux` must have been granted root permissions by your root manager app like `SuperSU` or `Magisk` for the `sudo` script to work.
    
    Check [Termux:Tasker Github](https://github.com/termux/termux-tasker) and [RunCommand Intent](https://github.com/termux/termux-app/wiki/RUN_COMMAND-Intent) for more details on `RUN_COMMAND` intent configuration.
    
    Check [sudo](https://github.com/agnostic-apollo/sudo) for more details for the `sudo` script.
    
    
    The result of commands is not received back when commands are run with the `RUN_COMMAND` intent, hence all templates run their commands in the foreground terminal session so that the results can be viewed. The `--sleep=3` command option is also passed to `sudo` so that it sleeps for `3` seconds before exiting. If this is not passed, the terminal would immediately close after `sudo` executes its commands without giving the user a chance to view the results in the terminal session. Moreover, a `Wait` action of `2` seconds is also added to the task after each template so that the templates execute in order and don't start until the previous one has at least ideally started executing.
    
    The default value of the `%comma_alternative` variable in this task is set to `‚` (`#U+201A`, `&sbquo;`, `&#8218;`, `single low-9 quotation mark`). This is the same character that is replaced with the simple comma `,` (`U+002C`, `&comma;`, `&#44;`, `comma`) by the `sudo` script by default when the `-r` option is passed. You can use a different character that should be replaced using the `--comma-alternative` option.
    
    
    Template 1 runs the `$PREFIX/bin/sudo --sleep=3 dumpsys -l` command in the foreground using `TermuxCommand()` function as a template for the `path` `command_type` to list all android services. The arguments do not contain any simple comma characters and hence can be passed directly without having to replace them with `%comma_alternative` characters.
    
    
    Template 2 runs the `$PREFIX/bin/sudo -sr --sleep=3 '%core_script' '%argument_1' '%argument_2'` command in the foreground using `TermuxCommand()` function as a template for the `script` `command_type` with `bash` as the `sudo shell`, which would be chosen automatically by default without having to pass the `--shell` option. The `termux_tasker_basic_bash_test` `bash` script text is passed as the `core_script` argument with 2 complex dynamic args that may contain simple comma characters. The `-s` command option is passed to set `command_type` to `script`. The `-r` command option is passed so that `sudo` parses the arguments as per `RUN_COMMAND` intent rules. The `sudo` executable is passed to the intent using the `%executable` variable. The `%core_script`, `%argument_1` and `%argument_2` variables are used to store the dynamic values to be sent as `$1`, `$2` and `$3` respectively to `sudo`. They are first all set in the `%arguments` variable separated by simple comma characters, which is passed to the intent. The `%core_script`, `%argument_1` and `%argument_2` variables may contain any type of characters, even a simple comma, but simple commas must be replaced with the `%comma_alternative` characters before the intent action is run so that their values are not split into multiple arguments by the intent. To replace the simple commas, `Variable Search Replace` action is run to replace all simple commas `,` with the `%comma_alternative` variable value. The `Variable Search Replace` action must be used separately for each argument variable before adding it to the `%arguments` variable.  Do not set multiple arguments in the same variable and use `Variable Search Replace` action on it since that will result in incorrect argument splitting. This template shows how you can dynamically create intent commands at runtime using variables and send them via the `RUN_COMMAND` intent to Termux for execution, including passing the script text itself without having to create a physical file in `~/.termux/tasker/` directory.
    
    
    Template 3 is almost the same as Template 2, but it runs the `$PREFIX/bin/sudo -sr --shell=python --sleep=3 '%core_script' '%argument_1' '%argument_2'` command in the foreground using `am` command as a template for the `script` `command_type` with `python` as the `sudo shell`. The `termux_tasker_basic_python_test` `python` script text is passed as the `core_script` argument with 2 complex dynamic args that may contain simple comma characters. However, as already mentioned that this template uses the `am` command, the `%arguments` variable is passed to the `com.termux.RUN_COMMAND_ARGUMENTS` string array extra surrounded with single quotes inside a shell and hence any single quotes inside the `%arguments` variable also need to be escaped before running the intent command to prevent incorrect quoting. To escape the single quotes, `Variable Search Replace` action is run to replace all single quotes `'` with one single quote, followed by one backslash, followed by two single quotes `'\''`. So `%arguments` surrounded with single quotes that would have been passed like `'some arg with single quote ' in it'` will be passed as `'some arg with single quote '\'' in it'`. This is basically 3 parts `'some arg with single quote '`, `\'` and `' in it'` but when processed, it will be considered as one single argument with the value `some arg with single quote ' in it` that is passed as the `com.termux.RUN_COMMAND_ARGUMENTS` string array extra value. The `Variable Search Replace` action does not need to be used separately for each argument variable before adding it to the `%arguments` variable, running it on only the final `%arguments` variable would be fine, unlike how its done on individual argument variables like `%argument_1`, etc for usage with `Termux:Tasker` plugin app.
    
    
    Template 4 runs `$PREFIX/bin/sudo --shell-pre-commands="echo 'starting sudo shell';" --title='sudo' su` in the foreground using `TermuxCommand()` function as a template for the `su` `command_type` to start an interactive `bash` shell with priority to termux binaries and libraries. The `--shell-pre-commands` option is passed to run some commands before starting the interactive `bash` shell and its argument is **not** passed surrounded with single or double quotes to prevent whitespace splitting in the intent action, like done for usage with `Termux:Tasker` plugin app since splitting will occur on simple comma characters instead. If there are going to be simple comma characters in arguments to the command options or in the main arguments to `sudo`, you must replace them with `%comma_alternative` variable value and pass the `-r` option. The `--title` option is passed to set the title of the terminal.
    
    
    The `$PREFIX/` is a shortcut for the termux prefix directory `/data/data/com.termux/files/usr/`. The `~/` is a shortcut for the termux home directory `/data/data/com.termux/files/home/`. These shortcuts can be used in any path arguments to the `sudo` command.
    
    
    The `%command_failed` variable will be set if the intent action failed, this is detected by whether `%err` or `%errmsg` is set by the intent action. If you run multiple intent actions in the same task or are using `Local Variable Passthrough`, then you must clear the `%command_failed` variable and optionally the `%errmsg` variable with the `Variable Clear` action before running each intent action, in case they were already set, like by a previously failed intent action after which the task was not stopped.
    ##
    
    
    **Parameters:** `-`
    
    
    **Returns:** `-`
    
    
    **Control:**
    
    ```
    version_name: 0.1.0
    ```>
    A1: Anchor 

    A2: Variable Set [ 
        Name:%task_name 
        To:Termux RUN_COMMAND Intent Sudo Templates 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] 

    <Template 1 Start>
    A3: Anchor 

    <Goto "Template 2 Start"
    Enable this action to skip running this template>
    A4: [X] Goto [ 
        Type:Action Label 
        Number:1 
        Label:Template 2 Start ] 

    <Run `$PREFIX/bin/sudo --sleep=3 dumpsys -l` Command In Foreground>
    A5: Anchor 

    A6: Variable Clear [ 
        Name:%command_failed/%errmsg 
        Pattern Matching:On 
        Local Variables Only:On 
        Clear All Variables:Off ] 

    <Run Termux RUN_COMMAND Intent Command with TermuxCommand() Function>
    A7: Tasker Function [  
        Function:TermuxCommand($PREFIX/bin/sudo,--sleep=3,dumpsys,-l,/data/data/com.termux/files/home,false) Continue Task After Error:On ] 

    A8: Variable Set [ 
        Name:%command_failed 
        To:err = `%err`
    
    errmsg =
    ```
    %errmsg
    ```
    
    stdout =
    ```
    %stdout
    ```
    
    stderr =
    ```
    %stderr
    ``` 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] If [ %err Set | %errmsg Set ]

    A9: If [ %command_failed Set ]

        <remove %err and %errmsg if not set>
        A10: Variable Search Replace [ 
            Variable:%command_failed 
            Search:^err = `\%err`[\n]+errmsg =[\n]```[\n]\%errmsg[\n]```[\n]+ 
            Ignore Case:Off 
            Multi-Line:Off 
            One Match Only:Off 
            Store Matches In Array: 
            Replace Matches:On 
            Replace With: Continue Task After Error:On ] If [ %command_failed Set ]

        A11: Text Dialog [  
            Title:Template 1 Command
        Failed 
            Text:%command_failed 
            Button 1:OK 
            Button 2: 
            Button 3: 
            Close After (Seconds):30 
            Use HTML:Off ] 

        A12: Stop [ 
            With Error:Off 
            Task: ] 

    A13: End If 

    <Template 1 End>
    A14: Anchor 

    A15: Wait [ 
        MS:0 
        Seconds:2 
        Minutes:0 
        Hours:0 
        Days:0 ] 

    <Template 2 Start>
    A16: Anchor 

    <Goto "Template 3 Start"
    Enable this action to skip running this template>
    A17: [X] Goto [ 
        Type:Action Label 
        Number:1 
        Label:Template 3 Start ] 

    <Run `$PREFIX/bin/sudo -sr --sleep=3 '%core_script' '%argument_1' '%argument_2'` Command In Foreground>
    A18: Anchor 

    <set `‚` (`#U+201A`, `&sbquo;`, `&#8218;`, `single low-9 quotation mark`) to %comma_alternative>
    A19: Variable Set [ 
        Name:%comma_alternative 
        To:‚ 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] 

    <set `$PREFIX/bin/sudo` to %executable>
    A20: Variable Set [ 
        Name:%executable 
        To:$PREFIX/bin/sudo 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] 

    <set `termux_tasker_basic_bash_test` script text to %core_script>
    A21: Variable Set [ 
        Name:%core_script 
        To:#if parameter count is not 2
    if [ $# -ne 2 ]; then
    echo "Invalid parameter count '$#' to 'termux_tasker_basic_bash_test'" 1>&2
    echo "$*" 1>&2
    exit 1
    fi
    
    echo "\$1=\`$1\`"
    echo "\$2=\`$2\`"
    
    exit 0 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] 

    A22: Variable Set [ 
        Name:%argument_1 
        To:json 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] 

    A23: Variable Set [ 
        Name:%argument_2 
        To:{
        "name":"I'm Termux",
        "license":"GPLv3",
        "addons": {
            "1":"Termux:API",
            "2":"Termux:Boot",
            "3":"Termux:Float",
            "4":"Termux:Styling",
            "5":"Termux:Tasker",
            "6":"Termux:Widget"
        }
    } 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] 

    <replace all simple commas `,` (`U+002C`, `&comma;`, `&#44;`, `comma`) with %comma_alternative characters>
    A24: Variable Search Replace [ 
        Variable:%core_script 
        Search:, 
        Ignore Case:Off 
        Multi-Line:Off 
        One Match Only:Off 
        Store Matches In Array: 
        Replace Matches:On 
        Replace With:%comma_alternative ] If [ %core_script Set ]

    <replace all simple commas `,` (`U+002C`, `&comma;`, `&#44;`, `comma`) with %comma_alternative characters>
    A25: Variable Search Replace [ 
        Variable:%argument_1 
        Search:, 
        Ignore Case:Off 
        Multi-Line:Off 
        One Match Only:Off 
        Store Matches In Array: 
        Replace Matches:On 
        Replace With:%comma_alternative ] If [ %argument_1 Set ]

    <replace all simple commas `,` (`U+002C`, `&comma;`, `&#44;`, `comma`) with %comma_alternative characters>
    A26: Variable Search Replace [ 
        Variable:%argument_2 
        Search:, 
        Ignore Case:Off 
        Multi-Line:Off 
        One Match Only:Off 
        Store Matches In Array: 
        Replace Matches:On 
        Replace With:%comma_alternative ] If [ %argument_2 Set ]

    <set `-sr,--sleep=3,%core_script,%argument_1,%argument_2` to %arguments>
    A27: Variable Set [ 
        Name:%arguments 
        To:-sr,--sleep=3,%core_script,%argument_1,%argument_2 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] 

    <set `%executable,%arguments,/data/data/com.termux/files/home,false` to %termux_command>
    A28: Variable Set [ 
        Name:%termux_command 
        To:%executable,%arguments,/data/data/com.termux/files/home,false 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] 

    A29: Variable Clear [ 
        Name:%command_failed/%errmsg 
        Pattern Matching:On 
        Local Variables Only:On 
        Clear All Variables:Off ] 

    <Run Termux RUN_COMMAND Intent Command with TermuxCommand() Function>
    A30: Tasker Function [  
        Function:TermuxCommand(%termux_command) Continue Task After Error:On ] 

    A31: Variable Set [ 
        Name:%command_failed 
        To:err = `%err`
    
    errmsg =
    ```
    %errmsg
    ``` 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] If [ %err Set | %errmsg Set ]

    A32: If [ %command_failed Set ]

        A33: Text Dialog [  
            Title:Template 2 Command
        Failed 
            Text:%command_failed 
            Button 1:OK 
            Button 2: 
            Button 3: 
            Close After (Seconds):30 
            Use HTML:Off ] 

        A34: Stop [ 
            With Error:Off 
            Task: ] 

    A35: End If 

    <Template 2 End>
    A36: Anchor 

    A37: Wait [ 
        MS:0 
        Seconds:2 
        Minutes:0 
        Hours:0 
        Days:0 ] 

    <Template 3 Start>
    A38: Anchor 

    <Goto "Template 4 Start"
    Enable this action to skip running this template>
    A39: [X] Goto [ 
        Type:Action Label 
        Number:1 
        Label:Template 4 Start ] 

    <Run `$PREFIX/bin/sudo -sr --shell=python --sleep=3 '%core_script' '%argument_1' '%argument_2'` Command In Foreground>
    A40: Anchor 

    <set `‚` (`#U+201A`, `&sbquo;`, `&#8218;`, `single low-9 quotation mark`) to %comma_alternative>
    A41: Variable Set [ 
        Name:%comma_alternative 
        To:‚ 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] 

    <set `$PREFIX/bin/sudo` to %executable>
    A42: Variable Set [ 
        Name:%executable 
        To:$PREFIX/bin/sudo 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] 

    <set `termux_tasker_basic_python_test` script text to %core_script>
    A43: Variable Set [ 
        Name:%core_script 
        To:import sys
    
    argv_size = len(sys.argv) - 1
    
    # if parameter count is not 2
    if argv_size != 2:
    print("Invalid parameter count '%s' to 'termux_tasker_basic_python_test'" % argv_size, file=sys.stderr)
    print("%s" % " ".join(sys.argv[1:]), file=sys.stderr)
    sys.exit(1)
    
    print("$1=`%s`" % sys.argv[1])
    print("$2=`%s`" % sys.argv[2])
    
    sys.exit(0) 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] 

    A44: Variable Set [ 
        Name:%argument_1 
        To:json 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] 

    A45: Variable Set [ 
        Name:%argument_2 
        To:{
        "name":"I'm Termux",
        "license":"GPLv3",
        "addons": {
            "1":"Termux:API",
            "2":"Termux:Boot",
            "3":"Termux:Float",
            "4":"Termux:Styling",
            "5":"Termux:Tasker",
            "6":"Termux:Widget"
        }
    } 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] 

    <replace all simple commas `,` (`U+002C`, `&comma;`, `&#44;`, `comma`) with %comma_alternative characters>
    A46: Variable Search Replace [ 
        Variable:%core_script 
        Search:, 
        Ignore Case:Off 
        Multi-Line:Off 
        One Match Only:Off 
        Store Matches In Array: 
        Replace Matches:On 
        Replace With:%comma_alternative ] If [ %core_script Set ]

    <replace all simple commas `,` (`U+002C`, `&comma;`, `&#44;`, `comma`) with %comma_alternative characters>
    A47: Variable Search Replace [ 
        Variable:%argument_1 
        Search:, 
        Ignore Case:Off 
        Multi-Line:Off 
        One Match Only:Off 
        Store Matches In Array: 
        Replace Matches:On 
        Replace With:%comma_alternative ] If [ %argument_1 Set ]

    <replace all simple commas `,` (`U+002C`, `&comma;`, `&#44;`, `comma`) with %comma_alternative characters>
    A48: Variable Search Replace [ 
        Variable:%argument_2 
        Search:, 
        Ignore Case:Off 
        Multi-Line:Off 
        One Match Only:Off 
        Store Matches In Array: 
        Replace Matches:On 
        Replace With:%comma_alternative ] If [ %argument_2 Set ]

    <set `-sr,--shell=python,--sleep=3,%core_script,%argument_1,%argument_2` to %arguments>
    A49: Variable Set [ 
        Name:%arguments 
        To:-sr,--shell=python,--sleep=3,%core_script,%argument_1,%argument_2 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] 

    <replace all single quotes (') with ('\'')>
    A50: Variable Search Replace [ 
        Variable:%arguments 
        Search:' 
        Ignore Case:Off 
        Multi-Line:On 
        One Match Only:Off 
        Store Matches In Array: 
        Replace Matches:On 
        Replace With:'\\'' ] If [ %arguments Set ]

    A51: Variable Clear [ 
        Name:%command_failed/%errmsg 
        Pattern Matching:On 
        Local Variables Only:On 
        Clear All Variables:Off ] 

    <Run Termux RUN_COMMAND Intent Command with `am` Command>
    A52: Run Shell [ 
        Command:am startservice --user 0 -n com.termux/com.termux.app.RunCommandService -a com.termux.RUN_COMMAND --es com.termux.RUN_COMMAND_PATH '%executable' --esa com.termux.RUN_COMMAND_ARGUMENTS '%arguments' --es com.termux.RUN_COMMAND_WORKDIR '/data/data/com.termux/files/home' --ez com.termux.RUN_COMMAND_BACKGROUND 'false' 
        Timeout (Seconds):0 
        Use Root:Off 
        Store Output In:%stdout 
        Store Errors In:%stderr 
        Store Result In: Continue Task After Error:On ] 

    A53: Variable Set [ 
        Name:%command_failed 
        To:err = `%err`
    
    errmsg =
    ```
    %errmsg
    ```
    
    stdout =
    ```
    %stdout
    ```
    
    stderr =
    ```
    %stderr
    ``` 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] If [ %err Set | %errmsg Set ]

    A54: If [ %command_failed Set ]

        <remove %err and %errmsg if not set>
        A55: Variable Search Replace [ 
            Variable:%command_failed 
            Search:^err = `\%err`[\n]+errmsg =[\n]```[\n]\%errmsg[\n]```[\n]+ 
            Ignore Case:Off 
            Multi-Line:Off 
            One Match Only:Off 
            Store Matches In Array: 
            Replace Matches:On 
            Replace With: Continue Task After Error:On ] If [ %command_failed Set ]

        A56: Text Dialog [  
            Title:Template 3 Command
        Failed 
            Text:%command_failed 
            Button 1:OK 
            Button 2: 
            Button 3: 
            Close After (Seconds):30 
            Use HTML:Off ] 

        A57: Stop [ 
            With Error:Off 
            Task: ] 

    A58: Else 

        A59: [X] Text Dialog [  
            Title:Template 3 Command Result 
            Text:stdout =
        ```
        %stdout
        ```
        
        stderr =
        ```
        %stderr
        ``` 
            Button 1:OK 
            Button 2: 
            Button 3: 
            Close After (Seconds):30 
            Use HTML:Off ] 

    A60: End If 

    <Template 3 End>
    A61: Anchor 

    A62: Wait [ 
        MS:0 
        Seconds:2 
        Minutes:0 
        Hours:0 
        Days:0 ] 

    <Template 4 Start>
    A63: Anchor 

    <Goto "Return"
    Enable this action to skip running this template>
    A64: [X] Goto [ 
        Type:Action Label 
        Number:1 
        Label:Return ] 

    <Run `$PREFIX/bin/sudo --shell-pre-commands="echo 'starting sudo shell';" --title='sudo' su` Command In Foreground>
    A65: Anchor 

    A66: Variable Clear [ 
        Name:%command_failed/%errmsg 
        Pattern Matching:On 
        Local Variables Only:On 
        Clear All Variables:Off ] 

    <Run Termux RUN_COMMAND Intent Command with TermuxCommand() Function>
    A67: Tasker Function [  
        Function:TermuxCommand($PREFIX/bin/sudo,--shell-pre-commands=echo 'starting sudo shell';,--title='sudo',su,/data/data/com.termux/files/home,false) Continue Task After Error:On ] 

    A68: Variable Set [ 
        Name:%command_failed 
        To:err = `%err`
    
    errmsg =
    ```
    %errmsg
    ``` 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] If [ %err Set | %errmsg Set ]

    A69: If [ %command_failed Set ]

        A70: Text Dialog [  
            Title:Template 4 Command
        Failed 
            Text:%command_failed 
            Button 1:OK 
            Button 2: 
            Button 3: 
            Close After (Seconds):30 
            Use HTML:Off ] 

        A71: Stop [ 
            With Error:Off 
            Task: ] 

    A72: End If 

    <Template 4 End>
    A73: Anchor 

    <Return>
    A74: Anchor 
``````

##
&nbsp;


*This file was automatically generated using [tasker_config_utils v0.5.0](https://github.com/Taskomater/tasker_config_utils).*
