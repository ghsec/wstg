# Тестирование на Политику безопасности контента

| ID |
| ------------- |
| WSTG-CONF-12 |

## Резюме

Политика безопасности контента (CSP) - это декларативная политика allow-list, применяемая через заголовок ответа «Content-Security-Policy» или эквивалентный элемент «<meta>». Это позволяет разработчикам ограничивать источники, из которых такие ресурсы, как JavaScript, CSS, изображения, файлы и т. Д. загружены. CSP - это эффективная методика защиты глубины для снижения риска таких уязвимостей, как кросс-сайтовый скриптинг (XSS) и Clickjacking.

Политика безопасности контента поддерживает директивы, которые позволяют детальный контроль потоку политик. (Видеть [References](#references) для более подробной информации.)

## Цели теста

- Просмотрите заголовок Content-Security-Policy или метаэлемент, чтобы определить неправильные конфигурации.

## Как проверить

Чтобы проверить неправильную конфигурацию в CSP, ищите небезопасные конфигурации, изучая `Content-Security-Policy` Заголовок ответа HTTP или CSP `meta` элемент в прокси-инструменте:

- `unsafe-inline` Директива позволяет встроить сценарии или стили, делая приложения восприимчивыми к атакам XSS.
- `unsafe-eval` директива позволяет использовать `eval ()` в приложении.
- Ресурсы, такие как скрипты, могут быть загружены из любого источника с помощью подстановочного знака (`*`).
    - Также рассмотрим подстановочные знаки, основанные на частичных совпадениях, таких как: `https: // *` или `*.cdn.com`.
    - Подумайте, предоставляют ли перечисленные источники конечные точки JSONP, которые могут использоваться для обхода CSP или политики того же происхождения.
- Обрамление может быть включено для всех источников с помощью подстановочного (` *`) источника для директивы `frame-ancestors`.
- Бизнес-критические приложения должны требовать строгой политики.

## Восстановление

Настройте строгую политику безопасности контента, которая уменьшает поверхность атаки приложения. Разработчики могут проверить силу политики безопасности контента, используя такие онлайн-инструменты, как [Google CSP Evaluator](https://csp-evaluator.withgoogle.com/).

### Strict Policy

Строгая политика - это политика, которая обеспечивает защиту от классических сохраненных, отраженных и некоторых атак DOM XSS и должна быть оптимальной целью любой команды, пытающейся внедрить CSP

Google пошел дальше и создал руководство для принятия строгого CSP на основе nonces. На основе презентации в [LocoMocoSec](https://speakerdeck.com/lweichselbaum/csp-a-successful-mess-between-hardening-and-mitigation?slide=55), следующие две политики могут быть использованы для применения строгой политики:

Moderate Strict Policy:

```HTTP
script-src 'nonce-r4nd0m' 'strict-dynamic';
object-src 'none'; base-uri 'none';
```

Locked down Strict Policy:

```HTTP
script-src 'nonce-r4nd0m';
object-src 'none'; base-uri 'none';
```

## Tools

- [Google CSP Evaluator](https://csp-evaluator.withgoogle.com/)
- [CSP Auditor - Burp Suite Extension](https://portswigger.net/bappstore/35237408a06043e9945a11016fcbac18)
- [CSP Generator Chrome](https://chrome.google.com/webstore/detail/content-security-policy-c/ahlnecfloencbkpfnpljbojmjkfgnmdc) / [Firefox](https://addons.mozilla.org/en-US/firefox/addon/csp-generator/)

## References

- [OWASP Content Security Policy Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
- [Mozilla Developer Network: Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [CSP Level 3 W3C](https://www.w3.org/TR/CSP3/)
- [CSP with Google](https://csp.withgoogle.com/docs/index.html)
- [Content-Security-Policy](https://content-security-policy.com/)
- [Google CSP Evaluator](https://csp-evaluator.withgoogle.com/)
- [CSP A Successful Mess Between Hardening And Mitigation](https://speakerdeck.com/lweichselbaum/csp-a-successful-mess-between-hardening-and-mitigation)
