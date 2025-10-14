+++
draft = true
date = 2025-09-27
title = "Building Blocks: How I Structure Code"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

**TODO Intro.**

Optimising code for:
- Maintainability
- Testability
- Readability
- Performance

**TODO My approach describes the whole program declaratively as building blocks**

**TODO What type of app are we building?**

## Object Types

Types:
- Data
- Handlers
- Flows
- Contexts
- Factories

### Data
We need to represent things (nouns) in our applications. Data objects help by grouping related attributes.

Data objects are immutable. To edit one, create a new one with the updated values.
I find that removing side effects from code makes it easier to reason about and test.
It's widespread in functional languages for the same reason.

Data objects shouldn't _do_ anything. They should only _store_ data. I would only consider adding a method
to a data object if it presents the same data in a different format that is useful in many contexts.

Where practical, it shouldn't be possible to create an invalid data object.
I perform validation during object construction in the data object's constructor.
This cuts down significantly on edge cases that need to be handled when dealing with the data object,
preventing bugs and making the code easier to reason about.

> In Python, the `dataclass` decorator is a great way to create data objects.
{.tip}

```python
from dataclasses import dataclass


@dataclass(frozen=True, kw_only=True)
class MonthDay:
    month: int
    day: int

    # Validation upon construction. It is not possible to have an invalid `DayOfYear` object.
    def __post_init__(self) -> None:
        assert 1 <= self.month <= 12, "Month must be between 1 and 12"

        if self.month in (1, 3, 5, 7, 8, 10, 12):
            assert 1 <= self.day <= 31, "Day must be between 1 and 31"
        elif self.month in (4, 6, 9, 11):
            assert 1 <= self.day <= 30, "Day must be between 1 and 30"
        else:
            assert 1 <= self.day <= 29, "Day must be between 1 and 29"

    @property
    def is_leap_day(self) -> bool:
        return self.month == 2 and self.day == 29
```

```python
from dataclasses import dataclass

from month_day import MonthDay


@dataclass(frozen=True, kw_only=True)
class Person:
    first_name: str
    last_name: str
    birthday: MonthDay

    @property
    def full_name(self) -> str:
        return f"{self.first_name} {self.last_name}"
```

> This is a bad way of storing names.
> Many people don't have names following the *first name/last name* structure.
> If you're unfortunate enough to need name handling in your application, first read the [W3C guidelines](https://www.w3.org/International/questions/qa-personal-names).
{.warning}

### Handlers
A handler is an object which performs some business logic given some data objects.

It accepts arguments which can be used to customise its behaviour during construction.

Once constructed, a handler is immutable. The methods that it exposes can be called many times with different arguments.
I find that I almost always need only one method on a handler; if I'm wanting to define a second, that usually implies
that I want two handlers that exist separately.

This might sound like a more complex function where we could just pass the static and the dynamic arguments in!
But it gives us a lot of flexibility, especially around testing.

```python
from dataclasses import dataclass, field
from datetime import date

from month_day import MonthDay
from person import Person


@dataclass(frozen=True, kw_only=True)
class BirthdayHandler:
    leap_day_equivalent: MonthDay = field(
        default_factory=lambda: MonthDay(month=2, day=28)
    )

    def is_birthday(self, person: Person, today: date) -> bool:
        is_leap_year = today.year % 4 == 0 and (
            today.year % 100 != 0 or today.year % 400 == 0
        )

        if not is_leap_year and person.birthday.is_leap_day:
            return (
                today.day == self.leap_day_equivalent.day
                and today.month == self.leap_day_equivalent.month
            )
        else:
            return (
                today.day == person.birthday.day
                and today.month == person.birthday.month
            )
```

<details>
<summary>Unit tests for the birthday handler</summary>

```python
from datetime import date

from birthday_handler import BirthdayHandler
from person import Person
from day_of_year import DayOfYear


def test_birthday_normal() -> None:
    person = Person(
        first_name="Jane",
        last_name="Doe",
        birthday=DayOfYear(month=1, day=1),
    )
    handler = BirthdayHandler()
    assert handler.is_birthday(person=person, today=date(2025, 1, 1))
    assert not handler.is_birthday(person=person, today=date(2025, 6, 3))


def test_birthday_leap_year() -> None:
    person = Person(
        first_name="Jack",
        last_name="Lousma",
        birthday=DayOfYear(month=2, day=29),
    )
    handler = BirthdayHandler()
    assert handler.is_birthday(person=person, today=date(2020, 2, 29))
    assert not handler.is_birthday(person=person, today=date(2020, 2, 28))
    assert handler.is_birthday(person=person, today=date(2021, 2, 28))
    assert not handler.is_birthday(person=person, today=date(2021, 3, 1))

    # Configure the handler to consider a leap day equivalent to March 1st.
    day_after_handler = BirthdayHandler(leap_day_equivalent=DayOfYear(month=3, day=1))
    assert not day_after_handler.is_birthday(person=person, today=date(2021, 2, 28))
    assert day_after_handler.is_birthday(person=person, today=date(2021, 3, 1))
```

</details>

**TODO Introduce abstracts**

### Flows
A flow is the 'glue' that connects multiple handlers.

Flows are immutable in the same way as handlers.

**TODO Executing a controller executes the whole thing, not returning a handler.**

### Contexts
A context is used to track state during the execution of a flow. 
They are the only objects that are mutable in my coding style.

The context lives as long as the flow executes for a single request.
It is usually created at the start of the flow's execution. It might be returned from the flow directly 
or transformed into some other object first.

**TODO Augment the existing example with a context tracking 
which users were notified and which failed, for what reasons.**

### Factories
A factory is an object that helps create other objects.

It can be easy to get carried away with factories, abstracting away a lot of the configuration options 
behind a small set of flags.

**TODO Give a bad example of a factory that hides details, then a builder pattern that is good.**

## Small Patterns

- from - only downwards, mention circular imports
- global config, parse at top level, dynamic config can be handled and passed down
