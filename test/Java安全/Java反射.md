#### 获取原始类

`Class.forName("完整类名带包名")`

`对象.getClass()` 对象指已实例化的对象

`任何类型.Class`

以上三种方式获得的都是实例化的Class对象

#### 通过反射实例化对象

`对象.newInstance()`：对象指的是实例化的Class对象，返回一个 由实例化的Class对象 实例化的对象，实际上是调用了实例化的Class对象无参的构造方法，须保证无参构造存在才可以，否则会抛出 `java.lang.InstantiationException`异常。

#### Class类方法

```
方法名	备注
public T newInstance()							创建对象
public String getName()							返回完整类名带包名
public String getSimpleName()						返回类名
public Field[] getFields()						返回类中public修饰的属性
public Field[] getDeclaredFields()					返回类中所有的属性
public Field getDeclaredField(String name)				根据属性名name获取指定的属性
public native int getModifiers()					获取属性的修饰符列表,返回的修饰符是一个数字，每个数字是修饰符的代号
							 		(一般配合Modifier类的toString(int x)方法使用)
public Method[] getDeclaredMethods()					返回类中所有的实例方法
public Method getDeclaredMethod(String name, Class<?>… parameterTypes)	根据方法名name和方法形参获取指定方法
public Constructor<?>[] getDeclaredConstructors()			返回类中所有的构造方法
public Constructor getDeclaredConstructor(Class<?>… parameterTypes)	根据方法形参获取指定的构造方法
----	----
public native Class<? super T> getSuperclass()				返回调用类的父类
public Class<?>[] getInterfaces()					返回调用类实现的接口集合
```

#### 获取Method

`Class对象.getDeclaredMethods()`：返回一个Method类数组

`Class对象.getDeclaredMethod(function_name, arg1_type, arg2_type)`：第一个位置填类函数名（类型为字符串），其余参数为该参数类型的Class类

Method类方法：

```
public String getName()				返回方法名
public int getModifiers()			获取方法的修饰符列表,返回的修饰符是一个数字，每个数字是修饰符的代号
						(一般配合Modifier类的toString(int x)方法使用)
public Class<?> getReturnType()			以Class类型，返回方法类型(一般配合Class类的getSimpleName()方法使用)
public Class<?>[] getParameterTypes()		返回方法的修饰符列表（一个方法的参数可能会有多个。）
						结果集一般配合Class类的getSimpleName()方法使用
public Object invoke(Object obj, Object… args)	调用方法
```

#### 通过Method来调用函数

`Method对象.invoke(obj,arg1,arg2)`：obj为已经创建的对象（由实例化的Class对象 实例化的对象），其余为要传入的实参
