# Приступая к работе

В этом документе описаны шаги по созданию многопользовательской игры с помощью Mirror. Описанные здесь инструкции являются упрощенной версией инструкций более сложного уровня для реальной игры; она не всегда работает в точности так, как сложная, но она предоставляет базовый рецепт процесса.

## Видео туториалы <a href="#video-tutorials" id="video-tutorials"></a>

Посмотрите эти [потрясающие видеоролики](../../gaidy-ot-kommyuniti/video-tutorials.md)  показывающие как начать работу с Mirror.

## Шаблоны скриптов

* Создавайте новые скрипты Network Behaviour и другие сценарии быстрее

Смотреть [Шаблоны скриптов](script-templates.md).

## Network Manager Setup <a href="#networkmanager-set-up" id="networkmanager-set-up"></a>

* Создайте новый Network Manager из меню [Assets > Create > Mirror](script-templates.md).
* Добавьте новый GameObject на сцену и переименуйте его в “NetworkManager”.
* Добавьте только что созданному объекту Network Manager компонент “NetworkManager”.
* Также добавьте [NetworkManagerHUD](../components/network-manager-hud.md) на данный объект. Он реализует пользовательский интерфейс для управления состоянием сетевой игры.

Смотреть [Как пользоваться NetworkManager'ом](../components/network-manager.md).

## Prefab игрока <a href="#player-prefab" id="player-prefab"></a>

* Найдите prefab для объекта игрока или же создайте его
* Добавьте NetworkIdentity компонент prefab'у игрока
* Назначьте `Player Prefab` в поле "player prefab" NetworkManager’а&#x20;
* Удалите объект игрока на сцене если он там есть

Смотрите [Player Objects](../guides/gameobjects/player-gameobjects.md) для получения более подробной информации.

## Движение игрока <a href="#player-movement" id="player-movement"></a>

* Добавьте NetworkTransform компонент на prefab игрока
* Проверьте чтобы SyncDirection у компонента стоял на селекторе ClientToServer.
* Обновите скрипт передвижения, чтобы выполнялась проверка на `isLocalPlayer`
* Добавьте метод OnStartLocalPlayer для того чтобы в нём взять контроль над главной камерой сцены.

Например, данный скрипт обрабатывает движение только для локального игрока:

```csharp
using UnityEngine;
using Mirror;

public class Controls : NetworkBehaviour
{
    void Update()
    {
        // exit from update if this is not the local player
        if (!isLocalPlayer) return;

        // handle player input for movement
    }
}
```

## Базовое состояние игрока <a href="#basic-player-game-state" id="basic-player-game-state"></a>

* Наследуйте скрипты, содержащие важные данные именно от NetworkBehaviour вместо MonoBehaviours
* Используйте SyncVar для синхронизации важных переменных

Смотреть [Синхронизация состояний](../guides/synchronization/).

## Удаленные действия <a href="#networked-actions" id="networked-actions"></a>

* Наследуйте скрипты, которые выполняют важные действия именно от NetworkBehaviour вместо MonoBehaviours
* Обновите важные функции игрока на \[Command]

Смотреть [Удаленные действия](../guides/communications/remote-actions.md).

## Неигровые GameObjects <a href="#non-player-game-objects" id="non-player-game-objects"></a>

Исправьте неигровые prefab'ы, такие как враги:

* Добавьте NetworkIdentity компонент
* Добавьте NetworkTransform компонент
* Назначьте эти prefab'ы в массив "Spawnable Objects" в NetworkManager'е
* Обновите скрипты с действиями и состояниями для работы в сети

## Спавнеры <a href="#spawners" id="spawners"></a>

* Унаследуйте сценарии спавна на NetworkBehaviours
* Измените спавнеры на работу только со стороны сервера (используйте isServer проверку или `OnStartServer()` функцию)
* Вызывайте `NetworkServer.Spawn()` для создания игровых объектов

## Позиция спавна для игроков <a href="#spawn-positions-for-players" id="spawn-positions-for-players"></a>

* Создайте новый GameObject и разместите в точку для стартовой позиции
* Добавьте NetworkStartPosition компонент на данный объект
