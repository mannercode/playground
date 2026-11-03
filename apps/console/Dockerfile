# syntax=docker/dockerfile:1

FROM node:26.13.0-bookworm-slim@sha256:bc0a2afd3b74ce12222fc8bab99271e66257f021078f5f4a5d687a77d280c1d9 AS build

RUN npm install -g pnpm@12.4.2

WORKDIR /workspace
COPY package.json pnpm-workspace.yaml pnpm-lock.yaml ./
COPY apps/console/package.json apps/console/
COPY tools/dev-tools/package.json tools/dev-tools/free-port.js tools/dev-tools/tunnel.sh tools/dev-tools/

RUN --mount=type=cache,id=nest-seed-pnpm,target=/pnpm/store,sharing=locked \
    HUSKY=0 pnpm install --frozen-lockfile --filter './apps/console...' --store-dir /pnpm/store

COPY apps/console/ apps/console/
RUN pnpm --filter './apps/console' --fail-if-no-match run build

FROM node:26.13.0-bookworm-slim@sha256:bc0a2afd3b74ce12222fc8bab99271e66257f021078f5f4a5d687a77d280c1d9 AS runtime

ENV HOSTNAME=0.0.0.0 \
    NODE_ENV=production \
    PORT=3100

WORKDIR /workspace
COPY --from=build --chown=node:node /workspace/apps/console/.next/standalone ./
COPY --from=build --chown=node:node /workspace/apps/console/.next/static ./apps/console/.next/static

USER node

EXPOSE 3100
CMD ["node", "apps/console/server.js"]
