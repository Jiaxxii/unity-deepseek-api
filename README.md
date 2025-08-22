# [请使用最新库: `https://github.com/Jiaxxii/UniDeepSeek`](https://github.com/Jiaxxii/UniDeepSeek)
# DeepSeek Unity Chat Integration

- 本README由deepseek-r1生成

## 概述
本组件为Unity引擎与DeepSeek大语言模型的集成方案，提供以下核心功能：
- 支持标准对话补全（含流式响应）
- 支持前缀续写（FIM，含流式响应）
- 支持普通V3模型与深度思考模型
- 自动对话历史管理
- 多线程安全设计

## 功能特性
### 核心组件
- **ChatProcessor**：基础通信处理器
  - 派生 **DeepseekChat**（对应官网普通V3模型）
  - 派生 **DeepseekReasoner**（对应深度思考模型）
  
### 核心功能
```csharp
// 消息收集器（自动维护对话上下文）
var messagesCollector = new MessagesCollector(...);

// 请求参数体系
ChatMessageRequest / ReasonerMessageRequest / FimRequest

// 三种调用方式：
1. 同步请求：await deepseekChat.ChatCompletionAsync()
2. 流式迭代器：await foreach (var data in ...)
3. 回调式流式：ChatCompletionStreamAsync(onReceiveData: ...)
```

## 使用示例
### 基础配置
```csharp
[SerializeField] private string apiKey; // 在Unity Inspector中配置API Key

// 初始化消息收集器（自动维护对话历史）
var messagesCollector = new MessagesCollector(
    new SystemMessage("你叫「西」，是..."),
    new UserMessage("早上好呀~")
);
```

### 请求参数配置
```csharp
var messageRequest = new ChatMessageRequest(ChatModel.Chat, messagesCollector)
{
    MaxToken = 1024,
    // 你不必手动开启流式配置，因为在请求时会根据调用的方法自动添加或关闭流式配置
    // StreamOptions = new StreamOptions(includeUsage: false)
};
```

### 发起请求
#### 同步请求
```csharp
ChatCompletion response = await deepseekChat.ChatCompletionAsync();
Debug.Log(response.GetMessage().Content);
```

#### 流式响应（迭代器式）
```csharp
await foreach (var data in deepseekChat.ChatCompletionStreamAsync(report => {}))
{
    if (data.HasCompleteMsg())
    {
        Debug.Log(data.GetMessage().Content);
    }
}
```

#### 流式响应（回调式）
```csharp
var response = await deepseekChat.ChatCompletionStreamAsync(data => 
{
    Debug.Log(data.GetMessage().Content);
});
```

### 前缀续写功能
```csharp
var prefixMessage = new AssistantPrefixMessage("开头文本");
var response = await deepseekChat.ChatCompletionAsync(prefixMessage);
```

## 参数说明
### 请求类型
| 类型 | 说明 |
|------|------|
| `ChatMessageRequest` | V3模型专用参数 |
| `ReasonerMessageRequest` | 深度思考模型参数 |
| `FimRequest` | 前缀续写专用参数 |

### 模型选择
```csharp
// 使用深度思考模型
var reasoner = new DeepseekReasoner(apiKey, request);

// 使用普通V3模型
var chat = new DeepseekChat(apiKey, request);
```

## 注意事项
1. **线程安全**：所有方法均支持多线程调用
2. **消息验证**：建议调用 `messagesCollector.CheckAndThrow()`
3. **资源释放**：场景销毁时调用 `ChatProcessor.Dispose()`
4. **流式配置**：无需手动设置StreamOptions
5. **模型兼容**：深度思考模型进行对话前缀续写时不进行思考

## 版本信息
- 当前版本基于 2025年4月18日 API 规范
- 支持 Unity 2021.3+ 版本
- 需要 .NET 4.x 运行时
