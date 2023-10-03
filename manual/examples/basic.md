# Basic

Основной пример иллюстрирует, как управлять объектами пользовательского интерфейса из объекта Player, используя локально созданный экземпляр Prefab'a `PlayerUI` с [SyncVars](../guides/synchronization/syncvars.md) и Событиями

<div align="left">

<img src="../../.gitbook/assets/image (103).png" alt="">

</div>

Канвас сцены имеет скрипт `CanvasUI` с ссылкой на ребенка:

<div align="left">

<img src="../../.gitbook/assets/image (12).png" alt="Scene Canvas">

</div>

Prefab'ы `PlayerUI` UI фрагементы, у которых есть скрипт `PlayerUI` с ссылкой на их дочерний объект:

<div align="left">

<img src="../../.gitbook/assets/image (109).png" alt="PlayerUI Prefab">

</div>

Скрипт игрока на объекте Player имеет ссылку на Prefab `PlayerUI` и дерево из `SyncVars`:

<div align="left">

<img src="../../.gitbook/assets/image (119).png" alt="Player Object">

</div>

Скрипт игрока также содержит три события, которые вызываются из [SyncVar hooks](../guides/synchronization/syncvar-hooks.md):

```
public event System.Action<int> OnPlayerNumberChanged;
public event System.Action<Color32> OnPlayerColorChanged;
public event System.Action<int> OnPlayerDataChanged;
```

Когда объект игрока спавнится на клиенте, `PlayerUI` создает ребенка в `PlayersPanel` который лежит в канвасе при помощи ссылки на скрипт `CanvasUI`, и метод `SetPlayer` вызывается с соответствующей ссылкой на скрипт игрока. Скрипт `PlayerUI` подписывается на описанные выше события и обновляет свой UI когда `SyncVars` обновляются с сервера.
