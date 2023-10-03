# Поднятие, Бросание, и дочерние объекты

Часто возникает вопрос о том, как обращаться с объектами, которые прикреплены в качестве дочерних элементов Prefab'a игрока, о которых все клиенты должны знать и синхронизировать, например, какое оружие оснащено, сбор объектов сетевой сцены и игроки, бросающие объекты в сцене.

{% hint style="warning" %}
Mirror не поддерживает сразу несколько компонентов Network Identity в одной иерархии объекта. Поскольку объект игрока должен иметь Network Identity, ни один из его дочерних объектов не может его иметь.
{% endhint %}

## Дочерние объекты <a href="#child-objects" id="child-objects"></a>

Давайте начнем с простого случая с единственной точкой прикрепления, которая находится где-то внизу иерархии нашего игрока, например, с кистью на конце руки. В сценарии, который наследуется от NetworkBehaviour в Prefab'e игрока, у нас была бы ссылка на `GameObject`, где точка прикрепления может быть назначена в инспекторе, перечисление SyncVar с различными вариантами того, что держит игрок, и hook у SyncVar для отображения удерживаемого элемента на основе нового значения.

На изображении ниже у Кайла есть пустой игровой объект, `RightHand`, добавленный к запястью, а также несколько готовых элементов для экипировки (мяч, коробка, цилиндр) и сценарий экипировки игрока для работы с ними.

{% hint style="info" %}
**ПРИМЕЧАНИЕ**: Prefab'ы предметов это _только визуальная часть_... у них нет скриптов, и у них _не должно_ быть сетевых компонентов. Конечно, у них могут быть скрипты, основанные на monobehaviour, на которые можно ссылаться и вызывать из ClientRpc в Prefab'e игрока.
{% endhint %}

Инспектор показывает `RightHand` расположенный в 2 местах, скрипт оснащения игрока, а также цель дочернего компонента Network Transform, чтобы мы могли при необходимости корректировать относительное положение точки подключения (не визуально) для всех клиентов.

![](<../../../.gitbook/assets/image (114).png>)

Ниже приведен сценарий экипировки игрока для обработки смены экипированного предмета, а также некоторые примечания для рассмотрения:

* Хотя мы могли бы просто прикрепить все визуальные элементы во время разработки и просто включать / отключать их на основе перечисления, это бы не очень хорошо масштабировалось для многих элементов, и если на них есть скрипты для того, как они будут вести себя в игре, к примеру для анимации, спецэффектов и т.д. это может стать уродливым довольно быстро, поэтому этот пример локально создает экземпляры и уничтожает их вместо этого в качестве выбора визуализации.
* В примере не предпринимается никаких усилий для устранения смещения положения между предметом и точкой крепления, например, для выравнивания захвата или рукоятки предмета по руке. С этим лучше всего справиться в сценарии monobehaviour для элемента, который имеет общедоступные поля для локального положения и поворота, которые можно задать в конструкторе, и немного кода в Start, чтобы применить эти значения в локальных координатах относительно родительской точки присоединения.

```csharp
using UnityEngine;
using System.Collections;
using Mirror;

public enum EquippedItem : byte
{
    nothing,
    ball,
    box,
    cylinder
}

public class PlayerEquip : NetworkBehaviour
{
    public GameObject sceneObjectPrefab;

    public GameObject rightHand;

    public GameObject ballPrefab;
    public GameObject boxPrefab;
    public GameObject cylinderPrefab;

    [SyncVar(hook = nameof(OnChangeEquipment))]
    public EquippedItem equippedItem;

    void OnChangeEquipment(EquippedItem oldEquippedItem, EquippedItem newEquippedItem)
    {
        StartCoroutine(ChangeEquipment(newEquippedItem));
    }

    // Since Destroy is delayed to the end of the current frame, we use a coroutine
    // to clear out any child objects before instantiating the new one
    IEnumerator ChangeEquipment(EquippedItem newEquippedItem)
    {
        while (rightHand.transform.childCount > 0)
        {
            Destroy(rightHand.transform.GetChild(0).gameObject);
            yield return null;
        }

        switch (newEquippedItem)
        {
            case EquippedItem.ball:
                Instantiate(ballPrefab, rightHand.transform);
                break;
            case EquippedItem.box:
                Instantiate(boxPrefab, rightHand.transform);
                break;
            case EquippedItem.cylinder:
                Instantiate(cylinderPrefab, rightHand.transform);
                break;
        }
    }

    void Update()
    {
        if (!isLocalPlayer) return;

        if (Input.GetKeyDown(KeyCode.Alpha0) && equippedItem != EquippedItem.nothing)
            CmdChangeEquippedItem(EquippedItem.nothing);
        if (Input.GetKeyDown(KeyCode.Alpha1) && equippedItem != EquippedItem.ball)
            CmdChangeEquippedItem(EquippedItem.ball);
        if (Input.GetKeyDown(KeyCode.Alpha2) && equippedItem != EquippedItem.box)
            CmdChangeEquippedItem(EquippedItem.box);
        if (Input.GetKeyDown(KeyCode.Alpha3) && equippedItem != EquippedItem.cylinder)
            CmdChangeEquippedItem(EquippedItem.cylinder);
    }

    [Command]
    void CmdChangeEquippedItem(EquippedItem selectedItem)
    {
        equippedItem = selectedItem;
    }
}
```

## Выбрасывание предметов <a href="#dropping-items" id="dropping-items"></a>

Теперь, когда мы можем экипировать предметы, нам нужен способ отправить текущий предмет в мир как сетевой предмет. Помните, что, как дочерний объект визуализации, Prefab'ы вообще не содержат сетевых компонентов.

