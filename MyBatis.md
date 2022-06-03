# 工具类

## SQL

        `SQL`工具类。可以很方便地在`Java`代码中动态构建`SQL`语句。

```java
//SELECT id,name FROM user WHERE (id = #{id} AND name = #{name}) 
// OR (name is null)
SQL sql = new SQL(){{
            SELECT("id,name");
            FROM("user");
            WHERE("id = #{id}");
            WHERE("name = #{name}");
            OR();
            WHERE("name is null");
}};
```

        `SQL`继承自`AbstractSQL`只重写了该类的`getSelf()`方法：

```java
public class SQL extends AbstractSQL<SQL> {
  @Override
  public SQL getSelf() {
    return this;
  }
}
```

        `AbstractSQL`类中维护了一个`SQLStatement`内部类的实例和一系列构造`SQL`语句的方法。

```java
public abstract class AbstractSQL<T> {

  private static final String AND = ") \nAND (";
  private static final String OR = ") \nOR (";

  private final SQLStatement sql = new SQLStatement();

  public abstract T getSelf();

  public T SELECT(String... columns) {
    sql().statementType = SQLStatement.StatementType.SELECT;
    sql().select.addAll(Arrays.asList(columns));
    return getSelf();
  }

  public T SELECT_DISTINCT(String columns) {
    sql().distinct = true;
    SELECT(columns);
    return getSelf();
  }

  ...
}
```

        `SQLStatement`内部类用于描述一个`SQL`语句，该类中通过`StatementType`确定SQL语句的类型。`SQLStatement`类中还维护了一系列的`ArrayList`属性，当调

用`SELECT(、UPDATE()`等方法时，这些方法的参数内容会记录在这些`ArrayList`对象中：

```java
private static class SQLStatement {

    public enum StatementType {
      DELETE, INSERT, SELECT, UPDATE
    }

    StatementType statementType;
    List<String> sets = new ArrayList<>();
    List<String> select = new ArrayList<>();
    List<String> tables = new ArrayList<>();
    List<String> join = new ArrayList<>();
    List<String> innerJoin = new ArrayList<>();
    List<String> outerJoin = new ArrayList<>();
    List<String> leftOuterJoin = new ArrayList<>();
    List<String> rightOuterJoin = new ArrayList<>();
    List<String> where = new ArrayList<>();
    List<String> having = new ArrayList<>();
    List<String> groupBy = new ArrayList<>();
    List<String> orderBy = new ArrayList<>();
    List<String> lastList = new ArrayList<>();
    List<String> columns = new ArrayList<>();
    List<List<String>> valuesList = new ArrayList<>();
    boolean distinct;
    String offset;
    String limit;

    ...
}
```

        `AbstrastSQL`类重写了`toString()`方法，该方法中会调用`SQLStatement`对象的`sql()`方法生成`SQL`字符串：

```java
// AbstractSQL 类
public String toString() {
    StringBuilder sb = new StringBuilder();
    sql().sql(sb);
    return sb.toString();
}
```

```java
// SQLStatement 类
public String sql(Appendable a) {
      SafeAppendable builder = new SafeAppendable(a);
      if (statementType == null) {
        return null;
      }

      String answer;

      switch (statementType) {
        case DELETE:
          answer = deleteSQL(builder);
          break;

        case INSERT:
          answer = insertSQL(builder);
          break;

        case SELECT:
          answer = selectSQL(builder);
          break;

        case UPDATE:
          answer = updateSQL(builder);
          break;

        default:
          answer = null;
      }

      return answer;
}


private String selectSQL(SafeAppendable builder) {
      // 追加 select 子句
      if (distinct) {
        sqlClause(builder, "SELECT DISTINCT", select, "", "", ", ");
      } else {
        sqlClause(builder, "SELECT", select, "", "", ", ");
      }
      // 追加 from 子句
      sqlClause(builder, "FROM", tables, "", "", ", ");
      // 追加 join 子句
      joins(builder);
      // 追加 where 子句
      sqlClause(builder, "WHERE", where, "(", ")", " AND ");
      // 追加 group by 子句
      sqlClause(builder, "GROUP BY", groupBy, "", "", ", ");
      // 追加 having 子句
      sqlClause(builder, "HAVING", having, "(", ")", " AND ");
      // 追加 order by 子句
      sqlClause(builder, "ORDER BY", orderBy, "", "", ", ");
      limitingRowsStrategy.appendClause(builder, offset, limit);
      return builder.toString();
}


private void sqlClause(SafeAppendable builder, String keyword, List<String> parts, String open, String close,
                           String conjunction) {
      if (!parts.isEmpty()) {
        if (!builder.isEmpty()) {
          builder.append("\n");
        }
        // 拼接 SQL 关键字
        builder.append(keyword);
        builder.append(" ");
        // 拼接关键字后开始字符
        builder.append(open);
        String last = "________";
        for (int i = 0, n = parts.size(); i < n; i++) {
          String part = parts.get(i);
          // 如果 SQL 关键字对应的子句内容不为 or 或 and ，则追加连接关键字
          if (i > 0 && !part.equals(AND) && !part.equals(OR) && !last.equals(AND) && !last.equals(OR)) {
            builder.append(conjunction);
          }
          // 追加子句内容
          builder.append(part);
          last = part;
        }
        // 追加关键字后结束字符
        builder.append(close);
      }
}
```

