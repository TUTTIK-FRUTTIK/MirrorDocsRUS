# Пользовательский спавн персонажа

Многие игры нуждаются в кастомизации персонажа. Возможно, вы захотите выбрать цвет волос, глаз, кожи, рост, расу и т.д.

По умолчанию Mirror создаст экземпляр объекта игрока сам. Хотя это удобно, это может помешать вам настроить его. Mirror предоставляет возможность переопределить создание объекта игрока и настроить его.

1. Создайте класс, который наследуется от `NetworkManager` если вы еще этого не сделали. Например:

```csharp
public class MMONetworkManager : NetworkManager
{
    ...
}
```

и используйте это в вашем Network manager.

1. Откройте ваш Network Manager в инспекторе и выключите галочку "Auto Create Player".
2. Создайте сообщение, описывающее вашего игрока. Например:

```csharp
public struct CreateMMOCharacterMessage : NetworkMessage
{
    public Race race;
    public string name;
    public Color hairColor;
    public Color eyeColor;
}

public enum Race
{
    None,
    Elvish,
    Dwarvish,
    Human
}
```

1. Создайте ваши prefab'ы игроков (столько, сколько вам нужно) и добавьте их в "Register Spawnable Prefabs" в вашем Network Manager, или добавьте только один prefab в поле player prefab в инспекторе.
2. Отправьте свое сообщение и зарегистрируйте игрока:

```csharp
public class MMONetworkManager : NetworkManager
{
    public override void OnStartServer()
    {
        base.OnStartServer();

        NetworkServer.RegisterHandler<CreateMMOCharacterMessage>(OnCreateCharacter);
    }

    public override void OnClientConnect()
    {
        base.OnClientConnect();

        // you can send the message here, or wherever else you want
        CreateMMOCharacterMessage characterMessage = new CreateMMOCharacterMessage
        {
            race = Race.Elvish,
            name = "Joe Gaba Gaba",
            hairColor = Color.red,
            eyeColor = Color.green
        };

        NetworkClient.Send(characterMessage);
    }

    void OnCreateCharacter(NetworkConnectionToClient conn, CreateMMOCharacterMessage message)
    {
        // playerPrefab is the one assigned in the inspector in Network
        // Manager but you can use different prefabs per race for example
        GameObject gameobject = Instantiate(playerPrefab);

        // Apply data from the message however appropriate for your game
        // Typically Player would be a component you write with syncvars or properties
        Player player = gameobject.GetComponent();
        player.hairColor = message.hairColor;
        player.eyeColor = message.eyeColor;
        player.name = message.name;
        player.race = message.race;

        // call this to use this gameobject as the primary controller
        NetworkServer.AddPlayerForConnection(conn, gameobject);
    }
}
```

## Состояние готовности <a href="#ready-state" id="ready-state"></a>

В дополнение к плеерам, клиентские подключения также находятся в состоянии “готовности”. Хост отправляет клиентам, которые готовы, информацию о созданных игровых объектах и обновлениях синхронизации состояний; клиентам, которые не готовы, эти обновления не отправляются. Когда клиент изначально подключается к серверу, он еще не готов. Находясь в этом состоянии "неготовности", клиент может выполнять действия, которые не требуют взаимодействия в режиме реального времени с состоянием игры на сервере, такие как загрузка сцен, предоставление игроку возможности выбрать аватар или заполнить поля для входа в систему. Как только клиент завершит всю свою предигровую работу и все его ресурсы будут загружены и он сможет вызвать `NetworkClient.Ready` чтобы войти в состояние “готовности”. Приведенный выше простой пример демонстрирует реализацию состояний готовности; поскольку добавление игрока с помощью `NetworkServer.AddPlayerForConnection` также переводит клиента в состояние готовности, если он еще не находится в этом состоянии.

Клиенты могут отправлять и получать network messages без состояния готовности, что также означает, что они могут делать это, не имея активного игрового объекта игрока. Таким образом, клиент в меню или на экране выбора может подключиться к игре и взаимодействовать с ней, даже если у него нет игрового объекта игрока. Смотрите документацию о [Network Messages](../communications/network-messages.md) для получения более подробной информации об отправке сообщений без вызовов без command и RPC.

## Смена игроков <a href="#switching-players" id="switching-players"></a>

Чтобы заменить игровой объект игрока для клиента, используйте `NetworkServer.ReplacePlayerForConnection`. Это полезно для ограничения команд, которые игроки могут отдавать в определенное время, например, на экране предыгровой комнаты. Эта функция принимает те же аргументы, что и `AddPlayerForConnection`, но позволяет уже иметь объект игрока для этого клиента. Старый игровой объект игрока не обязательно уничтожать. `NetworkRoomManager` использует этот прием для переключения с игрового объекта `NetworkRoomPlayer` к игровому объекту игрока в самой игре, когда все игроки в комнате будут готовы.

Вы также можете использовать `ReplacePlayerForConnection` чтобы возродить игрока или изменить объект, представляющий игрока. В некоторых случаях лучше просто отключить игровой объект и сбросить его игровые атрибуты при возрождении. Следующий пример кода демонстрирует, как на самом деле заменить игровой объект player новым игровым объектом:

```csharp
public class MyNetworkManager : NetworkManager
{
    public void ReplacePlayer(NetworkConnectionToClient conn, GameObject newPrefab)
    {
        // Кэшировать ссылку на текущий объект игрока
        GameObject oldPlayer = conn.identity.gameObject;

        // Instantiate новый объект игрока и рассказать об этом клиентам
        // Включить значение true для параметра keepAuthority чтобы предотвратить смену владельца
        NetworkServer.ReplacePlayerForConnection(conn, Instantiate(newPrefab), true);

        // Удалите предыдущий объект игрока, который теперь был заменен
        // Для завершения замены требуется задержка.
        Destroy(oldPlayer, 0.1f);
    }
}
```

Если игровой объект игрока для подключения уничтожен, то этот клиент не может выполнять Command. Однако они по-прежнему могут отправлять Network Messages.

Чтобы использовать `ReplacePlayerForConnection` вы должны иметь `NetworkConnection`  для игрового объекта и клиента игрока, чтобы установить связь между игровым объектом и клиентом. Обычно это свойство `connectionToClient` в классе `NetworkBehaviour`, но если старый объект игрока уже был уничтожен, то это может быть недоступно.

Чтобы найти соединение, доступно несколько списков. При использовании `NetworkRoomManager`, игроки комнаты будут доступны в `roomSlots`. `NetworkServer` также имеет список из `connections`.
