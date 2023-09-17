# Удалено

Некоторые функции Unity Networking (UNet) были удалены из Mirror или изменены по разным причинам. На этой странице будут указаны все измененные и удаленные функции, свойства и методы, причина изменения или удаления и возможные альтернативы.

{% hint style="info" %}
**Примечание:** Некоторые изменения в этом документе могут применяться к предстоящему выпуску Asset Store.
{% endhint %}

## Match Namespace и Host Migration <a href="#match-namespace--host-migration" id="match-namespace--host-migration"></a>

Как часть служб Unity, все это пространство имен было удалено. С самого начала это работало не очень хорошо, и невероятно плохо взаимодействовало с основной частью пакета. Мы ожидаем, что это, наряду с другими внутренними сервисами, будет предоставляться через автономные приложения, которые имеют интеграцию с Mirror.

## Network Server Simple <a href="#network-server-simple" id="network-server-simple"></a>

Это было очень непрактично для своей небольшой задачи, поэтому оно было удалено. Есть гораздо более простые способы создать базовый сервер прослушивания, с одним из наших транспортных средств или без него.

## Couch Co-Op <a href="#couch-co-op" id="couch-co-op"></a>

Основная сеть была значительно упрощена за счет удаления этого низко висящего фрукта. Оно было слишком багнутым и сложным, чтобы это исправлять. Если вам необходимо нечто подобное, рассмотрите невидимый player prefab как канал для прослушивания спавна актуальных player prefab'ов с client authority. Все входные данные будут проходить через prefab conduit для управления объектами player.

## Message Types <a href="#message-types" id="message-types"></a>

`MsgType` enum был удален. Все типы сообщений генерируются динамически. Используйте `Send` вместо этого.

## Network Transform <a href="#network-transform" id="network-transform"></a>

[Network Transform](../components/network-transform/) был полностью изменен так, что он теперь синхронизирует только позицию, вращение и размер, имеет параметры управления и определяет, следует ли их интерполировать, и использует Snapshot Interpolation. В нём есть куча виртуальных методов и [шаблоны скриптов](script-templates.md) для создания вашей собственной версии. Поддержка Rigidbody была удалена чтобы создать отдельный [Network Rigidbody](../components/network-rigidbody.md) компонент.

## Network Animator <a href="#network-animator" id="network-animator"></a>

[Network Animator](../components/network-animator.md) также был упрощен, поскольку он объединяет все параметры аниматора в одно сообщение об обновлении.

## SyncVar Hook Parameters <a href="#syncvar-hook-parameters" id="syncvar-hook-parameters"></a>

[SyncVar](../guides/synchronization/syncvars.md) значения свойств теперь обновляются перед вызовом перехватчика, и для перехватчиков теперь требуются два параметра того же типа, что и свойство: `oldValue` и `newValue`

## SyncListSTRUCT <a href="#syncliststruct" id="syncliststruct"></a>

Используйте [SyncList](../guides/synchronization/synclists.md) вместо этого.

## SyncList Classes

* `SyncListString` было заменено на `SyncList<string>` .
* `SyncListFloat` было заменено на `SyncList<float>`.
* `SyncListInt` было заменено на `SyncList<int>`.
* `SyncListUInt` было заменено на `SyncList<uint>`.
* `SyncListBool` было заменено на `SyncList<bool>`.

Смотрите [документацию](../guides/synchronization/synclists.md) для получения более подробного содержания.

## SyncList Operations <a href="#synclist-operations" id="synclist-operations"></a>

* `OP_REMOVE` было заменено на `OP_REMOVEAT`
* `OP_DIRTY` было заменено на `OP_SET`

See [documentation](../guides/synchronization/synclists.md) for more details.

## SyncDictionary Operations <a href="#syncidictionary-operations" id="syncidictionary-operations"></a>

* `OP_DIRTY` was replaced by `OP_SET`

Смотрите [документацию](../guides/synchronization/synclists.md) для получения более подробного содержания.

## SyncObject

* Теперь это класс, а не интерфейс.
* `Flush` - Используйте `ClearChanges` вместо этого.

## Quality of Service Flags <a href="#quality-of-service-flags" id="quality-of-service-flags"></a>

В классическом UNet флаги QoS использовались для определения того, как пакеты попадают на удаленный конец. Например, если вам нужно, чтобы пакету был присвоен приоритет в очереди, вы бы указали флаг высокого приоритета, который Unity LLAPI затем получил бы и обработал соответствующим образом. К сожалению, это вызвало много дополнительной работы для транспортного уровня, и некоторые флаги QoS не работали должным образом из-за ошибочного кода, который полагался на слишком много магии.

