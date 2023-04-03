---
layout: post
title: "Bash Scripting - Syntax Refresher"
date: 2022-03-08
categories: linux
excerpt_separator: <!--more-->
---

The more languages you learn, the easier it gets to learn another one. Most of the differences is only in the syntax of the language. Concepts like variables, loops, and functions largely remain the same. Here is a quick Bash Scripting syntax refresher. <!--more-->

### Shebang - #!

'''bash
#!/bin/bash
'''

The shebang's purpose is to specify the interpreter that will be used to execute the script. The path that follows, `/bin/bash` is the path to the interpreter. This line must be the first line of the file.

### Variables

```bash
myvar=value
```

Bash variable assignment must not have whitespaces on either side of the equals sign. <br>
A few special varibles:
- `$0` - name of bash script
- `$1 - $9` - the first 9 arguments to the bash script
- `$?` - the return status of previous command / function
- `$HOSTNAME` - hostname of the machine

### Substitutions

```bash
echo "... $variable"
myvar=$( ls )
```

Variable substitution only works with `""` double quotes. Single quotes take each character literally.<br>
Command Substitution works with `()` brackets, any newlines in the command output is removed.

### Inputs

```bash
read -sp myvar
```

`read` command reads input into the variable. Options like `-p` (prompt) and `-s` (silent) is available.

### Conditionals

```bash
if [ -s ./nonempty.txt ]
then
    echo "nonempty"
else
    echo "empty or no such file"
fi
```

If statments are enclosed by `if` and `fi`, with `then` and `else` leading to commands to execute depending on the result of the expression stated within `[]` square brackets.
The expression inside `[]` is determined using command `test`. use `help test` for more information.

### Loops

```bash
while [ <some test> ]
do
    <commands>
done

until [ <some test> ]
do
    <commands>
done

for var in <list>
do
    <commands>
done
```

The list used in for loops is defined by a series of strings separated by spaces.
The `seq` command with command substitution could help here.

### Functions

```bash
func_name () {
    <commands>
}

function func_name {
    <commands>
    return sth
}

func_name
```

Both ways of defining a function is valid.
Bash functions do not accept any parameters or arguments in the `()` brackets. They are just for decoration.
Function definition must appear before any calls to the function.
Call the function with its function name, with no brackets after it.

### Further Information
Here are some resources that may help you to learn more:<br>
[Ryan Tutorials - Bash Scripting](https://ryanstutorials.net/bash-scripting-tutorial/)<br>
[Hackersploit Bash Scripting](https://hackersploit.org/bash-scripting/)
