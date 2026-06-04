---
trigger: always_on
description: 测试和部署规范
---

# 测试和部署规范

## 测试框架规范

### 1. 单元测试
```python
# tests/test_etl/test_crawler.py
import pytest
import asyncio
from unittest.mock import Mock, patch, AsyncMock
from etl.crawler.webpage_spider.counselor.spiders.wiki import WikiSpider

class TestWikiSpider:
    """Wiki爬虫测试类"""
    
    @pytest.fixture
    def spider(self):
        """创建爬虫实例"""
        spider = WikiSpider()
        spider.settings = Mock()
        return spider
    
    @pytest.fixture
    def mock_response(self):
        """模拟响应对象"""
        response = Mock()
        response.url = "https://example.com/test"
        response.status = 200
        response.css = Mock()
        return response
    
    def test_extract_title_success(self, spider, mock_response):
        """测试标题提取成功场景"""
        # 设置模拟数据
        mock_response.css.return_value.get.return_value = "测试标题"
        
        # 执行测试
        title = spider.extract_title(mock_response)
        
        # 验证结果
        assert title == "测试标题"
        mock_response.css.assert_called()
    
    def test_extract_title_fallback(self, spider, mock_response):
        """测试标题提取回退机制"""
        # 设置模拟数据 - 没有找到标题
        mock_response.css.return_value.get.return_value = None
        mock_response.url = "https://example.com/test-page"
        
        # 执行测试
        title = spider.extract_title(mock_response)
        
        # 验证回退到URL
        assert title == "test-page"
    
    def test_clean_text(self, spider):
        """测试文本清理功能"""
        # 测试数据
        dirty_text = "  这是一个    测试\n\n文本  \t  "
        
        # 执行清理
        clean_text = spider.clean_text(dirty_text)
        
        # 验证结果
        assert clean_text == "这是一个 测试 文本"
        assert not clean_text.startswith(' ')
        assert not clean_text.endswith(' ')
    
    def test_should_follow_allowed_domain(self, spider):
        """测试域名白名单过滤"""
        spider.allowed_domains = ['example.com']
        
        # 允许的域名
        assert spider.should_follow('https://example.com/page') == True
        
        # 不允许的域名
        assert spider.should_follow('https://other.com/page') == False
    
    def test_should_follow_blocked_extensions(self, spider):
        """测试文件扩展名过滤"""
        # 被阻止的文件类型
        assert spider.should_follow('https://example.com/file.pdf') == False
        assert spider.should_follow('https://example.com/doc.docx') == False
        
        # 允许的页面
        assert spider.should_follow('https://example.com/page.html') == True
    
    @pytest.mark.asyncio
    async def test_parse_content_integration(self, spider, mock_response):
        """集成测试：解析页面内容"""
        # 设置复杂的模拟响应
        mock_response.css.side_effect = [
            Mock(get=Mock(return_value="测试标题")),  # 标题提取
            Mock(css=Mock(return_value=Mock(getall=Mock(return_value=["测试", "内容"])))),  # 内容提取
        ]
        
        # 执行解析
        result = spider.parse_content(mock_response)
        
        # 验证结果结构
        assert 'url' in result
        assert 'title' in result
        assert 'content' in result
        assert result['title'] == "测试标题"
```

### 2. API测试
```python
# tests/test_api/test_knowledge_search.py
import pytest
from fastapi.testclient import TestClient
from unittest.mock import patch, Mock
from app import app

client = TestClient(app)

class TestKnowledgeSearchAPI:
    """知识搜索API测试"""
    
    @pytest.fixture
    def mock_search_service(self):
        """模拟搜索服务"""
        with patch('api.routes.knowledge.search.search_service') as mock:
            yield mock
    
    def test_search_success(self, mock_search_service):
        """测试搜索成功场景"""
        # 设置模拟返回
        mock_search_service.search.return_value = [
            {
                "title": "测试文档",
                "content": "这是测试内容",
                "score": 0.95,
                "url": "https://example.com/test"
            }
        ]
        
        # 发送请求
        response = client.post("/api/knowledge/advanced-search", json={
            "query": "测试查询",
            "openid": "test_user"
        })
        
        # 验证响应
        assert response.status_code == 200
        data = response.json()
        assert data["code"] == 200
        assert len(data["data"]) == 1
        assert data["data"][0]["title"] == "测试文档"
    
    def test_search_empty_query(self):
        """测试空查询参数"""
        response = client.post("/api/knowledge/advanced-search", json={
            "query": "",
            "openid": "test_user"
        })
        
        assert response.status_code == 422  # 参数验证错误
    
    def test_search_missing_openid(self):
        """测试缺少openid参数"""
        response = client.post("/api/knowledge/advanced-search", json={
            "query": "测试查询"
        })
        
        assert response.status_code == 422
    
    def test_search_service_error(self, mock_search_service):
        """测试搜索服务异常"""
        # 设置服务抛出异常
        mock_search_service.search.side_effect = Exception("搜索服务异常")
        
        response = client.post("/api/knowledge/advanced-search", json={
            "query": "测试查询",
            "openid": "test_user"
        })
        
        assert response.status_code == 200
        data = response.json()
        assert data["code"] == 500
        assert "错误" in data["message"]
    
    @pytest.mark.parametrize("query,expected_count", [
        ("南开大学", 5),
        ("计算机科学", 3),
        ("*模式", 2),  # 通配符查询

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NKU-WIKI/nkuwiki](https://github.com/NKU-WIKI/nkuwiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
