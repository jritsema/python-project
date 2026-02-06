# python-project

Opinionated guidance for scaffolding out Python projects.

This is essentially a markdown file packaged for various AI tools including:

- [Agent Skill](https://agentskills.io/home)

- [Kiro Power](https://kiro.dev/powers/)

## What is it?

Tell your AI agent:

```
setup a new python project
```

The agent will:

1. **Ask you questions** about your project (name, description, Docker support, AWS SDK)
2. **Generate all project files** including `pyproject.toml`, `Makefile`, `main.py`, `.gitignore`, logging utilities, and optional `Dockerfile` and `.dockerignore`
3. **Initialize everything** by running `uv python install`, creating a virtual environment, syncing dependencies, and creating a git repository
4. **Give you next steps** to start developing immediately

You go from zero to a fully configured, production-ready Python project in under a minute.

## Key Benefits

**Instant Project Setup**

- Answer 4 simple questions, get a complete Python project structure
- No manual file creation or configuration
- Consistent project layout across all your Python projects

**Modern Tooling with `uv`**

- 10-100x faster than pip for package installation
- Automatic lockfile (`uv.lock`) for reproducible builds
- True dependency resolution (prevents conflicts)
- Python version management built-in

**Production-Ready**

- Git repository initialized automatically
- Proper logging utilities included
- Optimized multi-stage Dockerfile (if requested)

**Simple Make Interface**

- Consistent workflow across all projects
- Easy onboarding for team members
- No need to remember `uv` commands
- `make start` automatically loads environment variables from `.env` if present
- container builds leverage python version in `pyproject.toml` (one place to update)

## How To Work With The Project

```bash
make init              # Initialize project (Python, venv, dependencies)
make install           # Install all dependencies from lockfile
make install <package> # Add a new package
make start             # Run the project (loads .env if present)
make build             # Build container image with correct Python version from pyproject.toml
```

## Usage

You can use the https://skills.sh CLI tool to install it into a variety of agentic tools.

```sh
# npx skills add jritsema/python-project
npx skills add git@github.com:jritsema/python-project.git
```

For Kiro IDE, you can install it as a power. Go to the Kiro Powers tab and click install. Use the following URL:

```
https://github.com/jritsema/python-project/tree/main/power
```
