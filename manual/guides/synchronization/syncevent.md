# SyncEvent (Устарело)

{% hint style="warning" %}
SyncEvents были удалены в версии 18.0.0, смотрите [Issue](https://github.com/vis2k/Mirror/pull/2178) для получения более подробной информации.
{% endhint %}

Это атрибут, который может быть присвоен событиям в классах NetworkBehaviour, чтобы разрешить их вызов на клиенте при вызове события на сервере.

События SyncEvent вызываются пользовательским кодом на сервере, а затем вызываются для соответствующих клиентских объектов на клиентах, подключенных к серверу. Аргументы для вызова события сериализуются по сети, так что клиентское событие вызывается с теми же значениями, что и метод на сервере. Эти события должны начинаться с префикса "Событие"..

```csharp
using UnityEngine;
using Mirror;

public class DamageClass : NetworkBehaviour
{
    public delegate void TakeDamageDelegate(int amount, float dir);

    [SyncEvent]
    public event TakeDamageDelegate EventTakeDamage;

    [Command]
    public void CmdDoMe(int val)
    {
        EventTakeDamage(val, 1.0f);
    }
}

public class Other : NetworkBehaviour
{
    public DamageClass damager;
    int health = 100;

    void Start()
    {
        if (NetworkClient.active)
            damager.EventTakeDamage += TakeDamage;
    }

    public void TakeDamage(int amount, float dir)
    {
        health -= amount;
    }
}
```

SyncEvents позволяют распространять сетевые действия на другие скрипты, прикрепленные к объекту. В приведенном выше примере другой класс регистрируется для события TakeDamage в DamageClass. Когда событие происходит в классе DamageClass на сервере, метод TakeDamage() будет вызван в другом классе клиентского объекта. Это позволяет создавать модульные сетевые системы, которые могут быть расширены за счет новых сценариев, реагирующих на генерируемые ими события.
