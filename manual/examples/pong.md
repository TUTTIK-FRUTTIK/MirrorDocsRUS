# Pong

Простой пример по тому "Как создать многопользовательскую игру с Mirror" является Pong, которые выключен в пакет AssetStore от Mirror. Он отображает пример использования NetworkManager, NetworkManagerHUD, NetworkBehaviour, NetworkIdentity, NetworkTransform, NetworkStartPosition и NetworkingAttributes.

![](<../../.gitbook/assets/image (81).png>)

## Установка количества игроков <a href="#setting-the-number-of-players" id="setting-the-number-of-players"></a>

Прежде всего, давайте взглянем на объект NetworkManager в main сцене. При добавлении компонента NetworkManager на объект, несколько настроек по умолчанию уже установлены (**Don't destroy on Load**, **Run in Background**, ...) Для игры в Pong максимальное количество игроков равно 2, поэтому настройка **Network Info/Max connections** будет тоже равняться 2. Поскольку других сцен нет (room, online or offline scene) в этом примере поля для **Offline Scene** и **Online Scene** останутся пустыми.

## Создание игрока <a href="#creating-the-player" id="creating-the-player"></a>

Кроме того, каждому игроку нужна ракетка для игры. У каждого игрока, который присоединится к игре, будет свой собственный управляемый объект, который представляет его в игре. Этот самый объект называется _PlayerObject_. Для спавна Prefab'a _PlayerObject_ должен быть создан и хранить в себе компонент NetworkIdentity с галочкой **Local Player Authority**. **Local Player Authority** позволяет игроку управлять свойствами игровых объектов и изменять их (к примеру передвижение). NetworkManager'у нужна ссылка на этот Prefab, который будет расположен в **Spawn Info/Player Prefab**. Чтобы синхронизировать движение игрока по сети, сборный Prefab также должен иметь NetworkTransform.

![](<../../.gitbook/assets/image (58).png>)

## Начальная позиция игрока <a href="#player-start-position" id="player-start-position"></a>

Основная сцена содержит 2 игровых объекта, в которых есть только компонент NetworkStartPosition (объекты RacketSpawnLeft, RacketSpawnRight). Эти transform'ы будут автоматически зарегистрированы в NetworkManager как позиции для спавна.

![](<../../.gitbook/assets/image (34).png>)

## Настройка сети <a href="#setting-up-the-network" id="setting-up-the-network"></a>

Очень удобным компонентом для установления/тестирования соединений является [Network Manager HUD](../components/network-manager-hud.md). Он предоставляет базовую функциональность для запуска игры в качестве клиента, сервера или хоста (клиент и сервер одновременно). Ему требуется компонент Network Manager.

<figure><img src="../../.gitbook/assets/image (35) (1).png" alt=""><figcaption><p>Network Manager HUD</p></figcaption></figure>

## Мяч для понга <a href="#the-ball-of-pong" id="the-ball-of-pong"></a>

Мяч является основным объектом внимания в Pong, так как это объект, необходимый для набора очков. Его компонент NetworkIdentity не имеет ни **Server Only** ни **Local Player Authority** галочек, поскольку он управляется физическим движком сервера и может подвергаться влиянию игроков. Как и в случае с _PlayerObject_ позиция синхронизируется через NetworkTransform. При наличии нескольких сцен мяч может спанитьсяNetworkManager'ом, но для простоты этого примера он помещен непосредственно в основную сцену.
