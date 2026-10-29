# Stat Checks — 능력치로 난이도 굴리기

Stat Checks는 시나리오에 명령 하나를 추가합니다. 바로 `check`입니다. 캐릭터의 능력치를 가져와 주사위 굴림과 보너스를 더하고, 합계를 난이도와 비교한 뒤 성공이냐 실패냐에 따라 이야기를 이어 갑니다. 비주얼 노벨은 이것으로 경비병을 설득하고, 타일 RPG는 녹슨 문을 부수는 데 씁니다. 능력치 자체는 Stats Engine과 게임의 규칙(RPG Plus에서는 Classic RPG Rules)이 가져 옵니다.

## 게임 안에서

RPG Plus 프롤로그에서 길이 갈라지는 자리에 나그네가 앉아 있습니다. 선택지 하나는 자기가 거는 판정 — 매력 대 10 — 을 스스로 밝힙니다:

![해질녘의 갈림길. 질문 "그녀에게 뭐라고 할래?"와 두 선택지: "정중하게 길을 묻는다 [매력 10]", "말없이 지나친다".](assets/seer-choice.jpg)

선택 후에는 굴림이 한 줄로 표시됩니다:

![대화창에 표시된 줄: "[매력: 11 대 10] 성공".](assets/check-line.jpg)

여기서 주인공은 매력 8의 1레벨 무뢰배입니다. d20이 3이 나와 합계는 11 대 10 — 성공입니다. 이야기는 성공 갈래로 이어지고, 나그네는 주인공에게 부적을 던져 줍니다.

![성공 갈래: "예의 바르네. 별일이야." 그녀가 클로버 부적을 던져 준다.](assets/check-branch.jpg)

합계가 10에 미치지 못했다면 그 줄에는 *실패*가 표시되고, 이야기는 그녀가 고개를 돌리는 다른 갈래로 흘러갔을 것입니다.

## 그 뒤의 시나리오

`rpg_plus_game/scenarios/rpg_plus/prologue.scenario`에서 가져 온 것입니다(줄은 줄임):

```scenario
option "정중하게 길을 묻는다 [매력 10]" -> :ask_way
option "말없이 지나친다" -> :leave

:ask_way
check cha 10 who=rpg_hero else=snub -> :gift

:gift
text "예의 바르네. 별일이야..."
action item.add id=classic/clover_charm
goto -> :leave

:snub
text "알아서 찾게 될 거야..."
```

`check cha 10`은 매력을 난이도 10에 대해 굴립니다. 성공하면 화살표 뒤의 레이블로, 실패하면 `else=`의 레이블로 건너뜁니다. `else=`가 없으면 실패는 그저 다음 줄로 이어집니다.

## check 명령

```scenario
check <stat> <difficulty> [who=player] [dice=d20] [bonus=2] [else=label] [quiet=1] -> :label
```

- `<stat>` — 게임의 능력치 id. 예: `cha`, `str`.
- `<difficulty>` — 넘어야 할 숫자. 합계가 이것 이상이면 판정은 성공입니다.
- `who=` — 캐릭터 id, 또는 `player`(기본값).
- `dice=` — `d20`, `2d6`, `1d100` 등. `none`은 아무것도 굴리지 않아 판정이 단순 기준선이 됩니다. 능력치만으로, 늘 같은 방식으로 결정됩니다 — 노벨에 유용합니다("충분히 용감한가, 아닌가").
- `bonus=` — 합계에 더해집니다. 음수여도 됩니다.
- `else=` — 실패일 때 갈 레이블.
- `quiet=1` — 굴림 줄을 표시하지 않습니다.

합계는 **능력치 + 주사위 + 보너스**입니다.

### 기본 주사위

`dice=`가 없으면 판정은 게임 설정 `checks.dice`를 쓰고, 그것도 없으면 `d20`을 씁니다. RPG Plus는 자기 컬렉션에 이렇게 정해 두었습니다:

```json
"settings": { "checks.dice": "d20" }
```

모든 판정에서 우연을 빼고 싶은 게임은 `"checks.dice": "none"`으로 정합니다.

### 결과를 제 방식으로 말하기

판정 뒤에는 이 대화 변수들에 굴림이 남습니다: `check_stat`, `check_value`, `check_roll`, `check_bonus`, `check_total`, `check_dc`, `check_result`. `quiet=1`과 함께라면 `ptext` 줄이 이야기의 말로 결과를 표현할 수 있습니다:

```scenario
check str 14 quiet=1 else=stuck -> :open
:stuck
ptext "모든 ${check_total}을(를) 짜내어 밀지만 문은 버틴다."
```

표준 줄은 번역 키 `checks/line`(`[${check_stat}: ${check_total} 대 ${check_dc}] ${check_result}`)입니다. 게임은 자기 로컬라이즈 파일에서 이것을 바꿀 수 있습니다.

## 문제가 생겼을 때

1. **합계가 주사위뿐이다.** 능력치 id가 틀렸거나 그 캐릭터에게 그 능력치가 없습니다.
2. **결과가 늘 같다.** 게임의 `checks.dice`가 `none`이거나 줄에 `dice=none`이 적혀 있습니다.
3. **주사위 문구가 깨졌다.** 잘못된 형식의 `dice=`는 장면을 망가뜨리는 대신 아무것도 굴리지 않아, 판정이 단순 기준선이 됩니다. 주사위는 `d20`, `2d6`, `1d100`처럼 쓰세요.
