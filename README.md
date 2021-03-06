# hyper-trust-dns-connector

[![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](./LICENSE)
[![crates.io](https://meritbadge.herokuapp.com/hyper-trust-dns-connector)](https://crates.io/crates/hyper-trust-dns-connector)
[![Released API docs](https://docs.rs/hyper-trust-dns-connector/badge.svg)](https://docs.rs/hyper-trust-dns-connector)

A crate to make [trust-dns-resolver](https://docs.rs/trust-dns-resolver)'s
asynchronous resolver compatible with [hyper](https://docs.rs/hyper) client,
to use instead of the default dns threadpool.

[Documentation](https://docs.rs/hyper-trust-dns-connector)

## Motivations

By default hyper HttpConnector uses the std provided resolver wich is blocking in a threadpool
with a configurable number of threads. This crate provides an alternative using trust_dns_resolver,
a dns resolver written in Rust, with async features.

## Usage

[trust-dns-resolver](https://docs.rs/trust-dns-resolver) creates a background that needs to
be spawned on top of an executor to run dns queries. It does not need to be spawned on the
same executor as the client, but will deadlock if spawned on top of another executor that
runs on the same thread.

## Example

```rust
extern crate hyper_trust_dns_connector;
extern crate hyper;
extern crate tokio;

use hyper_trust_dns_connector::new_async_http_connector;
use hyper::{Client, Body};
use tokio::runtime::Runtime;

fn main() {
    let mut rt = Runtime::new().expect("couldn't create runtime");
    let (http, background) = new_async_http_connector()
        .expect("couldn't create connector");
    let client = Client::builder()
        .executor(rt.executor())
        .build::<_, Body>(http);
    rt.spawn(background);
    let status_code = rt
        .block_on(client.get(hyper::Uri::from_static("http://httpbin.org/ip")))
        .map(|res| res.status())
        .expect("error during the request");
    println!("status is {:?}", status_code);
}
```

## Contributing

If you need a feature implemented, or want to help, don't hesitate to open an issue or a PR.

## License

Provided under the MIT license ([LICENSE](LICENSE) or <http://opensource.org/licenses/MIT>)