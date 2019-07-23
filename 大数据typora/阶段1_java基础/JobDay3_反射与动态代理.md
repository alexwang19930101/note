## 反射与动态代理(待总结 类加载)

### 1.反射

#### 1.1 概述

​	Java对象运行时会有两种类型：编译时类型和运行时类型。比如一句Person p = new Student()反映多态(成员变量都看左，方法运行时看右)的代码中会生成一个p变量，该变量编译时类型为Person运行时为Student。为了对象的运行时方法需要在运行时知道对象和类的真实信息。有以下两种方式：

1. 假设在编译时运行时知道对象和类的具体信息，可以使用`instanceof`判断后进行`强制类型转换`。
2. 编译时无需知道对象和类的具体信息，运行时发现 --> 反射。

#### 1.2 获取Java.lang.Class对象的3种方式

1. `Class.forName(String className)`静态方法。`String className`是类的全限定类名(包名.类名)
2. 调用该类的静态成员变量`class`,eg:`Person.class`
3. 调用对象的`getClass()`方法。

实际上一般用第二种方式来获取Class对象，不过在方法一加载字节码文件在导包的情况下使用广泛

#### 1.3获取信息的主要方法

以下是一些Class的主要方法

|       return        | description                                                  |
| :-----------------: | ------------------------------------------------------------ |
|  ` Constructor<T>`  | `getConstructor(Class<?>... parameterTypes)`  返回一个 `Constructor` 对象，它反映此 `Class`  对象所表示的类的指定公共构造方法。 |
| ` Constructor<?>[]` | `getConstructors()`             返回一个包含某些 `Constructor` 对象的数组，这些对象反映此 `Class`  对象所表示的类的所有公共构造方法。 |
|  ` Constructor<T>`  | `getDeclaredConstructor(Class<?>... parameterTypes) `  返回一个 `Constructor` 对象，该对象反映此 `Class`  对象所表示的类或接口的指定构造方法。 |
| ` Constructor<?>[]` | `getDeclaredConstructors()`             返回 `Constructor` 对象的一个数组，这些对象反映此 `Class`  对象表示的类声明的所有构造方法。 |
|      ` Method`      | `getMethod(String name, Class<?>... parameterTypes)`             返回一个 `Method` 对象，它反映此 `Class`  对象所表示的类或接口的指定公共成员方法。 |
|     ` Method[]`     | `getMethods()`             返回一个包含某些 `Method` 对象的数组，这些对象反映此 `Class`  对象所表示的类或接口（包括那些由该类或接口声明的以及从超类和超接口继承的那些的类或接口）的公共 *member* 方法。 |
|      ` Method`      | `getDeclaredMethod(String name, Class<?>... parameterTypes)`             返回一个 `Method` 对象，该对象反映此 `Class`  对象所表示的类或接口的指定已声明方法。 |
|     ` Method[]`     | `getDeclaredMethods()`             返回 `Method` 对象的一个数组，这些对象反映此 `Class`  对象表示的类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法。 |
|     ` Field[]`      | `getFields()`             返回一个包含某些 `Field` 对象的数组，这些对象反映此 `Class`  对象所表示的类或接口的所有可访问公共字段。 |
|      ` Field`       | `getField(String name)`             返回一个 `Field` 对象，它反映此 `Class`  对象所表示的类或接口的指定公共成员字段。 |
|      ` Field`       | `getDeclaredField(String name)`             返回一个 `Field` 对象，该对象反映此 `Class`  对象所表示的类或接口的指定已声明字段。 |
|     ` Field[]`      | `getDeclaredFields()`             返回 `Field` 对象的一个数组，这些对象反映此 `Class`  对象所表示的类或接口所声明的所有字段。 |
|    ` Class<?>[]`    | `getInterfaces()`  确定此对象所表示的类或接口实现的接口。    |
|      ` Type[]`      | `getGenericInterfaces()`             返回表示某些接口的 `Type`，这些接口由此对象所表示的类或接口直接实现。 |

