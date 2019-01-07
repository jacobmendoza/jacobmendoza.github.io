---
layout: post
title:  "Events, in two contexts"
date:   2018-05-25 17:40:57 +0000
categories: ARTICLE RAILS EVENT-DRIVEN EVENTSOURCING
---
Recently I have been involved in the design of a system to capture and store changes in the state of some entities in our software.

We wanted to have a log that was a real reflection of the history of the entity so we could prove its validity and point exactly to all the transitions that led to it. We also needed to make sure that some subsystems and services were notified of some of these events.

The conversations happened mainly around two concepts that have the event in common: **Event Sourcing** (the internal state of entities is rebuilt from a list of events that capture every change in it) and **Event-driven architecture** (the architectural pattern whereby applications are built producing and consuming events).

Unfortunately, these two topics are extremely broad, they have things in common but they are not the same. These are only a few notes that I took while working on it.

Please, if you feel that there is something wrong or not accurate enough, get in touch!.

# Application Integration Options

The need to integrate different applications arose quite quickly in a still non-connected world. Different vendors and solutions that had to work over the same data.

Now we live in an incredibly distributed world. Not only internet, that opens a new world of interactions but also the way we design our applications: Different processes, micro-services, etc. It is so normal and present in our work every day, that our approach to it is much more implicit.

A normal web application is in a lot of cases based on a frontend part for customers, frontend for administrators, and a backend that holds the operations. They’re seen as a whole, but they are different processes (one running in a server, the others in the client browser) that need to communicate. And sometimes it’s more complicated than this. We need to connect different languages, services, APIs and environments.

There are four main approaches to connect applications. For the experienced programmer, some of them will quickly awake their intuitions about what is good and what it isn’t. In any case, it is a good exercise to review them to analyze what each of one gives us or takes away.  

## 1. File transfer

One application writes a file in a format that another later will be able to read. Applications will need to agree on how to exchange this information, names, locations, format, and timing. One needs to trigger the update in the destination application.

## 2. Shared database

The applications agree on some specific schema, and they all access to the same database. Consistency is ensured between different applications as we have a transaction management system that will deal in almost all scenarios with problems of this nature.

This solution has also problems. In growing organizations, it becomes harder to agree on schemas suitable for all needs and delays start to spread between deliveries of different teams. Also, and this is quite important, **this method guarantees the exchange of data, but not behavior**. When we need to share behavior (a need that arises quickly) this method will force us to re-implement the functionality, probably in different languages and technologies.

## 3. Remote procedure invocation

This would give us a mechanism to invoke functionality in a different application, it could be to trigger a side effect on the destination system or to read a piece of data that has been computed for us. The connected systems now can have different schemas underneath, and the only thing that they share is the contracts of the functionalities that can be invoked. The applications are, therefore, encapsulated.
In the era of web services and APIs, this is broadly used. We are extremely used to it, and for good reasons. It is in a lot of different cases, the default approach to application integration.

Of course, it is not without problems. When the procedure invocation is remote, the performance hit is bigger. Sometimes operations need to be carefully designed to allow bigger granularities and to reduce I/O.

Also, the systems are, even with this method, still coupled. We are leaving out of the picture the internal representation of the systems, which is good, but the source and destination are directly connected via some location information (where it is) and whatever representation they are using to communicate (the format of the contract, the application layer, etc). Whenever the source system needs to do its operation, the destination needs to be available to do its part. The communication is synchronous. We have a coupling also that refers to timing.

## 4. Messaging

Decouples completely the application that generates the event and the one receiving it, allowing to multiple transformations in the middle. One is only coupled with the representation of the event itself and forces everyone involved to think in an asynchronous way. Although this depends on the method, generally speaking, if the consumer is not available the events can be consumed later.

Testing and debugging and infrastructure is or can be more complex.

# Modeling Events

Definition and some good ideas and practices can be applied to modeling events in both contexts, Event Sourcing, and Event-driven architectures.

The event is an immutable piece of data that signals that something relevant occurred in the context of the domain, and domain entity that we are describing. They are characterized by a name and a payload.

Ideally, the name tries to capture the real intent behind the operation, in other words, what was happening described with words relevant for the domain. As the event is just a piece of data, the more specific we are, the easier things will get as our system grows. Names itself can carry a lot of meaning. Just to put one really quick example, if one is working in a banking domain, and a mortgage has been accepted, we could have:

```
  MortgageModified  { amount: x, deal_date: y, accepted: true } //BAD
MortgageAccepted { amount: x, deal_date: y } //GOOD
```

Usually, anything that remembers to CRUD terminology should make us think that something better is available. **Events too broad with coarse payloads usually mean that consumers will need to infer the domain meaning by themselves**. If that is the case, it is likely that it will happen in several places leading to duplication.

# The event in Event Sourcing (Domain event)

Entities describe their state as the summation of a stream of events that describe all the transitions. Each event describes one change.

```
f(state, event) => state
```

Domain events only contain information that can’t be derived from past events and that is needed to reconstruct the state. Although some extra information that captures the context in which the event happened may be added, the idea is to be strict and add just the information that characterizes the fact.

The payload should not be designed in a way that makes easier computing the final state. The complexity for that computation belongs to the function. **These events can be seen as messages that we send to the aggregate so it can reconstruct itself**, so some of the problems related to messaging (versioning, information capture) can appear. To understand better how schema evolution works, “Enterprise Integration Patterns” (Hohpe, Wolf) is a good start.

It is reasonable to feel worried about its design and the number of details that one should include, for example. It is very similar to the analysis that is done for use-cases analysis (actors and use cases must be identified to understand what is the fact that has just happened).  In this analysis, we also try to make explicit concepts that could have been previously left as implicit. If you are redesigning a system with a previous implementation in place, it is ok to create events for business facts that did not have before a direct reflection on the data.

# The event in Event-driven architectures

In this case, the event is the basic piece of communication of applications or processes that want to exchange data. The main reason for the event to exist is so it can be consumed by others.

The payload of the event can be manually arranged in the code of your application when something happens and sent to a channel, or it could be an enriched version of the domain event described previously.

Whereas in the domain events we were trying to be careful with the amount of information that we were inserting and the main goal was to reconstruct the aggregate state, in this case, communication with external processes or applications is what we are after. Any piece of information that can be needed by destinations downstream can be inserted and it makes sense for consumers to lead the design of the API/contracts.

On the other hand and although I am not really sure if it applies in strict terms, the Interface Segregation Principle must be also taken into account and make sure we do not make clients depend on the information they do not use. Once the data is there, clients can rely on it. So different granularities in different events must be carefully reviewed to avoid this.

**This information is decoupling the processes involved and becomes part of an architectural boundary**. We provide data that the destination process won’t have to find by itself. Therefore, it does not need to know how to or from where, retrieve it.

There are cases where this is not enough, and the destination process needs more data than the one contained in the event. Some events just trigger bigger computations. No matter what method is finally used, it will be wise to get this extra information without accessing to internal details of the sender, like the database. This may sound evident, but it deserves a sentence. Subsequent communications must be done via well-established contracts. RPC is a good option.
