# `core`: functions module for [shellfire]

This module provides an essential framework for every [shellfire] application. It is included without any extra code being required. However, there is a template for programs that needs to be completed to make use of it.

The framework includes functions for most common needs, difficulties and complexities when writing shell script. The major areas it covers are:-

* [Core utilities](#namespace-core)
* [Base64 decoding](#namespace-core_base64)
* [Command Line option parsing, including long options, help messages and exit codes](#namespace-core_commandline)
* [Managing child processes](#namespace-core_children)
* [Compatibility across shells and distros](#namespace-core_compatibility)
* [Configuration](#namespace-core_configuration)
* [Binary dependency management and documentation](#namespace-core_dependency)
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
* [Variable and Value Helpers](#namespace-core_variable)
* [Arrays, even in non-bash shells](#namespace-core_variable_array)

## Overview

### Overridding a module's function(s)

If, for some reason, one of the functions isn't to your own liking then you can override its definition. Simply provide your own inside your [shellfire] application's `_program()` function. This override will then also be used for [fatten]ing automatically. Of course, we'd encourage you to submit a pull request to improve our definition for the benfit of everybody.

### Namespaces

Shell script doesn't have namespaces. [shellfire] does. And by the simple way folks have done it in C for generations. With an underscore and a prefix.

Functions are namespaced, that is, they are prefixed by a hierarchial names separated by underscores, '`_`', so they don't collide. Wherever possible, all the functions for a particular namespace are in a file named after the namespace, in a folder path related to it. For instance, the function `core_variable_array_initialise()` is in the namespace `core_variable_array` and is located in the file `core/variable/array.functions`.

To use functions belonging to a namespace, all one has to do is `use` it, eg `core_usesIn core/variable array`. If there is only a single hierarchy, as there is for some modules (eg [urlencode]), then the syntax is `core_usesIn urlencode`. This logic automatically finds and loads the functions, and their dependencies, and so on. It can't enter a circular loop, either. And when a [shellfire] application is [fatten]ed (by which we make a complete standalone shell script from all these namespaces), only those functions so loaded are included.

A very small number of user definable definitions (see next section) are not in any namespace. These are 'namespaceless'.

If a variable or function starts with an underscore, `_`, then it is intended that its use is private to that namespace. The exception to this is `_program`, which is used for user definable definitions.

### User Definable Definitions
When a global constant or function starts `_program_`, it is intended to be defined by the [shellfire] application _outside_ of the `_program()` function. Most global functions are optional; defaults are assumed. Most global constants need to be defined. The [shellfire tutorial 'overdrive'](https://github.com/shellfire-dev/shellfire/tree/master/tutorial) shows all the common settings. Each namespace adds `_program_` definitions as `_program_<namespace>_`. Some `_program_` definitions are outside of any namespace - these are 'namespaceless'.

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
* `core_dependency_declaresAsArray()`
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

## Namespaceless


### User-Definable Constants

User-Definable constants are intended to be implemented in the global scope of your [shellfire] application. Most of these constants are mandatory.

|User Definable Constant|Optional?|Purpose|Example|Notes|
|-----------------------|---------|-------|-------|-----|
|`_program_name`|_No_|Name of your program once [fatten]ed|`_program_name='overdrive'`|Ideally, make this match the file name|
|`_program_version`|_No_|Version of your program once [fatten]ed|`_program_name='unversioned'`|Always specify `unversioned`|
|`_program_package_or_build`|_No_|branch or package variant details|`_program_package_or_build=''`|Always specify `''`. Used by custom packagers.|
|`_program_copyrightAndLicenseStatement`|_Yes_|Copyright and license statement. Warranties, disclaimers and authorship.|May include line feeds.|[fatten]ing will embed your `COPYRIGHT` file if not specified.|
|`_program_path`|_No_|Location of `_program_*Path` sub paths|`_program_path="$([ "${_program_fattening_program_path+set}" = 'set' ] && printf '%s\n' "$_program_fattening_program_path" || ([ "${0%/*}" = "${0}" ] && printf '%s\n' '.' || printf '%s\n' "${0%/*}"))"`|Only used by un[fatten]ed programs and during [fattten]ing. Not used once [fatten]ed. Example doesn't work for pdksh derivatives.|
|`_program_libPath`|_No_|Location of `lib`|`_program_libPath="${_program_path}/lib"`||
|`_program_etcPath`|_No_|Location of `etc`|`_program_etcPath="${_program_path}/etc"`||
|`_program_varPath`|_No_|Location of `var`|`_program_varPath="${_program_path}/var"`||
|`_program_entrypoint`|_No_|Entry point for your program|`_program_entrypoint='overdrive'`|Ensure a function `overdrive()` exists inside `_program()`|
|`_program_namespace`|_Yes_|Namespace of your program's functions|`_program_entrypoint='overdrive'`|Defaults to `_program_name`, but needed if this contains anything other than A-Z, a-z, 0-9 and underscore (eg hyphens)|
|`_program_arrayDelimiter`|_Yes_|Override the array delimited in use to support older shells|`_program_arrayDelimiter=','`|Defaults to `\r`. Override with `,` to support MinGW / MSYS Bash 3.1.7 and GoW. Use "$(printf '\001')" if supporting bash 4.2+, ash or dash and you need to deal with data containing `\r`.|

***

### User-Definable Functions

#### `_program()`

Mandatory. Define all your functions inside this, and any calls to `core_usesIn()`, `core_dependency_requires`, etc. Your entry point function (see above section [User-Definable Constants](#user-definable-constants), `_program_entrypoint`) should be in here.

***
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







## Namespace `core_commandLine`

This namespace is mostly invisible, but provides a very rich and sophisticated command line parser so that you can use mixed short and long options, varargs, optional arguments and more. It provides semi-automatic help message generation, including licensing information. It does not depend on any external `getopt` programs.

All of its behaviour is controlled by defining `_program_*` functions and setting `_program_*` variables in your [shellfire] application. By default, it includes several common options for `--help`, `--verbose` and `--version` use and parsing. Options can be 'golfed' (eg `-abc4` is supported, and can be interpreted as `-a bc4`, `-a -b -c -4` and `-a -b c4`, etc, depending on what's defined) but this isn't recommend as it's ambiguous. XFree86-style (eg as used by openssl) options with a single hyphen aren't supported. `--` for end-of-options is, too.

Command line parsing happens after configuration has been loaded, so that command line options can override configured values. Implementations can take advantage of this to note if a value was set by configuration or command line when issuing error messages.

### To use in code

This namespace is included by default.

### Global Constants

#### Exit Codes

This constants provide values for common exit conditions, and align with the BSD sysexits.h values, stdlib.h values and a value used by bash.

|Constant|Value|Explanation|
|--------|-----|-----------|
|core_commandLine_exitCode_OK|0|successful termination (aka EXIT_SUCCESS in stdlib.h)|
|core_commandLine_exitCode_FAILURE|1|Not in sysexist.h, but so commonly used (aka EXIT_FAILURE in stdlib.h)|
|core_commandLine_exitCode_MISUSEBUILTIN|2|Not in sysexits.h, misuse of builtin (bash specific)|
|core_commandLine_exitCode_USAGE|64|command line usage error|
|core_commandLine_exitCode_DATAERR|65|data format error|
|core_commandLine_exitCode_NOINPUT|66|cannot open input|
|core_commandLine_exitCode_NOUSER|67|addressee unknown|
|core_commandLine_exitCode_NOHOST|68|host name unknown|
|core_commandLine_exitCode_UNAVAILABLE|69|service unavailable|
|core_commandLine_exitCode_SOFTWARE|70|internal software error|
|core_commandLine_exitCode_OSERR|71|system error (e.g., can't fork)|
|core_commandLine_exitCode_OSFILE|72|critical OS file missing|
|core_commandLine_exitCode_CANTCREAT|73|can't create (user) output file|
|core_commandLine_exitCode_IOERR|74|input/output error|
|core_commandLine_exitCode_TEMPFAIL|75|temp failure; user is invited to retry|
|core_commandLine_exitCode_PROTOCOL|76|remote error in protocol|
|core_commandLine_exitCode_NOPERM|77|permission denied|
|core_commandLine_exitCode_CONFIG|78|configuration error|

These constants are _not_ `readonly`, as this doesn't work in some ksh derivatives.

### Functions

There are no public functions.

### User-Definable Functions

User-Definable functions are intended to be implemented in the global scope of your [shellfire] application. You are not obligated to implement any of these functions; if you don't, a naive default is used instead.

***
#### `_program_commandLine_parseInitialise()`

|Parameter|Value|Optional|
|---------|-----|--------|

_No parameters._

Implement this function if you need to define some variables or perform any actions before the command line is parsed. A typical use case is to define default values to be used if an option isn't specified and isn't configured. It can also be used to set any variables that wold be used in generated help message (see below).

***
#### `_program_commandLine_helpMessage()`

|Parameter|Value|Optional|
|---------|-----|--------|

_No parameters._

Implement this function to argment the help message that is displayed when a user asks for help (`-h` or `--help`), or a command line parse error occurs. When this function is executed by the parser, you can set some `_program_commandLine_helpMessage_*` variables that can then be used to provide a richer help:-

|Variable|Use|Example|
|--------|---|-------|
|`_program_commandLine_helpMessage_usage`|A one line usage message|`[OPTION]... -- [SCRIPTLETS]...`|
|`_program_commandLine_helpMessage_description`|Multi-line description|`Connects to a MQTT server and runs SCRIPTLETS.\nMay be used as a command interpreter in scripts starting #!/usr/bin/env ${_program_name}\nif on the PATH.`\*|
|`_program_commandLine_helpMessage_options`|Multi-line options|See below†|
|`_program_commandLine_helpMessage_optionsSpacing`|Space characters to align things|`   `|
|`_program_commandLine_helpMessage_configurationKeys`|Multi-line configuration settings|See below‡|
|`_program_commandLine_helpMessage_examples`|A one line example usage|`${_program_name} --server test.mosquitto.org`|

Note that using variables such as `${_program_name}` allows you to change your program name in the future without search-replace fury.

\* The `'\n` represents a line feed; in shell script, you'd write:-

```bash
_program_commandLine_helpMessage_description="Connects to a MQTT server and runs SCRIPTLETS.
May be used as a command interpreter in scripts starting #!/usr/bin/env ${_program_name}
if on the PATH."
```

† Multi-line options:-

```bash
_program_commandLine_helpMessage_options="
-t, --tunnel TUNNEL       Network tunnel TUNNEL to use. One of 'none', 'tls',
                        'cryptcat'. Defaults to '${_program_default_tunnel}'.
                        'WebSockets', 'WebSocketsSecure', 'SSH' and 'telnet'
                        are unimplemented but possible if there's demand.
-s, --server HOST         Server hostname, HOST, is a hostname,
                        IPv4 address, IPv6 address, unix domain socket
                        path or serial character device path."
```

‡ Multi-line configuration settings:-

```bash
_program_commandLine_helpMessage_configurationKeys="
  bishbosh_tunnel         Equivalent to --tunnel
  bishbosh_server         Equivalent to --server
  bishbosh_port           Equivalent to --port
"
```

***
#### `_program_commandLine_optionExists()`

|Parameter|Value|Optional|
|---------|-----|--------|

_No parameters._

Implement this function to tell the command line parser that a short or long option exists, and, if it does, what kind of argument it takes. The variable `optionName` is in scope, and contains the command line option without any leading hyphens and without any trailing spaces or equals sign, `=`. This function is usually implemented using a `case` statement.

The function should write to standard out one of four values:-

|Value|Description|
|-----|-----------|
|`yes-argumentless`|Option exists but does not take an argument (`--help` would be an example)|
|`yes-argumented`|Option exists but requires an argument|
|`yes-optionally-argumented`|Option exists but may have an argument. Use this with care, as it can lead to ambiguous results.|
|`no`|Option doesn't exist.|

The options `h`, `help`, `v`, `verbose`, `version` are handled for you and can not be overridden.

An example illustrates best:-

```bash
_program_commandLine_optionExists()
{
	case "$optionName" in
		
		# c would be -c
		# clean would be --clean
		c|clean)
			echo yes-argumentless
		;;
		
		# eg -t mytunnel
		# eg --tunnel=mytunnel
		# eg --tunnel mytunnel
		# If there are additional equals signs, they're handled correctly, eg
		# eg --tunnel=mytunnel=something
		t|tunnel)
			echo 'yes-argumented'
		;;
		
		tunnel-tls-use-der)
			echo 'yes-optionally-argumented'
		;;
		
		*)
			echo 'no'
		;;
		
	esac
}
```

***
#### `_program_commandLine_processOptionWithoutArgument()`

|Parameter|Value|Optional|
|---------|-----|--------|

_No parameters._

Implement this function to be able to act on an option you specified as either `yes-argumentless` or `yes-optionally-argumented` (and which had no argument). The variable `optionName` is in scope. For example:-

```bash
_program_commandLine_processOptionWithoutArgument()
{
	case "$optionName" in
		
		r|random-client-id)
			bishbosh_randomClientId=1
		;;

		tunnel-tls-use-der)
			bishbosh_tunnelTlsUseDer=1
		;;
		
		tunnel-tls-verify)
			bishbosh_tunnelTlsVerify=1
		;;
		
	esac
}
```

There is not need for a default `*` case as it can not occur.

***
#### `_program_commandLine_processOptionWithArgument()`

|Parameter|Value|Optional|
|---------|-----|--------|

_No parameters._

Implement this function to be able to act on an option you specified as either `yes-argumentless` or `yes-optionally-argumented` (and which had no argument). The variable `optionName` is in scope. For example:-

```bash
_program_commandLine_processOptionWithoutArgument()
{
	case "$optionName" in
		
		r|random-client-id)
			bishbosh_randomClientId=1
		;;

		tunnel-tls-use-der)
			bishbosh_tunnelTlsUseDer=1
		;;
		
		tunnel-tls-verify)
			bishbosh_tunnelTlsVerify=1
		;;
		
	esac
}
```
There is not need for a default `*` case as it can not occur.

May be called more than once for a given `optionName` if it is present more than once on the command line. 

***
#### `_program_commandLine_processOptionWithArgument()`

|Parameter|Value|Optional|
|---------|-----|--------|

_No parameters._

Implement this function to be able to act on an option you specified as `yes-argumented`. The variables `optionName` and `optionValue` are in scope. For example:-

```bash
_program_commandLine_processOptionWithArgument()
{
	case "$optionName" in
		
		p|port)
			core_validate_nonDynamicPort $core_commandLine_exitCode_USAGE 'option' "$optionNameIncludingHyphens" "$optionValue"
			bishbosh_port="$optionValue"
		;;
		
		t|tunnel)
			bishbosh_validate_tunnel $core_commandLine_exitCode_USAGE 'option' "$optionNameIncludingHyphens" "$optionValue"
			bishbosh_tunnel="$optionValue"
		;;
		
		s|server)
			bishbosh_validate_address $core_commandLine_exitCode_USAGE 'option' "$optionNameIncludingHyphens" "$optionValue"
			bishbosh_server="$optionValue"
		;;
		
	esac
}
```

There is not need for a default `*` case as it can not occur. Note the use of the `core_validate` namespace and a private `${_program_name}_validate` namespace.

May be called more than once for a given `optionName` if it is present more than once on the command line.

***
#### `_program_commandLine_handleNonOptions()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`…`|Zero or more non options|_Yes in the sense that there may be zero_|

The command line parse calls this function once for the gathered set of non-options (ie all the values after `--` in the command line or when option parsing has ended) in the command line.

Use the value as necessary; you can add them to a cross-shell compliant array with `core_variable_array_append my_non_options_array "$@"`.

This example treats the non options as shell-source:-

```bash
_program_commandLine_handleNonOptions()
{
	local scriptletFilePath
	local suitableForSourceBuiltInFilePath
	for scriptletFilePath in "$@"
	do
		suitableForSourceBuiltInFilePath=$(core_compatibility_dirname "$scriptletFilePath")/"$(core_compatibility_basename "$scriptletFilePath")"
		. "$suitableForSourceBuiltInFilePath" || core_exitError "Could not source SCRIPTLET '$scriptletFilePath'"
	done
}
```

***
#### `_program_commandLine_validate()`

|Parameter|Value|Optional|
|---------|-----|--------|

_No parameters._

The command line parser calls function after all parsing is complete. Use this function to check for things like:-

* a mandatory command line option has been passed
* a given combination of options is valid
* an option wasn't passed, but was set in configuration, so now we should validate the configuration value

Typically, this logic makes calls to functions in the `core_validate` namespace. For example:-

```bash
_program_commandLine_validate()
{
	if core_variable_isUnset bishbosh_sessionPath; then
		# _program_default_sessionPath might be specified in _program_commandLine_parseInitialise if it is a computed value
		bishbosh_sessionPath="$_program_default_sessionPath"
		core_validate_folderPathReadableAndSearchableAndWritable $core_commandLine_exitCode_CONFIG 'defaulted value for' '--session-path' "$bishbosh_sessionPath"
	else
		core_validate_folderPathReadableAndSearchable $core_commandLine_exitCode_CONFIG 'configuration setting' 'bishbosh_sessionPath' "$bishbosh_sessionPath"
	fi
}
```

***





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







## Namespace `core_dependency`

This namespace is mostly invisible but provides a powerful framework to:-

* automatically install dependent executables (in packages) using distro-specific package managers
* modify the `PATH` so that known-good executables are used
* document global variables that need to be included in any [fatten]ed [shellfire] application.
* document complex requirements

### To use in code

This namespace is included by default.

### Functions

***
#### `core_dependency_declares()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`…`|Zero or more global variables in to include in a [fatten]ed program|_Yes_|

The processing of [fatten]ing a program needs to know if any global variables are required. This function should only be called from global scope or immediately inside the function `_program()` with the names of any variables. Commonly, this done immediately after declaring and setting a global variable (or constant). This is how the `core_commandLine` namespace exposes global constants for exit codes. For example:-

```bash
# command line usage error
core_commandLine_exitCode_USAGE=64
core_dependency_declares core_commandLine_exitCode_USAGE
```

***
#### `core_dependency_declaresAsArray()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`…`|Zero or more global array variables in to include in a [fatten]ed program|_Yes_|

This function operates like `core_dependency_declares()` above, but is for array variables. It exists because the declaration of a global array that needs to be included in a [fatten]ed program can occur before the array is defined.


***
#### `core_dependency_requires()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`packageManager`|Name of a package manager, or `'*` for all known package managers.|_Yes_|
|`…`|Zero or more executables|_Yes_|

This function should only be called from global scope or immediately inside the function `_program()` with the names of any variables.

This function informs the dependency management framework that a cerain executable (binary), as name in `…`, should be on the `PATH`. If it isn't, it arranges at application bootstrap time for it to be downloaded and installed. This depends on a `.path` file in `etc/shellfire/paths.d/${packageManager}`. For example, for the executable `dd` on Mac OS X, there should be `etc/shellfire/paths.d/Homebrew/dd.path` (this ensures that the `dd` is the same across multiple platforms with the same switches). It is usually used to 'decorate' a function (a bit like a Java annotation), so that there is an obvious and logical relationship to a function's implementation and its external dependencies.

The `packageManager` denotes if this dependency is needed for all operating environments (`'*'`) or just a particular one (eg `Homebrew`).

For example:-
```bash
core_dependency_requires '*' dd
my_function()
{
	dd if=/path/to/input/file of=/path/to/output/file
}
```

Using this approach, it becomes very easy to find all your programs dependencies and document them, even if you don't use the automatic installation process. A veru useful thing to do it you're looking to fine tune your BusyBox configuration, say.


***
#### `core_dependency_fallback()`

Design subject to change; do not use. Explicitly not fully documented. Intended to provide a way to document fallbacks.

***
#### `core_dependency_oneOf()`

Design may be subject to change; use with caution. Explicitly not fully documented. Intended to provide a way to document alternatives.


***

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
* `defaulted value for` if being used for a default when no configuration or command line option was set, and the default isn't valid (eg when an option might be a folder path, and the default folder path doesn't exist)

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

***

## Namespace `core_variable`

This namespace provides a wide range of helper functions to work with variables and values.

### To use in code

This namespace is included by default. No additional actions are required.

### Functions

***
#### `core_variable_isSet()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|A variable name|_No_|

Returns a `0` exit code if the variable is set (defined) (but not empty or null), or a non-zero one if it isn't. Be aware that there is a bug in bash 3.2 on Mac OS X when invoked as `sh` that treats a defined, but not set `local` variable, as if it is empty.

***
#### `core_variable_isUnset()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|A variable name|_No_|

Returns a `0` exit code if the variable is unset (undefined) (but not empty or null), or non-zero one if it is. Be aware that there is a bug in bash 3.2 on Mac OS X when invoked as `sh` that treats a defined, but not set `local` variable, as if it is empty.

***
#### `core_variable_indirectValue()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|A variable name|_No_|

Sets the variable `core_variable_indirectValue_result` to the value pointed to by `variableName`. Exists because most shells don't provide a way to do this simply. Deliberately does not write to standard out, even though that would have been easier, as the caller, needs to capture standard out would then strip trailing line feeds and so end up with a different variable value.

***
#### `core_variable_unset()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|A variable name|_No_|

Unsets (undefines) the variable pointed to by `variableName`. This method exists because the normal `unset` built-in misbehaves on `pdksh` when trying to unset non-existent variables (even if `set +u` is used)! One could argue that these shells are more correct, but, since the common ones (bash included) don't behave this way, this is the only cross-shell way to be consistent. We do not redefine the `unset` built in as this doesn't work in `pdksh` (which, despite ceasing development in 1999, is still widely used in the BSDs).

***
#### `core_variable_setVariable()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|A variable name|_No_|
|`value`|A value to set it to|_No_|

Sets the variable pointed to by `variableName` to `value`.


***
#### `core_variable_isUnsetOrEmpty()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|A variable name|_No_|

Returns a `0` exit code if the variable is unset (undefined) or empty (null in Shell speak!), or non-zero one if it is. Be aware that there is a bug in bash 3.2 on Mac OS X when invoked as `sh` that treats a defined, but not set `local` variable, as if it is empty.

***
#### `core_variable_setVariableIfUnset()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|A variable name|_No_|
|`variableDefaultValue`|A value to set it to|_No_|

Sets the variable pointed to by `variableName` to `value` if, and only if, it is unset.

***
#### `core_variable_contains()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|
|`contains`|A value to check for|_No_|

Returns a `0` exit code if the `value` contains `contains`, or non-zero one otherwise.

***
#### `core_variable_matches()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|
|`match`|A value to match|_No_|

Returns a `0` exit code if the the shell glob expression `match` matches the `value`, or non-zero one otherwise. Avoid using bashisms.

***
#### `core_variable_startsWith()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|
|`startsWith`|A value to match|_No_|

Returns a `0` exit code if the `value` starts with `startsWith`, or non-zero one otherwise.

***
#### `core_variable_doesNotStartWith()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|
|`startsWith`|A value to match|_No_|

Returns a `0` exit code if the `value` does not start with `startsWith`, or non-zero one otherwise.

***
#### `core_variable_endsWith()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|
|`endsWith`|A value to match|_No_|

Returns a `0` exit code if the `value` ends with `endsWith`, or non-zero one otherwise.

***
#### `core_variable_doesNotEndWith()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|
|`endsWith`|A value to match|_No_|

Returns a `0` exit code if the `value` does not end with `endsWith`, or non-zero one otherwise.

***
#### `core_variable_firstCharacter()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|

Writes to standard out the first character of `value`.

***
#### `core_variable_lastCharacter()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|

Writes to standard out the last character of `value`.

***
#### `core_variable_trimSpaceAndHorizontalTab()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|

Writes to standard out `value` with any leading or trailing spaces and horizontal tabs removed (POSIX character class `[:blank:]`).

***
#### `core_variable_trimWhitespace()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|

Writes to standard out `value` with any leading or trailing whitespace removed (POSIX character class `[:space:]`).

***
#### `core_variable_allButLastN()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|
|`numberToOmit`|Number of characters to remove|_No_|

Writes to standard out `value` with the trailing `numberToOmit` characters removed.

***
#### `core_variable_allButLast()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|

Writes to standard out `value` with the final character removed. Useful for stripping a trailing `/` from path prefixes.

***
#### `core_variable_allButFirstN()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|
|`numberToOmit`|Number of characters to remove|_No_|

Writes to standard out `value` with the leading `numberToOmit` characters removed.

***
#### `core_variable_allButFirst()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|

Writes to standard out `value` with the leading character removed. Useful for stripping a leading `/` from path prefixes.

***
#### `core_variable_characterByCharacter()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|
|`callback`|A callback (function name)|_No_|

Calls `callback` for each character `character` of `value`. Inefficient. Useful for building any manner of parsers in shell script, though.

***
#### `core_variable_isTrue()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|

Returns an exit code of:-

* `0` if `value` is a boolean true (`T`, `t`, `true`, `True`, `TRUE`, `Y`, `y`, `yes`, `Yes`, `YES`, `on`, `On`, `ON`, `1`)
* `1` if `value` is a boolean false (`F`, `f`, `false`, `False`, `FALSE`, `N`, `n`, `no`, `No`, `NO`, `off`, `Off`, `OFF`, `0`)
* `2` if `value` is not valid

***
#### `core_variable_isFalse()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|

Returns an exit code of:-

* `0` if `value` is a boolean false (`F`, `f`, `false`, `False`, `FALSE`, `N`, `n`, `no`, `No`, `NO`, `off`, `Off`, `OFF`, `0`)
* `1` if `value` is a boolean true (`T`, `t`, `true`, `True`, `TRUE`, `Y`, `y`, `yes`, `Yes`, `YES`, `on`, `On`, `ON`, `1`)
* `2` if `value` is not valid

***
#### `core_variable_isInvalidBoolean()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`value`|A value|_No_|

Returns an exit code of:-

* `0` if `value` is not valid
* `1` if `value` is a boolean false (`F`, `f`, `false`, `False`, `FALSE`, `N`, `n`, `no`, `No`, `NO`, `off`, `Off`, `OFF`, `0`)
* `1` if `value` is a boolean true (`T`, `t`, `true`, `True`, `TRUE`, `Y`, `y`, `yes`, `Yes`, `YES`, `on`, `On`, `ON`, `1`)














***

## Namespace `core_variable_array`

This namespace provides a wide range of helper functions to pretend variables are arrays. It uses a bespoke separator character. There is experimental support to make bash arrays appear to work under this pretence. This module works hard to overcome serious bugs in bash 3.2 when run as `sh` (unset variables are empty; most control code separate characters don't work). Do not use arrays with file paths if there's any chance they could contain `\r`.

To use arrays in global scope, one does:-

```bash
# Optional, core_variable_array_append initialises if necessary
core_variable_array_initialise myArray

# Append values - spaces are correctly handled
core_variable_array_append myArray 'value1' 'value 2'
core_variable_array_append myArray 'etc'

length=$(core_variable_array_length myArray)
if core_variable_array_isEmpty myArray; then
	echo Empty!
fi

# will be `value 2`
itemAtIndex1="$(core_variable_array_at myArray 1)"

# will be 'value1, value2, etc'
echo "$(core_variable_array_string myArray ', ')"

# Use to call an executable or function with all arguments
# Will do  cat 'value1' 'value 2' 'etc'
core_variable_array_passToFunctionAsArguments myArray cat

# Iterate
index=0
some_callback()
{
	echo "array element at index $index in $core_variable_array_element"
	index=$((index+1))
}
core_variable_array_iterate myArray some_callback

# Iterate as if members were callbacks
core_variable_array_iterate myArray 'arg to pass to every callback'

```

### To use in code

This namespace is included by default. No additional actions are required.

### Functions

***
#### `core_variable_array_initialise()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|An variable name|_No_|

Initialises an array. `variableName` will then exist. If a local array is desired, then a little more work is needed to work-around shell interpreter bugs; one needs to define a second variable `${variableName}_initialised`. This is because under the covers two variables are used to manage arrays (primarily because some versions of bash can't differentiate between undefined and defined local variables)

```
local myArray
local myArray_initialised
core_variable_array_initialise myArray
```

***
#### `core_variable_array_append()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|An variable name|_No_|
|`…`|Zero or more elements to append|_Yes, but specify at least one for to be useful!_|

Appends elements `…` to an array. If the array doesn't yet exist, it is initialised if `…` has one or more elements. Caveats as above about local arrays.

```
core_variable_array_append myArray 'value1' 'value 2'
```

***
#### `core_variable_array_appendUniquely()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|An variable name|_No_|
|`…`|Zero or more elements to append|_Yes, but specify at least one for to be useful!_|

Appends elements `…` to an array if they haven't already been added. If the array doesn't yet exist, it is initialised if `…` has one or more elements. Caveats as above about local arrays.

```
core_variable_array_append myArray 'value1' 'value 2'
```

***
#### `core_variable_array_unset()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|An variable name|_No_|

Unsets an array.

***
#### `core_variable_array_isSet()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|An variable name|_No_|

Returns a `0` exit code if the array `variableName` is set or a non-zero one if it isn't.

***
#### `core_variable_array_isUnset()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|An variable name|_No_|

Returns a `0` exit code if the array `variableName` is unset or a non-zero one if it isn't.

***
#### `core_variable_array_isEmpty()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|An variable name|_No_|

Returns a `0` exit code if the array `variableName` is empty or a non-zero one if it isn't.

***
#### `core_variable_array_length()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|An variable name|_No_|

Writes the length of the array `variableName` to standard out.

***
#### `core_variable_array_at()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|An variable name|_No_|
|`index`|A zero-based index|_No_|

Set the variable `core_variable_array_index_element` to the element value at index `index` in the array `variableName`.

***
#### `core_variable_array_string()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|An variable name|_No_|
|`separator`|A string to separate the array elements|_No_|

Writes the concatenation of the elements of the array `variableName`, separated by `separator`, to standard out.

***
#### `core_variable_array_contains()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|An variable name|_No_|
|`containsValue`|An element value to find|_No_|

Returns a `0` exit code if the array `variableName` contains an element matching the value `containsValue` or a non-zero one if it isn't.

***
#### `core_variable_array_iterate()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|An variable name|_No_|
|`callback`|A function name|_No_|
|`…`|Zero or more elements to pass to `callback`|_Yes_|

Iterates over each element of the array, setting the variable `core_variable_array_element` before calling `callback` with `…`.

***
#### `core_variable_array_iterateShortcut()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|An variable name|_No_|
|`callback`|A function name|_No_|
|`…`|Zero or more elements to pass to `callback`|_Yes_|

Iterates over each element of the array, setting the variable `core_variable_array_element` before calling `callback` with `…`.
Expects the `callback` to return an exit code. If the exit code if `0`, then iteration is shortcircuited and the function returns `0`. If all callbacks are called and none return `0`, then the function returns an exit code of `1`.

***
#### `core_variable_array_iterateAsCallbacks()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|An variable name|_No_|
|`…`|Zero or more elements to pass to `callback`|_Yes_|

Iterates over each element of the array, treating it as the name of function to callback. Calls each callback with `…`.

***
#### `core_variable_array_iterateAsCallbacksShortcut()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|An variable name|_No_|
|`…`|Zero or more elements to pass to `callback`|_Yes_|

Iterates over each element of the array, treating it as the name of function to callback. Calls each callback with `…`. Expects callbacks to return an exit code. If the exit code if `0`, then iteration is shortcircuited and the function returns `0`. If all callbacks are called and none return `0`, then the function returns an exit code of `1`.

***
#### `core_variable_array_passToFunctionAsArguments()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`variableName`|An variable name|_No_|
|`function`|Either a function name or an executable on the path|_No_|

Passes all the elements of the array `variableName` to a function `function`. Useful for building up complex command lines, eg for `curl`:-


```bash
local optionsForCurl
local optionsForCurlInitialised
core_variable_array_initialise optionsForCurl

if [ "$(core_init_verbosity)" -gt 0 ]; then
	core_variable_array_append optionsForCurl --verbose
fi
core_variable_array_append optionsForCurl --url 'http://www.stormmq.com/'
if [ -n "$myUserCredentials" ]; then
	core_variable_array_append optionsForCurl --user "$myUserCredentials"
fi

# calls curl with, perhaps, curl --verbose --url 'http://www.stormmq.com/'
# Spaces in arguments are correctly handled
core_variable_array_passToFunctionAsArguments optionsForCurl curl
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