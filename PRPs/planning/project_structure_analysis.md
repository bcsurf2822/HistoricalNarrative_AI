# Project Structure Analysis - Historical Narrative AI

## Current Directory Structure

```
HistoricalNarrative_AI/
├── .claude/                    # Claude Code configurations and commands
│   └── commands/               # Custom commands for Claude Code
├── .git/                       # Git version control
├── .gitignore                  # Git ignore patterns (.env only)
├── README.md                   # Basic project readme
├── agent/                      # Main agent implementations directory
│   ├── .env                    # Environment variables (API keys)
│   ├── .venv/                  # Python virtual environment
│   ├── __pycache__/            # Python bytecode cache
│   ├── brave_search_agent.py   # Implemented search agent with tool decorator
│   ├── client.py               # Basic Bedrock client example
│   ├── constants.py            # Shared constants (SESSION_ID generation)
│   ├── interactive_history_agent.py  # Empty placeholder file
│   ├── requirements.txt        # Python dependencies
│   └── video_generation_agent.py     # Empty placeholder file
├── hackathon_resources/        # Hackathon-specific resources
├── PRPs/                       # Project Requirements and Planning
│   ├── INITIAL.md              # Initial project documentation
│   ├── ai_docs/                # AI-generated documentation
│   ├── planning/               # Planning documents directory
│   └── templates/              # Document templates
└── resources/                  # Design and planning resources
    ├── design/                 # Architecture and design documents
    └── planning/               # Planning flow documents
```

## Code Organization Patterns

### 1. Agent Implementation Pattern

Based on `brave_search_agent.py`, the established pattern for agents includes:

```python
# Standard imports and environment setup
import os
from dotenv import load_dotenv
from mcp import StdioServerParameters, stdio_client
from strands import Agent, tool
from strands.models import BedrockModel
from strands.tools.mcp import MCPClient
from constants import SESSION_ID

# Environment variable loading
load_dotenv()
os.environ["STRANDS_TOOL_CONSOLE_MODE"] = "enabled"

# Tool decorator for agent exposure
@tool
def agent_name(parameter: str) -> str:
    """
    Agent description for tool usage
    Args:
        parameter: Parameter description
    Returns:
        Output description
    """
    response = agent(parameter)
    return response

# MCP client initialization pattern
try:
    mcp_client = MCPClient(
        lambda: stdio_client(
            StdioServerParameters(
                command="docker",
                args=["run", "-i", "--rm", "-e", f"API_KEY={api_key}", "service_name"]
            )
        )
    )
except Exception as e:
    raise Exception(f"Failed to initialize MCP Client: {str(e)}")

# Agent initialization with system prompt
agent = Agent(
    model=BedrockModel(model_id="us.anthropic.claude-3-7-sonnet-20250219-v1:0"),
    system_prompt=system_prompt,
    tools=tools,
    trace_attributes={"session.id": SESSION_ID}
)
```

### 2. Import and Dependency Patterns

- **Relative imports**: Use `from constants import SESSION_ID` for shared utilities
- **External libraries**: strands-agents, strands-agents-tools, mcp, uv
- **AWS services**: boto3 for direct AWS service access
- **Environment management**: python-dotenv for configuration

### 3. Configuration Management

- **Environment variables**: Stored in `/agent/.env`
- **Shared constants**: Centralized in `/agent/constants.py`
- **API keys**: Environment-based with validation
- **Session tracking**: UUID-based session IDs for tracing

## Development Environment Setup

### Virtual Environment
- **Location**: `/agent/.venv/`
- **Python version**: 3.11
- **Management**: Standard venv (not conda/pipenv)

### Dependencies (`requirements.txt`)
```
uv                      # Fast Python package installer
strands-agents          # Agent framework
strands-agents-tools    # Agent tooling
mcp                     # Model Context Protocol
```

### Environment Variables Pattern
```bash
# API Keys for external services
BRAVE_API_KEY=<api_key>
API_KEY=<bedrock_api_key>
ACCESS_KEY=<aws_access_key>
Secre_acess=<aws_secret_key>  # Note: typo in original
```

