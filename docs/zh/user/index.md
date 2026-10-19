# 用户指南

面向运行流程的人：你给编码智能体一个任务、一个工作的循环和一个预算，humanize 驱动它们直到工作完成或预算花完。这里没有任何一页要求你写 Python；写流程是[编织者指南](/zh/weaver/)的事。

::: info 本指南如何组织
- **从这里开始**与**教程**是课程：按顺序跟做，每条命令都写出，每步检查过后才进下一步。
- **其余每一页**针对一件事：它的用途与所需、一段短到可直接粘贴的*试一试*、原理、每个部分都有讲解的完整示例、如何确认成功、失败怎么办、以及下一步去哪。
- **查阅**页与[参考](/reference/)，是在你知道要找什么的时候用的。
:::

## 从这里开始

<div class="u7-path">

1. **[安装](/zh/user/installation)**
   humanize，外加一个已登录的编码智能体 CLI。约五分钟。
2. **[首次运行](/zh/user/first-run)**
   选一个流程，给它一个智能体和预算，看它修好一个 bug，然后停下。
3. **[安全](/zh/user/security)**
   在流程接触你在乎的工作前要检查什么。

</div>

## 教程

然后带着一件完整的工作走完全程。每个教程都是一条路线，每一步都有真实输出：

| 教程 | 你会学到 | 耗时 |
| --- | --- | --- |
| [攻克基准](/user/tutorials/take-home) | 两个智能体轮流、任务文件、核对结果以防作弊 | 你一小时，机器数小时 |
| [移植项目](/user/tutorials/port-a-project) | 一个执行者加一个评审者、一个自行结束的运行 | 约一小时 |
| [造一个编码智能体](/user/tutorials/build-an-agent) | 三个流程接力：想法、计划、构建 | 一个下午 |

## 其余章节

其余页面正在翻译中，暂时链接到英文原版：

- **运行中**：[与运行中的一轮对话](/user/steering)、[旁问 (/btw)](/user/btw)、[多会话并行](/user/conversations)、[监视运行 (monitor)](/user/monitor)、[暂离 (/afk)](/user/afk)、[停止](/user/stopping)、[放着继续跑](/user/leaving)、[浏览器中 (hmz web)](/user/web)、[macOS 应用](/user/desktop)、[任务板](/user/board)
- **提示符下**：[补全](/user/completion)、[历史](/user/history)、[设置 (/settings)](/user/settings)
- **智能体与账号**：[账号](/user/settings#accounts)、[投入度](/user/efforts)、[降级回退](/user/settings#fallback)、[成本与速率](/user/tally)、[权限](/user/permissions)、[技能](/user/skills)、[提问](/user/questions)
- **工作落点**：[容器](/user/containers)、[远程执行](/user/remote-execution)
- **运行之后**：[续跑](/user/resuming)、[导出](/user/export)、[备份](/user/backups)、[追踪](/user/tracing)
- **无人值守**：[无人值守运行](/user/unattended)、[CI 中的 humanize](/user/ci)
- **查阅**：[排障](/user/troubleshooting)、[报告](/user/reporting)、[术语表](/zh/user/concepts)、[CLI 参考](/reference/cli)、[TUI 参考](/reference/tui)、[Web 参考](/reference/web)

<style scoped>
.u7-path ol {
  list-style: none;
  counter-reset: step;
  padding-left: 0;
  display: grid;
  gap: 12px;
}
.u7-path li {
  counter-increment: step;
  position: relative;
  margin: 0;
  padding: 14px 16px 14px 60px;
  border: 1px solid var(--vp-c-divider);
  border-radius: 10px;
  background: var(--vp-c-bg-soft);
  color: var(--vp-c-text-2);
}
.u7-path li::before {
  content: counter(step);
  position: absolute;
  left: 16px;
  top: 14px;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  display: grid;
  place-items: center;
  font-weight: 600;
  color: var(--vp-c-white);
  background: var(--vp-c-brand-1);
}
.u7-path li strong {
  display: block;
  font-size: 1.05em;
}
</style>
