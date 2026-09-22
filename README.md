# Ship with AI

本站点是 **AI Genius —— 第 5 季第 3 集："Ship with AI：自信地完成审查、加固与部署"**
的配套网站与现场演示仓库。

该仓库演示了如下生命周期：

```text
Issue → Copilot 起草 PR → Copilot 代码审查 → Agent 自动合并 →
GitHub Actions 安全门禁 → Dependabot 修复 → GitHub Pages
```

其安全主题是 **OWASP Top 10:2025 A03 —— 软件供应链失效**。可录制的初始状态故意包含一个
过时的依赖项、一个采用不安全默认配置的未激活工作流，以及一个可选的伪造密钥扫描（fake
secret-scanning）测试数据。核心演示不依赖 Generic patterns（通用模式），因为该设置并非对
所有演示者可用。演示者仍需展示 Secret Protection（密钥保护）与 Push protection（推送保护）
的配置位置。

带版本号的发布契约存放在 [`demo-kit.json`](./demo-kit.json) 中。维护者按照
[`MAINTAINER-RELEASE.md`](./MAINTAINER-RELEASE.md) 发布新的模板版本。

## 选择你的路径

| 我想…… | 从这里开始 |
|---|---|
| 跟随本场分享的节奏 | [`AUDIENCE-WALKTHROUGH.md`](./AUDIENCE-WALKTHROUGH.md) |
| 搭建并完整排练演示 | [`MANUAL-DEMO-GUIDE.md`](./MANUAL-DEMO-GUIDE.md) |
| 排练完成后进行演示或录制 | [`PRESENTER-RUNSHEET.md`](./PRESENTER-RUNSHEET.md) |
| 理解安全课程要点 | [`src/pages/secure-supply-chain.astro`](./src/pages/secure-supply-chain.astro) |
| 探索具体实现 | [`src/pages/pipeline.astro`](./src/pages/pipeline.astro) |

`RUNSHEET.md` 保留为指向两份角色专属指南的兼容性入口。

## 本地运行

请使用 Node.js 24，版本由 `.nvmrc` 指定。

```bash
git clone https://github.com/anothergeorgecoldham/ship-with-ai.git
cd ship-with-ai
npm ci
npm run dev
```

初始状态包含刻意设置的训练用告警项。本地运行适合学习用途；请勿将其作为生产应用部署。
`npm run build` 会在 `dist/` 目录中生成静态站点。

## 仓库结构

```text
src/
  pages/                         学习内容页面
  components/FeedbackWidget.astro
  lib/                           反馈逻辑与仅演示用的配置
scripts/
  bootstrap-demo.mjs             带防护的仓库配置脚本
  preflight-demo.mjs             录制就绪检查
  check-audit-state.mjs          确定性的依赖策略检查
.github/
  demo/deploy.yml                未激活的审查示例工作流
  workflows/initialize-demo.yml  一次性的"改造前"部署
  workflows/pull-request-checks.yml
  workflows/dependency-policy.yml
  dependabot.yml
```

## 准备演示仓库

基于已发布的模板创建一个一次性仓库。预览 bootstrap 变更：

```bash
npm run demo:bootstrap -- --repo <owner>/<repository>
```

仅在确认目标仓库无误后才应用变更：

```bash
npm run demo:bootstrap -- --repo <owner>/<repository> --apply
```

Bootstrap 会启用自动合并（auto-merge），并创建一个激活的默认分支规则集，要求 GitHub
Actions 的 `build` 与 `audit` 检查通过，且所需人工审批数为零。它还会阻止默认分支被删除
和强制推送，且不设置任何绕过者（bypass actors）。不要假设模板设置和规则集会随之继承。

`audit` 检查会在每个指向 `main` 的 PR 上运行。当 `marked` 仍处于其预置版本时，它只允许
预期的含漏洞初始状态通过；Dependabot 的更新以及修复之后的 PR 必须通过干净状态策略。
生产部署则始终要求干净状态。

自动合并仍需针对每个非草稿 PR 单独启用，不会自动对所有 PR 生效。如果检查已全部通过，
直接合并属于正常情况。手动验证与恢复方法参见
[分支规则与自动合并设置](./MANUAL-DEMO-GUIDE.md#branch-rules-and-auto-merge)。

录制之前，请手动确认 Copilot coding agent 与 Agent Merge 可用，然后运行：

```bash
npm run demo:preflight -- --repo <owner>/<repository> --confirm-copilot
```

在最后一行输出 `READY TO RECORD` 之前不要开始录制。除非显式传入 `--allow-canonical`，
否则会保护规范源仓库不被 bootstrap 写入。
