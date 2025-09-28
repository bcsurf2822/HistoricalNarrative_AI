# Strands Agent Implementation Patterns

This document analyzes the existing Strands agent implementation patterns based on the `brave_search_agent.py` reference implementation and provides definitive guidelines for implementing new agents in the codebase.

## File Structure and Organization

### Primary Files
- **Main Agent**: `agent/{agent_name}_agent.py` - The primary implementation
- **Constants**: `agent/constants.py` - Shared constants and session management
- **Client**: `agent/client.py` - Basic Bedrock client example
- **Requirements**: `agent/requirements.txt` - Dependencies specification

### Dependencies Structure
```
uv
strands-agents
strands-agents-tools
mcp
```

## 1. Import Patterns

### Standard Import Structure
```python
import os
from dotenv import load_dotenv
from mcp import StdioServerParameters, stdio_client
from strands import Agent, tool
from strands.models import BedrockModel
from strands.tools.mcp import MCPClient
from constants import SESSION_ID
```

### Import Pattern Analysis
- **System imports first**: `os` for environment handling
- **Third-party imports**: `dotenv` for environment loading, `mcp` for MCP server communication
- **Strands framework**: Core `Agent` and `tool` decorator, `BedrockModel` for Claude integration
- **MCP integration**: `MCPClient` for external service communication
- **Local imports last**: `SESSION_ID` from constants for session tracking

## 2. Environment Configuration Patterns

### Environment Setup
```python
load_dotenv()
os.environ["STRANDS_TOOL_CONSOLE_MODE"] = "enabled"
```

### Environment Variable Validation
```python
BRAVE_API_KEY = os.getenv("BRAVE_API_KEY")

if not BRAVE_API_KEY:
    raise ValueError("BRAVE_API_KEY environment variable is required")
```

**Pattern Requirements:**
- Always call `load_dotenv()` at module level
- Enable console mode for tool debugging
- Validate required environment variables early with descriptive error messages
- Use `ValueError` with clear error messages for missing required variables

## 3. Tool Decorator Pattern

### Function Tool Definition
```python
@tool
def brave_search_agent(query: str) -> str:
    """
    Search assistant agent for handling general queries
    Args:
        query: A request to the search assistant

    Returns:
        Output from interaction
    """
    response = agent(query)
    print("\n\n")
    return response
```

**Pattern Requirements:**
- Use `@tool` decorator for agent functions
- Clear, descriptive docstring with Args and Returns sections
- Type hints for parameters and return values
- Simple delegation to agent instance
- Include spacing prints for CLI readability
- Return string responses

## 4. MCP Client Integration Pattern

### MCP Client Initialization
```python
try:
    # Initialize Brave MCP server
    brave_mcp_server = MCPClient(
        lambda: stdio_client(
            StdioServerParameters(
                command="docker",
                args=[
                    "run",
                    "-i",
                    "--rm",
                    "-e",
                    f"BRAVE_API_KEY={BRAVE_API_KEY}",
                    "mcp/brave-search",
                ],
            )
        )
    )
except Exception as e:
    raise Exception(f"Failed to initialize MCP Client: {str(e)}")
```

**Pattern Requirements:**
- Wrap MCP client initialization in try-catch blocks
- Use lambda functions for stdio_client creation
- Pass environment variables through Docker `-e` flags
- Use descriptive error messages with original exception details
- Follow Docker best practices (`-i`, `--rm` flags)

### MCP Server Connection Management
```python
# Initialize the MCP server connection once and reuse it
brave_mcp_server.__enter__()

try:
    # Agent initialization here
    pass
except Exception as e:
    brave_mcp_server.__exit__(None, None, None)
    raise e
```

**Pattern Requirements:**
- Manually manage context manager lifecycle
- Call `__enter__()` once at module level
- Ensure `__exit__()` is called on errors
- Reuse connection across agent calls

## 5. Agent Initialization Pattern

### BedrockModel Configuration
```python
model = BedrockModel(
    model_id="us.anthropic.claude-3-7-sonnet-20250219-v1:0",
)
```

### Agent Setup
```python
# Get available tools from MCP server
tools = brave_mcp_server.list_tools_sync()

agent = Agent(
    model=model,
    system_prompt=system_prompt,
    tools=tools,
    trace_attributes={"session.id": SESSION_ID},
)
```

