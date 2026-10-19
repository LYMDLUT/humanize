<script setup>
import Term from '../../.vitepress/theme/components/user-prompt/Term.vue'
</script>

# 安装

安装 humanize、登录一个编码智能体 CLI、确认 humanize 能看到它。几分钟即可，大部分时间花在 CLI 自己的登录上。

::: info 概览
- **你将得到** `PATH` 上的 `hmz` 命令，以及至少一个它能驱动的编码智能体。
- **何时使用** 在一台机器上首次部署 humanize，或为已有部署添加后端。
- **你需要** **Python 3.12 或更新版本**，以及多数编码智能体 CLI 需要的 `npm`。
:::

## 试一试

以 Claude Code 为例，全部过程是：

```sh
uv tool install hmz
npm i -g @anthropic-ai/claude-code && claude auth login
hmz --version
```

下面的步骤解释每行做什么、其余安装方式，以及每个后端的登录方法。

## 原理

humanize 不直接和模型对话。它像人一样驱动你已经在用的**编码智能体 CLI**（如 Claude Code 或 Codex）：启动它们、交给任务、读它们做了什么。humanize 能驱动的每个 CLI 是一个**后端**（backend）。

所以有两样东西要装，而且互相独立：

| 你安装 | 谁登录它 | 它给你什么 |
| --- | --- | --- |
| humanize，装一次 | 无：它不持有任何账号 | `hmz` 命令 |
| 一个或多个编码智能体 CLI | 你，用 CLI 自己的登录 | 流程运行的智能体 |

