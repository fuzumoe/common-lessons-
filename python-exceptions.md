# 🐍 Python Exceptions --- A Complete Beginner-to-Intermediate Guide

## 🔰 Introduction

In Python, exceptions are errors that occur **during the execution** of
a program.\
Instead of silently failing, Python immediately stops and shows you what
went wrong.

Understanding exceptions makes your programs:

-   more reliable\
-   easier to debug\
-   safer for users\
-   more predictable

This lesson covers **everything a beginner should know**, from basic
try/except to raising custom exceptions.

------------------------------------------------------------------------

# ⭐ What Is an Exception?

An **exception** is a runtime error --- something unexpected that breaks
the normal flow of a program.

Example:

``` python
print(10 / 0)
```

Output:

    ZeroDivisionError: division by zero

When Python sees an error it cannot handle, it "raises" an exception.

------------------------------------------------------------------------

# 🧭 Why Exceptions Are Important

Without exception handling, your program crashes as soon as something
goes wrong.

Useful for:

-   validating user input\
-   handling missing files\
-   detecting invalid data\
-   writing network or database programs\
-   building robust applications

------------------------------------------------------------------------

# 📚 Common Python Exceptions

  Exception             Meaning
  --------------------- --------------------------------------
  `ZeroDivisionError`   Dividing by zero
  `ValueError`          Wrong value type, e.g., `int("abc")`
  `TypeError`           Using wrong type, e.g., `"abc" + 5`
  `IndexError`          List index out of range
  `KeyError`            Missing dictionary key
  `NameError`           Variable not defined
  `FileNotFoundError`   File is missing
  `AttributeError`      Attribute does not exist
  `ImportError`         Module cannot be imported
  `RuntimeError`        Generic runtime error

------------------------------------------------------------------------

# 🏛️ Python Exception Hierarchy (Simplified)

    BaseException
     ├── Exception
     │    ├── ArithmeticError
     │    │     ├── ZeroDivisionError
     │    │     └── OverflowError
     │    ├── ValueError
     │    ├── TypeError
     │    ├── IndexError
     │    ├── KeyError
     │    ├── FileNotFoundError
     │    └── RuntimeError
     └── SystemExit
     └── KeyboardInterrupt

------------------------------------------------------------------------

# 🛡️ Handling Exceptions with try/except

## Basic Structure

``` python
try:
    # Code that might fail
except SomeException:
    # What to do if that error happens
```

------------------------------------------------------------------------

# 🎯 Example 1: Handling Incorrect Input

``` python
try:
    age = int(input("Enter your age: "))
except ValueError:
    print("Please enter a number.")
```

------------------------------------------------------------------------

# 🎯 Example 2: Multiple Except Blocks

``` python
try:
    x = int("hello")
    y = 10 / 0
except ValueError:
    print("Cannot convert text into a number.")
except ZeroDivisionError:
    print("Cannot divide by zero.")
```

------------------------------------------------------------------------

# 🎯 Example 3: Multiple Exceptions Together

``` python
try:
    risky_operation()
except (ValueError, TypeError):
    print("Invalid input or operation.")
```

------------------------------------------------------------------------

# 🔥 Raising Exceptions Manually (`raise`)

``` python
raise Exception("Something went wrong")
```

Example:

``` python
age = -5
if age < 0:
    raise ValueError("Age cannot be negative")
```

------------------------------------------------------------------------

# 🧙 Custom Exceptions

``` python
class InvalidAgeError(Exception):
    pass
```

------------------------------------------------------------------------

# ⚠️ Best Practices

-   Catch specific exceptions\
-   Avoid bare `except:`\
-   Use `finally` for cleanup\
-   Use `raise` for invalid states

------------------------------------------------------------------------

# 🧪 Practice Activities

1.  Divide two numbers (handle ValueError, ZeroDivisionError)\
2.  Read a file (handle FileNotFoundError)\
3.  Create custom exception PasswordTooShortError\
4.  Validate temperature using raise

 