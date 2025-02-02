# Просмотр содержимого веб-страницы для утечки информации

|ID          |
|------------|
|WSTG-INFO-05|

## Резюме

Программистам очень часто и даже рекомендуется включать подробные комментарии и метаданные в свой исходный код. Однако комментарии и метаданные, включенные в HTML-код, могут раскрывать внутреннюю информацию, которая не должна быть доступна потенциальным злоумышленникам. Комментарии и проверка метаданных должны быть сделаны, чтобы определить, не просочилась ли какая-либо информация.

Для современных веб-приложений использование клиентского JavaScript для внешнего интерфейса становится все более популярным. Популярные передовые технологии строительства используют клиентский JavaScript, такой как ReactJS, AngularJS или Vue.  Подобно комментариям и метаданным в HTML-коде, многие программисты также обрабатывают конфиденциальную информацию в переменных JavaScript на интерфейсе. Чувствительная информация может включать (но не ограничивается): частные ключи API (* например,.* неограниченный ключ API карты Google), внутренние IP-адреса, конфиденциальные маршруты (* например,.* маршрут к скрытым страницам администратора или функциональности) или даже учетным данным. Эта конфиденциальная информация может быть утечка из такого внешнего кода JavaScript. Необходимо провести обзор, чтобы определить, не просочилась ли какая-либо конфиденциальная информация, которая может быть использована злоумышленниками для злоупотреблений.

Для крупных веб-приложений проблемы с производительностью вызывают серьезную обеспокоенность у программистов. Программисты использовали различные методы для оптимизации производительности внешнего интерфейса, включая синтетические таблицы стилей (SASS), Sassy CSS (SCSS), веб-пакет и т. Д. Используя эти технологии, код внешнего интерфейса иногда становится все труднее понять и его трудно отладить, и из-за этого программисты часто развертывают файлы исходной карты для целей отладки. «Карта источника» - это специальный файл, который соединяет минифицированную / уродливую версию актива (CSS или JavaScript) с оригинальной версией автора. Программисты все еще обсуждают, приносить ли файлы исходной карты в производственную среду. Тем не менее, нельзя отрицать, что файлы карты источника или файлы для отладки, если они будут выпущены в производственную среду, сделают их источник более читаемым человеком. Злоумышленникам может быть проще находить уязвимости на интерфейсе или собирать с них конфиденциальную информацию. Проверка кода JavaScript должна быть сделана, чтобы определить, выставлены ли какие-либо отладочные файлы с внешнего интерфейса. В зависимости от контекста и чувствительности проекта эксперт по безопасности должен решить, должны ли файлы существовать в производственной среде или нет.

## Цели теста

- Просмотрите комментарии и метаданные веб-страницы, чтобы найти утечку информации.
- Соберите файлы JavaScript и просмотрите код JS, чтобы лучше понять приложение и найти утечку информации.
- Определите, существуют ли исходные файлы карты или другие внешние файлы отладки.

## Как проверить

### Просмотр комментариев и метаданных на веб-странице

HTML-комментарии часто используются разработчиками для включения отладочной информации о приложении. Иногда они забывают о комментариях и оставляют их в производственной среде. Тестеры должны искать комментарии HTML, которые начинаются с `<!--`.

Проверьте исходный код HTML для комментариев, содержащих конфиденциальную информацию, которая может помочь злоумышленнику получить более полное представление о приложении. Это может быть код SQL, имена пользователей и пароли, внутренние IP-адреса или информация об отладке.

```html
...
<div class="table2">
  <div class="col1">1</div><div class="col2">Mary</div>
  <div class="col1">2</div><div class="col2">Peter</div>
  <div class="col1">3</div><div class="col2">Joe</div>

<!-- Query: SELECT id, name FROM app.users WHERE active='1' -->

</div>
...
```

Тестер может даже найти что-то вроде этого:

```html
<!-- Use the DB administrator password for testing:  f@keP@a$$w0rD -->
```