## ScriptRunner

        `ScriptRunner`工具类用于读取脚本文件中的`SQL`语句并执行。`ScriptRunner`工具类的构造方法需要一个`java.sql.Connection`对象作为参数。创建

`ScriptRunner`对象后，调用该对象的`runScript()`方法即可，该方法接收一个读取`SQL`脚本文件的`Reader`对象作为参数：

```java
try {
    Connection con = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/db", "root", "123456");
    ScriptRunner runner = new ScriptRunner(con);
    runner.runScript(Resources.getResourceAsReader("xxx.sql"));
}
catch (Exception e){
    e.printStackTrace();
}
```

        `ScriptRunner`工具类中提供了一些属性，用于控制执行`SQL`脚本的一些行为：

```java
public class ScriptRunner {

  // SQL 异常是否中断程序执行
  private boolean stopOnError;

  // 是否抛出 SQLWarning 警告
  private boolean throwWarning;

  // 是否自动提交
  private boolean autoCommit;

  // 为 true ，批量执行文件中的 SQL 语句
  // 为 false，逐条执行 SQL 语句，默认情况下，SQL 语句以分号分隔
  private boolean sendFullScript;

  // 是否去除 Windows 系统换行符中的 \r
  private boolean removeCRs;

  // 设置 Statement 属性是否支持转义处理
  private boolean escapeProcessing = true;
  // 日志输出位置
  private PrintWriter logWriter = new PrintWriter(System.out);

  // 错误日志输出位置
  private PrintWriter errorLogWriter = new PrintWriter(System.err);
  // 脚本中 SQL 语句的分隔符，默认为分号
  private String delimiter = DEFAULT_DELIMITER;

  // 是否支持 SQL 语句分隔符单独占一行
  private boolean fullLineDelimiter;

  ...
}
```

        上面的属性，可以使用对应的 `Setter`方法来设置。`ScriptRunner`仅提供了一个`runScript`方法用于执行`SQL`脚本文件：

```java
// ScriptRunner 类
public void runScript(Reader reader) {
    // 设置事务是否自动提交
    setAutoCommit();

    try {
      if (sendFullScript) {
        // 一次性批量执行
        executeFullScript(reader);
      } else {
        // 逐条执行
        executeLineByLine(reader);
      }
    } finally {
      rollbackConnection();
    }
}


private void executeLineByLine(Reader reader) {
    StringBuilder command = new StringBuilder();
    try {
      BufferedReader lineReader = new BufferedReader(reader);
      String line;
      while ((line = lineReader.readLine()) != null) {
        // 处理每行内容
        handleLine(command, line);
      }
      commitConnection();
      checkForMissingLineTerminator(command);
    } catch (Exception e) {
      String message = "Error executing: " + command + ".  Cause: " + e;
      printlnError(message);
      throw new RuntimeSqlException(message, e);
    }
}


private void handleLine(StringBuilder command, String line) throws SQLException {
    String trimmedLine = line.trim();
    // 判断该行是否是 SQL 注释
    if (lineIsComment(trimmedLine)) {
      Matcher matcher = DELIMITER_PATTERN.matcher(trimmedLine);
      if (matcher.find()) {
        delimiter = matcher.group(5);
      }
      println(trimmedLine);
    }
    // 判断该行是否包含分号 
    else if (commandReadyToExecute(trimmedLine)) {
      // 获取分号之前的内容
      command.append(line, 0, line.lastIndexOf(delimiter));
      command.append(LINE_SEPARATOR);
      println(command);
      // 执行完整的 SQL 语句
      executeStatement(command.toString());
      command.setLength(0);
    }
    // 不包含分号，说明 SQL 语句未结束，追加到本行内容到之前读取的内容中 
    else if (trimmedLine.length() > 0) {
      command.append(line);
      command.append(LINE_SEPARATOR);
    }
}
```

## SqlRunner

        `SqlRunner`工具类对`JDBC`做了很好的封装，结合`SQL`工具类，能够很方便地通过`Java`代码执行`SQL`语句并检索`SQL`执行结果。

        方法：

                `closeConnection`：用于关闭`Connection`对象。

                `selectOne`：执行`SELECT`语句，`SQL`语句中可以使用占位符，如果`SQL`中包含占位符，则可变参数用于为参数占位符赋值，该方法只返回一条记录。若

