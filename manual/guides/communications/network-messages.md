# Network Messages

Для начала мы вам рекомендуем использовать [Commands и RPC](remote-actions.md) вызовы, а также [SyncVar](../synchronization/syncvars.md), но вы можете отправлять собственные низкоуровневые network messages. Это может быть полезно, если вы хотите, чтобы клиенты отправляли сообщения, не привязанные к игровым объектам, такие как ведение журнала, аналитика или информация о профилировании.

Существует общедоступный интерфейс под названием NetworkMessage, который вы можете расширить, чтобы создавать сериализуемые структуры сетевых сообщений. Этот интерфейс имеет функции сериализации и десериализации, которые принимают объекты writer и reader. Вы можете реализовать эти функции самостоятельно, но мы рекомендуем вам позволить Mirror сгенерировать их за вас.

Автоматические сгенерированные Serialize/Deserialize могут эффективно справляться с любыми [поддерживаемыми типами Mirror](../data-types.md). Сделайте своих участников общедоступными. Если вам нужны классы или сложные контейнеры, такие как List и Dictionary, вы можете реализовать собственные методы Serialize и Deserialize.

Чтобы отправить сообщение, используйте метод `Send()` в классах NetworkClient, NetworkServer, и NetworkConnection, которые все сработают одинаково, используя структуру сообщения, производную от NetworkMessage. Приведенный ниже код демонстрирует, как отправлять сообщение и обрабатывать его:

Чтобы объявить пользовательский класс Network Message и использовать его:

```csharp
using UnityEngine;
using Mirror;

public class Scores : MonoBehaviour
{
    public struct ScoreMessage : NetworkMessage
    {
        public int score;
        public Vector3 scorePos;
        public int lives;
    }

    public void SendScore(int score, Vector3 scorePos, int lives)
    {
        ScoreMessage msg = new ScoreMessage()
        {
            score = score,
            scorePos = scorePos,
            lives = lives
        };

        NetworkServer.SendToAll(msg);
    }

    public void SetupClient()
    {
        NetworkClient.RegisterHandler<ScoreMessage>(OnScore);
        NetworkClient.Connect("localhost");
    }

    public void OnScore(ScoreMessage msg)
    {
        Debug.Log("OnScoreMessage " + msg.score);
    }
}
```

Обратите внимание, что для нет кода сериализации для класса `ScoreMessage` в этом примере исходного кода. Тело функций сериализации автоматически генерируется для этого класса с помощью Mirror.
