---
title: Usages of tr
date: 2019-02-14 22:15:54
tags: [linux, shell, bash, tr]
---

The command `tr` is an abbreviation of "translate" or "transliterate". 

In general, the `tr` command is executed in the formula: `tr [OPTION] SET1 [SET2]`. It's a character-wise command.



#### Example

For first impression, look at the example below.

```bash
$ tr "10" "-_" <<< "110101EOF"
--_-_-EOF
```

Basically, the `tr` command translates "1" into "-" and "0" into "_". The remain part stays the same.



#### Other Options

After that easy example, let's find out what else we can do:

- `-s` : Replace consecutive repeats of character in the `SET1` with single occurrence. It can be used like this : `tr -s "[a-z]"`.
- `-c` : It indicates the complement of `SET1`. 
- `-d` : Delete all characters listed in `SET1`.
- `-t` : `SET2` is extended to the length of `SET1` by repeating its last character as necessary. Excess characters  of `SET2` are ignored.

These options are to be filled into the general formula introduced at the beginning of this article. Straightforward though it is, It still takes practice to master.



For more insights, refer to GOOGLE SEARCH.
