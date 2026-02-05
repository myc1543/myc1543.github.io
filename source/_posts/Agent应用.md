---
title: Agent应用
typora-root-url: Agent应用
date: 2026-01-29 16:31:34
tags: [Agent]
categories: [AI]
description: 再引入Spring AI之后，有许多参数可以配置，本篇文章来讲讲这些参数应该怎么使用。
---

用Spring AI调用模型大概是这样的

```
chatClient.prompt()           // 1. 创建prompt构建器
    .user(message)            // 2. 设置用户消息
    .system(systemPrompt)     // 3. 设置系统提示词（可选）
    .options(options)         // 4. 设置模型参数（可选）
    .call()                   // 5. 执行调用
    .content();               // 6. 获取结果
```

每个参数都会对结果产生影响



## 1. 输出模式

在成熟的产品中，我们和AI对话时，AI会把生成的结果一点点呈现出来，就像打字机一样，这是流式输出。
![image-20260129163724725](image-20260129163724725.png)



Spring AI为我们提供了两种输出模式

- 同步输出模式：客户端发送请求后会等待，直到AI模型生成完整的响应内容后一次性返回
- 流式输出模式：流式输出采用SSE技术，AI模型生成内容时会逐步推送给客户端

### 1.1 同步输出

```
public String chat(@RequestBody String message) {
        return chatClient.prompt()
                .user(message)
                .call()
                .content();
    }
```

ChatClient提供call方法，一次性返回所有结果



### 1.2 流式输出

```
 public Flux<String> streamChat(@RequestParam String message) {
        return chatClient.prompt()
                .user(message)
                .stream()
                .content();
    }
```

ChatClient提供stream方法，流式输出，使用Flux返回结果



### 1.3 输出模式对比

流式输出通常是更好的选择，因为和模型交互是一段比较长的时间，如果一直阻塞不给用户反馈，会带来不好的产品体验。使用流式输出，用户可以实时看到内容生成，感觉更加流畅，并且采用非阻塞IO，资源利用率高，适合长文本输出。

另一方面，流式输出需要前端做对应的处理，接受流式处理。



## 2. 提示词

大语言模型通过注意力机制来处理输入，提示词通过强有力的上下文信号，引导模型关注特定知识领域。

Spring AI提供两种方式来预设提示词

- 调用模型时
- 创建client时

### 2.1 调用模型时

```
public String chat(String message) {
        return chatClient.prompt()
            .system("你是一位Java专家")  // ← 每次调用时设置
            .user(message)
            .call()
            .content();
    }
```

通过system方法预设提示词



### 2.2 创建client时

```
 @Bean
    public ChatClient getChatClient(ChatModel chatModel) {
        return ChatClient.builder(chatModel)
                .defaultSystem("你是一位Java专家")
                .build();
    }
```



### 2.3 对比

提示词参考

```
# 身份定位
你是"智游AI"，一位专业的智能旅行规划助手。你能够整合天气、交通、景点、口碑等多源信息，为用户提供个性化、可动态调整的旅行行程规划。

## 核心能力

### 1. 信息整合
实时获取并分析：天气预报、交通票价、景点客流、用户评价、餐饮住宿、特殊事件等信息。

### 2. 用户理解
深入了解用户的：预算约束、兴趣偏好、体力状况、时间安排、特殊需求、同行人员。

### 3. 智能规划
- 时空优化：合理安排景点顺序，最小化交通时间
- 节奏控制：根据体力动态调整游览强度
- 成本优化：在预算内最大化体验价值
- 避峰策略：推荐人少时段，避开高峰
- 应急预案：为关键节点提供备选方案

### 4. 动态调整
当出现天气变化、景点闭馆、体力不支、交通延误、预算超支等情况时，主动优化行程。

## 工作流程

### 第一步：需求收集
友好地询问用户：
- 📍 目的地、出行日期、天数
- 💰 总预算及分配偏好
- ❤️ 兴趣偏好（文化/自然/美食/休闲/冒险等）
- 🏃 体力评估（年龄段、步行距离、游览强度）
- 👥 同行信息及特殊需求
- ⏰ 时间偏好（起床时间、午休、晚间活动）

如信息不完整，主动询问缺失项。

### 第二步：信息分析
向用户说明正在收集信息：
🔍 正在为您收集信息...
✓ 已获取天气预报
✓ 已分析景点客流
✓ 已查询交通票价
✓ 已整合口碑数据
📊 数据分析完成，开始生成行程...

### 第三步：行程生成

按以下格式输出：

```markdown
# 🎒 您的专属旅行计划

## 📋 行程概览
- 目的地：[城市]
- 日期：[日期]（共X天）
- 预算：¥[总额]（预估：¥[金额]，剩余：¥[金额]）
- 主题：[如"文化探索+美食之旅"]

## 🌤️ 天气预报
| 日期 | 天气 | 温度 | 建议 |
|------|------|------|------|
| Day1 | ☀️晴 | 18-28℃ | 适合户外 |

---

## 📅 Day 1：[主题]

### 🌅 上午（09:00-12:00）
#### 📍 [景点名]
- 地址：[地址]
- 门票：¥[价格] | 游览时长：[时间]
- 亮点：[核心看点]
- 客流：🟢较少 | 避坑：[提示]
- 交通：[方式及时间]

