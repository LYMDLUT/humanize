<script setup>
import { withBase } from 'vitepress'
import WeaverLoom from '../../.vitepress/theme/components/weaver-index/WeaverLoom.vue'
</script>

# 编织者指南

**编织者**（weaver）写流程：那些说明驱动哪些智能体、各被问什么、按什么顺序、何时结束的 Python。你已经运行过流程；这里是你写一个的地方。

<WeaverLoom />

::: info 开始之前
- 一次流程运行的经历。没有的话从[用户指南](/zh/user/)开始。
- 能读懂的 Python：`async def`、`await`、类，偶尔一个 [pydantic](https://docs.pydantic.dev/) 模型。
- 一个不介意被智能体编辑的 git 仓库。
:::

这里每一页都是流程能做的一件事的指南。每页从你要构建什么开始，讲清思路，再走完一个可复制的完整流程：每个标记行（`# ①`）做什么、以什么命令运行、真实运行打印了什么、如何确认成功、如何不花一个 token 测试，然后是变体与陷阱。

本指南其余页面正在翻译中，暂以英文提供：[编写第一个流程](/weaver/writing-a-flow)、[在测试下构建](/weaver/tutorials/build-under-test)、[循环](/weaver/loops)、[流程自己的参数](/weaver/flow-settings)、[多轮并行](/weaver/async-flows)、[流程调用流程](/weaver/calling-flows)、[分支会话](/weaver/branching)、[工作树、副本与草稿](/weaver/worktrees)、[目标](/weaver/goals)、[形状化答案](/weaver/shapes)、[钩子](/weaver/hooks)、[智能体问流程](/weaver/tools)、[人作为智能体](/weaver/human-agent)、[测试流程](/weaver/testing-flows)、[Flowverse](/weaver/flowverses)。

## 从这里开始

四页，按顺序。第一页让你五分钟跑起自己的流程；其余让它生长。

<div class="hmz-paths">
  <a :href="withBase('/weaver/writing-a-flow')">
    <strong>1 · 你的第一个流程</strong>
    <span>一个目录、一个装饰器、一个角色。写它，跑它。</span>
  </a>
  <a :href="withBase('/weaver/tutorials/build-under-test')">
    <strong>2 · 在测试下构建</strong>
    <span>一个教程：一个智能体写、pytest 判、第二个智能体评审。</span>
  </a>
  <a :href="withBase('/weaver/loops')">
    <strong>3 · 循环</strong>
    <span>下一轮记得什么，什么结束循环。</span>
  </a>
  <a :href="withBase('/weaver/flow-settings')">
    <strong>4 · 流程自己的参数</strong>
    <span>一个变成 <code>-p</code> 与提示符下表单的 pydantic 模型。</span>
  </a>
</div>

## 让流程生长

| | |
| --- | --- |
| [多轮并行](/weaver/async-flows) | `gather`、`TaskGroup`、各配一个会话 |
| [流程调用流程](/weaver/calling-flows) | 站在别人写的流程上 |
| [分支会话](/weaver/branching) | 一段会话两条出路，只付一次钱 |
| [工作树、副本与草稿](/weaver/worktrees) | 一个智能体同时在多个目录工作 |

## 智能体可被要求什么

| | |
| --- | --- |
| [目标](/weaver/goals) | CLI 自己的目标：它决定何时完成 |
| [形状化答案](/weaver/shapes) | 一轮以 pydantic 模型而非散文作答 |
| [钩子](/weaver/hooks) | 你的代码，在会话的各个时刻运行 |
| [智能体问流程](/weaver/tools) | 轮中一问，由流程自己的代码作答 |
| [人作为智能体](/weaver/human-agent) | 你，被流程像任何智能体一样驱动 |

智能体可触达什么、随身带什么，也由流程声明：[权限](/user/permissions)与[技能](/user/skills)为你写好。

## 就绪之后

| | |
| --- | --- |
| [测试流程](/weaver/testing-flows) | 流程跑在脚本化智能体上：毫秒级，零花费 |
| [Flowverse](/weaver/flowverses) | 按发布出版流程，供人按名安装或按 ref 调用 |

## 下一步

可读的真实流程在 [Flows](https://humanfia.ai/flows/)。流程 API 的每个参数、拒绝与返回见[参考 › 流程](/reference/flows)。