Проверьте информацию о версии HTML для действительных номеров версий и URL-адресов определения типа данных (DTD)

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
```

- `strict.dtd` - строгий DTD по умолчанию
- `loose.dtd` - свободный DTD
- `frameset.dtd` - DTD для документов с рамками

екоторые теги `META` не предоставляют активных векторов атаки, а позволяют злоумышленнику профилировать приложение:

```html
<META name="Author" content="Andrew Muller">
```

Обычный (но нет [WCAG](https://www.w3.org/WAI/standards-guidelines/wcag/) совместимый) тег `META` есть [Refresh](https://en.wikipedia.org/wiki/Meta_refresh).

```html
<META http-equiv="Refresh" content="15;URL=https://www.owasp.org/index.html">
```

Обычное использование тега `META` - указывать ключевые слова, которые поисковая система может использовать для улучшения качества результатов поиска.

```html
<META name="keywords" lang="en-us" content="OWASP, security, sunshine, lollipops">
```

Хотя большинство веб-серверов управляют индексацией поисковых систем через файл `robots.txt`, им также могут управлять теги `META`. Тег ниже посоветует роботам не индексировать и не переходить по ссылкам на странице HTML, содержащей тег.

```html
<META name="robots" content="none">
```

[Platform for Internet Content Selection (PICS)](https://www.w3.org/PICS/) а также [Protocol for Web Description Resources (POWDER)](https://www.w3.org/2007/powder/) обеспечить инфраструктуру для связи метаданных с интернет-контентом.

### Идентификация кода JavaScript и сбор файлов JavaScript

Программисты часто жестко кодируют конфиденциальную информацию с переменными JavaScript на интерфейсе. Тестеры должны проверить исходный код HTML и найти код JavaScript между тегами `<script> `и` </ script> `. Тестеры также должны идентифицировать внешние файлы JavaScript для просмотра кода (файлы JavaScript имеют расширение файла `.js` и имя файла JavaScript, обычно помещаемого в атрибут `src` (source) тега `<script> `).

Проверьте код JavaScript на наличие любых конфиденциальных утечек информации, которые могут быть использованы злоумышленниками для дальнейшего злоупотребления или манипулирования системой. Ищите такие значения, как: ключи API, внутренние IP-адреса, конфиденциальные маршруты или учетные данные. Например:

```javascript
const myS3Credentials = {
  accessKeyId: config('AWSS3AccessKeyID'),
  secretAcccessKey: config('AWSS3SecretAccessKey'),
};
```

Тестер может даже найти что-то вроде этого:

```javascript
var conString = "tcp://postgres:1234@localhost/postgres";
```

Когда ключ API найден, тестировщики могут проверить, установлены ли ограничения ключа API для каждой службы или по IP, HTTP-рефереру, приложению, SDK и т. Д.

Например, если тестеры нашли ключ API карты Google, они могут проверить, ограничен ли этот ключ API IP или ограничен только API карты Google. Если ключ API Google ограничен только для API-интерфейсов Google Map, злоумышленники все равно могут использовать этот ключ API для запроса неограниченных API-интерфейсов Google Map, и владелец приложения должен заплатить за это.

```html

<script type="application/json">
...
{"GOOGLE_MAP_API_KEY":"AIzaSyDUEBnKgwiqMNpDplT6ozE4Z0XxuAbqDi4", "RECAPTCHA_KEY":"6LcPscEUiAAAAHOwwM3fGvIx9rsPYUq62uRhGjJ0"}
...
</script>
```

В некоторых случаях тестеры могут находить конфиденциальные маршруты из кода JavaScript, такие как ссылки на внутренние или скрытые страницы администратора.

```html
<script type="application/json">
...
"runtimeConfig":{"BASE_URL_VOUCHER_API":"https://staging-voucher.victim.net/api", "BASE_BACKOFFICE_API":"https://10.10.10.2/api", "ADMIN_PAGE":"/hidden_administrator"}
...
</script>
```

### Идентификация файлов исходной карты

Файлы исходной карты обычно загружаются при открытии DevTools. Тестеры также могут найти файлы исходной карты, добавив расширение «.map» после расширения каждого внешнего файла JavaScript. Например, если тестер видит файл `/static/js/main.chunk.js`, он может проверить его исходный файл карты, посетив `/static/js/main.chunk.js.map`.

#### Black-Box Testing

Проверьте файлы исходной карты на наличие конфиденциальной информации, которая может помочь злоумышленнику получить более полное представление о приложении. Например:

```json
{
  "version": 3,
  "file": "static/js/main.chunk.js",
  "sources": [
    "/home/sysadmin/cashsystem/src/actions/index.js",
    "/home/sysadmin/cashsystem/src/actions/reportAction.js",
    "/home/sysadmin/cashsystem/src/actions/cashoutAction.js",
    "/home/sysadmin/cashsystem/src/actions/userAction.js",
    "..."
  ],
  "..."
}
```

Когда веб-сайты загружают файлы исходной карты, внешний исходный код станет читабельным и его будет легче отлаживать.

## Tools

- [Wget](https://www.gnu.org/software/wget/wget.html)
- Browser "view source" function
- Eyeballs
- [Curl](https://curl.haxx.se/)
- [Burp Suite](https://portswigger.net/burp)
- [Waybackurls](https://github.com/tomnomnom/waybackurls)
- [Google Maps API Scanner](https://github.com/ozguralp/gmapsapiscanner/)

## References

- [KeyHacks](https://github.com/streaak/keyhacks)

### Whitepapers

- [HTML version 4.01](https://www.w3.org/TR/1999/REC-html401-19991224)
- [XHTML](https://www.w3.org/TR/2010/REC-xhtml-basic-20101123/)
- [HTML version 5](https://www.w3.org/TR/html5/)
