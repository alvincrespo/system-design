# Rate Limiting - API Gateway

<img src="../visuals/rate_limiter_api_gateway.png" alt="Rate Limiter - API Gateway" width="600px"/>

## Request Flow

1. A client sends an HTTP request to the system.
2. The request first reaches the **API Gateway** (e.g., Nginx), which acts as the entry point.
3. The API Gateway applies rate limiting logic:
   - If the request is within the allowed limit, it is forwarded to the backend web servers.
   - If the request exceeds the allowed limit, the API Gateway responds directly with HTTP 429 Too Many Requests.
4. Rejected requests **never reach the backend servers**, protecting them from overload.

## Key Differences from Server-Side

Rate limiting happens at the **infrastructure layer**, outside your application and before requests reach your backend servers. This enables centralized, global enforcement across all your services.

## Advantages

- **Global enforcement**: All backend servers share the same rate limit, ensuring consistent limits regardless of which server a request reaches
- **Early rejection**: Requests exceeding limits are rejected at the gateway, never reaching your backend services
- **Protects backends**: Prevents resource-intensive servers from processing bad requests
- **Centralized control**: Single point to manage and adjust rate limits for your entire system
- **Scales well**: Adding more backend servers doesn't complicate rate limiting enforcement
- **Decoupled from applications**: No need to implement rate limiting logic in each service

## Disadvantages

- **External dependency**: Requires maintaining a separate gateway infrastructure (Nginx, Kong, AWS API Gateway, etc.)
- **Single point of failure**: If the gateway fails and isn't redundant, all requests are blocked
- **Not business-logic aware**: Gateway-level limits can't easily consider application context or specific business rules
- **Network overhead**: All requests pass through an additional network layer
- **Configuration complexity**: Managing rate limits across multiple services and endpoints can become complex

## Trade-offs

You gain the ability to enforce consistent, global rate limits across your entire system, but require external infrastructure and lose some flexibility to customize limits based on application logic.

## References

[Amazon API Gateway - Throttle requests to your REST APIs for better throughput in API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-request-throttling.html)

[Azure - API Management - Advanced request throttling](https://learn.microsoft.com/en-us/azure/api-management/api-management-sample-flexible-throttling)

[Google Cloud - API Gateway - Quotas and limits](https://cloud.google.com/api-gateway/docs/quotas)

[Rate Limiting with NGINX](https://blog.nginx.org/blog/rate-limiting-nginx)

[NGINX Limit Request Module](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html)

[Deploying NGINX as an API Gateway, Part 2: Protecting Backend Services](https://www.f5.com/company/blog/nginx/deploying-nginx-plus-as-an-api-gateway-part-2-protecting-backend-services)

[mod_ratelimit - Apache HTTP Server Version 2.4](https://httpd.apache.org/docs/2.4/mod/mod_ratelimit.html)

[API Rate Limiting: Beginner's Guide](https://konghq.com/blog/learning-center/what-is-api-rate-limiting)
