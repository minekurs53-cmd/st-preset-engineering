# profile-spec.md — preset-profile.json 字段规范

> **何时读**：第 0 步建/校 profile 时。
> **原则**：profile 是 skill 的实例参数注入点。骨架（SKILL.md）跨预设不变；**换预设家族 = 换 profile**。profile 里只放"这台机器上可验证的事实"，每条路径/命令必须实际可跑。

## 查找顺序与失效处理

1. 当前工作目录 → 2. 仓库根 → 3. `~/.preset-profiles/<预设名>.json`
- 都没有：按下面"最小 profile"先建骨架字段，S0/S1 跑完补齐 structure 节
- profile 引用的文件/命令跑不通：**明确报告哪条失效**，不许静默跳过、不许猜测替代

**新建 profile 写到哪**（按会话性质选，并在交付中声明）：
- 项目内长期维护该预设 → 仓库根 `preset-profile.json`（入库，随仓库版本化）
- 跨项目/临时分析 → `~/.preset-profiles/<预设名>.json`（不入任何仓库）
- 会话被禁止写仓库 → 输出目录，并在交付中注明"正式位置应为 X，本次按约束落临时目录"

## 完整 schema（带注释）

```jsonc
{
  "preset": {
    "name": "<预设名>-重铸版",                       // 展示名
    "source_dir_readonly": "<源预设目录>/",          // 只读源（可为 null：无源档场景）
    "working_copy": "<工作副本路径>.json",           // 当前工作副本（含敏感正文→仓库外）
    "verify_command": "python <tool_dir>/01_source_manifest.py",  // 只读校验命令；期望输出 [verify] OK
    "tool_dir": "<tool_dir>",                       // 工具目录（相对仓库根或绝对路径）
    "sensitive_strategy": "placeholder-pipeline",   // placeholder-pipeline | quarantine | none
    "outbound_files_pattern": "*.public.*",         // 可外发档的判定模式（词表命中 0）
    "hash_baseline": {"file": "源文件名", "sha256_16": "e91b1ef8ac231f30"} // 冒烟/外来档的只读基线
  },
  "structure": {
    "switch_layer": "prompt_order",                 // 通用事实，不用改
    "order_blocks": [100001],                       // character_id 列表（有的预设多块）
    "group_notation": {"❗": "组内必选一", "❔": "可选零或一", "❕": "自由多选", "🆎": "同开同关", "🔒": "常开锚点"},
    // ↑ 换预设时必须重填！陌生记号 → 中止问用户，不猜测（示例中的记号是某预设作者私有的）
    "zone_marker_style": "〈…〉标题条目",            // 阶段边界怎么识别；无标记的预设写 "none"
    "branch_zone": "〈填充功能相关〉",                // 模型分支所在阶段名（没有分支预设写 null）
    "model_branches": [ /* 见下 */ ],
    "live_slots": ["<家族前缀>_前置处理", "…"],      // 活槽名单（来自 S1 档案槽位表）
    "tag_contracts": [ /* 见下 */ ],
    "default_switch_state": "GLM 路线 B：❗1 GLM用尾部 + ❗2📅非预填充输出模板 ON；抗缺陷 ON / 抗平淡 OFF"
  },
  "adaptation": {
    "user_models": ["glm-5.3", "deepseek"],         // 用户实际在用的模型（真实使用态优先于出厂默认）
    "matrix": "<方法论文档>/model-capability-matrix.json",
    "sop_adapt": "<方法论文档>/新模型适配SOP.md",
    "sop_feature": "<方法论文档>/新功能加入SOP.md"
  },
  "complexity": {
    "entries": 186, "scripts": 4, "regexes": 15,
    "extension_blocks": ["regex_scripts", "tavern_helper", "SPreset"],
    "level": "heavy"                                // light | medium | heavy，判据见 autopsy.md
  }
}
```

## model_branches 条目格式

```jsonc
{
  "model": "GLM",
  "tail": "❗1 GLM用尾部",              // ❗1 尾部输入条目名
  "template": "❗2📅非预填充输出模板",   // ❗2 输出模板条目名（可与别家共用）
  "slot_written": "<家族前缀>_思维链_非预填充定位",  // 本路线 ❗1 setvar 的定位槽
  "slot_read_by_template": null,        // 模板 getvar 的定位槽；**必须与 slot_written 同槽或为 null**——
                                        //  不同槽 = 配对断裂（实测教训：断裂无任何报错，只表现为输出降级），建档时必查
  "extra_params": "聊天补全来源须为对应服务商"  // 模型侧前置条件（没有就省略）
}
```

## tag_contracts 条目格式（S4 改正文前的必查清单）

```jsonc
[
  {"literal": "<thinking>", "consumers": ["去多余思维链提示词(正则,只认thinking)"]},
  {"literal": "</(?:think|thinking)>", "consumers": ["思维链美化(正则,双兼容)"]},
  {"literal": "<details><summary>(实时|小)总结", "consumers": ["10楼以上只留小总结(正则,S9:改一侧=整楼清空)"]}
]
```
生成方法：对预设全库扫"正则 find/replaceString 里的字面量"与"条目正文里的格式标记"，交集即契约。UI 脚本也算消费方（按条目名/组号索引的引用）。

## 最小 profile（冷启动模板）

```jsonc
{
  "preset": {"name": "<文件名>", "source_dir_readonly": null, "working_copy": "<用户给的路径>",
             "verify_command": null, "tool_dir": null, "sensitive_strategy": "none",
             "outbound_files_pattern": null, "hash_baseline": {"file": "<路径>", "sha256_16": "<S0 算>"}},
  "structure": {"switch_layer": "prompt_order", "order_blocks": [], "group_notation": {},
                "zone_marker_style": "unknown", "branch_zone": null, "model_branches": [],
                "live_slots": [], "tag_contracts": [], "default_switch_state": "unknown"},
  "adaptation": {"user_models": [], "matrix": null, "sop_adapt": null, "sop_feature": null},
  "complexity": {"entries": 0, "scripts": 0, "regexes": 0, "extension_blocks": [], "level": "unknown"}
}
```
S0 填 hash_baseline；S1 跑完 p1 --preset 填 structure/complexity 大半；**group_notation 与 zone_marker_style 确定不了就当面问用户**。

## 验收规则

- profile 里出现 `unknown` 的字段 ≠ 失败，但**依赖该字段的步骤必须先把它消解**（跑工具或问用户）
- 敏感策略 `quarantine`：工作副本放 gitignore 目录；`none`：无敏感内容，可直接入库
- 破甲/对抗类档位语义**默认不登记**（条目名自含语义；少登记少敏感面）——除非用户明确要求
