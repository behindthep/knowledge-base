- [1JDBC](#1-JDBC)
  - [](#)
- [2JPA](#2-JPA)
- [3JVM](#3-JVM)
- [4ЖЦ объекта](#4-жизненный-цикл-java-объекта)
---

## 1 JDBC

Одна из основных целей языка программирования - хранение и обработка информации. Чтобы лучше понять работу хранения данных стоит немного времени выделить на теорию и архитектуру приложений. Например, можно ознакомиться с литературой, а именно с книгой "[Software Architect's Handbook: Become a successful software architect by implementing effective arch...](https://goo.gl/aB3WLm)" авторства Joseph Ingeno. Как сказано, есть некий Data Tier или "Слой данных". Он включает в себя место хранения данных (например, SQL базу данных) и средства для работы с хранилищем данных (например, JDBC, о котором и пойдёт речь). Так же на сайте Microsoft есть статья: "[Проектирование уровня сохраняемости инфраструктуры](https://docs.microsoft.com/ru-ru/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)" в которой описывается архитектурное решение выделения из Data Tier дополнительного слоя — Persistence Layer. В таком случае Data Tier — это уровень хранения самих данных, в то время как Persistence Layer - это некоторый уровень абстракции для работы с данными из хранилища с уровня Data Tier. К уровню Persistence Layer можно отнести шаблон "DAO" или различные ORM. Но ORM — это тема отдельного разговора. Как Вы могли уже понять, вначале появился Data Tier. Ещё с времён JDK 1.1 в Java мире появился JDBC (Java DataBase Connectivity — соединение с базами данных на Java). Это стандарт взаимодействия Java-приложений с различными СУБД, реализованный в виде пакетов java.sql и javax.sql, входящих в состав Java SE:

Данный стандарт описан специфкицией "[JSR 221 JDBC 4.1 API](http://download.oracle.com/otn-pub/jcp/jdbc-4_1-mrel-spec/jdbc4.1-fr-spec.pdf)". Данная спецификация рассказывает нам о том, что JDBC API предоставляет программный доступ к реляционным базам данных из программ, написанных на Java. Так же рассказывает о том, что JDBC API является частью платформы Java и входит поэтому в Java SE и Java EE. JDBC API представлен двумя пакетами: java.sql and javax.sql. Давайте тогда с ними и познакомимся.


Чтобы понять что такое вообще JDBC API нам понадобится Java приложение. Удобнее всего воспользоваться одной из систем сборки проектов. Например, воспользуемся [Gradle](https://gradle.org/install/). Более подробно про Gradle можно прочитать в небольшом обзоре: "[Краткое знакомство с Gradle](https://javarush.com/groups/posts/2126-kratkoe-znakomstvo-s-gradle)". Для начала инициализируем новый Gradle проект. Так как функциональность Gradle реализуется через плагины, то для инициализации нам нужно воспользоваться "[Gradle Build Init Plugin](https://docs.gradle.org/current/userguide/build_init_plugin.html)":

```java
gradle init --type java-application
```

Откроем после этого билд скрипт — файл **build.gradle**, где описывается наш проект и то, как с ним нужно работать. Нас интересует блок "**dependencies**", где описываются зависимости — то есть те библиотеки/фрэймворки/api, без которых мы не можем работать и от которых мы зависим. По умолчанию мы увидим что-то вроде:

```java
dependencies {
    // This dependency is found on compile classpath of this component and consumers.
    implementation 'com.google.guava:guava:26.0-jre'
    // Use JUnit test framework
    testImplementation 'junit:junit:4.12'
}
```

Почему мы тут это видим? Это зависимости нашего проекта, которые нам сгенерировал автоматически Gradle при создании проекта. А так же потому что guava - это отдельная библиотека, не входящая в комплект с Java SE. JUnit так же не входит в комплект с Java SE. Но JDBC у нас есть "из коробки", то есть входит в состав Java SE. Получается JDBC у нас есть. Отлично. Что же нам ещё надо? Есть такая замечательная схема:

Как мы видим, и это логично, база данных является внешним компонентом, которого нет изначально в Java SE. Это объясняется просто - существует огромное количество баз данных и работать вы можете захотеть с любой. Например, есть PostgreSQL, Oracle, MySQL, H2. Каждая из этих баз данных поставляется отдельной компанией, которые называются поставщиками баз данных или database vendors. Каждая база данных написана на каком-то своём языке программирования (не обязательно Java). Чтобы с базой данных можно было работать из Java приложения поставщик базы данных пишет особый драйвер, который является своего образа адаптером. Такие JDBC совместимые (то есть у которых есть JDBC драйвер) ещё называют "JDBC-Compliant Database". Тут можно провести аналогию с компьютерными устройствами. Например, в блокноте есть кнопка "Печать". Каждый раз когда вы её нажимаете программа сообщает операционной системе, что приложение блокнот хочет напечатать. И у Вас есть принтер. Чтобы научить разговаривать единообразно вашу операционную систему с принтером Canon или HP понадобятся разные драйверы. Но для Вас, как пользователя, ничего не изменится. Вы по прежнему будете нажимать одну и ту же кнопку. Так и с JDBC. Вы выполняете один и тот же код, просто "под капотом" могут работать разные базы данных. Думаю, тут очень понятный подход. Каждый такой JDBC драйвер — это некоторый артефакт, библиотека, jar файл. Он то и является зависимостью для нашего проекта. Например, мы можем выбрать базу данных "[H2 Database](http://www.h2database.com/html/cheatSheet.html)" и тогда нам надо добавить зависимость следующим образом:

```java
dependencies {
    implementation 'com.h2database:h2:1.4.197'
```

То, как найти зависимость и как её описать указано на официальных сайтах поставщика БД или на "[Maven Central](https://mvnrepository.com/repos/central)". JDBC драйвер не является базой данных, как Вы поняли. А лишь является проводником к ней. Но есть такое понятие, как "[In memory databases](http://www.h2database.com/html/features.html#in_memory_databases)". Это такие базы данных, которые существуют в памяти на время жизни вашего приложения. Обычно, это часто используют для тестирования или для учебных целей. Это позволяет не ставить отдельный сервер баз данных на машине. Что нам очень даже подойдёт для знакомств с JDBC. Вот и готова наша песочница и мы приступаем.

## Connection

у нас есть JDBC драйвер, есть JDBC API. Как мы помним, JDBC расшифровывается как Java DataBase Connectivity. Поэтому, всё начинается с Connectivity - возможности устанавливать подключение. А подключение — это Connection. Обратимся снова к тексту [спецификации JDBC](http://download.oracle.com/otn-pub/jcp/jdbc-4_1-mrel-spec/jdbc4.1-fr-spec.pdf) и посмотрим на оглавление. В главе "**CHAPTER 4 Overview**" (overview - обзор) обратимся к разделу "**4.1 Establishing a Connection**" (установление соединения) сказано, что существует два способа подключения к БД:

- Через DriverManager
- Через DataSource

Разберёмся с DriverManager'ом. Как сказано, DriverManager позволяет подключиться к базе данных по указанному URL, а так же загружает JDBC Driver'ы, который он нашёл в CLASSPATH (а раньше, до JDBC 4.0 загружать класс драйвера надо было самостоятельно). Про соединение с БД есть отдельная глава "CHAPTER 9 Connections". Нас интересует, как получить соединение через DriverManager, поэтому нам интересен раздел "9.3 The DriverManager Class". В нём указано, как мы можем получить доступ к БД:

```java
Connection con = DriverManager.getConnection(url, user, passwd);
```

Параметры можно взять с сайта выбранной нами базы данных. В нашем случае это H2 — "[H2 Cheat Sheet](http://www.h2database.com/html/cheatSheet.html)". Перейдём в подготовленный Gradle'ом класс AppTest. Он содержит JUnit тесты. JUnit тест — это метод, который помечен аннотацией `@Test`. Юнит тесты не являются темой данного обзора, поэтому просто ограничимся пониманием того, что это описанные определённым образом методы, цель которых что-то протестировать. Согласно специфкиации JDBC и сайту H2 проверим, что мы получили подключение к БД. Напишем метод получения подключения:

```java
private Connection getNewConnection() throws SQLException {
	String url = "jdbc:h2:mem:test";
	String user = "sa";
	String passwd = "sa";
	return DriverManager.getConnection(url, user, passwd);
}
```

Теперь напишем тест для этого метода, который проверит, что подключение действительно устанавливается:

```java
@Test
public void shouldGetJdbcConnection() throws SQLException {
	try(Connection connection = getNewConnection()) {
		assertTrue(connection.isValid(1));
		assertFalse(connection.isClosed());
	}
}
```

Данный тест при выполнении проверит, что полученное подключение валидное (корректно созданное) и что оно не закрыто. Благодаря использованию конструкции [try-with-resources](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html) мы освободим ресурсы после того, как они нам больше не нужны. Это убережёт нас от "провисших" соединений и утечек памяти. Так как любые действия с БД требуют подключения, то давайте для остальных тестовых методов, помеченных @Test, обеспечим в начале теста Connection, который мы освободим после теста. Для этого нам понадобится две аннотации: @Before и @After Добавим в класс AppTest новое поле, которое будет хранить JDBC подключение для тестов:

```java
private static Connection connection;
```

И добавим новые методы:

```java
@Before
public void init() throws SQLException {
	connection = getNewConnection();
}
@After
public void close() throws SQLException {
	connection.close();
}
```

Теперь, любому тестовому методу гарантируется наличие JDBC connection и он не должен каждый раз сам его создавать.

## Statements

Далее нас интересует Statements или выражения. Они описаны в документации в главе "**CHAPTER 13 Statements**". Во-первых, там сказано, что существует несколько типов или видов statement'ов:

- Statement: SQL выражение, которое не содержит параметров
- PreparedStatement : Подготовленное SQL выражение, содержащее входные параметры
- CallableStatement : SQL выражение с возможностью получить возвращаемое значение из хранимых процедур (SQL Stored Procedures).

Итак, имея подключение, мы можем в рамках этого подключения выполнить какой-нибудь запрос. Поэтому, логично, что экземпляр SQL выражения изначально мы получаем из Connection. Начать нужно с создания таблицы. Опишем запрос создания таблицы в виде переменной типа String. Как это сделать? Воспользуемся каким-нибудь обучающим руководством, вроде "[sqltutorial.org](http://www.sqltutorial.org/sql-create-table/)", "[sqlbolt.com](https://sqlbolt.com/lesson/creating_tables)", "[postgresqltutorial.com](http://www.postgresqltutorial.com/postgresql-create-table/)", "[codecademy.com](https://www.codecademy.com/learn/learn-sql)". Воспользуемся, например, примером из курса SQL на [khanacademy.org](https://www.khanacademy.org/computing/computer-programming/sql/sql-basics/pt/creating-a-table-and-inserting-data). Добавим метод выполнения выражения в БД:

```java
private int executeUpdate(String query) throws SQLException {
	Statement statement = connection.createStatement();
	// Для Insert, Update, Delete
	int result = statement.executeUpdate(query);
	return result;
}
```

Добавим метод создания тестовой таблицы с использованием прошлого метода:

```java
private void createCustomerTable() throws SQLException {
	String customerTableQuery = "CREATE TABLE customers " +
                "(id INTEGER PRIMARY KEY, name TEXT, age INTEGER)";
	String customerEntryQuery = "INSERT INTO customers " +
                "VALUES (73, 'Brian', 33)";
	executeUpdate(customerTableQuery);
	executeUpdate(customerEntryQuery);
}
```

Теперь протестируем это:

```java
@Test
public void shouldCreateCustomerTable() throws SQLException {
	createCustomerTable();
	connection.createStatement().execute("SELECT * FROM customers");
}
```

Теперь давайте выполним запрос, да ещё и с параметром:

```java
@Test
public void shouldSelectData() throws SQLException {
 	createCustomerTable();
 	String query = "SELECT * FROM customers WHERE name = ?";
	PreparedStatement statement = connection.prepareStatement(query);
	statement.setString(1, "Brian");
	boolean hasResult = statement.execute();
	assertTrue(hasResult);
}
```

JDBC не поддерживает именованные параметры для PreparedStatement, поэтому сами параметры указываются вопросами, а указывая значение мы указываем индекс вопроса (начиная с 1, а не с нуля). В последнем тесте мы получили true как признак того, есть ли результат. Но как представлен результат запроса в JDBC API? А представлен он как ResultSet.

## ResultSet

Понятие ResultSet описано в спецификации JDBC API в главе "CHAPTER 15 Result Sets". Прежде всего, там сказано, что ResultSet предоставляет методы для получения и манипуляции результатами выполненных запросов. То есть если метод execute вернул нам true, значит мы можем получить и ResultSet. Давайте вынесем вызов метода createCustomerTable() в метод init, который отмечен как @Before. Теперь доработаем наш тест shouldSelectData:

```java
@Test
public void shouldSelectData() throws SQLException {
	String query = "SELECT * FROM customers WHERE name = ?";
	PreparedStatement statement = connection.prepareStatement(query);
	statement.setString(1, "Brian");
	boolean hasResult = statement.execute();
	assertTrue(hasResult);
	// Обработаем результат
	ResultSet resultSet = statement.getResultSet();
	resultSet.next();
	int age = resultSet.getInt("age");
	assertEquals(33, age);
}
```

Тут стоит отметить, что next — это метод, который двигает так называемый "курсор". Курсор в ResultSet указывает на некоторую строку. Таким образом, чтобы считать строку, на неё нужно этот самый курсор установить. Когда курсор перемещается, то метод перемещения курсора возвращает true, если курсор валидный (правильный, корректный), то есть указывает на данные. Если возвращает false, значит данных нет, то есть курсор не указывает на данные. Если попытаться получить данные с невалидным курсором, то мы получим ошибку: No data is available Ещё интересно, что через ResultSet можно обновлять или даже вставлять строки:

```java
@Test
public void shouldInsertInResultSet() throws SQLException {
	Statement statement = connection.createStatement(ResultSet.TYPE_SCROLL_SENSITIVE, ResultSet.CONCUR_UPDATABLE);
	ResultSet resultSet = statement.executeQuery("SELECT * FROM customers");
	resultSet.moveToInsertRow();
	resultSet.updateLong("id", 3L);
	resultSet.updateString("name", "John");
	resultSet.updateInt("age", 18);
	resultSet.insertRow();
	resultSet.moveToCurrentRow();
}
```

## RowSet

JDBC помимо ResultSet вводит такое понятие, как RowSet. Подробнее можно прочитать здесь: "[JDBC Basics: Using RowSet Objects](https://docs.oracle.com/javase/tutorial/jdbc/basics/rowset.html)". Существуют различные вариации использования. Например, самый простой случай может выглядеть так:

```java
@Test
public void shouldUseRowSet() throws SQLException {
 	JdbcRowSet jdbcRs = new JdbcRowSetImpl(connection);
 	jdbcRs.setCommand("SELECT * FROM customers");
	jdbcRs.execute();
	jdbcRs.next();
	String name = jdbcRs.getString("name");
	assertEquals("Brian", name);
}
```

Как видно, RowSet похож на симбиоз statement (мы указали через него command) и выполнили command. Через него же мы управляем курсором (вызывая метод next) и из него же получаем данные. Интересен не только такой подход, но и возможные реализации. Например, CachedRowSet. Он является "отключённым" (то есть не использует постоянное подключение к БД) и требует явного выполнения синхронизации с БД:

```java
CachedRowSet jdbcRsCached = new CachedRowSetImpl();
jdbcRsCached.acceptChanges(connection);
```

Подробнее можно прочитать в tutorial на сайте Oracle: "[Using CachedRowSetObjects](https://docs.oracle.com/javase/tutorial/jdbc/basics/cachedrowset.html)".

## Metadata

Кроме запросов, подключение к БД (т.е. экземпляр класса Connection) предоставляет доступ к метаданным - данным о том, как настроена и как устроена наша база данных. Но для начала озвучим несколько ключевых моментов: URL подключения к нашей БД: "jdbc:h2:mem:test". test - это название нашей базы данных. Для JDBC API это каталог. И название будет в верхнем регистре, то есть TEST. Схема по умолчанию ([Default schema](http://www.h2database.com/html/grammar.html#set_schema)) для H2 - PUBLIC. Теперь, напишем тест, который показывает все пользовательские таблицы. Почему пользовательские? Потому что в базах данных есть не только пользовательские (те, которые мы сами создали при помощи create table выражений), но и системные таблицы. Они необходимы, чтобы хранить системную информацию о структуре БД. У каждой БД такие системные таблицы могут храниться по-разному. Например, в H2 они хранятся в схеме "[INFORMATION_SCHEMA](http://www.h2database.com/html/systemtables.html)". Интересно, что INFORMATION SCHEMA является общим подходом, но Oracle пошли иным путём. Подробнее можно прочитать здесь: "[INFORMATION_SCHEMA и Oracle](http://www.sql-tutorial.ru/ru/book_information_schema_and_oracle.html)". Напишем тест, получающий метаданные по пользовательским таблицам:

```java
@Test
public void shoudGetMetadata() throws SQLException {
	// У нас URL = "jdbc:h2:mem:test", где test - название БД
	// Название БД = catalog
	DatabaseMetaData metaData = connection.getMetaData();
	ResultSet result = metaData.getTables("TEST", "PUBLIC", "%", null);
	List<String> tables = new ArrayList<>();
	while(result.next()) {
		tables.add(result.getString(2) + "." + result.getString(3));
	}
	assertTrue(tables.contains("PUBLIC.CUSTOMERS"));
}
```

## Пул подключений

Пулу подключений в спецификации JDBC отведен раздел "Chapter 11 Connection Pooling". В нём же и даётся главное обоснование необходимости пула подключений. Каждый Coonection - это физическое подключение к БД. Его создание и закрытие - довольно "дорогая" работа. JDBC предоставляет лишь API для пула соединений. Поэтому, выбор реализации остаётся за нами. Например, к таким реализациям относится [HikariCP](https://github.com/brettwooldridge/HikariCP). Соответственно, нам понадобится добавить пул к нам в зависимости проекта:

```java
dependencies {
    implementation 'com.h2database:h2:1.4.197'
    implementation 'com.zaxxer:HikariCP:3.3.1'
    testImplementation 'junit:junit:4.12'
}
```

Теперь надо как-то пул этот задействовать. Для этого нужно выполнить инициализацию источника данных, он же Datasource:

```java
private DataSource getDatasource() {
	HikariConfig config = new HikariConfig();
	config.setUsername("sa");
	config.setPassword("sa");
	config.setJdbcUrl("jdbc:h2:mem:test");
	DataSource ds = new HikariDataSource(config);
	return ds;
}
```

И напишем тест на получение подключения из пула:

```java
@Test
public void shouldGetConnectionFromDataSource() throws SQLException {
	DataSource datasource = getDatasource();
	try (Connection con = datasource.getConnection()) {
		assertTrue(con.isValid(1));
	}
}
```

## Транзакции

Один из самых интересных моментов, связанных с JDBC - это транзакции. В спецификации JDBC им отведена глава "CHAPTER 10 Transactions". Прежде всего стоит понять, что же такое транзакция. Транзакция — это группа логически объединённых последовательных операций по работе с данными, обрабатываемая или отменяемая целиком. Когда начинается транзакция при использовании JDBC? Как гласит спецификация, это решает непосредственно JDBC Driver. Но обычно, новая транзакция начинается тогда, когда текущее SQL выражение (SQL statement) потребует транзакцию и транзакции ещё не создано. Когда заканчивается транзакция? Это регулируется атрибутом автокоммита (auto-commit). Если автокоммит включен, то транзакция будет завершена после того, как SQL выражение будет "выполнено". Что такое "выполнено" зависит от типа SQL выражения:

- Data Manipulation Language, он же DML (Insert, Update, Delete)  
    Транзакция завершается как только завершилось выполнение действия
Select Statements  
Транзакция завершается тогда, когда ResultSet будет закрыт ([ResultSet#close](https://docs.oracle.com/javase/8/docs/api/java/sql/ResultSet.html#close--))- CallableStatement и выражения, возвращающие несколько результатов  
    Когда все ассоциированные ResultSets будут закрыты и все выходные данные получены (включая кол-во апдейтов)

Так ведёт себя именно JDBC API. Как обычно, напишем на это тест:

```java
@Test
public void shouldCommitTransaction() throws SQLException {
	connection.setAutoCommit(false);
	String query = "INSERT INTO customers VALUES (1, 'Max', 20)";
	connection.createStatement().executeUpdate(query);
	connection.commit();
	Statement statement = connection.createStatement();
 	statement.execute("SELECT * FROM customers");
	ResultSet resultSet = statement.getResultSet();
	int count = 0;
	while(resultSet.next()) {
		count++;
	}
	assertEquals(2, count);
}
```

Всё просто. Но это так, пока у нас всего одна транзакция. А что делать, когда их несколько? Нужно их изолировать друг от друга. Поэтому, поговорим об уровнях изоляции транзакции и как с ними справляется JDBC.

## Уровни изоляции

Откроем подраздел "10.2 Transaction Isolation Levels" спецификации JDBC. Тут прежде чем дальше двигаться хочется всомнить про такую штуку, как ACID. ACID описывает требования к транзакционной системе.

- Atomicity(Атомарность):  
    Никакая транзакция не будет зафиксирована в системе частично. Будут либо выполнены все её подоперации, либо не выполнено ни одной.
- Consistency(Согласованность):  
    Каждая успешная транзакция по определению фиксирует только допустимые результаты.
- Isolation(Изолированность):  
    Во время выполнения транзакции параллельные транзакции не должны оказывать влияния на её результат.
- Durability(Долговечность):  
    Если транзакция успешно завершенеа, сделанные в ней изменения не будут отменены из-за какого-либо сбоя.

Говоря про уровни изоляции транзакции мы говорим как раз про требование "Isolation". Изолированность — требование "дорогое", поэтому в реальных БД существуют режимы, не полностью изолирующие транзакцию (уровни изолированности Repeatable Read и ниже). На википедии есть отличное объяснение того, какие проблемы могут возникать при работе с транзакциями. Подробнее стоит прочитать здесь: "[Проблемы параллельного доступа с использованием транзакций](https://ru.wikipedia.org/wiki/%D0%A3%D1%80%D0%BE%D0%B2%D0%B5%D0%BD%D1%8C_%D0%B8%D0%B7%D0%BE%D0%BB%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%BD%D0%BE%D1%81%D1%82%D0%B8_%D1%82%D1%80%D0%B0%D0%BD%D0%B7%D0%B0%D0%BA%D1%86%D0%B8%D0%B9#%D0%9F%D1%80%D0%BE%D0%B1%D0%BB%D0%B5%D0%BC%D1%8B_%D0%BF%D0%B0%D1%80%D0%B0%D0%BB%D0%BB%D0%B5%D0%BB%D1%8C%D0%BD%D0%BE%D0%B3%D0%BE_%D0%B4%D0%BE%D1%81%D1%82%D1%83%D0%BF%D0%B0_%D1%81_%D0%B8%D1%81%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5%D0%BC_%D1%82%D1%80%D0%B0%D0%BD%D0%B7%D0%B0%D0%BA%D1%86%D0%B8%D0%B9)". Прежде чем мы напишем наш тест, давайте чуть изменим наш Gradle Build Script: добавим блок с properties, то есть с настройками нашего проекта:

```java
ext {
    h2Version = '1.3.176' // 1.4.177
    hikariVersion = '3.3.1'
    junitVersion = '4.12'
}
```

Далее, используем это в версиях:

```java
dependencies {
    implementation "com.h2database:h2:${h2Version}"
    implementation "com.zaxxer:HikariCP:${hikariVersion}"
    testImplementation "junit:junit:${junitVersion}"
}
```

Вы могли заметить, что версия h2 стала ниже. Позже мы увидим, зачем. Итак, как же применять уровни изолированности? Давайте посмотрим сразу небольшой практический пример:

```java
@Test
public void shouldGetReadUncommited() throws SQLException {
	Connection first = getNewConnection();
	assertTrue(first.getMetaData().supportsTransactionIsolationLevel(Connection.TRANSACTION_READ_UNCOMMITTED));
	first.setTransactionIsolation(Connection.TRANSACTION_READ_UNCOMMITTED);
	first.setAutoCommit(false);
	// Транзакиця на подключение. Поэтому первая транзакция с ReadUncommited вносит изменения
	String insertQuery = "INSERT INTO customers VALUES (5, 'Max', 15)";
	first.createStatement().executeUpdate(insertQuery);
	// Вторая транзакция пытается их увидеть
	int rowCount = 0;
	JdbcRowSet jdbcRs = new JdbcRowSetImpl(getNewConnection());
	jdbcRs.setCommand("SELECT * FROM customers");
	jdbcRs.execute();
	while (jdbcRs.next()) {
		rowCount++;
	}
	assertEquals(2, rowCount);
}
```

Интересно, что данный тест может упасть на вендоре, который не поддерживает TRANSACTION_READ_UNCOMMITTED (например, sqlite или HSQL). А ещё уровень транзакции может просто не сработать. Помните мы указывали версию драйвера H2 Database? Если мы поднимем её до h2Version = '1.4.177' и выше, то READ UNCOMMITTED перестанет работать, хотя код мы не меняли. Это ещё раз доказывает, что выбор вендора и версии драйвера - это не просто буквы, от этого будет в реальности зависеть то, как будут выполняться ваши запросы. Про то, как исправить это поведение в версии 1.4.177 и как это не работает в версиях выше можно прочитать здесь: "[Support READ UNCOMMITTED isolation level in MVStore mode](https://github.com/h2database/h2database/issues/216)".



JDBC — это мощный инструмент в руках Java для работы с базами данных. Надеюсь, данный небольшой обзор поможет стать для Вас отправной точкой или поможет освежить в памяти что-нибудь. Ну и на закуску немного дополнительных материалов:

- Огненный доклад: "[Transactions: myths, surprises and opportunities](https://www.youtube.com/watch?v=5ZjhNTM8XU8)" от Martin Kleppmann
- Юрий Ткач: "[JPA. Транзакции](https://www.youtube.com/watch?v=4PKZRQAtf38&t=90s)"
- Юрик Ткач: "[JDBC - Java для тестировщиков](https://www.youtube.com/watch?v=aje9gtFEyC4)"
- Бесплатный курс на Udemy: "[JDBC and MySQL](https://www.udemy.com/how-to-connect-java-jdbc-to-mysql)"
- "[Обработка объектов CallableStatement](https://www.ibm.com/developerworks/ru/library/dm-0802tiwary/index.html)"
- IBM Developer: "[Java Database Connectivity](https://developer.ibm.com/articles/j-5things10/)"
- IBM Knowledge Center: "[Getting started with JDBC](https://www.ibm.com/support/knowledgecenter/en/ssw_ibm_i_71/rzaha/jdbcgets.htm)"


## 2 JPA

Но как только нужен одновременный доступ на чтение и редактирование, когда появляется нагрузка (т.е. одновременно поступает несколько обращений), хранение данных просто в файлах становится проблемой. Подробнее о том, какие проблемы решают БД и каким образом, советую прочитать в статье "[Как устроены базы данных](https://habr.com/ru/company/oleg-bunin/blog/358984/)". Значит, наши данные мы решаем хранить в базе данных. С давних пор Java умеет работать с базами данных при помощи JDBC API (The Java Database Connectivity). Подробнее про JDBC можно прочитать тут: "[JDBC или с чего всё начинается](https://javarush.com/groups/posts/2172-jdbc-ili-s-chego-vsje-nachinaetsja)". Но время шло и разработчики каждый раз сталкивались с необходимостью писать однотипный и ненужный "обслуживающий" код (так называемый Boilerplate code) для тривиальных операций по сохранению Java объектов в БД и наоборот, созданию Java объектов по данным из БД. И тогда для решения этих проблем на свет появилось такое понятие, как ORM. **ORM — Object-Relational Mapping или в переводе на русский объектно-реляционное отображение.** Это технология программирования, которая связывает базы данных с концепциями объектно-ориентированных языков программирования. Если упростить, то ORM это связь Java объектов и записей в БД: 

![JPA : Знакомство с технологией - 2](https://cdn.javarush.com/images/article/4b8f2fcd-2c48-497e-96a6-bf2100f93425/800.webp)

ORM — это по сути концепция о том, что Java объект можно представить как данные в БД (и наоборот). Она нашла воплощение в виде спецификации JPA — Java Persistence API. Спецификация — это уже описание Java API, которое выражает эту концепцию. Спецификация рассказывает, какими средствами мы должны быть обеспечены (т.е. через какие интерфейсы мы сможем работать), чтобы работать по концепции ORM. И как использовать эти средства. Реализацию средств спецификация не описывает. Это даёт возможность использовать для одной спецификации разные реализации. Можно упростить и сказать, что спецификация — это описание API. Текст спецификации JPA можно найти на сайте Oracle: "[JSR 338: JavaTM Persistence API](http://download.oracle.com/otn-pub/jcp/persistence-2_2-mrel-spec/JavaPersistence.pdf)". Следовательно, чтобы использоать JPA нам требуется некоторая реализацию, при помощи которой мы будем пользоваться технологией. Реализации JPA ещё называют JPA Provider. Одной из самых заметных реализаций JPA является [Hibernate](http://hibernate.org/). Поэтому, предлагаю её и рассмотреть.


JPA — это про Java, то нам понадобится Java проект. Мы могли бы сами вручную создать структуру каталогов, сами добавить нужные библиотеки. Но куда удобнее и правильнее использовать системы автоматизации сборки проектов (т.е. по сути это просто программа, которая за нас будет управлять сборкой проектов. Создавать каталоги, подкладывать в classpath нужные библиотеки и т.д.). Одной из такой систем является Gradle. Подробнее про Gradle можно прочитать здесь: "[Краткое знакомство с Gradle](https://javarush.com/groups/posts/2126-kratkoe-znakomstvo-s-gradle)". Как мы знаем, функциональность Gradle (т.е. действия, которые он может сделать) реализованы при помощи различных Gradle Plugins. Воспользуемся Gradle и плагином "[Gradle Build Init Plugin'ом](https://docs.gradle.org/current/userguide/build_init_plugin.html)". Выполним комманду:

```none
gradle init --type java-application
```

Gradle за нас сделает нужную структуру каталогов, создаст базовое декларативное описание проекта в билд скрипте `build.gradle`. Итак, у нас появилось приложение. Нам надо подумать, что мы хотим описывать или моделировать нашим приложением. Давайте воспользуемся каким-нибудь средством моделирования, например: 

[app.quickdatabasediagrams.com](https://app.quickdatabasediagrams.com/#/) ![JPA : Знакомство с технологией - 4](https://cdn.javarush.com/images/article/695f9085-8069-4007-888d-c2a98b21f1f6/800.webp)

описали нашу "доменную модель". Домен — это некоторая "предметная область". Вообще, домен — это "владение" на латыни. В средние века так назывались области, которыми владели короли или феодалы. А во французском языке это стало словом "domaine", которое переводится просто как "область". Таким образом мы описали нашу "доменную модель" = "предметную модель". Каждый элемент этой модели — это некоторая "сущность", что-то из реальной жизни. В нашем случае это сущности: Категория (`Category`), Тема (`Topic`). Создадим для сущностей отдельный пакет, например с именем model. И добавим туда Java классы, описывающие сущности. В Java коде такие сущности представляют из себя обычный [POJO](https://www.geeksforgeeks.org/pojo-vs-java-beans/), который может выглядеть так:

```java
public class Category {
    private Long id;
    private String title;

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }
}
```

Скопируем содержимое класса и сделаем по аналогии класс `Topic`. Отличаться он будет лишь тем, что он знает про категорию, к которой относится. Поэтому, добавим в класс `Topic` поле категории и методы работы с ней:

```java
private Category category;

public Category getCategory() {
	return category;
}

public void setCategory(Category category) {
	this.category = category;
}
```

Теперь у нас есть Java приложение, имеющее свою доменную модель. Пора теперь приступать к подключению к проекту JPA.

## Добавление JPA

Итак, как мы помним, JPA — это про то, что мы будем сохранять что-то в БД. Следовательно, нам нужна база данных. Чтобы использовать подключение к БД в своём проекте нам нужно добавить в зависимости библиотеку, для подключения к БД. Как мы помним, мы использовали Gradle, который создал нам билд скрипт `build.gradle`. В нём то мы и опишем зависимости, которые нужны нашему проекту. Зависимости — это те библиотеки, без которых не может работать наш код. Начнём с описания зависимости от подключения к БД. Делаем это так же, как бы делали, работая просто с JDBC:

```none
dependencies {
	implementation 'com.h2database:h2:1.4.199'
```

Теперь у нас есть БД. Мы можем теперь добавить в наше приложение уровень или слой (layer), отвечающий за отображение наших Java объектов в понятия базы данных (с Java языка на язык SQL). Как мы помним, мы собираемся использовать для этого реализацию спецификации JPA под названием Hibernate:

```none
dependencies {
	implementation 'com.h2database:h2:1.4.199'
	implementation 'org.hibernate:hibernate-core:5.4.2.Final'
```

сконфигурировать JPA. Если мы прочитаем спецификацию и раздел "8.1 Persistence Unit", то мы узнаем, что Persistence Unit — это некоторая некоторое объединение конфигураций, метаданных и сущностей. И чтобы JPA заработал, нужно описать хотя бы один Persistence Unit в конфигурационном файле, который имеет название `persistence.xml`. Его расположение описано в главе специфкиации "8.2 Persistence Unit Packaging". Согласно этому разделу, если у нас Java SE окружение, то мы должны положить его в корень каталога META-INF.

![JPA : Знакомство с технологией - 6](https://cdn.javarush.com/images/article/3560e674-fabb-4b2c-8a25-e9346a3b1368/256.webp)

Содержание скопируем из примера, приведённого в спецификации JPA в главе "`8.2.1 persistence.xml file`":

```java
<persistence>
	<persistence-unit name="JavaRush">
        <description>Persistence Unit For test</description>
        <class>hibernate.model.Category</class>
        <class>hibernate.model.Topic</class>
    </persistence-unit>
</persistence>
```

кто наш JPA Provider, т.е. тот, кто реализует спецификацию JPA:

```java
<provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
```

добавим настройки (`properties`). Часть из них (начинаются на `javax.persistence`) являются стандартными JPA конфигурациями и описаны в спецификации JPA в разделе "8.2.1.9 properties". Часть конфигураций являются провайдер-специфичными (в нашем случае, влияют на Hibernate как на Jpa Provider'а. Наш блок настроек будет выглядеть так:

```java
<properties>
    <property name="javax.persistence.jdbc.driver" value="org.h2.Driver" />
    <property name="javax.persistence.jdbc.url" value="jdbc:h2:mem:db1;DB_CLOSE_DELAY=-1;MVCC=TRUE" />
    <property name="javax.persistence.jdbc.user" value="sa" />
    <property name="javax.persistence.jdbc.password" value="" />
    <property name="hibernate.show_sql" value="true" />
    <property name="hibernate.hbm2ddl.auto" value="create" />
</properties>
```

Теперь у нас есть JPA-совместимый конфиг `persistence.xml`, есть JPA провайдер Hibernate и есть база данных H2, а так же есть 2 класса, который являются нашей доменной моделью. Давайте заставим это всё наконец-то отработать. В каталоге `/test/java` наш Gradle любезно нам сгенерировал шаблон для Unit тестов и назвал его AppTest. Давайте используем его. Как гласит глава "7.1 Persistence Contexts" спецификации JPA, сущности в мире JPA живут в некотором пространстве, которое называется "Контекст персистенции" (или Контексте постоянства, Persistence Context). Но напрямую мы не работаем с Persistence Context. Для этого мы используем `Entity Manager` или "менеджер сущностей". Именно он знает про контекст и про то, какие там живут сущности. Мы же взаимодействуем с `Entity Manager`'ом. Тогда остаётся только понять, откуда нам достать этот `Entity Manager`? Согласно главе "7.2.2 Obtaining an Application-managed Entity Manager" спецификации JPA мы должны использовать `EntityManagerFactory`. Поэтому, вооружимся спецификацией JPA и возьмём пример из главы "7.3.2 Obtaining an Entity Manager Factory in a Java SE Environment" и оформим его в виде простейшего Unit теста:

```java
@Test
public void shouldStartHibernate() {
	EntityManagerFactory emf = Persistence.createEntityManagerFactory( "JavaRush" );
	EntityManager entityManager = emf.createEntityManager();
}
```

Уже этот тест покажет ошибку "Unrecognized JPA persistence.xml XSD version". Причина — в `persistence.xml` нужно правильно указать используемую схему, как это сказано в спецификации JPA в разделе "8.3 persistence.xml Schema":

```java
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence
             http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd"
             version="2.2">
```

важен порядок элементов. Поэтому, `provider` должен быть указан до перечисления классов. После этого тест выполнится успешно. Непосредственное подключение JPA мы выполнили. Прежде чем мы будем двигаться дальше, подумаем про остальные тесты. Каждый наш тест будет требовать `EntityManager`. чтобы у каждого теста был свой `EntityManager` на начало выполнения. чтобы БД каждый раз была новая. Благодаря тому, что мы используем `inmemory` вариант, достаточно закрывать `EntityManagerFactory`. Создание `Factory` — дорогая операция. Но для тестов — это оправдано. JUnit позволяет задать методы, которые будут выполняться перед (Before) и после (After) выполнением каждого теста:

```java
public class AppTest {
    private EntityManager em;

    @Before
    public void init() {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory( "JavaRush" );
        em = emf.createEntityManager();
    }

    @After
    public void close() {
        em.getEntityManagerFactory().close();
        em.close();
    }
```

перед выполнением любого теста будет создана новая `EntityManagerFactory`, что повлечёт за собой создание новой БД, т.к. `hibernate.hbm2ddl.auto` имеет значение `create`. А из новой фабрики получим новый `EntityManager`.

## Сущности (Entities)

создали ранее классы, описывающие нашу доменную модель. Мы уже говорили, что это наши "сущности". Это и есть Entity, которыми мы будем управлять при помощи `EntityManager`. Напишем простой тест по сохранению сущности категории:

```java
@Test
public void shouldPersistCategory() {
	Category cat = new Category();
	cat.setTitle("new category");
	// JUnit обеспечит тест свежим EntityManager'ом
	em.persist(cat);
}
```

Но сразу этот тест не заработает, т.к. мы получим различные ошибки, которые нам помогут понять, что такое сущности:

- `Unknown entity: hibernate.model.Category`  
    Почему же Hibernate не понимает, что `Category` это `entity`? Всё дело в том, что сущности должны быть описаны по стандарту JPA.  
    Классы сущностей должны быть аннотированы аннотацией `@Entity`, как сказано в главе "2.1 The Entity Class" спецификации JPA.
    
- `No identifier specified for entity: hibernate.model.Category`  
    Для сущностей должен быть указан уникальный идентификатор, по которому можно отличить одну запись от другой.  
    Согласно главе "2.4 Primary Keys and Entity Identity" спецификации JPA "Every entity must have a primary key", т.е. каждая сущность должна иметь "первичный ключ". Такой первичный ключ должен быть указан аннотацией `@Id`
    
- `ids for this class must be manually assigned before calling save()`  
    Идентификатор должен откуда-то появиться. Его можно указать вручную, а можно получить автоматически.  
    Поэтому, как и указано в главах "11.2.3.3 GeneratedValue" и "11.1.20 GeneratedValue Annotation", мы можем указать аннотацию `@GeneratedValue`.
    

чтобы класс категории стал сущностью мы должны выполнить следующие изменения:

```java
@Entity
public class Category {
    @Id
    @GeneratedValue
    private Long id;
```

аннотация `@Id` указыает на то, какой использовать `Access Type`. Подробнее про тип доступа можно прочитать в спецификации JPA, в разделе "2.3 Access Type". Если очень кратко, то т.к. мы указали `@Id` над полем (`field`), то тип доступа будет по умолчанию `field-based`, а не `property-based`. Следовательно, провайдер JPA будет читать и сохранять значения напрямую из полей. Если бы мы поместили `@Id` над геттером, то использовался бы `property-based` доступ, т.е. через геттер и сеттер. При выполнении теста мы видим в том числе то, какие запросы отправляются в базу (благодаря опции `hibernate.show_sql`). Но при сохранении мы не видим никаких `insert`'ов. Получается, что мы на самом деле ничего не сохранили? JPA позволяет синхронизировать контекст персистенции и БД при помощи метода `flush`:

```java
entityManager.flush();
```

Но если мы его сейчас выполним, то получим ошибку: _no transaction is in progress_. И тут наступает пора узнать про то, как JPA использует транзакции.

![JPA : Знакомство с технологией - 8](https://cdn.javarush.com/images/article/eace2b76-9155-45c2-a579-7a50141de44c/256.webp)

## JPA Transactions

в основе JPA лежит понятие контекст персистенции (Persistence Context). Это место, где живут сущности. А мы управляем сущностями через `EntityManager`. Когда мы выполняем комманду `persist`, то мы помещаем сущность в контекст. Точнее, мы говорим `EntityManager`'у, что это нужно сделать. Но контекст этот — это просто некоторая область хранения. Его даже иногда называют "кэшем первого уровня". Но его нужно соединить с базой данных. Комманда `flush`, которая ранее у нас упала с ошибкой, синхронизирует данные из контекста персистенции с БД. Но для этого требуется транспорт и этим транспортом является транзакция. Транзакции в JPA описаны в разделе спецификации "7.5 Controlling Transactions". Для использования транзакций в JPA есть специальный API:

```java
entityManager.getTransaction().begin();
entityManager.getTransaction().commit();
```

добавить управление транзакциями в наш код, который выполняется до тестов и после:

```java
@Before
public void init() {
	EntityManagerFactory emf = Persistence.createEntityManagerFactory( "JavaRush" );
	em = emf.createEntityManager();
	em.getTransaction().begin();
}
@After
public void close() {
	if (em.getTransaction().isActive()) {
		em.getTransaction().commit();
        }
	em.getEntityManagerFactory().close();
	em.close();
}
```

После добавления мы увидим в логе insert выражение на языке SQL, которых ранее не было:

![JPA : Знакомство с технологией - 9](https://cdn.javarush.com/images/article/d2bc6bda-19a5-440e-b160-06cc43859a6f/256.webp)

Изменения, накопленные в `EntityManager` было при помощи транзакции закоммичены (подтверждены и сохранены) в БД. Давайте попробуем теперь найти нашу сущность. Создадим тест на поиск сущности по её ID:

```java
@Test
public void shouldFindCategory() {
	Category cat = new Category();
	cat.setTitle("test");
	em.persist(cat);
	Category result = em.find(Category.class, 1L);
	assertNotNull(result);
}
```

получим ранее сохранённую нами сущность, но в логе мы не увидим SELECT запросов. А всё по тому, что мы говорим: "Менеджер сущностей, найди пожалуйста мне сущность Категория с ID=1". А менеджер сущностей сначала смотрит у себя в контексте (использует его своего рода кэш), и только если не находит — идёт искать в БД. Стоит изменить ID на 2 (такого нет, мы сохранили только 1 экземпляр), как мы увидим, что `SELECT` запрос появляется. Потому что в контексте не найдено сущностей и `EntityManager` пытается найти сущность БД.. Существуют разные комманды, которыми мы можем управлять состоянием сущности в контексте. Переход сущности из одного состояния в другое называется жизненным циклом сущности — `lifecycle`.

![JPA : Знакомство с технологией - 10](https://cdn.javarush.com/images/article/b86aabbf-c2d7-4b4a-be74-69849ce0b331/256.webp)

## Entity Lifecycle

Жизненный цикл сущностей описан в спецификации JPA в главе "3.2 Entity Instance’s Life Cycle". Т.к. сущности живут в контексте и ими управляет `EntityManager`, то говорят, что сущности управляемые, т.е. managed. Давайте посмотрим на этапы жизни сущности:

```java
// 1. New или Transient (временный)
Category cat = new Category();
cat.setTitle("new category");
// 2. Managed или Persistent
entityManager.persist(cat);
// 3. Транзакция завершена, все сущности в контексте detached
entityManager.getTransaction().begin();
entityManager.getTransaction().commit();
// 4. Сущность изымаем из контекста, она становится detached
entityManager.detach(cat);
// 5. Сущность из detached можно снова сделать managed
Category managed = entityManager.merge(cat);
// 6. И можно сделать Removed. Интересно, что cat всё равно detached
entityManager.remove(managed);
```

![JPA : Знакомство с технологией - 11](https://cdn.javarush.com/images/article/9fbfeb41-128b-4aba-bfd6-2a0cc0ed1315/512.webp)

![JPA : Знакомство с технологией - 12](https://cdn.javarush.com/images/article/6e5ca9c6-4c4d-449c-83b6-177012590ed6/256.webp)

## Mapping

В JPA мы можем описать отношения сущностей между друг другом. Вспомним, что мы уже разбирали отношения сущностей между друг другом, когда мы разбирались с нашей доменной моделью. Тогда мы использовали ресурс [quickdatabasediagrams.com](https://app.quickdatabasediagrams.com/#/):

![JPA : Знакомство с технологией - 13](https://cdn.javarush.com/images/article/09dbb7c0-a02e-4d77-8f54-008457c888f1/512.webp)

Установление связей между сущностями называется маппингом или ассоциированием (Association Mappings). Виды ассоциаций, которые могут быть установлены при помощи JPA представлены ниже:

![JPA : Знакомство с технологией - 14](https://cdn.javarush.com/images/article/4afb8b7e-a138-4fa3-902a-8be961395103/512.webp)

сущность `Topic`, которая описывает тему. Что мы можем сказать про отношение `Topic` к `Category`? Много `Topic` будут принадлежать одной категории. Следовательно, нам нужна ассоциация `ManyToOne`. Выразим эту связь на языке JPA:

```java
@ManyToOne
@JoinColumn(name = "category_id")
private Category category;
```

какие аннотации ставить, можно запомнить, что последняя часть отвечает за поле, над которым указана аннотация. `ToOne` — конкретный экземпляр. `ToMany` — коллекции. Сейчас у нас связь односторонняя. Давайте сделаем из неё двустороннюю связь. Добавим в `Category` знание о всех `Topic`, которые входят в эту категорию. Оканчиваться должен на `ToMany`, потому что у нас список `Topic`. То есть отношение "Ко многим" темам. Остаётся вопрос — `OneToMany` или `ManyToMany`:

![JPA : Знакомство с технологией - 15](https://cdn.javarush.com/images/article/3d94a500-c55e-4a39-a423-c8fb7ed119b1/512.webp)

"[Explain ORM oneToMany, manyToMany relation like I'm five](https://dev.to/mah3uz/explain-orm-onetomany-manytomany-relation-like-im-five-94i)". Если категория имеет связь с `ToMany` топиков, то каждый из этих топиков может иметь только одну категорию, то будет `One`, а иначе `Many`. Таким образом, в `Category` список всех тем будет выглядеть следующий образом:

```java
@OneToMany(cascade = CascadeType.ALL)
@JoinColumn(name = "topic_id")
private Set<Topic> topics = new HashSet<>();
```

в сущности `Category` описать геттер для получения списка всех тем:

```java
public Set<Topic> getTopics() {
	return this.topics;
}
```

Двунаправленные отношения — сложный для автоматического отслеживания момент. JPA перекладывает эту обязанность на разработчика. когда мы устанавливаем в сущности `Topic` связь с `Category`, мы должны обеспечить непротиворечивость данных самостоятельно.

```java
public void setCategory(Category category) {
	category.getTopics().add(this);
	this.category = category;
}
```

Напишем для проверки простой тест:

```java
@Test
public void shouldPersistCategoryAndTopics() {
	Category cat = new Category();
	cat.setTitle("test");
	Topic topic = new Topic();
	topic.setTitle("topic");
	topic.setCategory(cat);
 	em.persist(cat);
}
```

Маппинг. при помощи каких средств это достигается.

- "[Ultimate Guide – Association Mappings with JPA and Hibernate](https://thoughts-on-java.org/ultimate-guide-association-mappings-jpa-hibernate/)".
- "[JPA и связи между объектами](https://easyjava.ru/data/jpa/jpa-i-svyazi-mezhdu-obektami/)"
- "[Связанные сущности в Hibernate](http://java-online.ru/hibernate-entities.xhtml)"

![JPA : Знакомство с технологией - 16](https://cdn.javarush.com/images/article/f29f552b-e340-43c1-92a8-97c72fd746dd/256.webp)

## JPQL

JPA вводит инструмент — запросы на языке Java Persistence Query Language. Этот язык похож на SQL, но использует объектную модель Java, а не SQL таблицы. Рассмотрим пример:

```java
@Test
public void shouldPerformQuery() {
	Category cat = new Category();
	cat.setTitle("query");
	em.persist(cat);
	Query query = em.createQuery("SELECT c from Category c WHERE c.title = 'query'");
 	assertNotNull(query.getSingleResult());
}
```

в запросе мы использовали указание на сущность `Category`, а не таблицу. А так же на поле этой сущности `title`. JPQL предоставляет множество полезных возможностей и претендует на отдельную статью. Подробнее можно ознакомиться в обзоре:

- "[Ultimate Guide to JPQL Queries with JPA and Hibernate](https://thoughts-on-java.org/jpql/)"

![JPA : Знакомство с технологией - 17](https://cdn.javarush.com/images/article/eb27dd97-f714-4613-80b3-de46af969c7d/256.webp)

## Criteria API

JPA вводит инструмент динамического построения запросов. Пример использования Criteria API:

```java
@Test
public void shouldFindWithCriteriaAPI() {
	Category cat = new Category();
	em.persist(cat);
	CriteriaBuilder cb = em.getCriteriaBuilder();
	CriteriaQuery<Category> query = cb.createQuery(Category.class);
	Root<Category> c = query.from(Category.class);
	query.select(c);
	List<Category> resultList = em.createQuery(query).getResultList();
	assertEquals(1, resultList.size());
}
```

равносилен выполнению запроса "`SELECT c FROM Category c`". **Criteria API**

- [JPA Criteria API Queries](https://www.objectdb.com/java/jpa/query/criteria)
- [JPA Criteria](https://easyjava.ru/data/jpa/jpa-criteria/)
- [Динамические типобезопасные запросы в JPA 2.0](https://www.ibm.com/developerworks/ru/library/j-typesafejpa/index.html)

JPA предоставляет огромное количество возможностей и инструментов. Каждый из них требует опыта и знаний. Даже в рамках обзора JPA вышло упоминуть не всё, не говоря уже о детальном погружении. Но надеюсь, после прочтения стало понятнее, что вообще такое ORM и JPA, как это работает и что с этим можно сделать.

- [JPA - Введение](https://www.youtube.com/watch?v=r8SlsJjmm8o&list=PLCA5CB42F5A816A17&index=5)
- [Thoughts On Java - Hibernate Beginner](https://www.youtube.com/watch?v=uVLujq7_35E&list=PL50BZOuKafAYFT_F4Yris5Vj2ApwzUfmR)
- [Thoughts On Java - Hibernate Tips](https://www.youtube.com/watch?v=4deURytlQLI&list=PL50BZOuKafAbXxVJiD9csunZfQOJ5X7hP)
- [Advanced JPA Topics](https://www.objectdb.com/java/jpa/persistence/advanced)

## 3 JVM

Виртуальная машина Java — основная часть платформы Java Runtime Environment (JRE), которая интерпретирует байт-код Java для запуска программ.

использование JVM для запуска программы Java в любой операционной среде. В её основе реализуется принцип WORA (Write once, run anywhere — «написал один раз, запускай везде»), который сильно упростил процессы разработки.

_Техническое:_ JVM — спецификация программы, которая обеспечивает среду выполнения кода Java.

_Неофициальное:_ JVM запускает приложения Java, c помощью настроенных параметров для управления всеми программными ресурсами.

## Что делает виртуальная машина Java

JVM выполняет две основные функции:

- позволяет запускать Java-программы на любом устройстве или в любой операционной системе;
- даёт доступ к управлению памятью программ и её оптимизации.

JVM — среда выполнения со стандартизированной конфигурацией, мониторингом и управлением, она естественным образом подходит для контейнерной разработки с использованием таких технологий, как [Docker](https://www.nic.ru/help/kratkij-ekskurs-v-docker_11188.html).

## Роль Java Virtual Machine

место виртуальной машины Java в выполняемых процессах.

![47](https://www.nic.ru/help/upload/image/unnamed\(5\).jpg)

JVM образует слой между операционной системой и программами Java.

скомпилированная Java-программа будет связываться с Java Virtual Machine, а JVM будет общаться с операционной системой, являясь своего рода посредником между скомпилированными файлами классов и операционной системой.

## Файл .class и байт-код

Когда дело доходит до выполнения программы, главное, что интересует виртуальную машину Java — это определённый формат файла – .class.

Файлы классов содержат наполовину скомпилированный код, называемый байт-кодом, который в свою очередь предоставляет JVM инструкции, таблицу символов и другую вспомогательную информацию.

## Архитектура виртуальной машины Java
Понять, что такое виртуальная машина Java, будет немного проще, если познакомиться с её архитектурой и тем, как она работает. Поэтому важно рассмотреть строение JVM и особенности её частей.

Java Virtual Machine состоит из трёх отдельных компонентов:
- загрузчик классов;
- область памяти;
- исполнительный механизм.

![](https://www.nic.ru/help/upload/image/unnamed%20\(1\)\(4\).jpg)

1. **Загрузчик классов**

Загрузчики классов отвечают за динамическую загрузку файлов .class в виртуальную машину Java и сохранения байт-кода в области метода JVM, о которой мы поговорим чуть позже.

Загрузчик классов Java бывает трёх типов:

- **BootStrap ClassLoader** (Загрузчик классов начальной загрузки) — это машинный код, который запускает операцию, когда её вызывает JVM. Его задача — загрузить классы из папки rt.jar.
- **Extension ClassLoader** (Загрузчик классов расширений) является дочерним элементом Bootstrap ClassLoader и загружает расширения основных классов Java из каталога jre/lib/ext или любого другого каталога, на который указывает java.ext.dirs.
- **System ClassLoader** (Системный загрузчик классов) загружает классы, найденные в переменной среды CLASSPATH, -classpath или параметре командной строки -cp.

2. **Область памяти**

Область памяти или, как её ещё называют, _область данных времени выполнения_ JVM состоит из 5 частей:

- **Область метода** предназначена для хранения данных файлов .class: например, метаданные, данные полей и методов, а также код метода. Область метода создаётся автоматически при запуске виртуальной машины, и для каждой ВМ существует только одна область метода.
- **Область кучи**. В куче хранятся все объекты и соответствующие им переменные экземпляра. Когда мы создаём новый экземпляр класса, он сразу же загружается в область кучи, которая остается единственной во время выполнения задачи.
- **Область стека**. В неё загружаются все локальные переменные, вызовы методов и частичные результаты.
- **Регистры ПК**. В регистре ПК хранится адреса виртуальных машин Java, выполняющих операцию в данный момент. В Java каждый поток получает свой собственный регистр ПК.
- **Стеки нативных методов**. Нативные методы — это методы, написанные на C или C++. Виртуальная машина JVM хранит стеки, которые поддерживают такие методы, с отдельным стеком собственных методов, выделенным для каждого потока.

3. **Исполнительный механизм**

Этот тип программного обеспечения используется для тестирования оборудования и программного обеспечения. При этом он не сохраняет никакой информации о тестируемом продукте.

Он стоит из:
- интерпретатора;
- JIT-компилятора;
- сборщика мусора.

Перед выполнением программы интерпретатор и компилятор JIT («Just-in-time» — «точно в нужное время») преобразуют байт-код в машинные инструкции. Интерпретатор делает это построчно.

когда в сценарии обнаруживается повторяющийся код, к нему подключается JIT-компилятор для ускорения операции. Затем он компилирует байт-код и заменяет его собственным кодом. Такой процесс повышает производительность всей системы.

Но за что же в таком случае отвечает сборщик мусора? В некоторых других языках программирования (как C++) освобождение памяти от объектов без циклических ссылок зависит лишь от самого программиста. Однако в JVM этим занимается сборщик мусора, что оптимизирует использование памяти.

сборка мусора выполняется в JVM автоматически через определённые отрезки времени и не требует отдельного внимания специалистов. этот процесс можно попробовать принудительно запустить, вызвав System.gc(), но нет никакой гарантии, что это сработает.

виртуальная машина Java изначально предназначалась только для запуска и выполнения программ Java, на сегодняшний день она способна поддерживать многие языки сценариев и программирования, что лишь укрепило популярность данной платформы.

---

что происходит под капотом JVM после того, как мы запускаем написанное Java приложение.

## 2. Компиляция в байт-код

![Компиляция и исполнение Java приложений под капотом - 2](https://cdn.javarush.com/images/article/fef10693-b1f3-479a-a02e-29414cdc2a79/1024.webp)

Начнем с теории. Когда мы пишем какое-либо приложение, мы создаем файл с расширением `.java` и помещаем в него код на языке программирования Java. Такой файл, содержащий код, понятный человеку, называется **файлом с исходным кодом**. После того, как файл с исходным кодом готов, нужно его выполнить! Но на стадии в нем содержится информация, понятная только человеку. Java — мультиплатформенный язык программирования. Это значит, что программы, написанные на языке Java, можно выполнять на любой платформе, где установлена специальная исполняющая система Java. Такая система называется Java Virtual Machine (JVM). Для того, чтобы перевести программу из исходного кода в код, понятный JVM, нужно её скомпилировать. Код, понятный JVM называется байт-кодом и содержит набор инструкций, которые в дальнейшем будет исполнять виртуальная машина. Для компиляции исходного кода в байт-код существует компилятор `javac`, входящий в поставку JDK (Java Development Kit). На вход компилятор принимает файл с расширением `.java`, содежащий исходный код программы, а на выходе выдает файл с расширением `.class`, содержащий байт-код, необходимый для исполнения программы виртуальной машиной. После того, как программа была скомпилирована в байт-код, она может быть выполнена с помощью виртуальной машины.
## 3. компиляции и выполнения программы

программа, содержащаяся в файле `Calculator.java`, которая принимает 2 численных аргумента командной строки и печатает результат их сложения:

```java
class Calculator {
    public static void main(String[] args){
        int a = Integer.valueOf(args[0]);
        int b = Integer.valueOf(args[1]);

        System.out.println(a + b);
    }
}
```

скомпилировать эту программу в байт-код, воспользуемся компилятором `javac` в командной строке:

```java
javac Calculator.java
```

После компиляции на выходе мы получаем файл с байт-кодом `Calculator.class`, который мы можем исполнить при помощи установленной на нашем компьютере java-машины командой java в командной строке:

```java
java Calculator 1 2
```

после названия файла были указаны 2 аргумента командной строки — числа 1 и 2. После выполнения программы, в командной строке выведется число 3. класс, который живет сам по себе. Но что, если класс находится в каком либо пакете? Смоделируем такую ситуацию: создадим директории `src/ru/javarush` и поместим туда наш класс. он выглядит следующим образом (добавили имя пакета в начале файла):

```java
package ru.javarush;

class Calculator {
    public static void main(String[] args){
        int a = Integer.valueOf(args[0]);
        int b = Integer.valueOf(args[1]);

        System.out.println(a + b);
    }
}
```

Скомпилируем такой класс следующей командой:

```java
javac -d bin src/ru/javarush/Calculator.java
```

В этом примере мы использовали дополнительную опцию компилятора `-d bin`, которая складывает скомпилированные файлы в директорию `bin` со структурой, аналогичной директории `src`, при этом директория `bin` должна быть создана заранее. Такой прием используется, чтобы не путать файлы с исходным кодом с файлами с байт-кодом. Перед запуском скомпилированной программы стоит пояснить понятие `classpath`. `Classpath` — это путь, относительно которого виртуальная машина будет искать пакеты и скомпилированные классы. Тоесть, таким образом мы говорим виртуальной машине какие директории в файловой системе являются корневыми для иерархии пакета Java. `Classpath` можно укзать при запуска программы с помощью флага `-classpath`. Запуск программы осуществляем с помощью команды:

```java
java -classpath ./bin ru.javarush.Calculator 1 2
```

В этом примере нам потребовалось указать полное имя класса, включая имя пакета, в котором он находится. Финальное дерево файлов выглядит следующим образом:

```java
├── src
│     └── ru
│          └── javarush
│                  └── Calculator.java
└── bin
      └── ru
           └── javarush
                   └── Calculator.class
```

## 4. Выполнение программы виртуальной машиной

запустили написанную программу. Но что же происходит в момент запуска скомпилированной программы виртуальной машиной? Для начала разберемся, что означают понятия компиляции и интерпретации кода. **Компиляция** — трансляция программы, составленной на исходном языке высокого уровня, в эквивалентную программу на низкоуровневом языке, близком машинному коду. **Интерпретация** — пооператорный (покомандный, построчный) анализ, обработка и тут же выполнение исходной программы или запроса (в отличие от компиляции, при которой программа транслируется без её выполнения). Язык Java обладает как компилятором (`javac`), так и интерпретатором, в роли которого выступает виртуальная машина, которая построчно преобразует байт-код в машинный код и тут же его исполняет. Таким образом, когда мы запускаем скомпилированную программу, виртуальная машина начинает её интерпретацию, то есть построчное преобразование байт-кода в машинный код, а так же его исполнение. К сожалению, чистая интерпретация байт-кода является довольно долгим процессом и делает язык java медленным в сравнении с его конкурентами. Дабы избежать этого, был введен механизм, позволяющий ускорить интерпретацию байт-кода виртуальной машиной. Этот механизм называется Just-in-time компиляцией (JITC).

## 5. Just-in-time (JIT) компиляция

Простыми словами, механизм Just-In-Time компиляции заключается в следующем: если в программе присутствуют части кода, которые выполняются много раз, то их можно скомпилировать один раз в машинный код, чтобы в будущем ускорить их выполнение. После компиляции такой части программы в машинный код, при каждом следующем вызове этой части программы виртуальная машина будет сразу выполнять скомпилированный машинный код, а не интерпретировать его, что естественно ускорит выполнение программы. Ускорение работы программы достигается за счет увеличения потребления памяти (где-то же нам нужно хранить скомпилированный машинный код!) и за счет увеличения временных затрат на компиляцию во время исполнения программы. JIT компиляция — довольно сложный механизм, поэтому пройдемся по верхам. Всего существует 4 уровня JIT компиляции байт-кода в машинный код. Чем выше уровень компиляции, тем он сложнее, но и одновременно выполнение такого участка будет быстрее, чем участка с меньшим уровнем. JIT — компилятор самостоятельно решает, какой уровень компиляции задать для каждого фрагмента программы на основе того, как часто выполняется этот фрагмент. Под капотом JVM использует 2 JIT-компилятора — C1 и C2. C1 компилятор так же называется клиентским компилятором и способен скомпилировать код только до 3-его уровня. За 4-ый, самый сложны и быстрый уровень компиляции отвечает компилятор C2.

[![AI Native University](https://javarush.com/assets/images/site/featured-content/ai-native-developer/ru.png)](https://javarush.com/s/dynamic_banners_ai_native_university_ru)

![Компиляция и исполнение Java приложений под капотом - 3](https://cdn.javarush.com/images/article/59e6a0ff-a9cc-4570-82b9-ff2c047cade0/1024.webp)

Из вышесказанного можно сделать вывод о том, что для простых, клиентских приложений, выгоднее использовать компилятор C1, так как в этом случае нам важно как быстро стартует приложение. Серверные, долгоживущие приложения могут стартовать большее количество времени, однако в дальнейшем должны работать и выполнять свою функцию быстро — тут нам подойдет компилятор C2.

При запуске Java — программы на x32 версии JVM мы в ручную можем указать, какой режим мы хотим использовать, при помощи флагов `-client` и `-server`. При указании флага `-client` JVM не будет производить сложные оптимизации с байт-кодом, что ускорит время старта приложения и уменьшит количество потребляемой памяти. При указании флага `-server` приложение будет стартовать большее количество времени из-за сложных оптимизаций байт-кода и будет использовать больше памяти для хранения машинного кода, однако в дальнейшем работать такая программа будет быстрее. В x64 версии JVM флаг `-client` игнорируется и по умолчанию используется серверная конфигурация приложения.

1. Компилятор _javac_ преобразует исходный код программы в байт-код, который может быть выполнен на любой платформе, на которой установлена виртуальная машина Java; 
2. После компиляции JVM интерпретирует получившийся байт-код;
3. Для ускорения работы Java-приложений, JVM использует механизм Just-In-Time компиляции, который преобразует наиболее часто выполняемые участки программы в машинный код и хранит их в памяти.
    

## 4 Жизненный цикл Java объекта

при создании любого объекта Java-машиной под него выделяется память. В реальной большой программе создаются десятки и сотни тысяч объектов, под каждый из которых в памяти выделяется свой кусочек.

сколько существуют все эти объекты? “Живут” ли они все время, пока работает наша программа? Разумеется, нет. При всех достоинствах Java-объектов, они не бессмертны. У объектов есть собственный жизненный цикл. Сегодня мы чуть-чуть отдохнем от написания кода и рассмотрим этот процесс. он является очень важным для понимания работы программы и распоряжения ресурсами.

```java
Cat cat = new Cat();//вот сейчас и начался жизненный цикл нашего объекта Cat!
```

Вначале виртуальная Java-машина выделяет необходимый объем памяти для создания объекта. Потом она создает на него ссылку, в нашем случае — `cat`, чтобы иметь возможность его отслеживать. После этого происходит инициализация всех переменных, вызов конструктора и вот — наш свежий объект уже живет своей жизнью. Срок жизни у объектов разный, точных цифр здесь не существует. в течение какого-то времени он живет внутри программы и выполняет свои функции. объект является “живым” пока на него есть ссылки.

```java
public class Car {

   String model;

   public Car(String model) {
       this.model = model;
   }

   public static void main(String[] args) {
       Car lamborghini  = new Car("Lamborghini Diablo");
       lamborghini = null;

   }

}
```

В методе `main()` объект машины Lamborghini Diablo перестает быть живым уже на второй строке. На него была всего одна ссылка, а теперь этой ссылке был присвоен `null`. Поскольку на Lamborghini Diablo не осталось ссылок, он становится “мусором”. Ссылку при этом не обязательно обнулять:

```java
public class Car {

   String model;

   public Car(String model) {
       this.model = model;
   }

   public static void main(String[] args) {
       Car lamborghini  = new Car("Lamborghini Diablo");

       Car lamborghiniGallardo = new Car("Lamborghini Gallardo");
       lamborghini = lamborghiniGallardo;
   }

}
```

создали второй объект, после чего взяли ссылку `lamborghini` и присвоили ей этот новый объект. Теперь на объект `Lamborghini Gallardo` указывает две ссылки, а на объект `Lamborghini Diablo` — ни одной. объект `Diablo` становится мусором. И в этот момент в работу вступает встроенный механизм Java под названием сборщик мусора, или по-другому — Garbage Collector, GC.

Сборщик мусора — внутренний механизм Java, который отвечает за освобождение памяти, то есть удаление из нее ненужных объектов. Мы не зря выбрали для его изображения картинку с роботом-пылесосом. Ведь сборщик мусора работает примерно так же: в фоновом режиме он “ездит” по твоей программе, собирает мусор, и при этом ты с ним практически не взаимодействуешь. Его работа — удалять объекты, которые уже не используются в программе. Таким образом он освобождает в компьютере память для других объектов. Помнишь в начале лекции мы говорили, что в обычной жизни тебе приходится следить за состоянием твоего компьютера и удалять старые файлы? Так вот, в случае с Java-объектами сборщик мусора делает это вместо тебя. Garbage Collector запускается многократно в течение работы твоей программы: его не надо вызывать специально и отдавать команды, хотя технически это возможно. Позднее мы еще поговорим о нем и разберем процесс его работы более детально. В момент, когда сборщик мусора добрался до объекта, перед самым его уничтожением, у объекта вызывается специальный метод — `finalize()`. Его можно использовать, чтобы освободить какие-то дополнительные ресурсы, которые использовал объект. Метод `finalize()` принадлежит классу `Object`. То есть, наравне с `equals()`, `hashCode()` и `toString()`, с которыми ты уже познакомился ранее, он есть у любого объекта. Его отличие от других методов в том, что он... как бы это сказать... весьма своенравен. А именно — перед уничтожением объекта он вызывается далеко не всегда. Программирование — штука точная. Программист говорит компьютеру что-то сделать — компьютер это и делает. Ты, полагаю, уже привык к такому поведению, и тебе поначалу может быть сложно принять идею: “Перед уничтожением объектов вызывается метод `finalize()` класса `Object`. Или не вызывается. Как повезет!” Тем не менее, это действительно так. Java-машина сама определяет, вызывать метод `finalize()` в каждом конкретном случае или нет. Например, давай попробуем ради эксперимента запустить такой код:

```java
public class Cat {

   private String name;

   public Cat(String name) {
       this.name = name;
   }

   public Cat() {
   }

   public static void main(String[] args) throws Throwable {

       for (int i = 0 ; i < 1000000; i++) {

           Cat cat = new Cat();
           cat = null;//вот здесь первый объект становится доступен сборщику мусора
       }
   }

   @Override
   protected void finalize() throws Throwable {
       System.out.println("Объект Cat уничтожен!");
   }
}
```

Мы создаем объект `Cat` и уже в следующей строчке кода обнуляем единственную ссылку на него. И так — миллион раз. Мы явно переопределили метод `finalize()`, и он должен миллион раз вывести строку в консоль, каждый раз перед уничтожением объекта `Cat`. Но нет! на моем компьютере он отработал всего 37346 раз! То есть только в 1 случае из 27-ми установленная у меня Java-машина принимала решение вызвать метод `finalize()` — в остальных случаях сборка мусора проходила без этого. Попробуй запустить этот код у себя: скорее всего, результат будет отличаться. Как видишь,`finalize()` трудно назвать надежным партнером :) Поэтому небольшой совет на будущее: не стоит полагаться на метод `finalize()` в случае с освобождением каких-то критически важных ресурсов. Может JVM его вызовет, а может нет. Если твой объект при жизни занимал какие-то суперважные для производительности ресурсы, например, держал открытым соединение с базой данных, лучше создай в своем классе специальный метод для их освобождения и вызови его явно, когда объект уже будет не нужен. Так ты точно будешь знать, что производительность твоей программы не пострадает. В самом начале мы сказали, что работа с памятью и удаление мусора очень важны, и это действительно так. Неподобающая работа с ресурсами и непонимание процесса сборки ненужных объектов могут привести к утечке памяти. Это одна из самых известных ошибок в программировании. Неправильно написанный программистом код может привести к тому, что для вновь созданных объектов каждый раз будет выделяться новая память, при этом старые, ненужные объекты будут недоступны для удаления сборщиком мусора. Раз уж мы привели аналогию с роботом пылесосом, представь, что будет, если перед запуском робота разбросать по дому носки, разбить стеклянную вазу и оставить на полу разобранный конструктор Lego. Робот, попытается что-то сделать, но в один прекрасный момент он застрянет.

Для его правильной работы нужно держать пол в нормальном состоянии и убирать оттуда все, с чем не справится пылесос. По такому же принципу работает и сборщик мусора. Если в программе будет оставаться много объектов, которые он не может собрать (как носок или Lego для робота-пылесоса), в один прекрасный момент память закончится. И зависнет не только написанная тобой программа, но и все остальные программы, запущенные в этот момент на компьютере. Для них тоже не будет хватать памяти. Вот так выглядят в Java жизненный цикл объектов и сборщик мусора.


### Сереализация

Бинарная сереализация - для внутренних задач, где контроль обоих сторон процесса.
Для обмена с внешними системами — текстовые форматы и библиотеки: JSON, XML, Jackson, Gson, JAXB.

