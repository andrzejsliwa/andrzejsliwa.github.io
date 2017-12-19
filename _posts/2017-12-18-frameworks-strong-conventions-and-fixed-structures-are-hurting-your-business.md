---
layout: post
date: 2017-12-18 7:30
title: "Frameworks, strong conventions and fixed structures are hurting your business"
tags:
- DDD
- Frameworks
- Event Sourcing
---
For every young engineer frameworks and strong conventions are the source of peace, a “safe harbour”, an ultimate answer he or she desires to know. Giving a stable environment, building blocks where one matches another, where each piece has its own place where it fits best.

Frameworks and conventions promise to bring order to the chaotic world surrounding us. Well defined structures, everywhere.

If you leave well defined frames, borders, chains of conventions and structures behind you, then you’ll see that the only problem you should focus on is the domain problem you’re working on. Your domain problem is the only important problem to solve, if you’re neglecting frameworks and conventions for a second.

If you’re able to step away from trying to force your system design into REST conventions or directory structures required by your framework, then there’s a chance to come back to the essence of the problem you’re trying to solve.

The key points are here:

- How does the business process look like, exactly?
- How is the business working?
- What are the workflows between participants?
- Which events happen when and where?
- What is the source of these events?

What are we doing instead? We are trying to fit every problem in to the shape of our frames. We’re doing this often very strongly, using our muscle memory. We form code like Play-doh® in our hands, so we do with business requirements trying to fit them into the shape of our frames required by conventions and frameworks.

Somebody wrote: “Framework is code, where if somebody removes all business domain specifics, you’d remain with all the assumptions”.

By using strong conventions or well defined shapes, you’re damaging your perception of the problem or the business domain. If something doesn’t fit, you’re applying “force” to make it fit. This way some pieces fall apart, some pieces are lost. These pieces are important. These pieces will make a difference.

Over years business people and product owners were demotivated trying to explain their domain problems to engineers. They tried and failed, because engineers only saw technical challenges, parts, CRUDs, RESTful operations and all of these building blocks provided by the framework of their choice.

Today, business people and product owners are trying to use the same technical jargon to be able to communicate with engineers. But the real context, the real meaning is lost in the translation.

In many software communities I see a positive development. Small and well defined libraries or packages are preferred as building blocks over frameworks with strong conventions. These libraries or packages don’t provide opinionated DSLs. They provide just a simple API giving you full freedom about their usage.

These kind of building blocks remind me of LEGO®. Sure you can build one model with every set (or even two, following the provided instructions). These building blocks have a simple universal shape, which can be combined to form more complex shapes. They are small, playing a very small role. You can use them to build anything you can imagine. There’s not a single place where they fit, they fit everywhere you need them.

Somebody will tell me: “You know but thanks to this standardisation, formalisation done by frameworks, we can deliver faster and more often, we are more productive”.

This reminds me one tweet:
<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Worried that TDD will slow down your programmers? Don&#39;t. They probably need slowing down.</p>&mdash; ☕ J. B. Rainsberger (@jbrains) <a href="https://twitter.com/jbrains/status/167297606698008576?ref_src=twsrc%5Etfw">February 8, 2012</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>



Maybe, just maybe you should slow down. Stop this racing machine and think, if you really understand what you’ve been ask to build. Whether your perception of the requirements matches those of your product owner.

After all, it’s not only a matter of performance or speed. You can own a $ 2M race car, you can drive it at 420 kmph, but what if I’ll tell you: it’s in the wrong direction? We’re not doing software development for the art or for the number of tickets delivered weekly. We’re doing it to solve real problems, to deliver business value.

I’m pretty sure if you would choose 5 software developers randomly and give them same ticket, they will build 5 totally different implementations. Because it’s matter of perception.

What would happen when they would choose 5 different frameworks? You can guess!

But why does this happen? Probably all of them know the same rules, paradigms, coding standards, methodologies, architectures, design patterns.

We’re all human. And like in the case of frameworks and conventions, we like it when decisions are made for us. Because that’s exactly what happens when you’re using them. Very often you don’t even know those people and their values.

If this sounds familiar to you, please stop for a moment. Take some time to reflect. Take a look on the different options you have. Maybe take a closer look on the approach Domain Driven Design takes. Just read or watch more about Event Storming process.

Go back to the roots, where the real value is deep understanding of the problem domain. For a nice beginning stop using technical jargon at work which makes your vocabulary much poorer. Focus on the words used by the business, try to understand their meaning. You have the full right not to understand it in the first place. You’re responsible to ask for an explanation as many times as it’s needed until you and your domain experts agree and all of you talk about the same thing. Keep these words, their meaning and use them in your software. Unambiguous Language is your treasure.