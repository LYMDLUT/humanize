<script setup>
import Term from '../../.vitepress/theme/components/user-prompt/Term.vue'
</script>

# 安全

在把流程指向你在乎的工作前要检查什么，以及智能体工作时是什么在约束它。

::: danger 智能体的任何动作都不会请你确认
每个流程的智能体都以跳过审批的方式运行，无论什么流程、无论你是否在看。它们自行编辑文件、运行命令、提交代码。约束智能体的是流程声明的权限，见下。
:::

::: info 概览
- **你将**做一次可撤销的运行、读流程各角色可触达什么、知道每轮计费到哪个账号。
- **何时使用** 即将在真实仓库上运行流程、安装流程，或保存账号。
- **你需要** humanize 已安装，以及流程工作仓库的 git。
:::

## 试一试

流程碰你在乎的东西之前的清单：

<div class="u7-checklist">

- **工作可以撤销。** 在 git 仓库里运行并打标签（`git tag start`）：`git diff start` 随后显示流程改过的一切（无论是否提交），`git reset --hard start` 一条命令复原。
- **你读过各角色可触达什么。** 它在流程代码里，不在任何菜单里。
- **你信任安装的每个流程。** 列出流程就会运行它的代码。
- **你知道每个智能体以哪个账号运行。** 就是智能体的 `account` 行，每轮都计费到它。

</div>

## 原理

流程无人值守地运行智能体，所以防护必须在运行前定好，而不是运行中。三件事决定它，没有一件是提示词：

