# Part 8: DevOps

---

## Chapter 65: Linux Administration

### Introduction

Linux powers the vast majority of cloud infrastructure. Proficiency with the command line is essential for deploying, troubleshooting, and managing applications.

### Essential Commands

| Category | Commands |
|----------|----------|
| File operations | `ls`, `cp`, `mv`, `rm`, `find`, `locate`, `tree` |
| File content | `cat`, `less`, `head`, `tail`, `grep`, `awk`, `sed` |
| Permissions | `chmod`, `chown`, `umask`, `ls -l` |
| Processes | `ps`, `top`, `htop`, `kill`, `systemctl` |
| Networking | `curl`, `wget`, `ss`, `netstat`, `ping`, `traceroute`, `nslookup` |
| Disk | `df`, `du`, `mount`, `fdisk`, `lsblk` |
| System | `uname`, `lscpu`, `free`, `uptime`, `dmesg` |

### User & Permission Management

```bash
useradd -m -s /bin/bash john
passwd john
usermod -aG docker john
userdel -r john

groupadd developers
gpasswd -a alice developers

chmod 755 script.sh
chmod 600 /etc/ssl/private/key.pem
chown -R appuser:appgroup /var/www/app
```

### Process Management

```bash
ps aux
ps aux --sort=-%mem | head -10
top -o %MEM

kill -15 PID     # SIGTERM (graceful)
kill -9 PID      # SIGKILL (force)

systemctl start|stop|restart|status myapp
journalctl -u myapp -f
```

### Networking

```bash
ss -tlnp
netstat -tlnp
nslookup example.com
curl -I https://api.example.com
curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' https://api.example.com/endpoint
traceroute google.com
```

### Disk Management

```bash
df -h
du -sh /var/log/
du --max-depth=2 -h / | sort -rh | head -10
mount /dev/sdb1 /mnt/data
```

### Common Mistakes

- Running everything as root
- Using 777 permissions on files
- Not configuring log rotation (disk fills up)
- Forgetting to set umask for security
- Hardcoding IP addresses instead of hostnames
- Not using SSH key-based authentication

> ⚠️ **Warning:** Running `chmod 777` is a security hazard — it grants read, write, and execute permissions to everyone. Always use the most restrictive permissions possible.

> ⚠️ **Warning:** Never disable SELinux or AppArmor in production. These mandatory access control systems provide an essential defense layer even if a process is compromised.

> ✅ **Best Practice:** Always use SSH key-based authentication and disable password-based SSH login in production environments.

> ✅ **Best Practice:** Set up log rotation with `logrotate` to prevent disk-full incidents. Configure retention policies to rotate logs daily or based on size.

> ❓ **Interview Question:** You run `chmod 644 file.txt` — what permissions does each entity get?
> **Answer:** Owner gets read/write (6), group gets read (4), others get read (4).

> 💡 **Pro Tip:** Use `watch -n 1 'command'` to repeatedly run a command every second — invaluable for monitoring real-time changes like disk usage or process listings.

### Real World Analogy

Think of Linux permissions like a hotel key card system. The **owner** is the hotel manager with full access. The **group** is the housekeeping staff (read/write to certain floors). **Others** are guests with limited access. The **root user** is the master key — powerful but dangerous if misused.

### Practical Exercises

1. Create a new user, add them to the `sudo` group, and verify they can run privileged commands
2. Use `find` to locate all files with world-writable permissions (`-perm -o+w`) in `/etc`
3. Write a bash script that monitors disk usage and sends an alert email if usage exceeds 80%
4. Configure log rotation for a custom application log using `logrotate`
5. Use `ss -tlnp` to identify all listening ports and their associated processes

### Revision Notes

- **`chmod`**: Change file permissions — `u` (owner), `g` (group), `o` (others), `a` (all)
- **`chown`**: Change file owner/group
- **`ps`/`top`**: Process monitoring; use `top -o %MEM` to sort by memory
- **`systemctl`**: Manage systemd services — `start`, `stop`, `restart`, `status`, `enable`
- **`journalctl`**: View systemd logs; `-u` for specific service, `-f` for follow mode
- **Networking**: `ss` is the modern replacement for `netstat`; `curl -I` shows response headers
- **Disk**: `df -h` for human-readable disk usage; `du -sh *` for per-directory sizes

---

## Chapter 66: Git Branching Strategies

### Introduction

A branching strategy defines how teams use branches in Git to manage code changes, releases, and collaboration.

### GitFlow

GitFlow uses two main branches (`main` and `develop`) plus supporting branches.

| Branch | Purpose | Base | Merges Into |
|--------|---------|------|-------------|
| `main` | Production-ready code | - | - |
| `develop` | Integration branch | `main` | `main` (via release) |
| `feature/*` | New features | `develop` | `develop` |
| `release/*` | Release preparation | `develop` | `main` + `develop` |
| `hotfix/*` | Critical production fixes | `main` | `main` + `develop` |

**Pros**: Clear structure, supports hotfixes, works for scheduled releases.
**Cons**: Complex, overhead for small teams, not ideal for CI/CD with frequent deployments.

### Trunk-Based Development

All developers commit to a single branch (`main`) with short-lived feature branches.

| Practice | Description |
|----------|-------------|
| Short-lived branches | Feature branches live < 1 day |
| Frequent commits | Commit multiple times daily |
| Feature flags | Hide incomplete features behind flags |
| CI required | Automated tests on every commit |

**Pros**: Simplest model, supports CI/CD, less merge overhead.
**Cons**: Requires feature flags, discipline, and strong CI.

### GitHub Flow

- Create branch from `main`
- Make changes, commit, push
- Open a Pull Request
- Review, discuss, deploy from branch
- Merge to `main` and deploy

### Comparison

| Feature | GitFlow | Trunk-Based | GitHub Flow |
|---------|---------|-------------|-------------|
| Complexity | High | Low | Low |
| Release cycles | Scheduled | Continuous | Continuous |
| Hotfix support | First-class | Feature flags | Branch from main |
| PR workflow | Yes | Yes | Yes |
| Best for | Large teams, versioned releases | Startups, SaaS, CI/CD | Open source, small teams |

### Common Mistakes

- Long-lived feature branches (merge conflicts)
- Merging `develop` to `main` without testing
- Cherry-picking commits between branches
- Not deleting branches after merging
- Forgetting to rebase feature branches

> ⚠️ **Warning:** Long-lived feature branches (weeks instead of days) inevitably lead to painful merge conflicts. Merge or rebase frequently to keep your branch up to date.

> ⚠️ **Warning:** Cherry-picking commits between branches can lead to duplicate commits and confusing history. Prefer rebasing or merging instead.

> ✅ **Best Practice:** Always delete branches after merging. Most Git hosting platforms (GitHub, GitLab, Bitbucket) offer an auto-delete option on merged PRs.

> ✅ **Best Practice:** Use a pull request template to ensure every PR includes a description, testing notes, and a checklist. This standardizes reviews across the team.

> ❓ **Interview Question:** What is the difference between `git merge` and `git rebase`?
> **Answer:** `merge` creates a new commit that joins two histories, preserving the full branch timeline. `rebase` rewrites commit history by applying commits from one branch onto another, creating a linear history.

> 💡 **Pro Tip:** Use `git rebase -i HEAD~n` to interactively squash, reword, or reorder the last `n` commits before pushing. This keeps the commit history clean and meaningful.

### Real World Analogy

