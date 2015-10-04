-# More object-oriented programming

# Introduction

Now that you have a handle on constructing classes we'll explore the various Groovy constructs for creating a more
extensible codebase:

* Interfaces: define a set of method signatures that are then implemented by a class (or classes).
* Traits: add abilities to a class
* Inheritance: Allows a class to inherit the functionality of another class and extend it.
    * When `ChildClass` inherits from `ParentClass`, we say that `ChildClass` is a _subclass_ of `ParentClass` and that `ParentClass` is a _superclass_ of `ChildClass`[^wp].

First up, we'll look at how to organise your code before those classes get out of hand.

[^wp]: See: [Wikipedia: Inheritance (object-oriented programming)](https://en.wikipedia.org/wiki/Inheritance_%28object-oriented_programming%29#Subclasses_and_superclasses)

## The Shapes demo
Throughout this section I'll build up an example library for working with two-dimensional shapes. You'll see this in chapters with the __The shapes demo__ prefix and a full listing is available [as an appendix](#chshapesdemo).

As the Shapes demo source code is laid out as a larger project and divided into packages, you won't be able to run it
via `groovyConsole`.
To support you in trying out the demo I've setup the [shapes demo](http://www.groovy-tutorial.org/shapes-demo/)
mini site. This provides a number of handy resources:

* A guide to building (compiling) the code both directly using `groovyc` and with the [Gradle build tool](http://gradle.org/).
* Links to download the code
* Various reports on the code

Once you've read through the chapters in this section, head to the [shapes demo](http://www.groovy-tutorial.org/shapes-demo/)
 mini site and give it a try.
