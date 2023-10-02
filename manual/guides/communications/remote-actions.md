# Удаленные действия

Сетевая система имеет способы выполнения действий по всей сети. Действия такого типа иногда называются удаленными вызовами процедур. В сетевой системе существует два типа RPC: Command, которые вызываются с клиента и выполняются на сервере; и вызовы ClientRpc, которые вызываются на сервере и выполняются на клиентах.

На диаграмме ниже показаны направления, в которых выполняются удаленные действия:

![](<../../../.gitbook/assets/image (6) (1).png>)

## Command <a href="#commands" id="commands"></a>

Command отправляются от объектов player на клиенте к объектам player на сервере. В целях безопасности Command по умолчанию могут отправляться только с вашего объекта player, поэтому вы не можете управлять объектами других игроков. Вы можете обойти проверку контроля над объектом, используя `[Command(requiresAuthority = false)]`.

Чтобы превратить функцию в Command, добавьте к ней пользовательский атрибут \[Command] и необязательно добавьте префикс “Cmd” для соглашения об именовании. Теперь эта функция будет запускаться на сервере при ее вызове на клиенте. Любые параметры [поддерживаемого типа данных](../data-types.md) будут автоматически переданы на сервер с помощью Command.

Функции Command должны иметь префикс “Cmd” и не могут быть статическими. Это подсказка при чтении кода, вызывающего Command - эта функция является специальной и не вызывается локально, как обычная функция.

```csharp
public class Player : NetworkBehaviour
{
    // будет назначен в инспекторе
    public GameObject cubePrefab;

    void Update()
    {
        if (!isLocalPlayer) return;

        if (Input.GetKey(KeyCode.X))
            CmdDropCube();
    }

    [Command]
    void CmdDropCube()
    {
        if (cubePrefab != null)
        {
            Vector3 spawnPos = transform.position + transform.forward * 2;
            Quaternion spawnRot = transform.rotation;
            GameObject cube = Instantiate(cubePrefab, spawnPos, spawnRot);
            NetworkServer.Spawn(cube);
        }
    }
}
```

Будьте осторожны при отправке Command от клиента в каждом кадре! Это может привести к большому объему сетевого трафика.

### Обход проверки контроля

Можно вызывать команды для объектов, не являющихся игроками, если верно любое из следующих условий:

* Объект был создан с правами клиента
* Объект имеет контроль клиента, установленный с помощью `NetworkIdentity.AssignClientAuthority`
* Command имеет опцию `requiresAuthority` стоящую на false.
  * Вы можете включить необязательный параметр `NetworkConnectionToClient sender = null` в методе Command, где Mirror передаст данный параметр за вас.
  * Не пытайтесь передать этот необязательный параметр сами... Эта попытка будет проигнорирована.

Command будут выполняться для экземпляра объекта именно на сервере, а не для экземпляра объекта на клиенте.

```csharp
public enum DoorState : byte
{
    Open, Closed, Locked
}

public class Door : NetworkBehaviour
{
    [SyncVar]
    public DoorState doorState;
    
    [Client]
    void OnMouseUpAsButton()
    {
        CmdSetDoorState();
    }

    [Command(requiresAuthority = false)]
    public void CmdSetDoorState(NetworkConnectionToClient sender = null)
    {
        bool hasDoorKey = sender.identity.GetComponent<PlayerState>().hasDoorKey;
        
        if (doorState == DoorState.Open)
        {
            doorState = hasDoorKey ? DoorState.Locked : DoorState.Closed;
            return;
        }
        
        if (doorState == DoorState.Locked && hasDoorKey)
        {
            doorState = DoorState.Open;
            return;
        }
        
        if (doorState == DoorState.Closed)
            doorState = DoorState.Open;
    }
}
```

## ClientRpc <a href="#clientrpc-calls" id="clientrpc-calls"></a>

