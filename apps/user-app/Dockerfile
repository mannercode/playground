# syntax=docker/dockerfile:1

FROM node:26.13.1-bookworm-slim@sha256:5df79989ed685e09f0a98019db4cebfb22fb9df1a53c0161664a011f7ff6b0f1 AS build

RUN npm install -g pnpm@12.4.2

WORKDIR /workspace
COPY package.json pnpm-workspace.yaml pnpm-lock.yaml ./
COPY apps/user-app/package.json apps/user-app/
COPY tools/dev-tools/package.json tools/dev-tools/free-port.js tools/dev-tools/tunnel.sh tools/dev-tools/

RUN --mount=type=cache,id=nest-seed-pnpm,target=/pnpm/store,sharing=locked \
    HUSKY=0 pnpm install --frozen-lockfile --filter './apps/user-app...' --store-dir /pnpm/store

COPY apps/user-app/ apps/user-app/
RUN pnpm --filter './apps/user-app' --fail-if-no-match run build

FROM node:26.13.1-bookworm-slim@sha256:5df79989ed685e09f0a98019db4cebfb22fb9df1a53c0161664a011f7ff6b0f1 AS runtime

ENV HOSTNAME=0.0.0.0 \
    NODE_ENV=production \
    PORT=3200

WORKDIR /workspace
COPY --from=build --chown=node:node /workspace/apps/user-app/.next/standalone ./
COPY --from=build --chown=node:node /workspace/apps/user-app/.next/static ./apps/user-app/.next/static

USER node

EXPOSE 3200
CMD ["node", "apps/user-app/server.js"]
