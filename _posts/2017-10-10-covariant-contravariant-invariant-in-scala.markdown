---
layout: [post, presentation]
title:  "CoVariant, ContraVariant and InVariant... Variances in Scala"
date:   2017-10-10 00:00:00 +0200
reveal:
  theme: ../../../css/theme/whiteish
  center: true
  history: true
tags:
- scala
- types
- covariant
- contravariant
- invariant
comments: true
---

## Variance

The variance in Scala aims to provide flexibility to the inheritance on parametric types.

Two examples where it helps are:

- Make `List[Dog]` a subtype of `List[Animal]`
- Reuse a function `Animal=>Boolean` as a function `Dog=>Boolean`

<!--slide-next-->

<!--more-->

### A concrete example

Consider the following example:

<img src='https://g.gravizo.com/svg?
@startuml;
skinparam monochrome false;
caption Figure 1. Example types;
scale max 900 width;
Animal <|-- Dog;
Animal <|-- Cat;
@enduml;
'>

```
// Our pet classes
sealed class Animal
class Dog extends Animal
class Cat extends Animal
```

<!--slide-down-->

Let's say you work for a veterinary. You're writing an API.

You want to **modularize the functions
that retrieve pets' information from a DB**, like:

- `getName`
- `getBreed`
- etc.

<!--slide-down-->

Then, we could define a class `Func` that will encapsulate an
_information retriever_ function, `f`.

Here is our first attempt:

```
class Func[I,O] (val f: I => O) {
  def apply(i: I): O = f(i)
}
```

<!--slide-down-->

Good! We can define our first instance of `Func`:

```
// Our first information retriever
// Is my animal a dog?
val isADog: Func[Animal, Boolean] = {
  new Func((i: Animal) => i.isInstanceOf[Dog])
}
```

<!--slide-next-->

We say that **_Fun_ is invariant in `I` (`Animal`) and invariant in `O` (`Boolean`)**, as there is not subtype association
done by the compiler.

<!--slide-next-->

## Generalizing `Func`: Covariance

<!--slide-down-->

### The problem

Let's say you handle many `Func` implementations:

```
val x:  Func[Animal, Boolean] = ...
val y:  Func[Animal, String] = ...
val z:  Func[Animal, Int] = ...
```

<!--slide-down-->

It would be good to be able to treat all `x`, `y` and `z` polymorphically.

For instance be able to do:

```
val l: List[Func[Animal, AnyVal]] =
   List(x, y, z) // won't work, invariance
```

Our initial declaration of `Func[I, O]` was invariant in both `I` and `O`.
It **does not allow this supertype relation**.

<!--slide-down-->

### The solution

The solution is **covariance**.

The principle: making `Clz` covariant in `A` means that

- `Cat <: Animal` implies
- `Clz[Cat] <: Clz[Animal]`

In other words: the inheritance of this **parametric** type follows the one from the **parameter** type.

<!--slide-down-->

Back to our example, we simply re-define `Func`, but making it covariant in `O`:

```
// now covariant on O
class Func[I, +O] (val f: I => O) {
  def apply(i: I): O = f(i)
}
```

<!--slide-down-->

```
// specific type
val isADog: Func[Animal, Boolean] = ...

// generic type
val covIsDogForAnyVal: Func[Animal, AnyVal] =
        isADog   // assigned to a more general type
                 // works because
                 // Boolean <: AnyVal,
                 // and thanks to covariance
                 // Func[X, Boolean] <: Func[X, AnyVal]

```

<!--slide-next-->

## Specializing `Func`: Contravariance

<!--slide-down-->

### The problem

Let's say we have our function `Func[Animal, Boolean]`. Given that `Dog <: Animal` (`Dog` is a subtype of `Animal`),
it seems natural to be able to apply such function to a `Dog` too.

<!--slide-down-->

### The solution

The solution is **contravariance**.

<!--slide-down-->

The principle: making `Clz` contravariant in `A` means that:

- `Cat <: Animal` implies
- `Clz[Cat] >: Clz[Animal]`

In other words the inheritance of this parametric type follows
inversely the one from the parameter type.

<!--slide-down-->

We simply redefine `Func` but making it contravariant in `I` this time:

```
// now contravariant in I
class Func[-I, O] (val f: I => O) {
  def apply(i: I): O = f(i)
}
```

<!--slide-down-->

```
// generic type
val isDog: Func[Animal, Boolean] =
        new Func((i: Animal) => i.isInstanceOf[Dog])

// specific type
val contrvarIsDog: Func[Dog, Boolean] =
        isDog // assigned to a more specific type
              // works because
              // Dog <: Animal, and thanks
              // to contravariance
              // Func[Dog, X] >: Func[Animal, X]

```

<!--slide-next-->

## Use in the Scala library

I invite you to take a look at the implementation of [the trait Function2 in Scala v2.12](https://github.com/scala/scala/blob/v2.12.3/src/library/scala/Function2.scala).

See its declaration (ignore the `@specialized` annotation):

```
trait Function2[-T1, -T2, +R] extends ...
```

What are the consequences of using covariance and contravariance?

<!--slide-down-->

For instance for the function:

```
val f: Function2[Animal, Cat, Dog] = ...
```

Which of these casts are illegal?

```
val f1: Function2[Animal, Cat, Animal] = f
val f2: Function2[Dog, Cat, Dog] = f
val f3: Function2[Cat, Cat, Dog] = f
val f4: Function2[Animal, Cat, Animal] = f
val f5: Function2[Animal, Cat, Cat] = f
val f6: Function2[Animal, Dog, Dog] = f

```

<!--slide-next-->

## Summary

This is the result of applying variances:

<img src='https://g.gravizo.com/svg?
@startuml;
skinparam monochrome false;
caption Figure 2. Covariance and Contravariance;
scale max 900 width;
"List[Animal]" <|-- "List[Dog]": "Covariance [+A]";
"List[Animal]" <|-- "List[Cat]";
"Funct[Dog]" <|-- "Funct[Animal]": "Contravariance [-A]";
note left of "Funct[Animal]": When instanciated,\ncan be reused\nas Funct[Dog]\nthanks to\ncontravariance;
note right of "List[Dog]": Can be added\nto a List[Animal]\nthanks to\ncovariance;
@enduml;
'>

<!--slide-next-->

## References

See the [official Scala documentation on variance](https://docs.scala-lang.org/tour/variances.html).

<!--slide-next-->

# Enjoy!
