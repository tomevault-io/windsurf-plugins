---
trigger: always_on
description: 本客户端基于讯飞开放平台 星辰Agent 实现，提供星火智能体工作流调用能力[官方文档](https://www.xfyun.cn/doc/spark/Agent04-API%E6%8E%A5%E5%85%A5.html#_2-%E5%B7%A5%E4%BD%9C%E6%B5%81-api-%E9%9B%86%E6%88%90)
---

# 星火智能体 API文档

## 简介

本客户端基于讯飞开放平台 星辰Agent 实现，提供星火智能体工作流调用能力[官方文档](https://www.xfyun.cn/doc/spark/Agent04-API%E6%8E%A5%E5%85%A5.html#_2-%E5%B7%A5%E4%BD%9C%E6%B5%81-api-%E9%9B%86%E6%88%90)

## 功能列表

| 方法名       | 功能说明                |
| ------------ | ----------------------- |
| completion() | 执行工作流(流式/非流式) |
| resume()     | 恢复工作流(流式)        |
| uploadFile() | 文件上传                |

## 使用准备

1. 前往[星辰Agent-发布管理](https://agent.xfyun.cn/management/release/workflow)发布工作流为API
3. 发布成功后绑定应用获取以下凭证:
   - API Secret
   - API Key
   - API Flowid

## 快速开始

1、添加maven依赖

```xml
<dependency>
    <groupId>cn.xfyun</groupId>
    <artifactId>websdk-java-spark</artifactId>
    <!--请替换成最新稳定版本-->
    <version>2.1.5</version>
</dependency>
```

2、Java代码

```java
package cn.xfyun.demo.spark;

import cn.xfyun.api.AgentClient;
import cn.xfyun.config.PropertiesConfig;
import cn.xfyun.exception.BusinessException;
import cn.xfyun.model.agent.AgentChatParam;
import cn.xfyun.model.agent.AgentResumeParam;
import cn.xfyun.service.agent.AgentCallback;
import cn.xfyun.util.StringUtils;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import okhttp3.Call;
import okhttp3.Response;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;
import java.io.IOException;
import java.net.URISyntaxException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Objects;
import java.util.Scanner;

/**
 * （ai-ppt-v2）智能体工作流
 * 1、APPID、APISecret、APIKey、APIPassword信息获取：<a href="https://www.xfyun.cn/doc/spark/Agent01-%E5%B9%B3%E5%8F%B0%E4%BB%8B%E7%BB%8D.html">...</a>
 * 2、文档地址：<a href="https://www.xfyun.cn/doc/spark/PPTv2.html">...</a>
 */
public class AgentClientApp {

    private static final Logger logger = LoggerFactory.getLogger(AgentClientApp.class);
    private static final String API_KEY = PropertiesConfig.getApiKey();
    private static final String API_SECRET = PropertiesConfig.getApiSecret();
    private static final SimpleDateFormat sdf = new SimpleDateFormat("yyy-MM-dd HH:mm:ss.SSS");
    private static String filePath;
    private static String resourcePath;

    static {
        try {
            filePath = "image/hidream_1.jpg";
            resourcePath = Objects.requireNonNull(AIPPTV2ClientApp.class.getResource("/")).toURI().getPath();
        } catch (URISyntaxException e) {
            logger.error("获取资源路径失败", e);
        }
    }

    public static void main(String[] args) {
        try {
            AgentClient client = new AgentClient.Builder(API_KEY, API_SECRET).build();

            JSONObject parameter = JSONObject.parseObject("{\"AGENT_USER_INPUT\": \"今天天气怎么样\"}");
            AgentChatParam agentChatParam = AgentChatParam.builder()
                    // .flowId("7351173267335847938")
                    .flowId("7351431612989308928")
                    .parameters(parameter)
                    .build();

            // 流式请求
            stream(client, agentChatParam);

            // 非流式请求
            // generate(client, agentChatParam);

            // 上传文件
            // uploadFile(client, new File(resourcePath + filePath));
        } catch (Exception e) {
            logger.error("请求失败", e);
        }
    }

    private static void uploadFile(AgentClient client, File file) throws IOException {
        String result = client.uploadFile(file);
        logger.info(result);
    }

    private static void generate(AgentClient client, AgentChatParam agentChatParam) throws IOException {
        String result = client.completion(agentChatParam);
        logger.info("工作流返回结果：{}", result);
        JSONObject obj = JSON.parseObject(result);
        int code = obj.getIntValue("code");
        if (code != 0) {
            logger.error(result);
            return;
        }
        JSONObject messages = obj.getJSONArray("choices").getJSONObject(0).getJSONObject("delta");
        logger.info("解析结果: {}", messages);
    }

    private static void stream(AgentClient client, AgentChatParam agentChatParam) {
        Date dateBegin = new Date();
        StringBuilder finalResult = new StringBuilder();
        StringBuilder thingkingResult = new StringBuilder();

        client.completion(agentChatParam, getCallback(finalResult, thingkingResult, client, dateBegin));
    }

    private static AgentCallback getCallback(StringBuilder finalResult, StringBuilder thingkingResult, AgentClient client, Date dateBegin) {
        return new AgentCallback() {
            @Override
            public void onEvent(Call call, String id, String type, String data) {
                resultHandler(data, finalResult, thingkingResult, client, dateBegin);
            }

            @Override
            public void onFail(Call call, Throwable t) {
                logger.error("sse通信出错", t);
            }

            @Override
            public void onClosed(Call call) {
                logger.info("sse断开链接");
                call.cancel();
            }

            @Override
            public void onOpen(Call call, Response response) {
                logger.info("sse建立链接");
            }
        };
    }

    /**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iFLYTEK-OP/websdk-java](https://github.com/iFLYTEK-OP/websdk-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
