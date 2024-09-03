# Шпаргалка по злоупотреблению

## Введение

Часто, при указании уровня безопасности приложения, в требованиях можно встретить следующие пункты:

- *Приложение должно быть безопасным*.
- *Приложение должно быть защищено от всех атак, нацеленных на эту категорию приложений*.
- *Приложение должно быть защищено от атак из списка OWASP TOP 10*.
- ...

Эти требования слишком общие, и, следовательно, бесполезны для команды разработки.

Чтобы создать безопасное приложение с практической точки зрения, важно идентфицировать атаки, от которых приложение должно защищаться, в зависимости от его бизнес- и технического контекста.

### Цель

Цель этой шпаргалки - предоставить объяснение того, что такое **Злоупотребление**, почему злоупотребления важны при рассмотрении безопасности приложения, и, наконец, предложить прагматический подход к созданию списка злоупотреблений и отслеживанию их для каждой функции, планируемой для реализации в приложении. Данную шпаргалку можно использовать для этой цели независимо от используемой методики проекта (водопад или гибкая).

**Важное примечание:**

```text
Основная цель этой шпаргалки - предоставить прагматический подход, чтобы позволить компании или команде проекта начать создавать и обрабатывать список злоупотреблений, а затем адаптировать предложенные элементы к своему контексту/культуре с целью создания собственной методики.

Эту шпаргалку можно рассматривать как руководство по началу работы.
```

### Контекст и подход

#### Почему важно чётко идентифицировать атаки

Чёткая идентификация атак, отк которых приложение должно защищаться, является важной составляющей для выполнения следующих шагов в проекте или спринте:

- Оценить бизнес-риски для каждой из идентифицированных атак, чтобы выполнить отбор в зависимости от бизнес-риска и бюджета проекта/спринта.
- Вывести требования безопасности и добавить их в спецификацию проекта или пользовательские истории и критерии приёмки спринта.
- Оценить дополнительные затраты на первоначальном этапе проекта/спринта, которые будут необходимы для реализации контрмер.
- Что касается контрмер: позволить команде проекта определить их и установить, в каком месте они должны быть расположены (сеть, инфраструктура, код и т.д.).

#### Понятие злоупотребления

Чтобы помочь в создании списк атак, полезна концепция **Злоупотреблений**.

**Злоупотребление** можно определить как:

```text
Способ использования функции, который не был ожидаем разработчиком, что позволяет злоумышленнику влиять на функциональность или результат использования функции на основе действий (или ввода) злоумышленника.
```

Synopsys определяют **Злоупотребление** следующим образом:

```text
Злоупотребления и случаи неправильного использования описывают, как пользователи злоупотребляют
или эксплуатируют слабости в контролях программных функций для атаки на приложение.

Это может привести к реальному бизнес-импакту, когда напрямую атакуются бизнес-функциональности, что может привести к потерям или негативному пользовательскому опыту.

Случаи злоупотребления также могут быть эффективным способом формирования требований безопасности, которые приведут к надлежащей защите этих критически важных бизнес-кейсов.
```

