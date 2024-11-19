# Bash Scripting Guide

- Version: 0.0.1
- Author:
  - Nathan Nellans
  - Email: me@nathannellans.com
  - Web:
    - https://www.nathannellans.com
    - https://github.com/nnellans/bash-guide

> [!IMPORTANT]
> This is an advanced guide and assumes you already know the basics of Bash.  Think of this more like an advanced cheat sheet.  I went through various sources, captured any notes that I felt were important, and organized them into the README file you see here.

> [!WARNING]
> This is a live document.  Some of the sections are still a work in progress.  I will be continually updating it over time.

> [!TIP]
> Google's [Shell Style Guide](https://google.github.io/styleguide/shellguide.html) has many great recommendations and best practices that I also agree with.  When you see me use `GSG:` that means the following recommendation will be coming from Google's Style Guide.

---

# Table of Contents
- [Script File Basics](#script-file-basics)
- [Commands](#commands)
- [Variables](#variables)
  - [Shell Variables](#shell-variables)
  - [Environment Variables](#environment-variables)
- [Shell Functions](#shell-functions)
  - [Local Variables](#local-variables)
- [Aliases](#aliases)
- [Viewing Variables, Functions, and Aliases](#viewing-variables-functions-and-aliases)
- [Standard Input, Output, and Error](#standard-input-output-and-error)
- [Pattern Matching](#pattern-matching)
- [Shell Arithmetic](#shell-arithmetic)
- [Shell Expansions](#shell-expansions)
- [If Statements](#if-statements)
  - [test and [ ] commands](#conditional-1-the-test-and----commands)
  - [\[\[ \]\] commands](#conditional-2-the----commands)
  - [(( )) commands](#conditional-3-the----commands)
- [Case Statements](#case-statements)
- [Loops](#loops)
- [Array Variables](#array-variables)
- [Positional Parameters](#positional-parameters)

---
# Script File Basics

Shell scripts commonly use the `.sh` file extension.

### Comments

```shell
# comments start with the hashtag / pound sign symbol
echo "something" # comments can also go after a command, with at least 1 space before the #
```

### Shebang

- This should be on line 1 of all Bash scripts.  It tells the kernel which interpreter to use when running the script
- Standard example: `#!/bin/bash`
  - Note: not all systems place `bash` in `/bin`
- Portable version: `#!/usr/bin/env bash`
  - This version will run the first `bash` found in the `$PATH` variable
  - Note: not all systems place `env` in `/usr/bin`

### Shell options

The `set` command can be used to configure the shell.  Some common options are:
- `set -x` enables debug mode, where commands and arguments are printed out as they are executed
- `set -e` exits the script immediately if any command fails (returns a non-zero exit status)
- `set -u` exits the script immediately if you try to use a variable that's undefined
- `set -o pipefail`
  - When piping commands together Bash will only return the exit status of the last command
  - This `set` command tells Bash to return a non-zero exit status if ANY of the commands in the pipeline fail
- Bash lets you combine short parameters together, so you can use all options together: `set -xeuo pipefail`

---

# Commands

There are multiple types of commands you can run with bash:
1. Executables (like binaries and scripts)
2. Shell built-ins and keywords
3. Shell functions
4. Aliases

I created the graphic below to highlight how you can get information & help on the 4 types of commands:

![](images/bash-commands.png)

Various notes about running commands:

```shell
# put more than 1 command on a single line by separating them with semicolons
echo "this"; echo "that"; echo "up"; echo "down"

# extend a single command across multiple lines using backslashes
ls --no-group \
  --inode \
  --reverse \
  --recursive \
  --size

# send the output from command1 to the input of command2 using a pipe
command1 | command2

# extend long pipelines across multiple lines
command1 \
  | command2 \
  | command3

# logical command operators for specifying AND and OR logic
command1 && command2  # run command1 and, only if its successful, then run command2
command1 || command2  # run command1 and, only if it fails, then run command2

# in scripts, consider using the long form of parameters, when available, for better readability
du --all --human-readable  # use this
du -ah                     # instead of this

# finding the exit status of the last commmand
echo "$?"  # the special variable $? expands to the exit status of the last command
```

> [GSG](https://google.github.io/styleguide/shellguide.html): You should try to keep your lines to 80 characters or less

---

# Variables

### Shell Variables

Shell variables are only available to the particular shell session or script in which they are created.

Naming conventions:
- May contain letters, numbers, and underscores
- Must not start with a number
- Names are case-sensitive, so `varname` is different than `VarName`

> [GSG](https://google.github.io/styleguide/shellguide.html):
> - Shell variable names should be all lowercase, with underscores to separate words
> - Constant (read-only) variable names should be all uppercase, with underscores to separate words

Defining Shell variables:

```shell
# just assign a value to it
# must be no spaces between the end of the variable name, the equals sign, and the start of the value
variable_name="Some value"

# define multiple variables on the same line
var1="value1" var2="value2" var3="value3"

# another method is to use the declare command
declare var_name="value"
declare -r var_name="value" # the -r marks this variable as read-only (constant)
declare -i var_name=34527   # the -i marks this variable with the 'integer' attribute

# another way to set a variable as readonly
readonly var_name="value"
```

> [GSG](https://google.github.io/styleguide/shellguide.html): For sake of clarity use `readonly` instead of `declare -r`

Using Shell variables:

```shell
# you can use a variable by preceding its name with a $ symbol
echo "$var_name"

# curly braces are optional, but they are necessary if you have to do string concatenation
echo "${var_name}plusSomeMoreText"
```

> [GSG](https://google.github.io/styleguide/shellguide.html):
> - Always double-quote strings containing variables
> - Prefer `${var_name}` over `$var_name`, except:
>   - Don't use braces for special variables like `$@` or `$!`, unless strictly necessary
>   - Don't use braces for the first 10 positional parameters like `$0` and `$6`

Assigning temporary values to variables:

```shell
# assign values to variables in the same line that you run a command
# these assignments will take effect only for the duration of the specified command
var1="value1" var2="value2" command
```

### Environment Variables

These are very similar to Shell variables, but have the added benefit of being usable in any child shells or processes that are created.

Naming standards: same as Shell variables

Defining Environment variables:

```shell
# method 1: assign a value to a Shell variable, then 'export' it to an Environment variable
VAR_NAME="some value"
export VAR_NAME

# method 2: use export to create an Environment variable and assign a value at the same time
export VAR_NAME="some value"

# method 3: use the declare command to create an Environment variable
declare -x VAR_NAME="some value" # the -x tells declare to 'export' this variable
```

> [GSG](https://google.github.io/styleguide/shellguide.html): For sake of clarity use `export` instead of `declare -x`

Using Environment variables: same as Shell variables

# Shell Functions

> [!IMPORTANT]  
> In your code, you must define your functions first before you can call them

Naming standards: same as Shell variables

Defining Shell Functions:

```shell
# method 1
name_of_function() {
  command1
  command2
  return #optional
}

# method 2
# the () is optional with this method
function name_of_function() {
  command1
  command2
  return #optional
}
```

> [GSG](https://google.github.io/styleguide/shellguide.html):
> - The opening `{` should be on the same line as the function name
> - There should be no space between the function name and `()`
> - The `function` keyword enhances quick identification of functions

The `return` command is optional and is not required.
- By default, a Shell function will return the exit code from the last command it runs
- If you want your function to return a specific exit code, use `return` followed by a number

Using (calling) Shell functions:

```shell
# treat a shell function just like any command and call it by name
command1
command2
function_name
```

Use Positional Parameters to pass arguments to a Shell function:

```shell
# defining the function
function_name() {
  echo "$1"
  echo "$2"
  echo "$3"
}

# calling the function
function_name "arg1" "arg2" "arg3"
```

### Local Variables

You can define and use Local variables inside your functions.
- Local variables are visible only to the function where they are defined, and its children
- The names of Local variables will not conflict with other global variables defined elsewhere
  - This helps you to create portable functions
 
Defining and using Local variables in a function:

```shell
# method 1: using the local command. this is the preferred method
function_name() {
  local var_name="Albert"
  echo "You can call me ${var_name}"
}

# method 2: using the declare command
# when used inside of a function, declare will create local variables by default
function_name() {
  declare var_name="Albert"
  echo "You can call me ${var_name}"
}
```

> [GSG](https://google.github.io/styleguide/shellguide.html): Don't use `local` to create a variable and give it a value (using command substitution) in the same line. The  `local` command does not propagate the exit code from the command substitution.
> 
> ```shell
> # bad example
> local var_name="$(someCommand)" # this will return the exit code from local, not the command substitution
>
> # correct way
> local var_name
> var_name="$(someCommand)"
> ```

# Aliases

> [GSG](https://google.github.io/styleguide/shellguide.html): Aliases should be avoided in scripts. For almost every purpose, shell functions are preferred over aliases.

```shell
# define an alias
alias name_of_alias='commands;commands;commands'

# remove an alias
unalias name_of_alias

# use an alias
command1
command2
alias_name
```

# Viewing Variables, Functions, and Aliases

There are multiple commands to view the Shell variables, Environment variables, Functions, and Aliases that are defined in your environment. Below, you'll see a graphic I created that shows some of those commands, and what information each one will return.

![](images/bash-environment.png)

---

# Standard Input, Output, and Error

| Stream | Purpose | Default | File Descriptor |
| --- | --- | --- | --- |
| `stdin` | Feed data into a command | keyboard | `0` |
| `stdout` | Output data from a command | screen | `1` |
| `stderr` | Output errors from a comamnd | screen | `2` |

### Redirecting Standard Input

```shell
# method 1: run command1 first and "pipe" its output into command2's standard input
command1 | command2

# method 2: take the contents of file.txt and feed it into command1's standard input
command1 < file.txt

# method 3a: "Here Document": feed a whole body of text into command1's standard input
# the "token" should not be found anywhere else in the body of text
# it is common to use the following token:  _EOF_
command1 << token
line of text
another line of text
last line for now
token

# method 3b: using <<- instead of << lets you indent a Here Document for better readability
# only tab characters are supported (not spaces)
# be careful with how your text editor treats tabs vs. spaces
command1 <<- token
    indented line of text
    another line of text
    last line for now
token

# method 4: "Here String": feed a single line of text into command1's standard input
command1 <<< "line of text"
```

### Redirecting Standard Output

```shell
# write stdout to a file
command > file.txt   # overwrite
command >> file.txt  # append

# redirect stdout to stderr
command 1>&2
command >&2   # file descriptor 1 (stdout) is assumed, so the 1 can be omitted

# suppress stdout by redirecting to /dev/null
command > /dev/null
```

### Redirecting Standard Error

```shell
# write stderr to a file
command 2> file.txt   # overwrite
command 2>> file.txt  # append

# suppress stderr by redirecting to /dev/null
command 2> /dev/null
```

### Redirecting stdout & stderr to the same file

```shell
# method 1: the traditional & most compatible way
# this statement contains 2 redirections, and they must be in this order
# first, redirect stdout to a file. second, redirect stderr to stdout
command > file.txt 2>&1

# method 2: streamlined method for newer versions of Bash
command &> file.txt   # overwrite
command &>> file.txt  # append
```

---

# Pattern Matching

Used in a few ways: parameter expansion, filename expansion, the `[[ … ]]` compound commands, the `case` statement, etc.

The following is not an exhaustive list:

```shell
*            # matches any string, including null
?            # matches any single character

[abcABC123]  # matches any one of the characters from the enclosed set
[D-H3-8]     # matches any one of the characters from the enclosed ranges

[!abc]       # negation, matches any character NOT from the enclosed set
[^abc]       # also negation, matches any character NOT from the enclosed set

[-abc]       # match a literal - character by including it first or last
[abc-]       # match a literal - character by including it first or last

# matches any character belonging to the given class
# this is not an exhaustive list of classes
[[:alnum:]]
[[:alpha:]]
[[:ascii:]]
[[:blank:]]
[[:digit:]]
[[:word:]]

# negation, matches any character NOT belonging to the given class
[![:alpha:]]
```

---

# Shell Arithmetic

Used in a few ways: arithmetic expansion, the `(( … ))` compound commands, the `let` & `expr` commands, `for` loops, etc.

> [GSG](https://google.github.io/styleguide/shellguide.html): Always use `(( … ))` or `$(( … ))` rather than `let` or `expr` or `$[ … ]`

This is not an exhaustive list:

```shell
int1 + int2   # addition
int1 - int2   # subtraction
int1 * int2   # multiplication
int1 / int2   # division
int1 % int2   # remainder / modulo
int1 ** int2  # exponentiation / power of

# some special operators that also do assignment
++int1         # increment by 1. same as the assignment: int1 = int1 + 1
--int1         # decrement by 1. same as the assignment: int1 = int1 - 1
int1 += int2   # increment by N. same as the assignment: int1 = int1 + int2
int1 -= int2   # decrement by N. same as the assignment: int1 = int1 - int2
int1 *= int2   # same as the assignment: int1 = int1 * int2
int1 /= int2   # same as the assignment: int1 = int1 / int2
```

Comparison operators that are useful in `if` statements with `(( … ))`:

```shell
int1 == int2  # equal to
int1 != int2  # not equal to
int1 < int2   # less than
int1 <= int2  # less than or equal to
int1 > int2   # greater than
int1 >= int2  # greater than or equal to

! expression1               # expression1 does NOT succeed
expression1 && expression2  # both expression1 AND expression2 succeeds
expression1 || expression2  # either expression1 OR expression2 succeeds

# ternary conditional operator
# test expression1 first
# - if it succeeds, THEN test expression2
# - if it fails, ELSE test expression3
# will return the exit status from expression2 or 3 (whichever one gets tested)
expression1 ? expression2 : expression3
```

---

# Shell Expansions

Bash performs 7 different types of shell expansions:

### 1. Brace Expansion

Brace Expansion lets you generate multiple strings from a given pattern. The pattern can be:
- A comma-separated list of strings
- A range of integers or characters, using 2 periods (`..`) to separate the starting and ending integers/characters
  - Optionally, you can include an integer at the end which specifies the increment that will be used

Brace Expansion is NOT performed on anything inside double quotes

```shell
# comma-separated list
echo {a,b,c}     # returns: a b c

# range
echo {a..z}      # alpha range, returns: a b c d e f etc.
echo {a..z..5}   # with optional increment, returns every 5th letter: a f k p u z
echo {0..30}     # integer range, returns: 0 1 2 3 4 5 etc.
echo {0..30..7}  # with optional increment, returns every 7th number: 0 7 14 21 28
echo {001..100}  # you can add zeros and the expansion will zero-pad the result: 001 002 003 etc.

# preamble and postscript
echo da{me,re,ze}       # with optional preamble, returns: dame dare daze
echo {her,my,them}self  # with optional postscript, returns: herself myself themself
echo bi{ng,plan,pol}e   # with optional preamble & postscript, returns: binge biplane bipole

# nesting brace expansions is supported
echo {one{1,2},two{1,2}}  # returns: one1 one2 two1 two2
```

### 2. Tilde Expansion

Tilde Expansion is NOT performed on anything inside double quotes

```shell
# home directories
~      # expands to the value of the $HOME variable
~user  # expands to the given user's home directory

# pwd variables
~+  # expands to the value of the $PWD variable
~-  # expands to the value of the $OLDPWD variable

# dirs variables
~3   # same as the string returned from: dirs +3 (the + is assumed)
~+4  # same as the string returned from: dirs +4
~-5  # same as the string returned from: dirs -5
```

### 3. Parameter Expansion

AKA variable expansions, which we touched on in the Variables section. There is a LOT involved with Parameter expansion:

Standard Variable Expansion:
- For more info, see the full section on [Variables](#variables) from earlier

```shell
$var_name    # expands to the value of var_name
${var_name}  # expands to the value of var_name
```

Parameter Expansions dealing with non-existent or empty Variables:

```shell
${var_name:-"some other value"}
# if $var_name is set, then expand to its value
# if $var_name is unset or empty, then expand to "some other value"

${var_name:="some other value"}
# same as the :- operator above, with one additional change:
# if $var_name is unset or empty, then also assign "some other value" to $var_name

${var_name:+"some other value"}
# if $var_name is set, then don't use its value, return "some other value" instead
# if $var_name is unset or empty, then return nothing at all

${var_name:?"some other value"}
# if $var_name is set, then expand to its value
# if $var_name is unset or empty, then exit with an error, and send "some other value" to stderr
```

Parameter expansions dealing with length:

```shell
${#var_name}  # returns the length of $var_name's value
```

Parameter expansions dealing with string manipulations:

```shell
# substring
${var_name:5}      # start at offset 5, return the remainder of the string
${var_name:5:10}   # start at offset 5, return the next 10 characters
${var_name: -5}    # return the last 5 characters from the end of the string, must have a space
${var_name: -7:3}  # start at the 7th character from the end, return the next 3 characters, must have a space
${var_name:5:-2}   # start at offset 5, return the remainder of the string but stopping before the last 2 characters

# prefix / suffix removal with pattern matching
${var_name#pattern}   # remove leading portion of the value that's matched by pattern (shortest match)
${var_name##pattern}  # remove leading portion of the value that's matched by pattern (longest match)
${var_name%pattern}   # remove ending portion of the value that's matched by pattern (shortest match)
${var_name%%pattern}  # remove ending portion of the value that's matched by pattern (longest match)

# search and replace with pattern matching
# /replacementString is optional, if omitted the occurences will be deleted
${var_name/pattern/replacementString}   # replaces first occurence only
${var_name//pattern/replacementString}  # replaces all occurences
${var_name/#pattern/replacementString}  # match much occur at the beginning
${var_name/%pattern/replacementString}  # match much occur at the end

# converting case
# pattern is optional, it defines the exact characters that can be converted, like [A-F]
${var_name,,pattern}  # converts the entire value to lowercase
${var_name,pattern}   # converts the first letter of the value to lowercase
${var_name^^pattern}  # converts the entire value to uppercase
${var_name^pattern}   # converts the first letter of the value to uppercase
```

Returning names of Variables:

```shell
# expands to all variable names that begin with the given prefix
${!prefix*}  # if double-quoted, expands to one big word with all variable names
${!prefix@}  # if double-quoted, expands to a separate word for each variable name
```

### 4. Command Substitution

A command substitution will expand to the output from the given command.

```shell
$(command)  # standard form
`command`   # alternate, older form using backticks
```

> [GSG](https://google.github.io/styleguide/shellguide.html): Use `$(command)` instead of backticks.

```shell
# nesting command substitutions
$(command1 $(command2) param2)
`command1 \`command2\` param2`  # the older form requires escaping the inner backquotes
```

> [GSG](https://google.github.io/styleguide/shellguide.html): The `$(command)` format doesn’t change when nested and is easier to read.

```shell
# shortcut for cat command substitutions
$(cat file.txt) # instead of this
$(< file.txt)   # you can use this
```

### 5. Arithmetic Expansion

Expands to the result of an arithmetic expression
- Only supports integers
- Supports the operators from the [Shell Arithmetic](#shell-arithmetic) section

```shell
$(( 2 + 4 ))  # expands to 6
```

Nesting arithmetic expansion is supported, but it might be easier to just use parenthesis for grouping subexpressions instead:

```shell
$(( $(( 5 + 5 )) * 2 ))  # nesting arithmetic expansions
$(( (5 + 5) * 2 ))       # grouping subexpressions with parenthesis
```

### 6. Word Splitting

- By default, Word Splitting looks for spaces, tabs, and newline characters and treats them as delimiters between words
- Spaces, tabs, and newlines are the default value for the special Shell variable `$IFS` (which stands for Internal Field Separator)
  - If you want Word Splitting to operate differently, then update the value of `$IFS`
- Word Splitting only occurs on the results of other expansions, namely Parameter Expansion, Command Substitution, and Arithmetic Expansion
- Word Splitting is NOT performed on anything inside double quotes

### 7. Filename Expansion

- When Bash finds a `*`, `?`, or `[` character it considers the word to be a pattern that will be used for Filename Expansion
- Uses standard [Pattern Matching](#pattern-matching)
- Filename Expansion is NOT performed on anything inside double quotes

```shell
# list all files under /usr/bin/ that start with the letter m
ls /usr/bin/m*
```

### Effect of using Double Quotes

|  | Expanded inside Double Quotes |
| --- | --- |
| Brace Expansion | ❌ |
| Tilde Expansion | ❌ |
| Parameter Expansion | ✔️ |
| Command Substitution | ✔️ |
| Arithmetic Expansion | ✔️ |
| Word Splitting | ❌ |
| Filename Expansion | ❌ |

---

# If Statements

```shell
if conditional; then
  commands
elif conditional; then
  commands
else
  commands
fi
```

> [GSG](https://google.github.io/styleguide/shellguide.html):
> - Put `; then` at the end of the same line that contains `if` or `elif`
> - `fi` statements should be on their own line vertically aligned with the opening statement

- `conditional` can be any command, or set of commands, that will return an exit status
  - In Bash, if a command is successful it has an exit status of `0`, and if it fails it will have a non-zero exit status
- `elif` and `else` are optional, only use them if you need to
- There are multiple ways to write your conditional expression. Three of those ways will be explored below

### Conditional 1: the `test` and `[ … ]` commands

The `test` command comes in 2 different forms:

```shell
test expression  # less common form
[ expression ]   # widely used form
```

`test` supports a few logical operators for its expressions:

```shell
[ ! expression1 ]               # expression1 does NOT succeed
[ expression1 -a expression2 ]  # both expression1 AND expression2 succeeds
[ expression1 -o expression2 ]  # either expression1 OR expression2 succeeds
```

`test` has many different types of tests. I will highlight some of them below. This is not an exhaustive list.

```shell
# file-based expressions
[ -e file_name ]        # the given file exists
[ -f file_name ]        # the given file exists as a regular file
[ -d dir_name ]         # the given file exists as a directory

# string-based expressions
[ string ]              # the given string's length is greater than 0
[ -n string ]           # same as above. this is the preferred form
[ -z string ]           # the given string's length is 0
[ string1 = string2 ]   # string1 is equal to string2. don't use this form, as it can be confused with assignment
[ string1 == string2 ]  # same as above. this is the preferred form
[ string1 != string2 ]  # string1 is not equal to string2

# numeric comparisons
[ int1 -eq int2 ]       # int1 equal to int2
[ int1 -ne int2 ]       # int1 not equal to int2
[ int1 -gt int2 ]       # int1 greater than int2
[ int1 -ge int2 ]       # int1 greater than or equal to int2
[ int1 -lt int2 ]       # int1 less than int2
[ int1 -le int2 ]       # int1 less than or equal to int2
```

> [GSG](https://google.github.io/styleguide/shellguide.html): For numeric comparisons you should really use `(( … ))` instead

### Conditional 2: the `[[ … ]]` commands

- `[[ … ]]` is the enhanced replacement for `test`.
- `[[ … ]]` supports everything that `test` supports, with a few notable changes and additions:

Changes to logical operators:

```shell
# for AND, use && instead of -a
[[ command1 && command2 ]]

# for OR, use || instead of -o
[[ command1 || command2 ]]
```

Newly added expressions:

```shell
# test if a string matches a regular expression
[[ string =~ regexPattern ]]

# test if a string matches (or not) a pattern
[[ string == pattern ]]
[[ string != pattern ]]
```

> [GSG](https://google.github.io/styleguide/shellguide.html): `[[ … ]]` is preferred over `test` and `[ … ]`

### Conditional 3: the `(( … ))` commands

- Only supports arithmetic expressions
- Supports the operators from the [Shell Arithmetic](#shell-arithmetic) section

---

# Case Statements

By default, when `case` finds the first match it will run the given commands and then exit, with no subsequent matches being attempted.  This behavior can be modified by changing `;;` to a different option (but also see the `GSG:` note below)
- As a fallback, it is common to use `*` as the final pattern as this will always match
- You can define multiple patterns on the same clause using the OR logical operator: `|`
- Uses standard [Pattern Matching](#pattern-matching)

```shell
case expressionToMatch in
  pattern1)
    commands
    ;;
  pattern2 | pattern3)
    commands
    ;;
  *)
    commands
    ;;
esac

# using single-line clauses
case expressionToMatch in
  pattern1) command ;;
  pattern2) command ;;
  *) command ;;
esac
```

> [GSG](https://google.github.io/styleguide/shellguide.html):
> - Try to always use `;;` and avoid the alternative `;&` and `;;&` notations
> - For single-line clauses, put a space after the closing `)` of pattern, as well as a space before the ending `;;`

---

# Loops

> [GSG](https://google.github.io/styleguide/shellguide.html):
> - Put `; do` at the end of the same line that contains `while`, `until`, or `for`
> - `done` statements should be on their own line vertically aligned with the opening statement

### While Loops

`while` loops continue as long as the `testCommand` succeeds (has an exit status of `0`).<br />

```shell
while testCommand; do
  otherCommands
done
```

### Until Loops

`until` loops continue as long as the `testCommand` fails (has a non-zero exit status).<br />Put another way, the loop will continue UNTIL the `testCommand` finally succeeds.<br />

```shell
until testCommand; do
  otherCommands
done
```

### For Loops

`for` loops come in 2 different forms:

```shell
# first form
for symbolic_var in list; do
  commands
  echo "${symbolic_var}"
done
```

`in list` is optional. If omitted, then it will loop through the positional parameters (but see below)

> [GSG](https://google.github.io/styleguide/shellguide.html): Although it is possible to omit `in "$@"` in `for` loops we recommend consistently including it for clarity

```shell
# second form
for (( expression1; expression2; expression3 )); do
  commands
done
```

- expression1 initiates the loop counter, for example: `i = 0`
- expression2 defines the "while" loop condition, for example: `i < 5`
- expression3 runs after every loop to iterate the counter, for example: `++i`

---

# Array Variables

- Bash arrays are one-dimensional (think of a single-column spreadsheet)
- Arrays can be:
  - "indexed" which means they use a zero-based numerical index
  - "associative" which means they use a string-based index

Naming standards: same as Shell variables

Defining Indexed Arrays:

```shell
# 1. assign a value to a single index
array_name[2]="Some value"

# 2. assign multiple values in one line
array_name=("firstValue" "secondValue" "thirdValue")

# 3. assign multiple values and their indexes in one line
array_name=([0]="firstValue" [1]="secondValue")

# 4. using declare -a
declare -a array_name
declare -a array_name=("firstValue" "secondValue")

# 5. mark as readonly with declare -a -r
declare -a -r array_name=("firstValue" "secondValue")

# 6. mark as readonly with readonly -a
readonly -a array_name=("firstValue" "secondValue")
```

Defining Associative Arrays:

```shell
# 1. using declare with -A
declare -A array_name
declare -A array_name([firstIndex]="firstValue" [secondindex]="secondValue")

# 2. assign a value to a single index
array_name[firstIndex]="firstValue"  # must 'declare' the associative array first

# 3. mark as readonly with declare -A -r
declare -A -r array_name=([firstIndex]="firstValue" [secondIndex]="secondValue")

# 4. mark as readonly with readonly -A
readonly -A array_name=([firstIndex]="firstValue" [secondIndex]="secondValue")
```

Using Array values:

```shell
# preceed its name with a $ symbol, braces are required
echo "${array_name[3]}"      # indexed arrays: expands to the value at index 3
echo "${array_name[first]}"  # associate arrays: expands to the value at index "first"
```

Special Array syntax:

```shell
# expands to the total number of elements in the array
${#array_name[*]}
${#array_name[@]}

# expands to the length of the value for element `3`
${#array_name[3]}

# expands to all values in the array
${array_name[*]}  # if double-quoted, expands to one big word with all values
${array_name[@]}  # if double-quoted, expands to a separate word for each value

# expands to all indexes in the array
${!array_name[*]}  # if double-quoted, expands to one big word with all indexes
${!array_name[@]}  # if double-quoted, expands to a separate word for each value

# add values to the end of an array with +=
array_name+=("someValue" "anotherValue" "lastValue")
```

Substring Manipulation:

```shell
# indexed arrays: start at the 5th value, return the remaining values
${name[*]:5}       # if double-quoted, expands to one big word with all returned values
${name[@]:5}       # if double-quoted, expands to a separate word for each returned value

# indexed arrays: start at the 7th value from the end, return the next 3 values, must have a space
${name[*]: -7:3}   # if double-quoted, expands to one big word with all returned values
${name[*]: -7:3}   # if double-quoted, expands to a separate word for each returned value

# all arrays: for the value of $name[10] start at offset 5, return the remainder of the value
${name[10]:5}
```

Sorting an array:

```shell
# there is no built-in way to sort an array
# instead, use a for loop to get all array values, and pipe them to the sort command
unsorted_array=(q e h z a w r)
sorted_array=( $(for i in "${unsorted_array[@]}"; do echo "$i"; done | sort) )
echo "${sorted_array[@]}"
```

---

# Positional Parameters

- Used in a few circumstances:
  - Arguments passed to the shell when its invoked
  - Arguments passed to a script
  - Arguments passed to a shell function
- Can be reassigned using the `set` command
- Positional Parameter `0` always expands to the name of the shell or shell script, which means the first actual passed argument will be at Positional Parameter `1`

Using Positional Parameters:

```shell
$3
${5}   # for 0 through 9, braces are optional
${25}  # starting with 10 and higher, braces are required
```

Substring Manipulation

```shell
# start at parameter `5` and return the remainder of parameters
${*:5}  # if double-quoted, expands to one big word with all returned parameters
${@:5}  # if double-quoted, expands to a separate word for each returned parameter

# start at the 7th parameter from the end and return the next 3 parameters
${*: -7:3}  # if double-quoted, expands to one big word with all returned parameters 
${@: -7:3}  # if double-quoted, expands to a separate word for each returned parameter 
```

Special Positional Parameter syntax:

```shell
# expands to the total number of positional parameters
$#
${#*}
${#@}

# expands to the length of the value for the 5th positional parameter
${#5}

# expands to the values of all positional parameters
$*  # if double-quoted, expands to one big word with all positional parameters values
$@  # if double-quoted, expands to a separate word for each positional parameter value
```
