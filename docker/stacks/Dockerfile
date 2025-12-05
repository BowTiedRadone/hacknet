FROM rust:bookworm AS builder

# These will be overridden by the value from docker-compose
ARG STACKS_CORE_REPO=https://github.com/stacks-network/stacks-core.git
ARG STACKS_CORE_BASE_BRANCH=master
ARG USE_LOCAL_REPO=false

# RUN echo "Building Stacks Core from branch: $STACKS_CORE_BASE_BRANCH"

RUN apt-get update && apt-get install -y git libclang-dev llvm

# Clone from GitHub if USE_LOCAL_REPO is false, otherwise expect local source
RUN if [ "$USE_LOCAL_REPO" = "false" ]; then \
      echo "Building Stacks Core from branch: $STACKS_CORE_BASE_BRANCH"; \
      git clone --branch $STACKS_CORE_BASE_BRANCH --single-branch --depth=1 $STACKS_CORE_REPO /code/stacks-core; \
    else \
      echo "Building Stacks Core from local repository"; \
      mkdir -p /code/stacks-core; \
    fi

WORKDIR /code/stacks-core

# Copy local source if USE_LOCAL_REPO is true
COPY . /code/stacks-core/
# Run an build that we'll cache the result of and then build the code
RUN cargo build --features monitoring_prom,slog_json --bin stacks-node --bin stacks-signer

FROM debian:bookworm-slim AS stacks-node
RUN apt-get update \
    && apt-get install -y ca-certificates --no-install-recommends \
    && apt-get clean && rm -rf /var/lib/apt/lists/*
RUN mkdir /data
COPY --from=builder /code/stacks-core/target/debug/stacks-node /usr/local/bin/stacks-node
STOPSIGNAL SIGTERM

FROM debian:bookworm-slim AS stacks-signer
RUN apt-get update \
    && apt-get install -y ca-certificates --no-install-recommends \
    && apt-get clean && rm -rf /var/lib/apt/lists/*
RUN mkdir /data
COPY --from=builder /code/stacks-core/target/debug/stacks-signer /usr/local/bin/stacks-signer
STOPSIGNAL SIGTERM
