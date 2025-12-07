# Rate Limiting - Server Side

<img src="../visuals/rate_limiter_server_side_middleware.png" alt="Rate Limiter - Server Side" width="600px"/>

## Request Flow

1. A user starts by sending a request from their client (e.g., web browser or mobile app).
2. An HTTP request is sent to the server.
3. The web server processes the request against the rate limiter.
4. Rate limiting logic is applied:
    - If the request is within the allowed limit, the server processes the request.
    - If the request exceeds the allowed limit, the server responds with a rate limit error (e.g., HTTP 429 Too Many Requests).

## Key Differences from API Gateway

Rate limiting happens **inside your application code** (middleware layer), not at an external gateway. This means each server instance enforces its own rate limits independently, making it difficult to enforce global limits across multiple instances.

## Advantages

- **Simple to implement**: No external infrastructure needed; rate limiting logic lives in your application code
- **Fast**: No network calls required; checks happen in-memory within your application
- **Business-logic aware**: Can incorporate application context when making rate-limit decisions
- **Flexible**: Easily customize limits based on specific endpoints or application features

## Disadvantages

- **Per-instance limits**: Each server instance has its own rate limit counter, not shared across instances
- **Difficult to scale**: When you scale horizontally (add more servers), enforcing a global limit becomes complex
- **Resource consumption**: Rejected requests still reach your server and consume processing resources before being rejected
- **Inconsistent enforcement**: A user can potentially exceed the intended global limit by distributing requests across multiple server instances

## Trade-offs

You gain simplicity and direct control over the logic, but sacrifice the ability to enforce consistent, global rate limits across your distributed system.

## References

[Application-Layer Rate Limiting](https://stack.convex.dev/rate-limiting)

[Rate limiting middleware in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/performance/rate-limit?view=aspnetcore-10.0)

[Slow Down! Rate Limiting Deep Dive in Java](https://www.codereliant.io/p/rate-limiting-deep-dive)

[Building a Zero-Dependency Rate Limiter in Go (Token Bucket, Leaky Bucket, Sliding Window)](https://dev.to/makennsky/building-a-zero-dependency-rate-limiter-in-go-token-bucket-leaky-bucket-sliding-window-134n)
