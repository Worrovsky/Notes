https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html

## 1. IoC контейнер

### 1.1 Введение

* IoC - процесс, когда объекты устанавливают свои зависимости через параметры конструкторов, свойства или параметры фабричных методов. Контейнер использует эти механизмы для настройки связей между объектами
* Основные пакеты:
	- org.springframework.beans 
	- org.springframework.context
	- основа BeanFactory, из нее ApplicationContext
* Метаданные конфигурации - совокупность бинов и зависимостей между ними

### 1.2 Контейнер

* Интерфейс org.springframework.context.ApplicationContext
* отвечает за создание и связывание бинов
* информацию о структуре проекта получает читая метаданные
* метаданные могут быть в виде 
	- XML
	- java аннотаций
	- java кода
* реализации ApplicationContext
	- ClassPathXmlApplicationContext
	- FileSystemXmlApplicationContext
* Xml-конфигурация

пример

	<beans>
		<bean id="" class="">
		</bean>
	</beans>

	здесь id - идентификатор бина (контейнер использует его для работы с бином)
	class - полное имя класса бина


#### Создание объекта контейнера

* вызов конструктора с указанием файлов настроек

`ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");`

* можно использовать несколько файлов конфигурации
	- перечислением файлов в конструкторе контекста
	- используя import в xml-файле

напр. так:

	...
	<beans>
    		<import resource="services.xml"/>
    		<import resource="resources/messageSource.xml"/>
    ...

* несколько файлов конфигурации могут представлять разные логические уровни / модули приложения

#### Использование контейнера

* метод `T getBean(String name, Class<T> requiredType)`

`PetStoreService service = context.getBean("petStore", PetStoreService.class);`

* не рекомендуется явно вызывать `getBean()` (лучше через Dependency Injection) , тогда не будет зависимости от Spring API


### 1.3 Бины

* Создаются на основе конфигурации (напр. XML)
* внутри контейнера существуют в виде объектов `BeanDefinition`
* содержат:
	- полное имя класса, реализующего бин
	- элементы конфигурации, определяющие поведение бина (scope, callback)
	- ссылки на другие бины-зависимости
* есть возможность зарегистрировать созданный вручную bean в контейнере (но редко исползуется)

#### 1.3.1 Имена бинов

* каждый бин имеет одно или больше имя
* имя должно быть уникально в пределах контейнера
* в xml-конфигурации имя задается параметром `id`	
* дополнительные можно указать в свойстве `name` (через , ; или пробел)
* если имя не указано явно, генерируется авто, но доступ к бину тогда не получить
* в особых случаях (inner beans, autowiring) имена не задаются
* имена даются по стандартной конвенции ('accountManager', 'accountService', 'userDao' и т. п.)
* можно назначать алиасы
	- `<alias name="fromName" alias="toName"/>`
	- используется в сложных проектах, когда один и тот же бин в разных подсистемах называется по разному

#### 1.3.2 Инстанциация бинов

* при конфигурировании указывается тип(класс) объекта
* для xml-конфигурации через атрибут `class`
* внутреннее представление - свойство `Class` экземпляра `BeanDefinition`
* свойство `Class` может использоваться двумя способами
	- контейнер напрямую создает объект класса через конструктор (аналог `new`)
	- у класса будет вызван статический или не статический фабричный метод. Тип возвращаемого объекта может быть другим 
	
##### создание через конструктор:

* обычно создают JavaBean с конструктором по умолчанию без аргументов
* хотя контейнер может работать с любым классом, без привязки к JavaBean - `<bean id="exampleBean" class=examples.ExampleBean/>`
* `<bean name="anotherExample" class="examples.AnotherExample"/>`
* параметры метода указываются в теге `constructor-arg`

##### создание через статический фабричный метод

* указываем класс в атрибуте `class`
* указываем метод в атрибуте `factory-method`
* параметры метода также указываются в теге `constructor-arg`
* при этом тип бина в общем случае не совпадает с атрибутом `class` 

`<bean id="client" class="examples.Client" factory-method="createInstance"/>`

Пример класса
	
	public class Client {
		public static Client createInstance() {
				 return new Client()
		}
	}

##### создание через фабричный метод экземпляра

* атрибут `class` не указываем
* в атрибуте `factory-bean` указываем класс с фабричным методом (id / имя бина)
* в `factory-method` - сам метод

