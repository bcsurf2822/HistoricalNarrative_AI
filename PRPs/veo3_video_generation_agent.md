name: "VEO3 Video Generation Agent Implementation"
description: |

## Purpose
Comprehensive PRP for implementing VEO3 video generation agent using Strands framework with LiteLLM integration for historical narrative video creation.

## Core Principles
1. **Context is King**: Include ALL necessary documentation, examples, and caveats
2. **Validation Loops**: Provide executable tests/lints the AI can run and fix
3. **Information Dense**: Use keywords and patterns from the codebase
4. **Progressive Success**: Start simple, validate, then enhance
5. **Global rules**: Be sure to follow all rules in CLAUDE.md

---

## Goal

**Feature Goal**: Implement a fully functional VEO3 video generation agent that integrates seamlessly with the existing Strands agent ecosystem, enabling text-to-video and image-to-video generation using Google's VEO3 model via LiteLLM proxy.

**Deliverable**: Complete `agent/video_generation_agent.py` with:
- Text-to-video generation capability using VEO3
- Image-to-video generation with input image support
- Interactive CLI interface following existing patterns from `brave_search_agent.py`
- Async polling for long-running video operations
- Comprehensive error handling and retry logic
- @tool decorator integration for use by other agents

**Success Definition**: An AI agent can successfully generate 8-second videos from text prompts or input images, with proper status tracking, error handling, and user feedback, deployed as both standalone CLI and integrated tool.

## Why

- **Historical Narrative Enhancement**: Enable creation of visual storytelling content for historical narratives in the hackathon project
- **Agent Ecosystem Integration**: Expand the existing multi-agent system (brave_search_agent) with video generation capabilities
- **Production-Ready Implementation**: Leverage Google's state-of-the-art VEO3 model for high-quality video generation with audio
- **Scalable Architecture**: Build foundation for future video generation features and workflows

## What

### User-Visible Behavior
- Interactive CLI that accepts text prompts for video generation
- Support for image-to-video generation with file path input
- Real-time progress updates during 30s-6min generation process
- Video URI output with automatic SynthID watermarking
- Graceful error handling with user-friendly messages
- @tool decorator exposure for integration with other agents

### Technical Requirements
- VEO3 model integration via LiteLLM proxy
- Google AI Studio authentication setup
- Async/await patterns for long-running operations
- Exponential backoff retry logic with jitter
- Session tracking using existing UUID system
- Comprehensive logging following [FILENAME-FUNCTION] format
- Support for multiple aspect ratios (16:9, 9:16, 1:1)
- Resolution options (720p, 1080p)
- Negative prompt support for content control

### Success Criteria
- [ ] Generate 8-second video from text prompt in under 6 minutes
- [ ] Generate video from input image with proper validation
- [ ] Handle all error scenarios gracefully without crashes
- [ ] Interactive CLI matches existing agent patterns
- [ ] Integration as @tool works with other agents
- [ ] All linting and type checking passes
- [ ] Unit tests cover core functionality
- [ ] Real API integration test succeeds

## All Needed Context

### Documentation & References (list all context needed to implement the feature)
```yaml
# MUST READ - Include these in your context window

# Codebase Analysis - Created by research agents
- docfile: PRPs/planning/strands_agent_patterns.md
  why: Complete reference patterns from brave_search_agent.py including @tool usage, MCP integration, CLI patterns, error handling, and session tracking

- docfile: PRPs/planning/project_structure_analysis.md
  why: Project organization, dependencies, configuration patterns, and integration guidelines for new agents

# Technical Implementation - Created by research agents
- docfile: PRPs/ai_docs/litellm_veo3_integration.md
  why: Complete LiteLLM setup, VEO3 API configuration, authentication patterns, error handling, and production deployment guidance

- docfile: PRPs/ai_docs/video_generation_best_practices.md
  why: Industry best practices for video API implementations, polling strategies, error handling, and UX patterns

- docfile: PRPs/ai_docs/python_async_patterns.md
  why: Modern async/await patterns, retry logic implementations, context managers, and long-running operation handling

# Reference Implementation Files
- file: agent/brave_search_agent.py
  why: Primary template for agent structure, MCP integration patterns, interactive CLI design, and @tool decorator usage

- file: agent/constants.py
  why: Session tracking pattern with UUID generation for debugging and tracing

- file: agent/requirements.txt
  why: Current dependency management approach and where to add new dependencies

- file: agent/.env
  why: Environment variable patterns and authentication setup approach

# Official Documentation URLs - From research
- url: https://docs.litellm.ai/docs/proxy/veo_video_generation
  why: Primary LiteLLM VEO3 integration documentation with proxy setup and configuration examples

- url: https://ai.google.dev/gemini-api/docs/video
  why: Official VEO3 API reference with request/response formats and parameter specifications

- url: https://docs.litellm.ai/docs/pass_through/google_ai_studio
  why: Google AI Studio authentication and setup patterns for LiteLLM integration

- url: https://ai.google.dev/gemini-api/docs/rate-limits
  why: VEO3 rate limiting and quota considerations for production implementation
```

