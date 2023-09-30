# Типы данных

Клиент и сервер могут передавать данные друг другу через [удаленные вызовы](communications/remote-actions.md), [синхронизацию состояний](synchronization/) или через [Network Messages](communications/network-messages.md)

Mirror поддерживает ряд типов данных, которые вы можете использовать в них, такие как:

* Базовые C# типы (byte, int, char, uint, UInt64, float, string, и т.д.)
* Встроенные математические типы Unity (Vector3, Quaternion, Rect, Plane, Vector3Int, и т.д.)
* Встроенные типы Unity, которые под капотом являются структурами (Color, Sprite, Texture2D, Ray, и т.д.)
* URI
* `NetworkIdentity`, `NetworkBehaviour`
* GameObject, но только с компонентом `NetworkIdentity` который был заспавнен в сети
  * **Не** prefab!
  * Смотрите важные подробности в секции [GameObjects](gameobjects/).
* Структуры с любым из вышеперечисленных типов данных
  * Рекомендуется реализовать IEquatable, и пометить структуру как readonly, поскольку изменение одного из свойств **не** приводит к повторной синхронизации
* Классы, у которых каждое поле имеет поддерживаемый тип данных
  * Они будут выделять мусор и будут создаваться заново в получателе при каждой отправке.
* ScriptableObject, у которого каждое поле имеет поддерживаемый тип данных
  * Они будут выделять мусор и будут создаваться заново в получателе при каждой отправке.
* Массивы любого из вышеперечисленных типов
  * Не поддерживается с [SyncVars или Sync\* collections](synchronization/)
* ArraySegments из любого из вышеперечисленных типов
  * Не поддерживается с [SyncVars или Sync\* collections](synchronization/)

## Game Objects <a href="#game-objects" id="game-objects"></a>

Game Objects в SyncVars, SyncLists, и SyncDictionaries в некоторых случаях хрупкие, и их следует использовать с осторожностью.

* Пока игровой объект уже существует как на сервере, так и на клиенте, ссылка должна быть в порядке.

Когда данные синхронизации поступают на клиент, указанный игровой объект может еще не существовать на этом клиенте, что приводит к нулевым значениям в данных синхронизации. Tэто происходит потому, что внутри Mirror передает `netId` из `NetworkIdentity` и пытается найти его в клиентском словаре `NetworClient.spawned`.

Если объект еще не был создан на клиенте, совпадение не будет найдено. Это может быть в той же полезной нагрузке, особенно для присоединения клиентов, но после синхронизации данных из другого объекта. Это также может быть значение null, поскольку игровой объект не является наблюдателем другого, [`Interest Management`](../interest-management/).

Вы можете обнаружить, что синхронизировать данные более надежно через `NetworkIdentity.netID` (uint). Вместо этого выполните свой собственный поиск в `NetworkClient.spawned` чтобы получить объект, возможно, в корутине:

```csharp
    public GameObject target;

    [SyncVar(hook = nameof(OnTargetChanged))]
    public uint targetID;

    void OnTargetChanged(uint _, uint newValue)
    {
        target = null;
        
        if (NetworkClient.spawned.TryGetValue(targetID, out NetworkIdentity identity))
            target = identity.gameObject;
        else
            StartCoroutine(SetTarget());
    }

    IEnumerator SetTarget()
    {
        while (target == null)
        {
            yield return null;
            if (NetworkClient.spawned.TryGetValue(targetID, out NetworkIdentity identity))
                target = identity.gameObject;
        }
    }
```

## Пользовательские типы данных <a href="#custom-data-types" id="custom-data-types"></a>

Иногда вы не хотите, чтобы Mirror генерировал сериализацию для ваших собственных типов. Например, вместо сериализации всех данных вы можете захотеть сериализовать только идентификатор данных, и получатель сможет искать сведения о данных по идентификатору в предопределенном списке или базе данных.

Иногда вам может потребоваться сериализовать данные, которые используют другой тип, не поддерживаемый Mirror, например DateTime.

Вы можете добавить поддержку для любого типа, добавив методы расширения в `NetworkWriter` и `NetworkReader`. Например, чтобы добавить поддержку для `DateTime`, добавьте это где-нибудь в свой проект:

```csharp
public static class DateTimeReaderWriter
{
      public static void WriteDateTime(this NetworkWriter writer, DateTime dateTime)
      {
          writer.WriteInt64(dateTime.Ticks);
      }
     
      public static DateTime ReadDateTime(this NetworkReader reader)
      {
          return new DateTime(reader.ReadInt64());
      }
}
```

