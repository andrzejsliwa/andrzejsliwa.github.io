---
layout: post
date: 2017-09-22 18:30
title: "Thoughts after Rails + Domain Driven Design workshop"
tags:
- DDD
- CQRS
- Event Sourcing
---
Today I come back from **Rails + Domain Driven Design** workshop organised and made by [Arkency](http://arkency.com/). To be honest I wasn’t the best target audience for such workshop. The main reason for it is that I can't consider myself as a beginner in the topics of **Domain Driven Design**, **CQRS** or **Event Sourcing**. Even then I took part of this workshop to validate some of my experiences and knowledge about mentioned concepts. 

For the price of the workshop I got **3 strong practitioners** ([Andrzej Krzywda](https://twitter.com/andrzejkrzywda
), [Paweł Pacana](https://twitter.com/pawelpacana), [Robert Pankowiecki](https://twitter.com/pankowecki)) in the same room. 
We had discussed numbers of different topics. Some of them were related to **RailsEventStore** ecosystem. We also talked about directions and roadmap of changes which are [coming to it](https://github.com/RailsEventStore/rails_event_store/pull/86). Some discussions were more related to conceptual scooping and storming. The outcome was some ideas about the scope of workshop and about potential next steps in development of **RailsEventStore**. In particular I proposed a few potential improvements such as:
 
- Versioning of events 
- Strong contracts: [contract.ruby](https://egonschiele.github.io/contracts.ruby/), [dry-types](http://dry-rb.org/gems/dry-types/), [dry-struct](http://dry-rb.org/gems/dry-struct/)
- Validations ([dry-validation](http://dry-rb.org/gems/dry-validation/)) and multi-command forms (Task based UI)
- Error handling strategies for events processing
[https://github.com/RailsEventStore/rails_event_store/issues/111](https://github.com/RailsEventStore/rails_event_store/issues/111)
- Handling of conflicts errors [https://github.com/RailsEventStore/rails_event_store/issues/107](https://github.com/RailsEventStore/rails_event_store/issues/107)
- Support for refactoring (renaming / moving of Events, Aggregates) [https://github.com/RailsEventStore/rails_event_store/issues/113](https://github.com/RailsEventStore/rails_event_store/issues/113)
[https://github.com/RailsEventStore/rails_event_store/issues/112](https://github.com/RailsEventStore/rails_event_store/issues/112)

## The New Hope (or Empire Strike Back;)
The most promising discussion was about idea of how to remove conditional logic from aggregates which slowly become similar to state machines. I think [**Andrzej Krzywda**](https://twitter.com/andrzejkrzywda
) proposed the concept of instead of keeping just one class of aggregate you can model aggregates as multiple classes equivalent to the specific state of the aggregate. This means that on each **apply** of the **event** to the aggregate we are **promoting /  upgrading** aggregate by replacing one instance of class with **new** instance of **class** which represent **next state**. This will let us keep only methods possible to call on specific state (and class) of aggregate. Promoting of Aggregate classes can look like that:

```
Order -> OrderPlaced -> OrderPaid -> OrderCanceled
```

Internal state should then be passed to the new instance once it is promoted. The concept is really interesting. I have few concerns about it:

- How to implement / model promotion process with keeping in mind
that way of testing should stay simple?
- How to handle dependency injection in promotion process?
- How to pass internal state from one instance to the next one?
- How handle use case when method will not be available? (because of promoting). I see so far two solutions for it: check if instance respond to method or let it fail. In such case monitoring will be important

The idea definitely deserves investing some time on it, I would like to verify the thesis: ***The code will be more clean and more explicit than maintaining internal state and using aggregate which act as state machine*** (with raising / throwing custom exceptions on wrong states of aggregate)

I would like to verify this thesis in the near future by prototyping such solution.

## Thanks

Going back to the workshop I would like to thank [Andrzej Krzywda](https://twitter.com/andrzejkrzywda
), [Paweł Pacana](https://twitter.com/pawelpacana), [Robert Pankowiecki](https://twitter.com/pankowecki) for being patient and for giving their time.

## Retrospection

From my observations I saw many people in the workshop who **weren't afraid to ask** any questions and used this time to gain knowledge and also the people who **didn’t ask** and didnt always come back with clear vision and understanding of presented concepts. My advice for such people is clear: **use better** time offered by the experts. **You paid for it**. I don’t know better way of learning than **asking questions**. Reading books will **not answer** questions in case of **doubt** ;)

## Call For Action
Dear readers please let me know what you think about idea of promoting aggregates proposed by [Andrzej Krzywda](https://twitter.com/andrzejkrzywda). Maybe you already tried such approach? Maybe you have other ideas to share? Please share your opinion in the comments. If you are more interested about Event Sourcing and RailsEventStore ecosystem please visit [documentation](http://railseventstore.org/) and [github](https://github.com/RailsEventStore/rails_event_store). Feel free to drop the idea or pull request there.
