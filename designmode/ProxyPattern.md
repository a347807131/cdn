#代理设计模式

标签：`AOP`

---

##静态代理

- 基于接口的静态代理
> 注:被代理的对象至少实现了一个需要被代理的接口

- 基于子类的静态代理

###简介

###静态代理示例

代理接口
```java
public interface Human {
	void sing(float money);
	void dance(float money);
	void eat();
}
```

被代理对象
```java
public class SpringBrother implements Human {

	public void sing(float money) {
		System.out.println("拿到钱"+money+"开始唱歌");
	}

	public void dance(float money) {
		System.out.println("拿到钱"+money+"开始跳舞");
	}

	public void eat() {
		System.out.println("吃西餐");
	}

}
```

代理对象
```java
//静态代理
public class Middleman implements Human{
	private Human h;
	public Middleman(Human h){
		this.h = h;
	}
	public void sing(float money) {
		if(money>10000)
			h.sing(money/2);
	}

	public void dance(float money) {
		if(money>20000)
			h.dance(money/2);
	}

	public void eat() {
		
	}

}
```

主方法(基于接口)
```java
public class Main {

	public static void main(String[] args) {
		final Human bitch = new SpringBrother();
		/*
		 ClassLoadder loader:类加载器。固定写法：和被代理人用相同的类加载器即可。
		 Interface[] interfaces:代理对象要实现的接口。固定写法：和被代理人相同即可。
		 InvocationHandler h：接口。如何代理。策略设计模式。
		 */
		Human middleman = (Human)Proxy.newProxyInstance(bitch.getClass().getClassLoader(), 
				bitch.getClass().getInterfaces(), 
				new InvocationHandler() {
					//匿名内部类：具体策略
					//只要执行任何方法，都会经过该方法
					/*
					 返回值：当前调用的方法的返回值
					 Object proxy：当前代理对象的引用。
					 Method method:当前执行的方法。
					 Object[] args:当前指定的方法用到的参数
					 */
					public Object invoke(Object proxy, Method method, Object[] args)
							throws Throwable {
						if("sing".equals(method.getName())){
							float money = (Float)args[0];
							if(money>10000){
								method.invoke(bitch, money/2);
							}
						}
						if("dance".equals(method.getName())){
							float money = (Float)args[0];
							if(money>20000){
								method.invoke(bitch, money/2);
							}
						}
						return null;
					}
				});
		middleman.sing(100000);
		middleman.dance(200000);
	}

}
```
基于子类
```java
public class Main {
	public static void main(String[] args) {
		final SpringBrother bitch = new SpringBrother();//不能是final的
		//用他的子类作为代理类
		/*
		 Class superclass:父类型
		 Callback callback：如何代理
		 */
		SpringBrother middleman = (SpringBrother) Enhancer.create(SpringBrother.class, new MethodInterceptor(){
			//匿名内部类：具体策略
			//只要执行任何方法，都会经过该方法
			/*
			 返回值：当前调用的方法的返回值
			 Object proxy：当前代理对象的引用。
			 Method method:当前执行的方法。
			 Object[] args:当前指定的方法用到的参数
			 */
			public Object intercept(Object proxy, Method method, Object[] args,
					MethodProxy arg3) throws Throwable {
				if("sing".equals(method.getName())){
					float money = (Float)args[0];
					if(money>10000){
						method.invoke(bitch, money/2);
					}
				}
				if("dance".equals(method.getName())){
					float money = (Float)args[0];
					if(money>20000){
						method.invoke(bitch, money/2);
					}
				}
				return null;
			}
			
		});
		middleman.sing(100000);
		middleman.dance(200000);
	}
}
```

##动态代理

- 基于接口的动态代理
- 基于子类的动态代理