[Источник Synopsys](https://www.synopsys.com/blogs/software-security/abuse-cases-can-drive-security-requirements.html)

#### Как определить список злоупотреблений

Существует много различных способов определить список злоупотреблений для функции (которая может быть сопоставлена с пользовательской историей в Agile-проектах)

Проект [OWASP Open SAMM](https://owasp.org/www-project-samm/) предлагает следующий подход в *Потоке B* к практикам безопасности *тестирования основанного на требованиях* для уровня зрелости 2:

```text
Случаи неправильного использования и злоупотребления описывают непреднамеренные и злонамеренные сценарии использования приложения, описывая, как злоумышленник может это сделать. Создайте случаи неправильного использования и злоупотребления для использования или эксплуатации слабостей в контролях функций программного обеспечения для атаки на приложение. Используйте модели злоупотреблений для приложения, чтобы служить источником для идентификации конкретных тестов безопасности, которые напрямую или косвенно эксплуатируют сценарии злоупотреблений.

Злоупотребление функциональностью, иногда называемое «атакой бизнес-логики», зависит от дизайна и реализации функций и особенностей приложения. Примером является использование процесса сброса пароля для перечисления учетных записей. В рамках тестирования бизнес-логики определите важные бизнес-правила для приложения и превратите их в эксперименты для проверки того, правильно ли приложение применяет бизнес-правило. Например, в приложении для торговли акциями, разрешено ли злоумышленнику начать сделку в начале дня и зафиксировать цену, держать транзакцию открытой до конца дня, а затем завершить продажу, если цена акции возросла, или отменить, если цена упала?
```

Источник Open SAMM: [Verification Requirements Driven Testing Stream B](https://owaspsamm.org/model/verification/requirements-driven-testing/stream-b/)

Еще один способ создания списка может быть следующим (более ориентированным на подход «снизу вверх» и сотрудничество):

Проведите воркшоп с участием людей со следующими профилями:

- **Бизнес-аналитики**: Ключевые сотрудники, которые будут описывать каждую функцию с точки зрения бизнеса.
- **Аналитики по рискам**: Сотрудники компании, которые будут оценивать бизнес-риски предлагаемой атаки (иногда это роль выполняет бизнес-аналитик, в зависимости от компании).
- **Тестировщики на проникновение (пентестеры)**: "Атакующие", которые будут предлагать атаки, которые можно осуществить на рассматриваемые бизнес-функции. Если в компании нет сотрудника с таким профилем, можно запросить услуги внешних специалистов. При возможности включите двух пентестеров с разным опытом, чтобы увеличить количество потенциальных атак, которые будут выявлены и учтены.
- **Технические лидеры проектов**: Технические специалисты проекта, которые обеспечат технический обмен мнениями по поводу атак и контрмер, выявленных в ходе воркшопа
- **Аналитики по обеспечению качества или функциональный тестировщик**: Сотрудники, которые могут иметь хорошее представление о том, как приложение или функция должны работать (позитивное тестирование), как они не должны работать (негативное тестирование), и что вызывает их сбои (случаи отказов).

Во время воркшопа (продолжительность зависит от объема списка функций, но 4 часа — хорошее начало) будут обработаны все бизнес-функции, которые будут включены в проект или спринт. Результатом воркшопа будет список атак (случаев злоупотребления) для всех бизнес-функций. Каждому случаю злоупотребления будет присвоен уровень риска, который позволит фильтровать и расставлять приоритеты.

Важно учитывать как **технические**, так и **бизнесовые** случаи злоупотребления и соответственно их помечать.

*Пример:*

- Технический случай злоупотребления: Добавление Cross-Site-Scripting (XSS) инъекции в поле ввода комментария..
- Бизнесовый случай злоупотребления: Возможность произвольного изменения цены товара в интернет-магазине перед оформлением заказа, что приводит к тому, что пользователь платит меньшую сумму за желаемый товар.

#### Когда определять списокк случаев злоупотребления

В проектах, реализуемых по методологии Agile, воркшоп по определению должен проводиться после собрания, на котором пользовательские истории (User Stories) включаются в спринт.

В проектах, реализуемых по каскадной модели (Waterfall), воркшоп по определению должен проводиться, когда бизнес-функции, которые будут реализованы, уже определены и известны бизнесу.

Независимо от используемого подхода к проекту (Agile или Waterfall), случаи злоупотребления, которые выбраны для рассмотрения, должны стать требованиями безопасности в каждом разделе спецификации функции (Waterfall) или критериями приемки пользовательской истории (Agile), чтобы позволить оценить дополнительные затраты/усилия, а также идентифицировать и внедрить контрмеры.

Каждому случаю злоупотребления должен быть присвоен уникальный идентификатор, чтобы обеспечить его отслеживание на протяжении всего проекта/спринта (подробности по этому пункту будут приведены в разделе предложения).

Пример формата уникального идентификатора может быть **ABUSE_CASE_001**.

На следующем рисунке показан обзор последовательности различных шагов, задействованных в процессе (слева направо):

![Обзорная схема](../assets/Abuse_Case_Cheat_Sheet_Overview.png)

### Предложение

Предложение будет сосредоточено на результатах воркшопа, объясненных в предыдущем разделе.

#### Шаг 1: Подготовка к воркшопу

Во-первых, даже если это кажется очевидным, ключевые бизнес-сотрудники должны быть уверены, что они знают, понимают и могут объяснить бизнес-функции, которые будут обрабатываться во время воркшопа.

Во-вторых, создайте новый файл Microsoft Excel (вы также можете использовать Google Sheets или любое другое аналогичное программное обеспечение) со следующими листами (или вкладками):

- **ФУНКЦИИ (FEATURES)**
    - Будет содержать таблицу со списком бизнес-функций, запланированных для воркшопа.
- **СЛУЧАИ ЗЛОУПОТРЕБЛЕНИЯ (ABUSE CASES)**
    - Будет содержать таблицу со всеми случаями злоупотребления, выявленными в ходе воркшопа.
- **КОНТРМЕРЫ (COUNTERMEASURES)**
    - Будет содержать таблицу со списком возможных контрмер (краткое описание), придуманных для выявленных случаев злоупотребления.
    - Этот лист не обязателен, но может быть полезным (для каждого случая злоупотребления), если устранение проблемы легко реализуемо и может повлиять на рейтинг риска.
    - Контрмеры могут быть определены профилем AppSec (специалистом по безопасности приложений) в ходе воркшопа, потому что сотрудник AppSec должен уметь не только совершать атаки, но и строить или определять меры защиты (это не всегда характерно для профиля пентестера, поскольку его основное внимание сосредоточено на атакующей стороне, поэтому комбинация пентестер + AppSec очень эффективна для получения всестороннего обзора).

Ниже представлено изображение каждого листа вместе с примером содержания, которое будет заполняться во время воркшопа:

Лист *ФУНКЦИИ*:

| Идентификатор функции | Имя функции |           Краткое описание функции           |
|:-----------------:|:---------------------:|:---------------------------------------------:|
| FEATURE_001       | DocumentUploadFeature | Позволяет пользователю загрузить документ вместе с сообщением |

*COUNTERMEASURES* sheet:

| Countermeasure unique ID | Countermeasure short description                       | Countermeasure help/hint                                |
|--------------------------|--------------------------------------------------------|---------------------------------------------------------|
| DEFENSE_001              | Validate the uploaded file by loading it into a parser | Use advice from the OWASP Cheat Sheet about file upload |

*ABUSE CASES* sheet:

| Abuse case unique ID | Feature ID impacted |                     Abuse case's attack description                     | Attack referential ID (if applicable) | CVSS V3 risk rating (score) |                CVSS V3 string                | Kind of abuse case | Countermeasure ID applicable | Handling decision (To Address or Risk Accepted) |
|:--------------------:|:-------------------:|:-----------------------------------------------------------------------:|:-------------------------------------:|:---------------------------:|:--------------------------------------------:|:------------------:|:----------------------------:|:-----------------------------------------------:|
| ABUSE_CASE_001       | FEATURE_001         | Upload Office file with malicious macro in charge of dropping a malware | CAPEC-17                              | HIGH (7.7)                  | CVSS:3.0/AV:N/AC:H/PR:L/UI:R/S:C/C:N/I:H/A:H | Technical          | DEFENSE_001                  | To Address                                      |

#### Step 2: During the workshop

Use the spreadsheet to review all the features.

For each feature, follow this flow:

1. Key business people explain the current feature from a business point of view.
2. Penetration testers propose and explain a set of attacks that they can perform against the feature.
3. For each attack proposed:
    1. Appsec proposes a countermeasure and a preferred set up location (infrastructure, network, code, design...).
    2. Technical people give feedback about the feasibility of the proposed countermeasure.
    3. Penetration testers use the CVSS v3 (or other standard) calculator to determine a risk rating. (ex: [CVSS V3 calculator](https://www.first.org/cvss/calculator/3.0))
    4. Risk leaders should accept or modify the risk rating to determine the final risk score which accurately reflects the real business impact for the company.

4. Business, Risk, and Technical leaders should find a consensus and filter the list of abuses for the current feature to keep the ones that must be addressed, and then flag them accordingly in the *ABUSE CASES* sheet (**if risk is accepted then add a comment to explain why**).
5. Pass to next feature...

If the presence of penetration testers is not possible then you can use the following references to identify the applicable attacks on your features:

- [OWASP Automated Threats to Web Applications](https://owasp.org/www-project-automated-threats-to-web-applications/)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/stable/)
- [OWASP Mobile Testing Guide](https://github.com/OWASP/owasp-mstg)
- [Common Attack Pattern Enumeration and Classification (CAPEC)](https://capec.mitre.org/)

Important note on attacks and countermeasure knowledge base(s):

```text
With time and experience across projects, you will obtain your own dictionary of attacks and countermeasures
that are applicable to the kind of application in your business domain.

This dictionary will speed up the future workshops in a significant way.

To promote the creation of this dictionary, you can, at the end of the project/sprint, gather the list
of attacks and countermeasures identified in a central location (wiki, database, file...) that will be
used during the next workshop in combination with input from penetration testers.
```

#### Step 3: After the workshop

The spreadsheet contains (at this stage) the list of all abuse cases that must be handled and, potentially (depending on the capacity) corresponding countermeasures.

Now, there are two remaining task:

1. Key business people must update the specification of each feature (waterfall) or the User Story of each feature (agile) to include the associated abuse cases as Security Requirements (waterfall) or Acceptance Criteria (agile).
2. Key technical people must evaluate the overhead in terms of expense/effort to take into account the countermeasure.

#### Step 4: During implementation - Abuse cases handling tracking

In order to track the handling of all the abuse cases, the following approach can be used:

If one or several abuse cases are handled at:

- **Design, Infrastructure or Network level**
    - Make a note in the documentation or schema to indicate that *This design/network/infrastructure takes into account the abuse cases ABUSE_CASE_001, ABUSE_CASE_002, ABUSE_CASE_xxx*.
- **Code level**
    - Put a special comment in the classes/scripts/modules to indicate that *This class/module/script takes into account the abuse cases ABUSE_CASE_001, ABUSE_CASE_002, ABUSE_CASE_xxx*.
    - Dedicated annotation like `@AbuseCase(ids={"ABUSE_CASE_001","ABUSE_CASE_002"})` can be used to facilitate tracking and allow identification into integrated development environment.

Using this way, it becomes possible (via some minor scripting) to identify where abuse cases are addressed.

#### Step 5: During implementation - Abuse cases handling validation

As abuse cases are defined, it is possible to put in place automated or manual validations to ensure that:

- All the selected abuse cases are handled.
- An abuse case is correctly/completely handled.

Validations can be of the following varieties:

- Automated (run regularly at commit, daily or weekly in the Continuous Integration Jobs of the project):
    - Custom audit rules in Static Application Security Testing (SAST) or Dynamic Application Security Testing (DAST) tools.
    - Dedicated unit, integration or functional security oriented tests.
    - ...
- Manual:
    - Security code review between project's peers during the design or implementation.
    - Provide the list of all abuse cases addressed to pentesters so that they may validate the protection efficiency for each abuse case during an intrusion test against the application (the pentester will validate that the attacks identified are no longer effective and will also try to find other possible attacks).
    - ...

Adding automated tests also allow teams to track the effectiveness of countermeasures against abuse cases and determine if the countermeasures are still in place during a maintenance or bug fixing phase of a project (to prevent accidental removal/disabling). It is also useful when a [Continuous Delivery](https://continuousdelivery.com/) approach is used, to ensure that all abuse cases protections are in place before opening access to the application.

### Example of derivation of Abuse Cases as User Stories

The following section shows an example of derivation of Abuse Cases as User Stories, here using the [OWASP TOP 10](https://owasp.org/www-project-top-ten/) as input source.

Threat Oriented Personas:

- Malicious User
- Abusive User
- Unknowing User

#### A1:2017-Injection

*Epic:*

Almost any source of data can be an injection vector, environment variables, parameters, external and internal web services, and all types of users. [Injection](https://owasp.org/www-community/Injection_Flaws) flaws occur when an attacker can send hostile data to an interpreter.

*Abuse Case:*

As an attacker, I will perform an injection attack (SQL, LDAP, XPath, or NoSQL queries, OS commands, XML parsers, SMTP headers, expression languages, and ORM queries) against input fields of the User or API interfaces

#### A2:2017-Broken Authentication

*Epic:*

Attackers have access to hundreds of millions of valid username and password combinations for credential stuffing, default administrative account lists, automated brute force, and dictionary attack tools. Session management attacks are well understood, particularly in relation to unexpired session tokens.

*Abuse Case:*

As an attacker, I have access to hundreds of millions of valid username and password combinations for credential stuffing.

*Abuse Case:*

As an attacker, I have default administrative account lists, automated brute force, and dictionary attack tools I use against login areas of the application and support systems.

*Abuse Case:*

As an attacker, I manipulate session tokens using expired and fake tokens to gain access.

#### A3:2017-Sensitive Data Exposure

*Epic:*

Rather than directly attacking crypto, attackers steal keys, execute man-in-the-middle attacks, or steal clear text data off the server, while in transit, or from the user's client, e.g. browser. A manual attack is generally required. Previously retrieved password databases could be brute forced by Graphics Processing Units (GPUs).

*Abuse Case:*

As an attacker, I steal keys that were exposed in the application to get unauthorized access to the application or system.

*Abuse Case:*

As an attacker, I execute man-in-the-middle attacks to get access to traffic and leverage it to obtain sensitive data and possibly get unauthorized access to the application.

*Abuse Case:*

As an attacker, I steal clear text data off the server, while in transit, or from the user's client, e.g. browser to get unauthorized access to the application or system.

*Abuse Case:*

As an attacker, I find and target old or weak cryptographic algorithms by capturing traffic and breaking the encryption.

#### A4:2017-XML External Entities (XXE)

*Epic:*

Attackers can exploit vulnerable XML processors if they can upload XML or include hostile content in an XML document, exploiting vulnerable code, dependencies or integrations.

*Abuse Case:*

As an attacker, I exploit vulnerable areas of the application where the user or system can upload XML to extract data, execute a remote request from the server, scan internal systems, perform a denial-of-service attack, as well as execute other attacks.

*Abuse Case:*

As an attacker, I include hostile content in an XML document which is uploaded to the application or system to extract data, execute a remote request from the server, scan internal systems, perform a denial-of-service attack, as well as execute other attacks.

*Abuse Case:*

As an attacker, I include malicious XML code to exploit vulnerable code, dependencies or integrations to extract data, execute a remote request from the server, scan internal systems, perform a denial-of-service attack (e.g. Billion Laughs attack), as well as execute other attacks.

#### A5:2017-Broken Access Control

*Epic:*

Exploitation of access control is a core skill of attackers. Access control is detectable using manual means, or possibly through automation for the absence of access controls in certain frameworks.

*Abuse Case:*

As an attacker, I bypass access control checks by modifying the URL, internal application state, or the HTML page, or simply using a custom API attack tool.

*Abuse Case:*

As an attacker, I manipulate the primary key and change it to access another's users record, allowing viewing or editing someone else's account.

*Abuse Case:*

As an attacker, I manipulate sessions, access tokens, or other access controls in the application to act as a user without being logged in, or acting as an admin/privileged user when logged in as a user.

*Abuse Case:*

As an attacker, I leverage metadata manipulation, such as replaying or tampering with a JSON Web Token (JWT) access control token or a cookie or hidden field manipulated to elevate privileges or abusing JWT invalidation.

*Abuse Case:*

As an attacker, I exploit Cross-Origin Resource Sharing CORS misconfiguration allowing unauthorized API access.

*Abuse Case:*

As an attacker, I force browsing to authenticated pages as an unauthenticated user or to privileged pages as a standard user.

*Abuse Case:*

As an attacker, I access APIs with missing access controls for POST, PUT and DELETE.

*Abuse Case:*

As an attacker, I target default crypto keys in use, weak crypto keys generated or re-used, or keys where rotation is missing.

*Abuse Case:*

As an attacker, I find areas where the user agent (e.g. app, mail client) does not verify if the received server certificate is valid and perform attacks where I get unauthorized access to data.

#### A6:2017-Security Misconfiguration

*Epic:*

Attackers will often attempt to exploit unpatched flaws or access default accounts, unused pages, unprotected files and directories, etc to gain unauthorized access or knowledge of the system.

*Abuse Case:*

As an attacker, I find and exploit missing appropriate security hardening configurations on any part of the application stack, or improperly configured permissions on cloud services.

*Abuse Case:*

As an attacker, I find unnecessary features which are enabled or installed (e.g. unnecessary ports, services, pages, accounts, or privileges) and attack or exploit the weakness.

*Abuse Case:*

As an attacker, I use default accounts and their passwords to access systems, interfaces, or perform actions on components which I should not be able to.

*Abuse Case:*

As an attacker, I find areas of the application where error handling reveals stack traces or other overly informative error messages I can use for further exploitation.

*Abuse Case:*

As an attacker, I find areas where upgraded systems, latest security features are disabled or not configured securely.

*Abuse Case:*

As an attacker, I find security settings in the application servers, application frameworks (e.g. Struts, Spring, ASP.NET), libraries, databases, etc. not set to secure values.

*Abuse Case:*

As an attacker, I find the server does not send security headers or directives or are set to insecure values.

#### A7:2017-Cross-Site Scripting (XSS)

*Epic:*

XSS is the second most prevalent issue in the OWASP Top 10, and is found in around two-thirds of all applications.

*Abuse Case:*

As an attacker, I perform reflected XSS where the application or API includes unvalidated and unescaped user input as part of HTML output. My successful attack can allow the attacker to execute arbitrary HTML and JavaScript in my victim's browser. Typically the victim will need to interact with some malicious link that points to an attacker-controlled page, such as malicious watering hole websites, advertisements, or similar.

*Abuse Case:*

As an attacker, I perform stored XSS where the application or API stores unsanitized user input that is viewed at a later time by another user or an administrator.

*Abuse Case:*

As an attacker, I perform DOM XSS where JavaScript frameworks, single-page applications, and APIs that dynamically include attacker-controllable data to a page is vulnerable to DOM XSS.

#### A8:2017-Insecure Deserialization

*Epic:*

Exploitation of deserialization is somewhat difficult, as off-the-shelf exploits rarely work without changes or tweaks to the underlying exploit code.

*Abuse Case:*

As an attacker, I find areas of the application and APIs where deserialization of hostile or tampered objects can be supplied. As a result, I can focus on an object and data structure related attacks where the attacker modifies application logic or achieves arbitrary remote code execution if there are classes available to the application that can change behavior during or after deserialization. Or I focus on data tampering attacks such as access-control-related attacks where existing data structures are used but the content is changed.

#### A9:2017-Using Components with Known Vulnerabilities

*Epic:*

While it is easy to find already-written exploits for many known vulnerabilities, other vulnerabilities require concentrated effort to develop a custom exploit.

*Abuse Case:*

As an attacker, I find common open source or closed source packages with weaknesses and perform attacks against vulnerabilities and exploits which are disclosed

#### A10:2017-Insufficient Logging & Monitoring

*Epic:*

Exploitation of insufficient logging and monitoring is the bedrock of nearly every major incident. Attackers rely on the lack of monitoring and timely response to achieve their goals without being detected. In 2016, identifying a breach took an [average of 191 days](https://www-01.ibm.com/common/ssi/cgi-bin/ssialias?htmlfid=SEL03130WWEN) allowing substancial chance for damage to be inflicted.

*Abuse Case:*

As an attacker, I attack an organization and the logs, monitoring systems, and teams do not see or respond to my attacks.

## Sources of the schemas

All figures were created using <https://www.draw.io/> site and exported (as PNG image) for integration into this article.

All XML descriptor files for each schema are available below (using XML description, modification of the schema is possible using DRAW.IO site):

[Schemas descriptors archive](../assets/Abuse_Case_Cheat_Sheet_SchemaBundle.zip)