Branching strategies are like lanes on a highway. **GitFlow** is a multi-lane highway with service roads and dedicated exit ramps (release branches). **Trunk-Based Development** is a single-lane expressway where everyone merges quickly. **GitHub Flow** is a two-lane road with frequent merge points. Choose the one that matches your traffic pattern (team size, release frequency).

### Practical Exercises

1. Create a GitFlow-style repository: initialize `main` and `develop`, create a feature branch, merge it
2. Practice interactive rebase: create 5 small commits on a branch, then squash them into 2 using `git rebase -i`
3. Simulate a merge conflict: have two team members modify the same file, then resolve the conflict manually
4. Set up branch protection rules on GitHub requiring PR reviews and passing status checks
5. Practice `git bisect` to find the commit that introduced a bug

### Revision Notes

- **GitFlow**: `main` (production), `develop` (integration), `feature/*`, `release/*`, `hotfix/*`
- **Trunk-Based**: Short-lived branches (< 1 day), feature flags, CI required
- **GitHub Flow**: Branch from `main` → PR → review → merge → deploy
- **Key commands**: `git merge`, `git rebase`, `git cherry-pick`, `git log --oneline --graph`
- **PR best practices**: Small changes, descriptive titles, linked issues, squash commits
- **Protection rules**: Require PR reviews, status checks, up-to-date branches

---

## Chapter 67: GitHub Actions

### Introduction

GitHub Actions is a CI/CD platform that automates build, test, and deployment pipelines directly from GitHub repositories.

### Workflow Structure

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: "22"

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: "npm"
      - run: npm ci
      - run: npm test
      - run: npm run build

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      - run: aws s3 sync ./dist s3://my-app-bucket --delete
```

### Workflow Components

| Component | Description | Example |
|-----------|-------------|---------|
| `on` | Trigger events | `push`, `pull_request`, `schedule` |
| `jobs` | Groups of steps | `test`, `lint`, `deploy` |
| `runs-on` | Runner environment | `ubuntu-latest`, `self-hosted` |
| `strategy.matrix` | Multi-version testing | `node-version: [18, 20, 22]` |
| `steps` | Sequential actions | `actions/checkout` |
| `uses` | Reusable action | `actions/setup-node@v4` |
| `secrets` | Encrypted variables | `${{ secrets.DOCKER_PASSWORD }}` |

### Matrix Builds

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node: [18, 20, 22]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci
      - run: npm test
```

### Common Mistakes

- Hardcoding secrets instead of using `${{ secrets }}`
- Running long workflows without caching dependencies
- Not using matrix strategy for multi-version testing
- Building artifacts without caching (slow pipelines)

> ⚠️ **Warning:** Never hardcode secrets directly in workflow files or repository files. Always use GitHub Secrets (`${{ secrets.MY_SECRET }}`) and consider using OpenID Connect (OIDC) for cloud authentication instead of long-lived access keys.

> ⚠️ **Warning:** Self-hosted runners without proper isolation can expose your CI/CD pipeline to security risks. Always run self-hosted runners in ephemeral, isolated environments.

> ✅ **Best Practice:** Pin action versions by commit SHA (e.g., `actions/checkout@a81bbbf`) instead of tags to prevent supply chain attacks from compromised action releases.

> ✅ **Best Practice:** Use `actions/cache` to cache `node_modules`, `~/.npm`, pip, Maven, and other dependency directories. This can cut workflow run times by 50-80%.

> ❓ **Interview Question:** How do GitHub Actions differ from a traditional CI server like Jenkins?
> **Answer:** GitHub Actions is a managed, event-driven CI/CD platform integrated with GitHub. Jenkins is self-hosted, more customizable, but requires infrastructure management. Actions uses YAML workflows with community-created actions; Jenkins uses Groovy-based pipelines.

> 💡 **Pro Tip:** Use reusable workflows (called with `workflow_call`) to share common pipelines across repositories. Define them in a central `.github` repo and reference them by `{owner}/{repo}/.github/workflows/{name}@ref`.

### Real World Analogy

GitHub Actions is like an automated factory assembly line. Each **workflow** is a production line triggered by an event (push, PR). **Jobs** are stations on the line, and **steps** are individual tasks at each station. The **matrix strategy** is like having parallel assembly lines for different product variants. **Caching** is the parts inventory that keeps production fast.

### Practical Exercises

1. Create a CI workflow that runs on push to `main`: install dependencies, run lint, run tests, build
2. Add a matrix strategy to test across Node.js 18, 20, and 22 on both ubuntu-latest and windows-latest
3. Configure a deployment job that only runs after tests pass and only on the `main` branch
4. Add OIDC-based authentication to AWS using `aws-actions/configure-aws-credentials`
5. Create a reusable workflow for deployment and call it from two different application repos

### Revision Notes

- **Triggers**: `push`, `pull_request`, `schedule` (cron), `workflow_dispatch` (manual)
- **Matrix strategy**: Multi-version/multi-OS testing with `strategy.matrix`
- **Caching**: `actions/cache` with a `key` based on lockfile hash
- **Secrets**: `${{ secrets.NAME }}` — use for tokens, keys, passwords
- **Environment protection**: `environment:` with required reviewers
- **Reusable workflows**: `workflow_call` and `workflow_dispatch` for composition

---

## Chapter 68: GitLab CI

### Introduction

GitLab CI is a built-in CI/CD system that uses `.gitlab-ci.yml` to define pipelines.

### Basic Configuration

```yaml
image: node:22

stages:
  - test
  - build
  - deploy

cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/

before_script:
  - npm ci

test:
  stage: test
  script:
    - npm test

build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/

deploy_staging:
  stage: deploy
  script:
    - aws s3 sync dist/ s3://staging-bucket
  environment:
    name: staging
  only:
    - develop

deploy_production:
  stage: deploy
  script:
    - aws s3 sync dist/ s3://production-bucket
  environment:
    name: production
  only:
    - main
  when: manual
```

### Common Mistakes

- Missing `cache` configuration (slow pipelines)
- Using `npm install` instead of `npm ci`
- Forgetting `artifacts` for build outputs
- Running all jobs on every branch without `only`/`except`

> ⚠️ **Warning:** Forgetting `artifacts` means build outputs from one stage are unavailable to subsequent stages. Each job runs in a fresh environment — artifacts are the only way to pass built files between stages.

> ⚠️ **Warning:** Be careful with `rules:` — overlapping or incorrectly ordered rules can cause unexpected job behavior. Always test complex rule logic in a feature branch first.

> ✅ **Best Practice:** Use the `include` keyword with `project` and `file` to share pipeline templates across multiple repos. This is GitLab's equivalent of GitHub's reusable workflows.

> ✅ **Best Practice:** Always use masked and protected CI/CD Variables for sensitive data. Masked variables are hidden in job logs; protected variables are only available to pipelines on protected branches.

> ❓ **Interview Question:** What is the difference between `only`/`except` and `rules` in GitLab CI?
> **Answer:** `only`/`except` are simpler, legacy syntax for branch-based job control. `rules` is more powerful, supporting conditional logic with `if`, `changes`, `exists`, and `when` clauses. `rules` is the recommended approach for newer projects.

> 💡 **Pro Tip:** Use GitLab CI's `parallel:matrix` to run the same job with different parameters (e.g., test against multiple DB versions). This is more efficient than creating separate jobs for each combination.

### Real World Analogy

