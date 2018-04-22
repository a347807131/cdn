#java数据库连接规范

##JdbcUtil
```java
//与具体的数据库解耦
public class JdbcUtil {

	private static String driverClass;
	private static String url;
	private static String user;
	private static String password;
	
	static{
		try {
			InputStream in = JdbcUtil.class.getClassLoader().getResourceAsStream("dbcfg.properties");
			Properties props = new Properties();
			props.load(in);
			
			driverClass = props.getProperty("driverClass");
			url = props.getProperty("url");
			user = props.getProperty("user");
			password = props.getProperty("password");
			Class.forName(driverClass);
		} catch (Exception e) {
			throw new ExceptionInInitializerError(e);
		}
		
	}

	public static Connection getConnection() {
		try {
			Connection conn = DriverManager.getConnection(url,user,password);
			return conn;
		} catch (SQLException e) {
			throw new RuntimeException(e);
		}
	}

	public static void release(ResultSet rs, Statement stmt, Connection conn) {
		if (rs != null) {
			try {
				rs.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
			rs = null;
		}
		if (stmt != null) {
			try {
				stmt.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
			stmt = null;
		}
		if (conn != null) {
			try {
				conn.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
			conn = null;
		}
	}
}
```

##TransactionManager
```java
//事务管理器 解耦
public class TransationManager {
	private static ThreadLocal<Connection> tl = new ThreadLocal<Connection>();
	public static Connection getConnection(){
		Connection conn = tl.get();//从当前线程上获得链接
		if(conn==null){
			conn = DBCPUtil.getConnection();
			tl.set(conn);//把链接绑定到当前线程上
		}
		return conn;
	}
	public static void startTransaction(){
		Connection conn = getConnection();
		try {
			conn.setAutoCommit(false);
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}
	public static void commit(){
		Connection conn = getConnection();
		try {
			conn.commit();
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}
	public static void rollback(){
		Connection conn = getConnection();
		try {
			conn.rollback();
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}
	public static void release(){
		Connection conn = getConnection();
		try {
			conn.close();
			tl.remove();//与线程池有关，解除关系
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}
}
```

##对象代理工厂
```java
public class BeanFactory {
	/**
	 * 产生BusinessService的实例
	 * @param isProxy ture，返回代理类。false，原来的类
	 * @return
	 */
	public static BusinessService getBusinessService(boolean isProxy){
		final BusinessService s = new BusinessServiceImpl();
		if(isProxy){
			//返回实现的代理类
			BusinessService proxyS = (BusinessService)Proxy.newProxyInstance(s.getClass().getClassLoader(), 
					s.getClass().getInterfaces(), new InvocationHandler() {
						
						@Override
						public Object invoke(Object proxy, Method method, Object[] args)
								throws Throwable {
							Object rtValue = null;
							try{
								long time = System.currentTimeMillis();
								TransationManager.startTransaction();
								rtValue = method.invoke(s, args);
								TransationManager.commit();
								System.out.println(method.getName()+" cost time "+(System.currentTimeMillis()-time) +" millis second");
							} catch (Exception e) {
									TransationManager.rollback();
									throw new RuntimeException(e);
							}finally{
									TransationManager.release();
							}
							return rtValue;
						}
					});
			return proxyS;
		}else{
			return s;
		}
	}
}
```

##达到的目的

- 默认就开启了事务支持
- 高度解耦
```java
public class Test{
    public void t1(){
    	BusinessService s = BeanFactory.getBusinessService(true);
    		System.out.println(s.getClass().getName());
    		s.transfer("aaa", "bbb", 100);
    }
}
```

