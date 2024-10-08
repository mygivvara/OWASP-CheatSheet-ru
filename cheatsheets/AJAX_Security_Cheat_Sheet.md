# Шпаргалка по безопасности AJAX

## Введение

Данный документ поставит стартовую точку в безопасности AJAX и, мы надеемся, будет обновляться и расширяться достаточно часто, для предоставления более подробной информации о некоторых фреймворках и технологиях.

### Клиентская сторона (JavaScript)

#### Используйте `.innerText` вместо `.innerHTML`

Использование `.innerText` предотвратит большинство проблем связанных с XSS, поскольку оно будет кодировать текст автоматически.

#### Не используйте `eval()`, `new Function()` и другие инструменты оценки кода

`eval()` является вредной функцией, никогда её не используйте. Необходимость её использования зачастую показывает проблемы в дизайне приложения.

#### Канонизируйте данные отправляемые потребителю (кодируйте перед использованием)

Когда вы используете данные для сборки HTML, скриптов, CSS, XML, JSON, и т.д. убедитесь что вы принимаете во внимание, что эти данные должны быть представленные в буквальном смысле, чтобы сохранить её логическое значение.

Данные должны быть закодированы перед использованием, для предотвращения проблем с разного рода инъекциями, и чтобы удостовериться в том, что логический смысл сохранён.

[Ознакомьтесь с проектом Java кодировщика OWASP.](https://owasp.org/www-project-java-encoder/)

#### Не полагайтесь на клиентскую логику в вопросах безопасности

Не забывайте, что пользователь управляет логикой клиентской стороны. Существуют различные плагины для браузера, с помощью которых можно устанавливать точки остановки, пропускать код, изменять значения и т. д. Никогда не полагайтесь на клиентскую логику в вопросах безопасности.

#### Никогда не полагайтесь на клиентскую бизнес-логику

Так же как и в вопросах безопасности, убедитесь, что все важные бизнес-правила и логика дублируются на серверной стороне, чтобы пользователь не смог обойти необходимую логику и не сделать чего-то глупого или, что хуже, дорогостоящего.

#### Избегайте написания кода сериализации.

Это сложно, и даже небольшая ошибка может привести к серьёзным проблемам с безопасностью. Уже существует множество фреймворков, предоставляющих эту функциональность.

Обратитесь к [странице JSON](http://www.json.org/) для ссылок.

#### Избегайте динамического построения XML или JSON

Так же как при построении HTML или SQL, это может привести к уязвимостям, связанным с внедрением XML. Поэтому лучше воздержитесь от этого или, по крайней мере, используйте библиотеку кодирования или безопасную библиотеку JSON или XML, чтобы обеспечить безопасность атрибутов и данных элементов.

- [Предотвращение XSS (Cross Site Scripting)](Cross_Site_Scripting_Prevention_Cheat_Sheet.md)
- [Предотвращение SQL инъекций](SQL_Injection_Prevention_Cheat_Sheet.md)

#### Никогда не передавайте секреты клиенту

Всё что знает клиент, будет знать и пользователь. Так что пожалуйста, держите секреты на сервере.

#### Не проводите шифрование на стороне клиента

Используйте TLS/SSL и шифруйте данные на сервере!

#### Не выполняйте влияющую на безопасность логику на стороне клиента

Это общее правило, которое выручает меня, если я что-то упустил. :)

### Серверная сторона

#### Используйте защиту от CSRF

Ознакомьтесь со шпаргалкой [Предотвращение Cross-Site Request Forgery (CSRF)](Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.md).

#### Защищайте от перехвата JSON для старых браузеров
##### Ознакомьтесь с механизмом защиты от перехвата JSON в AngularJS

Ссылка на требуемую секцию документации AngularJS [JSON Vulnerability Protection](https://docs.angularjs.org/api/ng/service/$http#json-vulnerability-protection).

##### Всегда возвращайте JSON с объектом снаружи

Всегда используйте объект в качестве внешнего примитива для JSON-строк:

**Уязвимо:**

```json
[{"object": "inside an array"}]
```

**Не уязвимо:**

```json
{"object": "not inside an array"}
```

**Также не уязвимо:**

```json
{"result": [{"object": "inside an array"}]}
```

#### Избегайте написания кода сериализации на стороне сервера

Помните о ссылочных и значимых типах! Ищите существующую библиотеку, которая была проверена.

Сервисы могут вызываться пользователями напрямую
Даже если вы ожидаете, что только клиентский AJAX-код будет вызывать эти сервисы, пользователи тоже могут это делать.

Убедитесь, что вы проверяете входные данные и рассматриваете их так, как если бы они находились под контролем пользователя (потому что так оно и есть!).

#### Избегайте ручного построения XML или JSON, используйте фреймворк

Используйте фреймворк, и будьте в безопасности. Ручное построение приведёт к проблемам с безопасностью.

#### Используйте схемы JSON и XML для веб-сервисов

Необходимо использовать стороннюю библиотеку для валидации веб-сервисов.
