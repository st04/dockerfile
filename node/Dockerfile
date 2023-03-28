# Latest LTS version of docker.
FROM node:18.15.0-bullseye-slim as node

# Use official LTS Ubuntu base image for a secure base. 
FROM ubuntu:jammy-20230308 as base

# Replace npm in CMD with tini for better kernel signal handling
# You may also need development tools to build native npm addons:
# apt-get install gcc g++ make
RUN apt-get update \
    && apt-get -qq install -y --no-install-recommends \
    tini \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT ["/usr/bin/tini", "--"]

# Copy in the specific version of node that we require from a node docker image
# this avoids depdency package installs (python3) that the deb package requires.
COPY --from=node /usr/local/include/ /usr/local/include/
COPY --from=node /usr/local/lib/ /usr/local/lib/
COPY --from=node /usr/local/bin/ /usr/local/bin/
RUN corepack disable && corepack enable

# Create a non-priviledges user and group. 
RUN groupadd --gid 1000 node \
    && useradd --uid 1000 --gid node --shell /bin/bash --create-home node \
    && mkdir /app \
    && chown -R node:node /app

# Production stage
FROM base as prd
EXPOSE 3000
ENV NODE_ENV=production
WORKDIR /app
USER node
COPY --chown=node:node package*.json yarn*.lock ./
RUN npm ci --omit=dev && npm cache clean --force
COPY --chown=node:node . .
CMD ["npm", "run", "start"]