Во-первых, давайте добавим ввод в методе Update и метод `CmdDropItem`:

```csharp
    void Update()
    {
        if (!isLocalPlayer) return;

        if (Input.GetKeyDown(KeyCode.Alpha0) && equippedItem != EquippedItem.nothing)
            CmdChangeEquippedItem(EquippedItem.nothing);
        if (Input.GetKeyDown(KeyCode.Alpha1) && equippedItem != EquippedItem.ball)
            CmdChangeEquippedItem(EquippedItem.ball);
        if (Input.GetKeyDown(KeyCode.Alpha2) && equippedItem != EquippedItem.box)
            CmdChangeEquippedItem(EquippedItem.box);
        if (Input.GetKeyDown(KeyCode.Alpha3) && equippedItem != EquippedItem.cylinder)
            CmdChangeEquippedItem(EquippedItem.cylinder);

        if (Input.GetKeyDown(KeyCode.X) && equippedItem != EquippedItem.nothing)
            CmdDropItem();
    }
```

```csharp
    [Command]
    void CmdDropItem()
    {
        // Instantiate the scene object on the server
        Vector3 pos = rightHand.transform.position;
        Quaternion rot = rightHand.transform.rotation;
        GameObject newSceneObject = Instantiate(sceneObjectPrefab, pos, rot);

        // set the RigidBody as non-kinematic on the server only (isKinematic = true in prefab)
        newSceneObject.GetComponent<Rigidbody>().isKinematic = false;

        SceneObject sceneObject = newSceneObject.GetComponent<SceneObject>();

        // set the child object on the server
        sceneObject.SetEquippedItem(equippedItem);

        // set the SyncVar on the scene object for clients
        sceneObject.equippedItem = equippedItem;

        // set the player's SyncVar to nothing so clients will destroy the equipped child item
        equippedItem = EquippedItem.nothing;

        // Spawn the scene object on the network for all to see
        NetworkServer.Spawn(newSceneObject);
    }
```

На изображении выше есть поле `sceneObjectPrefab` которое присваивается Prefab'у, который будет выступать в качестве контейнера для наших Prefab'ов. В Prefab'e SceneObject есть скрипт SceneObject с SyncVar, подобный скрипту Player Equipped, и метод SetEquippedItem, который принимает общее значение enum в качестве параметра.

```csharp
using UnityEngine;
using System.Collections;
using Mirror;

public class SceneObject : NetworkBehaviour
{
    [SyncVar(hook = nameof(OnChangeEquipment))]
    public EquippedItem equippedItem;

    public GameObject ballPrefab;
    public GameObject boxPrefab;
    public GameObject cylinderPrefab;

    void OnChangeEquipment(EquippedItem oldEquippedItem, EquippedItem newEquippedItem)
    {
        StartCoroutine(ChangeEquipment(newEquippedItem));
    }

    // Since Destroy is delayed to the end of the current frame, we use a coroutine
    // to clear out any child objects before instantiating the new one
    IEnumerator ChangeEquipment(EquippedItem newEquippedItem)
    {
        while (transform.childCount > 0)
        {
            Destroy(transform.GetChild(0).gameObject);
            yield return null;
        }

        // Use the new value, not the SyncVar property value
        SetEquippedItem(newEquippedItem);
    }

    // SetEquippedItem is called on the client from OnChangeEquipment (above),
    // and on the server from CmdDropItem in the PlayerEquip script.
    public void SetEquippedItem(EquippedItem newEquippedItem)
    {
        switch (newEquippedItem)
        {
            case EquippedItem.ball:
                Instantiate(ballPrefab, transform);
                break;
            case EquippedItem.box:
                Instantiate(boxPrefab, transform);
                break;
            case EquippedItem.cylinder:
                Instantiate(cylinderPrefab, transform);
                break;
        }
    }
}
```

На приведенном ниже изображении во время выполнения Ball(Clone) прикреплен к объекту `RightHand`, и Box(Clone) прикреплена к SceneObject(Clolne), который показан в инспекторе.

{% hint style="info" %}
На визуальных Prefab'ax есть простые коллайдеры (сфера, коробка, капсула). Если в вашем визуальном Prefab'e есть mesh коллайдер, он должен быть помечен как Convex, чтобы работать с Rigidbody в контейнере SceneObject.
{% endhint %}

![](<../../../.gitbook/assets/image (124).png>)

## Поднятие предметов <a href="#pickup-items" id="pickup-items"></a>

Теперь, когда у нас в сцене упала коробка, нам нужно поднять ее снова. Чтобы сделать это, необходимо добавить метод `CmdPickupItem` в скрипт Equip:

```csharp
    // CmdPickupItem is public because it's called from a script on the SceneObject
    [Command]
    public void CmdPickupItem(GameObject sceneObject)
    {
        // set the player's SyncVar so clients can show the equipped item
        equippedItem = sceneObject.GetComponent<SceneObject>().equippedItem;

        // Destroy the scene object
        NetworkServer.Destroy(sceneObject);
    }
```

Данный метод будет просто вызываться по событию `OnMouseDown` в скрипте объекта:

```csharp
    void OnMouseDown()
    {
        NetworkClient.localPlayer.GetComponent<PlayerEquip>().CmdPickupItem(gameObject);
    }
```

Теперь, когда SceneObject(Clone) сетевой, мы можем передать его непосредственно в `CmdPickupItem` на объекте игрока установить SyncVar снаряженного предмета и уничтожить объект сцены.

Для всего этого примера единственный Prefab, который необходимо зарегистрировать в Network Manager кроме того, игрок является Prefab'ом SceneObject.
