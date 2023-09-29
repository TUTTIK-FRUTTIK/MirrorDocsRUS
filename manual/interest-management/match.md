---
description: Match Interest Management
---

# Match

## Match Interest Management

{% hint style="danger" %}
Не используйте это для игр, в которых есть физика... Если она есть, используйте вместо данного компонента компонент [**Scene Interest Management**](scene.md).
{% endhint %}

Match Interest Management предназначен для нефизических игр, таких как карточные, настольные, аркадные игры.

### Перед началом

Добавьте компонент **Match Interest Management** на объект, который имеет компонент **Network Manager**:

![](<../../.gitbook/assets/image (68).png>)

И добавьте компонент **Network Match** всем сетевым объектам включая prefab игрока, это будет задействовано в матче.

![](<../../.gitbook/assets/image (47).png>)

Во время выполнения назначайте один и тот же`matchId` игрокам и объектам, которые учавствуют другом в матче, тоесть у объектов конкретного матча должен быть один и тот же  `matchId`.

Посмотрите наш пример **Multiple Matches**, который идет в комплекте с Mirror для справки и вдохновения.
