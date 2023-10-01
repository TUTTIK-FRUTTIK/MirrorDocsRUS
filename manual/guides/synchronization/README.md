# Синхронизация

Синхронизация состояний относится к синхронизации значений, таких как целые числа, числа с плавающей запятой, строки и логические значения, принадлежащие скриптам.

Синхронизация состояния осуществляется с сервера на удаленные клиенты. У локального клиента нет сериализованных данных. Ему это не нужно, потому что он разделяет сцену с сервером. Однако hook'и SyncVar вызываются на локальных клиентах.

Данные не синхронизируются в обратном направлении - от клиентов к серверу. Чтобы синхронизировать данные, вам нужно использовать \[Command].

* [SyncVars](syncvars.md)\
  SyncVars - это переменные в скриптах, наследуемые от NetworkBehaviour, которые синхронизируются с сервера на клиенты.
* [SyncEvents (Устарело)](syncevent.md)\
  SyncEvents - это сетевые события, подобные событиям ClientRpc, но вместо вызова функции для игрового объекта они запускают события. **ВАЖНО:** это было удалено в версии 18.0.0, смотрите здесь [Issue](https://github.com/vis2k/Mirror/pull/2178) для получения более подробной информации.
* [SyncLists](synclists.md)\
  SyncList содержат list'ы значений и синхронизируют данные с сервера на клиенты.
* [SyncDictionary](syncdictionary.md)\
  SyncDictionary - это ассоциативный массив, содержащий неупорядоченный список пар ключ-значение.
* [SyncHashSet](synchashset.md)\
  Неупорядоченный набор значений, которые не повторяются.
* [SyncSortedSet](syncsortedset.md)\
  Отсортированный набор значений, которые не повторяются.

## Sync To Owner <a href="#sync-to-owner" id="sync-to-owner"></a>

Часто бывает так, что вы не хотите, чтобы некоторые данные игрока были видны другим игрокам. В инспекторе измените "Network Sync Mode" с "Observers" (по умолчанию) на "Owner" чтобы сообщить Mirror о необходимости синхронизации данных только с клиентом-владельцем.

Например, предположим, что вы создаете систему инвентаря. Предположим, игроки A, B и C находятся в одной и той же области. Всего во всей сети будет 12 объектов:

* У клиента A на карте есть игрок A (он сам), игрок B и игрок C
* У клиента B на карте есть игрок A, игрок B (он сам) и игрок C
* У клиента C на карте есть игрок A, игрок B и игрок C (он сам)
* На сервере есть игрок A, Игрок B, игрок C

у каждого из них есть компонент инвентаря

Предположим, игрок А подбирает какую-то добычу. Сервер добавляет добычу в инвентарь игрока A, который будет иметь [SyncLists](synclists.md) из предметов.

По умолчанию Mirror теперь должен синхронизировать инвентарь игрока A везде, это означает что он отправит сообщения об обновлении клиенту A, клиенту B и клиенту C, потому что у всех них есть копия игрока A. Это расточительно, клиенту B и клиенту C не нужно знать об инвентаре игрока A, они никогда не увидят этого на экране. Это также проблема безопасности, кто-то может взломать клиент и отобразить инвентарь других людей и использовать его в своих интересах.

Если вы установите для параметра "Network Sync Mode" в компоненте инвентаря значение "Owner", то инвентарь игрока A будет синхронизирован только с клиентом A.

Теперь предположим, что вместо 3 человек у вас в районе 50 человек, и один из них собирает добычу. Это означает, что вместо отправки 50 сообщений 50 разным клиентам вы отправите только 1. Это может оказать большое влияние на пропускную способность вашей игры.

Другие типичные варианты использования включают квесты, руку игрока в карточной игре, навыки, опыт или любые другие данные, которыми вам не нужно делиться с другими игроками.

## Расширенная синхронизация состояний <a href="#advanced-state-synchronization" id="advanced-state-synchronization"></a>

В большинстве случаев использования SyncVars достаточно для того, чтобы ваши игровые скрипты сериализовали свое состояние для клиентов. Однако в некоторых случаях вам может потребоваться более сложный код сериализации. Эта страница актуальна только для продвинутых разработчиков, которым нужны индивидуальные решения для синхронизации, выходящие за рамки обычной функции Mirror SyncVar.

## Пользовательские функции сериализации <a href="#custom-serialization-functions" id="custom-serialization-functions"></a>

Чтобы выполнить свою собственную пользовательскую сериализацию, вы можете реализовать виртуальные функции из NetworkBehaviour, которые будут использоваться для сериализации SyncVar. Этими функциями являются:

```csharp
public virtual bool OnSerialize(NetworkWriter writer, bool initialState);
```

```csharp
public virtual void OnDeserialize(NetworkReader reader, bool initialState);
```

Используйте флаг `initialState` чтобы различать, когда игровой объект сериализуется в первый раз, и когда могут быть отправлены дополнительные обновления. При первой отправке игрового объекта клиенту он должен содержать полный снимок состояния, но последующие обновления могут сэкономить на пропускной способности, включая только постепенные изменения.

Функция `OnSerialize` должно возвращать значение true, указывающее на то, что обновление должно быть отправлено. Если он возвращает значение true, то биты dirty для этого скрипта устанавливаются равными нулю. Если он возвращает значение false, то биты dirty не изменяются. Это позволяет накапливать множество изменений в сценарии с течением времени и отправлять их, когда система будет готова, вместо каждого кадра.

Функция `OnSerialize` вызывается только для `initialState` или когда `NetworkBehaviour` является dirty. `NetworkBehaviour` будет dirty только в том случае, если `SyncVar` или `SyncObject` (например `SyncList`) изменились с момента последнего вызова OnSerialize. После того, как данные были отправлены, `NetworkBehaviour` больше не будет dirty до следующего `syncInterval` (установите в инспекторе). `NetworkBehaviour` также может быть помечен как dirty при ручном вызове `SetDirtyBit` (это не позволяет обойти ограничение syncInterval).

Хоть это и работает, обычно лучше позволить Mirror генерировать эти методы и предоставлять [пользовательские сериализаторы](../serialization.md) для вашего специфичного поля.

## Поток сериализации <a href="#serialization-flow" id="serialization-flow"></a>

Игровой объект с прикрепленным компонентом Network Identity может содержать несколько скриптов, унаследованных от `NetworkBehaviour`. Процесс сериализации этих игровых объектов выглядит следующим образом:

На сервере:

* Каждый `NetworkBehaviour` имеет dirty mask. Эта маска доступна внутри `OnSerialize` как `syncVarDirtyBits`
* Каждому SyncVar в скрипте `NetworkBehaviour` назначается бит в dirty mask.
* Изменение значения SyncVars приводит к тому, что бит для этого SyncVar устанавливается в dirty mask
* В качестве альтернативы, вызывается `SetDirtyBit` который непосредственно записывает бит в dirty mask
* Игровые объекты с NetworkIdentity проверяются на сервере как часть цикла обновления
* Если какой нибудь `NetworkBehaviour` у `NetworkIdentity` является dirty, тогда пакет `UpdateVars` будет создан для данного игрового объекта
* Пакет `UpdateVars` заполняется вызовом `OnSerialize` на каждый `NetworkBehaviour` на игровом объекте
* `NetworkBehaviours` которые не являются dirty, запишут ноль в пакет для их dirty битов
* `NetworkBehaviours` которые являются dirty, записывают свою dirty mask, затем значения для измененных синхронизаторов
* Если `OnSerialize` возвращает true у `NetworkBehaviour`, dirty mask будет сброшена для данного `NetworkBehaviour`, таким образом, он не будет отправляться снова до тех пор, пока его значение не изменится.
* пакет `UpdateVars` отправляется готовым клиентам, которые наблюдают за игровым объектом

На клиенте:

* `UpdateVars packet` будет получен для игрового объекта
* Функция `OnDeserialize` будет вызвана для каждого скрипта `NetworkBehaviour` на игровом объекте
* Каждый скрипт `NetworkBehaviour` на игровом объекте прочитает dirty mask.
* Если dirty mask для `NetworkBehaviour` является нулевой, функция `OnDeserialize` вернется, больше ничего не читая
* Если dirty mask не является нулевым значением, тогда функция `OnDeserialize` прочитает значение для SyncVar которые соответствуют установленным dirty битам
* Если есть функции SyncVar hook, они вызываются со значением, считанным из потока.

Итак, для этого скрипта:

```csharp
public class data : NetworkBehaviour
{
    [SyncVar(hook = nameof(OnInt1Changed))]
    public int int1 = 66;

    [SyncVar]
    public int int2 = 23487;

    [SyncVar]
    public string MyString = "Example string";

    void OnInt1Changed(int oldValue, int newValue)
    {
        // do something here
    }
}
```

В следующем примере показан код, сгенерированный Mirror для функции `SerializeSyncVars` которая вызывается внутри `NetworkBehaviour.OnSerialize`:

```csharp
public override bool SerializeSyncVars(NetworkWriter writer, bool initialState)
{
    // Записывает любые SyncVars в базовом классе
    bool written = base.SerializeSyncVars(writer, forceAll);

    if (initialState)
    {
        // При первой отправке игрового объекта клиенту отправьте все данные (и никаких dirty битов)
        writer.WritePackedUInt32((uint)this.int1);
        writer.WritePackedUInt32((uint)this.int2);
        writer.Write(this.MyString);
        return true;
    }
    else 
    {
        // Записывает, какие SyncVar были изменены
        writer.WritePackedUInt64(base.syncVarDirtyBits);

        if ((base.get_syncVarDirtyBits() & 1u) != 0u)
        {
            writer.WritePackedUInt32((uint)this.int1);
            written = true;
        }

        if ((base.get_syncVarDirtyBits() & 2u) != 0u)
        {
            writer.WritePackedUInt32((uint)this.int2);
            written = true;  
        }

        if ((base.get_syncVarDirtyBits() & 4u) != 0u)
        {
            writer.Write(this.MyString);
            written = true;     
        }

        return written;
    }
}
```

В следующем примере показан код, сгенерированный Mirror для функции `DeserializeSyncVars` которая вызывается внутри `NetworkBehaviour.OnDeserialize`:

```csharp
public override void DeserializeSyncVars(NetworkReader reader, bool initialState)
{
    // Читает все SyncVars в базовом классе
    base.DeserializeSyncVars(reader, initialState);

    if (initialState)
    {
        // При первой отправке игрового объекта клиенту считайте все данные (и никаких dirty битов)
        int oldInt1 = this.int1;
        this.int1 = (int)reader.ReadPackedUInt32();
        // если старые и новые значения не равны, вызовите hook
        if (!base.SyncVarEqual(num, ref this.int1))
        {
            this.OnInt1Changed(num, this.int1);
        }

        this.int2 = (int)reader.ReadPackedUInt32();
        this.MyString = reader.ReadString();
        return;
    }

    int dirtySyncVars = (int)reader.ReadPackedUInt32();
    // является ли 1- й SyncVar dirty
    if ((dirtySyncVars & 1) != 0)
    {
        int oldInt1 = this.int1;
        this.int1 = (int)reader.ReadPackedUInt32();
        // если старые и новые значения не равны, вызовите hook
        if (!base.SyncVarEqual(num, ref this.int1))
        {
            this.OnInt1Changed(num, this.int1);
        }
    }

    // является ли 2- й SyncVar dirty
    if ((dirtySyncVars & 2) != 0)
    {
        this.int2 = (int)reader.ReadPackedUInt32();
    }

    // является ли 3- й SyncVar dirty
    if ((dirtySyncVars & 4) != 0)
    {
        this.MyString = reader.ReadString();
    }
}
```

Если `NetworkBehaviour` имеет базовый класс, который также имеет функции сериализации, функции базового класса также должны вызываться.

Обратите внимание, что пакеты `UpdateVar` обновления состояния, созданные для игровых объектов, могут быть объединены в буферах перед отправкой клиенту, поэтому один пакет транспортного уровня может содержать обновления для нескольких игровых объектов.
