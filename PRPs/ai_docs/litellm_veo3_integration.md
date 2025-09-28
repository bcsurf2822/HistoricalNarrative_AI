# LiteLLM VEO3 Integration Guide

## Overview

This document provides comprehensive guidance for integrating Google's VEO3 video generation model with LiteLLM proxy for Strands agents. VEO3 is Google's latest AI video generation model that produces high-fidelity videos with native audio integration, supporting both text-to-video and image-to-video workflows.

## Official Documentation Links

### Primary Resources
- **LiteLLM VEO Video Generation**: https://docs.litellm.ai/docs/proxy/veo_video_generation
- **Google AI Studio SDK**: https://docs.litellm.ai/docs/pass_through/google_ai_studio
- **VEO3 in Gemini API**: https://ai.google.dev/gemini-api/docs/video
- **Google AI Studio VEO3 Model**: https://aistudio.google.com/models/veo-3
- **Vertex AI VEO Documentation**: https://cloud.google.com/vertex-ai/generative-ai/docs/model-reference/veo-video-generation

### Configuration and Setup
- **LiteLLM Configuration Overview**: https://docs.litellm.ai/docs/proxy/configs
- **Setting API Keys**: https://docs.litellm.ai/docs/set_keys
- **Google AI Studio Authentication**: https://ai.google.dev/gemini-api/docs/api-key
- **Production Best Practices**: https://docs.litellm.ai/docs/proxy/prod

### Rate Limits and Quotas
- **Gemini API Rate Limits**: https://ai.google.dev/gemini-api/docs/rate-limits
- **Vertex AI Quotas**: https://cloud.google.com/vertex-ai/generative-ai/docs/quotas

## Environment Setup

### Required Environment Variables

```bash
# Primary API key (GOOGLE_API_KEY takes precedence over GEMINI_API_KEY)
export GOOGLE_API_KEY="your-google-ai-studio-api-key"
# Alternative option
export GEMINI_API_KEY="your-google-ai-studio-api-key"

# LiteLLM Proxy settings
export LITELLM_MASTER_KEY="sk-your-master-key"
export LITELLM_PORT="4000"
```

### Getting Google AI Studio API Key

1. Visit Google AI Studio API Keys page: https://aistudio.google.com/app/apikey
2. Create a new API key for your project
3. Store the key securely and never commit to version control
4. Set the environment variable before starting your application

## LiteLLM Proxy Configuration

### Basic config.yaml Setup

```yaml
model_list:
  - model_name: veo-3-video
    litellm_params:
      model: google_ai_studio/veo-3.0-generate-preview
      api_key: os.environ/GOOGLE_API_KEY
      api_base: https://generativelanguage.googleapis.com

# General proxy settings
general_settings:
  master_key: os.environ/LITELLM_MASTER_KEY
  proxy_batch_write_at: 60

# Router settings for reliability
router_settings:
  routing_strategy: simple-shuffle
  timeout: 300  # Extended timeout for video generation

# Error handling and retry configuration
litellm_settings:
  drop_params: true
  set_verbose: false
  json_logs: true

# Health check configuration
health_check_details:
  healthy_endpoints:
    - /health/liveliness
    - /health/readiness
```

### Production Configuration

```yaml
model_list:
  - model_name: veo-3-video-prod
    litellm_params:
      model: google_ai_studio/veo-3.0-generate-preview
      api_key: os.environ/GOOGLE_API_KEY
      api_base: https://generativelanguage.googleapis.com
      timeout: 600  # 10 minutes for long video generation

# Enhanced logging for production
litellm_settings:
  success_callback: ["langfuse", "prometheus"]
  failure_callback: ["slack", "langfuse"]
  cache: true
  cache_params:
    type: "redis"

# Security settings
general_settings:
  master_key: os.environ/LITELLM_MASTER_KEY
  ui_access_mode: "admin_only"
  allow_user_auth: true

# Rate limiting
router_settings:
  redis_host: os.environ/REDIS_HOST
  redis_password: os.environ/REDIS_PASSWORD
  enable_pre_call_checks: true
```

