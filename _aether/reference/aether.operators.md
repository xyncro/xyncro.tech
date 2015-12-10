---
title: Aether.Operators
section: reference
sort: 3
---

Aether can be used without the adoption of any custom operators, but particularly in the case of optic composition, the use of operators can serve to make the code significantly more succinct and readable. The number of operators required has been significantly reduced from earlier versions of Aether, making the usage increasingly friendly to the code reader.

The operators are shown here in their bracketed prefix form to make the type signatures more obvious, but are usually used as infix operators.

## Composition

There are two composition operators in Aether, providing infix versions of the composition functions detailed in [Aether][aether]. They are shown here with the artificial types as used in [Aether][aether] for the function forms.

{% highlight fsharp linenos=table %}
// Lens<'a,'b> -> [Optic|Morphism]<'b,'c> -> Optic<'a,'c>
(>->)

// Prism<'a,'b> -> [Optic|Morphism]<'b,'c> -> Prism<'a,'c>
(>?>)
{% endhighlight %}

The choice of layout is purely subjective, but longer and more complex optic compositions are often easier to read when arranged vertically.

## Operations

As with composition, operators are provided for the three key operations relating to optics, get, set, and map. Again, the artificial types as used in [Aether][aether] are shown.

{% highlight fsharp linenos=table %}
// 'a -> Optic<'a,'b> -> ['b|'b option]
(^.)

// 'b -> Optic<'a,'b> -> 'a -> 'a
(^=)

// ('b -> 'b) -> Optic<'a,'b> -> 'a -> 'a
(^&)

{% endhighlight %}

<!--- Local --->

[aether]: /aether/reference/aether.html