​		查询结果行数不等于一，则会抛出`SQLException`异常。

                `selectAll`：该方法和`selectOne()`方法的作用相同，只不过该方法可以返回多条记录，方法返回值是一个`List`对象，`List`中包含多个`Map`对象，每

​		个`Map`对象对应数据库中的一行记录。

                `insert`：执行一条`INSERT`语句，插入一条记录。

                `update`：更新若干条记录。

                `delete`：删除若干条记录。

                `run`：执行任意一条`SQL`语句，最好为`DDL`语句。

```java
// SqlRunner 类
public List<Map<String, Object>> selectAll(String sql, Object... args) throws SQLException {
    try (PreparedStatement ps = connection.prepareStatement(sql)) {
      // 设置 SQL 中的占位符
      setParameters(ps, args);
      // 执行查询操作
      try (ResultSet rs = ps.executeQuery()) {
        // 将查询结果转化为 List
        return getResults(rs);
      }
    }
}


private void setParameters(PreparedStatement ps, Object... args) throws SQLException {
    for (int i = 0, n = args.length; i < n; i++) {
      if (args[i] == null) {
        throw new SQLException("SqlRunner requires an instance of Null to represent typed null values for JDBC compatibility");
      } else if (args[i] instanceof Null) {
        ((Null) args[i]).getTypeHandler().setParameter(ps, i + 1, null, ((Null) args[i]).getJdbcType());
      } else {
        // 根据参数类型获取对应的 TypeHandler
        TypeHandler typeHandler = typeHandlerRegistry.getTypeHandler(args[i].getClass());
        if (typeHandler == null) {
          throw new SQLException("SqlRunner could not find a TypeHandler instance for " + args[i].getClass());
        } else {
          // 为占位符赋值
          typeHandler.setParameter(ps, i + 1, args[i], null);
        }
      }
    }
}


private List<Map<String, Object>> getResults(ResultSet rs) throws SQLException {
    List<Map<String, Object>> list = new ArrayList<>();
    List<String> columns = new ArrayList<>();
    List<TypeHandler<?>> typeHandlers = new ArrayList<>();
    // 获取 ResultSetMetaData 对象，通过 ResultSetMetaData 获取所有列名
    ResultSetMetaData rsmd = rs.getMetaData();
    for (int i = 0, n = rsmd.getColumnCount(); i < n; i++) {
      columns.add(rsmd.getColumnLabel(i + 1));
      try {
        // 获取类的 JDBC 类型
        Class<?> type = Resources.classForName(rsmd.getColumnClassName(i + 1));
        // 根据类型获取对应的 TypeHandler
        TypeHandler<?> typeHandler = typeHandlerRegistry.getTypeHandler(type);
        if (typeHandler == null) {
          typeHandler = typeHandlerRegistry.getTypeHandler(Object.class);
        }
        typeHandlers.add(typeHandler);
      } catch (Exception e) {
        typeHandlers.add(typeHandlerRegistry.getTypeHandler(Object.class));
      }
    }
    // 遍历 ResultSet
    while (rs.next()) {
      Map<String, Object> row = new HashMap<>();
      // 将记录行转换为 Map 对象
      for (int i = 0, n = columns.size(); i < n; i++) {
        String name = columns.get(i);
        TypeHandler<?> handler = typeHandlers.get(i);
        // 通过 TypeHandler 将 JDBC 类型转换为 Java 类型
        row.put(name.toUpperCase(Locale.ENGLISH), handler.getResult(rs, name));
      }
      list.add(row);
    }
    return list;
}
```

## MetaObject

        `MetaObject`是`MyBatis`中的反射工具类。使用`MetaObject`工具类，可以很优雅地获取和设置对象的属性值。

![](image/QQ截图20220602171829.png)

![](image/QQ截图20220602171950.png)

## MetaClass

        `MetaClass`是`MyBatis`中的反射工具类，与`MetaOjbect`不同的是，`MetaObject`用于获取和设置对象的属性值，而`MetaClass`则用于获取类相关的信息。

![](image/QQ截图20220602172250.png)

## ObjectFactory

        `ObjectFactory`是`MyBatis`中的对象工厂，`MyBatis`每次创建`Mapper`映射结果对象的新实例时，都会使用一个对象工厂实例来完成。`ObjectFactory`接口只

有一个默认的实现，即`DefaultObjectFactory`，默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认构造方法，要么在参数映射存在的时候通过参数构造

方法来实例化。

        `ObjectFactory`是`MyBatis`提供的一种扩展机制。有些情况下，在得到映射结果之前需要处理一些逻辑，或者在执行该类的有参构造方法时，在传入参数之

前，要对参数进行一些处理，这时可以通过自定义`ObjectFactory`来实现。

