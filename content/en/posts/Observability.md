+++
date = '2026-01-23T16:43:38+08:00'
draft = 'False'
title = 'Observability'
categories = ["Observibility"]
tags= ["Observibility"]
series= [""]
ShowToc = true
TocOpen = true
+++

# Observability

![](https://i.imgur.com/FEPLgvU.png)
[source](https://developer.ibm.com/articles/observability-insights-and-automation/)

**Observability** is the practice of instrumenting a system to collect data that allows you to understand its internal state from the outside.

In modern complex environments like microservices, we need layered observability help us to tell the status from our business to the infrastructure underneath. Observability gives you the power to explore and debug problems you've never seen before ("unknown unknowns") by asking arbitrary questions about your system's behavior.

The pyramid clear depict a common business story from finding a service is down to IT need to deep dive to investigate the incident.

To achieve this, applications must be instrumented to emit **telemetry**—data about their performance and behavior. This data is typically categorized into three foundational pillars.

## The Three Pillars of Observability

![](https://i.imgur.com/OU4NjDB.png)

[Source](https://developer.ibm.com/articles/observability-insights-and-automation/)

While metrics, logs, and traces are often called the "three pillars," their true power is unlocked when they are correlated, allowing you to move seamlessly between them to diagnose issues.

### 1. Metrics: The "What"
Metrics are aggregated, numerical data captured over a period of time. They are excellent for understanding the overall health of a system at a glance and for setting up alerts. ex. CPU usage、API response time

*   **What they answer:** "What is the CPU usage?", "What is the error rate of my API?", "How many requests per second are we handling?"

*   **Key Concepts:**
    *   **Dimensionality (維度 ):** Metrics are more powerful when they have dimensions (also called labels or tags), which are key-value pairs that add context. For example, instead of just `http_requests_total`, you could have `http_requests_total{method="POST", status="500"}`.

    *   **Cardinality (基數) :** This refers to the number of unique combinations of dimensions. **High cardinality** (e.g., using a unique `userID` as a dimension) can be challenging for monitoring systems to store and query efficiently. As the number of time series grows, queries become more computationally expensive. Additionally, any significant event, e.g., a code release involving immutable infrastructure, will result in a flood of simultaneous writes to the database.

*   **Typical Tool:** **Prometheus** is a leading open-source tool for collecting and storing time-series metric data.

### 2. Logs: The "Why"
A log is a timestamped, immutable record of a discrete event that occurred at a specific time. Logs provide the most detailed, granular context about what was happening inside an application or system. For instance, system error message or exception event

* **What they answer:** "Why did this specific request fail?", "What was the exact error message?", "What sequence of events led to this crash?"

*  **Typical Tool:** **Grafana Loki** is a log aggregation system designed to be cost-effective and easy to operate, especially when integrated with other tools like Prometheus.

### 3. Traces: The "Where"
A trace represents the end-to-end journey of a single request as it moves through all the different services in a distributed system. Spans track specific operations that a request makes, painting a picture of what happened during the time in which that operation was executed.

ex. 在 iThome 使用 Facebook 的 SSO（Single Sign-On），一個單純的登入 “Request”，可能就橫跨了 iThome 的 Backend Service、Facebook 的 SSO Backend、iThome 的 Redis，這些衍生出來的 Requests，跟最初的登入 Request 結合起來呈現的歷程就稱為 Trace

*   **What they answer:** "Where is the latency in this user's request?", "Which downstream service is causing the bottleneck?", "What is the full path of this API call through our microservices?"
*   **Typical Tool:** **Grafana Tempo** is a distributed tracing backend that can store and retrieve traces from various sources.

* Each unit of work within a trace is called a **span**.
	1. **A large number of spans within a single trace:**
    This would indicate a request that traverses many different services or performs a significant number of operations. While not inherently a problem, a trace with a very large number of spans could imply a highly complex interaction or potentially an inefficient design if too many microservices are involved for a simple request. It might make the visual representation of the trace more dense, but the primary purpose of traces is to map out these complex journeys.
    
	2. **A span with a long duration (i.e., a "high" amount of time spent in that span):**
    This is a direct and crucial insight that traces provide. If a particular span has a long duration, it means that the specific operation or service represented by that span is taking a significant amount of time to complete. This is exactly what traces are designed to help identify:
    
    - "Where is the latency in this user's request?"
    - "Which downstream service is causing the bottleneck?"

    Therefore, a "high span" in terms of duration would pinpoint a performance bottleneck within your system, allowing you to investigate that specific operation or service further, potentially by jumping to its logs to find the root cause.

### Bringing It All Together with Visualization
The ultimate goal is to use these pillars together. A typical workflow might be:
1.  An **alert** fires from a **metric** (e.g., latency is too high).
2.  You look at a **trace** for a slow request to see which specific service is the bottleneck.
3.  You jump from that trace to the **logs** of that service at that exact time to find the root cause error message.

**Grafana** is the de-facto tool for creating dashboards that visualize and correlate data from all three pillars in a single pane of glass.

## Enabling Observability with OpenTelemetry
Manually instrumenting code to produce all this data can be complex. **OpenTelemetry** is a vendor-neutral, open-source observability framework. It provides a standardized set of APIs, libraries, and agents to collect and export telemetry data (metrics, logs, and traces) from your applications, freeing you from being locked into a single vendor's ecosystem.

## Applying Observability: Measuring Service Reliability
![](https://i.imgur.com/VjkRy2U.png)

[differentiating-between-slo-vs-sla-vs-sli](https://www.solarwinds.com/blog/differentiating-between-slo-vs-sla-vs-sli)

Once you have rich telemetry data, you can move beyond just fixing bugs and start measuring what matters to your users and the business. This is done using a framework of SLIs, SLOs, and SLAs.

#### 1. Service Level Indicators (SLIs)
An SLI is a direct, quantifiable measure of a service's performance. It is the raw data you get from your observability tools.
*   **Example:** The percentage of successful HTTP requests, or the latency of 95% of requests.

#### 2. Service Level Objectives (SLOs)
An SLO is the **internal target** you set for an SLI over a period of time. It is the goal your engineering team strives to meet.
*   **Example:** 99.9% of API requests will be successful over a 30-day period.

#### 3. Service Level Agreements (SLAs)
An SLA is a formal, often contractual, commitment to a customer that defines the consequences if SLOs are not met (e.g., financial penalties or service credits). SLAs are typically less strict than internal SLOs to provide a buffer.
*   **Example:** A promise of 99.5% uptime per month, with a 10% bill credit if breached.

This framework connects the technical data from your observability platform directly to business and customer satisfaction goals. # Observability
[Observability](https://opentelemetry.io/docs/concepts/observability-primer/#what-is-observability) lets you understand a system from the outside by ask questions about that system without knowing its inner workings.

# Reference
[Observability primer](https://opentelemetry.io/docs/concepts/observability-primer/)

[Observability at Twitter: technical overview, part I](https://blog.x.com/engineering/en_us/a/2016/observability-at-twitter-technical-overview-part-i)

[Observability at Twitter: technical overview, part II](https://blog.x.com/engineering/en_us/a/2016/observability-at-twitter-technical-overview-part-ii)
[Observability 的過去與現在]
https://ithelp.ithome.com.tw/m/articles/10319113

[Why Your Observability Strategy Needs High Cardinality Data](https://betterstack.com/community/guides/observability/high-cardinality-observability/)

[Understanding High Cardinality in Observability](https://www.observeinc.com/blog/understanding-high-cardinality-in-observability)

