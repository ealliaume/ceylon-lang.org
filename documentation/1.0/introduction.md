---
layout: documentation
title: Quick Introduction to Ceylon
tab: documentation
unique_id: docspage
author: Gavin King
doc_root: ..
---

# Quick introduction

It's impossible to get to the essence of a programming language by looking
at a list of its features. What really *makes* the language is how all the
little bits work together. And that's impossible to appreciate without 
actually writing code. In this section we're going to try to quickly show 
you enough of Ceylon to get you interested enough to actually try it out.
This is *not* a comprehensive feature list!

## A familiar, readable syntax

Ceylon's syntax is ultimately derived from C. So if you're a C, Java, or C#
programmer, you'll immediately feel right at home. Indeed, one of the goals
of the language is for most code to be immediately readable to people who
*aren't* Ceylon programmers, and who *haven't* studied the syntax of the
language.

Here's what a simple function looks like:

<!-- implicit-id:point: class Point() { shared Float x = 0.0; shared Float y = 0.0; } -->
<!-- cat-id:point -->
<!-- cat: void m() { -->
    function distance(Point from, Point to) {
        return ((from.x-to.x)**2 + (from.y-to.y)**2)**0.5;
    }
<!-- cat: } -->

Here's a simple class:

    shared class Counter(Integer initialValue=0) {
        
        variable value count := initialValue;
        
        shared Integer currentValue {
            return count;
        }
        
        shared void increment() {
            count++;
        }
        
    }

Here's how we create and iterate sequences:

<!-- cat: void m() { -->
    String[] names = { "Tom", "Dick", "Harry" };
    for (name in names) {
        print("Hello, " name "!");
    }
<!-- cat: } -->

If these code examples look boring to you, well, that's kinda the idea -
they're boring because you understood them immediately!

## Declarative syntax for treelike structures

Hierarchical structures are so common in computing that we have dedicated
languages like XML for dealing with them. But when we want to have procedural
code that interacts with hierarchical structures, the "impedence mismatch"
between XML and our programming language causes all sorts of problems. So
Ceylon has a special built-in "declarative" syntax for defining hierarchical 
structures. This is especially useful for creating user interfaces:

<!-- check:none -->
    Table table {
        title="Squares";
        rows=5;
        Border border {
            padding=2;
            weight=1;
        }
        Column {
            heading="x";
            width=10;
            String content(Integer row) {
                return row.string;
            }
        },
        Column {
            heading="x**2";
            width=10;
            String content(Integer row) {
                return (row**2).string;
            }
        }
    }

But it's much more generally useful, forming a great foundation for 
expressing everything from build scripts to test suites:

<!-- check:none -->
    Suite tests {
        Test { 
            name = "sqrt() function";
            void run() {
                assert(sqrt(1)==1);
                assert(sqrt(4)==2);
                assert(sort(9)==3);
            }
        },
        Test {
            name = "sqr() function";
            void run() {
                assert(sqr(1)==1);
                assert(sqr(2)==4);
                assert(sqr(3)==9);
            }
        }
    }

Any framework that combines Java and XML requires special purpose-built 
tooling to achieve type-checking and authoring assistance. Ceylon frameworks
that make use of Ceylon's built-in support for expressing treelike structures
get this, and more, for free.

## Principal typing, union types, and intersection types

Ceylon's conventional-looking syntax hides a powerful type system that is 
able to express things that other static type systems simply can't. All
types in Ceylon can, at least in principle, be expressed within the type
system itself. There's no primitive types, arrays, or anything similar. 

The type system is based on analysis of "best" or *principal* types. For 
every expression, a unique, most specific type may be determined, without 
the need to analyze the rest of the expression in which it appears. And all 
types used internally by the compiler are *denotable* - that is, they can be 
expressed within the language itself. What this means in practice is that 
the compiler always produces errors that humans can understand, even when 
working with complex generic types. The Ceylon compiler *never* produces 
error messages with mystifying non-denotable types like Java's `List<capture#3-of ?>`.

An integral part of this system of denotable principal types is first-class
support for union and intersection types. A *union type* is a type which
accepts instances of any one of a list of types:

<!-- check:none -->
    Person|Organization personOrOrganization = ... ;

An *intersection type* is a type which accepts instances of all of a list
of types:

