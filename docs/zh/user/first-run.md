<script setup>
import Term from '../../.vitepress/theme/components/user-prompt/Term.vue'
import TermScreen from '../../.vitepress/theme/components/user-running/TermScreen.vue'

const agent = 'agent · claude/claude-haiku-4-5-20251001:low'
const keys = '/ commands · shift+enter newline · ← monitor · ctrl+c stop'
const running = [
  {
    label: '第 1 轮',
    lines: [
      '[dim]─ reading agent ─[/][n]1[/]',
      '',
      '[dim]❯[/] Fix the bug in calc.py.',
      '',
      '[dim]● agent is working[/]',
      '',
      "[g]●[/] I'll read the file to identify the bug.",
      '',
      '[g]●[/] Found the bug: the `add` function is subtracting instead of adding. Let me fix it.',
      '',
      '[g]●[/] Done. The `add` function was returning `a - b` instead of `a + b`—now it correctly adds the two numbers.',
      '',
      '[dim]✻ Worked for 7s · agent[/][n]2[/]',
      '',
      { t: '[dim]● agent is working · conversation 2 of 2[/][n]3[/]', hl: true },
      '',
      { r: `[m]${agent} · ● 2[/][n]4[/]` },
      { r: '[m]input 70 · output 1.1k · cache_read 146.5k · cache_write 22.2k[/][n]5[/]' },
      { r: '[m]$0.05 · 73 out/s[/]' },
      { rule: true },
      { prompt: '' },
      { rule: true },
      { l: '[c]·|·[/] agent… [m](3s · ctrl+c twice to stop)[/][n]6[/]', keys },
    ],
    caption:
      '一次真实运行，Claude Haiku 以投入度 <code>low</code>。第一轮七秒修好 bug；第二轮刚开始，在全新会话里。',
  },
  {
    label: '已停止',
    lines: [
      '[dim]● agent is working · conversation 2 of 2[/]',
      '',
      '[g]●[/] Let me check the git log to see what was supposed to be in this calculator.',
      '',
      '[dim]— press ctrl+c again to stop the flow —[/]',
      '[dim]— stopping the flow —[/]',
      '[y]●[/] [dim]cutting the turn off: interrupted[/]',
      '[y]●[/] [dim]turn cut off: interrupted[/]',
      '',
      '[dim]✻ Worked for 7s · agent[/]',
      '',
      { r: `[m]${agent} · ○ 2[/]` },
      { r: '[m]input 130 · output 2.2k · cache_read 286.7k · cache_write 29.9k[/]' },
      { r: '[m]$0.08 · 114 out/s[/]' },
      { rule: true },
      { prompt: '' },
      { rule: true },
      {
        l: '[c]◉[/] ralph_loop[m] · ~/tmp/humanize-demo[/][n]7[/]',
        keys: '/ commands · shift+enter newline · ← monitor · ctrl+c exit',
        hl: true,
      },
    ],
    caption:
      '两次 <kbd>ctrl+c</kbd>。进行中的一轮被切断，状态行回到流程待机。',
  },
]
</script>

# 首次运行

你在一个临时仓库里配置 `ralph_loop`，对着一个小 bug 启动它，看它工作，修好后停下。结束时该目录记住了配置，下次运行只需一行。

::: info 概览
- **你将**选一个流程、给它的角色配智能体和运行预算、以一个任务启动、旁观、停止。
- **何时使用** 刚装好 humanize，或想看流程在提示符下如何配置。
- **你需要** humanize 和一个已登录的编码智能体 CLI（[安装](/zh/user/installation)）、git、约十分钟。运行本身花几美分。
:::

::: warning 用临时仓库
流程的智能体以跳过审批的方式运行：不经询问就编辑文件、运行命令。在可以丢弃的地方开始，并在把流程指向你在乎的工作前阅读[安全](/zh/user/security)。
:::

<HmzCast name="first-run" alt="The flow menu: a project flow is chosen, its one agent is set to another model and effort, a budget of 20 minutes is set, and the menu is saved. The task is then typed at the prompt." />

<p class="u7-caption">同样的步骤，以替身 CLI 在项目自己的流程 <code>@local/twice</code> 上录制。</p>

## 试一试

在一个可以丢弃的 git 仓库里：

```text
hmz
/flow                        选 ralph_loop，设好智能体与预算，然后保存
❯ Fix the bug in calc.py.    任务；流程随之启动
ctrl+c ctrl+c                bug 修好后停止
```

下面逐步展开。

## 原理

三个词概括你要配置的东西：