​	对于不加declareed的方法只能获取到public标识的部分信息，同时通过declareed获取到所有信息后需要通过setAccessable方法来设置强制操作private等类型的原本不可操作的变量或方法。

​	getInterfaces()对于动态代理创建动态代理对象很有用。而`getGenericInterfaces()`可以返回Type ，是 Java 编程语言中所有类型的公共高级接口。它们包括原始类型、参数化类型、数组类型、类型变量和基本类型。` eg：ParameterizedType `表示参数化类型，如 Collection<String>参数化类型在反射方法首次需要时创建（在此包中指定）。当创建参数化类型 p 时，p 实例化的一般类型声明会被解析，并且按递归方式创建 p  的所有类型参数。有关类型变量创建过程的详细信息，请参阅 `TypeVariable`。重复创建的参数化类型无效。 

 

| return  | description                                                  |
| ------- | ------------------------------------------------------------ |
| `[]`    | `getActualTypeArguments()`             返回表示此类型实际类型参数的 `Type` 对象的数组。 |
| `Type`  | `getRawType()`             返回 `Type` 对象，表示声明此类型的类或接口。 |
| ` Type` | `getOwnerType()`             返回 `Type` 对象，表示此类型是其成员之一的类型。 |



##### costructor的主要方法

|   返回值    | description                                                  |
| :---------: | ------------------------------------------------------------ |
|    ` T`     | `newInstance(Object... initargs)`             使用此 `Constructor`  对象表示的构造方法来创建该构造方法的声明类的新实例，并用指定的初始化参数初始化该实例。 |
| ` Class<T>` | `getDeclaringClass()`             返回 `Class` 对象，该对象表示声明由此 `Constructor`  对象表示的构造方法的类 |

method

| 返回值      | description                                                  |
| ----------- | ------------------------------------------------------------ |
| ` Object`   | `invoke(Object obj, Object... args)`             对带有指定参数的指定对象调用由此 `Method` 对象表示的底层方法。 |
| ` Class<?>` | `**getReturnType**()`             返回一个 `Class` 对象，该对象描述了此 `Method`  对象所表示的方法的正式返回类型。 |

field

| 返回值    | description                                                  |
| --------- | ------------------------------------------------------------ |
| ` void`   | `set(Object obj, Object value)`             将指定对象变量上此 `Field` 对象表示的字段设置为指定的新值。 |
| ` Object` | `get(Object obj)`             返回指定对象上此 `Field` 表示的字段的值。 |

以下是一些操作

##### constructor

```java
package com.itheima;

import java.lang.reflect.Constructor;

public class ConstructorDemo {
	public static void main(String[] args) throws ReflectiveOperationException {
		Class class1 = Class.forName("com.itheima.Student");
		//newInstance()实际上是调用无参构造,如果是私有的是获取不到的
		Object object = class1.newInstance();
		
		Student stu = (Student) object;
		System.out.println(stu.name);
		System.out.println("------------");
		
		//getConstructors()获取所有的public的构造方法
		Constructor[] constructors = class1.getConstructors();
		for (Constructor constructor : constructors) {
			System.out.println(constructor);
		}
		
		System.out.println("------------");
		/**
		 * 返回一个 Constructor 对象，它反映此 Class 对象所表示的类的指定公共构造方法。parameterTypes 参数是 Class 对象的一个数组，这些 Class 对象按声明顺序标识构造方法的形参类型。 如果此 Class 对象表示非静态上下文中声明的内部类，则形参类型作为第一个参数包括显示封闭的实例。 
		 * 要反映的构造方法是此 Class 对象所表示的类的公共构造方法，其形参类型与 parameterTypes 所指定的参数类型相匹配。 
		 * 参数：
		 * 	parameterTypes - 参数数组 
		 * 返回：
		 * 	与指定的 parameterTypes 相匹配的公共构造方法的 Constructor 对象 
		 * 抛出： 
		 * 	NoSuchMethodException - 如果找不到匹配的方法。 
		 * 	SecurityException - 如果存在安全管理器 s，并满足下列任一条件： 
		 * 		调用 s.checkMemberAccess(this, Member.PUBLIC) 拒绝访问构造方法 
		 * 		调用者的类加载器不同于也不是当前类的类加载器的一个祖先，并且对 s.checkPackageAccess() 的调用拒绝访问该类的包 
		 * 
		 * 注意是公共的 public
		 */
		Constructor constructor = class1.getConstructor(String.class,int.class);
		System.out.println(constructor);
		Constructor constructor1 = class1.getConstructor();
		System.out.println(constructor1);
		System.out.println("------------");
		
		/**
		 * getDeclaredConstructors()
		 * 返回 Class 对象的一个数组，这些对象反映声明为此 Class 对象所表示的类的成员的所有类和接口。
		 * 包括该类所声明的公共、保护、默认（包）访问及私有类和接口，但不包括继承的类和接口。
		 * 如果该类不将任何类或接口声明为成员，或者此 Class 对象表示基本类型、数组类或 void，则此方法返回一个长度为 0 的数组。 
		 * 返回：
		 * 	Class 对象的数组，表示该类的所有 declared 成员 
		 */
		Constructor[] delConstructors = class1.getDeclaredConstructors();
		for (Constructor constructor11 : delConstructors) {
			System.out.println(constructor11);
		}
		
		System.out.println("------------");
		Constructor conPri = class1.getDeclaredConstructor(String.class);
		conPri.setAccessible(true);
		conPri.newInstance("a");
	}
}
```

