# Просмотр метафайлов веб-сервера для утечки информации

|ID          |
|------------|
|WSTG-INFO-03|

## Резюме

В этом разделе описывается, как тестировать различные файлы метаданных на предмет утечки информации из пути (ей) веб-приложения или функциональности. Кроме того, список каталогов, которых следует избегать паукам, роботам или скаулерам, также может быть создан как зависимость [Map execution paths through application](07-Map_Execution_Paths_Through_Application.md). Другая информация также может быть собрана для идентификации поверхности атаки, технологических деталей или для использования в социальной инженерии.

## Цели теста

- Определите скрытые или запутанные пути и функциональность посредством анализа файлов метаданных.
- Извлечение и картирование другой информации, которая может привести к лучшему пониманию рассматриваемых систем.

## Как проверить

> Любое из действий, выполненных ниже с `wget`, также может быть выполнено с `curl`. Многие инструменты динамического тестирования безопасности приложений (DAST), такие как ZAP и Burp Suite, включают проверки или анализ этих ресурсов как часть их функциональности паука / гусеницы. Их также можно идентифицировать, используя различные [Google Dorks](https://en.wikipedia.org/wiki/Google_hacking) или использование расширенных функций поиска, таких как `inurl:`.

### Роботы

Веб-пауки, роботы или скаулеры извлекают веб-страницу, а затем рекурсивно пересекают гиперссылки для получения дополнительного веб-контента. Их принятое поведение определяется [Robots Exclusion Protocol](https://www.robotstxt.org) из [robots.txt](https://www.robotstxt.org/) файл в корневом каталоге.

Например, начало файла `robots.txt` из [Google](https://www.google.com/robots.txt) Выбрана 5 мая 2020 года приведена ниже:

```text
User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
Disallow: /sdch
...
```

The [User-Agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) директива относится к конкретной сети spider/robot/crawler. Например, `User-Agent: Googlebot` относится к пауку из Google в то время как `User-Agent: bingbot` относится к скалеру из Microsoft. `User-Agent: *` в приведенном выше примере относится ко всем [web spiders/robots/crawlers](https://support.google.com/webmasters/answer/6062608?visit_id=637173940975499736-3548411022&rd=1).

`Disallow` директива указывает, какие ресурсы запрещены spiders/robots/crawlers. В приведенном выше примере запрещено следующее:

```text
...
Disallow: /search
...
Disallow: /sdch
...
```

Веб spiders/robots/crawlers can [intentionally ignore](https://blog.isc2.org/isc2_blog/2008/07/the-attack-of-t.html) `Disallow` директивы, указанные в файле `robots.txt`. Следовательно, `robots.txt` не следует рассматривать как механизм для обеспечения соблюдения ограничений на доступ, хранение или повторную публикацию веб-контента третьими лицами.

Файл `robots.txt` извлекается из корневого каталога веб-сервера. Например, чтобы получить `robots.txt` из `www.google.com`, используя `wget` или `curl`:

```bash
$ curl -O -Ss http://www.google.com/robots.txt && head -n5 robots.txt
User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
...
```

#### Анализируйте robots.txt с помощью инструментов Google Webmaster

Владельцы веб-сайтов могут использовать функцию Google "Analyze robots.txt" для анализа веб-сайта как части его [Google Webmaster Tools](https://www.google.com/webmasters/tools). Этот инструмент может помочь с тестированием, и процедура выглядит следующим образом:

1. Войдите в Google Webmaster Tools с учетной записью Google.
2. На панели инструментов введите URL для сайта, который будет проанализирован.
3. Выберите между доступными методами и следуйте инструкциям на экране.

### META Теги

Теги `<META> `расположены в разделе `HEAD` каждого документа HTML и должны быть согласованы на веб-сайте в том случае, если начальная точка robot/spider/crawler не начинается со ссылки на документ, отличной от webroot, т.е. а [deep link](https://en.wikipedia.org/wiki/Deep_linking). Директива роботов также может быть указана с помощью определенного [META tag](https://www.robotstxt.org/meta.html).

#### Роботы МЕТА Tag

Если нет `<META NAME="ROBOTS" ... >` запись тогда «Протокол исключения роботов» по умолчанию «INDEX, FOLLOW» соответственно. Поэтому две другие действительные записи, определенные «Протоколом исключения роботов», имеют префикс «NO ...`т.е. `NOINDEX` и `NOFOLLOW`.

На основании директив (й) Disallow, перечисленных в файле `robots.txt` в webroot, выполняется регулярный поиск по выражению `<META NAME = "ROBOTS" `на каждой веб-странице, и результат сравнивается с `robots .txt` файл в webroot.

#### Разные META Информационные теги

Организации часто встраивают информационные теги META в веб-контент для поддержки различных технологий, таких как программы чтения с экрана, предварительные просмотры в социальных сетях, индексация в поисковых системах и т. Д. Такая метаинформация может иметь значение для тестировщиков при определении используемых технологий и дополнительных путей / функциональности для изучения и тестирования. Следующая метаинформация была получена из `www.whitehouse.gov` через источник страницы просмотра 2020 мая 05:

```html
...
<meta property="og:locale" content="en_US" />
<meta property="og:type" content="website" />
<meta property="og:title" content="The White House" />
<meta property="og:description" content="We, the citizens of America, are now joined in a great national effort to rebuild our country and to restore its promise for all. – President Donald Trump." />
<meta property="og:url" content="https://www.whitehouse.gov/" />
<meta property="og:site_name" content="The White House" />
<meta property="fb:app_id" content="1790466490985150" />
<meta property="og:image" content="https://www.whitehouse.gov/wp-content/uploads/2017/12/wh.gov-share-img_03-1024x538.png" />
<meta property="og:image:secure_url" content="https://www.whitehouse.gov/wp-content/uploads/2017/12/wh.gov-share-img_03-1024x538.png" />
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:description" content="We, the citizens of America, are now joined in a great national effort to rebuild our country and to restore its promise for all. – President Donald Trump." />
<meta name="twitter:title" content="The White House" />
<meta name="twitter:site" content="@whitehouse" />
<meta name="twitter:image" content="https://www.whitehouse.gov/wp-content/uploads/2017/12/wh.gov-share-img_03-1024x538.png" />
<meta name="twitter:creator" content="@whitehouse" />
...
<meta name="apple-mobile-web-app-title" content="The White House">
<meta name="application-name" content="The White House">
<meta name="msapplication-TileColor" content="#0c2644">
<meta name="theme-color" content="#f5f5f5">
...
```

### Sitemaps

Карта сайта - это файл, в котором разработчик или организация могут предоставить информацию о страницах, видео и других файлах, предлагаемых сайтом или приложением, а также об отношениях между ними. Поисковые системы могут использовать этот файл для более разумного изучения вашего сайта. Тестеры могут использовать файлы `sitemap.xml`, чтобы узнать больше о сайте или приложении, чтобы изучить его более полно.

Следующий отрывок взят из основной карты сайта Google, полученной 2020 мая 05.

```bash
$ wget --no-verbose https://www.google.com/sitemap.xml && head -n8 sitemap.xml
2020-05-05 12:23:30 URL:https://www.google.com/sitemap.xml [2049] -> "sitemap.xml" [1]

<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.google.com/schemas/sitemap/0.84">
  <sitemap>
    <loc>https://www.google.com/gmail/sitemap.xml</loc>
  </sitemap>
  <sitemap>
    <loc>https://www.google.com/forms/sitemaps.xml</loc>
  </sitemap>
...
```

Исследуя оттуда, тестер может захотеть получить карту сайта gmail `https://www.google.com/gmail/sitemap.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9" xmlns:xhtml="http://www.w3.org/1999/xhtml">
  <url>
    <loc>https://www.google.com/intl/am/gmail/about/</loc>
    <xhtml:link href="https://www.google.com/gmail/about/" hreflang="x-default" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/el/gmail/about/" hreflang="el" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/it/gmail/about/" hreflang="it" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/ar/gmail/about/" hreflang="ar" rel="alternate"/>
...
```

### Security TXT

`security.txt` является [proposed standard](https://securitytxt.org/) что позволяет веб-сайтам определять политики безопасности и контактные данные. Есть несколько причин, которые могут представлять интерес в сценариях тестирования, включая, но не ограничиваясь:

- Определение дальнейших путей или ресурсов для включения в обнаружение / анализ.
- Сбор информации с открытым исходным кодом.
- Поиск информации о Bug Bounties и т. Д.
- Социальная инженерия.

Файл может присутствовать либо в корне веб-сервера, либо в каталоге `.well-known /`. Ex:

- `https://example.com/security.txt`
- `https://example.com/.well-known/security.txt`

Вот пример реального мира, полученный из LinkedIn 2020, май 05:

```bash
$ wget --no-verbose https://www.linkedin.com/.well-known/security.txt && cat security.txt
2020-05-07 12:56:51 URL:https://www.linkedin.com/.well-known/security.txt [333/333] -> "security.txt" [1]
# Conforms to IETF `draft-foudil-securitytxt-07`
Contact: mailto:security@linkedin.com
Contact: https://www.linkedin.com/help/linkedin/answer/62924
Encryption: https://www.linkedin.com/help/linkedin/answer/79676
Canonical: https://www.linkedin.com/.well-known/security.txt
Policy: https://www.linkedin.com/help/linkedin/answer/62924
```

### Humans TXT

`humans.txt` это инициатива по знанию людей, стоящих за сайтом. Он принимает форму текстового файла, который содержит информацию о разных людях, которые внесли свой вклад в создание сайта. Видеть [humanstxt](http://humanstxt.org/) для получения дополнительной информации. Этот файл часто (хотя и не всегда) содержит информацию для карьерных или рабочих мест / путей.

Следующий пример был получен из Google 2020, май 05:

```bash
$ wget --no-verbose  https://www.google.com/humans.txt && cat humans.txt
2020-05-07 12:57:52 URL:https://www.google.com/humans.txt [286/286] -> "humans.txt" [1]
Google is built by a large team of engineers, designers, researchers, robots, and others in many different sites across the globe. It is updated continuously, and built with more tools and technologies than we can shake a stick at. If you'd like to help us out, see careers.google.com.
```

### Другой .well-known Источники информации

Существуют и другие RFC и интернет-черновики, которые предлагают стандартизированное использование файлов в каталоге `.well-known /`. Списки которых можно найти [here](https://en.wikipedia.org/wiki/List_of_/.well-known/_services_offered_by_webservers) или [here](https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml).

Тестеру было бы довольно просто просмотреть RFC / drafts, создать список, который будет предоставлен скайлеру или фуззеру, чтобы проверить наличие или содержание таких файлов.

## Tools

- Browser (View Source or Dev Tools functionality)
- curl
- wget
- Burp Suite
- ZAP