...затем вы можете использовать `DateTime` в ваших `[Command]` или даже `SyncList`

## Наследование и полиморфизм <a href="#inheritance-and-polymorphism" id="inheritance-and-polymorphism"></a>

Иногда вам может потребоваться отправить своим командам полиморфный тип данных. Mirror не сериализует имя типа, чтобы уменьшить размер сообщений и по соображениям безопасности, поэтому Mirror не может определить тип полученного объекта, просмотрев сообщение.

> **Этот код не работает "из коробки".**

```csharp
class Item 
{
    public string name;
}

class Weapon : Item
{
    public int hitPoints;
}

class Armor : Item
{
    public int hitPoints;
    public int level;
}

class Player : NetworkBehaviour
{
    [Command]
    void CmdEquip(Item item)
    {
        // ВАЖНО: это не работает. Mirror передаст вам объект типа item
        // даже если вы передадите оружие или доспехи.
        if (item is Weapon weapon)
        {
            // Этот предмет является оружием, 
            // может быть, вам нужно взять его в руки
        }
        else if (item is Armor armor)
        {
            // возможно, вы захотите надеть броню на тело
        }
    }

    [Command]
    void CmdEquipArmor(Armor armor)
    {
        // ВАЖНО: это тоже не сработает, вы получите броню, но 
        // броня не будет иметь действительного Item.name , даже если вы передали броню с именем
    }
}
```

CmdEquip будет работать, если вы предоставите пользовательский сериализатор для типа `Item`. К примеру:

```csharp

public static class ItemSerializer 
{
    const byte WEAPON = 1;
    const byte ARMOR = 2;

    public static void WriteItem(this NetworkWriter writer, Item item)
    {
        if (item is Weapon weapon)
        {
            writer.WriteByte(WEAPON);
            writer.WriteString(weapon.name);
            writer.WritePackedInt32(weapon.hitPoints);
        }
        else if (item is Armor armor)
        {
            writer.WriteByte(ARMOR);
            writer.WriteString(armor.name);
            writer.WritePackedInt32(armor.hitPoints);
            writer.WritePackedInt32(armor.level);
        }
    }

    public static Item ReadItem(this NetworkReader reader)
    {
        byte type = reader.ReadByte();
        switch(type)
        {
            case WEAPON:
                return new Weapon
                {
                    name = reader.ReadString(),
                    hitPoints = reader.ReadPackedInt32()
                };
            case ARMOR:
                return new Armor
                {
                    name = reader.ReadString(),
                    hitPoints = reader.ReadPackedInt32(),
                    level = reader.ReadPackedInt32()
                };
            default:
                throw new Exception($"Invalid weapon type {type}");
        }
    }
}
```

## ScriptableObjects <a href="#scriptable-objects" id="scriptable-objects"></a>

Люди часто хотят отправлять ScriptableObjects с клиента или сервера. Например, у вас может быть куча мечей, созданных как объекты, доступные для сценариев, и вы хотите поместить экипированный меч в syncvar. Это будет работать нормально, Mirror сгенерирует средства чтения и записи для объектов scriptable, вызвав ScriptableObject.Создайте экземпляр и скопируйте все данные.

Однако сгенерированные программы чтения и записи подходят не для каждого случая. ScriptableObjects часто ссылаются на другие ресурсы, такие как текстуры, Prefab'ы или другие типы, которые невозможно сериализовать. Объекты, доступные для написания сценариев, часто сохраняются в папке "Ресурсы". Скриптовые объекты иногда содержат в себе большой объем данных. Сгенерированные средства чтения и записи могут не работать или быть недостаточными для этих ситуаций.

Вместо передачи данных ScriptableObject вы можете передать имя, и другая сторона сможет выполнить поиск того же объекта по имени. Таким образом, вы можете иметь любые данные в вашем скриптовом объекте. Вы можете сделать это, предоставив пользовательские средства чтения и записи. Вот пример:

```csharp
[CreateAssetMenu(fileName = "New Armor", menuName = "Armor Data")]
class Armor : ScriptableObject
{
    public int Hitpoints;
    public int Weight;
    public string Description;
    public Texture2D Icon;
    // ...
}

public static class ArmorSerializer 
{
    public static void WriteArmor(this NetworkWriter writer, Armor armor)
    {
       // нет необходимости сериализовывать данные, просто название брони
       writer.WriteString(armor.name);
    }

    public static Armor ReadArmor(this NetworkReader reader)
    {
        // загрузите ту же броню по названию. Данные будут получены из ресурса в папке Resources
        return Resources.Load(reader.ReadString());
    }
}
```
