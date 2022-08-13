# JDBC

JDBC（Java DataBaseConnectivity）就是Java连接数据库的一种技术。

![image-20220725214542527](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220725214542527.png)

## JDBC组成

- DriverManager类：管理驱动程序，通过驱动连接数据库。

- Connection接口：描述Java程序与数据库的连接。

- Statement接口：描述执行器，可以通过Connection对象向数据库发送sql语句。

- ResultSet接口：获取数据库Sql语句执行的结果。

![image-20220725214637395](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220725214637395.png)

## JDBC步骤

1） 加载驱动程序

2） 创建连接对象

3） 创建执行对象

4） 执行SQL语句

5） 处理返回结果

6） 关闭访问资源

```java
public static int execute(String sql){
    Connection connection = null;
    Statement statement = null;
    try {
        // 1.加载驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
        // 2.创建连接对象
        String url = "jdbc:mysql://localhost:3307/staff?serverTimezone=GMT%2B8&useUnicode=true&Character=utf-8";
        String username = "root";
        String password = "xiaoping";
        connection = DriverManager.getConnection(url, username, password);
        // 3.创建执行对象
        statement = connection.createStatement();
        // 4.执行sql
        int result = statement.executeUpdate(sql);
        // 5.处理结果，并返回
        return result;

    } catch (ClassNotFoundException | SQLException e) {
        e.printStackTrace();
    } finally {
        // 6.关闭连接资源
        try {
            if (statement!=null)
                statement.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        try {
            if (connection!=null)
                connection.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }
    return 0;
}
```

## PreparedStatement接口

**Statement接口在解决动态参数问题时，采用的是字符串拼接**，容易产生sql注入

PerparedStatement接口，是Statement的子接口

**PreparedStatement采用参数绑定的方式，代替字符串的拼接，在进行参数绑定时，会根据参数的类型决定如何处理参数；**

**优点***

1. 避免sql注入
2. 对sql进行预处理，从而提高sql语句的执行速度
3. 可以进行批处理

案例：

```java
String sql = "delete from staff where id = ?"
preparedStatement = connection.prepareStatement(sql);
if (params != null) {
	// 进行参数绑定
    for (int i = 0; i < params.length; i++) {
    	//第一个参数代表绑定的位置（即第几个?，从1开始） 第二个参数代表绑定的值
    	preparedStatement.setObject(i + 1, params[i]);
    }
}
```



## ResultSet接口

在JDBC操作中，数据库所有查询记录将使用ResultSet进行接收，并使用ResultSet显示内容。

常用方法：

```java
boolean next();		//将指针移到下一行
getXxx("columnName");	//返回列名对应的列内容
getXxx(index);		//index代表列的下标，从1开始
```

```java
ResultSet resultSet = preparedStatement.executeQuery();
while (resultSet.next()) {
    Dept dept = new Dept();
    dept.setId(resultSet.getInt("id"));
    dept.setName(resultSet.getString("name"));
    dept.setRoom(resultSet.getString("room"));
    dept.setPhone(resultSet.getString("phone"));
    deptList.add(dept);
}
```

### ResultSetMetaDate 元数据

**ResultSetMetaData 叫元数据**，是数据库 列对象，**以列为单位封装为对象**。

元数据，指的是其包含列名，列值，列类型，列长度等等有用信息。

常用方法：

```java
metaData.getColumnName(i);        //获取该列的原始名字,i是列的下标，从1开始
metaData.getColumnLabel(i);		  //获取该列的别名,i是列的下标，从1开始
metaData.getColumnCount();		  //获取列的数量
```

示例：把查询结果放入Map中

```java
//存放所有查询结果的List
List<Map<String, Object>> list = new ArrayList<>();
ResultSet resultSet = preparedStatement.executeQuery();
ResultSetMetaData metaData = resultSet.getMetaData();
while (resultSet.next()) {
    Map<String, Object> row = new HashMap<>();

    //获取该条记录的每列数据 并存入Map中
    int columnCount = metaData.getColumnCount();
        for (int i = 1; i <= columnCount; i++) {
        String columnName = metaData.getColumnLabel(i);
        row.put(columnName, resultSet.getObject(i));
	}

	list.add(row);
}
```

## JDBC封装

