---
title: Tool Calling
typora-root-url: Tool Calling
date: 2026-02-05 10:35:15
tags: [AI]
categories: [AI]
description: Spring AI为开发者提供了模型调用API的能力，增强大模型的能力
---

## 1.背景引入

大语言模型本质上是基于海量文本信息训练的统计模型，核心机制是学习词与词之间的关系。这就导致了模型对时间的理解仅局限于文本中出现的日期表达式，而不是真实的时间

![image-20260205104355317](image-20260205104355317.png)

可以看到，模型的输出和现实时间不一致。

模型的训练数据有明确的截止时间（例如GPT-4为2023年底），在此之后发生的事件都不在模型知识训练范围内。

![image-20260205105209219](image-20260205105209219.png)

针对以上不足的信息，我们需要为模型提供查找外部信息的能力，也就是让大模型可以调用我们自己创建的API。



## 2. 函数调用原理

大模型仅负责 **决定是否调用工具** 和 **提供参数**，实际执行逻辑由客户端（Spring 应用）实现，确保工具调用的可控性与安全性。

![img](1769816-20250414102420526-2145105587.png)

### 2.1. 工具元数据注入

在发起Chat Request时，将工具描述（name/description）、参数结构（input schema）等元数据封装至请求体，建立大模型的工具调用能力基线。

### 2.2. 模型决策响应

大模型根据上下文推理生成工具调用指令（tool_calls字段），返回包含选定工具名称及结构化参数的中间响应。

### 2.3. 服务端路由执行

Spring AI模块解析工具调用指令，通过服务发现机制定位目标工具实例，注入参数并触发同步/异步执行。

### 2.4. 执行结果标准化

工具返回原始执行结果后，系统进行数据类型校验、异常捕获和JSON序列化处理，生成模型可解析的标准化数据结构。

### 2.5. 上下文增强推理

将标准化结果作为新增上下文（tool_outputs）回传大模型，触发基于增强上下文的二次推理流程。

### 2.6. 终端响应生成

模型综合初始请求与工具执行结果，生成最终自然语言响应，完成工具增强的对话闭环。



## 3.函数调用

### 3.1 创建一个时间工具类

```
ublic class DateTimeTools {
    @Tool(description = "获取当前日期和时间。")
    String getCurrentDateTime() {
        log.info("调用时间工具");
        return LocalDateTime.now().atZone(LocaleContextHolder.getTimeZone().toZoneId()).toString();
    }
}
```

- @Tool注解定义当前方法可以被大模型调用
- description告诉大模型该方法的用途



### 3.2 配置ChatClient

默认配置

```
  @Bean
    public ChatClient getChatClient(ChatModel chatModel, ChatMemory chatMemory) throws Exception {

        return ChatClient.builder(chatModel)
                .defaultTools(new DateTimeTools()) // 时间工具
                .defaultAdvisors(new MessageChatMemoryAdvisor(chatMemory))
                .build();
    }
```



调用配置

```
 String response = chatClient.prompt(message)
                .tools(new DateTimeTools())
                .call()
                .content();
```



### 3.3 测试

![image-20260205110442269](image-20260205110442269.png)

现在模型可以获取正确的时间了。



## 4. Spring AI某些版本使用流式输出调用工具出现的bug

![image-20260205110717899](image-20260205110717899.png)

https://github.com/alibaba/spring-ai-alibaba/issues/4066

当流式输出时，会出现调用方法参数为空的bug。(Spring AI版本 1.1.0-M1)

建议使用同步输出或者更换到新版本
