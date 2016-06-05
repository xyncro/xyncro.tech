---
title: Chiron
section: reference
order: 1
---

<div class="contents">
    <h1>Contents</h1>
    <ul>
        <li><a href="#the-json-type">The Json Type</a></li>
        <li><a href="#the-json-function">The Json Function</a></li>
        <li><a href="#the-json-computation-expression">The Json Computation Expression</a></li>
    </ul>
</div>

## <a name="the-json-type"></a>The Json Type

The simple heart of Chiron is a basic type model of JSON. It's useful to know what this looks like (a simple recursive discriminated union representing JSON).

{% highlight fsharp linenos=table %}
type Json =
    | Array of Json list
    | Bool of bool
    | Null of unit
    | Number of decimal
    | Object of Map<string, Json>
    | String of string
{% endhighlight %}

Additionally, the Json type includes static members to supply a prism for each potential case (Chiron is intended to be usable with [Aether][aether] -- see Aether for more on prisms and related concepts).

{% highlight fsharp linenos=table %}
Json.Array_  // Prism<Json,Json list>
Json.Bool_   // Prism<Json,bool>
Json.Null_   // Prism<Json,unit>
Json.Number_ // Prism<Json,decimal>
Json.Object_ // Prism<Json,Map<string,Json>>
Json.String_ // Prism<Json,string>
{% endhighlight %}

## <a name="the-json-function"></a>The Json Function

To work with JSON, we also define a function for composing operations on an instance of the Json type, together with a notion of success or failure of an operation.

{% highlight fsharp linenos=table %}
type Json<'a> =
    Json -> JsonResult<'a> * Json

 and JsonResult<'a> =
    | Value of 'a
    | Error of string
{% endhighlight %}

This simple function type lets us define all of the normal functions we need to create a simple monadic way of working with JSON (feel free to ignore the "m-word" part of that sense if you'd rather - you can use Chiron quite easily without diving in to theory if you'd rather not).

These functions are all available in a Json module, along with an extra "error" function:

{% highlight fsharp linenos=table %}
Json.init  // 'a -> Json<'a>
Json.error // string -> Json<'a>
Json.bind  // Json<'a> -> ('a -> Json<'b>) -> Json<'b>
Json.apply // Json<'a -> 'b> -> Json<'a> -> Json<'b>
Json.map   // ('a -> 'b) -> Json<'a> -> Json<'b>
Json.map2  // ('a -> 'b -> 'c) -> Json<'a> -> Json<'b> -> Json<'c>
{% endhighlight %}

## <a name="the-json-computation-expression"></a>The Json Computation Expression

Using the function types defined previously, a simple computation expression is created for working with the Json function, giving an F# native way to work with JSON data.

{% highlight fsharp linenos=table %}
let someJson =
    json {
        ... }
{% endhighlight %}

Much of the remaining Chiron API is designed to work with the Json function type, and so works with the Json computation expression.
