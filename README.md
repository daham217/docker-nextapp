# docker-nextapp

A production-ready Next.js application containerized using a multi-stage Docker build and pushed to Docker Hub. Built as a DevOps project covering Docker best practices — multi-stage builds, non-root user security, and image publishing.

## What This App Is

A default Next.js 16 application served in a lean, secure Docker container. The focus is not the app itself but how it is packaged and shipped using Docker.

## New Techniques Used

**Multi-stage Docker build** — three separate stages in one Dockerfile. Each stage starts fresh so the final image contains only what is needed to run the app, not the source code or dev dependencies. Reduces image size from ~600MB to ~80MB.

**Non-root user** — a system group called nodejs and a user called nextjs are created inside the container. The app runs as nextjs, never as root. Running as root is dangerous because if someone breaks out of the container they get root access on the host machine.

**npm ci instead of npm install** — installs exact versions from package-lock.json. Faster and deterministic — no version surprises between environments.

**NEXT_TELEMETRY_DISABLED** — disables Next.js anonymous usage tracking inside the container.

**standalone output** — next.config.ts is set to output: standalone which tells Next.js to bundle everything into a single server.js file. This is what makes the final image self-contained.

**.dockerignore** — excludes node_modules, .next, and .git from the build context. Without this Docker would copy hundreds of megabytes of files it doesn't need, slowing down every build.

## Dockerfile Stages

Stage 1 - deps
Installs libc6-compat (required by some npm packages on Alpine) then runs npm ci to install exact dependencies from the lockfile.

Stage 2 - builder
Copies node_modules from Stage 1 and the full source code. Runs npm run build to produce the optimized .next/standalone output.

Stage 3 - runner
Fresh Alpine image. Creates nodejs group and nextjs user. Copies only the three things needed at runtime: public/, .next/standalone/, .next/static/. Switches to non-root user and starts the app.

## How to Run Locally

Build the image:
docker build -t docker-nextapp:latest .

Run the container:
docker run -p 3000:3000 docker-nextapp:latest

Open http://localhost:3000 in your browser.

## Docker Hub

Image is publicly available on Docker Hub:
docker pull daham27/docker-nextapp:latest
docker run -p 3000:3000 daham27/docker-nextapp:latest

## Steps to Push to Docker Hub

1. Log in to Docker Hub
docker login -u yourusername

2. Build with your Docker Hub username as the tag
docker build -t yourusername/docker-nextapp:latest .

3. Push the image
docker push yourusername/docker-nextapp:latest

Image is now publicly available at https://hub.docker.com/r/yourusername/docker-nextapp
