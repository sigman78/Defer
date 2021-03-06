Defer
=======

Defer is a C++17 library for async binding primitive, non-blocking offspring of std::future and a piece of monadic-alike sugar.

## Documentation

See [design document](design.md) for now as the source of inspiration and design notes.

## Example

```c++
// Async dispatcher (where defered result is materialized)
Defer<Response> asyncOp(Request req) {
    Promise<Response> promise;
    auto reqId = server.makeRequest(req);
    server.handleRequest(reqId, [promise, reqId](){ promise.resolve(server.retrieveResult(reqId)); });
    return promise.defer();
}

// Async API sequential composition
Defer<Handle> openFile(string name);
Defer<Bytes> readFile(Handle file);
Defer<JSON> parse(Bytes buffer);

openFile("config.json")
    .then(readFile)
    .then(parse)
    .success([](JSON json) {
        config = json;
    })
    .fail([]() {
        die();
    });
    
```

## License

This library is available to anybody free of charge, under the terms of MIT License (see LICENSE.md).
