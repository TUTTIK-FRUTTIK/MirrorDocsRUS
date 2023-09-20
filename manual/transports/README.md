# Транспорты

## Встроенные транспорты

Здесь собраны все транспорты, включенные в Mirror.

* [KCP ](kcp-transport.md)UDP транспорт основанный на kcp.c, line-by-line трансляция в C#
* [Telepathy](telepathy-transport.md) - Простой, основанный на сообщениях, MMO-масштабируемая TCP-сеть на C#. И никакой магии.
* [Simple Web Sockets](websockets-transport/) - WebGL транспорт Mirror предназначенный для браузерных клиентов.
* [Multiplexer](multiplex-transport.md) - Соединительный транспорт, позволяющий серверу одновременно обрабатывать клиентов на разных транспортах, например настольных клиентах, использующих Telepathy, вместе с клиентами WebGL, использующими Websockets.
* [Latency Simulation](latency-simulaton-transport.md) - Транспорт посредник для тестирования в неидеальных условиях работы сети

## Дополнительные транспорты

Эти перевозки осуществляются третьими лицами за пределами Mirror.

* [Monke](https://github.com/JesusLuvsYooh/monke) - plug and play зашифрованный транспорт посредник для mirror.
* [Ignorance](ignorance.md) - надежный и ненадежный последовательный UDP-транспорт основанный на ENet.
* [LiteNetLibTransport](litenetlib-transport.md) - UDP транспорт основанный на [LiteNetLib](https://github.com/RevenantX/LiteNetLib).

## Ретрансляционные транспорты

Эти транспорты поддерживаются третьими сторонами и используют инфраструктуру ретрансляции для подключения клиентов к серверам за брандмауэрами / NAT.

* [Steam - FizzySteamworks](fizzysteamworks-transport.md) - Транспорт использующий Steam P2P network, построенный на Steamworks.NET.
* [Steam - FizzyFacepunch](fizzyfacepunch-transport.md) - Транспорт использующий Steam P2P network, построенный на Facepunch.Steamworks.
* [Epic - Epic Online Services](https://github.com/FakeByte/EpicOnlineTransport) - Ретрансляционный транспорт использующий Epic's free relay service.
* [LRM - Light Reflective Mirror](https://github.com/Derek-R-S/Light-Reflective-Mirror) - Ретрансляционный транспорт для WebGL клиентов.
* [OculusP2P - Oculus Platform](https://github.com/hyferg/MirrorOculusP2P) - Ретрансляционный транспорт для Oculus Quest 1 & 2.

## Смена транспорта

Смена транспорта очень проста и требует всего нескольких шагов:

* Откройте сцену и найдите объект имеющий Network Manager компонент
* Добавьте нужный вам компонент транспорта через кнопку Add Component
* Затем в этот же висячий компонент транспорта перетащите в поле "Transport" у компонента Network Manager
* Удалите старый компонент транспорта (опционально)

Если у вас возникли проблемы с подключением к транспорту, требующему переадресации портов, убедитесь, что для переадресации портов используется правильный протокол (TCP / UDP).
