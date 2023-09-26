# Network Room Player

Network Room Player сохраняет состояние каждого игрока для [Network Room Manager](network-room-manager.md) пока он находится в комнате. При использовании этого компонента вам необходимо написать скрипт, который позволяет игрокам указывать, что они готовы начать игру, что устанавливает свойство ReadyToBegin.

Игровой объект с компонентом Network Room Player должен иметь компонент Network Identity. Когда вы создаете на объекте компонент Network Room Player, Unity автоматически создает на нём же компонент Network Identity если он не был добавлен ранее.

![](<../../.gitbook/assets/image (32).png>)

* **Show Room GUI**\
  Включите это, чтобы отобразить графический интерфейс разработчика для игроков в комнате. Этот пользовательский интерфейс предназначен только для простоты разработки. Это включено по умолчанию.
* **Ready To Begin**\
  Диагностический индикатор готовности игрока.
* **Index**\
  Диагностический индекс игрока, по типу Player 1, Player 2, и т. д.
* **Network Sync Interval**\
  Скорость, с которой информация передается от Network Room Player к серверу.

## Методы <a href="#methods" id="methods"></a>

### Виртуальные методы клиента <a href="#client-virtual-methods" id="client-virtual-methods"></a>

```csharp
public virtual void OnClientEnterRoom() {}
public virtual void OnClientExitRoom() {}
public virtual void OnClientReady(bool readyState) {}
```
