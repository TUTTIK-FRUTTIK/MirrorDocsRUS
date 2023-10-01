# SyncVars

SyncVars - это свойства классов, наследуемых от NetworkBehaviour, которые синхронизируются с сервера на клиенты. Когда создается игровой объект или новый игрок присоединяется к текущей игре, ему отправляется последнее состояние всех SyncVar сетевых объектов, которые ему видны. Используйте пользовательский атрибут `SyncVar` чтобы указать, какие переменные в вашем скрипте вы хотите синхронизировать.

Состояние SyncVars применяется к игровым объектам на клиентах перед вызовом `OnStartClient()`, поэтому состояние объектов всегда актуально внутри `OnStartClient()`.

SyncVars могут использовать любые [типы данных, поддерживаемые Mirror](../data-types.md). У вас может быть до 64 SyncVar в одном скрипте NetworkBehaviour, включая SyncLists (смотрите следующий раздел ниже).

Сервер автоматически отправляет обновления SyncVar при изменении значения SyncVar, поэтому вам не нужно отслеживать, когда они изменяются, или отправлять информацию об изменениях самостоятельно. Изменение значения в инспекторе не вызовет обновления.

> Атрибут [SyncVar hook](syncvar-hooks.md) может использоваться для указания метода, который будет вызываться при изменении значения SyncVar на клиенте.

## Пример использования SyncVar <a href="#syncvar-example" id="syncvar-example"></a>

Допустим, у нас есть сетевой объект со скриптом под названием Enemy:

```csharp
public class Enemy : NetworkBehaviour
{
    [SyncVar]
    public int health = 100;

    void OnMouseUp()
    {
        NetworkIdentity ni = NetworkClient.connection.identity;
        PlayerController pc = ni.GetComponent<PlayerController>();
        pc.currentTarget = gameObject;
    }
}
```

`PlayerController` может выглядеть примерно так:

```csharp
public class PlayerController : NetworkBehaviour
{
    public GameObject currentTarget;

    void Update()
    {
        if (isLocalPlayer)
            if (currentTarget != null &&currentTarget.tag == "Enemy")
                if (Input.GetKeyDown(KeyCode.X))
                    CmdShoot(currentTarget);
    }

    [Command]
    public void CmdShoot(GameObject enemy)
    {
        // Здесь отличное место чтобы выполнить проверку на попадание после выстрела
        // так как данный метод выполняется на сервере
        enemy.GetComponent<Enemy>().health -= 5;
    }
}
```

В этом примере, когда игрок нажимает на врага, игровой объект врага присваивается переменной `PlayerController.currentTarget`. Когда игрок нажимает на клавишу X с выбранной целью, эта цель передается в метод \[Command], который выполняется на сервере, для вычисления `health` SyncVar. У всех клиентов обновится значение ХП у этого врага. Затем вы можете создать пользовательский интерфейс на враге (к примеру полоску ХП), чтобы показывать его текущее значение ХП.

## Наследование классов <a href="#class-inheritance" id="class-inheritance"></a>

SyncVars работает с наследованием классов. Рассмотрим этот пример:

```csharp
class Pet : NetworkBehaviour
{
    [SyncVar] 
    string name;
}

class Cat : Pet
{
    [SyncVar]
    public Color32 color;
}
```

Вы можете перетащить компонент Cat на ваш Prefab кота и у него будут синхронизироваться оба его `name` и `color`.

> **Внимание:** Оба`Cat` и `Pet` должно быть в том же билде. Если они находятся в разных билдах, убедитесь, что `name` не изменяется внутри `Cat`, вместо этого добавьте метод классу `Pet`.
