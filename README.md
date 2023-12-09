demo4 - Running Boa JavaScript Interpreter in an Internet Computer Canister

This demo project showcases the functionality of a canister interpreting JavaScript using the [Boa](https://github.com/boa-dev/boa) engine.

## Prerequisites

Before running this demo, ensure you have the following installed on your system:


* [rust](https://doc.rust-lang.org/book/ch01-01-installation.html)
* [DFINITY SDK(dfx)](https://internetcomputer.org/docs/current/developer-docs/setup/install/)
* [wasi2ic](https://github.com/wasm-forge/wasi2ic)

You can proceed by either creating this project from scratch or by cloning this repository and jumping to the "Deployment and Testing" stage.

## Creating the Project from Scratch

* Create the project using `dfx new --type rust --no-frontend demo4`
* Navigate to the folder: `demo4/src/demo4_backend`
* Add the Boa engine dependency: `cargo add boa_engine`
* Add the ic_polyfill dependency: `cargo add ic-wasi-polyfill`

* Modify the `demo4/src/demo4_backend/src/lib.rs` file, create the `eval` method, so that it takes an arbitrary JavaScript and executes it.

```rust
#[ic_cdk::query]
fn eval(js_code: String) -> String {

    use boa_engine::{Context, Source};
    
    let mut context = Context::default();
    
    match context.eval(Source::from_bytes(&js_code)) {
        Ok(res) => {
            return res.to_string(&mut context).unwrap().to_std_string_escaped();
        }
        Err(e) => {
            eprintln!("Uncaught {e}");
        }
    };

    return String::from("");
}


#[ic_cdk::init]
fn init() {
    ic_wasi_polyfill::init(&[0u8;32], &[]);
}

```


## Deployment and testing

Start the `dfx` environment in a separate console:
```
  dfx start
```
This window will show you the communication with your canister.


Once you have the demo4 project in your folder, enter the folder and deploy the canister with:

```bash
  dfx canister create --all
```

You have to build the project inside the `demo4` folder for a `wasm32-wasi` target (you might need to install the target using rustup), do this with the command:
```bash
  cargo build --release --target wasm32-wasi
```

If everything works out, you can now enter the new folder `target/wasm32-wasi/release`, it should contain the file: 'demo4_backend.wasm'.

This file cannot be deployed directly because it has WASI dependencies in it. 
(You can check that by converting the .wasm file to its textual representation with the `wasm2wat` command).

Now, use the `wasi2ic` tool to re-route the dependencies:
```bash
  wasi2ic demo4_backend.wasm no_wasi.wasm
```
It creates a new file called `no_wasi.wasm`.
before deploying it, you need to compress it `gzip no_wasi.wasm`. Now you can deploy it manually:

```bash
  dfx canister install --mode reinstall --wasm no_wasi.gzip demo4_backend
```

Accept the canister overwrite with 'yes'. Now call the canister with some JavaScript code:

```bash
  dfx canister call demo4_backend eval '("a=5;b=7;a+b")'
```

You should now in the console window see the result of the code execution.

