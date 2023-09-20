---
description: https://github.com/Chykary/FizzySteamworks/
---

# FizzySteamworks Transport

FizzySteamworks это Steam P2P транспорт для Mirror, он использует Steam's P2P сервис для прямого подключения или ретрансляции вашего соединения с другими игроками. FizzySteamworks основан на [Steamworks.Net](https://github.com/rlabrecque/Steamworks.NET).

Вы можете найти его релиз [**здесь**](https://github.com/Chykary/FizzySteamworks/releases) вместе с последней версией Steamworks.Net включительно или вы можете скачать с репозитория [**отсюда**](https://github.com/Chykary/FizzySteamworks).

## Features <a href="#features" id="features"></a>

* Несколько настраиваемых каналов: Вы можете настроить каналы в транспортном средстве, независимо от того, хотите ли вы использовать только 1 или 5 ненадежных каналов (лучше оставить нулевой канал надежным)..
* Steam Nat Punching & Relay : Транспорт будет использовать Steam для передачи данных по Nat в пункт назначения, и если это не сработает, будет использоваться ретрансляционный сервер steam, чтобы вы всегда могли подключаться (задержка может быть разной).
* Никаких изменений кода не требуется : Если вы уже используете Mirror, вам просто нужно подключить этот транспорт (возможно, добавить свой идентификатор приложения steam в свою сборку), и все должно работать так же, как и любой другой транспорт Mirror. "Это просто работает" - Todd Howard

![](<../../.gitbook/assets/image (70).png>)

## Титры <a href="#credits" id="credits"></a>

* [Fizz Cube](https://github.com/FizzCube) : Автор данного транспорта.
* [Chykary](https://github.com/Chykary/FizzySteamworks) : Текущий Мейнтейнер этого транспорта.
* [rlabrecque](https://github.com/rlabrecque) : Создатель Steamworks.Net.
* [vis2k](https://github.com/vis2k) : Создатель Mirror.
* Valve : Steam
