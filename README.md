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
- [Standard Input, Output, and Error](#standard-input-output-and-error)

---
# Script File Basics

Bash scripts are just text files. While they don't require a file extension, it's common for them to use the `.sh` file extension.

### Comments
Comments start with the `#` symbol

```shell
# this is a comment

someCommand # comments can even go at the end of a command, as long as you have at least one space before the #
```

### Shebang
All bash scripts should start with a shebang line at the top.  It tells the kernel which interpreter to use when running the script.

Standard example:
- ```shell
  #!/bin/bash
  ```
- This may not work in 100% of cases, as some systems place `bash` in a different location other than `/bin`

Portable (somewhat) version:
- ```shell
  #!/usr/bin/env bash
  ```
- For portability reasons, it is commonly recommended to use a shebang like this
- This version will run the first `bash` found in your `$PATH` variable
- This also may not work for 100% of cases, as some systems place `env` in a different location other than `/usr/bin`

### Setting shell options

The `set` command can be used to configure the shell.  Some common options are:
- `set -x` enables debug mode, where commands and arguments are printed out as they are executed
- `set -e` exits the script immediately if any command returns a non-zero exit status
- `set -u` exits the script immediately if you try to use a variable that's undefined
- `set -o pipefail`
  - When piping commands together (`command1 | command2`) Bash will only return the exit status of the last command
  - This `set` option tells Bash to return a non-zero exit status if ANY of the commands in the pipeline fail
- Bash lets you combine short parameters together, so you can set all options at once with : `set -xeuo pipefail`

---

# Commands

There are multiple types of commands you can run within bash:
1. Executables (like binaries and scripts)
2. Shell built-ins and keywords
3. Shell functions
4. Aliases

I created the graphic below to highlight how you can get information & help on the 4 types of commands:

![](images/bash-commands.png)

You can put more than 1 command on a single line by separating them with semicolons:

```shell
echo "this"; echo "that"; echo "up"; echo "down"
```

You can extend a single command over multiple lines for better readability using backslashes:

```shell
ls --no-group \
  --inode \
  --reverse \
  --recursive \
  --size
```

You can send the output from command1 to the input of command2 using a pipe

```shell
command1 | command2
```

Bash supports a couple of command operators for specifying AND and OR logic:

```shell
# run command1 and, only if its successful, then run command2
command1 && command2

# run command1 and, only if it fails, then run command2
command1 || command2
```

For better readability in scripts, consider using the long form of parameters, when available:

```shell
# instead of this
du -ah

# use this
du --all --human-readable
```

---

# Variables

Variables in bash come in a couple of different flavors

### Shell Variables

Shell variables are only available to the particular shell session or script in which they are created.

Naming conventions:
- May contain letters, numbers, and underscores
- Must not start with a number
- Names are case-sensitive, so `varname` is different than `VarName`
- It is common to use all lowercase letters for normal shell variables
- It is common to use all uppercase letters for "constants" (or, shell variables where the value never changes)

Defining Shell variables:

```shell
# you can define a shell variable by simply assigning a value to it
# there must be no spaces between the end of the variable name, the equals sign, and the start of the value
variable_name="Some value"

# you can define multiple variables on the same line
var1="value1" var2="value2" var3="value3"

# another method is to use the declare command
declare var_name="value"
declare -r var_name="value" # the -r marks this variable as read-only (constant)
declare -i var_name=34527   # the -i marks this variable with the 'integer' attribute
declare -a var_name         # the -a declares an indexed array
declare -A var_name         # the -A declares an associative array
```

Using Shell variables:

> [!NOTE]  
> It is advisable to always enclose variables with double-quotes, with the exception being when you need to use word splitting.

```shell
# you can use a variable by preceding it with a $ symbol
echo "$var_name"

# optionally, you can add curly braces to the name
# this is necessary if you have to do string concatenation
echo "${var_name}plusSomeMoreText"
```

Assigning temporary values to variables:

```shell
# you can assign values to variables in the same line that you run a command
# these assignments take effect only for the duration of the specified command
var1="value1" var2="value2" command
```

### Environment Variables

Just like Shell variables, Environment variables can be used in the current shell session.  However, Environment variables also have the benefit of being usable in any child shells or processes that are created.

Naming standards are the same as Shell variables, however it is common convention to use all uppercase letters for Environment variables.

Defining Environment variables:

```shell
# method 1: first assign a value to a Shell variable, and then 'export' it into an Environment variable
VAR_NAME="some value"
export VAR_NAME

# method 2: use the export command to create an Environment variable and assign a value at the same time
export VAR_NAME="some value"

# method 3: you can also use the declare command to create an Environment variable
declare -x VAR_NAME="some value" # the -x tells declare to 'export' this variable
```

Using Environment variables:

Environment variables can be used in the same exact way Shell variables are used.

# Shell Functions

Shell Functions allow you to define reusable blocks of code.

Defining Shell Functions:

> [!IMPORTANT]  
> In your code, you must define your functions first before you can call them

```shell
# method 1: this is the most compatible method
name_of_function() {
  command1
  command2
  return #optional
}

# method 2
function name_of_function {
  command1
  command2
  return #optional
}
```

The `return` command is optional and is not required.
- By default, a Shell function will return the exit code from the last command it runs
- However, if you would like your function to return a specific exit code, then you can use `return` followed by a number.  For example, `return 1` would make your function return an exit code of 1

Using (calling) Shell functions:

```shell
# treat a shell function just like any command and call it by name
command1
command2
function_name
```

Use Positional Parameters to pass arguments to a Shell function:<br />*(Positional Parameters will be discussed in more detail later)*

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
# method 1: using the local command
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

# Aliases

When you think of Aliases you should think of "nicknames".  You can create an Alias to represent a complicated command, or set of commands.  For example, if you were a Kubernetes admin you might run the following command often:  `kubectl get pods --namespace default --out wide`.  You could create an alias for this called `kgp`. After that, you can simply run `kgp` instead of that long command.

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

If you want to view all of the Shell variables, Environment variables, Functions, and Aliases that are currently defined in your environment, then there are several commands you can run to find that info. Below, you'll see a graphic I created that shows which commands to run, and what information each one will return.

![](images/bash-environment.png)

---

# Standard Input, Output, and Error

Standard Input (`stdin`), Standard Output (`stdout`), and Standard Error (`stderr`) are streams of information that a command can use.

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

# method 3a: feed a whole body of text into command1's standard input
# this is known as a "Here Document"
# the starting and ending "token" should not be found anywhere else in the given body of text
# it is common to use the following token:  _EOF_
command1 << token
line of text
another line of text
last line for now
token

# method 3b: you can indent a Here Document for better readability
# use <<- instead of <<
# the body of text can be indented with tab characters only (not spaces)
# be careful with how your text editor treats tabs vs. spaces
command1 <<- token
    indented line of text
    another line of text
    last line for now
token

# method 4: feed a single line of text into command1's standard input
# this is known as a "Here String"
command1 <<< "line of text"
```

### Redirecting Standard Output

```shell
# write stdout to a file
command > file.txt  # overwrite
command >> file.txt # append

# suppress stdout by redirecting to /dev/null
command > /dev/null
```

### Redirecting Standard Error

Remember, the shell references `stderr` by its file descriptor of `2`

```shell
# write stderr to a file
command 2> file.txt  # overwrite
command 2>> file.txt # append

# suppress stderr by redirecting to /dev/null
command 2> /dev/null
```

### Redirecting stdout & stderr to the same file

Remember, the shell references `stdout` and `stderr` by file descriptors `1` and `2`, respectively

```shell
# method 1: the traditional & most compatible way
# this statement contains 2 redirections, and they must be in this order
# first, redirect stdout to a file
# second, redirect stderr to stdout
command > file.txt 2>&1

# method 2: streamlined method that works in newer versions of Bash
command &> file.txt  # overwrite
command &>> file.txt # append
```

---

# If Statements

