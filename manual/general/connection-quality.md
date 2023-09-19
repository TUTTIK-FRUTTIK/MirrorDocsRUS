# Качество подключения

Mirror много работает над устранением плохих сетевых подключений.

Однако, мы можем сделать не так уж и много.

{% hint style="warning" %}
При плохом соединении рекомендуется выводить предупреждение о "**плохом соединении**".
{% endhint %}

ConnectionQuality в Mirror состоит из трёх частей, чтобы сделать всё довольно простым для вашей игры.

### **ConnectionQuality.cs**

Автономные connection quality enum и эвристика качества подключения, которые при необходимости можно использовать за пределами Unity.

Mirror обеспечивает следующие уровни качества соединения

```csharp
public enum ConnectionQuality : byte
{
    EXCELLENT,  // идеальный опыт для клиентов с отличным соединением
    GOOD,       // очень играбельно для всех, кроме ребят с лучшим соединением
    FAIR,       // очень заметная задержка, уже не очень приятно
    POOR,       // неиграбельно
    ESTIMATING, // ещё проверяется
}
```

В настоящее время мы предлагаем две эвристики:

* **Simple** (основано на пинге и дрожании)
* **Pragmatic** (основано на Snapshot Interpolation).

### NetworkPingDisplay

Этот компонент можно добавить в NetworkManager для отображения индикатора Ping и качества соединения в правом нижнем углу экрана. Не стесняйтесь изменять это в соответствии с вашими потребностями или создавать свои собственные.

<figure><img src="../../.gitbook/assets/2023-06-25 - connection quality, gui, callback.png" alt=""><figcaption></figcaption></figure>

### **NetworkManager Callbacks**

*   **CalculateConnectionQuality()** может быть перезаписано, чтобы ввести вашу собственную эвристику. По умолчанию, оно использует **Simple** эвристику. Это вызывается каждые **connectionQualityInterval** в секундах, которое сконфигурировано в the NetworkManager.

    ```csharp
    protected virtual void CalculateConnectionQuality()
    {
        NetworkClient.connectionQuality = ConnectionQualityHeuristics.Simple(
            NetworkTime.rtt, 
            NetworkTime.rttVar
        );
    }
    ```
*   **OnConnectionQualityChanged**() может быть использовано для показа предупреждения клиенту. По умолчанию, оно отправляет сообщение в логи - юзабельно для дебага и т.д.

    ```csharp
    protected virtual void OnConnectionQualityChanged(ConnectionQuality previous, ConnectionQuality current)
    {
        Debug.Log($"[Mirror] Connection Quality changed from {previous} to {current}");
    }
    ```
