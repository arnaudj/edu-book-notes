<!-- TOC -->

- 1. Misc
  - 1.1. Micro-frontends: anti-patterns | Luca Mezzalira | EnterpriseNG 2021
  - 1.2. Martin Fowler - MFE
    - 1.2.1. Cross application communication
    - 1.2.2. Testing

<!-- /TOC -->

# 1. Misc

Notes on micro-frontends from misc. sources.

## 1.1. Micro-frontends: anti-patterns | Luca Mezzalira | EnterpriseNG 2021
https://www.youtube.com/watch?v=gfKTFqjsT1M

A MFE is the technical representation of a business subdomain.

A MFE should:
- Be independent
- Have defined input/outputs
- Not be extensible
- Be domain aware: i.e, it's up to the MFE to decide what to display (based on inputs & API calls). Avoid leaks: other MFEs should not know about its internals.
- Minimize code shared with other subdomains.
- Be owned by a single team

Pitfalls to avoid
- Avoid coupling: use Event Emitter instead of having each MFE interacting with a global state.
- Unidirectional data flow
- Granularity: MFE vs Remote components: if 2 MFEs (mfe1#compA, mfe2#compB) consume the same API: maybe the domain is not well defined, and these 2 components should be part of the same MFE: a shared component library (mfe1#compA, mfe1#compB).


Rule of thumb: 
- a component: we allow different use cases by exposing multiple properties
- MFE: we encapsulate the logic, and communicate via events

## 1.2. Martin Fowler - MFE
https://martinfowler.com/articles/micro-frontends.html

### 1.2.1. Cross application communication

> we want our micro frontends to communicate by sending messages or events to each other, and avoid having any shared state

> Just like sharing a database across microservices, as soon as we share our data structures and domain models, we create massive amounts of coupling, and it becomes extremely difficult to make changes.

### 1.2.2. Testing
> If there are user journeys that span across micro frontends, then you could use functional testing to cover those, but keep the functional tests focussed on validating the integration of the frontends, and not the internal business logic of each micro frontend, which should have already been covered by unit tests.