GitLab CI is like a restaurant kitchen with a well-organized workflow. The **stages** are the kitchen stations (prep, cook, plate, serve). The **cache** is the pantry with frequently used ingredients. **Artifacts** are the partially prepared dishes passed between stations. The `include` keyword lets you bring in standardized recipes from your recipe book.

### Practical Exercises

1. Create a `.gitlab-ci.yml` with three stages (test, build, deploy) and cache for `node_modules`
2. Use `parallel:matrix` to run tests against Node.js 18 and 20 with PostgreSQL 15 and 16
3. Configure a manual deploy job for production that requires approval
4. Extract a common CI template into a separate project and reference it with `include`
5. Set up environment-specific variables in GitLab CI/CD Settings with protection

### Revision Notes

- **Stages**: Define execution order — `test` → `build` → `deploy`
- **Cache**: Share dependency directories across jobs and pipelines
- **Artifacts**: Pass built files between stages (e.g., `dist/` from build → deploy)
- **`rules`**: Modern conditional job execution (replaces `only`/`except`)
- **`include`**: Reuse pipeline configs from other files or repos
- **`parallel`**: Run multiple instances of the same job for test splitting

---

## Chapter 69: Jenkins

### Introduction

Jenkins is a self-hosted automation server for CI/CD. It supports both declarative and scripted pipelines.

### Declarative Pipeline

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Install') {
            steps { sh 'npm ci' }
        }
        stage('Test') {
            steps { sh 'npm test' }
            post {
                always { junit 'reports/**/*.xml' }
            }
        }
        stage('Build') {
            steps { sh 'npm run build' }
        }
        stage('Deploy') {
            when { branch 'main' }
            steps {
                sh 'aws s3 sync dist/ s3://production-bucket'
            }
        }
    }

    post {
        always { cleanWs() }
        failure {
            slackSend(color: '#ff0000', message: "Build failed: ${env.JOB_NAME}")
        }
    }
}
```

### Scripted Pipeline

```groovy
node {
    stage('Checkout') { checkout scm }
    stage('Install & Build') {
        sh 'npm ci && npm run build'
    }
    stage('Parallel Tests') {
        parallel(
            unit: { sh 'npm run test:unit' },
            integration: { sh 'npm run test:integration' }
        )
    }
    stage('Deploy') {
        if (env.BRANCH_NAME == 'main') {
            sh 'aws s3 sync dist/ s3://production-bucket'
        }
    }
}
```

### Declarative vs Scripted

| Aspect | Declarative | Scripted |
|--------|------------|----------|
| Syntax | Structured, simpler | Groovy-based, flexible |
| Learning curve | Easier | Steeper |
| Pipeline model | `pipeline { }` | `node { }` |
| When to use | Most projects | Complex pipelines, dynamic logic |

### Common Mistakes

- Running everything on a single node (no parallelism)
- Hardcoding credentials instead of using Jenkins Credentials
- Not cleaning workspace (`cleanWs()`)
- Long pipeline stages without timeouts

> ⚠️ **Warning:** Jenkins without proper backup is a single point of failure. Always back up `$JENKINS_HOME` (which contains all job configs, credentials, and plugin data) regularly.

> ⚠️ **Warning:** Running Jenkins with default security settings is extremely dangerous. Always enable authentication, configure authorization (Matrix-based security), and use HTTPS in production.

> ✅ **Best Practice:** Use Declarative Pipelines (not Scripted) for most projects — they offer better structure, visualization, and error handling out of the box.

> ✅ **Best Practice:** Implement the Shared Library pattern for reusable pipeline code. Store common functions, steps, and deployment logic in a separate Git repo that all pipelines can reference.

> ❓ **Interview Question:** How does a Jenkins Declarative Pipeline handle error recovery compared to a Scripted Pipeline?
> **Answer:** Declarative Pipelines have built-in `post` sections (`always`, `success`, `failure`, `unstable`) for cleanup and notifications. Scripted Pipelines require explicit `try/catch/finally` blocks for error handling.

> 💡 **Pro Tip:** Use Jenkins Pipeline's `parallel` directive inside `stage` — not just at the top level — to run integration tests, linting, and security scans concurrently while still keeping the pipeline organized.

### Real World Analogy

Jenkins is like a factory foreman who oversees the entire production process. The **pipeline** is the assembly line blueprint. **Stages** are the workstations. The **shared library** is the standard operating procedures manual. **Credentials** are the key cabinet. Unlike cloud CI services (GitHub Actions, GitLab CI), Jenkins requires you to build and maintain the entire factory yourself.

### Practical Exercises

1. Create a Declarative Pipeline with stages: Checkout, Build, Test, Archive, and Deploy
2. Add a `parallel` block to run unit tests and integration tests concurrently
3. Store a cloud provider API key in Jenkins Credentials and reference it in a pipeline
4. Implement a post-build action that sends a Slack notification on failure
5. Create a Jenkins Shared Library with a common "deploy to Kubernetes" function

### Revision Notes

- **Declarative Pipeline**: Structured `pipeline { agent; stages; post }` — preferred for most cases
- **Scripted Pipeline**: Groovy-based `node { stage { } }` — for complex dynamic logic
- **Credentials**: Use `withCredentials([[$class: 'UsernamePasswordMultiBinding']])` — never hardcode
- **Cleanup**: Always include `cleanWs()` in `post { always }`
- **Timeouts**: Add `timeout(time: 10, unit: 'MINUTES')` to every stage
- **Shared Library**: Store in `vars/` directory, reference with `@Library('my-lib')`

---

## Chapter 70: Docker

### Introduction

Docker is a containerization platform that packages applications and their dependencies into lightweight, portable containers.

### Why It Matters

Docker eliminates "it works on my machine" problems, enables consistent environments across development, staging, and production, and is the foundation of modern cloud-native applications.

### Images vs Containers

- **Image**: Read-only template with instructions for creating a container
- **Container**: Runnable instance of an image (has a writable layer)

### Dockerfile Best Practices

```dockerfile
FROM node:22-alpine AS base
WORKDIR /app

# 1. Install dependencies first (layer caching)
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# 2. Copy source and build
COPY . .
RUN npm run build

# 3. Production stage (multi-stage build)
FROM node:22-alpine AS production
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app

COPY --from=base --chown=appuser:appgroup /app/dist ./dist
COPY --from=base --chown=appuser:appgroup /app/node_modules ./node_modules
COPY --from=base --chown=appuser:appgroup /app/package.json ./

USER appuser
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

ENV NODE_ENV=production
CMD ["node", "dist/index.js"]
```

### Dockerfile Instructions

| Instruction | Purpose | Best Practice |
|-------------|---------|--------------|
| `FROM` | Base image | Use specific tags (`node:22-alpine`), not `latest` |
| `WORKDIR` | Working directory | Always set before COPY/RUN |
| `COPY` | Copy files | Use `.dockerignore` to exclude unnecessary files |
| `RUN` | Execute commands | Combine commands to reduce layers |
| `EXPOSE` | Document port | Informational only |
| `ENV` | Environment variables | Use `NODE_ENV=production` |
| `USER` | Run as non-root | Create and use app user |
| `HEALTHCHECK` | Container health | Critical for orchestration |
| `CMD` / `ENTRYPOINT` | Default command | Prefer `CMD` for runnable args |

### Docker Layers

Each instruction in a Dockerfile creates a layer. Layers are cached and reused.

```dockerfile
FROM node:22-alpine          # Layer 1: base OS
COPY package*.json ./         # Layer 2: deps (cached if package.json unchanged)
RUN npm ci                    # Layer 3: dependencies
COPY . .                      # Layer 4: source (changes frequently)
RUN npm run build             # Layer 5: build
CMD ["node", "dist/index.js"] # Layer 6: start command
```

### Multi-stage Builds

```dockerfile
# Stage 1: Build
FROM node:22 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production (smaller image)
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Docker Networking

