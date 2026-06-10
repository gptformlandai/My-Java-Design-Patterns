# Java Design Patterns: Categories With Importance Meters

This file categorizes every repository pattern by primary pattern type and adds two study meters for each record.

## Meter Meaning

- Very High: study first; frequently useful and/or commonly discussed in senior interviews.
- High: important; strong practical value or recurring interview relevance.
- Medium: useful in specific contexts; learn after the high-priority set.
- Low: niche; understand the idea, but do not over-invest early.
- Optional: specialized or rare for typical Java/backend interview preparation.

## How to Use This File

For MAANG interview preparation, prioritize rows where the MAANG interview meter is Very High or High.
For professional software design growth, prioritize rows where the software usage meter is Very High or High.

## Creational and Object Construction Patterns

| Pattern | MAANG interview meter | Software usage meter |
|---|---|---|
| `builder` | Very High | Very High |
| `dependency-injection` | Very High | Very High |
| `factory` | Very High | Very High |
| `factory-method` | Very High | High |
| `singleton` | Very High | High |
| `abstract-factory` | High | High |
| `prototype` | High | Medium |
| `object-pool` | Medium | Medium |
| `step-builder` | Medium | Medium |
| `factory-kit` | Low | Low |
| `monostate` | Low | Low |
| `multiton` | Low | Low |

## Structural and Composition Patterns

| Pattern | MAANG interview meter | Software usage meter |
|---|---|---|
| `adapter` | Very High | Very High |
| `proxy` | Very High | Very High |
| `bridge` | High | Medium |
| `composite` | High | Medium |
| `decorator` | High | High |
| `facade` | High | High |
| `flyweight` | Medium | Medium |
| `virtual-proxy` | Medium | Medium |
| `composite-entity` | Low | Low |
| `composite-view` | Low | Low |
| `extension-objects` | Low | Low |
| `private-class-data` | Low | Low |
| `role-object` | Low | Low |
| `twin` | Optional | Optional |

## Behavioral and Interaction Patterns

| Pattern | MAANG interview meter | Software usage meter |
|---|---|---|
| `observer` | Very High | High |
| `strategy` | Very High | Very High |
| `chain-of-responsibility` | High | High |
| `command` | High | High |
| `iterator` | High | Very High |
| `state` | High | High |
| `template-method` | High | High |
| `visitor` | High | Medium |
| `callback` | Medium | High |
| `delegation` | Medium | High |
| `double-dispatch` | Medium | Medium |
| `filterer` | Medium | Medium |
| `interpreter` | Medium | Low |
| `mediator` | Medium | Medium |
| `memento` | Medium | Medium |
| `null-object` | Medium | Medium |
| `acyclic-visitor` | Low | Low |
| `collecting-parameter` | Low | Medium |
| `commander` | Low | Low |
| `servant` | Low | Low |
| `special-case` | Low | Medium |

## Architecture and Application Structure Patterns

| Pattern | MAANG interview meter | Software usage meter |
|---|---|---|
| `clean-architecture` | High | High |
| `command-query-responsibility-segregation` | High | High |
| `hexagonal-architecture` | High | High |
| `layered-architecture` | High | Very High |
| `service-layer` | High | Very High |
| `domain-model` | Medium | High |
| `monolithic-architecture` | Medium | High |

## Data Access, Persistence, and Transaction Patterns

| Pattern | MAANG interview meter | Software usage meter |
|---|---|---|
| `repository` | Very High | Very High |
| `data-access-object` | High | High |
| `lazy-loading` | High | High |
| `unit-of-work` | High | High |
| `data-mapper` | Medium | Medium |
| `identity-map` | Medium | Medium |
| `optimistic-offline-lock` | Medium | Medium |
| `single-table-inheritance` | Medium | Medium |
| `transaction-script` | Medium | Medium |
| `version-number` | Medium | Medium |
| `abstract-document` | Low | Low |
| `dao-factory` | Low | Low |
| `metadata-mapping` | Low | Low |
| `serialized-entity` | Low | Low |
| `serialized-lob` | Low | Low |
| `table-inheritance` | Low | Medium |
| `table-module` | Low | Low |

## Domain Modeling and Business Rule Patterns

| Pattern | MAANG interview meter | Software usage meter |
|---|---|---|
| `specification` | High | High |
| `value-object` | High | High |
| `money` | Medium | Medium |
| `parameter-object` | Medium | Medium |
| `notification` | Low | Medium |
| `property` | Low | Medium |

## Microservices and Distributed Systems Patterns

| Pattern | MAANG interview meter | Software usage meter |
|---|---|---|
| `leader-election` | Very High | High |
| `microservices-api-gateway` | Very High | High |
| `saga` | Very High | High |
| `sharding` | Very High | High |
| `anti-corruption-layer` | High | High |
| `microservices-distributed-tracing` | High | High |
| `microservices-idempotent-consumer` | High | High |
| `strangler` | High | High |
| `gateway` | Medium | High |
| `microservices-aggregrator` | Medium | Medium |
| `microservices-log-aggregation` | Medium | High |
| `microservices-self-registration` | Medium | Medium |
| `partial-response` | Medium | Medium |
| `server-session` | Medium | Medium |
| `tolerant-reader` | Medium | Medium |
| `ambassador` | Low | Medium |
| `client-session` | Low | Medium |
| `microservices-client-side-ui-composition` | Low | Medium |

