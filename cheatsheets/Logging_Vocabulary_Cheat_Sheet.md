# Шпаргалка по лексике для ведения журнала приложений

В этом документе предлагается стандартный словарь для регистрации событий безопасности. Цель состоит в том, чтобы упростить мониторинг и оповещение таким образом, чтобы, если предположить, что разработчики отслеживают ошибки и регистрируют их, используя этот словарь, мониторинг и оповещение можно было бы улучшить, просто используя эти термины.

## Обзор

Каждый год IBM Security поручает Ponemon Institute опросить компании по всему миру на предмет информации о нарушениях безопасности, мерах по их устранению и связанных с этим затратах; результат называется стоимостью отчета о нарушениях данных.

В дополнение к миллионам долларов, потерянных из-за утечек, в отчете говорится, что **среднее время обнаружения** утечки по-прежнему составляет около **200 дней**. Очевидно, что возможность мониторинга приложений и оповещения об аномальном поведении улучшит время выявления и смягчения последствий атаки на наши приложения.

![Отчет IBM о затратах на утечку данных за 2020 год](../assets/cost-of-breach-2020.png)

> Исследование IBM "Затраты на утечку данных", 2020, рис.34, стр.52, [https://www.ibm.com/security/data-breach]

Этот стандарт ведения журнала будет направлен на определение конкретных ключевых слов, которые при последовательном применении ко всему программному обеспечению позволят группам просто отслеживать эти события во всех приложениях и быстро реагировать в случае атаки.

## Допущения

- Группы по наблюдению/SRE должны поддерживать использование этого стандарта и поощрять разработчиков к его использованию
- Система реагирования на инциденты должна либо использовать эти данные, либо предоставлять средства, с помощью которых другие группы мониторинга могут отправлять уведомления о тревоге, предпочтительно программно.
- Архитекторы должны поддерживать, внедрять и вносить свой вклад в этот стандарт
- Разработчики должны принять этот стандарт и приступить к его внедрению (требуются знания и намерение для понимания потенциальных атак и устранения этих ошибок в коде).

## Приступаем к работе

Напоминаю, что цель ведения журнала - иметь возможность оповещать о конкретных событиях безопасности. Конечно, первым шагом к ведению журнала этих событий является хорошая обработка ошибок, если вы не отслеживаете события, вам не нужно регистрировать событие.

### Идентификация событий

Для лучшего понимания ведения журнала событий системы безопасности было бы полезно получить хорошее представление о моделировании угроз на высоком уровне, даже если это простой подход:

1. **Что может пойти не так?**

- Заказы: может ли кто-то сделать заказ от имени другого лица?
- Аутентификация: могу ли я войти в систему от имени другого лица?
- Авторизация: могу ли я просмотреть чужую учетную запись?

2. **Что произойдет, если это произойдет?**

- Заказы: Я разместил заказ от имени другого лица... на заброшенный склад в Нью-Джерси. Ой.
- Потом я похвастался этим на 4Chan.
- Потом я рассказал об этом New York Times.

3. **Кто мог намереваться это сделать?**

- Преднамеренные атаки хакеров.
- Сотрудник, "тестирующий", как все работает.
- Неправильно запрограммированный API, который делает то, чего автор не планировал.

## Формат

_ПРИМЕЧАНИЕ: Все даты должны быть записаны в формате [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) **СО смещением** по UTC для обеспечения максимальной переносимости_

```
{
    "datetime": "2021-01-01T01:01:01-0700",
    "appid": "foobar.netportal_auth",
    "event": "AUTHN_login_success:joebob1",
    "level": "INFO",
    "description": "User joebob1 login successfully",
    "useragent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36",
    "source_ip": "165.225.50.94",
    "host_ip": "10.12.7.9",
    "hostname": "portalauth.foobar.com",
    "protocol": "https",
    "port": "440",
    "request_uri": "/api/v2/auth/",
    "request_method": "POST",
    "region": "AWS-US-WEST-2",
    "geo": "USA"
}
```

## Словарь

Ниже приведены различные типы событий, которые должны быть зафиксированы. Для каждого типа события есть префикс типа "authn" и дополнительные данные, которые должны быть включены для этого события.

Например, включены фрагменты полного формата ведения журнала, но полный журнал событий должен соответствовать приведенному выше формату.

---

## Аутентификация [AUTHN]

### ### auth_login_success[:идентификатор пользователя]

**Описание**
Все события входа в систему должны быть записаны, включая успешный результат.

**Уровень:**
информация

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "authn_login_success:joebob1",
    "level": "INFO",
    "description": "User joebob1 login successfully",
    ...
}
```

---

 ### auth_login_success после неудачи[:user id,retries]

**Описание**
Пользователь успешно вошел в систему после предыдущей ошибки.

**Уровень:**
информация

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "authn_login_successafterfail:joebob1,2",
    "level": "INFO",
    "description": "User joebob1 login successfully",
    ...
}
```

