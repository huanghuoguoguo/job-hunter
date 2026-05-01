# job-hunter

BOSS 直聘岗位搜索、筛选、重复检测与辅助投递 skill。它依托 OpenCLI 的 `opencli boss` 和 `opencli browser` 命令工作，适合用 Claude Code（`cc`）打开后直接让 agent 配置环境并开始使用。

## 依赖

- OpenCLI（我们的 fork，包含 BOSS `--jobType` 全职/实习过滤等增强）：https://github.com/huanghuoguoguo/OpenCLI
  - 上游仓库：https://github.com/jackwener/opencli
- Node.js 21+
- Chrome 或 Chromium
- OpenCLI Browser Bridge 扩展
- 已登录 BOSS 直聘的浏览器会话

> **注意**：上游 npm 包 `@jackwener/opencli` 不带本 skill 依赖的 `--jobType` 参数。如果需要按"全职/实习"在 API 层精准过滤，请按下文从 fork 源码安装。

推荐安装 OpenCLI（仅做基础用法时可选用上游 npm 包）：

```bash
npm install -g @jackwener/opencli
```

从我们的 fork 源码安装（推荐，带 `--jobType` 等过滤增强）：

```bash
git clone https://github.com/huanghuoguoguo/OpenCLI.git
cd OpenCLI
npm install
npm run build
npm link
```

## 快速开始

克隆本仓库：

```bash
git clone https://github.com/huanghuoguoguo/job-hunter.git
cd job-hunter
cc
```

在 `cc` 里直接发送：

```text
请根据 README 帮我检查并配置 job-hunter 的运行环境：
1. 检查 opencli 是否可用，不可用就指导我安装。
2. 运行 opencli doctor，确认 Browser Bridge 和 Chrome 连接正常。
3. 提醒我打开 Chrome 并登录 zhipin.com。
4. 如果还没有用户偏好.md，就从用户偏好.template.md 复制一份。
5. 引导我填写地点倾向、薪资倾向、岗位倾向、行业倾向、公司倾向、经验学历和投递规则。
6. 配置完成后，用 /job-hunter 或 SKILL.md 中的流程帮我搜索岗位。
```

Windows PowerShell 手动复制模板：

```powershell
Copy-Item .\用户偏好.template.md .\用户偏好.md
```

macOS/Linux 手动复制模板：

```bash
cp 用户偏好.template.md 用户偏好.md
```

## 环境检查

运行：

```bash
opencli doctor
```

你需要看到 Browser Bridge 和浏览器连接是可用状态。然后打开 Chrome，登录：

```text
https://www.zhipin.com
```

可以先用一个小搜索验证：

```bash
opencli boss search "产品经理" --city 北京 --limit 3 -f json
```

如果能返回岗位 JSON，说明 BOSS 搜索链路可用。

## 配置文件

本仓库提供模板：

| 文件 | 说明 |
|------|------|
| `SKILL.md` | skill 主体说明和执行流程 |
| `CLAUDE.md` | Claude Code 打开仓库后的项目指令 |
| `用户偏好.template.md` | 用户求职偏好配置模板 |
| `.gitignore` | 忽略真实用户偏好、投递记录和本地日志 |

复制 `用户偏好.template.md` 为 `用户偏好.md` 后，填写自己的偏好：

- 不投递公司
- 谨慎投递公司
- 跳过岗位关键词
- 目标岗位关键词
- 跳过行业关键词
- 目标城市
- 经验、学历、薪资
- 工作方式和投递规则
- 其他备注

`用户偏好.md` 已被 `.gitignore` 忽略。用户的地点、薪资、岗位倾向、公司倾向、行业倾向等长期配置都写在这个文件里；偏好变化时只改 `用户偏好.md`，不要改 `SKILL.md`。

不要提交真实简历、真实用户偏好、投递记录、Cookie、截图或浏览器会话信息。

## 使用方式

在 `cc` 中可以直接说：

```text
帮我用 job-hunter 搜索 10 个岗位，只看上海和杭州，先不要投递，只输出推荐列表。
```

或：

```text
帮我投递 20 个符合用户偏好.md 配置的岗位，投递前检查按钮状态，已沟通过的跳过。
```

skill 会按 `SKILL.md` 的流程执行：

1. 读取用户目标和 `用户偏好.md`。
2. 使用 `opencli boss search` 搜索岗位。
3. 使用 `opencli boss detail` 补充详情。
4. 过滤不符合要求的岗位。
5. 打开岗位页检查按钮文案。
6. 只在按钮为"立即沟通"时辅助点击。
7. 输出本轮结果表格。

## 常用命令

```bash
# 推荐：加 --jobType 全职 直接过滤掉实习岗（需要 fork 版本）
opencli boss search "<关键词>" --city <城市> --jobType 全职 --limit 20 -f json

# 不带 jobType 时会混合校招与实习，需要在岗位名上后处理
opencli boss search "<关键词>" --city <城市> --limit 20 -f json

opencli boss detail "<security_id>" -f json
opencli browser open "<job_url>"
opencli browser eval "document.querySelector('.btn-startchat')?.textContent?.trim()"
opencli browser click ".btn-startchat" --nth 0
```

## 故障处理

- `opencli: command not found`：安装 OpenCLI，或确认 npm 全局 bin 在 PATH 中。
- `Browser Bridge not connected`：安装并启用 OpenCLI Browser Bridge 扩展，然后重新运行 `opencli doctor`。
- BOSS 返回空结果：确认 Chrome 已登录 `zhipin.com`，并尝试换城市或关键词。
- 按钮不是"立即沟通"：不要自动点击，按 `SKILL.md` 记录为已沟通过、待确认或异常。

## 许可证

未指定。按你的仓库策略添加 `LICENSE`。