## ProxyFactory

        `ProxyFactory`是`MyBatis`中的代理工厂，主要用于创建动态代理对象，`ProxyFactory`接口有两个不同的实现，分别为`CglibProxyFactory`和

`JavassistProxyFactory`。从实现类的名称可以看出，`MyBatis`支持两种动态代理策略，分别为`Cglib`和`Javassist`动态代理。`ProxyFactory`主要用于实现

`MyBatis`的懒加载功能。当开启懒加载后，`MyBatis`创建`Mapper`映射结果对象后，会通过`ProxyFactory`创建映射结果对象的代理对象。当我们调用代理对象的

`Getter`方法获取数据时，会执行`CglibProxyFactory`或`JavassistProxyFactory`中定义的拦截逻辑，然后执行一次额外的查询。



# 核心组件

![](image/QQ截图20220603090523.png)

        `Configuration`：用于描述`MyBatis`的主配置信息，其他组件需要获取配置信息时，直接通过`Configuration`对象获取。除此之外，`MyBatis`在应用启动时，

将`Mapper`配置信息、类型别名、`TypeHandler`等注册到`Configuration`组件中，其他组件需要这些信息时，也可以从`Configuration`对象中获取。

        `MappedStatement`：`MappedStatement`用于描述`Mapper`中的`SQL`配置信息，是对`Mapper XML`配置文件中`<select|update|delete|insert>`等标签或者

`@Select/@Update`等注解配置信息的封装。

        `SqlSession`：`SqlSession`是`MyBatis`提供的面向用户的`API`，表示和数据库交互时的会话对象，用于完成数据库的增删改查功能。`SqlSession`是`Executor`

组件的外观，目的是对外提供易于理解和使用的数据库操作接口。

        `Executor`：`Executor`是`MyBatis`的`SQL`执行器，`MyBatis`中对数据库所有的增删改查操作都是由`Executor`组件完成的。

        `StatementHandler`：`StatementHandler`封装了对`JDBC Statement`对象的操作，比如为`Statement`对象设置参数，调用`Statement`接口提供的方法与数据库交

互，等等。

        `ParameterHandler`：当`MyBatis`框架使用的`Statement`类型为`CallableStatement`和`PreparedStatement`时，`ParameterHandler`用于为`Statement`对象参数

占位符设置值。

        `ResultSetHandler`：`ResultSetHandler`封装了对`JDBC`中的`ResultSet`对象操作，当执行`SQL`类型为`SELECT`语句时，`ResultSetHandler`用于将查询结果转

换成`Java`对象。

        `TypeHandler`：`TypeHandler`是`MyBatis`中的类型处理器，用于处理`Java`类型与`JDBC`类型之间的映射。它的作用主要体现在能够根据`Java`类型调用

`PreparedStatement`或`CallableStatement`对象对应的`setXXX()`方法为`Statement`对象设置值，而且能够根据`Java`类型调用`ResultSet`对象对应的`getXXX()`获取

`SQL`执行结果。

        

        `SqlSession`组件，它是用户层面的`API`。实际上`SqlSession`是`Executor`组件的外观，目的是为用户提供更友好的数据库操作接口，这是设计模式中外观模

式的典型应用。真正执行`SQL`操作的是`Executor`组件，`Executor`可以理解为`SQL`执行器，它会使用`StatementHandler`组件对`JDBC`的`Statement`对象进行操作。

当`Statement`类型为`CallableStatement`和`PreparedStatement`时，会通过`ParameterHandler`组件为参数占位符赋值。`ParameterHandler`组件中会根据`Java`类型

找到对应的`TypeHandler`对象，`TypeHandler`中会通过`Statement`对象提供的`setXXX()`方法为`Statement`对象中的参数占位符设置值。`StatementHandler`组件使

用`JDBC`中的`Statement`对象与数据库完成交互后，当`SQL`语句类型为`SELECT`时，`MyBatis`通过`ResultSetHandler`组件从`Statement`对象中获取`ResultSet`对

象，然后将`ResultSet`对象转换为`Java`对象。



## Configuration

        `MyBatis`框架的配置信息有两种，一种是配置`MyBatis`框架属性的主配置文件；另一种是配置执行`SQL`语句的`Mapper`配置文件。`Configuration`的作用是描

述`MyBatis`主配置文件的信息。`Configuration`类中定义了一系列的属性用来控制`MyBatis`运行时的行为：

