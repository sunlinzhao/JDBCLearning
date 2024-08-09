# JDBCLearning

数据持久化处理：

1. 存储磁盘文件，使用IO流；
2. 存储在网络上；
3. 存储在数据库中；(常用)

### 0. 简介

Java DataBase Connectivity：JDBC，称为Java数据库连接。它是一种用于数据库访问的应用程序API,由一组用Java编写的类和接口组成。是Java语言中用来**规范**客户端程序如何访问数据库的应用程序接口。

> 使用JDBC技术，可以用统一的语法对多种数据库进行访问，而不用担心数据库操作语言的差异。❤️

![image.png](assets/image0.png)

#### （1）JDBC 体系结构

JDBC的结构可以划分为两层：

- 面向底层的 JDBC Driver Interface((驱动程序管理器接口) （各个数据库厂商提供JAVA的，符合JDBC规范的驱动）
- 面向程序员的 JDBC API 👍

> JDBC API 使用动动程序管理器并指定数据库的驱动程序来提供与异构数据库的透明连接。
>
> JDBC 驱动管理器确保使用正确的驱动程序来访问每个数据源。驱动程序管理器能够支持连接到多个异构数据库的多个并发驱动程序。

![image.png](assets/image1.png)

#### （2）相关 API

- Driver: 数据库厂商提供数据库的驱动；
- DriverManager: 驱动管理器，
  - 作用：注册驱动，让程序加载一下厂商提供的驱动；
  - 获取连接：有了连接之后，就可以操作数据库；
- Connection：连接，表示和数据库的连接对象，获取 Statement 对象，语句对象 CUDR；
- Statement：用来操纵SQL语句，发送给数据库去执行；
  - 执行增加、删除、修改，对数据库记录有影响的语句；
  - 执行查询，返回 ResultSet；
- ResultSet：结果集，用来封装查询结果；

#### （3）获取连接操作

1. 获取驱动：[mysqlconnectorjava8030.jar](assets/mysql-connector-java-8.0.30.jar)
   https://repo1.maven.org/maven2/

> https://repo1.maven.org/maven2/mysql/mysql-connector-java/5.1.48/

2. 导入库

![image.png](assets/image2.png)

> com.mysql.jdbc.Driver 是 mysql-connector-java 5中的;
>
> com.mysql.cj.jdbc.Driver 是mysql-connector-java 6 以及以上中的;

3. 获取连接

![image.png](assets/image3.png)

> com.mysql.cj.jdbc.Driver 实现了 java.sql.Driver 接口，后者是 JDBC 的规范，所以创建驱动对象的时候用 java.sql.Driver，便于更换与维护
>
> URL: `协议:数据库://地址:端口号/数据库名`, 例如：`jdbc:mysql://localhost:3306/JDBC`

##### a. 方式一：直接通过 Driver 接口的 connect 方法去获取连接

```java
    // 获取连接方式一
    public static void GetConnection1() throws SQLException {
        // com.mysql.cj.jdbc.Driver 实现了 java.sql.Driver 接口，后者是 JDBC 的规范，所以创建驱动对象的时候用 java.sql.Driver，便于更换与维护
        Driver driver = new com.mysql.cj.jdbc.Driver();
        Properties properties = new Properties();
        properties.setProperty("user", "root");
        properties.setProperty("password", "root");
        // 通过 java 连接数据库 协议
        // 协议:数据库://地址:端口号/数据库名
        String url = "jdbc:mysql://localhost:3306/JDBC";
        Connection connect = driver.connect(url, properties);
        System.out.println(connect);
    }
```

##### b. 获取连接方式二：通过反射方式静态加载驱动类

```java
    public static void GetConnection2() throws Exception {
        // 通过反射获取驱动的构造方法创建实例
        // 既如此，我们便可以在需要更换数据源时，直接更换传入的字符串（代表驱动类），也可以将该字符串（代表驱动类）写入到配置文件中，便于管理和维护
        Object o = Class.forName("com.mysql.cj.jdbc.Driver").getConstructor().newInstance();
        // 强转为 java.sql.Driver
        Driver driver = (Driver) o;
        Properties properties = new Properties();
        properties.setProperty("user", "root");
        properties.setProperty("password", "root");
        String url = "jdbc:mysql://localhost:3306/JDBC";
        Connection connect = driver.connect(url, properties);
        System.out.println(connect);
    }
```

##### c. 获取连接方式三：通过 DriverManager 获取连接

```java
    public static void getConnection3() throws Exception {
        Object o = Class.forName("com.mysql.cj.jdbc.Driver").getConstructor().newInstance();
        // 强转为 java.sql.Driver
        Driver driver = (Driver) o;
        DriverManager.registerDriver(driver);
        String url = "jdbc:mysql://localhost:3306/JDBC";
        Connection connection = DriverManager.getConnection(url, "root", "root");
        System.out.println(connection);
    }
```

##### d. 获取连接方式四：通过 DriverManager 获取连接 （静态代码块注册）

![image.png](assets/image4.png)

> com.mysql.cj.jdbc.Driver 有一个静态代码块，在类加载时会被执行，注册到驱动管理，因此不需要再次注册 ❤️

```java
    public static void getConnection4() throws Exception {
        // 提供URL地址
        String url = "jdbc:mysql://localhost:3306/JDBC";
        // 加载驱动
        // com.mysql.cj.jdbc.Driver 有一个静态代码块，在类加载时会被执行，注册到驱动管理，因此不需要再次注册
        Class.forName("com.mysql.cj.jdbc.Driver");
        // 获取连接对象
        Connection connection = DriverManager.getConnection(url, "root", "root");
        System.out.println(connection);
    }
```

e. 获取连接方式五：读取配置文件 ❤️

> 标准获取连接的方式 ❤️

```properties
# mysql config
mysql.user=root
mysql.password=root
mysql.driver=com.mysql.cj.jdbc.Driver
mysql.url=jdbc:mysql://localhost/JDBC
```

```java
    public static void getConnection5() throws Exception{
        // 读取配置文件
        ResourceBundle resourceBundle = ResourceBundle.getBundle("db");
        String user = resourceBundle.getString("mysql.user");
        String password = resourceBundle.getString("mysql.password");
        String url = resourceBundle.getString("mysql.url");
        String driver = resourceBundle.getString("mysql.driver");
        // 加载驱动
        Class.forName(driver);
        // 获取连接
        Connection connection = DriverManager.getConnection(url, user, password);
        System.out.println(connection);
    }
```

#### （4）常见连接错误

1. mysql 服务没有启动

> com.mysql.cj,jdbc.exceptions.CommunicationsException: Communications link failure

2. 驱动类名错误

> java.lang.ClassNotFoundException: com.mysql.cj.jdbc.driver

3. 驱动包没有导入

> java.lang.ClassNotFoundException: com.mysql.cj.jdbc.Driver

4. 数据库名字错误：

> java.sql.SQLSyntaxErrorException: Unknown database 'testtest'

5. URL地址错误

> java.sqlSQLException: No suitable driver found for jdbc:oracle://localhost:3306/test
>
> The driver has not received any packets from the server.

6. 用户名或密码错误

> java.sql.SQLException: Access denied for user 'root'@'localhost'
>
> Access denied for user 'root1'@'ocalhost' (using password: YES)

### 1. Statement 语句对象

#### （1）获取 Statement 实现数据插入

> java.sql.Statement

> Statement statement = connection.createStatement();
>
> int i = statement.executeUpdate(sql); // 返回整数表示，影响了几行数据

#### （2）或取 Statement 实现数据查询

##### a. Resultset 结果集

> - 用来封装查询的结果。
> - 最开始，结果集的指针指向第一条数据之前(库顶)的位置。
> - 通过调用next方法来判断是否有下一条数据，如果有下一条数据，则指针向下移动一位。
> - 整体上对于结果集，可以使用循环遍历。

```java
public class TestStatement {
    public static void main(String[] args) throws Exception {
//        insert();
        query();
    }

    public static Connection connect() throws Exception {
        // 读取配置文件
        ResourceBundle bundle = ResourceBundle.getBundle("db");
        String user = bundle.getString("mysql.user");
        String password = bundle.getString("mysql.password");
        String url = bundle.getString("mysql.url");
        String driver = bundle.getString("mysql.driver");
        // 加载驱动
        Class.forName(driver);
        // 获取连接
        Connection connection = DriverManager.getConnection(url, user, password);
        return connection;
    }
    public static void insert() throws Exception {
        // 获取连接
        Connection connect = connect();
        // 获取语句对象
        Statement statement = connect.createStatement();
        // 从键盘输入数据
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入员工编号:");
        String code = sc.next();
        System.out.println("请输入员工姓名:");
        String name = sc.next();
        System.out.println("请输入员工年龄:");
        String age = sc.next();
        // 拼接 SQL 语句
        String sql = "INSERT INTO employees (code, name, age) VALUES ('" + code + "', '" + name + "', " + age + ")";
        // executeUpdate() 主要执行DML语句（DDL语句也可以执行） execute 都可以执行
        // 插入操作
        int i = statement.executeUpdate(sql); // 返回整数表示，影响了几行数据
        if(i>0){
            System.out.println("Successfully! " + i + "row(s) affected");
        } else {
            System.out.println("failed!");
        }
        // 关闭资源
        sc.close();
        statement.close();
        connect.close();
    }

    public static void query() throws Exception {
        // 获取连接
        Connection connect = connect();
        // 获取语句对象
        Statement statement = connect.createStatement();
        // 查询操作
        String selectSql = "select * from employees";
        // executeQuery() 主要执行DQL数据查询语言
        ResultSet resultSet = statement.executeQuery(selectSql); // 返回结果集
        // 打印表头
        System.out.println("id" + "\t" + "code" + "\t" + "name" + "\t" + "age" + "\t" + "gender" + "\t" + "salary" + "\t" + "dept_id");
        // 打印数据
        while (resultSet.next()){
            System.out.print(resultSet.getInt(1) + "\t"); // 列索位置引从 1 开始
            System.out.print(resultSet.getString(2) + "\t");
            System.out.print(resultSet.getString(3) + "\t");
            System.out.print(resultSet.getInt(4) + "\t");
            System.out.print(resultSet.getInt(5) + "\t");
            System.out.print(resultSet.getDouble(6) + "\t");
            System.out.print(resultSet.getInt(7) + "\t");
            System.out.println("\n");
        }
        // 关闭资源
        resultSet.close();
        statement.close();
        connect.close();
    }
}
```

> 操作 JDBC 的步骤：❤️
>
> 1. 提供连接数据库的 URL 地址
> 2. Class.forName 加载驱动；
> 3. 获得连接对象 DriverManager.getconnection()；
> 4. 获取语句对象 Statement；
> 5. 操纵sql语句，执行 CRUD；
> 6. 操纵结果集对象 Resultset；
> 7. 关闭资源；
> 8. 处理异常 SQLException；



### 2. SQL 注入攻击 ❤️ 

#### （1）模拟登陆业务逻辑

![image.png](assets/image5.png)

![image.png](assets/image6.png)

> SQL 注入攻击的产生，是由于使用 SQL 语句拼接的方式进行查询，其中的参数位置通过**字符串拼接**的方式形成 SQL 语句，可能导致特殊的语法使得命中查询

原因：SQL 语句拼接

#### （2）解决 SQL 攻击：PreparedStatement 预编译的方式获得 SQL 语句 ❤️ 

> - PreparedStatement 接口继承自 Statement 接口，表示预编译的 SQL 语句对象
> - `PreparedStatement preparedStatement = connection.prepareStatement(sql_model)` 允许接收一个带有参数缺省位（?）的SQL字符串
>   > String sql_model = "insert into employees (code, name, age) values (?, ?, ?)";
>   >
> - `preparedStatement.setString(1, code);` 的方法传递参数

> - 预编译带有缺省位（?）的 SQL 字符串，？位置的参数时固定的，不会产生因拼接而产生的歧义；
> - SQL 模板编译后可以多次使用，比如一次插入多条数据，只要参数位置对得上；

```java
public class TestPreparedStatement {
    public static void main(String[] args) {
        Connection connection = DBUtil.getConnection();
        String sql_model = "insert into employees (code, name, age) values (?, ?, ?)";
        try {
            // 预编译的方式获得 SQL 语句对象，需要先获得 SQL语句 模板
            PreparedStatement preparedStatement = connection.prepareStatement(sql_model);
            // 从键盘获取数据
            Scanner sc = new Scanner(System.in);
            System.out.println("输入编号：");
            String code = sc.next();
            System.out.println("输入姓名：");
            String name = sc.next();
            System.out.println("输入年龄：");
            String age = sc.next();
            // 填入数据; 参数索引位置从1开始
            preparedStatement.setString(1, code);
            preparedStatement.setString(2, name);
            preparedStatement.setString(3, age);
            int i = preparedStatement.executeUpdate();
            if(i>0){
                System.out.println("插入成功！");
            } else {
                System.out.println("插入失败！");
            }
            // 一个模板，多次插入
            preparedStatement.setString(1, "1030");
            preparedStatement.setString(2, "李火旺");
            preparedStatement.setString(3, "23");
            i = preparedStatement.executeUpdate();
            if(i>0){
                System.out.println("插入成功！");
            } else {
                System.out.println("插入失败！");
            }
            // 关闭资源
            preparedStatement.close();
            connection.close();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }
}

```

#### 3. UUID的使用

UUID：Universally Unique Identifier，通用唯一识别码；是一种在分布式系统中生成唯一标识符的方法。UUID 通常用于确保不同系统之间生成的标识符是唯一的，即使这些系统之间没有直接通信。

> UUID 的特点：
>
> - 唯一性：每个 UUID 都是唯一的，几乎不可能出现重复;
> - 无序性：UUID 之间没有固定的顺序关系;
> - 跨平台：UUID 在不同的操作系统和编程语言中都可以生成和识别;
> - 长度固定：标准的 UUID 由 32 个十六进制数字组成，通常分为五个部分显示，格式为 8-4-4-4-12 的 16 进制数字组合;

数据库设计中，主键的选择：（每个数据项都有一个主键字段）

> 1. 自然主键：跟业务逻辑有关，如学号；
> 2. 代理主键：与业务逻辑无关
>    1. 自动增长；（不安全）
>    2. 随机唯一；（重复概率极小）

```java
public class TestUUID {
    public static void main(String[] args) {
//        System.out.println(UUID.randomUUID().toString()); // 7ddfef2b-b213-4a97-a7ce-0313457ec5f3
        String sql = "insert into uuid values(?,?)";
        for (int i = 0; i < 100; i++) {
            DBUtil.executePreparedUpdate(sql, UUID.randomUUID().toString().replace("-",""), i);
        }
    }
}
```



### 4. MySQL 数据类型与 Java 数据类型的对应关系

![image.png](assets/image7.png)

> getstring()方法的通用性: 😕(用于读取数据)
>
> getstring()//通用方法,可以获取所有数据类型。适用于显示，所有的数据通过这种方法获取，都变成字符了，如果想要获取后再做其他计算，不能使用该通用方法



222





222222
