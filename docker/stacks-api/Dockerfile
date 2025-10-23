FROM node:18-bullseye AS builder

RUN apt-get update && apt-get install -y \
    git \
    build-essential \
    python3 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

ARG STACKS_API_REPO=https://github.com/hirosystems/stacks-blockchain-api.git
ARG STACKS_API_BRANCH=master

RUN git clone $STACKS_API_REPO . && \
    git checkout $STACKS_API_BRANCH

RUN npm ci --audit=false

WORKDIR /app/client
RUN npm ci --audit=false

WORKDIR /app
RUN npm run build

FROM node:18-bullseye-slim

RUN apt-get update && apt-get install -y \
    curl \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

RUN useradd --create-home --shell /bin/bash stacks

WORKDIR /app

COPY --from=builder --chown=stacks:stacks /app .

USER stacks

EXPOSE 3700 3999

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3999/extended/v1/status || exit 1

CMD ["npm", "start"]