#### 🍜 午餐：[餐厅]
- 位置：[位置] | 人均：¥[价格]
- 特色：[菜品] | 评分：⭐⭐⭐⭐⭐ [分数]

### 🌆 下午（14:00-18:00）
[同上格式]

### 🌃 晚上（19:00-21:00）
#### 🎭 [活动]
[活动信息]

#### 🏨 住宿：[酒店]
- 价格：¥[价格]/晚 | 评分：⭐⭐⭐⭐
- 设施：[设施] | 理由：[选择理由]

### 💡 今日小贴士
- ⚠️ [注意事项]
- 🎒 [携带物品]
- 📸 [拍照建议]

### 💰 今日预算：¥[金额]

---

## 🎯 行程亮点
1. ✨ 必打卡：[TOP3景点]
2. 🍽️ 美食：[特色美食]
3. 📷 拍照点：[推荐位置]

## 🔄 备选方案
- 如遇下雨：[替代方案]
- 如时间不足：[可压缩/跳过景点]
- 如预算超支：[节省方案]

## 💬 互动提示
您可以随时告诉我：
- "调整Day2安排" → 重新规划
- "预算太紧" → 更经济方案
- "增加美食" → 增加美食安排
- "太累了" → 降低强度
```



| 无提示词                                                | 有提示词                         |
| ------------------------------------------------------- | -------------------------------- |
| ![image-20260129173930530](image-20260129173930530.png) | ![](image-20260129174023905.png) |



通过对比可以发现，在添加提示词的情况下，输出结果更加具体，质量更高。





## 3. 上下文

AI对话中的上下文就是对话的记录内容。如果AI不能记住上下文，就只能进行一次对话，无法记住前面的对话内容，不能进行多轮对话

| 未开启上下文                                            | 已开启上下文                                            |
| ------------------------------------------------------- | ------------------------------------------------------- |
| ![image-20260129190728283](image-20260129190728283.png) | ![image-20260129192008642](image-20260129192008642.png) |

不难看出，Agent应用，应该是需要有记忆上下文的能力。



### 3.1 MessageChatMemoryAdvisor

Spring AI提供了MessageChatMemoryAdvisor 用来存储上下文

**加载ChatMemory**

```
 @Bean
    public ChatMemory chatMemory() {
        return new InMemoryChatMemory();
    }
```

**创建 ChatClient时，设置memory**

```
 return ChatClient.builder(chatModel)
							 	// 配置Advisor
                .defaultAdvisors(new MessageChatMemoryAdvisor(chatMemory))
                .build();
    }
```

**调用**

```
 public Flux<String> chatWithMemory(String sessionId, String message){
        return chatClient.prompt()
                .user(message)
                .advisors(advisorSpec -> advisorSpec
                        // 指定会话ID
                        .param(CHAT_MEMORY_CONVERSATION_ID_KEY, sessionId)
                        // 保留最近10轮对话
                        .param(CHAT_MEMORY_RETRIEVE_SIZE_KEY, 10)
                )
                .stream()
                .content();
    }
```



本质上这种方法是把对话的内容存放到内存中，然后把内容带到与模型的交互中



### 3.2 Redis存储上下文

把对话的内容以每条记录为{"角色" : "内容"}的形式记录下来，组合成MessageList。每次请求对话，加载以往的记录，调用模型，将返回值添加到已有的记录中持久保存。通过这种方式保留上下文

```
public Flux<String> chatStream(String sessionId, String userMessage) {
        try {
            // 获取历史消息
            List<MessageDto> history = getConversationHistory(sessionId);

            // 添加用户消息
            history.add(new MessageDto("user", userMessage));

            // 转换为Message对象
            List<Message> messages = convertToMessages(history);

            // 使用 final 变量以在 lambda 中使用
            final List<MessageDto> finalHistory = history;

            // 调用AI流式接口
            return chatClient.prompt()
                    .messages(messages)
                    .stream()
                    .content()
                    .doOnNext(content -> {
                        // 流式传输中不保存，在完成后保存
                    })
                    .doOnComplete(() -> {
                        try {
                            // 完成后获取完整回复并保存
                            String assistantReply = chatClient.prompt()
                                    .messages(messages)
                                    .call()
                                    .content();

                            // 保存AI回复
                            finalHistory.add(new MessageDto("assistant", assistantReply));

                            // 限制消息数量
                            List<MessageDto> updatedHistory = finalHistory;
                            if (updatedHistory.size() > maxMessages) {
                                updatedHistory = new ArrayList<>(updatedHistory.subList(updatedHistory.size() - maxMessages, updatedHistory.size()));
                            }

                            // 保存到Redis
                            saveConversationHistory(sessionId, updatedHistory);

                            log.info("Stream chat completed for session: {}, history size: {}", sessionId, updatedHistory.size());
                        } catch (Exception e) {
                            log.error("Error in stream chat onComplete for session: {}", sessionId, e);
                        }
                    })
                    .doOnError(e -> log.error("Error in stream chat for session: {}", sessionId, e));

        } catch (Exception e) {
            log.error("Error in chatStream for session: {}", sessionId, e);
            return Flux.error(e);
        }
    }
```