Пример
	
	<bean id="serviceLocator" class="examples.Locator"/>
	<bean factory-bean="serviceLocator" factory-method="createInstance"/>


### 1.4 Зависимости

#### 1.4.1 Dependency Injection (DI)

##### Основы
* DI - процесс, с помощью которого объекты определяют свои зависимости только через конструкторы, аргументы фабричных методов или свойства (сеттеры)
* контейнер внедряет зависимости, когда создает объекты
* сами объекты не знают / не управляют зависимостями - инверсия контроля
* основано на интерфейсах или абстрактных классах
* преимущества:
	- чище код
	- легче тестировать
* два варианта DI:
	- constructor-based DI
	- setter-based DI


##### Constructor-based

* контейнер вызывает конструктор с аргументами, которые представляют зависимости
* вызов статического фабричного метода с параметрами эквивалентен
* класс обычный POJO

пример
	
	public class SimpleMovieLister {
		private MovieFinder movieFinder; // dependency
          
		public SimpleMovieLister(MovieFinder movieFinder) {
			this.movieFinder = movieFinder;
		}
	}

###### Варианты указания аргументов конструктора

* в общем применяется соответствие по типу
* аргументы ссылочных типов и задаются через ссылки на бины
	- просто перечисляются параметры, в этом же порядке переносятся в конструктор
	
пример

	<beans>
		<bean id="foo" class=x.y.Foo>
			<constructor-arg ref="baz"/>
			<constructor-arg ref="bar"/>
		</bean>
		<bean id="baz" class="x.y.Baz"/>
		<bean id="bar" class="x.y.Bar"/>
	</beans>

* аргументы примитивных типов
	- контейнер сам не сможет вывести тип
	- нужно явно указывать

пример
	
	<bean id=".." class="..">
		<constructor-arg type="int" value="42"/>
		<constructor-arg type="java.lang.String" value="43"/>
	</bean>

* можно указать индекс параметра
	- начинается с 0
	- `<constructor-arg index="0" value="7500000"/>`
* можно указывать имя параметра 
	- `<constructor-arg name="years" value="7500000"/>`
	- работает только если компилировать с флагом debug
	- или использовать аннотацию `@ConstructorProperties` для именования параметров

##### setter-based

* контейнер вызывает сеттеры после вызова конструктора или фабричного метода без аргументов
* класс - обычный POJO с сеттерами
* дополнительно внедрение через сеттеры можно использовать после обычной инициализации через конструктор
* значение указывается в теге `property`
	
пример

	<bean id="..." class="Foo">
		<!-- в классе Foo должны быть методы setBeanOne() и setBeanTwo() -->
		<property name="beanOne" ref="exampleBean"/>
		<property name="beanTwo">
			<ref bean="anotherExampleBean/">
		</property>
	</bean>

##### constructor-based vs setter-based

* рекомендуется:
	- для обязательных зависимостей, немутабельных, не null - constructor-based
	- для дополнительных, изменяемых - setter-based



##### Процесс разрешения зависимостей

* создается `ApplicationContext` и инициализируется метаданными, которые описывают бины. Конфигурация определяется через XML, аннотации или код
* Зависимости бина выражены в виде свойств, аргументов конструкторов или аргументов фабричных методов. Эти зависимости поставляются бину при его создании
* Каждое свойство или аргумент это или значение, или ссылка на другой бин
* Значение свойств/аргументов, не являющиеся бинами, преобразуются в конкретные типы (int, String, boolean и т. п.)
* Контейнер проверят правильность конфигурации. Но создание бинов и их свойств не происходит. При создании контейнера создаются только singleton-бины. Остальные создаются по требованию(тогда же создаются их зависисмости)
* Есть проблема циклической зависимости, когда А внедряется через конструктор в В, В через конструктор в А. Это будет исключение в runtime. Можно избежать, если внедрять через setter'ы.

#### 1.4.2 Зависимости и конфигурация

* Свойства бинов и аргументы конструкторов:
	- как ссылки на другие бины
	- как встроенные типы (примитивные, String)
	- через `<property/>`
	- через `<constructor-arg/>`

##### Примитивные типы

	<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource">
	    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
	    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
	    <property name="username" value="root"/>
	    <property name="password" value="masterkaoli"/>
	</bean>

