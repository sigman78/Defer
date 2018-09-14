Defered
=======

Defered is a C++17 library for non-blocking variant of std::future and a piece of monadic-like sugar.

## Documentation

See (design.md) for now as the source of inspiration and design notes.

## Example

```c++
// Async dispatcher (where defered result is materialized)
Defer<Response> asyncOp(Request req) {
    Promise<Response> promise;
    auto reqId = startReq(req);
    addCompletionHandler(reqId, [promise, reqId](){ promise.resolve(getResult(reqId)); });
    return promise.defer();
}
```

## License

This library is available to anybody free of charge, under the terms of MIT License (see LICENSE.md).