ClientRpc отправляются от объектов на сервере к объектам на клиентах. Они могут быть отправлены с любого серверного объекта с созданным NetworkIdentity. Поскольку сервер обладает контролем над всеми, то нет никаких проблем с безопасностью, связанных с тем, что серверные объекты могут отправлять эти вызовы. Чтобы преобразовать функцию в вызов ClientRpc, добавьте к ней пользовательский атрибут \[ClientRpc] и необязательно добавьте префикс “Rpc” для соглашения об именовании. Теперь эта функция будет запускаться на клиентах при ее вызове на сервере. Любые параметры [поддерживаемого типа данных](../data-types.md) будут автоматически переданы клиентам с вызовом ClientRpc..

Функция ClientRpc должна иметь префикс “Rpc” и не может быть статичной. Это подсказка при чтении кода, вызывающего метод - эта функция является специальной и не вызывается локально, как обычная функция.

```csharp
public class Player : NetworkBehaviour
{
    int health;

    public void TakeDamage(int amount)
    {
        if (!isServer) return;

        health -= amount;
        RpcDamage(amount);
    }

    [ClientRpc]
    public void RpcDamage(int amount)
    {
        Debug.Log("Took damage:" + amount);
    }
}
```

При запуске игры от имени хоста с локальным клиентом вызовы ClientRpc будут вызываться на локальном клиенте, даже если он находится в том же процессе, что и сервер. Таким образом, поведение локальных и удаленных клиентов одинаково для вызовов ClientRpc.

### Excluding Owner

Сообщения ClientRpc отправляются только наблюдателям объекта в соответствии с его [Network Visibility](../../interest-management/). Объекты игрока всегда являются наблюдателями самих себя. В некоторых случаях вы можете захотеть исключить клиента-владельца при вызове ClientRpc. Это делается с помощью `includeOwner` и опции: `[ClientRpc(includeOwner = false)]`.

## TargetRpc <a href="#targetrpc-calls" id="targetrpc-calls"></a>

Функции TargetRpc вызываются вашим кодом на сервере, а затем вызываются для соответствующего клиентского объекта на клиенте указанного как `NetworkConnection`. Аргументы для вызова RPC сериализуются по сети, так что клиентская функция вызывается с теми же значениями, что и функция на сервере. Эти функции должны начинаться с префикса "Target" в соответствии с соглашением об именовании и не могут быть статическими.

**Важен контекст:**

* Если первым параметров вашего метода TargetRpc является `NetworkConnection`, тогда именно этот клиент выполнит данный метод
* Если первый параметр любого другого типа, то данный метод выполнит клиент, в скрипте которого и создан данный метод.

В этом примере показано, как клиент может использовать Command для отправки запроса серверу (`CmdMagic`) путем включения данных другого игрока `connectionToClient` в качестве одного из параметров TargetRpc, вызываемого непосредственно из этого Command метода:

```csharp
public class Player : NetworkBehaviour
{
    public int health;

    [Command]
    void CmdMagic(GameObject target, int damage)
    {
        target.GetComponent<Player>().health -= damage;

        NetworkIdentity opponentIdentity = target.GetComponent<NetworkIdentity>();
        TargetDoMagic(opponentIdentity.connectionToClient, damage);
    }

    [TargetRpc]
    public void TargetDoMagic(NetworkConnectionToClient target, int damage)
    {
        // Это появится на клиенте соперника, а не на клиенте атакующего игрока
        Debug.Log($"Magic Damage = {damage}");
    }

    // Исцелить самого себя
    [Command]
    public void CmdHealMe()
    {
        health += 10;
        TargetHealed(10);
    }

    [TargetRpc]
    public void TargetHealed(int amount)
    {
        // Нет параметра NetworkConnection, поэтому метод передается владельцу скрипта
        Debug.Log($"Health increased by {amount}");
    }
}
```

## Аргументы удаленных действий <a href="#arguments-to-remote-actions" id="arguments-to-remote-actions"></a>

Аргументы, передаваемые в Command и ClientRpc, сериализуются и отправляются по сети. Вы можете использовать любой [поддерживаемый тип Mirror](../data-types.md).

Аргументы для удаленных действий не могут быть подкомпонентами игровых объектов, таких как экземпляры скриптов или Transform.
