{% assign project = site.data.projects[include.project] %}

## NuGet

Packages (release and pre-release) for __{{ project.name }}__ are released [via NuGet][package]. {{ project.name }} can be installed using the NuGet Package Manager, or the NuGet Console:

{% highlight batch %}
PM> Install-Package {{ project.name }}
{% endhighlight %}

## Paket

[Paket][paket] comes highly recommended as a dependency management tool for .NET projects ({{ project.name }} uses Paket tooling for development). {{ project.name }} has been designed to work well with Paket, respecting semantic versioning to make the most of the strict version control and version locking capabilities that Paket provides.

Additionally, {{ project.name }} is written as [a single file][file] that can be directly referenced as a source file using the Paket [GitHub dependencies][paket.github] features.

[package]: {{ project.artefacts.nuget }}
[paket]: https://fsprojects.github.io/Paket/
[paket.github]: https://fsprojects.github.io/Paket/github-dependencies.html
[file]: https://github.com/xyncro/{{ page.collection }}/tree/master/src/{{ project.name }}/{{ project.name }}.fs
