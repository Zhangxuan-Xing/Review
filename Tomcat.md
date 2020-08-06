# 类加载机制

> 类加载器的布局结构是一种父子树的形式。然而，Tomcat 违反双亲委派模型，对于一些未加载的非基础类，各个WebAppClassLoader会优先加载，加载不到时再交给CommonClassLoader处理。

## 1.双亲委派

> **为什么要遵循双亲委派机制?**
>
> - 保证核心类的安全。防止开发者取了和jdk核心类库中一样的包名和类名，委托给父类加载器能保证JDK类库的类优先加载。
> - 保证类的唯一性。先检查是否加载过这个类，避免相同类被多次加载。

### 1.1 类加载器

​	如果从Java虚拟机的角度来看的话，其实类加载器只分为两种：一种是启动类加载器(即Bootstrap ClassLoader)，通过使用JNI来实现，我们无法获取到到它的实例；另一种则是Java语言实现`java.lang.ClassLoader`的子类。

​	一般从我们的角度来看，会根据类加载路径会把类加载器分为3种：BootstrapClassLoader ,ExtClassLoader,AppClassLoader.后两者是`sun.misc.Launcher`类的内部类，而前者在JDK源码中是没有与之对应的类的，倒是在`sun.misc.Launcher`中可以看到一些它的加载路径信息。

#### 1.1.1 BootstrapClassLoader

​	BootstrapClassLoader： 引导类加载器，在虚拟机层，用C++编写。用于加载`rt.jar`等运行时基础类库，也被称作“Root ClassLoader”。它从%JAVA_RUNTIME_JRE%/lib目录加载，但并不是将该目录所有的类库都加载，它会加载一些符合文件名称的，例如：rt.jar,resources.jar等。在`sun.misc.Launcher`源码中也可以看得它的加载路径：

`private static String bootClassPath = System.getProperty("sun.boot.class.path");`

#### 1.1.2 ExtClassLoader

​	ExtClassLoader：扩展类加载器，实现类为`sun.misc.Launcher$ExtClassLoader`，加载%JAVA_RUNTIME_JRE%/lib/ext/目录下的jar包，也可以在`sun.misc.Launcher`源码中也可以看加载路径：

`String s = System.getProperty("java.ext.dirs");`

#### 1.1.3 AppClassLoader

​	Appication ClassLoader：应用程序类加载器，实现类为`sun.misc.Launcher$AppClassLoader`。从`sun.misc.Launcher`的构造函数中可以看到，当`AppClassLoader`被初始化以后，它会被设置为当前线程的上下文类加载器以及保存到`Launcher`类的loader属性中，而通过`ClassLoader.getSystemClassLoader()`获取的也正是该类加载器(Launcher.loader)。应用类加载器从用户类路径中加载类库，可以在源码中看到：

`final String s = System.getProperty("java.class.path");`

### 1.2 关系梳理

![1596247985542](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1596247985542.png)

​	由图看到Bootstrap ClassLoader并不在继承链上，因为它是虚拟机内置的类加载器，对外不可见。可以看到顶层`ClassLoader`有一个parent属性，用来表示着类加载器之间的层次关系（双亲委派模型）；注意，`ExtClassLoader`类在初始化时显式指定了parent为null，所以它的父类加载器默认为`Bootstrap ClassLoader`。在tomcat中都是通过扩展`URLClassLoader`来实现自己的类加载器。