В Mirror, QoS флаги были заменены на систему "каналов". Стандартный транспорт [Telepathy](../transports/telepathy-transport.md) не использует каналы потому что полностью основан на TCP, остальные транспорты, такие как [Ignorance](../transports/ignorance.md) и [LiteNetLib](../transports/litenetlib-transport.md) поддерживают это.

В настоящее время определены следующие каналы:

* `Channels.Reliable = 0`
* `Channels.Unreliable = 1`

## Changes by Class <a href="#changes-by-class" id="changes-by-class"></a>

### NetworkManager <a href="#networkmanager" id="networkmanager"></a>

* `NetworkConnection` было заменено на `NetworkConnectionToClient` во всех местах.
* `networkPort`\
  Удален для разделения на транспортный компонент. Не все транспорты используют порты, только те, у кого действительно есть для этого поле. Смотрите [Транспорты](../transports/) для получения более подробного содержания.
* `IsHeadless()`\
  Use compiler symbol `UNITY_SERVER` instead.
* `ConfigureServerFrameRate` was renamed to `ConfigureHeadlessFrameRate`.
* `client`\
  Use NetworkClient directly, it will be made static soon. For example, use `NetworkClient.Send(message)` instead of `NetworkManager.client.Send(message)`.
* `IsClientConnected()`\
  Use static property `NetworkClient.isConnected` instead.
* `onlineScene` and `offlineScene`\
  These store full paths now, so use SceneManager.GetActiveScene().path instead.
* `OnStartClient(NetworkClient client)`\
  Override OnStartClient() instead since all `NetworkClient` methods are static now.
* `OnClientChangeScene(string newSceneName)`\
  Override `OnClientChangeScene(string newSceneName, SceneOperation sceneOperation, bool customHandling)` instead.
* `OnClientChangeScene(string newSceneName, SceneOperation sceneOperation)`\
  Override `OnClientChangeScene(string newSceneName, SceneOperation sceneOperation, bool customHandling)` instead.
* `OnServerAddPlayer(NetworkConnection conn, AddPlayerMessage extraMessage)`\
  Override `OnServerAddPlayer(NetworkConnection conn)` instead. See [Custom Player Spawn Guide](../guides/gameobjects/custom-character-spawning.md) for details.
* `OnServerRemovePlayer(NetworkConnection conn, NetworkIdentity player)`\
  Use `NetworkServer.RemovePlayerForConnection(NetworkConnection conn, GameObject player, bool keepAuthority = false)` instead.
*   `OnServerError(NetworkConnection conn, int errorCode)`

    Replaced with `OnServerError(NetworkConnection conn, Exception exception)`.
*   `OnClientError(NetworkConnection conn, int errorCode)`

    Replaced with `OnClientError(Exception exception)`.
* `disconnectInactiveConnections` and `disconnectInactiveTimeout` were removed.
* OnClient\* virtual methods no longer take a `NetworkConnection` parameter. Remove the parameter from your overrides and use `NetworkClient.connection` in your code instead.
* `serverTickRate` renamed to `sendRate`.
* `serverTickInterval` moved to `NetworkServer`.

### NetworkManagerHUD

*   `showGUI` was removed.

    Disable the component instead.

### NetworkRoomManager <a href="#networkroommanager" id="networkroommanager"></a>

* `NetworkConnection` was replaced by `NetworkConnectionToClient` in many places.
* `OnRoomServerCreateGamePlayer(NetworkConnection conn)`\
  Use `OnRoomServerCreateGamePlayer(NetworkConnection conn, GameObject roomPlayer)` instead.
* `OnRoomServerSceneLoadedForPlayer(GameObject roomPlayer, GameObject gamePlayer)`\
  Use `OnRoomServerSceneLoadedForPlayer(NetworkConnection conn, GameObject roomPlayer, GameObject gamePlayer)` instead.
*   Client virtual methods no longer take a `NetworkConnection` parameter.

    Use `NetworkClient.connection` within your overrides.

### NetworkIdentity <a href="#networkidentity" id="networkidentity"></a>

* `clientAuthorityOwner`\
  Use connectionToClient instead
* `GetSceneIdenity`\
  Use `GetSceneIdentity` instead (typo in original name)
* `RemoveClientAuthority(NetworkConnection conn)`\
  NetworkConnection parameter is no longer needed and nothing is returned
*   `spawned` dictionary

    This has been split up to `NetworkServer.spawned` and `NetworkClient.spawned` dictionaries.
