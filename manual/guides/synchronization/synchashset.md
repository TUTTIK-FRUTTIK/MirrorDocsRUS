# SyncHashSet

`SyncHashSet` очень похож на C# [HashSet](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.hashset-1) который синхронизирует свое содержимое с сервера на клиенты.

SyncHashSet может содержать [любой поддерживаемый тип данных в Mirror](../data-types.md)

## Использование <a href="#usage" id="usage"></a>

{% hint style="info" %}
SyncHashSet должен быть помечен как **readonly** и инициализирован в конструкторе.
{% endhint %}

Добавьте поле SyncHashSet в ваш класс NetworkBehaviour. Например:

```csharp
public class Player : NetworkBehaviour
{
    [SerializeField]
    public readonly SyncHashSet<string> skills = new SyncHashSet<string>();

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

Вы также можете определить, когда изменяется SyncHashSet. Это полезно для обновления вашего персонажа в клиенте или определения того, когда вам нужно обновить свою базу данных.

Подписаться на событие обратного вызова обычно можно в `Start`, `OnClientStart` или `OnServerStart`.

{% hint style="warning" %}
Обратите внимание, что к моменту вашей подписки набор уже будет заполнен, поэтому вы не получите запрос на исходные данные, только обновления.
{% endhint %}

```csharp
public class Player : NetworkBehaviour
{
    [SerializeField]
    public readonly SyncHashSet<string> buffs = new SyncHashSet<string>();

    // это добавит делегата на клиент.
    // Вместо этого используйте OnStartServer, если вы хотите, чтобы это было на сервере
    public override void OnStartClient()
    {
        buffs.Callback += OnBuffsChanged;

        // Обработать начальную полезную нагрузку SyncHashSet
        foreach (string buff in buffs)
            OnBuffsChanged(SyncSet<string>.Operation.OP_ADD, buff);
    }

    // SyncHashSet наследуется от SyncSet, поэтому используйте SyncSet здесь
    void OnBuffsChanged(SyncSet<string>.Operation op, string buff)
    {
        switch (op)
        {
            case SyncSet<string>.Operation.OP_ADD:
                // Добавлен бафф к персонажу
                break;
            case SyncSet<string>.Operation.OP_REMOVE:
                // Удален бафф с персонажа
                break;
            case SyncSet<string>.Operation.OP_CLEAR:
                // Снял все баффы с персонажа
                break;
        }
    }
}
```
