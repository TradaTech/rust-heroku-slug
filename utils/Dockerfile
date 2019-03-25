FROM ubuntu:18.04 as curl
RUN apt-get update && apt-get install -y curl libssl-dev pkg-config

FROM curl as node
WORKDIR /node
RUN curl https://nodejs.org/dist/v11.12.0/node-v11.12.0-linux-x64.tar.gz | \
  tar xzf - --strip-components=1
RUN rm -rf node/share
RUN rm -rf node/include

FROM curl as rust
WORKDIR /rust
ENV HOME=/rust
RUN curl https://sh.rustup.rs | sh -s -- -y
ENV PATH=$PATH:$HOME/.cargo/bin
RUN rm -rf $HOME/.rustup/toolchains/nightly-x86_64-unknown-linux-gnu/share
RUN rustup target add wasm32-unknown-unknown
RUN rustup component add rustfmt-preview
RUN apt-get install -y gcc file make
RUN CARGO_TARGET_DIR=/tmp/cargo cargo install wasm-bindgen-cli --vers 0.2.37
RUN CARGO_TARGET_DIR=/tmp/cargo cargo install wasm-gc
WORKDIR /tmp/wbg
COPY Cargo.toml .
COPY utils/blank.rs utils/blank.rs
RUN CARGO_TARGET_DIR=/rust/wasm-bindgen cargo build --release --target wasm32-unknown-unknown

# Minimize the size of this rust installation
RUN rm -rf $HOME/.cargo/registry
RUN strip -g $HOME/.cargo/bin/*
RUN rm -rf $HOME/.rustup/toolchains/nightly-x86_64-unknown-linux-gnu/bin/rust-gdb
RUN rm -rf $HOME/.rustup/toolchains/nightly-x86_64-unknown-linux-gnu/bin/rust-lldb
RUN rm -rf $HOME/.rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/codegen-backends/*emscripten*

FROM ubuntu:18.04
COPY --from=rust /rust /app
COPY --from=node /node /app/node
COPY ./app /app
RUN tar czvf slug.tgz ./app
