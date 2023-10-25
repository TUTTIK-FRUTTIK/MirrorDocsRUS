---
description: Текст написан JesusLuvsYooh / StephenAllenGames.co.uk
---

# Unity Canvas HUD

## Конечный результат:

![A Unity Canvas that covers the majority of the OnGUI NetworkManagerHUDD component features.](../.gitbook/assets/Canvas0.jpg)

## Часть 1

Пустой проект, импорт Mirror из Asset Store/ Discord Releases unity package.

Откройте свою сцену, для этого руководства мы будем использовать Mirror/Examples/Tanks

Вы уже должны быть знакомы с примерами и NetworkManager HUD, они выглядят примерно так:

![OnGUI NetworkManagerHUD.](../.gitbook/assets/Canvas1.jpg)

![Mirror/Examples/Tank/Scenes/Scene](../.gitbook/assets/Canvas2.jpg)

## Часть 2

Создайте канвас в сцене, щелкнув правой кнопкой мыши "UI canvas" или в меню вверху "GameObject, UI, Canvas".\
Поставьте canvas scaler на “Scale with Screen Size”, это поможет сохранить все одинакового размера как на экранах с низким, так и с высоким разрешением, и лучше всего установить его перед добавлением содержимого в канвас.

Затем создайте и прикрепите новый скрипт к канвасу, я назвал его CanvasHUD.

![Scaling.](../.gitbook/assets/Canvas4.jpg)

![Script.](../.gitbook/assets/Canvas3.jpg)

## Часть 3

Откройте этот новый скрипт и откройте Mirror NetworkManagerHUD (для ссылок).

Добавьте следующий код в качестве начального шаблона для CanvasHUD.

{% code title="CanvasHUD.cs" %}
```csharp
using Mirror;
using UnityEngine;
using UnityEngine.UI;

public class CanvasHUD : MonoBehaviour
{
	public Button buttonHost;

	public void Start()
	{
		buttonHost.onClick.AddListener(ButtonHost);
  }

	public void ButtonHost()
	{
		NetworkManager.singleton.StartHost();
	}
}
```
{% endcode %}

Создайте кнопку внутри основного канваса и перетащите ее в переменную Canvas “ButtonHost”. В этом руководстве мы не будем слишком зацикливаться на макете и внешнем виде канваса, но проявите фантазию и разместите содержимое там, где вам заблагорассудится :)

![Simple button.](../.gitbook/assets/Canvas5.jpg)

## Часть 4

Тест! Запустите игру и нажмите свою собственную “Host Button”, игра должна начаться.

Поздравляем, это первый шаг к использованию Unity Canvas с Mirror и обновлению с NetworkManagerHUD OnGUI.

![New canvas button and old OnGUI](../.gitbook/assets/Canvas6.jpg)

## Часть 5

Если вы проверите старый HUD, его можно обобщить на 2 части. Кнопки "Пуск" (перед подключением) и ‘Стоп’ (после подключения).

Создайте 2 панели пользовательского интерфейса внутри канваса, переименуйте их в Panel Start и Panel Stop, удалите компонент изображения из Panel Stop, таким образом, мы сможем отличить их друг от друга.

Перетащите вашу “Button Host” в Panel Start.

![Before connecting.](../.gitbook/assets/Canvas7.jpg)

![After connecting.](../.gitbook/assets/Canvas8.jpg)

![Summarised sections.](../.gitbook/assets/Canvas9.jpg)

## Часть 6

Добавьте следующие переменные в свой скрипт CanvasHUD, эти переменные охватывают большую часть того, что необходимо.

{% code title="CanvasHUD.cs" %}
```csharp
public GameObject PanelStart;
public GameObject PanelStop;

public Button buttonHost, buttonServer, buttonClient, buttonStop;

public InputField inputFieldAddress;

public Text serverText;
public Text clientText;
```
{% endcode %}

Далее, добавьте больше пользовательского интерфейса! Захватывающее право!:D

