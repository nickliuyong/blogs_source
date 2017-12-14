---
title: jdbc的加载过程
date: 2016-10-17 10:06:26
tags: 
     - jdbc  
     - 面试 
---

### jdbc连接mysql数据库

``` java
/* 获取数据库连接的函数*/
public static Connection getConnection() {
    Connection con = null;	//创建用于连接数据库的Connection对象
    try {
        Class.forName("com.mysql.jdbc.Driver");// 加载Mysql数据驱动
        
        con = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/myuser", "root", "root");// 创建数据连接
        
    } catch (Exception e) {
        System.out.println("数据库连接失败" + e.getMessage());
    }
    return con;	//返回所建立的数据库连接
}
```
<!-- more -->

### 源码分析
``` java
Class.forName("com.mysql.jdbc.Driver"); // 通过类加载器去实例化Driver
//Driver部分源码如下
private final static CopyOnWriteArrayList<DriverInfo> registeredDrivers = new CopyOnWriteArrayList<DriverInfo>();
static {
        try {
            DriverManager.registerDriver(new Driver());
        } catch (SQLException var1) {
            throw new RuntimeException("Can\'t register driver!");
        }
}
public static synchronized void registerDriver(java.sql.Driver driver)
        throws SQLException {
        /* Register the driver if it has not already been added to our list */
        if(driver != null) {
            registeredDrivers.addIfAbsent(new DriverInfo(driver));
        } else {
            // This is for compatibility with the original DriverManager
            throw new NullPointerException();
        }
        println("registerDriver: " + driver);

}
//从上可知，加载Driver的过程，实际上就是往registeredDrivers注册了一个驱动。
con = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/myuser", "root", "root");// 创建数据连接，其实就是去registeredDrivers找对应的驱动的过程
//部分源码如下：
private static Connection getConnection(
    //TODO 省略
    for(DriverInfo aDriver : registeredDrivers) {
                // If the caller does not have permission to load the driver then
                // skip it.
                if(isDriverAllowed(aDriver.driver, callerCL)) {
                    try {
                        println("    trying " + aDriver.driver.getClass().getName());
                        //主要是通过url字符串匹配来获取相应的connect，具体实现可参考下方源码
                        Connection con = aDriver.driver.connect(url, info);
                        if (con != null) {
                            // Success!
                            println("getConnection returning " + aDriver.driver.getClass().getName());
                            return (con);
                        }
                    } catch (SQLException ex) {
                        if (reason == null) {
                            reason = ex;
                        }
                    }
                } else {
                    println("    skipping: " + aDriver.getClass().getName());
                }
            }
｝
//以下是postgresql的 connect 源码
public Connection connect(String url, Properties info) throws SQLException {
        if(!url.startsWith("jdbc:postgresql:")) {
            return null;
        } else {
            Properties defaults;
           //TODO 省略
        ｝
}
```
### 总结
> 一次面试带来的技术积累  
  一次面试带来的反思         