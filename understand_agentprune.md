# AgentPrune `run_mmlu.py` 参数笔记

本文整理 `experiments/run_mmlu.py` 的运行参数、默认值、可选值、作用，以及它们在代码里的出处。

## 总入口

MMLU 实验入口是：

```bash
python -u experiments/run_mmlu.py [args]
```
参数定义集中在 [experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 20 行） 的 `parse_args()`，主要是第 22-49 行。参数解析后，在 `main()` 里构造 `Graph`、下载/读取 MMLU 数据、可选训练剪枝、最后评估。

关键调用链：

```text
parse_args()
  -> main()
    -> get_kwargs(mode, N)
    -> Graph(...)
    -> download()
    -> MMLUDataset('dev'), MMLUDataset('val')
    -> if optimized_spatial or optimized_temporal: train(...)
    -> evaluate(...)
```

代码出处：

- 参数定义：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 22-49 行）
- 参数传给 Graph：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 67-73 行）
- 参数传给 train：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 78-80 行）
- 参数传给 evaluate：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 82 行）
- 拓扑生成：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 87-160 行）

## 参数总表

| 参数 | 默认值 | 可选值/类型 | 作用 | 主要出处 |
| --- | --- | --- | --- | --- |
| `--mode` | `FullConnected` | 固定枚举，见下文 | 选择 agent 通信拓扑，以及是否混入 Fake agent | [run_mmlu.py](./experiments/run_mmlu.py)（第 22-25 行）, [run_mmlu.py](./experiments/run_mmlu.py)（第 87-160 行） |
| `--lr` | `0.1` | `float` | Adam 优化 graph 边 logits 的学习率 | [run_mmlu.py](./experiments/run_mmlu.py)（第 26-27 行）, [train_mmlu.py](./experiments/train_mmlu.py)（第 33 行） |
| `--batch_size` | `4` | `int` | train/eval 每批处理多少道题；也影响并发 API 请求数 | [run_mmlu.py](./experiments/run_mmlu.py)（第 28-29 行）, [train_mmlu.py](./experiments/train_mmlu.py)（第 41 行）, [evaluate_mmlu.py](./experiments/evaluate_mmlu.py)（第 28-44 行） |
| `--agent_names` | `AnalyzeAgent` | agent 注册名列表 | 指定图中使用哪些 agent 类型 | [run_mmlu.py](./experiments/run_mmlu.py)（第 30-31 行）, [run_mmlu.py](./experiments/run_mmlu.py)（第 63 行）, [graph.py](./AgentPrune/graph/graph.py)（第 124-133 行） |
| `--agent_nums` | `5` | int 列表 | 每种 agent 类型创建几个；必须和 `agent_names` 等长 | [run_mmlu.py](./experiments/run_mmlu.py)（第 32-34 行）, [run_mmlu.py](./experiments/run_mmlu.py)（第 53-54 行）, [run_mmlu.py](./experiments/run_mmlu.py)（第 63 行） |
| `--num_iterations` | `10` | `int` | 优化剪枝训练轮数；只有开启优化时生效 | [run_mmlu.py](./experiments/run_mmlu.py)（第 34-35 行）, [run_mmlu.py](./experiments/run_mmlu.py)（第 78-80 行）, [train_mmlu.py](./experiments/train_mmlu.py)（第 35 行） |
| `--imp_per_iterations` | `5` | `int` | 每隔多少轮调用一次剪枝 | [run_mmlu.py](./experiments/run_mmlu.py)（第 36-37 行）, [train_mmlu.py](./experiments/train_mmlu.py)（第 82-83 行） |
| `--num_rounds` | `1` | `int` | 每道题 agent 通信/推理轮数 | [run_mmlu.py](./experiments/run_mmlu.py)（第 38-39 行）, [train_mmlu.py](./experiments/train_mmlu.py)（第 47 行）, [evaluate_mmlu.py](./experiments/evaluate_mmlu.py)（第 56 行）, [graph.py](./AgentPrune/graph/graph.py)（第 270-293 行） |
| `--pruning_rate` | `0.25` | `float` | 每次剪枝时剪掉的边比例 | [run_mmlu.py](./experiments/run_mmlu.py)（第 40-41 行）, [train_mmlu.py](./experiments/train_mmlu.py)（第 82-83 行）, [graph.py](./AgentPrune/graph/graph.py)（第 317-339 行） |
| `--llm_name` | `gpt-4-1106-preview` | `str` | 传给 OpenAI-compatible 后端的模型名 | [run_mmlu.py](./experiments/run_mmlu.py)（第 42-43 行）, [graph.py](./AgentPrune/graph/graph.py)（第 58 行）, [llm_registry.py](./AgentPrune/llm/llm_registry.py)（第 19-27 行） |
| `--domain` | `mmlu` | `str` | 任务域名，用于选择 prompt set；MMLU 实验一般不改 | [run_mmlu.py](./experiments/run_mmlu.py)（第 44-45 行）, [graph.py](./AgentPrune/graph/graph.py)（第 57 行）, agents 中的 `PromptSetRegistry.get(domain)` |
| `--decision_method` | `FinalRefer` | decision agent 注册名 | 最终节点如何汇总多个 agent 输出 | [run_mmlu.py](./experiments/run_mmlu.py)（第 46-47 行）, [graph.py](./AgentPrune/graph/graph.py)（第 62 行）, [final_decision.py](./AgentPrune/agents/final_decision.py)（第 1 行） |
| `--optimized_spatial` | `False` | flag | 开启空间通信边优化 | [run_mmlu.py](./experiments/run_mmlu.py)（第 48 行）, [graph.py](./AgentPrune/graph/graph.py)（第 60 行）, [graph.py](./AgentPrune/graph/graph.py)（第 71-74 行） |
| `--optimized_temporal` | `False` | flag | 开启跨轮 temporal 边优化 | [run_mmlu.py](./experiments/run_mmlu.py)（第 49 行）, [graph.py](./AgentPrune/graph/graph.py)（第 61 行）, [graph.py](./AgentPrune/graph/graph.py)（第 76-79 行） |

## `--mode`

`--mode` 决定 agent 图的初始拓扑，以及是否让部分 agent 变成 Fake/恶意角色。

定义出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 22-25 行）

可选值：

```text
DirectAnswer
FullConnected
Random
Chain
Debate
Layered
Star
Mesh
FakeFullConnected
FakeRandom
FakeChain
FakeStar
FakeMesh
FakeAGRandom
FakeAGFull
```

实现出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py) 的 `get_kwargs()`，第 87-160 行。

