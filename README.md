# GHAction for the Free Pascal Compiler

This GitHub Action allows you to compile Pascal programs using the Free Pascal Compiler.
It will parse the compiler's diagnostics and create annotations on your commits and pull requests.

## Inputs

| Name           | Required | Description                 | Default     |
| -------------- | :------: | --------------------------- | ----------- |
| `exclude-path` |          | List of paths to exclude.   |             |
| `fail-on`      |          | Strictness level.           | `e`         |
| `flags`        |          | Flags passed to FPC.        |             |
| `fpc`          |          | FPC executable to use.      | _see below_ |
| `source`       | Yes      | Main source file.           |             |
| `user-defined` |          | Show user-defined messages. | `true`      |
| `verbosity`    |          | Verbosity level.            | `ew`        |
| `workdir`      |          | Working directory.          |             |

### exclude-path

The `exclude-path` input can be used to ignore compiler messages pertaining to some files.
This is useful if you bundle some third-party libraries (or library headers) with your code
and want to exclude those files from generating annotations.

The value for this input is a list of paths, separated by a comma (`:`) on Linux and MacOS,
and by a semicolon (`;`) on MS Windows. Paths can be either absolute or relative.
Relative paths are resolved against the repository root, regardless of the `workdir` input.

If an entry ends with a slash (or backslash, on MS Windows) it is treated as a directory,
in which case all files in said directory (and its subdirectories) are ignored.
Otherwise, the entry is treated as a file name and a strict match is required.

### fail-on

The `fail-on` input can be used to control when the Action should fail.
By default, the Action will fail only if an error occurs (or if the compiler crashes).
If you want to be more strict with your code, you can use this option to have the Action
mark itself as "failed" when any warnings occur.

While FPC does have an option allowing for treating warnings as errors, `-Se` - and it even
allows you to treat not only warnings, but also notes and hints as errors - it has one significant drawback:
it enables "stop on first error" behaviour as well. Using `fail-on` with this Action allows you
to retain the "keep compiling until a fatal error occurs" behaviour and diagnose more warnings
during a single run of your CI workflow.

The value for this input follows the same rules as the one for `verbosity`.
Note that setting this input does **not** affect `verbosity` - if you use `ew` for `fail-on`,
you **must** have `ew` in your verbosity settings.

### flags

The `flags` input can be used to pass custom flags (i.e. options) to FPC. The passed string is interpreted
as a list of space-separated values. The leading dash on each flag can be omitted.

### fpc

The `fpc` input can be used to provide a full path to the Free Pascal Compiler executable.
When omitted, the Action behaves as follows:
- On Linux/MacOS: use `fpc`, i.e. rely on the executable being somewhere in `$PATH`.
- On Windows: the following list of directories is checked in search of an existing FPC installation.
  `X.Y.Z` stands for the version number of FPC. If multiple versions are found, the latest is preferred.
  * `C:\fpc\X.Y.Z`
  * `C:\Program Files\fpc\X.Y.Z`
  * `C:\Program Files (x86)\fpc\X.Y.Z`
  * `C:\lazarus\fpc\X.Y.Z`
  * `C:\Program Files\lazarus\fpc\X.Y.Z`
  * `C:\Program Files (x86)\lazarus\fpc\X.Y.Z`

### user-defined

The `user-defined` input can be used to control whether user-defined messages
(i.e. those produced using `{$FATAL}`, `${ERROR}`, `{$WARNING}`, `{$NOTE}` and `{$HINT}`
compiler directives) should be shown.

Note that, when a user-defined message causes the Action to fail (e.g. a `${FATAL}` directive is met,
or fail-on-warning is enabled and a `${WARNING}` directive is met), it **will** be shown,
even if this input was set to `false`.

### verbosity

The `verbosity` input can be used to control the desired verbosity level.
The value can be any combination of the following (case-insensitive) letters:
- `e` for errors
- `w` for warnings
- `n` for notes
- `h` for hints

Note that these are exclusive, i.e. a value of `n` will result in just the notes being printed,
without errors or warnings. You need `ewn` (or `new`, the order doesn't matter) if you want all three.

Implementation-wise, the flags passed to the Free Pascal compiler are `-v0 -vibXXX`,
where `XXX` is the value for this input.
As such, if you want to set the verbosity level through the `flags` input,
you **need** to set this input to an empty string - this disables adding the two `-v` flags.

## Getting FPC

This Action assumes that FPC is already installed in your build environment; it does not handle
installing it for you. The minimum required version is 2.1.2 (released March 2007).

You can install FPC yourself by adding one of the following steps to your GHActions workflow.

### Ubuntu

```
- name: Install FPC
  run: |
    export DEBIAN_FRONTEND=noninteractive
    sudo apt-get update
    sudo apt-get install -y fpc
```

### MacOS

```
- name: Install FPC
  run: |
    brew update
    brew install fpc
```

### MS Windows

As of the time of writing, Chocolatey does not have a separate package for FPC,
so you'll have to install Lazarus instead (it comes with a bundled copy of the compiler).

```
- name: Install Lazarus
  run: |
    choco install lazarus
```

## Licence

This Action is made available under the terms of the zlib licence.
For the full text of the licence, consult the `LICENCE` file.
