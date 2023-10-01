# SyncList

SyncLists является массивом, основанном на листах, похожих на C# [List](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1?view=netframework-4.7.2) который синхронизирует свое содержимое с сервера на клиенты.

SyncList может содержать в себе любые [поддерживаемые типы данных Mirror](../data-types.md).

## Различия с UNET <a href="#differences-with-hlapi" id="differences-with-hlapi"></a>

UNET также поддерживает SyncLists, но мы переработали их архитектуру, чтобы сделать более эффективными и простыми в использовании. Некоторые из ключевых отличий включают:

* В UNET, SyncLists были синхронизированы сразу же, как только они изменились. Если вы добавляете 10 элементов, это означает отправку 10 отдельных сообщений. Mirror синхронизирует SyncLists вместе с SyncVars. 10 элементов и другие SyncVar объединяются в одно сообщение. Mirror также учитывает интервал синхронизации при синхронизации списков.
* В UNET если вам нужен список структур, вы должны использовать `SyncListStruct`, мы изменили его на просто `SyncList`
* В UNET обратный вызов - это делегат. В Mirror мы изменили его на событие, чтобы вы могли добавлять много подписчиков.
* В UNET обратный вызов сообщает вам об операции и индексе. В Mirror обратный вызов также получает элемент. Мы внесли это изменение, чтобы можно было определить, какой элемент был удален.
* В UNET вы должны создать класс, который наследуется от SyncList. В Mirror вы можете просто использовать SyncList (начиная с версии 20.0.0)

## Использование <a href="#usage" id="usage"></a>

Добавьте поле SyncList в ваш класс NetworkBehaviour.

{% hint style="info" %}
SyncList должен быть объявлен как **readonly** и инициализирован в конструкторе.
{% endhint %}

```csharp
public struct Item
{
    public string name;
    public int amount;
    public Color32 color;
}

public class Player : NetworkBehaviour
{
    public readonly SyncList<Item> inventory = new SyncList<Item>();

    public int coins = 100;

    [Command]
    public void CmdPurchase(string itemName)
    {
        if (coins > 10)
        {
            coins -= 10;
            Item item = new Item
            {
                name = "Sword",
                amount = 3,
                color = new Color32(125, 125, 125, 255)
            };

            // во время следующей синхронизации все клиенты увидят этот элемент
            inventory.Add(item);
        }
    }
}
```

Вы также можете определить, когда изменяется SyncList на клиенте или сервере. Это полезно для обновления вашего персонажа при добавлении снаряжения или определения того, когда вам нужно обновить свою базу данных. Подписаться на событие обратного вызова обычно можно в `Start`, `OnClientStart`, или `OnServerStart`.

{% hint style="warning" %}
Обратите внимание, что к моменту вашей подписки список уже будет заполнен, поэтому вы не получите запрос на исходные данные, а только обновления.
{% endhint %}

```csharp
class Player : NetworkBehaviour {

    public override void OnStartClient()
    {
        inventory.Callback += OnInventoryUpdated;
        
        // Обработать начальную полезную нагрузку SyncList
        for (int index = 0; index < inventory.Count; index++)
            OnInventoryUpdated(SyncList<Item>.Operation.OP_ADD, index, new Item(), inventory[index]);
    }

    void OnInventoryUpdated(SyncList<Item>.Operation op, int index, Item oldItem, Item newItem)
    {
        switch (op)
        {
            case SyncList<Item>.Operation.OP_ADD:
                // индекс - это место, где он был добавлен в список
                // newItem - это новый элемент
                break;
            case SyncList<Item>.Operation.OP_INSERT:
                // индекс - это место, где он был вставлен в список
                // newItem - это новый элемент
                break;
            case SyncList<Item>.Operation.OP_REMOVEAT:
                // индекс - это место, где он был удален из списка
                // oldItem - это элемент, который был удален
                break;
            case SyncList<Item>.Operation.OP_SET:
                // индекс относится к элементу, который был изменен
                // oldItem - это предыдущее значение элемента в индексе
                // newItem - это новое значение для элемента в индексе
                break;
            case SyncList<Item>.Operation.OP_CLEAR:
                // список был очищен
                break;
        }
    }
}
```
