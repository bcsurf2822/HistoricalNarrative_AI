# VEO3 Video Generation with Strands and LiteLLM

## Goal

Use StrandsAgent SDK with LiteLLM to connect to Google AI Studio for VEO3 video generation. Based on research, Google AI Studio is the recommended approach for easier authentication and setup compared to Vertex AI.

We will place our code in `agent/video_generation_agent.py` and if recommended create the other files

## Strands VEO3 Video Generation Implementation

### 1. Environment Setup

```bash
# Install Strands with LiteLLM support
pip install 'strands-agents[litellm]' strands-agents-tools

# Set your Google AI Studio API key
export GEMINI_API_KEY="your_google_ai_studio_api_key"

# Start LiteLLM proxy server (required for VEO3 video generation)
litellm --port 4000
```

### 2. Core Python Implementation

```python
import os
import requests
import time
from typing import Optional, Dict, Any
from strands import Agent, tool
from strands.models.litellm import LiteLLMModel

# Configure environment
os.environ["GEMINI_API_KEY"] = "your_google_ai_studio_api_key"

# LiteLLM proxy configuration for VEO3 video generation
LITELLM_BASE_URL = "http://localhost:4000"
VEO_HEADERS = {
    "Content-Type": "application/json",
    "x-goog-api-key": os.environ["GEMINI_API_KEY"]
}

# Configure LiteLLM model for the Strands agent (text-based interactions)
# Note: VEO video generation is handled via LiteLLM proxy endpoints, not the model directly
agent_model = LiteLLMModel(
    client_args={
        "api_key": os.environ["GEMINI_API_KEY"]
    },
    model_id="gemini-1.5-flash",  # Use Gemini for agent text processing
    params={
        "temperature": 0.7,
        "max_tokens": 1000
    }
)
```

### 3. Video Generation Tools

