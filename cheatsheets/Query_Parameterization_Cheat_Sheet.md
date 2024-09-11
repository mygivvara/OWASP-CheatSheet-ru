# Шпаргалка по параметризации запроса

## Вступление

[SQL-инъекция](https://owasp.org/www-community/attacks/SQL_Injection) - одна из самых опасных веб-уязвимостей. Настолько, что она заняла первое место как в Топ-10 OWASP [версии 2013 года](https://wiki.owasp.org/index.php/Top_10_2013-A1-Injection), так и в [версии 2017 года](https://owasp.org/www-project-top-ten/2017/A1_2017-Injection.html). По состоянию на 2021 год, она занимает 3-е место в [Топ-10 OWASP](https://owasp.org/Top 10/A 03_2021-Injection/).

Это представляет серьезную угрозу, поскольку внедрение SQL позволяет вредоносному коду злоумышленника изменять структуру SQL-инструкции веб-приложения таким образом, чтобы можно было украсть данные, модифицировать их или потенциально облегчить внедрение команд в базовую ОС.

Эта шпаргалка является производной от [Шпаргалки по предотвращению SQL-инъекций](SQL_Injection_Prevention_Cheat_Sheet.md).

## Примеры параметризованных запросов

Внедрение SQL-кода лучше всего предотвратить с помощью [*параметризованных запросов*](SQL_Injection_Prevention_Cheat_Sheet.md). На следующей диаграмме с примерами реального кода показано, как создавать параметризованные запросы на большинстве распространенных веб-языков. Цель этих примеров кода - продемонстрировать веб-разработчику, как избежать внедрения SQL-кода при создании запросов к базе данных в веб-приложении.

Пожалуйста, обратите внимание, что многие клиентские платформы и библиотеки предлагают параметризацию запросов на стороне клиента. Эти библиотеки часто просто создают запросы с конкатенацией строк перед отправкой необработанных запросов на сервер. Пожалуйста, убедитесь, что параметризация запросов выполняется на стороне сервера!

### Примеры подготовленных инструкций

#### Использование встроенной функции Java

```java
String custname = request.getParameter("customerName");
String query = "SELECT account_balance FROM user_data WHERE user_name = ? ";  
PreparedStatement pstmt = connection.prepareStatement( query );
pstmt.setString( 1, custname);
ResultSet results = pstmt.executeQuery( );
```

#### Использование Java в режиме гибернации

```java
// HQL
@Entity // объявить как сущность;
@NamedQuery(
 name="findByDescription",
 query="FROM Inventory i WHERE i.productDescription = :productDescription"
)
public class Inventory implements Serializable {
 @Id
 private long id;
 private String productDescription;
}

// Пример использования
// Это тоже должно быть валидировано
String userSuppliedParameter = request.getParameter("Product-Description");
// Выполните проверку вводимых данных для обнаружения атак
List<Inventory> list =
 session.getNamedQuery("findByDescription")
 .setParameter("productDescription", userSuppliedParameter).list();

// Критерии API
// Это тоже должно быть валидировано
String userSuppliedParameter = request.getParameter("Product-Description");
// Выполните проверку вводимых данных для обнаружения атакacks
Inventory inv = (Inventory) session.createCriteria(Inventory.class).add
(Restrictions.eq("productDescription", userSuppliedParameter)).uniqueResult();
```

#### Использование встроенной функции .NET

```csharp
String query = "SELECT account_balance FROM user_data WHERE user_name = ?";
try {
   OleDbCommand command = new OleDbCommand(query, connection);
   command.Parameters.Add(new OleDbParameter("customerName", CustomerName Name.Text));
   OleDbDataReader reader = command.ExecuteReader();
   // …
} catch (OleDbException se) {
   // error handling
}
```

#### Использование встроенной функции ASP .NET

```csharp
string sql = "SELECT * FROM Customers WHERE CustomerId = @CustomerId";
SqlCommand command = new SqlCommand(sql);
command.Parameters.Add(new SqlParameter("@CustomerId", System.Data.SqlDbType.Int));
command.Parameters["@CustomerId"].Value = 1;
```

#### Использование Ruby с ActiveRecord

```ruby
## Create
Project.create!(:name => 'owasp')
## Read
Project.all(:conditions => "name = ?", name)
Project.all(:conditions => { :name => name })
Project.where("name = :name", :name => name)
## Update
project.update_attributes(:name => 'owasp')
## Delete
Project.delete(:name => 'name')
```

#### Использование встроенной функции Ruby

```ruby
insert_new_user = db.prepare "INSERT INTO users (name, age, gender) VALUES (?, ? ,?)"
insert_new_user.execute 'aizatto', '20', 'male'
```

#### Использование PHP с объектами данных PHP

```php
$stmt = $dbh->prepare("INSERT INTO REGISTRY (name, value) VALUES (:name, :value)");
$stmt->bindParam(':name', $name);
$stmt->bindParam(':value', $value);
```

#### Использование встроенной функции ColdFusion

```coldfusion
<cfquery name = "getFirst" dataSource = "cfsnippets">
    SELECT * FROM #strDatabasePrefix#_courses WHERE intCourseID =
    <cfqueryparam value = #intCourseID# CFSQLType = "CF_SQL_INTEGER">
</cfquery>
```

#### Использование PERL с независимым от базы данных интерфейсом

```perl
my $sql = "INSERT INTO foo (bar, baz) VALUES ( ?, ? )";
my $sth = $dbh->prepare( $sql );
$sth->execute( $bar, $baz );
```

#### Использование Rust с SQLx
<!-- contributed by GeekMasher -->

```rust
// Вводимые данные из CLI args, но могут быть любыми
let username = std::env::args().last().unwrap();

// Использование встроенных макросов (проверка во время компиляции)
let users = sqlx::query_as!(
        User,
        "SELECT * FROM users WHERE name = ?",
        username
    )
    .fetch_all(&pool)
    .await 
    .unwrap();

// Использование встроенных функций
let users: Vec<User> = sqlx::query_as::<_, User>(
        "SELECT * FROM users WHERE name = ?"
    )
    .bind(&username)
    .fetch_all(&pool)
    .await
    .unwrap();
```

### Примеры хранимых процедур

SQL, который вы пишете в своем веб-приложении, - не единственное место, где могут быть обнаружены уязвимости при внедрении SQL. Если вы используете хранимые процедуры и динамически создаете SQL внутри них, вы также можете создать уязвимости при внедрении SQL.

Динамический SQL можно параметризовать с помощью переменных bind, чтобы обеспечить безопасность динамически создаваемого SQL.

Вот несколько примеров использования переменных bind в хранимых процедурах в различных базах данных.

#### Oracle, используя PL/SQL

##### Обычная хранимая процедура

Динамический SQL не создается. Параметры, передаваемые хранимым процедурам, естественным образом привязываются к их местоположению в запросе, при этом ничего особенного не требуется:

```sql
PROCEDURE SafeGetBalanceQuery(UserID varchar, Dept varchar) AS BEGIN
   SELECT balance FROM accounts_table WHERE user_ID = UserID AND department = Dept;
END;
```

##### Хранимая процедура с использованием привязки переменных в SQL Run с EXECUTE

Переменные привязки используются для того, чтобы сообщить базе данных, что входные данные для этого динамического SQL являются "данными", а не, возможно, кодом:

```sql
PROCEDURE AnotherSafeGetBalanceQuery(UserID varchar, Dept varchar)
          AS stmt VARCHAR(400); result NUMBER;
BEGIN
   stmt := 'SELECT balance FROM accounts_table WHERE user_ID = :1
            AND department = :2';
   EXECUTE IMMEDIATE stmt INTO result USING UserID, Dept;
   RETURN result;
END;
```

#### SQL Server с использованием Transact-SQL

##### Обычная хранимая процедура

Динамический SQL не создается. Параметры, передаваемые в хранимые процедуры, естественным образом привязываются к их местоположению в запросе, при этом ничего особенного не требуется:

```sql
PROCEDURE SafeGetBalanceQuery(@UserID varchar(20), @Dept varchar(10)) AS BEGIN
   SELECT balance FROM accounts_table WHERE user_ID = @UserID AND department = @Dept
END
```

##### Хранимая процедура с использованием связывания переменных в SQL Run с EXEC

Переменные привязки используются для того, чтобы сообщить базе данных, что входные данные этого динамического SQL являются «данными», а не возможным кодом:

```sql
PROCEDURE SafeGetBalanceQuery(@UserID varchar(20), @Dept varchar(10)) AS BEGIN
   DECLARE @sql VARCHAR(200)
   SELECT @sql = 'SELECT balance FROM accounts_table WHERE '
                 + 'user_ID = @UID AND department = @DPT'
   EXEC sp_executesql @sql,
                      '@UID VARCHAR(20), @DPT VARCHAR(10)',
                      @UID=@UserID, @DPT=@Dept
END
```

## Ссылки

- [Сайт Bobby Tables (созданный по мотивам веб-комикса XKCD) содержит множество примеров параметризованных подготовленных инструкций и хранимых процедур на разных языках](http://bobby-tables.com/)
- OWASP [Шпаргалка по предотвращению SQL-инъекций](SQL_Injection_Prevention_Cheat_Sheet.md)