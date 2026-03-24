FROM node:22.13.0-alpine

LABEL com.centurylinklabs.watchtower.lifecycle.pre-check="scripts/watchtower-hooks/pre-check.sh"
LABEL com.centurylinklabs.watchtower.lifecycle.pre-update="scripts/watchtower-hooks/pre-update.sh"
LABEL com.centurylinklabs.watchtower.lifecycle.post-update="scripts/watchtower-hooks/post-update.sh"
LABEL com.centurylinklabs.watchtower.lifecycle.post-check="scripts/watchtower-hooks/post-check.sh"

WORKDIR /app

ENV PORT=4001
ENV COUCH_DB_URL=https://couchdb.budi.live:5984
ENV BUDIBASE_ENVIRONMENT=PRODUCTION
ENV SERVICE=app-service
ENV POSTHOG_TOKEN=phc_bIjZL7oh2GEUd2vqvTBH8WvrX0fWTFQMs6H5KQxiUxU
ENV ACCOUNT_PORTAL_URL=https://account.budibase.app
ENV TOP_LEVEL_PATH=/

# handle node-gyp and install postgres client for pg_dump utils
RUN apk add --no-cache \
    g++ \
    make \
    python3 \
    jq \
    bash \
    postgresql-client \
    git \
    procps \
    iproute2 \
    curl \
    bind-tools \
    netcat-openbsd \
    openssl \
    lsof \
    strace \
    less \
    coreutils \
    tzdata

RUN yarn global add pm2

# Copy package files for dependency installation
COPY package.json ./
COPY yarn.lock ./

# Install yarn packages with caching
ARG TARGETPLATFORM
RUN --mount=type=cache,target=/root/.yarn/${TARGETPLATFORM} YARN_CACHE_FOLDER=/root/.yarn/${TARGETPLATFORM} yarn install --production --ignore-scripts \
    && yarn cache clean \
    && apk del g++ make python3 \
    && rm -rf /tmp/* /root/.node-gyp /usr/local/lib/node_modules/npm/node_modules/node-gyp

# Copy application files
COPY . .

ARG BUDIBASE_VERSION=0.0.0
ARG GIT_COMMIT_SHA=unknown
ENV BUDIBASE_VERSION=$BUDIBASE_VERSION
ENV DD_GIT_REPOSITORY_URL=https://github.com/budibase/budibase
ENV DD_GIT_COMMIT_SHA=$GIT_COMMIT_SHA
ENV DD_VERSION=$BUDIBASE_VERSION

EXPOSE 4001

# have to add node environment production after install
# due to this causing yarn to stop installing dev dependencies
# which are actually needed to get this environment up and running
ENV NODE_ENV=production
# This is required for isolated-vm to work on Node 20+
ENV NODE_OPTIONS="--no-node-snapshot"
ENV CLUSTER_MODE=""
ENV TOP_LEVEL_PATH=/app

# Create a default startup script if docker_run.sh doesn't exist
RUN if [ ! -f ./docker_run.sh ]; then \
        echo '#!/bin/bash' > ./docker_run.sh && \
        echo 'exec node dist/index.js' >> ./docker_run.sh && \
        chmod +x ./docker_run.sh; \
    else \
        chmod +x ./docker_run.sh; \
    fi

CMD ["./docker_run.sh"]
