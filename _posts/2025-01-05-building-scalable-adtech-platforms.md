---
layout: post
title: "Building Scalable and Highly Available Systems: Lessons from Ad-Tech"
date: 2025-01-05 11:00:00 +0300
categories: [architecture, scalability, backend]
tags: [ad-tech, high-availability, performance, caching, microservices, system-design]
author: Onur Atci
---

If there is one domain that pushes the boundaries of system design, it's Ad-Tech. In my experience working on ad management platforms, specifically during my time at Getir, I quickly realized that traditional approaches to backend architecture simply don't survive contact with ad serving traffic. 

When you're dealing with ad impression tracking, real-time bidding, and campaign targeting, milliseconds matter. The system needs to handle massive spikes in throughput while maintaining strict latency SLAs, often under 50ms. In this post, I want to share some critical lessons I've learned about building scalable, highly available systems under extreme load.

## 1. The Power (and Danger) of Caching Strategies

In Ad-Tech, your database is your bottleneck. Period. You cannot afford to query a relational database for campaign configuration on every ad request.

### The Problem with Basic Caching
Initially, you might think a simple Redis cache is enough:

```java
public Campaign getActiveCampaign(String locationId) {
    Campaign cached = redisTemplate.opsForValue().get("campaign:" + locationId);
    if (cached != null) {
        return cached;
    }
    Campaign dbResult = campaignRepository.findActiveByLocation(locationId);
    redisTemplate.opsForValue().set("campaign:" + locationId, dbResult, Duration.ofMinutes(5));
    return dbResult;
}
```

This works fine for a blog, but under a massive traffic spike (like a push notification going out to millions of users), if the cache expires, you'll experience a "Cache Stampede". Thousands of concurrent requests will hit the database simultaneously, bringing it down instantly.

### The Solution: Multi-Level Caching with Async Refresh
To solve this, we moved to a robust multi-level caching architecture:

1. **L1 Cache (In-Memory)**: Caffeine cache for immediate configuration reads.
2. **L2 Cache (Distributed)**: Redis cluster for shared state across instances.
3. **Background Refresh**: Instead of expiring cache entries and reading synchronously, we update caches asynchronously via background workers or change-data-capture (CDC) mechanisms like Debezium.

```java
@Service
public class CampaignService {
    // In-memory cache with near-instant access
    private final LoadingCache<String, Optional<Campaign>> localCache = 
        Caffeine.newBuilder()
            .maximumSize(10_000)
            .refreshAfterWrite(1, TimeUnit.MINUTES) // Async background refresh
            .build(key -> fetchFromRedisOrDb(key));

    public Optional<Campaign> getActiveCampaign(String locationId) {
        return localCache.get(locationId);
    }
}
```

With this approach, the main thread never gets blocked querying the database, guaranteeing latency bounds.

## 2. Event-Driven Asynchronous Processing

Ad tracking generates a firehose of data: impressions, clicks, viewability metrics, etc. Processing these synchronously is a recipe for disaster.

During a Black Friday campaign, an influx of requests can easily overwhelm synchronous APIs. To build resilience, the ingestion layer must be decoupled from the processing layer.

### Enter Kafka
We implemented Apache Kafka to ingest all tracking events. 

```java
@RestController
@RequestMapping("/v1/track")
public class TrackingController {
    private final KafkaTemplate<String, TrackingEvent> kafkaTemplate;

    @PostMapping("/impression")
    public ResponseEntity<Void> trackImpression(@RequestBody ImpressionPayload payload) {
        // We only acknowledge receipt. Processing happens later.
        TrackingEvent event = mapper.toEvent(payload, System.currentTimeMillis());
        kafkaTemplate.send("ad-impressions", event.getCampaignId(), event);
        
        return ResponseEntity.accepted().build();
    }
}
```

This simple architectural shift allows the edge APIs to scale horizontally and absorb massive traffic spikes simply by appending to a distributed log. Downstream consumers can process these events at their own pace without risking system stability.

## 3. Designing for Failure (Resilience Patterns)

In distributed systems, failures are guaranteed. A third-party data enrichment service will time out, Redis instances might fail over, and network partitions will happen. The goal is graceful degradation.

### Circuit Breakers
If a downstream service is struggling, sending it more requests will only make it worse. We rely heavily on Circuit Breakers (like Resilience4j).

```java
@CircuitBreaker(name = "fraudDetectionService", fallbackMethod = "fallbackFraudCheck")
public boolean checkFraud(AdRequest request) {
    return fraudClient.scoreRequest(request);
}

// Fallback method prevents cascading failures
public boolean fallbackFraudCheck(AdRequest request, Throwable t) {
    log.warn("Fraud service unavailable, defaulting to safe tier for request {}", request.getId());
    return true; // Graceful degradation: allow request but flag for async review
}
```

### Rate Limiting and Load Shedding
We also implemented strict API gateways that employ token bucket algorithms to shed excess load when we hit predefined thresholds. It's much better to drop 5% of requests and serve the other 95% flawlessly, than to accept 100% of requests and have the whole system crash.

## 4. NoSQL for High Throughput Writes

While PostgreSQL is fantastic for relational data (like campaign settings or billing information), it struggles as a real-time event sink.

For analytics and metric aggregation, we adopted NoSQL solutions (like Cassandra or MongoDB depending on the exact read/write patterns). The schema-less nature and eventual consistency models allowed us to handle thousands of writes per second seamlessly. Data points from Kafka consumers were aggregated and dumped into NoSQL stores in micro-batches to maximize throughput.

## Conclusion

Scaling a platform is less about choosing a magic framework and more about understanding system bottlenecks and managing trade-offs. In Ad-Tech, the trade-off is often consistency for availability and lower latency. 

By utilizing aggressive multi-tier caching, asynchronous event-driven pipelines, circuit breakers, and the right database for the right job, we successfully transformed fragile architectures into robust, resilient platforms capable of handling whatever traffic the business threw at us.

---

*Are you tackling scaling challenges in your current projects? Have you implemented similar patterns, or do you prefer other approaches? Let's connect on [LinkedIn](https://linkedin.com/in/onuratci) and discuss!*
