# AutoEngineer-CLI

**An Autonomous Software Engineering Multi-Agent System** — Give it a task in plain English, and it writes the code for you.

[![PyPI version](https://badge.fury.io/py/autoengineer-cli.svg)](https://pypi.org/project/autoengineer-cli/)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

AutoEngineer-CLI is a production-grade CLI tool that orchestrates **5 specialized AI agents** to autonomously analyze, architect, implement, test, and review code — all from a single natural language command.

## What It Does

```bash
autoengineer --repo ./my-project --task "Build a JWT authentication API with login, register, and token refresh"
```

AutoEngineer will:

1. **Parse** your requirements into a structured plan
2. **Architect** the solution by scanning your existing codebase
3. **Write actual code files** into your repository
4. **Test** the code in an isolated Podman sandbox
5. **Review** the implementation and generate a quality report

All powered by **free-tier LLMs** via OpenRouter — no paid API keys required.

## Architecture

The system runs a sequential pipeline of 5 specialized agents:

| Agent | Model | Role |
|-------|-------|------|
| Manager | xiaomi/mimo-v2 | Parses requirements, creates task breakdown |
| Architect | xiaomi/mimo-v2 | Scans codebase, creates JSON execution plan |
| Coder | mistralai/devstral | Writes actual files using write_file tool |
| QA | nvidia/nemotron | Tests code in Podman sandbox, reports issues |
| Reviewer | xiaomi/mimo-v2 | Reads and reviews actual files, generates report |

### Agent Tools

| Tool | Description | Used By |
|------|-------------|---------|
| `write_file` | Create or overwrite files in the repo | Coder |
| `read_file` | Read file contents for analysis | Architect, Coder, QA, Reviewer |
| `list_directory` | Explore repository structure | Architect, Coder, QA, Reviewer |
| `podman_sandbox` | Execute code in isolated containers | QA |

## Installation

**Recommended — pipx (isolated install):**

```bash
pipx install autoengineer-cli
```

**Using pip:**

```bash
pip install autoengineer-cli
```

**Using uv:**

```bash
uv tool install autoengineer-cli
```

**From source:**

```bash
git clone https://github.com/abhi-14gyan/AutoEngineer-CLI.git
cd AutoEngineer-CLI
pip install -e .
```

## Setup

### 1. Get a Free OpenRouter API Key

1. Go to [openrouter.ai](https://openrouter.ai/) and create an account
2. Navigate to **Keys** and click **Create Key**
3. Copy your API key

### 2. Set the API Key

**Windows CMD:**

```cmd
set OPENROUTER_API_KEY=sk-or-v1-your-key-here
```

**Windows PowerShell:**

```powershell
$env:OPENROUTER_API_KEY="sk-or-v1-your-key-here"
```

**Linux or macOS:**

```bash
export OPENROUTER_API_KEY="sk-or-v1-your-key-here"
```

**Or create a `.env` file** in your working directory:

```
OPENROUTER_API_KEY=sk-or-v1-your-key-here
```

The tool automatically loads `.env` files via `python-dotenv`.

### 3. Install Podman (Optional)

The QA agent uses Podman to run code in isolated containers. Without Podman, the QA agent will still analyze code but won't execute it.

- **Windows** — [Podman Desktop](https://podman-desktop.io/)
- **macOS** — `brew install podman`
- **Linux** — `sudo apt install podman` or `sudo dnf install podman`

## Usage

### Basic Command

```bash
autoengineer --repo ./my-project --task "Create a REST API with user authentication"
```

### CLI Options

| Flag | Description |
|------|-------------|
| `-r, --repo PATH` | Path to the repository to work on (required) |
| `-t, --task TEXT` | Description of the task to solve (required) |
| `-o, --output PATH` | Save the final report to a file |
| `-v, --verbose` | Enable verbose agent output |
| `--max-retries INT` | Max retry attempts on API errors (default: 3) |
| `--version` | Show version |
| `--help` | Show help message |

### Examples

```bash
# Simple script generation
autoengineer -r ./my-app -t "Create a Python script that calculates Fibonacci numbers"

# Full-stack feature
autoengineer -r ./my-app -t "Add JWT authentication with login, register, and token refresh endpoints"

# Add to existing codebase
autoengineer -r ./existing-project -t "Add logging middleware to all API endpoints"

# Save report to file
autoengineer -r ./my-app -t "Create a CLI calculator" -o report.md

# Verbose mode (see agent reasoning)
autoengineer -r ./my-app -t "Build a todo app backend" -v
```

## Tech Stack

| Component | Technology | Purpose |
|-----------|------------|---------|
| AI Orchestration | [CrewAI](https://crewai.com/) | Multi-agent task pipeline |
| LLM Gateway | [LiteLLM](https://litellm.ai/) + [OpenRouter](https://openrouter.ai/) | Model-agnostic LLM access |
| Models | Xiaomi MiMo-V2, Mistral Devstral, NVIDIA Nemotron | Free-tier with 262k+ context |
| Containerization | [Podman](https://podman.io/) | Rootless sandboxed code execution |
| CLI | [Click](https://click.palletsprojects.com/) | Command-line interface |
| Config | [python-dotenv](https://github.com/theskumar/python-dotenv) | Environment variable management |
| Build | [Hatch](https://hatch.pypa.io/) + [uv](https://docs.astral.sh/uv/) | Packaging and publishing |

## Project Structure

```
AutoEngineer-CLI/
├── src/
│   └── autoengineer_cli/
│       ├── __init__.py            # Package metadata and version
│       ├── main.py                # CLI entry point and task pipeline
│       ├── config.py              # Centralized configuration
│       ├── agents.py              # 5 CrewAI agent definitions
│       └── tools/
│           ├── __init__.py
│           ├── file_writer.py     # FileWriter, FileReader, DirectoryLister
│           └── podman_sandbox.py  # Podman container execution tool
├── main.py                        # Development entry point
├── config.py                      # Development config
├── agents.py                      # Development agents
├── tools/                         # Development tools
├── pyproject.toml                 # Build configuration
├── requirements.txt               # Dependencies
├── .env.example                   # Example environment variables
├── LICENSE                        # MIT License
└── README.md
```

## Security

- **Path Traversal Prevention** — File tools validate all paths stay within the target repository
- **Network Isolation** — Podman containers run with `--network=none`
- **Memory Limits** — Containers capped at 512MB by default
- **Execution Timeouts** — 60-second limit on sandboxed code execution
- **No Secrets in Code** — API keys loaded from environment variables or `.env`

## Configuration

All settings are centralized in `config.py`:

| Setting | Default | Description |
|---------|---------|-------------|
| `MANAGER_MODEL` | `xiaomi/mimo-v2-flash:free` | LLM for task parsing |
| `ARCHITECT_MODEL` | `xiaomi/mimo-v2-flash:free` | LLM for architecture planning |
| `CODER_MODEL` | `mistralai/devstral-2512:free` | LLM for code generation |
| `QA_MODEL` | `nvidia/nemotron-3-nano-30b-a3b:free` | LLM for testing |
| `REVIEWER_MODEL` | `xiaomi/mimo-v2-flash:free` | LLM for code review |
| `AGENT_MAX_ITER` | `25` | Max iterations per agent |
| `AGENT_MAX_RPM` | `10` | Max requests per minute |
| `CONTAINER_TIMEOUT` | `60` | Podman execution timeout (seconds) |
| `CONTAINER_MEMORY_LIMIT` | `512m` | Podman memory limit |

## Development Setup

```bash
# Clone the repo
git clone https://github.com/abhi-14gyan/AutoEngineer-CLI.git
cd AutoEngineer-CLI

# Create virtual environment
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # Linux/macOS

# Install in development mode
pip install -e ".[dev]"

# Set your API key
cp .env.example .env
# Edit .env and add your OPENROUTER_API_KEY

# Run directly
python main.py --repo ./test-project --task "Create a hello world script"
```

### Building and Publishing

```bash
# Build the package
python -m uv build

# Publish to PyPI
python -m uv publish --token YOUR_PYPI_TOKEN
```

## Output

AutoEngineer generates:

1. **Source code files** — Written directly to your repository
2. **Configuration files** — `requirements.txt`, configs, etc.
3. **Final Report** — Comprehensive Markdown report with code quality assessment, security analysis, performance notes, and improvement recommendations

## Known Limitations

- **Free-tier rate limits** — OpenRouter free models may have rate limits during peak usage. The tool retries automatically on transient errors.
- **Podman required for testing** — Without Podman, the QA agent performs static analysis only.
- **Model quality varies** — Free-tier models may occasionally produce incomplete code. Retry or use paid models for better results.

## Contributing

Contributions are welcome!

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Make your changes
4. Run tests: `pytest`
5. Submit a Pull Request

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [CrewAI](https://crewai.com/) — Multi-agent orchestration framework
- [OpenRouter](https://openrouter.ai/) — Free-tier LLM API gateway
- [LiteLLM](https://litellm.ai/) — Universal LLM interface
- [Podman](https://podman.io/) — Rootless container engine

---

**Made with ❤️ by [abhi-14gyan](https://github.com/abhi-14gyan)**
