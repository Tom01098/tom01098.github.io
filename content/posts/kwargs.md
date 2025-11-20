+++ 
draft = true
date = 2025-10-15
title = "Avoiding Kwargs Antipatterns"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

## Introduction

-- Most bad code comes over time, taking the short cut.

> Explicit is better than implicit.

-- Note about upcoming code structure post.

-- These are all simplifications of examples I've seen in the wild.

-- Reaching for kwargs often indicates a structural problem.

## The Spaghetti Interface

### Solution: Describe the What, not the How

## The Bad Factory
A factory is some code which abstracts the creation of an object. These are great — when used properly — because they
segregate creation from use.

For example, let's imagine we work for a car manufacturer. The company thinks that it might start creating other types
of vehicles in the future, so we shouldn't couple our code too tightly to cars. Keeping things simple for demonstration,
we might model cars like this:

```python
from abc import ABC
from dataclasses import dataclass


class Vehicle(ABC):
    pass


@dataclass
class Car(Vehicle):
    seats: int
```

We then decide that it would be best to abstract away the construction of a `Vehicle` object behind a factory which 
takes the name of the vehicle type and the `seats` parameter. We could then call it like this:

```python
vehicle = get_vehicle(vehicle_type="car", seats=4)
```

The definition is trivial and boilerplate-y:

```python
def get_vehicle(vehicle_type: str, seats: int) -> Vehicle:
    match vehicle_type:
        case "car":
            return Car(seats=seats)
        case _:
            raise ValueError(f"Unknown vehicle type: {vehicle_type}")
```

After some time, the company decides it wants to enter the defence industry, starting with producing tanks.

We might model a tank like this:

```python
@dataclass
class Tank(Vehicle):
    ammunition_type: str
```

We now have a problem. Different types of vehicles need completely unrelated attributes. It definitely doesn't make
sense to describe a car as having a type of ammunition, for example! But we need to be able to pass those attributes
into our factory. This is where a lot of developers reach for kwargs:

```python
def get_vehicle(vehicle_type: str, **kwargs) -> Vehicle:
    match vehicle_type:
        case "car":
            return Car(**kwargs)
        case "tank":
            return Tank(**kwargs)
        case _:
            raise ValueError(f"Unknown vehicle type: {vehicle_type}")
```

> In this example, we directly `**` (unpack) the kwargs into the constructor. A proper factory might do something more
complex, such as validating certain combinations of related parameters.
{.info}

This is functional, letting us write code like this:

```python
>>> get_vehicle(vehicle_type="tank", ammunition_type="armour piercing")
Tank(ammunition_type='armour piercing')
```

But we can now pass in incorrect parameters for the type of vehicle, causing an exception within the factory:

```python
>>> get_vehicle(vehicle_type="car", ammunition_type="armour piercing")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/home/tom/kwargs/factory.py", line 22, in get_vehicle
    return Car(**kwargs)
           ^^^^^^^^^^^^^
TypeError: Car.__init__() got an unexpected keyword argument 'ammunition_type'
```

The factory is now hard to reason about because the calling code needs to know its implementation details, something
that it tries to hide.

The `get_vehicle` factory only gets harder to use as:
- more vehicle types are added.
- more parameters are added to some, not all, vehicle types.
- rules become baked deep into the factory for what combinations of parameters are allowed.
- the distance between the caller and object construction increases.

### Solution: Don't Use a Factory
Think carefully before you reach for a factory. I find that object construction is usually something that's undesirable
to abstract over. A car and a tank aren't things that have much in common; it's hard to think of code that would
construct one in the abstract, without knowing what you were constructing! However, it's easy to think of _behaviour_
they share.

In the vehicle example, one thing we could reasonably expect all vehicles to do is handle their speed. Adding a 
function that accelerates the vehicle makes sense for all vehicle types. But I prefer to handle construction of the
`Car` that is passed into it explicitly.

```python
def accelerate_forward(vehicle: Vehicle, value: float) -> None:
    vehicle.speed += value

    
if __name__ == "__main__":
    car = Car(seats=2)
    accelerate_forward(car, 1)
```

> OOP thinking would have the `accelerate_forward` function be a method on the `Vehicle` base class. I try to avoid
having methods on base classes. I prefer my base classes to act more like traditional interfaces, with other functions
and objects operating on them. I'd like to dive into that in more detail in the future!
{.info}

## Summary
-- Better to be concrete than abstract until it's needed.

Recall the Zen of Python:

> Explicit is better than implicit.
