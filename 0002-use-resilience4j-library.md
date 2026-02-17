# Use Resilience4j as Fault Tolerance Library

## Context
Following the decision to adopt the Circuit Breaker pattern (ADR-0001), we require a production-ready implementation. Building a thread-safe and performant Circuit Breaker from scratch is error-prone. We evaluated available Java libraries, primarily comparing **Netflix Hystrix** and **Resilience4j**. Note that Hystrix is currently in maintenance mode and no longer actively developed.

## Decision
We will use **Resilience4j** as our standard fault tolerance library. Specifically, we will integrate `resilience4j-spring-boot2` into our microservices. This library offers a lightweight, functional programming model and integrates seamlessly with the Spring ecosystem.

## Rationale
* **Active Maintenance:** Resilience4j is actively maintained and is the recommended replacement for Hystrix by the Spring Cloud team.
* **Modular Design:** Unlike Hystrix, Resilience4j is modular. We can import only the Circuit Breaker module to keep the application lightweight.
* **Spring Boot Integration:** It provides excellent support for Spring Boot via AOP (Aspect-Oriented Programming) annotations and external configuration.

## Consequences
**Pros:**
* Simplifies implementation using the `@CircuitBreaker` annotation.
* Centralized configuration via `application.yml` allows tuning without code changes.
* Built-in support for metrics monitoring (Micrometer/Prometheus).

**Cons:**
* Developers must learn the specific configuration parameters (e.g., sliding window type, minimum number of calls).
* Requires adding external dependencies to the project `pom.xml`.

## Sample code
**1. Service Implementation (Annotation approach):**
```java
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import org.springframework.stereotype.Service;

@Service
public class InventoryService {

    private static final String BACKEND_A = "inventory";

    @CircuitBreaker(name = BACKEND_A, fallbackMethod = "fallbackInventory")
    public String getInventory(String productId) {
        // Simulating a call to a downstream service
        return restTemplate.getForObject("/api/inventory/" + productId, String.class);
    }

    // Fallback method executes when the Circuit is OPEN or the call fails
    public String fallbackInventory(String productId, Throwable t) {
        return "Fallback: Inventory information unavailable for product " + productId;
    }
}
