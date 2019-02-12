---
title: Usages of awk
date: 2019-02-11 19:55:30
tags: [awk, linux, shell, bash]
---

#### Basic Understanding

The common template of using `awk` is like this: `awk ' { actions } ' textFile`.

Say I have a text file containing lines:

```
------ test.txt
hello world
heLLoWorld ha
London Paris
Changsha Wuhan
Beijing NewYork
Hell Nowhere
```

This command `awk 'BEGIN{IGNORECASE=1} /^hel/ { print $2 }' test.txt` emits result:

```
world
ha
Nowhere
```

So basically, `awk` just processes every line in the text file respectively and gives out the combined result.

#### How to Do Regular Expressions

According to the example above as well as the following one, you can easily get the point of doing regex with `awk`.

Let's say we have a text file called `test.txt` whose content is as follows:

```txt
test 123
TEd show
Howday Tom
Hahahha
testing, take it easy
```

To grab every line where there is word "test", try this: `awk " /test/ { print } " test.txt`. The part specifying the regular expression is `/test/`

#### How to do Conditionals

For example, `awk ' { if ( $1 ~ /test/ ) print } ' test.txt` will find out every line whose first column is word "test" (case-sensitive).

#### How to Specify Column Seperator

with `-F` option, you can denote the separator in place of white space, which is the default. Look at the example : `awk -F, ' { print $1 } ' test.txt` will print out the first column of each line separated by commas.



Please refer to GOOGLE SEARCH for more in-depth insights.
