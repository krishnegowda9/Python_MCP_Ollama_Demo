<div align="center">

# Python MCP + Ollama Demo

### A local AI assistant that can use real tools through the Model Context Protocol

Ask a local Ollama model a question. The model can request a calculation, look
up mock weather, or inspect basic system information. A Python MCP server
executes the requested tool and returns its result to the model.

**Python 3.12 · Ollama · Gemma 4 · Model Context Protocol**

</div>

---

## Contents

- [What this project does](#what-this-project-does)
- [Architecture and workflow](#architecture-and-workflow)
- [Tech stack](#tech-stack)
- [Available tools](#available-tools)
- [Example session](#example-session)
- [Run the demo on Windows](#run-the-demo-on-windows)
- [Add a new tool](#add-a-new-tool)
- [Why MCP and Ollama?](#why-mcp-and-ollama)
- [Project files](#project-files)
- [Troubleshooting](#troubleshooting)

## What this project does

This is a small end-to-end demo of **tool calling with a local large language
model (LLM)**:

1. The Python client starts the MCP server as a subprocess.
2. The client discovers the tools the server offers.
3. It gives the model the user's prompt and those tool definitions.
4. Ollama's model chooses whether to request a tool.
5. The client forwards each requested tool call to the MCP server.
6. The server runs the Python tool logic and returns the result.
7. The client sends the result back to the model, which writes a final reply.

The demo prompt is defined at the bottom of `ollama_mcp_client.py`. You can
edit it to try other requests.

## Architecture and workflow

```text
┌───────────────────┐        MCP over stdio        ┌──────────────────┐
│ ollama_mcp_client │ ◄──────────────────────────► │    server.py     │
│                   │  discover tools / call tools │                  │
└─────────┬─────────┘                               └────────┬─────────┘
          │                                                  │
          │ Ollama Python client                            │ Python logic
          │ prompt + tool schemas                            ├─ add
          ▼                                                  ├─ get_weather
┌───────────────────┐      tool result / final reply         └─ systemInfo
│ Ollama model      │ ◄───────────────────────────────────────────────
│ gemma4:e2b        │
│ running locally   │
└───────────────────┘
```

The MCP server advertises a tool name, description, and JSON input schema.
`ollama_mcp_client.py` translates that schema into Ollama's function-tool
format. The model returns a proposed function name and arguments; the client
then calls the matching MCP tool and returns its text result to the model.

**The model does not execute Python code.** The client routes tool requests and
the MCP server owns and executes the tool implementations.

## Tech stack

| Technology | Role |
| --- | --- |
| Python 3.12 | Runs the client and MCP server |
| [Ollama](https://ollama.com/) | Runs and serves the local language model |
| `gemma4:e2b` | Default model configured in `ollama_mcp_client.py` |
| [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk) | Exposes and calls tools over standard input/output (stdio) |
| `ollama` Python package | Sends prompts and tool definitions to Ollama |
| `psutil` | Reads total system memory for the `systemInfo` tool |

`fastapi` and `uvicorn` are also listed in `requirements.txt`, but the current
demo does not start an HTTP API with them.

## Available tools

| Tool | Input | What it does | Example result |
| --- | --- | --- | --- |
| `add` | `a` and `b` (numbers) | Adds two numbers | `100` for `42 + 58` |
| `get_weather` | `city` (string) | Looks up a city in an in-code mock weather table | `Cloudy, 15°C` for `London` |
| `systemInfo` | No required input | Returns the platform and total physical memory as JSON | `{"platform": "win32", "memory": "15.70 GB"}` |

Weather is **mock data**, not a live weather service. The `systemInfo` tool
reports the computer running the MCP server.

## Example session

The current client sends this sample prompt:

> What is 42 + 58? Also, what's the weather like in London? and Also return
> current system details

An illustrative run looks like this (the model's exact wording and tool-call
order can vary):

```text
[MCP] Tools available: ['add', 'get_weather', 'systemInfo']

User: What is 42 + 58? Also, what's the weather like in London? and Also return current system details
[Ollama] Calling tool: add({'a': 42, 'b': 58})
[MCP]    Result: 100
[Ollama] Calling tool: get_weather({'city': 'London'})
[MCP]    Result: Cloudy, 15°C
[Ollama] Calling tool: systemInfo({})
[MCP]    Result: {"platform": "win32", "memory": "15.70 GB"}

Assistant: 42 + 58 is 100. London is cloudy at 15°C. Your system is running
Windows and has 15.70 GB of memory.
```

The platform and memory in your output depend on the computer running the
server.

## Run the demo on Windows

### 1. Install Ollama and download the model

Install [Ollama for Windows](https://ollama.com/download/windows). When
installation finishes, open a **new PowerShell window** and check that the
command is available:

```powershell
ollama --version
```

Open the Ollama app from the Start menu and download the model:

```powershell
ollama pull gemma4:e2b
```

### 2. Set up the Python environment

In PowerShell, go to the project folder and create a virtual environment:

```powershell
cd "C:\Users\krish\OneDrive\Desktop\Python-mcp-ollama\Python-MCP-Demo"
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

If script activation is blocked by PowerShell, skip activation and install
packages with the environment's Python:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

### 3. Start the demo

Ensure the Ollama app is running, then, from the project folder:

```powershell
python ollama_mcp_client.py
```

Without activating the virtual environment, run:

```powershell
.\.venv\Scripts\python.exe ollama_mcp_client.py
```

The client starts `server.py` automatically; **you do not need a second
terminal for the MCP server**.

## Add a new tool

Add each tool in two places in `server.py`:

1. Register its name, description, and JSON input schema in `list_tools()`.
2. Implement its behavior in `call_tool()`, returning MCP text content.

For example, to add a `multiply` tool:

### 1. Advertise the tool

Add this entry to the list returned by `list_tools()`:

```python
types.Tool(
    name="multiply",
    description="Multiply two numbers",
    inputSchema={
        "type": "object",
        "properties": {
            "a": {"type": "number", "description": "First number"},
            "b": {"type": "number", "description": "Second number"},
        },
        "required": ["a", "b"],
    },
),
```

### 2. Implement its logic

Add a branch to `call_tool()` before the `else` for unknown tools:

```python
elif name == "multiply":
    result = arguments["a"] * arguments["b"]
    return [types.TextContent(type="text", text=str(result))]
```

Restart `ollama_mcp_client.py` after editing. The client discovers the server's
tool list each time it starts, so **you do not need to change the client** for
this tool. Try a prompt such as:

> Use the multiply tool to calculate 6 times 7.

The tool name and description help the model decide when the tool is
appropriate; the input schema describes the arguments the client should send.

## Why MCP and Ollama?

### Why MCP?

The **Model Context Protocol (MCP)** gives tools a consistent way to describe
their names, purposes, and inputs, and a standard way for clients to discover
and call them. In this demo, tool definitions and implementations live on the
MCP server; the client adapts those definitions for the model. That separation
makes it easier to add tools without embedding each implementation in the
client.

### Why Ollama?

Ollama makes it straightforward to run a supported model on your own computer
and provides a local API for the Python client. This demo uses Ollama instead
of a hosted model API, so its model requests go to the local Ollama service.
The model and its download still require disk space and local compute.

## Project files

```text
Python-MCP-Demo/
├── server.py             # MCP server, tool schemas, and tool implementations
├── ollama_mcp_client.py  # Connects Ollama to the MCP server
├── requirements.txt      # Python package dependencies
├── .gitignore            # Ignores virtual environments and local artifacts
└── README.md             # Project guide
```

## Troubleshooting

| Problem | What to check |
| --- | --- |
| `ollama` is not recognized | Install Ollama for Windows, wait for installation to finish, then open a new PowerShell window and try `ollama --version`. |
| `Cannot connect to Ollama` | Start the Ollama app and retry. The Ollama service must be reachable before starting the client. |
| Model not found | Run `ollama pull gemma4:e2b`, or update `MODEL` in `ollama_mcp_client.py` to a model you have downloaded. |
| A tool does not appear | Add its schema to `list_tools()` in `server.py`, then restart the client. |
| Tool call says `Unknown tool` | Make sure the same tool name appears in both `list_tools()` and `call_tool()`. |
| A model does not call a tool | Use a model that supports tool calling and ask explicitly for the relevant operation. |
