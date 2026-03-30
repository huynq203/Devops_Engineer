##### Dockerfile ##### 
## Builder Stage ##
FROM rust:1.68.2-slim as builder

WORKDIR /usr/src/app
RUN apt-get update && apt-get install musl-tools --no-install-recommends -y
RUN rustup target add x86_64-unknown-linux-musl
COPY Cargo.toml ./

RUN mkdir src/ && echo "fn main() {println!(\"if you see this, the build broke\")}" > src/main.rs && \
  cargo build --target x86_64-unknown-linux-musl --release && \
  rm -f target/x86_64-unknown-linux-musl/release/deps/eldevops*

COPY . ./

RUN cargo build --target x86_64-unknown-linux-musl --release

## Run stage ##
FROM alpine:3.17 as runtime
COPY --from=builder /usr/src/app/target/x86_64-unknown-linux-musl/release/eldevops/ /usr/local/bin/

ENTRYPOINT ["/usr/local/bin/eldevops"]