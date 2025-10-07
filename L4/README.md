**Современные технологии серверной разработки**
## Лабораторная работа №4 (10 ч)

**Тема:** Разработка RESTful-сервиса с использованием спецификации JAX-RS.

**Цель работы:** изучить ключевые моменты спецификации *JAX-RS* для создания *RESTful*-сервиса с открытым *API*. Разработать *RESTful*-сервис для работы с БД.

### Теоретические сведения

#### Что такое *RESTful*-сервис
*RESTful*-сервис - это веб-сервис, построенный на архитектурных принципах *REST* (*Representational State Transfer* - передача репрезентативного состояния или передача "самоописываемого" состояния). Другими словами, *REST* - это набор правил того, как программисту организовать написание кода серверного приложения, чтобы все системы легко обменивались данными и приложение можно было масштабировать.

Он предоставляет доступ к данным и операциям над ними через стандартные *HTTP*-запросы. Такие сервисы обрабатывают запросы от клиентов (например, веб-приложений или мобильных приложений) и возвращают данные, как правило, в формате *JSON*. 

*RESTful*-сервисы применяются для обмена информацией между различными системами, обеспечивая простой и масштабируемый способ взаимодействия между ними. *RESTful*-сервисы используются в распределенных системах и в микросервисной архитектуре.

#### Про спецификацию *JAX-RS*
*JAX-RS* (*Java API for RESTful Web Services*) - это спецификация *Java*, предназначенная для упрощения разработки *RESTful* веб-сервисов. Она определяет набор аннотаций и интерфейсов, позволяющих создавать *REST*-сервисы с минимальными усилиями, используя стандартные *HTTP*-методы и работая с форматами данных, такими как *JSON* или *XML*. 

С помощью *JAX-RS* можно легко описывать маршруты и соответствующие им методы обработки *HTTP*-запросов, а также автоматически преобразовывать *Java*-объекты в *JSON* и обратно. 

Спецификация используется в таких фреймворках, как *Jersey* и *RESTEasy*, и часто применяется в корпоративной *Java*-разработке для построения *API* и микросервисов.

#### По разработке приложения
Проект создаем с использованием *Maven*. Архетип: `maven-archetype-webapp`.

Для разработки *RESTful*-сервиса с использованием спецификации *JAX-RS* рекомендуется использовать ее реализацию *Jersey* (простыми словами: *JAX-RS* - это набор интерфейсов и аннотаций, *Jersey* - это реализация этих интерфейсов).

**Зависимости для *Jersey* (все дополнительные зависимости *Jersey* должны быть той же версии, что и *Jersey Core*):**

```XML
<dependencies>
	<!-- https://mvnrepository.com/artifact/org.glassfish.jersey.core/jersey-server -->
	<!-- Jersey Core -->
	<dependency>
		<groupId>org.glassfish.jersey.core</groupId>
		<artifactId>jersey-server</artifactId>
		<version>3.1.10</version>
	</dependency>

	<!-- https://mvnrepository.com/artifact/org.glassfish.jersey.containers/jersey-container-servlet -->
	<!-- Jersey Servlet Integration -->
	<dependency>
		<groupId>org.glassfish.jersey.containers</groupId>
		<artifactId>jersey-container-servlet</artifactId>
		<version>3.1.10</version>
	</dependency>

	<!-- https://mvnrepository.com/artifact/org.glassfish.jersey.media/jersey-media-json-jackson -->
	<!-- Jackson для JSON -->
	<dependency>
		<groupId>org.glassfish.jersey.media</groupId>
		<artifactId>jersey-media-json-jackson</artifactId>
		<version>3.1.10</version>
	</dependency>
	
	<!-- https://mvnrepository.com/artifact/org.glassfish.jersey.inject/jersey-hk2 -->
	<!-- Dependency Injection для Jersey -->
	<dependency>
		<groupId>org.glassfish.jersey.inject</groupId>
		<artifactId>jersey-hk2</artifactId>
		<version>3.1.10</version>
	</dependency>

	<!-- https://mvnrepository.com/artifact/jakarta.servlet/jakarta.servlet-api -->
	<!-- Servlet API (для Tomcat) -->
	<dependency>
		<groupId>jakarta.servlet</groupId>
		<artifactId>jakarta.servlet-api</artifactId>
		<version>6.1.0</version>
		<scope>provided</scope>
	</dependency>
</dependencies>
```