| Network Type | Isolation | Use Case |
|-------------|-----------|----------|
| `bridge` (default) | Containers on same bridge can communicate | Single-host apps |
| `host` | No isolation (uses host networking) | Performance-critical apps |
| `none` | No networking | Isolated containers |
| `overlay` | Multi-host networking | Docker Swarm, Kubernetes |

```bash
docker network create my-network
docker run -d --name api --network my-network -p 3000:3000 my-api
docker network connect my-network my-container
```

### Docker Volumes

```bash
docker volume create pgdata
docker run -d -v pgdata:/var/lib/postgresql/data postgres:16
docker run -d -v /host/path:/container/path my-app
docker run -d --tmpfs /tmp my-app
```

| Mount Type | Persistence | Performance |
|------------|-------------|-------------|
| Named volume | Persistent | Good |
| Bind mount | Depends on host | Good |
| tmpfs | Non-persistent | Best |

### Docker Compose

```yaml
version: "3.9"

services:
  app:
    build:
      context: .
      target: production
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]

  redis:
    image: redis:7-alpine
    volumes:
      - redisdata:/data
    command: redis-server --appendonly yes

volumes:
  pgdata:
  redisdata:
```

### Common Docker Mistakes

- Using `latest` tag (unpredictable builds)
- Running containers as root
- Storing secrets in images (use build args or secrets mount)
- Creating fat images (use multi-stage builds)
- Not using `.dockerignore` (large context, slow builds)
- Installing unnecessary packages (increase attack surface)
- Not setting memory limits (container can use all host memory)

> ⚠️ **Warning:** Using `:latest` means you get different images across builds and environments. Always pin to a specific version tag like `node:22-alpine` or a digest `node@sha256:...`.

> ⚠️ **Warning:** Docker build arguments (`--build-arg`) are visible in the image history. Never pass secrets like API keys or passwords via build args. Use Docker BuildKit's `--secret` mount instead.

> ✅ **Best Practice:** Use multi-stage builds to keep production images small. The final stage should only contain the runtime binary and minimal dependencies — no build tools, no source code.

> ✅ **Best Practice:** Always run containers as a non-root user. Create an app user in the Dockerfile with `RUN adduser -D appuser && USER appuser` (Alpine) or `RUN useradd -m appuser && USER appuser` (Debian).

> ❓ **Interview Question:** What is the difference between `CMD` and `ENTRYPOINT` in a Dockerfile?
> **Answer:** `CMD` provides default arguments that can be overridden when running the container. `ENTRYPOINT` configures a container to run as an executable — its arguments are not easily overridden. They are often combined: `ENTRYPOINT ["node"]` + `CMD ["app.js"]`.

