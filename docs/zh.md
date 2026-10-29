# Stat Checks — 属性掷骰对抗难度

Stat Checks 为剧本添加一条命令：`check`。它取角色的一个属性，加上骰子和加值，把总数与难度比较，再让故事按成功或失败继续。视觉小说用它说服卫兵，图块 RPG 用它撞开一扇锈门。属性本身来自 Stats Engine 和游戏的规则（RPG Plus 中是 Classic RPG Rules）。

## 在游戏里

RPG Plus 序章中，一位陌生女子坐在岔路口。其中一个选项直接写明它要做的检定——魅力对抗 10：

![黄昏的岔路口；问题"你要对她说什么？"和两个选项："客气地问路［魅力 10］"和"默默走过"。](assets/seer-choice.jpg)

做出选择后，掷骰结果只占一行：

![对话框中显示一行"[魅力：11 对 10] 成功"。](assets/check-line.jpg)

这里的主角是 1 级游荡者，魅力 8；d20 掷出 3，总数 11 对 10——成功。故事进入成功分支：陌生女子扔给主角一枚护符。

![成功分支："有礼貌。真稀奇。"她把一枚四叶草护符扔给你。](assets/check-branch.jpg)

如果总数低于 10，这一行会显示*失败*，故事则走另一个分支——她转过身去。

## 背后的剧本

出自 `rpg_plus_game/scenarios/rpg_plus/prologue.scenario`（有删节）：

```scenario
option "客气地问路［魅力 10］" -> :ask_way
option "默默走过" -> :leave

:ask_way
check cha 10 who=rpg_hero else=snub -> :gift

:gift
text "有礼貌。真稀奇……"
action item.add id=classic/clover_charm
goto -> :leave

:snub
text "你自己会找到的……"
```

`check cha 10` 掷魅力对抗难度 10。成功就跳到箭头后的标签，失败跳到 `else=` 指定的标签。没有 `else=` 时，失败就只是继续执行下一行。

## check 命令

```scenario
check <stat> <difficulty> [who=player] [dice=d20] [bonus=2] [else=label] [quiet=1] -> :label
```

- `<stat>` — 游戏中的属性 id，例如 `cha` 或 `str`。
- `<difficulty>` — 需要达到的数字：总数不低于它，检定即成功。
- `who=` — 角色 id，或 `player`（默认）。
- `dice=` — `d20`、`2d6`、`1d100` 等。`none` 不掷任何骰子，检定变成单纯的门槛：只看属性，每次结果都一样——适合小说（"够不够勇敢"）。
- `bonus=` — 加进总数；可以为负。
- `else=` — 失败时跳转的标签。
- `quiet=1` — 不显示掷骰行。

总数 = **属性 + 骰子 + 加值**。

### 默认骰子

没写 `dice=` 时，检定使用游戏设置 `checks.dice`，没有该设置则为 `d20`。RPG Plus 在自己的合集中设置了它：

```json
"settings": { "checks.dice": "d20" }
```

想让所有检定都不靠运气的游戏，可以设 `"checks.dice": "none"`。

### 用自己的话讲述结果

检定之后，这些对话变量保存着掷骰信息：`check_stat`、`check_value`、`check_roll`、`check_bonus`、`check_total`、`check_dc` 和 `check_result`。配合 `quiet=1`，一行 `ptext` 就能用故事自己的话来讲述结果：

```scenario
check str 14 quiet=1 else=stuck -> :open
:stuck
ptext "你使出全部的 ${check_total} 去推，门却纹丝不动。"
```

标准一行是翻译键 `checks/line`（`[${check_stat}：${check_total} 对 ${check_dc}] ${check_result}`）；游戏可以在自己的本地化文件里改掉它。

## 如果遇到问题

1. **总数只有骰子。** 属性 id 写错了，或者角色没有这个属性。
2. **结果总是一样。** 游戏的 `checks.dice` 是 `none`，或者这一行里写了 `dice=none`。
3. **骰子写法不对。** 格式错误的 `dice=` 不会弄坏场景，只是什么都不掷——检定于是变成单纯的门槛。骰子请写成 `d20`、`2d6`、`1d100`。
