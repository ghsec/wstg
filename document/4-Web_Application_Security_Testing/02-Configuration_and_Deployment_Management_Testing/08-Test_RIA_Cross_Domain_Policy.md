# Тестовая политика RIA Cross Domain

| ID |
| ------------- |
| WSTG-CONF-08 |

## Резюме

Rich Internet Applications (RIA) приняли файлы политики Adobe crossdomain.xml, чтобы обеспечить контролируемый междоменный доступ к данным и потреблению услуг с использованием таких технологий, как Oracle Java, Silverlight и Adobe Flash. Следовательно, домен может предоставлять удаленный доступ к своим услугам из другого домена. Однако часто файлы политик, которые описывают ограничения доступа, плохо настроены. Плохая конфигурация файлов политики позволяет проводить межсайтовые атаки и может позволить третьим сторонам получить доступ к конфиденциальным данным, предназначенным для пользователя.

### Что такое файлы политики междоменного пространства?

Файл политики междоменного домена определяет разрешения, которые веб-клиент, такой как Java, Adobe Flash, Adobe Reader и т. Д. использовать для доступа к данным в разных доменах. Для Silverlight Microsoft приняла подмножество Adobe crossdomain.xml и дополнительно создала собственный файл политики кросс-домена: clientaccesspolicy.xml.

Всякий раз, когда веб-клиент обнаруживает, что ресурс должен запрашиваться из другого домена, он сначала ищет файл политики в целевом домене, чтобы определить, разрешено ли выполнение междоменных запросов, включая заголовки и соединения на основе сокетов.

Файлы основных политик находятся в корне домена. Клиенту может быть поручено загрузить другой файл политики, но он всегда сначала проверяет основной файл политики, чтобы убедиться, что основной файл политики разрешает запрашиваемый файл политики.

#### Crossdomain.xml против. Clientaccesspolicy.xml

Большинство приложений RIA поддерживают crossdomain.xml. Однако в случае Silverlight он будет работать только в том случае, если crossdomain.xml указывает, что доступ разрешен из любого домена. Для более детального контроля с Silverlight необходимо использовать clientaccesspolicy.xml.

Файлы политик предоставляют несколько типов разрешений:

- Принимаемые файлы политик (основные файлы политик могут отключать или ограничивать определенные файлы политик)
- Носочки разрешения
- Права заголовка
- разрешения доступа HTTP / HTTPS
- Разрешение доступа на основе криптографических учетных данных

Пример чрезмерно разрешающего файла политики:

```xml
<?xml version="1.0"?>
<!DOCTYPE cross-domain-policy SYSTEM
"http://www.adobe.com/xml/dtds/cross-domain-policy.dtd">
<cross-domain-policy>
    <site-control permitted-cross-domain-policies="all"/>
    <allow-access-from domain="*" secure="false"/>
    <allow-http-request-headers-from domain="*" headers="*" secure="false"/>
</cross-domain-policy>
```

### Как можно злоупотреблять файлами политики междоменов?

- Чрезмерно разрешающая междоменная политика.
- Генерация ответов сервера, которые могут рассматриваться как файлы политики междоменного пространства.
- Использование функции загрузки файлов для загрузки файлов, которые могут рассматриваться как файлы политики междоменного пространства.

### Воздействие злоупотребления междоменным доступом

- Обездействовать защите CSRF.
- Данные чтения ограничены или иным образом защищены политиками перекрестного происхождения.

## Цели теста

- Просмотр и проверка файлов политики.

## Как проверить

### Тестирование на слабые стороны файлов политики RIA

Чтобы проверить слабость файла политики RIA, тестер должен попытаться извлечь файлы политики crossdomain.xml и clientaccesspolicy.xml из корня приложения и из каждой найденной папки.

Например, если URL приложения `http://www.owasp.org`, тестер должен попытаться загрузить файлы `http://www.owasp.org/crossdomain.xml` and `http://www.owasp.org/clientaccesspolicy.xml`.

После извлечения всех файлов политики разрешенные разрешения должны проверяться по принципу наименьших привилегий. Запросы должны поступать только с доменов, портов или протоколов, которые необходимы. Следует избегать чрезмерных разрешительных политик. Политики с `* `в них должны быть внимательно изучены.

#### Example

```xml
<cross-domain-policy>
    <allow-access-from domain="*" />
</cross-domain-policy>
```

##### Result Expected

- A list of policy files found.
- A list of weak settings in the policies.

## Tools

- Nikto
- OWASP Zed Attack Proxy Project
- W3af

## References

- Adobe: ["Cross-domain policy file specification"](http://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html)
- Adobe: ["Cross-domain policy file usage recommendations for Flash Player"](http://www.adobe.com/devnet/flashplayer/articles/cross_domain_policy.html)
- Oracle: ["Cross-Domain XML Support"](http://www.oracle.com/technetwork/java/javase/plugin2-142482.html#CROSSDOMAINXML)
- MSDN: ["Making a Service Available Across Domain Boundaries"](http://msdn.microsoft.com/en-us/library/cc197955(v=vs.95).aspx)
- MSDN: ["Network Security Access Restrictions in Silverlight"](http://msdn.microsoft.com/en-us/library/cc645032(v=vs.95).aspx)
- Stefan Esser: ["Poking new holes with Flash Crossdomain Policy Files"](http://www.hardened-php.net/library/poking_new_holes_with_flash_crossdomain_policy_files.html)
- Jeremiah Grossman: ["Crossdomain.xml Invites Cross-site Mayhem"](http://jeremiahgrossman.blogspot.com/2008/05/crossdomainxml-invites-cross-site.html)
- Google Doctype: ["Introduction to Flash security"](http://code.google.com/p/doctype-mirror/wiki/ArticleFlashSecurity)
- UCSD: [Analyzing the Crossdomain Policies of Flash Applications](http://cseweb.ucsd.edu/~hovav/dist/crossdomain.pdf)
