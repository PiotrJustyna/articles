# readme

This article will cover building web apis characterized by:

* heavy use of i/o (implemented as asynchronous `Task`s)
* high throughput
* fast horizontally scaling

using the Orleans framework. It will also attempt to compare features like:

* request handling in unpredictable traffic load scenarios
* horizontal scaling speed
* horizontal scaling techniques

of web apis where heavy tasking (i/o) is done in api runtime versus those where heavy i/o is delegate to external runtimes (Orleans virtual actors).

## scenarios

### scenario 1

#### participants

* main web api
* sub web api

On every incoming request to the main api, it creates an artificial workload of N\* tasks and makes a call to the sub web api which also creates identical artificial workload of N\* tasks. In total, we should expect 2N + 1 tasks in the web api group combined (+1 as there needs to be one task allocated to communicate between the main and the sub api).

\*this is configurable

#### conditions

* horizontal scaling:
    * main web api scales in response to the volume of requests it receives
    * sub api scales in response to the volume of requests it receives

Problem: in such scaling conditions there will be a delay between the main api and the sub api scale. Ideally, the sub api should start scaling as the main api starts scaling.

### scenario 2

#### participants

* main web api
* sub web api
* orleans cluster

The main difference between scenario 2 and scenario 1 is that in scenario 2 we are introducing a cluster of Orleans silos which will be responsible for processing the artificial workloads both apis produce.

#### conditions

* horizontal scaling:
    * both apis scale in response to the volume of requests the main api receives which addresses the delay problem outlined in scenario 1.

## measurements

| test | scenario 1 | scenario 2 |
| --- | --- | --- |
| 1. max per-container throughput where p99 is under SLA | ? | ? |
