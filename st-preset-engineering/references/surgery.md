# surgery.md — S4 开手术（生成器模式）

> **何时读**：S4；**动手改 JSON 前必读**。
> **为什么**：1MB JSON 手改必错且不可审计。生成器 + 三断言让"改了什么、没改什么"由机器说了算——实战中 186 条目的大手术与 1 条目的小修改用同一套纪律。

## 生成器三断言（模板骨架）

```python
import json, io, copy, hashlib
doc = copy.deepcopy(json.load(io.open(SRC, encoding="utf-8")))
changes = []  # 枚举预期变更

def edit_once(idx_or_name, old, new, label):
    """断言1 唯一性：目标串在该条目恰出现 1 次，否则中止"""
    c = doc["prompts"][idx]["content"]
    assert c.count(old) == 1, f"{label} 命中 {c.count(old)} 次（预期1），中止"
    doc["prompts"][idx]["content"] = c.replace(old, new)
    changes.append(label)

# ……全部编辑……

# 断言2 枚举：变更条目集合 == 预期集合
diff = [i for i,(a,b) in enumerate(zip(src["prompts"], doc["prompts"]))
        if json.dumps(a,sort_keys=True)!=json.dumps(b,sort_keys=True)]
assert diff == sorted(expected_idx), f"变更条目不符: {diff}"

# 断言3 等值：除枚举字段外逐字节一致（顶层/扩展层同理）
for k in doc:
    if k not in ALLOWED_CHANGED_KEYS:
        assert json.dumps(src[k],sort_keys=True)==json.dumps(doc[k],sort_keys=True), f"{k} 被意外改动"

json.dump(doc, open(OUT,"w",encoding="utf-8",newline="\n"), ensure_ascii=False, indent=4)
```

## 断言先校准（教训）

- "逐字节一致"类断言**先在真实差异上校准**：派生产物的占位符编号、重排序等噪声要先归一化，否则验收永远失败或永远虚过。
- 装配对账的噪声处理：把 `«BLOCK:NNN»`/`«NAME:…»` 归一化成 `«BLOCK:X»` 再逐消息 diff。
- emoji 锚点必失败：**定位条目用数字/ASCII 子串**（"1600-2500"），不用 emoji 全名。

## 五类修改的操作要点

| 类型 | 要点 |
|---|---|
| 翻开关 | **成对操作**（❗1+❗2 一起翻）；枚举**全部**成员的翻转（含把默认 ON 的翻 OFF——漏枚举=双模板事故）；翻转后组内 ON 恰 1。**换档类开关加一步**：全库扫其他 ON 条目是否写死旧档位数值（含中文数字，如"正文不足一千二百字"这类硬下限）——这类语义冲突 L1 查不出（组规则全绿），只有正文扫描或 L3 能抓到 |
| 改条目正文 | 先做**契约对侧扫描**（标签/格式字面量在正则与脚本里的消费方）；一处编辑一个 `edit_once`；改标签 = 同步全部消费方 |
| 加条目 | **键集与 role 都照抄同族相邻条目**（同族实存条目是键集母本，勿照抄静态模板——模板可能缺 attach_*/injection_order 等实存键）；新条目**必须进 order**（`{"identifier":…, "enabled":false}`），插入点=同族区域末尾；新条目默认 OFF |
| 删条目 | prompts[] 与 order **一起删**（否则幽灵引用）；删前查脚本/正则对它的引用；删带 setvar 的停用条目反而是排雷（僵尸地雷） |
| 改正则 | 双 False 禁令（拆显示侧/提示词侧）；**多层拷贝同步**（原生层+扩展绑定层）；pattern 字面量与条目正文同时改 |

## 手术前后

- **改前**：S0 只读校验（前）；确认工作副本非源档；敏感产物落仓库外。
- **改后**：S0 校验（后，确认源没被碰）→ S5 三层验证 → sha256 记入交付说明。
- **多轮手术**：每轮 S4→S5 结束**回 S2 重新拿真值**——上一轮已改变装配行为，旧真值作废。

## 高危区清单（动手前对照）

- [ ] 停用条目里的 `{{setvar::主槽::}}`——启用即清空累积槽（僵尸地雷），删除或摘除 setvar
- [ ] assistant 角色条目末尾 = 伪预填充续写点——**不在续写点之后追加指令**（会被当成模型自己的话）
- [ ] Main Prompt 类初始化条目里的 setvar——删死槽初始化是安全的（渲染恒空），但要先证"恒空"（初始值为空串且无人写）
- [ ] 常开锚点（🔒）——动它影响所有路线；改动必须过 L3
- [ ] 带 UI 的脚本引用的条目名/组号——改名/删除前全库扫引用
