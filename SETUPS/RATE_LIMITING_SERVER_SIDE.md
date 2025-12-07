# Rate Limiting - Server Side

<img src="../visuals/rate_limiter_server_side_middleware.png" alt="Rate Limiter - Server Side" width="600px" alt="Single Server Setup"/>

## Request Flow

1. A user starts by sending a request from their client (e.g., web browser or mobile app).
2. An HTTP request is sent to the server.
3. The web server processes the request against the rate limiter.
4. Rate limiting logic is applied:
    - If the request is within the allowed limit, the server processes the request.
    - If the request exceeds the allowed limit, the server responds with a rate limit error (e.g., HTTP 429 Too Many Requests).

## References

[Application-Layer Rate Limiting](https://stack.convex.dev/rate-limiting)

[Rate limiting middleware in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/performance/rate-limit?view=aspnetcore-10.0)

[Slow Down! Rate Limiting Deep Dive in Java](https://www.codereliant.io/p/rate-limiting-deep-dive)

[Building a Zero-Dependency Rate Limiter in Go (Token Bucket, Leaky Bucket, Sliding Window)](https://dev.to/makennsky/building-a-zero-dependency-rate-limiter-in-go-token-bucket-leaky-bucket-sliding-window-134n)
