# Test Driven Development

* write the least amount of code possible
* double loop
* red-green-refactor
* cheating
* checklist-as-code
* it's a discipline

The practice of TDD really boils down to this: First, you write a test that
declares what you think your code should do when it's finished. Then, you run
that test and see where it fails. Then, you write a tiny bit of code to fix the
error and run the test again. Repeat until the test is passing, then do any
refactoring and cleanup as needed, decide if you need any more tests, and when
you're finally satisfied, move on.



I think TDD is often misunderstood because people hear an admonition to "write
your tests first!" but not an explanation of the advantages. Tests are often
thought of as a tool for adding reliability and, from a pure reliability
perspective, it's hard to see what benefit comes from writing tests about code
you haven't even written, especially when so much programming is exploratory and
you have to write it to figure out what you should test.

The thing is, in my opinion, TDD is not about reliability. At least, not the
kind of reliability you probably associate with traditional testing. In fact, I
think it's important to distinguish between 

In my opinion, test-driven
development is much easier to make sense of if you view it first and foremost as
a strategy for offloading mental overhead into the computer.

Let's think about the traditional, "natural" approach to writing code. You start
with some objective -- implement a feature, fix a bug, tweak something
aesthetically, change a data format, whatever -- and then you write code to fix
it. At any given point in the process, you invariably have a bunch of stuff
floating around in your head. You have your mental model of the code, your
expectation of the objective you're trying to accomplish, and a bunch of other
stuff that pops up along the way (tweaks you want to make, ideas for
refactoring, issues you've spotted that need cleanup, a super cool new feature
you want to implement, etc.) Throughout the whole process, you have to juggle
all this information in your head, perhaps paging some of it out to notes as you
go. Eventually, you think you've got it all hammered out, so you run the code
and try to see if it worked. Hopefully this is a relatively quick process or
else this becomes tedious. Unfortunately, most of the time, you find there's
something you missed, so you keep plunking away until you've gotten it working.






## Double loop TDD

In brief:

1. Write a functional test.
2. Write unit tests and fix them until the functional test passes.
3. Repeat.

In detail:

1. Write a functional test that describes the behavior that you want, usually
  framed as a narrative about the user.
2. See where it fails.
3. Write a unit test for the first piece of behavior needed to fix the failure.
4. See where the unit test fails.
5. Write the code to fix the unit test.
6. Check that the unit test passes.
7. Check where the functional test fails now.
8. Repeat from 3 until the functional test passes.
9. Add more test cases if you aren't convinced they make sense.
10. Refactor the code until you're happy with it.
11. Repeat.