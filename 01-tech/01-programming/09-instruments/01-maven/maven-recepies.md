### 1.1 Префикс плагина для запуска целей

`maven-${prefix}-plugin` так задаются id для стандартных плагинов
`${prefix}-maven-plugin` так плагины от сторонних разработчиков

при определении префикса отбрасываются `maven`, `plugin`

можно явно указать префикс (! при создании собственного плагина)

    <plugin>
         <artifactId>...
         <configuration>   
            <goalPrefix>somePrefix</goalPrefix>
        </configuration>
    </plugin>

[stackoverflow](https://stackoverflow.com/questions/40205664/how-does-maven-plugin-prefix-resolution-work-why-is-it-resolving-findbugs-but)

### 1.2 Конфигурация плагинов

#### Вариант 1.

элемент `configuration` в pom-файле

    <build>
        <plugins>
            <plugin>
                <artifactId>
                ...
                <configuration>
                    ...
    ...

Элементы конфигурации определяются по именам полей в Mojo-классе

#### Вариант 2.

Можно указывать конфигурацию параметром в CLI через `-D`.
    
    mvn myplugin:someGoal -Dparam=...

Как именно называется параметр см. в коде `@Parameter(property = "param")`

напр. `frontend.npm.arguments` для цели `npm` у frontend-maven-plugin



#### Тег executions

Для настройки плагинов, выполнения в разные фазы и т. п.

!! не работает в блоке `<pluginManagment` 

    <build>
        <plugins>
            <plugin>
            ...
            <executions>
                <execution>
                    <id>...</id>
                    <phase>...</phase>
                    <configuration>
                        ...
                    </configuration>
                    <goals>
                        <goal>...</goal>
                    </goals>
                </execution>
            </executions>
    ...

Если есть элемент `phase`, цель будет выполнена в указанную фазу
Если нет - тогда в фазу по умолчанию для этой цели
Если у цели нет фазы по умолчанию - вообще не будет выполнена

Id для разных execution в пределах одного плагина должны быть уникальны

#### Запуск через CLI

    mvn myPlugin:myGoal@executionId

#### Тег inherited для плагинов

Определяет будет ли плагин применятся для дочерних pom
по умолчанию - `true`


### 1.3 Жизненный цикл

Цикл -> фазы -> цели

Встроенные циклы: clean, default, site

через команду `mvn` вызываются фазы

    mvn clean delpoy

или конкретные цели

    mvn clean:clean

элемент `<packaging>` меняет состав целей, в зависимости от значения




### 1.4 Модули

[Mvn by example](https://books.sonatype.com/mvnex-book/reference/multimodule.html)

Группировка внутри проекта (иерархия папок). Есть общий pom (корневой), есть pom для каждого модуля. 

В корневом pom есть координаты проекта и перечень модулей: 

    <groupId>org.example</groupId>
    <artifactId>simple-parent</artifactId>
    <modules>
        <module>simple-weather</module>
        <module>simple-webapp</module>
    </modules>

Также можно объявлять настройки и зависимости, которые будут наследоваться модулями

Модуль имеет ссылку на родительский pom и свой `artifactId`. Может иметь свои зависимости/настройки и т. п.

    parent>
        <groupId>org.example/groupId>
        <artifactId>simple-parent</artifactId>
    </parent>
    <artifactId>simple-webapp</artifactId>

При запуске build проекта maven выполняет цели каждого модуля, причем сортирует модули. 

Сортировка определяется следующими отношениями:

* зависимость одного модуля от другого
* когда модуль является плагином для другого модуля
* зависимость плагина от другого модуля
* настройки билда на другой модуль
* просто порядок перечисления модулей в pom

## 2. Профили

[docs](https://maven.apache.org/guides/introduction/introduction-to-profiles.html)

### 2.1 Типы профилей

* на уровне проекта (задаются в pom.xml)
* на уровне пользователя `%USER_HOME%/.m2/settings.xml`
* глобальные профили `${maven.home}/conf/settings.xml`

### 2.2 Активация

1) из CLI через `-P`

    mvn phase -P profile-1, profile-2

2) на основе состояния окружения: секция `activation` в профиле. Ограниченный набор свойств: версия jdk, наличие системного свойства или значение системного свойства

    <profiles>
        <profile>
            <activaton>
                <jdk>1.8</jdk>
            ...

Вот при наличии свойства `debug` с любым значением

    <activation>
        <property>
            <name>debug</name>
        ...
запускаем `mvn ... -Ddebug=smthg`

При отсутствии свойства
    
    ...
    <name>!debug</name>

По свойству с указанным значением (можно инвертировать)

    <property>
        <name>debug</name>
        <value>true</value>
    ...

По наличию/отсутствию файлов 

    <activation>
      <file>
        <missing>target/generated-sources/axistools/wsdl2java/org/apache/maven</missing>
      </file>
    </activation>

Явное указание активности

    <profile>
        <id>profile-1</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        ...

На практике если активировать другой профиль напр. через `property`, профиль с `activeByDefault=true` будет неактивным


9) (не рекомендуется из-за невоспроизводимости) через секцию `<activeProfiles>` в файле `%USER_HOME%/.m2/settings.xml` (id указываются)


### 2.3 Настраиваемые свойства

для профилей во внешних файлах (глобальные, на уровне пользователя) можно менять только `<repositories>`, `<pluginRepositories>` и `<properties>`

для профилей на уровне pom-файла:

* зависимости `<dependencies>`
* плагины и настройки `<build><plugins>`
* репозитории `<repositories>`, `<pluginRepositories>`
* свойства `<properties>`
* модули `<modules>`   
* и др

### 2.4 Порядок профилей

Свойства из активных профилей перезаписывают (или дополняют в случае коллекций) глобальные свойства 

Порядок: определенные позже (ближе к концу файла) имеют приоритет над определенными ранее

### 2.5 Получение активных профилей

    mvn help:active-profiles

или

    mvn help:all-profiles

дополнительно можно получить эффективный pom

    mvn help:effective-pom
    mvn help:effective-pom -Doutput 1.xml // c выводом в файл

можно в этих коммандах активировать профили через `-P` или через переменные `-Denv`


## 3. build блок

### 3.1 Основы

2 вида: в корне проекта и в профилях

    <project>
        <build>...</build>

        <profiles>
            <profile>
                <build>...</build>
            </project>
        </profiles
    </project>

Также делится на **BaseBuild** и просто **Build**

### 3.2 Набор BaseBuild

**defaultGoal**
**directory** - куда будут помещена файлы после билда
**finalName** - как будет называться результирующий файл
**filter** - фильтры

вот пример со значениями по умолчанию

    <build>
        <defaultGoal>install</defaultGoal>
        <directory>${basedir}/target</directory>
        <finalName>${artifactId}-${version}</finalName>
    </build>    

**resources** - управляет ресурсами: где исходные ресурсы, куда их помещать, что включать / исключать

**plugins** - управление плагинами

**pluginManagement** - задает настройки плагинов для дочерних пом в родительском пом-файле. Если это корневой проект - никак не влиет. Дочерние пом-файлы будут наследовать настройки, но все равно надо явно включать плагин в дочернем

###3.3 Набор Build

**directories** набор директорий: source, test, output и др.

**extensions** - расширения