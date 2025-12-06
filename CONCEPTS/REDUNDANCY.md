# Redundancy

Redundancy is the practice of **duplicating critical components or functions so that if one fails, another can take over—eliminating single points of failure**. Each layer of your architecture (web servers, application servers, databases, load balancers, network paths, even data centers) has backup capacity ready to handle traffic if something breaks.

## Building Redundancy Step-by-Step

**Stage 1: <a href="../SETUPS/SINGLE_SERVER_SETUP.md" target="_blank">Single Server</a>**

Everything runs on one box—web server, application code, database. If that machine dies, you're offline. No redundancy exists.

**Stage 2: Separate the Database**

You move your database to its own server. Now a web server crash doesn't kill your data, but you still have single points of failure at both tiers. It's slightly better for maintenance and resource isolation, but not truly redundant.

**Stage 3: Add a Second Web Server + Load Balancer**

You introduce a load balancer in front of two web servers. If one web server fails, traffic routes to the other. You now have redundancy at the web tier, but your load balancer and database are still single points of failure.

**Stage 4: Database Replication**

You set up a primary database with a replica (or multiple replicas). If the primary fails, you can promote a replica. Now you have redundancy at the data tier. However, failover might be manual or slow depending on your setup.

**Stage 5: Redundant Load Balancers**

You deploy a pair of load balancers in an active-passive or active-active configuration, often using a virtual IP that floats between them. If one dies, the other takes over. Now your entry point is redundant.

**Stage 6: Multi-Availability-Zone Deployment**

You spread your redundant components across physically separate data centers (availability zones). A power outage or network failure in one zone doesn't take you down because traffic shifts to components in another zone.

**Stage 7: Multi-Region**

For the most critical systems, you replicate everything across geographically distant regions. A natural disaster or regional outage still leaves you operational.

## Key Principles

At each stage, you're asking: **"What happens if this component fails?"** If the answer is "the system goes down" then you add redundancy there.

The goal is to reach a state where no single failure—hardware, software, network, or even an entire facility—causes an outage.

## References

- [Redundancy (Wikipedia)](https://en.wikipedia.org/wiki/Redundancy_(engineering))
- [High availability and scalability on AWS](https://docs.aws.amazon.com/whitepapers/latest/real-time-communication-on-aws/high-availability-and-scalability-on-aws.html)
- [Patterns for scalable and resilient apps](https://docs.cloud.google.com/architecture/scalable-and-resilient-apps)
- [Single Point of Failure](https://en.wikipedia.org/wiki/Single_point_of_failure)
- [What is load balancing? | How load balancers work](https://www.cloudflare.com/learning/performance/what-is-load-balancing/)
- [What is data replication?](https://www.ibm.com/think/topics/data-replication)
- [Availability Zones: The Complete Guide for 2026](https://www.splunk.com/en_us/blog/learn/availability-zones.html)
