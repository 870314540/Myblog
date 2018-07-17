> 动态语言，是指程序在运行时可以改变其结构：新的函数可以引进，已有的函数可以被删除等结构上的变化。

从动态语言能在运行时改变程序结构结构或则变量类型上看，Java和C、C++一样都不属于动态语言。 

Java通过反射机制，可以在程序运行时加载，探知和使用编译期间完全未知的类，并且可以生成相关类对象实例，从而可以调用其方法或则改变某个属性值。
所以JAVA也可以算得上是一个半动态的语言。


###Java反射

 在运行状态中，对于任意一个类都能够知道这个类所有的属性和方法；并且对于任意一个对象，都能够调用它的任意一个方法；
 这种动态获取信息以及动态调用对象方法的功能成为Java语言的反射机制
 
 
 反射API用来生成JVM中的类、接口或则对象的信息。 
- Class类：反射的核心类，可以获取类的属性，方法等信息。 
- Field类：Java.lang.reflec包中的类，表示类的成员变量，可以用来获取和设置类之中的属性值。 
- Method类： Java.lang.reflec包中的类，表示类的方法，它可以用来获取类中的方法信息或者执行方法。 
- Constructor类： Java.lang.reflec包中的类，表示类的构造方法。


### 操作示例

1、三种方法 获取class
```
Class clazz = Person.class;

Class clazz = Class.forName("类的全路径"); //最常用

Person p  = new Person();
Class clazz = p.getClass();

```

2、获取全部的方法\属性\构造方法
> Declared获取全部，不管是私有和公有   1.获取访问类的Class对象   2.调用Class对象的方法返回访问类的方法和属性信息
 
```
//including public, protected, default (package)  access, and private methods, but excluding inherited methods.
  Method[] method=clazz.getDeclaredMethods();

  //This includes public, protected, default (package) access, and private fields, but excludes inherited fields.
  Field[] field=clazz.getDeclaredFields();

  Constructor[] constructor=clazz.getDeclaredConstructors();

```
3、创建对象
当我们获取到所需类的Class对象后，可以用它来创建对象，有两种方法：

- 使用Class对象的**newInstance()**方法来创建该Class对象对应类的实例，但是**这种方法要求该Class对象对应的类有默认的空构造器。**

```
Class<?> userClass = Class.forName("com.example.demo.test.test1.UserEntity");
UserEntity userEntity = (UserEntity) userClass.newInstance();
```

- 先使用Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance()方法来创建 Class对象对应类的实例,通过这种方法可以选定构造方法创建实例。

```
 //获取构造方法
 Constructor c=clazz.getDeclaredConstructor(String.class,String.class,int.class);
 //创建对象并设置属性
 Person p1=(Person) c.newInstance("张四","女",20);

```

4、常规使用步骤

反射调用一般分为3个步骤：

得到要调用类的class
得到要调用的类中的方法(Method)
方法调用(invoke)

```
        Class<?> userClass = Class.forName("com.example.demo.test.test1.UserEntity");
        UserEntity userEntity = (UserEntity) userClass.newInstance();

        //第一种方法
        int money = userEntity.getMoney();

        //第二种方法,（无参的示例：借钱）
        Method getMoney = userClass.getMethod("getMoney");//得到方法对象
        Object money2 = getMoney.invoke(userEntity);//调用借钱方法，得到返回值
        System.out.println("实际拿到钱为：" + money2);

        //第二种方法,（单个参数的示例：还钱）
        Method repay1 = userClass.getMethod("repay",int.class);//得到方法对象,有参的方法需要指定参数类型
        repay1.invoke(userEntity,3000);//执行还钱方法，有参传参

        //第二种方法,（多个参数的示例：还钱）
        Method repay2 = userClass.getMethod("repay", String.class,int.class);//得到方法对象,有参的方法需要指定参数类型
        repay2.invoke(userEntity,"小飞",5000);//执行还钱方法，有参传参

```

###深入一下

`getDeclaredMethod*()`
　　获取的是类自身声明的所有方法，包含public、protected和private方法。

`getMethod*()`
　　获取的是类的所有共有方法，这就包括自身的所有public方法，和从基类继承的、从接口实现的所有public方法。

 在方法调用中，**参数类型必须正确**，这里需要注意的是不能使用包装类替换基本类型，比如不能使用Integer.class代替int.class。
 否则会抛 `java.lang.NoSuchMethodException`

 
 如果直接通过反射给类的private成员变量赋值，是不允许的，这时我们可以通过setAccessible方法解决。
 前面`field.setAccessible(true);//设置允许访问 ` 


`public Object invoke(Object obj,Object... args)`,这个方法对带有指定参数的指定对象调用由此 Method 对象表示的底层方法。个别参数被自动解包，以便与基本形参相匹配，基本参数和引用参数都随需服从方法调用转换。






