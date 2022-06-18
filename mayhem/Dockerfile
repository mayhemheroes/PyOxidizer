# Build Stage
FROM ghcr.io/evanrichter/cargo-fuzz:latest as builder

RUN git clone --depth=1 https://github.com/openssl/openssl.git
RUN cp -r openssl/fuzz/corpora/x509 /testsuite

## Add source code to the build stage.
ADD . /src
WORKDIR /src

RUN echo building instrumented harnesses && \
    bash -c "pushd x509-certificate/fuzz && cargo +nightly -Z sparse-registry fuzz build && popd" && \
    mv x509-certificate/fuzz/target/x86_64-unknown-linux-gnu/release/from_der /from_der && \
    echo done

RUN echo building non-instrumented harnesses && \
    export RUSTFLAGS="--cfg fuzzing -Clink-dead-code -Cdebug-assertions -C codegen-units=1" && \
    bash -c "pushd x509-certificate/fuzz && cargo +nightly -Z sparse-registry build --release && popd" && \
    mv x509-certificate/fuzz/target/release/from_der /from_der_no_inst && \
    echo done

# Package Stage
FROM rustlang/rust:nightly

COPY --from=builder /from_der /from_der_no_inst /
COPY --from=builder /testsuite/ /testsuite/
