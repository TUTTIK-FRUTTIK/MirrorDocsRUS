---
description: https://github.com/vis2k/Telepathy
---

# Telepathy Transport

Простой, основанный на сообщениях, MMO-масштабируемый TCP на C#. И никакой магии.

* Telepathy в уме был спроектирован с [KISS Principle](https://en.wikipedia.org/wiki/KISS\_principle).
* Telepathy быстрый и очень стабильный, спроектирован для [MMO](https://assetstore.unity.com/packages/templates/systems/ummorpg-51212) масштабируемой сети.
* Telepathy использует фрейминг, поэтому все отправленное будет получено таким же образом.
* Telepathy написан на C# и также может быть использован в Unity3D.
* Telepathy доступен на [GitHub](https://github.com/vis2k/Telepathy)

## Что делает Telepathy особенным? <a href="#what-makes-telepathy-special" id="what-makes-telepathy-special"></a>

Telepathy был спроектирован для [uMMORPG](https://assetstore.unity.com/packages/templates/systems/ummorpg-51212) после 3 лет в аду UDP.

Нам нужна была библиотека, которая была бы:

* Стабильной и с отсутствием ошибок: Telepathy использовало только 400 строчек кода. Никакой магии.
* С высокой производительностью: Telepathy может обрабатывать тысячи подключений и пакетов.
* Конкурентноспособной: Telepathy использует один поток для каждого соединения. Он может интенсивно использовать многоядерные процессоры.
* Простой: Telepathy сам заботится обо всём. Всё что нужно, так это вызвать Connect/GetNextMessage/Disconnect.
* Основанная на сообщениях: если мы отправляем 10, а затем 2 байта, то другой конец получает 10, а затем 2 байта, но никогда не 12 сразу.

MMORPG безумно сложно создать, и мы создали Telepathy, чтобы нам больше никогда не пришлось беспокоиться о низкоуровневых сетях.

## Что насчет... <a href="#what-about" id="what-about"></a>

* Асинхронные сокеты: не показали лучших результатов в наших тестах.
* ConcurrentQueue: совместимость с .NET 3.5 важна для Unity. В любом случае это было не быстрее нашего SafeQueue.
* UDP vs. TCP: Minecraft и World of Warcraft - две крупнейшие многопользовательские игры всех времен, и обе они используют сеть TCP. На это есть причины.