<!-- check:none -->
    Printable&Sized&Persistent printableSizedPersistent = ... ;

Union and intersection types are occasionally useful as a convenience in 
ordinary code. More importantly, they help make things that are complex 
and magical in other languages (especially generic type argument inference) 
simple and straightforward in Ceylon. For example, consider the following 
sequences:

<!-- check:none -->
    value stuff = { "hello", "world", 1.0, -1 };
    value joinedStuff = join({"hello", "world"}, {1.0, 2.0}, {});

The compiler automatically infers the types: 

* `Sequence<String|Float|Integer>` for `stuff`, and 
* `Empty|Sequence<String|Float>` for `joinedStuff`. 

These are the correct principal types of the expressions. We didn't need to 
explictly specify any types anywhere.

We've worked hard to keep the type system [quite simple at its core][manifesto]. 
This makes the language easier to learn, and helps control the number of 
buggy or unintuitive corner cases. And a highly regular type system also 
makes it easier to write generic code.

[manifesto]: /blog/2012/01/18/type-system-manifesto

## Mixin inheritance

Like Java, Ceylon has classes and interfaces. A class may inherit a single
superclass, and an arbitrary number of interfaces. An interface may inherit
an arbitrary number of other interfaces, but may not extend a class other
than `Object`. Unlike Java, interfaces may define concrete members. Thus,
Ceylon supports a restricted kind of multiple inheritance, called *mixin
inheritance*. 

<!-- check:none -->
    interface Sized {
        
        shared formal Integer size;
    
        shared Boolean empty {
            return size==0;
        }
    
    }
    
    interface Printable {
    
        shared void printIt() {
            print(this);
        }
        
    }
    
    object empty satisfies Sized & Printable {
    
        shared actual Integer size {
            return 0;
        }
        
    }

What really distinguished interfaces from classes in Ceylon is that 
interfaces are *stateless*. That is, an interface may not directly hold
a reference to another object, it may not have initialization logic, and
it may not be directly instantiated. Ceylon neatly avoids the need to
perform any kind of "linearization" of supertypes.

## Polymorphic attributes

Ceylon doesn't have fields, at least not in the traditional sense.
Instead, *attributes* are polymorphic, and may be refined by a subclass, 
just like methods in other object-oriented languages. 

An attribute might be a simple value:

<!-- cat: String firstName = "John"; -->
<!-- cat: String lastName = "Doe"; -->
    String name = firstName + " " + lastName;

It might be a getter:

<!-- cat: String firstName = "John"; -->
<!-- cat: String lastName = "Doe"; -->
    String name {
        return firstName + " " + lastName;
    }

Or it might be a getter/setter pair:

<!-- cat: variable String fullName := "John Doe"; -->
    String name {
        return fullName;
    }
    
    assign name {
        fullName := name;
    }

In Ceylon, we don't need to write trivial getters or setters, since the
state of a class is always completely abstracted from clients of the 
class.

## Typesafe null and safer type narrowing

There's no `NullPointerException` in Ceylon, nor anything similar. Ceylon
requires us to be explicit when we declare a value that might be null, or
a method that might return null. For example, if `name` might be null, 
we must declare it like this:

<!-- check:none -->
    String? name = ...

Which is actually just an abbreviation for:

<!-- check:none -->
    String|Nothing name = ...

An attribute of type `String?` might refer to an actual instance of `String`, 
or it might refer to the value `null` (the only instance of the class `Nothing`). 
So Ceylon won't let us do anything useful with a value of type `String?` 
without first checking that it isn't null using the special `if (exists ...)` 
construct.

    void hello(String? name) {
        if (exists name) {
            print("Hello, " name "!");
        }
        else {
            print("Hello, world!");
        }
    }

Similarly, there's no `ClassCastException` in Ceylon. Instead, the 
`if (is ...)` and `case (is ...)` constructs test and narrow the type of
a value in a single step. Indeed, the code above is really just a shortcut
way of writing the following: 

    void hello(String? name) {
        if (is String name) {
            print("Hello, " name "!");
        }
        else {
            print("Hello, world!");
        }
    }

## Enumerated subtypes

