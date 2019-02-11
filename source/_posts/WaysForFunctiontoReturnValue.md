---
title: Ways For Function to Return Value
date: 2019-02-07 16:33:05
tags:
---


Modularization design of code has many advantages. As an essential role regarding to code modularization, exploitation of functions ought to be gotten the hang of.

The basic function declaration syntax is like this:
```bash
function funcIdentifier() {
    # commands
}
```
or this
```bash
funcIdentifier () {
    # commands
}
```

After the declaration, you can invoke the function like this
```bash
funcIdentifier param1 param2
```

And inside of the function, you can easily get the parameters using `$1`, `$2`, `$3` ,etc. 



But how to get the result value ? There are a few ways to do the trick.

- Using `echo` inside of function and with the help of command substitution you get the result


* `eval` command'll be evaluated twice. First is of the contents at both side of equal sign, the second one is the actual value assignment
* Global variables



In order to get stuffs across intuitively, I provide examples for first two approaches :

```bash
# 1.
function sum () {
    echo `expr $1 + $2`
}

result=$(sum 12 13)

# 2.
function sum () {
    eval $1='`' "expr $2 + $3" '`'
}
result=0
sum result 12 13
```



Understanding these two techniques of returning values from functions will help you understand more about the shell interpreter.

