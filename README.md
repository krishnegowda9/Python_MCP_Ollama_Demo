# Python MCP Ollama Demo

This project runs an MCP server with `add`, `get_weather`, and `systemInfo`
tools, then connects those tools to a local Ollama model.

## Requirements

- Python 3.10 or newer
- [Ollama for Windows](https://ollama.com/download/windows)
- An Ollama model that supports tool calling (the client defaults to
  `gemma4:e2b`)

## Setup on Windows

1. Install [Ollama for Windows](https://ollama.com/download/windows).
2. After setup finishes, open a **new** PowerShell window so the Ollama
   command is available, then check it:

```powershell
ollama --version
```

If PowerShell says `ollama` is not recognized, Ollama is not installed or the
terminal was opened before installation completed. Finish the installation
and open a new PowerShell window.

3. Open the Ollama app from the Start menu and wait for it to start. In
   PowerShell, download the model:

```powershell
ollama pull gemma4:e2b
```

4. Set up the Python environment from this project directory:

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

If PowerShell blocks virtual-environment activation, use the venv interpreter
directly instead:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
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

If the client reports that Ollama cannot be reached, make sure the Ollama app
is running before starting the client.
