---
description: Есть ID?
---

# ID

## Asset Id <a href="#asset-id" id="asset-id"></a>

Mirror использует GUID для Asset ids. Каждый Prefab с компонентом NetworkIdentity имеет Asset Id, что просто конвертирует Unity AssetDatabase.AssetPathToGUID в 16 bytes. Mirror это нужно для того, чтобы знать, какие Prefab'ы создавать.

## Scene Id <a href="#scene-id" id="scene-id"></a>

Mirror использует uint для Scene Ids. Каждому игровому объекту с NetworkIdentity в сцене (иерархии) назначается id в OnPostProcessScene. Mirror это необходимо для того, чтобы отличать объекты сцены друг от друга, поскольку Unity не имеет уникального идентификатора для разных игровых объектов в сцене.

## Network Instance Id (aka NetId) <a href="#network-instance-id-aka-netid" id="network-instance-id-aka-netid"></a>

Mirror использует uint для NetId. каждому NetworkIdentity назначается NetId в NetworkIdentity.OnStartServer, или после его спавна. Mirror использует id при передаче сообщений между клиентом и сервером, чтобы указать, какой объект является получателем сообщения.

## Connection Id <a href="#connection-id" id="connection-id"></a>

Каждое сетевое подключение имеет идентификатор подключения, который присваивается транспортным уровнем низкого уровня. Идентификатор соединения 0 зарезервирован для локального подключения, когда сервер также является клиентом (хостом)
