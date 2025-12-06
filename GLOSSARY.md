# Glossary

## Table of Contents

- [Metrics](#metrics)
  - [Availability](#availability)
  - [Back-of-the-Envelope Calculation](#back-of-the-envelope-calculation)
  - [Latency](#latency)
    - [Latency Numbers Every Programmer Should Know](#latency-numbers-every-programmer-should-know)
    - [Power of Two](#power-of-two)
  - [Software License Agreement (SLA)](#software-license-agreement-sla)
  - [Query's Per Second (QPS)](#querys-per-second-qps)

## Metrics

### Availability

The proportion of time a system is operational and accessible when required for use. It is often expressed as a percentage of uptime over a given period.

| Availability | Downtime/Day | Downtime/Week | Downtime/Month | Downtime/Year |
|--------------|--------------|---------------|----------------|---------------|
| 99% | 14m 24s | 1h 41m | 7h 18m | 3d 15h 36m |
| 99.9% | 1m 26s | 10m 5s | 43m 50s | 8h 46m |
| 99.99% | 8.6s | 1m 0s | 4m 23s | 52m 36s |
| 99.999% | 864ms | 6s | 26s | 5m 16s |
| 99.9999% | 86ms | 604ms | 2.6s | 31.5s |

### Back-of-the-Envelope Calculation

A rough calculation or estimate made quickly with minimal data, often used for initial assessments or feasibility studies.

*Source: Jeff Dean, Google* [Building Software Systems At Google and Lessons Learned](https://youtu.be/modXC5IWTJI?si=hyVLZ-55y_0ph16g)

### Latency

The time it takes for a system to respond to a request, typically measured in milliseconds (ms). It reflects the responsiveness of the system.

#### Latency Numbers Every Programmer Should Know

| Operation | Time |
|-----------|------|
| L1 cache reference | 0.5 ns |
| Branch mispredict | 5 ns |
| L2 cache reference | 7 ns |
| Mutex lock/unlock | 25 ns |
| Main memory reference | 100 ns |
| Compress 1KB with Zippy | 3,000 ns (3 µs) |
| Send 1KB over 1 Gbps network | 10,000 ns (10 µs) |
| Read 4KB randomly from SSD | 150,000 ns (150 µs) |
| Read 1MB sequentially from memory | 250,000 ns (250 µs) |
| Round trip within same datacenter | 500,000 ns (500 µs) |
| Read 1MB sequentially from SSD | 1,000,000 ns (1 ms) |
| Disk seek | 10,000,000 ns (10 ms) |
| Read 1MB sequentially from disk | 20,000,000 ns (20 ms) |
| Send packet CA → Netherlands → CA | 150,000,000 ns (150 ms) |

*Source: Jeff Dean, Google* [Building Software Systems At Google and Lessons Learned](https://youtu.be/modXC5IWTJI?si=hyVLZ-55y_0ph16g)

**Legend:**
- **ns** = nanoseconds (10⁻⁹ seconds)
- **µs** = microseconds (10⁻⁶ seconds)
- **ms** = milliseconds (10⁻³ seconds)

#### Power of Two

A mathematical concept where numbers are expressed as 2 raised to an exponent. Commonly used in computing to represent memory sizes and data quantities.

| Power | Exact Value | Approx Value | Bytes |
|-------|-------------|--------------|-------|
| 7 | 128 | | |
| 8 | 256 | | |
| 10 | 1,024 | 1 thousand | 1 KB |
| 16 | 65,536 | | 64 KB |
| 20 | 1,048,576 | 1 million | 1 MB |
| 30 | 1,073,741,824 | 1 billion | 1 GB |
| 32 | 4,294,967,296 | | 4 GB |
| 40 | 1,099,511,627,776 | 1 trillion | 1 TB |
| 50 | 1,125,899,906,842,624 | 1 quadrillion | 1 PB |

### Software License Agreement (SLA)

A formal contract between a service provider and a customer that defines the level of service expected, including performance metrics, responsibilities, and remedies for service failures.

[Amazon Compute - Service Level Agreement (SLA)](https://aws.amazon.com/compute/sla/)
[Google Cloud - Compute Engine Service Level Agreement (SLA)](https://cloud.google.com/compute/sla)
[Microsoft Azure - Service Level Agreements (SLA) for Online Services](https://azure.microsoft.com/en-us/support/legal/sla/)

### Query's Per Second (QPS)

The number of queries processed by the system per second. It is a measure of the system's throughput.

---