> 💡 **Pro Tip:** Use `docker scout` (Docker's built-in vulnerability scanner) to analyze images for CVEs before pushing to a registry. Integrate it into your CI/CD pipeline for automated security gates.

### Real World Analogy

Docker images are like Russian nesting dolls (matryoshka). The base OS image is the outer doll. Each `RUN`/`COPY` instruction adds another layer inside. Multi-stage builds are like building a prototype (builder stage) and then only keeping the finished product (production stage). The `.dockerignore` file is the packaging list — it tells Docker what NOT to pack when shipping.

### Practical Exercises

1. Write a Dockerfile for a Node.js app using multi-stage builds: builder stage → production stage
2. Create a `.dockerignore` file and measure the build context size with and without it
3. Run a container with `--memory=256m --cpus=0.5` and verify limits using `docker stats`
4. Set up Docker Compose with three services: app, PostgreSQL, and Redis with health checks
5. Use `docker scout` to scan your production image and fix any critical vulnerabilities

### Revision Notes

- **Images vs Containers**: Image = template/blueprint; Container = running instance
- **Dockerfile layering**: Each instruction creates a cacheable layer; order matters
- **Multi-stage builds**: `FROM ... AS builder` → `COPY --from=builder`
- **Networking**: `bridge` (default, isolated), `host` (shared), `overlay` (multi-host)
- **Volumes**: Named (managed), Bind (host path), tmpfs (memory-only)
- **Health checks**: `HEALTHCHECK CMD curl ...` — required for orchestration
- **Compose**: Define multi-container apps in YAML with `docker compose up`

---

## Chapter 71: Kubernetes

### Introduction

Kubernetes (K8s) is an open-source container orchestration platform for automating deployment, scaling, and management of containerized applications.

### Why It Matters

Kubernetes has become the standard for running production container workloads. It provides self-healing, auto-scaling, service discovery, rolling updates, and infrastructure abstraction.

### Architecture

- **Control Plane**: API Server, Scheduler, Controller Manager, etcd
- **Worker Nodes**: kubelet, kube-proxy, container runtime
- **Pods**: Smallest deployable unit (one or more containers)

### Core Resources

```yaml
# Pod
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  containers:
    - name: app
      image: my-app:latest
      ports:
        - containerPort: 3000
      resources:
        requests:
          memory: "256Mi"
          cpu: "250m"
        limits:
          memory: "512Mi"
          cpu: "500m"
      livenessProbe:
        httpGet:
          path: /health
          port: 3000
        initialDelaySeconds: 10
      readinessProbe:
        httpGet:
          path: /ready
          port: 3000
        initialDelaySeconds: 5
```

```yaml
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: app
          image: my-app:latest
          ports:
            - containerPort: 3000
```

```yaml
# Service
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP  # ClusterIP | NodePort | LoadBalancer
```

```yaml
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  NODE_ENV: production
  LOG_LEVEL: info

# Secret
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQxMjM=
```

```yaml
# Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-service
                port:
                  number: 80
```

```yaml
# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

```yaml
# HorizontalPodAutoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

### kubectl Commands

| Command | Purpose |
|---------|---------|
| `kubectl get pods` | List pods |
| `kubectl get deployments` | List deployments |
| `kubectl get services` | List services |
| `kubectl logs pod-name` | View pod logs |
| `kubectl describe pod pod-name` | Detailed pod info |
| `kubectl exec -it pod-name -- /bin/sh` | Shell into container |
| `kubectl apply -f manifest.yaml` | Create/update resources |
| `kubectl delete -f manifest.yaml` | Delete resources |
| `kubectl port-forward pod-name 8080:3000` | Port forwarding |
| `kubectl top pods` | Resource usage |

### Common Mistakes

- Not setting resource requests and limits
- Using `latest` tag for images (unpredictable deployments)
- Not configuring liveness and readiness probes
- Storing secrets in ConfigMaps
- Exposing pods directly instead of using Services
- Running as root in containers

> ⚠️ **Warning:** Not setting resource requests or limits can cause the scheduler to overcommit nodes, leading to pod evictions and node instability. Always set both `requests` (guaranteed minimum) and `limits` (hard cap).

> ⚠️ **Warning:** Labels and selectors must match exactly. A typo in a label selector means the Service won't route traffic to your Pods, and there is no error message — traffic will just silently fail.

> ✅ **Best Practice:** Define PodDisruptionBudgets for critical workloads to ensure minimum availability during voluntary disruptions (node maintenance, cluster upgrades).

> ✅ **Best Practice:** Use NetworkPolicies to enforce micro-segmentation. By default, all pods can communicate with each other — explicit NetworkPolicies restrict traffic to only what's necessary.

> ❓ **Interview Question:** What happens when a node runs out of resources in a Kubernetes cluster?
> **Answer:** Kubernetes evicts pods based on their QoS class: BestEffort (no limits) first, then Burstable (requests < limits), then Guaranteed (requests == limits). The kubelet terminates pods to reclaim resources.

> 💡 **Pro Tip:** Use `kubectl get events --sort-by='.lastTimestamp'` to see the most recent cluster events, which is often faster than `kubectl describe` for troubleshooting.

### Real World Analogy

Kubernetes is like a hotel management system. The **control plane** is the front desk (API Server) and management office (Scheduler, Controller Manager). **Worker nodes** are the guest rooms/floors. **Pods** are the guests staying in rooms. **Services** are the room phone directory — you call a service name and get connected. **Deployments** are the reservation system ensuring the right number of rooms are ready. The **HPA** is like adjusting staff levels based on occupancy.

### Practical Exercises

1. Deploy a simple web app using a Deployment with 3 replicas, resource limits, and probes
2. Expose the deployment via a ClusterIP Service, then port-forward to access it locally
3. Create a ConfigMap for app config and mount it as environment variables in pods
4. Set up a HorizontalPodAutoscaler that scales based on CPU utilization above 70%
5. Perform a rolling update: change the image tag, watch `kubectl rollout status`, then rollback

### Revision Notes

- **Pods**: Smallest unit; wrap one or more containers with shared storage/network
- **Deployments**: Declarative updates for Pods and ReplicaSets
- **Services**: Stable network endpoint — ClusterIP (internal), NodePort (node port), LoadBalancer (cloud LB)
- **ConfigMaps/Secrets**: Decouple config from pods; Secrets are base64 encoded (not encrypted by default)
- **Ingress**: HTTP/HTTPS routing to Services, supports TLS termination
- **HPA**: Auto-scales based on CPU, memory, or custom metrics
- **Probes**: `livenessProbe` (restart if dead), `readinessProbe` (stop traffic if not ready), `startupProbe` (slow-start apps)

---

## Chapter 72: Helm

### Introduction

Helm is the package manager for Kubernetes. It uses Charts (packaged Kubernetes resources) to define, install, and upgrade applications.

### Chart Structure

```
my-chart/
  Chart.yaml          # Metadata
  values.yaml         # Default configuration
  charts/             # Sub-chart dependencies
  templates/          # Kubernetes manifest templates
    deployment.yaml
    service.yaml
    _helpers.tpl      # Named templates (partials)
```

### Chart.yaml

```yaml
apiVersion: v2
name: my-app
description: A full-stack application
type: application
version: 1.0.0
appVersion: "1.0.0"
dependencies:
  - name: postgresql
    version: "12.x"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
```

### values.yaml

```yaml
replicaCount: 3

image:
  repository: my-app
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  requests:
    memory: 256Mi
    cpu: 250m
  limits:
    memory: 512Mi
    cpu: 500m
```

### Helm Commands

```bash
# Install a chart
helm install my-release ./my-chart
helm install my-release ./my-chart -f prod-values.yaml

# Upgrade
helm upgrade my-release ./my-chart
helm upgrade my-release ./my-chart --set image.tag=v2.0.0

# Rollback
helm rollback my-release 1

# Manage
helm list
helm history my-release
helm uninstall my-release

# Template (dry run)
helm template ./my-chart
helm install my-release ./my-chart --dry-run --debug

# Repositories
helm repo add bitnami https://charts.bitnami.com/bitnami
helm search repo postgresql
```

### Common Mistakes

- Not using `.Values` in templates (hardcoding)
- Forgetting whitespace control (`{{-` / `-}}`)
- Storing secrets in values.yaml
- Not versioning charts properly
- Ignoring dependency updates

> ⚠️ **Warning:** Storing secrets (passwords, tokens) in `values.yaml` and committing them to Git is a security risk. Use SealedSecrets, External Secrets Operator, or a vault solution like HashiCorp Vault for sensitive data.

> ⚠️ **Warning:** Forgetting the whitespace control characters `{{-` (chomp left) and `-}}` (chomp right) in Go templates results in excessive newlines and malformed YAML. Always verify rendered output with `helm template`.

> ✅ **Best Practice:** Use `helm lint` and `helm template --debug` before any install or upgrade. This catches template syntax errors and shows the exact YAML that will be applied.

> ✅ **Best Practice:** Separate environment-specific overrides into dedicated files: `values-dev.yaml`, `values-staging.yaml`, `values-prod.yaml`, and apply with `-f` flags in the correct order.

> ❓ **Interview Question:** How does Helm manage rollbacks and what information does it store?
> **Answer:** Helm stores release history as Secrets (or ConfigMaps in Helm 2) in the same namespace. Each release revision records the manifest, values, and metadata. `helm rollback` reapplies the previous revision's manifest.

> 💡 **Pro Tip:** Use `helm get manifest RELEASE_NAME` to see the actual Kubernetes resources generated for a release. This is invaluable for debugging when the deployed resources don't match expectations.

### Real World Analogy

Helm is like an appliance installation manual plus a parts catalog. The **Chart** is the instruction booklet. **values.yaml** is the customization checklist (color, size, features). **Templates** are the assembly instructions with blanks to fill in from the checklist. A **repository** is the store where you browse available appliances. `helm upgrade` is like upgrading your appliance with new features while keeping your custom settings.

### Practical Exercises

1. Create a Helm chart for a web application with Deployment, Service, and Ingress templates
2. Use `values.yaml` with sensible defaults and create a `prod-values.yaml` that overrides replica count and resources
3. Run `helm template` and `helm lint` on your chart, observe the rendered YAML output
4. Install the chart, upgrade it with a new image tag, then rollback to the previous revision
5. Add a subchart dependency (e.g., PostgreSQL from Bitnami) and configure it via your parent chart's values

### Revision Notes

- **Chart structure**: `Chart.yaml`, `values.yaml`, `templates/`, `charts/`, `_helpers.tpl`
- **Go templates**: `{{ .Values.replicaCount }}`, `{{- with ... }}`, `range`, `if/else`
- **Commands**: `helm install`, `upgrade`, `rollback`, `list`, `history`, `uninstall`
- **`--dry-run --debug`**: Preview rendered manifests before applying
- **Dependencies**: Managed in `Chart.yaml` under `dependencies[]`, fetched with `helm dependency update`
- **Versioning**: `version` (chart version) and `appVersion` (application version)

---

## Chapter 73: Terraform

### Introduction

Terraform is an Infrastructure as Code (IaC) tool that enables you to define and provision infrastructure using a declarative configuration language (HCL).

### Why It Matters

Terraform manages infrastructure across multiple cloud providers, tracks state, and enables version-controlled, reproducible infrastructure deployments.

### HCL Syntax

```hcl
terraform {
  required_version = ">= 1.5"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "instance_type" {
  type    = string
  default = "t3.medium"
}

resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "main-vpc"
    Environment = var.environment
  }
}

resource "aws_subnet" "public" {
  count                   = 3
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index)
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet-${count.index}"
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public[0].id

  root_block_device {
    volume_size = 30
    volume_type = "gp3"
  }

  tags = {
    Name = "web-server"
  }
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-22.04-amd64-server-*"]
  }
}

data "aws_availability_zones" "available" {
  state = "available"
}

output "vpc_id" {
  value = aws_vpc.main.id
}

output "instance_ip" {
  value = aws_instance.web.public_ip
}
```

### Terraform State

- **Purpose**: Maps real-world resources to your configuration
- **Storage**: Local (terraform.tfstate) or remote (S3, Terraform Cloud)
- **Best Practice**: Always use remote state with locking

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/network/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

### Modules

```hcl
module "vpc" {
  source   = "terraform-aws-modules/vpc/aws"
  version  = "5.0.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  enable_vpn_gateway = false

  tags = {
    Environment = "production"
  }
}
```

### Terraform Commands

```bash
# Initialize (download providers, modules)
terraform init

# Format code
terraform fmt

# Validate configuration
terraform validate

# Plan changes
terraform plan
terraform plan -out=tfplan

# Apply changes
terraform apply
terraform apply tfplan

# Destroy resources
terraform destroy

# Manage state
terraform state list
terraform state show aws_instance.web
terraform state rm aws_instance.web
terraform import aws_instance.web i-1234567890abcdef0
```

### Common Mistakes

- Storing sensitive values in plain text (use `sensitive = true` or vault)
- Not using remote state (team collaboration issues)
- Manually modifying resources outside Terraform (drift)
- Not using modules for reusable infrastructure
- Hardcoding values instead of using variables
- Forgetting `terraform fmt` and `terraform validate`

> ⚠️ **Warning:** Manually modifying infrastructure created by Terraform (e.g., changing an RDS instance type in the console) causes **drift**. Terraform state no longer matches reality, and the next `apply` may overwrite or destroy manual changes.

> ⚠️ **Warning:** Terraform state files contain sensitive information in plain text (passwords, ARNs, IPs). Never commit `.tfstate` files to Git. Always use remote state with encryption at rest and in transit.

> ✅ **Best Practice:** Always use remote state with a locking mechanism (S3 + DynamoDB for AWS, GCS + object versioning for GCP, Azure Storage with lease blobs). This prevents concurrent modifications by multiple team members.

> ✅ **Best Practice:** Run `terraform plan` as part of your CI/CD pipeline with a human approval gate before `apply`. Use GitHub's PR comments or GitLab MR summaries to share the plan output with reviewers.

> ❓ **Interview Question:** What happens if `terraform apply` fails partway through?
> **Answer:** Terraform follows an "apply incrementally" model. Resources that were already created remain in the state file. You can fix the error and re-run `apply`, which will skip already-created resources (found in state) and continue with the remaining ones.

> 💡 **Pro Tip:** Use `terraform workspace` to manage multiple environments (dev, staging, prod) with the same configuration. Each workspace has its own state file, allowing you to reference the workspace name in your configuration with `${terraform.workspace}`.

### Real World Analogy

Terraform is like an architectural blueprint for a building. The **HCL configuration** is the blueprint itself. **State** is the as-built survey showing what was actually constructed. **Modules** are prefabricated sections (standardized rooms, electrical layouts). **Plan** is the contractor's estimate showing what changes will be made. **Remote state** is the shared document repository so all architects work from the same plans.

### Practical Exercises

1. Write Terraform config to create an AWS VPC with public/private subnets, use `terraform plan` and `terraform apply`
2. Migrate from local state to remote state with S3 backend and DynamoDB locking
3. Create a reusable module for an EC2 instance with security group and deploy it in two environments
4. Use `terraform import` to bring an existing manually-created resource under Terraform management
5. Implement workspaces for dev/staging/prod environments and demonstrate switching between them

### Revision Notes

- **HCL**: HashiCorp Configuration Language — declarative, human-readable
- **Core commands**: `init` (download providers/modules), `fmt` (format), `validate`, `plan`, `apply`, `destroy`
- **State**: Maps config to real resources — always use remote state with locking
- **Variables**: `variable "name" {}` + `var.name` — use `.tfvars` files for environment values
- **Modules**: Reusable infrastructure packages from registry or custom
- **Expressions**: `count`, `for_each`, `for`, `lookup`, `cidrsubnet` for dynamic infrastructure
- **Sensitive data**: Use `sensitive = true`, vault providers, or encrypted variables

---

## Chapter 74: Nginx

### Introduction

Nginx is a high-performance web server, reverse proxy, load balancer, and HTTP cache. It handles thousands of concurrent connections with low memory usage.

### Core Configuration

```nginx
# /etc/nginx/nginx.conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
    multi_accept on;
    use epoll;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"'

    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    include /etc/nginx/conf.d/*.conf;
}
```

### Reverse Proxy

```nginx
server {
    listen 80;
    server_name api.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    location / {
        proxy_pass http://node_app;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 60s;
    }

    location /static/ {
        alias /var/www/my-app/public/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    location /health {
        proxy_pass http://node_app/health;
        access_log off;
    }

    client_max_body_size 10M;
}
```

### Load Balancing

```nginx
upstream node_app {
    least_conn;
    server 10.0.1.10:3001 weight=3;
    server 10.0.1.11:3001 weight=2;
    server 10.0.1.12:3001;
    keepalive 32;
}

upstream api_servers {
    ip_hash;  # session persistence
    server api1.example.com;
    server api2.example.com;
}
```

### Static File Serving & Caching

```nginx
server {
    listen 80;
    server_name static.example.com;
    root /var/www/static;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location ~* \.(jpg|jpeg|png|gif|ico|css|js|webp)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }

    location ~* \.(html|json)$ {
        expires 1h;
        add_header Cache-Control "public, must-revalidate";
    }
}
```

### Rate Limiting

```nginx
http {
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_conn_zone $binary_remote_addr zone=addr:10m;

    server {
        location /api/ {
            limit_req zone=api burst=20 nodelay;
            limit_conn addr 10;
            proxy_pass http://api_backend;
        }
    }
}
```

### Common Mistakes

- Not setting worker_processes to `auto`
- Not enabling `sendfile` and `tcp_nopush`
- Using default error pages (leaks server info)
- Not setting proper SSL ciphers
- Forgetting to test config with `nginx -t`
- Not configuring log rotation

> ⚠️ **Warning:** Not setting `worker_processes auto` means your Nginx instance won't utilize all available CPU cores, leaving performance on the table. Always let Nginx detect the optimal number.

> ⚠️ **Warning:** Leaking Nginx version and server information in error pages and headers is a security risk. Add `server_tokens off;` in the `http` block to hide version details.

> ✅ **Best Practice:** Always run `nginx -t` before reloading configuration. A syntax error means Nginx will fail to restart, taking your service down. Use `nginx -s reload` for zero-downtime config changes.

> ✅ **Best Practice:** Enable `gzip` (or `brotli`) compression for text-based responses (HTML, CSS, JS, JSON). This reduces bandwidth by 60-80% with minimal CPU overhead.

> ❓ **Interview Question:** How does Nginx handle request processing differently from Apache?
> **Answer:** Nginx uses an event-driven, asynchronous, non-blocking architecture with a single master process and multiple worker processes. Apache uses a process/thread-per-connection model. Nginx handles thousands of concurrent connections with far less memory.

> 💡 **Pro Tip:** Use the `$upstream_response_time` variable in your log format to capture backend response times. This helps identify slow upstream services without external monitoring tools.

### Real World Analogy

Nginx is like a highly efficient airport control tower. The **worker processes** are air traffic controllers — they handle multiple flights (connections) simultaneously using an event-driven approach. The **reverse proxy** is the transfer desk routing passengers to the correct gate. **Load balancing** distributes arriving flights across multiple runways. **Rate limiting** is the check-in counter that prevents too many passengers from overwhelming security.

### Practical Exercises

1. Configure Nginx as a reverse proxy for a Node.js app running on port 3000 with SSL termination
2. Set up load balancing across three upstream servers with least_conn algorithm and health checks
3. Implement rate limiting (10 requests per second) for an API endpoint with a burst of 20
4. Configure static file serving with cache headers (1 year for assets, 1 hour for HTML)
5. Enable gzip compression, set up custom error pages, and verify with `curl -I`

### Revision Notes

- **`worker_processes auto`**: Automatically uses all CPU cores
- **`events { worker_connections 1024; }`**: Max simultaneous connections per worker
- **Reverse proxy**: `proxy_pass http://upstream;` with header forwarding
- **Load balancing**: `upstream` block with `least_conn`, `ip_hash`, or default round-robin
- **Rate limiting**: `limit_req_zone` + `limit_req` with burst and nodelay
- **SSL**: Modern protocols (`TLSv1.2 TLSv1.3`), strong ciphers, `ssl_certificate` + `ssl_certificate_key`
- **Testing**: Always `nginx -t` before `nginx -s reload`

---

## Chapter 75: CI/CD Pipelines

### Introduction

A CI/CD pipeline automates the steps to get code from commit to production: build, test, deploy. Continuous Integration (CI) merges code changes frequently and runs automated tests. Continuous Deployment/Delivery (CD) automatically deploys to production or staging.

### Pipeline Stages

| Stage | Activities | Tools |
|-------|------------|-------|
| Source | Checkout code, lint | Git, ESLint, Prettier |
| Build | Compile, bundle, build artifacts | Webpack, Vite, tsc |
| Test | Unit, integration, e2e | Jest, Cypress, Playwright |
| Security | SAST, dependency scan, secrets scan | Snyk, SonarQube, Trivy |
| Package | Build Docker image, push to registry | Docker, kaniko |
| Deploy Staging | Deploy to staging, smoke tests | Helm, kubectl, Terraform |
| Deploy Production | Deploy to production, canary/blue-green | ArgoCD, Spinnaker |
| Verify | Health checks, monitoring, alerts | Prometheus, Datadog |

### CI/CD Best Practices

- **Pipeline as Code**: Define pipelines in version control
- **Fast Feedback**: Run fast tests first, slow tests in parallel
- **Deterministic Builds**: Use lockfiles and pinned versions
- **Immutable Artifacts**: Build once, promote through environments
- **Environment Parity**: Dev, staging, production should be as similar as possible
- **Security Scanning**: Integrate SAST, dependency scanning, and container scanning
- **Rollback Strategy**: Always have a way to revert deployments

> ⚠️ **Warning:** Skipping the security scanning stage in your CI/CD pipeline is a common source of production vulnerabilities. Integrate at least a dependency vulnerability scanner (e.g., Snyk, Dependabot, Trivy) before the build stage.

> ⚠️ **Warning:** Environment drift between dev, staging, and production is one of the top causes of deployment failures. Use Docker containers and IaC (Terraform, CloudFormation) to enforce environment parity.

> ✅ **Best Practice:** Implement the "build once, deploy everywhere" principle. Build a single immutable artifact (Docker image) in CI, store it in a registry, and promote the exact same artifact through staging and production.

> ✅ **Best Practice:** Set up a rollback mechanism before every deployment. Whether it's `kubectl rollout undo`, a Helm rollback, or a blue-green switch, ensure you can revert within minutes.

> ❓ **Interview Question:** What's the difference between Continuous Delivery and Continuous Deployment?
> **Answer:** Continuous Delivery means every change that passes all stages of the pipeline is potentially deployable but requires manual approval for production. Continuous Deployment automatically deploys every change that passes the pipeline to production with no manual gate.

> 💡 **Pro Tip:** Use semantic release (or similar) to automate versioning and changelog generation based on conventional commits. This eliminates manual version bumps and release notes.

### Real World Analogy

A CI/CD pipeline is like an airport baggage handling system. **Source** is the passenger dropping off their luggage. **Build** is the sorting machine. **Test** is the security scanner checking for prohibited items. **Deploy Staging** is the test flight. **Deploy Production** is the actual flight taking off. **Rollback** is recalling a plane that hasn't taken off yet. Each stage must pass before moving to the next, just like luggage must pass through each checkpoint.

### Practical Exercises

1. Design a complete CI/CD pipeline with all stages (source → build → test → security → package → deploy staging → verify → deploy production)
2. Implement a blue-green deployment strategy using Kubernetes Services and label selectors
3. Set up a canary deployment that routes 10% of traffic to the new version for 5 minutes before full rollout
4. Configure a rollback mechanism using Helm that automatically reverts if health checks fail within the first 3 minutes
5. Integrate a security scanner (e.g., Trivy) into an existing CI pipeline to scan Docker images for vulnerabilities

### Revision Notes

- **CI**: Merge code frequently, run automated tests for fast feedback
- **CD**: Automatically deploy to staging (Continuous Delivery) or production (Continuous Deployment)
- **Pipeline stages**: Source → Build → Test → Security → Package → Deploy → Verify
- **Deployment strategies**: Rolling (gradual), Blue-Green (switch), Canary (percentage), Recreate (kill-all)
- **Key principles**: Pipeline as Code, build once, fast feedback, environment parity, rollback readiness

### Deployment Strategies

| Strategy | Description | Downtime | Risk | Complexity |
|----------|-------------|----------|------|------------|
| Rolling | Gradual replacement of instances | None | Low | Low |
| Blue-Green | Two identical environments, switch traffic | None | Medium | Medium |
| Canary | Small subset first, then full rollout | None | Low | High |
| A/B Testing | Route by user segment for testing | None | Low | High |
| Recreate | Kill all, start new | Yes | High | Low |

---

## Chapter 76: Monitoring — Prometheus & Grafana

### Introduction

Prometheus is an open-source monitoring and alerting toolkit that collects metrics from targets at regular intervals. Grafana provides visualization and dashboards.

### Prometheus Architecture

- **Prometheus Server**: Scrapes and stores metrics
- **Exporters**: Expose metrics from applications/node (node_exporter, blackbox_exporter)
- **Alertmanager**: Handles alerts, deduplication, silencing
- **Pushgateway**: For short-lived job metrics
- **Service Discovery**: Automatically finds targets (Kubernetes, Consul)

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alerts.yml"

scrape_configs:
  - job_name: "node"
    static_configs:
      - targets: ["localhost:9100"]

  - job_name: "api"
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_label_app]
        regex: my-api
        action: keep
