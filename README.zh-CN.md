# CR-Sentinel

[English](README.md) | **中文**

---

一个 [Claude Code](https://claude.com/claude-code) skill —— 在机器学习论文**投稿或 camera-ready 阶段**做最后一轮 sanity check。

它**不会**替你改论文。它只把那些会损害审稿人信任、或者直接卡住提交流程的问题找出来：

- 未解析的 `\ref` / `\cite`（`??`、`[?]`）、未定义的 label、残留的 `TODO`/`FIXME`
- Abstract / intro 里的数字对不上最终表格
- 重跑实验后过期的 best / second-best 加粗
- 图注和图本身对不上
- 看起来合理但其实查不到的引用
- Rebuttal 痕迹（"我们感谢审稿人……" "为回应审稿人的关切……"）混进了 camera-ready
- 找不到目标的 Appendix 引用
- 提交流程风险（页数、表单、合作者签字）
- **忘记去匿名化**，或反过来 —— 投稿阶段身份泄漏

输出是一份按严重程度排序的 punch list：**Blockers / Should-fix / Worth a look / Could not verify / Submission checklist**，最后还会给一张**章节状态表**让你一眼看清还有什么没修。

---

## 安装

### 方案 A —— Plugin marketplace（推荐）

在 Claude Code 里一条命令搞定：

```
/plugin marketplace add SeanAlpaca818/CR-Sentinel
/plugin install CR-Sentinel@CR-Sentinel
```

需要的话重启 Claude Code。装完后 skill 全局可用。

**升级到新版本：**

```
/plugin marketplace update CR-Sentinel
/plugin update CR-Sentinel@CR-Sentinel
```

第一条刷新 marketplace 元数据让 Claude Code 看到新版本；第二条执行升级。

如果 `/plugin update` 把你带到 Discover 面板而不是直接升级（Claude Code 在检测不到版本 delta 时会这样），就走兜底方案 —— 干净卸载再装：

```
/plugin uninstall CR-Sentinel@CR-Sentinel
/plugin install CR-Sentinel@CR-Sentinel
```

### 方案 B —— Claude.ai 网页端 / 桌面 app（上传 `.skill` 文件）

Claude Code 的 plugin 不会在 claude.ai 出现。给网页端（或 Mac/Windows 桌面 app）用，要上传打包好的 `.skill` 文件：

1. 从仓库下载 [`cr-sentinel.skill`](cr-sentinel.skill)。
2. 在 claude.ai 打开 **Settings → Capabilities → Skills**（有些账号叫 **Features**）。
3. 点 **Upload skill**，选刚下载的文件。
4. 任何新对话里它会在你提到 camera-ready / 论文 review 类问题时自动触发，也可以在 "+" 菜单里手动挂上。

注意：网页端要求 skill 名字小写，所以那边显示成 **`cr-sentinel`**。Claude Code 版本保留 `CR-Sentinel` 名 —— 同一个 skill 的两种封装，互不影响。

### 方案 C —— 手动安装（plain skill，Claude Code）

直接 clone 到 Claude 的 skills 目录：

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/SeanAlpaca818/CR-Sentinel.git /tmp/cr-sentinel
cp -r /tmp/cr-sentinel/skills/CR-Sentinel ~/.claude/skills/
```

或者一条命令：

```bash
git clone --depth 1 https://github.com/SeanAlpaca818/CR-Sentinel.git \
  && cp -r CR-Sentinel/skills/CR-Sentinel ~/.claude/skills/ \
  && rm -rf CR-Sentinel
```

新开一个 Claude Code session 就能用了。

---

## 使用

装好之后，下面几种方式都能触发：

- **显式**：`/CR-Sentinel`，或"用 CR-Sentinel 帮我 review 这篇论文"
- **隐式** —— 你说类似这些话时它会自动触发：
  - "我在准备 NeurIPS 的 camera-ready，你能帮我看下 `main.tex` 吗？"
  - "帮我把 rebuttal 的实验整合进 appendix，但别让它读起来像 rebuttal 回复。"
  - "确认下 abstract 里的数字和 Table 2 对得上。"
  - "投稿前最后检查 —— PDF 有什么问题没？"

调用时它**一定会先问你两个问题**：
1. 哪个会议、哪一年（NeurIPS 2026、ICLR 2026 这种）
2. 投稿阶段还是 camera-ready 阶段

然后它会**联网查那一年的 policy**（页数、必须章节、模板、匿名规则等），按 policy 调整后再开始 check。

### 你需要给它什么

要拿到最好的报告，把这些指给 Claude：

1. LaTeX 源码目录（`main.tex` 之类）
2. 编译好的 PDF
3. 如果有的话，`.log` / `.blg` 文件
4. 已接收 / 已投稿那一版（这样它能 diff 出真正变化的部分）

只有 PDF 也行 —— skill 还是能跑，只是它会明确标注哪些项"无法验证"。

---

## 它会检查什么

| 类别 | 例子 |
|---|---|
| 构建健全性 | `??`、`[?]`、未定义的 ref、`TODO`/`FIXME`、靠近页数上限的 overfull box |
| 去匿名化 / 匿名性 | "Anonymous Submission" 残留、匿名 GitHub 链接、PDF metadata 泄漏作者；投稿阶段反向检查 |
| Scope & claims | Abstract 出现接收版里没有的新主张 |
| 数字 & 表格 | Headline 加速比、bold/underline 标错、metric scale 不一致、caption 缺单位 |
| 图 | 图注 vs. 图内容、左右上下描述、硬编码的图号 |
| 数学符号 & 公式引用 | $\theta$ vs $\phi$ 漂移、硬编码 "Equation 4"、`\cref` 不一致 |
| 引用 & URL | 查不到的条目、把 GitHub repo 当 paper 引、未加 `{{}}` 的公司作者、DOI 404 |
| Appendix | `\ref{app:...}` 能不能 resolve、rebuttal 腔残留、关键 takeaway 没浮到正文 |
| 实验设置 | 模型版本、LoRA rank、GPU-hours、unique 样本 vs. 训练曝光数 |
| 语言 & tone | "we thank the reviewer" 类残留、绝对化动词（"proves"、"obviously"）、未限定的 claim |
| 致谢 & 作者信息 | 第一次出现的致谢段、funding 格式、作者顺序与 venue portal 一致 |
| Venue-specific | NeurIPS Paper Checklist、ICLR Reproducibility、ACL/EMNLP/NAACL Limitations、ICML Broader Impact |
| 提交风险 | 页数、表单、合作者签字、PDF 字体内嵌、补充材料政策 |

完整规则见 [`skills/CR-Sentinel/SKILL.md`](skills/CR-Sentinel/SKILL.md)。

---

## License

MIT
