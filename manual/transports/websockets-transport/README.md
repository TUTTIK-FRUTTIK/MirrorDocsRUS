# WebSockets Transport

Транспорт, использующий протокол websocket. Это позволяет использовать этот транспорт в WebGL-сборках unity.

![Simple Web Transport Inspector](<../../../.gitbook/assets/image (1) (1) (1).png>)

## Logging <a href="#logging" id="logging"></a>

Log levels можно выставлять в выпадающем списке в транспорте или настроить `Mirror.SimpleWeb.Log.level`.

Транспорт принимает выпадающее значение в  методах `Awake` и `OnValidate`.

#### Log methods <a href="#log-methods" id="log-methods"></a>

Log methods в транспорте [ConditionalAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.conditionalattribute?view=netstandard-2.0) удаляются в зависимости от того, что определяет препроцессор.

Эти определения препроцессора влияют на ведение журнала

* `DEBUG` оставляет warn/error логи
* `SIMPLEWEB_LOG_ENABLED` оставляет все логи

Без `SIMPLEWEB_LOG_ENABLED` информации или подробное ведение журнала никогда не произойдет, даже если уровень журнала это позволяет.

Смотрите [Unity docs](https://docs.unity3d.com/Manual/PlatformDependentCompilation.html) о том как поставить собственные определения.

## Установка SSL

{% hint style="info" %}
ПРИМЕЧАНИЕ: WebGL работает намного лучше с обратным прокси-сервером, и это, как правило, проще в настройке и обслуживании, чем использование файлов cert.json и PFX.

\
Посмотрите инструкции на странице [Reverse Proxy](reverse-proxy.md).
{% endhint %}

Если вы размещаете свою сборку webgl в домене https, вам нужно будет использовать wss, для чего потребуется сертификат ssl.

### Pre-Setup

* Вам нужно доменное имя
  * С записью DNS, указывающей на облачный сервер
* Настройка облачного сервера: [Как настроить облачный сервер google](https://mirror-networking.com/docs/Articles/Guides/DevServer/gcloud/index.html)

> примечание: Возможно, вам потребуется открыть порт 80 для certbot

### Получить сертификат

Следуйте инструкциям здесь:

[https://letsencrypt.org/getting-started/](https://letsencrypt.org/getting-started/) [https://certbot.eff.org/instructions](https://certbot.eff.org/instructions)

Найдите инструкции для вашей версии сервера, ниже приведена ссылка для `Ubuntu 18.04 LTS (bionic)`

[https://certbot.eff.org/lets-encrypt/ubuntubionic-other](https://certbot.eff.org/lets-encrypt/ubuntubionic-other)

Для инструкции 7

```
sudo certbot certonly --standalone
```

После заполнения деталей вы получите результат, подобный этому

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/simpleweb.example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/simpleweb.example.com/privkey.pem
   Your cert will expire on 2021-01-07. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

`simpleweb.example.com` должно быть вашим доменом

### Создайте cert.pfx

Чтобы создать файл pfx, который может использовать SimpleWebTransport, запустите эту команду в папке `/etc/letsencrypt/live/simpleweb.example.com/`

```
openssl pkcs12 -export -out cert.pfx -inkey privkey.pem -in cert.pem -certfile chain.pem
```

Вам будет предложено ввести пароль, вы можете установить пароль или оставить его пустым.

Возможно, вам потребуются права суперпользователя, чтобы сделать это:

```
su

cd /etc/letsencrypt/live/simpleweb.example.com/
```

Примечание: В настоящее время версия mono, поставляемая с unity, не может загружать файлы pfx, сгенерированные OpenSSL версии 3. Вам нужно будет добавить `-legacy` аргумент командной строки для приведенной выше команды openssl для создания совместимого файла pfx.

### Используйте cert.pfx

Вы можете либо скопировать файл cert.pfx в папку вашего сервера, либо создать символическую ссылку

Перемещение

```
mv /etc/letsencrypt/live/simpleweb.example.com/cert.pfx ~/path/to/server/cert.pfx
```

Символическая ссылка

```
ln -s /etc/letsencrypt/live/simpleweb.example.com/cert.pfx ~/path/to/server/cert.pfx
```

#### создайте файл cert.json

Создайте `cert.json` который SimpleWebTransport сможет читать

Выполните команду в папке `~/path/to/server/`

Если вы оставили пароль пустым при создании сертификата:

```
echo '{ "path":"./cert.pfx", "password": "" }' > cert.json
```

Если вы установили пароль "yourPassword" при создании сертификата:

```
echo '{ "path":"./cert.pfx", "password": "yourPassword" }' > cert.json
```

#### Запустите ваш сервер

После `cert.json` и `cert.pfx` серверная папка будет выглядеть так:

```
ServerFolder
|- demo_server.x86_64
|- cert.json
|- cert.pfx
```

Затем сделайте серверный файл исполняемым

```
chmod +x demo_server.x86_64
```

Для запуска в активном терминале используйте

```
./demo_server.x86_64
```

Для запуска в фоновом режиме используйте

```
nohup ./demo_server.x86_64 &
```

> `nohup` подразумевает: Для запуска в фоновом режиме используйте исполняемый файл, который продолжит работать после того, как вы закроете сеанс ssh. Знак `&` означает, что ваш сервер будет работать в фоновом режиме

> возможно вам понадобится использовать `sudo` чтобы запустить вашу символическую ссылку

#### Подключитесь к вашей игре

Проверьте, все ли работает при подключении с помощью редактора или сборки

установите свой домен (`simpleweb.example.com`) в поле имени хоста, а затем запустите клиент

### Debugging

Чтобы проверить, работает ли ваш pfx-файл за пределами unity, вы можете использовать `pfxTestServer.js`.

Чтобы использовать это, установите `nodejs` затем установите путь к pfx и запустите его с помощью `node pfxTestServer.js`

После этого вы сможете посетить `https://simpleweb.example.com:8000` и получите ответ сервера (измените порт и домен в соответствии с вашими потребностями)
