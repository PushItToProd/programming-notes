When to use classes
===================


Case 0: When the library/framework tells you to
-----------------------------------------------

This is the case you'll probably encounter first. Sometimes you'll be using some
existing tool and it'll just tell you to use a class, so you gotta.

There are a couple likely cases where this will come up.

### Configurable behavior

In the simpler case, you have classes with pre-determined methods, like
[Django's class-based
views](https://docs.djangoproject.com/en/3.0/topics/class-based-views/intro/#using-class-based-views),
where you're basically given a specific set of methods to implement and Django
handles the rest. Roughly speaking, you can think of these as templates for you
to follow in implementation. The library or framework tells you what members
your class needs and what types they should have, and you put them where it
wants them.

These are hopefully relatively straightforward to understand. By implementing
the class as the library expects, you're enabling it to call those methods for
you when they're needed. For example, Django calls your class-based view's
methods when it gets a request and then handles the response it generates. This
allows you to focus on the behavior of your view in a specific context without
worrying about any of the other implementation details of request handling.

### ✨ Magic ✨ classes

The more confusing case consists of classes which do something special behind
the scenes to handle dynamic members that you define. Some common examples you
might see are Python test cases and Django models.

Python's `TestCase` class allows you to define methods with names that begin
`test_`, which can be run a few ways, such as invoking `python -m unittest
test_strings.py` in your shell. This will automatically run each appropriately
named method of your `TestCase` classes, detecting whether they fail and
handling any errors along the way.

```python
# test_strings.py
# An example of Python's TestCase class via 
# https://docs.python.org/3/library/unittest.html#basic-example
import unittest


class TestStringMethods(unittest.TestCase):
    def test_upper(self):
        self.assertEqual('foo'.upper(), 'FOO')

    def test_isupper(self):
        self.assertTrue('FOO'.isupper())
        self.assertFalse('Foo'.isupper())
```

Django models are even fancier. If you define a class like the one below, Django
will automatically create a database table with appropriate columns on it and
allow you to use instances of your class to interact with the table, creating,
updating, and querying records.

```python
# models.py
# An example of a Django model adapted from
# https://docs.djangoproject.com/en/3.1/topics/db/models/.
from django.db import models


class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)


# somewhere else, you can use this like so:
p = Person(name="Fred Flintstone", shirt_size="L")
p.save()
print(p.name)  # this will print "Fred Flintstone"
```

In both of these cases, the precise members being declared are not determined by
the tool you're using. Nothing in Python's `TestCase` class requires a method
named `test_upper()`, but somehow the `python -m testcase` command is able to
find and run it automatically. Likewise, nothing in Django's `Model` class
expects fields named `first_name` or `last_name` to exist, but it can still
magically find them and enable you to interact with the database using them.

With these classes, you're once again going to create them when they're dictated
by the tools you're using, and just like the simpler classes with explicitly
required members, you'll find that these classes have specific expectations on
what they contain.

Looking at our examples above, you can see that they each follow certain
expectations. The `TestCase` class needs to define methods with names like
`test_something` that call various `assertSomething` methods, while the `Model`
class needs to define fields assigned things like `CharField`. It wouldn't make
sense to define `first_name = models.CharField(...)` on your `TestStringMethods`
class, nor to copy the `test_upper()` method over to your `Person` class.

Understanding exactly why these classes work as they do requires a deeper
understanding of classes in general, but in these cases it's definitely possible
to get by without really knowing more, as long as you accept that it might not
totally make sense at first.

Next, let's turn to cases where you'd use classes yourself.

Case 1: Data structures
-----------------------

```python
class Person:
    def __init__(self, first_name, last_name):
        self.first_name = first_name
        self.last_name = last_name
```

Case 2: Configurable behavior
-----------------------------

```python
jira = JIRA(your_jira_url, your_jira_credentials)
jira.get_issue('ABC-555')
```

Case 3: Parameterizing behavior
-------------------------------

### Simpler case

* create a generic function or class that interacts with JIRA
* inject a `JIRA` object with params for the JIRA server you want
* now you can use the same code on multiple JIRAs

### More complex case



```

```


Mixing and matching behavior with inheritance
---------------------------------------------