## Testing and Documentation Standards

### Current State
- **Testing**: No visible test patterns or test directories
- **Documentation**: Heavy use of markdown in `/resources/` and `/PRPs/`
- **Code documentation**: Docstrings following Google style in agent functions

### Logging Patterns
Based on user instructions, logging should follow:
```python
print(f"[FILENAME-FUNCTION] description and log of what is happening")
```

## Integration Patterns Between Agents

### Tool Exposure Pattern
Agents are made available as tools to other agents using the `@tool` decorator:

```python
@tool
def brave_search_agent(query: str) -> str:
    """Search assistant agent for handling general queries"""
    response = agent(query)
    return response
```

### Agent Communication
- **Session tracking**: Shared SESSION_ID from constants.py
- **Tool calling**: Agents can call other agents as tools
- **MCP integration**: Docker-based MCP clients for external services

## Video Generation Agent Integration Plan

### 1. File Placement
Create the video generation agent at:
```
/Users/benjamincorbett/code/hackathons/HistoricalNarrative_AI/agent/video_generation_agent.py
```

### 2. Following Established Patterns

#### Required Imports
```python
import os
from dotenv import load_dotenv
from strands import Agent, tool
from strands.models import BedrockModel
from constants import SESSION_ID
```

#### Tool Decorator
```python
@tool
def video_generation_agent(prompt: str, style: str = "historical") -> str:
    """
    Video generation agent for creating historical narrative videos
    Args:
        prompt: Description of the video content to generate
        style: Video style (historical, documentary, etc.)
    Returns:
        Video generation status and metadata
    """
```

#### Environment Variables
Add to `/agent/.env`:
```bash
# Video generation service API key
VIDEO_API_KEY=<api_key>
# Or relevant video generation service credentials
```

#### Dependencies
Add to `/agent/requirements.txt`:
```
# Video generation dependencies (examples)
openai              # If using OpenAI video generation
requests            # For API calls
pillow              # For image processing
```

### 3. Integration with Existing Agents

The video generation agent should:
- Follow the same MCP client pattern if using external services
- Use the shared SESSION_ID for tracing
- Be callable from other agents using the @tool decorator
- Include proper error handling and logging

### 4. Constants and Configuration

Add video-specific constants to `/agent/constants.py`:
```python
# Video generation settings
DEFAULT_VIDEO_DURATION = 30  # seconds
SUPPORTED_VIDEO_FORMATS = ["mp4", "mov", "avi"]
MAX_VIDEO_LENGTH = 300  # seconds
```

## Development Workflow Recommendations

### 1. Environment Setup
```bash
cd /Users/benjamincorbett/code/hackathons/HistoricalNarrative_AI/agent
source .venv/bin/activate
pip install -r requirements.txt
```

### 2. Testing Pattern
- Create manual testing in `__main__` block following brave_search_agent.py pattern
- Add interactive console interface for development
- Include error handling and user feedback

### 3. Documentation
- Add comprehensive docstrings following existing pattern
- Document in `/PRPs/ai_docs/` following project structure
- Update architecture documentation in `/resources/design/`

## Security and Best Practices

### 1. Environment Variables
- Keep `.env` file in gitignore (already configured)
- Validate required environment variables on startup
- Use meaningful error messages for missing configuration

### 2. Error Handling
```python
try:
    # Agent initialization
except Exception as e:
    raise Exception(f"Failed to initialize Video Generation Agent: {str(e)}")
```

### 3. Logging
Follow the established pattern:
```python
print(f"[video_generation_agent-generate_video] Starting video generation for prompt: {prompt}")
```

## Conclusion

The project follows a clean, modular architecture with clear separation of concerns. The video generation agent should integrate seamlessly by following the established patterns for:

- Agent implementation with @tool decorator
- MCP client integration for external services
- Shared constants and session tracking
- Environment-based configuration
- Error handling and logging standards

The `/agent/` directory is the appropriate location for all agent implementations, with supporting configuration in `constants.py` and `requirements.txt`.