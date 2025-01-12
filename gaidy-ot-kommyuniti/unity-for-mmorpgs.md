---
description: >-
  Копия нашего ответа пользователю Unity, спрашивающему об использовании Unity
  для MMO
---

# Unity для MMORPG

Многие пользователи хотят создавать MMORPG с Unity / Mirror. Хотя Unity - не лучший выбор для игровых серверов, он дает большие преимущества. Этот пост был нашим ответом пользователю на форумах Unity, который спрашивал о [MMOs & Unity](https://forum.unity.com/threads/mmo-server-with-unity.1410480/#post-8868246). Не забудьте также ознакомиться с нашим очень старым гайдом "[How to make an MMORPG](https://noobtuts.com/articles/how-to-make-a-mmorpg)", что, по сути, и привело к появлению Mirror в первую очередь.\
\
\[...]

Причина, по которой вы бы использовали Unity для клиента и сервера, заключается в производительности. Совместное использование физики, ресурсов, навигации и кода экономит вам массу времени. Таким образом было создано несколько небольших MMO, и вы можете настроить таргетинг примерно на 200-300 CCU на экземпляр Unity server. Например, Inferna, Samutale и Naica - это ММО, которые были полностью созданы в Unity (Mirror)..\
[![](https://forum.unity.com/proxy.php?image=https%3A%2F%2Fuser-images.githubusercontent.com%2F16416509%2F178149040-b54e0fa1-3c41-4925-8428-efd0526f8d44.jpg\&hash=06e9b9a127560446c7522abeb8b17e09)](https://forum.unity.com/proxy.php?image=https%3A%2F%2Fuser-images.githubusercontent.com%2F16416509%2F178149040-b54e0fa1-3c41-4925-8428-efd0526f8d44.jpg\&hash=06e9b9a127560446c7522abeb8b17e09)\
\
Обратите внимание, что чем ниже ваш тикрейт, тем большего CCU вы можете достичь. Например, если ваш сервер работает на частоте 60 Гц, это дает вам 16 мс для обновления всего мира. Если ваш сервер работает на частоте 10 Гц, это дает вам 100 мс. Имея в 6 раз больше времени на обновление, вы, вероятно, сможете получить в 2-3 раза больше CCU.\
\
Именно так столько ранних MMO достигли 500+ CCU уже 15 лет назад. Многие из них работали на частоте 10 Гц или меньше. Недостатком является дополнительная задержка. Но, эй, 500 CCU :smile:.\\

Имейте в виду, что Unity - не самый лучший выбор для игровых серверов, если вы заботитесь о надежности, стабильности, масштабе. Автономные проекты на C# (за пределами Unity) могут запускаться на netcore, что значительно быстрее и стабильнее.\
\
За пределами Unity у вас также есть больше возможностей выбора языков программирования, которые могут лучше подходить для крупномасштабных / высокопроизводительных серверов.\\

* Go стоит того, чтобы ему научиться. Конкурентоспособный язык, разработанный Google для масштабируемых систем. Гораздо проще выполнять многопоточность, чем с C#, благодаря go-процедурам. Также собирается мусор, как например C#.\\
* Rust тоже хорош. Простая производительность как у C / C++, но с сохранением памяти. В многопоточности тоже хорошо себя показывает, потому что Rust защищает вас от скачков данных (в отличие от Go и C#). Не собирается мусор, так что не будет пауз GC, о которых стоило бы беспокоиться.
* C# по-прежнему остается отличным выбором. Однако некоторые из настроек сети низкого уровня по умолчанию не очень хороши. Например, [здесь](https://github.com/dotnet/runtime/issues/30797) продолжается дискуссия о распределении сокетов C# UDP. Для выделения ресурсов требуется сборка мусора, что приводит к паузам GC, что приводит к проблемам с производительностью на крупномасштабных серверах.

\
Стоит отметить, что Unity находится в процессе перехода на netcore. Это было бы здорово для игровых серверов!\
\
**TL;DR:**\
Unity - самое медленное и нестабильное решение для игровых серверов. Но это также и самое продуктивное решение.\
\
Выбери что-нибудь одно :thumbsup:
