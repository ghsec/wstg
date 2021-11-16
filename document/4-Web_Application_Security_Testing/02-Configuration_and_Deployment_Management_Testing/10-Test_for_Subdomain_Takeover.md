# Тест на поглощение субдомена

| ID |
| ------------- |
| WSTG-CONF-10 |

## Резюме

Успешная эксплуатация такого рода уязвимости позволяет противнику требовать и контролировать субдомен жертвы. Эта атака опирается на следующее:

1. Внешняя запись поддомена DNS-сервера жертвы настроена так, чтобы указывать на несуществующий или неактивный ресурс / внешнюю службу / конечную точку. Распространение продуктов XaaS (Anything as a Service) и общедоступных облачных сервисов предлагает множество потенциальных целей для рассмотрения.
2. Поставщик услуг, размещающий ресурс / внешнюю службу / конечную точку, не обрабатывает проверку владения субдоменом должным образом.

Если поглощение поддомена является успешным, возможны самые разные атаки (обслуживание вредоносного контента, физинг, кража файлов cookie сеанса пользователя, учетных данных и т. Д.).). Эта уязвимость может быть использована для самых разных записей ресурсов DNS, включая: `A`, `CNAME`, `MX`, `NS`, `TXT` и т. Д. С точки зрения серьезности атаки поглощение субдомена «NS» (хотя и менее вероятно) оказывает наибольшее влияние, поскольку успешная атака может привести к полному контролю всей зоны DNS и домена жертвы.

### GitHub

1. Жертва (victim.com) использует GitHub для разработки и настроила запись DNS (`coderepo.victim.com`) для доступа к ней.
2. Жертва решает перенести свой репозиторий кода из GitHub на коммерческую платформу и не удаляет `coderepo.victim.com` со своего DNS-сервера.
3. Противник узнает, что `coderepo.victim.com` размещен на GitHub и использует GitHub Pages для получения `coderepo.victim.com` с помощью своей учетной записи GitHub.

### Истекший домен

1. Жертва (victim.com) владеет другим доменом (victimotherdomain.com) и использует запись CNAME (www) для ссылки на другой домен (`www.victim.com` -> `victimotherdomain.com`)
2. В какой-то момент срок действия victivotherdomain.com истекает и доступен для регистрации любому. Поскольку запись CNAME не удаляется из зоны DNS viction.com, любой, кто регистрирует `victimotherdomain.com`, имеет полный контроль над `www.victim.com` до появления записи DNS.

## Цели теста

- Перечислите все возможные домены (предыдущие и текущие).
- Определите забытые или неправильно настроенные домены.

## Как проверить

### Тестирование черного ящика

Первым шагом является подсчет DNS-серверов жертвы и записей ресурсов. Существует несколько способов выполнения этой задачи, например, подсчет DNS с использованием списка общих словарей поддоменов, грубой силы DNS или с помощью поисковых систем в Интернете и других источников данных OSINT.

Используя команду dig, тестер ищет следующие сообщения ответа DNS-сервера, которые требуют дальнейшего расследования:

- `NXDOMAIN`
- `SERVFAIL`
- `REFUSED`
- `no servers could be reached.`

#### Тестирование DNS A, CNAME Record Subdomain Takeover

Выполните базовую инвентаризацию DNS в домене жертвы (`victim.com`), используя `dnsrecon` :

```bash
$ ./dnsrecon.py -d victim.com
[*] Performing General Enumeration of Domain: victim.com
...
[-] DNSSEC is not configured for victim.com
[*]      A subdomain.victim.com 192.30.252.153
[*]      CNAME subdomain1.victim.com fictioussubdomain.victim.com
...
```

Определите, какие записи ресурсов DNS мертвы, и укажите на неактивные / неиспользуемые сервисы. Использование команды dig для записи `CNAME`:

```bash
$ dig CNAME fictioussubdomain.victim.com
; <<>> DiG 9.10.3-P4-Ubuntu <<>> ns victim.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 42950
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
```

Следующие ответы DNS требуют дальнейшего расследования: `NXDOMAIN`.

Чтобы протестировать запись `A`, тестер выполняет поиск базы данных whois и идентифицирует GitHub как поставщика услуг :

```bash
$ whois 192.30.252.153 | grep "OrgName"
OrgName: GitHub, Inc.
```

Тестер посещает `subdomain.victim.com` или выдает запрос HTTP GET, который возвращает ответ «404 - файл не найден», который является четким признаком уязвимости.

![GitHub 404 File Not Found response](images/subdomain_takeover_ex1.jpeg)\
*Figure 4.2.10-1: GitHub 404 File Not Found response*

The tester claims the domain using GitHub Pages:

![GitHub claim domain](images/subdomain_takeover_ex2.jpeg)\
*Figure 4.2.10-2: GitHub claim domain*

#### Тестирование поглощения субдомена NS Record

Определите все серверы имен для домена в области:

```bash
$ dig ns victim.com +short
ns1.victim.com
nameserver.expireddomain.com
```

В этом вымышленном примере тестер проверяет, активен ли домен `expireddomain.com` с помощью поиска регистратора домена. Если домен доступен для покупки, поддомен уязвим.

Следующие ответы DNS требуют дальнейшего расследования: `SERVFAIL` or `REFUSED`.

### Тестирование серой коробки

Тестер имеет файл зоны DNS, что означает, что подсчет DNS не требуется. Методология тестирования одинакова.

## Восстановление

Чтобы снизить риск поглощения субдомена, уязвимая запись (и) DNS-ресурса должна быть удалена из зоны DNS. Постоянный мониторинг и периодические проверки рекомендуются в качестве наилучшей практики.

## Tools

- [dig - man page](https://linux.die.net/man/1/dig)
- [recon-ng - Web Reconnaissance framework](https://github.com/lanmaster53/recon-ng)
- [theHarvester - OSINT intelligence gathering tool](https://github.com/laramies/theHarvester)
- [Sublist3r - OSINT subdomain enumeration tool](https://github.com/aboul3la/Sublist3r)
- [dnsrecon - DNS Enumeration Script](https://github.com/darkoperator/dnsrecon)
- [OWASP Amass DNS enumeration](https://github.com/OWASP/Amass)

## References

- [HackerOne - A Guide To Subdomain Takeovers](https://www.hackerone.com/blog/Guide-Subdomain-Takeovers)
- [Subdomain Takeover: Basics](https://0xpatrik.com/subdomain-takeover-basics/)
- [Subdomain Takeover: Going beyond CNAME](https://0xpatrik.com/subdomain-takeover-ns/)
- [can-i-take-over-xyz - A list of vulnerable services](https://github.com/EdOverflow/can-i-take-over-xyz/)
- [OWASP AppSec Europe 2017 - Frans Rosén: DNS hijacking using cloud providers – no verification needed](https://2017.appsec.eu/presos/Developer/DNS%20hijacking%20using%20cloud%20providers%20%E2%80%93%20no%20verification%20needed%20-%20Frans%20Rosen%20-%20OWASP_AppSec-Eu_2017.pdf)
