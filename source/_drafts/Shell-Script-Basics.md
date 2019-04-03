---
title: Shell Script Basics
date: 2019-02-03 19:27:28
tags: shell bash linux
---

## Ways of Arithmatic Calculation in Shell Scripts

- `$(())` gives the result of the calculation
- Also you can do calculation using `` `expr CALCULATION` ``
- To do calculation and assign the result directly, you can try this: `((result = $a + $b))`

## Ways For Function to Return Value

* `eval` --- eval do evaluation twice. First of the content at the right of equal sign, second the assigment (To be accurate, the second time is called assigment, not evaluation).
* using echo inside of function and with the help of command substitution you get the result
* global variable

## Usage of `AWK`

#### Basic Understanding 

The common template of using `awk` is like this: `awk " { actions } " textFile`.

So basically, `awk` just processes every line in the text file respectively and gives out the combined result.

#### How to do Regular Expressions

Let's say we have a text file called "test.txt" whose content is as follows:
```txt
test 123
TEd show
Howday Tom
Hahahha
testing, take it easy
```

To grab every line where there is word "test", try this: `awk " /test/ { print } " test.txt`. The part specifying the regular expression is `/test/`

#### How to do Conditionals

For example, `awk " { if ( $1 ~ /test/ ) print } " test.txt` will find out every line whose first column is word "test" (case-sensitive).

#### How to Specify Column Seperator

with `-F` option, you can denote the seperator in place of white space, which is the default. Look at the example : `awk -F, " { print $1 } " test.txt` will print out the first column of each line seperated by commas.

## `Cut` command

#### Specifying Delimiter

We can use `-d` option to specify delimiter other than tab the default one.

#### Specifying LSIT

