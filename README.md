# go-gRPC best practices

TODO: Gather all the best practices for gRPC with `golang`.
<!--
## How to separate private and public calls

## How to map structs

## How to add inline tags/modify existing tags/carry out validation

## How to validate a request is idempotent?

Add timestamp to the request

```go
message Request {
  string from = 1;
  string to = 2;
  float amount = 3;
  int64 timestamp = 4; // or GUID
}
message Response {
  // Repeated calls return the same result
  int64 confirmation = 1;
}
```

## How to improve performance?

Add pagination in grpc to limit.

Avoid long running operations - the longer it takes, the more likely it will need to be retried.

Add timeout in header.

## Define default value

## Do not return error blindly

Inspect and translate errors instead. Is it retryable?

## Deadlines 

Setting a timeout:

```go
ctx, cancel := context.WithTimeout(context.Background(), 5 * time.Second)
defer cancel()

res, err := client.Call(ctx, req)
```

Setting a deadline:

```go
ctx, cancel := context.WithDeadline(context.Background(), time.Unix(1481231232, 0))
defer cancel()

res, err := client.Call(ctx, req)
```

## Deadlines Propagation
Don't reuse context


```go
func (s *Server) MyRequestHandler (ctx context.Context, ...) (*Res, error) {
  d, _ := ctx.Deadline()
  ctx1, cancel := context.WithDeadline(ctx, d.Add(-150 * time.Millisecond))
  client1.Call(ctx1, ...)


  ctx2, cancel := context.WithDeadline(ctx, d.Add(-150 * time.Millisecond)))
  client1.Call(ctx2, ...)
}
```

## Rate-limiting
Server:

```go
import "golang.org/x/time/rate"

s := grpc.NewServer(grpc.InTapHandler(rateLimiter))

func rateLimiter(ctx context.Context, info *tap.Info) (context.Context, error) {
  if m[user] == nil {
    m[user] = rate.NewLimiter(5, 1) // QPS, burst
  }
  if !m[user].Allow() {
    return nil, status.Errorf(codes.ResourceExhausted, "client exceeded rate limit")
  }
  return ctx, nil
}
```
Implement local rate limiting:
Client:
```go

import "golang.org/x/time/rate"

var limiter = rate.NewLimiter(5, 1) // QPS, burst

func MyHandler(ctx context.Context, req Request) (Response, err) {
  if err := limiter.Wait(ctx); err != nil {
    retunr nil, err
  }
  return c.Call(ctx, req)
}
```

## Retries 

gRFC A6

## Memory management

gRPC-Go does not limit concurrent server goroutines

Option 1: Set listener limit and concurrent stream limit
```go
import "golang.org/x/net/netutil"

listener = netutil.LimitListener(listener, connectionLimit)
grpc.NewServer(grpc.MaxConcurrentStreams(streamsLimit))
```

Option 2: Use tap handler to error when too many rpcs are in flight / or when system memory is too low

option 3: Use health reporting and load balancers to redirect traffic


Set a maximum request payload size
```go
grpc.NewServer(grpc.MaxRecvMsgSize(4096)) // bytes
```-->