# Компоненты

Это основные компоненты включенные в Mirror:

* [Network Animator](network-animator.md)\
  Компонент Network Animator позволяет синхронизировать состояния анимации для сетевых объектов. Он синхронизирует состояние и параметры контроллера Animator.
* [Network Authenticator](network-authenticators/)\
  Network Authenticator облегчает интеграцию учетных записей пользователей и учетных данных в ваше приложение.
* [Network Discovery](network-discovery.md)\
  Network Discovery использует широковещательную передачу UDP по локальной сети, позволяя клиентам находить работающий сервер и подключаться к нему.
* [Network Identity](network-identity.md)\
  Компонент Network Identity лежит в основе высокоуровневого API сети Mirror. Он управляет уникальным идентификатором игрового объекта в сети и использует этот идентификатор для информирования сетевой системы об игровом объекте.
* [Network Manager](network-manager.md)\
  Network Manager - это компонент для управления сетевыми аспектами многопользовательской игры.
* [Network Manager HUD](network-manager-hud.md)\
  Network Manager HUD - это инструмент для быстрого запуска, который поможет вам сразу приступить к созданию вашей многопользовательской игры, не создавая предварительно пользовательский интерфейс для создания игры / подключения / присоединения. Это позволяет вам сразу перейти к программированию игрового процесса и означает, что вы сможете создать свою собственную версию этих элементов управления позже в рамках вашего графика разработки.
*   [Network Ping Display](network-ping-display.md)

    Network Ping Display показывает время пинга для клиентов, используя OnGUI
*   [Network Rigidbody](network-rigidbody.md)

    Network Rigidbody синхронизирует скорость и другие свойства твердого тела по всей сети.
*   [Network Lerp Rigidbody](network-lerp-rigidbody.md)

    Network Lerp Rigidbody синхронизирует скорость и другие свойства твердого тела по всей сети.
* [Network Room Manager](network-room-manager.md)\
  Network Room Manager - это дополнительный компонент Network Manager, который предоставляет базовую функциональную комнату.
* [Network Room Player](network-room-player.md)\
  The Network Room Player это компонент, необходимый для prefab'ов игрока, используемых в сцене комнаты с описанным выше Network Room Manager.
* [Network Start Position](network-start-position.md)\
  Network Start Position используется Network Manager'ом при создании объектов проигрывателя. Положение и поворот начальной позиции сети используются для размещения вновь созданного объекта player.
* [Network Statistics](network-statistics.md)\
  Показывает сетевые сообщения и байты, отправляемые и принимаемые в секунду через OnGUI.
* [Network Transform](network-transform/)\
  Компонент Network Transform синхронизирует перемещение и вращение игровых объектов по сети. Обратите внимание, что компонент Network Transform синхронизирует только заспавненные сетевые игровые объекты.