```

### PromQL (Prometheus Query Language)

```promql
# CPU usage percentage
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage percentage
100 * (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)

# Request rate per second
rate(http_requests_total{status=~"5.."}[5m])

# 95th percentile latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Alert: high error rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
```

### Grafana Dashboards

- Use community dashboards from grafana.com/dashboards
- Create panels: Time series, Stats, Table, Bar gauge, Heatmap
- Set up alerts with notification channels (Slack, PagerDuty, Email)
- Use annotations for deployments

### Common Mistakes

- Not setting retention and storage limits (disk fills up)
- Using too high scrape frequency (wasteful)
- Not configuring alerting rules
- Not using recording rules for expensive queries
- Overly complex PromQL queries

> ⚠️ **Warning:** Without configuring retention and storage limits, Prometheus will consume all available disk space. Always set `--storage.tsdb.retention.time=30d` and `--storage.tsdb.retention.size=50GB` flags.

> ⚠️ **Warning:** High-cardinality labels (like user IDs, email addresses, or session IDs in metric labels) can cause Prometheus to consume excessive memory and crash. Each unique label combination creates a new time series.

> ✅ **Best Practice:** Monitor the four golden signals of monitoring: **Latency** (time to serve requests), **Traffic** (request rate), **Errors** (error rate), and **Saturation** (how "full" your service is).

> ✅ **Best Practice:** Use recording rules for expensive or frequently executed PromQL queries. They pre-compute results on a schedule and store them as new time series, reducing query load on the Prometheus server.

> ❓ **Interview Question:** How does Prometheus's pull-based model differ from a push-based monitoring system like DataDog?
> **Answer:** Prometheus scrapes targets at regular intervals (pull), which means the monitoring system controls the collection rate and can detect when targets are down. Push-based systems require targets to send data, making it harder to detect silent failures.

> 💡 **Pro Tip:** Use `rate()` instead of `irate()` for alerting rules. `rate()` averages over a time window, providing smoother results, while `irate()` uses only the last two data points and is more sensitive to spikes but less reliable for alerting.

### Real World Analogy

Prometheus and Grafana are like a car's dashboard and warning system. **Prometheus** is the network of sensors collecting data: speed (request rate), engine temperature (CPU), fuel level (disk space), tire pressure (memory). **PromQL** is the query language to ask questions like "What was the average speed in the last 5 minutes?". **Grafana** is the dashboard display — speedometer, fuel gauge, temperature gauge, and the check engine light (alerts).

### Practical Exercises

1. Install and configure Prometheus with node_exporter and verify metrics are being scraped
2. Write PromQL queries for: CPU usage by instance, memory usage percentage, request error rate, and 95th percentile latency
3. Create a Grafana dashboard with panels for the four golden signals of a web application
4. Configure Alertmanager to send alerts to Slack when error rate exceeds 5% for 5 minutes
5. Set up recording rules for expensive queries and compare query performance with and without them

### Revision Notes

- **Prometheus**: Pull-based monitoring, TSDB storage, PromQL query language
- **Architecture**: Prometheus Server → Exporters / Pushgateway → Alertmanager
- **PromQL**: `rate()`, `histogram_quantile()`, `avg by(instance)`, recording rules
- **Grafana**: Dashboard visualization, alerting, annotation support
- **Four golden signals**: Latency, Traffic, Errors, Saturation
- **Alertmanager**: Deduplication, grouping, routing, silencing, inhibition
- **Cardinality**: Keep label values bounded — never use user IDs, emails, or random values as labels

---

## Chapter 77: ELK Stack (Elasticsearch, Logstash, Kibana)

### Introduction

The ELK Stack is a set of three open-source tools for log management: Elasticsearch (storage/search), Logstash (ingest/transform), and Kibana (visualization).

### Architecture

```
Application Logs -> Filebeat -> Logstash -> Elasticsearch -> Kibana
                                      |
                              (parse, enrich, filter)