* можно использовать p-namespace 
	- `<bean id="..." class="..." p:driverClassName="com.mysql.jdbc.Driver"/>`

##### Элемент idref

Способ передачи ссылки на бин. Контейнер на этапе развертывания проверяет наличие бина по ссылке.

	<bean id="theTargetBean" class="..."/>
	<bean id="theClientBean" class="...">
		<property name	="targetName">
			<idref bean="theTargetBean"/>
		</property>
	</bean>

следующий код делает то же самое. Но проверки нет и после создания бина при попытке установить значение может возникнуть исключение 

	<bean id="theTargetBean" class="..."/>
	<bean id="theClientBean" class="..."/>
		<property name="targetName" value="theTargetBean"/>
	</bean>

Рекомендуется первый вариант (из-за дополнительной проверки)

##### Ссылки на другие бины

* через элемент `ref` внутри тегов `constructor-arg` или `property`
* используется как зависимость для других бинов
* инициализируется по требованию (ср. с бином, объявленным тегом `bean`)
* область видимости, валидация зависит от того, внутри каких тегов используется (`bean`, `local`, `parent`)
* два варианта
	- атрибутом `<property name=".." ref=".."/>`
	- тэгом `<ref bean="..."/>`

##### Inner bean

* тег `<bean/>` внутри `<constructor-arg/>` или `<property/>`
* служит для объявления бина вместо использования ссылки (`<ref/>`)
* для внутреннего бина не нужно указывать id/name (если есть - не используется)
* доступ к внутреннему бину получить нельзя
* флаг `scope` игнорируется

пример

	<bean id="outer" class="...">
        <property name="target">
	        <bean class="com.example.Person"> 
	            <property name="name" value="Fiona Apple"/>
	        </bean>
	    </property>
	</bean>

##### Коллекции

* теги `<list/>`, `<set/>`, `<map/>`, `<props/>` соответствуют коллекциям List, Set, Map, Properties
* list - любой список, массив и т.п.
* set - не обязательно java.util.Set, просто уникальность значений
* map - ключ-значение любых типов
* props - ключ-значение строкового типа
* 

пример

	<bean id="" class="">
		<!-- results in a setSomeProperies(java.util.Properties) call -->
		<property name="someProperies">
			<props>
				<prop key="admin">adm@mail.com</prop>
				<prop key="user">us@mail.com</prop>
			</props>
		</property>
		
		<property name="someList">
			<list>
				<value>a list element</value>
				<ref bean="someBean">
			</list>
		</property>
		
		<property name="someMap">
	        <map>
	            <entry key="an entry" value="just some string"/>
	            <entry key ="a ref" value-ref="myDataSource"/>
	        </map>
	    </property>
	
	    <property name="someSet">
	        <set>
	            <value>just some string</value>
	            <ref bean="myDataSource" />
	        </set>
	    </property>
	
	</bean>

##### Null и пустые строки

* пустая строка просто указывается `<property name="email" value=""/>`
* для null специальный элемент `<null/>`

:

	<bean class="ExampleBean">
		<property name="email">
			<null/>
		</propery>
	</bean>


##### Составные свойства

	<bean id="something" class="examples.FirstClass">
    	<property name="foo.bar.baz" value="123" />
	</bean>

* если у свойства есть другое свойство, можно указывать через точку
* будет работать, пока все свойства в цепочке не null, иначе получим NullPointerException

#### 1.4.3 Использование depends-on

* Обычно бин зависит от другого, когда другой бин является его свойством
* но может быть неявная зависимость, когда нужно чтобы один бин был создан перед другим
* атрибут `depends-on` заставляет инициализировать бины, указазанные в атрибуте, раньше чем бин, содержащий атрибут
* можно передавать список бинов (, ; пробел - разделители)
* `<bean id="beanOne" class="BeanOne" depend-on="beanTwo, beanTen"/>`


#### 1.4.4 Ленивая инициализация бинов

* по умолчанию ApplicationContext инициализирует бины-синглетоны сразу при своей инициализации
* это удобно, т. к. синглетоны - это обычно конфигурации/окружение и ошибки будут видны сразу
* можно отключить это поведение `<bean id="lazy" class=".." lazy-init ="true">`
* можно вообще включить ленивую инициализацию для всего контейнера
`<beans default-lazy-init="true">`


