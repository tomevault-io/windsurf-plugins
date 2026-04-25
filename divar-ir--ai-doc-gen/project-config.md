---
trigger: always_on
description: Guidelines for developing AI agents and tools using pydantic-ai
---


# AI Agent Development Guidelines

## Agent Architecture

### Multi-Agent Coordination

The system uses **concurrent multi-agent execution** with error isolation:

```python
# Example from AnalyzerAgent
async def run(self):
    agent_tasks = {}
    
    if not self._config.exclude_code_structure:
        agent_tasks["Structure"] = self._run_agent(
            agent=self._structure_analyzer_agent,
            user_prompt=self._render_prompt("agents.structure_analyzer.user_prompt"),
            file_path=self._config.repo_path / ".ai" / "docs" / "structure_analysis.md",
        )
    
    # Run all agents concurrently
    results = await asyncio.gather(*agent_tasks.values(), return_exceptions=True)
    
    # Validate partial success
    self.validate_succession(analysis_files)
```

### Agent Configuration Pattern

```python
class MyAgentConfig(BaseModel):
    repo_path: Path = Field(..., description="Repository path")
    exclude_feature: bool = Field(default=False, description="Exclude feature")
    custom_setting: str = Field(default="default", description="Custom setting")

class MyAgent:
    def __init__(self, cfg: MyAgentConfig) -> None:
        self._config = cfg
        self._prompt_manager = PromptManager(
            file_path=Path(__file__).parent / "prompts" / "my_agent.yaml"
        )
```

## LLM Model Configuration

### Model Property Pattern

```python
@property
def _llm_model(self) -> Tuple[Model, ModelSettings]:
    retrying_http_client = create_retrying_client()
    
    # Support multiple providers
    if "gemini" in config.MY_LLM_MODEL:
        model = GeminiModel(
            model_name=config.MY_LLM_MODEL,
            provider=CustomGeminiGLA(
                api_key=config.MY_LLM_API_KEY,
                base_url=config.MY_LLM_BASE_URL,
                http_client=retrying_http_client,
            ),
        )
    else:
        model = OpenAIModel(
            model_name=config.MY_LLM_MODEL,
            provider=OpenAIProvider(
                base_url=config.MY_LLM_BASE_URL,
                api_key=config.MY_LLM_API_KEY,
                http_client=retrying_http_client,
            ),
        )
    
    settings = ModelSettings(
        temperature=config.MY_LLM_TEMPERATURE,
        max_tokens=config.MY_LLM_MAX_TOKENS,
        timeout=config.MY_LLM_TIMEOUT,
        parallel_tool_calls=config.MY_PARALLEL_TOOL_CALLS,
    )
    
    return model, settings
```

### Agent Instantiation

```python
@property
def _my_specialized_agent(self) -> Agent:
    model, model_settings = self._llm_model
    
    return Agent(
        name="My Specialized Agent",
        model=model,
        model_settings=model_settings,
        system_prompt=self._render_prompt("agents.my_agent.system_prompt"),
        tools=[
            FileReadTool().get_tool(),
            ListFilesTool().get_tool(),
        ],
        retries=config.MY_AGENT_RETRIES,
    )
```

## Prompt Management

### YAML Prompt Structure

```yaml
# src/agents/prompts/my_agent.yaml
agents:
  my_agent:
    system_prompt: |
      You are a specialized code analyzer.
      
      Your task is to analyze {{ repo_path }} and provide insights.
      
      Available tools:
      - Read-File: Read file contents with line ranges
      - List-Files: List directory contents with filtering
    
    user_prompt: |
      Analyze the repository at {{ repo_path }}.
      
      Focus on:
      1. Architecture patterns
      2. Code organization
      3. Key components
```

### Prompt Rendering

```python
def _render_prompt(self, key: str) -> str:
    return self._prompt_manager.render_prompt(
        key=key,
        repo_path=str(self._config.repo_path),
        custom_var=self._config.custom_setting,
    )
```

## Tool Development

### Tool Interface

```python
from pydantic_ai import Tool
from pydantic_ai.exceptions import ModelRetry
from opentelemetry import trace
import config
from utils import Logger

class MyCustomTool:
    def get_tool(self):
        return Tool(
            self._run,
            name="My-Custom-Tool",
            takes_ctx=False,
            max_retries=config.TOOL_MY_CUSTOM_TOOL_MAX_RETRIES,
        )
    
    def _run(self, param1: str, param2: int = 10) -> str:
        """
        Tool description that the LLM sees.
        
        Args:
            param1: Description of param1
            param2: Description of param2 (default: 10)
        
        Returns:
            Description of return value
        """
        Logger.debug(f"Running My-Custom-Tool with param1={param1}, param2={param2}")
        
        span = trace.get_current_span()
        span.set_attribute("tool.input.param1", param1)
        span.set_attribute("tool.input.param2", param2)
        
        try:
            # Tool implementation
            result = self._do_work(param1, param2)
            
            span.set_attribute("tool.output", result)
            Logger.debug(f"My-Custom-Tool completed successfully")
            
            return result
        
        except FileNotFoundError as e:
            raise ModelRetry(message=f"File not found: {e}")
        except PermissionError as e:
            raise ModelRetry(message=f"Permission denied: {e}")
        except Exception as e:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/divar-ir) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
