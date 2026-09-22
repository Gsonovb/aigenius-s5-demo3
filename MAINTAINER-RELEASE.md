# 演示套件发布流程

本流程用于发布规范（canonical）初始状态。面向维护者，而非演示者。

## 发布契约

`demo-kit.json` 是发布标识符、运行时、依赖版本、预期审计状态、Dependabot 行为以及可选
密钥扫描信号的权威来源。仅在一次完整重新认证的发布中才更新它。

规范仓库始终发布**初始状态**。完成状态在每个一次性演示仓库中产生，并保留在该仓库的
拉取请求与历史记录中。

## 准备

1. 确认 `.github/workflows/deploy.yml` 不存在。
2. 确认 `.github/demo/deploy.yml` 存在。
3. 审查全部变更并提交到 `main`。
4. 运行 `npm ci`。
5. 在干净的检出目录中运行 `npm run demo:preflight -- --local-only`。
6. 推送 `main`，然后重新运行本地 preflight。

只要 preflight 报告本地失败，就不要发布。
在全新模板仓库中成功运行的 **Initialize demo site** 是权威的公共 npm 注册表校验；
本地环境可以使用经批准的注册表代理。

更改 bootstrap、合并规则或依赖策略后，在认证之前运行
`node --test scripts/demo-setup.test.mjs`。这些离线回归测试覆盖规则集漂移、刻意设置的
初始状态、干净的后续 PR，以及审计失败报告。

## 发布初始引用

创建并推送与 `releaseId` 匹配、追加 `-start` 的附注标签：

```bash
git tag -a demo-2026.09-start -m "Ship with AI demo start state 2026.09"
git push origin demo-2026.09-start
```

只有在标签与默认分支都指向已认证的初始状态之后，才在 **Settings → General** 下启用
**Template repository**（模板仓库）。

## 认证模板

1. 通过 **Use this template** 创建新的公开仓库。
2. 克隆新仓库并运行 `npm ci`。
3. 先以 dry-run 模式运行 bootstrap，再以 `--apply` 应用。
4. 等待 Dependabot。如果 Generic patterns 可用，同时等待可选的密钥告警。
5. 要求完整 preflight 输出 `READY TO RECORD`。
6. 完整执行演示者流程表两遍。
7. 保留成功的临时仓库，作为本次发布的完成状态参照。
8. 在 GitHub release 说明中记录初始标签、完成仓库 URL、最终提交 SHA、Pages URL 以及
   各环节耗时。

如果认证过程暴露失败，请禁用 **Template repository**，修正规范初始状态，递增发布
标识符，并重走整个流程。绝不移动已发布的初始标签。
