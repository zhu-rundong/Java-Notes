# JVM是如何实现反射的

## 什么是反射？

反射（Reflection）是 Java 语言的一个非常重要的特性，它允许运行中的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性。

## 反射调用的实现

反射的调用，也就是 `Method.invoke` 方法

```java
    public Object invoke(Object obj, Object... args)
        throws IllegalAccessException, IllegalArgumentException,
           InvocationTargetException {
        if (!override) {
            if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
                Class<?> caller = Reflection.getCallerClass();
                checkAccess(caller, clazz, obj, modifiers);
            }
        }
        MethodAccessor ma = methodAccessor;             // read volatile
        if (ma == null) {
            ma = acquireMethodAccessor();
        }
        return ma.invoke(obj, args);
    }
```

通过源码发现，`Method.invoke` 方法实际上委派给 `MethodAccessor` 来处理。 `MethodAccessor` 是一个接口，它有两个已有的具体实现：

- `NativeMethodAccessorImpl`：本地方法来实现反射调用
- `DelegatingMethodAccessorImpl`：委派模式来实现反射调用

每个 Method 实例的第一次反射调用都会生成一个委派实现，它所委派的具体实现便是一个本地实现。本地实现非常容易理解。当进入了 JVM 内部之后，我们便拥有了 Method 实例所指向方法的具体地址。这时候，反射调用无非就是将传入的参数准备好，然后调用进入目标方法。

通过打印反射调用到目标方法时的栈轨迹来具体看一下：

```java
public class JavaReflectionDemo {

    public static void target(int i) {
        new Exception("#" + i).printStackTrace();
    }

    public static void main(String[] args) throws Exception {
        Class<?> klass = Class.forName("com.zrd.jvm.JavaReflectionDemo");
        Method method = klass.getMethod("target", int.class);
        method.invoke(null, 0);
    }
}
// OutPut (jdk8)
java.lang.Exception: #0
	at com.zrd.jvm.JavaReflectionDemo.target(JavaReflectionDemo.java:14)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at com.zrd.jvm.JavaReflectionDemo.main(JavaReflectionDemo.java:20)
```

可以看到，反射调用先是调用了`Method.invoke`，然后进入委派实现（`DelegatingMethodAccessorImpl`），再然后进入本地实现（`NativeMethodAccessorImpl`），最后到达目标方法。

为什么反射调用还要采取委派实现作为中间层？直接交给本地实现不可以么？

Java 的反射调用机制还设立了另一种动态生成字节码的实现（下称动态实现），直接使用 invoke 指令来调用目标方法。之所以采用委派实现，便是为了能够在本地实现以及动态实现中切换。

动态实现和本地实现相比，其运行效率要快上 20 倍。这是因为动态实现无需经过 Java 到 C++ 再到 Java 的切换，但由于生成字节码十分耗时，仅调用一次的话，反而是本地实现要快上 3 到 4 倍。

考虑到许多反射调用仅会执行一次，JVM 设置了一个阈值 15（可以通过 -`Dsun.reflect.inflationThreshold`= 来调整），当某个反射调用的调用次数在 15 之下时，采用本地实现；当超过 15 时，便开始动态生成字节码，并将委派实现的委派对象切换至动态实现，这个过程我们称之为 `Inflation`。

```java
public class JavaReflectionDemo {
    public static void main(String[] args) throws Exception {
        Class<?> klass = Class.forName("com.zrd.jvm.JavaReflectionDemo");
        Object o = klass.newInstance();
        Method method = klass.getMethod("target", int.class);
        for (int i = 1; i <= 20; i++) {
            method.invoke(o, i);
        }
    }
    public static void target(int i) {
        System.out.println("#" + i);
    }
}
```

执行上面代码时时加入`-XX:+TraceClassLoading`参数，可以看到控制台输出一堆 log，把其中相关的部分截取出来如下：

