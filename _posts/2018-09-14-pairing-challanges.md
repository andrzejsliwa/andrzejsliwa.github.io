---
layout: post
date: 2018-09-14 6:00
title: "Pairing Challenges"
tags:
- Pairing
---
Pair programming is a popular technique in which two software engineers work together at the same workstation, by sitting in front of the same computer or by screen/keyboard/codebase sharing in real time.
I'm practicing Pairing already for a few years with excellent outcomes but also with some tradeoffs. 
Today I would like to focus on tradeoffs.
Recently I had feedback talks with my team members about tradeoffs related to pairing. 
In discussions, they provided lots of insights from last few weeks of pairing together. Based on these discussions and my experience I formed a few open questions:

- how to maximize pairing benefits and avoid issues like being too exhausted, brain melted, etc.?
- how to let 3rd person jump in (in same feature team) and understand technical issues of the project?

# Trade-offs

Based on this questions we can separate a few tradeoffs related to constant pairing:

- super exhausting
- not flexible (working hours, daily rhythm breaking, blocking)
- knowledge transfer only between both pairing mates

# Ideas

In discussions with my team members I collected a set of ideas which address some of the tradeoffs listed above:

#### Pairing only on kickoff - of structure/architecture then splitting

Cons:
  - none

Pros:
  - this will let decide together about the architecture and structure of the project, about serious blockers come from development
  - then split on particular tickets based on agreed structure & contracts
  - this will reduce stress level and exhaustion
  - this will let us work in a more flexible way while staying in sync
  - this will provide better performance and velocity

#### Live reviews - without specific time constraints or rules, an extended version of GitHub PR reviews

Cons:
- will require more time
- will need the context switch

Pros:
- will give a chance to discuss the technical decision with full understanding of decision context
- will provide better quality reviews (than just code style, used tips&tricks, etc.)

#### Small posts - where you can show cons/pros and share your experiences and lesson learned

Cons:
- will require some time to prepare it

Pros:
- let us present our point of view and lesson learned without running long, live and unstructured debates
- will structure out thoughts
- will be a perfect base for retrospecting our self

#### Wrap up meetings - workshops/presentations/live coding for all technical teams, once per 1 or 2 weeks, to share architecture and technical shifts and lesson learned

Cons:
- will require some time to prepare for it
- will need some time to participate
- will be difficult to avoid unstructured debates or discussions

Pros:
- will make technical knowledge transfer between tech teams
- we will have a chance for discussion

#### Once per month Technical All Hands - for the whole business department/company to present our work and share outcomes with non-technical team members, with a focus on business value)

Cons:
- will require some time (we already doing it)
- I'm still not sure how to structure discussions or open questions with the large audience

Pros:
- it's increasing transparency
- giving more context
- let us focus more on business value than the technical aspects
- allow us to collect feedback

#### More **Async** way - [Async Remote](https://blog.arkency.com/async-remote/)

Cons (and Pros)
- will require more communication on dedicated slack channels
- will need more structured discussion focused on a well-defined outcome (when will be difficult to achieve the outcome we can write a post about the subject)
- will persist one-to-one discussion outcomes to rest of the team (as summary similar to "Meeting minutes")
- will increase the number of channels, but it will be more structured on oriented on the subject (vs. discussion on Engineering channel)

# Conclusion
Pairing is an individual experience, limited by personal preferences, working style, and personality. 
Founding good pairing mate isn't easy and requires building the trustful and respectful relation, 
after years of working together some mates can almost read your mind. Like every technique have own trade-offs.
Provided ideas are not ultimate answers, they are instead set of ideas about how to make pairing a more enjoyable 
experience and more beneficial for all of us.