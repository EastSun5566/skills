# Node.js Docker Reference

This reference synthesizes the official Node.js Docker image guidance and Docker's Node.js guide. It was checked on 2026-07-12 against:

- [Node.js Docker image best practices](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md)
- [Docker Node.js guide](https://docs.docker.com/guides/nodejs/)
- [Docker Node.js guide source](https://github.com/docker/docs/blob/main/content/guides/nodejs.md)

Use the upstream Node.js document for behavior specific to the official `node` image. Use Docker's guide for general build, caching, development, Compose, and test workflows. Treat the application repository and deployment platform as the source of truth for project-specific requirements.

## Contents

- [Base image](#base-image)
- [Build context and cache](#build-context-and-cache)
- [Dependency installation](#dependency-installation)
- [Multi-stage builds](#multi-stage-builds)
- [Users and permissions](#users-and-permissions)
- [Processes and signals](#processes-and-signals)
- [Configuration and secrets](#configuration-and-secrets)
- [Native modules and Alpine](#native-modules-and-alpine)
- [Runtime operations](#runtime-operations)
- [Development and tests](#development-and-tests)
- [Review checklist](#review-checklist)
- [Example production shape](#example-production-shape)

## Base image

- Select a supported Node.js release that matches the application's declared engine and production policy.
- Use an explicit tag that communicates the intended Node.js and operating-system family. Use a digest when the deployment requires immutable inputs, together with an intentional update process.
- Prefer the official `node` image as the general baseline. Use a hardened, distroless, or runtime-only image only when its package availability, debugging model, certificates, timezone data, user model, and operational constraints are understood.
- Do not select Alpine solely because it is smaller. Its musl libc and reduced package set can complicate native dependencies and debugging. Debian slim is often the simpler compatibility choice.
- Keep build and runtime stages ABI-compatible. Do not copy native `node_modules` between incompatible architectures, libc implementations, or operating-system releases.

## Build context and cache

- Start modern Dockerfiles with a compatible syntax directive when using BuildKit-only features:

  ```dockerfile
  # syntax=docker/dockerfile:1
  ```

- Copy or bind-mount package manifests and the selected lockfile before application source so dependency installation can be cached independently.
- Use cache mounts for package-manager download caches when BuildKit is available. A cache mount improves rebuild speed but must not be required for correctness.
- Copy only required source and configuration into each stage.
- Maintain a `.dockerignore` that commonly excludes `.git`, local `node_modules`, build output rebuilt in the image, coverage, logs, editor files, and local secrets such as `.env`.
- Check exceptions: do not exclude the lockfile, patches, workspace manifests, package-manager configuration, generated sources required by the build, or runtime assets that must be copied.

## Dependency installation

- Detect the package manager from repository evidence, not preference:
  - npm: `package-lock.json` and usually `npm ci`;
  - pnpm: `pnpm-lock.yaml` and usually `pnpm install --frozen-lockfile`;
  - Yarn: `yarn.lock` and the version-appropriate immutable or frozen-lockfile option.
- Respect the project's pinned package-manager version, commonly through the `packageManager` field and Corepack where supported by that Node.js image.
- Fail clearly when the expected lockfile is missing. Do not include a broad shell branch that silently chooses whichever lockfile happens to exist unless the repository intentionally supports multiple package managers.
- Install all dependencies in a build or test stage. Produce a separate production-only dependency set when the runtime needs `node_modules`.
- Do not assume setting `NODE_ENV=production` alone produces the intended dependency graph; use the package manager's explicit production-dependency option.
- Prefer local binaries through package scripts or the package manager over global npm packages. If a global package is unavoidable, install it in a non-root-owned prefix and pin it appropriately.
- Omit install-time audit or network extras only when consistent with the project's security and CI policy.

## Multi-stage builds

Use separate stages when they reduce runtime contents or clarify verification:

1. A dependency or build stage installs the locked dependency graph.
2. A build stage compiles TypeScript, bundles code, or produces framework artifacts.
3. An optional test stage runs checks using the build environment.
4. A runtime stage contains only the Node.js runtime, production dependencies, compiled output, and required runtime assets.

Additional rules:

- Copy the minimum artifacts between stages; avoid copying the entire builder filesystem.
- Keep native dependencies compatible with the final stage. Reinstall them in a compatible production-dependency stage when necessary.
- Do not remove npm, Yarn, Corepack, shells, or operating-system tools through fragile manual deletion merely to reduce size. Prefer a runtime image designed without them when that restriction is required.
- Keep development and production stages distinct when Compose uses bind mounts or watch mode.

## Users and permissions

- Run production workloads without root privileges unless the application has a documented requirement.
- The official Node.js image provides a `node` user, conventionally UID 1000. Do not assume the same user or UID exists in unrelated runtime images.
- Set ownership while copying when possible:

  ```dockerfile
  COPY --chown=node:node --from=build /app/dist ./dist
  ```

- Create and own writable directories before `USER node`. Identify caches, uploads, temporary files, SQLite databases, framework-generated files, and mounted volumes that require writes.
- Avoid recursively changing ownership over large trees in a separate layer when `COPY --chown` or targeted directory creation is sufficient.
- Consider arbitrary-UID platforms. Avoid depending on a writable home directory or a fixed numeric UID unless the deployment contract guarantees it.

## Processes and signals

- Use exec-form commands so the application receives container signals directly:

  ```dockerfile
  CMD ["node", "dist/server.js"]
  ```

- Prefer launching Node.js directly in production. Package-manager wrappers and shell-form commands add processes and may alter signal or exit-code behavior.
- Ensure the application handles `SIGTERM` and closes servers, workers, and external connections within the platform's termination grace period.
- PID 1 must also reap orphaned child processes. If the application or its dependencies spawn children, use the runtime's init option such as `docker run --init`, or include a small init such as Tini or `dumb-init`.
- Do not add two init layers. Understand whether the deployment platform already supplies one.
- Use `ENTRYPOINT` only for a stable wrapper or executable contract. If a wrapper is necessary, end it with `exec "$@"` so the target process replaces the shell.

## Configuration and secrets

- Set non-sensitive defaults with `ENV` only when they are valid across deployments. Runtime configuration should remain overridable.
- Set `NODE_ENV=production` for production runtime behavior when the application and dependencies rely on it. Do not use it as a secret transport or as a substitute for an explicit production dependency install.
- Never bake credentials into `ARG`, `ENV`, source files, package-manager configuration, or copied `.env` files. Build arguments and deleted files can remain visible in image history or layers.
- Use BuildKit secret or SSH mounts for credentials needed only during build. Use the orchestrator's runtime secret mechanism for application credentials.
- Avoid persisting registry tokens in the final filesystem. Scope private-registry access to the dependency-install step.

## Native modules and Alpine

- Identify native modules from install logs, package metadata, or tools such as `node-gyp`, not only from direct dependencies.
- Install compilers, Python, headers, and other build tools only in a build stage or a temporary virtual package group.
- Ensure the runtime contains the shared libraries required by compiled modules.
- Build native modules for the same CPU architecture, libc, Node.js ABI, and relevant operating-system family as the runtime stage.
- Do not blindly copy a host `node_modules` directory into an image. Install dependencies inside a matching container stage.
- When using Alpine, verify musl compatibility and required compatibility packages. Switch to a Debian-based image when that is the smaller operational risk.

## Runtime operations

- `EXPOSE` documents the intended port; it does not publish the port or configure the application to listen on all interfaces.
- Add a health check only when it represents meaningful application readiness or liveness and the runtime image contains a suitable probe. Prefer orchestrator-native probes when they provide clearer separation.
- Configure CPU and memory limits in Compose or the deployment platform, not in the image. Test Node.js behavior under the actual memory limit.
- Consider a read-only root filesystem and reduced Linux capabilities when the application and platform support them. Provide explicit writable mounts or temporary filesystems.
- Keep logs on stdout and stderr unless the deployment contract specifies another sink.
- Include only required certificates, timezone data, shared libraries, and runtime assets. Verify these explicitly when using minimal images.
- Define graceful shutdown, restart, rollout, and migration behavior outside the Dockerfile when they belong to the deployment platform.

## Development and tests

- Keep development conveniences such as watch mode, source bind mounts, debuggers, and full development dependencies out of the production target.
- Use a named development target when Compose needs a different command or filesystem layout.
- Add a test target when containerized tests materially verify the production build environment. Run it in CI or as part of an explicit build workflow rather than leaving an unused stage as implied assurance.
- Verify the final stage, not only the builder. At minimum, confirm the image starts, listens on the expected interface and port, runs as the intended user, and stops cleanly.
- When practical, inspect the final image for unexpected files, development dependencies, secrets, known vulnerabilities, and missing shared libraries.

## Review checklist

### Correctness

- Does the Node.js version satisfy the application and dependency requirements?
- Does the build use the repository's package manager, version, and lockfile?
- Are all build outputs and runtime assets copied to the final stage?
- Are native modules built for the final runtime environment?
- Does the command start the intended server or worker and return useful exit codes?

### Reproducibility and performance

- Is dependency installation deterministic?
- Can dependency layers be reused when only source files change?
- Are BuildKit caches optional optimizations rather than correctness dependencies?
- Is the build context narrow and free of local artifacts?

### Security

- Does the runtime use a non-root user with only necessary write access?
- Are secrets absent from build arguments, environment declarations, copied files, and image history?
- Are build tools and development dependencies absent from the final stage when practical?
- Is the base image supported and maintained through an update process?

### Operations

- Does PID 1 receive and forward signals correctly and reap children when necessary?
- Does the application shut down within the platform's grace period?
- Are ports, health probes, writable paths, logs, resource limits, and runtime configuration consistent with deployment manifests?
- Has the final target been built and exercised, not merely inspected?

## Example production shape

Adapt this example to the repository. Do not copy its Node.js version, paths, package manager, port, or base image without verification.

```dockerfile
# syntax=docker/dockerfile:1

ARG NODE_VERSION=24-bookworm-slim

FROM node:${NODE_VERSION} AS build
WORKDIR /app

COPY package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm npm ci

COPY . .
RUN npm test && npm run build

FROM node:${NODE_VERSION} AS production-deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm npm ci --omit=dev

FROM node:${NODE_VERSION} AS runtime
ENV NODE_ENV=production
WORKDIR /app

COPY --chown=node:node --from=production-deps /app/node_modules ./node_modules
COPY --chown=node:node --from=build /app/dist ./dist
COPY --chown=node:node package.json ./package.json

USER node
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

If the deployment must use an init process, add it deliberately through the platform or image and verify signal handling. If the application is bundled into a self-contained artifact, omit `node_modules`. If tests require external services or secrets, run them in a dedicated CI step instead of baking those dependencies into the production build.
