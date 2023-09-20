---
description: https://github.com/MirrorNetworking/LiteNetLibTransport/
---

# LiteNetLib Transport

**LiteNetLib транспорт для Mirror.**

## Использование <a href="#usage" id="usage"></a>

1. Скачайте unity package из [Releases ](https://github.com/MirrorNetworking/LiteNetLibTransport/releases)и импортируйте это в свой проект (Он не содержит Mirror, Mirror у вас уже должен быть)
2. Поставьте компонент `LiteNetLibTransport` на gameobject с компонентом NetworkManager и поставьте его заодно в поле транспорта в NetworkManager

## Features <a href="#features" id="features"></a>

* UDP
* Встроенный Network Discovery и UPnP
* Полностью управляемый код
* Не сильно нагружает CPU и RAM
* Маленький размер пакетов ( 1 байт для ненадежных пакетов, 3 байта для надежных )
* Различная механика отправки
* Reliable with order
* Reliable without order
* Упорядоченный, но ненадежный с предотвращением дублирования
* Простые UDP-пакеты без порядка и надежности
* Автоматическое объединение небольших пакетов
* Автоматическая фрагментация надежных пакетов
* Автоматическое обнаружение MTU
* Запросы времени NTP
* Симулирование потери пакетов и задержки
* Поддержка IPv6 (двойной режим)
* Статистика подключений (нужен DEBUG или STATS\_ENABLED флаги)
* Multicasting (для обнаружения хостов в локальной сети)

## IL2CPP Warning! <a href="#il2cpp-warning" id="il2cpp-warning"></a>

С IL2CPP, IPv6 поддерживается только в версии Unity 2018.3.6f1 и более поздних потому что:\
[Unity ChangeLog](https://unity3d.com/unity/whats-new/2018.3.6)

> IL2CPP: Добавлена поддержка протокола IPv6 в Windows. (1099133)
>
> IL2CPP: Корректная индикация о том, что IPv6 на неподдерживаемых-IPv6 платформах. (1108823)

Кроме того, опция повторного использования адреса сокета недоступна в IL2CPP.

## Титры <a href="#credits" id="credits"></a>

RevenantX - за [LiteNetLib](https://github.com/RevenantX/LiteNetLib/releases)\
vis2k & Paul - за [Mirror](https://assetstore.unity.com/packages/tools/network/mirror-129321)\
Coburn -за [Ignorance](https://github.com/SoftwareGuy/Ignorance) который я использовал в качестве примера\
Dankrushen - за то, что помог мне найти одну маленькую ошибку, которую я не мог найти в течение двух дней\
Lucas Ontivero - за [Open.Nat](https://github.com/lontivero/Open.NAT/releases), использованный для UPnP\
shiena - за NetworkDiscoveryHUD
