# job-hunter

BOSS 直聘岗位搜索、筛选、重复检测与辅助投递 skill，依托 `opencli boss` 和 `opencli browser` 工作。

## 功能

- 使用 `opencli boss search` 搜索岗位或获取推荐岗位。
- 使用 `opencli boss detail` 查看岗位详情。
- 读取用户自定义 `黑名单.md` 做公司、岗位、行业、城市、薪资等过滤。
- 打开岗位页面后检查按钮文案，避免重复沟通。
- 在用户明确要求投递且按钮为"立即沟通"时辅助点击。
- 输出已沟通、已沟通过、跳过、待确认、异常等结果记录。

## 使用前准备

1. 安装并配置 `opencli`。
2. 打开 Chrome，并登录 `zhipin.com`。
3. 安装并连接 Browser Bridge 扩展。
4. 将 `黑名单.template.md` 复制为 `黑名单.md`，填写自己的过滤规则。

## 文件

| 文件 | 说明 |
|------|------|
| `SKILL.md` | skill 主体说明和执行流程 |
| `黑名单.template.md` | 用户过滤配置模板 |
| `.gitignore` | 忽略真实黑名单、投递记录和本地日志 |

## 配置

复制模板：

```bash
cp 黑名单.template.md 黑名单.md
```

然后在 `黑名单.md` 中填写自己的公司黑名单、谨慎公司、跳过岗位、目标岗位、城市、经验、学历、薪资等偏好。

不要提交真实 `黑名单.md`、简历、投递记录、Cookie、截图或浏览器会话信息。

## 依赖命令

```bash
opencli boss search "<关键词>" --city <城市> --limit 20 -f json
opencli boss detail "<security_id>" -f json
opencli browser open "<job_url>"
opencli browser eval "document.querySelector('.btn-startchat')?.textContent?.trim()"
opencli browser click ".btn-startchat" --nth 0
```

## 许可证

未指定。按你的仓库策略添加 `LICENSE`。
