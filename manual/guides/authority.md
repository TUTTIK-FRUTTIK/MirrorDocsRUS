# Контроль

## Network Authority <a href="#network-authority" id="network-authority"></a>

Authority - это способ принятия решения о том, кому принадлежит объект и кто имеет над ним контроль.

### Server Authority <a href="#server-authority" id="server-authority"></a>

Authority сервера означает, что сервер имеет контроль над объектом. Сервер имеет контроль над объектами по умолчанию. Это означает, что сервер будет управлять предметами коллекционирования, движущимися платформами, NPC и любыми другими сетевыми объектами, которые не принадлежат игроку.

### Client Authority <a href="#client-authority" id="client-authority"></a>

Authority клиента означает, что клиент имеет контроль над объектом.

Когда клиент имеет контроль над объектом, это означает, что он может вызывать [Commands](communications/remote-actions.md#commands) и что объект будет автоматически уничтожен при отключении клиента.

Даже если клиент имеет контроль над объектом, сервер по-прежнему управляет SyncVar и другими функциями сериализации. Клиенту необходимо будет использовать [Commands](communications/remote-actions.md#commands) чтобы обновить состояние на сервере, чтобы оно могло синхронизироваться с другими клиентами.

### Как выдать authority <a href="#how-to-give-authority" id="how-to-give-authority"></a>

По умолчанию сервер имеет контроль над всеми объектами. Сервер может предоставлять контроль над объектами, которыми клиент должен управлять, таким как объект player.

Если вы спавните объект игрока с помощью `NetworkServer.AddPlayerForConnection` тогда ему автоматически будет предоставлен контроль.

#### Использование NetworkServer.Spawn <a href="#using-networkserverspawn" id="using-networkserverspawn"></a>

Вы можете предоставить контроль клиенту при создании объекта. Это делается путем передачи соединения с сообщением о спавне

```csharp
GameObject go = Instantiate(prefab);
NetworkServer.Spawn(go, connectionToClient);
```

#### Использование identity.AssignClientAuthority <a href="#using-identityassignclientauthority" id="using-identityassignclientauthority"></a>

Вы можете предоставить контроль клиенту в любое время, используя `AssignClientAuthority`. Это можно сделать, вызвав `AssignClientAuthority` на объекте, которому вы хотите предоставить контроль.

```csharp
identity.AssignClientAuthority(conn);
```

Возможно, вы захотите сделать это, когда игрок подберет предмет

```csharp
// Command на объекте игрока
void CmdPickupItem(NetworkIdentity item)
{
    item.AssignClientAuthority(connectionToClient); 
}
```

### Как удалить контроль <a href="#how-to-remove-authority" id="how-to-remove-authority"></a>

Вы можете использовать `identity.RemoveClientAuthority` чтобы удалить контроль клиента над объектом.

```csharp
identity.RemoveClientAuthority();
```

Контроль не может быть удален с объекта player. Вместо этого вам придется заменить контролирующего клиента с помощью `NetworkServer.ReplacePlayerForConnection`.

### On Authority <a href="#on-authority" id="on-authority"></a>

Когда объекту предоставляется контроль или он удаляется из него, этому клиенту будет отправлено сообщение с уведомлением об этом. Это приведет к тому, что будут вызваны функции `OnStartAuthority` или `OnStopAuthority`.

### Check Authority <a href="#check-authority" id="check-authority"></a>

#### Client Side <a href="#client-side" id="client-side"></a>

Свойство `identity.isOwned` может использоваться для проверки того, имеет ли локальный игрок контроль над объектом.

#### Server Side <a href="#server-side" id="server-side"></a>

Свойство `identity.connectionToClient` можно проверить, чтобы увидеть, какой клиент имеет контроль над объектом. Если оно равно null, то у сервера есть полномочия.
