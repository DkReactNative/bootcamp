# Получайте ваши внешние ресурсы вне Node

<br/><br/>

### Объяснение в один абзац

В классическом веб-приложении серверная часть предоставляет интерфейс/графику браузеру. Очень распространенным подходом в мире Node является использование статического промежуточного программного обеспечения Express для оптимизации статических файлов для клиента. НО - Node не является типичным веб-приложением, поскольку использует один поток, который не оптимизирован для одновременного обслуживания нескольких файлов. Вместо этого рассмотрите возможность использования обратного прокси-сервера (например, nginx, HAProxy), облачного хранилища или CDN (например, AWS S3, хранилища BLOB-объектов Azure и т.д.), который использует множество оптимизаций для этой задачи и обеспечивает гораздо лучшую пропускную способность. Например, специализированное промежуточное программное обеспечение, такое как nginx, воплощает прямые перехватчики между файловой системой и сетевой картой и использует многопоточный подход, чтобы минимизировать вмешательство в множественные запросы.

Ваше оптимальное решение может носить одну из следующих форм:

1. Используя обратный прокси-сервер - ваши статические файлы будут расположены прямо рядом с вашим приложением Node, только запросы к папке со статическими файлами будут обрабатываться прокси-сервером, который находится перед вашим приложением Node, таким как nginx. Используя этот подход, ваше Node-приложение отвечает за развертывание статических файлов, но не за их обслуживание. Коллеге вашего внешнего интерфейса понравится этот подход, поскольку он предотвращает запросы внешнего происхождения от внешнего интерфейса.

2. Облачное хранилище - ваши статические файлы НЕ будут частью содержимого приложения Node, они будут загружены в такие сервисы, как AWS S3, Azure BlobStorage или другие подобные сервисы, созданные для этой миссии. Используя этот подход, ваше Node-приложение не несет ответственности за развертывание статических файлов и за их обслуживание, поэтому между Node и Frontend проводится полная развязка, которая в любом случае обрабатывается различными командами.

<br/><br/>

### Пример конфигурации: типичная конфигурация nginx для обслуживания статических файлов

```nginx
# configure gzip compression
gzip on;
keepalive 64;

# defining web server
server {
listen 80;
listen 443 ssl;

# handle static content
location ~ ^/(images/|img/|javascript/|js/|css/|stylesheets/|flash/|media/|static/|robots.txt|humans.txt|favicon.ico) {
root /usr/local/silly_face_society/node/public;
access_log off;
expires max;
}
```

<br/><br/>

### Что говорят другие блоггеры

Из блога [StrongLoop](https://strongloop.com/strongblog/best-practices-for-express-in-production-part-two-performance-and-reliability/):

> … В процессе разработки вы можете использовать [res.sendFile()](http://expressjs.com/4x/api.html#res.sendFile) для обслуживания статических файлов. Но не делайте этого в производственном процессе, потому что эта функция должна считывать данные из файловой системы для каждого запроса файла, поэтому она сталкивается со значительной задержкой и влияет на общую производительность приложения. Обратите внимание, что res.sendFile() не реализован с системным вызовом sendfile, что сделает его гораздо более эффективным. Вместо этого используйте статическое промежуточное программное обеспечение (или что-то подобное), оптимизированное для обслуживания файлов для приложений Express. Еще лучшим вариантом является использование обратного прокси-сервера для обслуживания статических файлов; см. Использование обратного прокси-сервера для получения дополнительной информации ...

<br/><br/>
