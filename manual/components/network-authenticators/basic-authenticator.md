# Basic Authenticator

Mirror включает в себя Basic Authenticator в папке Mirror / Authenticators который просто использует простое имя пользователя и пароль.

* Переместите скрипт Basic Authenticator в инспектор к объекту в вашей сцене имеющий компонент Network Manager
* Компонент Basic Authenticator будет автоматически присвоен к полю Authenticator в Network Manager

Когда вы закончите, это должно выглядеть примерно так:

<div align="left">

<img src="../../../.gitbook/assets/image (31).png" alt="Network Manager with Basic Authenticator assigned">

</div>

<div align="left">

<img src="../../../.gitbook/assets/image (105).png" alt="Basic Authenticator">

</div>

* Учетные данные сервера могут быть установлены во время разработки или во время выполнения программы
* Учетные данные клиента будут установлены во время выполнения, например, из полей ввода пользовательского интерфейса **до** вызова `NetworkManager.singleton.StartClient();`

{% hint style="info" %}
Вам не нужно ничего назначать в Event Listeners, если только вы не хотите подписаться на события в своем собственном коде для своих собственных целей. У Mirror есть внутренние прослушиватели для обоих событий.
{% endhint %}
