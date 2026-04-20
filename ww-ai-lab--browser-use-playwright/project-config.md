---
trigger: always_on
description: 参考 [src/core/recorder.py](mdc:src/core/recorder.py) 的实现：
---

# 浏览器自动化开发规范

## Browser-Use 集成规范

### 录制阶段实现
参考 [src/core/recorder.py](mdc:src/core/recorder.py) 的实现：

```python
from browser_use import Agent

class WorkflowRecorder:
    """Browser-Use工作流录制器"""
    
    async def record_workflow(self, name: str, task_description: str) -> Workflow:
        """使用Browser-Use录制工作流"""
        agent = Agent(
            task=task_description,
            llm=self.llm,
            browser_config=self.browser_config
        )
        
        # 启动录制会话
        result = await agent.run()
        
        # 转换为标准工作流格式
        workflow = self._convert_to_workflow(result, name)
        return workflow
```

### Browser-Use 配置
```python
from browser_use import BrowserConfig

browser_config = BrowserConfig(
    headless=False,           # 录制时显示浏览器
    browser_type="chromium",  # 使用Chromium
    viewport_size=(1920, 1080),
    user_data_dir="./chrome-profiles",  # 复用浏览器配置
    extra_chromium_args=[
        "--disable-blink-features=AutomationControlled",
        "--disable-dev-shm-usage"
    ]
)
```

### 步骤类型映射
Browser-Use动作到工作流步骤的映射：

```python
BROWSER_USE_TO_STEP_TYPE = {
    "navigate": StepType.NAVIGATE,
    "click": StepType.CLICK,
    "type": StepType.FILL,
    "select": StepType.SELECT,
    "wait": StepType.WAIT,
    "scroll": StepType.SCROLL,
    "hover": StepType.HOVER,
    "press": StepType.PRESS_KEY,
    "screenshot": StepType.SCREENSHOT,
    "extract": StepType.EXTRACT
}
```

## Playwright 执行规范

### 执行器架构
```python
from playwright.async_api import async_playwright, Browser, Page

class PlaywrightExecutor:
    """Playwright工作流执行器"""
    
    def __init__(self):
        self.browser: Optional[Browser] = None
        self.context = None
        self.page: Optional[Page] = None
    
    async def execute_workflow(self, workflow: Workflow, context: Dict[str, str]) -> ExecutionResult:
        """执行工作流"""
        try:
            await self._setup_browser()
            
            for step in workflow.steps:
                await self._execute_step(step, context)
                
            return ExecutionResult(success=True)
            
        except Exception as e:
            logger.error("工作流执行失败", error=str(e))
            return ExecutionResult(success=False, error=str(e))
        finally:
            await self._cleanup()
```

### 步骤执行器
使用工厂模式创建不同类型的步骤执行器：

```python
class StepExecutor(ABC):
    @abstractmethod
    async def execute(self, page: Page, step: WorkflowStep, context: Dict[str, str]) -> StepResult:
        pass

class NavigateExecutor(StepExecutor):
    async def execute(self, page: Page, step: WorkflowStep, context: Dict[str, str]) -> StepResult:
        url = self._render_template(step.url, context)
        await page.goto(url, timeout=step.timeout or 30000)
        return StepResult(success=True, step_id=step.id)

class ClickExecutor(StepExecutor):
    async def execute(self, page: Page, step: WorkflowStep, context: Dict[str, str]) -> StepResult:
        selector = step.selector or step.xpath
        await page.click(selector, timeout=step.timeout or 10000)
        return StepResult(success=True, step_id=step.id)

class FillExecutor(StepExecutor):
    async def execute(self, page: Page, step: WorkflowStep, context: Dict[str, str]) -> StepResult:
        selector = step.selector or step.xpath
        value = self._render_template(step.value, context)
        await page.fill(selector, value, timeout=step.timeout or 10000)
        return StepResult(success=True, step_id=step.id)
```

### 并发执行管理
```python
import asyncio
from asyncio import Semaphore

class ConcurrentExecutor:
    def __init__(self, max_concurrent: int = 5):
        self.semaphore = Semaphore(max_concurrent)
        self.browser_pool = BrowserPool(max_browsers=max_concurrent)
    
    async def execute_batch(self, workflows: List[Workflow]) -> List[ExecutionResult]:
        """批量执行工作流"""
        tasks = []
        for workflow in workflows:
            task = self._execute_with_semaphore(workflow)
            tasks.append(task)
        
        results = await asyncio.gather(*tasks, return_exceptions=True)
        return [r if isinstance(r, ExecutionResult) else ExecutionResult(success=False, error=str(r)) for r in results]
    
    async def _execute_with_semaphore(self, workflow: Workflow) -> ExecutionResult:
        async with self.semaphore:
            browser = await self.browser_pool.acquire()
            try:
                executor = PlaywrightExecutor(browser)
                return await executor.execute_workflow(workflow)
            finally:
                await self.browser_pool.release(browser)
```

## 浏览器配置管理

### Chrome Profile 复用
```python
class BrowserProfileManager:
    """浏览器配置文件管理器"""
    
    def __init__(self, profiles_dir: Path = Path("chrome-profiles")):
        self.profiles_dir = profiles_dir
        self.profiles_dir.mkdir(exist_ok=True)
    
    def get_profile_path(self, profile_name: str = "default") -> Path:
        """获取浏览器配置文件路径"""
        profile_path = self.profiles_dir / profile_name
        profile_path.mkdir(exist_ok=True)
        return profile_path
    
    async def launch_with_profile(self, profile_name: str = "default") -> Browser:
        """使用指定配置文件启动浏览器"""
        async with async_playwright() as p:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WW-AI-Lab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