## Video Generation Implementation

### Basic Video Generation with LiteLLM

```python
import requests
import time
import json
from typing import Dict, Any, Optional

class VEO3VideoGenerator:
    def __init__(self, proxy_base_url: str = "http://localhost:4000", api_key: str = "anything"):
        self.base_url = f"{proxy_base_url}/gemini/v1beta"
        self.api_key = api_key
        self.headers = {
            "x-goog-api-key": self.api_key,
            "Content-Type": "application/json"
        }

    def generate_video(self, prompt: str, duration: int = 8) -> Dict[str, Any]:
        """
        Generate video using VEO3 model

        Args:
            prompt: Text description for video generation
            duration: Video duration in seconds (max 8 for current VEO3)

        Returns:
            Dict containing operation details for polling
        """
        url = f"{self.base_url}/models/veo-3.0-generate-preview:predictLongRunning"

        payload = {
            "instances": [{
                "prompt": prompt,
                "config": {
                    "duration": f"{duration}s"
                }
            }]
        }

        try:
            response = requests.post(url, headers=self.headers, json=payload, timeout=30)
            response.raise_for_status()
            return response.json()

        except requests.RequestException as e:
            print(f"[VEO3VideoGenerator-generate_video] Error generating video: {e}")
            raise

    def poll_operation(self, operation_name: str, max_wait_time: int = 600) -> Optional[Dict[str, Any]]:
        """
        Poll for video generation completion

        Args:
            operation_name: Operation identifier from generate_video response
            max_wait_time: Maximum time to wait in seconds

        Returns:
            Video data when complete, None if timeout
        """
        poll_url = f"{self.base_url}/operations/{operation_name}"
        start_time = time.time()

        while time.time() - start_time < max_wait_time:
            try:
                response = requests.get(poll_url, headers=self.headers, timeout=10)
                response.raise_for_status()
                result = response.json()

                if result.get("done"):
                    print(f"[VEO3VideoGenerator-poll_operation] Video generation completed")
                    return result.get("response", {})

                print(f"[VEO3VideoGenerator-poll_operation] Still generating... waiting 10s")
                time.sleep(10)

            except requests.RequestException as e:
                print(f"[VEO3VideoGenerator-poll_operation] Polling error: {e}")
                time.sleep(5)

        print(f"[VEO3VideoGenerator-poll_operation] Timeout after {max_wait_time}s")
        return None

    def generate_and_wait(self, prompt: str, duration: int = 8) -> Optional[str]:
        """
        Generate video and wait for completion

        Returns:
            Video URI when ready, None if failed
        """
        try:
            # Start generation
            operation = self.generate_video(prompt, duration)
            operation_name = operation.get("name")

            if not operation_name:
                print(f"[VEO3VideoGenerator-generate_and_wait] No operation name received")
                return None

            # Poll for completion
            result = self.poll_operation(operation_name)

            if result and "predictions" in result:
                video_uri = result["predictions"][0].get("generated_video", {}).get("uri")
                return video_uri

            return None

        except Exception as e:
            print(f"[VEO3VideoGenerator-generate_and_wait] Generation failed: {e}")
            return None
```

### Advanced Configuration with Strands Integration

