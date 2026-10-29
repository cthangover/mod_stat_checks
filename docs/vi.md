# Stat Checks — tung chỉ số đối đầu độ khó

Stat Checks thêm vào kịch bản một lệnh duy nhất: `check`. Lệnh lấy một chỉ số của nhân vật, cộng một lần tung xúc xắc và một điểm cộng, đem tổng so với độ khó, rồi cho câu chuyện tiếp theo kết quả thành hay bại. Tiểu thuyết thị giác dùng nó để thuyết phục lính gác, RPG bản ô dùng nó để phá một cánh cửa rỉ sét. Bản thân các chỉ số đến từ Stats Engine và luật của game (trong RPG Plus là Classic RPG Rules).

## Trong game

Trong phần mở đầu của RPG Plus, một người lạ ngồi nơi ba đường giao nhau. Một lựa chọn tự công bố phép kiểm mà nó thực hiện — Sức hút đối với 10:

![Ngã ba đường lúc chạng vạng; câu hỏi "Ngươi sẽ nói gì với cô ấy?" và hai lựa chọn: "Lễ phép hỏi đường [Sức hút 10]", "Lặng lẽ đi qua".](assets/seer-choice.jpg)

Sau lựa chọn, lần tung hiện thành một dòng:

![Hộp hội thoại với dòng "[Sức hút: 11 so với 10] Thành công".](assets/check-line.jpg)

Nhân vật ở đây là một kẻ du hành cấp 1 với Sức hút 8; d20 ra 3, nên tổng là 11 so với 10 — thành công. Câu chuyện đi theo nhánh thành công: người lạ ném cho nhân vật một lá bùa.

![Nhánh thành công: "Lễ phép. Lạ thật." Cô ném cho ngươi một lá bùa cỏ bốn lá.](assets/check-branch.jpg)

Nếu tổng dưới 10, dòng đó sẽ ghi *Thất bại* và câu chuyện sẽ đi theo nhánh kia, nơi cô ấy quay đi.

## Kịch bản phía sau

Từ `rpg_plus_game/scenarios/rpg_plus/prologue.scenario` (các dòng đã rút gọn):

```scenario
option "Lễ phép hỏi đường [Sức hút 10]" -> :ask_way
option "Lặng lẽ đi qua" -> :leave

:ask_way
check cha 10 who=rpg_hero else=snub -> :gift

:gift
text "Lễ phép. Lạ thật..."
action item.add id=classic/clover_charm
goto -> :leave

:snub
text "Tự ngươi sẽ tìm ra..."
```

`check cha 10` tung Sức hút đối với độ khó 10. Thành công thì nhảy đến nhãn sau mũi tên, thất bại thì đến nhãn của `else=`. Không có `else=`, một lần thất bại đơn giản là tiếp tục dòng kế tiếp.

## Lệnh check

```scenario
check <stat> <difficulty> [who=player] [dice=d20] [bonus=2] [else=label] [quiet=1] -> :label
```

- `<stat>` — một id chỉ số của game, ví dụ `cha` hay `str`.
- `<difficulty>` — con số cần đạt: phép kiểm thành công khi tổng ít nhất bằng nó.
- `who=` — id nhân vật, hoặc `player` (mặc định).
- `dice=` — `d20`, `2d6`, `1d100` v.v. `none` không tung gì cả: phép kiểm thành ngưỡng thuần túy — chỉ số tự quyết định, mỗi lần như nhau. Hữu ích trong tiểu thuyết ("đủ can đảm hay chưa").
- `bonus=` — cộng vào tổng; có thể âm.
- `else=` — nhãn cho lần thất bại.
- `quiet=1` — không hiển thị dòng tung xúc xắc.

Tổng bằng **chỉ số + xúc xắc + điểm cộng**.

### Xúc xắc mặc định

Không có `dice=`, phép kiểm dùng cài đặt `checks.dice` của game, nếu không thì `d20`. RPG Plus đặt nó trong bộ sưu tập của mình:

```json
"settings": { "checks.dice": "d20" }
```

Game nào muốn mọi phép kiểm đều không nhờ may rủi thì đặt `"checks.dice": "none"`.

### Kể kết quả theo cách riêng của bạn

Sau mỗi phép kiểm, các biến hội thoại sau giữ lại lần tung: `check_stat`, `check_value`, `check_roll`, `check_bonus`, `check_total`, `check_dc` và `check_result`. Với `quiet=1`, một dòng `ptext` có thể kể kết quả bằng ngôn từ của câu chuyện:

```scenario
check str 14 quiet=1 else=stuck -> :open
:stuck
ptext "Ngươi dồn toàn bộ ${check_total} vào cú đẩy, nhưng cánh cửa vẫn trơ trơ."
```

Dòng chuẩn là khóa dịch `checks/line` (`[${check_stat}: ${check_total} so với ${check_dc}] ${check_result}`); game có thể thay nó trong các tệp bản địa hóa của mình.

## Khi có trục trặc

1. **Tổng chỉ có xúc xắc.** Id chỉ số sai, hoặc nhân vật không có chỉ số đó.
2. **Luôn cùng một kết quả.** `checks.dice` của game là `none`, hoặc dòng lệnh có `dice=none`.
3. **Chữ xúc xắc hỏng.** `dice=` viết sai dạng sẽ không tung gì thay vì làm hỏng cảnh, nên phép kiểm thành ngưỡng thuần túy. Hãy viết xúc xắc dạng `d20`, `2d6`, `1d100`.
