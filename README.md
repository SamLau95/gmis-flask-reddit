# gmis-freddit

A minimal Flask app managed by uv.

## Requirements

- Python 3.10+
- uv (package manager)

Install uv (macOS/Linux):

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## Setup

Sync dependencies (creates a local virtual env in `.venv`):

```bash
uv sync
```

## Run

Run using the Flask CLI:

```bash
uv run flask --app app run --debug
```

Or run directly with Python:

```bash
uv run python app.py
```

Open http://127.0.0.1:5000/ to see "Hello, World!".


