# Linux Foundation Certified System Administrator (LFCS)

## Lesson 15 Логгирование

#### Основы

* основные варианты сервисов
    - `syslog` 
        + сохраняет данные в `/var/log`
    - syslog-ng
    - rsyslog
    - systemd-journald 
        + runtime-версия (при перезагрузке теряются данные)
    - logrotate - вспомогательный сервис, следит напр. за размерами лога

#### systemd-journald

* результаты используются напр. при вызове `systemctl status`
* `journalctl` - просмотр логов файловой системы
    - содержится в памяти, не сохраняется при перезагрузке по умолчанию
    - сделать сохраняемым: `/var/log` `mkdir journal` (root)
    - `/ ...` - поиск (основан на less)
* настройки `/etc/systemd/journald.conf` (`man journald.conf`)

#### rsyslogd

* настройки состоят из селектора, приоритета и действия
    - селекторы - предопределенные идентификаторы сервисов, для которых возможно логгирование (см. `man rsyslog`)
        + `kern`
        + `authpriv`
        + `mail` и др.
    - приритет
        + emerg
        + crit
        + debug
    - действия
        + обычно имя файла в `/var/log/`, куда будут сохранятся логи
        + модули вывода типа `:omusrmsg: `
* `systemctl status rsyslog`
* конфигурация в `/etc/rsyslog.conf`
    - загрузка модулей, обеспечивающих логгирование
        + `$ModLoad imuxsock` - локальное системное логгирование через команду `logger`
    - правила
        + `*.info;mail.none     /var/log/messages` - все сообщения с приоритетом info, кроме почтовых будут в файл messages
        + `mail.*   -/var/log/maillog` - все почтовые сообщения - в файл maillog. `-` - работа через буфер (снижает нагрузку при интенсивном логгировании (почтовый сервер)) 
* отправка лога `logger -p <Приоритет> <Сообщение>`

#### logrotate

* очистка и архивирование файлов логов
* работает на основе crond (`/etc/cron.daily/logrotate`)
* настройки `/etc/logrotate.conf`
    - частота очистки, архивирования
    - специальные настройки для отдельных файлов