### Current Codebase tree (run `tree` in the root of the project) to get an overview of the codebase
```bash
HistoricalNarrative_AI/
├── agent/
│   ├── .venv/
│   ├── __pycache__/
│   ├── .env                           # Environment variables
│   ├── brave_search_agent.py          # Reference implementation
│   ├── client.py                      # Basic Bedrock client
│   ├── constants.py                   # Session tracking
│   ├── interactive_history_agent.py   # Placeholder
│   ├── requirements.txt               # Dependencies
│   └── video_generation_agent.py      # Target implementation (empty)
├── PRPs/
│   ├── ai_docs/                       # Research documentation
│   ├── planning/                      # Codebase analysis
│   └── templates/
└── README.md
```

### Desired Codebase tree with files to be added and responsibility of file
```bash
agent/
├── video_generation_agent.py          # Main implementation - VEO3 agent with @tool decorator
├── requirements.txt                   # Updated with litellm dependency
└── .env                               # Updated with GOOGLE_API_KEY variable
```

### Known Gotchas of our codebase & Library Quirks
```python
# CRITICAL: Strands framework requires specific patterns
# - @tool decorator for agent exposure to other agents
# - MCP client integration for external services via Docker
# - BedrockModel for LLM backend (not LiteLLM for agent itself)
# - Session tracking via constants.SESSION_ID for tracing

# CRITICAL: VEO3 via LiteLLM has specific constraints
# - Requires LiteLLM proxy server running on localhost:4000
# - Only 8-second videos maximum length currently
# - Long-running operations (30s-6min) require polling
# - Rate limits are strict even for paid accounts
# - Content policy violations fail silently in some cases

# CRITICAL: Environment setup dependencies
# - Google AI Studio API key (not Vertex AI)
# - LiteLLM proxy server must be running before agent
# - Docker required for MCP integration pattern (see brave_search_agent.py)

# CRITICAL: Async patterns required
# - All video generation operations are long-running
# - Must use async/await with proper context managers
# - Exponential backoff with jitter for retries
# - Resource cleanup essential for memory management
```

## Implementation Blueprint

### Data models and structure

Create the core data models to ensure type safety and consistency with VEO3 operations.

```python
# Video Generation Request/Response Models
from typing import Dict, Any, Optional, Literal
from dataclasses import dataclass

@dataclass
class VideoGenerationRequest:
    prompt: str
    aspect_ratio: Literal["16:9", "9:16", "1:1"] = "16:9"
    resolution: Literal["720p", "1080p"] = "720p"
    negative_prompt: str = ""
    image_path: Optional[str] = None  # For image-to-video

@dataclass
class VideoGenerationResult:
    success: bool
    operation_name: Optional[str] = None
    video_uri: Optional[str] = None
    error: Optional[str] = None
    status: str = "pending"
    model: str = "veo-3.0-generate-preview"

@dataclass
class OperationStatus:
    done: bool
    success: bool
    video_uri: Optional[str] = None
    error: Optional[str] = None
```

### List of tasks to be completed to fulfill the PRP in the order they should be completed