#### 1.4.5 Autowiring

##### Основы

* Автоматическое связывание бинов
* через атрибут `autowire` элемента `<bean/>`
* Преимущества:
	- упрощение (нет необходимости указывать свойства / параметры)
	- обновление конфигурации без модификации конфигурации
* Режимы:
	- no - по умолчанию, связывание обычным способом через `ref`
	- byName - Spring ищет бины с именами, равными именам свойств
	- byType - если в контейнере есть единственный бин с типом, соответствующим типу свойства, он и будет установлен. Если несколько - исключение. Если нет вообще - ничего не будет
	- constructor - аналогично byType, но для аргументов конструкторов

##### Преимущества / недостатки

* Смешивать автосвязывание и обычный режим не очень: может быть неочевидным, что среди обычного внедрения через XMl есть пара через связывание
* Нельзя связывать примитивные типы
* Явное внедрение через свойства / конструкторы перекрывает автосвязывание
* Автосвязывание явно не документирует связи
* Средства документирования не учитывают автосвязывание
* Не работает при наличии нескольких бинов одного типа (byType)

##### Исключение бина из области Autowiring

* можно убрать бин из кандидатов на связывание для типа byType
* `<bean id=".." class=".." autowire-candidate="false"`
* можно на уровне контейнера через регулярные выражения отбирать бины для связывания
`<beans default-autowire-candidates="*Repository">`
* но `autowire-candidate` в теге `bean` имеет больший приоритет


#### 1.4.6 Инъекция методов

	<ОТЛОЖЕНО ИЗУЧЕНИЕ ПОКА>

### 1.5 Области видимости (scope) бинов

#### Основы

Варианты областей видимости
* singleton - (по умолчанию) объект в единственном экземпляре на контейнер
* prototype - любое количество экземпляров бина
* request - свой экземпляр на каждый HTTP запрос. Только для web-контекста
* session - свой экземпляр на HTTP сессию. Только для web-контекста
* application - один экземпляр на ServletContext. Только для web-контекста
* websocket - один экземпляр на WebSocket. Только для web-контекста
* пользовательская область видимости

* указывается при объявлении бина
`<bean id=".." class=".." scope="prototype">`

#### Singleton

* Контейнер всегда возвращает один и тот же экземпляр на все запросы этого бина
* Режим по умолчанию
* Если не указана ленивая инициализация, бины с этой областью видимости создаются при инициализации контейнера

#### Prototype

* На каждый запрос бина (`getBean()` или инъекция) создается новый экземпляр
* Создается в момент запроса
* Этот тип подходит для бинов, имеющих состояние
* Контейнер поддерживает укороченный жизненный цикл: есть на создание, но не вызывает destroy-методы. Освобождением бинов с prototype типом занимается клиент


### 1.6 Настройка поведения бинов

#### 1.6.1 Методы жизненного цикла

	<ОТЛОЖЕНО ИЗУЧЕНИЕ ПОКА>

### 1.7 Наследование описания бинов

* можно через xml, можно программно через класс `ChildBeanDefinition`
* дочерний бин наследует все свойства, значения родительского, может их переопределять / добавлять новые
* для этого в дочернем бине в атрибуте `parent` указывается имя родительского бина
* следующие свойства не наследуются:
	- depend on
	- autowire режим
	- singleton
	- lazy init
* в родительском бине можно выставить атрибут `abstract="true"`:
	- если при этом не указать класс бина, бин будет истинным шаблоном и создать экземпляр такого бина нельзя
	- можно указать класс, тогда в случае синглтона, контейнер не будет заранее инициализировать этот бин (обычные singleton-бины инициализируются при старте контейнера) 

### 1.8 

### 1.9 Конфигурирование на аннотациях

#### Основы 

* Аннотации vs xml:
	- аннотации короче, больше возможностей(?)
	- xml не трогают код, позволяют менять настройки без перекомпиляции
* при совместном использовании: сначала применяются аннотации, затем xml, т. е. xml настройки перезаписывают настройки от аннотаций
* Для включения аннотаций добавляем в xml-файл в элемент `<beans/>` строку `<context:annotation-config/>` (+ область имен namespace `xmlns:context=...`)


