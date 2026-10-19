<script setup>
import ChangePath from '../../.vitepress/theme/components/contributing-index/ChangePath.vue'
</script>

# 贡献

欢迎提交拉取请求。本页把一份 humanize checkout 准备到可改的状态：装好、按 CI 的方式检查、并给出每处改动要遵守的规则。bug 与功能请求走 [issue 表单](https://github.com/humanfia/humanize/issues/new/choose)；大改动请先开 feature request。

拉取请求如何提出与评审见 [CONTRIBUTING.md](https://github.com/humanfia/humanize/blob/main/CONTRIBUTING.md)，谁来做决定、如何成为评审者或维护者，见组织的 [GOVERNANCE.md](https://github.com/humanfia/.github/blob/main/GOVERNANCE.md)。所有参与者遵守[行为准则](https://github.com/humanfia/.github/blob/main/CODE_OF_CONDUCT.md)。

<ChangePath />

::: tip 第一次来？
[你的第一个补丁](/contributing/tutorials/first-patch)把一处小改动从克隆带到拉取请求，每条命令都写出。
:::

::: info 开始之前
`git` 和 [`uv`](https://docs.astral.sh/uv/)。别的都不用：`uv sync` 连 Python 一起带来。
:::

## 环境准备

```sh
git clone https://github.com/humanfia/humanize.git
cd humanize
uv sync                    # ①
uv run pre-commit install  # ②
```

1. **`uv sync`** 按 `uv.lock` 钉死的版本在 `.venv` 建好项目环境，含 Python。
2. **`pre-commit install`** 让每次提交前跑 CI 用的那些钩子。

工具一律通过 `uv run` 而非 `uvx` 运行，用的才是 `uv.lock` 钉的版本。想验证环境，跑一个包的单元测试：

```sh
uv run pytest tests/unit/flows
```

## 检查

::: code-group

```sh [推送前]
uv run pre-commit run --all-files   # 格式、lint、类型：秒级
uv run pytest                       # 单元与集成：分钟级
```

```sh [写代码时]
uv run pytest tests/unit/<package>  # 快循环
```

```sh [有系统测试覆盖时]
uv run pytest tests/system/test_flows_ralph.py   # 真智能体、真 token
```

:::

**推送前**的两条必须通过，它们就是 CI 跑的。`uv run pytest` 跑 `tests/unit` 与 `tests/integration`，不含 `tests/system`。

| | 你的机器 | CI |
| --- | --- | --- |
| pre-commit 钩子 | ✓ | ✓ `static`，每次推送与拉取请求 |
| `tests/unit/` | ✓ | ✓ 每个包单独跑（有改动时），Linux 与 macOS |
| `tests/integration/` | ✓ | ✓ 每个主题单独跑（所跑内容有改动时），Linux 与 macOS |
| `tests/system/` | ✓ 手动、一次一个文件（改动需要时） | 从不 |
| [macOS 应用](#the-macos-app) | ✓ Mac 上（改 `desktop/` 时） | ✓ `desktop`（macOS，`desktop/` 有改动时） |

[CI](/contributing/ci) 有每个 job 及如何读一次运行。

## macOS 应用 {#the-macos-app}

`desktop/` 是 [macOS 应用](/user/desktop)：`hmz web` 页面上的一个窗口，Rust 写就、用 [Tauri](https://tauri.app/)，遵循[其规范](https://github.com/humanfia/humanize/blob/main/specs/features/desktop.md)。改它需要 Mac、[Rust](https://rustup.rs/)、Node 与 pnpm。从 `main` 构建时版本号为 `0.0.0`，且不安装任何东西：`pnpm dev` 对着你 `PATH` 上的 `hmz` 打开，`HUMANIZE_HMZ` 可指定别的（比如本 checkout 的）：

```sh
cd desktop
pnpm install
HUMANIZE_HMZ="$PWD/../.venv/bin/hmz" pnpm dev
```

想试发布版如何安装 `hmz`：以 PyPI 上 `hmz web` 支持 `--attached` 的版本号构建：`pnpm tauri build --bundles app --config '{"version":"<it>"}'`。

推送前跑 CI 的 `desktop` job 跑的：

```sh
cd desktop/src-tauri
cargo fmt --check
cargo clippy --all-targets -- -D warnings
cargo test
cd .. && pnpm tauri build --debug --bundles app,dmg
```

发布附带的两种架构磁盘镜像：`pnpm tauri build --target universal-apple-darwin --bundles dmg`，先 `rustup target add aarch64-apple-darwin x86_64-apple-darwin`。布局由 Finder 完成，首次构建会请求让终端控制 Finder。背景是 `src-tauri/dmg/background.svg`，改动后 `pnpm dmg-background` 重制打包器取的那两张 tiff。

## 测试放哪

测试恰好放三个目录之一，按它拼装 humanize 的多少。目录决定它何时运行：无需写标记。

| | 装什么 | 规则 |
| --- | --- | --- |
| `tests/unit/<package>/` <Badge type="tip" text="CI" /> | `src/hmz/` 的单个包，放在以它命名的目录：`cli`、`coganchor`、`daemon`、`flows`、`runtime`、`sdk`、`tui` | 只用包的公开名，绝不碰 `_private`。其余 `hmz` 包一律 mock。无子进程、套接字、网络。独立 job 须 3 分钟内完成 |
| `tests/integration/test_<topic>_*.py` <Badge type="tip" text="CI" /> | 多个包接起来，对着本仓库写的替身：替身 CLI、假 app server、回环套接字、mock LLM 服务 | 扁平，无子目录。topic 为 `core`、`agents`、`tui`、`daemon`、`anchor`、`web` 之一，各一个 job，10 分钟内完成 |
| `tests/system/` <Badge type="warning" text="you" /> | 真智能体做真任务：装好并登录的 CLI，在 `tests/system/sample/` 上工作，按需 docker 或 ssh。`tests/system/web/` 里是 `hmz web` 页面在真实浏览器里被 Playwright 绘制与点击 | 每个测试 5 至 30 分钟与真 token；`web/` 里分钟级、零 token。从不进 CI |

`ruff` 的 `SLF001` 与 `pyright` 的 `reportPrivateUsage` 会挂掉碰私有名的测试。测试 import 别的包处把它 mock 掉，让测试只对该包的承诺负责，而不是对其他包恰好的行为。

**只在改动碰到某条路径时跑对应系统测试**（比如某后端的驱动、内置流程或机器），以及发布前。绝不在每次改动跑全部：合计数小时、花真钱。按名字跑，如上面**有系统测试覆盖时**；文件名 `test_<area>_<what>.py`，area 为 `flows`、`harness`、`machines`、`tui`。所需不在本机时（某 CLI、其登录、docker、ssh）测试会跳过并说明原因。

::: danger 绝不把登录拷进测试的家目录
Codex 的 ChatGPT 登录与 Claude Code 的订阅登录会自我续期，续期会吊销其他拷贝持有的 token：以 `~/.codex/auth.json` 或 `~/.claude/.credentials.json` 的拷贝登录的容器或临时家目录，会把你本机登出。需要它们的测试让 CLI 以 local 运行，或把 CLI 自己的家原位挂载（`--volume=DIR:DIR`）。pytest 临时目录下任何文件含这类刷新 token 时运行即失败（`tests/conftest.py` 的 `_copies_no_sign_in`）。
:::

## 代码要遵守什么

- **`pyright` 严格模式**，覆盖 `src` 与 `tests`。`# type: ignore` 关闭：抑制必须指名 pyright 规则。
- **`ruff` 全规则**，`pyproject.toml` 里写了理由的除外。
- **Google 风格 docstring**，处处类型注解。
- **优先用维护良好的流行库**，而非自己造。
- **每层只 import 分层表允许的。** 见[架构](/contributing/architecture)。
- **`specs/` 是契约。** 改代码去匹配 SPEC。除非被要求，不改 SPEC。
- **改流程运行方式**的，还要保住 [humanfia/flowverse](https://github.com/humanfia/flowverse) 列出的流程能跑。
- **改行为的同一拉取请求里更新文档。** 规则见[文档工作指南](/contributing/docs)。

## 提交

[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)：`fix(agents): …`、`docs(contributing): …`，破坏性改动在冒号前加 `!`。改动与测试进同一个提交。

## 下一步

| 目的 | 阅读 |
| --- | --- |
| 端到端完成一处小改动 | [你的第一个补丁](/contributing/tutorials/first-patch) |
| 增改一页文档 | [给文档加一页](/contributing/tutorials/a-page-of-docs) |
| 找改动落哪，或加后端、加命令 | [架构](/contributing/architecture) |
| 读 CI 对改动跑了什么、为什么 | [CI](/contributing/ci) |
| 按规则检查一页文档 | [文档工作指南](/contributing/docs) |