##### Field

```java
package com.itheima;

import java.lang.reflect.Field;

public class FieldDemo {
	public static void main(String[] args) throws ReflectiveOperationException {
		@SuppressWarnings("unchecked")
		Class<Student> class1 = (Class<Student>) Class.forName("com.itheima.Student");

		Field[] fields = class1.getFields();
		for (Field field : fields) {
			System.out.println(field);
		}
		
		Field fieldId = class1.getField("id");
		System.out.println(fieldId);
		
		Student newInstance = class1.newInstance();
		//修改属性值set(Object obj, Object value),Object obj是要操作的对象
		fieldId.set(newInstance, 123);
		//获取属性值
		System.out.println(fieldId.getInt(newInstance));
		System.out.println("-----------");
		
		Field[] fieldsAll = class1.getDeclaredFields();
		for (Field field : fieldsAll) {
			System.out.println(field);
		}
		
		Field fieldAll = class1.getDeclaredField("age");
		System.out.println(fieldAll);
		System.out.println("-----------");
		
		fieldAll.setAccessible(true);
		fieldAll.set(newInstance, 12);
		System.out.println(fieldAll.get(newInstance));
		
	}
}
```

##### Method

```java
package com.itheima;

import java.lang.reflect.Method;

public class MethodDemo {
	public static void main(String[] args) throws ReflectiveOperationException {
		Class<Student> classDemo = (Class<Student>) Class.forName("com.itheima.Student");
		
		Method[] methods = classDemo.getDeclaredMethods();
		for (Method method : methods) {
			System.out.println(method);
		}
		
		System.out.println("-----------");
		
		Method method1 = classDemo.getDeclaredMethod("function");
		Method method2 = classDemo.getDeclaredMethod("function1");
		Method method3 = classDemo.getDeclaredMethod("function",int.class);
		Method method4 = classDemo.getDeclaredMethod("function1",int.class);
		
		System.out.println(method1);
		System.out.println(method2);
		System.out.println(method3);
		System.out.println(method4);
		
		Student newInstance = classDemo.newInstance();
		System.out.println(method1.invoke(newInstance));
		System.out.println(method2.invoke(newInstance));
		System.out.println(method3.invoke(newInstance,1));
		System.out.println(method4.invoke(newInstance,1));
		
		Method method11 = classDemo.getDeclaredMethod("func");
		Method method21 = classDemo.getDeclaredMethod("func1");
		Method method31 = classDemo.getDeclaredMethod("func",int.class);
		Method method41 = classDemo.getDeclaredMethod("func1",int.class);
		
		method11.setAccessible(true);
		method21.setAccessible(true);
		method31.setAccessible(true);
		method41.setAccessible(true);
		
		System.out.println(method11.invoke(newInstance));
		System.out.println(method21.invoke(newInstance));
		System.out.println(method31.invoke(newInstance,1));
		System.out.println(method41.invoke(newInstance,1));
	}
}
```

