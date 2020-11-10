Object-oriented programming intution
====================================

A brief rant about OOP tutorials
--------------------------------

A lot of OOP tutorials convey the basic mechanics of it -- classes, members,
methods, inheritance, and so on -- but not when and why to use them.
Unfortunately, a lot of the guidance on that front -- SOLID, DRY, KISS, coupling
and cohesion, etc. -- is maddeningly abstract for my taste.

I think part of the problem is the focus on asinine, contrived examples without
any real world application. You'll see `Dog` classes with `name` members and
`bark()` methods, but these are totally useless because they're constructed in
isolation without any sense of how such a pointless thing would be used in real
world code.

And that, I think is what gets left out in a lot of OOP discussion: how do
objects get _used_? What is going to consume that mythical `Dog` object? What
use is its `bark()` method? You don't know and you can't know because it's not a
real example. However, I do think they offer us that key insight: that the value
of classes is in the benefit to the things that consume them.

The convenience of types
------------------------

Classes are basically a way of creating custom types. You can see this in
languages like C++ where you indicate the types of function and method
parameters.

```c++
// This function takes two ints (integers) as its arguments.
int add_ints(int a, int b) { return a + b; }

// If we declare our own class...
class Contact {/* omitted */}

// ...we can write a function that takes instances of it as arguments.
void save_contact(Contact c) {/* omitted */}
```

Why is this useful? Well, let's consider the utility of some baked-in Python
types with a contrived example:

```python
>>> def print_length(obj):
...   print(len(obj))
... 
>>> print_length("abc")
3
>>> print_length((1, 2, 3))
3
>>> print_length([1, 2, 3])
3
>>> print_length({1, 2, 3})
3
>>> print_length({1: 1, 2: 2, 3: 3})
3
```

Here we have a function `print_length()` that can take a string, a tuple, a
list, a set, or a dictionary and print its length, without having to do anything
special in the function itself. From our perspective as consumers of Python's
various built-in collection types, this is extremely convenient because we can
easily construct less pointless functions than `print_length()` that can take
an arbitrary collection type and not worry about what exactly it is.

This is actually really powerful, because what this means is we can take an
object whose type we don't know and apply an operation on it without having to
care.

Parameterizable behavior
------------------------

If we can harness the property above, we gain the ability to parameterize
behavior. That is, we can pass objects as parameters to functions (among other
things) and functions can use the objects' behavior in customizable ways.




Recommended Reading
-------------------

Practical Object-Oriented Design: An Agile Primer Using Ruby