# Angel Objects

TODO: Lead in with a motivating example with multiple angels, some global, some not?
TODO: Boil down examples to more obvious ones. (Maybe? If that's sufficiently motivating?)
TODO: Include version with explicit angel parameter. Maybe even without decorators?
TODO: Mention where I got with implementation?

## Motivating problem

Sometimes one object is needed by many functions in the call stack and you need to decide between making it a global, or adding an identical parameter to almost every function in the hierarchy. I've seen this in both high level and low levels areas of programming including:

* Different parts of the program use a logging api (but might use different logging levels/settings/destinations).
* Different parts of a program use different allocation strategies from a custom memory allocator.
* Some GUI widgets have some settings that control how their sub-widgets draw themselves.
* You want to split parts of a large class into other classes, but they still need some common functionality from the large class instance.

I picked examples where using a global isn't satisfying because you want different values in different places. I think that passing the parameter to every relevant function (or adding a member to every class) is the right way, if we can avoid it needing a lot of boilerplate.

If we can fix the boilerplate I think that's a pattern which would be powerful in lots of other situations, but I think people got into the habit of ignoring cases which couldn't be solved by the techniques they already had available.

We might well end up adding quite a few "semi-global" objects. Like functions or classes this adds to total lines of code, but which add clarity, because the information moves out of "the programmer has to memorise it" and into "you can see at a glance which functions use this value and which don't".

## Proposed syntax

An `@Angel(angel_bazflags: Bazflags)` decoration on a function makes the following changes:
1. Adds a `angel_bazflags: Bazflag` argument to the end of the argument list.
2. Rewrites any function calls in the function body to add angel_bazflags to the argument list if it's not already present, if that function also had the `@Angel(angel_bazflags: Bazflags)` annotation.

An `@Angel(angel_bazflags: Bazflags)` decoration on a class declaration or implementation makes the following changes:
1. Adds an `angel_bazflags` member of type `Bazflag` to the class. Constructors are treated as having the same decorator, and automatically initialise the `angel_bazflags` class member with the additional `angel_bazflags` parameter.
2. Class member functions are rewritten so that calls to functions with the decorator are rewritten to add the `angel_bazflags` argument if it's not already present, with the value of the `angel_bazflags` class member variable.

### Notes on use:

* You can mix and match angel decorators.
* It's trivial to use (or the default behaviour of "passing along the corresponding argument from the current namespace", but it's easy to specify a different value when that's what you want. And both are visible at a glance without adding visual clutter.
* It's quick to see which angels a function has, and which functions don't use a widely used angel.
* It's quick to add or remove the decorator from a function. (If you need to add or remove a decorator from lots of functions you might still need tooling.)

### Further notes:

* The rewriting is probably clearer in languages with named function parameters, but that that isn't necessary.

## Comparison to existing concepts

The name "Angel object" is used as a reference to "God objects" and to "messengers". How does this compare to existing language features or patterns?

Notice that:
* Classes are special cases of angel objects. You can implement a class `Foo` in an equivalent way by specifying `@Angel(self: Foo)` or `@Angel(angel_foo: Foo)` on each member function.
* You can generalise the concept of a class by allowing functions to be members of multiple "classes" when the syntax provides a convenient way of f1 calling f2, and providing angel_foo, angel_logging and angel_gobal_settings from f1's context. As if all functions are "members" of the logging class. This can also allow Bar and Baz to be member classes of Foo, and all Bar and Baz functions to receive a Foo pointer as well.
* Test fixtures are a special case of angel objects. But test fixtures need to use a special-purpose syntax, and "magic at a distance" configuration files controlling which parameters are provided.
* Dependency injection frameworks are a special case of angel objects (I think?). Except for the same "magic at a distance" configuration files controlling which parameters are provided.

## Possible Implementations

Rust macro

Python decorator. Rewriting contents? Or adding wrapper to function which rewriting function table to update default.

### Existing Alternatives:

Linking tricks
"god" object with angel objects as members

### Efficient implementations

Not my priority here
Stack vs global vs member variable
Could be optimised if needed
