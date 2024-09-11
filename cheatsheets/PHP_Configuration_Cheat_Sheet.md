# Шпаргалка по настройке PHP

## Вступление

Эта страница предназначена для того, чтобы помочь тем, кто настраивает PHP и веб-сервер, на котором он работает, для обеспечения максимальной безопасности.

Ниже вы найдете информацию о правильных настройках файла `php.ini` и инструкции по настройке веб-серверов Apache, Nginx и Caddy.

Для получения информации об общей безопасности кодовой базы PHP, пожалуйста, обратитесь к двум следующим замечательным руководствам:

- [Руководство по безопасности PHP от Paragonie на 2018 год](https://paragonie.com/blog/2017/12/2018-guide-building-secure-php-software)
- [Потрясающая защита PHP](https://github.com/guardrailsio/awesome-php-security)

## Настройка и развертывание PHP

### php.ini

Некоторые из приведенных ниже настроек необходимо адаптировать к вашей системе, в частности `session.save_path`, `session.cookie_path` (например, `/var/www/mysite`) и `session.cookie_domain` (например, `ExampleSite.com`).

У вас должна быть установлена [поддерживаемая версия PHP](https://www.php.net/supported-versions.php) (на момент написания этой статьи 8.1 - самая старая версия, получающая поддержку безопасности от PHP, хотя поставщики дистрибутивов часто предоставляют расширенную поддержку). Ознакомьтесь с [основными директивами `php.ini`](https://www.php.net/manual/ini.core.php) в руководстве по PHP для получения полной информации о каждом значении в файле конфигурации `php.ini`.

Вы можете найти копию следующих значений в [готовом к использованию файле `php.ini` здесь](https://github.com/dan ehrlich 1/very-secure-php-ini).

#### Обработка ошибок PHP

```text
expose_php              = Off
error_reporting         = E_ALL
display_errors          = Off
display_startup_errors  = Off
log_errors              = On
error_log               = /valid_path/PHP-logs/php_error.log
ignore_repeated_errors  = Off
```

Помните, что на рабочем сервере необходимо установить `display_errors` в значение `Off`, и нелишним будет часто просматривать журналы.

#### Общие настройки PHP

```text
doc_root                = /path/DocumentRoot/PHP-scripts/
open_basedir            = /path/DocumentRoot/PHP-scripts/
include_path            = /path/PHP-pear/
extension_dir           = /path/PHP-extensions/
mime_magic.magicfile    = /path/PHP-magic.mime
allow_url_fopen         = Off
allow_url_include       = Off
variables_order         = "GPCS"
allow_webdav_methods    = Off
session.gc_maxlifetime  = 600
```

`allow_url_*` предотвращает простое преобразование [LFI](https://www.acunetix.com/blog/articles/local-file-inclusion-lfi/) в [RFI](https://www.acunetix.com/blog/articles/remote-file-inclusion-rfi/).

#### Обработка загрузки файлов на PHP

```text
file_uploads            = On
upload_tmp_dir          = /path/PHP-uploads/
upload_max_filesize     = 2M
max_file_uploads        = 2
```

Если ваше приложение не использует загрузку файлов, а единственными данными, которые пользователь будет вводить/загружать, являются формы, не требующие вложения документов, `file_uploads` следует поставить значение `Off`.

#### Обработка исполняемого файла PHP

```text
enable_dl               = Off
disable_functions       = system, exec, shell_exec, passthru, phpinfo, show_source, highlight_file, popen, proc_open, fopen_with_path, dbmopen, dbase_open, putenv, move_uploaded_file, chdir, mkdir, rmdir, chmod, rename, filepro, filepro_rowcount, filepro_retrieve, posix_mkfifo
disable_classes         =
```

Это опасные функции PHP. Вам следует отключить все, что вы не используете.

#### Обработка сеанса PHP

Настройки сеанса являются одними из наиболее важных параметров, на которые следует обратить особое внимание при настройке. Рекомендуется изменить `session.name` на что-то новое.

```text
 session.save_path                = /path/PHP-session/
 session.name                     = myPHPSESSID
 session.auto_start               = Off
 session.use_trans_sid            = 0
 session.cookie_domain            = full.qualified.domain.name
 #session.cookie_path             = /application/path/
 session.use_strict_mode          = 1
 session.use_cookies              = 1
 session.use_only_cookies         = 1
 session.cookie_lifetime          = 14400 # 4 hours
 session.cookie_secure            = 1
 session.cookie_httponly          = 1
 session.cookie_samesite          = Strict
 session.cache_expire             = 30
 session.sid_length               = 256
 session.sid_bits_per_character   = 6
```

#### Еще несколько параноидальных проверок системы безопасности

```text
session.referer_check   = /application/path
memory_limit            = 50M
post_max_size           = 20M
max_execution_time      = 60
report_memleaks         = On
html_errors             = Off
zend.exception_ignore_args = On
```

### Snuffleupagus

[Snuffleupagus](https://snuffleupagus.readthedocs.io) является продолжением Suhosin для PHP 7 и более поздних версий, с [современными функциями](https://snuffleupagus.readthedocs.io/features.html). Он считается стабильным и может использоваться в рабочей среде.
