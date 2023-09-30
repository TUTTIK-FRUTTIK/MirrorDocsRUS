---
description: Custom Interest Management
---

# Custom

## Custom Interest Management

**Mirror** позволяет вам реализовать пользовательские Interest Management решения. Например:

* На основе **Raycast**, чтобы игроки DotA не видели других игроков за стеной или в кустах
* **Прогнозирующий Raycast** для игр, подобных Counter-Strike. Wallhack'и показывают игроков за стенами. Вы могли бы создать пользовательскую систему управления интересами, чтобы отправлять врагов игроку только незадолго до того, как они выскочат из-за стены. В шутерах от первого лица игроки двигаются быстро, поэтому вам все равно придется отправить им пару миллисекунд, прежде чем они станут видимыми.

Все вышеперечисленные индивидуальные решения возможны в Mirror. Чтобы понять, как может быть реализован Interest Management, давайте пройдемся по нему шаг за шагом.

### Шаблон скрипта

Mirror включает в себя [**шаблон скрипта**](../general/script-templates.md) для собственного interest management. Он полностью прокомментирован со всеми переопределениями виртуальных методов, которые уже были отключены для вас. Если вы раньше пользовались нашим устаревшим Interest Management, то они должны показаться вам знакомыми.

* **OnCheckObserver** вызывается, когда кто-то спавнится. Возвращает значение true, если "identity" может быть виден "newObserver"
* **OnRebuildObservers** перестраивает наблюдателей для заданного **Network Identity**. Результат сохраняется в **newObservers**.
  * Mirror автоматически поместит **newObservers** внутрь **identity.observers**. Мы не делаем этого напрямую, потому что это немного сложнее, чем добавление / удаление. Mirror позаботится об этом. Беспокоиться не о чем :)
* **RebuildAll** это вспомогательная функция для перестройки каждого **заспавненного** Network Identity наблюдателя.
  * Ваши реализации, вероятно, захотят вызывать это каждый интервал.

### **на примере Distance**

Distance Interest Management это самая простая и прямолинейная реализация. Давайте пройдемся по нему, чтобы увидеть, как наследовать от абстрактного класса `InterestManagement`.

```csharp
public class DistanceInterestManagement : InterestManagement
{
    [Tooltip("The maximum range that objects will be visible at.")]
    public int visRange = 10;

    [Tooltip("Rebuild all every 'rebuildInterval' seconds.")]
    public float rebuildInterval = 1;
    double lastRebuildTime;

    public override bool OnCheckObserver(NetworkIdentity identity, NetworkConnection newObserver)
    {
        return Vector3.Distance(identity.transform.position, newObserver.identity.transform.position) <= visRange;
    }

    public override void OnRebuildObservers(NetworkIdentity identity, HashSet<NetworkConnection> newObservers, bool initialize)
    {
        Vector3 position = identity.transform.position;
        
        // для каждого соединения
        foreach (NetworkConnectionToClient conn in NetworkServer.connections.Values)
            // если прошел аутентификацию и присоединился к миру
            if (conn != null && conn.isAuthenticated && conn.identity != null)
                // проверить расстояние до нашего 'identity'
                if (Vector3.Distance(conn.identity.transform.position, position) < visRange)
                    // добавить к результату
                    newObservers.Add(conn);
    }

    [ServerCallback]
    void Update()
    {
        // перестраивать всех заспавненных наблюдателей NetworkIdentity каждый интервал времени
        if (NetworkTime.time >= lastRebuildTime + rebuildInterval)
        {
            RebuildAll();
            lastRebuildTime = NetworkTime.time;
        }
    }
}
```

Это не слишком сложно, не так ли?

* **OnCheckObserver** просто сравнивает расстояние между заспавненным объектом и соединением. У соединения есть основной игрок, которого мы используем здесь для проверки расстояния.
  * _Обратите внимание, что вы также могли бы проверить каждый объект, которым владеет соединение. Например, если Боб появляется, а Алиса находится недостаточно близко, питомец Алисы может быть достаточно близко, и поэтому Алиса и Боб все равно должны видеть друг друга, несмотря на то, что находятся немного за пределами нормального диапазона._
* **OnRebuildObservers** его задача состоит в том, чтобы вернуть HashSet внутри которого хранятся **Network Connection**s которые могут видеть наш **Network Identity**. Итак, очевидно, что мы просто итерируем все **NetworkServer.connections** и проверяем расстояние их основного игрока до нашего Network Identity.
  * _Note that we only check the ones that are authenticated and have a player in the world. Connections that are still logging in or choosing characters shouldn't observe anything._
* **Update** его задача состоит в том, чтобы время от времени вызывать **RebuildAll()**. Если мы не вызываем **RebuildAll()**, тогда Mirror никогда не будет перестраивать наблюдателей.

### Host Mode Visibility

В режиме хоста кто-то запускает сервер, одновременно играя на нем сам, так что вы можете подумать:

* **Я - сервер**. **Сервер видит всех**. **Поэтому я должен видеть всех**.

_Технически_ это верно, но если вам посчастливилось когда-либо побывать в **локальной** игре, то вы запомните ее по-другому:

![Лучшие деньки.](<../../.gitbook/assets/image (63).png>)

Например, кто-то в локальной сети проводит игру Counter-Strike или DotA. Давайте на минутку задумаемся об этом случае:

* **Хост** запускает **сервер**. **Сервер** хранит в памяти **всё состояние мира**, тем не менее, **хост игрок** видит только мир вокруг себя.

Идея заключается в том, чтобы хост игрок был постоянным участником игры. Локальные игры были бы не очень веселыми, если бы вы играли в DotA / Counter Strike, а хост всегда видел позицию всех остальных, верно?

{% hint style="info" %}
**Очевидно, что хост может читерить.** Если вы читерите в локальной игре, то вам нужна профессиональная помощь.
{% endhint %}

Mirror имеет виртуальный метод `SetHostVisibility(NetworkIdentity, bool)` который включает / отключает средства визуализации в режиме хоста. Другими словами, состояние мира все еще существует - принимающий игрок просто не видит его. Вы можете переопределить это в своей пользовательской системе в соответствии с вашими потребностями.
