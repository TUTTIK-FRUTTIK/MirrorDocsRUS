# Timestamp Batching

Давайте узнаем, как Mirror отправляет сообщения.

## Batching

Каждое отправляемое вами сообщение будет обрабатываться пакетно до конца кадра, чтобы минимизировать пропускную способность и транспортные вызовы. К примеру, если вы отправляете много сообщений по 10 байтов, то мы могли бы вместо этого отправить одно размером в \~120 байт.

Что касается транспорта, то довольно удобно отправлять сообщения порциями по 1200 байт _(смотреть_ [_MTU_](https://en.wikipedia.org/wiki/Maximum\_transmission\_unit)_)_. Сообщения, размер которых превышает **MTU**, отправляются одним пакетом. Если быть точным, транспорт определяет размер партии, к которой стремится Mirror, с помощью `Transport.GetBatchThreshold()`.

Mirror batching является двунаправленным. Это означает, что и клиент, и сервер пакуют свои сообщения и отправляют их в конце кадра.

{% hint style="success" %}
Короче говоря, пакетная обработка значительно сокращает пропускную способность и повышает производительность.
{% endhint %}

## Timestamps (метки времени)

Для некоторых сетевых компонентов полезно точно знать, когда сообщение было отправлено удаленным устройством.

К примеру, `NetworkTransform` получает позицию на сервере и затем выполняет интерполяцию между ними. Для плавной интерполяции нам нужно точно воссоздать то, что произошло на сервере. Для этого нам нужно знать, когда объект находился в определенном положении на сервере.

Очевидное решение - просто отправлять оба `timestamp` и `position` всё время:

```csharp
[Rpc]
public void RpcPositionUpdate(float timestamp, Vector3 position)
{
    // ...
}
```

На самом деле, именно так выглядит ранняя версия нашего нового `NetworkTransform` компонента.

В приведенном выше коде мы тратим большую часть пропускной способности, потому что для каждого сообщения с позицией нам также нужно добавлять 4-ех байтное значение `float` (или даже лучше, 8 байтов `double` для лучшей точности). При синхронизации больших миров пропускная способность быстро увеличивалась бы.

`NetworkTransform` это только один из многих компонентов. Несколько остальным тоже нужны такие TimeStamps, что еще больше увеличило бы пропускную способность.

Чтобы сделать жизнь легче, Mirror включает _8 байт_ `double` _для точности и_ `timestamp` в каждой обработке (Batch). Вместо добавления этого в каждое сообщение, мы включаем это раз в \~1200 байтовую обработку, что едва сказывается на пропускной способности.

Для любого обработчика сообщений в Mirror, вы можете взять timestamp из обработки прибывшее из `NetworkConnection.remoteTimeStamp`.

* на **клиенте**, все данные объектов поступают с сервера в виде сообщений/обработок. Таким образом, в любой момент времени, вы можете взять данные из `Rpc`/`OnDeserialize`/`OnMessage` обработчика, который был отправлен сервером через `NetworkClient.connection.remoteTimeStamp`.
  * Обратите внимание, что на клиенте мы не используем `connectionToServer` потому что только объекты, принадлежащие игроку, имеют подключение к серверу. Вместо этого мы используем на клиенте `NetworkClient.connection` к серверу, который всегда будет гарантированно там находиться.
* На **сервере**, только игрок владеющий объектом получает сообщения от подключенных игроков. Таким образом, в любой момент времени вы можете найти данные, находящиеся в `Cmd`/`OnDeserialize`/`OnMessage` перехватчике, отправленные клиентом посредством `connectionToClient.remoteTimeStamp`.

{% hint style="info" %}
**Timestamp Batching** это уникальный подход для синхронизации мира в Mirror.\
\
Например, Delta Snapshots в Quake идеально подходят для игр в жанре FPS, где весь мир помещается в одно сообщение о состоянии мира, но не идеальны для более крупных MMO-миров с большим количеством объектов. Или, возможно, вы работаете над многопользовательским текстовым приключением, в котором практически нет состояния мира, но все равно много сетевых сообщений.\
\
**Timestamp Batching** хорошо вписывается в архитектуру Mirror. Это должно помочь вам сократить пропускную способность независимо от того, над каким типом проекта вы работаете.
{% endhint %}