#### 1.9.1 @Required

* Применяется к сеттерам в бинах
* отмечает, что свойство должно быть установлено во время конфигурации (через autowiring или явно, неважно)
* контейнер выбросит исключение, если свойство не будет установлено
* типа защита от NPE
* в 5.1 deprecated: для обязательных свойств можно использовать инъекцию через конструктор (если бин создается контейнером и включена поддержка аннотаций, тогда даже не надо явно указывать связывание при наличии единственного констрруктрора)


#### 1.9.2 @Autowired

* Автоматическое подтягивание зависимостей
* может использоватся для
	- конструкторов
	- сеттеров 
	- просто полей
	- полей массивов или коллекций
	- полей типа Map c ключом типа String (ключами будут имена, значениями - экземпляры бинов)
	- параметров конструкторов, методов
* вообще можно использовать для любого метода (не обязательно сеттера) и с любым количеством параметров (привязка по типу)
* если связывание не смогло найти подходящий бин - исключение(!проверить!)
* можно отключить обязательность `@Autowired(required = false)`, тогда если не найдены бины, поля будут проинициализированы значениями по умолчанию
* когда @Autowired применяется к нескольким конструкторам объекта:
	- просто нельзя объявлять на нескольких конструкторах
	- один конструктор допускается с `requred=true`, остальные false
	- выбирается наиболее "заполненный" конструктор
	- если только один конструктор - он будет вызван, даже без аннотаций
	- если аннотированные конструкторы не подошли, и есть простой конструктор без параметров, он будет вызван
* @Autowired с `requred=true` это замена @Required

#### 1.9.3 @Primary как уточнение для autowiring

* используется для бина (совместно с @Bean)
* означает что, если при связывании (по типу) есть несколько кандидатов, будет выбран с этой аннотацией (если он один)
*  напр. `@Bean @Primary public MovieCatalog firstMovieCatalog() {...}`
*  то же самое можно через xml `<bean ... primary="true"/>`

#### 1.9.4 @Qualifier как уточнение для autowiring

* используется совместно с @Autowired для устранения неоднозначностей
* напр. так `@Autowired @Qualifier(<beanId>)` - уточнение по имени бина
* можно применять для отдельного параметра `@Autowired public void someMethod(@Qualifier("main") MovieCatalog catalog, ...)`
* можно по другому
	- в xml в бине `<bean ...> <qualifier value="someValue"/>`
	- при внедрении `@Autowired @Qualifier("someValue")`
* квалификаторы не обязательно должны быть уникальны (напр. для внедрения в коллекции)
	
##### Собстенные квалификаторы

	// создаем аннотацию
	import java.lang.annotation.*
	@Target({ElementType.FIELD, ElementType.PARAMETER})
	@Retention(RetentionPolicy.RUNTIME)
	@Qualifier
	public @interface Genre {
		String value();
	}
    
    // используем при автосвязывании
    public class MovieRecomender {
    	@Autowired
    	@Genre("Action")
    	private MovieCatalog actionCatalog;
    	...
    }
    
    // детализируем бины в xml-конфиге с указанием типа квалификатора и значения
	...
		<bean ...> 
			<qualifier type="Genre" value="Acction"> // можно тип сокращенно
		</bean>
		<bean ...>
			<qualifier type="com.example.Genre" value="Comedy"> // можно полный
		</bean>

Можно создавать простые аннотации без значений, просто по типу (опускаем везде значение, тело аннотации - пустое)

Можно дополнительно еще значения в аннотации



	










#### 1.9.5 Универальные типы как неявная форма автосвязывания

Если есть типизированный интерфейс, бины реализуют этот интерфейс с конкретным типом (не совпадают у бинов), тогда чтобы внедрить бины в поля с конкретными типами достаточно @Autowired (квалификаторы не нужны)

#### 1.9.6 Пользовательский CustomAutowireConfigurer

#### 1.9.7 @Resource

* JSR-250, паттерн из Java EE
* `javax.annotation.Resource`
* аннотация в качестве параметра принимает имя бина `@Resource("myBean")`
* если не указан, автоматически заполняет по имени поля или из сеттера
* если не находит по имени, как и @Autowired ищет по типу

#### 1.9.8 Использование @PostConstruct, @PreDestroy

