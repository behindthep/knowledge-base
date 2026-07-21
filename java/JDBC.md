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
