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
All bash scripts should start with a shebang statement at the top.  This statement tells the kernel which interpreter to use when running the script.  A common shebang statement for a bash script looks like this:

```shell
#!/bin/bash
```

This may not work in 100% of cases, as some systems place `bash` in a different location other than `/bin`.  Therefore, it is commonly recommended to use a shebang like this, instead:

```shell
#!/usr/bin/env bash
```

This second form will run the first `bash` found in your `$PATH` variable.  But, this may not work for 100% of cases either, because `env` may not always be found under `/usr/bin`

# 
