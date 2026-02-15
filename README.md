# AI Agent (Gemini Function Calling)

A small CLI coding agent that uses Gemini function calling to interact with a local project.

## What It Does

- Accepts a user prompt from the command line
- Lets Gemini choose tool calls
- Executes local file tools in a restricted working directory (`./calculator`)
- Returns tool outputs back to Gemini until a final response is produced

## Project Structure

- `main.py`: CLI entrypoint and model loop
- `call_function.py`: tool registration + dispatcher
- `prompts.py`: system instruction for the model
- `config.py`: runtime configuration
- `functions/`: tool implementations
  - `get_files_info.py`
  - `get_file_content.py`
  - `write_file.py`
  - `run_python_file.py`
- `calculator/`: sandboxed working directory used by tools

## Requirements

- Python 3.13+
- `uv`
- Gemini API key

## Setup

1. Create `.env` in project root:

```env
GEMINI_API_KEY=your_api_key_here
```

2. Install dependencies:

```bash
uv sync
```

## Run

Basic:

```bash
uv run main.py "what files are in the root?"
```

Verbose (shows token usage + function calls):

```bash
uv run main.py "what files are in the root?" --verbose
```

## Available Functions

Registered in `call_function.py` as a Gemini tool:

- `get_files_info(directory=".")`
- `get_file_content(file_path)`
- `write_file(file_path, content)`
- `run_python_file(file_path, args=[])`

`working_directory` is injected automatically from `config.py`:

- `WORKING_DIR = "./calculator"`

## Safety Model

Each function resolves paths with `os.path.abspath`/`os.path.normpath` and blocks path traversal outside `WORKING_DIR`.

Examples of blocked paths:

- absolute paths like `/bin`
- parent traversal like `../...`

## Config

From `config.py`:

- `MAX_CHARS`: max file content read length
- `WORKING_DIR`: base directory allowed for tool operations
- `MAX_ITERS`: max model/tool loop iterations

## Test Helpers

You have script-style checks for each tool:

```bash
uv run test_get_files_info.py
uv run test_get_file_content.py
uv run test_write_file.py
uv run test_run_python_file.py
```

## Known Limitation

Gemini free-tier quotas can return `429 RESOURCE_EXHAUSTED`, which may fail runs even if local tool code is correct. Re-run after the retry window or use a plan with higher limits.
