---
name: job-hunter
description: >
  BOSS直聘岗位搜索、筛选、重复检测与辅助投递工作流。
  当用户说"找工作"、"投递岗位"、"帮我投递"、"Boss直聘"时使用。
  依托 opencli boss 搜索/查看岗位，结合用户自定义配置过滤岗位，并通过浏览器按钮状态避免重复沟通。
argument-hint: [简历路径] [目标数量] [关键词] [城市]
user-invocable: true
allowed-tools: Bash, Read, Agent
---

# BOSS 求职助手使用手册

**定位**：面向需要在 BOSS 直聘搜索、筛选、记录岗位的用户。该 skill 不绑定具体学校、届别、行业、岗位方向或个人经历，所有偏好都应从用户输入、简历和本地配置文件读取。

## 使用前准备

1. 已安装并可运行 `opencli`。
2. Chrome 已打开，并已登录 `zhipin.com`。
3. Browser Bridge 扩展已安装并连接。
4. 准备一份简历文件，或在对话中说明求职方向、城市、学历、经验、薪资等要求。
5. 如需固定求职偏好，复制 `用户偏好.template.md` 为 `用户偏好.md` 后自行填写。

## 核心能力

1. **岗位搜索**：使用 `opencli boss search` 按关键词、城市、经验、学历、薪资、行业搜索岗位。
2. **岗位详情**：使用 `opencli boss detail` 查看职位描述、公司信息、招聘者信息。
3. **偏好过滤**：按用户偏好文件中的公司、岗位、城市、薪资、经验、学历等要求过滤结果。
4. **重复检测**：打开岗位详情页，通过按钮文案判断是否已经沟通过。
5. **辅助投递**：仅在确认是可投递岗位时点击"立即沟通"。
6. **结果记录**：记录已投递、已沟通、跳过、失败的岗位和原因。

## 参数约定

| 参数 | 默认策略 | 说明 |
|------|----------|------|
| 简历路径 | 用户指定；未指定则询问或跳过简历分析 | 用于提取求职方向、技能、城市、学历等信息 |
| 目标数量 | 用户指定；未指定可用 10 | 本轮计划成功沟通的岗位数量 |
| 搜索关键词 | 用户指定优先；否则从简历和偏好生成 | 不要写死某个行业或岗位族 |
| 搜索城市 | 用户指定优先；否则从配置/简历推断 | 支持单城市或多城市 |
| 经验要求 | 用户指定优先；否则不限制 | 示例：应届、1年以内、1-3年 |
| 学历要求 | 用户指定优先；否则不限制 | 示例：大专、本科、硕士、博士 |
| 薪资范围 | 用户指定优先；否则不限制 | 示例：10-15K、15-20K |
| 行业要求 | 用户指定优先；否则不限制 | 示例：教育培训、金融、医疗健康 |

## 用户配置

默认读取同目录下的 `用户偏好.md`。如果文件不存在，提示用户从模板创建：

```text
复制 用户偏好.template.md
到   用户偏好.md
然后填写自己的求职偏好。
```

用户的长期求职配置都放在 `用户偏好.md`：地点、薪资、岗位倾向、行业倾向、公司倾向、经验学历、技能关键词、工作方式和投递规则。偏好变化时只改 `用户偏好.md`，不要改 `SKILL.md`。

## 工作流程

### 1. 收集目标

先明确本轮任务：

- 投递还是只搜索岗位？
- 目标数量是多少？
- 简历文件在哪里？
- 是否限定城市、行业、岗位、经验、学历、薪资？
- 是否需要读取 `用户偏好.md`？

如果用户已经给出足够信息，直接执行；信息缺失但不影响搜索时，使用保守默认值并在结果中说明。

### 2. 生成搜索词

关键词来源按优先级处理：

1. 用户明确指定的关键词。
2. 简历中的岗位目标、技能、项目方向。
3. `用户偏好.md` 中的目标岗位、岗位倾向和技能关键词。
4. 平台推荐岗位，不带关键词搜索作为补充。

不要把某个专业、行业、能力标签或城市写死到 skill 中。

### 3. 搜索岗位

关键词搜索：

```bash
opencli boss search "<关键词>" --city <城市> --limit 20 -f json
```

带筛选条件搜索：

```bash
opencli boss search "<关键词>" --city <城市> --experience <经验> --degree <学历> --salary <薪资> --industry <行业> --limit 20 -f json
```

推荐岗位：

```bash
opencli boss search --city <城市> --limit 20 -f json
```

多城市、多关键词时采用流水线方式：每次搜索一批，过滤后立即检查和记录，避免一次性打开过多页面。

### 4. 打开页面前过滤

读取 `用户偏好.md` 后，对搜索结果先做静态过滤：