---

### ### auth_login_fail[:userid]

**Описание**
Все события входа в систему, включая сбой, должны быть записаны.

**Уровень:**
предупреждать

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "authn_login_fail:joebob1",
    "level": "WARN",
    "description": "User joebob1 login failed",
    ...
}
```

---

### authn_login_fail_max[:userid,maxlimit(int)]

**Описание**
Все события входа в систему, включая сбой, должны быть записаны.

**Уровень:**
предупреждать

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "authn_login_fail_max:joebob1,3",
    "level": "WARN",
    "description": "User joebob1 reached the login fail limit of 3",
    ...
}
```

---

### authn_login_lock[:userid,reason]

**Описание**
Если существует функция блокировки учетной записи после x повторных попыток или при других обстоятельствах, блокировка должна быть зарегистрирована с соответствующими данными.

**Уровень:**
Предупреждение

**Причины:**

- максимальное количество попыток: Достигнуто максимальное количество повторных попыток
- подозрительный: В учетной записи наблюдалась подозрительная активность
- клиент: Клиент запросил блокировку своей учетной записи
- другое: Другое

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "authn_login_lock:joebob1,maxretries",
    "level": "WARN",
    "description": "User joebob1 login locked because maxretries exceeded",
    ...
}
```

---

### authn_password_change[:userid]

**Описание**
Каждое изменение пароля должно регистрироваться, включая идентификатор пользователя, для которого оно было произведено.

**Уровень:**
Информация

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "authn_password_change:joebob1",
    "level": "INFO",
    "description": "User joebob1 has successfully changed their password",
    ...
}
```

---

### authn_password_change_fail[:userid]

**Описание**
Неудачная попытка сменить пароль. Также могут возникать другие события, такие как "authn_login_lock`.

**Уровень:**
Информация

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "authn_password_change_fail:joebob1",
    "level": "INFO",
    "description": "User joebob1 failed to changing their password",
    ...
}
```

---

### authn_impossible_travel[:userid,region1,region2]

**Описание**
Когда пользователь входит в систему из одного города и внезапно появляется в другом, который находится слишком далеко, чтобы добраться туда за разумное время, это часто указывает на возможный захват учетной записи.

**Уровень:**: КРИТИЧЕСКИЙ

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "authn_impossible_travel:joebob1,US-OR,CN-SH",
    "level": "CRITICAL",
    "description": "User joebob1 has accessed the application in two distant cities at the same time",
    ...
}
```

---

### auth_token_created[:идентификатор пользователя, права доступа]

**Описание**
Когда токен создается для доступа к сервису, он должен быть записан

