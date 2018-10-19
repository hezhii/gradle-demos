# Gradle demos

根据 Gradle 官网上的[教程](https://gradle.org/guides/)编写的例子，学习 Gradle 的使用。

## 创建简单地 Gradle 项目

主要内容：

- 如何安装 Gradle
- 使用 Gradle 初始化项目
- Gradle Wrapper
- 如何创建任务
- 如何使用插件
- 命令行工具
- 构建扫描

具体内容见[博客](https://blog.whezh.com/gradle-basic-usage/)

## 构建 Java 应用

初始化应用时指定类型为 `java-application`：

```bash
gradle init --type java-application
```

执行构建时会执行测试用例，可以查看报告

## 构建 Java web 应用

构建 Java web 应用使用 [war](https://docs.gradle.org/4.10-rc-2/userguide/war_plugin.html) 插件，该插件继承了 java 插件并提供 web 应用的支持。
默认使用 `src/main/webapp` 文件存放 web 相关资源。

```text
webdemo/
    src/
        main/
            java/
            webapp/
        test
            java/
```

配置使用 `war` 插件
```gradle
plugins {
    id 'war'  
}
```

生成 Gradle wrapper：`gradle wrapper --gradle-version=4.10-rc-2`

使用 `greety` 插件可以很方便的在 Jetty 或 Tomcat 中运行和测试 web 应用：
```gradle
plugins {
    id 'org.gretty' version '2.2.0' 
}
```

启动 web 应用：`./gradlew appRun`。

单元测试使用 [Mockito](http://site.mockito.org/) mock 数据。
功能测试使用 [Selenium](https://www.seleniumhq.org/)。
```gradle
gretty {
    integrationTestTask = 'test' // 告诉 gretty 在测试时启动服务，测试完成后关闭
}

// ... rest from before ...

dependencies {
    providedCompile 'javax.servlet:javax.servlet-api:3.1.0'
    testCompile 'junit:junit:4.12'
    testCompile 'org.mockito:mockito-core:2.7.19'
    testCompile 'io.github.bonigarcia:webdrivermanager:1.6.1' 
    testCompile 'org.seleniumhq.selenium:selenium-java:3.3.1' 
}
```