```java
public class DBUtil {

    private static final String URL = "jdbc:mysql://localhost:3307/staffmgr?serverTimezone=GMT%2b8&useUnicode=true" +
            "&Character=utf-8";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "xiaoping";

    /**
     * 获取连接
     *
     * @return
     */
    private Connection getConnection() throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.cj.jdbc.Driver");
        return DriverManager.getConnection(URL, USERNAME, PASSWORD);
    }

    /**
     * 关闭资源
     *
     * @param connection
     * @param stmt
     * @param resultSet
     */
    private void close(Connection connection, Statement stmt, ResultSet resultSet) {
        try {
            if (resultSet != null) {
                resultSet.close();
            }
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        try {
            if (stmt != null) {
                stmt.close();
            }
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        try {
            if (connection != null) {
                connection.close();
            }
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }

    /**
     * 通用CUD
     *
     * @param sql
     * @param params
     * @return
     */
    public int update(String sql, Object... params) {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        try {
            connection = getConnection();
            preparedStatement = connection.prepareStatement(sql);
            if (params != null) {
                for (int i = 0; i < params.length; i++) {
                    preparedStatement.setObject(i + 1, params[i]);
                }
            }

            int result = preparedStatement.executeUpdate();
            return result;
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            close(connection, preparedStatement, null);
        }
        return 0;
    }

    /**
     * 通用查询 （返回map）
     * @param sql
     * @param params
     * @return
     */
    public List<Map<String, Object>> select(String sql, Object... params) {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        List<Map<String, Object>> list = new ArrayList<>();
        try {
            connection = getConnection();
            preparedStatement = connection.prepareStatement(sql);
            if (params != null) {
                for (int i = 0; i < params.length; i++) {
                    preparedStatement.setObject(i + 1, params[i]);
                }
            }
            resultSet = preparedStatement.executeQuery();
            ResultSetMetaData metaData = resultSet.getMetaData();
//            metaData.getColumnCount();//获取列的数量
//            metaData.getColumnName(1); //根据列的下标，获取列名
//            metaData.getColumnLabel(1);//根据列的下标，获取别名
            while (resultSet.next()) {
                Map<String, Object> row = new HashMap<>();

                //获取数据
                int columnCount = metaData.getColumnCount();
                for (int i = 1; i <= columnCount; i++) {
                    String columnName = metaData.getColumnLabel(i);
                    row.put(columnName, resultSet.getObject(i));
                }

                list.add(row);
            }
            return list;
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            close(connection, preparedStatement, resultSet);
        }
        return list;
    }

    /**
     * 通用查询 （返回指定对象）
     * @param cls
     * @param sql
     * @param params
     * @param <T>
     * @return
     */
    public <T> List<T> select(Class<T> cls, String sql, Object... params) {
        //调用select获取全是Map的List
        List<Map<String, Object>> list = select(sql, params);
        List<T> res = new ArrayList<>();
        if (list != null) {
            for (Map<String, Object> map : list) {
                T t = mapToObj(map, cls);
                res.add(t);
            }
        }
        return res;
    }

    /**
     * Map转到Object
     * @param map
     * @param cls
     * @param <T>
     * @return
     */
    private <T> T mapToObj(Map<String, Object> map, Class<T> cls) {
        // 反射+内省，内省是对反射的封装，基于JavaBean
        try {
            // 反射创建对象
            T t = cls.newInstance();
            // 获取JavaBean信息
            BeanInfo beanInfo = Introspector.getBeanInfo(cls);
            // 获取cls类的属性信息
            PropertyDescriptor[] propertyDescriptors = beanInfo.getPropertyDescriptors();
            // 遍历所有属性
            for (PropertyDescriptor prop : propertyDescriptors) {
                //属性的名字
                String propName = prop.getName();
                Method setters = prop.getWriteMethod();//set方法
//                prop.getReadMethod();   //get方法
                // 调用set方法，将map中的值设置到属性上
                Object value = map.get(propName);
                if (value != null && setters != null) {
                    try {
                        setters.invoke(t, value);
                    } catch (InvocationTargetException e) {
                        e.printStackTrace();
                    }
                }
            }
            return t;
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (IntrospectionException e) {
            e.printStackTrace();
        }

        return null;
    }

    /**
     * 查询一条 返回map
     * @param sql
     * @param params
     * @return
     */
    public Map<String, Object> selectOne(String sql, Object... params) {
        List<Map<String, Object>> list = select(sql, params);
        if (list != null && list.size() > 0) {
            return list.get(0);
        }
        return null;
    }

    /**
     * 查询一条 返回对应对象
     * @param cls
     * @param sql
     * @param params
     * @param <T>
     * @return
     */
    public <T> T selectOne(Class<T> cls, String sql, Object... params){
        Map<String, Object> map = selectOne(sql, params);
        if (map!=null){
            return mapToObj(map, cls);
        }
        return null;
    }

}

```

