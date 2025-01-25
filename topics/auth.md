# Авторизация

<tldr>
<p>О работе авторизации и идентификации МЭШа</p>
</tldr>

## Авторизация

МЭШ как продукт не имеет собственной системы авторизации и полностью полагается на OAuth2.

| Регион | Провайдер OAuth2                                               |
|--------|----------------------------------------------------------------|
| Москва | mos.ru                                                         |
| Другие | gosuslugi.ru, kauth [...ru/login](https://school.mos.ru/login) |

### mos.ru

<chapter collapsible="true" title="Ссылки">
Полный путь к модулю: <code>https://school.mos.ru/v3/auth/sudir/login</code>
Authorize URI: <code>https://login.mos.ru/sps/oauth/ae?response_type=code&access_type=offline&client_id=dnevnik.mos.ru&scope=openid+profile+birthday+contacts+snils+blitz_user_rights+blitz_change_password&redirect_uri=https://school.mos.ru/v3/auth/sudir/callback</code>
Redirect URI: <code>https://school.mos.ru/v3/auth/sudir/callback</code>
</chapter>

Используется в Москве, очень трудно автоматизировать из-за большого количества систем защиты, в данный момент используется [headless браузер](https://github.com/RedGuyRu/DnevnikApi/blob/master/src/PuppeteerAuthenticator.js) для автоматизации входа.

#### Процесс ручной авторизации

1. Запускаем headless браузер с аргументом `--disable-blink-features=AutomationControlled`
2. Переходим на `https://school.mos.ru`
3. По клику переходим на страницу авторизации, на текущий момент актуальный selector для этого - `#root > div > div.style_main-container__3z5Nv > main > section > div > div.style_sec-intro_left__2XBWp > div.style_sec-intro_aside__2Be41 > div > div.style_aside-login__3YTaH > div.style_aside-login_action__2KJI4 > div`
4. На полученной странице вводим логин и пароль в поля `#login` и `#password`
5. Подтверждаем вход нажатием кнопки `#bind`
6. 1. Если на странице появился блок `#sms-code` - сайт ждёт ввода СМС кода, если вы не можете заставить пользователя предоставить его, можно перейти на OTP, для этого нажимаем на кнопку `div.twoFa__social--main > div > a:nth-child(2)` или `a[href^="/sps/login/methods2/totp"]` (появляется один из них)
   2. Если на странице появился блок `#otp` - вводим сгенерированный код в поле `#otp` и нажимаем `#save`
   3. Если на странице появился блок `body > div.system__layout > main > section > div > section > h1` - сайт спрашивает нас о доверии к браузеру, нажимаем `#agree`
7. 1. Если мы получили ответ от запроса `https://dnevnik.mos.ru/mobile/api/profile` - мы успешно авторизировались
   2. Если мы сделали запрос на любой метод начинающийся с `https://school.mos.ru/api/family/web/v1/profile` - мы успешно авторизировались
   3. Если на странице появился блок `#pswdMethod-c > blockquote > p` - пароль не верен
8. После успешной авторизации вам необходимо получить значения из cookies браузера

| Cookie        | Значение                                         |
|---------------|--------------------------------------------------|
| profile_id    | ID профиля (непосредственно привязанного к роли) |
| profile_roles | Роли профиля                                     |
| auth_token    | Токен                                            |
| user_id       | ID пользователя (аккаунт всецело)                |

### gosuslugi.ru

<chapter collapsible="true" title="Ссылки">
Полный путь к модулю: <code>https://myschool.72to.ru/v3/auth/esia/login</code>
Authorize URI: <code>https://esia.gosuslugi.ru/aas/oauth2/v2/ac?response_type=code&access_type=offline&client_id=ELSH01721&client_certificate_hash=449ED7E1F84E466499AB928A1439639B75FAE9C2AF61BC768D74FD5C1ECE5020&client_secret=y3_GrxOz8UZXwGe07EXytG3MI7-kK9SkOpT5KzKJUK8fNLFKEfFq9O5YwrQXAauquDFEW-LOsE9zvxqZcfif8Q&timestamp=[REDACTED]&scope=openid+fullname+snils+kid_fullname+kid_birthdate+kid_snils+birth_cert_doc+kid_birth_cert_doc+gender+birthdate&redirect_uri=https%3A%2F%2Fmyschool.72to.ru%2Fv3%2Fauth%2Fesia%2Fcallback&state=[REDACTED]</code>
Redirect URI: <code>https://myschool.72to.ru/v3/auth/esia/callback</code>
</chapter>

Используется в Тюмени и других регионов. Вполне автоматизируется [пример](https://github.com/daniilak/auth-gosuslugi/blob/main/auth_gosuslugi/auth.py).

### myschool.72to.ru/login

<chapter collapsible="true" title="Ссылки">
Полный путь к модулю: <code>https://myschool.72to.ru/v3/auth/kauth/login</code>
Authorize URI: <code>https://myschool.72to.ru/login/?response_type=code&access_type=offline&client_id=kauth&scope=openid+profile+birthday+contacts+snils&redirect_uri=https%3A%2F%2Fmyschool.72to.ru%2Fv3%2Fauth%2Fkauth%2Fcallback%3F</code>
Redirect URI: <code>https://myschool.72to.ru/v3/auth/kauth/callback</code>
</chapter>

Используется в Тюмени. Возможно автоматизируется, но не исследован ввиду отсутствия аккаунтов учеников из Тюмени.

## Идентификация

Для идентификации запроса используется сочетание заголовков `Auth-token`, `Profile-Id` и Cookies `auth_token`, `student_id`.

## OAuth

API позволяет производить OAuth авторизацию, для мобильных приложений. Для этого необходимо отправить пользователя на страницу авторизации OAuth.

`https://login.mos.ru/sps/oauth/ae?scope=birthday%20contacts%20openid%20profile%20snils%20blitz_change_password%20blitz_user_rights%20blitz_qr_auth&access_type=offline&response_type=code&client_id=CLIENT_ID&redirect_uri=dnevnik-mes://oauth2redirect&promt=login&code_challenge=ffd`

Учтите что `CLIENT_ID` создаётся в рантайме перед вызовом для этого надо сделать POST запрос на `https://login.mos.ru/sps/oauth/register`, в теле запроса:
```json
{
"softwareId":"dnevnik.mos.ru",
"deviceType":"android_phone",
"softwareStatement":"eyJ0eXAiOiJKV1QiLCJibGl0ejpraW5kIjoiU09GVF9TVE0iLCJhbGciOiJSUzI1NiJ9.eyJncmFudF90eXBlcyI6WyJhdXRob3JpemF0aW9uX2NvZGUiLCJwYXNzd29yZCIsImNsaWVudF9jcmVkZW50aWFscyIsInJlZnJlc2hfdG9rZW4iXSwic2NvcGUiOiJiaXJ0aGRheSBibGl0el9jaGFuZ2VfcGFzc3dvcmQgYmxpdHpfYXBpX3VzZWNfY2hnIGJsaXR6X3VzZXJfcmlnaHRzIGNvbnRhY3RzIG9wZW5pZCBwcm9maWxlIGJsaXR6X3JtX3JpZ2h0cyBibGl0el9hcGlfc3lzX3VzZXJfY2hnIGJsaXR6X2FwaV9zeXNfdXNlcnMgYmxpdHpfYXBpX3N5c191c2Vyc19jaGcgc25pbHMgYmxpdHpfYXBpX3N5c191c2VjX2NoZyBibGl0el9xcl9hdXRoIiwianRpIjoiYTVlM2NiMGQtYTBmYi00ZjI1LTk3ODctZTllYzRjOTFjM2ZkIiwic29mdHdhcmVfaWQiOiJkbmV2bmlrLm1vcy5ydSIsInNvZnR3YXJlX3ZlcnNpb24iOiIxIiwicmVzcG9uc2VfdHlwZXMiOlsiY29kZSIsInRva2VuIl0sImlhdCI6MTYzNjcyMzQzOSwiaXNzIjoiaHR0cHM6Ly9sb2dpbi5tb3MucnUiLCJyZWRpcmVjdF91cmlzIjpbImh0dHA6Ly9sb2NhbGhvc3QiLCJzaGVsbDovL2F1dGhwb3J0YWwiLCJkbmV2bmlrLW1lczovL29hdXRoMnJlZGlyZWN0IiwiaHR0cHM6Ly9zY2hvb2wubW9zLnJ1L2F1dGgvbWFpbi9jYWxsYmFjayIsImh0dHBzOi8vc2Nob29sLm1vcy5ydS92MS9vYXV0aC9jYWxsYmFjayIsImh0dHBzOi8vZG5ldm5pay5tb3MucnUvc3VkaXIiLCJodHRwczovL3NjaG9vbC5tb3MucnUvYXV0aC9jYWxsYmFjayIsImh0dHA6Ly9kbmV2bmlrLm1vcy5ydS9zdWRpciJdLCJhdWQiOlsiZG5ldm5pay5tb3MucnUiXX0.EERWGw5RGhLQ1vBiGrdG_eJrCyJEyan-H4UWT1gr4B9ZfP58pyJoVw5wTt8YFqzwbvHNQBnvrYfMCzOkHpsU7TxlETJpbWcWbnV5JI-inzXGyKCic2fAVauVCjos3v6AFiP6Uw6ZXIC6b9kQ5WgRVM66B9UwAB2MKTThTohJP7_MNZJ0RiOd8RLlvF4C7yfuqoGU2-KWLwr78ATniTvYFWszl8jAi_SiD9Ai1GWW4mO9-JQ01f4N9umC5Cy2tYiZhxbaz2rOsAQBBjY6rbCCJbCpb1lyGfs2qhhAB-ODGTq7W7r1WBlAm5EXlPpuW_9pi8uxdxiqjkG3d6xy7h7gtQ",
"redirect_uris":"dnevnik-mes://oauth2redirect"
}
```
И в headers:
```http
authorization: Bearer FqzGn1dTJ9BQCHgV0rmMjtYFIgaFf9TrGVEzgtju-zbtIbeJSkIyDcl0e2QMirTNpEqovTT8NvOLZI0XklVEIw
```

После успешной авторизации браузер перенаправит на адрес `dnevnik-mes://oauth2redirect`.

> Опытным путём было выяснено, что `http://localhost` разрешён для redirect-url.