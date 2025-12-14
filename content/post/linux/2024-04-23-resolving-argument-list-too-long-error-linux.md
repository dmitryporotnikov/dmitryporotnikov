---
id: 797
title: "Resolving the 'Argument list too long' Error When Deleting Files in Linux"
summary: "Resolving the 'Argument list too long' Error When Deleting Files in Linux"
date: '2024-04-23T08:46:11+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=797'
permalink: /2024/04/23/resolving-argument-list-too-long-error-linux/
categories:
    - Linux
---

## Resolving the 'Argument list too long' Error When Deleting Files in Linux

When working on Linux, you may occasionally encounter the error `/bin/rm: cannot execute [Argument list too long]` while attempting to delete a large number of files. This issue typically arises when the shell command exceeds the allowed limit for arguments.

Suppose you have a directory filled with files ending with the extension `.garbagefile`, and you decide to clean up by deleting them. You might use the following command:

```
rm -f *.garbagefile
```

However, if there are too many files matching the pattern, the command fails with:

```
/bin/rm: cannot execute [Argument list too long]
```

This error occurs because the shell expands `*.garbagefile` to all matching files, creating a very long command line that exceeds the system’s limit.

To overcome this limitation, you can use the `find` command combined with `xargs`. Here’s how it can be done:

```
find . -maxdepth 1 -name '*.garbagefile' | xargs rm
```

This method is efficient because `xargs` handles the splitting of the list into sufficiently small chunks to avoid the original error.