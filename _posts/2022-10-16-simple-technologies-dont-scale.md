---
title: Simple technologies don't scale
permalink: simple-technologies-dont-scale
---
Most of us are intimidated by big, complex problems. And since most of us are a bit cocky (in a positive way!)
we are often inclined to dismiss inherited complex products and technologies as "old" or "bloated" and tend to solve
every problem with "refactor" using some "new technology". Similarly, we tend to dismiss as "not needed"
most of the things we don't understand. Don't know about you, but I have to confess that throughout my career
I have sinned on all of those accounts.

<!--more-->

Let me tell you one secret: complex things did not end up that way because their authors were sadists.
They are complex because problems they are supposed to solve are... guess what... utterly complex! During my career,
I've seen many cycles in which the New Tech emerges, much simpler to use than the Old Tech. Demos are appealing,
conferences overflowed with talks about it. Gradually, new programmers adopt New Tech and start building
complex projects around it. However, as their projects grow in size, certain problems start to emerge,
problems nobody envisioned during New Tech's demo days. One after another, new features are added to New Tech,
everyone happily adopts them (since they are now aware why they need them) and for some time everyone is happy.
However, since the arrow of time goes irreversibly forward, even newer programmers start joining the team
and they start perceiving this New Tech as "bloated". Soon, the Even Newer Tech emerges,
much simpler to use than the New Tech. Demos are appealing, conferences overflowed with talks and a new cycle begins.

I've seen these cycles multiple times:

* Java started its life as a simplified C++. Over time, JDK grew in size from having 212 classes
  in version 1.0 to 4,124 classes in JDK 19. Add to that **order of magnitude** more classes from Java's ecosystem
  that you are expected to be familiar with in everyday development. Also, don't let it slip your mind that
  each of those classes has on average 10 member methods you can call.

* PHP was born around the same time as Java and back then it didn't have even basic constructs as an `if` statement.
  Today, PHP has advanced concepts like class inheritance and no less than 8,745 built-in functions.

* JavaScript was born at that time, too. I can't find exact size of standard library it had in version 1.0,
  but I do remember that I went through everything there was to read about it in a **single weekend**.
  Today's JavaScript is mastery of asynchronous programming, has class inheritance and an unimaginably huge ecosystem.

I could continue this list, but you get the [point](https://xkcd.com/2677/).

Unfortunately, forces driving developers to seek solutions in simpler tools are strong. Let me give you one example.

Recently I had to publish a new library to the Maven Central repository.
In short, these are the [steps](https://central.sonatype.org/publish/requirements/) I had to follow:

1) Verify my domain, so I could get permission to publish my artifacts under reverse domain group ID.
2) Generate PGP keypair and sign release. I had an existing PGP keypair, but it expired, so I had to google how to extend it.
3) Publish the public key so anyone who downloads my artifacts can verify their authenticity.
4) Figure out how Maven Central treats `-SNAPSHOT` releases and how it treats regular releases.
5) Satisfy various Maven Central conventions to be allowed to publish my artifacts.
6) Setup Gradle build for this whole workflow.
7) Setup GitHub Action to automatically publish new artifacts when I tag my repo.

Although I have considerate experience using Java and its tooling, these steps took me **almost the whole day**.
I never published the NPM module, but [from what I could read](https://docs.npmjs.com/creating-and-publishing-scoped-public-packages)
their rules look much simpler. Cargo is notorious for allowing you to
[depend directly on the Git repository](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#specifying-dependencies-from-git-repositories).
Certainly, both look like they are much easier to use.
And they probably are. At least until you first [miss](https://arstechnica.com/information-technology/2016/03/rage-quit-coder-unpublished-17-lines-of-javascript-and-broke-the-internet/)
strict publishing rules or meet issues reproducing builds of your old project that referenced deleted Git repo.
Also, I have no doubt that at some point both NPM and Cargo will tight up their rules and force their users
do most of the complex steps Maven Central requires today, at which point someone will decide that they have also became
"bloated", invent New Simple Tool and a new cycle will begin.

I admit it may be a bit opinionated, but I believe that at least to some degree, these cycles are caused by
[programmers migrating to management](/stop-promoting-programmers), hampering transfer of knowledge
and experience and leaving younger devs to learn a lot on their own, often repeating the same mistakes we all did
when we were starting. I bet in a few years no one will remember the leftpad incident anymore.
	
My advice to anyone entering software development is: be skeptical about things that demo well.
In your career, you (hopefully) won't be writing demos, but complex software that will require you to solve
some hard problems. And those problems could not be solved with "few lines of code" or yet another annotation.
Instead, you'll need complex tools, probably just like the ones you ditched at the beginning of your career.

Let me finish this with quote from Jim Rohn:

> Don't wish it was easier, wish you were better.  
> Don't wish for less problems, wish for more skills.  
> Don't wish for less challenge, wish for more wisdom.
