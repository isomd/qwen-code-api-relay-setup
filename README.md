# Qwen Code 怎么配置第三方 API？中转站接入与多模型切换教程

## 仓库附件

- [settings.example.json](settings.example.json)
- [封面提示词](cover-prompt.md)

模板只含占位符或环境变量引用，请按下文合并到自己的配置，保留现有设置。资料核对日期为 2026-10-21；本仓库没有声称已经完成付费线路实测。

---

Qwen Code 可以连接兼容接口，不必只使用工具默认的模型服务。想接 API 站时，可以把地址、模型和密钥变量写成一条完整的模型配置，再从 `/model` 中选择。

本文由 Orbis 项目方整理，以 [Orbis](https://orbiorbi.com) 为例。重点是把接入配置固定下来，避免切换模型时混用了另一套地址或凭据。

## 准备客户端与线路

运行 `qwen --version` 确认版本。尚未安装时，从 [官方快速开始](https://qwenlm.github.io/qwen-code-docs/zh/users/quickstart/) 安装；本文从已安装 CLI 开始。

在 [模型广场](https://orbiorbi.com/pricing) 选择提供 OpenAI Chat Completions 的模型线路，为它创建 Key，再复制调用 ID。本文使用：

```text
Base URL：https://api.orbiorbi.com/v1
模型 ID：deepseek-v4.1-flash
密钥变量：ORBIS_API_KEY
```

核对时该模型有提供 `openai` 协议的公开线路。所选线路的快速接入页若给出不同说明，以页面为准。

## 把模型写进用户级 settings.json

通常使用 `~/.qwen/settings.json`，Windows 对应 `%USERPROFILE%\.qwen\settings.json`。已有文件先备份，合并 `modelProviders` 中的条目，不要覆盖其他模型或项目设置。

```json
{
  "modelProviders": {
    "openai": [
      {
        "id": "deepseek-v4.1-flash",
        "name": "DeepSeek V4.1 Flash · Orbis",
        "envKey": "ORBIS_API_KEY",
        "baseUrl": "https://api.orbiorbi.com/v1"
      }
    ]
  }
}
```

`id` 是发给接口的模型 ID，`name` 是界面显示名称。`envKey` 写环境变量名，不能把真实 Key 填到这个字段。

这里使用的 `openai` 表示兼容协议，并不表示请求会发给 OpenAI 官网；实际目的地址由 `baseUrl` 决定。

当前官方文档要求每个 `modelProviders` 键对应一个模型数组。网上若看到把它包成 `{ "protocol": ..., "models": [...] }` 的示例，不要直接照搬到当前格式。自定义 Provider ID 还需要 `providerProtocol` 映射；本文使用内置键 `openai`，不需要额外映射。

## 提供 Key，进入模型选择器

Windows PowerShell：

```powershell
$env:ORBIS_API_KEY = "替换为这条线路的完整 API Key"
qwen
```

macOS / Linux：

```bash
export ORBIS_API_KEY="替换为这条线路的完整 API Key"
qwen
```

进入交互界面后执行 `/model`，选择“DeepSeek V4.1 Flash · Orbis”。如果出现认证选择，使用第三方或自定义提供商路径，不要把 Orbis Key 填进另一家服务的账号登录流程。

先发“只回复 OK”，然后到 Orbis 调用日志核对这次请求。只在文件里写入模型条目，不代表当前会话已经选中它。

## 多模型配置，按线路拆开凭据

以后需要添加模型，可以在相应协议的数组里增加条目。每一项都写完整的 `id`、`baseUrl` 和 `envKey`，尤其是使用不同线路时。

例如，第二条线路可以引用 `ORBIS_REVIEW_API_KEY`，不要默认沿用第一把 Key。Orbis 的 Key 绑定线路，当前线路是否包含另一个模型，要回到接入页确认。

Qwen Code 官方文档把这种已声明模型配置与临时运行参数区分开来。频繁切换时，固定的模型目录更容易检查；如果同时使用启动参数、临时环境变量和旧模型快照，应先确认 `/model` 当前选择的是哪一项。

## 用它检查一份真实配置说明

接通后，可以在演示仓库中给出一个具体任务：

> 阅读 README 和项目实际使用的配置文件，整理一张“文档字段—代码读取位置”的表。只列确实找到的字段，找不到依据的单独列出，先不要修改任何文件。

这样既能检查文件读取，也能看它是否把文档描述当成了代码事实。确认结果后，再选一条错误说明，让它修改 README，检查 `git diff`。

如果准备比较两个模型，让它们分别在新会话中做同一任务，保留输入、输出和账单。一次完成情况不能代表长期准确率，也不能只凭回答风格判断实际模型来源。

## 配置不生效，从这几处查

| 现象 | 检查项 |
| --- | --- |
| `/model` 没有新模型 | 文件位置、JSON 语法、数组结构和 Provider 键 |
| 选中后缺少凭据 | `envKey` 是否与当前进程中的变量名一致 |
| 自定义 Provider 整组消失 | 是否缺少 `providerProtocol`，修改后是否重启 |
| 401 或模型不存在 | Key 所在线路、调用 ID、账户余额及有效期 |
| 404 | 接口协议和 Base URL，是否误用了完整请求地址 |

先用最小配置验证，再按实际需要增加推理、图片或其他参数。客户端增加一个能力字段，不代表 Orbis 这条线路已经支持它。

## 参考资料

- [Qwen Code 官方模型提供商配置](https://qwenlm.github.io/qwen-code-docs/zh/users/configuration/model-providers/)
- [Qwen Code 官方认证配置](https://qwenlm.github.io/qwen-code-docs/zh/users/configuration/auth/)
- [Orbis 快速开始](https://orbiorbi.com/docs/quickstart/)
- [Orbis 线路选择](https://orbiorbi.com/docs/choose-route/)

资料核对日期：2026-10-21。本文为配置方法与验证步骤，本轮未实际调用付费模型，不将配置示例称为已跑通记录。
