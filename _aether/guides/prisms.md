---
title: Prisms
section: guides
sort: 3
---

In the previous guide, introducing [Lenses][lenses] and their basic motivations, we saw ways of working with very predictable data structures. All of the types were guaranteed to be present and correct, and our composition of lenses to focus deep in to a data structure was sure to succeed.

In the real world though, it's possible to have data structures which aren't quite so simple to work with.

Consider the following pair of types:

{% highlight fsharp %}
type RecordA =
    { B: RecordB option }

 and RecordB =
    { Value: string }
{% endhighlight %}

They're quite familiar from our exploration in [Lenses][lenses], except for one detail -- the property of B in RecordA is an *option* of RecordB.

## Lens Problems

Let's see what's wrong with just using simple lenses now, by adding lenses to our types.

{% highlight fsharp %}
type RecordA =
    { B: RecordB option }

    static member B_ =
        (fun a -> a.B), (fun b a -> { a with B = b })

 and RecordB =
    { Value: string }

    static member Value_ =
        (fun b -> b.Value), (fun value b -> { b with Value = value })
{% endhighlight %}

We've got lenses that look correct, but what happens when we try and compose them to get from RecordA to Value?

{% highlight fsharp %}
// Fails to type check!
let avalue_ =
    RecordA.B_ >-> RecordB.Value_
{% endhighlight %}

We've got a lens from RecordA to a RecordB option, but a lens from a RecordB to a string -- the types won't match up. This is clearly not going to work as it is.

## Enter Prisms

Prisms work with cases like this. A simple way to put it is that a prism is like a lens, but to something which *is* or *is not* there. In this case, because the type is optional, but we'll some some other common cases shortly.

So what does a prism look like? Well, here are the types of Lens and Prism:

{% highlight fsharp %}
/// Lens from 'a -> 'b.
type Lens<'a,'b> =
    ('a -> 'b) * ('b -> 'a -> 'a)

/// Prism from 'a -> 'b.
type Prism<'a,'b> =
    ('a -> 'b option) * ('b -> 'a -> 'a)
{% endhighlight %}
    
They're very similar indeed -- except that the getter in the prism returns an option of 'a, rather than a simple 'a. Now we can write logic which will compose prisms, which is slightly different to composing lenses.

Here are our types again, now with a prism provided instead of a lens on RecordA, and type signatures:

{% highlight fsharp %}
type RecordA =
    { B: RecordB option }

    // Prism<RecordA,RecordB>
    static member B_ =
        (fun a -> a.B), (fun b a -> { a with B = Some b })

 and RecordB =
    { Value: string }

    // Lens<RecordB,string>
    static member Value_ =
        (fun b -> b.Value), (fun value b -> { b with Value = value })
{% endhighlight %}

We can see that we've modified B_ to return the option value in the getter, but to take a normal value in the setter, so we conform to the type of Prism.

Now here's our composition again, with the type signature:

{% highlight fsharp %}
// Prism<RecordA,string>
let avalue_ =
    RecordA.B_ >?> RecordB.Value_
{% endhighlight %}

We've composed a prism (on the left) with a lens, and returned a new prism.

__Note:__ We've used a new operator here for composition. The >?> operator is the equivalent to the >-> operator, but works when composing a prism on the left hand side, instead of a lens.

Now we can start to get an intuition for how this new prism will behave.

{% highlight fsharp %}
// Returns Some "Hello World!"
let a1value =
    Optic.get avalue_ a1

// Returns None
let a2value =
    Optic.get avalue_ a2

// Sets the Value to Goodbye World!
let a11 =
    Optic.set avalue_ "Goodbye World!" a1

// Value is not set, as B is None
let a22 =
    Optic.set avalue_ "Goodbye World!" a2
{% endhighlight %}

We can see that we actually gain something here -- a quick and type checked way to work with more complex data structures, where we may not have such predictable data. The behaviour is quite straightforward, and composition still works, even though we're using lenses and prisms.

__NOTE:__ From this point on we are likely to use the term "optics" to collectively refer to either lenses or prisms.

## Examples

We've seen that a prism is a good match for the obvious example -- where the value is literally an option. Other less obvious matches are all around though, especially in languages like F#. Here are a few places (and this is certainly non-exhaustive) where prisms are the natural answer.

### Union Types

Prisms are applicable to all union types (Option is a union type of course, with cases Some and None). How we would we write prisms for a custom union type of our own?

{% highlight fsharp %}
type MyUnion =
    | First of int
    | Second of string

    // Prism<MyUnion,int>
    static member First_ =
        (fun m ->
            match m with 
            | First i -> Some i
            | _ -> None),
        (fun i m ->
            match m with
            | First _ -> First i
            | m -> m)
{% endhighlight %}

Here we've written a prism to the value of MyUnion when the case is First. Our getter is simple, converting the case of First to Some and any other case to None. Our setter however is more complicated. We only set the value of a the First case if the Union was the First case to start with -- otherwise we are structurally altering our MyUnion instance, and not just the *content* of the MyUnion instance.

__NOTE:__ This is what we would consider a *well-behaved* prism. For more on the behaviour expected of both lenses and prisms, see the guide to [Laws][laws]. The prism we saw in our first RecordA example was not well-behaved, but was written simply to help first understanding. Here is the well-behaved version of that prism, such that the outer option type is not changeable through the use of the prism:

{% highlight fsharp %}
type RecordA =
    { B: RecordB option }

    // Prism<RecordA,RecordB>
    static member B_ =
        (fun a -> a.B),
        (fun b a ->
            match a with
            | a when a.B.IsSome -> { a with B = Some b }
            | a -> a)
{% endhighlight %}

Our previously intuited behaviour remains the same however -- correctly.

### Lists

Working with lists is also an excellent candidate for prisms. The elements we consider to be part of a list (head and tail) are either there or not, and thus can be defined through prisms. An item at an indexed position within a list also is either present or absent, so can also be expressed as a prism.

Here are some appropriate (generic) prisms for list elements. (These are provided as part of Aether -- see the [Reference][reference] section for provided lenses, prisms, etc.)

{% highlight fsharp %}
// Prism<'a list,'a>
List.head_

// Prism<'a list,'a list>
List.tail_

// int -> Prism<'a list,'a>
List.pos_
{% endhighlight %}

## Further

We've seen that prisms can be a very useful approach to dealing with uncertainty in data structures, and the basics of writing our own prisms. There is of course still more to Aether, but we now have, in the combination of lenses and prisms, some powerful tools with which to work on data. These tools are general and predictable, and hopefully readily accessible when using Aether.

Next steps: [Morphisms][morphisms].

Relevant reference: [Aether][aether] and [Aether.Operators][aether.operators].

<!--- Local --->

[lenses]: /aether/guides/lenses.html
[morphisms]: /aether/guides/morphisms.html
[laws]: /aether/guides/laws.html
[reference]: /aether/reference
[aether]: /aether/reference/aether.html
[aether.operators]: /aether/reference/aether.operators.html