**Уровень:**: ИНФОРМАЦИЯ

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "aws.foobar.com",
    "event": "authn_token_created:app.foobarapi.prod,create,read,update",
    "level": "INFO",
    "description": "A token has been created for app.foobarapi.prod with create,read,update",
    ...
}
```

---

### authn_token_revoked[:userid,tokenid]

**Описание**
Токен был отозван для данной учетной записи.

**Уровень:**: ИНФОРМАЦИЯ

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "aws.foobar.com",
    "event": "authn_token_revoked:app.foobarapi.prod,xyz-abc-123-gfk",
    "level": "INFO",
    "description": "Token ID: xyz-abc-123-gfk was revoked for user app.foobarapi.prod",
    ...
}
```

---

### authn_token_reuse[:userid,tokenid]

**Описание**
Была предпринята попытка повторно использовать ранее отозванный токен.

**Уровень:**: КРИТИЧЕСКИЙ

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "aws.foobar.com",
    "event": "authn_token_reuse:app.foobarapi.prod,xyz-abc-123-gfk",
    "level": "CRITICAL",
    "description": "User app.foobarapi.prod attempted to use token ID: xyz-abc-123-gfk which was previously revoked",
    ...
}
```

---

### authn_token_delete[:appid]

**Описание**
Когда токен удаляется, он должен быть записан

**Уровень:**: ПРЕДУПРЕЖДЕНИЕ

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "authn_token_delete:foobarapi",
    "level": "WARN",
    "description": "The token for foobarapi has been deleted",
    ...
}
```

---

## Authorization [AUTHZ]

---

### authz_fail[:userid,resource]

**Описание**
Была предпринята попытка несанкционированного доступа к ресурсу

**Уровень:**: КРИТИЧЕСКИЙ

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "authz_fail:joebob1,resource",
    "level": "CRITICAL",
    "description": "User joebob1 attempted to access a resource without entitlement",
    ...
}
```

---

### authz_change[:userid,from,to]

**Описание**
Права пользователя или организации были изменены

**Уровень:**: ПРЕДУПРЕЖДЕНИЕ

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "authz_change:joebob1,user,admin",
    "level": "WARN",
    "description": "User joebob1 access was changed from user to admin",
    ...
}
```

---

### authz_admin[:userid,event]

**Описание**
Все действия привилегированных пользователей, таких как администратор, должны записываться.

**Уровень:**: ПРЕДУПРЕЖДЕНИЕ

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "authz_admin:joebob1,user_privilege_change",
    "level": "WARN",
    "description": "Administrator joebob1 has updated privileges of user foobarapi from user to admin",
    ...
}
```

---

## Чрезмерное употребление [EXCESS]

### excess_rate_limit_exceeded[userid,max]

**Описание**
Следует установить ожидаемые предельные значения обслуживания и предупреждать о превышении, даже если это просто для управления затратами и масштабирования.

**Уровень:**: ПРЕДУПРЕЖДЕНИЕ

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "excess_rate_limit_exceeded:app.foobarapi.prod,100000",
    "level": "WARN",
    "description": "User app.foobarapi.prod has exceeded max:100000 requests",
    ...
}
```

---

## File Upload [UPLOAD]

### upload_complete[userid,filename,type]

**Описание**
При успешной загрузке файла первым шагом в процессе проверки является подтверждение того, что загрузка завершена.

**Уровень:**: ИНФОРМАЦИЯ

**Пример:**

```
    {
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "upload_complete:joebob1,user_generated_content.png,PNG",
    "level": "INFO",
    "description": "User joebob1 has uploaded user_generated_content.png",
    ...
}
```

---

### upload_stored[filename,from,to]

**Описание**
Одним из этапов проверки правильности загрузки файла является перемещение/переименование файла, а при возврате содержимого конечным пользователям никогда не указывайте исходное имя файла при загрузке. Это справедливо как при хранении в файловой системе, так и в блочном хранилище.

