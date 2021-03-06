Iron [![Build Status](https://secure.travis-ci.org/iron/iron.png?branch=master)](https://travis-ci.org/iron/iron)
====

> Extensible, Concurrency Focused Web Development in Rust.

## Response Timer Example

```rust
struct ResponseTime;

impl Key for ResponseTime { type Value = u64 }

impl BeforeMiddleware for ResponseTime {
    fn before(&self, req: &mut Request) -> IronResult<()> {
        // Set the current time for retrieval later.
        req.extensions.insert::<ResponseTime>(precise_time_ns());
        Ok(())
    }

    fn catch(&self, req: &mut Request, _: &mut IronError) {
        // On an error just do the same thing.
        let _ = self.before(req);
    }
}

impl AfterMiddleware for ResponseTime {
    fn after(&self, req: &mut Request, res: Response) -> IronResult<Response> {
        // Get the time we set earlier, compare it to now.
        let delta = precise_time_ns() - *req.extensions.find::<ResponseTime>().unwrap();
        println!("Request took: {} ms", (delta as f64) / 1000000.0);
        Ok(res)
    }

    fn catch(&self, req: &mut Request, _: &mut IronError) {
        let delta = precise_time_ns() - *req.extensions.find::<ResponseTime>().unwrap();

        // Print something different on errors.
        println!("Request errored, and took: {} ms", (delta as f64) / 1000000.0);
    }
}

fn main() {
    // Create our Handler
    let chain = Chain::new(|&: _: &mut Request| {
        // Send back 200, "Hello World"
        Ok(Response::with((status::Ok, "Hello World!")))
    });

    // Add our response timer.
    chain.link((ResponseTime, ResponseTime));

    // Kick off serving.
    Iron::new(chain).listen("localhost:3000").unwrap();
}
```

## Overview

Iron is a high level web framework built in and for Rust, built on
[hyper](https://github.com/hyperium/hyper). Iron is designed to take advantage
of Rust's greatest features - it's excellent type system and its principled
approach to ownership in both single threaded and multi threaded contexts.

Iron is highly concurrent and can scale horizontally on more machines behind a
load balancer or by running more threads on a more powerful machine. Iron
avoids the bottlenecks encountered in highly concurrent code by avoiding shared
writes and locking in the core framework.

Iron is 100% safe code:

```sh
$ ack unsafe src | wc
       0       0       0
```

## Philosophy

Iron is meant to be as extensible and pluggable as possible; Iron's core is
concentrated and avoids unnecessary features by leaving them to middleware,
plugins, and modifiers.

Middleware, Plugins, and Modifiers are the main ways to extend Iron with new
functionality. Most extensions that would be provided by middleware in other
web frameworks are instead addressed by the much simpler Modifier and Plugin
systems.

Modifiers allow external code to manipulate Requests and Response in an ergonomic
fashion, allowing third-party extensions to get the same treatment as modifiers
defined in Iron itself. Plugins allow for lazily-evaluated, automically cached
extensions to Requests and Responses, perfect for parsing, accessing, and
otherwise lazily manipulating an http connection.

Middleware are only used when it is necessary to modify the control flow of a
Request flow, hijack the entire handling of a Request, check an incoming
Request, or to do final post-processing. This covers areas such as routing,
mounting, static asset serving, final template rendering, authentication, and
logging.

Iron comes with only basic modifiers for setting the status, body, and various
headers, and the infrastructure for creating modifiers, plugins, and
middleware. No plugins or middleware are bundled with Iron.

## Performance

Iron averages [84,000+ requests per second for hello world](https://github.com/iron/iron/wiki/How-to-Benchmark-hello.rs-Example)
and is mostly IO-bound, spending over 70% of its time in the kernel send-ing or
recv-ing data.\*

\* *Numbers from profiling on my OS X machine, your milage may vary.*

## Core Extensions

Iron aims to fill a void in the Rust web stack - a high level framework that is
*extensible* and makes organizing complex server code easy.

Extensions are painless to build, and the [core bundle](https://github.com/iron/core)
already includes\*:

Middleware:
- [Routing](https://github.com/iron/router)
- [Mounting](https://github.com/iron/mount)
- [Static File Serving](https://github.com/iron/static)
- [Logging](https://github.com/iron/logger)

Plugins:
- [JSON Body Parsing](https://github.com/iron/body-parser)
- [URL Encoded Data Parsing](https://github.com/iron/urlencoded)
- [Cookies](https://github.com/iron/cookie)
- [Sessions](https://github.com/iron/session)

Both:
- [Shared Memory (also used for Plugin configuration)](https://github.com/iron/persistent)

This allows for insanely flexible and powerful setups and allows nearly all
of Iron's features to be swappable - you can even change the middleware
resolution algorithm by swapping in your own `Chain`.

\* Due to the rapidly evolving state of the Rust ecosystem, not everything
builds all the time. Please be patient and file issues for breaking builds,
we're doing our best.

## Underlying HTTP Implementation

Iron is based on and uses [`hyper`](https://github.com/hyperium/hyper) as its
HTTP implementation, and lifts several types from it, including its header
representation, status, and other core HTTP types. It is usually unnecessary to
use `hyper` directly when using Iron, since Iron provides a facade over
`hyper`'s core facilities, but it is sometimes necessary to depend on it as
well.

<!--
FIXME: expand on when it is necessary to user hyper for serving,
e.g. when doing HTTPS.
-->

## Installation

If you're using `Cargo`, just add Iron to your `Cargo.toml`:

```toml
[dependencies.iron]
version = "*"
```

## [Documentation](http://ironframework.io/doc/iron)

The documentation is hosted [online](http://ironframework.io/doc/iron) and
auto-updated with each successful release. You can also use `cargo doc` to
build a local copy.

## [Examples](/examples)

Check out the [examples](/examples) directory!

You can run an individual example using `cargo run --example example-name`.
Note that for benchmarking you should make sure to use the `--release` flag,
which will cause cargo to compile the entire toolchain with optimizations.
Without `--release` you will get truly sad numbers.

## Getting Help

Feel free to ask questions as github issues in this or other related repos.

The best place to get immediate help is on IRC, on any of these channels on the
mozilla network:

- `#rust-webdev`
- `#iron`
- `#rust`

One of the maintainers or contributors is usually around and can probably help.
We encourage you to stop by and say hi and tell us what you're using Iron for,
even if you don't have any questions. It's invaluable to hear feedback from users
and always nice to hear if someone is using the framework we've worked on.

## Maintainers

Jonathan Reem ([reem](https://github.com/reem)) is the core maintainer and
author of Iron.

Commit Distribution (as of `8e55759`):

```
Jonathan Reem (415)
Zach Pomerantz (123)
Michael Sproul (9)
Patrick Tran (5)
Corey Richardson (4)
Bryce Fisher-Fleig (3)
Barosl Lee (2)
Christoph Burgdorf (2)
da4c30ff (2)
arathunku (1)
Cengiz Can (1)
Darayus (1)
Eduardo Bautista (1)
Mehdi Avdi (1)
Michael Sierks (1)
Nerijus Arlauskas (1)
SuprDewd (1)
```

## License

MIT

