---
description: Гайд от JesusLuvsYooh / StephenAllenGames.co.uk, редактирован James Frowen
---

# Mirror Quick Start Project

В настоящее время это руководство расскажет вам о:

* [Базовая настройка сцены](quick-start-guide.md#part-1)
* [Движение игрока](quick-start-guide.md#part-4)
* [Именование и цвет](quick-start-guide.md#part-8)
* [Скрипт для кнопок на канвасе](quick-start-guide.md#part-11)
* [Смена оружия](quick-start-guide.md#part-12)
* [Настройка сетевых объектов сцены](quick-start-guide.md#part-15)
* [Переключение между меню и сценами](quick-start-guide.md#part-16)
* [Стрельба из оружия](quick-start-guide.md#part-20)

Лучше всего сначала создать мини-тренировочную игру, прежде чем переделать вашу одиночную или создавать свой новый идеальный мультиплеер.

Готовые примеры Mirror отлично подходят для использования в качестве справочников. Рекомендуется использовать их при настройке подключения с портами и брандмауэрами. Это может быть огромная тема, которая меняется от человека к человеку и не рассматривается в этом руководстве, здесь мы будем использовать localHost (несколько экземпляров игры на одном ПК).\
\
**Конечный результат:**

![](../.gitbook/assets/QS-image--036.jpg)

### Часть 1 <a href="#part-1" id="part-1"></a>

Создайте пустой проект, импортируйте Mirror из [Asset Store](https://assetstore.unity.com/packages/tools/network/mirror-129321).

### Часть 2 <a href="#part-2" id="part-2"></a>

* Создайте новую сцену, сохраните ее и добавьте в настройки билда
* Создайте новый игровой объект, назовите его NetworkManager в сцене и добавьте эти 3 компонента
  * NetworkManager
  * KCPTransport (TelepathyTransport уже устарел, вам не нужно использовать сразу KCP и Telepathy)
  * NetworkManagerHUD
* В компоненте NetworkManager поместите ваши оффлайн и онлайн сцены в нужные слоты, пока у нас есть только одна сцена, так что вставьте свою сцену в оба слота
  * Сцена должна быть в настройках билда, прежде чем перетаскивать ее в поле

![Note: KCP Transport has replaced Telepathy in newer versions](../.gitbook/assets/QS-image--000.jpg)

![](../.gitbook/assets/QS-image--001.jpg)

### Часть 3 <a href="#part-3" id="part-3"></a>

Настраиваем сцену

* Добавьте простой Plane с такими настройками:
  * position (0, -1, 0)
  * scale (2, 2, 2)
* (опционально) добавьте на него материал, я добавил грязь, используемую в других примерах Mirror
* Далее мы добавляем ещё один игровой объект, название не имеет значения
* Добавьте компонент `NetworkStartPosition` на этот GameObject
* Продублируйте игровой объект несколько раз и разбросайте его по полу вашей сцены так, чтобы у вас было несколько точек возрождения. Я сделал 4, по одному возле каждого угла

![](../.gitbook/assets/QS-image--002.jpg)

### Часть 4 <a href="#part-4" id="part-4"></a>

Создаём игрока

* Создайте капсулу, используя меню, как показано на рисунке
* Добавьте к нему компонент NetworkTransform, это автоматически добавит Network Identity
* Включите Client Authority в NetworkTransform

(Примечание: Более новые версии Mirror вместо Client Authority используют "Sync Direction", так что уберите Client Authority и поставьте Sync Direction на "Client To Server".)

![](../.gitbook/assets/QS-image--003.jpg)

* Переименуйте объект игрока
* Добавьте на него пустой скрипт
* Переместите этот объект в Project для создания его prefab'a
* А затем удалите объект игрока со сцены

![](../.gitbook/assets/QS-image--004.jpg)

* Переместите ваш объект игрока в Network manager,
* Поставьте Player Spawn Method на Round Robin.

![](../.gitbook/assets/image--005.jpg)

### Часть 5 <a href="#part-5" id="part-5"></a>

Добавьте следующий код в скрипт вашего игрока.

```csharp
using Mirror;
using UnityEngine;

namespace QuickStart
{
    public class PlayerScript : NetworkBehaviour
    {
        public override void OnStartLocalPlayer()
        {
            Camera.main.transform.SetParent(transform);
            Camera.main.transform.localPosition = new Vector3(0, 0, 0);
        }

        void Update()
        {
            if (!isLocalPlayer) { return; }

            float moveX = Input.GetAxis("Horizontal") * Time.deltaTime * 110.0f;
            float moveZ = Input.GetAxis("Vertical") * Time.deltaTime * 4f;

            transform.Rotate(0, moveX, 0);
            transform.Translate(0, 0, moveZ);
        }
    }
}
```

### Часть 6 <a href="#part-6" id="part-6"></a>

Нажмите play в редакторе Unity, а затем кнопку Host (сервер + клиент) в окне игры. Вы должны иметь возможность передвигаться в капсуле с видом от первого лица.

![](../.gitbook/assets/QS-image--006.jpg)

### Часть 7 <a href="#part-7" id="part-7"></a>

Создайте и запустите свою сцену, откройте ее, разместите на одной и нажмите кнопку клиента на другой. Поздравляю, вы создали мини-многопользовательскую игру!

![](../.gitbook/assets/QS-image--007.jpg)

### Часть 8 <a href="#part-8" id="part-8"></a>

Добавляем имя игрока над головой

* Внутри вашего Prefab'a игрока создайте пустой GameObject
* Переименуйте его в что то типо `FloatingInfo`
  * position Y to 1.5
  * scale X to -1
* Внутри `FloatingInfo` создайте 3D текст используя Unity menu (GameObject - 3D Object - 3D Text),
* Установите его, как показано на рисунке ниже

![](../.gitbook/assets/QS-image--008.jpg)

### Часть 9 <a href="#part-9" id="part-9"></a>

Обновите ваш скрипт игрока в соответствии с кодом ниже:

```csharp
using Mirror;
using UnityEngine;

namespace QuickStart
{
    public class PlayerScript : NetworkBehaviour
    {
        public TextMesh playerNameText;
        public GameObject floatingInfo;

        private Material playerMaterialClone;

        [SyncVar(hook = nameof(OnNameChanged))]
        public string playerName;

        [SyncVar(hook = nameof(OnColorChanged))]
        public Color playerColor = Color.white;

        void OnNameChanged(string _Old, string _New)
        {
            playerNameText.text = playerName;
        }

        void OnColorChanged(Color _Old, Color _New)
        {
            playerNameText.color = _New;
            playerMaterialClone = new Material(GetComponent<Renderer>().material);
            playerMaterialClone.color = _New;
            GetComponent<Renderer>().material = playerMaterialClone;
        }

        public override void OnStartLocalPlayer()
        {
            Camera.main.transform.SetParent(transform);
            Camera.main.transform.localPosition = new Vector3(0, 0, 0);
            
            floatingInfo.transform.localPosition = new Vector3(0, -0.3f, 0.6f);
            floatingInfo.transform.localScale = new Vector3(0.1f, 0.1f, 0.1f);

            string name = "Player" + Random.Range(100, 999);
            Color color = new Color(Random.Range(0f, 1f), Random.Range(0f, 1f), Random.Range(0f, 1f));
            CmdSetupPlayer(name, color);
        }

        [Command]
        public void CmdSetupPlayer(string _name, Color _col)
        {
            // player info sent to server, then server updates sync vars which handles it on all clients
            playerName = _name;
            playerColor = _col;
        }

        void Update()
        {
            if (!isLocalPlayer)
            {
                // make non-local players run this
                floatingInfo.transform.LookAt(Camera.main.transform);
                return;
            }

            float moveX = Input.GetAxis("Horizontal") * Time.deltaTime * 110.0f;
            float moveZ = Input.GetAxis("Vertical") * Time.deltaTime * 4f;

            transform.Rotate(0, moveX, 0);
            transform.Translate(0, 0, moveZ);
        }
    }
}
```

### Часть 10 <a href="#part-10" id="part-10"></a>

Добавьте объекты `PlayerNameText` и `FloatingInfo` под скриптом игрока как показано ниже

![](../.gitbook/assets/QS-image--009.jpg)

Теперь, если вы создадите и запустите, разместите на одном сервере, присоединитесь на другом, вы увидите имена игроков и цвета, синхронизированные по сети!

Молодец, 5 звезд тебе!

![](../.gitbook/assets/QS-image--010.jpg)

### Часть 11 <a href="#part-11" id="part-11"></a>

Сетевой объект сцены, к которому все могут получить доступ и настроить.

Создайте SceneScript.cs, добавьте его на пустой GameObject в сцене именуемый как SceneScript.

Затем создайте Canvas с текстом и кнопкой, аналогично приведенным ниже.

![](../.gitbook/assets/QS-image--011.jpg)

Добавьте переменную в sceneScript, функцию Awake, и CmdSendPlayerMessage в PlayerScript.cs Также добавьте новую строку с именем игрока в CmdSetupPlayer();

```csharp
private SceneScript sceneScript;

void Awake()
{
    //запускается на всех игроках
    sceneScript = GameObject.FindObjectOfType<SceneScript>();
}

[Command]
public void CmdSendPlayerMessage()
{
    if (sceneScript) 
        sceneScript.statusText = $"{playerName} says hello {Random.Range(10, 99)}";
}

[Command]
public void CmdSetupPlayer(string _name, Color _col)
{
    //информация об игроке отправляется на сервер, затем сервер обновляет sync var который обрабатывает это на всех клиентах
    playerName = _name;
    playerColor = _col;
    sceneScript.statusText = $"{playerName} joined.";
}

public override void OnStartLocalPlayer()
{
    sceneScript.playerScript = this;
    //. . . . ^ добавьте сюда новую строку
```

Добавьте данный код в скрипт SceneScript.cs

```csharp
using Mirror;
using UnityEngine;
using UnityEngine.UI;

namespace QuickStart
{
    public class SceneScript : NetworkBehaviour
    {
        public Text canvasStatusText;
        public PlayerScript playerScript;

        [SyncVar(hook = nameof(OnStatusTextChanged))]
        public string statusText;

        void OnStatusTextChanged(string _Old, string _New)
        {
            //вызывается из sync var hook, чтобы обновить информацию об игроке на экране всех игроков
            canvasStatusText.text = statusText;
        }

        public void ButtonSendMessage()
        {
            if (playerScript != null)  
                playerScript.CmdSendPlayerMessage();
        }
    }
}
```

* Прикрепите функцию ButtonSendMessage в вашу кнопку Canvas.
* Прикрепите текст на канвасе к переменной в SceneScript.
  * игнорируйте переменные SceneScript и playerScript, они будут назначены автоматически!
* Прикрепите компонент NetworkIdentity к объекту SceneScript, если это не было сделано автоматически.

![](https://mirror-networking.com/docs/Articles/CommunityGuides/MirrorQuickStartGuide/image--012.jpg) ![](../.gitbook/assets/QS-image--012.jpg) ![](../.gitbook/assets/image--013.jpg)

Теперь, если вы билдите и запускаете, хостите и присоединяетесь, вы можете отправлять сообщения и вести текстовый журнал действий!

Яхууу!

![](../.gitbook/assets/image--014.jpg) ![](../.gitbook/assets/image--015.jpg)

Экспериментируйте и приспосабливайтесь, получайте удовольствие!

![](../.gitbook/assets/QS-image--016.jpg)

### Часть 12 <a href="#part-12" id="part-12"></a>

Смена оружия! Кусок кода.

Добавьте следующее в свой скрипт PlayerScript.cs

```csharp
private int selectedWeaponLocal = 1;
public GameObject[] weaponArray;

[SyncVar(hook = nameof(OnWeaponChanged))]
public int activeWeaponSynced = 1;

void OnWeaponChanged(int _Old, int _New)
{
    // отключает прошлое оружие
    // в диапазоне, не равное null
    if (0 < _Old && _Old < weaponArray.Length && weaponArray[_Old] != null)
        weaponArray[_Old].SetActive(false);
    
    // включает новое оружие
    // в диапазоне, не равное null
    if (0 < _New && _New < weaponArray.Length && weaponArray[_New] != null)
        weaponArray[_New].SetActive(true);
}

[Command]
public void CmdChangeActiveWeapon(int newIndex)
{
    activeWeaponSynced = newIndex;
}

void Awake() 
{
    // отключает все оружия
    foreach (var item in weaponArray)
        if (item != null)
            item.SetActive(false); 
}
```

Добавьте кнопку смены оружия в Update. Только локальный игрок может поменять своё оружие, это делается при помощи проверки на `!isLocalPlayer`.

```csharp
void Update()
{
    if (!isLocalPlayer)
    {
        // make non-local players run this
        floatingInfo.transform.LookAt(Camera.main.transform);
        return;
    }

    float moveX = Input.GetAxis("Horizontal") * Time.deltaTime * 110.0f;
    float moveZ = Input.GetAxis("Vertical") * Time.deltaTime * 4f;

    transform.Rotate(0, moveX, 0);
    transform.Translate(0, 0, moveZ);

    if (Input.GetButtonDown("Fire2")) //Fire2 is mouse 2nd click and left alt
    {
        selectedWeaponLocal += 1;

        if (selectedWeaponLocal > weaponArray.Length) 
            selectedWeaponLocal = 1; 

        CmdChangeActiveWeapon(selectedWeaponLocal);
    }
}
```

### Часть 13 <a href="#part-13" id="part-13"></a>

Модели оружия

Сначала добавьте простой куб для оружия, вы можете изменить это позже.

* Кликните пару раз на prefab игрока чтобы войти в его настройки
* Добавьте пустой GameObject "WeaponsHolder" с позицией и вращением на 0,0,0.
* Внутри данного GameObject, создайте куб, (GameObject, 3D object, cube)- Удалите box коллайдер.
* Переименуйте это на `Weapon1`, измените положение и масштаб в соответствии с приведенными ниже изображениями.

![](../.gitbook/assets/QS-image--017.jpg)

Дублируйте Weapon 1 чтобы создать Weapon 2 и измените его масштаб и положение, теперь у вас должно получиться 2 разных вида ‘weapons’!

![](../.gitbook/assets/QS-image--018.jpg)

### Часть 14 <a href="#part-14" id="part-14"></a>

Финал смены оружия.

* Добавьте эти 2 объекта в ваш массив оружий скрипта PlayerScript.cs.
* Отключите второе ружье, чтобы при спавне в руках отображалось только 1 оружие.

![](../.gitbook/assets/QS-image--019.jpg)

Build and run!

Вы должны видеть, как каждый игрок меняет оружие, и все, чем оснащен ваш игрок, будет автоматически отображаться при появлении новых игроков (магия sync var и hook!)

![](../.gitbook/assets/QS-image--020.jpg)

### Часть 15 <a href="#part-15" id="part-15"></a>

Здесь мы внесем небольшую корректировку, так как используем GameObject.Find(), данный способ не может гарантировать что найдет в сцене объект именно с Network Identity. На изображении ниже вы можете видеть, что наш объект сцены NetworkIdentity отключен, поскольку они отключены до тех пор, пока игрок не будет в состоянии "готов" (статус готовности обычно устанавливается при появлении игрока).

![](../.gitbook/assets/QS-image--021.jpg)

Таким образом, выбранный нами обходной путь заключается в том, чтобы использовать GameObject.Find() для того чтобы найти не сетевой объект на сцене, у которого будет объект с Network Identity с предустановленными переменными.

Создайте новый скрипт с названием SceneReference.cs и добавьте в него только одну переменную.

```csharp
using UnityEngine;

namespace QuickStart
{
    public class SceneReference : MonoBehaviour
    {
        public SceneScript sceneScript;
    }
}
```

Откройте SceneScript.cs и добавьте следующую переменную.

```csharp
public SceneReference sceneReference;
```

Теперь в вашей сцене Unity создайте gameobject, назовите его SceneReference и добавьте новый скрипт. В обоих игровых объектах сцены установите ссылки друг на друга. Таким образом, SceneReference может обращаться к SceneScript, а SceneScript - к SceneReference.

![](../.gitbook/assets/QS-image--022.jpg)

Откройте PlayerScript.cs и перезапишите функцию Awake на эту:

```csharp
void Awake()
{
	//запускается на всех игроках
	sceneScript = GameObject.Find(“SceneReference”).GetComponent<SceneReference>().sceneScript;
}
```

### Часть 16 <a href="#part-16" id="part-16"></a>

Меню и переключение сцен, здесь мы перейдем из оффлайн меню по кнопке "играть" к списку игр с кнопкой возврата и HUD host / join, к вашей онлайн-карте, а затем ко второй карте, на которую можно переключиться хосту.

Откройте SceneScript.cs и добавьте следующую функцию.

```csharp
public void ButtonChangeScene()
{
    if (isServer)
    {
        Scene scene = SceneManager.GetActiveScene();
        if (scene.name == "MyScene")
            NetworkManager.singleton.ServerChangeScene("MyOtherScene");
        else
            NetworkManager.singleton.ServerChangeScene("MyScene");
    }
    else
        Debug.Log("You are not Host.");
}
```

![](../.gitbook/assets/QS-image--023.jpg)

Продублируйте вашу предыдущую кнопку Canvas, переименуйте ее и переместите, затем настройте OnClick() так, чтобы он указывал на SceneScript.ButtonChangeScene, как на изображении.

Затем перетащите свой NetworkManager в свой проект, чтобы сделать его prefab'ом, таким образом, любые изменения, которые мы внесем позже, будут применяться ко всем ним. Если вы еще этого не сделали, вы можете разложить свой проект по папкам, по одной для скриптов, prefab'ов, сцен, текстур и т.д. :)

![](../.gitbook/assets/QS-image--024.jpg)

### Часть 17 <a href="#part-17" id="part-17"></a>

Сохраните, а затем продублируйте свою сцену MyScene, переименуйте, чтобы создать меню, список игр и MyOtherScene, затем добавьте их в настройки сборки, причем меню должно быть первым.

![](../.gitbook/assets/QS-image--025.jpg)

Откройте сцену с меню, удалите точки спавна, SceneScript, SceneReference, Network Manager и Plane, чтобы это выглядело как показано ниже. Отрегулируйте кнопку на canvas так, чтобы она отображала "играть", отцентрируйте ее. Здесь вы могли бы добавить сцену с оценками, раздел контактов, новости и т.д

Создайте скрипт Menu.cs, добавьте его в объект Menu.

![](../.gitbook/assets/QS-image--026.jpg)

Добавьте код в Menu.cs, затем в кнопке перетащите игровой объект меню в On Click () и установите его в Menu.LoadScene, как на картинке.

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

namespace QuickStart
{
    public class Menu : MonoBehaviour
    {
        public void LoadScene()
        {
            SceneManager.LoadScene("GamesList");
        }
    }
}
```

![](../.gitbook/assets/QS-image--027.jpg)

### Часть 18 <a href="#part-18" id="part-18"></a>

Откройте сцену GamesList, сделайте аналогично меню, но СОХРАНИТЕ NetworkManager prefab.

Создайте файл GamesList.cs, добавьте код и добавьте его в GamesList gameobject в сцене. Отрегулируйте кнопку canvas так, чтобы она отображала меню (это наша кнопка "Назад"). Это должно выглядеть так, как показано на рисунке ниже.

* Список игр - это то место, куда вы можете добавить содержимое сервера списка, или matchmaker, или просто кнопки host и join, аналогично NetworkManagerHUD по умолчанию, пока оставьте это. :)

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

namespace QuickStart
{
    public class GamesList : MonoBehaviour
    {
        public void LoadScene()
        {
            SceneManager.LoadScene("Menu");
        }
    }
}
```

![](../.gitbook/assets/QS-image--028.jpg)

### Часть 19 <a href="#part-19" id="part-19"></a>

Откройте сцену MyOtherScene, это вторая карта. Измените цвет фона камеры и материал пола (или что-нибудь еще, просто чтобы вы могли видеть, что обе сцены разные. Подводя итог, можно сказать, что MyScene - это карта 1, а MyOtherScene - это карта 2.

![](../.gitbook/assets/QS-image--029.jpg)

В вашем prefab'е NetworkManager в ПРОЕКТЕ (не в сцене), добавьте в offline меню и MyScene в переменную Online. Это должно изменить все prefab'ы NetworkManager, чтобы они имели эти настройки.

![](../.gitbook/assets/QS-image--030.jpg)

Build and Run, нажмите Play в Menu для того чтобы перейти GamesList, затем кликните на Host (для игрока 1). для игрока 2 нажмите Play в Menu, ну а затем присоединитесь из GamesList.

Теперь хост может менять сцены между картой 1 и картой 2, и если кто-либо отключит или остановит игру, сцена меню загрузится, чтобы начать снова. Весь этот процесс можно упорядочить, но он должен обеспечить хороший шаблон переключения сцен для вашей игры на Mirror :)

### Часть 20 <a href="#part-20" id="part-20"></a>

Здесь мы добавим базовую стрельбу из оружия, используя prefab'ы с rigidbody. Обычно лучше использовать рейкастинг с изображением запущенного объекта и сохранять физические объекты для таких вещей, как гранаты и пушечные ядра. В этом разделе также будет отсутствовать множество приемов защиты от мошенничества, чтобы сделать руководство простым, но в любом случае, мы начинаем!

Дважды щелкните по prefab'у игрока, чтобы открыть ее, создайте пустые игровые объекты и выровняйте их с концом вашего оружия, добавьте их как дочерние к каждому оружию. Одним оружием могут быть короткие пистолеты, другим - длинные винтовки, поэтому место появления объектов будет разным.

![](../.gitbook/assets/QS-image--031.jpg)

Создайте скрипт Weapon.cs, а затем добавьте объекты Weapon1 и Weapon 2 внутрь prefab'a игрока.

```csharp
using UnityEngine;

namespace QuickStart
{
    public class Weapon : MonoBehaviour
    {
        public float weaponSpeed = 15.0f;
        public float weaponLife = 3.0f;
        public float weaponCooldown = 1.0f;
        public int weaponAmmo = 15;

        public GameObject weaponBullet;
        public Transform weaponFirePosition;
    }
}
```

### Часть 21 <a href="#part-21" id="part-21"></a>

Теперь вернемся в вашу сцену, мы создадим 2 пули, в меню Unity перейдите к GameObject, 3D Object, Sphere. Добавьте rigidbody к этой сфере, сделайте размер 0.2, 0.2, 0.2, затем сохраните его как prefab в проекте. Проделайте то же самое с кубиком, чтобы у вас получились две пули разного вида.

![](../.gitbook/assets/QS-image--032.jpg)

Снова зайдите в свой prefab игрока, выберите оружие и задайте переменные в сценарии оружия.

![](../.gitbook/assets/QS-image--033.jpg) ![](../.gitbook/assets/QS-image--034.jpg)

### Часть 22 <a href="#part-22" id="part-22"></a>

В SceneScript.cs добавьте эту переменную и функцию.

```csharp
public Text canvasAmmoText;
    
public void UIAmmo(int _value)
{
    canvasAmmoText.text = "Ammo: " + _value;
}
```

Войдите в MtScene (карта 1). Продублируйте Canvas statusText, переименуйте в Ammo, затем перетащите этот text UI Ammo в SceneScript gameobject, переменную canvasAmmoText. Сделайте это как на MyScene (карта 1), так и на MyOtherScene (карта 2), поскольку мы еще не связали или не подготовили наши сценарии canvas и scene для автоматического обновления изменений на каждой карте.

![](../.gitbook/assets/QS-image--035.jpg)

Откройте PlayerScript.cs, добавьте эти две переменные:

```csharp
private Weapon activeWeapon;
private float weaponCooldownTime;  
```

В функции ‘OnWeaponChanged’ обновите ее новой строкой, чтобы она выглядела следующим образом. 

```csharp
void OnWeaponChanged(int _Old, int _New)
{
    // disable old weapon
    // in range and not null
    if (0 < _Old && _Old < weaponArray.Length && weaponArray[_Old] != null)
        weaponArray[_Old].SetActive(false);
    
    // enable new weapon
    // in range and not null
    if (0 < _New && _New < weaponArray.Length && weaponArray[_New] != null)
    {
        weaponArray[_New].SetActive(true);
        activeWeapon = weaponArray[activeWeaponSynced].GetComponent<Weapon>();
        if (isLocalPlayer)
            sceneScript.UIAmmo(activeWeapon.weaponAmmo);
    }
}
```

В Awake() добавьте это в конце:

```csharp
if (selectedWeaponLocal < weaponArray.Length && weaponArray[selectedWeaponLocal] != null)
{
    activeWeapon = weaponArray[selectedWeaponLocal].GetComponent<Weapon>();
    sceneScript.UIAmmo(activeWeapon.weaponAmmo);
}
```

В Update() добавьте это в конце:

```csharp
if (Input.GetButtonDown("Fire1") ) //Fire1 is mouse 1st click
{
    if (activeWeapon && Time.time > weaponCooldownTime && activeWeapon.weaponAmmo > 0)
    {
        weaponCooldownTime = Time.time + activeWeapon.weaponCooldown;
        activeWeapon.weaponAmmo -= 1;
        sceneScript.UIAmmo(activeWeapon.weaponAmmo);
        CmdShootRay();
    }
}
```

Добавьте эти две функции после завершения работы функции Update() {}.

```csharp
[Command]
void CmdShootRay()
{
    RpcFireWeapon();
}

[ClientRpc]
void RpcFireWeapon()
{
    //bulletAudio.Play(); muzzleflash  etc
    GameObject bullet = Instantiate(activeWeapon.weaponBullet, activeWeapon.weaponFirePosition.position, activeWeapon.weaponFirePosition.rotation);
    bullet.GetComponent<Rigidbody>().velocity = bullet.transform.forward * activeWeapon.weaponSpeed;
    Destroy(bullet, activeWeapon.weaponLife);
}
```

Build and Run, у вас должна быть стрельба с разной скоростью и временем перезарядки у всех игроков :)

![](../.gitbook/assets/QS-image--036.jpg)
