# SyncSortedSet

`SyncSortedSet` очень похож на C# [SortedSet](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.sortedset-1) который синхронизируют свое содержимое с сервера на клиенты.

В отличие от SyncHashSets, все элементы в SyncSortedSet сортируются при вставке. Пожалуйста, обратите внимание, что это имеет некоторые последствия для производительности.

SyncSortedSet может содержать в себе [любой тип данных, поддерживаемый Mirror](../data-types.md)

## Использование <a href="#usage" id="usage"></a>

{% hint style="info" %}
SyncSortedSet должен быть помечен как **readonly** и инициализирован в конструкторе.
{% endhint %}

Добавьте поле SyncSortedSet в ваш класс NetworkBehaviour. К примеру:

```csharp
class Player : NetworkBehaviour
{
    public readonly SyncSortedSet<string> skills = new SyncSortedSet<string>();
    int skillPoints = 10;

    [Command]
    public void CmdLearnSkill(string skillName)
    {
        if (skillPoints > 1)
        {
            skillPoints--;
            skills.Add(skillName);
        }
    }
}
```

Вы также можете определить, когда изменяется SyncSortedSet. Это полезно для обновления вашего персонажа в клиенте или определения того, когда вам нужно обновить свою базу данных. Подписаться на событие обратного вызова можно обычно в `Start`, `OnClientStart` или `OnServerStart`.

{% hint style="warning" %}
Обратите внимание, что к моменту вашей подписки набор уже будет инициализирован, поэтому вы не получите запрос на исходные данные, только обновления.
{% endhint %}

```csharp
class Player : NetworkBehaviour
{
    [SerializeField]
    public readonly SyncSortedSet<string> buffs = new SyncSortedSet<string>();

    // это добавит делегата на клиент.
    // Вместо этого используйте OnStartServer, если вы хотите, чтобы это было на сервере
    public override void OnStartClient()
    {
        buffs.Callback += OnBuffsChanged;

        // Обработать начальную полезную нагрузку SyncSortedSet
        foreach (string buff in buffs)
            OnBuffsChanged(SyncSortedSet<string>.Operation.OP_ADD, buff);
    }

    // SyncSortedSet наследуется от SyncSet, поэтому используйте SyncSet здесь
    void OnBuffsChanged(SyncSortedSet<string>.Operation op, string buff)
    {
        switch (op)
        {
            case SyncSortedSet<string>.Operation.OP_ADD:
                // Добавлен бафф к персонажу
                break;
            case SyncSortedSet<string>.Operation.OP_REMOVE:
                // Удален бафф с персонажа
                break;
            case SyncSortedSet<string>.Operation.OP_CLEAR:
                // снял все баффы с персонажа
                break;
        }
    }
}
```