#### Создание точки старта приложения
Создайте класс (например, `Main`). Он должен наследоваться от класса `Application`. Также укажите префикс для всех *URL* с помощью аннотации `ApplicationPath`:

```Java
@ApplicationPath("/api/v1")  
public class Main extends Application {  
      
}
```

Далее создайте тестовый контроллер и класс `Test` с любыми полями. С помощью аннотации `Path` укажите *URL*, запросы по которому будет обрабатывать этот контроллер. Также с помощью аннотации `Produces` укажите тип данных, возвращаемые методами контроллера (формат *JSON*):

```Java
@Path("/test")  
@Produces(MediaType.APPLICATION_JSON)  
public class TestController {  
    @GET  
    public Response get() {  
        return Response.ok(new Test("test")).build();  
    }  
}
```

Выполните сборку проекта и развертывание в *Apache Tomcat*. Проверьте работоспособность сервиса, открыв браузер (или *Postman*) и введите *URL*:

```
http://<хост:порт>/название_приложения/api/v1/test
```

Сервис должен вернуть объект `Test` в формате *JSON*.

Более подробно про реализацию обработчиков можно прочитать в [документации к *Jersey* (глава 3)](https://eclipse-ee4j.github.io/jersey.github.io/documentation/latest/user-guide.html#jaxrs-resources).

#### ORM фреймворк Hibernate
Для использования фреймворка *Hibernate* необходимо подключить следующие зависимости:

```XML
<!-- https://mvnrepository.com/artifact/org.hibernate.orm/hibernate-core -->  
<dependency>  
    <groupId>org.hibernate.orm</groupId>  
    <artifactId>hibernate-core</artifactId>  
    <version>6.6.13.Final</version>  
</dependency>  
  
<!-- https://mvnrepository.com/artifact/com.mysql/mysql-connector-j -->  
<dependency>  
    <groupId>com.mysql</groupId>  
    <artifactId>mysql-connector-j</artifactId>  
    <version>9.2.0</version>  
</dependency>
```

- `hibernate-core` является реализацией интерфейсов *Jakarta Persistence API* (*JPA*) - это набор интерфейсов для работы с БД
- `mysql-connector-j` - драйвер для работы с БД *MySQL*
	- если используете другую БД, то необходимо подключить другую зависимость, соответствующую этой БД

Далее необходимо создать файл конфигурации для *JPA*. В папке `resources/META-INF` создайте файл `persistence.xml`:

```XML
<persistence xmlns="https://jakarta.ee/xml/ns/persistence"
             version="3.0">

	<!-- Указываем название конфигурации -->
    <persistence-unit name="mysql-persistence-unit">

        <!-- Здесь указываем классы-сущности -->
        <class>gstu.thehwk.model.Student</class>
        <class>gstu.thehwk.model.Group</class>

        <properties>
            <!-- Строка подключения к БД -->
            <property name="jakarta.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/students?useSSL=false&amp;serverTimezone=UTC"/>
            <!-- Имя пользователя БД -->
            <property name="jakarta.persistence.jdbc.user" value="root"/>
            <!-- Пароль -->
            <property name="jakarta.persistence.jdbc.password" value="pass"/>
            <!-- Драйвер JDBC (в данном случае: MySQL) -->
            <property name="jakarta.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver"/>

            <!-- Поведение схемы -->
            <!-- В данном случае фреймворк проверяет соответствие
            существующей схемы БД с метаданными объектов, но
            не вносит никаких изменений, если схемы разные.
            Если схемы разные, то соответствующие сообщения будут 
            выведены в лог -->
            <property name="hibernate.hbm2ddl.auto" value="validate"/>
        </properties>
    </persistence-unit>
</persistence>
```

Для работы с БД необходимо использовать интерфейс `EntityManager` - это основной интерфейс для взаимодействия с базой данных в контексте объектно-реляционного отображения. Он управляет жизненным циклом сущностей, обеспечивает их сохранение, загрузку, обновление и удаление из базы данных.

Создадим класс `DbContext`, который отвечает за инициализацию объекта, реализующего интерфейс `EntityManager`:

```Java
import jakarta.persistence.EntityManager;  
import jakarta.persistence.EntityManagerFactory;  
import jakarta.persistence.Persistence;  
  
public class DbContext {  
    private static DbContext instance;  
  
    private final EntityManagerFactory emf;  
    private final EntityManager em;  
  
    private DbContext() {
	    // указываем название конфигурации JPA  
        emf = Persistence.createEntityManagerFactory("mysql-persistence-unit");  
        em = emf.createEntityManager();  
    }  
  
    public static DbContext getInstance() {  
        if (instance == null) {  
            instance = new DbContext();  
        }  
  
        return instance;  
    }  
  
    public EntityManager getEntityManager() {  
        return em;  
    }  
  
    public void close() {  
        if (em != null && em.isOpen()) {  
            em.close();  
        }  
  
        if (emf != null && emf.isOpen()) {  
            emf.close();  
        }  
    }  
}
```

`EntityManager` должен быть создан в единственном экземпляре для всего приложения и также он должен быть закрыт при завершении приложения. Для этого необходимо создать класс-слушатель и определить в нем методы, которые будут вызываться при старте и завершении приложения:

```Java  
import gstu.thehwk.db.DbContext;  
import jakarta.servlet.ServletContextEvent;  
import jakarta.servlet.ServletContextListener;  
  
public class WebContextListener implements ServletContextListener {  
    @Override  
    public void contextInitialized(ServletContextEvent sce) {  
        // Пробуем инициализировать EntityManager  
        DbContext.getInstance();  
    }  
  
    @Override  
    public void contextDestroyed(ServletContextEvent sce) {
        // Закрываем EntityManager 
        DbContext.getInstance().close();  
    }  
}
```

Теперь, используя `EntityManager`, можно работать с БД. При этом, он автоматически будет использовать реализацию *Hibernate* (т.е. мы абстрагируемся от реализации и используем только интерфейсы).

#### Пример реализации классов-сущностей

Классы-сущности представляют описание таблиц БД. Каждый такой класс помечается аннотацией `Entity`, чтобы указать *JPA*, что это сущность. Также добавляется аннотация `Table`, если нужно явно указать имя таблицы. 

Каждый столбец таблицы описывается полем класса с аннотацией `Column`, а первичный ключ - с помощью `Id` (часто вместе с `GeneratedValue` для автоинкремента). 

Важно, чтобы у класса был публичный конструктор без аргументов, а сами поля либо были открыты, либо имели геттеры и сеттеры. *JPA* будет автоматически связывать объекты с записями в базе данных.

Например, пусть есть две сущности: студенты и группы. Они имеют связь один-ко-многим (в одной группе может быть много студентов). Класс, описывающий сущность "Группа":

```Java
import jakarta.persistence.*;  
  
import java.util.ArrayList;  
import java.util.List;  
  
@Entity  
@Table(name = "`groups`")  
public class Group {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private long id;  
    @Column(name = "`name`")  
    private String name;  
  
    @OneToMany(mappedBy = "group", cascade = CascadeType.ALL)  
    private List<Student> students = new ArrayList<>();  
  
    public Group() { }  
  
    // ... get/set
}
```

А это класс, описывающий сущность "Студент":

```Java
import jakarta.persistence.*;  
  
@Entity  
@Table(name = "students")  
public class Student {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private long id;  
    @Column(name = "first_name")  
    private String firstName;  
    @Column(name = "last_name")  
    private String lastName;  
  
    @ManyToOne(targetEntity = Group.class, cascade = CascadeType.ALL)  
    @JoinColumn(name = "group_id")  
    private Group group;  
  
    public Student() {}  
  
    // ... get/set
}
```

Обратите внимание как описывается связь один-ко-многим.

*Примечание: в аннотации `OneToMany` в параметре mappedBy необходимо указать название поля класса, а не поля таблицы.*

При необходимости можно с помощью аннотаций указать ограничения к полям.

#### Пример реализации слоя доступа к данным
Все операции с БД необходимо выполнять с использованием транзакций.

```Java
import jakarta.persistence.*;

import java.util.ArrayList;
import java.util.List;

public class GroupDao {
    private final EntityManager em;

    GroupDao(EntityManager em) {
        this.em = em;
    }

    /**
     * Выборка группы по id
     */
    public Group getById(int id) {
        Group obj;

        em.getTransaction().begin();
        obj = em.find(Group.class, id);
        em.getTransaction().commit();

        return obj;
    }

    /**
     * Выборка множества групп с пагинацией
     * @param pageSize количество записей на одной "странице"
     * @param pageNumb номер "страницы"
     */
    public List<Group> getWithPagination(int pageSize, int pageNumb) {
        // Описываем процедуру, которую хотим вызвать
        // и указываем необходимые параметры
        StoredProcedureQuery query = em
                .createStoredProcedureQuery("paginated_groups_select", Group.class)
                .registerStoredProcedureParameter("page_size", Integer.class, ParameterMode.IN)
                .registerStoredProcedureParameter("page_number", Integer.class, ParameterMode.IN)
                .setParameter("page_size", pageSize)
                .setParameter("page_number", pageNumb);

        List<Group> groups = new ArrayList<>();

        // Вызываем процедуру в рамках транзакции
        em.getTransaction().begin();
        groups = query.getResultList();
        em.getTransaction().commit();

        return groups;
    }

    /**
     * Сохранение или обновление группы
     * (если id ненулевой, то запись обновляется,
     * иначе добавляется)
     */
    public Group saveOrUpdate(Group group) {
        Group merged = null;

        try {
            em.getTransaction().begin();
            merged = em.merge(group);
            em.getTransaction().commit();
        } catch (PersistenceException e) {
            em.getTransaction().rollback();    // откатываем транзакцию
            // Здесь выбрасываем свое исключение с описанием проблемы
        }

        return merged;
    }

    /**
     * Удаление группы
     * (если группа с указанным id не существует,
     * то ошибки не будет. Рекомендуется выполнять
     * проверку на уровне выше (например, в сервисе))
     */
    public void remove(Group group) {
        em.getTransaction().begin();
        em.remove(group);
        em.getTransaction().commit();
    }
}
```

#### Особенности использования классов-сущностей в контроллерах

Классы-сущности содержат циклические зависимости (обратите внимание, что классы `Student` и `Group` имеют ссылки друг на друга), что автоматически создает трудности для сериализации объектов этих классов.

Для решения этой проблемы можно создать *DTO*-классы, в которых отсутствуют такие циклические зависимости и использовать их для сериализации и дальнейшей передачи по сети.

Пример *DTO*-класса для `Group`:

```Java
import java.util.ArrayList;  
import java.util.List;  
  
public class GroupDto {  
    private int id;  
    private String name;  
    private List<StudentDto> students = new ArrayList<>();  
  
    public static GroupDto toDto(Group group) {  
        GroupDto dto = new GroupDto();  
        dto.id = group.getId();  
        dto.name = group.getName();  
  
        for (Student s : group.getStudents()) {  
            dto.students.add(StudentDto.toDto(s));  
        }  
  
        return dto;  
    }  
  
    public static List<GroupDto> toDto(List<Group> groups) {  
        List<GroupDto> dtos = new ArrayList<>();  
  
        for (Group g : groups) {  
            dtos.add(GroupDto.toDto(g));  
        }  
  
        return dtos;  
    }
}
```

Т.к. слой доступа к данным возвращает объекты классов-сущностей, необходима прослойка между контроллерами и слоем доступа к данным, в которой будет происходить преобразование этих объектов в *DTO*. Роль такой прослойки могут выполнять сервисы (*Services*).

Задачей сервисов является инкапсуляция бизнес-логики приложения. Помимо передачи данных между контроллерами и слоем доступа к данным, в них может быть реализована валидация, подготовка данных и вызов внешних *API*.

Пример метода дя удаления группы по `id` в классе-сервисе: 

```Java
public class GroupService {  
    // ...
    
    public boolean remove(int id) {
	    // Перед удалением проверяем есть ли
	    // вообще запись с указанным id
        Group group = dao.getById(id);
  
        if (group != null) {
	        // если она есть, то удаляем ее
            dao.remove(group);  
            return true;  
        }
  
        return false;  
    }
    
	// ...
}
```

#### Реализация классов-контроллеров

После реализации сервисов можно приступить к реализации контроллеров. Например:

```Java
@Path("/groups")  
@Produces(MediaType.APPLICATION_JSON)  
@Consumes(MediaType.APPLICATION_JSON)  
public class GroupController {  
    private final GroupService service = GroupService.getInstance();  
  
    @GET  
    public List<GroupDto> getAll(@QueryParam("page_size") int pageSize, @QueryParam("page_numb") int pageNumb) {  
        return service.getAll(pageSize, pageNumb);  
    }  
  
    @GET  
    @Path("/{id}")  
    public Response getById(@PathParam("id") int id) {  
        if (id < 0) {  
            return Response.status(400).build();  
        } else {  
            return Response.ok(service.getById(id)).build();  
        }  
    }  
  
    @POST  
    public Response saveOrUpdate(Group group) {  
        return Response.ok(service.saveOrUpdate(group)).build();  
    }
}
```

Используя аннотации `QueryParam` и `PathParam` можно получить параметры *GET*-запроса.

В качестве возвращаемого типа метода контроллера может быть как и объект класса `Response`, так и любой другой объект. *Jersey* автоматически сериализует их в формат *JSON*.

Класс `Response` используется для создания и настройки *HTTP*-ответов. Он позволяет явно задавать статус ответа (например, `200 OK`, `404 Not Found`), заголовки, тело ответа, а также управлять другими параметрами.

### Задание

Разработать *RESTful* сервис для работы с БД, предоставляющий открытый *API*, с использованием спецификации *JAX-RS*. Разработать документацию к *API* в [*Postman*](https://www.postman.com/) ([ссылка на краткую теорию](./Postman%20theory.md)). Варианты БД:
- для групп ИТП, ИТИ - взять по курсовой по РПБДИС
- для группы ИТД - из курса "Базы данных NoSQL"

**Требования к БД:**
- реализовать минимум 3 связанные таблицы
- реализовать процедуры для выборки записей с использованием пагинации (частичная загрузка данных)
- реализовать процедуру для фильтрации данных
- реализовать процедуру для группировки данных

**Требования к серверу:**
- организовать приложение по следующей структуре:
	- классы-модели
	- слой доступа к данным: работает с БД
	- контроллеры: обрабатывают *HTTP*-запросы
		- отвечают только лишь за получение данных из объекта-запроса и формирование объекта-ответа. Всю остальную работу делегируют сервисам
	- сервисы: используют слой доступа к данным (или другие слои, при наличии)
- для работы с БД использовать фреймворк [*Hibernate*](https://hibernate.org/) (или классический *JDBC*)
- *API* должен предоставлять методы для выполнения *CRUD*-операций + методы для фильтрации и группировки данных. При этом для выборки множества записей использовать процедуры с пагинацией
- формат сообщений: *JSON*
- во всех *URL* рекомендуется добавлять префикс `/api/v1` (где `v1` - версия *API*)
- использовать следующее правило о наименовании *URL* (на примере данных о пользователях: `users`):
	- для выборки множества пользователей: `/users`
	- для выборки конкретного пользователя: `/users/1` (где `1` - *id* пользователя) или `/users?id=1`
- *HTTP*-методы:
	- `GET` - для выборки данных
	- `POST` - для добавления данных
	- `PUT` - для изменения данных
	- `DELETE` - для удаления данных
- для документирования и верификации *API* использовать *Postman*
	- в *Postman* создать запросы (*CRUD*, фильтрация, группировка) для двух таблиц с отношением "один-ко-многим", при этом запросы разбить на две коллекции (т.е. 2 таблицы - 2 коллекции). Коллекции поместить в один *API* и написать для него документацию с описанием *API*-методов и с примерами запросов
- реализовать обработку ошибок (формирование ответа с соответствующим *HTTP*-кодом и сообщением об ошибке)

**Отчёт по лабораторной работе должен содержать:**
1. Цель и задачи работы.
2. Схема БД и ее описание.
3. Описание API (список методов и их краткое описание).
4. Скриншот с результатом вызова методов API в Postman.
5. Скриншот с частью документации к API в Postman.
6. Вывод по выполненной работе.