## Reliability, Resilience, and Operations Patterns

| Pattern | MAANG interview meter | Software usage meter |
|---|---|---|
| `caching` | Very High | Very High |
| `circuit-breaker` | Very High | High |
| `rate-limiting-pattern` | Very High | Very High |
| `retry` | Very High | Very High |
| `throttling` | Very High | High |
| `backpressure` | High | High |
| `queue-based-load-leveling` | High | High |
| `feature-toggle` | Medium | High |
| `health-check` | Medium | High |

## Concurrency, Async, and Parallel Processing Patterns

| Pattern | MAANG interview meter | Software usage meter |
|---|---|---|
| `producer-consumer` | Very High | Very High |
| `thread-pool-executor` | Very High | Very High |
| `double-checked-locking` | High | Medium |
| `master-worker` | High | High |
| `reactor` | High | High |
| `active-object` | Medium | Medium |
| `actor-model` | Medium | Medium |
| `async-method-invocation` | Medium | High |
| `event-based-asynchronous` | Medium | High |
| `guarded-suspension` | Medium | Medium |
| `monitor` | Medium | Medium |
| `poison-pill` | Medium | Medium |
| `promise` | Medium | High |
| `balking` | Low | Low |
| `half-sync-half-async` | Low | Medium |
| `leader-followers` | Low | Low |
| `lockable-object` | Low | Medium |

## Messaging, Eventing, and Reactive Flow Patterns

| Pattern | MAANG interview meter | Software usage meter |
|---|---|---|
| `event-driven-architecture` | Very High | High |
| `publish-subscribe` | Very High | High |
| `event-queue` | High | High |
| `event-sourcing` | High | Medium |
| `event-aggregator` | Medium | Medium |
| `flux` | Medium | Medium |
| `data-bus` | Low | Medium |

## Web, Presentation, and UI Patterns

| Pattern | MAANG interview meter | Software usage meter |
|---|---|---|
| `model-view-controller` | Very High | Very High |
| `front-controller` | High | High |
| `intercepting-filter` | High | High |
| `component` | Medium | High |
| `model-view-presenter` | Medium | Medium |
| `model-view-viewmodel` | Medium | High |
| `bloc` | Low | Medium |
| `context-object` | Low | Medium |
| `converter` | Low | Medium |
| `model-view-intent` | Low | Medium |
| `page-controller` | Low | Medium |
| `presentation-model` | Low | Medium |
| `service-to-worker` | Low | Medium |
| `templateview` | Low | Medium |
| `view-helper` | Low | Medium |

## Functional and Pipeline Processing Patterns

| Pattern | MAANG interview meter | Software usage meter |
|---|---|---|
| `map-reduce` | Very High | High |
| `fanout-fanin` | High | High |
| `pipeline` | High | High |
| `collection-pipeline` | Medium | High |
| `combinator` | Medium | Medium |
| `currying` | Medium | Medium |
| `function-composition` | Medium | High |
| `monad` | Low | Low |
| `trampoline` | Low | Low |

## Testing Patterns

| Pattern | MAANG interview meter | Software usage meter |
|---|---|---|
| `arrange-act-assert` | Medium | High |
| `page-object` | Medium | High |
| `service-stub` | Medium | Medium |
| `object-mother` | Low | Medium |

## Performance, Game, and Low-Level Optimization Patterns

| Pattern | MAANG interview meter | Software usage meter |
|---|---|---|
| `data-locality` | Medium | Medium |
| `spatial-partition` | Medium | Low |
| `bytecode` | Low | Medium |
| `dirty-flag` | Low | Medium |
| `double-buffer` | Low | Low |
| `game-loop` | Optional | Low |
| `subclass-sandbox` | Optional | Optional |
| `update-method` | Optional | Optional |

## Language, JVM, and Idiom Patterns

| Pattern | MAANG interview meter | Software usage meter |
|---|---|---|
| `dynamic-proxy` | Medium | High |
| `execute-around` | Medium | High |
| `fluent-interface` | Medium | High |
| `marker-interface` | Medium | Medium |
| `resource-acquisition-is-initialization` | Medium | Medium |
| `curiously-recurring-template-pattern` | Low | Low |
| `separated-interface` | Low | Medium |
| `type-object` | Low | Low |
| `mute-idiom` | Optional | Optional |

## Enterprise Application Patterns

| Pattern | MAANG interview meter | Software usage meter |
|---|---|---|
| `data-transfer-object` | Very High | Very High |
| `registry` | Medium | Medium |
| `service-locator` | Medium | Medium |
| `business-delegate` | Low | Medium |
| `session-facade` | Low | Medium |

## Validation

- Total modules in repository: 178
- Total categorized: 178
- Missing entries: 0
- Duplicate entries: 0
- Extra entries: 0