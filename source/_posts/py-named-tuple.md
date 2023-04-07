---
title: "Step-by-Step Guide to Using namedtuple in Python for Beginners"
tags:
  - python
  - style
categories:
  - learning log
date: 2022-01-23
description: Learn how to use namedtuple in Python with our step-by-step guide designed for beginners. Our tutorial provides practical examples and best practices for creating lightweight and memory-efficient data structures in Python. Whether you're new to programming or an experienced developer, our tutorial will help you master the techniques needed to take advantage of Python's powerful features and simplify your code.
---


I recently got side-tracked into exploring the basics of `namedtuple()` as 
I got a glimpse of its usage in our engineering codebase. Here's my summary:


## Mutable and Hashable

To understand the behavior of `namedtuple()`, it is best to also visit the
concept of Python object's mutability and hashability. These two concepts are
closely linked.

**Hashability**: an object's is hashable when its hash value never changed during its lifetime

>Most of Python’s immutable built-in objects are hashable; 
> mutable containers (such as lists or dictionaries) are not;
> immutable containers (such as tuples and frozensets) 
> are only hashable if their elements are hashable. 
> **Objects** which are **instances** of user-defined classes are hashable by default

**Mutability**: an object with a fixed value and cannot be altered is immutable
(For example: `int`, `float`, `string`, `tuple`).
in contrast, an object can keep its value while keeping its `id()` is mutable.
(For example: `list`, `dict`)

### `hash()` and `id()`

**identity**: `id()`, the identity of the two same value variables are the same
If two objects (that exist at the same time) have the same identity, they're actually two references to the same object.
>The `is` operator compares items by identity, `a is b` is equivalent to `id(a) == id(b)`.

**hash value**: `hash()`, hash value is based off an object's value, and hash value
must remain the same for the lifetime of the object. If an object is mutable,
then it doesn't make sense for it to have hash.
>The hash value is an integer which is used to quickly compare dictionary keys or sets.

### Why Hash?

Hash values are very useful, as they enable quick look-up of values 
in a large collection of values, it's commonly used in `set` and `dict`. 

with `if x in elements:`:
- In a `list`, Python needs to go through the whole list 
and compare `x`'s value with each value in the list elements. 

- In a `set`, Python keeps track of each element's hash, 
Python will get the hash-value for `x`, 
look that up in an internal structure and find elements that have the same hash as `x`.

It also means you can have non-hashable objects in a `list`, 
but not in a `set` or as **keys** in a `dict`. 

<a name='hash-example'></a>
### Example:

There is no way to change an `int` object's value without re-assigning (copy) it
to a different object.

```python
x = 5
# id(x) is equal to 3054677212104
x = 6
# id(x) is equal to 3054677212080
```

But for `list`, you can edit its value after assignment while keeping its `id()`
the same. (note: use `list` built-in function rather than re-assignment,
this is the same for `x.sort` vs. `x=sorted(x)`)

```python
x = [5]
# ORIGINAL: id(x) is equal to 3054706521672
x.append(6)
# CORRECT: id(x) is equal to 3054706521672
x = [5, 6]
# WRONG: id(x) is equal to 3054713375816
```

## NamedTuple

A data class are just regular classes that are geared towards storing state, 
rather than containing a lot of logic, `namedtuple()` is one kind of data classes.
> Every time you create a class that mostly consists of attributes, you make a data class.

With `namedtuple()`, you can create **immutable** sequence types 
that allow you to access their values using descriptive field names 
and the dot notation instead of unclear integer indices.

### Initialization

1. **typename**: `str`, class name of the `namedtuple`
2. **field names**: names that are used to access values in the `namedtuple`, it can be declared using any of the following:
    - iterable of strings: ["a", "b", "c"]
    - a string with name seperated by white spaces: "a b c"
    - a string with name separated by commas: "a, b, c"

Example:
```python
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])

>> Point
# <class '__main__.Point'>
>> Point(2, 4)
# Point(x=2, y=4)
```

### Access and Edit Value

It is very straight-forward to access a tuple's attribute value using dot notation

this gives `namedtuple` a great edge against `dict` or `tuple`.
```python
Person = namedtuple('Person', 'name children')
jj = Person('Johnny J', ['Tobby'])

>> jj.children
# ['Tobby']
```

Since `namedtuple` is immutable, you can't assign value to its attribute;
what you can do is to use `._replace()`; and also, its value can be mutable, like
a `list`.

```python
>> jj.children = ['Tobby', 'Wang']
# AttributeError: can't set attribute

>> jj.children.append('Wang')
# Correct

>> jj = jj._replace(name='John J')
# Correct
```

