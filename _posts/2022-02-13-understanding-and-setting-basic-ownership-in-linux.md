---
layout: post
title: "Understanding and Setting Basic Ownership in Linux"
date: 2022-02-13
categories: linux
excerpt_separator: <!--more-->
---

### Ownership

In Linux, there can exist more than one user that has access to one machine. To obtain security within this multi-user OS, users (and groups) can have *ownership* over files, directories and processes that determine the permission granted to any other user.
<!--more-->
*Root* can change ownership of all files and directories.

### Using CLI for ownership

Although it is possible to change ownership and permission using the graphical user interface, it is better (reduces accidental operations / mistakes) to use the command line interface. The command to use is `chown` (*Ch*ange *Own*er). 

```shell
chown [OPTION]... [OWNER][:[GROUP]] FILE...
```

Values enclosed in [] square brackets are optional. If given, the ownership of the file will be given to that of [OWNER], and the group ownership to [GROUP]. Options like -R changes ownership of all files within the directory if a directory path is given instead of a file path.

A command to just change the group ownership also exists, consult the man page for `chgrp`.

### Permissions

A file has permissions for its owner, its group and world (others). You can use the `ls -l` command to view a file's permissions. A typical permissions string looks like:

> -rwxr-xr-x

The first character shows the file type. Common file types are:
- '-' for normal data files
- 'd' for directories
- 'l' for symbolic links

The next three characters are the owner's permissions:
- 'r' the owner can read,
- 'w' write,
- 'x' and execute the file.

The next three characters are the group's permissions:
- 'r' the group can read,
- 'w' cannot write,
- 'x' can execute the file.

The last three characters are the world permissions (others, who are both not the owner and not a member of the group).

 If a directory grants write permission, the user can create, delete or rename files in the directory even if the user does not have permission to write to those files. If a directory grants execute permission, it grants the user with the permission to enter the directory, i.e. to access files within the directory.

 The superuser *root* can read or write any file regardless of the permission set, but requires the execute permission set in order to run a file.

### Permissions in Octal-mode 

Permissions can also be represented in a code format (Octal-mode). Imagine a 3-bit binary value with read, write and execute permissions corresponding to each bit in order. If a permission is set, the bit would be 1, and otherwise the bit would be 0. Now just convert the binary into denary and that is our permission code. For example, the previous permission would translate to 755. '111' is 7 in denary and '101' is 5.

### Using CLI for permission

To change permissions using the command line interface, use the command `chmod`.

```shell
chmod [OPTION]... OCTAL-MODE FILE...
```

The chmod also performs in symbolic-mode, which can be more flexible than the octal-mode. However, symbolic mode is beyond the scope of this post.


[Back](/)
