---
title: Usages of uniq
date: 2019-03-03 23:56:27
tags: [linux, shell, bash, uniq]
---

`uniq` is just an abbreviation of word unique. It's capable of performing actions about finding unique or duplicated elements. It usually works with `sort` command because it  **does not detect repeated lines unless they are adjacent.**



##  Important Options

Some important options are listed below:

- `-i`: Do case-insensitive comparisons
- `-c`: Count the occurrences of lines
- `-u` and `-d` : With `-u` you get only unique lines and `-d` duplicated ones.



## Other Possible Options

1. To limit comparison between lines to only the first N characters, use `-w`
2. Avoid comparing the first N characters with `-s` option
3. Avoid comparing first N fields with option `-f`
4. More on [Google](www.google.com)



## Examples

With a simple example you can figure out how it works, say I have a **sorted** text file `colors.txt`:

```
Green  is the favorite of David.
Green  is the favorite of Thomas.
Purple is the favorite of Daniel.
Red    is the favorate of James.
Red    is the favorite of Christopher.
Yello  is the favorite of Joseph.
```

To count the occurrences of colors, you can try this: `uniq -c -w 5 colors.txt`. Here comes the console printout:

```
   2 Green  is the favorite of David.
   1 Purple is the favorite of Daniel.
   2 Red    is the favorate of James.
   1 Yello  is the favorite of Joseph.
```

The digit before each line denotes the times of occurrences.

When the amount of the lines is great, `uniq` command can refine some important statistic features. 

> Note that the part of line avoided is not directly removed, but you can still use other tools to achieve this goal.



And to get unique lines or duplicated ones, use `-u` or `-d` respectively. Here are the results:

```
$ uniq -d -w5 colors.txt
Green  is the favorite of David.
Red    is the favorate of James.

$ uniq -u -w5 colors.txt
Purple is the favorite of Daniel.
Yello  is the favorite of Joseph.
```



In the example above, `-w` cannot be cut out when focusing on colors. But in daily uses, `uniq` often targets better preprocessed lines, where `-w` is not so necessary.



Can you get the number of colors appearing in `more_colors.txt` the text file below? Try `uniq` yourself.

```
more_colors.txt 
---------------
Green  is the favorite of David.
Green  is the favorite of Thomas.
Purple is the favorite of Daniel.
Red    is the favorate of James.
Red    is the favorite of Christopher.
Yello  is the favorite of Joseph.
yello  is the favorite of Paul.
pink   is my favorite.
Blue   is Harry's favorite.
Black  is the favorite of nobody.
Grey   is the favorite of Kyle.
yello  is the favorite of Stan.
Orange is the favorite of Kenny.
Red    is the favorite of Eric.
Pink   is the favorite of Butters.
```

