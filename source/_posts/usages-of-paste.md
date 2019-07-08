---
title: Usages of paste
date: 2019-03-12 00:04:20
tags: [linux, paste, shell]
thumbnail: https://www.computerhope.com/cdn/linux/paste.gif
---

[Command `Paste` Reference](http://www.theunixschool.com/2012/07/10-examples-of-paste-command-usage-in.html)

Paste is mainly used for processing text files. As all known, `stdin` can also be treated as a file.

## Paste Between Multiple Files

The command `paste file1 file2` will give a result where each line contains corresponding lines of every operand file delimited by a tab. And to designate delimiter of your own, simply use a `-d` option like this: `paste -d'/' file1 file2`. 

To do the trick of "paste" multiple files with similar names, apply wildcards like this: `paste file*`



To better illustrate its function, let's look at an example:

`a.txt` the file contains:

```
1
2
3
4
```

`b.txt` is as follows:

```
a
b
c
d
```

And the command `paste -d ',' a.txt b.txt` gives the result:

```
1,a
2,b
3,c
4,d
```



## Paste For One File

By default, `paste` operation on one file sees each line as an atom manipulative unit. That is, taking each line as a single file.

There is also `-d` option working just like what it is for in multiple-file circumstance. 

And there is special usage under this circumstance. To get it across clearer, let's look at another example:

A text file labeled `data.txt` contains this:

```txt
China
Chinese
Chile
Unknown
Brazil
DontknowEither
Ameriaca
English
```

For command `paste -d',' - -  < data.txt`, the output is:

```txt
China,Chinese
Chile,Unknown
Brazil,DontknowEither
America,English
```

And for `paste -d',' - - - < data.txt`, we get this:

```txt
China,Chinese,Chile
Unknown,Brazil,DontknowEither
America,English
```

Look at and learn from the differences between these two commands and outputs, which are easily understandable enough.
