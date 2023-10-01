# SyncVar Hook

Атрибут hook можно использовать для указания функции, которая будет вызываться при изменении значения SyncVar.

* Метод Hook должен иметь два параметра того же типа, что и свойство SyncVar. Один для старого значения, один для нового значения.
* Hook всегда вызывается после установки значения свойства. Вам не нужно устанавливать его самостоятельно.
* Hook срабатывает только для измененных значений, и изменение значения в инспекторе не вызовет обновления.
* Начиная с версии 11.1.4 (Март 2020) и позже, hook могут быть виртуальными методами и переопределяться в производном классе.

Ниже приведен простой пример присвоения случайного цвета каждому игроку при его появлении на сервере. Все клиенты будут видеть всех игроков в правильных цветах, даже если они присоединятся позже.

> Примечание: Сигнатура для методов перехвата была изменена в версии 9.0 (февраль 2020 г.) на имеющую 2 параметра (старые и новые значения). Если вы используете более старую версию, методы hook имеют только один параметр (новое значение).

```csharp
using UnityEngine;
using Mirror;

public class PlayerController : NetworkBehaviour
{
    [SyncVar(hook = nameof(SetColor))]
    Color playerColor = Color.black;

    // Unity создает клон материала каждый раз, когда используется GetComponent().material.
    // Кэшируйте его здесь и уничтожьте в onDestroy, чтобы предотвратить утечку памяти.
    Material cachedMaterial;

    public override void OnStartServer()
    {
        base.OnStartServer();
        playerColor = Random.ColorHSV(0f, 1f, 1f, 1f, 0.5f, 1f);
    }

    void SetColor(Color oldColor, Color newColor)
    {
        if (cachedMaterial == null)
            cachedMaterial = GetComponent().material;

        cachedMaterial.color = newColor;
    }

    void OnDestroy()
    {
        Destroy(cachedMaterial);
    }
}
```

## Порядок вызова Hook <a href="#hook-call-order" id="hook-call-order"></a>

Hooks вызываются в том порядке, в котором syncvars определены в файле.

```csharp
public class MyBehaviour : NetworkBehaviour 
{
    [SyncVar] 
    int X;

    [SyncVar(hook = nameof(Hook1))] 
    int Y;

    [SyncVar(hook = nameof(Hook2))]
    int Z;
}
```

если все X, Y и Z установлены на сервере одновременно, то порядок вызовов будет следующим:

1. X значение установлено
2. Y значение установлено
3. Hook1 вызван
4. Z значение установлено
5. Hook2 вызван