* также из JSR-250
* еще одна альтернатива для жизненного цикла
* отмечаются методы бина для запуска после создания и перед уничтожением



### 1.10 Сканирование classpath и управление компонентами

#### 1.10.1 Аннотация @Component и ее "дочерние" аннотации

* @Component - общая аннотация для любого компонента Spring
* @Repository, @Service, @Controller - специализации @Component для конкретных применений
* Лучше применять конкретные аннотации вместо общей @Component

#### 1.10.2 Мета-аннотации и составные аннотации

мета-аннотация - это аннотация, применяемая к другой аннотации
так напр. в самом Spring @Component применен к аннотации @Service
	
	@Target(ElementType.TYPE)
	@Retention(RetentionPolicy.RUNTIME)
	@Component
	public @interface Service{ ...}

Можно комбинировать мета-аннотации для получения составных аннотаций. Например в @RestController из Spring MVC иcпользует @Controller и @ResponceBody


#### 1.10.3 Автоматическое определение классов и регистрация bean definition

* контекст может автоматически определять классы бинов, помеченные @Component или дочерними, и загружать их описания
* для этого конфигурация должна быть настроена на автодетектирование:
	- нужен класс-конфигурация с аннотацией @Configuration
	- к нему же добавляется аннотация @ComponentScan с указанием, где искать бины
	- `@ComponentScan(basePackages="com.example")` или просто `@ComponentScan("com.example")`
	- можно несколько пакетов
* альтернативно через xml в теге `<beans/>`	
	- `<context:component-scan base-packages="com.example"/>`
	- при этом `component-scan` включает в себя функциональность `annotation-config`
	- также автоматически включаются `AutowiredAnnotationBeanPostProcessor ` и `CommonAnnotationBeanPostProcessor`

#### 1.10.4 Фильтры для настройки сканирования

* дополнительно ограничить компоненты
* фильтры добавляются в 
	- параметры `includeFilters`, `excludeFilters` аннотации `@ComponentScan`
	- дочерние теги `include-filter`, `exclude-filter` тега `component-scan`
* каждый фильтр содержит атрибуты тип и значение / выражение
* варианты типов
	- аннотации (по умолчанию) - в качестве значения - аннотация, которую контрейнер будет искать для включения/исключения бина из кандидатов
	- по классу/интерфейсу - значение - класс, объекты этих классов - кандидаты
	- через аспекты
	- через регулярные выражения - на имя класса
	- пользовательские

Пример:

	@Configuration
	@ComponentScan(basePackage="com.example",
			includeFilters = @Filter(type=FilterType.REGEX, pattern=".*Stub"))
			excludeFilters = @Filter(Repository.class)) // исключаем все с аннотацией @Repository
	public class Config { ... }

Можно отключить фильтры по умолчанию, при этом перестанут определятся классы с аннотациями @Component, @Service и т. п.:
	
	@ComponentScan( ... useDefaultFilters=false)



#### 1.10.5 Определение бинов внутри компонента

Также как и внутри конфигураци @Configuration можно определять бины с помощью аннотации @Bean через фабричный метод

	@Component
	public class FactoryMethodComponent {
		@Bean
		@Qualifier("public")
		public TestBean publicInstance() {
			return new TestBean("publicInstance");
		}
	}

#### 1.10.6 Именование компонентов при автодетектировании

* Аннотации @Component и дочерние имеют параметр `value`. Если его значение задано, оно становится именем бина. 
* Если параметр не установлен, имя определяется BeanNameGenerator (имя класса с прописной буквы)
* поведение BeanNameGenerator можно переопределять

Примеры

	@Service("myMovieListener") // имя задано явно
	public class SimpleMovieListener {..}
	
	@Repository
	public class MovieFinderImpl ... // имя определяется как "movieFinderImpl"



#### 1.10.7 Область видимости для компонентов

* по-умолчанию singleton как и для обычных бинов
* изменить можно через аннотацию @Scope, напр. `@Scope("prototype")`

#### 1.10.8 Квалификаторы

#### 1.10.9 Индексирование компонентов-кандидатов

* Для ускорения запуска больших приложений на этапе компиляции создается статический список всех компонентов и контекст при запуске обращается к нему, не сканируя classpath
 

### 1.11 Аннотации по стандарту JSR-330 (@Inject, @Named)