```python
@tool
def generate_video_from_text(
    prompt: str,
    aspect_ratio: str = "16:9",
    resolution: str = "720p",
    negative_prompt: str = ""
) -> Dict[str, Any]:
    """
    Generate a video using Google VEO3 model from text prompt via LiteLLM proxy.

    Args:
        prompt: Text description for the video to generate
        aspect_ratio: Video aspect ratio (16:9, 9:16, or 1:1)
        resolution: Video resolution (720p or 1080p)
        negative_prompt: What to avoid in the video

    Returns:
        Dictionary with generation result
    """
    try:
        print(f"[VIDEO_AGENT-generate_video_from_text] Initiating video generation for: {prompt}")

        # Step 1: Initiate video generation via LiteLLM proxy
        url = f"{LITELLM_BASE_URL}/models/veo-3.0-generate-preview:predictLongRunning"
        payload = {
            "instances": [{
                "prompt": prompt,
                "negative_prompt": negative_prompt,
                "aspect_ratio": aspect_ratio,
                "resolution": resolution
            }]
        }

        response = requests.post(url, headers=VEO_HEADERS, json=payload)

        if response.status_code != 200:
            return {
                "success": False,
                "error": f"Failed to initiate video generation: {response.text}",
                "prompt": prompt
            }

        operation_name = response.json().get("name")
        print(f"[VIDEO_AGENT-generate_video_from_text] Operation started: {operation_name}")

        # Step 2: Poll for completion
        operation_result = _wait_for_video_completion(operation_name)

        if operation_result["success"]:
            return {
                "success": True,
                "prompt": prompt,
                "aspect_ratio": aspect_ratio,
                "resolution": resolution,
                "operation_name": operation_name,
                "video_uri": operation_result["video_uri"],
                "status": "Video generation completed",
                "model": "veo-3.0-generate-preview"
            }
        else:
            return operation_result

    except Exception as e:
        print(f"[VIDEO_AGENT-generate_video_from_text] Error: {str(e)}")
        return {
            "success": False,
            "error": str(e),
            "prompt": prompt
        }

def _wait_for_video_completion(operation_name: str, max_wait_time: int = 360) -> Dict[str, Any]:
    """
    Poll for video generation completion via LiteLLM proxy.

    Args:
        operation_name: The operation name returned from video generation
        max_wait_time: Maximum time to wait in seconds (default 6 minutes)

    Returns:
        Dictionary with completion result
    """
    print(f"[VIDEO_AGENT-_wait_for_video_completion] Polling operation: {operation_name}")
    start_time = time.time()

    while time.time() - start_time < max_wait_time:
        try:
            # Check operation status via LiteLLM proxy
            status_url = f"{LITELLM_BASE_URL}/{operation_name}"
            response = requests.get(status_url, headers=VEO_HEADERS)

            if response.status_code == 200:
                operation_data = response.json()

                if operation_data.get("done"):
                    # Operation completed - extract video URI
                    if "response" in operation_data:
                        video_response = operation_data["response"]
                        generated_samples = video_response.get("generateVideoResponse", {}).get("generatedSamples", [])

                        if generated_samples and "video" in generated_samples[0]:
                            video_uri = generated_samples[0]["video"]["uri"]
                            print(f"[VIDEO_AGENT-_wait_for_video_completion] Video ready: {video_uri}")
                            return {
                                "success": True,
                                "video_uri": video_uri,
                                "operation_name": operation_name
                            }

                    # Check for error in response
                    if "error" in operation_data:
                        return {
                            "success": False,
                            "error": f"Video generation failed: {operation_data['error']}",
                            "operation_name": operation_name
                        }

                print(f"[VIDEO_AGENT-_wait_for_video_completion] Still processing... waiting 20 seconds")
                time.sleep(20)
            else:
                print(f"[VIDEO_AGENT-_wait_for_video_completion] Status check failed: {response.status_code}")
                time.sleep(10)

        except Exception as e:
            print(f"[VIDEO_AGENT-_wait_for_video_completion] Error polling: {str(e)}")
            time.sleep(10)

    return {
        "success": False,
        "error": f"Video generation timed out after {max_wait_time} seconds",
        "operation_name": operation_name
    }

@tool
def generate_video_from_image(
    prompt: str,
    image_path: str,
    aspect_ratio: str = "16:9",
    resolution: str = "720p"
) -> Dict[str, Any]:
    """
    Generate a video using Google VEO3 model from an input image via LiteLLM proxy.

    Args:
        prompt: Text description for the video to generate
        image_path: Path to the input image file
        aspect_ratio: Video aspect ratio (16:9, 9:16, or 1:1)
        resolution: Video resolution (720p or 1080p)

    Returns:
        Dictionary with generation result
    """
    try:
        print(f"[VIDEO_AGENT-generate_video_from_image] Processing image: {image_path}")

        # Validate image file exists
        if not os.path.exists(image_path):
            return {
                "success": False,
                "error": f"Image file not found: {image_path}",
                "prompt": prompt
            }

        # Read and encode image as base64
        import base64
        with open(image_path, "rb") as image_file:
            image_data = base64.b64encode(image_file.read()).decode('utf-8')

        # Step 1: Initiate video generation from image via LiteLLM proxy
        url = f"{LITELLM_BASE_URL}/models/veo-3.0-generate-preview:predictLongRunning"
        payload = {
            "instances": [{
                "prompt": prompt,
                "input_image": {
                    "mime_type": "image/jpeg",  # Adjust based on actual image type
                    "data": image_data
                },
                "aspect_ratio": aspect_ratio,
                "resolution": resolution
            }]
        }

        response = requests.post(url, headers=VEO_HEADERS, json=payload)

        if response.status_code != 200:
            return {
                "success": False,
                "error": f"Failed to initiate video generation from image: {response.text}",
                "prompt": prompt,
                "image_path": image_path
            }

        operation_name = response.json().get("name")
        print(f"[VIDEO_AGENT-generate_video_from_image] Operation started: {operation_name}")

        # Step 2: Poll for completion
        operation_result = _wait_for_video_completion(operation_name)

        if operation_result["success"]:
            return {
                "success": True,
                "prompt": prompt,
                "image_path": image_path,
                "aspect_ratio": aspect_ratio,
                "resolution": resolution,
                "operation_name": operation_name,
                "video_uri": operation_result["video_uri"],
                "status": "Video generation from image completed",
                "model": "veo-3.0-generate-preview"
            }
        else:
            return operation_result

    except Exception as e:
        print(f"[VIDEO_AGENT-generate_video_from_image] Error: {str(e)}")
        return {
            "success": False,
            "error": str(e),
            "prompt": prompt,
            "image_path": image_path
        }

@tool
def check_video_status(operation_name: str) -> Dict[str, Any]:
    """
    Check the status of a video generation operation via LiteLLM proxy.

    Args:
        operation_name: The operation name returned from video generation

    Returns:
        Dictionary with operation status
    """
    try:
        print(f"[VIDEO_AGENT-check_video_status] Checking operation: {operation_name}")

        # Check operation status via LiteLLM proxy
        status_url = f"{LITELLM_BASE_URL}/{operation_name}"
        response = requests.get(status_url, headers=VEO_HEADERS)

        if response.status_code != 200:
            return {
                "success": False,
                "error": f"Failed to check status: {response.text}",
                "operation_name": operation_name
            }

        operation_data = response.json()

        if operation_data.get("done"):
            # Operation completed
            if "response" in operation_data:
                video_response = operation_data["response"]
                generated_samples = video_response.get("generateVideoResponse", {}).get("generatedSamples", [])

                if generated_samples and "video" in generated_samples[0]:
                    video_uri = generated_samples[0]["video"]["uri"]
                    return {
                        "success": True,
                        "operation_name": operation_name,
                        "status": "completed",
                        "video_uri": video_uri
                    }

            # Check for error
            if "error" in operation_data:
                return {
                    "success": False,
                    "operation_name": operation_name,
                    "status": "failed",
                    "error": operation_data["error"]
                }

        # Still processing
        return {
            "success": True,
            "operation_name": operation_name,
            "status": "processing",
            "message": "Video generation still in progress"
        }

    except Exception as e:
        print(f"[VIDEO_AGENT-check_video_status] Error: {str(e)}")
        return {
            "success": False,
            "error": str(e),
            "operation_name": operation_name
        }
```

