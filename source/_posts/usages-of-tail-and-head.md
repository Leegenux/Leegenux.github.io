---
title: Usages of tail and head
date: 2019-02-14 00:00:56
tags:
---

The commands `tail` and `head`, as their names suggest, respectively print the beginning and ending of files. 



You can easily understand the usages of `tail` with this example: `tail -n 50 /var/log/auth.log`. This command shows last 50 lines of the `auth.log` file. And the usages of `head` are alike. There is only one basic and intuitive option.

#### Specifying Unit

You wanna lines? Use `-n`, and there are `-c` for characters, `-b` for bytes.

With this knowledge, you can print first 5 characters of a file with command: `head -c 5 filename`.



#### Special Usage of tail

To show a real-time output of a changing file, we can use the `-f` or `--follow` options: `tail -f /var/log/auth.log` prints the end of the file and updates the output when lines are appended to file.

It's really helpful and handy when you debug your Linux system under some circumstances.


