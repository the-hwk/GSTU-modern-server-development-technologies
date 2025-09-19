**Современные технологии серверной разработки**

## Лабораторная работа №1 (6 ч)

**Тема:** Использование *JDBC* для работы с БД в *Java*.

**Цель работы:** научиться использовать *JDBC* для работы с БД в *Java*.

#### Теоретические сведения

***JDBC*** (*Java Database Connectivity*) - это стандартный *API Java* для взаимодействия с реляционными базами данных. Он обеспечивает единый способ отправки *SQL*-запросов и получения результатов, независимо от конкретной СУБД.

Основые классы *JDBC*:
- `DriverManager` - управляет драйверами и устанавливает соединение с БД
- `Connection` - объект соединения с базой данных
- `Statement` / `PreparedStatement` - объекты для отправки *SQL*-запросов
- `ResultSet` - результат выполнения запроса `SELECT`, позволяет построчно читать данные
- `SQLException` - класс для обработки ошибок работы с БД

Для использования *JDBC* необходимо:
1. Загрузить драйвер (обычно достаточно добавить библиотеку в проект, современные драйверы регистрируются автоматически).
2. Установить соединение:

```java
Connection conn = DriverManager.getConnection(url, user, password);
```
    
3. Создать запрос:

```java
Statement stmt = conn.createStatement();
```
    
или с параметрами (для защиты от *SQL*-инъекций):

```java
PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
ps.setInt(1, 5);
```
    
4. Выполнить запрос:
    - `executeQuery()` - для `SELECT`, возвращает объект класса `ResultSet`
    - `executeUpdate()` - для `INSERT`, `UPDATE`, `DELETE`, возвращает количество изменённых строк
5. Обработать результат:

```java
while (rs.next()) {
    String name = rs.getString("name");
}
```
    
6. Закрыть ресурсы (в обратном порядке):

```java
rs.close();    // ResultSet
stmt.close();  // Statement
conn.close();  // Connection
```

*Примечание*: эти классы реализуют интерфейс `Autocloseable`, поэтому их можно использовать в конструкции *try-with-resources*.

#### Рекомендации по разработке простого Java-приложения для работы с реляционной БД

Приложение, реализующее работу с БД, можно разбить на несколько компонентов:
- классы, отвечающие за загрузку и работу с конфигурацией приложения
- классы, отвечающие за соединение с БД
- модели данных
- слой доступа к данным
- UI

Т.к. приложение должно использовать БД, необходимо создать файл конфигурации для возможности изменения строки подключения к БД без пересборки проекта. В *Java* файлы конфигурации имеют расширение `.properties` и хранятся в папке *resources*, которая находится в папке `src/main`. Пример файла конфигурации:

```properties
db_url="jdbc:mysql://localhost:3306/test"
db_user="root"
db_pass="some_password"
```

Для чтения файла конфигурации можно создать класс с использованием паттерна "*Singleton*". Пример реализации:

```Java
package gstu.thehwk.util;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

import gstu.thehwk.exception.PropertyFileNotFoundException;

public class AppProperties {
    private static AppProperties instance;

    private static final String FILE_NAME = "app.properties"; 

    private final Properties properties;

    private AppProperties() {
        this.properties = new Properties();
    }

    /**
     * Загружает файл конфигурации приложения
     * @throws IOException если файл конфигурации не удается прочитать
     * @throws PropertyFileNotFoundException если файл конфигурации отсутствует
     */
    public void load() throws IOException, PropertyFileNotFoundException {
        try (InputStream in = AppProperties.class.getClassLoader().
                getResourceAsStream(FILE_NAME)) {
            if (in == null) {
                throw new PropertyFileNotFoundException(FILE_NAME);
            }
            
            this.properties.load(in);
        }
    }

    /**
     * Возвращает объект
     * @return
     */
    public static AppProperties getInstance() {
        if (instance == null) {
            instance = new AppProperties();
        }
        return instance;
    }

    /**
     * Возвращает свойство по указанному ключу (key) или null, если запрашиваемого свойства нет
     * @param key
     * @return Строка свойства или null, если запрашиваемого свойства нет
     */
    public String getProperty(String key) {
        return this.properties.getProperty(key);
    }
}
```

Для работы с сетевым соединением с БД необходимо реализовать класс с использованием паттерна "*Singleton*". Пример реализации:

```Java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

import gstu.thehwk.util.AppProperties;

public class ConnectionManager {
    private static ConnectionManager instance;

    // все строки-константы храним в переменных
    private static final String DB_URL_KEY  = "db_url";
    private static final String DB_USER_KEY = "db_user";
    private static final String DB_PASS_KEY = "db_pass";

    private Connection connection;

    private ConnectionManager() {
        
    }

    public static ConnectionManager getInstance() {
        if (instance == null) {
            instance = new ConnectionManager();
        }
        return instance;
    }

    /**
     * Подключается к БД и возвращает объект соединения с БД
     * @return
     * @throws SQLException если установить подключение не удалось
     */
    public Connection getConnection() throws SQLException {
        // проверяем создан ли объект соединения с БД, а если создан, то закрыто ли соединение
        if (connection == null || connection.isClosed()) {
            AppProperties properties = AppProperties.getInstance();
            String url = properties.getProperty(DB_URL_KEY);
            String user = properties.getProperty(DB_USER_KEY);
            String pass = properties.getProperty(DB_PASS_KEY);

            connection = DriverManager.getConnection(url, user, pass);
        }

        return connection;
    }

    public void closeConnection() throws SQLException {
        // проверяем создан ли объект соединения с БД, а если создан, то открыто ли соединение
        if (connection != null && !connection.isClosed()) {
            connection.close();
        }
    }
}
```

