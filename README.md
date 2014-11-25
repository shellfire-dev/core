# `core`: functions module for [shellfire]

This module provides an essential framework for every [shellfire] application. It is included without any extra code being required. However, there is a template for programs that needs to be completed to make use of it.

The framework includes functions for most common needs, difficulties and complexities when writing shell script. The major areas it covers are:-

* [Temporary File handling](#namespace-core_temporaryFiles)
* [Umasks](#namespace-core_umask)
* [Argument validation](#namespace-core_validate)

## Overview

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