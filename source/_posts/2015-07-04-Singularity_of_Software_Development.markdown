---
layout: post
title: Singularity of Software Development
date: 2015-06-04 23:48:23 +0530
comments: true
categories: Design, Software
---

_A physician, a civil engineer, and a computer scientist were arguing about what was the oldest profession in the world. The physician remarked, “Well, in the Bible, it says that God created Eve from a rib taken out of Adam. This clearly required surgery, and so I can rightly claim that mine is the oldest profession in the world.” The civil engineer interrupted, and said, “But even earlier in the book of Genesis, it states that God created the order of the heavens and the earth from out of the chaos. This was the first and certainly the most spectacular application of civil engineering. Therefore, fair doctor, you are wrong: mine is the oldest profession in the world.” The computer scientist leaned back in her chair, smiled, and then said confidently, “Ah, but who do you think created the chaos?”_

One prime (not the only) reason for the chaos is rigidity of software architecture and design, by rigidity I meant how easy or hard is it to replace an existing algorithm or a feature or a component with a new and better one. If a component can be replaced with a better one, like replace one caching strategy with other, RDBMS with NoSQL without having to refactor entire code base then design/architecture can be called flexible or stable. But how many times changes like these are contained to a class or two.

In present world where there are millions of software developers connected through internet and in the virtual company of Uncle Bob, Martin Fowler, Mark Seemann and a long list of craftsmen we still see software projects with rigid architectures and bad code. Why do these happen?

In Ramayan, Sita asks Ravana

#### इह सन्तो न वा सन्ति सतो वा नानुवर्तसे |

#### तथाहि विपरीता ते बुद्धिराचारवर्जिता ||

Which roughly translates to “Is there no learned person in Lanka who can preach you righteous behavior or is it that he/she preaches and you don’t listen!!! Where is the problem?”

This dialogue between Sita and Ravana suits software professionals more than any other. There are myriad of articles, books, videos on OOAD, SOLID principles and, Patterns and Practices by renowned software professionals that will help in building optimal designs and architecture. But good designs and code are rare sightings. One reason for current status quo, is that engineers and architects spend good amount of time debating about current and upcoming features and creating architecture and designs for present and future needs. Outcome of these discussions is, bring things together, which should never have been together, like employee and his/her reporting functionalities. This coupling leads to rigidity and software becomes unmanageable. “Change is inevitable” this is undeniable truth that history, software craftsmen and all resources on the internet have been saying and engineers have been ignoring.

To avoid these familiar situations in the future, every software professional should ask oneself following set of questions before proposing and finalizing an architecture and design

* How will my architecture/design handle change?

* How many components/modules will be affected if I’ve to replace a component with another. For instance, if we are changing message tracking functionality will alerting be impacted? How will I manage changes to tracking without affecting alerting?

* Am I designing for the exact customer requirements or am I over-designing assuming that customer will have additional requirements in future?

* How will I unit-test my code?

* If I’ve to make changes to my code do I have unit-tests to check if I broke any existing functionality?

* Am I taking any direct dependency on external components like using stored procedures in my class functions.

As long as we are designing for present and not future and ensuring that coupling is minimal, chaos can be managed and reduced. Take care of present, future will take care of itself This discussion is more relevant in the age of cloud computing than before because proliferation of new SaaS and PaaS services with competitive pricing models and the need to stay competitive has put us under tremendous pressure. We have to choose right services and replace existing services without major downtime and maintenance. If we are tightly coupled to existing services and their interfaces then any change in existing interfaces or strategy can potentially put one out-of-business.

To conclude, it is not anymore about right and perfect architectures and designs it is all about optimal architectures and designs, which can meet customer needs at that point of time and can co-evolve with customer business strategy. This is because, software is no more a tactical investment it is strategical advantage for organizations and strategies are influenced by many external factors which are beyond developers span of control and all developers can do is keep their designs open for changes.
