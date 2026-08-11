---
type: arquitetura
description: "Software should handle errors gracefully. Designing for failure yields a self-healing infrastructure that acts with the maturity that is ..."
tags:
  - notes/arquitetura
---

# Design for Failure

## What is this?
Software should handle errors gracefully.

Designing for failure yields a self-healing infrastructure that acts with the maturity that is expected of recent workloads. [^1]: ## Key factors
### Disposability
Fast startup and graceful shutdown.

Maybe to think the workload as a /"Crash-only Software"/[^3] would be nice here. This is also the motto of Erlang[^4][^5] and can be be applied outside the language[^6]. This is a form of /Defensive Programming/[^7].
### Logs (as an event streaming)
For understanding when the software failed and why.
### Dev/Prod parity
Keep development and productions environment as similar as possible.

The dev/prod parity is also a factor of the **The Twelve-Factor App**
## Handling failure on other services
### Circuit Breakers
/TODO: Write a note about CB/

Circuit Breakers are important to allow for software to keep running on a degraded state or gracefully recovers from
### Retries
Errors 5xx and throttling errors should be retried[^2].

[^1]: [Design for Failure - Running Containerized Microservices on AWS](https://docs.aws.amazon.com/whitepapers/latest/running-containerized-microservices/design-for-failure.html)
[^2]: [Error retries and exponential backoff in AWS - AWS General Reference](https://docs.aws.amazon.com/general/latest/gr/api-retries.html)
[^3]: [Crash-only software: More than meets the eye {LWN.net}](https://lwn.net/Articles/191059/)
[^4]: [Erlang: The Programming Language That Quietly Powers WhatsApp and WeChat - DE...](https://dev.to/serokell/is-erlang-worth-learning-in-2021-28h2)
[^5]: [Let It Crash](https://verraes.net/2014/12/erlang-let-it-crash/)
[^6]: [The Let It Crash Philosophy Outside Erlang - Stratus3D](http://stratus3d.com/blog/2020/01/20/applying-the-let-it-crash-philosophy-outside-erlang/)
[^7]: [Defensive programming - Wikipedia](https://en.wikipedia.org/wiki/Defensive_programming)