**Уровень:**: ИНФОРМАЦИЯ

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "upload_stored:user_generated_content.png,kjsdhkrjhwijhsiuhdf000010202002",
    "level": "INFO",
    "description": "File user_generated_content.png was stored in the database with key abcdefghijk101010101",
    ...
}
```

---

### upload_validation[filename,(virusscan|imagemagick|...):(FAILED|incomplete|passed)]

**Описание**
Все загружаемые файлы должны быть проверены как на корректность (действительно относятся к типу файлов x), так и на безопасность (не содержат вирусов).

**Уровень:**: ИНФОРМАЦИОННЫЙ|КРИТИЧЕСКИЙ

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "upload_validation:filename,virusscan:FAILED",
    "level": "CRITICAL",
    "description": "File user_generated_content.png FAILED virus scan and was purged",
    ...
}
```

---

### upload_delete[userid,fileid]

**Описание**
Когда файл удаляется по обычным причинам, он должен быть записан.

**Уровень:**: ИНФОРМАЦИЯ

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "upload_delete:joebob1,",
    "level": "INFO",
    "description": "User joebob1 has marked file abcdefghijk101010101 for deletion.",
    ...
}
```

---

## Input Validation [INPUT]

### input_validation_fail[:field,userid]

**Описание**
Сбой проверки введенных данных на стороне сервера может быть вызван либо тем, что: а) на клиенте не была предоставлена достаточная проверка, либо б) проверка на стороне клиента была пропущена. В любом случае это возможность для атаки, и ее следует быстро устранить.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "input_validation_fail:date_of_birth,joebob1",
    "level": "WARN",
    "description": "User joebob1 submitted data that failed validation.",
    ...
}
```

---

## Malicious Behavior [MALICIOUS

### malicious_excess_404:[userid|IP,useragent]

**Описание**
Когда пользователь делает многочисленные запросы на несуществующие файлы, это часто указывает на попытки "принудительного просмотра" файлов, которые могли бы существовать, и часто является поведением, указывающим на злой умысел.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "malicious_excess404:123.456.789.101,M@l1c10us-Hax0rB0t0-v1",
    "level": "WARN",
    "description": "A user at 123.456.789.101 has generated a large number of 404 requests.",
    ...
}
```

---

### malicious_extraneous:[userid|IP,inputname,useragent]

**Описание**
Когда пользователь отправляет данные серверному обработчику, которые не ожидались, это может указывать на поиск ошибок проверки ввода. Если ваша серверная служба получает данные, которые она не обрабатывает, или для которых нет входных данных, это указывает на вероятное злонамеренное использование.

**Уровень:**
критический

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "malicious_extraneous:dr@evil.com,creditcardnum,Mozilla/5.0 (X11; Linux x86_64; rv:10.0) Gecko/20100101 Firefox/10.0",
    "level": "WARN",
    "description": "User dr@evil.com included field creditcardnum in the request which is not handled by this service.",
    ...
}
```

---

### malicious_attack_tool:[userid|IP,toolname,useragent]

**Описание**
Когда очевидные средства атаки идентифицируются либо по сигнатуре, либо с помощью агента пользователя, их следует регистрировать.

**ЧТО НУЖНО СДЕЛАТЬ:** В будущей версии этого стандарта должны быть ссылки на известные средства атаки, сигнатуры и строки агента пользователя. Например, инструмент "Nikto" по умолчанию оставляет свой пользовательский агент со строкой типа **_"Mozilla/5.00 (Nikto/2.1.6) (Отклонения: Нет) (Тест: Проверка порта)"_**

