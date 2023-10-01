# SyncDictionary

SyncDictionary представляет собой ассоциативный массив, содержащий неупорядоченный список пар ключ-значение. Ключи и значения могут быть любых типов, которые [поддерживаются в Mirror](../data-types.md). По умолчанию мы используем .Net [Dictionary](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2?view=netcore-3.1) что может накладывать дополнительные ограничения на ключи и значения.

Работа SyncDictionary очень похожа на [SyncLists](synclists.md): когда вы вносите изменение на сервере, это изменение распространяется на всех клиентов и вызывается обратный вызов. Передаются только дельты.

## Использование <a href="#usage" id="usage"></a>

Добавьте поле в ваш класс NetworkBehaviour типа `SyncDictionary`.

{% hint style="info" %}
`SyncDictionary должен` быть помечен как **readonly** и инициализирован в конструкторе.
{% endhint %}

{% hint style="warning" %}
Обратите внимание, что к тому времени, когда вы подпишетесь на обратный вызов, словарь уже будет инициализирован, поэтому вы не получите вызов для получения исходных данных, только обновления.
{% endhint %}

## Простой пример <a href="#simple-example" id="simple-example"></a>

```csharp
using UnityEngine;
using Mirror;

public struct Item
{
    public string name;
    public int hitPoints;
    public int durability;
}

public class ExamplePlayer : NetworkBehaviour
{
    public readonly SyncDictionary<string, Item> Equipment = new SyncDictionary<string, Item>();

    public override void OnStartServer()
    {
        Equipment.Add("head", new Item { name = "Helmet", hitPoints = 10, durability = 20 });
        Equipment.Add("body", new Item { name = "Epic Armor", hitPoints = 50, durability = 50 });
        Equipment.Add("feet", new Item { name = "Sneakers", hitPoints = 3, durability = 40 });
        Equipment.Add("hands", new Item { name = "Sword", hitPoints = 30, durability = 15 });
    }

    public override void OnStartClient()
    {
        // Оборудование уже заполнено всем, что настроил сервер
        // но мы можем подписаться на обратный вызов на случай, если он будет обновлен позже
        equipment.Callback += OnEquipmentChange;

        // Process initial SyncDictionary payload
        foreach (KeyValuePair<string, Item> kvp in equipment)
            OnEquipmentChange(SyncDictionary<string, Item>.Operation.OP_ADD, kvp.Key, kvp.Value);
    }

    void OnEquipmentChange(SyncDictionary<string, Item>.Operation op, string key, Item item)
    {
        switch (op)
        {
            case SyncIDictionary<string, Item>.Operation.OP_ADD:
                // добавлена запись
                break;
            case SyncIDictionary<string, Item>.Operation.OP_SET:
                // запись изменена
                break;
            case SyncIDictionary<string, Item>.Operation.OP_REMOVE:
                // запись удалена
                break;
            case SyncIDictionary<string, Item>.Operation.OP_CLEAR:
                // Dictionary был очищен
                break;
        }
    }
}
```

По умолчанию, SyncDictionary использует [Dictionary](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2?view=netcore-3.1) чтобы хранить данные. Если вы хотите использовать другую реализацию `IDictionary` такую как [SortedList](https://docs.microsoft.com/en-us/dotnet/api/system.collections.sortedlist?view=netcore-3.1) или [SortedDictionary](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.sorteddictionary-2?view=netcore-3.1), используйте `SyncIDictionary` и передайте экземпляр словаря, который вы хотите, чтобы он использовался. Например:

```csharp
public class ExamplePlayer : NetworkBehaviour
{
    public readonly SyncIDictionary Equipment = 
        new SyncIDictionary(new SortedList());
}
```
