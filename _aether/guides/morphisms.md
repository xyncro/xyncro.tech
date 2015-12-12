---
title: Morphisms
section: guides
sort: 4
---

We've seen some of the major building blocks that Aether provides in the previous two guides, [Lenses][lenses] and [Prisms][prisms]. Those are very useful, but as it turns out there's another small set of concepts which can add even more power and flexibility, without adding much complexity.

In fact morphisms are, if anything, simpler forms of the lenses and prisms we've already seen. Rather than letting us work with a subset of a data structure, they let us work with the whole data structure -- but in a different shape.

That isn't the most obvious concept without an example, so let's look at a very simple case where we might want to think about using morphisms.

## Isomorphisms

There are two kinds of morphism that we'll use. The first will be an isomorphism -- the distinction between the two will become clear soon.

Let's invent an example type:

{% highlight fsharp %}
type Record =
    { Age: string }

    static member Age_ =
        (fun x -> x.Age), (fun age x -> { x with Age = age })
{% endhighlight %}

A very simple record, with an Age property, and a useful lens to access it. Awkwardly though, the age property is of type string and -- in this fictitious example -- we can't change that. However, in our world, we think of ages as integers, and we know that the age property is actually always numeric -- it's just stored as a string.

So, we could do something like this whenever we wanted to use the age:

{% highlight fsharp %}
let record =
    { Age = "42" }

// Get the Age as an int
let age =
    Int32.Parse (Optic.get Record.Age_ record)

// Set the Age given an int
let record' =
    Optic.set Record.Age_ (string 43) record
{% endhighlight %}

This works, of course, but is going to lead to a lot of duplication of fiddly bits of code. An isomorphism can fix this!

Let's take a look at the type of an isomorphism, and write one for this situation:

{% highlight fsharp %}
// Isomorphism between 'a <> 'b.
type Isomorphism<'a,'b> =
    ('a -> 'b) * ('b -> 'a)

// Isomorphism<string,int>
let stringint_ =
    (fun s -> Int32.Parse s), (fun i -> string i)
{% endhighlight %}

It's very basic - just a pair of functions to convert back and forth from one type to another. It's a lot like a lens -- just simpler!

What can we do with this new isomorphism? Well, as with optics, we can compose it. Here we compose it with the age lens on our record, to give us a new lens from our record type straight to an int:

{% highlight fsharp %}
// Lens<Record,int>
let recordage_ =
    Record.Age_ >-> stringint_
{% endhighlight %}
    
Now we can work with the age through our lens as if it was an int, oblivious to the underlying implementation in the data structure.

__NOTE:__ This can be quite an advantage to working with optics and morphisms in general -- we've abstracted our data access away from the data type. It's perfectly possible to change the underlying data type and the appropriate optic, and yet for users of the optic to be completely insulated from any of that change. Especially when we're working with data types outside of our control (and maybe versioned, or external), this can prove extremely useful in constructing stable systems. 

## Epimorphisms

The second type of morphism we use is an epimorphism. This is also very simple in terms of types, and looks like this:

{% highlight fsharp %}
// Epimorphism between 'a <> 'b.
type Epimorphism<'a option,'b> =
    ('a -> 'b option) * ('b -> 'a)
{% endhighlight %}

As we can see, it's very much like an Isomorphism, except our "map from" function returns an option type. This is to handle those cases where the type _may not be convertible_. In fact if we think about it, our first isomorphism in our example above could perhaps have been interpreted in this way as -- strictly speaking -- not all strings can be parsed as an integer.

In our first case, we were willing to take this risk because we assumed that all strings were parseable, but if that assumption couldn't be made, we could write an epimorphism instead, like this:

{% highlight fsharp %}
let stringint_ =
    (fun s ->
        match Int32.TryParse s with
        | true, i -> Some i
        | _ -> None),
    (fun i -> string i)
{% endhighlight %}

Our composition now looks the same, but the resulting type is different:

{% highlight fsharp %}
// Prism<Record,int>
let recordage_ =
    Record.Age_ >-> stringint_
{% endhighlight %}

Our resulting type is now a prism from a record to an int, as the epimorphism means that we may or may not have a resulting value.

## Composition

As we've seen, morphisms can be very useful at the end of a chain of composition, but they can be even more powerful within the chain. Imagine that we have a morphism from some type to another, where our second type has optics to parts of it. Now we can have a chain of transformation allowing us to work with complex data in far more powerful ways.

Here's an example -- with more code this time, as we're getting used to the look and feel of optics and morphisms now!

{% highlight fsharp %}
// We know that Ages here will be a comma separated list of zero
// or more integer ages.
type Record =
    { Ages: string }

    static member Ages_ =
        (fun x -> x.Ages), (fun ages x -> { x with Ages = ages })

// An isomorphism to split our string to multiple strings
let stringstrings_ : Isomorphism<string,string list> =
    (fun s -> List.ofArray (s.Split ',')),
    (fun ss -> String.Join (",", ss))

// An isomorphism from a string list -> int list
let stringsints_ : Isomorphism<string list,int list> =
    (List.map Int32.Parse), (List.map string)

// Lens<Record,int list>
let recordints_ =
        Record.Ages_
    >-> stringstrings_
    >-> stringsints_

// Prism<Record,int>
let recordfirstint_ =
        recordints_
    >-> List.head_
{% endhighlight %}

Now we've got two very powerful optics. We could work with the ages transparently as a list of ints, using all of our normal tools for working with lists. Or we could work with the first in age, if it's present -- and still have type checking and safety.

For example:

{% highlight fsharp %}
let record =
    { Ages = "24,56,45,10" }

// 56
let oldest =
    Optic.get recordints_ record |> List.max

// { Ages = "25,57,46,11" }
let record' =
    Optic.map recordints_ (List.map ((+) 1)) record
{% endhighlight %}

As we can see this can be a very strong tool, and the overall combination of lenses, prism, isomorphisms and epimorphisms gives us a complete toolbox for working with complex data, through composition and functional practices.

Hopefully these three guides have given you a sense for what can be achieved, even though they just scratch the surface. For a complete guide to what's available with Aether, there's a complete reference available.

Next steps: [Reference][reference]

<!--- Local --->

[lenses]: /aether/guides/lenses.html
[prisms]: /aether/guides/prisms.html
[reference]: /aether/reference