### 4. Strands Agent Configuration

```python
# Create the video generation agent using LiteLLM model for text processing
# and LiteLLM proxy endpoints for VEO3 video generation
video_agent = Agent(
    model=agent_model,  # Uses Gemini via LiteLLM for text processing
    system_prompt="""You are a professional video generation assistant powered by Google's VEO3 model via LiteLLM.

You help users create high-quality videos from text descriptions or input images using Google's state-of-the-art VEO3 video generation model.

Available capabilities:
- Generate 8-second videos from text prompts with native audio
- Generate videos from input images (image-to-video)
- Support multiple aspect ratios (16:9, 9:16, 1:1)
- Support different resolutions (720p, 1080p)
- Handle negative prompts to avoid unwanted content
- Check operation status for long-running generations

When users request videos:
1. Use the appropriate generation tool based on their input (text or image)
2. Suggest optimal parameters based on their use case (social media = 9:16, etc.)
3. Provide clear status updates and operation tracking
4. Handle errors gracefully and suggest alternatives
5. Explain that videos include automatically generated audio

Technical notes:
- Video generation uses LiteLLM proxy for Google AI Studio VEO3 access
- Operations are long-running (30 seconds to 6 minutes)
- Videos are retained for 2 days after generation
- All videos include SynthID watermarking""",
    tools=[generate_video_from_text, generate_video_from_image, check_video_status]
)
```

### 5. Usage Examples

```python
# Basic video generation from text
async def create_text_video():
    response = await video_agent.run(
        "Create an 8-second video of a golden retriever playing in a sunflower field during golden hour"
    )
    print(f"[VIDEO_AGENT-create_text_video] {response}")
    return response

# Video generation from image
async def create_image_video():
    response = await video_agent.run(
        "Generate a video from the image at ./images/coffee_cup.jpg showing steam rising from hot coffee with soft morning lighting"
    )
    print(f"[VIDEO_AGENT-create_image_video] {response}")
    return response

# Advanced video with specific parameters
async def create_social_media_video():
    response = await video_agent.run("""
        Generate a vertical video for Instagram Stories showing:
        - A close-up of coffee being poured into a white ceramic mug
        - Steam rising from the hot coffee
        - Soft morning lighting
        - Use 9:16 aspect ratio for mobile
        - Avoid any text or logos in the video
        - Make it 6 seconds long
    """)
    print(f"[VIDEO_AGENT-create_social_media_video] {response}")
    return response
```

