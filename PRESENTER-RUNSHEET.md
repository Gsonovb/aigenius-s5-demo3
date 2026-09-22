# 演示者流程表 —— "Ship with AI"

本文件是面向其他语言或地区重新交付 AI Genius S5E3 的操作指南。观众应使用
[`AUDIENCE-WALKTHROUGH.md`](./AUDIENCE-WALKTHROUGH.md)。
发布契约记录在 `demo-kit.json` 中；不要在演示者仓库里修改其中的版本。

首次登台前，演示者必须按
[`MANUAL-DEMO-GUIDE.md`](./MANUAL-DEMO-GUIDE.md) 排练，该指南包含从模板创建直到最终部署的
每一条设置命令与界面操作。本流程表是压缩后的录制视图。

## 交付契约

- **目标时长：** 10–12 分钟。
- **观众视角的故事线：** issue → AI 起草 → AI 审查 → 自动修复 → 安全门禁 → 部署。
- **仅在以下条件满足时开始：** `npm run demo:preflight -- --repo <owner>/<repo> --confirm-copilot`
  以 `READY TO RECORD` 结束。
- **不要提前修复：** 未激活的生产工作流或 `marked@0.3.19`。
- **可选：** 仅在 Generic patterns 可用时展示演示密钥测试数据。

## 翻译指引

翻译口头讲解、issue 标题和课程内容。不要翻译文件名、命令、工作流名称、设置名称、
依赖名称或预期状态标签。每个讲解点保持一句话，使字幕和同传与屏幕内容保持同步。

## 至少提前一天准备

1. 基于已发布的演示模板创建新的公开仓库并克隆。
2. 运行 `npm ci`。
3. 预览 bootstrap 变更：

   ```bash
   npm run demo:bootstrap -- --repo <owner>/<repo>
   ```

4. 应用配置并初始化"改造前"站点：

   ```bash
   npm run demo:bootstrap -- --repo <owner>/<repo> --apply
   ```

5. 确认激活的默认分支规则集要求来自 GitHub Actions 的 `build` 与 `audit` 检查。
   在一次独立的排练中，确认在有变更的功能实现会话里，**Create PR** 下拉菜单中出现
   **Agent merge**。这是对应用（app）的检查，不是仓库设置，也不是对 Dependabot 审查
   会话的检查。
6. 等待 `marked` 的 Dependabot PR。如果 Generic patterns 可用，同时等待可选的演示密钥
   扫描告警。
7. 运行 preflight。录制前解决每一项失败。

Bootstrap 会配置 Pages、CodeQL、Dependabot、密钥扫描、推送保护、自动合并、要求 `build`
与 `audit` 检查且所需人工审批为零的规则集，以及自动 Copilot 审查。它会尝试启用
Generic patterns，但该可选设置并非对所有账户可用。仓库设置不会从 GitHub 模板继承。

## 录制前的最终检查

- "改造前"的 Pages URL 可以打开。
- 功能 issue 尚未创建。
- `.github/workflows/deploy.yml` 尚不存在。
   恰好有一个针对 `marked` 的 Dependabot PR 处于打开状态。
- `build` 与 `audit` 均为必需；`audit` 在纯功能 PR 上也会运行。
- 实现会话的 **Create PR → Agent merge** 路径已经排练过。不要把 **Submit review**
  会话或 GitHub 的 **Enable auto-merge** 与该路径混为一谈。
- 任何关于 Generic patterns 的 preflight 提示都只是信息性输出，不构成阻断。
- 没有无关的浏览器标签页、通知或凭据可见。
- preflight 报告 `READY TO RECORD`。

## Beat 0 —— Issue → PR

**时间：** 2 分钟。打开 **New issue** 并选择功能请求模板。仅在需要时翻译课程措辞；
保留文件名和验收标准。

> **Title:** Add Episode 3 lesson page / update the feedback widget
>
> Add a short lesson page (or update an existing one) covering the Episode 3 talking point. Add a
> topic field to the feedback widget and persist it with each submission.
>
> Activate `.github/demo/deploy.yml` as `.github/workflows/deploy.yml` without changing the
> provided workflow. Acceptance criteria: new/updated page renders under the site nav; feedback
> widget captures the topic and renders a submission end-to-end; `npm run build` succeeds.

仅创建 issue 本身不会创建 PR。按以下顺序操作：

