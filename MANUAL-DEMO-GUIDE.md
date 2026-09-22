# 完整手动演示指南

本指南带领演示者从一个全新的模板仓库走到最终部署的站点。在录制之前，先完整排练一遍。
录制期间保持 [`PRESENTER-RUNSHEET.md`](./PRESENTER-RUNSHEET.md) 打开，使用其中更精炼的
脚本与讲解要点。

GitHub 的界面标签会随产品演进变化。如果某个标签略有不同，请参考链接的 GitHub 文档，
并保证达成 **预期结果** 中描述的状态。

## 1. 确认前置条件

你需要：

- 一个能创建公开仓库的 GitHub 账户。
- 包含 Copilot coding agent 与 Copilot Code Review 的 Copilot 计划。
- 能访问 [GitHub Copilot 应用](https://github.com/copilot)。
- 已安装 Git、与 CI 一致的 [Node.js 24](https://nodejs.org/)、npm，以及
  [GitHub CLI](https://cli.github.com/)。
- 对该一次性演示仓库拥有管理员权限。

登录并验证工具：

```bash
node --version
npm --version
gh auth status
```

Node `22.12.0` 是强制的最低版本；推荐使用 Node 24。如果 GitHub CLI 报告缺少仓库或
工作流访问权限，请刷新其授权：

```bash
gh auth refresh -h github.com -s repo,workflow,read:org
```

不要用规范仓库 `anothergeorgecoldham/ship-with-ai` 进行排练或录制。

## 2. 从模板创建仓库

1. 打开 <https://github.com/anothergeorgecoldham/ship-with-ai>。
2. 在文件列表上方选择 **Use this template**。
3. 选择 **Create a new repository**。
4. 将你的账户或演示用组织选为 **Owner**。
5. 输入一个唯一名称，例如 `ship-with-ai-fr-demo`。
6. 选择 **Public**。
7. 保持 **Include all branches** 不勾选。
8. 选择 **Create repository**。
9. 等待新仓库页面加载完成。

**预期结果：** 新仓库的 `main` 分支上有一个初始提交。它与源仓库彼此独立，也不是 fork。

GitHub 参考文档：
[Creating a repository from a template](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template)。

## 3. 克隆并安装初始状态

替换下面命令中的两个占位符：

```bash
gh repo clone <owner>/<repository>
cd <repository>
npm ci
```

现在不要运行 `npm audit fix`、不要更新依赖，也还不要复制 `.github/demo/deploy.yml`。
初始状态是有意留有漏洞的。

**预期结果：** 安装完成，并报告那个刻意设置的 `marked` 告警。

## 4. 配置一次性仓库

先预览 bootstrap 将要做的变更：

```bash
npm run demo:bootstrap -- --repo <owner>/<repository>
```

阅读第一行打印的目标仓库。确认无误后应用配置：

```bash
npm run demo:bootstrap -- --repo <owner>/<repository> --apply
```

Bootstrap 会启用：

- 自动合并（auto-merge）；
- 一个默认分支规则集：要求 GitHub Actions 的 `build` 与 `audit` 检查、所需审批数为零、
  无绕过者，并启用删除与强制推送保护；
- Dependabot 告警与安全更新；
- 密钥扫描与推送保护；
- Generic patterns（在该可选设置可用时）；
- CodeQL 默认配置（default setup）；
- 自动 Copilot 代码审查；
- 使用 GitHub Actions 的 GitHub Pages；
- 一次性的 **Initialize demo site** 工作流。

**预期结果：** bootstrap 成功结束并打印初始化工作流的 URL。

## 5. 手动确认 GitHub 设置

即使 bootstrap 已经配置，也要做这些检查。它们能发现 API 响应无法证明的许可以及
策略限制。

### Copilot 功能

1. 打开你的 GitHub 个人 **Settings**。
2. 打开 **Copilot**，再打开 **Features**。
3. 确认 **Copilot code review** 已启用。
4. 确认 **Copilot cloud agent** 或 **Coding agent** 已启用。
5. 如果许可证由组织提供，盾牌图标可能表示该设置被组织强制。

### 仓库行为

1. 打开一次性仓库。
2. 选择 **Settings**。
3. 在 **General → Pull Requests** 下确认 **Allow auto-merge** 已启用。
4. 在 **Rules → Rulesets** 下打开 **Automatic Copilot code review**，确认其对默认分支
   处于激活状态。
5. 在 **Pages** 下确认 **Source** 为 **GitHub Actions**。
6. 在 **Security** 或 **Security and quality** 下确认 Dependabot、代码扫描、密钥扫描和
   推送保护均已启用。
7. 在 **Advanced Security → Secret Protection** 下查找 **Generic patterns**。可用就启用。
   如果未显示，则跳过可选的密钥扫描环节继续。

### 分支规则与自动合并

对每个从模板创建的仓库都要显式配置；不要假设模板的规则集和设置会随之继承。
bootstrap 会创建或更新如下规则集。在 **Settings → Rules → Rulesets** 下确认与之匹配：

| 设置 | 值 |
|---|---|
| Name | `Require build and audit before merge` |
| Enforcement status | **Active** |
| Target branches | **Default branch** |
| Bypass list | 空 |
| Restrict deletions | 启用 |
| Block force pushes | 启用 |
| Require a pull request before merging | 启用 |
| Required approvals | `0` |
| Require code owner review / approval of the most recent push | 禁用 |
| Require conversation resolution | 禁用 |
| Require status checks to pass | 启用 |
| Required checks | 来自 **GitHub Actions** 的 `build` 与 `audit` |
| Require branches to be up to date before merging | 禁用 |

如果规则集缺失或不正确，用 `--apply` 重跑 bootstrap（加 `--skip-deploy` 避免重新初始化
站点），或手动创建/更新。如果手动选择检查的列表中没有 `build` 和 `audit`，先开一个
准备性 PR，让工作流运行一遍，再选择这些确切的检查名称。Bootstrap 通过 API 配置它们，
无需等待选择器。

使用更新后的模板工作流以及上面的规则集。**Dependency policy** 必须在每个指向 `main` 的
PR 上运行，不带 `paths` 或 `paths-ignore` 过滤。否则纯功能 PR 会无限等待 `audit`。
bootstrap 修改的是仓库设置，而不是旧模板副本中的文件；在启用规则集之前，先把更新后的
工作流与审计脚本带入该仓库。

必需的 `audit` 检查保护了教学顺序：使用预置 `marked` 版本的普通 PR 必须匹配有意设置的
初始状态告警。Dependabot PR 以及修复之后的普通 PR 要求干净状态。生产门禁则始终要求
干净状态。因此一个绿色的演示 PR 并不意味着预置依赖对生产是安全的。

关于 GitHub 原生自动合并：将 PR 标记为 ready for review，并在必需检查待运行或失败时
选择 **Enable auto-merge**、确认合并方式。如果所有条件已经满足，直接合并是预期行为。
不要指望在录制时恰好抓到一个等待窗口。自动合并是按 PR 逐个启用的，手动或通过单独的
自动化；仓库设置不会让所有 PR 自动加入。Copilot 应用中的 Agent Merge 是另一套工作流，
见下文。自动 Copilot 审查保持独立，不要求人工审批。

### Coding agent 与 Agent Merge

1. 打开 [GitHub Copilot 应用](https://github.com/copilot)。
2. 打开 **My work**，确认可访问一次性仓库及其 issues。
3. 在一次单独的排练中，从功能 issue 启动一个实现会话，让它产出功能变更，如第 8 步所述。
4. 在该实现会话中，打开 **Create PR** 旁的箭头，确认列表中有 **Agent merge**。只查看，
   不要点击主操作按钮启动合并。
5. 保持录制仓库处于未改动的初始状态；不要复用功能 PR 或依赖 PR 已被合并过的仓库。

空会话可能还没有 **Create PR**。从他人 PR（包括 Dependabot 的）打开的会话可能显示的是
**Submit review**。这两种界面都不能证明 Agent Merge 可用。请使用该功能原始的、带有变更的
实现会话来做这项检查，而不是 PR 审查会话。

如果那里仍然没有 Agent Merge，请停下来检查应用版本、Copilot 计划、仓库访问权限和组织
策略。仅凭仓库规则无法证明该应用控件可用。

## 6. 等待安全准备完成

GitHub 需要时间扫描新的模板仓库。

1. 在仓库中打开 **Pull requests**。
2. 等待恰好一个更新 `marked` 的 Dependabot 拉取请求出现。
3. 打开 **Security** 或 **Security and quality**。
4. 在 **Dependabot** 下确认高危告警只涉及 `marked`。
5. 如果 Generic patterns 可用，确认存在一个指向 `src/lib/demo-secret-fixture.js` 的、
   处于打开状态的 HTTP bearer 头告警。否则跳过这个可选检查。
6. 在 **Actions** 下确认 **Initialize demo site** 已成功。
7. 打开该次运行的 Pages URL，确认初始站点可以加载。

不要合并或关闭已备好的 Dependabot 告警。如果可选的密钥告警存在，录制前也不要关闭它。

## 7. 运行录制 preflight

回到本地克隆：

```bash
npm run demo:preflight -- --repo <owner>/<repository> --confirm-copilot
```

解决每一项失败。在最后一行输出以下内容之前不要录制：

```text
READY TO RECORD
```

关于 Generic patterns 不可用或演示密钥告警缺失的 `[INFO]` 消息不阻断录制。

关闭无关的标签页和通知。保持以下页面打开：

- 仓库的 **Issues**、**Pull requests**、**Actions** 与安全页面；
- 初始的 Pages 站点；
- GitHub Copilot 应用的 **My work** 视图；
- `PRESENTER-RUNSHEET.md`。

## 8. 录制 Beat 0 —— 从 issue 到拉取请求

创建 issue 本身并不会创建 PR。在本流程中，issue 提供需求，Copilot 应用会话实现需求，
**Create PR** 发表变更。审查与 Agent Merge 期间保持同一个实现会话打开。

1. 在一次性仓库中，选择 **Issues → New issue**。
2. 对 **Feature request** 模板选择 **Get started**。
3. 如有需要，翻译 issue 标题与课程正文。
4. 不要翻译文件名、命令、依赖名称或验收标准。
5. 选择 **Create** 或 **Submit new issue**。
6. 在 Copilot 应用中打开 **My work**，找到该 issue 并选择 **New session**。使用本地
   worktree 实现会话，使默认分支保持不变。
7. 要求会话实现该 issue：

   ```text
   Implement this issue's lesson and feedback-widget changes. Copy .github/demo/deploy.yml
   to .github/workflows/deploy.yml unchanged. Keep dependencies and the lockfile unchanged,
   and preserve the seeded workflow and feedback-validation findings for the review exercise.
   Stop when the diff is ready for inspection. Do not commit, push, create a PR, or merge yet.
   If dependency installation is blocked by this host's registry, report it; do not change
   package versions or registry settings to work around it.
   ```

8. 检查实现 diff。不要为该 issue 再开第二个实现会话。
9. 打开 **Create PR** 旁的箭头，指出 **Agent merge**。暂时保持选中 **Create PR**：
   观众必须先看到审查，才能看到功能被允许合并。
10. 点击主按钮 **Create PR**，按其确认提示发布一个非草稿 PR。确保其描述包含
    `Closes #<issue-number>`。
11. 保持原始实现会话打开。可用浏览器或其 PR 面板展示新 PR，但到 Beat 2 时回到
    这个会话。

**预期结果：** 拉取请求更新课程/组件代码，并通过复制 `.github/demo/deploy.yml` 新增
`.github/workflows/deploy.yml`。

继续之前，检查 **Files changed**。如果 `package.json` 或 `package-lock.json` 有变化，
告诉 Copilot：

```text
Revert all changes to package.json and package-lock.json. Do not change dependencies.
```

等待修正提交，并以 GitHub 上绿色的 `build` 与 `audit` 检查为准。本地注册表失败不算
本地构建成功；以 GitHub 的实际检查结果为证据。

**替代路径，非 Agent Merge 主流程：** 在 GitHub 上把 issue 指派给云端 coding agent 也能
产出 PR。不要对同一个 issue 既指派又另起实现会话。把云端创建的 PR 在新审查会话中打开，
不保证出现相同的 Agent Merge 控件；如要选用该交接方式，请单独排练。

GitHub 参考文档：
[Managing issues and pull requests with the GitHub Copilot app](https://docs.github.com/en/copilot/how-tos/github-copilot-app/managing-issues-and-pull-requests)。

## 9. 录制 Beat 1 —— Copilot Code Review

1. 在 GitHub 上打开 Copilot 创建的拉取请求。
2. 展示绿色的构建检查与仅作参考的依赖审计。
3. 等待自动 Copilot 审查。
4. 如果没有出现审查，在右侧栏打开 **Reviewers**，手动请求 **Copilot**。
5. 打开 **Files changed**，展示行内发现。

预期的发现是：

- Actions 用标签而非完整提交 SHA 固定；
- `permissions: write-all`；
- `src/lib/feedback.js` 缺少输入校验。

Copilot 的措辞可能不同。重要的是风险和涉及的代码行，而非确切文字。如果缺一条发现，
请求一次重新审查。录制期间不要反复重跑审查。

GitHub 参考文档：
[Using GitHub Copilot code review](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/request-a-code-review/use-code-review)。

## 10. 录制 Beat 2 —— 处理审查并完成 Agent Merge

1. 回到第 8 步的**原始功能实现会话**，而不是新的 PR 审查会话。确认其关联 PR 是功能 PR，
   不是预备好的 Dependabot PR。
2. 在 PR 面板或浏览器中展示 Copilot 审查发现。
3. 在原始实现会话中输入：

   ```text
   Address all Copilot Code Review findings. Pin Actions to full commit SHAs, replace write-all
   with least-privilege Pages permissions, and validate feedback input. Do not change dependencies.
   ```

4. 检查产生的 diff，让会话把修复提交并推送到同一个 PR。保持依赖不变。
5. 确认针对最新提交，**Pull request checks**（`build`）与 **Dependency policy**（`audit`）
   均已开始或已通过。不要用旧提交的绿色结果充数。
6. 打开 **Create PR** 或当前 PR 操作旁的下拉菜单，选择 **Agent merge**。指出主按钮的
   标签变成了 **Agent merge**。
7. 点击主按钮 **Agent merge** 启动它。只选菜单项并不等于启动操作。会话必须管理已有的
   关联功能 PR，而不是创建重复的 PR。
8. 展示 Agent Merge 操作权限。按需允许 **Address reviews**、**Fix CI failures** 与
   **Resolve conflicts**。只有在预期审查修复已检查完毕、准备让功能落地之后，才允许
   **Merge pull request**。如果启动时 UI 已经允许合并，先完成检查再启动。
9. 让会话保持可见，展示它检查 PR 并在 GitHub 允许时合并的过程。展示 PR 的 **Merged**
   状态，然后切换到部署运行。

**预期结果：** Agent Merge 在必需检查通过后落地已审查的功能 PR。

说：*"这是 Copilot 应用里的 Agent Merge，不是 GitHub 的 Enable auto-merge 按钮。agent
会检查 PR 并处理被允许的后续工作；GitHub 仍在强制执行必需检查。"*

检查全绿并不妨碍 Agent Merge 工作。无需和待运行检查的窗口赛跑。如果一切条件已满足，
合并可能很快发生。不要静默地用原生 **Enable auto-merge** 或手动合并替代本环节，却把它
说成 Agent Merge。如果应用控件不可用，使用预备好的 Agent Merge 录制并注明是替代方案。

GitHub 参考文档：
[Managing issues and pull requests with the GitHub Copilot app](https://docs.github.com/en/copilot/how-tos/github-copilot-app/managing-issues-and-pull-requests)。
[Agent Merge workshop](https://awesome-copilot.github.com/learning-hub/copilot-workshops/app/6-agent-merge/)
展示了下拉菜单、启动按钮与合并权限。

## 11. 录制 Beat 3 —— 生产门禁阻断部署

1. 在 GitHub 上打开 **Actions**。
2. 打开由合并到 `main` 触发的新 **Build and deploy** 运行。
3. 打开构建任务。
4. 展示 `npm ci` 完成。
5. 展示 `node scripts/check-audit-state.mjs clean` 步骤失败，因为生产要求审计干净。
6. 展示站点构建步骤被跳过、部署任务未运行。

**预期结果：** 能正常工作的应用代码无法绕过供应链策略。

不要重跑失败的工作流；它应作为被阻断状态的证据保留。

## 12. 录制 Beat 4 —— 安全发现与依赖修复

1. 打开 **Security** 或 **Security and quality**。
2. 在 **Dependabot** 下展示 `marked@0.3.19` 的公告。
3. 打开 **Settings → Security and quality → Advanced Security**。
4. 在 **Secret Protection** 下展示密钥扫描已启用。
5. 展示 **Push protection** 已启用，并说明它会在受支持的提供商密钥进入仓库之前于推送时
   拦截。
6. 如果显示了 **Generic patterns**，说明它将检测范围扩展到提供商密钥之外。如果没有显示，
   说明可用性因账户而异，而核心保护仍然启用。
7. 可选：如果 Generic patterns 产生了训练告警，打开 **Secret scanning**，展示 bearer 头
   测试数据。明确说明它是一个无实际功能的测试值。
8. 打开 **Pull requests**，选择预备好的 Dependabot `marked` 更新。
9. 展示其绿色的 **Dependency policy** 与 **Pull request checks**。
10. 审查依赖 diff，然后在必需检查通过后于 GitHub 上合并这个预备好的 PR。不要在功能
    部署演示出门禁失败之前合并它。
11. 说明修复由 Dependabot 生成、由 GitHub 检查。Agent Merge 已在 Beat 2 的功能 PR 上
    明确演示；这是一次独立的依赖合并。

不要假设新的 Dependabot 审查会话会暴露 Agent Merge：它可能显示 **Submit review**。只有在
单独排练并验证过该路径时，才对该 PR 使用 Agent Merge。绿色的 PR 可能直接提供立即合并，
而不是原生 **Enable auto-merge**。

**预期结果：** 依赖更新由独立生成、经检查后合并，且没有削弱生产门禁。无论 Generic
patterns 是否可用，观众都能看到 Secret Protection 与 Push protection 的配置位置。

## 13. 录制 Beat 5 —— 成功部署

1. 回到 **Actions**。
2. 打开由 Dependabot 合并触发的新 **Build and deploy** 运行。
3. 展示构建、干净的依赖策略与部署全部成功完成。
4. 打开工作流或仓库 **Deployments** 区块中的部署 URL。
5. 导航到新增或更新的课程页面。
6. 提交一条包含主题（topic）的反馈。
7. 展示该条反馈渲染在页面上。

**预期结果：** 修复后的依赖状态到达 GitHub Pages，完成的功能端到端可用。

以这句话收尾：

```text
Issue → AI draft → AI review → automated fix → security gate → deployment
```

## 14. 录制之后

1. 在录制内容复审完成之前保留一次性仓库。
2. 将仓库 URL、功能 PR、Dependabot PR、失败的工作流运行、成功的工作流运行以及 Pages
   URL 与录制笔记一起存档。
3. 不要为了重拍而重置该仓库。
4. 重拍或换一种语言时，从模板创建新仓库，从第 2 步重新开始。

规范模板保持不变，随时可供下一位演示者使用。

## 故障排查

| 问题 | 处理 |
|---|---|
| `gh` 无法修改工作流 | 运行 `gh auth refresh -h github.com -s repo,workflow,read:org` |
| bootstrap 目标指向规范仓库 | 停止，重新创建/克隆一个一次性模板仓库 |
| 初始化找不到其工作流 | 确认模板仓库使用 `main` 且包含 `.github/workflows/initialize-demo.yml` |
| 出现多个 Dependabot PR | 不要录制；创建全新的模板仓库并重跑 preflight |
| 缺少 `marked` PR | 等待 GitHub 扫描完成，然后重跑 preflight |
| Generic patterns 不可用 | 继续，省略可选的密钥扫描环节 |
| Generic patterns 已启用但告警缺失 | preflight 将其报告为信息项后即可继续 |
| **Assignees** 中没有 Copilot | 确认 coding agent 许可、功能设置、组织策略与仓库访问权限 |
| 没有自动审查 | 在 PR 的 **Reviewers** 侧栏请求一次 Copilot |
| 会话显示 **Submit review** 而非 Agent Merge | 回到原始功能实现会话；新的 Dependabot 或云端 PR 审查会话不是已演示的创作路径 |
| 空会话没有 **Create PR** | 先实现功能并检查其 diff，再查看实现会话的 PR 操作下拉菜单 |
| 实现会话中没有 Agent Merge | 确认应用版本、仓库访问权限与 Copilot 计划；使用预备好的 Agent Merge 录制，不要把手动合并说成 Agent Merge |
| 没有 **Enable auto-merge** | 确认 PR 非草稿、**Allow auto-merge** 已启用、激活的规则集要求 `build` 与 `audit`；若一切已满足，直接使用立即合并 |
| `audit` 停留在 **Expected** 且无运行 | 使用更新后的模板工作流移除 **Dependency policy** 的依赖路径过滤，然后推送新提交触发两个检查；重跑被跳过的工作流不够 |
| preflight 报告合并规则缺失或不正确 | 用 `--apply --skip-deploy` 重跑 bootstrap，然后核对上文规则集；不要绕过它 |
| 功能 PR 更改了依赖 | 要求 Copilot 还原 `package.json` 与 `package-lock.json` |
| 功能 PR 检查失败 | 录制前诊断清楚；不要绕过必需检查 |
| 最终部署失败 | 保留失败的运行，使用已批准的兜底录制 |
