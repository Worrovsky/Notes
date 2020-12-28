# Linux Foundation Certified System Administrator (LFCS)

## Lesson 18 Mandatory access control

#### Основы 

* необходимость ограничения доступа:
    - есть сервисы (напр. http сервисы), которые запущены под каким-то пользователем
    - у этого пользователя должны быть ограничены права (даже напр. на создание файлов)
    - такие сервисы обычно `nologin`, но как проверить все?
    - основной способ: запрещаем все, разрешаем минимум
* 2 варианта реализации
    - SELinux (Red Hat, но  доступен и в других)
        + сложен
        + жестко ограничивает (сразу все запрещает)
        + на политиках
        + предпочтительнее?
    - AppArmor (SUSE, Debian)
        + проще
        + нужно явно указывать сервисы для ограничений
        + на профилях
