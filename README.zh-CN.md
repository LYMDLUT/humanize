<div align="center">

**简体中文** | [English](./README.md)

</div>

---

# humanize _(hmz)_

![humanize](https://socialify.git.ci/humanfia/humanize/image?description=1&font=Raleway&forks=1&issues=1&logo=https%3A%2F%2Fraw.githubusercontent.com%2Fhumanfia%2Fhumanize%2Frefs%2Fheads%2Fmain%2Fdocs%2Fpublic%2Flogo.svg&name=1&owner=1&pattern=Circuit+Board&pulls=1&stargazers=1&theme=Auto)

为 token 极限榨取而生的智能体流程系统。

文档：<https://docs.humanfia.ai/humanize/>（[中文版](https://docs.humanfia.ai/humanize/zh/)）

## 安装

需要 Python 3.12 或更新版本、[uv](https://docs.astral.sh/uv/)，以及一个已登录的编码智能体
CLI，如 Claude Code 或 Codex。

```sh
uv tool install hmz
```

三个后端各需要额外的包：`[dsh]` 对应 DeepSeek Harness，`[kimi]` 对应 Kimi
Code，`[litellm]` 对应直接调用的模型（或任何经 litellm 运行、以其可达账号行事的
CLI），`[all]` 三者全装。

```sh
uv tool install 'hmz[all]'
```

`hmz update` 升到最新发布，并保留安装时的 extras。

也可以不安装，用 uv 缓存的一份直接跑，`@latest` 表示最新发布：

```sh
uvx hmz@latest
uvx --from 'hmz[all]' hmz web
```

### 桌面应用

macOS 上，从[最新发布](https://github.com/humanfia/humanize/releases/latest)下载
`humanize-<version>-macos-universal.dmg`，把 humanize 拖进应用程序：它把 `hmz web`
装进一个独立窗口，运行的是上面安装的 `hmz`。首次打开的方法见[指南](https://docs.humanfia.ai/humanize/user/desktop)。

## 用法

在某个仓库里打开终端界面：

```sh
hmz
```

或在浏览器里查看本机全部 epic：

```sh
hmz web
```

或从 shell 运行一个流程：

```sh
hmz exec -f chat -a assistant=claude/claude-opus-5:high "What does this repository do?"
```

智能体以跳过审批的方式运行，请先在临时仓库里开始。
[文档](https://docs.humanfia.ai/humanize/zh/)的首页是一份快速上手。

## 贡献

欢迎拉取请求：
[CONTRIBUTING.md](https://github.com/humanfia/humanize/blob/main/CONTRIBUTING.md) 写明如何提出。报告 bug
或请求功能请走 [issue 表单](https://github.com/humanfia/humanize/issues/new/choose)；漏洞请按
[SECURITY.md](https://github.com/humanfia/humanize/blob/main/SECURITY.md)
私下报告。所有参与者遵守
[行为准则](https://github.com/humanfia/.github/blob/main/CODE_OF_CONDUCT.md)。

## 许可

Apache-2.0 &copy; Humanfia
