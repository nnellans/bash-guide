# Bash Guide

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

# Shebang
All bash scripts should start with a shebang line at the top.  This line tells the kernel which interpreter to use when running the script.

Standard example:
- This may not work in 100% of cases, as some systems place `bash` in a different location other than `/bin`
- ```shell
  #!/bin/bash
  ```
Portable (somewhat) version:
- For portability reasons, it is commonly recommended to use a shebang like this
- This version will run the first `bash` found in your `$PATH` variable
- This may not work for 100% of cases either, because `env` may not always be found under `/usr/bin` on all systems
- ```shell
  #!/usr/bin/env bash
  ```

# Commands

There are multiple types of commands you can run within bash:
1. Executables (like binaries and scripts)
2. Shell built-ins
3. Shell functions
4. Aliases

How to get information & help on commands:

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

Bash supports a couple of command operators for specifying AND and OR logic:

```shell
# run command1 and, only if its successful, then run command2
command1 && command2

# run command1 and, only if it fails, then run command2
command1 || command2
```

# Variables

Variables in bash come in a couple of different flavors

### Shell Variables

Shell variables are only available to the particular shell session or script in which they are created.

Naming conventions:
- May contain letters, numbers, and underscores
- Must not start with a number
- Names are case-sensitive, so `varname` is different than `VarName`
- It is common to use all lowercase letters for normal shell variables
- It is common to use all uppercase letters for "constants" (shell variables whose value should never change)

Defining Shell variables:

```shell
# you can defne a shell variable by simply assigning a value to it
# there must be no spaces between the end of the variable name, the equals sign, and the start of the value
variable_name="Some value"

# you can define multiple variables on the same line
var1="value1" var2="value2" var3="value3"

# another method is to use the declare command
declare varName="value"
declare -r varName="value" # the -r marks this variable as read-only (constant)
declare -i varName=34527   # the -i marks this variable with the 'integer' attribute
declare -a varName         # the -a declares an indexed array
declare -A varName         # the -A declares an associative array
```

Using Shell variables:

```shell
# best practice is to always enclose variables with double-quotes
# the exception is when you need to use word splitting

# you can use a variable by preceding it with a $ symbol
echo "$varname"

# optionally, you can add curly braces to the name to help with string concatenation
echo "${varname}_other_text"
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

Defining Shell variables:

```shell
# method 1: first assign a value to a Shell variable, and then 'export' it into an Environment variable
VAR_NAME="some value"
export VAR_NAME

# method 2: use the export command to create an Environment variable and assign a value at the same time
export VAR_NAME="some value"

# method 3: you can also use the declare command to create an Environment variable
declare -x VAR_NAME="some value" # the -x tells declare to 'export' this variable
```

Environment variables can be used in the same exact way that you use Shell variables.

### Local Variables

Local variables are used within Shell Functions.  Please see the section about Functions for more info.

# Shell Functions

Shell Functions allow you to 
