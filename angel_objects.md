# Angel Objects

## Premise

A common experience in programming is needing to add an argument to a function deep in a call stack, and realising that a lot of the functions above it in the call stack need to add the same argument just to pass it on the next function. This is inconvenient to do, and inconvenient to change if the argument changes. There are common ways of simplifying this, e.g. combining several arguments which need to be threaded through the call stack into a single struct, but I think that sometimes those are the correct design, and sometimes they're useful to reduce boilerplate but at the cost of using a less correct design.

Another common version is that there's an object which almost all functions in the code base need, e.g. an instance of a logging object, or global configuration for the program or for tests created with a test framework. These are usually provided by a global variable or an equivalent construct like a singleton or test fixture, but usually a design which passed them as arguments would be more correct in every way except for needing the inconvenient boilerplate. The global is sometimes called a god object, although the real objection to god objects are to objects which have a lot of different functionality not only a lot of state.

This post will describe a pattern which I think fulfills those needs with less boilerplate, has less downsides than existing alternatives, and might have other uses as well.

### Examples

Logging class available in almost all functions.
Logging class, but the `DatabaseManager("ExcessivelyLargeMeasurementsDatabase")` instance might be initialised with logging settings optimised for speed and the `DatabaseManager("WorryinglyFlakyUserDatabase") instance might be initialised with logging settings optimised for clarity.
Computer game which has a map object which contains many hero, obstacle, monster and effects classes, but those classes are effectively always members of a map object and need to access some set of map object members.
A GUI widget provides some global drawing settings to panes, tabs, buttons, etc inside it. But some panes override the settings for the buttons inside them. And there's several different sets of settings, and each component usually needs some of them but not others.
Some code that communicates across a network calls time functions or file-access functions. You provide those through an abstraction so they can be replaced with a mock for testing. For testing you want to test the communication between "the code, instantiated with the fake time and fake filesystem of computer A" and "the code, instantiated with the fake time and fake filesystem of computer B".

Geenral vibe. A class collects together variables needed by a string of functions in the call stack which are all associated with a single class. A global provides variables to all functions in the call stack. But some variables naturally belong to a unit larger than a class but smaller than a program, like to one module of the code base (but without requiring that it is instantiated as a singleton), or to one class that contains a lot of interacting classes.

## Syntax

### Example

Old way. This uses pseudocode which can use named arguments:

```
fn process(Str filename)
{
  ...
  foo(arg1, arg2, arg3, angel_bazflags=BazFlags(...))
  ...
}

fn foo(Arg1 arg1, Arg2 arg2, Arg3 arg3, BazFlags angel_bazflags)
{
  ...
  bar(a, b, angel_bazflags=angel_bazflags)
  ...
}

fn bar(ArgA a, ArgB b, BazFlags angel_bazflags)
{
  ...
  loop(x=...) {
    bar(x, b, angel_bazflags=angel_bazflags)
  }
  ...
}

fn baz(ArgX x, ArgY y, BazFlags angel_bazflags)
{
  // Do something that needs x, y and bazflags
  ...
  ...
}

```

I propose a pattern which is syntactic sugar for the same thing. In pseudocode:

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

### Specification

For functions:
1. An `@Angel(Bazflags bazflags)` decoration on a function adds an `angel_bazflag` argument of type `Bazflag` to the end of the argument list. The prefix to the argument is added so that humans can clearly see which arguments came from angel magic.
2. The decoration also finds any function calls in the function body which call a function with an angel decoration for the same variable name, and and adds the variable as another parameter if the parameter was not provided.

Note that you can have multiple angels. The syntax might be @Angel(Type1 variable1, Type2 variable2), or might simply be too separate decorations. Note that the rewriting is probably clearer in languages with named function parameters, but that that isn't necessary.

A similar construct can be done for classes:
1. An `@Angel(Bazflags bazflags)` decoration on a class adds an `angel_bazflag` member of type `Bazflag` to the class, and initialises it with a similar parmater added to the constructor.
2. All class member functions are rewritten so that calls to function calls (including constructors) which expect an `angel_bazflags` parameter but aren't are given one are given the `angel_bazflags` class member variable.

## Comparison to Existing Language Features and Patterns

Globals.
Fixtures
Dependency injection frameworks.
Classes.
...?

And:

I've tried to give illistrative examples of where this would be useful. That's because I think that doing the right thing in those cases illustrates that there are many cases like that where people make do with the most common existing patterns, but that it would be a lot easier to work with, that you will discover if you had the freedom to write them right rather than being constrainted by "Well, my language only supports classes or globals so if there's any cases where those work less well I just pretend they don't exist".

## Passive-aggressive FAQ



