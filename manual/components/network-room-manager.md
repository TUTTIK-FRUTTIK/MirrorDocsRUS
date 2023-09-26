# Network Room Manager

**Пожалуйста посмотрите пример Room в папке Examples в вашей папке Mirror.**

Network Room Manager является специализированным типом [Network Manager](network-manager.md) который обеспечивает многопользовательскую комнату перед входом в основную игровую сцену игры. Это позволяет вам настроить сеть с:

* Максимальным лимитом пользователей
* Автоматическим стартом, когда все игроки готовы
* Блокирующейся после запуска игрой, не допуская опоздавших участников
* Настраиваемыми способами выбора игроками опций во время нахождения в комнате

Существует два типа объектов игрока с Network Room Manager:

**Room Player Prefab**

* По одному для каждого игрока
* Создается при подключении клиента или добавлении игрока
* Сохраняется до тех пор, пока клиент не отключится
* Содержит флаг готовности и данные конфигурации
* Выполняет команды в комнате
* Обязан использовать компонент [Network Room Player](network-room-player.md)

**Player Prefab**

* По одному для каждого игрока
* Создается при запуске игровой сцены
* Уничтожается при выходе из игровой сцены
* Обрабатывает команды в игре

![](<../../.gitbook/assets/image (13).png>)

## Параметры <a href="#properties" id="properties"></a>

* **Show Room GUI**\
  Отображает элементы управления OnGUI по умолчанию для комнаты.
* **Min Players**\
  Минимальное количество игроков, необходимое для начала игры.
* **Room Player Prefab**\
  Prefab, который нужно создать для игроков, когда они войдут в комнату (требуется компонент Network Room Player).
* **Room Scene**\
  Сцена, которую нужно использовать для оформления комнаты.
* **Gameplay Scene**\
  Сцена, которую нужно использовать для основной игры.
* **pendingPlayers**\
  Список, содержащий игроков, которые готовы начать играть.
* **roomSlots**\
  Список, который управляет слотами для подключенных клиентов в комнате.
* **allPlayersReady**\
  Bool, указывающий, готовы ли все игроки начать игру. Это значение меняется по мере того, как игроки вызывают`CmdChangeReadyState` указывающее true или false, и будет установлено значение false при подключении нового клиента.

## Методы <a href="#methods" id="methods"></a>

### Виртуальные методы сервера <a href="#server-virtual-methods" id="server-virtual-methods"></a>

```csharp
public virtual void OnRoomStartHost() {}
public virtual void OnRoomStopHost() {}
public virtual void OnRoomStartServer() {}
public virtual void OnRoomServerConnect(NetworkConnection conn) {}
public virtual void OnRoomServerDisconnect(NetworkConnection conn) {}
public virtual void OnRoomServerSceneChanged(string sceneName) {}
public virtual GameObject OnRoomServerCreateRoomPlayer(NetworkConnection conn)
{
    return null;
}
public virtual GameObject OnRoomServerCreateGamePlayer(NetworkConnection conn)
{
    return null;
}
public virtual bool OnRoomServerSceneLoadedForPlayer(GameObject roomPlayer, GameObject gamePlayer)
{
    return true;
}
public virtual void OnRoomServerPlayersReady()
{
    ServerChangeScene(GameplayScene);
}
```

### Виртуальные методы клиента <a href="#client-virtual-methods" id="client-virtual-methods"></a>

```csharp
public virtual void OnRoomClientEnter() {}
public virtual void OnRoomClientExit() {}
public virtual void OnRoomClientConnect(NetworkConnection conn) {}
public virtual void OnRoomClientDisconnect(NetworkConnection conn) {}
public virtual void OnRoomStartClient() {}
public virtual void OnRoomStopClient() {}
public virtual void OnRoomClientSceneChanged(NetworkConnection conn) {}
public virtual void OnRoomClientAddPlayerFailed() {}
```
