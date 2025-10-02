---
description: 通过modelfile文件安装本地大模型
---

# ModelFile

```mdx-code-block
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
```

Ollama Model File 语法：[modelfile.md](https://github.com/facebook/docusaurus/tree/main/packageshttps://github.com/ollama/ollama/blob/main/docs/modelfile.md).

:::tip 背景

本地下载第三方模型后，通过modelfile文件安装后，对话时胡言乱语，且一直不停生成。原因是模型文件中没有正确配置模型模板（Template）和 stop。

因为不同模型系列（如 Llama、Mistral、Qwen 等）的提示词格式可能不同，错误的模板会导致模型回答混乱。

:::

## 常见模型Template，摘自ollama {#requirecommonments}

### Qwen



## 其他Template,来自网络 {#other}

```bash
##格式0

FROM tinyllama-my-model.gguf

### Set the system message
SYSTEM """
You are a super helpful helper.
"""

PARAMETER stop <s>
PARAMETER stop </s>

#格式0的运行方式：ollama run my-model "<s>\nQ: \nWhat is the capital of France?\nA:\n"
```

```bash
##格式1
FROM ./llama3-unsloth.Q8_0.gguf

TEMPLATE """{{- if .System }}
<|system|>
{{ .System }}
{{- end }}
<|user|>
{{ .Prompt }}
<|assistant|>
"""

SYSTEM """You are a helpful, smart, kind, and efficient AI assistant.Your name is Aila. You always fulfill the user's requests to the best of your ability."""


PARAMETER temperature 0.8
PARAMETER num_ctx 8192
PARAMETER stop "<|system|>"
PARAMETER stop "<|user|>"
PARAMETER stop "<|assistant|>"
```

```bash
##格式2
FROM GEITje-7B-chat-v2.gguf
TEMPLATE """{{- if .System }}
<|system|>
{{ .System }}
</s>
{{- end }}
<|user|>
{{ .Prompt }}
</s>
<|assistant|>
"""
PARAMETER temperature 0.2
PARAMETER num_ctx 8192
PARAMETER stop "<|system|>"
PARAMETER stop "<|user|>"
PARAMETER stop "<|assistant|>"
PARAMETER stop "</s>"
```

```bash
##格式3
FROM models/tinyllama-1.1b-chat-v0.3.Q6_K.gguf
PARAMETER temperature 0.7
PARAMETER stop "<|im_start|>"
PARAMETER stop "<|im_end|>"
TEMPLATE """
<|im_start|>system
{{ .System }}<|im_end|>
<|im_start|>user
{{ .Prompt }}<|im_end|>
<|im_start|>assistant
"""
SYSTEM """You are a helpful assistant."""
```

```bash
FROM /root/zml/DeepSeek-R1-Distill-Qwen-7B-Q4_K_M.gguf 
TEMPLATE """
{{- if .System }}{{ .System }}{{ end }} 
{{- range $i, $_:= .Messages }} 
{{- $last := eq (len (slice $.Messages $i)) 1}} 
{{- if eq .Role "user" }}< | User| >{{ .Content }} 
{{- else if eq .Role "assistant" }}< | Assistant | > {{ .Content }}{{- if not $last }}< | end_of_sentence | >{{- end }} 
{{- end }} 
{{- if and $last (ne .Role "assistant") }}<| Assistant | >{{- end }} 
{{- end }}
""" 
PARAMETER stop "<|begin_of_sentence|>" 
PARAMETER stop "<lend_of_sentence|>" 
PARAMETER stop "<|user|>" 
PARAMETER stop "<|Assistant|>"
```

### demo详解

```bash
FROM testmodel.gguf
#设置temperature为1,[更高的数值回答更加发散，更低的数值回答更加保守]。越大的数值回答约有创造性，默认0.8。
PARAMETER temperature 0.7

#回答方式：这个参数会让模型像chatGPT一样 “引入-分点-总结” 的方式进行回答
TEMPLATE """{{ if .System }}<|start_header_id|>system<|end_header_id|>
 
{{ .System }}<|eot_id|>{{ end }}{{ if .Prompt }}<|start_header_id|>user<|end_header_id|>
 
{{ .Prompt }}<|eot_id|>{{ end }}<|start_header_id|>assistant<|end_header_id|>
 
{{ .Response }}<|eot_id|>"""

#设置停止回答：遇到什么情况就停止回答，比如重复说话了，或者怎么怎么地的，库库放进去就完了
PARAMETER stop "<|start_header_id|>"
PARAMETER stop "<|end_header_id|>"
PARAMETER stop "<|eot_id|>"
PARAMETER stop "<|reserved_special_token"
 
 
#设置tokens限制，防止回答重复：num_ctx参数是限制回答的token数量；repeat_penalty参数设置惩罚重复的强度。较高的值（例如，1.5）将对重复进行更严厉的惩罚，而较低的值（例如，0.9）将更宽松。 （默认值：1.1）。repeat_last_n参数设置模型回溯多远以防止重复。 （默认值：64，0 = 禁用，-1 = num_ctx）
PARAMETER num_ctx 4096
PARAMETER repeat_penalty 1.5
PARAMETER repeat_last_n 1024
 
#设置系统级别的提示词：根据自己需要设置，模型会按照类似的方式回答你的问题。system：为模型提供系统消息的替代方式。 user：用户可能会提出的示例消息。 assistant：模型应该如何回应的示例消息。
SYSTEM 现在你是xxxx有限公司矿建领域的个人助理，我是一个矿山建设领域的工程师，你要帮我解决我的专业性问题。
MESSAGE user 你好
MESSAGE assistant 我在，我是xxxx有限公司的矿建电子个人助理，请问有什么我可以帮助您的嘛？
MESSAGE user 人工地层冻结主要采用机械式压缩机制冷技术吗？
MESSAGE assistant 是的，人工地层冻结主要采用机械式压缩机制冷技术。
MESSAGE user 解释人工地层冻结的主要制冷方法。
```


## 长文本处理时，回复的内容像鸟语 {#long}

默认 上下文token上限太低了，请求参数中的options中num_ctx参数，默认值上下文使用token只有4096，大约就是 输入 加上输出的中文一共6000-8000字，超过了就开始说鸟语。
模型一般就是会写，支持上下文 8k，32k，128k， 也就是 8 * 1024个token
例如你的模型支持32k上下文，那就可以设置为 32 * 1024 = 32,768

```bash
PARAMETER num_ctx 4096
```

## 加载模型
```bash
# mymodelname是你自定义的模型名称，可随意设置
ollama create mymodelname -f ./testmodel.modelfile
```