# 7.16-17安排

### Langgraph CLI（如何构建agent/Studio UI监控agent/）

安装Langgraph 本地服务 pip install -U "langgraph-cli[inmem]"

langgraph本地服务启动要求

langgraph.json（运行命令的当前目录下必须存在该配置文件，作用：声明图入口、依赖、环境文件路径。）

# langgraph.json 完整配置规范（LangGraph CLI / LangGraph Platform）

`langgraph.json` 是 LangGraph 官方部署、本地调试、云端托管的核心配置文件，用于定义项目依赖、Graph 入口、环境、镜像、运行参数等。

## 一、基础完整模板（包含你截图里的所有字段 + 全部可选配置）

```
{
  // 可选：IDE语法校验、自动补全（推荐加上）
  "$schema": "https://langgra.ph/schema.json",
  // 【必填】项目依赖列表
  "dependencies": [
    ".",
    "langgraph>=0.2.0",
    "langchain-openai",
    "./libs/shared"
  ],
  // 【必填】定义所有可暴露运行的Graph
  "graphs": {
    "agent": "./src/agent/graph.py:graph",
    "rag_chat": "./src/rag/graph.py:rag_graph"
  },
  // 可选：环境变量文件路径
  "env": ".env",
  // 可选：基础镜像发行版
  "image_distro": "wolfi",
  // 可选：指定Python版本，默认3.11
  "python_version": "3.11",
  // 可选：自定义UI组件入口（前端渲染）
  "ui": {
    "chat_ui": "./ui/index.js"
  },
  // 可选：pip自定义配置文件路径
  "pip_config_file": "./pip.conf",
  // 可选：构建时前置执行命令
  "build_commands": [
    "uv sync",
    "mkdir static"
  ],
  // 可选：运行时资源限制（云端部署）
  "resources": {
    "cpu": "1",
    "memory": "2Gi"
  },
  // 可选：运行超时时间（秒）
  "timeout": 300,
  // 可选：是否启用流式输出压缩
  "enable_stream_compression": true
}
```

## 二、逐字段详细规范说明

### 1. `$schema`（可选，推荐）

- 值：固定字符串 `"https://langgra.ph/schema.json"`
- 作用：VSCode/IDE 自动校验 JSON 格式、提示字段、补全配置，避免写错键名。

### 2. `dependencies`【必填】数组

定义项目安装依赖，支持三种写法：

1. **本地目录**：`"."` 代表当前项目根目录（读取 `pyproject.toml` / `requirements.txt`）；`"./libs/utils"` 导入本地子包
2. **pip 包名**：`"langgraph>=0.2.0"`、`"langchain-anthropic"` 标准 pip 依赖写法
3. **Git 依赖**：`"git+https://xxx.git"`

示例：

```
"dependencies": [
  ".",
  "langgraph>=0.2.30",
  "langchain-openai>=0.1.5",
  "./common"
]
```

### 3. `graphs`【必填】对象

注册所有可启动的 LangGraph 图实例，**键 = 图名称（运行时标识），值 = 文件：变量路径**

#### 路径格式规范：`文件相对路径:导出的Graph变量名`

以你截图为例：

```
"agent": "./src/agent/graph.py:graph"
```

- `./src/agent/graph.py`：相对 `langgraph.json` 的源码文件
- `graph`：该文件中导出的 `StateGraph`/`CompiledGraph` 变量名（必须是编译完成的图）

支持多图同时注册：

```
"graphs": {
  "chat_agent": "./src/chat/graph.py:chat_graph",
  "tool_agent": "./src/tools/graph.py:tool_graph"
}
```

运行时通过 `langgraph dev --graph chat_agent` 指定启动对应图。

### 4. `env`（可选）字符串

- 值：`.env`、`./config/.env.prod` 环境变量文件相对路径
- 作用：本地调试 / 云端部署自动加载该文件内的密钥、API 地址等环境变量；多环境可写多个 env 文件切换。 你截图配置 `"env": ".env"` 代表读取项目根目录 `.env`。

### 5. `image_distro`（可选）字符串

构建容器镜像时的基础发行版，官方支持两个值：

- `wolfi`（默认，轻量安全，推荐，你截图使用此值）
- `debian`（传统 Debian 镜像，体积更大）

### 6. `python_version`（可选）字符串

指定容器内 Python 版本，支持 `3.10` / `3.11` / `3.12`，默认 `3.11`

```
"python_version": "3.12"
```

### 7. `ui`（可选）对象

自定义对话前端 UI 组件，键为 UI 标识，值为前端 JS 入口文件路径，仅 LangGraph Studio / 云端生效：

```
"ui": {
  "custom_chat": "./frontend/main.js"
}
```

### 8. `build_commands`（可选）字符串数组

镜像构建阶段前置执行的 shell 命令，适合预处理、静态资源打包：

```
"build_commands": [
  "uv pip compile requirements.in",
  "npm run build"
]
```

### 9. `resources`（可选）对象（云端部署专用）

限制运行实例 CPU、内存资源：

```
"resources": {
  "cpu": "0.5",
  "memory": "1Gi"
}
```

### 10. `timeout`（可选）数字

单轮对话 / 图执行最大超时秒数，默认 300 秒。

### 11. `pip_config_file`（可选）字符串

自定义 pip 镜像源配置文件路径，如 `./pip.conf`。

### 12. `enable_stream_compression`（可选）布尔

开启 SSE 流式输出 gzip 压缩，减少网络流量，默认`false`。

原始配置：

```
{
  "dependencies": ["."],
  "graphs": {
    "agent": "./src/agent/graph.py:graph"
  },
  "env": ".env",
  "image_distro": "wolfi"
}
```

对应含义：

1. `dependencies: ["."]`：读取当前项目根目录的 Python 依赖（requirements/pyproject.toml）
2. `graphs.agent`：注册名为`agent`的图，入口文件 `src/agent/graph.py`，文件内导出变量名 `graph`
3. `env: ".env"`：自动加载根目录环境变量文件
4. `image_distro: "wolfi"`：构建镜像使用轻量 Wolfi 基础系统

## 四、最小可用最简配置（仅必填字段）

```
{
  "$schema": "https://langgra.ph/schema.json",
  "dependencies": ["."],
  "graphs": {
    "agent": "./graph.py:graph"
  }
}
```

## 五、使用命令配套说明

写完 `langgraph.json` 后常用 CLI 命令：

1. 本地调试：`langgraph dev`（默认启动 graphs 下第一个图）

2. 指定图启动：`langgraph dev --graph agent`

3. 构建部署镜像：`langgraph build`

4. 导出生产配置：`langgraph export`

   

LangSmith API Key （不配置密钥也能启动服务，但无法使用 Studio 可视化、链路追踪功能）

```
LANGSMITH_API_KEY=ls_xxxx
# 可选关闭追踪（不连LangSmith云端）
LANGSMITH_TRACING=false
```



执行 `langgraph dev`启动服务时，**终端工作目录必须是 langgraph.json 所在根目录**



### embedding模型微调





### ayncio线程操作理解



# 7.19

## 大模型微调

