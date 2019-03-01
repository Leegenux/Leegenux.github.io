---
title: Usages of sort
date: 2019-03-01 20:21:12
tags: [linux, sort, shell, bash]
---

The command `sort`'s job is how it is literally understood.

Some useful options are listed below. For rarer usages, just go for [manual page](http://man7.org/linux/man-pages/man1/sort.1.html).

- `-t`: Specifies the delimiter separating each line into columns
- `-k`: Denotes by which column to sort.
  - `-n`: Enable numeric sorting in place of lexicographical sorting the default behavior.
- `-r`: Reverse, that is, do descending sorting.

Given a csv file `test.txt`:
```
China,5
Chinese,7
Chile,5
Unknown,7
Brazil,6
DontknowEither,14
America,7
English,7
```
The first column is the word, the second column is the number of letters the word consists of.

To sort by the first column descending and lexicographically, use command `sort -k 1 -t ',' -r test.txt` or `sort -r test.txt`. Then you'll get the result:
```
Unknown,7
English,7
DontknowEither,14
Chinese,7
China,5
Chile,5
Brazil,6
America,7
```

And to sort by the number of the letters consisting of the word, execute this command: `sort -k 2 -n -t ',' test.txt`. Then it prints the result:
```
Chile,5
China,5
Brazil,6
America,7
Chinese,7
English,7
Unknown,7
DontknowEither,14
```

Application of `sort` may save you a bunch of time in shell scripting especially when dealing with forms and tables. You have to master it if you script.