- 公司名命中"不投递公司"：跳过。
- 公司名命中"谨慎投递公司"：记录为"待确认"，不要自动点击。
- 岗位名命中"跳过岗位关键词"：跳过。
- 岗位名命中"谨慎岗位关键词"：记录为"待确认"，不要自动点击。
- 行业、地点、薪资、经验、学历不符合用户要求：跳过。
- 岗位描述或技能标签明显不符合用户方向：跳过。
- 命中用户偏好中的谨慎项：记录为"待确认"，不要自动点击。

过滤必须保留原因，便于用户复盘。

#### 4.1 实习/猎头/匿名岗启发式过滤

BOSS 部分岗位虽然用了"应届"标签但实际是实习或猎头代发，这些应当跳过：

- **薪资单位是 `元/天` / `元/时`** → 实习岗位（不是月薪 K）。
- **岗位名包含 `实习 / 实习生`** → 实习岗位。
- **公司名包含 `某大型 / 某中型 / 某知名 / ㊙️` 等隐去公司名的字样** → 多为猎头/外包代发，建议跳过。
- **岗位名包含 `算法工程师 / 测试开发 / 嵌入式 / 具身 / 硬件` 等用户偏好的跳过类目**：直接跳过。

> 上游 `--jobType` 过滤需要安装 [huanghuoguoguo/OpenCLI](https://github.com/huanghuoguoguo/OpenCLI) fork 后可用：
> `opencli boss search "..." --jobType 全职` 直接在 API 层过滤掉所有实习岗。

### 5. 查看详情

搜索结果中有 `security_id` 时，优先用详情命令补充信息：

```bash
opencli boss detail "<security_id>" -f json
```

详情中重点检查：

- 岗位职责和任职要求。
- 公司行业、规模、融资阶段。
- 招聘者身份和活跃时间。
- 工作地点、薪资、学历、经验。
- 是否命中用户偏好中的跳过项或谨慎项。

### 6. 重复检测 + 活跃度检查（合并到一次 eval）

打开岗位页面后，**用一次 eval 同时拿 3 个信号**：按钮文案、HR 活跃文案、岗位发布时间。这样投递循环里每个岗位只需要 1 次 open + 2 次 eval，避免来回 round-trip：

```bash
opencli browser open "<job_url>"
sleep 3
BTN=$(opencli browser eval "document.querySelector('.btn-startchat')?.textContent?.trim() || ''" | tr -d '\r\n')
ACTIVE=$(opencli browser eval "document.querySelector('.boss-active-time')?.textContent?.trim() || ''" | tr -d '\r\n')
```

按钮文案处理：

| 按钮文本 | 状态 | 处理 |
|----------|------|------|
| 立即沟通 | 未沟通过 | 进入活跃度判断 |
| 继续沟通 | 已沟通过 | 跳过并记录 |
| 其他/空 | 状态不明 | 不自动点击，记录异常 |

HR 活跃度文案（`.boss-active-time` 元素）：

| 活跃文案 | 含义 | 默认处理 |
|----------|------|----------|
| `刚刚活跃` / `N 分钟内活跃` / `N 小时内活跃` | 当下/几小时内 | ✅ 投递 |
| `今日活跃` | 今日登录 | ✅ 投递 |
| `3 日内活跃` / `本周活跃` | 7 日内 | ✅ 投递 |
| `2 周内活跃` / `近期活跃` | 8-14 日 | ⚠️ 视用户要求决定 |
| `本月活跃` / `月内活跃` | 15-30 日 | ❌ 默认跳过（HR 已不主动） |
| `半年内活跃` / `去年活跃` / 空 | ≥ 30 日 / 不明 | ❌ 默认跳过 |

如果用户没特殊要求，建议默认 **跳过 `本月 / 月内 / 半年 / 去年` 以及更久的**。`2 周内 / 近期` 视情况，活跃文案空时 BOSS 通常代表 ≥ 一个月（也建议跳过）。

可选：从页面 JSON-LD 读岗位发布日期作为补充信号：

```bash
UPDATE=$(opencli browser eval "
  let d=''; document.querySelectorAll('script[type=\"application/ld+json\"]').forEach(s=>{
    try{const j=JSON.parse(s.textContent); if(j.upDate) d=j.upDate;}catch(e){}
  }); d
" | tr -d '\r\n')
```

`upDate` 是 ISO 时间戳（如 `2026-04-30T17:30:38`），用来识别新发布岗位（24-72 小时内）。

### 7. 辅助投递

只在满足以下条件时点击：

- 用户要求执行投递。
- 该岗位未命中过滤规则（含 4.1 启发式与活跃度检查）。
- 按钮文案明确为"立即沟通"。

```bash
opencli browser click ".btn-startchat" --nth 0
```

点击后等待 2-3 秒再继续下一个岗位。BOSS 直聘通常会发送用户在平台预设的招呼语；不要在 skill 中写死招呼语。

**典型批量投递循环（边搜边投，单次能稳定跑 50 个）**：

```bash
while read jid; do
  if [ $SUCCESS -ge $TARGET ]; then break; fi
  opencli browser open "https://www.zhipin.com/job_detail/$jid.html" > /dev/null 2>&1
  sleep 3
  BTN=$(opencli browser eval "document.querySelector('.btn-startchat')?.textContent?.trim() || ''" | tr -d '\r\n')
  ACTIVE=$(opencli browser eval "document.querySelector('.boss-active-time')?.textContent?.trim() || ''" | tr -d '\r\n')
  if [ "$BTN" = "继续沟通" ]; then
    echo "$jid | SKIP_DONE"
  elif [ "$BTN" = "立即沟通" ]; then
    if echo "$ACTIVE" | grep -qE '本月|月内|半年|去年'; then
      echo "$jid | SKIP_INACTIVE | $ACTIVE"
    else
      opencli browser click ".btn-startchat" --nth 0 > /dev/null 2>&1
      SUCCESS=$((SUCCESS+1))
      echo "$jid | OK $SUCCESS/$TARGET | $ACTIVE"
    fi
  else
    echo "$jid | ERR [$BTN]"
  fi
  sleep 2
done < /tmp/queue.txt
```

### 8. 结果输出

投递或搜索完成后输出简洁表格：

```markdown
**本轮完成**：目标 10 个，成功 8 个，跳过 12 个，异常 1 个

| 岗位 | 公司 | 城市 | 状态 | 原因 |
|------|------|------|------|------|
| 产品经理 | 示例公司 | 上海 | 已沟通 | 按钮为立即沟通 |
| 运营专员 | 示例公司 | 北京 | 跳过 | 公司命中不投递公司 |
| 市场经理 | 示例公司 | 杭州 | 已沟通过 | 按钮为继续沟通 |
```

状态建议统一为：

- `已沟通`
- `已沟通过`
- `跳过`
- `待确认`
- `异常`

## 命令速查

```bash
# 搜索岗位
opencli boss search "<关键词>" --city <城市> --limit 20 -f json

# 搜索推荐岗位
opencli boss search --city <城市> --limit 20 -f json

# 加筛选条件搜索
opencli boss search "<关键词>" --city <城市> --experience <经验> --degree <学历> --salary <薪资> --industry <行业> --limit 20 -f json

# 推荐使用 fork（带 --jobType 全职 过滤实习）
opencli boss search "<关键词>" --city <城市> --jobType 全职 --limit 20 -f json

# 查看详情
opencli boss detail "<security_id>" -f json

# 打开岗位页
opencli browser open "<job_url>"

# 一次拿三个信号：按钮 + HR 活跃 + 发布时间
opencli browser eval "document.querySelector('.btn-startchat')?.textContent?.trim()"
opencli browser eval "document.querySelector('.boss-active-time')?.textContent?.trim()"
opencli browser eval "let d=''; document.querySelectorAll('script[type=\"application/ld+json\"]').forEach(s=>{try{const j=JSON.parse(s.textContent); if(j.upDate) d=j.upDate;}catch(e){}}); d"

# 点击立即沟通
opencli browser click ".btn-startchat" --nth 0
```

## 进阶提示

### 关键词 vs 推荐
不带 query 的"推荐岗位"基于 BOSS 对用户的画像（曾点击/曾沟通），通常拉来高经验算法岗。**目标投递场景下关键词搜索精准度高得多**——按用户简历方向选 5-8 个关键词（如 `AI Agent / RAG / 大模型应用 / 智能体 / Java`），每个关键词 × 每个目标城市分别搜，去重后排序。

### 反爬规避
不要直接在搜索页用 `fetch` 连续调几十次 detail API，会触发 BOSS 的 `code 36/37 您的账户存在异常行为` 限流。**优先用 `opencli boss search/detail` CLI**——它内部会先 navigateTo 模拟真实浏览，再 fetch，反爬触发率低很多。批量循环之间至少留 2 秒 sleep。

### 上游已知问题
原版 `clis/boss/search.js` 的 `EXP_MAP` 把"应届"误标为 `108`（实际是 BOSS 的"在校生"代码），导致 `--experience 应届` 实际查的是实习。修复版本和 `--jobType` 过滤都在 [huanghuoguoguo/OpenCLI](https://github.com/huanghuoguoguo/OpenCLI) 的 `feat/boss-jobtype-filter` 分支（已提 PR [#1231](https://github.com/jackwener/opencli/pull/1231)）。从该分支 build 后能拿到正确的应届生（`102`）数据 + 实习/全职/兼职过滤参数。

### 候选打分参考
若候选数远多于目标数，建议加一个简单分数排序：

- 公司命中"大厂关键词"（阿里/字节/腾讯/美团/百度/快手/京东/拼多多/小米/网易/华为 等）+3
- 岗位名命中用户目标关键词（Agent/RAG/LLM/智能体 等）+2
- API 返回 `bossOnline: true`（HR 当下在线）+1

按分数倒排，先投高分。