#### 2. 动态代理

可以通过Proxy类和InvacationHandler接口来生成JDK动态代理类或对象代理对象。

Proxy

| return            | description                                                  |
| ----------------- | ------------------------------------------------------------ |
| `static Class<?>` | `getProxyClass(ClassLoader loader, Class<?>... interfaces)`  返回代理类的 `java.lang.Class` 对象，并向其提供类加载器和接口数组。 |
| `static Object`   | `newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)`             返回一个指定接口的代理类实例，该接口可以将方法调用指派到指定的调用处理程序。 |

​	实际上第一个方法生成动态代理类后要创建对象还是要传入一个InvocationHandler对象，所以第二个方法用的更多。

##### InvocationHandler

`InvocationHandler` 是代理实例的*调用处理程序* 实现的接口。  

每个代理实例都具有一个关联的调用处理程序。对代理实例调用方法时，将对方法调用进行编码并将其指派到它的调用处理程序的 `invoke`  方法。 

| return    | description                                                  |
| --------- | ------------------------------------------------------------ |
| ` Object` | `invoke(Object proxy, Method method, Object[] args)`             在代理实例上处理方法调用并返回结果。 |

​	



当程序使用反射方式为接口或者类生成动态代理对象时，这时它对于接口的实现需要implements一些方法，但，它会去将其替换为InvocationHandler的invoke方法。

##### 动态代理和AOP

如需要实现为类的某个方法添加前置与后置的处理，就可以用到动态代理。

AOP代理就是这样，它包含了目标对象的方法，但是可以在其前后插入一些通用处理。

以下是一个例子：

​	Interface Dog狗的顶层接口

```java
package 动态代理;

public interface Dog {
	void info();
	void run();
}D
```

​	GunDog是狗的一个实现类实现了info和run方法

```java
package 动态代理;

public class GunDog implements Dog{

	@Override
	public void info() {
		// TODO Auto-generated method stub
		System.out.println("一条GunDog");
	}

	@Override
	public void run() {
		// TODO Auto-generated method stub
		System.out.println("GunDog跑");
	}

}
```

​	DogUtils是需要添加的前置后置处理

```java
package 动态代理;

public class DogUtil {
	public void method1 () {
		System.out.println("method1----------");
	}
	public void method2 () {
		System.out.println("method2----------");
	}
}
```

​	MyInvocationHandler是InvocationHandler的一个实现，通过成员变量target接受需要处理的对象，通过反射为target的method方法添加前置与后置操作。

```java
package 动态代理;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

import org.omg.CORBA.portable.InvokeHandler;

public class MyInvocationHandler implements InvocationHandler {
	private Object target;
	public void setTarge(Object target) {
		this.target = target;
	}
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		// TODO Auto-generated method stub
		DogUtil du = new DogUtil();
		du.method1();
		Object result = method.invoke(target, args);
		du.method2();
		return result;
	}

}
```

​	MyProxyFactory是动态代理类的工厂，由传入的target为MyInvocationHandler进行初始化并返回动态代理对象。

```java
package 动态代理;

import java.lang.reflect.Proxy;

public class MyProxyFactory {
	public static Object getProxy(Object target) {
		MyInvocationHandler handler = new MyInvocationHandler();
		handler.setTarge(target);
		
		return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), handler);
	}
}
```

​	Test是一个测试类

```java
package 动态代理;

public class Test {
	public static void main(String[] args) {
		Dog target = new GunDog();
		Dog dog = (Dog) MyProxyFactory.getProxy(target);
		dog.info();
		dog.run();
	}
}
```

​	执行结果如下

```
method1----------
一条GunDog
method2----------
method1----------
GunDog跑
method2----------
```

##### 反射与泛型

使用 泛型 与 Type