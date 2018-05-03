---
layout: post
title: [Tomcat源码分析—2.Bootstrap初始化]
categories: [Tomcat]
tags: [tomcat,源码分析,bootstrap初始化]
id: [18918567575552]
fullview: false
---

### 1.Boostrap初始化

进入Bootstrap.init()方法，该方法主要做了如下几个事情。

* 初始化三个类加载器ClassLoader(common、server、shared)，其中common为server和shared的父级类加载器，高版本的server和shared类加载器不再使用，三个都是同一个common类加载器实例。


* 使用server ClassLoader预加载tomcat部分类，以免报AccessControlException异常。

* 使用server ClassLoader实例化Catalina对象。

* 设置shared ClassLoader为Catalina父级类加载器。



```java
    /**
     * Initialize daemon.
     */
    public void init() throws Exception {

        initClassLoaders();

        Thread.currentThread().setContextClassLoader(catalinaLoader);

        SecurityClassLoad.securityClassLoad(catalinaLoader);

        // Load our startup class and call its process() method
        if (log.isDebugEnabled())
            log.debug("Loading startup class");
        Class<?> startupClass =
            catalinaLoader.loadClass
            ("org.apache.catalina.startup.Catalina");
        Object startupInstance = startupClass.newInstance();

        // Set the shared extensions class loader
        if (log.isDebugEnabled())
            log.debug("Setting startup class properties");
        String methodName = "setParentClassLoader";
        Class<?> paramTypes[] = new Class[1];
        paramTypes[0] = Class.forName("java.lang.ClassLoader");
        Object paramValues[] = new Object[1];
        paramValues[0] = sharedLoader;
        Method method =
            startupInstance.getClass().getMethod(methodName, paramTypes);
        method.invoke(startupInstance, paramValues);

        catalinaDaemon = startupInstance;

    }
```

### 2.创建Common ClassLoader

如上，Bootstrap在初始化过程中首先回去创建三个类加载器（common、server、shared）。进入Boostrap.creatClassLoader(name,parent)方法，该方法主要是用于创建类加载器。


```java
    private ClassLoader createClassLoader(String name, ClassLoader parent)
        throws Exception {

        String value = CatalinaProperties.getProperty(name + ".loader");
        if ((value == null) || (value.equals("")))
            return parent;

        value = replace(value);

        List<Repository> repositories = new ArrayList<>();

        String[] repositoryPaths = getPaths(value);

        for (String repository : repositoryPaths) {
            // Check for a JAR URL repository
            try {
                @SuppressWarnings("unused")
                URL url = new URL(repository);
                repositories.add(
                        new Repository(repository, RepositoryType.URL));
                continue;
            } catch (MalformedURLException e) {
                // Ignore
            }

            // Local repository
            if (repository.endsWith("*.jar")) {
                repository = repository.substring
                    (0, repository.length() - "*.jar".length());
                repositories.add(
                        new Repository(repository, RepositoryType.GLOB));
            } else if (repository.endsWith(".jar")) {
                repositories.add(
                        new Repository(repository, RepositoryType.JAR));
            } else {
                repositories.add(
                        new Repository(repository, RepositoryType.DIR));
            }
        }

        return ClassLoaderFactory.createClassLoader(repositories, parent);
    }
```

如上，CatalinaProperties.getProperty()方法首先会去找系统的catalina.config对应值的文件，如果文件不存在，则去查找tomcat目录下conf文件夹下的catalina.properties文件，如果文件不存在，则查找Jar包内的catalina.properties文件，找到对应的common.loader、server.loader、shared.loader key对应的value。如下是Jar包下catalina.properties

的部分信息。

```java
common.loader=${catalina.base}/lib","${catalina.base}/lib/*.jar","${catalina.home}/lib","${catalina.home}/lib/*.jar
server.loader=
shared.loader=
```

因为server.loader和shared.loader的值为空，所以直接返回common类加载器，即common、server、shared为同一类加载器（在旧版本tomcat是三个不同的类加载器）。


之后replace会将${catalina.base}和${catalina.home}替换为实际tomcat路径。最后进入ClassLoaderFactory.createClassLoader()方法，实例化URLClassLoader对象并返回。

### 总结

Bootstrap初始化主要完成了如下几个事情。


* 读取catalina.properties配置文件信息，该文件路径是可配置的。首先去System.getProperties("catalina.config")获取路径；如果没有，则查找tomcat目录下conf文件夹下的catalina.properties文件；如果还是没有，则回去tomcat的Jar包中查找catalina.properties文件。

* 构建common、server、shared ClassLoader，这三个类加载器都是同一个实例。

* 通过common ClassLoader去加载一些必要的类，以免造成AccessControlException异常。

* 最后实例化Catalina对象，并设置其父类加载器shared ClassLoader。




