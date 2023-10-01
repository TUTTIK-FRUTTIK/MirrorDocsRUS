# Сериализация

На этой странице подробно рассказывается о сериализации, основы смотрите в разделе [типы данных](data-types.md).

Mirror создает функции сериализации и десериализации для типов использующих Weaver. Weaver редактирует библиотеки dll после того, как unity скомпилирует их с помощью Mono.Cecil. Это позволяет Mirror иметь множество сложных функций, таких как SyncVar, ClientRpc и сериализация сообщений, без необходимости пользователю вручную все настраивать.

## Правила и советы <a href="#rules-and-tips" id="rules-and-tips"></a>

Существуют некоторые правила и ограничения для того, что может делать Weaver. Некоторые функции усложняют работу и их трудно поддерживать, поэтому они не были реализованы. Эти функции не являются невозможными для реализации и могут быть добавлены, если на них будет высокий спрос.

* Вы должны иметь возможность писать пользовательские функции Read / Write для любого типа, и Weaver будет использовать.
  * Это означает, что если существует неподдерживаемый тип, такой как `int[][]`, создайте свою Read/Write функцию чтобы синхронизировать `int[][]` в SyncVar/ClientRpc/и т.д.
* Если у вас есть тип, содержащий поле, которое невозможно сериализовать, вы можете пометить это поле с помощью `[System.NonSerialized]` и weaver будет игнорировать это

### Неподдерживаемые типы <a href="#unsupported-types" id="unsupported-types"></a>

Некоторые из этих типов не поддерживаются из-за добавленной ими сложности, как упоминалось выше.

> ПРИМЕЧАНИЕ: Типы в этом списке могут иметь пользовательские средства записи.

* Неровный и многомерный массив
* Типы унаследованные от `UnityEngine.Component`
* `UnityEngine.Object`
* `UnityEngine.ScriptableObject`
* Универсальные типы, например `MyData`
  * Пользовательское Read/Write должно объявлять T, например `MyData`
* Интерфейсы
* Типы, которые ссылаются сами на себя

### Встроенные функции Read Write <a href="#built-in-read-write-functions" id="built-in-read-write-functions"></a>

Mirror предоставляет некоторые встроенные функции Read/Write. Их можно найти в `NetworkReaderExtensions` и `NetworkWriterExtensions`.

Это неконкурентный список типов, которые имеют встроенные функции, проверьте классы выше, чтобы увидеть полный список.

