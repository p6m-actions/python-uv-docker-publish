# Python UV Docker Publish

![Latest Release](https://img.shields.io/github/v/release/p6m-actions/python-uv-docker-publish?style=flat-square&label=Latest%20Release&color=blue)

## Description

A GitHub Action that builds and publishes Python applications using UV as Docker containers. This action simplifies the process of containerizing Python UV projects and pushing them to container registries.

## Usage

```yaml
- name: Build and publish Docker image
  uses: p6m-actions/python-uv-docker-publish@v1
  with:
    image-name: my-python-app
    image-tag: ${{ github.sha }}
    registry: ghcr.io
    push: true
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `dockerfile-path` | Path to the Dockerfile | No | `Dockerfile` |
| `context-path` | Build context path | No | `.` |
| `image-name` | Docker image name | Yes | - |
| `image-tag` | Docker image tag | No | `latest` |
| `registry` | Container registry URL | No | - |
| `push` | Whether to push the image to registry | No | `true` |
| `build-args` | Build arguments for Docker build (multiline string) | No | - |
| `platforms` | Target platforms for multi-platform build | No | `linux/amd64` |
| `python-version` | Python version to use in the Docker image | No | `3.12` |
| `uv-version` | UV version to use for dependency management | No | `latest` |

## Outputs

| Output | Description |
|--------|-------------|
| `image-digest` | The digest of the built image |
| `image-metadata` | Build result metadata |
| `image-uri` | Full image URI with registry and tag |

## Examples

### Basic Usage

```yaml
- name: Build and publish to Docker Hub
  uses: p6m-actions/python-uv-docker-publish@v1
  with:
    image-name: myapp
    image-tag: v1.0.0
```

### With Registry

```yaml
- name: Build and publish to GitHub Container Registry
  uses: p6m-actions/python-uv-docker-publish@v1
  with:
    image-name: myapp
    image-tag: ${{ github.sha }}
    registry: ghcr.io/${{ github.repository_owner }}
```

### With Custom Python and UV Versions

```yaml
- name: Build with specific Python and UV versions
  uses: p6m-actions/python-uv-docker-publish@v1
  with:
    image-name: myapp
    image-tag: latest
    python-version: "3.11"
    uv-version: "0.4.0"
```

### With Build Arguments

```yaml
- name: Build with custom build args
  uses: p6m-actions/python-uv-docker-publish@v1
  with:
    image-name: myapp
    image-tag: latest
    build-args: |
      PYTHON_VERSION=3.12
      UV_VERSION=latest
      APP_ENV=production
```

### Multi-platform Build

```yaml
- name: Build for multiple platforms
  uses: p6m-actions/python-uv-docker-publish@v1
  with:
    image-name: myapp
    image-tag: latest
    platforms: linux/amd64,linux/arm64
```

### Complete Workflow Example

```yaml
name: Build and Deploy Python UV App

on:
  push:
    branches: [main]

jobs:
  build-and-publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup UV
        uses: p6m-actions/python-uv-setup@v1
      
      - name: Login to Container Registry
        uses: p6m-actions/python-uv-repository-login@v1
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and publish Docker image
        uses: p6m-actions/python-uv-docker-publish@v1
        with:
          image-name: myapp
          image-tag: ${{ github.sha }}
          registry: ghcr.io/${{ github.repository_owner }}
          python-version: "3.12"
          uv-version: "latest"
```

## Integration with Other UV Actions

This action works seamlessly with other Python UV and Docker GitHub Actions:

- [`python-uv-setup`](https://github.com/p6m-actions/python-uv-setup) - Set up UV in your workflow
- [`python-uv-build`](https://github.com/p6m-actions/python-uv-build) - Build Python packages with UV
- [`python-uv-repository-login`](https://github.com/p6m-actions/python-uv-repository-login) - Login to container registries
- [`python-uv-repository-publish`](https://github.com/p6m-actions/python-uv-repository-publish) - Publish packages to PyPI
- [`python-uv-cut-tag`](https://github.com/p6m-actions/python-uv-cut-tag) - Create release tags
- [`docker-buildx-setup`](https://github.com/p6m-actions/docker-buildx-setup) - Set up Docker Buildx for multi-platform builds

## Dockerfile Requirements

Your Dockerfile should be compatible with UV. Here's a basic example:

```dockerfile
FROM python:3.12-slim

# Install UV
RUN pip install uv

# Set working directory
WORKDIR /app

# Copy UV configuration files
COPY pyproject.toml uv.lock ./

# Install dependencies using UV
RUN uv sync --frozen

# Copy application code
COPY . .

# Expose port
EXPOSE 8000

# Run the application
CMD ["uv", "run", "python", "-m", "your_app"]
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.