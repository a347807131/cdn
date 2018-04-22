#数据库连接池

标签： JDBC DBCP

---

##概念

- 介绍
> DBCP(DataBase connection pool)数据库连接池是 apache 上的一个Java连接池项目。DBCP通过连接池预先同数据库建立一些连接放在内存中(即连接池中)，应用程序需要建立数据库连接时直接到从接池中申请一个连接使用，用完后由连接池回收该连接，从而达到连接复用，减少资源消耗的目的。

- 图解
![图解](https://raw.githubusercontent.com/a347807131/ms/master/dbcp/dbcp.png)

##简单连接池实现
```java
//模拟连接池
public class SimpleConnectionPool {
	//存放链接对象的池
	private static List<Connection> pool = Collections.synchronizedList(new ArrayList<Connection>());
	//最开始初始化一些链接到池中
	static{
		for(int i=0;i<10;i++){
			Connection conn = JdbcUtil.getConnection();
			pool.add(conn);
		}
	}
	//从池中获取一个链接
	public static Connection getConnection(){
		if(pool.size()>0){
			return pool.remove(0);
		}else{
			throw new RuntimeException("服务器忙");
		}
	}
	//用完后还回池中
	public static void release(Connection conn){
		pool.add(conn);
	}
}
```

##标准数据源(基于动态代理)
```java
//标准的数据源
public class MyDataSource implements DataSource {
	
	private static List<Connection> pool = Collections.synchronizedList(new ArrayList<Connection>());
	//最开始初始化一些链接到池中
	static{
		for(int i=0;i<10;i++){
			Connection conn = JdbcUtil.getConnection();
			pool.add(conn);
		}
	}
	
	public Connection getConnection() throws SQLException {
		if(pool.size()>0){
			final Connection conn =  pool.remove(0);//数据库驱动的
			Connection proxyConn = (Connection)Proxy.newProxyInstance(conn.getClass().getClassLoader(), 
					conn.getClass().getInterfaces(), new InvocationHandler() {
						public Object invoke(Object proxy, Method method, Object[] args)
								throws Throwable {
							if("close".equals(method.getName())){
								//用户调用的是close方法：还回池中
								return pool.add(conn);
							}else{
								//调用原来对象的对应方法
								return method.invoke(conn, args);
							}
						}
					});
			return proxyConn;
		}else{
			throw new RuntimeException("连接池忙");
		}
	}
	
	public PrintWriter getLogWriter() throws SQLException {
		return null;
	}

	public void setLogWriter(PrintWriter out) throws SQLException {

	}

	public void setLoginTimeout(int seconds) throws SQLException {

	}

	public int getLoginTimeout() throws SQLException {
		return 0;
	}

	public <T> T unwrap(Class<T> iface) throws SQLException {
		return null;
	}

	public boolean isWrapperFor(Class<?> iface) throws SQLException {
		return false;
	}

	public Connection getConnection(String username, String password)
			throws SQLException {
		return null;
	}

}
```
代理数据源
