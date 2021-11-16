# Определите точки входа приложения

|ID          |
|------------|
|WSTG-INFO-06|

## Резюме

Перечисление приложения и его поверхности атаки является ключевым предшественником, прежде чем можно будет провести какое-либо тщательное тестирование, поскольку оно позволяет тестеру определять вероятные области слабости. Этот раздел призван помочь идентифицировать и наметить области в приложении, которые должны быть исследованы после завершения подсчета и картирования.

## Цели теста

- Определите возможные точки входа и инъекции с помощью анализа запросов и ответов.

## Как проверить

Перед началом любого тестирования тестер всегда должен хорошо понимать приложение и то, как пользователь и браузер общаются с ним. Когда тестер проходит через приложение, ему следует обратить внимание на все HTTP-запросы, а также на все параметры и поля формы, которые передаются в приложение. Они должны уделять особое внимание тому, когда используются запросы GET и когда запросы POST используются для передачи параметров в приложение. Кроме того, им также необходимо обратить внимание на то, когда используются другие методы для услуг RESTful.

Обратите внимание, что для просмотра параметров, отправленных в теле запросов, таких как запрос POST, тестер может захотеть использовать такой инструмент, как перехватчик прокси (см [tools](#tools)). В запросе POST, тестер также должен особо отмечать любые скрытые поля формы, которые передаются в приложение, поскольку они обычно содержат конфиденциальную информацию, такие как государственная информация, количество предметов, цена предметов, что разработчик никогда не собирался никого видеть или менять.

По опыту автора, было очень полезно использовать перехватчик и электронную таблицу для этого этапа тестирования. Прокси-сервер будет отслеживать каждый запрос и ответ между тестером и приложением при его изучении. Кроме того, в этот момент тестировщики обычно задерживают каждый запрос и ответ, чтобы они могли видеть точно каждый заголовок, параметр и т. Д. это передается в приложение и что возвращается. Иногда это может быть довольно утомительно, особенно на крупных интерактивных сайтах (вспомните банковское приложение). Тем не менее, опыт покажет, что искать, и этот этап может быть значительно сокращен.

Когда тестер проходит через приложение, он должен принять к сведению любые интересные параметры в URL, пользовательских заголовках или теле запросов / ответов и сохранить их в электронной таблице. Электронная таблица должна включать запрашиваемую страницу (может быть также полезно добавить номер запроса из прокси, для дальнейшего использования) интересные параметры, тип запроса (ПОЛУЧИТЬ, ПОСТ, так далее) если доступ аутентифицирован / не аутентифицирован, если используется TLS, если это часть многоэтапного процесса, если используются WebSockets, и любые другие соответствующие примечания. После того, как они наметили все области приложения, они могут пройти через приложение и протестировать каждую из областей, которые они определили, и сделать заметки о том, что сработало, а что не сработало. В оставшейся части этого руководства будет указано, как проверить каждую из этих областей, представляющих интерес, но этот раздел должен быть предпринят до того, как может начаться любое фактическое тестирование.

Ниже приведены некоторые пункты интересов для всех запросов и ответов. В разделе запросов сфокусируйтесь на методах GET и POST, так как они появляются в большинстве запросов. Обратите внимание, что могут использоваться другие методы, такие как PUT и DELETE. Часто эти более редкие запросы, если они разрешены, могут выявить уязвимости. В этом руководстве есть специальный раздел, посвященный тестированию этих методов HTTP.

### Requests

- Определите, где используются GET и где используются POST.
- Определите все параметры, используемые в запросе POST (они находятся в теле запроса).
- В запросе POST обратите особое внимание на любые скрытые параметры. Когда POST отправляется, все поля формы (включая скрытые параметры) будут отправлены в тело сообщения HTTP в приложение. Обычно они не видны, если не используется прокси-сервер или исходный код HTML. Кроме того, показана следующая страница, ее данные и уровень доступа могут различаться в зависимости от значения скрытого параметра (параметров).
- Определить все параметры, используемые в запросе GET (т.е., URL), в частности строка запроса (обычно после a ? знак).
- Определите все параметры строки запроса. Они обычно в формате пары, например `foo = bar`. Также обратите внимание, что многие параметры могут находиться в одной строке запроса, например, разделенной символом `&`, `\ ~`, `: `, или любым другим специальным символом или кодировкой.
- Особое примечание, когда речь идет об определении нескольких параметров в одной строке или в запросе POST, заключается в том, что для выполнения атак потребуется некоторые или все параметры. Тестер должен идентифицировать все параметры (даже если они закодированы или зашифрованы) и определить, какие из них обрабатываются приложением. Более поздние разделы руководства определят, как проверить эти параметры. На этом этапе просто убедитесь, что каждый из них идентифицирован.
- Также обратите внимание на любые дополнительные или пользовательские заголовки типа, которые обычно не видны (например, `debug: false`).

### Responses

- Определите, где установлены новые файлы cookie (заголовок «Set-Cookie»), изменен или добавлен.
- Определите, где есть какие-либо перенаправления (3xx HTTP-код состояния), 400 кодов состояния, в частности 403 Запрещено, и 500 внутренних ошибок сервера во время обычных ответов (т.е., неизмененные запросы).
- Также обратите внимание, где используются интересные заголовки. Например, «Сервер: BIG-IP» указывает, что сайт сбалансирован по нагрузке. Таким образом, если сайт сбалансирован по нагрузке и один сервер неправильно настроен, то тестер может сделать несколько запросов на доступ к уязвимому серверу в зависимости от типа используемой балансировки нагрузки.

### OWASP Attack Surface Detector

Инструмент «Детектор поверхности атаки» (ASD) исследует исходный код и раскрывает конечные точки веб-приложения, параметры, которые принимают эти конечные точки, и тип данных этих параметров. Это включает в себя несвязанные конечные точки, которые паук не сможет найти, или необязательные параметры, полностью неиспользованные в клиентском коде. Он также имеет возможность рассчитать изменения поверхности атаки между двумя версиями приложения.

Детектор поверхности атаки доступен в виде плагина для ZAP и Burp Suite, а также доступен инструмент командной строки. Инструмент командной строки экспортирует поверхность атаки как выход JSON, который затем может использоваться плагинами ZAP и Burp Suite. Это полезно в тех случаях, когда исходный код не предоставляется непосредственно тестеру проникновения. Например, тестер проникновения может получить выходной файл json от клиента, который не хочет предоставлять сам исходный код.

#### Как использовать

Файл баночки CLI доступен для скачивания [https://github.com/secdec/attack-surface-detector-cli/releases](https://github.com/secdec/attack-surface-detector-cli/releases).

Вы можете запустить следующую команду для ASD, чтобы идентифицировать конечные точки из исходного кода целевого веб-приложения.

`java -jar attack-surface-detector-cli-1.3.5.jar <source-code-path> [flags]`

Вот пример запуска команды против [OWASP RailsGoat](https://github.com/OWASP/railsgoat).

```text
$ java -jar attack-surface-detector-cli-1.3.5.jar railsgoat/
Beginning endpoint detection for '<...>/railsgoat' with 1 framework types
Using framework=RAILS
[0] GET: /login (0 variants): PARAMETERS={url=name=url, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_contro
ller.rb (lines '6'-'9')
[1] GET: /logout (0 variants): PARAMETERS={}; FILE=/app/controllers/sessions_controller.rb (lines '33'-'37')
[2] POST: /forgot_password (0 variants): PARAMETERS={email=name=email, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/
password_resets_controller.rb (lines '29'-'38')
[3] GET: /password_resets (0 variants): PARAMETERS={token=name=token, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/p
assword_resets_controller.rb (lines '19'-'27')
[4] POST: /password_resets (0 variants): PARAMETERS={password=name=password, paramType=QUERY_STRING, dataType=STRING, user=name=user, paramType=QUERY_STRING, dataType=STRING, confirm_password=name=confirm_password, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/password_resets_controller.rb (lines '5'-'17')
[5] GET: /sessions/new (0 variants): PARAMETERS={url=name=url, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_controller.rb (lines '6'-'9')
[6] POST: /sessions (0 variants): PARAMETERS={password=name=password, paramType=QUERY_STRING, dataType=STRING, user_id=name=user_id, paramType=SESSION, dataType=STRING, remember_me=name=remember_me, paramType=QUERY_STRING, dataType=STRING, url=name=url, paramType=QUERY_STRING, dataType=STRING, email=name=email, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_controller.rb (lines '11'-'31')
[7] DELETE: /sessions/{id} (0 variants): PARAMETERS={}; FILE=/app/controllers/sessions_controller.rb (lines '33'-'37')
[8] GET: /users (0 variants): PARAMETERS={}; FILE=/app/controllers/api/v1/users_controller.rb (lines '9'-'11')
[9] GET: /users/{id} (0 variants): PARAMETERS={}; FILE=/app/controllers/api/v1/users_controller.rb (lines '13'-'15')
... snipped ...
[38] GET: /api/v1/mobile/{id} (0 variants): PARAMETERS={id=name=id, paramType=QUERY_STRING, dataType=STRING, class=name=class, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/api/v1/mobile_controller.rb (lines '8'-'13')
[39] GET: / (0 variants): PARAMETERS={url=name=url, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_controller.rb (lines '6'-'9')
Generated 40 distinct endpoints with 0 variants for a total of 40 endpoints
Successfully validated serialization for these endpoints
0 endpoints were missing code start line
0 endpoints were missing code end line
0 endpoints had the same code start and end line
Generated 36 distinct parameters
Generated 36 total parameters
- 36/36 have their data type
- 0/36 have a list of accepted values
- 36/36 have their parameter type
--- QUERY_STRING: 35
--- SESSION: 1
Finished endpoint detection for '<...>/railsgoat'
----------
-- DONE --
0 projects had duplicate endpoints
Generated 40 distinct endpoints
Generated 40 total endpoints
Generated 36 distinct parameters
Generated 36 total parameters
1/1 projects had endpoints generated
To enable logging include the -debug argument
```

Вы также можете создать выходной файл JSON, используя флаг `-json`, который может использоваться плагином как для ZAP, так и для Burp Suite. Смотрите следующие ссылки для более подробной информации.

- [Home of ASD Plugin for OWASP ZAP](https://github.com/secdec/attack-surface-detector-zap/wiki)
- [Home of ASD Plugin for PortSwigger Burp](https://github.com/secdec/attack-surface-detector-burp/wiki)

### Тестирование на точки входа приложения

Ниже приведены два примера того, как проверить точки входа в приложение.

#### Example 1

В этом примере показан запрос GET, который позволит приобрести товар в онлайн-заявке на покупки.

```http
GET /shoppingApp/buyme.asp?CUSTOMERID=100&ITEM=z101a&PRICE=62.50&IP=x.x.x.x HTTP/1.1
Host: x.x.x.x
Cookie: SESSIONID=Z29vZCBqb2IgcGFkYXdhIG15IHVzZXJuYW1lIGlzIGZvbyBhbmQgcGFzc3dvcmQgaXMgYmFy
```

> Все параметры запроса, такие как CUSTOMERID, ITEM, PRICE, IP и Cookie, которые могут быть просто закодированы параметрами или параметрами, используемыми для состояния сеанса.

#### Example 2

В этом примере показан запрос POST, который войдет в приложение.

```http
POST /example/authenticate.asp?service=login HTTP/1.1
Host: x.x.x.x
Cookie: SESSIONID=dGhpcyBpcyBhIGJhZCBhcHAgdGhhdCBzZXRzIHByZWRpY3RhYmxlIGNvb2tpZXMgYW5kIG1pbmUgaXMgMTIzNA==;CustomCookie=00my00trusted00ip00is00x.x.x.x00

user=admin&pass=pass123&debug=true&fromtrustIP=true
```

Можно отметить, что параметры отправляются в нескольких местах:

1. В строке запроса: `service`
2. В заголовке Cookie: `SESSIONID`, `CustomCookie`
3. В теле запроса: `user`, `pass`, `debug`, `fromtrustIP`

Наличие различных мест инъекции предоставляет злоумышленнику возможности цепочки, которые могут повысить вероятность обнаружения ошибки в коде обработки.

## Tools

- [OWASP Zed Attack Proxy (ZAP)](https://www.zaproxy.org/)
- [Burp Suite](https://www.portswigger.net/burp/)
- [Fiddler](https://www.telerik.com/fiddler)

## References

- [RFC 2616 – Hypertext Transfer Protocol – HTTP 1.1](https://tools.ietf.org/html/rfc2616)
- [OWASP Attack Surface Detector](https://owasp.org/www-project-attack-surface-detector/)
