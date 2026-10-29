# Stat Checks — rolling a stat against a difficulty

Stat Checks adds one scenario command, `check`: take a character's stat, add a dice roll and a bonus, compare the total with a difficulty, and continue the story on success or on failure. A visual novel uses it to persuade a guard, a tile RPG to force a rusty door. The stats themselves come from Stats Engine and the game's rules (in RPG Plus, Classic RPG Rules).

## In the game

In the RPG Plus prologue a stranger stands at a crossroads. One answer names the check it makes — Charisma against 10:

![The crossroads at dusk; the question "What will you say to her?" and two answers: "Politely ask the way [Charisma 10]" and "Walk past in silence".](assets/seer-choice.jpg)

After the answer the roll is shown as one line:

![The dialog box with the line "[Charisma: 11 vs 10] Success".](assets/check-line.jpg)

The hero here is a level 1 rogue with Charisma 8; the d20 rolled 3, so the total is 11 against 10 — a success. The story goes on with the success branch: the stranger gives the hero a charm.

![The success branch: "Polite. Fancy that." She tosses you a clover charm.](assets/check-branch.jpg)

With a total below 10 the line would say *Failure* and the story would take the other branch, where she turns away.

## The scenario behind it

From `rpg_plus_game/scenarios/rpg_plus/prologue.scenario` (lines shortened):

```scenario
option "Politely ask the way [Charisma 10]" -> :ask_way
option "Walk past in silence" -> :leave

:ask_way
check cha 10 who=rpg_hero else=snub -> :gift

:gift
text "Polite. Fancy that..."
action item.add id=classic/clover_charm
goto -> :leave

:snub
text "Find it yourself..."
```

`check cha 10` rolls Charisma against difficulty 10. On success it jumps to the label after the arrow, on failure to the `else=` label. Without `else=` a failure simply goes on with the next line.

## The check command

```scenario
check <stat> <difficulty> [who=player] [dice=d20] [bonus=2] [else=label] [quiet=1] -> :label
```

- `<stat>` — a stat id of the game, such as `cha` or `str`.
- `<difficulty>` — the number to reach: the check succeeds when the total is at least this.
- `who=` — a character id, or `player` (the default).
- `dice=` — `d20`, `2d6`, `1d100` and so on. `none` rolls nothing, which makes the check a plain threshold: the stat alone decides, the same way every time — useful in a novel ("brave enough or not").
- `bonus=` — added to the total; it may be negative.
- `else=` — the label for a failure.
- `quiet=1` — do not show the roll line.

The total is **stat + dice + bonus**.

### Default dice

Without `dice=` the check uses the game's setting `checks.dice`, else `d20`. RPG Plus sets it in its collection:

```json
"settings": { "checks.dice": "d20" }
```

A game that wants all its checks without chance sets `"checks.dice": "none"`.

### Telling the result your own way

After a check these dialog variables hold the roll: `check_stat`, `check_value`, `check_roll`, `check_bonus`, `check_total`, `check_dc` and `check_result`. With `quiet=1` a `ptext` line can show it in the story's words:

```scenario
check str 14 quiet=1 else=stuck -> :open
:stuck
ptext "You push with all your ${check_total}, but the door holds."
```

The standard line is the translation key `checks/line` (`[${check_stat}: ${check_total} vs ${check_dc}] ${check_result}`); a game may override it in its locale files.

## If something goes wrong

1. **The total is only the dice.** The stat id is wrong, or the character has no such stat.
2. **Always the same result.** The game's `checks.dice` is `none`, or `dice=none` is written in the line.
3. **Bad dice text.** A malformed `dice=` rolls nothing rather than breaking the scene, so the check becomes a plain threshold. Write the dice as `d20`, `2d6`, `1d100`.
