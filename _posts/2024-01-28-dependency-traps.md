---
title: Dependency traps
permalink: dependency-traps
---
Theory goes that a lot what was accomplished in computer science was actually accomplished due to the laziness
of the actors involved. And it's not hard to find examples supporting that theory, but my favorite is about programmers
in the 50s that didn't like writing their programs in assembly language, so they invented new programming language(s)
and accompanying tools (compilers) to translate those new languages back into assembly. True, writing all those tools
took quite some time, but after they were done with them, they suddenly became a lot faster in their craft,
offsetting time spent on building the tools by a large margin.

<!--more-->

And although one would expect that, as our tools become better and hardware faster, the need for improvements in tooling
would drop over time. It seems quite the opposite is happening: for every new technology we invent, sooner or later
someone decides that it's too time consuming to use it and invents some [new way of simplifying it](/simple-technologies-dont-scale).
And although that's how progress is made, that approach can, and often does, have its dark side too.
Especially when we overdo it.

First of all, it ups the rate at which new tools are entering our domain to exponential levels and it's probably
the most significant reason why programmers these days feel that they simply [can't follow the pace of innovations](/stop-promoting-programmers).
Not much less important is segmentation of ecosystems, which actually **slows down** the overall pace of the industry,
by requiring each new ecosystem to invest significant efforts to bootstrap all the tools they decided to replace.
Finally, and possibly the darkest side some "innovations" impose on us is their false promise that they'll simplify our jobs,
but what they do instead is just drive us into a dependency trap escaping from which usually turns out to require
way more time then this new "innovation" ever have saved us.
And omnipresent frameworks are probably the best examples of these traps.

I guess there are good and bad types of laziness. Good ones drive innovation into higher levels of abstraction,
allowing us to be more productive by providing us with different building blocks for our products.
Bad laziness drives us towards tools that promise higher productivity by prebuilding what their authors believe
should be common across all products. These differences are sometimes referred to as a [library vs. framework debate](https://www.brandons.me/blog/libraries-not-frameworks)
in which "library" is something that offers building blocks on a different level of abstraction while at the same time
keeping compatibility with the rest of the ecosystem, while "framework" represents opinionated sub-ecosystem
in which someone (framework authors) have made most of the choices on behalf of their users,
freeing them from having to make them for themself, but at the same time striping them from a freedom to do so, forever.

Inherent problems with every framework is that they indeed cover many common patterns we are all familiar with
and in turn demo pretty well. Add this one annotation and you get a full CRUD interface! Declare that other property
and it automagically gets exposed as a series of API endpoints! It's easy to fall for these promises,
no questions about that. However, as one almost always finds out, sooner or later your product (in case it's successful,
which we all hope for) starts requiring functionalities, optimizations and other details that authors of your framework
of choice never envisioned. And that's where you start feeling trapped: you have already invested a lot of resources
building your product on top of some framework and now, when your product has finally become popular
and you need more specialized features, your familiar framework suddenly started to actually slow you down,
as you invest more and more time trying to find different workarounds for its limitations. In other words,
your much appreciated tool suddenly has turned from a friend to a foe. And, since you have already invested so much time
into building your product around it, refactoring it now is usually not an option, so you keep pushing on,
paying the price (with interest!) of the choices you made early on.

If, instead, you built your product from the start around independent, mutually compatible libraries,
with a clear separation of concerns between them and decoupled from your business logic as much as possible,
then replacing any one of them individually would be much easier task, giving you freedom to optimize for whatever
your specific service does. Doing a lot of JSON (de)serializations? Go ahead and use that obscure, complicated,
but [insanely fast JSON library](https://github.com/ngs-doo/dsl-json)!
Need a type-safe way to work with SQL and some low-level database primitives? Ditch ORMs and use one of those
[SQL](https://www.jooq.org/)-[first](https://jdbi.org/) [libraries](http://querydsl.com/).
Need to support rich payloads and both REST and GraphQL are too slow in your use case?
Try something [highly specialized](https://nar.steatoda.com/) (shameless plug, I know :) ).
Niche libraries like these rarely (if ever) come bundled with any framework.
For sure, not going the framework route and integrating all those libraries yourself will also bear some cost,
but it will also keep the door open for every new opportunity to make your service more optimized.
And having an opportunity to later upgrade any of them individually, without upgrading the whole framework is just an additional benefit.

So whenever someone demos you some shiny new framework that "automagically" does 98% of the job for you ["out of the box"](https://spring.io/projects/spring-boot/),
factor in all the opportunities that you'll inevitably lose if you go on and accept all the boundaries of that framework
(escaping from which may be possible, but it's rarely [easy](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/support/AbstractAnnotationConfigDispatcherServletInitializer.html)).
Or, whenever someone offers you some ["simple DSL"](https://docs.jboss.org/hibernate/core/3.3/reference/en/html/queryhql.html)
as a substitute for an [industry standard](https://en.wikipedia.org/wiki/SQL),
think about is it worth joining that sub-ecosystem for some short-term benefits
if in the end you'll have to learn all the peculiarities, but this time in an [obscure tech](https://www.toptal.com/java/how-hibernate-ruined-my-career).

To end this post with a disclaimer, let me mention one special case that represents a notable exception to this rule: prototypes.
With prototypes, you usually have to build something (barely) functional as soon as possible and then iterate at an even greater pace.
In this case, frameworks, with their pre-built components, are indeed a useful tool to use.
Problem here lies in the fact that many "prototypes" simply become "products", be it from the lack of time, resources
or simply by declaration from the management. [Don't fall into that trap](https://blog.cleancoder.com/uncle-bob/2014/05/11/FrameworkBound.html).