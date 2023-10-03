# Пользовательские функции спавна

Вы можете использовать функции обработчика спавна, чтобы настроить поведение объектов при спавне созданных игровых объектов на клиенте. Функции обработчика спавна гарантируют, что у вас есть полный контроль над тем, как вы создаете игровой объект, а также как вы его уничтожаете.

Используйте `NetworkClient.RegisterSpawnHandler` или `NetworkClient.RegisterPrefab` чтобы регистрировать функции для спавна и уничтожения игровых объектов на клиенте. Сервер спавнит игровые объекты напрямую, а затем спавнит их на клиентах с помощью этой функции. Эти функции принимает либо asset ID, либо prefab и две функции делегата: одна обрабатывает создание объекта на клиенте, и одна обрабатывает уничтожение объекта на клиенте. Asset ID может быть динамичным, или просто найденным на prefab'е игрового объекта который вы хотите заспавнить.

Делегаты Spawn / Unspawn будут выглядеть примерно так:

**Spawn Handler**

```csharp
GameObject SpawnDelegate(Vector3 position, System.Guid assetId) 
{
    // делайте что-нибудь здесь
}
```

или

```csharp
GameObject SpawnDelegate(SpawnMessage msg) 
{
    // делайте что-нибудь здесь
}
```

**UnSpawn Handler**

```csharp
void UnSpawnDelegate(GameObject spawned) 
{
    // делайте что-нибудь здесь
}
```

Когда prefab сохраняется, его поле `assetId` будет назначено автоматически. Если вы хотите создавать prefab'ы во время выполнения, вам придется сгенерировать новый GUID.

### **Генерация Prefab'а в реальном времени**

```csharp
// генерация нового уникального assetId 
System.Guid creatureAssetId = System.Guid.NewGuid();

// регистрация обработчика для нового assetId
NetworkClient.RegisterSpawnHandler(creatureAssetId, SpawnCreature, UnSpawnCreature);
```

**Использование существующего prefab'а**

```csharp
// зарегистрируйте prefab у которого будет пользовательский спавн и передайте в обработчик
NetworkClient.RegisterPrefab(coinAssetId, SpawnCoin, UnSpawnCoin);
```

**Spawn on Server**

```csharp
// спавн монеты - SpawnCoin вызывается на клиенте
NetworkServer.Spawn(gameObject, coinAssetId);
```

Сами функции спавна реализуются с помощью подписки делегата. Вот это спавнер монет. The `SpawnCreature` would look the same, but have different spawn logic:

```csharp
public GameObject SpawnCoin(SpawnMessage msg)
{
    return Instantiate(m_CoinPrefab, msg.position, msg.rotation);
}

public void UnSpawnCoin(GameObject spawned)
{
    Destroy(spawned);
}
```

При использовании пользовательских функций спавна иногда полезно иметь возможность отменять спавн игровых объектов, не уничтожая их. Это можно сделать, вызвав `NetworkServer.UnSpawn`. Это приводит к тому, что объект становится `Reset` на сервере и отправляет `ObjectDestroyMessage` клиентам. `ObjectDestroyMessage` приведет к вызову пользовательской функции unspawn на клиентах. Если нет функции unspawn, то вместо этого объект будет уничтожаться (`Destroy`)

Обратите внимание, что на хосте игровые объекты не создаются для локального клиента, поскольку они уже существуют на сервере. Это также означает, что никакие функции spawn или unspawn-обработчика не будут вызываться.

### [Pooling Game Objects](https://ru.wikipedia.org/wiki/%D0%9E%D0%B1%D1%8A%D0%B5%D0%BA%D1%82%D0%BD%D1%8B%D0%B9\_%D0%BF%D1%83%D0%BB)

Чтобы избежать таких методов как Instantiate и Destroy часто используя GameObjects, вместо этого может быть полезно отправить их в пул.

![](../../../.gitbook/assets/2022-04-04\_20-21-49@2x.png)

Вот пример того, как вы могли бы настроить простую систему пула игровых объектов с пользовательскими обработчиками появления. При появлении и отмене появления игровые объекты помещаются в пул или извлекаются из него.

