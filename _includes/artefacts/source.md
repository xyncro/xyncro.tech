{% assign project = site.data.projects[include.project] %}

## GitHub

Source for __{{ project.name }}__ is available [on GitHub][github], and development and issue tracking takes place in the open. Issues, pull requests, etc. are all welcomed.

To quickly get the source locally:

```shell
$ git clone {{ project.artefacts.github }}.git
```

## Licensing

{{ project.name }} is made available under an MIT license, which can be found [as part of the source repository][github.license].

[github]: {{ project.artefacts.github }}
[github.license]: {{ project.artefacts.github }}/blob/master/LICENSE.md
