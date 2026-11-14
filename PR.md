# PR 材料（本地草稿，不提交到市场仓库）

## 标题

Add codebuddy-cli-model 1.2.0 — CodeBuddy CLI direct-connect (Api 4)

## 正文

## 新增插件：codebuddy-cli-model 1.2.0

| 项 | 内容 |
| --- | --- |
| 插件名 / ID / 作者 | CodeBuddy CLI 直连 / `codebuddy-cli-model` / Grace-alan510 |
| 版本与兼容范围 | 1.2.0，鲸鲸 **4.1.2 或以上**（Api 4） |
| 用途 | 在 `chat.generate` 阶段调用本机 CodeBuddy CLI 生成回复，替代本轮默认模型；调用失败自动回退原模型 |
| 使用方法 | 装 ZIP → 启用并确认 `system.full` → 卡片「设置」填 `{"apiKey":"你的Key"}` → 保存 → 发普通消息 |
| 安装包 | `packages/codebuddy-cli-model-1.2.0.zip` |
| SHA-256 | `fcaf89d30971dff462916aa9aba67972ba481e1e58dff2b9817e4d7a58603315` |
| 源码 | 本 PR `plugins/codebuddy-cli-model/`；固定 commit 见 `provenance/codebuddy-cli-model-1.2.0.json` |
| 权限 | `Hooks = ["chat.prompt", "chat.generate"]`；`Permissions = ["system.full"]`；`ExecutionTimeoutMs = 120000` |

### 它做什么

- 不需要再跑 OpenAI 兼容桥接，不用改 provider / settings；
- 模型固定（默认 `deepseek-v4.1-flash`），不会随 CLI 默认档位漂移；
- 换模型在微信里发「模型」→ 回编号即可，选择写入鲸鲸官方插件数据目录，仅含模型 ID；
- 失败时每个会话提示一次并回退到原模型，不把 stderr / Key / 路径刷到微信。

### 权限与隐私

- `chat.prompt` 只把 `system` 存进按会话隔离的 state；`chat.generate` 只把「本轮用户消息」
  （可选最近 N 条历史，默认 0）通过 argv（`shell:false`）交给本机 CLI。
- CLI 以 `-p` 无头模式运行，`--tools ""`、`--no-session-persistence`。
- 不写日志、不落盘聊天内容、不主动发消息、不修改宿主配置。
- 唯一写盘：插件数据目录 `model-choice.json`（仅模型 ID）。
- 源码与安装包不含 API Key、账号、主机名、绝对路径或真实聊天数据；Key 只由使用者在本机设置里填写。
- **行为提示**：启用后提醒确认等规则话术也会被 CLI 接管；可在设置 `skipPatterns`
  填「提醒,取消提醒,提醒列表」跳过。README 与卡片描述均已注明。

### 试装记录（鲸鲸 4.1.2 / Windows 11）

| 场景 | 预期 | 结果 |
| --- | --- | --- |
| 正常输入（中文「只回复两个字：通过」） | 由本机 CodeBuddy CLI 返回文本 | 返回「通过」，12.1 s，宿主无回退告警 |
| 空输入 | 交回宿主原流程，不调用 CLI | 通过（离线契约测试） |
| 只发「模型」 | 返回编号菜单，不需要 Key | 通过 |
| 配置框为 `{}` / 示例文字 | 微信里给出可复制的 JSON 模板，回退原模型 | 通过 |
| 停用插件 | 立即回到原模型 | 通过（宿主真实 worker E2E） |
| 卸载 | 卡片消失 | 通过 |

离线契约测试 52/52；宿主真实 `JavaScriptPlugins` 端到端通过；市场 `validate.py` exit 0。

### 检查清单

- [x] ZIP 根目录只有 `plugin.json` + `plugin.js`
- [x] `Api: 4`、`MinAppVersion: 4.1.2`、`Permissions: ["system.full"]`
- [x] `Hooks` 均为 Api 4 允许值，无重复
- [x] 安装包未覆盖旧版本；旧版本不存在（首次投稿）
- [x] 提供 `provenance/codebuddy-cli-model-1.2.0.json`（发布者数字 ID + 固定 commit）
- [x] 未手改 `index*.json`
- [x] 已在 4.1.2 正式客户端试装
