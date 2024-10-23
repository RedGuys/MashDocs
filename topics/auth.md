# Авторизация

<tldr>
<p>Об работе авторизации и идентификация МЭШе</p>
</tldr>

## Авторизация

МЭШ как продукт не имеет собственной системы авторизации и полностью полагается на OAuth2.

| Регион | Провайдер OAuth2                                |
|--------|-------------------------------------------------|
| Москва | mos.ru                                          |
| Тюмень | gosuslugi.ru и локальная myschool.72to.ru/login |

### mos.ru

<chapter collapsible="true" title="Ссылки">
Полный путь к модулю: `https://school.mos.ru/v3/auth/sudir/login`
Authorize URI: `https://login.mos.ru/sps/oauth/ae?response_type=code&access_type=offline&client_id=dnevnik.mos.ru&scope=openid+profile+birthday+contacts+snils+blitz_user_rights+blitz_change_password&redirect_uri=https://school.mos.ru/v3/auth/sudir/callback`
Redirect URI: `https://school.mos.ru/v3/auth/sudir/callback`
</chapter>

Используется в Москве, очень трудно автоматизировать из-за большого количества систем защиты, в данный момент используется [headless браузер](https://github.com/RedGuyRu/DnevnikApi/blob/master/src/PuppeteerAuthenticator.js) для автоматизации входа.

### gosuslugi.ru

<chapter collapsible="true" title="Ссылки">
Полный путь к модулю: `https://myschool.72to.ru/v3/auth/esia/login`
Authorize URI: `https://esia.gosuslugi.ru/aas/oauth2/v2/ac?response_type=code&access_type=offline&client_id=ELSH01721&client_certificate_hash=449ED7E1F84E466499AB928A1439639B75FAE9C2AF61BC768D74FD5C1ECE5020&client_secret=y3_GrxOz8UZXwGe07EXytG3MI7-kK9SkOpT5KzKJUK8fNLFKEfFq9O5YwrQXAauquDFEW-LOsE9zvxqZcfif8Q&timestamp=[REDACTED]&scope=openid+fullname+snils+kid_fullname+kid_birthdate+kid_snils+birth_cert_doc+kid_birth_cert_doc+gender+birthdate&redirect_uri=https%3A%2F%2Fmyschool.72to.ru%2Fv3%2Fauth%2Fesia%2Fcallback&state=[REDACTED]` 
Redirect URI: `https://myschool.72to.ru/v3/auth/esia/callback`
</chapter>

Используется в Тюмени. Вполне автоматизируется [пример](https://github.com/daniilak/auth-gosuslugi/blob/main/auth_gosuslugi/auth.py).

### myschool.72to.ru/login

<chapter collapsible="true" title="Ссылки">
Полный путь к модулю: `https://myschool.72to.ru/v3/auth/kauth/login`
Authorize URI: `https://myschool.72to.ru/login/?response_type=code&access_type=offline&client_id=kauth&scope=openid+profile+birthday+contacts+snils&redirect_uri=https%3A%2F%2Fmyschool.72to.ru%2Fv3%2Fauth%2Fkauth%2Fcallback%3F`
Redirect URI: `https://myschool.72to.ru/v3/auth/kauth/callback`
</chapter>

Используется в Тюмени. Возможно автоматизируется, но не исследован ввиду отсутствия аккаунтов учеников из Тюмени.

## Идентификация

Для идентификации запроса используется сочетание заголовков `Auth-token`, `Profile-Id` и Cookies `auth_token`, `student_id`.