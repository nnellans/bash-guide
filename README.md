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

# Shebang Statement
All bash scripts should start with a shebang statement at the top.  This statement tells the kernel which interpreter to use when running the script.  Some common shebang statements for a bash script look like this:

```shell
#!/bin/bash

# This may not work in 100% of cases, as some systems place bash in a different location other than /bin
```

Portable (somewhat) version:

```shell
#!/usr/bin/env bash

# For portability reasons, it is commonly recommended to use a shebang like this
# This verison will run the first bash found in your $PATH variable
# This may not work for 100% of cases either, because env may not always be found under /usr/bin on all systems
```

# Commands

There are multiple types of commands you can run within bash:
1. Executables (like binaries and scripts)
2. Shell built-ins
3. Shell functions
4. Aliases

![](images/bash-commands.png)

> [!NOTE]  
> You can put more than 1 command on a single line by separating them with semicolons


