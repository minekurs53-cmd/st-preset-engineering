# profile-spec.md — preset-profile.json 字段规范

> **何时读**：第 0 步建/校 profile 时。
> **原则**：profile 是 skill 的实例参数注入点。骨架（SKILL.md）跨预设不变；**换预设家族 = 换 profile**。profile 里只放"这台机器上可验证的事实"，每条路径/命令必须实际可跑。

## 查找顺序与失效处理

1. 当前工作目录 → 2. 仓库根 → 3. `~/.preset-profiles/<预设名>.json`
- 都没有：按下面"最小 profile"先建骨架字段，S0/S1 跑完补齐 structure 节。**时序许可**：`verify_command`/`tool_dir` 在骨架期允许先占位（填计划路径）——首建时刻工具还不存在是正常的；对应工具建成并通过自测后立即回填，那时起"每条命令可跑"生效
- profile 引用的文件/命令跑不通：**明确报告哪条失效**，不许静默跳过。出路不是找一个行为不一致的替代品，而是按 `toolmap.md` 的建工具方法论与对应契约卡在本地实现等价工具，自测通过后登记进 profile 再继续
- `tool_dir` 为 null 不是失败：先按最小流程推进，到需要某张契约卡的步骤时再按卡建工具（方法论第 5 条：先过自测再上真实数据）

**新建 profile 写到哪**（按会话性质选，并在交付中声明）：
- 项目内长期维护该预设 → 仓库根 `preset-profile.json`（入库，随仓库版本化）
- 跨项目/临时分析 → `~/.preset-profiles/<预设名>.json`（不入任何仓库）
- 会话被禁止写仓库 → 输出目录，并在交付中注明"正式位置应为 X，本次按约束落临时目录"
- **多预设独立沙盒**（一次会话测多个预设、互不污染冷启动）→ 每个预设一个工作文件夹，profile 落各自文件夹内——前三种按"一个 profile 跟随一个预设"理解即可，沙盒只是把文件夹当隔离单元

## 完整 schema（带注释）

```jsonc
{
  "preset": {
    "name": "<预设名>-重铸版",                       // 展示名
    "source_dir_readonly": "<源预设目录>/",          // 只读源（可为 null：无源档场景）
    "working_copy": "<工作副本路径>.json",           // 当前工作副本（含敏感正文→仓库外）
    "verify_command": "python <tool_dir>/01_source_manifest.py",  // 只读校验命令；期望输出 [verify] OK
    "tool_dir": "<tool_dir>",                       // 工具目录（相对仓库根或绝对路径）；null = 尚无本地工具，按 toolmap 契约建后登记
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
    // ↑ zone_marker_style 只是展示口径；**真正驱动工具的是下面这些可选正则字段（数据驱动，样式无关）**。
    //   实测六种真实样式：〈…〉标题条目 / 成对边界条目（——X开始——…——X结束——）/ 同名成对边界（toggle 型）/ emoji+分类名前缀 / HTML 标签成对条目（`<background>…</background>`）/ "----X" 连字符标题。
    //   内置样式识别规则不可信（实测首版只认一种，其余全漏）——profile 正则优先于工具内置。
    "zone_title_re": "^----[^\\s-]+",                // 可选：标题条目的识别正则（样式无关）
    "zone_pair_start_re": null,                      // 可选：成对边界——开始条目正则（无则 null）
    "zone_pair_end_re": null,                        // 可选：成对边界——结束条目正则
    //   ↑ 同名成对边界（toggle 型）：start/end 填**相同**正则时按"同名二次出现=关闭"处理（实测：边界条目起止同名 ——X—— 两次出现、首次开启二次关闭；不同名边界仍按顺序开启语义）
    "zone_category_re": null,                        // 可选：emoji+分类名前缀样式——逐条目提取分类名的正则（样式无关）
    // ↑ 第 3 样式（emoji+分类名前缀）无 title/pair 字段可用，实测需自增此字段（`^[^︱丨]{1,6}[︱丨]\s*([^-\s丨︱]{1,10})`，捕获组 1 = 分类名）。
    //   ⚠️ 正则含捕获组时，**捕获组 1 = 区名**（如 `━━━━ X ━━━━` 分隔条目只要纯分类名 label，靠捕获组取）——title_re 同理。
    "block_rules": [                                 // 可选：数据驱动的组规则（工具按此机检，不猜语义）
      // {"kind": "count_in", "zone": "<阶段>", "field": "文风", "max": 2},
      // {"kind": "mutex", "members": ["«NAME:1|system»", "«NAME:2|system»", "«NAME:3|system»"]}
      //   mutex 规则的成员 = 同槽多写入者 × 名内标注（"(选一)" 等）× 记号聚类 三路证据的交集
    ],
    "branch_zone": "〈填充功能相关〉",                // 模型分支所在阶段名（没有分支预设写 null）
    "model_branches": [ /* 见下 */ ],
    "live_slots": ["<家族前缀>_前置处理", "…"],      // 活槽名单（来自 S1 档案槽位表）
    "extension_hookpoints": [                        // 可选：扩展挂点清单（autopsy §2 三问结论落此）
      // [{"block": "regex_scripts", "kind": "ST原生正则", "status": "effective",
      //   "note": "谁读/谁写/改它影响什么"}]
    ],
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
    "level": "heavy",                               // light | medium | heavy，判据见 autopsy.md
    "note": null                                    // 可选：判据与实质不符时注记（如"按字面 heavy，无脚本无 UI，实质≈medium"）
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
                "zone_marker_style": "unknown", "zone_title_re": null, "zone_pair_start_re": null,
                "zone_pair_end_re": null, "zone_category_re": null, "block_rules": [],
                "extension_hookpoints": [], "branch_zone": null, "model_branches": [],
                "live_slots": [], "tag_contracts": [], "default_switch_state": "unknown"},
  "adaptation": {"user_models": [], "matrix": null, "sop_adapt": null, "sop_feature": null},
  "complexity": {"entries": 0, "scripts": 0, "regexes": 0, "extension_blocks": [], "level": "unknown"}
}
```
S0 填 hash_baseline；S1 跑完 p1 --preset 填 structure/complexity 大半；**group_notation 与 zone_marker_style 确定不了就当面问用户**；用户不在场（异步任务）→ 登记"推定 + 待确认"继续其余流程，**不许猜测语义往下走**。

## 验收规则

- profile 里出现 `unknown` 的字段 ≠ 失败，但**依赖该字段的步骤必须先把它消解**（跑工具或问用户）
- 敏感策略 `quarantine`：工作副本放 gitignore 目录；`none`：无敏感内容，可直接入库
- 破甲/对抗类档位语义**默认不登记**（条目名自含语义；少登记少敏感面）——除非用户明确要求
