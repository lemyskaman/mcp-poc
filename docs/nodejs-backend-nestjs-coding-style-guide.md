# Node.js TypeScript Coding Style Guide for Backend APIs with NestJS

*RESTful \& GraphQL APIs / Clean Architecture (Vertical Slice, Ports \& Adapters)*
*Version 2025-08-17*
[lemyskaman/nodejs-backend-nestjs-coding-style-guide.md](https://gist.github.com/lemyskaman/760d1915bedc9b736253263b07940b6e)

This guide compiles proven practices from:

- **NestJS documentation**
- **Node.js documentation**
- Academic and industry sources (IEEE, Elsevier, journals, books)
- Renowned dev sources (Medium, Hacker News, top GitHub repos)

It applies to backend systems (RESTful/GraphQL) using **NestJS** and **TypeScript**, emphasizing Hexagonal/Clean Architecture (Ports \& Adapters), modular "vertical slice" boundaries, Domain-Driven Design (DDD), and pragmatic maintainability.[^1][^2][^3][^4][^5][^6][^7][^8][^9][^10][^11]

***

## Table of Contents

1. [Philosophy](#philosophy)
2. [Recommended File/Folder Structure](#recommended-filefolder-structure)
3. [General TypeScript \& Node.js Practices](#general-typescript--nodejs-practices)
4. [Project Initialization \& Tooling](#project-initialization--tooling)
5. [NestJS-Specific Patterns](#nestjs-specific-patterns)
6. [Architectural Guidelines (Clean/Hexagonal/DDD)](#architectural-guidelines-cleanhexagonalddd)
7. [RESTful \& GraphQL API Design](#restful--graphql-api-design)
8. [Testing Strategy](#testing-strategy)
9. [Error Handling \& Observability](#error-handling--observability)
10. [Security \& Robustness](#security--robustness)
11. [References \& Further Reading](#references--further-reading)

***

## Philosophy

- **Explicitness**: Favor clarity, descriptive names, explicit types and return types, full words (no abbreviations).
- **Single Responsibility**: Short functions, logic isolated per domain concern.
- **SOLID**: Lean heavily on SOLID and composition, not deep inheritance.
- **Isolation**: Domain is independent from frameworks and infrastructure.
- **Ports \& Adapters (Hexagonal)**: Contract-driven, outside-in design.
- **Vertical Slices (Feature Modules/Bounded Contexts)**: Align code to domain modules, not technical layers.[^4][^6][^8]

***

## Recommended File/Folder Structure

*Vertical Slices + Ports \& Adapters (Hexagonal/Clean Architecture/DDD)*

```
src/
├── modules/                # Each domain/feature ("vertical slice") is a self-contained module
│   ├── user/
│   │   ├── application/        # Use cases, command/query handlers, service logic (domain orchestrators)
│   │   ├── domain/             # Pure business objects, domain services, factories, value objects, aggregates
│   │   │   ├── user.entity.ts
│   │   │   ├── user.value-object.ts
│   │   │   └── user.repository.port.ts   # Output "port" interfaces (contracts)
│   │   ├── infrastructure/     # Adapters for persistence, http, 3rd party, external drivers
│   │   │   ├── user.repository.adapter.ts
│   │   │   └── user.model.ts             # ORM/datastore representation
│   │   ├── presentation/        # HTTP & API adapters (controllers, resolvers, DTOs, validators)
│   │   │   ├── user.controller.ts
│   │   │   ├── user.graphql.resolver.ts
│   │   │   └── user.dto.ts
│   │   ├── user.module.ts       # Local NestJS module, wires dependencies (DI "assembly root")
│   │   └── index.ts
│   └── ... (other features/bounded contexts)
├── shared/                # Shared kernel: base entities, exceptions, utilities, infrastructure, types
│   ├── config/
│   ├── exceptions/
│   ├── logging/
│   ├── guards/
│   ├── middleware/
│   ├── interceptors/
│   ├── decorators/
│   ├── dtos/
│   ├── types/
├── main.ts                # NestJS app bootstrap
├── app.module.ts          # Root module, composes all feature modules
└── ...
test/                     # Unit and E2E tests, mirrors src module/feature structure
```


***

## General TypeScript \& Node.js Practices

- **Type Safety**: Always specify parameter and return types, including for async/Promise results; avoid `any` unless absolutely necessary.[^3][^12][^13][^1]
- **Naming**:
    - *PascalCase* for classes, interfaces, types, enums.
    - *camelCase* for variables, function names, method names.
    - *kebab-case* for file/folder names.
    - *SCREAMING_SNAKE_CASE* for module-level constants and environment variables.
- **Structure**:
    - 1 export per file.
    - 1 class and its contract(s) per file.
    - Prefer composition, not inheritance; deeply nested class trees are discouraged.[^1][^3]
    - Expose only contracts/ports, hide implementation details.
- **Constants/Config Values**:
    - Store in separate files (`config/`), load via environment variables, never hardcode secrets in code.[^10][^13][^3]

***

## Project Initialization \& Tooling

- **tsconfig**: Enable `strict`, `noImplicitAny`, `strictNullChecks`, `strictPropertyInitialization`.
- **Lint/Format**: Use ESLint + Prettier with strict rules, auto-run in CI/CD and pre-commit hooks (`husky`, `lint-staged`).[^3][^10]
- **Testing**: Use Jest or equivalent, target both unit and e2e per feature/module.
- **Documentation**: Use JSDoc/TSDoc and OpenAPI (`@nestjs/swagger`) for REST; schema comments for GraphQL.
- **Git**: Use `.env.example`, ignore secret keys, and follow semantic commit conventions (Conventional Commits).[^14][^10]

***

## NestJS-Specific Patterns

- **Modules per feature/domain**: Each main domain gets a module (`user`, `order`, etc.) containing ALL related code.[^15][^8][^4]
- **Providers**: Implement business logic in services; interact with domain only via contracts (ports). Never place app logic in controllers.
- **DTOs**: Use for ALL HTTP/GraphQL boundaries. Validate with `class-validator` decorators.
- **Pipes**: Use for transformation/parsing; custom pipes for cross-cutting validation.
- **Guards**: For authentication/authorization checks.
- **Interceptors**: For logging, error mapping, throttling, metrics, or response transformation.
- **Global error filters**: Centralize uncaught exceptions, log, transform to client-friendly errors.

***

## Architectural Guidelines (Clean/Hexagonal/DDD)

### Core Principles

- **Isolation**: Domain layer is framework-agnostic, no imports from NestJS/libs/ORMs, only TypeScript and domain logic.[^6][^9][^14]
- **Ports and Adapters**:
    - Ports = TypeScript interfaces (contracts).
    - Adapters = Infrastructure (e.g., database, API), Presentation (controllers, GraphQL, CLI, schedulers), or other services implementing ports/contracts.
- **Dependency Inversion**: Domain imports only ports; adapters implement and are plugged via dependency injection.[^2][^6]
- **Domain-Driven Design**:
    - Isolate domain models/aggregates and business logic (rules, invariants). Follow DDD patterns (aggregate, entity, value object, domain service, repository, factory).[^7][^16][^8][^9]
    - Use factories, value objects, and aggregate methods to enforce invariants and encapsulate logic.
- **CQRS/Vertical Slices** (optional but recommended for complex apps): Split reads (queries) and writes (commands) into independent, request-handling classes for composability, testability, and clarity.[^17][^8][^2]
- **Bounded contexts**: Each module represents a business context, no global state, clear contracts for inter-context communication.[^18][^16][^8][^7]

***

## RESTful \& GraphQL API Design

### REST Best Practices

- Follow resource naming (`/users/:id`, `/orders/:orderId/items`).[^19][^15]
- Use proper HTTP verbs: GET (read), POST (create), PUT/PATCH (update), DELETE (delete).
- Response format: Consistent `ApiResponse<T>` (status, payload/data, error object).[^19]
- Pagination: Use query params (`?page=1&limit=10`), return metadata in response.
- Implement versioning (URL-based `/v1/...` or Accept headers).
- Use OpenAPI (Swagger) via `@nestjs/swagger` to document all endpoints.


### GraphQL Best Practices

- Use code-first (`@nestjs/graphql`) with TypeScript classes; use decorators for type and resolver definitions.[^20][^21][^22][^23]
- Strongly type all arguments, inputs, and return values.
- Keep resolvers thin; delegate to service/use-case layer.
- Input validation via GraphQL DTOs (+ `class-validator`).
- Apply authorization at field/resolver level with guards.
- Return union or error types for mutation failures, not just raw errors.[^24][^25][^26]
- Group GraphQL schema/types/docs with relevant module.
- For advanced cases, leverage query complexity limitations and depth-limiting middlewares for security.

***

## Testing Strategy

- Unit tests: Each domain, application (use case), and adapter class has its own test file. Use mocks/fakes for dependencies via ports.
- E2E tests: Use Supertest (REST) or Apollo (GraphQL) in `test/` or `e2e/` folders, mirroring feature structure.
- Arrange-Act-Assert naming in tests (`inputUserDto`, `mockUserRepo`, `expectedResult`).[^1][^3]
- Use Given-When-Then format for behavioral/acceptance tests.
- 100% test coverage for core domain, >80% for rest of codebase.
- Automate tests in CI/CD and fail builds if thresholds not met.

***

## Error Handling \& Observability

- Centralized error filter (`@Catch`) for all exceptions; consistent error response structure (error code, message, context).[^2][^6][^3]
- Do not leak stack traces in production responses.
- Errors in adapters are mapped to domain-relevant error objects and thrown up; log all unexpected errors with context.
- Structured logging with tags for request ID, error type, user, etc.
- Use interceptors or middleware for logging and metrics (trace every request, latency, error code, user, etc).
- Leverage metrics endpoints for Prometheus or equivalent, plus audit logs for sensitive operations.

***

## Security \& Robustness

- Use `class-validator` in DTOs for validation and transformation.[^2][^1]
- Parametrize all database queries to avoid SQL injection.
- Secure all secrets in environment variables, never in code.
- Implement guards for all protected routes, both REST and GraphQL.[^3][^1][^2]
- Use global and per-route guards for authentication and RBAC (role-based access control).
- Sanitize/escape user inputs; avoid raw user-provided GraphQL queries when possible.
- Limit query depth/breadth for GraphQL endpoints.
- Rate-limit sensitive or public API endpoints.[^19]

***

## File Tree Example: Clean Architecture, Vertical Slice, Ports \& Adapters

```plaintext
src/
├── modules/
│   ├── user/
│   │   ├── domain/
│   │   │   ├── user.entity.ts
│   │   │   ├── user.service.ts
│   │   │   └── user.repository.port.ts
│   │   ├── application/
│   │   │   ├── create-user.usecase.ts
│   │   │   ├── get-user-by-id.query.ts
│   │   │   └── index.ts
│   │   ├── infrastructure/
│   │   │   ├── user.repository.adapter.ts
│   │   │   ├── user.model.ts
│   │   │   └── db.module.ts
│   │   ├── presentation/
│   │   │   ├── user.controller.ts
│   │   │   ├── user.graphql.resolver.ts
│   │   │   ├── user.dto.ts
│   │   │   └── user.router.ts
│   │   └── user.module.ts
│   └── order/
│       └── ... (same structure)
├── shared/
│   ├── config/
│   ├── exceptions/
│   ├── guards/
│   ├── interceptors/
│   ├── logging/
│   ├── types/
│   └── utils/
├── main.ts
├── app.module.ts
└── ...
test/
```

- Each `module` is a vertical slice, containing all layers.
- Each module contains: domain logic, application logic, infra adapters, API layer.
- “Ports” are TypeScript interfaces in `domain/`, “adapters” are their implementations in `infrastructure/`.
- Dependency injection wires adapters at module boundaries (no direct dependencies).[^8][^9][^6][^2]

***

## References \& Further Reading

- [NestJS Docs - Best Practices, GraphQL, Adapters](https://docs.nestjs.com/)
- [Node.js Documentation](https://nodejs.org/en/docs)[^27][^13]
- [TypeScript (NestJS) Best Practices - cursorrules.org](https://cursorrules.org/article/typescript-nestjs-best-practices-cursorrules-promp)[^1][^3]
- [Medium: Clean Node.js Architecture — NestJS \& TypeScript](https://betterprogramming.pub/clean-node-js-architecture-with-nestjs-and-typescript-34b9398d790f)[^28]
- [DEV: Best NestJS Practices, Hexagonal \& Event-Driven Architecture](https://dev.to/geampiere/building-flexible-applications-with-hexagonal-and-event-driven-architecture-in-nestjs-578i)[^2]
- [Rida Kaddir: Clean Code using Hexagonal Architecture in NestJS](https://ridakaddir.com/blog/post/nestjs-clean-code-using-hexagonal-architecture)[^6]
- [NestJS Clean Architecture Demo](https://github.com/damienbeaufils/nestjs-clean-architecture-demo)[^4]
- [LogRocket: Build a GraphQL API with NestJS](https://blog.logrocket.com/how-to-build-graphql-api-nestjs/)[^20]
- [IEEE/Elsevier/journals: DDD, CQRS, Clean Architecture in Microservices][^29][^16][^30][^31][^7][^18]
- [LinkedIn: Node.js TypeScript Best Practices](https://www.linkedin.com/pulse/structuring-nodejs-projects-typescript-best-practices-f-rutes-42isf)[^5]
- [Books: Mastering TypeScript, API Design Patterns][^32][^33][^19]
- [Academic: Implementation \& benefit of DDD+Clean Architecture][^16][^7][^17][^18]
- [HackerNews, Reddit: Node.js/NestJS DDD/Hexagonal design discussions][^34]

***

*Adopting these patterns—modular feature domains, ports/adapters for true separation, exhaustive type-driven design, test-ability, and explicitness—results in backends that are maintainable, testable, robust, and modern.*

***

## Endnote

_Evolve the standards above for your team and business. This living guide should grow in depth with feedback and collective learning._

<div style="text-align: center">⁂</div>

[^1]: https://cursorrules.org/article/typescript-nestjs-best-practices-cursorrules-promp

[^2]: https://dev.to/geampiere/building-flexible-applications-with-hexagonal-and-event-driven-architecture-in-nestjs-578i

[^3]: https://cursor.directory/rules/typescript

[^4]: https://github.com/damienbeaufils/nestjs-clean-architecture-demo

[^5]: https://www.linkedin.com/pulse/structuring-nodejs-projects-typescript-best-practices-f-rutes-42isf

[^6]: https://ridakaddir.com/blog/post/nestjs-clean-code-using-hexagonal-architecture

[^7]: https://journal.pubmedia.id/index.php/pjise/article/view/2511

[^8]: https://dev.to/sallbro/comparing-domain-driven-design-ddd-and-clean-architecture-8d

[^9]: https://dev.to/bendix/applying-domain-driven-design-principles-to-a-nest-js-project-5f7b

[^10]: https://dev.to/yugjadvani/enterprise-grade-nodejs-leveraging-typescript-eslint-prettier-for-production-excellence-39lj

[^11]: http://www.diva-portal.org/smash/get/diva2:1889284/FULLTEXT01.pdf

[^12]: https://www.w3schools.com/nodejs/nodejs_typescript.asp

[^13]: https://www.robinwieruch.de/typescript-node-js/

[^14]: https://andrea-acampora.github.io/nestjs-ddd-devops/

[^15]: https://docs.nestjs.com/first-steps

[^16]: https://www.onlinescientificresearch.com/articles/domaindriven-design-in-modern-software-architecture-best-practices-and-patterns.pdf

[^17]: https://journals.politehnica.dp.ua/index.php/it/article/view/552

[^18]: https://ieeexplore.ieee.org/document/10495888/

[^19]: https://www.multiplayer.app/system-architecture/api-design-patterns/

[^20]: https://blog.logrocket.com/how-to-build-graphql-api-nestjs/

[^21]: https://docs.nestjs.com/graphql/quick-start

[^22]: https://paclinjanja.hashnode.dev/graphql-api-nestjs-mongodb-a-modern-way-part-1

[^23]: https://dev.to/husnain/apollo-graphql-implementation-in-nestjs-3chn

[^24]: https://iciset.in/Paper8011.pdf

[^25]: https://arxiv.org/pdf/2112.08267.pdf

[^26]: https://arxiv.org/pdf/2208.04784.pdf

[^27]: https://nodejs.org/api/typescript.html

[^28]: https://betterprogramming.pub/clean-node-js-architecture-with-nestjs-and-typescript-34b9398d790f

[^29]: https://ieeexplore.ieee.org/document/10568262/

[^30]: https://www.temjournal.com/content/124/TEMJournalNovember2023_1985_1994.html

[^31]: https://arxiv.org/abs/2407.02512

[^32]: https://www.semanticscholar.org/paper/6b9c1dc22b7e26b6e1e2b1affabb6405ad5dfe0f

[^33]: https://www.semanticscholar.org/paper/a67eaa5b127d26645c03fa5e680a8efe3bd65007

[^34]: https://www.reddit.com/r/Nestjs_framework/comments/1fudq55/should_i_use_ddd_or_clean_architecture_in_nestjs/

[^35]: https://scholar.kyobobook.co.kr/article/detail/4010071396694

[^36]: https://ph.pollub.pl/index.php/jcsi/article/view/2910

[^37]: https://journal.esrgroups.org/jes/article/view/2170

[^38]: https://csecurity.kubg.edu.ua/index.php/journal/article/view/860

[^39]: https://ieeexplore.ieee.org/document/10275654/

[^40]: https://www.semanticscholar.org/paper/7b2476e2fb757f042c66c77ff28474dad7f8bddb

[^41]: https://www.nature.com/articles/s41409-019-0679-x

[^42]: https://www.semanticscholar.org/paper/f15679e4385d2a854af3e1b130bc4c6209866c9e

[^43]: https://arxiv.org/pdf/2302.12163.pdf

[^44]: https://arxiv.org/pdf/2108.08027.pdf

[^45]: http://arxiv.org/pdf/2409.00921.pdf

[^46]: https://arxiv.org/pdf/2408.14431.pdf

[^47]: https://drops.dagstuhl.de/opus/volltexte/2015/5218/pdf/8.pdf

[^48]: https://arxiv.org/pdf/2004.01321.pdf

[^49]: https://arxiv.org/pdf/2101.04622.pdf

[^50]: https://arxiv.org/pdf/2101.00756.pdf

[^51]: https://arxiv.org/pdf/2007.08048.pdf

[^52]: https://arxiv.org/pdf/1511.02956.pdf

[^53]: https://dev.to/bsachref/nestjs-best-practice-85

[^54]: https://dev.to/drbenzene/best-nestjs-practices-and-advanced-techniques-9m0

[^55]: https://www.youtube.com/watch?v=4_4p5Ojs5XA

[^56]: https://arxiv.org/abs/1906.07535

[^57]: https://arxiv.org/pdf/1809.08319.pdf

[^58]: https://arxiv.org/pdf/2209.05833.pdf

[^59]: http://arxiv.org/pdf/2412.13918.pdf

[^60]: https://arxiv.org/pdf/1701.00626.pdf

[^61]: https://www.mdpi.com/2073-431X/10/11/138/pdf

[^62]: https://dl.acm.org/doi/pdf/10.1145/3651140

[^63]: http://arxiv.org/pdf/2402.16567.pdf

[^64]: https://arxiv.org/html/2501.08947

[^65]: https://arxiv.org/pdf/2409.12646.pdf

[^66]: https://arxiv.org/pdf/2203.09903.pdf

[^67]: https://ph.pollub.pl/index.php/jcsi/article/download/2744/2542

[^68]: https://zenodo.org/record/7994295/files/2023131243.pdf

[^69]: https://arxiv.org/pdf/2403.04935.pdf

[^70]: https://dl.acm.org/doi/pdf/10.1145/3609427

[^71]: https://arxiv.org/html/2408.08363v1

[^72]: https://docs.nestjs.com/websockets/adapter

[^73]: https://dev.to/tak089/typescript-design-patterns-5hei

[^74]: https://www.reddit.com/r/typescript/comments/oowj2s/question_what_is_a_good_design_pattern_for/

[^75]: https://ieeexplore.ieee.org/document/10278057/

[^76]: https://conferences.vntu.edu.ua/index.php/mccs/mccs2024/paper/view/22182

[^77]: http://jrpsjournal.in/index.php/j/article/view/50

[^78]: http://arxiv.org/pdf/2307.09090.pdf

[^79]: http://arxiv.org/pdf/2307.06382.pdf

[^80]: https://arxiv.org/pdf/2210.01647.pdf

[^81]: https://arxiv.org/pdf/2302.00740.pdf

[^82]: https://www.frontiersin.org/articles/10.3389/fdata.2024.1448481/full

[^83]: https://arxiv.org/html/2407.02512v1

[^84]: https://zenodo.org/records/6890610/files/ICSA-conformance-assessment-of-decisions.pdf

[^85]: https://dl.acm.org/doi/pdf/10.1145/3689488.3689990

[^86]: https://www.mdpi.com/2076-3417/12/22/11439/pdf?version=1668584569

[^87]: http://arxiv.org/pdf/1709.10255.pdf

[^88]: https://ijrpr.com/uploads/V5ISSUE3/IJRPR23457.pdf

[^89]: https://repository.stcloudstate.edu/csit_etds/5/

[^90]: https://blog.logrocket.com/express-typescript-node/