**Pattern Requirements:**
- Use latest Claude Sonnet model ID from Bedrock
- Retrieve tools dynamically from MCP server using `list_tools_sync()`
- Include session tracking with `SESSION_ID` from constants
- Wrap initialization in try-catch with proper cleanup

## 6. System Prompt Structure Pattern

### System Prompt Template
```python
system_prompt = """You are an intelligent search and research assistant with access to real-time web information.

    Your capabilities include:
    - Searching the web for current information and news
    - Researching topics across various domains
    - Providing accurate, up-to-date answers with reliable sources
    - Synthesizing information from multiple sources
    - Fact-checking and verification

    When responding:
    - Always cite your sources when possible
    - Distinguish between factual information and opinions
    - Provide comprehensive yet concise answers
    - If information is uncertain or contradictory, mention this
    - Suggest follow-up questions when appropriate
    - Focus on accuracy and reliability

    For research queries:
    1. Search for the most current and relevant information
    2. Cross-reference multiple sources when possible
    3. Provide context and background information
    4. Summarize key findings clearly
    5. Highlight any limitations or uncertainties in the data"""
```

**Pattern Requirements:**
- Start with clear role definition
- List specific capabilities in bullet points
- Include behavioral guidelines ("When responding:")
- Provide step-by-step instructions for complex workflows
- Use consistent indentation (4 spaces)
- Include quality and accuracy guidelines

## 7. Error Handling Patterns

### Exception Handling Structure
```python
try:
    # Main operation
    response = brave_search_agent(user_input)
except Exception as e:
    print(f"❌ Error processing search query: {str(e)}")
    print("🔧 Please try rephrasing your question or check your connection")
```

**Pattern Requirements:**
- Use emoji indicators (❌ for errors, 🔧 for solutions)
- Provide both technical error details and user-friendly guidance
- Include recovery suggestions
- Use `str(e)` for exception message extraction

### Logging Format (Following CLAUDE.md Guidelines)
```python
# When implementing logging, use format:
# [FILENAME-FUNCTION] description and log of what is happening
print(f"[brave_search_agent-initialize] Connecting to MCP server")
```

## 8. Interactive CLI Interface Pattern

### Welcome Banner Structure
```python
print("====================================================================================")
print("🔍  WELCOME TO YOUR PERSONAL SEARCH ASSISTANT  🔍")
print("====================================================================================")
print("🌐 I'm your intelligent research companion ready to help with:")
print("   🔎 Real-time web searches and information lookup")
print("   📰 Current news and trending topics")
# ... more capabilities
print()
print("🛠️  Powered by:")
print("   • Brave AI - Advanced web search capabilities")
# ... more details
print()
print("💡 Tips:")
print("   • Ask specific questions for better results")
# ... more tips
print()
print("🚪 Type 'exit' to quit anytime")
print("====================================================================================")
```

### Interactive Loop Pattern
```python
while True:
    try:
        user_input = input("🔍 You: ").strip()
        if not user_input:
            print("💭 Please ask me a question or type 'exit' to quit")
            continue

        if user_input.lower() in ["exit", "quit", "bye", "goodbye"]:
            print()
            print("========================================================")
            print("👋 Thanks for exploring with me!")
            print("🌐 Keep discovering and learning!")
            print("🔍 See you next time!")
            print("========================================================")
            break

        print("🤖 SearchBot: ", end="")
        try:
            response = brave_search_agent(user_input)
        except Exception as e:
            print(f"❌ Error processing search query: {str(e)}")
            print("🔧 Please try rephrasing your question or check your connection")

    except KeyboardInterrupt:
        print("\n")
        print("============================================================")
        print("👋 Brave Search Assistant interrupted!")
        print("🌐 Thanks for researching with me!")
        print("🔍 Happy exploring!")
        print("============================================================")
        break
    except Exception as e:
        print(f"❌ An error occurred: {str(e)}")
        print("🔧 Please try again or type 'exit' to quit")
        print()
```

**Pattern Requirements:**
- Use consistent emoji theming throughout
- Provide clear visual separators with equal signs
- Include multiple exit commands
- Handle both normal exit and KeyboardInterrupt
- Provide contextual error messages
- Use `end=""` for inline responses
- Strip input and validate non-empty

## 9. Session Management Pattern

