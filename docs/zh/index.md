---
layout: home
---

<script setup>
import { withBase } from 'vitepress'
import HmzRun from '../.vitepress/theme/components/home/HmzRun.vue'
import HmzTabs from '../.vitepress/theme/components/home/HmzTabs.vue'
import WhyFlows from '../.vitepress/theme/components/home/WhyFlows.vue'

const WAYS = [
  { id: 'prompt', name: '在提示符下', hint: 'hmz' },
  { id: 'exec', name: '在 shell 中', hint: 'hmz exec' },
]
</script>

<HmzHero />

## 为什么需要流程

编码智能体在**会话**中工作：一段对话，一轮接一轮。给它一个目标，它会一直做到自称完成为止。小任务这样足够。大任务里，会话越拉越长，智能体迷失方向，也没人核查它的话。更长的会话解决不了这个问题。能解决的是为这些会话做一份计划：哪个智能体被问什么、在哪个对话里、谁核查工作、何时停下。这份计划就是一个**流程**（flow）。

没有一个流程适合所有任务，因为任务是不同种类的问题：

- **构建项目是约束满足。** 要求的功能必须齐、测试必须过、lint 必须干净。没人在意是哪个程序做到的，所以第一个通过全部检查的程序就够了。先工作、再让一个全新智能体反复评审、直到无可修改的循环，就能找到这样一个程序。这就是 RLCR，[Humanize 1](https://humanfia.ai/flows/humanize1) 的循环，而 [`rlar`](https://humanfia.ai/flows/rlar) 随 humanize 一同发布。
- **快速内核是优化问题。** 每个候选都跑得起来，过不过不是问题，问题是最好的那个有多快。在我们的内核实验里，Codex 倾向微调，爬到最近的极值点就停下；Claude Code 倾向大步重写，有时反而比起点更慢。[Flame-chase](https://humanfia.ai/flows/flame-chase) 让两个全新智能体在同一个仓库上交替：一次重写把工作从一个极值点上抬起，再由微调爬上下一个。
- **定理证明、超参数调优等**又是别的问题，各有自己的流程，比如 [递归 Lean 证明器](https://humanfia.ai/flows/recursive-lean-prover)。

<WhyFlows />

基准测试上的流程级消融实验说了同样的事：赢在一种任务上的流程，输在另一种上。humanize 让你按任务选流程、配置流程，或用 Python 编织你自己的流程。消融实验的发现见 humanfia.ai 上的 [Flow Science](https://humanfia.ai/research/flow-science#findings)。

<p class="hmz-vision">方向所在：一个把 <b>100T 模型</b>、<b>每秒 1K token</b>、<b>1M 个智能体</b>、<b>1M 个环境</b>和 <b>1M 分钟</b>放到同一个大问题上的流程框架。这是一个方向，不是一次测量。</p>

## 它如何组合

流程是一个 Python 函数，说明哪些智能体轮流上场、各自被问什么、何时结束。这是 `rlar`（humanize 自带的评审循环）的一次运行：一个智能体干活，一个全新的读它做的，评审就是下一轮的提示。

<HmzRun />

每次运行之下都是同样的四层：流程做决定，humanize 执行并记录每一轮，编码智能体干活，工作落进一个环境。

<HmzArch />

## 运行一个流程

安装 humanize，给它一个带 bug 的临时仓库，看智能体把它修好。

::: warning 此处的智能体不经确认直接行动
每个流程的智能体都以跳过审批的方式运行：它们自行编辑文件、运行命令、提交代码。请用临时仓库，并在把流程指向你在乎的工作前阅读[安全](/zh/user/security)。
:::

<div class="hmz-steps">

### 安装 humanize

你需要 Python 3.12 或更新版本、[uv](https://docs.astral.sh/uv/) 或 pip，以及一个已登录的编码智能体 CLI：Claude Code、Codex 或[其他任何一个](/features/backends)。humanize 在那个登录之下驱动它。

::: code-group

```sh [uv]
uv tool install hmz
```

```sh [pip]
pip install hmz
```

```sh [DeepSeek Harness 或 Kimi Code]
uv tool install 'hmz[all]'
```

:::

不安装也可以试用：在本页写 `hmz` 的地方，键入 `uvx hmz` 即可（[更多](/zh/user/installation#uvx)）。

### 建一个临时仓库

`calc.py` 把加法写成了减法。这个 bug 就是待办的工作。

```sh
mkdir -p ~/tmp/humanize-demo && cd ~/tmp/humanize-demo && git init -q
printf 'def add(a, b):\n    return a - b\n' > calc.py
git add -A && git commit -qm "a calculator with a bug in it" && git tag before
```

### 在它上面跑一个流程

流程选 `ralph_loop`。它一轮又一轮给智能体同一个任务，每轮都在全新会话里，而且工作做完也不停：由它的预算或你来停。

<HmzTabs :tabs="WAYS" label="运行一个流程的两种方式">
<template #prompt>

在仓库里打开终端界面。首次启动会问一次是否向 humanize 报告错误；随意作答。

```sh
hmz
```

输入 `$`、流程名和任务：

<div class="hmz-term" data-title="hmz">

```text
$ralph_loop Fix the bug in calc.py.
```

</div>

第一次运行会打开流程菜单并占住这一行。为它唯一角色 `agent` 选 CLI、模型和投入度，设一个预算，按**保存**：这会保存配置并开始运行。目录会记住配置：下次同一行直接开始运行。

也可以从菜单开始：输入 `/flow`，选 `ralph_loop`，配置好，按**保存**，然后在 `❯` 处输入任务。

按两次 <kbd>ctrl+c</kbd> 停止流程，`/` 列出全部命令，`/exit` 退出。想在浏览器里看同一次运行，在同一目录的另一个终端里运行 `hmz web`：见[在浏览器中](/user/web)。

<HmzCast name="tui" alt="hmz 打开，/ 列出命令，/flow 列出此处的流程，→ 打开一个流程的角色；esc 逐层退回首个界面" />

<p class="hmz-cast-caption">
  <code>/</code> 列出命令，<code>/flow</code> 列出待配置的流程。以替身 CLI 录制，没有智能体上场。
</p>

</template>
<template #exec>

同一运行写成一行。`ralph_loop` 随 humanize 发布，无需先装。选你的 CLI：

::: code-group

```sh [Claude Code]
hmz exec -f ralph_loop -a agent=claude/claude-opus-5:high -p budget.duration=10m "Fix the bug in calc.py."
```

```sh [Codex]
hmz exec -f ralph_loop -a agent=codex/gpt-5.6-sol:high -p budget.duration=10m "Fix the bug in calc.py."
```

```sh [Antigravity]
hmz exec -f ralph_loop -a agent=agy/gemini-3.7-flash-high:high -p budget.duration=10m "Fix the bug in calc.py."
```

```sh [Qwen Code]
hmz exec -f ralph_loop -a agent=qwen/qwen3-coder-plus:high -p budget.duration=10m "Fix the bug in calc.py."
```

```sh [Kimi Code]
hmz exec -f ralph_loop -a agent=kimi/kimi-code/k3:high -p budget.duration=10m "Fix the bug in calc.py."
```

```sh [Grok Build]
hmz exec -f ralph_loop -a agent=grok/grok-4.6:high -p budget.duration=10m "Fix the bug in calc.py."
```

:::

- `-f` 指定流程。
- `-a agent=CLI/MODEL:EFFORT` 填它的唯一角色 `agent`：哪个 CLI、哪个模型、模型想多用力。
- `-p budget.…` 限定这次运行可花费的上限：`duration`（时长）、`cost`（美元）或 `output_tokens`。不设则拒绝启动。

按 <kbd>ctrl+c</kbd> 停止。[命令行](/reference/cli) 有全部旗标。

</template>
</HmzTabs>

### 看它做了什么

```sh
git diff before
```

```diff
 def add(a, b):
-    return a - b
+    return a + b
```

</div>

::: details 如果它没有启动

| 如果 | 那么 |
| --- | --- |
| `hmz exec` 报 `not installed` | `ralph_loop` 是内置的；来自 [flowverse](/weaver/flowverses) 的流程要安装后才能运行。打开 `hmz`，输入 `/flow`，按**安装更多…**，打开 `official` 并安装。 |
| 模型被拒绝 | 模型 id 随 CLI 版本变化，账号决定可用哪些。在 `hmz` 里输入 `/flow ralph_loop`，打开 `agent` 行：它会列出你的 CLI 提供的模型。 |
| 你的 CLI 不在标签里 | 全部后端及登录方式见[安装](/zh/user/installation)。 |

:::

**下一步。**[用户指南](/zh/user/) 讲界面、账号与回看运行；它的教程各领一件真实工作走完全程：[攻克基准](/user/tutorials/take-home)、[移植项目](/user/tutorials/port-a-project)、[造一个编码智能体](/user/tutorials/build-an-agent)。无人值守、每次推送或每夜运行，见 [CI 中的 humanize](/user/ci)。humanfia.ai 上的 [Flows](https://humanfia.ai/flows/) 列出其余可运行的流程。

## 编织一个流程

流程是一个 async Python 函数，声明它驱动的智能体。把下面这个存进临时仓库的 `.hmz/flows/twice/__init__.py`。它让一个智能体做任务，然后在同一会话里评审自己的工作。会话只是它的历史；每一轮说明它在哪里运行，未指定环境的一轮在启动目录里运行：

::: code-group

```python{6,19-23} [.hmz/flows/twice/__init__.py]
from hmz.flows import Agent, AgentCollection, EnvCollection, FlowContext, FlowParams
from hmz.flows import LocalEnv, flow


class Agents(AgentCollection):
    builder: Agent  # 一个角色：-a builder=… 填它


class Envs(EnvCollection):
    workspace: LocalEnv  # 运行所在目录


@flow(agents=Agents, envs=Envs, params=FlowParams)
async def twice(
    task: str, *, agents: Agents, envs: Envs, params: FlowParams, ctx: FlowContext
) -> None:
    """先做工作，再回读并修正错处。"""
    builder = agents["builder"]
    session = await builder.spawn()  # 一个会话：只是它的历史
    await builder.run(task, session=session, env=envs["workspace"])
    await builder.run(  # 同一会话，所以它记得；未指定 env：再次用工作区
        "Now review what you just did, and fix anything that is wrong.", session=session
    )
```

:::

像任何流程一样运行：在 shell 里按名，或在提示符下用 `$@local/twice`。

::: code-group

```sh [hmz exec]
hmz exec -f twice -a builder=claude/claude-opus-5:high -p budget.cost=5 "Add a subtract function to calc.py."
```

```text [在提示符下]
$@local/twice Add a subtract function to calc.py.
```

:::

**下一步。**[编织者指南](/zh/weaver/) 是流程对智能体能问的一切：循环、参数、目标、钩子与工作树。[在测试下构建](/weaver/tutorials/build-under-test)从零写一个有用的流程；[测试一个流程](/weaver/testing-flows)让它在替身智能体上先跑，不花一个 token。

## 参与 {#work-on-humanize}

```sh
git clone https://github.com/humanfia/humanize.git && cd humanize
uv sync
```

每一处改动都要通过这两个：

```sh
uv run pre-commit run --all-files
uv run pytest
```

[贡献](/zh/contributing/) 说明改动落在哪里、检查要求什么、何时还要跑驱动真实 CLI 的测试。[你的第一个补丁](/contributing/tutorials/first-patch)把一处改动从克隆带到拉取请求。

## 接下来去哪

<div class="hmz-paths by-three">
  <a :href="withBase('/zh/user/')">
    <strong>用户指南</strong>
    <span>运行流程：界面、智能体与账号、工作落点、回看运行。</span>
  </a>
  <a href="https://humanfia.ai/flows/">
    <strong>流程目录</strong>
    <span>humanize 自带的与官方 flowverse 列出的每个流程，各配播放的循环，在 humanfia.ai。</span>
  </a>
  <a :href="withBase('/zh/weaver/')">
    <strong>编织者指南</strong>
    <span>编写流程：流程能对智能体要求什么，如何编写与测试。</span>
  </a>
  <a :href="withBase('/features/')">
    <strong>特性</strong>
    <span>humanize 如何工作，画给你看：每个能力一张图。</span>
  </a>
  <a :href="withBase('/reference/')">
    <strong>参考</strong>
    <span>每个命令、按键、旗标与 Python 调用，逐一说明。</span>
  </a>
  <a :href="withBase('/zh/contributing/')">
    <strong>贡献</strong>
    <span>参与 humanize 本身的开发：分层、检查与这些文档。</span>
  </a>
</div>
