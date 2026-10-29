# Stat Checks — tirar una estadística contra una dificultad

Stat Checks añade un comando de escenario, `check`: toma la estadística de un personaje, suma una tirada de dados y un bonus, compara el total con una dificultad y continúa la historia según salga bien o mal. Una novela visual lo usa para convencer a un guardia; un RPG de tiles, para forzar una puerta oxidada. Las estadísticas en sí vienen de Stats Engine y de las reglas del juego (en RPG Plus, Classic RPG Rules).

## En el juego

En el prólogo de RPG Plus una forastera aguarda en un cruce de caminos. Una de las respuestas anuncia la prueba que supone: Carisma contra 10:

![El cruce al anochecer; la pregunta «¿Qué le dirás?» y dos respuestas: «Preguntar el camino con cortesía [Carisma 10]» y «Pasar de largo en silencio».](assets/seer-choice.jpg)

Tras la respuesta, la tirada se muestra en una sola línea:

![El cuadro de diálogo con la línea «[Carisma: 11 contra 10] Éxito».](assets/check-line.jpg)

El héroe es aquí un pícaro de nivel 1 con Carisma 8; el d20 sacó un 3, así que el total es 11 contra 10: un éxito. La historia sigue por la rama del éxito: la forastera lanza al héroe un amuleto.

![La rama del éxito: «—Educado. Mira tú. —Te lanza un amuleto de trébol».](assets/check-branch.jpg)

Con un total por debajo de 10, la línea diría *Fallo* y la historia tomaría la otra rama, en la que ella se da la vuelta.

## El escenario que hay detrás

De `rpg_plus_game/scenarios/rpg_plus/prologue.scenario` (líneas acortadas):

```scenario
option "Preguntar el camino con cortesía [Carisma 10]" -> :ask_way
option "Pasar de largo en silencio" -> :leave

:ask_way
check cha 10 who=rpg_hero else=snub -> :gift

:gift
text "—Educado. Mira tú..."
action item.add id=classic/clover_charm
goto -> :leave

:snub
text "—Ya lo encontrarás..."
```

`check cha 10` tira Carisma contra dificultad 10. Si tiene éxito salta a la etiqueta que sigue a la flecha; si falla, a la etiqueta de `else=`. Sin `else=`, un fallo simplemente continúa con la línea siguiente.

## El comando check

```scenario
check <stat> <difficulty> [who=player] [dice=d20] [bonus=2] [else=label] [quiet=1] -> :label
```

- `<stat>` — un id de estadística del juego, por ejemplo `cha` o `str`.
- `<difficulty>` — el número que hay que alcanzar: la prueba tiene éxito cuando el total llega al menos a eso.
- `who=` — un id de personaje, o `player` (por defecto).
- `dice=` — `d20`, `2d6`, `1d100` y demás. `none` no tira nada: la prueba se convierte en un umbral simple; decide solo la estadística, siempre igual — útil en una novela («lo bastante valiente o no»).
- `bonus=` — se suma al total; puede ser negativo.
- `else=` — la etiqueta del fallo.
- `quiet=1` — no mostrar la línea de la tirada.

El total es **estadística + dados + bonus**.

### Dados por defecto

Sin `dice=`, la prueba usa el ajuste `checks.dice` del juego y, si no existe, `d20`. RPG Plus lo fija en su colección:

```json
"settings": { "checks.dice": "d20" }
```

Un juego que quiera todas sus pruebas sin azar pone `"checks.dice": "none"`.

### Contar el resultado a tu manera

Tras una prueba, estas variables del diálogo guardan la tirada: `check_stat`, `check_value`, `check_roll`, `check_bonus`, `check_total`, `check_dc` y `check_result`. Con `quiet=1`, una línea `ptext` puede contarlo con palabras de la historia:

```scenario
check str 14 quiet=1 else=stuck -> :open
:stuck
ptext "Apuñas con todo tu ${check_total}, pero la puerta no cede."
```

La línea estándar es la clave de traducción `checks/line` (`[${check_stat}: ${check_total} contra ${check_dc}] ${check_result}`); un juego puede sustituirla en sus archivos de localización.

## Si algo falla

1. **El total es solo los dados.** El id de la estadística está mal, o el personaje no tiene esa estadística.
2. **Siempre el mismo resultado.** El `checks.dice` del juego es `none`, o en la línea está escrito `dice=none`.
3. **Texto de dados roto.** Un `dice=` mal formado no tira nada en lugar de romper la escena, así que la prueba se queda en un umbral simple. Escribe los dados como `d20`, `2d6`, `1d100`.
