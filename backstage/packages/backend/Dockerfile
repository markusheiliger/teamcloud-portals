# This dockerfile builds an image for the backend package.
# It should be executed with the root of the repo as docker context.
#
# Before building this image, be sure to have run the following commands in the repo root:
#
# yarn install
# yarn tsc
# yarn build
#
# Once the commands have been run, you can build the image using `yarn build-image`

FROM node:16-bullseye-slim

WORKDIR /app

# Copy repo skeleton first, to avoid unnecessary docker cache invalidation.
# The skeleton contains the package.json of each package in the monorepo,
# and along with yarn.lock and the root package.json, that's enough to run yarn install.
COPY yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

# Install packages:
# - sqlite3 dependencies
# - mkdocs for techdoc generation
RUN apt-get update && \
    apt-get install -y libsqlite3-dev python3 python3-pip cmake g++ && \
    pip3 install mkdocs-techdocs-core==0.2.2 && \
    rm -rf /var/lib/apt/lists/* && \
    yarn config set python /usr/bin/python3

RUN yarn install --frozen-lockfile --production --network-timeout 300000 && rm -rf "$(yarn cache dir)"

# Then copy the rest of the backend bundle, along with any other files we might want.
COPY packages/backend/dist/bundle.tar.gz app-config.yaml app-config.*.yaml ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

# If using PostgreSql instead of sqlite3 we require SSL 
# transport encryption by default
ENV PGSSLMODE=require

CMD ["node", "packages/backend", "--config", "app-config.yaml"]
