---
name: nodejs-docker-best-practices
description: Create, review, and improve Dockerfiles and container configurations for production Node.js applications. Use for Node.js image selection, dependency installation, multi-stage builds, build caching, non-root execution, signal handling, native modules, image size, runtime security, .dockerignore files, Compose services, or containerized test workflows.
---

# Node.js Docker Best Practices

Inspect the application before changing its container setup. Apply only the guidance that fits its runtime, package manager, framework, native dependencies, and deployment platform.

Read [references/best-practices.md](references/best-practices.md) before creating or substantially changing a production Dockerfile. Use its review checklist for smaller reviews.

## Workflow

1. Inspect the relevant files, including `Dockerfile*`, `.dockerignore`, Compose files, package manifests, lockfiles, build configuration, and deployment manifests.
2. Determine:
   - the supported Node.js version and image family;
   - the package manager and exact lockfile;
   - build, test, and runtime commands;
   - generated artifacts and production dependency needs;
   - native modules or operating-system packages;
   - runtime ports, writable paths, signals, and shutdown behavior.
3. Preserve the project's package manager, lockfile, runtime contract, and deployment assumptions unless the user asks to change them.
4. Design the stages and layer order before editing. Keep build tooling out of the runtime image when practical.
5. Implement the smallest coherent change. Update `.dockerignore`, Compose, CI, or deployment files only when the Dockerfile change requires it.
6. Verify the touched path with the strongest checks available: parse or lint the Dockerfile, build the relevant target, run its tests, start the container, and exercise shutdown or health behavior as appropriate.

## Decision Rules

- Prefer a supported, explicit Node.js image tag. Choose Debian slim, Alpine, distroless, or hardened images according to compatibility and operational needs rather than image size alone.
- Use the repository's lockfile and deterministic install command. Fail instead of silently selecting a different package manager or generating a new lockfile.
- Separate dependency installation from source copying so dependency layers remain cacheable. Use BuildKit cache mounts when the build environment supports them.
- Use multi-stage builds when compilation, testing, or native build tools are unnecessary at runtime. Keep build and runtime operating systems and libc compatible when copying native modules.
- Run as a non-root user. Ensure copied files and required writable directories have the correct ownership before switching users.
- Launch the application with exec-form `CMD`. Prefer invoking `node` directly in production unless a required launcher correctly forwards signals.
- Decide explicitly how PID 1 responsibilities are handled. Use the platform's init option or a small init process when child reaping or signal forwarding requires it, and make the application handle graceful shutdown.
- Keep credentials out of Dockerfile `ARG`, `ENV`, copied files, and image history. Use runtime secrets or BuildKit secret mounts.
- Keep the build context narrow with `.dockerignore`; never exclude files required by lockfile-based installation or the build.
- Treat `NODE_ENV=production`, `EXPOSE`, health checks, resource limits, and read-only filesystems as runtime-contract decisions, not boilerplate to add blindly.
- Avoid global package installation when a project-local dependency or package-manager runner is sufficient.

## Review Output

When reviewing an existing setup:

1. Report concrete findings in severity order with file and line references.
2. Explain the build, runtime, security, or operability consequence.
3. Recommend the smallest safe fix and note meaningful tradeoffs.
4. Distinguish verified defects from optional hardening or image-size improvements.
5. Do not rewrite a working Dockerfile solely to match a preferred template.

When creating or modifying files, summarize the resulting build/runtime shape and the verification performed.

## Freshness

The bundled reference is a synthesis, not a frozen substitute for upstream documentation. If the request asks for the latest image tags, Node.js support status, Dockerfile syntax, Docker features, or security recommendations, verify those details against the official sources linked in the reference before answering.
