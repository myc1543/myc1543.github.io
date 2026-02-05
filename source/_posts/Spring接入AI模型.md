---
title: Spring接入AI模型
typora-root-url: Spring接入AI模型
date: 2026-01-29 16:17:02
tags: [AI]
categories: [AI]
description: 记录一下如何Spring如何接入AI模型
---

## 1.SpringAI介绍

`SpringAI`解决了AI集成的根本难题：将企业数据和AI模型连接起来。
Spring AI提供的抽象是开发AI应用的基础。
Spring AI提供了以下功能

- 跨多家 AI 提供商的 Chat、文本转图像和嵌入模型的可移植 API 支持。支持同步和流式A PI 选项。
- 支持所有主要的 AI 模型提供商，如 Anthropic、OpenAI、微软、亚马逊、谷歌和 Ollama。支持的模型类型包括：
  - 对话补全
  - 潜入
  - 文生图
  - 音频转录
  - 文本转语音
  - 内容审核
- 结构化输出 - 将 AI 模型输出映射到 POJO。
- 跨向量数据库提供商的可移植API



## 2. 引入Maven坐标

| 组件        | 版本要求     | 备注          |
| ----------- | ------------ | ------------- |
| JDK         | 17+          | 推荐LTS版本   |
| Spring Boot | 3.0+         | 需支持Java 17 |
| Maven       | 3.8+         | 或Gradle 7.5+ |
| OpenAI API  | 自行选择模型 | 需申请API Key |

在pom.xml中添加Spring AI Starter依赖

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.4.3</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<properties>
    <java.version>17</java.version>
    <spring-ai.version>1.0.0-M6</spring-ai.version>
</properties>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
</dependency>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>${spring-ai.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

```



## 3.配置文件

application.yml中

```
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}
      base-url: https://dashscope.aliyuncs.com/compatible-mode
      chat:
        options:
          model: qwen-turbo
          max-tokens: 2000
```



| 参数名              | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| `api-key`           | OpenAI的API密钥，用于身份验证和API调用授权                   |
| `base-url`          | 指定AI服务的API端点地址                                      |
| `model` (chat)      | 指定使用的聊天模型，决定AI的能力和成本                       |
| `temperature`       | 控制输出的随机性和创造性，取值0.0-2.0，越低越确定，越高越有创意 |
| `max-tokens`        | 限制单次响应的最大token数量，控制回复长度和API成本           |
| `model` (embedding) | 用于文本向量化的嵌入模型，用于语义搜索、相似度计算等场景     |



## 4.使用阿里云的模型服务平台

阿里云可以提供100万token的免费额度，所以可以接入阿里的模型。
阿里的官方文档:https://help.aliyun.com/zh/model-studio/compatibility-of-openai-with-dashscope

| 页面         | 网址                                                         |
| ------------ | :----------------------------------------------------------- |
| 配置API Key  | https://bailian.console.aliyun.com/?spm=a2c4g.11186623.0.0.6029533aOs1B0X&tab=model#/api-key |
| 查看模型用量 | https://bailian.console.aliyun.com/?spm=a2c4g.11186623.0.0.5211175fiLZA7q&tab=model#/model-usage/free-quota |



## 5.demo

### 5.1 配置bean

```
/**
 * Spring AI ChatClient 配置类
 * 用于创建和配置 ChatClient bean，以支持 Spring AI 的聊天功能
 */
@Configuration
public class ChatClientConfig {

    /**
     * 创建 ChatClient bean，用于与 AI 模型交互
     */
    @Bean
    public ChatClient gameChatClient(ChatModel chatModel) {
        return ChatClient.builder(chatModel)
                .build();
    }
}
```



### 5.2 编写接口

```
@RestController
@RequestMapping("/api/chat")
@Slf4j
public class ChatController {

    @Autowired
    ChatClient chatClient;

    @PostMapping("/send")
    public String chat(@RequestBody String message) {
        return chatClient.prompt()
                .user(message)
                .call()
                .content();
    }

    /**
     * @return 流式响应结果
     */
    @GetMapping("/stream")
    public Flux<String> streamChat(@RequestParam String message) {
        return chatClient.prompt()
                .user(message)
                .stream()
                .content();
    }
}
```

可以看一下效果

![image-20260129161612068](image-20260129161612068.png)
