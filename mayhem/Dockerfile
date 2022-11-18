# Build Stage
FROM ghcr.io/evanrichter/cargo-fuzz:latest as builder

## Dependencies
RUN apt update && apt install -y libudev-dev

## Add source code to the build stage.
ADD . /src
WORKDIR /src

## Currently nightly is broken
RUN rustup toolchain install nightly-2022-06-30

RUN echo building instrumented harnesses && \
    bash -c "pushd discovery-handlers/udev/fuzz && cargo +nightly-2022-06-30 -Z sparse-registry fuzz build && popd" && \
    mv discovery-handlers/udev/fuzz/target/x86_64-unknown-linux-gnu/release/parse_udev_rule /parse_udev_rule && \
    echo done

RUN echo building non-instrumented harnesses && \
    export RUSTFLAGS="--cfg fuzzing -Clink-dead-code -Cdebug-assertions -C codegen-units=1" && \
    bash -c "pushd discovery-handlers/udev/fuzz && cargo +nightly-2022-06-30 -Z sparse-registry build --release && popd" && \
    mv discovery-handlers/udev/fuzz/target/release/parse_udev_rule /parse_udev_rule_no_inst && \
    echo done

# Package Stage
FROM rustlang/rust:nightly

COPY --from=builder /parse_udev_rule /parse_udev_rule_no_inst /
COPY --from=builder /lib/x86_64-linux-gnu/libudev.so.1 /lib/x86_64-linux-gnu/
