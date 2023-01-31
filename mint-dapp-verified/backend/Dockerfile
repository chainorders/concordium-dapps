ARG rust_base_image=rust:1.64
FROM ${rust_base_image} AS rust_build
WORKDIR /verifier
COPY ./deps/concordium-rust-sdk /deps/concordium-rust-sdk
COPY ./backend/verifier/src ./src
COPY ./backend/verifier/Cargo.lock ./Cargo.lock
COPY ./backend/verifier/Cargo.toml ./Cargo.toml

# Install protobuf
RUN wget https://github.com/protocolbuffers/protobuf/releases/download/v3.15.3/protoc-3.15.3-linux-x86_64.zip \
    && unzip protoc-3.15.3-linux-x86_64.zip \
    && mv ./bin/protoc /usr/bin/protoc \
    && chmod +x /usr/bin/protoc

RUN cargo build --release

FROM ubuntu:22.04
WORKDIR /build

ENV PORT=8100
ENV NODE=http://172.17.0.1:20000
ENV LOG_LEVEL=info
ENV STATEMENT='[{"type":"AttributeInRange","attributeTag":"dob","lower":"18000101","upper":"20091212"}]'
ENV VERIFY_KEY=""
ENV SIGN_KEY=""

COPY --from=rust_build ./verifier/target/release/verifier ./main
RUN chmod +x ./main

CMD ./main --node "${NODE}" --port "${PORT}" --log-level "${LOG_LEVEL}" --statement "${STATEMENT}" --verify-key "${VERIFY_KEY}" --sign-key "${SIGN_KEY}"