[Reference](https://www.computerhope.com/unix/ucut.htm)

We can denote field, character or byte with `-f`, `-c` and `-b` respectively.

#### Specify Range

Following the LIST options are range field, where you can specify a single one (`-c 3`) or a continous range (`-c 1-10`) and you can combine those together with commas (`-c 3,7-9,11`)

By the way, bare it in mind that it is 1 based index system other than 0.

## `tail` and `head`

#### Specifying Units

You wanna lines? Use `-n`, and there are `-c` for characters, `-b` for bytes.

## Usage of `tr`

The `tr` is a abbreviation of "translate" or "transliterate"

In general, the `tr` command is execute in the format of this: `tr [OPTION] SET1 [SET2]`

- `-s` : Replace sequence of repeated character in the character set with single occurence. It can't be used like this : `tr -s "[a-z]"`.
- `-c` : It indicates the complement of the first set. (补集)
- `-d` : Delete all characters listed in the first set
- `-t` : SET2 is extended to length of SET1 by repeating its last character as necessary. Excess characters of SET2 are ignored.

## Command `sort`

Some useful options are listed below. For more rare usages' reference, just go for manual page.

* `-t`: Specifys the delimiter which seperates the columns in your text file.
* `-k`: Denotes by which column to sort.
* `-n`: Sorting lexicographically is the default behavior, and to enable numeric sorting you must pass a `-n` option.
* `-r`: Reverse, that is descending sorting.

## The *heredoc* and *here string* syntax

[Reference Page](https://stackoverflow.com/questions/2500436/how-does-cat-eof-work-in-bash)

#### *heredoc* syntax

The **heredoc** syntax is very useful when coping with multi-line string manipulation in shell scripts. It works in the format of `cat << [delimiter]`.

The lines after the cat line and before the delimiter line, known as here document, are all passed to the stdout. Note that the delimiter line should be only one single word without any spaces around, unless you're using `<<-` heredoc syntax, which ignores every space and tab before any non-space character in every line, enabling indenting here document in shell scripts.

#### *here string* syntax

Here string works like piped instream.

For example:
```bash
$ tr 'a-z' 'A-Z' <<< 'a
hello
world
'
```
Gives 
```bash
A
HELLO
WORLD
```
as output.

## How to escape with Backslash in Shell Script

Words of the form `$'string'` are treated specially. For example, `$'\a'` is decoded as alert(bell)

## Command `Uniq`

Some important options are listed below:

- `-i`: Do case insensitive comparisons
- `-c`: Count the occurences of lines
- `-u` and `-d` : With `-u` you get only unique lines and `-d` duplicated lines.

#### Other possible options

1. To limit comparison only to the first N characters, use `-w`
2. Avoid comparing the first N characters with `-s` option
3. Avoid comparing first N fields with option `-f`
4. More on google


## Command `Paste`

[Command `Paste` Reference](http://www.theunixschool.com/2012/07/10-examples-of-paste-command-usage-in.html)

#### Paste Between Multiple Files

Paste is mainly used for processing text files. As all known, stdin can also be treated as file.

The command `paste file1 file2` will create a result text where each line contains corresponding lines of every operand file delimited by a tab. And to designate delimiter of your own, simply use a `-d` option like this: `paste -d'/' file1 file2`. 

To do the trick of "paste" multiple files with similar name, write command like this: `paste file*`

#### Paste For One File 

By default, `paste` opertion on one file take each line as manipulative units. That is, taking each line as a single file.

There are also `-d` option working just like it is in multiple-file circumstance. 

And there are some other special usages that really help. To get it across easier, let's look at a example:

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

Look at the difference between these two commands, which counts.

## How to Check If String Contains Part

[StackOverflow Reference](https://stackoverflow.com/questions/229551/how-to-check-if-a-string-contains-a-substring-in-bash)

By exploiting wildcard technique, you can check if string contain substring with command like the example below:
```bash
string="Hello"
if [[ $string == *"el"* ]] ; then
    echo "The string contains 'el'"
fi
```

## How to Append Component to Array

Use syntax: `array+=(component)`

## How to Slice an String

- `${string:m:n}`: From mth to nth
- `${string:n}`: Starting from nth

## Problem:Lonely Integer

[Problem Description](https://www.hackerrank.com/challenges/lonely-integer-2/problem?h_r=next-challenge&h_v=zen&h_r=next-challenge&h_v=zen&h_r=next-challenge&h_v=zen)

A super simple solution:
```bash
read # Get the first line
tr ' ' '\n' | sort | uniq u
```

## `grep` Tutorial

#### Basics
[GREP reference](https://www.gnu.org/software/sed/manual/html_node/Regular-Expressions.html "Click to jump")

1. line and word anchors
2. character classes
3. wildcards

#### `\>` `\<` and `\b`

Above are 3 typical word anchors, differences are the locations that they denote.

`\>` denotes end of word and `\<` start of word, however `\b` indicates either.

#### Back Reference

Without back reference, wildcards, character classes and other subtitutions followed by number restrictions might work in a way unexpected.

For example, `[0-9]\{2\}` might be interpreted as `[0-9][0-9]`, which can match all 2 consecutive occurrences of digits such as 12, 92, 32. But what if I just want to match two same consecutive occurrences? You can easily make things work using back reference : `\([0-9]\)\1`. 

There are a few points you should attach significance to to better understand it:
- The referenced part is quoted with escaped brackets `\(`, `\)`
- The main body of reference syntax is consistent of slash `\` with a following digit, the digit is a index to the parenthesized parts, which are implicity numbered by counting occurrences of `\(` left-to-right.


## `sed` tutorial

For example, substitute the first occurrence of word "the" with "that" using this command: `sed -e "s/\<the\>/that/1"`.

To be frank, sed is in a bit part about regular expression. And the rules apply to grep also work with sed.

#### About the Last Digit 

The last digit specifies which occurrence of match to replace. For example:
`sed -E "s/[0-9]+/num/2"` replaces the second occurrence of digit with string "num".

#### Reference Of the Match

To do reference, the same syntax -- back reference still applies to `sed`, and it can even be used in the substitution part.

For example: `sed -E "s/(.+) (.+)/\2 \1/g"` will reverse the ordering of segments seperated by single space.


#### Do Case-insensitive Match

The default behaviour of sed is case-sensitive. To activate case-insensitive manner in terms of sed substitution. You just need to add a letter "I" in the last part of script just like this: `sed -E "s/the/this/gI"`

## Single Quote and Double Quote

Single quotes won't interpolate anything, but double quotes will. For example: variables, backticks, certain \ escapes, etc. Enclosing characters in single quotes ( ' ) preserves the literal value of each character within the quotes. A single quote may not occur between single quotes, even when preceded by a backslash.
