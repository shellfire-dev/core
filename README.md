# `core`: functions module for [shellfire]

This module provides an essential framework for every [shellfire] application. It is included without any extra code being required. However, there is a template for programs that needs to be completed to make use of it.

The framework includes functions for most common needs, difficulties and complexities when writing shell script. The major areas it covers are:-

TODO
core_dependency ?
core_variable
core_variable_array
core_commandLine

* [Core utilities](#namespace-core)

* [Base64 decoding](#namespace-core_base64)
* [Managing child processes](#namespace-core_children)
* [Compatibility across shells and distros](#namespace-core_compatibility)
* [Configuration](#namespace-core_configuration)
* [File reading utilities](#namespace-core_file)
* [Function manipulation, including existence and plugin registration](#namespace-core_functions)
* [Miscellaneous init functions](#namespace-core_init)
* [Path utility functions](#namespace-core_path)
* [Embedding snippets](#namespace-core_snippet)
* [Temporary File handling](#namespace-core_temporaryfiles)
* [Coloured terminal output](#namespace-core_terminal)
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


## Namespace `core_base64_decode`

This namespace allows you to base64 decode strings and files in a cross-script, cross-distro way irrespective of the foibles of whatever `base64` programs (or none) you may have.

### To use in code

This namespace is included by default.

### Functions

***
#### `core_base64_decode_string`
|Parameter|Value|Optional|
|---------|-----|--------|
|`string`|Base64 encoded string to decode.|_No_|
|`decodedFilePath`|Where to write decoded string to.|_No_|
|`append`|Boolean; `yes` to append to `decodedFilePath`, `no` to overwrite.|_No_|
|`index62Character`|Character for index 62 in base64. Typically `+`|_No_|
|`index63Character`|Character for index 63 in base64. Typically `\`|_No_|

Decoding to a file, `decodedFilePath`, sees counter-intuitive but is required because base64 encoded data can contain embedded ASCII NUL characters, which are incorrectly handled by most shells. `index*Character` allows you to use different base64 alphabets, such as base64url.

***
#### `core_base64_decode_file`
|Parameter|Value|Optional|
|---------|-----|--------|
|`encodedFilePath`|Base64 encoded file to decode.|_No_|
|`decodedFilePath`|Where to write decoded string to.|_No_|
|`append`|Boolean; `yes` to append to `decodedFilePath`, `no` to overwrite.|_No_|
|`index62Character`|Character for index 62 in base64. Typically `+`|_No_|
|`index63Character`|Character for index 63 in base64. Typically `\`|_No_|

Decodes `encodedFilePath` to `decodedFilePath`. `index*Character` allows you to use different base64 alphabets, such as base64url.

## Namespace `core_children`

This namespace exists to provide simple management of child processes.

### To use in code

This namespace is included by default.

### Functions

***
#### `core_children_killOnExit()`
|Parameter|Value|Optional|
|---------|-----|--------|
|`…`|Zero or more PIDs of child processes to kill on application exit|_Yes_|

After starting a background process, use this function to register child processes to clean up (kill) on exit using a `TERM` signal. For example:-

```bash
(
	sleep 1
	core_compatibility_echo 'tick-tock'
) &
core_children_killOnExit $?
```

## Namespace `core_compatibility`

This namespace exists to ensure cross-shell and cross `PATH` compatibility. It creates the functions `pushd` and `popd` if they don't exist, and, if they do, redirects their standard out to `/dev/null`.

### To use in code

This namespace is included by default.

### Functions

***
#### `core_compatibility_basename()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`path`|File path|_No_|

Writes the file portion of `path` to standard out. If `path` is just an unqualified string (eg `my-file`), writes `my-file`. If `path`
 is `/path/to/file`, writes `file`. Lastly, if file is `path/to/file`, writes `file`. Does not rely on the `basename` executable being on the `PATH`.

***
#### `core_compatibility_dirname()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`path`|File path|_No_|

Writes the directory portion of `path` to standard out. If `path` is just an unqualified string (eg `my-file`), writes `.`, otherwise writes everything before the final `/`. Does not rely on the `dirname` executable being on the `PATH`.

***
#### `core_compatibility_which()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`executable`|executable|_No_|

Acts like the `which` external command, but does not rely on it. Writes to standard out the full path to `executable` if it exists, otherwise writes to standard out `executable` if it is a function or built in. Returns an exit code of `0` if the `executable` exists (or is a function or built in) or returns `1` if not. Does not rely on the `which` executable being on the `PATH`.

***
#### `core_compatibility_whichNoOutput()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`executable`|executable|_No_|

Acts like `core_compatibility_which()` but writes nothing to standard out.

***
#### `core_compatibility_echo()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`message`|string|_No_|

Writes `message` followed by new line. Escape sequences in `message` are not acted on.

***
#### `core_compatibility_sleepSupportsFractionalSeconds()`

|Parameter|Value|Optional|
|---------|-----|--------|

_No parameters._

Returns an exit code of `0` if fractional values are accepted by `sleep`, otherwise returns `1`. Most modern versions of `sleep` support fractional seconds, except on AIX (whose user binaries are horrendously dated - and IBM has the gall to charge for it, too)!



## Namespace `core_configuration`

A [shellfire] application can have any of its command line options set in configuration files. These can be monolithic or composed of fragments (like Debian's run-parts). Or both. And they can exist in many locations. The approach lets an administrator of your application manage settings per-machine, per-user and per-environment; and they can still override settings per-run using command line options. Using fragments, sensitive passwords or credentials can be managed separately to more mundance configuration, so minimising failure when moving from development to production, say, or making it difficult to accidentally check in sensitive data to source control.

Nearly all the functions in this namespace are used internally to configure your [shellfire] application. The only one you need to know about is `core_configuration_blacklist`, which lets you state that an environment variable path is no longer to be overridden. And, if paranoid, you can also use `readonly` to prevent configuration settings from being changed.

Configuration files are just regular shell script, although it's advisable to avoid doing anything more mundane than:-

```bash
myapp_inputPath="/path/to/input"
myapp_outputPath="/path/to/output"
# etc

# But you can also blacklist, to prevent these locations being used
core_configuration_blacklist HOME SHELLFIRE_RC SHELLFIRE_RC_D
```

### Configuration Locations
Anything you can do with a command line switch, you can do as configuration. But configuration can also be used with scripts. Indeed, the configuration syntax is simply shell script. Configuration files _should not_ be executable. This means that if you _really_ want to, you can let an administrator override just about any feature or behaviour of a program - although that's not explicitly supported. Configuration can be in any number of locations. Configuration may be a single file, or a folder of files; in the latter case, every file in the folder is parsed in 'shell glob-expansion order' (typically ASCII sort order of file names). Locations are searched in order as follows:-

1. Global (Per-machine)
  1. The file `INSTALL_PREFIX/etc/shellfire/rc` where `INSTALL_PREFIX` is where `${_program_name}` has been installed.
  2. Any files in the folder `INSTALL_PREFIX/etc/shellfire/rc.d`
  3. The file `INSTALL_PREFIX/etc/${_program_name}/rc` where `INSTALL_PREFIX` is where `${_program_name}` has been installed.
  4. Any files in the folder `INSTALL_PREFIX/etc/${_program_name}/rc.d`
2. Per User, where `HOME` is your home folder path\*
  1. The file `HOME/.shellfire/rc`
  2. Any files in the folder `HOME/.shellfire/rc.d`
  3. The file `HOME/.${_program_name}/rc`
  4. Any files in the folder `HOME/.${_program_name}/rc.d`
3. Per Environment
  1. The file in the environment variable `shellfire_RC` (if the environment variable is set and the path is readable)
  2. Any files in the folder in the environment variable `shellfire_RC_D` (if the environment variable is set and the path is searchable)
  3. The file in the environment variable `${_program_name}_RC` (if the environment variable is set and the path is readable)
  4. Any files in the folder in the environment variable `${_program_name}_RC_D` (if the environment variable is set and the path is searchable)

Nothing stops any of these paths, or files in them, being symlinks.

_\* An installation as a daemon using a service account would normally set `HOME` to something like `/var/lib/bishbosh`._


### To use in code

This namespace is included by default. No additional actions are required.

### Functions


***
#### `core_configuration_blacklist`

|Parameter|Value|Optional|
|---------|-----|--------|
|`…`|Zero or more environment variables to blacklist|_Yes_|

Use this function inside a configuration file (such as a per-machine one like `INSTALL_PREFIX/etc/${_program_name}/rc` above) and specify any environment variables that should not be used to load configuration from. This prevents an user overriding machine-level configuration. The environment variables you can blacklist are:-

* `HOME`
* `shellfire_RC`
* `shellfire_RC_D`
* `${_program_name}_RC`
* `${_program_name}_RC_D`


## Namespace `core`

These are helper functions to manipulate files character-by-character in shells that don't support a sophisticated `read`.

### To use in code

This namespace is included by default. No additional actions are required. All of these functions are actually defined in `core/init.functions`.

### Functions

***
#### `core_message`

|Parameter|Value|Optional|
|---------|-----|--------|
|`messageKind`|Category of the message|_No_|
|`message`|Text of the message|_No_|

Writes `message` to standard out, with a prefix of `messageKind`. Colour-codes the message if standard out is a supported terminal. Does not write the message if the `messageKind` is not suitable for the current verbosity. Valid values of `messageKind` are:-

|`messageKind`|Written When|
|-------------|------------|
|`FAIL`|Always written|
|`WARN`|Always written|
|`NOTICE`|`core_init_verbosity()` is at least `1`|
|`INFO`|`core_init_verbosity()` is at least `2`|
|`TODO`|`core_init_verbosity()` is at least `2`|
|`DEBUG`|`core_init_verbosity()` is at least `3`|

If `messageKind`is unrecognised the message is always written.

***
#### `core_exitError`

|Parameter|Value|Optional|
|---------|-----|--------|
|`exitCode`|Non-zero exit code of the application, ideally one of the constants `core_commandLine_exitCode_*`|_No_|
|`message`|Text of the message|_No_|

Writes `message` as a `messageKind` of `FAIL` and then exits the application with exit code `exitCode`.

***
#### `core_TODO`

|Parameter|Value|Optional|
|---------|-----|--------|
|`message`|Text of the message|_No_|

Writes `message` as a `messageKind` of `TODO`. Useful for documenting incomplete features and being reminded when the application runs.

***
#### `core_uses`

|Parameter|Value|Optional|
|---------|-----|--------|
|`parentNamespace`|Absolute path containing functions to load|_No_|

_or_

|Parameter|Value|Optional|
|---------|-----|--------|
|`parentNamespace`|Root namsepace|_No_|
|`…`|Zero or more relative namespaces to load, specified as relative paths if nested, with no leading `/`. Should not end in `.functions`.|_Yes_|

This function should only be called from global scope or immediately inside the function `_program()`.

Examples:-
```bash
# Load the `urlencode` namespace from lib/shellfire/urlencode
core_usesIn urlencode
# Load the `urlencode_template` namespace from lib/shellfire/urlencode
core_usesIn urlencode template
# Load the `github_api_v3_releases` namespace
core_usesIn github/api/v3 releases
# Or, if two namespaces are wanted
core_usesIn github/api v3 v3/releases
```


***
#### `core_uses`

|Parameter|Value|Optional|
|---------|-----|--------|
|`libPath`|Absolute path containing functions to load|_No_|
|`libraryName`|A folder that should exist under `libPath`|_No_|
|`…`|Zero or more libraries to dynamically load as plug ins; specified as relative paths if nested, with no leading `/`. Should not end in `.functions`.|_Yes_|

This is an advanced function for implementing which can be used to implement run-time plugin loading. Handles circular dependencies. Can cause havoc; use with care.



## Namespace `core_file`

These are helper functions to manipulate files character-by-character in shells that don't support a sophisticated `read`.

### To use in code

This namespace is included by default. No additional actions are required.

### Functions

***
#### `core_file_characterByCharacter`

|Parameter|Value|Optional|
|---------|-----|--------|
|`filePath`|A file to read character-by-character|_No_|
|`…`|Zero or more function names to use as callbacks|_Yes_|

Iterates over a file character-by-character. For each character, it calls each callback in `…` with the variable `character` set to the character. If the character is a line feed, then `character` is empty (this is a shell limitation, but not a problem in use).

***
#### `core_file_characterByCharacterCreate`

|Parameter|Value|Optional|
|---------|-----|--------|
|`inputFile`|A file to read character-by-character|_No_|
|`outputFile`|A file to read character-by-character|_No_|

Converts a file into a set of lines (line feed delimited), each of which contains one character (or, in the case of line feed, no character).



## Namespace `core_functions`

These are helper functions to manipulate shell functions.

### To use in code

This namespace is included by default. No additional actions are required.

### Functions

***
#### `core_functions_exist`

|Parameter|Value|Optional|
|---------|-----|--------|
|`functionName`|A name of a function|_No_|

Returns an exit code of `0` if the function (not binary on the `PATH`) exists in the current program, and `1` if it doesn't.

***
#### `core_functions_register`

|Parameter|Value|Optional|
|---------|-----|--------|
|`functionsVariableName`|A variable to use to store the function names|_No_|
|`…`|Zero or more function names|_Yes_|

Appends zero or more function names to use as callbacks, in order. A function can be added more than once. The list of functions will be included in any [fatten]ed binary. Use this to register callbacks and plugins on first load of modules. This function should only be called from global scope or immediately inside the function `_program()`. `functionsVariableName` is created and initialised as an array if it doesn't exist.

***
#### `core_functions_execute`

|Parameter|Value|Optional|
|---------|-----|--------|
|`functionsVariableName`|A variable to use to store the function names|_No_|
|`…`|Zero or more arguments to pass to callbacks|_Yes_|

This function executes all functions registered in `functionsVariableName` as callbacks, passing the value of `…` as arguments to them.


## Namespace `core_init`

These are mostly functions used internally.

### To use in code

This namespace is included by default. No additional actions are required.

### Functions

***
#### `core_init_defines`

|Parameter|Value|Optional|
|---------|-----|--------|
|`…`|Zero or more functions or global constants|_Yes_|

This function should only be called from global scope or immediately inside the function `_program()`. It is used to document that a function or global constant, that should be found inside a functions file, is actually to be found in `init.functions`.

***
#### `core_init_verbosity`

|Parameter|Value|Optional|
|---------|-----|--------|

_No parameters._

This function writes to standard out a positive integer number representing the current verbosity level set by `--verbose` for a [shellfire] application. It is typically used to decide if to output debugging information, eg

```bash
if [ "$(core_init_verbosity)" -gt 2 ]; then
	cat "/path/to/some/file" 1>&2
fi
```

If using `core_message()`, then this is checked internally - there's no need to guard calls to `core_message()` with this function.

***
#### `core_init_language`

|Parameter|Value|Optional|
|---------|-----|--------|

_No parameters._

This function writes to standard out the current language setting, or `en_US.UTF-8` if there isn't one. Useful if you need to manipulate language sensitive programs which don't use `LC_*` or `LANG` variables (which we set automatically).


## Namespace `core_path`

These are utility functions that check for certain properties of paths (files, folders, that sort of thing). Unlike the `core_validate` namespace functions, these ones do not exit the application. Instead, they return an exit code, and so are intended to be used from constructs such as `if` statements or `&&`.

For example, to check if a path is a readable, searchable folder path:-

```bash
if ! core_path_isReadableAndSearchableFolderPath '/etc'; then
	core_message WARN "'/etc' is not a readable, searchable folder path. This is very odd."
fi
```

### To use in code

This namespace is included by default. No additional actions are required.

### Functions

***
#### `core_path_isReadableFilePath()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`filePath`|Path to a potential file|_No_|

Returns an exit code of `0` if `filePath` is a readable, extant file or `1` if it is not.


***
#### `core_path_isReadableNonEmptyFilePath()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`filePath`|Path to a potential file|_No_|

Returns an exit code of `0` if `filePath` is a readable, non-empty extant file or `1` if it is not.

***
#### `core_path_isReadableNonEmptyExecutableFilePath()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`filePath`|Path to a potential file|_No_|

Returns an exit code of `0` if `filePath` is a readable, non-empty executable extant file or `1` if it is not.

***
#### `core_path_isReadableAndSearchableFolderPath()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`folderPath`|Path to a potential folder|_No_|

Returns an exit code of `0` if `folderPath` is a readable, searchable extant folder or `1` if it is not.

***
#### `core_path_isReadableAndSearchableAndWritableFolderPath()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`folderPath`|Path to a potential folder|_No_|

Returns an exit code of `0` if `folderPath` is a readable, searchable and writable extant folder or `1` if it is not.







## Namespace `core_snippet`

This namespace allows you to embed plain text and binary snippets inside shell scripts - a bit like shell archives. Snippets can be retrieved and used as temporary files, variable values or even interpreted as here docs (they make excellent simple templates)! Snippets let you separate functionality from data, and so source control and version data separately. They also let you deploy unusual compiled binaries with your code - just make sure you've statically linked everything…

Binary snippets are base64 encoded. Plain text snippets are embedded as is (they are called raw), although trailing line feeds are stripped on retrieval due to fundamental shell limitations - watch out! For advanced use cases, it is possible to define your own storage codec. Perhaps Ascii85? If so, please contribute a patch if others could benefit.

Snippets are stored in `lib/shellfire/${_program_name}` as files ending in `.snippet`. File names can consist only of A-Z, a-z, 0-9 and underscores to maximise cross-shell compatibility. For a `_program_name=example`, you might use a snippet as follows:-

```bash
core_snippet_embed raw my_snippet
example_useMySnippet()
{
	# Extract to a file
	# 'no' here means do not append. Specify 'yes' to append to the end of /path/to/extract/to
	core_snippet_retrieve my_snippet 'no' /path/to/extract/to
	
	# Extract to a temporary file
	local TMP_FILE
	core_temporaryFiles_newFileToRemoveOnExit
	local mySnippetTemporaryFilePath="$TMP_FILE"
	core_snippet_retrieve my_snippet 'no' "$mySnippetTemporaryFilePath"
	
	# Use as a heredoc - make sure any functions $(somefunc) and variables ${somevar} are defined, or your program will exit!
	# This allows you to store templates as snippets
	core_snippet_retrieveAndSourceAsHereDoc my_snippet
}
```
By convention, the embed statement 'decorates' the function that'll use the snippet. The codec here is `raw` - it could also be `base64`.

### To use in code

This namespace is included by default. No additional actions are required.

### Functions

***
#### `core_snippet_embed()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`codecName`|`raw` or `base64`|_No_|
|`…`|Zero or more `snippetName` snippet names|_Yes, although at least one is needed to be useful_|

You _must_ call this function _only_ from global scope or immediately inside the function `_program()`. If you define your own codec, then you must define the functions `core_snippet_${codecName}_encoder` and `core_snippet_${codecName}_decoder`. These do not take arguments, but the variables `snippetName` and `snippetFilePath` are in scope (and `snippetAppend`, a boolean, for `core_snippet_${codecName}_decoder`).

***
#### `core_snippet_retrieve()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`snippetName`|Name of the snippet used in `core_snippet_embed()`|_No_|
|`snippetAppend`|Specify `no` to overwrite any existing file or `yes` to append to the end.|_No_|
|`snippetFilePath`|Where to write the snippet to.|_No_|

Retrieves a snippet and writes it to `snippetFilePath`. If the `raw` codec was used when embedding the snippet with `core_snippet_embed()`, then the resultant data will have any trailing line feeds stripped and ASCII NUL characters will have been ignored.

***
#### `core_snippet_retrieveAndSourceAsHereDoc()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`snippetName`|Name of the snippet used in `core_snippet_embed()`|_No_|

Retrieves a snippet and treats it as if it was a here doc (ie `<<EOF`). If the `raw` codec was used when embedding the snippet with `core_snippet_embed()`, then the resultant data will have any trailing line feeds stripped and ASCII NUL characters will have been ignored.


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






## Namespace `core_terminal`

This namespace exposes functions to safely write coloured strings to a terminal on a file descriptor. If a terminal is not present, it outputs plain text without terminal escape sequences, so standard error (or whatever else) can be redirected to a file, logged, etc. It has a small list of known ANSI VT102 (ECMA-48) compliant terminals to which it outputs escape sequences directly. If your terminal isn't listed, it falls back to the `tput` program, and then to plain text. The odd reason for this counter-intuitive, apparently anti-best-practice behaviour is that `tput` is frequently missing (especially on BusyBox based systems), broken or incorrectly set up (as is commonly the case, especially on AIX and the BSDs).

If you are using a terminal that supports ANSI escape sequences and isn't supported, please submit a pull request.

### To use in code

This namespace is included by default. No additional actions are required. Please note that currently all functions are defined in `core/init.functions` as they are used during [shellfire] application bootstrapping.

### Functions

***
#### `core_terminal_ansiSupported()`

|Parameter|Value|Optional|
|---------|-----|--------|

_No parameters._

Used to determine if a terminal supports ANSI escape sequences. Only currently used if `tput` is not available. Modify this function to detect your ANSI terminal type if we don't support it. Returns an exit code of `0` is supported and `1` otherwise.

***
#### `core_terminal_tput()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`…`|One or more tput arguments|_No_|

Defensive wrapping of `tput` in case it is missing, broken or incompletely installed.

***
#### `core_terminal_colour()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`fd`|File descriptor to check is a terminal. Use `1` for standard out and `2` for standard error.|_No_|
|`ground`|What to colour: `foreground` or `background`|_No_|
|`colour`|File descriptor to check is a terminal|_No_|

`ground` may be one of:-

|Value|Description|
|-----|-----------|
|`foreground`|Colour the foreground|
|`background`|Colour the background|

`colour` may be one of:-

|Value|
|-----|
|`black`|
|`red`|
|`green`|
|`yellow`|
|`blue`|
|`magenta`|
|`cyan`|
|`white`|
|`default`|

This functions write an escape sequence to standard out. Either redirect it the file descriptor specified in `fd`, or capture it in a string for latter use, eg

```bash
coloured="My $(core_terminal_colour 2 foreground red)red$(core_terminal_reset) string"
printf '%s\n' "$coloured" 1>&2
```

***
#### `core_terminal_effect()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`fd`|File descriptor to check is a terminal. Use `1` for standard out and `2` for standard error.|_No_|
|`…`|Zero or more effect names|_Yes, but specify at least one for to be useful!_|

`ground` may be one of:-

|Value|Description|
|-----|-----------|
|`foreground`|Colour the foreground|
|`background`|Colour the background|

`effect` may be one of:-

|Value|
|-----|
|`bold`|
|`dim`|
|`blink`|
|`reversed`|
|`invisible`|

This functions write an escape sequence to standard out. Either redirect it the file descriptor specified in `fd`, or capture it in a string for latter use, eg

```bash
affected="My $(core_terminal_effect 2 bold blink)bold, blinking$(core_terminal_reset) string"
printf '%s\n' "$affected" 1>&2
```

***
#### `core_terminal_reset()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`fd`|File descriptor to check is a terminal. Use `1` for standard out and `2` for standard error.|_No_|

Resets all terminal escape sequences by writing to standard out. Use this as in the examples above.

***




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
|`…`|Zero or more signal names|_Yes, but specify at least one for to be useful!_|

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