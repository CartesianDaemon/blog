# Angel Objects

## Motivating problem

Sometimes one object is needed by many functions in the call stack and you contemplate adding an identical parameter to each function and call site in a hierarchy.

Examples which I've come across include:
* Almost all functions in the codebase need access to a logging object.
* An build script has some flags which apply throughout most of its implementation, e.g. "dry-run".
* Different parts of a program use different strategies or pools from a custom memory allocator.
* A game uses system functions for current time, but wants to wrap them so that not-real-time gameplay can be used for replays and testing.
* GUI widget ...
* You want to split out some of the functionality of your game Map object into an Item object, a score object, etc, but some of the functionality of the Map object still needs to be equally available to all the member objects.

In particular the version of the object used should be determined by the call site:
* The program might call a function and want to capture its logging output.
* Each components calls common utility functions, but needs them to use the memory allocation settings for this component.
* You might have more than one Map object (e.g. moving between multiple rooms, multi-player, pre-loading the next level, running tests in parallel)

The usual compromises are to make it global and avoid the cases which don't work with a single global value. Or to pass it as an argument in a small enough number of functions its not too onerous.

I think the ideal requirements would be:
* It's possible for the call site to specify what value should be used for the parameter.
* But it's easy for most functions to pass along the same parameter they received.
* It's easy to see which functions _don't_ need to use this parameter.
* It's easy for a function to use multiple such parameters.
* It's trivial to add or remove a function's dependency on one of these parameters, without editing a lot of boilerplate.

I think the "right" design that fulfills these is to pass the object as a function parameter, if only we could avoid the amount of boilerplate that needs. I think most people get used to avoiding noticing when it would be useful to treat a parameter this way because they're used to having to use one of the compromises.

## Proposed syntax 

The name "Angel object" is used as a reference to "God objects" and to "messengers". It proposes a syntax which is convenient to read and edit, which is syntactic sugar for providing the necessary arguments on the call stack.

An `@Angel(bazflags: Bazflags)` decoration on a function makes the following changes:
1. Adds a `bazflag: Bazflag` argument to the end of the argument list.
2. Rewrites any function calls in the function body to add a bazflag to the argument list if it's not already present, if that function also had the `@Angel(bazflags: Bazflags)` annotation.

An `@Angel(Bazflags bazflags)` decoration on a class declaration or implementation makes the following changes:
1. Adds an `bazflag` member of type `Bazflag` to the class. Constructors are treated as having the same decorator, and automatically initialise the `bazflag` class member with the additional `bazflag` parameter.
2. Class member functions are rewritten so that calls to functions with the decorator are rewritten to add the `bazflags` argument if it's not already present, with a value of the `bazflag` class member variable.

Note:
* It'd probably be good practice to use a conventional prefix on angel arguments so humans can see they're special.
* You can have multiple angel decorator.
* The rewriting is probably clearer in languages with named function parameters, but that that isn't necessary.

TODO Q:
* Most convenient way when there's not a single convenient constructor.
* Can I be more clear or motivating?

## Examples

Floating point settings provided to "complicated_calculation" function..?

## Comparison to existing concepts

Alternatives:

linking tricks
"god" object with angel objects as members

Generalisation of:

Classes
Test fixtures

## Possible Implementations

Rust macro
Python decorator adding wrapper to function which rewriting function table to update default

## Efficiency

Not my priority here
Stack vs global vs member variable
Could be optimised if needed
...

==========================

## Triage use case examples:

Logging class, but the `DatabaseManager("ExcessivelyLargeMeasurementsDatabase")` instance might be initialised with logging settings optimised for speed and the `DatabaseManager("WorryinglyFlakyUserDatabase") instance might be initialised with logging settings optimised for clarity.
A GUI widget provides some global drawing settings to panes, tabs, buttons, etc inside it. But some panes override the settings for the buttons inside them. And there's several different sets of settings, and each component usually needs some of them but not others.

## Triage Syntax Example

```
fn process(Str filename)
{
  ...
  foo(arg1, arg2, arg3, angel_bazflags=BazFlags(...))
  ...
}

@Angel(Bazflags bazflags)
fn foo(Arg1 arg1, Arg2 arg2, Arg3 arg3)
{
  ...
  bar(a, b)
  ...
}

@Angel(Bazflags bazflags)
fn bar(ArgA a, ArgB b)
{
  ...
  loop(x=...) {
    bar(x, b)
  }
  ...
}

@Angel(Bazflags bazflags)
fn baz(ArgX x, ArgY y)
{
  // Do something that needs x, y and angel_bazflags
  ...
  ...
}

```