* Некоторые [примитивные типы C#](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/built-in-types)
* Общие структуры Unity
  * Vector3
  * Quaternion
  * Rect
  * Ray
  * Guid
* NetworkIdentity, GameObject, Transform

### NetworkIdentity, GameObject, Transform

`netId` объекта отправляется по сети, и объект с таким же `netId` возвращается с другой стороны у клиента или сервера. Если `netId` равно нулю или объект не найден, тогда будет возвращено `null`.

### Сгенерированные функции Read Write <a href="#generated-read-write-functions" id="generated-read-write-functions"></a>

Weaver будет генерировать функции Read Write для

* Классов или структур
* Enums
* Массивов
  * например `int[]`
* ArraySegments
  * например `ArraySegment`
* Lists
  * например `List`

#### Классы и структуры <a href="#classes-and-structs" id="classes-and-structs"></a>

Weaver будет Read/Write каждое общедоступное поле в типе, если только поле не помечено знаком `[System.NonSerialized]`. Если в классе или структуре есть неподдерживаемый тип, Weaver не сможет выполнить функции чтения/записи для него.

> ПРИМЕЧАНИЕ: Weaver не проверяет свойства

#### Enums <a href="#enums" id="enums"></a>

Weaver будет использовать базовый тип перечисления для их чтения и записи. По умолчанию это `int`.

К примеру `Switch` будет использовать тип `byte` Read/Write функций чтобы сериализоваться

```csharp
public enum Switch : byte
{
    Left,
    Middle,
    Right,
}
```

#### Collections <a href="#collections" id="collections"></a>

Weaver сгенерирует записи для коллекций, перечисленных выше. Weaver будет использовать функцию чтения/записи элементов. Элемент должен иметь функцию чтения/ записи, поэтому должен быть поддерживаемого типа или иметь пользовательскую функцию чтения / записи.

К примеру:

* `float[]` является поддерживаемым типом, поскольку Mirror имеет встроенную функцию чтения / записи для `float`.
* `MyData[]` является поддерживаемым типом, поскольку Weaver способен генерировать функцию чтения / записи для `MyData`

```csharp
public struct MyData
{
    public int someValue;
    public float anotherValue;
}
```

## Добавление пользовательских функций Read/Write <a href="#adding-custom-read-write-functions" id="adding-custom-read-write-functions"></a>

Read Write функции - это статические методы в виде:

```csharp
public static void WriteMyType(this NetworkWriter writer, MyType value)
{
    // напишите данные MyType здесь
}

public static MyType ReadMyType(this NetworkReader reader)
{
    // прочитайте данные MyType здесь
}
```

Наилучшей практикой является создание функций чтения/записи [extension methods](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods) таким образом, их можно вызывать как `writer.WriteMyType(value)`.

Это хорошая идея - вызвать `ReadMyType` и `WriteMyType` таким образом, очевидно, для какого типа они предназначены. Однако название функции не имеет значения, weaver должен быть в состоянии найти ее независимо от того, как она называется.

#### Пример свойств <a href="#properties-example" id="properties-example"></a>

Weaver не записывает свойства, но для отправки их по сети можно использовать пользовательский writer.

Это может быть полезно, если вы хотите иметь приватный набор для своих свойств

```csharp
public struct MyData
{
    public int someValue { get; private set; }
    public float anotherValue { get; private set; }

    public MyData(int someValue, float anotherValue)
    {
        this.someValue = someValue;
        this.anotherValue = anotherValue;
    }
}

public static class CustomReadWriteFunctions 
{
    public static void WriteMyType(this NetworkWriter writer, MyData value)
    {
        writer.WriteInt32(value.someValue);
        writer.WriteSingle(value.anotherValue);
    }

    public static MyData ReadMyType(this NetworkReader reader)
    {
        return new MyData(reader.ReadInt32(), reader.ReadSingle());
    }
}
```

#### Пример неподдерживаемого типа <a href="#unsupported-type-example" id="unsupported-type-example"></a>

Rigidbody является неподдерживаемым типом, поскольку он наследуется от `Component`. Но можно добавить пользовательский writer, чтобы он синхронизировался с помощью NetworkIdentity если тот к нему привязан.

```csharp
public struct MyCollision
{
    public Vector3 force;
    public Rigidbody rigidbody;
}

public static class CustomReadWriteFunctions
{
    public static void WriteMyCollision(this NetworkWriter writer, MyCollision value)
    {
        writer.WriteVector3(value.force);

        NetworkIdentity networkIdentity = value.rigidbody.GetComponent<NetworkIdentity>();
        writer.WriteNetworkIdentity(networkIdentity);
    }

    public static MyCollision ReadMyCollision(this NetworkReader reader)
    {
        Vector3 force = reader.ReadVector3();

        NetworkIdentity networkIdentity = reader.ReadNetworkIdentity<NetworkIdentity>();
        Rigidbody rigidBody = networkIdentity != null
            ? networkIdentity.GetComponent()
            : null;

        return new MyCollision
        {
            force = force,
            rigidbody = rigidBody,
        };
    }
}
```

Выше приведены функции для `MyCollision`, но вместо этого вы могли бы добавить функции для `Rigidbody` и пусть weaver создаст write для `MyCollision`.

```csharp
public static class CustomReadWriteFunctions
{
    public static void WriteRigidbody(this NetworkWriter writer, Rigidbody rigidbody)
    {
        NetworkIdentity networkIdentity = rigidbody.GetComponent<NetworkIdentity>();
        writer.WriteNetworkIdentity(networkIdentity);
    }

    public static Rigidbody ReadRigidbody(this NetworkReader reader)
    {
        NetworkIdentity networkIdentity = reader.ReadNetworkIdentity();
        Rigidbody rigidBody = networkIdentity != null
            ? networkIdentity.GetComponent<Rigidbody>()
            : null;

        return rigidBody;
    }
}
```

## Debugging <a href="#debugging" id="debugging"></a>

Вы можете использовать такие инструменты, как [ILSpy](https://github.com/icsharpcode/ILSpy) и [dnSpy](https://github.com/0xd4d/dnSpy) чтобы просмотреть соответствующий код после того, как Weaver изменил его. Это может помочь понять и отладить то, что делают Mirror и Weaver.
