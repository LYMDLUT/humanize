---
pageClass: hmz-ref
---

<script setup>
import '../../.vitepress/theme/components/ref-cli/ref.css'
import RefAtlas from '../../.vitepress/theme/components/ref-index/RefAtlas.vue'
</script>

# 参考

humanize 按实际构建的完整技术规范：每个命令、选项、语法、按键、界面、设置、协议消息、文件、环境变量、Python 符号、默认值、上限与错误。每页陈述行为；任务与讲解在[用户指南](/zh/user/)、[编织者指南](/zh/weaver/)与[特性](/zh/features/)。

<RefAtlas />

本章节的详细页面正在翻译中，暂以英文提供：[CLI](/reference/cli)、[TUI](/reference/tui)、[Web](/reference/web)、[Daemon](/reference/daemon)、[SDK](/reference/sdk)、[流程](/reference/flows)、[智能体](/reference/agents)、[机器](/reference/machines)、[提供商](/reference/providers)、[远程执行](/reference/remote-execution)、[追踪](/reference/tracing)、[文件](/reference/files)、[环境变量](/reference/environment)、[设置](/reference/settings)。

## 记法约定 {#conventions}

| 记法 | 含义 |
| --- | --- |
| `monospace` | 字面文本：按所示键入、打印或存储。 |
| `<name>` | 值的占位符，含义随文说明。 |
| `[x]` | 可选。 |
| `x…`、`x [x …]` | 一个或多个。 |
| `a \| b` | 备选其一。 |
| `{a, b}` | 全部组合，如同 shell 花括号展开。 |
| EBNF 块 | `=` 定义、`,` 连接、`\|` 交替、`[ ]` 可选、`{ }` 零或多个、`( )` 分组、`" "` 终结符、`? ?` 散文。 |
| `~/.hmz` | 设了且非空时的 `$HUMANIZE_HOME`。 |
| `hmz: …` | 按原样打印的文本；开头的 `hmz: ` 或 `hmz exec: error: ` 是消息的一部分。 |
| 类型 | 源码中书写的 Python 注解（`str \| None`、`tuple[str, ...]`）。 |
| *as local* | 以本机自己的登录运行的 CLI，无 humanize 账号。 |
| workspace（工作区） | 一次运行或一个界面的启动目录（解析后的路径）。 |
| run（运行） | 一个流程的一次执行；记为一个 epic。 |
| role（角色） | 流程为智能体、环境或局外人声明的名字。 |
| frontend（前端） | 接在一个 [host](/reference/daemon) 上的界面或程序。 |

行为与 `specs/` 下规范不一致时，以这些页面描述的代码行为为准。