### 6. Error Handling & Retry Logic

```python
import asyncio
from typing import Union

async def robust_video_generation(
    agent: Agent,
    prompt: str,
    max_retries: int = 3,
    base_delay: float = 5.0
) -> Union[Dict[str, Any], None]:
    """
    Generate video with retry logic for handling rate limits and temporary failures.

    Args:
        agent: The Strands video generation agent
        prompt: The video generation prompt
        max_retries: Maximum number of retry attempts
        base_delay: Base delay in seconds for exponential backoff

    Returns:
        Generation result or None if all retries failed
    """
    for attempt in range(max_retries):
        try:
            result = await agent.run(prompt)
            print(f"[VIDEO_AGENT-robust_video_generation] Success on attempt {attempt + 1}")
            return result

        except Exception as e:
            print(f"[VIDEO_AGENT-robust_video_generation] Attempt {attempt + 1} failed: {str(e)}")

            if attempt < max_retries - 1:
                # Exponential backoff
                delay = base_delay * (2 ** attempt)
                print(f"[VIDEO_AGENT-robust_video_generation] Retrying in {delay} seconds...")
                await asyncio.sleep(delay)
            else:
                print(f"[VIDEO_AGENT-robust_video_generation] All {max_retries} attempts failed")
                return None

    return None
```

### 7. Integration with Existing Codebase

```python
# main.py - Example integration
import asyncio
from agent.video_generation_agent import video_agent, robust_video_generation

async def main():
    """Main entry point for video generation testing"""

    # Test basic video generation
    print("[MAIN] Testing basic video generation...")
    result = await create_text_video()

    # Test robust video generation with retry logic
    print("[MAIN] Testing robust video generation...")
    robust_result = await robust_video_generation(
        video_agent,
        "A peaceful mountain landscape with flowing water at sunset",
        max_retries=3
    )

    if robust_result:
        print(f"[MAIN] Video generation successful: {robust_result}")
    else:
        print("[MAIN] Video generation failed after all retries")

if __name__ == "__main__":
    asyncio.run(main())
```

## Key Benefits of This Implementation

- **Python-Based**: Proper Strands SDK implementation in Python
- **Simplified Auth**: Google AI Studio only needs an API key
- **Cost Effective**: VEO3 Fast costs $0.40/second with audio vs $0.75/second for standard
- **Clean Architecture**: Uses Strands @tool decorator for proper tool integration
- **Error Recovery**: Built-in retry logic for rate limits and failures
- **Type Safety**: Full type hints for better development experience
- **Logging**: Consistent logging format following user guidelines

## Model Capabilities

### VEO3 Models Available:
- `veo-3.0-generate-preview`: Standard VEO3 model
- `veo-3.0-fast-generate-preview`: Faster, more cost-effective version

### Supported Features:
- **Video from Text**: Generate 8-second videos from text descriptions
- **Video from Image**: Create videos using input images as starting frames
- **Multiple Formats**: Support for 16:9, 9:16, and 1:1 aspect ratios
- **Resolutions**: 720p and 1080p output options
- **Negative Prompts**: Specify what to avoid in generated content

## Reference Documentation

### Strands Documents

**LiteLLM**: https://strandsagents.com/latest/documentation/docs/user-guide/concepts/model-providers/litellm/

**Strands Lite LLM GitHub**: `/Users/benjamincorbett/code/aws/strands/samples/01-tutorials/01-fundamentals/02-model-providers/02-openai-model`

### Vertex AI Documents

- [Generate videos from text](https://cloud.google.com/vertex-ai/generative-ai/docs/video/generate-videos-from-text#googlegenaisdk_videogen_with_txt-python_genai_sdk)
- [Generate videos from an image](https://cloud.google.com/vertex-ai/generative-ai/docs/video/generate-videos-from-an-image)