### Using `._asdict()`

The built-in function `._asdict()` converts `namedtuple` into a dictionary.

```python
Person = namedtuple("Person", "name age height")
jane = Person("Jane", 25, 1.75)

>> jane._asdict()
# {'name': 'Jane', 'age': 25, 'height': 1.75}
```

and to generate a namedtuple object from dictionary

```python
d = {
   'name': 'Jane',
   'age': 25,
   'height': 1.75
}

jane = Person(**d)
```

## `@dataclass`

`@dataclass` came out after Python 3.7, which is similar to `namedtuple`, but they are mutable.
thus, we can set value to a `@dataclass` attribute.

```python
from dataclasses import dataclass

@dataclass
class Person:
    name: str
    age: int
    country: str = "Canada"

jane = Person("Jane", 25)

>> jane.name = "Jane Doe"
>> jane.name
'Jane Doe'
```

### frozen attribute

if we want `@dataclass` to behave like `namedtuple` with an un-editable "protected" attribute,
just use `@dataclass(frozen=True)`.

### override `__iter__()`

`@dataclass` are also not iterable by default, unlike `namedtuple`. We can achieve
that by implementing the special method `.__iter__()`:

```python
from dataclasses import astuple, dataclass

@dataclass
class Person:
   ...
   def __iter__(self):
        return iter(astuple(self))
```

## Subclassing `namedtuple`

Subclassing `namedtuple` gives us additional functionality.

```python
BasePerson = namedtuple("BasePerson", "name birthdate country")

class Person(BasePerson):
    """A namedtuple subclass to hold a person's data."""
    __slots__ = ()
    
    def __repr__(self):
        return "Name: {}, age: {} years old".format(self.name, self.age)
    
    @property
    def age(self):
        return (date.today() - self.birthdate).days // 365
```

In the above example, subclassing from `namedtuple` provides us better documentation
(i.e. `Person.__doc__`), better string representation (i.e. `print jane`) and an
extra property to access based off a Person's instance attribute value.

### `__new__()` constructor

[Zechong Hu's Blog - Inheritance for Python Namedtuples](http://zecong.hu/2019/08/10/inheritance-for-namedtuples/)

To override the constructor for `namedtuple` class with default value:

```python
BasePerson = namedtuple("BasePerson", ["name", "birthdate" ,"country"])

class Person(BasePerson):
    __slots__ = ()
    def __new__(cls, name, birthdate="2000.01.01", country=None):
        return super(Person, cls).__new__(cls, name, birthdate, country)
```

### `__slots__`

The special attribute `__slots__` explicitly state what attribute you want
your class instances to have.

By default, when an instance (object) is created, 
`__dict__` is used to store an object's (writable) attributes.
A **dynamic** dictionary:
1. requires more memory 
2. takes longer time to create.

Because `namedtuple` makes immutable instances that are lightweight,
we need to prevent the creation of `__dict__` to get the benefit while subclassing
by setting `__slots__` as empty tuple.

In a more general note, please consider using `__slots__` when creating
tons of objects, this saves memory and time when instancing. 

#### Comparison `__dict__` vs. `__slots__`

```python
class Person(object):
   def __init__(self, name, age):
      self.name = name
      self.age = age

john = Person('john', 15)

>> john.__dict__
# {'age': 15, 'name': 'john'}
```

```python
class Person(object):
    __slots__ = ['name', 'age']
    def __init__(self, name, age):
        self.name = name
        self.age = age

john = Person('john', 15)

>> john.__slots__
# ['name', 'age']
>> john.__dict__
# AttributeError: 'Person' object has no attribute '__dict__'
```

## Reference

[Python Docs - Glossary](https://docs.python.org/3/glossary.html)

[Medium megha mohan - Mutable vs Immutable Objects in Python](https://medium.com/@meghamohan/mutable-and-immutable-side-of-python-c2145cf72747)

[Real Python - namedtuple](https://realpython.com/python-namedtuple/)

[Stack Overflow - What are data classes and how are they different from common classes?](https://stackoverflow.com/questions/47955263/)

[Geeks for Geesk - Use of \_\_slots__](https://www.geeksforgeeks.org/python-use-of-__slots__/)

[Stack Overflow - Usage of \_\_slots__?](https://stackoverflow.com/questions/472000/)

[Stack Overflow - Difference between hash() and id()](https://stackoverflow.com/questions/34402522)

[Stack Overflow - Two variables in Python have same id, but not lists or tuples](https://stackoverflow.com/questions/38189660/)

[Stack Overflow - What does hash do in python?](https://stackoverflow.com/questions/17585730)