```shell
[Loaded com.zrd.jvm.JavaReflectionDemo from file:/D:/IdeaProjects/github/learning/jvm/target/classes/]
[Loaded sun.launcher.LauncherHelper$FXHelper from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
[Loaded java.lang.Class$MethodArray from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
[Loaded java.lang.Void from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
#1
#2
#3
#4
#5
#6
#7
#8
#9
#10
#11
#12
#13
#14
#15
[Loaded sun.reflect.ClassFileConstants from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
[Loaded sun.reflect.AccessorGenerator from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
[Loaded sun.reflect.MethodAccessorGenerator from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
[Loaded sun.reflect.ByteVectorFactory from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
[Loaded sun.reflect.ByteVector from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
[Loaded sun.reflect.ByteVectorImpl from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
[Loaded sun.reflect.ClassFileAssembler from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
[Loaded sun.reflect.UTF8 from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
[Loaded sun.reflect.Label from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
[Loaded sun.reflect.Label$PatchInfo from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
[Loaded sun.reflect.MethodAccessorGenerator$1 from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
[Loaded sun.reflect.ClassDefiner from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
[Loaded sun.reflect.ClassDefiner$1 from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
[Loaded sun.reflect.GeneratedMethodAccessor1 from __JVM_DefineClass__]
#16
#17
#18
#19
#20
[Loaded java.lang.Shutdown from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
[Loaded java.lang.Shutdown$Lock from C:\Develop Program\java\jdk1.8.0_181\jre\lib\rt.jar]
```

可以看到，在第 16 次的时候，JVM 额外加载了不少类。其中，最重要的当属 `GeneratedMethodAccessor1`（第 33 行），这时便开始动态生成字节码，并将委派实现的委派对象切换至动态实现。

反射调用的 Inflation 机制是可以通过参数（`-Dsun.reflect.noInflation=true`）来关闭的。这样一来，在反射调用一开始便会直接生成动态实现，而不会使用委派实现或者本地实现。

## 反射调用的开销

在上面的例子中，我们先后进行了 `Class.forName`，`Class.getMethod` 以及 `Method.invoke` 三个操作。其中，`Class.forName` 会调用本地方法，`Class.getMethod` 则会遍历该类的公有方法。如果没有匹配到，它还将遍历父类的公有方法，这两个操作都非常费时。

> 注意，以 `getMethod` 为代表的查找方法操作，会返回查找得到结果的一份拷贝。因此，我们应当避免在热点代码中使用返回 Method 数组的 `getMethods` 或者 `getDeclaredMethods` 方法，以减少不必要的堆空间消耗。

在实践中，我们往往会在应用程序中缓存 `Class.forName` 和 `Class.getMethod` 的结果。下面就只关注反射调用本身的性能开销，主要原因有三个：

1. 变长参数方法导致的 Object 数组
2. 基本类型的自动装箱、拆箱
3. 方法内联

反射调用时还会做两个额外操作：

第一，由于 `Method.invoke` 是一个变长参数方法，在字节码层面它的最后一个参数会是 Object 数组。Java 编译器会在方法调用处生成一个长度为传入参数数量的 Object 数组，并将传入参数一一存储进该数组中。

第二，由于 Object 数组不能存储基本类型，Java 编译器会对传入的基本类型参数进行自动装箱。

这两个操作除了带来性能开销外，还可能占用堆内存，使得 `GC` 更加频繁。那么，如何消除这部分开销呢？

第一个因变长参数而自动生成的 Object 数组。既然每个反射调用对应的参数个数是固定的，那么我们可以选择在循环外新建一个 Object 数组，设置好参数，并直接交给反射调用。

```java
public class JavaReflectionDemo {
    public static void main(String[] args) throws Exception {
        Class<?> klass = Class.forName("com.zrd.jvm.JavaReflectionDemo");
        Object o = klass.newInstance();
        Method method = klass.getMethod("target", int.class);
        // 在循环外构造参数数组
        Object[] arg = new Object[1];
        for (int i = 1; i <= 20; i++) {
            arg[0] = i;
            method.invoke(o, arg);
        }
    }
    public static void target(int i) {
        System.out.println("#" + i);
    }
}
```

第二个自动装箱，Java 缓存了[-128, 127]中所有整数所对应的 Integer 对象。当需要自动装箱的整数在这个范围之内时，便返回缓存的 Integer，否则需要新建一个 Integer 对象。

因此，我们可以将这个缓存的范围扩大至覆盖 128（对应参数`-Djava.lang.Integer.IntegerCache.high`=128），便可以避免需要新建 Integer 对象的场景。

至于方法内联，接下来文章中再详细解释。

