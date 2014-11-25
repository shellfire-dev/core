# `core`: functions module for [shellfire]

This module provides an essential framework for every [shellfire] application. It is included without any extra code being required. However, there is a template for programs that needs to be completed to make use of it.

The framework includes functions for most common needs, difficulties and complexities when writing shell script. The major areas it covers are:-

* [Temporary File handling](#namespace-core_temporaryfiles)
* [Signal Handling](#namespace-core_trap)
* [umasks](#namespace-core_umask)
* [Argument validation](#namespace-core_validate)

## Overview

### Overridding a module's function(s)

If, for some reason, one of the functions isn't to your own liking then you can override its definition. Simply provide your own inside your [shellfire] application's `_program()` function. This override will then also be used for [fatten]ing automatically. Of course, we'd encourage you to submit a pull request to improve our definition for the benfit of everybody.

### Namespaces

Shell script doesn't have namespaces. [shellfire] does. And by the simple way folks have done it in C for generations. With an underscore and a prefix.

Functions are namespaced, that is, they are prefixed by a hierarchial names separated by underscores, '`_`', so they don't collide. Wherever possible, all the functions for a particular namespace are in a file named after the namespace, in a folder path related to it. For instance, the function `core_variable_array_initialise()` is in the namespace `core_variable_array` and is located in the file `core/variable/array.functions`.

To use functions belonging to a namespace, all one has to do is `use` it, eg `core_usesIn core/variable array`. If there is only a single hierarchy, as there is for some modules (eg [urlencode]), then the syntax is `core_usesIn urlencode`. This logic automatically finds and loads the functions, and their dependencies, and so on. It can't enter a circular loop, either. And when a [shellfire] application is [fatten]ed (by which we make a complete standalone shell script from all these namespaces), only those functions so loaded are included.

### A few criticial functions aren't where you'd expect

Some functions in `core` are so important that they are used to bootstrap a [shellfire] application; this is called `init`. These functions live in `init.functions`. However, many of them are also useful during the runtime of a [shellfire] application, and naturally belong to a specific namespace, eg the function `core_variable_isSet()` logically belongs to `core_variable`. Where this is the case, the corresponding file, in this case `core/variable/variable.functions` will contain a one-line entry `core_init_defines core_variable_isSet` documenting the fact.

These are those functions:-

#### `core_compatibility`

* `core_compatibility_basename()`
* `core_compatibility_dirname()`
* `core_compatibility_which()`
* `core_compatibility_whichNoOutput()`

#### `core_variable`

* `core_variable_isSet()`
* `core_variable_isUnset()`
* `core_variable_indirectValue()`
* `core_variable_unset()`

#### `core_dependency`

* `core_dependency_declares()`
* `core_dependency_requires()`
* `core_dependency_oneOf()`
* `core_dependency_fallback()`

#### `core_terminal`

* `core_terminal_ansiSupported()`
* `core_terminal_ansiSequence()`
* `core_terminal_tput()`
* `core_terminal_colour()`
* `core_terminal_effect()`
* `core_terminal_reset()`

#### `core`

* `core_message()`
* `core_exitError()`
* `core_TODO()`
* `core_uses()`
* `core_usesIn()`


## Importing

To import this module, add a git submodule to your repository. From the root of your git repository in the terminal, type:-
```bash
mkdir -p lib/shellfire
cd lib/shellfire
git submodule add "https://github.com/shellfire-dev/core.git"
cd -
git submodule init --update
```

You may need to change the url `https://github.com/shellfire-dev/core.git` above if using a fork.

You will also need to add paths - include the module [paths.d].

## Namespace `core_temporaryFiles`

This namespace exposes two simple functions to manage temporary files and folders. It abstracts away all the variability of different operating systems, BSDs and Linux distros. Under the covers, it handles signals and manages arrays so that temporary resources are always cleaned up, no matter what. Of course, you're still free to `rm` a file when you're done with it - a good idea if you need many temporary files.

### To use in code

This namespace is included by default. No additional actions are required.

### Functions

***
#### `core_temporaryFiles_newFileToRemoveOnExit()`

|Parameter|Value|Optional|
|---------|-----|--------|

_No parameters._

On return, sets the variable `TMP_FILE` to the path to an extant temporary file with no content. Uses the most secure temporary file approach available, with as random-as-possible names (securely so on Linux, Mac OS X and FreeBSD), and restricted file permissions (writable and readable only by the current user).

Ideally, you should make `TMP_FILE` local and then copy the value immediately, as a subsequent call to `core_temporaryFiles_newFileToRemoveOnExit()` will overwrite it (this can happen if using recursion, as local variables are inheritated in shell script). Example:-

```bash
local TMP_FILE
core_temporaryFiles_newFileToRemoveOnExit
ourTemporaryFilePath="$TMP_FILE"
```

***
#### `core_temporaryFiles_newFolderToRemoveOnExit()`

|Parameter|Value|Optional|
|---------|-----|--------|

_No parameters._

On return, sets the variable `TMP_FOLDER` to the path to an extant temporary folder. Uses the most secure temporary file approach available, with as random-as-possible names (securely so on Linux, Mac OS X and FreeBSD), and restricted file permissions (writable, readable and searchable only by the current user).

Ideally, you should make `TMP_FOLDER` local and then copy the value immediately, as a subsequent call to `core_temporaryFiles_newFolderToRemoveOnExit()` will overwrite it (this can happen if using recursion, as local variables are inheritated in shell script). Example:-

```bash
local TMP_FOLDER
core_temporaryFiles_newFolderToRemoveOnExit
ourTemporaryFolderPath="$TMP_FOLDER"
```


## Namespace `core_trap`

This namespace exposes functions to manage signals (traps). It also sets up and registers internal logic to make clean up of common resources straightforward and error-free.

### To use in code

This namespace is included by default. No additional actions are required.

### Functions

***
#### `core_trap_addOnCleanUp()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`handler`|The name of a handler function to callback|_No_|

Use this function to register a callback for signals that cause the application to exit, and where a clean up action is needed. There is no need to use this method for temporary file clean up; that's internally registered.

***
#### `core_trap_addHandler()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`handler`|The name of a handler function to callback|_No_|
|`…`|Zero or more signal names|_Yes, but specify at least one for to be useful!_

Use this function to register a callback for zero or more signals. Signal names that are cross-platform are:-

|Signal Name|
|-----|
|`EXIT`|
|`HUP`|
|`INT`|
|`QUIT`|
|`ABRT`|
|`PIPE`|
|`TERM`|
|`TSTP`|
|`USR1`|
|`USR2`|
|`INFO`|

***

## Namespace `core_umask`

This namespace is used internally to ensure a sensible default umask of `0022` is set.

### To use in code

This namespace is included by default. No additional actions are required.

### Functions

There are no public functions.


## Namespace `core_validate`

This namespace exposes functions that validate a supplied value. If the validation fails, the program exits. The intended use case is for validating command line and configuration arguments at program initialisation. Some [shellfire] applications supplement these with additional validate functions in their own namespaces; by convention, this is done as `${_program_name}_validate_FUNCTIONNAME` in a file `${_program_name}/validate.functions`.

All of the functions for validation follow the same pattern, and take the same four arguments: `code`, `category`, `name` and `value`. For example, to validate that the `--output-path` command line switch was given a valid folder path, you might do this inside your [shellfire] application, `example`:-

```bash
_program_commandLine_processOptionWithArgument()
{
	case "$optionName" in
		
		…
		
		o|output-path)
			core_validate_folderPathIsReadableAndSearchableAndWritableOrCanBeCreated $core_commandLine_exitCode_USAGE 'option' "$optionNameIncludingHyphens" "$optionValue"
			example_outputPath="$optionValue"
		;;
		
		…
		
	esac
}
```

Of course, it'd be nice if users could specify a default `--output-path` in a configuration file. They can do this by creating, say `/etc/example/rc` (one of many possibilities) with the content:-

```bash
example_outputPath='/var/tmp/example'
```

You'd then want to be able to validate that the configured value _or_ the value specified on the command line was right. Inside your [shellfire] application, you could add the following:-

```bash
_program_commandLine_validate()
{
	…

	if core_variable_isSet example_outputPath; then
		core_validate_folderPathIsReadableAndSearchableAndWritableOrCanBeCreated $core_commandLine_exitCode_CONFIG 'configuration setting' 'example_outputPath' "$example_outputPath"
	else
		core_message INFO "Defaulting --output-path to current working directory"
		example_outputPath="$(pwd)"
	fi
	
	…
}
```

This works because we follow a _convention_: a command-line _long_ option is named `${_program_name}_lowercaseUppercase`. If your program was `overdrive` and you had a long option `--number-of-gears`, then the option would be `overdrive_numberOfGears`.

### Conventions

#### `code`
Conventions for `code` are:-

* Never use `0`
* Avoid `1` and `2`
* Always use the most apt value from the `commandLine` interface exit codes (`core_commandLine_exitCode_*`)
* Never use values above `127`
* Use `core_commandLine_exitCode_USAGE` if `category` is `option`
* Use `core_commandLine_exitCode_CONFIG` if `category` is `configuration setting`

#### `category`
Conventions for `category` are:-

* `option` for command line options (switches) such as `-o` and `--output-path` above
* `configuration setting` for configuration settings such as `example_outputPath`
* `argument` if being used for validating function arguments (not a common use case)

### To use in code

This namespace is included by default. No additional actions are required.

### Functions

***
#### `core_validate_pathNotEmpty()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`code`|An exit code (should not be zero). Ideally use one of the constants `core_commandLine_exitCode_*`.|_No_|
|`category`|What you are validating.|_No_|
|`name`|Name of what you are validating, eg `--output-path` or `example_outputPath`|_No_|
|`value`|Raw, supplied value of what you are validating|_No_|

Use this function to validate that a path (`value`) is not empty (`''`) . Exits with `code` if validation fails.

***
#### `core_validate_folderPathReadableAndSearchable()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`code`|An exit code (should not be zero). Ideally use one of the constants `core_commandLine_exitCode_*`.|_No_|
|`category`|What you are validating.|_No_|
|`name`|Name of what you are validating, eg `--output-path` or `example_outputPath`|_No_|
|`value`|Raw, supplied value of what you are validating|_No_|

Use this function to validate that a path (`value`) is a readable, searchable (and so extant) folder. Exits with `code` if validation fails.

***
#### `core_validate_folderPathReadableAndSearchableAndWritable()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`code`|An exit code (should not be zero). Ideally use one of the constants `core_commandLine_exitCode_*`.|_No_|
|`category`|What you are validating.|_No_|
|`name`|Name of what you are validating, eg `--output-path` or `example_outputPath`|_No_|
|`value`|Raw, supplied value of what you are validating|_No_|

Use this function to validate that a path (`value`) is a readable, searchable, writable (and so extant) folder. Exits with `code` if validation fails.

***
#### `core_validate_parentFolderPathReadableAndSearchable()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`code`|An exit code (should not be zero). Ideally use one of the constants `core_commandLine_exitCode_*`.|_No_|
|`category`|What you are validating.|_No_|
|`name`|Name of what you are validating, eg `--output-path` or `example_outputPath`|_No_|
|`value`|Raw, supplied value of what you are validating|_No_|

Use this function to validate that the parent of the path (`value`) is a readable, searchable (and so extant) folder. Exits with `code` if validation fails.

***
#### `core_validate_parentFolderPathReadableAndSearchableAndWritable()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`code`|An exit code (should not be zero). Ideally use one of the constants `core_commandLine_exitCode_*`.|_No_|
|`category`|What you are validating.|_No_|
|`name`|Name of what you are validating, eg `--output-path` or `example_outputPath`|_No_|
|`value`|Raw, supplied value of what you are validating|_No_|

Use this function to validate that the parent of the path (`value`) is a readable, searchable, writable (and so extant) folder. Exits with `code` if validation fails. Useful for checking that a path that _might_ be created will succeed.

***
#### `core_validate_folderPathIsReadableAndSearchableAndWritableOrCanBeCreated()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`code`|An exit code (should not be zero). Ideally use one of the constants `core_commandLine_exitCode_*`.|_No_|
|`category`|What you are validating.|_No_|
|`name`|Name of what you are validating, eg `--output-path` or `example_outputPath`|_No_|
|`value`|Raw, supplied value of what you are validating|_No_|

Use this function to validate that the the path (`value`) is a readable, searchable, writable (and so extant) folder _or_ it can be created. Exits with `code` if validation fails. Useful for things like output paths.


***
#### `core_validate_filePathReadable()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`code`|An exit code (should not be zero). Ideally use one of the constants `core_commandLine_exitCode_*`.|_No_|
|`category`|What you are validating.|_No_|
|`name`|Name of what you are validating, eg `--output-path` or `example_outputPath`|_No_|
|`value`|Raw, supplied value of what you are validating|_No_|

Use this function to validate that the the path (`value`) is a readable (and so extant) file. Exits with `code` if validation fails.


***
#### `core_validate_filePathReadableAndExecutable()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`code`|An exit code (should not be zero). Ideally use one of the constants `core_commandLine_exitCode_*`.|_No_|
|`category`|What you are validating.|_No_|
|`name`|Name of what you are validating, eg `--output-path` or `example_outputPath`|_No_|
|`value`|Raw, supplied value of what you are validating|_No_|

Use this function to validate that the the path (`value`) is a readable, executable (and so extant) file. Exits with `code` if validation fails.


***
#### `core_validate_filePathReadableAndExecutableAndNotEmpty()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`code`|An exit code (should not be zero). Ideally use one of the constants `core_commandLine_exitCode_*`.|_No_|
|`category`|What you are validating.|_No_|
|`name`|Name of what you are validating, eg `--output-path` or `example_outputPath`|_No_|
|`value`|Raw, supplied value of what you are validating|_No_|

Use this function to validate that the the path (`value`) is a readable, executable (and so extant), not empty file (as most executables should be). Exits with `code` if validation fails.

***
#### `core_validate_socketPathReadableAndWritable()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`code`|An exit code (should not be zero). Ideally use one of the constants `core_commandLine_exitCode_*`.|_No_|
|`category`|What you are validating.|_No_|
|`name`|Name of what you are validating, eg `--output-path` or `example_outputPath`|_No_|
|`value`|Raw, supplied value of what you are validating|_No_|

Use this function to validate that the the path (`value`) is a readable, writable (and so extant) Unix domain socket. Exits with `code` if validation fails.

***
#### `core_validate_characterDeviceFileReadableAndWritable()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`code`|An exit code (should not be zero). Ideally use one of the constants `core_commandLine_exitCode_*`.|_No_|
|`category`|What you are validating.|_No_|
|`name`|Name of what you are validating, eg `--output-path` or `example_outputPath`|_No_|
|`value`|Raw, supplied value of what you are validating|_No_|

Use this function to validate that the the path (`value`) is a readable, writable (and so extant) character device. Exits with `code` if validation fails.

***
#### `core_validate_isBoolean()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`code`|An exit code (should not be zero). Ideally use one of the constants `core_commandLine_exitCode_*`.|_No_|
|`category`|What you are validating.|_No_|
|`name`|Name of what you are validating, eg `--output-path` or `example_outputPath`|_No_|
|`value`|Raw, supplied value of what you are validating|_No_|

Use this function to validate that the the `value` is a Boolean. Exits with `code` if validation fails.

Of course, there are no such things as booleans in shell script, but we can model booleans as strings. Booleans are used in many places in [shellfire], as they document behaviour much better.

A `value` is a boolean and true if it is
* `true`, `TRUE`, `True` or `T`
* `yes`, `YES`, `Yes`, `Y`
* `on`, `ON`, `On`
* `1`

A `value` is a boolean and false if it is
* `false`, `FALSE`, `False` or `F`
* `no`, `NO`, `No`, `N`
* `off`, `OFF`, `Off`
* `0`


***
#### `core_validate_isUnsignedInteger()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`code`|An exit code (should not be zero). Ideally use one of the constants `core_commandLine_exitCode_*`.|_No_|
|`category`|What you are validating.|_No_|
|`name`|Name of what you are validating, eg `--output-path` or `example_outputPath`|_No_|
|`value`|Raw, supplied value of what you are validating|_No_|

Use this function to validate that the the `value` is an unsigned integer. Exits with `code` if validation fails. An unsigned integer is considered to be any unprefix decimal value greater than or equal to 0.



***
#### `core_validate_nonDynamicPort()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`code`|An exit code (should not be zero). Ideally use one of the constants `core_commandLine_exitCode_*`.|_No_|
|`category`|What you are validating.|_No_|
|`name`|Name of what you are validating, eg `--output-path` or `example_outputPath`|_No_|
|`value`|Raw, supplied value of what you are validating|_No_|

Use this function to validate that the the `value` is a non-dynamic port. Exits with `code` if validation fails. A non-dynamic port is one between 1 and 65535 inclusive.









[swaddle]: https://github.com/raphaelcohn/swaddle "Swaddle homepage"
[shellfire]: https://github.com/shellfire-dev "shellfire homepage"
[fatten]: https://github.com/shellfire-dev/fatten "fatten homepage"
[core]: https://github.com/shellfire-dev/core "shellfire core module homepage"
[paths.d]: https://github.com/shellfire-dev/paths.d "paths.d shellfire module homepage"
[github api]: https://github.com/shellfire-dev/github "github shellfire module homepage"
[jsonwriter]: https://github.com/shellfire-dev/jsonwriter "jsonwriter shellfire module homepage"
[jsonreader]: https://github.com/shellfire-dev/jsonreader "jsonreader shellfire module homepage"
[xmlwriter]: https://github.com/shellfire-dev/xmlwriter "xmlwriter shellfire module homepage"
[unicode]: https://github.com/shellfire-dev/unicode "unicode shellfire module homepage"
[version]: https://github.com/shellfire-dev/version "version shellfire module homepage"