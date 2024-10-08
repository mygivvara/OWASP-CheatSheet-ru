# Шпаргалка по безопасности микросервисов

## Вступление

Архитектура микросервисов все чаще используется для проектирования и внедрения прикладных систем как в облачных, так и в локальных инфраструктурах, крупномасштабных приложений и сервисов. Существует множество проблем безопасности, которые необходимо решать на этапах разработки и внедрения приложений. Основными требованиями безопасности, которые необходимо учитывать на этапе проектирования, являются аутентификация и авторизация. Поэтому архитекторам безопасности приложений жизненно важно понимать и правильно использовать существующие архитектурные шаблоны для реализации аутентификации и авторизации в системах, основанных на микросервисах. Цель этой шпаргалки - определить такие шаблоны и дать рекомендации архитекторам безопасности приложений по возможным способам их использования.

## Авторизация на пограничном уровне

В простых сценариях авторизация может выполняться только на пограничном уровне (API gateway). API gateway можно использовать для централизованного обеспечения авторизации для всех нижестоящих микросервисов, устраняя необходимость в обеспечении аутентификации и контроля доступа для каждой отдельной службы. В таких случаях NIST рекомендует внедрять смягчающие меры контроля, такие как взаимная аутентификация, для предотвращения прямых анонимных подключений к внутренним службам (обход шлюза API). Следует отметить, что авторизация на пограничном уровне имеет [следующие ограничения](https://www.youtube.com/watch?v=UnXjwCWgBKU):

- Передача всех решений об авторизации на шлюз API может быстро усложнить управление в сложных экосистемах со множеством ролей и правил контроля доступа.
- Шлюз API может стать единственной точкой принятия решений, что может нарушить принцип “глубокой защиты”.
- Обычно шлюз API принадлежит операционным командам, поэтому команды разработчиков не могут напрямую вносить изменения в авторизацию, что замедляет скорость из-за дополнительных затрат на связь и процессы.
  
В большинстве случаев команды разработчиков реализуют авторизацию в обоих местах – на пограничном уровне, на грубом уровне детализации, и на уровне сервиса. Для аутентификации внешнего объекта edge может использовать токены доступа (ссылочный токен или автономный токен), передаваемые через HTTP-заголовки (например, “Cookie” или “Авторизация”), или использовать MTLS.

## Авторизация на уровне сервиса

Авторизация на уровне сервиса предоставляет каждому микросервису больше возможностей для применения политик контроля доступа.
Для дальнейшего обсуждения мы будем использовать термины и определения в соответствии с [NIST SP 800-162](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-162.pdf). Функциональные компоненты системы контроля доступа можно классифицировать следующим образом:

- Точка администрирования политик (PAP): Предоставляет пользовательский интерфейс для создания, управления, тестирования и отладки правил контроля доступа.
- Точка принятия политических решений (PDP): Компьютеры принимают решения о доступе, оценивая применимую политику контроля доступа.
- Точка применения политики (PEP): Обеспечивает принятие политических решений в ответ на запрос субъекта, запрашивающего доступ к защищенному объекту.
- Центр информации о политике (PIP): служит источником поиска атрибутов или данных, необходимых для оценки политики, для предоставления информации, необходимой PDP для принятия решений.

![NIST ABAC framework](../assets/NIST_ABAC.png)

### Авторизация на уровне сервиса: существующие шаблоны

#### Децентрализованный шаблон

Команда разработчиков реализует PDP и PEP непосредственно на уровне кода микросервиса. Все правила контроля доступа и атрибуты, необходимые для реализации этого правила, определены и хранятся в каждом микросервисе (шаг 1). Когда микросервис получает запрос вместе с некоторыми метаданными авторизации (например, контекст конечного пользователя или идентификатор запрашиваемого ресурса), микросервис анализирует его (шаг 3) для выработки решения о политике контроля доступа, а затем применяет авторизацию (шаг 4).
![Децентрализованный шаблон HLD](../assets/Dec_pattern_HLD.png)

Существующие языковые платформы программирования позволяют командам разработчиков реализовывать авторизацию на уровне микросервисов. Например, [Spring Security позволяет](https://www.youtube.com/watch?v=v2J32nd0g24) разработчикам включать проверку областей (например, с использованием областей, извлеченных из входящего JWT) на сервере ресурсов и использовать ее для принудительной авторизации.

Реализация авторизации на уровне исходного кода означает, что код должен обновляться всякий раз, когда команда разработчиков хочет изменить логику авторизации.

#### Централизованный шаблон с единой точкой принятия политических решений

В этом шаблоне правила контроля доступа определяются, хранятся и оцениваются централизованно. Правила контроля доступа определяются с помощью PAP (шаг 1) и передаются в централизованный PDP вместе с атрибутами, необходимыми для оценки этих правил (шаг 2). Когда субъект вызывает конечную точку микросервиса (шаг 3), код микросервиса вызывает централизованный PDP посредством сетевого вызова, и PDP генерирует решение о политике контроля доступа, оценивая входные данные запроса в соответствии с правилами и атрибутами контроля доступа (шаг 4). На основании решения PDP микросервис выполняет принудительную авторизацию (шаг 5).

![Централизованная модель с встроенной точкой принятия политик (HLD](../assets/Single_PDP_HLD.png)

Чтобы определить правила контроля доступа, команды разработчиков/операторов должны использовать какой-либо язык или нотацию. Примером может служить расширяемый язык разметки контроля доступа (XACML) и язык управления доступом следующего поколения (NAC), который является стандартом для описания правил политики.

Этот шаблон может вызвать проблемы с задержкой из-за дополнительных сетевых вызовов удаленной конечной точки PDP, но его можно устранить, кэшируя решения политики авторизации на уровне микросервисов. Следует отметить, что PDP должен работать в режиме высокой доступности, чтобы предотвратить проблемы с отказоустойчивостью и доступностью. Архитекторы безопасности приложений должны сочетать его с другими шаблонами (например, авторизацией на уровне шлюза API), чтобы обеспечить соблюдение принципа "глубокой защиты".

#### Централизованный шаблон со встроенной точкой принятия политических решений

В этом шаблоне правила контроля доступа определяются централизованно, но сохраняются и оцениваются на уровне микросервисов. Правила контроля доступа определяются с помощью PAP (шаг 1) и передаются во встроенный PDP вместе с атрибутами, необходимыми для оценки этих правил (шаг 2). Когда субъект вызывает конечную точку микросервиса (шаг 3), код микросервиса вызывает PDP, и PDP генерирует решение о политике контроля доступа, оценивая входные данные запроса в соответствии с правилами и атрибутами контроля доступа (шаг 4). На основании решения PDP микросервис выполняет принудительную авторизацию (шаг 5).

![Централизованная модель с встроенной точкой принятия политик (HLD](../assets/Embed_PDP_HLD.png)

PDP-код в этом случае может быть реализован в виде встроенной библиотеки микросервисов или sidecar в архитектуре service mesh. Из-за возможных сбоев в работе сети/хоста и задержек в сети рекомендуется реализовать встроенный PDP в виде библиотеки микросервисов или вспомогательной программы на том же хосте, что и микросервис. Встроенный PDP обычно хранит политику авторизации и связанные с ней данные в памяти, чтобы минимизировать внешние зависимости во время принудительной авторизации и добиться низкой задержки. Основное отличие от подхода “Централизованный шаблон с единой точкой принятия политических решений” заключается в том, что решения об авторизации не хранятся на стороне микросервиса, вместо этого на стороне микросервиса хранится актуальная политика авторизации. Следует отметить, что решения об авторизации при кэшировании могут привести к применению устаревших правил авторизации и нарушениям контроля доступа.

Netflix presented ([link](https://www.youtube.com/watch?v=R6tUNpRpdnY), [link](https://conferences.oreilly.com/velocity/vl-ca-2018/public/schedule/detail/66606.html)) a real case of using “Centralized pattern with embedded PDP” pattern to implement authorization on the microservices level.

![Централизованная модель с встроенной точкой принятия политик (HLD](../assets/Netflix_AC.png)

- Портал политик и хранилище политик - это системы на основе пользовательского интерфейса для создания правил контроля доступа, управления ими и контроля версий.
- Агрегатор извлекает данные, используемые в правилах контроля доступа, из всех внешних источников и поддерживает их в актуальном состоянии.
- Распространитель извлекает правила контроля доступа (из хранилища политик) и данные, используемые в правилах контроля доступа (из агрегаторов), для распространения их среди PDP.
- PDP (библиотека) асинхронно извлекает правила и данные контроля доступа и поддерживает их в актуальном состоянии для обеспечения авторизации компонентом PEP.

### Рекомендации по осуществлению авторизации

1. Для достижения масштабируемости не рекомендуется жестко прописывать политику авторизации в исходном коде (децентрализованный шаблон), а вместо этого использовать специальный язык для выражения политики. Цель состоит в том, чтобы вывести авторизацию из кода, а не просто использовать шлюз/прокси-сервер в качестве контрольной точки. Рекомендуемым шаблоном для авторизации на уровне сервиса является "Централизованный шаблон со встроенным PDP" из-за его гибкости и широкого внедрения.
2. Решение для авторизации должно быть решением на уровне платформы; специальная команда (например, команда безопасности платформы) должна отвечать за разработку и эксплуатацию решения для авторизации, а также за совместное использование схемы микросервисов/библиотеки/компонентов, которые реализуют авторизацию, среди групп разработчиков.
3. Решение для авторизации должно основываться на широко используемых решениях, поскольку внедрение пользовательского решения имеет следующие недостатки:
    - Команды безопасности или инженеров должны создавать и поддерживать пользовательское решение.
    - Необходимо создавать и поддерживать клиентские библиотеки SDK для каждого языка, используемого в архитектуре системы.
    - Необходимо обучать каждого разработчика пользовательскому API службы авторизации и интеграции, и нет сообщества с открытым исходным кодом, из которого можно было бы получать информацию.
4. Существует вероятность того, что не все политики контроля доступа могут быть реализованы шлюзами/прокси-серверами и библиотекой/компонентами общей авторизации, поэтому некоторые специфические правила контроля доступа все еще должны быть реализованы на уровне бизнес-кода микросервиса. Для этого рекомендуется, чтобы команды разработчиков микросервисов использовали простые анкеты/контрольные списки для выявления таких требований к безопасности и надлежащего их учета в ходе разработки микросервисов.
5. Рекомендуется внедрить принцип “глубокой защиты” и обеспечить авторизацию на:
    - Уровень шлюза и прокси-сервера с высокой степенью детализации.
    - Уровень микросервиса, использующий общую библиотеку авторизации / компоненты для обеспечения принятия точных решений.
    - Уровень бизнес-кода микросервиса для реализации правил контроля доступа, специфичных для бизнеса.
6. При разработке, утверждении и внедрении политики контроля доступа должны быть внедрены формальные процедуры.

## Распространение идентификационных данных внешнего объекта

Чтобы принимать детальные решения об авторизации на уровне микросервиса, микросервис должен понимать контекст вызывающей стороны (например, идентификатор пользователя, роли/группы пользователей). Чтобы позволить внутреннему сервисному уровню принудительно выполнять авторизацию, пограничный уровень должен распространять аутентифицированный внешний идентификатор объекта (например, контекст конечного пользователя) вместе с запросом к нижестоящим микросервисам. Одним из простейших способов распространения идентификатора внешнего объекта является повторное использование токена доступа, полученного edge, и передача его внутренним микросервисам. Однако следует отметить, что этот подход крайне небезопасен из-за возможной утечки токенов внешнего доступа и может увеличить вероятность атаки, поскольку обмен данными основан на собственной реализации системы, основанной на токенах. Если внутренняя служба непреднамеренно подключена к внешней сети, то к ней можно получить прямой доступ, используя утечку токена доступа. Эта атака невозможна, если внутренняя служба принимает только формат токена, известный только внутренним службам. Этот шаблон также не зависит от внешних токенов доступа, т.е. внутренние службы должны понимать внешние токены доступа и поддерживать широкий спектр методов аутентификации для извлечения идентификационных данных из различных типов внешних токенов (например, JWT, cookie, токен OpenID Connect).

### Распространение идентичности: существующие модели

#### Отправка идентификатора внешней сущности в виде открытых или самозаверяющих структур данных

При таком подходе микросервис извлекает идентификатор внешней сущности из входящего запроса (например, путем анализа входящего токена доступа), создает структуру данных (например, JSON или самозаверяющий JWT) с этим контекстом и передает ее внутреннему микросервису.
В этом случае микросервис-получатель должен доверять вызывающему микросервису. Если вызывающий микросервис хочет нарушить правила контроля доступа, он может сделать это, указав в HTTP-заголовке любой идентификатор пользователя/клиента или роли пользователей, которые он хочет. Такой подход подходит только в средах с высоким уровнем доверия, где каждый микросервис разрабатывается надежной командой разработчиков, применяющей методы безопасной разработки программного обеспечения.

#### Использование структуры данных, подписанной доверенным эмитентом

В этом шаблоне после того, как внешний запрос аутентифицируется службой аутентификации на пограничном уровне, структура данных, представляющая идентификатор внешнего объекта (например, содержащая идентификатор пользователя, роли/группы пользователей или разрешения), генерируется, подписывается или шифруется доверенным эмитентом и передается внутренним микросервисам.
![Распространение подписанного идентификатора](../assets/Signed_ID_propogation.png)

[Netflix представил](https://www.infoq.com/presentations/netflix-user-identity/) реальный пример использования этого шаблона: структура под названием “Passport”, которая содержит идентификатор пользователя и его атрибуты и которая защищена HMAC на пограничном уровне для каждого входящего запроса. Эта структура распространяется на внутренние микросервисы и никогда не выставляется наружу.

1. Служба аутентификации Edge Authentication Service (EAS) получает секретный ключ от системы управления ключами.
2. EAS получает токен доступа (например, в cookie, JWT, OAuth2-токен) из входящего запроса.
3. EAS расшифровывает маркер доступа, разрешает идентификатор внешней сущности и отправляет его внутренним сервисам в подписанной структуре “Passport”».
4. Внутренние сервисы могут извлекать идентификационные данные пользователя для обеспечения авторизации (например, для реализации авторизации на основе идентификационных данных) с помощью обёрток.
5. При необходимости внутренний сервис может распространить структуру “Passport” на последующие сервисы в цепочке вызовов.

![Netflix ID propagation approach](../assets/Netflix_ID_prop.png)
Следует отметить, что шаблон не зависит от внешнего токена доступа и позволяет отделять внешние объекты от их внутренних представлений.

### Рекомендации по внедрению распространения идентификационных данных

1. Чтобы реализовать независимую от внешних токенов доступа и расширяемую систему, отделите токены доступа, выданные для внешнего объекта, от его внутреннего представления. Используйте единую структуру данных для представления и распространения идентификационных данных внешнего объекта среди микросервисов. Служба пограничного уровня должна проверить входящий внешний токен доступа, создать внутреннюю структуру представления сущности и распространить ее на нижестоящие службы.
2. Использование внутренней структуры представления объектов, подписанной (симметричным или асимметричным шифрованием) доверенным эмитентом, является рекомендуемым шаблоном, принятым сообществом.
3. Внутренняя структура представления объектов должна быть расширяемой, чтобы можно было добавлять больше утверждений, что может привести к низкой задержке.
4. Внутренняя структура представления сущности не должна быть доступна извне (например, браузеру или внешнему устройству).

## Межсервисная аутентификация

### Существующие шаблоны

#### Взаимная защита на транспортном уровне

Благодаря подходу mTLS каждый микросервис может законным образом идентифицировать, с кем он общается, в дополнение к обеспечению конфиденциальности и целостности передаваемых данных. Каждый микросервис в развертывании должен содержать пару открытых/закрытых ключей и использовать эту пару ключей для аутентификации в микросервисах-получателях через mTLS. mTLS обычно реализуется с помощью автономной инфраструктуры открытых ключей. Основными проблемами, связанными с использованием MTLS, являются предоставление ключей и настройка доверия, отзыв сертификата и ротация ключей.

#### Основанный на токенах

Подход, основанный на токенах, работает на прикладном уровне. Токен - это контейнер, который может содержать идентификатор вызывающего абонента (идентификатор микросервиса) и его разрешения (области действия). Вызывающий микросервис может получить подписанный токен, вызвав специальную службу маркеров безопасности, используя свой собственный идентификатор службы и пароль, а затем прикрепить его к каждому исходящему запросу, например, через HTTP-заголовки. Вызываемый микросервис может извлечь токен и проверить его онлайн или офлайн.
![Signed ID propagation](../assets/Token_validation.png)

1. Интерактивный сценарий:
    - Для проверки входящих токенов микросервис вызывает централизованную службу служебных токенов посредством сетевого вызова.
    - Могут быть обнаружены отозванные (скомпрометированные) токены.
    - Высокая задержка.
    - Должны применяться к критическим запросам.
2. Автономный сценарий:
    - Для проверки входящих токенов микросервис использует загруженный открытый ключ сервиса service token.
    - Отозванные (скомпрометированные) токены могут быть не обнаружены.
    - Низкая задержка.
    - Следует применять к некритичным запросам.
В большинстве случаев аутентификация на основе токенов работает по протоколу TLS, что обеспечивает конфиденциальность и целостность передаваемых данных.

## Ведение журнала

Службы ведения журнала в системах, основанных на микросервисах, направлены на соблюдение принципов подотчетности и отслеживаемости и помогают выявлять аномалии безопасности в операциях с помощью анализа журналов. Поэтому для разработчиков систем безопасности приложений жизненно важно понимать и адекватно использовать существующие архитектурные шаблоны для реализации ведения журнала аудита в системах на основе микросервисов для обеспечения безопасности операций. На рисунке ниже показан проект высокоуровневой архитектуры, основанный на следующих принципах:

- Каждый микросервис записывает лог-сообщение в локальный файл, используя стандартный вывод (через stdout, stderr).
- Агент ведения журнала периодически извлекает лог-сообщения и отправляет (публикует) их посреднику сообщений (например, NATS, Apache Kafka).
- Центральная служба ведения журнала подписывается на сообщения в посреднике сообщений, получает их и обрабатывает.
![Logging pattern](../assets/ms_logging_pattern.png)

Ниже приведены рекомендации высокого уровня по архитектуре подсистемы ведения журнала с обоснованиями.

1. Микросервис не должен отправлять сообщения журнала непосредственно в центральную подсистему ведения журнала с использованием сетевого взаимодействия. Микросервис должен записывать свои сообщения журнала в локальный файл журнала:
    - это позволяет снизить угрозу потери данных из-за сбоя службы ведения журнала в результате атаки или в случае их затопления легитимным микросервисом
    - в случае отключения службы ведения журнала микросервис по-прежнему будет записывать сообщения журнала в локальный файл (без потери данных), и после восстановления службы ведения журнала журналы будут доступны для отправки;
2. Должен быть выделенный компонент (агент ведения журнала), отделенный от микросервиса. Агент ведения журнала должен собирать данные журнала в микросервисе (считывать локальный файл журнала) и отправлять их в центральную подсистему ведения журнала. Из-за возможных проблем с задержкой в сети агент ведения журнала должен быть развернут на том же хосте (виртуальной или физической машине), что и микросервис:
    - это позволяет снизить угрозу потери данных из-за сбоя службы ведения журнала в результате атаки или в случае его затопления легитимным микросервисом
    - в случае сбоя агента ведения журнала микросервис все равно записывает информацию в файл журнала, агент ведения журнала после восстановления прочитает файл и отправит информацию посреднику сообщений;
3. При возможной DoS-атаке на центральную подсистему ведения журнала агент ведения журнала не должен использовать асинхронный шаблон запроса/ответа для отправки сообщений журнала. Должен быть брокер сообщений для реализации асинхронного соединения между агентом ведения журнала и центральной службой ведения журнала:
    - это позволяет снизить угрозу потери данных из-за сбоя службы ведения журнала в случае их затопления легитимным микросервисом
    - в случае отключения службы ведения журнала микросервис по-прежнему будет записывать сообщения журнала в локальный файл (без потери данных), а после восстановления службы ведения журнала журналы будут доступны для отправки;
4. Агент ведения журнала и посредник обмена сообщениями должны использовать взаимную аутентификацию (например, на основе TLS) для шифрования всех передаваемых данных (сообщений журнала) и аутентификации самих себя:
    - это позволяет смягчать такие угрозы, как: подмена микросервиса, ведение журнала/подмена транспортной системы, внедрение сетевого трафика, перехват сетевого трафика
5. Посредник сообщений должен применять политику контроля доступа для предотвращения несанкционированного доступа и реализации принципа наименьших привилегий:
    - это позволяет снизить угрозу повышения привилегий микросервисом
6. Агент ведения журнала должен фильтровать/очищать выходные сообщения журнала, чтобы гарантировать, что конфиденциальные данные (например, личные данные, пароли, API-ключи) никогда не будут отправлены в центральную подсистему ведения журнала (принцип минимизации данных). Для получения подробного обзора элементов, которые следует исключить из ведения журнала, пожалуйста, ознакомьтесь с [OWASP Logging Cheat Sheet](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Logging_Cheat_Sheet.md#data-to-exclude).
7. Микросервисы должны генерировать идентификатор корреляции, который однозначно идентифицирует каждую цепочку вызовов и помогает группировать сообщения журнала для их проверки. Агент ведения журнала должен включать идентификатор корреляции в каждое сообщение журнала.
8. Агент ведения журнала должен периодически предоставлять данные о работоспособности и статусе, чтобы указать на их доступность или отсутствие.
9. Агент ведения журнала должен публиковать лог-сообщения в структурированном формате журналов (например, JSON, CSV).
10. Агент ведения журнала должен добавлять к сообщениям журнала контекстные данные, например, контекст платформы (имя хоста, имя контейнера), контекст среды выполнения (имя класса, имя файла).

Для получения подробного обзора событий, которые следует регистрировать, и возможного формата данных, пожалуйста, ознакомьтесь с [Правилами ведения журнала OWASP Sheet](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Logging_Cheat_Sheet.md#which-events-to-log) и [Правилами ведения словаря приложений Sheet](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Logging_Vocabulary_Cheat_Sheet.md)

## Ссылки на литературу

- [Специальная публикация NIST 800-204](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-204.pdf) “Стратегии обеспечения безопасности для прикладных систем, основанных на микросервисах”
- [Специальная публикация NIST 800-204A](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-204A.pdf) “Создание защищенных приложений на основе микросервисов с использованием архитектуры Service-Mesh”
- [Безопасность микросервисов в действии](https://www.manning.com/books/microservices-security-in-action), Прабат Сиривардана и Нуван Диас, 2020 год, Мэннинг
