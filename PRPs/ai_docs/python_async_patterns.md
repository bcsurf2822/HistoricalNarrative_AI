# Python Async/Await Patterns and Best Practices for Long-Running Operations

## Table of Contents
1. [Core Concepts and Best Practices](#core-concepts-and-best-practices)
2. [Async Retry Patterns with Exponential Backoff](#async-retry-patterns-with-exponential-backoff)
3. [Async Context Manager Patterns](#async-context-manager-patterns)
4. [Timeout Handling with AsyncIO](#timeout-handling-with-asyncio)
5. [Error Handling and Logging Patterns](#error-handling-and-logging-patterns)
6. [Common Anti-Patterns to Avoid](#common-anti-patterns-to-avoid)
7. [Resource Management and Cleanup](#resource-management-and-cleanup)
8. [Production-Ready Examples](#production-ready-examples)

## Core Concepts and Best Practices

### 1. Handling CPU-Bound Operations

**Problem**: Blocking (CPU-bound) code should not be called directly in async functions.

**Solution**: Use executors for CPU-intensive tasks:

```python
import asyncio
import concurrent.futures
from concurrent.futures import ThreadPoolExecutor

async def cpu_intensive_operation(data):
    """Offload CPU-bound work to thread pool."""
    loop = asyncio.get_event_loop()

    def blocking_operation(data):
        # CPU-intensive work here
        return process_data(data)

    with ThreadPoolExecutor() as executor:
        result = await loop.run_in_executor(executor, blocking_operation, data)
    return result
```

### 2. Concurrency Control with Semaphores

**Pattern**: Limit concurrent operations to prevent resource exhaustion:

```python
import asyncio

# Limit to 100 concurrent operations
semaphore = asyncio.Semaphore(100)

async def controlled_operation(url):
    async with semaphore:
        async with aiohttp.ClientSession() as session:
            async with session.get(url) as response:
                return await response.text()
```

### 3. Task Management Best Practices

**Correct Pattern**: Use `asyncio.create_task()` for parallel execution:

```python
async def process_multiple_items(items):
    # Create tasks for parallel execution
    tasks = [asyncio.create_task(process_item(item)) for item in items]

    # Wait for all tasks to complete
    results = await asyncio.gather(*tasks, return_exceptions=True)

    # Handle results and exceptions
    successful_results = []
    for i, result in enumerate(results):
        if isinstance(result, Exception):
            logger.error(f"Task {i} failed: {result}")
        else:
            successful_results.append(result)

    return successful_results
```

### 4. Event Loop Optimization

**Performance Tip**: Use uvloop for 2-4x performance improvement:

```python
import asyncio
import uvloop

# Single line for significant performance boost
asyncio.set_event_loop_policy(uvloop.EventLoopPolicy())

async def main():
    # Your async application code here
    pass

if __name__ == "__main__":
    asyncio.run(main())
```

## Async Retry Patterns with Exponential Backoff

### Using Tenacity Library (Recommended)

```python
import asyncio
from tenacity import (
    retry,
    stop_after_attempt,
    wait_exponential,
    retry_if_exception_type,
    before_sleep_log
)
import logging

logger = logging.getLogger(__name__)

@retry(
    stop=stop_after_attempt(5),
    wait=wait_exponential(multiplier=1, min=4, max=10),
    retry=retry_if_exception_type((aiohttp.ClientError, asyncio.TimeoutError)),
    before_sleep=before_sleep_log(logger, logging.WARNING)
)
async def robust_api_call(url, payload):
    """API call with exponential backoff retry."""
    timeout = aiohttp.ClientTimeout(total=30)

    async with aiohttp.ClientSession(timeout=timeout) as session:
        async with session.post(url, json=payload) as response:
            response.raise_for_status()
            return await response.json()
```

### Using Backoff Library

```python
import backoff
import aiohttp

@backoff.on_exception(
    backoff.expo,
    aiohttp.ClientError,
    max_time=60,
    jitter=backoff.full_jitter  # Prevents thundering herd
)
async def resilient_request(url):
    """Request with backoff and jitter."""
    async with aiohttp.ClientSession(raise_for_status=True) as session:
        async with session.get(url) as response:
            return await response.text()
```

### Custom Retry Implementation

```python
import asyncio
import random
from typing import Callable, TypeVar, Any

T = TypeVar('T')

async def retry_with_backoff(
    coro_func: Callable[..., T],
    max_retries: int = 5,
    base_delay: float = 1.0,
    max_delay: float = 60.0,
    exponential_base: float = 2.0,
    jitter: bool = True,
    retryable_exceptions: tuple = (Exception,)
) -> T:
    """
    Retry an async function with exponential backoff and jitter.

    Args:
        coro_func: Async function to retry
        max_retries: Maximum number of retry attempts
        base_delay: Base delay in seconds
        max_delay: Maximum delay in seconds
        exponential_base: Base for exponential backoff
        jitter: Whether to add random jitter
        retryable_exceptions: Tuple of exceptions to retry on
    """
    last_exception = None

    for attempt in range(max_retries + 1):
        try:
            return await coro_func()
        except retryable_exceptions as e:
            last_exception = e

            if attempt == max_retries:
                raise last_exception

            # Calculate delay with exponential backoff
            delay = min(base_delay * (exponential_base ** attempt), max_delay)

            # Add jitter to prevent thundering herd
            if jitter:
                delay = delay * (0.5 + random.random() * 0.5)

            logger.warning(f"Attempt {attempt + 1} failed: {e}. Retrying in {delay:.2f}s")
            await asyncio.sleep(delay)

    raise last_exception
```

## Async Context Manager Patterns

### Using @asynccontextmanager Decorator

```python
from contextlib import asynccontextmanager
import aiohttp
import logging

logger = logging.getLogger(__name__)

@asynccontextmanager
async def http_session_manager(timeout: int = 30):
    """Async context manager for HTTP session with proper cleanup."""
    timeout_config = aiohttp.ClientTimeout(total=timeout)
    connector = aiohttp.TCPConnector(limit=100, limit_per_host=30)

    session = aiohttp.ClientSession(
        timeout=timeout_config,
        connector=connector
    )

    try:
        logger.info("[HTTP_SESSION-__aenter__] Created HTTP session")
        yield session
    except Exception as e:
        logger.error(f"[HTTP_SESSION-__aexit__] Error in session: {e}")
        raise
    finally:
        logger.info("[HTTP_SESSION-__aexit__] Cleaning up HTTP session")
        await session.close()
        await connector.close()
```

### Class-Based Async Context Manager

```python
import asyncio
import aiopg

class DatabaseConnectionManager:
    """Production-ready database connection manager."""

    def __init__(self, dsn: str, pool_size: int = 10):
        self.dsn = dsn
        self.pool_size = pool_size
        self.pool = None
        self.connection = None

    async def __aenter__(self):
        logger.info("[DATABASE-__aenter__] Initializing database connection")
        try:
            # Create connection pool
            self.pool = await aiopg.create_pool(
                dsn=self.dsn,
                minsize=1,
                maxsize=self.pool_size
            )

            # Get connection from pool
            self.connection = await self.pool.acquire()
            return self.connection

        except Exception as e:
            logger.error(f"[DATABASE-__aenter__] Failed to connect: {e}")
            await self._cleanup()
            raise

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        logger.info("[DATABASE-__aexit__] Cleaning up database connection")
        await self._cleanup()

        if exc_type:
            logger.error(f"[DATABASE-__aexit__] Exception occurred: {exc_val}")
            return False  # Don't suppress exceptions

        return True

    async def _cleanup(self):
        """Protected cleanup method to prevent cancellation issues."""
        try:
            if self.connection:
                await self.pool.release(self.connection)
                self.connection = None

            if self.pool:
                self.pool.close()
                await self.pool.wait_closed()
                self.pool = None

        except Exception as e:
            logger.error(f"[DATABASE-_cleanup] Error during cleanup: {e}")
```

### Cancellation-Safe Resource Management

```python
import asyncio
from contextlib import asynccontextmanager

@asynccontextmanager
async def cancellation_safe_resource():
    """Context manager that protects cleanup from cancellation."""
    resource = await acquire_resource()

    try:
        yield resource
    finally:
        # Protect cleanup from cancellation
        try:
            # Use shield to protect critical cleanup
            await asyncio.shield(release_resource(resource))
        except asyncio.CancelledError:
            # Log but don't re-raise during cleanup
            logger.warning("[RESOURCE-__aexit__] Cleanup cancelled, resource may leak")
        except Exception as e:
            logger.error(f"[RESOURCE-__aexit__] Error during cleanup: {e}")
```

## Timeout Handling with AsyncIO

### Modern Timeout Patterns (Python 3.11+)

```python
import asyncio

async def robust_operation_with_timeout():
    """Modern timeout handling with asyncio.timeout()."""
    try:
        async with asyncio.timeout(10.0):
            result = await long_running_operation()
            return result
    except TimeoutError:
        logger.warning("[OPERATION-timeout] Operation timed out after 10 seconds")
        return None
    except Exception as e:
        logger.error(f"[OPERATION-error] Operation failed: {e}")
        raise
```

### aiohttp Timeout Configuration

```python
import aiohttp

async def configure_http_timeouts():
    """Production-ready HTTP timeout configuration."""

    # Granular timeout configuration
    timeout = aiohttp.ClientTimeout(
        total=60,           # Total timeout for the entire request
        connect=10,         # Connection timeout
        sock_read=30,       # Socket read timeout
        sock_connect=10     # Socket connection timeout
    )

    # Connection pooling configuration
    connector = aiohttp.TCPConnector(
        limit=100,              # Total connection pool size
        limit_per_host=30,      # Max connections per host
        ttl_dns_cache=300,      # DNS cache TTL
        use_dns_cache=True,
    )

    async with aiohttp.ClientSession(
        timeout=timeout,
        connector=connector
    ) as session:

        try:
            async with session.get('https://api.example.com/data') as response:
                return await response.json()

        except aiohttp.ClientTimeout as e:
            logger.error(f"[HTTP-timeout] Request timed out: {e}")
            raise
        except aiohttp.ClientError as e:
            logger.error(f"[HTTP-error] Client error: {e}")
            raise
```

### Timeout with Graceful Cancellation

```python
async def graceful_timeout_operation(timeout_seconds: float = 30.0):
    """Operation with graceful timeout and cleanup."""

    async def cleanup_task():
        """Cleanup coroutine for timeout scenarios."""
        logger.info("[CLEANUP-start] Starting graceful cleanup")
        # Perform cleanup operations
        await cleanup_resources()
        logger.info("[CLEANUP-complete] Graceful cleanup completed")

    try:
        async with asyncio.timeout(timeout_seconds):
            return await main_operation()

    except TimeoutError:
        logger.warning(f"[OPERATION-timeout] Operation timed out after {timeout_seconds}s")

        # Perform cleanup with its own timeout
        try:
            async with asyncio.timeout(10.0):  # Cleanup timeout
                await cleanup_task()
        except TimeoutError:
            logger.error("[CLEANUP-timeout] Cleanup timed out")

        raise
```

## Error Handling and Logging Patterns

### Async Logging with QueueHandler (Recommended)

```python
import asyncio
import logging
import queue
from logging.handlers import QueueHandler, QueueListener
import threading

def setup_async_logging():
    """Set up non-blocking async logging."""

    # Create a queue for log records
    log_queue = queue.Queue()

    # Create handlers for actual log processing
    file_handler = logging.FileHandler('app.log')
    console_handler = logging.StreamHandler()

    # Format handlers
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - [%(funcName)s:%(lineno)d] - %(message)s'
    )
    file_handler.setFormatter(formatter)
    console_handler.setFormatter(formatter)

    # Create queue listener to process logs in separate thread
    queue_listener = QueueListener(
        log_queue,
        file_handler,
        console_handler,
        respect_handler_level=True
    )

    # Create non-blocking queue handler
    queue_handler = QueueHandler(log_queue)

    # Configure root logger
    root_logger = logging.getLogger()
    root_logger.setLevel(logging.INFO)
    root_logger.addHandler(queue_handler)

    # Start the listener
    queue_listener.start()

    return queue_listener

# Usage in async application
async def main():
    """Main async application with proper logging."""
    queue_listener = setup_async_logging()
    logger = logging.getLogger(__name__)

    try:
        logger.info("[MAIN-start] Starting async application")
        await run_application()
        logger.info("[MAIN-complete] Application completed successfully")

    except Exception as e:
        logger.error(f"[MAIN-error] Application failed: {e}", exc_info=True)
        raise
    finally:
        logger.info("[MAIN-shutdown] Shutting down logging")
        queue_listener.stop()
```

### Structured Error Handling Pattern

```python
import asyncio
import logging
from dataclasses import dataclass
from typing import Optional, Dict, Any
from enum import Enum

class ErrorType(Enum):
    NETWORK_ERROR = "network_error"
    TIMEOUT_ERROR = "timeout_error"
    VALIDATION_ERROR = "validation_error"
    RESOURCE_ERROR = "resource_error"
    UNKNOWN_ERROR = "unknown_error"

@dataclass
class OperationResult:
    """Structured result for async operations."""
    success: bool
    data: Optional[Any] = None
    error_type: Optional[ErrorType] = None
    error_message: Optional[str] = None
    retry_after: Optional[float] = None
    metadata: Dict[str, Any] = None

    def __post_init__(self):
        if self.metadata is None:
            self.metadata = {}

async def safe_async_operation(
    operation_func,
    operation_name: str,
    **kwargs
) -> OperationResult:
    """Wrapper for safe async operations with structured error handling."""

    logger = logging.getLogger(__name__)
    start_time = asyncio.get_event_loop().time()

    try:
        logger.info(f"[{operation_name}-start] Starting operation with args: {kwargs}")

        result = await operation_func(**kwargs)

        duration = asyncio.get_event_loop().time() - start_time
        logger.info(f"[{operation_name}-success] Operation completed in {duration:.2f}s")

        return OperationResult(
            success=True,
            data=result,
            metadata={"duration": duration}
        )

    except asyncio.TimeoutError as e:
        duration = asyncio.get_event_loop().time() - start_time
        logger.warning(f"[{operation_name}-timeout] Operation timed out after {duration:.2f}s")

        return OperationResult(
            success=False,
            error_type=ErrorType.TIMEOUT_ERROR,
            error_message=str(e),
            retry_after=min(duration * 2, 60),  # Exponential backoff
            metadata={"duration": duration}
        )

    except aiohttp.ClientError as e:
        duration = asyncio.get_event_loop().time() - start_time
        logger.error(f"[{operation_name}-network_error] Network error: {e}")

        return OperationResult(
            success=False,
            error_type=ErrorType.NETWORK_ERROR,
            error_message=str(e),
            retry_after=5.0,
            metadata={"duration": duration}
        )

    except Exception as e:
        duration = asyncio.get_event_loop().time() - start_time
        logger.error(f"[{operation_name}-error] Unexpected error: {e}", exc_info=True)

        return OperationResult(
            success=False,
            error_type=ErrorType.UNKNOWN_ERROR,
            error_message=str(e),
            metadata={"duration": duration}
        )
```

## Common Anti-Patterns to Avoid

### ❌ Blocking Operations in Async Code

```python
# DON'T DO THIS
async def bad_example():
    time.sleep(1)  # Blocks entire event loop!
    response = requests.get('http://example.com')  # Blocking I/O!
    return response.json()

# ✅ DO THIS INSTEAD
async def good_example():
    await asyncio.sleep(1)  # Non-blocking
    async with aiohttp.ClientSession() as session:
        async with session.get('http://example.com') as response:
            return await response.json()
```

### ❌ Resource Leaks

```python
# DON'T DO THIS - Resource leak
async def bad_resource_handling():
    session = aiohttp.ClientSession()
    response = await session.get('http://example.com')
    return await response.json()  # Session never closed!

# ✅ DO THIS INSTEAD
async def good_resource_handling():
    async with aiohttp.ClientSession() as session:
        async with session.get('http://example.com') as response:
            return await response.json()
```

### ❌ Mixed Sync/Async Patterns

```python
# DON'T DO THIS - Inefficient
async def bad_mixed_pattern():
    result1 = await operation1()  # Wait for first
    result2 = await operation2()  # Then wait for second
    return result1, result2

# ✅ DO THIS INSTEAD - Parallel execution
async def good_parallel_pattern():
    task1 = asyncio.create_task(operation1())
    task2 = asyncio.create_task(operation2())
    return await asyncio.gather(task1, task2)
```

### ❌ Uncontrolled Concurrency

```python
# DON'T DO THIS - Can overwhelm system
async def bad_concurrency(urls):
    tasks = [fetch_url(url) for url in urls]  # All at once!
    return await asyncio.gather(*tasks)

# ✅ DO THIS INSTEAD - Controlled concurrency
async def good_concurrency(urls, max_concurrent=10):
    semaphore = asyncio.Semaphore(max_concurrent)

    async def limited_fetch(url):
        async with semaphore:
            return await fetch_url(url)

    tasks = [limited_fetch(url) for url in urls]
    return await asyncio.gather(*tasks)
```

## Resource Management and Cleanup

### Comprehensive Resource Manager

```python
import asyncio
import aiofiles
import aioredis
from contextlib import AsyncExitStack

class ResourceManager:
    """Comprehensive async resource manager."""

    def __init__(self):
        self.resources = {}
        self.cleanup_tasks = []

    async def __aenter__(self):
        logger.info("[RESOURCE_MANAGER-start] Initializing resources")
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        logger.info("[RESOURCE_MANAGER-cleanup] Starting resource cleanup")

        # Run all cleanup tasks
        if self.cleanup_tasks:
            await asyncio.gather(*self.cleanup_tasks, return_exceptions=True)

        # Clear resources
        self.resources.clear()
        logger.info("[RESOURCE_MANAGER-complete] Resource cleanup completed")

    async def add_http_session(self, name: str, **kwargs):
        """Add managed HTTP session."""
        session = aiohttp.ClientSession(**kwargs)
        self.resources[name] = session

        # Register cleanup
        async def cleanup():
            await session.close()

        self.cleanup_tasks.append(cleanup())
        return session

    async def add_redis_connection(self, name: str, url: str):
        """Add managed Redis connection."""
        redis = await aioredis.from_url(url)
        self.resources[name] = redis

        # Register cleanup
        async def cleanup():
            await redis.close()

        self.cleanup_tasks.append(cleanup())
        return redis

    def get_resource(self, name: str):
        """Get managed resource by name."""
        return self.resources.get(name)

# Usage example
async def application_with_resources():
    async with ResourceManager() as rm:
        # Initialize resources
        http_session = await rm.add_http_session(
            'main_session',
            timeout=aiohttp.ClientTimeout(total=30)
        )

        redis = await rm.add_redis_connection(
            'cache',
            'redis://localhost:6379'
        )

        # Use resources
        await do_work_with_resources(http_session, redis)

        # Automatic cleanup on exit
```

## Production-Ready Examples

### Video Generation Agent Pattern

```python
import asyncio
import aiohttp
from tenacity import retry, stop_after_attempt, wait_exponential
from contextlib import asynccontextmanager
import logging

logger = logging.getLogger(__name__)

class VideoGenerationAgent:
    """Production-ready async video generation agent."""

    def __init__(self, api_base_url: str, max_concurrent: int = 5):
        self.api_base_url = api_base_url
        self.semaphore = asyncio.Semaphore(max_concurrent)
        self.session = None

    @asynccontextmanager
    async def managed_session(self):
        """Context manager for HTTP session lifecycle."""
        timeout = aiohttp.ClientTimeout(
            total=300,      # 5 minutes for video generation
            connect=10,     # 10 seconds to connect
            sock_read=60    # 60 seconds to read response
        )

        connector = aiohttp.TCPConnector(
            limit=20,
            limit_per_host=10,
            ttl_dns_cache=300
        )

        session = aiohttp.ClientSession(
            timeout=timeout,
            connector=connector
        )

        try:
            logger.info("[VIDEO_AGENT-session] Created HTTP session")
            yield session
        finally:
            logger.info("[VIDEO_AGENT-session] Closing HTTP session")
            await session.close()
            await connector.close()

    @retry(
        stop=stop_after_attempt(3),
        wait=wait_exponential(multiplier=2, min=4, max=30),
        retry_error_callback=lambda retry_state: logger.warning(
            f"[VIDEO_AGENT-retry] Attempt {retry_state.attempt_number} failed"
        )
    )
    async def generate_video(self, prompt: str, video_id: str) -> dict:
        """Generate video with retry logic and proper error handling."""

        async with self.semaphore:  # Rate limiting
            async with self.managed_session() as session:

                payload = {
                    "prompt": prompt,
                    "video_id": video_id,
                    "quality": "high",
                    "duration": 30
                }

                try:
                    logger.info(f"[VIDEO_AGENT-generate] Starting generation for {video_id}")

                    async with session.post(
                        f"{self.api_base_url}/generate",
                        json=payload
                    ) as response:

                        response.raise_for_status()
                        result = await response.json()

                        logger.info(f"[VIDEO_AGENT-success] Generated video {video_id}")
                        return result

                except aiohttp.ClientTimeout:
                    logger.error(f"[VIDEO_AGENT-timeout] Generation timed out for {video_id}")
                    raise

                except aiohttp.ClientError as e:
                    logger.error(f"[VIDEO_AGENT-error] Client error for {video_id}: {e}")
                    raise

    async def process_batch(self, video_requests: list) -> list:
        """Process multiple video generation requests concurrently."""

        logger.info(f"[VIDEO_AGENT-batch] Processing {len(video_requests)} requests")

        tasks = [
            asyncio.create_task(
                self.generate_video(req['prompt'], req['video_id'])
            )
            for req in video_requests
        ]

        results = await asyncio.gather(*tasks, return_exceptions=True)

        # Process results and separate successes from failures
        successful = []
        failed = []

        for i, result in enumerate(results):
            if isinstance(result, Exception):
                failed.append({
                    'video_id': video_requests[i]['video_id'],
                    'error': str(result)
                })
                logger.error(f"[VIDEO_AGENT-batch_error] Failed: {result}")
            else:
                successful.append(result)

        logger.info(
            f"[VIDEO_AGENT-batch_complete] "
            f"Successful: {len(successful)}, Failed: {len(failed)}"
        )

        return {
            'successful': successful,
            'failed': failed,
            'total_processed': len(video_requests)
        }

# Usage example
async def main():
    """Main application entry point."""

    # Setup async logging
    queue_listener = setup_async_logging()

    try:
        agent = VideoGenerationAgent(
            api_base_url="https://api.videoservice.com",
            max_concurrent=3
        )

        video_requests = [
            {"prompt": "A cat playing piano", "video_id": "video_1"},
            {"prompt": "Sunset over mountains", "video_id": "video_2"},
            {"prompt": "Dancing robot", "video_id": "video_3"},
        ]

        results = await agent.process_batch(video_requests)

        print(f"Generated {len(results['successful'])} videos successfully")

        if results['failed']:
            print(f"Failed to generate {len(results['failed'])} videos")

    finally:
        queue_listener.stop()

if __name__ == "__main__":
    # Use uvloop for better performance
    try:
        import uvloop
        asyncio.set_event_loop_policy(uvloop.EventLoopPolicy())
    except ImportError:
        pass  # uvloop not available

    asyncio.run(main())
```

## Key Takeaways for Production Use

1. **Always use context managers** for resource management
2. **Implement proper retry logic** with exponential backoff and jitter
3. **Use semaphores** to control concurrency and prevent resource exhaustion
4. **Set up non-blocking logging** with QueueHandler/QueueListener
5. **Handle timeouts gracefully** with proper cleanup
6. **Use structured error handling** for better debugging and monitoring
7. **Avoid blocking operations** in async code - use thread pools for CPU-bound work
8. **Monitor and log** all async operations with proper context
9. **Test thoroughly** with realistic loads and failure scenarios
10. **Use established libraries** (tenacity, aiohttp) rather than rolling your own

## Additional Resources

- [Python AsyncIO Documentation](https://docs.python.org/3/library/asyncio.html)
- [Tenacity Retry Library](https://github.com/jd/tenacity)
- [aiohttp Documentation](https://docs.aiohttp.org/)
- [uvloop Performance](https://github.com/MagicStack/uvloop)
- [Async Best Practices Guide](https://superfastpython.com/asyncio-best-practices/)