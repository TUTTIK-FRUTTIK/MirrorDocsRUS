# Network Rigidbody

> Network Rigidbody помечен как "Экспериментальный" на данный момент, пожалуйста, делитесь любыми проблемами или багами, которые вы обнаружите в нем, и используйте на свой страх и риск, если будете создавать готовую игру использующую данный компонент.

Компонент Network Rigidbody синхронизирует velocity и другие параметры rigidbody по всей сети. Этот компонент полезен, когда у вас есть не kinematic rigidbody к которому постоянно применяется сила, к примеру гравитация, но также хотите применить силу или изменить скорость к этому Rigidbody, серверу или клиенту с полномочиями. Например, объекты, которые перемещаются и прыгают с помощью Rigidbody, использующего гравитацию.

Объект с компонентом Network Rigidbody должен также иметь компонент Network Identity. Когда вы добавляете компонент Network Rigidbody на GameObject, Mirror также добавляет компонент Network Identity на этот объект, если он не был добавлен ранее.

Network Rigidbody лучше всего работает, когда есть также NetworkTransform чтобы объект сохранял синхронное положение и скорость.

![](<../../.gitbook/assets/image (39) (1).png>)

По умолчанию, Network Rigidbody является серверо-авторитарным, если вы не установите флажок на поле **Client Authority**. Client Authority применяется как к игровым объектам, так и к неигровым объектам, которые были специально назначены клиенту, но только для этого компонента. Если этот параметр включен, изменения значений отправляются с клиента на сервер.

Опция **Sensitivity** позволяет вам установить минимальные пороговые значения перед отправкой значений по сети. Это помогает минимизировать сетевой трафик при очень небольших изменениях.

Для некоторых объектов вы можете не захотеть, чтобы они вращались, но вам не нужно синхронизировать Angular Velocity. **Clear Angular Velocity** установит угловую скорость равной нулю для каждого кадра, что приведет к минимизации вращении объектов. То же самое относится и к **Clear Velocity**. Если **Clear Velocity Velocity** включен, то очистка игнорируется.

Обычно изменения отправляются всем наблюдателям объекта, на котором находится этот компонент. Настройте **Sync Mode** на Owner Only чтобы сделать изменения приватными для сервера и клиента владельца.

Вы можете использовать **Sync Interval** чтобы указать, как часто он будет синхронизироваться (в секундах). Это относится как к Client Authority, так и к Server Authority.