- **流程**（flow）是循环：哪个智能体被问什么、按什么顺序、何时结束。此处用的 [`ralph_loop`](https://humanfia.ai/flows/ralph-loop) 一轮又一轮给同一个智能体同一个任务，每轮都在全新会话里，直到预算花完或你停它。
- **角色**（role）是流程声明的槽位，如 `agent`、`builder`、`reviewer`。你用**智能体**（agent）填每个角色：一个 CLI、它运行的账号、一个模型、一个投入度。
- **预算**（budget）是一个 epic 可花费的上限：时间、美元或输出 token。除 `chat` 外每个流程都要，因为多数流程是不会自行结束的循环。

三样都在 `/flow` 菜单里配好、保存，然后输入任务。在你发出第一行之前，什么都不会运行。

## 1. 造一个待修的东西

`calc.py` 把加法写成了减法：

```sh
mkdir -p ~/tmp/humanize-demo && cd ~/tmp/humanize-demo && git init -q
printf 'def add(a, b):\n    return a - b\n' > calc.py
git add -A && git commit -qm "a calculator with a bug in it"
git tag start
```

标签记下起点，之后无论智能体是否提交，你都能看到它改过的一切。

## 2. 打开界面

```sh
hmz
```

首次运行时，humanize 会问是否向开发者报告出错情况。问题写明报告带什么；随意作答，`/settings` 以后可改（[报告](/user/reporting)）。

界面打开在 `chat` 上：一个智能体，一次一轮地回答你。要自行运转的工作，你选**流程**。

## 3. 选一个流程 {#_3-choose-a-flow}

输入 `/flow` 并按 <kbd>enter</kbd>。它开在**已安装**（Installed）页：此处可运行的每个流程，humanize 内置的排最前。按 <kbd>/</kbd>，输入 `ralph`，按 <kbd>enter</kbd> 回到列表，再在 `ralph_loop` 上按 <kbd>enter</kbd>：

<Term title="hmz · /flow">

<pre>  <span class="m">hmz › /flow ›</span> <span class="p b">Installed</span>
  <span class="m">Flows ready to run here: pick one to set it up and run it, or install more</span>
  <span class="m">from a flowverse.</span>

  <span class="p">╭────────────────────────────────────────────────────────────────────────╮</span>
  <span class="p">│</span>  ralph                                                                 <span class="p">│</span> <span class="n">1</span>
  <span class="p">╰────────────────────────────────────────────────────────────────────────╯</span>
  <span class="p">╭────────────────────────────────────────────────────────────────────────╮</span>
   <span class="p">built in</span> <span class="n">2</span>
  <span class="sel"> <b>ralph_loop</b>                                                     built in </span>
     <span class="m">The task again and again, a fresh session every round.</span>

   <b>stateful_ralph</b>                                                 <span class="m">built in</span>
     <span class="m">The task again and again, in one session that remembers.</span>
  <span class="p">╰────────────────────────────────────────────────────────────────────────╯</span>

    Install more…   Update   Uninstall   Copy here <span class="n">3</span>   Search…       Save

  <b>enter</b> set up   <b>/</b> search   <b>tab</b> actions   <b>esc</b> clear search</pre>

</Term>

1. **搜索框**保留名字含你输入字母（按序）的流程。<kbd>esc</kbd> 清空，再按退出菜单。
2. **`built in`** 是这些流程的来源：随 humanize 发布。你从 [flowverse](/zh/user/concepts#flowverse) 安装的列在其名下，本项目自己的列在 `local` 下。
3. **Copy here** 把光标处的流程复制进本项目，供阅读或修改。现在用不到；[安全](/zh/user/security#listing-a-flow-runs-its-python)说明何时需要。

::: details 比这些更多的流程
**Install more…** 进入 flowverse：humanize 可安装的流程索引，`official` 排最前。`hmz` 每次打开都在后台拉取。打开一个，再打开一个流程，安装；此后它列在这里，行尾是版本。见 [Flowverse](/weaver/flowverses#managing-flowverses)。
:::

## 4. 给角色配智能体

流程开在**角色**（roles）页：每个它驱动的智能体一行，下面是 `budget`、`profile flow`、`profile environment`，按钮在最后，**Save** 收尾。`ralph_loop` 只有一个角色 `agent`。humanize 用它找到的第一个 CLI 填好了它，所以那行已有名字。

<Term title="hmz · /flow › ralph_loop">

<pre>  <span class="m">hmz › /flow › Installed ›</span> <span class="p b">ralph_loop</span>                    <span class="y">● unsaved changes</span>
  <span class="m">Configure each role: an agent (CLI, account, model and effort) or an</span>
  <span class="m">environment; then what the flow takes and what an epic may spend.</span>

  <span class="p">╭────────────────────────────────────────────────────────────────────────╮</span>
  <span class="sel"> <b>agent</b>                                     claude/claude-opus-5-5:high ▸ </span> <span class="n">1</span>
     <span class="m">agent</span>
   ────────────────────────────────────────────────────────────────────────
   <b>budget</b>                                                           <span class="m">none ▸</span> <span class="n">2</span>
     <span class="m">what an epic may spend: none set; an epic needs one</span>
   ────────────────────────────────────────────────────────────────────────
   <b>profile flow</b>                                                    <span class="m">○ off ▾</span> <span class="n">3</span>
     <span class="m">traced only</span>
   ────────────────────────────────────────────────────────────────────────
   <b>profile environment</b>                                             <span class="m">○ off ▾</span>
     <span class="m">traced only</span>
  <span class="p">╰────────────────────────────────────────────────────────────────────────╯</span>

     <span class="btn"> Update </span>  <span class="btn"> Uninstall </span>  <span class="btn"> Copy here </span>                             <span class="btn"> Save </span> <span class="n">4</span>

  <b>enter</b> open   <b>tab</b> actions   <b>esc</b> back</pre>

</Term>

1. **`agent`**，流程唯一的角色，填它的智能体写作 `cli/model:effort`。
2. **`budget`**：还没设。第 5 步设。
3. **`profile flow`** 与 **`profile environment`** 保持关闭：那是要把流程自身的 Python 或运行启动的程序放进[追踪](/user/tracing#profiling-a-run)时用的。
4. **Save**：按它之前，改动一概不保存。`● unsaved changes` 说明有待保存的改动。

在 `agent` 上按 <kbd>enter</kbd>。智能体是四行：

<Term title="hmz · /flow › ralph_loop › agent">

<pre>  <span class="m">hmz › /flow › Installed › ralph_loop ›</span> <span class="p b">Set up agent</span>
  <span class="m">Configure this agent: select its CLI, account, model, and reasoning
  effort.</span>

  <span class="p">╭────────────────────────────────────────────────────────────────────────╮</span>
   <b>cli</b>                                                            <span class="a">claude ▸</span> <span class="n">1</span>
     <span class="m">coding agent CLI to use</span>
   ────────────────────────────────────────────────────────────────────────
   <b>account</b>                                                      <span class="a">as local ▸</span> <span class="n">2</span>
     <span class="m">account to run as</span>
   ────────────────────────────────────────────────────────────────────────
   <b>model</b>                                       <span class="a">claude-haiku-4-5-20251001 ▸</span> <span class="n">3</span>
     <span class="m">model to use</span>
   ────────────────────────────────────────────────────────────────────────
  <span class="sel"> <b>effort</b>                                                            low ▾ </span> <span class="n">4</span>
  <span class="sel">   reasoning effort                              </span><span class="p">╭─ effort ─────────────╮</span>
                                                   <span class="p">│</span> ultracode            <span class="p">│</span>
                                                   <span class="p">│</span> max                  <span class="p">│</span>
                                                   <span class="p">│</span> xhigh                <span class="p">│</span>
                                                   <span class="p">│</span> high                 <span class="p">│</span>
                                                   <span class="p">│</span> medium               <span class="p">│</span>
                                                   <span class="p">│</span><span class="sel"> low ✔                </span><span class="p">│</span>
                                                   <span class="p">╰──────────────────────╯</span>
  <span class="p">╰────────────────────────────────────────────────────────────────────────╯</span>

                                                                     <span class="m"> Save </span> <span class="n">5</span>

  <b>enter</b> choose   <b>esc</b> close</pre>

</Term>

1. **`cli`**：执行轮次的编码智能体 CLI。只列出能填此角色的 CLI。换一个会清空账号和模型，因为两者属于 CLI。
2. **`account`**：`as local` 即以你登录的 CLI 运行。列表下方的**Add an account**可建别的；见[账号](/user/settings#accounts)。
3. **`model`**：该 CLI 报告可跑的模型之一。列表旁有各模型的投入度，下方**Check again**重新询问。
4. **`effort`**：模型想多用力，从模型自己的列表选。`▾` 表示 <kbd>enter</kbd> 或点击会在行下展开全部投入度，最用力的在前，生效的带 `✔`；<kbd>↑</kbd> <kbd>↓</kbd> 加 <kbd>enter</kbd>，或点击选择，<kbd>esc</kbd> 不选。见[投入度](/user/efforts)。
5. **Save** 保存智能体。没有任何一行改动之前按不了。

标 `▸` 的行会打开列表：<kbd>enter</kbd> 或 <kbd>→</kbd>，然后选；每个列表都开在当前生效项上。首次运行，像这里一样选个小模型、低投入度。然后 <kbd>tab</kbd> 到 **Save**，<kbd>enter</kbd> 保存。顶部那行是来路 `hmz › /flow › Installed › ralph_loop › Set up agent`：<kbd>esc</kbd> 或点击 `ralph_loop` 回去。

自带设置的流程会在角色前先问它。`ralph_loop` 没有。

## 5. 设预算

在 `budget` 上按 <kbd>enter</kbd>：

<Term title="hmz · /flow › ralph_loop › budget">

<pre>  <span class="m">hmz › /flow › Installed › ralph_loop ›</span> <span class="p b">Set budget for ralph_loop</span> <span class="y">● unsaved changes</span>
  <span class="m">An epic stops at whichever limit it reaches first; at least one limit is
  required. Leave empty or 0 for no limit.</span>

  <span class="p">╭────────────────────────────────────────────────────────────────────────╮</span>
   <b>duration</b>                                                            <span class="a">20m</span> <span class="n">1</span>
     <span class="m">maximum epic duration: 1h30m, 90s, PT2H; empty for no limit</span>
   ────────────────────────────────────────────────────────────────────────
  <span class="sel"> <b>cost</b>                                                                0.5 </span> <span class="n">2</span>
  <span class="sel">   maximum cost in US dollars, 0 for no limit                            </span>
   ────────────────────────────────────────────────────────────────────────
   <b>output_tokens</b>                                                         <span class="a">0</span>
     <span class="m">maximum output tokens, 0 for no limit</span>
   ────────────────────────────────────────────────────────────────────────
   <b>graceful</b>                                                         <span class="g">● on ▾</span> <span class="n">3</span>
     <span class="m">finish the current turn when a limit is reached</span>
  <span class="p">╰────────────────────────────────────────────────────────────────────────╯</span>

                                                                      <span class="btn"> Set </span> <span class="n">4</span>

  <b>enter</b> change   <b>tab</b> actions   <b>esc</b> back</pre>

</Term>

1. **`duration`**：运行最多多久。<kbd>enter</kbd>，输入 `20m`，<kbd>enter</kbd>。`1h30m`、`90s`、`PT2H` 都认。
2. **`cost`**：最多花多少美元。<kbd>enter</kbd>，输入 `0.5`，<kbd>enter</kbd>：第一个按键替换掉原有的 `0.0`。`output_tokens` 同理。
3. **`graceful`**：开时到达上限让进行中的一轮做完；关时立即切断。<kbd>enter</kbd> 或点击在其下展开两项。
4. **Set** 保存全部四项：<kbd>tab</kbd> 过去按 <kbd>enter</kbd>。`budget` 行随后读作 `set`，其下是 `what an epic may spend: stops at 20m, $0.50`。

运行先到哪个上限就停。若你的账号按 token 计费，`duration` 之外再设 `cost`。见[每次运行都有预算](/features/allowances)。

## 6. 保存

<kbd>tab</kbd> 到角色下方的 **Save**，按 <kbd>enter</kbd>。状态行读作 `◉ ralph_loop`，humanize 提示 `enter a task to start the flow`。

保存之前一切不生效。<kbd>esc</kbd> 后退，带着改动离开 `/flow` 会弹框问 `Save?`，有 **Save** 与 **Discard**。没预算的流程拒绝保存：`this flow requires a budget: set the budget first`。

## 7. 告诉它做什么

```text
❯ Fix the bug in calc.py.
```

按 <kbd>enter</kbd>，流程启动。智能体的话边说边流进记录：

<TermScreen title="hmz · ralph_loop" :frames="running" />

按编号看要点：

1. **`─ reading agent ─`** 说明这是谁的记录：运行落在它打开的第一个会话的智能体上，你说的在最上方，并跟着它进入每个新会话。
2. **`✻ Worked for 7s · agent`** 收尾一轮：第一轮完成，`calc.py` 已经修好。
3. **`conversation 2 of 2`**：Ralph 循环每轮开全新会话，智能体每次都初见任务。预算花完前不停。
4. **智能体行**：它的角色、它跑什么、一轮进行中是 `●`（轮间是 `○`）。数字是它持有的会话数。
5. **运行已花费**：各类 token、美元、模型写字的速度。见[成本与速率](/user/tally)。
6. **状态行**：谁在干活、干了多久、怎么停。右端是当前可用按键；`ctrl+c stop` 表示 <kbd>ctrl+c</kbd> 将开始一次停止。
7. **已停止**：回到 `◉ ralph_loop` 和 `ctrl+c exit`，因为没有东西在跑。

运行期间：

| 目的 | 操作 | 详见 |
| --- | --- | --- |
| 对智能体说点什么 | 输入一行按 <kbd>enter</kbd>；进入进行中的一轮 | [与运行中的一轮对话](/user/steering) |
| 看谁在干活、谁交给了谁 | 不输入时按 <kbd>←</kbd> | [监视运行](/user/monitor) |
| 停止流程 | 两次 <kbd>ctrl+c</kbd>，或 `/stop` | [停止](/user/stopping) |
| 离开但让它继续 | 关终端，或 `/exit` | [放着继续跑](/user/leaving) |

Ralph 循环一直转到预算花完。`calc.py` 修好后，按两次 <kbd>ctrl+c</kbd> 停它。

### 确认成功

从第 1 步的标签看它做了什么：

```sh
git diff start
```

```diff
 def add(a, b):
-    return a - b
+    return a + b
```

`hmz` 里的 `/epics` 列出你刚做的运行，标记为 `stopped`。

## 下次：一行

humanize 记住此目录的配置，下次在这里 `hmz` 直接开在 `ralph_loop`，智能体与预算照旧。输入任务、<kbd>enter</kbd>。

这个目录记住的是：

- 你最后保存的流程，`hmz` 开在它上；
- 此处配置过的每个流程：各角色的智能体（CLI、账号、模型、投入度）、流程自身的设置、预算。

不开菜单换流程：行首加 `$` 接流程名：

```text
❯ $ralph_loop Fix the bug in calc.py.
```

| 流程在 | 写作 |
| --- | --- |
| 内置，或从 `official` 安装 | `$ralph_loop` |
| 本项目 `.hmz/flows/` | `$@local/twice` |
| 你的 `~/.hmz/flows/` | `$@user/twice` |
| 从你添加的 flowverse 安装 | `$@<flowverse>/<flow>` |

已配置过的流程立即启动。没配置过的开在角色菜单上，保存即启动。`$ralph_loop` 后面没有字时只是选流程。

[`/settings` 的工作区页](/user/settings#workspace)展示并可以忘记此目录记住的一切。

## 排障

### `this flow requires a budget: set the budget first`

保存被拒，因为预算行仍是 `none set; an epic needs one`。回第 5 步至少设一个上限。

### cost 行读作 `0.00.5`

你输入的加进了行内原有值。在该行按 <kbd>enter</kbd>，删掉再输。

### `hmz: cannot choose a flow while one is running`

流程运行中输入了 `$name` 或 `/flow <name>`。先停（[停止](/user/stopping)）。单独的 `/flow` 打开的是运行中流程的智能体页，在那里保存的就是下次运行的配置。

### 智能体行选了不想要的 CLI

humanize 用找到的第一个 CLI 填角色。在该角色上按 <kbd>enter</kbd>，再在 `cli` 上选别的，然后重新选它的模型。

### [Flows](https://humanfia.ai/flows/) 页里的流程没列出来

一开始只有 humanize 内置的流程。其余从 flowverse 安装：见[第 3 步](#_3-choose-a-flow)下的说明。

## 下一步

- [安全](/zh/user/security)：把流程指向真正的工作前要检查什么。
- 教程各带一件真实工作走完全程：[攻克基准](/user/tutorials/take-home)、[移植项目](/user/tutorials/port-a-project)、[造一个编码智能体](/user/tutorials/build-an-agent)。
- [Flows](https://humanfia.ai/flows/) 列出 humanize 与官方 flowverse 提供的每个流程。
- [无人值守运行](/user/unattended)：同一运行写成一行 `hmz exec`。

<style scoped>
.u7-caption {
  margin-top: -8px;
  font-size: 0.85em;
  color: var(--vp-c-text-2);
}
kbd {
  display: inline-block;
  padding: 0 6px;
  border: 1px solid var(--vp-c-divider);
  border-bottom-width: 2px;
  border-radius: 5px;
  background: var(--vp-c-bg-soft);
  font-family: var(--vp-font-family-mono);
  font-size: 0.85em;
  line-height: 1.6;
  white-space: nowrap;
}
</style>

