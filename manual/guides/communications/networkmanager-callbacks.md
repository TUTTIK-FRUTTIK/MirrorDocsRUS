# Обратные вызовы NetworkManager

{% hint style="info" %}
Смотрите также [NetworkManager](https://mirror-networking.com/docs/api/Mirror.NetworkManager.html) в API Reference.
{% endhint %}

Существует ряд событий, которые могут произойти в ходе обычной работы многопользовательской игры, таких как запуск хоста, присоединение игрока или уход игрока. Каждое из этих возможных событий имеет связанный с ним обратный вызов, который вы можете реализовать в своем собственном коде, чтобы предпринять действие при возникновении события.

Чтобы сделать это для `NetworkManager`, вам нужно создать свой собственный скрипт, который будет наследоваться от `NetworkManager`. Затем вы можете переопределить виртуальные методы в `NetworkManager` с вашей собственной реализацией того, что должно произойти, когда произойдет данное событие.

На этой странице перечислены все виртуальные методы (обратные вызовы), которые вы можете реализовать в `NetworkManager`, когда они происходят. Выполняемые обратные вызовы и порядок их выполнения немного различаются в зависимости от того, в каком режиме запущена ваша игра, поэтому обратные вызовы каждого режима перечислены ниже отдельно.

Игра может быть запущена в одном из трех режимов: хост, клиент или только сервер. Обратные вызовы для каждого режима перечислены ниже:

## Режим Хоста: <a href="#host-mode" id="host-mode"></a>

Когда хост запускается:

* `OnStartServer`
* `OnStartHost`
* `OnServerConnect`
* `OnStartClient`
* `OnClientConnect`
* `OnServerSceneChanged`
* `OnServerReady`
* `OnServerAddPlayer`
* `OnClientChangeScene`
* `OnClientSceneChanged`

Когда клиент подключается:

* `OnServerConnect`
* `OnServerReady`
* `OnServerAddPlayer`

Когда клиент отключается:

* `OnServerDisconnect`

Когда хост останавливается:

* `OnStopHost`
* `OnServerDisconnect`
* `OnStopClient`
* `OnStopServer`

## Режим клиента <a href="#client-mode" id="client-mode"></a>

Когда клиент запускается:

* `OnStartClient`
* `OnClientConnect`
* `OnClientChangeScene`
* `OnClientSceneChanged`

Когда клиент останавливается:

* `OnStopClient`
* `OnClientDisconnect`

## Режим сервера <a href="#server-mode" id="server-mode"></a>

Когда сервер запускается:

* `OnStartServer`
* `OnServerSceneChanged`

Когда сервер подключается:

* `OnServerConnect`
* `OnServerReady`
* `OnServerAddPlayer`

Когда сервер отключается:

* `OnServerDisconnect`

Когда сервер останавливается:

* `OnStopServer`
