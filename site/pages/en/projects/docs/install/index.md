---
page_ref: "@ARK_PROJECT__VARIANT@/agnostic-apollo/sudo/docs/@ARK_DOC__VERSION@/install/index.md"
---

# sudo Install Docs

<!-- @ARK_DOCS__HEADER_PLACEHOLDER@ -->

The [`sudo`](https://github.com/agnostic-apollo/sudo) install instructions are available below. For build instructions, check [`build`](../developer/build/index.md) docs.

### Contents

- [Install Dependencies](#install-dependencies)
- [Install Sources](#install-sources)

---

&nbsp;





## Install Dependencies

Using `sudo` in Termux shells only has the following dependencies.

- [Termux](https://github.com/termux/termux-app) app version: minimum `>= 0.100`, **recommended `>= 0.119.0`**.
- [`bash`](https://www.gnu.org/software/bash/manual/bash.html) version: `>= 4.1`.
- A `su` binary that supports the `-c`, `--shell`, `--preserve-environment` and `--mount-master` options. [Magisk](https://github.com/topjohnwu/Magisk) and [SuperSU](https://su.chainfire.eu/) are recommended and their `su` binaries support the required options. Note that the [limited `su` provided by Android debug builds](https://cs.android.com/android/platform/superproject/+/master:system/extras/su/su.cpp) for `adb root` does not support these options. Check [`su` search paths](../usage/index.md#su-search-paths) for more info.

However, to use `sudo` with [Termux:Tasker](https://github.com/termux/termux-tasker) plugin app and [RUN_COMMAND Intent](https://github.com/termux/termux-app/wiki/RUN_COMMAND-Intent) requires the following app versions to be installed. Check [Passing Arguments](../usage/index.md#passing-arguments) section and [Termux:Tasker `Setup Instructions`](https://github.com/termux/termux-tasker#setup-instructions) section for details.

- [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm) app version: `>= 5.9.4.beta`
- [Termux:Tasker](https://github.com/termux/termux-tasker) app version: `>= 0.5`

---

&nbsp;





## Install Sources

`sudo` can be installed from the following sources.

- [Termux Packages Repository](#termux-packages-repository)
- [GitHub](#github)

**The [Termux Packages Repository](#termux-packages-repository) is the recommended way to install `sudo`.**

The Termux app version `>= v0.119.0` exports path environment variables like `$TERMUX_ROOTFS`, `$TERMUX_HOME` and `$TERMUX_PREFIX` under the `TERMUX_` env root scope (`$TERMUX_ENV__S_ROOT`). `sudo` version `>= 1.0.0` uses these dynamic paths instead of relying on hardcoded `com.termux` paths, which will not work if Termux is built for a different rootfs or for a fork. In future, multiple rootfs support may be added in Termux app as well. The hardcoded `com.termux` paths refers to `/data/data/com.termux/*` paths for the [upstream Termux app](https://github.com/termux/termux-app) with the `com.termux` app package name.

- If running in Termux app version `< 0.119.0` where environment variables are not exported:  
    - If `sudo` is built with [Termux Packages Build Infrastructure](../developer/build/index.md#termux-packages-build-infrastructure), then placeholder path values that are replaced during build time are used.  
    - If `sudo` is built [On Device With `make`](../developer/build/index.md#on-device-with-make) or downloaded from [`master` branch](https://github.com/agnostic-apollo/sudo/blob/master/sudo) or [GitHub Releases](https://github.com/agnostic-apollo/sudo/releases), then hardcoded `/data/data/com.termux/*` paths are used.  
- If running in Termux app version `>= 0.119.0` where environment variables are exported:  
    - If `sudo` is built with [Termux Packages Build Infrastructure](../developer/build/index.md#termux-packages-build-infrastructure), then environment variables are used if set, with a fallback to placeholder path values that are replaced during build time.  
    - If `sudo` is built [On Device With `make`](../developer/build/index.md#on-device-with-make) or downloaded from [`master` branch](https://github.com/agnostic-apollo/sudo/blob/master/sudo) or [GitHub Releases](https://github.com/agnostic-apollo/sudo/releases), then environment variables are used if set, with a fallback to hardcoded `/data/data/com.termux/*` paths.  

Hence, if `sudo` is not built with the [Termux Packages Build Infrastructure](../developer/build/index.md#termux-packages-build-infrastructure), then it will not work in Termux app forks that have not merged the changes done in Termux app version `>= 0.119.0` for exporting path environment variables into their own apps.

Termux variables are set in the `sudo_set_termux_env_variables` function of the `sudo` script, which internally calls the [`termux_gtsev_bash__get_termux_scoped_env_variable()`](https://github.com/termux/termux-packages//blob/master/packages/termux-core/src/scripts/termux_gtsev_bash__get_termux_scoped_env_variable) function that is used by the [`termux-get-termux-scoped-env-variable.bash`](https://github.com/termux/termux-core-package/blob/v0.1.0/src/scripts/termux-get-termux-scoped-env-variable.bash.in) util provided by the `termux-core` package. The `@*@` placeholders (like `@TERMUX__ROOTFS@`) in the `sudo_set_termux_env_variables` function exist so that they are replaced during build time by the Termux build infrastructure through the `Makefile`. Check [`termux-get-termux-scoped-env-variable` usage docs](https://github.com/termux/termux-core-package/blob/master/site/pages/en/projects/docs/usage/utils/.md) for more info.

## &nbsp;

&nbsp;



### Termux Packages Repository

To install `sudo` package from the [`main` channel](https://github.com/termux/termux-packages/blob/master/packages/sudo/build.sh) of the [Termux packages repository](https://packages.termux.dev), run the following commands.

```shell
pkg install sudo
```

## &nbsp;

&nbsp;



### GitHub

The `sudo` file should be installed in termux `bin` directory `${TERMUX__PREFIX:-$PREFIX}/bin`. It should have `Termux app (u<userid>_a<appid>)` user (like `u0_a100`) `uid:gid` ownership and have executable (`700`/`rwx------`) permission before it can be executed.

&nbsp;

#### Basic

**The Basic way is the recommended way to install `sudo` from GitHub, especially if you have limited understanding of shell commands.**

[`curl`](https://curl.se) will automatically be installed to download `sudo`.

Download `sudo` from the **`latest` [GitHub Releases](https://github.com/agnostic-apollo/sudo/releases)** and convert it into an executable. Copy all the commands and run them together in the shell.

```shell
pkg install curl && \
install_dir="${TERMUX__PREFIX:-$PREFIX}/bin" && \
mkdir -p "$install_dir" && \
rm -f "$install_dir/sudo" && \
curl -L 'https://github.com/agnostic-apollo/sudo/releases/latest/download/sudo' -o "$install_dir/sudo"
owner="$(stat -c "%u" "$install_dir")" && chown "$owner:$owner" "$install_dir/sudo" && \
chmod 700 "$install_dir/sudo" && \
termux-fix-shebang "$install_dir/sudo"
```

&nbsp;

#### Advance

1. Set install directory path and create it and delete any existing `sudo` file.  

```shell
install_dir="${TERMUX__PREFIX:-$PREFIX}/bin" && \
mkdir -p "$install_dir" && \
rm -f "$install_dir/sudo"
```

2. Download the `sudo` file. If using [`curl`](https://curl.se), then it must be installed to download `sudo`. Run `pkg install curl` to install `curl`.  

    - **Using `curl` from [GitHub Releases](https://github.com/agnostic-apollo/sudo/releases)** with a non-root shell.  
        - Latest release:  

            `curl -L 'https://github.com/agnostic-apollo/sudo/releases/latest/download/sudo' -o "$install_dir/sudo"`  

        - Specific release:  

            `curl -L 'https://github.com/agnostic-apollo/sudo/releases/download/v0.1.0/sudo' -o "$install_dir/sudo"`  

    - **Manually from [GitHub Releases](https://github.com/agnostic-apollo/sudo/releases)** listed under the `Assets` dropdown menu to the android download directory and then copy it to install directory using `cat` command or a root file browser. Termux app must have `storage` permission if running `cat` command.  

       `cat "/storage/emulated/0/Download/sudo" > "$install_dir/sudo"`  

    - **Using `curl` from [`master` branch](https://github.com/agnostic-apollo/sudo/tree/master)** (*may be unstable*) with a non-root shell.  

        `curl -L 'https://github.com/agnostic-apollo/sudo/raw/master/sudo' -o "$install_dir/sudo"`  

3. Set `Termux app (u<userid>_a<appid>)` user ownership and executable (`700`/`rwx------`) permission.  

    - If you used `curl` or `cat` to copy the file, then use a non-root shell to set ownership and permissions with `chown` and `chmod` commands respectively:  

        `owner="$(stat -c "%u" "$install_dir")" && chown "$owner:$owner" "$install_dir/sudo" && chmod 700 "$install_dir/sudo";`  

    - If you used a root file browser to copy the file, then use `su` to start a root shell to set ownership, permissions and SeLinux context ([1](https://github.com/agnostic-apollo/Android-Docs/blob/master/site/pages/en/projects/docs/os/selinux/security-context.md), [2](https://github.com/agnostic-apollo/Android-Docs/blob/master/site/pages/en/projects/docs/os/selinux/context-types.md)) for Termux user with `chown`, `chmod` and `restorecon` commands respectively:  

    	`owner="$(stat -c "%u" "$install_dir")" && su -c "chown \"$owner:$owner\" \"$install_dir/sudo\" && chmod 700 \"$install_dir/sudo\" && restorecon \"$install_dir/sudo\"";`  

4. Replace `#!/usr/bin/bash` shebang in `sudo` script with path to `bash` under the current Termux rootfs `bin` path by running [`termux-fix-shebang`](https://github.com/termux/termux-tools/blob/master/scripts/termux-fix-shebang.in) on it so that `sudo` works properly even if `libtermux-exec.so` is not set in `$LD_PRELOAD`. Check [`termux-exec` `Linux vs Termux `bin` paths`](https://github.com/termux/termux-exec-package/blob/master/site/pages/en/projects/docs/technical/index.md#linux-vs-termux-bin-paths) and [`termux-tasker` `Termux Environment`](https://github.com/termux/termux-tasker#termux-environment) docs for more info.

    `termux-fix-shebang "$install_dir/sudo"`


---

&nbsp;
