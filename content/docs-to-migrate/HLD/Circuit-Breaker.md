# Circuit Breaker ⚡ Pattern in Microservices

![](/images/1_WkUIE-a7thjUl2Yh9VVkkg1.webp)

## Introduction

In this article, we are going to briefly discuss the circuit breaker design pattern in microservices. Let’s start with a simple example.

![](/images/1_zzkyJE6YRAqN-1lNqVUnWw.webp)

Let’s say we have a large\-scale system with several microservices. Order\-Service and Payment\-Service are two microservices among them. After the user clicks the “Complete Order” button, a request will be sent to the backend. In the backend, requests will go through several other services such as User\-Service, Item\-Service, etc. Finally, the request will go from Order\-Service to Payment\-Service. Then Payment\-Service will process the card details, get the payments done and the payment status will be sent to Order\-Service back.
## Problem

![](/images/1_O5Wkt4dCszr1BCJSg1-3Wg.webp)

However, let’s assume that Payment Service fails to process requests for a while due to some reason such as network issues, unavailability of the service, service failure, etc. Meanwhile, the system receives 10,000 purchasing requests. What happens is, that Order\-Service sends each request to Payment\-Service using available threads and waits for a response. However, after a few seconds, these requests will be timed out. This will be a huge drawback due to ineffective network, thread, memory, and database allocation for each request. Most importantly this drawback will cascade to all other services, that are involved in this order\-processing task.

## Solution

This issue can be easily solved if we could identify, that there is some ongoing issue with Payment\-Service. Hence, Order\-Service can avoid sending new requests to Payment\-Service. Rather it can immediately send a suitable fail response, for incoming new payment processing requests.
Now let’s see how we can identify whether Payment\-Service is working or not. For this, we can use some statistical techniques along with 3 states.

* **Closed State**: In this state, the circuit is closed. This means we send requests to Payment\-Service. After receiving a success/failure response from Payment\-Service, we calculate the failure rate of the last N requests. If the failure rate > X threshold, then the state will be switched to Open\-State. Otherwise, stay in Closed\-State.

* **Open\-State**: In this state, the circuit is open. This means we do not send requests to Payment\-Service. We stay in this state for T secs and switch state to Half\-Open\-State.

* **Half\-Open\-State**: In this state, the circuit is partially closed. This means we send only a few n \(n<<N\) requests and check the failure rate. If it still fails we move back to Open\-State or go to Closed\-State.

The diagram below explains these three states.

![](/images/1_SNSpWicSI0KXmRDIKKwTig.webp)

## Implementation

Below is a sample Java code snippet for Circuit Breaker using [resilience4j](https://resilience4j.readme.io/docs/examples) with discussed X, T, N, and n parameters.

```java
// CODE IN ORDER-SERVICE
// Create a custom configuration for a CircuitBreaker
CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
    .failureRateThreshold(50) // X
    .waitDurationInOpenState(Duration.ofMillis(1000)) // T
    .ringBufferSizeInClosedState(100) // N
    .ringBufferSizeInHalfOpenState(10) // n
    .build();

// Build Circuit Breaker
CircuitBreaker circuitBreaker= 
  CircuitBreaker.ofDefaults("Payment-Service", circuitBreakerConfig);

// Decorate Function
Supplier<String> decoratedSupplier = CircuitBreaker
        .decorateSupplier(circuitBreaker, this::callPaymentService);

// Make Call
Try<String> result = Try.ofSupplier(decoratedSupplier);
```

Likewise, we can use this microservice design pattern called Circuit Breaker Pattern to prevent an application from calling a service, which most likely failed.

For more info please visit [Link](https://resilience4j.readme.io/docs/examples)

Thank You ❤️\
Happy Coding 🙌

