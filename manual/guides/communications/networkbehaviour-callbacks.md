# Обратные вызовы NetworkBehaviour

## Обратные вызовы NetworkBehaviour <a href="#networkbehaviour-callbacks" id="networkbehaviour-callbacks"></a>

{% hint style="info" %}
Смотрите также [NetworkBehaviour](https://mirror-networking.com/docs/api/Mirror.NetworkBehaviour.html) в API Reference.
{% endhint %}

Существует ряд событий, которые могут произойти в ходе обычной работы многопользовательской игры, таких как запуск хоста, присоединение игрока или уход игрока. Каждое из этих возможных событий имеет связанный с ним обратный вызов, который вы можете реализовать в своем собственном коде, чтобы предпринять действие при возникновении события.

Когда вы создаете скрипт, который наследуется от `NetworkBehaviour`, вы можете написать свою собственную реализацию того, что должно произойти, когда произойдут эти события. Чтобы сделать это, вы переопределяете виртуальные методы в классе, наследованном от `NetworkBehaviour` с вашей собственной реализацией того, что должно произойти, когда произойдет какое то событие.

Это полный список виртуальных методов (обратных вызовов), которые вы можете реализовать в `NetworkBehaviour`, и где они вызываются

### Только сервер <a href="#server-only" id="server-only"></a>

* OnStartServer
  * вызывается когда объект спавнится на сервере
* OnStopServer
  * вызывается когда объект уничтожается или деспавнится на сервере
* OnSerialize
  * вызывается, когда класс сериализуется перед отправкой клиенту, при переопределении обязательно вызывается `base.OnSerialize`

### Только клиент <a href="#client-only" id="client-only"></a>

* OnStartClient
  * вызывается когда объект спавнится на клиенте
* OnStartAuthority
  * вызывается когда клиент имеет контроль при спавне объекта (к примеру локальный игрок)
  * вызывается когда сервер даёт клиенту контроль над объектом
* OnStartLocalPlayer
  * вызывается когда объект относится к локальному игроку
* OnStopAuthority
  * вызывается когда у клиента отобрали контроль над объектом (к примеру когда локальный игрок был заменён, но объект не уничтожен)
* OnStopClient
  * вызывается когда объект уничтожается на клиенте с помощью сообщений `ObjectDestroyMessage` или `ObjectHideMessage`

## Example flows <a href="#example-flows" id="example-flows"></a>

Ниже приведен пример порядка вызова для различных режимов

> ПРИМЕЧАНИЕ: `Start` вызывается unity перед первым кадром, в то время как обычно это происходит после обратных вызовов Mirror. Но если вы не вызовите `NetworkServer.Spawn` в кадре вместе с `instantiate`, тогда start может быть вызван первее

> ПРИМЕЧАНИЕ: `OnRebuildObservers` и `OnSetHostVisibility` сейчас находятся в `NetworkVisibility` вместо `NetworkBehaviour`

### Режим сервера <a href="#server-mode" id="server-mode"></a>

Когда вызывается NetworkServer.Spawn (например, при подключении нового клиента и создании объекта игрока)

* `OnStartServer`
* `OnRebuildObservers`
* `Start`

### Режим клиента <a href="#client-mode" id="client-mode"></a>

Когда локальный игрок был заспавнен для клиента

* `OnStartAuthority`
* `OnStartClient`
* `OnStartLocalPlayer`
* `Start`

### Режим хоста <a href="#host-mode" id="host-mode"></a>

Они вызываются только на **игровом объекте игрока** когда клиент подключен:

* `OnStartServer`
* `OnRebuildObservers`
* `OnStartAuthority`
* `OnStartClient`
* `OnSetHostVisibility`
* `OnStartLocalPlayer`
* `Start`
