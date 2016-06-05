---
title: Aether
section: reference
order: 2
---

Aether is a relatively simple library -- there's nothing inherently complex about the way that optics are implemented in Aether. However this reference will likely make most sense if you have some expectation about the kinds of things you'll see -- if you're not familiar with lenses, prisms, etc. it's probably best to start with the [Guides][guides] to get some background.

We'll use the term "optic" here to refer to a lens or a prism, and "morphism" to refer to an isomorphism or epimorphism.

## Types

The first thing defined as part of Aether is a set of simple types for our optics and morphisms:

{% highlight fsharp %}
// Optics

type Lens<'a,'b> =
    ('a -> 'b) * ('b -> 'a -> 'a)

type Prism<'a,'b> =
    ('a -> 'b option) * ('b -> 'a -> 'a)

// Morphisms

type Isomorphism<'a,'b> =
    ('a -> 'b) * ('b -> 'a)

type Epimorphism<'a,'b> =
    ('a -> 'b option) * ('b -> 'a)
{% endhighlight %}

The meanings behind these should be fairly obvious. Each type is defined as the appropriate pair of functions -- we adopt a simple approach to optics in Aether.

## Composition

The first useful functions provided by Aether are for composition. Earlier versions of Aether had specialised functions for composing each different optic/morphism combination, but Aether has now reduced this to just two functions, under "Compose":

{% highlight fsharp %}
// 'a -> 'b -> 'c
Compose.lens

// 'a -> 'b -> 'c
Compose.prism
{% endhighlight %}

The most obvious thing about these functions is that the type signature, at first glance, is not very useful! This is because these functions are inlined and statically inferred. However, the type system won't let you misuse them in practice.

In essence, the type signatures are as follows, with some artificial Optic and Morphism types showing possibilities:

{% highlight fsharp %}
// Lens<'a,'b> -> [Optic|Morphism]<'b,'c> -> Optic<'a,'c>
Compose.lens

// Prism<'a,'b> -> [Optic|Morphism]<'b,'c> -> Prism<'a,'c>
Compose.prism
{% endhighlight %}

Lenses, depending on which optic or morphism they're composed with, may compose to either a lens or a prism. Prisms always compose to a further prism.

Using these functions we can compose optics of some complexity (although syntactically it is much more succinct to do so using [Aether.Operators][aether.operators]). These new optics can then be used with the functions in the following section.

## Operations

Having optics is no good unless we can use them, and the functions for working with optics to do something useful are defined under "Optic":

{% highlight fsharp %}
// 'a -> 'b -> 'c
Optic.get

// 'a -> 'b -> 'c
Optic.set

// 'a -> 'b -> 'c
Optic.map
{% endhighlight %}

Again, we see some less than helpful type signatures. Here are the effective type signatures, again using artificial types to stand in for possibilities:

{% highlight fsharp %}
// Optic<'a,'b> -> 'a -> ['b|'b option]
Optic.get

// Optic<'a,'b> -> 'b -> 'a -> 'a
Optic.set

// Optic<'a,'b> -> ('b -> 'b) -> 'a -> 'a
Optic.map
{% endhighlight %}

These functions should be fairly obvious in intent; getting, setting and mapping the value at the focus of an optic. These three simple functions give us most of the power of Aether.

## Helpers

Aether has a couple of helpers to support a common scenario of using a morphism as the start of a composition, by turning the morphism in to a lens or prism.

{% highlight fsharp %}
// Isomorphism<'a,'b> -> Lens<'a,'b>
Lens.ofIsomorphism

// Epimorphism<'a,'b> -> Prism<'a,'b>
Prism.ofEpimorphism
{% endhighlight %}

## Provided Optics and Morphisms

Although you'll have to provide your own optics and morphisms for cases involving your own types (see the [Guides][guides] for more on this), Aether does provide various optics and morphisms for some common data structures.

### Basics

Some simple basic lenses for identity and tuples are provided, and don't require qualification for usage.

{% highlight fsharp %}
// Lens<'a,'a>
id_

// Lens<('a * _), 'a>
fst_

// Lens<(_ * 'a), 'a>
snd_
{% endhighlight %}

### Array

A simple isomorphism is provided between arrays and lists, so that list based optics and morphisms can be used with arrays.

{% highlight fsharp %}
// Isomorphism<'a [],'a list>
Array.list_
{% endhighlight %}

### Choice

Prisms are provided for the two cases of the standard Choice type.

{% highlight fsharp %}
// Prism<Choice<'a,_>,'a>
Choice.choice1Of2_

// Prism<Choice<_,'a>,'a>
Choice.choice2Of2_
{% endhighlight %}

### List

Prisms to the head, tail, and indexed element of a list are provided, along with an isomorphism to an array.

{% highlight fsharp %}
// Prism<'a list,'a>
List.head_

// Prism<'a list,'a list>
List.tail_

// int -> Prism<'a list,'a>
List.pos_

// Isomorphism<'a list,'a []>
List.array_
{% endhighlight %}

### Map

Prisms and lenses in to maps are provided, along with isomorphisms between a map and lists/arrays of key/value pairs. These are considered to be weak isomorphisms, as they're not guaranteed to maintain the same shape if, for example, keys are modified in the paired form.

The prism form of map access and the lens form of map access have different goals in usage and are mentioned in the [Guides][guides].

{% highlight fsharp %}
// 'a -> Prism<Map<'a,'b>,'b>
Map.key_

// 'a -> Lens<Map<'a,'b>,'b option>
Map.value_

// Isomorphism<Map<'a,'b>,('a * 'b) []>
Map.array_

// Isomorphism<Map<'a,'b>,('a * 'b) list>
Map.list_
{% endhighlight %}

### Option

A prism to the value (if there is one) of an Option type is provided.

{% highlight fsharp %}
// Prism<'a option,'a>
Option.value_
{% endhighlight %}

## Further Reference

See the reference on [Aether.Operators][aether.operators] for more, as the operator based syntax reduces the amount of code required significantly, particularly for composition.

<!--- Local --->

[guides]: /aether/guides
[aether.operators]: /aether/reference/aether.operators.html
