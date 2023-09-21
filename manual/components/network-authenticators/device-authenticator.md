# Device Authenticator

Device Authenticator использует `SystemInfo.deviceUniqueIdentifier` чтобы идентифицировать клиента.

Для платформ, которые не поддерживают `deviceUniqueIdentifier` сгенерируется идентификатор GUID и сохранится в `PlayerPrefs`.

{% hint style="warning" %}
ПРИМЕЧАНИЕ: deviceUniqueIdentifier может быть подделан, поэтому нет никакой гарантии безопасности.
{% endhint %}

* Переместите скрипт Device Authenticator в инспектор к объекту в вашей сцене имеющий компонент Network Manager
* Компонент Device Authenticator будет автоматически присвоен к полю Authenticator в Network Manager

Когда вы закончите, это должно выглядеть примерно так:

<div align="left">

<img src="../../../.gitbook/assets/image (100).png" alt="Network Manager with Device Authenticator assigned">

</div>

<div align="left">

<img src="../../../.gitbook/assets/image (101).png" alt="Device Authenticator">

</div>

{% hint style="info" %}
Вам не нужно ничего назначать в Event Listeners, если только вы не хотите подписаться на события в своем собственном коде для своих собственных целей. У Mirror есть внутренние прослушиватели для обоих событий.
{% endhint %}
