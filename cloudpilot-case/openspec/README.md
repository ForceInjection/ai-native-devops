# openspec · CloudPilot CLI 工作区

> 本目录是 **OpenSpec CLI 实况工作区**（`openspec init` 生成，目录名为 CLI 固定约定），不是 P4 阶段工件。

## 它是什么

`04-openspec/`（评估基准 Golden Set，只读）的 **CLI 实况副本 + 演进容器**：

| 工件 | 说明 |
| :--- | :--- |
| `project.md` | 项目上下文（数据契约/约定/已知 gap/术语表），`/opsx:*` 每次执行必读 |
| `specs/` | CLI 校验的主基线（3 个 capability，迁移自 04-openspec 并转换格式） |
| `contracts/` | 常驻契约工件（SDD book-code 惯例，被 design/tasks 共同引用） |
| `adrs/` | 常驻架构决策记录（D1-D4 提升 + IV-N/B-N 编号溯源规则） |
| `changes/archive/` | 已归档的 change（生命周期留痕，半年后可回读原始动机） |

## 怎么用

```bash
cd cloudpilot-case        # 必须在案例目录执行（CLI 向上找最近的 openspec/）
openspec validate --all   # 校验主基线 + changes
openspec new change <name>        # 开始一次新变更
openspec archive <name> --yes     # 归档：delta 合并进 specs/，change 移入 changes/archive/
```

实践记录见 [`ai-native-devops/sdd-paradigms-synthesis.md`](../../ai-native-devops/sdd-paradigms-synthesis.md)（§4 实践实录）。

## 与 04-openspec 的分工

- **04-openspec/**：P4 阶段产出，评估基准，AI 生成的参考形态（中文 frontmatter、`## ADDED Requirements` 头），只读。
- **本目录**：工具链运行时状态（CLI 规范格式），演进（新 change、编号补齐、契约/决策工件）只发生在这里。
- 主基线若需跟随 04-openspec 修订：复制 + 转换后 `openspec validate --specs` 校验，不做原地修改。
