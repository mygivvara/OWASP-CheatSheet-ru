# Шпаргалка по массовому присвоению

## Введение

### Определение

Программные платформы иногда позволяют разработчикам автоматически привязывать параметры HTTP-запросов к переменным или объектам программного кода, чтобы упростить использование этой платформы разработчиками. Иногда это может нанести вред.

Злоумышленники иногда могут использовать эту методологию для создания новых параметров, которые разработчик никогда не планировал, что, в свою очередь, создает или перезаписывает новые переменные или объекты в программном коде, которые не были предусмотрены.

Это называется уязвимостью массового присвоения.

### Альтернативные названия

В зависимости от языка/фреймворка, о котором идет речь, у этой уязвимости может быть несколько [альтернативных названий](https://cwe.mitre.org/data/definitions/915.html):

- **Массовое присвоение:** Ruby on Rails, Node JS.
- **Автоматическое привязывание:** Spring MVC, ASP NET MVC.
- **Внедрение объектов:** PHP.

### Пример

Предположим, что существует форма для редактирования информации об учетной записи пользователя:

```html
<form>
     <input name="userid" type="text">
     <input name="password" type="text">
     <input name="email" text="text">
     <input type="submit">
</form>  
```

Вот объект, к которому привязана форма:

```java
public class User {
   private String userid;
   private String password;
   private String email;
   private boolean isAdmin;

   //Getters & Setters
}
```

Вот контроллер, обрабатывающий запрос:

```java
@RequestMapping(value = "/addUser", method = RequestMethod.POST)
public String submit(User user) {
   userService.add(user);
   return "successPage";
}
```

Вот типичный запрос:

```text
POST /addUser
...
userid=bobbytables&password=hashedpass&email=bobby@tables.com
```

А вот эксплойт, в котором мы устанавливаем значение атрибута `is Admin` экземпляра класса `User`:

```text
POST /addUser
...
userid=bobbytables&password=hashedpass&email=bobby@tables.com&isAdmin=true
```

### Возможность использования

Эта функциональность становится доступной, когда:

- Злоумышленник может угадать общие конфиденциальные поля.
- Злоумышленник имеет доступ к исходному коду и может просматривать модели для конфиденциальных полей.
- И объект с конфиденциальными полями имеет пустой конструктор.

### Практический пример GitHub

В 2012 году GitHub был взломан с помощью массового присвоения. Пользователь мог загрузить свой открытый ключ в любую организацию и, таким образом, вносить любые последующие изменения в свои репозитории. [Сообщение в блоге GitHub](https://blog.github.com/2012-03-04-public-key-security-vulnerability-and-mitigation/).

### Решения

- Разрешить - вывести список обязательных, конфиденциальных полей.
- Заблокировать - вывести список обязательных, конфиденциальных полей.
- Использовать [Объекты передачи данных](https://martinfowler.com/eaaCatalog/dataTransferObject.html) (DTO).

## Общие решения

Архитектурный подход заключается в создании объектов передачи данных и отказе от привязки вводимых данных непосредственно к объектам домена. В DTO включены только те поля, которые предназначены для редактирования пользователем.

```java
public class UserRegistrationFormDTO {
 private String userid;
 private String password;
 private String email;

 //NOTE: isAdmin поле отсутствует

 //Getters & Setters
}
```

## Решения, зависящие от языка и фреймворка

### Spring MVC

#### Список разрешений

```java
@Controller
public class UserController
{
    @InitBinder
    public void initBinder(WebDataBinder binder, WebRequest request)
    {
        binder.setAllowedFields(["userid","password","email"]);
    }
...
}
```

Взгляните [here](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/validation/DataBinder.html#setAllowedFields-java.lang.Строка...-) для документации.

#### Блок-листинг

```java
@Controller
public class UserController
{
   @InitBinder
   public void initBinder(WebDataBinder binder, WebRequest request)
   {
      binder.setDisallowedFields(["isAdmin"]);
   }
...
}
```

Взгляните [here](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/validation/DataBinder.html#setDisallowedFields-java.lang.Строка...-) для документации.

### NodeJS + Mongoose

#### Allow-listing

```javascript
var UserSchema = new mongoose.Schema({
    userid: String,
    password: String,
    email : String,
    isAdmin : Boolean,
});

UserSchema.statics = {
    User.userCreateSafeFields: ['userid', 'password', 'email']
};

var User = mongoose.model('User', UserSchema);

_ = require('underscore');
var user = new User(_.pick(req.body, User.userCreateSafeFields));
```

Ознакомьтесь с документацией [здесь](http://underscorejs.org/#pick).

#### Block-listing

```javascript
var massAssign = require('mongoose-mass-assign');

var UserSchema = new mongoose.Schema({
    userid: String,
    password: String,
    email : String,
    isAdmin : { type: Boolean, protect: true, default: false }
});

UserSchema.plugin(massAssign);

var User = mongoose.model('User', UserSchema);

/** Static method, useful for creation **/
var user = User.massAssign(req.body);

/** Instance method, useful for updating**/
var user = new User;
user.massAssign(req.body);

/** Static massUpdate method **/
var input = { userid: 'bhelx', isAdmin: 'true' };
User.update({ '_id': someId }, { $set: User.massUpdate(input) }, console.log);
```

Ознакомьтесь с документацией [здесь](https://www.npmjs.com/package/mongoose-mass-assign).

### Ruby On Rails

Ознакомьтесь с документацией [здесь](https://guides.rubyonrails.org/v3.2.9/security.html#mass-assignment).

### Django

Ознакомьтесь с документацией [здесь](https://coffeeonthekeyboard.com/massassignmentsecurity-part-10-855/).

### ASP NET

Ознакомьтесь с документацией [here](https://odetocode.com/Blogs/scott/archive/2012/03/11/complete-guide-to-mass-assignment-in-asp-net-mvc.aspx).

### PHP Laravel + Eloquent

#### Allow-listing

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    private $userid;
    private $password;
    private $email;
    private $isAdmin;

    protected $fillable = array('userid','password','email');
}
```

Ознакомьтесь с документацией [здесь](https://laravel.com/docs/5.2/eloquent#mass-assignment).

#### Block-listing

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    private $userid;
    private $password;
    private $email;
    private $isAdmin;

    protected $guarded = array('isAdmin');
}
```

Ознакомьтесь с документацией [здесь](https://laravel.com/docs/5.2/eloquent#mass-assignment).

### Grails

Ознакомьтесь с документацией [здесь](http://spring.io/blog/2012/03/28/secure-data-binding-with-grails/).

### Play

Ознакомьтесь с документацией [здесь](https://www.playframework.com/documentation/1.4.x/controllers#nobinding).

### Jackson (средство отображения объектов в формате JSON)

Взгляните [здесь](https://www.baeldung.com/jackson-field-serializable-deserializable-or-not ) и [here](http://lifelongprogrammer.blogspot.com/2015/09/using-jackson-view-to-protect-mass-assignment.html ) для получения документации.

### GSON (средство отображения объектов в формате JSON)

Взгляните [here](https://sites.google.com/site/gson/gson-user-guide#TOC-Excluding-Fields-From-Serialization-and-Deserialization ) и [здесь](https://stackoverflow.com/a/27986860 ) для документации.

### JSON-Lib (JSON Object Mapper)

Ознакомьтесь с документацией [здесь](http://json-lib.sourceforge.net/advanced.html).

### Flexjson (JSON Object Mapper)

Ознакомьтесь с документацией [здесь](http://flexjson.sourceforge.net/#Serialization).

## Ссылки и дальнейшее чтение

- [Массовое присвоение, Rails и вы](https://code.tutsplus.com/tutorials/mass-assignment-rails-and-you--net-31695)