| 什么 | 由谁决定 | 在哪检查 |
| --- | --- | --- |
| 每个智能体可读、写、触达什么 | 流程，逐角色声明 | 流程代码，或 [Flows](https://humanfia.ai/flows/) 上它的页面 |
| 谁的代码运行 | 你，通过安装流程 | `/flow` 的已安装页 |
| 一轮用谁的凭据 | 你，在智能体的 `account` 行 | `/flow`，以及 `/settings` 的账号页 |

Git 让其余一切敢于尝试：运行前的标签把智能体做过的一切变成一个可读的 diff 和一条复原命令。

## 示例：运行前读流程的授权

你想运行 [`aot`](https://humanfia.ai/flows/aot)（从描述写出一个流程）到一个你在乎的仓库。先让运行可撤销：

```sh
cd ~/src/app
git status --short             # 没有你自己未提交的东西可丢
git tag start
```

然后读它各角色可触达什么。在 `hmz` 里打开 `/flow`。若 `aot` 未安装，按 **Install more…**，打开 `official` 安装，回到 **Installed**。光标放到 `aot` 上，<kbd>tab</kbd> 到 **Copy here**，<kbd>enter</kbd>：

<Term title="hmz · /flow">

<pre>  <span class="m">hmz › /flow ›</span> <span class="p b">Installed</span>
  <span class="m">…</span>
  <span class="p">╭────────────────────────────────────────────────────────────────────────╮</span>
   <span class="p">official</span>
   <b>aot</b>                                                               <span class="a">0.1.0</span>
     <span class="m">Writes a flow from a description: drafted, loaded, smoke-run, revie…</span>

   <span class="p">local</span> <span class="n">1</span>
   <b>@local/aot</b>
     <span class="m">Writes a flow from a description: drafted, loaded, smoke-run, revie…</span>
  <span class="p">╰────────────────────────────────────────────────────────────────────────╯</span>
   <span class="m">copied to .hmz/flows/aot -- you can edit it, and aot now points to it</span> <span class="n">2</span>

    Install more…   Update   Uninstall   <span class="sel"> Copy here </span>   Search…       Save

  <b>enter</b> copy   <b>←/→</b> move   <b>tab</b> list   <b>esc</b> back</pre>

</Term>

现在流程是项目里的一个目录，它的授权一搜即得：

```sh
grep -rn "_permission" .hmz/flows/aot
```

```console
.hmz/flows/aot/__init__.py:51:    _permission = Permission(local=PermissionKind.READ, online=PermissionKind.NONE)
```

按编号看要点：

1. **`local`** 出现在 `official` 下：这份拷贝现在是项目自己的流程，已安装 `aot` 的任何更新都不改变你运行的。
2. **`aot now points to it`**：此目录里的 `$aot` 与 `-f aot` 运行你的拷贝，就是你刚读的代码。

搜索结果里：唯一角色 critic 声明 `local=READ`、`online=NONE`。它读草稿、从不写、触不到网。没声明的角色（此处是其余全部）按下表默认运行：只写本目录，不碰你别的任何东西。

### 确认成功

- `git tag --list start` 打出 `start`。
- 运行后 `git diff start` 显示流程改过的一切，仓库之外没有任何默认授权角色写过的东西。
- `/flow` 里各角色的 `account` 行读作你想要的账号：`as local` 或你的某个账号。

## 流程允许智能体触达什么

流程为它驱动的每个角色声明权限。你在提示符下选不了它，所以运行前读流程。四个范围：

| 范围 | 覆盖什么 | 未声明的角色得到 |
| --- | --- | --- |
| `local` | 会话的工作目录 | `ALL`：可读可写 |
| `user` | 智能体运行账号的家目录其余部分 | `READ` |
| `system` | 机器上其余一切 | `READ` |
| `online` | CLI 自带的网络搜索与抓取 | `ALL` |

实际效果：

- **每个范围都在智能体与它运行的每条命令上强制执行**，Linux 用 Landlock、macOS 用 Seatbelt。默认授权的智能体只写工作目录和它必须的东西：CLI 自己的设置、登录与会话、专属临时目录、共享的 `/tmp` 与 `/var/tmp`、设备、Linux 上的 `/proc`。macOS 上它的登录含登录钥匙串，Claude Code 就存在那里。
- **Claude Code、Codex、Grok Build、Qwen Code 与 pi 绝不写你的设置。** 每个进程在为它单独创建、停止即删的家目录里启动：CLI 运行中会改写的文件是副本（Claude Code 的 `.claude.json`、备份与 `settings.json`；Codex 与 Grok Build 的 `config.toml`；Qwen Code 的 `settings.json`；pi 的 `models.json` 与 `settings.json`），其余是指向你自己的链接，所以登录与会话是你自己的，流程的技能也在你的旁边。写进副本的内容（比如信任此工作区）随副本而去，humanize 加进去的也一样：Qwen Code 的投入度、钩子与工具服务器，网关账号给 Grok Build 建的模型项或给 pi 建的 provider。你的全局设置与项目自己的设置按 CLI 一贯的方式读取。
- **`user` 为 `ALL`** 允许智能体改你的用户账号能改的一切，约束它的是文件权限；以 root 运行时，机器其余部分仍是 `system`。
- **只读角色**（`local` 为 `READ` 或 `NONE`）同时在 CLI 的只读模式下运行。
- **`online` 为 `NONE`** 切断网络，智能体的模型与登录所在主机除外。CLI 的网络工具也一并关闭（在可告知的范围内）。
- **容器或 ssh 主机里的工作同样被围栏。** 智能体在那台机器上运行的每条命令都受那台机器自己的 Landlock 或 Seatbelt 约束，围绕它自己的工作目录与家目录。
- **无法持有的授权会被拒绝，而不是放宽。** 在没有 Landlock 的 Linux 内核上、humanize 已在另一沙箱内的 Mac 上、以及此类机器上的工作，角色只能以全范围 `ALL` 运行。
- **一些出口仍在。** Landlock 与 Seatbelt 都不管 Unix 套接字，所以智能体仍能连上别的程序监听的套接字，包括 docker 守护进程；macOS 上经 Mach 通信的系统服务同理。macOS 上 `online` 为 `NONE` 时，智能体运行的程序可在任意地址上服务，不止回环。而智能体能写的，之后的运行都能读。

[权限](/user/permissions)列出官方流程各自声明什么，以及各 CLI 如何执行它。

## 列出流程就是运行它的 Python {#listing-a-flow-runs-its-python}

流程是 Python 目录，humanize 运行它才知道它是什么。打开 `/flow`、或在提示符下以 `$` 指名，都会运行 humanize 列出的每个流程：内置的、你装的、本项目的、你自己的。

从 [flowverse](/weaver/flowverses) 安装流程因此是把这台机器托付给来源仓库，如同安装软件包：只装你敢当软件包装的。添加 flowverse 本身不运行任何东西：它是发布索引，不含代码。`official` 始终在：humanize 自己的，在 [humanfia/flowverse](https://github.com/humanfia/flowverse)，其维护者审阅列出的每个发布，降低但不消除风险。

已安装流程固定在发布指名的 commit，直到你更新它：`hmz` 每次打开都拉取 flowverse，只在有新发布时提示。想把你读过的版本固定下来：`/flow` 里光标放到它上，按 **Copy here**，如示例那样：它被复制进本项目自己的流程，此处该流程的名字运行拷贝。

## 你的凭据在哪

| 智能体以什么运行 | humanize 保存什么 |
| --- | --- |
| `as local`（默认） | 无。CLI 读它自己的登录，一直存在它存的地方。 |
| `/settings` 账号页建的登录 | 该 CLI 登录时写的文件，在 `~/.hmz/providers/<cli>/<name>/` 下 |
| `/settings` 账号页建的 key 或网关 | 你输入的内容：key 与网关 URL，在同目录的 `provider.json` 里 |

- 这些目录和文件只有你能读。
- `/settings` 的账号页说明账号设置哪些变量，绝不显示其值。你输入的密钥以圆点显示。
- 以账号运行的一轮会清掉其他账号会用到的键，所以 shell 配置里遗留的 key 顶替不了它。
- 删除 `~/.hmz` 删除全部账号。CLI 自己的登录原地不动。

账号凭据如何到达一轮，见 [Providers 参考](/reference/providers)。

## 其他值得知道的

- **报告。** 只有你首次同意过，humanize 才向开发者报告出错。报告带的不止崩溃；[报告](/user/reporting)列出送什么、绝不送什么。
- **TCP 上的远程目标**对能连到那个端口的任何人都是那台机器的一个 shell。给它真 token，或改用 `ssh://` 和 `docker://`。见[远程执行](/user/remote-execution)。
- **`/afk` 管提问，不管动作。** 它决定流程能否问你什么。它绝不会让智能体行动前先问你。
- **`dsh` 智能体始终全权访问**，无视流程声明。
- **项目的钩子只在项目所在的机器上运行。** Claude Code 的 `hooks`、`.mcp.json` 的 stdio server 等项目设置会在 CLI 运行处启动程序。若流程的智能体与其环境的工作不在同一台机器（harness `local` 而工作在别处，或 harness 运行时不在工作的机器上），项目还有这类设置时 humanize 拒绝打开会话。让智能体在那台机器上运行（harness `env`），或移除该设置。见[远程执行](/reference/remote-execution#a-projects-own-programs)。

## 排障

### 角色在运行开始前被拒

它的授权在此无法持有：无 Landlock 的内核、已在别的沙箱里的 Mac、或此类机器，只有全范围 `ALL` 的角色能跑。而 `cursor-agent` 的角色带 `online` 为 `NONE` 在任何机器上都被拒，因为 Cursor 的网络工具关不掉。在带 Landlock 的 Linux 或无外层沙箱的 macOS 上运行，或给角色换 CLI。见[权限](/user/permissions)。

### 流程改了我没料到的东西

用 `git diff start` 全看，用 `git reset --hard start` 全撤。智能体做的提交也一并撤销，因为标签更早。仓库之外的文件不在 git 里：默认授权的角色写不到那里，但授权了 `user` 或 `system` 为 `ALL` 的可以。

### 上周读过的流程行为变了

它更新了：更新会把新发布装在你读过的版本之上。在 `/flow` 的流程页选你读过的发布（**Switch to**），再 **Copy here** 固定为拷贝。

### 一轮计费到了错误的账号

智能体的 `account` 行决定它。打开 `/flow`，角色上 <kbd>enter</kbd>，检查 `account`。不存在的账号让该智能体的每轮都失败并指名它；绝不会悄悄回退到 `as local`。

## 报告漏洞

私下报告，绝不在公开 issue 里：[SECURITY.md](https://github.com/humanfia/humanize/blob/main/SECURITY.md) 写明方法、什么算漏洞、以及何时回复你。

## 下一步

- [权限](/user/permissions)：各官方流程授予什么，各 CLI 如何执行。
- [账号](/user/settings#accounts)：API key、网关与第二个登录，彼此隔离。
- [Flowverse](/weaver/flowverses)：安装流程意味着什么，以及发布你自己的。
- [容器](/user/containers)：把工作放到错误够不着的地方。

<style scoped>
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
.u7-checklist ul {
  list-style: none;
  padding-left: 0;
  border: 1px solid var(--vp-c-divider);
  border-radius: 8px;
  background: var(--vp-c-bg-soft);
  padding: 12px 16px;
}
.u7-checklist li {
  position: relative;
  padding-left: 1.9em;
  margin: 8px 0;
}
.u7-checklist li::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0.3em;
  width: 1.05em;
  height: 1.05em;
  border: 2px solid var(--vp-c-brand-1);
  border-radius: 4px;
}
</style>
