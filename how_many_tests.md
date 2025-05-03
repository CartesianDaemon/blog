# "How many tests is the right amount of tests" -> "If you would test it manually, you should test it automatically instead."

Rewrite as FAQ?

## How many tests? Enough to ensure...

* When someone (you, another developer, a user) uses the program, they do not discover that it's suddenly been broken.
* When you change the code, you know you haven't broken anything.

## How many tests? If...

* If you would test this manually, you should test it automatically instead. (Or if that's impractical, documented when you need to rerun the manual tests.)
* If you change the code, you should have enough tests that you know you haven't broken any.

## Everything else

If you need to certify particular things work, then you definitely need that, you don't need me to tell you that.

I find most other things flow from these principles. Hard to write that many tests? Maybe you need some better test automation. Unit tests or integration tests? Whichever is most convenient.

# Also see:

* Unit tests vs integration tests. Narrowness of tests.