### 1.12 Конфигурация на основе Java

#### 1.12.1 Основные концепции: @Bean, @Configuration

* аннотация @Bean отмечает метод, который создает, настраивает объект. Этот объект будет включен в контейнер (т. е. будет бином)
* может использоваться внутри @Component или (чаще) @Configuration
* аннотация @Configuration отмечает класс, назначение которого - определять бины

#### 1.12.2 Spring контейнер через AnnotatoinConfigApplicationContext

* такой контекст способен находить бины, настроенные через
	- @Configuration классы
	- @Component классы
	- с аннотацией по JSR-330
* создание контекста из конфигурационного класса (@Configuration)
	- `ApplicationContext context = new AnnotatoinConfigApplicationContext(AppConfig.class);`
	- при этом бин класса AppConfig уже в контейнере
* или можно перечислить классы с аннотациями @Component, @Inject
	- `ApplicationContext ctx = new AnnotatoinConfigApplicationContext(MyComponent1.class, Dependency1.class);`
* 2-й вариант: создать через пустой конструктор, затем методом `register()`

Например:

	ApplicationContext ctx = new AnnotatoinConfigApplicationContext();
	ctx.register(AppConfig.class);
	ctx.register(MyComponent.class);


* Разрешение сканирования: 
	- через аннотацию @ComponentScan("пакеты") у конфигурационного класса 
	- через вызов метода `scan(String package)`:
		+ создаем контекст через пустой конструктор
		+ вызываем `scan()` (если есть конфигурационный класс, он подтягивается, т. к. сам неявно @Component)
		+ вызываем метод refresh() для подгрузки бинов из конфигурации


#### 1.12.3 Аннотация @Bean

* применяется к методам (фабричный очевидно)
* аналог элемента <bean/>
* можно использовать внутри @Component или @Configuration
* при этом фабричный метод может иметь параметры, эти параметры без дополнительных аннотаций автосвязываются с бинами по типу
* если внутри метода c аннотацией @Bean вызывается другой метод с аннотацией @Bean, Spring перехватит такой вызов и заменит его бином сразу

##### Методы жизненного цикла

* поддерживается JSR-250 (javax.annotation.*)
	- в классах, реализующих бины, можно отмечать методы аннотациями @PostConstruct и @PreDestroy
* методы цикла Spring через реализацию интерфейсов
	- InitializingBean
	- DisposableBean
	- Lifecycle
* методы интерфейсов семейства *Aware (BeanFactoryAware, BeanNameAware и т. п.)
* произвольные методы класса через параметры `initMethod` и `destroyMethod` аннотации @Bean
* если класс имеет публичные методы `close()` или `shutdown()`, Spring без дополнительных указаний будет их вызывать. Отключить можно `destroyMethod=""`

##### Область видимости

* аннотация @Scope
* + дополнительно через прокси 

##### Имена, псевдонимы, описание

* `@Bean(name = "myBean")`
* можно указать несколько псевдонимов `@Bean({"data1", "alias2"})`
* описание можно для целей мониторинга `@Description("Some bean for ...")`




#### 1.12.4 Аннотация @Configuration

* применяется к классам
* означает, что класс содержит определения бинов

##### Межбинные зависимости

Если один бин имеет в зависимости другой, это выражается просто вызовом одного метода другим

	@Configuration
	class Config {
		@Bean 
		public BeanOne beanOne() {
			return new BeanOne(beanTwo());
		}
		@Bean
		public BeanTwo beanTwo() {
			return new BeanTwo();
		}
	}

Такое работает только внутри @Configuration, не работает внутри напр. @Component 

##### Особенности внутренней работы

	@Configuration
	...
	@Bean
	public Service getClient1() {
		return new ServiceImpl(clientDao());
	}
	@Bean
	public Service getClient2() {
		return new ServiceImpl(clientDao());
	}
	@Bean
	public ClientDao clientDao(){
		return new ClientDao();
	}

* Здесь объект `ClientDao ` создается один раз, поскольку это бин и область видимости singleton
* Spring использует прокси-классы 
* с помощью библиотеки CGLIB (часть фреймворка)
* здесь классы не могут быть final (проксирование реализуется через наследование) 	(не удалось воспроизвести)
* внутри @Component поведение другое: через явную иньекцию
