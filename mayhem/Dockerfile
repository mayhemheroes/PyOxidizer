# Build Stage
FROM --platform=linux/amd64 ubuntu:22.04 AS builder

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y curl build-essential
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --default-toolchain nightly
ENV PATH="/root/.cargo/bin:${PATH}"
RUN cargo install -f cargo-fuzz

ADD . /src

# Build in an isolated workspace so cargo fuzz resolves workspace root
# to here, not /src (the PyOxidizer workspace)
WORKDIR /tmp/x509build
RUN cargo init --lib --name x509-certificate .
RUN cp -r /src/x509-certificate/fuzz .
# Switch dep from path=".." to crates.io; drop stale lockfile
RUN sed -i 's|path = "\.\."|version = "0.25"|' fuzz/Cargo.toml && \
    rm -f fuzz/Cargo.lock

RUN echo building instrumented harnesses && \
    cargo fuzz build from_der && \
    mv fuzz/target/x86_64-unknown-linux-gnu/release/from_der /from_der && \
    echo done

RUN echo building non-instrumented harnesses && \
    export RUSTFLAGS="--cfg fuzzing -Clink-dead-code -Cdebug-assertions -C codegen-units=1" && \
    cd fuzz && cargo build --release && cd .. && \
    mv fuzz/target/release/from_der /from_der_no_inst && \
    echo done

# Package Stage
FROM ubuntu:22.04
COPY --from=builder /from_der /from_der_no_inst /
