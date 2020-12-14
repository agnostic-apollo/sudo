# Termux Tasker Plugin Sudo Templates

&nbsp;
## Export Info:
**Tasker Version:** `5.11.7.beta`  
**Timestamp:** `2020-12-14 15.53.34`  
&nbsp;



&nbsp;
## Profile Names:
**Count:** `0`




## Scene Names:
**Count:** `0`




## Task Names:
**Count:** `1`

- `Termux Tasker Plugin Sudo Templates`
##
&nbsp;



&nbsp;
## Profiles Info:
&nbsp;



&nbsp;
## Tasks Info:
&nbsp;

### Task 1
**Name:** `Termux Tasker Plugin Sudo Templates`  
**ID:** `992`  
**Collision Handling:** `Abort New Task`  
**Keep Device Awake:** `false`  

#### Help:

A task that provides templates for running `sudo` script commands with the Termux:Tasker plugin in `superuser (root)` context in termux. This task requires Termux:Tasker version `>= 0.5`. Tasker must be granted `com.termux.permission.RUN_COMMAND` permission. The `sudo` script must be installed at `$PREFIX/bin/sudo`. The `allow-external-apps` property must also be set to `true` in `~/.termux/termux.properties` file since the `$PREFIX/bin/sudo` absolute path is outside the `~/.termux/tasker/` directory, otherwise the plugin actions will fail. For android `>= 10`, Termux must also be granted `Draw Over Apps` permissions so that foreground commands automatically start executing without the user having to manually click the `Termux` notification in the status bar dropdown notifications list for the commands to start. The device must be rooted and ideally `Termux` must have been granted root permissions by your root manager app like `SuperSU` or `Magisk` for the `sudo` script to work.