Пока не беспокойтесь о коде, посмотрите на изображение ниже, чтобы увидеть, что нужно.

Внутри “Start Panel” должны быть 3 кнопки, поле ввода и необязательный текст заголовка. Панель "Stop" должна содержать одну кнопку и 2 текста, которые вы можете удалить, добавить и настроить позже, но пока следуйте этому руководству, чтобы все совпадало.

Перетащите весь новый пользовательский интерфейс в переменные скрипта CanvasHUD, если вы правильно пометили их все при работе, это будет более простой задачей.

![Scene view, Hierarchy layout and script variables.](../.gitbook/assets/Canvas10.jpg)

## Часть 7

Теперь, чтобы код заставил все это работать, различные части будут сопровождаться комментариями для объяснения. И это все, теперь вы создали свой собственный пользовательский интерфейс Unity Canvas HUD или обновили OnGUI NetworkManagerHUD! :D

{% code title="CanvasHUD.cs" %}
```csharp

    private void Start()
    {
        //Обновляет текст на канвасе, если вы вручную изменили ip адрес в Network Manager перед запуском игровой сцены
        if (NetworkManager.singleton.networkAddress != "localhost") { inputFieldAddress.text = NetworkManager.singleton.networkAddress; }

        //Добавляет listener в основное поле ввода и вызывает метод при изменении значения.
        inputFieldAddress.onValueChanged.AddListener(delegate { ValueChangeCheck(); });

        //Обязательно прикрепите эти кнопки в инспекторе
        buttonHost.onClick.AddListener(ButtonHost);
        buttonServer.onClick.AddListener(ButtonServer);
        buttonClient.onClick.AddListener(ButtonClient);
        buttonStop.onClick.AddListener(ButtonStop);

        //Это обновляет Unity canvas, мы должны вручную вызывать его при каждом изменении, в отличие от устаревшего OnGUI.
        SetupCanvas();
    }

    // Вызывается при изменении значения текстового поля.
    public void ValueChangeCheck()
    {
        NetworkManager.singleton.networkAddress = inputFieldAddress.text;
    }

    public void ButtonHost()
    {
        NetworkManager.singleton.StartHost();
        SetupCanvas();
    }

    public void ButtonServer()
    {
        NetworkManager.singleton.StartServer();
        SetupCanvas();
    }

    public void ButtonClient()
    {
        NetworkManager.singleton.StartClient();
        SetupCanvas();
    }

    public void ButtonStop()
    {
        // остановить хост, если вы являетесь хостом
        if (NetworkServer.active && NetworkClient.isConnected)
        {
            NetworkManager.singleton.StopHost();
        }
        // остановить клиент, если вы являетесь клиентом
        else if (NetworkClient.isConnected)
        {
            NetworkManager.singleton.StopClient();
        }
        // остановить сервер, если вы являетесь сервером
        else if (NetworkServer.active)
        {
            NetworkManager.singleton.StopServer();
        }

        SetupCanvas();
    }

    public void SetupCanvas()
    {
        // Здесь мы удалим большую часть пользовательского интерфейса canvas, который может быть изменен.

        if (!NetworkClient.isConnected && !NetworkServer.active)
        {
            if (NetworkClient.active)
            {
                PanelStart.SetActive(false);
                PanelStop.SetActive(true);
                clientText.text = "Connecting to " + NetworkManager.singleton.networkAddress + "..";
            }
            else
            {
                PanelStart.SetActive(true);
                PanelStop.SetActive(false);
            }
        }
        else
        {
            PanelStart.SetActive(false);
            PanelStop.SetActive(true);

            // server / client status message
            if (NetworkServer.active)
            {
                serverText.text = "Server: active. Transport: " + Transport.active;
                // Обратите внимание, что более старые версии Mirror используют: Transport.activeTransport
            }
            if (NetworkClient.isConnected)
            {
                clientText.text = "Client: address=" + NetworkManager.singleton.networkAddress;
            }
        }
    }
```
{% endcode %}
