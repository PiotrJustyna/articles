- [orleans](#orleans)
  - [distributed systems](#distributed-systems)
  - [microservices and their challenges](#microservices-and-their-challenges)
  - [actors](#actors)
  - [virtual actors](#virtual-actors)
  - [actors in distributed stateless and stateful applications](#actors-in-distributed-stateless-and-stateful-applications)
  - [examples](#examples)
    - [example 1](#example-1)
    - [example 2](#example-2)
  - [lessons learned](#lessons-learned)
  - [references](#references)
    - [1 - Distributed systems: principles and paradigms.](#1---distributed-systems-principles-and-paradigms)
    - [2 - A Universal Modular Actor Formalism for Artificial Intelligence](#2---a-universal-modular-actor-formalism-for-artificial-intelligence)
    - [3 - Pattern: Microservice Architecture](#3---pattern-microservice-architecture)
  - [resources](#resources)

# orleans

This document outlines the purpose and the application of the Actor Model in Emailage's Email Risk Score API using Microsoft Orleans.

## distributed systems

*A distributed system is a system whose components are located on different networked computers, which communicate and coordinate their actions by passing messages to one another from any system.*[[1]](#1---distributed-systems-principles-and-paradigms)

A good example of a distributed system is a group of containerised applications:

* web APIs:
  * placed behind a load balancer
  * serving customer traffic
* supporting applications

where applications communicate with a group of data sources and with each other via message queues or direct messages, perform business decisions, report on their health, etc.

## microservices and their challenges

Microservices are often characterised by:

* their lack of internal state
* application instances being expendable - relatively simple to spin up and to tear down

which makes it fair to categorise microservices as distributed stateless applications.

While microservices allow engineers to easily scale distributed systems (both horizontally and vertically), they also present a number of challenges which become especially apparent when mission-critical speed is essential, namely:

* scaling - the ability to correctly respond to sudden changes in traffic volumes (it is not uncommon to see 10x traffic spikes in our APIs). Application instances should be quick to wake up, warm up and start working with a load balancer to serve traffic.
* request processing speed - the API should ideally be able to perform its tasks in predictable, relatively constant time.
* performance bottlenecks - if the application has a performance bottleneck (e.g. one task requiring excesive amounts of resources), scaling the application horizontally will just replicate those performance bottlenecks across the fleet of application instances.
* implementation speed - deployment, maintenance, inter-service communication, testing complexity [[3]](#3---pattern-microservice-architecture)

* [TODO] diagram - a fleet of web APIs - https://plantuml.com/component-diagram

## actors

The Actor Model is a mathematical theory of computation that treats “Actors” as the universal primitives of concurrent digital computation [[2]](#2---a-universal-modular-actor-formalism-for-artificial-intelligence). The model has been used both as a framework for a theoretical understanding of concurrency, and as the theoretical basis for several practical implementations of concurrent systems. Recently, the Actor Model gained new popularity with the advent of cloud-based computing where:

* massive, distributed parallelism
* ability to quickly analyse and understand distributed systems by humans

are of critical importance.

In the simplest terms, Actors are characterised by two features:

* they don't share state - each actor is an independent entity with its own memory
* they only communicate indirectly via messages allowing for greater decoupling

Using the Actor Model in distributed systems allows for breaking down complex, resource-intensive problems into more manageable, focused fragments and sharing them between the components of a distributed system.

## virtual actors

Orleans paper.

Orleans-specific terminology:

* grain - virtual actor (implemented as a distributed class)
* silo - actor host (implemented as a .NET process, hosted in a container)
* cluster - group of silos working together and sharing grains (implemented as a group of containers)

[TODO] cluster illustration

## actors in distributed stateless and stateful applications

The goal is to cleanly divide two aspects of every application:

* I/O - any database, service calls introducing uncertainty (latency, failure, etc.)
* state and pure (no side effects) state manipulations: mapping, filtering, business decisions, etc.

* API tasks are focused on what they should be focused - on handling customer requests
* stateful - potential for caching
* more granular horizontal scalability
* homogenous or heterogenous clusters
* all systems become much easier to reason about
* code becomes very reusable
* significant number of API tasks saved and can be dedicated to processing more customer requests
* distributed runtimes
* if the work is partitioned cleverly, Actors can give a tremendous performance boost to the API
* supporting applications can also use actors - better code reusability
* fast deployments (heterogenous silos), fast scaling (no warmup, internal load balancing and infrastructure state management)

## examples

### example 1

### example 2

## lessons learned

generous memory allocation
autoscaling criteria

## references

### 1 - Distributed systems: principles and paradigms.

Tanenbaum, Andrew S.; Steen, Maarten van (2002). Distributed systems: principles and paradigms. Upper Saddle River, NJ: Pearson Prentice Hall. ISBN 0-13-088893-1.

### 2 - A Universal Modular Actor Formalism for Artificial Intelligence

Hewitt, Carl; Bishop, Peter; Steiger, Richard (1973). "A Universal Modular Actor Formalism for Artificial Intelligence". IJCAI.

### 3 - Pattern: Microservice Architecture

https://microservices.io/patterns/microservices.html

## resources

* Orleans source code: https://github.com/dotnet/orleans
* Orleans documentation: https://dotnet.github.io/orleans/
* Coyote source code: https://github.com/microsoft/coyote/
* Actor Model of Computation: Scalable Robust Information Systems: https://arxiv.org/ftp/arxiv/papers/1008/1008.1459.pdf