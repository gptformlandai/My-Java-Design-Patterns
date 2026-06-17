# Optional Read

This folder tracks every categorized pattern that is not already covered by the main learning modules.

Use this as a second-pass study map. These patterns are mostly Low, Medium-specialized, or Optional for MAANG/backend preparation. They are still useful to recognize, but they should not interrupt the main high-value sequence.

## How This File Is Ordered

- Order follows [PATTERN_TYPES_CATEGORIZATION.md](../PATTERN_TYPES_CATEGORIZATION.md) from first to last.
- Already-covered main modules are excluded.
- Each entry includes the meters, source module, and focused subtopics to study.

## Quick Index

| # | Pattern | Category | MAANG | Software |
|---|---|---|---|---|
| 1 | [factory-kit](#1-factory-kit) | Creational and Object Construction | Low | Low |
| 2 | [monostate](#2-monostate) | Creational and Object Construction | Low | Low |
| 3 | [multiton](#3-multiton) | Creational and Object Construction | Low | Low |
| 4 | [composite-entity](#4-composite-entity) | Structural and Composition | Low | Low |
| 5 | [composite-view](#5-composite-view) | Structural and Composition | Low | Low |
| 6 | [extension-objects](#6-extension-objects) | Structural and Composition | Low | Low |
| 7 | [private-class-data](#7-private-class-data) | Structural and Composition | Low | Low |
| 8 | [role-object](#8-role-object) | Structural and Composition | Low | Low |
| 9 | [twin](#9-twin) | Structural and Composition | Optional | Optional |
| 10 | [acyclic-visitor](#10-acyclic-visitor) | Behavioral and Interaction | Low | Low |
| 11 | [collecting-parameter](#11-collecting-parameter) | Behavioral and Interaction | Low | Medium |
| 12 | [commander](#12-commander) | Behavioral and Interaction | Low | Low |
| 13 | [servant](#13-servant) | Behavioral and Interaction | Low | Low |
| 14 | [special-case](#14-special-case) | Behavioral and Interaction | Low | Medium |
| 15 | [abstract-document](#15-abstract-document) | Data Access, Persistence, and Transaction | Low | Low |
| 16 | [dao-factory](#16-dao-factory) | Data Access, Persistence, and Transaction | Low | Low |
| 17 | [metadata-mapping](#17-metadata-mapping) | Data Access, Persistence, and Transaction | Low | Low |
| 18 | [serialized-entity](#18-serialized-entity) | Data Access, Persistence, and Transaction | Low | Low |
| 19 | [serialized-lob](#19-serialized-lob) | Data Access, Persistence, and Transaction | Low | Low |
| 20 | [table-inheritance](#20-table-inheritance) | Data Access, Persistence, and Transaction | Low | Medium |
| 21 | [table-module](#21-table-module) | Data Access, Persistence, and Transaction | Low | Low |
| 22 | [notification](#22-notification) | Domain Modeling and Business Rule | Low | Medium |
| 23 | [property](#23-property) | Domain Modeling and Business Rule | Low | Medium |
| 24 | [ambassador](#24-ambassador) | Microservices and Distributed Systems | Low | Medium |
| 25 | [client-session](#25-client-session) | Microservices and Distributed Systems | Low | Medium |
| 26 | [microservices-client-side-ui-composition](#26-microservices-client-side-ui-composition) | Microservices and Distributed Systems | Low | Medium |
| 27 | [balking](#27-balking) | Concurrency, Async, and Parallel Processing | Low | Low |
| 28 | [half-sync-half-async](#28-half-sync-half-async) | Concurrency, Async, and Parallel Processing | Low | Medium |
| 29 | [leader-followers](#29-leader-followers) | Concurrency, Async, and Parallel Processing | Low | Low |
| 30 | [lockable-object](#30-lockable-object) | Concurrency, Async, and Parallel Processing | Low | Medium |
| 31 | [data-bus](#31-data-bus) | Messaging, Eventing, and Reactive Flow | Low | Medium |
| 32 | [bloc](#32-bloc) | Web, Presentation, and UI | Low | Medium |
| 33 | [context-object](#33-context-object) | Web, Presentation, and UI | Low | Medium |
| 34 | [converter](#34-converter) | Web, Presentation, and UI | Low | Medium |
| 35 | [model-view-intent](#35-model-view-intent) | Web, Presentation, and UI | Low | Medium |
| 36 | [page-controller](#36-page-controller) | Web, Presentation, and UI | Low | Medium |
| 37 | [presentation-model](#37-presentation-model) | Web, Presentation, and UI | Low | Medium |
| 38 | [service-to-worker](#38-service-to-worker) | Web, Presentation, and UI | Low | Medium |
| 39 | [templateview](#39-templateview) | Web, Presentation, and UI | Low | Medium |
| 40 | [view-helper](#40-view-helper) | Web, Presentation, and UI | Low | Medium |
| 41 | [monad](#41-monad) | Functional and Pipeline Processing | Low | Low |
| 42 | [trampoline](#42-trampoline) | Functional and Pipeline Processing | Low | Low |
| 43 | [object-mother](#43-object-mother) | Testing | Low | Medium |
| 44 | [bytecode](#44-bytecode) | Performance, Game, and Low-Level Optimization | Low | Medium |
| 45 | [dirty-flag](#45-dirty-flag) | Performance, Game, and Low-Level Optimization | Low | Medium |
| 46 | [double-buffer](#46-double-buffer) | Performance, Game, and Low-Level Optimization | Low | Low |
| 47 | [game-loop](#47-game-loop) | Performance, Game, and Low-Level Optimization | Optional | Low |
| 48 | [subclass-sandbox](#48-subclass-sandbox) | Performance, Game, and Low-Level Optimization | Optional | Optional |
| 49 | [update-method](#49-update-method) | Performance, Game, and Low-Level Optimization | Optional | Optional |
| 50 | [curiously-recurring-template-pattern](#50-curiously-recurring-template-pattern) | Language, JVM, and Idiom | Low | Low |
| 51 | [separated-interface](#51-separated-interface) | Language, JVM, and Idiom | Low | Medium |
| 52 | [type-object](#52-type-object) | Language, JVM, and Idiom | Low | Low |
| 53 | [mute-idiom](#53-mute-idiom) | Language, JVM, and Idiom | Optional | Optional |
| 54 | [business-delegate](#54-business-delegate) | Enterprise Application | Low | Medium |
| 55 | [session-facade](#55-session-facade) | Enterprise Application | Low | Medium |

---

## 1. factory-kit

Category: Creational and Object Construction Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [factory-kit](../../github-repo/factory-kit)

Subtopics to study:
- Factory as a map of builders or suppliers.
- Adding new product creation without changing client code.
- Difference from Factory Method and Abstract Factory.
- When supplier registration is cleaner than large switch statements.
- Risk of hiding construction rules behind string or enum keys.

---

## 2. monostate

Category: Creational and Object Construction Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [monostate](../../github-repo/monostate)

Subtopics to study:
- Shared static state across multiple object instances.
- Difference between Monostate and Singleton.
- Why object identity differs while state stays shared.
- Thread-safety concerns around static mutable fields.
- Why this is rarely preferred in modern application design.

---

## 3. multiton

Category: Creational and Object Construction Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [multiton](../../github-repo/multiton)

Subtopics to study:
- One controlled instance per key.
- Difference from Singleton, Registry, and Object Pool.
- Enum-keyed or map-keyed instance management.
- Lifecycle and eviction concerns.
- When dependency injection or configuration is cleaner.

---

## 4. composite-entity

Category: Structural and Composition Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [composite-entity](../../github-repo/composite-entity)

Subtopics to study:
- Coarse-grained object composed of dependent objects.
- Enterprise Java origin and entity aggregation.
- Reducing remote calls through bundled entity access.
- Relationship to DTO and Composite.
- Why modern persistence frameworks often hide this need.

---

## 5. composite-view

Category: Structural and Composition Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [composite-view](../../github-repo/composite-view)

Subtopics to study:
- Building a page from reusable view fragments.
- Header/body/footer/template composition.
- Difference from Composite, Template View, and Component.
- Server-side rendering layout patterns.
- When component frameworks make this pattern implicit.

---

## 6. extension-objects

Category: Structural and Composition Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [extension-objects](../../github-repo/extension-objects)

Subtopics to study:
- Adding optional behavior without changing a core class.
- Asking an object for supported extension interfaces.
- Difference from Decorator, Adapter, and Role Object.
- Plugin-like capability discovery.
- Risk of runtime casts and unclear extension contracts.

---

## 7. private-class-data

Category: Structural and Composition Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [private-class-data](../../github-repo/private-class-data)

Subtopics to study:
- Moving immutable or sensitive data into a private data holder.
- Reducing accidental mutation.
- Encapsulation of constructor parameters.
- Difference from Value Object and immutable records.
- Why modern Java records/final fields often solve similar problems.

---

## 8. role-object

Category: Structural and Composition Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [role-object](../../github-repo/role-object)

Subtopics to study:
- Object behavior changes based on attached roles.
- Dynamic role assignment and removal.
- Difference from State, Strategy, and Decorator.
- Modeling users/entities with multiple responsibilities.
- Complexity of role lookup and behavior coordination.

---

## 9. twin

Category: Structural and Composition Patterns  
MAANG interview meter: Optional  
Software usage meter: Optional  
Repository module: [twin](../../github-repo/twin)

Subtopics to study:
- Simulating multiple inheritance by pairing cooperating objects.
- Bidirectional references between twins.
- Difference from Adapter and Bridge.
- When language limitations motivate the pattern.
- Maintenance risk from tightly coupled twin objects.

---

## 10. acyclic-visitor

Category: Behavioral and Interaction Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [acyclic-visitor](../../github-repo/acyclic-visitor)

Subtopics to study:
- Visitor variant that avoids cyclic dependencies.
- Visitor interfaces split by visitable type.
- Difference from classic Visitor.
- Compile-time dependency reduction.
- Trade-off: more interfaces and runtime type checks.

---

## 11. collecting-parameter

Category: Behavioral and Interaction Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [collecting-parameter](../../github-repo/collecting-parameter)

Subtopics to study:
- Passing a mutable collector through recursive or multi-step logic.
- Accumulating validation errors, results, or diagnostics.
- Difference from return aggregation and Notification.
- When a collector clarifies multi-output flows.
- Risk of hidden side effects if overused.

---

## 12. commander

Category: Behavioral and Interaction Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [commander](../../github-repo/commander)

Subtopics to study:
- Encapsulating command invocation and execution context.
- Difference from Command pattern.
- Central coordinator for issuing operations.
- Request routing and operational control.
- Why standard Command is usually the more important interview pattern.

---

## 13. servant

Category: Behavioral and Interaction Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [servant](../../github-repo/servant)

Subtopics to study:
- Moving common behavior into a helper that acts on many objects.
- Difference from Utility class, Strategy, and Visitor.
- Servant operates on serviced objects through a shared interface.
- Avoiding duplicated behavior across unrelated classes.
- Risk of an anemic helper that knows too much.

---

## 14. special-case

Category: Behavioral and Interaction Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [special-case](../../github-repo/special-case)

Subtopics to study:
- Representing exceptional cases as ordinary objects.
- Difference from Null Object.
- Avoiding repeated conditional checks.
- Modeling unknown, missing, guest, or default cases.
- Keeping special behavior explicit and safe.

---

## 15. abstract-document

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [abstract-document](../../github-repo/abstract-document)

Subtopics to study:
- Flexible object model backed by key-value properties.
- Trait-like interfaces over dynamic maps.
- Difference from Property pattern and document databases.
- Handling evolving schemas.
- Type-safety trade-offs.

---

## 16. dao-factory

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [dao-factory](../../github-repo/dao-factory)

Subtopics to study:
- Factory responsible for creating DAO implementations.
- Switching persistence technology behind a common DAO API.
- Difference from Abstract Factory and DAO.
- Environment-specific DAO wiring.
- Why DI containers often replace this in modern Java.

---

## 17. metadata-mapping

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [metadata-mapping](../../github-repo/metadata-mapping)

Subtopics to study:
- Mapping object fields to database schema using metadata.
- Runtime mapping tables, annotations, or descriptors.
- Difference from Data Mapper.
- Schema evolution and metadata maintenance.
- Reflection and performance considerations.

---

## 18. serialized-entity

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [serialized-entity](../../github-repo/serialized-entity)

Subtopics to study:
- Storing an entire entity as serialized data.
- Snapshot persistence and object graph storage.
- Query limitations when data is opaque.
- Versioning serialized formats.
- When JSON/document storage is better modeled explicitly.

---

## 19. serialized-lob

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [serialized-lob](../../github-repo/serialized-lob)

Subtopics to study:
- Storing serialized data in a large object column.
- Difference from Serialized Entity.
- Handling binary or JSON payloads in relational databases.
- Query, migration, and indexing trade-offs.
- When to split fields into normal columns.

---

## 20. table-inheritance

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [table-inheritance](../../github-repo/table-inheritance)

Subtopics to study:
- Mapping inheritance hierarchies to database tables.
- Class table, concrete table, and single table approaches.
- Difference from Single Table Inheritance.
- Join cost and schema complexity.
- ORM configuration implications.

---

## 21. table-module

Category: Data Access, Persistence, and Transaction Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [table-module](../../github-repo/table-module)

Subtopics to study:
- One business logic class per database table.
- Working with record sets rather than rich domain objects.
- Difference from Domain Model and Transaction Script.
- Enterprise application architecture origin.
- When service classes become table-centric.

---

## 22. notification

Category: Domain Modeling and Business Rule Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [notification](../../github-repo/notification)

Subtopics to study:
- Collecting multiple validation or domain errors.
- Returning structured error information instead of throwing immediately.
- Difference from Collecting Parameter.
- UI/API validation response shaping.
- Avoiding exceptions for expected validation failures.

---

## 23. property

Category: Domain Modeling and Business Rule Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [property](../../github-repo/property)

Subtopics to study:
- Dynamic properties attached to objects.
- Flexible attributes when schema varies.
- Difference from Abstract Document.
- Type validation and property metadata.
- Risk of losing compile-time safety.

---

## 24. ambassador

Category: Microservices and Distributed Systems Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [ambassador](../../github-repo/ambassador)

Subtopics to study:
- Helper service deployed beside an application.
- Offloading connectivity, retries, monitoring, or protocol logic.
- Difference from Sidecar and Proxy.
- Service mesh relationship.
- Operational complexity and deployment coupling.

---

## 25. client-session

Category: Microservices and Distributed Systems Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [client-session](../../github-repo/client-session)

Subtopics to study:
- Keeping session state on the client.
- Cookies, JWTs, local storage, and signed tokens.
- Difference from Server Session.
- Security, tampering, expiry, and revocation.
- Stateless server scaling benefits.

---

## 26. microservices-client-side-ui-composition

Category: Microservices and Distributed Systems Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [microservices-client-side-ui-composition](../../github-repo/microservices-client-side-ui-composition)

Subtopics to study:
- Browser/mobile client composes UI from multiple services.
- Difference from API Gateway and Backend-for-Frontend.
- Latency and error handling across many calls.
- Ownership of UI fragments by service teams.
- When client complexity becomes too high.

---

## 27. balking

Category: Concurrency, Async, and Parallel Processing Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [balking](../../github-repo/balking)

Subtopics to study:
- Refusing to execute when object state is not appropriate.
- Guarded state checks before work.
- Difference from Guarded Suspension.
- Idempotency and state-machine safety.
- Concurrency visibility and synchronization.

---

## 28. half-sync-half-async

Category: Concurrency, Async, and Parallel Processing Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [half-sync-half-async](../../github-repo/half-sync-half-async)

Subtopics to study:
- Separating async I/O layer from synchronous processing layer.
- Queue boundary between async and sync halves.
- Difference from Reactor and Producer-Consumer.
- Backpressure and thread pool sizing.
- Useful in network servers and middleware.

---

## 29. leader-followers

Category: Concurrency, Async, and Parallel Processing Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [leader-followers](../../github-repo/leader-followers)

Subtopics to study:
- Thread pool where one leader waits for events.
- Followers take over leadership after event handling.
- Difference from thread-per-request and Reactor.
- Reducing handoff overhead in event demultiplexing.
- Complexity of coordination.

---

## 30. lockable-object

Category: Concurrency, Async, and Parallel Processing Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [lockable-object](../../github-repo/lockable-object)

Subtopics to study:
- Object controls its own lock state.
- Encapsulating lock/unlock behavior.
- Difference from Monitor and RAII lock guards.
- Preventing illegal mutation while locked.
- Deadlock and ownership concerns.

---

## 31. data-bus

Category: Messaging, Eventing, and Reactive Flow Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [data-bus](../../github-repo/data-bus)

Subtopics to study:
- Central bus for publishing data/messages inside an app.
- Members subscribe to receive bus data.
- Difference from Publish-Subscribe and Event Bus.
- Loose coupling versus hidden global message flow.
- Ordering and delivery expectations.

---

## 32. bloc

Category: Web, Presentation, and UI Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [bloc](../../github-repo/bloc)

Subtopics to study:
- Business Logic Component style for UI state.
- Events in, states out.
- Difference from MVVM, Flux, and MVI.
- Common in Flutter/Dart architecture.
- Managing async UI workflows predictably.

---

## 33. context-object

Category: Web, Presentation, and UI Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [context-object](../../github-repo/context-object)

Subtopics to study:
- Encapsulating request or execution context.
- Avoiding long parameter lists across layers.
- Difference from Parameter Object.
- Request metadata, user identity, locale, correlation ID.
- Risk of becoming a bag of unrelated state.

---

## 34. converter

Category: Web, Presentation, and UI Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [converter](../../github-repo/converter)

Subtopics to study:
- Converting between data representations.
- DTO/entity/view model conversion.
- Difference from Adapter and Mapper.
- One-way versus two-way conversion.
- Avoiding conversion logic scattered across controllers.

---

## 35. model-view-intent

Category: Web, Presentation, and UI Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [model-view-intent](../../github-repo/model-view-intent)

Subtopics to study:
- UI flow as intent -> model -> view.
- Difference from MVC, MVVM, and Flux.
- Unidirectional UI state management.
- Immutable state and render loops.
- Handling async effects cleanly.

---

## 36. page-controller

Category: Web, Presentation, and UI Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [page-controller](../../github-repo/page-controller)

Subtopics to study:
- One controller per page or route.
- Difference from Front Controller and MVC controller.
- Simple server-side web applications.
- Request handling and view selection.
- Scaling problems as page count grows.

---

## 37. presentation-model

Category: Web, Presentation, and UI Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [presentation-model](../../github-repo/presentation-model)

Subtopics to study:
- Model of UI state independent from widgets.
- Difference from MVVM ViewModel.
- Testable presentation logic.
- Synchronizing presentation model and view.
- Useful in desktop and complex UI forms.

---

## 38. service-to-worker

Category: Web, Presentation, and UI Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [service-to-worker](../../github-repo/service-to-worker)

Subtopics to study:
- Front controller delegates to services before view rendering.
- Combines dispatcher and view helper ideas.
- Difference from Front Controller.
- Server-side request processing pipeline.
- Legacy Java web architecture context.

---

## 39. templateview

Category: Web, Presentation, and UI Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [templateview](../../github-repo/templateview)

Subtopics to study:
- Rendering dynamic pages through templates.
- Separating markup from business logic.
- Difference from View Helper and Composite View.
- Template engines and server-side rendering.
- Keeping templates presentation-focused.

---

## 40. view-helper

Category: Web, Presentation, and UI Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [view-helper](../../github-repo/view-helper)

Subtopics to study:
- Helpers provide presentation logic to views.
- Avoiding business logic in templates.
- Difference from Template View and MVC.
- Formatting, lookup, and UI-specific calculations.
- Risk of helpers becoming service layers.

---

## 41. monad

Category: Functional and Pipeline Processing Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [monad](../../github-repo/monad)

Subtopics to study:
- Wrapping values with context.
- `map`, `flatMap`, and chaining computations.
- Optional, Either, Future-like examples.
- Error handling without exceptions.
- Why Java developers usually encounter this through libraries.

---

## 42. trampoline

Category: Functional and Pipeline Processing Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [trampoline](../../github-repo/trampoline)

Subtopics to study:
- Turning recursive calls into iterative steps.
- Avoiding stack overflow in languages without tail-call optimization.
- Difference from normal recursion.
- Continuation-style execution.
- Performance and readability trade-offs.

---

## 43. object-mother

Category: Testing Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [object-mother](../../github-repo/object-mother)

Subtopics to study:
- Central factory for common test objects.
- Difference from Test Data Builder.
- Reducing duplicate fixture setup.
- Risk of shared test data becoming too broad.
- Keeping test intent readable.

---

## 44. bytecode

Category: Performance, Game, and Low-Level Optimization Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [bytecode](../../github-repo/bytecode)

Subtopics to study:
- Encoding behavior as instructions interpreted by a VM.
- Difference from Interpreter.
- Scriptable behavior and game logic.
- Instruction set design.
- Debuggability and safety of custom bytecode.

---

## 45. dirty-flag

Category: Performance, Game, and Low-Level Optimization Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [dirty-flag](../../github-repo/dirty-flag)

Subtopics to study:
- Marking derived data as stale.
- Recomputing only when needed.
- Difference from caching.
- Scene graph transforms, UI layout, and persistence dirty checking.
- Resetting dirty state correctly.

---

## 46. double-buffer

Category: Performance, Game, and Low-Level Optimization Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [double-buffer](../../github-repo/double-buffer)

Subtopics to study:
- Writing to one buffer while reading from another.
- Swapping buffers atomically.
- Avoiding tearing or partial updates.
- Graphics/game rendering and concurrent state snapshots.
- Memory cost of duplicate buffers.

---

## 47. game-loop

Category: Performance, Game, and Low-Level Optimization Patterns  
MAANG interview meter: Optional  
Software usage meter: Low  
Repository module: [game-loop](../../github-repo/game-loop)

Subtopics to study:
- Continuous loop for processing input, update, and render.
- Fixed timestep versus variable timestep.
- Frame rate, lag, and catch-up updates.
- Difference from event-driven UI loops.
- Keeping simulation deterministic.

---

## 48. subclass-sandbox

Category: Performance, Game, and Low-Level Optimization Patterns  
MAANG interview meter: Optional  
Software usage meter: Optional  
Repository module: [subclass-sandbox](../../github-repo/subclass-sandbox)

Subtopics to study:
- Base class exposes protected operations for subclasses.
- Subclasses customize behavior within a controlled sandbox.
- Difference from Template Method.
- Game/entity scripting use case.
- Inheritance coupling risk.

---

## 49. update-method

Category: Performance, Game, and Low-Level Optimization Patterns  
MAANG interview meter: Optional  
Software usage meter: Optional  
Repository module: [update-method](../../github-repo/update-method)

Subtopics to study:
- Each object updates itself once per frame/tick.
- Difference from Game Loop.
- Object-local behavior in simulations.
- Ordering and dependency concerns.
- When centralized systems outperform per-object updates.

---

## 50. curiously-recurring-template-pattern

Category: Language, JVM, and Idiom Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [curiously-recurring-template-pattern](../../github-repo/curiously-recurring-template-pattern)

Subtopics to study:
- Generic base class parameterized by subclass type.
- Fluent APIs returning precise subclass type.
- Difference from normal inheritance.
- Self-referential generics in Java.
- Complexity and confusing type signatures.

---

## 51. separated-interface

Category: Language, JVM, and Idiom Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [separated-interface](../../github-repo/separated-interface)

Subtopics to study:
- Interface lives in a different package/module from implementation.
- Dependency direction and boundary enforcement.
- Difference from Dependency Inversion.
- Plugin/provider architectures.
- Avoiding accidental dependency on concrete classes.

---

## 52. type-object

Category: Language, JVM, and Idiom Patterns  
MAANG interview meter: Low  
Software usage meter: Low  
Repository module: [type-object](../../github-repo/type-object)

Subtopics to study:
- Representing types as runtime objects instead of subclasses.
- Data-driven type definitions.
- Difference from Strategy and Prototype.
- Game item/entity type catalogs.
- Validation and behavior placement.

---

## 53. mute-idiom

Category: Language, JVM, and Idiom Patterns  
MAANG interview meter: Optional  
Software usage meter: Optional  
Repository module: [mute-idiom](../../github-repo/mute-idiom)

Subtopics to study:
- Suppressing or ignoring exceptions intentionally.
- Cleanup paths where failure is non-critical.
- Difference from proper exception handling.
- Logging, observability, and safety concerns.
- Use sparingly and document why silence is acceptable.

---

## 54. business-delegate

Category: Enterprise Application Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [business-delegate](../../github-repo/business-delegate)

Subtopics to study:
- Client-side abstraction over business service lookup/invocation.
- Hiding remote service details from presentation code.
- Difference from Service Locator and Facade.
- Retry/fallback and service selection concerns.
- Legacy enterprise Java context.

---

## 55. session-facade

Category: Enterprise Application Patterns  
MAANG interview meter: Low  
Software usage meter: Medium  
Repository module: [session-facade](../../github-repo/session-facade)

Subtopics to study:
- Coarse-grained facade over business components.
- Reducing chatty remote calls.
- Difference from Facade and Service Layer.
- Transaction boundary coordination.
- Legacy EJB/service architecture context.