各模式大意：

| mode | 空间边 spatial | temporal 边 | 备注 |
| --- | --- | --- | --- |
| `DirectAnswer` | `[[0]]` | `[[0]]` | 单 agent 直接回答；`node_kwargs=[{'role':'Normal'}]` |
| `FullConnected` | 非自己到自己的边全开 | 全开 | 普通全连接多 agent |
| `Random` | 随机 0/1，自己到自己为 0 | 随机 0/1 | 随机拓扑 |
| `Chain` | 链式 `j -> j+1` | 首尾 temporal 边 | 顺序链式通信 |
| `Debate` | 全 0 | 全开 | 空间通信关闭，主要靠多轮 temporal 信息 |
| `Layered` | 随机分两层，上一层连下一层 | 全开 | 分层拓扑 |
| `Mesh` | 上三角连接，前面的 agent 连后面的 agent | 全开 | mesh/DAG 风格拓扑 |
| `Star` | 0 号 agent 连其他 agent | 全开 | 星型拓扑 |
| `Fake*` | 对应普通拓扑 | 对应普通拓扑 | 部分节点 role 设为 `Fake` |
| `FakeAG*` | 对应普通拓扑 | 对应普通拓扑 | 部分节点 role 设为 `Fake`，非 Fake 节点 role 为 `None` |

Fake 角色设置出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 151-154 行）

```python
if 'Fake' in mode and 'AG' not in mode:
    node_kwargs = [{'role':'Fake'} if i % 2 == N % 2 else {'role':'Normal'} for i in range(N)]
elif 'Fake' in mode and 'AG' in mode:
    node_kwargs = [{'role':'Fake'} if i % 2 == N % 2 else {'role':None} for i in range(N)]
```

## `--agent_names` 和 `--agent_nums`

这两个参数共同决定图里有哪些 agent，以及每种 agent 有几个。

定义出处：

- `--agent_names`：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 30-31 行）
- `--agent_nums`：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 32-33 行）

长度检查出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 53-54 行）

展开逻辑出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 63 行）

```python
agent_names = [name for name,num in zip(args.agent_names,args.agent_nums) for _ in range(num)]
```

例子：

```bash
--agent_names AnalyzeAgent --agent_nums 5
```

得到：

```text
AnalyzeAgent, AnalyzeAgent, AnalyzeAgent, AnalyzeAgent, AnalyzeAgent
```

例子：

```bash
--agent_names AnalyzeAgent AdverarialAgent --agent_nums 3 2
```

得到：

```text
AnalyzeAgent x 3 + AdverarialAgent x 2
```

可用 agent 来自注册表导入，主要看 [AgentPrune/agents/__init__.py](./AgentPrune/agents/__init__.py)（第 1-25 行） 和各 agent 文件里的 `@AgentRegistry.register(...)`。

MMLU 常用：

- `AnalyzeAgent`
- `AdverarialAgent`

注意：仓库里名字拼成了 `AdverarialAgent`，不是 `AdversarialAgent`。

## `--decision_method`