humanize 以**你登录的方式**运行每个 CLI。智能体菜单称其为 `as local`。什么都不复制：你的登录留在 CLI 存放它的地方。要以另一个账号运行 CLI，见[账号](/user/settings#accounts)。

## 1. 安装 humanize

::: code-group

```sh [uv tool]
uv tool install hmz
```

```sh [pipx]
pipx install hmz
```

```sh [pip]
pip install hmz
```

:::

每种方式都从 [PyPI](https://pypi.org/project/hmz/) 给你 `hmz` 命令。`uv tool` 和 `pipx` 把它装进独立环境，任何目录都能用；`pip` 装进当前激活的环境，所以只有那个环境存活时才有 `hmz`。

::: tip 在 Mac 上，应用会替你装好
[macOS 应用](/user/desktop)从 Dock 打开独立窗口运行 `hmz web`，并用 `uv tool` 自行安装 `hmz`（没有 uv 时会先下载）：打开它，这一步无需运行。它把 `hmz` 保持在应用自己的版本，所以装了应用就更新应用而不是 `hmz`。
:::

### 不装也能运行 {#uvx}

`uvx` 从 uv 创建并缓存的环境里运行 `hmz`，不在 `PATH` 上放任何东西。`hmz` 之后的参数都传给它：

```sh
uvx hmz                          # 终端界面，在此仓库中
uvx hmz web                      # 本机全部 epic，浏览器中
uvx hmz@latest                   # 最新版本，而非 uv 缓存的那个
uvx --from 'hmz[all]' hmz        # 带上下面说的 extras
```

每行都是一个完整的 `hmz` 命令，所以本页写 `hmz` 的地方键入 `uvx hmz` 即可。`hmz update` 在这里不适用：`uvx hmz@latest` 才是这种方式升级的途径。

### 三个后端需要额外的包 {#the-two-backends-that-are-extras}

除非你要用 DeepSeek Harness、Kimi Code 或 litellm，否则跳过。每个都需要 humanize 环境里的一个 Python 包：

| Extra | 用于 |
| --- | --- |
| `hmz[dsh]` | DeepSeek Harness（`dsh`）。无 CLI 可装。 |
| `hmz[kimi]` | Kimi Code（`kimi`），与其 CLI 并用。 |
| `hmz[litellm]` | litellm（`litellm`）：直接调用一个模型，无 CLI。还有它的网关：大多数 CLI 以 litellm 可达的任意账号运行（[哪些](/reference/providers#through-litellm)）。 |
| `hmz[all]` | 三者全要。 |

::: code-group

```sh [uv tool]
uv tool install 'hmz[all]'
```

```sh [pipx]
pipx install 'hmz[all]'
```

```sh [pip]
pip install 'hmz[all]'
```

:::

之后可以再补，无需重装 humanize。智能体菜单会列出缺 extra 的后端，行内附安装命令。

## 2. 登录一个编码智能体 {#signing-each-backend-in}

humanize 驱动由你自行安装并登录的编码智能体 CLI。一个就够。选你的：

::: code-group

```sh [Claude Code]
npm i -g @anthropic-ai/claude-code
claude auth login
```

```sh [Codex]
npm i -g @openai/codex
codex login                    # 无浏览器？codex login --device-auth
```

```sh [Cursor Agent]
curl https://cursor.com/install -fsS | bash
cursor-agent login
```

```sh [Antigravity]
# 安装 Google 的 Antigravity CLI 使 `agy` 出现在 PATH，然后：
agy                            # 它会让你登录 Google 账号
```

```sh [Grok Build]
npm i -g @xai-official/grok
grok login                     # 无浏览器？grok login --device-auth
```

```sh [Kimi Code]
npm i -g @moonshot-ai/kimi-code
kimi login                     # 加上上面的 [kimi] extra
```

```sh [Qwen Code]
npm i -g @qwen-code/qwen-code
qwen                           # 然后 /auth 选套餐或密钥，设好后 /quit
```

```sh [opencode]
npm i -g opencode-ai
opencode auth login
```

```sh [mimocode]
npm i -g @mimo-ai/cli
mimo auth login
```

```sh [MiniMax Code]
npm i -g @minimax-ai/code
mcode login                    # 或用 MiniMax API key：mcode provider set-minimax-key
```

```sh [pi]
npm i -g @earendil-works/pi-coding-agent
pi                             # 然后 /login，登录后 /exit
```

```sh [Oh My Pi]
curl -fsSL https://omp.sh/install | sh
omp                            # 然后 /login，登录后 /exit
```

```sh [DeepSeek Harness]
# 无 CLI：安装上面的 [dsh] extra，并给它一个 DeepSeek API key
export DEEPSEEK_API_KEY=sk-…
```

:::

已经登录的话，无事可做。

::: tip API key，或一个 CLI 的两个账号
在 `hmz` 内的 [`/settings` 的账号页](/user/settings#accounts)建一个账号。它可以持有登录、API key 或你自己的网关，一个流程能同时跑同一 CLI 的两个账号。
:::

## 3. 确认可用 {#check-what-you-have}

```sh
hmz --version
```

```console
hmz 0.1.0
```

然后在任意目录打开界面：

```sh
hmz
```

## 示例：`hmz` 首次打开

首次运行时，humanize 会问它找到的每个 CLI 能跑哪些模型，可能要一分钟。然后问一个它自己的问题。这是一台登录了 Claude Code、Codex 和 Qwen Code 的机器上的样子：

<Term title="hmz · the first launch">

<pre>  <span class="p">╭────────────────────────────────────────────────────────────────╮</span>
  <span class="p">│</span>                                                                <span class="p">│</span>
  <span class="p">│</span>  <span class="p b">Report errors to humanize?</span>                                    <span class="p">│</span> <span class="n">1</span>
  <span class="p">│</span>  <span class="m">Send error reports to help fix bugs. Sent: the error and</span>      <span class="p">│</span>
  <span class="p">│</span>  <span class="m">where in humanize it occurred; … Never sent: nothing you</span>      <span class="p">│</span>
  <span class="p">│</span>  <span class="m">typed: no task, prompt, or command; … You can change this</span>     <span class="p">│</span>
  <span class="p">│</span>  <span class="m">later in /settings.</span>                                           <span class="p">│</span>
  <span class="p">│</span>                                                                <span class="p">│</span>
  <span class="p">│</span>  <span class="sel"> Yes </span>  <span class="btn"> No </span>                                                   <span class="p">│</span>
  <span class="p">│</span>                                                                <span class="p">│</span>
  <span class="p">│</span>  <b>enter</b> yes   <b>esc</b> ask again next time                           <span class="p">│</span>
  <span class="p">│</span>                                                                <span class="p">│</span>
  <span class="p">╰────────────────────────────────────────────────────────────────╯</span>

                                  <span class="m">assistant · qwen/qwen3-coder-plus:high</span> <span class="n">2</span>
<span class="d">──────────────────────────────────────────────────────────────────────</span>
<span class="p">❯</span>
<span class="d">──────────────────────────────────────────────────────────────────────</span>
  <span class="a">◉</span> chat · ~/src/demo <span class="n">3</span>   <span class="d">/ commands · shift+enter newline · ctrl+c exit</span></pre>

</Term>

按编号看要点：

1. **报告询问**，覆盖屏幕的对话框。它完整说明报告带什么、绝不带什么。两个按钮就是答案：**Yes** 有焦点，<kbd>enter</kbd> 即同意，<kbd>→</kbd> 移到 **No**。怎么答都行：`/settings` 以后可改，<kbd>esc</kbd> 下次再问。见[报告](/user/reporting)。
2. **提示符上方那行**说明 humanize 将启动的智能体：`role · cli/model:effort`。此处角色是 `assistant`，CLI 是 humanize 找到的第一个能报出模型的。这里出现 CLI 和模型，就证明 humanize 能驱动它。
3. **状态行**说明选中的[流程](/zh/user/concepts#flow)（`chat`）与目录。右侧是当前可用的按键。

### 确认成功

- `hmz --version` 打印出版本。
- 提示符上方那行读作 `assistant · <cli>/<model>:<effort>`，而不止 `assistant`。
- `/flow` 后在角色上按 <kbd>enter</kbd>，其 `cli` 行列出你的 CLI。

用 `/exit` 离开。现在做你的[首次运行](/zh/user/first-run)。

## 特定后端的注意事项

::: details DeepSeek Harness：key、模型与平台
`[dsh]` extra 装在 Linux（x86-64 与 arm64）和 macOS（arm64）上。

用以下任一方式给它 key：

- 环境变量 `DEEPSEEK_API_KEY`，或项目 `.env` 文件。`DEEPSEEK_BASE_URL` 指向别的端点。
- dsh 自己 CLI 保存的 key（若你也有）：`dsh web`，然后 **Settings → Models**。
- [`/settings` 账号页](/user/settings#accounts)上的一个账号：`key` 填 DeepSeek key；或 `openai-gateway` / `anthropic-gateway` / `gemini-gateway` 填代理或第三方 key 及其 URL（分别对应 OpenAI、Anthropic、Gemini 的 API 形态）。网关按端点自己的投入度运行；可在那里搜索的智能体能抓取网页但不能搜索。

没有 key 时，humanize 列出 `dsh` 但不替你选它。它的模型是 `deepseek-v4-flash` 和 `deepseek-v4-pro`，投入度有 `max`、`high`、`low`、`off`：

```sh
hmz exec -f chat -a assistant=dsh/deepseek-v4-flash:high "say hello"
```

`chat` 与随 humanize 发布的六个循环见 [FlowBench 得分](https://humanfia.ai/flows/)。来自 flowverse 的流程（如 `flame_chasoid:parallel_git_pr`）要从 `/flow` 安装后才能运行；`hmz exec` 不会安装任何东西。

`dsh` 智能体无视流程声明，始终全权访问。见[安全](/zh/user/security)。
:::

::: details 任何说 Agent Client Protocol 的其他 CLI
在 [`/settings` 的账号页](/user/settings#accounts)添加，它就会出现在上述十三个旁边。见[多个后端，一个智能体](/features/backends)。
:::

## humanize 把东西放在哪

humanize 记住的一切都在 `~/.hmz/` 下：你的运行、每个目录的配置、你的账号、拉取的 flowverse 和安装的流程。设 `HUMANIZE_HOME` 可挪到别处。项目自己的流程和从项目导出的运行在该项目的 `.hmz/`。完整清单见 [CLI 参考](/reference/cli)。

旧版本用 `~/.humanize/` 和 `.humanize/`。你无需处理：humanize 首次查看时会各自改名成 `.hmz`。两个都在时，用 `.hmz`，旧的保持原样。设了 `HUMANIZE_HOME` 时 `~/.humanize` 原地不动，所以 `~/.humanize/flows` 里你自己的流程要自己移到 `~/.hmz/flows`。见[文件](/reference/files#moved-from-humanize)。

## 排障

### 提示符上方那行只显示 `assistant`

没有 CLI 应答：humanize 一个都没找到，或找到的未登录、说不出自己跑什么。此时输入任务会报 `hmz: no coding agent is installed`。

humanize 依次找 `PATH`、`~/.local/bin`、`/usr/local/bin`、`/opt/homebrew/bin`、`/usr/bin` 和 `/bin`。看 shell 能找到哪些：

```sh
command -v agy claude codex cursor-agent grok kimi mcode mimo omp opencode pi qwen
```

列出了却仍不提供的，多半是没登录：重跑[第 2 步](#signing-each-backend-in)的登录，再开 `hmz`。

### `hmz: command not found`

`pip` 装进了未激活的环境，或 `uv tool` / `pipx` 的目录不在 `PATH`。运行 `uv tool update-shell` 或 `pipx ensurepath`，再开新 shell。

### 我期望的模型不见了

CLI 跑什么是一次性询问并记住一周，之后在后台再问。想立刻问：打开 `/flow`，在角色上按 <kbd>enter</kbd>，再在 `model` 行按 <kbd>enter</kbd>，然后按列表下的**再检查一次**。

### `dsh`、`kimi` 或 `litellm` 带着安装命令出现

缺它对应的 extra。运行行内命令，或按[三个后端需要额外的包](#the-two-backends-that-are-extras)重装。

## 升级

```sh
hmz update
```

`hmz update` 向 PyPI 询问最新版本，让当初安装 `hmz` 的工具（`uv tool`、`pipx` 或环境里的 `pip`）来装，保留 extras。有新版本时 `hmz` 与 `hmz web` 都会提示。从 checkout 安装的用 `git pull` 更新。手动等价操作：

::: code-group

```sh [uv tool]
uv tool upgrade hmz
```

```sh [pipx]
pipx install --force hmz
```

```sh [pip]
pip install --upgrade hmz
```

:::

每种都升到最新。`uv tool upgrade` 保留 extras；`pipx` 和 `pip` 是重装，要重新写 extra：`pip install --upgrade 'hmz[all]'`。

用 [macOS 应用](/user/desktop)时改更新应用：下次无运行打开时，它会把 `uv tool` 的 `hmz` 拉回应用自己的版本，保留 extras。

每个版本的变更见 [GitHub Releases](https://github.com/humanfia/humanize/releases)。

## 卸载

先停掉留下的运行：进入其目录运行 `hmz`，然后 `/exit`。

::: code-group

```sh [uv tool]
uv tool uninstall hmz
```

```sh [pipx]
pipx uninstall hmz
```

```sh [pip]
pip uninstall hmz
```

:::

要连 humanize 记住的一切（含账号）一并清除，删掉 `~/.hmz` 或 `HUMANIZE_HOME` 指向的目录：

```sh
rm -rf ~/.hmz
```

编码智能体 CLI 及其登录保持原样。

## 下一步

- [首次运行](/zh/user/first-run)：选流程、给智能体和预算、看它干活。
- [安全](/zh/user/security)：流程接触你在乎的工作前要检查什么。
- [账号](/user/settings#accounts)：以 API key、网关或第二个登录运行 CLI。
- [多个后端，一个智能体](/features/backends)：各后端能与不能的事。