```java
public class Configuration {

  protected Class<?> configurationFactory;

  // 用于注册 Mapper 接口信息，建立 Mapper 接口的 Class 对象和 MapperProxyFactory 对象之间的关系，其中MapperProxyFactory对象用于创建Mapper动态代理对象
  protected final MapperRegistry mapperRegistry = new MapperRegistry(this);
  // 用于注册 MyBatis 插件信息，MyBatis插件实际上就是一个拦截器
  protected final InterceptorChain interceptorChain = new InterceptorChain();
  // 用于注册所有的TypeHandler，并建立 Jdbc 类型、Java 类型与 TypeHandler 之间的对应关系
  protected final TypeHandlerRegistry typeHandlerRegistry = new TypeHandlerRegistry(this);
  // 用于注册所有的类型别名
  protected final TypeAliasRegistry typeAliasRegistry = new TypeAliasRegistry();
  // 用于注册 LanguageDriver，LanguageDriver 用于解析 SQL 配置，将配置信息转换为 SqlSource 对象
  protected final LanguageDriverRegistry languageRegistry = new LanguageDriverRegistry();
  // 描述 <insert|select|update|delete> 等标签或者通过 @Select、@Delete、@Update、@Insert 等注解配置的 SQL 信息。MyBatis 将所有的MappedStatement 对象注册到该属性中，其中Key为 Mapper 的 Id，Value 为 MappedStatement 对象
  protected final Map<String, MappedStatement> mappedStatements = new StrictMap<MappedStatement>("Mapped Statements collection")
      .conflictMessageProducer((savedValue, targetValue) ->
          ". please check " + savedValue.getResource() + " and " + targetValue.getResource());
  // 用于注册 Mapper 中配置的所有缓存信息，其中 Key 为 Cache 的 Id，也就是 Mapper 的命名空间， Value 为 Cache 对象
  protected final Map<String, Cache> caches = new StrictMap<>("Caches collection");
  // 用于注册 Mapper 配置文件中通过 <resultMap> 标签配置的 ResultMap 信息，ResultMap 用于建立 Java 实体属性与数据库字段之间的映射关系，其中 Key 为ResultMap 的 Id ，该 Id 是由 Mapper 命名空间和 <resultMap> 标签的 id 属性组成的， value 为解析 <resultMap> 标签后得到的 ResultMap 对象
  protected final Map<String, ResultMap> resultMaps = new StrictMap<>("Result Maps collection");
  // 用于注册 Mapper 中通过 <parameterMap> 标签注册的参数映射信息。 Key 为 ParameterMap 的 Id ，由 Mapper 命名空间和 <parameterMap> 标签的 id 属性构成，Value 为解析 <parameterMap> 标签后得到的 ParameterMap 对象。
  protected final Map<String, ParameterMap> parameterMaps = new StrictMap<>("Parameter Maps collection");
  // 用于注册 KeyGenerator，KeyGenerator 是 MyBatis 的主键生成器，MyBatis 中提供了 3 种 KeyGenerator，即 Jdbc3KeyGenerator（数据库自增主键）、NoKeyGenerator（无自增主键）、SelectKeyGenerator（通过select语句查询自增主键，例如oracle的sequence）
  protected final Map<String, KeyGenerator> keyGenerators = new StrictMap<>("Key Generators collection");
  
  // 用于注册所有 Mapper XML 配置文件路径
  protected final Set<String> loadedResources = new HashSet<>();
  // 用于注册 Mapper 中通过 <sql> 标签配置的 SQL 片段， Key 为 SQL 片段的 Id，Value 为 MyBatis 封装的表示 XML 节点的 XNode 对象
  protected final Map<String, XNode> sqlFragments = new StrictMap<>("XML fragments parsed from previous mappers");
  // 用于注册解析出现异常的 XMLStatementBuilder 对象
  protected final Collection<XMLStatementBuilder> incompleteStatements = new LinkedList<>();
  // 用于注册解析出现异常的 CacheRefResolver 对象
  protected final Collection<CacheRefResolver> incompleteCacheRefs = new LinkedList<>();
  // 用于注册解析出现异常的 ResultMapResolver 对象
  protected final Collection<ResultMapResolver> incompleteResultMaps = new LinkedList<>();
  // 用于注册解析出现异常的 MethodResolver 对象
  protected final Collection<MethodResolver> incompleteMethods = new LinkedList<>();
  
    
  // Configuration 会作为 ParameterHandler、ResultSetHandler、StatementHandler、Executor 组件的工厂类
  // ParameterHandler 组件的工厂方法
  public ParameterHandler newParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql)
  // ResultSetHandler 组件的工厂方法
  public ResultSetHandler newResultSetHandler(Executor executor, MappedStatement mappedStatement, RowBounds rowBounds, ParameterHandler parameterHandler,
      ResultHandler resultHandler, BoundSql boundSql)
  // StatementHandler 组件的工厂方法
  public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) 
  // Executor 组件的工厂方法
  public Executor newExecutor(Transaction transaction, ExecutorType executorType) 
    
  ...
}
```



## Executor

​		真正执行`SQL`的是`Executor`组件。`Executor`接口中定义了对数据库的增删改查方法，其中`query()`和`queryCursor()`方法用于执行查询操作，`update()`方

