# Проверьте методы HTTP

|ID          |
|------------|
|WSTG-CONF-06|

HTTP предлагает ряд методов, которые можно использовать для выполнения действий на веб-сервере (стандарт HTTP 1.1 называет их «методами», но они также обычно описываются как «глаголы»). Хотя GET и POST являются наиболее распространенными методами, которые используются для доступа к информации, предоставляемой веб-сервером, HTTP допускает несколько других (и несколько менее известных) методов. Некоторые из них могут быть использованы в гнусных целях, если веб-сервер неправильно настроен.

[RFC 7231 –  Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content](https://tools.ietf.org/html/rfc7231) определяет следующие действительные методы HTTP-запроса или глаголы:

- [`GET`](https://tools.ietf.org/html/rfc7231#section-4.3.1)
- [`HEAD`](https://tools.ietf.org/html/rfc7231#section-4.3.2)
- [`POST`](https://tools.ietf.org/html/rfc7231#section-4.3.3)
- [`PUT`](https://tools.ietf.org/html/rfc7231#section-4.3.4)
- [`DELETE`](https://tools.ietf.org/html/rfc7231#section-4.3.5)
- [`CONNECT`](https://tools.ietf.org/html/rfc7231#section-4.3.6)
- [`OPTIONS`](https://tools.ietf.org/html/rfc7231#section-4.3.7)
- [`TRACE`](https://tools.ietf.org/html/rfc7231#section-4.3.8)

Однако большинству веб-приложений требуется только отвечать на запросы GET и POST, получая пользовательские данные в строке запроса URL или добавляя их соответственно к запросу. Стандартные ссылки стиля `<a href = ""> </ a> `, а также формы, определенные без метода, запускают запрос GET; данные формы, представленные через `<form method = 'POST'> </ form>`, запускают POST запросы. Звонки JavaScript и AJAX могут отправлять методы, отличные от GET и POST, но обычно это не нужно. Поскольку другие методы используются так редко, многие разработчики не знают или не принимают во внимание, как реализация этих методов на веб-сервере или платформе приложения влияет на функции безопасности приложения.

## Цели теста

- Перечислите поддерживаемые методы HTTP.
- Тест на обход контроля доступа.
- Проверьте уязвимости XST.
- Проверьте методы переопределения метода HTTP.

## Как проверить

### Откройте для себя поддерживаемые методы

Чтобы выполнить этот тест, тестеру нужен какой-то способ выяснить, какие методы HTTP поддерживаются проверяемым веб-сервером. Хотя метод HTTP `OPTIONS` обеспечивает прямой способ сделать это, проверьте ответ сервера, выдав запросы разными методами. Это может быть достигнуто путем ручного тестирования или чего-то вроде [`http-methods`](https://nmap.org/nsedoc/scripts/http-methods.html) Nmap скрипт.

Чтобы использовать сценарий Nmap `http-methods` для тестирования конечной точки `/index.php` на сервере `localhost` с помощью HTTPS, выполните команду :

```bash
nmap -p 443 --script http-methods --script-args http-methods.url-path='/index.php' localhost
```

При тестировании приложения, которое должно принимать другие методы, например,. RESTful Web Service, тщательно протестируйте его, чтобы убедиться, что все конечные точки принимают только те методы, которые им требуются.

#### Тестирование метода PUT

1. Захватите базовый запрос цели с помощью веб-прокси.
2. Измените метод запроса на `PUT` и добавьте файл `test.html` и отправьте запрос на сервер приложений.

   ```html
   PUT /test.html HTTP/1.1
   Host: testing-website

   <html>
   HTTP PUT Method is Enabled
   </html>
   ```

3. Если ответ сервера с кодами успеха 2XX или повторными инструкциями 3XX, а затем подтвердите с помощью запроса `GET` для файла `test.html`. Приложение уязвимо.

Если метод HTTP `PUT` не разрешен для базового URL или запроса, попробуйте другие пути в системе.

> ПРИМЕЧАНИЕ. Если вам удалось загрузить веб-оболочку, вам следует перезаписать ее или убедиться, что команда безопасности цели осведомлена и удалила компонент сразу после проверки концепции.

Используя метод `PUT`, злоумышленник может поместить произвольный и потенциально вредоносный контент в систему, что может привести к удаленному выполнению кода, порче сайта или отказу в обслуживании.

### Тестирование для обхода контроля доступа

Найдите страницу для посещения, которая имеет ограничение безопасности, так что запрос GET обычно вызывает перенаправление 302 на страницу входа в систему или вызывает прямой вход в систему. Запросы на выдачу с использованием различных методов, таких как HEAD, POST, PUT и т. Д. а также произвольно составленные методы, такие как BILBAO, FOOBAR, CATS и т. д. Если веб-приложение отвечает «HTTP / 1.1 200 OK», который не является страницей входа, может быть возможно обойти аутентификацию или авторизацию. Следующий пример использует [Nmap's `ncat`](https://nmap.org/ncat/).

```bash
$ ncat www.example.com 80
HEAD /admin HTTP/1.1
Host: www.example.com

HTTP/1.1 200 OK
Date: Mon, 18 Aug 2008 22:44:11 GMT
Server: Apache
Set-Cookie: PHPSESSID=pKi...; path=/; HttpOnly
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
Set-Cookie: adminOnlyCookie1=...; expires=Tue, 18-Aug-2009 22:44:31 GMT; domain=www.example.com
Set-Cookie: adminOnlyCookie2=...; expires=Mon, 18-Aug-2008 22:54:31 GMT; domain=www.example.com
Set-Cookie: adminOnlyCookie3=...; expires=Sun, 19-Aug-2007 22:44:30 GMT; domain=www.example.com
Content-Language: EN
Connection: close
Content-Type: text/html; charset=ISO-8859-1
```

Если система кажется уязвимой, выдайте CSRF-подобные атаки, такие как следующие, чтобы более полно использовать проблему:

- `HEAD /admin/createUser.php?member=myAdmin`
- `PUT /admin/changePw.php?member=myAdmin&passwd=foo123&confirm=foo123`
- `CATS /admin/groupEdit.php?group=Admins&member=myAdmin&action=add`

Используя вышеупомянутые три команды, измененные в соответствии с требованиями тестируемого приложения и тестирования, будет создан новый пользователь, назначен пароль, и пользователь сделает администратора, все с использованием слепой отправки запроса.

### Тестирование на потенциал межсайтового отслеживания

Примечание: чтобы понять логику и цели атаки межсайтовой трассировки (XST), нужно быть знакомым с ней [cross-site scripting attacks](https://owasp.org/www-community/attacks/xss/).

Метод `TRACE`, предназначенный для тестирования и отладки, инструктирует веб-сервер отражать полученное сообщение обратно клиенту. Этот метод, хотя и явно безвредный, может быть успешно использован в некоторых сценариях для кражи учетных данных законных пользователей. Эта техника атаки была открыта Иеремией Гроссманом в 2003 году в попытке обойти [HttpOnly](https://owasp.org/www-community/HttpOnly) атрибут, который направлен на защиту файлов cookie от доступа к JavaScript.  Однако метод TRACE можно использовать для обхода этой защиты и доступа к cookie, даже если этот атрибут установлен.

Тест на потенциал отслеживания между площадками путем выдачи запроса, такого как:

```bash
$ ncat www.victim.com 80
TRACE / HTTP/1.1
Host: www.victim.com
Random: Header

HTTP/1.1 200 OK
Random: Header
...
```

Веб-сервер вернул 200 и отразил случайный заголовок, который был установлен на месте. Для дальнейшего использования этой проблемы:

```bash
$ ncat www.victim.com 80
TRACE / HTTP/1.1
Host: www.victim.com
Attack: <script>prompt()</script>
```

Приведенный выше пример работает, если ответ отражается в контексте HTML.

В старых браузерах атаки были предприняты с помощью [XHR](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) технология, которая просочилась в заголовки, когда сервер их отражает (*e.g.* Cookies, Authorization tokens, etc.) и обошел меры безопасности, такие как [HttpOnly](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md#httponly-attribute) атрибут. Эту атаку можно выполнить в последних браузерах, только если приложение интегрируется с технологиями, похожими на Flash.

### Тестирование по методу HTTP

Некоторые веб-фреймворки предоставляют способ переопределить фактический метод HTTP в запросе, эмулируя отсутствующие глаголы HTTP, передавая некоторый пользовательский заголовок в запросах. Основная цель этого - обойти некоторое промежуточное ПО (например,. ограничение прокси, брандмауэра), когда разрешенные методы обычно не охватывают глаголы, такие как `PUT` или `DELETE`. Для такого туннелирования глаголов можно использовать следующие альтернативные заголовки:

- `X-HTTP-Method`
- `X-HTTP-Method-Override`
- `X-Method-Override`

Чтобы проверить это, в сценариях, где ограниченные глаголы, такие как PUT или DELETE, возвращают «Метод 405 не разрешен», повторите тот же запрос с добавлением альтернативных заголовков для переопределения метода HTTP и наблюдайте, как система реагирует. Приложение должно отвечать другим кодом состояния (* например,.* 200) в случаях, когда поддерживается переопределение метода.

Веб-сервер в следующем примере не допускает метод `DELETE` и блокирует его:

```bash
$ ncat www.example.com 80
DELETE /resource.html HTTP/1.1
Host: www.example.com

HTTP/1.1 405 Method Not Allowed
Date: Sat, 04 Apr 2020 18:26:53 GMT
Server: Apache
Allow: GET,HEAD,POST,OPTIONS
Content-Length: 320
Content-Type: text/html; charset=iso-8859-1
Vary: Accept-Encoding
```

После добавления заголовка `X-HTTP-Method` сервер отвечает на запрос 200 :

```bash
$ ncat www.example.com 80
DELETE /resource.html HTTP/1.1
Host: www.example.com
X-HTTP-Method: DELETE

HTTP/1.1 200 OK
Date: Sat, 04 Apr 2020 19:26:01 GMT
Server: Apache
```

## Восстановление

- Убедитесь, что разрешены только необходимые заголовки и что разрешенные заголовки правильно настроены.
- Убедитесь, что не реализованы обходные пути для обхода мер безопасности, применяемых пользовательскими агентами, фреймворками или веб-серверами.

## Tools

- [Ncat](https://nmap.org/ncat/)
- [cURL](https://curl.haxx.se/)
- [nmap http-methods NSE script](https://nmap.org/nsedoc/scripts/http-methods.html)
- [w3af plugin htaccess_methods](http://w3af.org/plugins/audit/htaccess_methods)

## References

- [RFC 2109](https://tools.ietf.org/html/rfc2109) and [RFC 2965](https://tools.ietf.org/html/rfc2965): "HTTP State Management Mechanism"
- [HTACCESS: BILBAO Method Exposed](https://web.archive.org/web/20160616172703/http://www.kernelpanik.org/docs/kernelpanik/bme.eng.pdf)
- [Amit Klein: "XS(T) attack variants which can, in some cases, eliminate the need for TRACE"](https://www.securityfocus.com/archive/107/308433)
- [Fortify - Misused HTTP Method Override](https://vulncat.fortify.com/en/detail?id=desc.dynamic.xtended_preview.often_misused_http_method_override)
- [CAPEC-107: Cross Site Tracing](https://capec.mitre.org/data/definitions/107.html)
