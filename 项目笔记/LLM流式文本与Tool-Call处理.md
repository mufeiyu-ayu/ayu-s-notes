# LLM 流式文本与 Tool Call 处理

本文串联三个核心文件：

| 文件                                    | 职责                            |
| ------------------------------------- | ----------------------------- |
| `agent-runtime.service.ts`            | 创建并驱动 sampling，发送文本或执行工具       |
| `openai-compatible-stream.adapter.ts` | 把厂商原始 chunk 转成项目内部事件          |
| `model-sampling-decision.ts`          | 消费内部事件，产出文本或最终 SamplingDecision |

`openai-compatible.client.ts` 只作为模型请求入口简单带过，不是本文重点。

## 一、共同调用链

```ts
const sampling = streamModelSampling(
  this.llmService.chatStream(modelInputItems, chatStreamOptions),
  samplingAttemptId,
)

let samplingResult = await sampling.next()
```

两层都是异步生成器。创建 `sampling` 时还没有请求模型，首次 `sampling.next()` 才启动整条链路：

```text
sampling.next()
→ AgentRuntimeService 驱动 streamModelSampling()
→ llmService.chatStream() 请求模型并得到原始 stream
→ adaptOpenAICompatibleStream() 转换事件
→ streamModelSampling() 消费事件
→ 返回文本片段或最终 SamplingDecision
```

模型请求体大致如下：

```ts
{
  model: 'deepseek-chat',
  messages: [
    { role: 'system', content: '你是 SEO Agent……' },
    { role: 'user', content: '用户消息' },
  ],
  tools: [
    {
      type: 'function',
      function: {
        name: 'search_articles',
        description: '按关键词查询文章……',
        parameters: {
          type: 'object',
          properties: {
            query: { type: 'string' },
            languageCode: { type: 'string' },
            limit: { type: 'integer' },
          },
          required: ['query'],
          additionalProperties: false,
        },
      },
    },
  ],
  stream: true,
  stream_options: { include_usage: true },
}
```

## 二、情况一：模型直接生成普通文本

用户输入：

```text
请给我一个 SEO 标题建议
```

### 1. SDK 返回原始 chunks

```ts
{ choices: [{ delta: { content: '建议标题：' }, finish_reason: null }] }
{ choices: [{ delta: { content: 'SP Himeko 完整攻略' }, finish_reason: null }] }
{ choices: [{ delta: {}, finish_reason: 'stop' }] }
{
  choices: [],
  usage: {
    prompt_tokens: 500,
    completion_tokens: 12,
    total_tokens: 512,
  },
}
```

### 2. 适配器转换为内部事件

```ts
{ type: 'text_delta', delta: '建议标题：' }
{ type: 'text_delta', delta: 'SP Himeko 完整攻略' }
{
  type: 'usage',
  usage: { inputTokens: 500, outputTokens: 12, totalTokens: 512 },
}
{ type: 'response_completed', finishReason: 'stop' }
```

`response_completed` 是项目生成的统一事件，表示原始模型流已经完整结束。

### 3. `streamModelSampling()` 处理事件

收到 `text_delta` 时：

```ts
outputMode = 'final_answer'
yield event.delta
```

因此外层前两次 `sampling.next()` 分别得到：

```ts
{ done: false, value: '建议标题：' }
{ done: false, value: 'SP Himeko 完整攻略' }
```

外层 `while` 将文本累计到 `content`，并通过 `assistant_delta` 实时发给前端。

最后一次 `sampling.next()` 会在内部处理 `usage` 和 `response_completed`，然后得到：

```ts
{
  done: true,
  value: {
    type: 'final_answer',
    summary: {
      samplingAttemptId: 'run_123:sampling-1',
      finishReason: 'stop',
      usage: { inputTokens: 500, outputTokens: 12, totalTokens: 512 },
      toolCallCount: 0,
      textChars: 19,
      intermediateTextChars: 0,
    },
  },
}
```

