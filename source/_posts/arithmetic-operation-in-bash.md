---
title: Arithmetic Operation In Bash
date: 2019-02-07 16:33:05
tags:
---


To learn about this topic, let's start with a problem.

Snippet of code below is to be implemented:
```bash
a=13
b=12

# Please calculate their quotinent (a / b) 
```

As much as I have learnt about arithmetic operations, there are 4 kinds of feasible solutions for BASH.

- `$(( expression ))` gives the result of the calculation
- Also you can do calculation using `` `expr expression` ``
- To do calculation and assign the result directly, you can try this: `((result = $a + $b))` as well as `((result = a + b))`, where result, `a` and `b` are all shell variables.
- Using bc calculator with pipe.

For that problem, and with the help of the hints, we can provide 4 structurally different solutions respectively.

```bash
# Sol.1 
result=$((a / b))

# Sol.2
result=`expr a / b`

# Sol.3
((result = a / b))

# Sol.4  
result=`echo "scale=16; $a/$b" | bc"`

```

If you try them all out, you will find that only the last solution gives float number as result. So one thing you should bear in mind is that built-in calculation of bash only does integers. To do float points, you must get the assistanse of external tools, in which `bc` is the one mostly referred to.

> Built-in arithmetic operation in bash only does integers.

With this basic experience with arthmetic operation of bash and help of GOOGLE SEARCH, you can now solve more problems.