```Java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // 首先，查找该类是否已经被加载过了
            Class c = findLoadedClass(name);
            if (c == null) {  //未被加载过
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {  // 父类加载器不为null，则调用父类加载器尝试加载
                        c = parent.loadClass(name, false);
                    } else {   // 父类加载器为null，则调用本地方法，交由启动类加载器加载，所以说ExtClassLoader的父类加载器为Bootstrap ClassLoader
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                }
                if (c == null) { //仍然加载不到，只能由本加载器通过findClass去加载
                    long t1 = System.nanoTime();
                    c = findClass(name);
                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

​	与此同时，扩展类加载器是应用类加载器的父类：

```Java
loader = AppClassLoader.getAppClassLoader(extcl);//loader是ClassLoader的属性,extcl是扩展类加载器实例
```

### 1.3  加载机制

> 在JVM中并不是一次性把所有的文件都加载到，而是一步一步的，按照需要进行加载，比如JVM启动时，会通过不同的类加载器加载不同的类。

![1596246628552](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1596246628552.png)

**类加载步骤如下**：

1. 用户自己的类加载器，把加载请求传给父加载器，父加载器再传给其父加载器，一直到加载器树的顶层。
2. 最顶层的类加载器首先针对其特定的位置加载，如果加载不到就转交给子类。
3. 如果一直到底层的类加载都没有加载到，那么就会抛出异常ClassNotFoundException。

　　**因此，按照这个过程可以想到，如果同样在CLASSPATH指定的目录中和自己工作目录中存放相同的class，会优先加载CLASSPATH目录中的文件。**

![1596248181322](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1596248181322.png)

------

## 2. Tomcat

### 2.1 解决问题

Tomcat 作为一个Web容器，打破双亲委派模型主要为了解决以下问题：

- 不同应用程序依赖不同的版本的同一个第三方库
- 相同版本的类图理应共享
- 容器类库与应用类库隔离
- Jsp文件修改无需重启

### 2.2 类加载器

![1596694540551](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1596694540551.png)

​	顶层类加载和JVM一致，CommonClassLoader、CatalinaClassLoader、SharedClassLoader和WebappClassLoader则是Tomcat自定义的类加载器，它们分别加载 /common/*、/server/*、/shared/*（在tomcat 6之后已经合并到根目录下的lib目录下）和 /WebApp/WEB-INF/* 中的Java类库。其中WebApp类加载器和Jsp类加载器通常会存在多个实例，每一个Web应用程序对应一个WebApp类加载器，每一个JSP文件对应一个Jsp类加载器。

- **CommonLoader**：最基本的类加载器，加载路径中的class可以被Tomcat容器本身以及各个Webapp访问；
- CatalinaLoader：容器私有的类加载器，加载路径中的class对于Webapp不可见；
- SharedLoader：各Webapp共享的类加载器，加载路径中的class对于所有Webapp可见，但对于Tomcat容器不可见；
- **ParallelWebappClassLoader**：各个Webapp私有的类加载器，加载路径中的class只对当前Webapp可见。每个应用对应一个 ParallelWebappClassLoader，这样子可以避免不同应用同名类加载冲突的问题

------

------

- CommonClassLoader能加载的类都可以被Catalina ClassLoader和SharedClassLoader使用，从而实现了公有类库的共用，CatalinaClassLoader和Shared ClassLoader实现加载类相互隔离。
- ParallelWebappClassLoader可使用SharedClassLoader加载的类，各实例之间相互隔离。
- JsperLoader的加载范围仅仅是这个JSP文件所编译出来的那一个.Class文件，它出现的目的就是为了被丢弃：当Web容器检测到JSP文件被修改时，会替换掉目前的JasperLoader的实例，并通过再建立一个新的Jsp类加载器来实现JSP文件的HotSwap功能。
- 若CommonClassLoader想加载ParallelWebappClassLoader中的类，可以使用线程上下文类加载器实现，让父类加载器请求子类加载器去完成类加载的动作。

### 2.3 关系梳理

![1596248542767](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1596248542767.png)

​	Common,Catalina,Shared类加载器是URLClassLoader类的一个实例，只是它们的类加载路径不一样，在tomcat/conf/catalina.properties配置文件中配置。WebAppClassLoader继承WebAppClassLoaderBase,基本所有逻辑都在WebAppClassLoaderBase为中实现了，可以看出 **tomcat以URLClassLoader为基础进行扩展**。

```java
private void initClassLoaders() {
        commonLoader = createClassLoader("common", null);  // commonLoader的加载路径为common.loader
        if( commonLoader == null ) {
            commonLoader=this.getClass().getClassLoader();
        }
        catalinaLoader = createClassLoader("server", commonLoader); // 加载路径为server.loader，默认为空，父类加载器为commonLoader
        sharedLoader = createClassLoader("shared", commonLoader); // 加载路径为shared.loader，默认为空，父类加载器为commonLoader
    }
 private ClassLoader createClassLoader(String name, ClassLoader parent) throws Exception {
        String value = CatalinaProperties.getProperty(name + ".loader");
        if ((value == null) || (value.equals("")))
            return parent;      // catalinaLoader与sharedLoader的加载路径均为空，所以直接返回commonLoader对象，默认3者为同一个对象
    }
```

### 2.4 加载机制

**[Tomcat 类加载步骤：](https://tomcat.apache.org/tomcat-8.5-doc/class-loader-howto.html)**

1. 使用bootstrap启动类加载器
2. 使用Webapp类加载器（对应于WEB-INF/…中的应用）
3. 使用system系统类加载器
4. 使用common类加载器

**核心代码**：`WebAppClassLoaderBase.loadClass()`