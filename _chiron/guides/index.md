---
title: Overview
section: guides
order: 1
---

Chiron is a JSON library for F#. It can handle all of the usual things you'd want to do with JSON, (parsing and formatting, serialization and deserialization) as well as providing some additional ways to work with JSON using optics (see [Aether][aether] for more on optics -- Aether is the supported optics implementation).

Chiron works rather differently to most .NET JSON libraries with which you might be familiar, using neither reflection nor annotation, but instead uses a simple functional style to be very explicit about the relationship of types to JSON. This gives a lot of power and flexibility.

We're still working on a full set of guides to Chiron, but there is already writing out there -- see the [Links][links] section.

## Reading and writing Json values

Chiron is split into `Json<_>` which is for reading and writing JSON and `Json` which is the object model for JSON.

Let's start by writing a discriminated union serialiser:

```fsharp
type DU =
  | A
  | B
// serialiser
open Chiron
open Chiron.Operators
open Aether
open Aether.Operators
let serialise (d: DU): Json<unit> =
  match d with
  | A ->
    Json.Optic.set Json.String_ "a"
  | B ->
    Json.Optic.set Json.String_ "b"
```

Similarly, we can write the deserialiser:

```fsharp
let deserialise _: Json<DU> =
      function
      | "a" -> A
      | _   -> B
  <!> Json.Optic.get Json.String_
```

That's the very basics. TODO: writing code for fatter objects and choosing to either keep the serialiser inside or outside of the type declaration.


<!--- Local --->

[aether]: /aether
[links]: /chiron/guides/links.html
