# essay-translation-grader

一个给 **英语作文与段落翻译** 评分批改的 AI Skill（面向 Claude / ZCode 等支持 SKILL.md 的 agent）。

## 它做什么

用户在对话里给一个文件路径（题目 + 自己的作答），Skill 会：

- 读出内容，自动判断哪段是**题目**、哪段是**作答**；
- 按四六级标准给作文和翻译**打分定档**；
- 直接输出一份固定格式报告：**总分 + 分项分、逐句批改、改写范例、总体评语**。

不写文件，报告直接在对话里给出，用户复制走。

## 评分标准

- **作文**：大学英语四六级官方整体评分（Global Scoring）五档 —— 14 / 11 / 8 / 5 / 2 分，含字数扣分、只写一两段、跑题白卷等细则。
- **翻译**：四六级官方段落翻译五档 —— 13–15 / 10–12 / 7–9 / 4–6 / 1–3 分，以"是否准确表达原文意思"为主。
- **考研英语**（备用）：英语一 / 英语二大小作文档位。
- 报分以 **710 分制**为主（15 分制 × 7.1），同时给出 15 分制档位。

## 档位校准

`references/calibration.md` 收录了四六级官方五份样卷（档位 14 / 11 / 8 / 5 / 2）全文与档间分界要点。判档时拿不准，就找错误密度和切题度最接近的样卷对照 —— 区分档位看的是**内容扣提纲的程度**和**错误是否妨碍理解**，不是字数。

五份样卷实测：本 Skill 独立判档与官方档位**五档全中**。

## 怎么用（给别人看的一段）

Skill 不是远程服务，没有 API 可以被“调用”——它就是一个本地文件夹（`SKILL.md` + 素材）。别人要用，就是把本仓库**装进他自己 agent 的 skills 目录**，之后那个 agent 就会在对话里自动触发它。整件事只有两步：**装到 skills 目录 → 新开会话**。

### 第一步：装到 skills 目录

克隆到对应 agent 的 skills 路径（用 HTTPS，无需配置 SSH key）：

```bash
# ZCode（用户级，所有项目可用）
git clone https://github.com/wxyqx/essay-translation-grader.git ~/.zcode/skills/essay-translation-grader

# Claude Code（用户级）
git clone https://github.com/wxyqx/essay-translation-grader.git ~/.claude/skills/essay-translation-grader
```

各 agent 的安装路径：

| agent | 用户级（全局） | 项目级（仅当前仓库） |
| --- | --- | --- |
| ZCode | `~/.zcode/skills/<name>/` 或 `~/.agents/skills/<name>/` | `<项目>/.zcode/skills/<name>/` 或 `<项目>/.agents/skills/<name>/` |
| Claude Code | `~/.claude/skills/<name>/` | `<项目>/.claude/skills/<name>/` |
| Claude.ai（网页版） | 把整个仓库打包成 ZIP，在 Settings → Capabilities → Skills 里上传 | — |

> 目录名（这里是 `essay-translation-grader`）必须和 `SKILL.md` 里的 `name` 字段一致，否则不生效。

### 第二步：新开会话

Skill 是在会话**初始化时**被扫描发现的，装完要**新开一个会话**（不是刷新当前对话）。

### 怎么触发

- **自动触发**：用户直接说“帮我看看这篇作文/翻译能不能过”“给这篇估个分”“批改一下”，agent 匹配到 `SKILL.md` 里 `description` 描述的语境就会自动加载。
- **强制触发**：用斜杠命令显式点名，最稳：
  ```
  /skill essay-translation-grader 批改 D:\作文\我的作文.txt
  ```
  （部分客户端是 `/essay-translation-grader`，取决于其命令解析。）

### 想一行命令装好？

克隆方式已是最简。若希望别人确认后再装，也可把本仓库做成 agent 市场的插件包，但多数场景下“git clone 到 skills 目录”就够了。

## 目录结构

```
essay-translation-grader/
├── SKILL.md                      # 主文件：输入处理、评分标准、输出模板、给分规则
├── references/
│   └── calibration.md            # 作文档位校准样卷
└── README.md
```

## 说明

- 作文档来自四六级官方评分资料；翻译与考研档为按通用标准整理，尚无官方样卷校准。
- 输入若同时含作文与翻译，两部分分别出报告；只有其一则只评其一。
