# Node.js N-API for Rust! [work in progress]

[![Build Status][travis-badge]][travis-url]

High-level N-API bindings for Node.js addons written in Rust.

**Warning**: this is a proof-of-concept implementation that's not intended
for use yet. The project is under initial phase of development, the API is a
quite sketchy and is going to be refactored heavily. If you are interested in
contributing, though, it is super welcome!

The project is covered by a [Code of Conduct][coc].

## Crates

* [`napi-sys`][napi-sys]: low-level bindings to N-API generated from
  [`node_api.h`](https://github.com/nodejs/node/blob/master/src/node_api.h)
  using [`bindgen`](https://github.com/rust-lang-nursery/rust-bindgen).
* [`napi`][napi]: high-level and rusty wrappers around `napi-sys`.
* [`napi-derive`][napi-derive]: contains a procedural macro that allows to
  construct typesafe structures that represent N-API callback parameters and
  automatically validate the arguments that JavaScript code passes in.

## Example

Check out <https://github.com/aqrln/napi-rs/tree/master/example> to see the
full source code and project structure of this example.

### `lib.rs`

```rust
#[macro_use]
extern crate napi;
#[macro_use]
extern crate napi_derive;

use napi::{NapiEnv, NapiNumber, NapiResult, NapiUndefined};

#[derive(NapiArgs)]
struct HelloArgs;

fn hello<'a>(env: &'a NapiEnv, _: &HelloArgs) -> NapiResult<NapiUndefined<'a>> {
    println!("Hello from the Rust land!");
    NapiUndefined::new(env)
}

#[derive(NapiArgs)]
struct AddArgs<'a> {
    first: NapiNumber<'a>,
    second: NapiNumber<'a>,
}

fn add<'a>(env: &'a NapiEnv, args: &AddArgs<'a>) -> NapiResult<NapiNumber<'a>> {
    let first = args.first.to_i32()?;
    let second = args.second.to_i32()?;
    NapiNumber::from_i32(env, first + second)
}

napi_callback!(example_hello, hello);
napi_callback!(example_add, add);
```

### `example.js`

```javascript
'use strict';

const addon = require('./build/Release/example.node');

addon.hello();
console.log(addon.add(1, 2));
```

[coc]: https://github.com/aqrln/napi-rs/blob/master/CODE_OF_CONDUCT.md
[napi]: https://crates.io/crates/napi
[napi-derive]: https://crates.io/crates/napi-derive
[napi-sys]: https://crates.io/crates/napi-sys
[travis-badge]: https://travis-ci.org/aqrln/napi-rs.svg?branch=master
[travis-url]: https://travis-ci.org/aqrln/napi-rs