**Уровень:**
критический

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "malicious_attack_tool:127.0.0.1,nikto,Mozilla/5.00 (Nikto/2.1.6) (Evasions:None) (Test:Port Check)",
    "level": "WARN",
    "description": "Attack traffic indicating use of Nikto coming from 127.0.0.1",
    ...
}
```

---

### malicious_cors:[userid|IP,useragent,referer]

**Описание**
Попытки, совершаемые из несанкционированных источников, должны, конечно, блокироваться, но по возможности также регистрироваться. Даже если мы заблокируем незаконный запрос из разных источников, сам факт отправки запроса может свидетельствовать об атаке.

_ПРИМЕЧАНИЕ: Знаете ли вы, что в исходной спецификации HTTP слово "referer" написано с ошибкой? Правильным написанием должно быть "referrer", но исходная опечатка сохраняется по сей день и используется здесь намеренно._

**Уровень:**
критический

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "malicious_cors:127.0.0.1,Mozilla/5.0 (X11; Linux x86_64; rv:10.0) Gecko/20100101 Firefox/10.0,attack.evil.com",
    "level": "WARN",
    "description": "An illegal cross-origin request from 127.0.0.1 was referred from attack.evil.com"
    ...
}
```

---

### malicious_direct_reference:[userid|IP, useragent]

**Описание**
Распространенной атакой против аутентификации и авторизации является прямой доступ к объекту без учетных данных или соответствующих полномочий доступа. Неспособность предотвратить этот недостаток раньше входила в десятку OWASP под названием **Небезопасная прямая ссылка на объект**. Если предположить, что вы правильно предотвратили эту атаку, регистрация попытки будет полезна для выявления злоумышленников.

**Уровень:**
КРИТИЧЕСКИЙ

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "malicious_direct:joebob1, Mozilla/5.0 (X11; Linux x86_64; rv:10.0) Gecko/20100101 Firefox/10.0",
    "level": "WARN",
    "description": "User joebob1 attempted to access an object to which they are not authorized",
    ...
}
```

---

## Изменения привилегий [PRIVILEGE]

Этот раздел посвящен изменениям привилегий объектов, таким как разрешения на чтение/запись/выполнение, или объектам в базе данных, для которых изменена метаинформация авторизации.

Изменения пользователя/учетной записи рассматриваются в разделе "Управление пользователями".

---

### privilege_permissions_changed:[userid,file|object,fromlevel,tolevel]

**Описание**
Отслеживание изменений в объектах, доступ к которым ограничен, может выявить попытки повышения прав доступа к этим файлам со стороны неавторизованных пользователей.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "malicious_direct:joebob1, /users/admin/some/important/path,0511,0777",
    "level": "WARN",
    "description": "User joebob1 changed permissions on /users/admin/some/important/path",
    ...
}
```

---

## Изменения в конфиденциальных данных [DATA]

Необязательно регистрировать или оповещать об изменениях во всех файлах, но в случае особо важных файлов или данных важно, чтобы мы отслеживали изменения и оповещали об изменениях.

---

### sensitive_create:[userid,file|object]

**Описание**
Когда создается новый фрагмент данных, который помечается как конфиденциальный или помещается в каталог/таблицу/хранилище, где хранятся конфиденциальные данные, это создание должно периодически регистрироваться и проверяться.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "sensitive_create:joebob1, /users/admin/some/important/path",
    "level": "WARN",
    "description": "User joebob1 created a new file in /users/admin/some/important/path",
    ...
}
```

---

### sensitive_read:[userid,file|object]

**Описание**
Все данные, помеченные как конфиденциальные или помещенные в каталог/таблицу/хранилище, где хранятся конфиденциальные данные, должны регистрироваться и периодически проверяться.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "sensitive_read:joebob1, /users/admin/some/important/path",
    "level": "WARN",
    "description": "User joebob1 read file /users/admin/some/important/path",
    ...
}
```

---

### sensitive_update:[userid,file|object]

**Описание**
Все данные, помеченные как конфиденциальные или помещенные в каталог/таблицу/хранилище, где хранятся конфиденциальные данные, должны периодически обновляться и проверяться.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "sensitive_update:joebob1, /users/admin/some/important/path",
    "level": "WARN",
    "description": "User joebob1 modified file /users/admin/some/important/path",
    ...
}
```

---

### sensitive_delete:[userid,file|object]

**Описание**
Все данные, помеченные как конфиденциальные или помещенные в каталог/таблицу/хранилище, где хранятся конфиденциальные данные, должны периодически регистрироваться и проверяться при удалении. Файл не должен быть немедленно удален, но должен быть помечен для удаления, и архив файла должен храниться в соответствии с требованиями законодательства / конфиденциальности.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "sensitive_delete:joebob1, /users/admin/some/important/path",
    "level": "WARN",
    "description": "User joebob1 marked file /users/admin/some/important/path for deletion",
    ...
}
```