* Local Player Authority checkbox\
  This checkbox is no longer needed, and we simplified how [Authority](../guides/authority.md) works in Mirror.

### NetworkBehaviour <a href="#networkbehaviour" id="networkbehaviour"></a>

* `NetworkConnection` was replaced by `NetworkConnectionToClient` in many places.
* `sendInterval` attribute\
  Use `NetworkBehaviour.syncInterval` field instead. Can be modified in the Inspector too.
* `List m_SyncObjects`\
  Use `List syncObjects` instead.
* `OnSetLocalVisibility(bool visible)`\
  Override `OnSetHostVisibility(bool visible)` instead.
* `OnRebuildObservers`, `OnCheckObserver`, and `OnSetHostVisibility` were moved to a separate class called `NetworkVisibility`
* `NetworkBehaviour.OnNetworkDestroy` was renamed to `NetworkBehaviour.OnStopClient`.
* `getSyncVarHookGuard` renamed to `GetSyncVarHookGuard`.
* `setSyncVarHookGuard` - renamed to `SetSyncVarHookGuard`.
* `SetDirtyBit` - Use `SetSyncVarDirtyBit` instead.
* `[Command]` attribute parameter `ignoreAuthority` replaced with `requiresAuthority`.
* `[ClientRpc]` attribute parameter `includeOwner` replace with `excludeOwner`.
* `hasAuthority` renamed to `isOwned`.

### NetworkConnection <a href="#networkconnection" id="networkconnection"></a>

* `hostId`\
  Removed because it's not needed ever since we removed LLAPI as default. It's always 0 for regular connections and -1 for local connections. Use `connection.GetType() == typeof(NetworkConnection)` to check if it's a regular or local connection.
* `isConnected`\
  Removed because it's pointless. A `NetworkConnection` is always connected.
* `InvokeHandlerNoData(int msgType)`\
  Use `InvokeHandler` instead.
* `playerController`\
  renamed to `identity` since that's what it is: the `NetworkIdentity` for the connection.
* `RegisterHandler(short msgType, NetworkMessageDelegate handler)`\
  Use `NetworkServer.RegisterHandler()` or `NetworkClient.RegisterHandler()` instead.
* `UnregisterHandler(short msgType)`\
  Use `NetworkServer.UnregisterHandler()` or `NetworkClient.UnregisterHandler()` instead.
* `Send(int msgType, MessageBase msg, int channelId = Channels.Reliable)`\
  Use `Send(msg, channelId)` instead.
* `clientOwnedObjects` renamed to `owned`.

### NetworkServer <a href="#networkserver" id="networkserver"></a>

* `NetworkConnection` was replaced by `NetworkConnectionToClient` in many places.
* `FindLocalObject(uint netId)`\
  Use `NetworkServer.spawned[netId].gameObject` instead.
* `RegisterHandler(int msgType, NetworkMessageDelegate handler)`\
  Use `RegisterHandler(T msg)` instead.
* `RegisterHandler(MsgType msgType, NetworkMessageDelegate handler)`\
  Use `RegisterHandler(T msg)` instead.
*   `RegisterHandler(Action handler, bool requireAuthentication = true)`

    Use `RegisterHandler(Action<NetworkConnection, T), requireAuthentication = true)` instead.
* `UnregisterHandler(int msgType)`\
  Use `UnregisterHandler(T msg)` instead.
* `UnregisterHandler(MsgType msgType)`\
  Use `UnregisterHandler(T msg)` instead.
* `SendToAll(int msgType, MessageBase msg, int channelId = Channels.Reliable)`\
  Use `SendToAll(T msg, int channelId = Channels.Reliable)` instead.
* `SendToClient(int connectionId, int msgType, MessageBase msg)`\
  Use `NetworkConnection.Send(T msg, int channelId = Channels.Reliable)` instead.
* `SendToClient(int connectionId, T msg)`\
  Use `NetworkConnection.Send(T msg, int channelId = Channels.Reliable)` instead.
* `SendToClientOfPlayer(NetworkIdentity identity, int msgType, MessageBase msg)`\
  Use `identity.connectionToClient.Send<T>(T message, int channelId = Channels.Reliable)` instead.
* `SendToReady(NetworkIdentity identity, short msgType, MessageBase msg, int channelId = Channels.Reliable)`\
  Use `identity.connectionToClient.Send()` instead.
*   `SendToReady(NetworkIdentity identity, T message, bool includeOwner = true, int channelId = Channels.Reliable)`

    Renamed to `SendToReadyObservers`.
