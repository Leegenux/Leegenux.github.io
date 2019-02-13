---
title: Usages of cut
date: 2019-02-12 21:59:56
tags: [linux, shell, bash, cut]
---

`cut` command is really useful when it comes to table data handling.

For example, I have a comma-separated values file:

```
---- test.txt
John,Smith,34,London
Arthur,Evans,21,Newport
George,Jones,32,Truro
```

And for generating a special table containing only names and ages, I can execute this: `cut -d',' -f 1,3 < test.txt`. Expectedly here comes the result:

```
John,34
Arthur,21
George,32
```



In most cases, you need to specify 3 options to properly manipulate the efficient `cut` command.

#### Specifying Delimiter

We can use `-d` option to specify delimiter other than tab the default one. And the delimiter may be specified only when operating on fields(`-f` option).

#### Specifying LIST

`cut` can operate on fields, characters or bytes respectively with `-f`, `-c` and `-b` .

#### Specifying Range

Following the **LIST** option is the range, where you can specify a point (`-c 3`) or a consecutive range (`-c 1-10`) and you can combine those together with commas (`-c 3,7-9,11`).

> Above, `-c` might be replaced with `-f` and `-b` for corresponding purpose.

By the way, bare it in mind that it is one-based numbering system the `cut` command is working with other than zero-based one.

There is a option `--complement`, which indicates complement of currently specified range as range.



For more examples, please refer to web page --- [Reference](https://www.computerhope.com/unix/ucut.htm)