---

## Ошибки в последовательности [SEQUENCE]

Также называемая **_атакой бизнес-логики_**, если в системе ожидается определенный путь, а попытка пропустить его или изменить порядок следования может указывать на злой умысел.

---

### sequence_fail:[userid]

**Описание**
Когда пользователь переходит к какой-либо части приложения не по порядку, это может свидетельствовать о намеренном нарушении бизнес-логики и должно отслеживаться.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "sequence_fail:joebob1",
    "level": "WARN",
    "description": "User joebob1 has reached a part of the application out of the normal application flow.",
    ...
}
```

---

## Session Management [SESSION]

### session_created:[userid]

**Описание**
Когда создается новый аутентифицированный сеанс, этот сеанс может регистрироваться и отслеживаться активность.

**Уровень:**
Информация

**Пример:**

```
    {
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "session_created:joebob1",
    "level": "INFO",
    "description": "User joebob1 has started a new session",
    ...
}
```

---

### session_renewed:[userid]

**Описание**
Когда пользователь получает предупреждение об истечении срока действия сеанса и решает продлить его, это действие должно быть зарегистрировано. Кроме того, если рассматриваемая система содержит строго конфиденциальные данные, для продления сеанса может потребоваться дополнительная проверка.

**Уровень:**
Информация

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "session_renewed:joebob1",
    "level": "WARN",
    "description": "User joebob1 was warned of expiring session and extended.",
    ...
}
```

---

### session_expired:[userid,reason]

**Описание**
По истечении срока действия сеанса, особенно в случае аутентифицированного сеанса или с конфиденциальными данными, этот сеанс может быть запротоколирован и в него могут быть включены уточняющие данные. Код причины может быть любым, например: выход из системы, тайм-аут, отменен и т.д. Сессии ни в коем случае не должны удаляться, а должны быть с истекшим сроком действия в случае требования об отзыве.

**Уровень:**
Информация

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "session_expired:joebob1,revoked",
    "level": "WARN",
    "description": "User joebob1 session expired due to administrator revocation.",
    ...
}
```

---

### session_use_after_expire:[userid]

**Описание**
В случае, если пользователь пытается получить доступ к системам с истекшим сеансом, может оказаться полезным войти в систему, особенно в сочетании с последующим сбоем при входе. Это может указывать на случай, когда злоумышленник пытается перехватить сеанс или получить прямой доступ к компьютеру/браузеру другого человека.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "session_use_after_expire:joebob1",
    "level": "WARN",
    "description": "User joebob1 attempted access after session expired.",
    ...
}
```

---

## Системные события [SYS]

### sys_startup:[userid]

**Описание**
При первом запуске системы может оказаться полезным зарегистрировать запуск, даже если система является бессерверной или контейнерной, особенно если это возможно для регистрации пользователя, запустившего систему.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "sys_startup:joebob1",
    "level": "WARN",
    "description": "User joebob1 spawned a new instance",
    ...
}
```

---

### sys_shutdown:[userid]

**Описание**
При завершении работы системы может оказаться полезным зарегистрировать событие, даже если система является бессерверной или контейнерной, особенно если это возможно для регистрации пользователя, который инициировал запуск системы.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "sys_shutdown:joebob1",
    "level": "WARN",
    "description": "User joebob1 stopped this instance",
    ...
}
```

---

### sys_restart:[userid]