```python
import asyncio
from typing import List, Dict, Any
import aiohttp

class StrandsVEO3Agent:
    def __init__(self, litellm_config: Dict[str, Any]):
        self.config = litellm_config
        self.video_generator = VEO3VideoGenerator(
            proxy_base_url=litellm_config.get("proxy_url", "http://localhost:4000"),
            api_key=litellm_config.get("api_key", "anything")
        )

    async def generate_historical_video(self, narrative_prompt: str, context: Dict[str, Any]) -> Dict[str, Any]:
        """
        Generate historical narrative video with enhanced prompting

        Args:
            narrative_prompt: Base narrative description
            context: Historical context and metadata

        Returns:
            Generation result with video URI and metadata
        """
        # Enhance prompt with historical context
        enhanced_prompt = self._enhance_historical_prompt(narrative_prompt, context)

        try:
            # Generate video asynchronously
            video_uri = await asyncio.get_event_loop().run_in_executor(
                None,
                self.video_generator.generate_and_wait,
                enhanced_prompt,
                context.get("duration", 8)
            )

            return {
                "success": True,
                "video_uri": video_uri,
                "prompt": enhanced_prompt,
                "context": context,
                "timestamp": time.time()
            }

        except Exception as e:
            print(f"[StrandsVEO3Agent-generate_historical_video] Error: {e}")
            return {
                "success": False,
                "error": str(e),
                "prompt": enhanced_prompt,
                "context": context
            }

    def _enhance_historical_prompt(self, base_prompt: str, context: Dict[str, Any]) -> str:
        """Enhance prompt with historical context and VEO3 best practices"""

        # Historical context enhancement
        time_period = context.get("time_period", "")
        location = context.get("location", "")
        style = context.get("visual_style", "cinematic, historical documentary style")

        enhanced = f"{base_prompt}"

        if time_period:
            enhanced += f" Set in {time_period}"
        if location:
            enhanced += f" in {location}"

        # VEO3 optimization
        enhanced += f". {style}, high quality, detailed, realistic lighting"

        # Ensure prompt length is optimal (VEO3 works best with detailed but concise prompts)
        if len(enhanced) > 500:
            enhanced = enhanced[:500] + "..."

        return enhanced
```

## Error Handling and Best Practices

### Comprehensive Error Handling

```python
import logging
from enum import Enum
from dataclasses import dataclass
from typing import Optional, Dict, Any

class VEO3ErrorType(Enum):
    QUOTA_EXCEEDED = "quota_exceeded"
    INVALID_API_KEY = "invalid_api_key"
    CONTENT_POLICY_VIOLATION = "content_policy_violation"
    TIMEOUT = "timeout"
    RATE_LIMITED = "rate_limited"
    NETWORK_ERROR = "network_error"
    UNKNOWN = "unknown"

@dataclass
class VEO3Error:
    error_type: VEO3ErrorType
    message: str
    details: Optional[Dict[str, Any]] = None
    retry_after: Optional[int] = None

class VEO3ErrorHandler:
    def __init__(self):
        self.logger = logging.getLogger(__name__)

    def handle_response_error(self, response: requests.Response) -> VEO3Error:
        """Parse and categorize API response errors"""

        status_code = response.status_code

        try:
            error_data = response.json()
        except:
            error_data = {"message": response.text}

        # Quota exceeded (common with VEO3)
        if status_code == 429 or "quota" in error_data.get("message", "").lower():
            retry_after = response.headers.get("Retry-After")
            return VEO3Error(
                error_type=VEO3ErrorType.QUOTA_EXCEEDED,
                message="VEO3 quota exceeded. Try again later.",
                details=error_data,
                retry_after=int(retry_after) if retry_after else 3600
            )

        # Authentication errors
        if status_code in [401, 403]:
            return VEO3Error(
                error_type=VEO3ErrorType.INVALID_API_KEY,
                message="Invalid or missing API key for Google AI Studio",
                details=error_data
            )

        # Content policy violations
        if "policy" in error_data.get("message", "").lower():
            return VEO3Error(
                error_type=VEO3ErrorType.CONTENT_POLICY_VIOLATION,
                message="Content violates VEO3 usage policies",
                details=error_data
            )

        # Rate limiting
        if status_code == 429:
            return VEO3Error(
                error_type=VEO3ErrorType.RATE_LIMITED,
                message="Rate limit exceeded",
                details=error_data,
                retry_after=60
            )

        return VEO3Error(
            error_type=VEO3ErrorType.UNKNOWN,
            message=f"HTTP {status_code}: {error_data.get('message', 'Unknown error')}",
            details=error_data
        )

    def should_retry(self, error: VEO3Error) -> bool:
        """Determine if operation should be retried"""
        retry_errors = {
            VEO3ErrorType.NETWORK_ERROR,
            VEO3ErrorType.TIMEOUT,
            VEO3ErrorType.RATE_LIMITED
        }
        return error.error_type in retry_errors

    def get_retry_delay(self, error: VEO3Error, attempt: int) -> int:
        """Calculate retry delay with exponential backoff"""
        if error.retry_after:
            return error.retry_after

        base_delay = 2 ** attempt  # Exponential backoff
        return min(base_delay, 300)  # Max 5 minutes
```

