{% assign project = site.data.projects[page.collection] %}

# Packages

## NuGet

Packages (release and pre-release) for {{ project.name }} are released [via NuGet]({{ project.artefacts.nuget }}). {{ project.name }} can be installed using the NuGet Package Manager, or the NuGet Console:

```bat
PM> Install-Package {{ project.name }}
```

## Paket

[Paket](https://fsprojects.github.io/paket/) is recommended as a dependency management tool for .NET projects ({{ project.name }} uses Paket tools), and {{ project.name }} works well with Paket, respecting semantic versioning to make the most of the strict version control and version locking capabilities that Paket provides.
