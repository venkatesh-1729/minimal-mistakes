---
categories: Actor-System
---

> Actor model in computer science is a mathematical model of concurrent computation that treats "actors" as the universal primitives of concurrent computation. -- Wikipedia

# Actor Model
You can find libraries and frameworks implementing this model in many language [here](https://en.wikipedia.org/wiki/Actor_model#Actor_libraries_and_frameworks).

Some of the other concurrency models are : 
* CSP (communicating sequential processes).
* Threads.
* Futures.
* Coroutines.
* Petri Nets.

## Properties of Actor
* Actors are persistent.
* Encapsulate internal state.
* Actors are asynchronous.

## What can actors do ?
* Create new actors.
* Receive messages and in response to that:
    + Make local decisions (alter local state).
    + Perform arbitrary, side effecting action.
    + Send messages.
    + Responds to the sender 0 or more times.
    + Processes exactly one message a time.

> Don't communicate by sharing memory; instead share memory by communicating. -- Effective Go

## Properties of Communication
* No channels or intermediaries (like in CSP).
* Best effort delivary.
* At most one delivary.
* Messages can take arbitrary long to be delivered.
* No message ordering guarantees.

## Address
* Identifies an actor.
* May also represent a proxy/forwarder to an actor.
* Contains location and transport information.
* Location transparency.
* One address may represent many actors (pool).
* One actor can have many addresses.

## Handling Failures

### Supervision 
* The running state of an actor is monitored and managed by another actor (supervisor).

#### Properties of Supervision
* Constantly monitors running state of actor.
* Can perform actions based on state of actor.
* Supervision Trees.

## Transparent Lifecycle Management
* Address don't change during restarts.
* Mailboxes are persisted outside of actors.

## Actor Usecases
* Processing pipelines.
* Streaming data.
* Multi-user concurrency.
* Systems high uptime requirements.
* Application with shared state.

## Actor Anti-Usecases 
* Non concurrent systems.
* Performance critical applications.
* There is no mutable state.

## Drawbacks
* Too many actors.
* Testing.
* Debugging.