```yaml
Task 1: Environment Setup
UPDATE agent/requirements.txt:
  - ADD 'strands-agents[litellm]' replacing existing strands-agents
  - PRESERVE existing dependencies (uv, strands-agents-tools, mcp)
  - VERIFY no version conflicts

UPDATE agent/.env:
  - ADD GOOGLE_API_KEY environment variable
  - PRESERVE existing BRAVE_API_KEY and AWS credentials
  - FOLLOW existing environment variable patterns

Task 2: Core Video Generation Tools
CREATE agent/video_generation_agent.py:
  - MIRROR import structure from: agent/brave_search_agent.py
  - IMPORT required modules (os, time, requests, base64, asyncio)
  - IMPORT Strands components (@tool, Agent, BedrockModel)
  - IMPORT constants.SESSION_ID for session tracking
  - SETUP environment loading with load_dotenv()

Task 3: Video Generation Functions Implementation
IMPLEMENT generate_video_from_text function:
  - USE @tool decorator following brave_search_agent.py pattern
  - ACCEPT (prompt, aspect_ratio, resolution, negative_prompt) parameters
  - RETURN VideoGenerationResult with proper error handling
  - INTEGRATE LiteLLM proxy calls to VEO3 endpoint

IMPLEMENT generate_video_from_image function:
  - USE @tool decorator pattern
  - ACCEPT (prompt, image_path, aspect_ratio, resolution) parameters
  - VALIDATE image file exists before processing
  - ENCODE image as base64 for API submission
  - RETURN VideoGenerationResult with proper error handling

IMPLEMENT check_video_status function:
  - USE @tool decorator pattern
  - ACCEPT operation_name parameter
  - RETURN current operation status
  - HANDLE both completed and in-progress states

Task 4: Polling and Async Operation Support
IMPLEMENT _wait_for_video_completion helper:
  - PRIVATE function (no @tool decorator)
  - IMPLEMENT exponential backoff with jitter pattern
  - USE max_wait_time parameter (default 360 seconds)
  - POLL every 20 seconds for operation status
  - RETURN VideoGenerationResult when complete

Task 5: Agent Configuration and System Prompt
CONFIGURE Strands Agent:
  - USE BedrockModel with claude-3-7-sonnet (following brave_search_agent.py)
  - CREATE comprehensive system_prompt describing capabilities
  - INTEGRATE session tracking with SESSION_ID
  - REGISTER all @tool functions

Task 6: Interactive CLI Implementation
IMPLEMENT main CLI interface:
  - MIRROR CLI patterns from brave_search_agent.py exactly
  - USE same emoji theming and user experience patterns
  - IMPLEMENT input validation and graceful error handling
  - PROVIDE clear progress feedback during generation
  - HANDLE keyboard interrupts and exit commands

Task 7: Error Handling and Retry Logic
IMPLEMENT robust_video_generation helper:
  - ASYNC function with retry logic
  - USE exponential backoff for rate limit errors
  - HANDLE quota exceeded, authentication, and network errors
  - PROVIDE meaningful error messages to users
  - LOG errors using [FILENAME-FUNCTION] format

Task 8: Integration Testing and Validation
CREATE test scenarios:
  - VERIFY text-to-video generation works end-to-end
  - TEST image-to-video generation with sample image
  - VALIDATE error handling with invalid inputs
  - CONFIRM @tool integration with other agents
```

### Per task pseudocode as needed added to each task

