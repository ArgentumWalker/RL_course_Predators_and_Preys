# Reinforcement Learning 2: Охотники и жертвы
Набор окружений для второй части курса по обучению с подкреплением.

## How to
### Об окружениях
Окружение представляет собой Grid World, в котором живут охотники и жертвы.
Агенту предоставляется возможность управлять командой охотников, а жертвы на каждом ходу совершают случайное действие.

Задача охотников - поймать как можно больше жертв за отведенное время. Каждая пойманная жертва дает 1 очко команде. Если охотник ловит охотника из команды соперников, то команда получает 3 очка, а также, в случае, если команда пойманного имеет больший счет, по 0.1 за каждое 1 очко разницы в счете между командой пойманного и командой поймавшего. Пойманный охотник появится на поле пропустив свой ход.

Эпизод завершается по истечении времени или после того, как все жертвы будут пойманы.

Состояние среды - карта, на которой расположены сущности. Значения ячеек карты расшифровываются следующим образом:
* `(-1, 0)` - пустая клетка
* `(-1, -1)` - непроходимый камень
* `(-1, 1)` - бонус. Бонус может накапливаться для каждого хищника. Пока у хищника есть хотя бы один бонус, его невозможно съесть.
* `(team, i)` - сущность из команды `team` с индексом `i`. Для агента его номер команды всегда равен `0`. Номер команды жертв всегда равен `num_teams` (количеству команд охотников). Все команды-соперники имеют индексы от `1` до `num_teams-1`.

Каждому охотнику доступно всего 5 действий: `0` - стоять на месте, и `1-4` - двигаться в заданном направлении. Жертвы всегда ходят последними, а охотники ходят в случайном порядке.

На каждом шаге среда принимает на вход список действий охотников. Размер списка равен размеру команды, а индексы соответствуют индексам на поле.

На выход среда возвращает текущее состояние `state` и дополнительную информацию `info`.

Дополнительная информация содержит следующие поля:
* `"eaten"` - словарь из пойманных существ. Ключи - номер команды и индекс пойманного существа, значение - номер команды и индекс существа, которое его поймало.
* `"predators"` - список охотников из команды агента. Обратите внимание, что `x` соответствует горизонтальной координате, а `y` - вертикальной, т.е. охотник находится в клетке `state[y, x]`.
* `"enemies"` - список охотников из команд-соперников.

### Создание окружений
Все доступные окружения находятся в файле `world/envs.py`. Для их создания необходимо также создать Realm и MapGenerator.

Пример создания окружения для одного агента:
```python
from world.envs import OnePlayerEnv
from world.realm import Realm
from world.map_loaders.single_team import SingleTeamLabyrinthMapLoader

env = OnePlayerEnv(Realm(SingleTeamLabyrinthMapLoader(), 1))
```

### Изменение параметров карт
На данный момент доступны два вида карт для генерации: лабиринты и скалы.

Им соответствуют определенные генераторы карт, для которых можно задавать различные параметры: количество жертв, размер карты, радиус появления охотников и количество точек появления.

Кроме того, для лабиринтов можно менять максимальное и минимальное количество дополнительных проходов, что позволяет регулировать сложность карты. 
Аналогично, для карты со скалами можно менять вероятность появления камней и вероятность появления дополнительных камней по соседству с уже появившимися.

### Рендеринг эпизода
Для удобства рендеренга в файле `world/util.py` назодится обертка над окружением, которая позволяет покадрово отобразить эпизод.

Для того, чтобы отрендерить эпизод, создайте обертку над окружением. Далее завершите эпизод также, как с обычным окружением. После завершения эпизода вызовите метод `.render` у обертки. Метод на вход принимает путь, по которому будут сохранены кадры эпизода (каждый кадр будет сохранен отдельным файлом), а также количество пикселей на ячейку карты.

### Создание агента
Итоговый агент должен находиться в файле `agent.py` и иметь два метода:
1. `reset(state, info)` - вызывается в начале эпизода. В нем можно выполнить любой необходимый предпосчет.
2. `get_actions(state, info)` - вызывается на каждом шаге эпизода. Возвращает список действий охотников.

### Заскриптованные агенты
В файле `world/scripted_agents.py` можно найти примеры заскриптованных агентов: случайного агента и агента, идущего к ближайшей жертве.

Эти агенты могут пригодиться вам как во время обучения, так и для тестирования собственных решений.

## Задачи
**ОБРАТИТЕ ВНИМАНИЕ** Оценка выставляется по _последней_ посылке.
### Задача 1: поймать их всех
Задача заключается в том, чтобы поймать как можно большее число жертв за отведенное время. 

На всех тестовых картах 100 жертв и 5 охотников каждой команды, размер карты 40x40, а эпизод длится не более 300 шагов.

Тестовые карты делятся на три части:
* 8 карт со скалами. Вероятность появления камней варьируется от `0.01` до `0.15`, вероятность появления дополнительных камней варьируется от `0` до `0.21`.
* 8 карт с лабиринтами. Количество дополнительных проходов варьируется от `24` до `3`.
* 4 карты созданных руками.

Оценка линейно зависит от количества пойманных жертв в среднем на всех картах. Максимальная оценка при 80 и более пойманных жертвах.

### Задача 2: против бота
Задача заключается в том, чтобы победить бота на соревновательной карте. В качестве бота выступает заскриптованный агент, который идет к ближайшей жертве.


Тестовые карты делятся на три части:
* 8 карт со скалами. Вероятность появления камней варьируется от `0.01` до `0.15`, вероятность появления дополнительных камней варьируется от `0` до `0.21`.
* 8 карт с лабиринтами. Количество дополнительных проходов варьируется от `12` до `1`.
* 4 карты созданных руками.

Оценка линейно зависит от количества побед над ботом (очки команды бота меньше очков команды агента). Максимальная при 80% и более побед.


### Задача 3: турнир
Задача заключается в том, чтобы победить как можно большее количество других участников соревнования.


Тестовые карты делятся на три части:
* 8 карт со скалами. Вероятность появления камней варьируется от `0.01` до `0.15`, вероятность появления дополнительных камней варьируется от `0` до `0.21`.
* 8 карт с лабиринтами. Количество дополнительных проходов варьируется от `12` до `1`.
* 4 карты созданных руками.

Оценка зависит от квантили, в которую попало решение участника в конце турнира. 