**Описание**
При перезапуске системы может оказаться полезным зарегистрировать событие, даже если система является бессерверной или контейнерной, особенно если это возможно для регистрации пользователя, запустившего систему.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "sys_restart:joebob1",
    "level": "WARN",
    "description": "User joebob1 initiated a restart",
    ...
}
```

---

### sys_crash[:reason]

**Описание**
Если есть возможность зафиксировать нестабильное состояние, приводящее к сбою системы, может быть полезно зарегистрировать это событие, особенно если оно вызвано атакой.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "sys_crash:outofmemory,
    "level": "WARN",
    "description": "The system crashed due to Out of Memory error.",
    ...
}
```

---

### sys_monitor_disabled:[userid,monitor]

**Описание**
Если в вашей системе есть агенты, отвечающие за целостность файлов, ресурсы, ведение журнала, защиту от вирусов и т.д. особенно важно знать, были ли они остановлены и кем.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "sys_monitor_disabled:joebob1,crowdstrike",
    "level": "WARN",
    "description": "User joebob1 has disabled CrowdStrike",
    ...
}
```

---

### sys_monitor_enabled:[userid,monitor]

**Описание**
Если в вашей системе есть агенты, отвечающие за целостность файлов, ресурсы, ведение журнала, защиту от вирусов и т.д. особенно важно знать, запускаются ли они снова после остановки и кем.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "sys_monitor_enabled:joebob1,crowdstrike",
    "level": "WARN",
    "description": "User joebob1 has enabled CrowdStrike",
    ...
}
```

---

## User Management [USER]

### user_created:[userid,newuserid,attributes[one,two,three]]

**Описание**
При создании новых пользователей полезно регистрировать особенности события создания пользователя, особенно если новые пользователи могут быть созданы с правами администратора.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "user_created:joebob1,user1,admin:create,update,delete",
    "level": "WARN",
    "description": "User joebob1 created user1 with admin:create,update,delete privilege attributes",
    ...
}
```

---

### user_updated:[userid,onuserid,attributes[one,two,three]]

**Описание**
При обновлении пользователей полезно записывать в журнал информацию о событии обновления пользователя, особенно если пользователи могут обновляться с правами администратора.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "user_updated:joebob1,user1,admin:create,update,delete",
    "level": "WARN",
    "description": "User joebob1 updated user1 with attributes admin:create,update,delete privilege attributes",
    ...
}
```

---

### user_archived:[userid,onuserid]

**Описание**
Всегда лучше архивировать пользователей, а не удалять, за исключением случаев, когда это необходимо. При архивировании пользователей полезно записывать в журнал информацию о событии архивации пользователей. Злоумышленник может использовать эту функцию, чтобы отказать в обслуживании законным пользователям.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "user_archived:joebob1,user1",
    "level": "WARN",
    "description": "User joebob1 archived user1",
    ...
}
```

---

### user_deleted:[userid,onuserid]

**Описание**
Всегда лучше архивировать пользователей, а не удалять, за исключением случаев, когда это необходимо. При удалении пользователей полезно записывать в журнал информацию о событии удаления пользователя. Злоумышленник может использовать эту функцию, чтобы отказать в обслуживании законным пользователям.

**Уровень:**
Предупреждение

**Пример:**

```
{
    "datetime": "2019-01-01 00:00:00,000",
    "appid": "foobar.netportal_auth",
    "event": "user_deleted:joebob1,user1",
    "level": "WARN",
    "description": "User joebob1 has deleted user1",
    ...
}
```

---

## Исключения

Важно не только то, что вы регистрируете, но и то, чего вы НЕ регистрируете. Личная или секретная информация, исходный код, ключи, сертификаты и т.д. никогда не должны регистрироваться.

Для получения подробного обзора элементов, которые следует исключить из ведения журнала, пожалуйста, ознакомьтесь с [инструкцией по ведению журнала OWASP](../cheatsheets/Logging_Cheat_Sheet.md#данные-для-исключения).