### Robust Video Generation with Retry Logic

```python
class RobustVEO3Generator(VEO3VideoGenerator):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.error_handler = VEO3ErrorHandler()
        self.max_retries = 3

    def generate_video_with_retry(self, prompt: str, duration: int = 8) -> Dict[str, Any]:
        """Generate video with comprehensive error handling and retry logic"""

        for attempt in range(self.max_retries + 1):
            try:
                return self.generate_video(prompt, duration)

            except requests.RequestException as e:
                if hasattr(e, 'response') and e.response is not None:
                    error = self.error_handler.handle_response_error(e.response)
                else:
                    error = VEO3Error(
                        error_type=VEO3ErrorType.NETWORK_ERROR,
                        message=str(e)
                    )

                self.error_handler.logger.warning(
                    f"[RobustVEO3Generator-generate_video_with_retry] Attempt {attempt + 1} failed: {error.message}"
                )

                if attempt < self.max_retries and self.error_handler.should_retry(error):
                    delay = self.error_handler.get_retry_delay(error, attempt)
                    self.error_handler.logger.info(
                        f"[RobustVEO3Generator-generate_video_with_retry] Retrying in {delay} seconds..."
                    )
                    time.sleep(delay)
                    continue

                # Final attempt failed
                raise Exception(f"Video generation failed after {attempt + 1} attempts: {error.message}")

        raise Exception("Max retries exceeded")
```

## Rate Limits and Quota Management

### Current VEO3 Limitations (as of 2025)

- **Video Length**: Maximum 8 seconds per generation
- **Daily Quota**: Restrictive limits even for paid users during preview
- **Concurrent Requests**: Limited concurrent generation capacity
- **Quota Reset**: Daily quotas reset at midnight Pacific time

### Quota Management Strategy

```python
import time
from datetime import datetime, timezone
from typing import Dict, Optional

class VEO3QuotaManager:
    def __init__(self):
        self.daily_usage = 0
        self.last_reset = datetime.now(timezone.utc).date()
        self.quota_limit = 50  # Conservative estimate
        self.rate_limit_window = 60  # seconds
        self.requests_in_window = []

    def can_make_request(self) -> tuple[bool, Optional[str]]:
        """Check if request can be made within quota and rate limits"""

        # Reset daily counter if new day
        current_date = datetime.now(timezone.utc).date()
        if current_date > self.last_reset:
            self.daily_usage = 0
            self.last_reset = current_date

        # Check daily quota
        if self.daily_usage >= self.quota_limit:
            return False, f"Daily quota exceeded ({self.quota_limit})"

        # Check rate limiting
        now = time.time()
        self.requests_in_window = [
            req_time for req_time in self.requests_in_window
            if now - req_time < self.rate_limit_window
        ]

        if len(self.requests_in_window) >= 10:  # Max 10 requests per minute
            return False, "Rate limit exceeded (10 requests/minute)"

        return True, None

    def record_request(self):
        """Record successful request"""
        self.daily_usage += 1
        self.requests_in_window.append(time.time())

        print(f"[VEO3QuotaManager-record_request] Usage: {self.daily_usage}/{self.quota_limit}")
```

## Troubleshooting Guide

### Common Issues and Solutions

1. **Quota Exceeded Errors**
   - **Symptom**: "quota exceeded" or 429 status codes
   - **Solution**: Implement exponential backoff, check quota reset times
   - **Prevention**: Use quota management and request batching

