# Шпаргалка по Symfony

## Вступление

Эта шпаргалка предназначена для того, чтобы дать разработчикам советы по безопасности при создании приложений с использованием платформы Symfony framework.
В ней рассматриваются распространенные уязвимости и рекомендации по обеспечению безопасности ваших приложений Symfony.

Несмотря на то, что Symfony поставляется со встроенными механизмами безопасности, разработчики должны знать о потенциальных уязвимостях и рекомендациях по обеспечению безопасности создаваемых ими приложений.
Цель данного руководства - охватить общие вопросы безопасности, подчеркивая важность понимания функций безопасности Symfony и способов их эффективного использования.
Независимо от того, являетесь ли вы новичком в Symfony или опытным разработчиком, который хочет усовершенствовать свои методы обеспечения безопасности, этот документ станет ценным источником информации.
Следуя изложенным здесь рекомендациям, вы сможете повысить безопасность своих приложений Symfony и создать более безопасную цифровую среду для пользователей и данных.

## Основные разделы

### Межсайтовый скриптинг (XSS)

Межсайтовый скриптинг (XSS) - это тип атаки, при которой вредоносный JavaScript-код вводится в отображаемую переменную.
Например, если значение переменной name равно `<script>alert('hello')</script>`, и мы отображаем его в HTML следующим образом: `Hello {{name}}`, то введенный скрипт будет выполнен при рендеринге HTML.

Symfony по умолчанию поставляется с шаблонами twig, которые автоматически защищают приложения от XSS-атак, используя **экранирование вывода** для преобразования переменных, содержащих специальные символы, заключая переменную в оператор `{{ }}`.

```twig
<p>Hello {{name}}</p>
{# if 'name' is '<script>alert('hello!')</script>', Twig will output this:
'<p>Hello &lt;script&gt;alert(&#39;hello!&#39;)&lt;/script&gt;</p>' #}
```

Если вы визуализируете переменную, которая является надежной и содержит HTML-содержимое, вы можете использовать *Twig raw filter*, чтобы отключить экранирование выходных данных.

```twig
<p>{{ product.title|raw }}</p>
{# if 'product.title' is 'Lorem <strong>Ipsum</strong>', Twig will output
exactly that instead of 'Lorem &lt;strong&gt;Ipsum&lt;/strong&gt;' #}
```