Check [Termux:Tasker Github](https://github.com/termux/termux-tasker) for more details on plugin configuration and variables and how to handle them.

Check [sudo](https://github.com/agnostic-apollo/sudo) for more details for the `sudo` script.


Template 1 runs the `$PREFIX/bin/sudo dumpsys -l` command in the background as a template for the `path` `command_type` to list all android services. The args sent do not contain any quotes or special characters and can simply be sent, optionally surrounded with double quotes.


Template 2 runs the `$PREFIX/bin/sudo -s '%core_script' '%argument_1' '%argument_2'` command in the background as a template for the `script` `command_type` with `bash` as the `sudo shell`, which would be chosen automatically by default without having to pass the `--shell` option. The `termux_tasker_basic_bash_test` `bash` script text is passed as the `core_script` argument with 2 complex dynamic args that may contain quotes or special characters, basically a literal string. The `-s` command option is passed to set `command_type` to `script`. The result is received back in the `%stdout` variable. The `sudo` executable is passed to the plugin using the `%executable` variable. The `%core_script`, `%argument_1` and `%argument_2` variables are used to store the dynamic values to be sent as `$1`, `$2` and `$3` respectively to `sudo` and are sent surrounded with single quotes instead of double quotes. They are first all set in the `%arguments` variable surrounded with single quotes and separated by a whitespace which is passed to the plugin. The `%core_script`, `%argument_1` and `%argument_2` variables may contain any type of characters, even a single quote, but single quotes must be escaped before the plugin action is run. To escape the single quotes, `Variable Search Replace` action is run to replace all single quotes `'` with one single quote, followed by one backslash, followed by two single quotes `'\''`. So `%argument_1` surrounded with single quotes that would have been passed like `'some arg with single quote ' in it'` will be passed as `'some arg with single quote '\'' in it'`. This is basically 3 parts `'some arg with single quote '`, `\'` and `' in it'` but when processed, it will be considered as one single argument with the value `some arg with single quote ' in it` that is passed to the executable as `$2`. The `Variable Search Replace` action must be used separately for each argument variable before adding it to the `%arguments` variable.  Do not set multiple arguments in the same variable and use `Variable Search Replace` action on it since that will result in incorrect quoting. This template shows how you can dynamically create plugin commands at runtime using variables and send them to the plugin for execution, including passing the script text itself without having to create a physical file in `~/.termux/tasker/` directory.


Template 3 is almost the same as Template 2, but it runs the `$PREFIX/bin/sudo -s --shell=python '%core_script' '%argument_1' '%argument_2'` command in the background as a template for the `script` `command_type` with `python` as the `sudo shell`. The `termux_tasker_basic_python_test` `python` script text is passed as the `core_script` argument with 2 complex dynamic args that may contain quotes or special characters, basically a literal string. The result is received back in the `%stdout` variable.


Template 4 runs `$PREFIX/bin/sudo --shell-pre-commands="echo 'starting sudo shell';" --title='sudo' su` in the foreground as a template for the `su` `command_type` to start an interactive `bash` shell with priority to termux binaries and libraries. The `--shell-pre-commands` option is passed to run some commands before starting the interactive `bash` shell and its argument is passed surrounded with double quotes to prevent whitespace splitting. The `--title` option is passed to set the title of the terminal. Since commands will be run in a foreground terminal session, the `%stdout`, `%stderr` and `%result` variables will not be returned and only `%err` and `%errmsg` may be returned if the action fails.


The `$PREFIX/` is a shortcut for the termux prefix directory `/data/data/com.termux/files/usr/`. The `~/` is a shortcut for the termux home directory `/data/data/com.termux/files/home/`. These shortcuts can be used in the `Executable` and the `Working Directory` plugin fields and in any path arguments to the `sudo` command. The scripts or binaries in the `~/.termux/tasker/` directory do not require them to prefixed with the full path, just set the name in the `Executable` field.


The `%command_failed` variable will be set if the plugin action failed, this is detected by whether `%err` or `%errmsg` is set by the plugin action or if `%result` does not equal `0` for background commands. If you run multiple plugin actions in the same task or are using `Local Variable Passthrough`, then you must clear the `%command_failed` variable and optionally the `%errmsg`, `%stdout`, `%stderr` and `%result` variables with the `Variable Clear` action before running each plugin action, in case they were already set, like by a previously failed plugin action after which the task was not stopped.


To debug arguments being passed or any errors, you can check `logcat` after increasing log level to `Debug`. Check `Debugging` section of `README.md` for more details.
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
Task Name: Termux Tasker Plugin Sudo Templates

Actions:
    <A task that provides templates for running `sudo` script commands with the Termux:Tasker plugin in `superuser (root)` context in termux. This task requires Termux:Tasker version `>= 0.5`. Tasker must be granted `com.termux.permission.RUN_COMMAND` permission. The `sudo` script must be installed at `$PREFIX/bin/sudo`. The `allow-external-apps` property must also be set to `true` in `~/.termux/termux.properties` file since the `$PREFIX/bin/sudo` absolute path is outside the `~/.termux/tasker/` directory, otherwise the plugin actions will fail. For android `>= 10`, Termux must also be granted `Draw Over Apps` permissions so that foreground commands automatically start executing without the user having to manually click the `Termux` notification in the status bar dropdown notifications list for the commands to start. The device must be rooted and ideally `Termux` must have been granted root permissions by your root manager app like `SuperSU` or `Magisk` for the `sudo` script to work.
    
    Check [Termux:Tasker Github](https://github.com/termux/termux-tasker) for more details on plugin configuration and variables and how to handle them.
    
    Check [sudo](https://github.com/agnostic-apollo/sudo) for more details for the `sudo` script.
    
    
    Template 1 runs the `$PREFIX/bin/sudo dumpsys -l` command in the background as a template for the `path` `command_type` to list all android services. The args sent do not contain any quotes or special characters and can simply be sent, optionally surrounded with double quotes.
    
    
    Template 2 runs the `$PREFIX/bin/sudo -s '%core_script' '%argument_1' '%argument_2'` command in the background as a template for the `script` `command_type` with `bash` as the `sudo shell`, which would be chosen automatically by default without having to pass the `--shell` option. The `termux_tasker_basic_bash_test` `bash` script text is passed as the `core_script` argument with 2 complex dynamic args that may contain quotes or special characters, basically a literal string. The `-s` command option is passed to set `command_type` to `script`. The result is received back in the `%stdout` variable. The `sudo` executable is passed to the plugin using the `%executable` variable. The `%core_script`, `%argument_1` and `%argument_2` variables are used to store the dynamic values to be sent as `$1`, `$2` and `$3` respectively to `sudo` and are sent surrounded with single quotes instead of double quotes. They are first all set in the `%arguments` variable surrounded with single quotes and separated by a whitespace which is passed to the plugin. The `%core_script`, `%argument_1` and `%argument_2` variables may contain any type of characters, even a single quote, but single quotes must be escaped before the plugin action is run. To escape the single quotes, `Variable Search Replace` action is run to replace all single quotes `'` with one single quote, followed by one backslash, followed by two single quotes `'\''`. So `%argument_1` surrounded with single quotes that would have been passed like `'some arg with single quote ' in it'` will be passed as `'some arg with single quote '\'' in it'`. This is basically 3 parts `'some arg with single quote '`, `\'` and `' in it'` but when processed, it will be considered as one single argument with the value `some arg with single quote ' in it` that is passed to the executable as `$2`. The `Variable Search Replace` action must be used separately for each argument variable before adding it to the `%arguments` variable.  Do not set multiple arguments in the same variable and use `Variable Search Replace` action on it since that will result in incorrect quoting. This template shows how you can dynamically create plugin commands at runtime using variables and send them to the plugin for execution, including passing the script text itself without having to create a physical file in `~/.termux/tasker/` directory.
    
    
    Template 3 is almost the same as Template 2, but it runs the `$PREFIX/bin/sudo -s --shell=python '%core_script' '%argument_1' '%argument_2'` command in the background as a template for the `script` `command_type` with `python` as the `sudo shell`. The `termux_tasker_basic_python_test` `python` script text is passed as the `core_script` argument with 2 complex dynamic args that may contain quotes or special characters, basically a literal string. The result is received back in the `%stdout` variable.
    
    
    Template 4 runs `$PREFIX/bin/sudo --shell-pre-commands="echo 'starting sudo shell';" --title='sudo' su` in the foreground as a template for the `su` `command_type` to start an interactive `bash` shell with priority to termux binaries and libraries. The `--shell-pre-commands` option is passed to run some commands before starting the interactive `bash` shell and its argument is passed surrounded with double quotes to prevent whitespace splitting. The `--title` option is passed to set the title of the terminal. Since commands will be run in a foreground terminal session, the `%stdout`, `%stderr` and `%result` variables will not be returned and only `%err` and `%errmsg` may be returned if the action fails.
    
    
    The `$PREFIX/` is a shortcut for the termux prefix directory `/data/data/com.termux/files/usr/`. The `~/` is a shortcut for the termux home directory `/data/data/com.termux/files/home/`. These shortcuts can be used in the `Executable` and the `Working Directory` plugin fields and in any path arguments to the `sudo` command. The scripts or binaries in the `~/.termux/tasker/` directory do not require them to prefixed with the full path, just set the name in the `Executable` field.
    
    
    The `%command_failed` variable will be set if the plugin action failed, this is detected by whether `%err` or `%errmsg` is set by the plugin action or if `%result` does not equal `0` for background commands. If you run multiple plugin actions in the same task or are using `Local Variable Passthrough`, then you must clear the `%command_failed` variable and optionally the `%errmsg`, `%stdout`, `%stderr` and `%result` variables with the `Variable Clear` action before running each plugin action, in case they were already set, like by a previously failed plugin action after which the task was not stopped.
    
    
    To debug arguments being passed or any errors, you can check `logcat` after increasing log level to `Debug`. Check `Debugging` section of `README.md` for more details.
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
        To:Termux Tasker Plugin Sudo Templates 
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

    <Run `$PREFIX/bin/sudo dumpsys -l` Command In Background>
    A5: Anchor 

    A6: Variable Clear [ 
        Name:%command_failed/%errmsg/%stdout/%stderr/%result 
        Pattern Matching:On 
        Local Variables Only:On 
        Clear All Variables:Off ] 

    <Run Termux:Tasker Plugin Command>
    A7: Termux [ Configuration:$PREFIX/bin/sudo dumpsys -l Timeout (Seconds):10 Continue Task After Error:On ] 

    A8: Variable Set [ 
        Name:%command_failed 
        To:err = `%err`
    
    errmsg =
    ```
    %errmsg
    ```
    
    exit_code = `%result`
    
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
        Max Rounding Digits:3 ] If [ %err Set | %errmsg Set | %result neq 0 ]

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

    A13: Else 

        A14: Text Dialog [  
            Title:Template 1 Command Result 
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

    A15: End If 

    <Template 1 End>
    A16: Anchor 

    <Template 2 Start>
    A17: Anchor 

    <Goto "Template 3 Start"
    Enable this action to skip running this template>
    A18: [X] Goto [ 
        Type:Action Label 
        Number:1 
        Label:Template 3 Start ] 

    <Run `$PREFIX/bin/sudo -s '%core_script' '%argument_1' '%argument_2'` Command In Background>
    A19: Anchor 

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

    <replace all single quotes (') with ('\'')>
    A24: Variable Search Replace [ 
        Variable:%core_script 
        Search:' 
        Ignore Case:Off 
        Multi-Line:On 
        One Match Only:Off 
        Store Matches In Array: 
        Replace Matches:On 
        Replace With:'\\'' ] If [ %core_script Set ]

    <replace all single quotes (') with ('\'')>
    A25: Variable Search Replace [ 
        Variable:%argument_1 
        Search:' 
        Ignore Case:Off 
        Multi-Line:On 
        One Match Only:Off 
        Store Matches In Array: 
        Replace Matches:On 
        Replace With:'\\'' ] If [ %argument_1 Set ]

    <replace all single quotes (') with ('\'')>
    A26: Variable Search Replace [ 
        Variable:%argument_2 
        Search:' 
        Ignore Case:Off 
        Multi-Line:On 
        One Match Only:Off 
        Store Matches In Array: 
        Replace Matches:On 
        Replace With:'\\'' ] If [ %argument_2 Set ]

    <set `-s '%core_script' '%argument_1' '%argument_2'` to %arguments>
    A27: Variable Set [ 
        Name:%arguments 
        To:-s '%core_script' '%argument_1' '%argument_2' 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] 

    A28: Variable Clear [ 
        Name:%command_failed/%errmsg/%stdout/%stderr/%result 
        Pattern Matching:On 
        Local Variables Only:On 
        Clear All Variables:Off ] 

    <Run Termux:Tasker Plugin Command>
    A29: Termux [ Configuration:%executable %arguments Timeout (Seconds):10 Continue Task After Error:On ] 

    A30: Variable Set [ 
        Name:%command_failed 
        To:err = `%err`
    
    errmsg =
    ```
    %errmsg
    ```
    
    exit_code = `%result`
    
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
        Max Rounding Digits:3 ] If [ %err Set | %errmsg Set | %result neq 0 ]

    A31: If [ %command_failed Set ]

        <remove %err and %errmsg if not set>
        A32: Variable Search Replace [ 
            Variable:%command_failed 
            Search:^err = `\%err`[\n]+errmsg =[\n]```[\n]\%errmsg[\n]```[\n]+ 
            Ignore Case:Off 
            Multi-Line:Off 
            One Match Only:Off 
            Store Matches In Array: 
            Replace Matches:On 
            Replace With: Continue Task After Error:On ] If [ %command_failed Set ]

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

    A35: Else 

        A36: Text Dialog [  
            Title:Template 2 Command Result 
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

    A37: End If 

    <Template 2 End>
    A38: Anchor 

    <Template 3 Start>
    A39: Anchor 

    <Goto "Template 4 Start"
    Enable this action to skip running this template>
    A40: [X] Goto [ 
        Type:Action Label 
        Number:1 
        Label:Template 4 Start ] 

    <Run `$PREFIX/bin/sudo -s --shell=python '%core_script' '%argument_1' '%argument_2'` Command In Background>
    A41: Anchor 

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

    <replace all single quotes (') with ('\'')>
    A46: Variable Search Replace [ 
        Variable:%core_script 
        Search:' 
        Ignore Case:Off 
        Multi-Line:On 
        One Match Only:Off 
        Store Matches In Array: 
        Replace Matches:On 
        Replace With:'\\'' ] If [ %core_script Set ]

    <replace all single quotes (') with ('\'')>
    A47: Variable Search Replace [ 
        Variable:%argument_1 
        Search:' 
        Ignore Case:Off 
        Multi-Line:On 
        One Match Only:Off 
        Store Matches In Array: 
        Replace Matches:On 
        Replace With:'\\'' ] If [ %argument_1 Set ]

    <replace all single quotes (') with ('\'')>
    A48: Variable Search Replace [ 
        Variable:%argument_2 
        Search:' 
        Ignore Case:Off 
        Multi-Line:On 
        One Match Only:Off 
        Store Matches In Array: 
        Replace Matches:On 
        Replace With:'\\'' ] If [ %argument_2 Set ]

    <set `-s --shell=python '%core_script' '%argument_1' '%argument_2'` to %arguments>
    A49: Variable Set [ 
        Name:%arguments 
        To:-s --shell=python '%core_script' '%argument_1' '%argument_2' 
        Recurse Variables:Off 
        Do Maths:Off 
        Append:Off 
        Max Rounding Digits:3 ] 

    A50: Variable Clear [ 
        Name:%command_failed/%errmsg/%stdout/%stderr/%result 
        Pattern Matching:On 
        Local Variables Only:On 
        Clear All Variables:Off ] 

    <Run Termux:Tasker Plugin Command>
    A51: Termux [ Configuration:%executable %arguments Timeout (Seconds):10 Continue Task After Error:On ] 

    A52: Variable Set [ 
        Name:%command_failed 
        To:err = `%err`
    
    errmsg =
    ```
    %errmsg
    ```
    
    exit_code = `%result`
    
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
        Max Rounding Digits:3 ] If [ %err Set | %errmsg Set | %result neq 0 ]

    A53: If [ %command_failed Set ]

        <remove %err and %errmsg if not set>
        A54: Variable Search Replace [ 
            Variable:%command_failed 
            Search:^err = `\%err`[\n]+errmsg =[\n]```[\n]\%errmsg[\n]```[\n]+ 
            Ignore Case:Off 
            Multi-Line:Off 
            One Match Only:Off 
            Store Matches In Array: 
            Replace Matches:On 
            Replace With: Continue Task After Error:On ] If [ %command_failed Set ]

        A55: Text Dialog [  
            Title:Template 3 Command
        Failed 
            Text:%command_failed 
            Button 1:OK 
            Button 2: 
            Button 3: 
            Close After (Seconds):30 
            Use HTML:Off ] 

        A56: Stop [ 
            With Error:Off 
            Task: ] 

    A57: Else 

        A58: Text Dialog [  
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

    A59: End If 

    <Template 3 End>
    A60: Anchor 

    <Template 4 Start>
    A61: Anchor 

    <Goto "Return"
    Enable this action to skip running this template>
    A62: [X] Goto [ 
        Type:Action Label 
        Number:1 
        Label:Return ] 

    <Run `$PREFIX/bin/sudo --shell-pre-commands="echo 'starting sudo shell';" --title='sudo' su` Command In Foreground>
    A63: Anchor 

    A64: Variable Clear [ 
        Name:%command_failed/%errmsg/%stdout/%stderr/%result 
        Pattern Matching:On 
        Local Variables Only:On 
        Clear All Variables:Off ] 

    <Run Termux:Tasker Plugin Command>
    A65: Termux [ Configuration:$PREFIX/bin/sudo --shell-pre-commands="echo 'starting sudo s Timeout (Seconds):10 Continue Task After Error:On ] 

    A66: Variable Set [ 
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

    A67: If [ %command_failed Set ]

        A68: Text Dialog [  
            Title:Template 4 Command
        Failed 
            Text:%command_failed 
            Button 1:OK 
            Button 2: 
            Button 3: 
            Close After (Seconds):30 
            Use HTML:Off ] 

        A69: Stop [ 
            With Error:Off 
            Task: ] 

    A70: End If 

    <Template 4 End>
    A71: Anchor 

    <Return>
    A72: Anchor 
``````

##
&nbsp;


*This file was automatically generated using [tasker_config_utils v0.5.0](https://github.com/Taskomater/tasker_config_utils).*
