When to use classes: a brief guide
==================================

Data structures
---------------

Conceptually, I think the simplest use case for classes is as a simple data
structure.

**Contrived example:** If you're writing a to-do list application, you might
start by simply modeling your list as a list of strings.

```python
todos = [
  "Get milk",
  "Finish OOP tutorial",
  "Rewatch The Matrix",
]
```

This might work for a very simple case, but how do you handle completed items?
You could just delete them, but maybe you want to be able to see them later. And
you may also want to associate other data with them, like a due date, tags, and
so on. Eventually, a list just won't cut it.

Let's start with just completion state, though.

You could try handling this with more of Python's built-in data types. You could
use a list of tuples, for example:

```python
todos = [
  ("Get milk", True),
  ("Finish OOP tutorial", False),
  ("Rewatch The Matrix", False),
]
```

However, that's not very descriptive, plus you have to be careful when adding
new fields to be sure that every place you use them is correctly updated.

Dictionaries are another option here, but while they make the field names more
explicit, they still face the problem of adding new ones, not to mention that
typos may cause subtle bugs.

```python
todos = [
  {"task": "Get milk", "completed": True},
  {"task": "Finish OOP tutorial", "completed": False},
  {"task": "Rewatch The Matrix", "completed": False},
]
```

For example, suppose we now want to add a due date field as well. The problem is
that you either need to update every piece of code that generates to-do items to
include the due date or you need to make sure that every location in code that
expects a due date can handle the case where the field is absent.


```python
class TodoItem:
    def __init__(self, task, complete=False, due_date=None):
        self.task = task
        self.complete = complete
        self.due_date = due_date


todos = [
  TodoItem()
]
```

A lot of the time we have units of data that are too complex to reasonably fit
in a regular data type. For example, if you're writing a to-do list application
in Python and you want your to-do items to have a description, completion state,
assignee, and due date, you could just pass around a dictionary or a tuple
everywhere, but defining a class lets you make the fields of the object explicit
and makes it easier to extend in the future. It's also much easier to identify
when an object of your custom type is being constructed, so if you want to add
a field or change one, it's much easier to track down everywhere it's used than
if you just implicitly construct dictionaries.

Parameterizing behavior
-----------------------

Metaprogramming magic
---------------------

So this is a bit of an edge case. Sometimes, classes are more 