Далее необходимо реализовать классы сущностей (модели). Класс модели должен содержать только поля и методы для их получения/изменения. Также рекомендуется переопределять методы `hashCode` и `equals`, если модель будет использована в операциях сравнения. Вариаций конструкторов должно быть несколько. Пример:

```Java
package gstu.thehwk.model;

public class Group {
    // id должен быть неизменяемым
    private final long id;
    private String name;

    /**
     * Базовый конструктор.
     * Используется в операциях извлечения записей из БД.
     * @param id
     * @param name
     */
    public Group(long id, String name) {
        this.id = id;
        this.name = name;
    }

    /**
     * Конструктор без id.
     * Необходим в операциях создания новой записи в БД.
     * @param name
     */
    public Group(String name) {
        this(-1, name);
    }

    /**
     * Копирующий конструктор с id.
     * Используется после выполнения операции создания 
     * новой записи в БД (БД, как правило, после создания
     * записи возвращает сгенерированный id этой записи)
     * @param id
     * @param group
     */
    public Group(long id, Group group) {
        this(id, group.name);
    }

    public long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result;
        result = prime * result + ((name == null) ? 0 : name.hashCode());
        return result;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() != obj.getClass())
            return false;

        Group other = (Group) obj;
        if (id != other.id)
            return false;
        if (name == null) {
            if (other.name != null)
                return false;
        } else if (!name.equals(other.name))
            return false;
        return true;
    }

    @Override
    public String toString() {
        return "Group [id=" + id + ", name=" + name + "]";
    }
}
```

Далее необходимо создать слой для работы с данными. Для этого можно использовать паттерн "*DAO*". Класс, реализованный с помощью этого паттера, содержит методы для выполнения *CRUD* операций с БД и другие методы для выполнения более сложных *SQL*-запросов. Пример реализации:

```Java
package gstu.thehwk.dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

import gstu.thehwk.db.ConnectionManager;
import gstu.thehwk.model.Group;

public class GroupDAO {
    private static final String F_ID   = "Id";
    private static final String F_NAME = "GName";

    private static final String Q_FIND_BY_ID = "SELECT * FROM StudentsGroups WHERE id = ?";
    private static final String Q_INSERT     = "INSERT INTO StudentsGroups (GName) VALUES (?)";

    /**
     * Конструктор package-private (доступен только классам
     * из того же пакета, что и этот класс)
     */
    GroupDAO() {

    }

    public Group findById(int id) throws SQLException {
        Connection connection = ConnectionManager.getInstance().getConnection();

        try (PreparedStatement statement = connection.prepareStatement(Q_FIND_BY_ID)) {
            statement.setInt(1, id);
            try (ResultSet resultSet = statement.executeQuery()) {
                return this.handleResultSet(resultSet);
            }
        }
    }

    public Group insert(Group group) throws SQLException {
        Connection connection = ConnectionManager.getInstance().getConnection();

        try (PreparedStatement statement = connection.prepareStatement(Q_INSERT)) {
            statement.setString(1, group.getName());
            statement.executeUpdate();

            try (ResultSet generatedKeys = statement.getGeneratedKeys()) {
                if (generatedKeys.next()) {
                    int id = generatedKeys.getInt(1);
                    return new Group(id, group);
                }
            }
        }

        return group;
    }

    public boolean update(Group group) {
        // code
    }

    public boolean delete(Group group) {
        // code
    }

    private Group handleResultSet(ResultSet resultSet) throws SQLException {
        if (resultSet.next()) {
            int id = resultSet.getInt(F_ID);
            String name = resultSet.getString(F_NAME);
            return new Group(id, name);
        }
    
        return null;
    }
}
```

Для получения объектов *DAO*-классов можно использовать паттерн "Фабрика" (в данном случае он немного изменен). Пример реализации:

```Java
public class DAOFactory {
    private final static GroupDAO groupDAO = new GroupDAO();
    private final static FacultyDAO facultyDAO = new FacultyDAO();

    public static GroupDAO getGroupDAO() {
        return groupDAO;
    }

    public static FacultyDAO getFacultyDAO() {
        return facultyDAO;
    }
}
```

В таком случае *DAO*-классы будут существовать в одном экземпляре.

Осталось только реализовать консольный *UI*, который использует разработанные классы моделей и *DAO*-классы.

Метод *main* должен содержать только вызовы основных слоев приложения. Пример:

```Java
import gstu.thehwk.db.ConnectionManager;
import gstu.thehwk.ui.ConsoleUI;
import gstu.thehwk.ui.UIRunner;
import gstu.thehwk.util.AppProperties;

public class Main 
{
    public static void main( String[] args )
    {
        try {
            // Загружаем конфигурацию приложения
            AppProperties.getInstance().load();
            // Подключаемся к БД (тестируем первоначальное соединение)
            ConnectionManager.getInstance().getConnection();

            // Запускаем консольный UI
            UIRunner ui = new ConsoleUI();
            ui.run();
            
            // Закрываем соединение с БД
            ConnectionManager.getInstance().closeConnection();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

Структура проекта выглядит следующим образом:
- `src/main/`
	- `java/some_name/`
		- `dao` - содержит *DAO*-классы и класс-фабрику этих классов
		- `db` - содержит класс для соединения с БД
		- `exception` - содержит классы-исключения
		- `model` - содержит классы моделей
		- `ui` - содержит классы, отвечающие за *UI*
		- `util` - содержит класс для работы с конфигурацией приложения
		- `Main.java`
	- `resources`
		- `app.properties`

*Примечание: в случае использования графического UI (GUI) загрузку конфигурации, подключение к БД и закрытие соединения можно вынести в класс, отвечающий за работу с GUI, чтобы, например, если отсутствует файл конфигурации, можно было вывести предупреждающее окно или строку в GUI*.