In object-oriented programming, it's usually considered bad practice to write 
long `switch` statements that handle all subtypes of a type. It makes the code 
non-extensible. Adding a new subtype to the system causes the `switch` 
statements to break. So in object-oriented code, we usually try to refactor 
constructs like this to use an abstract method of the supertype that is 
refined as appropriate by subtypes.

However, there is a class of problems where this kind of refactoring isn't 
appropriate. In most object-oriented languages, these problems are usually 
solved using the "visitor" pattern. Unfortunately, a visitor class actually 
winds up more verbose than a `switch`, and no more extensible. There is, on
the other hand, one major advantage of the visitor pattern: the compiler 
produces an error if we add a new subtype and forget to handle it in one
of our visitors.

Ceylon gives us the best of both worlds. We can specify an *enumerated list
of subtypes* when we define a supertype:

<!-- check:none -->
    abstract class Node() of Leaf | Branch {}

And we can write a `switch` statement that handles all the enumerated subtypes:

<!-- check:none -->
    Node node = ... ;
    switch (node)
    case (is Leaf) { ... }
    case (is Branch) { .... }

Now, if we add a new subtype of `Node`, we must add the new the subtype to the
`of` clause of the declaration of `Node`, and the compiler will produce an error
at every `switch` statement which doesn't handle the new subtype.

## Type aliases and type inference

Fully-explicit type declarations very often make difficult code much easier to
understand. But there are other occasions where the repetition of a verbose
generic type can detract from the readability of the code. We've observed that:

1. explicit type annotations are of much less value for local declarations, and
2. repetition of a parameterized type with the same type arguments is common
   and extremely noisy in Java.

Ceylon addresses the first problem by allowing type inference for local 
declarations. For example:

<!-- check:none -->
    value names = LinkedList { "Tom", "Dick", "Harry" };

    function sqrt(Float x) { return x**0.5; }
    
    for (item in order.items) { ... }

On the other hand, for declarations which are accessible outside the compilation 
unit in which they are defined, Ceylon requires an explicit type annotation. We 
think this makes the code more readable, not less, and it makes the compiler more 
efficient and less vulnerable to stack overflows.

Ceylon addresses the second problem via *type aliases*, which are very similar
to a `typedef` in C. A type alias can act as an abbreviation for a generic type
together with its type arguments:

    interface Strings = List<String>;

We encourage the use of both these language features where - *and only where* -
they make the code more readable.

## Higher-order functions

Like most programming languages, Ceylon lets you pass a function to another 
function. 

A function which operates on other functions is called a *higher-order function*. 
For example:

<!-- id:repeat -->
    void repeat(Integer times, 
            void iterate(Integer i)) {
        for (i in 1..times) {
            iterate(i);
        }
    } 

When invoking a higher-order function, we can either pass a reference to a named 
function:

<!-- cat-id:repeat -->
<!-- cat: void m() { -->
    void printSqr(Integer i) {
        print(i**2);
    }
    
    repeat(5, printSqr);
<!-- cat: } -->

Or we can specify the argument function inline, either like this:

<!-- cat-id:repeat -->
<!-- cat: void m() { -->
    repeat(5, (Integer i) print(i**2));
<!-- cat: } -->

Or, using a named argument invocation, like this:

<!-- cat-id:repeat -->
<!-- cat: void m() { -->
    repeat {
        times = 5;
        void iterate(Integer i) {
            print(i**2);
        }
    };
<!-- cat: } -->

It's even possible to pass a member method or attribute reference to a higher
order function:

<!-- check:none -->
    String[] names = ... ;
    String[] uppercaseNames = map(names, String.uppercase);

## Comprehensions

Filtering and transforming streams of values is one of the main things computers
are good at. Therefore, Ceylon provides a special syntax which makes these operations
especially convenient. Anywhere you could provide a list of expressions (a sequence
instantiation, or a "vararg"), Ceylon lets you write a comprehension instead. For
example, the following expression instantiates a sequence of names:

    { for (p in people) p.firstName + " " + p.lastName }

This expression gives us a sequence of adults:

    { for (p in people) if (p.age>=18) p }

This expression produces a `Map` of name to `Person`:

    HashMap { for (p in people) p.firstName + " " + p.lastName -> p }

This expression creates a set of employers:

    HashSet { for (p in people) for (j in p.jobs) j.organization }

