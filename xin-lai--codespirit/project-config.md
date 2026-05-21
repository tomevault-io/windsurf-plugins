---
trigger: always_on
description: CodeSpirit 测试开发规范 - 单元测试、集成测试、测试项目结构和Mock使用
---


# 测试开发规范

## 测试项目结构

```
Tests/
├── ApiServices/              # API服务测试
│   ├── CodeSpirit.ExamApi.Tests/
│   └── CodeSpirit.IdentityApi.Tests/
├── Components/               # 组件测试
│   ├── CodeSpirit.Amis.Tests/
│   └── CodeSpirit.LLM.Tests/
├── Infrastructure/           # 基础设施测试
└── Shared/                   # 共享测试工具
```

## 测试命名约定

### 测试类命名
- 格式：`{被测试类名}Tests`
- 示例：`UserServiceTests`、`QuestionsControllerTests`

### 测试方法命名
- 格式：`{方法名}_{场景}_{预期结果}`
- 使用 `[Fact]` 或 `[Theory]` 特性
- 示例：
  ```csharp
  [Fact]
  public async Task GetByIdAsync_ValidId_ReturnsUserDto()
  {
      // Arrange
      // Act
      // Assert
  }
  
  [Theory]
  [InlineData(1)]
  [InlineData(2)]
  public async Task GetByIdAsync_InvalidId_ThrowsNotFoundException(long id)
  {
      // Arrange
      // Act & Assert
  }
  ```

## 测试框架

- **单元测试**: xUnit
- **Mock框架**: Moq 或 NSubstitute
- **断言**: FluentAssertions（推荐）或 xUnit 内置断言

## 单元测试示例

```csharp
using Xunit;
using FluentAssertions;
using Moq;
using CodeSpirit.ExamApi.Services;
using CodeSpirit.ExamApi.Services.Interfaces;

namespace CodeSpirit.ExamApi.Tests.Services;

/// <summary>
/// 题目服务测试
/// </summary>
public class QuestionServiceTests
{
    private readonly Mock<IRepository<Question>> _repositoryMock;
    private readonly Mock<IMapper> _mapperMock;
    private readonly QuestionService _service;
    
    public QuestionServiceTests()
    {
        _repositoryMock = new Mock<IRepository<Question>>();
        _mapperMock = new Mock<IMapper>();
        _service = new QuestionService(_repositoryMock.Object, _mapperMock.Object);
    }
    
    [Fact]
    public async Task GetQuestionAsync_ValidId_ReturnsQuestionDto()
    {
        // Arrange
        var questionId = 1L;
        var question = new Question { Id = questionId, Content = "测试题目" };
        var questionDto = new QuestionDto { Id = questionId, Content = "测试题目" };
        
        _repositoryMock.Setup(r => r.GetByIdAsync(questionId))
            .ReturnsAsync(question);
        _mapperMock.Setup(m => m.Map<QuestionDto>(question))
            .Returns(questionDto);
        
        // Act
        var result = await _service.GetQuestionAsync(questionId);
        
        // Assert
        result.Should().NotBeNull();
        result.Id.Should().Be(questionId);
        result.Content.Should().Be("测试题目");
        _repositoryMock.Verify(r => r.GetByIdAsync(questionId), Times.Once);
    }
    
    [Fact]
    public async Task GetQuestionAsync_InvalidId_ThrowsNotFoundException()
    {
        // Arrange
        var questionId = 999L;
        _repositoryMock.Setup(r => r.GetByIdAsync(questionId))
            .ReturnsAsync((Question?)null);
        
        // Act & Assert
        await Assert.ThrowsAsync<BusinessException>(() => 
            _service.GetQuestionAsync(questionId));
    }
}
```

## 集成测试示例

```csharp
using Xunit;
using Microsoft.AspNetCore.Mvc.Testing;
using CodeSpirit.ExamApi;
using System.Net.Http;
using System.Threading.Tasks;

namespace CodeSpirit.ExamApi.Tests.Integration;

/// <summary>
/// 题目API集成测试
/// </summary>
public class QuestionsControllerIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    
    public QuestionsControllerIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }
    
    [Fact]
    public async Task GetQuestions_ReturnsSuccessStatusCode()
    {
        // Arrange
        var request = new HttpRequestMessage(HttpMethod.Get, "/exam/api/Questions");
        
        // Act
        var response = await _client.SendAsync(request);
        
        // Assert
        response.EnsureSuccessStatusCode();
        var content = await response.Content.ReadAsStringAsync();
        content.Should().NotBeNullOrEmpty();
    }
}
```

## Mock 使用规范

### 使用 Moq

```csharp
// 设置返回值
_mockService.Setup(s => s.GetByIdAsync(It.IsAny<long>()))
    .ReturnsAsync(new UserDto { Id = 1 });

// 验证调用
_mockService.Verify(s => s.GetByIdAsync(1), Times.Once);

// 设置异常
_mockService.Setup(s => s.GetByIdAsync(It.IsAny<long>()))
    .ThrowsAsync(new BusinessException("用户不存在"));
```

### 使用 NSubstitute

```csharp
// 设置返回值
_substituteService.GetByIdAsync(Arg.Any<long>())
    .Returns(new UserDto { Id = 1 });

// 验证调用
_substituteService.Received(1).GetByIdAsync(1);

// 设置异常
_substituteService.GetByIdAsync(Arg.Any<long>())
    .ThrowsAsync(new BusinessException("用户不存在"));
```

## 测试数据准备

### 使用 Fixture

```csharp
public class QuestionFixture
{
    public static Question CreateValidQuestion()
    {
        return new Question
        {
            Id = 1,
            Content = "测试题目",
            Type = QuestionType.SingleChoice,
            CreatedAt = DateTime.UtcNow
        };
    }
    
    public static CreateQuestionDto CreateValidCreateDto()
    {
        return new CreateQuestionDto
        {
            Content = "测试题目",
            Type = QuestionType.SingleChoice
        };
    }
}
```

## 异步测试

```csharp
[Fact]
public async Task CreateQuestionAsync_ValidDto_ReturnsQuestionDto()
{
    // Arrange
    var dto = QuestionFixture.CreateValidCreateDto();
    
    // Act
    var result = await _service.CreateQuestionAsync(dto);
    
    // Assert
    result.Should().NotBeNull();
    result.Content.Should().Be(dto.Content);
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xin-lai/CodeSpirit](https://github.com/xin-lai/CodeSpirit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
