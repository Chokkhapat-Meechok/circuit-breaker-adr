# Adopt Circuit Breaker Pattern

## Context
Our system architecture relies on synchronous HTTP communication between microservices. Currently, when a downstream service (e.g., Inventory Service) becomes unresponsive or experiences high latency, the calling service waits indefinitely until a timeout occurs. This leads to thread pool exhaustion and resource waste. Consequently, a single service failure can propagate upstream, causing a **Cascading Failure** that brings down the entire application. We need a stability pattern to detect failures and prevent them from spreading.

## Decision
We will adopt the **Circuit Breaker Pattern** to wrap all synchronous network calls. The mechanism will operate based on a state machine with three states:

1.  **Closed:** The system operates normally. Requests are allowed through. If the failure rate exceeds a configured threshold, the state changes to **Open**.
2.  **Open:** To prevent overloading the failing service, all requests are blocked immediately (Fail Fast) without attempting to contact the downstream service. After a specified timeout expires, the state changes to **Half-Open**.
3.  **Half-Open:** A limited number of probe requests are allowed to pass. If these requests succeed, the circuit returns to **Closed**. If they fail, it reverts to **Open**.

## Rationale
* **Fault Tolerance:** It prevents catastrophic cascading failures by isolating the problematic service.
* **Fail Fast:** Instead of waiting for a timeout, the system rejects requests immediately when the circuit is open, improving user experience by providing instant feedback.
* **System Recovery:** The "Half-Open" state allows the system to self-heal and automatically recover once the downstream service is back online.

## Consequences
**Pros:**
* Increases overall system resilience and uptime.
* Preserves system resources (CPU/Threads) during outages.
* Enables graceful degradation (e.g., returning fallback data).

**Cons:**
* Adds complexity to the codebase and testing procedures.
* Requires careful configuration of thresholds and timeouts to avoid "flapping" (switching between Open and Closed states too frequently).

## Sample code
```java
// Pseudo-code demonstrating the Circuit Breaker logic
public Response callExternalService() {
    if (circuitBreaker.isOpen()) {
        // Fail Fast: Return error or fallback immediately
        return getFallbackResponse();
    }

    try {
        // Closed/Half-Open: Attempt to call the service
        Response response = httpClient.sendRequest();
        circuitBreaker.recordSuccess();
        return response;
    } catch (Exception e) {
        // Record failure to calculate threshold
        circuitBreaker.recordFailure(e);
        throw e;
    }
}