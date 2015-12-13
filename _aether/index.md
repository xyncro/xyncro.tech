---
title: QuickStart
---

Hopefully you're here because you're interested in __Aether__, an open source optics (lenses, prisms, etc.) library for [F#][fsharp]. You might simply want to start with this:

{% assign project = site.data.projects["aether"] %}

{% highlight batch %}
PM> Install-Package Aether
{% endhighlight %}

If you'd like a bit more information however, we've got you covered.

## Chat

If you want to ask any questions on Aether, or have suggestions, please come along and chat in the [Aether Room][room].

## Guides

If you're not familiar with lenses and related ideas in functional programming, you probably want to start with the [Guides][guides] which will take you through an introduction (including motivations and key ideas), all the way to being comfortable with using Aether.

The [Guides][guides] section also has a handy section of links to other places discussing lenses, etc. if you want to dig further in, or explore how Aether compares with other language implementations.

## Reference

If you're already familiar with lenses and other optics in functional programming, you probably want to head straight to the [Reference][reference], or even just grab the [Packages][packages] and get started.

## Contributing

Contributions in any form are welcome. The easiest way to contribute is via the main Git repository, detailed in [Source][source].

## Updates

For updates to Aether, including releases, new tutorials, reference documentation, etc. keep an eye on [Updates][updates] and follow [{{ site.twitter.name }}][twitter] on Twitter.

## Acknowledgements

Aether is a much more useful and well designed/tested library now than it was, which owes much to the work of various people (both code and ideas/suggestions). Particular gratitude must go to [Marcus Griep][griep] for his contributions, which have made Aether more reliable and useful.

<!--- Local --->

[guides]: /aether/guides
[reference]: /aether/reference
[packages]:  /aether/packages
[source]: /aether/source
[updates]: /aether/updates

<!--- External --->

[room]: https://gitter.im/xyncro/aether
[fsharp]: http://fsharp.org
[twitter]: {{ site.twitter.url }}

<!--- People --->

[griep]: https://github.com/neoeinstein
