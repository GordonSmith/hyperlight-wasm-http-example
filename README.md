# `hyperlight-wasm` http example

This is a minimal example of a
[hyperlight-wasm](https://github.com/hyperlight-dev/hyperlight-wasm)
host application. It implements just enough of the `wasi:http` api
to run the [sample_wasi_http_rust
server](https://github.com/bytecodealliance/sample-wasi-http-rust).

## Prerequisites

1. [Rust](https://www.rust-lang.org/tools/install), including the `x86_64-unknown-none` target (which may be installed via e.g. `rustup target add x86_64-unknown-none`)
2. `clang`
3. [`just`](https://github.com/casey/just) (optional, but recommended)

For building the C++ guest you will also need:

4. `cmake` (>= 3.25)
5. `ninja-build`

If you want to follow the manual build instructions, you will also need:

6. [`wasm-tools`](https://github.com/bytecodealliance/wasm-tools)
7. [`cargo-component`](https://github.com/bytecodealliance/cargo-component)
8. [`hyperlight-wasm-aot`](https://github.com/hyperlight-dev/hyperlight-wasm)

## Simple setup

### Building

```sh
# Install JS dependencies
npm install
```

```sh
just build
```

### Running

```sh
# Run Rust
just run-rust

# Run JS
just run-js

# Run C++
just run-cpp
```

From another terminal, you can then test the server:

```sh
curl http://localhost:3000/
curl -w'\n' -d "hola mundo" http://127.0.0.1:3000/echo
curl -I -H "x-language: spanish" http://127.0.0.1:3000/echo-headers
# get the content of .gitignore from github.com/jprendes/hyperlight-wasm-http-example
curl -w'\n' http://127.0.0.1:3000/proxy
```

## Manual setup

### Building

Compile the WIT and set the environment variables used when building
(both the host and the guest):

```sh
wasm-tools component wit hyperlight.wit -w -o hyperlight-world.wasm
```

Build Rust:
```
cargo build
```

Build JS:
```
npm run build
```

### Running

Build the guest component:
```sh
cargo component build --release \
    --manifest-path guest_rust/Cargo.toml \
    --target-dir target
```

AOT compile it:

```sh
cargo install hyperlight-wasm-aot
hyperlight-wasm-aot compile --component \
    target/wasm32-wasip1/release/sample_wasi_http_rust.wasm \
    out/sample_wasi_http_rust.aot
```

You can then run the server:

Rust:
```sh
cargo run -- out/sample_wasi_http_rust.aot
```

JS:
```sh
cargo run -- out/sample_wasi_http_js.aot
```

### C++ guest

The C++ guest uses [vcpkg](https://vcpkg.io) to install [wasi-sdk](https://github.com/WebAssembly/wasi-sdk) automatically. The vcpkg submodule and overlay port handle the download and setup.

#### Simple setup

```sh
just build-cpp-component
just run-cpp
```

#### Manual setup

Bootstrap vcpkg (first time only):
```sh
./vcpkg/bootstrap-vcpkg.sh
```

Configure and build the C++ guest using CMake presets (this installs wasi-sdk via vcpkg automatically):
```sh
cmake --preset linux-release
cmake --build build
```

Convert the core wasm module into a wasi component and AOT compile it:
```sh
wasm-tools component new \
    build/guest_cpp_wasm/src/guest_cpp_wasm-build/guest_cpp.wasm \
    -o out/sample-wasi-http-cpp.wasm
hyperlight-wasm-aot compile --component \
    out/sample-wasi-http-cpp.wasm
```

Run the server:
```sh
cargo run -- out/sample-wasi-http-cpp.aot
```

## Try it yourself!

[GitHub codespaces](https://codespaces.new/hyperlight-dev/hyperlight-wasm-http-example)