* `SpawnWithClientAuthority(GameObject obj, GameObject player)`\
  Use `Spawn(GameObject obj, GameObject player)` instead.
* `SpawnWithClientAuthority(GameObject obj, NetworkConnection ownerConnection)`\
  Use `Spawn(GameObject obj, NetworkConnection ownerConnection)` instead.
* `SpawnWithClientAuthority(GameObject obj, Guid assetId, NetworkConnection ownerConnection)`\
  Use `Spawn(GameObject obj, Guid assetId, NetworkConnection ownerConnection)` instead.
* `disconnectInactiveConnections` and `disconnectInactiveTimeout` were removed.
* `NoConnections` was renamed to `NoExternalConnections`.
*   `DisconnectAllExternalConnections` / `DisconnectAllConnections`

    Use `DisconnectAll` instead.
* `OnError` renamed to `OnTransportError` for clarity.

### NetworkClient <a href="#networkclient" id="networkclient"></a>

* `NetworkClient singleton`\
  Use `NetworkClient` directly. Singleton isn't needed anymore as all functions are static now.\
  Example: `NetworkClient.Send(message)` instead of `NetworkClient.singleton.Send(message)`.
* `allClients`\
  Use `NetworkClient` directly instead. There is always exactly one client.
* `GetRTT()`\
  Use `NetworkTime.rtt` instead.
*   `readyConnection`

    Use `connection` instead.
*   `isLocalClient`

    Use `isHostClient` instead.
*   `DisconnectLocalServer()`

    Use `NetworkClient.Disconnect()` instead.
* `RegisterHandler(int msgType, NetworkMessageDelegate handler)`\
  Use `RegisterHandler(T msg)` instead.
* `RegisterHandler(MsgType msgType, NetworkMessageDelegate handler)`\
  Use `RegisterHandler(T msg)` instead.
*   `RegisterHandler(Action<NetworkConnection, T> handler, bool requireAuthentication = true)`

    Use `RegisterHandler(Action<T> handler, bool requireAuthentication = true)` instead.
* `UnregisterHandler(int msgType)`\
  Use `UnregisterHandler(T msg)` instead.
* `UnregisterHandler(MsgType msgType)`\
  Use `UnregisterHandler(T msg)` instead.
*   `Ready(NetworkConnection conn)`

    Use `Ready()` without the `NetworkConnection` parameter instead.
* `Send(short msgType, MessageBase msg)`\
  Use `Send(T msg, int channelId = Channels.Reliable)` with no message id instead
* `ShutdownAll()`\
  Use `Shutdown()` instead. There is only one client.
* `OnError` renamed to `OnTransportError` for clarity.

### ClientScene <a href="#clientscene" id="clientscene"></a>

* Merged into `NetworkClient`.

### Network Scene Checker

* Replaced by [Scene Interest Management](../interest-management/scene.md).

### Network Proximity Checker

* Replaced by [Spatial Hash](../interest-management/spatial-hashing.md) / [Distance](../interest-management/distance.md) Interest Management.

### Network Match Checker

* Replaced by Network Match and requires [Match Interest Managemen](../interest-management/match.md)t.

### Network Owner Checker

* Replaced by Network Team and requires [Team Interest Management](../interest-management/team.md).

### Network Authenticator

* `NetworkConnection` was replaced by `NetworkConnectionToClient` in many places.
* `OnClientAuthenticate` no longer takes a `NetworkConnection` parameter.\
  Use `NetworkClient.connection` as needed.
* `OnClientAuthenticated` event no longer takes a `NetworkConnection` parameter.\
  Use `NetworkClient.connection` as needed.
*   `NetworkConnection` is no longer used in client message handlers.

    Use `NetworkClient.connection` within your handlers instead.
* `ClientAccept` and `ClientReject` no longer needs a `NetworkConnection` parameter.

### NetworkTime

* `NetworkTime.timeVar` was renamed to `timeVariance`.
* `NetworkTime.timeSd` was renamed to `timeStandardDeviation`.
* `NetworkTime.rttVar` was renamed to `rttVariance`.
* `NetworkTime.rttSd` was renamed to `rttStandardDeviation`.

### Transport

* `activeTransport` renamed to `active`.

### Messages <a href="#messages" id="messages"></a>

Basic messages of simple types were all removed as unnecessary bloat. You can create your own message classes instead.

* `StringMessage`
* `ByteMessage`
* `BytesMessage`
* `IntegerMessage`
* `DoubleMessage`
* `EmptyMessage`

