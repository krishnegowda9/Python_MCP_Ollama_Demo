# Python MCP Ollama Demo

This project runs an MCP server with `add`, `get_weather`, and `systemInfo`
tools, then connects those tools to a local Ollama model.

## Requirements

- Python 3.10 or newer
- [Ollama for Windows](https://ollama.com/download/windows)
- An Ollama model that supports tool calling (the client defaults to
  `gemma4:e2b`)

## Setup on Windows

Open PowerShell in this directory and run:

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
ollama pull gemma4:e2b
```

If PowerShell blocks virtual-environment activation, use the venv interpreter
directly instead:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
ollama pull gemma4:e2b
```

## Run

Make sure the Ollama app is running, then run the client from this directory:

```powershell
python ollama_mcp_client.py
```

Or, without activating the venv:

```powershell
.\.venv\Scripts\python.exe ollama_mcp_client.py
```

The client starts `server.py` automatically over standard input/output, fetches
its tools, and sends a sample prompt to Ollama. To use a different installed
model, change the `MODEL` value in `ollama_mcp_client.py`.