最终决策节点负责汇总多个 agent 的输出。

定义出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 46-47 行）

传入 Graph 出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 67-73 行）

Graph 中创建 decision node 出处：[AgentPrune/graph/graph.py](./AgentPrune/graph/graph.py)（第 62 行）

```python
self.decision_node = AgentRegistry.get(decision_method, **{"domain": self.domain, "llm_name": self.llm_name})
```

常用可选值：

| decision_method | 作用 | 代码出处 |
| --- | --- | --- |
| `FinalRefer` | 把其他 agent 输出拼成 prompt，再调用 LLM 做最终判断 | [AgentPrune/agents/final_decision.py](./AgentPrune/agents/final_decision.py)（第 68-105 行） |
| `FinalDirect` | 直接取最后一个空间前驱的输出 | [AgentPrune/agents/final_decision.py](./AgentPrune/agents/final_decision.py)（第 107-139 行） |
| `FinalMajorVote` | 对各 agent 输出做后处理后多数投票 | [AgentPrune/agents/final_decision.py](./AgentPrune/agents/final_decision.py)（第 142-186 行） |
| `FinalWriteCode` | HumanEval/code 任务用，带代码测试反馈 | [AgentPrune/agents/final_decision.py](./AgentPrune/agents/final_decision.py)（第 9-65 行） |

MMLU baseline 常用 `FinalMajorVote`；如果用 `FinalRefer`，最后一步还会额外调用一次 LLM。

## `--optimized_spatial` 和 `--optimized_temporal`

这两个 flag 控制是否学习/优化通信边。

定义出处：

- `--optimized_spatial`：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 48 行）
- `--optimized_temporal`：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 49 行）

传入 Graph 出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 67-73 行）

Graph 参数定义和保存出处：[AgentPrune/graph/graph.py](./AgentPrune/graph/graph.py)（第 33-45 行）, [AgentPrune/graph/graph.py](./AgentPrune/graph/graph.py)（第 60-61 行）

可训练参数定义出处：[AgentPrune/graph/graph.py](./AgentPrune/graph/graph.py)（第 71-79 行）

```python
self.spatial_logits = torch.nn.Parameter(..., requires_grad=optimized_spatial)
self.temporal_logits = torch.nn.Parameter(..., requires_grad=optimized_temporal)
```

如果两个 flag 都不开：

- 不会调用 `train()`
- 直接在 val split 上评估
- `--num_iterations`、`--lr`、`--imp_per_iterations`、`--pruning_rate` 基本不生效

如果任意一个开启：

- 先用 MMLU dev split 做优化训练
- 再用 MMLU val split 做评估

触发训练出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 78-80 行）

```python
if args.optimized_spatial or args.optimized_temporal:
    await train(...)
```

## `--num_iterations`

含义：训练/优化通信图的迭代轮数。

定义出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 34-35 行）

传入训练出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 78-80 行）

真正生效出处：[experiments/train_mmlu.py](./experiments/train_mmlu.py)（第 35 行）

```python
for i_iter in range(num_iters):
```

例如：

```bash
--num_iterations 10
```

表示训练阶段会跑：

```text
Iter 0
Iter 1
...
Iter 9
```

每个 iteration 会取 `batch_size` 条 dev 数据，调用 agent 得到答案，计算 utility 和 loss，然后更新边 logits。

loss 位置：[experiments/train_mmlu.py](./experiments/train_mmlu.py)（第 57-73 行）

```python
single_loss = - log_prob * utility
total_loss = torch.mean(torch.stack(loss_list))
total_loss.backward()
optimizer.step()
```

## `--imp_per_iterations` 和 `--pruning_rate`

`--imp_per_iterations` 表示每隔多少轮剪枝一次。

定义出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 36-37 行）

训练中使用出处：[experiments/train_mmlu.py](./experiments/train_mmlu.py)（第 82-83 行）

```python
if (i_iter+1) % imp_per_iters == 0:
    spatial_masks, temporal_masks = graph.update_masks(pruning_rate)
```

`--pruning_rate` 表示每次剪枝比例。

定义出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 40-41 行）

剪枝实现出处：[AgentPrune/graph/graph.py](./AgentPrune/graph/graph.py)（第 317-339 行）

大意：

- 找当前没有被 mask 掉的边
- 根据 `spatial_logits` 或 `temporal_logits` 排序
- 按 `pruning_rate` 比例剪掉 logit 较小的一批边
- 被剪掉的边 mask 设为 0

例子：

```bash
--num_iterations 10 --imp_per_iterations 5 --pruning_rate 0.25
```

表示训练 10 轮，每 5 轮剪一次，每次按 25% 比例剪掉当前可用边中较弱的边。

## `--lr`

Adam 优化器学习率。

定义出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 26-27 行）

传入训练出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 79-80 行）