```python
# Task 3: Video Generation Functions Implementation
@tool
def generate_video_from_text(
    prompt: str,
    aspect_ratio: str = "16:9",
    resolution: str = "720p",
    negative_prompt: str = ""
) -> Dict[str, Any]:
    """Generate video from text using VEO3 via LiteLLM proxy"""
    try:
        print(f"[VIDEO_AGENT-generate_video_from_text] Starting generation: {prompt}")

        # PATTERN: Validate input first (see constants patterns)
        if not prompt.strip():
            return {"success": False, "error": "Empty prompt provided"}

        # GOTCHA: LiteLLM proxy must be running on localhost:4000
        url = f"{LITELLM_BASE_URL}/models/veo-3.0-generate-preview:predictLongRunning"
        payload = {
            "instances": [{
                "prompt": prompt,
                "negative_prompt": negative_prompt,
                "aspect_ratio": aspect_ratio,
                "resolution": resolution
            }]
        }

        # CRITICAL: Use proper headers with Google API key
        headers = {
            "Content-Type": "application/json",
            "x-goog-api-key": os.environ["GOOGLE_API_KEY"]
        }

        response = requests.post(url, headers=headers, json=payload)

        if response.status_code != 200:
            return {"success": False, "error": f"API call failed: {response.text}"}

        operation_name = response.json().get("name")

        # PATTERN: Use polling helper for long operations
        result = _wait_for_video_completion(operation_name)

        return {
            "success": result["success"],
            "operation_name": operation_name,
            "video_uri": result.get("video_uri"),
            "status": "completed" if result["success"] else "failed",
            "prompt": prompt
        }

    except Exception as e:
        print(f"[VIDEO_AGENT-generate_video_from_text] Error: {str(e)}")
        return {"success": False, "error": str(e), "prompt": prompt}

# Task 4: Polling Implementation
def _wait_for_video_completion(operation_name: str, max_wait_time: int = 360) -> Dict[str, Any]:
    """Poll for video completion with exponential backoff"""
    print(f"[VIDEO_AGENT-_wait_for_video_completion] Polling: {operation_name}")
    start_time = time.time()

    while time.time() - start_time < max_wait_time:
        try:
            # GOTCHA: VEO3 operations can take 30s-6min
            status_url = f"{LITELLM_BASE_URL}/{operation_name}"
            response = requests.get(status_url, headers=VEO_HEADERS)

            if response.status_code == 200:
                data = response.json()

                if data.get("done"):
                    # PATTERN: Extract video URI from nested response
                    if "response" in data:
                        video_response = data["response"]
                        samples = video_response.get("generateVideoResponse", {}).get("generatedSamples", [])

                        if samples and "video" in samples[0]:
                            return {
                                "success": True,
                                "video_uri": samples[0]["video"]["uri"]
                            }

                    # CRITICAL: Check for errors in response
                    if "error" in data:
                        return {"success": False, "error": data["error"]}

                # PATTERN: Progressive backoff - start with 20s intervals
                print(f"[VIDEO_AGENT-_wait_for_video_completion] Still processing...")
                time.sleep(20)
            else:
                print(f"[VIDEO_AGENT-_wait_for_video_completion] Status check failed")
                time.sleep(10)

        except Exception as e:
            print(f"[VIDEO_AGENT-_wait_for_video_completion] Polling error: {str(e)}")
            time.sleep(10)

    return {"success": False, "error": f"Timeout after {max_wait_time} seconds"}

# Task 5: Agent Configuration
model = BedrockModel(
    model_id="us.anthropic.claude-3-7-sonnet-20250219-v1:0",  # PATTERN: Same as brave_search_agent
)

system_prompt = """You are a professional video generation assistant powered by Google's VEO3 model.

Your capabilities include:
- Generate 8-second videos from text descriptions with native audio
- Create videos from input images (image-to-video)
- Support multiple aspect ratios (16:9, 9:16, 1:1)
- Handle different resolutions (720p, 1080p)
- Process negative prompts to avoid unwanted content

CRITICAL CONSTRAINTS:
- Videos are limited to 8 seconds maximum
- Generation takes 30 seconds to 6 minutes
- Rate limits apply even for paid accounts
- All videos include SynthID watermarking

When generating videos:
1. Validate input parameters thoroughly
2. Provide clear status updates during processing
3. Handle errors gracefully with helpful messages
4. Suggest optimal parameters for user's use case
5. Explain processing time expectations"""

# PATTERN: Follow brave_search_agent initialization exactly
agent = Agent(
    model=model,
    system_prompt=system_prompt,
    tools=[generate_video_from_text, generate_video_from_image, check_video_status],
    trace_attributes={"session.id": SESSION_ID},
)
```

### Integration Points
```yaml
ENVIRONMENT:
  - add to: agent/.env
  - pattern: "GOOGLE_API_KEY=your_google_ai_studio_api_key"
  - required: LiteLLM proxy server running on localhost:4000

DEPENDENCIES:
  - add to: agent/requirements.txt
  - pattern: "'strands-agents[litellm]' instead of 'strands-agents'"
  - preserve: existing uv, strands-agents-tools, mcp dependencies

AGENT_INTEGRATION:
  - expose via: @tool decorator for each function
  - pattern: other agents can call video_generation_agent("prompt text")
  - session tracking: uses constants.SESSION_ID for tracing

LITELLM_PROXY:
  - startup requirement: litellm --port 4000
  - configuration: Google AI Studio API key
  - endpoint: http://localhost:4000/models/veo-3.0-generate-preview
```

## Validation Loop

