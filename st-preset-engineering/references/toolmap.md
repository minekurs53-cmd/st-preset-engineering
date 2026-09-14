# toolmap.md — 工具调用卡

> **何时读**：任何时候要调工具前。
> **通用性标注**：✅=任意预设可用；🔶=可用但输出含实例专属判读；⛔=依赖实例数据，换家族需替换。
> 所有路径相对 `profile.tool_dir`。**工具被拒绝时先看是不是自己放错位置，不绕过工具。**

## 01_source_manifest.py — 源只读校验 ✅

- 命令：`python 01_source_manifest.py`
- 期望：`[verify] OK — 全部源文件哈希与清单一致`
- 失效判据：任何 `[FAIL]`/哈希不一致 → **停止一切手术**，先查谁改了源
- 时机：S0（改前改后各一次）；其他预设用自己的 hash_baseline（sha256 前 16 位）

## 02_sanitize_nsfw.py — 脱敏器 ✅（词表可换）

| 子命令 | 用途 | 注意 |
|---|---|---|
| `classify --in <预设> --out <CSV>` | 词表分类（SAFE/MIXED/OVERRIDE/EXPLICIT） | CSV 含原文 → 工具强制只写敏感目录 |
| `apply --in --out --map --public-json` | 产隔离档 + PUBLIC 档 | PUBLIC 档=词表命中 0=可外发 |
| `verify-structure --src --san --map` | 验证"改动完全局限于被映射槽位" | 手术后跑，防误伤骨架 |
| `explain` | 诊断哪个词条触发了脱敏 | **调词表前必跑**——实测一个通用词会误判 32% 内容 |
| `restore` | 从脱敏副本逐字节还原 | 校验管线完整性 |

## 03_p0_pipeline.py — 一键脱敏+全部验收 ✅

- 命令：`python 03_p0_pipeline.py [--all]`
- 期望：`总判定：全部通过 ✅`
- 动过脱敏管线/词表后必跑

## p1_evidence.py — 结构档案（S1 核心）✅

```bash
python p1_evidence.py report --preset <任意预设.json>   # 单版本档案（孤儿/死槽/分组/正则/脚本）
python p1_evidence.py report --target <已登记版本>       # 已登记源版本 ⛔（target 名单是实例数据）
python p1_evidence.py drift                             # 跨版本漂移（多版本时）
```
- 读法见 autopsy.md §1；五项风险判据见 verification.md L1
- 限度：分组判读依赖记号+阶段标记——陌生记号体系降级为参考，语义问用户

## p5_groups_build.py — 分组注册表自检 ⛔（registry 是实例数据）

- 命令：`python p5_groups_build.py --check`（只验不写；不带 --check 会重写文件）
- 期望：`自检 PASS：N 组…`；注册表与 p1 报告 §3 逐组对账
- 通用场景：换预设家族时不复用此 registry，用 profile.model_branches + 组互斥语义替代

## p5_agent_eval.py — 离线装配 + 断言 🔶

```bash
python p5_agent_eval.py assemble --config default --pub <PUBLIC档> --tag <名> --out-dir <仓库外目录>
python p5_agent_eval.py check <回复.txt> --template <已登记模板名> --turn N
```
- `assemble`：✅ 任意 PUBLIC 档；产出消息序列供改前改后逐消息 diff
- `check`：⛔ **模板是实例专属**（字数区间/标签字面/选项枚举），且只适用**标签型思维链**渠道——reasoning 通道模型（响应含 reasoning_content）的输出别用它打分

## p2_mock_llm.py — mock 端点（S2 请求侧真值）✅

- 命令：`python p2_mock_llm.py --port 9999`
- 配套：酒馆 → Custom (OpenAI-compatible) → `http://127.0.0.1:9999/v1` + 任意 key + 模型名任意 → 连接 → 发消息 → 读 dump
- dump 落仓库外临时目录；用临时聊天、用后零残留
- 证明范围：装配层；**不等于用户实跑时模型所见**（脚本层会改写请求历史）

## 04_audit_for_git.py — 提交前审计 ✅

- 命令：`python 04_audit_for_git.py`
- 读法：**命中列**（第一列）——待入库文件必须 0；启发式标记列（第三列）是描述性用词的粗筛（"覆盖/规则/合规"这类词组合会触发），既有入库文档也带 1，**以命中列为准**
- 提交前必跑；新文件命中 >0 → 化名/脱敏，不许硬交
