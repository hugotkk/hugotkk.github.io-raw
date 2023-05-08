---
title: "Understanding gRPC: Key Concepts and Examples"
date: 2023-05-08
tags:
- grpc
---

## Workflow of RPC server

1. Receive a RPC request
2. Parse request into message, context
3. Service interceptor
4. RPCMethodHandler
5. Give RPC response [metadata (optional), message protobuf object / raise exception]

Metadata has to be sent explicitly.

## Steps to work on RPC client-server

### Server
- Create RPC server object
- Create servicer object
- Add middleware to server
- Add servicer to RPC
- Start the server

### Client
- Create channel object
- Add middleware to channel
- Send RPC request with stub object

### Ping-pong between client-server:

- In bidirectional streaming RPC, the call is initiated by the client invoking the method, and the server receives the client metadata, method name, and deadline
- The server can choose to send back its initial metadata or wait for the client to start streaming messages
- Client and server-side stream processing is application-specific
- Since the two streams are independent, the client and server can read and write messages in any order
- For example, a server can wait until it has received all of a client’s messages before writing its messages, or the server and client can play “ping-pong”
- This is similar to a WebSocket, providing a live chat between client and server from the stub (channel)

Example flow:
1. Client yields request_stream1
2. Server receives request_stream1
3. Server yields result_stream1
4. Client yields stream2
5. Server receives stream2
6. Server yields result_stream2

- The client keeps yielding stream, and the server iterates through the stream
- The server retrieves and stores each stream until `iterator.next() == null`
- The server responds with a stream message and ends the stream

When sending two stream objects:
- It is still considered a single request
- Multiple messages are sent through the channel (stub)
- Each stream message is sent to the same target server (behind ALB)
- The channel can last a long time unless the client or server closes it, passes the `deadline`, or reaches the `idle_timeout`
- When streaming, it follows a consumer and producer pattern
- In Client streaming RPC, the client acts as a producer and server as a consumer
- In Bidirectional streaming RPC, both client and server act as producers and consumers simultaneously

Reference: https://www.oreilly.com/library/view/grpc-up-and/9781492058328/ch04.html

## My Lab to Experience gRPC

For experiencing gRPC, I created a [simple client and server](https://gist.github.com/hugotkk/9438baf731a53fc56451ed0d861c9686) to manage the records of a message "video."

Install the basic packages needed:
pip3 install grpcio grpcio-tools grpcio_reflection

To start the server:
```
python3 server.py
```

To compile the proto:
```
python3 -m grpc_tools.protoc -I. --python_out=. --pyi_out=. --grpc_python_out=. video_service.proto
```

To send requests to the server:
```
export SERVER=localhost:50051
```
Unary - stream:
```
grpcurl -proto video_service.proto -plaintext $SERVER video_service.VideoService/ListVideos
```
Unary - (with param):
```
grpcurl -proto video_service.proto -plaintext -d '{"id":"2", "title": "Sample Video", "author": "John Doe", "url": "https://example.com/sample.mp4", "duration": 120}' $SERVER video_service.VideoService/AddVideo
grpcurl -proto video_service.proto -plaintext -d '{"id":"1", "title": "Sample Video", "author": "John Doe", "url": "https://example.com/sample.mp4", "duration": 120}' $SERVER video_service.VideoService/AddVideo
```
Stream:
```
grpcurl -proto video_service.proto -plaintext -d @ $SERVER video_service.VideoService/GetVideos << EOF
{"id":"2"}
{"id":"1"}
EOF
grpcurl --plaintext $SERVER list
```

Interactive way to send a stream:

```
grpcurl --proto video_service.proto -insecure -vv -d @ $SERVER video_service.VideoService/GetVideo
{"id":"2"}
wait for the response for video 2
{"id":"1"}
wait for the response for video 1
```

## Examples

There are some [examples](https://github.com/grpc/grpc/tree/master/examples/python) worth reading from the gRPC library repository.

### Concepts related

Health checking:
- Special servicers that require an extra package
- Can service health checks with its own thread pools, not affecting the thread pool of other servicers

Interceptors:

    Server Interceptors:
    - Modify the request from the client before processing.
    - Interrupt each received RPC request.
    - Call `continuation` to proceed.
    - Return a handler to override the default handler.

    Client Interceptors:
    - Invoked before the RPC request is sent and after the RPC response is received.
    - Add a middleware to the channel, which can extend multiple per-RPC-type client interceptors. Implement a function per RPC type.
    - Inside the functions, modify the RPC request from `ClientCallDetails`, then call `continuation`. This will send the RPC call, wait for the response from the server, and then return the RPC response. The response can be modified if desired.

Metadata:
- Similar to headers in HTTP

Multiplex:
- One channel with multiple stubs
- One server with multiple services

Route guide:
- Best example to understand unary-unary, unary-stream, stream-unary, and stream-stream rpc requests

### Options / Settings related

- Keepalive
- Timeout
- Load balancing policies
- Retry
- Wait for ready - send the request when the service is not busy
- Compression - gzip

## ALB

Sticky sessions can be tricky:
- Sticky sessions are used when there are many clients that keep requesting the service and want to reuse connections.

Sticky sessions in gRPC are not ideal when working with load balancing:
- Load balancing will not work effectively, as it causes traffic to be sent to the same instance.
- If there are limited clients but each sends a lot of requests, we want the requests to distribute evenly to the backends. However, sticky sessions cause the same client's requests to be sent to the same instance, overloading the backend.

For more information on gRPC load balancing, visit:
https://majidfn.com/blog/grpc-load-balancing/

### Health Check

gRPC is an HTTP request with a binary payload:
- It uses POST requests and the URI is the method (e.g., video_service.VideoService/ListVideo).
- Status codes: 0 is normal; 12 is not implemented.