### Level 1: Syntax & Style
```bash
# Run these FIRST - fix any errors before proceeding
# Note: This project uses Python with uv for dependency management

# Check Python syntax and basic validation
python -m py_compile agent/video_generation_agent.py

# If available, run type checking (install mypy if needed via uv)
# uv pip install mypy
# mypy agent/video_generation_agent.py --ignore-missing-imports

# Expected: No syntax errors. If errors exist, fix them before proceeding.
```

### Level 2: Environment and Dependencies Validation
```bash
# Verify environment setup
cd agent/

# Check that .env has required variables
grep -q "GOOGLE_API_KEY" .env || echo "ERROR: GOOGLE_API_KEY missing from .env"
grep -q "BRAVE_API_KEY" .env && echo "✓ BRAVE_API_KEY found"

# Verify requirements.txt has been updated
grep -q "strands-agents\[litellm\]" requirements.txt || echo "ERROR: strands-agents[litellm] not found in requirements.txt"

# Install dependencies using uv
uv pip install -r requirements.txt

# Expected: All dependencies install without conflicts
```

### Level 3: LiteLLM Proxy Setup Validation
```bash
# CRITICAL: LiteLLM proxy must be running before testing
# Start in separate terminal: litellm --port 4000

# Test proxy connectivity
curl -s http://localhost:4000/health || echo "ERROR: LiteLLM proxy not running on port 4000"

# Test Google AI Studio authentication
export GOOGLE_API_KEY=$(grep GOOGLE_API_KEY agent/.env | cut -d'=' -f2)
curl -s -H "x-goog-api-key: $GOOGLE_API_KEY" \
  -H "Content-Type: application/json" \
  http://localhost:4000/models || echo "ERROR: Authentication failed"

# Expected: Proxy responds with available models including VEO3
```

### Level 4: Unit Tests for Core Functions
```python
# CREATE test_video_generation_agent.py in agent/ directory
import sys
import os
sys.path.append(os.path.dirname(os.path.abspath(__file__)))

from video_generation_agent import generate_video_from_text, generate_video_from_image, check_video_status

def test_text_to_video_validation():
    """Test input validation for text-to-video"""
    # Test empty prompt
    result = generate_video_from_text("")
    assert result["success"] == False
    assert "empty" in result["error"].lower()
    print("✓ Empty prompt validation works")

def test_image_to_video_validation():
    """Test input validation for image-to-video"""
    # Test non-existent image file
    result = generate_video_from_image("test prompt", "/nonexistent/image.jpg")
    assert result["success"] == False
    assert "not found" in result["error"].lower()
    print("✓ Image file validation works")

def test_video_status_check():
    """Test video status checking with invalid operation"""
    result = check_video_status("invalid_operation_id")
    assert result["success"] == False
    print("✓ Status check validation works")

if __name__ == "__main__":
    test_text_to_video_validation()
    test_image_to_video_validation()
    test_video_status_check()
    print("All validation tests passed!")
```

```bash
# Run validation tests
cd agent/
python test_video_generation_agent.py

# Expected: All validation tests pass without errors
```

### Level 5: Integration Test with Real API
```bash
# CRITICAL: Only run if LiteLLM proxy is running and authenticated
# This will consume API quota - use sparingly

# Create test script for real API integration
cat > agent/test_integration.py << 'EOF'
import sys
import os
sys.path.append(os.path.dirname(os.path.abspath(__file__)))

from video_generation_agent import generate_video_from_text

def test_real_video_generation():
    """Test actual video generation with VEO3 - CONSUMES QUOTA"""
    print("[INTEGRATION_TEST] Starting real video generation test...")

    # Simple, safe prompt
    prompt = "A single red apple on a white table"

    result = generate_video_from_text(
        prompt=prompt,
        aspect_ratio="1:1",  # Square for fastest generation
        resolution="720p"    # Lower resolution for speed
    )

    print(f"[INTEGRATION_TEST] Result: {result}")

    if result["success"]:
        print(f"✓ Video generated successfully: {result['video_uri']}")
        return True
    else:
        print(f"✗ Video generation failed: {result['error']}")
        return False

if __name__ == "__main__":
    success = test_real_video_generation()
    if success:
        print("Integration test PASSED")
    else:
        print("Integration test FAILED")
EOF

# Run integration test (WARNING: Consumes API quota)
python agent/test_integration.py

# Expected: Video generation completes with valid video_uri returned
```