Ознакомьтесь с [документацией по экранированию выходных данных Twig](https://twig.symfony.com/doc/3.x/api.html#escaper-extension), чтобы получить представление о том, как отключить экранирование выходных данных для определенного блока или всего шаблона.

Для получения другой информации о предотвращении XSS, не относящейся к Symfony, вы можете обратиться к [Руководству по предотвращению межсайтового скриптинга](Cross_Site_Scripting_Prevention_Cheat_Sheet.md).

### Подделка межсайтовых запросов (CSRF)

Компонент Symfony Form автоматически включает токены CSRF в формы, обеспечивая встроенную защиту от атак CSRF.
Symfony автоматически проверяет эти токены, устраняя необходимость в ручном вмешательстве для защиты вашего приложения.

По умолчанию токен CSRF добавляется в виде скрытого поля под названием `_token`, но это можно настроить с помощью других параметров для каждой отдельной формы:

```php
use Symfony\Component\Form\AbstractType;
use Symfony\Component\OptionsResolver\OptionsResolver;

class PostForm extends AbstractType
{

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            // ... 
            'csrf_protection' => true,  // enable/disable csrf protection for this form
            'csrf_field_name' => '_csrf_token',
            'csrf_token_id'   => 'post_item', // change arbitrary string used to generate
        ]);
    }

}
```

Если вы не используете Symfony Forms, вы можете самостоятельно сгенерировать и проверить токены CSRF. Для этого вам необходимо установить компонент `symfony/security-csrf`.

```bash
composer install symfony/security-csrf
```

Включите/отключите защиту от CSRF в файле `config/packages/framework.yaml`:

```yaml
framework:
    csrf_protection: ~
```

Далее рассмотрим этот HTML-шаблон Twig, когда токен CSRF генерируется функцией Twig `csrf_token()`

```twig
<form action="{{ url('delete_post', { id: post.id }) }}" method="post">
    <input type="hidden" name="token" value="{{ csrf_token('delete-post') }}">
    <button type="submit">Delete post</button>
</form>
```

Затем вы можете получить значение токена CSRF в контроллере, используя функцию `isCsrfTokenValid()`:

```php
use App\Entity\Post;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class ExampleController extends AbstractController
{

    #[Route('/posts/{id}', methods: ['DELETE'], name: 'delete_post')]
    public function delete(Post $post, Request $request): Response 
    { 
        $token = $request->request->get('token')
        if($this->isCsrfTokenValid($token)) {
            // ...
        }
        
        // ...
    }
}
```

Вы можете найти более подробную информацию о CSRF, не связанную с Symfony, в разделе [Подделка межсайтовых запросов (CSRF) Шпаргалка](Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.md).

### SQL-инъекция

SQL-инъекция - это тип уязвимости в системе безопасности, которая возникает, когда злоумышленник может манипулировать SQL-запросом таким образом, что может выполнить произвольный SQL-код.
Это может позволить злоумышленникам просматривать, изменять или удалять данные в базе данных, что потенциально может привести к несанкционированному доступу или потере данных.

Symfony, особенно при использовании с Doctrine ORM (объектно-реляционное отображение), обеспечивает защиту от SQL-инъекций с помощью параметров подготовленных инструкций.
Благодаря этому сложнее ошибочно вводить незащищенные запросы, однако это все еще возможно.
В следующем примере показано **небезопасное использование DQL**:

```php
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class ExampleController extends AbstractController {
    
    public function getPost(Request $request, EntityManagerInterface $em): Response
    {
        $id = $request->query->get('id');

        $dql = "SELECT p FROM App\Entity\Post p WHERE p.id = " . $id . ";";
        $query = $em->createQuery($dql);
        $post = $query->getSingleResult();

        // ...
    }
}

```

В приведенных ниже примерах показаны **правильные способы** обеспечения защиты от SQL-инъекций:

- Использование встроенного метода entity repository

```php
$id = $request->query->get('id');
$post = $em->getRepository(Post::class)->findOneBy(['id' => $id]);
```

- Использование языка Doctrine DQL

```php
$query = $em->createQuery("SELECT p FROM App\Entity\Post p WHERE p.id = :id");
$query->setParameter('id', $id);
$post = $query->getSingleResult();
```

- Использование построителя запросов DBAL

```php
$qb = $em->createQueryBuilder();
$post = $qb->select('p')
            ->from('posts','p')
            ->where('id = :id')
            ->setParameter('id', $id)
            ->getQuery()
            ->getSingleResult();
```

Для получения дополнительной информации о Doctrine вы можете обратиться к [их документации](https://www.doctrine-project.org/index.html).
Вы также можете обратиться к [Руководству по предотвращению SQL-инъекций](SQL_Injection_Prevention_Cheat_Sheet.md) для получения дополнительной информации, которая не относится ни к Symfony, ни к Doctrine.

### Внедрение команд

Внедрение команд происходит, когда вредоносный код внедряется в прикладную систему и выполняется.
Для получения дополнительной информации обратитесь к [Шпаргалке по защите от внедрения команд](OS_Command_Injection_Defense_Cheat_Sheet.md).

Рассмотрим следующий пример, в котором файл удаляется с помощью функции exec() без какого-либо вывода входных данных:

```php
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\Attribute\AsController;
use Symfony\Component\Routing\Annotation\Route;

#[AsController]
class ExampleController 
{

    #[Route('/remove_file', methods: ['POST'])]
    public function removeFile(Request $request): Response
    {
        $filename =  $request->request->get('filename');
        exec(sprintf('rm %s', $filename));

        // ...
    }
}
```

В приведенном выше коде нет проверки правильности введенных пользователем данных. Представьте, что произойдет, если пользователь введет вредоносное значение типа `test.txt && rm -rf .` . Чтобы снизить этот риск, рекомендуется использовать собственные функции PHP, такие как в данном случае метод `unlink()` или метод `remove()` компонента файловой системы Symfony вместо `exec()`.

Для получения информации о конкретных функциях файловой системы PHP, относящихся к вашему случаю, вы можете обратиться к [документации по PHP](https://www.php.net/manual/en/refs.fileprocess.file.php) или [Документации по компонентам файловой системы Symfony](https://symfony.com/doc/current/components/filesystem.html).

### Открытое перенаправление

Открытое перенаправление - это уязвимость системы безопасности, которая возникает, когда веб-приложение перенаправляет пользователей на URL-адрес, указанный в недопустимом параметре. Злоумышленники используют эту уязвимость для перенаправления пользователей на вредоносные сайты.

В предоставленном фрагменте кода PHP:

```php
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\Attribute\MapQueryParameter;
use Symfony\Component\Routing\Annotation\Route;

class ExampleController extends AbstractController 
{

    #[Route('/dynamic_redirect', methods: ['GET'])]
    public function dynamicRedirect(#[MapQueryParameter] string $url): Response 
    {
        return $this->redirect($url);
    }
}
```

Функция контроллера перенаправляет пользователей на основе параметра запроса "url" без надлежащей проверки. Злоумышленники могут создавать вредоносные URL-адреса, ведущие ничего не подозревающих пользователей на вредоносные сайты. Чтобы предотвратить открытое перенаправление, всегда проверяйте и очищайте пользовательский ввод перед перенаправлением и избегайте использования ненадежного ввода непосредственно в функциях перенаправления.

### Уязвимости при загрузке файлов

Уязвимости при загрузке файлов - это проблемы безопасности, возникающие, когда приложение неправильно проверяет и обрабатывает загрузку файлов. Важно обеспечить безопасность загрузки файлов, чтобы предотвратить различные типы атак. Вот несколько общих рекомендаций, которые помогут устранить эту проблему в Symfony:

#### Проверка типа и размера файла

Всегда проверяйте тип файла на стороне сервера, чтобы убедиться, что принимаются только разрешенные типы файлов.
Кроме того, рассмотрите возможность ограничения размера загружаемых файлов для предотвращения атак типа "отказ в обслуживании" и обеспечения того, чтобы у вашего сервера было достаточно ресурсов для обработки загрузок.

Пример с атрибутами PHP:

```php
use Symfony\Component\HttpFoundation\File\UploadedFile;
use Symfony\Component\Validator\Constraints\File;

class UploadDto
{
    public function __construct(
        #[File(
            maxSize: '1024k',
            mimeTypes: [
                'application/pdf',
                'application/x-pdf',
            ],
        )]
        public readonly UploadedFile $file,
    ){}
}
```

Пример с формой Symfony:

```php
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\FileType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Validator\Constraints\File;

class FileForm extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('file', FileType::class, [
                'constraints' => [
                    new File([
                        'maxSize' => '1024k', 
                        'mimeTypes' => [
                            'application/pdf',
                            'application/x-pdf',
                        ],
                    ]),
                ],
            ]);
    }
}
```

#### Используйте уникальные имена файлов

Убедитесь, что каждый загруженный файл имеет уникальное имя, чтобы предотвратить перезапись существующих файлов. Вы можете использовать комбинацию уникального идентификатора и исходного имени файла для создания уникального имени.

#### Надежно храните загруженные файлы

Храните загруженные файлы вне общедоступного каталога, чтобы предотвратить прямой доступ к ним. Если вы используете общедоступный каталог для их хранения, настройте свой веб-сервер таким образом, чтобы запретить доступ к каталогу загрузки.

Дополнительные сведения см. в [Инструкции по загрузке файлов](File_Upload_Cheat_Sheet.md).

### Обход каталога

Атака с обходом каталогов или путей направлена на доступ к файлам и каталогам, хранящимся на сервере, путем манипулирования входными данными, которые ссылаются на файлы с помощью последовательностей “../” *точка-косая черта* и их вариаций или с использованием абсолютных путей к файлам.
Для получения более подробной информации обратитесь к [OWASP Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal).

Вы можете защитить свое приложение от атаки с обходом каталогов, проверив правильность абсолютного пути к запрашиваемому файлу или удалив информацию о каталоге из имени файла, введенного при вводе.

- Проверьте, существует ли путь, используя функцию PHP *realpath*, и убедитесь, что он ведет в каталог хранилища

```php
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\Attribute\MapQueryParameter;
use Symfony\Component\Routing\Annotation\Route;

class ExampleController extends AbstractController 
{

    #[Route('/download', methods: ['GET'])]
    public function download(#[MapQueryParameter] string $filename): Response 
    {
        $storagePath = $this->getParameter('kernel.project_dir') . '/storage';
        $filePath = $storagePath . '/' . $filename;

        $realBase = realpath($storagePath);
        $realPath = realpath($filePath);

        if ($realPath === false || !str_starts_with($realPath, $realBase))
        {
            //Directory Traversal!
        }

        // ...

    }
}
```

- Извлеките информацию из каталога с помощью функции PHP *basename*

```php
// ...

$storagePath = $this->getParameter('kernel.project_dir') . '/storage';
$filePath = $storagePath . '/' . basename($filename);

// ...
```

### Уязвимости зависимостей

Уязвимости в зависимостях могут подвергать ваше приложение различным рискам, поэтому крайне важно внедрять лучшие практики.
Поддерживайте все компоненты Symfony и сторонние библиотеки в актуальном состоянии.

Composer, менеджер зависимостей для PHP, упрощает обновление пакетов PHP:

```bash
composer update
```

При использовании нескольких зависимостей некоторые из них могут содержать уязвимости в системе безопасности.
Для решения этой проблемы Symfony поставляется с программой [Symfony Security Checker](https://symfony.com/doc/current/setup.html#checking-security-vulnerabilities). Этот инструмент специально проверяет файл *composer.lock* в вашем проекте, чтобы выявить любые известные уязвимости в установленных зависимостях и устранить любые потенциальные проблемы с безопасностью в вашем проекте Symfony.

Чтобы использовать средство проверки безопасности, выполните следующую команду, используя [Symfony CLI](https://github.com/symfony-cli/symfony-cli):

```bash
symfony check:security
```

Вам также следует рассмотреть подобные инструменты:

- [Local PHP Security Checker](https://github.com/fabpot/local-php-security-checker)

- [Enlightn Security Checker](https://github.com/enlightn/security-checker)

### Совместное использование ресурсов разных источников Cross-Origin Resource Sharing (CORS)

CORS - это функция безопасности, реализованная в веб-браузерах для контроля того, как веб-приложения в одном домене могут запрашивать ресурсы, размещенные в других доменах, и взаимодействовать с ними.

В Symfony вы можете управлять политиками CORS с помощью `nelmio/cors-bundle`. Этот пакет позволяет вам точно управлять правилами CORS без изменения настроек вашего сервера.
To install it with Composer, run:

```bash
composer require nelmio/cors-bundle
```

Для пользователей Symfony Flex при установке автоматически создается базовый конфигурационный файл в каталоге "config/packages". Взгляните на пример конфигурации для маршрутов, начинающихся с префикса */API*.

```yaml
# config/packages/nelmio_cors.yaml
nelmio_cors:
    defaults:
        origin_regex: true
        allow_origin: ['*']
        allow_methods: ['GET', 'OPTIONS', 'POST', 'PUT', 'PATCH', 'DELETE']
        allow_headers: ['*']
        expose_headers: ['Link']
        max_age: 3600
    paths:
        '^/api': ~  # ~ означает, что конфигурации для этого пути наследуются от значений по умолчанию
```

### Заголовки, связанные с безопасностью

Рекомендуется повысить безопасность вашего приложения Symfony, добавив к вашим ответам важные заголовки безопасности, такие как:

- Strict-Transport-Security
- X-Frame-Options
- X-Content-Type-Options
- Content-Security-Policy
- X-Permitted-Cross-Domain-Policies
- Referrer-Policy
- Clear-Site-Data
- Cross-Origin-Embedder-Policy
- Cross-Origin-Opener-Policy
- Cross-Origin-Resource-Policy
- Cache-Control

Для получения более подробной информации об отдельных заголовках обратитесь к [Проекту защищенных заголовков OWASP](https://owasp.org/www-project-secure-headers/).

В Symfony вы можете добавлять эти заголовки вручную или автоматически, прослушивая [ResponseEvent](https://symfony.com/doc/current/reference/events.html#kernel-response) для каждого ответа или настраивая веб-серверы, такие как Nginx или Apache.

```php
use Symfony\Component\HttpFoundation\Request;

$response = new Response();
$response->headers->set('X-Frame-Options', 'SAMEORIGIN');
```

### Управление сеансами и файлами cookie

По умолчанию сеансы настроены и включены в безопасном режиме. Однако ими можно управлять вручную в `config/packages/framework.yaml` с помощью ключа `framework.session`. Убедитесь, что в конфигурации сеанса указано следующее, чтобы повысить осведомленность вашего приложения.

Убедитесь, что для параметра `cookie_secure` явно не установлено значение `false` (по умолчанию для него установлено значение `true`). Установка значения `true` только для http означает, что файл cookie не будет доступен с помощью JavaScript.

```yaml
cookie_httponly: true
```

Обязательно установите короткую продолжительность сеанса. В соответствии с [рекомендациями Owasp](Session_Management_Cheat_Sheet.md) Продолжительность сеанса должна составлять 2-5 минут для приложений с высокой стоимостью и 15-30 минут для приложений с низким уровнем риска.

```yaml
cookie_lifetime: 5
```

Рекомендуется установить для параметра `cookie_samesite` значение `lax` или `strict`, чтобы предотвратить отправку файлов cookie из разных источников. `lax` позволяет отправлять файлы cookie вместе с "безопасными" навигациями верхнего уровня и запросами с того же сайта. При использовании `strict` было бы невозможно отправить какие-либо файлы cookie, если HTTP-запрос исходит не из того же домена.

```yaml
cookie_samesite: lax|strict
```

Установка значения `cookie_secure` на `auto` гарантирует нам, что файлы cookie отправляются только по защищенным соединениям, что означает `true` для протокола HTTPS и `false` для протокола HTTP.

```yaml
cookie_secure: auto
```

OWASP предоставляет более общую информацию о сеансах в [Шпаргалке по управлению сеансами](Session_Management_Cheat_Sheet.md).
Вы также можете ознакомиться с [Защитой файлов cookie Guide](https://owasp.org/www-chapter-london/assets/slides/OWASPLondon20171130_Cookie_Security_Myths_Misconceptions_David_Johansson.pdf).

---
В Symfony сеансы управляются самой платформой и полагаются на механизмы обработки сеансов Symfony, а не на обработку сеансов PHP по умолчанию с помощью директивы `session.auto_start = 1` в файле php.ini.
Директива `session.auto_start = 1` в PHP используется для автоматического запуска сеанса при каждом запросе, минуя явные вызовы `session_start()`. Однако при использовании Symfony для управления сеансами рекомендуется отключить `session.auto_start`, чтобы предотвратить конфликты и непредвиденное поведение.

### Идентификация

[Symfony Security](https://symfony.com/doc/current/security.html) предоставляет надежную систему аутентификации, которая включает провайдеров, брандмауэры и средства контроля доступа для обеспечения безопасной и контролируемой среды доступа. Параметры аутентификации можно настроить в `config/packages/security.yaml`.

- **Поставщики услуг**

    Проверка подлинности Symfony позволяет поставщикам извлекать информацию о пользователях из различных типов хранилищ, таких как базы данных, LDAP или пользовательские источники. Поставщики получают пользователей на основе определенного свойства и загружают соответствующий пользовательский объект.

    В приведенном ниже примере представлен [Entity User Provider](https://symfony.com/doc/current/security/user_providers.html#security-entity-user-provider), который использует Doctrine для поиска пользователя по уникальному идентификатору.

    ```yaml
    providers:
        app_user_provider:
            entity:
                class: App\Entity\User
                property: email
    ```

- **Брандмауэры**

    Symfony использует брандмауэры для определения конфигураций безопасности для различных частей приложения. Каждый брандмауэр определяет определенный набор правил и действий для входящих запросов. Они защищают различные разделы приложения, указывая, какие маршруты или URL-адреса защищены, какие механизмы аутентификации следует использовать и как обрабатывать несанкционированный доступ. Брандмауэр может быть связан с определенными шаблонами, методами запроса, средствами контроля доступа и поставщиками аутентификации.

    ```yaml
    firewalls:
        dev: # отключите защиту маршрутов, используемых в среде разработки
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false
        admin: # обрабатывать аутентификацию в шаблонных маршрутах /admin
            lazy: true
            provider: app_user_provider
            pattern: ^/admin
            custom_authenticator: App\Security\AdminAuthenticator
            logout:
                path: app_logout
                target: app_login
        main: # главный брандмауэр, который включает в себя все остальные маршруты
            lazy: true
            provider: app_user_provider
    ```

- **Контроль доступа**

    Управление доступом определяет, какие пользователи могут получить доступ к определенным частям приложения. Эти правила состоят из шаблонов путей и требуемых ролей или разрешений. Правила управления доступом настраиваются с помощью ключа `access_control`.

    ```yaml
    access_control:
        - { path: ^/admin, roles: ROLE_ADMIN } # only user with ROLE_ADMIN role is allowed
        - { path: ^/login, roles: PUBLIC_ACCESS } # everyone can access this route
    ```

### Раскрытие информации об обработке ошибок

Symfony имеет надежную систему обработки ошибок. По умолчанию приложения Symfony настроены на отображение подробных сообщений об ошибках только в среде разработки по соображениям безопасности. В рабочей среде отображается страница с общими ошибками. Система обработки ошибок Symfony также позволяет настраивать страницы с ошибками на основе различных кодов состояния HTTP, обеспечивая удобство работы с пользователем. Кроме того, Symfony регистрирует подробную информацию об ошибках, помогая разработчикам эффективно выявлять и устранять проблемы.

Для получения дополнительной информации об обработке ошибок, не связанных с Symfony, обратитесь к [Шпаргалке по обработке ошибок](Error_Handling_Cheat_Sheet.md).

### Конфиденциальные данные

В Symfony лучшим способом хранения конфигураций, таких как ключи API и т.д., является использование переменных окружения, которые зависят от местоположения приложения.
Для обеспечения безопасности конфиденциальных данных Symfony предоставляет систему управления секретами, в которой значения дополнительно кодируются с использованием криптографических ключей и хранятся как **секреты**.

Рассмотрим пример, в котором API_KEY хранится как секрет:

Чтобы сгенерировать пару криптографических ключей, вы можете выполнить следующую команду. Файл с закрытым ключом является высокочувствительным, и его не следует хранить в хранилище.

```bash
bin/console secrets:generate-keys
```

Эта команда сгенерирует файл для секрета API_KEY в `config/secrets/env(dev|prod|etc.)`.

```bash
bin/console secret:set API_KEY
```

Вы можете получить доступ к секретным значениям в своем коде таким же образом, как и к переменным среды.
Очень важно отметить, что если существуют переменные среды и секретные данные с одинаковыми именами, **значения из переменных среды всегда будут переопределять секретные данные**.

Для получения более подробной информации обратитесь к [Документации Symfony Secrets](https://symfony.com/doc/current/configuration/secrets.html).

### Подводя итог

- Убедитесь, что ваше приложение не находится в режиме отладки во время работы. Чтобы отключить режим отладки, установите для переменной окружения `APP_ENV` значение `prod`:

    ```ini
    APP_ENV=prod
    ```

- Убедитесь, что ваша конфигурация PHP безопасна. Вы можете обратиться к [Шпаргалке по настройке PHP](PHP_Configuration_Cheat_Sheet.md) для получения дополнительной информации о безопасных настройках конфигурации PHP.

- Убедитесь, что SSL-сертификат правильно настроен на вашем веб-сервере, и настройте его на принудительное использование HTTPS, перенаправив HTTP-трафик на HTTPS.

- Реализуйте заголовки безопасности для повышения уровня безопасности вашего приложения.

- Убедитесь, что права доступа к файлам и каталогам установлены правильно, чтобы минимизировать риски для безопасности.

- Регулярно создавайте резервные копии своей рабочей базы данных и критически важных файлов. Разработайте план восстановления, позволяющий быстро восстановить ваше приложение в случае возникновения каких-либо проблем.

- Используйте средства проверки безопасности для сканирования ваших зависимостей на предмет выявления известных уязвимостей.

- Рассмотрите возможность настройки инструментов мониторинга и механизмов отчетности об ошибках для быстрого выявления и устранения проблем в вашей производственной среде. Изучите такие инструменты, как [Blackfire.io](https://www.blackfire.io).

## Ссылки на литературу

- [Symfony CSRF Documentation](https://symfony.com/doc/current/security/csrf.html)
- [Symfony Twig Documentation](https://symfony.com/doc/current/templates.html)
- [Symfony Validation Documentation](https://symfony.com/doc/current/validation.html)
- [Symfony Blackfire Documentation](https://symfony.com/doc/current/the-fast-track/en/29-performance.html)
- [Doctrine Security Documentation](https://www.doctrine-project.org/projects/doctrine-dbal/en/3.7/reference/security.html)
