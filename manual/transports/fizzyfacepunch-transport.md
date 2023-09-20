---
description: https://github.com/Chykary/FizzyFacepunch/
---

# FizzyFacepunch Transport

FizzyFacepunch это Steam P2P транспорт для Mirror, он использует Steam's P2P сервис для прямого подключения или ретрансляции вашего соединения с другими игроками. FizzyFacepunch основан на [Facepunch.Steamworks](https://github.com/Facepunch/Facepunch.Steamworks).

Вы можете скачать его [**Здесь**](https://github.com/Chykary/FizzyFacepunch/releases) или вы можете установить его из репозитория [**отсюда**](https://github.com/Chykary/FizzyFacepunch).

## Features <a href="#features" id="features"></a>

* Несколько настраиваемых каналов: Вы можете настроить каналы в транспортном средстве, независимо от того, хотите ли вы использовать только 1 или 5 ненадежных каналов (лучше оставить нулевой канал надежным)..
* Steam Nat Punching & Relay : Транспорт будет использовать Steam для передачи данных по Nat в пункт назначения, и если это не сработает, будет использоваться ретрансляционный сервер steam, чтобы вы всегда могли подключаться (задержка может быть разной).
* Никаких изменений кода не требуется : Если вы уже используете Mirror, вам просто нужно подключить этот транспорт (возможно, добавить свой идентификатор приложения steam в свою сборку), и все должно работать так же, как и любой другой транспорт Mirror. "Это просто работает" - Todd Howard

![](<../../.gitbook/assets/image (117).png>)

## Credits <a href="#credits" id="credits"></a>

* [Chykary](https://github.com/Chykary/FizzyFacepunch) : Автор данного транспорта.
* [Facepunch](https://github.com/Facepunch) : Создатель Facepunch.Steamworks.
* [vis2k](https://github.com/vis2k) : Создатель Mirror.
* Valve : Steam
