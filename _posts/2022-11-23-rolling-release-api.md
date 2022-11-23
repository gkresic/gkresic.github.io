---
title: Rolling-release API
permalink: rolling-release-api
---
Every programmer knows that the only stable product is a dead product. Live products, on the other hand,
constantly evolve to adapt to changing markets, new users or both.

<!--more-->

Iterating a product in today's highly available world where downtimes are often perceived as something that should
be avoided whenever possible is an art on its own and things get even more complicated when that product involves
multiple components that communicate over some standardized interface like API.
I'm not talking about microservices here - even simple backend + frontend architecture (aka "webapp") uses some form
of API for communication. If your backend is also used by mobile clients or maybe even 3rd party clients
that are completely outside of your control, upgrades get even more complicated.
Problem is that it's usually not simple - if even possible - to synchronize upgrades to every component in the ecosystem
(for example, the long review process on app stores is something none of us can control).

Instead, we need to somehow:

1. For some transitional period support both deprecated and new elements in our API.
   Note that this implies that we can not delete anything before deprecating it first.
2. Detect and track usage of all those deprecated elements and log it somewhere,
   so that we know when it's safe to completely remove them from our API. As a bonus, it would be nice to somehow signal
   to the clients whenever they request something that is being deprecated, so that they could automatically
   detect changes in our API without requiring them to poll our API docs regularly.

Unfortunately, most of the backend developers still default to a simple, but crude and expensive solution
to this problem: API versioning. You all have seen it, URLs like `https://api.domain.com/v1/...`,
maybe even have implemented a few yourself. Idea behind it is simple: whenever you need a backward-incompatible change,
bump an API version number, **duplicate** functionality in the new version adding new and omitting deprecated elements.
For some time, you'll maintain both old and new versions, track usage of both and when the number of requests
on the old version drops below an acceptable threshold, delete it from the app.
Sounds good, but in practice there is one big issue with this approach: it's **expensive**.

You all guess where the costs came from - that duplication of functionality needed every time you bump a version number.
Duplicated code needs twice as much maintenance, tests and supervision. All extremely costly
and that's a reason you see a lot of `v1` APIs, some `v2`, but far less `v3` and almost none of `v4`, `v5` or greater
([Facebook](https://developers.facebook.com/docs/graph-api/changelog) being one notable exception,
but their concept of "expensive" is probably *slightly* different from yours :) ).

Before proposing a better solution, let me first classify the type of changes that are most common in any API.

First are the changes in endpoint signatures, meaning for example either path to your REST API or query parameters
it uses have changed. Sometimes some endpoints get deleted, without any successor. These types of changes
are relatively simple to handle, but they also represent a minority of cases when we are talking about API changes.

Far more common and way more challenging are the changes within data structures used in requests and/or responses.
One common scenario is deprecating a property in one of your entities or replacing it with another,
often of different type. For example, let's say we have an entity called `User` that has text property `role`
which designates what role within our app that user should have (like `viewer`, `manager`, `admin` etc.).
Let's also say that at some point we need to support users that can be assigned to more than one role.
In this case, just dropping `role` property from our model is not feasible, since existing clients still expect it.
Instead, we'll probably decide to go with backward-compatible upgrade: keep `role`,
but introduce new property `roles` of type `List<Role>`. For some transitional period, we'll initialize `role`
with first (or highest, or whatever suits our business domain) role in our new data model so that old clients
have at least something to work with and `roles` will hold all of them and be used by upgraded clients.
Over time, clients will be upgraded to read `roles` instead of `role` and we would be able to drop `role` from our model.
Obvious question is: when is it safe to drop deprecated `role` property? How do we know that all clients are upgraded?

On a side note, note that applying version numbers to your API basically turns this second (complicated) problem
into first (easy) one, but at the cost of very expensive migrations.

Now, is there a way to handle both of these changes **without** applying versioning to our API?

Like I already said, for API endpoints and their parameters we know exactly which endpoint client has invoked
and which query parameters it provided, so maintaining backwards compatibility is easy.
If an app receives a request on a deprecated endpoint it shouldn't be too complicated to apply needed transformations
to convert that request into a new form. Important: **don't rewrite request URLs** on your reverse proxy,
since that will prevent you from detecting those deprecated requests in your application.

Tracking usage of certain properties from our data model is far more complicated. Since they are used by the client,
the only way to detect their usage is by requiring the client to provide that information when making an API request.
That specification may be purely informational, but it's much more error-proof if you actually return back to the client
only properties it explicitly requested. This way there will never be any discrepancy between properties that client
says it's requesting and what it's actually being read on its side.
Fortunately, there are
[many](https://graphql.org/#without-versions)
[protocols](https://jsonapi.org/format/#fetching-sparse-fieldsets)
[already](https://www.odata.org/)
[supporting](https://nar.steatoda.com/motivation/#versioning)
this way of communication and it's up to you to decide which one you'll adopt (full disclosure: I'm the author of Nar).
Note that this puts some pressure on the clients to maintain definitions of object graphs they are requesting from the API,
but that effort quickly pays off the moment you introduce a first change into your API. As a bonus,
by specifying exact object graphs you wish to retrieve from API, you can optimize your requests to return
only the things that are really needed, reducing both bandwidth usage and latency.
Implicitly, this also frees you from having to maintain different APIs for different clients
([Backend for frontend pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/backends-for-frontends))
since every client can use differently hydrated responses.

One special case of change in data model is when you change the type of response itself, let's say from returning
one object to returning an array of them. This is most easily solved by - not changing response types, of course :).
Instead, design your API up front in a way that each endpoint doesn't return your data model objects directly,
but some dedicated "response" object that wraps the data model that represents actual response payload.
So, instead of returning your `User` entity directly, return `Response` entity that has the named `user` property
that holds the user you want to return. This way your clients will have to specify which properties in responses they require,
allowing you to track access to deprecated ones. Additionally, using these response "wrappers" allows you to
augment responses with additional metadata like paging information etc.

Finally, I already mentioned that it's a good practice to warn clients automatically when they make a request
that accesses some deprecated part of your API. Once you implement everything to be able to *detect* such cases,
choosing a warning mechanism should be easy and dependant on the API protocol you are using. For most common REST APIs,
returning some detailed warning as part of HTTP response headers will usually suffice.
