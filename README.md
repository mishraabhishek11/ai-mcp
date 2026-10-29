# ai-mcp

A command-line assistant that discovers tools from an [MCP](https://modelcontextprotocol.io) server (built with FastMCP) and uses any OpenAI-compatible API to pick and call the right one.

## How it works

1. `agent.py` starts the MCP server (`mcp_tools/server.py`) and connects to it as a client.
2. It discovers the tools the server exposes.
3. **Planner:** a model call receives the user's request plus every tool's name and description, and replies with the name of one tool.
4. The selected tool is executed on the MCP server.
5. A second model call turns the raw tool result into a short, conversational answer.

The planner is prompt-based routing, not native function calling, so each request costs two API calls.

## Tools

Defined in `mcp_tools/server.py`:

| Tool                | Description                                               |
| ------------------- | --------------------------------------------------------- |
| `current_time`      | Returns the current date and time (`dd-mm-yyyy HH:MM:SS`) |
| `roll_dice`         | Rolls a six-sided die                                     |
| `generate_password` | Generates a secure random password (default length 12)    |

To add a tool, decorate a function with `@mcp.tool()` in `server.py`. The agent discovers it automatically.

## Project structure

```
ai-mcp/
├── agent.py            # Planner, response generation, main loop
├── mcp_tools/
│   ├── client.py       # connect / disconnect / discover_tools / execute_tool
│   └── server.py       # FastMCP server and tool definitions
├── .env                # Configuration (not committed)
└── README.md
```

> The package is named `mcp_tools`, not `mcp`, on purpose: a local `mcp` folder shadows the official `mcp` SDK that `fastmcp` depends on and breaks imports.

## Setup

Requires Python 3.10+.

```bash
python -m venv .venv
.venv\Scripts\activate        # Windows
# source .venv/bin/activate   # macOS / Linux

pip install fastmcp openai python-dotenv
```

Create a `.env` file in the project root:

```
BASE_URL=http://localhost:11434/v1
API_KEY=ollama
MODEL=your-model-name
```

`BASE_URL` can point at any OpenAI-compatible endpoint (Ollama, OpenAI, etc.).

## Run

```bash
python agent.py
```

Example:

```
Available Tools
----------------
current_time
roll_dice
generate_password

User : roll a dice
Planner Selected: roll_dice

You rolled a 4.
```

## Known limitations

- Tool selection fails if the model replies with anything other than an exact tool name.
- Tools are called without arguments, so `generate_password` always uses its default length.
- The agent handles a single request, then exits.
- API and connection errors are not caught.
- There is no `requirements.txt` and no tests.

## Planned work

- Native function calling with tool arguments
- Error handling and retries
- Interactive chat loop with conversation history
- `requirements.txt` and tests
