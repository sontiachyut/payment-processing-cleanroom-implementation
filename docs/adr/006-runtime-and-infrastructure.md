# ADR 006 — Runtime support and incremental infrastructure

**Status:** proposed with unresolved testing-generation choice. Baseline remains Java 21/Maven/Spring Web/PostgreSQL/JPA/Flyway/Resilience4j, Kafka at M7, requested JUnit 5/Mockito/Testcontainers, k6, OTel/Micrometer/Prometheus/Grafana, later Compose/Actions/ECS/RDS.

Research found [Boot 3.5.16 ended OSS support](https://spring.io/blog/2026/06/25/spring-boot-3-5-16-available-now/); its [testing docs](https://docs.spring.io/spring-boot/3.5/reference/testing/index.html) use JUnit 5, while [current Boot testing](https://docs.spring.io/spring-boot/reference/testing/index.html) uses JUnit 6. Do not silently mix managed versions, choose an unsupported release, or replace the owner's explicit JUnit 5 requirement. M1 must verify a supported compatible combination or obtain explicit approval to use current Boot/Jupiter 6; a commercially supported legacy route is a separate cost/availability choice, not assumed.

No runtime is pinned or build generated now. Redis/Kubernetes/microservices/native images are deferred absent need. Compose/local correctness precedes CI/cloud; deployment and billable resources require later explicit approval. Review exact dependency/resilience/telemetry compatibility at implementation, and keep the support limitation visible rather than fabricating a final version matrix.
