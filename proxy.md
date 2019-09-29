[TOC]

# #静态代理

# #动态代理

## ##JDK动态代理 ，生成接口实现类

Proxy类 InvocationHandler类

被代理的类需要继承接口，调用时，也是调用接口的方法。

```java
// 接口
public interface PersonDao {
  public List<Person> getAll();
}

public class Person {
	private String name;
}

// 接口实现类
public class PersonDaoImp implements PersonDao {
	@Override
	public List<Person> getAll() {
		...	
	}
}

public class Main{
  public static void main(String[] args) {
		PersonDaoImp instance = new PersonDaoImp();
		PersonDao pd = (PersonDao) Proxy.newProxyInstance(instance.getClass().getClassLoader(), instance.getClass().getInterfaces(), new InvocationHandler() {
			@Override
			public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
				System.out.println("before");
				Object invoke = method.invoke(instance, args);
				System.out.println("after");
				return invoke;
			}
		});
    List<Person> all = pd.getAll();
	}
}

```



**Proxy.class源码**

```java
public class Proxy implements java.io.Serializable {
  ...
  private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
        proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());
  ...
  
	// 1
	public static Object newProxyInstance(ClassLoader loader,Class<?>[]interfaces, 			InvocationHandler h)throws IllegalArgumentException{
		...
		// 生成代理类字节码，加载代理类的Class类。
		Class<?> cl = getProxyClass0(loader, intfs);
		...
	}
  
	// 2
	private static Class<?> getProxyClass0(ClassLoader loader,Class<?>... interfaces) {
		if (interfaces.length > 65535) {
			throw new IllegalArgumentException("interface limit exceeded");
		}
		// 若proxyClassCache中没有，创建代理类字节码和生成代理类的Class类，如果在缓存里就直接返回。
		// 否则从ProxyClassFactory创建class字节码文件和Class类
		return proxyClassCache.get(loader, interfaces);
	}
  
  
	private static final class ProxyClassFactory implements BiFunction<ClassLoader, Class<?>[], Class<?>> {
    ...
		public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {
      ...
      // 4 
      // 生成代理类的字节码文件，名字为proxyName，实现的接口有interfaces
			byte[] proxyClassFile = ProxyGenerator.generateProxyClass(proxyName, interfaces, accessFlags);
      ...
    }
		...
	}
  ...
}
```



**WeakCache.class源码**

```java
final class WeakCache<K, P, V> {
		private final BiFunction<K, P, ?> subKeyFactory;
  	// 3
		public V get(K key, P parameter) {
			...
      // subKeyFactory.apply(key, parameter) 生成class字节码
      // ProxyClassFactory
			Object subKey = Objects.requireNonNull(subKeyFactory.apply(key, parameter));
      ...
    }
}
```



## ##CGLIB代理，生成代理的子类

被代理的类不需要继承接口。动态创建子类的方法，所以对final方法无法做出代理。

# #面向切面AspectJ

与前两种代理Proxy 和 CGLIB 不同，前两种代理是在运行时生成class字节码，这种是编译时，生成class字节码。



