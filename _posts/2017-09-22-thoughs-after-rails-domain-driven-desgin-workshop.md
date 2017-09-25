---
layout: post
date: 2017-09-22 18:30
title: "Thoughts after Rails + Domain Driven Design workshop"
tags:
- DDD
- CQRS
- Event Sourcing
---
Today I come back from **Rails + Domain Driven Design** workshop organised and made by [Arkency](http://arkency.com/). To be honest I wasn’t best target audience for such workshop. The main reason for it is that I can't consider my self as beginner in topics of **Domain Driven Design**, **CQRS** or **Event Sourcing**. Even then I took a part of this workshop to validates some of my experiences and knowledge about mentioned concepts. 

For the price of training I got **3 strong practitioners** ([Andrzej Krzywda](https://twitter.com/andrzejkrzywda
), [Paweł Pacana](https://twitter.com/pawelpacana), [Robert Pankowiecki](https://twitter.com/pankowecki)) in same room. 
We had discussed numbers of different topics. Some of them were related to **RailsEventStore** ecosystem. We also talk ed about directions and roadmap of changes which [coming to it](https://github.com/RailsEventStore/rails_event_store/pull/86). Some discussion were more related for conceptual scooping and storming. The outcome of it was some ideas about scope of training, about potential next step in development of **RailsEventStore**. In particular I proposed already few potentials improvements such as:
 
- versioning of events 
- strong contracts: [contract.ruby](https://egonschiele.github.io/contracts.ruby/), [dry-types](http://dry-rb.org/gems/dry-types/), [dry-struct](http://dry-rb.org/gems/dry-struct/)
- validations ([dry-validation](http://dry-rb.org/gems/dry-validation/)) and multi-command forms (Task based UI)
- error handling strategies for events processing
[https://github.com/RailsEventStore/rails_event_store/issues/111](https://github.com/RailsEventStore/rails_event_store/issues/111)
- handling of conflicts errors [https://github.com/RailsEventStore/rails_event_store/issues/107](https://github.com/RailsEventStore/rails_event_store/issues/107)
- support for refactoring (renaming / moving of Events, Aggregates) [https://github.com/RailsEventStore/rails_event_store/issues/113](https://github.com/RailsEventStore/rails_event_store/issues/113)
[https://github.com/RailsEventStore/rails_event_store/issues/112](https://github.com/RailsEventStore/rails_event_store/issues/112)

## The New Hope
The most promising discussion was about idea of how to remove conditional logic from aggregates which slowly become similar to state machines. I think [**Andrzej Krzywda**](https://twitter.com/andrzejkrzywda
) proposed the concept when instead of keeping just one class of aggregate you can model aggregate as multiple classes equivalent to specific state of aggregate. This means that on each **apply** of the **event** to the aggregate we are **promoting /  upgrading** aggregate by replacing one instance of class with **new** instance of **class** which represent **next state**. This will lets keep only methods possible to call on specific state (and class) of aggregate. Promoting of Aggregate classes can look like that:

```
Order -> OrderPlaced -> OrderPaid -> OrderCanceled
```

Internal state should be then passed to new instance once is promoted. The concept is really interesting. I have few concerns about it:

- how to implement / model promotion process with keeping in mind
that way of testing should stay simple?
- how to handle dependency injection in promotion process?
- how to pass internal state from one instance to the next one?
- how handle use case when method will not be available? (because of promoting). I see so far two solutions for it: check if instance respond to method or let it fail. In such case monitoring will be important

The idea definitely deserves for investing some time on it, I would like to verify the thesis: ***The code will be more clean and more explicit than maintaining internal state and using aggregate which act as state machine*** (with rising / throwing custom exceptions on wrong states of aggregate)

I would like to verify this thesis is near future by prototyping such solution.

## Thanks

Going back to the training I would like to thanks to [Andrzej Krzywda](https://twitter.com/andrzejkrzywda
), [Paweł Pacana](https://twitter.com/pawelpacana), [Robert Pankowiecki](https://twitter.com/pankowecki) for being patien and for given time.

## Retrospection

From my observations I saw many people on workshop who **didn’t afraid ask** any questions and use this time to gain the knowledge and also the people who **didn’t ask** and not always come back with clear vision and understanding of presented concepts. My advice for such people is clear: **use better** time offered by the experts. **You paid it**. I don’t know better way of learning than **asking questions**. Books through the reading will **not answer** questions in case of **doubt** ;)

## Call For Action
Dear Readers please let me know what do you think about idea of promoting aggregates proposed by [Andrzej Krzywda](https://twitter.com/andrzejkrzywda). Maybe you already tried such approach? Maybe You have other ideas to share? Please share your opinion in comments. If you are more interested about Event Sourcing and RailsEventStore ecosystem please visit [documentation](http://railseventstore.org/) and [github](https://github.com/RailsEventStore/rails_event_store). Feel free to drop the idea or pull request there.