法用于执行插入、删除、修改操作。

![](image/QQ截图20220603104550.png)

​		`SimpleExecutor`是基础的`Executor`，能够完成基本的增删改查操作。

​		`ResueExecutor`对`JDBC`中的`Statement`对象做了缓存，当执行相同的`SQL`语句时，直接从缓存中取出`Statement`对象进行复用，避免了频繁创建和销毁

`Statement`对象，从而提升系统性能，这是享元思想的应用。

​		`BatchExecutor`则会对调用同一个`Mapper`执行的`update、insert、delete`操作，调用`Statement`对象的批量操作功能。

​		另外，`MyBatis`支持一级缓存和二级缓存，当`MyBatis`开启了二级缓存功能时，会使用`CachingExecutor`对`SimpleExecutor、ResueExecutor、BatchExecutor`

进行装饰，为查询操作增加二级缓存功能，这是装饰器模式的应用。

​		`Executor`与数据库交互需要`Mapper`配置信息，`MyBatis`通过`MappedStatement`对象描述`Mapper`的配置信息，因此`Executor`需要一个`MappedStatement`对象

作为参数。`MyBatis`在应用启动时，会解析所有的`Mapper`配置信息，将`Mapper`配置解析成`MappedStatement`对象注册到`Configuration`组件中，可以调用

`Configuration`对象的`getMappedStatement()`方法获取对应的`MappedStatement`对象，获取`MappedStatement`对象后，根据`SQL`类型调用`Executor`对象的

`query()`或者`update()`方法即可。



## MappedStatement

​		`MappedStatement`描述`<select|update|insert|delete>`或者`@Select、@Update`等注解配置的`SQL`信息。

```java
public final class MappedStatement {

  private String resource; // Mapper 配置文件路径
  private Configuration configuration; // Configuration 对象的引用，方便获取 MyBatis 配置信息及 TypeHandler、TypeAlias 等信息
  private String id; // 在命名空间中唯一的标识符，可以被用来引用这条配置信息
  private Integer fetchSize; // 用于设置 JDBC 中 Statement 对象的 fetchSize 属性，该属性用于指定 SQL 执行后返回的最大行数
  private Integer timeout; // 驱动程序等待数据库返回请求结果的秒数，超时将会抛出异常
  private StatementType statementType; // 参数可选值为 STATEMENT、PREPARED、CALLABLE，这会让 MyBatis 分别使用 Statement、PreparedStatement、CallableStatement 与数据库交互，默认值为 PREPARED
  private ResultSetType resultSetType; // 参数可选值为 FORWARD_ONLY、SCROLL_SENSITIVE、SCROLL_INSENSITIVE，用于设置 ResultSet 对象的特征
  private SqlSource sqlSource; // 解析 <select|update|insert|delete> ，将 SQL 语句配置信息解析为 SqlSource 对象
  private Cache cache; // 二级缓存实例，根据 Mapper 中的 <cache> 标签配置信息创建对应的 Cache 实现
  private ParameterMap parameterMap; // 该属性已经废弃
  private List<ResultMap> resultMaps;
  private boolean flushCacheRequired;
  private boolean useCache; // 是否使用二级缓存。如果将其设置为 true，则会导致本条语句的结果被缓存在 MyBatis 的二级缓存中，对应 <select> 标签，该属性的默认值为 true
  private boolean resultOrdered; // 这个设置仅针对嵌套结果 select 语句适用，如果为 true，就是假定嵌套结果包含在一起或分组在一起，这样的话，当返回一个主结果行的时候，就不会发生对前面结果集引用的情况。这就使得在获取嵌套结果集的时候不至于导致内存不够用，默认值为 false
  private SqlCommandType sqlCommandType;
  private KeyGenerator keyGenerator; // 主键生成策略，默认为 Jdbc3KeyGenerator，即数据库自增主键。当配置了<selectKey> 时，使用 SelectKeyGenerator 生成主键
  private String[] keyProperties;
  private String[] keyColumns;
  private boolean hasNestedResultMaps; // <select> 标签中通过 resultMap 属性指定 ResultMap 是不是嵌套的 ResultMap。
  private String databaseId; // 如果配置了 databaseIdProvider，MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句
  private Log statementLog; // 用于输出日志
  private LanguageDriver lang; // 该属性用于指定 LanguageDriver 实现，MyBatis 中的 LanguageDriver 用于解析 <select|update|insert|delete> 标签中的 SQL 语句，生成 SqlSource 对象
  private String[] resultSets; // 这个设置仅对多结果集的情况适用，它将列出语句执行后返回的结果集并每个结果集给一个名称，名称使用逗号分隔
    
  ...
}
```



## StatementHandler

​		`StatementHandler`组件封装了对`JDBC Statement`的操作：

