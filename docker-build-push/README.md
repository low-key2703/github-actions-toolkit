# Docker Build & Push Action

Build and push Docker images with advanced features: layer caching, multi-platform support, and flexible tagging.

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `context` | ❌ No | `.` | Build context path |
| `image` | ✅ Yes | - | Image name (e.g., `myapp`, `username/myapp`) |
| `tags` | ❌ No | `latest` | Comma-separated tags (e.g., `latest,1.0.0,stable`) |
| `registry` | ❌ No | `docker.io` | Container registry URL |
| `username` | ❌ No | `''` | Registry username |
| `password` | ❌ No | `''` | Registry password or token |
| `dockerfile` | ❌ No | `Dockerfile` | Path to Dockerfile |
| `push` | ❌ No | `true` | Push image to registry (true/false) |
| `platforms` | ❌ No | `linux/amd64` | Target platforms |
| `build-args` | ❌ No | `''` | Build arguments (one per line) |

## Features

- **Layer Caching** - Faster rebuilds using registry cache
- **Multi-Platform** - Build for ARM64, AMD64, and more simultaneously
- **Multiple Tags** - Tag one build with multiple versions
- **Build Args** - Pass variables to Dockerfile
- **Flexible Registries** - Docker Hub, GHCR, ECR, GCR, and more

## Usage Examples

### Basic Build and Push
```yaml
- name: Build and push
  uses: ./docker-build-push
  with:
    image: 'myapp'
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}
```

### Multiple Tags
```yaml
- name: Build with multiple tags
  uses: ./docker-build-push
  with:
    image: 'myapp'
    tags: 'latest,1.0.0,${{ github.sha }}'
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}
```

### Custom Dockerfile Location
```yaml
- name: Build from custom Dockerfile
  uses: ./docker-build-push
  with:
    context: './backend'
    dockerfile: 'docker/Dockerfile.prod'
    image: 'backend'
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}
```

### Multi-Platform Build
```yaml
- name: Build for multiple architectures
  uses: ./docker-build-push
  with:
    image: 'myapp'
    platforms: 'linux/amd64,linux/arm64'
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}
```

### Build with Arguments
```yaml
- name: Build with build args
  uses: ./docker-build-push
  with:
    image: 'myapp'
    build-args: |
      NODE_VERSION=18
      ENV=production
      BUILD_DATE=${{ github.event.head_commit.timestamp }}
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}
```

### Build Only (No Push)
```yaml
- name: Build for testing
  uses: ./docker-build-push
  with:
    image: 'myapp'
    push: 'false'
```

### Push to GitHub Container Registry
```yaml
- name: Push to GHCR
  uses: ./docker-build-push
  with:
    image: 'myapp'
    registry: 'ghcr.io'
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
```

### Complete CI/CD Pipeline
```yaml
name: Build and Deploy

on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Docker meta
        id: meta
        run: |
          # Generate tags based on git ref
          if [[ $GITHUB_REF == refs/tags/* ]]; then
            VERSION=${GITHUB_REF#refs/tags/v}
            echo "tags=latest,$VERSION" >> $GITHUB_OUTPUT
          else
            echo "tags=latest,dev" >> $GITHUB_OUTPUT
          fi

      - name: Build and push
        uses: ./docker-build-push
        with:
          image: 'username/myapp'
          tags: ${{ steps.meta.outputs.tags }}
          platforms: 'linux/amd64,linux/arm64'
          build-args: |
            VERSION=${{ steps.meta.outputs.version }}
            COMMIT_SHA=${{ github.sha }}
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
```

## Registry Support

### Docker Hub
```yaml
registry: 'docker.io'  # or omit (default)
username: <docker-username>
password: <docker-password>
```

### GitHub Container Registry (GHCR)
```yaml
registry: 'ghcr.io'
username: ${{ github.actor }}
password: ${{ secrets.GITHUB_TOKEN }}
```

### Google Container Registry (GCR)
```yaml
registry: 'gcr.io'
username: '_json_key'
password: ${{ secrets.GCP_SA_KEY }}
```

### AWS Elastic Container Registry (ECR)
```yaml
# Requires aws-actions/amazon-ecr-login first
registry: '<account-id>.dkr.ecr.<region>.amazonaws.com'
```

## Caching Explained

This action uses **registry cache** for lightning-fast rebuilds:

**First Build:**
```
Layer 1: FROM node:18        ████████ 2m 30s
Layer 2: COPY package.json   ████     45s
Layer 3: RUN npm install     ████████ 3m 15s
Layer 4: COPY src/           ████     30s
Total: 7 minutes
```

**Second Build (nothing changed):**
```
Layer 1: CACHED ✅
Layer 2: CACHED ✅
Layer 3: CACHED ✅
Layer 4: CACHED ✅
Total: 15 seconds
```

**Third Build (only src/ changed):**
```
Layer 1: CACHED ✅
Layer 2: CACHED ✅
Layer 3: CACHED ✅
Layer 4: REBUILDING ████
Total: 45 seconds
```

## Multi-Platform Builds

Build once, run everywhere:
```yaml
platforms: 'linux/amd64,linux/arm64'
```

**Supported platforms:**
- `linux/amd64` - Intel/AMD (most servers, desktops)
- `linux/arm64` - ARM 64-bit (Apple Silicon, AWS Graviton, Raspberry Pi 4)
- `linux/arm/v7` - ARM 32-bit (Raspberry Pi 2/3)
- `linux/386` - 32-bit x86

## Best Practices

**1. Use specific tags for production:**
```yaml
tags: 'latest,${{ github.sha }},v1.0.0'
```

**2. Don't push on pull requests:**
```yaml
push: ${{ github.event_name != 'pull_request' }}
```

**3. Optimize Dockerfile for caching:**
```dockerfile
# Good - dependencies cached separately
COPY package.json package-lock.json ./
RUN npm ci
COPY src/ ./src/

# Bad - changes to src/ invalidate npm install cache
COPY . .
RUN npm ci
```

**4. Use build args for versioning:**
```yaml
build-args: |
  VERSION=${{ github.ref_name }}
  BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ')
```

## Security Tips

- Store credentials in GitHub Secrets
- Use personal access tokens, not passwords
- For GHCR, use built-in `GITHUB_TOKEN`
- Rotate credentials regularly
- Use minimal permissions (read/write packages only)

## Production Alternatives

For production environments, consider official Docker actions:
```yaml
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3

- name: Login to Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}

- name: Build and push
  uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: user/app:latest
```

**Why This Action Still Valuable:**
- All-in-one solution (setup + login + build)
- Customizable: Full control over build process
