# codebuddy-cli-model

鲸鲸（WhaleGirl）**4.1.2+** 的 Api 4 消息插件：在 `chat.generate` 阶段直接调用本机
**CodeBuddy CLI** 生成回复，替换本轮默认模型调用。

- 不需要额外的 OpenAI 兼容桥接进程；
- 不需要把鲸鲸的 provider 改成 `custom`，不用改 `settings.json`；
- 调用失败自动回退到鲸鲸原来的模型；
- 默认不发送聊天历史、不写日志、不主动发消息、不修改宿主配置。

## 安装

1. 打开鲸鲸左侧「**插件市场**」页，在**顶部第一行**勾选
   「**启用 JavaScript 插件（实验性，完整系统权限）**」。
   ⚠️ 这个开关不在左侧「设置」页。
2. 同一页点「从本地安装」，选择 `packages/codebuddy-cli-model-1.2.0.zip`。
3. 点「启用」并确认 `system.full` 授权。
4. 打开插件卡片「设置」，在唯一的配置框里填：

```json
{"apiKey":"在这里粘贴你的 CodeBuddy API Key"}
```

把示例文字换成自己的 Key，点「保存设置」。
（Key 只存在本机授权文件里，不在安装包和本仓库中。）

5. 发一条**普通**消息验证。⚠️ 发「模型」收到菜单**不能**证明配置成功——
   菜单命令不需要 Key、也不调用 CLI。

## 换模型

默认 `deepseek-v4.1-flash`。不用手敲完整 ID：

```
发「模型」        → 带编号的菜单
回一个数字「9」   → 立即切换，之后一直生效
发「模型 全部」  → 全部 22 个
发「当前模型」    → 查当前模型
发「恢复默认模型」→ 还原默认
```

选择写入鲸鲸官方插件数据目录的 `model-choice.json`，内容只有一个模型 ID，
不含 Key / 聊天内容 / 路径。卡片设置 `model` 优先级最高，填了它菜单就不生效。

## 构建与测试

零依赖，只需 Node.js：

```bash
cd plugins/codebuddy-cli-model
node build.js                 # 从 plugin.json 读版本，生成 ../../packages/<id>-<version>.zip
node tests/run.js             # 52/52 离线契约测试（mock CLI，无需 Key / 网络）
WHALE_ROOT=<鲸鲸目录> node tests/host-e2e.js   # 宿主真实执行器端到端
```

`build.js` 用固定 DOS 时间戳，同样源码 → 同样 ZIP。

## 清单

```json
{
  "Id": "codebuddy-cli-model",
  "Api": 4,
  "Kind": "message",
  "Hooks": ["chat.prompt", "chat.generate"],
  "Permissions": ["system.full"],
  "MinAppVersion": "4.1.2",
  "ExecutionTimeoutMs": 120000
}
```

## 安全边界

- Api 4 是**完整系统权限**，不是沙箱；只启用信任来源。
- 插件只把「本轮用户消息」（可选最近 N 条历史）通过 argv（`shell:false`）交给本机 CLI；
  CLI 以 `-p` 无头模式运行，禁用工具（`--tools ""`）、不持久化会话。
- 错误提示只含类别码，不回显 stderr、Key 或路径。
- 唯一写盘：插件数据目录的 `model-choice.json`（仅模型 ID）。
- 不含 API Key、账号、主机名、绝对路径或真实聊天数据。

## License

MIT，见 [LICENSE](LICENSE)。