```java
public interface StatementHandler {
  // 该方法用于创建 JDBC Statement 对象，并完成S tatement 对象的属性设置
  Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException;
  // 该方法使用 MyBatis 中的 ParameterHandler 组件为 PreparedStatement 和 CallableStatement 参数占位符设置值
  void parameterize(Statement statement) throws SQLException;
  // 将 SQL 命令添加到批处量执行列表中
  void batch(Statement statement) throws SQLException;
  // 调用 Statement 对象的 execute() 方法执行更新语句
  int update(Statement statement) throws SQLException;
  // 执行查询语句，并使用 ResultSetHandler 处理查询结果集
  <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException;
  // 带游标的查询，返回 Cursor 对象，能够通过 Iterator 动态地从数据库中加载数据，适用于查询数据量较大的情况，避免将所有数据加载到内存中
  <E> Cursor<E> queryCursor(Statement statement) throws SQLException;
  // 获取 Mapper 中配置的 SQL 信息，BoundSql 封装了动态 SQL 解析后的 SQL 文本和参数映射信息
  BoundSql getBoundSql();
  // 获取 ParameterHandler 实例
  ParameterHandler getParameterHandler();

}
```

![](image/QQ截图20220603111458.png)

​		`SimpleStatementHandler`继承至`BaseStatementHandler`，封装了对`JDBC Statement`对象的操作。

​		`PreparedStatementHandler`封装了对`JDBC PreparedStatement`对象的操作。

​		`CallableStatementHandler`则封装了对`JDBC CallableStatement`对象的操作。

​		`RoutingStatementHandler`会根据`Mapper`配置中的`statementType`属性`(`取值为`STATEMENT、PREPARED、CALLABLE)`创建对应的`StatementHandler`实现。



## TypeHandler

​		使用`JDBC API`开发应用程序，涉及`Java`类型和`JDBC`类型转换的两种情况如下：

​				1、`PreparedStatement`对象为参数占位符设置值时，需要调用`PreparedStatement`接口中提供的一系列的`setXXX()`方法，将`Java`类型转换为对应的

​		`JDBC`类型并为参数占位符赋值。

​				2、执行`SQL`语句获取`ResultSet`对象后，需要调用`ResultSet`对象的`getXXX()`方法获取字段值，此时会将`JDBC`类型转换为`Java`类型。

​		`MyBatis`中使用`TypeHandler`解决上面两种情况下，`JDBC`类型与`Java`类型之间的转换：

```java
public interface TypeHandler<T> {
  // 为 PreparedStatement 对象设置参数
  void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException;
  // 根据列名称获取该列值
  T getResult(ResultSet rs, String columnName) throws SQLException;
  // 根据列索引获取该列值
  T getResult(ResultSet rs, int columnIndex) throws SQLException;
  // 获取存储过程调用结果
  T getResult(CallableStatement cs, int columnIndex) throws SQLException;
}
```

​		`MyBatis`通过`TypeHandlerRegistry`建立`JDBC`类型、`Java`类型与`TypeHandler`之间的映射关系：

```java
public final class TypeHandlerRegistry {

  // JDBC 类型 <=> TypeHandler
  private final Map<JdbcType, TypeHandler<?>> jdbcTypeHandlerMap = new EnumMap<>(JdbcType.class);
  // Java 类型 <=> JDBC 类型 <=> TypeHandler
  private final Map<Type, Map<JdbcType, TypeHandler<?>>> typeHandlerMap = new ConcurrentHashMap<>();
  private final TypeHandler<Object> unknownTypeHandler;
  // TypeHandler Class 对象 <=> TypeHandler
  private final Map<Class<?>, TypeHandler<?>> allTypeHandlersMap = new HashMap<>();

  private static final Map<JdbcType, TypeHandler<?>> NULL_TYPE_HANDLER_MAP = Collections.emptyMap();

  private Class<? extends TypeHandler> defaultEnumTypeHandler = EnumTypeHandler.class;

  /**
   * The default constructor.
   */
  public TypeHandlerRegistry() {
    this(new Configuration());
  }

  /**
   * The constructor that pass the MyBatis configuration.
   *
   * @param configuration a MyBatis configuration
   * @since 3.5.4
   */
  public TypeHandlerRegistry(Configuration configuration) {
    this.unknownTypeHandler = new UnknownTypeHandler(configuration);

    register(Boolean.class, new BooleanTypeHandler());
    register(boolean.class, new BooleanTypeHandler());
    register(JdbcType.BOOLEAN, new BooleanTypeHandler());
    register(JdbcType.BIT, new BooleanTypeHandler());

    ...
  }
  ...
}
```



## ParameterHandler

​		当使用`PreparedStatement`或者`CallableStatement`对象时，如果`SQL`语句中有参数占位符，在执行`SQL`语句之前，就需要为参数占位符设置值。

