# Rate Limiter [System Design]

A rate limiter is a mechanism to allow or block requests based on a predefined threshold. This document outlines different algorithms and techniques used for rate limiting, along with design considerations and optimizations.

## Threshold Management Algorithms

### 1. Token Bucket
#### Schema:
- **Bucket Size**: Maximum capacity of tokens in the bucket.
- **Refill Size**: Rate at which tokens are added to the bucket.

### 2. Leaky Bucket
#### Schema:
- **Queue Size**: Maximum number of requests that can be queued.
- **Outflow Rate**: Rate at which requests are processed (ensures a steady flow).

### 3. Sliding Fixed Window
#### Schema:
- **Unit Time Division**: Time frame divided into fixed intervals (e.g., seconds).
- **Threshold**: Maximum number of requests allowed within a unit time.

### 4. Sliding Window Log
#### Schema:
- **Unit Time Division**: Time intervals stored in Redis.
- **Threshold**: Maximum number of requests allowed in a time frame.

### 5. Sliding Window Counter
#### Schema:
- **Current Time Window**: Weight (30%).
- **Previous Time Window**: Weight (70%).
- **Threshold Level**: Example: 7 requests per minute.

#### Calculation Example:
```
allow_req = (prev_window_req_count * 0.70) + current_window_req_count
allow_req = (5 * 0.70) + 3
allow_req = 3.5 + 3 = 6.5 (round down) => 6 < 7
** Request Accepted **
```

## High-Level Design (HLD)
- A **draw.io diagram** is recommended to visualize the architecture. *(TODO)*

## Rate Limiting Rules (Inspired by Lyft)
- **Domain**: `auth`
- **Descriptors**:
  ```
  key: auth_type
  value: login
  unit: minute
  request_per_unit: 5
  ```

## Rate Limit Headers (Response HTTP Headers)
These headers inform the client about their rate-limiting status:
- `X-Ratelimit-Retry-After`: Time after which the client can retry.
- `X-Ratelimit-Limit`: Total number of requests allowed.
- `X-Ratelimit-Remaining`: Remaining requests available in the current window.

## Considerations in Distributed Environments

### 1. Race Conditions
- Mitigated using **Lua scripts** and **Redis sorted sets**.

### 2. Synchronization Issues
- Handled via **Redis Master-Slave Replication**.

## Performance Optimizations

### 1. Geographical Location Deployment
- Reduce latency by deploying rate limiters closer to users geographically.
- Achieved via **consistent hashing**.

## Monitoring
- Continuous monitoring of rate-limiting metrics to ensure system health and performance.