1. 创建 issue，然后在 Copilot 应用的 **My work** 中打开它并选择 **New session**。
   使用本地 worktree 实现会话；不要同时把 issue 指派给云端 agent。
2. 要求 Copilot 实现该 issue，保留预置的审查发现，并在 diff 可供检查时停下，不提交、
   不推送、不创建/合并 PR。
3. 检查 diff。如果 `package.json` 或 `package-lock.json` 有变化，让其还原这些文件。
4. 打开 **Create PR** 下拉菜单，展示 **Agent merge** 可用，但保持 **Create PR** 被选中。
   点击主按钮 **Create PR** 发布非草稿的功能 PR，并用 `Closes #<issue-number>` 关联 issue。
5. 在 Beat 1 和 Beat 2 期间保持这个原始实现会话打开。

云端指派同样可以生成 PR，但那是替代路径。在新审查会话中打开那个 PR 可能显示
**Submit review**，而不是本流程使用的创作控件。

**讲解点：** *"AI 写得很快，连流水线都搭好了 —— 但你会直接照原样发布吗？"*

## Beat 1 —— GitHub Copilot Code Review

**时间：** 2 分钟。打开 Copilot 创建的 PR，展示构建检查为绿色，而构建任务中的原始依赖
审计仅作参考信息。独立的必需 `audit` 检查执行的是演示预期状态，并不要求在功能合并前
完成修复。

被激活的工作流和更新后的反馈处理器都在 PR diff 中。Copilot 应留下行内评论，指出
（在其仍然存在的范围内）：

- Actions 用标签而非完整提交 SHA 固定（`.github/workflows/deploy.yml`）
- `permissions: write-all` 而非最小权限（`.github/workflows/deploy.yml`）
- 反馈提交处理器缺少输入校验（`src/lib/feedback.js`）

### 点出扫描器抓不到的发现

在审查评论中，停在缺失输入校验这一条上：反馈组件接受空值且没有长度上限。把它与
工作流和依赖项的发现做对比。

说：*"注意这一条。任何扫描器都不会发现它 —— 它不是漏洞，只是代码不够好。这就是
扫描器和审查者的区别。"*

Preflight 会校验该发现仍存在于初始状态中。如果这项检查失败，请在录制前恢复预置内容；
缺少它，Code Review 和安全环节就会讲成同一个故事。

**讲解点：** *"让 AI 当你的审查者 —— 它读懂的是意图，不只是特征签名。"*

## Beat 2 —— Agent Merge ⭐

**时间：** 2 分钟。

1. 回到**原始功能实现会话**，而不是新的 PR 审查会话。
2. 要求 Copilot 处理审查发现：将 Actions 固定到 SHA、收窄权限、添加校验。检查修复内容，
   让其提交/推送到同一个 PR，不更改依赖。
3. 打开 PR 操作下拉菜单，选择 **Agent merge**，并展示按钮标签的变化。
4. 点击主按钮 **Agent merge**，为关联的功能 PR 启动它。
5. 展示其允许的操作。按需允许审查/CI/冲突修复，只有在准备落地已检查过的变更时才允许
   **Merge pull request**。如果合并权限已经开启，先完成检查再启动。
6. 展示 agent 检查 PR 的过程，以及必需检查通过后 PR 的 **Merged** 状态。

**明确说出：** *"这是 Copilot 应用里的 Agent Merge，不是 GitHub 的 Enable auto-merge。
GitHub 负责强制执行检查；agent 负责管理被允许的后续工作和合并。"*

不要依赖可见的等待窗口。Agent Merge 在检查已全绿时也能工作，合并可能很快发生。自动
Copilot 审查彼此独立，不需要人工审批。手动合并或原生自动合并不能替代这一环节的展示。

**讲解点：** *"审查发现变成经过验证的变更，而不是又一次手动交接。"*

## Beat 3 —— GitHub Actions（CI/CD）

**时间：** 1 分钟。

展示已完成加固的 `deploy.yml` 运行在 `npm audit` 门禁处停下。在 Beat 4 合并 Dependabot PR
之后，下一次运行将会完成。

### 展示工作流，而不只是运行它

在打开失败的生产运行之前，先打开 `.github/workflows/pull-request-checks.yml`。只指向：

- `on: pull_request` —— *"每个拉取请求都会运行它。"*
- `npm audit --audit-level=high` —— *"它在合并前报告依赖风险；在预置状态下仅作参考信息，
  以便我们演示生产门禁。"*
