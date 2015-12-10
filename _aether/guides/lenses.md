---
title: Lenses
section: guides
sort: 2
---

__Note:__ This guide assumes (as with F#) that our data structures are immutable. Lenses in mutable languages are less valuable. When we talk of getting and setting values, we mean returning a new, modified version of a data structure.

When people first come across lenses, the first reaction is often one of doubt. Why would we want to take some seemingly complicated approach to working with data, when the same thing must be possible using existing language features?

To see why we might want to consider lenses, we'll look at a simple example. We'll define three record types:

{% highlight fsharp %}
type RecordA =
    { B: RecordB }

 and RecordB =
    { C: RecordC }

 and RecordC =
    { Value: string }
{% endhighlight %}

Now, let's assume we've got an instance of RecordA, and try and work with it:

{% highlight fsharp %}
let a =
    { B =
        { C =
            { Value = "Hello World!"} } }

// Getting the inner value is quite simple...
let value =
    a.B.C.Value

// Setting it to a new value is less so...
let a' =
    { a with B =
                { a.B with C =
                            { a.B.C with Value = "Aargh!" } } }
{% endhighlight %}

As we can see, even with only three types and levels of nesting in our data structures, working with values deep within the data structure starts to get awkward. It's beginning to be confusing to read, even with some fairly aggressive use of whitespace and indentation and very simple type names!

With real world code, and more complex data structures, this begins to get very difficult to read and maintain.

## Discovering Lenses

What we want is a way of working with any part of a data structure in a consistent and predictable way. Let's start with a simpler case -- just two types -- and find a different way to work with them that's more consistent.

First of all, our types:

{% highlight fsharp %}
type RecordA =
    { B: RecordB }

 and RecordB =
    { Value: string }
 {% endhighlight %}

### Get

We'll begin by finding a more general way of getting a part of a data structure -- a simple getter function. Here are the two we need for our types:

{% highlight fsharp %}
let getB =
    fun a -> a.B

let getValue =
    fun b -> b.Value
{% endhighlight %}

They're very simple, but the important thing to note is that they're also very general. In this case we have record types, but as long as we have a function which can return a part of a data structure from a whole data structure, that's a usable getter -- and the mechanism is now shielded from the caller of the function.

Let's use these simple functions to write another function for getting the string value from inside an instance of RecordA:

{% highlight fsharp %}
let get =
    fun a -> getValue (getB a)
{% endhighlight %}

Still very simple, and it's clear that we can simply combine getters to get a new and more powerful getter. In fact this is just very simple function composition -- we could write the above function like so:

{% highlight fsharp %}
let get =
    getB >> getValue
{% endhighlight %}

### Set

The next thing to do is to write the obvious pair of functions -- setters:

{% highlight fsharp %}
let setB =
    fun b a -> { a with B = b }

let setValue =
    fun value b -> { b with Value = value }
{% endhighlight %}

Quite simple, and once again, though we're using records and the update mechanism is record update syntax, that mechanism is not exposed outside the function. They're just simple functions. Let's write the simple combined setter for setting the string value given an instance of RecordA.

{% highlight fsharp %}
let set =
    fun value a -> setB (setValue value (getB a)) a
{% endhighlight %}

What are we doing here? Well, it's more combining of functions, although the setter is a little more complicated than the getter. We need to get the existing value of B, set a new string value on it, and then set that to be the new value
of B. This means that we need to use a getter function here too.

However, with just a very simple get and set function for each type, we've managed to create two new functions which simplify a previously more complex operation.

### Composition

Of course, if we had to manually combine functions every time we wished to do this, we wouldn't be saving ourselves much work at all. As it turns out, we can standardize this easily.

Let's define a type called Lens:

{% highlight fsharp %}
type Lens<'a,'b> =
    ('a -> 'b) * ('b -> 'a -> 'a)
{% endhighlight %}

It's nothing more than a pair of functions - a getter and a setter. We've made it generic, but all of the functions we defined previously match this signature with the appropriate type parameters.

Now we can say that given two lenses, we could combine the getters to form a new getter -- and combine the setters and one of the getters to form a new setter. In short, we can combine (or compose) two lenses to form a new lens!

Here's a function which will compose two lenses:

{% highlight fsharp %}
// Lens<'a,'b> -> Lens<'b,'c> -> Lens<'a,'c>
let compose =
    fun (g1,s1) (g2,s2) ->
        (fun a -> g2 (g1 a)), (fun c a -> s1 (s2 c (g1 a)) a)
{% endhighlight %}

Although this looks awkward as we've pattern matched the getters and setters from our arguments, it's just doing the same as our composed functions above -- but now for all lenses. We've now got a way of working with data at any point within a larger data structure, as long as we can find (or make) a lens from one part to the next.

Of course, to make life easier, we can write the general version of our get and set functions. Now we can simply give them a lens, and receive the function to get or set whatever the lens defines.

{% highlight fsharp %}
// Lens<'a,'b> -> 'a -> 'b
let get (g,_) =
    fun a -> g a

// Lens<'a,'b> -> 'b -> 'a -> 'a
let set (_,s) =
    fun b a -> s b a
{% endhighlight %}

Note that our lenses can be composed of any number of other lenses now. Composing 5 lenses to focus in on data deep within a complex data structure is now trivial -- and looks and feels as simple as working directly with any other functions.

### Map

One last thing to add here, is that the concept of mapping using a lens is now trivial -- maping a function over a part of a data structure given a lens is as simple as "get the value, apply the function to the value, set the new value to be the result".

## Aether

Of course, Aether provides all of this and more out of the box. All that you need to provide to use Aether are lenses appropriate to your own data structures. Let's see what this would look like in Aether, with our types modified to provide lenses:

{% highlight fsharp %}
type RecordA =
    { B: RecordB }

    static member B_ =
        (fun a -> a.B), (fun b a -> { a with B = b})

 and RecordB =
    { Value: string }

    static member Value_ =
        (fun b -> b.Value), (fun value b -> { b with Value = value })
{% endhighlight %}

All we've had to do is find somewhere to store some lenses (by convention, we usually add them as static members to types like this, and suffix the names with an underscore).

Now we can use Aether:

{% highlight fsharp %}
open Aether

let a =
    { B = { Value = "Hello World!" } }

// A lens, composed using Aether
let avalue_ =
    Compose.lens RecordA.B_ RecordB.Value_

// Get the value using an existing lens
let value =
    Optic.get avalue_ a

// Set the value using an existing lens
let a' =
    Optic.set avalue_ "GoodbyeWorld!" a
{% endhighlight %}

In fact, Aether actually lets us make this more succinct if we wish. Rather than using Compose.lens, we could simply compose our lenses in place. We could also use a terser syntax with composition operators.

{% highlight fsharp %}
open Aether
open Aether.Operators

let a =
    { B = { Value = "Hello World!" } }

// A lens, composed using Aether operators
let avalue_
    RecordA.B_ >-> RecordB.Value_

// Get the value using an existing lens
let value =
    Optic.get avalue_ a

// or...

// Set the value using a new lens
let a' =
    Optic.set (RecordA.B_ >-> RecordB.Value_) "GoodbyeWorld!" a
{% endhighlight %}

(Although not shown, Aether provides an Optic.map function, for mapping a function given a lens).

Although in the case of our simple two type system, this might seem complex, the complexity remains constant regardless of the increasing complexity of our data structures. The small investment quickly pays off as our requirements become more taxing.

There's lots more to Aether -- it helps you cover more complex and demanding cases than are seen here. But all of the functionality is built from these same principles -- that composing general functions to work with data structures is the most powerful and flexible way to work, and avoids special cases and syntax which become difficuly to comprehend in complex cases.



Next steps: [Prisms][prisms].

Relevant reference: [Aether][aether] and [Aether.Operators][aether.operators].

<!--- Local --->

[prisms]: /aether/guides/prisms.html
[aether]: /aether/reference/aether.html
[aether.operators]: /aether/reference/aether.operators.html
