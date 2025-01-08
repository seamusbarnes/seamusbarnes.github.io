---
layout: post
title: "What is the Python Walrus Operator (:=)?"
date: 2025-01-08 10:00:00 +0000
categories: example
tags: [python]
---

The `Python` "walrus operator" is a neat little operator introduced in `Python 3.8` that allows you to assign a variable that is immediately available for use on the same line. This makes code more concise and readable (if you understand how the operator works!).

For example, instead of defining two separate variables and then performing an operation using them, you could define a single variable and then simultaneously asign the second variable whilst performing an operation on them both together:

```python
# attempting without the walrus operator
x = 10
y = x + 5
print(y)

# attempting with the walrus operator
a = 10
print(b := x + 5)
```

The walrus operator is even more powerful when used in loops. For example, instead of continuously re-assigning the varialbe `value`, you could asign it and evaluate it simultaneously:

```python
# attempting without the walrus operator
value = input("Enter some input: ")
while value != "quit":
    print(f"You entered: {value}")
    value = input("Enter some input: ") # you must re-assign a new variable in the while loop

# attempting with the walrus operator
while (value := input("Enter some input: ")) != "quit": #the variable is asigned and evaluated simultaneously
    print(f"You entered: {value}")
```

The walrus operator can also be used to make .txt processing more straightforward:

```python
# attempting without the walrus operator
with open("data.txt") as file:
    line = file.readline()
    while line and "keyword" not in line:
        print(line.strip())
        line = file.readline()

# attempting with the walrus operator
with open("data.txt") as file:
    while (line := file.readline()) and "keyword" not in line:
        print(line.strip())
```

Two words of caution!

1. The walrus operator never actually needs to be used. It is always possible to be more verbose and assign variables and then operate on them separately. This is preferable sometimes to avoid confusion.
2. Variable assignments made using the walrus operator have local scope. They only exist within the block (e.g. function or class) in which they are assigned.
