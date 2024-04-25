---
page_ref: "@ARK_PROJECT__VARIANT@/agnostic-apollo/sudo/docs/@ARK_DOC__VERSION@/developer/test/index.md"
---

# sudo Test Docs

<!-- @ARK_DOCS__HEADER_PLACEHOLDER@ -->

[`sudo_tests`](https://github.com/agnostic-apollo/sudo/blob/master/tests/sudo_tests) can be used to run tests for [`sudo`](https://github.com/agnostic-apollo/sudo).

Check the `sudo_tests` file for setup instructions, or run it with the `--help-extra` option.

If `sudo` is installed with a Termux package manager, then `sudo_tests` gets installed at `$TERMUX__PREFIX/libexec/installed-tests/sudo/sudo_tests`.

To show help, run `${TERMUX__PREFIX:-$PREFIX}/libexec/installed-tests/sudo/sudo_tests --help-extra`.

To run all tests, run `${TERMUX__PREFIX:-$PREFIX}/libexec/installed-tests/sudo/sudo_tests -vv`. Tests will only be run for whatever shells are currently installed. The `--only-bash-tests` option can be passed to only test `bash` shell. Additionally, the `su`, `asu`, `path`, or `script` can be passed to only run tests for specific command types.

---

&nbsp;





## Help

```
sudo_tests is a script that run tests for the sudo script.


Usage:
  sudo_tests [command_options]
  sudo_tests [command_options] <su|asu|path|script>
  sudo_tests [command_options] <su|asu|path|script> <shell> <test_number>


Available command_options:
  [ -h | --help ]    Display this help screen.
  [ --help-extra ]   Display more help about how sudo_tests command
                     works.
  [ -q | --quiet ]   Set log level to 'OFF'.
  [ -v | -vv ]       Set log level to 'DEBUG', 'VERBOSE', `VVERBOSE'.
  [ --only-bash-tests ] Run only bash shell tests.


Setting log level '>= DEBUG' will fail test validation for tests that
match the output, its mainly for debugging.
```

---

&nbsp;