结果：第一轮模型请求已经产生最终回答，外层结束 sampling 循环，不执行工具。

## 三、情况二：模型生成 Tool Call

用户输入：

```text
帮我查询 SP Himeko 相关文章
```

### 1. SDK 分片返回 Tool Call

```ts
// 工具名称和调用 ID
{
  choices: [{
    delta: {
      tool_calls: [{
        index: 0,
        id: 'call_123',
        function: { name: 'search_articles', arguments: '' },
      }],
    },
    finish_reason: null,
  }],
}

// arguments 碎片
{
  choices: [{
    delta: {
      tool_calls: [{
        index: 0,
        function: { arguments: '{"query":"SP ' },
      }],
    },
    finish_reason: null,
  }],
}

{
  choices: [{
    delta: {
      tool_calls: [{
        index: 0,
        function: { arguments: 'Himeko","limit":5}' },
      }],
    },
    finish_reason: null,
  }],
}

// Tool Call 生成结束，不代表工具已执行
{ choices: [{ delta: {}, finish_reason: 'tool_calls' }] }
```

### 2. 适配器拼接并转换事件

`OpenAICompatibleToolCallAccumulator` 先把参数碎片拼成：

```json
{"query":"SP Himeko","limit":5}
```

然后适配器依次产生：

```ts
{
  type: 'tool_call_completed',
  toolCall: {
    providerCallId: 'call_123',
    name: 'search_articles',
    argumentsJson: '{"query":"SP Himeko","limit":5}',
  },
}
{ type: 'usage', usage: { inputTokens: 520, outputTokens: 20, totalTokens: 540 } }
{ type: 'response_completed', finishReason: 'tool_calls' }
```

含义：

```text
tool_call_completed → 一个完整 Tool Call 请求已经准备好
response_completed  → 这一轮模型流已经完全结束
```

两者都不表示后端工具已经执行成功。

### 3. `streamModelSampling()` 汇总决策

它在同一次 `sampling.next()` 中消费上述事件：

```text
保存完整 Tool Call
→ 保存 usage
→ 保存 finishReason: tool_calls
→ 校验本轮只有一个 Tool Call
→ return tool_call
```

因为中间没有向外 `yield` 文本，外层第一次 `sampling.next()` 就可能直接得到：

```ts
{
  done: true,
  value: {
    type: 'tool_call',
    call: {
      callId: 'call_123',
      toolName: 'search_articles',
      rawArgumentsJson: '{"query":"SP Himeko","limit":5}',
      samplingAttemptId: 'run_123:sampling-1',
    },
    intermediateText: '',
    summary: {
      samplingAttemptId: 'run_123:sampling-1',
      finishReason: 'tool_calls',
      usage: { inputTokens: 520, outputTokens: 20, totalTokens: 540 },
      toolCallCount: 1,
      textChars: 0,
      intermediateTextChars: 0,
    },
  },
}
```

### 4. 外层执行工具并发起第二轮采样

```text
AgentRuntimeService 拿到 tool_call
→ ToolInvocationService 校验并执行 search_articles
→ 得到 ToolResult.modelContent
→ 把 assistant_tool_call 和 tool_result 加入模型输入
→ Sampling #2 再次请求模型
→ 模型以 text_delta + stop 生成最终文本
```

因此当前 `[1, 2]` 的含义是：

```text
Sampling #1：直接回答，或生成一次 Tool Call
Sampling #2：根据工具结果生成最终回答
```

## 四、最简记忆

```text
AgentRuntimeService
→ 创建 sampling，并通过 next() 驱动整条链路

adaptOpenAICompatibleStream
→ 拼接 Tool Call 碎片
→ 输出 text_delta / tool_call_completed / usage / response_completed

streamModelSampling
→ 普通文本：逐段 yield，最后 return final_answer
→ Tool Call：汇总事件，最后 return tool_call

AgentRuntimeService
→ 文本发给前端，Tool Call 交给后端执行
```
