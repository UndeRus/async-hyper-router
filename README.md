# async-hyper-router
## Usage

To use the library just add: 
 
```text
async_hyper_router = { git = "https://github.com/Caspinol/async-hyper-router.git" }
```
to your Cargo.toml

## Example
```
#[macro_use]
extern crate lazy_static;

extern crate tokio_core;
extern crate hyper;
extern crate futures;
extern crate hyper_router;

use futures::future::FutureResult;

use hyper::StatusCode;
use hyper::header::ContentLength;
use hyper::server::{Http, Request, Response, Service};
use hyper_router::{HyperRouter};

static HELLO_GET: &'static [u8] = b"Hello, Get!";
static HELLO_POST: &'static [u8] = b"Hello, Post!";

lazy_static! {
    static ref ROUTER: HyperRouter = HyperRouter::new()
        .get("/hello_get", get_handler)
        .post("/hello_post", post_handler);
}

fn get_handler(_: Request) -> Response {
    Response::new()
        .with_header(ContentLength(HELLO_GET.len() as u64))
        .with_body(HELLO_GET)
}

fn post_handler(_: Request) -> Response {
    Response::new()
        .with_header(ContentLength(HELLO_POST.len() as u64))
        .with_body(HELLO_POST)
}

struct MyServer(&'static HyperRouter);

impl Service for MyServer {
    type Request = Request;
    type Response = Response;
    type Error = hyper::Error;
    type Future = FutureResult<Response, hyper::Error>;

    fn call(&self, req: Request) -> Self::Future {
        
        futures::future::ok(match self.0.find_handler(&req) {
            Ok(handler) => handler(req),
            Err(_) => Response::new()
                .with_status(StatusCode::NotFound),
        })
    }
}

fn main() {
    let addr = "127.0.0.1:3000".parse().unwrap();
    
    let mut server = Http::new().bind(&addr, || Ok(MyServer(&ROUTER))).unwrap();

    server.no_proto();
    println!("Listening on http://{}.", server.local_addr().unwrap());
    server.run().unwrap();
}
 ```
