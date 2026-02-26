# Exception Handling

If infra isn’t available or a dependency isn’t ready, best practice is to stop infinite retries and switch to bounded retries + backoff + a durable buffer, so we don’t overwhelm the system or lose data.

Below is the pattern set that works well in AWS/EKS microservices.

#### 1) Classify the failure

We want different behavior for each class:

* Transient (timeouts, throttling, brief 5xx): retry is useful.
* Dependency down / infra not available (service endpoint down, DNS fail, auth/secret missing, cluster not ready): retries usually won’t help quickly.
* Permanent (4xx validation, schema mismatch): retries are harmful.
  
Implement this via error mapping (exception types / HTTP codes / MQ/Kafka errors).

#### 2) Use bounded retries with exponential backoff + jitters

Do:

* max attempts (e.g., 3–8 depending on SLA)
* exponential backoff (e.g., 200ms → 400 → 800 → 1.6s…)
* jitter to prevent thundering herd
* cap the max delay (e.g., 30s)

Don’t:

* infinite retry loops in-process
* fixed-interval retries across many pods

#### 3) Add a circuit breaker (fast-fail when infra is down)

When dependency is unavailable, a circuit breaker:

opens after N failures

fails fast for a cool-down window

optionally lets a few “probe” requests through (half-open)

This protects:

your thread pools

CPU

connection pools

downstream dependencies when they recoverteams

(Resilience4j is common for Java; similar patterns exist for other stacks.)

4) Decouple with a durable buffer (so you can “retry later” safely)

If the work must not be lost, don’t keep it in memory while retrying.

Use one of these “store-and-forward” patterns:

Kafka/MSK: produce event, process asynchronously; failed processing goes to retry topic / DLQ topic

SQS (if applicable): built-in visibility timeout + DLQ

Transactional Outbox (best for “DB write + publish event” consistency)

This is the cleanest way to survive infra outages without losing requests.

5) Add DLQ + replay (make failures recoverable)

After bounded retries:

send to DLQ (or a “parking lot” topic/queue)

include reason, stack, correlation IDs, payload metadata

build a replay tool/process (manual or automated)

This turns outages into operations instead of incidents.

6) Control concurrency during recovery (avoid surge on comeback)

When infra comes back:

limit worker concurrency

use rate limiting / token bucket

gradually ramp (HPA + KEDA rules can help)

Otherwise you get “recovery storms” and flap again.

7) Be explicit in readiness/liveness + startup behavior

If a service cannot operate without a dependency:

readiness probe should fail until critical deps are reachable (so traffic doesn’t route)

but don’t crash-loop just because a dependency is down—crash loops amplify outages

Common approach:

liveness: “process is healthy”

readiness: “can serve requests now”

8) Observability is part of the solution

You need:

retry counters

circuit breaker state

DLQ volume and age

dependency error rates

dashboards + alerts (and runbooks)

Practical default you can standardize (good enterprise baseline)

For sync calls (HTTP/gRPC):

retries: 3 attempts max

backoff: exp + jitter, max 5–10s

circuit breaker: open after ~10 failures / 30s, cool-down 30–60s

fallback: return 503 quickly + correlation id

For async processing (Kafka/MQ consumers):

no infinite retries in the consumer loop

retry topic with delay (or scheduled retries) + max retry count

DLQ after max retries

idempotency key + dedup store if at-least-once delivery