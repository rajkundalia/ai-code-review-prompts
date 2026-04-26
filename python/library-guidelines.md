# Library Guidelines

Reference guide for library-specific patterns and gotchas. Not loaded automatically — consult when working with these libraries.

---

## `requests`

The #1 gotcha: `requests` has no default timeout and will hang forever. A second gotcha: `requests.Session` has no built-in mechanism to enforce a default timeout — setting `session.timeout = 10` is silently ignored. Enforce it via a thin wrapper:

```python
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

class BoundSession:
    """Thin wrapper that enforces a timeout on every request."""

    def __init__(self, *, retries: int = 3, timeout: float = 30.0) -> None:
        self.timeout = timeout
        self._session = requests.Session()
        retry = Retry(
            total=retries,
            backoff_factor=0.5,
            status_forcelist=[429, 500, 502, 503, 504],
            allowed_methods=["GET", "POST", "PUT", "DELETE"],
        )
        self._session.mount("https://", HTTPAdapter(max_retries=retry))

    def get(self, url: str, **kwargs: object) -> requests.Response:
        kwargs.setdefault("timeout", self.timeout)
        response = self._session.get(url, **kwargs)
        response.raise_for_status()
        return response
```

- Catch specific exceptions: `Timeout`, `ConnectionError`, `HTTPError` — not broad `RequestException`.
- Never call `requests.get()` inside `async def` — use `asyncio.to_thread(session.get, url)`.
- Never hardcode API keys or auth tokens. Use env vars or session-level auth headers.

### Testing `requests`

Inject a `BoundSession` mock rather than patching globally:

```python
@pytest.fixture
def mock_session(mocker):
    session = mocker.MagicMock(spec=BoundSession)
    response = mocker.MagicMock(ok=True, status_code=200)
    response.json.return_value = {"status": "ok"}
    response.raise_for_status.return_value = None
    session.get.return_value = response
    return session

def test_handles_timeout(mock_session):
    mock_session.get.side_effect = requests.Timeout("timed out")
    client = MyClient(session=mock_session)
    with pytest.raises(requests.Timeout):
        client.fetch("/data")
```

`spec=BoundSession` makes the mock reject calls to attributes that don't exist on the real class, catching interface drift at test time.

---

## `claude-agent-sdk`

- `query()` returns `AsyncIterator[Message]` — always consume with `async for`.
- Type-check messages: `isinstance(message, AssistantMessage | ResultMessage)`. Don't assume structure.
- Handle SDK exceptions explicitly: `CLINotFoundError`, `ProcessError`, `CLIJSONDecodeError`.
- Use `ClaudeAgentOptions` to restrict `allowed_tools`, set `permission_mode`, and set `system_prompt`.
- Custom tools: `@tool` decorator + `create_sdk_mcp_server()` for in-process MCP servers.
- Load project config with `setting_sources=["project"]`.
- `ANTHROPIC_API_KEY` via env var only.

```python
from claude_agent_sdk import (
    query,
    ClaudeAgentOptions,
    AssistantMessage,
    ResultMessage,
    ToolUseBlock,
    CLINotFoundError,
    ProcessError,
)

async def run_agent(prompt: str) -> str | None:
    options = ClaudeAgentOptions(
        allowed_tools=["Read", "Edit", "Glob"],
        permission_mode="acceptEdits",
    )
    try:
        async for message in query(prompt=prompt, options=options):
            if isinstance(message, ResultMessage):
                return message.result
    except CLINotFoundError:
        logger.error("Claude CLI not found")
        raise
    except ProcessError as e:
        logger.error("Agent failed: exit code %s", e.exit_code)
        raise
    return None
```

### Testing `claude-agent-sdk`

`asyncio_mode = "auto"` applies globally — no `@pytest.mark.asyncio` needed. Mock `query()` as an async generator:

```python
@pytest.fixture
def mock_agent_query(mocker):
    async def _mock(*args, **kwargs):
        from claude_agent_sdk import ResultMessage
        yield ResultMessage(result="mocked response")

    return mocker.patch("claude_agent_sdk.query", side_effect=_mock)

async def test_agent_returns_result(mock_agent_query):
    result = await run_agent("test prompt")
    assert result == "mocked response"
```