`ParameterHandler`的作用是在`PreparedStatementHandler`和`CallableStatementHandler`操作对应的`Statement`执行数据库交互之前为参数占位符设置值。

```java
public interface ParameterHandler {
  // 该方法用于获取执行 Mapper 时传入的参数对象
  Object getParameterObject();
  // 该方法用于为 JDBC PreparedStatement 或者 CallableStatement 对象设置参数值
  void setParameters(PreparedStatement ps) throws SQLException;
}
```

```java
public class DefaultParameterHandler implements ParameterHandler {
   
  ...

  @Override
  public void setParameters(PreparedStatement ps) {
    ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    if (parameterMappings != null) {
      // 获取所有参数的映射信息
      for (int i = 0; i < parameterMappings.size(); i++) {
        ParameterMapping parameterMapping = parameterMappings.get(i);
        if (parameterMapping.getMode() != ParameterMode.OUT) {
          Object value;
          // 参数属性名称
          String propertyName = parameterMapping.getProperty();
          // 根据参数属性名获取参数值
          if (boundSql.hasAdditionalParameter(propertyName)) { // issue #448 ask first for additional params
            value = boundSql.getAdditionalParameter(propertyName);
          } else if (parameterObject == null) {
            value = null;
          } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
            value = parameterObject;
          } else {
            MetaObject metaObject = configuration.newMetaObject(parameterObject);
            value = metaObject.getValue(propertyName);
          }
          // 获取参数对应的 TypeHandler
          TypeHandler typeHandler = parameterMapping.getTypeHandler();
          JdbcType jdbcType = parameterMapping.getJdbcType();
          if (value == null && jdbcType == null) {
            jdbcType = configuration.getJdbcTypeForNull();
          }
          try {
            // 为参数占位符设置值
            typeHandler.setParameter(ps, i + 1, value, jdbcType);
          } catch (TypeException | SQLException e) {
            throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
          }
        }
      }
    }
  }
}
```



## ResultSetHandler

​		`ResultSetHandler`用于在`StatementHandler`对象执行完查询操作或存储过程后，对结果集或存储过程的执行结果进行处理。

```java
public interface ResultSetHandler {
  // 获取 Statement 对象中的 ResultSet 对象，对 ResultSet 对象进行处理，返回包含结果实体的 List 对象
  <E> List<E> handleResultSets(Statement stmt) throws SQLException;
  // 将 ResultSet 对象包装成 Cursor 对象，对 Cursor 进行遍历时，能够动态地从数据库查询数据，避免一次性将所有数据加载到内存中
  <E> Cursor<E> handleCursorResultSets(Statement stmt) throws SQLException;
  // 处理存储过程调用结果
  void handleOutputParameters(CallableStatement cs) throws SQLException;
}
```

```java
public class DefaultResultSetHandler implements ResultSetHandler {

  ...  
    
  @Override
  public List<Object> handleResultSets(Statement stmt) throws SQLException {
    ErrorContext.instance().activity("handling results").object(mappedStatement.getId());

    final List<Object> multipleResults = new ArrayList<>();

    int resultSetCount = 0;
    // 获取 ResultSet 对象，并将 ResultSet 包装为 ResultSetWrapper
    ResultSetWrapper rsw = getFirstResultSet(stmt);
	// 获取 ResultMap 信息，一般只有一个 ResultMap
    List<ResultMap> resultMaps = mappedStatement.getResultMaps();
    int resultMapCount = resultMaps.size();
    validateResultMapsCount(rsw, resultMapCount);
    while (rsw != null && resultMapCount > resultSetCount) {
      ResultMap resultMap = resultMaps.get(resultSetCount);
      // 处理结果集
      handleResultSet(rsw, resultMap, multipleResults, null);
      rsw = getNextResultSet(stmt);
      cleanUpAfterHandlingResultSet();
      resultSetCount++;
    }
	// 处理 select 标签的 resultSets 属性
    String[] resultSets = mappedStatement.getResultSets();
    if (resultSets != null) {
      while (rsw != null && resultSetCount < resultSets.length) {
        ResultMapping parentMapping = nextResultMaps.get(resultSets[resultSetCount]);
        if (parentMapping != null) {
          String nestedResultMapId = parentMapping.getNestedResultMapId();
          ResultMap resultMap = configuration.getResultMap(nestedResultMapId);
          handleResultSet(rsw, resultMap, null, parentMapping);
        }
        rsw = getNextResultSet(stmt);
        cleanUpAfterHandlingResultSet();
        resultSetCount++;
      }
    }
	// 对 multipleResults 进行处理，如果只有一个结果集，返回结果集中的元素，否则返回多个结果集
    return collapseSingleResultList(multipleResults);
  }
}
```