### Level 6: Interactive CLI Testing
```bash
# Test the interactive CLI interface
cd agent/
python video_generation_agent.py

# Manual test checklist:
# 1. CLI starts with proper welcome message and emoji formatting
# 2. Accepts text prompts and provides progress feedback
# 3. Handles invalid inputs gracefully
# 4. Provides clear error messages for failures
# 5. Supports 'exit' command to quit gracefully
# 6. Handles Ctrl+C interruption properly

# Expected: Interactive experience matches brave_search_agent.py patterns
```

### Level 7: Agent Tool Integration Test
```python
# CREATE test_agent_integration.py in agent/ directory
import sys
import os
sys.path.append(os.path.dirname(os.path.abspath(__file__)))

from video_generation_agent import video_generation_agent

def test_agent_tool_usage():
    """Test using video generation as a tool from another agent"""
    print("[AGENT_INTEGRATION_TEST] Testing @tool decorator functionality...")

    # Test calling the agent as a tool
    response = video_generation_agent(
        "Generate a short video of a sunrise over mountains in 16:9 format"
    )

    print(f"[AGENT_INTEGRATION_TEST] Agent response: {response}")

    # Agent should return a string response, not a dict
    assert isinstance(response, str), "Agent should return string response"
    assert len(response) > 0, "Agent response should not be empty"

    print("✓ Agent tool integration works correctly")

if __name__ == "__main__":
    test_agent_tool_usage()
    print("Agent integration test completed!")
```

```bash
# Run agent integration test
cd agent/
python test_agent_integration.py

# Expected: Agent responds as a tool with proper string output
```

## Final Validation Checklist

### Pre-Deployment Verification
- [ ] All syntax validation passes: `python -m py_compile agent/video_generation_agent.py`
- [ ] Environment variables configured: `grep GOOGLE_API_KEY agent/.env`
- [ ] Dependencies installed: `uv pip install -r requirements.txt`
- [ ] LiteLLM proxy running: `curl http://localhost:4000/health`
- [ ] Unit tests pass: `python agent/test_video_generation_agent.py`
- [ ] Integration test succeeds: `python agent/test_integration.py` (optional - consumes quota)
- [ ] Interactive CLI works: Manual testing of `python agent/video_generation_agent.py`
- [ ] Agent tool integration works: `python agent/test_agent_integration.py`

### Success Metrics
- [ ] Text-to-video generation completes in under 6 minutes
- [ ] Image-to-video generation works with valid image files
- [ ] Error handling graceful for all failure scenarios
- [ ] CLI interface matches existing patterns from brave_search_agent.py
- [ ] @tool decorator integration allows other agents to use video generation
- [ ] Session tracking works with constants.SESSION_ID
- [ ] Logging follows [FILENAME-FUNCTION] format consistently

### Performance Validation
- [ ] Video generation operations don't block other processes
- [ ] Memory usage remains reasonable during long operations
- [ ] Network errors are handled with exponential backoff
- [ ] API rate limits respected with proper delays
- [ ] Generated videos include SynthID watermarking
- [ ] Video URIs are valid and accessible for 2+ days

---

## Anti-Patterns to Avoid
- ❌ Don't implement async patterns without proper context managers
- ❌ Don't ignore VEO3's 8-second video length limitation
- ❌ Don't skip polling - video generation is always long-running
- ❌ Don't hardcode API endpoints - use environment configuration
- ❌ Don't bypass input validation for performance
- ❌ Don't use sync requests library for long operations without timeouts
- ❌ Don't forget to handle LiteLLM proxy connection failures

## Confidence Score

**9/10** - High confidence for one-pass implementation success

**Reasoning**: This PRP provides comprehensive implementation guidance with:
- Complete codebase pattern analysis from existing working agent
- Detailed technical documentation with specific URLs and configurations
- Step-by-step implementation tasks with pseudocode including critical gotchas
- Extensive validation loops with executable commands
- Integration patterns proven in the existing codebase
- Error handling strategies based on real-world VEO3 limitations

**Risk Mitigation**: The validation loops provide multiple checkpoints to catch and fix issues early, and the reference implementation (brave_search_agent.py) provides a proven pattern to follow.