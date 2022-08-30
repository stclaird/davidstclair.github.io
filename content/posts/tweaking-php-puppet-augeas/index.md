---
title: "Tweaking your PHP via php.ini and Puppet Augeas"
date: 2016-09-23T15:28:38+01:00
authors: ["David StClair"]
draft: false
---
If you have ever wanted to make a change to some of the settings to your php setup via PHP.ini file then using puppet and the augeas type could be the perfect solution. [Augeas](https://augeas.net/docs/lenses.html)  will read in a configuration file and allow you to specify changes and save them for you. Augeas is not actually a puppet specific tool and can be used generally for configuration file alteration. Of course, you could use an ERB template to build the php.ini but honestly, that is overkill when you might only need to make just one or two alterations to your php.ini file.

In the following example I have php example located in my application users directory (which I incidentally built with https://php-build.github.io/)
```bash
/home/my-appuser/.phps/5.5.20/etc/php.ini
```
And I have decided that I want to turn off the php version number off in the http requests I send to my clients browser. Now, if I wanted to that manually alter that setting I would need to find the entry
```bash
/home/my-appuser/.phps/5.5.20/etc/php.ini
```
```bash
Expose_php = On
```
and change it to
```bash
Expose_php = Off
```
Since I want puppet to take care of this change for me, I am going to need to create an entry in my manifest which looks something like this:
```bash
augeas { "php.ini":
lens => "php.lns",
changes => [ "set expose_php Off" ],
incl => "/home/my-appuser/.phps/5.5.20/etc/php.ini",
context => "/files/home/edge-cms/.phps/5.5.20/etc/php.ini/PHP",
}
```
This should look reasonably familiar to most puppet coders but a couple of points worth mentioning; The “incl” statement tells puppet that we are using a non standard path out side of “/etc” where it would normally look for files. We also tell Augeas that we are changing a php.ini file and therefore we should use the php lens And finally we are setting the context to PHP. If you don’t you will likely get errors like the following doozy:

```bash
Debug: Augeas[php.ini](provider=augeas): Put failed on one or more files, output from /augeas//error:
Debug: Augeas[php.ini](provider=augeas): /augeas/files/home/edge-cms/.phps/5.5.20/etc/php.ini/error = put_failed
Debug: Augeas[php.ini](provider=augeas): /augeas/files/home/edge-cms/.phps/5.5.20/etc/php.ini/error/path = /files/home/edge-cms/.phps/5.5.20/etc/php.ini
Debug: Augeas[php.ini](provider=augeas): /augeas/files/home/edge-cms/.phps/5.5.20/etc/php.ini/error/lens = /usr/share/augeas/lenses/dist/php.aug:44.13-.35:
Debug: Augeas[php.ini](provider=augeas): /augeas/files/home/edge-cms/.phps/5.5.20/etc/php.ini/error/message = Failed to match
{ /\\.anon/ }?{ /(#comment[^]\001-\004\n\/][^]\001-\004\n\/]|#commen[^]\001-\004\n\/t][^]\001-\004\n\/])[^]\001-\004\n\/]*|#comment[^]\001-\004\n\/]|#commen[^]\001-\004\n\/t]|#commen|#com[^]\001-\004\n\/m][^]\001-\004\n\/][^]\001-\004\n\/]*|#com[^]\001-\004\n\/m]|#com|#comm[^]\001-\004\n\/e][^]\001-\004\n\/][^]\001-\004\n\/]*|#comm[^]\001-\004\n\/e]|#comm|#[^]\001-\004\n\/c]([^]\001-\004\n\/][^]\001-\004\n\/]*|)|#co[^]\001-\004\n\/m][^]\001-\004\n\/][^]\001-\004\n\/]*|#co[^]\001-\004\n\/m]|#co|#c[^]\001-\004\n\/o][^]\001-\004\n\/][^]\001-\004\n\/]*|#c[^]\001-\004\n\/o]|#c|#comme[^]\001-\004\n\/n][^]\001-\004\n\/][^]\001-\004\n\/]*|#comme[^]\001-\004\n\/n]|#comme|\\.ano((n[^]\001-\004\n\/]|[^]\001-\004\n\/n])[^]\001-\004\n\/]*|)|\\.an([^]\001-\004\n\/o][^]\001-\004\n\/]*|)|(\\.a[^]\001-\004\n\/n]|(\\.[^]\001-\004\n\/a]|[^]\001-\004\n#.\/][^]\001-\004\n\/])[^]\001-\004\n\/])[^]\001-\004\n\/]*|\\.a|\\.[^]\001-\004\n\/a]|[^]\001-\004\n#.\/][^]\001-\004\n\/]|[^]\001-\004\n#.\/]|\\.|#/ }*
with tree
{ "PHP" } { "CLI Server" } { "Date" } { "filter" } { "iconv" } { "intl" } { "sqlite" } { "sqlite3" } { "Pcre" } { "Pdo" } { "Pdo_mysql" } { "Phar" } { "mail function" } { "SQL" } { "ODBC" } { "Interbase" } { "MySQL" } { "MySQLi" } { "mysqlnd" } { "OCI8" } { "PostgreSQL" } { "Sybase-CT" } { "bcmath" } { "browscap" } { "Session" } { "MSSQL" } { "Assertion" } { "COM" } { "mbstring" } { "gd" } { "exif" } { "Tidy" } { "soap" } { "sysvshm" } { "ldap" } { "mcrypt" } { "dba" } { "opcache" } { "curl" } { "expose_php" = "Off" }
Debug: Augeas[php.ini](provider=augeas): Closed the augeas connection
Error: /Stage[main]/Role::Edge-cms/Augeas[php.ini]: Could not evaluate: Saving failed, see debug
```
