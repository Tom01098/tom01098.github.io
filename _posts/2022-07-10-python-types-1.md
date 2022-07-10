---
layout: post
title: An introduction to Python's type hints
date: 2022-07-10 11:44 +0000
---
Type hints give our code the super power of being more sure our Python code does what we want it to do before running it. By using type hints properly and accurately, we can use tools to prevent awkward - and sometimes critical - software mistakes.

In this series, I'll give a full breakdown of how to use all the type hint magic available in the Python language, using practical tips to help you best use them. I'll also cover some more advanced parts of how the language ties itself together.

### Values have types, variables have type hints
Consider the following Python code:

```python
x = 3
y = 4
print(x + y)  # 7
```

How do you know that the `x + y` will work when the code runs?

In this example it's trivial. What if we got `y` from calling a function?

```python
x = 3
y = get_some_value()
# This won't work if `get_some_value` returns something weird, like `None`.
print(x + y)
```

The only thing we can do is trust that `get_some_value` returns something we can add to `3`... and that's the problem that type hints exist to solve; they remove the need to trust that we're using code correctly. Sold? Cool.

Here's our last example, with type hints:

```python
x: int = 3
y: int = get_some_value()
print(x + y)
```

> Let's pin down some terminology before we continue so everything makes sense. A **value** is a an object that exists when the code runs. In the example, `3` is an example of a value. A **variable** is the name we give to a container for that value; `x` is our variable.

A type hint follows a colon (`:`) after a variable. These type hints tell us that `x` and `y` are both of type `int` when the code runs! From this, we can infer that `x + y` is safe, because `int`s can be added together!

Anything that is a type can be used as a type hint (I'll cover specifically what 'being a type' means in the future). For now, just know that it means you can use your own `class`es as type hints!

```python
class Person:
    name: str
    ...

tom: Person = Person(name="Tom", ...)
```

We can say that type hints give us the super power of moving the type from the value to the variable, which exists when we edit the code! This lets us use all sorts of tools to look at these types to help us develop and understand our code better. At the end of this post, we'll look at a powerful tool to make sure we use these hints correctly.

### Functions use types too
Type hints would be pretty useless if we could only use them on variables. We can use them on functions to define the inputs and output types.

```python
def hello_person_name(person: Person) -> str:
    return f"Hello, {person.name}!"
```

We've annotated our function to include the fact that `hello_person_name` takes a `Person` object and returns a `str`! The parameter annotation syntax is the same as we saw previously with variables. The return type appears after the parameters with the `->` arrow.

### Validate it with `mypy`
So type hints are just that; hints. They do nothing at runtime (besides expose some metadata if we want to do some metaprogramming, but that's besides the point here). So how can we actually use them?

There are many great tools in the ecosystem for introspecting programs that you can explore. `mypy` is the de-facto type checker and is part-led by Guido Van Rossum, the creator of Python.

Install `mypy` with:

```shell
> pip install mypy
```

> This post uses the current latest version available, 0.961.

You can then run `mypy` against a file or directory tree.

```shell
> mypy good.py 
Success: no issues found in 1 source file
```

If we wrote incorrectly typed code like below, `mypy` would tell us about it.

```python
def hello_person_name(person: Person) -> int:
    return f"Hello, {person.name}!"
```

```shell
> mypy bad.py 
bad.py:6: error: Incompatible return value type (got "str", expected "int")
Found 1 error in 1 file (checked 1 source file)
```

Lets break down what `mypy` is telling us. In the file `bad.py`, on line `6`, we attempted to return a `str` when the function should be returning an `int`. We can easily action this, and most error messages are easy to understand.

`mypy` is a powerful tool that you can customise as needed and it's all [well documented](https://mypy.readthedocs.io/en/stable/).

### Next steps
We've only covered a small fraction of what type hints have to offer, but what we have covered accounts for most uses of them. So get to work and start peppering them throughout your Python code :).

When I find the motivation, I'll explore what else you can add to your typing arsenal!