NetworkMessage requires structs in all cases - classes no longer supported

### NetworkReader <a href="#networkreader" id="networkreader"></a>

* `Read(byte[] buffer, int offset, int count)`\
  Use `ReadBytes` instead.
* `ReadPackedInt32(int value)` Use `ReadInt32(int value)` instead.
* `ReadPackedUInt32(uint value)` Use `ReadUInt32(uint value)` instead.
* `ReadPackedUInt64(ulong value)` Use `ReadUInt64(ulong value)` instead.
* `ReadBoolean` renamed to `ReadBool`.
* `ReadInt16` renamed to `ReadShort`.
* `ReadInt32` renamed to `ReadInt`.
* `Readint64` renamed to `ReadLong`.
* `ReadSingle` renamed to `ReadFloat`.

### NetworkWriter <a href="#networkwriter" id="networkwriter"></a>

* `Write(bool value)`\
  Use `WriteBool` instead.
* `Write(byte value)`\
  Use `WriteByte` instead.
* `Write(sbyte value)`\
  Use `WriteSByte` instead.
* `Write(short value)`\
  Use `WriteShort` instead.
* `Write(ushort value)`\
  Use `WriteUShort` instead.
* `Write(int value)`\
  Use `WriteInt` instead.
* `Write(uint value)`\
  Use `WriteUInt` instead.
* `Write(long value)`\
  Use `WriteLong` instead.
* `Write(ulong value)`\
  Use `WriteULong` instead.
* `Write(float value)`\
  Use `WriteFloat` instead.
* `Write(double value)`\
  Use `WriteDouble` instead.
* `Write(decimal value)`\
  Use `WriteDecimal` instead.
* `Write(string value)`\
  Use `WriteString` instead.
* `Write(char value)`\
  Use `WriteChar` instead.
* `Write(Vector2 value)`\
  Use `WriteVector2` instead.
* `Write(Vector2Int value)`\
  Use `WriteVector2Int` instead.
* `Write(Vector3 value)`\
  Use `WriteVector3` instead.
* `Write(Vector3Int value)`\
  Use `WriteVector3Int` instead.
* `Write(Vector4 value)`\
  Use `WriteVector4` instead.
* `Write(Color value)`\
  Use `WriteColor` instead.
* `Write(Color32 value)`\
  Use `WriteColor32` instead.
* `Write(Guid value)`\
  Use `WriteGuid` instead.
* `Write(Transform value)`\
  Use `WriteTransform` instead.
* `Write(Quaternion value)`\
  Use `WriteQuaternion` instead.
* `Write(Rect value)`\
  Use `WriteRect` instead.
* `Write(Plane value)`\
  Use `WritePlane` instead.
* `Write(Ray value)`\
  Use `WriteRay` instead.
* `Write(Matrix4x4 value)`\
  Use `WriteMatrix4x4` instead.
* `Write(NetworkIdentity value)`\
  Use `WriteNetworkIdentity` instead.
* `Write(GameObject value)`\
  Use `WriteGameObject` instead.
* `Write(byte[] buffer, int offset, int count)`\
  Use `WriteBytes` instead.
* `WritePackedInt32(int value)`\
  Use `WriteInt32(int value)` instead
* `WritePackedUInt32(uint value)`\
  Use `WriteUInt32(uint value)` instead
* `WritePackedUInt64(ulong value)`\
  Use `WriteUInt64(ulong value)` instead

### RemoteCallHelper

* Renamed to `RemoteProcedureCalls`.
* `CmdDelegate` renamed to `RemoteCallDelegate`.
* `MirrorInvokeType` renamed to `RemoteCallType`.

### Transport <a href="#transport" id="transport"></a>

* `GetConnectionInfo(int connectionId, out string address)`\
  Use `ServerGetClientAddress(int connectionId)` instead.
* `GetMaxBatchSize` renamed to `GetMaxPacketSize`.
*   `ClientSend(int channelId, ArraySegment segment)`

    Use `ClientSend(segment, channelId)` instead.
*   `ServerSend(int connectionId, int channelId, ArraySegment segment)`

    Use `ServerSend(connectionId, segment, channelId)` instead.

### Telepathy Transport <a href="#telepathytransport" id="telepathytransport"></a>

* `MaxMessageSize`\
  Use `MaxMessageSizeFromClient` or `MaxMessageSizeFromServer` instead.

### Fallback Transport

* This has been removed.

### Utils

* `Version` enum removed.
* `DefaultReliable` renamed to `Reliable`.
* `DefaultUnreliable` renamed to `Unreliable`.