2. **Authentication Failures**
   - **Symptom**: 401/403 errors, "invalid API key"
   - **Solution**: Verify GOOGLE_API_KEY environment variable
   - **Check**: Ensure API key has VEO access enabled

3. **Long Generation Times**
   - **Symptom**: Operations taking >10 minutes
   - **Solution**: Increase timeout values, implement proper polling
   - **Monitoring**: Add progress logging and health checks

4. **Content Policy Violations**
   - **Symptom**: Generation fails with policy error
   - **Solution**: Review and sanitize prompts
   - **Guidelines**: Follow Google's responsible AI guidelines

### Debugging Configuration

```yaml
# Add to config.yaml for debugging
litellm_settings:
  set_verbose: true
  debug: true
  log_level: "DEBUG"

# Enable request/response logging
general_settings:
  store_model_in_db: true
  proxy_batch_write_at: 1  # Immediate logging
```

### Health Check Implementation

```python
def health_check_veo3(proxy_url: str) -> Dict[str, Any]:
    """Comprehensive health check for VEO3 setup"""

    results = {
        "proxy_accessible": False,
        "api_key_valid": False,
        "veo3_available": False,
        "quota_available": False,
        "errors": []
    }

    try:
        # Check proxy accessibility
        response = requests.get(f"{proxy_url}/health", timeout=5)
        results["proxy_accessible"] = response.status_code == 200

        # Test VEO3 endpoint (dry run)
        generator = VEO3VideoGenerator(proxy_url)
        test_payload = {
            "instances": [{"prompt": "test", "config": {"duration": "1s"}}]
        }

        # This should return operation info without generating
        test_response = requests.post(
            f"{proxy_url}/gemini/v1beta/models/veo-3.0-generate-preview:predictLongRunning",
            headers=generator.headers,
            json=test_payload,
            timeout=10
        )

        if test_response.status_code == 200:
            results["api_key_valid"] = True
            results["veo3_available"] = True
            results["quota_available"] = True
        elif test_response.status_code == 429:
            results["api_key_valid"] = True
            results["veo3_available"] = True
            results["quota_available"] = False
            results["errors"].append("Quota exceeded")
        else:
            results["errors"].append(f"API error: {test_response.status_code}")

    except Exception as e:
        results["errors"].append(str(e))

    return results
```

## Production Deployment Checklist

### Pre-deployment
- [ ] Google AI Studio API key obtained and tested
- [ ] LiteLLM proxy configuration validated
- [ ] Environment variables properly set
- [ ] Health checks implemented
- [ ] Error handling and retry logic in place
- [ ] Quota management system configured
- [ ] Monitoring and logging setup

### Security Considerations
- [ ] API keys stored securely (not in code)
- [ ] Proxy access restricted (master key protection)
- [ ] Request validation implemented
- [ ] Content filtering enabled
- [ ] Rate limiting configured

### Monitoring Setup
- [ ] Success/failure callback handlers
- [ ] Video generation metrics tracking
- [ ] Quota usage monitoring
- [ ] Error rate alerting
- [ ] Performance metrics collection

## Additional Resources

### Community and Support
- **GitHub Repository**: https://github.com/BerriAI/litellm
- **Google AI Developers Forum**: https://discuss.ai.google.dev/
- **LiteLLM Discord**: https://discord.gg/wuPM9dRgDw

### Example Implementations
- **Google Cloud LiteLLM Proxy**: https://github.com/Cyclenerd/google-cloud-litellm-proxy
- **Comprehensive Config Guide**: https://dev.to/yigit-konur/comprehensive-litellm-configuration-guide-configyaml-with-all-options-included-3e65

### VEO3 Best Practices
- **Prompting Guide**: https://cloud.google.com/vertex-ai/generative-ai/docs/video/video-gen-prompt-guide
- **Responsible AI Guidelines**: https://cloud.google.com/vertex-ai/generative-ai/docs/video/responsible-ai-and-usage-guidelines

---

*This documentation is current as of September 2025. VEO3 is in preview and capabilities may change. Always refer to the latest official documentation for the most current information.*