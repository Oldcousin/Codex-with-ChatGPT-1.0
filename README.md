# Codex-with-ChatGPT-1.0

Codex Skill，用于让 ChatGPT 网页版分析项目、制定方案并审查结果，由 Codex 负责修改文件、执行命令和测试。这个仓库包含 1.0 的 Skill 和安装说明。运行连接器仍需要 [XiaoDuoYa/codex-with-chatgpt](https://github.com/XiaoDuoYa/codex-with-chatgpt) 提供的 C2C Bridge 和 CLI。

## 1.0 更新了什么

- **停机满 30 分钟后的连接检查。** 从 Codex 关闭到再次启动满 30 分钟时，在重启后的第一个相关协作请求中先做只读检查。仅启动 Codex 不执行 Skill。时间不明时也可检查，结果会标明时长未知。
- **健康连接继续使用。** 检查通过后，在内置浏览器通过 ChatGPT 核验工作区身份和实际文件读取。只有确认故障且请求允许恢复时，才修复受影响的环节。
- **替换旧连接前先验证新地址。** 新地址不可用时保留旧连接和聊天关联。HTTP 530 会先复查。需要永久删除旧连接时，先解释影响并等待用户明确确认，之后才重建、配对和验证。
- **仅检查模式严格只读。** 用现有的安全文件测试读取。没有适用文件时报告未验证文件读取。重连期间不自动安装更新或改动沙箱设置。
- **验证后才保存关联。** 旧聊天无法读取时，可以先在同一项目的新聊天验证；工作区和文件读取都通过后才保存替代关联。

## One-paste install · 一段话安装

不熟悉 Git、Node.js 或终端也可以安装。把下面整个代码块复制给 Codex。

```text
请帮我完整安装并配置 Codex-with-ChatGPT-1.0。请自行执行下面的步骤，需要我操作时再叫我。

1. 环境自检。检查 Git、Node.js、Corepack 和 cloudflared。已有环境能运行上游锁定的 pnpm 11 就复用；需要安装 Node.js 时使用 24 LTS。缺少系统工具时，macOS 用 Homebrew，Windows 用 winget；缺少 Corepack 时用 npm install --global corepack 补齐。
2. 下载两个仓库。把上游运行时 https://github.com/XiaoDuoYa/codex-with-chatgpt.git 克隆到 ~/codex-with-chatgpt。首次安装时切换到 0.1.3 对应的提交 535a07dbb7aee887924c53767d985c09d405319d；已有兼容运行时就复用。把 1.0 Skill https://github.com/Oldcousin/Codex-with-ChatGPT-1.0.git 克隆到 ~/codex-with-chatgpt-1.0；已有该仓库的干净检出时，从远端 main 快进更新。保留已有本地改动。
3. 构建上游运行时。需要新建或补齐构建时，在上游目录执行 corepack pnpm install 和 corepack pnpm build，再运行 node bin/c2c.js --help 检查 CLI 可用。
4. 安装 Codex-with-ChatGPT-1.0。若 CODEX_HOME 非空，使用它；否则使用 ~/.codex（Windows 默认 %USERPROFILE%\.codex）。覆盖已有新版文件前先备份。从 1.0 仓库把 skill/SKILL.md 和 skill/agents/openai.yaml 复制到 <codex-home>/skills/codex-with-chatgpt-1-0/，保留 agents/ 子目录。在安装后的 SKILL.md 中，找到 “The codex-with-chatgpt checkout lives at:” 一行，把 <upstream-checkout> 替换为本机上游运行时的绝对路径。确认新版文件齐全后，如有旧版 <codex-home>/skills/codex-with-chatgpt/，将其移到 skills 目录外备份。
5. 配置当前工作区。首次配置时按安装后的 SKILL.md 中的 first-time setup 流程执行，确认连接选项后运行 c2c setup，在 Codex 内置浏览器打开 ChatGPT 并完成配对。已有连接时先做只读检查，健康就复用；确认故障并获得恢复授权后，按 1.0 流程验证新地址。若必须删除旧连接，先说明影响，再单独征得我的明确确认。全程只用 Codex 内置浏览器。
6. 只有遇到必须由我完成的操作时才停下来，例如登录 ChatGPT 或 Cloudflare、输入验证码、完成两步验证、选择连接方式或确认删除。每次只告诉我一个要做的动作。
7. 完成后给我一份 ✓ 清单，确认 Skill 已安装、连接对应的是当前工作区，并让 ChatGPT 实际读取当前工作区里的一个安全文件，告诉我读到的是哪个文件。遇到问题先自行排查；如果文件读取未通过，请用简单的话说明卡在哪一步。
```

## 手动安装与配置

1. 准备 Git、Node.js、Corepack 和 `cloudflared`。参考上游 [中文安装说明](https://github.com/XiaoDuoYa/codex-with-chatgpt/blob/main/README.zh-CN.md)。本版本基于上游 `0.1.3`，对应 [提交 `535a07d`](https://github.com/XiaoDuoYa/codex-with-chatgpt/tree/535a07dbb7aee887924c53767d985c09d405319d)。
2. 若未安装上游运行时，克隆上游仓库并切换到上述提交，从该目录执行 `corepack pnpm install` 和 `corepack pnpm build`。使用能运行该仓库指定 pnpm 版本的 Node.js。已有运行时可先通过 `node <上游目录>/bin/c2c.js --help` 核对。
3. 按上方第 4 步备份和安装 [skill 目录](skill/)内的两个文件，目标目录为 `<codex-home>/skills/codex-with-chatgpt-1-0/`。填写上游运行时的绝对路径，再按下面的步骤连接当前项目。

## 在新项目里使用

在 Codex 桌面打开新项目的根目录，将安装、配对和任务指令发到这个项目的 Codex 聊天。

1. **安装或升级。** 第一次安装，或本机仍显示旧版名称时，复制上方「一段话安装」的完整代码块。它包含当前项目的首次配对。如果本机已经能从 `$` 菜单选中 **Codex-with-ChatGPT-1.0**，可以直接配对。手写调用使用 `$codex-with-chatgpt-1-0`，因为技能注册名只允许小写字母、数字和连字符。
2. **配对当前工作区。** 已安装但尚未连接的新项目，发送下面这条指令。如果 Codex 提示创建对应的 ChatGPT 项目，在内置浏览器建好并选择「仅限项目记忆」，然后告诉 Codex「好了」。等它确认项目名称和文件读取结果，再开始工作。

   ```text
   使用 $codex-with-chatgpt-1-0 为当前工作区完成首次配置。复用本机安装，
   在 Codex 内置浏览器建立这个项目的独立连接。请让 ChatGPT 确认当前工作区名称，
   实际读取一个安全文件，并在两项都通过后保存聊天关联。
   ```

3. **开始协作。** 在这个项目的 Codex 聊天中发送具体任务。

   ```text
   使用 $codex-with-chatgpt-1-0 完成【填写任务和验收标准】。
   先由 ChatGPT 读取项目并制定方案，再由 Codex 修改文件、执行操作和运行测试。
   最后让 ChatGPT 审查实际改动与测试结果。发现问题就继续修复、测试和复审。
   ```

已连接的项目可直接派任务。重启后只想验证状态时发「重启完成，检查连接」。希望失败时进入恢复流程，则发「重启完成，检查并在失败时重连」。

## 工作原理与故障排查

Codex 在 ChatGPT 中提交简短任务说明。ChatGPT 按需从工作区读取上下文、拟定方案和核对执行结果。C2C Bridge 为 ChatGPT 提供只读工作区访问，代码修改、Git 操作及本地测试由 Codex 执行。项目连接使用授权和一次性配对。详细边界见上游 [协议说明](https://github.com/XiaoDuoYa/codex-with-chatgpt/blob/main/docs/protocol.md)及[安全模型](https://github.com/XiaoDuoYa/codex-with-chatgpt/blob/main/docs/security.md)。

- **重启后文件读取失败**：发「检查并在失败时重连」，先检查本机服务和公开地址，再判断是否需要更新 ChatGPT 端。浏览器控制报错应单独排查。
- **临时地址经常变化**：如果已有托管在 Cloudflare 的域名，可参考上游说明配置固定地址。
- **更新上游**：把运行时升级作为单独的维护任务，保留 1.0 的重启检查、删除审核和显示元数据。直接用上游 Skill 覆盖会丢失这些定制。

## 验证和许可

1.0 已通过 Skill 格式校验、显示名发现和重连审核规则复核。曾对 12 个固定情境逐项推演，包括 29:59 与 30:00、结束时间未知、新地址不健康和拒绝删除。离线推演用于核对指令，真实停机满 30 分钟后的故障恢复仍需在实际环境验证。

本项目是 [XiaoDuoYa/codex-with-chatgpt](https://github.com/XiaoDuoYa/codex-with-chatgpt) 的 Skill 定制版本，沿用 [MIT License](LICENSE) 并保留上游版权声明。