优化器定义出处：[experiments/train_mmlu.py](./experiments/train_mmlu.py)（第 33 行）

```python
optimizer = torch.optim.Adam([graph.spatial_logits, graph.temporal_logits], lr=lr)
```

注意：这里优化的不是 LLM 参数，也不是 agent 参数，只是 graph 的边 logits。

## `--batch_size`

含义：每批处理多少道题。

定义出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 28-29 行）

训练中使用：[experiments/train_mmlu.py](./experiments/train_mmlu.py)（第 41-49 行）

```python
for i_record, record in zip(range(batch_size), loader):
```

评估中使用：[experiments/evaluate_mmlu.py](./experiments/evaluate_mmlu.py)（第 28-44 行）

```python
eval_loader(batch_size=eval_batch_size)
```

注意：每道题会触发多个 agent 调用 LLM，所以 `batch_size` 越大，并发 API 请求越多，可能更快，也更容易被限流。

## `--num_rounds`

含义：每道题进行多少轮 agent 通信。

定义出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 38-39 行）

训练中传入 graph：[experiments/train_mmlu.py](./experiments/train_mmlu.py)（第 47 行）

评估中传入 graph：[experiments/evaluate_mmlu.py](./experiments/evaluate_mmlu.py)（第 56 行）

Graph 中真正循环出处：[AgentPrune/graph/graph.py](./AgentPrune/graph/graph.py)（第 270-293 行）

```python
for round in range(num_rounds):
    log_probs += self.construct_spatial_connection()
    log_probs += self.construct_temporal_connection(round)
    ...
    self.update_memory()
```

轮数越多，agent 越有机会利用上一轮的 temporal memory，但 API 成本也会上升。

## `--llm_name`

含义：传给 OpenAI-compatible API 的模型名。

定义出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 42-43 行）

传入 Graph 出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 67-73 行）

Graph 保存出处：[AgentPrune/graph/graph.py](./AgentPrune/graph/graph.py)（第 58 行）

LLMRegistry 处理出处：[AgentPrune/llm/llm_registry.py](./AgentPrune/llm/llm_registry.py)（第 19-27 行）

```python
if model_name is None or model_name == "":
    model_name = "gpt-4o"

if model_name == 'mock':
    model = cls.registry.get(model_name)
else:
    model = cls.registry.get('GPTChat', model_name)
```

实际 API 请求在 [AgentPrune/llm/gpt_chat.py](./AgentPrune/llm/gpt_chat.py)（第 1 行） 里，读取 `.env` 中：

```env
BASE_URL=...
API_KEY=...
```

如果你的后端是 OpenAI-compatible，那么 `--llm_name` 可以写后端支持的任意模型名，例如：

```bash
--llm_name gpt-4o-mini
--llm_name qwen/qwen3.5-flash-02-23
```

## `--domain`

含义：任务域名，用来选择 prompt set。

定义出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 44-45 行）

传入 Graph 出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 67 行）

Graph 保存出处：[AgentPrune/graph/graph.py](./AgentPrune/graph/graph.py)（第 57 行）

在各 agent 中会通过：

```python
PromptSetRegistry.get(domain)
```

选择对应 prompt。MMLU 实验一般保持默认：

```bash
--domain mmlu
```

## 代码里的固定值

`run_mmlu.py` 里还有一个没有做成命令行参数的固定值：

```python
limit_questions = 153
```

出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 65 行）

评估时传入：

```python
evaluate(..., limit_questions=limit_questions, ...)
```

出处：[experiments/run_mmlu.py](./experiments/run_mmlu.py)（第 82 行）

所以当前 MMLU 评估默认只跑 val split 的前 153 道题，而不是完整 val 集 1531 道。

## 一个例子

```bash
python -u experiments/run_mmlu.py \
  --agent_nums 1 \
  --mode DirectAnswer \
  --decision_method FinalMajorVote \
  --agent_names AdverarialAgent \
  --batch_size 1 \
  --llm_name qwen/qwen3.5-flash-02-23
```

含义：

- 用 1 个 `AdverarialAgent`
- `DirectAnswer`，不做多 agent 通信
- 最终节点用 `FinalMajorVote`；因为只有 1 个 agent，所以等价于取该 agent 答案
- 每批 1 道题
- 使用模型名 `qwen/qwen3.5-flash-02-23`
- 没有开启 `--optimized_spatial` 或 `--optimized_temporal`，所以不训练/不剪枝，直接评估

如果要复现 AgentPrune 剪枝优化，更像 README 的命令是：

```bash
python -u experiments/run_mmlu.py \
  --agent_nums 6 \
  --mode FakeAGFull \
  --batch_size 4 \
  --num_iterations 10 \
  --imp_per_iterations 5 \
  --pruning_rate 0.25 \
  --num_rounds 1 \
  --optimized_spatial \
  --optimized_temporal
```