Here, we're using a comprehension as a method argument to format and print the names:

    print(", ".join { for (p in people) p.firstName + " " + p.lastName });

## Simplified generics with fully-reified types

Ceylon does not support Java-style wildcard type parameters, raw types, or any 
other kind of existential type. And the Ceylon compiler never even uses any kind 
of "non-denotable" type to reason about the type system. And there's no implicit 
constraints on type arguments. So generics-related error messages are understandable 
to humans.

Instead of wildcard types, Ceylon features *declaration-site variance*. A type 
parameter may be marked as covariant (`out`) or contravariant (`in`) by the class 
or interface that declares the parameter.

<!-- check:none -->
    interface Correspondence<in Key, out Item> { ... }

Ceylon has a more expressive system of generic type constraints with a much cleaner, 
more regular syntax. The syntax for declaring type constraints on a type parameter 
looks very similar to a class or interface declaration.

<!-- check:none -->
    interface Producer<in Input, out Value>
            given Value(Input input) satisfies Container { ... }

Ceylon's type system is fully reified. In particular, generic type arguments are 
reified, eliminating many problems that result from generic type argument erasure 
in Java.

## Operator polymorphism

Ceylon features a rich set of operators, including most of the operators supported 
by C and Java. True operator overloading is not supported. Sorry, you can't define 
the pope operator `<+|:-)` in Ceylon. And you can't redefine `*` to mean something 
that has nothing to do with numeric multiplication. However, each operator predefined 
by the language is defined to act upon a certain class or interface type, allowing 
application of the operator to any class which extends or satisfies that type. We 
call this approach *operator polymorphism*.

For example, the Ceylon language module defines the interface `Summable`.

<!-- check:none -->
    shared interface Summable<Other> of Other
            given Other satisfies Summable<Other> {
        shared formal Other plus(Other that);
    }

And the `+` operation is defined for values which are assignable to `Summable`.
The following expression:

<!-- check:none -->
    x+y

Is merely an abbreviation of:

<!-- check:none -->
    x.plus(y)

Likewise, `<` is defined in terms of the interface `Comparable`, `*` in terms of
the interface `Numeric`, and so on.

## Typesafe metaprogramming and annotations

Ceylon provides sophisticated support for meta-programming, including a typesafe 
metamodel and events. Generic code may invoke members reflectively and intercept 
member invocations. This facility is more powerful, and much more typesafe, than 
reflection in Java.

<!-- check:none -->
    Class<Person,Name> personClass = Person;
    Person gavin = personClass(Name("Gavin", "King"));

Ceylon supports program element annotations, with a streamlined syntax. Indeed,
annotations are even used for language modifiers like `abstract` and `shared` -
which are not keywords in Ceylon - and for embedding API documentation for the 
documentation compiler:

<!-- check:none -->
    doc "The user login action"
    by "Gavin King"
    throws (DatabaseException,
            "if database access fails")
    see (LogoutAction.logout)
    scope (session)
    action { description="Log In"; url="/login"; }
    shared deprecated
    void login(Request request, Response response) {
        ...
    }

Well, that was a bit of an extreme example!

## Modularity

Ceylon features language-level package and module constructs, along with language-level 
access control via the `shared` annotation which can be used to express block-local, 
package-private, module-private, and public visibility for program elements. There's 
no equivalent to Java's `protected`. Dependencies between modules are specified in
the module descriptor, which is written in Ceylon:

<!-- check:none -->
    Module module {
        name='org.jboss.example';
        version='1.0.0';
        
        doc="This module is just a silly example. You'll 
             find some proper modules in the community 
             repository [Ceylon Herd][Herd].
             
             [Herd]: http://modules.ceylon-lang.org
             
             Happy Herding!";
             
        Import {
            name='ceylon.math';
            version='0.3.0';
        },
        Import {
            name='ceylon.file';
            version='0.3.1';
        }
    }

The Ceylon compiler directly produces `.car` module archives in module repositories.
You're never exposed to unpackaged `.class` files.

At runtime, modules are loaded according to a peer-to-peer classloader architecture,
based upon the same module runtime that is used at the very core of JBoss AS 7.

## Take the Tour

We're done with the introduction. Take the [tour of Ceylon](#{page.doc_root}/tour/basics) 
for a full in-depth tutorial.
