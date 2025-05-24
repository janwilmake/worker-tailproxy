Attempt at making a proxy that proxies a request to another worker, but also tails that worker during the lifetime of the request to collect errors, warnings, and logs, effectively simplifying the API

[![](https://b.lmpify.com/Iteration_1)](https://lmpify.com/what-api-does-wrang-xqit6l0)


This worker:

1. Extracts required headers from the incoming request (script name, origin, account ID, and authorization)
2. Validates that all required headers are present
3. Creates a new request without the custom headers
4. Establishes a tail connection using the Cloudflare API (`/accounts/{account_id}/workers/scripts/{script_name}/tails`)
5. Collects logs via WebSocket during the request
6. Forwards the request to the target worker
7. Waits for the entire response body to be processed
8. Adds all logs to the `x-tail` response header as a JSON array
9. Cleans up by deleting the tail after the request is complete (`/accounts/{account_id}/workers/scripts/{script_name}/tails/{id}`)

The worker handles errors gracefully and ensures that the tail is deleted even if an error occurs. The code uses TypeScript comments for type checking and follows the modern Cloudflare Workers syntax with `export default { fetch }`.

[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/janwilmake/worker-tailproxy/tree/main) <!-- for easy deployment, ensure to add this into the readme of the created project -->

Limitations: since this worker waits for the entire response body to be processed, apis that normally stream will be slower, and apis that have large throughput (over ±40mb) won't work. This could potentially be solved using a [Trailer](https://reader.llmtext.com/md/developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Trailer)

# Usage


To use this worker, you need to make a request with the required headers:

```bash
curl -X POST "https://worker-tailproxy.[your-worker-subdomain].workers.dev/" \
  -H "x-script: target-worker-name" \
  -H "x-origin: https://target-worker.[your-worker-subdomain].workers.dev" \
  -H "x-account-id: your-cloudflare-account-id" \
  -H "x-cloudflare-authorization: Bearer your-cloudflare-api-token" \
  -H "Content-Type: application/json" \
  -d '{"example": "payload"}'
```

The API token needs to have the "Workers Tail" permission for the specified account. The response will include all logs collected during the request in the `x-tail` header.

# UI

[![](https://b.lmpify.com/Making_A_UI)](https://lmpify.com/httpsuithubcomj-wwbl3d0)

TODO:

- Create authenticated cors-proxy (useful for anything)
- Test UI and make it work, getting logs for requests to any worker, anywhere
- Discuss why not OTEL ()
- Need for tracing in service bindings too? https://lmpify.com/httpsuithubcomj-x6fpv60 also, check Durable Object logging support with the tail api.