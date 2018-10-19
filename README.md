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

## 执行 Webpack

通过 [Exec](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.Exec.html) 任务可以执行命令行工具。

```gradle
task webpack(type: Exec) { 
    commandLine "$projectDir/node_modules/.bin/webpack", "app/index.js", "$buildDir/js/bundle.js"
}
```

通过 [up-to-date checks](https://docs.gradle.org/current/userguide/more_about_tasks.html#sec:up_to_date_checks) 增量构建，内容没有修改时不重新构建。
必须指定任务的输入和输出。
```gradle
task webpack(type: Exec) {
    inputs.file("package-lock.json")
    inputs.dir("app")
    // NOTE: Add inputs.file("webpack.config.js") for projects that have it
    outputs.dir("$buildDir/js")

    commandLine "$projectDir/node_modules/.bin/webpack", "app/index.js", "$buildDir/js/bundle.js"
}
```

### 构建缓存

从Gradle 4.0开始，Gradle 可以通过构建缓存避免已经在不同的 VCS 分支上或由其他机器完成的工作。

1. 使 webpack 任务可缓存
```diff
task webpack(type: Exec) {
-   inputs.file("package-lock.json")
-   inputs.dir("app")
+   inputs.file("package-lock.json").withPathSensitivity(PathSensitivity.RELATIVE)  // a
+   inputs.dir("app").withPathSensitivity(PathSensitivity.RELATIVE)
    outputs.dir("$buildDir/js")
+   outputs.cacheIf { true } // b

    commandLine "$projectDir/node_modules/.bin/webpack", "app/index.js", "$buildDir/js/bundle.js"
}
```

a. 声明 package-lock.json 和 app 是可重定位的，[了解详情](https://guides.gradle.org/using-build-cache/#relocatability)。
b. 告诉 Gradle 如果构建缓存可用，则始终缓存此任务的输出。

强烈建议在编写可缓存的任务时使用[自定义任务类](https://docs.gradle.org/current/userguide/custom_tasks.html)。

2. 使用缓存 

```bash
gradle webpack --build-cache
```

来回修改内容，发现后面没有重新构建。

## 编写自定义任务

创建任务并添加一个 action 输出到控制台。

```gradle
task hello { 
    doLast { 
        println 'Hello, World!'
    }
}
```

`gradle tasks --all` 列出可用的任务。

`gradle hello` 执行任务

### 添加分组和任务描述

```diff
task hello {
+   group 'Welcome'
+   description 'Produces a greeting'

    doLast {
        println 'Hello, World'
    }
}
```

### 使信息可配置

在构建脚本中创建一个类。

```gradle
class Greeting extends DefaultTask { // 1,2
    String message //3
    String recipient

    @TaskAction // 4
    void sayGreeting() {
        println "${message}, ${recipient}!" // 5
    }
}

task hello(type: Greeting) { // 6
    group 'Welcome'
    description 'Produces a world greeting'
    message 'Hello' // 7
    recipient 'World'
}
```

1. 由于build.gradle 文件中的构建 DSL 是基于Groovy DSL，因此该类将是 Groovy 类。
2. 虽然 Gradle API 中的其他任务类可以在特定情况下使用，但继承 `{javadoc} /org/gradle/api/DefaultTask.html[DefaultTask]` 是最常见的情况。
3. 添加 `message` 和 `recipient` 属性使此自定义任务类型的实例可配置。
4. 任务默认操作注解。
5. 使用 Groovy 字符串模板打印消息。
6. 通过引用上面定义的 `Greeting` 类指定任务类型。
7. 配置属性。
