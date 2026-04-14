# 分支与发布流程

## 分支模型（示例：trunk-based）

| 分支 | 用途 | 保护规则 |
| --- | --- | --- |
| `main` | 可发布主线 | 必须通过 CI；禁止 force push |
| `feature/*` | 需求开发 | 经 Code Review 合并 |
| `hotfix/*` | 紧急修复 | 双审批 + 回归用例 |

## 提交信息

推荐 [Conventional Commits](https://www.conventionalcommits.org/)，便于生成 CHANGELOG。

## 发布节奏

- **版本号**：SemVer；破坏性变更升主版本。
- **制品**：容器镜像 tag 与 Git tag 一致；记录到发布说明。

## 回滚

说明回滚到上一镜像版本的操作步骤与数据迁移回滚策略（若有）。
