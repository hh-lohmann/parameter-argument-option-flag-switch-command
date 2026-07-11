###### Memo

# parameter vs. argument vs. option vs. flag vs. switch vs. command

Trying to get terms clear for talking about CLI parameters


*[hh lohmann &lt;hh.lohmann@gmail.com&gt;](mailto:hh.lohmann@gmail.com?subject=parameter-argument-option-flag-switch-command)*

<!-- see https://hh-lohmann.github.io/github-readme-pages-switch -->
<p align="center" id="github_readme_pages_switch" style="display:none;">
  <b><i>This page may be displayed more optimal in its
  <a href="https://hh-lohmann.github.io/parameter-argument-option-flag-switch-command">GitHub Pages view</a>
  </i></b>
</p>


## Synopsis

Not every CLI needs a thorough disctinction between "argument", "option", "parameter", "flag", "value", "switch", "command" and possibly "executable" and "script" or other labels as e.g. in [POSIX' utility conventions](#posix-utility-conventions), and even the [Bash manual](#bash-features-the-gnu-bash-reference-manual) - classifiable as the forefather of how things in a shell are organized - gives no definitions but rather uses terms interchangeable.

Precise as well as concise definitions can be found in the docs for [GNU's glib v2.2.3](#the-gnu-c-library-program-argument-syntax-conventions-glibc-223), [Python's optparse v3.14.6](#python-3146-documentation-optparse-terminology), [Sun's Java Tutorial](#sun-microsystems-the-java-tutorial-posix-conventions-for-command-line-arguments) and [GNU ddrescue v1.29](#gnu-ddrescue-v129-syntax-of-command-line-arguments) - the latter being the most current and most compact, so compact that it can be given here nearly in full (with markup added for code examples):

> * A command-line argument is an option if it begins with a hyphen (`-`).
> * Option names are single alphanumeric characters.
> * Certain options require an argument.
> * An option and its argument may or may not appear as separate tokens. (In other words, the whitespace separating them is optional). Thus, `-o foo` and `-ofoo` are equivalent.
> * One or more options without arguments, followed by at most one option that takes an argument, may follow a hyphen in a single token. Thus, `-abc` is equivalent to `-a -b -c`.
> * Options typically precede other non-option arguments.
> * The argument `--` terminates all options; any following arguments are treated as non-option arguments, even if they begin with a hyphen.
> * A token consisting of a single hyphen character is interpreted as an ordinary non-option argument. By convention, it is used to specify standard input, standard output, or a file named `-`.
>
> GNU adds _long options_ to these conventions:
> * A long option consists of two hyphens (`--`) followed by a name made of alphanumeric characters and hyphens. Option names are typically one to three words long, with hyphens to separate words. Abbreviations can be used for the long option names as long as the abbreviations are unique.

The following tries to harmonize this typical GNU precision with the typical real world wordings.


## Suggested terminology

### parameter
* Everything that is passed to an [executable](#executable), i.e. an [argument](#argument) / [option](#option) / [flag](#flag) / [switch](#switch) / [command](#command)
  * Note: The term "parameter" is chosen according to the established distinction for something that holds / represents a value (cf. [Bash Shell Parameters](#bash-shell-parameters) and [MDN Parameter](#mdn-parameter)) as opposed to "argument" for a stored / passed value itself (cf. [MDN Argument](#mdn-argument)), slightly extended here to the CLI typical value passing mechanisms "option" / "flag" / "switch" and "command"
  * Note that script interpreters traditionally store values passed via a CLI as an array unfortunately named `argv` ("argument vector") that moreover is constructed from the view of an operating system or a shell in that it also contains the [interpreter](#interpreter) and the [script](#script) as "arguments"
* **positional vs. named**
  * **positional**: The meaning / role of a parameter is determined by position
    * e.g. in `cp "adam.txt" "eve.txt"` the first parameter for `cp` is the source and the second the target so that changing the order as in `cp "eve.txt" "adam.txt"` would do something extremely different
  * **named**: The meaning / role of a parameter is determined by a name for it, typically implemented as an [option](#option) with a single argument
    * e.g. in `cp --source "adam.txt" --target "eve.txt"` the source and target for `cp` are given as a positional parameter for the respective option so that changing the order as in `cp --target "eve.txt" --source "adam.txt"` would do exactly the same

### executable
* The "program" vs. "app" called by the user
  * "utility" in [POSIX conventions](#posix-utility-conventions)
  * In Bash-like shells and for [npm's "bin"](#npm-docs-packagejson-bin) often a script with a [Shebang](#shebang)
  * For interpreted languages the [script](#script) that is passed to an [interpreter](#interpreter)

### script
* The [executable](#executable) for interpreted languages like JavaScript

### interpreter
* The actual shell command running the [script](#script)
* For JavaScript outside of a browser usually Node, Bun or Deno

### argument
* Formally the [value](#value) for a positional [parameter](#parameter) that is not a [command](#command)
* Effectively a [value](#value) passed to an [executable](#executable) or an [option](#option)
* Distinguished from [options](#option) by
  * being positional
  * being not prefixed with `--` or `-`
    * NB: Quoted arguments as in `--message "--sudo is not a valid option here"` of course may contain `--` / `-` to be quoted
* Distinguishable from [commands](#command) only by explicit positional definition
* **required** arguments are values without an execution is not possible
* **optional** arguments may be added to settle values that (may) play a role in execution
  * optional arguments are not to be confused with **[options](#option)**

### option
* Formally a named / non-positional [parameter](#parameter) to set or trigger a value
* Effectively a name-[value](#value) pair or a name with an intrinsic boolean value passed to an [executable](#executable)
  * An intrinsic boolean value is given when an option could only have one of two values (e.g. `true` / `false` or `yes` / `no` but not also `undefined`) so that omitting the option as a parameter can be interpreted as having one of the values and stating just the option name as the other
    * e.g. `-s` / `--silent` may express an intrinsic value "yes" for the option "silent" and therefore could be used instead of an extrinsic name-value assignment like `-s "yes"` / `--silent "yes"`, and so omitting `-s` / `--silent` would mean "no" as a shorter alternative to `-s "no"` / `--silent "no"`
  * Note that with appropriate parsing it could be posssible to have options for options (i.e. not only options for executables), but this may contradict the goal of an interface that could be used easily and intuitively on command lines ("chaining" separate CLIs via pipes or subshells would be an alternative to complex syntaxes)
* Distinguished from [arguments](#argument) / [commands](#command) by
  * being not-positional
  * being prefixed with `--` or `-`  (long options vs. short options, see below)
* name and value may be separated by a space ` ` or an equal sign `=`
    * i.e. `-x "y"` = `-x="y"`
      * Note that technically correct would be "by a sequence of one or more spaces ` ` that may contain not more than one equal sign `=` since tokens like `-x`, ` `, `=` and `"y"` generally may be surrounded by an arbitrary number of non-semantic spaces since these are reduced to a single arbitrary separator sign in interpretation
    * Note that this mainly reflects well established standards, not any deeper meaning
* **long options** are words consisting of letters and hyphens and prefixed by `--` (two hyphens)
  * e.g. `--encoding` or `--file-to-open`
  * Note that letters may be lower or upper case, but lower case should be preferred to avoid lower / upper confusions
* **short options** are single characters prefixed by `-` (one hyphen)
  * e.g. `-e`
  * multiple boolean short options (see below) may be combined into one parameter
    * e.g. `-e -x -z` = `-exz`
    * Note that this mainly reflects well established standards, not any deeper meaning
  * Note that letters may be lower or upper case, but lower case should be preferred to avoid lower / upper confusions

### flag
* A boolean [option](#option)
* **Use of "option" as term should be preferred**

### switch
* Especially in connection with positional [parameters](#parameter) sometimes used to refer to an [option](#option)
* **Use of "option" as term should be preferred**

### command
* A special first positional [parameter](#parameter) to specify a process with a dedicated set of parameters
  * e.g. for a `filehandler.js` different commands `read` and `delete` where reasonably only the former has an option `--encoding`
* Note that this pattern became popular with Git, but Git started as a collection of scripts where wrapper scripts started sub-scripts that were arguments ("commands") for their wrapper, i.e. this pattern may be generally an edge case
* May make semantic clearer
  * e.g. `filehandler.js read file.txt` vs. `filehandler.js --mode=read file.txt`
  * but also possible without a "command" by e.g. `filehandler.js --read file.txt` and explicit mutually exclusive options like e.g. `--read` and `--write`
* May introduce implementation complexity for checking arguments / options for a specific command vs. those for all commands / the executable itself
* Distinguishable from [arguments](#argument) only by explicit positional definition
* Distinguished from [options](#option) by never being prefixed with `--` or `-`

### value
* That what is represented by an [argument](#argument) or a an [option](#option)
* Usage as synonym for "argument" should be avoided


## Examples

### Executable without arguments / options

```sh
  ls
```

### Executable with argument

```sh
  ls *.json
```

### Executable with intrinsic option

```sh
  ls -l
```

### Executable with option with argument

```sh
  ls --sort "extension"
```

### Executable with command

```sh
  git log
```

### Executable with command and argument

```sh
  git log *.json
```

### Executable with command and intrinsic option

```sh
  git log --all
```

### Executable with command and option with argument

```sh
  git log --since 2026-07-11T09:00
```


## Details

Not covered here are
  * Conventional generalized `no-` prefix to set a boolean value to false
    * e.g. `--no-x` / `--no-xy` instead of `--x false` / `--x=false` / `--xy false` /`--xy=false`
    * Became popular with Git, but is syntactic sugar
  * Possible tokenization of a short option and its argument without space or qual sign
    * e.g. `-ofoo` instead of `-o foo` or `-o=foo`
    * Could be easily integrated in the [terminology](#suggested-terminology) here, but is obviously not as easy to handle for humans and algorithms to handle as a clear separation space or a equal sign
  * Number as option value with intrinsic option name
    * e.g. `-5` instead of `--length=5`
    * Regarded as a detail that could be implemented in different ways (e.g. direct parsing for a number as option name vs. first parse for known / unknown parameters and second parse over unknown parameters) 


## References

### Bash features: The GNU Bash Reference Manual
  * As single page: <https://www.gnu.org/software/bash/manual/bash.html>

### Bash Shell Parameters
  * <https://www.gnu.org/software/bash/manual/html_node/Shell-Parameters.html>

### Better Dev: Command line arguments anatomy explained with examples
  * <https://betterdev.blog/command-line-arguments-anatomy-explained/>

### GNU ddrescue v1.29: Syntax of command-line arguments
  * <https://lira.epac.to/DOCS/gddrescue/html/Argument-syntax.html>

### MangaD: Understanding Command-Line Arguments: Flags, Options, and Beyond
  * ChatGPT generated
  * <https://gist.github.com/MangaD/9ac227153849239d32232b4de36c7876>

### MDN: Argument
  * <https://developer.mozilla.org/en-US/docs/Glossary/Argument>

### MDN: Parameter
  * <https://developer.mozilla.org/en-US/docs/Glossary/Parameter>

### npm Docs: package.json: bin
  * <https://docs.npmjs.com/cli/v11/configuring-npm/package-json#bin>

### POSIX: Utility Conventions
  * The Open Group Base Specifications Issue 8: 12. Utility Conventions
    * POSIX.1-2024 is simultaneously IEEE Std 1003.1-2024 and The Open Group Standard Base Specifications, Issue 8
  * <https://pubs.opengroup.org/onlinepubs/9799919799/>

### Python 3.14.6 documentation: optparse: Terminology
  * <https://docs.python.org/3/library/optparse.html#terminology>

### Shebang
  * Wikipedia: <https://en.wikipedia.org/wiki/Shebang_(Unix)>

### Sun Microsystems: The Java Tutorial: POSIX Conventions for Command Line Arguments
  * <https://booksonline.nl/tutorial/essential/attributes/_posix.html>

### The GNU C Library: Program Argument Syntax Conventions (glibc-2.2.3)
  * https://ftp.gnu.org/old-gnu/Manuals/glibc-2.2.3/html_node/libc_511.html



<!-- see https://hh-lohmann.github.io/html-endspacer -->
<p id="endspacer" data-version="0.2.0" title="Endspacer - helps to align scrolling and positioning link targets | Scroll up to content or click vs. touch to jump to page top" align="center"><a href="#top"><img alt="Endspacer: './markdown-assets/endspacer.png' missing - see https://hh-lohmann.github.io/html-endspacer" src="./markdown-assets/endspacer.png" height="1000" width="100%"><br>[top]</a></p>
