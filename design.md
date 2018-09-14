Design
======

## Motivation


## Type notation

```
# sync: req -> rsp
Rsp call(Req)

# async cps: req -> cb -> Unit
void call(Req, CB<Rsp>)

# async defered: req -> defer<req>
Defer<Rsp> call(Req)
```

```
#### How invocation looks like

asyncCPS(Req{},[](Rsp){...})

asyncDef(Req{})
  .success([](Rsp){...});

#### Not so different after all? Consider chain call

asyncCPS1(Req{}, [](Rsp){
        asyncCPS2(rsp, [](Rsp2){
            asyncCPS3(rsp2, [](Rsp3){
                ... recursive and non composable!
            })
        });
    });

asyncDef(Req{})
    .then(asyncDef2)
    .then(asyncDef3)
    .success(...) # happy smiley
    .fail(...) # dead head
```

## Asynchronous non-mt

## Proposed syntax

```c++
template<typename Result, typename Error=void>
class Defer {
public:
    // movable only

    template<typename... Args>
    void resolve(Args&&...);

    template<typename... Args>
    void reject(Args&&...);

    template<typename Fn>
    Defer<Result, Error>& success(Fn&&);

    template<typename Fn>
    Defer<Result, Error>& fail(Fn&&);

    template<typename... Args>
    static Defer<Result, Error> resolved(Args&&...);

    template<typename... Args>
    static Defer<Result, Error> failed(Args&&...);

    template<typename NewResult, typename Fn>
    Defer<NewResult, Error> then(Fn&&);

    template<typename Fn>
    Defer<Result, Error> otherwise(Fn&&);
};

template<typename Result, typename Error=void>
class Promise {
public:
    // copyable

    Defer<Result, Error> defer();
};

```

## Usage examples

```
#### Convert Defered to CPS

void asyncCPS(Req r, CB<Rsp> cb) {
    // some time later cb(Rsp{})
}

Defer<Rsp> asyncDef(Req r) {
    Promise<Rsp> p;
    asyncCPS(r, [p](Rsp r){
        p.resolve(r);
    });
    return p.defer();
}

#### Convert CPS to Defered - even easier

Defer<Rsp> asyncDef(Req r) {
    // meat
}

void asyncCPS(Req r, CB<Rsp> cb) {
    asyncDef(r)
        .success(cb)
        .fail(die); # now we can handle errors properly
}

#### Composition

#### Example: gather

# some fn which called async (return nothing); has to wait async on their completion
Defer<void> initXXX() 

list<Defer<void>> -> Defer<void>

# count up and fire then complete
Defer<void> join(initialization_list<Defer> il) {
    Promise<void> p;
    shared_int counter = il.size();
    for(d: il) d.success([=](){
        if (--counter == 0) p.resolve();
    });
    return p.defer();
}

# and with error handling early-out
Defer<void> join(initialization_list<Defer> il) {
    Promise<void> p;
    shared_int counter = il.size();
    for(d: il) d.success([=](){
        if (--counter == 0) p.resolve();
    }).fail([=](){
        p.fail();
    });
    return p.defer();
}


# usage
join({svc1.init(), svc2.init(), svc3.init(), ...})
  .success(set_ready)
  .fail(die)

#### Example: fan-out and reduce

AsyncOp = Defer<Rsp> -> Req
Reduce = list<Defer<Rsp>> -> (Rsp -> list<Rsp> -> list<Rsp>) -> Defer<list<Rsp>>

Defer<Rsp> process(Req)

Defer<SumOfRsp> reduce(list<Defer<Rsp>> li, sumOp) {
    Promise<SumOfRsp> p;
    shared<SumOfRsp> sum;
    shared_int counter = li.size();
    for(e: li) p.success([=](Rsp r){
        sum = sumOp(sum, r);
        if (--counter == 0) p.resolve(sum);
    });
    return p.defer();
}
```

## References

  * [CSP Book, Tony Hoare](http://www.usingcsp.com/cspbook.pdf)
  * [Futures done right, Bartosz Milewski](https://bartoszmilewski.com/2009/03/10/futures-done-right/)
  * [Broken promises, Bartosz Milewski](https://bartoszmilewski.com/2009/03/03/broken-promises-c0x-futures/)
  * [I see a monad in your future, Bartosz Milewski](https://bartoszmilewski.com/2014/02/26/c17-i-see-a-monad-in-your-future/)
  * [WG21 N3721 proposal](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3721.pdf)
  * [ES6 Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
  * [Monadic operations for std::optional P0798R0](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/p0798r0.html)
  * [Monadic interface P0650R1](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/p0650r1.pdf)