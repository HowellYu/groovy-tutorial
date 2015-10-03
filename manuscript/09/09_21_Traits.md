# Traits {#chtraits}

I> Traits were introduced in [Groovy 2.3](http://groovy-lang.org/releasenotes/groovy-2.3.html#Groovy2.3releasenotes-Traits)

Where classes can be used to describe real or virtual "things", traits provide a construct for describing an ability or a set of related abilities. Let's consider two contexts in which we might use traits:

* Using a `class` we might describe an `Animal` in terms of its features (`diet`, `distribution`) and then enhance the `Animal` with traits such as `Hopping` (for a wallaby) or `Climbing` (for a koala) or both (for a tree kangaroo).
* A `BankAccount` class would have its properties (`balance`, `accountHolder`) but different accounts provides different abilities (or a combination thereof) such as `Deposit`, `Overdraft` or `CurrencyConversion`.

It's sometimes difficult to determine where a set of methods might instead be better grouped as a trait but the following may help your decision making:

* If the methods describe an ability or feature that is common to a variety of classes they're a good candidate for a trait
    * Even if this is at the abstract level and would need to be more specific for subtypes[^inherit] - a wallaby hops differently to a frog.
* Where the various methods manipulate properties or fields that are otherwise not accessed or only read by the class.

[^inherit]: We'll look into this further in the [chapter on Inheritance](#chinheritance).

Let's take a look at an example trait:

{lang=Java}
<<[A basic trait example](code/09/sport_trait.groovy)

This type of layout is much as we saw for `class` and `interface` definitions:

* The declaration starts with the `trait` keyword followed by the trait's name
* We use the CamelCase format for the trait's name - as we did for classes and interfaces
* Similar to a class, the body of the trait then provides the required properties, fields and methods
    * In the `Running` trait you can see that there's only one method declared: `startEvent()`.

When it comes to use the `trait` in a class, the `implements` key is used: `class SportingEvent implements Running`. The `class` declaration can implement zero or more traits in the same way it can implement zero or more interfaces. In fact, a `class` declaration can implement a mix of traits and interfaces as can be seen in the snippet below:

    class SportingEvent implements SafetyCheck, Running, Swimming {...}

At a guess, the `Running` and `Swimming` elements are traits and the `SafetyCheck` element could be a trait or an interface - we'd need to check the source code or `groovydoc`.

# Trait properties and fields
Traits can also declare properties:

{lang=Java}
<<[A trait with a property](code/09/sport_trait_prop.groovy)

As you can see in the example above, the `Running` trait's `distance` property essentially becomes a property of the `SportingEvent` class. As for classes, getters and setters are generated for properties but you can supply your own if you need additional functionality.

Fields within traits are somewhat different to those in classes. This can get a bit tricky so let's lay out the example code first:

{lang=Java}
<<[A trait with fields](code/09/sport_trait_prop_field.groovy)

In the `Running` trait above you can see:

* One property (as before): `Integer distance`
* A public field: `public String raceType = 'foot race event'`
* A private field: `private Integer maxCompetitors = 10`

Unlike properties, we cannot refer to a field as though it was a member of the `SportingEvent` class. In order to access the public and private fields I need to call `dash.Running__raceType` and `dash.Running__maxCompetitors` respectively. This notation uses the trait's fully qualified name followed by two underscores then the field: `<PackageName>_<TraitName>__<fieldName>` and is needed both in the implementing class and external code.

The fully qualified name format is a bit odd but, as we've looked at packages earlier it should be possible to clarify this:

* For a trait that is in the _default_ package (i.e. not explictly put in a package), we just need `<TraitName>__<fieldName>` - as seen in `Running__raceType`
* For a trait in a package, each package level is separated by an underscore (`_`):
    * If `Running` was declared in the `events` package: `events_Running__raceType`
    * If `Running` was declared in the `events.track` package: `events_track_Running__raceType`
    * and so on...

T> As we saw in the [Access Modifiers chapter](#chaccessmodifiers), Groovy doesn't enforce the `private` modifier in traits.

# Trait methods
Trait-defined methods are much the same as we saw with classes but the `private` access modifier prevents us from calling a trait's private methods. In the example below, the call to the private `SportingEvent.rigWinner` method (`race.rigWinner()`) will work but the call to the `Running` trait's private method (`race.slow()`) will cause a `groovy.lang.MissingMethodException`:

{lang=Java}
<<[Private methods](code/09/sport_private_method.groovy)

# Trait static members
As demonstrated in the example below, traits can have static properties, fields and methods:

{lang=Java}
<<[Traits and static members](code/09/sport_static.groovy)

* Accessing a static property: `SportingEvent.MAX_DISTANCE`
* Accessing a static field: `SportingEvent.Running__MAX_COMPETITORS`
* Accessing a static method: `SportingEvent.describeRules()`

Static member support doesn't appear to be "fully baked" at this time so it's a good idea to keep an eye on the [Groovy documentation](http://docs.groovy-lang.org/latest/html/documentation/#_static_methods_properties_and_fields).

# The class-trait relationship
When a trait is implemented by a class, the relation can be seen as the trait is "folded" into the implementing class. We saw this when we called the `SportingEvents` constructor and could see the `Running` trait's properties. Because of this relationship, traits can refer to `this` to access instance members.

The code below reveals that a trait's class is that of the implementer:

{title="Uncover the mystery",lang=Java}
    trait MyTrait {
        Class whoIsThis() {
            this.class
        }
    }

    class MyClass implements MyTrait {}

    assert new MyClass().whoIsThis() == MyClass

In the next example you can see `this` being used internal to the trait (`this.distance = distance`) and the class (`${this.name}`) but also from the class into the trait (`${this.distance}`):

{lang=Java}
<<[Traits and `this`, again](code/09/sport_this.groovy)

## Self types
That last example could have been rewritten such that the `getAdvert()` method is declared in the trait (rather than the class):

{lang=Java}
<<[Traits and `this`](code/09/sport_this2.groovy)

This works fine as `SportingEvent` has a `name` property but there's nothing enforcing this and you're exposed to the risk of a `groovy.lang.MissingPropertyException` being raised at runtime if the method/property/field can't be found.

The `@groovy.transform.SelfType` annotation is used if a trait needs to be tied to a specific implementing class. The example below demonstrates the `Running` trait annotated with `@SelfType(SportingEvent)`, indicating that the trait should only by implemented by `SportingEvent` (or one of its subtypes):

{lang=Java}
<<[The `@SelfType` annotation](code/09/sport_selftype.groovy)

By setting `@SelfType(SportingEvent)` we can ensure that Groovy will refuse to compile the following attempt:

    class Imposter implements Running {}

Just be mindful with this capability - you want to make sure that you aren't coupling your classes and traits too much. Thankfully, `@SelfType` can also be passed an interface, allowing for a more broadly implemented trait.

# Traits and interfaces
As Groovy's interfaces don't support default implementations it may be tempting to favour traits. This isn't a good idea and you should try to describe interactions within your code and with other developers via an API described in interfaces.

Once you've described your interface, a trait can implement the interface in the same manner as classes do, through the use of the `implements` keyword:

{title="Trait implementing an interface",lang=Java}
    interface Locomotion {
        String getDescription()
    }

    trait Hopping implements Locomotion {
        @Override
        String getDescription() {
            'hop hop hop'
        }
    }

# Implementing multiple traits {#chtraitsmultiple}
As mentioned earlier, a class can implement more than one trait. This is straight-forward if the traits don't intersect in terms of members (properties/fields/methods), as is the case in the example below:

{title="Simple implementation of two traits",lang=Java}
    trait Hopping {
        String hop() { 'I am hopping' }
    }

    trait Climbing {
        String climb() { 'I am climbing' }
    }

    class TreeKangaroo implements Hopping, Climbing {}

    TreeKangaroo lumholtz = new TreeKangaroo()

    println lumholtz.hop()
    println lumholtz.climb()

There's nothing too difficult in the `TreeKangaroo` example but what if the traits had methods with the same name? Let's belabour the sporting example one last time!

I believe that the Triathlon consists of three parts - Running, Swimming, and Riding - and the example below sets up each of these as traits that implement the same interface:

{lang=Java}
<<[Handling trait collision](code/09/triathlon.groovy)

It's easy to see that the `Triathlon` class now has three possibilities when `competition.startEvent()` is called. In this scenario, Groovy will use the trait that was declared last - `Riding`. I've listed my traits in the order I want to run the events but I really do want to run all three events.

Groovy lets me manually determine how the colliding trait members will be treated. First of all, the `Triathlon` class will need to provide its own `void startEvent()` method. Then each trait's `startEvent()` method will need to be called using the `<TraitName>.super.` prefix - e.g. `Running.super.startEvent()`.

In the improved `Triathlon` example below you'll notice that I've decided to implement the `Event` interface as a triathlon is an event consisting of three events:

{title="Manual selection of trait methods",lang=Java}
    class Triathlon implements Event, Running, Swimming, Riding {
        @Override
        void startEvent() {
            Running.super.startEvent()
            Swimming.super.startEvent()
            Riding.super.startEvent()
        }
    }

This will now give me the three-stage event I was after.

Before leaving this topic, there are some things to note about the example:

1. The traits don't have to implement the same interface, there just needs to be a collision in one or more of the trait members
1. I didn't have to override the `startEvent` method - I could have used any name - but that would return to Groovy's default of using the `startEvent` method of the last trait declared
1. Use of the `<TraitName>.super.` prefix doesn't have to occur just in cases of a collision - you may just use it to clarify a section of code.
1. Concepts such as overriding and `super` will be covered more fully in the [chapter on Inheritance](#chinheritance).

# The Shapes demo - Traits

The `Sides` trait is based on the notion that a two-dimensional shape consists of a set of sides (edges). In most cases there'd be at least 3 sides to a 2D shape (circles being the exception with 1 side) and it's possible to determine a shape's perimeter by adding up the lengths of the sides. In the `Sides` trait I wanted to provide classes with the ability to name each side using a single lower-case letter (e.g. `a`, `b`, `c`) and associate the side's length.

Let's take a look at the code for the `Sides` trait and then examine its components.

{lang=Java}
<<[The `Sides` trait](code/09/shapes-demo/src/main/groovy/org/groovy_tutorial/shapes/Sides.groovy)

Reviewing the code you'll see:

* Each side will be added to the `sideMap` with a lower-case letter as the key and the side's length as the value
    * The `SIDE_NAME_PATTERN` provides a very basic pattern to limit the acceptable keys
    * The `getSideMap()` will return a clone[^clone] of `sideMap` - this helps protect the property from changing externally to the trait.
* The `perimeter` field will hold the perimeter of the shape
    * This is calculated via the `getPerimeter()` method (more on this in a moment)
    * Note how the perimeter is calculated only once

Aside from the items listed above, you'll notice two versions of the `propertyMissing` method. This is a special Groovy method that is called when a getter or setter is called on a property that doesn't exist. The `propertyMissing(String name)` is called when code attempts to access (get) a non-existent property and `propertyMissing(String name, value)` is called when an attempt is made to mutate (set) a non-existent property. The getter is reasonably straight-forward as it just checks that the requested property name matches the `SIDE_NAME_PATTERN` and, if so, tries to access the property from `sideMap`.

The setter version of `propertyMissing` is a little more complex and, stepping through the method, we can see:

1. The requested property `name` must match `SIDE_NAME_PATTERN`
2. If the `perimeter` has already been calculated we throw an exception as `sideMap` is locked down once `perimeter` has been set
3. The `value` for the side (it's length) must be a `Number`
4. A utility method `ShapeUtil.checkSidesException` is called to ensure that `value > 0` as we don't want negative- or zero-length sides
5. Once all of those preconditions are met the property can be set

All of this results in the `Sides` trait providing implementing classes with not only the ability to store a list of sides and calculate the perimeter but also lets them use a nice letter-based notation for the sides.

Both the `Triangle` and `Rectangle` classes implement the `Sides` trait as well as the `TwoDimensionalShape` interface. By implementing `Sides`, these classes are provided with an implementation of the `getPerimeter()` method required by the `TwoDimensionalShape` interface.

We can see the interaction between the a shape class and the `Sides` trait by examining the `Rectangle` class:

{lang=Java}
<<[The `Rectangle` class](code/09/shapes-demo/src/main/groovy/org/groovy_tutorial/shapes/Rectangle.groovy)

Most of `Rectangle`'s use of the trait is seen in the constructor as we set the sides of the rectangle though a really easy-to-understand notation:

    a = length
    b = width
    c = length
    d = width

The use of the `Sides` trait means that instances of `Rectangle` can use notation such as `myRectangle.a`.

The `Rectangle` constructor also calls `this.perimeter` so as to calculate the perimeter - not because we specifically need it in the constructor but because it locks down the set of sides for the rectangle instance.

I> The `Circle` class could have implemented the `Sides` trait but I left this out so as to specifically demonstrate a `class` implementing an `interface`.

[^clone]: Cloning was mentioned briefly [in the last section](#secclone)