```

### Logstash Configuration

```conf
input {
  beats {
    port => 5044
  }
}

filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }

  date {
    match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]
  }

  mutate {
    remove_field => ["message", "original"]
  }

  useragent {
    source => "agent"
    target => "user_agent"
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```

### Elasticsearch Indexing

```json
PUT /logs-2026.07.04
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "refresh_interval": "5s"
  },
  "mappings": {
    "properties": {
      "@timestamp": { "type": "date" },
      "level": { "type": "keyword" },
      "message": { "type": "text" },
      "service": { "type": "keyword" },
      "duration_ms": { "type": "integer" },
      "status_code": { "type": "integer" },
      "user_agent": { "type": "keyword" }
    }
  }
}
```

### Filebeat

```yaml
filebeat.inputs:
  - type: container
    paths:
      - /var/log/containers/*.log

output.logstash:
  hosts: ["logstash:5044"]
```

### Common Mistakes

- Not setting index lifecycle management (ILM) — disk fills up
- Using too many shards (wasteful)
- Not structuring logs in JSON format from the application
- Making heavy aggregations on text fields (use keyword fields)
- Not securing Elasticsearch (no auth, exposed to internet)

> ⚠️ **Warning:** Exposing Elasticsearch to the internet without authentication is a common security incident waiting to happen. Always enable X-Pack security, use TLS, and never expose the Elasticsearch port (9200) directly.

> ⚠️ **Warning:** Without ILM, Elasticsearch will keep all indices indefinitely, eventually filling the disk. Configure ILM policies to migrate from hot → warm → cold → delete based on age or size thresholds.

> ✅ **Best Practice:** Always output application logs in structured JSON format. This allows Logstash or Elasticsearch to parse fields automatically without complex grok patterns and makes querying specific fields trivial.

> ✅ **Best Practice:** Use index templates to define consistent mappings and settings for all indices matching a pattern (e.g., `logs-*`). This ensures every new index has the correct field types and shard configuration.

> ❓ **Interview Question:** What is the difference between Logstash and Filebeat in the ELK stack?
> **Answer:** Filebeat is a lightweight, resource-efficient shipper that reads log files and forwards them. Logstash is a heavier data processing pipeline that can parse, transform, enrich, and filter logs from multiple inputs. Filebeat sends to Logstash for processing or directly to Elasticsearch.

> 💡 **Pro Tip:** Use Elasticsearch's `_cat/indices` API (`GET _cat/indices?v&s=store.size:desc`) to quickly identify the largest indices and investigate if they need ILM policies or optimization.

### Real World Analogy

The ELK Stack is like a library system for your logs. **Filebeat** is the bookmobile collecting logs from all neighborhoods (servers). **Logstash** is the cataloging department that categorizes, stamps, and enriches each book. **Elasticsearch** is the library shelves storing books by date (index) so they can be found instantly. **Kibana** is the reading room where you search, browse, and visualize your collection.

### Practical Exercises

1. Install the ELK stack (Elasticsearch, Logstash, Kibana) using Docker Compose
2. Configure Filebeat to ship Nginx access logs to Logstash, parse them with a grok filter, and index in Elasticsearch
3. Write structured JSON logs from a Node.js application and send them directly to Elasticsearch
4. Create an ILM policy that moves indices to warm after 7 days, cold after 30 days, and deletes after 90 days
5. Build a Kibana dashboard showing error rates, response time percentiles, and top error messages

### Revision Notes

- **ELK**: Elasticsearch (storage/search) + Logstash (ingest/transform) + Kibana (visualization)
- **Filebeat**: Lightweight log shipper — forward from source to Logstash/ES
- **Logstash**: Pipeline with `input` → `filter` (grok, mutate, date) → `output`
- **Elasticsearch**: Distributed search engine, indices with shards/replicas
- **ILM**: Index lifecycle management — hot → warm → cold → delete
- **Kibana**: Dashboards, Discover (search), Alerts, Canvas
- **Best practice**: JSON logs → structured fields → index templates → ILM → Kibana alerts

---