### Session ID Generation
```python
# In constants.py
import uuid

# Generate a unique session ID for tracing and debugging
SESSION_ID = str(uuid.uuid4())
```

### Session Integration
```python
agent = Agent(
    model=model,
    system_prompt=system_prompt,
    tools=tools,
    trace_attributes={"session.id": SESSION_ID},
)
```

**Pattern Requirements:**
- Generate UUID4 for unique session identification
- Store in constants.py for reuse across modules
- Include in agent trace_attributes for debugging
- Use string conversion of UUID

## 10. Docker Integration Pattern

### Docker Command Structure
```python
StdioServerParameters(
    command="docker",
    args=[
        "run",
        "-i",           # Interactive mode
        "--rm",         # Remove container after exit
        "-e",           # Environment variable
        f"BRAVE_API_KEY={BRAVE_API_KEY}",
        "mcp/brave-search",  # Container image
    ],
)
```

**Pattern Requirements:**
- Use interactive mode (`-i`) for stdio communication
- Auto-remove containers (`--rm`) for cleanup
- Pass secrets through environment variables
- Use descriptive container names

## 11. Response Dictionary Structure Pattern

### Simple String Response
```python
def agent_function(query: str) -> str:
    response = agent(query)
    return response
```

**Pattern Requirements:**
- Keep tool functions simple with string returns
- Let the agent handle complex response formatting
- Avoid complex dictionary structures in tool functions
- Delegate formatting to the underlying Strands agent

## 12. Common Anti-Patterns to Avoid

### ❌ Don't Do This:
```python
# Don't mix initialization and execution
def create_agent_and_run(query):
    model = BedrockModel(...)  # Bad: creates new model each time
    agent = Agent(...)
    return agent(query)

# Don't ignore environment validation
SOME_KEY = os.getenv("SOME_KEY")  # Bad: no validation

# Don't use generic error messages
except Exception as e:
    print("Error occurred")  # Bad: not helpful

# Don't hardcode configuration
model_id = "claude-3"  # Bad: not specific or current
```

### ✅ Do This Instead:
```python
# Initialize once at module level
model = BedrockModel(model_id="us.anthropic.claude-3-7-sonnet-20250219-v1:0")
agent = Agent(model=model, ...)

def agent_function(query: str) -> str:
    return agent(query)  # Good: reuse existing agent

# Validate environment variables
SOME_KEY = os.getenv("SOME_KEY")
if not SOME_KEY:
    raise ValueError("SOME_KEY environment variable is required")

# Provide helpful error messages
except Exception as e:
    print(f"❌ Error processing request: {str(e)}")
    print("🔧 Please check your input and try again")
```

## 13. File Naming and Organization Conventions

### File Naming Pattern
- `{service_name}_agent.py` - Main agent implementation
- `constants.py` - Shared constants and configuration
- `requirements.txt` - Python dependencies
- `client.py` - Basic client examples (optional)

### Function Naming Pattern
- `{service_name}_agent(query: str) -> str` - Main tool function
- Use descriptive system_prompt variable names
- Use service-specific MCP client variable names (e.g., `brave_mcp_server`)

## 14. Implementation Checklist

When implementing a new agent, ensure:

- [ ] Import structure follows the standard pattern
- [ ] Environment variables are loaded and validated
- [ ] MCP client is properly initialized with error handling
- [ ] Agent uses latest Claude Sonnet model ID
- [ ] System prompt follows the structured template
- [ ] Tool function uses @tool decorator with proper docstring
- [ ] Session tracking is included with SESSION_ID
- [ ] Interactive CLI follows the banner and loop patterns
- [ ] Error handling includes emoji indicators and helpful messages
- [ ] Docker integration follows best practices
- [ ] Context manager lifecycle is properly handled
- [ ] Exit commands and KeyboardInterrupt are handled
- [ ] Response formatting is consistent

## Summary

The Strands agent pattern emphasizes:
1. **Modularity**: Clean separation of concerns with proper imports
2. **Reliability**: Comprehensive error handling and validation
3. **Usability**: Rich interactive CLI with clear feedback
4. **Maintainability**: Consistent patterns and reusable components
5. **Observability**: Session tracking and proper logging
6. **Security**: Environment variable validation and Docker isolation

This pattern provides a robust foundation for implementing new agents while maintaining consistency across the codebase.