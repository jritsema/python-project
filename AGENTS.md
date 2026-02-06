---
name: python-project
description: Opinionated setup of a Python project from scratch.
displayName: Python Project
keywords: ["setup", "python", "project", "scaffold", "uv"]
author: John Ritsema
license: MIT-0
metadata:
  author: John Ritsema
  version: "0.1"
---

# Instructions

You are an AI agent that scaffolds Python projects using modern tooling with `uv`.

## Step 1: Gather Requirements

Ask the user the following questions:

1. **Project name**: What is the friendly name of your project? Default to the name of the current directory
2. **Project description**: Brief description of your project
3. **Container support**: Do you want container support? (yes/no)
4. **AWS SDK**: Do you need the AWS SDK (boto3)? (yes/no)

## Step 2: Create Project Structure

Based on the user's answers, create the following files in the current directory:

### pyproject.toml

```toml
[project]
name = "<project-slug>"
version = "0.1.0"
description = "<project-description>"
requires-python = ">=3.13"
dependencies = []

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["."]
```

If AWS SDK is needed, add `"boto3"` to the dependencies array.

### main.py

```python
import logging
from log import info

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
)


def main():
    logging.info("<project-name>")
    info({"project_name": "<project-name>"})


if __name__ == "__main__":
    main()
```

### log.py

```python
import logging
import json


def debug(obj):
    if logging.getLogger().level == logging.DEBUG:
        print(json.dumps(obj, indent=2, default=str))


def info(obj):
    if logging.getLogger().level <= logging.INFO:
        print(json.dumps(obj, indent=2, default=str))


def warn(obj):
    if logging.getLogger().level <= logging.WARNING:
        print(json.dumps(obj, indent=2, default=str))


def error(obj):
    print(json.dumps(obj, indent=2, default=str))
```

### Makefile

```makefile
all: help

.PHONY: help
help: Makefile
	@echo
	@echo " Choose a make command to run"
	@echo
	@sed -n 's/^##//p' $< | column -t -s ':' |  sed -e 's/^/ /'
	@echo

## init: initialize a new python project
.PHONY: init
init:
	uv python install
	uv venv
	uv sync

## install: add a new package (make install <package>), or install all project dependencies (make install)
.PHONY: install
install:
	@if [ -z "$(filter-out install,$(MAKECMDGOALS))" ]; then \
		echo "Installing dependencies"; \
		uv sync; \
	else \
		pkg="$(filter-out install,$(MAKECMDGOALS))"; \
		echo "Adding package $$pkg"; \
		uv add $$pkg; \
	fi

## start: run local project
.PHONY: start
start:
	clear
	@echo ""
	@if [ -f .env ]; then set -a && source .env && set +a && uv run python -u main.py; else uv run python -u main.py; fi

## build: build container image
.PHONY: build
build:
	@PYTHON_VERSION=$$(grep 'requires-python' pyproject.toml | sed -E 's/.*">=([0-9]+\.[0-9]+).*/\1/'); \
	PROJECT_NAME=$$(grep '^name = ' pyproject.toml | head -1 | sed 's/name = "\(.*\)"/\1/'); \
	echo "Building $$PROJECT_NAME with Python $$PYTHON_VERSION"; \
	docker build --build-arg PYTHON_VERSION=$$PYTHON_VERSION -t $$PROJECT_NAME .

%:
	@:
```

### .gitignore

```
.venv
.vscode
*.swp
__pycache__
.env
```

### .env

```
# Environment variables
# Copy this file and set your values
# Example:
# DATABASE_URL=postgresql://localhost/mydb
# API_KEY=your-api-key-here
```

### README.md

```markdown
# <project-name>

<project-description>

## Development

Initialize the project:
```
make init
```

Add dependencies:
```
make install <package-name>
```

Run the project:
```
make start
```
```

### AGENTS.md

```markdown
# <project-name>

This is a Python project using `uv` for dependency management.

## Project Structure

- `main.py` - Main entry point
- `log.py` - Logging utilities (debug, info, warn, error functions)
- `pyproject.toml` - Project metadata and dependencies
- `uv.lock` - Locked dependency versions
- `Makefile` - Development commands
- `.env` - Environment variables (not committed to git)

## Adding Dependencies

To add a new package:
```bash
make install <package-name>
```

This will:
1. Add the package to `pyproject.toml`
2. Update `uv.lock` with resolved versions
3. Install the package in the virtual environment

## Running the Project

```bash
make start
```

This automatically:
- Loads environment variables from `.env` if present
- Runs the project in the virtual environment

## Environment Variables

Add environment variables to `.env`:
```
DATABASE_URL=postgres://localhost/mydb
API_KEY=your-api-key-here
```

These are automatically loaded when running `make start`.

## Available Commands

- `make init` - Initialize project (first time setup)
- `make install` - Install all dependencies
- `make install <package>` - Add a new package
- `make start` - Run the project
- `make build` - Build container image (if Docker support enabled)

## Python Version

This project requires Python <requires-python-version> or higher (see `pyproject.toml`).

### Dockerfile (if container support requested)

```dockerfile
# Python version is extracted from pyproject.toml by `make build`
ARG PYTHON_VERSION=3.13

FROM python:${PYTHON_VERSION}-slim AS builder
WORKDIR /app

# Copy uv from official image
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

# Enable bytecode compilation for faster startup
ENV UV_COMPILE_BYTECODE=1

# Copy dependency files
COPY pyproject.toml uv.lock ./

# Install dependencies (not the project itself yet)
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-install-project --no-dev

# Copy the rest of the project
COPY . .

# Install the project
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-dev

FROM python:${PYTHON_VERSION}-slim
WORKDIR /app

# Copy the virtual environment from builder
COPY --from=builder /app/.venv /app/.venv

# Copy the application code
COPY . .

# Create non-root user
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

# Add venv to PATH
ENV PATH="/app/.venv/bin:$PATH"

ENTRYPOINT ["python", "-u", "main.py"]
```

### .dockerignore (if container support requested)

```
.git
.venv
__pycache__
```

## Step 3: Initialize Project

After creating the files, run:

```bash
uv python install
uv venv
uv sync
git init && git add . && git commit -m 'initial'
```

## Step 4: Provide Next Steps

Tell the user:

```
Project created successfully!

Next steps:
- Add dependencies: make install <package-name>
- Run the project: make start
```

## Notes

- The project slug should be the project name in lowercase with spaces replaced by hyphens
- `uv` handles Python version management, virtual environments, and dependency management
- Dependencies are declared in `pyproject.toml` and locked in `uv.lock`
- Use `uv add` to add dependencies (updates both pyproject.toml and uv.lock)
- Use `uv sync` to install dependencies from the lockfile
- Use `uv run` to run commands in the virtual environment
