# Тестирование атрибутов файлов cookie

| ID |
| ------------- |
| WSTG-SESS-02 |

## Резюме

Веб-куки (в данном случае называемые куки-файлами) часто являются ключевым вектором атаки для вредоносных пользователей (обычно нацеливаясь на других пользователей), и приложение всегда должно проявлять должную осмотрительность для защиты куки-файлов.

HTTP - это протокол без гражданства, означающий, что он не содержит ссылок на запросы, отправляемые одним и тем же пользователем. Чтобы исправить эту проблему, сеансы были созданы и добавлены к HTTP-запросам. Браузеры, как обсуждалось в [testing browser storage](../11-Client-side_Testing/12-Testing_Browser_Storage.md), содержать множество механизмов хранения. В этом разделе руководства каждый обсуждается тщательно.

Наиболее используемый механизм хранения сеансов в браузерах - это хранилище файлов cookie. Файлы cookie могут быть установлены сервером, включая a [`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie) заголовок в ответе HTTP или через JavaScript. Файлы cookie можно использовать по множеству причин, таких как:

- управление сессиями
- персонализация
- отслеживание

Чтобы защитить данные cookie, отрасль разработала средства, которые помогут заблокировать эти файлы cookie и ограничить их поверхность атаки. Со временем файлы cookie стали предпочтительным механизмом хранения веб-приложений, поскольку они обеспечивают большую гибкость в использовании и защите.

Средства защиты файлов cookie:

- [Cookie Attributes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Creating_cookies)
- [Cookie Prefixes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Cookie_prefixes)

## Цели теста

- Убедитесь, что для файлов cookie установлена правильная конфигурация безопасности.

## Как проверить

Ниже будет обсуждаться описание каждого атрибута и префикса. Тестер должен подтвердить, что они используются приложением должным образом. Файлы cookie можно просмотреть с помощью [intercepting proxy](#intercepting-proxy), или просмотрев банку с печеньем браузера.

### Атрибуты cookie

#### Безопасный атрибут

[`Secure`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#Secure) Атрибут указывает браузеру отправлять cookie только в том случае, если запрос отправляется по безопасному каналу, такому как `HTTPS`. Это поможет защитить cookie от передачи в незашифрованных запросах. Если доступ к приложению возможен как через `HTTP`, так и `HTTPS`, злоумышленник может перенаправить пользователя на отправку своего файла cookie как части незащищенных запросов.

#### HttpOnly Атрибут

The [`HttpOnly`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#HttpOnly) атрибут используется для предотвращения атак, таких как утечка сеанса, поскольку он не позволяет получить доступ к файлу cookie через сценарий на стороне клиента, такой как JavaScript.

> Это не ограничивает всю поверхность атаки XSS-атак, поскольку злоумышленник все еще может отправлять запрос вместо пользователя, но значительно ограничивает охват векторов XSS-атаки.

#### Атрибут домена

The [`Domain`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Scope_of_cookies) атрибут используется для сравнения домена cookie с доменом сервера, для которого выполняется запрос HTTP. Если домен совпадает или это поддомен, то [`path`](#path-attribute) атрибут будет проверен следующим.

Обратите внимание, что только хосты, принадлежащие указанному домену, могут устанавливать cookie для этого домена. Кроме того, атрибут `domain` не может быть доменом верхнего уровня (например, `.gov` или `.com`), чтобы серверы не устанавливали произвольные файлы cookie для другого домена (например, устанавливая cookie для `owasp.org`). Если атрибут домена не установлен, то имя хоста сервера, который сгенерировал cookie, используется в качестве значения по умолчанию для `domain`.

Например, если файл cookie установлен приложением на `app.mydomain.com` без набора атрибутов домена, то файл cookie будет повторно отправлен для всех последующих запросов для `app.mydomain.com` и его поддоменов (таких как `hacker .app.mydomain.com`), но не для `otherapp.mydomain.com`. Если разработчик хотел ослабить это ограничение, он мог бы установить атрибут `domain` на `mydomain.com`. В этом случае cookie-файлы будут отправляться на все запросы для поддоменов `app.mydomain.com` и `mydomain.com`, таких как `hacker.app.mydomain.com`, и даже `bank.mydomain.com`. Если на поддомене был уязвимый сервер (например, `otherapp.mydomain.com`) и атрибут `domain` был установлен слишком свободно (например, `mydomain.com`), то уязвимый сервер может быть используется собирать куки (например, токены сеанса) по всей области `mydomain.com`.

#### Атрибут пути

The [`Path`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Scope_of_cookies) Атрибут играет важную роль в настройке области действия файлов cookie в сочетании с [`domain`] (# domain-attribute). В дополнение к домену можно указать путь URL, для которого действителен файл cookie. Если домен и путь совпадают, файл cookie будет отправлен в запросе. Как и в случае с атрибутом домена, если атрибут пути установлен слишком свободно, он может сделать приложение уязвимым для атак других приложений на том же сервере. Например, если атрибут пути был установлен на корень веб-сервера `/`, то файлы cookie приложения будут отправляться в каждое приложение в пределах одного домена (если несколько приложений находятся под одним и тем же сервером). Пара примеров для нескольких приложений под одним сервером:

- `path=/bank`
- `path=/private`
- `path=/docs`
- `path=/docs/admin`

#### Expires Attribute

The [`Expires`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Permanent_cookies) атрибут используется для:

- установить постоянные куки
- ограничить продолжительность жизни, если сеанс живет слишком долго
- удалите cookie принудительно, установив его на дату прошлого

В отличие от [session cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Session_cookies), постоянные файлы cookie будут использоваться браузером до истечения срока действия файла cookie. Как только срок годности превысит установленное время, браузер удалит cookie.

#### Атрибут SameSite

The [`SameSite`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#SameSite_cookies) атрибут используется для утверждения, что cookie не следует отправлять вместе с запросами между сайтами. Эта функция позволяет серверу снизить риск утечки информации через границу. В некоторых случаях он также используется в качестве стратегии снижения риска (или механизма глубокой защиты) для предотвращения [cross-site request forgery](05-Testing_for_Cross_Site_Request_Forgery.md) атаки. Этот атрибут можно настроить в трех разных режимах:

- `Strict`
- `Lax`
- `None`

##### Strict Value

The `Strict` Значение является наиболее ограничивающим использованием `SameSite`, позволяя браузеру отправлять cookie только в сторонний контекст без навигации верхнего уровня. Другими словами, данные, связанные с cookie, будут отправляться только по запросам, соответствующим текущему сайту, указанному на панели URL браузера. Файл cookie не будет отправляться по запросам, сгенерированным сторонними веб-сайтами. Это значение особенно рекомендуется для действий, выполняемых в том же домене. Тем не менее, он может иметь некоторые ограничения с некоторыми системами управления сеансами, негативно влияющими на пользовательскую навигацию. Поскольку браузер не будет отправлять cookie-файлы по любым запросам, сгенерированным из стороннего домена или электронной почты, пользователь должен будет войти снова, даже если у него уже есть аутентифицированный сеанс.

##### Lax Value

The `Lax` значение менее ограничительно, чем `Strict`. Файл cookie будет отправлен, если URL равен домену файла cookie (первым сторонам), даже если ссылка идет от стороннего домена. Это значение считается большинством браузеров поведением по умолчанию, поскольку оно обеспечивает лучший пользовательский интерфейс, чем значение `Strict`. Это не запускает для активов, таких как изображения, где куки могут не понадобиться для доступа к ним.

##### None Value

The `None` Значение указывает, что браузер будет отправлять cookie-файлы по межсайтовым запросам (нормальное поведение перед реализацией `SamseSite`), только если также используется атрибут `Secure`, _e.g._ `SameSite = None; Secure`. Это рекомендуемое значение, вместо того, чтобы не указывать какое-либо значение `SameSite`, поскольку оно заставляет использовать [`secure` attribute](#secure-attribute).

### Префиксы печенья

По своей конструкции файлы cookie не имеют возможности гарантировать целостность и конфиденциальность информации, хранящейся в них. Эти ограничения не позволяют серверу быть уверенным в том, как были установлены атрибуты данного файла cookie при создании. Чтобы предоставить серверам такие функции обратно совместимым образом, отрасль представила концепцию [`Cookie Name Prefixes`](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-prefixes-00) облегчить передачу таких данных, встроенных как часть имени файла cookie.

#### Префикс хоста

The [`__Host-`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Cookie_prefixes) префикс ожидает, что куки будут выполнять следующие условия:

  1. Файл cookie должен быть установлен с [`Secure` attribute](#secure-attribute).
  2. Файл cookie должен быть установлен из URI, который пользовательский агент считает безопасным.
  3. Отправляется только хосту, который установил cookie и НЕ ДОЛЖЕН включать его [`Domain` attribute](#domain-attribute).
  4. Файл cookie должен быть установлен с [`Path`attribute](#path-attribute) со значением `/`, поэтому он будет отправляться на каждый запрос на хост.

По этой причине печенье `Set-Cookie: __Host-SID=12345; Secure; Path=/` будет принято, в то время как любой из следующих всегда будет отклонен:
`Set-Cookie: __Host-SID=12345`
`Set-Cookie: __Host-SID=12345; Secure`
`Set-Cookie: __Host-SID=12345; Domain=site.example`
`Set-Cookie: __Host-SID=12345; Domain=site.example; Path=/`
`Set-Cookie: __Host-SID=12345; Secure; Domain=site.example; Path=/`

#### Secure Prefix

The [`__Secure-`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Cookie_prefixes) префикс менее ограничительный и может быть введен путем добавления строки `__Secure-`, чувствительной к регистру, к имени cookie. Ожидается, что любой файл cookie, который соответствует префиксу `__Secure-`, будет выполнять следующие условия:

  1. Файл cookie должен быть установлен с атрибутом `Secure`.
  2. Файл cookie должен быть установлен из URI, который пользовательский агент считает безопасным.

### Сильные практики

Исходя из потребностей приложения и того, как должен функционировать cookie, должны применяться атрибуты и префиксы. Чем больше печенье заблокировано, тем лучше.

Собрав все это вместе, мы можем определить наиболее безопасную конфигурацию атрибута cookie как: `Set-Cookie: __Host-SID=<session token>; path=/; Secure; HttpOnly; SameSite=Strict`.

## Tools

### Intercepting Proxy

- [OWASP Zed Attack Proxy Project](https://www.zaproxy.org)
- [Web Proxy Burp Suite](https://portswigger.net)

### Browser Plug-in

- [Tamper Data for FF Quantum](https://addons.mozilla.org/en-US/firefox/addon/tamper-data-for-ff-quantum/)
- ["FireSheep" for FireFox](https://github.com/codebutler/firesheep)
- ["EditThisCookie" for Chrome](https://chrome.google.com/webstore/detail/editthiscookie/fngmhnnpilhplaeedifhccceomclgfbg?hl=en)
- ["Cookiebro - Cookie Manager" for FireFox](https://addons.mozilla.org/en-US/firefox/addon/cookiebro/)

## References

- [RFC 2965 - HTTP State Management Mechanism](https://tools.ietf.org/html/rfc2965)
- [RFC 2616 – Hypertext Transfer Protocol – HTTP 1.1](https://tools.ietf.org/html/rfc2616)
- [Same-Site Cookies - draft-ietf-httpbis-cookie-same-site-00](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-same-site-00)
- [The important "expires" attribute of Set-Cookie](https://seckb.yehg.net/2012/02/important-expires-attribute-of-set.html)
- [HttpOnly Session ID in URL and Page Body](https://seckb.yehg.net/2012/06/httponly-session-id-in-url-and-page.html)