```csharp
using UnityEngine;

namespace Mirror.Examples
{
    public class PrefabPool : MonoBehaviour
    {
        // singleton для облегчения доступа из других скриптов
        public static PrefabPool singleton;

        [Header("Settings")]
        public GameObject prefab;

        [Header("Debug")]
        public int currentCount;
        public Pool<GameObject> pool;

        void Start()
        {
            InitializePool();
            singleton = this;
            NetworkClient.RegisterPrefab(prefab, SpawnHandler, UnspawnHandler);
        }

        // используется при помощи NetworkClient.RegisterPrefab
        GameObject SpawnHandler(SpawnMessage msg) => Get(msg.position, msg.rotation);

        // используется при помощи NetworkClient.RegisterPrefab
        void UnspawnHandler(GameObject spawned) => Return(spawned);

        void OnDestroy()
        {
            NetworkClient.UnregisterPrefab(prefab);
        }

        void InitializePool()
        {
            // создайте пул с функцией генерации
            pool = new Pool<GameObject>(CreateNew, 5);
        }

        GameObject CreateNew()
        {
            // используйте этот объект в качестве родительского, чтобы объекты не загромождали иерархию
            GameObject next = Instantiate(prefab, transform);
            next.name = $"{prefab.name}_pooled_{currentCount}";
            next.SetActive(false);
            currentCount++;
            return next;
        }

        // Используется для извлечения объекта из пула.
        // Должен использоваться на сервере для получения следующего объекта
        // Используется на клиенте NetworkClient для создания объектов
        public GameObject Get(Vector3 position, Quaternion rotation)
        {
            GameObject next = pool.Get();

            // установите положение /поворот и установите объект активным
            next.transform.position = position;
            next.transform.rotation = rotation;
            next.SetActive(true);
            return next;
        }

        // Используется для помещения объекта обратно в пул
        // Должен использоваться на сервере после деспавна объекта
        // Используется на клиенте NetworkClient для деспавна объекта
        public void Return(GameObject spawned)
        {
            // выключить объект
            spawned.SetActive(false);

            // добавить обратно в пул
            pool.Return(spawned);
        }
    }
}

```

Чтобы использовать пул, добавьте компонент `PrefabPool` (приведенный выше код) к NetworkManager'у. Затем перетащите prefab, который вы хотите создать несколько раз, в поле Prefab.

{% hint style="warning" %}
Убедитесь что вы удалили Prefab из списка NetworkManager'а spawnable prefabs. Должен быть только один способ заспавнить его. В противном случае Mirror выдаст предупреждение.
{% endhint %}

Мы можем модифицировать наш пример Tanks чтобы продемонстрировать вам систему пулинга.

Откройте Tank.cs и найдите функцию CmdFire:

```csharp
[Command]
void CmdFire()
{
    GameObject projectile = Instantiate(projectilePrefab, projectileMount.position, projectileMount.rotation);
    NetworkServer.Spawn(projectile);
    RpcOnFire();
}
```

Вместо функции Instantiate, вытащите Prefab из пула:

```csharp
[Command]
void CmdFire()
{
    GameObject projectile = PrefabPool.singleton.Get(projectileMount.position, projectileMount.rotation);
    NetworkServer.Spawn(projectile);
    RpcOnFire();
}
```

Projectile.cs в данный момент самоуничтожаются посредством GameObject.Destroy:

```csharp
[Server]
void DestroySelf()
{
    NetworkServer.Destroy(gameObject);
}
```

Вместо этого мы можем просто задеспанить объект, вернув его в наш пул:

```csharp
[Server]
void DestroySelf()
{
    // возвращает prefab в пул
    NetworkServer.UnSpawn(gameObject);
    PrefabPool.singleton.Return(gameObject);
}
```

Нажмите кнопку "играть" и выпустите несколько снарядов. Обратите внимание, что ничего не создается (Instantiate). Вместо этого, NetworkManager имеет пул отключенных объектов-детей, которые он использует когда они нужны.

![](../../../.gitbook/assets/2022-04-04\_20-22-58@2x.png)