来源：[如何把你的 DeePseek-R1 微调为某个领域的专家？](https://mp.weixin.qq.com/s/gkkJTnAZVu81EK8H5DAjQw)

### 一、微调在解决什么问题

通用模型懂很多，但不一定懂你的领域话术、回答风格或复杂指令。微调就是用领域数据给基座模型“补课”，改少量参数，让它更贴场景。

和另外两条路别混：

| 方式 | 比喻 | 适合 |
|------|------|------|
| 长文本 | 当场读完超长阅读理解 | 长文写作/通读 |
| 知识库（RAG） | 开卷查资料 | 要最新、可更新的事实 |
| **微调** | 考前辅导班 | 风格、专业话术、固定推理套路 |

### 二、完整流程（概念层）

```text
选基座模型 → 准备标注数据 → 微调前基线测试
    → 设超参（LoRA + 训练参数）→ 训练
    → 同题复测 → 不满意就改数据/超参
    → 导出（GGUF）→ 上传 HF → Ollama/API 部署
```

三个核心概念：

1. **预训练模型**：已会通用能力的“学生”。文中用 `DeepSeek-R1-Distill-Llama-8B`（也可用 `unsloth/DeepSeek-R1-Distill-Qwen-7`）。
2. **数据集**：补课教材。平台常用 OpenAI 风格 `messages`（jsonl）；本文推理模型额外要有思考链字段 `Complex_CoT`。
3. **超参数**：补课强度。重点三个：
   - **Epochs**：过几遍数据；太少学不透，太多易过拟合
   - **Learning Rate**：每次更新幅度；文中 `2e-4`
   - **Batch Size**：每次看多少条；文中等价 `2 × 4 = 8`

### 三、路径 A：平台微调（硅基流动）

步骤：选模型（如 Qwen2.5-7B）→ 上传 `.jsonl` → 划 10% 验证集 → 设超参 → 训完拿模型名调用。

**数据格式（jsonl，一行一条）：**

```json
{"messages":[{"role":"system","content":"你是一位算命大师"},{"role":"user","content":"1992年...想了解健康运势"},{"role":"assistant","content":"...专业回答..."}]}
```

**调用微调后模型：**

```python
from openai import OpenAI

client = OpenAI(
    api_key="您的 APIKEY",  # https://cloud.siliconflow.cn/account/ak
    base_url="https://api.siliconflow.cn/v1",
)

messages = [{"role": "user", "content": "用当前语言解释微调模型流程"}]

response = client.chat.completions.create(
    model="您的微调模型名",
    messages=messages,
    stream=True,
    max_tokens=4096,
)

for chunk in response:
    print(chunk.choices[0].delta.content, end="")
```

平台限制：可选模型少、按 Token 计费、排队不可控 → 文中转 Colab + Unsloth。

### 四、路径 B：Colab + Unsloth 实战（核心代码）

环境：Colab → 运行时改为 **T4 GPU**。数据集：[Conard/fortune-telling](https://huggingface.co/datasets/Conard/fortune-telling)（字段：`Question` / `Response` / `Complex_CoT`）。

#### 1）安装依赖

```python
!pip install unsloth
!pip uninstall unsloth -y && pip install --upgrade --no-cache-dir --no-deps git+https://github.com/unslothai/unsloth.git
!pip install bitsandbytes unsloth_zoo
```

#### 2）加载 4bit 基座模型

```python
from unsloth import FastLanguageModel
import torch

max_seq_length = 2048
dtype = None
load_in_4bit = True

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/DeepSeek-R1-Distill-Llama-8B",
    max_seq_length=max_seq_length,
    dtype=dtype,
    load_in_4bit=load_in_4bit,
)
```

#### 3）微调前基线测试

```python
prompt_style = """以下是描述任务的指令，以及提供进一步上下文的输入。
请写出一个适当完成请求的回答。在回答之前，请仔细思考问题，并创建一个逻辑连贯的思考过程，以确保回答准确无误。

### 指令：
你是一位精通卜卦、星象和运势预测的算命大师。请回答以下算命问题。
### 问题：
{}
### 回答：
<think>
{}"""

question = "1992年闰四月初九巳时生人，女，想了解健康运势"

FastLanguageModel.for_inference(model)
inputs = tokenizer(
    [prompt_style.format(question, "")],
    return_tensors="pt",
).to("cuda")

outputs = model.generate(
    input_ids=inputs.input_ids,
    attention_mask=inputs.attention_mask,
    max_new_tokens=1200,
    use_cache=True,
)
print(tokenizer.batch_decode(outputs)[0])
```

#### 4）加载并格式化数据集

```python
from datasets import load_dataset

train_prompt_style = """以下是描述任务的指令，以及提供进一步上下文的输入。
请写出一个适当完成请求的回答。在回答之前，请仔细思考问题，并创建一个逻辑连贯的思考过程，以确保回答准确无误。

### 指令：
你是一位精通八字算命、紫微斗数、风水、易经卦象、塔罗牌占卜、星象、面相手相和运势预测等方面的算命大师。请回答以下算命问题。
### 问题：
{}
### 回答：
<think>
{}
</think>
{}
"""

EOS_TOKEN = tokenizer.eos_token

dataset = load_dataset(
    "Conard/fortune-telling",
    split="train[0:200]",
    trust_remote_code=True,
)
print(dataset.column_names)  # ['Question', 'Response', 'Complex_CoT']

def formatting_prompts_func(examples):
    inputs = examples["Question"]
    cots = examples["Complex_CoT"]
    outputs = examples["Response"]
    texts = []
    for question, cot, output in zip(inputs, cots, outputs):
        text = train_prompt_style.format(question, cot, output) + EOS_TOKEN
        texts.append(text)
    return {"text": texts}

dataset = dataset.map(formatting_prompts_func, batched=True)
print(dataset["text"][0])
```

要点：推理模型要把 **思考过程** 一并训进去，否则只学到“答”，学不到“怎么推”。

#### 5）LoRA 准备 + 训练参数 + 开训

文中关键数：`lr=2e-4`，`batch=2×4=8`，`max_steps≈70`，约 `70×8/200 ≈ 3` 个 epoch。

```python
from unsloth import FastLanguageModel, is_bfloat16_supported
from trl import SFTTrainer
from transformers import TrainingArguments

FastLanguageModel.for_training(model)

model = FastLanguageModel.get_peft_model(
    model,
    r=16,
    target_modules=[
        "q_proj", "k_proj", "v_proj", "o_proj",
        "gate_proj", "up_proj", "down_proj",
    ],
    lora_alpha=16,
    lora_dropout=0,
    bias="none",
    use_gradient_checkpointing="unsloth",
    random_state=3407,
    use_rslora=False,
    loftq_config=None,
)

trainer = SFTTrainer(
    model=model,
    tokenizer=tokenizer,
    train_dataset=dataset,
    dataset_text_field="text",
    max_seq_length=4096,
    dataset_num_proc=2,
    packing=False,
    args=TrainingArguments(
        per_device_train_batch_size=2,
        gradient_accumulation_steps=4,
        warmup_steps=5,
        max_steps=70,
        learning_rate=2e-4,
        fp16=not is_bfloat16_supported(),
        bf16=is_bfloat16_supported(),
        logging_steps=1,
        optim="adamw_8bit",
        weight_decay=0.01,
        lr_scheduler_type="linear",
        seed=3407,
        output_dir="outputs",
        report_to="none",
    ),
)

trainer_stats = trainer.train()
```

#### 6）微调后同题复测

```python
FastLanguageModel.for_inference(model)

question = "1992年闰四月初九巳时生人，女，想了解健康运势"
inputs = tokenizer(
    [prompt_style.format(question, "")],
    return_tensors="pt",
    padding=True,
    truncation=True,
).to("cuda")

outputs = model.generate(
    input_ids=inputs.input_ids,
    attention_mask=inputs.attention_mask,
    max_new_tokens=1200,
    pad_token_id=tokenizer.eos_token_id,
    temperature=0.7,
    top_p=0.9,
    use_cache=True,
)

response = tokenizer.batch_decode(
    outputs[:, inputs.input_ids.shape[1]:],
    skip_special_tokens=True,
)[0]
print(response)
```

#### 7）导出 GGUF 并上传 Hugging Face

Colab Secrets 里建 `HUGGINGFACE_TOKEN`（写权限），或直接填 token：

```python
from huggingface_hub import create_repo
from google.colab import userdata

hf_token = userdata.get("HUGGINGFACE_TOKEN")  # 或直接字符串
repo_id = "你的用户名/fortunetelling"

create_repo(repo_id, token=hf_token, exist_ok=True)

# 转 GGUF 并推到 Hub（适合 Ollama）
model.push_to_hub_gguf(
    repo_id,
    tokenizer,
    token=hf_token,
    quantization_method="q4_k_m",
)
```

#### 8）本地 Ollama 运行

```bash
ollama run hf.co/Conard/fortunetelling
# 换成你自己的：ollama run hf.co/{username}/{repository}
```

也可接到 Chatbox、AnythingLLM 等客户端。

### 五、参数怎么调（对照原文）

| 参数 | 文中取值 | 作用 |
|------|----------|------|
| `learning_rate` | `2e-4` | 学得快慢；过大易崩，过小学得慢 |
| `per_device_train_batch_size` × `gradient_accumulation_steps` | `2 × 4 = 8` | 等效 batch |
| `max_steps` | `70` | 总步数；配合数据量算 epoch |
| LoRA `r` / `lora_alpha` | `16` / `16` | 可训练容量；新手可先不动 |
| `load_in_4bit` | `True` | T4 上能塞进 8B |

效果不好时优先：**加高质量数据 / 改角色提示词**，再动 lr 和 steps。

### 六、流程串起来

```text
基座模型 4bit → 领域数据 + CoT → 微调前测试
→ LoRA + SFTTrainer → 微调后对比
→（不满意改数据）→ GGUF → HF → Ollama / API
```

### 七、超参数调优如何进行

核心原则：**先锁定数据质量，再小范围改“学多快、学几遍、每次看多少”**，用固定验证集对比，不要一次改一堆参数。

#### 1. 先定评估方式（否则调了也不知道好不好）

- 划出 **验证集**（约 10%～20%），训练过程中不碰
- 固定一批 **同题对比**（微调前/后各跑一遍）
- 看三类信号：
  - **训练 loss**：是否稳定下降
  - **验证表现**：是否更准、更贴风格，而不是只会背训练样本
  - **过拟合迹象**：训练很好、验证变差，或一换说法就不会

#### 2. 优先调的 3 个超参

| 超参 | 管什么 | 怎么调 |
|------|--------|--------|
| **Learning Rate** | 每次更新幅度 | 先从常用区间试：LoRA/SFT 常见 `1e-4`～`2e-4`（文中 `2e-4`）。过大：loss 震荡/崩；过小：学不动 |
| **Epochs / max_steps** | 学几遍 | 先小步（如 1～3 epoch）。不够再加；验证变差就停（早停） |
| **Batch Size** | 每次看多少条 | 显存不够就用梯度累积：`真实 batch ≈ per_device_batch × grad_accum`（文中 `2×4=8`） |

经验规则：

- **一次只改一类**（先 lr，再 steps，再 batch）
- 改完跑一轮短训，用同一验证题对比
- 小白可先用默认，再微调

#### 3. LoRA 相关（影响大，但别一上来猛拧）

| 参数 | 建议 | 说明 |
|------|------|------|
| `r` | 8～32，常用 16 | 越大容量越强，也越容易过拟合、越吃显存 |
| `lora_alpha` | 常取 `r` 或 `2r` | 实际缩放≈`alpha/r`；和 `r` 一起调 |
| `target_modules` | 注意力 + FFN 更稳 | 只训 `q/v` 更省，效果可能弱一点 |
| `lora_dropout` | 0～0.1 | 数据少、过拟合时再开 |

新手顺序：**先不动 LoRA，只调 lr + steps**；效果不够再加大 `r`。

#### 4. 推荐调优流程

1. **基线**：固定数据、固定验证题，用默认（如 `lr=2e-4`，约 1～3 epoch，`batch≈8`）训一版
2. **调学习率**（最重要）：试 `5e-5` / `1e-4` / `2e-4` / `5e-4`，选 loss 平滑下降且验证最好的
3. **调训练量**：在最佳 lr 上试不同 `max_steps` / epochs，验证开始变差就停
4. **调 batch**（显存允许时）：用梯度累积凑到 8～32 很常见
5. **再动 LoRA**：欠拟合增大 `r` 或加数据；过拟合减 epochs、加 dropout、减 `r`、加数据多样性
6. **最后才动次要项**：warmup、scheduler（linear/cosine）、weight_decay

#### 5. 现象 → 动作对照

| 现象 | 可能原因 | 动作 |
|------|----------|------|
| loss 不降 / 几乎不动 | lr 太小，或数据/格式有问题 | 先查数据，再略增大 lr |
| loss 暴涨、震荡 | lr 太大 | 降 lr（如减半） |
| 训练很好，验证差 / 只会背答案 | 过拟合或数据太少 | 减 steps、加验证相关新题、加数据多样性、减 `r` |
| 微调后几乎没变化 | 学不够或任务提示不一致 | 加 epochs、略增 lr、检查训练模板与推理模板是否一致 |
| 显存爆 | batch/序列太长 | 降 `per_device_batch`，加 `grad_accum`，开 4bit/gradient checkpointing |

一句话：**用固定验证集，先找合适的学习率，再找合适的训练步数，LoRA 参数最后动；一次只改一处。** 超参见效差时，优先回到数据和角色提示词。

### 八、数据集与角色提示词：怎么看好坏、怎么改

数据集和角色提示词比超参更重要。好坏不靠“感觉写得漂亮”，而靠：**模型训完后，在未见过的同类问题上，是否稳定地按你想要的身份、格式、推理方式回答**。

#### 1. 怎么看数据集好不好

**（1）先看结构是否统一（最容易翻车）**

好数据通常满足：

- 字段齐全、含义固定（如 `Question` / `Complex_CoT` / `Response`）
- 每条都是「输入 →（可选思考）→ 输出」，没有混进闲聊、广告、半截答案
- **训练模板和推理模板一致**（角色、标题、`<think>` 等标签前后一样）

差信号：

- 同一字段有时中文有时英文、有时空、有时长到爆炸
- 有的答很短，有的答几千字且风格完全不同
- 问题和答案对不上（答非所问）

**（2）再看内容质量（比数量重要）**

逐条抽检（建议先人工看 20～50 条）：

| 检查项 | 好 | 差 |
|--------|----|----|
| 任务对齐 | 每条都服务同一目标（如“算命大师问答”） | 混进翻译、闲聊、无关百科 |
| 答案正确/合理 | 逻辑自洽，关键信息对 | 事实错、自相矛盾、胡编 |
| 风格一致 | 语气、结构、专业度接近 | 有的像大师，有的像搜索引擎 |
| 可学模式 | 有重复可迁移的套路 | 每条都是特例，学不到规律 |
| 覆盖面 | 覆盖主要问法/边界情况 | 只有一种问法，换说法就崩 |
| 噪声 | 几乎没有乱码、重复、截断 | 复制粘贴痕迹重、大量重复 |

经验：**100 条干净一致的数据，往往优于 2000 条脏杂数据。**

**（3）训前诊断**

1. 随机抽 30 条，按上表打分（对齐/正确/风格）
2. 看多样性：问题长度、问法、主题是否太单一
3. 若有 CoT：思考是否真在“推理”，而不是把最终答案塞进思考里糊弄
4. 做 hold-out：留 10%～20% 验证题；问法应在训练分布内，但原文 ideally 不重复

验证集上若“只会背原句、换说法就不会”，多半是数据同质或过拟合，不是 lr 的问题。

#### 2. 数据集怎么改（按优先级）

**优先级 1：清洗与对齐**

- 删：答非所问、空答案、明显错误、重复近乎一样的样本
- 统一：标点、称呼、输出结构（例如固定：先结论 / 先分析再结论）
- 修正：角色串台（一会儿医生一会儿算命）

**优先级 2：补“你真正在意的行为”**

| 模型问题 | 怎么补数据 |
|----------|------------|
| 太短 | 加「完整结构」的优质长答 |
| 太啰嗦 | 加「简洁但完整」样本，并在提示里写清长度要求 |
| 换说法就不会 | 同一意图写 3～5 种问法（复述/口语/缺信息） |
| 边界差 | 加拒答、信息不足、冲突条件样本 |
| 推理弱 | 加高质量 CoT（步骤清楚），且 **CoT 与最终答案一致** |

**优先级 3：控制比例**

- 核心场景占多数（如 70%～80%）
- 难例/边界 10%～20%
- 通用闲聊尽量少（防把角色冲淡）

**优先级 4：扩充方式**

- 人工精标最好
- 用强模型生成后，**必须人工抽检纠错**
- 合成数据常见坑：套路雷同、幻觉、格式漂移——宁缺毋滥

改完做一次 **小集合冒烟训**（如 100～200 条），同题对比；有提升再扩。

#### 3. 怎么看角色提示词好不好

好提示词应同时满足：

1. **身份清晰**：你是谁、擅长什么、不做什么
2. **输出契约明确**：先想再答？分点？必须含哪些字段？
3. **约束可执行**：少空话（“专业、友好”），多可检查规则
4. **与数据一致**：数据里的助手表现，就是提示词描述的那个人
5. **训练/推理同一套**：微调时用的指令模板，推理时不要换皮

差提示词常见形态：

- 太虚：`你是一个乐于助人的 AI`（学不到领域行为）
- 太长太杂：几十条互相冲突的规矩
- 与数据打架：提示说“先给结论”，数据却总是先抒情后结论
- 只写身份不写任务：没有“收到什么输入、产出什么输出”

自检 5 问：

1. 陌生人只看提示词，能否复述出期望行为？
2. 去掉提示词，数据里的答案是否仍像同一个人？
3. 提示要求的结构，在 ≥80% 样本里是否真的出现？
4. 有没有互相矛盾的要求？
5. 推理时用的提示，是否几乎原样来自训练模板？

#### 4. 角色提示词怎么改

推荐结构（短而硬）：

```text
【身份】你是……（领域 + 能力边界）
【任务】根据用户问题，完成……
【推理】回答前先进行逐步分析（可用 <think>...</think>）
【输出格式】
1. …
2. …
【约束】
- 信息不足时先追问 / 明确说明假设
- 不编造无法从输入推出的关键事实
- 语气：……
【禁止】
- …
```

现象对照：

| 现象 | 提示词怎么改 | 数据怎么配合 |
|------|--------------|--------------|
| 不像目标角色 | 身份写具体（技能清单 + 说话方式） | 样本统一成该语气 |
| 格式乱 | 写死输出模板 + 举例 1 条 | 每条按模板重写 |
| 乱编 | 加“不足则追问/标注不确定” | 加拒答与不确定样本 |
| 太啰嗦/太短 | 写长度与详略规则 | 用对应风格样本“投票” |
| 推理模型不思考 | 强制 CoT 标签与步骤 | 每条提供真 CoT，别空壳 |

原则：**提示词负责“规定”；数据负责“示范”。** 规定了但数据里没示范 → 学不会；示范了但提示不要求 → 推理时容易漂。

#### 5. 可执行的迭代闭环

```text
1. 写清目标行为（3～5 条验收标准）
2. 写短提示词 + 做 30 条黄金样本（人工精品）
3. 小训一版 → 固定 20 道验证题打分
4. 归类失败：角色漂 / 格式错 / 知识错 / 不会泛化
5. 针对失败类型改数据（主）或改提示（辅）
6. 再训 → 对比同分表
```

验收标准示例：

- 身份符合（是/否）
- 结构符合（是/否）
- 关键信息合理（1～5）
- 是否胡编（是/否）
- 换一种问法是否仍稳（是/否）

#### 6. 硬规则

1. 先定验收标准，再写数据和提示
2. 提示与数据必须同源：训练里出现的角色句，推理原样用
3. 先精品小集，再扩量
4. 每次只改一类问题（先修格式，再修风格，再补难例）
5. CoT 要真推理：思考过程不能和最终答案矛盾
6. 超参见效差时，先回到数据和提示，别空转 lr

一句话：**好数据 = 目标一致 + 答案靠谱 + 风格统一 + 覆盖真实问法；好提示词 = 身份清 + 格式死 + 约束可查 + 与数据零冲突。** 用固定验证题打分驱动修改。

### 九、主流微调工具/平台操作流程（含核心代码）

共性步骤（所有工具几乎一样）：

```text
准备数据 → 选基座模型 → 配微调方式(LoRA等)与超参 → 训练
→ 验证对比 → 导出/合并 → 推理部署
```

下面按 **代码型** / **平台型** 写「大致能跟着做」的流程。命令与字段随版本可能微调，以各项目官方文档为准。

---

#### A. 代码型

##### 1）Unsloth（单卡极速 / Colab）

适合：个人学习、单卡 PoC（与上文「路径 B」相同）。

```text
装依赖 → 4bit 加载模型 → 前测 → 格式化数据集
→ get_peft_model(LoRA) → SFTTrainer.train()
→ 后测 → push_to_hub_gguf / 本地保存 → Ollama
```

核心骨架（精简版）：

```python
# 1. 安装
# !pip install unsloth
# !pip install --upgrade --no-cache-dir --no-deps git+https://github.com/unslothai/unsloth.git

from unsloth import FastLanguageModel, is_bfloat16_supported
from datasets import load_dataset
from trl import SFTTrainer
from transformers import TrainingArguments

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/DeepSeek-R1-Distill-Llama-8B",
    max_seq_length=2048,
    load_in_4bit=True,
)

model = FastLanguageModel.get_peft_model(
    model, r=16, lora_alpha=16, lora_dropout=0, bias="none",
    target_modules=["q_proj","k_proj","v_proj","o_proj","gate_proj","up_proj","down_proj"],
    use_gradient_checkpointing="unsloth",
)

# dataset = load_dataset(...); 格式化出 text 字段后：
trainer = SFTTrainer(
    model=model, tokenizer=tokenizer, train_dataset=dataset,
    dataset_text_field="text", max_seq_length=2048,
    args=TrainingArguments(
        per_device_train_batch_size=2, gradient_accumulation_steps=4,
        learning_rate=2e-4, max_steps=70, fp16=not is_bfloat16_supported(),
        bf16=is_bfloat16_supported(), optim="adamw_8bit", output_dir="outputs",
        report_to="none",
    ),
)
trainer.train()
# model.push_to_hub_gguf("user/repo", tokenizer, token="hf_xxx")
```

---

##### 2）LLaMA-Factory（WebUI + YAML，最常用工程入口）

适合：少写代码、多模型切换、团队统一流水线。

```text
克隆/安装 → 注册数据集(dataset_info.json) → 写 YAML 或开 WebUI
→ llamafactory-cli train → chat 验证 → export 合并 LoRA
```

安装与启动 WebUI：

```bash
git clone https://github.com/hiyouga/LLaMA-Factory.git
cd LLaMA-Factory
pip install -e .

# 可视化（点选模型/数据/超参）
llamafactory-cli webui
```

注册数据：把 json/jsonl 放到 `data/`，在 `data/dataset_info.json` 登记，例如：

```json
{
  "my_sft": {
    "file_name": "my_sft.json",
    "formatting": "sharegpt",
    "columns": { "messages": "messages" }
  }
}
```

ShareGPT / messages 示例（`my_sft.json` 一条）：

```json
{
  "messages": [
    {"role": "system", "content": "你是领域助手"},
    {"role": "user", "content": "用户问题"},
    {"role": "assistant", "content": "标准回答"}
  ]
}
```

训练 YAML 示例（`my_lora_sft.yaml`）：

```yaml
### model
model_name_or_path: Qwen/Qwen2.5-7B-Instruct
trust_remote_code: true

### method
stage: sft
do_train: true
finetuning_type: lora
lora_rank: 8
lora_target: all

### dataset
dataset: my_sft
template: qwen
cutoff_len: 2048
max_samples: 1000
overwrite_cache: true
preprocessing_num_workers: 4

### output
output_dir: saves/qwen25-7b/lora/sft
logging_steps: 10
save_steps: 200
plot_loss: true
overwrite_output_dir: true

### train
per_device_train_batch_size: 1
gradient_accumulation_steps: 8
learning_rate: 1.0e-4
num_train_epochs: 3.0
lr_scheduler_type: cosine
warmup_ratio: 0.1
bf16: true
```

命令：

```bash
llamafactory-cli train my_lora_sft.yaml
llamafactory-cli chat examples/inference/qwen3_lora_sft.yaml   # 按官方推理 yaml 改路径
llamafactory-cli export examples/merge_lora/qwen3_lora_sft.yaml # 合并 LoRA 导出完整权重
```

可把 Unsloth 作为加速后端（版本支持时在配置里开启），兼顾界面与速度。

---

##### 3）Axolotl（YAML + 多卡工程化）

适合：多 GPU、配置可复现、要接 FSDP/DeepSpeed。

```text
安装 → 准备 jsonl → 写 axolotl yaml → accelerate/axolotl train → 导出 adapter
```

安装（示意）：

```bash
pip install axolotl
# 或按官方文档用 Docker / pip install -e '.[flash-attn,...]'
```

数据（常见 instruction 格式）：

```json
{"instruction": "解释微调", "input": "", "output": "微调是在预训练模型上用领域数据继续训练..."}
```

配置片段（`sft.yml` 示意）：

```yaml
base_model: Qwen/Qwen2.5-7B-Instruct
model_type: AutoModelForCausalLM
tokenizer_type: AutoTokenizer

load_in_4bit: true
adapter: lora
lora_r: 16
lora_alpha: 16
lora_target_modules:
  - q_proj
  - v_proj
  - k_proj
  - o_proj

datasets:
  - path: data/train.jsonl
    type: alpaca
dataset_prepared_path: last_run_prepared
val_set_size: 0.1

sequence_len: 2048
sample_packing: true

micro_batch_size: 2
gradient_accumulation_steps: 4
num_epochs: 3
learning_rate: 0.0002
optimizer: adamw_bnb_8bit
lr_scheduler: cosine

bf16: auto
tf32: true
gradient_checkpointing: true
output_dir: ./outputs/qwen-lora
```

训练：

```bash
accelerate launch -m axolotl.cli.train sft.yml
# 或：axolotl train sft.yml
```

---

##### 4）Hugging Face（Transformers + PEFT + TRL）

适合：要深度改数据管线 / loss / 自定义逻辑。

```text
from_pretrained → LoraConfig + get_peft_model → SFTTrainer → save_pretrained
```

```python
from datasets import load_dataset
from peft import LoraConfig, get_peft_model
from transformers import AutoModelForCausalLM, AutoTokenizer, TrainingArguments
from trl import SFTTrainer

model_id = "Qwen/Qwen2.5-7B-Instruct"
tokenizer = AutoTokenizer.from_pretrained(model_id, trust_remote_code=True)
model = AutoModelForCausalLM.from_pretrained(
    model_id, torch_dtype="auto", device_map="auto", trust_remote_code=True
)

peft_config = LoraConfig(
    r=16, lora_alpha=32, lora_dropout=0.05,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj"],
    task_type="CAUSAL_LM",
)
model = get_peft_model(model, peft_config)

dataset = load_dataset("json", data_files="train.jsonl", split="train")
# 需自行 map 成 messages / text，与 chat_template 对齐

args = TrainingArguments(
    output_dir="out-hf-lora",
    per_device_train_batch_size=1,
    gradient_accumulation_steps=8,
    learning_rate=1e-4,
    num_train_epochs=3,
    logging_steps=10,
    bf16=True,
)

trainer = SFTTrainer(
    model=model,
    args=args,
    train_dataset=dataset,
    processing_class=tokenizer,  # 新版 TRL；旧版可能是 tokenizer=
)
trainer.train()
model.save_pretrained("out-hf-lora/adapter")
tokenizer.save_pretrained("out-hf-lora/adapter")
```

---

##### 5）MS-SWIFT（魔搭，Qwen 等国产模型友好）

适合：ModelScope 取模取数、主做通义/Qwen。

```text
pip install ms-swift → 准备数据 → swift sft → swift infer / 导出
```

```bash
pip install ms-swift -U

# LoRA SFT 示意（参数名以当前 CLI --help 为准）
swift sft \
  --model Qwen/Qwen2.5-7B-Instruct \
  --dataset './data/train.jsonl' \
  --train_type lora \
  --torch_dtype bfloat16 \
  --num_train_epochs 2 \
  --per_device_train_batch_size 1 \
  --gradient_accumulation_steps 8 \
  --learning_rate 1e-4 \
  --lora_rank 8 \
  --lora_alpha 32 \
  --output_dir output/qwen-swift \
  --eval_steps 100 \
  --save_steps 100
```

数据常用 messages / sharegpt 风格；也可用 ModelScope 上的公开数据集 ID。

---

#### B. 平台型

##### 6）硅基流动（SiliconFlow）

适合：没卡、快速验证业务。

```text
注册 → 微调页新建任务 → 选基座 → 上传 jsonl
→ 设验证集比例与超参 → 启动训练 → 拿到模型名 → OpenAI 兼容 API 调用
```

数据（jsonl，一行一条 messages）：

```json
{"messages":[{"role":"system","content":"你是领域助手"},{"role":"user","content":"问题"},{"role":"assistant","content":"回答"}]}
```

调用（训完后）：

```python
from openai import OpenAI

client = OpenAI(
    api_key="你的APIKEY",
    base_url="https://api.siliconflow.cn/v1",
)
resp = client.chat.completions.create(
    model="你的微调模型名",
    messages=[{"role": "user", "content": "测试问题"}],
    stream=False,
)
print(resp.choices[0].message.content)
```

注意：可选基座有限、可能排队、按量计费；大数据量任务更建议迁代码型。

---

##### 7）阿里云百炼 / PAI

适合：政企、通义生态、要全链路运维。

**百炼（偏 MaaS 点选）：**

```text
控制台 → 模型微调/定制 → 选通义基座 → 上传数据（按控制台模板）
→ 配置超参 → 提交训练 → 部署为在线服务 → SDK/API 调用
```

**PAI-DLC / PAI 训练（偏工程）：**

```text
准备 OSS 数据 → 创建训练任务（选镜像/资源）→ 指定启动命令
（可为 LLaMA-Factory / DeepSpeed 脚本）→ 产出写入 OSS → 再用 PAI-EAS 部署
```

调用侧一般是百炼/DashScope 或 EAS 的 HTTP/SDK；具体字段以当前控制台文档为准。思路与硅基类似：训完拿 endpoint + api_key 调 chat。

---

##### 8）火山方舟

适合：已在字节云 / 豆包生态。

```text
方舟控制台 → 微调/精调 → 选基座 → 上传数据集 → 配 SFT/DPO 等
→ 训练 → 发布为推理接入点 → 用方舟 SDK / OpenAPI 调用
```

流程形态与硅基/百炼同属「托管训练 + 托管推理」。

---

##### 9）OpenAI / Azure OpenAI Fine-tuning

适合：必须用 GPT 系且合规允许。

```text
准备 jsonl → 上传文件 → 创建 fine-tune job → 等状态 succeeded
→ 用新 model id 调用 chat.completions
```

训练数据格式：

```json
{"messages":[{"role":"system","content":"You are a helpful assistant."},{"role":"user","content":"Hello"},{"role":"assistant","content":"Hi!"}]}
```

```python
from openai import OpenAI
client = OpenAI()  # Azure 则配 azure_endpoint / api_version / azure_ad_token_provider

# 1) 上传
file = client.files.create(file=open("train.jsonl", "rb"), purpose="fine-tune")

# 2) 创建任务（模型名以账号可用列表为准，如 gpt-4.1-mini-2025-04-14 等）
job = client.fine_tuning.jobs.create(
    training_file=file.id,
    model="gpt-4o-mini-2024-07-18",
)

# 3) 轮询 job.status == "succeeded" 后取 fine_tuned_model
ft_model = client.fine_tuning.jobs.retrieve(job.id).fine_tuned_model

# 4) 调用
print(client.chat.completions.create(
    model=ft_model,
    messages=[{"role": "user", "content": "测试"}],
).choices[0].message.content)
```

---

##### 10）Hugging Face AutoTrain（半托管）

适合：HF 上快速试 LoRA。

```text
AutoTrain 页面/Space → 选基座与任务 → 上传/选数据集 → 训练
→ 得到 Hub 上的 adapter/repo → transformers/peft 或 Inference Endpoint 加载
```

也可用 AutoTrain Advanced 本地/云命令行（版本变化快，跟官方 README）。

---

#### C. 选型对照（做哪个流程）

| 目标 | 走哪条流程 |
|------|------------|
| Colab 单卡学会全流程 | Unsloth |
| 日常多模型、要 WebUI | LLaMA-Factory |
| 多卡可复现生产训练 | Axolotl / PAI+自有脚本 |
| 深度定制算法 | HF Transformers+PEFT+TRL |
| 主做 Qwen/魔搭 | MS-SWIFT |
| 没卡先看业务效果 | 硅基流动 / 百炼 / 方舟 |
| 必须 GPT | OpenAI / Azure Fine-tuning |
| 数据不能出内网 | 任一代码型 + 自有 GPU |

本地部署微调结果（代码型常见收尾）：

```bash
# 合并后的 GGUF / HF 仓库示例
ollama run hf.co/{username}/{repository}
```

或用 vLLM / llama.cpp 加载合并权重提供 OpenAI 兼容接口。

# 7.20

## Harness（工程控制架）工程架构笔记（Offer-Agent（模拟面试智能体项目） / 七层骨架）

核心原则：**LLM（大语言模型）只负责生成；Harness（工程控制架/外围骨架）负责决策、控流、校验与止损。**

评价标准：停得下来、控得住、查得清。

七层总览（后续分层补记）：

1. Control Plane（控制平面） — PLAN（规划） → EXECUTE（执行） → VERIFY（校验门禁）
2. Memory（记忆层）
3. Context（上下文层）
4. Sub-Agent（子智能体）层
5. RAG（检索增强生成）层
6. Skills（技能层）
7. MCP（模型上下文协议）层

---

## 第一层：Control Plane（控制平面）

### 1. 这一层在解决什么问题

传统 Agent（智能体）容易把「下一步做什么、何时结束」也交给模型的「自觉」，结果是：死循环、不可审计、失败后只会「再聊一轮」。

控制平面用状态机把每一步固定成：

```text
PLAN → EXECUTE → VERIFY
         ↑           │
         └── Replan ←┘（未超阈值）
              ↓（超阈值 / fatal）
           Hard Stop
```

- **PLAN（规划）**：本轮目标、步骤、停止条件 —— 由**代码/策略**写出，不是模型自由发明计划
- **EXECUTE（执行）**：调用 LLM（大语言模型）或工具，只做「这一步」的生成/检索
- **VERIFY（门禁）**：对产物做确定性检查；不过关则 Replan，超阈值则硬停机

与「Run 生命周期状态机」（pending → running → completed）不同：后者管整次任务起止；控制平面管 **running 内部每一步** 的控流。

### 2. PLAN（规划）：本轮目标、步骤、停止条件如何由代码写出

#### 2.1 PLAN（规划）产出的是可执行对象，不是自由文本

```python
@dataclass
class Step:
    id: str
    kind: str              # "retrieve" | "generate" | "score" | "tool"
    goal: str              # 本步要达成什么（日志/审计）
    inputs: dict           # 从 memory/context 取什么
    output_schema: dict    # 期望结构（给 VERIFY 用）
    max_retries: int = 2

@dataclass
class StopConditions:
    max_steps: int = 8
    max_replans: int = 2
    max_wall_seconds: int = 60
    success_when: list[str]   # 如 ["has_valid_question", "not_duplicate"]
    fail_when: list[str]      # 如 ["candidate_quit", "budget_exceeded"]

@dataclass
class Plan:
    round_goal: str           # 本轮总目标
    steps: list[Step]         # 有序步骤
    stop: StopConditions      # 硬停止条件
    on_fail: str              # "replan" | "abort" | "degrade"
```

字段谁写、谁能动（粗分；细拆见 2.4）：

| 字段 | 谁写 | 能否让 LLM（大语言模型）改 |
|------|------|----------------|
| `round_goal` / 任务等**语义字段** | LLM（大语言模型）可填（或模板 + LLM（大语言模型）），须过 Gate（门禁） | 建议可填，最终以 Gate（门禁）/合并为准 |
| `steps` 顺序 / `stop` / 其它**控流字段** | **策略代码硬规则** | 否（冲突以代码为准） |
| 题干措辞、追问文案等生成内容 | LLM（在 EXECUTE（执行）） | 是，但仍受 schema（结构约定）约束 |

#### 2.2 三种常见「写出」方式（可叠加）

**写法 A：纯规则表（面试题序最典型）**

```python
STAGE_PLANS = {
    "warmup": Plan(
        round_goal="出一道开场自我介绍题",
        steps=[
            Step("s1", "retrieve", "取开场题模板", ...),
            Step("s2", "generate", "按简历改写题面", output_schema=QUESTION_SCHEMA),
        ],
        stop=StopConditions(
            max_steps=3, max_replans=1,
            success_when=["schema_ok", "difficulty_in_range"],
            fail_when=["empty_bank"],
        ),
        on_fail="replan",
    ),
    "deep_dive": Plan(...),
}

def plan(state) -> Plan:
    return STAGE_PLANS[state.stage]  # 目标来自 stage，不是模型想出来的
```

**写法 B：策略函数（带状态分支）**

```python
def plan_next_question(state) -> Plan:
    # --- 目标：代码决定 ---
    if state.wrong_streak >= 2:
        goal, difficulty = "降难度出巩固题", state.difficulty - 1
    elif state.time_left_min < 5:
        goal, difficulty = "出收尾总结题", state.difficulty
    else:
        goal, difficulty = "按题序出下一题", state.difficulty

    # --- 步骤：代码拼装 ---
    steps = [
        Step("filter", "retrieve", "按方向/未考过过滤", inputs={"difficulty": difficulty}),
        Step("adapt", "generate", "简历个性化改题", output_schema=QUESTION_SCHEMA),
    ]

    # --- 停止条件：代码写死阈值 ---
    stop = StopConditions(
        max_steps=len(steps) + 2,
        max_replans=2,
        max_wall_seconds=45,
        success_when=["schema_ok", "not_asked_before", "difficulty_match"],
        fail_when=["no_candidate_left", "user_abort"],
    )
    return Plan(round_goal=goal, steps=steps, stop=stop, on_fail="replan")
```

这里所有 if/else 都是策略，模型不参与选分支。

**写法 C：LLM（大语言模型）只填「内容槽」，控流仍由代码锁死**

```python
# 允许：模型返回枚举（偏项目 / 偏算法）
hint = llm.classify(state.last_answer)

# 不允许：模型返回 {steps, stop} 然后直接执行
plan = build_plan(stage=state.stage, focus=hint)  # 仍走代码模板
```

一句话：**控流字段必须来自 `build_plan` / 规则表，不能来自模型自由生成的 JSON（数据交换格式）计划。**

#### 2.3 停止条件两类

1. **预算类**：步数、重规划次数、墙钟时间 → 到了就 hard_stop（硬停机）
2. **成功/失败谓词**：`success_when` / `fail_when` → 布尔函数，不是「模型觉得差不多了」

#### 2.4 PLAN（规划）混合结构：语义字段 vs 控流字段

PLAN（规划）结果通常是一份**混合结构**：业务语义字段可以让 LLM（大语言模型）填，控流字段应由代码/规则写入并校验，不能交给模型「自决」。

##### 为什么要拆成两类字段

| 类型 | 谁写 | 原因 |
|------|------|------|
| **语义字段** | LLM（或规则模板 + LLM（大语言模型）润色） | 「本轮问什么、追什么、怎么评」需要理解力 |
| **控流字段** | 代码硬规则 / 状态机 | 「能不能走、走哪条边、何时停」必须可判定、可审计、可强制 |

否则模型既写目标又写「我可以无限重试」，死循环和越权就回来了。

##### 语义字段（常由 LLM（大语言模型）填）

典型例如：

- **本轮目标**（goal）：如「考察候选人 Redis 缓存设计」
- **本轮任务**（tasks）：如「出一道追问 / 对上一答评分」
- **理由 / 假设**（rationale）：为何选这个考点
- **期望产出形态**（expected_output）：简答、打分表、是否需要追问点
- **个性化提示**（persona hints）：结合简历改题的要点

这些可以进 prompt（提示词），但**最终仍要过 Gate（门禁）**：格式对不对、是否在允许的题型集合里、是否与当前状态机阶段一致。

##### 控流字段是什么

**控流字段 = 驱动状态机边转移、门禁、预算与权限的字段。**  
它们回答的不是「说什么」，而是：

- 现在处在哪个状态？
- 下一步允许哪些动作？
- 失败怎么走？
- 还剩多少预算？
- 谁有权写全局状态？

由 **Harness（工程控制架）在 PLAN（规划）前后用代码写入/覆盖**，LLM（大语言模型）最多「建议」，**不能成为最终真相**。

##### 控流字段一般有什么（实用清单）

**1. 状态与阶段**

- `phase`：`PLAN | EXECUTE | VERIFY`（或更细：`ASK / SCORE / FOLLOW_UP / END`）
- `session_status`：`running | paused | completed | aborted`
- `step_id` / `turn_id`：本步唯一 ID，便于审计

**2. 允许的动作（白名单）**

- `allowed_actions`：如 `["ask_question", "score", "follow_up", "end"]`
- `forbidden_actions`：显式禁止（例如评分阶段禁止改题库）
- `selected_action`：**最终动作**——通常由规则在候选里选定，或 LLM（大语言模型）从白名单里选一个，代码再校验

**3. 路由与转移**

- `next_phase_on_success`：成功去哪（如 `EXECUTE` → `VERIFY`）
- `next_phase_on_failure`：失败去哪（如 `VERIFY` fail → `PLAN` / Replan）
- `transition_guard`：转移条件（分数阈值、是否已覆盖必考题等）——**布尔表达式由代码求值**

**4. 预算与停机（防死循环核心）**

- `retry_count` / `max_retries`
- `replan_count` / `max_replans`
- `loop_fingerprint`：本轮动作+目标哈希，用于检测重复循环
- `token_budget_remaining` / `time_budget_ms`
- `hard_stop`：`true` 则强制结束，不再调模型

**5. 门禁（门禁）相关**

- `gates_required`：本步必须过哪些门（格式、安全、业务）
- `gate_policy`：失败策略 `replan | repair | abort`
- `verification_schema`：VERIFY（校验门禁）用的 JSON（数据交换格式） Schema / 规则 ID（代码侧）

**6. 权限与隔离**

- `actor` / `sub_agent_id`：本轮由哪个子 Agent（智能体）执行
- `tool_allowlist`：可调工具（MCP（模型上下文协议）/题库）范围
- `write_scopes`：可写哪些记忆分区（只能写 working，不能改 procedural 等）

**7. 幂等与审计**

- `idempotency_key`
- `plan_version` / `policy_version`
- `parent_plan_id`：Replan 时指向上一版
- `trace_id`

**8. 业务进度（偏控流的「进度机」）**

在 Offer-Agent（模拟面试智能体项目）这类场景里，常由代码维护、PLAN（规划）只读：

- `question_index` / `required_coverage`
- `difficulty_band`（可由规则根据得分曲线调）
- `must_ask_remaining`：必考清单剩余
- `can_end`：是否满足结束条件

这些看起来像业务，但本质是**状态机条件**，应用规则更新。

##### 一份 PLAN（规划）长什么样（示意）

```json
{
  "semantic": {
    "goal": "考察 Redis 缓存穿透处理",
    "tasks": ["基于简历项目改一题", "准备 2 个追问点"],
    "rationale": "候选人简历有秒杀系统经历"
  },
  "control": {
    "phase": "PLAN",
    "selected_action": "ask_question",
    "allowed_actions": ["ask_question", "follow_up", "end"],
    "sub_agent_id": "question_gen",
    "tool_allowlist": ["mcp.question_bank.search"],
    "write_scopes": ["working", "episodic"],
    "max_retries": 2,
    "replan_count": 0,
    "max_replans": 3,
    "token_budget_remaining": 12000,
    "next_phase_on_success": "EXECUTE",
    "next_phase_on_failure": "PLAN",
    "gate_policy": "replan",
    "hard_stop": false,
    "can_end": false,
    "plan_version": 3,
    "parent_plan_id": "plan_17"
  }
}
```

约定建议：

1. LLM（大语言模型） **只输出 `semantic` + 可选的 `selected_action`（必须 ∈ `allowed_actions`）**
2. Harness（工程控制架） **合并/覆盖全部 `control`**，冲突以代码为准
3. VERIFY（校验门禁）时：语义可修，**控流违规直接 fail / abort，不让模型改 `hard_stop`、`max_*`**

##### 小结

- **是**：本轮目标、任务等语义字段常由 LLM（大语言模型）填。
- **控流字段**：状态、白名单动作、成功/失败路由、重试与预算、Gate（门禁）策略、子 Agent（智能体）/工具权限、写作用域、停机与审计 ID 等。
- **原则**：LLM（大语言模型）提议「做什么内容」；代码决定「能不能做、做几次、失败去哪、何时必须停」。

### 3. VERIFY（校验门禁）：如何创建门禁（门禁）

Gate（门禁） ≠ 再问一次模型「你觉得对不对」。  
Gate（门禁） = **一组确定性 Checker（硬编码 / 注册表化）**，对 EXECUTE（执行）产物判布尔/校验结构，汇总成裁决。  
**主路径是硬编码门禁**；可带规则引擎/schema（结构约定）库，但 **pass/fail 不交给「再问一次大模型觉不觉得行」当唯一裁决**。可选加模型质量分作参考特征，不能单独决定放行。

#### 3.1 裁决结构

```python
@dataclass
class GateVerdict:
    status: Literal["pass", "replan", "hard_stop"]
    reasons: list[str]
    failed_checks: list[str]

def verify(plan, state, artifacts) -> GateVerdict:
    results = [check(plan, state, artifacts) for check in GATE_REGISTRY]
    failed = [r for r in results if not r.ok]

    if not failed:
        return GateVerdict("pass", [], [])
    if any(r.fatal for r in failed):
        return GateVerdict("hard_stop", [r.name for r in failed], ...)
    return GateVerdict("replan", [r.name for r in failed], ...)
```

裁决含义：

```text
全过           → pass      → commit / 进入下一步
有失败且非 fatal → replan    → 改参数重跑（骨架仍由代码定）
fatal 或超预算  → hard_stop
```

#### 3.2 主要校验什么（硬编码门禁清单）按优先级，VERIFY（校验门禁）通常校这些类：

**（1）结构与契约（几乎每步都有）**

- 输出是否符合本步 `output_schema`（缺字段、类型错、多余非法字段）
- 是否是允许的 JSON（数据交换格式）/枚举值
- 工具调用结果是否写进约定结构  

不过 → 通常 `replan` / 同一步重试。

**（2）控流与预算（防死循环、防拖死）**

- 是否超时、是否超过 `max_retries` / `max_replans` / token（词元）·墙钟预算
- 是否出现循环指纹（同一 action+焦点反复失败）
- `hard_stop` 条件是否触发  

不过 → 常直接 `hard_stop` 或降级。

**（3）权限与隔离（防越权）**

- 本步是否只用了 `tool_allowlist` 内工具（执行期已拦；VERIFY（校验门禁）可再审计痕迹）
- 是否写了不允许的 Memory（记忆） kind / key
- 产出是否越出本 Agent（智能体）职责（如生成器产物里塞了只能由别的 Agent（智能体）写的字段）  

不过 → 多视为 fatal / 丢弃产物。

**（4）业务不变量（按领域注册）**

- 必填业务字段是否齐全
- 数值范围、状态机合法性（不能跳到非法状态）
- 与当前 `stage` / Plan 目标是否一致（类型、难度带等约束）不过 → `replan` 或由**代码**回退到澄清步（不是模型改流程）。

**（5）一致性与去重**

- 与 Working（工作记忆） / Episodic（情节记忆）中已有结果是否重复（id、指纹、近重复）
- 是否与 `tool_cache` / 权威查询结果矛盾（若做了核对查询）

**（6）有据（若本步用了 RAG（检索增强生成））**

- `citation_ids` ⊆ 本步已有的 `rag_hits`（对照引用，**不重跑一轮给提示词供料的 RAG（检索增强生成）**）
- 关键断言是否要求必须带引用（策略可配）

**（7）安全与合规**

- 敏感信息泄露、注入、违禁内容
- 对外高风险动作（资金、删除等）是否满足二次确认标记  

不过 → 多为 `hard_stop` 或转人工。

一句话：**结构契约、预算控流、权限隔离、业务不变量、去重一致性、RAG（检索增强生成）有据、安全合规。**

#### 3.3 VERIFY（校验门禁）会不会「检索」？和 RAG（检索增强生成）的关系

**可以查，但主业不是再跑一轮给模型补材料的 RAG（检索增强生成）。**

| 检查 | 数据从哪来 | 要不要再检索 |
|------|------------|--------------|
| schema（结构约定） / 格式 | 本步 artifacts | 否 |
| grounded（有据） | Working（工作记忆）里**已有的** `rag_hits` | 否（对照 id） |
| 去重 / 已用过 | 本地集合、DB、Episodic（情节记忆） | 可能 **查库**（校验查询） |
| 权限 / 预算 / 超时 | state、control | 否 |
| 业务规则 | 规则函数 + state | 一般否 |

```text
RAG（EXECUTE / retrieve 步）：取出材料 → Working → Context → 生成
VERIFY：看生成是否合格；grounded 复用本次 rag_hits，不重开供料检索
```

- **RAG（检索增强生成）**：为生成**供料**  
- **VERIFY（校验门禁）**：对产物**判合格**；需要时可以做只读核对/查重查询，目的是门禁不是装配提示词  

若 Checker（检查器）需要再访问外部（指纹库、权威 API（应用程序接口）、黑白名单）：

1. 仍走代码 Checker（检查器），不交给「VERIFY（校验门禁）专用大模型自由搜」  
2. 工具范围极窄、只读、有超时  
3. 与「给生成补上下文的 RAG（检索增强生成）」分开实现，避免混成同一步  

#### 3.4 典型 Checker（检查器）示例

| Checker（检查器） | 检查什么 | 失败通常走向 |
|---------|----------|--------------|
| `schema_ok` | 是否符合 `output_schema`（如 Pydantic） | replan |
| `not_duplicate` | id / 指纹是否已出现过 | replan |
| `in_range` / 业务约束 | 是否在 PLAN（规划）/stage 指定区间或合法集合 | replan |
| `grounded` | 引用是否来自本步 RAG（检索增强生成）/工具结果 | replan |
| `role_boundary` | 产物是否越权混入他职字段 | hard_stop（硬停机）或丢弃 |
| `safety` | 敏感/违禁内容 | hard_stop（硬停机） |
| `latency_budget` | 本步是否超时 | hard_stop（硬停机） |

```python
def check_schema(plan, state, artifacts):
    try:
        OutModel.model_validate(artifacts["main"])
        return CheckResult(ok=True, name="schema_ok")
    except ValidationError as e:
        return CheckResult(ok=False, name="schema_ok", detail=str(e))

def check_grounded(plan, state, artifacts):
    used = set(artifacts["main"].get("citation_ids", []))
    allowed = {h["id"] for h in state.working.get("rag_hits", [])}
    return CheckResult(ok=used <= allowed, name="grounded")

def check_not_duplicate(plan, state, artifacts):
    oid = artifacts["main"]["id"]
    return CheckResult(ok=oid not in state.seen_ids, name="not_duplicate")
```

#### 3.5 创建门禁的工程要点

1. **挂在状态机边沿上**：只有 VERIFY（校验门禁）=pass 才能 `executing → committed`；失败走 `replanning` 或 `aborted`
2. **检查器注册表化**：新增规则 = 往 `GATE_REGISTRY` 加函数，不改主循环
3. **失败可归因**：`reasons=["not_duplicate"]` 直接喂给 `replan_fn`（如排除已用 id）
4. **禁止用 LLM（大语言模型）当唯一 Gate（门禁）**：可加软质量分作参考，但 pass/fail 必须以规则为准

Replan 同样是代码策略：

```python
def replan_fn(state, reasons: list[str]) -> Plan:
    plan = plan_next(state)  # 骨架仍来自代码模板
    if "not_duplicate" in reasons:
        plan.steps[0].inputs["exclude_ids"] = state.seen_ids
    if "grounded" in reasons:
        plan.steps[0].inputs["require_retrieve"] = True
    return plan
```

### 4. 主循环如何把 PLAN（规划） / EXECUTE（执行） / VERIFY（校验门禁）串起来

```python
def run_round(state):
    plan = plan_fn(state)          # PLAN
    replans = 0

    while True:
        # --- 硬停止：不经过 LLM ---
        if timed_out(plan.stop) or replans > plan.stop.max_replans:
            return hard_stop(state, reason="budget")
        if any(pred(state) for pred in plan.stop.fail_when):
            return hard_stop(state, reason="fail_when")

        # --- EXECUTE ---
        artifacts = {}
        for step in plan.steps:
            artifacts[step.id] = execute(step, state, artifacts)

        # --- VERIFY ---
        verdict = verify(plan, state, artifacts)

        if verdict.status == "pass":
            apply(state, artifacts)
            return "ok"

        if verdict.status == "replan" and plan.on_fail == "replan":
            replans += 1
            plan = replan_fn(state, verdict.reasons)  # 仍是代码策略
            continue

        return hard_stop(state, reason=verdict.reasons)
```

### 5. 一轮「出下一题」示例

```text
PLAN
  goal:  按题序出一道中等难度后端题
  steps: retrieve → generate(改题)
  stop:  max_replans=2, success=[schema, 未重复, 难度]

EXECUTE
  retrieve: MCP/RAG 返回候选
  generate: LLM 改写题面 → artifacts

VERIFY (Gate)
  schema_ok? ✓
  not_asked_before? ✗  → verdict=replan

REPLAN（仍是代码）
  exclude 已考 id，重新 PLAN

EXECUTE #2 … → VERIFY #2 → pass → 写入 memory，进入下一 stage
```

若第 3 次仍失败或超时 → **hard_stop（硬停机）**，进入可恢复中断态，而不是继续 while True 调模型。

### 6. 与「让模型自己 PLAN（规划）」对比

| | 代码/策略 PLAN（规划） + VERIFY（校验门禁） | 模型自规划 |
|--|-------------------------|------------|
| 目标从哪来 | stage / 规则 / 策略函数 | 自由文本 |
| 步骤谁定 | 固定或表驱动 | 模型临时 invent |
| 何时停 | StopConditions + Gate（门禁） | 「我觉得做完了」 |
| 失败怎么办 | replan 策略 / 硬停 | 再聊一轮，易死循环 |
| 可审计性 | plan + verdict 可落库 | 难复盘 |

第一层要把 **目标、步骤、停止、门禁** 全部变成代码可执行、可测试、可计数的对象；LLM（大语言模型）只出现在某个 `Step.kind == "generate"` 的 EXECUTE（执行）里。

### 7. 一句话收束

**PLAN（规划） = 策略代码写出的 Plan 对象（目标/步骤/停止条件）；VERIFY（校验门禁） = 注册表化的硬编码 Checker（检查器），主校结构契约、预算控流、权限隔离、业务不变量、去重一致性、RAG（检索增强生成）有据、安全合规，产出 pass / replan / hard_stop（硬停机）。** 供料靠 RAG（检索增强生成），放行靠 VERIFY（校验门禁）——这是 Harness（工程控制架）「停得下来、控得住」的总闸。

---

## 第二层：Memory Layer（记忆层）

### 1. 这一层在解决什么问题

控制平面每步只关心「当前 Plan + 本步产物」。  
Memory（记忆）负责把**跨步、跨会话仍有用的状态**沉淀下来，并在 PLAN（规划） / Context（上下文）装配时按需取出。

要回答的工程问题：

- 什么该记住、存在哪
- 谁能写（`write_scopes`）
- 何时忘掉
- 取出来怎么喂给下一步（常再交给 Context（上下文）压缩）

```text
Control Plane / Sub-Agent
        │ read / write（受 write_scopes 约束）
        ▼
   Memory Facade
   ├── WorkingStore      （本轮/当前任务工作区）
   ├── EpisodicStore     （事件轨迹）
   ├── SemanticStore     （稳定事实/知识条目）
   └── ProceduralStore   （可复用流程/策略）
        │
        ▼（取用时常再交给 Context 层压缩）拼进下一步 prompt / 工具入参
```

和「聊天历史」的区别：聊天历史是原始对话流；Memory（记忆）是**结构化、分库、带读写策略**的状态。历史可以是 Episodic（情节记忆）的一种来源，但不是 Memory（记忆）的全部。

### 2. 四类 Memory（记忆）：按生命周期 + 用途拆

| 类型 | 存什么（抽象） | 典型生命周期 | 常见介质 |
|------|----------------|--------------|----------|
| **Working（工作记忆）** | 当前正在推进的那一小撮：本步输入、中间结果、未提交草稿 | 极短；一步或一轮结束后可清空/归档 | 进程内存 / Redis |
| **Episodic（情节记忆）** | 「发生过什么」：按时间的事件序列 | 中等；会话级或任务级 | DB / append-only log |
| **Semantic（语义记忆）** | 「世界里相对稳定的事实」：实体、属性、关系、结论条目 | 长；可跨会话 | 向量库 + KV/文档库 |
| **Procedural（程序记忆）** | 「怎么做」：可复用的流程、策略参数、技能配置 | 很长；偏配置，少被对话改写 | 配置库 / 代码+版本化配置 |

### 3. 统一条目形态与 Facade（门面接口）

```python
@dataclass
class MemoryItem:
    id: str
    kind: str              # working | episodic | semantic | procedural
    content: dict          # 结构化载荷，尽量别只塞纯文本
    embedding: list[float] | None
    created_at: float
    updated_at: float
    last_accessed_at: float
    importance: float      # 0~1，写入时由规则或打分器给定
    strength: float        # 当前「鲜活度」，衰减用
    tags: list[str]
    source: str            # 哪次 plan/step/tool 写入
    ttl_seconds: int | None
```

```python
class MemoryFacade:
    def write(self, kind: str, content: dict, *, scope_ok: set[str], **meta) -> str: ...
    def read(self, kind: str, query: Query) -> list[MemoryItem]: ...
    def update(self, id: str, patch: dict) -> None: ...
    def forget(self, predicate) -> int: ...   # 定时或步后触发
    def snapshot(self, task_id: str) -> dict: ...  # 审计/回放
```

**写权限**：Control Plane（控制平面）的 `write_scopes` 决定本步只能写哪些 kind。这是硬规则，不是 prompt（提示词）。  
LLM（大语言模型）不负责「记得对不对」；它最多生成待写入候选，**是否入库、进哪库、何时删除，全是代码。**

### 4. Working（工作记忆）：当前一轮控流的草稿区

**范围**：当前尚未提交的工作草稿，通常覆盖同一次 round / 同一次 plan 尝试里的：

```text
PLAN 产物 + EXECUTE 中间结果 + VERIFY 裁决（+ 同轮 REPLAN 的临时状态）
```

- **写入**：EXECUTE（执行）产物先落 working；VERIFY（校验门禁） pass 后再**晋升**；不通过则丢弃或标 failed
- **读取**：下一步 PLAN（规划）/EXECUTE（执行）默认只读本 task 的 working
- **REPLAN（重规划）时**：Working（工作记忆） **通常不清空**，在同一轮里改 plan、加重试计数
- **本轮结束**（VERIFY（校验门禁） pass，或最终 fail/abort）：归档后清空

Working（工作记忆）是**易变草稿区**，不要当长期真相源。

### 5. Working（工作记忆） → Episodic（情节记忆）：本轮结束后如何归档

VERIFY（校验门禁）结束（或本轮明确结束）后：

1. 抽结构化事件（action、结果、gate 原因、关键产出引用）
2. 可选再做一轮摘要
3. 写入 Episodic（情节记忆）
4. **清空或归档 Working（工作记忆）**

```text
round 开始 → 初始化 working
PLAN / EXECUTE / VERIFY / REPLAN… 都读写 working
VERIFY 结束（pass 或最终 fail/abort）
  → emit episodic events（+ 可选 summary）
  → clear working
```

**不是必须把 Working（工作记忆）全量原文搬进 Episodic（情节记忆）**——常见是压缩/结构化后再写入。

Episodic（情节记忆）要点：

- **结构**：append-only 事件流
- **写入**：每步/每轮结束由 Harness（工程控制架）打点，不是让模型「回忆一下写日记」
- **读取**：按时间窗、`task_id`、事件类型过滤；需要摘要时交给 Context（上下文）压缩
- **清理**：按保留窗口裁切（最近 N 事件或 T 天）

Episodic（情节记忆）回答的是 **what happened**，便于审计和防重复动作（可配合 `loop_fingerprint`）。

### 6. 什么会进 Semantic（与 Episodic（情节记忆）的分界）

| | Episodic（情节记忆） | Semantic（语义记忆） |
|--|----------|----------|
| 问的问题 | **发生过什么？** | **现在认为什么成立？** |
| 形态 | 时间线上的事件 | 可检索的事实/实体/关系 |
| 典型 | 「第 3 步调用工具 X，Gate（门禁）因 schema（结构约定）失败后 replan」 | 「用户时区=UTC+8」「项目默认模型=…」「实体 A 状态=active」 |

**不会**「所有历史都进 Semantic（语义记忆）」。只有从经历里**提炼出、值得跨轮复用的稳定结论**才进。

常见三类来源：

**A. 用户/系统的稳定偏好与设定**

- 偏好语言、默认工具、输出格式约定
- 来源：显式设置，或多次行为归纳后的 upsert

**B. 关于外部世界/任务对象的事实**

- 实体属性、关系、当前状态
- 来源：工具返回的结构化字段、VERIFY（校验门禁）通过后的可靠抽取
- 例：API（应用程序接口）返回 `user_id=42, plan=pro` → 写入 semantic；不是把整段 JSON（数据交换格式）对话塞进去

**C. 可泛化的结论（有门槛）**

- 从多次 Episodic（情节记忆）里蒸馏：「策略 B 在条件 C 下更稳」
- 通常要：**重复出现 / 高置信 / 人工或规则确认**，不是每步都写

**刻意不进 Semantic（语义记忆）的：**

- 一次性中间草稿、失败尝试的噪声
- 纯过程日志（那是 Episodic（情节记忆））
- 未过 Gate（门禁）的模型胡话
- 强时效、下一轮就变的东西（可留 Working（工作记忆）/Episodic（情节记忆），或 semantic 带短 TTL）

关系记成：

```text
Episodic：日志型「经过」
Semantic：知识型「结论/事实」（可 upsert、可检索）
Working →（本轮结束）→ 主要养 Episodic
Episodic / 工具结果 →（抽取/蒸馏，有条件）→ 养 Semantic
```

Semantic（语义记忆）实现要点：

- 条目 = 文本/结构化事实 + embedding（向量嵌入） + 元数据
- 写入要过滤；同一 `entity_key` 做 upsert；冲突用版本或「新覆盖旧但留历史」
- 读取：向量召回 → 元数据过滤 → top-k（前k条）

### 7. Procedural（程序记忆）：流程记忆（怎么做）存的是 **「这类任务标准怎么做」**：步骤模板、分支、失败策略、参数——偏配置/剧本，不是某次运行的故事。

- **写入**：发布配置、Admin/策略引擎更新；运行时 Agent（智能体） **极少**直接改
- **读取**：PLAN（规划）时 `build_plan()` 读当前 version 的 procedure
- **清理**：版本淘汰，不是时间衰减忘掉「怎么做」
- 常被实现成配置表，而不是向量库；与 Skills（技能） / Control Plane（控制平面）紧贴

#### 例子：文档入库流水线

```yaml
# procedural: ingest_document@v3
name: ingest_document
version: 3
steps:
  - id: validate
    kind: check
    rules: [has_file, size_lt_20mb, mime_in_whitelist]
  - id: extract
    kind: tool
    tool: parser.extract_text
  - id: chunk
    kind: tool
    tool: indexer.chunk
    params: { max_tokens: 512, overlap: 64 }
  - id: embed_and_upsert
    kind: tool
    tool: vector.upsert
on_fail:
  validate: abort
  extract: retry_then_abort
  chunk: abort
  embed_and_upsert: retry
budgets:
  max_retries: 2
  max_wall_seconds: 120
```

四类在同一次任务里如何分工：

| 类型 | 这份任务里是什么 |
|------|------------------|
| **Procedural（程序记忆）** | 上面这份「先校验再抽取再切片再入库」——跨任务复用，版本升级才改 |
| **Working（工作记忆）** | 这一次文件的路径、抽取正文草稿、当前 step |
| **Episodic（情节记忆）** | 这次 `validate ok → extract ok → …` 的事件链 |
| **Semantic（语义记忆）**（可能） | `doc_id=D1` 已入库、`hash=...`、所属 `collection=...` 等稳定事实 |

再例（退款 SOP（标准作业程序））：

```text
procedural: "refund_request"
  1) 校验订单状态 ∈ {paid, shipped}
  2) 计算可退金额
  3) 调用支付退款
  4) 写账务凭证
  on_fail: 任何一步失败 → 补偿/回滚，禁止跳步
```

这不是某次退款的日记（那是 Episodic（情节记忆）），而是**退款这件事的标准作业程序**。

### 8. 遗忘：时间衰减怎么做

「时间衰减遗忘」是**定时或每步后跑一遍**的维护任务，不是模型自己忘：

```text
strength(t) = importance * exp(-λ * age) * access_boost
```

- `age`：距 `last_accessed_at` 或 `updated_at`
- `λ`：按 kind 不同（working 极大，episodic 中等，semantic 小，procedural 基本不衰减）
- `access_boost`：被读到就加强

淘汰策略（可叠加）：

1. `strength < 阈值` → 删或降冷存储
2. 超过 `ttl_seconds` → 删
3. 容量上限 → 按 `strength` 淘汰
4. Working（工作记忆）：任务/轮次结束直接清空，不走衰减公式

```python
def decay_and_forget(store, now, kind_policy):
    for item in store.all():
        age = now - item.last_accessed_at
        item.strength = item.importance * exp(-kind_policy.lambda_ * age)
        if item.ttl and now - item.created_at > item.ttl:
            store.delete(item.id)
        elif item.strength < kind_policy.min_strength:
            store.archive_or_delete(item.id)
```

### 9. 与 Control / Context（上下文）的边界

| 层 | 职责 |
|----|------|
| **Memory（记忆）** | 存、取、衰减、权限、晋升（working → episodic/semantic） |
| **Control Plane（控制平面）** | 决定本步能否写、写哪些 kind；VERIFY（校验门禁）通过才 commit（提交落库） |
| **Context（上下文）** | 从 Memory（记忆） **取出的候选**再压缩/裁剪进模型窗口；KV-cache（键值缓存）复用的是拼好的前缀，不是 Memory（记忆）本身 |

典型一步：

```text
PLAN 读 procedural + 少量 semantic/episodic
  → EXECUTE 读写 working，可能读 semantic
  → VERIFY pass
  → commit：working 关键字段 → episodic 事件；可抽取的事实 → semantic upsert
  → forget 维护任务（异步也可）
```

### 10. 最小可落地实现

1. **Working（工作记忆）**：内存 dict，按 `task_id`
2. **Episodic（情节记忆）**：SQLite/Postgres 一张 `events` 表，只追加
3. **Semantic（语义记忆）**：`items` 表 + 向量索引（或先用 tag/key 查询）
4. **Procedural（程序记忆）**：YAML（配置格式）/DB 配置，启动加载
5. **Facade（门面接口） + write_scopes（写作用域） + 每步/每轮后 decay 任务**

进阶再加：记忆晋升流水线、冲突合并、冷热分层、与审计 trace 对齐。

### 11. 一句话收束

**Memory（记忆） = 四分库（working / episodic / semantic / procedural）+ 统一 Facade（门面接口） + 写权限 + 提交/晋升规则 + 按类型的衰减淘汰。**

- Working（工作记忆） ≈ 当前 PLAN（规划）+EXECUTE（执行）+VERIFY（含同轮 REPLAN（重规划））草稿区；本轮结束压缩/结构化进 Episodic（情节记忆）后清空
- Episodic（情节记忆） = 日志型「经过」；Semantic（语义记忆） = 有条件提炼的跨轮事实/结论（不是全部历史）
- Procedural（程序记忆） = 版本化的「怎么做」剧本/策略，主要由配置维护

---

## 第三层：Context Layer（上下文层）

### 1. 一句话定位（存侧 vs 读侧）

| | Memory（记忆） | Context（上下文） |
|--|--------|---------|
| 侧 | **存侧（写）** | **读侧（装）** |
| 何时 | 本轮 Working（工作记忆）结束并 VERIFY（校验门禁）/commit（提交落库）后为主（事件进 Episodic（情节记忆）；合格事实才进 Semantic（语义记忆）等） | **每次要调大模型之前** |
| 输入 | Working（工作记忆）产物、工具结果、Gate（门禁）裁决等 | **已有 Memory（记忆） + 当前尚未归档的 Working（工作记忆）** + 本轮 query/goal |
| 输出 | 四分库里的持久状态 | 本轮的 `messages` / 提示词 |
| 是否长期 | 是 | 否（每轮现装，用完可丢） |

**Memory（记忆）负责记住；Context（上下文）负责在调用前从「已记住的 + 当前草稿」里挑出东西，封装成提示词。**

小补充：

1. Working（工作记忆）在归档前也算 Memory（记忆）的一部分（极短命）；Context（上下文）读的是「库里的历史 + 眼前这份草稿」
2. Memory（记忆）不是只在整轮结束时才写：EXECUTE（执行）中途就会写 Working（工作记忆）；整轮结束主要是 **晋升/清空 Working（工作记忆） → Episodic（及有条件 → Semantic（语义记忆））**

### 2. 与 Episodic（情节记忆） / 五层压缩：不冲突

| | Memory（记忆） / Episodic（情节记忆） | Context（上下文）层 |
|--|-------------------|------------|
| 职责 | **存**「发生过什么」（可审计、可回放） | **装**「这一次推理窗口里放什么」 |
| 形态 | 较完整的事件流 / 摘要存档 | 面向单次 `messages` 的、**有损**视图 |
| 是否丢信息 | 尽量保留（可裁切冷数据，但是存储策略） | **故意压缩**，只为塞进 token（词元）预算 |

```text
Episodic（源）──read──► Context 五层压缩 ──► 最终 prompt/messages
                              │
Semantic / Working / Procedural 也可作为输入源
```

- 五层压缩 ≠ 替代 Episodic（情节记忆）；它是 Episodic（等）的**读侧投影**
- 压缩结果一般写回本步 Working（工作记忆）或临时 `ContextBundle`，**默认不把 Episodic（情节记忆）删掉重写成短文**
- 一句话：**Episodic（情节记忆）存磁带；Context（上下文）每场演出只剪一版播出版。**

### 3. 关键时序：Context（上下文）在「调 LLM（大语言模型）前」跑，不是「Working（工作记忆）整轮结束后更新」

```text
① PLAN（代码）写出本轮 goal / action
② 若本步需要调 LLM：
      立刻跑 Context L1→L5
      输入 = 已有 Memory + 当前已有 Working（可能还不完整）+ 本轮 query
      输出 = messages
③ EXECUTE：把 messages 送给 LLM / 调工具，产物写入 Working
④ VERIFY
⑤ 通过后：Working 归档 → Episodic；有条件才 upsert Semantic
⑥ 若还要再调 LLM（或 REPLAN 后再调）：再跑一遍 Context
```

| 说法 | 对不对 |
|------|--------|
| Context（上下文）在 Working（工作记忆）整轮结束之后才运行，用来「更新」 | **不对**（更新是 Memory（记忆） commit（提交落库）；Context（上下文）是装提示词） |
| Context（上下文）在每次要调 LLM（大语言模型）前运行 | **对** |
| 跑 Context（上下文）时 Working（工作记忆）可能已有本轮 PLAN（规划）/中间结果 | **对** |

「Working（工作记忆）完成 → 压缩进 Episodic（情节记忆）」和「Context（上下文）五层压缩」是两件事：前者是**写回存储**；后者是**临时做成提示词**。

### 4. 五层不是五块长期仓库

五层是**每次准备调用大模型时，现场跑一遍的加工流水线**。长期数据仍在 Memory（记忆）；五层只是中间形态，多数**不跨轮持久化**。

```text
【持久】Memory（working / episodic / semantic / procedural）
        │  每轮调用 LLM 前，现场跑一遍
        ▼
【瞬态】L1 → L2 → L3 → L4 → L5 → messages[]
        │
        ▼
【本轮】送进 LLM
        │
下一轮：再从 Memory 读，重新跑 L1→L5
       （不是「接着读上一轮 L3 仓库」）
```

流水线内部统一零件（示意）：

```python
{
  "id": "ep_042",
  "kind": "episodic",   # episodic | semantic | working | procedural
  "text": "……将进入提示词的正文……",
  "tokens": 120,
  "score": 0.0,         # L3 才认真填
  "meta": {"task_id": "t1", "ts": ..., "importance": 0.7, "recency": 0.9}
}
```

### 5. 五层各自：输入 → 怎么处理 → 输出形态 → 是否进提示词

| 层 | 名字 | 处理动作 | 输出存在哪 | 是否进入本轮提示词 |
|----|------|----------|------------|-------------------|
| L1 | Select（选取） | 多路捞取 | 临时 List | 否 |
| L2 | Filter（过滤） | 过滤去重 | 临时 List | 否 |
| L3 | Rank（排序） | 打分排序 | 临时 List | 否 |
| L4 | Condense（压缩精炼） | 摘要/截断 | 临时 List（text 已改） | 否（间接：内容被 L5 用） |
| L5 | Pack（装箱截断） | 分桶+固定顺序拼 messages | `messages[]` | **是，唯一入口** |

#### L1 Select（选取）：从 Memory（记忆）「捞候选」

**输入**：Memory（记忆）各库 + `task_id` / `goal` / `query` / 时间窗等。

**处理**（几乎不做「好不好」判断）：

| 来源 | 怎么取 |
|------|--------|
| working | 当前 task 的草稿整取 |
| episodic | 最近 N 条，或最近 T 时间内 |
| semantic | 按 query **向量 / 关键词** top-k（见 §6） |
| procedural | 当前启用的流程配置 |

**输出**：未排序、未压缩的 `List[Hit]`（原料筐，可能很长）。**不直接进提示词。**

#### L2 Filter（过滤）：扔掉不该进窗口的

**输入**：L1 的 `List[Hit]`。

**处理**：过期/无权限/未过 Gate（门禁）的失败草稿丢弃；正文重复只留一条；空文本丢弃。

**输出**：更短的 `List[Hit]`。**仍不进提示词。**

#### L3 Rank（排序）：打分排序

**输入**：L2 列表 + 本轮 `goal`/`query`。

**处理**：算 `score`（重合 / embedding（向量嵌入） / recency / importance / kind 权重），降序排序；一般**不改 `text`**。

**输出**：带 `score`、已排序的 `List[Hit]`。**仍不进提示词**（只决定谁优先占预算）。

#### L4 Condense（压缩精炼）：压缩正文（主战场）

**输入**：L3 列表 + token（词元）预算。

**处理**：

1. 高分 + 短 → 尽量保留原文
2. 低分或很长的 episodic → `text` 换成摘要/一行结构化短句
3. 预算不够 → 截断列表（丢掉排后面的）

**输出**：条数可能更少、`text` 已变短的 `List[Hit]`。  
改的是**流水线里的 Hit.text**；**默认不回写 Episodic（情节记忆）**。

#### L5 Pack（装箱截断）：装箱 → 真正的提示词

**输入**：L4 列表 + 稳定 `system_prefix` + 本轮 `user_turn`。

**处理**：按 kind 分桶，按**固定顺序**拼 `messages`（前缀稳、变化靠后 → 利 KV-cache（键值缓存））：

```text
procedural / 极稳部分 → 并入或紧跟 system
semantic 短事实     → [known_facts]
episodic 压缩结果   → [history_digest]
working 草稿        → [working]
本轮用户输入        → 最后一条 user
```

**输出示例**：

```python
# ContextBundle（瞬态）
{
  "system_prefix": "人设…硬规则…工具schema…(可含 procedure)",
  "sticky_facts": ["user.timezone=UTC+8"],
  "history_blocks": ["verify_failed gate=schema_ok ..."],
  "working_blocks": ["current plan: ..."],
  "user_turn": "请继续…"
}

# 喂给模型
[
  {"role":"system","content":"<prefix>\n[known_facts]\n- ..."},
  {"role":"system","content":"[history_digest]\n..."},
  {"role":"system","content":"[working]\n..."},
  {"role":"user","content":"<user_turn>"}
]
```

**只有 L5 输出进入大模型。**

### 6. L1 检索 Semantic（语义记忆）时：向量 / 关键词从哪来？

都来自 **这一次装配时构造的检索 query 文本**，再变成向量或关键词——不是库里预先为「下一次 Context（上下文）」存好的神秘查询向量。

#### query 文本通常从哪拼

1. 本轮用户输入
2. 当前 PLAN（规划）的 `goal` / `tasks` / action 描述
3. 当前 Working（工作记忆）里已有焦点字段（实体 id、文件名等）
4. （可选）查询改写收成短句

```python
def build_retrieval_query(plan, working, user_turn: str) -> str:
    parts = [user_turn]
    if plan and plan.goal:
        parts.append(plan.goal)
    if working.get("focus_entity"):
        parts.append(str(working["focus_entity"]))
    return "\n".join(p for p in parts if p).strip()
```

#### 关键词 / 向量

```python
keywords = tokenize(query_text)
hits_kw = semantic_store.keyword_search(keywords, top_k=20)

q_vec = embed(query_text)   # L1 现场算
hits_vec = semantic_store.vector_search(q_vec, top_k=20)
```

两侧对比：

```text
写入 Semantic 时（更早的某次 commit）：
  fact_text → embed(fact_text) → 存进向量索引

本次 L1（调 LLM 前）：
  query_text（user/plan/working）→ embed(query_text) → Top-k 近邻
```

### 7. 下一轮交互时，五层数据怎么再进提示词？

**下一轮不是把上一轮 L4 列表再拿出来用**，而是：

```text
上一轮：Memory ──L1→L5──► messages_r1 ──► LLM
              └─ 过程写入 Episodic；Working 更新/清空

下一轮：Memory（已含新 episodic）──再跑 L1→L5──► messages_r2
```

| 问题 | 答案 |
|------|------|
| 每层数据下轮如何输入提示词？ | 下轮重新 Select（选取）→…→Pack（装箱截断）；进提示词的永远是**新的 L5 messages** |
| L3 排序会保存吗？ | 默认否；下轮按新 goal 重算 |
| L4 摘要会保存吗？ | 默认否；需要可另存 derived summary，源仍是 Episodic（情节记忆） |
| 提示词里历史从哪来？ | 下轮 L1 再读 Episodic（情节记忆） → L4 再压成新的 `history_digest` |
| 跨轮真正带着走的是什么？ | **Memory（记忆）**，不是五层中间 List |

### 8. KV-Cache（键值缓存）复用

**核心：前缀尽量比特级稳定，变化尽量挤到后缀。**  
改了前面任意 token（词元），后面 KV 缓存通常整段失效。

工程做法：

1. **稳定前缀**：system 人设、工具 schema（结构约定）、硬规则、procedural —— 能固定就固定
2. **变化放尾**：本轮 user、working、新工具结果
3. 长历史不要塞进最前缀；走 L4 压成 digest，作为相对靠后的 block
4. 同一套模板渲染（JSON（数据交换格式） key 顺序、空白也要稳）
5. API（应用程序接口）若支持显式 cache breakpoint，打在 `system_prefix` 上

```text
[ system_prefix + tools + procedure ]  ← 尽量不变 → KV 长命
[ known_facts ]                        ← 偶尔变
[ history_digest ]                     ← 常变（已压缩）
[ working ]                            ← 每步变
[ user_turn ]                          ← 每轮变
```

**KV-Cache（键值缓存）复用 ≈ 输入消息的「前缀不变性」工程**；由 L5 Pack（装箱截断）落实，不是 Memory（记忆）的事。

### 9. 实现骨架（五层代码）

```python
from __future__ import annotations
from dataclasses import dataclass, field
from typing import Callable
import hashlib
import time

@dataclass
class MemHit:
    id: str
    kind: str
    text: str
    meta: dict = field(default_factory=dict)
    score: float = 0.0
    tokens: int = 0

@dataclass
class ContextBundle:
    system_prefix: str
    sticky_facts: list[str]
    history_blocks: list[str]
    working_blocks: list[str]
    user_turn: str

@dataclass
class TokenBudget:
    total: int = 8000
    reserved_for_output: int = 1000
    reserved_for_prefix: int = 1500

    @property
    def for_variable(self) -> int:
        return self.total - self.reserved_for_output - self.reserved_for_prefix


def estimate_tokens(text: str) -> int:
    return max(1, len(text) // 4)


def l1_select(memory, task_id: str, query: str, now: float) -> list[MemHit]:
    hits: list[MemHit] = []
    for item in memory.read("working", query={"task_id": task_id}):
        hits.append(MemHit(item.id, "working", item.content_text(), item.meta))
    for item in memory.read(
        "episodic",
        query={"task_id": task_id, "since": now - 86400, "limit": 200},
    ):
        hits.append(MemHit(item.id, "episodic", item.content_text(), item.meta))
    for item in memory.read("semantic", query={"text": query, "top_k": 20}):
        hits.append(MemHit(item.id, "semantic", item.content_text(), item.meta))
    for item in memory.read("procedural", query={"active": True}):
        hits.append(MemHit(item.id, "procedural", item.content_text(), item.meta))
    return hits


def l2_filter(hits: list[MemHit], seen_fp: set[str]) -> list[MemHit]:
    out = []
    for h in hits:
        if h.meta.get("expired") or h.meta.get("failed_draft"):
            continue
        fp = hashlib.sha1(h.text.strip().encode()).hexdigest()[:16]
        if fp in seen_fp:
            continue
        seen_fp.add(fp)
        h.tokens = estimate_tokens(h.text)
        out.append(h)
    return out


def l3_rank(hits: list[MemHit], query: str, goal: str) -> list[MemHit]:
    q = (query + " " + goal).lower()

    def score(h: MemHit) -> float:
        overlap = sum(1 for w in q.split() if w and w in h.text.lower())
        recency = h.meta.get("recency", 0.0)
        importance = h.meta.get("importance", 0.5)
        kind_boost = {"working": 1.2, "procedural": 1.1, "semantic": 1.0, "episodic": 0.9}[h.kind]
        return (overlap + 1e-3) * (0.4 + 0.3 * recency + 0.3 * importance) * kind_boost

    for h in hits:
        h.score = score(h)
    return sorted(hits, key=lambda x: x.score, reverse=True)


def l4_condense(hits: list[MemHit], budget: TokenBudget, summarize: Callable[[str], str]) -> list[MemHit]:
    kept, used = [], 0
    soft_limit = int(budget.for_variable * 0.7)
    for i, h in enumerate(hits):
        text = h.text
        if h.kind == "episodic" and (i >= 5 or h.tokens > 300):
            text = summarize(h.text)
        if used + estimate_tokens(text) > soft_limit and h.kind == "episodic":
            text = summarize(h.text)
        h2 = MemHit(h.id, h.kind, text, h.meta, h.score, estimate_tokens(text))
        kept.append(h2)
        used += h2.tokens
        if used >= budget.for_variable:
            break
    return kept


def l5_pack(hits: list[MemHit], *, system_prefix: str, user_turn: str, budget: TokenBudget) -> ContextBundle:
    sticky, history, working = [], [], []
    used = estimate_tokens(system_prefix)
    for h in hits:
        if used + h.tokens > budget.for_variable + budget.reserved_for_prefix:
            continue
        if h.kind == "semantic":
            sticky.append(h.text)
        elif h.kind == "working":
            working.append(h.text)
        elif h.kind == "episodic":
            history.append(h.text)
        elif h.kind == "procedural":
            system_prefix = system_prefix + "\n\n[procedure]\n" + h.text
            used += h.tokens
            continue
        used += h.tokens
    return ContextBundle(system_prefix, sticky, history, working, user_turn)


def to_messages(bundle: ContextBundle) -> list[dict]:
    system = bundle.system_prefix
    if bundle.sticky_facts:
        system += "\n\n[known_facts]\n" + "\n".join(f"- {x}" for x in bundle.sticky_facts)
    messages = [{"role": "system", "content": system}]
    if bundle.history_blocks:
        messages.append({"role": "system", "content": "[history_digest]\n" + "\n---\n".join(bundle.history_blocks)})
    if bundle.working_blocks:
        messages.append({"role": "system", "content": "[working]\n" + "\n".join(bundle.working_blocks)})
    messages.append({"role": "user", "content": bundle.user_turn})
    return messages


def build_context(memory, *, task_id, query, goal, user_turn, system_prefix, summarize, budget=None):
    budget = budget or TokenBudget()
    seen: set[str] = set()
    hits = l1_select(memory, task_id, query, time.time())
    hits = l2_filter(hits, seen)
    hits = l3_rank(hits, query, goal)
    hits = l4_condense(hits, budget, summarize)
    bundle = l5_pack(hits, system_prefix=system_prefix, user_turn=user_turn, budget=budget)
    return to_messages(bundle), bundle
```

### 10. 体积变化示意

| 阶段 | 数据在哪 | 形态 | 体积 |
|------|----------|------|------|
| Memory（记忆） | 磁盘/DB | 完整事件/事实 | 大 |
| L1 后 | 内存 | Hit 列表原文 | 很大 |
| L2 后 | 内存 | 过滤后 Hit | 大 |
| L3 后 | 内存 | 带分排序 Hit | 大 |
| L4 后 | 内存 | text 变短 / 条数变少 | 中→小 |
| L5 后 | API（应用程序接口） | `messages[]` | ≤ token（词元）预算 |

### 11. 一句话收束

**Context（上下文） = 调 LLM（大语言模型）前的读侧装配：从 Memory（记忆） + 当前 Working（工作记忆）经 L1→L5 压成 messages；与 Memory（记忆）存侧分工，与 Episodic（情节记忆）不冲突。**

- 五层是瞬态流水线，不是五块长期库；只有 L5 进入提示词
- Semantic（语义记忆） Top-k（前k条）的 query 向量/关键词来自**本轮** user/plan/working，库内向量来自**过去写入时**
- KV-cache（键值缓存）靠 L5「稳前缀 + 变后缀」

---

## 第四层：Sub-Agent Layer（子智能体层 / 多智能体协作）

### 0. 设计哲学（一句话）

**把流程调度、权限准入、资源分配的权力收归代码编排层，仅把内容生成与工具调用推理的自由度留给 LLM（大语言模型）；LLM（大语言模型）只负责干活，无权更改作业流程与准入规则。**

---

### 1. 多 Agent（智能体）协作底层控制权在代码编排器，不在 LLM（大语言模型）代码预先写死全流程阶段、步骤顺序（如 `STAGE_PLANS` / `PIPELINE` 规则表），编排器按序驱动执行。

- LLM（大语言模型） **不能**新增步骤、跳过步骤、调换步骤顺序
- 流程走向完全由代码锁死（可按 state 走**预先写好的**分支，分支表仍是代码）

```text
代码固定流水线
  Step1 → Step2 → Step3 → …
     │        │        │
     ▼        ▼        ▼
  每步策略（代码配置，可按 state 切换档位）
  - allowed_agents
  - tool_allowlist
  - prompt_template / output_schema
  - 预算 / Gate
     │
     ▼
  本步内：LLM 仅在沙箱里发挥
```

**常见误解校正：**

| 说法 | 是否采纳 |
|------|----------|
| 多 Agent（智能体） = 固定步骤 + 主 Agent（智能体）编排 + 步内沙箱 | ✅ 大方向对 |
| 主编排器优先是**代码**；若用「主 LLM（大语言模型）」只能在合法后继 step 白名单里选 | ✅ |
| 本步用哪个子 Agent（智能体）、开哪些工具、生成哪套 system 提示词，都交给 LLM（大语言模型） | ❌ 会把门禁变软 |
| LLM（大语言模型）在**已给定**的 Agent（智能体） + 工具箱 + 模板内填参、选调用顺序、写正文 | ✅ 真正的步内自由 |

---

### 2. 每一步绑定三层静态资源（门禁隔离根源）每个 Step 在配置时就硬绑定三件东西：

1. **允许的子 Agent（智能体）白名单**（`allowed_agents`）：本步骤只能调度哪些子智能体，越权直接抛隔离错误  
2. **限定工具箱**（`tool_allowlist`）：该步骤能调用哪些工具，其他工具无入口  
3. **固定提示词模板**：Agent（智能体）的角色、输出格式、约束规则提前写死；LLM（大语言模型）只能在模板框架内输出（可填槽，不宜让模型随意改写高权限 system prompt（提示词））

三者共同构成**门禁边界**：开门 / 放行权限由代码配置决定，LLM（大语言模型）没有权限修改门禁规则，没有权限自选/扩权 Agent（智能体）与工具。

「调节判断」也宜是**代码根据 state 选策略档**，而不是模型改白名单：

```python
PIPELINE = ["triage", "investigate", "draft", "close"]

STEP_POLICY = {
    "triage": {
        "allowed_agents": ["classifier"],
        "tool_allowlist": [],
        "output_schema": "ClassifyOut",
    },
    "investigate": {
        # 代码按 state 选档
        "billing": {
            "allowed_agents": ["tool_worker", "research_worker"],
            "tool_allowlist": ["billing.get_invoice", "kb.search"],
        },
        "access": {
            "allowed_agents": ["tool_worker"],
            "tool_allowlist": ["crm.get_user"],
        },
    },
    "draft": {
        "allowed_agents": ["writer"],
        "tool_allowlist": [],
        "output_schema": "ReplyOut",
    },
}

def policy_for(step, state):
    base = STEP_POLICY[step]
    if step == "investigate":
        return base[state.intent or "access"]
    return base
```

---

### 3. LLM（大语言模型）仅拥有「步内沙箱自由」，无流程调度自由

**沙箱内允许（✅）：**

- 在当前 Step、当前已选定 Agent（智能体）、给定工具、给定模板约束里自主决定：
  - 工具调用参数、调用顺序（仅限白名单内）
  - 文本生成措辞、推理过程
- 产出必须匹配本步 `output_schema`，再交 VERIFY（校验门禁）

**严禁越界（❌，不能自己开门禁）：**

- 不能主动切换到未授权的子 Agent（智能体）  
- 不能跳过本应执行的校验 Step  
- 不能调用本阶段禁止的工具  
- 不能擅自跳转到下一阶段 / 发明新步骤  
- 任何跨边界诉求由 `dispatch` 路由拦截，直接报错  

| 交给 LLM（合适） | 交给代码（合适） |
|------------------|------------------|
| 白名单里先调工具 A 还是 B | 本步白名单是什么 |
| 工具参数、检索 query | 能否调用某敏感 API（应用程序接口） |
| 回复正文、摘要、结构化字段内容 | 用哪套 system 模板、哪个 `sub_agent_id` |
| （可选）在 2～3 个**已批准**模板 ID 里选一个 | 模板全文；扩权工具箱 |

---

### 4. 硬隔离：六个维度（机制，不是文案）

**硬隔离** = 边界由 Harness（工程控制架）强制执行；不是靠 prompt（提示词）「你现在是某某，不要做别的」。

| | 软隔离 | 硬隔离 |
|--|--------|--------|
| 边界 | System prompt（提示词）劝说 | allowlist（白名单） / schema（结构约定） / 路由，越权失败 |
| 谁裁决 | 模型自觉 | Control Plane（控制平面） + 权限检查器 |

每个子 Agent（智能体）一张能力身份证 `SubAgentSpec`：

```text
SubAgentSpec
├── id / role
├── system_prompt          # Harness 注入，调用方/模型改不了最终权威版
├── input_schema / output_schema
├── tool_allowlist
├── read_scopes / write_scopes   # Memory 读写范围
├── network / secrets（可选）
└── runtime（可选：独立进程/容器）
```

#### （1）路由硬分：只有编排器能点名调用

子 Agent（智能体） **不能互相直接调用**；由控制平面按当前 Step 策略里的 `sub_agent_id` / 白名单调度。

```python
def dispatch(plan, state, context_messages):
    agent_id = plan.control["sub_agent_id"]   # 代码写入或代码映射后的最终值
    if agent_id not in plan.control["allowed_agents"]:
        raise IsolationError("agent not allowed in this step")
    return run_subagent(AGENTS[agent_id], state, context_messages)
```

#### （2）`sub_agent_id` 是不是大模型生成的 JSON（数据交换格式）字段？

**最终说了算的 `sub_agent_id`，不应是模型随便生成就直接执行。**

| 做法 | 来源 | 是否硬分 |
|------|------|----------|
| A. 纯代码 | 当前 step 策略直接指定 | ✅ 最硬 |
| B. 模型建议 + 代码裁决 | 模型在 `allowed_actions` 内选 → 代码 `action → agent_id` 映射 | ✅ 常用 |
| C. 模型自由填 agent 名且不校验 | 直接 dispatch（调度分发） | ❌ 软路由 |

```python
ACTION_TO_AGENT = {"generate": "generator", "critique": "critic", "fetch": "tool_worker"}

def finalize_step_control(llm_draft, state, step_policy):
    allowed = step_policy["allowed_actions"]  # 或直接 allowed_agents
    action = (llm_draft or {}).get("selected_action")
    if action not in allowed:
        action = allowed[0]  # 非法建议丢弃
    return {
        "selected_action": action,
        "sub_agent_id": ACTION_TO_AGENT[action],  # 以代码映射为准
        "allowed_agents": step_policy["allowed_agents"],
        "tool_allowlist": step_policy["tool_allowlist"],
    }
```

Plan 可以是 JSON（便于审计），但 **`control.*` 由代码写入/覆盖**；模型若输出整份 Plan，只吸收合法语义/白名单内 action。

#### （3）工具硬分

```python
def call_tool(agent: SubAgentSpec, name: str, args: dict):
    if name not in agent.tool_allowlist:
        raise IsolationError(f"{agent.id} cannot call {name}")
    return TOOLS[name](**args)
```

#### （4）Memory（记忆）读写 scope 硬分

子 Agent（智能体）通常只写 `working`；`working → episodic/semantic` 的晋升由 Harness（工程控制架）在 VERIFY（校验门禁）后 commit（提交落库），避免执行器直接改长期记忆。

#### （5）I/O Schema 硬分

非法字段进不了 `apply(state)`；校验失败走 REPLAN（重规划） / 拒收，而不是「模型说改就改」。

#### （6）运行时硬分（可选）同进程逻辑隔离（基线）→ 独立 worker 进程 → 容器 / 网络策略（涉及密钥、代码执行时加强）。

---

### 5. 和「LLM（大语言模型）自主规划 Agent（智能体）」对比

#### 架构 1：代码编排 + 门禁隔离（约束型）— 本笔记主推

- 流程、Agent（智能体）权限、工具范围、提示词模板全部（或按 state 档位）静态配置  
- LLM（大语言模型） = **步骤内的执行器**  
- 边界：流程调度、资源访问权归代码；内容生成与白名单内工具推理归模型  
- 优点：稳定、可控、幂等、易加校验；适合强规范、可审计业务  

#### 架构 2：LLM（大语言模型）自主规划（开放型，AutoGPT（自主规划类智能体）类）

- 无固定 Plan，模型自主决定下一步、调用哪个 Agent（智能体）、用什么工具  
- LLM（大语言模型） = 流程决策者 + 执行者  
- 边界靠 Prompt 软约束  
- 缺点：流程漂移、重复劳动、越权读写记忆、步骤不可预期；不适合强规范业务  

---

### 6. 与 Control / Memory（记忆） / Context（上下文） / VERIFY（校验门禁）闭环

1. **`STAGE_PLANS` / `PIPELINE`**：代码写死全流程编排（编排器驱动步骤）  
2. **`Plan.control` + `dispatch`**：每步 Agent（智能体）白名单、工具箱、预算；路由硬隔离  
3. **`SubAgentSpec`**：专属提示词模板、工具集、读写 scope、出入参 schema（结构约定）  
4. **Context（上下文）**：按当前子 Agent（智能体）的 `read_scopes` 装配窗口（不同 Agent（智能体）可见历史可不同）  
5. **Step 内 LLM（大语言模型）沙箱**：仅在当前约束内生成 / 调允许的工具  
6. **VERIFY（门禁）**：schema（结构约定）、业务 checker 等；模型不能绕过  
7. **Memory（记忆） commit（提交落库）**：VERIFY（校验门禁）通过后由 Harness（工程控制架）写 episodic / 有条件写 semantic；遗忘策略在存侧维护  

```text
编排器取当前 Step
  → 代码挂上 allowed_agents + tools + template
  → Context 按 read_scopes 装 messages
  → dispatch(sub_agent_id) 跑沙箱
  → VERIFY
  → pass：commit Memory，进入下一步
  → fail：REPLAN（通常改参数/重试，不交给模型改流程骨架）
```

---

### 7. 最小 Runtime 骨架

```python
@dataclass(frozen=True)
class SubAgentSpec:
    id: str
    system_prompt: str
    input_schema: type
    output_schema: type
    tool_allowlist: frozenset[str]
    read_scopes: frozenset[str]
    write_scopes: frozenset[str]

class IsolationError(Exception): ...

class SubAgentRuntime:
    def __init__(self, specs: dict[str, SubAgentSpec], tools: dict, memory, llm):
        self.specs, self.tools, self.memory, self.llm = specs, tools, memory, llm

    def run(self, agent_id: str, payload: dict, context_msgs: list) -> dict:
        agent = self.specs[agent_id]
        data = agent.input_schema.model_validate(payload)
        messages = [
            {"role": "system", "content": agent.system_prompt},
            *context_msgs,
            {"role": "user", "content": data.model_dump_json()},
        ]
        raw = self.llm.chat(messages)
        out = agent.output_schema.model_validate_json(extract_json(raw))
        self._write(agent, "working", f"artifacts.{agent.id}", out.model_dump())
        return out.model_dump()

    def invoke_tool(self, agent_id: str, name: str, args: dict):
        agent = self.specs[agent_id]
        if name not in agent.tool_allowlist:
            raise IsolationError(f"{agent_id} blocked tool {name}")
        return self.tools[name](**args)

    def _write(self, agent: SubAgentSpec, kind: str, key: str, value):
        if kind not in agent.write_scopes:
            raise IsolationError(f"{agent.id} cannot write {kind}")
        self.memory.write(kind, key, value, source=agent.id)
```

同一会话切换能力 = **编排器进入下一步 / 改 `sub_agent_id` 再 `run`**，而不是在同一条对话里靠文案换角色并继承上一角色的工具权。

---

### 8. 可选增强：丰富 state + 窄槽填充（仍不交出门禁）若步骤已固定仍嫌「目标文案/焦点」太僵，可在**不改骨架**前提下：

- 用丰富 `state`（阶段、实体、tool_cache、Gate（门禁）失败原因、覆盖率）让规则选出**候选菜单**  
- LLM（大语言模型） **只填槽**：在候选 `action` 内选择、写一句 goal、填 retrieve_query 等  
- `finalize`：校验 → 映射 `sub_agent_id` → materialize 本步已有 `step_templates`  

这与「步内沙箱自由」一致：**活的是槽位与正文，死的是步骤与门禁。**

---

### 9. 一句话收束

**多 Agent（智能体）协作本质 = 代码固定步骤并由编排器推进；每步由代码挂上允许的子 Agent（智能体） + 工具箱 + 提示词模板；LLM（大语言模型）只在步内沙箱中生成内容与推理调用；门禁与流程调度永不交给模型。**

---

## 第五层：RAG Layer（检索增强层）

### 1. 拿来干什么

**RAG Layer（检索增强层）** = 受门禁约束的「领域知识检索子系统」：按本步任务从外部/领域知识库可控取出「当前该用的材料」，再交给步内 Agent（智能体）或 Context（上下文）使用。

不是「随便向量搜一下」，而是带过滤、重排、权限与审计的检索流水线。

| 问题 | RAG（检索增强生成）层的答案 |
|------|----------------|
| 模型不知道的业务知识从哪来？ | 从预建知识库 / 文档库等检索，而不是靠参数记忆 |
| 和 Memory（记忆）.Semantic（语义记忆）啥区别？ | Semantic（语义记忆）多是**运行中沉淀的会话/实体事实**；RAG（检索增强生成）多是**预建的领域语料库**（也可共用检索引擎，职责仍宜分开） |
| 和 Context（上下文）啥关系？ | RAG（检索增强生成） = **取原料**；Context（上下文） = **把原料（+记忆）压进提示词** |
| 为啥单列一层？ | 检索要有多段过滤、重排、权限、审计；不能让模型直接「搜全世界」 |

---

### 2. 在哪个环节起作用

**主要两种入口：编排器显式调用，或步内白名单工具调用。**

```text
编排器进入某 Step（且策略允许检索）
        │
        ├─ A. 编排器显式 retrieve 步（代码写死 Step.kind = retrieve）
        │         → 直接调 RagLayer.search(...)
        │
        └─ B. 步内 Agent 工具调用（tool_allowlist 含 kb.search）
                  → LLM 决定要不要搜、搜什么 query（仍在沙箱内）
        │
        ▼
   RAG 返回 hits（带 id 的片段列表）
        │
        ▼
   写入 Working（如 rag_hits）/ 本步 artifacts
        │
        ├─ 同一步还要生成 → Context 把 hits 装进 messages 再调 LLM
        ├─ 或交给下一步 Step 用
        └─ VERIFY 可检查 citation 是否 ⊆ rag_hits
```

| 入口 | 谁触发 | 典型场景 |
|------|--------|----------|
| **编排器显式调用** | 代码 Step，不经过「模型决定搜不搜」 | 规定「生成前必须先检索」 |
| **工具调用** | 步内 LLM（大语言模型）在白名单里调 `kb.search` | 是否搜、搜几次由沙箱策略定，query 可由模型填 |

两种都是 **Harness（工程控制架）控制权内的调用**；模型不能在未授权 Step 里私自连知识库。

---

### 3. 调用之后干什么（是不是给提示词补信息）

**核心用途：给后续生成补充「可引用的外部材料」。** 路径一般是：

1. RAG（检索增强生成） → `hits[]`
2. 写入 **Working（工作记忆） / artifacts**（短存，便于审计、查重、grounded（有据可追溯））
3. **Context（上下文）** 装配时把 hits 编进 `messages`（如 `[retrieved]` 块）
4. 生成 Agent（智能体）基于材料写答案，并带上 `citation_ids`
5. VERIFY（校验门禁）：引用必须来自本次 hits

因此：

- ✅ 检索结果 → **经 Context（上下文）补充进提示词**
- ✅ 同时往往 **结构化落 Working（工作记忆）**，不只静默拼进 prompt（提示词）
- ❌ 不是 RAG（检索增强生成）直接改流程、直接当最终答案（除非做成「只检索不生成」的 Step）

```text
Working.rag_hits = [
  {id: "d1", text: "...", score: 0.82},
  {id: "d2", text: "...", score: 0.77},
]

Context 装提示词时增加：
  [retrieved]
  - [d1] ...
  - [d2] ...

→ 再调用生成 LLM
```

和「整轮 Context（上下文）」的关系：

- 显式 retrieve Step：常是 `retrieve →（Context）→ generate`
- 工具调用：同一步内 `search → 观察 hits → 再生成`，或 search 后由编排器再跑 Context（上下文）+生成

**分工一句话：RAG（检索增强生成）取材料；Context（上下文）把材料塞进提示词；LLM（大语言模型）用材料生成。**

---

### 4. 典型流水线（多阶段）

```text
L0  Query 构造     ← 本步 goal / 用户话 / Working 焦点（代码或步内 LLM 填参）
L1  粗召回 Recall  ← 向量 / 关键词 / 混合，多路合并
L2  硬过滤 Filter  ← 权限、类型、去重、已用过、元数据约束
L3  重排 Rerank    ← 精排模型或规则（相关度、难度、时效、多样性）
L4  截断 Pack      ← 按 token/条数预算切成引用块，带 source_id
```

与 Context（上下文）五层不要混：

- RAG（检索增强生成）的 Filter（过滤）/Rerank（重排） → **知识库命中质量**
- Context（上下文）的 L1–L5 → **整窗提示词装配**（可再消费 RAG（检索增强生成）结果）

---

### 5. 与 Memory（记忆） / Context（上下文）分工

| | RAG Layer（检索增强层） | Memory（记忆）.Semantic（语义记忆） | Context（上下文） |
|--|-----------|-----------------|---------|
| 主要语料 | 预建知识库 / 文档 | 运行期沉淀的事实 | 不存库，只装配 |
| 触发 | 某步工具调用 / 编排器显式 retrieve | commit（提交落库）写入、可读 | 每次调 LLM（大语言模型）前 |
| 输出 | 带 `id` 的引用命中列表 | 事实条目 | `messages` |

常见接法：RAG（检索增强生成）命中 → Working（工作记忆）.`rag_hits` → Context（上下文）打包 → 生成 → VERIFY（校验门禁）查 citation。

---

### 6. 如何实现（骨架）

#### 6.1 文档与索引

```python
@dataclass
class RagDoc:
    id: str
    text: str
    embedding: list[float]
    meta: dict  # type, tags, difficulty, updated_at, acl, ...

# 离线/启动：切块 → embed → 向量库 + 元数据表
```

#### 6.2 检索服务

```python
@dataclass
class RagHit:
    id: str
    text: str
    score: float
    meta: dict

@dataclass
class RagQuery:
    text: str
    top_k_recall: int = 50
    top_k_final: int = 5
    filters: dict = None       # {"type": "faq", "difficulty_max": 3}
    exclude_ids: set = None    # 本会话已用过的
    token_budget: int = 1500

class RagLayer:
    def __init__(self, vector_index, keyword_index, embed_fn, reranker=None):
        self.vec, self.kw, self.embed, self.reranker = (
            vector_index, keyword_index, embed_fn, reranker
        )

    def search(self, q: RagQuery) -> list[RagHit]:
        # L1 粗召回（多路）
        q_vec = self.embed(q.text)
        vec_hits = self.vec.search(q_vec, top_k=q.top_k_recall)
        kw_hits = self.kw.search(q.text, top_k=q.top_k_recall)
        merged = merge_by_id(vec_hits, kw_hits)  # RRF 或 max score

        # L2 硬过滤
        merged = [h for h in merged if self._pass_filters(h, q)]

        # L3 重排
        ranked = self._rerank(q.text, merged, q)

        # L4 按预算截断
        return self._pack(ranked, q.top_k_final, q.token_budget)

    def _pass_filters(self, h, q: RagQuery) -> bool:
        if q.exclude_ids and h.id in q.exclude_ids:
            return False
        f = q.filters or {}
        for k, v in f.items():
            if h.meta.get(k) != v and not _match_op(h.meta, k, v):
                return False
        if not acl_allow(h.meta.get("acl"), (q.filters or {}).get("principal")):
            return False
        return True

    def _rerank(self, query: str, hits: list[RagHit], q: RagQuery) -> list[RagHit]:
        if self.reranker:
            scores = self.reranker.score(query, [h.text for h in hits])
            for h, s in zip(hits, scores):
                h.score = s
        else:
            target_diff = (q.filters or {}).get("difficulty_target")
            for h in hits:
                bonus = 0.0
                if target_diff is not None:
                    bonus -= abs(h.meta.get("difficulty", 0) - target_diff) * 0.1
                bonus += recency_bonus(h.meta.get("updated_at"))
                h.score = h.score + bonus
        return sorted(hits, key=lambda x: x.score, reverse=True)

    def _pack(self, hits, k, budget) -> list[RagHit]:
        out, used = [], 0
        for h in hits:
            t = estimate_tokens(h.text)
            if used + t > budget or len(out) >= k:
                break
            out.append(h)
            used += t
        return out
```

#### 6.3 挂进门禁（工具暴露，禁止直连库）

```python
TOOLS = {
    "kb.search": lambda **kwargs: rag.search(RagQuery(**kwargs)),
}

# 某 Step 策略示例
"investigate": {
    "allowed_agents": ["research_worker"],
    "tool_allowlist": ["kb.search"],  # 只有这一步能检索
}
```

步内 LLM（大语言模型）只填 `text` / `filters` / `top_k`（schema（结构约定）校验）；**不能**扩权工具箱。

#### 6.4 VERIFY（校验门禁）「有据」检查

```python
def check_grounded(plan, state, artifacts):
    used_ids = set(artifacts["generate"].get("citation_ids", []))
    allowed = {h["id"] for h in state.working.get("rag_hits", [])}
    return used_ids <= allowed  # 引用必须来自本步 RAG 结果
```

---

### 7. 在 Harness（工程控制架）闭环中的位置

```text
Control/Step 策略允许 kb.search 或显式 retrieve
        │
        ▼
RAG：召回 → 过滤 → 重排 → 返回引用块
        │
        ▼
写入 Working / 交给 Context
        │
        ▼
Sub-Agent 基于引用生成
        │
        ▼
VERIFY（含 grounded）→ commit Memory
```

---

### 8. 一句话收束

**RAG Layer（检索增强层） = 在允许检索的 Step 里，由编排器显式调用或步内工具调用触发的受控检索；结果先入 Working（工作记忆）/artifacts，再经 Context（上下文）补充进提示词供生成使用，并用 VERIFY（校验门禁）做有据校验。**

---

## 第六层：Skills Layer（技能/业务编排层）

### 1. 起什么作用

**Skills（技能）层** = 可复用的「业务技能包」：确定性编排策略 + 个性化改写规则；给控制平面提供「这一步按什么剧本、材料怎么改」。

不是再开一个能改流程的自由 Agent（智能体）。

| 问题 | Skills（技能）层做什么 |
|------|-----------------|
| 流程里「业务上怎么推进」从哪来？ | **确定性编排**（任务序、阶段覆盖、跳过条件） |
| 同一技能如何贴当前对象？ | **个性化改写**（约束内改表述/换槽位，不改门禁） |
| 和 Control Plane（控制平面）啥关系？ | Control 管 **能不能走、谁执行、何时 VERIFY（校验门禁）**；Skills（技能）管 **走哪条业务剧本、参数怎么填** |
| 和 Procedural（程序记忆） Memory（记忆）啥关系？ | Procedural（程序记忆） **存** SOP（标准作业程序）/版本；Skills（技能） **加载并执行**成可调用模块 |
| 和 Sub-Agent（子智能体）啥关系？ | Skills（技能）产出焦点/模板/改写输入；Sub-Agent（子智能体）在沙箱里生成/调工具 |

```text
Control Plane 进入某 Stage/Step
        │
        ▼
Skills：选剧本 / 定序 / 个性化填槽（代码为主，LLM 最多填窄槽）
        │
        ▼
得到：本步 goal 文案、retrieve 参数、prompt 槽位、改写后的材料
        │
        ├─ 需要知识 → RAG / MCP
        └─ 需要生成 → Sub-Agent（门禁不变）
        │
        ▼
VERIFY → Memory commit
```

**Skills（技能） = 业务侧的技能与编排策略层；让固定流水线可产品化、可个性化，同时不把调度权交给模型。**

---

### 2. 和相邻层怎么分

| 层 | 管什么 |
|----|--------|
| Control Plane（控制平面） | 状态机、预算、dispatch（调度分发）、VERIFY（校验门禁） |
| **Skills（技能）** | 业务剧本、定序、个性化改写、技能入口 |
| Procedural（程序记忆） Memory（记忆） | 剧本的持久化与版本 |
| Sub-Agent（子智能体） | 步内执行（生成/工具） |
| RAG（检索增强生成） / MCP（模型上下文协议） | 取外部知识与服务 |

Skills（技能） **不**负责：改 `tool_allowlist`、发明新 Step、绕过 VERIFY（校验门禁）。

与步内沙箱的关系：

```text
Skills：定「本步业务上做什么、材料怎么套模板」
Control：定「谁执行、什么工具、如何 VERIFY」
Sub-Agent：在沙箱里把 Skills 给的输入变成产物
```

---

### 3. 典型能力（两类）

**A. 确定性编排（Deterministic Orchestration）**

- 阶段内任务顺序、必做清单、覆盖率
- 「缺实体 → 先澄清；齐了 → 才调查」等表驱动转移
- 输出给 Control：下一业务焦点、推荐 `action`（仍须 ∈ 白名单）

**B. 个性化改写（Personalization）**

- 同一技能模板 + 当前对象画像（档案、实体、元数据）
- 改的是文案/槽位/检索 query，不是门禁
- 结果进 Working（工作记忆），经 Context（上下文）进提示词；VERIFY（校验门禁）仍校 schema（结构约定）/约束

LLM（大语言模型）在 Skills（技能）中的边界：

| Skills（技能）/代码 | LLM（可选） |
|-------------|-------------|
| 选剧本、下一 focus 顺序 | 在模板占位符内写更顺的 goal |
| 模板全文、必填槽来自 state | 不得增删模板约束、不得改 tool 白名单 |
| action 最终以 Control 白名单为准 | 不得发明新 stage |

---

### 4. 如何实现

#### 4.1 技能包形态

```python
@dataclass
class Skill:
    id: str
    version: str
    stages: dict              # stage -> {required, order, transitions}
    allowed_actions: list[str]  # 仍 ⊆ Control 白名单
    templates: dict[str, str]
    slot_sources: list[str]
```

YAML（配置格式）示例（与 Procedural（程序记忆）对齐）：

```yaml
id: support_ticket_v3
stages:
  triage:
    order: [classify, maybe_clarify]
    required: [intent_set]
  investigate:
    order: [fetch_by_intent, search_kb]
    required: [evidence_ready]
  draft:
    order: [draft_reply]
templates:
  clarify: "请补充 {missing_field}，以便继续处理工单 {ticket_id}"
  draft: "针对 {intent}，结合已查到的 {evidence_keys} 回复用户"
slot_sources: [intent, entities, tool_cache, sentiment]
```

#### 4.2 编排器调用 Skills（技能）

```python
class SkillsLayer:
    def __init__(self, registry: dict[str, Skill]):
        self.registry = registry

    def next_focus(self, skill_id: str, state) -> dict:
        skill = self.registry[skill_id]
        stage_conf = skill.stages[state.stage]
        for item in stage_conf["order"]:
            if item not in state.done and self._preconditions_ok(item, state):
                return {"action_hint": item, "stage": state.stage, "skill": skill_id}
        return {"action_hint": "advance_stage"}

    def personalize(self, skill_id: str, template_key: str, state, llm_slots: dict | None = None) -> str:
        skill = self.registry[skill_id]
        tmpl = skill.templates[template_key]
        slots = {k: lookup(state, k) for k in skill.slot_sources}
        if llm_slots:
            for k, v in llm_slots.items():
                if "{" + k + "}" in tmpl:
                    slots[k] = v
        return tmpl.format(**{k: slots.get(k, "") for k in _names_in(tmpl)})

    def load(self, skill_id: str, version: str):
        sop = procedural.get(skill_id, version)
        self.registry[skill_id] = compile_sop_to_skill(sop)

    def resolve(self, skill_id: str, state) -> Skill:
        v = experiment.pick_version(skill_id, state)  # 灰度
        return self.registry[(skill_id, v)]
```

#### 4.3 和 Control 衔接

```python
def plan_step(state):
    focus = skills.next_focus("support_ticket_v3", state)
    action = focus["action_hint"]
    policy = policy_for(state.stage, state)
    if action not in policy["allowed_actions"]:
        action = policy["allowed_actions"][0]  # 技能建议非法则回退

    user_prompt = skills.personalize(
        "support_ticket_v3",
        template_key=action,
        state=state,
        llm_slots=optional_narrow_llm_fill(state),
    )
    return finalize_plan(action, policy, user_payload=user_prompt)
```

`compile_sop_to_skill`：把 SOP（标准作业程序） YAML（配置格式）编成 `Skill`；**Control 硬白名单以安全基线为准**，SOP（标准作业程序）只能在基线内收紧或选档，不能私自加危险工具（除非审批改基线）。

---

### 5. 能否通过 SOP（标准作业程序）版本进化？可以，但是「发版进化」不是「跑着自己长出新流程」

```text
SOP / Procedural（存侧，版本化）
        │  发版 / 加载
        ▼
Skills Layer（跑侧）
        │  轨迹、Gate、业务指标
        ▼
评估与蒸馏 → SOP vN+1 → Skills 切版本（可灰度）
```

| 可以进化的 | 不宜「自动进化」的 |
|------------|-------------------|
| 阶段顺序微调、必做项、模板文案、填槽规则、候选 action 菜单、分档 | `tool_allowlist` 扩权、绕过 VERIFY（校验门禁）、取消预算、放开任意 Agent（智能体） |

---

### 6. 如何评估 SOP（标准作业程序）对每个 `skill_id@version` 用 Episodic（情节记忆）/审计日志聚合：

| 类别 | 例 |
|------|----|
| 完成率 | 到达终态比例、人工接管率 |
| 门禁健康 | Gate（门禁） fail 率、reason 分布、平均 replan、hard_stop（硬停机）率 |
| 效率 | 平均步数、墙钟、token（词元）、工具调用次数 |
| 质量 | 业务验收、满意度、返工率、grounded（有据可追溯）失败率 |
| 稳定 | 同输入回放是否幂等、灰度是否回归 |

```text
固定评测集回放
  → 分别跑 SOP vN 与候选 vN+1
  → 对比指标 + Gate reason
  → 未变差且目标项变好 → 允许晋级灰度
```

---

### 7. 从运行中提炼什么、如何更新（及局限）

#### 7.1 同质流程的关键局限（疑问解答）

**定死 Skills（技能）后，轨迹在「步骤长什么样」上高度同质，很难靠「又跑出一套新流程」反哺 Skills（技能）。**

```text
固定 Skills → 行为被门禁塑成窄分布
  → 日志里几乎看不到「另一种更好流程」
  → 用这些日志提炼，容易只得到微调，难得到结构性创新
```

这是 **探索不足（exploitation-only）** 的设计取舍，不是实现漏了。

同质轨迹里仍有的信号（多在参数与分支，不在拓扑）：

| 流程看起来一样 | 仍可统计 |
|----------------|----------|
| 同一 stage 链 | Gate（门禁）失败原因分布、卡在哪一阶 |
| 同一类工具 | 缺参率、重试次数、工具耗时 |
| 同一模板 | 人工改稿 diff 热点、二次追问主题 |
| 同一 order | 某 intent/实体组合下完成率、升级率差异 |

→ 反哺的是：模板槽位、澄清话术、`when` 分档、阈值、是否强制 retrieve——**同拓扑下的策略精修**，不是流程拓扑自由生长。

#### 7.2 进化动力应主要来自哪里

| 来源 | 作用 |
|------|------|
| 门禁错误 / badcase（失败案例）聚类 | 驱动候选变体设计 |
| 人工接管 / 专家改稿 | 最强：脚本外做法再蒸馏 |
| 离线对照实验 | 人为准备 SOP（标准作业程序）′，评测集 A/B |
| 业务/合规变更 | 直接改 SOP（标准作业程序）版本 |
| 受控灰度 | 小流量试**预先设计的**候选版本 |

线上主路径负责稳态执行；大改编排靠 **候选 SOP（标准作业程序） + 评测**，不指望主流程日志自动长出新拓扑。

#### 7.3 推荐进化链（疑问确认后的校正版）

**从门禁/异常 badcase（失败案例）分析 → 设计同流程下多个候选 Skill（技能）/SOP（标准作业程序） → 回放或灰度评估 → 决定是否晋级默认版本。**

```text
线上主 Skill（固定拓扑）
  → Gate / 异常 / badcase 聚类（缺参、重复、grounded 失败、某 intent 卡住…）
  → 人工或受控分析：做成「同流程多个候选 Skill/SOP」
     （改分档、模板、前置澄清、阈值…一般不改门禁扩权）
  → 评测集回放 / 小流量灰度对比
  → 达标才切换默认版本；否则丢弃或再改候选
```

要点：

1. 进化信号主要来自门禁失败与 badcase（失败案例），不是「流程跑出新形状」  
2. 候选是同骨架变体；先评估再晋级  
3. 是否进化由评估（+必要时人工）决定，不是业务 Agent（智能体）运行时改写 Skills（技能）  

#### 7.4 信号 → 变更类型 / Patch（补丁变更）流水线

| 信号 | 可能变更 |
|------|----------|
| 某步 Gate（门禁） `missing_x` 很高 | 前序加 clarify；模板加必填槽 |
| 重复劳动多 | 跳过已 `done`；order 微调 |
| 某 intent 工具选错率高 | 分档策略改映射（仍代码表） |
| 用户常补同一信息 | 澄清模板/slot_sources |
| grounded（有据可追溯）差 | 强制先 retrieve；模板要求 citation |
| 人工修订高频 diff | 更新 draft 模板结构 |

```text
1. 聚合：skill_version × stage × gate_reason × intent
2. 提案：规则或「分析用 LLM」只输出受限 Patch JSON
3. 校验：不得擅自扩权；过 schema + 权限 diff
4. 人工评审（权限类强制）
5. 编译：Patch → SOP YAML vN+1 入 Procedural
6. 回放评测 → 灰度 → Skills 切换默认 version
```

```python
patch = {
    "skill_id": "support_ticket",
    "base_version": "3",
    "ops": [
        {"op": "add_template_slot", "template": "clarify", "slot": "order_id"},
        {"op": "reorder", "stage": "triage", "order": ["classify", "ask_clarify"]},
    ],
}

def apply_patch(sop_v3: dict, patch: dict) -> dict:
    sop = deepcopy(sop_v3)
    for op in patch["ops"]:
        ...  # 只允许白名单 op
    sop["version"] = next_version(sop_v3["version"])
    validate_sop_schema(sop)
    assert_no_privilege_escalation(sop_v3, sop)
    return sop
```

在线业务 LLM（大语言模型）仍只在步内沙箱干活；「读日志 → 写 Patch（补丁变更）」是离线/异步分析任务。

---

### 8. 一句话收束

**Skills（技能） = 版本化业务技能包：代码做确定性编排，模板（+可选窄槽 LLM（大语言模型））做个性化；由 Control 每步调用，不拥有改门禁与改流程拓扑的权力。**

进化靠 **SOP（标准作业程序）版本发版**：badcase（失败案例）/门禁分析 → 同拓扑多候选 → 评测/灰度 → 晋级；同质主流程难涌现新拓扑，反哺擅长策略精修而非流程自生长。

---

## 第七层：MCP Layer（外部能力接入层）

### 1. 主要干什么

**MCP（模型上下文协议）层**把「模型/子 Agent（智能体）要用的外部能力」（知识库、检索、CRM、文件系统、业务 API（应用程序接口）等）收成**独立、契约化的工具服务**，用统一协议（Model Context（上下文） Protocol）接入。

编排器与门禁只认「工具名 + schema（结构约定）」，不把实现焊死在主进程里。

| 职责 | 含义 |
|------|------|
| **能力外置** | 检索、业务 API（应用程序接口）等做成 MCP（模型上下文协议） Server，而不是写死在 Agent（智能体）代码里 |
| **统一接入** | 子 Agent（智能体）通过同一套「列工具 / 调工具」协议用能力，换后端不必改 Control/Skills（技能） |
| **配合门禁** | `tool_allowlist` 写的是 MCP（模型上下文协议）工具名；没挂上的服务等于不存在 |
| **可热插拔** | 换知识源、升级检索、加新 API（应用程序接口）：启停/替换 MCP（模型上下文协议）服务即可 |
| **可离线测** | 不连主模型也能单测 MCP（契约、权限、返回结构） |

```text
Control / Step 策略：tool_allowlist = ["kb.search", "crm.get_user"]
        │
        ▼
Sub-Agent 沙箱内发起 tool call
        │
        ▼
MCP Layer：按协议路由到对应 MCP Server
        │
        ▼
外部系统（向量库、DB、HTTP API…）
        │
        ▼
结果回 Working →（可选）Context → 生成 → VERIFY
```

与 RAG（检索增强生成）：RAG（检索增强生成） **可以是**某个 MCP（模型上下文协议）工具背后的实现（如 `kb.search` → RAG（检索增强生成）流水线）；**MCP（模型上下文协议）管接入与边界，RAG（检索增强生成）管检索怎么做得好**。

---

### 2. 为什么要这么干

| 若不做 MCP（能力写死在主 Agent（智能体）） | 做成 MCP（模型上下文协议）层 |
|----------------------------------|-------------|
| 换知识源要改主仓、重发整站 | 换 Server / 改连接配置（热插拔） |
| 工具实现与编排、门禁缠在一起，难单测 | 契约 + Mock Server，**离线测工具层** |
| 多 Agent（智能体）易共用「什么都能调」的上帝客户端 | 按 Step 挂不同 MCP（模型上下文协议）工具子集，**硬隔离更好落** |
| 团队边界糊：做检索必须懂整套 Harness（工程控制架） | 检索/业务 API（应用程序接口）团队只维护 MCP（模型上下文协议） Server |
| 审计难：调用散落各处 | 统一经 MCP（模型上下文协议）出口，便于日志与鉴权 |

**MCP（模型上下文协议）解决的工程问题**：把外部能力做成可替换、可测、可治理的服务边界，让 Harness（工程控制架）核心（编排 + 门禁 + 记忆 + 上下文）保持稳定——不是多一个时髦协议，更不是让模型拥有随便连外部世界的权力。

---

### 3. 和「普通 tool 函数」差在哪

本地 `def search(...):` 也能当工具，但 MCP（模型上下文协议）强调：

1. **进程/服务边界**：实现可独立部署、独立扩缩容  
2. **标准契约**：输入输出 schema（结构约定）、错误码约定清楚  
3. **发现与插拔**：运行时列出 tools，按配置挂载  
4. **与门禁对齐**：白名单 = 允许的 MCP（模型上下文协议） tool 名；扩权改配置，不是改 prompt（提示词）  

Harness（工程控制架）里：**LLM（大语言模型）只在白名单内选已挂载的 MCP（模型上下文协议）工具并填参**；挂哪些工具仍由代码/Step 策略决定。

---

### 4. 疑问解答：是不是「工具箱，和 Agent（智能体）隔离，注册后才拿工具」？

**大方向对**，表述收紧如下：

#### MCP（模型上下文协议） ≈ 工具箱（更准确：工具的「插座 + 实现」）

- **MCP（模型上下文协议） Server**：真正提供能力的一侧  
- **Harness（工程控制架）侧**：按协议发现/调用  
- 和 Sub-Agent（子智能体） **隔离**：Agent（智能体）内不内嵌业务实现，只认「工具名 + 参数」

#### 不是「主 Agent（大语言模型）随便注册」，而是「编排器按步挂载」

```text
代码编排器进入某 Step
  → 读本步策略：allowed_agents + tool_allowlist
  → dispatch 某个子 Agent，并挂上本步允许的 MCP 工具子集
  → 子 Agent 只能从这批已挂载工具里选、填参、执行
  → 未挂载的 MCP 工具 = 不存在
```

| 说法 | 是否采纳 |
|------|----------|
| MCP（模型上下文协议）是工具箱，和 Agent（智能体）隔离 | ✅ |
| Agent（智能体）「注册/被调度」后才从 MCP（模型上下文协议）拿工具执行 | ✅ 意思对 |
| 「主 Agent（智能体）注册」= 主 LLM（大语言模型）自己开门禁加工具 | ❌ |
| 「主 Agent（智能体）注册」= **Control/编排器（代码）按 Step 注册/挂载** | ✅ |

**一句话：MCP（模型上下文协议）提供可用工具池；每次子 Agent（智能体）被编排器调度进某步时，只带上该步白名单里的 MCP（模型上下文协议）工具再执行。**

---

### 5. 与七层闭环的位置

```text
STAGE / Skills 定业务焦点
  → Control 定本步 allowed_agents + tool_allowlist（MCP 工具名）
  → Context 装配提示词
  → Sub-Agent 在沙箱内调用已挂载 MCP 工具（如 RAG、业务 API）
  → 结果进 Working
  → VERIFY（可含 grounded：引用 ⊆ 本步工具/RAG 结果）
  → Memory commit
```

---

### 6. 一句话收束

**MCP（模型上下文协议）层 = 外部能力的标准插座与保险闸：主工程负责编排与门禁，具体本事插在 MCP（模型上下文协议）上；为热插拔、可离线测、团队解耦、工具权限可审计。子 Agent（智能体）与工具实现隔离，仅在被编排器按 Step 挂载白名单后才能从 MCP（模型上下文协议）取工具执行。**
