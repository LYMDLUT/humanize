<script setup>
import RunAnatomy from '../../.vitepress/theme/components/user-concepts/RunAnatomy.vue'
</script>

# 术语表

其余文档用到的词，每词一条，附深入阅读之处。

```text
epic ─── 一次流程运行的完整记录
 │
flow ─── 循环：谁被问什么，直到何时
 │
 ├── 角色 "builder" ─── 智能体：后端、模型、投入度
 │                       └── session ─── 一轮、一轮、一轮 …
 │
 └── 角色 "reviewer" ── 智能体：…
                         └── session ─── 一轮 …
```

## 用这些词说一次运行

下面的多数词汇于一条命令。这条运行 [`rlar`](https://humanfia.ai/flows/rlar) 流程：一个执行者干活，一个评审者逐轮检查：

```sh
hmz exec -f rlar \
    -a actor=claude/claude-opus-5-5:high \
    -a reviewer=codex/gpt-5.6-sol:high \
    -p budget.duration=2h,budget.cost=10 \
    "fix the flaky payment test"
```

1. **`-f rlar`** 指名[流程](#flow)：说明谁被问什么、何时结束的循环。
2. **`actor=` 与 `reviewer=`** 是它的两个[角色](#role)。各以一个[智能体](#agent)填充：一个[后端](#backend)（`claude`、`codex`）、一个模型、一个[投入度](#effort)（`high`）。不带 `@` 时以你登录 CLI 的[提供商](#provider)身份运行。
3. **`-p budget.duration=2h,budget.cost=10`** 是[预算](#budget)：两小时或十美元，先到先停。
4. **引号里那行**是任务。

运行时，执行者保持一个[会话](#session)，评审者每轮开新的；会话内每次交流是一[轮](#turn)。它们都在[工作区](#workspace)（你运行命令的目录）里工作，那是本次运行的一个[环境](#environment)。整次运行记为一个 [epic](#epic)，其后的[追踪](#trace)把每一轮画上同一条时间线。

<RunAnatomy />

同一次运行在提示符下是 `/flow`，同样的选择在菜单里做，任务敲在 `❯`。见[首次运行](/zh/user/first-run)。

## 智能体（Agent）{#agent}

填流程一个角色的东西：一个[后端](#backend)、一个模型、一个[投入度](#effort)，可选它以哪个[提供商](#provider)运行。命令行里写在它填的角色后面：

```text
reviewer=claude@work/claude-opus-4-8:high
   │       │     │         │           └── 投入度
   │       │     │         └── 模型
   │       │     └── 提供商；省略时以你的登录运行
   │       └── 后端
   └── 角色
```

智能体自己不持有会话：它是它开的每个[会话](#session)运行的配置。配置相同的两个智能体仍是两个。在 `/flow` 里配置：[首次运行](/zh/user/first-run)。

## 后端（Backend）{#backend}

humanize 驱动的编码智能体 CLI：`agy`、`claude`、`codex`、`cursor-agent`、`dsh`、`grok`、`kimi`、`mcode`、`mimo`、`omp`、`opencode`、`pi`、`qwen`。它是智能体的 `cli` 行。见[安装](/zh/user/installation)与[多个后端，一个智能体](/features/backends)。

## 预算（Budget）{#budget}

一次运行被停止前可花费的上限：`duration`（时长）、`cost`（美元）或 `output_tokens`（输出 token 数）。除 `chat` 外每个流程都需要。见[每次运行都有预算](/features/allowances)。单轮也可有流程设置的预算：[一轮可以被切断](/features/budgets)。

## 投入度（Effort）{#effort}

模型想多用力，用后端自己的词：`high`、`max`、`off` 等。每个模型有自己的列表。见[投入度](/user/efforts)。

## 环境（Environment）{#environment}

一轮的工作落在哪里：本目录、另一个目录、或另一台机器上的目录。流程声明它需要的，多数只要本目录。流程每跑一轮指名一个，所以一个[会话](#session)可能在多个环境里工作；未指名的一轮在[工作区](#workspace)里工作。见[远程执行](/user/remote-execution)与[容器](/user/containers)。

## Epic {#epic}

一次流程运行的完整记录：流程、它的智能体、它们开的每个会话。`/epics` 列出此目录的运行。见[追踪](/user/tracing)与[续跑](/user/resuming)。

## 流程（Flow）{#flow}

循环：说明哪个智能体被问什么、按什么顺序、何时结束的 Python。你在 `/flow` 选它，或以 `$` 指名。见[首次运行](/zh/user/first-run)与 [Flows](https://humanfia.ai/flows/)。

## Flowverse {#flowverse}

流程的索引：一个 git 仓库，列出每个流程的各发布及其仓库与 commit。它列出的流程从 `/flow` 安装后即可运行。`official` 是 humanize 自己的；其余在 `/flow` 的 Flowverses 页添加。内置或从 `official` 安装的流程按 `official` 的写法指名：humanfia 自己的写作 `aot`，其他人的写作 `<user>/<flow>`；从别的 flowverse 安装的写作 `@<flowverse>/<flow>`。见 [Flowverse](/weaver/flowverses)。

## 局外人（Outworlder）{#outworlder}

流程眼中的你：流程提问时在提示符前的任何人。`/afk` 说明你不在。见[暂离](/user/afk)。

## 权限（Permission）{#permission}

智能体可触达什么，由流程逐角色声明：工作目录、家目录其余部分、机器、网络。见[权限](/user/permissions)与[安全](/zh/user/security)。

## 提供商（Provider）{#provider}

一个后端的具名账号，与 CLI 自己的登录分开：一个登录、一个 API key、或一个网关。它是智能体的 `account` 行，`as local` 表示以你登录的 CLI 运行。见[账号](/user/settings#accounts)。

## 角色（Role）{#role}

流程声明的具名槽位，如 `agent`、`builder`、`reviewer`。你给每个角色一个智能体；流程说明角色做什么、可触达什么。见[首次运行](/zh/user/first-run)。

## 会话（Session）{#session}

与一个智能体的一段对话，跨轮保持。何时开新的，是流程最大的选择，因为它决定智能体记得什么：Ralph 循环每轮开新会话，所以智能体每次都从任务开始。见[多会话并行](/user/conversations)。

## 追踪（Trace）{#trace}

整次运行画成一条时间线，用 Perfetto 打开。见[追踪](/user/tracing)。

## 轮（Turn）{#turn}

与模型的一次交流：被告知什么、用工具做什么、回答什么。一轮进行中你输入的行进入那一轮。见[与运行中的一轮对话](/user/steering)。

## 编织者（Weaver）{#weaver}

写流程的人。[编织者指南](/zh/weaver/)为他们而写；用户指南从不要求 Python。

## 工作区（Workspace）{#workspace}

`hmz` 运行所在的目录。记住的东西、可离开再回来的运行、`/epics` 列出的运行，都属于同一个工作区。见[设置](/user/settings)。

---

想写自己的流程？[流程 API](/reference/flows) 里有全部 Python。