- `permissions: contents: read` —— *"它使用最小权限令牌，这正是 Copilot 提醒我们在生产
  工作流中修复的问题之一。"*

然后打开已加固的 `.github/workflows/deploy.yml`，指向
`node scripts/check-audit-state.mjs clean`：*"这就是强制执行门禁。在审计干净之前，它阻止
部署。"*

之后回到 **Actions**。不要逐行朗读任何一个文件。

说：*"这两段简短的工作流就是流水线：拉取请求会被检查，只有干净的 `main` 才能部署。
一次配置，此后每次变更都要经过它。"*

**讲解点：** *"构建并交付你代码的流水线，需要和代码本身接受同等严格的审视。"*

## Beat 4 —— AI 辅助的安全审查

**时间：** 2 分钟。展示：

- `marked@0.3.19` 的 **Dependabot 告警**（`package.json`）—— 该依赖确实被反馈组件使用，
  且受已知正则表达式拒绝服务漏洞影响。
- **必需设置展示：** 打开 **Settings → Security and quality → Advanced Security**，展示
  **Secret Protection** 以及已启用的 **Push protection**。说明受支持的提供商密钥会在
  推送前被检测并阻断。
- **可选：** 如果 Generic patterns 可用，展示密钥扫描标记出
  `src/lib/demo-secret-fixture.js` 中的伪造 bearer 头。它是一个无实际功能的训练值。

打开预备好的 `marked` Dependabot PR，展示其绿色检查，审查 diff，然后在 GitHub 上合并。
接着打开新的 **Build and deploy** 运行。这是依赖修复步骤；Agent Merge 已在 Beat 2 的
功能 PR 上明确演示过。不要假设有 **Submit review** 的 Dependabot 会话会暴露 Agent Merge
控件。只有在单独排练验证过的情况下才使用那条替代路径。

**讲解点：** *"这是供应链层面 —— 与 Beat 1 的代码审查是不同的层面。"*

## Beat 5 —— 端到端生命周期自动化

**时间：** 1–2 分钟。展示成功的 构建 → 审计 → 部署 运行，然后打开 Pages URL，
在更新后的页面上提交一条反馈。

拉远到架构图：

```
Issue → Copilot drafts PR → Copilot Code Review → Agent Merge →
GitHub Actions (build + supply-chain security + deploy) → GitHub Pages
```

**讲解点：** *"一次配置；未来的每次变更都让供应链保持健康。"*

**收尾点题：** *"你现在读到的这课内容，正是通过你刚看着走完的流水线发布的。"*

## 预期证据

| 预置问题 | 位置 | 由谁捕获 |
|---|---|---|
| 过时的 `marked` 依赖 | `package.json` | Dependabot |
| 未固定版本的 Actions | `.github/workflows/deploy.yml` | Copilot Code Review |
| 过宽的 `permissions: write-all` | `.github/workflows/deploy.yml` | Copilot Code Review |
| 受支持的提供商密钥 | 仓库推送 | Secret Protection 与 Push protection |
| 可选的伪造 bearer 头测试数据 | `src/lib/demo-secret-fixture.js` | 通用密钥扫描（如可用） |
| 缺失的输入校验 | `src/lib/feedback.js` | Copilot Code Review |

`marked` 的升级解决的是其依赖公告问题。对不可信渲染 HTML 做净化是独立的应用安全
议题，不属于核心录制内容。

## 恢复路径

| 情况 | 应对 |
|---|---|
| Copilot 更改了依赖 | 在审查前要求其还原 `package.json` 和 `package-lock.json` |
| 预期的审查评论缺失 | 请求一次重新审查；然后展示预置代码行并讲解应有的发现 |
| 功能 PR 检查变红 | 停下来诊断；不要绕过必需的检查 |
| Dependabot PR 缺失 | 停止，等 GitHub 扫描完成后重跑 preflight |
| Generic patterns 或其告警不可用 | 继续，省略可选的密钥扫描环节 |
| 工具栏显示 **Submit review** | 回到原始功能实现会话；不要用 Dependabot 审查会话演示创作控件 |
| 空会话没有 **Create PR** | 先实现功能，待变更就绪后再查看 PR 操作下拉菜单 |
| Agent Merge 不可用 | 使用预备的兜底录制；不要静默改用手动合并 |
| 最终部署失败 | 保留失败的运行可见，切换到预备的成功运行录制 |

录制期间不要重置、修补或临时改动依赖。另拍一条时请使用全新的模板仓库。
