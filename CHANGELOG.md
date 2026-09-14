# Changelog

版本号与 `st-preset-engineering/SKILL.md` frontmatter 的 `metadata.version` 保持同步。

## [1.1.0] — 2026-09-14

### 新增

- 英文版 skill `st-preset-engineering-en/`（SKILL.md + 11 篇 references 全量英译），与中文版骨架、纪律、版本号完全同步
- 英文版 `README.en.md`，与中文 README 内容对齐；两份 README 顶部互设语言切换链接
- `examples/preset-profile.sample.json`：实例 profile 脱敏模板（预设名、家族前缀、内部目录、路径均以 `<…>` 占位符呈现，任何预设家族可直接套用）

### 安全与隐私

- 全库隐私审计与历史重建：移除对私有维护项目（仓库名 / 内部治理文档章节 / 内部目录名 / 实例私有命名）的全部引用，改为平台通用表述；重建后的 git 历史不含任何私有项目关联信息
- SKILL.md 对「受治理仓库」的引用改为通用约定（以目标仓库自身协作文档为准），不指向任何具体私有仓库

### 变更

- 鸣谢收敛至 README 末尾单节；正文与 skill 文件不再提及具体预设家族名

## [1.0.0] — 2026-09-13

首次公开发布。

### 新增

- skill 本体 12 文件：`SKILL.md`（七步骨架 S0–S6 + 步骤路由表 + 三条不可违反纪律 + 复杂度自适应 + 反跑偏自查 + 安全边界声明）与 11 篇 references（red-lines / autopsy / truth-sources / diagnosis / surgery / verification / handover / pitfalls / best-practices / profile-spec / toolmap）
- `examples/`：真实实例 profile 的脱敏副本（含模型分支配对 / 活槽 / 标签契约 / heavy 复杂度分级示例）
- 发布合规：SKILL.md「安全边界声明」节（文件访问 / 网络 / 不做的操作 / 无密钥）与 `license` / `compatibility` / `metadata.version` 元数据
- `README.md`：功能说明、特色安全措施（敏感预设的安全处理与截断风险控制）、快速开始、系统要求、合规对照、FAQ

### 设计边界

- 配套工具带（结构档案 / 脱敏管线 / mock 端点 / 离线装配 / 审计等 Python 工具）**不随本仓库分发**：脱敏管线依赖敏感词表、部分注册表含实例数据；skill 通过 `profile.tool_dir` 引用，缺失时明确报告、不静默跳过
- 对抗类档位语义默认不登记进 profile（条目名自含语义，少登记少敏感面）
